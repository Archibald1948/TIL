# Lefthook - Husky보다 빠르고 강력한 Git 훅 관리

## 들어가며

Husky를 사용 중인데 pre-commit 훅이 실행될 때마다 5-10초 기다린다면? Node.js 오버헤드 때문입니다.
Lefthook은 Go로 작성되어 100배 더 빠릅니다.

Go 바이너리로 빌드된 Lefthook은 npm 패키지 설치 없이도 작동하며, JavaScript뿐 아니라 Python, Ruby, Java 등 모든 프로젝트에서 사용할 수 있습니다.
특히 대규모 모노레포에서 Lefthook의 병렬 실행 기능은 개발 속도를 획기적으로 향상시킵니다.

## 공식 사이트

https://github.com/evilmartians/lefthook

## Husky vs Lefthook

### 성능 비교

```
상황: 20개 파일에 lint + format 실행

Husky:
├── Node.js 시작 시간: 2-3초
├── ESLint 실행: 3-4초
├── Prettier 실행: 1-2초
└── 총 시간: 6-9초

Lefthook (순차 실행):
├── Lefthook 시작: 0.1초
├── ESLint 실행: 3-4초
├── Prettier 실행: 1-2초
└── 총 시간: 4-6초

Lefthook (병렬 실행):
├── Lefthook 시작: 0.1초
├── ESLint + Prettier 동시 실행: 4초
└── 총 시간: 4초
```

**결과:** Lefthook이 최대 2배 빠릅니다.

### 기능 비교

| 기능           | Husky             | Lefthook     |
| -------------- | ----------------- | ------------ |
| Git 훅 관리    | ✅                | ✅           |
| 병렬 실행      | ❌                | ✅           |
| 조건부 실행    | ❌                | ✅           |
| 다중 언어 지원 | npm만             | 모든 언어    |
| 성능           | 중간              | 최고         |
| 학습곡선       | 쉬움              | 중간         |
| 설정 파일      | package.json, .js | lefthook.yml |
| 크로스 플랫폼  | ✅                | ✅           |

**결론:** 성능이 중요하거나 다중 언어 프로젝트라면 Lefthook을 선택하세요.

## Lefthook 설치

### 1. macOS/Linux

```bash
# Homebrew
brew install lefthook

# 또는 직접 설치
curl https://raw.githubusercontent.com/evilmartians/lefthook/master/install.sh | bash
```

### 2. Windows

```powershell
# Scoop
scoop install lefthook

# 또는 Chocolatey
choco install lefthook
```

### 3. npm으로 설치 (모든 플랫폼)

```bash
npm install lefthook --save-dev
```

### 4. 설치 확인

```bash
lefthook version
# lefthook version 1.5.0
```

## 기본 설정

### 1. lefthook.yml 생성

프로젝트 루트에 `lefthook.yml` 파일을 생성합니다.

```yaml
# lefthook.yml - 기본 설정

version: 2

# 전역 설정
global:
  # 실패한 명령이 있어도 다른 명령 계속 실행
  fail: fast
  # 작은 폴더 내에 Git 저장소가 있을 때도 적용
  single: false
  # 출력 스타일 (text, json)
  output: text

# 명령어 정의 (모든 훅에서 재사용 가능)
commands:
  lint:
    glob: '*.js'
    run: eslint {staged_files}
  format:
    glob: '*.js'
    run: prettier --write {staged_files}
  test:
    glob: '*.test.js'
    run: npm test -- {staged_files}

# Pre-commit 훅 설정
pre-commit:
  parallel: true # 병렬 실행
  commands:
    lint:
      glob: '*.js'
      run: eslint {staged_files}
    format:
      glob: '*.js'
      run: prettier --write {staged_files}

# Pre-push 훅 설정
pre-push:
  commands:
    test:
      run: npm test

# Commit-msg 훅 설정
commit-msg:
  commands:
    commitlint:
      run: npx commitlint --edit {1}
```

### 2. Lefthook 초기화

```bash
lefthook install
```

이 명령은:

1. 모든 훅을 `.git/hooks` 디렉토리에 설치합니다
2. 기존 훅은 백업합니다
3. lefthook.yml을 Git에 커밋하도록 지시합니다

## 프로덕션 레벨 설정

### 1. 완벽한 lefthook.yml

```yaml
version: 2

# 전역 설정
global:
  # 실패 시 즉시 중단
  fail: fast
  # 최소 Node 버전
  min_version: 2.0.0
  # 환경 변수 설정
  env:
    NODE_ENV: 'development'
  # 실행 시간 초과 (초 단위)
  timeout: 300

# 모든 훅에서 사용할 수 있는 명령어 정의
commands:
  eslint:
    glob: '*.{js,jsx,ts,tsx}'
    run: eslint --fix {staged_files}
    description: 'ESLint을 실행하여 코드 품질 검사'

  prettier:
    glob: '*.{js,jsx,ts,tsx,json,css,scss,md}'
    run: prettier --write {staged_files}
    description: 'Prettier로 코드 포매팅'

  tsc:
    glob: '*.{ts,tsx}'
    run: tsc --noEmit
    description: 'TypeScript 타입 검사'

  test:
    glob: '*.test.{js,jsx,ts,tsx}'
    run: npm test -- --bail {staged_files}
    description: '테스트 실행'

  jest:
    glob: '**/__tests__/**/*.{js,jsx,ts,tsx}'
    run: npx jest {staged_files} --coverage
    description: 'Jest 테스트 실행'

# Pre-commit: 파일 수정 및 검증
pre-commit:
  # 병렬 실행 활성화
  parallel: true

  # 병렬 실행 시 최대 워커 수
  max_workers: 4

  # 실행 순서 (병렬 실행이 아닌 경우)
  # 순차적으로 실행됨
  skip:
    - merge
    - rebase

  # 커맨드 정의
  commands:
    # 1단계: 린트 + 포매팅
    lint-and-format:
      glob: '*.{js,jsx,ts,tsx}'
      run: eslint --fix {staged_files} && prettier --write {staged_files}
      stage_fix: true # 수정된 파일을 자동으로 스테이징
      description: 'ESLint + Prettier 실행'

    # 2단계: TypeScript 타입 검사
    type-check:
      glob: '*.{ts,tsx}'
      run: tsc --noEmit
      description: 'TypeScript 타입 검사'

    # 3단계: JSON 검증
    validate-json:
      glob: '*.json'
      run: node -c {staged_files}
      description: 'JSON 파일 검증'

# Pre-push: 원격에 푸시하기 전 검증
pre-push:
  parallel: false # 순차 실행 (테스트가 순서를 보장해야 함)

  skip:
    - merge
    - rebase

  commands:
    # 전체 테스트 스위트 실행
    test:
      run: npm test
      description: '전체 테스트 스위트 실행'

    # 타입 검사
    type-check:
      run: npm run type-check
      description: 'TypeScript 타입 검사'

    # 빌드 검증
    build:
      run: npm run build
      description: '프로덕션 빌드 검증'

# Commit-msg: 커밋 메시지 검증
commit-msg:
  commands:
    commitlint:
      run: npx commitlint --edit {1}
      description: 'Conventional Commits 검증'

# Prepare-commit-msg: 커밋 메시지 템플릿
prepare-commit-msg:
  commands:
    # 브랜치 이름을 커밋 메시지에 추가
    add-issue-id:
      run: |
        BRANCH=$(git symbolic-ref --short HEAD)
        ISSUE_ID=$(echo $BRANCH | grep -o '[A-Z]*-[0-9]*')
        if [ -n "$ISSUE_ID" ]; then
          sed -i.bak -e "1s/^/$ISSUE_ID: /" $1
        fi
      description: '브랜치 이름에서 Issue ID 자동 추가'

# Post-commit: 커밋 후 작업
post-commit:
  commands:
    update-hooks:
      run: lefthook install
      description: '훅이 변경되었으면 갱신'

# Post-merge: 병합 후 작업
post-merge:
  commands:
    install-deps:
      run: npm ci
      description: 'package-lock.json이 변경되었으면 의존성 재설치'
```

### 2. package.json 통합

```json
{
  "scripts": {
    "prepare": "lefthook install",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "npm run lint -- --fix",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "build": "vite build"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0",
    "commitlint": "^17.0.0",
    "@commitlint/config-conventional": "^17.0.0",
    "vitest": "^1.0.0",
    "lefthook": "^1.5.0"
  }
}
```

## 고급 기능

### 1. 조건부 실행

특정 조건에서만 명령을 실행합니다.

```yaml
version: 2

pre-commit:
  commands:
    # 브랜치명이 "main"일 때만 실행
    main-branch-check:
      run: |
        if [ "$(git rev-parse --abbrev-ref HEAD)" = "main" ]; then
          echo "Cannot commit directly to main branch"
          exit 1
        fi
      glob: '*'

    # 파일 크기가 1MB 이상일 때만 경고
    large-file-warning:
      run: |
        for file in {staged_files}; do
          size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file")
          if [ $size -gt 1048576 ]; then
            echo "⚠️  Large file: $file ($(($size / 1048576))MB)"
          fi
        done
      glob: '*'
      fail: false # 실패해도 계속 진행
```

### 2. 동적 글로브 패턴

```yaml
version: 2

pre-commit:
  commands:
    # Python 프로젝트
    python-lint:
      glob: '*.py'
      run: python -m pylint {staged_files}
      description: 'Python 린트'

    # Go 프로젝트
    go-fmt:
      glob: '*.go'
      run: gofmt -w {staged_files}
      description: 'Go 포매팅'

    # Ruby 프로젝트
    ruby-lint:
      glob: '*.rb'
      run: rubocop -a {staged_files}
      description: 'Ruby 린트'

    # 모든 언어 공통
    secrets-scan:
      glob: '*'
      run: |
        if grep -r "api_key\|password\|secret" {staged_files} 2>/dev/null; then
          echo "❌ Secrets detected in staged files"
          exit 1
        fi
      fail: true
      description: '민감한 정보 검사'
```

### 3. 병렬 실행 최적화

```yaml
version: 2

pre-commit:
  parallel: true
  max_workers: 4

  commands:
    # 그룹 1: 린트 & 포매팅 (병렬 실행)
    lint:
      glob: '*.{js,jsx,ts,tsx}'
      run: eslint --fix {staged_files}

    format:
      glob: '*.{js,jsx,ts,tsx}'
      run: prettier --write {staged_files}

    # 그룹 2: 타입 검사 (독립적으로 병렬 실행)
    type-check:
      glob: '*.{ts,tsx}'
      run: tsc --noEmit

    # 그룹 3: JSON 검증 (독립적으로 병렬 실행)
    validate-json:
      glob: '*.json'
      run: jq empty {staged_files}
```

### 4. 환경 변수와 스크립트

```yaml
version: 2

global:
  env:
    # Node 환경
    NODE_ENV: 'development'
    # npm 로그 레벨
    npm_config_loglevel: 'warn'
    # 커스텀 변수
    PROJECT_NAME: 'my-project'

pre-commit:
  commands:
    # 복잡한 스크립트
    custom-check:
      run: |
        #!/bin/bash
        set -e

        echo "🔍 Custom validation started..."

        # 1. 스테이징된 파일 확인
        for file in {staged_files}; do
          echo "Checking $file..."
          
          # 커스텀 검증 로직
          if ! grep -q "©" "$file"; then
            echo "⚠️  Copyright notice missing in $file"
          fi
        done

        echo "✅ Custom validation passed!"
        exit 0
      glob: '*.{js,jsx,ts,tsx}'
      description: '커스텀 검증'
```

## 실제 프로젝트 예시

### 모노레포 구조

```
monorepo/
├── packages/
│   ├── api/
│   ├── web/
│   └── mobile/
├── lefthook.yml
└── package.json
```

```yaml
version: 2

global:
  timeout: 300

# 각 패키지의 훅을 조건부로 실행
commands:
  api-lint:
    glob: 'packages/api/**/*.{ts,tsx}'
    run: cd packages/api && npm run lint:fix

  web-lint:
    glob: 'packages/web/**/*.{tsx,ts}'
    run: cd packages/web && npm run lint:fix

  mobile-lint:
    glob: 'packages/mobile/**/*.{ts,tsx}'
    run: cd packages/mobile && npm run lint:fix

pre-commit:
  parallel: true
  commands:
    api-lint:
      glob: 'packages/api/**/*.{ts,tsx}'
      run: cd packages/api && npm run lint:fix

    web-lint:
      glob: 'packages/web/**/*.{tsx,ts}'
      run: cd packages/web && npm run lint:fix

    mobile-lint:
      glob: 'packages/mobile/**/*.{ts,tsx}'
      run: cd packages/mobile && npm run lint:fix

pre-push:
  commands:
    api-test:
      run: cd packages/api && npm test

    web-test:
      run: cd packages/web && npm test

    mobile-test:
      run: cd packages/mobile && npm test
```

### Python 프로젝트

```yaml
version: 2

pre-commit:
  parallel: true
  commands:
    black:
      glob: '*.py'
      run: black {staged_files}
      description: 'Black으로 포매팅'

    pylint:
      glob: '*.py'
      run: pylint {staged_files}
      description: 'Pylint 코드 품질 검사'

    pytest:
      glob: 'tests/**/*.py'
      run: pytest {staged_files} -v
      description: 'pytest 테스트'

pre-push:
  commands:
    coverage:
      run: pytest --cov=src tests/
      description: '테스트 커버리지 검사'
```

## Husky에서 Lefthook으로 마이그레이션

### 1. 기존 Husky 제거

```bash
# Husky 제거
npm uninstall husky

# 훅 정리
rm -rf .husky
```

### 2. Lefthook 설치

```bash
npm install lefthook --save-dev
lefthook install
```

### 3. 설정 파일 변환

**Husky (package.json):**

```json
{
  "lint-staged": {
    "*.js": "eslint --fix",
    "*.{js,json}": "prettier --write"
  }
}
```

**Lefthook (lefthook.yml):**

```yaml
version: 2

pre-commit:
  commands:
    lint:
      glob: '*.js'
      run: eslint --fix {staged_files}

    format:
      glob: '*.{js,json}'
      run: prettier --write {staged_files}
```

### 4. 테스트

```bash
# 새 파일 생성
echo 'const x=1' > test.js

# 스테이징
git add test.js

# 커밋 (Lefthook이 자동으로 실행됨)
git commit -m "test: test file"
```

## 팀 협업 가이드

````markdown
# Lefthook 설정 가이드

## 초기 설정

```bash
git clone <repo>
npm install  # lefthook install이 자동 실행됨
```
````

## Pre-commit 훅

다음이 자동으로 실행됩니다:

- ESLint + 자동 수정
- Prettier + 자동 포매팅
- TypeScript 타입 검사

문제가 발생하면:

1. 에러 메시지 읽기
2. 파일 수정
3. `git add` 재실행
4. `git commit` 재시도

## Pre-push 훅

다음이 자동으로 실행됩니다:

- 전체 테스트 스위트
- 빌드 검증

테스트 실패 시 로컬에서 수정 후 재시도하세요.

## 훅 우회 (긴급 상황만)

```bash
git commit --no-verify
git push --no-verify
```

## 문제 해결

**Q: "lefthook: command not found"**
A: `npm install`을 실행하세요.

**Q: 훅이 실행되지 않음**
A: `lefthook install`을 실행하세요.

**Q: 특정 파일을 제외하고 싶음**
A: lefthook.yml의 glob 패턴을 수정하세요.

## Lefthook vs Husky 최종 비교

| 관점                   | Husky                   | Lefthook           |
| ---------------------- | ----------------------- | ------------------ |
| **성능**               | 느림 (Node.js 오버헤드) | 빠름 (Go 바이너리) |
| **병렬 실행**          | 없음                    | 있음               |
| **언어 지원**          | npm만                   | 모든 언어          |
| **설정**               | package.json + .js      | lefthook.yml       |
| **학습곡선**           | 쉬움                    | 중간               |
| **커뮤니티**           | 큼                      | 중간               |
| **단순한 프로젝트**    | ✅ 추천                 | 과할 수 있음       |
| **복잡한 프로젝트**    | 적합                    | ✅ 추천            |
| **모노레포**           | 괜찮음                  | ✅ 최고            |
| **다중 언어 프로젝트** | 부족함                  | ✅ 완벽함          |

## 결론

Lefthook은:

1. **성능 중심**: 개발자 경험을 최우선으로
2. **유연성**: YAML 기반 설정으로 복잡한 워크플로우 지원
3. **언어 무관**: 모든 기술 스택에서 사용 가능
4. **현대적**: 병렬 실행, 조건부 실행 등 고급 기능

특히 **다음 경우 Lefthook을 강력히 권장합니다:**

- 팀의 커밋 시간을 줄이고 싶을 때
- Python, Go, Ruby 등 다양한 언어를 사용할 때
- 모노레포 구조를 사용할 때
- 복잡한 전처리 로직이 필요할 때

지금 바로 Lefthook으로 업그레이드하여 개발 속도를 높여보세요!

## 참고 링크

- [Lefthook 공식 문서](https://lefthook.dev)
- [Lefthook GitHub](https://github.com/evilmartians/lefthook)
