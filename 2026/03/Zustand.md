# Zustand - 간단하고 강력한 상태 관리

## 들어가며

Redux를 사용할 때 이런 불만이 있었나요?

```
- 너무 많은 보일러플레이트 코드
- actions, reducers, dispatch... 복잡함
- 간단한 상태 관리도 너무 복잡해짐
- 번들 크기가 너무 큼
```

**Zustand**는 이 모든 문제를 해결합니다. 정말 간단한 문법으로 강력한 상태 관리를 할 수 있습니다.

## 공식 사이트

https://zustand-demo.pmnd.rs

# 1. Zustand란?

## 핵심 철학

```
Simplicity First!

Zustand = 간단함 + 강력함 + 작은 번들 크기
```

## Redux vs Zustand

### Redux (복잡함)

```typescript
// Action Types
const ADD_TODO = 'ADD_TODO'

// Action Creators
const addTodo = (text) => ({
  type: ADD_TODO,
  payload: text
})

// Reducer
const reducer = (state = [], action) => {
  switch(action.type) {
    case ADD_TODO:
      return [...state, { id: Date.now(), text: action.payload }]
    default:
      return state
  }
}

// Store 생성
const store = createStore(reducer)

// 사용
store.dispatch(addTodo('Learn Redux'))
```

### Zustand (간단함)

```typescript
import create from 'zustand'

const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text }]
  }))
}))

// 사용
useTodoStore((state) => state.addTodo('Learn Zustand'))
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install zustand
```

## 기본 사용법

```typescript
import { create } from 'zustand'

const useStore = create((set) => ({
  // State
  count: 0,
  name: 'John',

  // Actions
  increment: () => set((state) => ({ count: state.count + 1 })),
  setName: (name) => set({ name })
}))

// 컴포넌트에서 사용
function Counter() {
  const count = useStore((state) => state.count)
  const increment = useStore((state) => state.increment)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  )
}
```

# 3. 상태와 액션

## 기본 구조

```typescript
const useStore = create((set) => ({
  // ========= STATE =========
  count: 0,
  user: null,
  items: [],

  // ========= ACTIONS =========
  increment: () => set((state) => ({
    count: state.count + 1
  })),

  setUser: (user) => set({ user }),

  addItem: (item) => set((state) => ({
    items: [...state.items, item]
  }))
}))
```

## 상태 구독

```typescript
function MyComponent() {
  // 방법 1: 선택적 구독 (권장)
  const count = useStore((state) => state.count)

  // 방법 2: 전체 구독 (비권장 - 성능 저하)
  const store = useStore()

  // 방법 3: 여러 상태 동시에
  const { count, name } = useStore(
    (state) => ({
      count: state.count,
      name: state.name
    })
  )

  return <div>{count}</div>
}
```

# 4. 실전 예제

## Todo 앱

```typescript
import { create } from 'zustand'

interface Todo {
  id: number
  text: string
  completed: boolean
}

interface TodoStore {
  todos: Todo[]
  addTodo: (text: string) => void
  removeTodo: (id: number) => void
  toggleTodo: (id: number) => void
}

const useTodoStore = create<TodoStore>((set) => ({
  todos: [],

  addTodo: (text) => set((state) => ({
    todos: [
      ...state.todos,
      {
        id: Date.now(),
        text,
        completed: false
      }
    ]
  })),

  removeTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),

  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    )
  }))
}))

// 컴포넌트
function TodoList() {
  const todos = useTodoStore((state) => state.todos)
  const addTodo = useTodoStore((state) => state.addTodo)
  const toggleTodo = useTodoStore((state) => state.toggleTodo)
  const removeTodo = useTodoStore((state) => state.removeTodo)

  return (
    <div>
      <input
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            addTodo(e.currentTarget.value)
            e.currentTarget.value = ''
          }
        }}
      />
      {todos.map(todo => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span style={{
            textDecoration: todo.completed ? 'line-through' : 'none'
          }}>
            {todo.text}
          </span>
          <button onClick={() => removeTodo(todo.id)}>
            삭제
          </button>
        </div>
      ))}
    </div>
  )
}
```

## 사용자 인증 스토어

```typescript
interface User {
  id: number
  name: string
  email: string
}

interface AuthStore {
  user: User | null
  isLoading: boolean
  error: string | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  setUser: (user: User | null) => void
}

const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  isLoading: false,
  error: null,

  login: async (email, password) => {
    set({ isLoading: true, error: null })
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      })
      const user = await response.json()
      set({ user, isLoading: false })
    } catch (error) {
      set({ error: error.message, isLoading: false })
    }
  },

  logout: () => set({ user: null }),

  setUser: (user) => set({ user })
}))

// 컴포넌트
function LoginForm() {
  const { user, isLoading, error, login, logout } = useAuthStore()

  if (user) {
    return (
      <div>
        <p>{user.name}님 환영합니다!</p>
        <button onClick={logout}>로그아웃</button>
      </div>
    )
  }

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        const formData = new FormData(e.currentTarget)
        login(
          formData.get('email'),
          formData.get('password')
        )
      }}
    >
      <input name="email" placeholder="이메일" />
      <input name="password" type="password" placeholder="비밀번호" />
      <button disabled={isLoading}>
        {isLoading ? '로그인 중...' : '로그인'}
      </button>
      {error && <p style={{ color: 'red' }}>{error}</p>}
    </form>
  )
}
```

# 5. 여러 스토어 연결

## 독립적 스토어

```typescript
// stores/useCountStore.ts
const useCountStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}))

// stores/useUserStore.ts
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user })
}))

// 컴포넌트에서 함께 사용
function Dashboard() {
  const count = useCountStore((state) => state.count)
  const user = useUserStore((state) => state.user)

  return (
    <div>
      <p>{user?.name} - Count: {count}</p>
    </div>
  )
}
```

## 스토어 간 통신

```typescript
const useCountStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}))

const useNotificationStore = create((set) => ({
  messages: [],
  addMessage: (message) => set((state) => ({
    messages: [...state.messages, message]
  }))
}))

// 스토어 간 통신
function Counter() {
  const count = useCountStore((state) => state.count)
  const increment = useCountStore((state) => state.increment)
  const addMessage = useNotificationStore((state) => state.addMessage)

  const handleClick = () => {
    increment()
    addMessage(`Count increased to ${count + 1}`)
  }

  return <button onClick={handleClick}>+1</button>
}
```

# 6. Middleware와 Plugins

## 로컬 스토리지 지속성

```typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

const useStore = create(
  persist(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }))
    }),
    {
      name: 'app-storage',  // localStorage 키
      storage: localStorage  // 또는 sessionStorage
    }
  )
)

// 사용 - 자동으로 localStorage에 저장/복원
```

## DevTools 통합

```typescript
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }))
    }),
    { name: 'AppStore' }
  )
)

// Redux DevTools에서 상태 추적 가능
```

## 여러 Middleware 조합

```typescript
import { create } from 'zustand'
import { persist, devtools } from 'zustand/middleware'

const useStore = create(
  devtools(
    persist(
      (set) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 }))
      }),
      { name: 'store' }
    ),
    { name: 'AppStore' }
  )
)
```

## 커스텀 Middleware

```typescript
const useStore = create((set) => ({
  count: 0,
  increment: () => {
    console.log('Incrementing...')
    set((state) => ({ count: state.count + 1 }))
  }
}))

// 또는 커스텀 미들웨어 작성
const withLogger = (f) => (set, get, api) =>
  f(
    (...args) => {
      console.log('State changed:', args)
      set(...args)
    },
    get,
    api
  )

const useStore2 = create(
  withLogger((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 }))
  }))
)
```

# 7. Immer 통합

## 불변성 간단하게 처리

```typescript
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

interface Todo {
  id: number
  text: string
  completed: boolean
}

const useTodoStore = create<any>(
  immer((set) => ({
    todos: [] as Todo[],

    // Immer 덕분에 직접 수정 가능!
    addTodo: (text: string) => set((state) => {
      state.todos.push({
        id: Date.now(),
        text,
        completed: false
      })
    }),

    toggleTodo: (id: number) => set((state) => {
      const todo = state.todos.find(t => t.id === id)
      if (todo) {
        todo.completed = !todo.completed
      }
    }),

    removeTodo: (id: number) => set((state) => {
      const index = state.todos.findIndex(t => t.id === id)
      if (index > -1) {
        state.todos.splice(index, 1)
      }
    })
  }))
)

// 사용은 동일
```

# 8. 성능 최적화

## 선택적 구독

```typescript
// ✅ 좋음 - 필요한 것만 구독
const count = useStore((state) => state.count)

// ❌ 나쁨 - 전체 상태 구독 (불필요한 리렌더)
const store = useStore()
const count = store.count
```

## 구독 해제

```typescript
const unsubscribe = useStore.subscribe(
  (state) => state.count,
  (count) => console.log('Count changed:', count)
)

// 나중에 구독 해제
unsubscribe()
```

## useShallow (동일성 검사)

```typescript
import { useShallow } from 'zustand/react'

function MyComponent() {
  // 객체의 얕은 비교로 최적화
  const { count, name } = useStore(
    useShallow((state) => ({
      count: state.count,
      name: state.name
    }))
  )

  return <div>{count} - {name}</div>
}
```

# 9. 팀 협업 Best Practices

## 스토어 분리

```typescript
// stores/authStore.ts
export const useAuthStore = create((set) => ({
  user: null,
  login: async (email, password) => { /* ... */ },
  logout: () => set({ user: null })
}))

// stores/todoStore.ts
export const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => { /* ... */ },
  removeTodo: (id) => { /* ... */ }
}))

// stores/index.ts
export { useAuthStore } from './authStore'
export { useTodoStore } from './todoStore'
```

## 타입 안전성

```typescript
interface AuthState {
  user: User | null
  loading: boolean
  error: string | null
  login: (email: string, password: string) => Promise<void>
  logout: () => void
}

const useAuthStore = create<AuthState>((set) => ({
  user: null,
  loading: false,
  error: null,
  login: async (email, password) => { /* ... */ },
  logout: () => set({ user: null })
}))
```

## 디렉토리 구조

```
src/
├── stores/
│   ├── auth.ts
│   ├── todos.ts
│   ├── ui.ts
│   └── index.ts
├── components/
│   ├── Login.tsx
│   ├── TodoList.tsx
│   └── Navbar.tsx
```

# 10. 자주 묻는 질문

## Q: Redux와 Zustand 언제 사용?

**A:**
- Zustand: 간단한 프로젝트, 빠른 개발
- Redux: 대규모 프로젝트, 팀 협업, 엄격한 구조 필요

```typescript
TanStack Query + Zustand = 최고의 조합!
- TanStack Query: 서버 상태
- Zustand: 클라이언트 상태 (UI)
```

## Q: 초기값은 어디서 설정?

**A:**
```typescript
// 방법 1: 직접 설정
const useStore = create((set) => ({
  count: 0  // 초기값
}))

// 방법 2: 함수로 설정
const useStore = create((set) => ({
  count: localStorage.getItem('count') ?? 0
}))
```

## Q: 여러 컴포넌트에서 상태 공유?

**A:** 같은 스토어를 import하면 자동 공유

```typescript
// 모든 컴포넌트가 같은 상태 공유
const useStore = create((set) => ({ /* ... */ }))

// ComponentA.tsx
function ComponentA() {
  const count = useStore((state) => state.count)
}

// ComponentB.tsx
function ComponentB() {
  const count = useStore((state) => state.count)
  // 같은 값!
}
```

## Q: 비동기 작업은 어떻게?

**A:** async/await 사용

```typescript
const useStore = create((set) => ({
  data: null,
  loading: false,
  fetchData: async () => {
    set({ loading: true })
    try {
      const response = await fetch('/api/data')
      const data = await response.json()
      set({ data, loading: false })
    } catch (error) {
      set({ loading: false })
    }
  }
}))
```

# 11. Zustand vs 다른 라이브러리

```
Redux:
- 장점: 엄격한 구조, 큰 커뮤니티
- 단점: 보일러플레이트 많음, 복잡함

Zustand:
- 장점: 간단함, 작은 번들
- 단점: 커뮤니티 작음

Jotai/Recoil:
- 장점: Atomic 구조
- 단점: 러닝 커브

MobX:
- 장점: 자동 추적
- 단점: 마법같은 동작

→ 대부분의 경우 Zustand가 최고!
```

# 12. 체크리스트

Zustand 적용:

```
[ ] zustand 설치
[ ] 스토어 구조 설계
[ ] 스토어 파일 생성
[ ] 컴포넌트에서 사용
[ ] 타입 정의 (TypeScript)
[ ] localStorage 통합
[ ] DevTools 추가
[ ] 팀 협업 규칙 정의
[ ] 성능 최적화
```

# 결론

Zustand는:

✅ 매우 간단한 문법
✅ 작은 번들 크기 (2.4KB)
✅ 강력한 기능
✅ 뛰어난 성능
✅ TypeScript 지원

**간단하고 효율적인 상태 관리가 필요하면 Zustand를 선택하세요!**

## 참고 링크

- [Zustand 공식 문서](https://zustand.docs.pmnd.rs)
- [Zustand GitHub](https://github.com/pmndrs/zustand)
