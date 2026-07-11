# React Router - SPA 라우팅의 완벽 솔루션

## 들어가며

React로 여러 페이지를 가진 앱을 만들 때 어떻게 페이지를 전환하나요?

```typescript
// ❌ 나쁜 방법 (수동 상태 관리)
const [page, setPage] = useState('home')

return (
  <div>
    {page === 'home' && <HomePage />}
    {page === 'about' && <AboutPage />}
    {page === 'contact' && <ContactPage />}
    <button onClick={() => setPage('home')}>Home</button>
    <button onClick={() => setPage('about')}>About</button>
  </div>
)
```

**React Router**는 URL 기반의 선언적 라우팅을 제공합니다. 브라우저 히스토리도 자동으로 관리해줍니다.

## 공식 사이트

https://reactrouter.com

# 1. React Router란?

## 핵심 개념

```
React Router = SPA(Single Page Application)용 라우팅 라이브러리

특징:
✅ URL 기반 라우팅
✅ 선언적 라우트 정의
✅ 중첩된 라우트 지원
✅ 동적 라우트 매칭
✅ 프로그래매틱 네비게이션
✅ 라우트 가드
✅ 쿼리 파라미터 관리
✅ 브라우저 히스토리 자동 관리
```

## React Router v6의 특징

```
v5 vs v6:

v5:
- <Route component={} />
- <Link to="/page" />
- 더 복잡한 설정

v6:
- <Route element={<Component />} />
- <Link to="/page" />
- 더 간단한 API
- 훅 기반
- 중첩 라우트 쉬움
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install react-router-dom
```

## 기본 설정

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import HomePage from './pages/HomePage'
import AboutPage from './pages/AboutPage'
import ContactPage from './pages/ContactPage'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

# 3. 기본 라우팅

## 간단한 라우팅

```typescript
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom'

function HomePage() {
  return <h1>Home</h1>
}

function AboutPage() {
  return <h1>About</h1>
}

function ContactPage() {
  return <h1>Contact</h1>
}

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>

      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

## Link vs NavLink

```typescript
import { Link, NavLink } from 'react-router-dom'

function Navigation() {
  return (
    <nav>
      {/* Link: 일반 링크 */}
      <Link to="/">Home</Link>

      {/* NavLink: 활성화 상태 표시 */}
      <NavLink 
        to="/about"
        className={({ isActive }) => isActive ? 'active' : ''}
        style={({ isActive }) => ({
          color: isActive ? 'red' : 'black'
        })}
      >
        About
      </NavLink>

      {/* end prop: 정확한 매칭 */}
      <NavLink to="/products" end>
        Products
      </NavLink>
    </nav>
  )
}
```

# 4. 동적 라우트

## URL 파라미터

```typescript
import { BrowserRouter, Routes, Route, useParams } from 'react-router-dom'

// 상세 페이지 컴포넌트
function ProductDetail() {
  // URL 파라미터 추출
  const { id } = useParams()

  return (
    <div>
      <h1>Product Detail</h1>
      <p>Product ID: {id}</p>
    </div>
  )
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* :id는 동적 세그먼트 */}
        <Route path="/products/:id" element={<ProductDetail />} />
        
        {/* 여러 파라미터 */}
        <Route path="/users/:userId/posts/:postId" element={<PostDetail />} />
      </Routes>
    </BrowserRouter>
  )
}

function PostDetail() {
  const { userId, postId } = useParams()
  
  return <h1>User {userId}'s Post {postId}</h1>
}
```

## 쿼리 파라미터

```typescript
import { useSearchParams } from 'react-router-dom'

function SearchPage() {
  // 쿼리 파라미터 추출
  const [searchParams, setSearchParams] = useSearchParams()

  const query = searchParams.get('q') // /search?q=javascript
  const page = searchParams.get('page') // /search?q=javascript&page=2

  // 쿼리 파라미터 변경
  const handleSearch = (newQuery) => {
    setSearchParams({ q: newQuery, page: '1' })
  }

  return (
    <div>
      <input
        value={query || ''}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      <p>Query: {query}</p>
      <p>Page: {page}</p>
    </div>
  )
}
```

## 와일드카드 라우트

```typescript
import { BrowserRouter, Routes, Route } from 'react-router-dom'

function NotFound() {
  return <h1>404 - Page Not Found</h1>
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        
        {/* * 는 모든 경로와 매칭 (맨 마지막에 위치) */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  )
}
```

# 5. 중첩 라우트 (Nested Routes)

## 레이아웃과 함께 중첩

```typescript
import { BrowserRouter, Routes, Route, Outlet } from 'react-router-dom'

// 레이아웃 컴포넌트
function Layout() {
  return (
    <div>
      <header>Header</header>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/products">Products</Link>
        <Link to="/about">About</Link>
      </nav>
      
      {/* 자식 라우트가 여기에 렌더링됨 */}
      <main>
        <Outlet />
      </main>
      
      <footer>Footer</footer>
    </div>
  )
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* 레이아웃을 감싼 중첩 라우트 */}
        <Route element={<Layout />}>
          <Route path="/" element={<HomePage />} />
          <Route path="/products" element={<ProductsPage />} />
          <Route path="/products/:id" element={<ProductDetail />} />
          <Route path="/about" element={<AboutPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}
```

## 깊은 중첩

```typescript
import { Outlet } from 'react-router-dom'

// 상품 레이아웃
function ProductLayout() {
  return (
    <div className="product-layout">
      <aside>
        <ProductMenu />
      </aside>
      <main>
        <Outlet />
      </main>
    </div>
  )
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<Layout />}>
          <Route path="/" element={<HomePage />} />
          
          {/* 깊은 중첩 */}
          <Route path="/products" element={<ProductLayout />}>
            <Route index element={<ProductsList />} />
            <Route path=":id" element={<ProductDetail />} />
            <Route path=":id/reviews" element={<ProductReviews />} />
          </Route>
          
          <Route path="/about" element={<AboutPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}
```

# 6. 프로그래매틱 네비게이션

## useNavigate 훅

```typescript
import { useNavigate } from 'react-router-dom'

function LoginForm() {
  const navigate = useNavigate()

  const handleSubmit = async (credentials) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials)
    })

    if (response.ok) {
      // 로그인 성공 후 대시보드로 이동
      navigate('/dashboard')

      // 또는 이전 페이지로 돌아가기
      navigate(-1)

      // 쿼리 파라미터와 함께
      navigate('/products?sort=price')

      // 상태와 함께
      navigate('/dashboard', { state: { message: 'Login successful' } })
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 내용 */}
    </form>
  )
}
```

## 위치 상태 (Location State)

```typescript
import { useNavigate, useLocation } from 'react-router-dom'

// 페이지 1: 네비게이션
function ProductsList() {
  const navigate = useNavigate()

  const goToDetail = (productId) => {
    navigate(`/products/${productId}`, {
      state: { from: '/products', scrollPosition: window.scrollY }
    })
  }

  return (
    <button onClick={() => goToDetail(123)}>
      View Product
    </button>
  )
}

// 페이지 2: 상태 받기
function ProductDetail() {
  const location = useLocation()
  
  // state에 접근
  const { from, scrollPosition } = location.state || {}

  return (
    <div>
      <p>Came from: {from}</p>
      <button onClick={() => window.scrollTo(0, scrollPosition)}>
        Restore scroll position
      </button>
    </div>
  )
}
```

# 7. 라우트 가드 (Protected Routes)

## 로그인 필요한 라우트

```typescript
import { Navigate, Outlet } from 'react-router-dom'

// 인증 상태 확인
const useAuth = () => {
  const [isLoggedIn, setIsLoggedIn] = useState(false)
  // 실제로는 서버에서 확인
  return { isLoggedIn }
}

// ProtectedRoute 컴포넌트
function ProtectedRoute() {
  const { isLoggedIn } = useAuth()

  // 로그인 안 됨 → 로그인 페이지로 리다이렉트
  if (!isLoggedIn) {
    return <Navigate to="/login" replace />
  }

  // 로그인 됨 → 자식 라우트 렌더링
  return <Outlet />
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<LoginPage />} />

        {/* 보호된 라우트 */}
        <Route element={<ProtectedRoute />}>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}
```

## 역할 기반 접근 제어 (RBAC)

```typescript
import { Navigate, Outlet } from 'react-router-dom'

// 사용자 역할 확인
const useAuth = () => {
  const [user, setUser] = useState(null)
  return user // { id: 1, role: 'admin' }
}

// 역할 확인 라우트
function RoleProtectedRoute({ allowedRoles }) {
  const user = useAuth()

  if (!user) {
    return <Navigate to="/login" replace />
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />
  }

  return <Outlet />
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/login" element={<LoginPage />} />

        {/* 관리자만 접근 가능 */}
        <Route element={<RoleProtectedRoute allowedRoles={['admin']} />}>
          <Route path="/admin" element={<AdminPanel />} />
          <Route path="/admin/users" element={<UserManagement />} />
        </Route>

        {/* 로그인한 사용자만 접근 가능 */}
        <Route element={<RoleProtectedRoute allowedRoles={['admin', 'user']} />}>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Route>

        <Route path="/unauthorized" element={<Unauthorized />} />
      </Routes>
    </BrowserRouter>
  )
}
```

# 8. 지연 로딩 (Lazy Loading)

## 동적 임포트

```typescript
import { lazy, Suspense } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

// 동적 로딩
const HomePage = lazy(() => import('./pages/HomePage'))
const AboutPage = lazy(() => import('./pages/AboutPage'))
const ProductsPage = lazy(() => import('./pages/ProductsPage'))
const AdminPage = lazy(() => import('./pages/AdminPage'))

// 로딩 화면
function LoadingSpinner() {
  return <div>Loading...</div>
}

function App() {
  return (
    <BrowserRouter>
      {/* Suspense로 모든 lazy routes를 감싸기 */}
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/about" element={<AboutPage />} />
          <Route path="/products" element={<ProductsPage />} />
          <Route path="/admin" element={<AdminPage />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

## 라우트별 로딩 화면

```typescript
import { Suspense } from 'react'

function RouteWithLoading({ component: Component }) {
  return (
    <Suspense fallback={<PageLoader />}>
      <Component />
    </Suspense>
  )
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route 
          path="/products" 
          element={<RouteWithLoading component={ProductsPage} />} 
        />
        <Route 
          path="/admin" 
          element={<RouteWithLoading component={AdminPage} />} 
        />
      </Routes>
    </BrowserRouter>
  )
}
```

# 9. 라우트 설정 파일로 관리

## 라우트 배열

```typescript
// routes.tsx
import { lazy } from 'react'
import { RouteObject } from 'react-router-dom'
import Layout from './components/Layout'
import ProtectedRoute from './components/ProtectedRoute'

const HomePage = lazy(() => import('./pages/HomePage'))
const AboutPage = lazy(() => import('./pages/AboutPage'))
const ProductsPage = lazy(() => import('./pages/ProductsPage'))
const ProductDetail = lazy(() => import('./pages/ProductDetail'))
const Dashboard = lazy(() => import('./pages/Dashboard'))
const NotFound = lazy(() => import('./pages/NotFound'))

export const routes: RouteObject[] = [
  {
    element: <Layout />,
    children: [
      {
        path: '/',
        element: <HomePage />
      },
      {
        path: '/about',
        element: <AboutPage />
      },
      {
        path: '/products',
        element: <ProductsPage />
      },
      {
        path: '/products/:id',
        element: <ProductDetail />
      },
      {
        element: <ProtectedRoute />,
        children: [
          {
            path: '/dashboard',
            element: <Dashboard />
          }
        ]
      },
      {
        path: '*',
        element: <NotFound />
      }
    ]
  }
]

// App.tsx
import { BrowserRouter } from 'react-router-dom'
import { useRoutes } from 'react-router-dom'
import { routes } from './routes'

function App() {
  const element = useRoutes(routes)
  return element
}

export default App
```

# 10. 실전 예제

## 블로그 앱 라우팅

```typescript
import { BrowserRouter, Routes, Route, Outlet } from 'react-router-dom'
import { lazy, Suspense } from 'react'

// 페이지들
const HomePage = lazy(() => import('./pages/HomePage'))
const BlogList = lazy(() => import('./pages/BlogList'))
const BlogDetail = lazy(() => import('./pages/BlogDetail'))
const CreateBlog = lazy(() => import('./pages/CreateBlog'))
const EditBlog = lazy(() => import('./pages/EditBlog'))
const Profile = lazy(() => import('./pages/Profile'))
const AdminDashboard = lazy(() => import('./pages/AdminDashboard'))
const NotFound = lazy(() => import('./pages/NotFound'))

// 레이아웃
function MainLayout() {
  return (
    <div className="main-layout">
      <Header />
      <Sidebar />
      <main>
        <Outlet />
      </main>
      <Footer />
    </div>
  )
}

// 블로그 레이아웃
function BlogLayout() {
  return (
    <div className="blog-layout">
      <BlogSidebar />
      <div className="blog-content">
        <Outlet />
      </div>
    </div>
  )
}

// 보호된 라우트
function ProtectedRoute() {
  const { isLoggedIn } = useAuth()
  return isLoggedIn ? <Outlet /> : <Navigate to="/login" />
}

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          {/* 메인 레이아웃 */}
          <Route element={<MainLayout />}>
            <Route path="/" element={<HomePage />} />

            {/* 블로그 섹션 */}
            <Route path="/blog" element={<BlogLayout />}>
              <Route index element={<BlogList />} />
              <Route path=":id" element={<BlogDetail />} />
            </Route>

            {/* 보호된 라우트 */}
            <Route element={<ProtectedRoute />}>
              <Route path="/blog/create" element={<CreateBlog />} />
              <Route path="/blog/:id/edit" element={<EditBlog />} />
              <Route path="/profile" element={<Profile />} />
              <Route path="/admin" element={<AdminDashboard />} />
            </Route>

            {/* 404 */}
            <Route path="*" element={<NotFound />} />
          </Route>
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

# 11. 고급 기능

## useLocation으로 현재 경로 감시

```typescript
import { useLocation, useNavigate } from 'react-router-dom'
import { useEffect } from 'react'

function ScrollToTop() {
  const { pathname } = useLocation()

  useEffect(() => {
    // 라우트 변경 시 맨 위로 스크롤
    window.scrollTo(0, 0)
  }, [pathname])

  return null
}

function App() {
  return (
    <BrowserRouter>
      <ScrollToTop />
      <Routes>
        {/* 라우트들 */}
      </Routes>
    </BrowserRouter>
  )
}
```

## useMatch로 현재 라우트 매칭 확인

```typescript
import { useMatch } from 'react-router-dom'

function Navigation() {
  // 특정 경로가 매칭되는지 확인
  const isProductsPage = useMatch('/products')
  const isProductDetail = useMatch('/products/:id')

  return (
    <nav>
      <Link to="/products" className={isProductsPage ? 'active' : ''}>
        Products
      </Link>

      {isProductDetail && (
        <div className="breadcrumb">
          Products > {isProductDetail.params.id}
        </div>
      )}
    </nav>
  )
}
```

## 여러 라우터

```typescript
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      {
        path: '/',
        element: <HomePage />
      },
      {
        path: '/about',
        element: <AboutPage />
      }
    ]
  }
])

function App() {
  return <RouterProvider router={router} />
}
```

# 12. 팀 협업 Best Practices

## 라우트 타입 정의

```typescript
// types/routes.ts
export enum AppRoutes {
  HOME = '/',
  ABOUT = '/about',
  PRODUCTS = '/products',
  PRODUCT_DETAIL = '/products/:id',
  DASHBOARD = '/dashboard',
  ADMIN = '/admin',
  LOGIN = '/login'
}

// routes.tsx
import { AppRoutes } from '@/types/routes'

export const routes: RouteObject[] = [
  {
    path: AppRoutes.HOME,
    element: <HomePage />
  },
  {
    path: AppRoutes.PRODUCTS,
    element: <ProductsPage />
  },
  {
    path: AppRoutes.PRODUCT_DETAIL,
    element: <ProductDetail />
  }
]

// 컴포넌트에서 사용
import { AppRoutes } from '@/types/routes'

function Navigation() {
  return (
    <nav>
      <Link to={AppRoutes.HOME}>Home</Link>
      <Link to={AppRoutes.PRODUCTS}>Products</Link>
    </nav>
  )
}

// 프로그래매틱 네비게이션
function Example() {
  const navigate = useNavigate()

  const goToProduct = (id: number) => {
    navigate(AppRoutes.PRODUCT_DETAIL.replace(':id', id.toString()))
  }
}
```

# 13. 자주 묻는 질문

## Q: Link vs navigate?

**A:**
```typescript
// Link: 유저가 클릭할 때 네비게이트 (선언적)
<Link to="/about">About</Link>

// navigate: 프로그래매틱하게 네비게이트 (명령형)
const navigate = useNavigate()
navigate('/about') // 이벤트 처리 후 자동 네비게이트
```

## Q: useParams vs useSearchParams?

**A:**
```typescript
// useParams: URL 경로 파라미터
// /products/123 → const { id } = useParams() → id = '123'

// useSearchParams: 쿼리 문자열
// /products?sort=price&page=2 → 
// const [params] = useSearchParams()
// params.get('sort') → 'price'
```

## Q: 뒤로가기 버튼 기능은?

**A:**
```typescript
import { useNavigate } from 'react-router-dom'

function GoBackButton() {
  const navigate = useNavigate()

  return (
    <button onClick={() => navigate(-1)}>
      Go Back
    </button>
  )
}
```

# 14. 체크리스트

React Router 시작하기:

```
[ ] react-router-dom 설치
[ ] BrowserRouter 설정
[ ] 기본 라우트 작성
[ ] Link/NavLink로 네비게이션
[ ] useParams로 동적 라우트 처리
[ ] useNavigate로 프로그래매틱 네비게이션
[ ] 중첩 라우트 구현
[ ] 보호된 라우트 구현
[ ] 지연 로딩 설정
[ ] 라우트 배열로 관리
[ ] 팀 협업 규칙 정의
```

# 결론

React Router는:

✅ 선언적 라우팅
✅ 중첩 라우트 지원
✅ 동적 라우팅 용이
✅ 보호된 라우트
✅ 지연 로딩 지원
✅ 강력한 훅 API

**React SPA 개발에는 React Router가 필수입니다!**

## 참고 링크

- [React Router 공식 사이트](https://reactrouter.com)
- [React Router 공식 문서](https://reactrouter.com/home)
- [React Router GitHub](https://github.com/remix-run/react-router)
