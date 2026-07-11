# Prisma - 현대적 ORM의 완벽한 선택

## 들어가며

데이터베이스와 통신할 때 SQL을 직접 작성하는 것이 얼마나 번거로운지 알고 계신가요?

```sql
-- SQL 직접 작성 (번거로움)
SELECT * FROM users WHERE email = $1;
INSERT INTO users (name, email, age) VALUES ($1, $2, $3);
UPDATE users SET age = $1 WHERE id = $2;
DELETE FROM users WHERE id = $1;
```

Prisma는 이 모든 번거로움을 없앱니다.
타입 안전한 데이터베이스 접근, 자동 마이그레이션, 강력한 관계 설정 - 모든 것을 우아하게 처리합니다.

## 공식 사이트

https://github.com/prisma/prisma

## ORM이란?

### ORM의 개념

```
ORM = Object-Relational Mapping

관계형 데이터베이스의 테이블을 객체로 매핑합니다.

테이블      ↔️      클래스
Row        ↔️      객체
Column     ↔️      속성

예:
users 테이블  ↔️  User 클래스
id, name, email  ↔️  id, name, email 속성
```

### 기존 방식 vs Prisma

```typescript
// ❌ 기존 방식 (TypeORM)
import { getRepository } from 'typeorm';
import { User } from './User';

const userRepository = getRepository(User);
const user = await userRepository.findOne({
  where: { email: 'john@example.com' },
});

// ✅ Prisma 방식
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' },
});
```

**Prisma의 장점:**

- SQL을 쓸 필요가 없습니다
- 타입 안전합니다
- 마이그레이션이 쉽습니다
- 관계 쿼리가 직관적입니다
- IDE 자동완성이 완벽합니다

## Prisma 설치 및 기본 설정

### 1. 설치

```bash
npm install @prisma/client
npm install -D prisma
```

### 2. 초기화

```bash
npx prisma init
```

생성되는 파일:

```
.env                 # 데이터베이스 URL
prisma/
└── schema.prisma    # 데이터베이스 스키마 정의
```

### 3. 데이터베이스 연결

```env
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

## 스키마 정의

### 기본 구조

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 모델 정의
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String
  posts Post[]
}

model Post {
  id    Int     @id @default(autoincrement())
  title String
  body  String
  user  User    @relation(fields: [userId], references: [id])
  userId Int
}
```

### 필드 타입

```prisma
model User {
  // 기본 타입
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  age       Int?      // nullable

  // 문자열
  bio       String    @db.Text  // 긴 텍스트

  // 숫자
  salary    Float

  // 날짜
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  // 열거형
  role      Role      @default(USER)

  // JSON
  metadata  Json

  // 관계
  posts     Post[]
}

enum Role {
  ADMIN
  USER
  GUEST
}
```

### 관계 설정

```prisma
// 1대다 관계
model User {
  id    Int
  posts Post[]  // 사용자는 여러 게시물을 가짐
}

model Post {
  id     Int
  userId Int
  user   User    @relation(fields: [userId], references: [id])
}

// 다대다 관계
model Student {
  id        Int
  courses   Course[]
}

model Course {
  id       Int
  students Student[]
}

// 자기 참조
model User {
  id       Int
  name     String
  friends  User[]      @relation("Friends")
  friendOf User[]      @relation("Friends")
}
```

## Prisma 기본 CRUD 작업

### 1. Create (생성)

```typescript
// 단일 생성
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
  },
});

// 여러 개 생성
const users = await prisma.user.createMany({
  data: [
    { email: 'alice@example.com', name: 'Alice' },
    { email: 'bob@example.com', name: 'Bob' },
  ],
});

// 관계와 함께 생성
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John',
    posts: {
      create: [
        { title: 'First Post', body: 'Content...' },
        { title: 'Second Post', body: 'More content...' },
      ],
    },
  },
  include: {
    posts: true, // 관계 데이터도 함께 반환
  },
});
```

### 2. Read (조회)

```typescript
// 기본 조회
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' },
});

// 또는
const user = await prisma.user.findUnique({
  where: { id: 1 },
});

// 존재하면 반환, 없으면 null
const user = await prisma.user.findFirst({
  where: { name: 'John' },
});

// 여러 개 조회
const users = await prisma.user.findMany({
  where: {
    age: { gte: 18 }, // 18살 이상
  },
  orderBy: {
    createdAt: 'desc',
  },
  skip: 0,
  take: 10, // 페이지네이션
});

// 관계 데이터와 함께
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
    },
  },
});

// 개수 조회
const count = await prisma.user.count({
  where: { age: { gte: 18 } },
});
```

### 3. Update (수정)

```typescript
// 기본 수정
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Jane Doe',
    age: 30,
  },
});

// 조건부 수정 (많은 레코드)
const result = await prisma.user.updateMany({
  where: { age: { lt: 18 } }, // 18살 미만
  data: { role: 'GUEST' },
});

// 수정 또는 생성
const user = await prisma.user.upsert({
  where: { email: 'john@example.com' },
  update: { name: 'John Doe' },
  create: { email: 'john@example.com', name: 'John Doe' },
});

// 관계 수정
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    posts: {
      create: { title: 'New Post', body: 'Content' },
      deleteMany: { published: false },
    },
  },
});
```

### 4. Delete (삭제)

```typescript
// 기본 삭제
const user = await prisma.user.delete({
  where: { id: 1 },
});

// 여러 개 삭제
const result = await prisma.user.deleteMany({
  where: { age: { lt: 18 } },
});

// 모두 삭제
const result = await prisma.user.deleteMany();
```

## 마이그레이션

### 스키마 변경

```prisma
// 기존 스키마
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String
}

// 새로운 필드 추가
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String
  age   Int?    // 새로운 필드
  role  Role    @default(USER)  // 새로운 필드
}

enum Role {
  ADMIN
  USER
}
```

### 마이그레이션 실행

```bash
# 마이그레이션 생성 (스키마 변경 감지)
npx prisma migrate dev --name add_age_and_role

# 마이그레이션 적용
npx prisma migrate deploy

# 마이그레이션 상태 확인
npx prisma migrate status

# 마이그레이션 재설정 (개발 환경만)
npx prisma migrate reset
```

생성되는 파일:

```
prisma/migrations/
├── 20250129_add_age_and_role/
│   ├── migration.sql
│   └── migration_lock.toml
```

## 프로덕션 레벨 구현

### 1. 타입 안전한 서비스 레이어

```typescript
// src/services/userService.ts
import { prisma } from '@/lib/prisma';
import { User, Prisma } from '@prisma/client';

export class UserService {
  // 사용자 생성
  async createUser(data: Prisma.UserCreateInput): Promise<User> {
    try {
      return await prisma.user.create({ data });
    } catch (error) {
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        if (error.code === 'P2002') {
          throw new Error('Email already exists');
        }
      }
      throw error;
    }
  }

  // 사용자 조회
  async getUser(id: number): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id },
      include: { posts: true },
    });
  }

  // 사용자 목록 (페이지네이션)
  async listUsers(page: number = 1, limit: number = 10) {
    const skip = (page - 1) * limit;

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count(),
    ]);

    return {
      users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  // 사용자 수정
  async updateUser(id: number, data: Prisma.UserUpdateInput): Promise<User> {
    return prisma.user.update({
      where: { id },
      data,
      include: { posts: true },
    });
  }

  // 사용자 삭제
  async deleteUser(id: number): Promise<User> {
    return prisma.user.delete({
      where: { id },
    });
  }
}

export const userService = new UserService();
```

### 2. API 라우트에서 사용

```typescript
// src/routes/users.ts
import express from 'express';
import { userService } from '@/services/userService';

const router = express.Router();

// GET /users
router.get('/users', async (req, res) => {
  try {
    const page = parseInt(req.query.page as string) || 1;
    const result = await userService.listUsers(page);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST /users
router.post('/users', async (req, res) => {
  try {
    const user = await userService.createUser(req.body);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// GET /users/:id
router.get('/users/:id', async (req, res) => {
  try {
    const user = await userService.getUser(parseInt(req.params.id));
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// PUT /users/:id
router.put('/users/:id', async (req, res) => {
  try {
    const user = await userService.updateUser(
      parseInt(req.params.id),
      req.body,
    );
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// DELETE /users/:id
router.delete('/users/:id', async (req, res) => {
  try {
    await userService.deleteUser(parseInt(req.params.id));
    res.status(204).send();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default router;
```

### 3. 트랜잭션 처리

```typescript
// 복잡한 다중 작업 (트랜잭션)
async function transferMoney(
  fromUserId: number,
  toUserId: number,
  amount: number,
) {
  const result = await prisma.$transaction(async (tx) => {
    // 1단계: 보내는 사용자 확인
    const fromUser = await tx.user.findUnique({
      where: { id: fromUserId },
    });

    if (!fromUser || fromUser.balance < amount) {
      throw new Error('Insufficient balance');
    }

    // 2단계: 받는 사용자 확인
    const toUser = await tx.user.findUnique({
      where: { id: toUserId },
    });

    if (!toUser) {
      throw new Error('Recipient not found');
    }

    // 3단계: 송금
    const [updated1, updated2] = await Promise.all([
      tx.user.update({
        where: { id: fromUserId },
        data: { balance: { decrement: amount } },
      }),
      tx.user.update({
        where: { id: toUserId },
        data: { balance: { increment: amount } },
      }),
    ]);

    // 4단계: 거래 기록
    await tx.transaction.create({
      data: {
        fromUserId,
        toUserId,
        amount,
      },
    });

    return { from: updated1, to: updated2 };
  });

  return result;
}

// 사용
try {
  const result = await transferMoney(1, 2, 100);
  console.log('Transfer successful:', result);
} catch (error) {
  console.error('Transfer failed:', error.message);
}
```

### 4. Prisma Studio (GUI)

```bash
# 웹 기반 DB 관리 도구 실행
npx prisma studio

# localhost:5555에서 브라우저로 확인
# - 데이터 조회
# - 데이터 생성/수정/삭제
# - 관계 확인
```

## 고급 기능

### 1. Raw 쿼리 (SQL 직접 실행)

```typescript
// Prisma만으로 복잡한 쿼리를 작성하기 어려울 때
const result = await prisma.$queryRaw`
  SELECT u.id, u.name, COUNT(p.id) as post_count
  FROM users u
  LEFT JOIN posts p ON u.id = p.user_id
  GROUP BY u.id
`;

// Raw 데이터 수정
const result = await prisma.$executeRaw`
  UPDATE users SET role = 'ADMIN' WHERE age >= 21
`;
```

### 2. 전역 필터 (Middleware)

```typescript
// 모든 쿼리에 자동으로 삭제된 항목 제외
prisma.$use(async (params, next) => {
  if (params.model === 'Post' && params.action === 'findMany') {
    params.args.where = {
      ...params.args.where,
      deletedAt: null,
    };
  }
  return next(params);
});
```

### 3. 프리로딩 최적화

```typescript
// N+1 문제 방지
const users = await prisma.user.findMany({
  include: {
    posts: {
      include: {
        comments: true,
      },
    },
  },
});
```

## 다른 ORM과의 비교

| 기능             | Prisma         | TypeORM   | Sequelize |
| ---------------- | -------------- | --------- | --------- |
| **타입 안전성**  | ✅ 최고        | ✅ 좋음   | ❌ 약함   |
| **자동완성**     | ✅ 완벽        | ❌ 부분   | ❌ 약함   |
| **마이그레이션** | ✅ 우수        | ✅ 좋음   | ✅ 좋음   |
| **배우기 쉬움**  | ✅ 매우 쉬움   | ❌ 어려움 | ❌ 어려움 |
| **성능**         | ✅ 매우 좋음   | ✅ 좋음   | ⚠️ 중간   |
| **커뮤니티**     | ✅ 빠르게 증가 | ✅ 중간   | ✅ 크다   |
| **GUI 도구**     | ✅ Studio      | ❌        | ❌        |

## 팀 협업 가이드

```markdown
# Prisma 개발 규칙

## 데이터베이스 변경

### 스키마 수정 과정

1. prisma/schema.prisma 수정
2. `npx prisma migrate dev --name <description>` 실행
3. migration 파일 생성됨
4. PR 제출 (migration 파일 포함)
5. 코드 리뷰
6. merge 후 자동으로 적용

### 마이그레이션 파일 이름 규칙
```

npx prisma migrate dev --name add_user_age
npx prisma migrate dev --name create_post_table
npx prisma migrate dev --name add_unique_email

```

## 서비스 레이어 구조

모든 데이터베이스 접근은 서비스 레이어를 통해:
```

API Route → Service → Prisma

````

## 타입 정의

자동 생성되는 타입 사용:
```typescript
import { User, Post } from '@prisma/client';
````

## 테스트

```bash
# 테스트 데이터베이스 사용
DATABASE_URL="postgresql://test:test@localhost/test_db" npm test
```

## 롤백 방법

```bash
# 이전 마이그레이션으로 돌아가기
npx prisma migrate resolve --rolled-back <migration_name>

# 개발 환경 초기화
npx prisma migrate reset
```

```

## 결론

Prisma는:

✅ **타입 안전**: TypeScript와 완벽하게 통합
✅ **생산성**: SQL을 쓸 필요가 없음
✅ **유지보수**: 마이그레이션이 버전 관리됨
✅ **학습 곡선**: 매우 낮음
✅ **개발 경험**: 최고의 DX (Developer Experience)

특히 **Node.js/TypeScript 프로젝트**에서는 Prisma가 정답입니다.

**지금 바로 Prisma를 시작하세요!**
```

## 참고 링크

- [Prisma 공식 사이트](https://www.prisma.io)
- [Prisma 공식 문서](https://www.prisma.io/docs)
- [Prisma GitHub 저장소](https://github.com/prisma/prisma)
