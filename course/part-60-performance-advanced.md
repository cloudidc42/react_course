# Part 60: Performance Advanced ใน React/Next.js

> **ระดับ:** มืออาชีพระดับโลก / World-class Professional  
> **Steps:** 1986-2025  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [Core Web Vitals](#core-web-vitals)
2. [LCP Optimization](#lcp)
3. [INP Optimization](#inp)
4. [CLS Optimization](#cls)
5. [Bundle Analysis](#bundle-analysis)
6. [Tree Shaking](#tree-shaking)
7. [Code Splitting Strategies](#code-splitting)
8. [Preloading & Prefetching](#preloading)
9. [Service Workers & PWA](#service-workers)
10. [Quiz](#quiz)

---

## Step 1986: Core Web Vitals Overview {#core-web-vitals}

Core Web Vitals คือ metrics ที่ Google ใช้วัด User Experience ของเว็บ

```
Core Web Vitals (2024):
┌─────────────────────────────────────────────────────┐
│  LCP (Largest Contentful Paint)                      │
│  ├── วัด: เวลาที่ content ใหญ่ที่สุดโหลดเสร็จ         │
│  ├── ดี: ≤ 2.5 วินาที                                │
│  ├── พอใช้: 2.5-4.0 วินาที                            │
│  └── ไม่ดี: > 4.0 วินาที                              │
│                                                      │
│  INP (Interaction to Next Paint)                     │
│  ├── วัด: เวลาตอบสนองต่อ user interaction             │
│  ├── ดี: ≤ 200ms                                     │
│  ├── พอใช้: 200-500ms                                 │
│  └── ไม่ดี: > 500ms                                  │
│                                                      │
│  CLS (Cumulative Layout Shift)                       │
│  ├── วัด: ความเสถียรของ layout                        │
│  ├── ดี: ≤ 0.1                                       │
│  ├── พอใช้: 0.1-0.25                                  │
│  └── ไม่ดี: > 0.25                                    │
└─────────────────────────────────────────────────────┘
```

### Web Vitals Monitoring

```typescript
// app/layout.tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitalsReporter() {
  useReportWebVitals((metric) => {
    const { name, value, rating, navigationType } = metric
    
    // ส่งไป Analytics
    if (window.gtag) {
      window.gtag('event', name, {
        event_category: 'Web Vitals',
        value: Math.round(name === 'CLS' ? value * 1000 : value),
        event_label: rating,
        non_interaction: true,
        navigation_type: navigationType,
      })
    }
    
    // Log ใน development
    if (process.env.NODE_ENV === 'development') {
      const emoji = rating === 'good' ? '✅' : rating === 'needs-improvement' ? '⚠️' : '❌'
      console.log(`${emoji} ${name}: ${Math.round(value)}ms (${rating})`)
    }
    
    // ส่งไป custom endpoint
    fetch('/api/vitals', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: name,
        value: Math.round(value),
        rating,
        url: window.location.pathname,
        timestamp: Date.now(),
      }),
    })
  })
  
  return null
}
```

---

## Step 1987-1993: LCP Optimization {#lcp}

LCP วัดเวลาที่ element ที่ใหญ่ที่สุดโหลดเสร็จ (ปกติคือ hero image หรือ heading)

### 1. Priority Hero Image

```typescript
// app/page.tsx
import Image from 'next/image'

export default function HomePage() {
  return (
    <>
      {/* ✅ priority = preload ใน <head> */}
      <Image
        src="/hero.jpg"
        alt="Hero"
        width={1920}
        height={1080}
        priority            // สำคัญมาก! preload LCP image
        fetchPriority="high" // browser hint
        sizes="100vw"
        quality={85}
      />
    </>
  )
}
```

### 2. Preload Critical Resources

```typescript
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        {/* Preload LCP image */}
        <link
          rel="preload"
          as="image"
          href="/hero.jpg"
          imageSrcSet="/hero-640.jpg 640w, /hero-1280.jpg 1280w, /hero-1920.jpg 1920w"
          imageSizes="100vw"
        />
        
        {/* Preload critical font */}
        <link
          rel="preload"
          as="font"
          href="/fonts/Sarabun-Regular.woff2"
          type="font/woff2"
          crossOrigin="anonymous"
        />
        
        {/* DNS Prefetch สำหรับ external domains */}
        <link rel="dns-prefetch" href="https://api.example.com" />
        <link rel="preconnect" href="https://api.example.com" crossOrigin="anonymous" />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

### 3. Optimize Server Response Time

```typescript
// app/page.tsx - Static Generation + ISR
export const revalidate = 3600 // 1 ชั่วโมง

export default async function Page() {
  // Data fetch ที่ build time (ไม่ช้าตอน request)
  const data = await fetchData()
  return <div>{/* ... */}</div>
}

// เพิ่ม Cache Headers
// next.config.ts
const config = {
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=60, stale-while-revalidate=600',
          },
        ],
      },
    ]
  },
}
```

### 4. Resource Hints

```typescript
// components/ResourceHints.tsx
export function ResourceHints() {
  return (
    <>
      {/* Preconnect to critical origins */}
      <link rel="preconnect" href="https://fonts.googleapis.com" />
      <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="" />
      
      {/* Prefetch next page user will likely visit */}
      <link rel="prefetch" href="/products" />
      
      {/* Prerender if very confident user will visit */}
      {/* <link rel="prerender" href="/checkout" /> */}
    </>
  )
}
```

---

## Step 1994-1999: INP Optimization {#inp}

INP วัดเวลาตอบสนองต่อ interaction (click, tap, keyboard)

### 1. useTransition สำหรับ Non-urgent Updates

```typescript
'use client'
import { useState, useTransition } from 'react'

export function SearchPage() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()
  
  function handleSearch(e: React.ChangeEvent<HTMLInputElement>) {
    // Urgent: update input immediately
    setQuery(e.target.value)
    
    // Non-urgent: update results (can be interrupted)
    startTransition(() => {
      const filtered = expensiveFilter(e.target.value)
      setResults(filtered)
    })
  }
  
  return (
    <div>
      <input
        value={query}
        onChange={handleSearch}
        placeholder="ค้นหา..."
      />
      
      {isPending && <div className="opacity-50">กำลังค้นหา...</div>}
      
      <ResultsList results={results} />
    </div>
  )
}
```

### 2. useDeferredValue สำหรับ Heavy Rendering

```typescript
'use client'
import { useState, useDeferredValue, memo } from 'react'

function HeavyList({ items }: { items: string[] }) {
  return (
    <ul>
      {items.map((item, i) => (
        <li key={i} className="py-1">
          {/* Heavy rendering */}
          <ComplexItem data={item} />
        </li>
      ))}
    </ul>
  )
}

const DeferredHeavyList = memo(HeavyList)

export function FilterableList({ allItems }: { allItems: string[] }) {
  const [filter, setFilter] = useState('')
  
  // Deferred - อาจแสดงค่าเก่าระหว่างที่ยุ่งอยู่
  const deferredFilter = useDeferredValue(filter)
  
  const filteredItems = allItems.filter(item =>
    item.includes(deferredFilter)
  )
  
  const isStale = deferredFilter !== filter
  
  return (
    <div>
      <input
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="กรอง..."
      />
      
      <div className={isStale ? 'opacity-50' : ''}>
        <DeferredHeavyList items={filteredItems} />
      </div>
    </div>
  )
}
```

### 3. Web Workers สำหรับ Heavy Computation

```typescript
// workers/computation.worker.ts
self.onmessage = (e) => {
  const { data, operation } = e.data
  
  let result
  switch (operation) {
    case 'sort':
      result = [...data].sort((a, b) => a - b)
      break
    case 'filter':
      result = data.filter((x: number) => x > 0)
      break
    case 'transform':
      result = data.map((x: number) => x * 2)
      break
  }
  
  self.postMessage({ result })
}

// hooks/useWorker.ts
import { useEffect, useRef, useState } from 'react'

export function useWorker<T>(workerScript: string) {
  const workerRef = useRef<Worker | null>(null)
  const [result, setResult] = useState<T | null>(null)
  const [loading, setLoading] = useState(false)
  
  useEffect(() => {
    workerRef.current = new Worker(
      new URL(workerScript, import.meta.url)
    )
    
    workerRef.current.onmessage = (e) => {
      setResult(e.data.result)
      setLoading(false)
    }
    
    return () => workerRef.current?.terminate()
  }, [workerScript])
  
  const runTask = (data: unknown, operation: string) => {
    setLoading(true)
    workerRef.current?.postMessage({ data, operation })
  }
  
  return { result, loading, runTask }
}
```

---

## Step 2000-2004: CLS Optimization {#cls}

CLS วัด layout shift ที่เกิดจาก element เด้งไปมา

```typescript
// ✅ Reserve space สำหรับ images
<div style={{ aspectRatio: '16/9' }}>
  <Image src="/photo.jpg" alt="" fill className="object-cover" />
</div>

// ✅ Reserve space สำหรับ ads
<div style={{ minHeight: '250px', minWidth: '300px' }}>
  {/* ad component */}
</div>

// ✅ Avoid inserting content above existing content
// ❌ อย่า append notification banner ด้านบน
// ✅ ใช้ fixed/sticky position แทน
<div className="fixed top-0 w-full z-50">
  <NotificationBanner />
</div>

// ✅ Font fallback ที่ match size
// globals.css
@font-face {
  font-family: 'Sarabun';
  src: url('/fonts/Sarabun-Regular.woff2') format('woff2');
  font-display: swap;
  /* Adjust fallback font metrics */
  ascent-override: 95%;
  descent-override: 25%;
  line-gap-override: 0%;
}
```

---

## Step 2005-2010: Bundle Analysis {#bundle-analysis}

```bash
# Install Bundle Analyzer
npm install -D @next/bundle-analyzer

# next.config.ts
import withBundleAnalyzer from '@next/bundle-analyzer'

const withAnalyzer = withBundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
})

export default withAnalyzer({
  // your next config
})

# Run analysis
ANALYZE=true npm run build
```

```typescript
// next.config.ts - เพิ่มเติม
const config: NextConfig = {
  // แสดง bundle stats
  experimental: {
    // ดู build output ที่ละเอียด
  },
  
  // Webpack bundle analyzer
  webpack: (config, { buildId, dev, isServer }) => {
    if (process.env.ANALYZE) {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer')
      config.plugins.push(
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          reportFilename: isServer
            ? '../analyze/server.html'
            : './analyze/client.html',
        })
      )
    }
    return config
  },
}
```

### Size Limit Check

```json
// package.json
{
  "scripts": {
    "size": "size-limit",
    "analyze": "size-limit --why"
  },
  "size-limit": [
    {
      "path": ".next/static/chunks/pages/index*.js",
      "limit": "50 KB"
    },
    {
      "path": ".next/static/chunks/main*.js",
      "limit": "100 KB"
    }
  ]
}
```

---

## Step 2011-2015: Tree Shaking {#tree-shaking}

Tree Shaking คือการตัด code ที่ไม่ได้ใช้ออกจาก bundle

```typescript
// ❌ Import ทั้ง library (ใหญ่)
import _ from 'lodash'
const result = _.sortBy(items, 'name')

// ✅ Named import (tree-shakeable)
import { sortBy } from 'lodash-es'
const result = sortBy(items, 'name')

// ✅ ดีกว่า: ใช้ native JS
const result = [...items].sort((a, b) => a.name.localeCompare(b.name))

// ❌ Import ทั้ง icon library
import { FaHome, FaUser, FaCart } from 'react-icons/fa'

// ✅ Import แยกไฟล์
import { FaHome } from 'react-icons/fa/index'

// หรือใช้ lucide-react ที่ tree-shakeable
import { Home, User, ShoppingCart } from 'lucide-react'
```

### Package.json Sideeffects

```json
// package.json ใน library ของเรา
{
  "name": "my-ui-lib",
  "sideEffects": false,  // บอก webpack ว่าไม่มี side effects
  // หรือ
  "sideEffects": ["*.css", "*.scss"]  // เฉพาะ CSS มี side effects
}
```

---

## Step 2016-2020: Code Splitting {#code-splitting}

```typescript
// 1. Route-based splitting (อัตโนมัติใน Next.js)
// ทุก page ใน app/ จะถูก split อัตโนมัติ

// 2. Dynamic Import
import dynamic from 'next/dynamic'

// Component ที่โหลดแบบ lazy
const HeavyChart = dynamic(
  () => import('@/components/HeavyChart'),
  {
    loading: () => <ChartSkeleton />,
    ssr: false,  // ปิด SSR สำหรับ browser-only components
  }
)

// Modal ที่โหลดเมื่อเปิดเท่านั้น
const PaymentModal = dynamic(() => import('@/components/PaymentModal'))

export default function Dashboard() {
  const [showPayment, setShowPayment] = useState(false)
  
  return (
    <div>
      <button onClick={() => setShowPayment(true)}>ชำระเงิน</button>
      
      {/* โหลด PaymentModal เมื่อต้องการเท่านั้น */}
      {showPayment && (
        <PaymentModal onClose={() => setShowPayment(false)} />
      )}
      
      {/* HeavyChart โหลดแบบ lazy เสมอ */}
      <HeavyChart data={chartData} />
    </div>
  )
}

// 3. Conditional Split
const AdminPanel = dynamic(
  () => import('@/components/AdminPanel'),
  { ssr: false }
)

export function Layout({ user, children }) {
  return (
    <div>
      {children}
      {/* โหลดเฉพาะ admin */}
      {user.role === 'admin' && <AdminPanel />}
    </div>
  )
}
```

---

## Step 2021-2022: Preloading & Prefetching {#preloading}

```typescript
// next/link prefetch (default: true)
import Link from 'next/link'

// Prefetch on hover/viewport (default behavior)
<Link href="/products">ดูสินค้า</Link>

// ปิด prefetch สำหรับ pages ที่ไม่ค่อยใช้
<Link href="/admin/reports" prefetch={false}>Reports</Link>

// Manual prefetch
import { useRouter } from 'next/navigation'

function ProductCard({ product }) {
  const router = useRouter()
  
  // Prefetch เมื่อ hover
  const handleMouseEnter = () => {
    router.prefetch(`/products/${product.id}`)
  }
  
  return (
    <div onMouseEnter={handleMouseEnter}>
      <Link href={`/products/${product.id}`}>
        {product.name}
      </Link>
    </div>
  )
}

// Preload data
// app/products/page.tsx
export default async function ProductsPage() {
  // ดึงข้อมูลพร้อมกัน
  const [products, categories] = await Promise.all([
    fetchProducts(),
    fetchCategories(),
  ])
  
  return <ProductList products={products} categories={categories} />
}
```

---

## Step 2023-2025: Service Workers & PWA {#service-workers}

```bash
# Install next-pwa
npm install next-pwa
npm install -D @types/workbox-window
```

```typescript
// next.config.ts
import withPWA from 'next-pwa'

const config = withPWA({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development',
  
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.googleapis\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'google-fonts-cache',
        expiration: { maxEntries: 10, maxAgeSeconds: 365 * 24 * 60 * 60 },
        cacheableResponse: { statuses: [0, 200] },
      },
    },
    {
      urlPattern: /^https:\/\/api\.example\.com\/products\/.*/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'api-cache',
        expiration: { maxEntries: 100, maxAgeSeconds: 60 * 60 },
      },
    },
  ],
})({
  // your next config
})

export default config
```

### Web App Manifest

```json
// public/manifest.json
{
  "name": "My React App",
  "short_name": "ReactApp",
  "description": "แอปพลิเคชัน React ที่ยอดเยี่ยม",
  "theme_color": "#000000",
  "background_color": "#ffffff",
  "display": "standalone",
  "orientation": "portrait",
  "scope": "/",
  "start_url": "/",
  "icons": [
    { "src": "/icons/icon-72x72.png", "sizes": "72x72", "type": "image/png" },
    { "src": "/icons/icon-96x96.png", "sizes": "96x96", "type": "image/png" },
    { "src": "/icons/icon-192x192.png", "sizes": "192x192", "type": "image/png", "purpose": "maskable" },
    { "src": "/icons/icon-512x512.png", "sizes": "512x512", "type": "image/png", "purpose": "any" }
  ],
  "shortcuts": [
    {
      "name": "สั่งซื้อ",
      "short_name": "สั่งซื้อ",
      "description": "สั่งซื้อสินค้าใหม่",
      "url": "/order/new",
      "icons": [{ "src": "/icons/new-order.png", "sizes": "96x96" }]
    }
  ]
}
```

```typescript
// app/layout.tsx - Register Manifest
export const metadata = {
  manifest: '/manifest.json',
  themeColor: '#000000',
  appleWebApp: {
    capable: true,
    statusBarStyle: 'default',
    title: 'My App',
  },
  viewport: {
    width: 'device-width',
    initialScale: 1,
    maximumScale: 1,
  },
}
```

---

## 🧪 Quiz - Part 60

**ข้อ 1:** LCP ย่อมาจากอะไร และ threshold ที่ "ดี" คืออะไร?
- A) Largest Content Paint, < 1 วินาที
- B) Largest Contentful Paint, ≤ 2.5 วินาที
- C) Latest Content Painting, < 3 วินาที
- D) Longest Contentful Paint, ≤ 4 วินาที

**ข้อ 2:** `useTransition` ใช้เพื่ออะไร?
- A) เพิ่ม CSS animation
- B) Mark state updates ที่ไม่ urgent เพื่อ improve INP
- C) Defer network requests
- D) Preload components

**ข้อ 3:** Tree Shaking คืออะไร?
- A) การแบ่ง code เป็น chunks
- B) การลบ code ที่ไม่ได้ใช้ออกจาก bundle
- C) การ compress ไฟล์
- D) การ cache JavaScript

**ข้อ 4:** `dynamic(() => import('...'), { ssr: false })` ทำอะไร?
- A) Preload component
- B) Import component แบบ lazy ที่ไม่ render บน server
- C) Import component แบบ eager
- D) Cache component

**เฉลย:** 1-B, 2-B, 3-B, 4-B

---

## 🎓 จบหลักสูตร React/Next.js ระดับมืออาชีพ!

ยินดีด้วยที่คุณเรียนจบทั้ง 60 Parts! คุณได้เรียนรู้:

- React Fundamentals → Advanced Patterns
- Next.js 14 App Router → Edge Runtime
- Testing → CI/CD → Performance
- Security → PWA → Production Deploy

---

> **⬅️ Previous:** [Part 59: Security Best Practices](./part-59-security-best-practices.md)
