# Part 46: SEO Optimization ใน Next.js 14

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1441-1475  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [SEO คืออะไรและทำไมถึงสำคัญ](#seo-คืออะไร)
2. [Metadata API ใน Next.js 14](#metadata-api)
3. [Static Metadata](#static-metadata)
4. [Dynamic Metadata](#dynamic-metadata)
5. [Open Graph Tags](#open-graph-tags)
6. [Twitter Cards](#twitter-cards)
7. [Structured Data (JSON-LD)](#structured-data)
8. [Sitemap.xml](#sitemapxml)
9. [robots.txt](#robotstxt)
10. [Canonical URLs](#canonical-urls)
11. [Core Web Vitals](#core-web-vitals)
12. [Quiz](#quiz)

---

## Step 1441: SEO คืออะไรและทำไมถึงสำคัญ {#seo-คืออะไร}

SEO (Search Engine Optimization) คือกระบวนการปรับปรุงเว็บไซต์ให้ติดอันดับสูงในผลการค้นหาของ Search Engine เช่น Google, Bing หรือ Yahoo

### ทำไม SEO ถึงสำคัญ?

```
การเข้าชมเว็บไซต์ส่วนใหญ่มาจาก Organic Search
├── 53% ของการเข้าชมเว็บไซต์มาจาก Organic Search
├── 91% ของ Content ไม่ได้รับ Traffic จาก Google เลย
└── อันดับ 1 ใน Google ได้รับ Click-through rate สูงถึง 28.5%
```

### SEO สำหรับ React/Next.js

React เป็น Single Page Application (SPA) ซึ่งมีปัญหาเรื่อง SEO เพราะ:
- Search engine crawlers อาจไม่ execute JavaScript
- ไม่มี HTML ที่ถูก render ไว้ล่วงหน้า

**Next.js แก้ปัญหานี้ด้วย:**
- Server-Side Rendering (SSR)
- Static Site Generation (SSG)
- Incremental Static Regeneration (ISR)

---

## Step 1442: Metadata API ใน Next.js 14 {#metadata-api}

Next.js 14 มี Metadata API ใหม่ที่ทรงพลัง สามารถกำหนดได้ใน `layout.tsx` หรือ `page.tsx`

### วิธีการกำหนด Metadata

**วิธีที่ 1: Static Export**

```typescript
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'My App',
  description: 'This is my amazing application',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th">
      <body>{children}</body>
    </html>
  )
}
```

**วิธีที่ 2: generateMetadata Function**

```typescript
// app/blog/[slug]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next'

type Props = {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const slug = params.slug
  
  // ดึงข้อมูลจาก API
  const post = await fetchPost(slug)
  
  // เข้าถึง parent metadata
  const previousImages = (await parent).openGraph?.images || []
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage, ...previousImages],
    },
  }
}

export default function BlogPost({ params }: Props) {
  return <div>Blog Post: {params.slug}</div>
}
```

---

## Step 1443: Static Metadata {#static-metadata}

Static Metadata เหมาะสำหรับหน้าที่ข้อมูลไม่เปลี่ยนแปลง

```typescript
// app/about/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    absolute: 'เกี่ยวกับเรา | My App',
    // หรือใช้ template
    // template: '%s | My App',
    // default: 'My App',
  },
  description: 'เรียนรู้เพิ่มเติมเกี่ยวกับบริษัทของเรา',
  keywords: ['about', 'company', 'team'],
  authors: [{ name: 'John Doe', url: 'https://example.com' }],
  creator: 'John Doe',
  publisher: 'My Company',
  
  // Robots
  robots: {
    index: true,
    follow: true,
    nocache: false,
    googleBot: {
      index: true,
      follow: true,
      noimageindex: false,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  
  // Canonical
  alternates: {
    canonical: 'https://example.com/about',
    languages: {
      'en-US': 'https://example.com/en/about',
      'th-TH': 'https://example.com/th/about',
    },
  },
}
```

### Title Template

```typescript
// app/layout.tsx - Root Layout
export const metadata: Metadata = {
  title: {
    template: '%s | My Website',
    default: 'My Website', // fallback สำหรับหน้าที่ไม่มี title
  },
}

// app/blog/page.tsx - Child Page
export const metadata: Metadata = {
  title: 'Blog', // จะกลายเป็น "Blog | My Website"
}

// app/blog/[slug]/page.tsx - Dynamic Page
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug)
  return {
    title: post.title, // จะกลายเป็น "Post Title | My Website"
  }
}
```

---

## Step 1444: Dynamic Metadata {#dynamic-metadata}

Dynamic Metadata ใช้เมื่อต้องการดึงข้อมูลจาก API หรือ Database

```typescript
// app/products/[id]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next'
import { notFound } from 'next/navigation'

interface Product {
  id: number
  name: string
  description: string
  image: string
  price: number
  category: string
}

async function getProduct(id: string): Promise<Product | null> {
  try {
    const res = await fetch(`https://api.example.com/products/${id}`, {
      next: { revalidate: 3600 }, // Cache 1 hour
    })
    
    if (!res.ok) return null
    return res.json()
  } catch {
    return null
  }
}

export async function generateMetadata(
  { params }: { params: { id: string } },
  parent: ResolvingMetadata
): Promise<Metadata> {
  const product = await getProduct(params.id)
  
  if (!product) {
    return {
      title: 'Product Not Found',
    }
  }
  
  const previousImages = (await parent).openGraph?.images || []
  
  return {
    title: `${product.name} - ราคา ฿${product.price}`,
    description: product.description,
    openGraph: {
      title: product.name,
      description: product.description,
      type: 'website',
      images: [
        {
          url: product.image,
          width: 1200,
          height: 630,
          alt: product.name,
        },
        ...previousImages,
      ],
    },
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
      images: [product.image],
    },
    alternates: {
      canonical: `https://example.com/products/${params.id}`,
    },
  }
}

export default async function ProductPage({
  params,
}: {
  params: { id: string }
}) {
  const product = await getProduct(params.id)
  
  if (!product) {
    notFound()
  }
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>ราคา: ฿{product.price}</p>
    </div>
  )
}
```

---

## Step 1445: Open Graph Tags {#open-graph-tags}

Open Graph Protocol ช่วยให้เว็บไซต์ถูก share บน Social Media ได้สวยงาม

```typescript
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug)
  
  return {
    openGraph: {
      // ประเภทพื้นฐาน
      type: 'article',
      
      // ข้อมูลพื้นฐาน
      title: post.title,
      description: post.excerpt,
      url: `https://example.com/blog/${params.slug}`,
      siteName: 'My Blog',
      locale: 'th_TH',
      
      // รูปภาพ
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
          type: 'image/jpeg',
        },
        {
          url: post.thumbnailImage,
          width: 400,
          height: 300,
          alt: `${post.title} thumbnail`,
        }
      ],
      
      // สำหรับ Article
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      section: post.category,
      tags: post.tags,
    },
  }
}
```

### Open Graph สำหรับแต่ละประเภท

```typescript
// Website
openGraph: {
  type: 'website',
  url: 'https://example.com',
  title: 'My Website',
  description: 'Welcome to my website',
  siteName: 'My Website',
  images: [{ url: '/og-image.jpg' }],
}

// Article
openGraph: {
  type: 'article',
  publishedTime: '2024-01-01T00:00:00.000Z',
  modifiedTime: '2024-01-15T00:00:00.000Z',
  expirationTime: '2025-01-01T00:00:00.000Z',
  authors: ['Author Name'],
  section: 'Technology',
  tags: ['react', 'nextjs', 'seo'],
}

// Book
openGraph: {
  type: 'book',
  releaseDate: '2024-01-01T00:00:00.000Z',
  isbn: '978-3-16-148410-0',
  authors: ['Author Name'],
  tags: ['programming', 'javascript'],
}

// Profile
openGraph: {
  type: 'profile',
  firstName: 'John',
  lastName: 'Doe',
  username: 'johndoe',
  gender: 'male',
}

// Music
openGraph: {
  type: 'music.song',
  duration: 300,
  albums: [{ url: 'https://example.com/album/1' }],
  musicians: [{ url: 'https://example.com/musician/1' }],
}

// Video
openGraph: {
  type: 'video.movie',
  actors: [{ url: 'https://example.com/actor/1', role: 'Hero' }],
  directors: [{ url: 'https://example.com/director/1' }],
  writers: [{ url: 'https://example.com/writer/1' }],
  duration: 7200,
  releaseDate: '2024-01-01T00:00:00.000Z',
  tags: ['action', 'adventure'],
}
```

---

## Step 1446: Twitter Cards {#twitter-cards}

Twitter Cards ช่วยให้ tweet แสดงข้อมูลเว็บไซต์ได้สวยงาม

```typescript
export const metadata: Metadata = {
  twitter: {
    // ประเภท Card
    card: 'summary_large_image', // summary | summary_large_image | app | player
    
    // ข้อมูลพื้นฐาน
    title: 'My Page Title',
    description: 'My page description',
    
    // Creator/Site
    site: '@mysite',         // Twitter handle ของ website
    creator: '@johndoe',     // Twitter handle ของผู้สร้าง content
    
    // รูปภาพ
    images: ['https://example.com/twitter-card.jpg'],
  },
}
```

### Twitter Card Types

**1. summary (เล็ก)**
```typescript
twitter: {
  card: 'summary',
  title: 'Short Title',
  description: 'Short description',
  images: ['https://example.com/small-image.jpg'], // 1:1 ratio
}
```

**2. summary_large_image (ใหญ่)**
```typescript
twitter: {
  card: 'summary_large_image',
  title: 'Article Title',
  description: 'Article description',
  images: ['https://example.com/large-image.jpg'], // 2:1 ratio
  creator: '@author',
}
```

**3. app (สำหรับ App)**
```typescript
twitter: {
  card: 'app',
  app: {
    name: {
      iphone: 'My iPhone App',
      ipad: 'My iPad App',
      googleplay: 'My Android App',
    },
    id: {
      iphone: '123456789',
      ipad: '123456789',
      googleplay: 'com.example.app',
    },
    url: {
      iphone: 'myapp://page',
      ipad: 'myapp://page',
    },
  },
}
```

**4. player (สำหรับ Video/Audio)**
```typescript
twitter: {
  card: 'player',
  title: 'My Video',
  description: 'Watch my video',
  images: ['https://example.com/thumbnail.jpg'],
  players: [
    {
      playerUrl: 'https://example.com/player?id=123',
      streamUrl: 'https://example.com/stream.mp4',
      width: 1280,
      height: 720,
    }
  ],
}
```

---

## Step 1447: Structured Data (JSON-LD) {#structured-data}

Structured Data ช่วยให้ Google เข้าใจ content ได้ดีขึ้นและแสดง Rich Snippets

```typescript
// app/blog/[slug]/page.tsx
import { WithContext, Article } from 'schema-dts'

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  
  // JSON-LD Schema
  const jsonLd: WithContext<Article> = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author.name,
      url: `https://example.com/authors/${post.author.slug}`,
    },
    publisher: {
      '@type': 'Organization',
      name: 'My Blog',
      logo: {
        '@type': 'ImageObject',
        url: 'https://example.com/logo.png',
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `https://example.com/blog/${params.slug}`,
    },
  }
  
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>
        <h1>{post.title}</h1>
        {/* ... */}
      </article>
    </>
  )
}
```

### Schema Types ต่างๆ

**Product Schema**
```typescript
const productSchema = {
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: 'Product Name',
  description: 'Product description',
  image: 'https://example.com/product.jpg',
  sku: 'SKU-001',
  brand: {
    '@type': 'Brand',
    name: 'Brand Name',
  },
  offers: {
    '@type': 'Offer',
    price: '999.00',
    priceCurrency: 'THB',
    availability: 'https://schema.org/InStock',
    seller: {
      '@type': 'Organization',
      name: 'My Store',
    },
  },
  aggregateRating: {
    '@type': 'AggregateRating',
    ratingValue: '4.5',
    reviewCount: '100',
  },
}
```

**FAQ Schema**
```typescript
const faqSchema = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: [
    {
      '@type': 'Question',
      name: 'คำถามที่ 1?',
      acceptedAnswer: {
        '@type': 'Answer',
        text: 'คำตอบที่ 1',
      },
    },
    {
      '@type': 'Question',
      name: 'คำถามที่ 2?',
      acceptedAnswer: {
        '@type': 'Answer',
        text: 'คำตอบที่ 2',
      },
    },
  ],
}
```

**BreadcrumbList Schema**
```typescript
const breadcrumbSchema = {
  '@context': 'https://schema.org',
  '@type': 'BreadcrumbList',
  itemListElement: [
    {
      '@type': 'ListItem',
      position: 1,
      name: 'Home',
      item: 'https://example.com',
    },
    {
      '@type': 'ListItem',
      position: 2,
      name: 'Blog',
      item: 'https://example.com/blog',
    },
    {
      '@type': 'ListItem',
      position: 3,
      name: 'Post Title',
      item: 'https://example.com/blog/post-title',
    },
  ],
}
```

**Organization Schema**
```typescript
const organizationSchema = {
  '@context': 'https://schema.org',
  '@type': 'Organization',
  name: 'My Company',
  url: 'https://example.com',
  logo: 'https://example.com/logo.png',
  description: 'Company description',
  address: {
    '@type': 'PostalAddress',
    streetAddress: '123 Main St',
    addressLocality: 'Bangkok',
    addressCountry: 'TH',
    postalCode: '10110',
  },
  contactPoint: [
    {
      '@type': 'ContactPoint',
      telephone: '+66-2-123-4567',
      contactType: 'customer service',
      availableLanguage: ['Thai', 'English'],
    },
  ],
  sameAs: [
    'https://www.facebook.com/mycompany',
    'https://twitter.com/mycompany',
    'https://www.linkedin.com/company/mycompany',
  ],
}
```

**Event Schema**
```typescript
const eventSchema = {
  '@context': 'https://schema.org',
  '@type': 'Event',
  name: 'React Conference Thailand 2024',
  description: 'งาน Conference สำหรับ React Developer',
  startDate: '2024-06-15T09:00:00+07:00',
  endDate: '2024-06-15T17:00:00+07:00',
  location: {
    '@type': 'Place',
    name: 'IMPACT Arena',
    address: {
      '@type': 'PostalAddress',
      streetAddress: 'IMPACT Arena, Muang Thong Thani',
      addressLocality: 'Nonthaburi',
      addressCountry: 'TH',
    },
  },
  organizer: {
    '@type': 'Organization',
    name: 'React Thailand',
    url: 'https://reactthailand.com',
  },
  offers: {
    '@type': 'Offer',
    url: 'https://example.com/tickets',
    price: '500',
    priceCurrency: 'THB',
    availability: 'https://schema.org/InStock',
    validFrom: '2024-01-01T00:00:00+07:00',
  },
}
```

---

## Step 1448: Sitemap.xml {#sitemapxml}

Sitemap บอกให้ Search Engine รู้ว่ามีหน้าอะไรในเว็บไซต์

### Static Sitemap

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.5,
    },
  ]
}
```

### Dynamic Sitemap

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next'

async function getAllPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 },
  })
  return res.json()
}

async function getAllProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 },
  })
  return res.json()
}

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const [posts, products] = await Promise.all([
    getAllPosts(),
    getAllProducts(),
  ])
  
  const postUrls = posts.map((post: any) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }))
  
  const productUrls = products.map((product: any) => ({
    url: `https://example.com/products/${product.id}`,
    lastModified: new Date(product.updatedAt),
    changeFrequency: 'daily' as const,
    priority: 0.9,
  }))
  
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8,
    },
    {
      url: 'https://example.com/products',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 0.9,
    },
    ...postUrls,
    ...productUrls,
  ]
}
```

### Multiple Sitemaps

```typescript
// app/sitemap/[id]/route.ts
import { MetadataRoute } from 'next'

const POSTS_PER_SITEMAP = 1000

export async function generateStaticParams() {
  const totalPosts = await getTotalPosts()
  const totalSitemaps = Math.ceil(totalPosts / POSTS_PER_SITEMAP)
  
  return Array.from({ length: totalSitemaps }, (_, i) => ({
    id: String(i + 1),
  }))
}

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const id = parseInt(params.id)
  const offset = (id - 1) * POSTS_PER_SITEMAP
  
  const posts = await getPosts({ skip: offset, take: POSTS_PER_SITEMAP })
  
  const sitemap = posts.map((post: any) => `
    <url>
      <loc>https://example.com/blog/${post.slug}</loc>
      <lastmod>${new Date(post.updatedAt).toISOString()}</lastmod>
      <changefreq>weekly</changefreq>
      <priority>0.7</priority>
    </url>
  `).join('')
  
  return new Response(
    `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${sitemap}
    </urlset>`,
    {
      headers: {
        'Content-Type': 'application/xml',
      },
    }
  )
}
```

### Sitemap Index

```typescript
// app/sitemap-index.xml/route.ts
export async function GET() {
  const totalPosts = await getTotalPosts()
  const totalSitemaps = Math.ceil(totalPosts / 1000)
  
  const sitemaps = Array.from({ length: totalSitemaps }, (_, i) => `
    <sitemap>
      <loc>https://example.com/sitemap/${i + 1}</loc>
      <lastmod>${new Date().toISOString()}</lastmod>
    </sitemap>
  `).join('')
  
  return new Response(
    `<?xml version="1.0" encoding="UTF-8"?>
    <sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${sitemaps}
    </sitemapindex>`,
    {
      headers: {
        'Content-Type': 'application/xml',
      },
    }
  )
}
```

---

## Step 1449: robots.txt {#robotstxt}

robots.txt บอก Search Engine ว่าหน้าไหนที่สามารถ crawl ได้

### Static robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/private/', '/admin/', '/api/'],
    },
    sitemap: 'https://example.com/sitemap.xml',
    host: 'https://example.com',
  }
}
```

### Advanced robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        // กฎทั่วไปสำหรับทุก Bot
        userAgent: '*',
        allow: '/',
        disallow: [
          '/private/',
          '/admin/',
          '/api/',
          '/draft/',
          '/*?*', // Block URL ที่มี query parameters
        ],
      },
      {
        // กฎเฉพาะสำหรับ Googlebot
        userAgent: 'Googlebot',
        allow: '/',
        disallow: ['/private/'],
        crawlDelay: 2,
      },
      {
        // Block บาง Bot อย่างสมบูรณ์
        userAgent: [
          'AhrefsBot',
          'SemrushBot',
          'MJ12bot',
        ],
        disallow: '/',
      },
    ],
    sitemap: [
      'https://example.com/sitemap.xml',
      'https://example.com/blog-sitemap.xml',
    ],
    host: 'https://example.com',
  }
}
```

> **Output ที่ได้:**
> ```txt
> User-agent: *
> Allow: /
> Disallow: /private/
> Disallow: /admin/
> Disallow: /api/
> 
> User-agent: Googlebot
> Allow: /
> Disallow: /private/
> Crawl-delay: 2
> 
> User-agent: AhrefsBot
> Disallow: /
> 
> Sitemap: https://example.com/sitemap.xml
> Host: https://example.com
> ```

---

## Step 1450: Canonical URLs {#canonical-urls}

Canonical URL บอก Search Engine ว่า URL ไหนคือ URL หลักของ content นั้น

### เมื่อไรต้องใช้ Canonical

```
ปัญหา Duplicate Content:
├── https://example.com/products/shoe
├── https://example.com/products/shoe?color=red
├── https://example.com/products/shoe?size=40
└── https://example.com/shoes/shoe (ซ้ำกัน)

→ ทุก URL ควร canonical ไปที่ https://example.com/products/shoe
```

### วิธีกำหนด Canonical

```typescript
// app/products/[id]/page.tsx
export const metadata: Metadata = {
  alternates: {
    canonical: `https://example.com/products/${id}`,
  },
}
```

### Dynamic Canonical

```typescript
// app/products/[id]/page.tsx
export async function generateMetadata({ params }) {
  const product = await getProduct(params.id)
  
  return {
    alternates: {
      canonical: `https://example.com/products/${product.slug}`,
      languages: {
        'en-US': `https://example.com/en/products/${product.slug}`,
        'th-TH': `https://example.com/th/products/${product.slug}`,
      },
    },
  }
}
```

### Pagination Canonical

```typescript
// app/blog/page.tsx?page=2
// ทุกหน้าควร canonical ไปยัง /blog
export async function generateMetadata({ searchParams }) {
  const page = searchParams.page || 1
  
  return {
    alternates: {
      canonical: 'https://example.com/blog',
    },
    // ถ้าต้องการ paginated canonical
    // canonical: `https://example.com/blog?page=${page}`,
  }
}
```

---

## Step 1451: Core Web Vitals {#core-web-vitals}

Core Web Vitals คือ metrics ที่ Google ใช้วัดประสิทธิภาพและ UX ของเว็บไซต์

### 3 Core Web Vitals หลัก

```
LCP (Largest Contentful Paint)
├── วัด: เวลาที่ใช้ในการโหลด element ที่ใหญ่ที่สุด
├── ดี: ≤ 2.5 วินาที
├── ต้องปรับปรุง: 2.5s - 4s
└── แย่: > 4s

FID (First Input Delay) → ถูกแทนที่ด้วย INP
├── วัด: เวลาตอบสนองต่อการกระทำแรกของผู้ใช้
├── ดี: ≤ 100ms
└── แย่: > 300ms

INP (Interaction to Next Paint) - ใหม่ใน 2024
├── วัด: การตอบสนองต่อทุก interaction
├── ดี: ≤ 200ms
├── ต้องปรับปรุง: 200ms - 500ms
└── แย่: > 500ms

CLS (Cumulative Layout Shift)
├── วัด: ความเสถียรของ Layout ขณะโหลด
├── ดี: ≤ 0.1
├── ต้องปรับปรุง: 0.1 - 0.25
└── แย่: > 0.25
```

### วัด Core Web Vitals ใน Next.js

```typescript
// app/layout.tsx หรือ pages/_app.tsx
import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'FCP':
        // First Contentful Paint
        console.log('FCP:', metric.value)
        break
      case 'LCP':
        // Largest Contentful Paint
        console.log('LCP:', metric.value)
        break
      case 'CLS':
        // Cumulative Layout Shift
        console.log('CLS:', metric.value)
        break
      case 'FID':
        // First Input Delay
        console.log('FID:', metric.value)
        break
      case 'TTFB':
        // Time to First Byte
        console.log('TTFB:', metric.value)
        break
      case 'INP':
        // Interaction to Next Paint
        console.log('INP:', metric.value)
        break
    }
    
    // ส่งไปยัง Analytics
    sendToAnalytics(metric)
  })
  
  return null
}

function sendToAnalytics(metric: any) {
  const body = JSON.stringify(metric)
  
  // ใช้ sendBeacon เพื่อไม่ block navigation
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics', body)
  } else {
    fetch('/api/analytics', {
      body,
      method: 'POST',
      keepalive: true,
    })
  }
}
```

### การปรับปรุง LCP

```typescript
// 1. Priority Image
import Image from 'next/image'

export default function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero Image"
      width={1200}
      height={600}
      priority  // โหลดก่อนเลย ไม่รอ lazy load
      quality={85}
    />
  )
}

// 2. Preload Resource
// app/layout.tsx
export const metadata: Metadata = {
  // preload font
}

// หรือใน head
<link 
  rel="preload" 
  href="/fonts/custom.woff2" 
  as="font" 
  type="font/woff2" 
  crossOrigin="anonymous"
/>

// 3. Resource Hints
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="" />
<link rel="dns-prefetch" href="https://api.example.com" />
```

### การแก้ปัญหา CLS

```css
/* 1. กำหนด width/height ให้รูปภาพเสมอ */
img {
  aspect-ratio: 16 / 9;
  width: 100%;
  height: auto;
}

/* 2. Reserve space สำหรับ Dynamic Content */
.ad-container {
  min-height: 250px;
}

/* 3. Font Face - กำหนด font-display */
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/my-font.woff2') format('woff2');
  font-display: swap; /* หรือ optional, fallback */
}
```

```typescript
// 4. Skeleton Loading
function ProductSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="bg-gray-200 h-48 w-full rounded-lg mb-4" />
      <div className="bg-gray-200 h-4 w-3/4 rounded mb-2" />
      <div className="bg-gray-200 h-4 w-1/2 rounded" />
    </div>
  )
}

// 5. ใช้ Suspense
<Suspense fallback={<ProductSkeleton />}>
  <ProductList />
</Suspense>
```

---

## Step 1452-1455: SEO Best Practices เพิ่มเติม

### Meta Description ที่ดี

```typescript
// ✅ ดี: 150-160 ตัวอักษร, มี keyword, ดึงดูด
export const metadata: Metadata = {
  description: 'เรียน React และ Next.js จากศูนย์ถึงระดับมืออาชีพ พร้อม Project จริง Certificate และรับประกันงาน สมัครได้เลยวันนี้!',
}

// ❌ แย่: สั้นเกินไป
export const metadata: Metadata = {
  description: 'React Course',
}

// ❌ แย่: ยาวเกินไป
export const metadata: Metadata = {
  description: 'เรียน React และ Next.js จากศูนย์ถึงระดับมืออาชีพ พร้อม Project จริง Certificate และรับประกันงาน สมัครได้เลยวันนี้! เนื้อหาครอบคลุมทุกหัวข้อ ตั้งแต่พื้นฐาน JSX, Components, State, Props จนถึง Server Components, TypeScript, Testing',
}
```

### URL Structure ที่ดี

```
✅ URL ที่ดี:
├── https://example.com/blog/how-to-learn-react
├── https://example.com/products/iphone-15-pro-max
└── https://example.com/categories/smartphones

❌ URL ที่ไม่ดี:
├── https://example.com/blog?id=12345
├── https://example.com/p?cat=5&id=789
└── https://example.com/product_detail.php?product_id=456
```

### Internal Linking

```typescript
// components/RelatedPosts.tsx
import Link from 'next/link'

interface Post {
  slug: string
  title: string
  excerpt: string
}

export function RelatedPosts({ posts }: { posts: Post[] }) {
  return (
    <section aria-labelledby="related-posts-title">
      <h2 id="related-posts-title">บทความที่เกี่ยวข้อง</h2>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link 
              href={`/blog/${post.slug}`}
              // Descriptive anchor text - ดีกว่า "คลิกที่นี่"
            >
              {post.title}
            </Link>
            <p>{post.excerpt}</p>
          </li>
        ))}
      </ul>
    </section>
  )
}
```

### Image SEO

```typescript
// ✅ Best Practices สำหรับ Image SEO
import Image from 'next/image'

export function ProductImage({ product }) {
  return (
    <Image
      src={product.image}
      // Alt text ที่อธิบายรูปภาพอย่างชัดเจน
      alt={`${product.name} - สีดำ 256GB`}
      width={800}
      height={600}
      // ไฟล์ชื่อ descriptive: iphone-15-pro-max-black.jpg
      // ไม่ใช่: DSC_001234.jpg
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      quality={80}
    />
  )
}
```

### Heading Structure

```tsx
// ✅ ดี: มี hierarchy ที่ถูกต้อง
<main>
  <h1>หัวข้อหลักของหน้า (มีแค่ 1 ต่อหน้า)</h1>
  <section>
    <h2>หัวข้อหมวดหมู่ที่ 1</h2>
    <h3>หัวข้อย่อย</h3>
    <h3>หัวข้อย่อย</h3>
  </section>
  <section>
    <h2>หัวข้อหมวดหมู่ที่ 2</h2>
    <h3>หัวข้อย่อย</h3>
  </section>
</main>

// ❌ แย่: ข้าม heading level
<main>
  <h1>หัวข้อหลัก</h1>
  <h4>ข้าม h2, h3 ไปเลย</h4>
  <h2>ย้อนกลับมา h2</h2>
</main>
```

---

## Step 1456-1465: SEO Monitoring และ Tools

### Google Search Console Integration

```typescript
// app/layout.tsx
export const metadata: Metadata = {
  // Google Site Verification
  verification: {
    google: 'YOUR_GOOGLE_VERIFICATION_CODE',
    yandex: 'YOUR_YANDEX_CODE',
    other: {
      me: ['https://mastodon.social/@yourusername'],
    },
  },
}
```

### การ Monitor SEO Performance

```typescript
// lib/seo-analytics.ts
export async function trackSEOMetrics() {
  // ดึงข้อมูลจาก Google Search Console API
  const { google } = await import('googleapis')
  
  const auth = new google.auth.GoogleAuth({
    credentials: JSON.parse(process.env.GOOGLE_CREDENTIALS!),
    scopes: ['https://www.googleapis.com/auth/webmasters.readonly'],
  })
  
  const searchConsole = google.searchconsole({ version: 'v1', auth })
  
  const response = await searchConsole.searchanalytics.query({
    siteUrl: 'https://example.com',
    requestBody: {
      startDate: '2024-01-01',
      endDate: '2024-01-31',
      dimensions: ['query', 'page'],
      rowLimit: 100,
    },
  })
  
  return response.data
}
```

---

## Step 1466-1475: Checklist SEO สำหรับ Next.js

### Pre-Launch SEO Checklist

```markdown
## Technical SEO
- [ ] Title tags มี keyword หลัก (50-60 ตัวอักษร)
- [ ] Meta descriptions (150-160 ตัวอักษร)
- [ ] H1 tags มี 1 ต่อหน้า
- [ ] Image alt text ครบทุกรูป
- [ ] Sitemap.xml สร้างและ submit แล้ว
- [ ] Robots.txt ถูกต้อง
- [ ] Canonical URLs กำหนดแล้ว
- [ ] Structured Data (JSON-LD) ครบ
- [ ] Open Graph tags ครบ
- [ ] Twitter Card tags ครบ
- [ ] SSL Certificate ติดตั้งแล้ว
- [ ] HTTPS redirect ทำงาน
- [ ] Mobile-friendly
- [ ] Page speed > 90 (Lighthouse)
- [ ] Core Web Vitals ผ่าน
- [ ] No broken links
- [ ] 404 page กำหนดแล้ว
- [ ] XML sitemap ไม่มี broken links
- [ ] Robots.txt ไม่ block important pages
- [ ] Google Search Console setup
- [ ] Google Analytics setup

## Content SEO
- [ ] Keyword research ทำแล้ว
- [ ] Content คุณภาพสูง (500+ คำ)
- [ ] Internal linking
- [ ] External links มีคุณภาพ
- [ ] URL structure descriptive
- [ ] Page hierarchy ชัดเจน
```

---

## 🧪 Quiz - Part 46

**ข้อ 1:** `generateMetadata` ใน Next.js 14 แตกต่างจาก static `metadata` export อย่างไร?
- A) `generateMetadata` ทำงานเร็วกว่า
- B) `generateMetadata` สามารถ async และดึงข้อมูลจาก API ได้
- C) `generateMetadata` ไม่รองรับ TypeScript
- D) ไม่มีความแตกต่าง

**ข้อ 2:** Open Graph `og:type` ที่ใช้สำหรับ Blog Post คือ?
- A) `website`
- B) `blog`
- C) `article`
- D) `post`

**ข้อ 3:** Core Web Vitals ข้อใดวัดความเสถียรของ Layout?
- A) LCP
- B) FID
- C) INP
- D) CLS

**ข้อ 4:** ไฟล์ `robots.ts` ใน Next.js 14 Export อะไร?
- A) Object ธรรมดา
- B) String
- C) Function ที่ return `MetadataRoute.Robots`
- D) Array

**เฉลย:** 1-B, 2-C, 3-D, 4-C

---

> **➡️ Next:** [Part 47: Internationalization (i18n)](./part-47-internationalization.md)
