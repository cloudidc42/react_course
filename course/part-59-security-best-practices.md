# Part 59: Security Best Practices ใน React/Next.js

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1946-1985  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [OWASP Top 10 ใน React](#owasp-top-10)
2. [XSS Prevention](#xss-prevention)
3. [CSRF Protection](#csrf-protection)
4. [SQL Injection Prevention](#sql-injection)
5. [Input Validation](#input-validation)
6. [Content Security Policy](#csp)
7. [Environment Variables Security](#env-security)
8. [Authentication Security](#auth-security)
9. [Quiz](#quiz)

---

## Step 1946: OWASP Top 10 ใน React {#owasp-top-10}

OWASP (Open Web Application Security Project) Top 10 คือรายการ security risks ที่พบบ่อยที่สุด

```
OWASP Top 10 (2021):
1. Broken Access Control          → Authorization ไม่ดี
2. Cryptographic Failures         → Encryption ไม่ถูกต้อง
3. Injection                      → XSS, SQL Injection
4. Insecure Design                → ออกแบบไม่ปลอดภัย
5. Security Misconfiguration      → ตั้งค่าไม่ถูกต้อง
6. Vulnerable Components          → Dependencies เก่า
7. Auth & Session Failures        → Authentication อ่อนแอ
8. Software Integrity Failures    → Supply chain attacks
9. Logging & Monitoring Failures  → ไม่มี monitoring
10. Server-Side Request Forgery   → SSRF attacks
```

---

## Step 1947-1953: XSS Prevention {#xss-prevention}

XSS (Cross-Site Scripting) เกิดเมื่อ attacker inject JavaScript เข้าไปในหน้าเว็บ

### ประเภท XSS

```
1. Stored XSS:
   - Attacker ส่ง malicious script ไปเก็บใน database
   - ทุกคนที่เข้าหน้านั้นโดน attack

2. Reflected XSS:
   - Malicious script ใน URL parameter
   - เหยื่อต้องคลิก link นั้น

3. DOM-based XSS:
   - Manipulation ของ DOM โดยตรงใน JavaScript
```

### React ป้องกัน XSS โดยอัตโนมัติ

```typescript
// ✅ React escape HTML โดยอัตโนมัติ
function SafeComponent({ userInput }: { userInput: string }) {
  return <div>{userInput}</div>
  // แม้ userInput = '<script>alert("xss")</script>'
  // จะแสดงเป็น text ไม่ใช่ execute script
}

// ⚠️ dangerouslySetInnerHTML - อันตราย!
function DangerousComponent({ html }: { html: string }) {
  // ✅ Sanitize ก่อนเสมอ
  const sanitizedHtml = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a', 'ul', 'li', 'ol'],
    ALLOWED_ATTR: ['href', 'target'],
    FORCE_HTTPS: true,
  })
  
  return (
    <div
      dangerouslySetInnerHTML={{ __html: sanitizedHtml }}
    />
  )
}

// ❌ อย่าทำแบบนี้
function XSSVulnerable({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />
}
```

### DOMPurify Setup

```bash
npm install dompurify
npm install -D @types/dompurify
```

```typescript
// lib/sanitize.ts
import DOMPurify from 'dompurify'

// Config สำหรับ HTML content ทั่วไป
export function sanitizeHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: [
      'p', 'br', 'b', 'i', 'u', 'em', 'strong',
      'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
      'ul', 'ol', 'li',
      'a', 'img',
      'blockquote', 'pre', 'code',
      'table', 'thead', 'tbody', 'tr', 'th', 'td',
    ],
    ALLOWED_ATTR: [
      'href', 'src', 'alt', 'title', 'class',
      'target', 'rel', 'width', 'height',
    ],
    FORBID_ATTR: ['style', 'onerror', 'onload', 'onclick'],
    FORCE_HTTPS: true,
    ADD_ATTR: ['rel'], // เพิ่ม rel="noopener noreferrer" สำหรับ external links
  })
}

// Config ที่เข้มงวดกว่า สำหรับ user comments
export function sanitizeComment(text: string): string {
  return DOMPurify.sanitize(text, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href'],
    FORCE_HTTPS: true,
  })
}

// Strip HTML ทั้งหมด (สำหรับ plain text)
export function stripHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: [],
    ALLOWED_ATTR: [],
  })
}
```

### XSS ใน URL

```typescript
// ⚠️ อันตราย - javascript: URL
function DangerousLink({ href }) {
  return <a href={href}>Click me</a>
}

// ✅ ปลอดภัย - Validate URL
function SafeLink({ href, children }) {
  const isValidUrl = (url: string) => {
    try {
      const parsed = new URL(url)
      return ['http:', 'https:'].includes(parsed.protocol)
    } catch {
      return false
    }
  }
  
  if (!isValidUrl(href)) {
    return <span>{children}</span>
  }
  
  return (
    <a
      href={href}
      target="_blank"
      rel="noopener noreferrer"  // ป้องกัน tab-napping
    >
      {children}
    </a>
  )
}
```

---

## Step 1954-1958: CSRF Protection {#csrf-protection}

CSRF (Cross-Site Request Forgery) คือการหลอกให้ victim ส่ง request ที่ไม่ต้องการ

```
CSRF Attack:
1. Victim login ที่ bank.com
2. Victim เข้า evil.com
3. evil.com มี form ที่ส่งไปที่ bank.com/transfer
4. Browser ส่ง cookie ของ bank.com ไปด้วย
5. Bank โอนเงินโดยไม่รู้ว่าเป็น evil.com

ป้องกัน:
1. SameSite Cookie
2. CSRF Token
3. Double Submit Cookie
4. Verify Origin Header
```

### CSRF Protection ใน Next.js

```typescript
// lib/csrf.ts
import { randomBytes, createHmac } from 'crypto'
import { cookies } from 'next/headers'

const CSRF_SECRET = process.env.CSRF_SECRET!
const CSRF_COOKIE_NAME = 'csrf-token'

export function generateCSRFToken(): string {
  const random = randomBytes(32).toString('hex')
  const timestamp = Date.now().toString()
  const signature = createHmac('sha256', CSRF_SECRET)
    .update(`${random}:${timestamp}`)
    .digest('hex')
  
  return `${random}:${timestamp}:${signature}`
}

export function validateCSRFToken(token: string): boolean {
  const parts = token.split(':')
  if (parts.length !== 3) return false
  
  const [random, timestamp, signature] = parts
  
  // ตรวจสอบอายุ token (1 ชั่วโมง)
  if (Date.now() - parseInt(timestamp) > 3600000) return false
  
  const expectedSignature = createHmac('sha256', CSRF_SECRET)
    .update(`${random}:${timestamp}`)
    .digest('hex')
  
  // Timing-safe comparison
  return timingSafeEqual(signature, expectedSignature)
}

function timingSafeEqual(a: string, b: string): boolean {
  if (a.length !== b.length) return false
  let result = 0
  for (let i = 0; i < a.length; i++) {
    result |= a.charCodeAt(i) ^ b.charCodeAt(i)
  }
  return result === 0
}

// Middleware
// app/api/csrf/route.ts
export async function GET() {
  const token = generateCSRFToken()
  
  const response = Response.json({ token })
  response.headers.set(
    'Set-Cookie',
    `${CSRF_COOKIE_NAME}=${token}; SameSite=Strict; HttpOnly; Secure; Path=/`
  )
  
  return response
}

// Validate ใน Server Actions
export async function validateRequest(request: Request) {
  const token = request.headers.get('x-csrf-token')
  const cookieToken = cookies().get(CSRF_COOKIE_NAME)?.value
  
  if (!token || !cookieToken || token !== cookieToken) {
    throw new Error('Invalid CSRF token')
  }
  
  if (!validateCSRFToken(token)) {
    throw new Error('CSRF token validation failed')
  }
}
```

### SameSite Cookie

```typescript
// lib/auth.ts - การตั้งค่า Cookie ที่ปลอดภัย
export function setAuthCookie(response: Response, token: string) {
  response.headers.append(
    'Set-Cookie',
    [
      `auth-token=${token}`,
      'HttpOnly',           // ไม่ให้ JavaScript อ่าน
      'Secure',             // HTTPS เท่านั้น
      'SameSite=Strict',    // ป้องกัน CSRF
      'Path=/',
      `Max-Age=${60 * 60 * 24 * 7}`,  // 7 วัน
    ].join('; ')
  )
}
```

---

## Step 1959-1963: SQL Injection Prevention {#sql-injection}

```
SQL Injection:
Attacker ส่ง: username = "admin' OR '1'='1"
Query กลายเป็น: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
ผล: Login ได้โดยไม่ต้องมี password!
```

### ป้องกันด้วย Parameterized Queries

```typescript
// ❌ SQL Injection Vulnerable
async function getUserVulnerable(username: string) {
  const query = `SELECT * FROM users WHERE username = '${username}'`
  return db.execute(query) // อันตราย!
}

// ✅ Parameterized Query
async function getUserSafe(username: string) {
  // Prisma - ปลอดภัยโดยอัตโนมัติ
  return prisma.user.findUnique({
    where: { username },
  })
}

// ✅ Raw SQL กับ Parameterized
async function searchUsers(query: string) {
  // Prisma raw ที่ปลอดภัย
  return prisma.$queryRaw`
    SELECT * FROM users 
    WHERE name ILIKE ${'%' + query + '%'}
    ORDER BY created_at DESC
    LIMIT 20
  `
}

// ✅ drizzle-orm
import { eq, like } from 'drizzle-orm'

async function searchUsersDrizzle(query: string) {
  return db.select()
    .from(users)
    .where(like(users.name, `%${query}%`))
    .limit(20)
}
```

---

## Step 1964-1968: Input Validation {#input-validation}

```typescript
// lib/validators.ts กับ Zod
import { z } from 'zod'

// User Registration Schema
export const RegisterSchema = z.object({
  name: z
    .string()
    .min(2, 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร')
    .max(100, 'ชื่อต้องไม่เกิน 100 ตัวอักษร')
    .trim()
    .regex(/^[a-zA-Zก-๙\s]+$/, 'ชื่อต้องเป็นตัวอักษรเท่านั้น'),
  
  email: z
    .string()
    .email('อีเมลไม่ถูกต้อง')
    .toLowerCase()
    .trim(),
  
  password: z
    .string()
    .min(8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    .max(128, 'รหัสผ่านต้องไม่เกิน 128 ตัวอักษร')
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/,
      'รหัสผ่านต้องมีตัวพิมพ์เล็ก ตัวพิมพ์ใหญ่ ตัวเลข และอักขระพิเศษ'
    ),
  
  phone: z
    .string()
    .regex(/^0[689]\d{8}$/, 'เบอร์โทรศัพท์ไม่ถูกต้อง')
    .optional(),
  
  age: z
    .number()
    .int('อายุต้องเป็นจำนวนเต็ม')
    .min(18, 'ต้องมีอายุอย่างน้อย 18 ปี')
    .max(120, 'อายุไม่ถูกต้อง'),
})

export type RegisterInput = z.infer<typeof RegisterSchema>

// Validate ใน Server Action
async function registerUser(formData: FormData) {
  'use server'
  
  const rawData = {
    name: formData.get('name'),
    email: formData.get('email'),
    password: formData.get('password'),
    age: Number(formData.get('age')),
  }
  
  // Validate
  const result = RegisterSchema.safeParse(rawData)
  
  if (!result.success) {
    return {
      success: false,
      errors: result.error.flatten().fieldErrors,
    }
  }
  
  // ข้อมูลผ่านการ validate แล้ว
  const validData = result.data
  
  // Hash password
  const hashedPassword = await hashPassword(validData.password)
  
  // สร้าง user
  await prisma.user.create({
    data: {
      name: validData.name,
      email: validData.email,
      password: hashedPassword,
    },
  })
  
  return { success: true }
}
```

---

## Step 1969-1972: Content Security Policy {#csp}

CSP บอก Browser ว่า resource ใดที่อนุญาตให้โหลดได้

```typescript
// middleware.ts
import { NextResponse } from 'next/server'

const CSP = `
  default-src 'self';
  script-src 'self' 'unsafe-eval' 'unsafe-inline' https://cdn.example.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  img-src 'self' data: https: blob:;
  media-src 'self' https:;
  connect-src 'self' https://api.example.com wss://ws.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
`.replace(/\s+/g, ' ').trim()

export function middleware(request: Request) {
  const response = NextResponse.next()
  
  response.headers.set('Content-Security-Policy', CSP)
  response.headers.set('X-Content-Type-Options', 'nosniff')
  response.headers.set('X-Frame-Options', 'DENY')
  response.headers.set('X-XSS-Protection', '1; mode=block')
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin')
  response.headers.set(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), interest-cohort=()'
  )
  
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    )
  }
  
  return response
}
```

---

## Step 1973-1977: Environment Variables Security {#env-security}

```
Environment Variables Rules:
├── NEXT_PUBLIC_* = เห็นได้ทั้ง Server และ Client
└── ไม่มี prefix = เห็นได้เฉพาะ Server

❌ อันตราย:
NEXT_PUBLIC_DATABASE_URL=postgresql://...
NEXT_PUBLIC_API_SECRET=secret-key
NEXT_PUBLIC_JWT_SECRET=my-secret

✅ ปลอดภัย:
NEXT_PUBLIC_APP_URL=https://example.com   (public info เท่านั้น)
DATABASE_URL=postgresql://...             (server-only)
API_SECRET=secret-key                     (server-only)
JWT_SECRET=my-secret                      (server-only)
```

```typescript
// lib/env.ts - Validate Environment Variables
import { z } from 'zod'

const serverEnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NEXTAUTH_SECRET: z.string().min(32),
  API_SECRET_KEY: z.string().min(16),
  SMTP_HOST: z.string(),
  SMTP_PASSWORD: z.string(),
})

const clientEnvSchema = z.object({
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
})

// Validate เมื่อ server start
export function validateEnv() {
  if (typeof window === 'undefined') {
    const result = serverEnvSchema.safeParse(process.env)
    if (!result.success) {
      console.error('Invalid environment variables:', result.error.format())
      throw new Error('Invalid environment configuration')
    }
  }
  
  const clientResult = clientEnvSchema.safeParse(process.env)
  if (!clientResult.success) {
    throw new Error('Invalid client environment variables')
  }
}

// .env.example (commit ไว้ใน repo)
// DATABASE_URL=postgresql://username:password@host:5432/db
// JWT_SECRET=generate-with-openssl-rand-hex-64
// NEXTAUTH_SECRET=generate-with-openssl-rand-hex-32

// .env.local (ห้าม commit!)
// เพิ่มใน .gitignore เสมอ
```

---

## Step 1978-1985: Authentication Security {#auth-security}

```typescript
// lib/auth.ts - Secure Password Hashing
import bcrypt from 'bcryptjs'
import { randomBytes } from 'crypto'

const SALT_ROUNDS = 12  // ยิ่งมาก ยิ่งปลอดภัย (แต่ช้ากว่า)

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS)
}

export async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return bcrypt.compare(password, hash)
}

// Rate Limiting สำหรับ Login
import { LRUCache } from 'lru-cache'

const loginAttempts = new LRUCache<string, { count: number; firstAttempt: number }>({
  max: 10000,
  ttl: 15 * 60 * 1000, // 15 minutes
})

const MAX_ATTEMPTS = 5
const LOCKOUT_DURATION = 15 * 60 * 1000 // 15 minutes

export function checkRateLimit(identifier: string): {
  allowed: boolean
  remainingAttempts: number
  resetAt?: Date
} {
  const data = loginAttempts.get(identifier)
  
  if (!data) {
    loginAttempts.set(identifier, { count: 1, firstAttempt: Date.now() })
    return { allowed: true, remainingAttempts: MAX_ATTEMPTS - 1 }
  }
  
  if (data.count >= MAX_ATTEMPTS) {
    const resetAt = new Date(data.firstAttempt + LOCKOUT_DURATION)
    return { allowed: false, remainingAttempts: 0, resetAt }
  }
  
  data.count++
  loginAttempts.set(identifier, data)
  
  return {
    allowed: true,
    remainingAttempts: MAX_ATTEMPTS - data.count,
  }
}

// JWT Security
import jwt from 'jsonwebtoken'

const JWT_SECRET = process.env.JWT_SECRET!
const JWT_EXPIRES_IN = '15m'  // Short-lived access token
const REFRESH_TOKEN_EXPIRES_IN = '7d'

interface TokenPayload {
  userId: string
  email: string
  role: string
  sessionId: string
}

export function generateTokens(payload: Omit<TokenPayload, 'sessionId'>) {
  const sessionId = randomBytes(16).toString('hex')
  
  const accessToken = jwt.sign(
    { ...payload, sessionId },
    JWT_SECRET,
    {
      expiresIn: JWT_EXPIRES_IN,
      algorithm: 'HS256',
      issuer: 'myapp',
      audience: 'myapp-client',
    }
  )
  
  const refreshToken = jwt.sign(
    { userId: payload.userId, sessionId },
    JWT_SECRET,
    {
      expiresIn: REFRESH_TOKEN_EXPIRES_IN,
      algorithm: 'HS256',
    }
  )
  
  return { accessToken, refreshToken, sessionId }
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, JWT_SECRET, {
    algorithms: ['HS256'],
    issuer: 'myapp',
    audience: 'myapp-client',
  }) as TokenPayload
}
```

---

## 🧪 Quiz - Part 59

**ข้อ 1:** XSS Attack คืออะไร?
- A) การ overflow database
- B) การ inject malicious JavaScript ลงในหน้าเว็บ
- C) การขโมย SSL Certificate
- D) การ brute force password

**ข้อ 2:** `dangerouslySetInnerHTML` ควรทำอะไรก่อนใช้?
- A) ตรวจสอบ user authentication
- B) Sanitize HTML ด้วย DOMPurify
- C) Minify HTML
- D) Encode เป็น Base64

**ข้อ 3:** `SameSite=Strict` ใน Cookie ป้องกันอะไร?
- A) XSS
- B) CSRF
- C) SQL Injection
- D) Brute force

**ข้อ 4:** Environment variable ที่ขึ้นต้นด้วย `NEXT_PUBLIC_` แตกต่างอย่างไร?
- A) Encrypt อัตโนมัติ
- B) เข้าถึงได้จาก Client-side JavaScript
- C) เก็บใน database
- D) อ่านได้เฉพาะ Server

**เฉลย:** 1-B, 2-B, 3-B, 4-B

---

> **➡️ Next:** [Part 60: Performance Advanced](./part-60-performance-advanced.md)
