# Part 24: Error Handling - การจัดการข้อผิดพลาด

**Step 596-625** | ระดับ: ปานกลาง-สูง | เวลาเรียน: 2-3 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [Error Boundaries คืออะไร?](#error-boundaries-คืออะไร)
2. [สร้าง Error Boundary](#สร้าง-error-boundary)
3. [Try/Catch ใน Async Functions](#trycatch-ใน-async-functions)
4. [Global Error Handling](#global-error-handling)
5. [Error UI Patterns](#error-ui-patterns)
6. [Retry Logic](#retry-logic)
7. [Fallback Components](#fallback-components)
8. [Sentry Integration](#sentry-integration)
9. [Quiz](#quiz)

---

## Step 596-600: Error Boundaries คืออะไร?

Error Boundary คือ React component ที่ดัก JavaScript errors ใน component tree และแสดง fallback UI แทน

### ปัญหาที่ Error Boundaries แก้ได้

```
ไม่มี Error Boundary:
Component A ที่มี error
→ ทำให้ทั้ง App crash และแสดงหน้าขาวเปล่า

มี Error Boundary:
Component A ที่มี error
→ Error Boundary แสดง fallback UI แทน
→ ส่วนอื่นของ App ยังทำงานปกติ
```

### สิ่งที่ Error Boundary จับได้ และไม่จับได้

| จับได้ | ไม่จับได้ |
|--------|---------|
| Render errors | Event handlers |
| Lifecycle method errors | Async code |
| Child component errors | Server-side rendering errors |
| Constructor errors | Errors ใน Error Boundary เอง |

---

## Step 601-605: สร้าง Error Boundary

Error Boundary ต้องเป็น Class Component เพราะใช้ `componentDidCatch` lifecycle

```jsx
// components/ErrorBoundary.jsx
import React from 'react'

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
    }
  }

  static getDerivedStateFromError(error) {
    // อัพเดท state เพื่อ render fallback UI
    return { hasError: true, error }
  }

  componentDidCatch(error, errorInfo) {
    // Log error ไปยัง error reporting service
    console.error('Error caught by boundary:', error, errorInfo)
    this.setState({ errorInfo })

    // ส่ง error ไป Sentry
    // Sentry.captureException(error, { extra: errorInfo })
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null, errorInfo: null })
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback
      }

      return (
        <div style={{
          padding: '2rem',
          textAlign: 'center',
          backgroundColor: '#fff5f5',
          borderRadius: '8px',
          border: '1px solid #fc8181',
          margin: '1rem',
        }}>
          <h2 style={{ color: '#c53030' }}>เกิดข้อผิดพลาดบางอย่าง</h2>
          <p style={{ color: '#742a2a' }}>
            {this.state.error?.message || 'เกิดข้อผิดพลาดที่ไม่รู้จัก'}
          </p>
          
          {import.meta.env.DEV && this.state.errorInfo && (
            <details style={{ textAlign: 'left', marginTop: '1rem' }}>
              <summary>รายละเอียดข้อผิดพลาด (Dev only)</summary>
              <pre style={{ fontSize: '0.75rem', overflow: 'auto' }}>
                {this.state.errorInfo.componentStack}
              </pre>
            </details>
          )}

          <button
            onClick={this.handleReset}
            style={{
              marginTop: '1rem',
              padding: '0.5rem 1.5rem',
              backgroundColor: '#e53e3e',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
            }}
          >
            ลองใหม่
          </button>
        </div>
      )
    }

    return this.props.children
  }
}

export default ErrorBoundary
```

### การใช้ Error Boundary

```jsx
// App.jsx
import ErrorBoundary from './components/ErrorBoundary'

// ครอบทั้งแอพ
function App() {
  return (
    <ErrorBoundary>
      <Router>
        {/* ... */}
      </Router>
    </ErrorBoundary>
  )
}

// ครอบแค่บางส่วน
function Dashboard() {
  return (
    <div>
      <ErrorBoundary fallback={<div>Widget โหลดไม่ได้</div>}>
        <ChartWidget />
      </ErrorBoundary>

      <ErrorBoundary fallback={<div>ตารางโหลดไม่ได้</div>}>
        <DataTable />
      </ErrorBoundary>
    </div>
  )
}
```

### Error Boundary กับ Key Reset

```jsx
// ใช้ key เพื่อ reset Error Boundary เมื่อ navigate
import { useLocation } from 'react-router-dom'

function App() {
  const location = useLocation()

  return (
    // key เปลี่ยนเมื่อ route เปลี่ยน → Error Boundary reset
    <ErrorBoundary key={location.pathname}>
      <Routes>
        {/* ... */}
      </Routes>
    </ErrorBoundary>
  )
}
```

---

## Step 606-610: Try/Catch ใน Async Functions

### Patterns ต่างๆ

```jsx
// Pattern 1: try/catch/finally
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`)
    
    if (!res.ok) {
      const errorData = await res.json().catch(() => ({}))
      throw new Error(errorData.message || `HTTP ${res.status}`)
    }
    
    return await res.json()
  } catch (error) {
    // Log และ re-throw
    console.error('fetchUser error:', error)
    throw error
  } finally {
    // ทำงานเสมอ ไม่ว่าจะ success หรือ error
    console.log('fetchUser completed')
  }
}
```

```jsx
// Pattern 2: Result pattern (ไม่ throw)
async function safeGetUser(id) {
  try {
    const data = await fetchUser(id)
    return { data, error: null }
  } catch (error) {
    return { data: null, error: error.message }
  }
}

// การใช้งาน
const { data: user, error } = await safeGetUser(userId)
if (error) {
  console.error(error)
  return
}
// ใช้ user ได้เลย
```

```jsx
// Pattern 3: Promise-based error handling
async function fetchWithErrorHandling(url) {
  return fetch(url)
    .then(res => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      return res.json()
    })
    .catch(error => {
      console.error('Fetch error:', error)
      throw error
    })
}
```

### Custom Error Classes

```jsx
// utils/errors.js
class ApiError extends Error {
  constructor(message, status, data) {
    super(message)
    this.name = 'ApiError'
    this.status = status
    this.data = data
  }
}

class NetworkError extends Error {
  constructor() {
    super('ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้')
    this.name = 'NetworkError'
  }
}

class ValidationError extends Error {
  constructor(errors) {
    super('ข้อมูลไม่ถูกต้อง')
    this.name = 'ValidationError'
    this.errors = errors
  }
}

// ใช้งาน
async function createProduct(data) {
  try {
    const res = await api.post('/products', data)
    return res.data
  } catch (error) {
    if (error.response?.status === 422) {
      throw new ValidationError(error.response.data.errors)
    }
    if (error.response) {
      throw new ApiError(
        error.response.data.message,
        error.response.status,
        error.response.data
      )
    }
    throw new NetworkError()
  }
}

// ในการใช้งาน
try {
  await createProduct(formData)
} catch (error) {
  if (error instanceof ValidationError) {
    setFieldErrors(error.errors)
  } else if (error instanceof ApiError) {
    setErrorMessage(error.message)
  } else {
    setErrorMessage('เกิดข้อผิดพลาดที่ไม่ทราบสาเหตุ')
  }
}
```

---

## Step 611-615: Global Error Handling

### Window Error Events

```jsx
// main.jsx
// จับ JavaScript errors ทั้งหมด
window.addEventListener('error', (event) => {
  console.error('Global JS Error:', event.error)
  // ส่งไป error reporting service
})

// จับ Unhandled Promise Rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled Promise Rejection:', event.reason)
  event.preventDefault()  // ป้องกัน browser แสดง error
})
```

### Error Context

```jsx
// context/ErrorContext.jsx
import { createContext, useContext, useState } from 'react'

const ErrorContext = createContext(null)

export function ErrorProvider({ children }) {
  const [errors, setErrors] = useState([])

  const addError = (error) => {
    const id = Date.now()
    setErrors(prev => [...prev, { id, message: error, timestamp: new Date() }])
    
    // Auto dismiss หลัง 5 วินาที
    setTimeout(() => {
      removeError(id)
    }, 5000)
  }

  const removeError = (id) => {
    setErrors(prev => prev.filter(e => e.id !== id))
  }

  const clearErrors = () => setErrors([])

  return (
    <ErrorContext.Provider value={{ errors, addError, removeError, clearErrors }}>
      {children}
      <ErrorToastContainer errors={errors} onClose={removeError} />
    </ErrorContext.Provider>
  )
}

export function useError() {
  return useContext(ErrorContext)
}
```

### Error Toast Container

```jsx
// components/ErrorToastContainer.jsx
function ErrorToastContainer({ errors, onClose }) {
  if (errors.length === 0) return null

  return (
    <div style={{
      position: 'fixed',
      bottom: '1rem',
      right: '1rem',
      zIndex: 9999,
      display: 'flex',
      flexDirection: 'column',
      gap: '0.5rem',
    }}>
      {errors.map(error => (
        <div
          key={error.id}
          style={{
            backgroundColor: '#c53030',
            color: 'white',
            padding: '1rem 1.5rem',
            borderRadius: '8px',
            boxShadow: '0 4px 6px rgba(0,0,0,0.1)',
            display: 'flex',
            alignItems: 'center',
            gap: '1rem',
            minWidth: '300px',
            animation: 'slideIn 0.3s ease',
          }}
        >
          <span style={{ flex: 1 }}>{error.message}</span>
          <button
            onClick={() => onClose(error.id)}
            style={{
              background: 'none',
              border: 'none',
              color: 'white',
              cursor: 'pointer',
              fontSize: '1.2rem',
            }}
          >
            ✕
          </button>
        </div>
      ))}
      <style>{`
        @keyframes slideIn {
          from { transform: translateX(100%); opacity: 0; }
          to { transform: translateX(0); opacity: 1; }
        }
      `}</style>
    </div>
  )
}
```

---

## Step 616-618: Error UI Patterns

### Inline Error

```jsx
// ใช้ใต้ input field
function InputField({ label, error, ...props }) {
  return (
    <div style={{ marginBottom: '1rem' }}>
      <label>{label}</label>
      <input
        style={{
          width: '100%',
          padding: '0.5rem',
          border: `1px solid ${error ? '#e53e3e' : '#e2e8f0'}`,
          borderRadius: '4px',
        }}
        {...props}
      />
      {error && (
        <span style={{ color: '#e53e3e', fontSize: '0.875rem' }}>
          {error}
        </span>
      )}
    </div>
  )
}
```

### Banner Error

```jsx
function BannerError({ message, type = 'error', onClose }) {
  const styles = {
    error: { bg: '#fff5f5', border: '#fc8181', text: '#c53030' },
    warning: { bg: '#fffaf0', border: '#f6ad55', text: '#c05621' },
    info: { bg: '#ebf8ff', border: '#63b3ed', text: '#2b6cb0' },
  }

  const { bg, border, text } = styles[type]

  return (
    <div style={{
      backgroundColor: bg,
      border: `1px solid ${border}`,
      borderRadius: '8px',
      padding: '1rem',
      marginBottom: '1rem',
      display: 'flex',
      justifyContent: 'space-between',
      alignItems: 'flex-start',
    }}>
      <p style={{ color: text, margin: 0 }}>{message}</p>
      {onClose && (
        <button
          onClick={onClose}
          style={{ background: 'none', border: 'none', cursor: 'pointer', color: text }}
        >
          ✕
        </button>
      )}
    </div>
  )
}
```

### Full Page Error

```jsx
function FullPageError({ title, message, onRetry, onGoHome }) {
  return (
    <div style={{
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center',
      minHeight: '100vh',
      padding: '2rem',
      textAlign: 'center',
    }}>
      <div style={{ fontSize: '5rem', marginBottom: '1rem' }}>💔</div>
      <h1 style={{ color: '#c53030' }}>{title || 'เกิดข้อผิดพลาด'}</h1>
      <p style={{ color: '#718096', maxWidth: '500px' }}>
        {message || 'กรุณาลองใหม่อีกครั้งหรือติดต่อผู้ดูแลระบบ'}
      </p>
      <div style={{ display: 'flex', gap: '1rem', marginTop: '2rem' }}>
        {onRetry && (
          <button onClick={onRetry} style={buttonStyle('#e53e3e')}>
            ลองใหม่
          </button>
        )}
        <button onClick={onGoHome || (() => window.location.href = '/')}
          style={buttonStyle('#4a5568')}>
          กลับหน้าแรก
        </button>
      </div>
    </div>
  )
}

const buttonStyle = (bg) => ({
  padding: '0.75rem 1.5rem',
  backgroundColor: bg,
  color: 'white',
  border: 'none',
  borderRadius: '8px',
  cursor: 'pointer',
  fontSize: '1rem',
})
```

---

## Step 619-621: Retry Logic

### Basic Retry

```jsx
// utils/retry.js
async function withRetry(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    onRetry = null,
  } = options

  let lastError

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error

      if (attempt === maxAttempts) break

      // อย่า retry ถ้า 4xx error (client error)
      if (error.response?.status >= 400 && error.response?.status < 500) {
        throw error
      }

      onRetry?.(attempt, error)

      // Exponential backoff
      const waitTime = delay * Math.pow(2, attempt - 1)
      await new Promise(resolve => setTimeout(resolve, waitTime))
    }
  }

  throw lastError
}

// การใช้งาน
const data = await withRetry(
  () => fetch('/api/unstable-endpoint').then(r => r.json()),
  {
    maxAttempts: 3,
    delay: 1000,
    onRetry: (attempt, error) => {
      console.log(`Retry ${attempt}: ${error.message}`)
    }
  }
)
```

### useRetry Hook

```jsx
// hooks/useRetry.js
import { useState, useCallback } from 'react'

function useRetry(fn, options = {}) {
  const { maxAttempts = 3, delay = 1000 } = options
  const [attempt, setAttempt] = useState(0)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)
  const [data, setData] = useState(null)

  const execute = useCallback(async () => {
    setLoading(true)
    setError(null)

    for (let i = 1; i <= maxAttempts; i++) {
      try {
        setAttempt(i)
        const result = await fn()
        setData(result)
        setLoading(false)
        return result
      } catch (err) {
        if (i === maxAttempts) {
          setError(err.message)
          setLoading(false)
          throw err
        }
        await new Promise(resolve => setTimeout(resolve, delay * i))
      }
    }
  }, [fn, maxAttempts, delay])

  return { execute, loading, error, data, attempt }
}

export default useRetry
```

---

## Step 622-623: Fallback Components

```jsx
// components/fallbacks/NetworkError.jsx
function NetworkError({ onRetry }) {
  return (
    <div style={{ textAlign: 'center', padding: '3rem' }}>
      <div style={{ fontSize: '4rem' }}>🌐</div>
      <h2>ไม่มีการเชื่อมต่ออินเทอร์เน็ต</h2>
      <p>กรุณาตรวจสอบการเชื่อมต่อของคุณแล้วลองอีกครั้ง</p>
      <button onClick={onRetry}>ลองอีกครั้ง</button>
    </div>
  )
}

// components/fallbacks/PermissionDenied.jsx
function PermissionDenied() {
  const navigate = useNavigate()

  return (
    <div style={{ textAlign: 'center', padding: '3rem' }}>
      <div style={{ fontSize: '4rem' }}>🚫</div>
      <h2>ไม่มีสิทธิ์เข้าถึง</h2>
      <p>คุณไม่มีสิทธิ์ดำเนินการนี้</p>
      <button onClick={() => navigate(-1)}>ย้อนกลับ</button>
    </div>
  )
}

// components/fallbacks/NotFound.jsx
function NotFound() {
  return (
    <div style={{ textAlign: 'center', padding: '3rem' }}>
      <div style={{ fontSize: '4rem' }}>🔍</div>
      <h2>ไม่พบข้อมูล</h2>
      <p>ข้อมูลที่คุณต้องการอาจถูกลบหรือไม่มีอยู่</p>
    </div>
  )
}
```

---

## Step 624-625: Sentry Integration

### การติดตั้ง Sentry

```bash
npm install @sentry/react
```

```jsx
// main.jsx
import * as Sentry from '@sentry/react'

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.MODE,  // development / production
  tracesSampleRate: 1.0,  // 100% performance monitoring
  
  // ข้อมูล user
  beforeSend(event) {
    const user = JSON.parse(localStorage.getItem('user') || '{}')
    if (user.id) {
      event.user = { id: user.id, email: user.email }
    }
    return event
  },
})
```

### Error Boundary กับ Sentry

```jsx
// components/ErrorBoundary.jsx
import * as Sentry from '@sentry/react'

// ใช้ Sentry's built-in Error Boundary
const SentryErrorBoundary = Sentry.withErrorBoundary(MyComponent, {
  fallback: <ErrorFallback />,
  showDialog: true,
})

// หรือ integrate กับ custom boundary
componentDidCatch(error, errorInfo) {
  Sentry.captureException(error, {
    extra: {
      componentStack: errorInfo.componentStack,
    },
  })
}
```

### Manual Error Capture

```jsx
import * as Sentry from '@sentry/react'

// Capture exception manually
try {
  await riskyOperation()
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: 'checkout' },
    extra: { orderId: order.id },
  })
  setError('เกิดข้อผิดพลาดในการชำระเงิน')
}

// Capture message
Sentry.captureMessage('User attempted to access restricted resource', 'warning')
```

---

## Tips และ Best Practices

### 1. Error Boundary Granularity

```jsx
// ✅ ดี - ครอบแต่ละส่วนที่อาจ fail แยกกัน
<Layout>
  <ErrorBoundary fallback={<HeaderFallback />}>
    <Header />
  </ErrorBoundary>
  
  <ErrorBoundary fallback={<PageFallback />}>
    <MainContent />
  </ErrorBoundary>
  
  <ErrorBoundary fallback={<SidebarFallback />}>
    <Sidebar />
  </ErrorBoundary>
</Layout>
```

### 2. อย่า Swallow Errors

```jsx
// ❌ ไม่ดี - ซ่อน error
try {
  await doSomething()
} catch (e) {
  // ไม่ทำอะไร
}

// ✅ ดี - Log และแสดง error
try {
  await doSomething()
} catch (e) {
  console.error('Error in doSomething:', e)
  Sentry.captureException(e)
  setError(e.message)
}
```

### 3. User-friendly Error Messages

```jsx
// ❌ ไม่ดี - error ทางเทคนิค
"TypeError: Cannot read property 'name' of undefined"

// ✅ ดี - ข้อความที่ user เข้าใจได้
"ไม่สามารถโหลดข้อมูลสินค้าได้ กรุณาลองใหม่อีกครั้ง"
```

---

## Quiz - Part 24

**ข้อ 1**: Error Boundary จับ error จาก Event Handler ได้ไหม?
- a) ได้เสมอ
- b) ไม่ได้ ต้องใช้ try/catch แทน
- c) ได้เฉพาะ onClick
- d) ได้เฉพาะ onSubmit

**ข้อ 2**: `componentDidCatch` lifecycle method ทำงานอย่างไร?
- a) เรียกก่อน render ทุกครั้ง
- b) เรียกเมื่อ child component มี error
- c) เรียกเมื่อ component unmount
- d) เรียกเมื่อ state เปลี่ยน

**ข้อ 3**: Exponential Backoff ในการ retry หมายความว่าอะไร?
- a) Retry ทุก 1 วินาทีเสมอ
- b) Retry ช้าลงเรื่อยๆ (1s, 2s, 4s, ...)
- c) Retry เร็วขึ้นเรื่อยๆ
- d) ไม่มี delay

**ข้อ 4**: ทำไมต้องแบ่ง Error Boundary เป็นส่วนๆ?
- a) ทำให้ code อ่านง่ายขึ้น
- b) ทำให้ส่วนอื่นของแอพยังทำงานได้เมื่อส่วนหนึ่ง error
- c) เพิ่ม performance
- d) ลด bundle size

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 24

ใน Part นี้คุณได้เรียนรู้:
- ✅ Error Boundaries และการสร้าง
- ✅ Try/Catch patterns
- ✅ Custom Error Classes
- ✅ Global Error Handling
- ✅ Error UI Patterns (Inline, Banner, Full Page)
- ✅ Retry Logic กับ Exponential Backoff
- ✅ Fallback Components
- ✅ Sentry Integration

---

## Part ถัดไป

➡️ **[Part 25: Performance Optimization](./part-25-performance-optimization.md)**
- React.memo
- useMemo และ useCallback
- Lazy Loading Images
- Virtualization
