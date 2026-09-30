# Part 58: Error Boundaries & Suspense ขั้นสูง

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1911-1945  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [Error Boundaries ขั้นสูง](#error-boundaries)
2. [Suspense ขั้นสูง](#suspense)
3. [Error Recovery](#error-recovery)
4. [Retry Logic](#retry-logic)
5. [useTransition กับ Error](#usetransition-error)
6. [Streaming SSR](#streaming-ssr)
7. [Quiz](#quiz)

---

## Step 1911: Error Boundaries ขั้นสูง {#error-boundaries}

Error Boundaries จับ JavaScript errors ใน component tree และแสดง fallback UI

### Custom Error Boundary

```typescript
// components/ErrorBoundary.tsx
import React, { Component, ErrorInfo } from 'react'

interface ErrorBoundaryState {
  hasError: boolean
  error: Error | null
  errorInfo: ErrorInfo | null
  errorId: string | null
}

interface ErrorBoundaryProps {
  children: React.ReactNode
  fallback?: React.ComponentType<ErrorFallbackProps>
  onError?: (error: Error, errorInfo: ErrorInfo) => void
  onReset?: () => void
  resetKeys?: Array<unknown>  // เมื่อ keys เปลี่ยน จะ reset boundary
  level?: 'page' | 'section' | 'component'
}

interface ErrorFallbackProps {
  error: Error
  resetError: () => void
  errorId: string | null
}

export class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props)
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null,
    }
    this.resetError = this.resetError.bind(this)
  }
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return {
      hasError: true,
      error,
      errorId: generateErrorId(),
    }
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by ErrorBoundary:', error, errorInfo)
    
    this.setState({ errorInfo })
    
    // ส่ง error ไปยัง monitoring service
    this.props.onError?.(error, errorInfo)
    
    // Log ไปยัง Sentry, Datadog, etc.
    logError(error, {
      errorInfo,
      errorId: this.state.errorId,
      level: this.props.level || 'component',
    })
  }
  
  componentDidUpdate(prevProps: ErrorBoundaryProps) {
    // Auto-reset เมื่อ resetKeys เปลี่ยน
    if (
      this.state.hasError &&
      this.props.resetKeys !== undefined &&
      this.props.resetKeys !== prevProps.resetKeys
    ) {
      const hasChangedKey = this.props.resetKeys.some(
        (key, index) => key !== prevProps.resetKeys?.[index]
      )
      
      if (hasChangedKey) {
        this.resetError()
      }
    }
  }
  
  resetError() {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null,
    })
    this.props.onReset?.()
  }
  
  render() {
    if (this.state.hasError && this.state.error) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback
      
      return (
        <FallbackComponent
          error={this.state.error}
          resetError={this.resetError}
          errorId={this.state.errorId}
        />
      )
    }
    
    return this.props.children
  }
}

function generateErrorId(): string {
  return `err_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
}

function logError(error: Error, context: any) {
  // Sentry
  if (typeof window !== 'undefined' && window.Sentry) {
    window.Sentry.captureException(error, { extra: context })
  }
  
  // หรือ custom analytics
  fetch('/api/errors', {
    method: 'POST',
    body: JSON.stringify({ error: error.message, stack: error.stack, ...context }),
    keepalive: true,
  }).catch(() => {})
}
```

### Default Error Fallback

```typescript
// components/ErrorFallback.tsx
'use client'

interface ErrorFallbackProps {
  error: Error
  resetError: () => void
  errorId: string | null
}

export function DefaultErrorFallback({
  error,
  resetError,
  errorId,
}: ErrorFallbackProps) {
  return (
    <div
      role="alert"
      className="min-h-[200px] flex flex-col items-center justify-center p-8 bg-red-50 border border-red-200 rounded-xl"
    >
      <div className="text-4xl mb-4">⚠️</div>
      <h2 className="text-lg font-semibold text-red-700 mb-2">
        เกิดข้อผิดพลาด
      </h2>
      <p className="text-sm text-red-600 mb-4 text-center max-w-md">
        {process.env.NODE_ENV === 'development'
          ? error.message
          : 'ขอโทษที่เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง'}
      </p>
      
      {process.env.NODE_ENV === 'development' && (
        <details className="mb-4 text-xs text-red-500 max-w-lg overflow-auto">
          <summary className="cursor-pointer">Stack trace</summary>
          <pre className="mt-2 whitespace-pre-wrap">{error.stack}</pre>
        </details>
      )}
      
      {errorId && (
        <p className="text-xs text-gray-400 mb-4">Error ID: {errorId}</p>
      )}
      
      <div className="flex gap-3">
        <button
          onClick={resetError}
          className="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 text-sm"
        >
          ลองใหม่
        </button>
        <button
          onClick={() => window.location.reload()}
          className="px-4 py-2 bg-white text-red-600 border border-red-300 rounded-lg hover:bg-red-50 text-sm"
        >
          Reload หน้า
        </button>
      </div>
    </div>
  )
}
```

### Next.js error.tsx

```typescript
// app/error.tsx - Global error page
'use client'

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log error
    console.error(error)
    logError(error)
  }, [error])
  
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold text-red-600 mb-4">เกิดข้อผิดพลาด!</h1>
        <p className="text-gray-600 mb-8">
          {error.message || 'บางอย่างผิดพลาด กรุณาลองใหม่'}
        </p>
        {error.digest && (
          <p className="text-xs text-gray-400 mb-4">
            Error Digest: {error.digest}
          </p>
        )}
        <button
          onClick={reset}
          className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
        >
          ลองใหม่
        </button>
      </div>
    </div>
  )
}

// app/blog/[slug]/error.tsx - Section error
'use client'

export default function BlogError({
  error,
  reset,
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div className="p-4 bg-yellow-50 border border-yellow-200 rounded-lg">
      <p>ไม่สามารถโหลดบทความได้ กรุณาลองใหม่</p>
      <button onClick={reset} className="mt-2 text-blue-600 hover:underline">
        ลองใหม่
      </button>
    </div>
  )
}
```

---

## Step 1920-1928: Suspense ขั้นสูง {#suspense}

```typescript
// Nested Suspense
export default function Page() {
  return (
    <div>
      {/* Header โหลดทันที */}
      <Header />
      
      {/* Hero โหลดก่อน */}
      <Suspense fallback={<HeroSkeleton />}>
        <Hero />
      </Suspense>
      
      {/* Products โหลดหลัง */}
      <Suspense fallback={<ProductGridSkeleton />}>
        <ProductGrid />
        
        {/* Reviews โหลดทีหลัง Products */}
        <Suspense fallback={<ReviewsSkeleton />}>
          <Reviews />
        </Suspense>
      </Suspense>
    </div>
  )
}
```

### Skeleton Loading Components

```typescript
// components/skeletons/ProductCardSkeleton.tsx
export function ProductCardSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="bg-gray-200 h-48 w-full rounded-xl mb-4" />
      <div className="space-y-2">
        <div className="bg-gray-200 h-4 w-3/4 rounded" />
        <div className="bg-gray-200 h-4 w-1/2 rounded" />
        <div className="bg-gray-200 h-6 w-1/3 rounded mt-3" />
      </div>
    </div>
  )
}

export function ProductGridSkeleton({ count = 6 }: { count?: number }) {
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 gap-6">
      {Array.from({ length: count }).map((_, i) => (
        <ProductCardSkeleton key={i} />
      ))}
    </div>
  )
}

// Shimmer effect ที่สวยงาม
export function ShimmerCard() {
  return (
    <div className="relative overflow-hidden bg-white rounded-xl shadow-sm">
      <div
        className="absolute inset-0 -translate-x-full animate-[shimmer_2s_infinite]"
        style={{
          backgroundImage:
            'linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent)',
        }}
      />
      <div className="bg-gray-200 h-48 w-full" />
      <div className="p-4 space-y-3">
        <div className="bg-gray-200 h-4 w-4/5 rounded" />
        <div className="bg-gray-200 h-4 w-3/5 rounded" />
        <div className="bg-gray-200 h-8 w-1/3 rounded mt-2" />
      </div>
    </div>
  )
}
```

---

## Step 1929-1934: Error Recovery {#error-recovery}

```typescript
// hooks/useErrorRecovery.ts
import { useState, useCallback, useTransition } from 'react'

interface UseErrorRecoveryOptions {
  maxRetries?: number
  retryDelay?: number
  onMaxRetriesReached?: (error: Error) => void
}

export function useErrorRecovery<T>({
  maxRetries = 3,
  retryDelay = 1000,
  onMaxRetriesReached,
}: UseErrorRecoveryOptions = {}) {
  const [error, setError] = useState<Error | null>(null)
  const [retryCount, setRetryCount] = useState(0)
  const [isRetrying, setIsRetrying] = useState(false)
  
  const execute = useCallback(
    async (fn: () => Promise<T>): Promise<T | null> => {
      try {
        setError(null)
        const result = await fn()
        setRetryCount(0)
        return result
      } catch (err) {
        const error = err instanceof Error ? err : new Error('Unknown error')
        setError(error)
        return null
      }
    },
    []
  )
  
  const retry = useCallback(
    async (fn: () => Promise<T>): Promise<T | null> => {
      if (retryCount >= maxRetries) {
        onMaxRetriesReached?.(error!)
        return null
      }
      
      setIsRetrying(true)
      
      // Exponential backoff
      const delay = retryDelay * Math.pow(2, retryCount)
      await new Promise((resolve) => setTimeout(resolve, delay))
      
      setRetryCount((prev) => prev + 1)
      setIsRetrying(false)
      
      return execute(fn)
    },
    [error, execute, maxRetries, retryCount, retryDelay, onMaxRetriesReached]
  )
  
  const clearError = useCallback(() => {
    setError(null)
    setRetryCount(0)
  }, [])
  
  return {
    error,
    retryCount,
    isRetrying,
    execute,
    retry,
    clearError,
    hasReachedMaxRetries: retryCount >= maxRetries,
  }
}
```

### Error Recovery UI

```typescript
// components/DataLoader.tsx
'use client'

import { useState, useEffect } from 'react'
import { useErrorRecovery } from '@/hooks/useErrorRecovery'

interface DataLoaderProps<T> {
  fetchFn: () => Promise<T>
  children: (data: T, refetch: () => void) => React.ReactNode
  loadingFallback?: React.ReactNode
  errorFallback?: (error: Error, retry: () => void, retryCount: number) => React.ReactNode
}

export function DataLoader<T>({
  fetchFn,
  children,
  loadingFallback,
  errorFallback,
}: DataLoaderProps<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  
  const { error, retryCount, isRetrying, execute, retry, hasReachedMaxRetries } =
    useErrorRecovery<T>({ maxRetries: 3, retryDelay: 1000 })
  
  async function load() {
    setLoading(true)
    const result = await execute(fetchFn)
    if (result) setData(result)
    setLoading(false)
  }
  
  async function handleRetry() {
    setLoading(true)
    const result = await retry(fetchFn)
    if (result) setData(result)
    setLoading(false)
  }
  
  useEffect(() => { load() }, [])
  
  if (loading || isRetrying) return <>{loadingFallback || <div>Loading...</div>}</>
  
  if (error) {
    if (errorFallback) {
      return <>{errorFallback(error, handleRetry, retryCount)}</>
    }
    
    return (
      <div className="p-4 bg-red-50 rounded-lg">
        <p className="text-red-600 mb-2">{error.message}</p>
        {!hasReachedMaxRetries ? (
          <button onClick={handleRetry} className="text-blue-600 text-sm hover:underline">
            ลองใหม่ ({retryCount}/3)
          </button>
        ) : (
          <p className="text-sm text-gray-500">
            ไม่สามารถโหลดข้อมูลได้ กรุณา refresh หน้า
          </p>
        )}
      </div>
    )
  }
  
  if (!data) return null
  
  return <>{children(data, load)}</>
}
```

---

## Step 1935-1940: Retry Logic {#retry-logic}

```typescript
// lib/fetchWithRetry.ts
interface RetryOptions {
  maxRetries?: number
  retryDelay?: number
  shouldRetry?: (error: Error, attempt: number) => boolean
  onRetry?: (error: Error, attempt: number) => void
}

export async function fetchWithRetry<T>(
  url: string,
  options?: RequestInit,
  retryOptions?: RetryOptions
): Promise<T> {
  const {
    maxRetries = 3,
    retryDelay = 1000,
    shouldRetry = (error, attempt) => attempt < maxRetries,
    onRetry,
  } = retryOptions || {}
  
  let lastError: Error
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options)
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }
      
      return response.json()
    } catch (error) {
      lastError = error instanceof Error ? error : new Error('Network error')
      
      if (!shouldRetry(lastError, attempt + 1)) {
        throw lastError
      }
      
      onRetry?.(lastError, attempt + 1)
      
      // Exponential backoff with jitter
      const delay = retryDelay * Math.pow(2, attempt) + Math.random() * 1000
      await new Promise((resolve) => setTimeout(resolve, delay))
    }
  }
  
  throw lastError!
}

// การใช้งาน
const data = await fetchWithRetry<Post[]>('/api/posts', undefined, {
  maxRetries: 3,
  retryDelay: 500,
  shouldRetry: (error, attempt) => {
    // Retry สำหรับ network errors หรือ 5xx errors
    return attempt <= 3 && !error.message.includes('404')
  },
  onRetry: (error, attempt) => {
    console.log(`Retry attempt ${attempt}:`, error.message)
  },
})
```

---

## Step 1941-1945: Streaming SSR {#streaming-ssr}

Streaming SSR ช่วยให้ Next.js ส่ง HTML ให้ Browser ได้เรื่อยๆ โดยไม่ต้องรอทุกอย่างเสร็จ

```
Traditional SSR:
Server ──► รอ fetch ทุกอย่างก่อน ──► ส่ง HTML ทั้งหมด ──► Browser

Streaming SSR:
Server ──► ส่ง HTML เริ่มต้น ──► Browser แสดงผล
       ──► ส่ง chunk เพิ่มเติม ──► Browser update
       ──► ส่ง chunk สุดท้าย ──► Browser เสร็จสมบูรณ์
```

```typescript
// app/page.tsx - Streaming ด้วย Suspense
import { Suspense } from 'react'

// Component ที่ fetch ข้อมูลช้า
async function SlowComponent() {
  // Simulate slow fetch
  await new Promise((resolve) => setTimeout(resolve, 3000))
  const data = await fetchData()
  return <div>{data.content}</div>
}

// Component ที่ fetch ข้อมูลเร็ว
async function FastComponent() {
  const data = await fetchFastData()
  return <div>{data.content}</div>
}

export default function Page() {
  return (
    <div>
      {/* โหลดทันที */}
      <h1>หน้าหลัก</h1>
      
      {/* ส่วนที่เร็ว */}
      <Suspense fallback={<FastSkeleton />}>
        <FastComponent />
      </Suspense>
      
      {/* ส่วนที่ช้า - ไม่ block ส่วนอื่น */}
      <Suspense fallback={<SlowSkeleton />}>
        <SlowComponent />
      </Suspense>
    </div>
  )
}
```

### Streaming กับ generateMetadata

```typescript
// app/blog/[slug]/page.tsx
// generateMetadata ไม่ stream - ต้อง resolve ก่อน
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return { title: post.title }
}

// Page component stream ได้
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.excerpt}</p>
      
      {/* Content อาจโหลดช้า */}
      <Suspense fallback={<ContentSkeleton />}>
        <PostContent postId={post.id} />
      </Suspense>
      
      {/* Comments โหลดช้าที่สุด */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={post.id} />
      </Suspense>
    </article>
  )
}
```

---

## 🧪 Quiz - Part 58

**ข้อ 1:** Error Boundary จับ Error ประเภทใดได้?
- A) ทุก Error รวมถึง Event handlers
- B) JavaScript errors ใน render, lifecycle, constructor
- C) Network errors เท่านั้น
- D) TypeScript type errors

**ข้อ 2:** `useTransition` ช่วยเรื่อง Error อย่างไร?
- A) Catch errors อัตโนมัติ
- B) ทำให้ error recovery ไม่ block UI
- C) Retry อัตโนมัติ
- D) Log errors

**ข้อ 3:** Streaming SSR ช่วยอะไร?
- A) ลด bundle size
- B) ส่ง HTML ให้ browser ได้ทีละส่วน ไม่ต้องรอทุกอย่างเสร็จ
- C) Cache HTML
- D) ลด JavaScript

**ข้อ 4:** `app/error.tsx` ทำงานอย่างไร?
- A) เป็น Client Component ที่จับ errors ทุก route ใต้มัน
- B) เป็น Server Component
- C) ใช้กับ API routes เท่านั้น
- D) เป็น middleware

**เฉลย:** 1-B, 2-B, 3-B, 4-A

---

> **➡️ Next:** [Part 59: Security Best Practices](./part-59-security-best-practices.md)
