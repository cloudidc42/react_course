# Part 38: Next.js Middleware

## ข้อมูล Part
- **Steps:** 1111-1145
- **ระดับ:** Intermediate to Advanced
- **เวลาเรียน:** 2-3 ชั่วโมง
- **Prerequisites:** Part 37 (Client Components)

---

## สารบัญ

1. [Middleware คืออะไร](#1-middleware-คืออะไร)
2. [middleware.ts](#2-middlewarets)
3. [Request/Response Manipulation](#3-requestresponse-manipulation)
4. [Auth Middleware](#4-auth-middleware)
5. [Redirect/Rewrite](#5-redirectrewrite)
6. [Geo-based Redirects](#6-geo-based-redirects)
7. [Rate Limiting Middleware](#7-rate-limiting-middleware)
8. [Logging](#8-logging)
9. [Quiz](#quiz)

---

## Step 1111: Middleware คืออะไร

### 1. Middleware คืออะไร

Middleware คือ Code ที่รันก่อนที่ Request จะถึง Route Handler, Page หรือ API

```
User Request
    ↓
[Middleware] ← รันที่นี่ก่อน
    ↓
Route (Page/API)
    ↓
Response
```

#### ทำงานที่ไหน

```
Next.js Middleware รันที่ Edge Runtime:
- ใกล้กับผู้ใช้มากกว่า (CDN Edge)
- เร็วกว่า Server-side Code
- มีข้อจำกัด: ไม่มี Node.js APIs บางอย่าง
- รองรับ Web APIs เท่านั้น
```

#### Use Cases

```
✓ Authentication & Authorization
✓ Redirect (Localization, A/B Testing)
✓ Request/Response Headers Manipulation
✓ Logging & Analytics
✓ Rate Limiting
✓ Bot Detection
✓ Geo-based Routing
✓ CORS
✓ Feature Flags
```

---

## Step 1113: middleware.ts

### 2. middleware.ts

#### ตำแหน่งไฟล์

```
project/
├── middleware.ts    ← ต้องอยู่ระดับ Root (หรือใน src/)
├── app/
├── pages/
└── public/
```

#### Structure พื้นฐาน

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Logic ของ Middleware
  return NextResponse.next()  // ให้ Request ผ่านไป
}

// กำหนด Routes ที่ Middleware ทำงาน
export const config = {
  matcher: '/about/:path*',
  // หรือ Array
  matcher: ['/dashboard/:path*', '/admin/:path*'],
  // หรือ Regex
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

#### Matcher Patterns

```typescript
export const config = {
  matcher: [
    // Match เฉพาะ paths
    '/dashboard',
    '/dashboard/:path*',
    
    // Match ยกเว้น static files
    '/((?!_next/static|_next/image|favicon.ico).*)',
    
    // Match API routes
    '/api/:path*',
    
    // ไม่ Match specific paths
    '/((?!api/public|_next).*)',
  ],
}
```

#### Conditional Middleware

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // Auth Check
  if (pathname.startsWith('/dashboard')) {
    return authMiddleware(request)
  }
  
  // Localization
  if (pathname === '/') {
    return localizationMiddleware(request)
  }
  
  // API Rate Limiting
  if (pathname.startsWith('/api/')) {
    return rateLimitMiddleware(request)
  }
  
  return NextResponse.next()
}
```

---

## Step 1116: Request/Response Manipulation

### 3. Request/Response Manipulation

#### อ่าน Request Headers

```typescript
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  // อ่าน Headers
  const authorization = request.headers.get('authorization')
  const contentType = request.headers.get('content-type')
  const userAgent = request.headers.get('user-agent')
  const referer = request.headers.get('referer')
  
  // อ่าน Cookies
  const token = request.cookies.get('auth-token')?.value
  const language = request.cookies.get('language')?.value
  
  // URL Information
  const { pathname, search, searchParams } = request.nextUrl
  const origin = request.nextUrl.origin
  const host = request.headers.get('host')
  
  // IP Address
  const ip = request.ip || request.headers.get('x-forwarded-for')
  
  console.log({ pathname, token, ip })
  
  return NextResponse.next()
}
```

#### เพิ่ม Request Headers

```typescript
export function middleware(request: NextRequest) {
  // สร้าง Headers ใหม่
  const requestHeaders = new Headers(request.headers)
  
  // เพิ่ม Custom Headers
  requestHeaders.set('x-pathname', request.nextUrl.pathname)
  requestHeaders.set('x-origin', request.nextUrl.origin)
  requestHeaders.set('x-request-id', crypto.randomUUID())
  
  // ส่งต่อ Request พร้อม Headers ใหม่
  return NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  })
}

// การใช้งานใน Route Handler
// app/api/example/route.ts
export function GET(request: Request) {
  const pathname = request.headers.get('x-pathname')
  const requestId = request.headers.get('x-request-id')
  
  return Response.json({ pathname, requestId })
}

// การใช้งานใน Server Component
// app/page.tsx
import { headers } from 'next/headers'

export default function Page() {
  const headersList = headers()
  const pathname = headersList.get('x-pathname')
  const requestId = headersList.get('x-request-id')
  
  return <div>Request ID: {requestId}</div>
}
```

#### เพิ่ม Response Headers

```typescript
export function middleware(request: NextRequest) {
  const response = NextResponse.next()
  
  // Security Headers
  response.headers.set('X-Frame-Options', 'DENY')
  response.headers.set('X-Content-Type-Options', 'nosniff')
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
  response.headers.set(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(self)'
  )
  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-eval'; style-src 'self' 'unsafe-inline'"
  )
  
  // HSTS
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=31536000; includeSubDomains'
    )
  }
  
  return response
}
```

---

## Step 1120: Auth Middleware

### 4. Auth Middleware

#### Basic JWT Authentication

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import { jwtVerify } from 'jose'

const JWT_SECRET = new TextEncoder().encode(process.env.JWT_SECRET!)

async function verifyToken(token: string) {
  try {
    const { payload } = await jwtVerify(token, JWT_SECRET)
    return payload
  } catch {
    return null
  }
}

const PUBLIC_PATHS = [
  '/',
  '/about',
  '/blog',
  '/login',
  '/register',
  '/forgot-password',
]

const PROTECTED_PATHS = ['/dashboard', '/account', '/checkout']
const ADMIN_PATHS = ['/admin']

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // Skip public paths
  if (PUBLIC_PATHS.some(path => pathname === path || pathname.startsWith(path + '/'))) {
    return NextResponse.next()
  }
  
  // Get token from cookie
  const token = request.cookies.get('auth-token')?.value
  
  // Verify token
  const payload = token ? await verifyToken(token) : null
  
  // Protected routes - require auth
  if (PROTECTED_PATHS.some(path => pathname.startsWith(path))) {
    if (!payload) {
      const url = new URL('/login', request.url)
      url.searchParams.set('redirect', pathname)
      return NextResponse.redirect(url)
    }
  }
  
  // Admin routes - require admin role
  if (ADMIN_PATHS.some(path => pathname.startsWith(path))) {
    if (!payload) {
      return NextResponse.redirect(new URL('/login', request.url))
    }
    
    if (payload.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', request.url))
    }
  }
  
  // API routes - check authorization header
  if (pathname.startsWith('/api/') && !pathname.startsWith('/api/public')) {
    const authHeader = request.headers.get('authorization')
    const apiToken = authHeader?.replace('Bearer ', '')
    const apiPayload = apiToken ? await verifyToken(apiToken) : null
    
    if (!apiPayload) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }
    
    // เพิ่ม user info ไปยัง request headers
    const requestHeaders = new Headers(request.headers)
    requestHeaders.set('x-user-id', apiPayload.sub as string)
    requestHeaders.set('x-user-role', apiPayload.role as string)
    
    return NextResponse.next({ request: { headers: requestHeaders } })
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|public).*)'],
}
```

#### NextAuth.js Middleware

```typescript
// middleware.ts (ใช้กับ NextAuth.js v5)
import { auth } from '@/auth'

export default auth((req) => {
  const { nextUrl } = req
  const isLoggedIn = !!req.auth
  
  const isAuthRoute = nextUrl.pathname.startsWith('/auth/')
  const isProtectedRoute = nextUrl.pathname.startsWith('/dashboard') ||
                            nextUrl.pathname.startsWith('/account')
  
  if (isAuthRoute) {
    if (isLoggedIn) {
      return Response.redirect(new URL('/dashboard', nextUrl))
    }
    return null
  }
  
  if (!isLoggedIn && isProtectedRoute) {
    let callbackUrl = nextUrl.pathname
    if (nextUrl.search) {
      callbackUrl += nextUrl.search
    }
    
    const encodedCallbackUrl = encodeURIComponent(callbackUrl)
    return Response.redirect(
      new URL(`/auth/login?callbackUrl=${encodedCallbackUrl}`, nextUrl)
    )
  }
  
  return null
})

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

---

## Step 1124: Redirect/Rewrite

### 5. Redirect/Rewrite

#### Redirect

```typescript
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // Permanent Redirect (301)
  if (pathname === '/old-path') {
    return NextResponse.redirect(new URL('/new-path', request.url), {
      status: 301,
    })
  }
  
  // Temporary Redirect (307)
  if (pathname === '/temp') {
    return NextResponse.redirect(new URL('/temp-target', request.url))
    // default status คือ 307
  }
  
  // Conditional Redirect
  const token = request.cookies.get('token')?.value
  if (pathname.startsWith('/dashboard') && !token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  // Redirect กับ Params
  if (pathname === '/profile') {
    const userId = getUserIdFromToken(token)
    return NextResponse.redirect(
      new URL(`/users/${userId}`, request.url)
    )
  }
  
  return NextResponse.next()
}
```

#### Rewrite (URL ไม่เปลี่ยน แต่ Content เปลี่ยน)

```typescript
export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // A/B Testing
  if (pathname === '/landing') {
    const variant = Math.random() > 0.5 ? 'a' : 'b'
    return NextResponse.rewrite(
      new URL(`/landing-${variant}`, request.url)
    )
  }
  
  // API Proxy (ซ่อน Backend URL)
  if (pathname.startsWith('/api/backend/')) {
    const backendUrl = new URL(
      pathname.replace('/api/backend', ''),
      process.env.BACKEND_URL
    )
    return NextResponse.rewrite(backendUrl)
  }
  
  return NextResponse.next()
}
```

#### Internationalization Redirect

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import Negotiator from 'negotiator'
import { match } from '@formatjs/intl-localematcher'

const LOCALES = ['th', 'en', 'ja']
const DEFAULT_LOCALE = 'th'

function getLocale(request: NextRequest) {
  const acceptLanguage = request.headers.get('accept-language') ?? ''
  const headers = { 'accept-language': acceptLanguage }
  const languages = new Negotiator({ headers }).languages()
  
  return match(languages, LOCALES, DEFAULT_LOCALE)
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // ตรวจสอบว่ามี locale ใน pathname แล้วหรือยัง
  const pathnameHasLocale = LOCALES.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )
  
  if (!pathnameHasLocale) {
    const locale = getLocale(request)
    
    // Redirect ไป locale path
    return NextResponse.redirect(
      new URL(`/${locale}${pathname}`, request.url)
    )
  }
}

export const config = {
  matcher: ['/((?!_next|api|favicon.ico).*)'],
}
```

---

## Step 1128: Geo-based Redirects

### 6. Geo-based Redirects

#### Vercel Geo Headers

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // อ่าน Geo Information (Vercel)
  const country = request.geo?.country || 'US'
  const city = request.geo?.city
  const region = request.geo?.region
  const latitude = request.geo?.latitude
  const longitude = request.geo?.longitude
  
  // Block หรือ Redirect ตาม Country
  const BLOCKED_COUNTRIES = ['XX', 'YY']
  if (BLOCKED_COUNTRIES.includes(country)) {
    return NextResponse.json(
      { error: 'Service not available in your region' },
      { status: 403 }
    )
  }
  
  // Redirect ไปยัง Regional Site
  if (pathname === '/' && !request.cookies.has('country-preference')) {
    if (country === 'TH') {
      return NextResponse.redirect(new URL('/th', request.url))
    }
    if (country === 'JP') {
      return NextResponse.redirect(new URL('/jp', request.url))
    }
  }
  
  // เพิ่ม Country ไปยัง Headers
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-user-country', country)
  if (city) requestHeaders.set('x-user-city', city)
  
  return NextResponse.next({ request: { headers: requestHeaders } })
}
```

#### Currency Redirect

```typescript
const COUNTRY_CURRENCY = {
  TH: 'THB',
  US: 'USD',
  JP: 'JPY',
  GB: 'GBP',
  EU: 'EUR',
} as const

export function middleware(request: NextRequest) {
  const country = request.geo?.country || 'US'
  const currency = COUNTRY_CURRENCY[country as keyof typeof COUNTRY_CURRENCY] || 'USD'
  
  const response = NextResponse.next()
  
  // Set Currency Cookie ถ้ายังไม่มี
  if (!request.cookies.has('currency')) {
    response.cookies.set('currency', currency, {
      maxAge: 60 * 60 * 24 * 365, // 1 year
      path: '/',
    })
  }
  
  return response
}
```

---

## Step 1132: Rate Limiting Middleware

### 7. Rate Limiting Middleware

#### In-Memory Rate Limiting

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

// Simple in-memory rate limiter
// Note: ใน Production ควรใช้ Redis
const rateLimit = new Map<string, { count: number; resetTime: number }>()

function getRateLimitKey(request: NextRequest, prefix: string): string {
  const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown'
  return `${prefix}:${ip}`
}

function checkRateLimit(
  key: string,
  limit: number,
  windowMs: number
): { allowed: boolean; remaining: number; resetTime: number } {
  const now = Date.now()
  const entry = rateLimit.get(key)
  
  if (!entry || now > entry.resetTime) {
    rateLimit.set(key, { count: 1, resetTime: now + windowMs })
    return { allowed: true, remaining: limit - 1, resetTime: now + windowMs }
  }
  
  if (entry.count >= limit) {
    return { allowed: false, remaining: 0, resetTime: entry.resetTime }
  }
  
  entry.count++
  return { allowed: true, remaining: limit - entry.count, resetTime: entry.resetTime }
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // API Rate Limiting (100 requests per minute)
  if (pathname.startsWith('/api/')) {
    const key = getRateLimitKey(request, 'api')
    const result = checkRateLimit(key, 100, 60 * 1000)
    
    if (!result.allowed) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        {
          status: 429,
          headers: {
            'X-RateLimit-Limit': '100',
            'X-RateLimit-Remaining': '0',
            'X-RateLimit-Reset': String(result.resetTime),
            'Retry-After': String(Math.ceil((result.resetTime - Date.now()) / 1000)),
          },
        }
      )
    }
    
    const response = NextResponse.next()
    response.headers.set('X-RateLimit-Limit', '100')
    response.headers.set('X-RateLimit-Remaining', String(result.remaining))
    response.headers.set('X-RateLimit-Reset', String(result.resetTime))
    return response
  }
  
  // Auth Rate Limiting (5 requests per 15 minutes)
  if (pathname === '/api/auth/login') {
    const key = getRateLimitKey(request, 'auth')
    const result = checkRateLimit(key, 5, 15 * 60 * 1000)
    
    if (!result.allowed) {
      return NextResponse.json(
        { error: 'Too many login attempts' },
        { status: 429 }
      )
    }
  }
  
  return NextResponse.next()
}
```

#### Upstash Redis Rate Limiting

```typescript
// middleware.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis/edge'
import { NextRequest, NextResponse } from 'next/server'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
  analytics: true,
  prefix: 'middleware',
})

export async function middleware(request: NextRequest) {
  if (!request.nextUrl.pathname.startsWith('/api/')) {
    return NextResponse.next()
  }
  
  const ip = request.ip ?? '127.0.0.1'
  const { success, pending, limit, reset, remaining } = await ratelimit.limit(ip)
  
  if (!success) {
    return new NextResponse('Too Many Requests', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': new Date(reset).toISOString(),
      },
    })
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: '/api/:path*',
}
```

---

## Step 1136: Logging

### 8. Logging

#### Request Logging

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

function log(
  request: NextRequest,
  response: NextResponse | null,
  duration: number,
  error?: string
) {
  const logData = {
    timestamp: new Date().toISOString(),
    method: request.method,
    url: request.url,
    pathname: request.nextUrl.pathname,
    status: response?.status ?? 500,
    duration: `${duration}ms`,
    ip: request.ip || request.headers.get('x-forwarded-for') || 'unknown',
    userAgent: request.headers.get('user-agent'),
    referer: request.headers.get('referer'),
    country: request.geo?.country,
    error,
  }
  
  // Log ไปยัง Console (development) หรือ Logging Service (production)
  if (process.env.NODE_ENV === 'development') {
    console.log(JSON.stringify(logData))
  } else {
    // ส่งไป Logging Service (Datadog, Logtail, etc.)
    fetch(process.env.LOG_ENDPOINT!, {
      method: 'POST',
      body: JSON.stringify(logData),
      headers: { 'Content-Type': 'application/json' },
    }).catch(console.error)
  }
}

export function middleware(request: NextRequest) {
  const start = Date.now()
  
  try {
    const response = NextResponse.next()
    const duration = Date.now() - start
    
    // Log async (ไม่ block response)
    // ใช้ waitUntil ถ้า runtime รองรับ
    log(request, response, duration)
    
    return response
  } catch (error) {
    const duration = Date.now() - start
    log(request, null, duration, String(error))
    throw error
  }
}
```

#### Structured Logging

```typescript
// lib/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error'

interface LogEntry {
  level: LogLevel
  message: string
  timestamp: string
  context?: Record<string, unknown>
}

export function createLogger(context?: Record<string, unknown>) {
  function log(level: LogLevel, message: string, extra?: Record<string, unknown>) {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context: { ...context, ...extra },
    }
    
    if (process.env.NODE_ENV === 'development') {
      const color = {
        debug: '\x1b[36m',
        info: '\x1b[32m',
        warn: '\x1b[33m',
        error: '\x1b[31m',
      }[level]
      
      console.log(
        `${color}[${entry.level.toUpperCase()}]\x1b[0m ${entry.message}`,
        entry.context
      )
    } else {
      // Production: ส่งไป Logging Service
      console.log(JSON.stringify(entry))
    }
  }
  
  return {
    debug: (msg: string, ctx?: Record<string, unknown>) => log('debug', msg, ctx),
    info: (msg: string, ctx?: Record<string, unknown>) => log('info', msg, ctx),
    warn: (msg: string, ctx?: Record<string, unknown>) => log('warn', msg, ctx),
    error: (msg: string, ctx?: Record<string, unknown>) => log('error', msg, ctx),
  }
}

// middleware.ts
import { createLogger } from '@/lib/logger'

const logger = createLogger({ service: 'middleware' })

export function middleware(request: NextRequest) {
  const start = Date.now()
  const requestId = crypto.randomUUID()
  
  const requestLogger = createLogger({
    requestId,
    method: request.method,
    pathname: request.nextUrl.pathname,
    ip: request.ip,
  })
  
  requestLogger.info('Request started')
  
  const response = NextResponse.next()
  
  // เพิ่ม Request ID ไป Response
  response.headers.set('x-request-id', requestId)
  
  requestLogger.info('Request completed', {
    status: response.status,
    duration: Date.now() - start,
  })
  
  return response
}
```

---

## Step 1140: ตัวอย่างสมบูรณ์

### ตัวอย่างสมบูรณ์: Production Middleware

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import { jwtVerify } from 'jose'

// =========================================
// Configuration
// =========================================
const JWT_SECRET = new TextEncoder().encode(process.env.JWT_SECRET!)

const PUBLIC_ROUTES = [
  '/',
  '/about',
  '/contact',
  '/blog',
  '/products',
  '/login',
  '/register',
  '/forgot-password',
  '/reset-password',
  '/api/auth',
  '/api/public',
]

const PROTECTED_ROUTES = ['/dashboard', '/account', '/orders', '/checkout']
const ADMIN_ROUTES = ['/admin']
const API_ROUTES = ['/api/']

// =========================================
// Utility Functions
// =========================================
async function verifyJWT(token: string) {
  try {
    const { payload } = await jwtVerify(token, JWT_SECRET)
    return payload
  } catch {
    return null
  }
}

function isPublicRoute(pathname: string) {
  return PUBLIC_ROUTES.some(
    route => pathname === route || pathname.startsWith(route + '/')
  )
}

// =========================================
// Middleware Functions
// =========================================

async function handleAuth(request: NextRequest) {
  const { pathname } = request.nextUrl
  const token = request.cookies.get('auth-token')?.value
  const payload = token ? await verifyJWT(token) : null
  
  // Protected Routes
  if (PROTECTED_ROUTES.some(r => pathname.startsWith(r))) {
    if (!payload) {
      const url = new URL('/login', request.url)
      url.searchParams.set('redirect', pathname)
      return NextResponse.redirect(url)
    }
  }
  
  // Admin Routes
  if (ADMIN_ROUTES.some(r => pathname.startsWith(r))) {
    if (!payload) {
      return NextResponse.redirect(new URL('/login', request.url))
    }
    if (payload.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', request.url))
    }
  }
  
  // Add user info to headers
  if (payload) {
    const requestHeaders = new Headers(request.headers)
    requestHeaders.set('x-user-id', payload.sub as string)
    requestHeaders.set('x-user-role', payload.role as string)
    return NextResponse.next({ request: { headers: requestHeaders } })
  }
  
  return null
}

function handleSecurity(request: NextRequest, response: NextResponse) {
  // Security Headers
  response.headers.set('X-Frame-Options', 'SAMEORIGIN')
  response.headers.set('X-Content-Type-Options', 'nosniff')
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
  response.headers.set('X-XSS-Protection', '1; mode=block')
  
  return response
}

function handleCORS(request: NextRequest) {
  const origin = request.headers.get('origin')
  const allowedOrigins = [
    'https://myapp.com',
    process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '',
  ].filter(Boolean)
  
  if (request.method === 'OPTIONS') {
    const response = new NextResponse(null, { status: 200 })
    if (origin && allowedOrigins.includes(origin)) {
      response.headers.set('Access-Control-Allow-Origin', origin)
      response.headers.set(
        'Access-Control-Allow-Methods',
        'GET, POST, PUT, PATCH, DELETE, OPTIONS'
      )
      response.headers.set(
        'Access-Control-Allow-Headers',
        'Content-Type, Authorization'
      )
    }
    return response
  }
  
  return null
}

// =========================================
// Main Middleware
// =========================================
export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  const start = Date.now()
  const requestId = crypto.randomUUID()
  
  // Skip static files
  if (
    pathname.startsWith('/_next') ||
    pathname.startsWith('/favicon') ||
    pathname.includes('.')
  ) {
    return NextResponse.next()
  }
  
  // CORS Preflight
  const corsResponse = handleCORS(request)
  if (corsResponse) return corsResponse
  
  // Skip auth for public routes
  if (!isPublicRoute(pathname)) {
    const authResponse = await handleAuth(request)
    if (authResponse && authResponse.status >= 300) {
      return authResponse
    }
  }
  
  // Create response
  const response = NextResponse.next()
  
  // Apply security headers
  handleSecurity(request, response)
  
  // Add tracking headers
  response.headers.set('x-request-id', requestId)
  response.headers.set('x-response-time', `${Date.now() - start}ms`)
  
  return response
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

---

## Step 1142: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. ใช้ Matcher เพื่อจำกัด Middleware

✓ ระบุ paths ที่ต้องการเสมอ
✓ ยกเว้น static files (_next, public)
✓ Middleware ที่รัน path มาก = ช้าลง

## 2. Edge Runtime

✓ Middleware รันที่ Edge ไม่มี Node.js APIs ทุกอย่าง
✓ ใช้ 'jose' แทน 'jsonwebtoken' สำหรับ JWT
✓ ไม่มี fs module หรือ database connections โดยตรง

## 3. Performance

✓ Middleware ต้องเร็ว (< 25ms)
✓ หลีกเลี่ยง Complex Database Queries
✓ Cache JWT Verification ถ้าเป็นไปได้
✓ ใช้ Redis สำหรับ Rate Limiting ใน Production

## 4. Security

✓ Validate JWT ทุกครั้ง
✓ Set Security Headers ใน Middleware
✓ ไม่เก็บ Sensitive Data ใน Cookie โดยไม่ Encrypt

## 5. Error Handling

✓ Try-catch ใน async Middleware
✓ Log Errors อย่างเหมาะสม
✓ Return Response ที่มีความหมาย
```

---

## Quiz

### แบบทดสอบ Part 38

**คำถามที่ 1:** Middleware ใน Next.js รันที่ไหน?
- A) Browser
- B) Server (Node.js)
- C) Edge Runtime ✓
- D) Database

**คำถามที่ 2:** ไฟล์ middleware.ts ต้องอยู่ที่ไหน?
- A) ใน app/ folder
- B) ที่ Root ของ Project หรือใน src/ ✓
- C) ใน api/ folder
- D) ที่ไหนก็ได้

**คำถามที่ 3:** `NextResponse.rewrite()` แตกต่างจาก `NextResponse.redirect()` อย่างไร?
- A) ไม่ต่างกัน
- B) rewrite เปลี่ยน URL ที่เห็นใน Browser, redirect ไม่เปลี่ยน
- C) rewrite เปลี่ยน Content โดยไม่เปลี่ยน URL ที่ Browser เห็น ✓
- D) redirect เร็วกว่า rewrite

**คำถามที่ 4:** ทำไม Middleware ถึงใช้ 'jose' แทน 'jsonwebtoken'?
- A) 'jose' ฟรีกว่า
- B) Middleware รันที่ Edge Runtime ที่ไม่มี Node.js APIs ครบ, 'jose' รองรับ Web APIs ✓
- C) 'jose' เร็วกว่า
- D) 'jsonwebtoken' ถูก Deprecated

**คำถามที่ 5:** config.matcher ในไฟล์ middleware.ts ทำอะไร?
- A) กำหนด Routes สำหรับ API
- B) กำหนด Routes ที่ Middleware จะทำงาน ✓
- C) กำหนด Static Routes
- D) กำหนด Auth Routes

---

## สรุป Part 38

ใน Part นี้เราได้เรียนรู้:

1. **Middleware** - Code ที่รันก่อน Request ถึง Route
2. **middleware.ts** - ตำแหน่งและโครงสร้าง
3. **Request/Response** - อ่านและแก้ไข Headers, Cookies
4. **Auth Middleware** - JWT Verification, Role-based Access
5. **Redirect/Rewrite** - URL Manipulation
6. **Geo-based** - Country Detection, Currency
7. **Rate Limiting** - ป้องกัน Abuse
8. **Logging** - Structured Logs

---

➡️ **Part ถัดไป:** [Part 39: Next.js Authentication](./part-39-nextjs-authentication.md)
