# TanStack Query (React Query) - 서버 상태 관리의 혁신

## 들어가며

당신이 React에서 API 데이터를 관리할 때 이런 문제를 겪었나요?

```
- API 호출 후 state에 저장
- 로딩, 에러, 성공 상태 관리
- 캐싱 로직 직접 구현
- 데이터 동기화 문제
- 중복 요청 방지
- 페이지네이션 구현
- 무한 스크롤 구현
```

`TanStack Query (이전 React Query)`는 이 모든 것을 자동으로 해결합니다. 서버 상태 관리를 위한 가장 강력한 라이브러리입니다.

## 공식 사이트

https://tanstack.com/query/latest

# 1. TanStack Query란?

## 핵심 개념

```
클라이언트 상태 (Client State)
├─ UI 상태: 열림/닫힘, 포커스 등
└─ Redux/Zustand로 관리

서버 상태 (Server State)  ← TanStack Query
├─ API 응답 데이터
├─ 캐싱
├─ 동기화
└─ 자동 갱신
```

## TanStack Query의 특징

```
✅ 자동 캐싱: 중복 요청 방지
✅ 백그라운드 동기화: 항상 최신 데이터 유지
✅ 로딩/에러 상태 자동 관리
✅ 페이지네이션 지원
✅ 무한 스크롤 지원
✅ 낙관적 업데이트
✅ 폴링 지원
✅ DevTools 제공
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install @tanstack/react-query
```

## 기본 설정

```typescript
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,      // 5분
      gcTime: 1000 * 60 * 10,         // 10분
      retry: 1,
      refetchOnWindowFocus: true
    }
  }
})

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  )
}
```

# 3. 기본 사용법

## useQuery - 데이터 조회

```typescript
import { useQuery } from '@tanstack/react-query'

// API 함수
const fetchUsers = async () => {
  const response = await fetch('/api/users')
  return response.json()
}

// 컴포넌트
function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers
  })

  if (isLoading) return <div>로딩 중...</div>
  if (error) return <div>에러: {error.message}</div>

  return (
    <div>
      {data?.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  )
}
```

## useMutation - 데이터 변경

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query'

function CreateUser() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async (newUser) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser)
      })
      return response.json()
    },
    onSuccess: () => {
      // 캐시 무효화
      queryClient.invalidateQueries({ queryKey: ['users'] })
    }
  })

  return (
    <button onClick={() => mutation.mutate({ name: 'John' })}>
      사용자 추가
    </button>
  )
}
```

# 4. Query Key 전략

## Query Key 규칙

```typescript
// Good: 계층적 구조
['users']
['users', 'list']
['users', { id: 1 }]
['users', { id: 1, filters: { status: 'active' } }]

// Bad: 일관성 없음
['users_list']
['getUsers']
['user/1']
```

## Query Key 예시

```typescript
// 사용자 목록
['users']

// 특정 사용자
['users', { id: 1 }]

// 필터링된 사용자 목록
['users', { filter: 'active' }]

// 페이지된 사용자
['users', { page: 1, limit: 10 }]

// 포스트
['posts']
['posts', { id: 1 }]
['posts', { userId: 1 }]
['posts', { id: 1, comments: true }]

// 검색 결과
['search', { query: 'react' }]
```

## Query Key Factory

```typescript
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id) => [...userKeys.details(), id] as const
}

// 사용
useQuery({
  queryKey: userKeys.detail(1),
  queryFn: () => fetchUser(1)
})

// 무효화
queryClient.invalidateQueries({
  queryKey: userKeys.lists()
})
```

# 5. 고급 기능

## 페이지네이션

```typescript
function UserList() {
  const [page, setPage] = useState(1)

  const { data, isLoading } = useQuery({
    queryKey: ['users', { page }],
    queryFn: () => fetchUsers(page),
    keepPreviousData: true  // 이전 데이터 유지
  })

  return (
    <div>
      {data?.items.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      <button onClick={() => setPage(p => p - 1)}>이전</button>
      <button onClick={() => setPage(p => p + 1)}>다음</button>
    </div>
  )
}
```

## 무한 스크롤

```typescript
import { useInfiniteQuery } from '@tanstack/react-query'

function InfiniteUserList() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
    queryKey: ['users'],
    queryFn: ({ pageParam = 1 }) => fetchUsers(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextPage
  })

  return (
    <div>
      {data?.pages.map(page => (
        page.items.map(user => (
          <div key={user.id}>{user.name}</div>
        ))
      ))}
      {hasNextPage && (
        <button onClick={() => fetchNextPage()}>
          더보기
        </button>
      )}
    </div>
  )
}
```

## 낙관적 업데이트

```typescript
function UpdateUser({ userId }) {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: (newData) => updateUser(userId, newData),
    onMutate: async (newData) => {
      // 이전 쿼리 취소
      await queryClient.cancelQueries({
        queryKey: ['users', { id: userId }]
      })

      // 이전 데이터 스냅샷
      const previousData = queryClient.getQueryData(
        ['users', { id: userId }]
      )

      // 낙관적으로 UI 업데이트
      queryClient.setQueryData(['users', { id: userId }], newData)

      return { previousData }
    },
    onError: (err, newData, context) => {
      // 에러 시 이전 데이터로 롤백
      queryClient.setQueryData(
        ['users', { id: userId }],
        context.previousData
      )
    },
    onSuccess: () => {
      // 서버 데이터로 다시 동기화
      queryClient.invalidateQueries({
        queryKey: ['users', { id: userId }]
      })
    }
  })

  return (
    <button onClick={() => mutation.mutate({ name: 'New Name' })}>
      수정
    </button>
  )
}
```

## 폴링 (자동 갱신)

```typescript
function RealtimeData() {
  const { data } = useQuery({
    queryKey: ['stats'],
    queryFn: fetchStats,
    refetchInterval: 5000  // 5초마다 갱신
  })

  return <div>{data?.count}</div>
}
```

# 6. DevTools

## 설치 및 설정

```bash
npm install @tanstack/react-query-devtools
```

```typescript
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

## DevTools 기능

```
- 모든 query 상태 확인
- 캐시 데이터 시각화
- 수동 refetch
- 캐시 무효화
- mutation 추적
```

# 7. 실전 예제

## 사용자 목록 + 상세 보기

```typescript
// hooks/useUsers.ts
import { useQuery, useQueryClient } from '@tanstack/react-query'

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users')
      return response.json()
    },
    staleTime: 1000 * 60 * 5
  })
}

export function useUser(id) {
  return useQuery({
    queryKey: ['users', { id }],
    queryFn: async () => {
      const response = await fetch(`/api/users/${id}`)
      return response.json()
    }
  })
}

export function useCreateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (data) => {
      const response = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(data)
      })
      return response.json()
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
    }
  })
}

// components/UserList.tsx
function UserList() {
  const { data: users, isLoading } = useUsers()
  const createUser = useCreateUser()

  if (isLoading) return <div>로딩 중...</div>

  return (
    <div>
      {users?.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      <button onClick={() => createUser.mutate({ name: 'John' })}>
        추가
      </button>
    </div>
  )
}

// components/UserDetail.tsx
function UserDetail({ userId }) {
  const { data: user } = useUser(userId)

  return <div>{user?.name}</div>
}
```

## 복잡한 데이터 의존성

```typescript
function DashboardPage({ userId }) {
  // 사용자 데이터 먼저 로드
  const { data: user } = useUser(userId)

  // 사용자 데이터가 있으면 포스트 로드
  const { data: posts } = useQuery({
    queryKey: ['posts', { userId }],
    queryFn: () => fetchUserPosts(userId),
    enabled: !!user  // user가 있을 때만 쿼리 실행
  })

  return (
    <div>
      <h1>{user?.name}</h1>
      {posts?.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  )
}
```

# 8. 에러 처리

## 기본 에러 처리

```typescript
function UserList() {
  const {
    data,
    isLoading,
    error,
    isError
  } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    retry: 3,  // 3번 재시도
    retryDelay: (attemptIndex) => {
      // 지수 백오프: 1초, 2초, 4초...
      return Math.min(1000 * 2 ** attemptIndex, 30000)
    }
  })

  if (isLoading) return <Loading />
  if (isError) return <Error error={error} />
  return <UserDisplay data={data} />
}
```

## 에러 바운더리 통합

```typescript
function QueryErrorResetBoundary({ children }) {
  const [isReset, setIsReset] = useState(false)

  return (
    <ErrorBoundary
      onReset={() => setIsReset(true)}
      fallback={<ErrorFallback />}
    >
      {children}
    </ErrorBoundary>
  )
}

function UserList() {
  const { data } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    throwOnError: true  // 에러 시 ErrorBoundary로 던짐
  })

  return <div>{data?.map(u => u.name)}</div>
}
```

# 9. 성능 최적화

## 쿼리 캐싱 시간 설정

```typescript
// staleTime vs gcTime
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  staleTime: 1000 * 60 * 5,    // 5분 동안 데이터가 신선함
  gcTime: 1000 * 60 * 10       // 10분 동안 캐시 유지
})

// staleTime이 0이면 항상 데이터를 stale로 간주
// gcTime이 0이면 unmount 시 캐시 삭제
```

## 배치 요청

```typescript
function UserAndPostsList() {
  // 여러 쿼리를 한 번에 실행
  const [users, posts] = useQueries({
    queries: [
      {
        queryKey: ['users'],
        queryFn: fetchUsers
      },
      {
        queryKey: ['posts'],
        queryFn: fetchPosts
      }
    ]
  })

  return <div>...</div>
}
```

## 백그라운드 동기화 비활성화

```typescript
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  refetchOnMount: false,        // 마운트 시 refetch X
  refetchOnWindowFocus: false,  // 창 포커스 시 refetch X
  refetchOnReconnect: false     // 재연결 시 refetch X
})
```

# 10. 팀 협업 Best Practices

## 쿼리 훅 분리

```typescript
// queries/useUsers.ts
export const userQueries = {
  all: () => ['users'] as const,
  lists: () => [...userQueries.all(), 'list'] as const,
  list: (filters) => [...userQueries.lists(), { filters }] as const,
  details: () => [...userQueries.all(), 'detail'] as const,
  detail: (id) => [...userQueries.details(), id] as const
}

export function useUsers(filters) {
  return useQuery({
    queryKey: userQueries.list(filters),
    queryFn: () => fetchUsers(filters)
  })
}

export function useUser(id) {
  return useQuery({
    queryKey: userQueries.detail(id),
    queryFn: () => fetchUser(id)
  })
}

// queries/useUserMutations.ts
export function useCreateUser() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: userQueries.lists()
      })
    }
  })
}
```

## API 클라이언트 분리

```typescript
// api/client.ts
export const apiClient = {
  users: {
    list: (filters) => fetch(`/api/users?${new URLSearchParams(filters)}`),
    get: (id) => fetch(`/api/users/${id}`),
    create: (data) => fetch('/api/users', { method: 'POST', body: JSON.stringify(data) }),
    update: (id, data) => fetch(`/api/users/${id}`, { method: 'PUT', body: JSON.stringify(data) }),
    delete: (id) => fetch(`/api/users/${id}`, { method: 'DELETE' })
  }
}

// queries/useUsers.ts
export function useUsers(filters) {
  return useQuery({
    queryKey: userQueries.list(filters),
    queryFn: () => apiClient.users.list(filters).then(r => r.json())
  })
}
```

# 11. 자주 묻는 질문

## Q: Redux와 TanStack Query의 차이?

**A:** 
- Redux: 클라이언트 상태 관리 (UI 상태)
- TanStack Query: 서버 상태 관리 (API 데이터)

```
함께 사용 권장!
Redux + TanStack Query = 최고의 상태 관리
```

## Q: staleTime과 gcTime의 차이?

**A:**
- staleTime: 데이터가 "신선한" 상태로 간주되는 시간
- gcTime: 캐시가 메모리에 유지되는 시간

```typescript
staleTime: 5분, gcTime: 10분
→ 5분 후 데이터는 stale이지만 캐시는 유지
→ 10분 후 완전히 삭제
```

## Q: 동시에 같은 요청이 들어오면?

**A:** 자동으로 중복 제거 (deduping)

```typescript
// 동시에 같은 queryKey로 요청 → 하나의 요청만 실행
const q1 = useQuery({ queryKey: ['users'], ... })
const q2 = useQuery({ queryKey: ['users'], ... })
// 단 1번의 /api/users 호출
```

## Q: 낙관적 업데이트가 실패하면?

**A:** onMutate에서 저장한 previousData로 롤백

```typescript
onError: (err, variables, context) => {
  queryClient.setQueryData(
    queryKey,
    context.previousData
  )
}
```

# 12. 체크리스트

프로젝트에 TanStack Query 적용:

```
[ ] @tanstack/react-query 설치
[ ] QueryClient 생성 및 설정
[ ] QueryClientProvider로 앱 감싸기
[ ] Query Key Factory 생성
[ ] useQuery 훅 작성
[ ] useMutation 훅 작성
[ ] 에러 처리 구현
[ ] DevTools 추가
[ ] 캐싱 시간 설정
[ ] API 클라이언트 분리
```

# 결론

TanStack Query는:

✅ 서버 상태 관리 자동화
✅ 캐싱 최적화
✅ 백그라운드 동기화
✅ 개발자 경험 향상
✅ 성능 개선

**React 앱에서 API 데이터는 TanStack Query로 관리하세요!**

## 참고 링크

- [TanStack Query 공식 사이트](https://tanstack.com/query/latest)
- [TanStack Query 공식 문서](https://tanstack.com/query/latest/docs/framework/react/overview)
- [TanStack Query GitHub](https://github.com/TanStack/query)
