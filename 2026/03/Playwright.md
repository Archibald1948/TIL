# Playwright - 자동화 테스트와 웹 스크래핑의 완벽 솔루션

## 들어가며

수동으로 웹사이트를 테스트하는 것이 얼마나 번거로운가요?

```
수동 테스트:
1. 앱 실행
2. 로그인 클릭
3. 이메일 입력
4. 비밀번호 입력
5. 로그인 버튼 클릭
6. 로드 대기
7. 상품 검색
8. 결과 확인
9. 브라우저 새로고침
10. 다시 반복...

⏰ 시간이 엄청 오래 걸립니다
```

**Playwright**는 이 모든 과정을 자동으로 처리합니다. 한 번 자동화 스크립트를 작성하면, 언제든지 반복 실행할 수 있습니다.

## 공식 사이트

https://playwright.dev

# 1. Playwright란?

## 핵심 개념

```
Playwright = 자동화된 브라우저 제어

특징:
- Chromium, Firefox, WebKit 지원
- 크로스 브라우저 테스트
- E2E 테스트 자동화
- 웹 스크래핑
- 성능 테스트
- 시각적 비교
```

## Selenium vs Playwright

```
Selenium:
- 가장 오래된 도구
- 많은 기업에서 사용
- 상대적으로 느림

Playwright:
- 최신 기술
- 더 빠름
- 더 간단한 API
- 크로스 브라우저 지원
```

# 2. 설치 및 기본 설정

## 설치

```bash
npm install -D @playwright/test
npx playwright install
```

## 프로젝트 초기화

```bash
npm init playwright@latest
```

자동으로 다음 파일들이 생성됩니다:

```
playwright.config.ts
tests/
  ├── example.spec.ts
  └── ...
```

# 3. 기본 테스트 작성

## 첫 번째 테스트

```typescript
// tests/example.spec.ts
import { test, expect } from '@playwright/test';

test('기본 테스트', async ({ page }) => {
  // 1. 사이트 방문
  await page.goto('https://example.com');

  // 2. 요소 찾기
  const heading = page.locator('h1');

  // 3. 검증
  await expect(heading).toBeVisible();
  await expect(heading).toContainText('Example');
});
```

## 테스트 실행

```bash
# 모든 테스트 실행
npx playwright test

# 특정 파일 실행
npx playwright test tests/example.spec.ts

# 특정 테스트만 실행
npx playwright test -g "기본 테스트"

# UI 모드로 실행 (권장)
npx playwright test --ui

# 디버그 모드
npx playwright test --debug
```

# 4. 요소 선택 및 상호작용

## Locator 선택

```typescript
test('요소 선택하기', async ({ page }) => {
  // CSS 선택자
  const button = page.locator('button.submit');

  // XPath
  const link = page.locator('//a[@class="important"]');

  // Text로 찾기
  const loginButton = page.getByText('로그인');

  // Role로 찾기
  const input = page.getByRole('textbox', { name: '이메일' });

  // Label로 찾기
  const checkbox = page.getByLabel('동의합니다');

  // Placeholder로 찾기
  const search = page.getByPlaceholder('검색어를 입력하세요');
});
```

## 요소와 상호작용

```typescript
test('클릭, 입력, 선택', async ({ page }) => {
  await page.goto('https://example.com');

  // 클릭
  await page.click('button.submit');

  // 또는
  await page.locator('button.submit').click();

  // 텍스트 입력
  await page.fill('input[name="email"]', 'test@example.com');

  // 또는
  await page.locator('input[name="email"]').fill('test@example.com');

  // 텍스트 입력 (문자별)
  await page.type('input[name="password"]', 'password123');

  // 체크박스 선택
  await page.check('input[type="checkbox"]');

  // 라디오 버튼 선택
  await page.click('input[type="radio"][value="option1"]');

  // Select 드롭다운
  await page.selectOption('select', 'value1');

  // 키보드 입력
  await page.press('input', 'Enter');
  await page.keyboard.type('Hello');
  await page.keyboard.press('Tab');

  // 마우스 이동
  await page.mouse.move(100, 100);
  await page.mouse.click(100, 100);
  await page.mouse.dblclick(100, 100);

  // 스크롤
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
});
```

# 5. 검증 (Assertion)

## 요소 존재 확인

```typescript
test('요소 검증', async ({ page }) => {
  await page.goto('https://example.com');

  const button = page.locator('button');

  // 보이는지 확인
  await expect(button).toBeVisible();

  // 숨겨져 있는지 확인
  await expect(button).toBeHidden();

  // 활성화된 상태
  await expect(button).toBeEnabled();

  // 비활성화된 상태
  await expect(button).toBeDisabled();

  // 체크된 상태
  await expect(page.locator('input[type="checkbox"]')).toBeChecked();

  // 선택된 상태
  await expect(page.locator('option')).toBeSelected();

  // 포커스된 상태
  await expect(page.locator('input')).toBeFocused();
});
```

## 텍스트 검증

```typescript
test('텍스트 검증', async ({ page }) => {
  await page.goto('https://example.com');

  const heading = page.locator('h1');

  // 정확히 일치
  await expect(heading).toHaveText('Home');

  // 부분 일치
  await expect(heading).toContainText('Home');

  // 대소문자 무시
  await expect(heading).toContainText(/HOME/i);
});
```

## 속성 검증

```typescript
test('속성 검증', async ({ page }) => {
  await page.goto('https://example.com');

  const link = page.locator('a');

  // href 속성
  await expect(link).toHaveAttribute('href', '/page');

  // class 속성
  await expect(link).toHaveClass('active');

  // value 속성
  await expect(page.locator('input')).toHaveValue('john@example.com');

  // src 속성
  await expect(page.locator('img')).toHaveAttribute('src', /image\.png/);
});
```

## 개수 검증

```typescript
test('개수 검증', async ({ page }) => {
  await page.goto('https://example.com');

  // 요소 개수
  await expect(page.locator('li')).toHaveCount(5);

  // 최소/최대
  const items = page.locator('li');
  await expect(items).toHaveCount((value) => value >= 2);
});
```

# 6. 실전 테스트 시나리오

## 로그인 테스트

```typescript
test('사용자 로그인', async ({ page }) => {
  // 로그인 페이지 방문
  await page.goto('https://example.com/login');

  // 이메일 입력
  await page.fill('input[name="email"]', 'test@example.com');

  // 비밀번호 입력
  await page.fill('input[name="password"]', 'password123');

  // 로그인 버튼 클릭
  await page.click('button:has-text("로그인")');

  // 로그인 완료 대기
  await page.waitForURL('/dashboard');

  // 대시보드 확인
  await expect(page.locator('h1')).toContainText('대시보드');
});
```

## 폼 제출 테스트

```typescript
test('연락처 폼 제출', async ({ page }) => {
  await page.goto('https://example.com/contact');

  // 폼 작성
  await page.fill('input[name="name"]', 'John Doe');
  await page.fill('input[name="email"]', 'john@example.com');
  await page.fill('textarea[name="message"]', '안녕하세요');

  // 제출 버튼 클릭
  await page.click('button[type="submit"]');

  // 성공 메시지 대기
  await expect(page.locator('.success-message')).toBeVisible();
  await expect(page.locator('.success-message')).toContainText('감사합니다');
});
```

## 검색 기능 테스트

```typescript
test('상품 검색', async ({ page }) => {
  await page.goto('https://example.com/shop');

  // 검색창에 입력
  await page.fill('input[placeholder="검색"]', 'iPhone');

  // Enter 키 누르기
  await page.press('input[placeholder="검색"]', 'Enter');

  // 검색 결과 로드 대기
  await page.waitForSelector('.product-item');

  // 결과 확인
  const products = page.locator('.product-item');
  await expect(products.first()).toContainText('iPhone');
});
```

## 드롭다운 선택 테스트

```typescript
test('드롭다운 선택', async ({ page }) => {
  await page.goto('https://example.com');

  // 드롭다운 선택
  await page.selectOption('select[name="category"]', 'electronics');

  // 선택 확인
  await expect(page.locator('select[name="category"]')).toHaveValue(
    'electronics',
  );

  // 또는 getByLabel 사용
  await page.getByLabel('카테고리').selectOption('electronics');
});
```

# 7. 대기 및 타이밍

## 명시적 대기

```typescript
test('대기 처리', async ({ page }) => {
  await page.goto('https://example.com');

  // URL 변경 대기
  await page.waitForURL('/dashboard');

  // 특정 요소 나타날 때까지 대기
  await page.waitForSelector('.loader', { state: 'hidden' });

  // 요소가 보일 때까지 대기
  await page.waitForSelector('.content', { state: 'visible' });

  // 함수 실행 결과 대기
  await page.waitForFunction(() => {
    return document.querySelectorAll('.item').length === 10;
  });

  // 네트워크 요청 완료 대기
  await page.waitForLoadState('networkidle');

  // 콘텐츠 로드 완료 대기
  await page.waitForLoadState('domcontentloaded');

  // 전체 로드 완료
  await page.waitForLoadState('load');
});
```

## 타임아웃 설정

```typescript
test('타임아웃 설정', async ({ page }) => {
  // 기본 타임아웃: 30초

  // 특정 동작의 타임아웃 변경
  await page.click('button', { timeout: 5000 });

  // waitFor 타임아웃
  await page.waitForSelector('.item', { timeout: 10000 });

  // 전역 타임아웃 설정
  page.setDefaultTimeout(60000);
});
```

# 8. 멀티 페이지 및 팝업

## 새 탭/창 처리

```typescript
test('새 탭 열기', async ({ browser }) => {
  const page1 = await browser.newPage();
  await page1.goto('https://example.com');

  // 링크 클릭으로 새 탭 열기
  const [popup] = await Promise.all([
    page1.waitForEvent('popup'),
    page1.click('a[target="_blank"]'),
  ]);

  // 새 탭의 내용 확인
  await expect(popup.locator('h1')).toContainText('새 페이지');

  await page1.close();
  await popup.close();
});
```

## Context 사용

```typescript
test('여러 브라우저 컨텍스트', async ({ browser }) => {
  // 컨텍스트 1: 로그인한 사용자
  const context1 = await browser.newContext();
  const page1 = await context1.newPage();
  await page1.goto('https://example.com/login');
  // 로그인 처리...

  // 컨텍스트 2: 로그아웃한 사용자
  const context2 = await browser.newContext();
  const page2 = await context2.newPage();
  await page2.goto('https://example.com');
  // 비로그인 사용자 확인...

  await context1.close();
  await context2.close();
});
```

# 9. 스크린샷 및 비디오

## 스크린샷 촬영

```typescript
test('스크린샷 촬영', async ({ page }) => {
  await page.goto('https://example.com');

  // 전체 페이지 스크린샷
  await page.screenshot({ path: 'screenshot.png' });

  // 특정 요소만
  const button = page.locator('button');
  await button.screenshot({ path: 'button.png' });

  // 전체 페이지 (스크롤 포함)
  await page.screenshot({
    path: 'full-page.png',
    fullPage: true,
  });
});
```

## 비디오 녹화

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    // 모든 테스트 비디오 녹화
    video: 'retain-on-failure' // 실패한 테스트만
    // 또는
    video: 'on', // 모든 테스트
  }
})
```

## HTML 리포트

```bash
# 테스트 실행 후 리포트 생성
npx playwright test
npx playwright show-report
```

# 10. 설정 및 Fixture

## playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  // 브라우저 설정
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  // 로컬 서버 실행
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Fixture 작성

```typescript
// tests/fixtures.ts
import { test as base } from '@playwright/test';

type TestFixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<TestFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // 로그인 처리
    await page.goto('https://example.com/login');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("로그인")');
    await page.waitForURL('/dashboard');

    // 테스트에서 사용
    await use(page);

    // 정리
    // (필요시)
  },
});

// 사용
test('인증된 사용자 테스트', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/profile');
  await expect(authenticatedPage.locator('h1')).toContainText('프로필');
});
```

# 11. 웹 스크래핑

## 웹사이트 데이터 추출

```typescript
import { chromium } from 'playwright';

async function scrapeNews() {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  await page.goto('https://news.example.com');

  // 모든 뉴스 제목 추출
  const headlines = await page.locator('h2').allTextContents();

  // 각 기사의 링크와 제목
  const articles = await page.locator('.article').evaluateAll((elements) =>
    elements.map((el) => ({
      title: el.querySelector('h2')?.textContent,
      link: el.querySelector('a')?.getAttribute('href'),
      date: el.querySelector('.date')?.textContent,
    })),
  );

  console.log(articles);

  await browser.close();
}

scrapeNews();
```

## 동적 콘텐츠 스크래핑

```typescript
import { chromium } from 'playwright';

async function scrapeWithJavaScript() {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  await page.goto('https://example.com');

  // 모든 데이터 로드될 때까지 대기
  await page.waitForLoadState('networkidle');

  // JavaScript 실행 후 데이터 추출
  const data = await page.evaluate(() => {
    return Array.from(document.querySelectorAll('.item')).map((item) => ({
      title: item.querySelector('.title')?.textContent,
      price: item.querySelector('.price')?.textContent,
      rating: item.querySelector('.rating')?.textContent,
    }));
  });

  console.log(data);

  await browser.close();
}

scrapeWithJavaScript();
```

# 12. 팀 협업 Best Practices

## 테스트 구조화

```
tests/
├── auth/
│   ├── login.spec.ts
│   ├── logout.spec.ts
│   └── signup.spec.ts
├── products/
│   ├── search.spec.ts
│   ├── filter.spec.ts
│   └── detail.spec.ts
├── checkout/
│   ├── cart.spec.ts
│   └── payment.spec.ts
└── fixtures.ts
```

## Page Object Model

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page) {}

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.fill('input[name="email"]', email);
    await this.page.fill('input[name="password"]', password);
    await this.page.click('button[type="submit"]');
  }

  async isLoggedIn() {
    return await this.page.locator('.user-menu').isVisible();
  }
}

// tests/login.spec.ts
import { LoginPage } from '../pages/LoginPage';

test('로그인', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('test@example.com', 'password123');

  expect(await loginPage.isLoggedIn()).toBe(true);
});
```

# 13. CI/CD 통합

## GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

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

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

# 14. 자주 묻는 질문

## Q: Playwright vs Cypress?

**A:**

```
Playwright:
- 여러 브라우저 지원
- 더 빠름
- 웹 스크래핑도 가능
- 크로스 브라우저 테스트

Cypress:
- 더 간단한 학습곡선
- 더 나은 디버깅
- 한 브라우저씩만 테스트
- 더 큰 커뮤니티
```

## Q: 페이지 로드를 어떻게 기다리나?

**A:**

```typescript
// 방법 1: waitForLoadState
await page.waitForLoadState('networkidle');

// 방법 2: 특정 요소 대기
await page.waitForSelector('.content');

// 방법 3: URL 변경 대기
await page.waitForURL('/dashboard');
```

## Q: 동적 콘텐츠를 스크래핑하려면?

**A:**

```typescript
await page.waitForLoadState('networkidle');
const data = await page.evaluate(() => {
  // JavaScript로 데이터 추출
  return document.body.innerText;
});
```

# 15. 체크리스트

Playwright 프로젝트 시작:

```
[ ] @playwright/test 설치
[ ] playwright.config.ts 구성
[ ] 첫 테스트 작성
[ ] 로컬에서 테스트 실행
[ ] 여러 브라우저에서 테스트
[ ] Page Object Model 구현
[ ] Fixture 작성
[ ] 스크린샷/비디오 설정
[ ] CI/CD 통합
[ ] 팀 협업 규칙 정의
```

# 결론

Playwright는:

✅ 강력한 자동화 테스트
✅ 크로스 브라우저 지원
✅ 웹 스크래핑 가능
✅ 쉬운 API
✅ 빠른 실행 속도

**E2E 테스트가 필요하면 Playwright를 선택하세요!**

## 참고 링크

- [Playwright 공식 사이트](https://playwright.dev)
- [Playwright 공식 문서](https://playwright.dev/docs/intro)
- [Playwright GitHub](https://github.com/microsoft/playwright)
