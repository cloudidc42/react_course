# Part 30: TypeScript + React - Type-safe React Development

**Step 811-855** | ระดับ: สูง | เวลาเรียน: 5-6 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [TypeScript basics สำหรับ React](#typescript-basics-สำหรับ-react)
2. [Component Props Types](#component-props-types)
3. [useState TypeScript](#usestate-typescript)
4. [useRef TypeScript](#useref-typescript)
5. [Event Types](#event-types)
6. [Generic Components](#generic-components)
7. [Utility Types](#utility-types)
8. [API Response Types](#api-response-types)
9. [Type Guards](#type-guards)
10. [Advanced Patterns](#advanced-patterns)
11. [Quiz](#quiz)

---

## Step 811-815: TypeScript basics สำหรับ React

### การตั้งค่า TypeScript กับ Vite

```bash
# สร้างโปรเจกต์ใหม่ด้วย TypeScript
npm create vite@latest my-app -- --template react-ts

# หรือเพิ่ม TypeScript ในโปรเจกต์ที่มีอยู่
npm install --save-dev typescript @types/react @types/react-dom
```

### tsconfig.json ที่แนะนำ

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### TypeScript Types พื้นฐาน

```typescript
// Primitive types
const name: string = 'สมชาย'
const age: number = 25
const isActive: boolean = true
const nothing: null = null
const notDefined: undefined = undefined

// Array
const names: string[] = ['สมชาย', 'สมหญิง']
const ages: Array<number> = [25, 30]

// Tuple
const coordinate: [number, number] = [10, 20]

// Object
const user: {
  name: string
  age: number
  email?: string  // optional
} = {
  name: 'สมชาย',
  age: 25,
}

// Union
let id: string | number = 'abc-123'
id = 123

// Literal
type Theme = 'light' | 'dark' | 'system'
let theme: Theme = 'dark'

// Any - หลีกเลี่ยงถ้าเป็นไปได้
let anything: any = 'could be anything'

// Unknown - ดีกว่า any
let unknown: unknown = 'something'
if (typeof unknown === 'string') {
  console.log(unknown.toUpperCase())  // TypeScript รู้ว่าเป็น string
}
```

---

## Step 816-820: Component Props Types

### Interface vs Type

```typescript
// Interface - ขยายได้ (extends)
interface ButtonProps {
  label: string
  onClick: () => void
  disabled?: boolean
}

// Type - flexible กว่า
type ButtonProps = {
  label: string
  onClick: () => void
  disabled?: boolean
}

// ใช้ interface สำหรับ component props
// ใช้ type สำหรับ union, intersection, computed types
```

### Component Props ต่างๆ

```tsx
// Basic Props
interface UserCardProps {
  name: string
  age: number
  email?: string   // optional
  avatar: string | null
  role: 'admin' | 'user' | 'editor'
}

function UserCard({ name, age, email, avatar, role }: UserCardProps) {
  return (
    <div>
      <h3>{name}</h3>
      <p>อายุ: {age}</p>
      {email && <p>อีเมล: {email}</p>}
      {avatar && <img src={avatar} alt={name} />}
      <span>{role}</span>
    </div>
  )
}

// Children Props
interface ContainerProps {
  children: React.ReactNode  // รับ JSX ใดๆ
  className?: string
}

// หรือ
interface WrapperProps {
  children: React.ReactElement  // รับ single React element เท่านั้น
}

// Function as children
interface RenderProps {
  children: (data: { count: number; increment: () => void }) => React.ReactNode
}

// Event handler props
interface FormProps {
  onSubmit: (data: { email: string; password: string }) => void
  onChange?: (field: string, value: string) => void
  onCancel?: () => void
}

// Style props
interface StyledProps {
  style?: React.CSSProperties
  className?: string
}
```

### Extending HTML Elements

```tsx
// Extend HTML button
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger'
  loading?: boolean
  size?: 'sm' | 'md' | 'lg'
}

function Button({ variant = 'primary', loading, size = 'md', children, ...props }: ButtonProps) {
  const sizeClasses = {
    sm: { padding: '0.25rem 0.75rem', fontSize: '0.875rem' },
    md: { padding: '0.5rem 1rem', fontSize: '1rem' },
    lg: { padding: '0.75rem 1.5rem', fontSize: '1.125rem' },
  }

  const variantClasses = {
    primary: { backgroundColor: '#007bff', color: 'white' },
    secondary: { backgroundColor: '#6c757d', color: 'white' },
    danger: { backgroundColor: '#dc3545', color: 'white' },
  }

  return (
    <button
      style={{
        border: 'none',
        borderRadius: '4px',
        cursor: loading ? 'not-allowed' : 'pointer',
        opacity: loading ? 0.7 : 1,
        ...sizeClasses[size],
        ...variantClasses[variant],
      }}
      disabled={loading}
      {...props}
    >
      {loading ? 'กำลังโหลด...' : children}
    </button>
  )
}

// Extend HTML input
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string
  error?: string
  helperText?: string
}

function Input({ label, error, helperText, id, ...props }: InputProps) {
  return (
    <div>
      {label && <label htmlFor={id}>{label}</label>}
      <input id={id} {...props} />
      {error && <span style={{ color: 'red' }}>{error}</span>}
      {helperText && !error && <span style={{ color: 'gray' }}>{helperText}</span>}
    </div>
  )
}
```

---

## Step 821-825: useState TypeScript

```tsx
import { useState } from 'react'

// TypeScript infer type อัตโนมัติ
const [count, setCount] = useState(0)           // number
const [name, setName] = useState('')             // string
const [isOpen, setIsOpen] = useState(false)      // boolean

// ระบุ type ด้วยตัวเอง
const [user, setUser] = useState<User | null>(null)
const [items, setItems] = useState<Product[]>([])
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle')

// Interface
interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

function UserProfile() {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState<boolean>(false)

  const updateUser = (field: keyof User, value: string | number) => {
    setUser(prev => prev ? { ...prev, [field]: value } : null)
  }

  if (!user) return <div>ไม่มีข้อมูลผู้ใช้</div>

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  )
}
```

---

## Step 826-829: useRef TypeScript

```tsx
import { useRef, useEffect } from 'react'

// DOM element ref
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null)

  useEffect(() => {
    inputRef.current?.focus()  // ? เพราะ initial value เป็น null
  }, [])

  return <input ref={inputRef} type="text" />
}

// Mutable ref (ไม่ trigger re-render)
function Timer() {
  const intervalRef = useRef<number | null>(null)
  const [time, setTime] = useState(0)

  const start = () => {
    intervalRef.current = window.setInterval(() => {
      setTime(t => t + 1)
    }, 1000)
  }

  const stop = () => {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current)
      intervalRef.current = null
    }
  }

  useEffect(() => {
    return () => stop()  // cleanup
  }, [])

  return (
    <div>
      <p>เวลา: {time}s</p>
      <button onClick={start}>เริ่ม</button>
      <button onClick={stop}>หยุด</button>
    </div>
  )
}

// useRef กับ callback ref
function MeasureDiv() {
  const divRef = useRef<HTMLDivElement>(null)
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 })

  useEffect(() => {
    if (divRef.current) {
      const { width, height } = divRef.current.getBoundingClientRect()
      setDimensions({ width, height })
    }
  }, [])

  return (
    <div ref={divRef}>
      ขนาด: {dimensions.width} x {dimensions.height}
    </div>
  )
}
```

---

## Step 830-835: Event Types

```tsx
// Click Events
const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
  event.preventDefault()
  console.log('clicked')
}

// Change Events
const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
  console.log(event.target.value)
}

// สำหรับ select
const handleSelectChange = (event: React.ChangeEvent<HTMLSelectElement>) => {
  console.log(event.target.value)
}

// สำหรับ textarea
const handleTextareaChange = (event: React.ChangeEvent<HTMLTextAreaElement>) => {
  console.log(event.target.value)
}

// Submit Events
const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
  event.preventDefault()
  // access form data
}

// Keyboard Events
const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
  if (event.key === 'Enter') {
    console.log('Enter pressed')
  }
}

// Focus Events
const handleFocus = (event: React.FocusEvent<HTMLInputElement>) => {
  console.log('focused')
}

// Drag Events
const handleDragOver = (event: React.DragEvent<HTMLDivElement>) => {
  event.preventDefault()
}

// ตัวอย่างใน component
function SearchInput() {
  const [query, setQuery] = useState('')

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}  // TypeScript infer type จาก element
      onKeyDown={(e) => {
        if (e.key === 'Enter') {
          handleSearch(query)
        }
      }}
    />
  )
}
```

---

## Step 836-840: Generic Components

Generic components สร้าง reusable components ที่ type-safe

### Generic List

```tsx
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string | number
  emptyMessage?: string
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage = 'ไม่มีข้อมูล' }: ListProps<T>) {
  if (items.length === 0) {
    return <div>{emptyMessage}</div>
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  )
}

// ใช้งาน - TypeScript infer type อัตโนมัติ
<List
  items={products}  // Product[]
  keyExtractor={p => p.id}
  renderItem={product => <span>{product.name}</span>}
/>

<List
  items={users}  // User[]
  keyExtractor={u => u.email}
  renderItem={user => <span>{user.name}</span>}
/>
```

### Generic Select Component

```tsx
interface SelectOption<T> {
  value: T
  label: string
}

interface SelectProps<T> {
  options: SelectOption<T>[]
  value: T | null
  onChange: (value: T) => void
  placeholder?: string
  label?: string
}

function Select<T extends string | number>({
  options,
  value,
  onChange,
  placeholder,
  label,
}: SelectProps<T>) {
  return (
    <div>
      {label && <label>{label}</label>}
      <select
        value={value ?? ''}
        onChange={(e) => {
          const selectedOption = options.find(o => String(o.value) === e.target.value)
          if (selectedOption) onChange(selectedOption.value)
        }}
      >
        {placeholder && <option value="">{placeholder}</option>}
        {options.map(option => (
          <option key={String(option.value)} value={String(option.value)}>
            {option.label}
          </option>
        ))}
      </select>
    </div>
  )
}

// ใช้งาน
<Select<string>
  options={[
    { value: 'admin', label: 'ผู้ดูแลระบบ' },
    { value: 'user', label: 'ผู้ใช้ทั่วไป' },
  ]}
  value={selectedRole}
  onChange={setSelectedRole}
  placeholder="เลือกบทบาท"
/>
```

---

## Step 841-845: Utility Types

TypeScript มี utility types ที่มีประโยชน์มาก

```typescript
interface User {
  id: number
  name: string
  email: string
  password: string
  role: 'admin' | 'user'
  createdAt: Date
}

// Partial<T> - ทุก field เป็น optional
type PartialUser = Partial<User>
// { id?: number; name?: string; email?: string; ... }

// การใช้งาน: สำหรับ update
function updateUser(id: number, updates: Partial<User>) {
  // updates ไม่ต้องมีทุก field
}

// Required<T> - ทุก field เป็น required
type RequiredUser = Required<User>

// Pick<T, K> - เลือกเฉพาะ fields ที่ต้องการ
type UserPublic = Pick<User, 'id' | 'name' | 'email' | 'role'>
// { id: number; name: string; email: string; role: ... }

// Omit<T, K> - ลบ fields ที่ไม่ต้องการออก
type UserWithoutPassword = Omit<User, 'password'>
// ทุก field ของ User ยกเว้น password

// Readonly<T> - ทุก field เป็น readonly
type ImmutableUser = Readonly<User>
const user: ImmutableUser = { ... }
// user.name = 'new name'  // Error! Cannot assign to readonly property

// Record<K, V> - สร้าง object type
type RolePermissions = Record<'admin' | 'user' | 'editor', string[]>
const permissions: RolePermissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read'],
  editor: ['read', 'write'],
}

// ReturnType<T> - ดึง return type จาก function
function getUser() {
  return { id: 1, name: 'สมชาย' }
}
type GetUserReturn = ReturnType<typeof getUser>
// { id: number; name: string }

// Parameters<T> - ดึง parameter types
function createProduct(name: string, price: number, category: string) { }
type CreateProductParams = Parameters<typeof createProduct>
// [name: string, price: number, category: string]

// Exclude<T, U> - ลบ type จาก union
type AllEvents = 'click' | 'hover' | 'focus' | 'blur'
type MouseEvents = Exclude<AllEvents, 'focus' | 'blur'>
// 'click' | 'hover'

// Extract<T, U> - เอาเฉพาะ type ที่ match
type NumberOrString = string | number | boolean
type OnlyNumbers = Extract<NumberOrString, number>
// number

// NonNullable<T> - ลบ null และ undefined
type MaybeUser = User | null | undefined
type DefiniteUser = NonNullable<MaybeUser>
// User
```

---

## Step 846-849: API Response Types

```typescript
// Generic API Response wrapper
interface ApiResponse<T> {
  data: T
  message: string
  success: boolean
  timestamp: string
}

interface PaginatedResponse<T> {
  data: T[]
  pagination: {
    currentPage: number
    totalPages: number
    totalItems: number
    itemsPerPage: number
  }
}

// Product types
interface Product {
  id: number
  name: string
  description: string
  price: number
  stock: number
  category: Category
  images: string[]
  tags: string[]
  createdAt: string
  updatedAt: string
}

interface Category {
  id: number
  name: string
  slug: string
}

// Create types (ไม่มี id และ timestamps)
type CreateProductDTO = Omit<Product, 'id' | 'createdAt' | 'updatedAt' | 'category'> & {
  categoryId: number
}

// Update types (ทุก field optional)
type UpdateProductDTO = Partial<CreateProductDTO>

// API Functions
async function getProducts(params?: {
  page?: number
  limit?: number
  category?: string
  search?: string
}): Promise<PaginatedResponse<Product>> {
  const response = await api.get('/products', { params })
  return response.data
}

async function createProduct(data: CreateProductDTO): Promise<ApiResponse<Product>> {
  const response = await api.post('/products', data)
  return response.data
}

// Hooks
function useProducts(params?: { page?: number; category?: string }) {
  return useQuery<PaginatedResponse<Product>>({
    queryKey: ['products', params],
    queryFn: () => getProducts(params),
  })
}
```

---

## Step 850-852: Type Guards

Type Guards ช่วย TypeScript รู้ว่าเรากำลังทำงานกับ type ใด

```typescript
// typeof guard
function processInput(input: string | number) {
  if (typeof input === 'string') {
    return input.toUpperCase()  // string
  }
  return input.toFixed(2)  // number
}

// instanceof guard
function processError(error: Error | string) {
  if (error instanceof Error) {
    return error.message  // Error object
  }
  return error  // string
}

// in guard
interface AdminUser {
  id: number
  name: string
  adminSecret: string
}

interface RegularUser {
  id: number
  name: string
}

function processUser(user: AdminUser | RegularUser) {
  if ('adminSecret' in user) {
    console.log('Admin:', user.adminSecret)  // AdminUser
  } else {
    console.log('User:', user.name)  // RegularUser
  }
}

// Custom type guard (is predicate)
interface Cat {
  type: 'cat'
  meow: () => void
}

interface Dog {
  type: 'dog'
  bark: () => void
}

function isCat(animal: Cat | Dog): animal is Cat {
  return animal.type === 'cat'
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow()  // TypeScript รู้ว่าเป็น Cat
  } else {
    animal.bark()  // TypeScript รู้ว่าเป็น Dog
  }
}

// Discriminated Union
type LoadingState = { status: 'loading' }
type SuccessState<T> = { status: 'success'; data: T }
type ErrorState = { status: 'error'; error: string }

type FetchState<T> = LoadingState | SuccessState<T> | ErrorState

function renderState<T>(state: FetchState<T>, render: (data: T) => React.ReactNode) {
  switch (state.status) {
    case 'loading':
      return <LoadingSpinner />
    case 'success':
      return render(state.data)  // TypeScript รู้ว่ามี data
    case 'error':
      return <ErrorMessage message={state.error} />  // TypeScript รู้ว่ามี error
  }
}
```

---

## Step 853-855: Advanced Patterns

### Custom Hooks TypeScript

```tsx
// hooks/useLocalStorage.ts
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value
      setStoredValue(valueToStore)
      localStorage.setItem(key, JSON.stringify(valueToStore))
    } catch (error) {
      console.error(error)
    }
  }

  return [storedValue, setValue] as const
}

// ใช้งาน - TypeScript infer type
const [user, setUser] = useLocalStorage<User | null>('user', null)
const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light')
```

### Component Overloading

```tsx
// Polymorphic component
type PolymorphicProps<T extends React.ElementType = 'div'> = {
  as?: T
  children?: React.ReactNode
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>

function Box<T extends React.ElementType = 'div'>({
  as,
  children,
  ...props
}: PolymorphicProps<T>) {
  const Component = as || 'div'
  return <Component {...props}>{children}</Component>
}

// ใช้งาน
<Box>Default div</Box>
<Box as="section" aria-label="main content">Section</Box>
<Box as="button" onClick={() => {}}>Clickable</Box>
```

### Context TypeScript

```tsx
// context/ThemeContext.tsx
interface ThemeContextType {
  theme: 'light' | 'dark'
  toggleTheme: () => void
  setTheme: (theme: 'light' | 'dark') => void
}

// สร้าง context with default undefined
const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')

  const toggleTheme = () => setTheme(t => t === 'light' ? 'dark' : 'light')

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}

// Custom hook พร้อม type check
function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme ต้องใช้ภายใน ThemeProvider')
  }
  return context
}

export { ThemeProvider, useTheme }
```

---

## Tips และ Best Practices

### 1. อย่าใช้ `any` โดยไม่จำเป็น

```typescript
// ❌ ไม่ดี
function processData(data: any) { ... }

// ✅ ดีกว่า - ใช้ unknown แล้ว narrow type
function processData(data: unknown) {
  if (typeof data === 'string') { ... }
}

// ✅ ดีที่สุด - ระบุ type ชัดเจน
function processData(data: Product | User) { ... }
```

### 2. ใช้ `as const` สำหรับ readonly objects

```typescript
const ROUTES = {
  HOME: '/',
  PRODUCTS: '/products',
  DASHBOARD: '/dashboard',
} as const

type Route = typeof ROUTES[keyof typeof ROUTES]
// '/' | '/products' | '/dashboard'
```

### 3. Type assertion เมื่อจำเป็นจริงๆ

```typescript
// ❌ ไม่ดี - force cast
const user = data as User

// ✅ ดีกว่า - validate ก่อน
function isUser(data: unknown): data is User {
  return typeof data === 'object' && data !== null &&
    'id' in data && 'name' in data
}

if (isUser(data)) {
  console.log(data.name)  // safe!
}
```

---

## Quiz - Part 30

**ข้อ 1**: `Partial<T>` ทำอะไร?
- a) ลบ type ที่ไม่ต้องการ
- b) ทำให้ทุก field เป็น optional
- c) สร้าง readonly type
- d) clone type

**ข้อ 2**: `keyof T` ใช้ทำอะไร?
- a) ดึงค่าของ object
- b) ดึง keys ทั้งหมดของ type เป็น union
- c) ลบ key จาก type
- d) เรียงลำดับ keys

**ข้อ 3**: Type Guard `animal is Cat` หมายความว่าอะไร?
- a) animal เป็น Cat เสมอ
- b) function นี้ return boolean ที่บอกว่า animal เป็น Cat ไหม
- c) convert animal ให้เป็น Cat
- d) ตรวจสอบ runtime type เท่านั้น

**ข้อ 4**: `as const` ใช้ทำอะไร?
- a) Cast type
- b) ทำให้ค่าเป็น readonly และ infer literal types
- c) Create constant variable
- d) Prevent re-assignment

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 30

ใน Part นี้คุณได้เรียนรู้:
- ✅ TypeScript setup สำหรับ React
- ✅ Component Props Types (Interface, HTML extension)
- ✅ useState, useRef TypeScript
- ✅ Event Types ทั้งหมด
- ✅ Generic Components ที่ reusable
- ✅ Utility Types (Partial, Pick, Omit, Record, ...)
- ✅ API Response Types
- ✅ Type Guards (typeof, instanceof, in, custom)
- ✅ Advanced Patterns (Polymorphic, Context, Hooks)

---

## สรุป Module 3 (Part 21-30)

คุณได้ครอบคลุมหัวข้อสำคัญ:
- **Part 21**: React Router v6 - Multi-page navigation
- **Part 22**: Advanced Routing - Lazy loading, Guards, Code splitting
- **Part 23**: API Fetching - Fetch, Axios, Interceptors
- **Part 24**: Error Handling - Boundaries, Retry, Sentry
- **Part 25**: Performance - memo, useMemo, Virtualization
- **Part 26**: Redux Toolkit - State management ขนาดใหญ่
- **Part 27**: Zustand - State management แบบเบา
- **Part 28**: TanStack Query - Server state management
- **Part 29**: React Hook Form + Zod - Advanced forms
- **Part 30**: TypeScript + React - Type safety

---

## Part ถัดไป

➡️ **Module 4: Next.js Development**
- Part 31: Next.js Introduction
- Part 32: App Router และ Pages Router
- Part 33: Server Components
- Part 34: Server Actions
- Part 35: Next.js Data Fetching
