# Part 27: Zustand State Management - ทางเลือกที่เบากว่า Redux

**Step 701-730** | ระดับ: ปานกลาง | เวลาเรียน: 2-3 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [Zustand คืออะไร?](#zustand-คืออะไร)
2. [Zustand vs Redux](#zustand-vs-redux)
3. [การติดตั้ง](#การติดตั้ง)
4. [Create Store พื้นฐาน](#create-store-พื้นฐาน)
5. [Actions ใน Zustand](#actions-ใน-zustand)
6. [Async Actions](#async-actions)
7. [DevTools](#devtools)
8. [Immer Integration](#immer-integration)
9. [Subscriptions](#subscriptions)
10. [ตัวอย่าง Auth Store](#ตัวอย่าง-auth-store)
11. [Quiz](#quiz)

---

## Step 701-703: Zustand คืออะไร?

Zustand เป็น state management library ที่เบาและง่ายกว่า Redux

### คุณสมบัติของ Zustand

- **เบามาก**: ขนาดเพียง 1.5KB
- **ไม่ต้อง Provider**: ไม่ต้องครอบ component tree ด้วย Provider
- **Hooks-based**: ใช้ hook เรียก state โดยตรง
- **TypeScript friendly**: TypeScript support ดีมาก
- **Flexible**: ไม่บังคับ pattern ใดๆ

---

## Step 704-705: Zustand vs Redux

| Feature | Zustand | Redux Toolkit |
|---------|---------|---------------|
| Bundle size | ~1.5KB | ~11KB |
| Setup | ง่ายมาก | ปานกลาง |
| Boilerplate | น้อย | ปานกลาง |
| DevTools | ✅ | ✅ |
| Middleware | ✅ | ✅ |
| RTK Query | ❌ | ✅ |
| Time-travel debugging | ❌ | ✅ |
| Learning curve | ต่ำ | ปานกลาง |
| เหมาะกับ | Apps ขนาดกลาง | Apps ขนาดใหญ่ |

### เมื่อไหร่ใช้ Zustand?

```
✅ Zustand เหมาะเมื่อ:
- ต้องการ setup เร็ว
- Team เล็ก / Project ขนาดกลาง
- ไม่ต้องการ complex async logic
- ต้องการ bundle size เล็ก

✅ Redux Toolkit เหมาะเมื่อ:
- Project ใหญ่ ทีมใหญ่
- ต้องการ RTK Query สำหรับ API
- ต้องการ Time-travel debugging
- มี complex middleware
```

---

## Step 706-707: การติดตั้ง

```bash
npm install zustand
```

---

## Step 708-712: Create Store พื้นฐาน

### Counter Store

```jsx
// stores/counterStore.js
import { create } from 'zustand'

const useCounterStore = create((set, get) => ({
  // State
  count: 0,
  step: 1,

  // Actions
  increment: () => set(state => ({ count: state.count + state.step })),
  decrement: () => set(state => ({ count: state.count - state.step })),
  reset: () => set({ count: 0 }),
  setStep: (newStep) => set({ step: newStep }),
  
  // Action ที่ใช้ get() เพื่ออ่าน state ปัจจุบัน
  incrementDouble: () => {
    const { count, step } = get()
    set({ count: count + step * 2 })
  },
}))

export default useCounterStore
```

### การใช้งานใน Component

```jsx
// components/Counter.jsx
import useCounterStore from '../stores/counterStore'

function Counter() {
  // ดึงเฉพาะที่ต้องการ - component จะ re-render เมื่อค่านั้นเปลี่ยน
  const count = useCounterStore(state => state.count)
  const step = useCounterStore(state => state.step)
  const { increment, decrement, reset, setStep } = useCounterStore()

  return (
    <div>
      <h2>Count: {count}</h2>
      <div>
        <button onClick={decrement}>-</button>
        <button onClick={increment}>+</button>
        <button onClick={reset}>Reset</button>
      </div>
      <div>
        <label>Step: </label>
        <input
          type="number"
          value={step}
          onChange={e => setStep(Number(e.target.value))}
          min={1}
        />
      </div>
    </div>
  )
}
```

### Slice Pattern (organize state)

```jsx
// stores/index.js - รวม multiple stores หรือใช้ slices
import { create } from 'zustand'

// แยก state เป็น slices
const createCounterSlice = (set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
})

const createUserSlice = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
})

// รวม slices
const useStore = create((...args) => ({
  ...createCounterSlice(...args),
  ...createUserSlice(...args),
}))

export default useStore
```

---

## Step 713-717: Actions ใน Zustand

### Pattern ต่างๆ ของ Actions

```jsx
// stores/todoStore.js
import { create } from 'zustand'

let nextId = 1

const useTodoStore = create((set, get) => ({
  todos: [],
  filter: 'all',  // 'all' | 'active' | 'completed'

  // Simple action
  setFilter: (filter) => set({ filter }),

  // Action กับ state transform
  addTodo: (text) => set(state => ({
    todos: [...state.todos, {
      id: nextId++,
      text,
      completed: false,
      createdAt: new Date().toISOString(),
    }]
  })),

  // Action ที่ find และ update
  toggleTodo: (id) => set(state => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),

  // Action ที่ filter ออก
  deleteTodo: (id) => set(state => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),

  // Action ที่ clear completed
  clearCompleted: () => set(state => ({
    todos: state.todos.filter(todo => !todo.completed)
  })),

  // Computed / derived values (ใช้ get())
  getFilteredTodos: () => {
    const { todos, filter } = get()
    switch (filter) {
      case 'active': return todos.filter(t => !t.completed)
      case 'completed': return todos.filter(t => t.completed)
      default: return todos
    }
  },

  // Computed count
  get activeCount() {
    return get().todos.filter(t => !t.completed).length
  },
}))

export default useTodoStore
```

### Todo Component

```jsx
function TodoApp() {
  const [newTodo, setNewTodo] = useState('')
  const todos = useTodoStore(state => state.todos)
  const filter = useTodoStore(state => state.filter)
  const { addTodo, toggleTodo, deleteTodo, setFilter, clearCompleted, getFilteredTodos } = useTodoStore()

  const filteredTodos = getFilteredTodos()
  const activeCount = todos.filter(t => !t.completed).length

  const handleSubmit = (e) => {
    e.preventDefault()
    if (newTodo.trim()) {
      addTodo(newTodo.trim())
      setNewTodo('')
    }
  }

  return (
    <div style={{ maxWidth: '600px', margin: '0 auto', padding: '2rem' }}>
      <h1>Todo List</h1>

      <form onSubmit={handleSubmit} style={{ display: 'flex', gap: '0.5rem', marginBottom: '1rem' }}>
        <input
          value={newTodo}
          onChange={e => setNewTodo(e.target.value)}
          placeholder="เพิ่ม Todo ใหม่..."
          style={{ flex: 1, padding: '0.5rem' }}
        />
        <button type="submit">เพิ่ม</button>
      </form>

      <div style={{ display: 'flex', gap: '0.5rem', marginBottom: '1rem' }}>
        {['all', 'active', 'completed'].map(f => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            style={{
              padding: '0.25rem 0.75rem',
              backgroundColor: filter === f ? '#007bff' : 'white',
              color: filter === f ? 'white' : 'black',
              border: '1px solid #007bff',
              borderRadius: '4px',
            }}
          >
            {f === 'all' ? 'ทั้งหมด' : f === 'active' ? 'ยังไม่เสร็จ' : 'เสร็จแล้ว'}
          </button>
        ))}
      </div>

      <ul style={{ listStyle: 'none', padding: 0 }}>
        {filteredTodos.map(todo => (
          <li key={todo.id} style={{
            display: 'flex',
            alignItems: 'center',
            gap: '0.5rem',
            padding: '0.75rem',
            borderBottom: '1px solid #e2e8f0',
          }}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{
              flex: 1,
              textDecoration: todo.completed ? 'line-through' : 'none',
              color: todo.completed ? '#a0aec0' : 'inherit',
            }}>
              {todo.text}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              style={{ color: 'red', background: 'none', border: 'none', cursor: 'pointer' }}
            >
              ✕
            </button>
          </li>
        ))}
      </ul>

      <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: '1rem', color: '#718096' }}>
        <span>{activeCount} รายการที่ยังไม่เสร็จ</span>
        <button onClick={clearCompleted} style={{ background: 'none', border: 'none', cursor: 'pointer' }}>
          ล้างรายการที่เสร็จแล้ว
        </button>
      </div>
    </div>
  )
}
```

---

## Step 718-721: Async Actions

```jsx
// stores/productStore.js
import { create } from 'zustand'
import { productService } from '../services/productService'

const useProductStore = create((set, get) => ({
  products: [],
  loading: false,
  error: null,

  fetchProducts: async (params) => {
    set({ loading: true, error: null })
    try {
      const data = await productService.getAll(params)
      set({ products: data, loading: false })
    } catch (error) {
      set({ error: error.message, loading: false })
    }
  },

  createProduct: async (data) => {
    set({ loading: true })
    try {
      const newProduct = await productService.create(data)
      set(state => ({
        products: [newProduct, ...state.products],
        loading: false,
      }))
      return newProduct
    } catch (error) {
      set({ error: error.message, loading: false })
      throw error
    }
  },

  updateProduct: async (id, data) => {
    try {
      const updated = await productService.update(id, data)
      set(state => ({
        products: state.products.map(p => p.id === id ? updated : p)
      }))
      return updated
    } catch (error) {
      set({ error: error.message })
      throw error
    }
  },

  deleteProduct: async (id) => {
    try {
      await productService.delete(id)
      set(state => ({
        products: state.products.filter(p => p.id !== id)
      }))
    } catch (error) {
      set({ error: error.message })
      throw error
    }
  },

  clearError: () => set({ error: null }),
}))

export default useProductStore
```

---

## Step 722-724: DevTools

### Redux DevTools Integration

```bash
# ติดตั้ง browser extension ก่อน
# Chrome: Redux DevTools Extension
```

```jsx
// stores/counterStore.js
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'

const useCounterStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set(state => ({ count: state.count + 1 }), false, 'increment'),
      decrement: () => set(state => ({ count: state.count - 1 }), false, 'decrement'),
      reset: () => set({ count: 0 }, false, 'reset'),
    }),
    { name: 'Counter Store' }
  )
)
```

---

## Step 725-726: Immer Integration

Immer ทำให้เขียน state updates แบบ mutable ได้

```bash
npm install immer
```

```jsx
// stores/cartStore.js
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

const useCartStore = create(
  immer((set) => ({
    items: [],
    
    addItem: (product) => set(state => {
      // เขียน mutable โดยตรงได้เลย (Immer จัดการให้)
      const existing = state.items.find(i => i.id === product.id)
      if (existing) {
        existing.quantity++
      } else {
        state.items.push({ ...product, quantity: 1 })
      }
    }),

    removeItem: (id) => set(state => {
      state.items = state.items.filter(i => i.id !== id)
    }),

    updateQuantity: (id, qty) => set(state => {
      const item = state.items.find(i => i.id === id)
      if (item) {
        item.quantity = qty
      }
    }),

    clearCart: (state) => {
      state.items = []
    },
  }))
)
```

---

## Step 727-728: Subscriptions

### Subscribe ภายนอก Component

```jsx
// ฟัง state changes ภายนอก React
import useCounterStore from './stores/counterStore'

const unsubscribe = useCounterStore.subscribe(
  (state) => state.count,  // selector
  (count, prevCount) => {
    console.log(`Count changed: ${prevCount} → ${count}`)
    // ทำอะไรบางอย่างเมื่อ count เปลี่ยน
  }
)

// ยกเลิก subscription
// unsubscribe()
```

### Persist Middleware (Local Storage)

```jsx
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'

const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light',
      language: 'th',
      fontSize: 16,

      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      setFontSize: (size) => set({ fontSize: size }),
    }),
    {
      name: 'app-settings',  // localStorage key
      storage: createJSONStorage(() => localStorage),
      // เลือก fields ที่จะ persist
      partialize: (state) => ({
        theme: state.theme,
        language: state.language,
        fontSize: state.fontSize,
      }),
    }
  )
)
```

---

## Step 729-730: ตัวอย่าง Auth Store

```jsx
// stores/authStore.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import { authService } from '../services/authService'

const useAuthStore = create(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isLoading: false,
      error: null,

      // Computed
      get isAuthenticated() {
        return !!get().token
      },
      get isAdmin() {
        return get().user?.role === 'admin'
      },

      // Actions
      login: async (credentials) => {
        set({ isLoading: true, error: null })
        try {
          const { user, token } = await authService.login(credentials)
          set({ user, token, isLoading: false })
          return user
        } catch (error) {
          set({
            error: error.response?.data?.message || 'เข้าสู่ระบบไม่สำเร็จ',
            isLoading: false,
          })
          throw error
        }
      },

      register: async (data) => {
        set({ isLoading: true, error: null })
        try {
          const { user, token } = await authService.register(data)
          set({ user, token, isLoading: false })
          return user
        } catch (error) {
          set({
            error: error.response?.data?.message || 'สมัครสมาชิกไม่สำเร็จ',
            isLoading: false,
          })
          throw error
        }
      },

      logout: async () => {
        try {
          await authService.logout()
        } finally {
          set({ user: null, token: null, error: null })
        }
      },

      updateProfile: async (data) => {
        set({ isLoading: true })
        try {
          const updatedUser = await authService.updateProfile(data)
          set(state => ({
            user: { ...state.user, ...updatedUser },
            isLoading: false,
          }))
          return updatedUser
        } catch (error) {
          set({ error: error.message, isLoading: false })
          throw error
        }
      },

      clearError: () => set({ error: null }),
    }),
    {
      name: 'auth',
      partialize: (state) => ({
        user: state.user,
        token: state.token,
      }),
    }
  )
)

export default useAuthStore
```

### การใช้งาน Auth Store

```jsx
// ใน Login Component
import useAuthStore from '../stores/authStore'
import { useNavigate } from 'react-router-dom'

function LoginPage() {
  const [form, setForm] = useState({ email: '', password: '' })
  const { login, isLoading, error } = useAuthStore()
  const navigate = useNavigate()

  const handleSubmit = async (e) => {
    e.preventDefault()
    try {
      await login(form)
      navigate('/dashboard')
    } catch {
      // error จะถูก set ใน store แล้ว
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={form.email}
        onChange={e => setForm(f => ({ ...f, email: e.target.value }))}
        placeholder="อีเมล"
      />
      <input
        type="password"
        value={form.password}
        onChange={e => setForm(f => ({ ...f, password: e.target.value }))}
        placeholder="รหัสผ่าน"
      />
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
      </button>
    </form>
  )
}

// ใน Navbar
function Navbar() {
  const user = useAuthStore(state => state.user)
  const logout = useAuthStore(state => state.logout)
  const navigate = useNavigate()

  const handleLogout = async () => {
    await logout()
    navigate('/')
  }

  return (
    <nav>
      {user ? (
        <>
          <span>สวัสดี, {user.name}</span>
          <button onClick={handleLogout}>ออกจากระบบ</button>
        </>
      ) : (
        <Link to="/login">เข้าสู่ระบบ</Link>
      )}
    </nav>
  )
}
```

---

## Tips และ Best Practices

### 1. แยก stores ตาม domain

```
stores/
├── authStore.js      # Authentication
├── cartStore.js      # Shopping cart
├── uiStore.js        # UI state (modal, sidebar)
├── productStore.js   # Products
└── notificationStore.js  # Notifications
```

### 2. อย่า store ทุกอย่างใน global store

```jsx
// ❌ ไม่ดี - form state ควรอยู่ใน local
const useFormStore = create(set => ({
  email: '',
  password: '',
  // ...
}))

// ✅ ดี - ใช้ useState สำหรับ local form state
function LoginForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
}
```

### 3. ใช้ shallow comparison

```jsx
import { shallow } from 'zustand/shallow'

// ✅ ดี - ดึงหลาย values โดยไม่ re-render ถ้าไม่เปลี่ยน
const { count, step } = useCounterStore(
  state => ({ count: state.count, step: state.step }),
  shallow
)
```

---

## Quiz - Part 27

**ข้อ 1**: Zustand ต้องการ Provider ครอบ App ไหม?
- a) ต้องการเสมอ
- b) ไม่ต้องการ
- c) ต้องการสำหรับ async actions
- d) ขึ้นอยู่กับ version

**ข้อ 2**: `get()` ใน Zustand store ใช้ทำอะไร?
- a) ดึงข้อมูลจาก API
- b) อ่าน current state จากภายใน store
- c) Reset state
- d) Subscribe to changes

**ข้อ 3**: Persist middleware ใน Zustand ทำอะไร?
- a) ส่ง state ไป server
- b) บันทึก state ลง localStorage/sessionStorage
- c) Sync state ระหว่าง tabs
- d) Cache API responses

**ข้อ 4**: `shallow` ใน Zustand ใช้ทำอะไร?
- a) ทำ deep comparison
- b) ป้องกัน unnecessary re-renders เมื่อดึงหลาย values
- c) Clone state
- d) Freeze state

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 27

ใน Part นี้คุณได้เรียนรู้:
- ✅ Zustand คืออะไรและเปรียบกับ Redux อย่างไร
- ✅ Create Store พื้นฐาน
- ✅ Actions (Sync และ Async)
- ✅ DevTools integration
- ✅ Immer middleware
- ✅ Subscriptions และ Persist
- ✅ Auth Store ครบสมบูรณ์

---

## Part ถัดไป

➡️ **[Part 28: TanStack Query (React Query)](./part-28-react-query.md)**
- QueryClient Setup
- useQuery, useMutation
- Caching, Infinite Queries
- Optimistic Updates
