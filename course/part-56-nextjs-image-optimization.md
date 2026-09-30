# Part 56: Next.js Image Optimization

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1841-1875  
> **เวลาเรียน:** ~3 ชั่วโมง

---

## 📚 Table of Contents

1. [Next.js Image Component](#image-component)
2. [Automatic Optimization](#automatic-optimization)
3. [Responsive Images](#responsive-images)
4. [Blur Placeholder](#blur-placeholder)
5. [Remote Images](#remote-images)
6. [AVIF/WebP Format](#avif-webp)
7. [Custom Image Loader](#custom-loader)
8. [CDN Integration](#cdn-integration)
9. [Quiz](#quiz)

---

## Step 1841: Next.js Image Component {#image-component}

`next/image` เป็น Image Component ที่ optimize รูปภาพโดยอัตโนมัติ

```typescript
import Image from 'next/image'

// ตัวอย่างพื้นฐาน
export default function Example() {
  return (
    <Image
      src="/images/hero.jpg"       // path ของรูป
      alt="Hero Image"             // alt text (required)
      width={1200}                 // original width (px)
      height={600}                 // original height (px)
    />
  )
}
```

### Props ทั้งหมด

```typescript
import Image from 'next/image'

export function FullExample() {
  return (
    <Image
      // Required
      src="/images/photo.jpg"
      alt="Descriptive alt text"
      
      // Size (required สำหรับ static/remote images)
      width={800}
      height={600}
      
      // Layout
      fill={false}              // fill parent container
      sizes="(max-width: 768px) 100vw, 50vw"
      
      // Loading
      priority={false}          // true = preload (สำหรับ LCP image)
      loading="lazy"            // "lazy" | "eager"
      
      // Quality (1-100, default: 75)
      quality={80}
      
      // Placeholder
      placeholder="blur"        // "blur" | "empty"
      blurDataURL="data:..."    // สำหรับ remote images
      
      // Styling
      className="rounded-lg"
      style={{ objectFit: 'cover' }}
      
      // Event handlers
      onLoad={(e) => console.log('Loaded!')}
      onError={(e) => console.log('Error!')}
      
      // Optional
      unoptimized={false}       // true = ไม่ optimize (pass-through)
    />
  )
}
```

---

## Step 1845-1850: Automatic Optimization {#automatic-optimization}

```
Next.js Image Optimization:
├── Resize รูปภาพตามขนาดที่ต้องการ
├── Convert เป็น WebP/AVIF อัตโนมัติ
├── Lazy loading โดยอัตโนมัติ
├── Prevent Cumulative Layout Shift (CLS)
└── Serve จาก /_next/image endpoint
```

### next.config.ts สำหรับ Image

```typescript
// next.config.ts
import type { NextConfig } from 'next'

const config: NextConfig = {
  images: {
    // Formats ที่รองรับ (เรียงตาม preference)
    formats: ['image/avif', 'image/webp'],
    
    // ขนาดที่ generate
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    
    // Quality default
    quality: 75,
    
    // MinimumcacheTTL (วินาที)
    minimumCacheTTL: 60,
    
    // Domains ที่อนุญาต (Legacy - ใช้ remotePatterns แทน)
    // domains: ['example.com'],
    
    // Remote Patterns (แนะนำ)
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
      },
      {
        protocol: 'https',
        hostname: '**.cloudinary.com',
      },
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        port: '',
        pathname: '/images/**',
      },
    ],
    
    // ปิด optimization สำหรับ environment นี้
    // unoptimized: process.env.NODE_ENV !== 'production',
  },
}

export default config
```

---

## Step 1851-1855: Responsive Images {#responsive-images}

```typescript
// 1. Fixed Width Images
<Image
  src="/photo.jpg"
  alt="Photo"
  width={400}
  height={300}
/>

// 2. Responsive with sizes prop
<Image
  src="/photo.jpg"
  alt="Photo"
  width={1200}
  height={800}
  // บอก browser ว่าขนาดจะเป็นเท่าไรตาม viewport
  sizes="(max-width: 640px) 100vw,
         (max-width: 1024px) 50vw,
         33vw"
/>

// 3. Fill Parent Container
<div className="relative w-full h-64">
  <Image
    src="/photo.jpg"
    alt="Photo"
    fill
    style={{ objectFit: 'cover' }}
    sizes="(max-width: 768px) 100vw, 50vw"
  />
</div>

// 4. Hero Image (Full Width)
<div className="relative w-full" style={{ paddingBottom: '56.25%' }}>
  <Image
    src="/hero.jpg"
    alt="Hero"
    fill
    priority  // Preload เพราะเป็น LCP element
    style={{ objectFit: 'cover' }}
    sizes="100vw"
  />
</div>
```

### Gallery Component

```typescript
// components/Gallery.tsx
import Image from 'next/image'

interface GalleryImage {
  src: string
  alt: string
  width: number
  height: number
}

export function Gallery({ images }: { images: GalleryImage[] }) {
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
      {images.map((image, index) => (
        <div key={image.src} className="relative aspect-square overflow-hidden rounded-lg">
          <Image
            src={image.src}
            alt={image.alt}
            fill
            // Priority สำหรับรูปแรกๆ
            priority={index < 4}
            className="object-cover hover:scale-105 transition-transform duration-300"
            sizes="(max-width: 640px) 50vw,
                   (max-width: 1024px) 33vw,
                   25vw"
            quality={80}
          />
        </div>
      ))}
    </div>
  )
}
```

---

## Step 1856-1860: Blur Placeholder {#blur-placeholder}

```typescript
// 1. Static Images - Blur อัตโนมัติ
import heroImage from '@/public/images/hero.jpg'

export function Hero() {
  return (
    <Image
      src={heroImage}
      alt="Hero"
      placeholder="blur"  // ใช้ blurDataURL จาก import อัตโนมัติ
      priority
    />
  )
}

// 2. Remote Images - ต้องสร้าง blurDataURL เอง
export async function RemoteImage({ src }: { src: string }) {
  // วิธีที่ 1: ใช้ plaiceholder library
  const { base64 } = await getPlaiceholder(src)
  
  return (
    <Image
      src={src}
      alt="Remote Image"
      width={800}
      height={600}
      placeholder="blur"
      blurDataURL={base64}
    />
  )
}

// วิธีที่ 2: Inline SVG blur
const shimmer = (w: number, h: number) => `
  <svg width="${w}" height="${h}" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <defs>
      <linearGradient id="g">
        <stop stop-color="#333" offset="20%" />
        <stop stop-color="#222" offset="50%" />
        <stop stop-color="#333" offset="70%" />
      </linearGradient>
    </defs>
    <rect width="${w}" height="${h}" fill="#333" />
    <rect id="r" width="${w}" height="${h}" fill="url(#g)" />
    <animate xlink:href="#r" attributeName="x" from="-${w}" to="${w}" dur="1s" repeatCount="indefinite" />
  </svg>
`

const toBase64 = (str: string) =>
  typeof window === 'undefined'
    ? Buffer.from(str).toString('base64')
    : window.btoa(str)

export function BlurImage({ src, alt, width, height }: {
  src: string
  alt: string
  width: number
  height: number
}) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      placeholder="blur"
      blurDataURL={`data:image/svg+xml;base64,${toBase64(shimmer(width, height))}`}
    />
  )
}
```

---

## Step 1861-1865: Remote Images {#remote-images}

```typescript
// Remote images ต้องกำหนด remotePatterns ใน next.config.ts

// ตัวอย่าง: Unsplash
export function UnsplashImage({ imageId }: { imageId: string }) {
  return (
    <Image
      src={`https://images.unsplash.com/photo-${imageId}?w=800&h=600&fit=crop`}
      alt="Unsplash photo"
      width={800}
      height={600}
    />
  )
}

// ตัวอย่าง: Cloudinary
export function CloudinaryImage({
  publicId,
  width,
  height,
}: {
  publicId: string
  width: number
  height: number
}) {
  return (
    <Image
      src={`https://res.cloudinary.com/${process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME}/image/upload/w_${width},h_${height},c_fill,q_auto,f_auto/${publicId}`}
      alt={publicId}
      width={width}
      height={height}
      placeholder="blur"
      blurDataURL={`https://res.cloudinary.com/${process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME}/image/upload/w_16,h_16,q_1/${publicId}`}
    />
  )
}

// Dynamic URL
export function ProductImage({ product }: { product: any }) {
  // Handle missing image
  const src = product.image || '/images/placeholder-product.jpg'
  
  return (
    <div className="relative aspect-square">
      <Image
        src={src}
        alt={product.name}
        fill
        sizes="(max-width: 768px) 50vw, 25vw"
        className="object-cover"
        onError={(e) => {
          // Fallback ถ้ารูปโหลดไม่ได้
          ;(e.target as HTMLImageElement).src = '/images/placeholder-product.jpg'
        }}
      />
    </div>
  )
}
```

---

## Step 1866-1869: Custom Image Loader {#custom-loader}

```typescript
// lib/imageLoader.ts - Custom Cloudinary Loader
import type { ImageLoader } from 'next/image'

export const cloudinaryLoader: ImageLoader = ({ src, width, quality }) => {
  const params = [
    `w_${width}`,
    `q_${quality || 75}`,
    'c_fill',
    'f_auto',
  ].join(',')
  
  return `https://res.cloudinary.com/${process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME}/image/upload/${params}/${src}`
}

// การใช้งาน
import { cloudinaryLoader } from '@/lib/imageLoader'

<Image
  loader={cloudinaryLoader}
  src="my-public-id"  // Cloudinary public ID
  alt="Image"
  width={800}
  height={600}
/>

// Global Custom Loader ใน next.config.ts
const config = {
  images: {
    loader: 'custom',
    loaderFile: './lib/imageLoader.ts',
  },
}

// imageLoader.ts (สำหรับ global loader)
export default function myImageLoader({ src, width, quality }: {
  src: string
  width: number
  quality?: number
}): string {
  return `https://my-cdn.com/images/${src}?w=${width}&q=${quality || 75}`
}
```

---

## Step 1870-1875: CDN Integration {#cdn-integration}

```typescript
// next.config.ts - Asset Prefix สำหรับ CDN
const config: NextConfig = {
  // เปลี่ยน domain สำหรับ static assets
  assetPrefix: process.env.NODE_ENV === 'production'
    ? 'https://cdn.example.com'
    : '',
  
  images: {
    // Image endpoint (สำหรับ custom CDN)
    path: '/_next/image',  // default
    
    // สำหรับ export static site
    // unoptimized: true,
    
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
      },
    ],
  },
}

// Cloudfront + S3 Setup
const config: NextConfig = {
  assetPrefix: `https://${process.env.CLOUDFRONT_DOMAIN}`,
  
  images: {
    loader: 'custom',
    loaderFile: './lib/imageLoader.ts',
  },
}

// lib/imageLoader.ts สำหรับ CloudFront
export default function cloudFrontLoader({ src, width, quality }) {
  const params = new URLSearchParams({
    w: width.toString(),
    q: (quality || 75).toString(),
  })
  
  return `https://${process.env.NEXT_PUBLIC_CLOUDFRONT_DOMAIN}/images/${src}?${params}`
}
```

### Image Best Practices

```typescript
// ✅ Best Practices

// 1. Priority สำหรับ LCP image
<Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />

// 2. Descriptive alt text
<Image src="/product.jpg" alt="iPhone 15 Pro Max สีดำ 256GB" ... />

// 3. Correct sizes prop
<Image
  src="/photo.jpg"
  alt="Photo"
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>

// 4. Quality ที่เหมาะสม
// Photos: 75-85
// Graphics/logos: 90-100
<Image quality={80} ... />

// 5. Aspect ratio container สำหรับ fill
<div className="relative aspect-video">
  <Image src="/video-thumbnail.jpg" alt="Video" fill className="object-cover" />
</div>
```

---

## 🧪 Quiz - Part 56

**ข้อ 1:** ทำไมต้องกำหนด `width` และ `height` ใน Image component?
- A) เพื่อ resize รูปภาพ
- B) เพื่อ prevent CLS (Cumulative Layout Shift)
- C) เพื่อ limit file size
- D) เพื่อ enable WebP

**ข้อ 2:** `priority` prop ควรใช้กับ Image ใด?
- A) ทุก images
- B) รูปขนาดใหญ่
- C) LCP image (รูปที่ใหญ่ที่สุดในหน้า)
- D) Images ที่อยู่ใน footer

**ข้อ 3:** `fill` prop ใน Image ต้องใช้กับอะไร?
- A) Parent ที่มี `position: absolute`
- B) Parent ที่มี `position: relative`
- C) Parent ที่มี `display: flex`
- D) Parent ที่มี `overflow: hidden`

**ข้อ 4:** `placeholder="blur"` สำหรับ remote images ต้องการอะไรเพิ่ม?
- A) ไม่ต้องการอะไรเพิ่ม
- B) กำหนด `blurDataURL`
- C) กำหนด `quality`
- D) กำหนด `loader`

**เฉลย:** 1-B, 2-C, 3-B, 4-B

---

> **➡️ Next:** [Part 57: Next.js Fonts](./part-57-nextjs-fonts.md)
