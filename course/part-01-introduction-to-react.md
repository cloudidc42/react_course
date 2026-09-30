# Part 01: แนะนำ React.js และ Web Development
## Step 1-10 | ระดับ: พื้นฐานสุด

---

## 📋 สิ่งที่จะได้เรียนรู้ใน Part นี้

- React.js คืออะไร และทำไมถึงนิยมใช้
- ประวัติและวิวัฒนาการของ React
- เปรียบเทียบ React กับ Framework อื่นๆ
- แนวคิดหลักของ React
- Virtual DOM คืออะไร
- Component-Based Architecture
- โครงสร้างของ React App
- ภาพรวมของ Ecosystem React

---

## Step 1: React.js คืออะไร?

### ความหมายและนิยาม

**React.js** คือ JavaScript Library สำหรับสร้าง User Interface (UI) ที่ถูกพัฒนาโดย **Meta (Facebook)** และเปิดเป็น Open Source ในปี 2013

> 💡 **สำคัญ**: React เป็น **Library** ไม่ใช่ **Framework** เต็มรูปแบบ ซึ่งหมายความว่า React โฟกัสเฉพาะที่ส่วน View (UI) เท่านั้น ส่วนอื่นๆ เช่น Routing, State Management ต้องใช้ Library เพิ่มเติม

### React ทำอะไรได้บ้าง?

```
📱 Web Applications     → Facebook, Instagram, WhatsApp Web
🖥️ Desktop Apps        → Electron + React
📲 Mobile Apps         → React Native
🌐 Server-Side         → Next.js, Remix
🎮 Games/Interactive   → Three.js + React Three Fiber
```

### ทำไมต้องเรียน React?

1. **ความนิยมสูงสุด** - อันดับ 1 ใน Frontend Framework Survey ต่อเนื่องหลายปี
2. **Job Market** - บริษัทใหญ่ๆ ทั่วโลกใช้ React
3. **Community ใหญ่** - มี Library, Tutorial, Support มากมาย
4. **Reusable Components** - เขียนครั้งเดียวใช้ได้หลายที่
5. **Performance ดี** - ใช้ Virtual DOM ที่ฉลาด

---

## Step 2: ประวัติและวิวัฒนาการของ React

### Timeline ของ React

```
2011 │ Jordan Walke สร้าง FaxJS (ต้นแบบของ React) ที่ Facebook
     │
2013 │ React 0.3.0 เปิดตัวที่ JSConf US
     │ เป็น Open Source บน GitHub
     │
2015 │ React Native เปิดตัว (สำหรับ Mobile)
     │ React 0.13 - รองรับ ES6 Classes
     │
2016 │ React 15.0 - ปรับปรุง Performance
     │
2017 │ React 16.0 "Fiber" - เขียนใหม่ทั้งหมด
     │ Error Boundaries, Portals, Fragments
     │
2018 │ React 16.3 - Context API ใหม่
     │
2019 │ React 16.8 🎉 - เปิดตัว Hooks!
     │ (เปลี่ยนแปลงวิธีเขียน React ไปตลอดกาล)
     │
2020 │ React 17.0 - ไม่มีฟีเจอร์ใหม่ แต่ปรับ internals
     │
2022 │ React 18.0 🚀 - Concurrent Features
     │ Automatic Batching, Transitions, Suspense
     │
2024 │ React 19.0 - React Compiler, Actions
     │
2025+│ React 19.x - ปรับปรุงต่อเนื่อง
```

### ความเปลี่ยนแปลงสำคัญ: ก่อน vs หลัง Hooks (React 16.8)

**ก่อน Hooks (Class Components)**:
```jsx
// วิธีเก่า - Class Component
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>+1</button>
      </div>
    );
  }
}
```

**หลัง Hooks (Function Components)**:
```jsx
// วิธีใหม่ - Function Component + Hooks
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

> 💡 **ข้อสังเกต**: Hooks ทำให้โค้ดสั้น อ่านง่าย และ Reuse Logic ได้ง่ายขึ้นมาก ปัจจุบันนี้แนะนำให้ใช้ Function Components + Hooks เสมอ

---

## Step 3: เปรียบเทียบ React กับ Framework อื่นๆ

### React vs Vue.js vs Angular

| คุณสมบัติ | React | Vue.js | Angular |
|-----------|-------|--------|---------|
| ประเภท | Library | Framework | Full Framework |
| บริษัท | Meta | Community | Google |
| ขนาด Bundle | ~45KB | ~33KB | ~130KB |
| Syntax | JSX | Template | TypeScript + Decorators |
| Learning Curve | ปานกลาง | ต่ำ | สูง |
| Flexibility | สูงมาก | สูง | ต่ำ (Opinionated) |
| Job Market | สูงสุด | ปานกลาง | ปานกลาง |
| TypeScript | Optional | Optional | Built-in |

### เมื่อไหร่ควรใช้ React?

```
✅ ใช้ React เมื่อ:
   → ต้องการ Flexibility ในการเลือก Library
   → ทีมมีประสบการณ์ JavaScript
   → ต้องการ Ecosystem ที่ใหญ่
   → ต้องการ Job Market ที่กว้าง

✅ ใช้ Vue เมื่อ:
   → ต้องการเรียนรู้เร็ว
   → ทีมมาจาก Traditional Web Background
   → ชอบ Single File Components

✅ ใช้ Angular เมื่อ:
   → Enterprise Application ขนาดใหญ่
   → ต้องการ Opinionated Structure
   → ทีมชอบ TypeScript first approach
```

---

## Step 4: แนวคิดหลักของ React

### 4.1 Component-Based Architecture

ใน React ทุกอย่างคือ **Component** - เป็นส่วนๆ ของ UI ที่สามารถ Reuse ได้

```
🏠 App (Root Component)
├── 🧭 Navbar
│   ├── 🔗 Logo
│   ├── 🔗 NavLinks
│   └── 👤 UserMenu
├── 📄 MainContent
│   ├── 🃏 ProductCard
│   ├── 🃏 ProductCard
│   └── 🃏 ProductCard
└── 🦶 Footer
```

```jsx
// ตัวอย่าง: แต่ละ UI element คือ Component
function Logo() {
  return <img src="/logo.png" alt="Logo" />;
}

function NavLinks() {
  return (
    <nav>
      <a href="/">หน้าแรก</a>
      <a href="/about">เกี่ยวกับ</a>
      <a href="/contact">ติดต่อ</a>
    </nav>
  );
}

function Navbar() {
  // Navbar ประกอบด้วย Logo และ NavLinks
  return (
    <header>
      <Logo />
      <NavLinks />
    </header>
  );
}
```

### 4.2 Declarative Programming

React ใช้แนวคิด **Declarative** - เราบอก React ว่า UI ควรเป็นอย่างไร ไม่ใช่บอกว่าต้องทำอะไรทีละขั้น

```jsx
// ❌ Imperative (วิธี Vanilla JS)
const button = document.getElementById('btn');
const counter = document.getElementById('counter');
let count = 0;

button.addEventListener('click', function() {
  count++;
  counter.textContent = count; // บอกว่าต้องทำอะไร
});

// ✅ Declarative (วิธี React)
function Counter() {
  const [count, setCount] = useState(0);
  
  // บอกว่า UI ควรเป็นอย่างไรตาม state
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

### 4.3 Unidirectional Data Flow

ข้อมูลใน React ไหลทิศทางเดียว: **ลงมาเท่านั้น** (Parent → Child)

```
┌─────────────────────────────────┐
│         Parent Component        │
│  state = { user: "สมชาย" }      │
│                                 │
│  ส่ง props ลงไป               │
└─────────────┬───────────────────┘
              │ props.user = "สมชาย"
              ▼
┌─────────────────────────────────┐
│         Child Component         │
│  รับ props และแสดงผล            │
│  <p>{props.user}</p>            │
└─────────────────────────────────┘
```

```jsx
// Parent ส่งข้อมูลผ่าน props
function Parent() {
  const [username, setUsername] = useState("สมชาย");
  
  return <Child name={username} />;
}

// Child รับข้อมูลผ่าน props
function Child({ name }) {
  return <p>สวัสดี, {name}!</p>;
}
```

---

## Step 5: Virtual DOM คืออะไร?

### ปัญหาของ DOM จริง (Real DOM)

DOM (Document Object Model) คือโครงสร้างของ HTML Page การแก้ไข DOM โดยตรงมีปัญหา:

1. **ช้า** - การอัปเดต DOM จริงใช้เวลานาน
2. **Reflow/Repaint** - ทุกครั้งที่แก้ไข Browser ต้อง re-render ใหม่
3. **ไม่มีประสิทธิภาพ** - บางครั้งต้อง update element ที่ไม่จำเป็น

### Virtual DOM แก้ปัญหาอย่างไร?

```
📝 State เปลี่ยน
       │
       ▼
┌─────────────────┐
│   Virtual DOM   │ ← React สร้าง Virtual DOM ใหม่ (เร็วมาก)
│   (JavaScript   │
│    Object)      │
└────────┬────────┘
         │
         ▼ Diffing Algorithm (เปรียบเทียบ Virtual DOM เก่าและใหม่)
         │
         ▼
┌─────────────────┐
│  Only Changed   │ ← Update เฉพาะส่วนที่เปลี่ยนแปลงใน Real DOM
│  Parts Updated  │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│    Real DOM     │ ← Browser แสดงผล
└─────────────────┘
```

### ตัวอย่างการทำงาน Virtual DOM

```jsx
// สมมติ state เปลี่ยนจาก count=0 เป็น count=1

// Virtual DOM เก่า (JavaScript Object):
{
  type: 'div',
  children: [
    { type: 'p', children: ['Count: 0'] },  // ← เปลี่ยน
    { type: 'button', children: ['+1'] }    // ← ไม่เปลี่ยน
  ]
}

// Virtual DOM ใหม่:
{
  type: 'div',
  children: [
    { type: 'p', children: ['Count: 1'] },  // ← เปลี่ยน
    { type: 'button', children: ['+1'] }    // ← ไม่เปลี่ยน
  ]
}

// React จะ Update เฉพาะ <p> ใน Real DOM
// ไม่ต้อง Re-render ทั้ง div และ button
```

### Virtual DOM vs Real DOM

| | Real DOM | Virtual DOM |
|--|----------|-------------|
| ความเร็ว | ช้า (Browser Operation) | เร็ว (In-memory JS) |
| Update | Update ทั้ง tree | Update เฉพาะส่วนที่เปลี่ยน |
| Memory | น้อยกว่า | ใช้ Memory มากกว่า |
| ความซับซ้อน | ง่าย | ซับซ้อนกว่า แต่จัดการโดย React |

---

## Step 6: โครงสร้างของ React Application

### โครงสร้างไฟล์พื้นฐาน (Vite + React)

```
my-react-app/
├── 📁 public/               ← ไฟล์ Static (รูป, favicon)
│   └── vite.svg
├── 📁 src/                  ← Source Code หลัก
│   ├── 📁 assets/           ← รูปภาพ, Fonts
│   │   └── react.svg
│   ├── 📁 components/       ← React Components
│   │   ├── Button.jsx
│   │   └── Navbar.jsx
│   ├── 📁 pages/            ← หน้าต่างๆ
│   │   ├── Home.jsx
│   │   └── About.jsx
│   ├── 📁 hooks/            ← Custom Hooks
│   │   └── useAuth.js
│   ├── 📁 context/          ← Context Providers
│   │   └── AuthContext.jsx
│   ├── 📁 utils/            ← Utility Functions
│   │   └── formatDate.js
│   ├── App.jsx              ← Root Component
│   ├── App.css              ← Global Styles
│   ├── index.css            ← Base Styles
│   └── main.jsx             ← Entry Point
├── .eslintrc.cjs            ← ESLint Config
├── .gitignore
├── index.html               ← HTML Template
├── package.json             ← Dependencies
├── vite.config.js           ← Vite Config
└── README.md
```

### ไฟล์สำคัญที่ต้องรู้

**1. `index.html` - จุดเริ่มต้นของทุกอย่าง**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My React App</title>
  </head>
  <body>
    <!-- React App จะ render เข้าไปใน div นี้ -->
    <div id="root"></div>
    
    <!-- Entry point ของ React -->
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

**2. `src/main.jsx` - Entry Point ของ React**
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

// สร้าง React Root และ Render App เข้าไปใน #root
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

**3. `src/App.jsx` - Root Component**
```jsx
import './App.css'

function App() {
  return (
    <div className="App">
      <h1>สวัสดี React!</h1>
    </div>
  )
}

export default App
```

---

## Step 7: Ecosystem ของ React

### แผนที่ Ecosystem React

```
                    ┌─────────────┐
                    │   REACT     │
                    │   (Core)    │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼──────┐  ┌──────▼─────┐  ┌──────▼──────┐
    │  Rendering │  │   State    │  │   Routing   │
    └─────┬──────┘  └──────┬─────┘  └──────┬──────┘
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼───────┐
   │ Next.js     │  │ Redux Toolkit│  │ React Router│
   │ Remix       │  │ Zustand      │  │ TanStack    │
   │ Gatsby      │  │ Jotai        │  │ Router      │
   └─────────────┘  │ Recoil       │  └─────────────┘
                    └─────────────┘

    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
    │  Styling    │  │   Forms     │  │   Testing   │
    └─────┬───────┘  └──────┬──────┘  └──────┬──────┘
          │                 │                 │
   ┌──────▼──────┐  ┌───────▼─────┐  ┌───────▼─────┐
   │ Tailwind CSS│  │ React Hook  │  │ Jest        │
   │ styled-comp │  │ Form        │  │ React Test  │
   │ MUI         │  │ Formik      │  │ Library     │
   │ Chakra UI   │  │ Zod         │  │ Playwright  │
   └─────────────┘  └─────────────┘  └─────────────┘
```

### Library ยอดนิยมที่ต้องรู้

| หมวดหมู่ | Library | คะแนน npm/week |
|---------|---------|----------------|
| Meta-Framework | Next.js | 6M+ |
| State | Redux Toolkit | 3M+ |
| State | Zustand | 3M+ |
| Data Fetching | TanStack Query | 4M+ |
| Forms | React Hook Form | 5M+ |
| Validation | Zod | 6M+ |
| Styling | Tailwind CSS | 10M+ |
| UI Components | shadcn/ui | Rapidly growing |
| Testing | React Testing Library | 5M+ |
| Animation | Framer Motion | 2M+ |

---

## Step 8: React 18 และ Concurrent Features

### ฟีเจอร์ใหม่ใน React 18

**1. Automatic Batching** - รวม State Updates อัตโนมัติ
```jsx
// React 17 - ไม่ Batch ใน async functions
setTimeout(() => {
  setCount(c => c + 1); // Re-render ครั้งที่ 1
  setFlag(f => !f);     // Re-render ครั้งที่ 2
}, 1000);

// React 18 - Batch อัตโนมัติ แม้ใน async
setTimeout(() => {
  setCount(c => c + 1); // }
  setFlag(f => !f);     // } Re-render แค่ครั้งเดียว!
}, 1000);
```

**2. Transitions** - แยก UI Update ที่เร่งด่วนและไม่เร่งด่วน
```jsx
import { startTransition, useTransition } from 'react';

function SearchApp() {
  const [isPending, startTransition] = useTransition();
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  function handleSearch(input) {
    // เร่งด่วน: อัปเดต Input ทันที
    setQuery(input);
    
    // ไม่เร่งด่วน: ค้นหา (สามารถ interrupt ได้)
    startTransition(() => {
      setResults(searchDatabase(input));
    });
  }

  return (
    <div>
      <input onChange={e => handleSearch(e.target.value)} />
      {isPending ? <Spinner /> : <ResultsList results={results} />}
    </div>
  );
}
```

**3. Suspense บน Server** - Loading States ที่ดีขึ้น
```jsx
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <DataComponent /> {/* โหลดข้อมูลแบบ async */}
    </Suspense>
  );
}
```

---

## Step 9: React 19 - ฟีเจอร์ล่าสุด

### React Actions

```jsx
// React 19: Form Actions (ใหม่!)
function AddToCart({ productId }) {
  // useActionState จัดการ loading และ error อัตโนมัติ
  const [error, submitAction, isPending] = useActionState(
    async (prevState, formData) => {
      const error = await addToCartAsync(productId, formData);
      if (error) return error;
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <button type="submit" disabled={isPending}>
        {isPending ? 'กำลังเพิ่ม...' : 'เพิ่มในตะกร้า'}
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

### React Compiler (React 19)

```jsx
// ก่อน React Compiler: ต้องใช้ useMemo, useCallback เอง
function ExpensiveComponent({ data, onClick }) {
  const processedData = useMemo(() => processData(data), [data]);
  const handleClick = useCallback(() => onClick(data), [data, onClick]);
  
  return <div onClick={handleClick}>{processedData}</div>;
}

// หลัง React Compiler: Compiler จัดการ Optimization ให้อัตโนมัติ!
function ExpensiveComponent({ data, onClick }) {
  const processedData = processData(data); // Compiler optimize เอง
  const handleClick = () => onClick(data);  // ไม่ต้อง useCallback!
  
  return <div onClick={handleClick}>{processedData}</div>;
}
```

---

## Step 10: สรุปแนวคิดหลักและเตรียมพร้อมสำหรับการเรียน

### Mindset สำคัญสำหรับการเรียน React

```
🧠 Think in Components
   → แบ่ง UI เป็นส่วนย่อยๆ
   → แต่ละส่วนรับผิดชอบตัวเอง
   → Reusable และ Composable

🔄 Think in State
   → UI คือ Snapshot ของ State
   → State เปลี่ยน → UI อัปเดต
   → ไม่ Mutate State โดยตรง

⬇️ Think Unidirectional
   → ข้อมูลไหลลงมาผ่าน Props
   → Events ไหลขึ้นไปผ่าน Callbacks
   → ทำให้ Debug ง่ายขึ้น

✨ Think Declarative
   → บอกว่า UI ควรเป็นอย่างไร
   → ไม่บอกว่าต้องทำอะไร
   → React จัดการ DOM ให้เอง
```

### Checklist ก่อนเริ่ม Part ถัดไป

- [ ] เข้าใจว่า React คืออะไรและทำไมต้องใช้
- [ ] รู้จัก Component คืออะไร
- [ ] เข้าใจ Virtual DOM คืออะไร
- [ ] รู้จัก Declarative vs Imperative
- [ ] เข้าใจ Unidirectional Data Flow
- [ ] รู้จัก Ecosystem ของ React

### โค้ดตัวอย่างแรก: Hello React World

```jsx
// สร้างไฟล์ HelloWorld.jsx
import React from 'react';

// Component คือ Function ที่ Return JSX
function HelloWorld() {
  return (
    <div>
      <h1>สวัสดี โลก! 🌍</h1>
      <p>นี่คือ React Component แรกของฉัน</p>
    </div>
  );
}

export default HelloWorld;
```

```jsx
// ใช้ใน App.jsx
import HelloWorld from './HelloWorld';

function App() {
  return (
    <div>
      <HelloWorld />
    </div>
  );
}

export default App;
```

### คำถามทบทวน

1. React เป็น Library หรือ Framework? ต่างกันอย่างไร?
2. Virtual DOM ช่วยเพิ่ม Performance ได้อย่างไร?
3. Declarative Programming ต่างจาก Imperative อย่างไร?
4. Component-Based Architecture มีประโยชน์อย่างไร?
5. ทำไม Hooks ถึงเปลี่ยนแปลงการเขียน React ไปมาก?

---

## 📚 แหล่งเรียนรู้เพิ่มเติม

- [React Official Docs](https://react.dev) - เอกสารทางการล่าสุด
- [React Beta Docs](https://react.dev/learn) - Interactive Tutorial
- [React GitHub](https://github.com/facebook/react) - Source Code

---

## ➡️ Part ถัดไป

**[Part 02: ติดตั้งและตั้งค่า Development Environment →](./part-02-environment-setup.md)**

ใน Part 02 เราจะติดตั้ง:
- Node.js และ npm
- สร้าง React Project ด้วย Vite
- ทำความรู้จัก VS Code Extensions
- เรียนรู้คำสั่ง npm พื้นฐาน

---

*Part 01 จบแล้ว! ยินดีด้วย! 🎉 Step 1-10 ผ่านไปแล้ว*
