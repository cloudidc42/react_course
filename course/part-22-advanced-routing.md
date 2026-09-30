# Part 22: Advanced Routing - การ Routing ขั้นสูง

**Step 531-560** | ระดับ: สูง | เวลาเรียน: 3-4 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [Lazy Loading Routes](#lazy-loading-routes)
2. [Code Splitting](#code-splitting)
3. [Route Guards / Auth Protection](#route-guards--auth-protection)
4. [Dynamic Routes](#dynamic-routes)
5. [URL Search Params ขั้นสูง](#url-search-params-ขั้นสูง)
6. [Scroll Restoration](#scroll-restoration)
7. [Breadcrumbs](#breadcrumbs)
8. [Route-based Code Splitting](#route-based-code-splitting)
9. [Quiz](#quiz)

---

## Step 531-535: Lazy Loading Routes

Lazy Loading ช่วยให้แอพโหลดเร็วขึ้นโดยโหลด code เฉพาะหน้าที่ต้องการ

### ปัญหาที่แก้ด้วย Lazy Loading

```
ไม่มี Lazy Loading:
- โหลดครั้งแรก: ดาวน์โหลด code ทั้งแอพ 2MB
- ผู้ใช้รอนาน แม้แต่ routes ที่ไม่ได้ไป

มี Lazy Loading:
- โหลดครั้งแรก: ดาวน์โหลดเฉพาะ Home 200KB
- ไป /products: ดาวน์โหลด Products 150KB
- ไป /dashboard: ดาวน์โหลด Dashboard 300KB
```

### การใช้งาน React.lazy กับ Suspense

```jsx
// App.jsx
import { lazy, Suspense } from 'react'
import { Routes, Route } from 'react-router-dom'

// Lazy import แทน static import
const Home = lazy(() => import('./pages/Home'))
const About = lazy(() => import('./pages/About'))
const Products = lazy(() => import('./pages/Products'))
const Dashboard = lazy(() => import('./pages/Dashboard'))
const NotFound = lazy(() => import('./pages/NotFound'))

// Loading Component
function PageLoader() {
  return (
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      minHeight: '50vh',
    }}>
      <div>กำลังโหลด...</div>
    </div>
  )
}

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products" element={<Products />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </Suspense>
  )
}
```

### Custom Loading Component ที่สวยงาม

```jsx
// components/PageLoader.jsx
function PageLoader() {
  return (
    <div style={{
      position: 'fixed',
      top: 0,
      left: 0,
      right: 0,
      bottom: 0,
      display: 'flex',
      flexDirection: 'column',
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: 'rgba(255,255,255,0.9)',
      zIndex: 1000,
    }}>
      {/* Spinner */}
      <div style={{
        width: '50px',
        height: '50px',
        border: '5px solid #f3f3f3',
        borderTop: '5px solid #007bff',
        borderRadius: '50%',
        animation: 'spin 1s linear infinite',
      }} />
      <p style={{ marginTop: '1rem', color: '#666' }}>กำลังโหลดหน้า...</p>

      <style>{`
        @keyframes spin {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
        }
      `}</style>
    </div>
  )
}

export default PageLoader
```

### Skeleton Loading

```jsx
// components/SkeletonLoader.jsx
function SkeletonLoader() {
  return (
    <div style={{ padding: '2rem' }}>
      {/* Header skeleton */}
      <div style={{
        height: '40px',
        width: '60%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '1rem',
        animation: 'pulse 1.5s infinite',
      }} />
      
      {/* Content skeleton */}
      {[1,2,3].map(i => (
        <div key={i} style={{
          height: '20px',
          backgroundColor: '#e0e0e0',
          borderRadius: '4px',
          marginBottom: '0.5rem',
          width: `${100 - i * 10}%`,
          animation: 'pulse 1.5s infinite',
        }} />
      ))}

      <style>{`
        @keyframes pulse {
          0%, 100% { opacity: 1; }
          50% { opacity: 0.5; }
        }
      `}</style>
    </div>
  )
}

export default SkeletonLoader
```

---

## Step 536-540: Code Splitting

### Vite Bundle Analysis

```bash
# ติดตั้ง visualizer
npm install --save-dev rollup-plugin-visualizer

# วิเคราะห์ bundle size
npm run build
```

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      open: true,
      gzipSize: true,
    }),
  ],
  build: {
    rollupOptions: {
      output: {
        // แยก vendor chunks
        manualChunks: {
          'react-vendor': ['react', 'react-dom'],
          'router': ['react-router-dom'],
          'ui': ['@mui/material', '@mui/icons-material'],
        }
      }
    }
  }
})
```

### Dynamic Import นอกจาก Routes

```jsx
// Lazy load heavy library เฉพาะเมื่อต้องการ
function ChartPage() {
  const [ChartComponent, setChartComponent] = useState(null)

  const loadChart = async () => {
    const { default: Chart } = await import('./components/HeavyChart')
    setChartComponent(() => Chart)
  }

  return (
    <div>
      <button onClick={loadChart}>แสดงกราฟ</button>
      {ChartComponent && <ChartComponent />}
    </div>
  )
}
```

---

## Step 541-545: Route Guards / Auth Protection

### Route Guard หลายระดับ

```jsx
// components/guards/AuthGuard.jsx
import { Navigate, useLocation } from 'react-router-dom'
import { useAuth } from '../../hooks/useAuth'
import LoadingSpinner from '../LoadingSpinner'

function AuthGuard({ children, redirectTo = '/login' }) {
  const { user, isLoading } = useAuth()
  const location = useLocation()

  if (isLoading) {
    return <LoadingSpinner />
  }

  if (!user) {
    return (
      <Navigate 
        to={redirectTo} 
        state={{ from: location.pathname }} 
        replace 
      />
    )
  }

  return children
}

export default AuthGuard
```

```jsx
// components/guards/RoleGuard.jsx
import { Navigate } from 'react-router-dom'
import { useAuth } from '../../hooks/useAuth'

function RoleGuard({ children, allowedRoles, fallback = '/unauthorized' }) {
  const { user } = useAuth()

  if (!user) {
    return <Navigate to="/login" replace />
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to={fallback} replace />
  }

  return children
}

export default RoleGuard
```

```jsx
// components/guards/GuestGuard.jsx
// ป้องกันไม่ให้ user ที่ login แล้วเข้าหน้า login/register
import { Navigate } from 'react-router-dom'
import { useAuth } from '../../hooks/useAuth'

function GuestGuard({ children }) {
  const { user, isLoading } = useAuth()

  if (isLoading) return null

  if (user) {
    return <Navigate to="/dashboard" replace />
  }

  return children
}

export default GuestGuard
```

### การใช้ Guards ใน App.jsx

```jsx
// App.jsx
import AuthGuard from './components/guards/AuthGuard'
import RoleGuard from './components/guards/RoleGuard'
import GuestGuard from './components/guards/GuestGuard'

function App() {
  return (
    <Routes>
      {/* Guest only routes */}
      <Route path="/login" element={
        <GuestGuard>
          <Login />
        </GuestGuard>
      } />
      
      {/* Auth required */}
      <Route path="/dashboard" element={
        <AuthGuard>
          <Dashboard />
        </AuthGuard>
      } />

      {/* Admin only */}
      <Route path="/admin" element={
        <AuthGuard>
          <RoleGuard allowedRoles={['admin']}>
            <AdminPanel />
          </RoleGuard>
        </AuthGuard>
      } />

      {/* Editor & Admin */}
      <Route path="/cms" element={
        <AuthGuard>
          <RoleGuard allowedRoles={['admin', 'editor']}>
            <CmsPanel />
          </RoleGuard>
        </AuthGuard>
      } />
    </Routes>
  )
}
```

---

## Step 546-548: Dynamic Routes

### Wildcard Routes

```jsx
// จับทุก path ที่ขึ้นต้นด้วย /docs/
<Route path="/docs/*" element={<DocsLayout />}>
  <Route path="*" element={<DocPage />} />
</Route>
```

```jsx
// pages/DocPage.jsx
import { useParams } from 'react-router-dom'

function DocPage() {
  const params = useParams()
  const docPath = params['*']  // จะได้ path หลัง /docs/

  return (
    <div>
      <p>กำลังแสดง: {docPath}</p>
    </div>
  )
}
```

### Optional Route Segments

```jsx
// React Router v6.4+ รองรับ optional segments
<Route path="/products/:id?" element={<Products />} />
// ตรงกับ: /products, /products/1, /products/abc
```

---

## Step 549-552: URL Search Params ขั้นสูง

### Custom Hook สำหรับ Search Params

```jsx
// hooks/useQueryParams.js
import { useSearchParams } from 'react-router-dom'
import { useCallback } from 'react'

function useQueryParams() {
  const [searchParams, setSearchParams] = useSearchParams()

  const getParam = useCallback((key, defaultValue = '') => {
    return searchParams.get(key) || defaultValue
  }, [searchParams])

  const setParam = useCallback((key, value) => {
    setSearchParams(prev => {
      if (value === null || value === '') {
        prev.delete(key)
      } else {
        prev.set(key, value)
      }
      return prev
    })
  }, [setSearchParams])

  const setParams = useCallback((updates) => {
    setSearchParams(prev => {
      Object.entries(updates).forEach(([key, value]) => {
        if (value === null || value === '') {
          prev.delete(key)
        } else {
          prev.set(key, value)
        }
      })
      return prev
    })
  }, [setSearchParams])

  const clearParams = useCallback(() => {
    setSearchParams({})
  }, [setSearchParams])

  return {
    getParam,
    setParam,
    setParams,
    clearParams,
    searchParams,
  }
}

export default useQueryParams
```

### Filter Component กับ URL State

```jsx
// components/ProductFilters.jsx
import useQueryParams from '../hooks/useQueryParams'

const CATEGORIES = ['ทั้งหมด', 'อิเล็กทรอนิกส์', 'เสื้อผ้า', 'อาหาร']
const SORT_OPTIONS = [
  { value: 'name_asc', label: 'ชื่อ A-Z' },
  { value: 'name_desc', label: 'ชื่อ Z-A' },
  { value: 'price_asc', label: 'ราคาต่ำ-สูง' },
  { value: 'price_desc', label: 'ราคาสูง-ต่ำ' },
]

function ProductFilters() {
  const { getParam, setParam, setParams, clearParams } = useQueryParams()

  const category = getParam('category', 'ทั้งหมด')
  const sort = getParam('sort', 'name_asc')
  const minPrice = getParam('minPrice', '')
  const maxPrice = getParam('maxPrice', '')

  return (
    <div style={{
      background: '#f5f5f5',
      padding: '1.5rem',
      borderRadius: '8px',
      marginBottom: '1rem',
    }}>
      <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '1rem' }}>
        <h3>ตัวกรอง</h3>
        <button onClick={clearParams}>ล้างตัวกรองทั้งหมด</button>
      </div>

      {/* Category */}
      <div style={{ marginBottom: '1rem' }}>
        <label>หมวดหมู่:</label>
        <div style={{ display: 'flex', gap: '0.5rem', flexWrap: 'wrap', marginTop: '0.5rem' }}>
          {CATEGORIES.map(cat => (
            <button
              key={cat}
              onClick={() => setParam('category', cat === 'ทั้งหมด' ? null : cat)}
              style={{
                padding: '0.25rem 0.75rem',
                backgroundColor: category === cat ? '#007bff' : 'white',
                color: category === cat ? 'white' : 'black',
                border: '1px solid #007bff',
                borderRadius: '20px',
                cursor: 'pointer',
              }}
            >
              {cat}
            </button>
          ))}
        </div>
      </div>

      {/* Sort */}
      <div style={{ marginBottom: '1rem' }}>
        <label>เรียงตาม: </label>
        <select
          value={sort}
          onChange={e => setParam('sort', e.target.value)}
          style={{ marginLeft: '0.5rem', padding: '0.25rem' }}
        >
          {SORT_OPTIONS.map(opt => (
            <option key={opt.value} value={opt.value}>{opt.label}</option>
          ))}
        </select>
      </div>

      {/* Price Range */}
      <div>
        <label>ช่วงราคา:</label>
        <div style={{ display: 'flex', gap: '0.5rem', alignItems: 'center', marginTop: '0.5rem' }}>
          <input
            type="number"
            value={minPrice}
            onChange={e => setParam('minPrice', e.target.value)}
            placeholder="ต่ำสุด"
            style={{ width: '100px', padding: '0.25rem' }}
          />
          <span>-</span>
          <input
            type="number"
            value={maxPrice}
            onChange={e => setParam('maxPrice', e.target.value)}
            placeholder="สูงสุด"
            style={{ width: '100px', padding: '0.25rem' }}
          />
          <span>บาท</span>
        </div>
      </div>
    </div>
  )
}

export default ProductFilters
```

---

## Step 553-555: Scroll Restoration

React Router ไม่ scroll to top อัตโนมัติเมื่อ navigate

### ScrollToTop Component

```jsx
// components/ScrollToTop.jsx
import { useEffect } from 'react'
import { useLocation } from 'react-router-dom'

function ScrollToTop() {
  const { pathname } = useLocation()

  useEffect(() => {
    window.scrollTo(0, 0)
  }, [pathname])

  return null
}

export default ScrollToTop
```

```jsx
// App.jsx
import ScrollToTop from './components/ScrollToTop'

function App() {
  return (
    <BrowserRouter>
      <ScrollToTop />  {/* วางไว้ใน Router */}
      <Routes>
        {/* ... */}
      </Routes>
    </BrowserRouter>
  )
}
```

### Scroll กับ Hash

```jsx
// components/ScrollToHash.jsx
import { useEffect } from 'react'
import { useLocation } from 'react-router-dom'

function ScrollToHash() {
  const { hash } = useLocation()

  useEffect(() => {
    if (hash) {
      const element = document.getElementById(hash.slice(1))
      if (element) {
        element.scrollIntoView({ behavior: 'smooth' })
      }
    } else {
      window.scrollTo(0, 0)
    }
  }, [hash])

  return null
}
```

### Save and Restore Scroll Position

```jsx
// hooks/useScrollPosition.js
import { useEffect, useRef } from 'react'
import { useLocation } from 'react-router-dom'

function useScrollPosition() {
  const location = useLocation()
  const scrollPositions = useRef({})

  // บันทึก scroll position ก่อน navigate
  useEffect(() => {
    return () => {
      scrollPositions.current[location.pathname] = window.scrollY
    }
  }, [location.pathname])

  // restore scroll position
  useEffect(() => {
    const savedPosition = scrollPositions.current[location.pathname]
    if (savedPosition !== undefined) {
      window.scrollTo(0, savedPosition)
    } else {
      window.scrollTo(0, 0)
    }
  }, [location.pathname])
}

export default useScrollPosition
```

---

## Step 556-558: Breadcrumbs

Breadcrumbs แสดงลำดับของหน้าที่ผู้ใช้อยู่

### Route Config กับ Breadcrumbs

```jsx
// routes/routeConfig.js
export const routes = [
  { path: '/', label: 'หน้าแรก' },
  { path: '/products', label: 'สินค้า' },
  { path: '/products/:id', label: 'รายละเอียดสินค้า' },
  { path: '/dashboard', label: 'Dashboard' },
  { path: '/dashboard/profile', label: 'โปรไฟล์' },
  { path: '/dashboard/settings', label: 'ตั้งค่า' },
]
```

### Breadcrumbs Component

```jsx
// components/Breadcrumbs.jsx
import { Link, useLocation, useParams } from 'react-router-dom'

function Breadcrumbs() {
  const location = useLocation()
  const params = useParams()
  
  const pathnames = location.pathname.split('/').filter(Boolean)

  const buildPath = (index) => {
    return '/' + pathnames.slice(0, index + 1).join('/')
  }

  const getLabel = (segment) => {
    // แปลง path segment เป็น label ภาษาไทย
    const labels = {
      'products': 'สินค้า',
      'about': 'เกี่ยวกับ',
      'dashboard': 'Dashboard',
      'profile': 'โปรไฟล์',
      'settings': 'ตั้งค่า',
    }
    return labels[segment] || segment
  }

  return (
    <nav aria-label="breadcrumb" style={{ marginBottom: '1rem' }}>
      <ol style={{
        display: 'flex',
        listStyle: 'none',
        padding: 0,
        margin: 0,
        flexWrap: 'wrap',
        gap: '0.25rem',
      }}>
        <li>
          <Link to="/" style={{ color: '#007bff' }}>หน้าแรก</Link>
        </li>

        {pathnames.map((segment, index) => {
          const isLast = index === pathnames.length - 1
          const path = buildPath(index)

          return (
            <li key={path} style={{ display: 'flex', alignItems: 'center' }}>
              <span style={{ margin: '0 0.5rem', color: '#999' }}>/</span>
              {isLast ? (
                <span style={{ color: '#333' }}>{getLabel(segment)}</span>
              ) : (
                <Link to={path} style={{ color: '#007bff' }}>
                  {getLabel(segment)}
                </Link>
              )}
            </li>
          )
        })}
      </ol>
    </nav>
  )
}

export default Breadcrumbs
```

---

## Step 559-560: Route-based Code Splitting ขั้นสูง

### Preloading Routes

```jsx
// โหลดหน้าล่วงหน้าเมื่อ hover
import { lazy, Suspense } from 'react'

const Products = lazy(() => import('./pages/Products'))

// Preload function
function preloadProducts() {
  import('./pages/Products')
}

function Navbar() {
  return (
    <nav>
      <Link 
        to="/products"
        onMouseEnter={preloadProducts}  // โหลดล่วงหน้าเมื่อ hover
      >
        สินค้า
      </Link>
    </nav>
  )
}
```

### createBrowserRouter (v6.4+)

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      {
        index: true,
        element: <Home />,
        loader: async () => {
          // ดึงข้อมูลก่อน render
          const res = await fetch('/api/home')
          return res.json()
        },
      },
      {
        path: 'products',
        element: <Products />,
        loader: async () => {
          const res = await fetch('/api/products')
          return res.json()
        },
      },
      {
        path: 'products/:id',
        element: <ProductDetail />,
        loader: async ({ params }) => {
          const res = await fetch(`/api/products/${params.id}`)
          if (!res.ok) throw new Response('Not Found', { status: 404 })
          return res.json()
        },
        errorElement: <ProductError />,
      },
    ],
  },
])

function App() {
  return <RouterProvider router={router} />
}
```

### useLoaderData Hook (v6.4+)

```jsx
import { useLoaderData } from 'react-router-dom'

function Products() {
  const products = useLoaderData()  // ข้อมูลจาก loader

  return (
    <div>
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  )
}
```

### Route Actions (v6.4+)

```jsx
import { Form, useActionData, redirect } from 'react-router-dom'

// Action function
async function createProductAction({ request }) {
  const formData = await request.formData()
  const data = Object.fromEntries(formData)

  const res = await fetch('/api/products', {
    method: 'POST',
    body: JSON.stringify(data),
    headers: { 'Content-Type': 'application/json' }
  })

  if (!res.ok) {
    return { error: 'เกิดข้อผิดพลาด' }
  }

  return redirect('/products')
}

// Component
function CreateProduct() {
  const actionData = useActionData()

  return (
    <Form method="post" action="/products/new">
      <input name="name" placeholder="ชื่อสินค้า" />
      <input name="price" type="number" placeholder="ราคา" />
      <button type="submit">เพิ่มสินค้า</button>
      {actionData?.error && <p style={{ color: 'red' }}>{actionData.error}</p>}
    </Form>
  )
}
```

---

## Tips และ Best Practices

### 1. อย่า Lazy Load ทุก Component

```jsx
// ❌ Over-optimize - ไม่ต้องแยก small components
const Button = lazy(() => import('./Button'))  // ไม่จำเป็น

// ✅ Lazy load เฉพาะ pages หรือ heavy components
const Dashboard = lazy(() => import('./pages/Dashboard'))
const ChartLibrary = lazy(() => import('./HeavyChartComponent'))
```

### 2. จัดการ Suspense boundary อย่างเหมาะสม

```jsx
// ✅ ดี - Suspense ครอบ routes ทั้งหมด
<Suspense fallback={<PageLoader />}>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</Suspense>

// หรือ per-route
<Routes>
  <Route path="/" element={
    <Suspense fallback={<HomeSkeleton />}>
      <Home />
    </Suspense>
  } />
</Routes>
```

### 3. ใช้ URL เป็น Single Source of Truth

```jsx
// ✅ State ควรอยู่ใน URL เมื่อต้องการ share/bookmark
const [searchParams, setSearchParams] = useSearchParams()

// ❌ ไม่ดี - state ใน component จะหายเมื่อ refresh
const [filter, setFilter] = useState('all')
```

### 4. Error Boundaries กับ Lazy Routes

```jsx
import { Suspense } from 'react'
import ErrorBoundary from './components/ErrorBoundary'

const LazyComponent = lazy(() => import('./pages/HeavyPage'))

function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Suspense fallback={<PageLoader />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  )
}
```

---

## Quiz - Part 22

**ข้อ 1**: `React.lazy()` ต้องใช้ร่วมกับอะไร?
- a) ErrorBoundary
- b) Suspense
- c) useEffect
- d) memo

**ข้อ 2**: Route loader ใน v6.4+ ทำงานเมื่อใด?
- a) หลัง component render
- b) ก่อน component render
- c) เมื่อ click ปุ่ม submit
- d) เมื่อ component unmount

**ข้อ 3**: การใช้ URL เป็น state มีประโยชน์อย่างไร?
- a) ทำให้แอพเร็วขึ้น
- b) สามารถ bookmark/share URL ที่มี state ได้
- c) ลดการใช้ RAM
- d) ทำให้ code สั้นลง

**ข้อ 4**: Breadcrumbs ควรแสดงอะไร?
- a) รายการสินค้าทั้งหมด
- b) เส้นทางจากหน้าแรกถึงหน้าปัจจุบัน
- c) ประวัติการเข้าชมทั้งหมด
- d) Menu ด้านบน

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 22

ใน Part นี้คุณได้เรียนรู้:
- ✅ Lazy Loading Routes ด้วย React.lazy + Suspense
- ✅ Code Splitting และ Bundle Analysis
- ✅ Route Guards หลายระดับ (Auth, Role, Guest)
- ✅ Dynamic Routes แบบขั้นสูง
- ✅ URL Search Params Custom Hook
- ✅ Scroll Restoration
- ✅ Breadcrumbs Component
- ✅ Route-based Code Splitting ด้วย createBrowserRouter v6.4+

---

## Part ถัดไป

➡️ **[Part 23: API Fetching](./part-23-api-fetching.md)**
- Fetch API
- Axios
- Loading, Error, Success States
- AbortController
