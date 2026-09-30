# Part 06: State และ useState Hook
## Step 96-120 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 96: State คืออะไร?](#step-96)
- [Step 97: State vs Props](#step-97)
- [Step 98: useState Hook พื้นฐาน](#step-98)
- [Step 99: Rules of Hooks](#step-99)
- [Step 100: State ประเภท String](#step-100)
- [Step 101: State ประเภท Number](#step-101)
- [Step 102: State ประเภท Boolean](#step-102)
- [Step 103: State ประเภท Array](#step-103)
- [Step 104: State ประเภท Object](#step-104)
- [Step 105: Functional Updates](#step-105)
- [Step 106: Immutability ใน State Updates](#step-106)
- [Step 107: Multiple State Variables](#step-107)
- [Step 108: Lazy Initialization](#step-108)
- [Step 109: Counter Example](#step-109)
- [Step 110: Toggle Example](#step-110)
- [Step 111: Form State Example](#step-111)
- [Step 112: Array State - เพิ่มรายการ](#step-112)
- [Step 113: Array State - ลบรายการ](#step-113)
- [Step 114: Array State - แก้ไขรายการ](#step-114)
- [Step 115: Object State Operations](#step-115)
- [Step 116: State Batching](#step-116)
- [Step 117: State ที่ derived จาก State อื่น](#step-117)
- [Step 118: State Reset Patterns](#step-118)
- [Step 119: Common State Mistakes](#step-119)
- [Step 120: State Management Best Practices](#step-120)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 96: State คืออะไร? {#step-96}

**State** คือข้อมูลที่ component "จำ" ไว้ระหว่าง renders และเมื่อ state เปลี่ยน React จะ re-render component นั้นโดยอัตโนมัติ

### ทำไมต้องมี State?

ใน React, component เป็นฟังก์ชัน ปัญหาคือ local variables ใน function จะหายไปทุกครั้งที่ function ถูกเรียก

```jsx
// ❌ ปัญหา - variable ธรรมดาไม่ทำให้ re-render
function Counter() {
  let count = 0;  // จะกลับเป็น 0 ทุกครั้งที่ render!
  
  const handleClick = () => {
    count++;  // เปลี่ยนค่าได้ แต่ UI ไม่อัพเดท
    console.log(count); // แสดง count แต่ UI ยังเป็น 0
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

```jsx
// ✅ ถูก - State ทำให้ React รู้ว่าต้อง re-render
function Counter() {
  const [count, setCount] = React.useState(0);
  
  const handleClick = () => {
    setCount(count + 1);  // แจ้ง React ว่าต้อง re-render
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

### State ทำสองอย่าง

1. **เก็บข้อมูล** ระหว่าง renders
2. **Trigger re-render** เมื่อข้อมูลเปลี่ยน

---

## Step 97: State vs Props {#step-97}

| Feature | State | Props |
|---------|-------|-------|
| เจ้าของ | Component นั้นเอง | Parent Component |
| แก้ไขได้ | ✅ (ผ่าน setState) | ❌ Read-only |
| เปลี่ยนเมื่อ | Component เรียก setState | Parent render ใหม่ |
| Initial Value | กำหนดใน useState | มาจาก parent |
| ใช้สำหรับ | UI interactions, data ที่เปลี่ยน | Configuration, data จาก parent |

```jsx
function TemperatureDisplay({ 
  city,       // prop - ได้จาก parent, ไม่เปลี่ยน
  unit        // prop - ได้จาก parent
}) {
  // state - เปลี่ยนได้จากภายใน component นี้
  const [temperature, setTemperature] = React.useState(25);
  const [isLoading, setIsLoading] = React.useState(false);
  
  const fetchTemperature = async () => {
    setIsLoading(true);
    const temp = await getTemperature(city);
    setTemperature(temp);
    setIsLoading(false);
  };
  
  return (
    <div>
      <h2>{city}</h2>  {/* props */}
      {isLoading ? (   {/* state */}
        <p>กำลังโหลด...</p>
      ) : (
        <p>{temperature}°{unit}</p>  {/* state + props */}
      )}
      <button onClick={fetchTemperature}>รีเฟรช</button>
    </div>
  );
}
```

---

## Step 98: useState Hook พื้นฐาน {#step-98}

`useState` คือ React Hook ที่ใช้เพิ่ม state เข้าไปใน functional component

### Syntax

```jsx
const [state, setState] = useState(initialValue);
// state    = ค่าปัจจุบัน
// setState = function สำหรับอัพเดท state
// initialValue = ค่าเริ่มต้น (ใช้แค่ครั้งแรก)
```

### ตัวอย่างพื้นฐาน

```jsx
import React, { useState } from 'react';

function SimpleCounter() {
  // Declare state variable
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h2>Count: {count}</h2>
      
      {/* อัพเดท state */}
      <button onClick={() => setCount(count + 1)}>
        เพิ่ม
      </button>
      <button onClick={() => setCount(count - 1)}>
        ลด
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}
```

### การ Destructure ใน useState

```jsx
// useState คืน Array [currentValue, updaterFunction]
const result = useState(0);
const count = result[0];      // ค่าปัจจุบัน
const setCount = result[1];   // function สำหรับอัพเดท

// แต่ทั่วไปใช้ destructuring ด้านล่าง
const [count, setCount] = useState(0);
```

> **Tip:** ตั้งชื่อ state และ setter ให้เข้าคู่กัน: `[count, setCount]`, `[name, setName]`, `[isOpen, setIsOpen]`

---

## Step 99: Rules of Hooks {#step-99}

React Hooks มีกฎที่ต้องปฏิบัติตามเสมอ

### กฎที่ 1: เรียก Hooks ที่ Top Level เท่านั้น

```jsx
// ❌ ผิด - Hook ใน condition
function Component({ isEnabled }) {
  if (isEnabled) {
    const [data, setData] = useState(null); // ERROR!
  }
  // ...
}

// ❌ ผิด - Hook ใน loop
function Component({ items }) {
  items.forEach(item => {
    const [selected, setSelected] = useState(false); // ERROR!
  });
}

// ✅ ถูก - Hooks ที่ top level เสมอ
function Component({ isEnabled }) {
  const [data, setData] = useState(null);
  
  if (!isEnabled) return null; // condition หลัง hooks
  
  return <div>{data}</div>;
}
```

### กฎที่ 2: เรียก Hooks ใน React Functions เท่านั้น

```jsx
// ❌ ผิด - Hook ใน regular function
function regularFunction() {
  const [count, setCount] = useState(0); // ERROR!
}

// ✅ ถูก - Hook ใน React Component
function ReactComponent() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// ✅ ถูก - Hook ใน Custom Hook
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, setCount };
}
```

> **Warning:** ถ้าฝ่าฝืนกฎ React จะ throw error และ state จะ corrupt ข้ามหลาย renders

---

## Step 100: State ประเภท String {#step-100}

```jsx
function NameInput() {
  const [name, setName] = useState('');
  const [greeting, setGreeting] = useState('สวัสดี');
  
  const handleSubmit = () => {
    if (name.trim()) {
      setGreeting(`สวัสดี, ${name}!`);
    }
  };
  
  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="กรอกชื่อของคุณ"
      />
      <button onClick={handleSubmit}>ทักทาย</button>
      <p>{greeting}</p>
      <p>ตัวอักษร: {name.length}</p>
    </div>
  );
}
```

### String State Operations

```jsx
function TextEditor() {
  const [text, setText] = useState('');
  
  return (
    <div>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        rows={5}
      />
      
      <div className="stats">
        <p>ตัวอักษร: {text.length}</p>
        <p>คำ: {text.trim() ? text.trim().split(/\s+/).length : 0}</p>
        <p>บรรทัด: {text.split('\n').length}</p>
      </div>
      
      <div className="actions">
        <button onClick={() => setText(text.toUpperCase())}>
          UPPERCASE
        </button>
        <button onClick={() => setText(text.toLowerCase())}>
          lowercase
        </button>
        <button onClick={() => setText(text.trim())}>
          Trim
        </button>
        <button onClick={() => setText('')}>
          ล้าง
        </button>
      </div>
    </div>
  );
}
```

---

## Step 101: State ประเภท Number {#step-101}

```jsx
function Calculator() {
  const [value, setValue] = useState(0);
  const [history, setHistory] = useState([]);
  
  const operate = (operation) => {
    let newValue;
    switch(operation) {
      case 'double': newValue = value * 2; break;
      case 'half': newValue = value / 2; break;
      case 'square': newValue = value ** 2; break;
      case 'sqrt': newValue = Math.sqrt(value); break;
      case 'negate': newValue = -value; break;
      default: return;
    }
    
    setHistory(prev => [...prev, `${value} → ${newValue} (${operation})`]);
    setValue(newValue);
  };
  
  return (
    <div>
      <div className="display">
        <span>{value}</span>
      </div>
      
      <div className="buttons">
        <input 
          type="number"
          value={value}
          onChange={e => setValue(Number(e.target.value))}
        />
        <button onClick={() => operate('double')}>x2</button>
        <button onClick={() => operate('half')}>÷2</button>
        <button onClick={() => operate('square')}>x²</button>
        <button onClick={() => operate('sqrt')}>√</button>
        <button onClick={() => operate('negate')}>+/-</button>
        <button onClick={() => setValue(0)}>Reset</button>
      </div>
      
      <div className="history">
        <h3>ประวัติ:</h3>
        {history.map((item, i) => <p key={i}>{item}</p>)}
      </div>
    </div>
  );
}
```

---

## Step 102: State ประเภท Boolean {#step-102}

```jsx
function ToggleExample() {
  const [isVisible, setIsVisible] = useState(true);
  const [isDarkMode, setIsDarkMode] = useState(false);
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  
  return (
    <div style={{ background: isDarkMode ? '#333' : '#fff', color: isDarkMode ? '#fff' : '#333' }}>
      {/* Toggle visibility */}
      <button onClick={() => setIsVisible(!isVisible)}>
        {isVisible ? 'ซ่อน' : 'แสดง'} Panel
      </button>
      
      {/* Toggle dark mode */}
      <button onClick={() => setIsDarkMode(prev => !prev)}>
        {isDarkMode ? '☀️ Light' : '🌙 Dark'} Mode
      </button>
      
      {/* Toggle menu */}
      <button onClick={() => setIsMenuOpen(prev => !prev)}>
        {isMenuOpen ? '✕ ปิด' : '☰ เปิด'} Menu
      </button>
      
      {/* Conditional rendering based on boolean state */}
      {isVisible && (
        <div className="panel">
          <p>Panel content ที่ซ่อน/แสดงได้</p>
        </div>
      )}
      
      {isMenuOpen && (
        <nav className="menu">
          <a href="/">หน้าหลัก</a>
          <a href="/about">เกี่ยวกับ</a>
          <a href="/contact">ติดต่อ</a>
        </nav>
      )}
    </div>
  );
}
```

### Boolean State สำหรับ Loading

```jsx
function DataLoader() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [hasError, setHasError] = useState(false);
  
  const loadData = async () => {
    setIsLoading(true);
    setHasError(false);
    
    try {
      const response = await fetch('/api/data');
      const result = await response.json();
      setData(result);
    } catch (error) {
      setHasError(true);
    } finally {
      setIsLoading(false);
    }
  };
  
  if (isLoading) return <div>⏳ กำลังโหลด...</div>;
  if (hasError) return <div>❌ เกิดข้อผิดพลาด</div>;
  
  return (
    <div>
      {data ? <p>{JSON.stringify(data)}</p> : <p>ยังไม่มีข้อมูล</p>}
      <button onClick={loadData}>โหลดข้อมูล</button>
    </div>
  );
}
```

---

## Step 103: State ประเภท Array {#step-103}

เมื่อ state เป็น array **ต้องสร้าง array ใหม่** ทุกครั้งที่อัพเดท (ห้าม mutate)

### เพิ่มรายการ

```jsx
function FruitList() {
  const [fruits, setFruits] = useState(['แอปเปิ้ล', 'กล้วย']);
  const [newFruit, setNewFruit] = useState('');
  
  const addFruit = () => {
    if (!newFruit.trim()) return;
    
    // ✅ สร้าง array ใหม่
    setFruits(prev => [...prev, newFruit.trim()]);
    setNewFruit('');
  };
  
  const addAtStart = (item) => {
    setFruits(prev => [item, ...prev]);
  };
  
  const addAtIndex = (item, index) => {
    setFruits(prev => [
      ...prev.slice(0, index),
      item,
      ...prev.slice(index)
    ]);
  };
  
  return (
    <div>
      <ul>
        {fruits.map((fruit, i) => <li key={i}>{fruit}</li>)}
      </ul>
      <input value={newFruit} onChange={e => setNewFruit(e.target.value)} />
      <button onClick={addFruit}>เพิ่ม</button>
    </div>
  );
}
```

### ลบรายการ

```jsx
function RemovableList() {
  const [items, setItems] = useState([
    { id: 1, text: "รายการ 1" },
    { id: 2, text: "รายการ 2" },
    { id: 3, text: "รายการ 3" },
  ]);
  
  const removeItem = (id) => {
    // ✅ filter คืน array ใหม่
    setItems(prev => prev.filter(item => item.id !== id));
  };
  
  const removeFirst = () => {
    setItems(prev => prev.slice(1));
  };
  
  const removeLast = () => {
    setItems(prev => prev.slice(0, -1));
  };
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.text}
          <button onClick={() => removeItem(item.id)}>ลบ</button>
        </li>
      ))}
      <button onClick={removeFirst}>ลบตัวแรก</button>
      <button onClick={removeLast}>ลบตัวสุดท้าย</button>
    </ul>
  );
}
```

### แก้ไขรายการ

```jsx
function EditableList() {
  const [tasks, setTasks] = useState([
    { id: 1, text: "เรียน React", done: false },
    { id: 2, text: "ทำโปรเจค", done: false },
  ]);
  
  const toggleTask = (id) => {
    // ✅ map คืน array ใหม่
    setTasks(prev => prev.map(task => 
      task.id === id 
        ? { ...task, done: !task.done }
        : task
    ));
  };
  
  const updateText = (id, newText) => {
    setTasks(prev => prev.map(task =>
      task.id === id
        ? { ...task, text: newText }
        : task
    ));
  };
  
  return (
    <ul>
      {tasks.map(task => (
        <li 
          key={task.id}
          style={{ textDecoration: task.done ? 'line-through' : 'none' }}
        >
          <input
            type="checkbox"
            checked={task.done}
            onChange={() => toggleTask(task.id)}
          />
          {task.text}
        </li>
      ))}
    </ul>
  );
}
```

---

## Step 104: State ประเภท Object {#step-104}

Object state ต้องใช้ spread operator เพื่อสร้าง object ใหม่

```jsx
function UserForm() {
  const [user, setUser] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: 0,
    address: {
      city: '',
      country: 'ไทย'
    }
  });
  
  // อัพเดท field เดียว
  const updateField = (field, value) => {
    setUser(prev => ({
      ...prev,           // คัดลอก fields อื่นๆ
      [field]: value     // อัพเดท field ที่ต้องการ
    }));
  };
  
  // อัพเดท nested object
  const updateAddress = (field, value) => {
    setUser(prev => ({
      ...prev,
      address: {
        ...prev.address,  // คัดลอก address fields อื่นๆ
        [field]: value    // อัพเดท field ที่ต้องการ
      }
    }));
  };
  
  return (
    <form>
      <input
        placeholder="ชื่อ"
        value={user.firstName}
        onChange={e => updateField('firstName', e.target.value)}
      />
      <input
        placeholder="นามสกุล"
        value={user.lastName}
        onChange={e => updateField('lastName', e.target.value)}
      />
      <input
        placeholder="อีเมล"
        value={user.email}
        onChange={e => updateField('email', e.target.value)}
      />
      <input
        type="number"
        placeholder="อายุ"
        value={user.age}
        onChange={e => updateField('age', Number(e.target.value))}
      />
      <input
        placeholder="เมือง"
        value={user.address.city}
        onChange={e => updateAddress('city', e.target.value)}
      />
      
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </form>
  );
}
```

### Generic handleChange สำหรับ Form

```jsx
function RegistrationForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
  });
  
  // Generic handler - ใช้ได้กับทุก field
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  return (
    <form>
      <input
        name="username"        // ต้องตรงกับ key ใน state
        value={formData.username}
        onChange={handleChange}
        placeholder="ชื่อผู้ใช้"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="อีเมล"
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="รหัสผ่าน"
      />
      <input
        type="password"
        name="confirmPassword"
        value={formData.confirmPassword}
        onChange={handleChange}
        placeholder="ยืนยันรหัสผ่าน"
      />
    </form>
  );
}
```

---

## Step 105: Functional Updates {#step-105}

เมื่อ state ใหม่ขึ้นอยู่กับ state เก่า ควรใช้ **functional update** (ส่ง function เข้า setState)

### ทำไมต้องใช้ Functional Updates?

```jsx
// ❌ อาจมีปัญหา - ใช้ค่า count ที่อาจ stale
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleTripleClick = () => {
    // count ยังเป็น 0 ทั้งสาม calls!
    setCount(count + 1); // count = 0, set 1
    setCount(count + 1); // count ยังเป็น 0, set 1
    setCount(count + 1); // count ยังเป็น 0, set 1
    // ผลลัพธ์: count = 1 (ไม่ใช่ 3!)
  };
}

// ✅ ถูก - functional update ใช้ค่าล่าสุดเสมอ
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleTripleClick = () => {
    setCount(prev => prev + 1); // prev = 0, return 1
    setCount(prev => prev + 1); // prev = 1, return 2
    setCount(prev => prev + 1); // prev = 2, return 3
    // ผลลัพธ์: count = 3 ✅
  };
}
```

### เมื่อไหรควรใช้ Functional Updates?

1. เมื่อ state ใหม่ขึ้นอยู่กับ state เก่า
2. เมื่อเรียก setState หลายครั้งติดกัน
3. เมื่อ setState อยู่ใน async function หรือ callback

```jsx
function TaskManager() {
  const [tasks, setTasks] = useState([]);
  
  // ✅ Functional update กับ array
  const addTask = (newTask) => {
    setTasks(prev => [...prev, { id: Date.now(), ...newTask }]);
  };
  
  const removeTask = (id) => {
    setTasks(prev => prev.filter(task => task.id !== id));
  };
  
  const toggleTask = (id) => {
    setTasks(prev => prev.map(task =>
      task.id === id ? { ...task, done: !task.done } : task
    ));
  };
  
  // ✅ Functional update ใน async function
  const loadMoreTasks = async () => {
    const newTasks = await fetchTasks();
    setTasks(prev => [...prev, ...newTasks]);
  };
  
  return (
    <div>
      {tasks.map(task => (
        <div key={task.id}>
          <input type="checkbox" checked={task.done} onChange={() => toggleTask(task.id)} />
          <span style={{ textDecoration: task.done ? 'line-through' : 'none' }}>
            {task.title}
          </span>
          <button onClick={() => removeTask(task.id)}>ลบ</button>
        </div>
      ))}
      <button onClick={loadMoreTasks}>โหลดเพิ่ม</button>
    </div>
  );
}
```

---

## Step 106: Immutability ใน State Updates {#step-106}

**Immutability** หมายความว่าไม่แก้ไข data เดิม แต่สร้างสำเนาใหม่แทน

### ทำไม React ต้องการ Immutability?

React เปรียบเทียบ state เก่าและใหม่ด้วย reference equality (`===`) ถ้า mutate โดยตรง reference ไม่เปลี่ยน → React คิดว่า state ไม่เปลี่ยน → ไม่ re-render!

```jsx
// ❌ ผิด - Mutating state โดยตรง
function BadExample() {
  const [items, setItems] = useState([1, 2, 3]);
  
  const addItem = () => {
    items.push(4);      // Mutate! ❌
    setItems(items);    // Reference เดิม → React ไม่ re-render!
  };
  
  const [user, setUser] = useState({ name: "สมชาย", age: 25 });
  
  const updateAge = () => {
    user.age = 26;      // Mutate! ❌
    setUser(user);      // Reference เดิม → ไม่ re-render!
  };
}
```

```jsx
// ✅ ถูก - สร้าง copy ใหม่
function GoodExample() {
  const [items, setItems] = useState([1, 2, 3]);
  
  const addItem = () => {
    setItems([...items, 4]);         // Array ใหม่ ✅
    // หรือ
    setItems(prev => [...prev, 4]);  // Functional update ✅
  };
  
  const [user, setUser] = useState({ name: "สมชาย", age: 25 });
  
  const updateAge = () => {
    setUser({ ...user, age: 26 });           // Object ใหม่ ✅
    // หรือ
    setUser(prev => ({ ...prev, age: 26 })); // Functional update ✅
  };
}
```

### Immutable Operations Reference

```jsx
// ===== ARRAYS =====
const arr = [1, 2, 3];

// เพิ่มท้าย
const added = [...arr, 4];              // [1, 2, 3, 4]

// เพิ่มต้น
const prepended = [0, ...arr];          // [0, 1, 2, 3]

// เพิ่มตรงกลาง (index 1)
const inserted = [...arr.slice(0, 1), 1.5, ...arr.slice(1)]; // [1, 1.5, 2, 3]

// ลบ (index 1)
const removed = arr.filter((_, i) => i !== 1);  // [1, 3]

// แก้ไข (index 1)
const updated = arr.map((item, i) => i === 1 ? 99 : item); // [1, 99, 3]

// เรียงลำดับ
const sorted = [...arr].sort((a, b) => b - a); // [3, 2, 1]

// ===== OBJECTS =====
const obj = { a: 1, b: 2, c: { d: 3 } };

// อัพเดท property
const updated1 = { ...obj, b: 99 };             // { a: 1, b: 99, c: {...} }

// เพิ่ม property ใหม่
const added1 = { ...obj, newKey: "value" };      // { a: 1, b: 2, c: {...}, newKey: "value" }

// ลบ property
const { b, ...withoutB } = obj;                  // { a: 1, c: {...} }

// อัพเดท nested
const deepUpdated = {
  ...obj,
  c: { ...obj.c, d: 99 }                         // { a: 1, b: 2, c: { d: 99 } }
};
```

---

## Step 107: Multiple State Variables {#step-107}

React รองรับการใช้ `useState` หลายครั้งใน component เดียว

```jsx
function ProductPage() {
  // แยก state ตามหน้าที่
  const [product, setProduct] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const [selectedSize, setSelectedSize] = useState('M');
  const [selectedColor, setSelectedColor] = useState('red');
  const [isWishlisted, setIsWishlisted] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const totalPrice = product ? product.price * quantity : 0;
  
  const handleAddToCart = () => {
    console.log({
      product: product?.id,
      quantity,
      size: selectedSize,
      color: selectedColor
    });
  };
  
  return (
    <div>
      {isLoading && <p>โหลด...</p>}
      {error && <p>Error: {error}</p>}
      
      {product && (
        <div>
          <h1>{product.name}</h1>
          
          {/* Color selector */}
          <div>
            {['red', 'blue', 'green'].map(color => (
              <button
                key={color}
                onClick={() => setSelectedColor(color)}
                style={{ 
                  background: color,
                  border: selectedColor === color ? '3px solid black' : '1px solid gray'
                }}
              />
            ))}
          </div>
          
          {/* Size selector */}
          <div>
            {['S', 'M', 'L', 'XL'].map(size => (
              <button
                key={size}
                onClick={() => setSelectedSize(size)}
                className={selectedSize === size ? 'selected' : ''}
              >
                {size}
              </button>
            ))}
          </div>
          
          {/* Quantity */}
          <div>
            <button onClick={() => setQuantity(q => Math.max(1, q - 1))}>-</button>
            <span>{quantity}</span>
            <button onClick={() => setQuantity(q => q + 1)}>+</button>
          </div>
          
          <p>รวม: ฿{totalPrice}</p>
          
          <button onClick={handleAddToCart}>เพิ่มในตะกร้า</button>
          <button onClick={() => setIsWishlisted(prev => !prev)}>
            {isWishlisted ? '❤️' : '🤍'} Wishlist
          </button>
        </div>
      )}
    </div>
  );
}
```

### เมื่อไหรควรรวม State เป็น Object?

```jsx
// ✅ แยก state ที่ไม่เกี่ยวกัน
const [count, setCount] = useState(0);
const [isLoading, setIsLoading] = useState(false);

// ✅ รวม state ที่เกี่ยวข้องกัน
const [formData, setFormData] = useState({
  name: '',
  email: '',
  password: ''
});

// กฎง่ายๆ:
// รวม = state ที่อัพเดทพร้อมกันเสมอ หรือเกี่ยวข้องกันโดยตรง
// แยก = state ที่เปลี่ยนอิสระจากกัน
```

---

## Step 108: Lazy Initialization {#step-108}

ถ้า initial value ต้องคำนวณหนัก สามารถส่ง **function** เข้า useState เพื่อให้คำนวณแค่ครั้งแรก

```jsx
// ❌ คำนวณทุก render (แม้ใช้ครั้งแรกครั้งเดียว)
function ExpensiveComponent() {
  const [data, setData] = useState(heavyComputation()); // เรียกทุก render!
  
  return <div>{data}</div>;
}

// ✅ Lazy initialization - คำนวณครั้งเดียวตอนสร้าง
function ExpensiveComponent() {
  const [data, setData] = useState(() => heavyComputation()); // เรียกครั้งเดียว!
  
  return <div>{data}</div>;
}
```

### ตัวอย่างจริง - อ่านจาก localStorage

```jsx
function UserSettings() {
  // ✅ อ่าน localStorage แค่ครั้งเดียวตอนสร้าง
  const [settings, setSettings] = useState(() => {
    try {
      const saved = localStorage.getItem('userSettings');
      return saved ? JSON.parse(saved) : {
        theme: 'light',
        language: 'th',
        fontSize: 16
      };
    } catch {
      return { theme: 'light', language: 'th', fontSize: 16 };
    }
  });
  
  const updateSetting = (key, value) => {
    const newSettings = { ...settings, [key]: value };
    setSettings(newSettings);
    // บันทึกลง localStorage
    localStorage.setItem('userSettings', JSON.stringify(newSettings));
  };
  
  return (
    <div>
      <select 
        value={settings.theme}
        onChange={e => updateSetting('theme', e.target.value)}
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>
  );
}
```

---

## Step 109: Counter Example {#step-109}

ตัวอย่างสมบูรณ์ของ Counter ที่ใช้ State ครบถ้วน

```jsx
function AdvancedCounter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
  const [min, setMin] = useState(null);
  const [max, setMax] = useState(null);
  const [history, setHistory] = useState([0]);
  
  const increment = () => {
    const newCount = count + step;
    if (max !== null && newCount > max) return;
    setCount(newCount);
    setHistory(prev => [...prev, newCount]);
  };
  
  const decrement = () => {
    const newCount = count - step;
    if (min !== null && newCount < min) return;
    setCount(newCount);
    setHistory(prev => [...prev, newCount]);
  };
  
  const reset = () => {
    setCount(0);
    setHistory([0]);
  };
  
  const undo = () => {
    if (history.length <= 1) return;
    const newHistory = history.slice(0, -1);
    setHistory(newHistory);
    setCount(newHistory[newHistory.length - 1]);
  };
  
  return (
    <div className="counter">
      <h2>Advanced Counter</h2>
      
      <div className="settings">
        <label>
          Step:
          <input 
            type="number" 
            value={step}
            min={1}
            onChange={e => setStep(Math.max(1, Number(e.target.value)))}
          />
        </label>
        <label>
          Min:
          <input 
            type="number"
            value={min ?? ''}
            onChange={e => setMin(e.target.value ? Number(e.target.value) : null)}
            placeholder="ไม่จำกัด"
          />
        </label>
        <label>
          Max:
          <input 
            type="number"
            value={max ?? ''}
            onChange={e => setMax(e.target.value ? Number(e.target.value) : null)}
            placeholder="ไม่จำกัด"
          />
        </label>
      </div>
      
      <div className="display">
        <span className="count">{count}</span>
      </div>
      
      <div className="controls">
        <button onClick={decrement} disabled={min !== null && count - step < min}>
          -{step}
        </button>
        <button onClick={reset}>Reset</button>
        <button onClick={undo} disabled={history.length <= 1}>Undo</button>
        <button onClick={increment} disabled={max !== null && count + step > max}>
          +{step}
        </button>
      </div>
      
      <div className="history">
        <p>History: {history.join(' → ')}</p>
      </div>
    </div>
  );
}
```

---

## Step 110: Toggle Example {#step-110}

```jsx
function ToggleSwitch({ label, initialOn = false, onChange }) {
  const [isOn, setIsOn] = useState(initialOn);
  
  const toggle = () => {
    const newValue = !isOn;
    setIsOn(newValue);
    onChange?.(newValue);
  };
  
  return (
    <div className="toggle-switch">
      <span>{label}</span>
      <button
        onClick={toggle}
        style={{
          width: 60,
          height: 30,
          backgroundColor: isOn ? '#4CAF50' : '#ccc',
          border: 'none',
          borderRadius: 15,
          cursor: 'pointer',
          position: 'relative',
          transition: 'background-color 0.2s'
        }}
        role="switch"
        aria-checked={isOn}
      >
        <span
          style={{
            position: 'absolute',
            width: 26,
            height: 26,
            backgroundColor: 'white',
            borderRadius: '50%',
            top: 2,
            left: isOn ? 32 : 2,
            transition: 'left 0.2s'
          }}
        />
      </button>
      <span>{isOn ? 'เปิด' : 'ปิด'}</span>
    </div>
  );
}

// การใช้งาน
function Settings() {
  const [notifications, setNotifications] = useState(false);
  const [darkMode, setDarkMode] = useState(false);
  
  return (
    <div>
      <ToggleSwitch
        label="การแจ้งเตือน"
        initialOn={notifications}
        onChange={setNotifications}
      />
      <ToggleSwitch
        label="โหมดมืด"
        initialOn={darkMode}
        onChange={setDarkMode}
      />
    </div>
  );
}
```

---

## Step 111: Form State Example {#step-111}

```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    phone: '',
    subject: '',
    message: '',
    agree: false
  });
  
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isSubmitted, setIsSubmitted] = useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    // ล้าง error field นั้น
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'กรุณากรอกชื่อ';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'กรุณากรอกอีเมล';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'รูปแบบอีเมลไม่ถูกต้อง';
    }
    
    if (!formData.message.trim()) {
      newErrors.message = 'กรุณากรอกข้อความ';
    } else if (formData.message.length < 10) {
      newErrors.message = 'ข้อความต้องมีอย่างน้อย 10 ตัวอักษร';
    }
    
    if (!formData.agree) {
      newErrors.agree = 'กรุณายอมรับเงื่อนไข';
    }
    
    return newErrors;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    const validationErrors = validate();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      // Mock API call
      await new Promise(resolve => setTimeout(resolve, 1500));
      console.log('Form submitted:', formData);
      setIsSubmitted(true);
      setFormData({
        name: '', email: '', phone: '',
        subject: '', message: '', agree: false
      });
    } catch (error) {
      setErrors({ submit: 'เกิดข้อผิดพลาด กรุณาลองใหม่' });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  if (isSubmitted) {
    return (
      <div className="success-message">
        <h2>✅ ส่งข้อความสำเร็จ!</h2>
        <p>เราจะติดต่อกลับภายใน 24 ชั่วโมง</p>
        <button onClick={() => setIsSubmitted(false)}>ส่งข้อความอื่น</button>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>ชื่อ *</label>
        <input
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>อีเมล *</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>เบอร์โทร</label>
        <input
          name="phone"
          value={formData.phone}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>เรื่อง</label>
        <input
          name="subject"
          value={formData.subject}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>ข้อความ *</label>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows={5}
        />
        {errors.message && <span className="error">{errors.message}</span>}
        <small>{formData.message.length}/500 ตัวอักษร</small>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="agree"
            checked={formData.agree}
            onChange={handleChange}
          />
          ฉันยอมรับเงื่อนไขการใช้บริการ
        </label>
        {errors.agree && <span className="error">{errors.agree}</span>}
      </div>
      
      {errors.submit && <p className="error">{errors.submit}</p>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '⏳ กำลังส่ง...' : '📨 ส่งข้อความ'}
      </button>
    </form>
  );
}
```

---

## Step 112-114: Array State Operations (สรุป) {#step-112}

สรุป patterns สำคัญสำหรับ array state:

```jsx
// ========================================
// CRUD Operations สำหรับ Array State
// ========================================

function useList(initialItems = []) {
  const [items, setItems] = useState(initialItems);
  
  // Create - เพิ่มรายการ
  const addItem = (item) => {
    setItems(prev => [...prev, { id: Date.now(), ...item }]);
  };
  
  // Read - ไม่ต้องทำอะไร แค่ return items
  
  // Update - แก้ไขรายการ
  const updateItem = (id, updates) => {
    setItems(prev => prev.map(item => 
      item.id === id ? { ...item, ...updates } : item
    ));
  };
  
  // Delete - ลบรายการ
  const deleteItem = (id) => {
    setItems(prev => prev.filter(item => item.id !== id));
  };
  
  // Extra operations
  const reorderItems = (fromIndex, toIndex) => {
    setItems(prev => {
      const result = [...prev];
      const [removed] = result.splice(fromIndex, 1);
      result.splice(toIndex, 0, removed);
      return result;
    });
  };
  
  const sortItems = (compareFn) => {
    setItems(prev => [...prev].sort(compareFn));
  };
  
  const clearItems = () => setItems([]);
  
  return { items, addItem, updateItem, deleteItem, reorderItems, sortItems, clearItems };
}
```

---

## Step 115: Object State Operations {#step-115}

```jsx
function useObjectState(initialState) {
  const [state, setState] = useState(initialState);
  
  // อัพเดท field เดียว
  const setField = (field, value) => {
    setState(prev => ({ ...prev, [field]: value }));
  };
  
  // Merge object
  const mergeState = (updates) => {
    setState(prev => ({ ...prev, ...updates }));
  };
  
  // Reset
  const reset = () => setState(initialState);
  
  return [state, { setField, mergeState, reset, setState }];
}

// การใช้งาน
function ProfileEditor() {
  const [profile, { setField, mergeState, reset }] = useObjectState({
    name: 'สมชาย',
    email: 'somchai@example.com',
    bio: '',
    website: '',
    isPublic: true
  });
  
  return (
    <form>
      <input
        value={profile.name}
        onChange={e => setField('name', e.target.value)}
        placeholder="ชื่อ"
      />
      <input
        value={profile.email}
        onChange={e => setField('email', e.target.value)}
        placeholder="อีเมล"
      />
      <textarea
        value={profile.bio}
        onChange={e => setField('bio', e.target.value)}
        placeholder="เกี่ยวกับตัวคุณ"
      />
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

---

## Step 116: State Batching {#step-116}

React 18+ จะ batch state updates อัตโนมัติ แม้ใน setTimeout และ async functions

```jsx
function BatchingExample() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  const [items, setItems] = useState([]);
  
  console.log("Render!"); // ดู render count
  
  // React 18+: batch อัตโนมัติ = render ครั้งเดียว
  const handleClick = () => {
    setCount(c => c + 1);  // ไม่ render ทันที
    setText('updated');    // ไม่ render ทันที
    setItems(['a', 'b']);  // render ครั้งเดียวหลังจากนี้
  };
  
  // React 18+: batch ใน async ด้วย
  const handleAsync = async () => {
    await someAsyncOperation();
    setCount(c => c + 1);  // batch
    setText('async done'); // render ครั้งเดียว
  };
  
  // Force immediate render (ไม่ค่อยจำเป็น)
  // import { flushSync } from 'react-dom';
  // flushSync(() => setCount(c => c + 1)); // render ทันที
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Text: {text}</p>
      <button onClick={handleClick}>Update All</button>
    </div>
  );
}
```

---

## Step 117: State ที่ derived จาก State อื่น {#step-117}

**Derived state** คือค่าที่คำนวณจาก state อื่น ไม่ต้องเก็บใน state แยก

```jsx
function ShoppingCart() {
  const [items, setItems] = useState([
    { id: 1, name: "หูฟัง", price: 999, quantity: 1 },
    { id: 2, name: "คีย์บอร์ด", price: 1299, quantity: 2 },
  ]);
  const [discountCode, setDiscountCode] = useState('');
  const [appliedDiscount, setAppliedDiscount] = useState(0);
  
  // ❌ ผิด - ไม่ต้องเก็บ derived values ใน state
  // const [totalItems, setTotalItems] = useState(0);
  // const [subtotal, setSubtotal] = useState(0);
  // const [total, setTotal] = useState(0);
  
  // ✅ ถูก - คำนวณจาก items state
  const totalItems = items.reduce((sum, item) => sum + item.quantity, 0);
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const discountAmount = subtotal * (appliedDiscount / 100);
  const total = subtotal - discountAmount;
  
  const applyDiscount = () => {
    const codes = { 'SAVE10': 10, 'SAVE20': 20, 'HALF': 50 };
    setAppliedDiscount(codes[discountCode] || 0);
  };
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          <span>x{item.quantity}</span>
          <span>฿{item.price * item.quantity}</span>
        </div>
      ))}
      
      <p>จำนวน: {totalItems} ชิ้น</p>
      <p>ราคารวม: ฿{subtotal}</p>
      {appliedDiscount > 0 && (
        <p>ส่วนลด {appliedDiscount}%: -฿{discountAmount}</p>
      )}
      <p>ชำระ: ฿{total}</p>
      
      <input 
        value={discountCode} 
        onChange={e => setDiscountCode(e.target.value)}
        placeholder="รหัสส่วนลด"
      />
      <button onClick={applyDiscount}>ใช้โค้ด</button>
    </div>
  );
}
```

---

## Step 118: State Reset Patterns {#step-118}

```jsx
// Pattern 1: Reset ด้วย key prop
function ResettableForm({ userId }) {
  return (
    // เปลี่ยน key = สร้าง component ใหม่ = reset state
    <UserForm key={userId} userId={userId} />
  );
}

function UserForm({ userId }) {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  return (
    <form>
      <input value={name} onChange={e => setName(e.target.value)} />
      <input value={email} onChange={e => setEmail(e.target.value)} />
    </form>
  );
}

// Pattern 2: Manual reset
function EditForm({ initialData }) {
  const [data, setData] = useState(initialData);
  
  const reset = () => setData(initialData);
  const resetField = (field) => {
    setData(prev => ({ ...prev, [field]: initialData[field] }));
  };
  
  return (
    <form>
      <input 
        value={data.name}
        onChange={e => setData(p => ({ ...p, name: e.target.value }))}
      />
      <button type="button" onClick={() => resetField('name')}>
        Reset Name
      </button>
      <button type="button" onClick={reset}>
        Reset All
      </button>
    </form>
  );
}
```

---

## Step 119: Common State Mistakes {#step-119}

### Mistake 1: อ่าน State ทันทีหลัง setState

```jsx
// ❌ ผิด - count ยังเป็นค่าเก่า
function BadExample() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setCount(count + 1);
    console.log(count); // ยังเป็น 0! (state อัพเดทใน next render)
  };
}

// ✅ ถูก - คำนวณค่าใหม่แยก
function GoodExample() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    const newCount = count + 1;
    setCount(newCount);
    console.log(newCount); // ✅ 1
  };
}
```

### Mistake 2: Mutating State โดยตรง

```jsx
// ❌ ผิด
const [user, setUser] = useState({ name: "สมชาย", scores: [80, 90] });

const addScore = (score) => {
  user.scores.push(score); // Mutate!
  setUser(user);           // Reference เดิม → ไม่ re-render
};

// ✅ ถูก
const addScore = (score) => {
  setUser(prev => ({
    ...prev,
    scores: [...prev.scores, score]
  }));
};
```

### Mistake 3: State ที่ไม่จำเป็น

```jsx
// ❌ ผิด - เก็บ derived value ใน state
const [items, setItems] = useState([]);
const [count, setCount] = useState(0); // ไม่จำเป็น!

// ✅ ถูก - คำนวณตรงๆ
const [items, setItems] = useState([]);
const count = items.length; // derived
```

### Mistake 4: setState ใน render

```jsx
// ❌ ผิด - infinite loop!
function BadComponent() {
  const [count, setCount] = useState(0);
  setCount(count + 1); // เรียกทุก render → re-render → เรียกอีก → infinite loop!
  return <div>{count}</div>;
}
```

---

## Step 120: State Management Best Practices {#step-120}

### 1. State ใกล้กับที่ใช้งาน

```jsx
// ✅ ไม่ต้อง lift state ขึ้นไปโดยไม่จำเป็น
function ParentComponent() {
  return (
    <div>
      <ChildA />  {/* ChildA จัดการ state ของตัวเอง */}
      <ChildB />  {/* ChildB จัดการ state ของตัวเอง */}
    </div>
  );
}

// Lift state ขึ้นไปเมื่อต้องแชร์ระหว่าง siblings
function ParentWithSharedState() {
  const [sharedData, setSharedData] = useState(null);
  
  return (
    <div>
      <ChildA onChange={setSharedData} />
      <ChildB data={sharedData} />
    </div>
  );
}
```

### 2. ใช้ Immutable Updates เสมอ

```jsx
// ✅ เสมอ
setItems(prev => [...prev, newItem]);
setUser(prev => ({ ...prev, name: newName }));
```

### 3. Functional Updates เมื่อขึ้นอยู่กับค่าเก่า

```jsx
// ✅ ปลอดภัยกว่า
setCount(prev => prev + 1);
setItems(prev => [...prev, newItem]);
```

### 4. ไม่เก็บ Derived State

```jsx
// ✅ คำนวณตรงๆ ใน render
const total = items.reduce((sum, item) => sum + item.price, 0);
```

### 5. Group Related State

```jsx
// ✅ รวม state ที่เกี่ยวข้อง
const [pagination, setPagination] = useState({
  page: 1,
  pageSize: 10,
  total: 0
});
```

### 6. Validate State Updates

```jsx
const updateQuantity = (id, quantity) => {
  if (quantity < 1 || quantity > 99) return; // validate ก่อน
  setItems(prev => prev.map(item => 
    item.id === id ? { ...item, quantity } : item
  ));
};
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม

1. ทำไม React ต้องการ Immutability ใน State Updates?
2. Functional Updates แตกต่างจาก direct updates อย่างไร?
3. เมื่อไหรควรใช้ Lazy Initialization?
4. Derived State คืออะไร? ทำไมไม่ควรเก็บใน state?
5. Rules of Hooks มีกฎอะไรบ้าง?

### แบบฝึกหัด

**Exercise 1:** สร้าง `ShoppingList` component:
- เพิ่มรายการจาก input
- ลบรายการ
- Toggle done/undone
- แสดงจำนวนรายการที่ทำแล้ว vs ยังไม่ได้ทำ

**Exercise 2:** สร้าง `ImageGallery` component:
- State: `images` (array), `selectedIndex`, `isSlideshow`
- ปุ่ม Next/Prev
- ปุ่ม Toggle Slideshow (auto advance ทุก 3 วินาที)
- แสดง current/total

**Exercise 3:** สร้าง `SettingsPanel` component:
- State เป็น object: `{ theme, language, notifications, autoSave }`
- Controls สำหรับแต่ละ setting
- ปุ่ม Reset to defaults
- แสดง JSON preview ของ settings

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- State คืออะไร และต่างจาก Props อย่างไร
- useState Hook อย่างละเอียด
- State ทุกประเภท: string, number, boolean, array, object
- Functional Updates
- Immutability ใน State Updates
- Multiple State Variables
- Lazy Initialization
- ตัวอย่างสมบูรณ์: Counter, Toggle, Form

---

## อ่านต่อ

➡️ **[Part 07: Events และ Event Handlers](./part-07-events-and-handlers.md)**

เราจะเรียนรู้ Synthetic Events, Event Types ทั้งหมด, Form Events และการส่ง Arguments ให้ Event Handlers

---

*React Course - Part 06 | สร้างโดย React Course Thailand*
