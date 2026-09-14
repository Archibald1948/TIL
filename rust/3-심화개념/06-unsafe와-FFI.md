# unsafe와 FFI - 컴파일러의 보장 밖으로

## 들어가며

Rust 컴파일러의 검사는 **보수적**입니다. 안전하다고 증명할 수 없으면 거부합니다. 하지만 세상에는 컴파일러가 증명할 수 없어도 실제로는 안전한 코드가 있고, 운영체제나 C 라이브러리처럼 원래부터 Rust의 규칙 밖에 있는 것들도 있습니다.

`unsafe`는 "이 부분의 안전성은 컴파일러 대신 **개발자가 책임진다**"는 표시입니다.

```
❌ 오해: unsafe 블록 안에서는 빌림 검사기가 꺼진다
✅ 사실: 빌림 검사, 타입 검사는 그대로. 아래 5가지 "추가 능력"만 허용될 뿐
```

```
unsafe 로만 할 수 있는 5가지
1. 원시 포인터(raw pointer) 역참조
2. unsafe 함수/메서드 호출 (FFI 함수 포함)
3. 가변 static 변수 접근/수정
4. unsafe 트레이트 구현 (Send, Sync 등)
5. union 필드 접근
```

`Vec`, `String`, `HashMap`, `Arc`, `Mutex` 등 표준 라이브러리의 많은 부분이 내부적으로 unsafe를 쓰고, 그 위에 **안전한 API**를 제공합니다. 이것이 Rust의 핵심 전략입니다.

# 1. 원시 포인터

```rust
fn main() {
    let mut num = 5;

    // 원시 포인터 생성은 안전 (역참조만 unsafe)
    let r1 = &raw const num; // *const i32  (Rust 1.82+ 문법, 예전엔 &num as *const i32)
    let r2 = &raw mut num;   // *mut i32

    unsafe {
        println!("r1 = {}", *r1);
        *r2 = 10;
        println!("r2 = {}", *r2);
    }
    println!("num = {num}");

    // 임의의 주소도 만들 수는 있음 (역참조하면 정의되지 않은 동작)
    let _addr = 0x012345usize as *const i32;
}
```

```
참조 &T / &mut T                  원시 포인터 *const T / *mut T
- 항상 유효한 대상을 가리킴           - null, 댕글링일 수 있음
- 빌림 규칙 적용                     - 불변/가변 포인터 여러 개 동시 가능
- 자동 정리 없음 (참조는 소유 안 함)   - 자동 정리 없음
- null 불가                         - null 가능 (ptr.is_null())
```

# 2. unsafe 함수와 안전한 추상화

## unsafe fn

```rust
/// 슬라이스의 i 번째 요소를 경계 검사 없이 반환
///
/// # Safety
///
/// `i < slice.len()` 이어야 한다. 그렇지 않으면 정의되지 않은 동작.
unsafe fn get_unchecked_demo(slice: &[i32], i: usize) -> i32 {
    // edition 2024: unsafe fn 본문 안에서도 unsafe 연산은 unsafe 블록으로 감싸는 것이 원칙 (안 하면 경고)
    unsafe { *slice.as_ptr().add(i) }
}

fn main() {
    let v = [10, 20, 30];
    let x = unsafe { get_unchecked_demo(&v, 1) }; // 호출자가 조건을 보장
    println!("{x}");
}
```

- unsafe 함수에는 관례상 `# Safety` 문서 섹션에 **호출자가 지켜야 할 조건**을 적습니다.
- unsafe 블록에는 `// SAFETY: ...` 주석으로 **왜 안전한지** 적는 것이 관례입니다 (clippy 린트도 있음).

## 안전한 추상화 만들기: split_at_mut

```rust
// ❌ 컴파일 에러 예제
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    assert!(mid <= len);
    (&mut values[..mid], &mut values[mid..]) // 같은 슬라이스를 두 번 가변 빌림
}

fn main() {
    let mut v = [1, 2, 3];
    let _ = split_at_mut(&mut v, 1);
}
```

```
error[E0499]: cannot borrow `*values` as mutable more than once at a time
```

두 슬라이스는 겹치지 않으므로 실제로는 안전하지만 컴파일러는 이를 알 수 없습니다.

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len); // 안전 조건을 런타임에 검사

    // SAFETY: mid <= len 이므로 두 범위 [0, mid) 와 [mid, len) 은
    // 같은 할당 안에 있고 서로 겹치지 않는다.
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];
    let (a, b) = split_at_mut(&mut v, 3); // 호출하는 쪽은 unsafe 가 필요 없음
    a[0] = 100;
    b[0] = 400;
    println!("{v:?}");
}
```

함수 시그니처는 안전하고, `assert!`로 전제 조건을 보장하므로 **어떤 입력으로 호출해도** 정의되지 않은 동작이 발생하지 않습니다. 이것이 "안전한 추상화"입니다.

# 3. 가변 static

```rust
use std::sync::atomic::{AtomicU32, Ordering};

static mut COUNTER: u32 = 0;

// 대부분의 경우 static mut 대신 Atomic / Mutex / OnceLock 을 쓰는 것이 올바름
static SAFE_COUNTER: AtomicU32 = AtomicU32::new(0);

fn add_to_count(inc: u32) {
    // SAFETY: 이 예제는 싱글 스레드에서만 호출된다
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);
    add_to_count(4);
    // edition 2024: static mut 의 참조(&COUNTER)를 만드는 것은 기본적으로 에러 (static_mut_refs)
    // 값 복사로 읽기
    let value = unsafe { COUNTER };
    println!("COUNTER = {value}");

    SAFE_COUNTER.fetch_add(7, Ordering::Relaxed);
    println!("SAFE_COUNTER = {}", SAFE_COUNTER.load(Ordering::Relaxed));
}
```

# 4. unsafe 트레이트

```rust
use std::ptr::NonNull;

// 원시 포인터를 가진 타입은 자동으로 Send/Sync 가 되지 않음
struct MyBox<T> {
    ptr: NonNull<T>,
}

impl<T> MyBox<T> {
    fn new(value: T) -> Self {
        let b = Box::new(value);
        MyBox { ptr: NonNull::from(Box::leak(b)) }
    }
    fn get(&self) -> &T {
        // SAFETY: ptr 은 new 에서 유효한 Box 로부터 만들어졌고 drop 전까지 유효
        unsafe { self.ptr.as_ref() }
    }
}

impl<T> Drop for MyBox<T> {
    fn drop(&mut self) {
        // SAFETY: ptr 은 Box::leak 으로 얻은 것이므로 Box 로 되돌려 해제
        unsafe { drop(Box::from_raw(self.ptr.as_ptr())) };
    }
}

// SAFETY: MyBox<T> 는 T 를 단독 소유하므로 T 가 Send 면 스레드 간 이동이 안전
unsafe impl<T: Send> Send for MyBox<T> {}
// SAFETY: &MyBox<T> 로는 &T 만 얻을 수 있으므로 T 가 Sync 면 공유가 안전
unsafe impl<T: Sync> Sync for MyBox<T> {}

fn main() {
    let b = MyBox::new(String::from("스레드로 보내기"));
    let h = std::thread::spawn(move || println!("{}", b.get()));
    h.join().unwrap();
}
```

# 5. FFI - 다른 언어와 연결하기

## Rust에서 C 함수 호출

```rust
// edition 2024: extern 블록은 unsafe extern 으로 선언해야 함
unsafe extern "C" {
    fn abs(input: i32) -> i32;
    fn strlen(s: *const std::ffi::c_char) -> usize;

    // 호출해도 안전하다고 확신하는 함수는 safe 로 표시 가능 (Rust 1.82+)
    safe fn labs(input: std::ffi::c_long) -> std::ffi::c_long;
}

use std::ffi::{CStr, CString};

fn main() {
    // C 표준 라이브러리 함수 호출
    let a = unsafe { abs(-3) };
    let l = labs(-10); // safe 로 선언했으므로 unsafe 블록 불필요
    println!("abs(-3) = {a}, labs(-10) = {l}");

    // Rust 문자열 → C 문자열 (끝에 \0 추가)
    let s = CString::new("hello ffi").unwrap();
    let len = unsafe { strlen(s.as_ptr()) };
    println!("strlen = {len}");

    // C 문자열 → Rust 문자열
    let c_str: &CStr = c"C 문자열 리터럴"; // c"" 리터럴 (Rust 1.77+)
    println!("{}", c_str.to_str().unwrap());
}
```

```
Rust 타입            C 타입
i32 / u32           int / unsigned int (대부분 플랫폼)
std::ffi::c_int     int (플랫폼 독립적으로 정확)
*const c_char       const char*
CString / CStr      NUL 로 끝나는 문자열 (소유 / 빌림)
#[repr(C)] struct   C 구조체와 같은 메모리 배치
```

## C에서 Rust 함수 호출

```rust
// edition 2024: no_mangle 같은 속성은 unsafe(...) 로 감싸야 함
#[unsafe(no_mangle)]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}

#[repr(C)] // C 와 같은 필드 배치 보장
pub struct Point {
    pub x: f64,
    pub y: f64,
}

#[unsafe(no_mangle)]
pub extern "C" fn point_length(p: Point) -> f64 {
    (p.x * p.x + p.y * p.y).sqrt()
}

fn main() {
    println!("{}", rust_add(2, 3));
    println!("{}", point_length(Point { x: 3.0, y: 4.0 }));
}
```

```toml
# Cargo.toml - C 에서 링크할 수 있는 라이브러리로 빌드
[lib]
crate-type = ["cdylib"]    # .so / .dylib / .dll
# crate-type = ["staticlib"] # .a / .lib
```

```c
// main.c
#include <stdint.h>
int32_t rust_add(int32_t a, int32_t b);

int main(void) {
    return rust_add(2, 3);
}
```

C 헤더는 `cbindgen`으로 자동 생성할 수 있습니다.

## 바인딩 도구 생태계

| 도구 | 방향 | 용도 |
|---|---|---|
| `bindgen` | C/C++ → Rust | C 헤더에서 Rust `extern` 선언 자동 생성 |
| `cbindgen` | Rust → C | Rust 코드에서 C 헤더 생성 |
| `cxx` | Rust ↔ C++ | 안전한 C++ 상호 운용 |
| `PyO3` + `maturin` | Rust ↔ Python | Python 확장 모듈 (polars, ruff, uv 등) |
| `napi-rs` | Rust ↔ Node.js | Node 네이티브 애드온 (swc, rspack 등) |
| `wasm-bindgen` | Rust ↔ JS (WASM) | 브라우저에서 Rust 실행 |
| `uniffi` | Rust → Swift/Kotlin | 모바일 공유 라이브러리 |

## napi-rs로 Node.js에서 Rust 쓰기 (예시)

```rust
// (컴파일 검증 제외) - napi-rs 프로젝트 구조 필요
use napi_derive::napi;

#[napi]
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 | 1 => n,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

```js
// index.js
const { fibonacci } = require('./index.node');
console.log(fibonacci(30));
```

# 6. union

```rust
#[repr(C)]
union IntOrFloat {
    i: u32,
    f: f32,
}

fn main() {
    let u = IntOrFloat { f: 1.0 };
    // 어떤 필드가 유효한지 컴파일러가 모르므로 읽기는 unsafe
    let bits = unsafe { u.i };
    println!("1.0f32 의 비트 표현: {bits:#034b}");
    println!("to_bits 와 같음: {}", bits == 1.0f32.to_bits());
}
```

주로 C의 union과 FFI할 때 씁니다. Rust 코드에서는 보통 enum이 더 안전한 선택입니다.

# 7. 정의되지 않은 동작(UB)과 도구

unsafe 코드에서 다음을 하면 **정의되지 않은 동작**입니다. 컴파일러 최적화가 코드를 예측 불가능하게 바꿀 수 있습니다.

```
- 댕글링 / null / 정렬이 맞지 않는 포인터 역참조
- 빌림 규칙 위반 (&mut 두 개가 같은 데이터를 가리키게 만들기)
- 유효하지 않은 값 만들기 (bool 에 2, char 에 잘못된 코드 포인트, 초기화 안 된 메모리 읽기)
- 데이터 경쟁
- 잘못된 FFI 호출 규약
```

```bash
# Miri: unsafe 코드를 해석 실행하며 UB 를 검출
rustup +nightly component add miri
cargo +nightly miri test
cargo +nightly miri run

# Sanitizer (nightly)
RUSTFLAGS="-Zsanitizer=address" cargo +nightly run
```

## unsafe 사용 원칙

```
1. 최대한 작게: unsafe 블록은 꼭 필요한 연산만 감싼다
2. 안전한 API 로 감싼다: 모듈 경계 밖으로 unsafe 를 노출하지 않는다
3. 불변식을 문서화한다: # Safety (함수), // SAFETY: (블록)
4. 먼저 대안을 찾는다: std 나 검증된 크레이트에 이미 안전한 API 가 있는지
5. 테스트 + Miri 로 검증한다
6. #![forbid(unsafe_code)] 로 unsafe 가 필요 없는 크레이트는 아예 금지
```

## 정리

- `unsafe`는 컴파일러가 검증 못 하는 5가지 능력을 열어줄 뿐, 다른 검사는 그대로다
- 원시 포인터 역참조, unsafe 함수 호출, `static mut`, unsafe 트레이트, union 접근
- 핵심 전략은 **unsafe 내부 구현 + 안전한 공개 API** (예: `split_at_mut`, `Vec`)
- edition 2024에서는 `unsafe extern`(필수), `#[unsafe(no_mangle)]`(필수), `static mut` 참조 금지(기본 deny), unsafe fn 안의 unsafe 블록(경고) 등 규칙이 강화됐다
- FFI는 `extern "C"`, `#[repr(C)]`, `CString`/`CStr`로 연결하고, 실무에서는 bindgen/cxx/PyO3/napi-rs 사용
- Miri로 UB를 검사하고, `// SAFETY:` 주석으로 근거를 남기자

## 참고 문서

- 러스트 북 19.1 안전하지 않은 러스트 (한국어): https://doc.rust-kr.org/ch19-01-unsafe-rust.html
- The Rustonomicon (unsafe 전문서): https://doc.rust-lang.org/nomicon/
- Comprehensive Rust 안전하지 않은 러스트 (한국어): https://google.github.io/comprehensive-rust/ko/unsafe-rust.html
- Edition Guide - Rust 2024 변경점: https://doc.rust-lang.org/edition-guide/rust-2024/index.html
- Miri: https://github.com/rust-lang/miri
- bindgen: https://github.com/rust-lang/rust-bindgen
- PyO3: https://github.com/PyO3/pyo3
- napi-rs: https://github.com/napi-rs/napi-rs

## 이동

[← 이전: 매크로](./05-매크로.md) | [목차](../README.md) | [다음: 고급 트레이트와 타입 →](./07-고급-트레이트와-타입.md)
