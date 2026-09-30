# Part 37: Next.js Client Components

## ข้อมูล Part
- **Steps:** 1076-1110
- **ระดับ:** Intermediate
- **เวลาเรียน:** 3 ชั่วโมง
- **Prerequisites:** Part 36 (Server Components)

---

## สารบัญ

1. ['use client' Directive](#1-use-client-directive)
2. [เมื่อไหรใช้ Client Components](#2-เมื่อไหรใช้-client-components)
3. [Interactivity ใน Client Components](#3-interactivity-ใน-client-components)
4. [Client Side State](#4-client-side-state)
5. [Mixing Server และ Client Components](#5-mixing-server-และ-client-components)
6. [Data Passing Patterns](#6-data-passing-patterns)
7. [Quiz](#quiz)

---

## Step 1076: 'use client' Directive

### 1. 'use client' Directive

`'use client'` บอก Next.js ว่า Component นี้และ Components ที่ import มาทำงานที่ Client

#### การใช้งาน

```tsx
'use client'  // ต้องอยู่บนสุดของไฟล์

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  )
}
```

#### Client Boundary

```
'use client' สร้าง Client Boundary:
  
  Server Component (default)
  └── Server Component
      └── 'use client' ← Client Boundary เริ่มที่นี่
          └── Client Component
              └── Client Component (ไม่ต้องใส่ 'use client' อีก)
```

```tsx
// components/ParentServer.tsx (Server Component)
import ChildServer from './ChildServer'
import ClientChild from './ClientChild'

export default function ParentServer() {
  return (
    <div>
      <ChildServer />      {/* ยังเป็น Server Component */}
      <ClientChild />      {/* Client Component */}
    </div>
  )
}

// components/ClientChild.tsx
'use client'  // ← เริ่ม Client Boundary

import ClientGrandchild from './ClientGrandchild'

export default function ClientChild() {
  return (
    <div>
      <ClientGrandchild />  {/* ไม่ต้องใส่ 'use client' */}
    </div>
  )
}
```

#### ไม่ต้อง 'use client' ทุก Component

```tsx
// ❌ ไม่จำเป็น - Component นี้ใช้ใน Client Boundary แล้ว
'use client'  // ไม่จำเป็นถ้า Parent มี 'use client' แล้ว
import React from 'react'

function GrandchildComponent() {
  return <div>Grandchild</div>
}

// ✅ ถูกต้อง - ไม่ต้องใส่ 'use client' ซ้ำ
import React from 'react'

function GrandchildComponent() {
  return <div>Grandchild</div>
}
```

---

## Step 1079: เมื่อไหรใช้ Client Components

### 2. เมื่อไหรใช้ Client Components

#### ใช้ Client Component เมื่อ:

```
1. ต้องการ State (useState, useReducer)
2. ต้องการ Effects (useEffect, useLayoutEffect)
3. ต้องการ Event Handlers (onClick, onChange, etc.)
4. ต้องการ Browser APIs (localStorage, window, document)
5. ต้องการ Custom Hooks ที่ใช้ 1-4 ข้างต้น
6. ต้องการ React Context ที่มี State
7. ต้องการ Third-party Libraries ที่ใช้ Browser
```

#### ใช้ Server Component เมื่อ:

```
1. Fetch Data จาก API หรือ Database
2. Access Sensitive Data (API Keys, Auth)
3. ไม่ต้องการ Interactivity
4. ต้องการ Heavy Libraries (Markdown, PDF, etc.)
5. SEO Critical Content
```

#### การตัดสินใจ

```tsx
// ❓ ต้องการ State หรือไม่?
// YES → Client Component
// NO  → Server Component

// ❓ ต้องการ Events หรือไม่?
// YES → Client Component
// NO  → Server Component

// ❓ ต้องการ Browser APIs หรือไม่?
// YES → Client Component
// NO  → Server Component

// ❓ ต้องการ Fetch Data หรือไม่?
// YES → Server Component (preferred)
// YES + Interactivity → ส่ง Data จาก Server ไป Client
```

---

## Step 1082: Interactivity ใน Client Components

### 3. Interactivity ใน Client Components

#### Event Handlers

```tsx
'use client'

export default function InteractiveForm() {
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    const formData = new FormData(e.target as HTMLFormElement)
    // Process form
  }
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        onChange={handleChange}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  )
}
```

#### Browser APIs

```tsx
'use client'
import { useState, useEffect } from 'react'

export function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 })
  
  useEffect(() => {
    function updateSize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      })
    }
    
    updateSize()
    window.addEventListener('resize', updateSize)
    return () => window.removeEventListener('resize', updateSize)
  }, [])
  
  return (
    <div>
      Window: {size.width} x {size.height}
    </div>
  )
}
```

#### localStorage

```tsx
'use client'
import { useState, useEffect } from 'react'

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue
    
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value
      setStoredValue(valueToStore)
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
      }
    } catch (error) {
      console.error(error)
    }
  }
  
  return [storedValue, setValue] as const
}

export default function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light')
  
  return (
    <button
      onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}
      className={theme === 'dark' ? 'bg-gray-800 text-white' : 'bg-white text-gray-800'}
    >
      {theme === 'light' ? '🌙 Dark' : '☀️ Light'}
    </button>
  )
}
```

---

## Step 1085: Client Side State

### 4. Client Side State

#### useState

```tsx
'use client'
import { useState } from 'react'

interface CartItem {
  id: string
  name: string
  price: number
  quantity: number
}

export default function ShoppingCart() {
  const [items, setItems] = useState<CartItem[]>([])
  
  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    setItems(prev => {
      const existing = prev.find(i => i.id === item.id)
      if (existing) {
        return prev.map(i =>
          i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
        )
      }
      return [...prev, { ...item, quantity: 1 }]
    })
  }
  
  const removeItem = (id: string) => {
    setItems(prev => prev.filter(i => i.id !== id))
  }
  
  const updateQuantity = (id: string, quantity: number) => {
    if (quantity <= 0) {
      removeItem(id)
      return
    }
    setItems(prev =>
      prev.map(i => i.id === id ? { ...i, quantity } : i)
    )
  }
  
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  
  return (
    <div>
      <h2>Cart ({items.length} items)</h2>
      {items.map(item => (
        <div key={item.id} className="flex items-center gap-4">
          <span>{item.name}</span>
          <input
            type="number"
            value={item.quantity}
            onChange={e => updateQuantity(item.id, Number(e.target.value))}
            min={1}
            className="w-16 border px-2 py-1 rounded"
          />
          <span>{(item.price * item.quantity).toLocaleString()} บาท</span>
          <button onClick={() => removeItem(item.id)}>ลบ</button>
        </div>
      ))}
      <div className="font-bold text-xl mt-4">
        ยอดรวม: {total.toLocaleString()} บาท
      </div>
    </div>
  )
}
```

#### useReducer สำหรับ Complex State

```tsx
'use client'
import { useReducer } from 'react'

interface State {
  items: CartItem[]
  isOpen: boolean
  coupon: string | null
  discount: number
}

type Action =
  | { type: 'ADD_ITEM'; payload: CartItem }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' }
  | { type: 'TOGGLE_CART' }
  | { type: 'APPLY_COUPON'; payload: { code: string; discount: number } }

function cartReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find(i => i.id === action.payload.id)
      return {
        ...state,
        items: existing
          ? state.items.map(i =>
              i.id === action.payload.id
                ? { ...i, quantity: i.quantity + 1 }
                : i
            )
          : [...state.items, action.payload]
      }
    }
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload)
      }
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: action.payload.quantity <= 0
          ? state.items.filter(i => i.id !== action.payload.id)
          : state.items.map(i =>
              i.id === action.payload.id
                ? { ...i, quantity: action.payload.quantity }
                : i
            )
      }
    case 'CLEAR_CART':
      return { ...state, items: [] }
    case 'TOGGLE_CART':
      return { ...state, isOpen: !state.isOpen }
    case 'APPLY_COUPON':
      return { ...state, coupon: action.payload.code, discount: action.payload.discount }
    default:
      return state
  }
}

const initialState: State = {
  items: [],
  isOpen: false,
  coupon: null,
  discount: 0,
}

export default function Cart() {
  const [state, dispatch] = useReducer(cartReducer, initialState)
  
  return (
    <div>
      <button onClick={() => dispatch({ type: 'TOGGLE_CART' })}>
        Cart ({state.items.length})
      </button>
      {state.isOpen && (
        <div>
          {state.items.map(item => (
            <div key={item.id}>
              {item.name}
              <button onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: item.id })}>
                Remove
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  )
}
```

#### Context สำหรับ Global State

```tsx
// context/cart-context.tsx
'use client'
import { createContext, useContext, useReducer } from 'react'

// ... (State, Action, Reducer definitions)

const CartContext = createContext<{
  state: State
  dispatch: React.Dispatch<Action>
} | null>(null)

export function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, initialState)
  
  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  )
}

export function useCart() {
  const context = useContext(CartContext)
  if (!context) throw new Error('useCart must be used within CartProvider')
  return context
}

// app/layout.tsx (Server Component)
import { CartProvider } from '@/context/cart-context'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <CartProvider>  {/* Client Component Wrapper */}
          {children}    {/* Server Components ข้างในได้ */}
        </CartProvider>
      </body>
    </html>
  )
}
```

---

## Step 1089: Mixing Server และ Client Components

### 5. Mixing Server และ Client Components

#### Pattern ที่ถูกต้อง

```tsx
// ✅ Server Component ส่ง Server Components เป็น Children
// app/layout.tsx (Server)
export default function Layout({ children }) {
  return (
    <div>
      <ServerHeader />      {/* Server Component */}
      <ClientSidebar>       {/* Client Component */}
        {children}          {/* Server Component passed as prop */}
      </ClientSidebar>
    </div>
  )
}

// components/ClientSidebar.tsx
'use client'
import { useState } from 'react'

export default function ClientSidebar({
  children
}: {
  children: React.ReactNode
}) {
  const [collapsed, setCollapsed] = useState(false)
  
  return (
    <div className={collapsed ? 'w-16' : 'w-64'}>
      <button onClick={() => setCollapsed(!collapsed)}>
        {collapsed ? '→' : '←'}
      </button>
      <nav>{/* sidebar nav */}</nav>
      <main>{children}</main>
    </div>
  )
}
```

#### ❌ Pattern ที่ผิด

```tsx
// ❌ ไม่สามารถ Import Server Component ใน Client Component
'use client'
import ServerComponent from './ServerComponent' // ❌ จะถูกแปลงเป็น Client

export default function ClientComponent() {
  return <ServerComponent />  // ❌ ServerComponent กลายเป็น Client Component
}

// ✅ ทางแก้: ส่งเป็น children หรือ props
'use client'
export default function ClientWrapper({
  children,
  serverContent
}: {
  children: React.ReactNode
  serverContent: React.ReactNode
}) {
  return (
    <div>
      {children}       {/* อาจเป็น Server Component */}
      {serverContent}  {/* อาจเป็น Server Component */}
    </div>
  )
}
```

#### Interleaving Pattern

```tsx
// app/dashboard/page.tsx (Server Component)
import { Suspense } from 'react'
import dynamic from 'next/dynamic'

// Import Client Component
import { InteractiveChart } from '@/components/InteractiveChart'

// Lazy Load Heavy Client Component
const HeavyEditor = dynamic(() => import('@/components/HeavyEditor'), {
  loading: () => <div>Loading editor...</div>,
  ssr: false,  // ไม่ SSR สำหรับ Browser-only Component
})

async function DashboardPage() {
  // Server-side Data Fetching
  const stats = await getStats()
  const recentData = await getRecentData()
  
  return (
    <div>
      {/* Server Rendered */}
      <h1>Dashboard</h1>
      
      {/* Client Component รับ Data จาก Server */}
      <InteractiveChart data={stats} />
      
      {/* Lazy Loaded Client Component */}
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyEditor initialContent={recentData.content} />
      </Suspense>
    </div>
  )
}
```

#### Server Component ใน Client Component Tree

```tsx
// app/page.tsx (Server Component)
import ServerList from '@/components/ServerList'
import ClientContainer from '@/components/ClientContainer'

export default function Page() {
  return (
    <ClientContainer>
      <ServerList />  {/* Server Component เป็น children ของ Client */}
    </ClientContainer>
  )
}

// components/ClientContainer.tsx
'use client'
import { useState } from 'react'

export default function ClientContainer({
  children
}: {
  children: React.ReactNode
}) {
  const [filter, setFilter] = useState('')
  
  return (
    <div>
      <input
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter..."
      />
      {/* children (ServerList) ยังคง Server Component */}
      <div data-filter={filter}>
        {children}
      </div>
    </div>
  )
}
```

---

## Step 1093: Data Passing Patterns

### 6. Data Passing Patterns

#### Pattern 1: Props จาก Server ไป Client

```tsx
// app/products/page.tsx (Server Component)
import ProductFilter from '@/components/ProductFilter'

async function ProductsPage({ searchParams }) {
  const categories = await getCategories()  // Server-side
  const currentCategory = searchParams.category
  
  return (
    <div>
      {/* ส่ง Data จาก Server ไป Client Component */}
      <ProductFilter
        categories={categories}
        currentCategory={currentCategory}
      />
    </div>
  )
}

// components/ProductFilter.tsx (Client Component)
'use client'
import { useRouter, useSearchParams, usePathname } from 'next/navigation'

interface Props {
  categories: { id: string; name: string; slug: string }[]
  currentCategory: string | undefined
}

export default function ProductFilter({ categories, currentCategory }: Props) {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()
  
  const handleChange = (categorySlug: string) => {
    const params = new URLSearchParams(searchParams.toString())
    if (categorySlug) {
      params.set('category', categorySlug)
    } else {
      params.delete('category')
    }
    router.push(`${pathname}?${params.toString()}`)
  }
  
  return (
    <div>
      <button
        onClick={() => handleChange('')}
        className={!currentCategory ? 'active' : ''}
      >
        All
      </button>
      {categories.map(cat => (
        <button
          key={cat.id}
          onClick={() => handleChange(cat.slug)}
          className={currentCategory === cat.slug ? 'active' : ''}
        >
          {cat.name}
        </button>
      ))}
    </div>
  )
}
```

#### Pattern 2: Server Actions สำหรับ Mutations

```tsx
// app/actions/cart.ts
'use server'

import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

export async function addToCart(productId: string, quantity: number) {
  const cookieStore = cookies()
  const cartCookie = cookieStore.get('cart')
  const cart = cartCookie ? JSON.parse(cartCookie.value) : []
  
  const existing = cart.find((i: any) => i.productId === productId)
  if (existing) {
    existing.quantity += quantity
  } else {
    cart.push({ productId, quantity })
  }
  
  cookieStore.set('cart', JSON.stringify(cart), {
    maxAge: 60 * 60 * 24 * 7  // 7 days
  })
  
  revalidatePath('/cart')
}

// components/AddToCartButton.tsx (Client Component)
'use client'
import { useState } from 'react'
import { addToCart } from '@/app/actions/cart'

interface Props {
  productId: string
}

export default function AddToCartButton({ productId }: Props) {
  const [loading, setLoading] = useState(false)
  const [added, setAdded] = useState(false)
  
  const handleClick = async () => {
    setLoading(true)
    await addToCart(productId, 1)
    setAdded(true)
    setLoading(false)
    
    setTimeout(() => setAdded(false), 2000)
  }
  
  return (
    <button
      onClick={handleClick}
      disabled={loading}
      className={`px-4 py-2 rounded ${
        added ? 'bg-green-500' : 'bg-blue-500'
      } text-white`}
    >
      {loading ? 'Adding...' : added ? '✓ Added!' : 'Add to Cart'}
    </button>
  )
}
```

#### Pattern 3: Render Props

```tsx
// components/DataTable.tsx (Client Component)
'use client'
import { useState } from 'react'

interface Props<T> {
  data: T[]
  columns: {
    key: keyof T
    header: string
    render?: (value: T[keyof T], row: T) => React.ReactNode
  }[]
}

export default function DataTable<T extends { id: string }>({
  data,
  columns,
}: Props<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null)
  const [sortOrder, setSortOrder] = useState<'asc' | 'desc'>('asc')
  const [search, setSearch] = useState('')
  
  const handleSort = (key: keyof T) => {
    if (sortKey === key) {
      setSortOrder(o => o === 'asc' ? 'desc' : 'asc')
    } else {
      setSortKey(key)
      setSortOrder('asc')
    }
  }
  
  return (
    <div>
      <input
        type="search"
        value={search}
        onChange={e => setSearch(e.target.value)}
        placeholder="Search..."
        className="mb-4 border px-3 py-2 rounded w-full"
      />
      <table className="w-full">
        <thead>
          <tr>
            {columns.map(col => (
              <th
                key={String(col.key)}
                onClick={() => handleSort(col.key)}
                className="cursor-pointer text-left p-3 bg-gray-100"
              >
                {col.header}
                {sortKey === col.key && (
                  <span>{sortOrder === 'asc' ? ' ↑' : ' ↓'}</span>
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {data.map(row => (
            <tr key={row.id} className="border-t hover:bg-gray-50">
              {columns.map(col => (
                <td key={String(col.key)} className="p-3">
                  {col.render
                    ? col.render(row[col.key], row)
                    : String(row[col.key])}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}

// การใช้งาน (Server Component)
import DataTable from '@/components/DataTable'

async function UsersPage() {
  const users = await getUsers()
  
  return (
    <DataTable
      data={users}
      columns={[
        { key: 'name', header: 'Name' },
        { key: 'email', header: 'Email' },
        {
          key: 'createdAt',
          header: 'Joined',
          render: (value) => new Date(value as string).toLocaleDateString('th-TH')
        },
        {
          key: 'id',
          header: 'Actions',
          render: (_, row) => (
            <div className="flex gap-2">
              <a href={`/users/${row.id}`}>View</a>
              <a href={`/users/${row.id}/edit`}>Edit</a>
            </div>
          )
        }
      ]}
    />
  )
}
```

#### Pattern 4: URL State สำหรับ Shared State

```tsx
// components/ProductSearch.tsx (Client Component)
'use client'
import { useRouter, usePathname, useSearchParams } from 'next/navigation'
import { useCallback, useState } from 'react'
import { useDebouncedCallback } from 'use-debounce'

export default function ProductSearch() {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()
  
  const handleSearch = useDebouncedCallback((term: string) => {
    const params = new URLSearchParams(searchParams.toString())
    
    if (term) {
      params.set('search', term)
    } else {
      params.delete('search')
    }
    
    params.set('page', '1')  // Reset page
    router.replace(`${pathname}?${params.toString()}`)
  }, 300)
  
  return (
    <input
      defaultValue={searchParams.get('search') ?? ''}
      onChange={e => handleSearch(e.target.value)}
      placeholder="ค้นหาสินค้า..."
      className="w-full border px-4 py-2 rounded-lg"
    />
  )
}

// app/products/page.tsx (Server Component)
// ข้อมูลทั้ง Server และ Client Share ผ่าน URL
async function ProductsPage({ searchParams }) {
  const search = searchParams.search
  const page = Number(searchParams.page) || 1
  
  const products = await searchProducts(search, page)
  
  return (
    <div>
      <ProductSearch />  {/* Client: อ่าน/เขียน URL */}
      <ProductList products={products} />  {/* Server: อ่าน URL ผ่าน searchParams */}
    </div>
  )
}
```

---

## Step 1100: Tips และ Best Practices

### Tips และ Best Practices

```markdown
## 1. Push Client Boundary ให้ Leaf ที่สุด

✓ เพิ่ม 'use client' เฉพาะ Component ที่ต้องการจริงๆ
✓ แยก Interactive Parts ออกเป็น Component เล็กๆ

## 2. ส่ง Serializable Data เท่านั้น

✓ Server → Client Props ต้องเป็น JSON-serializable
✗ ไม่ส่ง Functions, Date Objects, Class Instances

## 3. ใช้ URL State สำหรับ Shared State

✓ URL State Share ได้ระหว่าง Server/Client
✓ Shareable, Bookmarkable
✓ ใช้ useSearchParams ใน Client, searchParams ใน Server

## 4. Server Actions สำหรับ Mutations

✓ ใช้ Server Actions แทน API Routes เมื่อทำได้
✓ ง่ายกว่า, Type-safe, ไม่ต้องสร้าง API

## 5. dynamic() สำหรับ Heavy Client Components

✓ ใช้ next/dynamic สำหรับ Browser-only Components
✓ ใส่ ssr: false เพื่อป้องกัน SSR Error
```

---

## Quiz

### แบบทดสอบ Part 37

**คำถามที่ 1:** `'use client'` ต้องอยู่ที่ไหนในไฟล์?
- A) ก่อน import statements
- B) บนสุดของไฟล์ก่อน code อื่นๆ ✓
- C) หลัง import statements
- D) ใน Component Function

**คำถามที่ 2:** ถ้า Parent เป็น Client Component, Child Component ต้องทำอะไร?
- A) ต้องเพิ่ม 'use client' ด้วย
- B) ไม่ต้องเพิ่ม 'use client' เพราะอยู่ใน Client Boundary แล้ว ✓
- C) ต้องเป็น Server Component เท่านั้น
- D) ไม่สามารถมี Child ได้

**คำถามที่ 3:** วิธีถูกต้องในการใช้ Server Component ใน Client Component คืออะไร?
- A) Import Server Component โดยตรง
- B) ส่งเป็น children props ✓
- C) ใช้ dynamic import
- D) ทำไม่ได้เลย

**คำถามที่ 4:** URL State เหมาะสำหรับ State ชนิดใด?
- A) User Authentication State
- B) Shopping Cart Items
- C) Filter, Search, Pagination ที่ต้องการ Share ✓
- D) UI Animation State

**คำถามที่ 5:** `next/dynamic` กับ `{ ssr: false }` ใช้เมื่อไหร่?
- A) ทุกครั้งที่ต้องการ Lazy Load
- B) Component ที่ใช้ Browser APIs เท่านั้น ไม่สามารถ SSR ได้ ✓
- C) Component ขนาดใหญ่ทุกอัน
- D) Server Components

---

## สรุป Part 37

ใน Part นี้เราได้เรียนรู้:

1. **'use client'** - สร้าง Client Boundary สำหรับ Interactive Components
2. **เมื่อไหรใช้** - State, Events, Browser APIs = Client
3. **Interactivity** - Event Handlers, Browser APIs, localStorage
4. **Client State** - useState, useReducer, Context
5. **Mixing Patterns** - Server ส่ง children ไป Client
6. **Data Patterns** - Props, Server Actions, URL State

---

➡️ **Part ถัดไป:** [Part 38: Next.js Middleware](./part-38-nextjs-middleware.md)
