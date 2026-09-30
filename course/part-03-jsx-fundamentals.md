# Part 03: JSX - JavaScript XML พื้นฐาน
## Step 26-45 | ระดับ: พื้นฐาน

---

## 📋 สิ่งที่จะได้เรียนรู้ใน Part นี้

- JSX คืออะไร และมาจากไหน
- JSX vs HTML ต่างกันอย่างไร
- กฎของ JSX ที่ต้องจำ
- การใช้ JavaScript Expressions ใน JSX
- Fragments คืออะไรและเมื่อไหรต้องใช้
- Comments ใน JSX
- Conditional Rendering เบื้องต้น
- JSX และ React.createElement()
- JSX Spread Attributes

---

## Step 26: JSX คืออะไร?

### นิยาม

**JSX (JavaScript XML)** คือ Syntax Extension ของ JavaScript ที่ทำให้เราเขียน HTML-like syntax ในไฟล์ JavaScript ได้

```jsx
// JSX - เขียนแบบ HTML ใน JavaScript
const element = <h1>สวัสดีโลก!</h1>;

// JavaScript ปกติ (ไม่มี JSX)
const element = React.createElement('h1', null, 'สวัสดีโลก!');
```

### JSX ไม่ใช่ HTML จริงๆ!

JSX จะถูก **Compile** (แปลง) เป็น JavaScript ก่อนที่ Browser จะรัน โดย Babel หรือ SWC

```jsx
// ที่เราเขียน (JSX)
function Greeting() {
  return (
    <div className="greeting">
      <h1>สวัสดี!</h1>
      <p>ยินดีต้อนรับ</p>
    </div>
  );
}

// Babel แปลงเป็น (JavaScript จริง)
function Greeting() {
  return React.createElement(
    'div',
    { className: 'greeting' },
    React.createElement('h1', null, 'สวัสดี!'),
    React.createElement('p', null, 'ยินดีต้อนรับ')
  );
}
```

### ทำไมต้องใช้ JSX?

```jsx
// ❌ ไม่ใช้ JSX - อ่านยาก
function Card() {
  return React.createElement(
    'div',
    { className: 'card' },
    React.createElement('h2', null, 'ชื่อสินค้า'),
    React.createElement('p', null, 'รายละเอียด'),
    React.createElement(
      'button',
      { onClick: handleClick },
      'ซื้อเลย'
    )
  );
}

// ✅ ใช้ JSX - อ่านง่ายมาก
function Card() {
  return (
    <div className="card">
      <h2>ชื่อสินค้า</h2>
      <p>รายละเอียด</p>
      <button onClick={handleClick}>ซื้อเลย</button>
    </div>
  );
}
```

---

## Step 27: กฎของ JSX - Return Element เดียว

### กฎข้อที่ 1: JSX ต้อง Return Element เดียว

```jsx
// ❌ ผิด! Return หลาย Element
function Wrong() {
  return (
    <h1>หัวข้อ</h1>  
    <p>เนื้อหา</p>   // Error!
  );
}

// ✅ ถูก! ห่อด้วย Parent Element
function Correct() {
  return (
    <div>
      <h1>หัวข้อ</h1>
      <p>เนื้อหา</p>
    </div>
  );
}

// ✅ ถูก! ใช้ Fragment <>
function AlsoCorrect() {
  return (
    <>
      <h1>หัวข้อ</h1>
      <p>เนื้อหา</p>
    </>
  );
}
```

### ทำไมถึงมีกฎนี้?

```jsx
// JSX ถูกแปลงเป็น React.createElement()
// ซึ่งต้อง Return value เดียวเท่านั้น

// ✅ ถูก (เปรียบเทียบกับ JavaScript)
function getResult() {
  return {  // return object เดียว
    first: 'หนึ่ง',
    second: 'สอง'
  };
}

// ❌ ผิด
function getWrong() {
  return 'หนึ่ง', 'สอง';  // SyntaxError ใน JS ด้วย
}
```

---

## Step 28: Fragments

### Fragment คืออะไร?

**Fragment** คือ Container ที่ไม่สร้าง DOM Element จริงๆ ใช้เมื่อต้องการ Group Elements โดยไม่เพิ่ม div ที่ไม่จำเป็น

```jsx
// ❌ เพิ่ม <div> ที่ไม่จำเป็น
function List() {
  return (
    <div>   {/* div นี้ไม่จำเป็น */}
      <li>รายการที่ 1</li>
      <li>รายการที่ 2</li>
    </div>
  );
}

// HTML ที่ได้:
// <div>  ← div ที่ไม่ต้องการ
//   <li>รายการที่ 1</li>
//   <li>รายการที่ 2</li>
// </div>

// ✅ ใช้ Fragment ไม่เพิ่ม DOM
function List() {
  return (
    <>
      <li>รายการที่ 1</li>
      <li>รายการที่ 2</li>
    </>
  );
}

// HTML ที่ได้:
// <li>รายการที่ 1</li>
// <li>รายการที่ 2</li>
```

### Short Syntax vs Long Syntax

```jsx
// Short Syntax: <>...</>
function Short() {
  return (
    <>
      <p>สั้น</p>
    </>
  );
}

// Long Syntax: <React.Fragment>...</React.Fragment>
// ใช้เมื่อต้องการ key prop (เช่นใน .map())
import { Fragment } from 'react';

function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <Fragment key={item.id}>  {/* ต้องใช้ Long Syntax เพื่อใส่ key */}
          <li>{item.name}</li>
          <li>{item.price}</li>
        </Fragment>
      ))}
    </ul>
  );
}
```

---

## Step 29: JSX vs HTML - ความแตกต่าง

### ความแตกต่างสำคัญ

| HTML | JSX | เหตุผล |
|------|-----|--------|
| `class` | `className` | `class` เป็น Reserved Keyword ใน JS |
| `for` (label) | `htmlFor` | `for` เป็น Reserved Keyword ใน JS |
| `onclick` | `onClick` | JSX ใช้ camelCase |
| `tabindex` | `tabIndex` | camelCase |
| `style="color:red"` | `style={{ color: 'red' }}` | Style เป็น Object |
| `<br>` | `<br />` | JSX ต้อง Self-close |
| `<img>` | `<img />` | JSX ต้อง Self-close |
| `<!-- comment -->` | `{/* comment */}` | Comment syntax ต่างกัน |

### ตัวอย่างครบถ้วน

```jsx
// ❌ HTML Syntax (ผิดใน JSX)
function HtmlWay() {
  return (
    <div class="container">           // ❌ ควรเป็น className
      <label for="name">ชื่อ</label>  // ❌ ควรเป็น htmlFor
      <input 
        type="text" 
        id="name"
        tabindex="1"                  // ❌ ควรเป็น tabIndex
        onclick="handleClick()"       // ❌ ควรเป็น onClick
        style="color: red"            // ❌ ควรเป็น Object
      >                               // ❌ ต้องปิดด้วย />
    </div>
  );
}

// ✅ JSX Syntax (ถูก)
function JsxWay() {
  return (
    <div className="container">
      <label htmlFor="name">ชื่อ</label>
      <input 
        type="text" 
        id="name"
        tabIndex={1}
        onClick={handleClick}
        style={{ color: 'red' }}
      />
    </div>
  );
}
```

### Style ใน JSX

```jsx
// Style ใน HTML
// <div style="color: red; background-color: blue; font-size: 16px">

// Style ใน JSX - ใช้ Object
function StyledComponent() {
  const styles = {
    color: 'red',
    backgroundColor: 'blue',   // camelCase! ไม่ใช่ background-color
    fontSize: '16px',          // camelCase! ไม่ใช่ font-size
    marginTop: '20px',
    padding: '10px 20px',
    borderRadius: '8px'
  };

  return (
    <div style={styles}>
      Styled Text
    </div>
  );
}

// หรือ Inline
function InlineStyled() {
  return (
    <div style={{ 
      color: 'red', 
      fontSize: '24px',
      fontWeight: 'bold'
    }}>
      Inline Styled
    </div>
  );
}
```

---

## Step 30: JavaScript Expressions ใน JSX

### ใช้ `{}` สำหรับ JavaScript

```jsx
function ExpressionDemo() {
  const name = "สมชาย";
  const age = 25;
  const price = 1299.99;

  return (
    <div>
      {/* ✅ ตัวแปร */}
      <p>ชื่อ: {name}</p>
      <p>อายุ: {age}</p>
      
      {/* ✅ การคำนวณ */}
      <p>อายุ + 10 = {age + 10}</p>
      
      {/* ✅ String Methods */}
      <p>ชื่อตัวใหญ่: {name.toUpperCase()}</p>
      
      {/* ✅ ternary operator */}
      <p>{age >= 18 ? 'ผู้ใหญ่' : 'เด็ก'}</p>
      
      {/* ✅ Number Format */}
      <p>ราคา: {price.toFixed(2)} บาท</p>
      <p>ราคา: {new Intl.NumberFormat('th-TH').format(price)} บาท</p>
      
      {/* ✅ Date */}
      <p>วันที่: {new Date().toLocaleDateString('th-TH')}</p>
      
      {/* ✅ Array join */}
      <p>สี: {['แดง', 'น้ำเงิน', 'เขียว'].join(', ')}</p>
    </div>
  );
}
```

### สิ่งที่ใช้ได้และใช้ไม่ได้ใน `{}`

```jsx
function WhatWorks() {
  const arr = [1, 2, 3];
  const obj = { name: 'สมชาย' };
  
  return (
    <div>
      {/* ✅ ใช้ได้ */}
      {"string"}
      {42}
      {3.14}
      {true}
      {null}          {/* แสดงผลเป็น nothing */}
      {undefined}     {/* แสดงผลเป็น nothing */}
      {arr}           {/* แสดง: 123 */}
      {arr.map(n => <span key={n}>{n}</span>)}
      {1 + 2}
      {"Hello " + "World"}
      {condition ? 'A' : 'B'}
      {condition && <p>Shown if true</p>}
      
      {/* ❌ ใช้ไม่ได้! */}
      {/* {if (true) { return 'hello' }}  ← SyntaxError */}
      {/* {{name: 'สมชาย'}}  ← Objects ไม่แสดงผล (Error) */}
      {/* {for (let i=0; i<3; i++) {}}  ← SyntaxError */}
    </div>
  );
}
```

### ความระวัง: Object ใน JSX

```jsx
function ObjectWarning() {
  const user = { name: 'สมชาย', age: 25 };
  
  return (
    <div>
      {/* ❌ Object ใช้โดยตรงไม่ได้ */}
      {/* {user}  → Error: Objects are not valid as a React child */}
      
      {/* ✅ ต้อง Access Property */}
      <p>{user.name}</p>
      <p>{user.age}</p>
      
      {/* ✅ หรือ Convert เป็น String */}
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
}
```

---

## Step 31: Boolean, Null, Undefined ใน JSX

### Falsy Values ใน JSX

```jsx
function FalsyValues() {
  return (
    <div>
      {/* ไม่แสดงผล (Render nothing) */}
      {false}
      {null}
      {undefined}
      
      {/* ⚠️ แสดงผลเป็น text "0"! */}
      {0}
      
      {/* ⚠️ แสดงผลเป็น text ""! */}
      {""}
    </div>
  );
}
```

### กับดักของ `&&` Operator

```jsx
function PitfallDemo({ count, items }) {
  return (
    <div>
      {/* ⚠️ กับดัก! ถ้า count = 0 จะแสดง "0" ไม่ใช่ Nothing */}
      {count && <p>มี {count} รายการ</p>}
      
      {/* ✅ แก้โดย Convert เป็น Boolean */}
      {count > 0 && <p>มี {count} รายการ</p>}
      {!!count && <p>มี {count} รายการ</p>}
      {Boolean(count) && <p>มี {count} รายการ</p>}
      
      {/* ✅ หรือใช้ Ternary */}
      {count ? <p>มี {count} รายการ</p> : null}
      
      {/* ⚠️ กับดักกับ Array */}
      {items.length && <ul>...</ul>}  {/* ถ้า items = [] แสดง "0" */}
      
      {/* ✅ แก้ */}
      {items.length > 0 && <ul>...</ul>}
    </div>
  );
}
```

---

## Step 32: Conditional Rendering พื้นฐาน

### วิธีต่างๆ ในการแสดงผลแบบมีเงื่อนไข

```jsx
function ConditionalDemo({ isLoggedIn, isAdmin, age, user }) {
  
  // วิธีที่ 1: if/else นอก JSX
  let content;
  if (isLoggedIn) {
    content = <p>ยินดีต้อนรับกลับมา!</p>;
  } else {
    content = <p>กรุณาเข้าสู่ระบบ</p>;
  }
  
  return (
    <div>
      {/* วิธีที่ 1: ตัวแปร */}
      {content}
      
      {/* วิธีที่ 2: Ternary Operator */}
      {isLoggedIn 
        ? <p>ยินดีต้อนรับ!</p> 
        : <p>กรุณา Login</p>
      }
      
      {/* วิธีที่ 3: && (เมื่อต้องการแสดงหรือไม่แสดง) */}
      {isAdmin && <button>จัดการระบบ</button>}
      
      {/* วิธีที่ 4: || (Default/Fallback) */}
      <p>{user.name || 'ไม่ระบุชื่อ'}</p>
      
      {/* วิธีที่ 5: Nullish Coalescing ?? */}
      <p>{user.bio ?? 'ไม่มีชีวประวัติ'}</p>
      
      {/* วิธีที่ 6: Nested Ternary (หลีกเลี่ยงถ้าเป็นไปได้) */}
      <p>
        {age < 13 ? 'เด็ก' : age < 18 ? 'วัยรุ่น' : 'ผู้ใหญ่'}
      </p>
    </div>
  );
}
```

---

## Step 33: Lists ใน JSX

### แสดงผล Array

```jsx
function FruitList() {
  const fruits = ['แอปเปิ้ล', 'มะม่วง', 'กล้วย', 'ส้ม'];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}

// ตัวอย่างที่ดีกว่า: ใช้ ID เป็น key
function UserList() {
  const users = [
    { id: 1, name: 'สมชาย', email: 'somchai@email.com' },
    { id: 2, name: 'สมหญิง', email: 'somying@email.com' },
    { id: 3, name: 'สมศักดิ์', email: 'somsak@email.com' },
  ];
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}
```

> 💡 **Key**: ทุกครั้งที่ใช้ `.map()` ต้องใส่ `key` prop - React ใช้มันในการระบุว่า Element ไหนเปลี่ยนแปลง

---

## Step 34: Comments ใน JSX

```jsx
function CommentDemo() {
  // ✅ JavaScript Comment ปกติ (นอก JSX)
  const title = "หัวข้อ";
  
  return (
    <div>
      {/* ✅ JSX Comment - ใช้อยู่ในนี้ */}
      <h1>{title}</h1>
      
      {/* 
        ✅ Multi-line JSX Comment
        สามารถเขียนได้หลายบรรทัด
      */}
      <p>เนื้อหา</p>
      
      {/* ❌ HTML Comment ใช้ไม่ได้ใน JSX! */}
      {/* <!-- นี่จะ Error --> */}
    </div>
  );
}
```

---

## Step 35: Self-Closing Tags

### Tags ที่ต้อง Self-Close ใน JSX

```jsx
function SelfClosingDemo() {
  return (
    <div>
      {/* ✅ JSX ต้อง Self-close สำหรับ Empty Elements */}
      <br />
      <hr />
      <img src="photo.jpg" alt="รูปภาพ" />
      <input type="text" />
      <link rel="stylesheet" href="style.css" />
      <meta charset="UTF-8" />
      
      {/* ✅ Custom Components ก็ Self-close ได้ */}
      <MyComponent />
      <Icon size={24} />
      
      {/* ❌ ใน HTML ไม่ต้อง แต่ JSX ต้อง! */}
      {/* <br>     ← Error ใน JSX */}
      {/* <img src="">  ← Error ใน JSX */}
    </div>
  );
}
```

---

## Step 36: JSX Spread Attributes

### Spread Props

```jsx
function Button({ className, onClick, children, ...rest }) {
  return (
    <button 
      className={`btn ${className}`} 
      onClick={onClick}
      {...rest}  // ส่ง props ที่เหลือทั้งหมด
    >
      {children}
    </button>
  );
}

// ใช้งาน
function App() {
  const buttonProps = {
    type: "submit",
    disabled: false,
    "aria-label": "Submit form"
  };
  
  return (
    <Button 
      className="primary" 
      onClick={handleSubmit}
      {...buttonProps}  // Spread Object เข้าไป
    >
      บันทึก
    </Button>
  );
}
```

### ระวังการ Spread

```jsx
// ⚠️ ระวัง! Spread Object ทั้งหมดอาจส่ง Props ที่ไม่ต้องการ
function SafeComponent({ validProp, ...rest }) {
  // กรอง Props ที่ไม่ต้องการออก
  const { dangerousProp, ...safeRest } = rest;
  
  return <div {...safeRest}>{validProp}</div>;
}
```

---

## Step 37: JSX ใน Variables และ Functions

### เก็บ JSX ในตัวแปร

```jsx
function App() {
  const heading = <h1>หัวข้อหลัก</h1>;
  const description = <p>รายละเอียดของเนื้อหา</p>;
  
  const renderButton = (text, onClick) => (
    <button onClick={onClick}>{text}</button>
  );
  
  return (
    <div>
      {heading}
      {description}
      {renderButton('คลิก', () => console.log('clicked'))}
    </div>
  );
}
```

### Render Functions

```jsx
function ProductPage({ products, isLoading, error }) {
  
  const renderLoading = () => (
    <div className="loading">
      <p>กำลังโหลด...</p>
    </div>
  );
  
  const renderError = (message) => (
    <div className="error">
      <p>เกิดข้อผิดพลาด: {message}</p>
    </div>
  );
  
  const renderProducts = (products) => (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
  
  if (isLoading) return renderLoading();
  if (error) return renderError(error.message);
  return renderProducts(products);
}
```

---

## Step 38: HTML Entities ใน JSX

```jsx
function EntitiesDemo() {
  return (
    <div>
      {/* HTML Entities */}
      <p>&copy; 2024 My Company</p>      {/* © */}
      <p>&amp; เครื่องหมาย &amp;</p>    {/* & */}
      <p>&lt;div&gt;</p>                  {/* <div> */}
      <p>ราคา &nbsp; 100 &nbsp; บาท</p>  {/* Non-breaking spaces */}
      
      {/* หรือใช้ Unicode */}
      <p>&#169; 2024</p>   {/* © */}
      <p>{'©'} 2024</p>  {/* © ด้วย JavaScript */}
      
      {/* ✅ แนะนำ: ใช้ Unicode String */}
      <p>{'© 2024 My Company'}</p>
    </div>
  );
}
```

---

## Step 39: Multiline JSX

### Parentheses สำหรับ Multiline

```jsx
// ❌ ผิด! หาก return บรรทัดเดียวกับ JSX
function Wrong() {
  return   // JavaScript ใส่ ; หลัง return อัตโนมัติ!
    <div>
      <p>ไม่ได้แสดง</p>
    </div>
  // จริงๆ แล้ว return undefined!
}

// ✅ ถูก! ใช้ Parentheses
function Correct() {
  return (
    <div>
      <p>แสดงผลถูกต้อง</p>
    </div>
  );
}

// ✅ ถูก! Return บรรทัดเดียวกัน
function AlsoCorrect() {
  return <div><p>บรรทัดเดียว</p></div>;
}
```

---

## Step 40: JSX และ TypeScript

### JSX กับ TypeScript (.tsx)

```tsx
// ใช้ไฟล์ .tsx สำหรับ TypeScript + JSX

interface UserCardProps {
  name: string;
  age: number;
  email?: string;  // Optional prop
}

function UserCard({ name, age, email }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>อายุ: {age}</p>
      {email && <p>อีเมล: {email}</p>}
    </div>
  );
}

// ใช้งาน
function App() {
  return (
    <UserCard 
      name="สมชาย"
      age={25}
      email="somchai@email.com"
    />
  );
}
```

---

## Step 41: Event Handling เบื้องต้นใน JSX

```jsx
function EventDemo() {
  
  const handleClick = () => {
    console.log('คลิกแล้ว!');
  };
  
  const handleChange = (event) => {
    console.log('ค่าที่พิมพ์:', event.target.value);
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();  // ป้องกัน Form Reload
    console.log('ส่ง Form แล้ว!');
  };
  
  return (
    <div>
      {/* Click Event */}
      <button onClick={handleClick}>คลิกที่นี่</button>
      
      {/* Arrow Function Inline */}
      <button onClick={() => alert('สวัสดี!')}>
        คลิกเพื่อ Alert
      </button>
      
      {/* Change Event */}
      <input 
        type="text" 
        onChange={handleChange}
        placeholder="พิมพ์อะไรก็ได้"
      />
      
      {/* Form Submit */}
      <form onSubmit={handleSubmit}>
        <input type="text" />
        <button type="submit">ส่ง</button>
      </form>
    </div>
  );
}
```

---

## Step 42: Render Children

### Children Props

```jsx
// Component รับ Children
function Card({ title, children }) {
  return (
    <div className="card">
      <h2 className="card-title">{title}</h2>
      <div className="card-body">
        {children}  {/* แสดงผลสิ่งที่อยู่ระหว่าง Tag */}
      </div>
    </div>
  );
}

// ใช้งาน - ส่ง Children ผ่าน Nesting
function App() {
  return (
    <div>
      <Card title="สินค้าแนะนำ">
        <img src="product.jpg" alt="สินค้า" />
        <p>รายละเอียดสินค้า</p>
        <button>เพิ่มลงตะกร้า</button>
      </Card>
      
      <Card title="โปรโมชั่น">
        <p>ลด 50%!</p>
      </Card>
    </div>
  );
}
```

---

## Step 43: JSX Best Practices

### แนวทางที่ดีในการเขียน JSX

```jsx
// ✅ 1. ตั้งชื่อ Component ด้วย PascalCase
function MyComponent() { ... }  // ✅
function myComponent() { ... }  // ❌

// ✅ 2. ไฟล์ Component ใช้ .jsx หรือ .tsx
// MyComponent.jsx  ✅
// myComponent.js   ❌ (งงว่ามี JSX หรือไม่)

// ✅ 3. ทุก Component ควรมี Props Validation
function Button({ text, onClick, disabled = false, variant = 'primary' }) {
  return (
    <button 
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {text}
    </button>
  );
}

// ✅ 4. Extract Logic ออกจาก JSX
function UserProfile({ user }) {
  // ❌ Logic ยุ่งใน JSX
  // return <p>{user.firstName + ' ' + user.lastName + ' (' + user.age + ')'}</p>;
  
  // ✅ Extract ออกมา
  const fullName = `${user.firstName} ${user.lastName}`;
  const displayText = `${fullName} (${user.age})`;
  
  return <p>{displayText}</p>;
}

// ✅ 5. ใช้ Semantic HTML
function Article({ title, content, date }) {
  return (
    <article>       {/* ใช้ article ไม่ใช่ div */}
      <header>
        <h1>{title}</h1>
        <time dateTime={date}>{date}</time>
      </header>
      <main>
        <p>{content}</p>
      </main>
    </article>
  );
}
```

---

## Step 44: ตัวอย่างโปรเจกต์จริง - Profile Card

```jsx
// src/components/ProfileCard.jsx
function ProfileCard({ user }) {
  const {
    name,
    avatar,
    role,
    bio,
    skills = [],
    socialLinks = {}
  } = user;

  return (
    <article className="profile-card">
      {/* Avatar */}
      <div className="avatar-container">
        <img 
          src={avatar || '/default-avatar.png'} 
          alt={`รูปโปรไฟล์ของ ${name}`}
          className="avatar"
        />
        <span className={`status-badge ${user.isOnline ? 'online' : 'offline'}`}>
          {user.isOnline ? 'ออนไลน์' : 'ออฟไลน์'}
        </span>
      </div>
      
      {/* User Info */}
      <div className="user-info">
        <h2 className="user-name">{name}</h2>
        {role && <p className="user-role">{role}</p>}
        {bio && <p className="user-bio">{bio}</p>}
      </div>
      
      {/* Skills */}
      {skills.length > 0 && (
        <div className="skills">
          <h3>ทักษะ</h3>
          <ul className="skill-list">
            {skills.map(skill => (
              <li key={skill} className="skill-tag">
                {skill}
              </li>
            ))}
          </ul>
        </div>
      )}
      
      {/* Social Links */}
      <div className="social-links">
        {socialLinks.github && (
          <a 
            href={socialLinks.github} 
            target="_blank" 
            rel="noopener noreferrer"
            aria-label="GitHub Profile"
          >
            GitHub
          </a>
        )}
        {socialLinks.linkedin && (
          <a 
            href={socialLinks.linkedin}
            target="_blank"
            rel="noopener noreferrer"
            aria-label="LinkedIn Profile"
          >
            LinkedIn
          </a>
        )}
      </div>
    </article>
  );
}

export default ProfileCard;
```

```jsx
// src/App.jsx - ใช้งาน ProfileCard
import ProfileCard from './components/ProfileCard';

function App() {
  const user = {
    name: "สมชาย ใจดี",
    avatar: "https://i.pravatar.cc/150?img=1",
    role: "Frontend Developer",
    bio: "ชอบเขียน React และดื่มกาแฟ ☕",
    skills: ["React", "TypeScript", "Node.js", "CSS"],
    isOnline: true,
    socialLinks: {
      github: "https://github.com/somchai",
      linkedin: "https://linkedin.com/in/somchai"
    }
  };

  return (
    <div style={{ 
      display: 'flex', 
      justifyContent: 'center', 
      padding: '40px' 
    }}>
      <ProfileCard user={user} />
    </div>
  );
}

export default App;
```

---

## Step 45: สรุปและ Quiz

### สรุปกฎ JSX สำคัญ

```
1. ✅ Return Element เดียว (หรือใช้ Fragment)
2. ✅ ปิด Tag ทุกอัน (รวม Self-closing <br />)
3. ✅ ใช้ className แทน class
4. ✅ ใช้ htmlFor แทน for
5. ✅ Event ใช้ camelCase (onClick, onChange)
6. ✅ Style ใช้ Object (style={{ color: 'red' }})
7. ✅ JavaScript ใน {} (เฉพาะ Expression ไม่ใช่ Statement)
8. ✅ Comment ใช้ {/* */}
9. ✅ Key prop ทุกครั้งที่ใช้ .map()
10. ✅ Component ชื่อขึ้นต้นด้วยตัวใหญ่ (PascalCase)
```

### Quiz

1. ผิดหรือถูก? `<div class="container">` เป็น JSX ที่ถูกต้อง
2. เมื่อไหรต้องใช้ Fragment?
3. ทำไม `{0 && <p>Text</p>}` ถึงแสดง "0" ไม่ใช่ Nothing?
4. Difference ระหว่าง `null` และ `undefined` ใน JSX คืออะไร?
5. ทำไมต้องใส่ `key` ใน `.map()`?

### คำตอบ Quiz

```
1. ❌ ผิด! ต้องใช้ className="container"
2. เมื่อต้องการ Group Elements โดยไม่เพิ่ม DOM Node
3. เพราะ 0 เป็น Falsy แต่ไม่ใช่ false/null/undefined
   React render ค่าตัวเลขออกมา
4. ทั้งคู่ไม่แสดงผล แต่ null ชัดเจนกว่าในเชิง Intent
5. React ใช้ key ในการ Track ว่า Element ไหนเปลี่ยน
   ทำให้ Re-render ได้ถูกต้องและมีประสิทธิภาพ
```

---

## ➡️ Part ถัดไป

**[Part 04: Components พื้นฐาน →](./part-04-components-basics.md)**

ใน Part 04 เราจะเรียนรู้:
- Function Components vs Class Components
- Component Props ลึกขึ้น
- Component Composition
- Exporting/Importing Components
- สร้าง UI Component Library เล็กๆ

---

*Part 03 จบแล้ว! 🎉 Step 26-45 ผ่านไปแล้ว*
