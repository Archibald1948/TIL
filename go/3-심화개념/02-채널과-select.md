# 채널과 select - 고루틴끼리 안전하게 대화하는 파이프

## 들어가며

고루틴을 여러 개 띄웠다면 이제 서로 데이터를 주고받아야 합니다.

```
공유 메모리 방식: 변수 하나를 여러 고루틴이 락을 잡고 읽고 씀
채널 방식:       값을 "보내고 받는" 파이프로 소유권을 넘김
```

채널은 Go 동시성의 핵심 도구이며, `select`와 함께 쓰면 **타임아웃, 취소, 여러 이벤트 대기**를 간결하게 표현할 수 있습니다.

```
JS 비유:
- 채널 ≈ 비동기 큐 (한쪽이 push, 한쪽이 await pop)
- select ≈ Promise.race (먼저 준비된 것 하나를 처리)
```

# 1. 채널 기본

## 생성, 보내기, 받기

```go
package main

import "fmt"

func main() {
	ch := make(chan string) // string 을 주고받는 채널

	go func() {
		ch <- "안녕" // 보내기 (받는 쪽이 준비될 때까지 블록)
	}()

	msg := <-ch // 받기 (보내는 쪽이 올 때까지 블록)
	fmt.Println(msg)
	// 출력: 안녕
}
```

```
ch <- v      → 채널에 v 보내기
v := <-ch    → 채널에서 받기
<-ch         → 받고 값은 버리기 (신호 용도)
```

## 버퍼 없는 채널 = 동기화 지점

버퍼 없는 채널(`make(chan T)`)은 **보내는 쪽과 받는 쪽이 만나야** 전달이 끝납니다. 그래서 두 고루틴의 실행 시점을 맞추는 용도로도 씁니다.

```go
package main

import "fmt"

func main() {
	done := make(chan struct{}) // 데이터 없이 신호만 (0바이트)

	go func() {
		fmt.Println("작업 중...")
		done <- struct{}{} // 끝났다고 알림
	}()

	<-done // 작업이 끝날 때까지 대기
	fmt.Println("작업 완료 확인")
}
```

## 데드락

```go
package main

func main() {
	ch := make(chan int)
	ch <- 1 // 받을 고루틴이 없음 → 영원히 블록
}
```

```
fatal error: all goroutines are asleep - deadlock!
```

**모든 고루틴이 블록되면** 런타임이 데드락을 감지하고 종료합니다. (일부 고루틴만 블록되면 감지하지 못하고 누수가 됩니다.)

# 2. 버퍼 있는 채널

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 3) // 버퍼 크기 3

	ch <- 1 // 버퍼에 여유가 있으면 블록 안 됨
	ch <- 2
	ch <- 3
	// ch <- 4 // 버퍼가 꽉 차면 블록 (여기서는 데드락)

	fmt.Println(len(ch), cap(ch))
	// 출력: 3 3

	fmt.Println(<-ch, <-ch, <-ch) // FIFO
	// 출력: 1 2 3
}
```

```
make(chan T)      → 버퍼 없음: 보내기/받기가 만나야 진행 (강한 동기화)
make(chan T, n)   → 버퍼 n: 버퍼가 차기 전까지 보내기가 블록 안 됨 (느슨한 결합)

버퍼 크기 선택:
- 대부분 0 또는 1
- "성능을 위해 크게" 는 문제를 숨길 뿐인 경우가 많음
- 정확한 개수를 알 때 (결과 N개 수집) 는 N
```

# 3. 채널 닫기와 range

## close와 comma ok

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 10
	close(ch) // 더 이상 보낼 값이 없다고 알림

	v, ok := <-ch
	fmt.Println(v, ok) // 버퍼에 남은 값은 받을 수 있음
	// 출력: 10 true

	v, ok = <-ch
	fmt.Println(v, ok) // 닫히고 비었으면 제로값 + false
	// 출력: 0 false
}
```

## range로 닫힐 때까지 받기

```go
package main

import "fmt"

func generate(n int) <-chan int {
	ch := make(chan int)
	go func() {
		defer close(ch) // 보내는 쪽이 다 보내고 닫음
		for i := range n {
			ch <- i * i
		}
	}()
	return ch
}

func main() {
	for v := range generate(5) { // 채널이 닫히면 루프 종료
		fmt.Print(v, " ")
	}
	fmt.Println()
	// 출력: 0 1 4 9 16
}
```

## 채널 규칙 정리

| 동작 | nil 채널 | 열린 채널 | 닫힌 채널 |
|------|---------|----------|----------|
| 보내기 `ch <- v` | 영원히 블록 | 블록 or 성공 | **panic** |
| 받기 `<-ch` | 영원히 블록 | 블록 or 성공 | 남은 값, 없으면 제로값 즉시 반환 |
| 닫기 `close(ch)` | **panic** | 성공 | **panic** |

```
✅ 채널은 "보내는 쪽"이 닫는다 (받는 쪽은 언제 끝날지 모름)
✅ 보내는 고루틴이 여러 개면 → 전부 끝난 뒤 WaitGroup 으로 기다렸다가 한 번 닫기
✅ 모든 채널을 꼭 닫을 필요는 없음 (GC 가 회수). range 로 받는 쪽이 끝을 알아야 할 때만 닫기
```

# 4. 방향 있는 채널

함수 시그니처에서 채널의 용도를 제한하면 실수를 컴파일 타임에 잡을 수 있습니다.

```go
package main

import "fmt"

// chan<- int : 보내기 전용
func producer(out chan<- int) {
	for i := range 3 {
		out <- i
	}
	close(out)
	// <-out // 컴파일 에러: 보내기 전용 채널에서 받을 수 없음
}

// <-chan int : 받기 전용
func consumer(in <-chan int) {
	for v := range in {
		fmt.Print(v, " ")
	}
	fmt.Println()
	// close(in) // 컴파일 에러: 받기 전용 채널은 닫을 수 없음
}

func main() {
	ch := make(chan int) // 양방향 채널은 단방향으로 자동 변환됨
	go producer(ch)
	consumer(ch)
	// 출력: 0 1 2
}
```

# 5. select

## 여러 채널 중 준비된 것 처리

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fast := make(chan string)
	slow := make(chan string)

	go func() {
		time.Sleep(10 * time.Millisecond)
		fast <- "빠른 응답"
	}()
	go func() {
		time.Sleep(50 * time.Millisecond)
		slow <- "느린 응답"
	}()

	for range 2 {
		select { // 준비된 case 하나를 실행 (여러 개 준비되면 무작위)
		case msg := <-fast:
			fmt.Println(msg)
		case msg := <-slow:
			fmt.Println(msg)
		}
	}
	// 출력:
	// 빠른 응답
	// 느린 응답
}
```

## 타임아웃

```go
package main

import (
	"fmt"
	"time"
)

func callAPI() <-chan string {
	ch := make(chan string, 1) // 버퍼 1: 타임아웃으로 아무도 안 받아도 고루틴이 끝날 수 있게
	go func() {
		time.Sleep(200 * time.Millisecond)
		ch <- "결과"
	}()
	return ch
}

func main() {
	select {
	case res := <-callAPI():
		fmt.Println(res)
	case <-time.After(50 * time.Millisecond):
		fmt.Println("타임아웃!")
	}
	// 출력: 타임아웃!
}
```

실무에서는 `time.After`보다 **context의 타임아웃**을 더 많이 씁니다 (context 문서).

## default: 논블로킹

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 1)

	// 논블로킹 받기
	select {
	case v := <-ch:
		fmt.Println("받음", v)
	default:
		fmt.Println("받을 값 없음")
	}

	// 논블로킹 보내기 (버퍼가 꽉 찼으면 버림)
	for i := range 3 {
		select {
		case ch <- i:
			fmt.Println("보냄", i)
		default:
			fmt.Println("버퍼 가득 참, 버림", i)
		}
	}
	// 출력:
	// 받을 값 없음
	// 보냄 0
	// 버퍼 가득 참, 버림 1
	// 버퍼 가득 참, 버림 2
}
```

## for-select 루프와 종료 신호

```go
package main

import "fmt"

func worker(jobs <-chan int, quit <-chan struct{}) {
	for {
		select {
		case j := <-jobs:
			fmt.Println("처리:", j)
		case <-quit:
			fmt.Println("종료 신호 받음")
			return // return 을 잊으면 무한 루프
		}
	}
}

func main() {
	jobs := make(chan int)
	quit := make(chan struct{})
	done := make(chan struct{})

	go func() {
		worker(jobs, quit)
		close(done)
	}()

	jobs <- 1
	jobs <- 2
	close(quit) // 닫힌 채널은 모든 수신자가 즉시 받음 → 브로드캐스트 효과
	<-done
	// 출력:
	// 처리: 1
	// 처리: 2
	// 종료 신호 받음
}
```

> **채널을 닫는 것 = 모든 수신자에게 동시에 알리는 브로드캐스트**입니다. `context.Done()`도 이 원리로 동작합니다.

## nil 채널로 case 비활성화

```go
package main

import "fmt"

func merge(a, b <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for a != nil || b != nil {
			select {
			case v, ok := <-a:
				if !ok {
					a = nil // nil 채널은 영원히 블록 → 이 case 는 다시 선택되지 않음
					continue
				}
				out <- v
			case v, ok := <-b:
				if !ok {
					b = nil
					continue
				}
				out <- v
			}
		}
	}()
	return out
}

func gen(vals ...int) <-chan int {
	ch := make(chan int)
	go func() {
		defer close(ch)
		for _, v := range vals {
			ch <- v
		}
	}()
	return ch
}

func main() {
	sum := 0
	for v := range merge(gen(1, 2, 3), gen(10, 20)) {
		sum += v
	}
	fmt.Println(sum)
	// 출력: 36
}
```

# 6. 티커와 함께 쓰기

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ticker := time.NewTicker(20 * time.Millisecond)
	defer ticker.Stop()

	deadline := time.After(70 * time.Millisecond)
	count := 0

	for {
		select {
		case <-ticker.C:
			count++
			fmt.Println("tick", count)
		case <-deadline:
			fmt.Println("끝")
			return
		}
	}
	// 출력:
	// tick 1
	// tick 2
	// tick 3
	// 끝
}
```

주기적인 헬스체크, 캐시 갱신, 메트릭 전송 같은 백그라운드 작업에 이 구조를 씁니다.

# 7. 채널로 세마포어 만들기

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

func main() {
	sem := make(chan struct{}, 3) // 동시에 최대 3개
	var running, maxRunning atomic.Int32
	var wg sync.WaitGroup

	for range 10 {
		wg.Go(func() {
			sem <- struct{}{}        // 자리 확보 (꽉 차면 대기)
			defer func() { <-sem }() // 자리 반납

			n := running.Add(1)
			for {
				m := maxRunning.Load()
				if n <= m || maxRunning.CompareAndSwap(m, n) {
					break
				}
			}
			time.Sleep(10 * time.Millisecond)
			running.Add(-1)
		})
	}
	wg.Wait()
	fmt.Println("최대 동시 실행:", maxRunning.Load())
	// 출력: 최대 동시 실행: 3
}
```

외부 API 호출 수 제한, DB 커넥션 수 제한 등에 쓰입니다. (`golang.org/x/sync/semaphore`, `errgroup.SetLimit`도 있음)

# 8. 채널 vs Mutex, 언제 무엇을?

```
채널이 어울리는 경우:
✅ 데이터의 소유권을 다른 고루틴에게 넘길 때 (파이프라인)
✅ 작업을 분배할 때 (worker pool)
✅ 비동기 결과를 전달할 때
✅ 이벤트/신호를 알릴 때 (종료, 취소)

Mutex 가 어울리는 경우:
✅ 구조체 내부 상태(캐시, 카운터, 맵)를 보호할 때
✅ 짧은 임계 구역
✅ 성능이 중요한 핫 경로
```

```
흔한 실수:
❌ 단순 카운터를 채널로 구현 → 코드가 복잡하고 느림
❌ 채널을 여러 곳에서 close → panic
❌ 받는 쪽 없이 보내는 고루틴 → 누수
❌ for-select 에서 종료 case 누락 → 누수
```

## 정리

- `make(chan T)`는 버퍼 없는 채널로 보내기/받기가 만나야 진행, `make(chan T, n)`은 버퍼 있음
- 채널은 보내는 쪽이 `close`, 받는 쪽은 `range` 또는 `v, ok := <-ch`
- 닫힌 채널에 보내거나 두 번 닫으면 panic, nil 채널은 영원히 블록
- `chan<- T`(보내기 전용), `<-chan T`(받기 전용)으로 용도 제한
- `select`로 여러 채널 대기, `time.After`로 타임아웃, `default`로 논블로킹
- 채널 close는 브로드캐스트 신호, 버퍼 채널은 세마포어로 활용 가능
- 상태 보호는 Mutex, 소유권 이동/작업 분배/신호는 채널

## 참고 링크

- A Tour of Go (Channels): https://go.dev/tour/concurrency/2
- Effective Go - Channels: https://go.dev/doc/effective_go#channels
- Go Concurrency Patterns: Pipelines: https://go.dev/blog/pipelines
- 언어 스펙 - Select statements: https://go.dev/ref/spec#Select_statements
- [한국어] Go by Example - Channels: https://mingrammer.com/gobyexample/channels/
- [한국어] Go by Example - Select: https://mingrammer.com/gobyexample/select/

이전 문서: [01. 고루틴](./01-고루틴.md) | 다음 문서: [03. sync 패키지](./03-sync-패키지.md)
