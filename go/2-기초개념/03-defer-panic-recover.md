# defer, panic, recover - 정리 작업과 예외 상황 다루기

## 들어가며

파일을 열었으면 닫고, 락을 걸었으면 풀어야 합니다. 다른 언어에서는 이렇게 합니다.

```
Java/JS  → try { ... } finally { close() }
Python   → with open(...) as f:
C#       → using (var f = ...)
Go       → defer f.Close()
```

그리고 Go에는 예외(exception)가 없는 대신, 정말 복구 불가능한 상황을 위해 `panic`과 `recover`가 있습니다.

```
defer   → 함수가 끝날 때 실행할 코드 예약
panic   → 프로그램 실행을 중단하고 스택을 거슬러 올라감
recover → defer 안에서 panic을 붙잡아 정상 흐름으로 복구
```

# 1. defer 기본

## 함수가 끝날 때 실행

```go
package main

import "fmt"

func main() {
	fmt.Println("시작")
	defer fmt.Println("defer 실행") // main이 끝날 때 실행
	fmt.Println("끝")
	// 출력:
	// 시작
	// 끝
	// defer 실행
}
```

## 리소스 정리 패턴

```go
package main

import (
	"fmt"
	"os"
)

func writeFile(path, content string) error {
	f, err := os.Create(path)
	if err != nil {
		return err
	}
	defer f.Close() // 열기 성공 직후에 바로 예약 → 닫는 걸 잊을 일이 없음

	if _, err := f.WriteString(content); err != nil {
		return err // 여기서 반환해도 Close 실행됨
	}
	return nil // 여기서도 Close 실행됨
}

func main() {
	path := os.TempDir() + "/defer-demo.txt"
	if err := writeFile(path, "hello"); err != nil {
		fmt.Println(err)
		return
	}
	data, _ := os.ReadFile(path)
	fmt.Println(string(data))
	os.Remove(path)
	// 출력: hello
}
```

```
✅ 여는 코드 바로 아래에 defer 닫기 → 짝이 눈에 보임
✅ return 이 여러 개여도, panic 이 나도 실행됨
⚠️ 에러 체크 "후에" defer 해야 함 (f가 nil일 때 Close 호출 방지)
```

자주 쓰는 짝:

```go
mu.Lock()
defer mu.Unlock()

resp, err := http.Get(url)
if err != nil { return err }
defer resp.Body.Close()

rows, err := db.Query(q)
if err != nil { return err }
defer rows.Close()

ctx, cancel := context.WithTimeout(ctx, time.Second)
defer cancel()
```

# 2. defer 동작 규칙

## 규칙 1: LIFO (스택 순서)

```go
package main

import "fmt"

func main() {
	for i := range 3 {
		defer fmt.Print(i, " ")
	}
	fmt.Println("루프 끝")
	// 출력:
	// 루프 끝
	// 2 1 0
}
```

나중에 예약한 것부터 실행됩니다. 리소스를 연 순서의 역순으로 닫히니 자연스럽습니다.

## 규칙 2: 인자는 defer 하는 시점에 평가

```go
package main

import "fmt"

func main() {
	x := 1
	defer fmt.Println("defer 인자:", x) // 이 시점의 x(1)가 저장됨

	defer func() {
		fmt.Println("클로저:", x) // 실행 시점의 x를 읽음
	}()

	x = 100
	// 출력:
	// 클로저: 100
	// defer 인자: 1
}
```

## 규칙 3: 이름 있는 반환값을 수정할 수 있다

```go
package main

import (
	"errors"
	"fmt"
)

func double(n int) (result int) {
	defer func() {
		result *= 2 // return 이후, 호출자에게 돌아가기 전에 실행
	}()
	return n // result = n 대입 → defer 실행 → 반환
}

// 실무 활용: Close 에러를 반환 에러에 합치기
type fakeFile struct{}

func (fakeFile) Close() error { return errors.New("close 실패") }

func process() (err error) {
	f := fakeFile{}
	defer func() {
		if cerr := f.Close(); cerr != nil {
			err = errors.Join(err, cerr)
		}
	}()
	return errors.New("처리 실패")
}

func main() {
	fmt.Println(double(5))
	// 출력: 10

	fmt.Println(process())
	// 출력:
	// 처리 실패
	// close 실패
}
```

> 쓰기용으로 연 파일은 `Close()`에서 디스크 쓰기 에러가 날 수 있습니다. 중요한 파일이면 위처럼 Close 에러도 확인합니다.

## 규칙 4: 함수 단위다 (블록 단위가 아님)

```go
// ❌ 루프 안 defer → 함수가 끝날 때까지 파일이 전부 열려 있음
func processAll(paths []string) error {
	for _, p := range paths {
		f, err := os.Open(p)
		if err != nil {
			return err
		}
		defer f.Close() // 10,000개 파일이면 10,000개가 동시에 열린 상태
		// ...
	}
	return nil
}

// ✅ 함수로 분리
func processAll(paths []string) error {
	for _, p := range paths {
		if err := processOne(p); err != nil {
			return err
		}
	}
	return nil
}

func processOne(path string) error {
	f, err := os.Open(path)
	if err != nil {
		return err
	}
	defer f.Close() // processOne 이 끝날 때마다 닫힘
	// ...
	return nil
}
```

## defer 성능

Go 1.14부터 대부분의 defer는 **open-coded defer**로 최적화되어 거의 비용이 없습니다. 성능 때문에 defer를 피할 필요는 없습니다.

# 3. panic

## panic이 일어나는 경우

```go
package main

import "fmt"

func main() {
	defer fmt.Println("defer는 panic 중에도 실행됨")

	s := []int{1, 2, 3}
	i := 5
	fmt.Println(s[i]) // panic: runtime error: index out of range [5] with length 3
}
```

```
런타임 panic 대표 사례:
- 슬라이스/배열 인덱스 범위 초과
- nil 포인터 역참조
- nil 맵에 쓰기
- 타입 단언 실패 (v.(T) 단일 반환 형태)
- 0으로 정수 나누기
- 닫힌 채널에 보내기
```

## panic 진행 과정

```
1. panic 발생 → 현재 함수 실행 중단
2. 현재 함수의 defer 들을 실행
3. 호출한 함수로 올라가서 그 함수의 defer 들을 실행
4. ... 고루틴의 최상단까지 반복
5. recover 가 없으면 → 에러 메시지 + 스택 트레이스 출력 후 프로그램 종료 (exit code 2)
```

## 직접 panic 호출

```go
func MustParse(s string) Config {
	cfg, err := Parse(s)
	if err != nil {
		panic(fmt.Sprintf("MustParse: %v", err))
	}
	return cfg
}
```

```
panic 을 직접 써도 되는 곳:
✅ Must 접두사 함수 (regexp.MustCompile, template.Must)
✅ 도달하면 안 되는 코드 (switch 의 default 에서 "unreachable")
✅ 프로그램 초기화 중 계속 진행이 불가능할 때

쓰면 안 되는 곳:
❌ 일반적인 에러 처리 (파일 없음, 잘못된 입력 등) → error 반환
❌ 라이브러리의 공개 API에서 panic 으로 에러 전달
```

# 4. recover

## panic 붙잡기

```go
package main

import "fmt"

func safeDivide(a, b int) (result int, err error) {
	defer func() {
		if r := recover(); r != nil { // panic 값을 받음
			err = fmt.Errorf("복구됨: %v", r)
		}
	}()
	return a / b, nil
}

func main() {
	fmt.Println(safeDivide(10, 2))
	fmt.Println(safeDivide(1, 0))
	fmt.Println("프로그램 계속 실행")
	// 출력:
	// 5 <nil>
	// 0 복구됨: runtime error: integer divide by zero
	// 프로그램 계속 실행
}
```

```
recover 규칙:
- defer 된 함수 안에서 "직접" 호출해야만 동작
- panic 중이 아니면 nil 반환
- 복구되면 panic 이 일어난 함수는 정상 반환 (이름 있는 반환값으로 결과 설정 가능)
```

## 고루틴의 panic은 다른 고루틴에서 잡을 수 없다

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("main에서 복구?", r) // 절대 실행 안 됨
		}
	}()

	go func() {
		panic("고루틴에서 panic") // 프로그램 전체가 죽음
	}()

	time.Sleep(100 * time.Millisecond)
}
```

**각 고루틴은 자기 panic을 자기가 recover 해야 합니다.** 그래서 고루틴을 띄울 때 recover를 감싸는 헬퍼를 만들기도 합니다.

```go
package main

import (
	"fmt"
	"sync"
)

func safeGo(wg *sync.WaitGroup, fn func()) {
	wg.Go(func() { // Go 1.25+
		defer func() {
			if r := recover(); r != nil {
				fmt.Println("고루틴 panic 복구:", r)
			}
		}()
		fn()
	})
}

func main() {
	var wg sync.WaitGroup
	safeGo(&wg, func() { panic("앗") })
	safeGo(&wg, func() { fmt.Println("정상 작업") })
	wg.Wait()
}
```

## 실무 활용: HTTP 서버 미들웨어

`net/http` 서버는 핸들러에서 panic이 나도 **해당 요청만 끊고 서버는 계속 동작**하도록 내부에서 recover 합니다. 직접 응답 코드를 제어하고 싶으면 미들웨어를 만듭니다.

```go
package main

import (
	"fmt"
	"log/slog"
	"net/http"
	"net/http/httptest"
	"runtime/debug"
)

func recoverMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if rec := recover(); rec != nil {
				slog.Error("panic 발생", "err", rec, "path", r.URL.Path, "stack", string(debug.Stack()))
				http.Error(w, "Internal Server Error", http.StatusInternalServerError)
			}
		}()
		next.ServeHTTP(w, r)
	})
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/boom", func(w http.ResponseWriter, r *http.Request) {
		var m map[string]int
		m["x"] = 1 // panic
	})

	h := recoverMiddleware(mux)
	rec := httptest.NewRecorder()
	h.ServeHTTP(rec, httptest.NewRequest("GET", "/boom", nil))
	fmt.Println(rec.Code)
	// 출력: 500 (slog 에러 로그와 함께)
}
```

Gin, Echo 같은 프레임워크에는 `Recovery` 미들웨어가 기본으로 들어 있습니다.

# 5. os.Exit와 log.Fatal은 defer를 무시한다

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	defer fmt.Println("실행 안 됨!")
	os.Exit(1) // 즉시 종료, defer 무시
}
```

`log.Fatal`도 내부에서 `os.Exit(1)`을 호출합니다. 그래서 정리 작업이 필요한 프로그램은 이런 구조를 씁니다.

```go
func main() {
	if err := run(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}

func run() error {
	db, err := openDB()
	if err != nil {
		return err
	}
	defer db.Close() // run 이 반환될 때 확실히 실행됨

	// ... 실제 로직
	return nil
}
```

# 6. panic 값과 re-panic

```go
package main

import (
	"errors"
	"fmt"
)

var ErrCritical = errors.New("critical")

func handle() (err error) {
	defer func() {
		r := recover()
		if r == nil {
			return
		}
		// 내가 처리할 수 있는 panic 만 복구하고 나머지는 다시 panic
		if e, ok := r.(error); ok && errors.Is(e, ErrCritical) {
			err = e
			return
		}
		panic(r)
	}()
	panic(fmt.Errorf("wrap: %w", ErrCritical))
}

func main() {
	fmt.Println(handle())
	// 출력: wrap: critical
}
```

> Go 1.21부터 `panic(nil)`은 `*runtime.PanicNilError`로 바뀌어서, `recover()`가 nil을 반환해 panic을 놓치는 문제가 사라졌습니다.

## 정리

- `defer`는 함수 종료 시 실행을 예약, 리소스를 연 직후 바로 정리 코드를 defer
- defer는 LIFO 순서, 인자는 defer 시점에 평가, 이름 있는 반환값 수정 가능
- defer는 함수 단위라 루프 안에서 쓰면 쌓임 → 함수로 분리
- `panic`은 복구 불가능한 프로그래머 실수용, 일반 에러는 `error` 반환
- `recover`는 defer 함수 안에서만 동작, 고루틴마다 따로 recover 해야 함
- `os.Exit`/`log.Fatal`은 defer를 실행하지 않음 → `main`은 `run() error` 패턴으로

## 참고 링크

- Defer, Panic, and Recover: https://go.dev/blog/defer-panic-and-recover
- Effective Go - Defer: https://go.dev/doc/effective_go#defer
- Effective Go - Recover: https://go.dev/doc/effective_go#recover
- 언어 스펙 - Handling panics: https://go.dev/ref/spec#Handling_panics
- [한국어] Go by Example - Defer: https://mingrammer.com/gobyexample/defer/
- [한국어] Go by Example - Panic: https://mingrammer.com/gobyexample/panic/

이전 문서: [02. 에러 처리](./02-에러-처리.md) | 다음 문서: [04. 제네릭](./04-제네릭.md)
