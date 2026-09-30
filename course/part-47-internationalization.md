# Part 47: Internationalization (i18n) ใน Next.js

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1476-1510  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [i18n คืออะไร](#i18n-คืออะไร)
2. [next-intl Library](#next-intl-library)
3. [Locale Routing](#locale-routing)
4. [Translation Files](#translation-files)
5. [Pluralization](#pluralization)
6. [Date/Number Formatting](#datenumber-formatting)
7. [RTL Support](#rtl-support)
8. [ตัวอย่าง Thai/English App](#thai-english-app)
9. [Quiz](#quiz)

---

## Step 1476: i18n คืออะไร {#i18n-คืออะไร}

**Internationalization (i18n)** คือกระบวนการออกแบบซอฟต์แวร์ให้รองรับหลายภาษาและวัฒนธรรม

```
i18n = Internationalization
     ↑                    ↑
     i                    n
     มี 18 ตัวอักษรระหว่าง i และ n
     
L10n = Localization (การ translate และ customize สำหรับแต่ละ locale)
```

### ความแตกต่างระหว่าง i18n และ L10n

```
i18n (Internationalization):
├── โครงสร้างพื้นฐานที่รองรับหลายภาษา
├── Format Date, Number, Currency
├── Text Direction (LTR/RTL)
└── Character Encoding

L10n (Localization):
├── การแปลข้อความ
├── การปรับ Content ให้เหมาะกับวัฒนธรรม
├── รูปแบบวันที่ท้องถิ่น
└── สกุลเงินท้องถิ่น
```

### Locale Format

```
BCP 47 Language Tag:
├── th         → ภาษาไทย
├── en         → ภาษาอังกฤษ
├── en-US      → English (United States)
├── en-GB      → English (United Kingdom)
├── zh-Hans    → Chinese (Simplified)
├── zh-Hant    → Chinese (Traditional)
└── ar         → Arabic
```

---

## Step 1477: next-intl Library {#next-intl-library}

`next-intl` เป็น Library i18n ที่ดีที่สุดสำหรับ Next.js App Router

### Installation

```bash
npm install next-intl
# หรือ
pnpm add next-intl
# หรือ
yarn add next-intl
```

### Project Structure

```
app/
├── [locale]/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── blog/
│   │   └── page.tsx
│   └── about/
│       └── page.tsx
├── i18n.ts
└── middleware.ts

messages/
├── en.json
├── th.json
└── ja.json

next.config.ts
```

### Configuration

```typescript
// next.config.ts
import createNextIntlPlugin from 'next-intl/plugin'

const withNextIntl = createNextIntlPlugin()

/** @type {import('next').NextConfig} */
const config = {
  // config อื่นๆ
}

export default withNextIntl(config)
```

```typescript
// i18n.ts
import { notFound } from 'next/navigation'
import { getRequestConfig } from 'next-intl/server'

const locales = ['en', 'th', 'ja'] as const
export type Locale = (typeof locales)[number]

export default getRequestConfig(async ({ locale }) => {
  // Validate locale
  if (!locales.includes(locale as any)) notFound()
  
  return {
    messages: (await import(`./messages/${locale}.json`)).default,
    timeZone: locale === 'th' ? 'Asia/Bangkok' : 'UTC',
    now: new Date(),
  }
})
```

### Middleware สำหรับ Locale Routing

```typescript
// middleware.ts
import createMiddleware from 'next-intl/middleware'

export default createMiddleware({
  // ภาษาที่รองรับ
  locales: ['en', 'th', 'ja'],
  
  // ภาษา default
  defaultLocale: 'th',
  
  // Strategy สำหรับ routing
  // 'always' - ใส่ locale ใน URL เสมอ: /th/about, /en/about
  // 'as-needed' - locale default ไม่ต้องใส่: /about (th), /en/about
  localePrefix: 'always',
  
  // Locale Detection
  localeDetection: true,
})

export const config = {
  // Match ทุก route ยกเว้น static files และ API
  matcher: ['/((?!api|_next|_vercel|.*\\..*).*)'],
}
```

---

## Step 1478: Locale Routing {#locale-routing}

### Folder Structure สำหรับ App Router

```typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl'
import { getMessages } from 'next-intl/server'

export default async function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode
  params: { locale: string }
}) {
  // โหลด messages สำหรับ Server Components
  const messages = await getMessages()
  
  return (
    <html lang={locale} dir={locale === 'ar' ? 'rtl' : 'ltr'}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
```

```typescript
// app/[locale]/page.tsx (Server Component)
import { useTranslations } from 'next-intl'
import { unstable_setRequestLocale } from 'next-intl/server'

export default function HomePage({
  params: { locale },
}: {
  params: { locale: string }
}) {
  unstable_setRequestLocale(locale)
  
  const t = useTranslations('HomePage')
  
  return (
    <main>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </main>
  )
}
```

### Language Switcher Component

```typescript
// components/LanguageSwitcher.tsx
'use client'

import { useLocale, useTranslations } from 'next-intl'
import { useRouter, usePathname } from 'next-intl/client'
import { Locale } from '@/i18n'

const languages = [
  { code: 'th', name: 'ภาษาไทย', flag: '🇹🇭' },
  { code: 'en', name: 'English', flag: '🇬🇧' },
  { code: 'ja', name: '日本語', flag: '🇯🇵' },
]

export function LanguageSwitcher() {
  const locale = useLocale()
  const router = useRouter()
  const pathname = usePathname()
  const t = useTranslations('LanguageSwitcher')
  
  function handleChange(newLocale: string) {
    router.replace(pathname, { locale: newLocale as Locale })
  }
  
  return (
    <div className="flex gap-2">
      {languages.map((lang) => (
        <button
          key={lang.code}
          onClick={() => handleChange(lang.code)}
          className={`
            px-3 py-1 rounded-full text-sm font-medium transition-colors
            ${locale === lang.code 
              ? 'bg-blue-600 text-white' 
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
            }
          `}
          aria-label={`Switch to ${lang.name}`}
          aria-current={locale === lang.code ? 'true' : 'false'}
        >
          {lang.flag} {lang.name}
        </button>
      ))}
    </div>
  )
}
```

### Link Component สำหรับ i18n

```typescript
// ใช้ Link จาก next-intl แทน next/link
import { Link } from 'next-intl'

export function Navigation() {
  return (
    <nav>
      {/* Link จะใส่ locale prefix อัตโนมัติ */}
      <Link href="/">หน้าหลัก</Link>
      <Link href="/about">เกี่ยวกับเรา</Link>
      <Link href="/blog">บล็อก</Link>
      
      {/* Link ไปยัง locale อื่น */}
      <Link href="/about" locale="en">
        English Version
      </Link>
    </nav>
  )
}
```

### generateStaticParams สำหรับ i18n

```typescript
// app/[locale]/blog/[slug]/page.tsx
import { getTranslations, unstable_setRequestLocale } from 'next-intl/server'

export function generateStaticParams() {
  const locales = ['en', 'th', 'ja']
  const slugs = ['react-basics', 'nextjs-guide', 'typescript-tips']
  
  // สร้าง params สำหรับทุก combination
  return locales.flatMap((locale) =>
    slugs.map((slug) => ({ locale, slug }))
  )
}

export async function generateMetadata({
  params: { locale, slug },
}: {
  params: { locale: string; slug: string }
}) {
  const t = await getTranslations({ locale, namespace: 'BlogPost' })
  const post = await getPost(slug, locale)
  
  return {
    title: post.title,
    description: post.excerpt,
    alternates: {
      languages: {
        'th': `/th/blog/${slug}`,
        'en': `/en/blog/${slug}`,
        'ja': `/ja/blog/${slug}`,
      },
    },
  }
}
```

---

## Step 1479: Translation Files {#translation-files}

### โครงสร้าง JSON Translation

```json
// messages/th.json
{
  "HomePage": {
    "title": "ยินดีต้อนรับสู่เว็บไซต์ของเรา",
    "description": "เรียนรู้ React และ Next.js ได้ที่นี่",
    "hero": {
      "heading": "พัฒนาทักษะ React ของคุณ",
      "subheading": "คอร์สออนไลน์คุณภาพสูงสำหรับนักพัฒนาทุกระดับ",
      "cta": "เริ่มเรียนฟรี"
    },
    "features": {
      "title": "ทำไมต้องเรียนกับเรา",
      "items": {
        "expert": "สอนโดยผู้เชี่ยวชาญ",
        "practical": "Project จริง",
        "community": "Community ที่แข็งแกร่ง"
      }
    }
  },
  "Navigation": {
    "home": "หน้าหลัก",
    "about": "เกี่ยวกับ",
    "blog": "บล็อก",
    "courses": "คอร์ส",
    "contact": "ติดต่อ"
  },
  "Footer": {
    "copyright": "© {year} บริษัท ของฉัน จำกัด",
    "privacy": "นโยบายความเป็นส่วนตัว",
    "terms": "ข้อกำหนดการใช้งาน"
  },
  "Common": {
    "loading": "กำลังโหลด...",
    "error": "เกิดข้อผิดพลาด",
    "retry": "ลองใหม่อีกครั้ง",
    "submit": "ส่ง",
    "cancel": "ยกเลิก",
    "save": "บันทึก",
    "delete": "ลบ",
    "edit": "แก้ไข",
    "search": "ค้นหา",
    "close": "ปิด"
  },
  "Auth": {
    "login": "เข้าสู่ระบบ",
    "logout": "ออกจากระบบ",
    "register": "สมัครสมาชิก",
    "email": "อีเมล",
    "password": "รหัสผ่าน",
    "forgotPassword": "ลืมรหัสผ่าน?",
    "loginError": "อีเมลหรือรหัสผ่านไม่ถูกต้อง",
    "registerSuccess": "สมัครสมาชิกสำเร็จ กรุณาตรวจสอบอีเมลของคุณ"
  }
}
```

```json
// messages/en.json
{
  "HomePage": {
    "title": "Welcome to Our Website",
    "description": "Learn React and Next.js here",
    "hero": {
      "heading": "Develop Your React Skills",
      "subheading": "High-quality online courses for developers of all levels",
      "cta": "Start Learning Free"
    },
    "features": {
      "title": "Why Learn With Us",
      "items": {
        "expert": "Taught by Experts",
        "practical": "Real Projects",
        "community": "Strong Community"
      }
    }
  },
  "Navigation": {
    "home": "Home",
    "about": "About",
    "blog": "Blog",
    "courses": "Courses",
    "contact": "Contact"
  },
  "Footer": {
    "copyright": "© {year} My Company Ltd",
    "privacy": "Privacy Policy",
    "terms": "Terms of Service"
  },
  "Common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "retry": "Try Again",
    "submit": "Submit",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "search": "Search",
    "close": "Close"
  },
  "Auth": {
    "login": "Sign In",
    "logout": "Sign Out",
    "register": "Register",
    "email": "Email",
    "password": "Password",
    "forgotPassword": "Forgot Password?",
    "loginError": "Invalid email or password",
    "registerSuccess": "Registration successful. Please check your email."
  }
}
```

### การใช้ Translation

```typescript
// Server Component
import { useTranslations } from 'next-intl'

export default function HomePage() {
  const t = useTranslations('HomePage')
  const tNav = useTranslations('Navigation')
  const tCommon = useTranslations('Common')
  
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
      
      {/* Nested keys */}
      <h2>{t('hero.heading')}</h2>
      <p>{t('hero.subheading')}</p>
      
      {/* Variables */}
      <p>{t('copyright', { year: new Date().getFullYear() })}</p>
      
      <button>{tCommon('submit')}</button>
    </div>
  )
}
```

```typescript
// Client Component
'use client'

import { useTranslations } from 'next-intl'
import { useState } from 'react'

export function ContactForm() {
  const t = useTranslations('ContactForm')
  const tCommon = useTranslations('Common')
  const [submitted, setSubmitted] = useState(false)
  
  return (
    <form onSubmit={() => setSubmitted(true)}>
      <label>{t('nameLabel')}</label>
      <input placeholder={t('namePlaceholder')} />
      
      <label>{t('emailLabel')}</label>
      <input type="email" placeholder={t('emailPlaceholder')} />
      
      <label>{t('messageLabel')}</label>
      <textarea placeholder={t('messagePlaceholder')} />
      
      <button type="submit">
        {submitted ? tCommon('loading') : tCommon('submit')}
      </button>
      
      {submitted && <p>{t('successMessage')}</p>}
    </form>
  )
}
```

### Rich Text (HTML) ใน Translations

```json
// messages/th.json
{
  "Terms": {
    "agreement": "ฉันยอมรับ <link>ข้อกำหนดการใช้งาน</link> และ <bold>นโยบายความเป็นส่วนตัว</bold>"
  }
}
```

```typescript
import { useTranslations } from 'next-intl'
import Link from 'next/link'

export function TermsCheckbox() {
  const t = useTranslations('Terms')
  
  return (
    <label>
      <input type="checkbox" />
      {t.rich('agreement', {
        link: (chunks) => (
          <Link href="/terms" className="text-blue-600 underline">
            {chunks}
          </Link>
        ),
        bold: (chunks) => (
          <strong className="font-semibold">{chunks}</strong>
        ),
      })}
    </label>
  )
}
```

---

## Step 1480: Pluralization {#pluralization}

Pluralization คือการจัดการรูปแบบภาษาตามจำนวน

### CLDR Plural Rules

```
most locales มี 2 forms:
├── one: 1 item
└── other: 0, 2, 3... items

ภาษาที่ซับซ้อนกว่า:
├── zero, one, two, few, many, other (Arabic)
├── one, few, many, other (Russian, Polish)
└── one, other, zero (Welsh, Latvian)
```

### Plural ใน next-intl

```json
// messages/th.json
{
  "Items": {
    "count": "{count, plural, =0 {ไม่มีรายการ} one {# รายการ} other {# รายการ}}",
    "products": "คุณมี {count, plural, =0 {ไม่มีสินค้า} one {# ชิ้น} other {# ชิ้น}} ในตะกร้า",
    "notifications": "{count, plural, =0 {ไม่มีการแจ้งเตือน} one {การแจ้งเตือน # รายการ} other {การแจ้งเตือน # รายการ}}"
  }
}
```

```json
// messages/en.json
{
  "Items": {
    "count": "{count, plural, =0 {No items} one {# item} other {# items}}",
    "products": "You have {count, plural, =0 {no products} one {# product} other {# products}} in your cart",
    "notifications": "{count, plural, =0 {No notifications} one {# notification} other {# notifications}}"
  }
}
```

```typescript
// การใช้งาน
const t = useTranslations('Items')

// แสดง: "ไม่มีรายการ" (th) / "No items" (en)
t('count', { count: 0 })

// แสดง: "1 รายการ" (th) / "1 item" (en)
t('count', { count: 1 })

// แสดง: "5 รายการ" (th) / "5 items" (en)
t('count', { count: 5 })
```

### Select (Gender/Type)

```json
// messages/th.json
{
  "Profile": {
    "gender": "{gender, select, male {ชาย} female {หญิง} other {ไม่ระบุ}}",
    "greeting": "สวัสดี {gender, select, male {คุณผู้ชาย} female {คุณผู้หญิง} other {คุณ}} {name}!"
  }
}
```

```typescript
const t = useTranslations('Profile')

t('gender', { gender: 'male' })    // "ชาย"
t('gender', { gender: 'female' })  // "หญิง"
t('gender', { gender: 'other' })   // "ไม่ระบุ"

t('greeting', { gender: 'female', name: 'สมหญิง' })
// "สวัสดี คุณผู้หญิง สมหญิง!"
```

---

## Step 1481: Date/Number Formatting {#datenumber-formatting}

### วันที่และเวลา

```typescript
import { useFormatter } from 'next-intl'

export function DateDisplay({ date }: { date: Date }) {
  const format = useFormatter()
  
  return (
    <div>
      {/* วันที่แบบสั้น */}
      <p>{format.dateTime(date, { dateStyle: 'short' })}</p>
      {/* th: 1/1/2024, en: 1/1/2024 */}
      
      {/* วันที่แบบยาว */}
      <p>{format.dateTime(date, { dateStyle: 'long' })}</p>
      {/* th: 1 มกราคม 2567, en: January 1, 2024 */}
      
      {/* วันที่แบบ full */}
      <p>{format.dateTime(date, { dateStyle: 'full' })}</p>
      {/* th: วันจันทร์ที่ 1 มกราคม พ.ศ. 2567 */}
      
      {/* เวลา */}
      <p>{format.dateTime(date, { timeStyle: 'short' })}</p>
      {/* th: 09:30, en: 9:30 AM */}
      
      {/* วันที่และเวลา */}
      <p>{format.dateTime(date, { 
        dateStyle: 'medium', 
        timeStyle: 'short' 
      })}</p>
      
      {/* Custom format */}
      <p>{format.dateTime(date, {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        weekday: 'long',
      })}</p>
      
      {/* Relative Time */}
      <p>{format.relativeTime(date)}</p>
      {/* "5 นาทีที่แล้ว" หรือ "5 minutes ago" */}
      
      <p>{format.relativeTime(date, new Date('2024-06-01'))}</p>
    </div>
  )
}
```

### ตัวเลขและสกุลเงิน

```typescript
import { useFormatter } from 'next-intl'

export function NumberDisplay({ value }: { value: number }) {
  const format = useFormatter()
  
  return (
    <div>
      {/* ตัวเลขทั่วไป */}
      <p>{format.number(1234567.89)}</p>
      {/* th: 1,234,567.89, en: 1,234,567.89 */}
      
      {/* เปอร์เซ็นต์ */}
      <p>{format.number(0.85, { style: 'percent' })}</p>
      {/* th: 85%, en: 85% */}
      
      {/* สกุลเงิน */}
      <p>{format.number(1999.99, {
        style: 'currency',
        currency: 'THB',
      })}</p>
      {/* th: ฿1,999.99, en: THB 1,999.99 */}
      
      <p>{format.number(29.99, {
        style: 'currency',
        currency: 'USD',
      })}</p>
      {/* th: US$29.99, en: $29.99 */}
      
      {/* หน่วย */}
      <p>{format.number(100, {
        style: 'unit',
        unit: 'kilometer',
        unitDisplay: 'short',
      })}</p>
      {/* 100 กม., 100 km */}
      
      {/* จำนวนจุดทศนิยม */}
      <p>{format.number(3.14159, {
        maximumFractionDigits: 2,
      })}</p>
      {/* 3.14 */}
    </div>
  )
}
```

### ตั้งค่า Formats เริ่มต้น

```typescript
// i18n.ts
export default getRequestConfig(async ({ locale }) => {
  return {
    messages: (await import(`./messages/${locale}.json`)).default,
    formats: {
      dateTime: {
        short: {
          day: 'numeric',
          month: 'short',
          year: 'numeric',
        },
        long: {
          day: 'numeric',
          month: 'long',
          year: 'numeric',
          hour: 'numeric',
          minute: 'numeric',
        },
      },
      number: {
        precise: {
          maximumFractionDigits: 5,
        },
        currency: {
          style: 'currency',
          currency: locale === 'th' ? 'THB' : 'USD',
        },
      },
      list: {
        enumeration: {
          style: 'long',
          type: 'conjunction',
        },
      },
    },
  }
})
```

---

## Step 1482: RTL Support {#rtl-support}

RTL (Right-to-Left) ใช้กับภาษาอาหรับ, ฮีบรู, เปอร์เซีย เป็นต้น

### ตั้งค่า RTL ใน Next.js

```typescript
// app/[locale]/layout.tsx
const isRTL = (locale: string) => ['ar', 'he', 'fa', 'ur'].includes(locale)

export default async function LocaleLayout({
  children,
  params: { locale },
}) {
  return (
    <html 
      lang={locale} 
      dir={isRTL(locale) ? 'rtl' : 'ltr'}
    >
      <body>{children}</body>
    </html>
  )
}
```

### Tailwind CSS สำหรับ RTL

```typescript
// tailwind.config.ts
module.exports = {
  // เปิด RTL support
  plugins: [require('@tailwindcss/typography')],
}
```

```tsx
// ใช้ rtl: prefix ใน Tailwind
export function Card() {
  return (
    <div className="flex items-center gap-4 rtl:flex-row-reverse">
      <img className="w-12 h-12" />
      <div>
        {/* text align */}
        <h3 className="text-left rtl:text-right">Title</h3>
        {/* margin/padding */}
        <p className="ml-4 rtl:mr-4 rtl:ml-0">Description</p>
      </div>
    </div>
  )
}

// หรือใช้ CSS Logical Properties (แนะนำ)
export function CardLogical() {
  return (
    <div style={{ display: 'flex', gap: '1rem' }}>
      <img className="w-12 h-12" />
      <div>
        {/* margin-inline แทน margin-left/right */}
        <p className="ms-4">Description</p>
        {/* padding-inline */}
        <div className="ps-4 pe-4">Content</div>
      </div>
    </div>
  )
}
```

### CSS Logical Properties

```css
/* Physical Properties (ไม่แนะนำสำหรับ i18n) */
.element {
  margin-left: 1rem;
  padding-right: 2rem;
  border-left: 1px solid;
  text-align: left;
}

/* Logical Properties (แนะนำ) */
.element {
  margin-inline-start: 1rem;  /* = margin-left ใน LTR, margin-right ใน RTL */
  padding-inline-end: 2rem;   /* = padding-right ใน LTR, padding-left ใน RTL */
  border-inline-start: 1px solid;
  text-align: start;           /* = left ใน LTR, right ใน RTL */
}
```

### Icon Direction ใน RTL

```typescript
// components/BackButton.tsx
'use client'

import { useLocale } from 'next-intl'
import { ArrowLeft, ArrowRight } from 'lucide-react'

export function BackButton() {
  const locale = useLocale()
  const isRTL = ['ar', 'he', 'fa'].includes(locale)
  
  return (
    <button className="flex items-center gap-2">
      {isRTL ? <ArrowRight /> : <ArrowLeft />}
      <span>Back</span>
    </button>
  )
}

// หรือใช้ CSS transform
export function BackButtonCSS() {
  return (
    <button className="flex items-center gap-2">
      <ArrowLeft className="rtl:rotate-180" />
      <span>Back</span>
    </button>
  )
}
```

---

## Step 1483-1495: ตัวอย่าง Thai/English App เต็มรูปแบบ {#thai-english-app}

### โครงสร้าง Project

```
my-i18n-app/
├── app/
│   ├── [locale]/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── about/
│   │   │   └── page.tsx
│   │   ├── blog/
│   │   │   ├── page.tsx
│   │   │   └── [slug]/
│   │   │       └── page.tsx
│   │   └── contact/
│   │       └── page.tsx
│   └── globals.css
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── LanguageSwitcher.tsx
│   └── BlogCard.tsx
├── messages/
│   ├── th.json
│   └── en.json
├── i18n.ts
├── middleware.ts
└── next.config.ts
```

### App Implementation

```typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl'
import { getMessages, getTranslations } from 'next-intl/server'
import { unstable_setRequestLocale } from 'next-intl/server'
import type { Metadata } from 'next'
import { Header } from '@/components/Header'
import { Footer } from '@/components/Footer'

const locales = ['th', 'en']

export function generateStaticParams() {
  return locales.map((locale) => ({ locale }))
}

export async function generateMetadata({
  params: { locale },
}: {
  params: { locale: string }
}): Promise<Metadata> {
  const t = await getTranslations({ locale, namespace: 'Metadata' })
  
  return {
    title: {
      template: '%s | ' + t('siteName'),
      default: t('siteName'),
    },
    description: t('siteDescription'),
    metadataBase: new URL('https://example.com'),
  }
}

export default async function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode
  params: { locale: string }
}) {
  unstable_setRequestLocale(locale)
  const messages = await getMessages()
  
  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          <Header />
          <main>{children}</main>
          <Footer />
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
```

```typescript
// app/[locale]/page.tsx
import { useTranslations } from 'next-intl'
import { unstable_setRequestLocale } from 'next-intl/server'
import Link from 'next/link'

export default function HomePage({
  params: { locale },
}: {
  params: { locale: string }
}) {
  unstable_setRequestLocale(locale)
  const t = useTranslations('HomePage')
  
  return (
    <div className="min-h-screen">
      {/* Hero Section */}
      <section className="py-20 text-center bg-gradient-to-r from-blue-600 to-purple-600 text-white">
        <div className="container mx-auto px-4">
          <h1 className="text-4xl md:text-6xl font-bold mb-6">
            {t('hero.heading')}
          </h1>
          <p className="text-xl md:text-2xl mb-8 opacity-90">
            {t('hero.subheading')}
          </p>
          <Link
            href={`/${locale}/courses`}
            className="bg-white text-blue-600 px-8 py-3 rounded-full font-semibold hover:bg-opacity-90 transition"
          >
            {t('hero.cta')}
          </Link>
        </div>
      </section>
      
      {/* Features Section */}
      <section className="py-16">
        <div className="container mx-auto px-4">
          <h2 className="text-3xl font-bold text-center mb-12">
            {t('features.title')}
          </h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
            {['expert', 'practical', 'community'].map((feature) => (
              <div key={feature} className="text-center p-6 rounded-xl border">
                <h3 className="text-xl font-semibold mb-2">
                  {t(`features.items.${feature}`)}
                </h3>
              </div>
            ))}
          </div>
        </div>
      </section>
    </div>
  )
}
```

```typescript
// components/Header.tsx
import { useTranslations, useLocale } from 'next-intl'
import Link from 'next/link'
import { LanguageSwitcher } from './LanguageSwitcher'

export function Header() {
  const t = useTranslations('Navigation')
  const locale = useLocale()
  
  const navItems = [
    { href: '/', label: t('home') },
    { href: '/about', label: t('about') },
    { href: '/blog', label: t('blog') },
    { href: '/courses', label: t('courses') },
    { href: '/contact', label: t('contact') },
  ]
  
  return (
    <header className="sticky top-0 bg-white shadow-sm z-50">
      <div className="container mx-auto px-4">
        <div className="flex items-center justify-between h-16">
          <Link href={`/${locale}`} className="text-xl font-bold text-blue-600">
            MyApp
          </Link>
          
          <nav className="hidden md:flex items-center gap-6">
            {navItems.map((item) => (
              <Link
                key={item.href}
                href={`/${locale}${item.href}`}
                className="text-gray-600 hover:text-blue-600 transition-colors"
              >
                {item.label}
              </Link>
            ))}
          </nav>
          
          <LanguageSwitcher />
        </div>
      </div>
    </header>
  )
}
```

```json
// messages/th.json (เพิ่มเติม)
{
  "Metadata": {
    "siteName": "MyApp - เรียน React",
    "siteDescription": "เรียน React และ Next.js สำหรับนักพัฒนาทุกระดับ"
  },
  "HomePage": {
    "hero": {
      "heading": "พัฒนาทักษะ React ของคุณ",
      "subheading": "คอร์สออนไลน์คุณภาพสูงสำหรับนักพัฒนาทุกระดับ",
      "cta": "เริ่มเรียนฟรี"
    },
    "features": {
      "title": "ทำไมต้องเรียนกับเรา",
      "items": {
        "expert": "สอนโดยผู้เชี่ยวชาญ 10+ ปี",
        "practical": "Project จริงที่ใช้งานได้",
        "community": "Community นักพัฒนา 50,000+ คน"
      }
    }
  },
  "Navigation": {
    "home": "หน้าหลัก",
    "about": "เกี่ยวกับเรา",
    "blog": "บล็อก",
    "courses": "คอร์ส",
    "contact": "ติดต่อ"
  },
  "Blog": {
    "title": "บล็อก",
    "readMore": "อ่านเพิ่มเติม",
    "publishedOn": "เผยแพร่เมื่อ {date}",
    "by": "โดย {author}",
    "minRead": "{minutes} นาที"
  }
}
```

---

## Step 1496-1510: Advanced i18n Patterns

### Type-safe Translations

```typescript
// types/i18n.ts
import type en from '../messages/en.json'

// สร้าง type จาก en.json (source of truth)
type Messages = typeof en

// Declare module augmentation
declare global {
  interface IntlMessages extends Messages {}
}
```

### Server-side getTranslations

```typescript
// ใช้ใน Server Components และ Route Handlers
import { getTranslations } from 'next-intl/server'

export async function generateMetadata({ params: { locale } }) {
  const t = await getTranslations({ locale, namespace: 'HomePage' })
  return { title: t('title') }
}

// ใน Route Handler
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const locale = searchParams.get('locale') || 'th'
  
  const t = await getTranslations({ locale, namespace: 'API' })
  
  return Response.json({
    message: t('success'),
  })
}
```

### Timezone Support

```typescript
// i18n.ts
export default getRequestConfig(async ({ locale }) => {
  const timezone = {
    th: 'Asia/Bangkok',
    en: 'America/New_York',
    ja: 'Asia/Tokyo',
  }[locale] || 'UTC'
  
  return {
    messages: (await import(`./messages/${locale}.json`)).default,
    timeZone: timezone,
  }
})

// การใช้งาน
const format = useFormatter()
format.dateTime(date, { timeStyle: 'short' })
// th: 09:00 (UTC+7)
// en: 21:00 (UTC-5)
```

### Lazy Loading Messages

```typescript
// i18n.ts - โหลดเฉพาะ namespace ที่จำเป็น
export default getRequestConfig(async ({ locale }) => {
  return {
    messages: {
      ...(await import(`./messages/${locale}/common.json`)).default,
      ...(await import(`./messages/${locale}/navigation.json`)).default,
      // โหลด pages ที่จำเป็น
    },
  }
})
```

---

## 🧪 Quiz - Part 47

**ข้อ 1:** BCP 47 Language Tag สำหรับภาษาไทยคือ?
- A) `thai`
- B) `TH`
- C) `th`
- D) `th-TH`

**ข้อ 2:** `next-intl` middleware ทำหน้าที่อะไร?
- A) แปล content อัตโนมัติ
- B) ตรวจสอบ locale ของ request และ redirect ไปยัง URL ที่ถูกต้อง
- C) โหลด translation files
- D) ตรวจสอบ authentication

**ข้อ 3:** วิธีรองรับ RTL ใน HTML คือ?
- A) `<html rtl="true">`
- B) `<html direction="rtl">`
- C) `<html dir="rtl">`
- D) `<html text-align="right">`

**ข้อ 4:** CSS Logical Property ที่แทน `margin-left` ใน i18n คือ?
- A) `margin-start`
- B) `margin-inline-start`
- C) `margin-logical-left`
- D) `margin-ltr-left`

**เฉลย:** 1-C, 2-B, 3-C, 4-B

---

> **➡️ Next:** [Part 48: Real-time Features](./part-48-real-time-features.md)
