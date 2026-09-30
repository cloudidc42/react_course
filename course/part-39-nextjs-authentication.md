# Part 39: Next.js Authentication

## ข้อมูล Part
- **Steps:** 1146-1190
- **ระดับ:** Advanced
- **เวลาเรียน:** 4-5 ชั่วโมง
- **Prerequisites:** Part 38 (Middleware)

---

## สารบัญ

1. [Auth Strategies Overview](#1-auth-strategies-overview)
2. [NextAuth.js v5 (Auth.js)](#2-nextauthjs-v5-authjs)
3. [Credentials Provider](#3-credentials-provider)
4. [OAuth (Google, GitHub)](#4-oauth-google-github)
5. [Session Management](#5-session-management)
6. [Protected Routes](#6-protected-routes)
7. [JWT vs Database Sessions](#7-jwt-vs-database-sessions)
8. [Role-based Access Control](#8-role-based-access-control)
9. [Quiz](#quiz)

---

## Step 1146: Auth Strategies Overview

### 1. Auth Strategies Overview

#### Authentication Patterns

```
1. Username/Password (Credentials)
   - Simple ที่สุด
   - ต้องจัดการ Security เอง
   - เหมาะกับ Internal Apps

2. OAuth / Social Login
   - Google, GitHub, Facebook
   - ไม่ต้องจัดการ Password
   - ผู้ใช้ไว้วางใจมากกว่า
   
3. Magic Link (Email)
   - ส่ง Link ไป Email
   - ไม่มี Password
   - Secure มาก

4. Multi-factor Authentication (MFA)
   - 2FA: OTP, Authenticator App
   - Security สูงสุด

5. SSO (Single Sign-On)
   - Enterprise: SAML, LDAP
   - OpenID Connect
```

#### Session Storage Options

```
1. JWT (JSON Web Token)
   - Stateless
   - ไม่ต้อง Database Lookup
   - ยกเลิกได้ยาก

2. Database Session
   - Stateful
   - ยกเลิกได้ทันที
   - ต้อง Database Lookup ทุกครั้ง

3. Cookie Session
   - HTTP-only Cookie
   - Secure, SameSite
   - Simple
```

---

## Step 1149: NextAuth.js v5

### 2. NextAuth.js v5 (Auth.js)

#### Installation

```bash
npm install next-auth@beta
npm install @auth/prisma-adapter  # ถ้าใช้ Database Sessions
```

#### Basic Setup

```typescript
// auth.ts (ระดับ Root)
import NextAuth from 'next-auth'
import Credentials from 'next-auth/providers/credentials'
import Google from 'next-auth/providers/google'
import GitHub from 'next-auth/providers/github'
import { PrismaAdapter } from '@auth/prisma-adapter'
import { prisma } from './lib/prisma'

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  
  providers: [
    Credentials({
      // ดูในส่วน Credentials Provider
    }),
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
  ],
  
  session: {
    strategy: 'jwt',  // หรือ 'database'
    maxAge: 30 * 24 * 60 * 60,  // 30 days
  },
  
  callbacks: {
    async jwt({ token, user, account }) {
      if (user) {
        token.id = user.id
        token.role = user.role
      }
      return token
    },
    
    async session({ session, token }) {
      if (token && session.user) {
        session.user.id = token.id as string
        session.user.role = token.role as string
      }
      return session
    },
  },
  
  pages: {
    signIn: '/login',
    signOut: '/logout',
    error: '/auth/error',
    verifyRequest: '/auth/verify-request',
  },
})

// app/api/auth/[...nextauth]/route.ts
export { handlers as GET, handlers as POST }
```

#### Type Augmentation

```typescript
// types/next-auth.d.ts
import { DefaultSession } from 'next-auth'

declare module 'next-auth' {
  interface Session {
    user: {
      id: string
      role: 'user' | 'admin' | 'moderator'
    } & DefaultSession['user']
  }
  
  interface User {
    role: 'user' | 'admin' | 'moderator'
  }
}

declare module 'next-auth/jwt' {
  interface JWT {
    id: string
    role: string
  }
}
```

#### .env Setup

```bash
# .env.local
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-very-long-secret-key  # openssl rand -base64 32

# Google
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# GitHub
GITHUB_ID=your-github-id
GITHUB_SECRET=your-github-secret

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
```

---

## Step 1153: Credentials Provider

### 3. Credentials Provider

#### Email/Password Login

```typescript
// auth.ts
import NextAuth from 'next-auth'
import Credentials from 'next-auth/providers/credentials'
import bcrypt from 'bcryptjs'
import { z } from 'zod'
import { prisma } from './lib/prisma'

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
})

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Credentials({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      
      async authorize(credentials) {
        // Validate input
        const result = LoginSchema.safeParse(credentials)
        if (!result.success) return null
        
        const { email, password } = result.data
        
        // Find user
        const user = await prisma.user.findUnique({
          where: { email },
          select: {
            id: true,
            name: true,
            email: true,
            password: true,
            role: true,
            emailVerified: true,
          }
        })
        
        if (!user || !user.password) return null
        
        // Check if email is verified
        if (!user.emailVerified) {
          throw new Error('Please verify your email first')
        }
        
        // Verify password
        const isValid = await bcrypt.compare(password, user.password)
        if (!isValid) return null
        
        // Return user object (ไม่รวม password)
        return {
          id: user.id,
          name: user.name,
          email: user.email,
          role: user.role,
        }
      },
    }),
  ],
})
```

#### Login Form

```tsx
// app/login/page.tsx
'use client'
import { useState } from 'react'
import { signIn } from 'next-auth/react'
import { useRouter, useSearchParams } from 'next/navigation'

export default function LoginPage() {
  const [error, setError] = useState('')
  const [loading, setLoading] = useState(false)
  const router = useRouter()
  const searchParams = useSearchParams()
  const redirect = searchParams.get('redirect') || '/dashboard'
  
  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    setLoading(true)
    setError('')
    
    const formData = new FormData(e.currentTarget)
    
    const result = await signIn('credentials', {
      email: formData.get('email') as string,
      password: formData.get('password') as string,
      redirect: false,
    })
    
    if (result?.error) {
      setError('อีเมลหรือรหัสผ่านไม่ถูกต้อง')
      setLoading(false)
      return
    }
    
    router.push(redirect)
    router.refresh()
  }
  
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8 p-8 bg-white rounded-xl shadow">
        <div>
          <h2 className="text-3xl font-bold text-center">เข้าสู่ระบบ</h2>
        </div>
        
        {error && (
          <div className="bg-red-50 border border-red-200 text-red-600 px-4 py-3 rounded">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">
              Email
            </label>
            <input
              name="email"
              type="email"
              required
              className="mt-1 block w-full border border-gray-300 rounded-md px-3 py-2"
              placeholder="your@email.com"
            />
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700">
              รหัสผ่าน
            </label>
            <input
              name="password"
              type="password"
              required
              className="mt-1 block w-full border border-gray-300 rounded-md px-3 py-2"
            />
          </div>
          
          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:opacity-50"
          >
            {loading ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
          </button>
        </form>
        
        <div className="relative">
          <div className="absolute inset-0 flex items-center">
            <div className="w-full border-t border-gray-300" />
          </div>
          <div className="relative flex justify-center text-sm">
            <span className="px-2 bg-white text-gray-500">หรือ</span>
          </div>
        </div>
        
        <div className="space-y-2">
          <button
            onClick={() => signIn('google', { callbackUrl: redirect })}
            className="w-full flex items-center justify-center gap-2 border border-gray-300 rounded-md py-2 px-4 hover:bg-gray-50"
          >
            <GoogleIcon />
            เข้าสู่ระบบด้วย Google
          </button>
          <button
            onClick={() => signIn('github', { callbackUrl: redirect })}
            className="w-full flex items-center justify-center gap-2 border border-gray-300 rounded-md py-2 px-4 hover:bg-gray-50"
          >
            <GitHubIcon />
            เข้าสู่ระบบด้วย GitHub
          </button>
        </div>
        
        <p className="text-center text-sm text-gray-600">
          ยังไม่มีบัญชี?{' '}
          <a href="/register" className="text-blue-600 hover:underline">
            สมัครสมาชิก
          </a>
        </p>
      </div>
    </div>
  )
}
```

#### Register Form

```tsx
// app/register/page.tsx
'use client'
import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { signIn } from 'next-auth/react'
import bcrypt from 'bcryptjs'

export default function RegisterPage() {
  const [error, setError] = useState('')
  const [loading, setLoading] = useState(false)
  const router = useRouter()
  
  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    setLoading(true)
    setError('')
    
    const formData = new FormData(e.currentTarget)
    const name = formData.get('name') as string
    const email = formData.get('email') as string
    const password = formData.get('password') as string
    const confirmPassword = formData.get('confirmPassword') as string
    
    if (password !== confirmPassword) {
      setError('รหัสผ่านไม่ตรงกัน')
      setLoading(false)
      return
    }
    
    // สร้าง Account
    const res = await fetch('/api/auth/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, email, password }),
    })
    
    if (!res.ok) {
      const data = await res.json()
      setError(data.error || 'เกิดข้อผิดพลาด')
      setLoading(false)
      return
    }
    
    // Auto Login หลังสมัคร
    await signIn('credentials', { email, password, callbackUrl: '/dashboard' })
  }
  
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="max-w-md w-full p-8 bg-white rounded-xl shadow">
        <h2 className="text-3xl font-bold text-center mb-6">สมัครสมาชิก</h2>
        
        {error && (
          <div className="bg-red-50 text-red-600 px-4 py-3 rounded mb-4">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label>ชื่อ</label>
            <input name="name" type="text" required className="input" />
          </div>
          <div>
            <label>Email</label>
            <input name="email" type="email" required className="input" />
          </div>
          <div>
            <label>รหัสผ่าน (อย่างน้อย 8 ตัวอักษร)</label>
            <input name="password" type="password" required minLength={8} className="input" />
          </div>
          <div>
            <label>ยืนยันรหัสผ่าน</label>
            <input name="confirmPassword" type="password" required className="input" />
          </div>
          <button type="submit" disabled={loading} className="btn-primary w-full">
            {loading ? 'กำลังสมัคร...' : 'สมัครสมาชิก'}
          </button>
        </form>
      </div>
    </div>
  )
}
```

#### Register API Route

```typescript
// app/api/auth/register/route.ts
import { NextRequest, NextResponse } from 'next/server'
import bcrypt from 'bcryptjs'
import { z } from 'zod'
import { prisma } from '@/lib/prisma'

const RegisterSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  password: z.string().min(8).regex(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    'Password must contain uppercase, lowercase and number'
  ),
})

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const data = RegisterSchema.parse(body)
    
    // Check if email already exists
    const existing = await prisma.user.findUnique({
      where: { email: data.email }
    })
    
    if (existing) {
      return NextResponse.json(
        { error: 'Email already registered' },
        { status: 409 }
      )
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(data.password, 12)
    
    // Create user
    const user = await prisma.user.create({
      data: {
        name: data.name,
        email: data.email,
        password: hashedPassword,
        role: 'user',
      },
      select: { id: true, name: true, email: true }
    })
    
    // TODO: Send verification email
    
    return NextResponse.json(user, { status: 201 })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      )
    }
    throw error
  }
}
```

---

## Step 1157: OAuth

### 4. OAuth (Google, GitHub)

#### Google OAuth Setup

```
1. ไปที่ Google Cloud Console
2. สร้าง Project
3. เปิดใช้ Google+ API
4. สร้าง Credentials → OAuth 2.0 Client IDs
5. เพิ่ม Authorized redirect URIs:
   - http://localhost:3000/api/auth/callback/google
   - https://myapp.com/api/auth/callback/google
```

#### GitHub OAuth Setup

```
1. ไปที่ GitHub Settings → Developer settings
2. OAuth Apps → New OAuth App
3. Homepage URL: http://localhost:3000
4. Authorization callback URL: 
   http://localhost:3000/api/auth/callback/github
```

#### OAuth ใน NextAuth

```typescript
// auth.ts
import NextAuth from 'next-auth'
import Google from 'next-auth/providers/google'
import GitHub from 'next-auth/providers/github'
import { PrismaAdapter } from '@auth/prisma-adapter'

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      authorization: {
        params: {
          prompt: 'consent',
          access_type: 'offline',
          response_type: 'code',
        },
      },
    }),
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
  ],
  
  callbacks: {
    async signIn({ user, account, profile }) {
      // Allow เฉพาะ Email ที่ Verify แล้ว
      if (account?.provider === 'google') {
        return profile?.email_verified === true
      }
      return true
    },
    
    async jwt({ token, user, account, profile }) {
      if (user) {
        token.id = user.id
        token.role = user.role || 'user'
      }
      if (account) {
        token.accessToken = account.access_token
        token.provider = account.provider
      }
      return token
    },
    
    async session({ session, token }) {
      session.user.id = token.id
      session.user.role = token.role
      return session
    },
  },
})
```

---

## Step 1160: Session Management

### 5. Session Management

#### ใช้ Session ใน Server Components

```tsx
// app/dashboard/page.tsx (Server Component)
import { auth } from '@/auth'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await auth()
  
  if (!session) {
    redirect('/login')
  }
  
  return (
    <div>
      <h1>สวัสดี, {session.user.name}!</h1>
      <p>Email: {session.user.email}</p>
      <p>Role: {session.user.role}</p>
    </div>
  )
}
```

#### ใช้ Session ใน Client Components

```tsx
// components/UserMenu.tsx (Client Component)
'use client'
import { useSession, signOut } from 'next-auth/react'

export default function UserMenu() {
  const { data: session, status } = useSession()
  
  if (status === 'loading') {
    return <div className="w-8 h-8 rounded-full bg-gray-200 animate-pulse" />
  }
  
  if (!session) {
    return (
      <a href="/login" className="btn-primary">
        เข้าสู่ระบบ
      </a>
    )
  }
  
  return (
    <div className="relative group">
      <button className="flex items-center gap-2">
        {session.user.image && (
          <img
            src={session.user.image}
            alt={session.user.name || ''}
            className="w-8 h-8 rounded-full"
          />
        )}
        <span>{session.user.name}</span>
      </button>
      
      <div className="absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-lg hidden group-hover:block">
        <a href="/account" className="block px-4 py-2 hover:bg-gray-100">
          โปรไฟล์
        </a>
        <a href="/dashboard" className="block px-4 py-2 hover:bg-gray-100">
          Dashboard
        </a>
        <button
          onClick={() => signOut({ callbackUrl: '/' })}
          className="w-full text-left px-4 py-2 hover:bg-gray-100 text-red-600"
        >
          ออกจากระบบ
        </button>
      </div>
    </div>
  )
}
```

#### SessionProvider Setup

```tsx
// app/layout.tsx
import { SessionProvider } from 'next-auth/react'
import { auth } from '@/auth'

export default async function RootLayout({ children }) {
  const session = await auth()
  
  return (
    <html>
      <body>
        <SessionProvider session={session}>
          {children}
        </SessionProvider>
      </body>
    </html>
  )
}
```

---

## Step 1163: Protected Routes

### 6. Protected Routes

#### Middleware Protection

```typescript
// middleware.ts
import { auth } from '@/auth'

export default auth((req) => {
  const isLoggedIn = !!req.auth
  const { pathname } = req.nextUrl
  
  const isAuthRoute = ['/login', '/register'].includes(pathname)
  const isDashboard = pathname.startsWith('/dashboard')
  const isAdmin = pathname.startsWith('/admin')
  
  if (isAuthRoute) {
    if (isLoggedIn) {
      return Response.redirect(new URL('/dashboard', req.nextUrl))
    }
    return null
  }
  
  if (!isLoggedIn && (isDashboard || isAdmin)) {
    const callbackUrl = encodeURIComponent(pathname)
    return Response.redirect(
      new URL(`/login?callbackUrl=${callbackUrl}`, req.nextUrl)
    )
  }
  
  if (isAdmin && req.auth?.user?.role !== 'admin') {
    return Response.redirect(new URL('/403', req.nextUrl))
  }
  
  return null
})

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

#### Component-level Protection

```tsx
// components/ProtectedRoute.tsx
import { auth } from '@/auth'
import { redirect } from 'next/navigation'

interface Props {
  children: React.ReactNode
  requiredRole?: 'user' | 'admin'
  redirectTo?: string
}

export default async function ProtectedRoute({
  children,
  requiredRole,
  redirectTo = '/login',
}: Props) {
  const session = await auth()
  
  if (!session) {
    redirect(redirectTo)
  }
  
  if (requiredRole && session.user.role !== requiredRole) {
    redirect('/403')
  }
  
  return <>{children}</>
}

// การใช้งาน
// app/admin/page.tsx
import ProtectedRoute from '@/components/ProtectedRoute'

export default function AdminPage() {
  return (
    <ProtectedRoute requiredRole="admin">
      <div>Admin Content</div>
    </ProtectedRoute>
  )
}
```

---

## Step 1167: JWT vs Database Sessions

### 7. JWT vs Database Sessions

#### JWT Sessions

```typescript
// auth.ts - JWT Strategy
export const { handlers, auth } = NextAuth({
  session: { strategy: 'jwt' },
  
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id
        token.role = user.role
      }
      return token
    },
    
    async session({ session, token }) {
      session.user.id = token.id
      session.user.role = token.role
      return session
    },
  },
})

// ข้อดี:
// - Stateless ไม่ต้อง DB Lookup
// - Scale ได้ดี
// - เร็ว

// ข้อเสีย:
// - ยกเลิก Session ยาก
// - ต้อง Refresh Token
// - ข้อมูลใน Token เก่าได้
```

#### Database Sessions

```typescript
// auth.ts - Database Strategy
import { PrismaAdapter } from '@auth/prisma-adapter'

export const { handlers, auth } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: 'database' },
  
  // ไม่ต้องมี jwt callback
  callbacks: {
    async session({ session, user }) {
      session.user.id = user.id
      session.user.role = user.role
      return session
    },
  },
})

// ข้อดี:
// - ยกเลิก Session ได้ทันที
// - ข้อมูลใน Session อัพเดทได้
// - Secure กว่า

// ข้อเสีย:
// - ต้อง DB Lookup ทุก Request
// - ช้ากว่าเล็กน้อย
```

#### Prisma Schema สำหรับ NextAuth

```prisma
// prisma/schema.prisma
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  password      String?   @db.Text
  role          Role      @default(USER)
  
  accounts Account[]
  sessions Session[]
  posts    Post[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime
  
  @@unique([identifier, token])
}

enum Role {
  USER
  MODERATOR
  ADMIN
}
```

---

## Step 1172: Role-based Access Control

### 8. Role-based Access Control

#### RBAC Setup

```typescript
// lib/rbac.ts
export type Role = 'user' | 'moderator' | 'admin'
export type Permission = 
  | 'posts:read'
  | 'posts:create'
  | 'posts:update:own'
  | 'posts:update:any'
  | 'posts:delete:own'
  | 'posts:delete:any'
  | 'users:read'
  | 'users:update:own'
  | 'users:update:any'
  | 'users:delete'
  | 'admin:access'

const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  user: [
    'posts:read',
    'posts:create',
    'posts:update:own',
    'posts:delete:own',
    'users:read',
    'users:update:own',
  ],
  moderator: [
    'posts:read',
    'posts:create',
    'posts:update:own',
    'posts:update:any',
    'posts:delete:own',
    'posts:delete:any',
    'users:read',
    'users:update:own',
  ],
  admin: [
    'posts:read',
    'posts:create',
    'posts:update:own',
    'posts:update:any',
    'posts:delete:own',
    'posts:delete:any',
    'users:read',
    'users:update:own',
    'users:update:any',
    'users:delete',
    'admin:access',
  ],
}

export function hasPermission(role: Role, permission: Permission): boolean {
  return ROLE_PERMISSIONS[role]?.includes(permission) ?? false
}

export function requirePermission(role: Role, permission: Permission): void {
  if (!hasPermission(role, permission)) {
    throw new Error(`Permission denied: ${permission}`)
  }
}
```

#### ใช้ RBAC ใน Server Components

```tsx
// app/posts/[id]/edit/page.tsx
import { auth } from '@/auth'
import { redirect } from 'next/navigation'
import { hasPermission } from '@/lib/rbac'
import { prisma } from '@/lib/prisma'
import { notFound } from 'next/navigation'

export default async function EditPostPage({ params }: { params: { id: string } }) {
  const session = await auth()
  
  if (!session) redirect('/login')
  
  const post = await prisma.post.findUnique({
    where: { id: params.id }
  })
  
  if (!post) notFound()
  
  const isAuthor = post.authorId === session.user.id
  const canEdit = isAuthor
    ? hasPermission(session.user.role as any, 'posts:update:own')
    : hasPermission(session.user.role as any, 'posts:update:any')
  
  if (!canEdit) redirect('/403')
  
  return (
    <div>
      <h1>แก้ไขบทความ</h1>
      <PostEditForm post={post} />
    </div>
  )
}
```

#### Permission Component

```tsx
// components/HasPermission.tsx
import { auth } from '@/auth'
import { hasPermission, Permission, Role } from '@/lib/rbac'

interface Props {
  permission: Permission
  children: React.ReactNode
  fallback?: React.ReactNode
}

export default async function HasPermission({
  permission,
  children,
  fallback = null,
}: Props) {
  const session = await auth()
  
  if (!session) return <>{fallback}</>
  
  const allowed = hasPermission(session.user.role as Role, permission)
  
  return allowed ? <>{children}</> : <>{fallback}</>
}

// การใช้งาน
export default async function PostActions({ post }) {
  const session = await auth()
  const isAuthor = post.authorId === session?.user?.id
  
  return (
    <div className="flex gap-2">
      <HasPermission permission="posts:update:own">
        {isAuthor && <a href={`/posts/${post.id}/edit`}>แก้ไข</a>}
      </HasPermission>
      
      <HasPermission permission="posts:delete:any">
        <DeleteButton postId={post.id} />
      </HasPermission>
    </div>
  )
}
```

#### ใช้ RBAC ใน API Routes

```typescript
// app/api/posts/[id]/route.ts
import { auth } from '@/auth'
import { hasPermission } from '@/lib/rbac'
import { prisma } from '@/lib/prisma'

export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  const session = await auth()
  
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  const post = await prisma.post.findUnique({ where: { id: params.id } })
  if (!post) {
    return Response.json({ error: 'Not found' }, { status: 404 })
  }
  
  const isAuthor = post.authorId === session.user.id
  const canDelete = isAuthor
    ? hasPermission(session.user.role as any, 'posts:delete:own')
    : hasPermission(session.user.role as any, 'posts:delete:any')
  
  if (!canDelete) {
    return Response.json({ error: 'Forbidden' }, { status: 403 })
  }
  
  await prisma.post.delete({ where: { id: params.id } })
  
  return new Response(null, { status: 204 })
}
```

---

## Step 1182: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Security

✓ ใช้ HTTPS เสมอ (Production)
✓ Set Secure, HttpOnly, SameSite ใน Cookies
✓ Hash Passwords ด้วย bcrypt (cost factor 12+)
✓ ไม่ส่ง Password Hash ไปยัง Client
✓ Rate Limit Auth Endpoints

## 2. Session Strategy

✓ JWT สำหรับ Simple Apps หรือ Microservices
✓ Database Sessions สำหรับ Apps ที่ต้องการ Revoke Sessions
✓ ตั้ง Session Expiry ที่เหมาะสม

## 3. OAuth

✓ ใช้ PKCE สำหรับ Security เพิ่มเติม
✓ ตรวจสอบ State Parameter ป้องกัน CSRF
✓ เก็บ Refresh Token ใน Database

## 4. RBAC

✓ เริ่มจาก Deny All หลักการ Least Privilege
✓ Check Permission ทั้งฝั่ง Client และ Server
✓ Log การเข้าถึงที่ไม่ได้รับอนุญาต

## 5. Error Messages

✓ ไม่บอกว่า Email มีในระบบหรือไม่ (ป้องกัน Enumeration)
✓ Generic Error Messages สำหรับ Auth Failures
```

---

## Quiz

### แบบทดสอบ Part 39

**คำถามที่ 1:** ข้อดีของ JWT Session เมื่อเทียบกับ Database Session คืออะไร?
- A) ปลอดภัยกว่า
- B) Stateless ไม่ต้อง Database Lookup ทุก Request ✓
- C) ยกเลิกได้ง่ายกว่า
- D) ข้อมูล Updated เสมอ

**คำถามที่ 2:** bcrypt ใช้ทำอะไร?
- A) Encrypt Data
- B) Hash Passwords แบบ One-way ที่ปลอดภัย ✓
- C) Sign JWT
- D) Generate Session Token

**คำถามที่ 3:** RBAC ย่อมาจากอะไร?
- A) React Based Access Control
- B) Role-Based Access Control ✓
- C) Request Based Authentication Control
- D) Response Based Access Control

**คำถามที่ 4:** ใน NextAuth v5, `auth()` ใน Server Component คืนค่าอะไร?
- A) JWT Token
- B) User ID
- C) Session Object หรือ null ✓
- D) Cookie

**คำถามที่ 5:** ทำไมไม่ควรบอก Error Message ว่า "Email ไม่มีในระบบ"?
- A) Error Message ควรกระชับ
- B) ป้องกัน Email Enumeration Attack ✓
- C) ประหยัด Bandwidth
- D) ไม่มีเหตุผล

---

## สรุป Part 39

ใน Part นี้เราได้เรียนรู้:

1. **Auth Strategies** - Credentials, OAuth, Magic Link, MFA
2. **NextAuth.js v5** - Setup, Providers, Callbacks
3. **Credentials** - Email/Password Login, Register
4. **OAuth** - Google, GitHub
5. **Session Management** - Server/Client Side
6. **Protected Routes** - Middleware, Component Level
7. **JWT vs Database** - Trade-offs
8. **RBAC** - Permission-based Access

---

➡️ **Part ถัดไป:** [Part 40: Database + Prisma](./part-40-nextjs-database-prisma.md)
