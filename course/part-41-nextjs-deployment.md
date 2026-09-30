# Part 41: Next.js Deployment

## ข้อมูล Part
- **Steps:** 1236-1275
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3 ชั่วโมง
- **Prerequisites:** Part 40 (Database + Prisma)

---

## สารบัญ

1. [Deploy บน Vercel](#1-deploy-บน-vercel)
2. [Environment Variables ใน Vercel](#2-environment-variables-ใน-vercel)
3. [Custom Domain](#3-custom-domain)
4. [Preview Deployments](#4-preview-deployments)
5. [Deploy บน Railway/Render](#5-deploy-บน-railwayrender)
6. [Docker Container](#6-docker-container)
7. [CI/CD with GitHub Actions](#7-cicd-with-github-actions)
8. [Performance Monitoring](#8-performance-monitoring)
9. [Quiz](#quiz)

---

## Step 1236: Deploy บน Vercel

### 1. Deploy บน Vercel

Vercel คือ Platform ที่สร้างโดย ผู้สร้าง Next.js รองรับ Next.js ทุก Feature

#### วิธี Deploy

**Method 1: Vercel CLI**

```bash
# ติดตั้ง Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy (จาก project root)
vercel

# Deploy to Production
vercel --prod
```

**Method 2: GitHub Integration (แนะนำ)**

```
1. Push code ไป GitHub
2. เข้า vercel.com
3. "Add New Project"
4. Import GitHub Repository
5. Configure Settings
6. Deploy
```

#### Vercel Configuration

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "installCommand": "npm install",
  "framework": "nextjs",
  
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        }
      ]
    }
  ],
  
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ],
  
  "rewrites": [
    {
      "source": "/api/v1/:path*",
      "destination": "https://api.example.com/:path*"
    }
  ]
}
```

#### next.config.js สำหรับ Production

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable React Strict Mode
  reactStrictMode: true,
  
  // Image Optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'res.cloudinary.com',
      },
      {
        protocol: 'https',
        hostname: '*.amazonaws.com',
      },
    ],
    formats: ['image/avif', 'image/webp'],
  },
  
  // Output Configuration
  output: 'standalone',  // สำหรับ Docker
  // หรือ
  output: 'export',  // Static HTML Export
  
  // Compress
  compress: true,
  
  // Power By Header
  poweredByHeader: false,
  
  // Experimental
  experimental: {
    // Turbopack
    turbo: {},
  },
}

module.exports = nextConfig
```

---

## Step 1240: Environment Variables

### 2. Environment Variables ใน Vercel

#### ประเภทของ Environment Variables

```
Development:  .env.local (ไม่ commit ไปยัง Git)
Preview:      สำหรับ Preview Deployments
Production:   สำหรับ Production

Public (NEXT_PUBLIC_*): ส่งไปยัง Browser
Private:                เฉพาะ Server เท่านั้น
```

#### ตั้งค่าใน Vercel Dashboard

```
Project Settings → Environment Variables
→ Add Variable
→ Key: DATABASE_URL
→ Value: postgresql://...
→ Environment: Production, Preview, Development
```

#### ตั้งค่าด้วย Vercel CLI

```bash
# เพิ่ม Environment Variable
vercel env add DATABASE_URL

# Pull Environment Variables ลง Local
vercel env pull .env.local

# List Environment Variables
vercel env ls

# Remove Environment Variable
vercel env rm DATABASE_URL
```

#### .env Files

```bash
# .env (commit ได้, ไม่มี secrets)
NEXT_PUBLIC_APP_NAME="My App"
NEXT_PUBLIC_APP_URL="https://myapp.com"

# .env.local (ไม่ commit)
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=...
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
STRIPE_SECRET_KEY=...

# .env.production (commit ได้, ไม่มี secrets)
NEXT_PUBLIC_APP_URL="https://myapp.com"
NEXT_PUBLIC_API_URL="https://api.myapp.com"

# .env.development (commit ได้)
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

#### การ Validate Environment Variables

```typescript
// lib/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  GOOGLE_CLIENT_ID: z.string().optional(),
  GOOGLE_CLIENT_SECRET: z.string().optional(),
})

export const env = envSchema.parse(process.env)

// การใช้งาน
import { env } from '@/lib/env'

const db = new PrismaClient({ datasources: { db: { url: env.DATABASE_URL } } })
```

---

## Step 1243: Custom Domain

### 3. Custom Domain

#### ตั้งค่า Custom Domain

```
1. ซื้อ Domain (Namecheap, GoDaddy, Google Domains)
2. เข้า Vercel Project → Settings → Domains
3. Add Domain: myapp.com
4. ตั้งค่า DNS:
   - CNAME: www → cname.vercel-dns.com
   - A Record: @ → 76.76.21.21
5. รอ DNS Propagation (5-10 นาที ถึง 24 ชั่วโมง)
```

#### ตั้งค่า www Redirect

```
Vercel จะ Redirect เอง:
myapp.com → www.myapp.com (หรือกลับกัน)

ตั้งค่าใน Vercel:
Settings → Domains → Primary Domain
```

#### SSL Certificate

```
Vercel จัดการ SSL ให้อัตโนมัติ:
- Let's Encrypt Certificate
- Auto-renew
- HTTPS Redirect อัตโนมัติ
```

---

## Step 1246: Preview Deployments

### 4. Preview Deployments

#### ทำงานอย่างไร

```
Git Push ไป Branch ใดก็ได้
→ Vercel Build อัตโนมัติ
→ สร้าง Unique URL: https://myapp-git-feature-xyz.vercel.app
→ Share URL กับ Team

Merge ไป main
→ Deploy ไปยัง Production
```

#### ปรับแต่ง Preview Deployments

```json
// vercel.json
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "develop": true,
      "*": true
    }
  }
}
```

#### Branch-specific Environment Variables

```
Vercel Dashboard → Environment Variables
→ Value: postgresql://staging-db...
→ Git Branch: staging (specific branch)
```

---

## Step 1249: Railway/Render

### 5. Deploy บน Railway/Render

#### Railway

```bash
# ติดตั้ง Railway CLI
npm i -g @railway/cli

# Login
railway login

# สร้าง Project
railway init

# Deploy
railway up

# Add Service (PostgreSQL)
railway add
# เลือก PostgreSQL

# ดู Environment Variables
railway variables
```

#### Railway Configuration

```
railway.toml
```

```toml
[build]
builder = "NIXPACKS"
buildCommand = "npm run build"

[deploy]
startCommand = "npm start"
healthcheckPath = "/api/health"
healthcheckTimeout = 100
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 10
```

#### Render

```yaml
# render.yaml
services:
  - type: web
    name: my-nextjs-app
    env: node
    buildCommand: npm install && npm run build
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: my-postgres
          property: connectionString
          
databases:
  - name: my-postgres
    databaseName: myapp
    user: myapp
```

---

## Step 1252: Docker Container

### 6. Docker Container

#### Dockerfile สำหรับ Next.js

```dockerfile
# Dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install dependencies
RUN npm ci --only=production
RUN npx prisma generate

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build Next.js
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Stage 3: Runner
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy built files
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### next.config.js สำหรับ Docker

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',  // สร้าง Standalone Bundle
}

module.exports = nextConfig
```

#### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - NEXTAUTH_SECRET=your-secret
      - NEXTAUTH_URL=http://localhost:3000
    depends_on:
      - db
    restart: unless-stopped
    
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
      
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - app

volumes:
  postgres_data:
```

#### .dockerignore

```
.git
.gitignore
node_modules
.next
.env*
!.env.example
README.md
Dockerfile*
docker-compose*
```

---

## Step 1256: CI/CD with GitHub Actions

### 7. CI/CD with GitHub Actions

#### Basic CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}

jobs:
  # ========================================
  # Test Job
  # ========================================
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Generate Prisma Client
        run: npx prisma generate
        
      - name: Run Migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
      
      - name: Run Tests
        run: npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
          NEXTAUTH_SECRET: test-secret
      
      - name: Run E2E Tests
        run: npm run test:e2e
  
  # ========================================
  # Build Job
  # ========================================
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Generate Prisma Client
        run: npx prisma generate
      
      - name: Build
        run: npm run build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}
          NEXT_PUBLIC_APP_URL: ${{ secrets.NEXT_PUBLIC_APP_URL }}
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: next-build
          path: .next/
  
  # ========================================
  # Deploy Job (Production)
  # ========================================
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-args: '--prod'
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

#### Docker CI/CD

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*.*.*']

jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Step 1260: Performance Monitoring

### 8. Performance Monitoring

#### Vercel Analytics

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react'
import { SpeedInsights } from "@vercel/speed-insights/next"

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  )
}
```

#### Custom Performance Monitoring

```typescript
// lib/monitoring.ts
export function reportWebVitals(metric: {
  id: string
  name: string
  startTime: number
  value: number
  label: string
}) {
  // ส่งไปยัง Analytics Service
  const body = JSON.stringify(metric)
  
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/metrics', body)
  } else {
    fetch('/api/metrics', {
      body,
      method: 'POST',
      keepalive: true,
    })
  }
}

// app/layout.tsx (Next.js 13+)
// Web Vitals ส่งผ่าน next/vitals
```

#### Error Monitoring ด้วย Sentry

```bash
npm install @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
```

```javascript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs"

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
  debug: false,
  replaysOnErrorSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  integrations: [
    Sentry.replayIntegration(),
  ],
})
```

#### Health Check API

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'

export async function GET() {
  try {
    // Check Database
    await prisma.$queryRaw`SELECT 1`
    
    return NextResponse.json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      services: {
        database: 'ok',
      }
    })
  } catch (error) {
    return NextResponse.json(
      {
        status: 'error',
        timestamp: new Date().toISOString(),
        services: {
          database: 'error',
        },
        error: String(error),
      },
      { status: 503 }
    )
  }
}
```

---

## Step 1267: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Vercel

✓ ใช้ Vercel สำหรับ Next.js (Optimized)
✓ ตั้งค่า Environment Variables ก่อน Deploy
✓ ใช้ Preview Deployments สำหรับ Review

## 2. Environment Variables

✓ ไม่ Commit Secrets ลง Git
✓ ใช้ .env.local สำหรับ Local Dev
✓ Validate Env Vars ด้วย Zod

## 3. Docker

✓ ใช้ Multi-stage Builds
✓ ตั้งค่า output: 'standalone' ใน next.config.js
✓ ใช้ Non-root User

## 4. CI/CD

✓ Run Tests ก่อน Deploy ทุกครั้ง
✓ ใช้ Environment-specific Configs
✓ Deploy ไป Staging ก่อน Production

## 5. Monitoring

✓ ติดตั้ง Error Tracking (Sentry)
✓ ติดตาม Web Vitals
✓ ตั้งค่า Health Check Endpoint
✓ Alert เมื่อ Error Rate สูง
```

---

## Quiz

### แบบทดสอบ Part 41

**คำถามที่ 1:** Vercel Preview Deployments ใช้ทำอะไร?
- A) ทดสอบ Production
- B) สร้าง Staging Environment อัตโนมัติสำหรับทุก Branch/PR ✓
- C) Cache Static Files
- D) Backup Database

**คำถามที่ 2:** `output: 'standalone'` ใน next.config.js ทำอะไร?
- A) Export เป็น Static HTML
- B) สร้าง Optimized Bundle สำหรับ Docker ✓
- C) Enable Server-side Rendering
- D) Disable Code Splitting

**คำถามที่ 3:** NEXT_PUBLIC_ นำหน้า Environment Variable ทำให้เกิดอะไร?
- A) Variable ปลอดภัยขึ้น
- B) Variable พร้อมใช้งานใน Client-side Code ✓
- C) Variable ใช้ได้เฉพาะ Production
- D) Variable ถูก Encrypt

**คำถามที่ 4:** Docker Multi-stage Build ช่วยอะไร?
- A) เร็วขึ้น
- B) ลด Image Size โดยไม่รวม Build Tools ใน Final Image ✓
- C) Secure มากขึ้น
- D) ไม่มีประโยชน์

**คำถามที่ 5:** Health Check API ใช้ทำอะไร?
- A) ตรวจสอบ User
- B) ตรวจสอบสถานะของ Application และ Services ✓
- C) Reset Application
- D) Clear Cache

---

## สรุป Part 41

ใน Part นี้เราได้เรียนรู้:

1. **Vercel** - Platform สำหรับ Next.js
2. **Environment Variables** - การจัดการ Secrets
3. **Custom Domain** - ตั้งค่า Domain
4. **Preview Deployments** - ทดสอบก่อน Production
5. **Railway/Render** - Alternative Platforms
6. **Docker** - Container Deployment
7. **CI/CD** - GitHub Actions Automation
8. **Monitoring** - Analytics, Error Tracking

---

➡️ **Part ถัดไป:** [Part 42: Testing React](./part-42-testing-react.md)
