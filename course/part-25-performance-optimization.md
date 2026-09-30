# Part 25: Performance Optimization - การเพิ่มประสิทธิภาพ

**Step 626-660** | ระดับ: สูง | เวลาเรียน: 4-5 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [React.memo](#reactmemo)
2. [useMemo ลึกขึ้น](#usememo-ลึกขึ้น)
3. [useCallback ลึกขึ้น](#usecallback-ลึกขึ้น)
4. [Lazy Loading Images](#lazy-loading-images)
5. [Code Splitting](#code-splitting)
6. [Bundle Analysis](#bundle-analysis)
7. [Lighthouse Optimization](#lighthouse-optimization)
8. [React DevTools Profiler](#react-devtools-profiler)
9. [Virtualization (react-window)](#virtualization-react-window)
10. [Debounce และ Throttle](#debounce-และ-throttle)
11. [Quiz](#quiz)

---

## Step 626-630: React.memo

`React.memo` ป้องกัน component re-render เมื่อ props ไม่เปลี่ยน

### ทำความเข้าใจ Re-render

```
ปกติ:
Parent re-renders → ทุก Child re-renders
(แม้ props ของ Child จะไม่เปลี่ยน)

กับ React.memo:
Parent re-renders → เช็ค props ของ Child
  - props เปลี่ยน → Child re-renders
  - props ไม่เปลี่ยน → ข้ามการ render
```

### การใช้งาน React.memo

```jsx
// ❌ ไม่ดี - re-render ทุกครั้งที่ parent render
function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard rendered:', product.id)
  
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
      <button onClick={() => onAddToCart(product)}>
        เพิ่มใส่ตะกร้า
      </button>
    </div>
  )
}

// ✅ ดี - re-render เฉพาะเมื่อ props เปลี่ยน
const ProductCard = React.memo(function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard rendered:', product.id)
  
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
      <button onClick={() => onAddToCart(product)}>
        เพิ่มใส่ตะกร้า
      </button>
    </div>
  )
})

export default ProductCard
```

### Custom Comparison Function

```jsx
const ProductCard = React.memo(
  function ProductCard({ product, onAddToCart }) {
    return (/* JSX */)
  },
  // Custom comparison - return true = ไม่ re-render, false = re-render
  (prevProps, nextProps) => {
    return (
      prevProps.product.id === nextProps.product.id &&
      prevProps.product.price === nextProps.product.price &&
      prevProps.product.name === nextProps.product.name
    )
  }
)
```

### เมื่อไหร่ควรใช้ React.memo

```jsx
// ✅ ควรใช้เมื่อ:
// 1. Component render บ่อยกับ props เดิม
// 2. Component มีการ render ที่ expensive (complex logic, large lists)
// 3. Component อยู่ในลูปที่ render หลายครั้ง

// ❌ ไม่ควรใช้เมื่อ:
// 1. Component เล็กและ render เร็ว
// 2. Props เปลี่ยนทุกครั้งอยู่แล้ว
// 3. เพิ่มโดยไม่มีการวัด performance
```

---

## Step 631-635: useMemo ลึกขึ้น

`useMemo` cache ผลลัพธ์ของการคำนวณ ป้องกันการคำนวณซ้ำเมื่อ dependencies ไม่เปลี่ยน

### ตัวอย่างที่ใช้บ่อย

```jsx
// การ filter และ sort ข้อมูล
function ProductList({ products, filterText, sortBy }) {
  // ❌ ไม่ดี - คำนวณทุกครั้งที่ render
  const filteredAndSorted = products
    .filter(p => p.name.toLowerCase().includes(filterText.toLowerCase()))
    .sort((a, b) => {
      if (sortBy === 'price_asc') return a.price - b.price
      if (sortBy === 'price_desc') return b.price - a.price
      return a.name.localeCompare(b.name)
    })

  // ✅ ดี - คำนวณใหม่เฉพาะเมื่อ dependencies เปลี่ยน
  const filteredAndSorted = useMemo(() => {
    return products
      .filter(p => p.name.toLowerCase().includes(filterText.toLowerCase()))
      .sort((a, b) => {
        if (sortBy === 'price_asc') return a.price - b.price
        if (sortBy === 'price_desc') return b.price - a.price
        return a.name.localeCompare(b.name)
      })
  }, [products, filterText, sortBy])

  return (
    <div>
      {filteredAndSorted.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

### useMemo กับ Object References

```jsx
// ❌ ปัญหา: object ใหม่ทุกครั้ง = child re-render ทุกครั้ง
function Parent() {
  const config = { theme: 'dark', language: 'th' }  // ใหม่ทุกครั้ง!
  return <Child config={config} />
}

// ✅ แก้: memo config
function Parent() {
  const config = useMemo(() => ({
    theme: 'dark',
    language: 'th',
  }), [])  // dependencies ว่าง = สร้างครั้งเดียว

  return <Child config={config} />
}
```

### Complex Calculations

```jsx
function Statistics({ data }) {
  const stats = useMemo(() => {
    if (!data.length) return null

    const sum = data.reduce((acc, val) => acc + val, 0)
    const mean = sum / data.length
    const sorted = [...data].sort((a, b) => a - b)
    const median = sorted.length % 2 === 0
      ? (sorted[sorted.length / 2 - 1] + sorted[sorted.length / 2]) / 2
      : sorted[Math.floor(sorted.length / 2)]
    
    const variance = data.reduce((acc, val) => acc + Math.pow(val - mean, 2), 0) / data.length
    const stdDev = Math.sqrt(variance)

    return { sum, mean, median, stdDev, min: sorted[0], max: sorted[sorted.length - 1] }
  }, [data])

  if (!stats) return <p>ไม่มีข้อมูล</p>

  return (
    <div>
      <p>ผลรวม: {stats.sum}</p>
      <p>เฉลี่ย: {stats.mean.toFixed(2)}</p>
      <p>มัธยฐาน: {stats.median}</p>
      <p>ส่วนเบี่ยงเบนมาตรฐาน: {stats.stdDev.toFixed(2)}</p>
    </div>
  )
}
```

---

## Step 636-640: useCallback ลึกขึ้น

`useCallback` cache function reference ป้องกัน function ใหม่ทุกครั้ง

### ปัญหา Function Reference

```jsx
// ❌ ปัญหา
function Parent() {
  const handleClick = () => {  // function ใหม่ทุก render
    console.log('clicked')
  }

  return <MemoizedChild onClick={handleClick} />
  // React.memo ใน Child จะไม่ work!
  // เพราะ handleClick เป็น function ใหม่เสมอ
}

// ✅ แก้
function Parent() {
  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])  // ไม่มี dependencies = function เดิม

  return <MemoizedChild onClick={handleClick} />
}
```

### useCallback กับ Dependencies

```jsx
function ProductList({ categoryId }) {
  const [products, setProducts] = useState([])

  // ✅ สร้าง function ใหม่เฉพาะเมื่อ categoryId เปลี่ยน
  const fetchProducts = useCallback(async () => {
    const data = await productService.getByCategory(categoryId)
    setProducts(data)
  }, [categoryId])

  useEffect(() => {
    fetchProducts()
  }, [fetchProducts])

  return (/* JSX */)
}
```

### useCallback กับ useReducer Pattern

```jsx
// ✅ Pattern ที่ดี: ใช้ dispatch แทน callback
function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState)

  // dispatch ไม่เปลี่ยน → ไม่ต้อง useCallback
  const addToCart = useCallback((product) => {
    dispatch({ type: 'ADD_ITEM', payload: product })
  }, [dispatch])  // dispatch stable

  const removeFromCart = useCallback((id) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id })
  }, [dispatch])

  return (
    <div>
      {state.items.map(item => (
        <CartItem
          key={item.id}
          item={item}
          onRemove={removeFromCart}
        />
      ))}
    </div>
  )
}
```

---

## Step 641-644: Lazy Loading Images

### Native Lazy Loading

```jsx
// ✅ HTML5 native lazy loading
<img 
  src="product.jpg" 
  alt="Product"
  loading="lazy"
  width={300}
  height={200}
/>
```

### Intersection Observer Hook

```jsx
// hooks/useIntersectionObserver.js
import { useState, useEffect, useRef } from 'react'

function useIntersectionObserver(options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false)
  const ref = useRef(null)

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting)
    }, options)

    if (ref.current) {
      observer.observe(ref.current)
    }

    return () => observer.disconnect()
  }, [])

  return { ref, isIntersecting }
}

export default useIntersectionObserver
```

### LazyImage Component

```jsx
// components/LazyImage.jsx
import { useState } from 'react'
import useIntersectionObserver from '../hooks/useIntersectionObserver'

function LazyImage({ src, alt, placeholder, ...props }) {
  const [loaded, setLoaded] = useState(false)
  const { ref, isIntersecting } = useIntersectionObserver({
    threshold: 0.1,
    rootMargin: '100px',  // โหลดล่วงหน้า 100px ก่อนเห็น
  })

  return (
    <div ref={ref} style={{ position: 'relative', ...props.containerStyle }}>
      {/* Placeholder */}
      {!loaded && (
        <div style={{
          position: 'absolute',
          inset: 0,
          backgroundColor: '#e2e8f0',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          animation: 'pulse 1.5s infinite',
        }}>
          {placeholder || <span style={{ color: '#a0aec0' }}>กำลังโหลด...</span>}
        </div>
      )}

      {/* Actual image - โหลดเฉพาะเมื่อเข้า viewport */}
      {isIntersecting && (
        <img
          src={src}
          alt={alt}
          onLoad={() => setLoaded(true)}
          style={{
            opacity: loaded ? 1 : 0,
            transition: 'opacity 0.3s ease',
            ...props.style,
          }}
          {...props}
        />
      )}

      <style>{`
        @keyframes pulse {
          0%, 100% { opacity: 1; }
          50% { opacity: 0.5; }
        }
      `}</style>
    </div>
  )
}

export default LazyImage
```

---

## Step 645-648: Bundle Analysis

### Vite Bundle Analyzer

```bash
npm install --save-dev rollup-plugin-visualizer

# build แล้วดู stats
npm run build
# จะเปิด browser แสดง bundle visualization
```

```js
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer'

export default {
  plugins: [
    visualizer({
      open: true,
      filename: 'dist/stats.html',
      gzipSize: true,
      brotliSize: true,
    }),
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            if (id.includes('react') || id.includes('react-dom')) {
              return 'react-vendor'
            }
            if (id.includes('react-router')) {
              return 'router'
            }
            if (id.includes('@tanstack')) {
              return 'tanstack'
            }
            return 'vendor'
          }
        },
      },
    },
  },
}
```

### Tree Shaking

```jsx
// ❌ ไม่ดี - import ทั้ง library
import _ from 'lodash'
const sorted = _.sortBy(array, 'name')

// ✅ ดี - import เฉพาะที่ใช้
import sortBy from 'lodash/sortBy'
const sorted = sortBy(array, 'name')
```

---

## Step 649-652: Lighthouse Optimization

### Core Web Vitals

| Metric | ดี | ต้องปรับ | แย่ |
|--------|-----|---------|-----|
| LCP (Largest Contentful Paint) | < 2.5s | 2.5-4s | > 4s |
| FID (First Input Delay) | < 100ms | 100-300ms | > 300ms |
| CLS (Cumulative Layout Shift) | < 0.1 | 0.1-0.25 | > 0.25 |
| FCP (First Contentful Paint) | < 1.8s | 1.8-3s | > 3s |
| TTFB (Time to First Byte) | < 800ms | 800ms-1.8s | > 1.8s |

### การแก้ปัญหา LCP

```jsx
// ✅ Preload critical images
<head>
  <link rel="preload" as="image" href="/hero.jpg" />
</head>

// ✅ ระบุ width/height เพื่อป้องกัน layout shift
<img src="hero.jpg" alt="Hero" width={1200} height={600} />

// ✅ ใช้ priority สำหรับ above-the-fold images
// (Next.js)
<Image src="/hero.jpg" alt="Hero" priority />
```

### การแก้ปัญหา CLS

```jsx
// ❌ ปัญหา - ไม่มี reserved space
<div>
  <img src="product.jpg" alt="Product" />  {/* jumps เมื่อโหลด */}
</div>

// ✅ แก้ - reserve space ด้วย aspect-ratio
<div style={{ aspectRatio: '16/9', backgroundColor: '#f0f0f0' }}>
  <img src="product.jpg" alt="Product" style={{ width: '100%', height: '100%', objectFit: 'cover' }} />
</div>
```

---

## Step 653-655: React DevTools Profiler

### การใช้ Profiler

```jsx
// 1. ติดตั้ง React DevTools extension ใน browser
// 2. เปิด DevTools → Profiler tab
// 3. กด Record
// 4. ทำ action ที่ต้องการวัด
// 5. กด Stop
// 6. ดู flame graph และ ranked chart
```

### Profiler API

```jsx
import { Profiler } from 'react'

function onRenderCallback(
  id,            // "id" prop ของ Profiler
  phase,         // "mount" หรือ "update"
  actualDuration, // เวลา render จริง (ms)
  baseDuration,   // เวลา render ที่ประเมิน (ms)
  startTime,     // เวลาเริ่ม render
  commitTime,    // เวลา commit
) {
  if (actualDuration > 10) {
    console.warn(`Slow render: ${id} took ${actualDuration}ms`)
  }
}

function App() {
  return (
    <Profiler id="ProductList" onRender={onRenderCallback}>
      <ProductList />
    </Profiler>
  )
}
```

---

## Step 656-658: Virtualization (react-window)

Virtualization render เฉพาะ items ที่อยู่ใน viewport ทำให้ list ยาวๆ เร็วขึ้นมาก

### การติดตั้ง

```bash
npm install react-window
```

### FixedSizeList

```jsx
import { FixedSizeList } from 'react-window'

const ITEMS = Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `รายการ ${i + 1}`,
}))

// Row component
const Row = ({ index, style, data }) => (
  <div style={{
    ...style,
    display: 'flex',
    alignItems: 'center',
    padding: '0 1rem',
    borderBottom: '1px solid #e2e8f0',
    backgroundColor: index % 2 === 0 ? 'white' : '#f7fafc',
  }}>
    {data[index].name}
  </div>
)

function VirtualizedList() {
  return (
    <FixedSizeList
      height={500}          // ความสูงของ container
      width="100%"
      itemCount={ITEMS.length}
      itemSize={50}          // ความสูงของแต่ละ row
      itemData={ITEMS}       // ส่งข้อมูลไปยัง Row
    >
      {Row}
    </FixedSizeList>
  )
}
```

### VariableSizeList

```jsx
import { VariableSizeList } from 'react-window'

// เมื่อแต่ละ row มีความสูงต่างกัน
const itemSizes = ITEMS.map(item => 
  item.description ? 80 : 50
)

function VariableList() {
  const getItemSize = (index) => itemSizes[index]

  return (
    <VariableSizeList
      height={500}
      width="100%"
      itemCount={ITEMS.length}
      itemSize={getItemSize}
    >
      {({ index, style }) => (
        <div style={style}>
          <h4>{ITEMS[index].name}</h4>
          {ITEMS[index].description && <p>{ITEMS[index].description}</p>}
        </div>
      )}
    </VariableSizeList>
  )
}
```

### FixedSizeGrid (2D)

```jsx
import { FixedSizeGrid } from 'react-window'

function ProductGrid({ products }) {
  const COLUMNS = 3
  const COLUMN_WIDTH = 250
  const ROW_HEIGHT = 300

  const Cell = ({ columnIndex, rowIndex, style }) => {
    const index = rowIndex * COLUMNS + columnIndex
    if (index >= products.length) return null

    const product = products[index]

    return (
      <div style={{ ...style, padding: '0.5rem' }}>
        <div style={{
          border: '1px solid #e2e8f0',
          borderRadius: '8px',
          padding: '1rem',
          height: '100%',
        }}>
          <h4>{product.name}</h4>
          <p>฿{product.price}</p>
        </div>
      </div>
    )
  }

  return (
    <FixedSizeGrid
      columnCount={COLUMNS}
      columnWidth={COLUMN_WIDTH}
      height={600}
      rowCount={Math.ceil(products.length / COLUMNS)}
      rowHeight={ROW_HEIGHT}
      width={800}
    >
      {Cell}
    </FixedSizeGrid>
  )
}
```

---

## Step 659-660: Debounce และ Throttle

### Debounce

```jsx
// hooks/useDebounce.js
import { useState, useEffect } from 'react'

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

export default useDebounce
```

```jsx
// ใช้งาน Search กับ Debounce
function SearchBox() {
  const [query, setQuery] = useState('')
  const debouncedQuery = useDebounce(query, 500)
  const [results, setResults] = useState([])

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([])
      return
    }

    // API call เรียกแค่เมื่อหยุดพิมพ์ 500ms
    searchAPI(debouncedQuery).then(setResults)
  }, [debouncedQuery])

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหา... (จะค้นหาหลังหยุดพิมพ์ 0.5 วินาที)"
      />
      <ul>
        {results.map(r => <li key={r.id}>{r.name}</li>)}
      </ul>
    </div>
  )
}
```

### Throttle

```jsx
// hooks/useThrottle.js
import { useState, useRef, useEffect } from 'react'

function useThrottle(value, interval = 200) {
  const [throttledValue, setThrottledValue] = useState(value)
  const lastUpdated = useRef(null)

  useEffect(() => {
    const now = Date.now()

    if (!lastUpdated.current || now - lastUpdated.current >= interval) {
      setThrottledValue(value)
      lastUpdated.current = now
    } else {
      const timer = setTimeout(() => {
        setThrottledValue(value)
        lastUpdated.current = Date.now()
      }, interval - (now - lastUpdated.current))

      return () => clearTimeout(timer)
    }
  }, [value, interval])

  return throttledValue
}

export default useThrottle
```

```jsx
// ใช้งาน Scroll Event กับ Throttle
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  const throttledScrollY = useThrottle(scrollY, 100)

  useEffect(() => {
    const handleScroll = () => setScrollY(window.scrollY)
    window.addEventListener('scroll', handleScroll)
    return () => window.removeEventListener('scroll', handleScroll)
  }, [])

  return <div>Scroll Position: {throttledScrollY}px</div>
}
```

---

## Tips และ Best Practices

### Performance Checklist

```
✅ ใช้ React.memo กับ components ที่ render บ่อย
✅ ใช้ useMemo กับ heavy calculations
✅ ใช้ useCallback กับ functions ที่ส่งไป memoized children
✅ Lazy load routes และ heavy components
✅ ระบุ key ที่ถูกต้องใน lists
✅ Virtualize lists ที่ยาวมาก (>1000 items)
✅ Debounce search inputs
✅ Optimize images (size, format, lazy loading)
✅ Analyze และ split bundles
✅ วัด performance ด้วย Profiler ก่อนและหลัง optimize
```

### อย่า Premature Optimize

```jsx
// ❌ ไม่ดี - optimize ทุกอย่างโดยไม่วัด
const value = useMemo(() => x + 1, [x])  // ไม่จำเป็นสำหรับ simple calculation
const fn = useCallback(() => {}, [])  // ไม่จำเป็นถ้าไม่ส่งไป memoized child

// ✅ ดี - วัดก่อน optimize
// 1. ใช้ React DevTools Profiler วัด render time
// 2. หา bottleneck จริงๆ
// 3. Apply optimization ที่จำเป็น
// 4. วัดอีกครั้งเพื่อยืนยัน
```

---

## Quiz - Part 25

**ข้อ 1**: `React.memo` จะป้องกัน re-render เมื่อใด?
- a) เมื่อ state ของ parent เปลี่ยน
- b) เมื่อ props ไม่เปลี่ยน
- c) เมื่อ context เปลี่ยน
- d) เสมอ

**ข้อ 2**: Virtualization ช่วยอะไร?
- a) Compress images
- b) Render เฉพาะ items ที่อยู่ใน viewport
- c) Cache API responses
- d) Reduce bundle size

**ข้อ 3**: Debounce ต่างจาก Throttle อย่างไร?
- a) Debounce = ทำงานช้าลง, Throttle = ทำงานเร็วขึ้น
- b) Debounce = ทำงานหลังหยุด, Throttle = ทำงานทุก N ms
- c) ไม่ต่างกัน
- d) Throttle ใช้กับ API เท่านั้น

**ข้อ 4**: CLS ย่อมาจากอะไร?
- a) Component Layout Size
- b) Cumulative Layout Shift
- c) Content Loading Speed
- d) Cache Level Score

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 25

ใน Part นี้คุณได้เรียนรู้:
- ✅ React.memo และ Custom Comparison
- ✅ useMemo สำหรับ heavy calculations
- ✅ useCallback สำหรับ stable function references
- ✅ Lazy Loading Images
- ✅ Bundle Analysis และ Code Splitting
- ✅ Core Web Vitals และ Lighthouse
- ✅ React DevTools Profiler
- ✅ Virtualization ด้วย react-window
- ✅ Debounce และ Throttle

---

## Part ถัดไป

➡️ **[Part 26: Redux Toolkit](./part-26-redux-toolkit.md)**
- Redux คืออะไร
- Redux Toolkit
- createSlice
- Async Thunks
