# Part 34: Next.js Data Fetching

## ข้อมูล Part
- **Steps:** 961-1000
- **ระดับ:** Intermediate to Advanced
- **เวลาเรียน:** 4 ชั่วโมง
- **Prerequisites:** Part 33 (Pages vs App Router)

---

## สารบัญ

1. [Server Components Data Fetching](#1-server-components-data-fetching)
2. [fetch() ใน Server Components](#2-fetch-ใน-server-components)
3. [Caching Strategies](#3-caching-strategies)
4. [revalidatePath, revalidateTag](#4-revalidatepath-revalidatetag)
5. [generateStaticParams (SSG)](#5-generatestaticparams-ssg)
6. [Streaming Data](#6-streaming-data)
7. [Parallel vs Sequential Fetching](#7-parallel-vs-sequential-fetching)
8. [Suspense + loading.tsx](#8-suspense--loadingtsx)
9. [Quiz](#quiz)

---

## Step 961: Server Components Data Fetching

### 1. Server Components Data Fetching

ใน App Router, Data Fetching เกิดขึ้นใน Server Components โดยตรง

#### รูปแบบก่อน App Router

```tsx
// Pages Router - getServerSideProps
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/products')
  const data = await res.json()
  return { props: { products: data } }
}

export default function Products({ products }) {
  // products มาจาก props
  return <ProductList products={products} />
}
```

#### รูปแบบใหม่ด้วย App Router

```tsx
// App Router - Direct fetching ใน Component
export default async function Products() {
  // fetch โดยตรงใน Component
  const res = await fetch('https://api.example.com/products')
  const products = await res.json()
  
  return <ProductList products={products} />
}
```

#### ข้อดีของการ Fetch ใน Server Components

```
1. ไม่มี Client-side JavaScript สำหรับ Data Fetching
2. API Keys ไม่ถูก Expose ไปยัง Client
3. ลด Waterfall (Server กับ Database อยู่ใกล้กัน)
4. สามารถ Query Database โดยตรงได้
5. ข้อมูล Render พร้อมกับ HTML
```

---

## Step 963: fetch() ใน Server Components

### 2. fetch() ใน Server Components

Next.js ขยาย `fetch()` API มาตรฐานให้รองรับ Caching และ Revalidating

#### Syntax พื้นฐาน

```typescript
const response = await fetch(url, {
  // Options มาตรฐาน
  method: 'GET',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data),
  
  // Options พิเศษของ Next.js
  cache: 'force-cache' | 'no-store',
  next: {
    revalidate: number,  // ISR: revalidate ทุก n วินาที
    tags: string[],      // Cache Tags สำหรับ On-demand Revalidation
  }
})
```

#### ตัวอย่างการใช้งาน

```tsx
// 1. Static (SSG) - Cache ตลอดไป
const staticData = await fetch('https://api.example.com/config', {
  cache: 'force-cache'
  // หรือไม่ใส่อะไรเลย (default คือ force-cache)
})

// 2. Dynamic (SSR) - ไม่ Cache เลย
const dynamicData = await fetch('https://api.example.com/user-data', {
  cache: 'no-store'
})

// 3. ISR - Cache แล้ว Revalidate ทุก 1 ชั่วโมง
const isrData = await fetch('https://api.example.com/products', {
  next: { revalidate: 3600 }
})

// 4. Tag-based Caching
const taggedData = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] }
})
```

#### Auto Deduplication

Next.js Deduplicate Request ที่เหมือนกันอัตโนมัติ:

```tsx
// lib/data.ts
export async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`)
  return res.json()
}

// app/dashboard/page.tsx
export default async function Dashboard() {
  const user = await getUser('123')    // Request 1
  return <div>{user.name}</div>
}

// app/dashboard/layout.tsx
export default async function DashboardLayout({ children }) {
  const user = await getUser('123')    // Request 2 - ถูก Deduplicate!
  return (
    <div>
      <Sidebar username={user.name} />
      {children}
    </div>
  )
}

// จะมีการ Fetch จริงแค่ครั้งเดียว เพราะ URL เหมือนกัน
```

#### fetch() กับ Database

```tsx
// lib/db.ts
import { prisma } from './prisma'

export async function getProducts() {
  // ไม่จำเป็นต้องใช้ fetch() ถ้า Query Database โดยตรง
  return prisma.product.findMany({
    include: { category: true }
  })
}

// ถ้าต้องการ Cache Manual
import { unstable_cache } from 'next/cache'

export const getCachedProducts = unstable_cache(
  async () => {
    return prisma.product.findMany()
  },
  ['products'],  // Cache Key
  {
    tags: ['products'],
    revalidate: 3600,
  }
)
```

---

## Step 966: Caching Strategies

### 3. Caching Strategies

Next.js มีหลายระดับของ Caching:

#### Caching Layers

```
┌─────────────────────────────────────────┐
│              Browser Cache              │
├─────────────────────────────────────────┤
│              CDN / Edge Cache            │
├─────────────────────────────────────────┤
│        Next.js Full Route Cache          │
│      (Cached HTML + RSC Payload)         │
├─────────────────────────────────────────┤
│         Next.js Data Cache              │
│          (fetch() Results)              │
├─────────────────────────────────────────┤
│          Request Memoization            │
│       (Per-request Deduplication)       │
└─────────────────────────────────────────┘
```

#### 1. Request Memoization

```tsx
// Automatic - เกิดขึ้นใน Single Request
// หากเรียก fetch URL เดิมหลายครั้งใน Request เดียว
// Next.js จะ Fetch จริงแค่ครั้งเดียว

async function getUser(id: string) {
  const res = await fetch(`/api/users/${id}`)
  return res.json()
}

export default async function Page({ params }) {
  // ทั้งสองนี้จะ Request เพียงครั้งเดียว
  const user1 = await getUser(params.id)
  const user2 = await getUser(params.id) // ได้จาก Memoization Cache
}
```

#### 2. Data Cache

```tsx
// Persists across Requests และ Deployments
// ควบคุมด้วย cache option ของ fetch()

// force-cache (default): ใช้ Cache ถ้ามี
const data = await fetch(url, { cache: 'force-cache' })

// no-store: ไม่ Cache เลย
const data = await fetch(url, { cache: 'no-store' })

// revalidate: Cache แต่ Revalidate เป็นระยะ
const data = await fetch(url, { next: { revalidate: 3600 } })
```

#### 3. Full Route Cache

```tsx
// Next.js Cache ทั้ง Route (HTML + RSC Payload)
// เกิดขึ้นอัตโนมัติสำหรับ Static Routes ตอน Build Time

// opt out: ถ้า Route มี Dynamic Functions
// - cookies(), headers()
// - searchParams
// - fetch() with no-store

// app/static-page/page.tsx
// Page นี้จะถูก Cache ทั้ง Route
export default async function StaticPage() {
  const data = await fetch(url, { next: { revalidate: 86400 } })
  return <div>{data.title}</div>
}

// app/dynamic-page/page.tsx
// Page นี้จะไม่ Cache (เพราะใช้ cookies)
import { cookies } from 'next/headers'

export default async function DynamicPage() {
  const user = cookies().get('user')  // ทำให้ Dynamic
  return <div>Hello {user?.value}</div>
}
```

#### 4. Router Cache (Client-side)

```tsx
// Next.js Cache RSC Payload ใน Browser Memory
// ทำให้ Navigation เร็วขึ้น
// Clear อัตโนมัติหลัง 30 วินาที (Dynamic) หรือ 5 นาที (Static)

// ล้าง Router Cache ด้วย router.refresh()
'use client'
import { useRouter } from 'next/navigation'

function RefreshButton() {
  const router = useRouter()
  return (
    <button onClick={() => router.refresh()}>
      Refresh Data
    </button>
  )
}
```

---

## Step 970: revalidatePath, revalidateTag

### 4. revalidatePath, revalidateTag

On-demand Revalidation ช่วยให้ Clear Cache เมื่อข้อมูลเปลี่ยน

#### revalidatePath

```tsx
// app/actions/posts.ts
'use server'
import { revalidatePath } from 'next/cache'

export async function createPost(data: FormData) {
  const title = data.get('title') as string
  const content = data.get('content') as string
  
  // สร้าง Post ใน Database
  await db.post.create({ data: { title, content } })
  
  // Clear Cache สำหรับ Path นี้
  revalidatePath('/blog')              // Clear /blog
  revalidatePath('/blog/[slug]', 'page') // Clear ทุก /blog/* pages
  revalidatePath('/', 'layout')        // Clear Root Layout Cache
}

export async function updatePost(id: string, data: FormData) {
  const post = await db.post.update({
    where: { id },
    data: { title: data.get('title') as string }
  })
  
  revalidatePath(`/blog/${post.slug}`)  // Clear เฉพาะ Post นี้
  revalidatePath('/blog')               // Clear Blog List ด้วย
}

export async function deletePost(id: string) {
  const post = await db.post.delete({ where: { id } })
  
  revalidatePath('/blog')
  revalidatePath(`/blog/${post.slug}`)
}
```

#### revalidateTag

```tsx
// กำหนด Tag ตอน Fetch
const posts = await fetch('/api/posts', {
  next: { tags: ['posts'] }
})

const post = await fetch(`/api/posts/${id}`, {
  next: { tags: ['posts', `post-${id}`] }
})

// app/actions/posts.ts
'use server'
import { revalidateTag } from 'next/cache'

export async function createPost(data: FormData) {
  await db.post.create({ data: { ... } })
  
  revalidateTag('posts')       // Clear ทุก fetch ที่มี tag 'posts'
}

export async function updatePost(id: string, data: FormData) {
  await db.post.update({ where: { id }, data: { ... } })
  
  revalidateTag(`post-${id}`) // Clear เฉพาะ post นี้
  revalidateTag('posts')       // Clear list ด้วย
}
```

#### On-demand Revalidation via API Route

```typescript
// app/api/revalidate/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath, revalidateTag } from 'next/cache'

// Webhook จาก CMS
export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-webhook-secret')
  
  // ตรวจสอบ Secret
  if (secret !== process.env.WEBHOOK_SECRET) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const body = await request.json()
  const { type, slug } = body
  
  if (type === 'post.created' || type === 'post.updated') {
    revalidatePath('/blog')
    revalidatePath(`/blog/${slug}`)
    revalidateTag('posts')
    
    return NextResponse.json({ revalidated: true, type, slug })
  }
  
  return NextResponse.json({ error: 'Unknown event type' }, { status: 400 })
}
```

---

## Step 974: generateStaticParams (SSG)

### 5. generateStaticParams (SSG)

`generateStaticParams` ใช้สร้าง Static Paths สำหรับ Dynamic Routes ตอน Build Time

#### Basic Usage

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache'
  }).then(r => r.json())
  
  return posts.map(post => ({
    slug: post.slug,
  }))
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return <article>{post.title}</article>
}
```

#### Nested Dynamic Routes

```tsx
// app/[category]/[product]/page.tsx

export async function generateStaticParams() {
  const categories = await getCategories()
  
  // Generate params สำหรับทุก Category/Product combination
  const params = []
  
  for (const category of categories) {
    const products = await getProducts(category.slug)
    
    for (const product of products) {
      params.push({
        category: category.slug,
        product: product.slug,
      })
    }
  }
  
  return params
}
```

#### ใช้ Parent Params

```tsx
// app/[category]/[product]/page.tsx
export async function generateStaticParams({
  params: { category },
}: {
  params: { category: string }
}) {
  // ได้รับ category จาก Parent generateStaticParams
  const products = await getProducts(category)
  
  return products.map(product => ({
    product: product.slug,
  }))
}
```

#### Dynamic Segments กับ fallback

```tsx
// app/blog/[slug]/page.tsx

// Option 1: Pre-generate ทุก Path
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(p => ({ slug: p.slug }))
}

// Option 2: Lazy Generate (Dynamically render ครั้งแรก แล้ว Cache)
// next.config.js
module.exports = {
  // ไม่มี generateStaticParams = Dynamic Rendering
}

// app/blog/[slug]/page.tsx (no generateStaticParams)
// จะ Render แบบ SSR และ Cache ผล

// Option 3: partial pre-generation + dynamic
export async function generateStaticParams() {
  // สร้างแค่ Posts ล่าสุด 10 อัน
  const posts = await getRecentPosts(10)
  return posts.map(p => ({ slug: p.slug }))
}

// Posts อื่นๆ จะถูก Render แบบ Dynamic เมื่อถูกเรียก
// แล้ว Cache ไว้
```

---

## Step 978: Streaming Data

### 6. Streaming Data

Streaming ช่วยส่ง HTML ให้ Browser ทีละส่วนแทนที่จะรอทั้งหมด

#### ทำงานอย่างไร

```
ไม่มี Streaming:
Server รอทุก Data ---> ส่ง HTML ทั้งหมด ---> Browser แสดง

มี Streaming:
Server ส่ง Shell (Layout) --> Browser แสดง Shell
Server Fetch Data 1      --> Browser แสดง Data 1
Server Fetch Data 2      --> Browser แสดง Data 2
```

#### Streaming ด้วย Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react'

async function UserStats({ userId }: { userId: string }) {
  // นี่อาจใช้เวลานาน
  const stats = await getUserStats(userId)
  return <StatsChart data={stats} />
}

async function RecentOrders({ userId }: { userId: string }) {
  const orders = await getRecentOrders(userId)
  return <OrderTable orders={orders} />
}

async function Recommendations() {
  const recs = await getRecommendations()
  return <RecList items={recs} />
}

export default function Dashboard() {
  return (
    <div className="grid grid-cols-2 gap-4">
      {/* Stats - รอ Data นาน */}
      <Suspense fallback={<StatsSkeleton />}>
        <UserStats userId="123" />
      </Suspense>
      
      {/* Orders - รอ Data นาน */}
      <Suspense fallback={<OrderSkeleton />}>
        <RecentOrders userId="123" />
      </Suspense>
      
      {/* Recommendations - อาจช้า */}
      <Suspense fallback={<RecSkeleton />}>
        <Recommendations />
      </Suspense>
    </div>
  )
}
```

#### loading.tsx คือ Streaming Built-in

```tsx
// app/dashboard/loading.tsx
// Next.js Wrap page.tsx ด้วย Suspense อัตโนมัติ
// โดยใช้ loading.tsx เป็น fallback

export default function DashboardLoading() {
  return (
    <div className="grid grid-cols-2 gap-4 animate-pulse">
      <div className="h-48 bg-gray-200 rounded-lg"></div>
      <div className="h-48 bg-gray-200 rounded-lg"></div>
    </div>
  )
}
```

#### Streaming ด้วย generateMetadata

```tsx
// app/blog/[slug]/page.tsx
// Metadata ถูก Resolve ก่อน Page Content
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    description: post.excerpt,
  }
}

// Page Component จะ Stream
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return (
    <article>
      <h1>{post.title}</h1>
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={post.id} />
      </Suspense>
    </article>
  )
}
```

---

## Step 982: Parallel vs Sequential Fetching

### 7. Parallel vs Sequential Fetching

#### Sequential Fetching (ช้า)

```tsx
// ❌ Sequential - รอ User ก่อน แล้วค่อย Fetch Posts
async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId)        // รอ 200ms
  const posts = await getUserPosts(userId)  // รออีก 300ms
  // รวม: 500ms
  
  return (
    <div>
      <UserCard user={user} />
      <PostList posts={posts} />
    </div>
  )
}
```

#### Parallel Fetching (เร็ว)

```tsx
// ✅ Parallel - Fetch พร้อมกัน
async function UserProfile({ userId }: { userId: string }) {
  // Start ทั้งคู่พร้อมกัน
  const [user, posts] = await Promise.all([
    getUser(userId),       // เริ่มพร้อมกัน
    getUserPosts(userId),  // เริ่มพร้อมกัน
  ])
  // รวม: max(200ms, 300ms) = 300ms
  
  return (
    <div>
      <UserCard user={user} />
      <PostList posts={posts} />
    </div>
  )
}
```

#### Parallel กับ Independent Components

```tsx
// ✅ แบบที่ดีที่สุด - แต่ละ Component Fetch เอง
// Fetch เกิดพร้อมกันเพราะ Streaming

export default function Dashboard() {
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserSection />   {/* Fetch user data */}
      </Suspense>
      
      <Suspense fallback={<PostSkeleton />}>
        <PostSection />   {/* Fetch post data พร้อมกัน */}
      </Suspense>
      
      <Suspense fallback={<StatsSkeleton />}>
        <StatsSection />  {/* Fetch stats พร้อมกัน */}
      </Suspense>
    </div>
  )
}

// แต่ละ Component Fetch ข้อมูลของตัวเอง
async function UserSection() {
  const user = await getUser()
  return <UserCard user={user} />
}

async function PostSection() {
  const posts = await getPosts()
  return <PostList posts={posts} />
}
```

#### Promise.allSettled สำหรับ Error Handling

```tsx
async function Dashboard() {
  const [userResult, postsResult, statsResult] = await Promise.allSettled([
    getUser(),
    getPosts(),
    getStats(),
  ])
  
  const user = userResult.status === 'fulfilled' ? userResult.value : null
  const posts = postsResult.status === 'fulfilled' ? postsResult.value : []
  const stats = statsResult.status === 'fulfilled' ? statsResult.value : null
  
  return (
    <div>
      {user ? <UserCard user={user} /> : <UserError />}
      {posts.length > 0 ? <PostList posts={posts} /> : <NoPosts />}
      {stats ? <StatsChart data={stats} /> : <StatsError />}
    </div>
  )
}
```

#### เมื่อไหรใช้ Sequential

```tsx
// Sequential เป็นสิ่งที่ถูกต้องเมื่อข้อมูลขึ้นอยู่กันกัน

async function ProductWithReviews({ productId }: { productId: string }) {
  // ต้องได้ Product ก่อนถึงจะรู้ว่า Reviews ของ Product ไหน
  const product = await getProduct(productId)
  
  // ขึ้นอยู่กับ product ที่ได้มา
  const category = await getCategory(product.categoryId)
  const relatedProducts = await getRelated(product.id, category.id)
  
  return <div>...</div>
}
```

---

## Step 987: Suspense + loading.tsx

### 8. Suspense + loading.tsx

#### Suspense Boundaries

```tsx
// app/page.tsx
import { Suspense } from 'react'

// Server Component ที่ใช้เวลานาน
async function SlowComponent() {
  await new Promise(resolve => setTimeout(resolve, 2000))
  const data = await fetchData()
  return <div>{data.title}</div>
}

// Fast Component
async function FastComponent() {
  const data = await fetchFastData()
  return <div>{data.title}</div>
}

export default function Page() {
  return (
    <div>
      {/* แสดงทันที */}
      <h1>Dashboard</h1>
      
      {/* รอ FastComponent แต่ SlowComponent แยกกัน */}
      <Suspense fallback={<p>Loading fast data...</p>}>
        <FastComponent />
      </Suspense>
      
      <Suspense fallback={<p>Loading slow data...</p>}>
        <SlowComponent />
      </Suspense>
    </div>
  )
}
```

#### Skeleton Loading Pattern

```tsx
// components/skeletons/PostSkeleton.tsx
export function PostSkeleton() {
  return (
    <div className="animate-pulse space-y-4">
      <div className="h-8 bg-gray-200 rounded w-3/4"></div>
      <div className="h-4 bg-gray-200 rounded w-1/4"></div>
      <div className="space-y-2">
        <div className="h-4 bg-gray-200 rounded"></div>
        <div className="h-4 bg-gray-200 rounded"></div>
        <div className="h-4 bg-gray-200 rounded w-5/6"></div>
      </div>
    </div>
  )
}

// components/skeletons/PostListSkeleton.tsx
export function PostListSkeleton() {
  return (
    <div className="space-y-8">
      {[1, 2, 3].map(i => (
        <PostSkeleton key={i} />
      ))}
    </div>
  )
}

// app/blog/page.tsx
import { Suspense } from 'react'
import { PostListSkeleton } from '@/components/skeletons'

async function BlogPosts() {
  const posts = await getPosts()
  return <PostList posts={posts} />
}

export default function BlogPage() {
  return (
    <div>
      <h1>Blog</h1>
      <Suspense fallback={<PostListSkeleton />}>
        <BlogPosts />
      </Suspense>
    </div>
  )
}
```

#### ตัวอย่าง E-commerce Page กับ Streaming

```tsx
// app/products/[id]/page.tsx
import { Suspense } from 'react'
import { notFound } from 'next/navigation'

// Critical Data (Fast)
async function ProductDetails({ id }: { id: string }) {
  const product = await getProduct(id)
  if (!product) notFound()
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p className="text-2xl font-bold">{product.price} บาท</p>
      <p>{product.description}</p>
    </div>
  )
}

// Reviews (Slow)
async function ProductReviews({ id }: { id: string }) {
  const reviews = await getProductReviews(id)
  return (
    <div>
      <h2>รีวิวสินค้า ({reviews.length})</h2>
      {reviews.map(review => (
        <ReviewCard key={review.id} review={review} />
      ))}
    </div>
  )
}

// Related Products (Slow)
async function RelatedProducts({ categoryId }: { categoryId: string }) {
  const related = await getRelatedProducts(categoryId)
  return (
    <div>
      <h2>สินค้าที่เกี่ยวข้อง</h2>
      <ProductGrid products={related} />
    </div>
  )
}

export default function ProductPage({ params }) {
  return (
    <div>
      {/* แสดงทันที */}
      <Suspense fallback={<ProductDetailsSkeleton />}>
        <ProductDetails id={params.id} />
      </Suspense>
      
      {/* โหลดทีหลัง ไม่ block หน้าหลัก */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews id={params.id} />
      </Suspense>
      
      <Suspense fallback={<RelatedSkeleton />}>
        <RelatedProducts categoryId="electronics" />
      </Suspense>
    </div>
  )
}
```

---

## Step 992: ตัวอย่างสมบูรณ์ - News Site

### ตัวอย่างสมบูรณ์: News Site

```tsx
// lib/news.ts
interface Article {
  id: string
  slug: string
  title: string
  excerpt: string
  content: string
  category: string
  publishedAt: string
  author: { name: string; avatar: string }
  image: string
}

export async function getArticles(options?: {
  category?: string
  limit?: number
}) {
  const params = new URLSearchParams()
  if (options?.category) params.set('category', options.category)
  if (options?.limit) params.set('limit', String(options.limit))
  
  const res = await fetch(
    `https://api.news.example.com/articles?${params}`,
    {
      next: { 
        tags: ['articles', options?.category ? `category-${options.category}` : ''],
        revalidate: 300  // ISR: 5 นาที
      }
    }
  )
  
  if (!res.ok) throw new Error('Failed to fetch articles')
  return res.json() as Promise<Article[]>
}

export async function getArticle(slug: string) {
  const res = await fetch(
    `https://api.news.example.com/articles/${slug}`,
    {
      next: {
        tags: ['articles', `article-${slug}`],
        revalidate: 300
      }
    }
  )
  
  if (res.status === 404) return null
  if (!res.ok) throw new Error('Failed to fetch article')
  return res.json() as Promise<Article>
}

// app/news/page.tsx
import { Suspense } from 'react'
import { getArticles } from '@/lib/news'

async function FeaturedArticle() {
  const [featured] = await getArticles({ limit: 1 })
  return (
    <div className="relative h-96 rounded-xl overflow-hidden">
      <img src={featured.image} alt={featured.title} className="w-full h-full object-cover" />
      <div className="absolute bottom-0 p-6 bg-gradient-to-t from-black text-white">
        <span className="badge">{featured.category}</span>
        <h2 className="text-2xl font-bold mt-2">{featured.title}</h2>
        <p className="text-gray-300 mt-1">{featured.excerpt}</p>
      </div>
    </div>
  )
}

async function CategoryNews({ category }: { category: string }) {
  const articles = await getArticles({ category, limit: 5 })
  return (
    <section>
      <h2 className="text-xl font-bold mb-4 capitalize">{category}</h2>
      <div className="space-y-4">
        {articles.map(article => (
          <ArticleCard key={article.id} article={article} />
        ))}
      </div>
    </section>
  )
}

export default function NewsPage() {
  const categories = ['technology', 'business', 'sports']
  
  return (
    <div className="container mx-auto py-8">
      <Suspense fallback={<FeaturedSkeleton />}>
        <FeaturedArticle />
      </Suspense>
      
      <div className="grid grid-cols-3 gap-8 mt-8">
        {categories.map(category => (
          <Suspense key={category} fallback={<CategorySkeleton />}>
            <CategoryNews category={category} />
          </Suspense>
        ))}
      </div>
    </div>
  )
}
```

---

## Step 996: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Parallel Fetching เมื่อทำได้

✓ ใช้ Promise.all() สำหรับ Independent Data
✓ ใช้ Suspense Boundaries เพื่อ Parallel Stream
✗ อย่า await ทีละตัวถ้าไม่จำเป็น

## 2. Cache Tags

✓ ตั้ง Tags ที่มีความหมาย เช่น 'posts', 'post-123'
✓ ใช้ revalidateTag เมื่อ Data เปลี่ยน
✓ จัดกลุ่ม Tags ตาม Entity

## 3. Streaming สำหรับ Better UX

✓ แยก Suspense Boundaries สำหรับ Independent Sections
✓ ใช้ Skeleton Loading แทน Spinner
✓ แสดง Critical Content ก่อน

## 4. generateStaticParams

✓ ใช้สำหรับ Known Dynamic Routes (Blog, Product)
✓ ไม่ต้องสร้างทุก Path - Lazy Generation ก็ได้
✓ ใช้ revalidate แทน Static ถ้า Data เปลี่ยนบ่อย

## 5. Error Handling

✓ ตรวจสอบ Response Status ใน fetch()
✓ ใช้ error.tsx สำหรับ UI
✓ Log Errors ไปยัง Monitoring Service
```

---

## Quiz

### แบบทดสอบ Part 34

**คำถามที่ 1:** `fetch(url, { next: { revalidate: 60 } })` คือ Rendering Strategy อะไร?
- A) SSG
- B) SSR
- C) ISR ✓
- D) CSR

**คำถามที่ 2:** `revalidateTag('posts')` ทำอะไร?
- A) Delete ทุก Post ใน Database
- B) Clear Cache ของทุก fetch() ที่มี tag 'posts' ✓
- C) Refresh หน้า Blog
- D) Rebuild ทั้ง Site

**คำถามที่ 3:** Request Memoization ทำงานอย่างไร?
- A) Cache Data ข้ามหลาย Requests
- B) Deduplicate fetch() URL เดิมใน Single Request ✓
- C) Cache Browser Response
- D) Block Duplicate Requests

**คำถามที่ 4:** เมื่อไหรควรใช้ Sequential Fetching แทน Parallel?
- A) เสมอ
- B) เมื่อ Data ชุดที่สองขึ้นอยู่กับผลจากชุดแรก ✓
- C) เมื่อต้องการ Performance ดี
- D) ไม่ควรใช้

**คำถามที่ 5:** `generateStaticParams` ใช้ทำอะไร?
- A) กำหนด URL Parameters สำหรับ API Routes
- B) สร้าง Static Paths สำหรับ Dynamic Routes ตอน Build Time ✓
- C) Cache ค่า Parameters
- D) Validate URL Parameters

---

## สรุป Part 34

ใน Part นี้เราได้เรียนรู้:

1. **Server Components Data Fetching** - fetch โดยตรงใน Component
2. **fetch() Options** - force-cache, no-store, revalidate, tags
3. **Caching Layers** - Request Memo, Data Cache, Full Route Cache, Router Cache
4. **revalidatePath/revalidateTag** - On-demand Revalidation
5. **generateStaticParams** - SSG สำหรับ Dynamic Routes
6. **Streaming** - ส่ง HTML ทีละส่วนด้วย Suspense
7. **Parallel vs Sequential** - Fetch พร้อมกันเมื่อทำได้

---

➡️ **Part ถัดไป:** [Part 35: Next.js API Routes (Route Handlers)](./part-35-nextjs-api-routes.md)
