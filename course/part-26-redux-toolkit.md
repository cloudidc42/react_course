# Part 26: Redux Toolkit - State Management ขั้นสูง

**Step 661-700** | ระดับ: สูง | เวลาเรียน: 5-6 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [Redux คืออะไร?](#redux-คืออะไร)
2. [ทำไมต้อง Redux Toolkit?](#ทำไมต้อง-redux-toolkit)
3. [การติดตั้งและ Setup](#การติดตั้งและ-setup)
4. [configureStore](#configurestore)
5. [createSlice](#createslice)
6. [useSelector และ useDispatch](#useselector-และ-usedispatch)
7. [Async Thunks (createAsyncThunk)](#async-thunks-createasyncthunk)
8. [RTK Query เบื้องต้น](#rtk-query-เบื้องต้น)
9. [ตัวอย่าง Shopping Cart](#ตัวอย่าง-shopping-cart-redux)
10. [Quiz](#quiz)

---

## Step 661-665: Redux คืออะไร?

Redux เป็น state management library ที่ช่วยจัดการ global state ของแอปพลิเคชัน

### แนวคิดหลัก

```
Store: ที่เก็บ state ทั้งหมด (single source of truth)
Action: วัตถุที่อธิบายสิ่งที่เกิดขึ้น { type, payload }
Reducer: function ที่รับ state + action แล้วคืน state ใหม่
Dispatch: วิธีส่ง action ไปยัง store
```

### Redux Data Flow

```
User Click → Dispatch(Action) → Reducer → New State → UI Update
```

### เมื่อไหร่ควรใช้ Redux?

```
✅ ควรใช้เมื่อ:
- State ที่ต้องการใช้ใน components หลายๆ ที่
- State ซับซ้อนและ nested มาก
- State ต้องมีประวัติ (undo/redo)
- หลาย components ต้อง update state เดียวกัน

❌ ไม่ต้องใช้เมื่อ:
- State ใช้แค่ใน component เดียว (ใช้ useState แทน)
- State ง่ายๆ ไม่กี่ค่า
- App ขนาดเล็ก (Context API เพียงพอ)
```

---

## Step 666-668: ทำไมต้อง Redux Toolkit?

Redux ดั้งเดิมมีปัญหา:
1. Boilerplate code เยอะมาก
2. ต้อง setup หลายอย่าง (redux, react-redux, redux-thunk, ...)
3. เขียน action types, action creators แยก

Redux Toolkit (RTK) แก้ปัญหาเหล่านี้:

```
Redux ดั้งเดิม:
- แยกไฟล์ types, actions, reducers
- ต้อง spread state เองทุกครั้ง
- Async logic ซับซ้อน

Redux Toolkit:
- createSlice รวมทุกอย่างในที่เดียว
- ใช้ Immer → แก้ state โดยตรงได้ (mutable-style)
- createAsyncThunk จัดการ async อย่างง่าย
```

---

## Step 669-670: การติดตั้งและ Setup

```bash
# ติดตั้ง
npm install @reduxjs/toolkit react-redux

# หรือ create-react-app พร้อม Redux template
npx create-react-app my-app --template redux
```

### โครงสร้างโปรเจกต์

```
src/
├── store/
│   ├── index.js          # configureStore
│   └── slices/
│       ├── cartSlice.js
│       ├── userSlice.js
│       └── productSlice.js
├── hooks/
│   └── useAppDispatch.js  # typed dispatch
└── App.jsx
```

---

## Step 671-673: configureStore

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit'
import cartReducer from './slices/cartSlice'
import userReducer from './slices/userSlice'
import productReducer from './slices/productSlice'

const store = configureStore({
  reducer: {
    cart: cartReducer,
    user: userReducer,
    products: productReducer,
  },
  // middleware เพิ่มเติม (RTK include redux-thunk อัตโนมัติ)
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(/* custom middlewares */),
  
  // DevTools เปิดอัตโนมัติใน development
  devTools: process.env.NODE_ENV !== 'production',
})

export default store
export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

```jsx
// main.jsx
import { Provider } from 'react-redux'
import store from './store'

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
)
```

---

## Step 674-678: createSlice

`createSlice` รวม actions, action creators, และ reducer ไว้ในที่เดียว

### Counter Slice (ตัวอย่างง่าย)

```jsx
// store/slices/counterSlice.js
import { createSlice } from '@reduxjs/toolkit'

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0,
    step: 1,
  },
  reducers: {
    increment: (state) => {
      state.value += state.step  // Immer ทำให้เขียน mutable ได้!
    },
    decrement: (state) => {
      state.value -= state.step
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    },
    resetCounter: (state) => {
      state.value = 0
    },
    setStep: (state, action) => {
      state.step = action.payload
    },
  },
})

// Export actions
export const {
  increment,
  decrement,
  incrementByAmount,
  resetCounter,
  setStep,
} = counterSlice.actions

// Export reducer
export default counterSlice.reducer
```

### Cart Slice

```jsx
// store/slices/cartSlice.js
import { createSlice } from '@reduxjs/toolkit'

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    totalQuantity: 0,
    totalPrice: 0,
  },
  reducers: {
    addToCart: (state, action) => {
      const { product } = action.payload
      const existingItem = state.items.find(item => item.id === product.id)

      if (existingItem) {
        existingItem.quantity += 1
      } else {
        state.items.push({ ...product, quantity: 1 })
      }

      state.totalQuantity += 1
      state.totalPrice += product.price
    },

    removeFromCart: (state, action) => {
      const { id } = action.payload
      const item = state.items.find(item => item.id === id)

      if (item) {
        state.totalQuantity -= item.quantity
        state.totalPrice -= item.price * item.quantity
        state.items = state.items.filter(item => item.id !== id)
      }
    },

    updateQuantity: (state, action) => {
      const { id, quantity } = action.payload
      const item = state.items.find(item => item.id === id)

      if (item && quantity > 0) {
        const diff = quantity - item.quantity
        item.quantity = quantity
        state.totalQuantity += diff
        state.totalPrice += diff * item.price
      }
    },

    clearCart: (state) => {
      state.items = []
      state.totalQuantity = 0
      state.totalPrice = 0
    },
  },
})

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions

// Selectors
export const selectCartItems = state => state.cart.items
export const selectTotalQuantity = state => state.cart.totalQuantity
export const selectTotalPrice = state => state.cart.totalPrice

export default cartSlice.reducer
```

---

## Step 679-682: useSelector และ useDispatch

### การอ่านและอัพเดท State

```jsx
import { useSelector, useDispatch } from 'react-redux'
import { increment, decrement, incrementByAmount } from './store/slices/counterSlice'

function Counter() {
  const value = useSelector(state => state.counter.value)
  const step = useSelector(state => state.counter.step)
  const dispatch = useDispatch()

  return (
    <div>
      <h2>Count: {value}</h2>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(incrementByAmount(10))}>+10</button>
    </div>
  )
}
```

### Selectors ที่ดี

```jsx
// ✅ ดี - ใช้ selector functions แยกจาก component
// store/slices/cartSlice.js
export const selectCartItems = state => state.cart.items
export const selectTotalItems = state => state.cart.totalQuantity
export const selectTotalPrice = state => state.cart.totalPrice

// Derived selectors กับ createSelector (reselect)
import { createSelector } from '@reduxjs/toolkit'

export const selectCartSummary = createSelector(
  selectCartItems,
  selectTotalItems,
  selectTotalPrice,
  (items, totalItems, totalPrice) => ({
    items,
    totalItems,
    totalPrice,
    hasItems: items.length > 0,
    discountedPrice: totalPrice * 0.9,  // 10% discount
  })
)

// ใน Component
function CartSummary() {
  const { totalItems, totalPrice, hasItems } = useSelector(selectCartSummary)

  if (!hasItems) return <p>ตะกร้าว่าง</p>

  return (
    <div>
      <p>จำนวน: {totalItems} ชิ้น</p>
      <p>รวม: ฿{totalPrice}</p>
    </div>
  )
}
```

---

## Step 683-688: Async Thunks (createAsyncThunk)

`createAsyncThunk` จัดการ async operations พร้อม loading/success/error states อัตโนมัติ

### สร้าง Async Thunk

```jsx
// store/slices/productSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import { productService } from '../../services/productService'

// สร้าง async thunk
export const fetchProducts = createAsyncThunk(
  'products/fetchAll',  // action type prefix
  async (params, { rejectWithValue }) => {
    try {
      const response = await productService.getAll(params)
      return response.data
    } catch (error) {
      return rejectWithValue(error.response?.data?.message || 'เกิดข้อผิดพลาด')
    }
  }
)

export const fetchProductById = createAsyncThunk(
  'products/fetchById',
  async (id, { rejectWithValue }) => {
    try {
      const response = await productService.getById(id)
      return response.data
    } catch (error) {
      return rejectWithValue(error.response?.data?.message || 'ไม่พบสินค้า')
    }
  }
)

export const createProduct = createAsyncThunk(
  'products/create',
  async (data, { rejectWithValue }) => {
    try {
      const response = await productService.create(data)
      return response.data
    } catch (error) {
      return rejectWithValue(error.response?.data || 'เกิดข้อผิดพลาด')
    }
  }
)

const productSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    currentProduct: null,
    loading: false,
    error: null,
    totalCount: 0,
  },
  reducers: {
    clearCurrentProduct: (state) => {
      state.currentProduct = null
    },
    clearError: (state) => {
      state.error = null
    },
  },
  extraReducers: (builder) => {
    // fetchProducts
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true
        state.error = null
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false
        state.items = action.payload.data
        state.totalCount = action.payload.total
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false
        state.error = action.payload
      })

    // fetchProductById
    builder
      .addCase(fetchProductById.pending, (state) => {
        state.loading = true
      })
      .addCase(fetchProductById.fulfilled, (state, action) => {
        state.loading = false
        state.currentProduct = action.payload
      })
      .addCase(fetchProductById.rejected, (state, action) => {
        state.loading = false
        state.error = action.payload
      })

    // createProduct
    builder
      .addCase(createProduct.fulfilled, (state, action) => {
        state.items.unshift(action.payload)
      })
  },
})

export const { clearCurrentProduct, clearError } = productSlice.actions
export default productSlice.reducer
```

### การใช้งาน Async Thunk

```jsx
function ProductPage() {
  const dispatch = useDispatch()
  const { items, loading, error } = useSelector(state => state.products)

  useEffect(() => {
    dispatch(fetchProducts({ page: 1, limit: 10 }))
  }, [dispatch])

  if (loading) return <LoadingSpinner />
  if (error) return <ErrorMessage message={error} />

  return (
    <div>
      {items.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

---

## Step 689-695: RTK Query เบื้องต้น

RTK Query เป็น data fetching/caching tool ที่มาพร้อม Redux Toolkit

### Setup

```jsx
// store/api/productsApi.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const productsApi = createApi({
  reducerPath: 'productsApi',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL,
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('token')
      if (token) {
        headers.set('Authorization', `Bearer ${token}`)
      }
      return headers
    },
  }),
  tagTypes: ['Product'],
  endpoints: (builder) => ({
    // GET all products
    getProducts: builder.query({
      query: (params) => ({
        url: '/products',
        params,
      }),
      providesTags: ['Product'],
    }),

    // GET single product
    getProduct: builder.query({
      query: (id) => `/products/${id}`,
      providesTags: (result, error, id) => [{ type: 'Product', id }],
    }),

    // CREATE product
    createProduct: builder.mutation({
      query: (data) => ({
        url: '/products',
        method: 'POST',
        body: data,
      }),
      invalidatesTags: ['Product'],  // auto refetch หลัง create
    }),

    // UPDATE product
    updateProduct: builder.mutation({
      query: ({ id, ...data }) => ({
        url: `/products/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Product', id }],
    }),

    // DELETE product
    deleteProduct: builder.mutation({
      query: (id) => ({
        url: `/products/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['Product'],
    }),
  }),
})

export const {
  useGetProductsQuery,
  useGetProductQuery,
  useCreateProductMutation,
  useUpdateProductMutation,
  useDeleteProductMutation,
} = productsApi
```

```jsx
// store/index.js - เพิ่ม RTK Query
import { productsApi } from './api/productsApi'

const store = configureStore({
  reducer: {
    cart: cartReducer,
    [productsApi.reducerPath]: productsApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(productsApi.middleware),
})
```

### การใช้งาน RTK Query

```jsx
// useGetProductsQuery
function ProductList() {
  const {
    data,
    isLoading,
    isError,
    error,
    refetch,
  } = useGetProductsQuery({ page: 1, limit: 10 })

  if (isLoading) return <LoadingSpinner />
  if (isError) return <ErrorMessage message={error.message} onRetry={refetch} />

  return (
    <div>
      {data?.data?.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}

// useMutation
function CreateProductForm() {
  const [createProduct, { isLoading, isSuccess, error }] = useCreateProductMutation()

  const handleSubmit = async (data) => {
    try {
      await createProduct(data).unwrap()
      alert('สร้างสินค้าสำเร็จ!')
    } catch (err) {
      console.error('Failed to create product:', err)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'กำลังบันทึก...' : 'บันทึก'}
      </button>
    </form>
  )
}
```

---

## Step 696-700: ตัวอย่าง Shopping Cart Redux

### Cart Slice สมบูรณ์

```jsx
// store/slices/cartSlice.js
import { createSlice, createSelector } from '@reduxjs/toolkit'

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: JSON.parse(localStorage.getItem('cart') || '[]'),
    couponCode: null,
    discount: 0,
  },
  reducers: {
    addToCart: (state, { payload: product }) => {
      const existing = state.items.find(i => i.id === product.id)
      if (existing) {
        existing.quantity++
      } else {
        state.items.push({ ...product, quantity: 1 })
      }
      localStorage.setItem('cart', JSON.stringify(state.items))
    },

    removeFromCart: (state, { payload: id }) => {
      state.items = state.items.filter(i => i.id !== id)
      localStorage.setItem('cart', JSON.stringify(state.items))
    },

    incrementQuantity: (state, { payload: id }) => {
      const item = state.items.find(i => i.id === id)
      if (item) item.quantity++
      localStorage.setItem('cart', JSON.stringify(state.items))
    },

    decrementQuantity: (state, { payload: id }) => {
      const item = state.items.find(i => i.id === id)
      if (item) {
        if (item.quantity === 1) {
          state.items = state.items.filter(i => i.id !== id)
        } else {
          item.quantity--
        }
      }
      localStorage.setItem('cart', JSON.stringify(state.items))
    },

    applyCoupon: (state, { payload: code }) => {
      const COUPONS = { 'SAVE10': 10, 'SAVE20': 20 }
      if (COUPONS[code]) {
        state.couponCode = code
        state.discount = COUPONS[code]
      }
    },

    clearCart: (state) => {
      state.items = []
      state.couponCode = null
      state.discount = 0
      localStorage.removeItem('cart')
    },
  },
})

export const {
  addToCart, removeFromCart,
  incrementQuantity, decrementQuantity,
  applyCoupon, clearCart,
} = cartSlice.actions

// Selectors
const selectCartState = state => state.cart

export const selectCartItems = createSelector(
  selectCartState,
  cart => cart.items
)

export const selectCartCount = createSelector(
  selectCartItems,
  items => items.reduce((sum, item) => sum + item.quantity, 0)
)

export const selectSubtotal = createSelector(
  selectCartItems,
  items => items.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

export const selectDiscount = createSelector(
  selectCartState,
  selectSubtotal,
  (cart, subtotal) => (subtotal * cart.discount) / 100
)

export const selectTotal = createSelector(
  selectSubtotal,
  selectDiscount,
  (subtotal, discount) => subtotal - discount
)

export default cartSlice.reducer
```

### Cart Component

```jsx
// components/Cart.jsx
import { useSelector, useDispatch } from 'react-redux'
import {
  selectCartItems, selectCartCount, selectSubtotal, selectTotal, selectDiscount,
  removeFromCart, incrementQuantity, decrementQuantity, applyCoupon, clearCart,
} from '../store/slices/cartSlice'

function Cart() {
  const dispatch = useDispatch()
  const items = useSelector(selectCartItems)
  const count = useSelector(selectCartCount)
  const subtotal = useSelector(selectSubtotal)
  const discount = useSelector(selectDiscount)
  const total = useSelector(selectTotal)
  const [couponInput, setCouponInput] = useState('')

  if (items.length === 0) {
    return (
      <div style={{ textAlign: 'center', padding: '3rem' }}>
        <div style={{ fontSize: '4rem' }}>🛒</div>
        <h2>ตะกร้าของคุณว่างเปล่า</h2>
      </div>
    )
  }

  return (
    <div style={{ maxWidth: '800px', margin: '0 auto', padding: '2rem' }}>
      <h2>ตะกร้าสินค้า ({count} ชิ้น)</h2>

      {/* Cart Items */}
      <div>
        {items.map(item => (
          <div key={item.id} style={{
            display: 'flex',
            alignItems: 'center',
            gap: '1rem',
            padding: '1rem',
            border: '1px solid #e2e8f0',
            borderRadius: '8px',
            marginBottom: '0.5rem',
          }}>
            <img src={item.image} alt={item.name} style={{ width: 80, height: 80, objectFit: 'cover' }} />
            <div style={{ flex: 1 }}>
              <h4 style={{ margin: 0 }}>{item.name}</h4>
              <p style={{ margin: 0, color: '#666' }}>฿{item.price}</p>
            </div>
            <div style={{ display: 'flex', alignItems: 'center', gap: '0.5rem' }}>
              <button onClick={() => dispatch(decrementQuantity(item.id))}>-</button>
              <span>{item.quantity}</span>
              <button onClick={() => dispatch(incrementQuantity(item.id))}>+</button>
            </div>
            <p style={{ fontWeight: 'bold' }}>฿{item.price * item.quantity}</p>
            <button
              onClick={() => dispatch(removeFromCart(item.id))}
              style={{ color: 'red', background: 'none', border: 'none', cursor: 'pointer' }}
            >
              ✕
            </button>
          </div>
        ))}
      </div>

      {/* Coupon */}
      <div style={{ display: 'flex', gap: '0.5rem', margin: '1rem 0' }}>
        <input
          value={couponInput}
          onChange={e => setCouponInput(e.target.value.toUpperCase())}
          placeholder="โค้ดส่วนลด (SAVE10, SAVE20)"
          style={{ flex: 1, padding: '0.5rem' }}
        />
        <button onClick={() => dispatch(applyCoupon(couponInput))}>ใช้โค้ด</button>
      </div>

      {/* Summary */}
      <div style={{
        backgroundColor: '#f7fafc',
        padding: '1rem',
        borderRadius: '8px',
      }}>
        <div style={{ display: 'flex', justifyContent: 'space-between' }}>
          <span>ราคาสินค้า:</span>
          <span>฿{subtotal}</span>
        </div>
        {discount > 0 && (
          <div style={{ display: 'flex', justifyContent: 'space-between', color: 'green' }}>
            <span>ส่วนลด:</span>
            <span>-฿{discount}</span>
          </div>
        )}
        <div style={{
          display: 'flex',
          justifyContent: 'space-between',
          fontWeight: 'bold',
          fontSize: '1.2rem',
          borderTop: '1px solid #e2e8f0',
          paddingTop: '0.5rem',
          marginTop: '0.5rem',
        }}>
          <span>รวมทั้งสิ้น:</span>
          <span>฿{total}</span>
        </div>
      </div>

      <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: '1rem' }}>
        <button onClick={() => dispatch(clearCart())}>ล้างตะกร้า</button>
        <button style={{
          padding: '0.75rem 2rem',
          backgroundColor: '#48bb78',
          color: 'white',
          border: 'none',
          borderRadius: '8px',
          cursor: 'pointer',
        }}>
          ชำระเงิน
        </button>
      </div>
    </div>
  )
}

export default Cart
```

---

## Tips และ Best Practices

### 1. Normalize State

```jsx
// ❌ ไม่ดี - nested arrays
{
  orders: [
    { id: 1, items: [{ id: 1, name: '...' }] }
  ]
}

// ✅ ดี - normalized (flat)
{
  orders: { ids: [1], entities: { 1: { id: 1, itemIds: [1] } } },
  items: { ids: [1], entities: { 1: { id: 1, name: '...' } } }
}
```

### 2. ใช้ createEntityAdapter

```jsx
import { createEntityAdapter, createSlice } from '@reduxjs/toolkit'

const productsAdapter = createEntityAdapter()

const productSlice = createSlice({
  name: 'products',
  initialState: productsAdapter.getInitialState({ loading: false }),
  reducers: {
    productAdded: productsAdapter.addOne,
    productsReceived: productsAdapter.setAll,
    productUpdated: productsAdapter.updateOne,
    productRemoved: productsAdapter.removeOne,
  },
})

// Selectors ฟรี!
export const {
  selectAll: selectAllProducts,
  selectById: selectProductById,
  selectIds: selectProductIds,
} = productsAdapter.getSelectors(state => state.products)
```

---

## Quiz - Part 26

**ข้อ 1**: createSlice รวมสิ่งใดไว้ด้วยกัน?
- a) Component และ Styles
- b) Actions, Action Creators, และ Reducer
- c) API calls และ State
- d) Router และ State

**ข้อ 2**: createAsyncThunk สร้าง action types อะไรบ้าง?
- a) pending, fulfilled, rejected
- b) start, success, error
- c) loading, done, failed
- d) begin, end, error

**ข้อ 3**: RTK Query ทำอะไรให้อัตโนมัติ?
- a) Render UI
- b) Caching, refetching, loading/error states
- c) Route navigation
- d) Form validation

**ข้อ 4**: `invalidatesTags` ใน RTK Query ทำงานอย่างไร?
- a) ลบ cache ทั้งหมด
- b) บอกให้ refetch queries ที่มี tag นั้น
- c) ป้องกัน duplicate requests
- d) เพิ่ม tag ให้ response

**คำตอบ**: 1-b, 2-a, 3-b, 4-b

---

## สรุป Part 26

ใน Part นี้คุณได้เรียนรู้:
- ✅ Redux concepts (Store, Action, Reducer, Dispatch)
- ✅ Redux Toolkit ทำให้ Redux ง่ายขึ้นอย่างไร
- ✅ configureStore และ Provider
- ✅ createSlice พร้อม Immer
- ✅ useSelector และ useDispatch
- ✅ createAsyncThunk
- ✅ RTK Query เบื้องต้น
- ✅ Shopping Cart ครบ

---

## Part ถัดไป

➡️ **[Part 27: Zustand State Management](./part-27-zustand-state-management.md)**
- Zustand vs Redux
- Create Store
- Async Actions
- Immer Integration
