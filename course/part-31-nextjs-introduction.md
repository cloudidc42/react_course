# Part 31: แนะนำ Next.js

## ข้อมูล Part
- **Steps:** 856-890
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3-4 ชั่วโมง
- **Prerequisites:** React Fundamentals, Hooks, Component Patterns

---

## สารบัญ

1. [Next.js คืออะไร](#1-nextjs-คืออะไร)
2. [ทำไมต้องใช้ Next.js](#2-ทำไมต้องใช้-nextjs)
3. [SSR, SSG, ISR, CSR คืออะไร](#3-ssr-ssg-isr-csr-คืออะไร)
4. [เปรียบเทียบ React vs Next.js](#4-เปรียบเทียบ-react-vs-nextjs)
5. [สร้าง Next.js Project](#5-สร้าง-nextjs-project)
6. [โครงสร้าง Project (App Router)](#6-โครงสร้าง-project-app-router)
7. [การรัน Development Server](#7-การรัน-development-server)
8. [Next.js vs Vite](#8-nextjs-vs-vite)
9. [Features ของ Next.js 14+](#9-features-ของ-nextjs-14)
10. [Quiz](#quiz)

---

## Step 856: แนะนำ Next.js Framework

### 1. Next.js คืออะไร

**Next.js** คือ React Framework ที่พัฒนาโดย Vercel สำหรับสร้าง Web Applications แบบ Full-Stack โดย Next.js ช่วยให้เราสามารถ:

- สร้าง **Server-Side Rendering (SSR)** ได้ง่าย
- สร้าง **Static Site Generation (SSG)** เพื่อ Performance ที่ดีที่สุด
- จัดการ **Routing** โดยอัตโนมัติผ่าน File System
- เขียน **API Routes** ได้ในโปรเจกต์เดียวกัน
- รองรับ **TypeScript** แบบ Built-in
- มี **Image Optimization** อัตโนมัติ

```
Next.js = React + Server Side Rendering + Routing + API + Optimization
```

### ประวัติ Next.js

```
2016 - Next.js เวอร์ชัน 1.0 ออกมาโดย Zeit (ปัจจุบันคือ Vercel)
2020 - Next.js 9.5: ISR (Incremental Static Regeneration)
2021 - Next.js 12: Middleware, SWC Compiler
2022 - Next.js 13: App Router (beta), Server Components
2023 - Next.js 13.4: App Router (stable)
2023 - Next.js 14: Server Actions (stable), Turbopack
2024 - Next.js 15: React 19 Support, Improved Caching
```

---

## Step 857: ทำไมต้องใช้ Next.js

### 2. ทำไมต้องใช้ Next.js

#### ปัญหาของ React แบบปกติ (Client-Side Only)

```
ผู้ใช้เปิด Browser
    ↓
Load HTML ว่างเปล่า (index.html)
    ↓
Download JavaScript Bundle (อาจใหญ่มาก)
    ↓
JavaScript รัน และสร้าง HTML
    ↓
Fetch Data จาก API
    ↓
Render UI
```

**ปัญหา:**
1. **SEO ไม่ดี** - Bot ของ Google อาจไม่เห็นเนื้อหา
2. **First Contentful Paint ช้า** - ผู้ใช้เห็นหน้าว่างนาน
3. **Performance** - ต้อง Download JS ก่อนจึงจะเห็นอะไร

#### Next.js แก้ปัญหาอย่างไร

```
ผู้ใช้เปิด Browser
    ↓
Server ประมวลผล React Components
    ↓
ส่ง HTML ที่มีเนื้อหาพร้อมกลับมา
    ↓
ผู้ใช้เห็นเนื้อหาทันที
    ↓
JavaScript Hydrate (ทำให้ Interactive)
```

### ข้อดีของ Next.js

```markdown
1. SEO Friendly
   - เนื้อหาอยู่ใน HTML แล้ว
   - Google Bot อ่านได้ทันที
   - Meta Tags จัดการได้ง่าย

2. Performance
   - Automatic Code Splitting
   - Image Optimization
   - Font Optimization
   - Link Prefetching

3. Developer Experience
   - File-based Routing
   - Hot Module Replacement
   - TypeScript Built-in
   - ESLint Built-in

4. Full-Stack Capability
   - API Routes ในโปรเจกต์เดียว
   - Server Actions
   - Database Integration

5. Deployment
   - Vercel Deployment ง่ายมาก
   - Static Export รองรับ
   - Edge Functions
```

---

## Step 858: SSR, SSG, ISR, CSR

### 3. SSR, SSG, ISR, CSR คืออะไร

#### Client-Side Rendering (CSR)

**CSR** คือการ Render HTML ที่ฝั่ง Browser (JavaScript)

```
Server ---> HTML ว่าง + JS Bundle ---> Browser
Browser รัน JS ---> สร้าง HTML ---> แสดงผล
```

```jsx
// React แบบปกติ (CSR)
// index.html
<div id="root"></div>

// main.jsx
ReactDOM.createRoot(document.getElementById('root')).render(<App />)

// App.jsx - ทุกอย่าง Render ที่ Browser
function App() {
  const [data, setData] = useState(null)
  
  useEffect(() => {
    fetch('/api/data').then(r => r.json()).then(setData)
  }, [])
  
  return <div>{data ? data.title : 'Loading...'}</div>
}
```

**เมื่อไหรใช้ CSR:**
- Dashboard ที่ไม่ต้องการ SEO
- Admin Panel
- Interactive Apps ที่ Data เปลี่ยนบ่อย

---

#### Server-Side Rendering (SSR)

**SSR** คือการ Render HTML ที่ฝั่ง Server ทุกครั้งที่มี Request

```
User Request ---> Server Render HTML ---> ส่ง HTML สมบูรณ์ ---> Browser
```

```jsx
// Next.js App Router - SSR (dynamic)
// app/products/page.tsx
async function ProductsPage() {
  // Code นี้รันที่ Server
  const products = await fetch('https://api.example.com/products', {
    cache: 'no-store'  // ไม่ Cache = SSR
  }).then(r => r.json())
  
  return (
    <div>
      <h1>Products</h1>
      {products.map(p => (
        <div key={p.id}>{p.name}</div>
      ))}
    </div>
  )
}

export default ProductsPage
```

**เมื่อไหรใช้ SSR:**
- หน้าที่ Data เปลี่ยนบ่อย (Real-time)
- หน้าที่ต้องการ Auth
- หน้าที่ Data แตกต่างตาม User

---

#### Static Site Generation (SSG)

**SSG** คือการ Render HTML ตอน Build Time และ Cache ไว้

```
Build Time: สร้าง HTML สำหรับทุก Page
Deploy: Upload HTML Files
User Request ---> CDN ส่ง HTML ที่ Cache ไว้
```

```jsx
// Next.js App Router - SSG (static)
// app/blog/page.tsx
async function BlogPage() {
  // Code นี้รันตอน Build Time เท่านั้น
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache'  // Cache = SSG
  }).then(r => r.json())
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  )
}

export default BlogPage
```

**เมื่อไหรใช้ SSG:**
- Blog Posts
- Marketing Pages
- Documentation
- E-commerce Product Pages (ที่ไม่เปลี่ยนบ่อย)

---

#### Incremental Static Regeneration (ISR)

**ISR** คือการ Regenerate Static Pages เป็นระยะๆ โดยไม่ต้อง Rebuild ทั้งหมด

```
Build Time: สร้าง HTML
User Request 1: ส่ง Cached HTML
ผ่านไป 60 วินาที
User Request 2: ส่ง Cached HTML (เก่า) แต่สั่ง Regenerate
User Request 3: ส่ง HTML ใหม่
```

```jsx
// Next.js App Router - ISR
// app/news/page.tsx
async function NewsPage() {
  const news = await fetch('https://api.example.com/news', {
    next: { revalidate: 60 }  // Revalidate ทุก 60 วินาที = ISR
  }).then(r => r.json())
  
  return (
    <div>
      {news.map(item => (
        <article key={item.id}>
          <h2>{item.title}</h2>
          <p>{item.content}</p>
        </article>
      ))}
    </div>
  )
}

export default NewsPage
```

**เมื่อไหรใช้ ISR:**
- News Site
- E-commerce (Price อัพเดทได้)
- Sports Scores
- หน้าที่ Data เปลี่ยนเป็นระยะๆ

---

#### เปรียบเทียบทั้ง 4 แบบ

```
┌─────────────┬──────────┬──────────┬──────────┬──────────┐
│ Rendering   │ CSR      │ SSR      │ SSG      │ ISR      │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ Render When │ Browser  │ Request  │ Build    │ Build+   │
│             │          │ Time     │ Time     │ Schedule │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ SEO         │ Poor     │ Good     │ Best     │ Good     │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ Performance │ Poor     │ Good     │ Best     │ Good     │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ Fresh Data  │ Always   │ Always   │ Build    │ Periodic │
├─────────────┼──────────┼──────────┼──────────┼──────────┤
│ Server Load │ Low      │ High     │ Low      │ Low      │
└─────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## Step 860: React vs Next.js

### 4. เปรียบเทียบ React vs Next.js

```
┌──────────────────┬────────────────────┬──────────────────────┐
│ Feature          │ React (Vite)       │ Next.js              │
├──────────────────┼────────────────────┼──────────────────────┤
│ Routing          │ ต้องใช้ Library    │ Built-in File-based  │
│                  │ (react-router)     │ Routing              │
├──────────────────┼────────────────────┼──────────────────────┤
│ Rendering        │ CSR เท่านั้น       │ CSR, SSR, SSG, ISR   │
├──────────────────┼────────────────────┼──────────────────────┤
│ API Endpoints    │ ต้องสร้าง Server  │ Built-in API Routes  │
│                  │ แยก               │                      │
├──────────────────┼────────────────────┼──────────────────────┤
│ SEO              │ ต้องทำเพิ่ม       │ Built-in             │
├──────────────────┼────────────────────┼──────────────────────┤
│ Image            │ ต้องจัดการเอง     │ Next/Image Component │
│ Optimization     │                   │                      │
├──────────────────┼────────────────────┼──────────────────────┤
│ Font             │ Manual             │ next/font            │
│ Optimization     │                   │                      │
├──────────────────┼────────────────────┼──────────────────────┤
│ TypeScript       │ ต้อง Setup        │ Built-in             │
├──────────────────┼────────────────────┼──────────────────────┤
│ Bundle Size      │ ต้องทำ Manual     │ Automatic Code Split │
├──────────────────┼────────────────────┼──────────────────────┤
│ Server Code      │ ไม่มี             │ Server Components    │
│                  │                   │ Server Actions       │
└──────────────────┴────────────────────┴──────────────────────┘
```

### เมื่อไหรใช้ React (Vite)

```
✓ SPA (Single Page Application) ที่ไม่ต้องการ SEO
✓ Internal Dashboard / Admin Panel
✓ Apps ที่ต้องการ Routing ง่ายๆ
✓ Projects ที่ต้องการ Control เต็มที่
✓ Teams ที่คุ้นเคยกับ React เท่านั้น
```

### เมื่อไหรใช้ Next.js

```
✓ E-commerce Site (ต้องการ SEO)
✓ Blog / Content Site
✓ Marketing Website
✓ Full-Stack Application
✓ Apps ที่ต้องการ Performance สูง
✓ Projects ที่ต้องการทั้ง Frontend และ Backend
```

---

## Step 862: สร้าง Next.js Project

### 5. สร้าง Next.js Project

#### การติดตั้ง

```bash
# ใช้ create-next-app (แนะนำ)
npx create-next-app@latest my-app

# หรือระบุ Options เอง
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"
```

#### คำถามระหว่าง Setup

```
? What is your project named? my-nextjs-app
? Would you like to use TypeScript? › Yes
? Would you like to use ESLint? › Yes
? Would you like to use Tailwind CSS? › Yes
? Would you like your code inside a `src/` directory? › No
? Would you like to use App Router? (recommended) › Yes
? Would you like to customize the import alias (@/*)? › No
```

#### Package.json ที่ได้

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "14.2.0",
    "react": "^18",
    "react-dom": "^18"
  },
  "devDependencies": {
    "typescript": "^5",
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "eslint": "^8",
    "eslint-config-next": "14.2.0",
    "tailwindcss": "^3.3.0",
    "postcss": "^8",
    "autoprefixer": "^10"
  }
}
```

---

## Step 864: โครงสร้าง Project

### 6. โครงสร้าง Project (App Router)

```
my-app/
├── app/                        # App Router (Next.js 13+)
│   ├── layout.tsx              # Root Layout
│   ├── page.tsx                # Home Page (/)
│   ├── globals.css             # Global CSS
│   ├── favicon.ico             # Favicon
│   ├── about/
│   │   └── page.tsx            # About Page (/about)
│   ├── blog/
│   │   ├── page.tsx            # Blog List (/blog)
│   │   └── [slug]/
│   │       └── page.tsx        # Blog Post (/blog/[slug])
│   └── api/
│       └── users/
│           └── route.ts        # API Route (/api/users)
├── components/                 # Reusable Components
│   ├── ui/
│   │   ├── Button.tsx
│   │   └── Card.tsx
│   └── layout/
│       ├── Header.tsx
│       └── Footer.tsx
├── lib/                        # Utilities / Helpers
│   ├── db.ts                   # Database Connection
│   └── utils.ts                # Utility Functions
├── public/                     # Static Files
│   ├── images/
│   └── fonts/
├── styles/                     # CSS Files
│   └── globals.css
├── types/                      # TypeScript Types
│   └── index.ts
├── next.config.js              # Next.js Config
├── tailwind.config.ts          # Tailwind Config
├── tsconfig.json               # TypeScript Config
├── .eslintrc.json              # ESLint Config
└── package.json
```

### ไฟล์สำคัญ

#### app/layout.tsx

```tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'My App',
  description: 'Generated by create next app',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th">
      <body className={inter.className}>
        {children}
      </body>
    </html>
  )
}
```

#### app/page.tsx

```tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <h1 className="text-4xl font-bold">
        Welcome to Next.js!
      </h1>
      <p className="text-lg">
        เริ่มต้นสร้าง Next.js App ของคุณ
      </p>
    </main>
  )
}
```

#### next.config.js

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable App Router (default ใน Next.js 13+)
  
  // Image Optimization
  images: {
    domains: ['example.com', 'images.unsplash.com'],
    // หรือใช้ remotePatterns
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '*.example.com',
      },
    ],
  },
  
  // Environment Variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true,
      },
    ]
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/proxy/:path*',
        destination: 'https://external-api.com/:path*',
      },
    ]
  },
}

module.exports = nextConfig
```

---

## Step 866: การรัน Development Server

### 7. การรัน Development Server

#### คำสั่งพื้นฐาน

```bash
# รัน Development Server
npm run dev        # http://localhost:3000

# Build สำหรับ Production
npm run build

# รัน Production Server (หลัง build)
npm start

# ตรวจสอบ Lint
npm run lint

# รัน Development Server บน Port อื่น
npm run dev -- -p 3001
```

#### Output ของ `npm run dev`

```
▲ Next.js 14.2.0
- Local:        http://localhost:3000
- Environments: .env.local

✓ Starting...
✓ Ready in 2.3s
```

#### Hot Module Replacement (HMR)

Next.js มี HMR ที่ช่วยให้ไม่ต้อง Reload Page เมื่อแก้ไข Code:

```
แก้ไข Component
    ↓
Next.js detect การเปลี่ยนแปลง
    ↓
อัพเดท Component ใน Browser อัตโนมัติ
    ↓
State ยังคงอยู่ (ไม่ Reset)
```

#### Environment Variables

```bash
# .env.local (สำหรับ Local Development)
DATABASE_URL=postgresql://localhost/mydb
NEXTAUTH_SECRET=your-secret-key
NEXT_PUBLIC_API_URL=https://api.example.com
```

```tsx
// การใช้งาน Environment Variables
// Server-side (ทุกตัวอ่านได้)
const dbUrl = process.env.DATABASE_URL

// Client-side (ต้องมี NEXT_PUBLIC_ นำหน้า)
const apiUrl = process.env.NEXT_PUBLIC_API_URL
```

---

## Step 868: Next.js vs Vite

### 8. Next.js vs Vite

#### Vite

```
✓ Fast Development Server (ESM-based)
✓ Hot Module Replacement เร็วมาก
✓ Build ด้วย Rollup
✓ เหมาะกับ SPA
✗ ไม่มี SSR Built-in
✗ ไม่มี File-based Routing
✗ ต้องตั้งค่า API Routes เอง
```

#### Next.js

```
✓ File-based Routing
✓ SSR, SSG, ISR Built-in
✓ API Routes Built-in
✓ Image Optimization
✓ Font Optimization
✓ SEO Built-in
✗ Learning Curve สูงกว่า
✗ อาจ Overkill สำหรับ SPA เล็กๆ
```

#### เปรียบเทียบ Dev Server Speed

```
Vite (ESM):        < 100ms startup
Next.js (Turbopack): ~300ms startup  
Next.js (Webpack):  ~1-2s startup
```

#### เปรียบเทียบ Build Time

```
Project ขนาดกลาง:
  Vite:    ~10-30 วินาที
  Next.js: ~30-60 วินาที (มี Pre-rendering)
```

---

## Step 870: Features ของ Next.js 14+

### 9. Features ของ Next.js 14+

#### Server Components (ค่าเริ่มต้นใน App Router)

```tsx
// app/page.tsx - Server Component by default
// ไม่มี 'use client'

async function Page() {
  // ทำงานที่ Server เท่านั้น
  const data = await fetch('https://api.example.com/data')
  const json = await data.json()
  
  return <div>{json.message}</div>
}
```

#### Server Actions

```tsx
// app/actions.ts
'use server'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string
  
  // Save to Database
  await db.post.create({ data: { title, content } })
}

// app/new-post/page.tsx
import { createPost } from '../actions'

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  )
}
```

#### Turbopack (Next.js 14)

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Turbopack เปิดใช้งานด้วย --turbo flag
}

// package.json
{
  "scripts": {
    "dev": "next dev --turbo"
  }
}
```

#### Partial Prerendering (PPR) - Experimental

```tsx
// app/page.tsx
import { Suspense } from 'react'
import StaticContent from './static-content'
import DynamicContent from './dynamic-content'

export default function Page() {
  return (
    <>
      {/* Static - ถูก Pre-render */}
      <StaticContent />
      
      {/* Dynamic - Load แยก */}
      <Suspense fallback={<Loading />}>
        <DynamicContent />
      </Suspense>
    </>
  )
}
```

#### next/image

```tsx
import Image from 'next/image'

export default function ProductImage() {
  return (
    <Image
      src="/product.jpg"
      alt="Product"
      width={800}
      height={600}
      priority          // Load ก่อน
      placeholder="blur" // Blur ระหว่าง Load
      blurDataURL="..."
    />
  )
}
```

#### next/font

```tsx
import { Inter, Noto_Sans_Thai } from 'next/font/google'

const inter = Inter({ 
  subsets: ['latin'],
  variable: '--font-inter'
})

const notoSansThai = Noto_Sans_Thai({
  subsets: ['thai'],
  weight: ['400', '700'],
  variable: '--font-thai'
})

export default function RootLayout({ children }) {
  return (
    <html lang="th" className={`${inter.variable} ${notoSansThai.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

#### next/link

```tsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog" prefetch={false}>Blog</Link>
      
      {/* External Link */}
      <Link href="https://nextjs.org" target="_blank" rel="noopener">
        Next.js Docs
      </Link>
    </nav>
  )
}
```

#### Metadata API

```tsx
import type { Metadata } from 'next'

// Static Metadata
export const metadata: Metadata = {
  title: 'My Blog',
  description: 'A blog about web development',
  openGraph: {
    title: 'My Blog',
    description: 'A blog about web development',
    images: ['/og-image.jpg'],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My Blog',
  },
}

// Dynamic Metadata
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug)
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage],
    },
  }
}
```

#### Caching in Next.js 14+

```tsx
// 1. Force Cache (SSG)
const data = await fetch(url, { cache: 'force-cache' })

// 2. No Store (SSR)
const data = await fetch(url, { cache: 'no-store' })

// 3. Revalidate (ISR)
const data = await fetch(url, { next: { revalidate: 3600 } })

// 4. Tag-based Revalidation
const data = await fetch(url, { next: { tags: ['products'] } })
```

---

## Step 880: ตัวอย่างโปรเจกต์แรก

### ตัวอย่าง: สร้าง Blog ง่ายๆ

#### Step 1: สร้าง Types

```typescript
// types/blog.ts
export interface Post {
  id: string
  title: string
  content: string
  createdAt: string
  author: {
    name: string
    avatar: string
  }
}
```

#### Step 2: สร้าง Data Layer

```typescript
// lib/posts.ts
import { Post } from '@/types/blog'

const POSTS: Post[] = [
  {
    id: '1',
    title: 'Getting Started with Next.js',
    content: 'Next.js is a React framework...',
    createdAt: '2024-01-01',
    author: {
      name: 'John Doe',
      avatar: '/avatars/john.jpg'
    }
  },
  {
    id: '2',
    title: 'Understanding Server Components',
    content: 'Server Components allow you to...',
    createdAt: '2024-01-02',
    author: {
      name: 'Jane Smith',
      avatar: '/avatars/jane.jpg'
    }
  }
]

export async function getPosts(): Promise<Post[]> {
  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 100))
  return POSTS
}

export async function getPost(id: string): Promise<Post | undefined> {
  return POSTS.find(post => post.id === id)
}
```

#### Step 3: สร้าง Layout

```tsx
// app/layout.tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import Link from 'next/link'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: {
    default: 'My Blog',
    template: '%s | My Blog'
  },
  description: 'A blog about web development',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th">
      <body className={inter.className}>
        <header className="bg-gray-800 text-white p-4">
          <nav className="container mx-auto flex gap-4">
            <Link href="/" className="font-bold text-xl">My Blog</Link>
            <Link href="/blog">Blog</Link>
            <Link href="/about">About</Link>
          </nav>
        </header>
        
        <main className="container mx-auto p-4 min-h-screen">
          {children}
        </main>
        
        <footer className="bg-gray-800 text-white p-4 text-center">
          <p>© 2024 My Blog</p>
        </footer>
      </body>
    </html>
  )
}
```

#### Step 4: สร้าง Home Page

```tsx
// app/page.tsx
import Link from 'next/link'
import { getPosts } from '@/lib/posts'

export default async function HomePage() {
  const posts = await getPosts()
  const recentPosts = posts.slice(0, 3)
  
  return (
    <div>
      <section className="py-12 text-center">
        <h1 className="text-4xl font-bold mb-4">
          Welcome to My Blog
        </h1>
        <p className="text-xl text-gray-600">
          เรียนรู้ Next.js และ React ด้วยกัน
        </p>
        <Link 
          href="/blog"
          className="mt-4 inline-block bg-blue-500 text-white px-6 py-2 rounded"
        >
          อ่านบทความ
        </Link>
      </section>
      
      <section>
        <h2 className="text-2xl font-bold mb-6">บทความล่าสุด</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          {recentPosts.map(post => (
            <article key={post.id} className="border rounded-lg p-4">
              <h3 className="text-xl font-semibold mb-2">
                <Link href={`/blog/${post.id}`} className="hover:underline">
                  {post.title}
                </Link>
              </h3>
              <p className="text-gray-600 text-sm">{post.createdAt}</p>
              <p className="text-gray-700 mt-2">
                {post.content.slice(0, 100)}...
              </p>
            </article>
          ))}
        </div>
      </section>
    </div>
  )
}
```

#### Step 5: สร้าง Blog List Page

```tsx
// app/blog/page.tsx
import type { Metadata } from 'next'
import Link from 'next/link'
import { getPosts } from '@/lib/posts'

export const metadata: Metadata = {
  title: 'Blog',
  description: 'รายการบทความทั้งหมด',
}

export default async function BlogPage() {
  const posts = await getPosts()
  
  return (
    <div>
      <h1 className="text-3xl font-bold mb-8">บทความทั้งหมด</h1>
      
      <div className="space-y-6">
        {posts.map(post => (
          <article key={post.id} className="border-b pb-6">
            <div className="flex items-center gap-3 mb-2">
              <img 
                src={post.author.avatar} 
                alt={post.author.name}
                className="w-8 h-8 rounded-full"
              />
              <span className="text-gray-600">{post.author.name}</span>
              <span className="text-gray-400">•</span>
              <time className="text-gray-600">{post.createdAt}</time>
            </div>
            <h2 className="text-xl font-semibold mb-2">
              <Link 
                href={`/blog/${post.id}`}
                className="hover:text-blue-500"
              >
                {post.title}
              </Link>
            </h2>
            <p className="text-gray-700">{post.content.slice(0, 200)}...</p>
          </article>
        ))}
      </div>
    </div>
  )
}
```

#### Step 6: สร้าง Blog Post Page

```tsx
// app/blog/[id]/page.tsx
import type { Metadata } from 'next'
import { notFound } from 'next/navigation'
import { getPosts, getPost } from '@/lib/posts'

// Generate Static Params for SSG
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(post => ({ id: post.id }))
}

// Generate Dynamic Metadata
export async function generateMetadata(
  { params }: { params: { id: string } }
): Promise<Metadata> {
  const post = await getPost(params.id)
  
  if (!post) return { title: 'Post Not Found' }
  
  return {
    title: post.title,
    description: post.content.slice(0, 160),
  }
}

export default async function BlogPostPage(
  { params }: { params: { id: string } }
) {
  const post = await getPost(params.id)
  
  if (!post) notFound()
  
  return (
    <article className="max-w-2xl mx-auto">
      <header className="mb-8">
        <h1 className="text-3xl font-bold mb-4">{post.title}</h1>
        <div className="flex items-center gap-3">
          <img 
            src={post.author.avatar}
            alt={post.author.name}
            className="w-10 h-10 rounded-full"
          />
          <div>
            <p className="font-medium">{post.author.name}</p>
            <time className="text-gray-500 text-sm">{post.createdAt}</time>
          </div>
        </div>
      </header>
      
      <div className="prose max-w-none">
        <p>{post.content}</p>
      </div>
    </article>
  )
}
```

---

## Step 885: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. เลือก Rendering Strategy ให้เหมาะสม

✓ หน้า Static (เช่น About Us) → SSG
✓ หน้า Dynamic (เช่น Dashboard) → CSR หรือ SSR
✓ หน้าที่ Data อัพเดทเป็นระยะ → ISR
✓ หน้าที่ต้องการ Real-time → SSR หรือ CSR

## 2. ใช้ Server Components เป็นค่าเริ่มต้น

✓ Default เป็น Server Component
✓ เพิ่ม 'use client' เมื่อต้องการ Interactivity เท่านั้น
✓ Server Components ไม่เพิ่ม JavaScript Bundle Size

## 3. Optimize Images ด้วย next/image

✓ ใช้ <Image> แทน <img>
✓ ระบุ width/height เสมอ
✓ ใช้ priority สำหรับ Above-the-fold Images

## 4. Optimize Fonts ด้วย next/font

✓ ใช้ next/font/google แทน Google Fonts CDN
✓ Font จะถูก Bundle กับ App

## 5. TypeScript

✓ ใช้ TypeScript เสมอ
✓ Type ทุก Props และ Return Values
✓ ใช้ Type จาก 'next' เช่น Metadata, NextRequest
```

---

## Quiz

### แบบทดสอบ Part 31

**คำถามที่ 1:** Next.js ใช้ Rendering Strategy อะไรสำหรับ `{ cache: 'no-store' }`?
- A) SSG
- B) SSR ✓
- C) ISR
- D) CSR

**คำถามที่ 2:** ไฟล์อะไรใน App Router ที่ทำหน้าที่เป็น Root Layout?
- A) page.tsx
- B) layout.tsx ✓
- C) template.tsx
- D) root.tsx

**คำถามที่ 3:** `next/image` ช่วยอะไร?
- A) แค่แสดงรูปภาพ
- B) Optimize รูปภาพอัตโนมัติ เช่น Resize, Format, Lazy Load ✓
- C) Upload รูปภาพไปยัง Server
- D) ไม่มีความแตกต่างจาก `<img>`

**คำถามที่ 4:** Environment Variable ที่ขึ้นต้นด้วย `NEXT_PUBLIC_` ใช้ทำอะไร?
- A) ซ่อนค่าจาก Client
- B) ทำให้ Variable เข้าถึงได้จาก Client-side Code ✓
- C) เป็น Production-only Variables
- D) ไม่มีความแตกต่างพิเศษ

**คำถามที่ 5:** ISR แตกต่างจาก SSG อย่างไร?
- A) ISR เร็วกว่า SSG
- B) ISR สร้าง HTML ตอน Build Time แต่ Regenerate ได้โดยไม่ต้อง Rebuild ทั้งหมด ✓
- C) ISR ทำงานที่ Client
- D) ไม่มีความแตกต่าง

---

## สรุป Part 31

ใน Part นี้เราได้เรียนรู้:

1. **Next.js** คือ React Framework ที่มี Feature ครบครัน
2. **SSR/SSG/ISR/CSR** แต่ละแบบเหมาะกับ Use Case ต่างกัน
3. การ**สร้าง Project** ด้วย `create-next-app`
4. **โครงสร้าง** App Router ที่ใช้ File System เป็น Routing
5. **Features** ใหม่ใน Next.js 14+ เช่น Server Actions, Turbopack

---

## แหล่งเรียนรู้เพิ่มเติม

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)
- [Next.js GitHub](https://github.com/vercel/next.js)
- [Vercel Blog](https://vercel.com/blog)

---

➡️ **Part ถัดไป:** [Part 32: Next.js App Router - การจัดการ Routing](./part-32-nextjs-routing.md)
