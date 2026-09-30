# Part 07: Events และ Event Handlers
## Step 121-145 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 121: Events ใน React คืออะไร?](#step-121)
- [Step 122: Synthetic Events](#step-122)
- [Step 123: การเพิ่ม Event Listener](#step-123)
- [Step 124: onClick Event](#step-124)
- [Step 125: onChange Event](#step-125)
- [Step 126: onSubmit Event](#step-126)
- [Step 127: onFocus และ onBlur](#step-127)
- [Step 128: onMouseOver และ onMouseOut](#step-128)
- [Step 129: onMouseMove Event](#step-129)
- [Step 130: onKeyDown, onKeyUp, onKeyPress](#step-130)
- [Step 131: onScroll Event](#step-131)
- [Step 132: Event Object Properties](#step-132)
- [Step 133: preventDefault](#step-133)
- [Step 134: stopPropagation](#step-134)
- [Step 135: Event Bubbling](#step-135)
- [Step 136: Event Delegation](#step-136)
- [Step 137: Passing Arguments to Event Handlers](#step-137)
- [Step 138: Handler ใน Class vs Function](#step-138)
- [Step 139: Form Events - Input](#step-139)
- [Step 140: Form Events - Select](#step-140)
- [Step 141: Form Events - Checkbox](#step-141)
- [Step 142: Form Events - Radio](#step-142)
- [Step 143: Form Events - File Upload](#step-143)
- [Step 144: Complex Form ตัวอย่าง](#step-144)
- [Step 145: Custom Event Patterns](#step-145)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 121: Events ใน React คืออะไร? {#step-121}

Events ใน React คือการตอบสนองต่อการกระทำของผู้ใช้ เช่น คลิก พิมพ์ เลื่อน เป็นต้น

### ความแตกต่างจาก HTML Events

| Feature | HTML | React |
|---------|------|-------|
| ชื่อ Event | lowercase `onclick` | camelCase `onClick` |
| Handler | String `"handleClick()"` | Function `{handleClick}` |
| preventDefault | `return false` | `e.preventDefault()` |
| Event Object | Native DOM Event | Synthetic Event |

```html
<!-- HTML -->
<button onclick="handleClick()">คลิก</button>
<input onchange="handleChange(event)">
```

```jsx
// React
<button onClick={handleClick}>คลิก</button>
<input onChange={handleChange} />
```

### การเขียน Event Handler

```jsx
function EventDemo() {
  // วิธีที่ 1: Named function
  const handleClick = () => {
    console.log("คลิกแล้ว!");
  };
  
  // วิธีที่ 2: Inline arrow function
  return (
    <div>
      <button onClick={handleClick}>Named Handler</button>
      
      {/* Inline */}
      <button onClick={() => console.log("Inline!")}>
        Inline Handler
      </button>
      
      {/* ❌ ผิด - เรียก function ทันที ไม่ใช่ส่ง reference */}
      {/* <button onClick={handleClick()}>คลิก</button> */}
    </div>
  );
}
```

> **Warning:** อย่าลืม `{}` ครอบ handler และห้ามใส่ `()` ต่อท้าย ไม่งั้น function จะถูกเรียกทันทีตอน render!

---

## Step 122: Synthetic Events {#step-122}

React ใช้ **SyntheticEvent** ซึ่งเป็น wrapper ของ native DOM event

### ทำไมต้องมี Synthetic Events?

1. **Cross-browser compatibility** - ทำงานเหมือนกันทุก browser
2. **Performance** - React ใช้ event delegation และ event pooling
3. **Consistent API** - API เดียวกันทุก event type

```jsx
function SyntheticEventDemo() {
  const handleClick = (e) => {
    // Synthetic Event properties
    console.log(e.type);           // "click"
    console.log(e.target);         // DOM element ที่ถูก click
    console.log(e.currentTarget);  // Element ที่ผูก handler
    console.log(e.nativeEvent);    // Native DOM Event
    console.log(e.timeStamp);      // เวลาที่เกิด event
    
    // React-specific
    console.log(e.bubbles);        // เกิด bubble ไหม?
    console.log(e.cancelable);     // ยกเลิกได้ไหม?
    console.log(e.isTrusted);      // Event จาก user จริงๆ ไหม?
  };
  
  return <button onClick={handleClick}>ทดสอบ Synthetic Event</button>;
}
```

### Async Event Handling

React 17+ ไม่มีปัญหาเรื่อง event pooling แล้ว แต่ React 16 ต้องระวัง

```jsx
// React 17+: ไม่มีปัญหา
const handleChange = (e) => {
  const value = e.target.value;
  setTimeout(() => {
    console.log(value); // ✅ ใช้ได้
  }, 1000);
};

// React 16: ต้องเก็บค่าก่อน (legacy)
const handleChangeLegacy = (e) => {
  const value = e.target.value; // เก็บค่าไว้ก่อน
  e.persist(); // หรือ persist event
  setTimeout(() => {
    console.log(value); // ✅ ใช้ value แทน e.target.value
  }, 1000);
};
```

---

## Step 123: การเพิ่ม Event Listener {#step-123}

### วิธีหลัก: JSX Event Props

```jsx
function EventListeners() {
  return (
    <div
      onClick={() => console.log("div clicked")}
      onMouseEnter={() => console.log("mouse entered")}
      onMouseLeave={() => console.log("mouse left")}
    >
      <button 
        onClick={() => console.log("button clicked")}
        onDoubleClick={() => console.log("double clicked")}
        onContextMenu={(e) => {
          e.preventDefault();
          console.log("right clicked");
        }}
      >
        คลิกฉัน
      </button>
    </div>
  );
}
```

### addEventListener กับ useEffect (สำหรับ window/document)

```jsx
import { useEffect } from 'react';

function KeyboardShortcuts() {
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.ctrlKey && e.key === 's') {
        e.preventDefault();
        console.log("บันทึก!");
      }
    };
    
    // เพิ่ม listener
    window.addEventListener('keydown', handleKeyDown);
    
    // Cleanup - ลบ listener เมื่อ unmount
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []); // [] = ทำครั้งเดียวตอน mount
  
  return <div>กด Ctrl+S เพื่อบันทึก</div>;
}
```

---

## Step 124: onClick Event {#step-124}

`onClick` เป็น event ที่ใช้บ่อยที่สุด

```jsx
function ClickDemo() {
  const [clickCount, setClickCount] = useState(0);
  const [lastButton, setLastButton] = useState('');
  
  const handleClick = (buttonName) => {
    setClickCount(prev => prev + 1);
    setLastButton(buttonName);
  };
  
  return (
    <div>
      <p>คลิกทั้งหมด: {clickCount}</p>
      <p>ปุ่มล่าสุด: {lastButton}</p>
      
      <button onClick={() => handleClick("ปุ่ม A")}>ปุ่ม A</button>
      <button onClick={() => handleClick("ปุ่ม B")}>ปุ่ม B</button>
      
      {/* Double click */}
      <div onDoubleClick={() => alert("Double clicked!")}>
        ดับเบิ้ลคลิกที่นี่
      </div>
      
      {/* Right click */}
      <div onContextMenu={(e) => {
        e.preventDefault();
        alert("Right clicked!");
      }}>
        Right click ที่นี่
      </div>
    </div>
  );
}
```

### Mouse Button Detection

```jsx
function MouseButtonDemo() {
  const handleMouseDown = (e) => {
    switch (e.button) {
      case 0: console.log("Left click"); break;
      case 1: console.log("Middle click"); break;
      case 2: console.log("Right click"); break;
    }
  };
  
  return (
    <div onMouseDown={handleMouseDown}>
      คลิกปุ่มต่างๆ ของเมาส์ที่นี่
    </div>
  );
}
```

---

## Step 125: onChange Event {#step-125}

`onChange` ใน React ทำงานเหมือน `oninput` ใน HTML (ทำงานทุกครั้งที่ค่าเปลี่ยน)

```jsx
function InputDemo() {
  const [value, setValue] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  
  const handleChange = (e) => {
    setValue(e.target.value);
  };
  
  // Debounce - ล่าช้าการอัพเดท
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, 500);
    
    return () => clearTimeout(timer);
  }, [value]);
  
  return (
    <div>
      <input
        type="text"
        value={value}
        onChange={handleChange}
        placeholder="พิมพ์บางอย่าง..."
      />
      <p>ค่าทันที: {value}</p>
      <p>ค่า debounced (500ms): {debouncedValue}</p>
      <p>ความยาว: {value.length} ตัวอักษร</p>
    </div>
  );
}
```

### onChange กับ Input Types ต่างๆ

```jsx
function AllInputTypes() {
  const [values, setValues] = useState({
    text: '',
    number: 0,
    range: 50,
    color: '#ff0000',
    date: '',
    time: ''
  });
  
  const handleChange = (name) => (e) => {
    const value = e.target.type === 'number' 
      ? Number(e.target.value) 
      : e.target.value;
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  return (
    <div>
      <input type="text" value={values.text} onChange={handleChange('text')} />
      <input type="number" value={values.number} onChange={handleChange('number')} />
      <input type="range" value={values.range} onChange={handleChange('range')} min="0" max="100" />
      <input type="color" value={values.color} onChange={handleChange('color')} />
      <input type="date" value={values.date} onChange={handleChange('date')} />
      <input type="time" value={values.time} onChange={handleChange('time')} />
      
      <pre>{JSON.stringify(values, null, 2)}</pre>
    </div>
  );
}
```

---

## Step 126: onSubmit Event {#step-126}

`onSubmit` จัดการการส่งฟอร์ม

```jsx
function LoginForm() {
  const [credentials, setCredentials] = useState({
    username: '',
    password: ''
  });
  const [error, setError] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    setCredentials(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault(); // ป้องกัน page refresh
    
    // Validation
    if (!credentials.username || !credentials.password) {
      setError('กรุณากรอกข้อมูลให้ครบ');
      return;
    }
    
    setIsSubmitting(true);
    setError('');
    
    try {
      // Mock API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      if (credentials.username === 'admin' && credentials.password === '1234') {
        alert('เข้าสู่ระบบสำเร็จ!');
      } else {
        setError('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง');
      }
    } catch (err) {
      setError('เกิดข้อผิดพลาด กรุณาลองใหม่');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          type="text"
          placeholder="ชื่อผู้ใช้"
          value={credentials.username}
          onChange={handleChange}
          disabled={isSubmitting}
        />
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          placeholder="รหัสผ่าน"
          value={credentials.password}
          onChange={handleChange}
          disabled={isSubmitting}
        />
      </div>
      
      {error && <p style={{ color: 'red' }}>{error}</p>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
      </button>
    </form>
  );
}
```

---

## Step 127: onFocus และ onBlur {#step-127}

`onFocus` เกิดเมื่อ element ได้รับ focus, `onBlur` เมื่อเสีย focus

```jsx
function FocusBlurDemo() {
  const [focusedField, setFocusedField] = useState(null);
  const [touchedFields, setTouchedFields] = useState(new Set());
  const [errors, setErrors] = useState({});
  
  const validateField = (name, value) => {
    switch (name) {
      case 'email':
        return /\S+@\S+\.\S+/.test(value) ? '' : 'อีเมลไม่ถูกต้อง';
      case 'phone':
        return /^\d{10}$/.test(value) ? '' : 'เบอร์โทรต้องมี 10 หลัก';
      default:
        return value.trim() ? '' : 'กรุณากรอกข้อมูล';
    }
  };
  
  const handleBlur = (e) => {
    const { name, value } = e.target;
    setFocusedField(null);
    // Mark field as touched
    setTouchedFields(prev => new Set([...prev, name]));
    // Validate on blur
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error }));
  };
  
  const handleFocus = (e) => {
    setFocusedField(e.target.name);
  };
  
  const fields = ['name', 'email', 'phone'];
  
  return (
    <form>
      {fields.map(field => (
        <div key={field} style={{ marginBottom: 16 }}>
          <label>{field}</label>
          <input
            name={field}
            onFocus={handleFocus}
            onBlur={handleBlur}
            style={{
              border: `2px solid ${
                focusedField === field ? 'blue' : 
                errors[field] && touchedFields.has(field) ? 'red' : 
                touchedFields.has(field) && !errors[field] ? 'green' : 
                '#ccc'
              }`
            }}
          />
          {touchedFields.has(field) && errors[field] && (
            <span style={{ color: 'red', fontSize: 12 }}>{errors[field]}</span>
          )}
        </div>
      ))}
    </form>
  );
}
```

---

## Step 128: onMouseOver และ onMouseOut {#step-128}

Mouse hover events สำหรับ interactive UI

```jsx
function HoverCard({ title, content }) {
  const [isHovered, setIsHovered] = useState(false);
  
  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      style={{
        padding: 20,
        border: '1px solid #ccc',
        borderRadius: 8,
        cursor: 'pointer',
        transform: isHovered ? 'scale(1.05)' : 'scale(1)',
        boxShadow: isHovered ? '0 4px 20px rgba(0,0,0,0.1)' : 'none',
        transition: 'all 0.2s ease',
        backgroundColor: isHovered ? '#f0f0ff' : 'white'
      }}
    >
      <h3>{title}</h3>
      {isHovered && <p>{content}</p>}
    </div>
  );
}

// Tooltip component
function Tooltip({ text, children }) {
  const [showTooltip, setShowTooltip] = useState(false);
  
  return (
    <span
      style={{ position: 'relative', display: 'inline-block' }}
      onMouseEnter={() => setShowTooltip(true)}
      onMouseLeave={() => setShowTooltip(false)}
    >
      {children}
      {showTooltip && (
        <span
          style={{
            position: 'absolute',
            bottom: '100%',
            left: '50%',
            transform: 'translateX(-50%)',
            backgroundColor: '#333',
            color: 'white',
            padding: '4px 8px',
            borderRadius: 4,
            fontSize: 12,
            whiteSpace: 'nowrap',
            zIndex: 1000
          }}
        >
          {text}
        </span>
      )}
    </span>
  );
}

// การใช้งาน
<Tooltip text="ข้อมูลเพิ่มเติม">
  <button>Hover ที่ปุ่มนี้</button>
</Tooltip>
```

---

## Step 129: onMouseMove Event {#step-129}

```jsx
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isTracking, setIsTracking] = useState(false);
  
  const handleMouseMove = (e) => {
    if (!isTracking) return;
    const rect = e.currentTarget.getBoundingClientRect();
    setPosition({
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    });
  };
  
  return (
    <div>
      <button onClick={() => setIsTracking(!isTracking)}>
        {isTracking ? 'หยุดติดตาม' : 'เริ่มติดตาม'}
      </button>
      
      <div
        style={{
          width: 400,
          height: 300,
          border: '2px solid #333',
          position: 'relative',
          cursor: isTracking ? 'crosshair' : 'default',
          overflow: 'hidden',
          backgroundColor: '#f5f5f5'
        }}
        onMouseMove={handleMouseMove}
      >
        {isTracking && (
          <>
            {/* Crosshair */}
            <div style={{
              position: 'absolute',
              left: position.x,
              top: 0,
              width: 1,
              height: '100%',
              background: 'red',
              pointerEvents: 'none'
            }} />
            <div style={{
              position: 'absolute',
              left: 0,
              top: position.y,
              width: '100%',
              height: 1,
              background: 'red',
              pointerEvents: 'none'
            }} />
            <div
              style={{
                position: 'absolute',
                left: position.x + 10,
                top: position.y + 10,
                background: 'rgba(0,0,0,0.7)',
                color: 'white',
                padding: '2px 6px',
                borderRadius: 4,
                fontSize: 12,
                pointerEvents: 'none'
              }}
            >
              ({Math.round(position.x)}, {Math.round(position.y)})
            </div>
          </>
        )}
        <p style={{ textAlign: 'center', paddingTop: 130 }}>
          {isTracking ? 'เลื่อนเมาส์ที่นี่' : 'คลิกปุ่มเพื่อเริ่มติดตาม'}
        </p>
      </div>
    </div>
  );
}
```

---

## Step 130: onKeyDown, onKeyUp, onKeyPress {#step-130}

Keyboard events สำหรับ keyboard shortcuts และ input validation

```jsx
function KeyboardDemo() {
  const [pressedKeys, setPressedKeys] = useState(new Set());
  const [lastKey, setLastKey] = useState('');
  
  const handleKeyDown = (e) => {
    setLastKey(`${e.key} (keyCode: ${e.keyCode})`);
    setPressedKeys(prev => new Set([...prev, e.key]));
    
    // Keyboard shortcuts
    if (e.ctrlKey && e.key === 'Enter') {
      e.preventDefault();
      console.log("Ctrl+Enter !")
    }
  };
  
  const handleKeyUp = (e) => {
    setPressedKeys(prev => {
      const newSet = new Set(prev);
      newSet.delete(e.key);
      return newSet;
    });
  };
  
  return (
    <div>
      <input
        onKeyDown={handleKeyDown}
        onKeyUp={handleKeyUp}
        placeholder="กด keys ต่างๆ ที่นี่..."
        style={{ width: 300, padding: 8 }}
      />
      <p>Key ล่าสุด: {lastKey}</p>
      <p>Keys ที่กดอยู่: {[...pressedKeys].join(', ') || 'ไม่มี'}</p>
    </div>
  );
}
```

### Number-only Input

```jsx
function NumberOnlyInput({ value, onChange }) {
  const handleKeyDown = (e) => {
    // อนุญาตเฉพาะ: digits, backspace, delete, arrow keys, ctrl+a
    const allowedKeys = [
      'Backspace', 'Delete', 'ArrowLeft', 'ArrowRight',
      'Tab', 'End', 'Home', '-'
    ];
    
    if (
      !allowedKeys.includes(e.key) &&
      !e.key.match(/^[0-9]$/) &&
      !(e.ctrlKey && ['a', 'c', 'v', 'x'].includes(e.key))
    ) {
      e.preventDefault();
    }
  };
  
  return (
    <input
      type="text"
      value={value}
      onChange={onChange}
      onKeyDown={handleKeyDown}
      placeholder="ตัวเลขเท่านั้น"
    />
  );
}
```

### Enter Key Submit

```jsx
function ChatInput({ onSend }) {
  const [message, setMessage] = useState('');
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      if (e.shiftKey) {
        // Shift+Enter = newline ใน textarea
        return;
      }
      e.preventDefault();
      if (message.trim()) {
        onSend(message.trim());
        setMessage('');
      }
    }
  };
  
  return (
    <div>
      <textarea
        value={message}
        onChange={e => setMessage(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="พิมพ์ข้อความ... (Enter เพื่อส่ง, Shift+Enter สำหรับบรรทัดใหม่)"
        rows={3}
      />
      <button onClick={() => {
        if (message.trim()) {
          onSend(message.trim());
          setMessage('');
        }
      }}>
        ส่ง
      </button>
    </div>
  );
}
```

---

## Step 131: onScroll Event {#step-131}

```jsx
function ScrollDemo() {
  const [scrollPosition, setScrollPosition] = useState(0);
  const [showBackToTop, setShowBackToTop] = useState(false);
  const containerRef = useRef(null);
  
  const handleScroll = (e) => {
    const { scrollTop, scrollHeight, clientHeight } = e.target;
    const progress = (scrollTop / (scrollHeight - clientHeight)) * 100;
    setScrollPosition(Math.round(progress));
    setShowBackToTop(scrollTop > 200);
  };
  
  const scrollToTop = () => {
    containerRef.current?.scrollTo({ top: 0, behavior: 'smooth' });
  };
  
  return (
    <div>
      {/* Progress bar */}
      <div style={{
        position: 'fixed',
        top: 0,
        left: 0,
        width: `${scrollPosition}%`,
        height: 4,
        backgroundColor: '#4CAF50',
        transition: 'width 0.1s'
      }} />
      
      <div
        ref={containerRef}
        onScroll={handleScroll}
        style={{ height: 400, overflowY: 'auto', border: '1px solid #ccc' }}
      >
        {Array.from({ length: 50 }, (_, i) => (
          <p key={i} style={{ padding: 8, borderBottom: '1px solid #eee' }}>
            บรรทัดที่ {i + 1} - Lorem ipsum dolor sit amet...
          </p>
        ))}
      </div>
      
      <p>เลื่อนแล้ว: {scrollPosition}%</p>
      
      {showBackToTop && (
        <button
          onClick={scrollToTop}
          style={{
            position: 'fixed',
            bottom: 20,
            right: 20,
            padding: '8px 16px',
            backgroundColor: '#4CAF50',
            color: 'white',
            border: 'none',
            borderRadius: 4,
            cursor: 'pointer'
          }}
        >
          ⬆ กลับขึ้นบน
        </button>
      )}
    </div>
  );
}
```

---

## Step 132: Event Object Properties {#step-132}

Properties ที่มีใน Event Object

```jsx
function EventProperties() {
  const handleClick = (e) => {
    console.group("Click Event Properties");
    
    // Target
    console.log("target:", e.target);
    console.log("currentTarget:", e.currentTarget);
    
    // Mouse position
    console.log("clientX:", e.clientX);  // relative to viewport
    console.log("clientY:", e.clientY);
    console.log("pageX:", e.pageX);      // relative to page
    console.log("pageY:", e.pageY);
    console.log("offsetX:", e.nativeEvent.offsetX);  // relative to element
    console.log("offsetY:", e.nativeEvent.offsetY);
    
    // Keyboard modifiers
    console.log("altKey:", e.altKey);
    console.log("ctrlKey:", e.ctrlKey);
    console.log("shiftKey:", e.shiftKey);
    console.log("metaKey:", e.metaKey);
    
    // Mouse button
    console.log("button:", e.button);  // 0=left, 1=middle, 2=right
    console.log("buttons:", e.buttons); // bitmask
    
    console.groupEnd();
  };
  
  const handleKeyDown = (e) => {
    console.group("KeyDown Event Properties");
    console.log("key:", e.key);           // "a", "Enter", "ArrowLeft"
    console.log("code:", e.code);         // "KeyA", "Enter", "ArrowLeft"
    console.log("keyCode:", e.keyCode);   // deprecated แต่ยังใช้ได้
    console.log("repeat:", e.repeat);     // กดค้างหรือไม่
    console.groupEnd();
  };
  
  return (
    <div>
      <button onClick={handleClick}>คลิกเพื่อดู Event Properties</button>
      <input onKeyDown={handleKeyDown} placeholder="กดปุ่มแล้วดู console" />
    </div>
  );
}
```

---

## Step 133: preventDefault {#step-133}

`preventDefault()` ป้องกันพฤติกรรม default ของ browser

```jsx
function PreventDefaultExamples() {
  // ป้องกัน form submit reload page
  const handleFormSubmit = (e) => {
    e.preventDefault();
    console.log("Form submitted without page reload!");
  };
  
  // ป้องกัน link navigation
  const handleLinkClick = (e) => {
    e.preventDefault();
    console.log("Link clicked but not navigated");
  };
  
  // ป้องกัน drag behavior
  const handleDragStart = (e) => {
    e.preventDefault();
    console.log("Drag prevented");
  };
  
  // ป้องกัน right-click menu
  const handleContextMenu = (e) => {
    e.preventDefault();
    console.log("Custom context menu here");
  };
  
  // ป้องกัน paste ใน input
  const handlePaste = (e) => {
    e.preventDefault();
    const pastedText = e.clipboardData.getData('text');
    console.log("Pasted:", pastedText);
    // ทำ custom paste logic ที่นี่
  };
  
  return (
    <div>
      <form onSubmit={handleFormSubmit}>
        <input type="text" />
        <button type="submit">Submit (no reload)</button>
      </form>
      
      <a href="https://google.com" onClick={handleLinkClick}>
        Link (blocked)
      </a>
      
      <div onContextMenu={handleContextMenu} style={{ padding: 20, border: '1px solid #ccc' }}>
        Right-click ที่นี่ (blocked)
      </div>
      
      <input onPaste={handlePaste} placeholder="ลอง paste ที่นี่" />
    </div>
  );
}
```

---

## Step 134: stopPropagation {#step-134}

`stopPropagation()` หยุดการ bubble ของ event

```jsx
function StopPropagationDemo() {
  const [log, setLog] = useState([]);
  
  const addLog = (message) => {
    setLog(prev => [...prev, `${new Date().toLocaleTimeString()}: ${message}`]);
  };
  
  return (
    <div
      onClick={() => addLog("Outer div clicked")}
      style={{ padding: 20, backgroundColor: '#ffeeee' }}
    >
      <h3>Outer Div (ฟัง click)</h3>
      
      <div
        onClick={() => addLog("Middle div clicked")}
        style={{ padding: 20, backgroundColor: '#eeffee' }}
      >
        <h4>Middle Div (ฟัง click)</h4>
        
        {/* ไม่ stop propagation */}
        <button onClick={() => addLog("Button clicked - bubble ขึ้นบน")}>
          ปุ่มปกติ (bubble ขึ้น)
        </button>
        
        {/* Stop propagation */}
        <button
          onClick={(e) => {
            e.stopPropagation(); // หยุด bubble
            addLog("Button clicked - ไม่ bubble ขึ้น");
          }}
        >
          ปุ่ม stopPropagation (ไม่ bubble)
        </button>
      </div>
      
      <div style={{ marginTop: 16, backgroundColor: '#f5f5f5', padding: 8 }}>
        <h4>Log:</h4>
        {log.map((entry, i) => <p key={i}>{entry}</p>)}
        <button onClick={(e) => { e.stopPropagation(); setLog([]); }}>
          ล้าง Log
        </button>
      </div>
    </div>
  );
}
```

---

## Step 135: Event Bubbling {#step-135}

Event Bubbling คือการที่ event ไหลจากล่างขึ้นบนใน DOM tree

```
Document
    └── Window
         └── body
              └── #app
                   └── .container     ← click event จะ bubble ขึ้น
                        └── .card
                             └── button ← user คลิกที่นี่
```

```jsx
function BubblingDemo() {
  const levels = ['Document Level', 'Body Level', 'App Level', 'Container Level', 'Card Level', 'Button Level'];
  const [clickedLevels, setClickedLevels] = useState([]);
  
  const addLevel = (level) => {
    setClickedLevels(prev => [...prev, level]);
    setTimeout(() => setClickedLevels([]), 2000); // clear หลัง 2 วิ
  };
  
  return (
    <div onClick={() => addLevel("App")} style={{ padding: 8, background: '#f0e0ff' }}>
      App Level
      <div onClick={() => addLevel("Container")} style={{ padding: 8, background: '#e0f0ff', margin: 4 }}>
        Container Level
        <div onClick={() => addLevel("Card")} style={{ padding: 8, background: '#e0ffe0', margin: 4 }}>
          Card Level
          <button onClick={() => addLevel("Button")}>
            คลิกปุ่มนี้ (Button Level)
          </button>
        </div>
      </div>
      
      <div style={{ marginTop: 8 }}>
        Event ที่เกิด (ล่างขึ้นบน):
        {clickedLevels.map((level, i) => (
          <span key={i} style={{ 
            display: 'inline-block', 
            margin: '0 4px',
            padding: '2px 8px',
            background: '#ddd',
            borderRadius: 4
          }}>
            {level} →
          </span>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 136: Event Delegation {#step-136}

Event Delegation คือการผูก event handler เดียวกับ parent แทนที่จะผูกกับทุก child

```jsx
// ❌ ผูก handler กับทุก item (ไม่มีประสิทธิภาพ)
function MenuWithoutDelegation({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => handleItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// ✅ Event Delegation - handler เดียวที่ parent
function MenuWithDelegation({ items, onSelect }) {
  const handleClick = (e) => {
    // ใช้ dataset เพื่อส่ง data
    const itemId = e.target.closest('[data-item-id]')?.dataset.itemId;
    if (itemId) {
      onSelect(Number(itemId));
    }
  };
  
  return (
    <ul onClick={handleClick}>
      {items.map(item => (
        <li 
          key={item.id} 
          data-item-id={item.id}
          style={{ cursor: 'pointer', padding: 8 }}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// Dynamic list - ดีมากกับ Event Delegation
function DynamicList() {
  const [items, setItems] = useState([
    { id: 1, name: "Item 1", action: "edit" },
    { id: 2, name: "Item 2", action: "delete" },
  ]);
  
  const handleAction = (e) => {
    const { action, itemId } = e.target.dataset;
    if (!action) return;
    
    switch (action) {
      case 'edit':
        console.log("Edit item:", itemId);
        break;
      case 'delete':
        setItems(prev => prev.filter(item => item.id !== Number(itemId)));
        break;
    }
  };
  
  return (
    <div onClick={handleAction}>
      {items.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          <button data-action="edit" data-item-id={item.id}>แก้ไข</button>
          <button data-action="delete" data-item-id={item.id}>ลบ</button>
        </div>
      ))}
    </div>
  );
}
```

---

## Step 137: Passing Arguments to Event Handlers {#step-137}

วิธีส่ง arguments เพิ่มเติมให้ event handlers

```jsx
function ArgumentsDemo() {
  const handleItemClick = (itemId, itemName, e) => {
    console.log("Item:", itemId, itemName);
    console.log("Event:", e.type);
  };
  
  const items = [
    { id: 1, name: "Apple" },
    { id: 2, name: "Banana" },
    { id: 3, name: "Cherry" },
  ];
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {/* วิธีที่ 1: Arrow function (แนะนำ) */}
          <button onClick={(e) => handleItemClick(item.id, item.name, e)}>
            {item.name} (Arrow)
          </button>
          
          {/* วิธีที่ 2: bind */}
          <button onClick={handleItemClick.bind(null, item.id, item.name)}>
            {item.name} (Bind)
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### Higher-Order Event Handler

```jsx
function HigherOrderHandlers() {
  // สร้าง handler factory
  const createHandler = (id, type) => (e) => {
    console.log(`${type} on item ${id}`, e);
  };
  
  // Handler ที่ return handler
  const handleSort = (field) => () => {
    console.log("Sort by:", field);
  };
  
  return (
    <div>
      {/* Sort buttons */}
      <button onClick={handleSort('name')}>เรียงตามชื่อ</button>
      <button onClick={handleSort('date')}>เรียงตามวันที่</button>
      <button onClick={handleSort('price')}>เรียงตามราคา</button>
      
      {/* Item handlers */}
      {[1, 2, 3].map(id => (
        <div key={id}>
          Item {id}
          <button onClick={createHandler(id, 'edit')}>แก้ไข</button>
          <button onClick={createHandler(id, 'delete')}>ลบ</button>
        </div>
      ))}
    </div>
  );
}
```

> **Tip:** การใช้ arrow function ใน JSX ทุกครั้งจะสร้าง function ใหม่ทุก render ซึ่งอาจส่งผลต่อ performance ในบางกรณี ถ้ามีปัญหาให้ใช้ useCallback

---

## Step 138: Handler ใน Class vs Function {#step-138}

เปรียบเทียบ event handling ใน class component และ function component

```jsx
// Class Component (Legacy)
class ClassCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    // ต้อง bind this ใน constructor
    this.handleIncrement = this.handleIncrement.bind(this);
  }
  
  handleIncrement() {
    this.setState({ count: this.state.count + 1 });
  }
  
  // หรือ arrow function property (ไม่ต้อง bind)
  handleDecrement = () => {
    this.setState({ count: this.state.count - 1 });
  };
  
  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={this.handleIncrement}>+</button>
        <button onClick={this.handleDecrement}>-</button>
      </div>
    );
  }
}

// Function Component (Modern) - ง่ายกว่ามาก!
function FunctionCounter() {
  const [count, setCount] = useState(0);
  
  // ไม่ต้อง bind, ไม่ต้อง this
  const handleIncrement = () => setCount(c => c + 1);
  const handleDecrement = () => setCount(c => c - 1);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleDecrement}>-</button>
    </div>
  );
}
```

---

## Step 139: Form Events - Input {#step-139}

```jsx
function InputFieldExamples() {
  const [values, setValues] = useState({
    text: '',
    number: '',
    email: '',
    password: '',
    search: '',
    url: '',
    tel: ''
  });
  
  const handleChange = (e) => {
    setValues(prev => ({ ...prev, [e.target.name]: e.target.value }));
  };
  
  return (
    <div>
      {/* Text Input */}
      <div>
        <label>ข้อความ:</label>
        <input
          type="text"
          name="text"
          value={values.text}
          onChange={handleChange}
          placeholder="พิมพ์ข้อความ"
          maxLength={100}
        />
        <small>{values.text.length}/100</small>
      </div>
      
      {/* Email Input */}
      <div>
        <label>อีเมล:</label>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          placeholder="example@email.com"
        />
      </div>
      
      {/* Password Input */}
      <div>
        <label>รหัสผ่าน:</label>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
        />
        <small>ความยาว: {values.password.length}</small>
      </div>
      
      {/* Search Input */}
      <div>
        <label>ค้นหา:</label>
        <input
          type="search"
          name="search"
          value={values.search}
          onChange={handleChange}
          onSearch={(e) => console.log("Search:", e.target.value)}
        />
      </div>
      
      {/* Number Input */}
      <div>
        <label>ตัวเลข:</label>
        <input
          type="number"
          name="number"
          value={values.number}
          onChange={handleChange}
          min={0}
          max={100}
          step={5}
        />
      </div>
    </div>
  );
}
```

---

## Step 140: Form Events - Select {#step-140}

```jsx
function SelectExamples() {
  const [selected, setSelected] = useState('');
  const [multiSelected, setMultiSelected] = useState([]);
  
  const provinces = [
    { value: 'bkk', label: 'กรุงเทพมหานคร' },
    { value: 'cnx', label: 'เชียงใหม่' },
    { value: 'pkt', label: 'ภูเก็ต' },
    { value: 'kbi', label: 'กระบี่' },
    { value: 'cmi', label: 'เชียงราย' },
  ];
  
  const handleMultiChange = (e) => {
    const options = [...e.target.options];
    const selected = options
      .filter(opt => opt.selected)
      .map(opt => opt.value);
    setMultiSelected(selected);
  };
  
  return (
    <div>
      {/* Single select */}
      <div>
        <label>จังหวัด:</label>
        <select 
          value={selected} 
          onChange={(e) => setSelected(e.target.value)}
        >
          <option value="">-- เลือกจังหวัด --</option>
          {provinces.map(province => (
            <option key={province.value} value={province.value}>
              {province.label}
            </option>
          ))}
        </select>
        <p>เลือก: {selected || 'ยังไม่ได้เลือก'}</p>
      </div>
      
      {/* Multi select */}
      <div>
        <label>จังหวัดที่ชอบ (เลือกได้หลายอัน):</label>
        <select
          multiple
          value={multiSelected}
          onChange={handleMultiChange}
          size={5}
          style={{ width: 200 }}
        >
          {provinces.map(province => (
            <option key={province.value} value={province.value}>
              {province.label}
            </option>
          ))}
        </select>
        <p>เลือก: {multiSelected.join(', ') || 'ยังไม่ได้เลือก'}</p>
      </div>
      
      {/* Select with groups */}
      <div>
        <label>เมนูอาหาร:</label>
        <select defaultValue="">
          <option value="">-- เลือกเมนู --</option>
          <optgroup label="อาหารไทย">
            <option value="padthai">ผัดไทย</option>
            <option value="somtam">ส้มตำ</option>
          </optgroup>
          <optgroup label="อาหารจีน">
            <option value="khaopadkrapow">ข้าวหน้าเป็ด</option>
            <option value="dimsum">ติ่มซำ</option>
          </optgroup>
        </select>
      </div>
    </div>
  );
}
```

---

## Step 141: Form Events - Checkbox {#step-141}

```jsx
function CheckboxExamples() {
  const [isAgreed, setIsAgreed] = useState(false);
  const [interests, setInterests] = useState({
    react: false,
    vue: false,
    angular: false,
    svelte: false
  });
  
  const handleInterestChange = (e) => {
    const { name, checked } = e.target;
    setInterests(prev => ({ ...prev, [name]: checked }));
  };
  
  const selectedInterests = Object.entries(interests)
    .filter(([, checked]) => checked)
    .map(([name]) => name);
  
  const handleSelectAll = (e) => {
    const checked = e.target.checked;
    setInterests(prev => 
      Object.fromEntries(Object.keys(prev).map(key => [key, checked]))
    );
  };
  
  const allChecked = Object.values(interests).every(Boolean);
  const someChecked = Object.values(interests).some(Boolean);
  
  return (
    <div>
      {/* Single Checkbox */}
      <label>
        <input
          type="checkbox"
          checked={isAgreed}
          onChange={(e) => setIsAgreed(e.target.checked)}
        />
        ฉันยอมรับเงื่อนไขการใช้บริการ
      </label>
      
      {/* Checkbox Group */}
      <div>
        <h4>เลือก Framework ที่สนใจ:</h4>
        
        {/* Select All */}
        <label>
          <input
            type="checkbox"
            checked={allChecked}
            ref={ref => {
              if (ref) ref.indeterminate = someChecked && !allChecked;
            }}
            onChange={handleSelectAll}
          />
          <strong>เลือกทั้งหมด</strong>
        </label>
        
        <div style={{ paddingLeft: 20 }}>
          {Object.entries(interests).map(([name, checked]) => (
            <label key={name} style={{ display: 'block' }}>
              <input
                type="checkbox"
                name={name}
                checked={checked}
                onChange={handleInterestChange}
              />
              {name.charAt(0).toUpperCase() + name.slice(1)}
            </label>
          ))}
        </div>
        
        <p>เลือก: {selectedInterests.join(', ') || 'ยังไม่ได้เลือก'}</p>
      </div>
    </div>
  );
}
```

---

## Step 142: Form Events - Radio {#step-142}

```jsx
function RadioExamples() {
  const [gender, setGender] = useState('');
  const [plan, setPlan] = useState('basic');
  
  const plans = [
    { value: 'free', label: 'Free', price: '฿0/เดือน', features: ['5 projects', '2GB storage'] },
    { value: 'basic', label: 'Basic', price: '฿199/เดือน', features: ['20 projects', '10GB storage'] },
    { value: 'pro', label: 'Pro', price: '฿599/เดือน', features: ['Unlimited projects', '100GB storage'] },
  ];
  
  return (
    <div>
      {/* Basic Radio Group */}
      <div>
        <h4>เพศ:</h4>
        {['male', 'female', 'other'].map(value => (
          <label key={value} style={{ marginRight: 16 }}>
            <input
              type="radio"
              name="gender"
              value={value}
              checked={gender === value}
              onChange={(e) => setGender(e.target.value)}
            />
            {value === 'male' ? 'ชาย' : value === 'female' ? 'หญิง' : 'อื่นๆ'}
          </label>
        ))}
        <p>เลือก: {gender || 'ยังไม่ได้เลือก'}</p>
      </div>
      
      {/* Card-style Radio Group */}
      <div>
        <h4>เลือกแผน:</h4>
        <div style={{ display: 'flex', gap: 16 }}>
          {plans.map(planOption => (
            <label
              key={planOption.value}
              style={{
                border: `2px solid ${plan === planOption.value ? '#4CAF50' : '#ddd'}`,
                borderRadius: 8,
                padding: 16,
                cursor: 'pointer',
                backgroundColor: plan === planOption.value ? '#f0fff0' : 'white',
                minWidth: 150
              }}
            >
              <input
                type="radio"
                name="plan"
                value={planOption.value}
                checked={plan === planOption.value}
                onChange={(e) => setPlan(e.target.value)}
                style={{ marginBottom: 8 }}
              />
              <div>
                <strong>{planOption.label}</strong>
                <p style={{ color: 'green', margin: '4px 0' }}>{planOption.price}</p>
                <ul style={{ fontSize: 12, padding: '0 0 0 16px', margin: 0 }}>
                  {planOption.features.map((f, i) => (
                    <li key={i}>{f}</li>
                  ))}
                </ul>
              </div>
            </label>
          ))}
        </div>
      </div>
    </div>
  );
}
```

---

## Step 143: Form Events - File Upload {#step-143}

```jsx
function FileUploadExample() {
  const [files, setFiles] = useState([]);
  const [previews, setPreviews] = useState([]);
  const [isDragging, setIsDragging] = useState(false);
  
  const handleFileChange = (e) => {
    processFiles(Array.from(e.target.files));
  };
  
  const processFiles = (fileList) => {
    const validFiles = fileList.filter(file => {
      const isValidType = file.type.startsWith('image/');
      const isValidSize = file.size <= 5 * 1024 * 1024; // 5MB
      return isValidType && isValidSize;
    });
    
    setFiles(validFiles);
    
    // สร้าง preview URLs
    const previewUrls = validFiles.map(file => ({
      url: URL.createObjectURL(file),
      name: file.name,
      size: file.size
    }));
    
    // ล้าง URLs เก่า
    previews.forEach(p => URL.revokeObjectURL(p.url));
    setPreviews(previewUrls);
  };
  
  const handleDrop = (e) => {
    e.preventDefault();
    setIsDragging(false);
    processFiles(Array.from(e.dataTransfer.files));
  };
  
  const handleDragOver = (e) => {
    e.preventDefault();
    setIsDragging(true);
  };
  
  const handleDragLeave = () => setIsDragging(false);
  
  const removeFile = (index) => {
    URL.revokeObjectURL(previews[index].url);
    setFiles(prev => prev.filter((_, i) => i !== index));
    setPreviews(prev => prev.filter((_, i) => i !== index));
  };
  
  return (
    <div>
      {/* Drag & Drop Zone */}
      <div
        onDrop={handleDrop}
        onDragOver={handleDragOver}
        onDragLeave={handleDragLeave}
        style={{
          border: `2px dashed ${isDragging ? '#4CAF50' : '#ccc'}`,
          borderRadius: 8,
          padding: 40,
          textAlign: 'center',
          backgroundColor: isDragging ? '#f0fff0' : '#fafafa',
          cursor: 'pointer',
          transition: 'all 0.2s'
        }}
        onClick={() => document.getElementById('fileInput').click()}
      >
        <p>🖼️ ลาก & วางรูปภาพที่นี่</p>
        <p>หรือคลิกเพื่อเลือกไฟล์</p>
        <p style={{ fontSize: 12, color: '#888' }}>รองรับ JPG, PNG, GIF (สูงสุด 5MB)</p>
      </div>
      
      <input
        id="fileInput"
        type="file"
        multiple
        accept="image/*"
        onChange={handleFileChange}
        style={{ display: 'none' }}
      />
      
      {/* Preview */}
      {previews.length > 0 && (
        <div style={{ display: 'flex', flexWrap: 'wrap', gap: 8, marginTop: 16 }}>
          {previews.map((preview, index) => (
            <div key={index} style={{ position: 'relative' }}>
              <img
                src={preview.url}
                alt={preview.name}
                style={{ width: 100, height: 100, objectFit: 'cover', borderRadius: 4 }}
              />
              <button
                onClick={() => removeFile(index)}
                style={{
                  position: 'absolute',
                  top: -8,
                  right: -8,
                  background: 'red',
                  color: 'white',
                  border: 'none',
                  borderRadius: '50%',
                  width: 20,
                  height: 20,
                  cursor: 'pointer',
                  fontSize: 12
                }}
              >
                ✕
              </button>
              <p style={{ fontSize: 10, textAlign: 'center', margin: '4px 0 0' }}>
                {(preview.size / 1024).toFixed(1)}KB
              </p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Step 144: Complex Form ตัวอย่าง {#step-144}

ตัวอย่าง form สมบูรณ์ที่รวมทุก event types

```jsx
function RegistrationForm() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    // Step 1
    firstName: '', lastName: '', email: '', phone: '',
    // Step 2
    username: '', password: '', confirmPassword: '',
    // Step 3
    plan: 'basic', interests: [], newsletter: false, agree: false
  });
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    if (type === 'checkbox' && name === 'interests') {
      setFormData(prev => ({
        ...prev,
        interests: checked
          ? [...prev.interests, value]
          : prev.interests.filter(i => i !== value)
      }));
    } else {
      setFormData(prev => ({
        ...prev,
        [name]: type === 'checkbox' ? checked : value
      }));
    }
    
    // ล้าง error
    setErrors(prev => ({ ...prev, [name]: '' }));
  };
  
  const validateStep = (stepNum) => {
    const newErrors = {};
    
    if (stepNum === 1) {
      if (!formData.firstName.trim()) newErrors.firstName = 'กรุณากรอกชื่อ';
      if (!formData.lastName.trim()) newErrors.lastName = 'กรุณากรอกนามสกุล';
      if (!formData.email.match(/\S+@\S+\.\S+/)) newErrors.email = 'อีเมลไม่ถูกต้อง';
    }
    
    if (stepNum === 2) {
      if (formData.username.length < 3) newErrors.username = 'ชื่อผู้ใช้ต้องมีอย่างน้อย 3 ตัวอักษร';
      if (formData.password.length < 8) newErrors.password = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
      if (formData.password !== formData.confirmPassword) newErrors.confirmPassword = 'รหัสผ่านไม่ตรงกัน';
    }
    
    if (stepNum === 3) {
      if (!formData.agree) newErrors.agree = 'กรุณายอมรับเงื่อนไข';
    }
    
    return newErrors;
  };
  
  const handleNext = () => {
    const stepErrors = validateStep(step);
    if (Object.keys(stepErrors).length > 0) {
      setErrors(stepErrors);
      return;
    }
    setStep(prev => prev + 1);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const stepErrors = validateStep(3);
    if (Object.keys(stepErrors).length > 0) {
      setErrors(stepErrors);
      return;
    }
    console.log("Submitted:", formData);
    alert('ลงทะเบียนสำเร็จ!');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Progress */}
      <div style={{ display: 'flex', marginBottom: 24 }}>
        {[1, 2, 3].map(s => (
          <div
            key={s}
            style={{
              flex: 1,
              height: 4,
              backgroundColor: s <= step ? '#4CAF50' : '#ddd',
              marginRight: s < 3 ? 4 : 0
            }}
          />
        ))}
      </div>
      <p>ขั้นตอนที่ {step} จาก 3</p>
      
      {step === 1 && (
        <div>
          <h3>ข้อมูลส่วนตัว</h3>
          <input name="firstName" value={formData.firstName} onChange={handleChange} placeholder="ชื่อ" />
          {errors.firstName && <span style={{color:'red'}}>{errors.firstName}</span>}
          <input name="lastName" value={formData.lastName} onChange={handleChange} placeholder="นามสกุล" />
          {errors.lastName && <span style={{color:'red'}}>{errors.lastName}</span>}
          <input type="email" name="email" value={formData.email} onChange={handleChange} placeholder="อีเมล" />
          {errors.email && <span style={{color:'red'}}>{errors.email}</span>}
          <input name="phone" value={formData.phone} onChange={handleChange} placeholder="เบอร์โทร" />
        </div>
      )}
      
      {step === 2 && (
        <div>
          <h3>ข้อมูลบัญชี</h3>
          <input name="username" value={formData.username} onChange={handleChange} placeholder="ชื่อผู้ใช้" />
          {errors.username && <span style={{color:'red'}}>{errors.username}</span>}
          <input type="password" name="password" value={formData.password} onChange={handleChange} placeholder="รหัสผ่าน" />
          {errors.password && <span style={{color:'red'}}>{errors.password}</span>}
          <input type="password" name="confirmPassword" value={formData.confirmPassword} onChange={handleChange} placeholder="ยืนยันรหัสผ่าน" />
          {errors.confirmPassword && <span style={{color:'red'}}>{errors.confirmPassword}</span>}
        </div>
      )}
      
      {step === 3 && (
        <div>
          <h3>ตั้งค่าบัญชี</h3>
          
          <div>
            <p>แผนการใช้งาน:</p>
            {['free', 'basic', 'pro'].map(p => (
              <label key={p}>
                <input type="radio" name="plan" value={p} checked={formData.plan === p} onChange={handleChange} />
                {p}
              </label>
            ))}
          </div>
          
          <div>
            <p>ความสนใจ:</p>
            {['React', 'Vue', 'Angular', 'Node.js'].map(interest => (
              <label key={interest}>
                <input
                  type="checkbox"
                  name="interests"
                  value={interest}
                  checked={formData.interests.includes(interest)}
                  onChange={handleChange}
                />
                {interest}
              </label>
            ))}
          </div>
          
          <label>
            <input type="checkbox" name="newsletter" checked={formData.newsletter} onChange={handleChange} />
            รับข่าวสาร newsletter
          </label>
          
          <label>
            <input type="checkbox" name="agree" checked={formData.agree} onChange={handleChange} />
            ฉันยอมรับเงื่อนไขการใช้บริการ
          </label>
          {errors.agree && <span style={{color:'red'}}>{errors.agree}</span>}
        </div>
      )}
      
      <div style={{ marginTop: 16, display: 'flex', gap: 8 }}>
        {step > 1 && (
          <button type="button" onClick={() => setStep(prev => prev - 1)}>
            ← ย้อนกลับ
          </button>
        )}
        {step < 3 ? (
          <button type="button" onClick={handleNext}>
            ถัดไป →
          </button>
        ) : (
          <button type="submit">
            ✅ ลงทะเบียน
          </button>
        )}
      </div>
    </form>
  );
}
```

---

## Step 145: Custom Event Patterns {#step-145}

### Custom Event with useReducer

```jsx
function useEventEmitter() {
  const [listeners] = useState(new Map());
  
  const on = (event, callback) => {
    if (!listeners.has(event)) {
      listeners.set(event, []);
    }
    listeners.get(event).push(callback);
    
    return () => {
      const callbacks = listeners.get(event) || [];
      listeners.set(event, callbacks.filter(cb => cb !== callback));
    };
  };
  
  const emit = (event, data) => {
    const callbacks = listeners.get(event) || [];
    callbacks.forEach(cb => cb(data));
  };
  
  return { on, emit };
}

// Composite Event Pattern
function CompositeForm({ onValidate, onDirty, onSubmit }) {
  const [value, setValue] = useState('');
  const [isDirty, setIsDirty] = useState(false);
  
  const handleChange = (e) => {
    const newValue = e.target.value;
    setValue(newValue);
    
    if (!isDirty) {
      setIsDirty(true);
      onDirty?.();
    }
    
    onValidate?.(newValue);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit?.(value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}

// Keyboard Shortcut Hook
function useKeyboardShortcut(key, callback, options = {}) {
  const { ctrlKey, shiftKey, altKey, enabled = true } = options;
  
  useEffect(() => {
    if (!enabled) return;
    
    const handler = (e) => {
      if (
        e.key === key &&
        e.ctrlKey === !!ctrlKey &&
        e.shiftKey === !!shiftKey &&
        e.altKey === !!altKey
      ) {
        e.preventDefault();
        callback(e);
      }
    };
    
    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [key, callback, ctrlKey, shiftKey, altKey, enabled]);
}

// การใช้งาน
function AppWithShortcuts() {
  const [isSaved, setIsSaved] = useState(false);
  
  useKeyboardShortcut('s', () => {
    setIsSaved(true);
    setTimeout(() => setIsSaved(false), 2000);
  }, { ctrlKey: true });
  
  useKeyboardShortcut('z', () => {
    console.log("Undo!");
  }, { ctrlKey: true });
  
  return (
    <div>
      <p>Keyboard Shortcuts:</p>
      <p>Ctrl+S = บันทึก</p>
      <p>Ctrl+Z = Undo</p>
      {isSaved && <p style={{ color: 'green' }}>✅ บันทึกแล้ว!</p>}
    </div>
  );
}
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม

1. Synthetic Event คืออะไร ต่างจาก native DOM event อย่างไร?
2. `preventDefault()` และ `stopPropagation()` ต่างกันอย่างไร?
3. Event Bubbling คืออะไร? มีประโยชน์อย่างไร?
4. เหตุใดการส่ง handler เป็น `onClick={handleClick}` ถึงดีกว่า `onClick={handleClick()}`?
5. `onChange` ใน React ทำงานอย่างไรกับ HTML `oninput`?

### แบบฝึกหัด

**Exercise 1:** สร้าง `ClickOutside` hook ที่:
- รับ ref และ callback
- เรียก callback เมื่อ click นอก element
- ใช้สำหรับปิด dropdown หรือ modal

**Exercise 2:** สร้าง `DragDropList`:
- รายการที่ drag & drop เพื่อเรียงลำดับได้
- แสดง visual feedback ขณะ drag
- อัพเดท order เมื่อ drop

**Exercise 3:** สร้าง Keyboard-navigable `Dropdown`:
- Arrow Up/Down เพื่อเลือก option
- Enter เพื่อยืนยัน
- Escape เพื่อปิด
- Type-ahead search

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- Synthetic Events และความแตกต่างจาก DOM events
- Event types ทั้งหมด: click, change, submit, keyboard, mouse, focus, scroll
- Event Object properties
- preventDefault และ stopPropagation
- Event Bubbling และ Event Delegation
- การส่ง arguments ให้ event handlers
- Form events ทุกประเภท: input, select, checkbox, radio, file
- Complex form example

---

## อ่านต่อ

➡️ **[Part 08: Conditional Rendering](./part-08-conditional-rendering.md)**

เราจะเรียนรู้วิธีแสดงผล UI แบบมีเงื่อนไข ด้วย if/else, ternary, &&, ?? และ patterns ต่างๆ

---

*React Course - Part 07 | สร้างโดย React Course Thailand*
