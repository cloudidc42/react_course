# Part 45: Accessibility (a11y)

## ข้อมูล Part
- **Steps:** 1401-1440
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3 ชั่วโมง
- **Prerequisites:** Part 44 (Design System)

---

## สารบัญ

1. [Web Accessibility (a11y)](#1-web-accessibility-a11y)
2. [Semantic HTML](#2-semantic-html)
3. [ARIA Attributes](#3-aria-attributes)
4. [Keyboard Navigation](#4-keyboard-navigation)
5. [Screen Reader](#5-screen-reader)
6. [Color Contrast](#6-color-contrast)
7. [Focus Management](#7-focus-management)
8. [Testing Accessibility](#8-testing-accessibility)
9. [Quiz](#quiz)

---

## Step 1401: Web Accessibility (a11y)

### 1. Web Accessibility (a11y)

Web Accessibility หมายถึง การทำให้ Website ใช้งานได้สำหรับทุกคน รวมถึงผู้พิการ

#### ทำไมต้องสำคัญ?

```
1. Moral - ทุกคนมีสิทธิ์เข้าถึงข้อมูล
2. Legal - กฎหมายหลายประเทศบังคับ (ADA, WCAG)
3. Business - ครอบคลุมผู้ใช้มากขึ้น (~15% ของประชากรโลก)
4. SEO - Search Engine ชอบ Semantic HTML
```

#### WCAG Guidelines

```
WCAG (Web Content Accessibility Guidelines)

Level A   - ขั้นต่ำ
Level AA  - มาตรฐาน (แนะนำ)
Level AAA - สูงสุด

4 หลักการ (POUR):
P - Perceivable   (มองเห็นได้)
O - Operable      (ใช้งานได้)
U - Understandable (เข้าใจได้)
R - Robust        (ทนทาน)
```

---

## Step 1405: Semantic HTML

### 2. Semantic HTML

Semantic HTML ใช้ Tags ที่มีความหมาย แทนการใช้แต่ div

#### Correct vs Incorrect

```html
<!-- ไม่ดี -->
<div class="header">
  <div class="nav">
    <div class="nav-item" onclick="...">Home</div>
  </div>
</div>

<div class="content">
  <div class="article">
    <div class="title">My Post</div>
    <div class="text">Content...</div>
  </div>
</div>

<!-- ดี -->
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>

<main>
  <article>
    <h1>My Post</h1>
    <p>Content...</p>
  </article>
</main>

<footer>
  <p>&copy; 2024 My App</p>
</footer>
```

#### Heading Hierarchy

```html
<!-- ไม่ดี - ข้าม Heading Levels -->
<h1>Page Title</h1>
<h3>Section</h3>  <!-- ข้าม h2 -->
<h5>Subsection</h5>

<!-- ดี - Sequential -->
<h1>Page Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>
```

#### Lists

```html
<!-- ไม่ดี -->
<div>
  <div>• Item 1</div>
  <div>• Item 2</div>
</div>

<!-- ดี -->
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```

#### Buttons vs Links

```html
<!-- Button = Action -->
<button type="button" onClick={handleDelete}>Delete</button>

<!-- Link = Navigation -->
<a href="/about">About</a>

<!-- ไม่ดี - div เป็น Button -->
<div onClick={handleClick} className="btn">Click me</div>

<!-- ดี -->
<button onClick={handleClick}>Click me</button>
```

---

## Step 1410: ARIA Attributes

### 3. ARIA Attributes

ARIA (Accessible Rich Internet Applications) เพิ่ม Semantic Information

#### aria-label

```tsx
// ปุ่มที่มีแค่ Icon ต้องมี aria-label
<button aria-label="Close dialog">
  <X className="h-4 w-4" />
</button>

// Input ที่ไม่มี Label
<input
  type="search"
  aria-label="Search products"
  placeholder="Search..."
/>
```

#### aria-describedby

```tsx
<div>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-describedby="email-hint email-error"
  />
  <p id="email-hint">We'll never share your email</p>
  {error && <p id="email-error" role="alert">{error}</p>}
</div>
```

#### aria-expanded, aria-controls

```tsx
function Accordion({ title, children }) {
  const [isOpen, setIsOpen] = useState(false)
  
  return (
    <div>
      <button
        aria-expanded={isOpen}
        aria-controls="accordion-content"
        onClick={() => setIsOpen(!isOpen)}
      >
        {title}
      </button>
      <div
        id="accordion-content"
        hidden={!isOpen}
        role="region"
      >
        {children}
      </div>
    </div>
  )
}
```

#### aria-live

```tsx
// แจ้ง Screen Reader เมื่อ Content เปลี่ยน
<div aria-live="polite" aria-atomic="true">
  {status && <p>{status}</p>}
</div>

// สำหรับ Error ที่สำคัญ
<div aria-live="assertive">
  {criticalError && <p role="alert">{criticalError}</p>}
</div>
```

#### aria-hidden

```tsx
// ซ่อนจาก Screen Reader
<span aria-hidden="true">👍</span>
<span className="sr-only">Like</span>

// Icon ที่ไม่ต้องการความหมาย
<svg aria-hidden="true" focusable="false">
  ...
</svg>
```

#### Roles

```tsx
// Landmark Roles
<div role="banner">Header</div>
<div role="navigation">Nav</div>
<div role="main">Main Content</div>
<div role="complementary">Sidebar</div>
<div role="contentinfo">Footer</div>

// Widget Roles
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</div>

<div role="tablist">
  <button role="tab" aria-selected="true">Tab 1</button>
  <button role="tab" aria-selected="false">Tab 2</button>
</div>
```

---

## Step 1416: Keyboard Navigation

### 4. Keyboard Navigation

ผู้ใช้ที่ใช้ Keyboard ต้องสามารถ Navigate ได้ทั้ง App

#### Tab Order

```tsx
// Focus จะไปตาม DOM Order ตามปกติ
// ใช้ tabIndex เมื่อจำเป็น

// tabIndex={0} - เพิ่มเข้า Tab Order
<div tabIndex={0} onKeyDown={handleKeyDown}>
  Focusable div
</div>

// tabIndex={-1} - Focusable แต่ไม่อยู่ใน Tab Order
<div tabIndex={-1} ref={dialogRef}>
  Dialog Content
</div>

// ไม่ควรใช้ tabIndex={1} หรือมากกว่า
```

#### Keyboard Event Handlers

```tsx
function MenuItem({ onClick, children }) {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault()
      onClick()
    }
  }
  
  return (
    <div
      role="menuitem"
      tabIndex={0}
      onClick={onClick}
      onKeyDown={handleKeyDown}
    >
      {children}
    </div>
  )
}
```

#### Skip Link

```tsx
// ให้ผู้ใช้ Skip ไปยัง Main Content ได้
export function SkipLink() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 focus:z-50 focus:p-4 focus:bg-white focus:text-black"
    >
      Skip to main content
    </a>
  )
}

// ใน Layout
<SkipLink />
<header>...</header>
<main id="main-content">
  {children}
</main>
```

---

## Step 1421: Screen Reader

### 5. Screen Reader

Screen Reader อ่านเนื้อหาให้ผู้ใช้ที่มองไม่เห็น

#### sr-only Class

```css
/* Tailwind sr-only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

```tsx
// Text ที่มองไม่เห็น แต่ Screen Reader อ่านได้
<button>
  <HeartIcon aria-hidden="true" />
  <span className="sr-only">Add to favorites</span>
</button>

// Loading State
<div role="status" aria-live="polite">
  {isLoading && (
    <>
      <Spinner aria-hidden="true" />
      <span className="sr-only">Loading...</span>
    </>
  )}
</div>
```

#### Image Alt Text

```tsx
// Informative Image
<img src="/product.jpg" alt="Red Nike Air Max 90 Running Shoes" />

// Decorative Image (ไม่มีความหมาย)
<img src="/decoration.png" alt="" role="presentation" />

// Image with Link
<a href="/products/1">
  <img src="/product.jpg" alt="View Nike Air Max 90 details" />
</a>

// Next.js Image
import Image from 'next/image'

<Image
  src="/product.jpg"
  alt="Red Nike Air Max 90"
  width={400}
  height={400}
/>
```

---

## Step 1426: Color Contrast

### 6. Color Contrast

Color Contrast Ratio ต้องผ่านมาตรฐาน WCAG

```
WCAG AA:
- Normal Text: Contrast Ratio >= 4.5:1
- Large Text (18pt+): Contrast Ratio >= 3:1
- UI Components: >= 3:1

WCAG AAA:
- Normal Text: >= 7:1
- Large Text: >= 4.5:1
```

#### Tools

```
1. WebAIM Contrast Checker: webaim.org/resources/contrastchecker
2. Chrome DevTools: ดู Contrast Ratio ใน Elements Panel
3. axe DevTools Browser Extension
4. Lighthouse (Chrome DevTools)
```

#### ตัวอย่าง

```tsx
// ไม่ดี - Contrast น้อยเกินไป
<p className="text-gray-300 bg-white">Low contrast text</p>
// White (#fff) vs Light Gray (#d1d5db) = ~1.3:1 (ไม่ผ่าน)

// ดี - Contrast เพียงพอ
<p className="text-gray-700 bg-white">Good contrast text</p>
// White (#fff) vs Dark Gray (#374151) = ~10:1 (ผ่าน)
```

#### ไม่ใช้สีอย่างเดียวในการสื่อความหมาย

```tsx
// ไม่ดี - ใช้สีอย่างเดียว
<span className="text-red-500">Error</span>

// ดี - ใช้สี + Icon + Text
<span className="flex items-center gap-1 text-red-500">
  <AlertCircle className="h-4 w-4" aria-hidden="true" />
  Error: Invalid email address
</span>
```

---

## Step 1430: Focus Management

### 7. Focus Management

จัดการ Focus ให้ถูกต้องสำหรับ Dynamic Content

#### Focus Trap (Modal)

```typescript
// hooks/useFocusTrap.ts
import { useEffect, useRef } from 'react'

export function useFocusTrap(isActive: boolean) {
  const containerRef = useRef<HTMLDivElement>(null)
  
  useEffect(() => {
    if (!isActive || !containerRef.current) return
    
    const focusableSelectors = [
      'a[href]',
      'button:not([disabled])',
      'input:not([disabled])',
      'select:not([disabled])',
      'textarea:not([disabled])',
      '[tabindex]:not([tabindex="-1"])',
    ].join(', ')
    
    const focusableElements = containerRef.current.querySelectorAll(focusableSelectors)
    const firstFocusable = focusableElements[0] as HTMLElement
    const lastFocusable = focusableElements[focusableElements.length - 1] as HTMLElement
    
    firstFocusable?.focus()
    
    const handleTabKey = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return
      
      if (e.shiftKey) {
        if (document.activeElement === firstFocusable) {
          lastFocusable?.focus()
          e.preventDefault()
        }
      } else {
        if (document.activeElement === lastFocusable) {
          firstFocusable?.focus()
          e.preventDefault()
        }
      }
    }
    
    document.addEventListener('keydown', handleTabKey)
    return () => document.removeEventListener('keydown', handleTabKey)
  }, [isActive])
  
  return containerRef
}
```

#### Accessible Modal

```tsx
// components/Modal.tsx
import { useEffect, useRef } from 'react'
import { useFocusTrap } from '@/hooks/useFocusTrap'

interface ModalProps {
  isOpen: boolean
  onClose: () => void
  title: string
  children: React.ReactNode
}

export function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const containerRef = useFocusTrap(isOpen)
  const previousFocusRef = useRef<HTMLElement | null>(null)
  
  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement as HTMLElement
    } else {
      previousFocusRef.current?.focus()
    }
  }, [isOpen])
  
  useEffect(() => {
    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose()
    }
    
    if (isOpen) {
      document.addEventListener('keydown', handleEsc)
    }
    
    return () => document.removeEventListener('keydown', handleEsc)
  }, [isOpen, onClose])
  
  if (!isOpen) return null
  
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      className="fixed inset-0 z-50 flex items-center justify-center"
    >
      <div
        className="fixed inset-0 bg-black/50"
        onClick={onClose}
        aria-hidden="true"
      />
      <div
        ref={containerRef}
        className="relative bg-white rounded-lg p-6 max-w-md w-full"
      >
        <h2 id="modal-title" className="text-xl font-bold mb-4">
          {title}
        </h2>
        
        {children}
        
        <button
          className="absolute top-4 right-4"
          onClick={onClose}
          aria-label="Close modal"
        >
          <X aria-hidden="true" />
        </button>
      </div>
    </div>
  )
}
```

---

## Step 1435: Testing Accessibility

### 8. Testing Accessibility

#### axe-core

```bash
npm install --save-dev @axe-core/react jest-axe
```

```typescript
// Button.test.tsx
import { render } from '@testing-library/react'
import { axe, toHaveNoViolations } from 'jest-axe'
import Button from './Button'

expect.extend(toHaveNoViolations)

test('Button has no accessibility violations', async () => {
  const { container } = render(<Button>Click me</Button>)
  
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

#### Playwright Accessibility

```typescript
// tests/e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

test('home page has no accessibility violations', async ({ page }) => {
  await page.goto('/')
  
  const results = await new AxeBuilder({ page }).analyze()
  
  expect(results.violations).toEqual([])
})

test('login page WCAG AA compliant', async ({ page }) => {
  await page.goto('/login')
  
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze()
  
  expect(results.violations).toEqual([])
})
```

#### eslint-plugin-jsx-a11y

```bash
npm install --save-dev eslint-plugin-jsx-a11y
```

```json
// .eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:jsx-a11y/recommended"
  ],
  "plugins": ["jsx-a11y"]
}
```

#### Manual Testing Checklist

```markdown
## Manual Testing

1. Keyboard Navigation
   [ ] Tab ผ่านทุก Interactive Element
   [ ] Focus Indicator มองเห็นได้ชัด
   [ ] ไม่มี Focus Trap ที่ไม่จำเป็น
   [ ] Skip Links ทำงาน

2. Screen Reader (NVDA, JAWS, VoiceOver)
   [ ] Headings อ่านสมเหตุสมผล
   [ ] Images มี Alt Text
   [ ] Form Fields มี Labels
   [ ] Error Messages ประกาศ

3. Color Contrast
   [ ] Text ผ่าน 4.5:1
   [ ] UI Components ผ่าน 3:1

4. Zoom
   [ ] 200% Zoom ใช้งานได้
   [ ] ไม่มี Horizontal Scroll ที่ 400%
```

---

## Step 1438: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## Accessibility Best Practices

1. Semantic HTML ก่อน ARIA เสมอ
   ใช้ <button> แทน <div role="button">

2. ทุก Interactive Element ต้องมี Accessible Name
   aria-label, aria-labelledby, หรือ Visible Text

3. Error Messages ต้องชัดเจน
   - บอกว่า Error อะไร
   - วิธีแก้ไข

4. Color ไม่ใช่ข้อมูลเดียว
   ใช้ Icon + Text ร่วมกับสี

5. Test ด้วย Keyboard เสมอ

6. ใช้ axe-core ใน Test Suite

7. Focus Management ใน Dynamic Content
   Modal, Dropdown, Toast
```

---

## Quiz

### แบบทดสอบ Part 45

**คำถามที่ 1:** WCAG ย่อมาจากอะไร?
- A) Web Content Accessibility Guidelines ✓
- B) Web Coding Access Guidelines
- C) Website Content Application Guidelines
- D) Web Component Accessibility Group

**คำถามที่ 2:** ทำไม `<button>` ดีกว่า `<div onClick={...}>` สำหรับ Action?
- A) เร็วกว่า
- B) มี Built-in Keyboard Support, Role, และ Focusable ✓
- C) Styling ง่ายกว่า
- D) ไม่ต่างกัน

**คำถามที่ 3:** `aria-live="polite"` ทำอะไร?
- A) ซ่อน Element จาก Screen Reader
- B) แจ้ง Screen Reader เมื่อ Content เปลี่ยน โดยรอให้ผู้ใช้ว่างก่อน ✓
- C) ทำให้ Element Focusable
- D) เพิ่ม Keyboard Shortcut

**คำถามที่ 4:** WCAG AA Text Contrast Ratio ขั้นต่ำคือเท่าไร?
- A) 2:1
- B) 3:1
- C) 4.5:1 ✓
- D) 7:1

**คำถามที่ 5:** Focus Trap ใน Modal สำคัญอย่างไร?
- A) ทำให้ Modal สวยขึ้น
- B) ป้องกัน Keyboard Users จากการ Tab ออกจาก Modal ✓
- C) ทำให้ Modal ปิดได้
- D) ปรับปรุง Performance

---

## สรุป Part 45

ใน Part นี้เราได้เรียนรู้:

1. **Web Accessibility** - WCAG Guidelines, POUR Principles
2. **Semantic HTML** - Tags ที่มีความหมาย
3. **ARIA Attributes** - aria-label, aria-live, roles
4. **Keyboard Navigation** - Tab Order, Skip Links
5. **Screen Reader** - sr-only, Alt Text
6. **Color Contrast** - WCAG Ratios
7. **Focus Management** - Focus Trap สำหรับ Modal
8. **Testing** - axe-core, eslint-plugin-jsx-a11y

---

## จบ Course Next.js!

ยินดีด้วยที่เรียนจบ Course นี้! คุณได้เรียนรู้:

- **React Basics** (Part 1-20)
- **Advanced React** (Part 21-30)
- **Next.js** (Part 31-41)
- **Testing** (Part 42-43)
- **Design System** (Part 44)
- **Accessibility** (Part 45)

---

**ขอบคุณที่เรียน React & Next.js Course!**
