# Part 51: Monorepo กับ Turborepo

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1636-1680  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [Monorepo คืออะไร](#monorepo-คืออะไร)
2. [Turborepo Setup](#turborepo-setup)
3. [Shared Packages](#shared-packages)
4. [Pipeline Definitions](#pipeline-definitions)
5. [Remote Caching](#remote-caching)
6. [Apps ใน Monorepo](#apps-ใน-monorepo)
7. [pnpm Workspaces](#pnpm-workspaces)
8. [Quiz](#quiz)

---

## Step 1636: Monorepo คืออะไร {#monorepo-คืออะไร}

Monorepo คือ Repository เดียวที่เก็บหลาย projects/packages ไว้ด้วยกัน

```
Polyrepo (ก่อน):
├── repo-web-app/
├── repo-mobile-app/
├── repo-admin-dashboard/
├── repo-shared-ui/
└── repo-shared-utils/

Monorepo (ใหม่):
my-company/
├── apps/
│   ├── web/          ← Next.js web app
│   ├── mobile/       ← React Native app
│   └── admin/        ← Admin dashboard
├── packages/
│   ├── ui/           ← Shared UI components
│   ├── utils/        ← Shared utilities
│   ├── types/        ← Shared TypeScript types
│   └── config/       ← Shared configs (eslint, tsconfig)
└── package.json
```

### Monorepo Tools เปรียบเทียบ

```
Turborepo (Vercel):
├── Build caching ดีเยี่ยม
├── Simple configuration
├── Task orchestration
└── Remote caching กับ Vercel

Nx (Nrwl):
├── Powerful generators
├── Graph visualization
├── More enterprise features
└── More complex setup

Lerna + pnpm:
├── Mature ecosystem
├── Fine-grained control
└── More configuration needed

Rush:
├── Microsoft-backed
├── Enterprise-focused
└── Complex setup
```

---

## Step 1637-1645: Turborepo Setup {#turborepo-setup}

### สร้าง Monorepo ใหม่

```bash
# สร้าง Turborepo monorepo
npx create-turbo@latest my-turborepo
cd my-turborepo

# หรือเพิ่ม Turborepo ใน existing project
npx turbo init
```

### โครงสร้างไฟล์

```
my-turborepo/
├── apps/
│   ├── web/                # Next.js App
│   │   ├── app/
│   │   ├── package.json
│   │   ├── next.config.ts
│   │   └── tsconfig.json
│   ├── docs/               # Documentation Site
│   │   ├── app/
│   │   └── package.json
│   └── api/                # Express/Fastify API
│       ├── src/
│       └── package.json
├── packages/
│   ├── ui/                 # Shared React components
│   │   ├── src/
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── eslint-config/      # Shared ESLint config
│   │   ├── index.js
│   │   └── package.json
│   ├── typescript-config/  # Shared tsconfig
│   │   ├── base.json
│   │   ├── nextjs.json
│   │   ├── react-library.json
│   │   └── package.json
│   └── utils/              # Shared utilities
│       ├── src/
│       └── package.json
├── turbo.json
├── package.json
└── pnpm-workspace.yaml
```

### turbo.json Configuration

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"],
      "cache": false
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "type-check": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "clean": {
      "cache": false
    }
  }
}
```

### Package.json Root

```json
{
  "name": "my-turborepo",
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "type-check": "turbo run type-check",
    "clean": "turbo run clean && rm -rf node_modules",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "turbo": "latest",
    "prettier": "^3.0.0",
    "@my-org/eslint-config": "*"
  },
  "packageManager": "pnpm@8.0.0"
}
```

---

## Step 1646-1655: Shared Packages {#shared-packages}

### UI Package

```typescript
// packages/ui/src/button.tsx
import * as React from 'react'
import { cn } from '@my-org/utils'

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'primary' | 'secondary' | 'outline' | 'ghost' | 'link' | 'destructive'
  size?: 'default' | 'sm' | 'lg' | 'icon'
}

const buttonVariants = {
  default: 'bg-primary text-primary-foreground hover:bg-primary/90',
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
  outline: 'border border-input bg-background hover:bg-accent',
  ghost: 'hover:bg-accent hover:text-accent-foreground',
  link: 'text-primary underline-offset-4 hover:underline',
  destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
}

const sizeVariants = {
  default: 'h-10 px-4 py-2',
  sm: 'h-9 rounded-md px-3',
  lg: 'h-11 rounded-md px-8',
  icon: 'h-10 w-10',
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'default', size = 'default', ...props }, ref) => {
    return (
      <button
        className={cn(
          'inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
          buttonVariants[variant],
          sizeVariants[size],
          className
        )}
        ref={ref}
        {...props}
      />
    )
  }
)

Button.displayName = 'Button'

export { Button }
```

```json
// packages/ui/package.json
{
  "name": "@my-org/ui",
  "version": "0.0.1",
  "private": true,
  "main": "./src/index.tsx",
  "types": "./src/index.tsx",
  "exports": {
    ".": "./src/index.tsx"
  },
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0"
  },
  "devDependencies": {
    "@my-org/eslint-config": "*",
    "@my-org/typescript-config": "*",
    "@types/react": "^18.2.0",
    "typescript": "^5.0.0"
  },
  "peerDependencies": {
    "react": "^18.2.0"
  }
}
```

### Utils Package

```typescript
// packages/utils/src/index.ts
export { cn } from './cn'
export { formatDate } from './formatDate'
export { formatCurrency } from './formatCurrency'
export { debounce } from './debounce'
export { throttle } from './throttle'
export * from './validators'

// packages/utils/src/cn.ts
import { type ClassValue, clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// packages/utils/src/formatCurrency.ts
export function formatCurrency(
  amount: number,
  currency: string = 'THB',
  locale: string = 'th-TH'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency,
    minimumFractionDigits: 0,
    maximumFractionDigits: 2,
  }).format(amount)
}

// packages/utils/src/validators.ts
export const validators = {
  email: (value: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  
  phone: (value: string) => /^(0[689]\d{8}|[+66][689]\d{8})$/.test(value),
  
  thaiId: (value: string) => {
    if (value.length !== 13) return false
    let sum = 0
    for (let i = 0; i < 12; i++) {
      sum += parseInt(value[i]) * (13 - i)
    }
    const checkDigit = (11 - (sum % 11)) % 10
    return checkDigit === parseInt(value[12])
  },
  
  url: (value: string) => {
    try {
      new URL(value)
      return true
    } catch {
      return false
    }
  },
  
  minLength: (min: number) => (value: string) => value.length >= min,
  
  maxLength: (max: number) => (value: string) => value.length <= max,
  
  required: (value: any) => value !== null && value !== undefined && value !== '',
}
```

### Shared TypeScript Config

```json
// packages/typescript-config/base.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Default",
  "compilerOptions": {
    "composite": false,
    "declaration": true,
    "declarationMap": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "moduleResolution": "node",
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true
  },
  "exclude": ["node_modules"]
}

// packages/typescript-config/nextjs.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Next.js",
  "extends": "./base.json",
  "compilerOptions": {
    "plugins": [{ "name": "next" }],
    "module": "esnext",
    "moduleResolution": "bundler",
    "allowJs": true,
    "jsx": "preserve",
    "incremental": true,
    "target": "ES2017"
  },
  "include": ["src", "next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Shared ESLint Config

```javascript
// packages/eslint-config/index.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint'],
  parser: '@typescript-eslint/parser',
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/no-explicit-any': 'warn',
    'no-console': ['warn', { allow: ['warn', 'error'] }],
  },
}

// packages/eslint-config/next.js
module.exports = {
  extends: ['./index.js', 'next/core-web-vitals'],
  rules: {
    '@next/next/no-html-link-for-pages': 'off',
  },
}
```

---

## Step 1656-1665: Pipeline Definitions {#pipeline-definitions}

### Advanced Pipeline Configuration

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [
        ".next/**",
        "!.next/cache/**",
        "dist/**",
        "build/**"
      ],
      "env": [
        "NODE_ENV",
        "NEXT_PUBLIC_API_URL",
        "DATABASE_URL"
      ]
    },
    
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**", "test-results/**"],
      "env": ["DATABASE_URL", "TEST_API_KEY"]
    },
    
    "test:unit": {
      "outputs": ["coverage/**"]
    },
    
    "test:e2e": {
      "dependsOn": ["build"],
      "outputs": ["test-results/**"],
      "cache": false
    },
    
    "lint": {
      "outputs": ["reports/**"]
    },
    
    "dev": {
      "cache": false,
      "persistent": true,
      "env": [
        "PORT",
        "NODE_ENV",
        "NEXT_PUBLIC_API_URL"
      ]
    },
    
    "deploy": {
      "dependsOn": ["build", "test", "lint"],
      "outputs": [],
      "cache": false
    },
    
    "generate": {
      "outputs": ["src/generated/**"],
      "cache": false
    },
    
    "db:migrate": {
      "cache": false
    },
    
    "storybook": {
      "cache": false,
      "persistent": true
    },
    
    "build-storybook": {
      "outputs": ["storybook-static/**"]
    }
  }
}
```

### Running Tasks

```bash
# Run ทุก apps
turbo run build

# Run เฉพาะ app
turbo run build --filter=web

# Run หลาย apps
turbo run build --filter=web --filter=docs

# Run ทุก apps ยกเว้น
turbo run build --filter=!api

# Run โดยอ้างอิง package
turbo run build --filter=@my-org/ui...

# Dry run (ไม่ execute จริง)
turbo run build --dry

# แสดง graph
turbo run build --graph

# Force rebuild (ข้าม cache)
turbo run build --force

# Concurrency
turbo run build --concurrency=5

# Parallel สูงสุด
turbo run dev --parallel
```

### Custom Workspace Scripts

```json
// package.json (root)
{
  "scripts": {
    "build": "turbo run build",
    "build:apps": "turbo run build --filter=./apps/*",
    "build:packages": "turbo run build --filter=./packages/*",
    "dev": "turbo run dev",
    "dev:web": "turbo run dev --filter=web",
    "test": "turbo run test",
    "test:unit": "turbo run test:unit",
    "test:e2e": "turbo run test:e2e",
    "lint": "turbo run lint",
    "lint:fix": "turbo run lint -- --fix",
    "format": "prettier --write \"**/*.{ts,tsx,md,json}\"",
    "type-check": "turbo run type-check",
    "clean": "turbo run clean",
    "clean:all": "turbo run clean && rimraf node_modules",
    "generate": "turbo run generate",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "turbo run build --filter=./packages/* && changeset publish"
  }
}
```

---

## Step 1666-1673: Remote Caching {#remote-caching}

Remote Caching ช่วยให้ CI/CD และสมาชิกในทีม share cache กัน

### Setup Vercel Remote Cache

```bash
# Login ไปยัง Vercel
npx turbo login

# Link กับ Vercel team
npx turbo link
```

### turbo.json สำหรับ Remote Cache

```json
{
  "$schema": "https://turbo.build/schema.json",
  "remoteCache": {
    "enabled": true,
    "apiUrl": "https://vercel.com"
  },
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**"]
    }
  }
}
```

### Custom Remote Cache (Self-hosted)

```bash
# Setup ด้วย Docker
docker run -it --env-file .env ducktors/turborepo-remote-cache
```

```yaml
# .env สำหรับ self-hosted cache
PORT=3000
TURBO_TOKEN=your-secret-token
STORAGE_PROVIDER=local  # local, s3, gcs, azure-blob
STORAGE_PATH=./cache
```

```json
// turbo.json
{
  "remoteCache": {
    "enabled": true,
    "apiUrl": "http://localhost:3000"
  }
}
```

### GitHub Actions กับ Remote Cache

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      
      - name: Build and Test
        run: pnpm run build
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ vars.TURBO_TEAM }}
```

---

## Step 1674-1678: Apps ใน Monorepo {#apps-ใน-monorepo}

### Next.js App ใน Monorepo

```typescript
// apps/web/next.config.ts
import type { NextConfig } from 'next'

const config: NextConfig = {
  transpilePackages: ['@my-org/ui', '@my-org/utils'],
  
  // ถ้าใช้ Turborepo environment variables
  env: {
    NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  },
  
  experimental: {
    typedRoutes: true,
  },
}

export default config
```

```json
// apps/web/package.json
{
  "name": "web",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "clean": "rm -rf .next"
  },
  "dependencies": {
    "@my-org/ui": "*",
    "@my-org/utils": "*",
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@my-org/eslint-config": "*",
    "@my-org/typescript-config": "*",
    "@types/node": "^20.0.0",
    "@types/react": "^18.2.0",
    "eslint": "^8.0.0",
    "typescript": "^5.0.0"
  }
}
```

---

## Step 1679-1680: pnpm Workspaces {#pnpm-workspaces}

### Setup pnpm Workspaces

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### pnpm Commands

```bash
# Install dependencies ทั้ง workspace
pnpm install

# เพิ่ม dependency ใน specific package
pnpm add react --filter web
pnpm add -D typescript --filter @my-org/ui

# เพิ่ม local package เป็น dependency
pnpm add @my-org/ui --filter web --workspace

# Run script ใน specific package
pnpm --filter web run build
pnpm --filter @my-org/ui run test

# Run script ทุก packages
pnpm --recursive run build
pnpm -r run lint

# Run ใน parallel
pnpm -r --parallel run test

# Update dependency ทุก packages
pnpm update react --recursive

# List packages
pnpm ls --recursive --depth 1
```

### Linking Local Packages

```json
// apps/web/package.json
{
  "dependencies": {
    "@my-org/ui": "workspace:*",
    "@my-org/utils": "workspace:*",
    "@my-org/types": "workspace:^1.0.0"
  }
}
```

### Versioning กับ Changesets

```bash
# ติดตั้ง changesets
pnpm add -D @changesets/cli -w

# Initialize
npx changeset init

# สร้าง changeset ใหม่
npx changeset add

# Version packages
npx changeset version

# Publish packages
npx changeset publish
```

```markdown
# .changeset/my-change.md
---
"@my-org/ui": minor
"@my-org/utils": patch
---

Added new Button variants and fixed utility function bugs
```

---

## 🧪 Quiz - Part 51

**ข้อ 1:** Turborepo ต่างจาก npm workspaces อย่างไร?
- A) รองรับ TypeScript มากกว่า
- B) เพิ่ม task orchestration, caching และ Remote caching
- C) Deploy ได้เร็วกว่า
- D) รองรับ Docker ดีกว่า

**ข้อ 2:** `dependsOn: ["^build"]` ใน turbo.json หมายความว่า?
- A) Build หลังจาก test
- B) Build หลังจาก dependencies ทุกตัว build เสร็จ
- C) Build ก่อน dependencies
- D) Build พร้อมกัน

**ข้อ 3:** Remote Caching ช่วยอะไร?
- A) Deploy code ไปยัง server
- B) Cache build artifacts เพื่อให้ CI/CD และทีม share กัน
- C) Cache database queries
- D) Cache API responses

**ข้อ 4:** `workspace:*` ใน pnpm หมายความว่า?
- A) Install จาก npm registry
- B) Install จาก local workspace package
- C) Install เวอร์ชันล่าสุด
- D) Install เวอร์ชัน stable

**เฉลย:** 1-B, 2-B, 3-B, 4-B

---

> **➡️ Next:** [Part 52: GraphQL กับ Apollo Client](./part-52-graphql-apollo.md)
