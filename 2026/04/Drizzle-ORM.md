# Drizzle ORM - TypeScript 우선의 SQL 라이브러리

## 들어가며

데이터베이스 작업할 때 이런 문제들이 있나요?

```typescript
// ❌ 순수 SQL (타입 안전성 없음)
const result = await db.query('SELECT * FROM users WHERE id = $1', [1]);
// result의 타입? 어떤 필드가 있는지?

// ❌ ORM (너무 무거움)
const user = await User.findById(1);
// 많은 메모리, 느린 성능

// ❌ Query Builder (타입이 복잡함)
const user = await db.select().from(users).where(eq(users.id, 1));
// 자동완성이 안 좋음
```

**Drizzle ORM**은 이 모든 문제를 해결합니다. TypeScript 우선 설계로 완벽한 타입 안전성을 제공하면서도 경량입니다.

## 공식 사이트

https://orm.drizzle.team

# 1. Drizzle ORM이란?

## 핵심 개념

```
Drizzle ORM = TypeScript 우선의 경량 SQL 라이브러리

특징:
✅ 완벽한 TypeScript 지원
✅ 경량 (작은 번들 크기)
✅ 타입 안전성
✅ 강력한 쿼리 빌더
✅ 마이그레이션 도구
✅ 여러 데이터베이스 지원
✅ Zero Runtime Magic
```

## Drizzle vs Prisma vs TypeORM

```
Drizzle:
- 매우 가벼움
- 타입 추론 최고
- 학습곡선 낮음
- SQL에 가까움

Prisma:
- 무겁지만 편함
- 자동 마이그레이션
- ORM 기능 많음
- 성능 중간

TypeORM:
- 무거움
- 데코레이터 기반
- 복잡한 설정
- 성능 낮음
```

# 2. 설치

## 설치 및 설정

```bash
# 기본 설치
npm install drizzle-orm

# 데이터베이스 드라이버
npm install pg              # PostgreSQL
npm install mysql2          # MySQL
npm install better-sqlite3  # SQLite

# 개발 의존성
npm install -D drizzle-kit
npm install -D @types/pg @types/node
```

# 3. 기본 설정

## PostgreSQL 연결

```typescript
// src/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
});

export const db = drizzle(pool);
```

## MySQL 연결

```typescript
// src/db.ts
import { drizzle } from 'drizzle-orm/mysql2';
import mysql from 'mysql2/promise';

const poolConnection = await mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
});

export const db = drizzle(poolConnection);
```

## SQLite 연결

```typescript
// src/db.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database(process.env.DATABASE_URL || 'sqlite.db');
export const db = drizzle(sqlite);
```

# 4. 스키마 정의

## 테이블 정의

```typescript
// src/schema.ts
import {
  integer,
  text,
  boolean,
  timestamp,
  pgTable,
} from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  email: text().notNull().unique(),
  password: text().notNull(),
  age: integer(),
  isActive: boolean().default(true),
  createdAt: timestamp().defaultNow(),
  updatedAt: timestamp().defaultNow(),
});

export const posts = pgTable('posts', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  title: text().notNull(),
  content: text(),
  userId: integer()
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  published: boolean().default(false),
  createdAt: timestamp().defaultNow(),
  updatedAt: timestamp().defaultNow(),
});

export const comments = pgTable('comments', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  text: text().notNull(),
  postId: integer()
    .notNull()
    .references(() => posts.id, { onDelete: 'cascade' }),
  userId: integer()
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  createdAt: timestamp().defaultNow(),
});
```

## 타입 추론

```typescript
import { typeof users } from './schema'

// 자동으로 타입 생성
type User = typeof users.$inferSelect
type NewUser = typeof users.$inferInsert

// 사용
const user: User = { /* ... */ }
const newUser: NewUser = { /* ... */ }
```

# 5. 기본 쿼리

## SELECT (조회)

```typescript
import { db } from './db';
import { users } from './schema';
import { eq } from 'drizzle-orm';

// 모든 사용자 조회
const allUsers = await db.select().from(users);

// 특정 사용자 조회
const user = await db.select().from(users).where(eq(users.id, 1));

// 특정 컬럼만 조회
const userNames = await db
  .select({ id: users.id, name: users.name })
  .from(users);

// 첫 번째 사용자만
const firstUser = await db.select().from(users).limit(1);
```

## INSERT (삽입)

```typescript
import { db } from './db';
import { users } from './schema';

// 단일 삽입
const result = await db
  .insert(users)
  .values({
    name: 'John Doe',
    email: 'john@example.com',
    password: 'hashed_password',
    age: 25,
  })
  .returning();

// 여러 개 삽입
const results = await db
  .insert(users)
  .values([
    { name: 'John', email: 'john@example.com', password: 'hash1' },
    { name: 'Jane', email: 'jane@example.com', password: 'hash2' },
    { name: 'Bob', email: 'bob@example.com', password: 'hash3' },
  ])
  .returning();
```

## UPDATE (수정)

```typescript
import { db } from './db';
import { users } from './schema';
import { eq } from 'drizzle-orm';

// 특정 사용자 수정
const result = await db
  .update(users)
  .set({ name: 'Jane Doe', age: 26 })
  .where(eq(users.id, 1))
  .returning();

// 여러 사용자 수정
const results = await db
  .update(users)
  .set({ isActive: false })
  .where(eq(users.isActive, true))
  .returning();
```

## DELETE (삭제)

```typescript
import { db } from './db';
import { users } from './schema';
import { eq } from 'drizzle-orm';

// 특정 사용자 삭제
await db.delete(users).where(eq(users.id, 1));

// 조건에 맞는 모든 사용자 삭제
await db.delete(users).where(eq(users.isActive, false));
```

# 6. 조건과 필터

## WHERE 조건

```typescript
import {
  eq,
  ne,
  gt,
  gte,
  lt,
  lte,
  like,
  inArray,
  between,
  and,
  or,
} from 'drizzle-orm';

// 동등 비교
await db.select().from(users).where(eq(users.id, 1));

// 같지 않음
await db.select().from(users).where(ne(users.status, 'inactive'));

// 크기 비교
await db.select().from(users).where(gt(users.age, 18)); // >
await db.select().from(users).where(gte(users.age, 18)); // >=
await db.select().from(users).where(lt(users.age, 65)); // <
await db.select().from(users).where(lte(users.age, 65)); // <=

// 문자열 검색
await db.select().from(users).where(like(users.name, '%John%'));

// 배열 포함
await db
  .select()
  .from(users)
  .where(inArray(users.id, [1, 2, 3]));

// 범위
await db
  .select()
  .from(users)
  .where(between(users.age, 18, 65));

// AND / OR
await db
  .select()
  .from(users)
  .where(and(eq(users.isActive, true), gte(users.age, 18), lt(users.age, 65)));

await db
  .select()
  .from(users)
  .where(or(eq(users.role, 'admin'), eq(users.role, 'moderator')));
```

# 7. 정렬과 제한

```typescript
import { asc, desc } from 'drizzle-orm';

// 정렬
const users = await db.select().from(users).orderBy(asc(users.name)); // 오름차순

const users = await db.select().from(users).orderBy(desc(users.createdAt)); // 내림차순

// 여러 컬럼으로 정렬
const users = await db
  .select()
  .from(users)
  .orderBy(desc(users.age), asc(users.name));

// 제한과 오프셋 (페이지네이션)
const page = 2;
const pageSize = 10;
const users = await db
  .select()
  .from(users)
  .limit(pageSize)
  .offset((page - 1) * pageSize);
```

# 8. 조인 (JOIN)

## INNER JOIN

```typescript
import { eq } from 'drizzle-orm';

// 사용자와 포스트 조인
const result = await db
  .select()
  .from(users)
  .innerJoin(posts, eq(users.id, posts.userId));

// 결과: [{ users: {...}, posts: {...} }, ...]
```

## LEFT JOIN

```typescript
// 모든 사용자와 그들의 포스트 (포스트 없으면 null)
const result = await db
  .select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId));
```

## 다중 조인

```typescript
const result = await db
  .select()
  .from(posts)
  .innerJoin(users, eq(posts.userId, users.id))
  .leftJoin(comments, eq(posts.id, comments.postId));
```

## 선택적 조인 데이터

```typescript
const result = await db
  .select({
    id: users.id,
    name: users.name,
    postCount: // 포스트 수... (집계 함수 필요)
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId))
```

# 9. 집계 함수

```typescript
import { count, sum, avg, min, max } from 'drizzle-orm';

// 사용자 수
const result = await db.select({ count: count() }).from(users);

// 합계
const total = await db.select({ sum: sum(users.age) }).from(users);

// 평균
const average = await db.select({ avg: avg(users.age) }).from(users);

// 최소/최대
const ages = await db
  .select({
    min: min(users.age),
    max: max(users.age),
  })
  .from(users);

// GROUP BY
const result = await db
  .select({
    role: users.role,
    count: count(),
  })
  .from(users)
  .groupBy(users.role);
```

# 10. 트랜잭션

```typescript
import { db } from './db';

// 트랜잭션 실행
const result = await db.transaction(async (tx) => {
  // 1단계: 사용자 생성
  const newUser = await tx
    .insert(users)
    .values({
      name: 'John',
      email: 'john@example.com',
      password: 'hashed',
    })
    .returning();

  // 2단계: 포스트 생성
  const newPost = await tx
    .insert(posts)
    .values({
      title: 'First Post',
      userId: newUser[0].id,
    })
    .returning();

  // 모든 작업이 성공하면 커밋, 실패하면 롤백
  return { user: newUser[0], post: newPost[0] };
});
```

# 11. 마이그레이션

## drizzle.config.ts

```typescript
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/schema.ts',
  out: './drizzle',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.DATABASE_URL,
  },
} satisfies Config;
```

## 마이그레이션 생성

```bash
# 마이그레이션 파일 생성
npx drizzle-kit generate:pg

# 데이터베이스에 적용
npx drizzle-kit migrate
```

## 마이그레이션 파일 예제

```sql
-- drizzle/0001_initial.sql
CREATE TABLE IF NOT EXISTS "users" (
  "id" serial PRIMARY KEY,
  "name" text NOT NULL,
  "email" text NOT NULL UNIQUE,
  "age" integer,
  "createdAt" timestamp DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS "posts" (
  "id" serial PRIMARY KEY,
  "title" text NOT NULL,
  "userId" integer NOT NULL REFERENCES "users"("id"),
  "createdAt" timestamp DEFAULT NOW()
);
```

# 12. 실전 예제

## Express + Drizzle

```typescript
// src/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle(pool, { schema });

// src/api/users.ts
import { Router } from 'express';
import { db } from '../db';
import { users } from '../schema';
import { eq } from 'drizzle-orm';

const router = Router();

// 모든 사용자 조회
router.get('/users', async (req, res) => {
  const allUsers = await db.select().from(users);
  res.json(allUsers);
});

// 특정 사용자 조회
router.get('/users/:id', async (req, res) => {
  const user = await db
    .select()
    .from(users)
    .where(eq(users.id, parseInt(req.params.id)))
    .limit(1);

  res.json(user[0] || null);
});

// 사용자 생성
router.post('/users', async (req, res) => {
  const newUser = await db.insert(users).values(req.body).returning();

  res.status(201).json(newUser[0]);
});

// 사용자 수정
router.put('/users/:id', async (req, res) => {
  const updated = await db
    .update(users)
    .set(req.body)
    .where(eq(users.id, parseInt(req.params.id)))
    .returning();

  res.json(updated[0]);
});

// 사용자 삭제
router.delete('/users/:id', async (req, res) => {
  await db.delete(users).where(eq(users.id, parseInt(req.params.id)));

  res.status(204).send();
});

export default router;
```

# 13. 복잡한 쿼리

## 서브쿼리

```typescript
import { alias } from 'drizzle-orm/pg-core';

// 각 사용자의 포스트 수
const result = await db
  .select({
    id: users.id,
    name: users.name,
    postCount: count(posts.id),
  })
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId))
  .groupBy(users.id);
```

## 재귀 쿼리 (CTE)

```typescript
import { sql } from 'drizzle-orm';

const result = await db.with(cte).select().from(cte);
// ...
```

# 14. 타입 안전성

```typescript
// 컴파일 타임에 에러 감지
const users = await db
  .select()
  .from(users)
  .where(eq(users.invalidColumn, 'value'));
// TypeScript Error: Property 'invalidColumn' does not exist

// 올바른 사용
const users = await db
  .select()
  .from(users)
  .where(eq(users.email, 'john@example.com'));
// ✅ 성공
```

# 15. 체크리스트

Drizzle ORM 프로젝트 시작하기:

```
[ ] 패키지 설치
[ ] 데이터베이스 연결 설정
[ ] 스키마 정의
[ ] 마이그레이션 생성 및 적용
[ ] 기본 CRUD 작성
[ ] 조인 쿼리 구현
[ ] 집계 함수 사용
[ ] 트랜잭션 구현
[ ] Express와 통합
```

# 결론

Drizzle ORM은:

✅ 완벽한 TypeScript 지원
✅ 경량이고 빠름
✅ 강력한 타입 안전성
✅ SQL에 가까우면서도 편함
✅ 학습곡선 낮음

**TypeScript 프로젝트의 데이터베이스는 Drizzle ORM으로!**

## 참고 링크

- [Drizzle ORM 공식 사이트](https://orm.drizzle.team)
- [Drizzle ORM 공식 문서](https://orm.drizzle.team/docs/overview)
- [Drizzle ORM GitHub](https://github.com/drizzle-team/drizzle-orm)
