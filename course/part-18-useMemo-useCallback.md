# Part 18: useMemo และ useCallback

## Step 406-435 | ระดับ: กลาง-สูง

---

## สารบัญ

- [Step 406: Performance Optimization ใน React](#step-406-performance-optimization-ใน-react)
- [Step 407: useMemo คืออะไร?](#step-407-usememo-คืออะไร)
- [Step 408: useMemo Syntax และตัวอย่าง](#step-408-usememo-syntax-และตัวอย่าง)
- [Step 409: useCallback คืออะไร?](#step-409-usecallback-คืออะไร)
- [Step 410: useCallback Syntax และตัวอย่าง](#step-410-usecallback-syntax-และตัวอย่าง)
- [Step 411: React.memo Component Memoization](#step-411-reactmemo-component-memoization)
- [Step 412: useMemo กับ useCallback ต่างกันอย่างไร](#step-412-usememo-กับ-usecallback-ต่างกันอย่างไร)
- [Step 413: Profiler Tool](#step-413-profiler-tool)
- [Step 414: เมื่อไหรควร Optimize](#step-414-เมื่อไหรควร-optimize)
- [Step 415: ตัวอย่าง Before/After Optimization](#step-415-ตัวอย่าง-beforeafter-optimization)
- [Step 416: Anti-Patterns](#step-416-anti-patterns)
- [Step 417: useTransition](#step-417-usetransition)
- [Step 418: useDeferredValue](#step-418-usedeferredvalue)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 406: Performance Optimization ใน React

### ทำไม React Re-render?

```jsx
function App() {
  const [count, setCount] = useState(0);
  
  // ทุกครั้งที่ count เปลี่ยน → App re-renders → ทุก component ข้างล่าง re-renders
  return (
    <div>
      <Counter count={count} onIncrement={() => setCount(c => c + 1)} />
      <ExpensiveList /> {/* re-renders แม้ไม่เกี่ยวกับ count! */}
    </div>
  );
}
```

### เครื่องมือ Optimization

| เครื่องมือ | ใช้สำหรับ |
|---|---|
| `React.memo` | ป้องกัน component re-render ที่ไม่จำเป็น |
| `useMemo` | Cache ผลลัพธ์การคำนวณที่แพง |
| `useCallback` | Cache function reference |
| `useTransition` | ทำ UI transitions ไม่ blocking |
| `useDeferredValue` | Defer การ update ค่า |

---

## Step 407: useMemo คืออะไร?

`useMemo` cache ผลลัพธ์ของ computation และคำนวณใหม่เฉพาะเมื่อ dependencies เปลี่ยน

```jsx
const memoizedValue = useMemo(() => {
  return expensiveComputation(deps);
}, [deps]);
```

### ปัญหาที่ useMemo แก้ไข

```jsx
function ExpensiveComponent({ items, filter }) {
  // ❌ คำนวณใหม่ทุกครั้งที่ component re-renders
  const filteredItems = items
    .filter(item => item.category === filter)
    .sort((a, b) => b.price - a.price)
    .slice(0, 100);

  return <ItemList items={filteredItems} />;
}

// ✅ Cache ผลลัพธ์
function OptimizedComponent({ items, filter }) {
  const filteredItems = useMemo(() => {
    console.log('Computing filtered items...');
    return items
      .filter(item => item.category === filter)
      .sort((a, b) => b.price - a.price)
      .slice(0, 100);
  }, [items, filter]); // คำนวณใหม่เฉพาะเมื่อ items หรือ filter เปลี่ยน

  return <ItemList items={filteredItems} />;
}
```

---

## Step 408: useMemo Syntax และตัวอย่าง

```jsx
import { useMemo, useState } from 'react';

// ตัวอย่าง 1: Fibonacci (expensive computation)
function FibonacciCalculator() {
  const [n, setN] = useState(30);

  // ❌ ไม่ใช้ useMemo - คำนวณทุก render
  // const result = calculateFib(n); // ช้ามาก!

  // ✅ ใช้ useMemo - cache ผลลัพธ์
  const result = useMemo(() => {
    function fib(num) {
      if (num <= 1) return num;
      return fib(num - 1) + fib(num - 2);
    }
    return fib(n);
  }, [n]); // คำนวณใหม่เฉพาะเมื่อ n เปลี่ยน

  return (
    <div>
      <input
        type="number"
        value={n}
        onChange={e => setN(Number(e.target.value))}
        min="1"
        max="40"
      />
      <p>Fibonacci({n}) = {result}</p>
    </div>
  );
}

// ตัวอย่าง 2: Filtering และ Sorting รายการใหญ่
function ProductSearch({ products }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [minPrice, setMinPrice] = useState(0);

  const filteredAndSorted = useMemo(() => {
    let result = products;
    
    // Filter
    if (searchTerm) {
      const term = searchTerm.toLowerCase();
      result = result.filter(p => 
        p.name.toLowerCase().includes(term) ||
        p.description.toLowerCase().includes(term)
      );
    }
    
    // Price filter
    result = result.filter(p => p.price >= minPrice);
    
    // Sort
    result = [...result].sort((a, b) => {
      switch (sortBy) {
        case 'price-asc': return a.price - b.price;
        case 'price-desc': return b.price - a.price;
        case 'name': return a.name.localeCompare(b.name);
        case 'rating': return b.rating - a.rating;
        default: return 0;
      }
    });
    
    return result;
  }, [products, searchTerm, sortBy, minPrice]);

  const stats = useMemo(() => ({
    count: filteredAndSorted.length,
    avgPrice: filteredAndSorted.length > 0
      ? filteredAndSorted.reduce((sum, p) => sum + p.price, 0) / filteredAndSorted.length
      : 0
  }), [filteredAndSorted]);

  return (
    <div>
      <input
        placeholder="ค้นหา..."
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
      />
      <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
        <option value="name">ชื่อ</option>
        <option value="price-asc">ราคา: ต่ำ-สูง</option>
        <option value="price-desc">ราคา: สูง-ต่ำ</option>
        <option value="rating">คะแนน</option>
      </select>
      <p>แสดง {stats.count} รายการ | ราคาเฉลี่ย: ฿{stats.avgPrice.toFixed(0)}</p>
      <ProductGrid products={filteredAndSorted} />
    </div>
  );
}

// ตัวอย่าง 3: Object identity สำหรับ Context
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // ✅ useMemo ป้องกันการสร้าง object ใหม่ทุก render
  const contextValue = useMemo(() => ({
    user,
    isAdmin: user?.role === 'admin',
    logout: () => setUser(null)
  }), [user]); // สร้างใหม่เฉพาะเมื่อ user เปลี่ยน

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
}
```

---

## Step 409: useCallback คืออะไร?

`useCallback` cache function reference และสร้างใหม่เฉพาะเมื่อ dependencies เปลี่ยน

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(deps);
}, [deps]);
```

### ปัญหาที่ useCallback แก้ไข

```jsx
// ❌ ปัญหา: Function สร้างใหม่ทุก render
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // Function ใหม่ทุก render → ทำให้ ChildComponent re-render
  const handleClick = () => {
    console.log('clicked');
  };
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {/* ChildComponent re-renders ทุกครั้งที่ count เปลี่ยน */}
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

const ChildComponent = React.memo(({ onClick }) => {
  console.log('ChildComponent renders');
  return <button onClick={onClick}>Child Button</button>;
});

// ✅ แก้ไขด้วย useCallback
function OptimizedParent() {
  const [count, setCount] = useState(0);
  
  // ✅ Function reference เหมือนเดิม (ถ้า dependencies ไม่เปลี่ยน)
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []); // [] = ไม่มี dependencies → function ไม่เปลี่ยน
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {/* ChildComponent ไม่ re-render เมื่อ count เปลี่ยน */}
      <ChildComponent onClick={handleClick} />
    </div>
  );
}
```

---

## Step 410: useCallback Syntax และตัวอย่าง

```jsx
import { useCallback, useState, useEffect } from 'react';

// ตัวอย่าง 1: Event handlers
function SearchBar({ onSearch }) {
  const [term, setTerm] = useState('');
  
  // ✅ Cache handler ที่ใช้ onSearch
  const handleChange = useCallback((e) => {
    const value = e.target.value;
    setTerm(value);
    onSearch(value);
  }, [onSearch]); // re-create เมื่อ onSearch เปลี่ยน
  
  const handleClear = useCallback(() => {
    setTerm('');
    onSearch('');
  }, [onSearch]);
  
  return (
    <div>
      <input value={term} onChange={handleChange} placeholder="Search..." />
      <button onClick={handleClear}>Clear</button>
    </div>
  );
}

// ตัวอย่าง 2: useCallback กับ useEffect
function DataFetcher({ userId }) {
  const [data, setData] = useState(null);
  
  // ✅ Memoize fetch function
  const fetchData = useCallback(async () => {
    const res = await fetch(`/api/users/${userId}`);
    const json = await res.json();
    setData(json);
  }, [userId]); // re-create เมื่อ userId เปลี่ยน
  
  useEffect(() => {
    fetchData(); // ✅ fetchData มี stable reference
  }, [fetchData]); // ✅ effect รันเมื่อ fetchData เปลี่ยน (= เมื่อ userId เปลี่ยน)
  
  return <div>{JSON.stringify(data)}</div>;
}

// ตัวอย่าง 3: Action creators
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = useCallback((text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, done: false }]);
  }, []); // ไม่ต้องใส่ setTodos เพราะ setter stable เสมอ
  
  const toggleTodo = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  }, []);
  
  const deleteTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={toggleTodo}
          onDelete={deleteTodo}
        />
      ))}
    </div>
  );
}

// TodoItem wrapped in React.memo จะไม่ re-render เมื่อ parent re-renders
const TodoItem = React.memo(function({ todo, onToggle, onDelete }) {
  console.log(`TodoItem ${todo.id} renders`);
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
});
```

---

## Step 411: React.memo Component Memoization

`React.memo` ป้องกัน component re-render เมื่อ props ไม่เปลี่ยน

```jsx
// ✅ React.memo - ป้องกัน re-render ที่ไม่จำเป็น
const ExpensiveComponent = React.memo(function({ data, onUpdate }) {
  console.log('ExpensiveComponent renders'); // จะเห็น log น้อยลง
  
  return (
    <div>
      {data.map(item => (
        <ComplexItem key={item.id} item={item} onUpdate={onUpdate} />
      ))}
    </div>
  );
});

// Custom comparison function
const ProductCard = React.memo(
  function({ product, isSelected, onSelect }) {
    return (
      <div
        onClick={() => onSelect(product.id)}
        style={{ border: isSelected ? '2px solid blue' : '1px solid #ccc' }}
      >
        <img src={product.image} alt={product.name} />
        <h3>{product.name}</h3>
        <p>฿{product.price}</p>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Return true ถ้า props เท่ากัน (ไม่ต้อง re-render)
    return (
      prevProps.product.id === nextProps.product.id &&
      prevProps.product.price === nextProps.product.price &&
      prevProps.isSelected === nextProps.isSelected
    );
    // ไม่สนใจ onSelect เปลี่ยนหรือไม่
  }
);

// ✅ ตัวอย่างที่ดี: React.memo + useCallback ทำงานร่วมกัน
function ProductList({ products }) {
  const [selectedId, setSelectedId] = useState(null);

  // ✅ useCallback ทำให้ function reference stable
  const handleSelect = useCallback((id) => {
    setSelectedId(id);
  }, []);

  return (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '16px' }}>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          isSelected={product.id === selectedId}
          onSelect={handleSelect} // ✅ stable reference
        />
      ))}
    </div>
  );
}
```

---

## Step 412: useMemo กับ useCallback ต่างกันอย่างไร

```jsx
// useMemo: cache VALUE (ผลลัพธ์ของ computation)
const memoizedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);

// useCallback: cache FUNCTION reference
const memoizedFn = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// useCallback เทียบเท่ากับ useMemo ที่ return function
const memoizedFn2 = useMemo(() => {
  return () => doSomething(a, b);
}, [a, b]);

// ตัวอย่าง: ความแตกต่างในการใช้งาน
function Example({ data, multiplier }) {
  // useMemo: ใช้ผลลัพธ์โดยตรง
  const processedData = useMemo(() => {
    return data.map(item => item * multiplier);
  }, [data, multiplier]);
  
  // useCallback: ใช้เพื่อส่งเป็น event handler
  const handleProcess = useCallback(() => {
    const result = data.map(item => item * multiplier);
    console.log(result);
  }, [data, multiplier]);
  
  return (
    <div>
      <ul>{processedData.map((n, i) => <li key={i}>{n}</li>)}</ul>
      <button onClick={handleProcess}>Process</button>
    </div>
  );
}
```

---

## Step 413: Profiler Tool

React Profiler ช่วย identify ส่วนที่ render ช้า

```jsx
import { Profiler } from 'react';

// ใช้ Profiler ครอบ component ที่ต้องการวัด
function App() {
  const onRenderCallback = (
    id, // ชื่อ Profiler
    phase, // "mount" หรือ "update"
    actualDuration, // เวลา render จริง (ms)
    baseDuration, // เวลา render โดยไม่มี memoization
    startTime,
    commitTime
  ) => {
    console.log({
      id, phase, actualDuration, baseDuration
    });
  };

  return (
    <Profiler id="ProductList" onRender={onRenderCallback}>
      <ProductList />
    </Profiler>
  );
}
```

### ใช้ React DevTools Profiler

```
1. เปิด Chrome DevTools
2. ไป tab "Profiler" ใน React DevTools
3. คลิก Record
4. ทำ interaction ที่ต้องการวัด
5. หยุด Record
6. ดู flame chart เพื่อหา component ที่ render ช้า
```

### Performance Measurement Hook

```jsx
function usePerformanceMeasure(name) {
  const startTimeRef = useRef(null);
  
  const start = useCallback(() => {
    startTimeRef.current = performance.now();
    performance.mark(`${name}-start`);
  }, [name]);
  
  const end = useCallback(() => {
    const endTime = performance.now();
    const duration = endTime - (startTimeRef.current || endTime);
    performance.mark(`${name}-end`);
    performance.measure(name, `${name}-start`, `${name}-end`);
    console.log(`${name}: ${duration.toFixed(2)}ms`);
    return duration;
  }, [name]);
  
  return { start, end };
}
```

---

## Step 414: เมื่อไหรควร Optimize

### Rule of Thumb

```
"Don't optimize prematurely"

ก่อน optimize ต้อง:
1. มีปัญหา performance จริงๆ
2. Profile และหา bottleneck
3. Measure before & after optimization
```

### เมื่อไหร่ useMemo/useCallback ช่วยได้จริง

```jsx
// ✅ ควรใช้ useMemo เมื่อ:
// 1. Computation แพงจริงๆ (> 1ms)
const heavyResult = useMemo(() => heavyComputation(data), [data]);

// 2. Object/Array ที่ส่งให้ memoized component
const memoizedConfig = useMemo(() => ({
  endpoint: '/api/data',
  headers: { Authorization: `Bearer ${token}` }
}), [token]);

// 3. Dependencies ของ useEffect
const queryParams = useMemo(() => 
  buildQueryParams(filters),
  [filters]
);
useEffect(() => { fetch('/api', queryParams); }, [queryParams]);

// ✅ ควรใช้ useCallback เมื่อ:
// 1. ส่ง function ให้ React.memo component
const handleClick = useCallback(() => {}, []);

// 2. Function เป็น dependency ของ useEffect/useMemo
const fetchData = useCallback(async () => {}, [userId]);
useEffect(() => { fetchData(); }, [fetchData]);

// ❌ ไม่ควรใช้ useMemo/useCallback เมื่อ:
// 1. Computation ง่ายและเร็ว
const doubled = useMemo(() => value * 2, [value]); // ❌ ไม่จำเป็น
const name = useMemo(() => `${first} ${last}`, [first, last]); // ❌ overhead เกิน

// 2. Component ที่ไม่ได้ wrap ด้วย React.memo
function SimpleComp({ onClick }) { // ไม่ใช่ React.memo
  return <button onClick={onClick}>Click</button>;
}
// const handleClick = useCallback(() => {}, []); // ❌ ไม่มีประโยชน์
```

---

## Step 415: ตัวอย่าง Before/After Optimization

### Before (ไม่ Optimize)

```jsx
function UnoptimizedDashboard({ userId }) {
  const [userData, setUserData] = useState(null);
  const [filter, setFilter] = useState('all');
  const [sortOrder, setSortOrder] = useState('asc');

  useEffect(() => {
    fetch(`/api/users/${userId}/data`).then(r => r.json()).then(setUserData);
  }, [userId]);

  // ❌ คำนวณใหม่ทุก render แม้ filter/sortOrder ไม่เปลี่ยน
  const processedData = userData?.items
    .filter(item => filter === 'all' || item.status === filter)
    .sort((a, b) => sortOrder === 'asc' ? a.value - b.value : b.value - a.value) || [];

  // ❌ Function ใหม่ทุก render
  const handleDelete = (id) => {
    setUserData(prev => ({
      ...prev,
      items: prev.items.filter(item => item.id !== id)
    }));
  };

  // ❌ Function ใหม่ทุก render
  const handleUpdate = (id, updates) => {
    setUserData(prev => ({
      ...prev,
      items: prev.items.map(item => item.id === id ? { ...item, ...updates } : item)
    }));
  };

  return (
    <div>
      <FilterBar filter={filter} onFilterChange={setFilter} />
      <SortBar sortOrder={sortOrder} onSortChange={setSortOrder} />
      <DataTable
        data={processedData}
        onDelete={handleDelete}
        onUpdate={handleUpdate}
      />
    </div>
  );
}
```

### After (Optimized)

```jsx
function OptimizedDashboard({ userId }) {
  const [userData, setUserData] = useState(null);
  const [filter, setFilter] = useState('all');
  const [sortOrder, setSortOrder] = useState('asc');

  useEffect(() => {
    fetch(`/api/users/${userId}/data`).then(r => r.json()).then(setUserData);
  }, [userId]);

  // ✅ Cache processed data
  const processedData = useMemo(() => {
    if (!userData?.items) return [];
    return userData.items
      .filter(item => filter === 'all' || item.status === filter)
      .sort((a, b) => sortOrder === 'asc' ? a.value - b.value : b.value - a.value);
  }, [userData, filter, sortOrder]); // คำนวณใหม่เฉพาะเมื่อ dependencies เปลี่ยน

  // ✅ Cache handlers
  const handleDelete = useCallback((id) => {
    setUserData(prev => ({
      ...prev,
      items: prev.items.filter(item => item.id !== id)
    }));
  }, []); // ไม่มี deps → function ไม่เปลี่ยน

  const handleUpdate = useCallback((id, updates) => {
    setUserData(prev => ({
      ...prev,
      items: prev.items.map(item => item.id === id ? { ...item, ...updates } : item)
    }));
  }, []); // ไม่มี deps → function ไม่เปลี่ยน

  // ✅ Cache stats
  const stats = useMemo(() => ({
    total: processedData.length,
    avgValue: processedData.reduce((s, i) => s + i.value, 0) / (processedData.length || 1)
  }), [processedData]);

  return (
    <div>
      <StatsBar stats={stats} />
      <FilterBar filter={filter} onFilterChange={setFilter} />
      <SortBar sortOrder={sortOrder} onSortChange={setSortOrder} />
      {/* DataTable ต้องเป็น React.memo เพื่อให้ useCallback มีผล */}
      <DataTable
        data={processedData}
        onDelete={handleDelete}
        onUpdate={handleUpdate}
      />
    </div>
  );
}

// ✅ DataTable ต้องใช้ React.memo
const DataTable = React.memo(function({ data, onDelete, onUpdate }) {
  console.log('DataTable renders');
  return (
    <table>
      <tbody>
        {data.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.value}</td>
            <td>
              <button onClick={() => onDelete(item.id)}>Delete</button>
              <button onClick={() => onUpdate(item.id, { value: item.value + 1 })}>+1</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
});
```

---

## Step 416: Anti-Patterns

### Anti-pattern 1: Over-optimization

```jsx
// ❌ useMemo ที่ไม่จำเป็น (overhead เกิน benefit)
function SimpleGreeting({ name }) {
  const greeting = useMemo(() => `Hello, ${name}!`, [name]); // ❌ ง่ายเกิน
  return <p>{greeting}</p>;
}

// ✅ ดีกว่า
function SimpleGreeting({ name }) {
  return <p>Hello, {name}!</p>; // ✅ คำนวณง่าย ไม่ต้อง memo
}
```

### Anti-pattern 2: Missing Dependencies

```jsx
// ❌ Missing dependency → stale closure
function Counter({ step }) {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(c => c + step); // ใช้ step
  }, []); // ❌ ขาด step → จะใช้ step ค่าเก่า

  // ✅ ใส่ dependencies ให้ครบ
  const incrementCorrect = useCallback(() => {
    setCount(c => c + step);
  }, [step]); // ✅
}
```

### Anti-pattern 3: New Object/Array ใน deps

```jsx
// ❌ Object ใหม่ทุก render → useMemo รัน re-compute ทุกครั้ง
function BadExample({ id }) {
  const options = { id, type: 'user' }; // Object ใหม่ทุก render
  
  const result = useMemo(() => {
    return expensiveComputation(options);
  }, [options]); // ❌ options เปลี่ยนทุก render!

  // ✅ ใส่ primitive values ใน deps แทน
  const resultFixed = useMemo(() => {
    return expensiveComputation({ id, type: 'user' });
  }, [id]); // ✅ id เป็น primitive
}
```

---

## Step 417: useTransition

`useTransition` ทำให้ UI state updates เป็น non-blocking

```jsx
import { useState, useTransition } from 'react';

function SearchWithTransition() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const value = e.target.value;
    setQuery(value); // Urgent update: input value
    
    // Non-urgent update: search results (can be interrupted)
    startTransition(() => {
      const filteredResults = hugeDataset.filter(item =>
        item.name.toLowerCase().includes(value.toLowerCase())
      );
      setResults(filteredResults);
    });
  };

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={handleSearch}
        placeholder="ค้นหา..."
      />
      {isPending ? (
        <p style={{ color: '#999' }}>กำลังค้นหา...</p>
      ) : (
        <ul>
          {results.map(item => <li key={item.id}>{item.name}</li>)}
        </ul>
      )}
    </div>
  );
}
```

---

## Step 418: useDeferredValue

`useDeferredValue` เลื่อนการ update ค่าให้เป็น lower priority

```jsx
import { useState, useDeferredValue } from 'react';

function DeferredSearch() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  // query อัปเดตทันที (input responsive)
  // deferredQuery อัปเดตช้ากว่า (search results can lag)
  
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <div style={{ opacity: isStale ? 0.5 : 1, transition: 'opacity 0.2s' }}>
        {/* ใช้ deferredQuery สำหรับ expensive operation */}
        <SearchResults query={deferredQuery} />
      </div>
    </div>
  );
}

const SearchResults = React.memo(function({ query }) {
  // Expensive computation ที่อาจ block UI
  const results = useMemo(() => {
    return hugeDataset.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [query]);

  return (
    <ul>
      {results.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});
```

---

## สรุป useMemo และ useCallback

```jsx
// ใช้ useMemo เมื่อ:
// 1. Computation แพง (filtering, sorting, calculations)
const data = useMemo(() => expensiveOp(input), [input]);

// 2. Object/Array ที่เป็น deps ของ useEffect/useMemo อื่น
const config = useMemo(() => ({ url, headers }), [url, token]);
useEffect(() => { fetch(config.url); }, [config]);

// 3. Context value
const ctxValue = useMemo(() => ({ user, logout }), [user]);

// ใช้ useCallback เมื่อ:
// 1. Function ส่งให้ React.memo component
const handler = useCallback(() => {}, [deps]);
<MemoizedChild onClick={handler} />

// 2. Function ที่เป็น deps ของ useEffect
const fetchFn = useCallback(async () => {}, [userId]);
useEffect(() => { fetchFn(); }, [fetchFn]);
```

---

## Quiz และแบบฝึกหัด

### Quiz

1. useMemo กับ useCallback ต่างกันอย่างไร?
2. React.memo ทำงานร่วมกับ useCallback อย่างไร?
3. เมื่อไหร่ควร optimize และเมื่อไหร่ไม่ควร?
4. useTransition กับ useDeferredValue ต่างกันอย่างไร?

### แบบฝึกหัด

**Exercise 1**: Optimize รายการสินค้าขนาดใหญ่

```jsx
// TODO: Optimize component นี้
function ProductCatalog({ products, currentUser }) {
  const [category, setCategory] = useState('all');
  const [priceRange, setPriceRange] = useState([0, 10000]);
  const [searchTerm, setSearchTerm] = useState('');
  
  // TODO: ใช้ useMemo สำหรับ filtering
  const filteredProducts = products.filter(p => 
    (category === 'all' || p.category === category) &&
    p.price >= priceRange[0] && p.price <= priceRange[1] &&
    p.name.includes(searchTerm)
  );
  
  // TODO: ใช้ useCallback สำหรับ handlers
  const handleAddToCart = (productId) => {
    cartService.add(productId, currentUser.id);
  };
  
  return (
    <div>
      {/* controls */}
      <ProductGrid
        products={filteredProducts}
        onAddToCart={handleAddToCart}
      />
    </div>
  );
}
```

**Exercise 2**: วัด performance ด้วย Profiler

```jsx
// TODO: ครอบ ProductCatalog ด้วย Profiler
// และ log ข้อมูล render duration
function App() {
  return (
    // TODO: ครอบด้วย <Profiler>
    <ProductCatalog products={largeProductList} />
  );
}
```

---

➡️ ไปต่อ: [Part 19: useRef Hook](./part-19-useRef-hook.md)
