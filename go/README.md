## Go

Go(Golang)에 관한 TIL입니다.

Go는 2009년 구글의 Robert Griesemer, Rob Pike, Ken Thompson이 발표한 오픈소스 프로그래밍 언어입니다. **적은 키워드와 단순한 문법, 빠른 컴파일, 런타임 없이 실행되는 단일 바이너리, 고루틴 기반의 쉬운 동시성**이 특징이며, Docker·Kubernetes·Terraform·Prometheus 같은 클라우드 인프라 도구와 백엔드 서버, CLI 도구 개발에 널리 쓰입니다. "기능을 더하기보다 덜어내는" 설계 철학 덕분에 누가 작성해도 비슷한 모양의 코드가 나오고, 대규모 팀에서도 읽기 쉬운 코드를 유지하기 좋습니다.

### 공식 사이트

https://go.dev

### 공식 GitHub

https://github.com/golang/go

### 작성 기준

- Go 1.27 (2026년 8월 릴리스) 기준으로 작성했고, 버전별로 추가된 기능은 도입 버전을 함께 표기했습니다.
- 문서의 `package main` 코드 예제는 로컬 Go 1.27.1 환경에서 실행/컴파일을 확인했습니다.

### 학습 로드맵

```
1단계: 문법
  설치/모듈 → 변수/타입 → 함수/제어문 → 슬라이스/맵 → 구조체/메서드 → 포인터 → 패키지

2단계: 기초 개념
  인터페이스 → 에러 처리 → defer/panic/recover → 제네릭 → 테스트 → 표준 라이브러리

3단계: 심화 개념
  고루틴 → 채널/select → sync → context → 동시성 패턴 → 메모리/GC/pprof → 이터레이터/리플렉션

4단계: 프로젝트
  todo CLI → REST API 서버 → 크롤러/채팅 서버 → gRPC/인프라 도구
```

### 목차

#### 1-문법

| 문서 | 한 줄 설명 |
|------|-----------|
| [01. 설치와 Go 모듈](./1-문법/01-설치와-Go-모듈.md) | 설치, go.mod/go.sum, 의존성 관리, go 명령어, 크로스 컴파일 |
| [02. 변수와 자료형](./1-문법/02-변수와-자료형.md) | var/:=, 제로값, 기본 타입, 문자열과 rune, 타입 변환, const/iota |
| [03. 함수와 제어 흐름](./1-문법/03-함수와-제어흐름.md) | 다중 반환, 클로저, 함수형 옵션, if/for/switch, 루프 변수 변경(1.22) |
| [04. 배열, 슬라이스, 맵](./1-문법/04-배열-슬라이스-맵.md) | 슬라이스 내부 구조와 공유 함정, append, slices/maps 패키지 |
| [05. 구조체와 메서드](./1-문법/05-구조체와-메서드.md) | 구조체, 생성자 관례, 값/포인터 리시버, 임베딩, 구조체 태그 |
| [06. 포인터](./1-문법/06-포인터.md) | &와 *, nil, 탈출 분석 맛보기, new(expr)(1.26), 값 vs 포인터 선택 |
| [07. 패키지와 가시성](./1-문법/07-패키지와-가시성.md) | 대소문자 가시성, import, internal, init, godoc, embed, 빌드 태그 |

#### 2-기초개념

| 문서 | 한 줄 설명 |
|------|-----------|
| [01. 인터페이스](./2-기초개념/01-인터페이스.md) | 암묵적 구현, 작은 인터페이스, 타입 단언/스위치, nil 인터페이스 함정 |
| [02. 에러 처리](./2-기초개념/02-에러-처리.md) | error 값, %w 래핑, errors.Is/As/AsType, 센티널/커스텀 에러, Join |
| [03. defer, panic, recover](./2-기초개념/03-defer-panic-recover.md) | defer 규칙, panic 전파, recover, 고루틴 panic, os.Exit 주의 |
| [04. 제네릭](./2-기초개념/04-제네릭.md) | 타입 파라미터, 제약, 제네릭 타입, 제네릭 메서드(1.27), 사용 기준 |
| [05. 테스트와 벤치마크](./2-기초개념/05-테스트와-벤치마크.md) | 테이블 테스트, httptest, b.Loop 벤치마크, 퍼징, 커버리지, synctest |
| [06. 표준 라이브러리](./2-기초개념/06-표준-라이브러리.md) | fmt, strings, strconv, os, io/bufio, json(v2), net/http, time, slog |

#### 3-심화개념

| 문서 | 한 줄 설명 |
|------|-----------|
| [01. 고루틴](./3-심화개념/01-고루틴.md) | 고루틴, WaitGroup(wg.Go), 데이터 레이스와 -race, GMP 스케줄러, 누수 |
| [02. 채널과 select](./3-심화개념/02-채널과-select.md) | 버퍼/비버퍼 채널, close와 range, 방향 채널, select, 타임아웃 |
| [03. sync 패키지](./3-심화개념/03-sync-패키지.md) | Mutex, RWMutex, Once/OnceValue, atomic, sync.Map, sync.Pool |
| [04. context](./3-심화개념/04-context.md) | 취소/타임아웃 전파, WithValue, HTTP context, graceful shutdown |
| [05. 동시성 패턴](./3-심화개념/05-동시성-패턴.md) | Worker Pool, errgroup, Pipeline, Fan-out/Fan-in, singleflight, rate limit |
| [06. 메모리 관리와 GC](./3-심화개념/06-메모리-관리와-GC.md) | 스택/힙, 탈출 분석, GC 동작, GOGC/GOMEMLIMIT, pprof, PGO |
| [07. 이터레이터와 리플렉션](./3-심화개념/07-이터레이터와-리플렉션.md) | range over func, iter.Seq/Pull, reflect로 필드/태그 다루기 |

#### 프로젝트와 자료

| 문서 | 한 줄 설명 |
|------|-----------|
| [4. 예시 프로젝트](./4-예시-프로젝트.md) | 난이도별 Go 프로젝트 아이디어, 추천 라이브러리, 참고 오픈소스 |
| [5. 참고 자료](./5-참고-자료.md) | 공식 문서, GitHub, 한국어 학습 자료, 커뮤니티, 연습 사이트 |

### 빠른 시작

```bash
# 설치 확인
go version

# 프로젝트 생성
mkdir hello && cd hello
go mod init github.com/username/hello

# main.go 작성 후 실행
go run .
```

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, Go!")
}
```

### 한국어로 시작하기 좋은 자료

- [한국어] Tucker의 Go 언어 프로그래밍 무료 유튜브 강의: https://www.youtube.com/playlist?list=PLy-g2fnSzUTBHwuXkWQ834QHDZwLx6v6j
- [한국어] A Tour of Go 한국어판 (커뮤니티 번역): https://ko-go-dev.shuijingwanwq.com/tour
- [한국어] 예제로 배우는 Go 프로그래밍: http://golang.site/go/basics
- [한국어] Learn Go with Tests 한국어 번역: https://miryang.gitbook.io/learn-go-with-tests
- [한국어] Uber Go 스타일 가이드: https://github.com/TangoEnSkai/uber-go-style-guide-kr
- 전체 목록은 [5. 참고 자료](./5-참고-자료.md) 참고
