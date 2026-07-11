# Prettier - 자동 코드 포매팅의 완벽 솔루션

## 들어가며

팀 프로젝트에서 코드 스타일이 다르면 얼마나 혼란스러운가요?

```javascript
// 개발자 1의 코드
const user = {name:"John",age:25}
if(user.name=="John"){console.log("Hello")}

// 개발자 2의 코드
const user = {
  name: "John",
  age: 25
}
if (user.name === "John") {
  console.log("Hello")
}

// 개발자 3의 코드
const user={name:'John',age:25};
if(user.name==='John'){console.log("Hello");}

❌ 코드 리뷰가 포매팅으로 엉망
❌ 스타일 논쟁이 끝나지 않음
❌ Git diff가 지저분함
```

**Prettier**는 이 모든 것을 자동으로 해결합니다. 저장하기만 하면 완벽하게 포매팅됩니다!

## 공식 사이트

https://prettier.io

# 1. Prettier란?

## 핵심 개념

```
Prettier = 자동 코드 포매터 (의견 있음 - Opinionated)

특징:
✅ 자동 포매팅 (손으로 수정할 필요 없음)
✅ 옵션이 적음 (선택 여지 없음 = 논쟁 없음)
✅ 매우 빠른 속도
✅ 다양한 파일 타입 지원
✅ 플러그인 시스템
```

## Prettier의 철학

```
"Stop debating style on your team. Save your time and mental energy.
Adopt Prettier and stop debating about style."

스타일 논쟁을 멈추세요.
시간과 정신 에너지를 절약하세요.
Prettier를 채택하고 스타일 논쟁을 종료하세요.
```

# 2. 설치

## 설치 방법

```bash
npm install -D prettier
```

매우 간단합니다. 의존성도 최소입니다!

# 3. 설정 파일

## .prettierrc.json (권장)

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

## 옵션 설명

```javascript
{
  // 문장 끝에 세미콜론 추가
  // true: const x = 1;
  // false: const x = 1
  "semi": true,

  // 여러 줄 배열/객체의 마지막 쉼표
  // "none": 쉼표 없음
  // "es5": ES5에서 유효한 곳만 (객체, 배열, 매개변수 X)
  // "all": 가능한 모든 곳
  "trailingComma": "es5",

  // 문자열에 따옴표 사용
  // true: 'hello'
  // false: "hello"
  "singleQuote": true,

  // 한 줄의 최대 길이
  // 초과하면 자동 줄바꿈
  "printWidth": 80,

  // 탭 크기 (스페이스 개수)
  "tabWidth": 2,

  // 탭 문자 사용
  // true: 탭 문자 (\t)
  // false: 스페이스
  "useTabs": false,

  // 화살표 함수 매개변수 괄호
  // "always": (x) => x
  // "avoid": x => x
  "arrowParens": "always",

  // 줄의 끝 문자
  // "lf": \n (Unix)
  // "crlf": \r\n (Windows)
  // "cr": \r (old Mac)
  // "auto": 자동 감지
  "endOfLine": "lf",

  // HTML 공백 민감도
  // "css": CSS display 속성 따름
  // "strict": 공백 유지
  // "ignore": 공백 무시
  "htmlWhitespaceSensitivity": "css",

  // 객체 속성명 따옴표
  // "as-needed": 필요할 때만 (name: value)
  // "consistent": 하나라도 필요하면 모두
  // "preserve": 입력된 대로
  "quoteProps": "as-needed",

  // JSX 따옴표 (singleQuote 영향 없음)
  "jsxSingleQuote": false,

  // JSX 괄호 위치
  // true: <Component />
  // false: <Component\n/>
  "bracketSameLine": false
}
```

## 다른 형식의 설정

### .prettierrc.js

```javascript
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
};
```

### .prettierrc.yaml

```yaml
semi: true
trailingComma: es5
singleQuote: true
printWidth: 80
tabWidth: 2
```

### package.json에 작성

```json
{
  "prettier": {
    "semi": true,
    "trailingComma": "es5",
    "singleQuote": true
  }
}
```

# 4. .prettierignore 파일

## 포매팅 제외 파일

```
# .prettierignore
node_modules
dist
build
.next
coverage

# 특정 파일
*.min.js
*.min.css

# 특정 디렉토리
src/legacy/**
public/vendor/
```

# 5. npm 스크립트

## package.json

```json
{
  "scripts": {
    "format": "prettier --write src",
    "format:check": "prettier --check src",
    "format:watch": "prettier --write --watch src"
  }
}
```

## 사용법

```bash
# 포매팅 실행 (파일 수정)
npm run format

# 포매팅 확인 (파일 변경 없음, 필요한지만 확인)
npm run format:check

# 감시 모드 (파일 변경 시 자동 포매팅)
npm run format:watch

# 특정 파일만
npm run format -- src/App.tsx

# 특정 확장자
npm run format -- "src/**/*.tsx"
```

# 6. VS Code 통합

## 확장 프로그램 설치

VS Code에서 "Prettier - Code formatter" 확장 설치

## .vscode/settings.json

```json
{
  // Prettier를 기본 포매터로 설정
  "editor.defaultFormatter": "esbenp.prettier-vscode",

  // 저장 시 자동 포매팅
  "editor.formatOnSave": true,

  // 붙여넣기 시 자동 포매팅
  "editor.formatOnPaste": true,

  // 타이핑 중 자동 포매팅
  "editor.formatOnType": false, // 주의: 성능 이슈 가능

  // 특정 언어 설정
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  }
}
```

이제 파일 저장 시 자동으로 완벽하게 포매팅됩니다!

# 7. 포매팅 예제

## 실제 변환

```javascript
// ❌ 포매팅 전
const user = { name: 'John', age: 25, email: 'john@example.com' };
const result = user.name == 'John' ? true : false;
if (result) {
  console.log('Hello');
}

const items = [1, 2, 3, 4, 5];
items.map((item) => item * 2);

// ✅ Prettier 포매팅 후
const user = {
  name: 'John',
  age: 25,
  email: 'john@example.com',
};
const result = user.name === 'John' ? true : false;
if (result) {
  console.log('Hello');
}

const items = [1, 2, 3, 4, 5];
items.map((item) => item * 2);
```

## 긴 줄 자동 줄바꿈

```javascript
// ❌ 포매팅 전
const message = '이것은 매우 긴 메시지입니다 그리고 한 줄에 모두 들어갑니다';
function calculateSomething(parameter1, parameter2, parameter3, parameter4) {
  return parameter1 + parameter2 + parameter3 + parameter4;
}

// ✅ Prettier 포매팅 후 (printWidth: 80)
const message = '이것은 매우 긴 메시지입니다 그리고 한 줄에 모두 들어갑니다';
function calculateSomething(parameter1, parameter2, parameter3, parameter4) {
  return parameter1 + parameter2 + parameter3 + parameter4;
}
```

# 8. Husky와 통합

## 자동 포매팅 (커밋 전)

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
  "*.{ts,tsx,js,jsx}": ["prettier --write"],
  "*.{json,md,css,scss}": ["prettier --write"]
}
```

이제 커밋할 때 자동으로 포매팅됩니다!

```bash
git add .
git commit -m "새 기능"  # 자동으로 Prettier 실행
```

# 9. CI/CD 통합

## 포매팅 확인 (자동 수정 없음)

```bash
# package.json 스크립트
"format:ci": "prettier --check src"
```

## GitHub Actions

```yaml
name: Format Check

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  format:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Check formatting
        run: npm run format:check
```

# 10. 플러그인

## Prettier 플러그인 사용

### Tailwind CSS 플러그인

```bash
npm install -D prettier-plugin-tailwindcss
```

```json
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

클래스명이 자동으로 정렬됩니다!

```jsx
// ❌ 포매팅 전
<div className="flex justify-center items-center bg-red-500 text-white p-4">

// ✅ 포매팅 후
<div className="flex items-center justify-center bg-red-500 p-4 text-white">
```

# 11. 팀 협업

## 설정 공유

### .vscode/extensions.json

```json
{
  "recommendations": ["esbenp.prettier-vscode"]
}
```

### FORMATTING_GUIDE.md

```markdown
# 코드 포매팅 가이드

## Prettier 자동 포매팅

모든 코드는 Prettier로 자동 포매팅됩니다.

### 설정

- .prettierrc.json 참조
- VS Code 저장 시 자동 포매팅

### 명령어

npm run format # 전체 포매팅
npm run format:check # 포매팅 확인 (변경 없음)

### 규칙

- 한 줄 최대 80자
- 2칸 들여쓰기
- 단일 따옴표 사용
- 마지막 쉼표 유지 (ES5 호환)
```

# 12. 자주 묻는 질문

## Q: Prettier가 제대로 안 되면?

**A:**

```bash
# VS Code에서 명령팔레트 (Ctrl+Shift+P)
> Prettier: Force Format Document

# 또는 npm에서
npm run format
```

## Q: 기존 코드를 일괄 포매팅하려면?

**A:**

```bash
# 전체 디렉토리 포매팅
npm run format -- src

# 특정 파일
npm run format -- src/App.tsx

# 특정 패턴
npm run format -- "src/**/*.jsx"
```

## Q: 라인 길이를 줄이려면?

**A:**

```json
{
  "printWidth": 100 // 기본값 80
}
```

## Q: import 정렬은 어떻게?

**A:** Prettier는 import 정렬을 하지 않습니다. ESLint의 다른 플러그인을 사용하세요.

```bash
npm install -D eslint-plugin-import
```

# 13. 체크리스트

Prettier 설정하기:

```
[ ] prettier 설치
[ ] .prettierrc.json 생성
[ ] .prettierignore 생성
[ ] npm 스크립트 추가
[ ] VS Code 확장 설치
[ ] VS Code 설정 (저장 시 자동 포매팅)
[ ] Husky pre-commit 훅 설정
[ ] 팀 가이드 문서화
```

# 결론

Prettier는:

✅ 자동 포매팅으로 시간 절약
✅ 스타일 논쟁 종료
✅ 일관된 코드 포매팅
✅ Git diff 간결
✅ 팀 협업 향상

**모든 프로젝트에 Prettier를 사용하세요!**

## 참고 링크

- [Prettier 공식 사이트](https://prettier.io)
- [옵션(Options) 문서](https://prettier.io/docs/en/options.html)
- [Prettier GitHub](https://github.com/prettier/prettier)
