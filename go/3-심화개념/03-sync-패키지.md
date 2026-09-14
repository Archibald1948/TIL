# sync 패키지 - Mutex, Once, Pool, 그리고 atomic

## 들어가며

채널이 Go 동시성의 상징이지만, 실무 코드를 열어보면 `sync.Mutex`도 정말 많이 쓰입니다.

```
캐시, 세션 저장소, 커넥션 목록, 통계 카운터
→ "여러 고루틴이 같은 상태를 읽고 쓴다"
→ 채널로 짜면 복잡해지고, Mutex 로 짜면 명확해지는 경우가 많음
```

이 문서에서 다루는 도구들입니다.

```
sync.Mutex / RWMutex   → 상호 배제 락
sync.WaitGroup         → 고루틴 완료 대기 (고루틴 문서 참고)
sync.Once / OnceValue  → 딱 한 번만 실행
sync.Map               → 동시성 안전 맵 (특수 용도)
sync.Pool              → 임시 객체 재사용
sync.Cond              → 조건 변수 (드묾)
sync/atomic            → 락 없는 원자적 연산
```

# 1. sync.Mutex

## 기본 사용

```go
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	mu     sync.Mutex // 보호할 필드 바로 위에 두는 것이 관례
	counts map[string]int
}

func NewCounter() *Counter {
	return &Counter{counts: make(map[string]int)}
}

func (c *Counter) Inc(key string) {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.counts[key]++
}

func (c *Counter) Get(key string) int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.counts[key]
}

func main() {
	c := NewCounter()
	var wg sync.WaitGroup
	for range 1000 {
		wg.Go(func() { c.Inc("visits") })
	}
	wg.Wait()
	fmt.Println(c.Get("visits"))
	// 출력: 1000
}
```

```
규칙:
✅ Lock 직후 defer Unlock → 중간에 return/panic 해도 안전
✅ Mutex 는 제로값으로 바로 사용 가능 (초기화 불필요)
✅ Mutex 를 포함한 구조체는 포인터로 전달 (복사 금지)
❌ Go 의 Mutex 는 재진입(reentrant) 불가 → 같은 고루틴에서 두 번 Lock 하면 데드락
```

## 복사하면 안 된다

```go
package main

import "sync"

type SafeData struct {
	mu   sync.Mutex
	data int
}

func (s SafeData) Read() int { // 값 리시버 → Mutex 가 복사됨
	s.mu.Lock()
	defer s.mu.Unlock()
	return s.data
}

func main() {
	s := SafeData{}
	s.Read()
}
```

```bash
go vet .
# ./main.go:10:9: Read passes lock by value: main.SafeData contains sync.Mutex
```

`go vet`의 `copylocks` 검사가 잡아줍니다. **Mutex를 가진 타입의 메서드는 포인터 리시버**로 만듭니다.

## 락 범위는 최소한으로

```go
// ❌ 느린 작업(네트워크, 디스크)을 락 안에서 수행
func (c *Cache) Refresh(key string) {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.data[key] = fetchFromAPI(key) // 수백 ms 동안 다른 모든 고루틴이 대기
}

// ✅ 느린 작업은 락 밖에서
func (c *Cache) Refresh(key string) {
	val := fetchFromAPI(key)

	c.mu.Lock()
	c.data[key] = val
	c.mu.Unlock()
}
```

## 데드락 주의: 락 순서

```
고루틴 A: muX.Lock() → muY.Lock()
고루틴 B: muY.Lock() → muX.Lock()
→ A 는 Y 를, B 는 X 를 기다리며 영원히 멈춤

해결: 여러 락을 잡을 때는 항상 같은 순서로
```

# 2. sync.RWMutex

읽기가 쓰기보다 **훨씬 많을 때** 씁니다.

```go
package main

import (
	"fmt"
	"sync"
)

type Config struct {
	mu     sync.RWMutex
	values map[string]string
}

func (c *Config) Get(key string) (string, bool) {
	c.mu.RLock() // 읽기 락: 여러 고루틴이 동시에 획득 가능
	defer c.mu.RUnlock()
	v, ok := c.values[key]
	return v, ok
}

func (c *Config) Set(key, value string) {
	c.mu.Lock() // 쓰기 락: 독점 (읽기 락도 모두 대기)
	defer c.mu.Unlock()
	c.values[key] = value
}

func main() {
	cfg := &Config{values: map[string]string{}}
	cfg.Set("env", "prod")

	var wg sync.WaitGroup
	for range 100 {
		wg.Go(func() { cfg.Get("env") })
	}
	wg.Wait()

	v, _ := cfg.Get("env")
	fmt.Println(v)
	// 출력: prod
}
```

```
RLock ↔ RLock  : 동시에 가능
RLock ↔ Lock   : 배타적
Lock  ↔ Lock   : 배타적
```

> RWMutex는 Mutex보다 내부 비용이 커서, 읽기 비율이 압도적이지 않거나 임계 구역이 매우 짧으면 오히려 느릴 수 있습니다. 확신이 없으면 Mutex로 시작하고 벤치마크로 판단합니다.

# 3. sync.Once

## 딱 한 번만 초기화

```go
package main

import (
	"fmt"
	"sync"
)

type DB struct{ name string }

var (
	instance *DB
	once     sync.Once
)

func GetDB() *DB {
	once.Do(func() { // 여러 고루틴이 동시에 호출해도 딱 한 번만 실행
		fmt.Println("DB 연결 생성")
		instance = &DB{name: "main"}
	})
	return instance
}

func main() {
	var wg sync.WaitGroup
	for range 10 {
		wg.Go(func() { GetDB() })
	}
	wg.Wait()
	fmt.Println(GetDB().name)
	// 출력:
	// DB 연결 생성
	// main
}
```

지연 초기화(lazy initialization), 싱글턴에 씁니다.

## OnceFunc / OnceValue / OnceValues (Go 1.21+)

```go
package main

import (
	"fmt"
	"os"
	"sync"
)

// 결과값을 캐싱하는 함수로 감싸기
var loadConfig = sync.OnceValues(func() (map[string]string, error) {
	fmt.Println("설정 파일 읽는 중...")
	_, err := os.ReadFile("/없는/config.json")
	if err != nil {
		return nil, err
	}
	return map[string]string{"env": "prod"}, nil
})

var hostname = sync.OnceValue(func() string {
	fmt.Println("hostname 조회")
	h, _ := os.Hostname()
	return h
})

func main() {
	for range 3 {
		_, err := loadConfig() // 첫 호출에서만 실행, 이후엔 같은 결과 반환
		fmt.Println("err != nil:", err != nil)
	}
	_ = hostname()
	_ = hostname()
	// 출력:
	// 설정 파일 읽는 중...
	// err != nil: true
	// err != nil: true
	// err != nil: true
	// hostname 조회
}
```

> 첫 실행에서 에러가 나도 **다시 시도하지 않고 같은 에러를 계속 반환**합니다. 재시도가 필요하면 직접 Mutex로 구현해야 합니다.

# 4. sync/atomic

## 원자적 타입 (Go 1.19+)

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

type Stats struct {
	requests atomic.Int64
	errors   atomic.Int64
	healthy  atomic.Bool
}

type Config struct {
	Version int
}

func main() {
	var s Stats
	s.healthy.Store(true)

	var wg sync.WaitGroup
	for i := range 1000 {
		wg.Go(func() {
			s.requests.Add(1)
			if i%10 == 0 {
				s.errors.Add(1)
			}
		})
	}
	wg.Wait()
	fmt.Println(s.requests.Load(), s.errors.Load(), s.healthy.Load())
	// 출력: 1000 100 true

	// CompareAndSwap: 현재 값이 old 일 때만 new 로 바꿈
	var state atomic.Int32
	fmt.Println(state.CompareAndSwap(0, 1)) // true
	fmt.Println(state.CompareAndSwap(0, 2)) // false (이미 1)

	// atomic.Pointer: 설정 전체를 원자적으로 교체 (읽기가 매우 많을 때 유용)
	var cfg atomic.Pointer[Config]
	cfg.Store(&Config{Version: 1})
	cfg.Store(&Config{Version: 2}) // 핫 리로드
	fmt.Println(cfg.Load().Version)
	// 출력: 2
}
```

```
atomic.Int32, Int64, Uint32, Uint64, Uintptr  → Add, Load, Store, Swap, CompareAndSwap
atomic.Bool
atomic.Pointer[T]                              → 타입 안전 포인터
atomic.Value                                   → 아무 타입 (Pointer[T] 권장)
```

```
atomic vs Mutex:
atomic → 단일 값(카운터, 플래그, 포인터 교체)에 적합, 가장 빠름
Mutex  → 여러 필드를 함께 일관성 있게 바꿔야 할 때 (atomic 여러 개로는 불가능)
```

> 옛날 코드의 `atomic.AddInt64(&n, 1)` 함수형 API보다 **타입 기반 API**가 실수(정렬 문제, 일반 접근과 혼용)를 줄여줍니다. Go 1.27의 `go fix`에는 이를 자동 변환하는 `atomictypes` modernizer가 있습니다.

# 5. sync.Map

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var m sync.Map // 제로값으로 사용, 타입은 any

	m.Store("a", 1)
	m.Store("b", 2)

	v, ok := m.Load("a")
	fmt.Println(v, ok) // 1 true

	actual, loaded := m.LoadOrStore("a", 100) // 있으면 기존 값
	fmt.Println(actual, loaded)               // 1 true

	m.Delete("b")

	m.Range(func(key, value any) bool {
		fmt.Println(key, value)
		return true // false 반환 시 순회 중단
	})
}
```

```
sync.Map 이 유리한 경우 (공식 문서 기준):
1. 키가 한 번 쓰이고 이후엔 거의 읽기만 하는 경우 (캐시가 계속 커지기만 할 때)
2. 여러 고루틴이 서로 겹치지 않는 키 집합을 다룰 때

그 외 대부분: map + Mutex(RWMutex) 가 더 명확하고 타입 안전함
```

> 값 타입이 `any`라서 매번 타입 단언이 필요합니다. 제네릭으로 감싼 래퍼를 만들어 쓰는 경우도 많습니다.

# 6. sync.Pool

## 임시 객체 재사용으로 GC 부담 줄이기

```go
package main

import (
	"bytes"
	"fmt"
	"sync"
)

var bufPool = sync.Pool{
	New: func() any { // 풀이 비었을 때 생성
		return new(bytes.Buffer)
	},
}

func render(name string) string {
	buf := bufPool.Get().(*bytes.Buffer)
	defer func() {
		buf.Reset() // 반드시 초기화 후 반납
		bufPool.Put(buf)
	}()

	buf.WriteString("<h1>")
	buf.WriteString(name)
	buf.WriteString("</h1>")
	return buf.String()
}

func main() {
	fmt.Println(render("gopher"))
	fmt.Println(render("rust"))
	// 출력:
	// <h1>gopher</h1>
	// <h1>rust</h1>
}
```

```
특징:
- GC 가 돌 때 풀의 객체가 언제든 사라질 수 있음 → 캐시로 쓰면 안 됨
- 짧게 쓰고 버리는 객체를 "초당 수만 번" 할당하는 핫 경로에서만 효과
- fmt, encoding/json 등 표준 라이브러리 내부에서도 사용

주의:
❌ Reset 없이 반납 → 이전 데이터가 섞임
❌ 크기가 들쭉날쭉한 버퍼(가끔 10MB) 반납 → 메모리가 계속 붙잡힘
❌ 프로파일링 없이 미리 최적화
```

# 7. sync.Cond

조건이 만족될 때까지 고루틴을 재우고 깨우는 저수준 도구입니다. **대부분 채널로 더 간단하게 표현**할 수 있어서 실무에서는 드뭅니다.

```go
package main

import (
	"fmt"
	"sync"
)

type Queue struct {
	mu    sync.Mutex
	cond  *sync.Cond
	items []int
}

func NewQueue() *Queue {
	q := &Queue{}
	q.cond = sync.NewCond(&q.mu)
	return q
}

func (q *Queue) Put(v int) {
	q.mu.Lock()
	q.items = append(q.items, v)
	q.mu.Unlock()
	q.cond.Signal() // 대기 중인 고루틴 하나 깨움 (Broadcast: 전부)
}

func (q *Queue) Get() int {
	q.mu.Lock()
	defer q.mu.Unlock()
	for len(q.items) == 0 { // 반드시 for 로 조건 재확인
		q.cond.Wait() // 락을 풀고 잠들었다가, 깨어나면 다시 락 획득
	}
	v := q.items[0]
	q.items = q.items[1:]
	return v
}

func main() {
	q := NewQueue()
	done := make(chan int)
	go func() { done <- q.Get() }()
	q.Put(42)
	fmt.Println(<-done)
	// 출력: 42
}
```

# 8. 선택 가이드

```
상황                                   → 도구
────────────────────────────────────────────────────────────
고루틴 N개 완료 대기                   → sync.WaitGroup (wg.Go)
카운터, 플래그                         → atomic.Int64, atomic.Bool
설정 전체를 통째로 교체                → atomic.Pointer[T]
구조체 상태 여러 필드 보호             → sync.Mutex
읽기가 압도적으로 많은 상태            → sync.RWMutex (벤치마크로 확인)
지연 초기화, 싱글턴                    → sync.Once / OnceValue
계속 늘어나기만 하는 캐시              → sync.Map
핫 경로의 임시 버퍼                    → sync.Pool
작업 분배, 결과 전달, 종료 신호        → 채널
첫 에러에서 전부 취소                  → errgroup (동시성 패턴 문서)
```

## 정리

- `sync.Mutex`는 Lock 직후 `defer Unlock`, 포함한 구조체는 포인터로만 전달
- 락 안에서 느린 작업 금지, 여러 락은 항상 같은 순서로
- `RWMutex`는 읽기가 압도적으로 많을 때만, 확신 없으면 Mutex
- `sync.Once`, Go 1.21+ `OnceFunc`/`OnceValue`/`OnceValues`로 한 번만 실행
- 단일 값은 `atomic.Int64`/`atomic.Bool`/`atomic.Pointer[T]`
- `sync.Map`은 특수한 접근 패턴에서만, 보통은 map + Mutex
- `sync.Pool`은 GC 부담이 큰 핫 경로의 임시 객체 재사용용
- `go vet`의 copylocks 검사와 `-race`로 실수 방지

## 참고 링크

- sync 패키지: https://pkg.go.dev/sync
- sync/atomic 패키지: https://pkg.go.dev/sync/atomic
- A Tour of Go (sync.Mutex): https://go.dev/tour/concurrency/9
- Go 메모리 모델 (동시성 보장 규칙): https://go.dev/ref/mem
- [한국어] Go by Example - Mutexes: https://mingrammer.com/gobyexample/mutexes/
- [한국어] Go by Example - Atomic Counters: https://mingrammer.com/gobyexample/atomic-counters/
- [한국어] Uber Go 스타일 가이드 (Mutex 제로값, 임베딩 주의 등): https://github.com/TangoEnSkai/uber-go-style-guide-kr

이전 문서: [02. 채널과 select](./02-채널과-select.md) | 다음 문서: [04. context](./04-context.md)
