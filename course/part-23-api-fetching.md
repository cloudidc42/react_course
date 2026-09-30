# Part 23: API Fetching - การดึงข้อมูลจาก API

**Step 561-595** | ระดับ: ปานกลาง | เวลาเรียน: 3-4 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [Fetch API พื้นฐาน](#fetch-api-พื้นฐาน)
2. [Axios](#axios)
3. [Loading, Error, Success States](#loading-error-success-states)
4. [useEffect + Fetch Pattern](#useeffect--fetch-pattern)
5. [Async/Await ใน Components](#asyncawait-ใน-components)
6. [AbortController - Cancel Requests](#abortcontroller---cancel-requests)
7. [Base URL Configuration](#base-url-configuration)
8. [Axios Interceptors](#axios-interceptors)
9. [Error Handling Strategies](#error-handling-strategies)
10. [Custom Hooks สำหรับ API](#custom-hooks-สำหรับ-api)
11. [Quiz](#quiz)

---

## Step 561-565: Fetch API พื้นฐาน

### GET Request

```jsx
// วิธีพื้นฐาน
fetch('https://jsonplaceholder.typicode.com/posts')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error))

// แบบ async/await
async function getPosts() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts')
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    const data = await response.json()
    return data
  } catch (error) {
    console.error('Error:', error)
    throw error
  }
}
```

### POST Request

```jsx
async function createPost(postData) {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${localStorage.getItem('token')}`,
    },
    body: JSON.stringify(postData),
  })

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`)
  }

  return response.json()
}
```

### PUT, PATCH, DELETE

```jsx
// PUT - แทนที่ทั้งหมด
async function updatePost(id, data) {
  const res = await fetch(`/api/posts/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  })
  return res.json()
}

// PATCH - อัพเดทบางส่วน
async function patchPost(id, data) {
  const res = await fetch(`/api/posts/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  })
  return res.json()
}

// DELETE
async function deletePost(id) {
  const res = await fetch(`/api/posts/${id}`, {
    method: 'DELETE',
  })
  return res.ok
}
```

---

## Step 566-570: Axios

### การติดตั้ง

```bash
npm install axios
```

### Axios เทียบกับ Fetch

| Feature | Fetch | Axios |
|---------|-------|-------|
| Response JSON | `res.json()` | `res.data` (auto) |
| Error on 4xx/5xx | ต้องเช็ค `res.ok` | throw error อัตโนมัติ |
| Request Cancel | AbortController | CancelToken / AbortController |
| Interceptors | ไม่มี | มี |
| Base URL | ต้องกำหนดเอง | `baseURL` option |
| Progress | ไม่มี built-in | มี `onUploadProgress` |

### Axios พื้นฐาน

```jsx
import axios from 'axios'

// GET
const response = await axios.get('/api/posts')
console.log(response.data)  // ข้อมูล JSON อยู่ใน .data

// POST
const response = await axios.post('/api/posts', {
  title: 'My Post',
  body: 'Post content',
})

// PUT
const response = await axios.put(`/api/posts/${id}`, data)

// PATCH
const response = await axios.patch(`/api/posts/${id}`, data)

// DELETE
await axios.delete(`/api/posts/${id}`)
```

### Axios Instance

```jsx
// utils/axios.js
import axios from 'axios'

const axiosInstance = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
})

export default axiosInstance
```

---

## Step 571-575: Loading, Error, Success States

### Pattern พื้นฐาน

```jsx
function PostList() {
  const [posts, setPosts] = useState([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    fetch('/api/posts')
      .then(res => {
        if (!res.ok) throw new Error('Network response was not ok')
        return res.json()
      })
      .then(data => {
        setPosts(data)
        setLoading(false)
      })
      .catch(err => {
        setError(err.message)
        setLoading(false)
      })
  }, [])

  if (loading) return <div>กำลังโหลด...</div>
  if (error) return <div>เกิดข้อผิดพลาด: {error}</div>

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### State Machine Pattern

```jsx
// ดีกว่า: ใช้ state machine แทน multiple states
function PostList() {
  const [state, setState] = useState({
    status: 'idle',  // 'idle' | 'loading' | 'success' | 'error'
    data: null,
    error: null,
  })

  const fetchPosts = async () => {
    setState({ status: 'loading', data: null, error: null })

    try {
      const res = await fetch('/api/posts')
      const data = await res.json()
      setState({ status: 'success', data, error: null })
    } catch (error) {
      setState({ status: 'idle', data: null, error: error.message })
    }
  }

  useEffect(() => {
    fetchPosts()
  }, [])

  switch (state.status) {
    case 'loading':
      return <LoadingSpinner />
    case 'error':
      return (
        <div>
          <p>เกิดข้อผิดพลาด: {state.error}</p>
          <button onClick={fetchPosts}>ลองอีกครั้ง</button>
        </div>
      )
    case 'success':
      return (
        <ul>
          {state.data.map(post => (
            <li key={post.id}>{post.title}</li>
          ))}
        </ul>
      )
    default:
      return null
  }
}
```

### UI Components สำหรับแต่ละ State

```jsx
// components/LoadingSpinner.jsx
function LoadingSpinner({ size = 40, color = '#007bff' }) {
  return (
    <div style={{ display: 'flex', justifyContent: 'center', padding: '2rem' }}>
      <div style={{
        width: size,
        height: size,
        border: `4px solid #f3f3f3`,
        borderTop: `4px solid ${color}`,
        borderRadius: '50%',
        animation: 'spin 1s linear infinite',
      }} />
      <style>{`
        @keyframes spin {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
        }
      `}</style>
    </div>
  )
}

// components/ErrorMessage.jsx
function ErrorMessage({ message, onRetry }) {
  return (
    <div style={{
      backgroundColor: '#fff5f5',
      border: '1px solid #fc8181',
      borderRadius: '8px',
      padding: '1.5rem',
      textAlign: 'center',
    }}>
      <div style={{ fontSize: '2rem' }}>⚠️</div>
      <h3 style={{ color: '#c53030' }}>เกิดข้อผิดพลาด</h3>
      <p style={{ color: '#742a2a' }}>{message}</p>
      {onRetry && (
        <button
          onClick={onRetry}
          style={{
            marginTop: '1rem',
            padding: '0.5rem 1.5rem',
            backgroundColor: '#e53e3e',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: 'pointer',
          }}
        >
          ลองอีกครั้ง
        </button>
      )}
    </div>
  )
}

// components/EmptyState.jsx
function EmptyState({ message = 'ไม่มีข้อมูล', icon = '📭' }) {
  return (
    <div style={{
      textAlign: 'center',
      padding: '3rem',
      color: '#718096',
    }}>
      <div style={{ fontSize: '3rem', marginBottom: '1rem' }}>{icon}</div>
      <p>{message}</p>
    </div>
  )
}
```

---

## Step 576-580: useEffect + Fetch Pattern

### Pattern ที่ถูกต้อง

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    let isMounted = true  // ป้องกัน state update หลัง unmount

    const fetchUser = async () => {
      try {
        setLoading(true)
        setError(null)
        
        const res = await fetch(`/api/users/${userId}`)
        if (!res.ok) throw new Error('User not found')
        
        const data = await res.json()
        
        if (isMounted) {  // เช็คก่อน update state
          setUser(data)
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message)
        }
      } finally {
        if (isMounted) {
          setLoading(false)
        }
      }
    }

    fetchUser()

    return () => {
      isMounted = false  // cleanup function
    }
  }, [userId])  // re-fetch เมื่อ userId เปลี่ยน

  if (loading) return <LoadingSpinner />
  if (error) return <ErrorMessage message={error} />
  if (!user) return null

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  )
}
```

### Fetch กับ Dependencies

```jsx
function SearchResults({ query, page, sortBy }) {
  const [results, setResults] = useState([])
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    if (!query) {
      setResults([])
      return
    }

    const fetchResults = async () => {
      setLoading(true)
      try {
        const params = new URLSearchParams({ q: query, page, sort: sortBy })
        const res = await fetch(`/api/search?${params}`)
        const data = await res.json()
        setResults(data.results)
      } finally {
        setLoading(false)
      }
    }

    fetchResults()
  }, [query, page, sortBy])  // re-fetch เมื่อ params เปลี่ยน

  return (
    <div>
      {loading ? <LoadingSpinner /> : (
        results.length === 0 
          ? <EmptyState message={`ไม่พบผลลัพธ์สำหรับ "${query}"`} />
          : results.map(result => <ResultCard key={result.id} result={result} />)
      )}
    </div>
  )
}
```

---

## Step 581-583: AbortController - Cancel Requests

AbortController ใช้ยกเลิก request ที่ค้างอยู่

### ปัญหาที่ต้องแก้

```
1. User พิมพ์เร็ว → เรียก API หลายครั้ง
2. Response อาจกลับมาไม่เป็นลำดับ
3. Component unmount ก่อน response กลับ → memory leak

การแก้: ยกเลิก request เก่าก่อนส่งใหม่
```

### การใช้ AbortController กับ Fetch

```jsx
function SearchBox() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    if (!query) {
      setResults([])
      return
    }

    const controller = new AbortController()

    const search = async () => {
      setLoading(true)
      try {
        const res = await fetch(`/api/search?q=${query}`, {
          signal: controller.signal  // ส่ง signal ไปกับ request
        })
        const data = await res.json()
        setResults(data)
      } catch (err) {
        if (err.name === 'AbortError') {
          console.log('Request cancelled')  // ถูกยกเลิก - ไม่ต้อง error
        } else {
          console.error('Fetch error:', err)
        }
      } finally {
        setLoading(false)
      }
    }

    search()

    return () => {
      controller.abort()  // ยกเลิก request เมื่อ cleanup
    }
  }, [query])

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหา..."
      />
      {loading && <span>กำลังค้นหา...</span>}
      <ul>
        {results.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  )
}
```

### AbortController กับ Axios

```jsx
import axios from 'axios'

function DataFetcher({ id }) {
  const [data, setData] = useState(null)

  useEffect(() => {
    const controller = new AbortController()

    axios.get(`/api/data/${id}`, {
      signal: controller.signal
    })
    .then(res => setData(res.data))
    .catch(err => {
      if (axios.isCancel(err)) {
        console.log('Request cancelled')
      }
    })

    return () => controller.abort()
  }, [id])

  return <div>{data?.name}</div>
}
```

---

## Step 584-587: Base URL Configuration

### Environment Variables

```bash
# .env.development
VITE_API_URL=http://localhost:3000/api

# .env.production
VITE_API_URL=https://api.example.com
```

### Axios Instance พร้อม Config สมบูรณ์

```jsx
// utils/api.js
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  timeout: 15000,
  headers: {
    'Content-Type': 'application/json',
  },
})

export default api
```

### API Service Layer

```jsx
// services/productService.js
import api from '../utils/api'

export const productService = {
  getAll: (params) => api.get('/products', { params }),
  
  getById: (id) => api.get(`/products/${id}`),
  
  create: (data) => api.post('/products', data),
  
  update: (id, data) => api.put(`/products/${id}`, data),
  
  delete: (id) => api.delete(`/products/${id}`),
  
  search: (query) => api.get('/products/search', { 
    params: { q: query } 
  }),
}

// services/userService.js
export const userService = {
  getProfile: () => api.get('/users/me'),
  updateProfile: (data) => api.patch('/users/me', data),
  changePassword: (data) => api.post('/users/change-password', data),
}
```

### การใช้งาน Service

```jsx
import { productService } from '../services/productService'

function Products() {
  const [products, setProducts] = useState([])

  useEffect(() => {
    productService.getAll({ page: 1, limit: 10 })
      .then(res => setProducts(res.data))
      .catch(err => console.error(err))
  }, [])

  const handleDelete = async (id) => {
    await productService.delete(id)
    setProducts(prev => prev.filter(p => p.id !== id))
  }

  return (/* JSX */)
}
```

---

## Step 588-591: Axios Interceptors

Interceptors ดักจับ request/response ทุกครั้งก่อนส่ง/รับ

### Request Interceptor

```jsx
// utils/api.js
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
})

// Request Interceptor - เพิ่ม token ทุก request
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// Response Interceptor - จัดการ error กลาง
api.interceptors.response.use(
  (response) => {
    return response
  },
  async (error) => {
    const originalRequest = error.config

    // Token หมดอายุ - ลอง refresh
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true

      try {
        const refreshToken = localStorage.getItem('refreshToken')
        const res = await axios.post('/api/auth/refresh', { refreshToken })
        
        const { token } = res.data
        localStorage.setItem('token', token)
        
        // ส่ง request เดิมอีกครั้งด้วย token ใหม่
        originalRequest.headers.Authorization = `Bearer ${token}`
        return api(originalRequest)
      } catch (refreshError) {
        // Refresh token หมดอายุ - logout
        localStorage.removeItem('token')
        localStorage.removeItem('refreshToken')
        window.location.href = '/login'
        return Promise.reject(refreshError)
      }
    }

    return Promise.reject(error)
  }
)

export default api
```

### Logging Interceptor

```jsx
// เพิ่ม logging ใน development
if (import.meta.env.DEV) {
  api.interceptors.request.use(config => {
    console.log(`📤 ${config.method?.toUpperCase()} ${config.url}`, config.data)
    return config
  })

  api.interceptors.response.use(
    response => {
      console.log(`📥 ${response.status} ${response.config.url}`, response.data)
      return response
    },
    error => {
      console.error(`❌ ${error.response?.status} ${error.config?.url}`, error.response?.data)
      return Promise.reject(error)
    }
  )
}
```

---

## Step 592-595: Error Handling Strategies

### HTTP Error Status Codes

| Status | ความหมาย | การจัดการ |
|--------|---------|----------|
| 400 | Bad Request | แสดง validation errors |
| 401 | Unauthorized | Redirect ไป login |
| 403 | Forbidden | แสดง "ไม่มีสิทธิ์" |
| 404 | Not Found | แสดงหน้า 404 |
| 422 | Validation Error | แสดง field errors |
| 429 | Too Many Requests | แสดง "ลองใหม่อีกครั้ง" |
| 500 | Server Error | แสดง "เกิดข้อผิดพลาดของระบบ" |

### Error Parser

```jsx
// utils/errorParser.js
export function parseApiError(error) {
  // Axios error
  if (error.response) {
    const { status, data } = error.response

    switch (status) {
      case 400:
        return data.message || 'ข้อมูลไม่ถูกต้อง'
      case 401:
        return 'กรุณาเข้าสู่ระบบ'
      case 403:
        return 'คุณไม่มีสิทธิ์ดำเนินการนี้'
      case 404:
        return 'ไม่พบข้อมูลที่ต้องการ'
      case 422:
        if (data.errors) {
          return Object.values(data.errors).flat().join(', ')
        }
        return data.message || 'ข้อมูลไม่ถูกต้อง'
      case 429:
        return 'คุณส่งคำขอมากเกินไป กรุณารอสักครู่'
      case 500:
        return 'เกิดข้อผิดพลาดของระบบ กรุณาลองใหม่ภายหลัง'
      default:
        return 'เกิดข้อผิดพลาดที่ไม่รู้จัก'
    }
  }

  // Network error
  if (error.request) {
    return 'ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้ กรุณาตรวจสอบการเชื่อมต่ออินเทอร์เน็ต'
  }

  return error.message || 'เกิดข้อผิดพลาดที่ไม่รู้จัก'
}
```

### Custom useFetch Hook

```jsx
// hooks/useFetch.js
import { useState, useEffect, useCallback } from 'react'
import { parseApiError } from '../utils/errorParser'

function useFetch(url, options = {}) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)

  const fetchData = useCallback(async () => {
    const controller = new AbortController()
    
    setLoading(true)
    setError(null)

    try {
      const res = await fetch(url, {
        ...options,
        signal: controller.signal,
      })

      if (!res.ok) {
        throw new Error(`HTTP ${res.status}: ${res.statusText}`)
      }

      const json = await res.json()
      setData(json)
    } catch (err) {
      if (err.name !== 'AbortError') {
        setError(parseApiError(err))
      }
    } finally {
      setLoading(false)
    }

    return () => controller.abort()
  }, [url])

  useEffect(() => {
    fetchData()
  }, [fetchData])

  return { data, loading, error, refetch: fetchData }
}

export default useFetch
```

### การใช้งาน useFetch

```jsx
function PostList() {
  const { data: posts, loading, error, refetch } = useFetch('/api/posts')

  if (loading) return <LoadingSpinner />
  if (error) return <ErrorMessage message={error} onRetry={refetch} />
  if (!posts?.length) return <EmptyState message="ไม่มีโพสต์" />

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

---

## Tips และ Best Practices

### 1. ใช้ Service Layer

```jsx
// ✅ ดี - แยก API logic ออกจาก Component
// services/api.js
export const getUser = (id) => api.get(`/users/${id}`)

// Component
const { data } = await getUser(userId)

// ❌ ไม่ดี - API logic ปนอยู่ใน Component
const res = await fetch(`https://api.example.com/users/${userId}`)
```

### 2. อย่าลืม Cleanup

```jsx
// ✅ Always cleanup
useEffect(() => {
  const controller = new AbortController()
  
  fetchData(controller.signal)
  
  return () => controller.abort()
}, [id])
```

### 3. จัดการ Race Conditions

```jsx
// ✅ ใช้ flag ป้องกัน race condition
useEffect(() => {
  let cancelled = false
  
  fetchData().then(data => {
    if (!cancelled) setData(data)
  })
  
  return () => { cancelled = true }
}, [id])
```

### 4. Cache responses เมื่อเหมาะสม

```jsx
const cache = new Map()

async function fetchWithCache(url) {
  if (cache.has(url)) {
    return cache.get(url)
  }
  
  const res = await fetch(url)
  const data = await res.json()
  cache.set(url, data)
  return data
}
```

---

## Quiz - Part 23

**ข้อ 1**: ทำไมต้องใช้ AbortController?
- a) ทำให้ request เร็วขึ้น
- b) ยกเลิก request เมื่อ component unmount หรือค่าเปลี่ยน
- c) เพิ่ม security
- d) ทำให้ code สั้นลง

**ข้อ 2**: Axios interceptor ใช้ทำอะไร?
- a) แสดง UI
- b) ดักจับ request/response ทุกครั้ง
- c) จัดการ routing
- d) เก็บ state

**ข้อ 3**: Response status 401 หมายความว่าอะไร?
- a) Not Found
- b) Server Error
- c) Unauthorized
- d) Bad Request

**ข้อ 4**: `isMounted` flag ใช้ป้องกันอะไร?
- a) Memory leak จาก event listeners
- b) State update หลัง component unmount
- c) Re-render ไม่จำเป็น
- d) Network error

**คำตอบ**: 1-b, 2-b, 3-c, 4-b

---

## สรุป Part 23

ใน Part นี้คุณได้เรียนรู้:
- ✅ Fetch API และ Axios
- ✅ Loading, Error, Success States
- ✅ useEffect + Fetch Pattern ที่ถูกต้อง
- ✅ AbortController สำหรับยกเลิก requests
- ✅ Base URL Configuration
- ✅ Axios Interceptors (Auth, Logging, Error)
- ✅ Error Handling Strategies
- ✅ Custom useFetch Hook

---

## Part ถัดไป

➡️ **[Part 24: Error Handling](./part-24-error-handling.md)**
- Error Boundaries
- Global Error Handling
- Retry Logic
- Sentry Integration
