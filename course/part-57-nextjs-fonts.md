# Part 57: Next.js Fonts

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1876-1910  
> **เวลาเรียน:** ~2 ชั่วโมง

---

## 📚 Table of Contents

1. [next/font คืออะไร](#nextfont-คืออะไร)
2. [Google Fonts](#google-fonts)
3. [Local Fonts](#local-fonts)
4. [Font Variables](#font-variables)
5. [Variable Fonts](#variable-fonts)
6. [Font Optimization](#font-optimization)
7. [Loading Strategy](#loading-strategy)
8. [Quiz](#quiz)

---

## Step 1876: next/font คืออะไร {#nextfont-คืออะไร}

`next/font` คือ built-in font optimization ใน Next.js ที่:
- Zero layout shift (ไม่มี CLS)
- Self-hosted fonts อัตโนมัติ
- Privacy-preserving (ไม่มี request ออกนอก)
- Preloading fonts อัตโนมัติ

```
ปัญหาก่อน next/font:
1. Font ต้อง request ออกไปยัง Google Fonts server
2. Layout shift เมื่อ font โหลดมาแทนที่ system font
3. Privacy concerns (Google tracking)
4. Performance issues

next/font แก้ปัญหา:
1. Download fonts ตอน build time
2. Self-host บน server ของเราเอง
3. ไม่มี external requests
4. Automatic font-display: swap
5. Preload fonts ที่สำคัญ
```

---

## Step 1877-1883: Google Fonts {#google-fonts}

### ใช้ Google Font พื้นฐาน

```typescript
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],  // โหลด subset ที่ต้องการ
  display: 'swap',     // font-display: swap
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

### หลาย Font Weights

```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  weight: ['300', '400', '500', '600', '700'],  // weights ที่ต้องการ
  style: ['normal', 'italic'],                   // styles
  display: 'swap',
  preload: true,
})

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',  // CSS variable
})

export default function RootLayout({ children }) {
  return (
    <html lang="th" className={`${inter.className} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

### Font สำหรับภาษาไทย

```typescript
// app/layout.tsx
import { Sarabun, Noto_Sans_Thai } from 'next/font/google'

// Sarabun - font ยอดนิยมสำหรับภาษาไทย
const sarabun = Sarabun({
  subsets: ['thai', 'latin'],  // ต้องใส่ 'thai' subset
  weight: ['300', '400', '500', '600', '700'],
  display: 'swap',
  variable: '--font-sarabun',
})

// Noto Sans Thai - รองรับทุก Unicode characters
const notoSansThai = Noto_Sans_Thai({
  subsets: ['thai'],
  weight: ['300', '400', '500', '700'],
  display: 'swap',
  variable: '--font-noto-thai',
})

export default function RootLayout({ children }) {
  return (
    <html lang="th" className={sarabun.className}>
      <body>{children}</body>
    </html>
  )
}
```

### Font Pair (Heading + Body)

```typescript
// fonts.ts
import { Prompt, Sarabun } from 'next/font/google'

export const prompt = Prompt({
  subsets: ['thai', 'latin'],
  weight: ['400', '500', '600', '700', '800'],
  variable: '--font-prompt',
  display: 'swap',
})

export const sarabun = Sarabun({
  subsets: ['thai', 'latin'],
  weight: ['300', '400', '500'],
  variable: '--font-sarabun',
  display: 'swap',
})

// app/layout.tsx
import { prompt, sarabun } from '@/fonts'

export default function RootLayout({ children }) {
  return (
    <html
      lang="th"
      className={`${prompt.variable} ${sarabun.variable}`}
    >
      <body className="font-sarabun">
        {/* Heading ใช้ Prompt */}
        {/* Body ใช้ Sarabun */}
        {children}
      </body>
    </html>
  )
}
```

```css
/* tailwind.config.ts หรือ globals.css */
@theme {
  --font-heading: var(--font-prompt);
  --font-body: var(--font-sarabun);
}

/* หรือใน tailwind.config.ts */
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        heading: ['var(--font-prompt)', 'sans-serif'],
        body: ['var(--font-sarabun)', 'sans-serif'],
      },
    },
  },
}
```

---

## Step 1884-1890: Local Fonts {#local-fonts}

```typescript
// fonts/index.ts
import localFont from 'next/font/local'

// Single font file
export const myFont = localFont({
  src: '../public/fonts/MyFont-Regular.woff2',
  variable: '--font-my-font',
  display: 'swap',
})

// Multiple weights/styles
export const anuphan = localFont({
  src: [
    {
      path: '../public/fonts/Anuphan-Light.woff2',
      weight: '300',
      style: 'normal',
    },
    {
      path: '../public/fonts/Anuphan-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: '../public/fonts/Anuphan-Medium.woff2',
      weight: '500',
      style: 'normal',
    },
    {
      path: '../public/fonts/Anuphan-SemiBold.woff2',
      weight: '600',
      style: 'normal',
    },
    {
      path: '../public/fonts/Anuphan-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
  ],
  variable: '--font-anuphan',
  display: 'swap',
})

// app/layout.tsx
import { anuphan } from '@/fonts'

export default function RootLayout({ children }) {
  return (
    <html lang="th" className={anuphan.variable}>
      <body className="font-anuphan">{children}</body>
    </html>
  )
}
```

### Font File Formats

```
Font Formats:
├── .woff2 (แนะนำ - เล็กที่สุด, รองรับ browser ใหม่)
├── .woff  (fallback สำหรับ browser เก่า)
├── .ttf   (True Type - ใหญ่กว่า)
└── .otf   (Open Type)

Browser Support (2024):
├── woff2: 97%+ (รองรับทุก modern browser)
└── woff:  95%+ (fallback สำหรับ IE)
```

---

## Step 1891-1895: Font Variables {#font-variables}

```typescript
// app/layout.tsx - CSS Variables approach
import { Inter, Sarabun } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

const sarabun = Sarabun({
  subsets: ['thai', 'latin'],
  weight: ['400', '500', '700'],
  variable: '--font-sarabun',
  display: 'swap',
})

export default function RootLayout({ children }) {
  return (
    <html className={`${inter.variable} ${sarabun.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

```css
/* globals.css */
:root {
  --font-sans: var(--font-inter), system-ui, sans-serif;
  --font-thai: var(--font-sarabun), 'Leelawadee UI', sans-serif;
}

body {
  font-family: var(--font-thai);
  /* Thai browsers จะใช้ sarabun */
  /* English text ใช้ inter */
}

h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-sans);
}

code, pre {
  font-family: var(--font-mono, monospace);
}
```

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
        thai: ['var(--font-sarabun)', 'Leelawadee UI', 'sans-serif'],
        mono: ['var(--font-jetbrains-mono)', 'monospace'],
      },
    },
  },
}

// การใช้งาน
<h1 className="font-sans font-bold">English Heading</h1>
<p className="font-thai">ข้อความภาษาไทย</p>
<code className="font-mono">const x = 1</code>
```

---

## Step 1896-1900: Variable Fonts {#variable-fonts}

Variable Fonts คือ font ไฟล์เดียวที่รวม weights/styles ทั้งหมด

```typescript
// Variable font จาก Google Fonts
import { Inter } from 'next/font/google'

// ไม่ต้องระบุ weights เพราะ Inter รองรับ variable font
const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
  // ไม่ต้องใส่ weight เพราะ variable font
})

// Local variable font
const myVariableFont = localFont({
  src: [
    {
      path: './fonts/MyFont-Variable.woff2',
      // ไม่ต้องระบุ weight เพราะ variable font
    },
  ],
  variable: '--font-my-variable',
})
```

```css
/* การใช้งาน Variable Font */
:root {
  --font-weight-thin: 100;
  --font-weight-normal: 400;
  --font-weight-bold: 700;
}

/* Animate font weight */
@keyframes weight-pulse {
  0%, 100% { font-variation-settings: 'wght' 400; }
  50% { font-variation-settings: 'wght' 700; }
}

.animated-title {
  animation: weight-pulse 2s ease-in-out infinite;
}

/* Responsive font weight */
p {
  font-variation-settings: 'wght' 400;
}

@media (prefers-color-scheme: dark) {
  p {
    /* เพิ่ม weight เล็กน้อยใน dark mode เพื่อ readability */
    font-variation-settings: 'wght' 425;
  }
}
```

---

## Step 1901-1905: Font Optimization {#font-optimization}

### Font Loading Best Practices

```typescript
// 1. Preload สำหรับ Critical fonts
const criticalFont = Inter({
  subsets: ['latin'],
  preload: true,  // preload ใน head (default: true)
  display: 'swap',
})

// 2. ไม่ preload สำหรับ Optional fonts
const optionalFont = Noto_Sans_JP({
  subsets: ['japanese'],
  preload: false,  // โหลดแบบ lazy
  display: 'optional',
})

// 3. Subset ที่เหมาะสม
// ✅ โหลดเฉพาะ subset ที่ต้องการ
const font = Sarabun({
  subsets: ['thai', 'latin'],  // ไม่ต้องโหลด greek, vietnamese etc.
})

// ❌ โหลดทุก subset
const font = Sarabun({
  subsets: ['thai', 'latin', 'latin-ext', 'vietnamese', 'cyrillic'],
  // ใหญ่เกินไป!
})
```

### Font Performance Metrics

```
Font Size Comparison (Sarabun-Regular):
├── woff2: ~50KB (แนะนำ)
├── woff:  ~65KB
└── ttf:   ~120KB

Thai Font Subsets:
├── thai subset: ~30KB
├── latin subset: ~20KB
└── total: ~50KB (เทียบกับ full font 150KB+)

Loading Time:
├── Self-hosted (next/font): ~50ms
└── Google Fonts CDN: ~100-300ms + DNS lookup
```

---

## Step 1906-1910: Loading Strategy {#loading-strategy}

```typescript
// font-display values และผลกระทบ

// swap (แนะนำสำหรับ Body text)
// - แสดง fallback font ทันที
// - เปลี่ยนเป็น web font เมื่อโหลดเสร็จ
// - อาจเกิด layout shift
const bodyFont = Inter({
  subsets: ['latin'],
  display: 'swap',
})

// optional (แนะนำสำหรับ non-critical)
// - Browser ตัดสินใจเอง
// - ถ้าโหลดเร็วพอก็ใช้, ถ้าไม่ก็ใช้ fallback
// - ไม่มี layout shift
const decorativeFont = localFont({
  src: './fonts/Decorative.woff2',
  display: 'optional',
})

// block (ไม่แนะนำ)
// - ซ่อน text จนกว่า font จะโหลด
// - ป้องกัน FOUT แต่ทำให้ text ไม่แสดง
const iconFont = localFont({
  src: './fonts/Icons.woff2',
  display: 'block',  // สมเหตุสมผลสำหรับ icon fonts
})

// fallback (สมดุล)
// - แสดง fallback font ทันที
// - ถ้า font โหลดเร็ว (<100ms) ก็ใช้ web font
// - ถ้าช้ากว่านั้นใช้ fallback ตลอด session
const headingFont = Prompt({
  subsets: ['thai', 'latin'],
  display: 'fallback',
})
```

### Font Fallback Stack

```typescript
// กำหนด fallback fonts ที่ดี
const sarabun = Sarabun({
  subsets: ['thai', 'latin'],
  variable: '--font-sarabun',
  fallback: [
    'Leelawadee UI',  // Windows Thai font
    'Leelawadee',     // Windows Thai font (older)
    'Tahoma',         // ครอบคลุม Thai เพราะมี Thai glyphs
    'sans-serif',     // System default
  ],
})
```

```css
/* System Font Stack ที่ดีสำหรับภาษาไทย */
body {
  font-family:
    var(--font-sarabun),
    'Leelawadee UI',
    'Leelawadee',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
}
```

---

## 🧪 Quiz - Part 57

**ข้อ 1:** ข้อดีหลักของ `next/font` เทียบกับ Google Fonts CDN คืออะไร?
- A) ฟรี
- B) Self-hosted ไม่มี external requests ป้องกัน CLS และ Privacy
- C) รองรับ fonts มากกว่า
- D) Font ขนาดเล็กกว่า

**ข้อ 2:** ภาษาไทยต้องใช้ subset ใดใน next/font?
- A) `'latin'`
- B) `'unicode'`
- C) `'thai'`
- D) `'asia'`

**ข้อ 3:** `font-display: swap` มีผลอย่างไร?
- A) ซ่อน text จนกว่า font จะโหลด
- B) แสดง fallback font ทันที และเปลี่ยนเป็น web font เมื่อโหลดเสร็จ
- C) ไม่โหลด web font
- D) โหลด web font ทันที

**ข้อ 4:** Variable Font คืออะไร?
- A) Font ที่มีขนาด file ใหญ่
- B) Font ไฟล์เดียวที่มีหลาย weights/styles ในตัว
- C) Font ที่โหลดได้หลาย URLs
- D) Font ที่ใช้ JavaScript

**เฉลย:** 1-B, 2-C, 3-B, 4-B

---

> **➡️ Next:** [Part 58: Error Boundaries & Suspense ขั้นสูง](./part-58-error-boundary-suspense.md)
