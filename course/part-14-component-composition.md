# Part 14: Component Composition

## Step 291-315 | ระดับ: กลาง

---

## สารบัญ

- [Step 291: Composition vs Inheritance](#step-291-composition-vs-inheritance)
- [Step 292: Children Props Pattern](#step-292-children-props-pattern)
- [Step 293: Children Props ขั้นสูง](#step-293-children-props-ขั้นสูง)
- [Step 294: Compound Components Pattern](#step-294-compound-components-pattern)
- [Step 295: Slot Pattern](#step-295-slot-pattern)
- [Step 296: Render Props Pattern](#step-296-render-props-pattern)
- [Step 297: Higher-Order Components (HOC)](#step-297-higher-order-components-hoc)
- [Step 298: HOC vs Render Props vs Hooks](#step-298-hoc-vs-render-props-vs-hooks)
- [Step 299: Container/Presentational Pattern](#step-299-containerpresentational-pattern)
- [Step 300: เมื่อไหร่ใช้ Pattern ไหน](#step-300-เมื่อไหร่ใช้-pattern-ไหน)
- [Step 301: ตัวอย่าง Design System](#step-301-ตัวอย่าง-design-system)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 291: Composition vs Inheritance

React แนะนำให้ใช้ **Composition** แทน **Inheritance**

### ทำไมไม่ใช้ Inheritance?

```jsx
// ❌ Inheritance approach (ไม่แนะนำใน React)
class BaseButton extends Component {
  render() {
    return <button>{this.props.children}</button>;
  }
}

class PrimaryButton extends BaseButton {
  render() {
    return (
      <button style={{ background: 'blue', color: 'white' }}>
        {this.props.children}
      </button>
    );
  }
}

// ❌ Inheritance สร้าง tight coupling และยาก extend
class IconPrimaryButton extends PrimaryButton {
  render() {
    return (
      <button style={{ background: 'blue', color: 'white' }}>
        <span>{this.props.icon}</span>
        {this.props.children}
      </button>
    );
  }
}
```

### Composition Approach

```jsx
// ✅ Composition - ยืดหยุ่นกว่า
function Button({ children, style, ...props }) {
  return (
    <button 
      style={{ padding: '8px 16px', borderRadius: '4px', ...style }} 
      {...props}
    >
      {children}
    </button>
  );
}

// Compose โดยใช้ children
function PrimaryButton({ children, ...props }) {
  return (
    <Button style={{ background: 'blue', color: 'white' }} {...props}>
      {children}
    </Button>
  );
}

function IconButton({ icon, children, ...props }) {
  return (
    <Button {...props}>
      <span aria-hidden="true">{icon}</span>
      {children}
    </Button>
  );
}

function IconPrimaryButton({ icon, children, ...props }) {
  return (
    <PrimaryButton {...props}>
      <span aria-hidden="true">{icon}</span>
      {children}
    </PrimaryButton>
  );
}
```

---

## Step 292: Children Props Pattern

`children` คือ prop พิเศษที่รับ content ระหว่าง opening และ closing tags

```jsx
// Layout Components ที่ใช้ children
function Container({ children, maxWidth = '1200px' }) {
  return (
    <div style={{ maxWidth, margin: '0 auto', padding: '0 16px' }}>
      {children}
    </div>
  );
}

function Card({ children, title }) {
  return (
    <div style={{ 
      background: 'white', 
      borderRadius: '8px', 
      boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
      overflow: 'hidden'
    }}>
      {title && (
        <div style={{ padding: '16px', borderBottom: '1px solid #eee' }}>
          <h3 style={{ margin: 0 }}>{title}</h3>
        </div>
      )}
      <div style={{ padding: '16px' }}>
        {children}
      </div>
    </div>
  );
}

function Section({ children, background = 'transparent' }) {
  return (
    <section style={{ background, padding: '60px 0' }}>
      <Container>
        {children}
      </Container>
    </section>
  );
}

// การใช้งาน
function App() {
  return (
    <Section background="#f5f5f5">
      <h1>Welcome to Our Site</h1>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '20px' }}>
        <Card title="Feature 1">
          <p>Description of feature 1</p>
          <button>Learn More</button>
        </Card>
        <Card title="Feature 2">
          <p>Description of feature 2</p>
          <button>Learn More</button>
        </Card>
        <Card title="Feature 3">
          <p>Description of feature 3</p>
          <button>Learn More</button>
        </Card>
      </div>
    </Section>
  );
}
```

---

## Step 293: Children Props ขั้นสูง

### React.Children API

```jsx
import { Children, cloneElement } from 'react';

// RadioGroup ที่ inject value ไปให้ children
function RadioGroup({ children, value, onChange, name }) {
  return (
    <div role="radiogroup">
      {Children.map(children, (child) => {
        if (!child) return null;
        
        // Clone child และเพิ่ม props
        return cloneElement(child, {
          checked: child.props.value === value,
          onChange: () => onChange(child.props.value),
          name
        });
      })}
    </div>
  );
}

function RadioOption({ value, label, checked, onChange, name }) {
  return (
    <label style={{ display: 'flex', alignItems: 'center', gap: '8px', cursor: 'pointer' }}>
      <input
        type="radio"
        value={value}
        checked={checked}
        onChange={onChange}
        name={name}
      />
      {label}
    </label>
  );
}

// การใช้งาน
function App() {
  const [selectedSize, setSelectedSize] = useState('medium');
  
  return (
    <RadioGroup 
      value={selectedSize} 
      onChange={setSelectedSize}
      name="size"
    >
      <RadioOption value="small" label="Small" />
      <RadioOption value="medium" label="Medium" />
      <RadioOption value="large" label="Large" />
    </RadioGroup>
  );
}
```

### Children.count และ Children.toArray

```jsx
function List({ children, emptyMessage = 'No items' }) {
  const count = Children.count(children);
  
  if (count === 0) {
    return <p style={{ color: '#999' }}>{emptyMessage}</p>;
  }
  
  return (
    <ul style={{ listStyle: 'none', padding: 0 }}>
      {Children.map(children, (child, index) => (
        <li key={index} style={{ 
          padding: '8px 0', 
          borderBottom: index < count - 1 ? '1px solid #eee' : 'none' 
        }}>
          {child}
        </li>
      ))}
    </ul>
  );
}

function Tabs({ children, defaultTab = 0 }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  const childArray = Children.toArray(children);
  
  return (
    <div>
      <div style={{ display: 'flex', borderBottom: '1px solid #eee' }}>
        {childArray.map((child, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            style={{
              padding: '8px 16px',
              background: 'none',
              border: 'none',
              borderBottom: activeTab === index ? '2px solid blue' : '2px solid transparent',
              cursor: 'pointer',
              color: activeTab === index ? 'blue' : 'inherit'
            }}
          >
            {child.props.label}
          </button>
        ))}
      </div>
      <div style={{ padding: '16px' }}>
        {childArray[activeTab]}
      </div>
    </div>
  );
}

function Tab({ label, children }) {
  return <div>{children}</div>;
}

// การใช้งาน
function App() {
  return (
    <Tabs defaultTab={0}>
      <Tab label="Overview">
        <h2>Overview Content</h2>
        <p>This is the overview tab.</p>
      </Tab>
      <Tab label="Details">
        <h2>Details Content</h2>
        <p>This is the details tab.</p>
      </Tab>
      <Tab label="Settings">
        <h2>Settings Content</h2>
        <p>This is the settings tab.</p>
      </Tab>
    </Tabs>
  );
}
```

---

## Step 294: Compound Components Pattern

Compound Components เป็น pattern ที่ components ทำงานร่วมกันโดยใช้ Context

```jsx
import { createContext, useContext, useState } from 'react';

// Accordion Compound Component
const AccordionContext = createContext();

function Accordion({ children, allowMultiple = false }) {
  const [openItems, setOpenItems] = useState(new Set());

  const toggleItem = (id) => {
    setOpenItems(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        if (!allowMultiple) next.clear();
        next.add(id);
      }
      return next;
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggleItem }}>
      <div style={{ border: '1px solid #e0e0e0', borderRadius: '8px', overflow: 'hidden' }}>
        {children}
      </div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ children, id }) {
  const { openItems, toggleItem } = useContext(AccordionContext);
  const isOpen = openItems.has(id);

  return (
    <AccordionItemContext.Provider value={{ isOpen, toggle: () => toggleItem(id) }}>
      <div style={{ borderBottom: '1px solid #e0e0e0' }}>
        {children}
      </div>
    </AccordionItemContext.Provider>
  );
}

const AccordionItemContext = createContext();

function AccordionHeader({ children }) {
  const { isOpen, toggle } = useContext(AccordionItemContext);
  
  return (
    <button
      onClick={toggle}
      style={{
        width: '100%',
        padding: '16px',
        background: 'none',
        border: 'none',
        textAlign: 'left',
        cursor: 'pointer',
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center',
        fontSize: '16px',
        fontWeight: '500'
      }}
      aria-expanded={isOpen}
    >
      {children}
      <span style={{ transform: `rotate(${isOpen ? 180 : 0}deg)`, transition: 'transform 0.2s' }}>
        ▼
      </span>
    </button>
  );
}

function AccordionContent({ children }) {
  const { isOpen } = useContext(AccordionItemContext);
  
  if (!isOpen) return null;
  
  return (
    <div style={{ 
      padding: '0 16px 16px',
      color: '#555',
      lineHeight: '1.6'
    }}>
      {children}
    </div>
  );
}

// Attach sub-components
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Content = AccordionContent;

// การใช้งาน
function FAQ() {
  const faqs = [
    { id: 'q1', question: 'React คืออะไร?', answer: 'React เป็น JavaScript library สำหรับสร้าง UI' },
    { id: 'q2', question: 'Hooks คืออะไร?', answer: 'Hooks เป็น functions ที่ให้ใช้ state และ lifecycle ใน function components' },
    { id: 'q3', question: 'Virtual DOM คืออะไร?', answer: 'Virtual DOM คือ representation ของ DOM ใน memory ที่ React ใช้ optimize การ update' }
  ];

  return (
    <div>
      <h2>คำถามที่พบบ่อย</h2>
      <Accordion allowMultiple>
        {faqs.map(faq => (
          <Accordion.Item key={faq.id} id={faq.id}>
            <Accordion.Header>{faq.question}</Accordion.Header>
            <Accordion.Content>{faq.answer}</Accordion.Content>
          </Accordion.Item>
        ))}
      </Accordion>
    </div>
  );
}
```

### Select Compound Component

```jsx
const SelectContext = createContext();

function Select({ children, value, onChange, placeholder = 'Select...' }) {
  const [isOpen, setIsOpen] = useState(false);

  const handleSelect = (optionValue, optionLabel) => {
    onChange(optionValue);
    setIsOpen(false);
  };

  const selectedLabel = (() => {
    let label = placeholder;
    Children.forEach(children, child => {
      if (child?.props?.value === value) {
        label = child.props.children;
      }
    });
    return label;
  })();

  return (
    <SelectContext.Provider value={{ selectedValue: value, onSelect: handleSelect }}>
      <div style={{ position: 'relative', display: 'inline-block', minWidth: '200px' }}>
        <button
          onClick={() => setIsOpen(o => !o)}
          style={{
            width: '100%', padding: '8px 12px',
            border: '1px solid #ccc', borderRadius: '4px',
            background: 'white', cursor: 'pointer',
            display: 'flex', justifyContent: 'space-between', alignItems: 'center'
          }}
        >
          <span>{selectedLabel}</span>
          <span>{isOpen ? '▲' : '▼'}</span>
        </button>
        {isOpen && (
          <div style={{
            position: 'absolute', top: '100%', left: 0, right: 0,
            background: 'white', border: '1px solid #ccc',
            borderRadius: '4px', boxShadow: '0 4px 8px rgba(0,0,0,0.1)',
            zIndex: 100, marginTop: '4px'
          }}>
            {children}
          </div>
        )}
      </div>
    </SelectContext.Provider>
  );
}

function SelectOption({ value, children }) {
  const { selectedValue, onSelect } = useContext(SelectContext);
  const isSelected = value === selectedValue;
  
  return (
    <div
      onClick={() => onSelect(value, children)}
      style={{
        padding: '8px 12px',
        cursor: 'pointer',
        background: isSelected ? '#e8f0fe' : 'white',
        color: isSelected ? '#1a73e8' : 'inherit',
        fontWeight: isSelected ? '600' : 'normal',
        ':hover': { background: '#f5f5f5' }
      }}
    >
      {children}
    </div>
  );
}

Select.Option = SelectOption;

// การใช้งาน
function App() {
  const [country, setCountry] = useState('');
  
  return (
    <div>
      <Select value={country} onChange={setCountry} placeholder="เลือกประเทศ">
        <Select.Option value="th">ไทย</Select.Option>
        <Select.Option value="jp">ญี่ปุ่น</Select.Option>
        <Select.Option value="us">สหรัฐอเมริกา</Select.Option>
        <Select.Option value="uk">สหราชอาณาจักร</Select.Option>
      </Select>
      {country && <p>เลือก: {country}</p>}
    </div>
  );
}
```

---

## Step 295: Slot Pattern

Slot Pattern ให้ parent กำหนด "slots" ที่ child สามารถใส่ content เข้าไป

```jsx
// Layout ที่มี header, sidebar, main, footer slots
function PageLayout({ header, sidebar, children, footer }) {
  return (
    <div style={{ display: 'grid', gridTemplateRows: 'auto 1fr auto', minHeight: '100vh' }}>
      {/* Header slot */}
      <header style={{ background: '#1a1a1a', color: 'white', padding: '16px' }}>
        {header}
      </header>
      
      {/* Body area */}
      <div style={{ display: 'grid', gridTemplateColumns: sidebar ? '250px 1fr' : '1fr' }}>
        {/* Sidebar slot */}
        {sidebar && (
          <aside style={{ background: '#f5f5f5', padding: '20px', borderRight: '1px solid #eee' }}>
            {sidebar}
          </aside>
        )}
        
        {/* Main content slot (children) */}
        <main style={{ padding: '20px' }}>
          {children}
        </main>
      </div>
      
      {/* Footer slot */}
      {footer && (
        <footer style={{ background: '#1a1a1a', color: 'white', padding: '16px', textAlign: 'center' }}>
          {footer}
        </footer>
      )}
    </div>
  );
}

// การใช้งาน
function App() {
  return (
    <PageLayout
      header={
        <nav>
          <h1>My App</h1>
          <div>
            <a href="/">Home</a>
            <a href="/about">About</a>
          </div>
        </nav>
      }
      sidebar={
        <ul style={{ listStyle: 'none', padding: 0 }}>
          <li><a href="/dashboard">Dashboard</a></li>
          <li><a href="/products">Products</a></li>
          <li><a href="/orders">Orders</a></li>
        </ul>
      }
      footer={<p>© 2024 My App</p>}
    >
      <h2>Main Content Here</h2>
      <p>This is the main content area.</p>
    </PageLayout>
  );
}
```

### Dialog ด้วย Slot Pattern

```jsx
function Dialog({ 
  isOpen, 
  onClose, 
  title,
  children,
  footer,
  size = 'md'
}) {
  const sizes = {
    sm: '400px',
    md: '560px',
    lg: '720px',
    xl: '900px'
  };

  if (!isOpen) return null;

  return (
    <div style={{
      position: 'fixed', inset: 0,
      background: 'rgba(0,0,0,0.5)',
      display: 'flex', alignItems: 'center', justifyContent: 'center',
      zIndex: 1000, padding: '20px'
    }}>
      <div style={{
        background: 'white', borderRadius: '8px',
        width: '100%', maxWidth: sizes[size],
        maxHeight: '90vh', display: 'flex', flexDirection: 'column'
      }}>
        {/* Header slot */}
        {title && (
          <div style={{
            padding: '16px 20px',
            borderBottom: '1px solid #eee',
            display: 'flex',
            justifyContent: 'space-between',
            alignItems: 'center'
          }}>
            <h2 style={{ margin: 0, fontSize: '18px' }}>{title}</h2>
            <button onClick={onClose} style={{ background: 'none', border: 'none', fontSize: '20px', cursor: 'pointer' }}>
              ×
            </button>
          </div>
        )}
        
        {/* Body (children) slot */}
        <div style={{ flex: 1, overflow: 'auto', padding: '20px' }}>
          {children}
        </div>
        
        {/* Footer slot */}
        {footer && (
          <div style={{ padding: '16px 20px', borderTop: '1px solid #eee', display: 'flex', justifyContent: 'flex-end', gap: '8px' }}>
            {footer}
          </div>
        )}
      </div>
    </div>
  );
}

// การใช้งาน
function ConfirmDialog({ isOpen, onConfirm, onCancel, message }) {
  return (
    <Dialog
      isOpen={isOpen}
      onClose={onCancel}
      title="ยืนยันการดำเนินการ"
      footer={
        <>
          <button onClick={onCancel}>ยกเลิก</button>
          <button onClick={onConfirm} style={{ background: 'red', color: 'white' }}>
            ยืนยัน
          </button>
        </>
      }
    >
      <p>{message}</p>
    </Dialog>
  );
}
```

---

## Step 296: Render Props Pattern

Render Props เป็น pattern ที่ส่ง function เป็น prop เพื่อ share logic ระหว่าง components

```jsx
// Mouse Tracker ด้วย Render Props
function MouseTracker({ render, children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove} style={{ height: '100vh', position: 'relative' }}>
      {/* รองรับทั้ง render prop และ children as function */}
      {render ? render(position) : (typeof children === 'function' ? children(position) : children)}
    </div>
  );
}

// การใช้งาน 1: render prop
function App1() {
  return (
    <MouseTracker render={({ x, y }) => (
      <div>
        <p>Mouse position: {x}, {y}</p>
        <div style={{
          position: 'absolute',
          left: x - 10, top: y - 10,
          width: 20, height: 20,
          background: 'red',
          borderRadius: '50%',
          pointerEvents: 'none'
        }} />
      </div>
    )} />
  );
}

// การใช้งาน 2: children as function
function App2() {
  return (
    <MouseTracker>
      {({ x, y }) => (
        <p>Position: {x}, {y}</p>
      )}
    </MouseTracker>
  );
}
```

### DataProvider ด้วย Render Props

```jsx
function DataProvider({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    setError(null);
    
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

  return render({ data, loading, error });
}

// การใช้งาน
function UserList() {
  return (
    <DataProvider
      url="https://jsonplaceholder.typicode.com/users"
      render={({ data: users, loading, error }) => {
        if (loading) return <p>Loading users...</p>;
        if (error) return <p>Error: {error}</p>;
        
        return (
          <ul>
            {users?.map(user => (
              <li key={user.id}>{user.name} - {user.email}</li>
            ))}
          </ul>
        );
      }}
    />
  );
}
```

---

## Step 297: Higher-Order Components (HOC)

HOC คือ function ที่รับ component และ return component ใหม่

```jsx
// withLoading HOC
function withLoading(WrappedComponent) {
  function WithLoadingComponent({ isLoading, loadingMessage = 'Loading...', ...props }) {
    if (isLoading) {
      return (
        <div style={{ textAlign: 'center', padding: '40px' }}>
          <div style={{ 
            width: '40px', height: '40px',
            border: '4px solid #f3f3f3',
            borderTop: '4px solid #3498db',
            borderRadius: '50%',
            animation: 'spin 1s linear infinite',
            margin: '0 auto 16px'
          }} />
          <p>{loadingMessage}</p>
        </div>
      );
    }
    
    return <WrappedComponent {...props} />;
  }
  
  WithLoadingComponent.displayName = `WithLoading(${WrappedComponent.displayName || WrappedComponent.name})`;
  
  return WithLoadingComponent;
}

// withAuth HOC
function withAuth(WrappedComponent, { redirectTo = '/login' } = {}) {
  function WithAuthComponent(props) {
    const { user, isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return <Navigate to={redirectTo} />;
    }
    
    return <WrappedComponent {...props} user={user} />;
  }
  
  WithAuthComponent.displayName = `WithAuth(${WrappedComponent.name})`;
  return WithAuthComponent;
}

// withErrorBoundary HOC
function withErrorBoundary(WrappedComponent, FallbackComponent) {
  class WithErrorBoundaryComponent extends Component {
    state = { hasError: false, error: null };
    
    static getDerivedStateFromError(error) {
      return { hasError: true, error };
    }
    
    render() {
      if (this.state.hasError) {
        return FallbackComponent ? (
          <FallbackComponent error={this.state.error} />
        ) : (
          <div>Something went wrong</div>
        );
      }
      return <WrappedComponent {...this.props} />;
    }
  }
  
  return WithErrorBoundaryComponent;
}

// การใช้งาน
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

function App() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  return (
    <UserListWithLoading 
      isLoading={loading}
      users={users}
      loadingMessage="กำลังโหลด Users..."
    />
  );
}
```

### Compose HOCs

```jsx
// Utility function สำหรับ compose HOCs
function compose(...hocs) {
  return (WrappedComponent) => {
    return hocs.reduceRight((component, hoc) => hoc(component), WrappedComponent);
  };
}

// การใช้งาน
const enhance = compose(
  withAuth,
  withLoading,
  withErrorBoundary
);

const EnhancedDashboard = enhance(Dashboard);
```

---

## Step 298: HOC vs Render Props vs Hooks

### เปรียบเทียบ 3 วิธี

```jsx
// 1. HOC approach
function withWindowSize(WrappedComponent) {
  return function WithWindowSizeComponent(props) {
    const [size, setSize] = useState({
      width: window.innerWidth,
      height: window.innerHeight
    });

    useEffect(() => {
      const handleResize = () => setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
      window.addEventListener('resize', handleResize);
      return () => window.removeEventListener('resize', handleResize);
    }, []);

    return <WrappedComponent {...props} windowSize={size} />;
  };
}

// 2. Render Props approach
function WindowSize({ render }) {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => setSize({
      width: window.innerWidth,
      height: window.innerHeight
    });
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return render(size);
}

// 3. Custom Hook approach (แนะนำมากที่สุด)
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => setSize({
      width: window.innerWidth,
      height: window.innerHeight
    });
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// การใช้งาน Custom Hook (ง่ายที่สุด)
function MyComponent() {
  const { width, height } = useWindowSize();
  return <p>{width} x {height}</p>;
}
```

### เมื่อไหร่ใช้อะไร

| Pattern | ใช้เมื่อ |
|---|---|
| **Custom Hooks** | Stateful logic ที่ต้องการ reuse (แนะนำมากที่สุด) |
| **Render Props** | เมื่อต้องการ control rendering อย่างยืดหยุ่น |
| **HOC** | เมื่อต้องการ cross-cutting concerns (auth, logging) |
| **Compound Components** | เมื่อ components มี relationship กัน |
| **Slot Pattern** | เมื่อต้องการ flexible layout |

---

## Step 299: Container/Presentational Pattern

```jsx
// Presentational Component - แสดงผลอย่างเดียว
function UserListPresentation({ users, isLoading, onSelectUser, selectedUserId }) {
  if (isLoading) return <div className="loading">Loading...</div>;
  
  return (
    <div className="user-list">
      {users.map(user => (
        <div
          key={user.id}
          className={`user-item ${user.id === selectedUserId ? 'selected' : ''}`}
          onClick={() => onSelectUser(user.id)}
        >
          <img src={user.avatar} alt={user.name} />
          <div>
            <h3>{user.name}</h3>
            <p>{user.email}</p>
          </div>
        </div>
      ))}
    </div>
  );
}

// Container Component - จัดการ logic และ data
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [selectedUserId, setSelectedUserId] = useState(null);

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setIsLoading(false);
      });
  }, []);

  return (
    <UserListPresentation
      users={users}
      isLoading={isLoading}
      selectedUserId={selectedUserId}
      onSelectUser={setSelectedUserId}
    />
  );
}
```

---

## Step 300: เมื่อไหร่ใช้ Pattern ไหน

```
สถานการณ์                              Pattern ที่แนะนำ
────────────────────────────────────────────────────────────
Reuse stateful logic                    Custom Hooks
Cross-cutting concerns (auth, log)       HOC
Flexible layouts                         Slot Pattern (named props)
Related components                       Compound Components
Share state between siblings             Context API
Generic wrappers                         Children Props
Complex render control                   Render Props
```

---

## Step 301: ตัวอย่าง Design System

```jsx
// Button System
function Button({ 
  variant = 'primary',
  size = 'md',
  isLoading = false,
  leftIcon,
  rightIcon,
  children,
  ...props 
}) {
  const variantStyles = {
    primary: { bg: '#007bff', color: 'white', border: 'none' },
    secondary: { bg: 'white', color: '#007bff', border: '1px solid #007bff' },
    danger: { bg: '#dc3545', color: 'white', border: 'none' },
    ghost: { bg: 'transparent', color: '#555', border: '1px solid #ccc' }
  };

  const sizeStyles = {
    sm: { padding: '4px 10px', fontSize: '12px', height: '28px' },
    md: { padding: '8px 16px', fontSize: '14px', height: '36px' },
    lg: { padding: '12px 24px', fontSize: '16px', height: '44px' }
  };

  const style = {
    ...variantStyles[variant],
    ...sizeStyles[size],
    borderRadius: '6px',
    cursor: props.disabled ? 'not-allowed' : 'pointer',
    opacity: props.disabled ? 0.6 : 1,
    display: 'inline-flex',
    alignItems: 'center',
    gap: '6px',
    fontWeight: '500',
    transition: 'all 0.2s'
  };

  return (
    <button style={style} {...props}>
      {isLoading ? (
        <span>Loading...</span>
      ) : (
        <>
          {leftIcon && <span>{leftIcon}</span>}
          {children}
          {rightIcon && <span>{rightIcon}</span>}
        </>
      )}
    </button>
  );
}

// Form Components
function FormField({ label, error, required, children, helpText }) {
  const id = React.useId();
  
  return (
    <div style={{ marginBottom: '16px' }}>
      {label && (
        <label
          htmlFor={id}
          style={{
            display: 'block',
            marginBottom: '4px',
            fontWeight: '500',
            fontSize: '14px'
          }}
        >
          {label}
          {required && <span style={{ color: 'red', marginLeft: '4px' }}>*</span>}
        </label>
      )}
      {React.cloneElement(children, { id })}
      {helpText && !error && (
        <p style={{ marginTop: '4px', fontSize: '12px', color: '#666' }}>
          {helpText}
        </p>
      )}
      {error && (
        <p style={{ marginTop: '4px', fontSize: '12px', color: '#dc3545' }}>
          {error}
        </p>
      )}
    </div>
  );
}

function Input({ style, error, ...props }) {
  return (
    <input
      style={{
        width: '100%',
        padding: '8px 12px',
        border: `1px solid ${error ? '#dc3545' : '#ccc'}`,
        borderRadius: '4px',
        fontSize: '14px',
        outline: 'none',
        boxSizing: 'border-box',
        ...style
      }}
      {...props}
    />
  );
}

// การใช้งาน Design System
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [isLoading, setIsLoading] = useState(false);

  const validate = () => {
    const newErrors = {};
    if (!email) newErrors.email = 'กรุณาใส่ email';
    if (!email.includes('@')) newErrors.email = 'Email ไม่ถูกต้อง';
    if (!password) newErrors.password = 'กรุณาใส่ password';
    if (password.length < 6) newErrors.password = 'Password ต้องมีอย่างน้อย 6 ตัวอักษร';
    return newErrors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const validationErrors = validate();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setIsLoading(true);
    await new Promise(resolve => setTimeout(resolve, 2000));
    setIsLoading(false);
    alert('Login successful!');
  };

  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: '400px', margin: '40px auto', padding: '32px', background: 'white', borderRadius: '8px', boxShadow: '0 2px 8px rgba(0,0,0,0.1)' }}>
      <h2 style={{ marginBottom: '24px', textAlign: 'center' }}>เข้าสู่ระบบ</h2>
      
      <FormField label="Email" required error={errors.email}>
        <Input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
          error={errors.email}
          placeholder="your@email.com"
        />
      </FormField>
      
      <FormField label="Password" required error={errors.password}>
        <Input
          type="password"
          value={password}
          onChange={e => setPassword(e.target.value)}
          error={errors.password}
          placeholder="••••••••"
        />
      </FormField>
      
      <Button type="submit" isLoading={isLoading} style={{ width: '100%', marginTop: '8px' }}>
        {isLoading ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}
      </Button>
    </form>
  );
}
```

---

## Quiz และแบบฝึกหัด

### Quiz

1. Composition แตกต่างจาก Inheritance อย่างไร?
2. Compound Components ใช้ mechanism อะไรในการแชร์ state?
3. HOC คืออะไรและควรใช้เมื่อไหร่?
4. Render Props แตกต่างจาก Custom Hooks อย่างไร?

### แบบฝึกหัด

**Exercise 1**: สร้าง `Tabs` compound component

```jsx
// TODO: สร้าง Tabs ที่ใช้งานแบบนี้:
<Tabs defaultTab="overview">
  <Tabs.List>
    <Tabs.Tab id="overview">Overview</Tabs.Tab>
    <Tabs.Tab id="details">Details</Tabs.Tab>
    <Tabs.Tab id="reviews">Reviews</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel id="overview">Overview content here...</Tabs.Panel>
    <Tabs.Panel id="details">Details content here...</Tabs.Panel>
    <Tabs.Panel id="reviews">Reviews content here...</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

**Exercise 2**: สร้าง `withLogger` HOC

```jsx
// TODO: HOC ที่ log ทุกครั้งที่ component render
// พร้อม props ที่ถูกส่งมา
function withLogger(WrappedComponent, componentName) {
  // implement here...
}
```

---

➡️ ไปต่อ: [Part 15: Context API](./part-15-context-api.md)
