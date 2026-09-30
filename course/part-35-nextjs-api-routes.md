# Part 35: Next.js Route Handlers (API Routes)

## ข้อมูล Part
- **Steps:** 1001-1035
- **ระดับ:** Intermediate to Advanced
- **เวลาเรียน:** 3-4 ชั่วโมง
- **Prerequisites:** Part 34 (Data Fetching)

---

## สารบัญ

1. [Route Handlers (app/api)](#1-route-handlers-appapi)
2. [GET, POST, PUT, DELETE, PATCH](#2-get-post-put-delete-patch)
3. [Request, Response Objects](#3-request-response-objects)
4. [Middleware](#4-middleware)
5. [CORS Configuration](#5-cors-configuration)
6. [API Error Handling](#6-api-error-handling)
7. [Rate Limiting](#7-rate-limiting)
8. [ตัวอย่าง CRUD API](#8-ตัวอย่าง-crud-api)
9. [Quiz](#quiz)

---

## Step 1001: Route Handlers Overview

### 1. Route Handlers (app/api)

Route Handlers คือ API Endpoints ใน App Router ที่ใช้ Web Standard API (Request/Response)

#### โครงสร้างพื้นฐาน

```
app/
└── api/
    ├── users/
    │   ├── route.ts           → GET /api/users, POST /api/users
    │   └── [id]/
    │       └── route.ts       → GET /api/users/:id, PUT, DELETE
    ├── posts/
    │   ├── route.ts
    │   └── [id]/
    │       ├── route.ts
    │       └── comments/
    │           └── route.ts   → GET /api/posts/:id/comments
    └── auth/
        ├── login/
        │   └── route.ts
        └── logout/
            └── route.ts
```

#### HTTP Methods ที่รองรับ

```typescript
// app/api/example/route.ts
export async function GET(request: Request) { }
export async function POST(request: Request) { }
export async function PUT(request: Request) { }
export async function PATCH(request: Request) { }
export async function DELETE(request: Request) { }
export async function HEAD(request: Request) { }
export async function OPTIONS(request: Request) { }
```

---

## Step 1003: GET, POST, PUT, DELETE, PATCH

### 2. GET, POST, PUT, DELETE, PATCH

#### GET - ดึงข้อมูล

```typescript
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  try {
    // อ่าน Query Parameters
    const searchParams = request.nextUrl.searchParams
    const page = Number(searchParams.get('page')) || 1
    const limit = Number(searchParams.get('limit')) || 10
    const category = searchParams.get('category')
    const search = searchParams.get('search')
    
    // Query Database
    const where: any = {}
    if (category) where.categoryId = category
    if (search) where.name = { contains: search, mode: 'insensitive' }
    
    const [products, total] = await Promise.all([
      prisma.product.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        include: { category: true },
        orderBy: { createdAt: 'desc' },
      }),
      prisma.product.count({ where }),
    ])
    
    return NextResponse.json({
      data: products,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      }
    })
  } catch (error) {
    console.error('GET /api/products error:', error)
    return NextResponse.json(
      { error: 'Internal Server Error' },
      { status: 500 }
    )
  }
}
```

#### POST - สร้างข้อมูล

```typescript
// app/api/products/route.ts
import { z } from 'zod'

const CreateProductSchema = z.object({
  name: z.string().min(1).max(255),
  description: z.string().min(1),
  price: z.number().positive(),
  categoryId: z.string().cuid(),
  stock: z.number().int().nonnegative().default(0),
})

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    
    // Validate Input
    const result = CreateProductSchema.safeParse(body)
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.errors },
        { status: 400 }
      )
    }
    
    const data = result.data
    
    // Check if Category exists
    const category = await prisma.category.findUnique({
      where: { id: data.categoryId }
    })
    
    if (!category) {
      return NextResponse.json(
        { error: 'Category not found' },
        { status: 400 }
      )
    }
    
    // Create Product
    const product = await prisma.product.create({
      data,
      include: { category: true }
    })
    
    return NextResponse.json(product, { status: 201 })
  } catch (error) {
    if (error instanceof SyntaxError) {
      return NextResponse.json(
        { error: 'Invalid JSON' },
        { status: 400 }
      )
    }
    throw error
  }
}
```

#### PUT - อัพเดทข้อมูลทั้งหมด

```typescript
// app/api/products/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

interface RouteContext {
  params: { id: string }
}

const UpdateProductSchema = z.object({
  name: z.string().min(1).max(255),
  description: z.string().min(1),
  price: z.number().positive(),
  categoryId: z.string().cuid(),
  stock: z.number().int().nonnegative(),
})

export async function PUT(
  request: NextRequest,
  { params }: RouteContext
) {
  try {
    const body = await request.json()
    const result = UpdateProductSchema.safeParse(body)
    
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.errors },
        { status: 400 }
      )
    }
    
    const product = await prisma.product.update({
      where: { id: params.id },
      data: result.data,
      include: { category: true }
    })
    
    return NextResponse.json(product)
  } catch (error: any) {
    if (error.code === 'P2025') {
      // Prisma: Record not found
      return NextResponse.json(
        { error: 'Product not found' },
        { status: 404 }
      )
    }
    throw error
  }
}
```

#### PATCH - อัพเดทข้อมูลบางส่วน

```typescript
// app/api/products/[id]/route.ts
const PatchProductSchema = z.object({
  name: z.string().min(1).max(255).optional(),
  description: z.string().min(1).optional(),
  price: z.number().positive().optional(),
  stock: z.number().int().nonnegative().optional(),
})

export async function PATCH(
  request: NextRequest,
  { params }: RouteContext
) {
  try {
    const body = await request.json()
    const result = PatchProductSchema.safeParse(body)
    
    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.errors },
        { status: 400 }
      )
    }
    
    // ลบ undefined values
    const updateData = Object.fromEntries(
      Object.entries(result.data).filter(([_, v]) => v !== undefined)
    )
    
    if (Object.keys(updateData).length === 0) {
      return NextResponse.json(
        { error: 'No fields to update' },
        { status: 400 }
      )
    }
    
    const product = await prisma.product.update({
      where: { id: params.id },
      data: updateData,
    })
    
    return NextResponse.json(product)
  } catch (error: any) {
    if (error.code === 'P2025') {
      return NextResponse.json(
        { error: 'Product not found' },
        { status: 404 }
      )
    }
    throw error
  }
}
```

#### DELETE - ลบข้อมูล

```typescript
// app/api/products/[id]/route.ts
export async function DELETE(
  request: NextRequest,
  { params }: RouteContext
) {
  try {
    // Soft Delete หรือ Hard Delete?
    const searchParams = request.nextUrl.searchParams
    const soft = searchParams.get('soft') === 'true'
    
    if (soft) {
      // Soft Delete
      await prisma.product.update({
        where: { id: params.id },
        data: { deletedAt: new Date() }
      })
    } else {
      // Hard Delete
      await prisma.product.delete({
        where: { id: params.id }
      })
    }
    
    return new NextResponse(null, { status: 204 })
  } catch (error: any) {
    if (error.code === 'P2025') {
      return NextResponse.json(
        { error: 'Product not found' },
        { status: 404 }
      )
    }
    throw error
  }
}
```

---

## Step 1007: Request, Response Objects

### 3. Request, Response Objects

#### NextRequest

```typescript
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  // URL และ Search Params
  const url = request.url                          // Full URL
  const pathname = request.nextUrl.pathname        // Path
  const searchParams = request.nextUrl.searchParams
  const id = searchParams.get('id')
  
  // Headers
  const authorization = request.headers.get('authorization')
  const contentType = request.headers.get('content-type')
  const userAgent = request.headers.get('user-agent')
  
  // Cookies
  const sessionToken = request.cookies.get('session')?.value
  
  // Body
  const json = await request.json()       // JSON body
  const text = await request.text()       // Text body
  const formData = await request.formData() // Form data
  const blob = await request.blob()       // Binary data
  
  // Geo (Vercel)
  const country = request.geo?.country
  const city = request.geo?.city
  
  // IP
  const ip = request.ip
}
```

#### NextResponse

```typescript
import { NextResponse } from 'next/server'

// JSON Response
return NextResponse.json({ data: 'hello' })
return NextResponse.json({ error: 'Not Found' }, { status: 404 })

// Redirect
return NextResponse.redirect(new URL('/login', request.url))

// Rewrite
return NextResponse.rewrite(new URL('/api/v2/users', request.url))

// Custom Status
return new NextResponse(null, { status: 204 })
return new NextResponse('Hello', { status: 200, headers: { 'Content-Type': 'text/plain' } })

// Set Headers
const response = NextResponse.json({ data: 'hello' })
response.headers.set('X-Custom-Header', 'value')
response.headers.set('Cache-Control', 'public, max-age=3600')

// Set Cookies
response.cookies.set('token', 'abc123', {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax',
  maxAge: 60 * 60 * 24 * 30, // 30 days
  path: '/',
})

// Delete Cookies
response.cookies.delete('token')

return response
```

---

## Step 1010: Middleware

### 4. Middleware

Middleware รันก่อนที่ Request จะถึง Route Handler หรือ Page

#### middleware.ts

```typescript
// middleware.ts (ระดับ Root)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  
  // Authentication Check
  const token = request.cookies.get('token')?.value
  
  const isAuthPage = pathname.startsWith('/login') || 
                     pathname.startsWith('/register')
  const isProtectedPage = pathname.startsWith('/dashboard') ||
                           pathname.startsWith('/admin')
  
  // Redirect to login if not authenticated
  if (isProtectedPage && !token) {
    const loginUrl = new URL('/login', request.url)
    loginUrl.searchParams.set('redirect', pathname)
    return NextResponse.redirect(loginUrl)
  }
  
  // Redirect to dashboard if already logged in
  if (isAuthPage && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }
  
  return NextResponse.next()
}

// กำหนด Routes ที่ Middleware ทำงาน
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/login',
    '/register',
  ],
  // หรือใช้ regex
  // matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

#### Advanced Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { jwtVerify } from 'jose'

const JWT_SECRET = new TextEncoder().encode(
  process.env.JWT_SECRET || 'fallback-secret'
)

async function verifyToken(token: string) {
  try {
    const { payload } = await jwtVerify(token, JWT_SECRET)
    return payload
  } catch {
    return null
  }
}

export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  
  // 1. Static Files - Skip
  if (
    pathname.startsWith('/_next') ||
    pathname.startsWith('/api/public') ||
    pathname.includes('.')
  ) {
    return NextResponse.next()
  }
  
  // 2. Auth Check
  const token = request.cookies.get('auth-token')?.value
  const payload = token ? await verifyToken(token) : null
  
  // 3. Protected Routes
  if (pathname.startsWith('/dashboard') || pathname.startsWith('/admin')) {
    if (!payload) {
      return NextResponse.redirect(new URL('/login', request.url))
    }
    
    // Admin Only
    if (pathname.startsWith('/admin') && payload.role !== 'admin') {
      return NextResponse.redirect(new URL('/403', request.url))
    }
    
    // Add user info to headers
    const requestHeaders = new Headers(request.headers)
    requestHeaders.set('x-user-id', payload.sub as string)
    requestHeaders.set('x-user-role', payload.role as string)
    
    return NextResponse.next({ request: { headers: requestHeaders } })
  }
  
  // 4. API Auth
  if (pathname.startsWith('/api/') && !pathname.startsWith('/api/public')) {
    const authHeader = request.headers.get('authorization')
    const apiToken = authHeader?.replace('Bearer ', '')
    const apiPayload = apiToken ? await verifyToken(apiToken) : null
    
    if (!apiPayload) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
}
```

---

## Step 1013: CORS Configuration

### 5. CORS Configuration

#### CORS Middleware

```typescript
// lib/cors.ts
import { NextRequest, NextResponse } from 'next/server'

const ALLOWED_ORIGINS = [
  'https://myapp.com',
  'https://www.myapp.com',
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '',
].filter(Boolean)

export function cors(request: NextRequest, response: NextResponse) {
  const origin = request.headers.get('origin')
  
  if (origin && ALLOWED_ORIGINS.includes(origin)) {
    response.headers.set('Access-Control-Allow-Origin', origin)
  }
  
  response.headers.set(
    'Access-Control-Allow-Methods',
    'GET, POST, PUT, PATCH, DELETE, OPTIONS'
  )
  response.headers.set(
    'Access-Control-Allow-Headers',
    'Content-Type, Authorization, X-Requested-With'
  )
  response.headers.set('Access-Control-Max-Age', '86400')
  
  return response
}

// app/api/public/route.ts
import { cors } from '@/lib/cors'

export async function OPTIONS(request: NextRequest) {
  const response = new NextResponse(null, { status: 200 })
  return cors(request, response)
}

export async function GET(request: NextRequest) {
  const data = await getPublicData()
  const response = NextResponse.json(data)
  return cors(request, response)
}
```

#### CORS ใน Middleware

```typescript
// middleware.ts
export function middleware(request: NextRequest) {
  // Handle CORS Preflight
  if (request.method === 'OPTIONS') {
    return new NextResponse(null, {
      status: 200,
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization',
      },
    })
  }
  
  const response = NextResponse.next()
  
  // Add CORS headers
  if (request.nextUrl.pathname.startsWith('/api/public')) {
    response.headers.set('Access-Control-Allow-Origin', '*')
  }
  
  return response
}
```

---

## Step 1016: API Error Handling

### 6. API Error Handling

#### Custom Error Classes

```typescript
// lib/errors.ts
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND')
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED')
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403, 'FORBIDDEN')
  }
}

export class ValidationError extends AppError {
  constructor(
    public details: any[],
    message = 'Validation failed'
  ) {
    super(message, 400, 'VALIDATION_ERROR')
  }
}
```

#### Error Handler Wrapper

```typescript
// lib/api-handler.ts
import { NextRequest, NextResponse } from 'next/server'
import { AppError, ValidationError } from './errors'
import { ZodError } from 'zod'

type Handler = (request: NextRequest, context?: any) => Promise<NextResponse>

export function withErrorHandler(handler: Handler): Handler {
  return async (request: NextRequest, context?: any) => {
    try {
      return await handler(request, context)
    } catch (error) {
      // Handle AppError
      if (error instanceof AppError) {
        const body: any = { error: error.message, code: error.code }
        if (error instanceof ValidationError) {
          body.details = error.details
        }
        return NextResponse.json(body, { status: error.statusCode })
      }
      
      // Handle Zod Validation Error
      if (error instanceof ZodError) {
        return NextResponse.json(
          {
            error: 'Validation failed',
            code: 'VALIDATION_ERROR',
            details: error.errors,
          },
          { status: 400 }
        )
      }
      
      // Handle Prisma Errors
      if ((error as any).code) {
        const prismaError = error as any
        if (prismaError.code === 'P2025') {
          return NextResponse.json(
            { error: 'Resource not found', code: 'NOT_FOUND' },
            { status: 404 }
          )
        }
        if (prismaError.code === 'P2002') {
          return NextResponse.json(
            { error: 'Resource already exists', code: 'CONFLICT' },
            { status: 409 }
          )
        }
      }
      
      // Log Unknown Errors
      console.error('Unhandled API Error:', error)
      
      return NextResponse.json(
        { error: 'Internal Server Error', code: 'INTERNAL_ERROR' },
        { status: 500 }
      )
    }
  }
}

// การใช้งาน
// app/api/users/route.ts
export const GET = withErrorHandler(async (request: NextRequest) => {
  const users = await getUsers()
  return NextResponse.json(users)
})

export const POST = withErrorHandler(async (request: NextRequest) => {
  const body = await request.json()
  // Validation, Creation, etc.
  return NextResponse.json(newUser, { status: 201 })
})
```

#### Global Error Response Format

```typescript
// lib/api-response.ts
interface ApiResponse<T> {
  data?: T
  error?: string
  code?: string
  details?: any[]
  pagination?: {
    page: number
    limit: number
    total: number
    totalPages: number
  }
}

export function successResponse<T>(data: T, status = 200) {
  return NextResponse.json<ApiResponse<T>>({ data }, { status })
}

export function errorResponse(
  error: string,
  status: number,
  code?: string,
  details?: any[]
) {
  return NextResponse.json<ApiResponse<never>>(
    { error, code, details },
    { status }
  )
}

export function paginatedResponse<T>(
  data: T[],
  pagination: { page: number; limit: number; total: number }
) {
  return NextResponse.json<ApiResponse<T[]>>({
    data,
    pagination: {
      ...pagination,
      totalPages: Math.ceil(pagination.total / pagination.limit),
    },
  })
}
```

---

## Step 1020: Rate Limiting

### 7. Rate Limiting

#### Simple In-Memory Rate Limiter

```typescript
// lib/rate-limit.ts
interface RateLimitConfig {
  windowMs: number  // Time window in milliseconds
  max: number       // Max requests per window
}

class RateLimiter {
  private requests = new Map<string, number[]>()
  
  check(identifier: string, config: RateLimitConfig): {
    success: boolean
    remaining: number
    resetTime: number
  } {
    const now = Date.now()
    const windowStart = now - config.windowMs
    
    const timestamps = this.requests.get(identifier) || []
    const validTimestamps = timestamps.filter(t => t > windowStart)
    
    if (validTimestamps.length >= config.max) {
      const oldestRequest = Math.min(...validTimestamps)
      return {
        success: false,
        remaining: 0,
        resetTime: oldestRequest + config.windowMs,
      }
    }
    
    validTimestamps.push(now)
    this.requests.set(identifier, validTimestamps)
    
    return {
      success: true,
      remaining: config.max - validTimestamps.length,
      resetTime: now + config.windowMs,
    }
  }
}

const limiter = new RateLimiter()

export function rateLimit(config: RateLimitConfig) {
  return (request: NextRequest): NextResponse | null => {
    const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown'
    const result = limiter.check(ip, config)
    
    if (!result.success) {
      return NextResponse.json(
        { error: 'Too many requests', retryAfter: result.resetTime },
        {
          status: 429,
          headers: {
            'X-RateLimit-Limit': String(config.max),
            'X-RateLimit-Remaining': '0',
            'X-RateLimit-Reset': String(result.resetTime),
            'Retry-After': String(Math.ceil((result.resetTime - Date.now()) / 1000)),
          },
        }
      )
    }
    
    return null
  }
}

// การใช้งาน
// app/api/auth/login/route.ts
const loginRateLimit = rateLimit({ windowMs: 15 * 60 * 1000, max: 5 })

export async function POST(request: NextRequest) {
  // Check Rate Limit
  const rateLimitResponse = loginRateLimit(request)
  if (rateLimitResponse) return rateLimitResponse
  
  // Process Login
  const body = await request.json()
  // ...
}
```

#### Rate Limiting ด้วย Upstash Redis

```typescript
// lib/rate-limit-redis.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
})

export const apiRateLimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '10 s'),
  analytics: true,
})

export const authRateLimit = new Ratelimit({
  redis,
  limiter: Ratelimit.fixedWindow(5, '15 m'),
})

// การใช้งาน
export async function POST(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1'
  const { success, remaining, reset } = await authRateLimit.limit(ip)
  
  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      {
        status: 429,
        headers: {
          'X-RateLimit-Remaining': String(remaining),
          'X-RateLimit-Reset': String(reset),
        },
      }
    )
  }
  
  // Process request
}
```

---

## Step 1025: ตัวอย่าง CRUD API

### 8. ตัวอย่าง CRUD API

#### สมบูรณ์: Blog Posts CRUD API

```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { prisma } from '@/lib/prisma'
import { withErrorHandler } from '@/lib/api-handler'
import { getAuthUser } from '@/lib/auth'

const CreatePostSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1),
  excerpt: z.string().max(500).optional(),
  categoryId: z.string().cuid(),
  published: z.boolean().default(false),
  tags: z.array(z.string()).optional(),
})

// GET /api/posts
export const GET = withErrorHandler(async (request: NextRequest) => {
  const sp = request.nextUrl.searchParams
  const page = Math.max(1, Number(sp.get('page')) || 1)
  const limit = Math.min(50, Math.max(1, Number(sp.get('limit')) || 10))
  const search = sp.get('search')
  const category = sp.get('category')
  const published = sp.get('published')
  
  const where: any = {}
  if (search) {
    where.OR = [
      { title: { contains: search, mode: 'insensitive' } },
      { content: { contains: search, mode: 'insensitive' } },
    ]
  }
  if (category) where.categoryId = category
  if (published !== null) where.published = published === 'true'
  
  const [posts, total] = await Promise.all([
    prisma.post.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      include: {
        author: { select: { id: true, name: true, avatar: true } },
        category: true,
        _count: { select: { comments: true } },
      },
      orderBy: { createdAt: 'desc' },
    }),
    prisma.post.count({ where }),
  ])
  
  return NextResponse.json({
    data: posts,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  })
})

// POST /api/posts
export const POST = withErrorHandler(async (request: NextRequest) => {
  const user = await getAuthUser(request)
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const body = await request.json()
  const data = CreatePostSchema.parse(body)
  
  const slug = data.title
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/(^-|-$)/g, '')
  
  const post = await prisma.post.create({
    data: {
      ...data,
      slug,
      authorId: user.id,
      tags: data.tags ? {
        connectOrCreate: data.tags.map(tag => ({
          where: { name: tag },
          create: { name: tag, slug: tag.toLowerCase().replace(/\s+/g, '-') },
        })),
      } : undefined,
    },
    include: {
      author: { select: { id: true, name: true } },
      category: true,
      tags: true,
    },
  })
  
  return NextResponse.json(post, { status: 201 })
})
```

```typescript
// app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { prisma } from '@/lib/prisma'
import { withErrorHandler } from '@/lib/api-handler'
import { getAuthUser } from '@/lib/auth'

interface Context {
  params: { id: string }
}

// GET /api/posts/:id
export const GET = withErrorHandler(async (
  request: NextRequest,
  { params }: Context
) => {
  const post = await prisma.post.findUnique({
    where: { id: params.id },
    include: {
      author: { select: { id: true, name: true, avatar: true, bio: true } },
      category: true,
      tags: true,
      comments: {
        include: {
          author: { select: { id: true, name: true, avatar: true } }
        },
        orderBy: { createdAt: 'desc' },
        take: 10,
      },
      _count: { select: { comments: true } },
    },
  })
  
  if (!post) {
    return NextResponse.json({ error: 'Post not found' }, { status: 404 })
  }
  
  // Increment View Count
  await prisma.post.update({
    where: { id: params.id },
    data: { viewCount: { increment: 1 } },
  })
  
  return NextResponse.json(post)
})

const UpdatePostSchema = z.object({
  title: z.string().min(1).max(255).optional(),
  content: z.string().min(1).optional(),
  excerpt: z.string().max(500).optional(),
  published: z.boolean().optional(),
  categoryId: z.string().cuid().optional(),
  tags: z.array(z.string()).optional(),
})

// PATCH /api/posts/:id
export const PATCH = withErrorHandler(async (
  request: NextRequest,
  { params }: Context
) => {
  const user = await getAuthUser(request)
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const post = await prisma.post.findUnique({ where: { id: params.id } })
  if (!post) {
    return NextResponse.json({ error: 'Post not found' }, { status: 404 })
  }
  
  if (post.authorId !== user.id && user.role !== 'admin') {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
  }
  
  const body = await request.json()
  const data = UpdatePostSchema.parse(body)
  
  const updatedPost = await prisma.post.update({
    where: { id: params.id },
    data: {
      ...data,
      tags: data.tags ? {
        set: [],
        connectOrCreate: data.tags.map(tag => ({
          where: { name: tag },
          create: { name: tag, slug: tag.toLowerCase().replace(/\s+/g, '-') },
        })),
      } : undefined,
    },
    include: {
      author: { select: { id: true, name: true } },
      category: true,
      tags: true,
    },
  })
  
  return NextResponse.json(updatedPost)
})

// DELETE /api/posts/:id
export const DELETE = withErrorHandler(async (
  request: NextRequest,
  { params }: Context
) => {
  const user = await getAuthUser(request)
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const post = await prisma.post.findUnique({ where: { id: params.id } })
  if (!post) {
    return NextResponse.json({ error: 'Post not found' }, { status: 404 })
  }
  
  if (post.authorId !== user.id && user.role !== 'admin') {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
  }
  
  await prisma.post.delete({ where: { id: params.id } })
  
  return new NextResponse(null, { status: 204 })
})
```

---

## Step 1030: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Validation ด้วย Zod

✓ Validate Input ทุกครั้งก่อน Process
✓ ใช้ safeParse() แทน parse() เพื่อ Handle Errors
✓ Define Schema แยกออกมาเพื่อ Reuse

## 2. Error Handling

✓ ใช้ Wrapper Function สำหรับ Error Handling
✓ Return Error ที่มีความหมาย (code, details)
✓ Log Server Errors ไปยัง Monitoring

## 3. Authentication

✓ ตรวจสอบ Auth ใน Route Handler
✓ ใช้ Middleware สำหรับ Global Auth
✓ ส่งคืน 401 สำหรับ Unauthenticated, 403 สำหรับ Unauthorized

## 4. Rate Limiting

✓ ใช้ Rate Limiting สำหรับ Public APIs
✓ ตั้ง Limit เข้มงวดขึ้นสำหรับ Auth Endpoints
✓ ใช้ Redis สำหรับ Distributed Rate Limiting

## 5. Security

✓ ใช้ HTTPS เสมอ
✓ Validate Content-Type
✓ Sanitize Input ก่อน Query Database
✓ ใช้ Parameterized Queries (Prisma ทำให้อัตโนมัติ)
```

---

## Quiz

### แบบทดสอบ Part 35

**คำถามที่ 1:** Route Handler ใน App Router ต้องอยู่ในไฟล์ชื่ออะไร?
- A) handler.ts
- B) index.ts
- C) route.ts ✓
- D) api.ts

**คำถามที่ 2:** HTTP Status Code ที่เหมาะสมสำหรับการสร้าง Resource ใหม่สำเร็จ?
- A) 200
- B) 201 ✓
- C) 204
- D) 202

**คำถามที่ 3:** `request.nextUrl.searchParams` ใช้ทำอะไร?
- A) อ่าน Request Body
- B) อ่าน Query Parameters จาก URL ✓
- C) อ่าน Request Headers
- D) อ่าน Cookies

**คำถามที่ 4:** เมื่อลบ Resource สำเร็จ ควรส่ง Status Code อะไร?
- A) 200
- B) 201
- C) 204 ✓
- D) 302

**คำถามที่ 5:** Middleware ใน Next.js รันที่ไหน?
- A) Browser
- B) Server ก่อน Request ถึง Route Handler ✓
- C) Database
- D) CDN

---

## สรุป Part 35

ใน Part นี้เราได้เรียนรู้:

1. **Route Handlers** - API Endpoints ด้วย Web Standard API
2. **HTTP Methods** - GET, POST, PUT, PATCH, DELETE
3. **Request/Response** - NextRequest, NextResponse
4. **Middleware** - Authentication, Logging, CORS
5. **Error Handling** - Custom Errors, Wrapper Functions
6. **Rate Limiting** - ป้องกัน Abuse
7. **CRUD API** - ตัวอย่างสมบูรณ์

---

➡️ **Part ถัดไป:** [Part 36: Server Components](./part-36-nextjs-server-components.md)
