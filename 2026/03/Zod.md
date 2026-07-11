# Zod - TypeScript 우선의 완벽한 스키마 검증

## 들어가며

API 응답 데이터를 받을 때 타입을 어떻게 보장하나요?

```typescript
// ❌ 위험한 코드
const response = await fetch('/api/user');
const user = await response.json();
console.log(user.email); // user가 정말 객체인지? email 필드가 있는지?

// ❌ 런타임 에러 발생 가능
// TypeError: Cannot read property 'email' of undefined
```

**Zod**는 런타임에서 데이터의 형태를 검증하고, TypeScript 타입을 자동으로 생성합니다. 완벽한 타입 안전성을 보장합니다.

## 공식 사이트

https://zod.dev

# 1. Zod란?

## 핵심 개념

```
Zod = TypeScript 우선의 스키마 검증 라이브러리

특징:
✅ TypeScript 네이티브
✅ 타입 추론 자동
✅ 구성 가능한 API
✅ 작은 번들 크기 (13KB)
✅ 런타임 검증
✅ 상세한 에러 메시지
```

## Zod vs Yup vs Joi

```
Zod:
- TypeScript 우선
- 더 나은 타입 추론
- 더 간단한 API
- 더 작은 번들
- 최신 트렌드

Yup:
- 전통적
- Formik과 많이 사용
- 충분히 기능 많음
- 좋은 문서

Joi:
- Node.js 중심
- 가장 완전한 기능
- 무거운 번들
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install zod
```

의존성이 없어서 설치가 간단합니다.

# 3. 기본 타입 검증

## 기본 타입들

```typescript
import { z } from 'zod';

// String
const stringSchema = z.string();
stringSchema.parse('hello'); // ✅ 성공
stringSchema.parse(123); // ❌ 에러

// Number
const numberSchema = z.number();
numberSchema.parse(42); // ✅ 성공
numberSchema.parse('42'); // ❌ 에러

// Boolean
const boolSchema = z.boolean();
boolSchema.parse(true); // ✅ 성공
boolSchema.parse('true'); // ❌ 에러

// Array
const arraySchema = z.array(z.string());
arraySchema.parse(['a', 'b', 'c']); // ✅ 성공
arraySchema.parse(['a', 2, 'c']); // ❌ 에러

// Enum
const roleSchema = z.enum(['admin', 'user', 'guest']);
roleSchema.parse('admin'); // ✅ 성공
roleSchema.parse('superuser'); // ❌ 에러

// Optional
const optionalSchema = z.string().optional();
optionalSchema.parse('hello'); // ✅ 성공
optionalSchema.parse(undefined); // ✅ 성공
optionalSchema.parse(null); // ❌ 에러

// Nullable
const nullableSchema = z.string().nullable();
nullableSchema.parse('hello'); // ✅ 성공
nullableSchema.parse(null); // ✅ 성공
nullableSchema.parse(undefined); // ❌ 에러

// Union
const unionSchema = z.union([z.string(), z.number()]);
unionSchema.parse('hello'); // ✅ 성공
unionSchema.parse(42); // ✅ 성공
unionSchema.parse(true); // ❌ 에러
```

# 4. 객체 검증

## 기본 객체

```typescript
import { z } from 'zod';

// 간단한 사용자 스키마
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().int().positive(),
});

// 데이터 검증
const userData = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 25,
};

const result = userSchema.parse(userData);
console.log(result); // ✅ 성공

// TypeScript 타입 자동 생성
type User = z.infer<typeof userSchema>;
// type User = {
//   id: number
//   name: string
//   email: string
//   age: number
// }

// 타입 자동 완성
const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 25,
};
```

## 중첩된 객체

```typescript
import { z } from 'zod';

const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string(),
});

const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  address: addressSchema, // 중첩된 스키마
});

const userData = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001',
  },
};

const result = userSchema.parse(userData);
// ✅ 성공

// 타입도 자동 생성
type User = z.infer<typeof userSchema>;
// type User = {
//   id: number
//   name: string
//   email: string
//   address: {
//     street: string
//     city: string
//     zipCode: string
//   }
// }
```

# 5. 상세한 검증

## String 검증

```typescript
import { z } from 'zod';

const emailSchema = z
  .string()
  .email('Invalid email format')
  .min(5, 'Email must be at least 5 characters')
  .max(255, 'Email must be less than 255 characters')
  .toLowerCase();

emailSchema.parse('JOHN@EXAMPLE.COM');
// 결과: 'john@example.com'

// URL 검증
const urlSchema = z.string().url('Invalid URL');
urlSchema.parse('https://example.com'); // ✅ 성공

// 정규표현식
const usernameSchema = z
  .string()
  .regex(
    /^[a-zA-Z0-9_]+$/,
    'Username can only contain letters, numbers, and underscores',
  )
  .min(3)
  .max(20);

// 고정 길이
const pinSchema = z.string().length(4, 'PIN must be exactly 4 characters');
```

## Number 검증

```typescript
import { z } from 'zod';

const ageSchema = z
  .number()
  .int('Age must be an integer')
  .min(0, 'Age cannot be negative')
  .max(150, 'Age must be realistic')
  .refine((age) => age >= 18, 'Must be 18 or older');

ageSchema.parse(25); // ✅ 성공
ageSchema.parse(15); // ❌ 에러

// 범위
const scoreSchema = z
  .number()
  .min(0)
  .max(100)
  .multipleOf(0.5, 'Score must be multiple of 0.5');

scoreSchema.parse(85.5); // ✅ 성공
scoreSchema.parse(85.3); // ❌ 에러
```

## Array 검증

```typescript
import { z } from 'zod';

// 배열 요소 검증
const tagsSchema = z.array(z.string()).nonempty('At least one tag is required');

tagsSchema.parse(['javascript', 'typescript']); // ✅ 성공
tagsSchema.parse([]); // ❌ 에러

// 배열 길이 검증
const itemsSchema = z
  .array(z.number())
  .min(1, 'At least 1 item required')
  .max(10, 'Maximum 10 items allowed');

itemsSchema.parse([1, 2, 3]); // ✅ 성공
itemsSchema.parse([]); // ❌ 에러

// 중복 제거
const uniqueTagsSchema = z
  .array(z.string())
  .refine((tags) => new Set(tags).size === tags.length, 'Tags must be unique');

uniqueTagsSchema.parse(['js', 'ts']); // ✅ 성공
uniqueTagsSchema.parse(['js', 'js']); // ❌ 에러
```

# 6. 커스텀 검증

## refine 사용

```typescript
import { z } from 'zod';

const passwordSchema = z
  .object({
    password: z
      .string()
      .min(8, 'Password must be at least 8 characters')
      .refine(
        (password) => /[A-Z]/.test(password),
        'Password must contain uppercase letter',
      )
      .refine(
        (password) => /[0-9]/.test(password),
        'Password must contain number',
      )
      .refine(
        (password) => /[!@#$%^&*]/.test(password),
        'Password must contain special character',
      ),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords do not match',
    path: ['confirmPassword'], // 에러 위치 지정
  });

const result = passwordSchema.parse({
  password: 'SecurePass123!',
  confirmPassword: 'SecurePass123!',
}); // ✅ 성공
```

## superRefine 사용 (더 세밀한 제어)

```typescript
import { z } from 'zod';

const userSchema = z
  .object({
    email: z.string().email(),
    age: z.number(),
  })
  .superRefine(async (data, ctx) => {
    // 비동기 검증 가능
    const userExists = await checkUserExists(data.email);

    if (userExists) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        path: ['email'],
        message: 'User already exists',
      });
    }

    // 복잡한 조건 검증
    if (data.age < 18 && data.age > 13) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        path: ['age'],
        message: 'Must be 18 or 13 and under',
      });
    }
  });
```

# 7. Transform 사용

## 데이터 변환

```typescript
import { z } from 'zod';

// 문자열을 숫자로 변환
const numberSchema = z.string().transform((val) => parseInt(val, 10));
const result = numberSchema.parse('42');
console.log(result); // 42 (number)

// 이메일 소문자 변환
const emailSchema = z
  .string()
  .email()
  .transform((val) => val.toLowerCase());
const result = emailSchema.parse('JOHN@EXAMPLE.COM');
console.log(result); // 'john@example.com'

// 복잡한 변환
const userSchema = z
  .object({
    firstName: z.string(),
    lastName: z.string(),
    email: z.string().email(),
  })
  .transform((data) => ({
    fullName: `${data.firstName} ${data.lastName}`,
    email: data.email.toLowerCase(),
  }));

const result = userSchema.parse({
  firstName: 'John',
  lastName: 'Doe',
  email: 'JOHN@EXAMPLE.COM',
});

console.log(result);
// {
//   fullName: 'John Doe',
//   email: 'john@example.com'
// }

// 타입도 자동 업데이트
type User = z.infer<typeof userSchema>;
// type User = {
//   fullName: string
//   email: string
// }
```

# 8. 에러 처리

## 기본 에러 처리

```typescript
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  age: z.number().int().positive(),
});

try {
  userSchema.parse({
    email: 'invalid-email',
    age: 'not-a-number',
  });
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log('Validation errors:');
    error.errors.forEach((err) => {
      console.log(`${err.path.join('.')}: ${err.message}`);
    });
    // 출력:
    // email: Invalid email
    // age: Expected number, received string
  }
}
```

## 상세한 에러 정보

```typescript
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email format'),
  age: z.number().min(18, 'Must be 18 or older'),
});

const result = userSchema.safeParse({
  email: 'invalid',
  age: 15,
});

if (!result.success) {
  console.log(result.error.format());
  // {
  //   email: { _errors: ['Invalid email format'] },
  //   age: { _errors: ['Must be 18 or older'] }
  // }

  // 모든 에러 상세 정보
  result.error.errors.forEach((error) => {
    console.log({
      path: error.path, // ['email']
      code: error.code, // 'invalid_string'
      message: error.message, // 'Invalid email format'
      received: error.received, // 'invalid'
    });
  });
}
```

## parse vs safeParse

```typescript
import { z } from 'zod';

const schema = z.number();

// parse - 에러 시 예외 발생
try {
  const result = schema.parse('not a number');
} catch (error) {
  console.error('Error:', error);
}

// safeParse - 에러를 반환 (try-catch 불필요)
const result = schema.safeParse('not a number');

if (result.success) {
  console.log('Valid:', result.data);
} else {
  console.error('Invalid:', result.error.format());
}
```

# 9. 실전 예제

## API 응답 검증

```typescript
import { z } from 'zod';

// API 응답 스키마
const userResponseSchema = z.object({
  id: z.number(),
  email: z.string().email(),
  name: z.string(),
  createdAt: z.string().datetime(),
  isActive: z.boolean(),
});

// API 호출
async function getUser(userId: string) {
  const response = await fetch(`/api/users/${userId}`);
  const data = await response.json();

  // 응답 검증
  const user = userResponseSchema.parse(data);

  // 이제 user의 타입이 완벽하게 보장됨
  console.log(user.id); // ✅ number
  console.log(user.email); // ✅ string
}

// 또는 안전하게
async function getUserSafe(userId: string) {
  const response = await fetch(`/api/users/${userId}`);
  const data = await response.json();

  const result = userResponseSchema.safeParse(data);

  if (result.success) {
    return result.data;
  } else {
    console.error('Invalid response:', result.error);
    return null;
  }
}
```

## 폼 제출 검증

```typescript
import { z } from 'zod';

const signupSchema = z
  .object({
    username: z
      .string()
      .min(3, 'Username must be at least 3 characters')
      .max(20, 'Username must be less than 20 characters')
      .regex(
        /^[a-zA-Z0-9_]+$/,
        'Username can only contain letters, numbers, and underscores',
      ),

    email: z.string().email('Invalid email format'),

    password: z
      .string()
      .min(8, 'Password must be at least 8 characters')
      .refine(
        (pwd) => /[A-Z]/.test(pwd),
        'Password must contain uppercase letter',
      )
      .refine((pwd) => /[0-9]/.test(pwd), 'Password must contain number'),

    confirmPassword: z.string(),

    age: z
      .number()
      .int('Age must be integer')
      .min(18, 'Must be 18 or older')
      .max(120, 'Age must be realistic'),

    terms: z.boolean().refine((val) => val === true, 'You must agree to terms'),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords do not match',
    path: ['confirmPassword'],
  });

// 폼 제출 처리
async function handleSignup(formData: unknown) {
  const result = signupSchema.safeParse(formData);

  if (!result.success) {
    // 검증 실패 - 에러 반환
    return {
      success: false,
      errors: result.error.format(),
    };
  }

  // 검증 성공 - 서버에 전송
  const response = await fetch('/api/signup', {
    method: 'POST',
    body: JSON.stringify(result.data),
  });

  return {
    success: true,
    data: await response.json(),
  };
}
```

## 데이터베이스 검증

```typescript
import { z } from 'zod';

const userSchema = z.object({
  id: z.number().int().positive(),
  email: z.string().email(),
  name: z.string().min(1),
  age: z.number().int().positive().optional(),
  tags: z.array(z.string()).default([]),
  role: z.enum(['admin', 'user', 'guest']).default('user'),
  createdAt: z.date().default(() => new Date()),
  updatedAt: z.date(),
});

// 데이터베이스에서 조회한 데이터 검증
async function getUserFromDB(id: number) {
  const row = await db.query('SELECT * FROM users WHERE id = ?', [id]);

  // 데이터 검증
  const user = userSchema.parse(row);

  return user;
}

// 일괄 검증
async function getAllUsers() {
  const rows = await db.query('SELECT * FROM users');

  // 배열 검증
  const users = z.array(userSchema).parse(rows);

  return users;
}
```

# 10. React Hook Form과 통합

## React Hook Form + Zod

```typescript
import { useForm } from 'react-hook-form'
import { z } from 'zod'
import { zodResolver } from '@hookform/resolvers/zod'

// Zod 스키마
const loginSchema = z.object({
  email: z.string()
    .min(1, 'Email is required')
    .email('Invalid email'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
})

// 타입 추론
type LoginFormData = z.infer<typeof loginSchema>

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema)
  })

  const onSubmit = (data: LoginFormData) => {
    console.log(data) // 타입 완벽 보장
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email')}
        placeholder="Email"
      />
      {errors.email && <p>{errors.email.message}</p>}

      <input
        {...register('password')}
        type="password"
        placeholder="Password"
      />
      {errors.password && <p>{errors.password.message}</p>}

      <button type="submit">Login</button>
    </form>
  )
}
```

# 11. 고급 기능

## Discriminated Union

```typescript
import { z } from 'zod';

// 결제 방법에 따른 다른 필드
const paymentSchema = z.discriminatedUnion('method', [
  z.object({
    method: z.literal('card'),
    cardNumber: z.string().length(16),
    cvv: z.string().length(3),
  }),
  z.object({
    method: z.literal('bank'),
    accountNumber: z.string(),
    bankCode: z.string(),
  }),
  z.object({
    method: z.literal('paypal'),
    email: z.string().email(),
  }),
]);

const result1 = paymentSchema.parse({
  method: 'card',
  cardNumber: '1234567890123456',
  cvv: '123',
}); // ✅ 성공

const result2 = paymentSchema.parse({
  method: 'paypal',
  email: 'john@example.com',
}); // ✅ 성공

// 타입도 자동 분별
type Payment = z.infer<typeof paymentSchema>;
// type Payment =
//   | { method: 'card'; cardNumber: string; cvv: string }
//   | { method: 'bank'; accountNumber: string; bankCode: string }
//   | { method: 'paypal'; email: string }
```

## Generics

```typescript
import { z } from 'zod';

// 재사용 가능한 페이지네이션 스키마
function createPaginationSchema<T extends z.ZodTypeAny>(itemSchema: T) {
  return z.object({
    items: z.array(itemSchema),
    total: z.number(),
    page: z.number().int().positive(),
    pageSize: z.number().int().positive(),
  });
}

const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
});

const usersPageSchema = createPaginationSchema(userSchema);

type UsersPage = z.infer<typeof usersPageSchema>;
// type UsersPage = {
//   items: Array<{ id: number; name: string; email: string }>
//   total: number
//   page: number
//   pageSize: number
// }
```

# 12. 팀 협업 Best Practices

## 스키마 분리 및 관리

```typescript
// schemas/user.ts
import { z } from 'zod';

export const userBaseSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().int().min(0),
});

export const createUserSchema = userBaseSchema.strict();

export const updateUserSchema = userBaseSchema.partial();

export const userResponseSchema = userBaseSchema.extend({
  id: z.number(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

export type User = z.infer<typeof userResponseSchema>;
export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;

// schemas/index.ts
export * from './user';
export * from './product';
export * from './order';
```

# 13. 자주 묻는 질문

## Q: TypeScript 타입과 Zod 스키마를 두 번 작성해야 하나?

**A:** 아니요! Zod 스키마에서 타입을 추론합니다.

```typescript
// ❌ 나쁜 방법 (중복)
interface User {
  name: string;
  email: string;
  age: number;
}

const userSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  age: z.number(),
});

// ✅ 좋은 방법 (한 번만)
const userSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  age: z.number(),
});

type User = z.infer<typeof userSchema>;
```

## Q: 백엔드와 프론트엔드에서 같은 스키마를 사용할 수 있나?

**A:** 네, monorepo에서 공유 가능합니다.

```
packages/
├── shared/
│   └── schemas/
│       └── user.ts  (Zod 스키마)
├── frontend/
└── backend/
```

## Q: null과 undefined의 차이?

**A:**

```typescript
const schema1 = z.string().optional(); // undefined 허용
schema1.parse(undefined); // ✅ 성공
schema1.parse(null); // ❌ 에러

const schema2 = z.string().nullable(); // null 허용
schema2.parse(null); // ✅ 성공
schema2.parse(undefined); // ❌ 에러

const schema3 = z.string().nullish(); // null과 undefined 모두 허용
schema3.parse(null); // ✅ 성공
schema3.parse(undefined); // ✅ 성공
```

# 14. 체크리스트

Zod 도입하기:

```
[ ] zod 설치
[ ] 기본 스키마 작성
[ ] 객체 스키마 작성
[ ] 검증 규칙 추가
[ ] 커스텀 검증 구현
[ ] API 응답 검증
[ ] React Hook Form과 통합
[ ] 에러 처리 구현
[ ] 스키마 모듈화
[ ] 팀 협업 규칙 정의
```

# 결론

Zod는:

✅ TypeScript 우선 설계
✅ 런타임 검증으로 타입 안전성
✅ 자동 타입 추론
✅ 작은 번들 크기
✅ 직관적인 API
✅ 뛰어난 에러 메시지

**TypeScript 프로젝트에서 데이터 검증은 Zod로 시작하세요!**

## 참고 링크

- [Zod 공식 사이트](https://zod.dev)
- [Zod GitHub](https://github.com/colinhacks/zod)
