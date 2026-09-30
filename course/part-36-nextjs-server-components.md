# Part 36: Next.js Server Components

## ข้อมูล Part
- **Steps:** 1036-1075
- **ระดับ:** Intermediate to Advanced
- **เวลาเรียน:** 3-4 ชั่วโมง
- **Prerequisites:** Part 35 (API Routes)

---

## สารบัญ

1. [Server Components คืออะไร](#1-server-components-คืออะไร)
2. [ข้อดีของ Server Components](#2-ข้อดีของ-server-components)
3. [สิ่งที่ทำไม่ได้ใน Server Components](#3-สิ่งที่ทำไม่ได้ใน-server-components)
4. [Data Fetching ใน Server Components](#4-data-fetching-ใน-server-components)
5. [Server Components + Database](#5-server-components--database)
6. [Streaming ด้วย Suspense](#6-streaming-ด้วย-suspense)
7. [ตัวอย่าง Blog Page](#7-ตัวอย่าง-blog-page)
8. [Quiz](#quiz)

---

## Step 1036: Server Components คืออะไร

### 1. Server Components คืออะไร

Server Components คือ React Components ที่ Render ที่ฝั่ง Server เท่านั้น โดยไม่ส่ง JavaScript ไปยัง Client

#### ก่อน Server Components

```
Client Browser
│
│── Download React + App JS Bundle (100KB-2MB)
│── React Render Components
│── Fetch Data จาก API
│── Re-render ด้วย Data
│
└── แสดงผลให้ผู้ใช้
```

#### หลัง Server Components

```
Server
│
│── React Render Server Components
│── Fetch Data โดยตรง (DB, API, Filesystem)
│── สร้าง HTML + RSC Payload
│
└── ส่งให้ Browser (HTML เล็กกว่า, JS น้อยกว่า)

Client Browser
│
│── รับ HTML → แสดงผลทันที
│── Download เฉพาะ Client Components JS
│── Hydrate Client Components
│
└── Interactive UI พร้อมใช้งาน
```

#### ทุก Component ใน App Router เป็น Server Component โดย Default

```tsx
// app/page.tsx
// ไม่มี 'use client' → Server Component

export default async function Page() {
  // สามารถ await ได้โดยตรง
  const data = await fetch('https://api.example.com/data')
  const json = await data.json()
  
  // สามารถ Query Database ได้โดยตรง
  const users = await prisma.user.findMany()
  
  return (
    <div>
      {json.title}
      {users.map(u => <p key={u.id}>{u.name}</p>)}
    </div>
  )
}
```

#### React Server Components (RSC) vs Server-Side Rendering (SSR)

```
SSR (ก่อน): 
  Server Render HTML → ส่งไป Browser → Hydrate ทั้งหมด

RSC:
  Server Render HTML + RSC Payload → ส่งไป Browser
  Browser Hydrate เฉพาะ Client Components
  Server Components = Zero Client JS
```

---

## Step 1038: ข้อดีของ Server Components

### 2. ข้อดีของ Server Components

#### 1. JavaScript Bundle เล็กลง

```tsx
// ❌ Client Component: Heavy Library ใน Bundle
'use client'
import { marked } from 'marked'    // ~50KB
import { highlight } from 'prismjs' // ~30KB

function BlogPost({ content }) {
  const html = marked(content)
  return <div dangerouslySetInnerHTML={{ __html: html }} />
}

// ✅ Server Component: Library ไม่อยู่ใน Bundle
// ไม่มี 'use client'
import { marked } from 'marked'    // ไม่อยู่ใน Client Bundle
import { highlight } from 'prismjs' // ไม่อยู่ใน Client Bundle

async function BlogPost({ content }) {
  const html = marked(content)
  return <div dangerouslySetInnerHTML={{ __html: html }} />
}
```

#### 2. Direct Database Access

```tsx
// Server Component - Query Database โดยตรง
import { prisma } from '@/lib/prisma'

async function UserList() {
  const users = await prisma.user.findMany({
    select: { id: true, name: true, email: true },
    orderBy: { name: 'asc' }
  })
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name} - {user.email}</li>
      ))}
    </ul>
  )
}
```

#### 3. ไม่ Expose API Keys

```tsx
// Server Component - API Key ปลอดภัย
async function SecureApiData() {
  const res = await fetch('https://api.example.com/data', {
    headers: {
      'API-Key': process.env.SECRET_API_KEY, // ไม่ถูกส่งไป Client
    }
  })
  const data = await res.json()
  return <div>{data.sensitiveInfo}</div>
}
```

#### 4. Better Performance

```tsx
// Server Component ทำงานที่ Server
// ซึ่งอยู่ใกล้กับ Database มากกว่า

// Client → API Call → Server → Database: 200ms
// Server Component → Database: 5ms

async function ProductList() {
  // เร็วกว่า Client-side fetch
  const products = await prisma.product.findMany()
  return <div>...</div>
}
```

#### 5. Automatic Code Splitting

```tsx
// Client Components ถูก Code Split อัตโนมัติ
// Server Components ไม่อยู่ใน JS Bundle เลย

// Route /blog ส่งเฉพาะ:
// - HTML ที่ Render แล้ว
// - Interactive Client Components เท่านั้น
```

---

## Step 1042: สิ่งที่ทำไม่ได้ใน Server Components

### 3. สิ่งที่ทำไม่ได้ใน Server Components

```typescript
// ❌ ไม่สามารถใช้ React Hooks ที่ใช้ State/Effect
import { useState, useEffect } from 'react'

export default function ServerComponent() {
  const [count, setCount] = useState(0)  // ❌ Error!
  useEffect(() => {}, [])                // ❌ Error!
  return <div>{count}</div>
}

// ❌ ไม่สามารถใช้ Event Handlers
export default function ServerComponent() {
  return (
    <button onClick={() => alert('hi')}>  {/* ❌ Error! */}
      Click me
    </button>
  )
}

// ❌ ไม่สามารถใช้ Browser APIs
export default function ServerComponent() {
  const width = window.innerWidth  // ❌ Error!
  return <div>{width}</div>
}

// ❌ ไม่สามารถใช้ Context ที่ใช้ state
import { useMyContext } from './context'  // ❌ ถ้า context ใช้ useState

// ❌ ไม่สามารถใช้ Class Components
export default class ServerComponent extends React.Component {
  // ❌ ไม่ได้รับการรองรับ
}
```

#### สิ่งที่ Server Components ทำได้

```typescript
// ✅ async/await
export default async function ServerComponent() {
  const data = await fetch(url)
  return <div>{data}</div>
}

// ✅ Database Queries
export default async function ServerComponent() {
  const users = await prisma.user.findMany()
  return <div>...</div>
}

// ✅ File System Access
import fs from 'fs'

export default async function ServerComponent() {
  const content = fs.readFileSync('data.json', 'utf8')
  return <div>{content}</div>
}

// ✅ Environment Variables (ทุกตัว ไม่ต้อง NEXT_PUBLIC_)
export default function ServerComponent() {
  const apiUrl = process.env.API_URL  // ✅ ปลอดภัย
  return <div>API: {apiUrl}</div>
}

// ✅ Render Client Components
export default function ServerComponent() {
  return (
    <div>
      <ClientButton />  {/* ✅ ได้ */}
    </div>
  )
}
```

---

## Step 1046: Data Fetching ใน Server Components

### 4. Data Fetching ใน Server Components

#### fetch() ใน Server Components

```tsx
// app/blog/page.tsx
async function BlogPage() {
  // Static Fetch (SSG)
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache'
  }).then(r => r.json())
  
  // Dynamic Fetch (SSR)
  const trending = await fetch('https://api.example.com/trending', {
    cache: 'no-store'
  }).then(r => r.json())
  
  // ISR
  const featured = await fetch('https://api.example.com/featured', {
    next: { revalidate: 3600 }
  }).then(r => r.json())
  
  return (
    <div>
      <FeaturedPosts posts={featured} />
      <TrendingPosts posts={trending} />
      <BlogList posts={posts} />
    </div>
  )
}
```

#### Data Fetching กับ Error Handling

```tsx
// lib/api.ts
export async function safeFetch<T>(
  url: string,
  options?: RequestInit
): Promise<{ data: T | null; error: string | null }> {
  try {
    const response = await fetch(url, options)
    
    if (!response.ok) {
      return {
        data: null,
        error: `HTTP Error: ${response.status} ${response.statusText}`
      }
    }
    
    const data = await response.json() as T
    return { data, error: null }
  } catch (err) {
    return {
      data: null,
      error: err instanceof Error ? err.message : 'Unknown error'
    }
  }
}

// app/products/page.tsx
async function ProductsPage() {
  const { data: products, error } = await safeFetch<Product[]>(
    'https://api.example.com/products'
  )
  
  if (error) {
    return <ErrorMessage message={error} />
  }
  
  if (!products || products.length === 0) {
    return <EmptyState />
  }
  
  return <ProductGrid products={products} />
}
```

#### Parallel Fetching

```tsx
// app/dashboard/page.tsx
async function DashboardPage() {
  // Fetch ทุกอย่างพร้อมกัน
  const [
    userStats,
    recentOrders,
    notifications,
    recommendations
  ] = await Promise.all([
    getUserStats(),
    getRecentOrders(5),
    getNotifications(),
    getRecommendations(),
  ])
  
  return (
    <div className="grid grid-cols-2 gap-6">
      <StatsPanel data={userStats} />
      <OrdersPanel orders={recentOrders} />
      <NotificationsPanel items={notifications} />
      <RecommendationsPanel items={recommendations} />
    </div>
  )
}
```

---

## Step 1050: Server Components + Database

### 5. Server Components + Database

#### Direct Database Access

```tsx
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : [],
  })

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}

// app/users/page.tsx
import { prisma } from '@/lib/prisma'

async function UsersPage() {
  const users = await prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true,
      _count: {
        select: { posts: true }
      }
    },
    orderBy: { createdAt: 'desc' }
  })
  
  return (
    <div>
      <h1>Users ({users.length})</h1>
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Posts</th>
            <th>Joined</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user._count.posts}</td>
              <td>{user.createdAt.toLocaleDateString('th-TH')}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

#### Data Access Layer Pattern

```typescript
// lib/dal/users.ts (Data Access Layer)
import { prisma } from '@/lib/prisma'
import { cache } from 'react'

// cache() Memoize การเรียกใน Single Request
export const getUser = cache(async (id: string) => {
  return prisma.user.findUnique({
    where: { id },
    include: {
      posts: { orderBy: { createdAt: 'desc' }, take: 5 },
      _count: { select: { posts: true, followers: true } }
    }
  })
})

export const getUserByEmail = cache(async (email: string) => {
  return prisma.user.findUnique({
    where: { email }
  })
})

export async function getUsers(options?: {
  page?: number
  limit?: number
  search?: string
}) {
  const page = options?.page ?? 1
  const limit = options?.limit ?? 10
  
  const where = options?.search ? {
    OR: [
      { name: { contains: options.search, mode: 'insensitive' as const } },
      { email: { contains: options.search, mode: 'insensitive' as const } },
    ]
  } : {}
  
  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { name: 'asc' },
    }),
    prisma.user.count({ where }),
  ])
  
  return {
    users,
    pagination: { page, limit, total, totalPages: Math.ceil(total / limit) }
  }
}

// app/users/page.tsx
import { getUsers } from '@/lib/dal/users'

async function UsersPage({ searchParams }) {
  const { users, pagination } = await getUsers({
    page: Number(searchParams.page) || 1,
    search: searchParams.search,
  })
  
  return (
    <div>
      <UserList users={users} />
      <Pagination {...pagination} />
    </div>
  )
}
```

#### Server Component กับ Auth

```tsx
// lib/auth.ts
import { cookies } from 'next/headers'
import { jwtVerify } from 'jose'
import { prisma } from './prisma'

export async function getCurrentUser() {
  const cookieStore = cookies()
  const token = cookieStore.get('auth-token')?.value
  
  if (!token) return null
  
  try {
    const secret = new TextEncoder().encode(process.env.JWT_SECRET!)
    const { payload } = await jwtVerify(token, secret)
    
    const user = await prisma.user.findUnique({
      where: { id: payload.sub as string },
      select: { id: true, name: true, email: true, role: true }
    })
    
    return user
  } catch {
    return null
  }
}

// app/profile/page.tsx
import { getCurrentUser } from '@/lib/auth'
import { redirect } from 'next/navigation'

async function ProfilePage() {
  const user = await getCurrentUser()
  
  if (!user) {
    redirect('/login')
  }
  
  const userPosts = await prisma.post.findMany({
    where: { authorId: user.id },
    orderBy: { createdAt: 'desc' }
  })
  
  return (
    <div>
      <h1>สวัสดี, {user.name}!</h1>
      <h2>บทความของคุณ</h2>
      <PostList posts={userPosts} />
    </div>
  )
}
```

---

## Step 1055: Streaming ด้วย Suspense

### 6. Streaming ด้วย Suspense

#### Suspense Boundary

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react'
import { prisma } from '@/lib/prisma'

// Component ที่ใช้เวลานาน
async function ExpensiveAnalytics() {
  // จำลองการ Query ที่ใช้เวลานาน
  const analytics = await prisma.$queryRaw`
    SELECT 
      DATE_TRUNC('day', created_at) as date,
      COUNT(*) as count
    FROM orders
    WHERE created_at > NOW() - INTERVAL '30 days'
    GROUP BY DATE_TRUNC('day', created_at)
    ORDER BY date
  `
  
  return <AnalyticsChart data={analytics} />
}

// Skeleton Component
function AnalyticsSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-6 bg-gray-200 rounded w-32 mb-4"></div>
      <div className="h-64 bg-gray-200 rounded"></div>
    </div>
  )
}

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* แสดง Skeleton ระหว่างรอ Analytics */}
      <Suspense fallback={<AnalyticsSkeleton />}>
        <ExpensiveAnalytics />
      </Suspense>
    </div>
  )
}
```

#### Nested Suspense

```tsx
// app/products/[id]/page.tsx
import { Suspense } from 'react'

async function ProductInfo({ id }) {
  const product = await getProduct(id)  // เร็ว ~50ms
  return <ProductCard product={product} />
}

async function ProductReviews({ id }) {
  const reviews = await getReviews(id)  // ช้า ~500ms
  return <ReviewList reviews={reviews} />
}

async function RelatedProducts({ categoryId }) {
  const related = await getRelated(categoryId)  // ช้า ~300ms
  return <ProductGrid products={related} />
}

export default function ProductPage({ params }) {
  return (
    <div>
      {/* โหลดก่อน ไม่ต้อง Suspense */}
      <Breadcrumb />
      
      {/* โหลดเร็ว */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductInfo id={params.id} />
      </Suspense>
      
      {/* โหลดช้า แต่ไม่ Block */}
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

#### React cache() สำหรับ Deduplication

```tsx
// lib/data.ts
import { cache } from 'react'
import { prisma } from './prisma'

// cache() ทำให้ Function นี้ถูกเรียกแค่ครั้งเดียวต่อ Request
// แม้จะเรียกหลายครั้งจากหลาย Components
export const getProduct = cache(async (id: string) => {
  console.log('Fetching product:', id)  // Log แค่ครั้งเดียว
  return prisma.product.findUnique({ where: { id } })
})

// app/products/[id]/page.tsx
async function ProductDetails({ id }) {
  const product = await getProduct(id)  // Call 1
  return <div>{product.name}</div>
}

async function ProductMeta({ id }) {
  const product = await getProduct(id)  // Call 2 - ใช้ Cache
  return <div>{product.description}</div>
}

// ทั้งสองใช้ Cache เดียวกัน - Fetch จริงแค่ครั้งเดียว
```

---

## Step 1060: ตัวอย่าง Blog Page

### 7. ตัวอย่าง Blog Page

#### โครงสร้าง Blog

```
app/
├── blog/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── loading.tsx
│   ├── [slug]/
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   └── not-found.tsx
│   └── category/
│       └── [category]/
│           └── page.tsx
```

#### Blog Layout

```tsx
// app/blog/layout.tsx
import type { Metadata } from 'next'
import Link from 'next/link'
import { prisma } from '@/lib/prisma'

export const metadata: Metadata = {
  title: {
    default: 'Blog',
    template: '%s | Blog',
  },
}

async function BlogSidebar() {
  const [categories, recentPosts] = await Promise.all([
    prisma.category.findMany({
      include: { _count: { select: { posts: true } } }
    }),
    prisma.post.findMany({
      where: { published: true },
      orderBy: { publishedAt: 'desc' },
      take: 5,
      select: { id: true, title: true, slug: true, publishedAt: true }
    })
  ])
  
  return (
    <aside className="w-72 shrink-0">
      <div className="bg-gray-50 rounded-lg p-4 mb-6">
        <h3 className="font-bold text-lg mb-3">Categories</h3>
        <ul className="space-y-2">
          {categories.map(cat => (
            <li key={cat.id}>
              <Link
                href={`/blog/category/${cat.slug}`}
                className="flex justify-between hover:text-blue-600"
              >
                <span>{cat.name}</span>
                <span className="text-gray-500 text-sm">
                  ({cat._count.posts})
                </span>
              </Link>
            </li>
          ))}
        </ul>
      </div>
      
      <div className="bg-gray-50 rounded-lg p-4">
        <h3 className="font-bold text-lg mb-3">Recent Posts</h3>
        <ul className="space-y-3">
          {recentPosts.map(post => (
            <li key={post.id}>
              <Link
                href={`/blog/${post.slug}`}
                className="hover:text-blue-600 block"
              >
                <span className="text-sm font-medium line-clamp-2">
                  {post.title}
                </span>
                <span className="text-xs text-gray-500 mt-1">
                  {post.publishedAt?.toLocaleDateString('th-TH')}
                </span>
              </Link>
            </li>
          ))}
        </ul>
      </div>
    </aside>
  )
}

export default function BlogLayout({ children }) {
  return (
    <div className="max-w-6xl mx-auto px-4 py-8">
      <div className="flex gap-8">
        <main className="flex-1 min-w-0">{children}</main>
        <Suspense fallback={<SidebarSkeleton />}>
          <BlogSidebar />
        </Suspense>
      </div>
    </div>
  )
}
```

#### Blog List Page

```tsx
// app/blog/page.tsx
import type { Metadata } from 'next'
import Link from 'next/link'
import Image from 'next/image'
import { prisma } from '@/lib/prisma'

export const metadata: Metadata = {
  title: 'Blog',
  description: 'อ่านบทความเกี่ยวกับ Next.js และ React',
}

interface PageProps {
  searchParams: { page?: string; category?: string }
}

async function BlogPosts({ searchParams }: PageProps) {
  const page = Number(searchParams.page) || 1
  const limit = 10
  
  const where = {
    published: true,
    ...(searchParams.category && {
      category: { slug: searchParams.category }
    })
  }
  
  const [posts, total] = await Promise.all([
    prisma.post.findMany({
      where,
      include: {
        author: { select: { id: true, name: true, avatar: true } },
        category: true,
        _count: { select: { comments: true } }
      },
      orderBy: { publishedAt: 'desc' },
      skip: (page - 1) * limit,
      take: limit,
    }),
    prisma.post.count({ where })
  ])
  
  const totalPages = Math.ceil(total / limit)
  
  return (
    <div>
      <div className="space-y-8">
        {posts.map(post => (
          <article key={post.id} className="flex gap-6">
            {post.coverImage && (
              <div className="shrink-0">
                <Image
                  src={post.coverImage}
                  alt={post.title}
                  width={200}
                  height={133}
                  className="rounded-lg object-cover"
                />
              </div>
            )}
            <div>
              <div className="flex items-center gap-2 mb-2">
                <Link
                  href={`/blog/category/${post.category.slug}`}
                  className="text-blue-600 text-sm hover:underline"
                >
                  {post.category.name}
                </Link>
                <span className="text-gray-300">•</span>
                <time className="text-gray-500 text-sm">
                  {post.publishedAt?.toLocaleDateString('th-TH', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric'
                  })}
                </time>
              </div>
              
              <h2 className="text-xl font-bold mb-2">
                <Link
                  href={`/blog/${post.slug}`}
                  className="hover:text-blue-600"
                >
                  {post.title}
                </Link>
              </h2>
              
              <p className="text-gray-600 line-clamp-2 mb-3">
                {post.excerpt}
              </p>
              
              <div className="flex items-center gap-4">
                <div className="flex items-center gap-2">
                  {post.author.avatar && (
                    <Image
                      src={post.author.avatar}
                      alt={post.author.name}
                      width={24}
                      height={24}
                      className="rounded-full"
                    />
                  )}
                  <span className="text-sm text-gray-600">
                    {post.author.name}
                  </span>
                </div>
                <span className="text-sm text-gray-500">
                  {post._count.comments} comments
                </span>
              </div>
            </div>
          </article>
        ))}
      </div>
      
      {/* Pagination */}
      {totalPages > 1 && (
        <div className="flex justify-center gap-2 mt-8">
          {Array.from({ length: totalPages }, (_, i) => i + 1).map(p => (
            <Link
              key={p}
              href={{ query: { ...searchParams, page: p } }}
              className={`px-4 py-2 rounded ${
                p === page
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-100 hover:bg-gray-200'
              }`}
            >
              {p}
            </Link>
          ))}
        </div>
      )}
    </div>
  )
}

export default function BlogPage({ searchParams }: PageProps) {
  return (
    <div>
      <h1 className="text-3xl font-bold mb-8">
        {searchParams.category ? `Category: ${searchParams.category}` : 'Blog'}
      </h1>
      <Suspense fallback={<PostListSkeleton />}>
        <BlogPosts searchParams={searchParams} />
      </Suspense>
    </div>
  )
}
```

#### Blog Post Page

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'
import Image from 'next/image'
import Link from 'next/link'
import { notFound } from 'next/navigation'
import { Suspense } from 'react'
import { prisma } from '@/lib/prisma'

interface Props {
  params: { slug: string }
}

export async function generateStaticParams() {
  const posts = await prisma.post.findMany({
    where: { published: true },
    select: { slug: true }
  })
  return posts.map(post => ({ slug: post.slug }))
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await prisma.post.findUnique({
    where: { slug: params.slug },
    select: {
      title: true,
      excerpt: true,
      coverImage: true,
      author: { select: { name: true } }
    }
  })
  
  if (!post) return { title: 'Not Found' }
  
  return {
    title: post.title,
    description: post.excerpt || undefined,
    openGraph: {
      title: post.title,
      description: post.excerpt || undefined,
      images: post.coverImage ? [post.coverImage] : [],
      type: 'article',
      authors: [post.author.name],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt || undefined,
    },
  }
}

async function PostComments({ postId }: { postId: string }) {
  const comments = await prisma.comment.findMany({
    where: { postId, parentId: null },
    include: {
      author: { select: { id: true, name: true, avatar: true } },
      replies: {
        include: {
          author: { select: { id: true, name: true, avatar: true } }
        }
      }
    },
    orderBy: { createdAt: 'desc' }
  })
  
  return (
    <div>
      <h2 className="text-xl font-bold mb-4">
        Comments ({comments.length})
      </h2>
      <CommentList comments={comments} />
      <AddCommentForm postId={postId} />
    </div>
  )
}

export default async function BlogPostPage({ params }: Props) {
  const post = await prisma.post.findUnique({
    where: { slug: params.slug, published: true },
    include: {
      author: {
        select: {
          id: true,
          name: true,
          avatar: true,
          bio: true,
          _count: { select: { posts: true } }
        }
      },
      category: true,
      tags: true,
    }
  })
  
  if (!post) notFound()
  
  // Increment View Count
  await prisma.post.update({
    where: { id: post.id },
    data: { viewCount: { increment: 1 } }
  })
  
  return (
    <article>
      {/* Header */}
      <header className="mb-8">
        <div className="flex items-center gap-2 mb-3">
          <Link
            href={`/blog/category/${post.category.slug}`}
            className="text-blue-600 hover:underline text-sm"
          >
            {post.category.name}
          </Link>
        </div>
        
        <h1 className="text-4xl font-bold leading-tight mb-4">
          {post.title}
        </h1>
        
        {post.excerpt && (
          <p className="text-xl text-gray-600 mb-6">{post.excerpt}</p>
        )}
        
        <div className="flex items-center gap-4">
          <div className="flex items-center gap-3">
            {post.author.avatar && (
              <Image
                src={post.author.avatar}
                alt={post.author.name}
                width={48}
                height={48}
                className="rounded-full"
              />
            )}
            <div>
              <p className="font-medium">{post.author.name}</p>
              <time className="text-gray-500 text-sm">
                {post.publishedAt?.toLocaleDateString('th-TH', {
                  year: 'numeric',
                  month: 'long',
                  day: 'numeric'
                })}
              </time>
            </div>
          </div>
        </div>
      </header>
      
      {/* Cover Image */}
      {post.coverImage && (
        <div className="relative aspect-video mb-8 rounded-xl overflow-hidden">
          <Image
            src={post.coverImage}
            alt={post.title}
            fill
            className="object-cover"
            priority
          />
        </div>
      )}
      
      {/* Content */}
      <div
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: post.contentHtml || '' }}
      />
      
      {/* Tags */}
      <div className="flex gap-2 mt-8">
        {post.tags.map(tag => (
          <Link
            key={tag.id}
            href={`/blog/tag/${tag.slug}`}
            className="px-3 py-1 bg-gray-100 rounded-full text-sm hover:bg-gray-200"
          >
            #{tag.name}
          </Link>
        ))}
      </div>
      
      {/* Author Bio */}
      <div className="mt-10 p-6 bg-gray-50 rounded-xl">
        <h3 className="font-bold text-lg mb-2">About the Author</h3>
        <div className="flex gap-4">
          {post.author.avatar && (
            <Image
              src={post.author.avatar}
              alt={post.author.name}
              width={80}
              height={80}
              className="rounded-full shrink-0"
            />
          )}
          <div>
            <p className="font-medium text-lg">{post.author.name}</p>
            <p className="text-gray-600">{post.author.bio}</p>
            <p className="text-sm text-gray-500 mt-1">
              {post.author._count.posts} บทความ
            </p>
          </div>
        </div>
      </div>
      
      {/* Comments - Load แยก */}
      <div className="mt-10">
        <Suspense fallback={<CommentsSkeleton />}>
          <PostComments postId={post.id} />
        </Suspense>
      </div>
    </article>
  )
}
```

---

## Step 1070: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Default เป็น Server Component

✓ ไม่ต้องทำอะไรพิเศษ - ทุก Component ใน App Router เป็น Server Component
✓ เพิ่ม 'use client' เมื่อต้องการ State, Events หรือ Browser APIs

## 2. ย้าย Client Components ไปไว้ที่ Leaf

✓ ทำให้ Server Component Boundary ใหญ่ที่สุด
✓ Client Components ควรอยู่ล่างสุดของ Tree

## 3. ใช้ cache() สำหรับ Data Deduplication

✓ import { cache } from 'react'
✓ Wrap Function ที่ Query Database
✓ ป้องกัน Query ซ้ำในหนึ่ง Request

## 4. Parallel Fetching เสมอ

✓ ใช้ Promise.all() สำหรับ Independent Data
✓ แยก Suspense Boundaries เพื่อ Parallel Stream

## 5. อย่า Prop-drill ข้ามหลาย Server Components

✓ ใช้ Data Access Layer
✓ แต่ละ Component Fetch ข้อมูลที่ต้องการเอง
```

---

## Quiz

### แบบทดสอบ Part 36

**คำถามที่ 1:** Server Components แตกต่างจาก Client Components อย่างไรในแง่ JavaScript Bundle?
- A) Server Components ใช้ Bundle ใหญ่กว่า
- B) Server Components ไม่อยู่ใน JavaScript Bundle ✓
- C) ขนาด Bundle เท่ากัน
- D) Server Components ใช้ Bundle แยก

**คำถามที่ 2:** อะไรทำไม่ได้ใน Server Component?
- A) Query Database
- B) ใช้ fetch()
- C) ใช้ useState ✓
- D) Read Environment Variables

**คำถามที่ 3:** `cache()` จาก 'react' ใช้ทำอะไร?
- A) Cache HTTP Responses
- B) Memoize Function Calls ต่อ Request เพื่อป้องกัน Duplicate Queries ✓
- C) Cache Component Output
- D) Cache Database Connections

**คำถามที่ 4:** เมื่อ Server Component ใช้ `cookies()` จาก 'next/headers' จะส่งผลอย่างไร?
- A) ไม่มีผลอะไร
- B) Route จะเปลี่ยนจาก Static เป็น Dynamic (ไม่ Cache) ✓
- C) Error เพราะ Server Component ใช้ cookies() ไม่ได้
- D) Cache Route ยาวขึ้น

**คำถามที่ 5:** ทำไมควรใช้ Data Access Layer (DAL) Pattern?
- A) ทำให้ Code เร็วขึ้น
- B) Centralize Data Fetching Logic, Reuse ได้, ง่ายต่อ Test ✓
- C) ลด Database Connections
- D) ไม่มีเหตุผลพิเศษ

---

## สรุป Part 36

ใน Part นี้เราได้เรียนรู้:

1. **Server Components** - Render ที่ Server ไม่มี Client JS
2. **ข้อดี** - Bundle เล็ก, Direct DB Access, API Keys ปลอดภัย
3. **ข้อจำกัด** - ไม่มี State, Events, Browser APIs
4. **Data Fetching** - fetch(), Parallel, Error Handling
5. **Database** - Direct Access, DAL Pattern, cache()
6. **Streaming** - Suspense Boundaries สำหรับ UX ที่ดี
7. **Blog Example** - ตัวอย่างสมบูรณ์

---

➡️ **Part ถัดไป:** [Part 37: Client Components](./part-37-nextjs-client-components.md)
