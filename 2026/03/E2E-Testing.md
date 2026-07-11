# E2E 테스트 - End-to-End 테스트 완벽 가이드

## 들어가며

당신의 앱이 제대로 작동하는지 어떻게 확인하나요?

```
수동 테스트:
1. 앱 실행
2. 사용자처럼 조작
3. 결과 확인
4. 반복...

❌ 시간이 많이 걸림
❌ 실수하기 쉬움
❌ 배포할 때마다 해야 함
```

**E2E 테스트(End-to-End Test)** 는 사용자의 실제 행동을 자동으로 시뮬레이션해서 앱이 제대로 작동하는지 확인합니다.

# 1. E2E 테스트란?

## 개념

```
E2E 테스트 = 사용자의 관점에서 전체 흐름을 테스트

사용자가 하는 모든 것을 자동으로 시뮬레이션:
- 브라우저 열기
- 웹사이트 방문
- 클릭, 입력
- 결과 확인
```

## 테스트 피라미드

```
        E2E 테스트 (느림, 비용 많음, 중요함)
       /            \
      /              \
  통합 테스트 (중간 속도, 중간 비용)
   /                    \
  /                      \
단위 테스트 (빠름, 저비용, 많은 테스트)
```

```
비율: 단위 테스트 70% : 통합 테스트 20% : E2E 테스트 10%
```

## 테스트 종류 비교

```
단위 테스트:
- 함수 하나씩 테스트
- 매우 빠름
- 많이 작성
- 예: sum(2, 3) === 5

통합 테스트:
- 여러 모듈의 상호작용 테스트
- 중간 속도
- 중간 정도 개수
- 예: API 호출 + 데이터 저장

E2E 테스트:
- 사용자 관점의 전체 흐름 테스트
- 느림
- 적게 작성 (중요한 것만)
- 예: 로그인 → 상품 검색 → 결제 → 완료
```

# 2. E2E 테스트가 필요한 경우

## E2E 테스트를 작성해야 하는 경우

```
✅ 중요한 사용자 흐름
- 회원가입
- 로그인
- 결제 프로세스
- 데이터 저장

✅ 여러 페이지/컴포넌트가 연관된 기능
- 장바구니 → 결제
- 검색 → 상세 페이지
- 로그인 → 대시보드

✅ 실제 API와의 상호작용
- 백엔드 API 호출
- 데이터베이스 조회
- 외부 서비스 연동
```

## E2E 테스트가 불필요한 경우

```
❌ 단순한 UI 변경
- 색상 변경
- 텍스트 수정

❌ 개별 함수의 로직
- sum(2, 3) 테스트
- 날짜 포맷팅

❌ 매우 자주 변경되는 부분
- 임시 기능
- 실험적 기능
```

# 3. E2E 테스트 도구 비교

## 주요 도구

```
Playwright:
✅ 크로스 브라우저 (Chromium, Firefox, WebKit)
✅ 빠른 속도
✅ 강력한 API
✅ 웹 스크래핑도 가능

Cypress:
✅ 초보자 친화적
✅ 뛰어난 디버깅 UI
✅ 좋은 문서
❌ Chrome만 지원 (초기 버전)
❌ 상대적으로 느림

Selenium:
✅ 가장 오래되고 많은 기업에서 사용
✅ 모든 브라우저 지원
❌ 학습곡선이 높음
❌ 상대적으로 느림

WebDriver:
✅ 표준 프로토콜
✅ 모든 브라우저 지원
❌ 설정이 복잡함
```

## 선택 기준

```
초보자         → Cypress
여러 브라우저  → Playwright
대규모 기업    → Selenium
```

# 4. 기본 E2E 테스트 작성 (Playwright)

## 간단한 테스트

```typescript
import { test, expect } from '@playwright/test'

test('홈페이지 방문 및 로드', async ({ page }) => {
  // 1. 페이지 이동
  await page.goto('https://example.com')

  // 2. 페이지 로드 확인
  await expect(page).toHaveTitle('Example Domain')

  // 3. 주요 요소 확인
  const heading = page.locator('h1')
  await expect(heading).toBeVisible()
  await expect(heading).toContainText('Example')
})
```

## 로그인 플로우 테스트

```typescript
test('사용자 로그인', async ({ page }) => {
  // 1. 로그인 페이지 이동
  await page.goto('https://example.com/login')

  // 2. 이메일 입력
  await page.fill('input[name="email"]', 'test@example.com')

  // 3. 비밀번호 입력
  await page.fill('input[name="password"]', 'password123')

  // 4. 로그인 버튼 클릭
  await page.click('button:has-text("로그인")')

  // 5. 로그인 완료 대기
  await page.waitForURL('/dashboard')

  // 6. 대시보드 확인
  const greeting = page.locator('[data-testid="greeting"]')
  await expect(greeting).toContainText('환영합니다')
})
```

## 상품 검색 및 구매 테스트

```typescript
test('상품 검색 및 구매', async ({ page }) => {
  // 1. 쇼핑몰 접속
  await page.goto('https://shop.example.com')

  // 2. 상품 검색
  await page.fill('input[placeholder="검색"]', 'iPhone')
  await page.press('input[placeholder="검색"]', 'Enter')

  // 3. 검색 결과 로드 대기
  await page.waitForSelector('.product-card')

  // 4. 첫 번째 상품 클릭
  await page.click('.product-card:first-child')

  // 5. 상품 상세 페이지 확인
  await page.waitForURL(/\/products\/\d+/)
  const price = page.locator('[data-testid="price"]')
  await expect(price).toContainText('$')

  // 6. 장바구니 추가
  await page.click('button:has-text("장바구니 추가")')

  // 7. 성공 메시지 확인
  const notification = page.locator('[role="alert"]')
  await expect(notification).toContainText('장바구니에 추가되었습니다')

  // 8. 장바구니 이동
  await page.click('a:has-text("장바구니")')

  // 9. 장바구니 확인
  const cartItem = page.locator('.cart-item')
  await expect(cartItem).toHaveCount(1)

  // 10. 결제 진행
  await page.click('button:has-text("결제하기")')

  // 11. 결제 페이지 확인
  await page.waitForURL('/checkout')
  await expect(page.locator('h1')).toContainText('결제')
})
```

# 5. 실전 예제

## 회원가입 플로우

```typescript
test('새 사용자 회원가입', async ({ page }) => {
  // 1. 회원가입 페이지 이동
  await page.goto('https://example.com/signup')

  // 2. 가입 정보 입력
  await page.fill('input[name="username"]', 'newuser')
  await page.fill('input[name="email"]', 'newuser@example.com')
  await page.fill('input[name="password"]', 'SecurePass123!')
  await page.fill('input[name="confirmPassword"]', 'SecurePass123!')

  // 3. 약관 동의
  await page.check('input[name="agreeTerms"]')

  // 4. 가입 버튼 클릭
  await page.click('button[type="submit"]')

  // 5. 가입 성공 확인
  await page.waitForURL('/welcome')
  const welcomeMessage = page.locator('[data-testid="welcome"]')
  await expect(welcomeMessage).toContainText('환영합니다, newuser!')

  // 6. 이메일 확인 링크 페이지 표시 확인
  const verificationMessage = page.locator('[role="alert"]')
  await expect(verificationMessage).toContainText('이메일로 확인 링크를 보냈습니다')
})
```

## 프로필 수정 테스트

```typescript
test('사용자 프로필 수정', async ({ page, context }) => {
  // 1. 로그인 (인증 필요)
  // ... 로그인 코드 ...

  // 2. 프로필 페이지 이동
  await page.goto('/profile')

  // 3. 프로필 수정 버튼 클릭
  await page.click('button:has-text("수정")')

  // 4. 수정 모드 확인
  const nameInput = page.locator('input[name="name"]')
  await expect(nameInput).toBeEnabled()

  // 5. 정보 수정
  await nameInput.clear()
  await nameInput.fill('New Name')
  await page.fill('input[name="bio"]', '새로운 소개입니다')

  // 6. 저장 클릭
  await page.click('button:has-text("저장")')

  // 7. 저장 완료 확인
  const successMessage = page.locator('[data-testid="success"]')
  await expect(successMessage).toContainText('프로필이 업데이트되었습니다')

  // 8. 프로필 다시 로드
  await page.reload()

  // 9. 변경 사항 확인
  const profileName = page.locator('[data-testid="profile-name"]')
  await expect(profileName).toContainText('New Name')
})
```

## 필터링 및 정렬 테스트

```typescript
test('상품 필터링 및 정렬', async ({ page }) => {
  // 1. 상품 목록 페이지 이동
  await page.goto('/products')

  // 2. 초기 상품 개수 확인
  const initialProducts = page.locator('[data-testid="product"]')
  const initialCount = await initialProducts.count()
  expect(initialCount).toBeGreaterThan(0)

  // 3. 가격 범위 필터 설정
  await page.fill('input[name="priceMin"]', '100')
  await page.fill('input[name="priceMax"]', '500')
  await page.click('button:has-text("필터")')

  // 4. 필터 적용 대기
  await page.waitForLoadState('networkidle')

  // 5. 필터된 상품 개수 확인
  const filteredProducts = page.locator('[data-testid="product"]')
  const filteredCount = await filteredProducts.count()
  expect(filteredCount).toBeLessThanOrEqual(initialCount)

  // 6. 정렬 변경
  await page.selectOption('select[name="sort"]', 'price-asc')

  // 7. 정렬 적용 대기
  await page.waitForLoadState('networkidle')

  // 8. 정렬 확인 (첫 상품이 가장 저가)
  const firstPrice = await page
    .locator('[data-testid="product"]:first-child [data-testid="price"]')
    .textContent()
  expect(firstPrice).toBeTruthy()
})
```

# 6. 데이터 기반 테스트

## 여러 사례로 테스트

```typescript
test.describe('다양한 검색어로 테스트', () => {
  const searchCases = [
    { query: 'iPhone', expectedCount: 10 },
    { query: 'iPad', expectedCount: 5 },
    { query: 'MacBook', expectedCount: 3 }
  ]

  searchCases.forEach(({ query, expectedCount }) => {
    test(`"${query}" 검색 결과`, async ({ page }) => {
      await page.goto('/search')
      await page.fill('input[name="q"]', query)
      await page.press('input[name="q"]', 'Enter')

      const results = page.locator('[data-testid="result"]')
      await expect(results).toHaveCount(expectedCount)
    })
  })
})
```

## 매개변수화된 테스트

```typescript
const users = [
  { email: 'admin@example.com', password: 'admin123', role: 'Admin' },
  { email: 'user@example.com', password: 'user123', role: 'User' },
  { email: 'guest@example.com', password: 'guest123', role: 'Guest' }
]

users.forEach(({ email, password, role }) => {
  test(`${role} 로그인`, async ({ page }) => {
    await page.goto('/login')
    await page.fill('input[name="email"]', email)
    await page.fill('input[name="password"]', password)
    await page.click('button[type="submit"]')

    await page.waitForURL('/dashboard')
    const roleIndicator = page.locator('[data-testid="role"]')
    await expect(roleIndicator).toContainText(role)
  })
})
```

# 7. 데이터베이스와 API 테스트

## API 모킹

```typescript
test('API 응답 모킹', async ({ page }) => {
  // API 응답 가로채기
  await page.route('/api/users', (route) => {
    route.abort() // 요청 차단
  })

  // 또는 응답 변조
  await page.route('/api/products', (route) => {
    route.continue({
      response: {
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([
          { id: 1, name: 'Product 1', price: 100 },
          { id: 2, name: 'Product 2', price: 200 }
        ])
      }
    })
  })

  await page.goto('/products')
  const products = page.locator('[data-testid="product"]')
  await expect(products).toHaveCount(2)
})
```

## 실제 API와 DB 테스트

```typescript
test('전체 사용자 플로우 (실제 API)', async ({ page }) => {
  // 1. 새 사용자 가입
  await page.goto('/signup')
  const uniqueEmail = `user-${Date.now()}@example.com`

  await page.fill('input[name="email"]', uniqueEmail)
  await page.fill('input[name="password"]', 'SecurePass123!')
  await page.click('button[type="submit"]')

  // 2. 가입 완료 확인
  await page.waitForURL('/welcome')

  // 3. 로그인
  await page.goto('/login')
  await page.fill('input[name="email"]', uniqueEmail)
  await page.fill('input[name="password"]', 'SecurePass123!')
  await page.click('button[type="submit"]')

  // 4. 로그인 성공 확인
  await page.waitForURL('/dashboard')

  // 5. 프로필에서 사용자 정보 확인
  await page.goto('/profile')
  const emailDisplay = page.locator('[data-testid="email"]')
  await expect(emailDisplay).toContainText(uniqueEmail)

  // 6. 로그아웃
  await page.click('button:has-text("로그아웃")')

  // 7. 로그아웃 확인
  await page.waitForURL('/login')
})
```

# 8. 고급 테크닉

## 대기 전략

```typescript
test('여러 대기 방식', async ({ page }) => {
  await page.goto('https://example.com')

  // 1. 요소 나타날 때까지
  await page.waitForSelector('.content', { state: 'visible' })

  // 2. 로더 사라질 때까지
  await page.waitForSelector('.loader', { state: 'hidden' })

  // 3. 네트워크 요청 완료
  await page.waitForLoadState('networkidle')

  // 4. URL 변경
  await page.waitForURL('/dashboard')

  // 5. 함수 실행 결과
  await page.waitForFunction(() => {
    return document.querySelectorAll('[data-testid="item"]').length === 10
  })
})
```

## 스크린샷 및 비디오

```typescript
test('스크린샷 캡처', async ({ page }) => {
  await page.goto('/error-page')

  // 전체 페이지 스크린샷
  await page.screenshot({ path: 'error-page.png', fullPage: true })

  // 특정 요소 스크린샷
  const errorBox = page.locator('.error-message')
  await errorBox.screenshot({ path: 'error-message.png' })
})
```

## 여러 탭/창 처리

```typescript
test('여러 탭에서 상호작용', async ({ browser }) => {
  // 새 브라우저 컨텍스트 생성
  const context = await browser.newContext()
  const page1 = await context.newPage()
  const page2 = await context.newPage()

  // 탭 1: 판매자 관리 페이지
  await page1.goto('/admin/products')
  await page1.fill('input[name="price"]', '100')
  await page1.click('button:has-text("저장")')

  // 탭 2: 구매자 페이지
  await page2.goto('/shop')
  await page2.reload() // 가격 변경 확인
  const price = page2.locator('[data-testid="price"]')
  await expect(price).toContainText('100')

  await context.close()
})
```

# 9. 설정 및 최적화

## playwright.config.ts 설정

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  testMatch: '**/*.spec.ts',

  // 테스트 타임아웃
  timeout: 30 * 1000,
  expect: {
    timeout: 5000
  },

  // 병렬 실행
  fullyParallel: true,
  workers: 4,

  // 실패 시 재시도 (CI 환경만)
  retries: process.env.CI ? 2 : 0,

  // 리포트
  reporter: 'html',

  // 기본 설정
  use: {
    baseURL: 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry'
  },

  // 크로스 브라우저 테스트
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] }
    }
  ],

  // 로컬 서버 시작
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI
  }
})
```

# 10. CI/CD 통합

## GitHub Actions

```yaml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

# 11. Best Practices

## 좋은 E2E 테스트의 특징

```
✅ 사용자 관점
- 실제 사용자가 하는 행동과 같음
- "로그인 버튼 클릭" ○
- "로그인 API 호출" ✗

✅ 독립적
- 다른 테스트에 의존하지 않음
- 순서가 없어도 실행 가능

✅ 결정적
- 매번 같은 결과
- 외부 요소 영향 최소화

✅ 명확한 이름
test('사용자가 로그인할 수 있다') ○
test('test1') ✗

✅ 적절한 대기
- 요소가 나타날 때까지 기다림
- 고정 시간 대기 피하기
```

## 피해야 할 패턴

```
❌ 구현 세부사항 테스트
test('컴포넌트 상태가 변경된다') // 나쁨
test('사용자가 버튼 클릭 후 목록이 업데이트된다') // 좋음

❌ 고정 시간 대기
await page.waitForTimeout(1000) // 나쁨
await page.waitForSelector('.item') // 좋음

❌ 너무 많은 검증
test('모든 것을 테스트한다') // 나쁨
test('장바구니에 상품이 추가된다') // 좋음

❌ 외부 서비스 의존
test('실제 메일을 보낸다') // 나쁨
test('메일 발송 API가 호출된다') // 좋음
```

# 12. 테스트 조직 구조

## 파일 구조

```
tests/
├── e2e/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   ├── signup.spec.ts
│   │   └── logout.spec.ts
│   ├── products/
│   │   ├── search.spec.ts
│   │   ├── filter.spec.ts
│   │   └── detail.spec.ts
│   ├── checkout/
│   │   ├── cart.spec.ts
│   │   ├── payment.spec.ts
│   │   └── confirmation.spec.ts
│   └── fixtures.ts
├── fixtures/
│   └── test-data.ts
└── utils/
    └── helpers.ts
```

## Page Object Model

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page) {}

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.page.fill('input[name="email"]', email)
    await this.page.fill('input[name="password"]', password)
    await this.page.click('button[type="submit"]')
  }

  async isLoggedIn() {
    return await this.page.waitForURL('/dashboard')
  }
}

// tests/login.spec.ts
import { LoginPage } from '../pages/LoginPage'

test('로그인', async ({ page }) => {
  const loginPage = new LoginPage(page)
  await loginPage.goto()
  await loginPage.login('test@example.com', 'password123')
  await expect(loginPage.isLoggedIn()).toBeTruthy()
})
```

# 13. 자주 묻는 질문

## Q: 얼마나 많은 E2E 테스트가 필요한가?

**A:** 테스트 피라미드 따르기
```
단위 테스트: 60%
통합 테스트: 30%
E2E 테스트: 10%

즉, 전체 테스트의 10% 정도만 E2E 테스트
```

## Q: E2E 테스트가 느린데?

**A:**
```typescript
// 1. 병렬 실행
workers: 4

// 2. 중요한 것만 테스트
// 모든 화면을 테스트하지 말고,
// 핵심 사용자 흐름만 테스트

// 3. 로컬 개발 서버 사용
webServer: { command: 'npm run dev' }
```

## Q: 이미지나 동영상을 클릭하려면?

**A:**
```typescript
// alt text로 찾기
const button = page.getByAltText('Submit')

// Accessible name으로 찾기
const button = page.getByRole('button', { name: 'Submit' })

// 또는 좌표로 클릭
const image = page.locator('img')
await image.click()
```

# 14. 체크리스트

E2E 테스트 시작하기:

```
[ ] E2E 테스트 도구 선택 (Playwright 추천)
[ ] 테스트 환경 설정
[ ] 첫 테스트 작성 및 실행
[ ] 핵심 사용자 흐름 파악
[ ] 테스트 케이스 작성
[ ] Page Object Model 구현
[ ] 데이터 기반 테스트
[ ] CI/CD 통합
[ ] 크로스 브라우저 테스트
[ ] 성능 최적화
[ ] 팀 협업 규칙 정의
```

# 결론

E2E 테스트는:

✅ 사용자 관점의 테스트
✅ 전체 애플리케이션 검증
✅ 자동화로 품질 보증
✅ 배포 전 신뢰도 확보

**핵심 사용자 흐름부터 E2E 테스트를 시작하세요!**

## 참고 링크

- [Playwright 공식 사이트](https://playwright.dev)
- [Cypress 공식 사이트](https://www.cypress.io)
- [Selenium 공식 사이트](https://www.selenium.dev)
