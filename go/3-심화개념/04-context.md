# context - 취소, 타임아웃, 요청 범위 값 전달

## 들어가며

HTTP 요청 하나를 처리하는 과정을 생각해 봅시다.

```
사용자 요청
  └─ 핸들러 고루틴
       ├─ DB 쿼리 (100ms)
       ├─ 외부 API 호출 (2s)
       └─ 캐시 조회 고루틴
```

그런데 사용자가 **중간에 브라우저를 닫았다면?** 아직 진행 중인 DB 쿼리, API 호출, 고루틴은 전부 헛일입니다. 또 외부 API가 30초 동안 응답이 없으면 계속 기다려야 할까요?

```
필요한 것:
1. 취소 신호를 하위 작업 전체에 전파하기
2. 타임아웃/데드라인 걸기
3. 요청 ID 같은 값을 호출 체인 전체에 전달하기
```

이걸 표준화한 것이 `context.Context`입니다. JS의 `AbortController` / `AbortSignal`과 비슷한 역할입니다.

```ts
// JS
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();
```

```go
// Go
ctx, cancel := context.WithCancel(context.Background())
req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
cancel()
```

# 1. Context 인터페이스

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool) // 데드라인이 있으면 반환
	Done() <-chan struct{}                   // 취소되면 닫히는 채널
	Err() error                              // 취소 이유: Canceled 또는 DeadlineExceeded
	Value(key any) any                       // 요청 범위 값
}
```

핵심은 `Done()`입니다. **취소되면 이 채널이 닫히고**, 닫힌 채널은 모든 수신자가 즉시 받을 수 있으므로 여러 고루틴에 동시에 신호가 전달됩니다.

## 루트 컨텍스트

```go
ctx := context.Background() // main, 초기화, 테스트의 최상위
ctx := context.TODO()       // 아직 어떤 컨텍스트를 쓸지 정하지 못했을 때 (임시 표시)
```

# 2. WithCancel: 수동 취소

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "종료:", ctx.Err())
			return
		default:
			// 실제 작업 한 단위
			time.Sleep(10 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go worker(ctx, "A")
	go worker(ctx, "B")

	time.Sleep(30 * time.Millisecond)
	cancel() // A, B 모두에게 취소 신호
	time.Sleep(20 * time.Millisecond)
	// 출력 (순서는 다를 수 있음):
	// A 종료: context canceled
	// B 종료: context canceled
}
```

```
✅ cancel 은 여러 번 호출해도 안전
✅ 작업이 정상적으로 끝나도 반드시 cancel 호출 → 안 하면 내부 리소스 누수
   → 관례: ctx, cancel := context.WithXxx(...) 바로 다음 줄에 defer cancel()
   (go vet 의 lostcancel 검사가 잡아줌)
```

# 3. WithTimeout / WithDeadline

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"time"
)

func slowQuery(ctx context.Context) (string, error) {
	select {
	case <-time.After(200 * time.Millisecond): // 200ms 걸리는 작업
		return "결과", nil
	case <-ctx.Done():
		return "", ctx.Err()
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
	defer cancel()

	res, err := slowQuery(ctx)
	if errors.Is(err, context.DeadlineExceeded) {
		fmt.Println("타임아웃:", err)
		return
	}
	fmt.Println(res)
	// 출력: 타임아웃: context deadline exceeded
}
```

```
WithTimeout(parent, 3*time.Second)       → 지금부터 3초 후
WithDeadline(parent, time.Date(...))     → 특정 시각까지
```

## 부모-자식 관계: 더 짧은 쪽이 이긴다

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	parent, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	// 자식이 10초를 요청해도 부모의 1초를 넘을 수 없음
	child, cancel2 := context.WithTimeout(parent, 10*time.Second)
	defer cancel2()

	d, _ := child.Deadline()
	fmt.Println(time.Until(d) <= time.Second)
	// 출력: true

	// 부모가 취소되면 자식도 모두 취소됨 (반대는 아님)
	p, pc := context.WithCancel(context.Background())
	c, cc := context.WithCancel(p)
	defer cc()
	pc()
	<-c.Done()
	fmt.Println("자식도 취소됨:", c.Err())
	// 출력: 자식도 취소됨: context canceled
}
```

```
Background
  └─ WithTimeout(5s)            ← HTTP 요청 전체
       ├─ WithTimeout(1s)       ← DB 쿼리
       └─ WithCancel            ← 병렬 작업들
            ├─ goroutine A
            └─ goroutine B

위쪽이 취소되면 → 아래쪽 전부 취소
```

# 4. 취소 원인 남기기 (Go 1.20+)

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"time"
)

var ErrUserQuit = errors.New("사용자가 나감")

func main() {
	ctx, cancel := context.WithCancelCause(context.Background())
	cancel(ErrUserQuit)

	fmt.Println(ctx.Err())          // context canceled
	fmt.Println(context.Cause(ctx)) // 사용자가 나감

	// 타임아웃에도 원인 지정 가능 (Go 1.21+)
	ctx2, cancel2 := context.WithTimeoutCause(context.Background(), time.Millisecond, errors.New("결제 API 3초 초과"))
	defer cancel2()
	<-ctx2.Done()
	fmt.Println(context.Cause(ctx2)) // 결제 API 3초 초과
}
```

로그에 "context canceled"만 찍혀서 원인을 알 수 없던 문제를 해결합니다.

# 5. WithValue: 요청 범위 값

```go
package main

import (
	"context"
	"fmt"
)

// 키 충돌을 막기 위해 비공개 타입을 키로 사용
type ctxKey int

const (
	requestIDKey ctxKey = iota
	userKey
)

func WithRequestID(ctx context.Context, id string) context.Context {
	return context.WithValue(ctx, requestIDKey, id)
}

func RequestID(ctx context.Context) string {
	id, _ := ctx.Value(requestIDKey).(string) // 없으면 ""
	return id
}

func handle(ctx context.Context) {
	fmt.Println("request_id =", RequestID(ctx))
}

func main() {
	ctx := WithRequestID(context.Background(), "req-123")
	handle(ctx)
	// 출력: request_id = req-123
}
```

```
WithValue 에 넣어도 되는 것:
✅ 요청 ID, 트레이스 ID, 인증된 사용자 정보, 로케일
   → "요청이 끝나면 의미가 없어지는" 요청 범위 메타데이터

넣으면 안 되는 것:
❌ DB 커넥션, 로거, 설정 → 구조체 필드나 함수 인자로
❌ 함수의 필수 파라미터 → 명시적 인자로 (숨겨진 의존성이 됨)

키:
❌ context.WithValue(ctx, "userID", 1)  // 문자열 키는 다른 패키지와 충돌 가능 (staticcheck SA1029 경고)
✅ 비공개 타입(type ctxKey int)을 키로 사용
```

# 6. HTTP에서 context

## 서버: 클라이언트 연결이 끊기면 자동 취소

```go
package main

import (
	"context"
	"fmt"
	"log/slog"
	"net/http"
	"net/http/httptest"
	"time"
)

func slowHandler(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context() // 클라이언트가 연결을 끊으면 취소됨

	select {
	case <-time.After(2 * time.Second): // 무거운 작업
		fmt.Fprintln(w, "완료")
	case <-ctx.Done():
		slog.Warn("요청 취소됨", "err", ctx.Err())
		return // 작업 중단
	}
}

func main() {
	srv := httptest.NewServer(http.HandlerFunc(slowHandler))
	defer srv.Close()

	// 클라이언트가 100ms 만에 포기
	ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
	defer cancel()
	req, _ := http.NewRequestWithContext(ctx, "GET", srv.URL, nil)
	_, err := http.DefaultClient.Do(req)
	fmt.Println("클라이언트 에러:", err != nil)

	time.Sleep(50 * time.Millisecond) // 서버 쪽 로그 출력 대기
	// 출력:
	// 클라이언트 에러: true
	// ... WARN 요청 취소됨 err="context canceled"
}
```

## 미들웨어에서 값 주입

```go
func requestIDMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		id := r.Header.Get("X-Request-ID")
		if id == "" {
			id = uuid.New().String() // Go 1.27+ 표준 uuid 패키지
		}
		ctx := WithRequestID(r.Context(), id)
		next.ServeHTTP(w, r.WithContext(ctx)) // 새 컨텍스트를 담은 요청 전달
	})
}
```

## 핸들러별 타임아웃

```go
func handler(w http.ResponseWriter, r *http.Request) {
	ctx, cancel := context.WithTimeout(r.Context(), 3*time.Second)
	defer cancel()

	user, err := db.GetUser(ctx, id) // database/sql 의 QueryContext 등은 ctx 취소 시 쿼리를 중단
	if errors.Is(err, context.DeadlineExceeded) {
		http.Error(w, "timeout", http.StatusGatewayTimeout)
		return
	}
	// ...
}
```

# 7. Graceful Shutdown

`Ctrl+C`(SIGINT)나 쿠버네티스의 SIGTERM을 받으면 **진행 중인 요청은 마무리하고** 종료하는 패턴입니다.

```go
package main

import (
	"context"
	"errors"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	// SIGINT, SIGTERM 을 받으면 취소되는 컨텍스트
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(2 * time.Second) // 오래 걸리는 요청
		w.Write([]byte("done\n"))
	})

	srv := &http.Server{Addr: ":8080", Handler: mux}

	go func() {
		slog.Info("서버 시작", "addr", srv.Addr)
		if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			slog.Error("서버 에러", "err", err)
			os.Exit(1)
		}
	}()

	<-ctx.Done() // 시그널 대기
	slog.Info("종료 신호 받음, 진행 중인 요청 마무리 중...")

	// 최대 10초 동안 기존 요청 처리를 기다림
	shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	if err := srv.Shutdown(shutdownCtx); err != nil {
		slog.Error("강제 종료", "err", err)
	}
	slog.Info("서버 종료 완료")
}
```

```
Shutdown 동작:
1. 리스너를 닫아 새 연결을 받지 않음
2. 유휴 연결을 닫음
3. 진행 중인 요청이 끝날 때까지 대기 (shutdownCtx 타임아웃까지)
4. ListenAndServe 는 http.ErrServerClosed 를 반환
```

# 8. context.AfterFunc 와 WithoutCancel (Go 1.21+)

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	// 취소되면 콜백 실행 (별도 고루틴에서)
	stop := context.AfterFunc(ctx, func() {
		fmt.Println("취소 감지 → 리소스 정리")
	})
	defer stop()

	// 부모의 값은 유지하되 취소는 전파받지 않는 컨텍스트
	// 예: 요청이 끝나도 계속되어야 하는 감사 로그 전송
	detached := context.WithoutCancel(ctx)

	cancel()
	time.Sleep(10 * time.Millisecond)
	fmt.Println("원본:", ctx.Err(), "/ detached:", detached.Err())
	// 출력:
	// 취소 감지 → 리소스 정리
	// 원본: context canceled / detached: <nil>
}
```

# 9. context 사용 규칙

```
✅ 첫 번째 파라미터로, 이름은 ctx
   func GetUser(ctx context.Context, id int) (*User, error)

✅ 구조체 필드에 저장하지 않기 → 호출마다 인자로 전달
   (예외: http.Request 처럼 요청 자체를 나타내는 타입)

✅ nil 컨텍스트 전달 금지 → 모르겠으면 context.TODO()

✅ WithCancel/WithTimeout 을 만들었으면 defer cancel()

✅ 오래 걸리는 작업/블로킹 작업은 ctx.Done() 을 확인
   (select 에 case <-ctx.Done() 추가, 또는 ctx 를 받는 API 사용)

✅ 라이브러리 함수가 ctx 를 받는다면 그대로 넘겨주기
   db.QueryContext(ctx, ...), http.NewRequestWithContext(ctx, ...),
   redis.Get(ctx, ...), grpc 클라이언트 호출 등

❌ context.Background() 를 함수 중간에서 새로 만들어 쓰기
   → 상위의 취소/타임아웃이 끊김
```

## 정리

- `context.Context`는 취소 신호, 데드라인, 요청 범위 값을 호출 체인 전체에 전달
- `WithCancel`/`WithTimeout`/`WithDeadline`로 파생, 만들면 반드시 `defer cancel()`
- 부모가 취소되면 자식 전부 취소, 데드라인은 더 짧은 쪽이 적용
- 작업 쪽에서는 `select { case <-ctx.Done(): return ctx.Err() }`로 협조적 취소
- `WithValue`는 요청 ID 같은 메타데이터만, 키는 비공개 타입
- HTTP 서버는 `r.Context()`, 클라이언트는 `NewRequestWithContext`
- `signal.NotifyContext` + `srv.Shutdown(ctx)`로 graceful shutdown
- Go 1.20+ `WithCancelCause`, 1.21+ `AfterFunc`, `WithoutCancel`, `WithTimeoutCause`

## 참고 링크

- context 패키지: https://pkg.go.dev/context
- Go Concurrency Patterns: Context: https://go.dev/blog/context
- Contexts and structs: https://go.dev/blog/context-and-structs
- os/signal 패키지: https://pkg.go.dev/os/signal
- Go by Example - Context: https://gobyexample.com/context
- [한국어] Tucker의 Go 언어 프로그래밍 유튜브 강의 (context 챕터 포함): https://www.youtube.com/playlist?list=PLy-g2fnSzUTBHwuXkWQ834QHDZwLx6v6j

이전 문서: [03. sync 패키지](./03-sync-패키지.md) | 다음 문서: [05. 동시성 패턴](./05-동시성-패턴.md)
