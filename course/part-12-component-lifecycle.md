# Part 12: Component Lifecycle

## Step 241-265 | ระดับ: กลาง

---

## สารบัญ

- [Step 241: React Component Lifecycle คืออะไร?](#step-241-react-component-lifecycle-คืออะไร)
- [Step 242: Lifecycle ใน Class Components](#step-242-lifecycle-ใน-class-components)
- [Step 243: Lifecycle ใน Function Components](#step-243-lifecycle-ใน-function-components)
- [Step 244: Phase 1 - Mounting](#step-244-phase-1---mounting)
- [Step 245: Phase 2 - Updating](#step-245-phase-2---updating)
- [Step 246: Phase 3 - Unmounting](#step-246-phase-3---unmounting)
- [Step 247: componentDidMount → useEffect](#step-247-componentdidmount--useeffect)
- [Step 248: componentDidUpdate → useEffect with deps](#step-248-componentdidupdate--useeffect-with-deps)
- [Step 249: componentWillUnmount → cleanup function](#step-249-componentwillunmount--cleanup-function)
- [Step 250: getDerivedStateFromProps equivalent](#step-250-getderivedstatefromprops-equivalent)
- [Step 251: shouldComponentUpdate → React.memo](#step-251-shouldcomponentupdate--reactmemo)
- [Step 252: Error Boundaries](#step-252-error-boundaries)
- [Step 253: React StrictMode และ Double Invoke](#step-253-react-strictmode-และ-double-invoke)
- [Step 254: Lifecycle Diagrams](#step-254-lifecycle-diagrams)
- [Step 255: Practical Examples](#step-255-practical-examples)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 241: React Component Lifecycle คืออะไร?

**Component Lifecycle** คือชีวิตของ component ตั้งแต่ถูกสร้าง แสดงผล อัปเดต จนถึงถูกลบออกจาก DOM

### 3 Phase หลัก

```
1. MOUNTING    → Component ถูกสร้างและเพิ่มเข้า DOM
2. UPDATING    → Component re-renders เนื่องจาก state หรือ props เปลี่ยน
3. UNMOUNTING  → Component ถูกลบออกจาก DOM
```

### ทำไมต้องรู้?

- รู้ว่าควรทำ side effects เมื่อใด
- หลีกเลี่ยง memory leaks
- Optimize performance
- Debug ปัญหาที่เกี่ยวกับ rendering

---

## Step 242: Lifecycle ใน Class Components

ก่อน Hooks จะมา นักพัฒนาใช้ Class Components กับ Lifecycle Methods

```jsx
import { Component } from 'react';

class LifecycleDemo extends Component {
  constructor(props) {
    super(props);
    // สร้าง state เริ่มต้น
    this.state = {
      count: 0,
      data: null
    };
    console.log('1. constructor - component initialized');
  }

  // MOUNTING PHASE
  componentDidMount() {
    // รันหลัง component render ครั้งแรก
    console.log('3. componentDidMount - component added to DOM');
    fetch('/api/data')
      .then(res => res.json())
      .then(data => this.setState({ data }));
  }

  // UPDATING PHASE
  componentDidUpdate(prevProps, prevState) {
    // รันทุกครั้งที่ state หรือ props เปลี่ยน
    console.log('componentDidUpdate');
    if (prevState.count !== this.state.count) {
      document.title = `Count: ${this.state.count}`;
    }
  }

  // UPDATING PHASE (optimization)
  shouldComponentUpdate(nextProps, nextState) {
    // ถ้า return false, component จะไม่ re-render
    return nextState.count !== this.state.count;
  }

  // UNMOUNTING PHASE
  componentWillUnmount() {
    // ทำความสะอาดก่อน component ถูกลบ
    console.log('componentWillUnmount - cleanup');
    clearInterval(this.intervalId);
  }

  render() {
    console.log('2. render - (re-render)');
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState(s => ({ count: s.count + 1 }))}>
          Increment
        </button>
      </div>
    );
  }
}
```

---

## Step 243: Lifecycle ใน Function Components

ด้วย Hooks เราจัดการ lifecycle ผ่าน `useEffect`

```jsx
import { useState, useEffect } from 'react';

function LifecycleHooksDemo() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);

  // componentDidMount equivalent
  useEffect(() => {
    console.log('Component mounted (componentDidMount)');
    
    fetch('/api/data')
      .then(res => res.json())
      .then(data => setData(data));
    
    // componentWillUnmount equivalent
    return () => {
      console.log('Component unmounting (componentWillUnmount)');
    };
  }, []);

  // componentDidUpdate equivalent
  useEffect(() => {
    console.log('count changed (componentDidUpdate)');
    document.title = `Count: ${count}`;
  }, [count]);

  console.log('Rendering (render)');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

### เปรียบเทียบ Class vs Hooks

| Class Method | Hooks Equivalent |
|---|---|
| `constructor` | `useState` initializer |
| `componentDidMount` | `useEffect(() => {...}, [])` |
| `componentDidUpdate` | `useEffect(() => {...}, [deps])` |
| `componentWillUnmount` | `useEffect(() => { return () => {...} }, [])` |
| `shouldComponentUpdate` | `React.memo` + `useMemo` |
| `getDerivedStateFromProps` | render-time calculation |
| `getSnapshotBeforeUpdate` | `useRef` before update |

---

## Step 244: Phase 1 - Mounting

Mounting คือกระบวนการที่ component ถูกสร้างและเพิ่มเข้า DOM ครั้งแรก

```jsx
function MountingDemo() {
  const [isShown, setIsShown] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsShown(s => !s)}>
        {isShown ? 'Hide' : 'Show'} Component
      </button>
      {isShown && <MountableComponent />}
    </div>
  );
}

function MountableComponent() {
  const [value, setValue] = useState('initial'); // ← useState initializer (เหมือน constructor)

  console.log('Render phase');

  useEffect(() => {
    // ← componentDidMount equivalent
    console.log('Mounted! Component is now in DOM');
    
    // ทำงานที่ต้องการ DOM เช่น:
    // - เรียก API
    // - ตั้งค่า subscriptions
    // - measure elements
    
    return () => {
      // ← componentWillUnmount
      console.log('Unmounting...');
    };
  }, []);

  return <div>Mounted Component: {value}</div>;
}
```

### Mounting Sequence

```
1. Function Component เรียก useState, useReducer (initialize state)
2. Component renders (JSX → Virtual DOM)
3. React commits to real DOM
4. useEffect with [] runs (componentDidMount equivalent)
```

---

## Step 245: Phase 2 - Updating

Updating เกิดขึ้นเมื่อ state หรือ props เปลี่ยนแปลง

```jsx
function UpdatingDemo() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // รันเมื่อ count เปลี่ยน
  useEffect(() => {
    console.log(`Count updated to: ${count}`);
    
    // เหมือน componentDidUpdate ที่ตรวจว่า count เปลี่ยน:
    // if (prevState.count !== this.state.count) { ... }
  }, [count]);

  // รันเมื่อ text เปลี่ยน
  useEffect(() => {
    console.log(`Text updated to: ${text}`);
  }, [text]);

  // รันทุกครั้งที่ update (เหมือน componentDidUpdate ไม่มี condition)
  useEffect(() => {
    console.log('Component updated (any state or prop change)');
  });

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="Type something..."
      />
    </div>
  );
}
```

### Updating Sequence

```
1. State หรือ Props เปลี่ยน
2. Component re-renders
3. React เปรียบเทียบ Virtual DOM (Reconciliation)
4. React อัปเดต real DOM ในส่วนที่เปลี่ยน
5. Cleanup จาก useEffect ก่อนหน้า (ถ้ามี)
6. useEffect รันใหม่ (ถ้า dependencies เปลี่ยน)
```

---

## Step 246: Phase 3 - Unmounting

Unmounting เกิดขึ้นเมื่อ component ถูกลบออกจาก DOM

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    console.log('Timer started');
    
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup รันเมื่อ component unmount
    return () => {
      console.log('Timer stopped (cleanup)');
      clearInterval(interval); // ✅ หยุด timer ก่อน unmount
    };
  }, []);

  return <div>Timer: {seconds}s</div>;
}

function UnmountingDemo() {
  const [showTimer, setShowTimer] = useState(true);

  return (
    <div>
      <button onClick={() => setShowTimer(s => !s)}>
        {showTimer ? 'Remove' : 'Add'} Timer
      </button>
      {showTimer && <Timer />}
    </div>
  );
}
```

### สิ่งที่ต้อง Cleanup

```jsx
function CompleteCleanupExample() {
  useEffect(() => {
    // 1. Event listeners
    const handler = () => {};
    window.addEventListener('resize', handler);
    
    // 2. Timers
    const timerId = setInterval(() => {}, 1000);
    const timeoutId = setTimeout(() => {}, 5000);
    
    // 3. Subscriptions
    const subscription = someObservable.subscribe(handler);
    
    // 4. WebSocket
    const ws = new WebSocket('wss://...');
    
    // 5. IntersectionObserver
    const observer = new IntersectionObserver(handler);
    observer.observe(someElement);
    
    // Cleanup ทั้งหมด
    return () => {
      window.removeEventListener('resize', handler);
      clearInterval(timerId);
      clearTimeout(timeoutId);
      subscription.unsubscribe();
      ws.close();
      observer.disconnect();
    };
  }, []);
}
```

---

## Step 247: componentDidMount → useEffect

```jsx
// Class Component (เดิม)
class OldComponent extends Component {
  componentDidMount() {
    // รันหลัง mount
    this.loadData();
    this.setupEventListeners();
    document.title = 'Component Mounted';
  }

  componentWillUnmount() {
    this.cleanup();
    document.title = 'My App';
  }
}

// Function Component (ใหม่)
function NewComponent() {
  // เทียบเท่า componentDidMount + componentWillUnmount
  useEffect(() => {
    // ← componentDidMount
    loadData();
    setupEventListeners();
    document.title = 'Component Mounted';

    return () => {
      // ← componentWillUnmount
      cleanup();
      document.title = 'My App';
    };
  }, []); // [] = mount/unmount เท่านั้น
}
```

### ตัวอย่างจริง: Data Loading on Mount

```jsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  // componentDidMount equivalent
  useEffect(() => {
    let isMounted = true;
    
    fetch('https://fakestoreapi.com/products')
      .then(res => res.json())
      .then(data => {
        if (isMounted) {
          setProducts(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (isMounted) {
          console.error(err);
          setLoading(false);
        }
      });

    return () => {
      isMounted = false; // componentWillUnmount
    };
  }, []); // Mount once

  if (loading) return <div>Loading products...</div>;

  return (
    <div>
      <h2>Products ({products.length})</h2>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '20px' }}>
        {products.map(product => (
          <div key={product.id} style={{ border: '1px solid #ccc', padding: '15px', borderRadius: '8px' }}>
            <img src={product.image} alt={product.title} style={{ width: '100%', height: '200px', objectFit: 'contain' }} />
            <h3 style={{ fontSize: '14px' }}>{product.title}</h3>
            <p style={{ color: 'green', fontWeight: 'bold' }}>${product.price}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 248: componentDidUpdate → useEffect with deps

```jsx
// Class Component (เดิม)
class OldSearchComponent extends Component {
  componentDidUpdate(prevProps, prevState) {
    // ตรวจว่า prop เปลี่ยนหรือไม่
    if (prevProps.searchTerm !== this.props.searchTerm) {
      this.fetchResults(this.props.searchTerm);
    }
    
    if (prevState.page !== this.state.page) {
      this.fetchResults(this.props.searchTerm, this.state.page);
    }
  }
}

// Function Component (ใหม่)
function NewSearchComponent({ searchTerm }) {
  const [page, setPage] = useState(1);
  const [results, setResults] = useState([]);

  // componentDidUpdate equivalent (watch searchTerm)
  useEffect(() => {
    if (!searchTerm) return;
    
    fetchResults(searchTerm, page).then(setResults);
  }, [searchTerm, page]); // รันเมื่อ searchTerm หรือ page เปลี่ยน
  
  return (
    <div>
      <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>
      <button onClick={() => setPage(p => p + 1)}>Next Page</button>
    </div>
  );
}
```

### ตัวอย่างจริง: Profile Sync

```jsx
function UserProfile({ userId }) {
  const [profile, setProfile] = useState(null);
  const [activity, setActivity] = useState([]);

  // Load profile เมื่อ userId เปลี่ยน
  useEffect(() => {
    setProfile(null); // Clear old profile
    
    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then(res => res.json())
      .then(data => setProfile(data));
  }, [userId]); // ← คล้าย componentDidUpdate ที่ตรวจ userId

  // Load activity เมื่อ userId เปลี่ยน
  useEffect(() => {
    setActivity([]);
    
    fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`)
      .then(res => res.json())
      .then(data => setActivity(data));
  }, [userId]);

  // Update document title เมื่อ profile เปลี่ยน
  useEffect(() => {
    if (profile) {
      document.title = `${profile.name}'s Profile`;
    }
    return () => { document.title = 'App'; };
  }, [profile]);

  if (!profile) return <div>Loading...</div>;

  return (
    <div>
      <h2>{profile.name}</h2>
      <p>{profile.email}</p>
      <h3>Posts ({activity.length})</h3>
      <ul>
        {activity.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Step 249: componentWillUnmount → cleanup function

```jsx
// Class Component (เดิม)
class OldAnalytics extends Component {
  componentDidMount() {
    this.analyticsSession = analytics.startSession({
      page: this.props.pageName
    });
    
    window.addEventListener('beforeunload', this.handleBeforeUnload);
  }

  componentWillUnmount() {
    // ต้อง cleanup ทุกอย่าง
    this.analyticsSession.end();
    window.removeEventListener('beforeunload', this.handleBeforeUnload);
  }
}

// Function Component (ใหม่)
function NewAnalytics({ pageName }) {
  useEffect(() => {
    // componentDidMount equivalent
    const session = analytics.startSession({ page: pageName });
    
    const handleBeforeUnload = () => {
      session.end();
    };
    window.addEventListener('beforeunload', handleBeforeUnload);

    // componentWillUnmount equivalent
    return () => {
      session.end();
      window.removeEventListener('beforeunload', handleBeforeUnload);
    };
  }, [pageName]);
}
```

### ตัวอย่าง: Focus Trap

```jsx
function Modal({ isOpen, onClose, children }) {
  useEffect(() => {
    if (!isOpen) return;

    // Save focus before modal opens
    const previousFocus = document.activeElement;
    
    // Focus modal
    const modal = document.getElementById('modal');
    modal?.focus();
    
    // Trap focus within modal
    function handleTabKey(e) {
      if (e.key !== 'Tab') return;
      
      const focusable = modal?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (!focusable?.length) return;
      
      const firstFocusable = focusable[0];
      const lastFocusable = focusable[focusable.length - 1];
      
      if (e.shiftKey) {
        if (document.activeElement === firstFocusable) {
          e.preventDefault();
          lastFocusable.focus();
        }
      } else {
        if (document.activeElement === lastFocusable) {
          e.preventDefault();
          firstFocusable.focus();
        }
      }
    }
    
    // Close on Escape
    function handleEscape(e) {
      if (e.key === 'Escape') onClose();
    }
    
    document.addEventListener('keydown', handleTabKey);
    document.addEventListener('keydown', handleEscape);
    
    // Cleanup (componentWillUnmount)
    return () => {
      document.removeEventListener('keydown', handleTabKey);
      document.removeEventListener('keydown', handleEscape);
      previousFocus?.focus(); // Restore focus
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div style={{
      position: 'fixed', inset: 0,
      background: 'rgba(0,0,0,0.5)',
      display: 'flex', alignItems: 'center', justifyContent: 'center',
      zIndex: 1000
    }}>
      <div
        id="modal"
        tabIndex={-1}
        style={{
          background: 'white', borderRadius: '8px',
          padding: '24px', maxWidth: '500px', width: '90%'
        }}
      >
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

---

## Step 250: getDerivedStateFromProps equivalent

```jsx
// Class Component
class OldComponent extends Component {
  static getDerivedStateFromProps(props, state) {
    if (props.value !== state.prevValue) {
      return {
        derivedValue: props.value * 2,
        prevValue: props.value
      };
    }
    return null;
  }
}

// Function Component - คำนวณ derived state ตอน render
function NewComponent({ value }) {
  // ✅ คำนวณ derived state โดยตรง (ไม่ต้องใช้ useEffect)
  const derivedValue = value * 2;
  
  return <div>{derivedValue}</div>;
}
```

### ตัวอย่าง: Price Formatter

```jsx
function ProductCard({ basePrice, discount, currency }) {
  // Derived state - คำนวณตอน render
  const discountedPrice = basePrice * (1 - discount / 100);
  const formattedPrice = new Intl.NumberFormat('th-TH', {
    style: 'currency',
    currency: currency || 'THB'
  }).format(discountedPrice);

  return (
    <div>
      <p>Original: {basePrice} บาท</p>
      <p>Discount: {discount}%</p>
      <p>Final: {formattedPrice}</p>
    </div>
  );
}
```

---

## Step 251: shouldComponentUpdate → React.memo

```jsx
// Class Component
class OldExpensiveComponent extends Component {
  shouldComponentUpdate(nextProps) {
    // Only re-render if id or name changes
    return (
      nextProps.id !== this.props.id || 
      nextProps.name !== this.props.name
    );
  }
  
  render() {
    return <div>{this.props.name}</div>;
  }
}

// Function Component - ใช้ React.memo
const NewExpensiveComponent = React.memo(function({ id, name }) {
  console.log('Rendering expensive component');
  return <div>{name}</div>;
});

// หรือ custom comparison
const OptimizedComponent = React.memo(
  function({ id, name, data }) {
    return <div>{name}: {data.length} items</div>;
  },
  (prevProps, nextProps) => {
    // Return true ถ้า props เท่ากัน (ไม่ต้อง re-render)
    return prevProps.id === nextProps.id && 
           prevProps.name === nextProps.name;
    // ไม่สนใจ data เปลี่ยน
  }
);
```

---

## Step 252: Error Boundaries

Error Boundaries ยังต้องใช้ Class Component (ไม่มี Hook equivalent สมบูรณ์แบบ)

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // รันเมื่อ child component throw error
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  // รันหลัง error เพื่อ log
  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error);
    console.error('Component stack:', errorInfo.componentStack);
    
    // ส่ง error ไป logging service
    // logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ padding: '20px', background: '#fee', border: '1px solid #f00' }}>
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// การใช้งาน
function App() {
  return (
    <ErrorBoundary>
      <ErrorBoundary>
        <ProductList />
      </ErrorBoundary>
      <ErrorBoundary>
        <Sidebar />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}

// Component ที่อาจ throw error
function BuggyComponent() {
  const [shouldThrow, setShouldThrow] = useState(false);
  
  if (shouldThrow) {
    throw new Error('I crashed!');
  }
  
  return (
    <button onClick={() => setShouldThrow(true)}>
      Trigger Error
    </button>
  );
}
```

---

## Step 253: React StrictMode และ Double Invoke

ใน Development mode, `React.StrictMode` จะ:
1. Double invoke effects (mount → unmount → mount)
2. Double invoke render functions
3. Warn เกี่ยวกับ deprecated APIs

```jsx
// main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

### ทำไม Double Invoke?

```jsx
function StrictModeExample() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Effect runs');
    // ใน StrictMode (development): รัน 2 ครั้ง
    // (mount → cleanup → mount อีกครั้ง)
    
    return () => {
      console.log('Cleanup runs');
    };
  }, []);

  return <button onClick={() => setCount(c => c + 1)}>
    Count: {count}
  </button>;
}
```

### Console Output ใน StrictMode (Dev)

```
Effect runs
Cleanup runs
Effect runs
```

### จะเกิดอะไรถ้า Effect ไม่ Cleanup อย่างถูกต้อง?

```jsx
// ❌ ปัญหา: ถ้าไม่ cleanup
function BadExample() {
  useEffect(() => {
    // ใน StrictMode: รัน 2 ครั้ง → event listener ถูก add 2 ครั้ง!
    window.addEventListener('click', handler);
    // ขาด cleanup!
  }, []);
}

// ✅ ถูกต้อง: cleanup ทำให้ StrictMode ทำงานถูก
function GoodExample() {
  useEffect(() => {
    window.addEventListener('click', handler);
    return () => window.removeEventListener('click', handler);
  }, []);
}
```

---

## Step 254: Lifecycle Diagrams

### Complete Function Component Lifecycle

```
MOUNTING:
┌─────────────────────────────────────────┐
│ 1. useState initializer runs (once)     │
│ 2. Component function runs (render)     │
│ 3. React updates the DOM                │
│ 4. useEffect(() => {...}, []) runs     │
└─────────────────────────────────────────┘

UPDATING (state/props change):
┌─────────────────────────────────────────┐
│ 1. Component function runs (re-render)  │
│ 2. React diffs & updates DOM            │
│ 3. Cleanup from previous effects        │
│ 4. useEffect with changed deps runs     │
└─────────────────────────────────────────┘

UNMOUNTING:
┌─────────────────────────────────────────┐
│ 1. Cleanup from all effects runs        │
│ 2. Component removed from DOM           │
└─────────────────────────────────────────┘
```

### useEffect Timing Diagram

```jsx
function TimingDemo() {
  const [value, setValue] = useState(0);

  // Effect 1: ทุก render
  useEffect(() => {
    console.log(`After every render: ${value}`);
  });

  // Effect 2: Mount only
  useEffect(() => {
    console.log('After mount only');
    return () => console.log('Before unmount');
  }, []);

  // Effect 3: When value changes
  useEffect(() => {
    console.log(`After value change: ${value}`);
    return () => console.log(`Before next value change, current: ${value}`);
  }, [value]);

  return <button onClick={() => setValue(v => v + 1)}>Value: {value}</button>;
}

/*
Initial mount:
  After mount only
  After value change: 0
  After every render: 0

After button click (value: 0 → 1):
  Before next value change, current: 0  (cleanup from Effect 3)
  After value change: 1                 (Effect 3 runs again)
  After every render: 1                  (Effect 1 runs)

On unmount:
  Before unmount                         (cleanup from Effect 2)
  Before next value change, current: 1  (cleanup from Effect 3)
*/
```

---

## Step 255: Practical Examples

### ตัวอย่าง 1: Data Sync Component

```jsx
function DataSyncDashboard({ userId, refreshInterval = 30000 }) {
  const [userData, setUserData] = useState(null);
  const [lastSync, setLastSync] = useState(null);
  const [syncing, setSyncing] = useState(false);

  const syncData = useCallback(async () => {
    setSyncing(true);
    try {
      const res = await fetch(`/api/users/${userId}/dashboard`);
      const data = await res.json();
      setUserData(data);
      setLastSync(new Date());
    } finally {
      setSyncing(false);
    }
  }, [userId]);

  // Initial load + refresh when userId changes
  useEffect(() => {
    syncData();
  }, [syncData]);

  // Auto-refresh every N seconds
  useEffect(() => {
    const interval = setInterval(syncData, refreshInterval);
    return () => clearInterval(interval);
  }, [syncData, refreshInterval]);

  // Update document title
  useEffect(() => {
    if (userData) {
      document.title = `${userData.name} - Dashboard`;
    }
    return () => { document.title = 'App'; };
  }, [userData]);

  return (
    <div>
      {syncing && <div>Syncing...</div>}
      {lastSync && <small>Last sync: {lastSync.toLocaleTimeString()}</small>}
      {userData && (
        <div>
          <h2>{userData.name}</h2>
          <p>Score: {userData.score}</p>
        </div>
      )}
    </div>
  );
}
```

### ตัวอย่าง 2: Animated List Item

```jsx
function AnimatedListItem({ children, isVisible }) {
  const [shouldRender, setShouldRender] = useState(isVisible);
  const [opacity, setOpacity] = useState(isVisible ? 1 : 0);

  // จัดการ mount/unmount animation
  useEffect(() => {
    if (isVisible) {
      setShouldRender(true);
      // ต้อง delay เล็กน้อยก่อน animate
      const timer = setTimeout(() => setOpacity(1), 10);
      return () => clearTimeout(timer);
    } else {
      setOpacity(0);
      // รอ animation เสร็จก่อน unmount
      const timer = setTimeout(() => setShouldRender(false), 300);
      return () => clearTimeout(timer);
    }
  }, [isVisible]);

  if (!shouldRender) return null;

  return (
    <div style={{
      opacity,
      transform: opacity === 1 ? 'translateY(0)' : 'translateY(-10px)',
      transition: 'all 0.3s ease'
    }}>
      {children}
    </div>
  );
}
```

### ตัวอย่าง 3: Intersection Observer

```jsx
function LazyImage({ src, alt }) {
  const imgRef = useRef(null);
  const [isIntersecting, setIsIntersecting] = useState(false);
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    const element = imgRef.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsIntersecting(true);
          observer.unobserve(element); // หยุด observe หลังจาก intersect ครั้งแรก
        }
      },
      { rootMargin: '100px' } // Load 100px ก่อนถึง viewport
    );

    observer.observe(element);

    return () => observer.disconnect(); // Cleanup
  }, []);

  return (
    <div
      ref={imgRef}
      style={{
        width: '100%',
        paddingBottom: '56.25%', // 16:9 aspect ratio
        position: 'relative',
        background: '#f0f0f0'
      }}
    >
      {isIntersecting && (
        <img
          src={src}
          alt={alt}
          onLoad={() => setLoaded(true)}
          style={{
            position: 'absolute',
            top: 0, left: 0,
            width: '100%', height: '100%',
            objectFit: 'cover',
            opacity: loaded ? 1 : 0,
            transition: 'opacity 0.3s'
          }}
        />
      )}
    </div>
  );
}
```

---

## สรุป Lifecycle

```jsx
// Lifecycle Summary Component
function LifecycleSummary({ id }) {
  const [data, setData] = useState(null);

  // MOUNT: โหลดข้อมูลครั้งแรก
  useEffect(() => {
    fetchData(id).then(setData);
    return () => cleanupData(); // UNMOUNT
  }, []); // eslint-disable-line

  // UPDATE: รีโหลดเมื่อ id เปลี่ยน
  useEffect(() => {
    setData(null);
    fetchData(id).then(setData);
  }, [id]); // รันเมื่อ id เปลี่ยน

  return data ? <div>{data.name}</div> : <div>Loading...</div>;
}
```

---

## Quiz และแบบฝึกหัด

### Quiz

1. ลำดับ lifecycle phases มีอะไรบ้าง?
2. `useEffect(() => {}, [])` เทียบกับ class method ใด?
3. ทำไม StrictMode ถึง invoke effects สองครั้งใน development?
4. Error Boundaries ต้องใช้ Class หรือ Function Component?

### แบบฝึกหัด

**Exercise 1**: สร้าง Component ที่แสดง lifecycle events

```jsx
// TODO: สร้าง LifecycleLogger component ที่:
// - Log ทุก phase (mount, update, unmount)
// - แสดง log บน screen
// - มีปุ่มให้ทดสอบ update
function LifecycleLogger() {
  // implement here...
}
```

**Exercise 2**: สร้าง Auto-save feature

```jsx
// TODO: สร้าง AutoSaveForm ที่:
// - Save ทุกครั้งที่ content เปลี่ยน (debounced 1 วินาที)
// - แสดงสถานะ "Saving..." / "Saved"
// - Cleanup timer เมื่อ unmount
function AutoSaveForm() {
  // implement here...
}
```

---

➡️ ไปต่อ: [Part 13: Styling in React](./part-13-styling-in-react.md)
