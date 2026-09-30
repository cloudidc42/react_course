# Part 42: Testing React

## ข้อมูล Part
- **Steps:** 1276-1320
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3.5 ชั่วโมง
- **Prerequisites:** Part 41 (Deployment)

---

## สารบัญ

1. [Testing Pyramid](#1-testing-pyramid)
2. [Jest Setup](#2-jest-setup)
3. [React Testing Library](#3-react-testing-library)
4. [render, screen, fireEvent, userEvent](#4-render-screen-fireevent-userevent)
5. [Testing Components](#5-testing-components)
6. [Testing Custom Hooks](#6-testing-custom-hooks)
7. [Mocking](#7-mocking)
8. [Test Coverage](#8-test-coverage)
9. [Quiz](#quiz)

---

## Step 1276: Testing Pyramid

### 1. Testing Pyramid

```
         /\
        /  \
       / E2E \
      /--------\
     /Integration\
    /--------------\
   /   Unit Tests   \
  /------------------\
```

- **Unit Tests** - ทดสอบ Function/Component แยกส่วน (เยอะสุด, เร็วสุด)
- **Integration Tests** - ทดสอบ หลาย Component ทำงานร่วมกัน
- **E2E Tests** - ทดสอบ User Flow ทั้งหมด (น้อย, ช้า)

---

## Step 1279: Jest Setup

### 2. Jest Setup

```bash
# สำหรับ Next.js
npm install --save-dev jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

#### jest.config.ts

```typescript
import type { Config } from 'jest'
import nextJest from 'next/jest.js'

const createJestConfig = nextJest({
  dir: './',
})

const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  setupFilesAfterFramework: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
  ],
}

export default createJestConfig(config)
```

#### jest.setup.ts

```typescript
import '@testing-library/jest-dom'
```

#### package.json scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

---

## Step 1283: React Testing Library

### 3. React Testing Library

RTL (React Testing Library) ทดสอบ Component จากมุมมองของผู้ใช้

#### หลักการ

```
ทดสอบสิ่งที่ User เห็นและโต้ตอบ
ไม่ทดสอบ Implementation Details
```

#### Core Queries

```typescript
// getBy* - หา Element 1 อัน (throw ถ้าไม่เจอ)
screen.getByText('Hello')
screen.getByRole('button', { name: /submit/i })
screen.getByLabelText('Email')
screen.getByPlaceholderText('Enter email')
screen.getByTestId('submit-btn')

// queryBy* - หา Element 1 อัน (return null ถ้าไม่เจอ)
screen.queryByText('Error')

// findBy* - Async (return Promise)
await screen.findByText('Loading...')

// getAllBy* / queryAllBy* / findAllBy* - หลาย Elements
screen.getAllByRole('listitem')
```

---

## Step 1286: render, screen, fireEvent, userEvent

### 4. render, screen, fireEvent, userEvent

#### render

```typescript
import { render, screen } from '@testing-library/react'
import Button from '@/components/Button'

test('renders button', () => {
  render(<Button>Click me</Button>)
  
  expect(screen.getByRole('button')).toBeInTheDocument()
  expect(screen.getByText('Click me')).toBeInTheDocument()
})
```

#### fireEvent vs userEvent

```typescript
import userEvent from '@testing-library/user-event'

// fireEvent - Simulate DOM Event ตรงๆ (ไม่ Realistic)
fireEvent.click(button)
fireEvent.change(input, { target: { value: 'hello' } })

// userEvent - จำลอง User Interaction จริงๆ (แนะนำ)
const user = userEvent.setup()
await user.click(button)
await user.type(input, 'hello')
await user.keyboard('{Enter}')
await user.tab()
```

---

## Step 1289: Testing Components

### 5. Testing Components

#### Testing Button Component

```typescript
// components/Button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Button from './Button'

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument()
  })
  
  it('calls onClick when clicked', async () => {
    const user = userEvent.setup()
    const handleClick = jest.fn()
    
    render(<Button onClick={handleClick}>Click me</Button>)
    
    await user.click(screen.getByRole('button'))
    
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>)
    
    expect(screen.getByRole('button')).toBeDisabled()
  })
  
  it('shows loading state', () => {
    render(<Button loading>Submit</Button>)
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument()
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

#### Testing Form Component

```typescript
// components/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import LoginForm from './LoginForm'

describe('LoginForm', () => {
  const mockOnSubmit = jest.fn()
  
  beforeEach(() => {
    mockOnSubmit.mockClear()
  })
  
  it('renders form fields', () => {
    render(<LoginForm onSubmit={mockOnSubmit} />)
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument()
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /login/i })).toBeInTheDocument()
  })
  
  it('submits with valid data', async () => {
    const user = userEvent.setup()
    render(<LoginForm onSubmit={mockOnSubmit} />)
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com')
    await user.type(screen.getByLabelText(/password/i), 'password123')
    await user.click(screen.getByRole('button', { name: /login/i }))
    
    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      })
    })
  })
  
  it('shows validation errors for empty fields', async () => {
    const user = userEvent.setup()
    render(<LoginForm onSubmit={mockOnSubmit} />)
    
    await user.click(screen.getByRole('button', { name: /login/i }))
    
    expect(await screen.findByText(/email is required/i)).toBeInTheDocument()
    expect(screen.getByText(/password is required/i)).toBeInTheDocument()
    expect(mockOnSubmit).not.toHaveBeenCalled()
  })
})
```

#### Testing Async Component

```typescript
// components/UserList.test.tsx
import { render, screen } from '@testing-library/react'
import UserList from './UserList'

global.fetch = jest.fn()

describe('UserList', () => {
  afterEach(() => {
    jest.clearAllMocks()
  })
  
  it('shows loading initially', () => {
    ;(fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => [],
    })
    
    render(<UserList />)
    expect(screen.getByText(/loading/i)).toBeInTheDocument()
  })
  
  it('shows users after loading', async () => {
    const users = [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ]
    
    ;(fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => users,
    })
    
    render(<UserList />)
    
    expect(await screen.findByText('Alice')).toBeInTheDocument()
    expect(screen.getByText('Bob')).toBeInTheDocument()
  })
  
  it('shows error on fetch failure', async () => {
    ;(fetch as jest.Mock).mockRejectedValueOnce(new Error('Network error'))
    
    render(<UserList />)
    
    expect(await screen.findByText(/error/i)).toBeInTheDocument()
  })
})
```

---

## Step 1295: Testing Custom Hooks

### 6. Testing Custom Hooks

#### renderHook

```typescript
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react'
import useCounter from './useCounter'

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter())
    expect(result.current.count).toBe(0)
  })
  
  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10))
    expect(result.current.count).toBe(10)
  })
  
  it('increments count', () => {
    const { result } = renderHook(() => useCounter())
    
    act(() => {
      result.current.increment()
    })
    
    expect(result.current.count).toBe(1)
  })
  
  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5))
    
    act(() => {
      result.current.decrement()
    })
    
    expect(result.current.count).toBe(4)
  })
  
  it('resets count', () => {
    const { result } = renderHook(() => useCounter(5))
    
    act(() => {
      result.current.increment()
      result.current.increment()
      result.current.reset()
    })
    
    expect(result.current.count).toBe(5) // Back to initial
  })
})
```

#### Testing Hook with Context

```typescript
// hooks/useAuth.test.ts
import { renderHook } from '@testing-library/react'
import { AuthProvider } from '@/contexts/AuthContext'
import { useAuth } from './useAuth'

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <AuthProvider>{children}</AuthProvider>
)

describe('useAuth', () => {
  it('returns null user when not logged in', () => {
    const { result } = renderHook(() => useAuth(), { wrapper })
    expect(result.current.user).toBeNull()
  })
})
```

---

## Step 1300: Mocking

### 7. Mocking

#### Mock Module

```typescript
// __mocks__/next/navigation.ts
export const useRouter = jest.fn(() => ({
  push: jest.fn(),
  replace: jest.fn(),
  back: jest.fn(),
  prefetch: jest.fn(),
}))

export const usePathname = jest.fn(() => '/')
export const useSearchParams = jest.fn(() => new URLSearchParams())
```

#### Mock ใน Test File

```typescript
// components/NavBar.test.tsx
import { render, screen } from '@testing-library/react'
import { useRouter } from 'next/navigation'
import NavBar from './NavBar'

jest.mock('next/navigation', () => ({
  useRouter: jest.fn(),
  usePathname: jest.fn(),
}))

describe('NavBar', () => {
  it('highlights active link', () => {
    ;(usePathname as jest.Mock).mockReturnValue('/about')
    
    render(<NavBar />)
    
    const aboutLink = screen.getByRole('link', { name: /about/i })
    expect(aboutLink).toHaveClass('active')
  })
})
```

#### Mock fetch/API Calls

```typescript
// Mock global fetch
beforeEach(() => {
  global.fetch = jest.fn()
})

afterEach(() => {
  jest.restoreAllMocks()
})

// Mock สำหรับ Success
;(fetch as jest.Mock).mockResolvedValueOnce({
  ok: true,
  json: jest.fn().mockResolvedValueOnce({ data: 'test' }),
})

// Mock สำหรับ Error
;(fetch as jest.Mock).mockRejectedValueOnce(new Error('Failed'))
```

#### Mock Prisma (Server Components)

```typescript
// __mocks__/@/lib/prisma.ts
export const prisma = {
  user: {
    findMany: jest.fn(),
    findUnique: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  },
  post: {
    findMany: jest.fn(),
    create: jest.fn(),
  },
}
```

---

## Step 1310: Test Coverage

### 8. Test Coverage

```bash
# Run Coverage
npm run test:coverage

# Output:
# ✓ Button.test.tsx (3 tests)
# ✓ LoginForm.test.tsx (5 tests)
# 
# Coverage Report:
# File          | % Stmts | % Branch | % Funcs | % Lines
# Button.tsx    |   100   |   95.2   |   100   |   100
# LoginForm.tsx |   87.5  |   80.0   |   88.9  |   87.5
```

#### jest.config.ts Coverage Thresholds

```typescript
const config: Config = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
    },
  },
}
```

---

## Step 1315: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## Testing Best Practices

1. ทดสอบ User Behavior ไม่ใช่ Implementation
   ✓ getByRole, getByText, getByLabelText
   ✗ getByClassName, getByTagName

2. ใช้ data-testid เป็น Last Resort
   ใช้ getByRole ก่อนเสมอ

3. Avoid Testing Implementation Details
   ✗ ทดสอบว่า state = { count: 1 }
   ✓ ทดสอบว่า Screen แสดง "1"

4. One Assertion per Test (ไม่จำเป็น แต่ช่วยให้ Debug ง่าย)

5. ใช้ describe() จัดกลุ่ม Tests

6. beforeEach/afterEach สำหรับ Setup/Teardown

7. Mock External Dependencies (API, DB)
```

---

## Quiz

### แบบทดสอบ Part 42

**คำถามที่ 1:** ทำไมควรใช้ `userEvent` แทน `fireEvent`?
- A) เร็วกว่า
- B) จำลอง User Interaction จริงๆ ได้สมจริงกว่า ✓
- C) ใช้ยากกว่า
- D) ต้อง Import เพิ่ม

**คำถามที่ 2:** `screen.queryByText('Error')` ต่างจาก `screen.getByText('Error')` อย่างไร?
- A) queryBy เร็วกว่า
- B) queryBy return null ถ้าไม่เจอ แทนที่จะ throw ✓
- C) queryBy Case Sensitive
- D) ไม่ต่างกัน

**คำถามที่ 3:** `renderHook` ใช้ทำอะไร?
- A) Render Component
- B) Test Custom Hooks โดยไม่ต้องสร้าง Component ✓
- C) Mock API
- D) Test Async Functions

**คำถามที่ 4:** ควรใช้ `data-testid` เมื่อใด?
- A) ทุก Element
- B) เมื่อไม่มีวิธีอื่น (Last Resort) ✓
- C) ไม่ควรใช้เลย
- D) เฉพาะ Form Elements

**คำถามที่ 5:** Coverage Threshold คืออะไร?
- A) ขนาดสูงสุดของ Test File
- B) เปอร์เซ็นต์ขั้นต่ำของ Code ที่ต้องถูก Test ✓
- C) จำนวน Tests ขั้นต่ำ
- D) เวลาสูงสุดในการ Run Tests

---

## สรุป Part 42

ใน Part นี้เราได้เรียนรู้:

1. **Testing Pyramid** - Unit, Integration, E2E
2. **Jest Setup** - Configuration สำหรับ Next.js
3. **React Testing Library** - Queries และ Best Practices
4. **render/screen/userEvent** - Tools สำคัญ
5. **Testing Components** - Form, Async, Interactive
6. **Testing Custom Hooks** - renderHook
7. **Mocking** - fetch, modules, Prisma
8. **Test Coverage** - Measurement และ Thresholds

---

➡️ **Part ถัดไป:** [Part 43: E2E Testing with Playwright](./part-43-e2e-testing-playwright.md)
