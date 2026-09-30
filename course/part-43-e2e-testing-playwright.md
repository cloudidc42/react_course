# Part 43: E2E Testing with Playwright

## ข้อมูล Part
- **Steps:** 1321-1360
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3 ชั่วโมง
- **Prerequisites:** Part 42 (Testing React)

---

## สารบัญ

1. [E2E Testing Introduction](#1-e2e-testing-introduction)
2. [Playwright Setup](#2-playwright-setup)
3. [Page Object Model](#3-page-object-model)
4. [Assertions](#4-assertions)
5. [API Testing](#5-api-testing)
6. [Visual Testing](#6-visual-testing)
7. [CI Integration](#7-ci-integration)
8. [Quiz](#quiz)

---

## Step 1321: E2E Testing Introduction

### 1. E2E Testing Introduction

E2E (End-to-End) Testing จำลองการใช้งานจริงของ User

#### ทำไมต้องใช้ E2E?

```
Unit/Integration Tests:
✓ เร็ว, Isolated
✗ ไม่รู้ว่า System ทั้งหมดทำงานร่วมกันได้ไหม

E2E Tests:
✓ ทดสอบ User Flow จริง
✓ ครอบคลุม Frontend + Backend + DB
✗ ช้ากว่า, ดูแลยากกว่า
```

#### Playwright vs Cypress vs Selenium

```
Playwright (Microsoft):
✓ รองรับ Chromium, Firefox, WebKit
✓ Fast Parallel Execution
✓ Built-in API Testing
✓ Visual Comparison

Cypress:
✓ Developer Experience ดี
✗ รองรับเฉพาะ Chromium
✗ ราคา (สำหรับ Features บางอย่าง)

Selenium:
✓ เก่าแก่ รองรับทุก Browser
✗ ช้า, Setup ยาก
```

---

## Step 1325: Playwright Setup

### 2. Playwright Setup

```bash
# ติดตั้ง
npm init playwright@latest

# หรือ
npm install --save-dev @playwright/test
npx playwright install  # ติดตั้ง Browsers
```

#### playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  
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
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

#### โครงสร้าง Test

```typescript
// tests/e2e/home.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Home Page', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
  })
  
  test('has title', async ({ page }) => {
    await expect(page).toHaveTitle(/My App/)
  })
  
  test('has navigation', async ({ page }) => {
    await expect(page.getByRole('navigation')).toBeVisible()
  })
  
  test('navigate to about page', async ({ page }) => {
    await page.getByRole('link', { name: /about/i }).click()
    await expect(page).toHaveURL('/about')
  })
})
```

---

## Step 1330: Page Object Model

### 3. Page Object Model

Page Object Model (POM) แยก Logic ออกจาก Test

```typescript
// tests/e2e/pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test'

export class LoginPage {
  readonly page: Page
  readonly emailInput: Locator
  readonly passwordInput: Locator
  readonly submitButton: Locator
  readonly errorMessage: Locator
  
  constructor(page: Page) {
    this.page = page
    this.emailInput = page.getByLabel(/email/i)
    this.passwordInput = page.getByLabel(/password/i)
    this.submitButton = page.getByRole('button', { name: /login/i })
    this.errorMessage = page.getByRole('alert')
  }
  
  async goto() {
    await this.page.goto('/login')
  }
  
  async login(email: string, password: string) {
    await this.emailInput.fill(email)
    await this.passwordInput.fill(password)
    await this.submitButton.click()
  }
  
  async expectErrorMessage(message: string) {
    await expect(this.errorMessage).toContainText(message)
  }
  
  async expectRedirectTo(url: string) {
    await expect(this.page).toHaveURL(url)
  }
}
```

#### ใช้งาน Page Object

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from './pages/LoginPage'

test.describe('Authentication', () => {
  let loginPage: LoginPage
  
  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page)
    await loginPage.goto()
  })
  
  test('successful login', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123')
    await loginPage.expectRedirectTo('/dashboard')
    await expect(page.getByText(/welcome/i)).toBeVisible()
  })
  
  test('invalid credentials', async () => {
    await loginPage.login('wrong@example.com', 'wrongpassword')
    await loginPage.expectErrorMessage('Invalid credentials')
  })
  
  test('empty form validation', async () => {
    await loginPage.submitButton.click()
    await loginPage.expectErrorMessage('Email is required')
  })
})
```

#### Complex Flow Test

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from './pages/LoginPage'
import { ProductPage } from './pages/ProductPage'
import { CartPage } from './pages/CartPage'
import { CheckoutPage } from './pages/CheckoutPage'

test('complete checkout flow', async ({ page }) => {
  // Login
  const loginPage = new LoginPage(page)
  await loginPage.goto()
  await loginPage.login('user@example.com', 'password123')
  
  // Add to Cart
  const productPage = new ProductPage(page)
  await productPage.goto('/products/1')
  await productPage.addToCart()
  
  // View Cart
  const cartPage = new CartPage(page)
  await cartPage.goto()
  await cartPage.expectItemCount(1)
  
  // Checkout
  const checkoutPage = new CheckoutPage(page)
  await checkoutPage.goto()
  await checkoutPage.fillShippingAddress({
    name: 'John Doe',
    address: '123 Main St',
    city: 'Bangkok',
  })
  await checkoutPage.submitOrder()
  
  // Verify
  await expect(page.getByText(/order confirmed/i)).toBeVisible()
})
```

---

## Step 1337: Assertions

### 4. Assertions

#### Built-in Assertions

```typescript
// Element Visibility
await expect(locator).toBeVisible()
await expect(locator).toBeHidden()

// Text Content
await expect(locator).toHaveText('Hello World')
await expect(locator).toContainText('Hello')
await expect(locator).toHaveText(/hello/i)  // RegExp

// Input Values
await expect(input).toHaveValue('test@example.com')
await expect(checkbox).toBeChecked()
await expect(checkbox).not.toBeChecked()

// Attributes
await expect(locator).toHaveAttribute('href', '/about')
await expect(locator).toHaveClass('active')

// URL
await expect(page).toHaveURL('/dashboard')
await expect(page).toHaveURL(/dashboard/)

// Title
await expect(page).toHaveTitle('My App')

// Count
await expect(locator).toHaveCount(3)

// Enabled/Disabled
await expect(button).toBeEnabled()
await expect(button).toBeDisabled()
```

#### Soft Assertions

```typescript
// Soft Assertion - ไม่ Stop เมื่อ Fail
test('soft assertions', async ({ page }) => {
  await page.goto('/')
  
  await expect.soft(page).toHaveTitle('My App')
  await expect.soft(page.getByText('Welcome')).toBeVisible()
  await expect.soft(page.getByRole('navigation')).toBeVisible()
  
  // ตรวจสอบทั้งหมดก่อน Fail
})
```

---

## Step 1342: API Testing

### 5. API Testing

```typescript
// tests/e2e/api.spec.ts
import { test, expect } from '@playwright/test'

test.describe('API Tests', () => {
  test('GET /api/posts', async ({ request }) => {
    const response = await request.get('/api/posts')
    
    expect(response.ok()).toBeTruthy()
    expect(response.status()).toBe(200)
    
    const data = await response.json()
    expect(data).toHaveProperty('posts')
    expect(Array.isArray(data.posts)).toBeTruthy()
  })
  
  test('POST /api/posts', async ({ request }) => {
    const response = await request.post('/api/posts', {
      data: {
        title: 'Test Post',
        content: 'Test Content',
      },
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.TEST_TOKEN}`,
      },
    })
    
    expect(response.status()).toBe(201)
    
    const post = await response.json()
    expect(post).toMatchObject({
      title: 'Test Post',
      content: 'Test Content',
    })
    expect(post.id).toBeDefined()
  })
  
  test('returns 404 for missing post', async ({ request }) => {
    const response = await request.get('/api/posts/99999')
    expect(response.status()).toBe(404)
  })
})
```

#### Authentication in API Tests

```typescript
// ตั้งค่า Auth State
test.use({
  extraHTTPHeaders: {
    'Authorization': `Bearer ${process.env.TEST_TOKEN}`,
  },
})
```

---

## Step 1347: Visual Testing

### 6. Visual Testing

#### Screenshot Comparison

```typescript
// tests/e2e/visual.spec.ts
import { test, expect } from '@playwright/test'

test('home page screenshot', async ({ page }) => {
  await page.goto('/')
  
  // Full Page Screenshot
  await expect(page).toHaveScreenshot('home.png')
  
  // Element Screenshot
  const hero = page.locator('.hero')
  await expect(hero).toHaveScreenshot('hero.png')
})
```

#### Update Snapshots

```bash
# สร้าง/Update Snapshots
npx playwright test --update-snapshots

# Run Visual Tests
npx playwright test --grep visual
```

---

## Step 1351: CI Integration

### 7. CI Integration

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Build app
        run: npm run build
        env:
          DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
      
      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000
      
      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## Step 1356: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## Playwright Best Practices

1. ใช้ Page Object Model เสมอ
   - ง่ายต่อการ Maintain
   - Reusable Code

2. ใช้ Role-based Selectors
   ✓ getByRole('button', { name: /submit/i })
   ✗ locator('.btn-submit')

3. ใช้ test.beforeEach สำหรับ Common Setup

4. Mock External Services ใน Tests
   - Stripe, Email Services

5. ใช้ Storage State สำหรับ Login
   - ไม่ต้อง Login ทุก Test

6. Parallel Tests - ระวัง Race Conditions

7. Visual Tests - อัพเดต Snapshots เมื่อ UI เปลี่ยน
```

---

## Quiz

### แบบทดสอบ Part 43

**คำถามที่ 1:** Page Object Model ช่วยอะไร?
- A) ทำให้ Tests เร็วขึ้น
- B) แยก Page Logic ออกจาก Test ทำให้ Maintain ง่ายขึ้น ✓
- C) ลด Code ที่ต้องเขียน
- D) รองรับหลาย Browser

**คำถามที่ 2:** `expect.soft()` ต่างจาก `expect()` อย่างไร?
- A) เร็วกว่า
- B) ไม่ Stop เมื่อ Fail ทำให้ตรวจสอบ Assertions ทั้งหมดได้ ✓
- C) Less Strict
- D) ไม่ต่างกัน

**คำถามที่ 3:** ทำไม Playwright ถึงดีกว่า Selenium?
- A) เก่าแก่กว่า
- B) เร็วกว่า รองรับหลาย Browser และมี Modern API ✓
- C) ฟรีกว่า
- D) Documentation ดีกว่า

**คำถามที่ 4:** `--update-snapshots` flag ใช้ทำอะไร?
- A) ลบ Snapshots เก่า
- B) สร้างหรืออัพเดต Screenshot Baselines ✓
- C) Run Tests ใหม่
- D) Clear Cache

**คำถามที่ 5:** ควร Upload อะไรใน CI Artifacts?
- A) Source Code
- B) Node Modules
- C) Playwright HTML Report เพื่อ Debug เมื่อ Fail ✓
- D) Database Backups

---

## สรุป Part 43

ใน Part นี้เราได้เรียนรู้:

1. **E2E Testing** - ทดสอบ User Flow จริง
2. **Playwright Setup** - Configuration และโครงสร้าง
3. **Page Object Model** - Organization Pattern
4. **Assertions** - Built-in และ Soft Assertions
5. **API Testing** - Test REST APIs ด้วย Playwright
6. **Visual Testing** - Screenshot Comparison
7. **CI Integration** - GitHub Actions

---

➡️ **Part ถัดไป:** [Part 44: Design System](./part-44-design-system.md)
