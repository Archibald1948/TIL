# async/await - 비동기 프로그래밍과 Tokio

## 들어가며

스레드는 강력하지만 비쌉니다. 스레드 하나당 수 MB의 스택 메모리를 쓰고, 문맥 교환 비용도 있습니다. 동시 접속 1만 개를 스레드 1만 개로 처리하기는 어렵습니다.

대부분의 시간을 **I/O를 기다리며** 보내는 작업(웹 서버, DB 쿼리, API 호출)에는 비동기가 적합합니다.

```
스레드 (03-동시성)            async/await
- CPU 작업 병렬화에 적합       - I/O 대기가 많은 작업에 적합
- OS 가 스케줄링 (선점형)       - 런타임이 스케줄링 (협력형, .await 지점에서 양보)
- 스레드당 수 MB              - 태스크당 수백 바이트 ~ 수 KB
```

JS의 async/await와 문법은 거의 같지만 중요한 차이가 있습니다.

```
JavaScript                          Rust
- Promise 는 만들자마자 실행 시작     - Future 는 .await 하거나 poll 하기 전엔 아무것도 안 함 (lazy)
- 런타임(이벤트 루프) 내장            - 런타임이 언어에 없음 → tokio 같은 크레이트 선택
- 싱글 스레드                        - 멀티 스레드 런타임 가능 → Send 제약이 따라옴
```

# 1. Future와 async 기본

## async fn은 Future를 반환한다

```rust
// 외부 크레이트 tokio 필요 (features = ["full"])
async fn say_hello() -> String {
    println!("say_hello 실행됨");
    "hello".to_string()
}

#[tokio::main] // main 을 비동기 런타임 위에서 실행하도록 변환하는 매크로
async fn main() {
    let fut = say_hello(); // 아직 아무것도 출력되지 않음 (lazy)
    println!("Future 생성됨");

    let s = fut.await; // 이 시점에 실행
    println!("{s}");
}
// 출력:
// Future 생성됨
// say_hello 실행됨
// hello
```

## Future 트레이트

```rust
// (컴파일 검증 제외) - 표준 라이브러리 정의 요약
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

```
동작 흐름
1. 런타임(executor)이 Future 의 poll 을 호출
2. 아직 준비 안 됨 → Pending 반환, 준비되면 깨워달라고 Waker 를 등록
3. I/O 준비 완료 → OS 이벤트(epoll/kqueue) → Waker 호출 → 런타임이 다시 poll
4. Ready(값) 반환 → 완료
```

컴파일러는 `async fn`을 **상태 머신(enum)**으로 변환합니다. 각 `.await` 지점이 하나의 상태가 됩니다. 그래서 힙 할당 없이도 비동기 코드가 동작합니다.

## 런타임 설정

```bash
cargo add tokio --features full
```

```rust
// 외부 크레이트 tokio 필요
// #[tokio::main] 은 대략 이렇게 전개됨
fn main() {
    tokio::runtime::Builder::new_multi_thread()
        .worker_threads(4)
        .enable_all()
        .build()
        .unwrap()
        .block_on(async {
            println!("런타임 안에서 실행");
        });
}
```

- `#[tokio::main]`: 멀티스레드 런타임 (기본 워커 수 = CPU 코어 수)
- `#[tokio::main(flavor = "current_thread")]`: 싱글 스레드 런타임

# 2. 동시에 실행하기

## 순차 vs 동시

```rust
// 외부 크레이트 tokio 필요
use std::time::{Duration, Instant};
use tokio::time::sleep;

async fn fetch(name: &str, ms: u64) -> String {
    sleep(Duration::from_millis(ms)).await; // 비동기 sleep (스레드를 막지 않음)
    format!("{name} 완료")
}

#[tokio::main]
async fn main() {
    // 순차: 약 300ms
    let start = Instant::now();
    let a = fetch("A", 100).await;
    let b = fetch("B", 200).await;
    println!("{a}, {b} - 순차 {:?}", start.elapsed());

    // 동시: 약 200ms (Promise.all 과 비슷)
    let start = Instant::now();
    let (a, b) = tokio::join!(fetch("A", 100), fetch("B", 200));
    println!("{a}, {b} - join! {:?}", start.elapsed());

    // 하나라도 Err 면 즉시 반환: try_join!
    let r: Result<(u32, u32), String> = tokio::try_join!(async { Ok(1) }, async { Err("실패".to_string()) });
    println!("{r:?}");
}
```

## spawn - 독립적인 태스크

```rust
// 외부 크레이트 tokio 필요
use std::time::Duration;
use tokio::task::JoinSet;

#[tokio::main]
async fn main() {
    // tokio::spawn: 런타임에 태스크를 맡기고 JoinHandle 반환 (스레드 spawn 과 비슷)
    let handle = tokio::spawn(async {
        tokio::time::sleep(Duration::from_millis(50)).await;
        42
    });
    println!("태스크 결과: {}", handle.await.unwrap());

    // JoinSet: 여러 태스크를 모아서 완료되는 순서대로 받기
    let mut set = JoinSet::new();
    for i in 0..5u64 {
        set.spawn(async move {
            tokio::time::sleep(Duration::from_millis(50 * (5 - i))).await;
            i
        });
    }
    while let Some(res) = set.join_next().await {
        println!("완료: {}", res.unwrap()); // 4, 3, 2, 1, 0 순서
    }
}
```

```
join!         → 같은 태스크 안에서 여러 Future 를 번갈아 poll (병렬 X, 동시 O)
tokio::spawn  → 별도 태스크로 런타임에 제출. 멀티스레드 런타임이면 다른 스레드에서 병렬 실행 가능
               → 그래서 spawn 하는 Future 는 Send + 'static 이어야 함
```

## select! - 먼저 끝나는 것 선택

```rust
// 외부 크레이트 tokio 필요
use std::time::Duration;
use tokio::time::{sleep, timeout};

async fn slow_query() -> &'static str {
    sleep(Duration::from_millis(500)).await;
    "쿼리 결과"
}

#[tokio::main]
async fn main() {
    // 먼저 완료된 분기만 실행되고 나머지 Future 는 drop(취소)됨
    tokio::select! {
        v = slow_query() => println!("{v}"),
        _ = sleep(Duration::from_millis(100)) => println!("100ms 초과, 취소"),
    }

    // 타임아웃 전용 함수
    match timeout(Duration::from_millis(100), slow_query()).await {
        Ok(v) => println!("{v}"),
        Err(_) => println!("timeout!"),
    }
}
```

Rust에서 **취소는 Future를 drop하는 것**입니다. `.await` 지점 사이에서 중단될 수 있으므로, 중간 상태가 깨지지 않게 설계해야 합니다(cancellation safety).

# 3. 비동기 채널과 공유 상태

## tokio::sync::mpsc

```rust
// 외부 크레이트 tokio 필요
use tokio::sync::{mpsc, oneshot};

#[derive(Debug)]
enum Command {
    Get { key: String, resp: oneshot::Sender<Option<i32>> },
    Set { key: String, value: i32 },
}

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<Command>(32); // bounded

    // 상태를 소유하는 매니저 태스크 (actor 패턴: 락 없이 상태 관리)
    let manager = tokio::spawn(async move {
        let mut store = std::collections::HashMap::new();
        while let Some(cmd) = rx.recv().await {
            match cmd {
                Command::Set { key, value } => {
                    store.insert(key, value);
                }
                Command::Get { key, resp } => {
                    let _ = resp.send(store.get(&key).copied());
                }
            }
        }
    });

    tx.send(Command::Set { key: "a".into(), value: 1 }).await.unwrap();

    let (resp_tx, resp_rx) = oneshot::channel(); // 응답용 일회성 채널
    tx.send(Command::Get { key: "a".into(), resp: resp_tx }).await.unwrap();
    println!("a = {:?}", resp_rx.await.unwrap());

    drop(tx); // 송신자 모두 drop → 매니저 루프 종료
    manager.await.unwrap();
}
```

| 채널 | 용도 |
|---|---|
| `mpsc` | 여러 생산자 → 하나의 소비자 |
| `oneshot` | 값 하나를 한 번만 (요청-응답) |
| `broadcast` | 여러 소비자에게 모두 전달 (pub/sub) |
| `watch` | 최신 값 하나만 유지 (설정 변경 알림) |

## 공유 상태: Arc<Mutex<T>>

```rust
// 외부 크레이트 tokio 필요
use std::sync::{Arc, Mutex};

#[tokio::main]
async fn main() {
    let counter = Arc::new(Mutex::new(0)); // 짧게 잡고 .await 전에 풀면 std Mutex 로 충분

    let mut handles = vec![];
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(tokio::spawn(async move {
            {
                let mut n = counter.lock().unwrap();
                *n += 1;
            } // .await 전에 가드 해제
            tokio::task::yield_now().await;
        }));
    }
    for h in handles {
        h.await.unwrap();
    }
    println!("{}", *counter.lock().unwrap());
}
```

```
std::sync::Mutex    → 락을 짧게 잡고 .await 를 넘기지 않을 때 (대부분의 경우, 더 빠름)
tokio::sync::Mutex  → 락을 잡은 채로 .await 해야 할 때 (예: DB 연결을 들고 비동기 호출)
```

std `MutexGuard`를 들고 `.await`하면, 가드가 `Send`가 아니어서 `tokio::spawn`에서 **컴파일 에러**가 납니다 (`future cannot be sent between threads safely`). 이것도 컴파일러가 교착 상태 위험을 알려주는 셈입니다.

# 4. 블로킹 작업 다루기

```rust
// 외부 크레이트 tokio 필요
use std::time::Duration;

fn heavy_compute(n: u64) -> u64 {
    std::thread::sleep(Duration::from_millis(100)); // CPU 작업 또는 블로킹 I/O 흉내
    (1..=n).sum()
}

#[tokio::main]
async fn main() {
    // ❌ async 안에서 블로킹 호출을 하면 워커 스레드가 멈춰 다른 태스크도 지연됨
    // let r = heavy_compute(1_000_000);

    // ✅ 블로킹 전용 스레드 풀로 보내기
    let r = tokio::task::spawn_blocking(|| heavy_compute(1_000_000)).await.unwrap();
    println!("{r}");

    // 파일 I/O 는 tokio::fs (내부적으로 spawn_blocking 사용)
    tokio::fs::write("/tmp/tokio_til_demo.txt", "hello").await.unwrap();
    let text = tokio::fs::read_to_string("/tmp/tokio_til_demo.txt").await.unwrap();
    println!("{text}");
}
```

```
async 안에서 하면 안 되는 것
- std::thread::sleep       → tokio::time::sleep
- std::fs, std::net 블로킹 I/O → tokio::fs, tokio::net
- 오래 걸리는 CPU 계산       → spawn_blocking 또는 rayon
- std Mutex 를 잡은 채 .await
```

# 5. 실전 예제: TCP 에코 서버

```rust
// 외부 크레이트 tokio 필요
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::net::{TcpListener, TcpStream};

async fn handle(stream: TcpStream) -> std::io::Result<()> {
    let peer = stream.peer_addr()?;
    let (reader, mut writer) = stream.into_split();
    let mut lines = BufReader::new(reader).lines();

    while let Some(line) = lines.next_line().await? {
        writer.write_all(format!("echo: {line}\n").as_bytes()).await?;
    }
    println!("{peer} 연결 종료");
    Ok(())
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:0").await?; // 0: 빈 포트 자동 선택
    let addr = listener.local_addr()?;
    println!("listening on {addr}");

    // 서버 태스크: 연결마다 태스크 하나 (수만 개도 가능)
    tokio::spawn(async move {
        loop {
            let (stream, _) = listener.accept().await.unwrap();
            tokio::spawn(async move {
                if let Err(e) = handle(stream).await {
                    eprintln!("에러: {e}");
                }
            });
        }
    });

    // 클라이언트로 테스트
    let mut client = TcpStream::connect(addr).await?;
    client.write_all(b"hello\nrust\n").await?;
    client.shutdown().await?;

    let mut resp = String::new();
    tokio::io::AsyncReadExt::read_to_string(&mut client, &mut resp).await?;
    print!("{resp}");
    Ok(())
}
```

# 6. 알아두면 좋은 것들

## 트레이트의 async fn (Rust 1.75+)

```rust
// 외부 크레이트 tokio 필요
trait Repository {
    async fn find_name(&self, id: u32) -> Option<String>;
}

struct MemoryRepo;

impl Repository for MemoryRepo {
    async fn find_name(&self, id: u32) -> Option<String> {
        tokio::task::yield_now().await;
        (id == 1).then(|| "kim".to_string())
    }
}

#[tokio::main]
async fn main() {
    let repo = MemoryRepo;
    println!("{:?}", repo.find_name(1).await);
}
```

예전에는 `async-trait` 크레이트가 필요했지만 이제 기본 지원됩니다. 다만 `dyn Repository`처럼 트레이트 객체로 쓰려면 아직 `async-trait` 크레이트나 수동 `Pin<Box<dyn Future>>`가 필요합니다.

## async 클로저 (Rust 1.85+)

```rust
// 외부 크레이트 tokio 필요
async fn retry<F>(times: u32, f: F) -> Result<u32, String>
where
    F: AsyncFn(u32) -> Result<u32, String>,
{
    let mut last = Err("시도 안 함".to_string());
    for attempt in 1..=times {
        last = f(attempt).await;
        if last.is_ok() {
            break;
        }
    }
    last
}

#[tokio::main]
async fn main() {
    let threshold = 3;
    let r = retry(5, async |n| if n >= threshold { Ok(n) } else { Err(format!("{n}번째 실패")) }).await;
    println!("{r:?}");
}
```

## Stream - 비동기 이터레이터

```rust
// 외부 크레이트 futures, tokio 필요
use futures::stream::{self, StreamExt};

#[tokio::main]
async fn main() {
    let results: Vec<u32> = stream::iter(1..=10)
        .map(|n| async move {
            tokio::time::sleep(std::time::Duration::from_millis(10)).await;
            n * n
        })
        .buffer_unordered(3) // 최대 3개씩 동시 실행
        .collect()
        .await;

    let mut sorted = results.clone();
    sorted.sort();
    println!("{sorted:?}");
}
```

## Pin은 왜 필요할까 (개념만)

async 상태 머신은 자기 자신의 필드를 가리키는 참조(**자기 참조 구조체**)를 가질 수 있습니다. 이런 값이 메모리에서 이동하면 내부 참조가 깨지므로, `Pin`으로 "이 값은 더 이상 이동하지 않는다"를 보장합니다. 일반적인 애플리케이션 코드에서는 직접 다룰 일이 거의 없고, `Box::pin(fut)`이나 `tokio::pin!(fut)`을 쓰는 정도입니다.

## 생태계

| 크레이트 | 용도 |
|---|---|
| `tokio` | 사실상 표준 비동기 런타임 |
| `axum` | tokio 팀의 웹 프레임워크 |
| `reqwest` | HTTP 클라이언트 |
| `sqlx` | 비동기 SQL (컴파일 타임 쿼리 검사) |
| `tonic` | gRPC |
| `futures` | Stream, 조합 유틸리티 |
| `tracing` | 비동기 친화적 로깅/트레이싱 |
| `smol`, `embassy` | 경량 런타임 / 임베디드용 비동기 |

## 정리

- `async fn`은 Future를 반환하고, Future는 `.await`하기 전까지 실행되지 않는다 (lazy)
- 런타임은 언어에 없으므로 `tokio`를 쓴다 (`#[tokio::main]`)
- 동시 실행은 `join!`, 독립 태스크는 `tokio::spawn`, 경쟁은 `select!`, 시간 제한은 `timeout`
- 취소 = Future drop
- 태스크 간 통신은 `mpsc`/`oneshot`/`broadcast`/`watch`
- async 안에서 블로킹 금지 → `spawn_blocking`, `tokio::fs`, `tokio::time::sleep`
- std `MutexGuard`를 `.await` 너머로 들고 가지 말 것
- 트레이트의 async fn(1.75+), async 클로저(1.85+)가 stable

## 참고 문서

- Tokio 튜토리얼: https://tokio.rs/tokio/tutorial
- Asynchronous Programming in Rust (async book): https://rust-lang.github.io/async-book/
- 러스트 비동기 프로그래밍 한글판 (async book 번역, 2022년 이후 업데이트 없음): https://sephiron99.github.io/async-book-ko/
- 러스트 북 17장 Async (영문, 한국어판은 아직 해당 장 없음): https://doc.rust-lang.org/book/ch17-00-async-await.html
- Comprehensive Rust async 기초 (한국어): https://google.github.io/comprehensive-rust/ko/async.html
- mini-redis (Tokio 학습용 예제 프로젝트): https://github.com/tokio-rs/mini-redis

## 이동

[← 이전: 동시성](./03-동시성.md) | [목차](../README.md) | [다음: 매크로 →](./05-매크로.md)
