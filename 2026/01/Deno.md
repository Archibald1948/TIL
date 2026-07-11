# Deno - Node.js의 문제를 근본적으로 해결한 JavaScript 런타임

## 들어가며

당신이 Node.js 생태계의 문제점을 느껴본 적이 있나요?

- npm의 의존성 지옥 (node_modules의 쓸데없는 크기)
- 보안 문제 (패키지가 무분별하게 파일 접근)
- 모듈 시스템의 혼란 (CommonJS vs ES Modules)
- TypeScript 설정의 복잡함

Deno는 Node.js의 창시자 Ryan Dahl이 Node.js의 "10년의 후회"를 토대로 만든 새로운 JavaScript 런타임입니다.
처음부터 올바르게 설계되어, 더 안전하고 효율적입니다.

## 공식 사이트

https://github.com/denoland/deno

## Node.js vs Deno 비교

### 핵심 차이점

```
Node.js (1009년):
├─ CommonJS + ES Modules 혼재
├─ package.json 중앙 집중식
├─ node_modules (비대함)
├─ 기본 보안 없음
├─ 도구 분산됨 (npm, webpack 등)
└─ TypeScript 별도 설정

Deno (2018년, 근본 재설계):
├─ ES Modules만 사용
├─ deno.json (간단함)
├─ 캐싱 (효율적)
├─ 기본 보안 (권한 시스템)
├─ 내장 도구 (Deno fmt, Deno lint 등)
└─ TypeScript 기본 지원
```

### 기능 비교

| 기능             | Node.js           | Deno           |
| ---------------- | ----------------- | -------------- |
| **JavaScript**   | ✅                | ✅             |
| **TypeScript**   | 별도 설정 필요    | ✅ 기본 지원   |
| **보안 권한**    | ❌                | ✅ 기본 제공   |
| **패키지 관리**  | npm               | URL 기반       |
| **포매터**       | 별도 (Prettier)   | ✅ deno fmt    |
| **린터**         | 별도 (ESLint)     | ✅ deno lint   |
| **테스트**       | 별도 (Jest 등)    | ✅ deno test   |
| **번들러**       | 별도 (webpack 등) | ✅ deno bundle |
| **node_modules** | 거대함            | 없음           |
| **학습곡선**     | 높음              | 낮음           |

## Deno의 혁신적 기능

### 1. TypeScript 기본 지원

추가 설정 없이 TypeScript를 기본으로 사용합니다.

```bash
# Node.js
npm install -D typescript ts-node @types/node
npx ts-node src/main.ts

# Deno
deno run src/main.ts  # TypeScript 자동 인식!
```

```typescript
// main.ts - 그냥 TypeScript로 작성
interface User {
  id: string;
  name: string;
  email: string;
}

async function getUser(id: string): Promise<User> {
  const response = await fetch(`https://api.example.com/users/${id}`);
  return response.json();
}

const user = await getUser('123');
console.log(user.name);
```

### 2. 권한 기반 보안

패키지가 무엇을 할 수 있는지 명시적으로 제어합니다.

```bash
# 아무 권한도 없음
deno run script.ts

# 파일 접근만 허용
deno run --allow-read script.ts

# 특정 파일만 접근 허용
deno run --allow-read=/tmp script.ts

# 네트워크 접근만 허용
deno run --allow-net script.ts

# 특정 도메인만 접근 허용
deno run --allow-net=api.example.com script.ts

# 환경 변수 접근 허용
deno run --allow-env script.ts

# 특정 환경 변수만 허용
deno run --allow-env=API_KEY script.ts

# 모든 권한 (좋지 않은 예)
deno run --allow-all script.ts
```

```typescript
// secure-example.ts
// 파일 읽기
const data = await Deno.readTextFile('./config.json');
// ↑ --allow-read 권한 필요

// 네트워크 요청
const response = await fetch('https://api.example.com/data');
// ↑ --allow-net=api.example.com 권한 필요

// 환경 변수
const apiKey = Deno.env.get('API_KEY');
// ↑ --allow-env=API_KEY 권한 필요
```

### 3. URL 기반 모듈 시스템

node_modules가 없습니다. 모듈을 URL로 직접 임포트합니다.

```typescript
// Node.js
import express from 'express';
import dotenv from 'dotenv';
import { z } from 'zod';

// Deno
import { serve } from 'https://deno.land/std@0.208.0/http/server.ts';
import { assertEquals } from 'https://deno.land/std@0.208.0/testing/asserts.ts';
import { z } from 'https://deno.land/x/zod@v3.22.4/mod.ts';
```

**장점:**

- node_modules 없음 (디스크 공간 절약)
- 명시적 버전 관리
- 직관적인 모듈 탐색

### 4. deno.json 설정 (간단함)

```json
// deno.json
{
  "name": "my-app",
  "version": "1.0.0",

  // 임포트 맵 (import alias)
  "imports": {
    "std/": "https://deno.land/std@0.208.0/",
    "oak": "https://deno.land/x/oak@v12.0.0/mod.ts",
    "@/": "./src/"
  },

  // 공개 API 정의
  "exports": {
    ".": "./src/mod.ts",
    "./server": "./src/server.ts"
  },

  // 스크립트 (npm scripts와 유사)
  "tasks": {
    "dev": "deno run --allow-net --allow-read src/server.ts",
    "test": "deno test --allow-net --allow-read",
    "fmt": "deno fmt",
    "lint": "deno lint"
  },

  // 린트 설정
  "lint": {
    "rules": {
      "tags": ["recommended"],
      "exclude": ["camelcase"]
    }
  },

  // 포매터 설정
  "fmt": {
    "semiColons": true,
    "singleQuote": true,
    "indentWidth": 2,
    "lineWidth": 100,
    "trailingComma": "es5"
  }
}
```

### 5. 내장 도구

```bash
# 포매팅 (Prettier 대체)
deno fmt  # 형식 정렬
deno fmt --check  # 확인만

# 린팅 (ESLint 대체)
deno lint  # 린트 검사

# 테스트 (Jest 대체)
deno test  # 테스트 실행
deno test --coverage  # 커버리지 계산

# 번들링
deno bundle src/main.ts dist/bundle.js  # 번들 생성

# 타입 체크
deno check src/main.ts  # 타입 검증만
```

## 프로덕션 레벨 Deno 프로젝트

### 1. 웹 서버 (Oak Framework)

```typescript
// src/server.ts
import { Application, Router } from '@/oak';
import { oakCors } from '@/oak_cors';

const app = new Application();
const router = new Router();

// 타입 정의
interface User {
  id: string;
  name: string;
  email: string;
}

// 라우트
router
  .get('/', (ctx) => {
    ctx.response.body = 'Welcome to Deno server!';
  })
  .get('/users/:id', async (ctx) => {
    const id = ctx.params.id;
    const user: User = {
      id,
      name: 'John Doe',
      email: 'john@example.com',
    };
    ctx.response.body = user;
  })
  .post('/users', async (ctx) => {
    const body = ctx.request.body();
    const user = await body.value;
    ctx.response.status = 201;
    ctx.response.body = { ...user, id: crypto.randomUUID() };
  });

// 미들웨어
app.use(oakCors());
app.use(router.routes());
app.use(router.allowedMethods());

// 서버 시작
const PORT = 3000;
console.log(`Server running on http://localhost:${PORT}`);
await app.listen({ port: PORT });
```

```json
// deno.json
{
  "imports": {
    "oak": "https://deno.land/x/oak@v12.0.0/mod.ts",
    "oak_cors": "https://deno.land/x/cors@v1.2.2/mod.ts",
    "@/": "./src/"
  },
  "tasks": {
    "dev": "deno run --allow-net --allow-read src/server.ts"
  }
}
```

```bash
deno task dev
# Server running on http://localhost:3000
```

### 2. 데이터베이스 연결

```typescript
// src/database.ts
import { Client } from 'https://deno.land/x/postgres@v0.17.0/mod.ts';

const client = new Client({
  user: Deno.env.get('DB_USER') || 'postgres',
  password: Deno.env.get('DB_PASSWORD') || 'postgres',
  hostname: Deno.env.get('DB_HOST') || 'localhost',
  port: 5432,
  database: Deno.env.get('DB_NAME') || 'myapp',
});

await client.connect();

export interface User {
  id: string;
  name: string;
  email: string;
  created_at: Date;
}

export async function getUser(id: string): Promise<User | null> {
  const result = await client.queryObject<User>(
    'SELECT * FROM users WHERE id = $1',
    [id],
  );
  return result.rows[0] || null;
}

export async function createUser(name: string, email: string): Promise<User> {
  const result = await client.queryObject<User>(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email],
  );
  return result.rows[0];
}
```

### 3. 마이그레이션 관리

```typescript
// src/migrations/001_create_users.ts
export const up = `
  CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR NOT NULL,
    email VARCHAR NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT now()
  );
  
  CREATE INDEX idx_users_email ON users(email);
`;

export const down = `
  DROP TABLE users;
`;

// src/migrate.ts
import { Client } from '@/postgres';

interface Migration {
  name: string;
  up: string;
  down: string;
}

async function migrate() {
  const client = new Client({
    /* 설정 */
  });

  await client.connect();

  // 마이그레이션 테이블 생성
  await client.queryArray(`
    CREATE TABLE IF NOT EXISTS migrations (
      id SERIAL PRIMARY KEY,
      name VARCHAR NOT NULL UNIQUE,
      executed_at TIMESTAMP DEFAULT now()
    )
  `);

  // 실행할 마이그레이션
  const migrations: Migration[] = [
    await import('./migrations/001_create_users.ts'),
  ];

  for (const migration of migrations) {
    const exists = await client.queryArray(
      'SELECT 1 FROM migrations WHERE name = $1',
      [migration.name],
    );

    if (!exists.rows.length) {
      console.log(`Executing migration: ${migration.name}`);
      await client.queryArray(migration.up);
      await client.queryArray('INSERT INTO migrations (name) VALUES ($1)', [
        migration.name,
      ]);
    }
  }

  await client.end();
}

if (import.meta.main) {
  await migrate();
}
```

### 4. 테스트

```typescript
// src/utils.test.ts
import { assertEquals, assertThrows } from 'std/testing/asserts.ts';
import { add, divide } from '@/utils.ts';

Deno.test('add function', () => {
  assertEquals(add(2, 3), 5);
  assertEquals(add(-1, 1), 0);
});

Deno.test('divide throws on zero', () => {
  assertThrows(() => divide(10, 0), Error, 'Cannot divide by zero');
});

// src/api.test.ts
import { assertEquals } from 'std/testing/asserts.ts';
import { getUser } from '@/api.ts';

Deno.test('getUser API', async () => {
  // Mock 설정
  const mockUser = { id: '1', name: 'John', email: 'john@example.com' };

  const result = await getUser('1');
  assertEquals(result, mockUser);
});
```

```bash
# 테스트 실행
deno task test

# 커버리지 포함
deno test --coverage=./coverage src/
deno coverage ./coverage
```

### 5. 환경 설정

```typescript
// src/config.ts
export interface Config {
  env: string;
  port: number;
  databaseUrl: string;
  apiKey: string;
  logLevel: string;
}

export function loadConfig(): Config {
  const env = Deno.env.get('NODE_ENV') || 'development';

  return {
    env,
    port: parseInt(Deno.env.get('PORT') || '3000'),
    databaseUrl: Deno.env.get('DATABASE_URL') || '',
    apiKey: Deno.env.get('API_KEY') || '',
    logLevel: Deno.env.get('LOG_LEVEL') || 'info',
  };
}

// 사용
const config = loadConfig();
console.log(`Running in ${config.env} mode`);
```

```bash
# .env 파일
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost/myapp
API_KEY=your-secret-key
LOG_LEVEL=debug
```

## Node.js에서 Deno로 마이그레이션

### 1. package.json → deno.json

```json
// package.json (기존)
{
  "dependencies": {
    "express": "^4.18.0",
    "cors": "^2.8.0",
    "dotenv": "^16.0.0",
    "zod": "^3.22.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "jest": "^29.0.0"
  },
  "scripts": {
    "start": "node src/index.js",
    "test": "jest"
  }
}
```

```json
// deno.json (Deno)
{
  "imports": {
    "oak": "https://deno.land/x/oak@v12.0.0/mod.ts",
    "cors": "https://deno.land/x/cors@v1.2.2/mod.ts",
    "zod": "https://deno.land/x/zod@v3.22.4/mod.ts",
    "std/": "https://deno.land/std@0.208.0/"
  },
  "tasks": {
    "start": "deno run --allow-net --allow-env --allow-read src/main.ts",
    "test": "deno test --allow-net --allow-env"
  }
}
```

### 2. CommonJS → ES Modules

```typescript
// Node.js (CommonJS)
const express = require('express');
const cors = require('cors');
const { z } = require('zod');

const app = express();
module.exports = { app };
```

```typescript
// Deno (ES Modules)
import { Application } from 'https://deno.land/x/oak@v12.0.0/mod.ts';
import cors from 'https://deno.land/x/cors@v1.2.2/mod.ts';
import { z } from 'https://deno.land/x/zod@v3.22.4/mod.ts';

export { app };
```

### 3. 권한 명시

```bash
# 마이그레이션된 스크립트 실행
deno run \
  --allow-net \           # 네트워크 접근
  --allow-read \          # 파일 읽기
  --allow-env \           # 환경 변수
  --allow-write \         # 파일 쓰기
  src/main.ts
```

## 팀 협업 가이드

````markdown
# Deno 프로젝트 온보딩 가이드

## 설치

```bash
# macOS/Linux
curl https://deno.land/x/install/install.sh | sh

# Windows
irm https://deno.land/x/install/install.ps1 | iex

# Homebrew
brew install deno
```
````

## 프로젝트 시작

```bash
git clone <repo>
deno cache --reload deno.json
deno task dev
```

## 개발 규칙

### 1. TypeScript 사용

- 항상 TypeScript로 작성
- 타입 정의 필수

### 2. 권한 최소화

- 필요한 권한만 요청
- 문서에 권한 명시

### 3. 내장 도구 사용

```bash
deno fmt           # 자동 포매팅
deno lint          # 린트
deno test          # 테스트
deno check         # 타입 검사
```

### 4. 의존성 관리

- deno.json에 import alias 정의
- 모듈 버전 명시

## 주의사항

- npm 패키지 사용 불가 (URL 기반만 가능)
- Node.js 표준 라이브러리 사용 불가
- 대신 Deno 표준 라이브러리 (std/) 사용

## FAQ

**Q: npm 패키지를 사용하고 싶어요.**
A: npm 레지스트리를 Deno와 호환되도록 하는 서비스(npm:)를 사용할 수 있습니다.

**Q: 패키지가 없으면 어떻게 하나요?**
A: Deno 생태계가 빠르게 성장 중입니다. 필요하면 직접 구현하거나 기여하세요.

## Deno의 미래

Deno는 다음 기능들을 준비 중입니다:

- **npm 호환성 개선**: npm 패키지 직접 사용 가능
- **TypeScript 성능 향상**: 컴파일 캐싱 개선
- **표준 라이브러리 확대**: 더 많은 내장 기능
- **배포 서비스**: Deno Deploy (서버리스)

## 결론

Deno는 **현대 JavaScript 개발의 표준**이 될 것입니다:

✅ **보안**: 명시적 권한 시스템
✅ **단순성**: TypeScript, 포매터, 린터가 모두 내장
✅ **성능**: Node.js보다 빠름
✅ **미래지향적**: ES Modules, URL 기반 모듈
✅ **생산성**: 보일러플레이트 최소화

특히 **신규 프로젝트**나 **마이크로서비스**에 강력히 권장됩니다.

Node.js의 한계를 느꼈다면, **지금 바로 Deno로 전환해보세요!**

## 참고 링크

- [Deno 공식 사이트](https://deno.com)
- [Deno 공식 문서](https://docs.deno.com)
- [Deno GitHub](https://github.com/denoland/deno)
