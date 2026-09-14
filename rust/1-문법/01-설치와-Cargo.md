# 설치와 Cargo - Rust 개발 환경 만들기

## 들어가며

Rust를 시작하면 가장 먼저 만나는 도구는 컴파일러(`rustc`)가 아니라 **Cargo**입니다.

```
JS/TS 개발자 입장에서 대응시키면:

nvm / fnm        → rustup   (툴체인 버전 관리)
node             → rustc    (컴파일러)
npm / pnpm       → cargo    (빌드 + 패키지 매니저 + 테스트 러너)
package.json     → Cargo.toml
package-lock     → Cargo.lock
npmjs.com        → crates.io
eslint           → clippy
prettier         → rustfmt
```

Cargo 하나로 프로젝트 생성, 빌드, 실행, 테스트, 문서 생성, 배포까지 전부 됩니다.

# 1. 설치

## rustup으로 설치 (권장)

```bash
# macOS / Linux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 설치 후 셸 재시작 또는
source "$HOME/.cargo/env"

# 버전 확인
rustc --version
cargo --version
```

Windows는 https://rustup.rs 에서 `rustup-init.exe`를 받아 실행합니다. (Visual Studio C++ Build Tools 필요)

> Homebrew의 `brew install rust`로도 설치되지만, 툴체인 전환이나 컴포넌트 추가가 불편해서 **rustup을 권장**합니다.

## rustup 주요 명령어

```bash
rustup update                  # 최신 stable로 업데이트
rustup toolchain list          # 설치된 툴체인 목록
rustup toolchain install nightly
rustup default stable          # 기본 툴체인 지정
rustup override set nightly    # 현재 디렉터리만 nightly 사용

rustup component add clippy rustfmt rust-analyzer
rustup target add wasm32-unknown-unknown   # 크로스 컴파일 타깃 추가

rustup doc                     # 오프라인 공식 문서 열기 (the book 포함)
rustup self uninstall          # 제거
```

## 릴리스 채널

```
stable   → 6주마다 새 버전 출시 (1.85, 1.86, ...)
beta     → 다음 stable 후보
nightly  → 매일 빌드, 불안정 기능(#![feature]) 사용 가능
```

프로젝트마다 툴체인을 고정하고 싶으면 루트에 `rust-toolchain.toml`을 둡니다.

```toml
# rust-toolchain.toml
[toolchain]
channel = "1.98.1"
components = ["clippy", "rustfmt"]
```

## 에디터

- **VS Code** + `rust-analyzer` 확장 (사실상 표준)
- **RustRover** (JetBrains)
- **Zed**, **Helix**, **Neovim** (rust-analyzer LSP 사용)

# 2. Hello, World!

## rustc로 직접 컴파일

```rust
// main.rs
fn main() {
    println!("Hello, world!");
}
```

```bash
rustc main.rs
./main
```

`println!` 뒤의 `!`는 함수가 아니라 **매크로**라는 뜻입니다. (매크로는 [심화개념/05-매크로](../3-심화개념/05-매크로.md)에서 다룹니다)

실무에서는 `rustc`를 직접 부를 일이 거의 없고 Cargo를 씁니다.

# 3. Cargo 기본

## 프로젝트 생성

```bash
cargo new hello          # 실행 파일(바이너리) 프로젝트
cargo new mylib --lib    # 라이브러리 프로젝트
cargo init               # 현재 폴더를 Cargo 프로젝트로
```

생성되는 구조:

```
hello/
├── Cargo.toml
├── .gitignore
└── src/
    └── main.rs     # --lib 이면 lib.rs
```

## Cargo.toml

```toml
[package]
name = "hello"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = { version = "1", features = ["derive"] }
rand = "0.10"

[dev-dependencies]    # 테스트/예제/벤치에서만 사용
pretty_assertions = "1"
```

`edition`은 언어 문법 세대입니다. `2015 → 2018 → 2021 → 2024` 순서이며, **Rust 1.85부터 2024 에디션이 stable**이 되었고 `cargo new`의 기본값입니다. 에디션이 달라도 크레이트끼리는 문제없이 섞어 쓸 수 있습니다.

## 버전 표기 (SemVer)

```toml
rand = "0.10"       # ^0.10 과 같음 → 0.10.x 허용, 0.11은 불허 (0.x 는 마이너가 호환 경계)
serde = "1.0.200"   # ^1.0.200 → 1.x.y (>= 1.0.200) 허용
tokio = "=1.40.0"   # 정확히 이 버전
foo = ">=1.2, <1.5"
```

npm과 달리 **캐럿(^)이 기본**이라 앞에 기호를 안 붙여도 됩니다.

## 자주 쓰는 명령어

```bash
cargo build              # 디버그 빌드 → target/debug/
cargo build --release    # 최적화 빌드 → target/release/
cargo run                # 빌드 + 실행
cargo run -- arg1 arg2   # 프로그램에 인자 전달
cargo check              # 컴파일 검사만 (빌드보다 훨씬 빠름)
cargo test               # 테스트 실행
cargo doc --open         # 내 프로젝트 + 의존성 문서 생성 후 열기

cargo add serde --features derive   # 의존성 추가 (Cargo.toml 자동 수정)
cargo remove serde
cargo update             # Cargo.lock 갱신

cargo fmt                # 코드 포맷팅 (rustfmt)
cargo clippy             # 린트 (흔한 실수, 비관용적 코드 지적)
cargo install ripgrep    # crates.io의 바이너리 설치
cargo tree               # 의존성 트리 보기
```

> 개발 중에는 `cargo check`를 가장 많이 씁니다. 코드 생성 단계를 건너뛰어서 빠릅니다.

## debug vs release

```
cargo build            cargo build --release
- 최적화 없음 (opt-level 0)   - 최적화 (opt-level 3)
- 빠른 컴파일                 - 느린 컴파일
- 정수 오버플로 시 panic       - 정수 오버플로 시 wrap around
- 디버그 심볼 포함             - 실행 속도 수십 배 빠를 수 있음
```

벤치마크나 성능 측정은 **반드시 `--release`**로 해야 합니다.

프로필은 Cargo.toml에서 조정할 수 있습니다.

```toml
[profile.release]
lto = true           # 링크 타임 최적화
codegen-units = 1    # 최적화 극대화 (빌드는 느려짐)
strip = true         # 심볼 제거로 바이너리 크기 감소
panic = "abort"      # unwind 대신 즉시 종료 (크기 감소)
```

# 4. 프로젝트 구조 확장

## 여러 바이너리, 예제

```
my-app/
├── Cargo.toml
├── src/
│   ├── main.rs          # 기본 바이너리 (cargo run)
│   ├── lib.rs           # 라이브러리 크레이트
│   └── bin/
│       └── tool.rs      # 추가 바이너리 (cargo run --bin tool)
├── examples/
│   └── demo.rs          # cargo run --example demo
├── tests/
│   └── integration.rs   # 통합 테스트
└── benches/
    └── bench.rs         # 벤치마크
```

Cargo는 **디렉터리 규칙(convention)**으로 타깃을 자동 인식하므로 따로 설정할 필요가 없습니다.

## 워크스페이스 (모노레포)

여러 크레이트를 한 저장소에서 관리할 때 씁니다. turborepo/pnpm workspace와 비슷한 개념입니다.

```toml
# 루트 Cargo.toml
[workspace]
resolver = "3"
members = ["crates/shared", "crates/cli", "crates/server"]

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
```

```toml
# crates/cli/Cargo.toml
[package]
name = "cli"
version = "0.1.0"
edition = "2024"

[dependencies]
shared = { path = "../shared" }
serde = { workspace = true }   # 루트에서 버전 상속
```

- `Cargo.lock`과 `target/`을 워크스페이스 전체가 공유합니다.
- `cargo build -p cli`처럼 특정 멤버만 빌드할 수 있습니다.

## features (조건부 컴파일)

```toml
[features]
default = ["json"]
json = ["dep:serde_json"]
full = ["json"]

[dependencies]
serde_json = { version = "1", optional = true }
```

```rust
#[cfg(feature = "json")]
pub fn to_json() { /* serde_json 사용 */ }
```

```bash
cargo build --no-default-features --features full
```

# 5. 알아두면 좋은 Cargo 도구

```bash
cargo install cargo-watch     # 파일 변경 시 자동 재실행: cargo watch -x run
cargo install cargo-edit      # cargo upgrade (의존성 버전 일괄 업그레이드)
cargo install cargo-expand    # 매크로 전개 결과 보기
cargo install cargo-nextest   # 더 빠른 테스트 러너
cargo install cargo-audit     # 의존성 보안 취약점 검사
cargo install cargo-outdated  # 오래된 의존성 확인
```

## Playground

설치 없이 브라우저에서 바로 실행해볼 수 있습니다: https://play.rust-lang.org

## 정리

- `rustup`으로 툴체인 관리, `cargo`로 거의 모든 작업을 한다
- `Cargo.toml`에 의존성과 설정, `Cargo.lock`은 정확한 버전 고정
- `edition = "2024"`가 현재 기본 (Rust 1.85부터)
- 개발 중엔 `cargo check`, 배포/성능 측정은 `cargo build --release`
- `cargo fmt` + `cargo clippy`는 습관처럼 돌리기
- 규모가 커지면 워크스페이스로 크레이트를 나눈다

## 참고 문서

- 설치 가이드: https://www.rust-lang.org/tools/install
- The Cargo Book: https://doc.rust-lang.org/cargo/
- 러스트 북 1장 시작해봅시다 (한국어): https://doc.rust-kr.org/ch01-00-getting-started.html
- 러스트 북 14장 카고와 Crates.io (한국어): https://doc.rust-kr.org/ch14-00-more-about-cargo.html
- 에디션 가이드: https://doc.rust-lang.org/edition-guide/

## 이동

[목차](../README.md) | [다음: 변수와 자료형 →](./02-변수와-자료형.md)
