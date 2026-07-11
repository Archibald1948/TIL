# 패키지 매니저 - 의존성 관리의 완벽한 이해

## 들어가며

소프트웨어 개발에서 가장 번거로운 일 중 하나는 **라이브러리와 의존성 관리**입니다.

```
문제 상황:
- 필요한 라이브러리 찾기
- 호환되는 버전 찾기
- 의존성의 의존성 해결하기
- 버전 업데이트 관리하기
- 팀원과 같은 버전 유지하기

→ 패키지 매니저가 이 모든 것을 자동화합니다!
```

## 패키지 매니저란?

### 개념

패키지 매니저는 **소프트웨어 패키지(라이브러리)를 자동으로 설치, 업데이트, 제거하고 의존성을 관리하는 도구**입니다.

```
패키지 매니저의 역할:

1. 설치: npm install express
   → npm 레지스트리에서 express 패키지 다운로드

2. 버전 관리: express@4.18.0
   → 특정 버전만 설치

3. 의존성 해결: express는 body-parser 필요
   → 자동으로 body-parser도 설치

4. 잠금: package-lock.json
   → 팀원들이 동일한 버전 사용하도록 보장

5. 스크립트 실행: npm start
   → 자동화된 명령어 실행
```

### 왜 필요한가?

```
패키지 매니저 없다면:
├─ 공식 사이트에서 직접 다운로드
├─ 수동으로 폴더 구성
├─ 버전 추적 어려움
├─ 의존성 충돌 처리 복잡
└─ 팀원과 버전 동기화 불가능

패키지 매니저 사용:
├─ 한 줄의 명령어로 설치
├─ 자동으로 폴더 구성
├─ package.json에서 버전 추적
├─ 자동 의존성 해결
└─ lock 파일로 버전 동기화
```

## 주요 패키지 매니저

### 1. npm (Node Package Manager)

#### 개요

- 공식 사이트: https://www.npmjs.com/
- GitHub: https://github.com/npm/cli
- 문서: https://docs.npmjs.com/
- 설치: Node.js 설치 시 함께 설치됨

**특징:**

- JavaScript/Node.js 표준 패키지 매니저
- 가장 큰 패키지 생태계 (수백만 개 패키지)
- 모두가 사용하는 표준
- 가장 오래된 패키지 매니저 (2010년)

#### 기본 명령어

```bash
# 패키지 설치
npm install express                    # 최신 버전 설치
npm install express@4.18.0            # 특정 버전 설치
npm install express@4.18.x            # 4.18.x 범위 내 최신
npm install                            # package.json 기반 설치

# 개발 환경 전용 설치
npm install --save-dev prettier       # 또는 npm install -D prettier

# 전역 설치
npm install -g @angular/cli           # 전역으로 설치

# 패키지 제거
npm uninstall express                 # 패키지 제거
npm uninstall -D prettier             # 개발 의존성 제거

# 버전 업데이트
npm update express                    # 안전한 범위 내 최신 버전
npm update                            # 모든 패키지 업데이트

# 버전 관리
npm outdated                          # 업데이트 가능한 패키지 확인
npm view express versions             # 패키지의 모든 버전 확인
npm list                              # 설치된 패키지 목록
npm list --depth=0                    # 최상위 패키지만 표시

# 스크립트 실행
npm run build                         # package.json의 build 스크립트 실행
npm start                             # start 스크립트 실행
npm test                              # test 스크립트 실행

# 캐시 관리
npm cache clean --force               # 캐시 삭제
npm cache verify                      # 캐시 검증

# 레지스트리 관리
npm config set registry https://...   # 레지스트리 변경
npm config get registry               # 현재 레지스트리 확인
npm login                             # npm 계정 로그인
npm publish                           # 패키지 게시
```

#### package.json 구조

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My awesome app",

  // 의존성
  "dependencies": {
    "express": "^4.18.0", // ^: 최신 마이너 버전까지
    "lodash": "~4.17.0", // ~: 최신 패치 버전까지
    "react": "18.0.0" // 정확한 버전
  },

  // 개발 환경 의존성 (프로덕션에 포함 안 됨)
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "@types/node": "^18.0.0"
  },

  // 스크립트 정의
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "build": "webpack",
    "test": "jest",
    "lint": "eslint ."
  },

  // 기타 메타데이터
  "author": "John Doe",
  "license": "MIT",
  "keywords": ["app", "awesome"],
  "homepage": "https://github.com/user/repo"
}
```

#### 버전 지정 규칙 (Semantic Versioning)

```
Version: MAJOR.MINOR.PATCH
예: 4.18.2

MAJOR: 호환되지 않는 변경
MINOR: 하위 호환 새 기능
PATCH: 하위 호환 버그 수정

버전 지정 방식:
4.18.2          # 정확히 4.18.2
^4.18.2         # >=4.18.2, <5.0.0 (마이너까지 자동 업데이트)
~4.18.2         # >=4.18.2, <4.19.0 (패치까지 자동 업데이트)
>=4.18.0        # 4.18.0 이상
<=4.20.0        # 4.20.0 이하
4.18.0 - 4.20.0 # 범위 지정
*               # 모든 버전
latest          # 최신 버전
```

### 2. Yarn

#### 개요

- 공식 사이트: https://yarnpkg.com/
- GitHub: https://github.com/yarnpkg/berry
- 문서: https://yarnpkg.com/getting-started
- 설치: npm install -g yarn
- 버전: Yarn 1.x (Classic), Yarn 3.x+ (Berry - 최신)

**특징:**

- Facebook(Meta)에서 만든 npm 대체 도구
- npm보다 빠른 설치 속도
- workspace 기능으로 모노레포 지원
- 더 엄격한 버전 관리 (lockfile)

#### 기본 명령어

```bash
# yarn 버전 확인
yarn --version

# 패키지 설치
yarn add express                      # 최신 버전 설치
yarn add express@4.18.0              # 특정 버전 설치
yarn install                         # package.json 기반 설치

# 개발 환경 전용 설치
yarn add --dev prettier              # 또는 yarn add -D prettier

# 전역 설치
yarn global add @angular/cli

# 패키지 제거
yarn remove express                  # 패키지 제거
yarn remove -D prettier              # 개발 의존성 제거

# 버전 업데이트
yarn upgrade express                 # 업데이트
yarn upgrade                         # 모든 패키지 업데이트
yarn up                              # 최신 버전으로 업데이트 (Yarn 3+)

# 버전 관리
yarn outdated                        # 업데이트 가능한 패키지 확인
yarn info express                    # 패키지 정보 확인
yarn why express                     # 왜 이 패키지가 필요한지 확인

# 스크립트 실행
yarn build                           # build 스크립트 실행
yarn start                           # start 스크립트 실행
yarn test                            # test 스크립트 실행

# workspace (모노레포)
yarn workspaces list                 # workspace 목록
yarn workspace <name> add express    # 특정 workspace에 패키지 추가

# 캐시 관리
yarn cache clean                     # 캐시 삭제
yarn cache list                      # 캐시된 패키지 확인
```

#### yarn.lock vs package-lock.json

```
npm (package-lock.json):
- 의존성의 정확한 버전 기록
- 계층적 구조 (깊음)
- 파일 크기 커짐

Yarn (yarn.lock):
- 더 빠른 설치
- 평탄한 구조 (flat node_modules)
- 더 나은 보안 검사
```

### 3. pnpm (performant npm)

#### 개요

- 공식 사이트: https://pnpm.io/
- GitHub: https://github.com/pnpm/pnpm
- 문서: https://pnpm.io/motivation
- 설치: npm install -g pnpm
- 특징: 가장 빠르고 효율적

**특징:**

- 가장 빠른 패키지 매니저
- 디스크 공간 70% 절약 (심볼릭 링크 사용)
- npm과 호환되는 명령어
- 모노레포 네이티브 지원

#### 기본 명령어

```bash
# pnpm 버전 확인
pnpm --version

# 패키지 설치
pnpm add express                     # 최신 버전 설치
pnpm add express@4.18.0             # 특정 버전 설치
pnpm install                        # package.json 기반 설치
pnpm i                              # 축약형

# 개발 환경 전용 설치
pnpm add -D prettier                # 개발 의존성 설치

# 전역 설치
pnpm add -g @angular/cli

# 패키지 제거
pnpm remove express                 # 패키지 제거
pnpm rm express                     # 축약형

# 버전 업데이트
pnpm update                         # 모든 패키지 업데이트
pnpm up                             # 최신 버전으로 업데이트
pnpm up --latest                    # 정말 최신 버전으로

# 버전 관리
pnpm outdated                       # 업데이트 가능한 패키지 확인
pnpm list                           # 설치된 패키지 목록
pnpm why express                    # 의존성 경로 확인

# 스크립트 실행
pnpm build                          # build 스크립트 실행
pnpm start                          # start 스크립트 실행
pnpm test                           # test 스크립트 실행

# 모노레포 (workspace)
pnpm -r install                     # 모든 workspace 설치
pnpm -r build                       # 모든 workspace 빌드
pnpm --filter <package> add express # 특정 package에 추가

# 저장소 (monorepo) 관리
pnpm install --frozen-lockfile      # CI/CD: 정확한 버전 설치
pnpm prune --prod                   # 개발 의존성 제거
```

#### pnpm 장점

```
메모리 사용:
npm (node_modules):  1GB
Yarn (node_modules): 800MB
pnpm (node_modules): 300MB  ← 70% 절약!

설치 속도:
npm:   30초
Yarn:  20초
pnpm:  10초  ← 3배 빠름!
```

### 4. Bun

#### 개요

- 공식 사이트: https://bun.sh/
- GitHub: https://github.com/oven-sh/bun
- 문서: https://bun.sh/docs
- 설치: curl -fsSL https://bun.sh/install | bash
- 특징: 차세대 JavaScript 런타임의 패키지 매니저

**특징:**

- Zig로 만들어진 초고속 런타임
- npm 호환 명령어
- npm 5배 빠른 설치 속도
- TypeScript 네이티브 지원
- 번들러, 테스트 러너 내장

#### 기본 명령어

```bash
# Bun 버전 확인
bun --version

# 패키지 설치
bun add express                      # 설치
bun add express@4.18.0              # 특정 버전
bun install                         # package.json 기반

# 개발 환경
bun add -D prettier                 # 개발 의존성

# 전역 설치
bun add -g @angular/cli

# 패키지 제거
bun remove express

# 업데이트
bun update                          # 모든 패키지 업데이트

# 스크립트 실행
bun run build
bun run start
bun run test

# 파일 실행 (Node.js 대체)
bun src/index.ts                    # TypeScript 직접 실행
bun run src/index.js                # JavaScript 실행

# 번들링
bun build --target bun src/index.ts

# 테스트
bun test
```

### 5. pip (Python)

#### 개요

- 공식 사이트: https://pypi.org/
- GitHub: https://github.com/pypa/pip
- 문서: https://pip.pypa.io/
- 설치: Python과 함께 자동 설치
- 대표 저장소: PyPI (Python Package Index)

**특징:**

- Python의 표준 패키지 매니저
- PyPI에 수백만 개 패키지 존재
- 가상 환경 지원 (venv)
- requirements.txt로 의존성 관리

#### 기본 명령어

```bash
# pip 버전 확인
pip --version
pip3 --version

# 패키지 설치
pip install django                  # 최신 버전
pip install django==4.2.0          # 특정 버전
pip install -r requirements.txt     # 파일 기반 설치

# 개발 환경
pip install django[dev]             # 선택적 의존성

# 패키지 제거
pip uninstall django

# 버전 업데이트
pip install --upgrade django
pip install -U django

# 버전 관리
pip list                           # 설치된 패키지 목록
pip outdated                       # 업데이트 가능한 패키지
pip show django                    # 패키지 정보

# requirements.txt 생성
pip freeze > requirements.txt      # 현재 환경 저장

# 가상 환경
python -m venv myenv              # 가상 환경 생성
source myenv/bin/activate         # 활성화 (macOS/Linux)
myenv\Scripts\activate            # 활성화 (Windows)
deactivate                        # 비활성화
```

#### requirements.txt 형식

```
# 버전 지정
django==4.2.0
requests>=2.28.0
flask~=2.0

# 주석
# 이것은 주석입니다

# 파일 참조
-r requirements-dev.txt

# 로컬 패키지
./my-local-package
```

### 6. Gradle (Java/Kotlin)

#### 개요

- 공식 사이트: https://gradle.org/
- GitHub: https://github.com/gradle/gradle
- 문서: https://docs.gradle.org/
- 설치: gradlew (gradle wrapper) 자동 설치
- 저장소: Maven Central, Gradle Central

**특징:**

- Java/Kotlin 표준 빌드 도구
- Maven보다 빠르고 유연함
- Kotlin DSL 지원
- 의존성 관리 통합

#### 기본 명령어

```bash
# Gradle 버전 확인
gradle --version
./gradlew --version

# 빌드
gradle build                       # 빌드
./gradlew build                   # wrapper 사용

# 패키지 설치 (build.gradle에서 정의)
gradle dependencies               # 의존성 확인

# 테스트
gradle test

# 정리
gradle clean

# 특정 태스크
gradle run                        # 실행
gradle assemble                  # 아티팩트 빌드
```

#### build.gradle (Groovy 형식)

```gradle
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:3.0.0'
    implementation 'com.google.guava:guava:31.1-jre'

    testImplementation 'junit:junit:4.13.2'
}
```

### 7. Cargo (Rust)

#### 개요

```
공식 사이트: https://doc.rust-lang.org/cargo/
GitHub: https://github.com/rust-lang/cargo
문서: https://doc.rust-lang.org/cargo/
설치: Rust 설치 시 함께 설치
저장소: crates.io (crates.io)
```

**특징:**

- Rust의 기본 패키지 매니저
- 빌드 도구 통합
- 매우 안전한 의존성 관리
- crates.io에 수십만 크레이트

#### 기본 명령어

```bash
# Cargo 버전 확인
cargo --version

# 새 프로젝트
cargo new my-project
cargo new --lib my-lib

# 패키지 추가
cargo add serde                  # 의존성 추가
cargo add --dev tokio          # 개발 의존성

# 제거
cargo remove serde

# 빌드
cargo build                     # 디버그 빌드
cargo build --release          # 릴리스 빌드

# 실행
cargo run                       # 빌드 및 실행
cargo run -- arg1 arg2         # 인자 전달

# 테스트
cargo test

# 문서
cargo doc --open               # 문서 생성 및 열기

# 배포
cargo publish                  # crates.io에 게시
```

#### Cargo.toml

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
criterion = "0.4"
```

## 패키지 매니저 비교표

| 매니저     | 언어       | 속도      | 점유율 | 저장소       | 명령어      |
| ---------- | ---------- | --------- | ------ | ------------ | ----------- |
| **npm**    | JavaScript | 중간      | 95%    | npmjs.com    | npm i       |
| **Yarn**   | JavaScript | 빠름      | 10%    | npmjs.com    | yarn add    |
| **pnpm**   | JavaScript | 가장 빠름 | 15%    | npmjs.com    | pnpm add    |
| **Bun**    | JavaScript | 초고속    | <1%    | npmjs.com    | bun add     |
| **pip**    | Python     | 중간      | 100%   | pypi.org     | pip install |
| **Gradle** | Java       | 빠름      | 40%    | gradle/maven | gradle add  |
| **Cargo**  | Rust       | 빠름      | 100%   | crates.io    | cargo add   |

## 고급 주제

### 1. 패키지 충돌 해결

```bash
# npm에서 같은 패키지의 여러 버전이 필요한 경우
npm ls express
npm dedupe              # 중복 제거 시도

# pnpm은 자동으로 효율적으로 관리
pnpm why express      # 왜 이 버전이 필요한지 확인
```

### 2. 프라이빗 패키지 저장소

```bash
# npm 프라이빗 레지스트리
npm config set registry https://private-registry.com
npm login --registry https://private-registry.com

# GitHub Packages 사용
npm set //npm.pkg.github.com/:_authToken TOKEN
npm install @username/private-package

# Artifactory, Nexus 등 사용 가능
```

### 3. 의존성 보안 검사

```bash
# npm 보안 감시
npm audit                 # 취약점 스캔
npm audit fix             # 자동 수정
npm audit fix --force     # 강제 수정

# Yarn
yarn audit               # 취약점 확인
yarn audit --fix         # 자동 수정

# Snyk (전문 보안 도구)
npm install -g snyk
snyk test               # 취약점 테스트
snyk fix                # 취약점 수정
```

### 4. 모노레포 관리

```bash
# npm workspace (7.0+)
npm install -w packages/core

# Yarn workspace
yarn workspace @monorepo/core add express

# pnpm workspace (권장)
pnpm -r install
pnpm --filter core add express
```

## 패키지 매니저 선택 가이드

### 각 팀이 선택해야 할 매니저

**JavaScript 프로젝트**

**npm 선택:**

- 작은 팀/프로젝트
- 기본 설정만 필요
- 모든 팀원이 npm에 익숙

**Yarn 선택:**

- workspace 기능 필요
- Facebook 생태계 사용
- 엄격한 버전 관리 필요

**pnpm 선택 (권장):**

- 대규모 모노레포
- 디스크 공간 절약 필요
- 설치 속도 중요
- npm 호환성 필요

**Bun 선택:**

- TypeScript 네이티브 사용
- 초고속 개발 원함
- 런타임도 Bun 사용 가능

## 버전 관리 best practices

1. lock file 반드시 커밋
   - package-lock.json (npm)
   - yarn.lock (Yarn)
   - pnpm-lock.yaml (pnpm)

2. package.json은 최소 요구사항
   - 정확한 버전은 lock file에서 관리

3. CI/CD에서

   ```bash
   npm ci              # npm
   yarn install --frozen-lockfile  # Yarn
   pnpm install --frozen-lockfile  # pnpm
   ```

4. 정기적 업데이트
   ```bash
   npm outdated
   npm update
   ```

## 주의사항

### 의존성 지옥

```
10개 패키지 설치
├─ 각 패키지는 평균 5개 의존성
├─ 각 의존성의 의존성 또 5개
└─ 총 node_modules: 1,000개+!

해결책:
- pnpm으로 중복 제거
- npm ls로 의존성 확인
- yarn why로 필요성 파악
```

### 보안

```
npm audit를 정기적으로 실행
특히 프로덕션 배포 전에:

npm audit --production
npm audit fix
npm audit fix --force (주의!)
```

### 성능

```
각 언어별 최적 선택:

JavaScript:
- 개발: pnpm (가장 빠름)
- 프로덕션: npm (가장 안정적)
- 대규모: pnpm + workspace

Python:
- pip + venv (기본)
- Poetry (모던, 권장)

Java:
- Gradle (모던, 권장)
- Maven (전통적)
```

## 공식 사이트 총정리

### JavaScript/Node.js

| 도구 | 공식 사이트 | 패키지 저장소 | 문서             |
| ---- | ----------- | ------------- | ---------------- |
| npm  | npmjs.com   | npmjs.com     | docs.npmjs.com   |
| Yarn | yarnpkg.com | npmjs.com     | yarnpkg.com/docs |
| pnpm | pnpm.io     | npmjs.com     | pnpm.io/docs     |
| Bun  | bun.sh      | npmjs.com     | bun.sh/docs      |

### Python

| 도구   | 공식 사이트       | 패키지 저장소 | 문서                   |
| ------ | ----------------- | ------------- | ---------------------- |
| pip    | pip.pypa.io       | pypi.org      | pip.pypa.io            |
| Poetry | python-poetry.org | pypi.org      | python-poetry.org/docs |
| pipenv | pipenv.pypa.io    | pypi.org      | pipenv.pypa.io         |

### Java

| 도구   | 공식 사이트      | 패키지 저장소     | 문서                    |
| ------ | ---------------- | ----------------- | ----------------------- |
| Maven  | maven.apache.org | mvnrepository.com | maven.apache.org/guides |
| Gradle | gradle.org       | mvnrepository.com | gradle.org/docs         |

### 기타 언어

| 언어 | 도구     | 공식 사이트             | 패키지 저장소 |
| ---- | -------- | ----------------------- | ------------- |
| PHP  | Composer | getcomposer.org         | packagist.org |
| Rust | Cargo    | doc.rust-lang.org/cargo | crates.io     |
| Ruby | Bundler  | bundler.io              | rubygems.org  |
| Go   | go mod   | golang.org              | pkg.go.dev    |

## 결론

### 각 언어별 추천

```

JavaScript:

- 새 프로젝트: pnpm
- 기존 프로젝트: npm
- 대규모 모노레포: pnpm workspace
- 초고속 필요: Bun

Python:

- 기본: pip + venv
- 모던: Poetry
- 데이터 과학: Conda

Java:

- 새 프로젝트: Gradle
- 기존 프로젝트: Maven
- Spring Boot: Gradle

PHP:

- Laravel/Symfony: Composer

Rust:

- 모든 프로젝트: Cargo

```

### 핵심 원칙

1. **팀에서 하나로 통일**
   - 여러 매니저 혼용 금지

2. **lock file 필수**
   - 모든 팀원이 동일 버전 사용

3. **정기적 업데이트**
   - 보안 패치는 즉시 적용
   - 주요 버전 업그레이드는 계획적으로

4. **보안 감시**
   - CI/CD에서 audit 실행
   - 취약점 자동 감지

패키지 매니저를 올바르게 사용하면, 의존성 관리의 번거로움을 완전히 제거할 수 있습니다!

## 참고 링크

- [npm 공식 문서](https://docs.npmjs.com/)
- [pnpm 공식 사이트](https://pnpm.io/)
- [Yarn 공식 사이트](https://yarnpkg.com/)
- [Bun 공식 문서](https://bun.sh/docs)
