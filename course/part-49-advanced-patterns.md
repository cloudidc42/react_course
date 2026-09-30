# Part 49: Advanced Patterns ใน React

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1551-1595  
> **เวลาเรียน:** ~5 ชั่วโมง

---

## 📚 Table of Contents

1. [Container/Presentational Pattern](#container-presentational-pattern)
2. [Observer Pattern](#observer-pattern)
3. [Flux Pattern](#flux-pattern)
4. [Command Pattern](#command-pattern)
5. [Factory Pattern สำหรับ Components](#factory-pattern)
6. [Dependency Injection](#dependency-injection)
7. [Adapter Pattern](#adapter-pattern)
8. [State Machine (XState)](#state-machine-xstate)
9. [Quiz](#quiz)

---

## Step 1551: Container/Presentational Pattern {#container-presentational-pattern}

Pattern นี้แยก Logic (Container) ออกจาก UI (Presentational) ทำให้ code สะอาดและ reuse ได้ดีขึ้น

### หลักการ

```
Container Component:
├── จัดการ State
├── ดึงข้อมูล (fetch data)
├── Handle events
└── ส่ง data ลงไปให้ Presentational via props

Presentational Component:
├── แสดง UI เท่านั้น
├── ไม่มี State (หรือมีน้อยมาก เฉพาะ UI state)
├── รับข้อมูลผ่าน props
└── เรียก callbacks จาก props
```

### ตัวอย่าง

```typescript
// ❌ Mixed Component - ยากต่อการ test และ reuse
function UserList() {
  const [users, setUsers] = useState([])
  const [isLoading, setIsLoading] = useState(true)
  
  useEffect(() => {
    fetch('/api/users')
      .then((r) => r.json())
      .then((data) => {
        setUsers(data)
        setIsLoading(false)
      })
  }, [])
  
  if (isLoading) return <div>Loading...</div>
  
  return (
    <div>
      {users.map((user) => (
        <div key={user.id} className="p-4 border rounded">
          <img src={user.avatar} alt={user.name} />
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  )
}

// ✅ Separated - แยกเป็น Container + Presentational
// Presentational Component
interface UserCardProps {
  user: {
    id: number
    name: string
    email: string
    avatar: string
    role: string
  }
  onSelect: (id: number) => void
}

function UserCard({ user, onSelect }: UserCardProps) {
  return (
    <div 
      className="p-4 border rounded cursor-pointer hover:bg-gray-50"
      onClick={() => onSelect(user.id)}
    >
      <img 
        src={user.avatar} 
        alt={user.name}
        className="w-12 h-12 rounded-full"
      />
      <h3 className="font-semibold">{user.name}</h3>
      <p className="text-gray-600 text-sm">{user.email}</p>
      <span className="badge">{user.role}</span>
    </div>
  )
}

// Presentational List
interface UserListViewProps {
  users: UserCardProps['user'][]
  isLoading: boolean
  error?: string
  onSelectUser: (id: number) => void
}

function UserListView({ users, isLoading, error, onSelectUser }: UserListViewProps) {
  if (isLoading) return <LoadingSpinner />
  if (error) return <ErrorMessage message={error} />
  if (users.length === 0) return <EmptyState message="ไม่พบผู้ใช้" />
  
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {users.map((user) => (
        <UserCard 
          key={user.id} 
          user={user} 
          onSelect={onSelectUser}
        />
      ))}
    </div>
  )
}

// Container Component
function UserListContainer() {
  const [users, setUsers] = useState([])
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | undefined>()
  const router = useRouter()
  
  useEffect(() => {
    fetch('/api/users')
      .then((r) => {
        if (!r.ok) throw new Error('Failed to fetch')
        return r.json()
      })
      .then((data) => {
        setUsers(data)
        setIsLoading(false)
      })
      .catch((err) => {
        setError(err.message)
        setIsLoading(false)
      })
  }, [])
  
  function handleSelectUser(id: number) {
    router.push(`/users/${id}`)
  }
  
  return (
    <UserListView
      users={users}
      isLoading={isLoading}
      error={error}
      onSelectUser={handleSelectUser}
    />
  )
}
```

### Modern Version: Custom Hooks + Presentational

```typescript
// hooks/useUsers.ts (เหมือน Container logic)
export function useUsers() {
  const [users, setUsers] = useState([])
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | undefined>()
  
  useEffect(() => {
    // fetch logic
  }, [])
  
  return { users, isLoading, error }
}

// Component ใช้ Hook + Presentational Component
function UsersPage() {
  const { users, isLoading, error } = useUsers()
  const router = useRouter()
  
  return (
    <UserListView
      users={users}
      isLoading={isLoading}
      error={error}
      onSelectUser={(id) => router.push(`/users/${id}`)}
    />
  )
}
```

---

## Step 1556: Observer Pattern {#observer-pattern}

Observer Pattern ช่วยให้ component อื่นๆ subscribe เพื่อรับ notification เมื่อ state เปลี่ยนแปลง

```typescript
// lib/EventEmitter.ts
type EventHandler<T = any> = (data: T) => void

class EventEmitter {
  private events: Map<string, Set<EventHandler>> = new Map()
  
  on<T>(event: string, handler: EventHandler<T>): () => void {
    if (!this.events.has(event)) {
      this.events.set(event, new Set())
    }
    
    this.events.get(event)!.add(handler)
    
    // Return unsubscribe function
    return () => this.off(event, handler)
  }
  
  off<T>(event: string, handler: EventHandler<T>): void {
    this.events.get(event)?.delete(handler)
  }
  
  emit<T>(event: string, data: T): void {
    this.events.get(event)?.forEach((handler) => handler(data))
  }
  
  once<T>(event: string, handler: EventHandler<T>): void {
    const wrapper = (data: T) => {
      handler(data)
      this.off(event, wrapper)
    }
    this.on(event, wrapper)
  }
}

export const eventBus = new EventEmitter()
```

```typescript
// hooks/useEventBus.ts
import { useEffect } from 'react'
import { eventBus } from '@/lib/EventEmitter'

export function useEventBus<T>(
  event: string,
  handler: (data: T) => void,
  deps: React.DependencyList = []
) {
  useEffect(() => {
    const unsubscribe = eventBus.on<T>(event, handler)
    return unsubscribe
  }, deps)
}

// การใช้งาน
function CartButton() {
  const [cartCount, setCartCount] = useState(0)
  
  useEventBus('cart:updated', (data: { count: number }) => {
    setCartCount(data.count)
  })
  
  return <button>Cart ({cartCount})</button>
}

// Component อื่นส่ง event
function ProductCard({ product }) {
  function addToCart() {
    // เพิ่มสินค้าใน cart
    fetch('/api/cart', {
      method: 'POST',
      body: JSON.stringify({ productId: product.id }),
    }).then(() => {
      eventBus.emit('cart:updated', { count: cartCount + 1 })
    })
  }
  
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={addToCart}>เพิ่มในตะกร้า</button>
    </div>
  )
}
```

---

## Step 1560: Flux Pattern {#flux-pattern}

Flux Pattern คือ Architecture pattern สำหรับจัดการ state ใน React แบบ Unidirectional data flow

```
Flux Architecture:
Action → Dispatcher → Store → View → Action → ...

Redux (Flux Implementation):
Action → Reducer → Store → Component → Action → ...
```

### Mini Flux Implementation

```typescript
// lib/store.ts
type Listener = () => void

interface Action {
  type: string
  payload?: any
}

type Reducer<S> = (state: S, action: Action) => S

class Store<S> {
  private state: S
  private reducer: Reducer<S>
  private listeners: Set<Listener> = new Set()
  
  constructor(reducer: Reducer<S>, initialState: S) {
    this.state = initialState
    this.reducer = reducer
  }
  
  getState(): S {
    return this.state
  }
  
  dispatch(action: Action): void {
    this.state = this.reducer(this.state, action)
    this.notify()
  }
  
  subscribe(listener: Listener): () => void {
    this.listeners.add(listener)
    return () => this.listeners.delete(listener)
  }
  
  private notify(): void {
    this.listeners.forEach((listener) => listener())
  }
}

// Cart Store
interface CartState {
  items: Array<{
    id: number
    name: string
    price: number
    quantity: number
  }>
  total: number
}

const initialCartState: CartState = {
  items: [],
  total: 0,
}

function cartReducer(state: CartState, action: Action): CartState {
  switch (action.type) {
    case 'CART_ADD_ITEM': {
      const existingItem = state.items.find((i) => i.id === action.payload.id)
      
      if (existingItem) {
        const items = state.items.map((i) =>
          i.id === action.payload.id
            ? { ...i, quantity: i.quantity + 1 }
            : i
        )
        return {
          items,
          total: items.reduce((sum, i) => sum + i.price * i.quantity, 0),
        }
      }
      
      const items = [...state.items, { ...action.payload, quantity: 1 }]
      return {
        items,
        total: items.reduce((sum, i) => sum + i.price * i.quantity, 0),
      }
    }
    
    case 'CART_REMOVE_ITEM': {
      const items = state.items.filter((i) => i.id !== action.payload.id)
      return {
        items,
        total: items.reduce((sum, i) => sum + i.price * i.quantity, 0),
      }
    }
    
    case 'CART_CLEAR':
      return initialCartState
    
    default:
      return state
  }
}

export const cartStore = new Store(cartReducer, initialCartState)
```

```typescript
// hooks/useStore.ts
function useStore<S>(store: Store<S>): S {
  const [state, setState] = useState(() => store.getState())
  
  useEffect(() => {
    return store.subscribe(() => {
      setState(store.getState())
    })
  }, [store])
  
  return state
}

// การใช้งาน
function CartSummary() {
  const cart = useStore(cartStore)
  
  return (
    <div>
      <p>จำนวน: {cart.items.length}</p>
      <p>รวม: ฿{cart.total.toFixed(2)}</p>
      <button onClick={() => cartStore.dispatch({ type: 'CART_CLEAR' })}>
        ล้างตะกร้า
      </button>
    </div>
  )
}
```

---

## Step 1565: Command Pattern {#command-pattern}

Command Pattern ช่วยให้ implement Undo/Redo functionality ได้

```typescript
// lib/CommandPattern.ts
interface Command {
  execute(): void
  undo(): void
  description: string
}

class CommandHistory {
  private undoStack: Command[] = []
  private redoStack: Command[] = []
  
  execute(command: Command): void {
    command.execute()
    this.undoStack.push(command)
    this.redoStack = [] // Clear redo stack
    
    console.log(`Executed: ${command.description}`)
  }
  
  undo(): void {
    const command = this.undoStack.pop()
    if (!command) return
    
    command.undo()
    this.redoStack.push(command)
    
    console.log(`Undone: ${command.description}`)
  }
  
  redo(): void {
    const command = this.redoStack.pop()
    if (!command) return
    
    command.execute()
    this.undoStack.push(command)
    
    console.log(`Redone: ${command.description}`)
  }
  
  canUndo(): boolean {
    return this.undoStack.length > 0
  }
  
  canRedo(): boolean {
    return this.redoStack.length > 0
  }
  
  getHistory(): string[] {
    return this.undoStack.map((c) => c.description)
  }
}

export const history = new CommandHistory()
```

```typescript
// ตัวอย่าง Text Editor Commands
class AddTextCommand implements Command {
  description: string
  
  constructor(
    private editor: { text: string },
    private textToAdd: string,
    private position: number
  ) {
    this.description = `Add "${textToAdd}" at position ${position}`
  }
  
  execute(): void {
    this.editor.text =
      this.editor.text.slice(0, this.position) +
      this.textToAdd +
      this.editor.text.slice(this.position)
  }
  
  undo(): void {
    this.editor.text =
      this.editor.text.slice(0, this.position) +
      this.editor.text.slice(this.position + this.textToAdd.length)
  }
}

// React Hook สำหรับ Command Pattern
function useCommandHistory() {
  const [canUndo, setCanUndo] = useState(false)
  const [canRedo, setCanRedo] = useState(false)
  
  function updateState() {
    setCanUndo(history.canUndo())
    setCanRedo(history.canRedo())
  }
  
  function execute(command: Command) {
    history.execute(command)
    updateState()
  }
  
  function undo() {
    history.undo()
    updateState()
  }
  
  function redo() {
    history.redo()
    updateState()
  }
  
  return { execute, undo, redo, canUndo, canRedo }
}
```

---

## Step 1570: Factory Pattern สำหรับ Components {#factory-pattern}

Factory Pattern ช่วยสร้าง Component ที่เหมาะสมตาม type ที่กำหนด

```typescript
// factories/ButtonFactory.tsx
type ButtonVariant = 'primary' | 'secondary' | 'danger' | 'ghost' | 'link'
type ButtonSize = 'sm' | 'md' | 'lg' | 'xl'

interface ButtonProps {
  variant?: ButtonVariant
  size?: ButtonSize
  loading?: boolean
  disabled?: boolean
  icon?: React.ReactNode
  children: React.ReactNode
  onClick?: () => void
}

// Base Button Styles
const baseStyles = 'inline-flex items-center justify-center gap-2 font-medium rounded-lg transition-all focus:outline-none focus:ring-2'

const variantStyles: Record<ButtonVariant, string> = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
  secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200 focus:ring-gray-500',
  danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
  ghost: 'bg-transparent text-gray-700 hover:bg-gray-100 focus:ring-gray-500',
  link: 'bg-transparent text-blue-600 hover:underline focus:ring-blue-500',
}

const sizeStyles: Record<ButtonSize, string> = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
  xl: 'px-8 py-4 text-xl',
}

// Button Factory
export function createButton(variant: ButtonVariant) {
  return function VariantButton({
    size = 'md',
    loading,
    disabled,
    icon,
    children,
    onClick,
    ...props
  }: Omit<ButtonProps, 'variant'>) {
    return (
      <button
        onClick={onClick}
        disabled={disabled || loading}
        className={`
          ${baseStyles}
          ${variantStyles[variant]}
          ${sizeStyles[size]}
          ${(disabled || loading) ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}
        `}
        {...props}
      >
        {loading ? (
          <span className="animate-spin rounded-full h-4 w-4 border-b-2 border-current" />
        ) : icon}
        {children}
      </button>
    )
  }
}

// สร้าง Button variants ต่างๆ
export const PrimaryButton = createButton('primary')
export const SecondaryButton = createButton('secondary')
export const DangerButton = createButton('danger')
export const GhostButton = createButton('ghost')
export const LinkButton = createButton('link')

// Generic Button ที่รับ variant
export function Button({ variant = 'primary', ...props }: ButtonProps) {
  const VariantButton = createButton(variant)
  return <VariantButton {...props} />
}
```

### Form Field Factory

```typescript
// factories/FormFieldFactory.tsx
type FieldType = 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'textarea' | 'select'

interface BaseFieldProps {
  name: string
  label: string
  required?: boolean
  error?: string
  hint?: string
}

interface InputFieldProps extends BaseFieldProps {
  type: Exclude<FieldType, 'textarea' | 'select'>
  value: string
  onChange: (value: string) => void
  placeholder?: string
}

interface TextareaFieldProps extends BaseFieldProps {
  type: 'textarea'
  value: string
  onChange: (value: string) => void
  rows?: number
}

interface SelectFieldProps extends BaseFieldProps {
  type: 'select'
  value: string
  onChange: (value: string) => void
  options: Array<{ value: string; label: string }>
}

type FieldProps = InputFieldProps | TextareaFieldProps | SelectFieldProps

const fieldClassNames = {
  label: 'block text-sm font-medium text-gray-700 mb-1',
  input: 'w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500',
  error: 'mt-1 text-sm text-red-600',
  hint: 'mt-1 text-sm text-gray-500',
}

export function FormFieldFactory(props: FieldProps) {
  const baseId = props.name
  
  function renderField() {
    switch (props.type) {
      case 'textarea':
        return (
          <textarea
            id={baseId}
            name={props.name}
            value={props.value}
            onChange={(e) => props.onChange(e.target.value)}
            rows={(props as TextareaFieldProps).rows || 4}
            required={props.required}
            className={`${fieldClassNames.input} resize-vertical`}
            aria-describedby={props.error ? `${baseId}-error` : undefined}
          />
        )
      
      case 'select':
        return (
          <select
            id={baseId}
            name={props.name}
            value={props.value}
            onChange={(e) => props.onChange(e.target.value)}
            required={props.required}
            className={fieldClassNames.input}
          >
            <option value="">-- เลือก --</option>
            {(props as SelectFieldProps).options.map((option) => (
              <option key={option.value} value={option.value}>
                {option.label}
              </option>
            ))}
          </select>
        )
      
      default:
        return (
          <input
            id={baseId}
            type={props.type}
            name={props.name}
            value={(props as InputFieldProps).value}
            onChange={(e) => (props as InputFieldProps).onChange(e.target.value)}
            placeholder={(props as InputFieldProps).placeholder}
            required={props.required}
            className={`
              ${fieldClassNames.input}
              ${props.error ? 'border-red-500 focus:ring-red-500' : 'border-gray-300'}
            `}
            aria-describedby={props.error ? `${baseId}-error` : undefined}
          />
        )
    }
  }
  
  return (
    <div className="mb-4">
      <label htmlFor={baseId} className={fieldClassNames.label}>
        {props.label}
        {props.required && <span className="text-red-500 ml-1">*</span>}
      </label>
      
      {renderField()}
      
      {props.error && (
        <p id={`${baseId}-error`} className={fieldClassNames.error} role="alert">
          {props.error}
        </p>
      )}
      
      {props.hint && !props.error && (
        <p className={fieldClassNames.hint}>{props.hint}</p>
      )}
    </div>
  )
}
```

---

## Step 1575: Dependency Injection {#dependency-injection}

Dependency Injection ใน React ทำได้ผ่าน Context API

```typescript
// services/api.ts
export interface ApiService {
  get<T>(url: string): Promise<T>
  post<T>(url: string, data: any): Promise<T>
  put<T>(url: string, data: any): Promise<T>
  delete(url: string): Promise<void>
}

// Real implementation
export class HttpApiService implements ApiService {
  private baseUrl: string
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl
  }
  
  async get<T>(url: string): Promise<T> {
    const res = await fetch(this.baseUrl + url, {
      headers: { Authorization: `Bearer ${getToken()}` },
    })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    return res.json()
  }
  
  async post<T>(url: string, data: any): Promise<T> {
    const res = await fetch(this.baseUrl + url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${getToken()}`,
      },
      body: JSON.stringify(data),
    })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    return res.json()
  }
  
  async put<T>(url: string, data: any): Promise<T> {
    const res = await fetch(this.baseUrl + url, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${getToken()}`,
      },
      body: JSON.stringify(data),
    })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    return res.json()
  }
  
  async delete(url: string): Promise<void> {
    const res = await fetch(this.baseUrl + url, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${getToken()}` },
    })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
  }
}

// Mock implementation สำหรับ Testing
export class MockApiService implements ApiService {
  private mockData: Record<string, any> = {}
  
  setMock(url: string, data: any) {
    this.mockData[url] = data
  }
  
  async get<T>(url: string): Promise<T> {
    return this.mockData[url] as T
  }
  
  async post<T>(url: string, data: any): Promise<T> {
    return { ...data, id: Date.now() } as T
  }
  
  async put<T>(url: string, data: any): Promise<T> {
    return data as T
  }
  
  async delete(url: string): Promise<void> {
    delete this.mockData[url]
  }
}
```

```typescript
// contexts/ServicesContext.tsx
const ServicesContext = createContext<{
  api: ApiService
  analytics: AnalyticsService
  logger: LoggerService
} | null>(null)

export function ServicesProvider({
  children,
  api,
  analytics,
  logger,
}: {
  children: React.ReactNode
  api: ApiService
  analytics: AnalyticsService
  logger: LoggerService
}) {
  return (
    <ServicesContext.Provider value={{ api, analytics, logger }}>
      {children}
    </ServicesContext.Provider>
  )
}

export function useServices() {
  const ctx = useContext(ServicesContext)
  if (!ctx) throw new Error('useServices must be used within ServicesProvider')
  return ctx
}

// Setup ใน app
// Production
const apiService = new HttpApiService(process.env.NEXT_PUBLIC_API_URL!)
const analyticsService = new GoogleAnalyticsService()
const loggerService = new ConsoleLogger()

// Testing
const mockApiService = new MockApiService()
const noopAnalyticsService = new NoopAnalyticsService()
const mockLoggerService = new MockLogger()
```

---

## Step 1580: Adapter Pattern {#adapter-pattern}

Adapter Pattern แปลง interface ของ third-party library ให้เข้ากับ codebase ของเรา

```typescript
// adapters/DateAdapter.ts
// เราใช้ dayjs แต่อาจเปลี่ยนเป็น date-fns ในอนาคต

export interface DateAdapter {
  format(date: Date | string, format: string): string
  parse(dateString: string, format: string): Date
  isValid(date: any): boolean
  addDays(date: Date, days: number): Date
  diffInDays(date1: Date, date2: Date): number
  startOfDay(date: Date): Date
  endOfDay(date: Date): Date
  isAfter(date1: Date, date2: Date): boolean
  isBefore(date1: Date, date2: Date): boolean
  toISOString(date: Date): string
}

// Dayjs Adapter
import dayjs from 'dayjs'
import 'dayjs/locale/th'
import buddhistEra from 'dayjs/plugin/buddhistEra'
dayjs.extend(buddhistEra)

export class DayjsAdapter implements DateAdapter {
  format(date: Date | string, format: string): string {
    return dayjs(date).format(format)
  }
  
  parse(dateString: string, format: string): Date {
    return dayjs(dateString, format).toDate()
  }
  
  isValid(date: any): boolean {
    return dayjs(date).isValid()
  }
  
  addDays(date: Date, days: number): Date {
    return dayjs(date).add(days, 'day').toDate()
  }
  
  diffInDays(date1: Date, date2: Date): number {
    return Math.abs(dayjs(date1).diff(dayjs(date2), 'day'))
  }
  
  startOfDay(date: Date): Date {
    return dayjs(date).startOf('day').toDate()
  }
  
  endOfDay(date: Date): Date {
    return dayjs(date).endOf('day').toDate()
  }
  
  isAfter(date1: Date, date2: Date): boolean {
    return dayjs(date1).isAfter(dayjs(date2))
  }
  
  isBefore(date1: Date, date2: Date): boolean {
    return dayjs(date1).isBefore(dayjs(date2))
  }
  
  toISOString(date: Date): string {
    return dayjs(date).toISOString()
  }
}

// สร้าง Context สำหรับ Date Adapter
const DateContext = createContext<DateAdapter>(new DayjsAdapter())

export function useDateAdapter() {
  return useContext(DateContext)
}

// ตัวอย่าง Payment Gateway Adapter
export interface PaymentAdapter {
  createPaymentIntent(amount: number, currency: string): Promise<{ clientSecret: string }>
  confirmPayment(clientSecret: string): Promise<{ success: boolean }>
  refund(paymentId: string, amount?: number): Promise<{ success: boolean }>
}

// Stripe Adapter
class StripeAdapter implements PaymentAdapter {
  private stripe: any
  
  constructor(publishableKey: string) {
    this.stripe = require('stripe')(publishableKey)
  }
  
  async createPaymentIntent(amount: number, currency: string) {
    const intent = await this.stripe.paymentIntents.create({
      amount: amount * 100, // Stripe ใช้ satang
      currency,
    })
    return { clientSecret: intent.client_secret }
  }
  
  async confirmPayment(clientSecret: string) {
    // confirm payment
    return { success: true }
  }
  
  async refund(paymentId: string, amount?: number) {
    await this.stripe.refunds.create({
      payment_intent: paymentId,
      amount: amount ? amount * 100 : undefined,
    })
    return { success: true }
  }
}

// OmiseGo Adapter
class OmiseAdapter implements PaymentAdapter {
  private omise: any
  
  async createPaymentIntent(amount: number, currency: string) {
    const charge = await this.omise.charges.create({
      amount: amount * 100,
      currency,
      card: '...',
    })
    return { clientSecret: charge.id }
  }
  
  async confirmPayment(clientSecret: string) {
    return { success: true }
  }
  
  async refund(paymentId: string, amount?: number) {
    return { success: true }
  }
}
```

---

## Step 1585-1595: State Machine (XState) {#state-machine-xstate}

XState ช่วยจัดการ state ที่ซับซ้อนด้วย State Machine

### Installation

```bash
npm install xstate @xstate/react
```

### State Machine พื้นฐาน

```typescript
// machines/checkoutMachine.ts
import { createMachine, assign } from 'xstate'

interface CheckoutContext {
  items: Array<{ id: string; price: number; quantity: number }>
  total: number
  paymentMethod: string | null
  orderId: string | null
  error: string | null
}

type CheckoutEvent =
  | { type: 'PROCEED_TO_SHIPPING' }
  | { type: 'PROCEED_TO_PAYMENT'; paymentMethod: string }
  | { type: 'PLACE_ORDER' }
  | { type: 'PAYMENT_SUCCESS'; orderId: string }
  | { type: 'PAYMENT_FAILURE'; error: string }
  | { type: 'RETRY' }
  | { type: 'BACK' }
  | { type: 'RESET' }

export const checkoutMachine = createMachine<CheckoutContext, CheckoutEvent>({
  id: 'checkout',
  initial: 'cart',
  
  context: {
    items: [],
    total: 0,
    paymentMethod: null,
    orderId: null,
    error: null,
  },
  
  states: {
    cart: {
      on: {
        PROCEED_TO_SHIPPING: 'shipping',
      },
    },
    
    shipping: {
      on: {
        PROCEED_TO_PAYMENT: {
          target: 'payment',
          actions: assign({
            paymentMethod: (_, event) => event.paymentMethod,
          }),
        },
        BACK: 'cart',
      },
    },
    
    payment: {
      on: {
        PLACE_ORDER: 'processing',
        BACK: 'shipping',
      },
    },
    
    processing: {
      invoke: {
        src: 'placeOrder',
        onDone: {
          target: 'success',
          actions: assign({
            orderId: (_, event) => event.data.orderId,
          }),
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message,
          }),
        },
      },
    },
    
    success: {
      type: 'final',
      on: {
        RESET: {
          target: 'cart',
          actions: assign({
            items: () => [],
            total: () => 0,
            paymentMethod: () => null,
            orderId: () => null,
            error: () => null,
          }),
        },
      },
    },
    
    error: {
      on: {
        RETRY: 'processing',
        BACK: 'payment',
      },
    },
  },
})
```

```typescript
// components/Checkout.tsx
'use client'

import { useMachine } from '@xstate/react'
import { checkoutMachine } from '@/machines/checkoutMachine'

export function CheckoutFlow() {
  const [state, send] = useMachine(checkoutMachine, {
    services: {
      placeOrder: async (context) => {
        const response = await fetch('/api/orders', {
          method: 'POST',
          body: JSON.stringify({
            items: context.items,
            paymentMethod: context.paymentMethod,
          }),
        })
        
        if (!response.ok) {
          throw new Error('Failed to place order')
        }
        
        return response.json()
      },
    },
  })
  
  return (
    <div className="max-w-lg mx-auto">
      {/* Progress Steps */}
      <div className="flex mb-8">
        {['cart', 'shipping', 'payment', 'processing', 'success'].map((step) => (
          <div
            key={step}
            className={`flex-1 text-center text-sm ${
              state.matches(step) ? 'text-blue-600 font-bold' : 'text-gray-400'
            }`}
          >
            {step}
          </div>
        ))}
      </div>
      
      {/* Cart State */}
      {state.matches('cart') && (
        <div>
          <h2 className="text-xl font-bold mb-4">ตะกร้าสินค้า</h2>
          <p>รวม: ฿{state.context.total}</p>
          <button
            onClick={() => send({ type: 'PROCEED_TO_SHIPPING' })}
            className="w-full bg-blue-600 text-white py-3 rounded-lg mt-4"
          >
            ไปหน้าที่อยู่จัดส่ง
          </button>
        </div>
      )}
      
      {/* Shipping State */}
      {state.matches('shipping') && (
        <div>
          <h2 className="text-xl font-bold mb-4">ที่อยู่จัดส่ง</h2>
          <div className="flex gap-2 mt-4">
            <button
              onClick={() => send({ type: 'BACK' })}
              className="flex-1 border border-gray-300 py-3 rounded-lg"
            >
              ย้อนกลับ
            </button>
            <button
              onClick={() =>
                send({
                  type: 'PROCEED_TO_PAYMENT',
                  paymentMethod: 'credit_card',
                })
              }
              className="flex-1 bg-blue-600 text-white py-3 rounded-lg"
            >
              ไปหน้าชำระเงิน
            </button>
          </div>
        </div>
      )}
      
      {/* Payment State */}
      {state.matches('payment') && (
        <div>
          <h2 className="text-xl font-bold mb-4">ชำระเงิน</h2>
          <div className="flex gap-2 mt-4">
            <button
              onClick={() => send({ type: 'BACK' })}
              className="flex-1 border border-gray-300 py-3 rounded-lg"
            >
              ย้อนกลับ
            </button>
            <button
              onClick={() => send({ type: 'PLACE_ORDER' })}
              className="flex-1 bg-green-600 text-white py-3 rounded-lg"
            >
              ยืนยันคำสั่งซื้อ
            </button>
          </div>
        </div>
      )}
      
      {/* Processing State */}
      {state.matches('processing') && (
        <div className="text-center py-8">
          <div className="animate-spin rounded-full h-16 w-16 border-b-2 border-blue-600 mx-auto mb-4" />
          <p className="text-gray-600">กำลังดำเนินการ...</p>
        </div>
      )}
      
      {/* Success State */}
      {state.matches('success') && (
        <div className="text-center py-8">
          <div className="text-5xl mb-4">✅</div>
          <h2 className="text-2xl font-bold text-green-600 mb-2">สั่งซื้อสำเร็จ!</h2>
          <p className="text-gray-600 mb-6">
            หมายเลขคำสั่งซื้อ: {state.context.orderId}
          </p>
          <button
            onClick={() => send({ type: 'RESET' })}
            className="bg-blue-600 text-white px-8 py-3 rounded-lg"
          >
            ซื้อสินค้าต่อ
          </button>
        </div>
      )}
      
      {/* Error State */}
      {state.matches('error') && (
        <div className="text-center py-8">
          <div className="text-5xl mb-4">❌</div>
          <h2 className="text-2xl font-bold text-red-600 mb-2">เกิดข้อผิดพลาด</h2>
          <p className="text-gray-600 mb-6">{state.context.error}</p>
          <div className="flex gap-2 justify-center">
            <button
              onClick={() => send({ type: 'BACK' })}
              className="border border-gray-300 px-6 py-3 rounded-lg"
            >
              กลับไปหน้าชำระเงิน
            </button>
            <button
              onClick={() => send({ type: 'RETRY' })}
              className="bg-blue-600 text-white px-6 py-3 rounded-lg"
            >
              ลองใหม่
            </button>
          </div>
        </div>
      )}
    </div>
  )
}
```

---

## 🧪 Quiz - Part 49

**ข้อ 1:** Container/Presentational Pattern มีประโยชน์หลักคืออะไร?
- A) ทำให้ code เร็วขึ้น
- B) แยก Logic และ UI ทำให้ test และ reuse ง่ายขึ้น
- C) ลดจำนวน re-renders
- D) ทำให้ TypeScript type safe ขึ้น

**ข้อ 2:** State Machine (XState) เหมาะกับ use case ใด?
- A) Simple form state
- B) Complex workflow ที่มีหลาย states และ transitions
- C) Fetching data
- D) Animation

**ข้อ 3:** Observer Pattern ใน React ใช้ implement ผ่านอะไร?
- A) useState
- B) useEffect
- C) EventEmitter หรือ EventBus
- D) useReducer

**ข้อ 4:** Factory Pattern สำหรับ Components มีประโยชน์อย่างไร?
- A) ทำให้ render เร็วขึ้น
- B) สร้าง Component ที่ถูกต้องตาม type/variant โดยอัตโนมัติ
- C) จัดการ state
- D) Handle errors

**เฉลย:** 1-B, 2-B, 3-C, 4-B

---

> **➡️ Next:** [Part 50: Micro-frontends](./part-50-micro-frontends.md)
