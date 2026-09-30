# Part 19: useRef Hook

## Step 436-460 | ระดับ: กลาง

---

## สารบัญ

- [Step 436: useRef คืออะไร?](#step-436-useref-คืออะไร)
- [Step 437: useRef สำหรับ DOM References](#step-437-useref-สำหรับ-dom-references)
- [Step 438: useRef สำหรับ Mutable Values](#step-438-useref-สำหรับ-mutable-values)
- [Step 439: useRef vs useState](#step-439-useref-vs-usestate)
- [Step 440: ตัวอย่าง Focus Management](#step-440-ตัวอย่าง-focus-management)
- [Step 441: ตัวอย่าง Scroll Control](#step-441-ตัวอย่าง-scroll-control)
- [Step 442: ตัวอย่าง Canvas Animation](#step-442-ตัวอย่าง-canvas-animation)
- [Step 443: useRef กับ Previous Value](#step-443-useref-กับ-previous-value)
- [Step 444: useRef กับ Timers](#step-444-useref-กับ-timers)
- [Step 445: forwardRef](#step-445-forwardref)
- [Step 446: useImperativeHandle](#step-446-useimperativehandle)
- [Step 447: useRef Anti-patterns](#step-447-useref-anti-patterns)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 436: useRef คืออะไร?

`useRef` ให้เข้าถึง:
1. **DOM Elements** โดยตรง
2. **Mutable value** ที่ไม่ trigger re-render เมื่อเปลี่ยน

```jsx
import { useRef } from 'react';

// useRef returns a mutable ref object
const myRef = useRef(initialValue);
// myRef = { current: initialValue }

// การเข้าถึง value
myRef.current; // อ่าน
myRef.current = newValue; // เขียน (ไม่ trigger re-render)
```

### ความแตกต่างหลัก

| | useState | useRef |
|---|---|---|
| Trigger re-render | ✅ ใช่ | ❌ ไม่ |
| Persistent across renders | ✅ ใช่ | ✅ ใช่ |
| Mutable | ✅ (ผ่าน setter) | ✅ (โดยตรง) |
| ใช้สำหรับ | State ที่แสดงบน UI | DOM refs, timers, previous values |

---

## Step 437: useRef สำหรับ DOM References

```jsx
import { useRef, useEffect } from 'react';

// ตัวอย่างพื้นฐาน: focus input
function AutoFocusInput() {
  const inputRef = useRef(null); // เริ่มต้น null

  useEffect(() => {
    // เข้าถึง DOM element ผ่าน ref.current
    inputRef.current?.focus();
  }, []); // focus เมื่อ mount

  return (
    <input
      ref={inputRef} // ← React จะ set inputRef.current = DOM element
      type="text"
      placeholder="Auto-focused!"
    />
  );
}

// ตัวอย่าง: อ่านค่าจาก DOM
function TextMeasurer() {
  const textRef = useRef(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useEffect(() => {
    if (textRef.current) {
      const { offsetWidth, offsetHeight } = textRef.current;
      setDimensions({ width: offsetWidth, height: offsetHeight });
    }
  }, []);

  return (
    <div>
      <p ref={textRef} style={{ display: 'inline-block' }}>
        วัดขนาดข้อความนี้
      </p>
      <p>
        ขนาด: {dimensions.width} x {dimensions.height} px
      </p>
    </div>
  );
}

// ตัวอย่าง: Video Player Control
function VideoPlayer({ src }) {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);

  const play = () => {
    videoRef.current?.play();
    setIsPlaying(true);
  };

  const pause = () => {
    videoRef.current?.pause();
    setIsPlaying(false);
  };

  const setPlaybackRate = (rate) => {
    if (videoRef.current) {
      videoRef.current.playbackRate = rate;
    }
  };

  const seekTo = (time) => {
    if (videoRef.current) {
      videoRef.current.currentTime = time;
    }
  };

  return (
    <div>
      <video ref={videoRef} src={src} onEnded={() => setIsPlaying(false)} />
      <div>
        <button onClick={isPlaying ? pause : play}>
          {isPlaying ? '⏸️ Pause' : '▶️ Play'}
        </button>
        <button onClick={() => seekTo(0)}>⏮️ Restart</button>
        <select onChange={e => setPlaybackRate(Number(e.target.value))}>
          <option value="0.5">0.5x</option>
          <option value="1" selected>1x</option>
          <option value="1.5">1.5x</option>
          <option value="2">2x</option>
        </select>
      </div>
    </div>
  );
}
```

---

## Step 438: useRef สำหรับ Mutable Values

```jsx
// useRef เก็บค่าที่เปลี่ยนได้ โดยไม่ trigger re-render

// ตัวอย่าง 1: นับจำนวน render
function RenderCounter() {
  const [value, setValue] = useState('');
  const renderCount = useRef(0);
  
  renderCount.current += 1; // เพิ่มทุก render แต่ไม่ trigger re-render

  return (
    <div>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        placeholder="พิมพ์อะไรก็ได้..."
      />
      <p>Render count: {renderCount.current}</p>
    </div>
  );
}

// ตัวอย่าง 2: เก็บ previous value
function PreviousValueTracker({ value }) {
  const prevValueRef = useRef(value);
  
  useEffect(() => {
    prevValueRef.current = value; // update หลัง render
  });
  
  const prevValue = prevValueRef.current; // ค่าจาก render ก่อน

  return (
    <div>
      <p>Current: {value}</p>
      <p>Previous: {prevValue}</p>
    </div>
  );
}

// ตัวอย่าง 3: เก็บ reference ที่ไม่ต้องการ trigger re-render
function DataFetcher({ userId }) {
  const [data, setData] = useState(null);
  const abortControllerRef = useRef(null); // เก็บ AbortController

  useEffect(() => {
    // ยกเลิก request เก่าก่อน
    abortControllerRef.current?.abort();
    
    // สร้าง controller ใหม่
    abortControllerRef.current = new AbortController();
    
    fetch(`/api/users/${userId}`, { 
      signal: abortControllerRef.current.signal 
    })
      .then(r => r.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });
    
    return () => abortControllerRef.current?.abort();
  }, [userId]);
  
  return <div>{JSON.stringify(data)}</div>;
}
```

---

## Step 439: useRef vs useState

```jsx
// เมื่อไหร่ใช้ useRef vs useState

// useState: เมื่อต้องการให้ UI อัปเดต
function StateCounter() {
  const [count, setCount] = useState(0); // ✅ แสดงบน UI
  
  return (
    <div>
      <p>Count: {count}</p> {/* จะอัปเดตเมื่อ count เปลี่ยน */}
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}

// useRef: เมื่อต้องการเก็บค่าแต่ไม่ต้องแสดง UI
function RefCounter() {
  const countRef = useRef(0); // ❌ ไม่แสดงบน UI อัตโนมัติ
  
  return (
    <div>
      {/* countRef.current จะเป็นค่าล่าสุด แต่ UI จะไม่ re-render */}
      <p>Count: {countRef.current}</p> {/* จะไม่อัปเดต! */}
      <button onClick={() => { countRef.current += 1; }}>+1</button>
    </div>
  );
}

// ✅ ใช้ทั้งสองร่วมกัน
function HybridCounter() {
  const [displayCount, setDisplayCount] = useState(0);
  const actualCount = useRef(0); // track ค่าจริง
  
  const increment = () => {
    actualCount.current += 1;
    if (actualCount.current % 10 === 0) {
      // อัปเดต UI เมื่อครบ 10
      setDisplayCount(actualCount.current);
    }
  };
  
  return (
    <div>
      <p>Actual clicks: {actualCount.current}</p>
      <p>Displayed (every 10): {displayCount}</p>
      <button onClick={increment}>Click Me</button>
    </div>
  );
}
```

---

## Step 440: ตัวอย่าง Focus Management

```jsx
// Auto-focus form field
function LoginForm() {
  const emailRef = useRef(null);
  const passwordRef = useRef(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    emailRef.current?.focus(); // focus เมื่อ mount
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    const email = emailRef.current.value;
    const password = passwordRef.current.value;
    
    if (!email) {
      setError('กรุณาใส่ email');
      emailRef.current?.focus(); // focus กลับไปที่ field ที่ error
      return;
    }
    
    if (!password) {
      setError('กรุณาใส่ password');
      passwordRef.current?.focus();
      return;
    }
    
    // submit...
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <div>
        <label>Email:</label>
        <input
          ref={emailRef}
          type="email"
          onKeyDown={e => e.key === 'Enter' && passwordRef.current?.focus()}
        />
      </div>
      <div>
        <label>Password:</label>
        <input
          ref={passwordRef}
          type="password"
        />
      </div>
      <button type="submit">Login</button>
    </form>
  );
}

// Focus Trap (สำหรับ Modal)
function useFocusTrap(isActive) {
  const containerRef = useRef(null);
  
  useEffect(() => {
    if (!isActive) return;
    
    const container = containerRef.current;
    if (!container) return;
    
    const focusableElements = container.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    if (focusableElements.length === 0) return;
    
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];
    
    firstElement.focus();
    
    function handleKeyDown(e) {
      if (e.key !== 'Tab') return;
      
      if (e.shiftKey) {
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        }
      } else {
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    }
    
    container.addEventListener('keydown', handleKeyDown);
    return () => container.removeEventListener('keydown', handleKeyDown);
  }, [isActive]);
  
  return containerRef;
}

// Modal ที่มี Focus Trap
function AccessibleModal({ isOpen, onClose, title, children }) {
  const containerRef = useFocusTrap(isOpen);
  
  if (!isOpen) return null;
  
  return (
    <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
      <div style={{ position: 'fixed', inset: 0, background: 'rgba(0,0,0,0.5)' }} onClick={onClose} />
      <div
        ref={containerRef}
        style={{
          position: 'fixed', top: '50%', left: '50%',
          transform: 'translate(-50%, -50%)',
          background: 'white', padding: '24px', borderRadius: '8px',
          minWidth: '300px', zIndex: 100
        }}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

---

## Step 441: ตัวอย่าง Scroll Control

```jsx
// Scroll to Bottom (Chat)
function ChatWindow({ messages }) {
  const bottomRef = useRef(null);
  const containerRef = useRef(null);
  const [autoScroll, setAutoScroll] = useState(true);

  useEffect(() => {
    if (autoScroll) {
      bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
    }
  }, [messages, autoScroll]);

  const handleScroll = () => {
    const container = containerRef.current;
    if (!container) return;
    
    const { scrollTop, scrollHeight, clientHeight } = container;
    const isAtBottom = scrollHeight - scrollTop - clientHeight < 50;
    setAutoScroll(isAtBottom);
  };

  return (
    <div>
      <div
        ref={containerRef}
        onScroll={handleScroll}
        style={{ height: '400px', overflowY: 'auto', border: '1px solid #ccc' }}
      >
        {messages.map(msg => (
          <div key={msg.id} style={{ padding: '8px', borderBottom: '1px solid #eee' }}>
            <strong>{msg.author}:</strong> {msg.text}
          </div>
        ))}
        <div ref={bottomRef} /> {/* Anchor สำหรับ scroll to bottom */}
      </div>
      {!autoScroll && (
        <button onClick={() => {
          setAutoScroll(true);
          bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
        }}>
          ⬇️ Scroll to bottom
        </button>
      )}
    </div>
  );
}

// Scroll Spy
function useScrollSpy(sectionIds) {
  const [activeSection, setActiveSection] = useState(sectionIds[0]);
  
  useEffect(() => {
    const observers = sectionIds.map(id => {
      const element = document.getElementById(id);
      if (!element) return null;
      
      const observer = new IntersectionObserver(
        ([entry]) => {
          if (entry.isIntersecting) {
            setActiveSection(id);
          }
        },
        { threshold: 0.5 }
      );
      
      observer.observe(element);
      return observer;
    }).filter(Boolean);
    
    return () => observers.forEach(obs => obs.disconnect());
  }, [sectionIds]);
  
  return activeSection;
}
```

---

## Step 442: ตัวอย่าง Canvas Animation

```jsx
function AnimatedCanvas({ width = 600, height = 400 }) {
  const canvasRef = useRef(null);
  const animationRef = useRef(null); // เก็บ requestAnimationFrame ID
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d');
    let balls = Array.from({ length: 20 }, () => ({
      x: Math.random() * width,
      y: Math.random() * height,
      vx: (Math.random() - 0.5) * 4,
      vy: (Math.random() - 0.5) * 4,
      radius: Math.random() * 15 + 5,
      color: `hsl(${Math.random() * 360}, 70%, 60%)`
    }));
    
    function draw() {
      ctx.clearRect(0, 0, width, height);
      
      balls.forEach(ball => {
        // Update position
        ball.x += ball.vx;
        ball.y += ball.vy;
        
        // Bounce off walls
        if (ball.x - ball.radius < 0 || ball.x + ball.radius > width) ball.vx *= -1;
        if (ball.y - ball.radius < 0 || ball.y + ball.radius > height) ball.vy *= -1;
        
        // Draw ball
        ctx.beginPath();
        ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
        ctx.fillStyle = ball.color;
        ctx.fill();
      });
      
      animationRef.current = requestAnimationFrame(draw);
    }
    
    if (isRunning) {
      draw();
    }
    
    return () => {
      if (animationRef.current) {
        cancelAnimationFrame(animationRef.current);
      }
    };
  }, [isRunning, width, height]);

  return (
    <div>
      <canvas
        ref={canvasRef}
        width={width}
        height={height}
        style={{ border: '2px solid #333', borderRadius: '8px', display: 'block' }}
      />
      <button
        onClick={() => setIsRunning(r => !r)}
        style={{ marginTop: '8px', padding: '8px 16px' }}
      >
        {isRunning ? '⏸️ Pause' : '▶️ Start'} Animation
      </button>
    </div>
  );
}
```

---

## Step 443: useRef กับ Previous Value

```jsx
// Custom Hook: usePrevious
function usePrevious(value) {
  const ref = useRef(undefined);
  
  useEffect(() => {
    ref.current = value; // อัปเดต ref หลัง render
  });
  
  return ref.current; // return ค่าจาก render ก่อน
}

// การใช้งาน
function PriceTracker({ price }) {
  const prevPrice = usePrevious(price);
  
  const priceChange = prevPrice !== undefined ? price - prevPrice : 0;
  const direction = priceChange > 0 ? '📈' : priceChange < 0 ? '📉' : '➡️';
  
  return (
    <div>
      <p>ราคาปัจจุบัน: ฿{price.toLocaleString()}</p>
      {prevPrice !== undefined && (
        <p>
          {direction} เปลี่ยนแปลง: {priceChange > 0 ? '+' : ''}{priceChange.toLocaleString()}
          {' '}({((priceChange / prevPrice) * 100).toFixed(2)}%)
        </p>
      )}
    </div>
  );
}

// useChangeEffect: รัน effect เฉพาะเมื่อ value เปลี่ยน (ไม่ใช่ครั้งแรก)
function useChangeEffect(effect, deps) {
  const isFirstRender = useRef(true);
  
  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }
    return effect();
  }, deps); // eslint-disable-line
}

// การใช้งาน
function UserList({ userId }) {
  const [users, setUsers] = useState([]);
  
  useChangeEffect(() => {
    // รันเฉพาะเมื่อ userId เปลี่ยน (ไม่รันครั้งแรก)
    console.log('userId changed to:', userId);
    fetchUsers(userId).then(setUsers);
  }, [userId]);
  
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

## Step 444: useRef กับ Timers

```jsx
// Debounce ด้วย useRef
function useDebounce(callback, delay) {
  const timerRef = useRef(null);
  
  const debouncedCallback = useCallback((...args) => {
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
  
  // Cleanup เมื่อ unmount
  useEffect(() => {
    return () => clearTimeout(timerRef.current);
  }, []);
  
  return debouncedCallback;
}

// Throttle ด้วย useRef
function useThrottle(callback, limit) {
  const inThrottle = useRef(false);
  
  return useCallback((...args) => {
    if (!inThrottle.current) {
      callback(...args);
      inThrottle.current = true;
      setTimeout(() => {
        inThrottle.current = false;
      }, limit);
    }
  }, [callback, limit]);
}

// Interval ด้วย useRef
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);
  
  useEffect(() => {
    savedCallback.current = callback; // อัปเดต ref ทุกครั้ง
  }, [callback]);
  
  useEffect(() => {
    if (delay === null) return; // null = หยุด interval
    
    const tick = () => savedCallback.current();
    const id = setInterval(tick, delay);
    return () => clearInterval(id);
  }, [delay]);
}

// การใช้งาน
function SearchInput() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const performSearch = useCallback(async (term) => {
    if (!term) { setResults([]); return; }
    const data = await fetch(`/api/search?q=${term}`).then(r => r.json());
    setResults(data);
  }, []);

  const debouncedSearch = useDebounce(performSearch, 500);

  const handleChange = (e) => {
    setQuery(e.target.value);
    debouncedSearch(e.target.value);
  };

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Search..." />
      <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>
    </div>
  );
}

// Auto-save ด้วย useInterval
function AutoSaveEditor() {
  const [content, setContent] = useState('');
  const [lastSaved, setLastSaved] = useState(null);
  const [isDirty, setIsDirty] = useState(false);

  const save = useCallback(async () => {
    if (!isDirty) return;
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify({ content })
    });
    setLastSaved(new Date());
    setIsDirty(false);
  }, [content, isDirty]);

  useInterval(save, isDirty ? 3000 : null); // บันทึกทุก 3 วินาที ถ้ามีการเปลี่ยนแปลง

  return (
    <div>
      <textarea
        value={content}
        onChange={e => {
          setContent(e.target.value);
          setIsDirty(true);
        }}
        rows={10}
        style={{ width: '100%', padding: '8px' }}
      />
      <p style={{ color: isDirty ? '#ff9800' : '#4caf50' }}>
        {isDirty 
          ? '● กำลังรอบันทึก...' 
          : lastSaved 
            ? `✓ บันทึกเมื่อ ${lastSaved.toLocaleTimeString()}` 
            : 'ยังไม่ได้บันทึก'
        }
      </p>
    </div>
  );
}
```

---

## Step 445: forwardRef

`forwardRef` ให้ parent component เข้าถึง DOM element ของ child component

```jsx
import { forwardRef, useRef } from 'react';

// ❌ ปกติ parent ไม่สามารถ ref child component
function Input({ placeholder, ...props }) {
  return <input placeholder={placeholder} {...props} />;
}

// ❌ จะ error!
// const inputRef = useRef(null);
// <Input ref={inputRef} /> // ← ไม่ work

// ✅ forwardRef ส่ง ref ลงไปให้ DOM element
const Input = forwardRef(function({ placeholder, label, error, ...props }, ref) {
  return (
    <div>
      {label && <label>{label}</label>}
      <input
        ref={ref} // ← forward ref ไปที่ DOM element
        placeholder={placeholder}
        style={{ border: error ? '1px solid red' : '1px solid #ccc' }}
        {...props}
      />
      {error && <p style={{ color: 'red', fontSize: '12px' }}>{error}</p>}
    </div>
  );
});

// ตั้งชื่อ component (สำหรับ DevTools)
Input.displayName = 'Input';

// การใช้งาน
function Form() {
  const firstNameRef = useRef(null);
  const lastNameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      firstName: firstNameRef.current.value,
      lastName: lastNameRef.current.value,
      email: emailRef.current.value
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input ref={firstNameRef} label="ชื่อ" placeholder="ชื่อ" />
      <Input ref={lastNameRef} label="นามสกุล" placeholder="นามสกุล" />
      <Input ref={emailRef} label="Email" type="email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### forwardRef กับ TypeScript

```typescript
import { forwardRef, InputHTMLAttributes } from 'react';

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, ...props }, ref) => {
    return (
      <div>
        {label && <label>{label}</label>}
        <input ref={ref} {...props} />
        {error && <p>{error}</p>}
      </div>
    );
  }
);
```

---

## Step 446: useImperativeHandle

`useImperativeHandle` กำหนด API ที่ expose ให้ parent ผ่าน ref

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

// Custom Input ที่มี imperative API
const FancyInput = forwardRef(function(props, ref) {
  const inputRef = useRef(null);
  const [value, setValue] = useState('');

  // กำหนด methods ที่ expose ให้ parent
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
    blur: () => inputRef.current?.blur(),
    clear: () => {
      setValue('');
      inputRef.current?.focus();
    },
    getValue: () => value,
    setValue: (newValue) => setValue(newValue),
    select: () => inputRef.current?.select()
  }), [value]); // deps: เมื่อ value เปลี่ยน ให้ recreate handle

  return (
    <input
      ref={inputRef}
      value={value}
      onChange={e => setValue(e.target.value)}
      {...props}
    />
  );
});

// การใช้งาน
function SmartForm() {
  const inputRef = useRef(null);
  
  const handleReset = () => {
    inputRef.current?.clear(); // ✅ เรียก method ที่ expose ผ่าน useImperativeHandle
  };
  
  const handleFill = () => {
    inputRef.current?.setValue('pre-filled value');
    inputRef.current?.focus();
  };
  
  const handleLog = () => {
    console.log('Current value:', inputRef.current?.getValue());
  };

  return (
    <div>
      <FancyInput ref={inputRef} placeholder="Type here..." />
      <div style={{ marginTop: '8px', display: 'flex', gap: '8px' }}>
        <button onClick={handleReset}>Clear</button>
        <button onClick={handleFill}>Pre-fill</button>
        <button onClick={handleLog}>Log Value</button>
      </div>
    </div>
  );
}
```

### ตัวอย่าง: Scrollable List

```jsx
const ScrollableList = forwardRef(function({ items }, ref) {
  const containerRef = useRef(null);
  
  useImperativeHandle(ref, () => ({
    scrollToTop: () => {
      containerRef.current?.scrollTo({ top: 0, behavior: 'smooth' });
    },
    scrollToBottom: () => {
      containerRef.current?.scrollTo({ 
        top: containerRef.current.scrollHeight, 
        behavior: 'smooth' 
      });
    },
    scrollToItem: (index) => {
      const items = containerRef.current?.children;
      if (items && items[index]) {
        items[index].scrollIntoView({ behavior: 'smooth', block: 'nearest' });
      }
    }
  }));
  
  return (
    <ul
      ref={containerRef}
      style={{ height: '300px', overflowY: 'auto', border: '1px solid #ccc', listStyle: 'none', padding: 0 }}
    >
      {items.map((item, i) => (
        <li key={i} style={{ padding: '12px', borderBottom: '1px solid #eee' }}>
          {item}
        </li>
      ))}
    </ul>
  );
});

function App() {
  const listRef = useRef(null);
  const items = Array.from({ length: 50 }, (_, i) => `Item ${i + 1}`);
  
  return (
    <div>
      <div style={{ marginBottom: '8px', display: 'flex', gap: '8px' }}>
        <button onClick={() => listRef.current?.scrollToTop()}>Top</button>
        <button onClick={() => listRef.current?.scrollToBottom()}>Bottom</button>
        <button onClick={() => listRef.current?.scrollToItem(24)}>Item 25</button>
      </div>
      <ScrollableList ref={listRef} items={items} />
    </div>
  );
}
```

---

## Step 447: useRef Anti-patterns

### Anti-pattern 1: ใช้ ref แทน state สำหรับ UI

```jsx
// ❌ ผิด: ใช้ ref สำหรับ value ที่ต้องแสดงบน UI
function BadCounter() {
  const countRef = useRef(0);
  
  return (
    <div>
      {/* จะไม่อัปเดตเมื่อ countRef.current เปลี่ยน! */}
      <p>Count: {countRef.current}</p>
      <button onClick={() => { countRef.current += 1; }}>+1</button>
    </div>
  );
}

// ✅ ถูก: ใช้ state สำหรับ UI
function GoodCounter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

### Anti-pattern 2: Read ref during render

```jsx
// ❌ ผิด: อ่าน ref.current ระหว่าง render (ค่าอาจไม่ถูกต้อง)
function BadRefRead() {
  const ref = useRef(null);
  
  // ❌ ref.current ยังไม่ถูก set ตอน first render
  const width = ref.current?.offsetWidth || 0;
  
  return <div ref={ref}>Width: {width}</div>;
}

// ✅ ถูก: อ่าน ref ใน useEffect
function GoodRefRead() {
  const ref = useRef(null);
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    // ✅ อ่านหลัง render (DOM ถูก set แล้ว)
    setWidth(ref.current?.offsetWidth || 0);
  }, []);
  
  return <div ref={ref}>Width: {width}</div>;
}
```

---

## สรุป useRef

| Use Case | Code |
|---|---|
| DOM reference | `const ref = useRef(null); <div ref={ref} />` |
| Mutable value | `const count = useRef(0); count.current++` |
| Previous value | `usePrevious(value)` custom hook |
| Timer | `const timer = useRef(null); timer.current = setTimeout(...)` |
| Forward ref | `const Input = forwardRef((props, ref) => <input ref={ref} />)` |
| Imperative API | `useImperativeHandle(ref, () => ({ focus, clear }))` |

---

## Quiz และแบบฝึกหัด

### Quiz

1. useRef ต่างจาก useState อย่างไรในเรื่อง re-render?
2. ทำไมต้องใช้ forwardRef?
3. useImperativeHandle ใช้สำหรับอะไร?
4. เมื่อไหร่ควรอ่านค่าจาก ref.current?

### แบบฝึกหัด

**Exercise 1**: สร้าง `useInterval` custom hook

```jsx
// TODO: สร้าง useInterval ที่:
// - รับ callback และ delay
// - delay = null → หยุด interval
// - Cleanup ถูกต้อง
function useInterval(callback, delay) {
  // implement here...
}
```

**Exercise 2**: สร้าง Resizable Panel

```jsx
// TODO: สร้าง ResizablePanel ที่:
// - ใช้ useRef เพื่อ track mouse position
// - Drag handle เพื่อ resize
// - แสดงขนาดปัจจุบัน
function ResizablePanel({ children }) {
  // implement here...
}
```

**Exercise 3**: สร้าง custom Input ด้วย forwardRef

```jsx
// TODO: สร้าง NumberInput ที่:
// - รับ ref จาก parent
// - expose: increment(), decrement(), reset(), getValue()
// - มี min/max validation

const NumberInput = forwardRef(function({ min = 0, max = 100, step = 1, defaultValue = 0 }, ref) {
  // implement here...
});
```

---

➡️ ไปต่อ: [Part 20: Custom Hooks](./part-20-custom-hooks.md)
