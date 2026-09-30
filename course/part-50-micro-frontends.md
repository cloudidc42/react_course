# Part 50: Micro-frontends

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1596-1635  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [Micro-frontends คืออะไร](#micro-frontends-คืออะไร)
2. [Module Federation (Webpack 5)](#module-federation)
3. [Nx Monorepo](#nx-monorepo)
4. [Shared Libraries](#shared-libraries)
5. [Independent Deployments](#independent-deployments)
6. [Communication between Micro-frontends](#communication)
7. [ข้อดีข้อเสีย](#ข้อดีข้อเสีย)
8. [Quiz](#quiz)

---

## Step 1596: Micro-frontends คืออะไร {#micro-frontends-คืออะไร}

Micro-frontends คือ Architecture pattern ที่แบ่ง Frontend Application ใหญ่ออกเป็น Application ย่อยๆ อิสระ ซึ่งแต่ละทีมพัฒนาและ Deploy ได้เอง

```
Monolithic Frontend (ก่อน):
┌─────────────────────────────────────────┐
│                                         │
│   React App (ทีมเดียว ดูแลทุกอย่าง)     │
│                                         │
│   Header | Product | Cart | Checkout    │
│   Profile | Orders | Search | Reviews   │
│                                         │
└─────────────────────────────────────────┘

Micro-frontends (ใหม่):
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Shell App   │  │  Product MFE │  │   Cart MFE   │
│  (Routing)   │  │  (ทีม B)     │  │  (ทีม C)     │
└──────────────┘  └──────────────┘  └──────────────┘
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Checkout MFE │  │  Profile MFE │  │  Search MFE  │
│  (ทีม D)     │  │  (ทีม E)     │  │  (ทีม F)     │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Integration Approaches

```
1. Build-time Integration:
   - npm packages
   - ง่ายที่สุด แต่ต้อง rebuild ทุกครั้ง

2. Run-time Integration via iframes:
   - Isolation สูงสุด
   - UX ไม่ดี, Performance แย่

3. Run-time Integration via JavaScript:
   - Module Federation (Webpack 5)
   - Single-SPA
   - ยืดหยุ่นสูง

4. Edge-side / Server-side Composition:
   - Next.js App Shell
   - Nginx composition
   - ดีสำหรับ SEO
```

---

## Step 1600-1615: Module Federation (Webpack 5) {#module-federation}

Module Federation ช่วยให้ application แต่ละตัว share code กันได้ใน runtime โดยไม่ต้อง rebuild

### Architecture

```
Shell App (Host)
├── โหลด micro-frontends แบบ lazy
├── จัดการ routing หลัก
└── ให้ shared dependencies

Product MFE (Remote)
├── export ProductList, ProductDetail
└── ใช้ dependencies จาก Shell

Cart MFE (Remote)
├── export CartButton, CartSidebar
└── ใช้ dependencies จาก Shell
```

### Setup webpack.config.js

```javascript
// shell-app/webpack.config.js (Host)
const { ModuleFederationPlugin } = require('webpack').container
const deps = require('./package.json').dependencies

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      
      remotes: {
        // ชื่อ: URL ที่ expose remoteEntry.js
        productApp: 'product@http://localhost:3001/remoteEntry.js',
        cartApp: 'cart@http://localhost:3002/remoteEntry.js',
        checkoutApp: 'checkout@http://localhost:3003/remoteEntry.js',
      },
      
      shared: {
        // Share React เพื่อไม่ให้โหลดซ้ำ
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
          eager: true,
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: deps['react-router-dom'],
        },
      },
    }),
  ],
}

// product-app/webpack.config.js (Remote)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'product',
      
      filename: 'remoteEntry.js', // Entry point ที่ expose
      
      exposes: {
        // ชื่อ: path ของ module ที่ expose
        './ProductList': './src/components/ProductList',
        './ProductDetail': './src/components/ProductDetail',
        './ProductCard': './src/components/ProductCard',
        './useProducts': './src/hooks/useProducts',
      },
      
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
}
```

### Shell App - การใช้งาน

```typescript
// shell-app/src/App.tsx
import React, { Suspense, lazy } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

// Lazy load Micro-frontends
const ProductList = lazy(() => import('productApp/ProductList'))
const ProductDetail = lazy(() => import('productApp/ProductDetail'))
const CartSidebar = lazy(() => import('cartApp/CartSidebar'))
const CheckoutFlow = lazy(() => import('checkoutApp/CheckoutFlow'))

function LoadingFallback() {
  return (
    <div className="flex items-center justify-center h-48">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
    </div>
  )
}

function ErrorBoundary({ fallback, children }: {
  fallback: React.ReactNode
  children: React.ReactNode
}) {
  // Error boundary implementation
  return <>{children}</>
}

export default function App() {
  return (
    <BrowserRouter>
      <div className="min-h-screen">
        <header className="bg-white shadow-sm">
          <nav className="container mx-auto px-4 flex items-center justify-between h-16">
            <a href="/" className="text-xl font-bold">MyStore</a>
            <Suspense fallback={<div>Loading cart...</div>}>
              <CartSidebar />
            </Suspense>
          </nav>
        </header>
        
        <main className="container mx-auto px-4 py-8">
          <Routes>
            <Route
              path="/products"
              element={
                <ErrorBoundary fallback={<div>Products unavailable</div>}>
                  <Suspense fallback={<LoadingFallback />}>
                    <ProductList />
                  </Suspense>
                </ErrorBoundary>
              }
            />
            <Route
              path="/products/:id"
              element={
                <ErrorBoundary fallback={<div>Product unavailable</div>}>
                  <Suspense fallback={<LoadingFallback />}>
                    <ProductDetail />
                  </Suspense>
                </ErrorBoundary>
              }
            />
            <Route
              path="/checkout"
              element={
                <ErrorBoundary fallback={<div>Checkout unavailable</div>}>
                  <Suspense fallback={<LoadingFallback />}>
                    <CheckoutFlow />
                  </Suspense>
                </ErrorBoundary>
              }
            />
          </Routes>
        </main>
      </div>
    </BrowserRouter>
  )
}
```

### TypeScript Types สำหรับ Module Federation

```typescript
// types/remotes.d.ts
declare module 'productApp/ProductList' {
  const ProductList: React.ComponentType<{
    category?: string
    limit?: number
  }>
  export default ProductList
}

declare module 'productApp/ProductDetail' {
  const ProductDetail: React.ComponentType<{
    productId: string
  }>
  export default ProductDetail
}

declare module 'cartApp/CartSidebar' {
  const CartSidebar: React.ComponentType
  export default CartSidebar
}

declare module 'checkoutApp/CheckoutFlow' {
  const CheckoutFlow: React.ComponentType
  export default CheckoutFlow
}
```

### Module Federation กับ Next.js

```javascript
// next.config.js ของ Shell App
const { NextFederationPlugin } = require('@module-federation/nextjs-mf')

module.exports = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'shell',
        filename: 'static/chunks/remoteEntry.js',
        remotes: {
          product: `product@http://localhost:3001/_next/static/chunks/remoteEntry.js`,
          cart: `cart@http://localhost:3002/_next/static/chunks/remoteEntry.js`,
        },
        shared: {},
        extraOptions: {
          automaticAsyncBoundary: true,
        },
      })
    )
    return config
  },
}
```

---

## Step 1616: Nx Monorepo {#nx-monorepo}

Nx เป็น build system ที่ช่วยจัดการ Monorepo สำหรับ Micro-frontends

### Setup Nx Workspace

```bash
# สร้าง Nx Workspace ใหม่
npx create-nx-workspace@latest my-micro-frontends \
  --preset=react \
  --nxCloud=false

# เพิ่ม Next.js applications
nx generate @nx/next:application shell --directory=apps/shell
nx generate @nx/next:application product --directory=apps/product
nx generate @nx/next:application cart --directory=apps/cart

# เพิ่ม Shared Libraries
nx generate @nx/react:library ui --directory=libs/ui
nx generate @nx/react:library store --directory=libs/store
nx generate @nx/react:library utils --directory=libs/utils
```

### โครงสร้าง Nx Workspace

```
my-micro-frontends/
├── apps/
│   ├── shell/              # Shell Application
│   ├── shell-e2e/          # E2E Tests
│   ├── product/            # Product Micro-frontend
│   ├── product-e2e/
│   ├── cart/               # Cart Micro-frontend
│   └── cart-e2e/
├── libs/
│   ├── ui/                 # Shared UI Components
│   │   ├── src/
│   │   │   ├── lib/
│   │   │   │   ├── Button/
│   │   │   │   ├── Card/
│   │   │   │   └── Modal/
│   │   │   └── index.ts
│   │   └── project.json
│   ├── store/              # Shared State (Zustand/Redux)
│   ├── utils/              # Shared Utilities
│   └── api-types/          # Shared TypeScript Types
├── nx.json
├── workspace.json
└── package.json
```

### nx.json Configuration

```json
{
  "tasksRunnerOptions": {
    "default": {
      "runner": "nx/tasks-runners/default",
      "options": {
        "cacheableOperations": ["build", "lint", "test", "e2e"]
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"]
    },
    "lint": {
      "inputs": ["default", "{workspaceRoot}/.eslintrc.json"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json"
    ]
  }
}
```

---

## Step 1620: Shared Libraries {#shared-libraries}

```typescript
// libs/ui/src/lib/Button/Button.tsx
import { forwardRef, ButtonHTMLAttributes } from 'react'

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  loading?: boolean
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', loading, children, ...props }, ref) => {
    const baseClasses = 'inline-flex items-center justify-center font-medium rounded-lg transition-colors'
    
    const variantClasses = {
      primary: 'bg-blue-600 text-white hover:bg-blue-700',
      secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200',
      danger: 'bg-red-600 text-white hover:bg-red-700',
    }
    
    const sizeClasses = {
      sm: 'px-3 py-1.5 text-sm',
      md: 'px-4 py-2 text-base',
      lg: 'px-6 py-3 text-lg',
    }
    
    return (
      <button
        ref={ref}
        className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`}
        disabled={loading || props.disabled}
        {...props}
      >
        {loading && <span className="mr-2 animate-spin">⟳</span>}
        {children}
      </button>
    )
  }
)

Button.displayName = 'Button'

// libs/ui/src/index.ts
export * from './lib/Button/Button'
export * from './lib/Card/Card'
export * from './lib/Modal/Modal'
export * from './lib/Input/Input'

// การใช้งานใน apps
// apps/product/src/components/ProductCard.tsx
import { Button, Card } from '@my-org/ui'

export function ProductCard({ product }) {
  return (
    <Card>
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
      <Button variant="primary">เพิ่มในตะกร้า</Button>
    </Card>
  )
}
```

### Shared API Types

```typescript
// libs/api-types/src/index.ts
export interface Product {
  id: string
  name: string
  description: string
  price: number
  images: string[]
  category: string
  inStock: boolean
  rating: number
  reviewCount: number
}

export interface CartItem {
  product: Product
  quantity: number
}

export interface Cart {
  items: CartItem[]
  total: number
  itemCount: number
}

export interface User {
  id: string
  name: string
  email: string
  avatar?: string
  role: 'customer' | 'admin'
}

export interface Order {
  id: string
  items: CartItem[]
  total: number
  status: 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled'
  createdAt: string
  shippingAddress: Address
}

export interface Address {
  name: string
  street: string
  city: string
  province: string
  postalCode: string
  phone: string
}
```

---

## Step 1625: Independent Deployments {#independent-deployments}

### Docker สำหรับแต่ละ MFE

```dockerfile
# apps/product/Dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx nx build product --prod

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/apps/product/.next/standalone ./
COPY --from=builder /app/apps/product/.next/static ./apps/product/.next/static
COPY --from=builder /app/apps/product/public ./apps/product/public

EXPOSE 3001
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  shell:
    build: ./apps/shell
    ports:
      - "3000:3000"
    environment:
      - PRODUCT_MFE_URL=http://product:3001
      - CART_MFE_URL=http://cart:3002
  
  product:
    build: ./apps/product
    ports:
      - "3001:3001"
  
  cart:
    build: ./apps/cart
    ports:
      - "3002:3002"
  
  checkout:
    build: ./apps/checkout
    ports:
      - "3003:3003"
```

### Vercel Deployment Configuration

```json
// vercel.json สำหรับ Monorepo
{
  "version": 2,
  "projects": {
    "shell": {
      "directory": "apps/shell",
      "framework": "nextjs"
    },
    "product": {
      "directory": "apps/product",
      "framework": "nextjs"
    },
    "cart": {
      "directory": "apps/cart",
      "framework": "nextjs"
    }
  },
  "rewrites": [
    {
      "source": "/products/:path*",
      "destination": "https://product.vercel.app/products/:path*"
    },
    {
      "source": "/cart/:path*",
      "destination": "https://cart.vercel.app/cart/:path*"
    }
  ]
}
```

---

## Step 1628-1632: Communication between Micro-frontends {#communication}

### 1. Custom Events

```typescript
// libs/events/src/index.ts
export const MFEEvents = {
  CART_ITEM_ADDED: 'mfe:cart:item-added',
  CART_ITEM_REMOVED: 'mfe:cart:item-removed',
  USER_LOGGED_IN: 'mfe:user:logged-in',
  USER_LOGGED_OUT: 'mfe:user:logged-out',
  NAVIGATION: 'mfe:navigation',
} as const

export function dispatchMFEEvent(event: string, detail: any) {
  window.dispatchEvent(new CustomEvent(event, { detail }))
}

export function subscribeMFEEvent(
  event: string,
  handler: (detail: any) => void
): () => void {
  const wrappedHandler = (e: Event) => handler((e as CustomEvent).detail)
  window.addEventListener(event, wrappedHandler)
  return () => window.removeEventListener(event, wrappedHandler)
}

// Product MFE - ส่ง event
function ProductCard({ product }) {
  function addToCart() {
    dispatchMFEEvent(MFEEvents.CART_ITEM_ADDED, {
      productId: product.id,
      name: product.name,
      price: product.price,
    })
  }
  
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={addToCart}>เพิ่มในตะกร้า</button>
    </div>
  )
}

// Cart MFE - รับ event
function CartBadge() {
  const [count, setCount] = useState(0)
  
  useEffect(() => {
    return subscribeMFEEvent(MFEEvents.CART_ITEM_ADDED, (detail) => {
      setCount((prev) => prev + 1)
    })
  }, [])
  
  return <span className="badge">{count}</span>
}
```

### 2. Shared State Store

```typescript
// libs/store/src/cartStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import type { CartItem, Product } from '@my-org/api-types'

interface CartStore {
  items: CartItem[]
  total: number
  itemCount: number
  
  addItem: (product: Product, quantity?: number) => void
  removeItem: (productId: string) => void
  updateQuantity: (productId: string, quantity: number) => void
  clearCart: () => void
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      total: 0,
      itemCount: 0,
      
      addItem: (product, quantity = 1) => {
        const { items } = get()
        const existing = items.find((i) => i.product.id === product.id)
        
        let newItems: CartItem[]
        if (existing) {
          newItems = items.map((i) =>
            i.product.id === product.id
              ? { ...i, quantity: i.quantity + quantity }
              : i
          )
        } else {
          newItems = [...items, { product, quantity }]
        }
        
        const total = newItems.reduce((sum, i) => sum + i.product.price * i.quantity, 0)
        const itemCount = newItems.reduce((sum, i) => sum + i.quantity, 0)
        
        set({ items: newItems, total, itemCount })
      },
      
      removeItem: (productId) => {
        const items = get().items.filter((i) => i.product.id !== productId)
        const total = items.reduce((sum, i) => sum + i.product.price * i.quantity, 0)
        const itemCount = items.reduce((sum, i) => sum + i.quantity, 0)
        set({ items, total, itemCount })
      },
      
      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeItem(productId)
          return
        }
        const items = get().items.map((i) =>
          i.product.id === productId ? { ...i, quantity } : i
        )
        const total = items.reduce((sum, i) => sum + i.product.price * i.quantity, 0)
        const itemCount = items.reduce((sum, i) => sum + i.quantity, 0)
        set({ items, total, itemCount })
      },
      
      clearCart: () => set({ items: [], total: 0, itemCount: 0 }),
    }),
    {
      name: 'cart-storage',
      partialize: (state) => ({ items: state.items }),
    }
  )
)
```

---

## Step 1633-1635: ข้อดีข้อเสีย {#ข้อดีข้อเสีย}

### ข้อดี

```
✅ Independent Development:
   - ทีมต่างๆ พัฒนาได้พร้อมกัน
   - ไม่ต้องรอกัน
   - ลด merge conflicts

✅ Independent Deployment:
   - Deploy แต่ละส่วนแยกกันได้
   - ลด risk ในการ deploy
   - Rollback ง่าย

✅ Technology Flexibility:
   - แต่ละทีมเลือก tech stack ได้
   - Upgrade dependencies ได้ทีละส่วน

✅ Scalability:
   - Scale ได้ตาม team size
   - เหมาะกับ large organization

✅ Fault Isolation:
   - ถ้า MFE หนึ่งพัง ส่วนอื่นยังทำงานได้
```

### ข้อเสีย

```
❌ Complexity:
   - Setup ซับซ้อนกว่า Monolith มาก
   - ต้องการ DevOps expertise สูง
   - Communication overhead ระหว่างทีม

❌ Performance:
   - โหลด JavaScript หลาย bundle
   - Shared dependencies อาจ conflict
   - Initial load time อาจนานขึ้น

❌ Consistency:
   - UI ต่างๆ อาจไม่สม่ำเสมอ
   - ต้องการ Design System ที่แข็งแกร่ง
   - Shared state ซับซ้อน

❌ Testing:
   - Integration testing ยากขึ้น
   - ต้องการ E2E testing ที่ครอบคลุม

❌ Overhead:
   - ต้องการ DevOps infrastructure มากขึ้น
   - Infrastructure cost สูงขึ้น
   - ไม่คุ้มสำหรับทีมเล็ก
```

### เมื่อไรควรใช้ Micro-frontends

```
ควรใช้เมื่อ:
✅ ทีม frontend มากกว่า 3-4 ทีม
✅ Application ใหญ่และซับซ้อน
✅ ต้องการ Independent deployment
✅ มี bounded contexts ชัดเจน
✅ มี DevOps team ที่แข็งแกร่ง

ไม่ควรใช้เมื่อ:
❌ ทีมขนาดเล็ก (< 5 คน)
❌ Application ขนาดกลาง-เล็ก
❌ Startup ที่ต้องการ velocity สูง
❌ ไม่มี DevOps expertise
❌ ยังไม่ชัดเจนเรื่อง domain boundaries
```

---

## 🧪 Quiz - Part 50

**ข้อ 1:** Module Federation ใช้กับ bundler อะไร?
- A) Vite
- B) Rollup
- C) Webpack 5
- D) esbuild

**ข้อ 2:** ปัญหาหลักของ Micro-frontends คืออะไร?
- A) ไม่รองรับ TypeScript
- B) Complexity ที่เพิ่มขึ้นและ Performance overhead
- C) ไม่สามารถ share state ได้
- D) ต้องใช้ backend เดียวกัน

**ข้อ 3:** Communication ระหว่าง Micro-frontends ที่ดีที่สุดคือ?
- A) Direct function calls
- B) Shared database
- C) Custom Events หรือ Shared Store
- D) REST API calls

**ข้อ 4:** Nx Workspace ช่วยอะไรใน Micro-frontends?
- A) Deploy อัตโนมัติ
- B) จัดการ Monorepo, build optimization, shared libraries
- C) ทำ Load balancing
- D) จัดการ Database

**เฉลย:** 1-C, 2-B, 3-C, 4-B

---

> **➡️ Next:** [Part 51: Monorepo with Turborepo](./part-51-monorepo-turborepo.md)
