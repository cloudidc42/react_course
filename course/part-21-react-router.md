# Part 21: React Router DOM v6 - การนำทางในแอปพลิเคชัน

**Step 496-530** | ระดับ: ปานกลาง | เวลาเรียน: 3-4 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [React Router คืออะไร?](#react-router-คืออะไร)
2. [การติดตั้ง React Router v6](#การติดตั้ง-react-router-v6)
3. [BrowserRouter, Routes, Route](#browserrouter-routes-route)
4. [Link และ NavLink](#link-และ-navlink)
5. [Navigate Component](#navigate-component)
6. [useNavigate Hook](#usenavigate-hook)
7. [useParams Hook](#useparams-hook)
8. [useSearchParams Hook](#usesearchparams-hook)
9. [useLocation Hook](#uselocation-hook)
10. [Nested Routes](#nested-routes)
11. [Protected Routes](#protected-routes)
12. [404 Not Found Page](#404-not-found-page)
13. [ตัวอย่าง Multi-page App](#ตัวอย่าง-multi-page-app)
14. [Quiz](#quiz)

---

## Step 496: React Router คืออะไร?

React เป็น Single Page Application (SPA) ซึ่งหมายความว่าแอพทั้งหมดโหลดในหน้าเดียว React Router ช่วยให้เราสามารถจำลองการนำทางระหว่างหน้าต่างๆ ได้โดยไม่ต้องโหลดหน้าใหม่

### ประวัติ React Router

- **v5**: ใช้ `Switch` และมี API แตกต่างจาก v6
- **v6**: เขียนใหม่ทั้งหมด ใช้ `Routes` แทน `Switch`, API ง่ายขึ้นมาก

### ทำไมต้อง React Router?

```
ไม่มี Router:
URL: /home → โหลดทั้งหน้าใหม่
URL: /about → โหลดทั้งหน้าใหม่
URL: /products → โหลดทั้งหน้าใหม่

มี React Router:
URL: /home → เปลี่ยน Component ทันที ไม่โหลดหน้าใหม่
URL: /about → เปลี่ยน Component ทันที ไม่โหลดหน้าใหม่
URL: /products → เปลี่ยน Component ทันที ไม่โหลดหน้าใหม่
```

---

## Step 497: การติดตั้ง React Router v6

```bash
# npm
npm install react-router-dom

# yarn
yarn add react-router-dom

# ตรวจสอบ version
npm list react-router-dom
```

### โครงสร้างโปรเจกต์

```
src/
├── components/
│   ├── Navbar.jsx
│   └── Footer.jsx
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Products.jsx
│   ├── ProductDetail.jsx
│   └── NotFound.jsx
├── App.jsx
└── main.jsx
```

---

## Step 498: BrowserRouter, Routes, Route

### การตั้งค่าพื้นฐาน

```jsx
// main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
)
```

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom'
import Home from './pages/Home'
import About from './pages/About'
import Products from './pages/Products'
import NotFound from './pages/NotFound'
import Navbar from './components/Navbar'

function App() {
  return (
    <div>
      <Navbar />
      <main>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/products" element={<Products />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </main>
    </div>
  )
}

export default App
```

### ความแตกต่างระหว่าง Router Types

| Router | ใช้กับ | URL ตัวอย่าง |
|--------|--------|------------|
| `BrowserRouter` | Web apps ทั่วไป | `https://app.com/about` |
| `HashRouter` | Static hosting | `https://app.com/#/about` |
| `MemoryRouter` | Testing/React Native | ไม่มี URL จริง |

---

## Step 499: หน้า Pages พื้นฐาน

```jsx
// pages/Home.jsx
function Home() {
  return (
    <div className="page">
      <h1>หน้าแรก</h1>
      <p>ยินดีต้อนรับสู่เว็บไซต์ของเรา</p>
    </div>
  )
}

export default Home
```

```jsx
// pages/About.jsx
function About() {
  return (
    <div className="page">
      <h1>เกี่ยวกับเรา</h1>
      <p>เราเป็นบริษัทที่ดีที่สุดในประเทศ</p>
    </div>
  )
}

export default About
```

```jsx
// pages/Products.jsx
const products = [
  { id: 1, name: 'สินค้า A', price: 100 },
  { id: 2, name: 'สินค้า B', price: 200 },
  { id: 3, name: 'สินค้า C', price: 300 },
]

function Products() {
  return (
    <div className="page">
      <h1>สินค้าทั้งหมด</h1>
      <ul>
        {products.map(product => (
          <li key={product.id}>
            {product.name} - ฿{product.price}
          </li>
        ))}
      </ul>
    </div>
  )
}

export default Products
```

---

## Step 500: Link และ NavLink

### Link Component

`Link` ใช้แทน `<a>` tag ธรรมดา เพื่อไม่ให้หน้า refresh

```jsx
import { Link } from 'react-router-dom'

// ❌ ไม่ดี - ทำให้หน้า refresh
<a href="/about">เกี่ยวกับ</a>

// ✅ ดี - ไม่ refresh หน้า
<Link to="/about">เกี่ยวกับ</Link>
```

```jsx
// components/Navbar.jsx
import { Link } from 'react-router-dom'

function Navbar() {
  return (
    <nav style={{ 
      display: 'flex', 
      gap: '1rem', 
      padding: '1rem',
      backgroundColor: '#333',
    }}>
      <Link to="/" style={{ color: 'white' }}>หน้าแรก</Link>
      <Link to="/about" style={{ color: 'white' }}>เกี่ยวกับ</Link>
      <Link to="/products" style={{ color: 'white' }}>สินค้า</Link>
    </nav>
  )
}

export default Navbar
```

### NavLink Component

`NavLink` เหมือน `Link` แต่สามารถกำหนด style พิเศษเมื่อ active ได้

```jsx
import { NavLink } from 'react-router-dom'

function Navbar() {
  const linkStyle = ({ isActive }) => ({
    color: isActive ? 'yellow' : 'white',
    fontWeight: isActive ? 'bold' : 'normal',
    textDecoration: isActive ? 'underline' : 'none',
  })

  return (
    <nav style={{ display: 'flex', gap: '1rem', padding: '1rem', backgroundColor: '#333' }}>
      <NavLink to="/" style={linkStyle} end>
        หน้าแรก
      </NavLink>
      <NavLink to="/about" style={linkStyle}>
        เกี่ยวกับ
      </NavLink>
      <NavLink to="/products" style={linkStyle}>
        สินค้า
      </NavLink>
    </nav>
  )
}
```

> **หมายเหตุ**: `end` prop บน `/` ทำให้ active เฉพาะเมื่อ URL ตรงกับ `/` พอดี ไม่ใช่ prefix

### NavLink กับ className

```jsx
<NavLink
  to="/products"
  className={({ isActive, isPending }) =>
    isActive ? 'nav-link active' : 'nav-link'
  }
>
  สินค้า
</NavLink>
```

```css
/* CSS */
.nav-link {
  color: white;
  text-decoration: none;
  padding: 0.5rem 1rem;
}

.nav-link.active {
  color: yellow;
  border-bottom: 2px solid yellow;
}
```

---

## Step 501: Navigate Component

`Navigate` ใช้สำหรับ redirect อัตโนมัติ

```jsx
import { Navigate } from 'react-router-dom'

// Redirect จากหน้าเก่าไปหน้าใหม่
function OldPage() {
  return <Navigate to="/new-page" replace />
}

// Redirect ตามเงื่อนไข
function Dashboard() {
  const isLoggedIn = false

  if (!isLoggedIn) {
    return <Navigate to="/login" replace />
  }

  return <div>Dashboard Content</div>
}
```

### replace vs push

```jsx
// replace - ไม่สามารถกด back ได้
<Navigate to="/login" replace />

// push (default) - สามารถกด back ได้
<Navigate to="/login" />
```

---

## Step 502: useNavigate Hook

`useNavigate` ใช้สำหรับ navigate ด้วย code (programmatic navigation)

```jsx
import { useNavigate } from 'react-router-dom'

function LoginForm() {
  const navigate = useNavigate()

  const handleLogin = async (e) => {
    e.preventDefault()
    // ทำ login logic
    const success = await loginAPI()

    if (success) {
      navigate('/dashboard')  // ไปหน้า dashboard
    }
  }

  return (
    <form onSubmit={handleLogin}>
      <input type="text" placeholder="Username" />
      <input type="password" placeholder="Password" />
      <button type="submit">เข้าสู่ระบบ</button>
    </form>
  )
}
```

### navigate options

```jsx
const navigate = useNavigate()

// navigate ไปหน้าต่างๆ
navigate('/about')                    // ไปหน้า about
navigate('/products/1')               // ไปหน้า product id 1
navigate(-1)                          // ย้อนกลับ (เหมือนกด Back)
navigate(1)                           // ไปข้างหน้า (เหมือนกด Forward)
navigate('/dashboard', { replace: true })  // replace history
navigate('/search', { state: { from: 'home' } })  // ส่ง state ไปด้วย
```

### ตัวอย่างที่ใช้บ่อย

```jsx
function ProductCard({ product }) {
  const navigate = useNavigate()

  return (
    <div 
      className="product-card"
      onClick={() => navigate(`/products/${product.id}`)}
    >
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
    </div>
  )
}
```

---

## Step 503: useParams Hook

`useParams` ใช้ดึงค่า dynamic segments จาก URL

### การกำหนด Dynamic Route

```jsx
// App.jsx
<Routes>
  <Route path="/products" element={<Products />} />
  <Route path="/products/:id" element={<ProductDetail />} />
  <Route path="/users/:userId/posts/:postId" element={<PostDetail />} />
</Routes>
```

### การใช้งาน useParams

```jsx
// pages/ProductDetail.jsx
import { useParams } from 'react-router-dom'
import { useState, useEffect } from 'react'

function ProductDetail() {
  const { id } = useParams()
  const [product, setProduct] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    // ดึงข้อมูลสินค้าตาม id
    fetch(`/api/products/${id}`)
      .then(res => res.json())
      .then(data => {
        setProduct(data)
        setLoading(false)
      })
  }, [id])

  if (loading) return <div>กำลังโหลด...</div>
  if (!product) return <div>ไม่พบสินค้า</div>

  return (
    <div>
      <h1>{product.name}</h1>
      <p>ราคา: ฿{product.price}</p>
      <p>{product.description}</p>
    </div>
  )
}

export default ProductDetail
```

```jsx
// ตัวอย่าง Multiple params
function PostDetail() {
  const { userId, postId } = useParams()

  return (
    <div>
      <p>User ID: {userId}</p>
      <p>Post ID: {postId}</p>
    </div>
  )
}
```

---

## Step 504: useSearchParams Hook

`useSearchParams` ใช้จัดการ query string ใน URL เช่น `?page=2&sort=price`

```jsx
import { useSearchParams } from 'react-router-dom'

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams()

  // อ่านค่าจาก URL
  const page = searchParams.get('page') || '1'
  const sort = searchParams.get('sort') || 'name'
  const search = searchParams.get('q') || ''

  const handleSearch = (e) => {
    const query = e.target.value
    setSearchParams({ q: query, page: '1' })
  }

  const handleSort = (newSort) => {
    setSearchParams(prev => {
      prev.set('sort', newSort)
      return prev
    })
  }

  const handlePageChange = (newPage) => {
    setSearchParams(prev => {
      prev.set('page', String(newPage))
      return prev
    })
  }

  return (
    <div>
      <input
        value={search}
        onChange={handleSearch}
        placeholder="ค้นหาสินค้า..."
      />

      <div>
        <span>เรียงตาม: </span>
        <button onClick={() => handleSort('name')}>ชื่อ</button>
        <button onClick={() => handleSort('price')}>ราคา</button>
        <button onClick={() => handleSort('date')}>วันที่</button>
      </div>

      {/* แสดงสินค้า */}

      <div>
        <button onClick={() => handlePageChange(Number(page) - 1)}>
          ก่อนหน้า
        </button>
        <span>หน้า {page}</span>
        <button onClick={() => handlePageChange(Number(page) + 1)}>
          ถัดไป
        </button>
      </div>
    </div>
  )
}
```

---

## Step 505: useLocation Hook

`useLocation` ให้ข้อมูลเกี่ยวกับ URL ปัจจุบัน

```jsx
import { useLocation } from 'react-router-dom'

function CurrentPage() {
  const location = useLocation()

  console.log(location)
  // {
  //   pathname: '/products',
  //   search: '?page=2',
  //   hash: '#top',
  //   state: { from: '/home' },
  //   key: 'default'
  // }

  return (
    <div>
      <p>URL: {location.pathname}</p>
      <p>Query: {location.search}</p>
      <p>Hash: {location.hash}</p>
    </div>
  )
}
```

### ตัวอย่าง: บันทึกหน้าก่อนหน้า

```jsx
// ส่ง state ขณะ navigate
function LoginButton() {
  const navigate = useNavigate()
  const location = useLocation()

  const handleClick = () => {
    navigate('/login', {
      state: { from: location.pathname }
    })
  }

  return <button onClick={handleClick}>เข้าสู่ระบบ</button>
}

// รับ state ในหน้า Login
function LoginPage() {
  const location = useLocation()
  const navigate = useNavigate()

  const from = location.state?.from || '/'

  const handleLogin = () => {
    // หลัง login สำเร็จ
    navigate(from, { replace: true })
  }

  return (
    <div>
      <p>คุณจะถูก redirect ไปยัง: {from}</p>
      <button onClick={handleLogin}>เข้าสู่ระบบ</button>
    </div>
  )
}
```

---

## Step 506-510: Nested Routes

Nested Routes ช่วยให้เราสร้าง Layout ที่ซ้อนกันได้

### โครงสร้างพื้นฐาน

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom'
import Dashboard from './pages/Dashboard'
import DashboardHome from './pages/DashboardHome'
import DashboardProfile from './pages/DashboardProfile'
import DashboardSettings from './pages/DashboardSettings'

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      
      {/* Nested Routes */}
      <Route path="/dashboard" element={<Dashboard />}>
        <Route index element={<DashboardHome />} />
        <Route path="profile" element={<DashboardProfile />} />
        <Route path="settings" element={<DashboardSettings />} />
      </Route>
    </Routes>
  )
}
```

### Dashboard Layout (Parent)

```jsx
// pages/Dashboard.jsx
import { Outlet, NavLink } from 'react-router-dom'

function Dashboard() {
  return (
    <div style={{ display: 'flex' }}>
      {/* Sidebar */}
      <aside style={{ width: '200px', background: '#f5f5f5', padding: '1rem' }}>
        <h3>Dashboard</h3>
        <nav>
          <NavLink to="/dashboard" end style={({ isActive }) => ({
            display: 'block',
            padding: '0.5rem',
            color: isActive ? 'blue' : 'black',
          })}>
            หน้าหลัก
          </NavLink>
          <NavLink to="/dashboard/profile" style={({ isActive }) => ({
            display: 'block',
            padding: '0.5rem',
            color: isActive ? 'blue' : 'black',
          })}>
            โปรไฟล์
          </NavLink>
          <NavLink to="/dashboard/settings" style={({ isActive }) => ({
            display: 'block',
            padding: '0.5rem',
            color: isActive ? 'blue' : 'black',
          })}>
            ตั้งค่า
          </NavLink>
        </nav>
      </aside>

      {/* Main Content - Outlet จะ render child routes ที่นี่ */}
      <main style={{ flex: 1, padding: '1rem' }}>
        <Outlet />
      </main>
    </div>
  )
}

export default Dashboard
```

```jsx
// pages/DashboardHome.jsx
function DashboardHome() {
  return (
    <div>
      <h2>ยินดีต้อนรับสู่ Dashboard</h2>
      <p>เลือกเมนูด้านซ้ายเพื่อเริ่มต้น</p>
    </div>
  )
}

export default DashboardHome
```

### Outlet กับ Context

```jsx
// ส่ง context ผ่าน Outlet
function Dashboard() {
  const user = { name: 'สมชาย', role: 'admin' }

  return (
    <div>
      <Outlet context={user} />
    </div>
  )
}

// รับ context ใน child
import { useOutletContext } from 'react-router-dom'

function DashboardProfile() {
  const user = useOutletContext()

  return <div>สวัสดี, {user.name}</div>
}
```

---

## Step 511-515: Protected Routes

Protected Routes ป้องกันไม่ให้ผู้ใช้ที่ยังไม่ได้ login เข้าถึงหน้าบางหน้า

### สร้าง ProtectedRoute Component

```jsx
// components/ProtectedRoute.jsx
import { Navigate, useLocation } from 'react-router-dom'

function ProtectedRoute({ children }) {
  const isAuthenticated = localStorage.getItem('token') !== null
  const location = useLocation()

  if (!isAuthenticated) {
    // redirect ไปหน้า login แล้วจำ URL ที่ต้องการไป
    return <Navigate to="/login" state={{ from: location }} replace />
  }

  return children
}

export default ProtectedRoute
```

### การใช้งาน Protected Routes

```jsx
// App.jsx
import ProtectedRoute from './components/ProtectedRoute'

function App() {
  return (
    <Routes>
      {/* Public Routes */}
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/login" element={<Login />} />

      {/* Protected Routes */}
      <Route path="/dashboard" element={
        <ProtectedRoute>
          <Dashboard />
        </ProtectedRoute>
      } />
      <Route path="/profile" element={
        <ProtectedRoute>
          <Profile />
        </ProtectedRoute>
      } />
    </Routes>
  )
}
```

### Protected Route แบบ Advanced

```jsx
// components/ProtectedRoute.jsx
import { Navigate, useLocation } from 'react-router-dom'
import { useAuth } from '../hooks/useAuth'

function ProtectedRoute({ children, requiredRole }) {
  const { user, isLoading } = useAuth()
  const location = useLocation()

  if (isLoading) {
    return <div>กำลังตรวจสอบสิทธิ์...</div>
  }

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }

  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />
  }

  return children
}

// ใช้งาน
<Route path="/admin" element={
  <ProtectedRoute requiredRole="admin">
    <AdminPanel />
  </ProtectedRoute>
} />
```

### useAuth Hook

```jsx
// hooks/useAuth.js
import { createContext, useContext, useState, useEffect } from 'react'

const AuthContext = createContext(null)

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    // ตรวจสอบ token จาก localStorage
    const token = localStorage.getItem('token')
    if (token) {
      // ดึงข้อมูล user จาก API
      fetch('/api/me', {
        headers: { Authorization: `Bearer ${token}` }
      })
        .then(res => res.json())
        .then(data => {
          setUser(data)
          setIsLoading(false)
        })
        .catch(() => {
          localStorage.removeItem('token')
          setIsLoading(false)
        })
    } else {
      setIsLoading(false)
    }
  }, [])

  const login = async (credentials) => {
    const res = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
      headers: { 'Content-Type': 'application/json' }
    })
    const data = await res.json()
    localStorage.setItem('token', data.token)
    setUser(data.user)
  }

  const logout = () => {
    localStorage.removeItem('token')
    setUser(null)
  }

  return (
    <AuthContext.Provider value={{ user, isLoading, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}

export function useAuth() {
  return useContext(AuthContext)
}
```

---

## Step 516-520: 404 Not Found Page

```jsx
// pages/NotFound.jsx
import { Link, useNavigate } from 'react-router-dom'

function NotFound() {
  const navigate = useNavigate()

  return (
    <div style={{
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center',
      minHeight: '60vh',
      textAlign: 'center',
    }}>
      <h1 style={{ fontSize: '6rem', margin: 0 }}>404</h1>
      <h2>ไม่พบหน้าที่คุณต้องการ</h2>
      <p>หน้านี้อาจถูกลบหรือย้ายไปแล้ว</p>

      <div style={{ display: 'flex', gap: '1rem', marginTop: '1rem' }}>
        <button onClick={() => navigate(-1)}>
          ย้อนกลับ
        </button>
        <Link to="/">
          <button>กลับหน้าแรก</button>
        </Link>
      </div>
    </div>
  )
}

export default NotFound
```

```jsx
// App.jsx - wildcard route ต้องอยู่ล่างสุด
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  {/* ... routes อื่นๆ ... */}
  <Route path="*" element={<NotFound />} />  {/* 404 */}
</Routes>
```

---

## Step 521-530: ตัวอย่าง Multi-page App ครบสมบูรณ์

### โครงสร้างโปรเจกต์

```
src/
├── components/
│   ├── Navbar.jsx
│   ├── ProtectedRoute.jsx
│   └── LoadingSpinner.jsx
├── context/
│   └── AuthContext.jsx
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Login.jsx
│   ├── Register.jsx
│   ├── Dashboard.jsx
│   ├── Products.jsx
│   ├── ProductDetail.jsx
│   └── NotFound.jsx
├── hooks/
│   └── useAuth.js
├── App.jsx
└── main.jsx
```

### App.jsx สมบูรณ์

```jsx
import { Routes, Route } from 'react-router-dom'
import { AuthProvider } from './context/AuthContext'
import Navbar from './components/Navbar'
import ProtectedRoute from './components/ProtectedRoute'
import Home from './pages/Home'
import About from './pages/About'
import Login from './pages/Login'
import Register from './pages/Register'
import Dashboard from './pages/Dashboard'
import DashboardHome from './pages/DashboardHome'
import DashboardProfile from './pages/DashboardProfile'
import DashboardSettings from './pages/DashboardSettings'
import Products from './pages/Products'
import ProductDetail from './pages/ProductDetail'
import NotFound from './pages/NotFound'

function App() {
  return (
    <AuthProvider>
      <div className="app">
        <Navbar />
        <main className="main-content">
          <Routes>
            {/* Public Routes */}
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/login" element={<Login />} />
            <Route path="/register" element={<Register />} />
            <Route path="/products" element={<Products />} />
            <Route path="/products/:id" element={<ProductDetail />} />

            {/* Protected Routes - Nested */}
            <Route path="/dashboard" element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }>
              <Route index element={<DashboardHome />} />
              <Route path="profile" element={<DashboardProfile />} />
              <Route path="settings" element={<DashboardSettings />} />
            </Route>

            {/* 404 */}
            <Route path="*" element={<NotFound />} />
          </Routes>
        </main>
      </div>
    </AuthProvider>
  )
}

export default App
```

### Navbar สมบูรณ์

```jsx
// components/Navbar.jsx
import { NavLink, useNavigate } from 'react-router-dom'
import { useAuth } from '../hooks/useAuth'

function Navbar() {
  const { user, logout } = useAuth()
  const navigate = useNavigate()

  const handleLogout = () => {
    logout()
    navigate('/')
  }

  const activeLinkStyle = ({ isActive }) => ({
    color: isActive ? '#007bff' : '#333',
    textDecoration: 'none',
    fontWeight: isActive ? 'bold' : 'normal',
    padding: '0.5rem',
  })

  return (
    <nav style={{
      display: 'flex',
      justifyContent: 'space-between',
      alignItems: 'center',
      padding: '1rem 2rem',
      backgroundColor: 'white',
      boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
    }}>
      <NavLink to="/" style={{ fontSize: '1.5rem', fontWeight: 'bold', textDecoration: 'none', color: '#333' }}>
        MyShop
      </NavLink>

      <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
        <NavLink to="/" style={activeLinkStyle} end>หน้าแรก</NavLink>
        <NavLink to="/products" style={activeLinkStyle}>สินค้า</NavLink>
        <NavLink to="/about" style={activeLinkStyle}>เกี่ยวกับ</NavLink>

        {user ? (
          <>
            <NavLink to="/dashboard" style={activeLinkStyle}>Dashboard</NavLink>
            <span>สวัสดี, {user.name}</span>
            <button onClick={handleLogout} style={{
              padding: '0.5rem 1rem',
              backgroundColor: '#dc3545',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
            }}>
              ออกจากระบบ
            </button>
          </>
        ) : (
          <>
            <NavLink to="/login" style={activeLinkStyle}>เข้าสู่ระบบ</NavLink>
            <NavLink to="/register" style={{
              ...activeLinkStyle({ isActive: false }),
              backgroundColor: '#007bff',
              color: 'white',
              padding: '0.5rem 1rem',
              borderRadius: '4px',
            }}>
              สมัครสมาชิก
            </NavLink>
          </>
        )}
      </div>
    </nav>
  )
}

export default Navbar
```

### Products Page กับ Search + Pagination

```jsx
// pages/Products.jsx
import { useState } from 'react'
import { useNavigate, useSearchParams } from 'react-router-dom'

const MOCK_PRODUCTS = Array.from({ length: 20 }, (_, i) => ({
  id: i + 1,
  name: `สินค้า ${i + 1}`,
  price: (i + 1) * 100,
  category: i % 2 === 0 ? 'อิเล็กทรอนิกส์' : 'เสื้อผ้า',
}))

function Products() {
  const navigate = useNavigate()
  const [searchParams, setSearchParams] = useSearchParams()
  
  const search = searchParams.get('q') || ''
  const page = Number(searchParams.get('page') || 1)
  const ITEMS_PER_PAGE = 6

  const filtered = MOCK_PRODUCTS.filter(p => 
    p.name.includes(search) || p.category.includes(search)
  )
  
  const totalPages = Math.ceil(filtered.length / ITEMS_PER_PAGE)
  const currentItems = filtered.slice((page - 1) * ITEMS_PER_PAGE, page * ITEMS_PER_PAGE)

  return (
    <div style={{ padding: '2rem' }}>
      <h1>สินค้าทั้งหมด</h1>

      <input
        type="text"
        value={search}
        onChange={e => setSearchParams({ q: e.target.value, page: '1' })}
        placeholder="ค้นหา..."
        style={{ padding: '0.5rem', marginBottom: '1rem', width: '100%' }}
      />

      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '1rem' }}>
        {currentItems.map(product => (
          <div
            key={product.id}
            onClick={() => navigate(`/products/${product.id}`)}
            style={{
              border: '1px solid #ddd',
              borderRadius: '8px',
              padding: '1rem',
              cursor: 'pointer',
            }}
          >
            <h3>{product.name}</h3>
            <p>{product.category}</p>
            <p style={{ fontWeight: 'bold' }}>฿{product.price}</p>
          </div>
        ))}
      </div>

      <div style={{ display: 'flex', justifyContent: 'center', gap: '0.5rem', marginTop: '2rem' }}>
        {Array.from({ length: totalPages }, (_, i) => (
          <button
            key={i + 1}
            onClick={() => setSearchParams(prev => { prev.set('page', String(i + 1)); return prev })}
            style={{
              padding: '0.5rem 1rem',
              backgroundColor: page === i + 1 ? '#007bff' : 'white',
              color: page === i + 1 ? 'white' : 'black',
              border: '1px solid #007bff',
              borderRadius: '4px',
              cursor: 'pointer',
            }}
          >
            {i + 1}
          </button>
        ))}
      </div>
    </div>
  )
}

export default Products
```

---

## Tips และ Best Practices

### 1. ใช้ index routes สำหรับ default content

```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route index element={<DashboardHome />} /> {/* /dashboard */}
  <Route path="profile" element={<Profile />} /> {/* /dashboard/profile */}
</Route>
```

### 2. ใช้ relative paths ใน nested routes

```jsx
// ใน Dashboard component
<NavLink to="profile">โปรไฟล์</NavLink>  // relative: /dashboard/profile
<NavLink to="/profile">โปรไฟล์</NavLink> // absolute: /profile
```

### 3. Handle loading state ใน Protected Routes

```jsx
function ProtectedRoute({ children }) {
  const { user, isLoading } = useAuth()

  if (isLoading) {
    return <LoadingSpinner />  // อย่า render ก่อน check เสร็จ
  }

  if (!user) {
    return <Navigate to="/login" replace />
  }

  return children
}
```

### 4. ใช้ useLocation เพื่อ debug routing

```jsx
import { useLocation } from 'react-router-dom'

function DebugRouter() {
  const location = useLocation()

  if (process.env.NODE_ENV === 'development') {
    console.log('Current route:', location.pathname)
  }

  return null
}
```

---

## Quiz - Part 21

**ข้อ 1**: อะไรคือความแตกต่างระหว่าง `Link` และ `NavLink`?
- a) NavLink ทำให้หน้า reload
- b) NavLink มี isActive state และ style พิเศษเมื่อ active
- c) Link ไม่ work ใน v6
- d) ไม่มีความแตกต่าง

**ข้อ 2**: Hook ใดใช้ดึงค่า `:id` จาก URL `/products/:id`?
- a) useLocation
- b) useSearchParams
- c) useParams
- d) useNavigate

**ข้อ 3**: ใน Nested Routes Component ใดใช้ render child route?
- a) `<Children />`
- b) `<Route />`
- c) `<Outlet />`
- d) `<Navigate />`

**ข้อ 4**: URL `/products?page=2&sort=price` จะใช้ hook ใดดึงค่า?
- a) useParams
- b) useLocation
- c) useSearchParams
- d) useNavigate

**คำตอบ**: 1-b, 2-c, 3-c, 4-c

---

## สรุป Part 21

ใน Part นี้คุณได้เรียนรู้:
- ✅ การตั้งค่า React Router v6
- ✅ การใช้ BrowserRouter, Routes, Route
- ✅ Link, NavLink, Navigate
- ✅ useNavigate, useParams, useSearchParams, useLocation
- ✅ Nested Routes กับ Outlet
- ✅ Protected Routes
- ✅ 404 Not Found Page
- ✅ ตัวอย่าง Multi-page App สมบูรณ์

---

## Part ถัดไป

➡️ **[Part 22: Advanced Routing](./part-22-advanced-routing.md)**
- Lazy Loading Routes
- Code Splitting
- Route Guards
- Dynamic Routes
- Scroll Restoration
