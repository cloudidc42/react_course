# Part 20: Custom Hooks

## Step 461-495 | ระดับ: กลาง-สูง

---

## สารบัญ

- [Step 461: Custom Hooks คืออะไร?](#step-461-custom-hooks-คืออะไร)
- [Step 462: กฎของ Hooks](#step-462-กฎของ-hooks)
- [Step 463: สร้าง Custom Hook แรก](#step-463-สร้าง-custom-hook-แรก)
- [Step 464: useFetch Hook](#step-464-usefetch-hook)
- [Step 465: useLocalStorage Hook](#step-465-uselocalstorage-hook)
- [Step 466: useDebounce Hook](#step-466-usedebounce-hook)
- [Step 467: useThrottle Hook](#step-467-usethrottle-hook)
- [Step 468: useMediaQuery Hook](#step-468-usemediaquery-hook)
- [Step 469: useOnClickOutside Hook](#step-469-useonclickoutside-hook)
- [Step 470: useKeyPress Hook](#step-470-useKeypress-hook)
- [Step 471: useForm Hook](#step-471-useform-hook)
- [Step 472: useGeolocation Hook](#step-472-usegeolocation-hook)
- [Step 473: useCopyToClipboard Hook](#step-473-usecopytoclipboard-hook)
- [Step 474: useScrollPosition Hook](#step-474-usescrollposition-hook)
- [Step 475: useWindowSize Hook](#step-475-usewindowsize-hook)
- [Step 476: useToggle Hook](#step-476-usetoggle-hook)
- [Step 477: useCountdown Hook](#step-477-usecountdown-hook)
- [Step 478: useAsync Hook](#step-478-useasync-hook)
- [Step 479: useIntersectionObserver Hook](#step-479-useintersectionobserver-hook)
- [Step 480: Testing Custom Hooks](#step-480-testing-custom-hooks)
- [Step 481: Best Practices](#step-481-best-practices)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 461: Custom Hooks คืออะไร?

Custom Hooks คือ JavaScript function ที่:
1. ชื่อขึ้นต้นด้วย `use`
2. ใช้ React Hooks ภายใน
3. Extract และ reuse stateful logic ระหว่าง components

```jsx
// ❌ ก่อนมี Custom Hooks: logic ซ้ำกัน
function ComponentA() {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);
  // ...
}

function ComponentB() {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth); // ซ้ำ!
  useEffect(() => {
    const handler = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler); // ซ้ำ!
  }, []);
  // ...
}

// ✅ หลังมี Custom Hooks: logic อยู่ที่เดียว
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);
  return width;
}

function ComponentA() {
  const width = useWindowWidth(); // ✅ สะอาด
  // ...
}

function ComponentB() {
  const width = useWindowWidth(); // ✅ ไม่ซ้ำ
  // ...
}
```

### ข้อดีของ Custom Hooks

- **Reusability**: ใช้ logic ซ้ำในหลาย components
- **Separation of Concerns**: แยก UI กับ Logic
- **Testability**: test logic แยกจาก UI
- **Readability**: component code สะอาดขึ้น

---

## Step 462: กฎของ Hooks

Hooks มีกฎ 2 ข้อที่ต้องปฏิบัติตาม:

### กฎที่ 1: ใช้ Hooks ที่ Top Level เท่านั้น

```jsx
// ❌ ผิด: ใช้ Hook ใน conditional
function BadComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [name, setName] = useState(''); // ❌ ใน if!
  }
  
  for (let i = 0; i < 3; i++) {
    useEffect(() => {}); // ❌ ใน loop!
  }
  
  return <div />;
}

// ✅ ถูก: ใช้ Hook ที่ top level เสมอ
function GoodComponent({ isLoggedIn }) {
  const [name, setName] = useState(''); // ✅ top level
  
  useEffect(() => {
    // ทำ conditional ข้างใน effect แทน
    if (!isLoggedIn) return;
    // ...
  }, [isLoggedIn]);
  
  return <div />;
}
```

### กฎที่ 2: ใช้ Hooks ใน React Functions เท่านั้น

```jsx
// ❌ ผิด: ใช้ Hook ใน regular function
function regularFunction() {
  const [state, setState] = useState(0); // ❌ ไม่ใช่ component/hook
}

// ✅ ถูก: ใช้ใน React Component
function MyComponent() {
  const [state, setState] = useState(0); // ✅
  return <div>{state}</div>;
}

// ✅ ถูก: ใช้ใน Custom Hook
function useMyHook() {
  const [state, setState] = useState(0); // ✅
  return [state, setState];
}
```

---

## Step 463: สร้าง Custom Hook แรก

```jsx
// ตัวอย่าง: useCounter
function useCounter(initialValue = 0, { min, max, step = 1 } = {}) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => {
      const next = c + step;
      return max !== undefined ? Math.min(next, max) : next;
    });
  }, [step, max]);
  
  const decrement = useCallback(() => {
    setCount(c => {
      const next = c - step;
      return min !== undefined ? Math.max(next, min) : next;
    });
  }, [step, min]);
  
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  const setValue = useCallback((value) => {
    setCount(prev => {
      let next = value;
      if (min !== undefined) next = Math.max(next, min);
      if (max !== undefined) next = Math.min(next, max);
      return next;
    });
  }, [min, max]);
  
  return {
    count,
    increment,
    decrement,
    reset,
    setValue,
    isAtMin: min !== undefined && count <= min,
    isAtMax: max !== undefined && count >= max
  };
}

// การใช้งาน
function ProductQuantity({ maxStock }) {
  const { count: quantity, increment, decrement, isAtMin, isAtMax } = useCounter(1, {
    min: 1,
    max: maxStock
  });
  
  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <button onClick={decrement} disabled={isAtMin}>-</button>
      <span style={{ minWidth: '30px', textAlign: 'center' }}>{quantity}</span>
      <button onClick={increment} disabled={isAtMax}>+</button>
      {isAtMax && <small style={{ color: '#999' }}>สูงสุด {maxStock} ชิ้น</small>}
    </div>
  );
}
```

---

## Step 464: useFetch Hook

```jsx
// useFetch: Generic data fetching hook
function useFetch(url, options = {}) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });
  
  const optionsRef = useRef(options);

  useEffect(() => {
    if (!url) {
      setState({ data: null, loading: false, error: null });
      return;
    }

    const controller = new AbortController();
    setState(prev => ({ ...prev, loading: true, error: null }));

    fetch(url, { ...optionsRef.current, signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);
        return res.json();
      })
      .then(data => {
        setState({ data, loading: false, error: null });
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setState({ data: null, loading: false, error: err.message });
        }
      });

    return () => controller.abort();
  }, [url]);

  return state;
}

// Version ที่สามารถ refetch ได้
function useFetchWithRefetch(url) {
  const [refreshTrigger, setRefreshTrigger] = useState(0);
  const result = useFetch(url ? `${url}?_t=${refreshTrigger}` : null);
  const refetch = useCallback(() => setRefreshTrigger(t => t + 1), []);
  
  return { ...result, refetch };
}

// การใช้งาน
function UserProfile({ userId }) {
  const { data: user, loading, error, refetch } = useFetchWithRefetch(
    userId ? `https://jsonplaceholder.typicode.com/users/${userId}` : null
  );

  if (loading) return <div className="skeleton">Loading...</div>;
  if (error) return (
    <div>
      <p>Error: {error}</p>
      <button onClick={refetch}>Retry</button>
    </div>
  );
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}

// Paginated fetch
function usePaginatedFetch(baseUrl, { pageSize = 10 } = {}) {
  const [page, setPage] = useState(1);
  const url = `${baseUrl}?_page=${page}&_limit=${pageSize}`;
  const { data, loading, error } = useFetch(url);
  
  return {
    data,
    loading,
    error,
    page,
    nextPage: () => setPage(p => p + 1),
    prevPage: () => setPage(p => Math.max(1, p - 1)),
    goToPage: setPage,
    isFirstPage: page === 1
  };
}
```

---

## Step 465: useLocalStorage Hook

```jsx
function useLocalStorage(key, initialValue) {
  // State เริ่มต้นจาก localStorage (lazy initialization)
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item !== null ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Set value ลง state และ localStorage
  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.warn(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  // Remove value
  const removeValue = useCallback(() => {
    try {
      setStoredValue(initialValue);
      window.localStorage.removeItem(key);
    } catch (error) {
      console.warn(`Error removing localStorage key "${key}":`, error);
    }
  }, [key, initialValue]);

  // Sync กับ changes จาก tab อื่น
  useEffect(() => {
    const handleStorageChange = (e) => {
      if (e.key === key && e.newValue !== null) {
        try {
          setStoredValue(JSON.parse(e.newValue));
        } catch {
          setStoredValue(e.newValue);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  return [storedValue, setValue, removeValue];
}

// การใช้งาน
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);
  const [language, setLanguage, removeLanguage] = useLocalStorage('language', 'th');

  return (
    <div>
      <h2>การตั้งค่า</h2>
      
      <div>
        <label>ธีม: </label>
        <select value={theme} onChange={e => setTheme(e.target.value)}>
          <option value="light">สว่าง</option>
          <option value="dark">มืด</option>
        </select>
      </div>
      
      <div>
        <label>ขนาดตัวอักษร: {fontSize}px </label>
        <input
          type="range"
          min="12"
          max="24"
          value={fontSize}
          onChange={e => setFontSize(Number(e.target.value))}
        />
      </div>
      
      <div>
        <label>ภาษา: </label>
        <select value={language} onChange={e => setLanguage(e.target.value)}>
          <option value="th">ไทย</option>
          <option value="en">English</option>
        </select>
        <button onClick={removeLanguage}>Reset</button>
      </div>
    </div>
  );
}
```

---

## Step 466: useDebounce Hook

```jsx
// Debounce value
function useDebounceValue(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Debounce function
function useDebounceCallback(callback, delay = 500) {
  const timerRef = useRef(null);
  const callbackRef = useRef(callback);
  
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  const debouncedFn = useCallback((...args) => {
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => {
      callbackRef.current(...args);
    }, delay);
  }, [delay]);

  useEffect(() => {
    return () => clearTimeout(timerRef.current);
  }, []);

  return debouncedFn;
}

// การใช้งาน
function SearchBox() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);

  // debounce value
  const debouncedTerm = useDebounceValue(searchTerm, 400);

  useEffect(() => {
    if (!debouncedTerm) {
      setResults([]);
      return;
    }

    setIsSearching(true);
    fetch(`https://api.example.com/search?q=${encodeURIComponent(debouncedTerm)}`)
      .then(r => r.json())
      .then(data => {
        setResults(data.results || []);
        setIsSearching(false);
      })
      .catch(() => setIsSearching(false));
  }, [debouncedTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="ค้นหา..."
      />
      {isSearching && <p>กำลังค้นหา...</p>}
      <ul>
        {results.map(r => <li key={r.id}>{r.title}</li>)}
      </ul>
    </div>
  );
}
```

---

## Step 467: useThrottle Hook

```jsx
function useThrottleValue(value, limit = 200) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(Date.now());

  useEffect(() => {
    if (Date.now() >= lastUpdated.current + limit) {
      lastUpdated.current = Date.now();
      setThrottledValue(value);
    } else {
      const timer = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, limit);

      return () => clearTimeout(timer);
    }
  }, [value, limit]);

  return throttledValue;
}

// Throttle callback
function useThrottleCallback(callback, limit = 200) {
  const lastCallRef = useRef(0);
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  return useCallback((...args) => {
    const now = Date.now();
    if (now - lastCallRef.current >= limit) {
      lastCallRef.current = now;
      callbackRef.current(...args);
    }
  }, [limit]);
}

// การใช้งาน: Scroll tracking
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  
  const handleScroll = useThrottleCallback(() => {
    setScrollY(window.scrollY);
  }, 100); // อัปเดตทุก 100ms เท่านั้น

  useEffect(() => {
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);

  return (
    <div style={{ position: 'fixed', top: 10, right: 10, background: 'rgba(0,0,0,0.7)', color: 'white', padding: '8px', borderRadius: '4px' }}>
      Scroll: {scrollY}px
    </div>
  );
}
```

---

## Step 468: useMediaQuery Hook

```jsx
function useMediaQuery(query) {
  const [matches, setMatches] = useState(
    () => window.matchMedia(query).matches
  );

  useEffect(() => {
    const mediaQueryList = window.matchMedia(query);
    
    const listener = (e) => setMatches(e.matches);
    
    // Modern API
    if (mediaQueryList.addEventListener) {
      mediaQueryList.addEventListener('change', listener);
      return () => mediaQueryList.removeEventListener('change', listener);
    } else {
      // Fallback สำหรับ older browsers
      mediaQueryList.addListener(listener);
      return () => mediaQueryList.removeListener(listener);
    }
  }, [query]);

  return matches;
}

// Breakpoint hooks
function useBreakpoints() {
  const isMobile = useMediaQuery('(max-width: 639px)');
  const isTablet = useMediaQuery('(min-width: 640px) and (max-width: 1023px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  const isLargeDesktop = useMediaQuery('(min-width: 1280px)');
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');
  const prefersDarkMode = useMediaQuery('(prefers-color-scheme: dark)');
  
  return {
    isMobile,
    isTablet,
    isDesktop,
    isLargeDesktop,
    prefersReducedMotion,
    prefersDarkMode,
    breakpoint: isMobile ? 'mobile' : isTablet ? 'tablet' : isDesktop ? 'desktop' : 'xl'
  };
}

// การใช้งาน
function ResponsiveNav() {
  const { isMobile, isDesktop } = useBreakpoints();
  const [menuOpen, setMenuOpen] = useState(false);
  
  const navLinks = [
    { href: '/', label: 'หน้าหลัก' },
    { href: '/products', label: 'สินค้า' },
    { href: '/about', label: 'เกี่ยวกับเรา' },
    { href: '/contact', label: 'ติดต่อ' }
  ];
  
  return (
    <nav style={{ background: '#1a1a1a', padding: '0 20px' }}>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', height: '60px' }}>
        <span style={{ color: 'white', fontWeight: 'bold', fontSize: '20px' }}>MyShop</span>
        
        {isDesktop ? (
          <div style={{ display: 'flex', gap: '24px' }}>
            {navLinks.map(link => (
              <a key={link.href} href={link.href} style={{ color: 'white', textDecoration: 'none' }}>
                {link.label}
              </a>
            ))}
          </div>
        ) : (
          <button
            onClick={() => setMenuOpen(o => !o)}
            style={{ background: 'none', border: 'none', color: 'white', fontSize: '24px', cursor: 'pointer' }}
          >
            {menuOpen ? '✕' : '☰'}
          </button>
        )}
      </div>
      
      {isMobile && menuOpen && (
        <div style={{ borderTop: '1px solid #333', padding: '8px 0' }}>
          {navLinks.map(link => (
            <a
              key={link.href}
              href={link.href}
              style={{ display: 'block', color: 'white', textDecoration: 'none', padding: '12px 0' }}
              onClick={() => setMenuOpen(false)}
            >
              {link.label}
            </a>
          ))}
        </div>
      )}
    </nav>
  );
}
```

---

## Step 469: useOnClickOutside Hook

```jsx
function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      // ไม่ทำอะไรถ้าคลิกใน element
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// การใช้งาน: Dropdown Menu
function Dropdown({ trigger, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useOnClickOutside(dropdownRef, useCallback(() => setIsOpen(false), []));

  return (
    <div ref={dropdownRef} style={{ position: 'relative', display: 'inline-block' }}>
      <div onClick={() => setIsOpen(o => !o)}>
        {trigger}
      </div>
      {isOpen && (
        <div style={{
          position: 'absolute',
          top: '100%',
          left: 0,
          background: 'white',
          border: '1px solid #ccc',
          borderRadius: '4px',
          boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          zIndex: 100,
          minWidth: '180px',
          marginTop: '4px'
        }}>
          {children}
        </div>
      )}
    </div>
  );
}

// ตัวอย่าง Context Menu
function ContextMenu() {
  const [menuPos, setMenuPos] = useState(null);
  const menuRef = useRef(null);

  useOnClickOutside(menuRef, useCallback(() => setMenuPos(null), []));

  const handleContextMenu = (e) => {
    e.preventDefault();
    setMenuPos({ x: e.clientX, y: e.clientY });
  };

  return (
    <div
      onContextMenu={handleContextMenu}
      style={{ height: '300px', background: '#f0f0f0', userSelect: 'none', display: 'flex', alignItems: 'center', justifyContent: 'center', cursor: 'context-menu' }}
    >
      คลิกขวาเพื่อเปิด Context Menu
      {menuPos && (
        <ul
          ref={menuRef}
          style={{
            position: 'fixed',
            left: menuPos.x, top: menuPos.y,
            background: 'white', border: '1px solid #ccc',
            borderRadius: '4px', boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
            padding: '4px 0', listStyle: 'none', margin: 0, zIndex: 1000
          }}
        >
          {['คัดลอก', 'วาง', 'ลบ', 'เปลี่ยนชื่อ'].map(item => (
            <li
              key={item}
              onClick={() => setMenuPos(null)}
              style={{ padding: '8px 16px', cursor: 'pointer' }}
            >
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## Step 470: useKeyPress Hook

```jsx
function useKeyPress(targetKey) {
  const [isPressed, setIsPressed] = useState(false);

  useEffect(() => {
    const handleDown = (e) => {
      if (e.key === targetKey) setIsPressed(true);
    };
    const handleUp = (e) => {
      if (e.key === targetKey) setIsPressed(false);
    };

    window.addEventListener('keydown', handleDown);
    window.addEventListener('keyup', handleUp);

    return () => {
      window.removeEventListener('keydown', handleDown);
      window.removeEventListener('keyup', handleUp);
    };
  }, [targetKey]);

  return isPressed;
}

// Keyboard shortcuts hook
function useKeyboardShortcut(shortcuts) {
  useEffect(() => {
    const handler = (e) => {
      const key = [
        e.ctrlKey && 'ctrl',
        e.metaKey && 'meta',
        e.shiftKey && 'shift',
        e.altKey && 'alt',
        e.key.toLowerCase()
      ].filter(Boolean).join('+');
      
      const shortcut = shortcuts[key];
      if (shortcut) {
        e.preventDefault();
        shortcut(e);
      }
    };

    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [shortcuts]);
}

// การใช้งาน
function NoteEditor() {
  const [content, setContent] = useState('');
  const [isSaving, setIsSaving] = useState(false);

  const shortcuts = useMemo(() => ({
    'ctrl+s': async () => {
      setIsSaving(true);
      await fetch('/api/save', { method: 'POST', body: content });
      setIsSaving(false);
    },
    'ctrl+z': () => {
      // undo logic
    },
    'ctrl+a': (e) => {
      const textarea = document.getElementById('note-textarea');
      textarea?.select();
    }
  }), [content]);

  useKeyboardShortcut(shortcuts);

  return (
    <div>
      <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '8px' }}>
        <h3>Note Editor</h3>
        <span style={{ color: '#999', fontSize: '12px' }}>
          {isSaving ? 'Saving...' : 'Ctrl+S to save'}
        </span>
      </div>
      <textarea
        id="note-textarea"
        value={content}
        onChange={e => setContent(e.target.value)}
        style={{ width: '100%', height: '300px', padding: '12px', fontSize: '14px', fontFamily: 'monospace' }}
      />
    </div>
  );
}
```

---

## Step 471: useForm Hook

```jsx
function useForm({ 
  initialValues = {}, 
  validate, 
  onSubmit 
}) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitCount, setSubmitCount] = useState(0);

  const setFieldValue = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  }, [errors]);

  const setFieldTouched = useCallback((name, isTouched = true) => {
    setTouched(prev => ({ ...prev, [name]: isTouched }));
  }, []);

  const validateForm = useCallback(() => {
    if (!validate) return {};
    return validate(values);
  }, [validate, values]);

  const handleSubmit = useCallback(async (e) => {
    e?.preventDefault();
    
    const validationErrors = validateForm();
    setErrors(validationErrors);
    setSubmitCount(c => c + 1);
    
    // Mark all fields as touched
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    if (Object.keys(validationErrors).some(k => validationErrors[k])) {
      return;
    }
    
    setIsSubmitting(true);
    try {
      await onSubmit?.(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [validateForm, values, onSubmit]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setSubmitCount(0);
  }, [initialValues]);

  const getFieldProps = useCallback((name) => ({
    name,
    value: values[name] ?? '',
    onChange: (e) => setFieldValue(name, e.target.value),
    onBlur: () => setFieldTouched(name)
  }), [values, setFieldValue, setFieldTouched]);

  const getFieldError = useCallback((name) => {
    return touched[name] ? errors[name] : undefined;
  }, [touched, errors]);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    submitCount,
    setFieldValue,
    setFieldTouched,
    handleSubmit,
    reset,
    getFieldProps,
    getFieldError,
    isValid: !Object.values(errors).some(Boolean)
  };
}

// การใช้งาน
function ContactForm() {
  const {
    values,
    isSubmitting,
    handleSubmit,
    getFieldProps,
    getFieldError
  } = useForm({
    initialValues: {
      name: '',
      email: '',
      phone: '',
      subject: '',
      message: ''
    },
    validate: (values) => {
      const errors = {};
      if (!values.name.trim()) errors.name = 'กรุณาใส่ชื่อ';
      if (!values.email.trim()) errors.email = 'กรุณาใส่ email';
      else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) errors.email = 'รูปแบบ email ไม่ถูกต้อง';
      if (!values.message.trim()) errors.message = 'กรุณาใส่ข้อความ';
      else if (values.message.length < 20) errors.message = 'ข้อความต้องมีอย่างน้อย 20 ตัวอักษร';
      return errors;
    },
    onSubmit: async (values) => {
      await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values)
      });
      alert('ส่งข้อความสำเร็จ!');
    }
  });

  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: '500px', margin: '0 auto' }}>
      <h2>ติดต่อเรา</h2>
      
      {[
        { name: 'name', label: 'ชื่อ-นามสกุล', type: 'text', placeholder: 'ชื่อของคุณ' },
        { name: 'email', label: 'Email', type: 'email', placeholder: 'email@example.com' },
        { name: 'phone', label: 'เบอร์โทร (ไม่บังคับ)', type: 'tel', placeholder: '08X-XXX-XXXX' },
        { name: 'subject', label: 'หัวข้อ', type: 'text', placeholder: 'หัวข้อของคุณ' }
      ].map(field => (
        <div key={field.name} style={{ marginBottom: '16px' }}>
          <label style={{ display: 'block', marginBottom: '4px', fontWeight: '500' }}>
            {field.label}
          </label>
          <input
            type={field.type}
            placeholder={field.placeholder}
            {...getFieldProps(field.name)}
            style={{
              width: '100%', padding: '8px 12px',
              border: `1px solid ${getFieldError(field.name) ? 'red' : '#ccc'}`,
              borderRadius: '4px', boxSizing: 'border-box'
            }}
          />
          {getFieldError(field.name) && (
            <p style={{ color: 'red', fontSize: '12px', margin: '4px 0 0' }}>
              {getFieldError(field.name)}
            </p>
          )}
        </div>
      ))}
      
      <div style={{ marginBottom: '16px' }}>
        <label style={{ display: 'block', marginBottom: '4px', fontWeight: '500' }}>
          ข้อความ
        </label>
        <textarea
          rows={5}
          {...getFieldProps('message')}
          style={{
            width: '100%', padding: '8px 12px',
            border: `1px solid ${getFieldError('message') ? 'red' : '#ccc'}`,
            borderRadius: '4px', resize: 'vertical', boxSizing: 'border-box'
          }}
        />
        {getFieldError('message') && (
          <p style={{ color: 'red', fontSize: '12px', margin: '4px 0 0' }}>
            {getFieldError('message')}
          </p>
        )}
      </div>
      
      <button
        type="submit"
        disabled={isSubmitting}
        style={{
          width: '100%', padding: '12px',
          background: '#007bff', color: 'white',
          border: 'none', borderRadius: '6px',
          fontSize: '16px', cursor: isSubmitting ? 'not-allowed' : 'pointer'
        }}
      >
        {isSubmitting ? 'กำลังส่ง...' : 'ส่งข้อความ'}
      </button>
    </form>
  );
}
```

---

## Step 472: useGeolocation Hook

```jsx
function useGeolocation(options = {}) {
  const [state, setState] = useState({
    loading: true,
    accuracy: null,
    altitude: null,
    altitudeAccuracy: null,
    heading: null,
    latitude: null,
    longitude: null,
    speed: null,
    timestamp: null,
    error: null
  });

  useEffect(() => {
    if (!navigator.geolocation) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: { code: 0, message: 'Geolocation is not supported' }
      }));
      return;
    }

    const onSuccess = (position) => {
      setState({
        loading: false,
        accuracy: position.coords.accuracy,
        altitude: position.coords.altitude,
        altitudeAccuracy: position.coords.altitudeAccuracy,
        heading: position.coords.heading,
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
        speed: position.coords.speed,
        timestamp: position.timestamp,
        error: null
      });
    };

    const onError = (error) => {
      setState(prev => ({
        ...prev,
        loading: false,
        error
      }));
    };

    const watchId = navigator.geolocation.watchPosition(
      onSuccess,
      onError,
      options
    );

    return () => navigator.geolocation.clearWatch(watchId);
  }, []); // eslint-disable-line

  return state;
}

// การใช้งาน
function LocationDisplay() {
  const { latitude, longitude, accuracy, loading, error } = useGeolocation({
    enableHighAccuracy: true,
    timeout: 10000
  });

  if (loading) return <p>กำลังโหลดตำแหน่ง...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <p>ละติจูด: {latitude?.toFixed(6)}</p>
      <p>ลองจิจูด: {longitude?.toFixed(6)}</p>
      <p>ความแม่นยำ: ±{accuracy?.toFixed(0)} เมตร</p>
      <a
        href={`https://www.google.com/maps?q=${latitude},${longitude}`}
        target="_blank"
        rel="noopener noreferrer"
      >
        เปิดใน Google Maps
      </a>
    </div>
  );
}
```

---

## Step 473: useCopyToClipboard Hook

```jsx
function useCopyToClipboard(timeout = 2000) {
  const [copied, setCopied] = useState(false);
  const timerRef = useRef(null);

  const copy = useCallback(async (text) => {
    try {
      if (navigator.clipboard?.writeText) {
        await navigator.clipboard.writeText(text);
      } else {
        // Fallback สำหรับ older browsers
        const textArea = document.createElement('textarea');
        textArea.value = text;
        textArea.style.position = 'fixed';
        textArea.style.opacity = '0';
        document.body.appendChild(textArea);
        textArea.focus();
        textArea.select();
        document.execCommand('copy');
        document.body.removeChild(textArea);
      }
      
      setCopied(true);
      clearTimeout(timerRef.current);
      timerRef.current = setTimeout(() => setCopied(false), timeout);
      return true;
    } catch (error) {
      console.error('Failed to copy:', error);
      return false;
    }
  }, [timeout]);

  useEffect(() => () => clearTimeout(timerRef.current), []);

  return [copied, copy];
}

// การใช้งาน
function CodeBlock({ code, language }) {
  const [copied, copy] = useCopyToClipboard();

  return (
    <div style={{ position: 'relative', background: '#1e1e1e', borderRadius: '8px', overflow: 'hidden' }}>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', padding: '8px 16px', background: '#2d2d2d' }}>
        <span style={{ color: '#9cdcfe', fontSize: '12px' }}>{language}</span>
        <button
          onClick={() => copy(code)}
          style={{
            background: copied ? '#28a745' : '#444',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            padding: '4px 12px',
            cursor: 'pointer',
            fontSize: '12px',
            transition: 'background 0.2s'
          }}
        >
          {copied ? '✓ Copied!' : 'Copy'}
        </button>
      </div>
      <pre style={{ margin: 0, padding: '16px', color: '#d4d4d4', overflowX: 'auto', fontSize: '14px' }}>
        <code>{code}</code>
      </pre>
    </div>
  );
}
```

---

## Step 474: useScrollPosition Hook

```jsx
function useScrollPosition(throttleMs = 50) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const throttleTimerRef = useRef(null);

  useEffect(() => {
    const updatePosition = () => {
      setPosition({ x: window.scrollX, y: window.scrollY });
    };

    const handleScroll = () => {
      if (throttleTimerRef.current) return;
      
      throttleTimerRef.current = setTimeout(() => {
        updatePosition();
        throttleTimerRef.current = null;
      }, throttleMs);
    };

    window.addEventListener('scroll', handleScroll, { passive: true });
    updatePosition(); // ตั้งค่าเริ่มต้น

    return () => {
      window.removeEventListener('scroll', handleScroll);
      clearTimeout(throttleTimerRef.current);
    };
  }, [throttleMs]);

  return position;
}

// Back to Top Button
function BackToTop() {
  const { y } = useScrollPosition();
  const isVisible = y > 300;

  const scrollToTop = () => {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  return (
    <button
      onClick={scrollToTop}
      style={{
        position: 'fixed',
        bottom: '24px',
        right: '24px',
        width: '48px',
        height: '48px',
        borderRadius: '50%',
        background: '#007bff',
        color: 'white',
        border: 'none',
        cursor: 'pointer',
        fontSize: '20px',
        boxShadow: '0 4px 12px rgba(0,0,0,0.2)',
        opacity: isVisible ? 1 : 0,
        transform: isVisible ? 'scale(1)' : 'scale(0)',
        transition: 'all 0.3s ease',
        zIndex: 100
      }}
      aria-label="Back to top"
    >
      ↑
    </button>
  );
}
```

---

## Step 475: useWindowSize Hook

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    let raf;
    
    const handleResize = () => {
      cancelAnimationFrame(raf);
      raf = requestAnimationFrame(() => {
        setSize({
          width: window.innerWidth,
          height: window.innerHeight
        });
      });
    };

    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
      cancelAnimationFrame(raf);
    };
  }, []);

  return size;
}
```

---

## Step 476: useToggle Hook

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, toggle, setTrue, setFalse];
}

// การใช้งาน
function AccordionPanel({ title, children }) {
  const [isOpen, toggle] = useToggle(false);
  
  return (
    <div>
      <button onClick={toggle}>{isOpen ? '▼' : '▶'} {title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}
```

---

## Step 477: useCountdown Hook

```jsx
function useCountdown({ 
  seconds: initialSeconds, 
  onComplete 
}) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isRunning, setIsRunning] = useState(false);
  const onCompleteRef = useRef(onComplete);
  
  useEffect(() => { onCompleteRef.current = onComplete; }, [onComplete]);

  useEffect(() => {
    if (!isRunning) return;
    if (seconds <= 0) {
      setIsRunning(false);
      onCompleteRef.current?.();
      return;
    }

    const timer = setTimeout(() => setSeconds(s => s - 1), 1000);
    return () => clearTimeout(timer);
  }, [isRunning, seconds]);

  const start = useCallback(() => setIsRunning(true), []);
  const pause = useCallback(() => setIsRunning(false), []);
  const reset = useCallback(() => {
    setIsRunning(false);
    setSeconds(initialSeconds);
  }, [initialSeconds]);

  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;

  return {
    seconds,
    minutes,
    remainingSeconds,
    isRunning,
    isComplete: seconds === 0,
    start, pause, reset,
    formatted: `${String(minutes).padStart(2, '0')}:${String(remainingSeconds).padStart(2, '0')}`
  };
}

// การใช้งาน
function OTPTimer({ onResend }) {
  const { formatted, isComplete, start, reset } = useCountdown({
    seconds: 60,
    onComplete: () => console.log('OTP expired')
  });

  useEffect(() => { start(); }, [start]);

  const handleResend = () => {
    reset();
    start();
    onResend?.();
  };

  return (
    <div>
      {isComplete ? (
        <button onClick={handleResend}>ส่ง OTP อีกครั้ง</button>
      ) : (
        <p>รอ {formatted} เพื่อส่ง OTP ใหม่</p>
      )}
    </div>
  );
}
```

---

## Step 478: useAsync Hook

```jsx
function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'success' | 'error'
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    setStatus('loading');
    setData(null);
    setError(null);

    try {
      const result = await asyncFunction(...args);
      setData(result);
      setStatus('success');
      return result;
    } catch (e) {
      setError(e);
      setStatus('error');
      throw e;
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) execute();
  }, [execute, immediate]);

  return {
    execute,
    status,
    data,
    error,
    isIdle: status === 'idle',
    isLoading: status === 'loading',
    isSuccess: status === 'success',
    isError: status === 'error'
  };
}
```

---

## Step 479: useIntersectionObserver Hook

```jsx
function useIntersectionObserver(options = {}) {
  const [entry, setEntry] = useState(null);
  const elementRef = useRef(null);
  
  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => setEntry(entry),
      options
    );
    
    observer.observe(element);
    return () => observer.disconnect();
  }, [options.threshold, options.root, options.rootMargin]);
  
  return [elementRef, entry];
}

// Lazy Loading Image
function LazyImage({ src, alt, ...props }) {
  const [ref, entry] = useIntersectionObserver({ threshold: 0.1, rootMargin: '200px' });
  const [loaded, setLoaded] = useState(false);
  const shouldLoad = entry?.isIntersecting;

  return (
    <div ref={ref} style={{ background: '#f0f0f0', minHeight: '200px', ...props.style }}>
      {shouldLoad && (
        <img
          src={src}
          alt={alt}
          onLoad={() => setLoaded(true)}
          style={{ opacity: loaded ? 1 : 0, transition: 'opacity 0.3s', width: '100%' }}
          {...props}
        />
      )}
    </div>
  );
}
```

---

## Step 480: Testing Custom Hooks

```jsx
// __tests__/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from '../hooks/useCounter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter(0));
    expect(result.current.count).toBe(0);
  });

  it('should increment', () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(1);
  });

  it('should not exceed max', () => {
    const { result } = renderHook(() => useCounter(9, { max: 10 }));
    act(() => {
      result.current.increment();
      result.current.increment(); // ควรหยุดที่ 10
    });
    expect(result.current.count).toBe(10);
    expect(result.current.isAtMax).toBe(true);
  });

  it('should reset to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    act(() => result.current.increment());
    act(() => result.current.reset());
    expect(result.current.count).toBe(5);
  });
});

// __tests__/useLocalStorage.test.js
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from '../hooks/useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('should use initial value when no stored value', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', 'default'));
    expect(result.current[0]).toBe('default');
  });

  it('should persist value', () => {
    const { result } = renderHook(() => useLocalStorage('test-key', ''));
    act(() => result.current[1]('new-value'));
    expect(localStorage.getItem('test-key')).toBe('"new-value"');
  });
});
```

---

## Step 481: Best Practices

### 1. ชื่อ Hook ต้องขึ้นต้นด้วย `use`

```jsx
// ✅ ถูก
function useFetch() {}
function useLocalStorage() {}
function useWindowSize() {}

// ❌ ผิด - React ไม่รู้ว่าเป็น Hook
function fetchData() { useState(null); } // Error!
```

### 2. Return อะไรก็ได้ (object, array, single value)

```jsx
// Return array (เหมือน useState)
function useToggle() {
  return [value, toggle];
}

// Return object (ถ้ามีหลาย values)
function useFetch() {
  return { data, loading, error, refetch };
}

// Return single value
function useWindowWidth() {
  return width;
}
```

### 3. Document dependencies ให้ชัดเจน

```jsx
// ✅ ดี: บอกว่า hook ต้องการอะไร
/**
 * @param {string} url - URL to fetch
 * @param {object} options - fetch options (memoize ก่อนส่ง!)
 * @returns {{ data, loading, error }}
 */
function useFetch(url, options) {}
```

---

## สรุป Custom Hooks

| Hook | ใช้สำหรับ |
|---|---|
| `useFetch` | Data fetching |
| `useLocalStorage` | Persist state ใน localStorage |
| `useDebounce` | Delay state updates |
| `useMediaQuery` | Responsive breakpoints |
| `useOnClickOutside` | Detect clicks outside element |
| `useKeyPress` | Keyboard interactions |
| `useForm` | Form state management |
| `useToggle` | Boolean state toggling |
| `useScrollPosition` | Track scroll position |
| `useCopyToClipboard` | Copy text to clipboard |

---

## Quiz และแบบฝึกหัด

### Quiz

1. กฎของ Hooks มีอะไรบ้าง?
2. ทำไมชื่อ Custom Hook ต้องขึ้นต้นด้วย `use`?
3. Custom Hooks ต่างจาก regular functions อย่างไร?
4. เมื่อไหร่ควรสร้าง Custom Hook?

### แบบฝึกหัด

**Exercise 1**: สร้าง `useHistory` Hook

```jsx
// TODO: สร้าง useHistory ที่มี:
// - undo: ย้อนกลับ state ก่อนหน้า
// - redo: ทำซ้ำ
// - canUndo: boolean
// - canRedo: boolean
// - set(value): ตั้งค่าใหม่
// - clear(): ล้าง history

function useHistory(initialValue) {
  // implement here...
}
```

**Exercise 2**: สร้าง `useVirtualList` Hook

```jsx
// TODO: สร้าง hook สำหรับ virtualized list ที่:
// - แสดงเฉพาะ items ที่อยู่ใน viewport
// - รองรับ items จำนวนมาก (10,000+)
// - scroll smoothly

function useVirtualList({ items, itemHeight, containerHeight }) {
  // implement here...
}
```

**Exercise 3**: สร้าง `useNotification` Hook

```jsx
// TODO: สร้าง hook ที่:
// - ขอ permission จาก browser
// - ส่ง desktop notifications
// - รองรับ icon, title, body
// - Return { permission, notify, isSupported }

function useBrowserNotification() {
  // implement here...
}
```

---

## 🎉 ยินดีด้วย!

คุณเรียนจบ Part 20 แล้ว! คุณได้เรียนรู้:

- ✅ useEffect สำหรับ side effects
- ✅ Component Lifecycle
- ✅ Styling ใน React (CSS Modules, Tailwind, etc.)
- ✅ Component Composition Patterns
- ✅ Context API สำหรับ global state
- ✅ useContext ขั้นสูง
- ✅ useReducer สำหรับ complex state
- ✅ useMemo และ useCallback สำหรับ optimization
- ✅ useRef สำหรับ DOM และ mutable values
- ✅ Custom Hooks สำหรับ reusable logic

➡️ ไปต่อ: Part 21: React Router (coming soon)
