# Part 53: Next.js Edge Runtime

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1721-1760  
> **เวลาเรียน:** ~3 ชั่วโมง

---

## 📚 Table of Contents

1. [Edge Runtime คืออะไร](#edge-runtime-คืออะไร)
2. [Edge Functions](#edge-functions)
3. [Edge Middleware](#edge-middleware)
4. [Vercel Edge Network](#vercel-edge-network)
5. [Performance Benefits](#performance-benefits)
6. [Limitations](#limitations)
7. [Edge Config](#edge-config)
8. [Quiz](#quiz)

---

## Step 1721: Edge Runtime คืออะไร {#edge-runtime-คืออะไร}

Edge Runtime เป็น JavaScript Runtime ที่ทำงานใกล้กับผู้ใช้ที่สุด บน Edge Servers ทั่วโลก แทนที่จะทำงานบน Origin Server เดียว

```
Traditional (Node.js Server):
User (Bangkok) ──── request ────► Server (US-East) ──── response ────► User
                    ~200ms latency

Edge Runtime:
User (Bangkok) ──── request ────► Edge Server (Singapore) ──── response ────► User
                    ~20ms latency
```

### Edge vs Node.js vs Serverless

```
Node.js (Lambda):
├── Full Node.js API
├── เข้าถึง file system ได้
├── Native modules ได้
├── Cold start: ~100ms-1s
└── ทำงานบน Origin Server

Edge Runtime:
├── Subset ของ Web APIs
├── ไม่มี file system
├── ไม่มี native modules
├── Cold start: ~0ms (เร็วมาก)
├── ทำงานบน Edge (CDN nodes)
└── ใกล้ผู้ใช้มากกว่า

Serverless:
├── Full Node.js API
├── Cold start อาจนาน
└── ทำงานบน Server ที่กำหนด
```

### Web APIs ที่รองรับใน Edge

```typescript
// รองรับ:
fetch()                    // HTTP requests
Request, Response          // Web request/response
Headers                    // HTTP headers
URL, URLSearchParams       // URL parsing
TextEncoder, TextDecoder   // Encoding
crypto                     // Web Crypto API
ReadableStream             // Streaming
WritableStream
TransformStream
setTimeout, setInterval    // Timers (จำกัด)
console                    // Logging
atob, btoa                 // Base64

// ไม่รองรับ:
fs, path, os               // Node.js modules
child_process              // Process spawning
net, http, https           // Node.js HTTP
Buffer (แทนด้วย Uint8Array)
__dirname, __filename
require()                  // CommonJS (ใช้ ESM แทน)
```

---

## Step 1725-1735: Edge Functions {#edge-functions}

### Route Handler กับ Edge Runtime

```typescript
// app/api/hello/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const name = searchParams.get('name') || 'World'
  
  return new Response(
    JSON.stringify({ message: `Hello, ${name}!` }),
    {
      headers: {
        'Content-Type': 'application/json',
        'Cache-Control': 'public, max-age=60',
      },
    }
  )
}

export async function POST(request: Request) {
  const body = await request.json()
  
  return Response.json({
    received: body,
    timestamp: new Date().toISOString(),
  })
}
```

### Edge Function สำหรับ Geolocation

```typescript
// app/api/geo/route.ts
export const runtime = 'edge'

export function GET(request: Request) {
  // Vercel ใส่ geo info ใน headers
  const country = request.headers.get('x-vercel-ip-country') || 'Unknown'
  const city = request.headers.get('x-vercel-ip-city') || 'Unknown'
  const region = request.headers.get('x-vercel-ip-country-region') || 'Unknown'
  const latitude = request.headers.get('x-vercel-ip-latitude') || '0'
  const longitude = request.headers.get('x-vercel-ip-longitude') || '0'
  const ip = request.headers.get('x-real-ip') || 
             request.headers.get('x-forwarded-for') || 
             'Unknown'
  
  return Response.json({
    ip,
    country,
    city,
    region,
    coordinates: {
      lat: parseFloat(latitude),
      lng: parseFloat(longitude),
    },
    timezone: request.headers.get('x-vercel-ip-timezone') || 'UTC',
  })
}
```

### Edge Function สำหรับ A/B Testing

```typescript
// app/api/ab-test/route.ts
export const runtime = 'edge'

const AB_TEST_COOKIE = 'ab-test-variant'

export function GET(request: Request) {
  const url = new URL(request.url)
  const existingCookie = request.headers.get('cookie')
  
  // Check existing cookie
  const existingVariant = getCookieValue(existingCookie, AB_TEST_COOKIE)
  
  if (existingVariant) {
    return Response.json({ variant: existingVariant })
  }
  
  // Assign variant (50/50 split)
  const variant = Math.random() < 0.5 ? 'A' : 'B'
  
  const response = Response.json({ variant })
  
  // Set cookie
  response.headers.set(
    'Set-Cookie',
    `${AB_TEST_COOKIE}=${variant}; Path=/; Max-Age=86400; SameSite=Lax`
  )
  
  return response
}

function getCookieValue(cookieString: string | null, name: string): string | null {
  if (!cookieString) return null
  const matches = cookieString.match(new RegExp(`(?:^|;\\s*)${name}=([^;]*)`))
  return matches ? matches[1] : null
}
```

### Edge Function สำหรับ Image Transformation

```typescript
// app/api/image/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const imageUrl = searchParams.get('url')
  const width = parseInt(searchParams.get('w') || '800')
  const quality = parseInt(searchParams.get('q') || '80')
  
  if (!imageUrl) {
    return new Response('Missing url parameter', { status: 400 })
  }
  
  // ดึงรูปภาพจาก origin
  const response = await fetch(imageUrl)
  
  if (!response.ok) {
    return new Response('Failed to fetch image', { status: 500 })
  }
  
  // ส่งรูปภาพกลับพร้อม cache headers
  return new Response(response.body, {
    headers: {
      'Content-Type': response.headers.get('Content-Type') || 'image/jpeg',
      'Cache-Control': 'public, max-age=86400, s-maxage=86400',
      'CDN-Cache-Control': 'public, max-age=31536000',
    },
  })
}
```

---

## Step 1736-1745: Edge Middleware {#edge-middleware}

Middleware ทำงานก่อนทุก request บน Edge

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import { jwtVerify } from 'jose'

export const config = {
  matcher: [
    // Match ทุก route ยกเว้น static files
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
    // Match routes เฉพาะ
    '/dashboard/:path*',
    '/admin/:path*',
  ],
}

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // ========================================
  // 1. Authentication
  // ========================================
  if (pathname.startsWith('/dashboard') || pathname.startsWith('/admin')) {
    const token = request.cookies.get('auth-token')?.value
    
    if (!token) {
      const loginUrl = new URL('/login', request.url)
      loginUrl.searchParams.set('callbackUrl', pathname)
      return NextResponse.redirect(loginUrl)
    }
    
    try {
      const secret = new TextEncoder().encode(process.env.JWT_SECRET!)
      const { payload } = await jwtVerify(token, secret)
      
      // ตรวจสอบ role สำหรับ admin
      if (pathname.startsWith('/admin') && payload.role !== 'admin') {
        return NextResponse.redirect(new URL('/403', request.url))
      }
      
      // ส่ง user info ไปยัง page headers
      const requestHeaders = new Headers(request.headers)
      requestHeaders.set('x-user-id', payload.sub as string)
      requestHeaders.set('x-user-role', payload.role as string)
      
      return NextResponse.next({
        request: { headers: requestHeaders },
      })
    } catch {
      const loginUrl = new URL('/login', request.url)
      const response = NextResponse.redirect(loginUrl)
      response.cookies.delete('auth-token')
      return response
    }
  }
  
  // ========================================
  // 2. Rate Limiting (Basic)
  // ========================================
  if (pathname.startsWith('/api/')) {
    const ip = request.headers.get('x-forwarded-for') || 
               request.headers.get('x-real-ip') || 
               'unknown'
    
    // ใช้ Upstash Redis สำหรับ production rate limiting
    // const { success } = await ratelimit.limit(ip)
    // if (!success) {
    //   return new NextResponse('Too Many Requests', { status: 429 })
    // }
  }
  
  // ========================================
  // 3. Internationalization
  // ========================================
  const locale = getLocale(request)
  const pathnameHasLocale = ['th', 'en', 'ja'].some(
    (loc) => pathname.startsWith(`/${loc}/`) || pathname === `/${loc}`
  )
  
  if (!pathnameHasLocale) {
    return NextResponse.redirect(
      new URL(`/${locale}${pathname}`, request.url)
    )
  }
  
  // ========================================
  // 4. Security Headers
  // ========================================
  const response = NextResponse.next()
  
  response.headers.set('X-DNS-Prefetch-Control', 'on')
  response.headers.set('X-Frame-Options', 'SAMEORIGIN')
  response.headers.set('X-Content-Type-Options', 'nosniff')
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
  response.headers.set(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=()'
  )
  
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    )
    response.headers.set(
      'Content-Security-Policy',
      [
        "default-src 'self'",
        "script-src 'self' 'unsafe-eval' 'unsafe-inline'",
        "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
        "font-src 'self' https://fonts.gstatic.com",
        "img-src 'self' data: https:",
        "connect-src 'self' https://api.example.com",
      ].join('; ')
    )
  }
  
  return response
}

function getLocale(request: NextRequest): string {
  // Check cookie
  const localeCookie = request.cookies.get('locale')?.value
  if (localeCookie) return localeCookie
  
  // Check Accept-Language header
  const acceptLanguage = request.headers.get('Accept-Language') || ''
  const preferredLocale = acceptLanguage.split(',')[0].split('-')[0]
  
  const supportedLocales = ['th', 'en', 'ja']
  if (supportedLocales.includes(preferredLocale)) {
    return preferredLocale
  }
  
  return 'th' // Default
}
```

### Middleware สำหรับ A/B Testing

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

const AB_VARIANTS = {
  homepage: {
    A: '/', // Original
    B: '/home-v2', // New version
    split: 0.5, // 50/50
  },
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  if (pathname === '/') {
    // Check existing assignment
    const existingVariant = request.cookies.get('homepage-variant')?.value
    
    if (existingVariant === 'B') {
      return NextResponse.rewrite(new URL('/home-v2', request.url))
    }
    
    if (existingVariant === 'A') {
      return NextResponse.next()
    }
    
    // Assign variant
    const variant = Math.random() < AB_VARIANTS.homepage.split ? 'A' : 'B'
    
    const response = variant === 'B'
      ? NextResponse.rewrite(new URL('/home-v2', request.url))
      : NextResponse.next()
    
    response.cookies.set('homepage-variant', variant, {
      maxAge: 60 * 60 * 24 * 7, // 1 week
    })
    
    return response
  }
  
  return NextResponse.next()
}
```

---

## Step 1746-1750: Vercel Edge Network {#vercel-edge-network}

```
Vercel Edge Network:
├── 100+ locations ทั่วโลก
├── Automatic routing ไปยัง closest edge
├── Built-in DDoS protection
├── Automatic SSL
└── Edge Config สำหรับ runtime configuration
```

### Edge Config

```typescript
// lib/edge-config.ts
import { createClient } from '@vercel/edge-config'

const edgeConfig = createClient(process.env.EDGE_CONFIG!)

// ใช้ใน middleware (Edge Runtime)
export async function getFeatureFlag(flag: string): Promise<boolean> {
  try {
    const value = await edgeConfig.get<boolean>(flag)
    return value ?? false
  } catch {
    return false
  }
}

// middleware.ts
import { getFeatureFlag } from './lib/edge-config'

export async function middleware(request: NextRequest) {
  const newFeatureEnabled = await getFeatureFlag('new-checkout-flow')
  
  if (newFeatureEnabled && request.nextUrl.pathname === '/checkout') {
    return NextResponse.rewrite(new URL('/checkout-v2', request.url))
  }
  
  return NextResponse.next()
}
```

---

## Step 1751-1760: Performance Benefits & Limitations {#performance-benefits}

### Performance Measurements

```
ตัวอย่างการวัด Performance:

Standard API Route (Node.js):
├── Cold start: 100-500ms
├── Latency: 100-300ms (จาก Asia ไป US server)
└── Memory: 128MB-1GB

Edge Function:
├── Cold start: 0ms (warm)
├── Latency: 10-50ms (ใกล้ผู้ใช้)
└── Memory: 128MB (จำกัด)
```

### เมื่อไรควรใช้ Edge Runtime

```typescript
// ✅ เหมาะกับ Edge Runtime:

// 1. Authentication/Authorization
export const runtime = 'edge'
export async function middleware(request: NextRequest) {
  const token = request.cookies.get('token')?.value
  if (!token) return NextResponse.redirect('/login')
  return NextResponse.next()
}

// 2. Geolocation-based content
export const runtime = 'edge'
export function GET(request: Request) {
  const country = request.headers.get('x-vercel-ip-country')
  if (country === 'TH') return showThaiContent()
  return showDefaultContent()
}

// 3. A/B Testing
// 4. Rate Limiting
// 5. Request transformation
// 6. Cache warming

// ❌ ไม่เหมาะกับ Edge Runtime:

// 1. Heavy computation
// 2. Database connections (Prisma ไม่รองรับ edge)
// 3. File system access
// 4. Node.js-specific modules
// 5. Large response bodies (> 4MB)
```

### Limitations

```
Edge Runtime Limitations:
├── ไม่รองรับ Node.js native modules (prisma, bcrypt, etc.)
├── Memory limit: 128MB
├── CPU limit: 50ms per request
├── Response size limit: 4MB
├── ไม่มี file system access
├── ไม่รองรับ net, http modules
├── No setTimeout > 30 seconds
└── Some npm packages ไม่รองรับ
```

### การแก้ปัญหา Database ใน Edge

```typescript
// ใช้ Prisma Accelerate (รองรับ Edge)
import { PrismaClient } from '@prisma/client/edge'
import { withAccelerate } from '@prisma/extension-accelerate'

const prisma = new PrismaClient().$extends(withAccelerate())

// ใช้ Neon HTTP สำหรับ Edge
import { neon } from '@neondatabase/serverless'

const sql = neon(process.env.DATABASE_URL!)

// ใช้ Drizzle ORM กับ HTTP driver
import { drizzle } from 'drizzle-orm/neon-http'
import { neon } from '@neondatabase/serverless'

const sql = neon(process.env.DATABASE_URL!)
const db = drizzle(sql)
```

---

## 🧪 Quiz - Part 53

**ข้อ 1:** Edge Runtime ต่างจาก Node.js Runtime อย่างไร?
- A) Edge รองรับ Node.js modules ทั้งหมด
- B) Edge ทำงานใกล้ผู้ใช้มากกว่า มี cold start น้อยกว่า แต่ API จำกัดกว่า
- C) Edge เร็วกว่าในทุกกรณี
- D) Edge ใช้ PHP

**ข้อ 2:** Middleware ใน Next.js ทำงานที่ไหน?
- A) บน Client
- B) บน Origin Server
- C) บน Edge ก่อน request ถึง application
- D) ใน Service Worker

**ข้อ 3:** ทำไม Prisma ถึงไม่รองรับ Edge Runtime โดยตรง?
- A) เพราะ Prisma ใช้ PHP
- B) เพราะ Prisma ใช้ Node.js native modules และ TCP connections
- C) เพราะ Prisma ไม่รองรับ TypeScript
- D) เพราะ Prisma ช้าเกินไป

**ข้อ 4:** Edge Config ใน Vercel ใช้ทำอะไร?
- A) เก็บ database records
- B) กำหนดค่า configuration ที่อ่านได้เร็วมากในระดับ Edge
- C) Deploy code
- D) Monitor performance

**เฉลย:** 1-B, 2-C, 3-B, 4-B

---

> **➡️ Next:** [Part 54: Next.js Caching Strategies](./part-54-nextjs-caching-strategies.md)
