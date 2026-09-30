# Part 33: Pages Router vs App Router

## ข้อมูล Part
- **Steps:** 926-960
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3 ชั่วโมง
- **Prerequisites:** Part 32 (Next.js Routing)

---

## สารบัญ

1. [Pages Router (Legacy)](#1-pages-router-legacy)
2. [App Router (Next.js 13+)](#2-app-router-nextjs-13)
3. [Migration Guide](#3-migration-guide)
4. [layout.tsx](#4-layouttsx)
5. [page.tsx, error.tsx, loading.tsx, not-found.tsx](#5-pagetsx-errortsx-loadingtsx-not-foundtsx)
6. [template.tsx](#6-templatetsx)
7. [Route Handlers (API Routes ใหม่)](#7-route-handlers-api-routes-ใหม่)
8. [Quiz](#quiz)

---

## Step 926: Pages Router Overview

### 1. Pages Router (Legacy)

Pages Router คือระบบ Routing เดิมของ Next.js ก่อน Version 13 ที่ยังคงรองรับในเวอร์ชันปัจจุบัน

#### โครงสร้าง Pages Router

```
pages/
├── _app.js          → Custom App Component (Global Layout)
├── _document.js     → Custom Document (HTML Structure)
├── index.js         → / (Home Page)
├── about.js         → /about
├── blog/
│   ├── index.js     → /blog
│   └── [slug].js    → /blog/:slug
└── api/
    ├── users.js     → /api/users
    └── posts/
        └── [id].js  → /api/posts/:id
```

#### _app.js - Global Layout

```jsx
// pages/_app.js
import type { AppProps } from 'next/app'
import Head from 'next/head'
import '../styles/globals.css'

export default function App({ Component, pageProps }: AppProps) {
  return (
    <>
      <Head>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
      </Head>
      <Header />
      <Component {...pageProps} />
      <Footer />
    </>
  )
}
```

#### _document.js - HTML Structure

```jsx
// pages/_document.js
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="th">
      <Head>
        <link rel="preconnect" href="https://fonts.googleapis.com" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

#### Data Fetching ใน Pages Router

```tsx
// pages/blog/[slug].tsx
import type { GetStaticProps, GetStaticPaths, GetServerSideProps } from 'next'

interface Post {
  slug: string
  title: string
  content: string
}

// ── SSG: getStaticProps + getStaticPaths ──
export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await getPosts()
  
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: false  // หรือ 'blocking' หรือ true
  }
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await getPost(params?.slug as string)
  
  if (!post) {
    return { notFound: true }
  }
  
  return {
    props: { post },
    revalidate: 60  // ISR: Revalidate ทุก 60 วินาที
  }
}

// ── SSR: getServerSideProps ──
export const getServerSideProps: GetServerSideProps = async (context) => {
  const { params, req, res, query } = context
  const post = await getPost(params?.slug as string)
  
  return {
    props: { post }
  }
}

export default function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

#### API Routes ใน Pages Router

```typescript
// pages/api/users.ts
import type { NextApiRequest, NextApiResponse } from 'next'

interface User {
  id: number
  name: string
  email: string
}

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<User[] | { error: string }>
) {
  if (req.method === 'GET') {
    const users = [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' },
    ]
    res.status(200).json(users)
  } else {
    res.status(405).json({ error: 'Method Not Allowed' })
  }
}
```

#### ข้อจำกัดของ Pages Router

```
✗ ไม่มี Server Components
✗ ทุก Page Load ทั้งหมดก่อน Render (ไม่มี Streaming)
✗ Layout ซ้อนกันหลายชั้นซับซ้อน
✗ Data Fetching แยกออกจาก Component (getStaticProps ฯลฯ)
✗ ไม่มี Server Actions
✗ Client JavaScript Bundle ใหญ่กว่า
```

---

## Step 930: App Router

### 2. App Router (Next.js 13+)

#### โครงสร้าง App Router

```
app/
├── layout.tsx           → Root Layout
├── page.tsx             → Home Page
├── error.tsx            → Error Boundary
├── loading.tsx          → Loading State
├── not-found.tsx        → 404 Page
├── globals.css
├── blog/
│   ├── layout.tsx       → Blog Layout
│   ├── page.tsx         → Blog List
│   ├── loading.tsx      → Blog Loading
│   └── [slug]/
│       ├── page.tsx     → Blog Post
│       └── not-found.tsx
└── api/
    └── users/
        └── route.ts     → Route Handler
```

#### Server Components vs Client Components

```
┌──────────────────────────────────────────────────────┐
│                     App Router                        │
│                                                      │
│  Server Components (default)                         │
│  ┌──────────────────────────────────────────────┐   │
│  │  - รันที่ Server เท่านั้น                     │   │
│  │  - ไม่มีใน JavaScript Bundle                  │   │
│  │  - ใช้ async/await ได้โดยตรง                  │   │
│  │  - ไม่มี useState, useEffect                  │   │
│  │  - ไม่มี Browser APIs                         │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  Client Components ('use client')                   │
│  ┌──────────────────────────────────────────────┐   │
│  │  - รันทั้ง Server (SSR) และ Browser          │   │
│  │  - มีใน JavaScript Bundle                    │   │
│  │  - ใช้ useState, useEffect ได้               │   │
│  │  - ใช้ Browser APIs ได้                      │   │
│  │  - ใช้ Event Handlers ได้                    │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

---

## Step 933: Migration Guide

### 3. Migration Guide

#### จาก Pages Router → App Router

**Step 1: สร้าง app/ Directory**

```
ก่อน Migration:
/
├── pages/
│   ├── _app.js
│   ├── index.js
│   └── about.js
└── styles/
    └── globals.css

หลัง Migration (ทีละน้อย):
/
├── pages/      (ยังคงทำงาน)
│   └── _app.js
├── app/        (เพิ่มใหม่)
│   ├── layout.tsx
│   └── about/
│       └── page.tsx
└── styles/
    └── globals.css
```

**Step 2: Migrate Layout**

```tsx
// Before: pages/_app.js
export default function App({ Component, pageProps }) {
  return (
    <html>
      <body>
        <Header />
        <Component {...pageProps} />
        <Footer />
      </body>
    </html>
  )
}

// After: app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="th">
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  )
}
```

**Step 3: Migrate Pages**

```tsx
// Before: pages/blog/[slug].js (Pages Router)
export async function getStaticPaths() {
  const posts = await fetchPosts()
  return {
    paths: posts.map(p => ({ params: { slug: p.slug } })),
    fallback: false
  }
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug)
  return { props: { post } }
}

export default function BlogPost({ post }) {
  return <article>{post.title}</article>
}

// After: app/blog/[slug]/page.tsx (App Router)
export async function generateStaticParams() {
  const posts = await fetchPosts()
  return posts.map(p => ({ slug: p.slug }))
}

export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug)
  return <article>{post.title}</article>
}
```

**Step 4: Migrate API Routes**

```typescript
// Before: pages/api/users.ts (Pages Router)
export default function handler(req, res) {
  if (req.method === 'GET') {
    res.status(200).json({ users: [] })
  }
}

// After: app/api/users/route.ts (App Router)
export async function GET() {
  return Response.json({ users: [] })
}
```

**Step 5: Migrate Client Components**

```tsx
// Before: pages/counter.js (Pages Router)
// ทุกอย่างเป็น Client Component โดย Default
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}

// After: components/Counter.tsx (App Router)
// ต้องเพิ่ม 'use client' เมื่อใช้ State/Events
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

---

## Step 936: layout.tsx

### 4. layout.tsx

Layout เป็น Component ที่ Shared ระหว่าง Routes และ Maintain State เมื่อ Navigate

#### Root Layout (บังคับ)

```tsx
// app/layout.tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'

const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap',
})

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App',
  },
  description: 'My Next.js Application',
  metadataBase: new URL('https://myapp.com'),
  openGraph: {
    type: 'website',
    locale: 'th_TH',
    url: 'https://myapp.com',
    siteName: 'My App',
  },
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th" suppressHydrationWarning>
      <body className={inter.className}>
        <ThemeProvider>
          <Header />
          <main className="min-h-screen">
            {children}
          </main>
          <Footer />
        </ThemeProvider>
      </body>
    </html>
  )
}
```

#### Nested Layout

```tsx
// app/dashboard/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    default: 'Dashboard',
    template: '%s | Dashboard',
  },
}

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="flex min-h-screen">
      {/* Sidebar */}
      <aside className="w-64 bg-gray-900 text-white">
        <nav className="p-4">
          <ul className="space-y-2">
            <li><a href="/dashboard">Overview</a></li>
            <li><a href="/dashboard/analytics">Analytics</a></li>
            <li><a href="/dashboard/settings">Settings</a></li>
          </ul>
        </nav>
      </aside>
      
      {/* Content */}
      <div className="flex-1 p-8">
        {children}
      </div>
    </div>
  )
}
```

#### Layout กับ Server Components

```tsx
// app/layout.tsx
import { getCurrentUser } from '@/lib/auth'

export default async function RootLayout({ children }) {
  // Server-side Auth Check
  const user = await getCurrentUser()
  
  return (
    <html>
      <body>
        <Header user={user} />
        {children}
      </body>
    </html>
  )
}
```

---

## Step 939: page.tsx, error.tsx, loading.tsx, not-found.tsx

### 5. page.tsx, error.tsx, loading.tsx, not-found.tsx

#### page.tsx

```tsx
// app/blog/page.tsx
import type { Metadata } from 'next'

// Static Metadata
export const metadata: Metadata = {
  title: 'Blog',
  description: 'อ่านบทความเกี่ยวกับ Web Development',
}

// Dynamic Metadata
export async function generateMetadata({ params, searchParams }) {
  const category = searchParams.category
  return {
    title: category ? `Blog - ${category}` : 'Blog',
  }
}

interface PageProps {
  params: { [key: string]: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default async function BlogPage({ params, searchParams }: PageProps) {
  const page = Number(searchParams.page) || 1
  const posts = await getPosts({ page })
  
  return (
    <div>
      <h1>Blog</h1>
      <PostList posts={posts} />
    </div>
  )
}
```

#### loading.tsx

```tsx
// app/blog/loading.tsx
// แสดงขณะ Page กำลัง Load
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-6"></div>
      {[1, 2, 3].map(i => (
        <div key={i} className="mb-4">
          <div className="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
          <div className="h-4 bg-gray-200 rounded w-1/2"></div>
        </div>
      ))}
    </div>
  )
}

// ทำงานอย่างไร:
// Next.js Wrap page.tsx ด้วย <Suspense fallback={<Loading />}>
// อัตโนมัติ
```

#### error.tsx

```tsx
// app/blog/error.tsx
'use client'  // Error Components ต้องเป็น Client Component

import { useEffect } from 'react'

interface ErrorProps {
  error: Error & { digest?: string }
  reset: () => void  // ลองใหม่อีกครั้ง
}

export default function Error({ error, reset }: ErrorProps) {
  useEffect(() => {
    // Log error ไปยัง Error Reporting Service
    console.error('Blog Error:', error)
  }, [error])
  
  return (
    <div className="flex flex-col items-center justify-center min-h-96">
      <h2 className="text-2xl font-bold text-red-600 mb-4">
        เกิดข้อผิดพลาด!
      </h2>
      <p className="text-gray-600 mb-6">
        {error.message || 'Something went wrong'}
      </p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        ลองใหม่
      </button>
    </div>
  )
}
```

#### global-error.tsx

```tsx
// app/global-error.tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    // global-error ต้องมี html และ body เอง
    <html>
      <body>
        <div className="min-h-screen flex items-center justify-center">
          <div className="text-center">
            <h2 className="text-3xl font-bold text-red-600 mb-4">
              เกิดข้อผิดพลาดร้ายแรง!
            </h2>
            <p className="mb-6">{error.message}</p>
            <button onClick={reset}>ลองใหม่</button>
          </div>
        </div>
      </body>
    </html>
  )
}
```

#### not-found.tsx

```tsx
// app/not-found.tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-9xl font-bold text-gray-300">404</h1>
        <h2 className="text-3xl font-bold text-gray-700 mb-4">
          ไม่พบหน้าที่คุณต้องการ
        </h2>
        <p className="text-gray-500 mb-8">
          หน้าที่คุณกำลังมองหาไม่มีอยู่หรือถูกย้ายไปแล้ว
        </p>
        <Link 
          href="/"
          className="px-6 py-3 bg-blue-500 text-white rounded-lg"
        >
          กลับหน้าหลัก
        </Link>
      </div>
    </div>
  )
}

// การ Trigger notFound():
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  if (!post) notFound()  // Renders not-found.tsx
  return <article>{post.title}</article>
}
```

---

## Step 942: template.tsx

### 6. template.tsx

`template.tsx` คล้าย `layout.tsx` แต่สร้าง Instance ใหม่ทุกครั้งที่ Navigate

#### layout.tsx vs template.tsx

```
layout.tsx:
- Persist State เมื่อ Navigate
- ไม่ Re-render เมื่อ Navigate
- เหมาะสำหรับ Persistent State (Cart, Auth, Theme)

template.tsx:
- สร้าง Instance ใหม่ทุกครั้ง
- Re-render เมื่อ Navigate
- เหมาะสำหรับ Page Transitions, Analytics
```

#### ตัวอย่าง template.tsx

```tsx
// app/template.tsx
'use client'
import { motion } from 'framer-motion'

export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  )
}
```

#### ตัวอย่าง Page View Analytics

```tsx
// app/template.tsx
'use client'
import { usePathname } from 'next/navigation'
import { useEffect } from 'react'
import { trackPageView } from '@/lib/analytics'

export default function Template({ children }) {
  const pathname = usePathname()
  
  useEffect(() => {
    // ทุกครั้งที่ Navigate จะ Track Page View
    trackPageView(pathname)
  }, [pathname])
  
  return <>{children}</>
}
```

---

## Step 945: Route Handlers

### 7. Route Handlers (API Routes ใหม่)

Route Handlers คือ API Endpoints ใหม่ใน App Router ที่ใช้ Web Request/Response API

#### โครงสร้าง

```
app/
└── api/
    ├── users/
    │   ├── route.ts          → /api/users
    │   └── [id]/
    │       └── route.ts      → /api/users/:id
    ├── posts/
    │   └── route.ts          → /api/posts
    └── upload/
        └── route.ts          → /api/upload
```

#### GET Handler

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'

// Mock Database
const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
]

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const search = searchParams.get('search')
  
  let result = users
  if (search) {
    result = users.filter(u => 
      u.name.toLowerCase().includes(search.toLowerCase())
    )
  }
  
  return NextResponse.json(result)
}
```

#### POST Handler

```typescript
// app/api/users/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const { name, email } = body
    
    // Validation
    if (!name || !email) {
      return NextResponse.json(
        { error: 'Name and email are required' },
        { status: 400 }
      )
    }
    
    // Create User
    const newUser = {
      id: Date.now(),
      name,
      email,
    }
    users.push(newUser)
    
    return NextResponse.json(newUser, { status: 201 })
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid request body' },
      { status: 400 }
    )
  }
}
```

#### PUT และ DELETE Handler

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

interface RouteParams {
  params: { id: string }
}

export async function GET(request: NextRequest, { params }: RouteParams) {
  const user = await getUserById(Number(params.id))
  
  if (!user) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 })
  }
  
  return NextResponse.json(user)
}

export async function PUT(request: NextRequest, { params }: RouteParams) {
  const body = await request.json()
  const updatedUser = await updateUser(Number(params.id), body)
  
  if (!updatedUser) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 })
  }
  
  return NextResponse.json(updatedUser)
}

export async function PATCH(request: NextRequest, { params }: RouteParams) {
  const body = await request.json()
  const user = await patchUser(Number(params.id), body)
  return NextResponse.json(user)
}

export async function DELETE(request: NextRequest, { params }: RouteParams) {
  const deleted = await deleteUser(Number(params.id))
  
  if (!deleted) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 })
  }
  
  return NextResponse.json({ message: 'User deleted' })
}
```

#### Headers และ Cookies

```typescript
// app/api/profile/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { cookies, headers } from 'next/headers'

export async function GET(request: NextRequest) {
  // อ่าน Headers
  const headersList = headers()
  const authorization = headersList.get('authorization')
  const userAgent = headersList.get('user-agent')
  
  // อ่าน Cookies
  const cookieStore = cookies()
  const token = cookieStore.get('token')
  
  // อ่าน Request Headers โดยตรง
  const authHeader = request.headers.get('authorization')
  
  if (!authHeader?.startsWith('Bearer ')) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    )
  }
  
  const tokenValue = authHeader.split(' ')[1]
  const user = await getUserFromToken(tokenValue)
  
  // ตั้งค่า Response Headers
  const response = NextResponse.json(user)
  response.headers.set('X-User-ID', String(user.id))
  
  // ตั้งค่า Cookies
  response.cookies.set('last-visit', new Date().toISOString(), {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 * 7, // 7 days
  })
  
  return response
}
```

#### CORS

```typescript
// app/api/public/route.ts
import { NextRequest, NextResponse } from 'next/server'

const CORS_HEADERS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
}

export async function OPTIONS() {
  return NextResponse.json({}, { headers: CORS_HEADERS })
}

export async function GET() {
  const data = await getPublicData()
  return NextResponse.json(data, { headers: CORS_HEADERS })
}
```

#### File Upload Handler

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { writeFile } from 'fs/promises'
import path from 'path'

export async function POST(request: NextRequest) {
  const formData = await request.formData()
  const file = formData.get('file') as File
  
  if (!file) {
    return NextResponse.json({ error: 'No file provided' }, { status: 400 })
  }
  
  // Check File Type
  if (!file.type.startsWith('image/')) {
    return NextResponse.json(
      { error: 'Only images are allowed' },
      { status: 400 }
    )
  }
  
  // Check File Size (Max 5MB)
  if (file.size > 5 * 1024 * 1024) {
    return NextResponse.json(
      { error: 'File too large (max 5MB)' },
      { status: 400 }
    )
  }
  
  const bytes = await file.arrayBuffer()
  const buffer = Buffer.from(bytes)
  
  const fileName = `${Date.now()}-${file.name}`
  const filePath = path.join(process.cwd(), 'public', 'uploads', fileName)
  
  await writeFile(filePath, buffer)
  
  return NextResponse.json({
    url: `/uploads/${fileName}`,
    size: file.size,
    type: file.type,
  })
}
```

#### Streaming Response

```typescript
// app/api/stream/route.ts
export async function GET() {
  const encoder = new TextEncoder()
  
  const stream = new ReadableStream({
    async start(controller) {
      const messages = ['Hello', ' World', '!']
      
      for (const message of messages) {
        controller.enqueue(encoder.encode(`data: ${message}\n\n`))
        await new Promise(resolve => setTimeout(resolve, 500))
      }
      
      controller.close()
    }
  })
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    }
  })
}
```

---

## Step 950: ตัวอย่างสมบูรณ์

### ตัวอย่างสมบูรณ์: Blog App ด้วย App Router

```
app/
├── layout.tsx
├── page.tsx
├── loading.tsx
├── error.tsx
├── not-found.tsx
├── blog/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── loading.tsx
│   └── [slug]/
│       ├── page.tsx
│       └── not-found.tsx
└── api/
    └── posts/
        ├── route.ts
        └── [id]/
            └── route.ts
```

```tsx
// app/blog/layout.tsx
export default function BlogLayout({ children }) {
  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <nav className="mb-8 text-sm">
        <a href="/" className="text-gray-500">Home</a>
        <span className="mx-2">/</span>
        <a href="/blog" className="text-gray-700">Blog</a>
      </nav>
      {children}
    </div>
  )
}

// app/blog/page.tsx
export default async function BlogListPage() {
  const posts = await getPosts()
  return (
    <div>
      <h1 className="text-4xl font-bold mb-8">Blog</h1>
      <div className="space-y-8">
        {posts.map(post => (
          <PostCard key={post.id} post={post} />
        ))}
      </div>
    </div>
  )
}

// app/blog/loading.tsx
export default function BlogLoading() {
  return (
    <div>
      <div className="h-10 bg-gray-200 rounded w-24 mb-8 animate-pulse"></div>
      {[1, 2, 3].map(i => (
        <div key={i} className="mb-8 animate-pulse">
          <div className="h-6 bg-gray-200 rounded w-3/4 mb-3"></div>
          <div className="h-4 bg-gray-200 rounded w-full mb-2"></div>
          <div className="h-4 bg-gray-200 rounded w-5/6"></div>
        </div>
      ))}
    </div>
  )
}

// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(p => ({ slug: p.slug }))
}

export async function generateMetadata({ params }) {
  const post = await getPostBySlug(params.slug)
  if (!post) return { title: 'Not Found' }
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      images: [post.coverImage],
    }
  }
}

export default async function BlogPostPage({ params }) {
  const post = await getPostBySlug(params.slug)
  if (!post) notFound()
  
  return (
    <article>
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        <p className="text-gray-500">{post.publishedAt}</p>
      </header>
      <div 
        className="prose"
        dangerouslySetInnerHTML={{ __html: post.contentHtml }}
      />
    </article>
  )
}
```

---

## Step 955: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. เลือก layout.tsx หรือ template.tsx ให้ถูก

✓ layout.tsx: ต้องการ Persist State (Sidebar, Cart)
✓ template.tsx: ต้องการ Re-render ทุก Navigate (Analytics, Transitions)

## 2. Error Boundary ที่เหมาะสม

✓ error.tsx ระดับ Folder สำหรับ Section-specific Errors
✓ app/global-error.tsx สำหรับ Root Layout Errors
✓ ต้องเป็น Client Component เสมอ

## 3. Loading States

✓ loading.tsx เพิ่ม User Experience
✓ ใช้ Skeleton Loading แทน Spinner เมื่อทำได้
✓ หลีกเลี่ยง Layout Shift

## 4. Not Found Pages

✓ สร้าง not-found.tsx ทุก Dynamic Route
✓ ใช้ notFound() ทันทีเมื่อ Resource ไม่พบ

## 5. Route Handlers

✓ ใช้ Web Standard Request/Response
✓ ตรวจสอบ Method ใน Handler
✓ ตั้งค่า CORS เมื่อมี External Client
```

---

## Quiz

### แบบทดสอบ Part 33

**คำถามที่ 1:** ไฟล์ `loading.tsx` ทำงานอย่างไร?
- A) ต้องเพิ่ม Suspense เองในทุก Component
- B) Next.js Wrap page.tsx ด้วย Suspense อัตโนมัติ ✓
- C) ทำงานเฉพาะกับ Client Components
- D) แสดงผลทุกครั้งที่เข้าหน้า

**คำถามที่ 2:** `error.tsx` ต้องเป็น Component ชนิดใด?
- A) Server Component
- B) Client Component ✓
- C) ทั้งสองอย่างได้
- D) ไม่สำคัญ

**คำถามที่ 3:** ความแตกต่างหลักระหว่าง `layout.tsx` และ `template.tsx` คืออะไร?
- A) template.tsx ใช้สำหรับ API Routes
- B) layout.tsx Persist State, template.tsx สร้าง Instance ใหม่ทุก Navigate ✓
- C) layout.tsx เร็วกว่า template.tsx
- D) ไม่มีความแตกต่าง

**คำถามที่ 4:** Route Handler ใน App Router ใช้ไฟล์ชื่ออะไร?
- A) handler.ts
- B) api.ts
- C) route.ts ✓
- D) endpoint.ts

**คำถามที่ 5:** `getServerSideProps` ใน Pages Router เทียบเท่ากับอะไรใน App Router?
- A) generateStaticParams
- B) ใช้ fetch() กับ `cache: 'no-store'` ใน Server Component ✓
- C) useEffect
- D) getStaticProps

---

## สรุป Part 33

ใน Part นี้เราได้เรียนรู้:

1. **Pages Router** - ระบบเก่าที่ยังใช้งานได้
2. **App Router** - ระบบใหม่ที่มี Server Components
3. **Migration** - วิธีย้ายจาก Pages Router ไป App Router
4. **layout.tsx** - Layout ที่ Persist State
5. **Reserved Files** - page, loading, error, not-found, template
6. **Route Handlers** - API Endpoints ใหม่ด้วย Web Standard API

---

➡️ **Part ถัดไป:** [Part 34: Next.js Data Fetching](./part-34-nextjs-data-fetching.md)
