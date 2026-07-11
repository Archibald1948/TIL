# Husky - Git 훅으로 자동화된 코드 품질 관리

## 들어가며

당신이 실수로 버그가 있는 코드를 커밋하거나, 스타일 가이드를 따르지 않은 코드를 메인 브랜치에 푸시했던 경험이 있나요?
Husky는 이런 인간의 실수를 완전히 자동화된 방식으로 방지합니다.

Husky는 Git 훅(hook)을 관리하는 도구입니다.
개발자가 코드를 커밋하거나 푸시하기 전에 자동으로 검증 작업을 수행하므로, 문제 있는 코드가 저장소에 들어가는 것을 원천적으로 차단합니다.

## 공식 사이트

https://github.com/typicode/husky

## Git 훅이란?

### 기본 개념

Git 훅은 특정 Git 이벤트가 발생할 때 자동으로 실행되는 스크립트입니다.
`.git/hooks` 디렉토리에 저장되며, Git이 특정 작업을 수행하려고 할 때 이 스크립트들을 실행합니다.

### 주요 Git 훅

```
pre-commit      : 커밋 직전 (가장 자주 사용)
prepare-commit-msg : 커밋 메시지 에디터 직전
commit-msg      : 커밋 메시지 작성 후
post-commit     : 커밋 완료 후
pre-push        : 푸시 직전
post-push       : 푸시 완료 후
pre-rebase      : 리베이스 직전
post-rebase     : 리베이스 완료 후
pre-merge-commit: 병합 직전
post-merge      : 병합 완료 후
```

### Git 훅의 한계

Git 훅은 원래 강력한 기능이지만, 몇 가지 문제가 있습니다:

1. **.git/hooks는 버전 관리되지 않음**: 훅 스크립트를 작성해도 저장소에 커밋할 수 없으므로, 팀원들이 자동으로 받지 못합니다.

2. **설정이 복잡함**: 수동으로 훅 스크립트를 작성하고 설정해야 합니다.

3. **크로스 플랫폼 문제**: Windows와 macOS에서 다르게 동작할 수 있습니다.

4. **유지보수 어려움**: 팀원들이 훅을 설정했는지 확인하기 어렵습니다.

**Husky가 이 모든 문제를 해결합니다.**

## Husky 설치 및 기본 설정

### 설치

```bash
# npm 프로젝트에서
npm install husky --save-dev

# 초기화
npx husky install
```

설치 후 프로젝트 구조:

```
project/
├── .husky/           # 새로 생성됨
│   ├── _/
│   │   └── husky.sh  # Husky 실행 스크립트
│   └── .gitignore
├── package.json
└── node_modules/
    └── husky/
```

### package.json 설정

```json
{
  "scripts": {
    "prepare": "husky install"
  },
  "devDependencies": {
    "husky": "^8.0.0",
    "lint-staged": "^14.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  }
}
```

`prepare` 스크립트는 `npm install` 실행 후 자동으로 실행되어, 팀원들이 훅을 자동으로 받을 수 있게 합니다.

## 실전: Pre-commit 훅 설정

### 1. 린트 + 포매팅 자동 수정

가장 기본적이고 중요한 훅입니다.

```bash
# pre-commit 훅 생성
npx husky add .husky/pre-commit "npx lint-staged"
```

`.husky/pre-commit` 파일이 생성됩니다:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

### 2. lint-staged 설정

`lint-staged`는 커밋할 파일들만 선택적으로 린트와 포매팅을 수행합니다.

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,css,scss}": ["prettier --write"],
    "*.md": ["prettier --write"]
  }
}
```

### 실제 작동 예시

```bash
# 파일 수정 후 스테이징
$ git add src/App.tsx src/utils.js

# 커밋 시도
$ git commit -m "feat: add new feature"

# Husky가 자동으로 실행됨
husky > pre-commit (node_modules/.bin/husky)
...
✔ Preparing lint-staged...
✔ Running tasks...
✔ Applying modifications...
✔ Clean up...

# 파일들이 자동으로 수정되고 커밋 진행
[main abc1234] feat: add new feature
 2 files changed, 15 insertions(+), 5 deletions(-)
```

문제가 있으면:

```bash
$ git commit -m "feat: add feature"

✖ Running tasks...
✖ "src/App.tsx" - eslint found issues

# 커밋이 실패하고, ESLint 에러를 표시
```

## Pre-push 훅 설정

푸시 직전에 전체 테스트를 실행하여, 테스트를 통과하지 못한 코드가 원격 저장소에 올라가는 것을 방지합니다.

```bash
npx husky add .husky/pre-push "npm run test"
```

`.husky/pre-push`:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run test
```

### 더 정교한 pre-push 훅

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "Running tests before push..."
npm run test

if [ $? -ne 0 ]; then
  echo "❌ Tests failed! Push aborted."
  exit 1
fi

echo "Running type check..."
npm run type-check

if [ $? -ne 0 ]; then
  echo "❌ Type check failed! Push aborted."
  exit 1
fi

echo "✅ All checks passed! Proceeding with push..."
exit 0
```

## Commit-msg 훅 설정

커밋 메시지 형식을 강제하여, 일관된 커밋 히스토리를 유지합니다.

### 설치

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

### 설정

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat', // 새로운 기능
        'fix', // 버그 수정
        'docs', // 문서 수정
        'style', // 코드 스타일 변경 (기능 변화 없음)
        'refactor', // 코드 리팩토링
        'perf', // 성능 개선
        'test', // 테스트 추가
        'chore', // 빌드, 패키지 관리 등
        'ci', // CI/CD 설정 변경
      ],
    ],
    'type-case': [2, 'always', 'lower-case'],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'subject-case': [2, 'always', 'lower-case'],
    'scope-case': [2, 'always', 'lower-case'],
  },
};
```

### 훅 추가

```bash
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### 작동 예시

```bash
# 올바른 형식
$ git commit -m "feat: add user authentication"
✅ Commit message validated successfully

# 잘못된 형식
$ git commit -m "added some stuff"
❌ subject should not be empty
❌ type must be one of [feat, fix, docs, ...]

# 커밋 실패
```

## 프로덕션 레벨 설정: 완전한 구현

### 1. 완벽한 package.json 설정

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "prepare": "husky install",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "npm run lint -- --fix",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:coverage": "vitest --coverage"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,css,scss,md}": ["prettier --write"]
  },
  "devDependencies": {
    "@commitlint/cli": "^17.0.0",
    "@commitlint/config-conventional": "^17.0.0",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "eslint": "^8.0.0",
    "eslint-config-prettier": "^9.0.0",
    "eslint-plugin-import": "^2.0.0",
    "eslint-plugin-react": "^7.0.0",
    "eslint-plugin-react-hooks": "^4.0.0",
    "husky": "^8.0.0",
    "lint-staged": "^14.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

### 2. 모든 훅 설정

```bash
# pre-commit 훅: 린트 + 포매팅
npx husky add .husky/pre-commit "npx lint-staged"

# pre-push 훅: 테스트 + 타입 체크
npx husky add .husky/pre-push "npm run test && npm run type-check"

# commit-msg 훅: 커밋 메시지 검증
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### 3. .husky/pre-commit (개선 버전)

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Checking code quality..."

npx lint-staged

if [ $? -ne 0 ]; then
  echo "❌ Lint-staged failed! Commit aborted."
  exit 1
fi

echo "✅ Code quality checks passed!"
exit 0
```

### 4. .husky/pre-push (개선 버전)

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🧪 Running tests..."
npm run test

if [ $? -ne 0 ]; then
  echo "❌ Tests failed! Push aborted."
  exit 1
fi

echo "📝 Checking types..."
npm run type-check

if [ $? -ne 0 ]; then
  echo "❌ Type check failed! Push aborted."
  exit 1
fi

echo "✅ All checks passed! Ready to push."
exit 0
```

### 5. .husky/prepare-commit-msg

커밋 메시지에 자동으로 브랜치 이름을 추가할 수 있습니다.

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

BRANCH=$(git symbolic-ref --short HEAD)
ISSUE_ID=$(echo $BRANCH | grep -o '[A-Z]*-[0-9]*')

if [ -n "$ISSUE_ID" ]; then
  # 예: "PROJ-123: add new feature"
  sed -i.bak -e "1s/^/$ISSUE_ID: /" "$1"
fi
```

## 팀 협업 가이드

### 협업 규칙 문서화

````markdown
# Git Hooks 및 Pre-commit 가이드

## 요구사항

모든 개발자는 다음을 준수해야 합니다:

### 1. 초기 설정

```bash
git clone <repository>
npm install  # 자동으로 husky install 실행됨
```
````

### 2. Pre-commit 자동 검증

커밋 직전에 다음이 자동으로 실행됩니다:

- ESLint: 코드 스타일 검증 및 자동 수정
- Prettier: 코드 포매팅 자동 수정
- lint-staged: 커밋할 파일만 검증 (성능 향상)

**만약 lint-staged가 실패하면:**

1. 에러 메시지를 읽고 문제 파악
2. 문제 수정
3. `git add` 재실행
4. `git commit` 재시도

### 3. Commit Message 규칙

Conventional Commits 형식을 따릅니다:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type (필수):**

- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 수정
- `style`: 코드 포매팅 (기능 변화 없음)
- `refactor`: 코드 리팩토링
- `perf`: 성능 개선
- `test`: 테스트 추가
- `chore`: 빌드, 패키지 관리
- `ci`: CI/CD 설정 변경

**Scope (선택):**
기능이나 모듈 이름 (예: auth, user, product)

**Examples:**

```
feat(auth): add user login functionality
fix(user): resolve password reset issue
docs: update API documentation
style(ui): format button component
refactor(api): simplify error handling
perf(database): optimize query performance
test(utils): add unit tests for validators
chore(deps): update dependencies
```

### 4. Pre-push 검증

푸시 직전에 다음이 자동으로 실행됩니다:

- 전체 테스트 스위트 (`npm run test`)
- TypeScript 타입 검증 (`npm run type-check`)

**테스트 실패 시:**

```bash
# 로컬에서 테스트 실행하여 확인
npm run test

# 실패한 테스트 수정
# 다시 푸시 시도
git push
```

### 5. 훅 우회 (⚠️ 매우 신중하게)

특수한 경우에만 훅을 우회할 수 있습니다:

```bash
# pre-commit 우회
git commit --no-verify -m "message"

# pre-push 우회
git push --no-verify
```

**주의:** `--no-verify`를 사용한 커밋은 코드 리뷰에서 지적될 것입니다.

### 6. 개발자 책임

- Husky가 요구하는 모든 검증을 존중하세요
- 자동 수정이 마음에 들지 않으면 설정을 변경 요청하세요
- 훅 우회는 긴급 상황에만 사용하세요

### 7. 팀 리드 책임

- Husky 설정을 정기적으로 검토하세요
- ESLint, Prettier 규칙을 팀과 함께 논의하세요
- 훅 설정 변경이 있으면 즉시 공지하세요

## FAQ

**Q: lint-staged가 실패했어요. 뭘 해야 하나요?**
A: 에러 메시지를 읽고 지정된 파일을 수정한 후 다시 커밋하세요.

**Q: 특정 파일을 린트 검사에서 제외하고 싶어요.**
A: .eslintignore와 .prettierignore 파일을 편집하세요.

**Q: 팀원이 Husky 없이 커밋했어요.**
A: 그 팀원은 `npm install` 후 `npx husky install`을 실행하지 않았습니다. 다시 실행하도록 안내하세요.

**Q: CI/CD에서도 같은 검증을 해야 하나요?**
A: 네. CI/CD 파이프라인에서도 동일한 lint, test, type-check를 실행하여 이중 검증합니다.

````

## Husky와 다른 도구의 통합

### ESLint와의 통합

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'prettier' // 반드시 마지막에
  ],
  plugins: ['@typescript-eslint', 'react', 'react-hooks'],
  rules: {
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'no-console': ['warn', { allow: ['warn', 'error'] }]
  }
};
````

### Prettier와의 통합

```javascript
// .prettierrc.js
module.exports = {
  semi: true,
  singleQuote: true,
  tabWidth: 2,
  trailingComma: 'es5',
  printWidth: 100,
  bracketSpacing: true,
};
```

### GitHub Actions와의 통합

```yaml
# .github/workflows/lint.yml
name: Lint & Test

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run tests
        run: npm run test

      - name: Check types
        run: npm run type-check
```

## 성능 최적화

### 1. 큰 파일 제외하기

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,css,scss,md}": ["prettier --write"],
    "!*.min.js": [] // 최소화된 파일 제외
  }
}
```

### 2. 캐싱 활성화

```json
{
  "lint-staged": {
    "*.js": ["eslint --fix --cache", "prettier --write"]
  }
}
```

### 3. 병렬 실행

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint:fix &
npm run format &
wait
```

## 결론

Husky는 단순한 도구가 아니라, 팀의 코드 품질을 지키는 자동화된 파수꾼입니다. 올바르게 설정하면:

- 버그를 커밋 직전에 잡을 수 있습니다
- 코드 스타일 관련 리뷰 피드백을 줄일 수 있습니다
- 테스트를 통과하지 못한 코드가 푸시되는 것을 방지합니다
- 일관된 커밋 메시지로 히스토리 관리가 쉬워집니다

특히 팀 프로젝트에서는 Husky 없이는 일관된 코드 품질을 유지하기 거의 불가능합니다. 지금 바로 도입해보세요!

## 참고 링크

- [Husky 공식 문서](https://typicode.github.io/husky)
- [Husky GitHub](https://github.com/typicode/husky)
- [lint-staged GitHub](https://github.com/lint-staged/lint-staged)
