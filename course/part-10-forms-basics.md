# Part 10: Forms พื้นฐาน (Forms Basics)
## Step 186-210 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 186: Forms ใน React คืออะไร?](#step-186)
- [Step 187: Controlled vs Uncontrolled Components](#step-187)
- [Step 188: Controlled Input พื้นฐาน](#step-188)
- [Step 189: Uncontrolled Input กับ useRef](#step-189)
- [Step 190: Textarea Controlled Component](#step-190)
- [Step 191: Select (Dropdown) Component](#step-191)
- [Step 192: Checkbox Component](#step-192)
- [Step 193: Radio Button Component](#step-193)
- [Step 194: File Input Component](#step-194)
- [Step 195: Multiple Inputs ด้วย Generic Handler](#step-195)
- [Step 196: Form Validation พื้นฐาน](#step-196)
- [Step 197: Validation Rules](#step-197)
- [Step 198: Real-time Validation](#step-198)
- [Step 199: Form Submission Handling](#step-199)
- [Step 200: Form Reset](#step-200)
- [Step 201: Dependent Fields](#step-201)
- [Step 202: Dynamic Form Fields](#step-202)
- [Step 203: Multi-step Form](#step-203)
- [Step 204: Form สำหรับ Edit (Prefilled)](#step-204)
- [Step 205: Complex Form Example - Registration](#step-205)
- [Step 206: Custom Form Hooks](#step-206)
- [Step 207: Form ที่ Accessible](#step-207)
- [Step 208: Form Libraries แนะนำ](#step-208)
- [Step 209: Form Patterns และ Anti-patterns](#step-209)
- [Step 210: สรุปและ Best Practices](#step-210)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 186: Forms ใน React คืออะไร? {#step-186}

Forms ใน React ทำงานแตกต่างจาก HTML form เล็กน้อย เพราะ React จัดการ state ของ form เอง

### ปัญหากับ HTML Forms ปกติ

```html
<!-- HTML form ปกติ -->
<form action="/submit" method="POST">
  <input type="text" name="username" />
  <button type="submit">Submit</button>
</form>
<!-- กด submit = page reload และส่งข้อมูลไปยัง server -->
```

### React Form - ควบคุมเองทั้งหมด

```jsx
function BasicForm() {
  const [username, setUsername] = React.useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault(); // ป้องกัน page reload
    console.log('ส่งข้อมูล:', username);
    // ส่งข้อมูลผ่าน API แทน
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="ชื่อผู้ใช้"
      />
      <button type="submit">ยืนยัน</button>
    </form>
  );
}
```

### ประเภทของ Form ใน React

1. **Controlled Components** - React เป็น "single source of truth"
2. **Uncontrolled Components** - DOM เก็บ state ของตัวเอง
3. **Hybrid** - ใช้ทั้งสองแบบตามเหมาะสม

---

## Step 187: Controlled vs Uncontrolled Components {#step-187}

### Controlled Component

React เป็นผู้ควบคุม state ของ input ทั้งหมด

```jsx
function ControlledInput() {
  const [value, setValue] = React.useState('');
  
  return (
    <input
      value={value}              // React ควบคุม value
      onChange={(e) => setValue(e.target.value)}  // อัพเดท state
    />
  );
}
// ข้อดี: React รู้ค่า input ตลอดเวลา, validation ทำได้ง่าย
// ข้อเสีย: ต้องเขียน onChange สำหรับทุก input
```

### Uncontrolled Component

DOM เก็บ state เอง, React อ่านค่าเมื่อต้องการ

```jsx
function UncontrolledInput() {
  const inputRef = React.useRef(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current.value); // อ่านค่าตอน submit
  };
  
  return (
    <>
      <input ref={inputRef} defaultValue="ค่าเริ่มต้น" />
      <button onClick={handleSubmit}>ดูค่า</button>
    </>
  );
}
// ข้อดี: โค้ดน้อยกว่า, ดีสำหรับ file inputs
// ข้อเสีย: React ไม่รู้ค่า input ตลอดเวลา
```

### เปรียบเทียบโดยตรง

| Feature | Controlled | Uncontrolled |
|---------|-----------|--------------|
| Single source of truth | React state | DOM |
| Real-time validation | ✅ ง่าย | ❌ ยาก |
| Programmatic control | ✅ ง่าย | ❌ ยาก |
| Code amount | เยอะกว่า | น้อยกว่า |
| File input | ❌ ไม่ได้ | ✅ ได้ |
| แนะนำสำหรับ | ส่วนใหญ่ | file upload, unstructured |

> **Tip:** ใน React ส่วนใหญ่แนะนำ Controlled Components เพราะ predictable และ debug ง่ายกว่า

---

## Step 188: Controlled Input พื้นฐาน {#step-188}

```jsx
function TextInputDemo() {
  const [name, setName] = React.useState('');
  const [email, setEmail] = React.useState('');
  const [age, setAge] = React.useState('');
  const [website, setWebsite] = React.useState('');
  const [phone, setPhone] = React.useState('');
  
  return (
    <form>
      {/* Text Input */}
      <div>
        <label htmlFor="name">ชื่อ:</label>
        <input
          id="name"
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="กรอกชื่อ"
        />
        <span>{name.length}/50</span>
      </div>
      
      {/* Email Input */}
      <div>
        <label htmlFor="email">อีเมล:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="example@email.com"
        />
      </div>
      
      {/* Number Input */}
      <div>
        <label htmlFor="age">อายุ:</label>
        <input
          id="age"
          type="number"
          value={age}
          onChange={(e) => setAge(e.target.value)}
          min="0"
          max="120"
        />
      </div>
      
      {/* URL Input */}
      <div>
        <label htmlFor="website">เว็บไซต์:</label>
        <input
          id="website"
          type="url"
          value={website}
          onChange={(e) => setWebsite(e.target.value)}
          placeholder="https://example.com"
        />
      </div>
      
      {/* Tel Input */}
      <div>
        <label htmlFor="phone">เบอร์โทร:</label>
        <input
          id="phone"
          type="tel"
          value={phone}
          onChange={(e) => setPhone(e.target.value)}
          placeholder="0812345678"
          pattern="[0-9]{10}"
        />
      </div>
      
      {/* แสดงค่าทั้งหมด */}
      <pre style={{ backgroundColor: '#f5f5f5', padding: 16 }}>
        {JSON.stringify({ name, email, age, website, phone }, null, 2)}
      </pre>
    </form>
  );
}
```

---

## Step 189: Uncontrolled Input กับ useRef {#step-189}

```jsx
function UncontrolledForm() {
  const nameRef = React.useRef(null);
  const emailRef = React.useRef(null);
  const messageRef = React.useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
      message: messageRef.current.value
    };
    
    console.log('Form Data:', formData);
    
    // Reset form
    e.target.reset();
  };
  
  const handleReset = () => {
    nameRef.current.value = '';
    emailRef.current.value = '';
    messageRef.current.value = '';
    nameRef.current.focus();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>ชื่อ:</label>
        <input ref={nameRef} type="text" defaultValue="" placeholder="ชื่อของคุณ" />
      </div>
      
      <div>
        <label>อีเมล:</label>
        <input ref={emailRef} type="email" placeholder="อีเมล" />
      </div>
      
      <div>
        <label>ข้อความ:</label>
        <textarea ref={messageRef} rows={4} placeholder="ข้อความ" />
      </div>
      
      <button type="submit">ส่ง</button>
      <button type="button" onClick={handleReset}>ล้าง</button>
    </form>
  );
}
```

### FormData API กับ Uncontrolled

```jsx
function FormDataExample() {
  const formRef = React.useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData.entries());
    
    console.log(data);
    // { name: "...", email: "...", message: "..." }
  };
  
  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <input name="name" placeholder="ชื่อ" />
      <input name="email" type="email" placeholder="อีเมล" />
      <textarea name="message" placeholder="ข้อความ" />
      <button type="submit">ส่ง</button>
    </form>
  );
}
```

---

## Step 190: Textarea Controlled Component {#step-190}

```jsx
function TextareaDemo() {
  const [content, setContent] = React.useState('');
  const MAX_LENGTH = 500;
  const remaining = MAX_LENGTH - content.length;
  
  // Auto-resize textarea
  const textareaRef = React.useRef(null);
  
  React.useEffect(() => {
    const textarea = textareaRef.current;
    if (textarea) {
      textarea.style.height = 'auto';
      textarea.style.height = textarea.scrollHeight + 'px';
    }
  }, [content]);
  
  return (
    <div>
      <label>ข้อความ:</label>
      <textarea
        ref={textareaRef}
        value={content}
        onChange={(e) => {
          if (e.target.value.length <= MAX_LENGTH) {
            setContent(e.target.value);
          }
        }}
        placeholder="กรอกข้อความ (สูงสุด 500 ตัวอักษร)..."
        style={{
          width: '100%',
          minHeight: 100,
          resize: 'none',  // ปิด manual resize เพราะ auto-resize แทน
          overflow: 'hidden',
          padding: 12,
          border: `1px solid ${remaining < 50 ? 'orange' : remaining === 0 ? 'red' : '#ddd'}`,
          borderRadius: 4
        }}
        rows={4}
      />
      
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between',
        fontSize: 12,
        color: remaining < 50 ? 'orange' : remaining === 0 ? 'red' : '#888'
      }}>
        <span>
          {content.trim().split(/\s+/).filter(Boolean).length} คำ
        </span>
        <span>
          {remaining} / {MAX_LENGTH} ตัวอักษร
        </span>
      </div>
    </div>
  );
}
```

---

## Step 191: Select (Dropdown) Component {#step-191}

```jsx
function SelectDemo() {
  const [country, setCountry] = React.useState('');
  const [province, setProvince] = React.useState('');
  
  const countries = [
    { code: 'TH', name: 'ประเทศไทย' },
    { code: 'JP', name: 'ญี่ปุ่น' },
    { code: 'US', name: 'สหรัฐอเมริกา' },
    { code: 'UK', name: 'สหราชอาณาจักร' },
  ];
  
  const provinces = {
    TH: ['กรุงเทพมหานคร', 'เชียงใหม่', 'ภูเก็ต', 'ขอนแก่น'],
    JP: ['Tokyo', 'Osaka', 'Kyoto'],
    US: ['New York', 'California', 'Texas'],
    UK: ['London', 'Manchester', 'Edinburgh'],
  };
  
  const handleCountryChange = (e) => {
    setCountry(e.target.value);
    setProvince(''); // reset province เมื่อ country เปลี่ยน
  };
  
  return (
    <div>
      {/* Simple select */}
      <div>
        <label>ประเทศ:</label>
        <select value={country} onChange={handleCountryChange}>
          <option value="">-- เลือกประเทศ --</option>
          {countries.map(c => (
            <option key={c.code} value={c.code}>{c.name}</option>
          ))}
        </select>
      </div>
      
      {/* Dependent select */}
      <div>
        <label>จังหวัด/รัฐ:</label>
        <select 
          value={province} 
          onChange={(e) => setProvince(e.target.value)}
          disabled={!country}
        >
          <option value="">
            {country ? '-- เลือกจังหวัด --' : '-- เลือกประเทศก่อน --'}
          </option>
          {country && provinces[country]?.map(p => (
            <option key={p} value={p}>{p}</option>
          ))}
        </select>
      </div>
      
      {country && province && (
        <p>เลือก: {countries.find(c => c.code === country)?.name}, {province}</p>
      )}
    </div>
  );
}
```

### Custom Select Component

```jsx
function CustomSelect({ options, value, onChange, placeholder = "เลือก..." }) {
  const [isOpen, setIsOpen] = React.useState(false);
  const containerRef = React.useRef(null);
  
  const selectedOption = options.find(opt => opt.value === value);
  
  // ปิดเมื่อคลิกนอก
  React.useEffect(() => {
    const handleClickOutside = (e) => {
      if (!containerRef.current?.contains(e.target)) {
        setIsOpen(false);
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);
  
  return (
    <div ref={containerRef} style={{ position: 'relative', display: 'inline-block', minWidth: 200 }}>
      <div
        onClick={() => setIsOpen(!isOpen)}
        style={{
          padding: '8px 12px',
          border: '1px solid #ddd',
          borderRadius: 4,
          cursor: 'pointer',
          display: 'flex',
          justifyContent: 'space-between',
          alignItems: 'center',
          backgroundColor: 'white'
        }}
      >
        <span>{selectedOption ? selectedOption.label : placeholder}</span>
        <span>{isOpen ? '▲' : '▼'}</span>
      </div>
      
      {isOpen && (
        <div style={{
          position: 'absolute',
          top: '100%',
          left: 0,
          right: 0,
          border: '1px solid #ddd',
          borderRadius: 4,
          backgroundColor: 'white',
          boxShadow: '0 4px 12px rgba(0,0,0,0.1)',
          zIndex: 100
        }}>
          {options.map(option => (
            <div
              key={option.value}
              onClick={() => {
                onChange(option.value);
                setIsOpen(false);
              }}
              style={{
                padding: '8px 12px',
                cursor: 'pointer',
                backgroundColor: option.value === value ? '#e8f5e9' : 'transparent',
                fontWeight: option.value === value ? 'bold' : 'normal'
              }}
            >
              {option.label}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

// การใช้งาน
const options = [
  { value: 'react', label: 'React' },
  { value: 'vue', label: 'Vue.js' },
  { value: 'angular', label: 'Angular' },
];

<CustomSelect
  options={options}
  value={selected}
  onChange={setSelected}
  placeholder="เลือก Framework"
/>
```

---

## Step 192: Checkbox Component {#step-192}

```jsx
function CheckboxDemo() {
  const [isAgreed, setIsAgreed] = React.useState(false);
  const [permissions, setPermissions] = React.useState({
    read: true,
    write: false,
    execute: false,
    admin: false
  });
  
  const handlePermissionChange = (key) => (e) => {
    const checked = e.target.checked;
    
    // ถ้า uncheck admin, uncheck ทั้งหมด
    if (key === 'admin' && !checked) {
      setPermissions({ read: false, write: false, execute: false, admin: false });
      return;
    }
    
    // ถ้า check admin, check ทั้งหมด
    if (key === 'admin' && checked) {
      setPermissions({ read: true, write: true, execute: true, admin: true });
      return;
    }
    
    setPermissions(prev => ({ ...prev, [key]: checked }));
  };
  
  // ตรวจสอบ indeterminate state
  const checkedCount = Object.values(permissions).filter(Boolean).length;
  const totalCount = Object.keys(permissions).length;
  const allChecked = checkedCount === totalCount;
  const someChecked = checkedCount > 0 && !allChecked;
  
  return (
    <div>
      {/* Simple checkbox */}
      <label style={{ display: 'flex', alignItems: 'center', gap: 8, cursor: 'pointer' }}>
        <input
          type="checkbox"
          checked={isAgreed}
          onChange={(e) => setIsAgreed(e.target.checked)}
          style={{ width: 18, height: 18 }}
        />
        <span>ฉันยอมรับ <a href="/terms">เงื่อนไขการใช้บริการ</a></span>
      </label>
      
      {/* Checkbox group with indeterminate */}
      <div style={{ marginTop: 16 }}>
        <label style={{ display: 'flex', alignItems: 'center', gap: 8, fontWeight: 'bold' }}>
          <input
            type="checkbox"
            checked={allChecked}
            ref={el => el && (el.indeterminate = someChecked)}
            onChange={(e) => setPermissions(
              Object.fromEntries(
                Object.keys(permissions).map(key => [key, e.target.checked])
              )
            )}
          />
          สิทธิ์ทั้งหมด
        </label>
        
        <div style={{ paddingLeft: 24, marginTop: 8 }}>
          {Object.entries(permissions).map(([key, value]) => (
            <label 
              key={key} 
              style={{ display: 'flex', alignItems: 'center', gap: 8, marginBottom: 4, cursor: 'pointer' }}
            >
              <input
                type="checkbox"
                checked={value}
                onChange={handlePermissionChange(key)}
              />
              {key === 'read' ? '📖 อ่าน' :
               key === 'write' ? '✏️ เขียน' :
               key === 'execute' ? '▶️ รัน' : '👑 Admin'}
            </label>
          ))}
        </div>
      </div>
    </div>
  );
}
```

---

## Step 193: Radio Button Component {#step-193}

```jsx
function RadioButtonDemo() {
  const [paymentMethod, setPaymentMethod] = React.useState('credit-card');
  const [deliveryOption, setDeliveryOption] = React.useState('standard');
  
  const paymentMethods = [
    { value: 'credit-card', label: '💳 บัตรเครดิต/เดบิต', description: 'Visa, Mastercard, JCB' },
    { value: 'bank-transfer', label: '🏦 โอนเงินธนาคาร', description: 'SCB, KBank, BBL' },
    { value: 'promptpay', label: '⚡ พร้อมเพย์', description: 'เลขบัตรประชาชน/เบอร์โทร' },
    { value: 'cod', label: '💵 เก็บเงินปลายทาง', description: 'ค่าส่ง +50 บาท' },
  ];
  
  const deliveryOptions = [
    { value: 'standard', label: 'มาตรฐาน', days: '3-5 วัน', price: 'ฟรี' },
    { value: 'express', label: 'ด่วน', days: '1-2 วัน', price: '฿50' },
    { value: 'same-day', label: 'ส่งวันนี้', days: 'ภายใน 4 ชั่วโมง', price: '฿150' },
  ];
  
  return (
    <div>
      {/* Payment Method - Card Style */}
      <h3>วิธีชำระเงิน</h3>
      <div style={{ display: 'flex', flexDirection: 'column', gap: 8 }}>
        {paymentMethods.map(method => (
          <label
            key={method.value}
            style={{
              display: 'flex',
              alignItems: 'center',
              gap: 12,
              padding: 16,
              border: `2px solid ${paymentMethod === method.value ? '#4CAF50' : '#ddd'}`,
              borderRadius: 8,
              cursor: 'pointer',
              backgroundColor: paymentMethod === method.value ? '#f0fff0' : 'white'
            }}
          >
            <input
              type="radio"
              name="payment"
              value={method.value}
              checked={paymentMethod === method.value}
              onChange={(e) => setPaymentMethod(e.target.value)}
            />
            <div>
              <p style={{ margin: 0, fontWeight: 600 }}>{method.label}</p>
              <p style={{ margin: 0, fontSize: 12, color: '#888' }}>{method.description}</p>
            </div>
          </label>
        ))}
      </div>
      
      {/* Delivery Option - Table Style */}
      <h3 style={{ marginTop: 24 }}>การจัดส่ง</h3>
      <table style={{ width: '100%', borderCollapse: 'collapse' }}>
        <thead>
          <tr style={{ backgroundColor: '#f5f5f5' }}>
            <th style={{ padding: 8, textAlign: 'left' }}>เลือก</th>
            <th style={{ padding: 8, textAlign: 'left' }}>ประเภท</th>
            <th style={{ padding: 8, textAlign: 'left' }}>ระยะเวลา</th>
            <th style={{ padding: 8, textAlign: 'left' }}>ค่าส่ง</th>
          </tr>
        </thead>
        <tbody>
          {deliveryOptions.map(option => (
            <tr 
              key={option.value}
              style={{ 
                backgroundColor: deliveryOption === option.value ? '#f0f7ff' : 'white',
                cursor: 'pointer'
              }}
              onClick={() => setDeliveryOption(option.value)}
            >
              <td style={{ padding: 8 }}>
                <input
                  type="radio"
                  name="delivery"
                  value={option.value}
                  checked={deliveryOption === option.value}
                  onChange={(e) => setDeliveryOption(e.target.value)}
                />
              </td>
              <td style={{ padding: 8, fontWeight: 600 }}>{option.label}</td>
              <td style={{ padding: 8 }}>{option.days}</td>
              <td style={{ padding: 8, color: option.price === 'ฟรี' ? '#4CAF50' : '#333' }}>
                {option.price}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Step 194: File Input Component {#step-194}

```jsx
function FileInputDemo() {
  const [selectedFile, setSelectedFile] = React.useState(null);
  const [preview, setPreview] = React.useState(null);
  const [error, setError] = React.useState('');
  const fileInputRef = React.useRef(null);
  
  const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  const MAX_SIZE = 5 * 1024 * 1024; // 5MB
  
  const handleFileChange = (e) => {
    const file = e.target.files[0];
    setError('');
    
    if (!file) {
      setSelectedFile(null);
      setPreview(null);
      return;
    }
    
    // Validate type
    if (!ALLOWED_TYPES.includes(file.type)) {
      setError('รองรับเฉพาะไฟล์ JPG, PNG, GIF, WebP เท่านั้น');
      return;
    }
    
    // Validate size
    if (file.size > MAX_SIZE) {
      setError(`ขนาดไฟล์ต้องไม่เกิน ${MAX_SIZE / 1024 / 1024}MB`);
      return;
    }
    
    setSelectedFile(file);
    
    // Create preview
    const reader = new FileReader();
    reader.onload = (e) => setPreview(e.target.result);
    reader.readAsDataURL(file);
  };
  
  const handleRemove = () => {
    setSelectedFile(null);
    setPreview(null);
    setError('');
    if (fileInputRef.current) fileInputRef.current.value = '';
  };
  
  return (
    <div>
      {/* Hidden file input */}
      <input
        ref={fileInputRef}
        type="file"
        accept="image/*"
        onChange={handleFileChange}
        style={{ display: 'none' }}
      />
      
      {/* Custom upload button */}
      {!selectedFile ? (
        <div
          onClick={() => fileInputRef.current?.click()}
          style={{
            width: 200,
            height: 200,
            border: '2px dashed #ddd',
            borderRadius: 8,
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            cursor: 'pointer',
            backgroundColor: '#fafafa'
          }}
        >
          <span style={{ fontSize: 32 }}>📷</span>
          <span style={{ marginTop: 8, color: '#888' }}>คลิกเพื่อเลือกรูป</span>
          <span style={{ fontSize: 12, color: '#aaa' }}>JPG, PNG, GIF สูงสุด 5MB</span>
        </div>
      ) : (
        <div style={{ position: 'relative', width: 200 }}>
          <img
            src={preview}
            alt="Preview"
            style={{ width: 200, height: 200, objectFit: 'cover', borderRadius: 8 }}
          />
          <button
            onClick={handleRemove}
            style={{
              position: 'absolute',
              top: -8,
              right: -8,
              width: 24,
              height: 24,
              backgroundColor: '#f44336',
              color: 'white',
              border: 'none',
              borderRadius: '50%',
              cursor: 'pointer',
              fontSize: 16,
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center'
            }}
          >
            ✕
          </button>
          <p style={{ margin: '8px 0 0', fontSize: 12, color: '#666' }}>
            {selectedFile.name} ({(selectedFile.size / 1024).toFixed(1)}KB)
          </p>
        </div>
      )}
      
      {error && <p style={{ color: 'red', fontSize: 13 }}>⚠️ {error}</p>}
    </div>
  );
}
```

---

## Step 195: Multiple Inputs ด้วย Generic Handler {#step-195}

```jsx
function GenericHandlerForm() {
  const [formData, setFormData] = React.useState({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    age: '',
    gender: '',
    newsletter: false,
    agree: false,
    plan: 'free',
    bio: ''
  });
  
  // Generic handler ที่จัดการทุก input type
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Text inputs */}
      <input name="firstName" value={formData.firstName} onChange={handleChange} placeholder="ชื่อ" />
      <input name="lastName" value={formData.lastName} onChange={handleChange} placeholder="นามสกุล" />
      <input name="email" type="email" value={formData.email} onChange={handleChange} placeholder="อีเมล" />
      <input name="phone" type="tel" value={formData.phone} onChange={handleChange} placeholder="เบอร์โทร" />
      <input name="age" type="number" value={formData.age} onChange={handleChange} placeholder="อายุ" min="0" max="120" />
      
      {/* Select */}
      <select name="gender" value={formData.gender} onChange={handleChange}>
        <option value="">-- เลือกเพศ --</option>
        <option value="male">ชาย</option>
        <option value="female">หญิง</option>
        <option value="other">อื่นๆ</option>
      </select>
      
      {/* Radio */}
      {['free', 'basic', 'pro'].map(plan => (
        <label key={plan}>
          <input
            type="radio"
            name="plan"
            value={plan}
            checked={formData.plan === plan}
            onChange={handleChange}
          />
          {plan}
        </label>
      ))}
      
      {/* Textarea */}
      <textarea name="bio" value={formData.bio} onChange={handleChange} placeholder="เกี่ยวกับตัวคุณ" />
      
      {/* Checkboxes */}
      <label>
        <input type="checkbox" name="newsletter" checked={formData.newsletter} onChange={handleChange} />
        รับ Newsletter
      </label>
      <label>
        <input type="checkbox" name="agree" checked={formData.agree} onChange={handleChange} />
        ยอมรับเงื่อนไข
      </label>
      
      <button type="submit">บันทึก</button>
      
      <pre style={{ fontSize: 12, backgroundColor: '#f5f5f5', padding: 16 }}>
        {JSON.stringify(formData, null, 2)}
      </pre>
    </form>
  );
}
```

---

## Step 196: Form Validation พื้นฐาน {#step-196}

```jsx
function ValidatedForm() {
  const [formData, setFormData] = React.useState({
    name: '', email: '', password: '', phone: ''
  });
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  
  const validators = {
    name: (value) => {
      if (!value.trim()) return 'กรุณากรอกชื่อ';
      if (value.trim().length < 2) return 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร';
      if (value.trim().length > 50) return 'ชื่อต้องไม่เกิน 50 ตัวอักษร';
      return '';
    },
    email: (value) => {
      if (!value.trim()) return 'กรุณากรอกอีเมล';
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'รูปแบบอีเมลไม่ถูกต้อง';
      return '';
    },
    password: (value) => {
      if (!value) return 'กรุณากรอกรหัสผ่าน';
      if (value.length < 8) return 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
      if (!/(?=.*[A-Z])/.test(value)) return 'ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว';
      if (!/(?=.*[0-9])/.test(value)) return 'ต้องมีตัวเลขอย่างน้อย 1 ตัว';
      return '';
    },
    phone: (value) => {
      if (!value) return '';  // ไม่บังคับ
      if (!/^0\d{9}$/.test(value)) return 'เบอร์โทรต้องเริ่มด้วย 0 และมี 10 หลัก';
      return '';
    }
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Validate เมื่อมี touched
    if (touched[name]) {
      setErrors(prev => ({ ...prev, [name]: validators[name]?.(value) || '' }));
    }
  };
  
  const handleBlur = (e) => {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    setErrors(prev => ({ ...prev, [name]: validators[name]?.(value) || '' }));
  };
  
  const validateAll = () => {
    const newErrors = {};
    Object.keys(validators).forEach(field => {
      newErrors[field] = validators[field](formData[field]);
    });
    setErrors(newErrors);
    setTouched(Object.fromEntries(Object.keys(validators).map(k => [k, true])));
    return Object.values(newErrors).every(e => !e);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateAll()) {
      console.log('Form valid! Submitting:', formData);
    }
  };
  
  const getFieldStatus = (name) => {
    if (!touched[name]) return 'default';
    if (errors[name]) return 'error';
    return 'success';
  };
  
  const fieldStyle = (name) => ({
    border: `2px solid ${
      getFieldStatus(name) === 'error' ? '#f44336' :
      getFieldStatus(name) === 'success' ? '#4CAF50' : '#ddd'
    }`,
    padding: '8px 12px',
    borderRadius: 4,
    width: '100%'
  });
  
  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: 400 }}>
      {['name', 'email', 'password', 'phone'].map(field => (
        <div key={field} style={{ marginBottom: 16 }}>
          <label>
            {field === 'name' ? 'ชื่อ *' :
             field === 'email' ? 'อีเมล *' :
             field === 'password' ? 'รหัสผ่าน *' : 'เบอร์โทร'}
          </label>
          <input
            name={field}
            type={field === 'password' ? 'password' : field === 'email' ? 'email' : 'text'}
            value={formData[field]}
            onChange={handleChange}
            onBlur={handleBlur}
            style={fieldStyle(field)}
          />
          {touched[field] && errors[field] && (
            <p style={{ color: '#f44336', fontSize: 12, margin: '4px 0 0' }}>
              ⚠️ {errors[field]}
            </p>
          )}
          {touched[field] && !errors[field] && formData[field] && (
            <p style={{ color: '#4CAF50', fontSize: 12, margin: '4px 0 0' }}>
              ✅ ถูกต้อง
            </p>
          )}
        </div>
      ))}
      
      <button 
        type="submit"
        style={{ width: '100%', padding: 12, backgroundColor: '#4CAF50', color: 'white', border: 'none', borderRadius: 4 }}
      >
        ลงทะเบียน
      </button>
    </form>
  );
}
```

---

## Step 197: Validation Rules {#step-197}

```jsx
// Validation Library ง่ายๆ
const Validators = {
  required: (message = 'กรุณากรอกข้อมูล') => (value) =>
    !value?.toString().trim() ? message : '',
  
  minLength: (min, message) => (value) =>
    value.length < min ? (message || `ต้องมีอย่างน้อย ${min} ตัวอักษร`) : '',
  
  maxLength: (max, message) => (value) =>
    value.length > max ? (message || `ต้องไม่เกิน ${max} ตัวอักษร`) : '',
  
  email: (message = 'รูปแบบอีเมลไม่ถูกต้อง') => (value) =>
    value && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? message : '',
  
  phone: (message = 'เบอร์โทรต้องมี 10 หลัก') => (value) =>
    value && !/^0\d{9}$/.test(value) ? message : '',
  
  min: (minVal, message) => (value) =>
    Number(value) < minVal ? (message || `ต้องมากกว่า ${minVal}`) : '',
  
  max: (maxVal, message) => (value) =>
    Number(value) > maxVal ? (message || `ต้องน้อยกว่า ${maxVal}`) : '',
  
  pattern: (regex, message = 'รูปแบบไม่ถูกต้อง') => (value) =>
    value && !regex.test(value) ? message : '',
  
  matches: (fieldName, message) => (value, allValues) =>
    value !== allValues[fieldName] ? message : '',
  
  // รวม validators หลายตัว
  compose: (...validators) => (value, allValues) => {
    for (const validator of validators) {
      const error = validator(value, allValues);
      if (error) return error;
    }
    return '';
  }
};

// การใช้งาน
const schema = {
  name: Validators.compose(
    Validators.required(),
    Validators.minLength(2),
    Validators.maxLength(50)
  ),
  email: Validators.compose(
    Validators.required(),
    Validators.email()
  ),
  password: Validators.compose(
    Validators.required(),
    Validators.minLength(8),
    Validators.pattern(/(?=.*[A-Z])(?=.*[0-9])/, 'ต้องมีตัวพิมพ์ใหญ่และตัวเลขอย่างน้อย 1 ตัว')
  ),
  confirmPassword: Validators.matches('password', 'รหัสผ่านไม่ตรงกัน'),
  age: Validators.compose(
    Validators.required(),
    Validators.min(18, 'ต้องมีอายุ 18 ปีขึ้นไป'),
    Validators.max(120, 'กรุณากรอกอายุที่ถูกต้อง')
  )
};
```

---

## Step 198: Real-time Validation {#step-198}

```jsx
function PasswordStrength({ password }) {
  const checks = [
    { label: 'อย่างน้อย 8 ตัวอักษร', test: (p) => p.length >= 8 },
    { label: 'มีตัวพิมพ์เล็ก (a-z)', test: (p) => /[a-z]/.test(p) },
    { label: 'มีตัวพิมพ์ใหญ่ (A-Z)', test: (p) => /[A-Z]/.test(p) },
    { label: 'มีตัวเลข (0-9)', test: (p) => /[0-9]/.test(p) },
    { label: 'มีสัญลักษณ์ (!@#$%)', test: (p) => /[!@#$%^&*]/.test(p) },
  ];
  
  const passed = checks.filter(c => c.test(password)).length;
  const strength = passed === 0 ? 0 : Math.floor((passed / checks.length) * 100);
  
  const getStrengthColor = () => {
    if (strength < 40) return '#f44336';
    if (strength < 70) return '#FF9800';
    return '#4CAF50';
  };
  
  const getStrengthLabel = () => {
    if (strength < 40) return 'อ่อน';
    if (strength < 70) return 'ปานกลาง';
    return 'แข็งแกร่ง';
  };
  
  if (!password) return null;
  
  return (
    <div style={{ marginTop: 8 }}>
      {/* Strength bar */}
      <div style={{ height: 4, backgroundColor: '#e0e0e0', borderRadius: 2, marginBottom: 8 }}>
        <div
          style={{
            height: '100%',
            width: `${strength}%`,
            backgroundColor: getStrengthColor(),
            borderRadius: 2,
            transition: 'width 0.3s, background-color 0.3s'
          }}
        />
      </div>
      <p style={{ fontSize: 12, color: getStrengthColor(), margin: '0 0 8px' }}>
        ความแข็งแกร่ง: {getStrengthLabel()}
      </p>
      
      {/* Checklist */}
      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 4 }}>
        {checks.map(check => (
          <span
            key={check.label}
            style={{
              fontSize: 11,
              color: check.test(password) ? '#4CAF50' : '#ccc',
              display: 'flex',
              alignItems: 'center',
              gap: 4
            }}
          >
            {check.test(password) ? '✅' : '⬜'} {check.label}
          </span>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 199: Form Submission Handling {#step-199}

```jsx
function SubmissionExample() {
  const [formData, setFormData] = React.useState({ name: '', email: '', message: '' });
  const [status, setStatus] = React.useState('idle'); // idle | submitting | success | error
  const [errorMessage, setErrorMessage] = React.useState('');
  
  const handleChange = (e) => {
    setFormData(prev => ({ ...prev, [e.target.name]: e.target.value }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Client-side validation
    if (!formData.name || !formData.email || !formData.message) {
      setStatus('error');
      setErrorMessage('กรุณากรอกข้อมูลให้ครบ');
      return;
    }
    
    setStatus('submitting');
    setErrorMessage('');
    
    try {
      // Mock API call
      const response = await new Promise((resolve, reject) => {
        setTimeout(() => {
          if (Math.random() > 0.2) {
            resolve({ success: true });
          } else {
            reject(new Error('Server error'));
          }
        }, 1500);
      });
      
      setStatus('success');
      setFormData({ name: '', email: '', message: '' });
      
    } catch (error) {
      setStatus('error');
      setErrorMessage(error.message || 'เกิดข้อผิดพลาด กรุณาลองใหม่');
    }
  };
  
  if (status === 'success') {
    return (
      <div style={{ textAlign: 'center', padding: 40 }}>
        <div style={{ fontSize: 64 }}>✅</div>
        <h2>ส่งข้อความสำเร็จ!</h2>
        <p>เราจะติดต่อกลับภายใน 24 ชั่วโมง</p>
        <button onClick={() => setStatus('idle')}>
          ส่งข้อความอีกครั้ง
        </button>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: 400 }}>
      <h2>ติดต่อเรา</h2>
      
      {status === 'error' && errorMessage && (
        <div style={{ backgroundColor: '#ffebee', padding: 12, borderRadius: 4, marginBottom: 16 }}>
          ❌ {errorMessage}
        </div>
      )}
      
      <div style={{ marginBottom: 16 }}>
        <label>ชื่อ *</label>
        <input
          name="name"
          value={formData.name}
          onChange={handleChange}
          disabled={status === 'submitting'}
          style={{ width: '100%', padding: 8 }}
        />
      </div>
      
      <div style={{ marginBottom: 16 }}>
        <label>อีเมล *</label>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          disabled={status === 'submitting'}
          style={{ width: '100%', padding: 8 }}
        />
      </div>
      
      <div style={{ marginBottom: 16 }}>
        <label>ข้อความ *</label>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows={5}
          disabled={status === 'submitting'}
          style={{ width: '100%', padding: 8 }}
        />
      </div>
      
      <button
        type="submit"
        disabled={status === 'submitting'}
        style={{
          width: '100%',
          padding: 12,
          backgroundColor: status === 'submitting' ? '#ccc' : '#4CAF50',
          color: 'white',
          border: 'none',
          borderRadius: 4,
          cursor: status === 'submitting' ? 'not-allowed' : 'pointer'
        }}
      >
        {status === 'submitting' ? '⏳ กำลังส่ง...' : '📨 ส่งข้อความ'}
      </button>
    </form>
  );
}
```

---

## Step 200: Form Reset {#step-200}

```jsx
function FormWithReset() {
  const initialState = {
    name: '',
    email: '',
    bio: '',
    plan: 'free',
    notifications: false
  };
  
  const [formData, setFormData] = React.useState(initialState);
  const [isDirty, setIsDirty] = React.useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    setIsDirty(true);
  };
  
  const handleReset = () => {
    if (isDirty && !window.confirm('ต้องการล้างข้อมูลที่กรอกไว้หรือไม่?')) return;
    setFormData(initialState);
    setIsDirty(false);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', formData);
    setIsDirty(false);
  };
  
  // Warn before leaving if dirty
  React.useEffect(() => {
    const handleBeforeUnload = (e) => {
      if (isDirty) {
        e.preventDefault();
        e.returnValue = 'มีข้อมูลที่ยังไม่ได้บันทึก';
      }
    };
    
    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, [isDirty]);
  
  return (
    <form onSubmit={handleSubmit}>
      {isDirty && (
        <p style={{ color: 'orange', fontSize: 13 }}>
          ⚠️ มีการเปลี่ยนแปลงที่ยังไม่ได้บันทึก
        </p>
      )}
      
      <input name="name" value={formData.name} onChange={handleChange} placeholder="ชื่อ" />
      <input name="email" value={formData.email} onChange={handleChange} placeholder="อีเมล" />
      
      <div style={{ display: 'flex', gap: 8, marginTop: 16 }}>
        <button type="submit">บันทึก</button>
        <button type="button" onClick={handleReset} disabled={!isDirty}>
          ล้างข้อมูล
        </button>
      </div>
    </form>
  );
}
```

---

## Step 201-202: Dependent Fields และ Dynamic Form Fields {#step-201}

```jsx
function DynamicPhoneNumbers() {
  const [phones, setPhones] = React.useState([
    { id: 1, type: 'mobile', number: '' }
  ]);
  
  const phoneTypes = ['mobile', 'home', 'work', 'other'];
  const maxPhones = 5;
  
  const addPhone = () => {
    if (phones.length >= maxPhones) return;
    setPhones(prev => [...prev, {
      id: Date.now(),
      type: 'mobile',
      number: ''
    }]);
  };
  
  const removePhone = (id) => {
    if (phones.length <= 1) return;
    setPhones(prev => prev.filter(p => p.id !== id));
  };
  
  const updatePhone = (id, field, value) => {
    setPhones(prev => prev.map(p =>
      p.id === id ? { ...p, [field]: value } : p
    ));
  };
  
  return (
    <div>
      <h3>เบอร์โทรศัพท์</h3>
      
      {phones.map((phone, index) => (
        <div key={phone.id} style={{ display: 'flex', gap: 8, marginBottom: 8, alignItems: 'center' }}>
          <select
            value={phone.type}
            onChange={(e) => updatePhone(phone.id, 'type', e.target.value)}
            style={{ width: 120 }}
          >
            {phoneTypes.map(type => (
              <option key={type} value={type}>{type}</option>
            ))}
          </select>
          
          <input
            type="tel"
            value={phone.number}
            onChange={(e) => updatePhone(phone.id, 'number', e.target.value)}
            placeholder="เบอร์โทร"
            style={{ flex: 1 }}
          />
          
          {phones.length > 1 && (
            <button
              type="button"
              onClick={() => removePhone(phone.id)}
              style={{ color: 'red', background: 'none', border: 'none', cursor: 'pointer' }}
            >
              🗑️
            </button>
          )}
        </div>
      ))}
      
      {phones.length < maxPhones && (
        <button type="button" onClick={addPhone} style={{ marginTop: 8 }}>
          + เพิ่มเบอร์โทร
        </button>
      )}
    </div>
  );
}
```

---

## Step 203: Multi-step Form {#step-203}

```jsx
function MultiStepForm() {
  const [step, setStep] = React.useState(1);
  const [formData, setFormData] = React.useState({
    // Step 1
    firstName: '', lastName: '', dob: '',
    // Step 2
    email: '', phone: '', address: '',
    // Step 3
    username: '', password: '',
    // Step 4
    preferences: { newsletter: false, notifications: true }
  });
  
  const totalSteps = 4;
  const progress = ((step - 1) / (totalSteps - 1)) * 100;
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    if (name.startsWith('pref_')) {
      const prefKey = name.replace('pref_', '');
      setFormData(prev => ({
        ...prev,
        preferences: { ...prev.preferences, [prefKey]: checked }
      }));
    } else {
      setFormData(prev => ({
        ...prev,
        [name]: type === 'checkbox' ? checked : value
      }));
    }
  };
  
  const steps = [
    {
      title: 'ข้อมูลส่วนตัว',
      fields: (
        <div>
          <input name="firstName" value={formData.firstName} onChange={handleChange} placeholder="ชื่อ *" />
          <input name="lastName" value={formData.lastName} onChange={handleChange} placeholder="นามสกุล *" />
          <input name="dob" type="date" value={formData.dob} onChange={handleChange} />
        </div>
      )
    },
    {
      title: 'ข้อมูลติดต่อ',
      fields: (
        <div>
          <input name="email" type="email" value={formData.email} onChange={handleChange} placeholder="อีเมล *" />
          <input name="phone" type="tel" value={formData.phone} onChange={handleChange} placeholder="เบอร์โทร" />
          <input name="address" value={formData.address} onChange={handleChange} placeholder="ที่อยู่" />
        </div>
      )
    },
    {
      title: 'ข้อมูลบัญชี',
      fields: (
        <div>
          <input name="username" value={formData.username} onChange={handleChange} placeholder="ชื่อผู้ใช้ *" />
          <input name="password" type="password" value={formData.password} onChange={handleChange} placeholder="รหัสผ่าน *" />
        </div>
      )
    },
    {
      title: 'การตั้งค่า',
      fields: (
        <div>
          <label>
            <input type="checkbox" name="pref_newsletter" checked={formData.preferences.newsletter} onChange={handleChange} />
            รับ Newsletter
          </label>
          <label>
            <input type="checkbox" name="pref_notifications" checked={formData.preferences.notifications} onChange={handleChange} />
            รับการแจ้งเตือน
          </label>
        </div>
      )
    }
  ];
  
  const currentStep = steps[step - 1];
  
  return (
    <div style={{ maxWidth: 500, margin: '0 auto' }}>
      {/* Progress */}
      <div style={{ marginBottom: 24 }}>
        <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: 8 }}>
          {steps.map((s, i) => (
            <div key={i} style={{ 
              display: 'flex', 
              flexDirection: 'column', 
              alignItems: 'center',
              flex: 1
            }}>
              <div style={{
                width: 32, height: 32,
                borderRadius: '50%',
                backgroundColor: i + 1 < step ? '#4CAF50' : i + 1 === step ? '#2196F3' : '#ddd',
                color: i + 1 <= step ? 'white' : '#666',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                fontSize: 14,
                fontWeight: 'bold'
              }}>
                {i + 1 < step ? '✓' : i + 1}
              </div>
              <span style={{ fontSize: 11, marginTop: 4, textAlign: 'center' }}>
                {s.title}
              </span>
            </div>
          ))}
        </div>
        <div style={{ height: 4, backgroundColor: '#ddd', borderRadius: 2 }}>
          <div style={{
            height: '100%',
            width: `${progress}%`,
            backgroundColor: '#4CAF50',
            borderRadius: 2,
            transition: 'width 0.3s'
          }} />
        </div>
      </div>
      
      <h2>{currentStep.title}</h2>
      
      <form onSubmit={e => e.preventDefault()}>
        {currentStep.fields}
        
        <div style={{ display: 'flex', gap: 8, marginTop: 24 }}>
          {step > 1 && (
            <button type="button" onClick={() => setStep(s => s - 1)}>
              ← ย้อนกลับ
            </button>
          )}
          
          {step < totalSteps ? (
            <button type="button" onClick={() => setStep(s => s + 1)}>
              ถัดไป →
            </button>
          ) : (
            <button 
              type="button"
              onClick={() => console.log('Submitted:', formData)}
              style={{ backgroundColor: '#4CAF50', color: 'white', padding: '8px 24px' }}
            >
              ✅ ลงทะเบียน
            </button>
          )}
        </div>
      </form>
    </div>
  );
}
```

---

## Step 204-205: Form สำหรับ Edit และ Complex Registration {#step-204}

```jsx
// Form สำหรับ Edit (Pre-filled)
function EditProfileForm({ userId }) {
  const [originalData, setOriginalData] = React.useState(null);
  const [formData, setFormData] = React.useState(null);
  const [isLoading, setIsLoading] = React.useState(true);
  const [isSaving, setIsSaving] = React.useState(false);
  
  const hasChanges = JSON.stringify(formData) !== JSON.stringify(originalData);
  
  React.useEffect(() => {
    // โหลดข้อมูลผู้ใช้
    const loadUser = async () => {
      setIsLoading(true);
      await new Promise(r => setTimeout(r, 800));
      
      const user = {
        name: 'สมชาย ใจดี',
        email: 'somchai@example.com',
        phone: '0812345678',
        bio: 'นักพัฒนา Software',
        website: 'https://somchai.dev'
      };
      
      setOriginalData(user);
      setFormData(user);
      setIsLoading(false);
    };
    
    loadUser();
  }, [userId]);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleReset = () => {
    setFormData(originalData);
  };
  
  const handleSave = async () => {
    setIsSaving(true);
    await new Promise(r => setTimeout(r, 1000));
    setOriginalData(formData);
    setIsSaving(false);
    alert('บันทึกสำเร็จ!');
  };
  
  if (isLoading) return <p>กำลังโหลด...</p>;
  if (!formData) return <p>ไม่พบข้อมูล</p>;
  
  return (
    <form>
      <h2>แก้ไขโปรไฟล์</h2>
      
      {['name', 'email', 'phone', 'bio', 'website'].map(field => (
        <div key={field} style={{ marginBottom: 12 }}>
          <label>{field}</label>
          <input
            name={field}
            value={formData[field] || ''}
            onChange={handleChange}
          />
          {formData[field] !== originalData[field] && (
            <span style={{ fontSize: 11, color: 'orange' }}>● มีการเปลี่ยนแปลง</span>
          )}
        </div>
      ))}
      
      <div style={{ display: 'flex', gap: 8 }}>
        <button
          type="button"
          onClick={handleSave}
          disabled={!hasChanges || isSaving}
          style={{ backgroundColor: '#4CAF50', color: 'white', padding: '8px 16px' }}
        >
          {isSaving ? '⏳ กำลังบันทึก...' : '💾 บันทึก'}
        </button>
        <button
          type="button"
          onClick={handleReset}
          disabled={!hasChanges}
        >
          ↩️ ยกเลิกการเปลี่ยนแปลง
        </button>
      </div>
      
      {hasChanges && (
        <p style={{ fontSize: 12, color: 'orange' }}>⚠️ มีการเปลี่ยนแปลงที่ยังไม่ได้บันทึก</p>
      )}
    </form>
  );
}
```

---

## Step 206: Custom Form Hooks {#step-206}

```jsx
// useForm - Custom Hook สำหรับ form management
function useForm(initialValues, validationSchema = {}) {
  const [values, setValues] = React.useState(initialValues);
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  const [isSubmitting, setIsSubmitting] = React.useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    const fieldValue = type === 'checkbox' ? checked : value;
    
    setValues(prev => ({ ...prev, [name]: fieldValue }));
    
    // Validate on change if already touched
    if (touched[name] && validationSchema[name]) {
      const error = validationSchema[name](fieldValue, { ...values, [name]: fieldValue });
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  };
  
  const handleBlur = (e) => {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    if (validationSchema[name]) {
      const error = validationSchema[name](value, values);
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  };
  
  const validateAll = () => {
    const newErrors = {};
    let isValid = true;
    
    Object.keys(validationSchema).forEach(name => {
      const error = validationSchema[name](values[name], values);
      newErrors[name] = error;
      if (error) isValid = false;
    });
    
    setErrors(newErrors);
    setTouched(Object.fromEntries(Object.keys(validationSchema).map(k => [k, true])));
    
    return isValid;
  };
  
  const handleSubmit = (onSubmit) => async (e) => {
    e.preventDefault();
    
    const isValid = validateAll();
    if (!isValid) return;
    
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };
  
  const setFieldValue = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };
  
  const getFieldProps = (name) => ({
    name,
    value: values[name] ?? '',
    onChange: handleChange,
    onBlur: handleBlur
  });
  
  const getFieldError = (name) => touched[name] ? errors[name] : '';
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    setFieldValue,
    getFieldProps,
    getFieldError,
    isValid: Object.values(errors).every(e => !e)
  };
}

// การใช้งาน useForm
function LoginFormWithHook() {
  const form = useForm(
    { email: '', password: '' },
    {
      email: (v) => !v ? 'กรุณากรอกอีเมล' : !/\S+@\S+\.\S+/.test(v) ? 'อีเมลไม่ถูกต้อง' : '',
      password: (v) => !v ? 'กรุณากรอกรหัสผ่าน' : v.length < 6 ? 'รหัสผ่านต้องมี 6 ตัวขึ้นไป' : ''
    }
  );
  
  const onSubmit = async (values) => {
    await new Promise(r => setTimeout(r, 1000));
    console.log('Login:', values);
  };
  
  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <div>
        <input
          type="email"
          {...form.getFieldProps('email')}
          placeholder="อีเมล"
        />
        {form.getFieldError('email') && (
          <p style={{ color: 'red', fontSize: 12 }}>{form.getFieldError('email')}</p>
        )}
      </div>
      
      <div>
        <input
          type="password"
          {...form.getFieldProps('password')}
          placeholder="รหัสผ่าน"
        />
        {form.getFieldError('password') && (
          <p style={{ color: 'red', fontSize: 12 }}>{form.getFieldError('password')}</p>
        )}
      </div>
      
      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
      </button>
    </form>
  );
}
```

---

## Step 207: Form ที่ Accessible {#step-207}

```jsx
function AccessibleForm() {
  const [values, setValues] = React.useState({ name: '', email: '' });
  const [errors, setErrors] = React.useState({});
  
  const handleChange = (e) => {
    setValues(prev => ({ ...prev, [e.target.name]: e.target.value }));
  };
  
  return (
    <form
      aria-labelledby="form-title"
      noValidate  // ให้ React จัดการ validation เอง
    >
      <h2 id="form-title">ลงทะเบียน</h2>
      
      {/* Accessible Input */}
      <div role="group" aria-labelledby="name-label">
        <label id="name-label" htmlFor="name">
          ชื่อ <span aria-label="จำเป็น" style={{ color: 'red' }}>*</span>
        </label>
        <input
          id="name"
          name="name"
          type="text"
          value={values.name}
          onChange={handleChange}
          required
          aria-required="true"
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? "name-error" : "name-hint"}
          autoComplete="name"
        />
        <span id="name-hint" style={{ fontSize: 12, color: '#666' }}>
          กรอกชื่อจริงของคุณ
        </span>
        {errors.name && (
          <span 
            id="name-error" 
            role="alert"
            style={{ color: 'red', fontSize: 12, display: 'block' }}
          >
            ⚠️ {errors.name}
          </span>
        )}
      </div>
      
      {/* Email Input */}
      <div>
        <label htmlFor="email">
          อีเมล <span aria-label="จำเป็น" style={{ color: 'red' }}>*</span>
        </label>
        <input
          id="email"
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          required
          aria-required="true"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? "email-error" : undefined}
          autoComplete="email"
        />
        {errors.email && (
          <span 
            id="email-error"
            role="alert"
            style={{ color: 'red', fontSize: 12, display: 'block' }}
          >
            ⚠️ {errors.email}
          </span>
        )}
      </div>
      
      {/* Submit button with loading state */}
      <button
        type="submit"
        aria-busy={false}
        aria-label="ลงทะเบียนบัญชีใหม่"
      >
        ลงทะเบียน
      </button>
    </form>
  );
}
```

---

## Step 208: Form Libraries แนะนำ {#step-208}

### Overview

| Library | ขนาด | Learning Curve | Feature |
|---------|------|---------------|---------|
| React Hook Form | เล็ก | ง่าย | Performance, Uncontrolled |
| Formik | กลาง | ปานกลาง | Controlled, เป็นที่นิยม |
| Final Form | เล็ก | ปานกลาง | Subscription-based |

### React Hook Form ตัวอย่าง

```jsx
// npm install react-hook-form
import { useForm } from 'react-hook-form';

function RHFForm() {
  const { 
    register, 
    handleSubmit, 
    formState: { errors, isSubmitting },
    watch,
    reset
  } = useForm();
  
  const password = watch('password');
  
  const onSubmit = async (data) => {
    await new Promise(r => setTimeout(r, 1000));
    console.log(data);
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          {...register('email', {
            required: 'กรุณากรอกอีเมล',
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: 'อีเมลไม่ถูกต้อง'
            }
          })}
          placeholder="อีเมล"
        />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      
      <div>
        <input
          type="password"
          {...register('password', {
            required: 'กรุณากรอกรหัสผ่าน',
            minLength: { value: 8, message: 'ต้องมี 8 ตัวขึ้นไป' }
          })}
          placeholder="รหัสผ่าน"
        />
        {errors.password && <p>{errors.password.message}</p>}
      </div>
      
      <div>
        <input
          type="password"
          {...register('confirmPassword', {
            required: 'กรุณายืนยันรหัสผ่าน',
            validate: (value) => value === password || 'รหัสผ่านไม่ตรงกัน'
          })}
          placeholder="ยืนยันรหัสผ่าน"
        />
        {errors.confirmPassword && <p>{errors.confirmPassword.message}</p>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'กำลังส่ง...' : 'ส่ง'}
      </button>
    </form>
  );
}
```

---

## Step 209: Form Patterns และ Anti-patterns {#step-209}

### Best Patterns

**1. Form State Organization**
```jsx
// ✅ รวม related state
const [formData, setFormData] = React.useState({ name: '', email: '' });
const [errors, setErrors] = React.useState({});
const [status, setStatus] = React.useState('idle');
```

**2. Generic Change Handler**
```jsx
// ✅ handler เดียวสำหรับทุก field
const handleChange = (e) => {
  const { name, value, type, checked } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: type === 'checkbox' ? checked : value
  }));
};
```

**3. Validate on Blur, Show Error on Blur**
```jsx
// ✅ ไม่แสดง error ทันที - แสดงเมื่อ user ออกจาก field
const handleBlur = (e) => {
  const { name, value } = e.target;
  setTouched(prev => ({ ...prev, [name]: true }));
  validateField(name, value);
};
```

### Anti-patterns

**1. ❌ Validate ทุก keystroke**
```jsx
// ❌ UX ไม่ดี - แสดง error ขณะที่ user ยังพิมพ์อยู่
const handleChange = (e) => {
  setValue(e.target.value);
  if (!e.target.value) setError('กรุณากรอก'); // แสดงทันที = ไม่ดี
};
```

**2. ❌ setState หลาย field แยกกัน**
```jsx
// ❌ เยอะเกินไป
const [name, setName] = React.useState('');
const [email, setEmail] = React.useState('');
const [phone, setPhone] = React.useState('');
// ...

// ✅ รวมเป็น object
const [formData, setFormData] = React.useState({ name: '', email: '', phone: '' });
```

**3. ❌ ลืม e.preventDefault()**
```jsx
// ❌ Page จะ reload!
const handleSubmit = () => {
  console.log('submit'); // ไม่มี e.preventDefault()
};
```

---

## Step 210: สรุปและ Best Practices {#step-210}

### Checklist สำหรับ Form ที่ดี

1. **ใช้ Controlled Components** สำหรับ inputs ส่วนใหญ่
2. **Validate ใน blur** ไม่ใช่ทุก keystroke
3. **แสดง error เฉพาะ touched fields**
4. **ป้องกัน double submit** ด้วย isSubmitting state
5. **e.preventDefault()** เสมอใน onSubmit
6. **Label กับ input** ต้องมี htmlFor-id pairing
7. **Error messages** ต้องชัดเจนและมี aria-describedby
8. **Loading state** ให้ feedback ระหว่าง submit
9. **Success state** แสดงผลสำเร็จ
10. **Reset ที่ถูกต้อง** กลับสู่ initialState

### สิ่งที่ต้องจำ

```jsx
// Form Template ที่สมบูรณ์
function CompleteForm() {
  const [formData, setFormData] = React.useState({ name: '', email: '' });
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  const [status, setStatus] = React.useState('idle'); // idle|submitting|success|error
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({ ...prev, [name]: type === 'checkbox' ? checked : value }));
    if (touched[name]) validate(name, value);
  };
  
  const handleBlur = (e) => {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    validate(name, value);
  };
  
  const validate = (name, value) => {
    // validation logic...
    setErrors(prev => ({ ...prev, [name]: /* error */ '' }));
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    // validate all, then submit
    setStatus('submitting');
    try {
      await submitData(formData);
      setStatus('success');
    } catch {
      setStatus('error');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* fields */}
      <button disabled={status === 'submitting'}>
        {status === 'submitting' ? 'กำลังส่ง...' : 'ส่ง'}
      </button>
    </form>
  );
}
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม

1. Controlled vs Uncontrolled Component ต่างกันอย่างไร?
2. ทำไมต้องใช้ `e.preventDefault()` ใน form submit?
3. Validate on Blur ดีกว่า Validate on Change อย่างไร?
4. `touched` state ใช้ทำอะไร?
5. Custom `useForm` hook มีประโยชน์อย่างไร?

### แบบฝึกหัด

**Exercise 1:** สร้าง `JobApplicationForm`:
- ชื่อ, อีเมล, เบอร์โทร
- ตำแหน่งที่สมัคร (select)
- ประสบการณ์ (number, ปี)
- เงินเดือนที่ต้องการ (range slider)
- แนบ Resume (file, PDF only, สูงสุด 2MB)
- Validation ครบทุก field
- Submit แล้วแสดงสรุปข้อมูล

**Exercise 2:** สร้าง `AddressForm` component:
- ที่อยู่, แขวง/ตำบล, เขต/อำเภอ, จังหวัด, รหัสไปรษณีย์
- Dependent: เลือกจังหวัด → เลือกเขต → เลือกตำบล
- Auto-fill รหัสไปรษณีย์เมื่อเลือกตำบล
- Validation ครบ

**Exercise 3:** Implement `useForm` Hook ที่มี:
- `getFieldProps(name)` - คืน props สำหรับ input
- `getFieldError(name)` - คืน error message
- `handleSubmit(onSubmit)` - handle form submission
- `reset()` - reset ทุก field
- `isValid` - true ถ้า form valid

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- Controlled vs Uncontrolled Components
- Input, Textarea, Select, Checkbox, Radio, File inputs
- Generic Change Handler
- Form Validation และ validation rules
- Real-time Validation
- Form Submission พร้อม loading/error/success states
- Form Reset และ Dirty State
- Dependent Fields
- Dynamic Form Fields
- Multi-step Form
- Custom useForm Hook
- Accessible Forms
- Form Libraries (React Hook Form)

---

## อ่านต่อ

➡️ **[Part 11: useEffect และ Side Effects](./part-11-useEffect-and-side-effects.md)**

เราจะเรียนรู้ useEffect Hook, Dependencies, Cleanup Functions, Data Fetching และ common side effect patterns

---

*React Course - Part 10 | สร้างโดย React Course Thailand*
