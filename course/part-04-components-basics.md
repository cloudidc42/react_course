# Part 04: Components พื้นฐาน
## Step 46-70 | ระดับ: พื้นฐาน

---

## 📋 สิ่งที่จะได้เรียนรู้ใน Part นี้

- Function Components vs Class Components
- การสร้างและ Export Component
- Props คืออะไรและวิธีใช้
- Default Props
- Component Composition
- สร้าง UI Component Library เล็กๆ
- Reusability ของ Components
- Component Naming Conventions

---

## Step 46: Function Components

### Function Component คืออะไร?

Function Component คือ **JavaScript Function** ที่:
1. รับ **props** เป็น Argument
2. Return **JSX**

```jsx
// รูปแบบ Function Component พื้นฐาน
function ComponentName(props) {
  return (
    <div>
      {/* JSX ที่จะแสดงผล */}
    </div>
  );
}

// หรือแบบ Arrow Function
const ComponentName = (props) => {
  return (
    <div>
      {/* JSX ที่จะแสดงผล */}
    </div>
  );
};

// หรือแบบ Arrow Function Short (ถ้า return บรรทัดเดียว)
const ComponentName = (props) => (
  <div>{props.text}</div>
);
```

### ตัวอย่าง Function Components หลากหลาย

```jsx
// 1. Component ง่ายๆ
function Hello() {
  return <h1>สวัสดี!</h1>;
}

// 2. Component รับ Props
function Greeting({ name }) {
  return <h1>สวัสดี, {name}!</h1>;
}

// 3. Component ที่มี Logic
function TemperatureConverter({ celsius }) {
  const fahrenheit = (celsius * 9/5) + 32;
  const kelvin = celsius + 273.15;
  
  return (
    <div>
      <p>Celsius: {celsius}°C</p>
      <p>Fahrenheit: {fahrenheit.toFixed(1)}°F</p>
      <p>Kelvin: {kelvin.toFixed(1)}K</p>
    </div>
  );
}

// 4. Component แบบ Arrow Function
const StatusBadge = ({ status }) => {
  const colors = {
    active: 'green',
    inactive: 'red',
    pending: 'orange'
  };
  
  return (
    <span style={{ 
      color: 'white', 
      backgroundColor: colors[status] || 'gray',
      padding: '4px 8px',
      borderRadius: '4px'
    }}>
      {status}
    </span>
  );
};
```

---

## Step 47: Class Components (Legacy)

> ⚠️ **หมายเหตุ**: Class Components เป็น Legacy ปัจจุบันใช้ Function Components แทน แต่ควรรู้เพื่อทำงานกับโค้ดเก่า

### Class Component Syntax

```jsx
import React, { Component } from 'react';

class ClassComponent extends Component {
  // Constructor สำหรับ Initial State
  constructor(props) {
    super(props);  // ต้อง call super(props) เสมอ
    this.state = {
      count: 0
    };
  }
  
  // Methods
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }
  
  // Lifecycle Methods
  componentDidMount() {
    console.log('Component mounted');
  }
  
  componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
      console.log('Count changed to:', this.state.count);
    }
  }
  
  componentWillUnmount() {
    console.log('Component unmounting');
  }
  
  // ต้อง implement render()
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>+1</button>
      </div>
    );
  }
}

export default ClassComponent;
```

### เปรียบเทียบ Class vs Function

```jsx
// Class Component
class Counter extends Component {
  state = { count: 0 };
  
  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          +1
        </button>
      </div>
    );
  }
}

// Function Component (แนะนำ ✅)
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

---

## Step 48: Export และ Import Components

### Named Export vs Default Export

```jsx
// ===== Default Export =====
// ไฟล์: Button.jsx
function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}

export default Button;  // Default Export

// Import:
import Button from './Button';           // ชื่ออะไรก็ได้
import MyButton from './Button';         // เปลี่ยนชื่อได้
import Btn from './Button';              // เปลี่ยนชื่อได้


// ===== Named Export =====
// ไฟล์: utils.jsx
export function formatDate(date) {
  return new Date(date).toLocaleDateString('th-TH');
}

export function formatCurrency(amount) {
  return new Intl.NumberFormat('th-TH', {
    style: 'currency',
    currency: 'THB'
  }).format(amount);
}

export const APP_NAME = 'My App';

// Import Named:
import { formatDate, formatCurrency } from './utils';    // ชื่อต้องตรง
import { formatDate as fd } from './utils';              // ใช้ Alias


// ===== ใช้ทั้งสองแบบ =====
// ไฟล์: components.jsx
export function Small() { return <small />; }    // Named
export function Large() { return <large />; }    // Named
export default function Default() { return <div />; }  // Default

// Import ทั้งสอง:
import Default, { Small, Large } from './components';
```

### Best Practice: หนึ่งไฟล์ หนึ่ง Component

```
📁 components/
├── Button.jsx          ← export default Button
├── Input.jsx           ← export default Input
├── Modal.jsx           ← export default Modal
└── index.js            ← Re-export ทั้งหมด (Barrel Export)
```

```js
// components/index.js - Barrel Export
export { default as Button } from './Button';
export { default as Input } from './Input';
export { default as Modal } from './Modal';

// Import แบบสั้น:
import { Button, Input, Modal } from './components';
```

---

## Step 49: Props คืออะไร

### Props (Properties)

**Props** คือข้อมูลที่ส่งจาก Parent Component ไปยัง Child Component

```jsx
// Parent ส่ง Props
function Parent() {
  return (
    <Child 
      name="สมชาย"              // string
      age={25}                  // number
      isActive={true}           // boolean
      hobbies={['ฟุตบอล', 'อ่านหนังสือ']}  // array
      address={{ city: 'กรุงเทพ' }}         // object
      onClick={() => console.log('คลิก!')}  // function
    />
  );
}

// Child รับ Props ผ่าน parameter แรก
function Child(props) {
  console.log(props);
  // {
  //   name: "สมชาย",
  //   age: 25,
  //   isActive: true,
  //   hobbies: ["ฟุตบอล", "อ่านหนังสือ"],
  //   address: { city: "กรุงเทพ" },
  //   onClick: [Function]
  // }
  
  return (
    <div>
      <p>ชื่อ: {props.name}</p>
      <p>อายุ: {props.age}</p>
      <p>สถานะ: {props.isActive ? 'ใช้งาน' : 'ไม่ใช้งาน'}</p>
      <button onClick={props.onClick}>คลิก</button>
    </div>
  );
}
```

### Destructuring Props

```jsx
// ❌ แบบยาว
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
    </div>
  );
}

// ✅ Destructuring ใน parameter
function UserCard({ name, email, role }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>{role}</p>
    </div>
  );
}

// ✅ Destructuring ใน function body
function UserCard(props) {
  const { name, email, role } = props;
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}
```

---

## Step 50: Default Props

### กำหนดค่า Default ให้ Props

```jsx
// วิธีที่ 1: Default Parameter Value (แนะนำ ✅)
function Button({ 
  text = 'คลิก', 
  color = 'blue', 
  size = 'medium',
  disabled = false 
}) {
  return (
    <button 
      style={{ color }}
      className={`btn btn-${size}`}
      disabled={disabled}
    >
      {text}
    </button>
  );
}

// ใช้ Default Values
<Button />                          // text="คลิก", color="blue"
<Button text="บันทึก" />            // text="บันทึก", color="blue"
<Button text="ลบ" color="red" />   // text="ลบ", color="red"


// วิธีที่ 2: defaultProps (Legacy)
function Button({ text, color, size }) {
  return (
    <button style={{ color }} className={`btn btn-${size}`}>
      {text}
    </button>
  );
}

Button.defaultProps = {
  text: 'คลิก',
  color: 'blue',
  size: 'medium'
};


// วิธีที่ 3: Nullish Coalescing ใน Component
function Button({ text, color, size }) {
  const btnText = text ?? 'คลิก';
  const btnColor = color ?? 'blue';
  
  return <button style={{ color: btnColor }}>{btnText}</button>;
}
```

---

## Step 51: PropTypes สำหรับ Type Checking

```bash
npm install prop-types
```

```jsx
import PropTypes from 'prop-types';

function UserProfile({ name, age, email, role, hobbies, address, onClick }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>อายุ: {age}</p>
    </div>
  );
}

// กำหนด Type ของ Props
UserProfile.propTypes = {
  name: PropTypes.string.isRequired,        // String, บังคับ
  age: PropTypes.number.isRequired,         // Number, บังคับ
  email: PropTypes.string,                  // String, ไม่บังคับ
  role: PropTypes.oneOf(['admin', 'user', 'moderator']),  // Enum
  hobbies: PropTypes.arrayOf(PropTypes.string),           // Array of strings
  address: PropTypes.shape({                // Object shape
    city: PropTypes.string.isRequired,
    country: PropTypes.string
  }),
  onClick: PropTypes.func                   // Function
};

// กำหนด Default Props
UserProfile.defaultProps = {
  role: 'user',
  hobbies: [],
  email: ''
};
```

> 💡 **TypeScript vs PropTypes**: ถ้าใช้ TypeScript ไม่จำเป็นต้องใช้ PropTypes เพราะ TypeScript ทำ Type Checking ที่ดีกว่า

---

## Step 52: Component Composition

### Composition Pattern

```jsx
// Building Blocks
function Avatar({ src, alt, size = 'medium' }) {
  const sizes = { small: 32, medium: 48, large: 64 };
  
  return (
    <img 
      src={src} 
      alt={alt}
      width={sizes[size]}
      height={sizes[size]}
      style={{ borderRadius: '50%', objectFit: 'cover' }}
    />
  );
}

function Username({ name, verified = false }) {
  return (
    <span>
      {name}
      {verified && ' ✓'}
    </span>
  );
}

function UserStats({ followers, following, posts }) {
  return (
    <div style={{ display: 'flex', gap: '16px' }}>
      <div><strong>{posts}</strong> โพสต์</div>
      <div><strong>{followers}</strong> ผู้ติดตาม</div>
      <div><strong>{following}</strong> กำลังติดตาม</div>
    </div>
  );
}

// Compose เป็น Component ใหญ่
function UserProfile({ user }) {
  return (
    <div className="profile">
      <Avatar src={user.avatar} alt={user.name} size="large" />
      <Username name={user.name} verified={user.verified} />
      <UserStats 
        followers={user.followers}
        following={user.following}
        posts={user.posts}
      />
    </div>
  );
}
```

### Containment Pattern (Children Props)

```jsx
// Generic Container Components
function Card({ children, title, footer }) {
  return (
    <div className="card">
      {title && <div className="card-header">{title}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

function Modal({ children, isOpen, onClose, title }) {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose}>✕</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
}

// ใช้งาน
function App() {
  return (
    <div>
      <Card 
        title="ข้อมูลผู้ใช้"
        footer={<button>บันทึก</button>}
      >
        <p>ชื่อ: สมชาย</p>
        <p>อายุ: 25</p>
      </Card>
      
      <Modal isOpen={true} onClose={() => {}} title="ยืนยันการลบ">
        <p>คุณแน่ใจหรือไม่ที่จะลบรายการนี้?</p>
        <button>ยืนยัน</button>
        <button>ยกเลิก</button>
      </Modal>
    </div>
  );
}
```

---

## Step 53: Specialized Components Pattern

```jsx
// Generic Button
function Button({ 
  children, 
  onClick, 
  variant = 'primary',
  size = 'medium',
  disabled = false,
  isLoading = false,
  ...rest  // รับ props เพิ่มเติม
}) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
      disabled={disabled || isLoading}
      {...rest}
    >
      {isLoading ? (
        <span>
          <span className="spinner" />
          กำลังโหลด...
        </span>
      ) : children}
    </button>
  );
}

// Specialized Buttons (Composed จาก Button)
function PrimaryButton(props) {
  return <Button {...props} variant="primary" />;
}

function DangerButton(props) {
  return <Button {...props} variant="danger" />;
}

function LoadingButton({ isLoading, ...props }) {
  return <Button {...props} isLoading={isLoading} />;
}

function IconButton({ icon, label, ...props }) {
  return (
    <Button {...props} aria-label={label}>
      <span className="icon">{icon}</span>
    </Button>
  );
}

// ใช้งาน
function ActionBar() {
  return (
    <div>
      <PrimaryButton onClick={handleSave}>บันทึก</PrimaryButton>
      <DangerButton onClick={handleDelete}>ลบ</DangerButton>
      <LoadingButton isLoading={isSaving}>กำลังบันทึก</LoadingButton>
      <IconButton icon="🗑️" label="Delete" onClick={handleDelete} />
    </div>
  );
}
```

---

## Step 54: Component สำหรับ Layout

```jsx
// Layout Components
function Container({ children, maxWidth = '1200px', padding = '16px' }) {
  return (
    <div style={{ 
      maxWidth, 
      margin: '0 auto',
      padding: `0 ${padding}`
    }}>
      {children}
    </div>
  );
}

function Grid({ children, columns = 3, gap = '16px' }) {
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: `repeat(${columns}, 1fr)`,
      gap
    }}>
      {children}
    </div>
  );
}

function Flex({ children, direction = 'row', align = 'center', justify = 'flex-start', gap = '8px', wrap = false }) {
  return (
    <div style={{
      display: 'flex',
      flexDirection: direction,
      alignItems: align,
      justifyContent: justify,
      gap,
      flexWrap: wrap ? 'wrap' : 'nowrap'
    }}>
      {children}
    </div>
  );
}

function Stack({ children, spacing = '8px' }) {
  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: spacing }}>
      {children}
    </div>
  );
}

// ใช้งาน
function ProductPage() {
  return (
    <Container>
      <h1>สินค้าของเรา</h1>
      <Grid columns={3} gap="24px">
        <ProductCard />
        <ProductCard />
        <ProductCard />
      </Grid>
    </Container>
  );
}
```

---

## Step 55: Component สำหรับ Form

```jsx
// Reusable Form Components
function FormField({ label, htmlFor, error, required, children }) {
  return (
    <div className="form-field">
      <label htmlFor={htmlFor}>
        {label}
        {required && <span className="required">*</span>}
      </label>
      {children}
      {error && <span className="error-message">{error}</span>}
    </div>
  );
}

function TextInput({ 
  id,
  name,
  value,
  onChange,
  placeholder,
  type = 'text',
  disabled = false,
  error
}) {
  return (
    <input
      id={id}
      name={name}
      type={type}
      value={value}
      onChange={onChange}
      placeholder={placeholder}
      disabled={disabled}
      className={`input ${error ? 'input-error' : ''}`}
      aria-invalid={!!error}
      aria-describedby={error ? `${id}-error` : undefined}
    />
  );
}

function Select({ id, name, value, onChange, options, placeholder, error }) {
  return (
    <select
      id={id}
      name={name}
      value={value}
      onChange={onChange}
      className={`select ${error ? 'select-error' : ''}`}
    >
      {placeholder && (
        <option value="" disabled>{placeholder}</option>
      )}
      {options.map(option => (
        <option key={option.value} value={option.value}>
          {option.label}
        </option>
      ))}
    </select>
  );
}

// ใช้งาน
function LoginForm() {
  return (
    <form>
      <FormField label="อีเมล" htmlFor="email" required>
        <TextInput
          id="email"
          name="email"
          type="email"
          placeholder="email@example.com"
        />
      </FormField>
      
      <FormField label="รหัสผ่าน" htmlFor="password" required>
        <TextInput
          id="password"
          name="password"
          type="password"
          placeholder="รหัสผ่าน"
        />
      </FormField>
      
      <button type="submit">เข้าสู่ระบบ</button>
    </form>
  );
}
```

---

## Step 56: Render Props Pattern

```jsx
// Render Props: ส่ง Function เป็น Props เพื่อ Render
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };
  
  return (
    <div 
      style={{ height: '300px', border: '1px solid gray' }}
      onMouseMove={handleMouseMove}
    >
      {render(position)}  {/* เรียก render function */}
    </div>
  );
}

// ใช้งาน Render Props
function App() {
  return (
    <MouseTracker 
      render={({ x, y }) => (
        <p>Mouse Position: ({x}, {y})</p>
      )}
    />
  );
}

// Children as Render Prop (ดีกว่า)
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  // ...
  return (
    <div onMouseMove={handleMouseMove}>
      {children(position)}
    </div>
  );
}

// ใช้งาน
function App() {
  return (
    <MouseTracker>
      {({ x, y }) => <p>({x}, {y})</p>}
    </MouseTracker>
  );
}
```

---

## Step 57: Higher Order Components (HOC)

```jsx
// HOC: Function ที่รับ Component และ Return Component ใหม่
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return (
        <div className="loading-container">
          <div className="spinner"></div>
          <p>กำลังโหลด...</p>
        </div>
      );
    }
    
    return <WrappedComponent {...props} />;
  };
}

// ใช้งาน HOC
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

// ใช้ใน App
function App() {
  const [isLoading, setIsLoading] = useState(true);
  const [users, setUsers] = useState([]);
  
  return (
    <UserListWithLoading 
      isLoading={isLoading}
      users={users}
    />
  );
}
```

---

## Step 58: Component สำหรับ Feedback UI

```jsx
// Loading Spinner
function Spinner({ size = 'medium', color = '#007bff' }) {
  const sizes = { small: 16, medium: 32, large: 48 };
  
  return (
    <div
      style={{
        width: sizes[size],
        height: sizes[size],
        border: `3px solid #f3f3f3`,
        borderTop: `3px solid ${color}`,
        borderRadius: '50%',
        animation: 'spin 1s linear infinite'
      }}
      role="status"
      aria-label="กำลังโหลด"
    />
  );
}

// Alert/Notification
function Alert({ type = 'info', title, message, onClose }) {
  const styles = {
    info: { bg: '#cce5ff', border: '#b8daff', icon: 'ℹ️' },
    success: { bg: '#d4edda', border: '#c3e6cb', icon: '✅' },
    warning: { bg: '#fff3cd', border: '#ffeeba', icon: '⚠️' },
    error: { bg: '#f8d7da', border: '#f5c6cb', icon: '❌' }
  };
  
  const style = styles[type];
  
  return (
    <div
      role="alert"
      style={{
        backgroundColor: style.bg,
        border: `1px solid ${style.border}`,
        borderRadius: '4px',
        padding: '12px 16px',
        display: 'flex',
        alignItems: 'flex-start',
        gap: '8px'
      }}
    >
      <span role="img" aria-hidden>{style.icon}</span>
      <div>
        {title && <strong>{title}</strong>}
        <p style={{ margin: 0 }}>{message}</p>
      </div>
      {onClose && (
        <button 
          onClick={onClose}
          style={{ marginLeft: 'auto', background: 'none', border: 'none', cursor: 'pointer' }}
          aria-label="ปิด"
        >
          ✕
        </button>
      )}
    </div>
  );
}

// Badge
function Badge({ children, variant = 'default', count }) {
  const variants = {
    default: { bg: '#6c757d', color: 'white' },
    primary: { bg: '#007bff', color: 'white' },
    success: { bg: '#28a745', color: 'white' },
    danger: { bg: '#dc3545', color: 'white' }
  };
  
  const style = variants[variant];
  
  return (
    <span style={{
      backgroundColor: style.bg,
      color: style.color,
      padding: '2px 8px',
      borderRadius: '12px',
      fontSize: '12px',
      fontWeight: 'bold'
    }}>
      {count !== undefined ? count : children}
    </span>
  );
}
```

---

## Step 59: สร้าง Mini Component Library

```jsx
// src/components/ui/index.js - สร้าง Library เล็กๆ

// ===== Button =====
export function Button({
  children,
  variant = 'primary',
  size = 'md',
  onClick,
  disabled = false,
  type = 'button',
  fullWidth = false,
  ...props
}) {
  const baseStyle = {
    display: 'inline-flex',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: '6px',
    fontWeight: '500',
    cursor: disabled ? 'not-allowed' : 'pointer',
    opacity: disabled ? 0.7 : 1,
    width: fullWidth ? '100%' : 'auto',
    border: 'none',
    transition: 'all 0.2s'
  };
  
  const variants = {
    primary: { backgroundColor: '#007bff', color: 'white' },
    secondary: { backgroundColor: '#6c757d', color: 'white' },
    success: { backgroundColor: '#28a745', color: 'white' },
    danger: { backgroundColor: '#dc3545', color: 'white' },
    outline: { backgroundColor: 'transparent', color: '#007bff', border: '1px solid #007bff' },
    ghost: { backgroundColor: 'transparent', color: '#007bff' }
  };
  
  const sizes = {
    sm: { padding: '4px 12px', fontSize: '14px' },
    md: { padding: '8px 16px', fontSize: '16px' },
    lg: { padding: '12px 24px', fontSize: '18px' }
  };
  
  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled}
      style={{ ...baseStyle, ...variants[variant], ...sizes[size] }}
      {...props}
    >
      {children}
    </button>
  );
}

// ===== Input =====
export function Input({
  label,
  id,
  type = 'text',
  value,
  onChange,
  placeholder,
  error,
  disabled = false,
  required = false,
  ...props
}) {
  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: '4px' }}>
      {label && (
        <label htmlFor={id} style={{ fontSize: '14px', fontWeight: '500' }}>
          {label}
          {required && <span style={{ color: 'red' }}> *</span>}
        </label>
      )}
      <input
        id={id}
        type={type}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        disabled={disabled}
        required={required}
        style={{
          padding: '8px 12px',
          borderRadius: '6px',
          border: error ? '1px solid #dc3545' : '1px solid #ced4da',
          fontSize: '16px',
          outline: 'none',
          backgroundColor: disabled ? '#e9ecef' : 'white',
          cursor: disabled ? 'not-allowed' : 'text'
        }}
        {...props}
      />
      {error && <span style={{ fontSize: '12px', color: '#dc3545' }}>{error}</span>}
    </div>
  );
}

// ===== Card =====
export function Card({ children, title, footer, padding = '16px', shadow = true }) {
  return (
    <div style={{
      borderRadius: '8px',
      border: '1px solid #dee2e6',
      boxShadow: shadow ? '0 2px 4px rgba(0,0,0,0.1)' : 'none',
      overflow: 'hidden',
      backgroundColor: 'white'
    }}>
      {title && (
        <div style={{ 
          padding, 
          borderBottom: '1px solid #dee2e6',
          fontWeight: '600',
          fontSize: '18px'
        }}>
          {title}
        </div>
      )}
      <div style={{ padding }}>{children}</div>
      {footer && (
        <div style={{ 
          padding, 
          borderTop: '1px solid #dee2e6',
          backgroundColor: '#f8f9fa'
        }}>
          {footer}
        </div>
      )}
    </div>
  );
}
```

---

## Step 60: Workshop - สร้าง Product Card

```jsx
// ===== Workshop: สร้าง E-commerce Product Card =====

// src/components/ProductCard.jsx
function ProductCard({ product, onAddToCart, onWishlist }) {
  const {
    id,
    name,
    price,
    originalPrice,
    image,
    category,
    rating,
    reviewCount,
    inStock,
    badge
  } = product;
  
  const discount = originalPrice 
    ? Math.round((1 - price / originalPrice) * 100) 
    : 0;
  
  const formatPrice = (amount) => 
    new Intl.NumberFormat('th-TH', {
      style: 'currency',
      currency: 'THB'
    }).format(amount);
  
  const renderStars = (rating) => {
    return Array.from({ length: 5 }, (_, i) => (
      <span key={i} style={{ color: i < rating ? '#ffc107' : '#dee2e6' }}>
        ★
      </span>
    ));
  };
  
  return (
    <article 
      className="product-card"
      style={{
        borderRadius: '12px',
        border: '1px solid #dee2e6',
        overflow: 'hidden',
        boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
        transition: 'transform 0.2s, box-shadow 0.2s',
        backgroundColor: 'white',
        position: 'relative'
      }}
    >
      {/* Badge */}
      {badge && (
        <span style={{
          position: 'absolute',
          top: '12px',
          left: '12px',
          backgroundColor: badge === 'NEW' ? '#28a745' : '#dc3545',
          color: 'white',
          padding: '4px 8px',
          borderRadius: '4px',
          fontSize: '12px',
          fontWeight: 'bold',
          zIndex: 1
        }}>
          {badge}
        </span>
      )}
      
      {/* Discount Badge */}
      {discount > 0 && (
        <span style={{
          position: 'absolute',
          top: '12px',
          right: '12px',
          backgroundColor: '#ff4444',
          color: 'white',
          padding: '4px 8px',
          borderRadius: '4px',
          fontSize: '12px',
          fontWeight: 'bold'
        }}>
          -{discount}%
        </span>
      )}
      
      {/* Product Image */}
      <div style={{ position: 'relative', paddingBottom: '75%', overflow: 'hidden' }}>
        <img 
          src={image} 
          alt={name}
          style={{
            position: 'absolute',
            inset: 0,
            width: '100%',
            height: '100%',
            objectFit: 'cover'
          }}
          loading="lazy"
        />
      </div>
      
      {/* Product Info */}
      <div style={{ padding: '16px' }}>
        <p style={{ 
          fontSize: '12px', 
          color: '#6c757d', 
          margin: '0 0 4px',
          textTransform: 'uppercase',
          letterSpacing: '0.5px'
        }}>
          {category}
        </p>
        
        <h3 style={{ 
          margin: '0 0 8px',
          fontSize: '16px',
          fontWeight: '600',
          lineHeight: '1.4',
          display: '-webkit-box',
          WebkitLineClamp: 2,
          WebkitBoxOrient: 'vertical',
          overflow: 'hidden'
        }}>
          {name}
        </h3>
        
        {/* Rating */}
        <div style={{ display: 'flex', alignItems: 'center', gap: '4px', marginBottom: '12px' }}>
          <div>{renderStars(Math.round(rating))}</div>
          <span style={{ fontSize: '14px', color: '#6c757d' }}>
            {rating.toFixed(1)} ({reviewCount})
          </span>
        </div>
        
        {/* Price */}
        <div style={{ marginBottom: '16px' }}>
          <span style={{ 
            fontSize: '20px', 
            fontWeight: 'bold',
            color: discount > 0 ? '#dc3545' : '#212529'
          }}>
            {formatPrice(price)}
          </span>
          {originalPrice && (
            <span style={{ 
              fontSize: '14px', 
              color: '#6c757d',
              textDecoration: 'line-through',
              marginLeft: '8px'
            }}>
              {formatPrice(originalPrice)}
            </span>
          )}
        </div>
        
        {/* Actions */}
        <div style={{ display: 'flex', gap: '8px' }}>
          <button
            onClick={() => onAddToCart(product)}
            disabled={!inStock}
            style={{
              flex: 1,
              padding: '10px',
              backgroundColor: inStock ? '#007bff' : '#6c757d',
              color: 'white',
              border: 'none',
              borderRadius: '6px',
              cursor: inStock ? 'pointer' : 'not-allowed',
              fontWeight: '500',
              fontSize: '14px'
            }}
          >
            {inStock ? '🛒 เพิ่มลงตะกร้า' : 'สินค้าหมด'}
          </button>
          
          <button
            onClick={() => onWishlist(product)}
            aria-label="เพิ่มในรายการโปรด"
            style={{
              padding: '10px 14px',
              backgroundColor: 'transparent',
              border: '1px solid #dee2e6',
              borderRadius: '6px',
              cursor: 'pointer',
              fontSize: '18px'
            }}
          >
            ♡
          </button>
        </div>
      </div>
    </article>
  );
}

export default ProductCard;
```

```jsx
// src/App.jsx - ใช้งาน ProductCard
import ProductCard from './components/ProductCard';

const products = [
  {
    id: 1,
    name: 'หูฟัง Sony WH-1000XM5 Wireless Noise Canceling',
    price: 10990,
    originalPrice: 13990,
    image: 'https://picsum.photos/300/225?random=1',
    category: 'Electronics',
    rating: 4.8,
    reviewCount: 2341,
    inStock: true,
    badge: 'HOT'
  },
  {
    id: 2,
    name: 'กระเป๋าเป้ Laptop Backpack 15.6"',
    price: 1290,
    originalPrice: null,
    image: 'https://picsum.photos/300/225?random=2',
    category: 'Accessories',
    rating: 4.5,
    reviewCount: 892,
    inStock: true,
    badge: 'NEW'
  },
  {
    id: 3,
    name: 'คีย์บอร์ด Mechanical Gaming Keyboard RGB',
    price: 3499,
    originalPrice: 4999,
    image: 'https://picsum.photos/300/225?random=3',
    category: 'Gaming',
    rating: 4.3,
    reviewCount: 567,
    inStock: false
  }
];

function App() {
  const handleAddToCart = (product) => {
    console.log('เพิ่มลงตะกร้า:', product.name);
    alert(`เพิ่ม "${product.name}" ลงตะกร้าแล้ว!`);
  };
  
  const handleWishlist = (product) => {
    console.log('เพิ่มใน Wishlist:', product.name);
  };
  
  return (
    <div style={{ padding: '40px' }}>
      <h1>สินค้าแนะนำ</h1>
      <div style={{
        display: 'grid',
        gridTemplateColumns: 'repeat(auto-fill, minmax(280px, 1fr))',
        gap: '24px'
      }}>
        {products.map(product => (
          <ProductCard
            key={product.id}
            product={product}
            onAddToCart={handleAddToCart}
            onWishlist={handleWishlist}
          />
        ))}
      </div>
    </div>
  );
}

export default App;
```

---

## Step 61-70: สรุปและแนวทางปฏิบัติที่ดี

### Component Best Practices

```jsx
// ✅ 1. Single Responsibility: แต่ละ Component ทำหน้าที่เดียว
// ❌ ผิด
function UserDashboard() {
  // ทำทุกอย่างใน Component เดียว - ยากต่อการ Maintain
  return <div>/* ... navigation + profile + posts + settings */</div>;
}

// ✅ ถูก
function UserDashboard() {
  return (
    <div>
      <Navigation />      {/* รับผิดชอบ Nav เท่านั้น */}
      <UserProfile />     {/* รับผิดชอบ Profile เท่านั้น */}
      <PostFeed />        {/* รับผิดชอบ Posts เท่านั้น */}
    </div>
  );
}

// ✅ 2. Props ควรเป็น Plain Values
// ❌ หลีกเลี่ยง
function BadComponent({ user }) {
  // user.getName() ← เรียก method ใน Component
  return <p>{user.getName()}</p>;
}

// ✅ ดีกว่า
function GoodComponent({ name }) {
  return <p>{name}</p>;
}

// ✅ 3. ไม่แก้ไข Props โดยตรง
function WrongChild({ items }) {
  items.push('new item');  // ❌ ห้ามแก้ไข Props!
  return <ul>{/* ... */}</ul>;
}

function CorrectChild({ items }) {
  const newItems = [...items, 'new item'];  // ✅ สร้าง Copy ใหม่
  return <ul>{/* ... */}</ul>;
}

// ✅ 4. ตั้งชื่อ Props ให้ชัดเจน
// ❌ ชื่อไม่ชัดเจน
function BadNaming({ data, fn, val, f }) { ... }

// ✅ ชัดเจน
function GoodNaming({ userData, onSubmit, defaultValue, formatFn }) { ... }

// ✅ 5. ใช้ TypeScript หรือ PropTypes เพื่อ Type Safety
interface ButtonProps {
  label: string;
  onClick: () => void;
  isDisabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
}

function TypeSafeButton({ label, onClick, isDisabled = false, variant = 'primary' }: ButtonProps) {
  return <button onClick={onClick} disabled={isDisabled}>{label}</button>;
}
```

### เมื่อไหรควรแยก Component

```
แยก Component เมื่อ:
1. 📏 Component มี JSX ยาวเกิน 50-100 บรรทัด
2. 🔄 ส่วนนั้น Reuse ได้ที่อื่น
3. 🎯 ส่วนนั้นมี State หรือ Logic เป็นของตัวเอง
4. 👥 ต้องการแบ่งงานกันในทีม
5. 🧪 ต้องการ Test แยกส่วน

ไม่ต้องแยก Component เมื่อ:
1. 📌 ใช้ที่เดียว และไม่ซับซ้อน
2. 🔗 Props ที่ต้องส่งมีมากเกินไปถ้าแยก
3. 🏃 Premature Optimization
```

---

## ➡️ Part ถัดไป

**[Part 05: Props และการส่งข้อมูล →](./part-05-props-and-data-flow.md)**

ใน Part 05 เราจะเรียนรู้:
- Props ลึกขึ้น (ทุกประเภท)
- Prop Drilling และปัญหา
- Children Props ขั้นสูง
- Lifting State Up
- Callback Props Pattern

---

*Part 04 จบแล้ว! 🎉 Step 46-70 ผ่านไปแล้ว*
