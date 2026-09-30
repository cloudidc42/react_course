# Part 32: Next.js App Router - การจัดการ Routing

## ข้อมูล Part
- **Steps:** 891-925
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3-4 ชั่วโมง
- **Prerequisites:** Part 31 (Next.js Introduction)

---

## สารบัญ

1. [App Router vs Pages Router](#1-app-router-vs-pages-router)
2. [File-based Routing](#2-file-based-routing)
3. [Dynamic Routes [id]](#3-dynamic-routes-id)
4. [Catch-all Routes [...slug]](#4-catch-all-routes-slug)
5. [Route Groups (folder)](#5-route-groups-folder)
6. [Parallel Routes](#6-parallel-routes)
7. [Intercepting Routes](#7-intercepting-routes)
8. [Link Component](#8-link-component)
9. [useRouter, usePathname, useSearchParams](#9-userouter-usepathname-usesearchparams)
10. [Quiz](#quiz)

---

## Step 891: App Router vs Pages Router

### 1. App Router vs Pages Router

#### Pages Router (เก่า - Next.js < 13)

```
pages/
├── index.js          → /
├── about.js          → /about
├── blog/
│   ├── index.js      → /blog
│   └── [slug].js     → /blog/:slug
└── api/
    └── users.js      → /api/users
```

```jsx
// pages/blog/[slug].js (Pages Router)
import { GetStaticProps, GetStaticPaths } from 'next'

export default function BlogPost({ post }) {
  return <article>{post.title}</article>
}

export async function getStaticPaths() {
  return {
    paths: [{ params: { slug: 'first-post' } }],
    fallback: false
  }
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug)
  return { props: { post } }
}
```

#### App Router (ใหม่ - Next.js 13+)

```
app/
├── layout.tsx         → Root Layout
├── page.tsx           → /
├── about/
│   └── page.tsx       → /about
├── blog/
│   ├── page.tsx       → /blog
│   └── [slug]/
│       └── page.tsx   → /blog/:slug
└── api/
    └── users/
        └── route.ts   → /api/users
```

```tsx
// app/blog/[slug]/page.tsx (App Router)
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return <article>{post.title}</article>
}

export async function generateStaticParams() {
  return [{ slug: 'first-post' }]
}
```

#### เปรียบเทียบ

```
┌──────────────────────┬─────────────────────┬─────────────────────┐
│ Feature              │ Pages Router        │ App Router          │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Server Components    │ ✗                   │ ✓ (Default)         │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Layouts              │ Manual (_app.js)    │ layout.tsx Built-in │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Data Fetching        │ getServerSideProps  │ fetch() in Component│
│                      │ getStaticProps      │                     │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Loading States       │ Manual              │ loading.tsx         │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Error Handling       │ Manual              │ error.tsx           │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Streaming            │ Limited             │ ✓ (Suspense)        │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Server Actions       │ ✗                   │ ✓                   │
└──────────────────────┴─────────────────────┴─────────────────────┘
```

---

## Step 893: File-based Routing

### 2. File-based Routing

App Router ใช้ File System ในการกำหนด Routes โดยแต่ละ Folder ใน `app/` จะกลายเป็น URL Segment

#### Reserved Files

```
app/
├── layout.tsx      # Layout สำหรับ Route และ Children
├── page.tsx        # UI หลักของ Route (ทำให้ Route Accessible)
├── loading.tsx     # Loading UI (Suspense)
├── not-found.tsx   # Not Found UI (404)
├── error.tsx       # Error UI
├── global-error.tsx # Global Error UI
├── route.ts        # API Endpoint
├── template.tsx    # Re-rendered Layout
└── default.tsx     # Fallback สำหรับ Parallel Routes
```

#### ตัวอย่าง Routing

```
app/
├── page.tsx                    → URL: /
├── about/
│   └── page.tsx               → URL: /about
├── blog/
│   ├── page.tsx               → URL: /blog
│   ├── layout.tsx             → Layout สำหรับ /blog/*
│   └── [slug]/
│       └── page.tsx           → URL: /blog/anything
├── dashboard/
│   ├── layout.tsx             → Layout สำหรับ /dashboard/*
│   ├── page.tsx               → URL: /dashboard
│   ├── settings/
│   │   └── page.tsx           → URL: /dashboard/settings
│   └── analytics/
│       └── page.tsx           → URL: /dashboard/analytics
└── api/
    ├── users/
    │   └── route.ts           → API: /api/users
    └── posts/
        └── route.ts           → API: /api/posts
```

#### Layout Nesting

```tsx
// app/layout.tsx - Root Layout (ทุก Page ใช้)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  )
}

// app/blog/layout.tsx - Blog Layout (เฉพาะ /blog/*)
export default function BlogLayout({ children }) {
  return (
    <div className="blog-container">
      <BlogSidebar />
      <main>{children}</main>
    </div>
  )
}

// app/blog/page.tsx - Blog List
export default function BlogPage() {
  return <div>Blog Posts List</div>
}

// ผลลัพธ์ที่ /blog:
// <RootLayout>
//   <BlogLayout>
//     <BlogPage />
//   </BlogLayout>
// </RootLayout>
```

---

## Step 895: Dynamic Routes

### 3. Dynamic Routes [id]

Dynamic Segments ถูกสร้างโดยใส่ชื่อ Folder ใน `[brackets]`

#### Basic Dynamic Route

```
app/
└── products/
    ├── page.tsx           → /products
    └── [id]/
        └── page.tsx       → /products/1, /products/abc, etc.
```

```tsx
// app/products/[id]/page.tsx
interface Props {
  params: {
    id: string
  }
}

export default async function ProductPage({ params }: Props) {
  const product = await getProduct(params.id)
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>ID: {params.id}</p>
    </div>
  )
}
```

#### Multiple Dynamic Segments

```
app/
└── shop/
    └── [category]/
        └── [id]/
            └── page.tsx   → /shop/electronics/123
```

```tsx
// app/shop/[category]/[id]/page.tsx
interface Props {
  params: {
    category: string
    id: string
  }
}

export default async function ShopItemPage({ params }: Props) {
  const { category, id } = params
  
  return (
    <div>
      <p>Category: {category}</p>
      <p>ID: {id}</p>
    </div>
  )
}
```

#### generateStaticParams สำหรับ SSG

```tsx
// app/blog/[slug]/page.tsx

// กำหนด Static Paths สำหรับ Build Time
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts')
    .then(r => r.json())
  
  return posts.map(post => ({
    slug: post.slug,
  }))
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return <article>{post.title}</article>
}
```

#### notFound() และ Not Found Page

```tsx
// app/products/[id]/page.tsx
import { notFound } from 'next/navigation'

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id)
  
  if (!product) {
    notFound() // แสดง not-found.tsx
  }
  
  return <div>{product.name}</div>
}

// app/products/[id]/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>Product Not Found</h2>
      <p>ไม่พบสินค้าที่คุณต้องการ</p>
    </div>
  )
}
```

---

## Step 898: Catch-all Routes

### 4. Catch-all Routes [...slug]

ใช้สำหรับ Match หลาย URL Segments พร้อมกัน

#### Basic Catch-all

```
app/
└── docs/
    └── [...slug]/
        └── page.tsx
```

```
/docs/intro              → params.slug = ['intro']
/docs/getting-started    → params.slug = ['getting-started']
/docs/api/endpoints      → params.slug = ['api', 'endpoints']
/docs/guide/intro/setup  → params.slug = ['guide', 'intro', 'setup']
```

```tsx
// app/docs/[...slug]/page.tsx
interface Props {
  params: {
    slug: string[]
  }
}

export default function DocsPage({ params }: Props) {
  const { slug } = params
  const path = slug.join('/')
  
  return (
    <div>
      <h1>Documentation</h1>
      <p>Path: {path}</p>
      <p>Segments: {slug.join(' > ')}</p>
    </div>
  )
}
```

#### Optional Catch-all [[...slug]]

```
app/
└── shop/
    └── [[...slug]]/
        └── page.tsx
```

```
/shop              → params.slug = undefined
/shop/clothes      → params.slug = ['clothes']
/shop/clothes/men  → params.slug = ['clothes', 'men']
```

```tsx
// app/shop/[[...slug]]/page.tsx
interface Props {
  params: {
    slug?: string[]
  }
}

export default function ShopPage({ params }: Props) {
  const { slug } = params
  
  if (!slug || slug.length === 0) {
    return <div>Shop Home</div>
  }
  
  if (slug.length === 1) {
    return <div>Category: {slug[0]}</div>
  }
  
  return (
    <div>
      <p>Category: {slug[0]}</p>
      <p>Subcategory: {slug[1]}</p>
    </div>
  )
}
```

#### ตัวอย่าง Documentation Site

```tsx
// app/docs/[[...slug]]/page.tsx
import { notFound } from 'next/navigation'

const DOCS_CONTENT = {
  '': { title: 'Introduction', content: '...' },
  'getting-started': { title: 'Getting Started', content: '...' },
  'api/overview': { title: 'API Overview', content: '...' },
  'api/endpoints': { title: 'API Endpoints', content: '...' },
}

export async function generateStaticParams() {
  return Object.keys(DOCS_CONTENT).map(path => ({
    slug: path ? path.split('/') : undefined
  }))
}

export default function DocsPage({ params }) {
  const path = params.slug?.join('/') ?? ''
  const doc = DOCS_CONTENT[path]
  
  if (!doc) notFound()
  
  return (
    <div>
      <h1>{doc.title}</h1>
      <p>{doc.content}</p>
    </div>
  )
}
```

---

## Step 901: Route Groups

### 5. Route Groups (folder)

Route Groups ทำให้ Organize Routes ได้โดยไม่ส่งผลต่อ URL โดยใส่ชื่อ Folder ใน `(parentheses)`

#### การจัดกลุ่ม Routes

```
app/
├── (marketing)/
│   ├── layout.tsx         → Layout สำหรับ Marketing Pages
│   ├── page.tsx           → /
│   ├── about/
│   │   └── page.tsx       → /about
│   └── pricing/
│       └── page.tsx       → /pricing
├── (dashboard)/
│   ├── layout.tsx         → Layout สำหรับ Dashboard
│   ├── dashboard/
│   │   └── page.tsx       → /dashboard
│   └── settings/
│       └── page.tsx       → /settings
└── (auth)/
    ├── layout.tsx         → Layout สำหรับ Auth Pages
    ├── login/
    │   └── page.tsx       → /login
    └── register/
        └── page.tsx       → /register
```

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({ children }) {
  return (
    <div>
      <MarketingHeader />
      {children}
      <MarketingFooter />
    </div>
  )
}

// app/(dashboard)/layout.tsx
export default function DashboardLayout({ children }) {
  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1">{children}</main>
    </div>
  )
}
```

#### Multiple Root Layouts

```
app/
├── (shop)/
│   ├── layout.tsx     → Shop Layout (มี <html> <body>)
│   └── page.tsx
└── (checkout)/
    ├── layout.tsx     → Checkout Layout (มี <html> <body>)
    └── page.tsx
```

---

## Step 904: Parallel Routes

### 6. Parallel Routes

Parallel Routes ให้ Render หลาย Pages ใน Layout เดียวกันพร้อมกัน โดยใช้ `@` นำหน้าชื่อ Folder

#### โครงสร้าง

```
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    └── @revenue/
        └── page.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  revenue,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  revenue: React.ReactNode
}) {
  return (
    <div>
      <div>{children}</div>
      <div className="grid grid-cols-2 gap-4">
        <div>{analytics}</div>
        <div>{revenue}</div>
      </div>
    </div>
  )
}

// app/dashboard/@analytics/page.tsx
export default function AnalyticsPage() {
  return (
    <div className="card">
      <h2>Analytics</h2>
      <AnalyticsChart />
    </div>
  )
}

// app/dashboard/@revenue/page.tsx
export default function RevenuePage() {
  return (
    <div className="card">
      <h2>Revenue</h2>
      <RevenueChart />
    </div>
  )
}
```

#### Parallel Routes กับ Conditional Rendering

```tsx
// app/dashboard/layout.tsx
import { checkAuth } from '@/lib/auth'

export default async function DashboardLayout({
  children,
  authenticated,
  unauthenticated,
}: {
  children: React.ReactNode
  authenticated: React.ReactNode
  unauthenticated: React.ReactNode
}) {
  const isLoggedIn = await checkAuth()
  
  return (
    <div>
      {isLoggedIn ? authenticated : unauthenticated}
    </div>
  )
}
```

---

## Step 907: Intercepting Routes

### 7. Intercepting Routes

Intercepting Routes ให้ Load Route ใหม่ใน Context ของ Route ปัจจุบัน (เช่น Modal)

#### Conventions

```
(.) - Match segments at same level
(..) - Match segments one level above
(..)(..) - Match segments two levels above
(...) - Match segments from root
```

#### ตัวอย่าง Photo Gallery Modal

```
app/
├── layout.tsx
├── page.tsx                    → Gallery
├── photo/
│   └── [id]/
│       └── page.tsx           → Full Photo Page /photo/1
└── @modal/
    ├── default.tsx             → null (no modal)
    └── (.)photo/
        └── [id]/
            └── page.tsx       → Photo Modal (intercepts /photo/1)
```

```tsx
// app/page.tsx - Gallery
import Link from 'next/link'

export default function Gallery() {
  const photos = [1, 2, 3, 4, 5]
  
  return (
    <div className="grid grid-cols-3 gap-4">
      {photos.map(id => (
        <Link key={id} href={`/photo/${id}`}>
          <img src={`/photos/${id}.jpg`} alt={`Photo ${id}`} />
        </Link>
      ))}
    </div>
  )
}

// app/@modal/(.)photo/[id]/page.tsx - Modal
'use client'
import { useRouter } from 'next/navigation'

export default function PhotoModal({ params }) {
  const router = useRouter()
  
  return (
    <dialog className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white p-4 rounded-lg max-w-2xl w-full">
        <button 
          onClick={() => router.back()}
          className="mb-4"
        >
          ✕ ปิด
        </button>
        <img 
          src={`/photos/${params.id}.jpg`} 
          alt={`Photo ${params.id}`}
          className="w-full"
        />
      </div>
    </dialog>
  )
}

// app/@modal/default.tsx
export default function Default() {
  return null
}

// app/layout.tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode
  modal: React.ReactNode
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  )
}
```

---

## Step 910: Link Component

### 8. Link Component

`<Link>` คือ Next.js Component สำหรับ Client-side Navigation ที่ Prefetch Routes อัตโนมัติ

#### การใช้งานพื้นฐาน

```tsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav>
      {/* Basic Link */}
      <Link href="/">Home</Link>
      
      {/* With className */}
      <Link href="/about" className="text-blue-500 hover:underline">
        About
      </Link>
      
      {/* Replace history instead of push */}
      <Link href="/login" replace>
        Login
      </Link>
      
      {/* Disable Prefetch */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page
      </Link>
    </nav>
  )
}
```

#### Link กับ Dynamic Routes

```tsx
import Link from 'next/link'

interface Post {
  id: string
  title: string
}

function PostList({ posts }: { posts: Post[] }) {
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>
          {/* String href */}
          <Link href={`/blog/${post.id}`}>
            {post.title}
          </Link>
          
          {/* Object href */}
          <Link href={{
            pathname: '/blog/[id]',
            query: { id: post.id }
          }}>
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  )
}
```

#### Active Link

```tsx
'use client'
import Link from 'next/link'
import { usePathname } from 'next/navigation'

const navItems = [
  { href: '/', label: 'Home' },
  { href: '/about', label: 'About' },
  { href: '/blog', label: 'Blog' },
  { href: '/contact', label: 'Contact' },
]

export default function Navigation() {
  const pathname = usePathname()
  
  return (
    <nav>
      {navItems.map(item => (
        <Link
          key={item.href}
          href={item.href}
          className={
            pathname === item.href
              ? 'text-blue-500 font-bold'
              : 'text-gray-600 hover:text-gray-900'
          }
        >
          {item.label}
        </Link>
      ))}
    </nav>
  )
}
```

#### Scroll Behavior

```tsx
// Scroll ไปยัง Element ที่ระบุ
<Link href="/blog#comments" scroll={true}>
  Go to Comments
</Link>

// ไม่ Scroll to Top
<Link href="/next-page" scroll={false}>
  Next Page
</Link>
```

---

## Step 913: useRouter, usePathname, useSearchParams

### 9. useRouter, usePathname, useSearchParams

#### useRouter

```tsx
'use client'
import { useRouter } from 'next/navigation'

export default function LoginPage() {
  const router = useRouter()
  
  const handleLogin = async () => {
    await login()
    
    // Navigate programmatically
    router.push('/dashboard')
    
    // Navigate และ Replace history
    router.replace('/dashboard')
    
    // Go back
    router.back()
    
    // Go forward
    router.forward()
    
    // Refresh current page
    router.refresh()
    
    // Prefetch a route
    router.prefetch('/heavy-page')
  }
  
  return (
    <form onSubmit={handleLogin}>
      <button type="submit">Login</button>
    </form>
  )
}
```

#### usePathname

```tsx
'use client'
import { usePathname } from 'next/navigation'

export default function Breadcrumb() {
  const pathname = usePathname()
  // pathname = '/blog/my-post'
  
  const segments = pathname.split('/').filter(Boolean)
  // segments = ['blog', 'my-post']
  
  return (
    <nav>
      <ol className="flex gap-2">
        <li>
          <a href="/">Home</a>
        </li>
        {segments.map((segment, i) => {
          const href = '/' + segments.slice(0, i + 1).join('/')
          return (
            <li key={href} className="flex items-center gap-2">
              <span>/</span>
              <a href={href} className="capitalize">
                {segment.replace(/-/g, ' ')}
              </a>
            </li>
          )
        })}
      </ol>
    </nav>
  )
}
```

#### useSearchParams

```tsx
'use client'
import { useSearchParams, usePathname, useRouter } from 'next/navigation'
import { useCallback } from 'react'

export default function ProductFilters() {
  const searchParams = useSearchParams()
  const pathname = usePathname()
  const { replace } = useRouter()
  
  // อ่าน Search Params
  const category = searchParams.get('category') ?? 'all'
  const sort = searchParams.get('sort') ?? 'newest'
  const page = searchParams.get('page') ?? '1'
  
  // อัพเดท Search Params
  const updateFilter = useCallback((key: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString())
    
    if (value) {
      params.set(key, value)
    } else {
      params.delete(key)
    }
    
    // Reset page เมื่อเปลี่ยน Filter
    if (key !== 'page') {
      params.set('page', '1')
    }
    
    replace(`${pathname}?${params.toString()}`)
  }, [searchParams, pathname, replace])
  
  return (
    <div className="flex gap-4">
      <select
        value={category}
        onChange={e => updateFilter('category', e.target.value)}
      >
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      
      <select
        value={sort}
        onChange={e => updateFilter('sort', e.target.value)}
      >
        <option value="newest">Newest</option>
        <option value="oldest">Oldest</option>
        <option value="price-low">Price: Low to High</option>
        <option value="price-high">Price: High to Low</option>
      </select>
    </div>
  )
}
```

#### useParams

```tsx
'use client'
import { useParams } from 'next/navigation'

// app/blog/[category]/[slug]/page.tsx
export default function BlogPost() {
  const params = useParams()
  // params = { category: 'tech', slug: 'my-post' }
  
  return (
    <div>
      <p>Category: {params.category}</p>
      <p>Slug: {params.slug}</p>
    </div>
  )
}
```

---

## Step 916: ตัวอย่างสมบูรณ์ - E-commerce Routing

### ตัวอย่างสมบูรณ์: E-commerce Routing

#### โครงสร้าง Routes

```
app/
├── (shop)/
│   ├── layout.tsx              → Shop Layout
│   ├── page.tsx                → / (Home)
│   ├── products/
│   │   ├── page.tsx            → /products
│   │   └── [id]/
│   │       ├── page.tsx        → /products/[id]
│   │       └── not-found.tsx   → Product Not Found
│   └── categories/
│       └── [category]/
│           └── [[...filters]]/
│               └── page.tsx    → /categories/[category]/[...filters]
├── (checkout)/
│   ├── layout.tsx              → Checkout Layout (no header/footer)
│   ├── cart/
│   │   └── page.tsx            → /cart
│   ├── checkout/
│   │   └── page.tsx            → /checkout
│   └── order-confirmation/
│       └── [orderId]/
│           └── page.tsx        → /order-confirmation/[orderId]
├── (account)/
│   ├── layout.tsx              → Account Layout
│   ├── account/
│   │   └── page.tsx            → /account
│   ├── orders/
│   │   ├── page.tsx            → /orders
│   │   └── [id]/
│   │       └── page.tsx        → /orders/[id]
│   └── wishlist/
│       └── page.tsx            → /wishlist
└── api/
    ├── products/
    │   └── route.ts            → GET /api/products
    └── cart/
        └── route.ts            → GET, POST /api/cart
```

#### app/(shop)/products/page.tsx

```tsx
import Link from 'next/link'
import { Suspense } from 'react'

async function ProductList({ searchParams }) {
  const { category, sort, page = '1' } = searchParams
  
  const products = await fetch(
    `/api/products?category=${category}&sort=${sort}&page=${page}`,
    { next: { revalidate: 300 } }
  ).then(r => r.json())
  
  return (
    <div className="grid grid-cols-4 gap-4">
      {products.map(product => (
        <Link key={product.id} href={`/products/${product.id}`}>
          <div className="border rounded-lg p-4 hover:shadow-lg">
            <img src={product.image} alt={product.name} />
            <h3>{product.name}</h3>
            <p className="text-blue-600">{product.price} บาท</p>
          </div>
        </Link>
      ))}
    </div>
  )
}

export default function ProductsPage({ searchParams }) {
  return (
    <div>
      <h1>สินค้าทั้งหมด</h1>
      <Suspense fallback={<div>กำลังโหลดสินค้า...</div>}>
        <ProductList searchParams={searchParams} />
      </Suspense>
    </div>
  )
}
```

#### app/(shop)/categories/[category]/[[...filters]]/page.tsx

```tsx
interface Props {
  params: {
    category: string
    filters?: string[]
  }
  searchParams: {
    sort?: string
    price?: string
  }
}

export default function CategoryPage({ params, searchParams }: Props) {
  const { category, filters = [] } = params
  const { sort, price } = searchParams
  
  // Parse filters: ['brand', 'Apple', 'color', 'white']
  const filterMap: Record<string, string> = {}
  for (let i = 0; i < filters.length; i += 2) {
    filterMap[filters[i]] = filters[i + 1]
  }
  
  return (
    <div>
      <h1 className="capitalize">{category}</h1>
      {Object.entries(filterMap).map(([key, value]) => (
        <span key={key} className="badge">
          {key}: {value}
        </span>
      ))}
      <ProductGrid category={category} filters={filterMap} sort={sort} />
    </div>
  )
}

// URL: /categories/electronics/brand/Apple/color/black
// params.category = 'electronics'
// params.filters = ['brand', 'Apple', 'color', 'black']
```

---

## Step 920: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. ใช้ Route Groups เพื่อจัดการ Layout

✓ แยก Layout สำหรับส่วนต่างๆ ด้วย Route Groups
✓ (marketing), (dashboard), (auth) เป็น Convention ที่ดี

## 2. generateStaticParams สำหรับ Dynamic Routes

✓ ใช้ generateStaticParams เมื่อ Routes มีจำนวนจำกัด
✓ ทำให้ Static Generation ทำงานได้กับ Dynamic Routes

## 3. Prefetching

✓ Link Component Prefetch อัตโนมัติใน Viewport
✓ ใช้ prefetch={false} สำหรับ Pages ที่หนัก
✓ ใช้ router.prefetch() สำหรับ Prefetch แบบ Manual

## 4. useSearchParams ต้องใช้ใน Client Component

✓ useSearchParams ต้องใช้ 'use client'
✓ Wrap ด้วย Suspense เมื่อใช้ใน Server Component Tree

## 5. ระวัง URL ซ้ำ

✗ อย่าสร้าง page.tsx ทั้งใน app/ และ app/(group)/
✗ จะทำให้เกิด Route Conflict Error
```

---

## Quiz

### แบบทดสอบ Part 32

**คำถามที่ 1:** ไฟล์อะไรที่ต้องมีใน Folder เพื่อให้ Route นั้นเข้าถึงได้ (Accessible)?
- A) layout.tsx
- B) page.tsx ✓
- C) route.ts
- D) index.tsx

**คำถามที่ 2:** Route Group คืออะไร?
- A) การจัดกลุ่ม Routes ที่เพิ่ม URL Segment
- B) การจัดกลุ่ม Routes โดยใส่ชื่อ Folder ใน `()` โดยไม่ส่งผลต่อ URL ✓
- C) การสร้าง Sub-routing ด้วย Nested Folders
- D) การ Group Routes เพื่อ Caching

**คำถามที่ 3:** `[...slug]` แตกต่างจาก `[[...slug]]` อย่างไร?
- A) ไม่แตกต่างกัน
- B) `[...slug]` ต้องมีอย่างน้อย 1 Segment, `[[...slug]]` ไม่ต้องมีก็ได้ ✓
- C) `[[...slug]]` รับได้มากกว่า
- D) `[...slug]` ใช้สำหรับ Files

**คำถามที่ 4:** useRouter จาก 'next/navigation' ใช้ทำอะไร?
- A) อ่านค่าจาก URL เท่านั้น
- B) Navigate programmatically ได้ เช่น push, replace, back ✓
- C) สร้าง Route ใหม่
- D) จัดการ State

**คำถามที่ 5:** Intercepting Routes ใช้ทำอะไร?
- A) Block การเข้าถึง Routes บางเส้นทาง
- B) Load Route ใหม่ใน Context ของ Route ปัจจุบัน เช่น Modal ✓
- C) Redirect Route
- D) Cache Route

---

## สรุป Part 32

ใน Part นี้เราได้เรียนรู้:

1. **App Router** vs Pages Router - App Router ดีกว่าและเป็น Default ใหม่
2. **File-based Routing** - Folder Structure คือ URL
3. **Dynamic Routes** `[id]` - สำหรับ ID ต่างๆ
4. **Catch-all Routes** `[...slug]` - สำหรับ Multiple Segments
5. **Route Groups** `(folder)` - จัดกลุ่มโดยไม่กระทบ URL
6. **Parallel Routes** `@slot` - Render หลาย Components พร้อมกัน
7. **Intercepting Routes** - Modal Pattern ที่ดีมาก
8. **Link Component** - Client-side Navigation
9. **Hooks** - useRouter, usePathname, useSearchParams

---

➡️ **Part ถัดไป:** [Part 33: Pages vs App Router](./part-33-nextjs-pages-app-router.md)
