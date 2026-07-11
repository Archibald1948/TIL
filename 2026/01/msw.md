# MSW (Mock Service Worker) - 네트워크 레벨 API 모킹

## 들어가며

당신이 프론트엔드 개발 중에 백엔드 API가 완성되기를 기다린 적이 있나요?
또는 백엔드 개발자가 특정 에러 상황을 재현하기 어려워 했나요?

MSW(Mock Service Worker)는 이 모든 문제를 해결합니다.
Service Worker API를 사용하여 **네트워크 레벨에서** API 요청을 가로채고, 정의된 핸들러가 응답을 제공합니다.
이는 단순한 모킹을 넘어, 실제 네트워크 요청처럼 작동합니다.

## 공식 사이트

https://github.com/mswjs/msw

## MSW의 핵심 개념

### 기존 방식 vs MSW

```
기존 방식 (fetch 모킹):
┌─ 테스트 코드
├─ jest.mock('/api/users', ...)
├─ 매번 설정 필요
└─ 부분적 모킹만 가능

MSW (네트워크 레벨 모킹):
┌─ 애플리케이션
├─ fetch / axios / 등 모든 HTTP 라이브러리
├─ Service Worker (MSW)
├─ 정의된 핸들러
└─ 응답 반환

장점:
- 라이브러리 무관 (fetch든 axios든 상관없음)
- 한 번 정의하면 모든 테스트/개발에 사용
- 실제 네트워크처럼 작동
```

### MSW가 해결하는 문제

```txt
// ❌ 문제 1: API 완성 대기
// 백엔드가 아직 구현 중이면 프론트엔드는 대기
// 월요일: API 문서 받음
// 목요일: 실제 API 완성
// → 4일 낭비

// ✅ MSW로 해결
// 월요일: API 문서 기반 MSW 핸들러 작성
// 화요일: 프론트엔드 개발 시작
// 목요일: 실제 API와 통합 (핸들러만 제거)

// ❌ 문제 2: 에러 상황 테스트 어려움
// 404, 500 에러는 언제 발생할까?
// 프로덕션 데이터로는 테스트 불가

// ✅ MSW로 해결
// scenarios.errorFlow() // 500 에러 시나리오
// scenarios.notFound()  // 404 에러 시나리오
// 원하는 만큼 테스트 가능

// ❌ 문제 3: 네트워크 지연 테스트
// 3G 환경은 어떻게?
// 느린 네트워크에서 로딩 상태가 제대로 표시되나?

// ✅ MSW로 해결
// await new Promise(r => setTimeout(r, 2000));  // 2초 지연
// 로딩 상태 테스트 가능
```

## MSW 설치 및 기본 설정

### 1. 설치

```bash
npm install msw --save-dev
```

### 2. 기본 핸들러 정의

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // GET /api/users/:id
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params;

    // 특정 사용자에 대한 mock 응답
    return HttpResponse.json({
      id,
      name: 'John Doe',
      email: 'john@example.com',
      avatar: 'https://example.com/avatar.jpg',
    });
  }),

  // POST /api/users
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();

    // 요청 본문을 기반으로 응답
    return HttpResponse.json(
      {
        id: '123',
        ...body,
        createdAt: new Date().toISOString(),
      },
      { status: 201 }, // 201 Created
    );
  }),

  // DELETE /api/users/:id
  http.delete('/api/users/:id', ({ params }) => {
    return HttpResponse.json({ message: 'User deleted' }, { status: 204 });
  }),

  // 에러 응답
  http.get('/api/users/999', () => {
    return HttpResponse.json({ error: 'User not found' }, { status: 404 });
  }),
];
```

### 3. 브라우저 환경 설정

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);

// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// 개발 환경에서만 MSW 활성화
if (process.env.NODE_ENV === 'development') {
  const { worker } = await import('./mocks/browser');
  await worker.start({
    // 경고 비활성화 (선택사항)
    onUnhandledRequest: 'bypass'
  });
}

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### 4. Node.js 테스트 환경 설정

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/__tests__/setup.ts
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from '../mocks/server';

// 모든 테스트 시작 전 서버 시작
beforeAll(() => server.listen());

// 각 테스트 후 핸들러 초기화
afterEach(() => server.resetHandlers());

// 모든 테스트 완료 후 서버 종료
afterAll(() => server.close());
```

## 실전: 복잡한 시나리오 관리

### 1. 핸들러 구조화

```typescript
// src/mocks/handlers/users.ts
import { http, HttpResponse } from 'msw';

export const userHandlers = [
  // 사용자 목록
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'Alice', email: 'alice@example.com' },
      { id: '2', name: 'Bob', email: 'bob@example.com' },
      { id: '3', name: 'Charlie', email: 'charlie@example.com' },
    ]);
  }),

  // 개별 사용자
  http.get('/api/users/:id', ({ params }) => {
    const users: Record<string, any> = {
      '1': { id: '1', name: 'Alice', email: 'alice@example.com' },
      '2': { id: '2', name: 'Bob', email: 'bob@example.com' },
    };

    if (params.id in users) {
      return HttpResponse.json(users[params.id]);
    }

    return HttpResponse.json({ error: 'User not found' }, { status: 404 });
  }),

  // 사용자 생성
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();

    // 유효성 검증
    if (!body.email || !body.name) {
      return HttpResponse.json(
        { error: 'Missing required fields' },
        { status: 400 },
      );
    }

    return HttpResponse.json(
      {
        id: Math.random().toString(),
        ...body,
        createdAt: new Date().toISOString(),
      },
      { status: 201 },
    );
  }),
];

// src/mocks/handlers/products.ts
import { http, HttpResponse } from 'msw';

export const productHandlers = [
  // 상품 목록
  http.get('/api/products', ({ request }) => {
    const url = new URL(request.url);
    const category = url.searchParams.get('category');

    const products = [
      { id: '1', name: 'Product 1', category: 'electronics', price: 100 },
      { id: '2', name: 'Product 2', category: 'books', price: 20 },
      { id: '3', name: 'Product 3', category: 'electronics', price: 200 },
    ];

    if (category) {
      return HttpResponse.json(products.filter((p) => p.category === category));
    }

    return HttpResponse.json(products);
  }),

  // 상품 검색
  http.get('/api/products/search', ({ request }) => {
    const url = new URL(request.url);
    const query = url.searchParams.get('q');

    if (!query) {
      return HttpResponse.json(
        { error: 'Query parameter required' },
        { status: 400 },
      );
    }

    return HttpResponse.json([
      { id: '1', name: `Product matching ${query}`, price: 100 },
    ]);
  }),
];

// src/mocks/handlers/index.ts
import { userHandlers } from './users';
import { productHandlers } from './products';

export const handlers = [...userHandlers, ...productHandlers];
```

### 2. 시나리오 관리

```typescript
// src/mocks/scenarios.ts
import { server } from './server';
import { http, HttpResponse } from 'msw';

export const scenarios = {
  // 정상 흐름
  normalFlow: () => {
    server.use(
      http.get('/api/users/1', () =>
        HttpResponse.json({ id: '1', name: 'John' }),
      ),
    );
  },

  // 네트워크 오류
  networkError: () => {
    server.use(
      http.get('/api/users/1', () => {
        return HttpResponse.error();
      }),
    );
  },

  // 타임아웃
  timeout: () => {
    server.use(
      http.get('/api/users/1', async () => {
        // 요청이 타임아웃될 때까지 대기
        await new Promise((r) => setTimeout(r, 10000));
        return HttpResponse.json({ id: '1' });
      }),
    );
  },

  // 느린 네트워크 (3G)
  slowNetwork: () => {
    server.use(
      http.get('/api/users/1', async () => {
        await new Promise((r) => setTimeout(r, 2000));
        return HttpResponse.json({ id: '1', name: 'John' });
      }),
    );
  },

  // 서버 오류
  serverError: () => {
    server.use(
      http.get('/api/users/1', () =>
        HttpResponse.json({ error: 'Internal Server Error' }, { status: 500 }),
      ),
    );
  },

  // 인증 오류
  unauthorized: () => {
    server.use(
      http.get('/api/users/1', () =>
        HttpResponse.json({ error: 'Unauthorized' }, { status: 401 }),
      ),
    );
  },

  // 데이터 검증 오류
  validationError: () => {
    server.use(
      http.post('/api/users', () =>
        HttpResponse.json(
          {
            error: 'Validation failed',
            details: {
              email: 'Invalid email format',
              name: 'Name is required',
            },
          },
          { status: 422 },
        ),
      ),
    );
  },
};

// 테스트에서 사용
describe('User API', () => {
  it('should fetch user successfully', async () => {
    scenarios.normalFlow();
    // 테스트
  });

  it('should handle 500 error', async () => {
    scenarios.serverError();
    // 에러 처리 테스트
  });

  it('should handle network error', async () => {
    scenarios.networkError();
    // 네트워크 오류 처리 테스트
  });

  it('should show loading during slow network', async () => {
    scenarios.slowNetwork();
    // 로딩 상태 표시 테스트
  });
});
```

## 프로덕션 레벨 구현

### 1. API Contract 문서화

```yaml
# api-contract.yaml

endpoints:
  GET /api/users/:id:
    description: 사용자 조회
    parameters:
      id:
        type: string
        required: true
        example: '123'

    responses:
      200:
        description: 성공
        body:
          id: string
          name: string
          email: string
          avatar?: string
          createdAt: ISO8601
        example:
          id: '123'
          name: 'John Doe'
          email: 'john@example.com'
          createdAt: '2025-01-29T10:00:00Z'

      404:
        description: 사용자 없음
        body:
          error: string
        example:
          error: 'User not found'

  POST /api/users:
    description: 사용자 생성
    requestBody:
      name: string (required)
      email: string (required)
      avatar?: string

    responses:
      201:
        description: 생성 성공
        body:
          id: string
          name: string
          email: string
          createdAt: ISO8601

      400:
        description: 유효성 검증 오류
        body:
          error: string
          details?: object
```

```typescript
// MSW 핸들러로 변환
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users/:id', ({ params }) => {
    if (!params.id || params.id === '999') {
      return HttpResponse.json({ error: 'User not found' }, { status: 404 });
    }

    return HttpResponse.json({
      id: params.id,
      name: 'John Doe',
      email: 'john@example.com',
      createdAt: new Date().toISOString(),
    });
  }),

  http.post('/api/users', async ({ request }) => {
    const body = await request.json();

    if (!body.name || !body.email) {
      return HttpResponse.json(
        {
          error: 'Validation failed',
          details: {
            name: body.name ? undefined : 'Required',
            email: body.email ? undefined : 'Required',
          },
        },
        { status: 400 },
      );
    }

    return HttpResponse.json(
      {
        id: Math.random().toString(),
        ...body,
        createdAt: new Date().toISOString(),
      },
      { status: 201 },
    );
  }),
];
```

### 2. 테스트 예제

```typescript
// src/__tests__/user.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { server } from '../mocks/server';
import { scenarios } from '../mocks/scenarios';

describe('User Component', () => {
  beforeEach(() => {
    scenarios.normalFlow();
  });

  afterEach(() => {
    server.resetHandlers();
  });

  it('should display user information', async () => {
    const response = await fetch('/api/users/1');
    const user = await response.json();

    expect(user.name).toBe('John');
    expect(user.email).toBe('john@example.com');
  });

  it('should handle user not found', async () => {
    scenarios.normalFlow();

    const response = await fetch('/api/users/999');
    expect(response.status).toBe(404);

    const error = await response.json();
    expect(error.error).toBe('User not found');
  });

  it('should create user', async () => {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: 'Alice',
        email: 'alice@example.com',
      }),
    });

    expect(response.status).toBe(201);
    const user = await response.json();
    expect(user.name).toBe('Alice');
  });

  it('should show validation error on invalid data', async () => {
    scenarios.validationError();

    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: 'Bob',
        // email 누락
      }),
    });

    expect(response.status).toBe(400);
    const error = await response.json();
    expect(error.details.email).toBeDefined();
  });

  it('should handle network error', async () => {
    scenarios.networkError();

    expect(async () => await fetch('/api/users/1')).rejects.toThrow();
  });

  it('should show loading during slow network', async () => {
    scenarios.slowNetwork();

    const promise = fetch('/api/users/1');

    // 즉시 로딩 상태 확인
    expect(document.querySelector('.loading')).toBeDefined();

    // 응답 대기
    const response = await promise;
    expect(response.status).toBe(200);
  });
});
```

### 3. React 컴포넌트 통합

```typescript
// src/components/UserProfile.tsx
import { useEffect, useState } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);

        if (!response.ok) {
          throw new Error(`User not found (${response.status})`);
        }

        const data = await response.json();
        setUser(data);
        setError(null);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
        setUser(null);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <div className="loading">Loading...</div>;
  if (error) return <div className="error">{error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

```typescript
// src/__tests__/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from '../components/UserProfile';
import { scenarios } from '../mocks/scenarios';
import { server } from '../mocks/server';

describe('UserProfile', () => {
  it('should display user info', async () => {
    scenarios.normalFlow();

    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });

  it('should show error when user not found', async () => {
    server.use(
      http.get('/api/users/999', () =>
        HttpResponse.json({ error: 'Not found' }, { status: 404 })
      )
    );

    render(<UserProfile userId="999" />);

    await waitFor(() => {
      expect(screen.getByText(/not found/i)).toBeInTheDocument();
    });
  });

  it('should show loading state', () => {
    scenarios.slowNetwork();

    render(<UserProfile userId="1" />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

## 팀 협업 가이드

```markdown
# MSW 사용 규칙

## 핸들러 관리

1. 모든 API 핸들러는 src/mocks/handlers에 정의
2. 엔드포인트별로 파일 분리 (users.ts, products.ts 등)
3. API Contract와 동기화 필수

## 개발 워크플로우

### Phase 1: API 설계 (공동)

- API 문서 작성
- MSW 핸들러 작성 (프론트엔드)
- 프론트엔드 개발 진행

### Phase 2: 백엔드 개발

- 실제 API 구현
- 테스트

### Phase 3: 통합

- MSW 핸들러 제거
- 실제 API와 연결
- 통합 테스트

## 시나리오 정의

모든 가능한 에러 케이스에 대한 시나리오 작성:

- 정상 응답
- 404 (리소스 없음)
- 400 (유효성 검증 오류)
- 401 (인증 오류)
- 500 (서버 오류)
- 네트워크 오류
- 느린 네트워크 (로딩 상태)

## 문제 해결

**Q: MSW가 요청을 가로채지 않음**
A: 핸들러 경로가 정확한지 확인하세요.

**Q: 특정 시나리오만 테스트하고 싶어요**
A: scenarios.scenarioName() 호출 후 테스트하세요.

**Q: 프로덕션에서도 MSW를 사용해야 하나?**
A: 아니오! 프로덕션에서는 MSW를 제거하세요.
```

## 결론

MSW는 **현대 프론트엔드 개발의 필수 도구**입니다:

✅ **병렬 개발**: 백엔드 완료 대기 불필요
✅ **완전한 테스트**: 모든 에러 시나리오 테스트 가능
✅ **네트워크 시뮬레이션**: 지연, 타임아웃 등 재현 가능
✅ **라이브러리 무관**: fetch, axios, 뭐든 상관없음
✅ **프로덕션 안전**: 개발/테스트에만 사용

**특히 다음 경우 강력히 권장됩니다:**

- 프론트엔드와 백엔드가 병렬 개발
- 엣지 케이스 테스트가 중요한 프로젝트
- E2E 테스트 안정성이 필요한 경우
- 네트워크 상황에 따른 UX 테스트

지금 바로 MSW를 도입하고 개발 속도를 높여보세요!

### 참고할만한 글/영상

- https://velog.io/@khy226/msw%EB%A1%9C-%EB%AA%A8%EC%9D%98-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0
- https://www.youtube.com/watch?v=7zcKu4-YPkU

## gemini와의 대화 🤖

반가워! MSW(Mock Service Worker)를 공부하려는구나? 이거 정말 물건이지. 백엔드 개발자 친구가 "아직 API 안 만들었는데요?"라고 할 때, 팔짱 끼고 "괜찮아, 나 이미 다 짜놨어"라고 폼 잡을 수 있게 해주는 아주 고마운 라이브러리야.

MSW의 핵심은 "네트워크 레벨에서 가짜 API를 가로채는 것"이야. 그래서 나중에 진짜 API가 나와도 React 코드는 한 줄도 수정할 필요가 없지.

자, 초보자 눈높이에서 '가짜 데이터로 살다가 진짜로 갈아타는 시뮬레이션'을 시작해 보자고!

## 1단계: MSW 세팅 (가짜 서버 설치하기)

먼저 도구가 필요해. 터미널을 열고 설치부터 하자.

```bash
npm install msw --save-dev
# 또는
yarn add msw --dev
```

그리고 MSW가 브라우저에서 네트워크 요청을 가로챌 수 있도록 '서비스 워커'라는 비서를 한 명 고용해야 해. 
다음 명령어를 치면 `public` 폴더에 `mockServiceWorker.js`라는 파일이 생길 거야.

```bash
npx msw init public/ --save
```

## 2단계: 가짜 데이터와 핸들러 만들기 (폴더 구조 설계)

네가 말한 대로 `mocks` 폴더를 만들어서 관리해 보자.

### 📂 mocks/data.ts (데이터 창고)
여기는 나중에 백엔드에서 줄 것 같은 데이터들을 미리 적어두는 곳이야.
```typescript
// mocks/data.ts
export const mockUsers = [
  { id: 1, name: '김철수', role: '개발자' },
  { id: 2, name: '이영희', role: '디자이너' },
];
```

### 📂 mocks/handlers.ts (주문 접수처)
"누가 `/api/users`로 요청하면 `mockUsers`를 보내줘!"라고 규칙을 정하는 곳이야.
```typescript
import { http, HttpResponse } from 'msw';
import { mockUsers } from './data';

export const handlers = [
  // GET 요청이 오면 가짜 데이터를 응답함
  http.get('/api/users', () => {
    return HttpResponse.json(mockUsers);
  }),
];
```

### 📂 mocks/browser.ts (비서 실행 준비)
```typescript
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

## 3단계: 리액트 앱에 연결 (가짜 서버 가동)

이제 리액트가 시작될 때 MSW가 작동하도록 설정해야 해. 보통 `main.tsx`나 `index.tsx`에서 작업해.

```typescript
// main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

async function enableMocking() {
  // 개발 환경에서만 MSW를 실행하도록 설정
  if (process.env.NODE_ENV !== 'development') {
    return;
  }

  const { worker } = await import('./mocks/browser');
  return worker.start();
}

enableMocking().then(() => {
  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
});
```

## 4단계: 리액트 컴포넌트에서 사용 (핵심 로직)

자, 이제 중요한 부분이야. 컴포넌트에서는 이 요청이 **가짜인지 진짜인지 몰라.** 그냥 평소처럼 `fetch`나 `axios`를 쓰면 돼.

```tsx
// App.tsx
import { useEffect, useState } from 'react';

function App() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    // MSW가 이 요청을 가로채서 mocks/data.ts의 내용을 돌려줌!
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => setUsers(data));
  }, []);

  return (
    <div>
      <h1>사용자 목록</h1>
      {users.map(user => <p key={user.id}>{user.name} ({user.role})</p>)}
    </div>
  );
}
```

## 5단계: 드디어 "실제" API가 완성됐다면? (스위치 끄기)

자, 이제 백엔드 개발자가 "API 다 만들었습니다! 이제 진짜 쓰세요!"라고 했어. 이때 네가 할 일은 아주 간단해.

### 방법 1: MSW 완전히 끄기
`main.tsx`에서 `worker.start()` 하는 부분을 주석 처리하거나 지워버려.

```typescript
// worker.start()를 안 하면? 
// 브라우저는 '/api/users'를 찾으러 진짜 네트워크(백엔드 서버)로 가버려!
```

### 방법 2: 특정 핸들러만 제거하기
전체는 유지하되 `users` API만 진짜를 쓰고 싶다면, `handlers.ts`에서 해당 코드만 지우면 돼. 그러면 MSW가 가로채지 않고 통과시켜버리거든.

### 왜 "핵심 로직"은 그대로인가요?
MSW는 브라우저 내부의 **네트워크 통신망**을 장악하고 있다가, 특정 주소로 가는 요청만 쓱 낚아채서 가짜 데이터를 주는 방식이야.
* **코드 상의 엔드포인트:** `/api/users` (그대로)
* **데이터를 가져오는 로직:** `fetch('/api/users')` (그대로)

## 요약하자면 이래!

1.  `data.ts`에 가짜 데이터를 넣고, `handlers.ts`에서 가로챌 주소를 정한다.
2.  React 컴포넌트는 **미래에 생길 진짜 주소**로 당당하게 요청을 보낸다.
3.  MSW가 중간에서 낚아채서 가짜 데이터를 준다.
4.  진짜 API가 나오면? **MSW만 꺼버린다.** 그럼 React 코드는 그대로인데 요청이 자연스럽게 진짜 서버로 흘러 들어간다.

이렇게 하면 나중에 백엔드 연결할 때 `URL`을 수정하거나 로직을 갈아엎을 필요가 전혀 없어서 아주 깔끔하지!

혹시 설정하다가 막히는 부분이 생기면 언제든 물어봐. 직접 코드를 짜보면서 에러를 보는 게 제일 빨리 배우는 길이야!

## 참고 링크

- [MSW 공식 사이트](https://mswjs.io)
- [MSW 공식 문서](https://mswjs.io/docs)
- [MSW GitHub](https://github.com/mswjs/msw)
