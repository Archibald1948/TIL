# ESLint - 코드 품질 검사의 완벽 솔루션

## 들어가며

코드를 작성할 때 이런 문제들이 없나요?

```javascript
// ❌ 나쁜 코드들
var name = "John"  // var 사용
if (name == "John") {  // == 사용
  console.log("Hello")  // console 방치
}

const unused = "이 변수는 사용하지 않음"
function() {  // 함수명 없음
  // ...
}

// 비동기 처리 에러
async function getData() {
  fetch('/api/data')  // await 없음 - 버그!
}
```

**ESLint**는 이런 문제들을 자동으로 찾아서 알려줍니다.

## 공식 사이트

https://eslint.org

# 1. ESLint란?

## 핵심 개념

```
ESLint = 코드의 문제점을 찾고 수정하는 정적 분석 도구

할 수 있는 것:
✅ 버그 찾기 (미사용 변수, 타입 오류 등)
✅ 나쁜 패턴 방지 (var 사용 금지, == 사용 금지)
✅ 코드 스타일 강제 (들여쓰기, 공백 등)
✅ 커스텀 규칙 추가
✅ 자동 수정 기능
```

## ESLint의 장점

```
✅ 버그를 빨리 찾음
✅ 팀 코드 스타일 통일
✅ 나쁜 패턴 사전 방지
✅ 자동 수정으로 시간 절약
✅ 개발 초보자도 좋은 습관 형성
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install -D eslint
npx eslint --init
```

## 자동 설정 선택

```
? How would you like to use ESLint?
→ To check syntax and find problems

? What type of modules does your project use?
→ JavaScript modules (import/export)

? Which framework does your project use?
→ React (또는 None)

? Does your project use TypeScript?
→ Yes/No

? Where does your code run?
→ Browser, Node

? What format do you like for your config file?
→ JavaScript
```

# 3. 기본 설정 파일

## .eslintrc.js (기본)

```javascript
module.exports = {
  // 환경 설정
  env: {
    browser: true, // 브라우저 글로벌 (window, document 등)
    es2021: true, // ES2021 문법 지원
    node: true, // Node.js 글로벌 (require, module 등)
  },

  // 기본 규칙 확장
  extends: [
    'eslint:recommended', // ESLint 권장 규칙
  ],

  // 파서 설정
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },

  // 규칙들
  rules: {
    'no-var': 'error', // var 사용 금지
    'prefer-const': 'error', // const 사용 강제
    eqeqeq: 'error', // === 사용 강제
    'no-console': 'warn', // console 사용 경고
    'no-debugger': 'error', // debugger 금지
    'no-unused-vars': 'warn', // 미사용 변수 경고
  },

  // 무시할 파일
  ignorePatterns: ['node_modules', 'dist', 'build'],
};
```

## React + TypeScript 설정

```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },

  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended',
  ],

  parser: '@typescript-eslint/parser',

  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
  },

  plugins: ['react', 'react-hooks', '@typescript-eslint'],

  rules: {
    // JavaScript 규칙
    'no-var': 'error',
    'prefer-const': 'error',
    eqeqeq: 'error',
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'no-debugger': 'error',
    'no-unused-vars': 'off', // TypeScript에서 처리

    // React 규칙
    'react/react-in-jsx-scope': 'off', // React 17+ JSX Transform
    'react/prop-types': 'off', // TypeScript 사용 시 불필요
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // TypeScript 규칙
    '@typescript-eslint/no-unused-vars': [
      'error',
      {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_',
      },
    ],
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
  },

  ignorePatterns: ['node_modules', 'dist', 'build', '.next'],
};
```

## 필요한 패키지 설치

```bash
# React + TypeScript
npm install -D eslint-plugin-react eslint-plugin-react-hooks
npm install -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

# 4. 주요 규칙 설명

## 변수 규칙

```javascript
rules: {
  // var 금지, const/let 강제
  'no-var': 'error',

  // 재할당하지 않으면 const 강제
  'prefer-const': 'error',

  // 미사용 변수 금지
  'no-unused-vars': 'error',

  // 미사용 매개변수는 무시 (언더스코어 접두사)
  'no-unused-vars': ['error', {
    argsIgnorePattern: '^_',
    varsIgnorePattern: '^_'
  }]
}
```

## 비교 규칙

```javascript
rules: {
  // == 금지, === 강제
  'eqeqeq': 'error',

  // null 비교는 === 강제
  'eqeqeq': ['error', 'always', { null: 'ignore' }]
}
```

## 에러 처리 규칙

```javascript
rules: {
  // console 사용 제한
  'no-console': 'warn',  // 모든 console 경고
  'no-console': ['warn', { allow: ['warn', 'error'] }],  // warn/error만 허용

  // debugger 문 금지
  'no-debugger': 'error',

  // try-catch 없이 throw 금지
  'no-throw-literal': 'error'
}
```

## 비동기 규칙

```javascript
rules: {
  // async 함수에서 return된 Promise를 await하지 않음
  '@typescript-eslint/no-floating-promises': 'error',

  // async 함수에서 await 없이 Promise 사용
  '@typescript-eslint/no-misused-promises': 'error'
}
```

# 5. npm 스크립트

## package.json

```json
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx,.js,.jsx",
    "lint:fix": "eslint src --ext .ts,.tsx,.js,.jsx --fix",
    "lint:watch": "eslint src --ext .ts,.tsx,.js,.jsx --watch"
  }
}
```

## 사용법

```bash
# 검사만 실행 (변경 없음)
npm run lint

# 자동 수정
npm run lint:fix

# 감시 모드 (파일 변경 시 자동 실행)
npm run lint:watch
```

# 6. VS Code 통합

## 확장 프로그램 설치

VS Code에서 "ESLint" 확장 설치

## .vscode/settings.json

```json
{
  "[javascript]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  },
  "[typescript]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  },
  "[typescriptreact]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  }
}
```

이제 파일 저장 시 ESLint 규칙 위반이 자동으로 수정됩니다!

# 7. 실전 예제

## 검사 예제

```typescript
// ❌ 문제가 있는 코드
var user = { name: 'John', age: 25 };
if (user.name == 'John') {
  console.log('Hello');
}

const unused = '사용하지 않음';

async function fetchData() {
  fetch('/api/data'); // await 없음!
}

// ✅ ESLint 수정 후
const user = { name: 'John', age: 25 };
if (user.name === 'John') {
  // console.log("Hello")
}

async function fetchData() {
  await fetch('/api/data');
}
```

# 8. 커스텀 규칙 예제

```javascript
module.exports = {
  rules: {
    // 복잡도가 높은 함수 제한
    complexity: ['warn', 10],

    // 함수 길이 제한
    'max-lines-per-function': [
      'warn',
      {
        max: 100,
        skipBlankLines: true,
        skipComments: true,
      },
    ],

    // 중첩 깊이 제한
    'max-depth': ['warn', 3],

    // 콜백 깊이 제한
    'max-nested-callbacks': ['warn', 3],

    // import 정렬
    'sort-imports': [
      'warn',
      {
        ignoreCase: true,
        ignoreDeclarationSort: true,
      },
    ],
  },
};
```

# 9. Husky와 통합

## 커밋 전 자동 린트

```bash
# 설치
npm install -D husky lint-staged
npx husky install

# pre-commit 훅 추가
npx husky add .husky/pre-commit "npx lint-staged"
```

## .lintstagedrc.json

```json
{
  "*.{ts,tsx,js,jsx}": ["eslint --fix"]
}
```

이제 커밋할 때 자동으로 린트가 실행됩니다!

```bash
git add .
git commit -m "새 기능"  # 자동으로 ESLint 실행
```

# 10. CI/CD 통합

## GitHub Actions

```yaml
name: ESLint

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint
```

# 11. 자주 묻는 질문

## Q: eslint:recommended에 뭐가 있나?

**A:**

```
- 모든 console 사용 금지
- 미사용 변수 검출
- 무한 루프 감지
- typeof 정확한 비교
- 함수 선언 전 사용 금지 등
```

## Q: --fix를 썼는데도 안 고쳐지는 경우?

**A:** 일부 규칙은 자동 수정이 불가능합니다. 수동으로 고쳐야 합니다.

```javascript
// ❌ 자동 수정 불가
function () {  // 함수명 없음

// ✅ 수동으로 고쳐야 함
function getData() {
```

## Q: 특정 파일만 무시하려면?

**A:**

```javascript
// .eslintignore
node_modules
dist
*.min.js
src/legacy/**

// 또는 .eslintrc.js에서
ignorePatterns: [
  'node_modules',
  'dist',
  'src/legacy'
]
```

# 12. 체크리스트

ESLint 설정하기:

```
[ ] ESLint 설치
[ ] 설정 파일 생성 (.eslintrc.js)
[ ] npm 스크립트 추가
[ ] VS Code 확장 설치
[ ] 저장 시 자동 수정 설정
[ ] Husky pre-commit 훅 설정
[ ] CI/CD 통합
[ ] 팀 규칙 문서화
```

# 결론

ESLint는:

✅ 버그를 조기에 발견
✅ 코드 스타일 통일
✅ 자동 수정으로 시간 절약
✅ 팀 협업 향상
✅ 초보자 실수 방지

**모든 JavaScript/TypeScript 프로젝트에 필수입니다!**

## 참고 링크

- [ESLint 공식 사이트](https://eslint.org)
- [ESLint 공식 문서](https://eslint.org/docs/latest/)
- [규칙(Rules) 레퍼런스](https://eslint.org/docs/latest/rules/)
- [ESLint GitHub](https://github.com/eslint/eslint)
