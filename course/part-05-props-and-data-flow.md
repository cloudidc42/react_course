# Part 05: Props และการส่งข้อมูล (Props and Data Flow)
## Step 71-95 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 71: Props คืออะไร?](#step-71)
- [Step 72: การส่ง Props พื้นฐาน](#step-72)
- [Step 73: Props ประเภท String](#step-73)
- [Step 74: Props ประเภท Number](#step-74)
- [Step 75: Props ประเภท Boolean](#step-75)
- [Step 76: Props ประเภท Array](#step-76)
- [Step 77: Props ประเภท Object](#step-77)
- [Step 78: Props ประเภท Function](#step-78)
- [Step 79: Props ประเภท JSX/ReactNode](#step-79)
- [Step 80: Default Props](#step-80)
- [Step 81: Destructuring Props](#step-81)
- [Step 82: Spread Operator กับ Props](#step-82)
- [Step 83: PropTypes Validation](#step-83)
- [Step 84: Children Props พื้นฐาน](#step-84)
- [Step 85: Children Props ขั้นสูง](#step-85)
- [Step 86: Render Props Pattern](#step-86)
- [Step 87: Prop Drilling คืออะไร](#step-87)
- [Step 88: ปัญหาของ Prop Drilling](#step-88)
- [Step 89: Data Flow ใน React](#step-89)
- [Step 90: Lifting State Up คืออะไร](#step-90)
- [Step 91: Lifting State Up ตัวอย่าง](#step-91)
- [Step 92: Callback Props Pattern](#step-92)
- [Step 93: Controlled Components ผ่าน Props](#step-93)
- [Step 94: Props vs State](#step-94)
- [Step 95: แนวทางปฏิบัติที่ดี (Best Practices)](#step-95)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 71: Props คืออะไร? {#step-71}

**Props** (Properties) คือกลไกหลักในการส่งข้อมูลจาก Component แม่ไปยัง Component ลูกใน React

### แนวคิดพื้นฐาน

Props ทำงานคล้ายกับ **arguments** ของฟังก์ชัน - คุณส่งข้อมูลเข้าไปใน Component และ Component นั้นจะใช้ข้อมูลนั้นในการแสดงผล

```jsx
// Component คือฟังก์ชันที่รับ props
function Greeting(props) {
  return <h1>สวัสดี, {props.name}!</h1>;
}

// การใช้งาน - ส่ง prop ชื่อ "name"
function App() {
  return <Greeting name="สมชาย" />;
}
```

### กฎสำคัญของ Props

1. **Props ไหลทางเดียว (One-way data flow)** - จากบนลงล่างเท่านั้น
2. **Props เป็น Read-only** - Component ลูกไม่สามารถแก้ไข props ได้
3. **Props คือ Object** - React รวม attributes ทั้งหมดเข้าเป็น object เดียว

```jsx
// ❌ ผิด - ห้ามแก้ไข props
function BadComponent(props) {
  props.name = "ชื่อใหม่"; // Error!
  return <h1>{props.name}</h1>;
}

// ✅ ถูก - อ่านได้อย่างเดียว
function GoodComponent(props) {
  return <h1>{props.name}</h1>;
}
```

> **Tip:** นึกถึง Props เหมือน parameter ของฟังก์ชัน - คุณส่งค่าเข้าไป แต่ฟังก์ชันไม่ควรแก้ไขค่าต้นฉบับ

---

## Step 72: การส่ง Props พื้นฐาน {#step-72}

### Syntax การส่ง Props

Props ถูกส่งเหมือน HTML attributes แต่สามารถส่ง JavaScript expressions ได้โดยใช้ `{}`

```jsx
// การส่ง props พื้นฐาน
function UserCard({ name, age, city }) {
  return (
    <div className="user-card">
      <h2>{name}</h2>
      <p>อายุ: {age} ปี</p>
      <p>เมือง: {city}</p>
    </div>
  );
}

function App() {
  return (
    <div>
      {/* ส่ง props โดยตรง */}
      <UserCard name="สมหญิง" age={25} city="กรุงเทพ" />
      
      {/* ส่ง props จาก variables */}
      <UserCard name="สมชาย" age={30} city="เชียงใหม่" />
    </div>
  );
}
```

### การรับ Props ทั้งหมด

```jsx
// วิธีที่ 1: รับเป็น object
function Component(props) {
  console.log(props); // { name: "...", age: ..., ... }
  return <div>{props.name}</div>;
}

// วิธีที่ 2: Destructuring (แนะนำ)
function Component({ name, age, city }) {
  return <div>{name} - {age} - {city}</div>;
}

// วิธีที่ 3: Destructuring กับ default values
function Component({ name = "ไม่ระบุ", age = 0 }) {
  return <div>{name} - {age}</div>;
}
```

---

## Step 73: Props ประเภท String {#step-73}

String เป็น prop ประเภทที่ใช้บ่อยที่สุด สามารถส่งได้สองวิธี

```jsx
function Button({ label, color, size }) {
  return (
    <button 
      style={{ backgroundColor: color, fontSize: size }}
    >
      {label}
    </button>
  );
}

function App() {
  return (
    <div>
      {/* วิธีที่ 1: ใช้ quotes (เหมาะกับ string ตรงๆ) */}
      <Button label="คลิกฉัน" color="blue" size="16px" />
      
      {/* วิธีที่ 2: ใช้ {} (เหมาะกับ expression หรือ variable) */}
      <Button label={"บันทึก"} color={"green"} size={"14px"} />
      
      {/* Template literals */}
      <Button label={`สวัสดี ${"World"}`} color="red" size="18px" />
    </div>
  );
}
```

### String Interpolation กับ Props

```jsx
function ProfileHeader({ firstName, lastName, title }) {
  const fullName = `${firstName} ${lastName}`;
  
  return (
    <header>
      <h1>{title} {fullName}</h1>
      <p>ยินดีต้อนรับ, {firstName}!</p>
    </header>
  );
}

// การใช้งาน
<ProfileHeader 
  firstName="สมชาย" 
  lastName="ใจดี" 
  title="คุณ"
/>
```

> **Warning:** อย่าลืมว่า prop ที่เป็น string ใน `{}` ต้องใช้ quotes ข้างใน เช่น `label={"hello"}` แต่ถ้าไม่มี `{}` ใช้แค่ `label="hello"` ได้เลย

---

## Step 74: Props ประเภท Number {#step-74}

Number props ต้องใช้ `{}` เสมอ ไม่สามารถใช้ quotes ได้

```jsx
function ProductCard({ name, price, quantity, rating }) {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>ราคา: ฿{price.toLocaleString()}</p>
      <p>จำนวน: {quantity} ชิ้น</p>
      <p>คะแนน: {rating}/5 ดาว</p>
      
      {/* คำนวณภายใน JSX */}
      <p>ราคารวม: ฿{(price * quantity).toLocaleString()}</p>
    </div>
  );
}

function App() {
  return (
    <div>
      {/* ✅ ถูก - number ใช้ {} */}
      <ProductCard 
        name="หูฟัง Bluetooth"
        price={1299}
        quantity={2}
        rating={4.5}
      />
      
      {/* ❌ ผิด - string ไม่ใช่ number */}
      {/* <ProductCard price="1299" /> */}
    </div>
  );
}
```

### การใช้ Number ใน Calculations

```jsx
function DiscountPrice({ originalPrice, discountPercent }) {
  const discountAmount = originalPrice * (discountPercent / 100);
  const finalPrice = originalPrice - discountAmount;
  
  return (
    <div>
      <p>ราคาเดิม: ฿{originalPrice}</p>
      <p>ส่วนลด: {discountPercent}%</p>
      <p>ราคาหลังลด: ฿{finalPrice.toFixed(2)}</p>
    </div>
  );
}

<DiscountPrice originalPrice={500} discountPercent={20} />
// Output: ราคาหลังลด: ฿400.00
```

---

## Step 75: Props ประเภท Boolean {#step-75}

Boolean props มีรูปแบบพิเศษใน JSX - ถ้าส่งแค่ชื่อ prop โดยไม่มีค่า จะถือว่าเป็น `true`

```jsx
function Alert({ message, isError, isVisible, isDismissible }) {
  if (!isVisible) return null;
  
  return (
    <div 
      className={`alert ${isError ? 'alert-error' : 'alert-success'}`}
    >
      <p>{message}</p>
      {isDismissible && <button>✕ ปิด</button>}
    </div>
  );
}

function App() {
  return (
    <div>
      {/* isError={true} - เขียนย่อได้เป็น isError */}
      <Alert 
        message="เกิดข้อผิดพลาด!" 
        isError 
        isVisible 
        isDismissible
      />
      
      {/* ส่งค่า false ต้องใช้ {} */}
      <Alert 
        message="บันทึกสำเร็จ!" 
        isError={false} 
        isVisible={true}
        isDismissible={false}
      />
    </div>
  );
}
```

### Boolean Props ที่พบบ่อย

```jsx
// Disabled state
<Button disabled>ปิดการใช้งาน</Button>

// Loading state
<Spinner loading={isLoading} />

// Feature flags
<FeatureComponent enabled={featureFlags.newUI} />

// Conditional behavior
<Modal open={isModalOpen} closeable={!isProcessing} />
```

> **Tip:** ตั้งชื่อ boolean props ด้วย prefix `is`, `has`, `can`, `should` เช่น `isLoading`, `hasError`, `canEdit` เพื่อให้อ่านเข้าใจง่าย

---

## Step 76: Props ประเภท Array {#step-76}

Array props ใช้สำหรับส่งข้อมูลหลายรายการ

```jsx
function TagList({ tags }) {
  return (
    <div className="tag-list">
      {tags.map((tag, index) => (
        <span key={index} className="tag">
          {tag}
        </span>
      ))}
    </div>
  );
}

function TodoList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id} className={item.done ? 'done' : ''}>
          {item.text}
        </li>
      ))}
    </ul>
  );
}

function App() {
  const tags = ["React", "JavaScript", "Frontend", "UI"];
  
  const todos = [
    { id: 1, text: "เรียน React", done: true },
    { id: 2, text: "ทำโปรเจค", done: false },
    { id: 3, text: "Deploy ขึ้น Production", done: false },
  ];
  
  return (
    <div>
      <TagList tags={tags} />
      <TodoList items={todos} />
    </div>
  );
}
```

### Array Methods กับ Props

```jsx
function NumberStats({ numbers }) {
  const sum = numbers.reduce((acc, n) => acc + n, 0);
  const avg = sum / numbers.length;
  const max = Math.max(...numbers);
  const min = Math.min(...numbers);
  
  return (
    <div>
      <p>ผลรวม: {sum}</p>
      <p>เฉลี่ย: {avg.toFixed(2)}</p>
      <p>มากสุด: {max}</p>
      <p>น้อยสุด: {min}</p>
    </div>
  );
}

<NumberStats numbers={[10, 25, 7, 43, 18]} />
```

---

## Step 77: Props ประเภท Object {#step-77}

Object props ใช้สำหรับส่งข้อมูลที่มีหลาย properties ที่เกี่ยวข้องกัน

```jsx
function UserProfile({ user }) {
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>อีเมล: {user.email}</p>
      <p>อายุ: {user.age} ปี</p>
      <p>เมือง: {user.address.city}, {user.address.country}</p>
    </div>
  );
}

function App() {
  const user = {
    name: "สมหญิง รักดี",
    email: "somying@example.com",
    age: 28,
    avatar: "https://example.com/avatar.jpg",
    address: {
      city: "กรุงเทพ",
      country: "ไทย",
      zipCode: "10100"
    }
  };
  
  return <UserProfile user={user} />;
}
```

### Nested Object Destructuring

```jsx
// Destructure object prop ภายใน component
function UserProfile({ user: { name, email, age, address: { city } } }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>{age} ปี</p>
      <p>{city}</p>
    </div>
  );
}

// หรือ Destructure ใน function body
function UserProfile({ user }) {
  const { name, email, age } = user;
  const { city, country } = user.address;
  
  return (
    <div>
      <h2>{name}</h2>
      <p>{email} | {age} ปี</p>
      <p>{city}, {country}</p>
    </div>
  );
}
```

> **Best Practice:** ถ้า object มีหลาย properties ที่ต้องใช้บ่อยๆ ให้ destructure ใน function body จะอ่านง่ายกว่า

---

## Step 78: Props ประเภท Function {#step-78}

Function props (Callback Props) ใช้สำหรับให้ component ลูกสื่อสารกลับมาหา component แม่

```jsx
// Component ลูก - รับ function เป็น prop
function Button({ label, onClick }) {
  return (
    <button onClick={onClick}>
      {label}
    </button>
  );
}

// Component ลูก ที่รับ function พร้อม argument
function CounterButton({ count, onIncrement, onDecrement, onReset }) {
  return (
    <div>
      <button onClick={onDecrement}>-</button>
      <span>{count}</span>
      <button onClick={onIncrement}>+</button>
      <button onClick={onReset}>Reset</button>
    </div>
  );
}

// Component แม่ - ส่ง function เป็น prop
function App() {
  const handleClick = () => {
    alert("คลิกแล้ว!");
  };
  
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <Button label="คลิกฉัน" onClick={handleClick} />
      
      <CounterButton
        count={count}
        onIncrement={() => setCount(count + 1)}
        onDecrement={() => setCount(count - 1)}
        onReset={() => setCount(0)}
      />
    </div>
  );
}
```

### Function Props กับ Parameters

```jsx
function ProductItem({ product, onAddToCart, onRemove }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
      {/* ส่ง product เป็น argument */}
      <button onClick={() => onAddToCart(product)}>
        เพิ่มในตะกร้า
      </button>
      <button onClick={() => onRemove(product.id)}>
        ลบ
      </button>
    </div>
  );
}

function App() {
  const handleAddToCart = (product) => {
    console.log("เพิ่ม:", product.name);
    // logic เพิ่มสินค้า
  };
  
  const handleRemove = (productId) => {
    console.log("ลบ ID:", productId);
    // logic ลบสินค้า
  };
  
  return (
    <ProductItem
      product={{ id: 1, name: "หูฟัง", price: 1299 }}
      onAddToCart={handleAddToCart}
      onRemove={handleRemove}
    />
  );
}
```

---

## Step 79: Props ประเภท JSX/ReactNode {#step-79}

JSX/ReactNode props ใช้สำหรับส่ง UI elements เป็น props ทำให้ component มีความยืดหยุ่นสูง

```jsx
// Component ที่รับ ReactNode เป็น prop
function Card({ header, body, footer }) {
  return (
    <div className="card">
      <div className="card-header">{header}</div>
      <div className="card-body">{body}</div>
      <div className="card-footer">{footer}</div>
    </div>
  );
}

function App() {
  return (
    <Card
      header={<h2>ชื่อ Card</h2>}
      body={
        <div>
          <p>เนื้อหา Card บรรทัดที่ 1</p>
          <p>เนื้อหา Card บรรทัดที่ 2</p>
        </div>
      }
      footer={
        <div>
          <button>ยกเลิก</button>
          <button>ยืนยัน</button>
        </div>
      }
    />
  );
}
```

### Icon Props Pattern

```jsx
// รับ icon เป็น JSX element
function MenuItem({ icon, label, badge }) {
  return (
    <div className="menu-item">
      {icon && <span className="icon">{icon}</span>}
      <span className="label">{label}</span>
      {badge && <span className="badge">{badge}</span>}
    </div>
  );
}

// การใช้งาน
<MenuItem 
  icon={<img src="/icons/home.svg" alt="" />}
  label="หน้าหลัก"
/>

<MenuItem 
  icon={<span>📧</span>}
  label="อีเมล"
  badge={<span className="count">5</span>}
/>
```

---

## Step 80: Default Props {#step-80}

Default Props ช่วยกำหนดค่าเริ่มต้นเมื่อ prop ไม่ได้รับค่า

```jsx
// วิธีที่ 1: Default values ใน destructuring (แนะนำ)
function Button({ 
  label = "คลิก", 
  color = "blue", 
  size = "medium",
  disabled = false,
  onClick = () => {}
}) {
  return (
    <button
      style={{ backgroundColor: color }}
      className={`btn btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
}

// ใช้งานโดยไม่ส่ง props บางตัว
<Button /> // ใช้ค่า default ทั้งหมด
<Button label="บันทึก" color="green" /> // override บาง prop
```

```jsx
// วิธีที่ 2: defaultProps (ยังใช้ได้แต่เป็น legacy)
function Avatar({ src, alt, size }) {
  return (
    <img 
      src={src} 
      alt={alt}
      width={size}
      height={size}
    />
  );
}

Avatar.defaultProps = {
  src: "/default-avatar.png",
  alt: "รูปโปรไฟล์",
  size: 50
};
```

### Default Props สำหรับ Object และ Array

```jsx
function UserList({ 
  users = [],           // Array default
  config = {            // Object default  
    showAge: true,
    showEmail: false
  }
}) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          {config.showAge && ` (${user.age})`}
          {config.showEmail && ` - ${user.email}`}
        </li>
      ))}
    </ul>
  );
}
```

> **Warning:** ระวังการกำหนด default value ของ object หรือ array ใน destructuring เพราะจะสร้าง object/array ใหม่ทุกครั้งที่ render อาจทำให้เกิด performance issue ในบาง case

---

## Step 81: Destructuring Props {#step-81}

Destructuring คือวิธีที่แนะนำในการรับ props เพราะอ่านง่ายและกระชับกว่า

```jsx
// ❌ ไม่แนะนำ - ต้องพิมพ์ props. ทุกที่
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
      <p>{props.age}</p>
    </div>
  );
}

// ✅ แนะนำ - Destructuring ทำให้สั้นลง
function UserCard({ name, email, age }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>{age}</p>
    </div>
  );
}
```

### Advanced Destructuring

```jsx
// Rename ตอน destructure
function Component({ userName: name, userEmail: email }) {
  return <div>{name} - {email}</div>;
}

// Nested destructuring
function Address({ 
  address: { 
    street, 
    city, 
    country = "ไทย"  // default value ใน nested
  } 
}) {
  return <p>{street}, {city}, {country}</p>;
}

// Rest operator
function Button({ label, style, className, ...rest }) {
  // rest จะมี props ที่เหลือทั้งหมด
  return (
    <button 
      style={style}
      className={className}
      {...rest}  // ส่ง event handlers และ props อื่นๆ
    >
      {label}
    </button>
  );
}

// ใช้งาน - onClick, id, data-testid จะอยู่ใน ...rest
<Button 
  label="บันทึก" 
  style={{ color: "white" }}
  onClick={handleSave}
  id="save-btn"
  data-testid="save-button"
/>
```

---

## Step 82: Spread Operator กับ Props {#step-82}

Spread operator `{...props}` ช่วยส่ง props ทั้งหมดจาก object ไปยัง component ได้สะดวก

```jsx
// Spread object เป็น props
function UserCard({ name, email, age }) {
  return (
    <div>
      <p>{name}</p>
      <p>{email}</p>
      <p>{age}</p>
    </div>
  );
}

function App() {
  const userProps = {
    name: "สมชาย",
    email: "somchai@example.com",
    age: 30
  };
  
  // แทนที่จะพิมพ์ name={userProps.name} email={userProps.email}...
  return <UserCard {...userProps} />;
}
```

### Spread กับการ Override Props

```jsx
const defaultButtonProps = {
  type: "button",
  className: "btn",
  disabled: false
};

// Props ทีมาทีหลัง override props ก่อนหน้า
<button 
  {...defaultButtonProps}
  disabled={true}        // override disabled เป็น true
  className="btn btn-primary"  // override className
>
  คลิก
</button>
```

### Spread Props ใน HOC Pattern

```jsx
// Higher-Order Component ใช้ spread props
function withLogging(WrappedComponent) {
  return function LoggedComponent(props) {
    console.log("Rendering:", WrappedComponent.name, props);
    
    // ส่ง props ทั้งหมดไปยัง component ที่ wrap
    return <WrappedComponent {...props} />;
  };
}

const LoggedButton = withLogging(Button);
<LoggedButton label="คลิก" onClick={handleClick} />
```

> **Warning:** ระวังการใช้ spread props โดยไม่ตรวจสอบ เพราะอาจส่ง DOM attributes ที่ไม่รู้จักไปยัง HTML element ทำให้เกิด warning

---

## Step 83: PropTypes Validation {#step-83}

PropTypes ช่วย validate ประเภทของ props ในระหว่าง development

```bash
# ติดตั้ง prop-types
npm install prop-types
```

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email, isAdmin, hobbies, address, onDelete }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>อายุ: {age}</p>
      <p>อีเมล: {email}</p>
      {isAdmin && <span className="badge">Admin</span>}
      <ul>
        {hobbies.map((hobby, i) => <li key={i}>{hobby}</li>)}
      </ul>
      <p>{address.city}, {address.country}</p>
      <button onClick={onDelete}>ลบ</button>
    </div>
  );
}

// กำหนด PropTypes
UserCard.propTypes = {
  name: PropTypes.string.isRequired,           // string บังคับ
  age: PropTypes.number,                       // number ไม่บังคับ
  email: PropTypes.string.isRequired,          // string บังคับ
  isAdmin: PropTypes.bool,                     // boolean
  hobbies: PropTypes.arrayOf(PropTypes.string), // array of strings
  address: PropTypes.shape({                   // object มี shape กำหนด
    city: PropTypes.string,
    country: PropTypes.string
  }),
  onDelete: PropTypes.func.isRequired,         // function บังคับ
};

// กำหนด default props
UserCard.defaultProps = {
  age: 0,
  isAdmin: false,
  hobbies: [],
  address: { city: "ไม่ระบุ", country: "ไทย" }
};
```

### PropTypes ชนิดต่างๆ

```jsx
Component.propTypes = {
  // Primitive types
  name: PropTypes.string,
  count: PropTypes.number,
  isActive: PropTypes.bool,
  
  // Any type
  data: PropTypes.any,
  
  // Specific values (enum)
  status: PropTypes.oneOf(['active', 'inactive', 'pending']),
  
  // Multiple types
  id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]),
  
  // Arrays
  tags: PropTypes.arrayOf(PropTypes.string),
  items: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number,
    name: PropTypes.string
  })),
  
  // Objects
  config: PropTypes.objectOf(PropTypes.bool),
  user: PropTypes.shape({
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }),
  
  // React elements
  children: PropTypes.node,
  icon: PropTypes.element,
  
  // Function
  onClick: PropTypes.func,
};
```

---

## Step 84: Children Props พื้นฐาน {#step-84}

`children` เป็น special prop ที่ React ส่งให้โดยอัตโนมัติ คือสิ่งที่อยู่ระหว่าง opening และ closing tag

```jsx
// Component ที่ใช้ children
function Container({ children }) {
  return (
    <div className="container">
      {children}
    </div>
  );
}

function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-title">{title}</div>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// การใช้งาน
function App() {
  return (
    <Container>
      <h1>นี่คือ children</h1>
      <p>ข้อความนี้อยู่ใน Container</p>
    </Container>
  );
}

function App() {
  return (
    <Card title="ข่าวสาร">
      <p>เนื้อหาข่าว...</p>
      <a href="#">อ่านต่อ</a>
    </Card>
  );
}
```

### Children ประเภทต่างๆ

```jsx
function Wrapper({ children }) {
  return <div>{children}</div>;
}

// String
<Wrapper>สวัสดี</Wrapper>

// Number
<Wrapper>{42}</Wrapper>

// JSX Element
<Wrapper><p>Paragraph</p></Wrapper>

// หลาย Elements
<Wrapper>
  <h1>Title</h1>
  <p>Content</p>
  <button>Button</button>
</Wrapper>

// Array
<Wrapper>
  {[1, 2, 3].map(n => <span key={n}>{n}</span>)}
</Wrapper>
```

---

## Step 85: Children Props ขั้นสูง {#step-85}

React.Children API ช่วย manipulate children ได้อย่างมีประสิทธิภาพ

```jsx
import React from 'react';

// นับจำนวน children
function ChildCounter({ children }) {
  const count = React.Children.count(children);
  return (
    <div>
      <p>จำนวน children: {count}</p>
      {children}
    </div>
  );
}

// Map over children
function StyledList({ children }) {
  return (
    <ul>
      {React.Children.map(children, (child, index) => (
        <li key={index} style={{ padding: '8px', borderBottom: '1px solid #eee' }}>
          {child}
        </li>
      ))}
    </ul>
  );
}

// การใช้งาน
<StyledList>
  <span>รายการที่ 1</span>
  <span>รายการที่ 2</span>
  <span>รายการที่ 3</span>
</StyledList>
```

### Slot Pattern ด้วย Children ที่มีชื่อ

```jsx
// Layout component ที่มีหลาย slots
function PageLayout({ children }) {
  // แยก children ตาม displayName หรือ type
  const header = React.Children.toArray(children).find(
    child => child.type === PageLayout.Header
  );
  const main = React.Children.toArray(children).find(
    child => child.type === PageLayout.Main
  );
  const sidebar = React.Children.toArray(children).find(
    child => child.type === PageLayout.Sidebar
  );
  
  return (
    <div className="page-layout">
      <header>{header}</header>
      <div className="content-area">
        <main>{main}</main>
        {sidebar && <aside>{sidebar}</aside>}
      </div>
    </div>
  );
}

// Named components (Slots)
PageLayout.Header = ({ children }) => <>{children}</>;
PageLayout.Main = ({ children }) => <>{children}</>;
PageLayout.Sidebar = ({ children }) => <>{children}</>;

// การใช้งาน
<PageLayout>
  <PageLayout.Header>
    <h1>ชื่อเว็บไซต์</h1>
  </PageLayout.Header>
  <PageLayout.Main>
    <article>เนื้อหาหลัก</article>
  </PageLayout.Main>
  <PageLayout.Sidebar>
    <nav>เมนูด้านข้าง</nav>
  </PageLayout.Sidebar>
</PageLayout>
```

### Cloning Children กับ Extra Props

```jsx
// ส่ง extra props ไปยัง children
function RadioGroup({ name, children, onChange }) {
  return (
    <div className="radio-group">
      {React.Children.map(children, child => 
        React.cloneElement(child, { name, onChange })
      )}
    </div>
  );
}

function RadioButton({ value, label, name, onChange }) {
  return (
    <label>
      <input 
        type="radio" 
        name={name} 
        value={value}
        onChange={onChange}
      />
      {label}
    </label>
  );
}

// การใช้งาน
<RadioGroup name="gender" onChange={handleChange}>
  <RadioButton value="male" label="ชาย" />
  <RadioButton value="female" label="หญิง" />
  <RadioButton value="other" label="อื่นๆ" />
</RadioGroup>
```

---

## Step 86: Render Props Pattern {#step-86}

Render Props คือ pattern ที่ component รับ function เป็น prop และเรียกใช้เพื่อ render

```jsx
// MouseTracker ที่ใช้ Render Props
function MouseTracker({ render }) {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });
  
  const handleMouseMove = (event) => {
    setPosition({
      x: event.clientX,
      y: event.clientY
    });
  };
  
  return (
    <div 
      style={{ height: '300px', border: '1px solid #ccc' }}
      onMouseMove={handleMouseMove}
    >
      {render(position)}
    </div>
  );
}

// การใช้งาน
function App() {
  return (
    <MouseTracker 
      render={({ x, y }) => (
        <p>ตำแหน่งเมาส์: ({x}, {y})</p>
      )}
    />
  );
}
```

### Render Props กับ children function

```jsx
// ใช้ children เป็น function (Function as Children)
function DataFetcher({ url, children }) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState(null);
  
  React.useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [url]);
  
  return children({ data, loading, error });
}

// การใช้งาน
<DataFetcher url="https://api.example.com/users">
  {({ data, loading, error }) => {
    if (loading) return <p>กำลังโหลด...</p>;
    if (error) return <p>เกิดข้อผิดพลาด: {error}</p>;
    return (
      <ul>
        {data.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    );
  }}
</DataFetcher>
```

---

## Step 87: Prop Drilling คืออะไร {#step-87}

**Prop Drilling** คือปรากฏการณ์ที่ต้องส่ง props ผ่าน component หลายชั้นเพื่อให้ถึง component ที่ต้องการจริงๆ

```jsx
// ตัวอย่าง Prop Drilling - ข้อมูล user ถูกส่งผ่านหลายชั้น
function App() {
  const user = { name: "สมชาย", role: "Admin" };
  
  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  // Dashboard ไม่ได้ใช้ user แต่ต้องส่งต่อ
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  // Sidebar ก็ไม่ได้ใช้ user แต่ต้องส่งต่อ
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  // UserMenu ต้องการ user จริงๆ
  return (
    <div>
      <p>สวัสดี, {user.name}</p>
      <p>Role: {user.role}</p>
    </div>
  );
}
```

### ปัญหาของ Prop Drilling

1. **Code ยากต่อการ maintain** - ต้องแก้หลายไฟล์เมื่อ prop เปลี่ยน
2. **Component ไม่ clean** - รับ props ที่ตัวเองไม่ได้ใช้
3. **Hard to refactor** - เมื่อ restructure component tree
4. **Confusing to debug** - ตามหา prop ยาก

---

## Step 88: ปัญหาของ Prop Drilling และวิธีแก้ {#step-88}

### วิธีแก้ปัญหา Prop Drilling

**วิธีที่ 1: Component Composition**
```jsx
// แทนที่จะส่ง user ผ่าน Dashboard
function App() {
  const user = { name: "สมชาย", role: "Admin" };
  
  // ส่ง UserMenu โดยตรงเป็น children
  return (
    <Dashboard>
      <Sidebar>
        <UserMenu user={user} />
      </Sidebar>
    </Dashboard>
  );
}

function Dashboard({ children }) {
  return <div className="dashboard">{children}</div>;
}

function Sidebar({ children }) {
  return <aside className="sidebar">{children}</aside>;
}

function UserMenu({ user }) {
  return <div>{user.name} ({user.role})</div>;
}
```

**วิธีที่ 2: Context API** (เรียนใน Part ถัดไป)
```jsx
// สร้าง Context
const UserContext = React.createContext(null);

function App() {
  const user = { name: "สมชาย", role: "Admin" };
  
  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// ใช้ Context ที่ component ใดก็ได้โดยตรง
function UserMenu() {
  const user = React.useContext(UserContext);
  return <div>{user.name} ({user.role})</div>;
}
```

---

## Step 89: Data Flow ใน React {#step-89}

React ใช้ **Unidirectional Data Flow** (การไหลของข้อมูลทางเดียว)

```
                    App (State)
                       │
                       ▼
                  Dashboard
                       │
               ┌───────┴───────┐
               ▼               ▼
           Sidebar           Content
               │
          UserMenu
```

### ทิศทางการไหลของข้อมูล

```jsx
// ข้อมูลไหลลงไปผ่าน Props
// Events ไหลขึ้นมาผ่าน Callbacks

function App() {
  const [items, setItems] = React.useState([]);
  
  const addItem = (newItem) => {
    setItems([...items, newItem]);
  };
  
  const removeItem = (id) => {
    setItems(items.filter(item => item.id !== id));
  };
  
  return (
    <div>
      {/* ข้อมูลไหลลง */}
      <ItemList items={items} onRemove={removeItem} />
      {/* Event ไหลขึ้น */}
      <AddItemForm onAdd={addItem} />
    </div>
  );
}
```

---

## Step 90: Lifting State Up คืออะไร {#step-90}

**Lifting State Up** คือ pattern ที่ย้าย state ขึ้นไปยัง component แม่ร่วม (common ancestor) เพื่อให้ component พี่น้องสามารถแชร์ข้อมูลกันได้

### เมื่อไหรต้องใช้?

เมื่อ component สองตัวขึ้นไปต้องการแชร์ state เดียวกัน

```
       ❌ State อยู่ใน component ลูก
       
          App
         /   \
     CompA   CompB
     (state)      
     
       ✅ Lift State Up
       
          App
         (state)
         /   \
     CompA   CompB
     (gets   (gets
      state)  state)
```

---

## Step 91: Lifting State Up ตัวอย่าง {#step-91}

### ตัวอย่าง: Temperature Converter

```jsx
// ❌ State แยกกัน - ไม่ sync กัน
function CelsiusInput() {
  const [celsius, setCelsius] = React.useState(0);
  return (
    <input 
      value={celsius}
      onChange={e => setCelsius(e.target.value)}
      placeholder="Celsius"
    />
  );
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = React.useState(32);
  return (
    <input 
      value={fahrenheit}
      onChange={e => setFahrenheit(e.target.value)}
      placeholder="Fahrenheit"
    />
  );
}
```

```jsx
// ✅ Lifted State - sync กัน
function TemperatureConverter() {
  // State อยู่ที่ parent
  const [celsius, setCelsius] = React.useState(0);
  const fahrenheit = (celsius * 9/5) + 32;
  
  const handleCelsiusChange = (e) => {
    setCelsius(Number(e.target.value));
  };
  
  const handleFahrenheitChange = (e) => {
    const f = Number(e.target.value);
    setCelsius((f - 32) * 5/9);
  };
  
  return (
    <div>
      <div>
        <label>Celsius:</label>
        <input 
          type="number"
          value={celsius.toFixed(1)}
          onChange={handleCelsiusChange}
        />
      </div>
      <div>
        <label>Fahrenheit:</label>
        <input 
          type="number"
          value={fahrenheit.toFixed(1)}
          onChange={handleFahrenheitChange}
        />
      </div>
      <p>{celsius}°C = {fahrenheit}°F</p>
    </div>
  );
}
```

### ตัวอย่าง: Shopping Cart

```jsx
function App() {
  const [cartItems, setCartItems] = React.useState([]);
  
  const addToCart = (product) => {
    setCartItems(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item => 
          item.id === product.id 
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  };
  
  const removeFromCart = (productId) => {
    setCartItems(prev => prev.filter(item => item.id !== productId));
  };
  
  const totalItems = cartItems.reduce((sum, item) => sum + item.quantity, 0);
  
  return (
    <div>
      <CartIcon count={totalItems} />
      <ProductList onAddToCart={addToCart} />
      <CartSummary items={cartItems} onRemove={removeFromCart} />
    </div>
  );
}

function CartIcon({ count }) {
  return (
    <div>
      🛒 <span>{count}</span>
    </div>
  );
}

function ProductList({ onAddToCart }) {
  const products = [
    { id: 1, name: "หูฟัง", price: 999 },
    { id: 2, name: "คีย์บอร์ด", price: 1299 },
    { id: 3, name: "เมาส์", price: 599 },
  ];
  
  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <span>{product.name} - ฿{product.price}</span>
          <button onClick={() => onAddToCart(product)}>
            เพิ่มในตะกร้า
          </button>
        </div>
      ))}
    </div>
  );
}

function CartSummary({ items, onRemove }) {
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  
  return (
    <div>
      <h2>ตะกร้าสินค้า</h2>
      {items.map(item => (
        <div key={item.id}>
          {item.name} x{item.quantity} = ฿{item.price * item.quantity}
          <button onClick={() => onRemove(item.id)}>ลบ</button>
        </div>
      ))}
      <p>รวม: ฿{total}</p>
    </div>
  );
}
```

---

## Step 92: Callback Props Pattern {#step-92}

Callback Props เป็น pattern สำคัญสำหรับการสื่อสารจาก child ไปยัง parent

### Pattern พื้นฐาน

```jsx
// Pattern: onXxx naming convention
function SearchBox({ onSearch, onClear }) {
  const [query, setQuery] = React.useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(query);  // ส่งค่า query กลับไป parent
  };
  
  const handleClear = () => {
    setQuery('');
    onClear();  // แจ้ง parent
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหา..."
      />
      <button type="submit">ค้นหา</button>
      <button type="button" onClick={handleClear}>ล้าง</button>
    </form>
  );
}

function App() {
  const [results, setResults] = React.useState([]);
  
  const handleSearch = (query) => {
    console.log("ค้นหา:", query);
    // fetch results...
    setResults([]); // mock
  };
  
  const handleClear = () => {
    setResults([]);
  };
  
  return (
    <div>
      <SearchBox onSearch={handleSearch} onClear={handleClear} />
      <SearchResults results={results} />
    </div>
  );
}
```

### Callback กับ Complex Data

```jsx
function EditableRow({ item, onSave, onDelete }) {
  const [isEditing, setIsEditing] = React.useState(false);
  const [editedName, setEditedName] = React.useState(item.name);
  
  const handleSave = () => {
    // ส่ง updated item กลับไป parent
    onSave({
      ...item,
      name: editedName
    });
    setIsEditing(false);
  };
  
  if (isEditing) {
    return (
      <tr>
        <td>
          <input 
            value={editedName}
            onChange={e => setEditedName(e.target.value)}
          />
        </td>
        <td>
          <button onClick={handleSave}>บันทึก</button>
          <button onClick={() => setIsEditing(false)}>ยกเลิก</button>
        </td>
      </tr>
    );
  }
  
  return (
    <tr>
      <td>{item.name}</td>
      <td>
        <button onClick={() => setIsEditing(true)}>แก้ไข</button>
        <button onClick={() => onDelete(item.id)}>ลบ</button>
      </td>
    </tr>
  );
}
```

---

## Step 93: Controlled Components ผ่าน Props {#step-93}

Controlled Components คือ component ที่ state ถูกควบคุมจาก parent ผ่าน props

```jsx
// Controlled Input Component
function TextInput({ value, onChange, label, error }) {
  return (
    <div className="form-field">
      <label>{label}</label>
      <input
        value={value}          // value มาจาก parent
        onChange={onChange}    // แจ้ง parent เมื่อมีการเปลี่ยนแปลง
        className={error ? 'input-error' : ''}
      />
      {error && <span className="error-text">{error}</span>}
    </div>
  );
}

// Parent ควบคุม state
function LoginForm() {
  const [formData, setFormData] = React.useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = React.useState({});
  
  const handleChange = (field) => (e) => {
    setFormData(prev => ({
      ...prev,
      [field]: e.target.value
    }));
    // ล้าง error เมื่อพิมพ์
    setErrors(prev => ({ ...prev, [field]: '' }));
  };
  
  const validate = () => {
    const newErrors = {};
    if (!formData.email) newErrors.email = 'กรุณากรอกอีเมล';
    if (!formData.password) newErrors.password = 'กรุณากรอกรหัสผ่าน';
    return newErrors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const validationErrors = validate();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    console.log("Login:", formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <TextInput
        label="อีเมล"
        value={formData.email}
        onChange={handleChange('email')}
        error={errors.email}
      />
      <TextInput
        label="รหัสผ่าน"
        value={formData.password}
        onChange={handleChange('password')}
        error={errors.password}
      />
      <button type="submit">เข้าสู่ระบบ</button>
    </form>
  );
}
```

---

## Step 94: Props vs State {#step-94}

ความแตกต่างระหว่าง Props และ State

| Feature | Props | State |
|---------|-------|-------|
| ใครเป็นเจ้าของ | Parent component | Component นั้นเอง |
| แก้ไขได้ไหม | ❌ Read-only | ✅ Mutable |
| เปลี่ยนเมื่อ | Parent re-renders | setState() ถูกเรียก |
| เก็บข้อมูลประเภท | ข้อมูลจาก parent | ข้อมูลภายใน component |
| ทิศทาง | บนลงล่าง | อยู่ใน component |

```jsx
function Counter({ initialCount }) {  // initialCount เป็น prop
  // ใช้ prop สร้าง initial state
  const [count, setCount] = React.useState(initialCount);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

// ตัวอย่างที่ดี - Props สำหรับ configuration, State สำหรับ interaction
function Accordion({ title, defaultOpen = false }) {
  const [isOpen, setIsOpen] = React.useState(defaultOpen); // state ควบคุม UI
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {title} {isOpen ? '▲' : '▼'}
      </button>
      {isOpen && <div>เนื้อหา...</div>}
    </div>
  );
}
```

### คำถามง่ายๆ ในการตัดสินใจ

1. **Props** ถ้า: ข้อมูลมาจาก parent, ไม่ต้องการให้ component นี้เปลี่ยนได้
2. **State** ถ้า: ข้อมูลเปลี่ยนแปลงได้จากภายใน component, ควบคุม UI

---

## Step 95: แนวทางปฏิบัติที่ดี (Best Practices) {#step-95}

### 1. ตั้งชื่อ Props ให้ชัดเจน

```jsx
// ❌ ไม่ชัดเจน
<Button txt="บันทึก" fn={save} flg={true} />

// ✅ ชัดเจน
<Button label="บันทึก" onClick={handleSave} isDisabled={isSaving} />
```

### 2. Event Handler Props ใช้ prefix `on`

```jsx
// ✅ Convention สากล
onSubmit, onClick, onChange, onDelete, onSave, onClose
```

### 3. Boolean Props ใช้ prefix `is`, `has`, `can`

```jsx
// ✅ อ่านเข้าใจง่าย
isLoading, isDisabled, hasError, canEdit, shouldRender
```

### 4. ไม่ส่ง Props ที่ไม่จำเป็น

```jsx
// ❌ ส่ง props ทั้ง object ทั้งๆ ที่ใช้แค่ name
<UserLabel user={user} />

// ✅ ส่งเฉพาะสิ่งที่ใช้
<UserLabel name={user.name} />
```

### 5. Destructure Props เสมอ

```jsx
// ❌
function Component(props) {
  return <div>{props.name} - {props.age}</div>;
}

// ✅
function Component({ name, age }) {
  return <div>{name} - {age}</div>;
}
```

### 6. กำหนด Default Props

```jsx
// ✅ ป้องกัน undefined errors
function Button({ label = "คลิก", variant = "primary", onClick = () => {} }) {
  return <button className={variant} onClick={onClick}>{label}</button>;
}
```

### 7. Validate Props ด้วย PropTypes หรือ TypeScript

```jsx
// PropTypes
Button.propTypes = {
  label: PropTypes.string.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  onClick: PropTypes.func
};

// หรือ TypeScript Interface
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
  onClick?: () => void;
}
```

### 8. ไม่ mutate Props

```jsx
// ❌ ผิด
function Component({ items }) {
  items.push("new item"); // ห้ามแก้ไข!
  return <ul>{items.map(item => <li>{item}</li>)}</ul>;
}

// ✅ สร้าง copy ใหม่แทน
function Component({ items }) {
  const updatedItems = [...items, "new item"];
  return <ul>{updatedItems.map(item => <li>{item}</li>)}</ul>;
}
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม (ตอบในใจ)

1. Props กับ State ต่างกันอย่างไร?
2. ทำไม Props ถึงเป็น Read-only?
3. Prop Drilling คืออะไร และมีวิธีแก้อย่างไรบ้าง?
4. เมื่อไหรควรใช้ Lifting State Up?
5. `children` prop คืออะไร?

### แบบฝึกหัด

**Exercise 1:** สร้าง `ProfileCard` component ที่รับ props:
- `name` (string, required)
- `role` (string, default: "Member")
- `avatar` (string)
- `isOnline` (boolean)
- `skills` (array of strings)
- `onMessage` (function callback)

**Exercise 2:** Lifting State Up
สร้าง Todo App ที่มี:
- `App` เป็น parent จัดการ todos state
- `TodoInput` รับ input และเรียก callback เมื่อเพิ่ม todo
- `TodoList` แสดง list และเรียก callback เมื่อลบ/toggle

**Exercise 3:** Callback Props
สร้าง `RatingStars` component ที่:
- แสดง 5 ดาว
- รับ `value` (controlled) และ `onChange` callback
- เรียก `onChange(newRating)` เมื่อคลิกดาว

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- Props ทุกประเภท: string, number, boolean, array, object, function, JSX
- Default Props และ PropTypes Validation
- Children Props ทั้งพื้นฐานและขั้นสูง
- Prop Drilling และปัญหา
- Lifting State Up Pattern
- Callback Props Pattern
- Data Flow ใน React (Unidirectional)

---

## อ่านต่อ

➡️ **[Part 06: State และ useState Hook](./part-06-state-and-useState.md)**

เราจะเรียนรู้เรื่อง State ในเชิงลึก useState Hook, Functional Updates, และ Immutability

---

*React Course - Part 05 | สร้างโดย React Course Thailand*
