# Part 28: TanStack Query (React Query) - Server State Management

**Step 731-770** | ระดับ: สูง | เวลาเรียน: 4-5 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [TanStack Query คืออะไร?](#tanstack-query-คืออะไร)
2. [QueryClient Setup](#queryclient-setup)
3. [useQuery Hook](#usequery-hook)
4. [useMutation Hook](#usemutation-hook)
5. [Caching และ Stale Time](#caching-และ-stale-time)
6. [Refetching Strategies](#refetching-strategies)
7. [Optimistic Updates](#optimistic-updates)
8. [Infinite Queries](#infinite-queries)
9. [Prefetching](#prefetching)
10. [Quiz](#quiz)

---

## Step 731-735: TanStack Query คืออะไร?

TanStack Query (เดิมชื่อ React Query) เป็น library สำหรับจัดการ "server state" - ข้อมูลที่มาจาก API

### Server State vs Client State

```
Client State: 
  - อยู่ใน memory ของ browser
  - User เป็นคนควบคุม
  - เช่น: form input, modal open/close, theme

Server State:
  - อยู่บน server
  - หลาย users แชร์
  - ต้องดึงมา, cache, sync
  - เช่น: products, users, posts
```

### ปัญหาที่ TanStack Query แก้

```
ไม่มี TanStack Query:
- เขียน loading/error state เอง
- ต้องจำ cache เอง
- Re-fetch logic ซับซ้อน
- Stale data ยากจัดการ

มี TanStack Query:
- Auto loading/error/success states
- Smart caching อัตโนมัติ
- Background refetching
- Window focus refetch
- Infinite queries built-in
```

---

## Step 736-738: QueryClient Setup

### การติดตั้ง

```bash
npm install @tanstack/react-query
npm install @tanstack/react-query-devtools  # สำหรับ debug
```

### Setup QueryClient

```jsx
// main.jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,    // 5 นาที - ข้อมูลยัง fresh
      gcTime: 30 * 60 * 1000,      // 30 นาที - เก็บ cache (เดิมชื่อ cacheTime)
      retry: 3,                      // retry 3 ครั้งเมื่อ error
      refetchOnWindowFocus: true,    // refetch เมื่อ focus กลับมา
      refetchOnReconnect: true,      // refetch เมื่อ reconnect
    },
    mutations: {
      retry: 0,  // ไม่ retry mutations
    },
  },
})

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      {/* DevTools - เฉพาะ development */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
)
```

---

## Step 739-745: useQuery Hook

### การใช้งานพื้นฐาน

```jsx
import { useQuery } from '@tanstack/react-query'
import { productService } from '../services/productService'

function ProductList() {
  const {
    data,          // ข้อมูลที่ได้
    isLoading,     // true ระหว่างโหลดครั้งแรก
    isFetching,    // true เมื่อกำลัง fetch (รวม background)
    isError,       // true เมื่อ error
    error,         // error object
    isSuccess,     // true เมื่อสำเร็จ
    refetch,       // function เรียก refetch ด้วยตัวเอง
    status,        // 'pending' | 'error' | 'success'
  } = useQuery({
    queryKey: ['products'],           // unique key สำหรับ cache
    queryFn: () => productService.getAll(),  // function ดึงข้อมูล
  })

  if (isLoading) return <LoadingSpinner />
  if (isError) return <ErrorMessage message={error.message} />

  return (
    <div>
      {isFetching && <span>กำลังอัพเดทข้อมูล...</span>}
      {data?.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

### queryKey - สำคัญมาก

```jsx
// queryKey คือ unique identifier ของแต่ละ query
// ใช้ cache แยกตาม key

// ข้อมูลทั่วไป
useQuery({ queryKey: ['products'] })
useQuery({ queryKey: ['users'] })

// ข้อมูล 1 รายการ
useQuery({ queryKey: ['products', 5] })  // product id 5
useQuery({ queryKey: ['users', 'me'] })

// ข้อมูลกับ parameters
useQuery({ queryKey: ['products', { page: 1, category: 'electronics' }] })
useQuery({ queryKey: ['search', query] })  // ดึงใหม่เมื่อ query เปลี่ยน
```

### Query กับ Parameters

```jsx
function Products({ page, category }) {
  const { data, isLoading } = useQuery({
    queryKey: ['products', { page, category }],  // key เปลี่ยนเมื่อ params เปลี่ยน
    queryFn: () => productService.getAll({ page, category }),
    // ไม่ fetch ถ้า page เป็น 0
    enabled: page > 0,
  })

  return (/* JSX */)
}
```

### Fetch Single Item

```jsx
function ProductDetail({ id }) {
  const { data: product, isLoading, isError } = useQuery({
    queryKey: ['products', id],
    queryFn: () => productService.getById(id),
    enabled: !!id,  // ไม่ fetch ถ้าไม่มี id
    // Initial data จาก list cache
    initialData: () => {
      return queryClient
        .getQueryData(['products'])
        ?.find(p => p.id === Number(id))
    },
  })

  if (isLoading) return <LoadingSpinner />
  if (isError) return <div>ไม่พบสินค้า</div>

  return (
    <div>
      <h1>{product.name}</h1>
      <p>฿{product.price}</p>
    </div>
  )
}
```

### Parallel Queries

```jsx
function Dashboard() {
  // ทั้งสาม queries ทำงานพร้อมกัน
  const productsQuery = useQuery({
    queryKey: ['products'],
    queryFn: productService.getAll,
  })
  
  const usersQuery = useQuery({
    queryKey: ['users'],
    queryFn: userService.getAll,
  })
  
  const ordersQuery = useQuery({
    queryKey: ['orders'],
    queryFn: orderService.getAll,
  })

  if (productsQuery.isLoading || usersQuery.isLoading || ordersQuery.isLoading) {
    return <LoadingSpinner />
  }

  return (
    <div>
      <p>สินค้า: {productsQuery.data?.length}</p>
      <p>ผู้ใช้: {usersQuery.data?.length}</p>
      <p>คำสั่งซื้อ: {ordersQuery.data?.length}</p>
    </div>
  )
}
```

### Dependent Query

```jsx
function UserPosts({ userId }) {
  // Query 1: ดึง user
  const { data: user } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => userService.getById(userId),
  })

  // Query 2: ดึง posts ของ user (รอ user ก่อน)
  const { data: posts } = useQuery({
    queryKey: ['users', userId, 'posts'],
    queryFn: () => postService.getByUserId(user.id),
    enabled: !!user,  // รอจนได้ user
  })

  return (/* JSX */)
}
```

---

## Step 746-752: useMutation Hook

### Create, Update, Delete

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function CreateProductForm() {
  const queryClient = useQueryClient()

  const createMutation = useMutation({
    mutationFn: (data) => productService.create(data),
    
    onSuccess: (newProduct) => {
      // Invalidate cache → refetch products
      queryClient.invalidateQueries({ queryKey: ['products'] })
      
      // หรือ update cache โดยตรง
      queryClient.setQueryData(['products'], (old) => [newProduct, ...old])
      
      alert('สร้างสินค้าสำเร็จ!')
    },
    
    onError: (error) => {
      alert(`เกิดข้อผิดพลาด: ${error.message}`)
    },
    
    onSettled: () => {
      // เรียกเสมอ ไม่ว่าจะ success หรือ error
      console.log('mutation settled')
    },
  })

  const handleSubmit = (formData) => {
    createMutation.mutate(formData)
    // หรือ async:
    // await createMutation.mutateAsync(formData)
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* fields */}
      <button type="submit" disabled={createMutation.isPending}>
        {createMutation.isPending ? 'กำลังบันทึก...' : 'บันทึก'}
      </button>
      {createMutation.isError && (
        <p style={{ color: 'red' }}>{createMutation.error.message}</p>
      )}
    </form>
  )
}
```

### Delete Mutation

```jsx
function ProductCard({ product }) {
  const queryClient = useQueryClient()

  const deleteMutation = useMutation({
    mutationFn: (id) => productService.delete(id),
    onSuccess: (_, id) => {
      queryClient.setQueryData(['products'], (old) =>
        old?.filter(p => p.id !== id)
      )
    },
  })

  return (
    <div>
      <h3>{product.name}</h3>
      <button
        onClick={() => deleteMutation.mutate(product.id)}
        disabled={deleteMutation.isPending}
        style={{ color: 'red' }}
      >
        {deleteMutation.isPending ? 'กำลังลบ...' : 'ลบ'}
      </button>
    </div>
  )
}
```

---

## Step 753-756: Caching และ Stale Time

### Cache Lifecycle

```
Query executed
→ Data is "fresh" (ตาม staleTime)
→ Data becomes "stale"
→ Next access triggers background refetch
→ Cache removed after gcTime (ไม่มีใคร subscribe)
```

### Stale Time Configuration

```jsx
// Per query
const { data } = useQuery({
  queryKey: ['products'],
  queryFn: productService.getAll,
  staleTime: 10 * 60 * 1000,  // 10 นาที - ไม่ refetch ถ้าข้อมูลยัง fresh
})

// ข้อมูลที่ไม่ค่อยเปลี่ยน
useQuery({
  queryKey: ['categories'],
  queryFn: categoryService.getAll,
  staleTime: Infinity,  // ไม่ stale เลย - โหลดครั้งเดียว
})

// ข้อมูล realtime
useQuery({
  queryKey: ['notifications'],
  queryFn: notificationService.getAll,
  staleTime: 0,  // stale ทันที - refetch ทุกครั้ง
  refetchInterval: 30000,  // refetch ทุก 30 วินาที
})
```

---

## Step 757-759: Refetching Strategies

```jsx
useQuery({
  queryKey: ['products'],
  queryFn: productService.getAll,
  
  // Auto refetch
  refetchOnWindowFocus: true,       // refetch เมื่อ tab กลับมา active
  refetchOnReconnect: true,          // refetch เมื่อ internet กลับมา
  refetchOnMount: true,              // refetch เมื่อ component mount
  refetchInterval: 30000,            // polling ทุก 30 วินาที
  refetchIntervalInBackground: false, // หยุด polling เมื่อ tab ไม่ active
})

// Manual refetch
function ProductList() {
  const { data, refetch, isFetching } = useQuery({
    queryKey: ['products'],
    queryFn: productService.getAll,
  })

  return (
    <div>
      <button onClick={() => refetch()} disabled={isFetching}>
        {isFetching ? 'กำลังโหลด...' : '🔄 รีเฟรช'}
      </button>
      {/* list */}
    </div>
  )
}
```

---

## Step 760-763: Optimistic Updates

Optimistic Updates อัพเดท UI ทันที ก่อน server confirm

```jsx
function ProductList() {
  const queryClient = useQueryClient()
  
  const likeMutation = useMutation({
    mutationFn: ({ id, liked }) => productService.toggleLike(id, liked),
    
    // ทำก่อน mutation
    onMutate: async ({ id, liked }) => {
      // ยกเลิก ongoing queries เพื่อป้องกัน conflict
      await queryClient.cancelQueries({ queryKey: ['products'] })
      
      // บันทึก state เก่า
      const previousProducts = queryClient.getQueryData(['products'])
      
      // อัพเดท cache ทันที (optimistic)
      queryClient.setQueryData(['products'], (old) =>
        old?.map(p => p.id === id ? { ...p, liked, likeCount: liked ? p.likeCount + 1 : p.likeCount - 1 } : p)
      )
      
      // return context สำหรับ rollback
      return { previousProducts }
    },
    
    // ถ้า error - rollback
    onError: (err, variables, context) => {
      queryClient.setQueryData(['products'], context.previousProducts)
    },
    
    // ทุกครั้ง - sync กับ server
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
    },
  })

  return (
    <div>
      {/* products list */}
    </div>
  )
}
```

---

## Step 764-767: Infinite Queries

```jsx
import { useInfiniteQuery } from '@tanstack/react-query'

function InfiniteProductList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
  } = useInfiniteQuery({
    queryKey: ['products', 'infinite'],
    queryFn: ({ pageParam }) =>
      productService.getAll({ page: pageParam, limit: 10 }),
    
    initialPageParam: 1,
    
    getNextPageParam: (lastPage, allPages) => {
      // คืน page ถัดไป หรือ undefined ถ้าหมดแล้ว
      if (lastPage.currentPage < lastPage.totalPages) {
        return lastPage.currentPage + 1
      }
      return undefined
    },
  })

  if (isLoading) return <LoadingSpinner />
  if (isError) return <ErrorMessage />

  // data.pages เป็น array ของแต่ละ page
  const allProducts = data.pages.flatMap(page => page.data)

  return (
    <div>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '1rem' }}>
        {allProducts.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      <div style={{ textAlign: 'center', padding: '2rem' }}>
        {hasNextPage ? (
          <button
            onClick={() => fetchNextPage()}
            disabled={isFetchingNextPage}
          >
            {isFetchingNextPage ? 'กำลังโหลด...' : 'โหลดเพิ่มเติม'}
          </button>
        ) : (
          <p>แสดงสินค้าทั้งหมดแล้ว</p>
        )}
      </div>
    </div>
  )
}
```

### Intersection Observer + Infinite Query

```jsx
import { useRef, useEffect } from 'react'
import { useInfiniteQuery } from '@tanstack/react-query'

function AutoLoadProductList() {
  const loadMoreRef = useRef(null)

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['products', 'infinite'],
    queryFn: ({ pageParam }) => productService.getAll({ page: pageParam }),
    initialPageParam: 1,
    getNextPageParam: (lastPage) =>
      lastPage.currentPage < lastPage.totalPages
        ? lastPage.currentPage + 1
        : undefined,
  })

  // Auto load เมื่อ scroll ถึงล่างสุด
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting && hasNextPage && !isFetchingNextPage) {
          fetchNextPage()
        }
      },
      { threshold: 0.1 }
    )

    if (loadMoreRef.current) {
      observer.observe(loadMoreRef.current)
    }

    return () => observer.disconnect()
  }, [hasNextPage, isFetchingNextPage, fetchNextPage])

  const products = data?.pages.flatMap(p => p.data) ?? []

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}

      {/* Trigger element */}
      <div ref={loadMoreRef} style={{ height: 20 }}>
        {isFetchingNextPage && <LoadingSpinner />}
      </div>

      {!hasNextPage && products.length > 0 && (
        <p style={{ textAlign: 'center' }}>แสดงทั้งหมดแล้ว</p>
      )}
    </div>
  )
}
```

---

## Step 768-770: Prefetching

โหลดข้อมูลล่วงหน้าก่อนที่ user จะ navigate

```jsx
import { useQueryClient } from '@tanstack/react-query'

function ProductList({ products }) {
  const queryClient = useQueryClient()

  const prefetchProduct = (id) => {
    queryClient.prefetchQuery({
      queryKey: ['products', id],
      queryFn: () => productService.getById(id),
      staleTime: 10 * 60 * 1000,  // ใช้ cached ถ้าไม่ stale
    })
  }

  return (
    <div>
      {products.map(product => (
        <Link
          key={product.id}
          to={`/products/${product.id}`}
          onMouseEnter={() => prefetchProduct(product.id)}  // prefetch เมื่อ hover
        >
          <ProductCard product={product} />
        </Link>
      ))}
    </div>
  )
}
```

### Server-side Prefetching (Next.js)

```jsx
// pages/products/index.js (Next.js)
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query'

export async function getServerSideProps() {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['products'],
    queryFn: productService.getAll,
  })

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  }
}

export default function ProductsPage({ dehydratedState }) {
  return (
    <HydrationBoundary state={dehydratedState}>
      <ProductList />
    </HydrationBoundary>
  )
}
```

---

## Tips และ Best Practices

### 1. Query Key Factory

```jsx
// ✅ ดี - organized query keys
export const productKeys = {
  all: ['products'],
  lists: () => [...productKeys.all, 'list'],
  list: (filters) => [...productKeys.lists(), filters],
  details: () => [...productKeys.all, 'detail'],
  detail: (id) => [...productKeys.details(), id],
}

// ใช้งาน
useQuery({ queryKey: productKeys.list({ page: 1 }) })
useQuery({ queryKey: productKeys.detail(id) })

// Invalidate เฉพาะ detail queries
queryClient.invalidateQueries({ queryKey: productKeys.details() })
```

### 2. ใช้ select สำหรับ transform data

```jsx
const { data: productNames } = useQuery({
  queryKey: ['products'],
  queryFn: productService.getAll,
  select: (data) => data.map(p => p.name),  // transform ใน query
})

// Component render ใหม่แค่เมื่อ names เปลี่ยน ไม่ใช่ทั้ง array
```

### 3. Error Boundary กับ TanStack Query

```jsx
useQuery({
  queryKey: ['products'],
  queryFn: productService.getAll,
  throwOnError: true,  // throw error ไปให้ ErrorBoundary จัดการ
})
```

---

## Quiz - Part 28

**ข้อ 1**: `staleTime` ใน TanStack Query หมายความว่าอะไร?
- a) เวลาที่ cache จะถูกลบ
- b) ช่วงเวลาที่ข้อมูลยัง "fresh" ไม่ต้องการ refetch
- c) Timeout ของ request
- d) เวลา retry

**ข้อ 2**: `queryKey` สำคัญอย่างไร?
- a) ใช้ระบุ component
- b) เป็น unique identifier สำหรับ cache
- c) ส่งไปกับ request
- d) กำหนดลำดับความสำคัญ

**ข้อ 3**: Optimistic Update คืออะไร?
- a) Update server ก่อน UI
- b) Update UI ทันทีก่อน server confirm
- c) Cache data ไว้ถาวร
- d) Skip การ validate

**ข้อ 4**: `invalidateQueries` ทำอะไร?
- a) ลบ cache ทั้งหมด
- b) Mark queries เป็น stale → trigger refetch
- c) ยกเลิก ongoing requests
- d) Reset error state

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 28

ใน Part นี้คุณได้เรียนรู้:
- ✅ TanStack Query concepts
- ✅ QueryClient Setup
- ✅ useQuery (single, parallel, dependent)
- ✅ useMutation (create, update, delete)
- ✅ Caching และ Stale Time
- ✅ Refetching Strategies
- ✅ Optimistic Updates
- ✅ Infinite Queries + Auto-load
- ✅ Prefetching

---

## Part ถัดไป

➡️ **[Part 29: React Hook Form](./part-29-forms-advanced.md)**
- React Hook Form
- Zod Validation
- Multi-step Forms
- File Upload
