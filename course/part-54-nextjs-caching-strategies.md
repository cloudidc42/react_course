# Part 54: Next.js Caching Strategies

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1761-1800  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [Caching ใน Next.js (4 ระดับ)](#caching-4-ระดับ)
2. [Request Memoization](#request-memoization)
3. [Data Cache](#data-cache)
4. [Full Route Cache](#full-route-cache)
5. [Router Cache](#router-cache)
6. [revalidatePath และ revalidateTag](#revalidate)
7. [unstable_cache](#unstable-cache)
8. [Incremental Static Regeneration (ISR)](#isr)
9. [Quiz](#quiz)

---

## Step 1761: Caching ใน Next.js (4 ระดับ) {#caching-4-ระดับ}

Next.js มีระบบ Caching 4 ระดับที่ทำงานร่วมกัน

```
Request ────►  Next.js Server
                │
                ▼
        ┌───────────────────┐
        │  1. Router Cache  │ (Client-side)
        │  (บน Browser)     │
        └────────┬──────────┘
                 │ cache miss
                 ▼
        ┌───────────────────┐
        │ 2. Full Route     │ (Server-side)
        │    Cache          │
        │ (HTML + RSC data) │
        └────────┬──────────┘
                 │ cache miss
                 ▼
        ┌───────────────────┐
        │  3. Data Cache    │ (Server-side)
        │  (fetch() cache)  │
        └────────┬──────────┘
                 │ cache miss
                 ▼
        ┌───────────────────┐
        │ 4. Request        │ (Per-request)
        │    Memoization    │
        │  (Dedup during    │
        │   render)         │
        └───────────────────┘
```

---

## Step 1762-1770: Request Memoization {#request-memoization}

Request Memoization ป้องกันไม่ให้ fetch ข้อมูลซ้ำกันในการ render ครั้งเดียว

```typescript
// ปัญหา: fetch ซ้ำ
// ถ้า Layout, Page และ Component ต่างๆ ต้อง fetch ข้อมูลเดียวกัน

// app/layout.tsx
async function Layout({ children }) {
  const user = await fetch('/api/user').then(r => r.json()) // fetch #1
  return <div>{children}</div>
}

// app/page.tsx
async function Page() {
  const user = await fetch('/api/user').then(r => r.json()) // fetch #2 (ซ้ำ!)
  return <div>Hello {user.name}</div>
}

// app/components/Header.tsx
async function Header() {
  const user = await fetch('/api/user').then(r => r.json()) // fetch #3 (ซ้ำอีก!)
  return <header>Welcome {user.name}</header>
}
```

```typescript
// Next.js Memoization แก้ปัญหา:
// fetch() calls ที่มี URL เหมือนกัน จะถูก deduplicate โดยอัตโนมัติ

// fetch #1 → network request
const user = await fetch('/api/user')

// fetch #2 → จาก memory cache (ไม่มี network request)
const user = await fetch('/api/user')

// Note: Memoization ใช้งานแค่ในระหว่าง render เดียว
// ไม่ถาวรข้าม requests
```

### Manual Memoization กับ React cache

```typescript
// lib/data.ts
import { cache } from 'react'

// cache() จาก React - memoize ใน single render tree
export const getUser = cache(async (userId: string) => {
  console.log('Fetching user...') // จะ log แค่ครั้งเดียวแม้เรียกหลายครั้ง
  
  const res = await fetch(`https://api.example.com/users/${userId}`)
  if (!res.ok) throw new Error('Failed to fetch user')
  return res.json()
})

export const getPost = cache(async (postId: string) => {
  const res = await fetch(`https://api.example.com/posts/${postId}`)
  if (!res.ok) throw new Error('Failed to fetch post')
  return res.json()
})

// ใช้ใน layout, page, components ได้เลย
// app/layout.tsx
async function RootLayout({ children }) {
  const user = await getUser('123') // fetch
  return <html>{children}</html>
}

// app/page.tsx
async function Page() {
  const user = await getUser('123') // memoized (ไม่ fetch ใหม่)
  return <div>{user.name}</div>
}
```

---

## Step 1771-1778: Data Cache {#data-cache}

Data Cache เก็บ results ของ fetch() requests ข้าม requests และ deployments

```typescript
// ตัวอย่างต่างๆ ของ Data Cache

// 1. Cache ตลอดไป (Static)
const data = await fetch('https://api.example.com/data', {
  cache: 'force-cache', // default
})

// 2. ไม่ Cache (Dynamic)
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store',
})

// 3. Cache แล้ว Revalidate ตามเวลา
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }, // revalidate ทุก 1 ชั่วโมง
})

// 4. Cache แล้ว Revalidate ตาม Tag
const data = await fetch('https://api.example.com/data', {
  next: { tags: ['products', 'featured'] },
})

// 5. ปิด Cache ทั้งหน้า (Dynamic Rendering)
export const dynamic = 'force-dynamic'
// หรือ
export const revalidate = 0
```

### Data Cache กับ Database Queries

```typescript
// Data Cache ใช้ได้แค่กับ fetch() ไม่ใช่ database queries โดยตรง
// สำหรับ database ใช้ unstable_cache หรือ next/cache

// ❌ ไม่ cache
const users = await prisma.user.findMany()

// ✅ cache ด้วย unstable_cache
import { unstable_cache } from 'next/cache'

const getUsers = unstable_cache(
  async () => {
    return prisma.user.findMany()
  },
  ['users-list'],
  {
    tags: ['users'],
    revalidate: 3600,
  }
)

const users = await getUsers()
```

---

## Step 1779-1784: Full Route Cache {#full-route-cache}

Full Route Cache เก็บ HTML และ RSC payload ของแต่ละ route ไว้บน server

```typescript
// Static Routes (Full Route Cache enabled by default)
// pages ที่ไม่มี dynamic data จะถูก cache โดยอัตโนมัติ

// app/blog/page.tsx - Static
export default async function BlogPage() {
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache',
  }).then(r => r.json())
  
  return (
    <div>
      {posts.map(post => <PostCard key={post.id} post={post} />)}
    </div>
  )
}

// Dynamic Routes (opt out of Full Route Cache)
// วิธีที่ 1: export dynamic
export const dynamic = 'force-dynamic'

// วิธีที่ 2: ใช้ cookies, headers
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = cookies()
  const theme = cookieStore.get('theme')
  // การใช้ cookies() ทำให้ page เป็น dynamic โดยอัตโนมัติ
  
  return <div>Theme: {theme?.value}</div>
}

// วิธีที่ 3: ใช้ searchParams
export default function Page({ searchParams }) {
  // การใช้ searchParams ทำให้ page เป็น dynamic
  const query = searchParams.q
  return <SearchResults query={query} />
}

// วิธีที่ 4: export revalidate = 0
export const revalidate = 0
```

### generateStaticParams สำหรับ Dynamic Routes

```typescript
// app/blog/[slug]/page.tsx
// สร้าง static pages สำหรับทุก slug

export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json())
  
  return posts.map((post: any) => ({
    slug: post.slug,
  }))
}

// revalidate ทั้งหมด
export const revalidate = 3600 // 1 hour

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`, {
    next: { revalidate: 3600 },
  }).then(r => r.json())
  
  return <article>{post.content}</article>
}
```

---

## Step 1785-1788: Router Cache {#router-cache}

Router Cache อยู่ใน Browser, cache RSC payload สำหรับ navigation

```
Router Cache:
├── อยู่ใน Browser Memory
├── Cache ระหว่าง navigations ใน session เดียว
├── Static segments: cache ตลอด session
├── Dynamic segments: cache 30 วินาที
└── ล้างเมื่อ refresh หน้า

เช่น:
1. ผู้ใช้เปิด /blog
2. คลิกไปที่ /blog/post-1 (cache ใน Router Cache)
3. กด Back → /blog (ดึงจาก Router Cache, ไม่ fetch ใหม่)
4. คลิกไปที่ /blog/post-1 อีกครั้ง (ดึงจาก Router Cache ถ้ายังไม่หมดอายุ)
```

```typescript
// ล้าง Router Cache ด้วย router.refresh()
'use client'
import { useRouter } from 'next/navigation'

export function RefreshButton() {
  const router = useRouter()
  
  return (
    <button onClick={() => router.refresh()}>
      Refresh
    </button>
  )
}

// ล้าง Router Cache ด้วย revalidatePath (Server Action)
'use server'
import { revalidatePath } from 'next/cache'

export async function updatePost(id: string) {
  await db.post.update({ ... })
  revalidatePath('/blog') // ล้าง cache ของ /blog และทุก route ใต้มัน
}
```

---

## Step 1789-1793: revalidatePath และ revalidateTag {#revalidate}

### revalidatePath

```typescript
// ล้าง cache ของ path ที่กำหนด

import { revalidatePath } from 'next/cache'

// ล้าง cache ของ page เดียว
revalidatePath('/blog')

// ล้าง cache ของ dynamic route
revalidatePath('/blog/my-post-slug')

// ล้าง cache ของ layout (ทุก pages ใต้ layout นั้น)
revalidatePath('/blog', 'layout')

// ล้าง cache ของ page
revalidatePath('/blog', 'page')

// ตัวอย่าง Server Action
// app/blog/[slug]/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function updatePost(formData: FormData) {
  const id = formData.get('id') as string
  const title = formData.get('title') as string
  const content = formData.get('content') as string
  
  await db.post.update({
    where: { id },
    data: { title, content },
  })
  
  // ล้าง cache ของหน้านี้และหน้า blog list
  revalidatePath(`/blog/${formData.get('slug')}`)
  revalidatePath('/blog')
  
  redirect(`/blog/${formData.get('slug')}`)
}
```

### revalidateTag

```typescript
// ล้าง cache โดยใช้ tag
import { revalidateTag } from 'next/cache'

// Server Action สำหรับ delete post
'use server'
export async function deletePost(id: string) {
  await db.post.delete({ where: { id } })
  
  // ล้าง cache ทุก fetch() ที่ใช้ tag 'posts'
  revalidateTag('posts')
}

// กำหนด tags ใน fetch()
const posts = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] }, // จะถูก revalidate เมื่อเรียก revalidateTag('posts')
})

const post = await fetch(`https://api.example.com/posts/${id}`, {
  next: { 
    tags: ['posts', `post-${id}`], // หลาย tags
    revalidate: 3600,
  },
})

// ตัวอย่างการใช้งาน Route Handler สำหรับ Webhook
// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret')
  
  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ message: 'Invalid secret' }, { status: 401 })
  }
  
  const { tag } = await request.json()
  
  revalidateTag(tag)
  
  return Response.json({
    revalidated: true,
    now: Date.now(),
  })
}
```

---

## Step 1794-1796: unstable_cache {#unstable-cache}

```typescript
// unstable_cache ใช้ cache ข้อมูลที่ไม่ใช่ fetch()
import { unstable_cache } from 'next/cache'

// ตัวอย่าง 1: Cache database queries
const getCachedPosts = unstable_cache(
  async (category?: string) => {
    return prisma.post.findMany({
      where: { category, status: 'published' },
      include: { author: true },
      orderBy: { publishedAt: 'desc' },
    })
  },
  ['posts-list'], // Cache key prefix
  {
    tags: ['posts'],
    revalidate: 3600, // 1 hour
  }
)

// ตัวอย่าง 2: Cache external API ที่ใช้ library อื่น
const getCachedWeather = unstable_cache(
  async (city: string) => {
    const weatherLib = new WeatherAPI(process.env.WEATHER_API_KEY!)
    return weatherLib.getWeather(city)
  },
  ['weather'],
  {
    tags: ['weather'],
    revalidate: 300, // 5 minutes
  }
)

// ตัวอย่าง 3: Cache user-specific data
const getCachedUserProfile = unstable_cache(
  async (userId: string) => {
    return prisma.user.findUnique({
      where: { id: userId },
      include: {
        posts: { take: 10 },
        followers: { take: 5 },
      },
    })
  },
  ['user-profile'],
  {
    tags: (userId: string) => [`user-${userId}`],
    revalidate: 60, // 1 minute
  }
)

// การใช้งาน
async function BlogPage() {
  const posts = await getCachedPosts('technology')
  return <PostList posts={posts} />
}

// Revalidate user profile
async function updateUser(userId: string, data: any) {
  await prisma.user.update({ where: { id: userId }, data })
  revalidateTag(`user-${userId}`)
}
```

---

## Step 1797-1800: Incremental Static Regeneration (ISR) {#isr}

ISR ช่วยให้ update static pages ได้โดยไม่ต้อง rebuild ทั้งหมด

### Time-based ISR

```typescript
// app/products/[id]/page.tsx
export const revalidate = 3600 // Revalidate ทุก 1 ชั่วโมง

export async function generateStaticParams() {
  const products = await getTopProducts(100) // สร้าง static pages สำหรับ top 100
  return products.map((p) => ({ id: p.id }))
}

export default async function ProductPage({
  params,
}: {
  params: { id: string }
}) {
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { revalidate: 3600 },
  }).then(r => r.json())
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>฿{product.price}</p>
      <p>อัพเดทล่าสุด: {new Date().toLocaleString('th-TH')}</p>
    </div>
  )
}
```

### On-demand ISR

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

// Webhook handler สำหรับ CMS
export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-webhook-secret')
  
  if (secret !== process.env.WEBHOOK_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const payload = await request.json()
  
  switch (payload.event) {
    case 'post.published':
    case 'post.updated':
      // ล้าง cache ของ post นั้น
      revalidatePath(`/blog/${payload.slug}`)
      revalidatePath('/blog')
      revalidateTag('posts')
      break
    
    case 'post.deleted':
      revalidatePath(`/blog/${payload.slug}`)
      revalidatePath('/blog')
      revalidateTag('posts')
      break
    
    case 'product.updated':
      revalidatePath(`/products/${payload.id}`)
      revalidateTag(`product-${payload.id}`)
      revalidateTag('products')
      break
    
    case 'settings.updated':
      revalidatePath('/', 'layout') // Revalidate ทุกหน้า
      break
    
    default:
      return Response.json({ error: 'Unknown event' }, { status: 400 })
  }
  
  return Response.json({
    revalidated: true,
    event: payload.event,
    timestamp: new Date().toISOString(),
  })
}
```

### ISR กับ Contentful CMS

```typescript
// lib/contentful.ts
import { createClient } from 'contentful'
import { unstable_cache } from 'next/cache'

const client = createClient({
  space: process.env.CONTENTFUL_SPACE_ID!,
  accessToken: process.env.CONTENTFUL_ACCESS_TOKEN!,
})

export const getContentfulPosts = unstable_cache(
  async () => {
    const entries = await client.getEntries({
      content_type: 'blogPost',
      order: ['-fields.publishDate'],
    })
    
    return entries.items.map((item) => ({
      id: item.sys.id,
      title: item.fields.title as string,
      slug: item.fields.slug as string,
      content: item.fields.content as string,
    }))
  },
  ['contentful-posts'],
  {
    tags: ['contentful', 'posts'],
    revalidate: 60, // Backup revalidation
  }
)

// Contentful Webhook → /api/revalidate
// เมื่อ content เปลี่ยน จะ revalidateTag('contentful')
```

### Cache Monitoring

```typescript
// การ debug cache ใน development

// next.config.ts
const config = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}

// Console output:
// GET https://api.example.com/posts 200 in 150ms (cache: MISS)
// GET https://api.example.com/posts 200 in 2ms (cache: HIT)
// GET https://api.example.com/posts 200 in 180ms (cache: STALE, revalidating)
```

---

## 🧪 Quiz - Part 54

**ข้อ 1:** Request Memoization ใน Next.js ต่างจาก Data Cache อย่างไร?
- A) Memoization ถาวรกว่า
- B) Memoization ใช้ memory แค่ในระหว่าง render เดียว Data Cache ข้าม requests
- C) Memoization เร็วกว่า
- D) ไม่มีความแตกต่าง

**ข้อ 2:** `revalidateTag('posts')` จะ invalidate cache อะไรบ้าง?
- A) ทุก pages
- B) เฉพาะ fetch() และ unstable_cache() ที่ใช้ tag 'posts'
- C) เฉพาะ database queries
- D) ทุก cookies

**ข้อ 3:** วิธีทำให้ page เป็น Dynamic โดยออกจาก Full Route Cache คือ?
- A) `export const cache = 'dynamic'`
- B) `export const dynamic = 'force-dynamic'`
- C) `export const static = false`
- D) `export const ssr = true`

**ข้อ 4:** ISR ย่อมาจากอะไร?
- A) Incremental Static Rendering
- B) Integrated Server Revalidation
- C) Incremental Static Regeneration
- D) Instant Static Refresh

**เฉลย:** 1-B, 2-B, 3-B, 4-C

---

> **➡️ Next:** [Part 55: CI/CD กับ GitHub Actions](./part-55-cicd-github-actions.md)
