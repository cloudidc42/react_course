# Part 08: Conditional Rendering
## Step 146-165 | ระดับ: พื้นฐาน-กลาง

---

## สารบัญ (Table of Contents)

- [Step 146: Conditional Rendering คืออะไร?](#step-146)
- [Step 147: If/Else Statement](#step-147)
- [Step 148: Ternary Operator](#step-148)
- [Step 149: Logical AND (&&)](#step-149)
- [Step 150: Logical OR (||)](#step-150)
- [Step 151: Nullish Coalescing (??)](#step-151)
- [Step 152: Return null เพื่อซ่อน Component](#step-152)
- [Step 153: Short-circuit Evaluation ในเชิงลึก](#step-153)
- [Step 154: Switch-based Rendering](#step-154)
- [Step 155: Object Map Rendering](#step-155)
- [Step 156: Enum Pattern](#step-156)
- [Step 157: Loading State Pattern](#step-157)
- [Step 158: Error State Pattern](#step-158)
- [Step 159: Empty State Pattern](#step-159)
- [Step 160: Loading/Error/Empty Combined Pattern](#step-160)
- [Step 161: Feature Flags Pattern](#step-161)
- [Step 162: Role-based Rendering](#step-162)
- [Step 163: Skeleton Loading](#step-163)
- [Step 164: Guard Clauses Pattern](#step-164)
- [Step 165: Best Practices และ Anti-patterns](#step-165)
- [Quiz และแบบฝึกหัด](#quiz)

---

## Step 146: Conditional Rendering คืออะไร? {#step-146}

**Conditional Rendering** คือการแสดง UI ที่แตกต่างกันตามเงื่อนไข เหมือน if/else ใน JavaScript แต่ใช้ใน JSX

### ทำไมต้องมี Conditional Rendering?

UI ของ Web App มักเปลี่ยนแปลงตามสถานะ เช่น:
- ผู้ใช้ Login แล้วหรือยัง?
- กำลังโหลดข้อมูลอยู่หรือไม่?
- มีข้อมูลให้แสดงหรือไม่?
- ผู้ใช้มีสิทธิ์เข้าถึงหน้านี้หรือไม่?

```jsx
// แนวคิดพื้นฐาน
function UserGreeting({ isLoggedIn, username }) {
  if (isLoggedIn) {
    return <h1>สวัสดี, {username}!</h1>;
  } else {
    return <h1>กรุณาเข้าสู่ระบบ</h1>;
  }
}
```

### วิธี Conditional Rendering ใน React

1. **if/else** - เหมาะกับ logic ซับซ้อน
2. **Ternary (`? :`)** - เหมาะกับ 2 ทางเลือก
3. **Logical AND (`&&`)** - เหมาะกับ show/hide
4. **Logical OR (`||`)** - เหมาะกับ fallback value
5. **Nullish Coalescing (`??`)** - เหมาะกับ null/undefined
6. **Switch** - เหมาะกับหลาย case
7. **Object Map** - เหมาะกับ enum-like state

---

## Step 147: If/Else Statement {#step-147}

วิธีดั้งเดิมและอ่านง่ายที่สุด แต่ใช้ใน JSX โดยตรงไม่ได้

```jsx
// วิธีที่ 1: แยก return แบบ early return
function UserStatus({ status }) {
  if (status === 'loading') {
    return <div>⏳ กำลังโหลด...</div>;
  }
  
  if (status === 'error') {
    return <div>❌ เกิดข้อผิดพลาด</div>;
  }
  
  if (status === 'empty') {
    return <div>📭 ไม่มีข้อมูล</div>;
  }
  
  return <div>✅ พร้อมใช้งาน</div>;
}
```

```jsx
// วิธีที่ 2: เก็บ JSX ในตัวแปร
function Dashboard({ isAdmin, userData }) {
  let header;
  let content;
  
  if (isAdmin) {
    header = <h1>Admin Dashboard</h1>;
    content = <AdminPanel />;
  } else {
    header = <h1>User Dashboard</h1>;
    content = <UserPanel />;
  }
  
  return (
    <div>
      {header}
      <Navigation isAdmin={isAdmin} />
      {content}
    </div>
  );
}
```

```jsx
// วิธีที่ 3: Function ที่ return JSX
function Greeting({ user }) {
  const renderGreeting = () => {
    if (!user) return null;
    
    if (user.isNewUser) {
      return (
        <div className="welcome-banner">
          <h2>ยินดีต้อนรับ, {user.name}!</h2>
          <p>เริ่มต้นการเดินทางของคุณกับเราได้เลย</p>
          <button>เริ่มต้นใช้งาน</button>
        </div>
      );
    }
    
    const daysSince = Math.floor((Date.now() - user.lastLogin) / (1000 * 60 * 60 * 24));
    
    if (daysSince > 30) {
      return (
        <div className="returning-banner">
          <h2>ยินดีต้อนรับกลับมา, {user.name}!</h2>
          <p>คุณไม่ได้มาเยี่ยมเราเป็นเวลา {daysSince} วัน</p>
        </div>
      );
    }
    
    return <p>สวัสดี, {user.name}!</p>;
  };
  
  return (
    <div>
      {renderGreeting()}
      <main>เนื้อหาหลัก</main>
    </div>
  );
}
```

### เมื่อไหรใช้ if/else

- Logic มีหลาย branch
- แต่ละ branch return component ที่ต่างกันมาก
- ต้องการ early return เพื่อความชัดเจน

---

## Step 148: Ternary Operator {#step-148}

Ternary `condition ? trueValue : falseValue` เหมาะกับ inline condition

```jsx
function ToggleButton({ isOn, onToggle }) {
  return (
    <button 
      onClick={onToggle}
      style={{ backgroundColor: isOn ? '#4CAF50' : '#f44336' }}
    >
      {isOn ? '✅ เปิด' : '❌ ปิด'}
    </button>
  );
}
```

### Ternary กับ Complex JSX

```jsx
function UserCard({ user, isLoading }) {
  return (
    <div className="card">
      {isLoading ? (
        <div className="skeleton">
          <div className="skeleton-avatar" />
          <div className="skeleton-text" />
        </div>
      ) : (
        <div className="user-info">
          <img src={user.avatar} alt={user.name} />
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      )}
    </div>
  );
}
```

### Nested Ternary (ใช้ด้วยความระมัดระวัง)

```jsx
// ❌ อ่านยาก - nested ternary
function StatusBadge({ status }) {
  return (
    <span className={
      status === 'active' ? 'badge-green' :
      status === 'pending' ? 'badge-yellow' :
      status === 'inactive' ? 'badge-red' :
      'badge-gray'
    }>
      {status === 'active' ? 'ใช้งาน' :
       status === 'pending' ? 'รอดำเนินการ' :
       status === 'inactive' ? 'ไม่ใช้งาน' :
       'ไม่ทราบสถานะ'}
    </span>
  );
}

// ✅ ดีกว่า - ใช้ object map แทน
const STATUS_CONFIG = {
  active: { className: 'badge-green', label: 'ใช้งาน' },
  pending: { className: 'badge-yellow', label: 'รอดำเนินการ' },
  inactive: { className: 'badge-red', label: 'ไม่ใช้งาน' },
};

function StatusBadge({ status }) {
  const config = STATUS_CONFIG[status] || { className: 'badge-gray', label: 'ไม่ทราบสถานะ' };
  
  return (
    <span className={config.className}>
      {config.label}
    </span>
  );
}
```

### เมื่อไหรใช้ Ternary

- มี 2 ทางเลือก (true/false)
- JSX ที่แสดงไม่ซับซ้อนมาก
- Inline ใน JSX

> **Warning:** ถ้า ternary มากกว่า 2 ระดับ nested ให้พิจารณาเปลี่ยนวิธีอื่น

---

## Step 149: Logical AND (&&) {#step-149}

`condition && element` - แสดง element เมื่อ condition เป็น truthy

```jsx
function NotificationBadge({ count }) {
  return (
    <div>
      <span>🔔</span>
      {count > 0 && (
        <span className="badge">{count}</span>
      )}
    </div>
  );
}
```

### ตัวอย่างการใช้งาน

```jsx
function UserProfile({ user, isOwner, hasPermission }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
      
      {/* แสดงเฉพาะเจ้าของ profile */}
      {isOwner && (
        <button>แก้ไขโปรไฟล์</button>
      )}
      
      {/* แสดงเมื่อมีสิทธิ์ */}
      {hasPermission && (
        <div className="admin-tools">
          <button>แบน</button>
          <button>ลบบัญชี</button>
        </div>
      )}
      
      {/* แสดงเมื่อ user มี posts */}
      {user.posts && user.posts.length > 0 && (
        <div>
          <h3>โพสต์ ({user.posts.length})</h3>
          {user.posts.map(post => (
            <div key={post.id}>{post.title}</div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### ⚠️ Gotcha: Number 0 กับ &&

```jsx
// ❌ Bug! - render "0" ใน DOM
function ItemList({ items }) {
  return (
    <div>
      {items.length && <ul>...</ul>}
      {/* ถ้า items.length = 0, จะ render เลข "0"! */}
    </div>
  );
}

// ✅ วิธีแก้ - ใช้ boolean explicitly
function ItemList({ items }) {
  return (
    <div>
      {items.length > 0 && <ul>...</ul>}
      {/* หรือ */}
      {!!items.length && <ul>...</ul>}
      {/* หรือ ternary */}
      {items.length ? <ul>...</ul> : null}
    </div>
  );
}
```

> **Warning:** อย่าใช้ number กับ `&&` โดยตรง เพราะ `0` เป็น falsy แต่ React ยัง render ตัวเลข 0 ออกมา!

---

## Step 150: Logical OR (||) {#step-150}

`value || fallback` - ใช้ fallback เมื่อ value เป็น falsy

```jsx
function UserAvatar({ name, avatarUrl }) {
  // ถ้าไม่มี avatarUrl ให้ใช้ default
  const src = avatarUrl || '/default-avatar.png';
  const displayName = name || 'ผู้ใช้ไม่ระบุชื่อ';
  
  return (
    <div>
      <img src={src} alt={displayName} />
      <p>{displayName}</p>
    </div>
  );
}
```

```jsx
function ProductTitle({ title, subtitle, description }) {
  return (
    <div>
      <h1>{title || 'ไม่มีชื่อสินค้า'}</h1>
      <h2>{subtitle || title || 'ไม่มีชื่อย่อย'}</h2>
      <p>{description || 'ยังไม่มีคำอธิบาย'}</p>
    </div>
  );
}
```

### ความแตกต่างระหว่าง || และ ??

```jsx
// || = falsy check (0, '', false, null, undefined)
// ?? = nullish check (null, undefined เท่านั้น)

const count = 0;
const name = '';

console.log(count || 10);    // 10  (0 เป็น falsy)
console.log(count ?? 10);    // 0   (0 ไม่ใช่ null/undefined)

console.log(name || 'ไม่ระบุ');   // 'ไม่ระบุ'  ('' เป็น falsy)
console.log(name ?? 'ไม่ระบุ');   // ''          ('' ไม่ใช่ null/undefined)
```

---

## Step 151: Nullish Coalescing (??) {#step-151}

`value ?? fallback` - ใช้ fallback เฉพาะเมื่อ value เป็น `null` หรือ `undefined`

```jsx
function ScoreDisplay({ score, maxScore }) {
  // score อาจเป็น 0 (valid) หรือ null (ไม่มีข้อมูล)
  return (
    <div>
      {/* ✅ ?? จะแสดง "ยังไม่มีคะแนน" เฉพาะเมื่อ score เป็น null/undefined */}
      <p>คะแนน: {score ?? 'ยังไม่มีคะแนน'}</p>
      
      {/* ❌ || จะแสดง "ยังไม่มีคะแนน" เมื่อ score เป็น 0 ด้วย! */}
      {/* <p>คะแนน: {score || 'ยังไม่มีคะแนน'}</p> */}
      
      {/* Chaining ?? */}
      <p>สูงสุด: {maxScore ?? score ?? 100}</p>
    </div>
  );
}

// ตัวอย่างการใช้งาน
<ScoreDisplay score={0} />         // แสดง: คะแนน: 0
<ScoreDisplay score={null} />      // แสดง: คะแนน: ยังไม่มีคะแนน
<ScoreDisplay score={undefined} /> // แสดง: คะแนน: ยังไม่มีคะแนน
```

### Optional Chaining กับ ??

```jsx
function UserInfo({ user }) {
  return (
    <div>
      {/* Optional chaining + nullish coalescing */}
      <p>{user?.profile?.displayName ?? user?.name ?? 'ผู้ใช้'}</p>
      <p>{user?.address?.city ?? 'ไม่ระบุเมือง'}</p>
      <p>{user?.subscription?.tier ?? 'Free'}</p>
    </div>
  );
}
```

---

## Step 152: Return null เพื่อซ่อน Component {#step-152}

การ return `null` จาก component จะไม่ render อะไรเลย แต่ component ยังอยู่ใน tree

```jsx
// Component ที่ซ่อนตัวเองได้
function Banner({ show, message, type = 'info' }) {
  // Return null = ไม่ render อะไร
  if (!show) return null;
  
  const colors = {
    info: '#e3f2fd',
    success: '#e8f5e9',
    warning: '#fff3e0',
    error: '#ffebee'
  };
  
  return (
    <div 
      style={{ 
        backgroundColor: colors[type],
        padding: '12px 16px',
        borderRadius: 4,
        marginBottom: 16
      }}
    >
      {message}
    </div>
  );
}

function App() {
  const [showBanner, setShowBanner] = useState(true);
  
  return (
    <div>
      <Banner 
        show={showBanner} 
        message="ข้อความสำคัญ!" 
        type="warning"
      />
      <button onClick={() => setShowBanner(false)}>
        ปิด Banner
      </button>
    </div>
  );
}
```

### null vs display:none vs unmount

```jsx
function VisibilityComparison() {
  const [show1, setShow1] = useState(true);
  const [show2, setShow2] = useState(true);
  const [show3, setShow3] = useState(true);
  
  return (
    <div>
      {/* วิธีที่ 1: Return null - unmount component */}
      {show1 && <HeavyComponent id="1" />}
      
      {/* วิธีที่ 2: CSS display:none - component ยังอยู่ใน DOM */}
      <div style={{ display: show2 ? 'block' : 'none' }}>
        <HeavyComponent id="2" />
      </div>
      
      {/* วิธีที่ 3: CSS visibility:hidden - ซ่อนแต่ยังกินพื้นที่ */}
      <div style={{ visibility: show3 ? 'visible' : 'hidden' }}>
        <HeavyComponent id="3" />
      </div>
    </div>
  );
}

// เลือกตามสถานการณ์:
// Return null = ไม่ต้องการ component ในหน่วยความจำ
// display:none = ต้องการให้ component คง state ไว้
// visibility:hidden = ต้องการให้ layout ไม่เปลี่ยน
```

---

## Step 153: Short-circuit Evaluation ในเชิงลึก {#step-153}

```jsx
// Short-circuit evaluation ทำงานอย่างไร?
// A && B = ถ้า A เป็น falsy คืน A, ถ้า A truthy คืน B
// A || B = ถ้า A เป็น truthy คืน A, ถ้า A falsy คืน B

function ShortCircuitExamples() {
  const user = null;
  const items = [];
  const count = 0;
  const name = '';
  
  return (
    <div>
      {/* null && element = null (ไม่ render) */}
      {user && <UserCard user={user} />}
      
      {/* false && element = false (ไม่ render) */}
      {false && <div>ไม่แสดง</div>}
      
      {/* ⚠️ 0 && element = 0 (render "0"!) */}
      {count && <p>Count</p>}
      
      {/* ✅ แก้โดยใช้ > 0 */}
      {count > 0 && <p>Count: {count}</p>}
      
      {/* '' && element = '' (render empty string) */}
      {name && <p>Name: {name}</p>}
      
      {/* Chaining */}
      {user && user.isActive && user.hasPermission && (
        <AdminPanel />
      )}
      
      {/* Optional chaining */}
      {user?.profile?.isComplete && (
        <ProfileBadge />
      )}
    </div>
  );
}
```

### Complex Short-circuit Patterns

```jsx
function ComplexConditions({ user, data, config }) {
  return (
    <div>
      {/* หลาย conditions */}
      {user && data && config && (
        <Dashboard user={user} data={data} config={config} />
      )}
      
      {/* Condition กับ function call */}
      {isAuthenticated() && hasRole('admin') && (
        <AdminButton />
      )}
      
      {/* Inline expressions */}
      {!user && (
        <p>กรุณา <a href="/login">เข้าสู่ระบบ</a></p>
      )}
      
      {/* Multiple elements */}
      {user?.isPremium && (
        <>
          <PremiumBadge />
          <PremiumFeatures />
          <SpecialOffers />
        </>
      )}
    </div>
  );
}
```

---

## Step 154: Switch-based Rendering {#step-154}

Switch เหมาะกับหลาย case ที่ชัดเจน

```jsx
// วิธีที่ 1: Switch ใน render function
function PageRenderer({ page }) {
  const renderPage = () => {
    switch (page) {
      case 'home':
        return <HomePage />;
      case 'about':
        return <AboutPage />;
      case 'contact':
        return <ContactPage />;
      case 'blog':
        return <BlogPage />;
      default:
        return <NotFoundPage />;
    }
  };
  
  return (
    <div>
      <Header currentPage={page} />
      {renderPage()}
      <Footer />
    </div>
  );
}
```

```jsx
// วิธีที่ 2: IIFE (Immediately Invoked Function Expression)
function StatusMessage({ status, errorMessage }) {
  return (
    <div>
      {(() => {
        switch (status) {
          case 'idle':
            return <p>รอดำเนินการ</p>;
          case 'loading':
            return <Spinner />;
          case 'success':
            return <p>✅ สำเร็จ</p>;
          case 'error':
            return <p>❌ {errorMessage}</p>;
          default:
            return null;
        }
      })()}
    </div>
  );
}
```

---

## Step 155: Object Map Rendering {#step-155}

Object Map เป็น pattern ที่สะอาดกว่า switch สำหรับ enum-like values

```jsx
// Object Map แทน switch
const ICON_MAP = {
  home: '🏠',
  user: '👤',
  settings: '⚙️',
  notification: '🔔',
  message: '💬',
};

function Icon({ name, size = 24 }) {
  return (
    <span style={{ fontSize: size }}>
      {ICON_MAP[name] || '❓'}
    </span>
  );
}
```

```jsx
// Component Map
const VIEW_COMPONENTS = {
  list: ListView,
  grid: GridView,
  table: TableView,
  calendar: CalendarView,
};

function DataDisplay({ viewType, data }) {
  const ViewComponent = VIEW_COMPONENTS[viewType] || ListView;
  
  return <ViewComponent data={data} />;
}
```

```jsx
// Status Badge ด้วย Object Map
const STATUS_CONFIG = {
  active: {
    label: 'ใช้งาน',
    color: '#4CAF50',
    icon: '✅',
    bgColor: '#e8f5e9'
  },
  pending: {
    label: 'รอดำเนินการ',
    color: '#FF9800',
    icon: '⏳',
    bgColor: '#fff3e0'
  },
  inactive: {
    label: 'ไม่ใช้งาน',
    color: '#9E9E9E',
    icon: '⏸️',
    bgColor: '#f5f5f5'
  },
  suspended: {
    label: 'ถูกระงับ',
    color: '#F44336',
    icon: '🚫',
    bgColor: '#ffebee'
  }
};

function StatusBadge({ status }) {
  const config = STATUS_CONFIG[status] || {
    label: status,
    color: '#666',
    icon: '❓',
    bgColor: '#f5f5f5'
  };
  
  return (
    <span
      style={{
        display: 'inline-flex',
        alignItems: 'center',
        gap: 4,
        padding: '2px 8px',
        borderRadius: 12,
        backgroundColor: config.bgColor,
        color: config.color,
        fontSize: 12,
        fontWeight: 600
      }}
    >
      {config.icon} {config.label}
    </span>
  );
}
```

---

## Step 156: Enum Pattern {#step-156}

สร้าง enum เพื่อจัดการ state ที่มีค่าจำกัด

```jsx
// สร้าง enum-like constants
const FETCH_STATUS = {
  IDLE: 'idle',
  LOADING: 'loading',
  SUCCESS: 'success',
  ERROR: 'error',
};

const VIEW_MODE = {
  LIST: 'list',
  GRID: 'grid',
  MAP: 'map',
};

function DataPage() {
  const [status, setStatus] = useState(FETCH_STATUS.IDLE);
  const [viewMode, setViewMode] = useState(VIEW_MODE.LIST);
  const [data, setData] = useState([]);
  const [error, setError] = useState(null);
  
  const fetchData = async () => {
    setStatus(FETCH_STATUS.LOADING);
    
    try {
      const result = await fetch('/api/data').then(r => r.json());
      setData(result);
      setStatus(FETCH_STATUS.SUCCESS);
    } catch (err) {
      setError(err.message);
      setStatus(FETCH_STATUS.ERROR);
    }
  };
  
  // Render based on status
  const renderContent = () => {
    switch (status) {
      case FETCH_STATUS.IDLE:
        return (
          <div>
            <p>กดปุ่มเพื่อโหลดข้อมูล</p>
            <button onClick={fetchData}>โหลดข้อมูล</button>
          </div>
        );
      
      case FETCH_STATUS.LOADING:
        return <LoadingSpinner />;
      
      case FETCH_STATUS.SUCCESS:
        return (
          <div>
            <ViewToggle mode={viewMode} onChange={setViewMode} />
            {viewMode === VIEW_MODE.LIST && <ListView items={data} />}
            {viewMode === VIEW_MODE.GRID && <GridView items={data} />}
            {viewMode === VIEW_MODE.MAP && <MapView items={data} />}
          </div>
        );
      
      case FETCH_STATUS.ERROR:
        return (
          <ErrorDisplay 
            message={error}
            onRetry={fetchData}
          />
        );
      
      default:
        return null;
    }
  };
  
  return (
    <div>
      <h1>ข้อมูล</h1>
      {renderContent()}
    </div>
  );
}
```

---

## Step 157: Loading State Pattern {#step-157}

```jsx
// Loading Spinner Component
function LoadingSpinner({ size = 40, color = '#4CAF50' }) {
  return (
    <div style={{ 
      display: 'flex', 
      justifyContent: 'center', 
      alignItems: 'center',
      padding: 40
    }}>
      <div
        style={{
          width: size,
          height: size,
          border: `4px solid #f3f3f3`,
          borderTop: `4px solid ${color}`,
          borderRadius: '50%',
          animation: 'spin 1s linear infinite'
        }}
      />
      <style>{`
        @keyframes spin {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
        }
      `}</style>
    </div>
  );
}

// Full Loading Overlay
function LoadingOverlay({ isLoading, children, message = 'กำลังโหลด...' }) {
  return (
    <div style={{ position: 'relative' }}>
      {isLoading && (
        <div style={{
          position: 'absolute',
          top: 0,
          left: 0,
          right: 0,
          bottom: 0,
          backgroundColor: 'rgba(255,255,255,0.8)',
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          zIndex: 10
        }}>
          <LoadingSpinner />
          <p style={{ marginTop: 8, color: '#666' }}>{message}</p>
        </div>
      )}
      {children}
    </div>
  );
}

// Button Loading State
function LoadingButton({ isLoading, onClick, children, loadingText = 'กำลังดำเนินการ...' }) {
  return (
    <button 
      onClick={onClick}
      disabled={isLoading}
      style={{
        display: 'flex',
        alignItems: 'center',
        gap: 8,
        padding: '8px 16px',
        opacity: isLoading ? 0.7 : 1,
        cursor: isLoading ? 'not-allowed' : 'pointer'
      }}
    >
      {isLoading && (
        <span style={{
          display: 'inline-block',
          width: 16,
          height: 16,
          border: '2px solid currentColor',
          borderTopColor: 'transparent',
          borderRadius: '50%',
          animation: 'spin 0.8s linear infinite'
        }} />
      )}
      {isLoading ? loadingText : children}
    </button>
  );
}
```

---

## Step 158: Error State Pattern {#step-158}

```jsx
// Error Display Component
function ErrorDisplay({ 
  message = 'เกิดข้อผิดพลาด', 
  details,
  onRetry,
  showDetails = false 
}) {
  const [isDetailsOpen, setIsDetailsOpen] = useState(false);
  
  return (
    <div style={{
      backgroundColor: '#ffebee',
      border: '1px solid #ef9a9a',
      borderRadius: 8,
      padding: 16,
      textAlign: 'center'
    }}>
      <div style={{ fontSize: 48 }}>❌</div>
      <h3 style={{ color: '#c62828', margin: '8px 0' }}>{message}</h3>
      
      {details && showDetails && (
        <div>
          <button 
            onClick={() => setIsDetailsOpen(!isDetailsOpen)}
            style={{ 
              background: 'none', 
              border: 'none', 
              color: '#888', 
              cursor: 'pointer',
              fontSize: 12
            }}
          >
            {isDetailsOpen ? '▲ ซ่อนรายละเอียด' : '▼ ดูรายละเอียด'}
          </button>
          
          {isDetailsOpen && (
            <pre style={{
              backgroundColor: '#fff',
              padding: 8,
              borderRadius: 4,
              fontSize: 12,
              textAlign: 'left',
              overflow: 'auto',
              maxHeight: 100
            }}>
              {details}
            </pre>
          )}
        </div>
      )}
      
      {onRetry && (
        <button
          onClick={onRetry}
          style={{
            marginTop: 12,
            padding: '8px 24px',
            backgroundColor: '#c62828',
            color: 'white',
            border: 'none',
            borderRadius: 4,
            cursor: 'pointer'
          }}
        >
          🔄 ลองใหม่
        </button>
      )}
    </div>
  );
}

// Error Boundary Component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <ErrorDisplay 
          message="เกิดข้อผิดพลาดที่ไม่คาดคิด"
          details={this.state.error?.message}
          showDetails={process.env.NODE_ENV === 'development'}
          onRetry={() => this.setState({ hasError: false, error: null })}
        />
      );
    }
    
    return this.props.children;
  }
}

// การใช้งาน
<ErrorBoundary>
  <SomeRiskyComponent />
</ErrorBoundary>
```

---

## Step 159: Empty State Pattern {#step-159}

```jsx
function EmptyState({ 
  icon = '📭',
  title = 'ไม่มีข้อมูล',
  description,
  action
}) {
  return (
    <div style={{
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center',
      padding: '60px 20px',
      textAlign: 'center',
      color: '#666'
    }}>
      <div style={{ fontSize: 64, marginBottom: 16 }}>{icon}</div>
      <h3 style={{ margin: '0 0 8px', color: '#333' }}>{title}</h3>
      {description && <p style={{ margin: '0 0 24px', maxWidth: 300 }}>{description}</p>}
      {action && action}
    </div>
  );
}

// ตัวอย่างการใช้งาน
function TodoList({ todos, onAdd }) {
  if (todos.length === 0) {
    return (
      <EmptyState
        icon="✅"
        title="ยังไม่มีรายการ"
        description="เพิ่มรายการแรกของคุณได้เลย!"
        action={
          <button 
            onClick={onAdd}
            style={{
              padding: '10px 24px',
              backgroundColor: '#4CAF50',
              color: 'white',
              border: 'none',
              borderRadius: 4,
              cursor: 'pointer'
            }}
          >
            + เพิ่มรายการ
          </button>
        }
      />
    );
  }
  
  return (
    <ul>
      {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}

// Search Empty State
function SearchResults({ query, results }) {
  if (!query) return null;
  
  if (results.length === 0) {
    return (
      <EmptyState
        icon="🔍"
        title="ไม่พบผลลัพธ์"
        description={`ไม่พบข้อมูลที่ตรงกับ "${query}" ลองค้นหาด้วยคำอื่น`}
      />
    );
  }
  
  return (
    <div>
      <p>พบ {results.length} ผลลัพธ์</p>
      {results.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}
```

---

## Step 160: Loading/Error/Empty Combined Pattern {#step-160}

Pattern ที่สมบูรณ์สำหรับจัดการทุกสถานะ

```jsx
function AsyncContent({ url, renderItem, emptyMessage = 'ไม่มีข้อมูล' }) {
  const [state, setState] = useState({
    status: 'idle',
    data: null,
    error: null
  });
  
  const fetchData = useCallback(async () => {
    setState({ status: 'loading', data: null, error: null });
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      const data = await response.json();
      setState({ status: 'success', data, error: null });
    } catch (error) {
      setState({ status: 'error', data: null, error: error.message });
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  const { status, data, error } = state;
  
  // Loading
  if (status === 'loading') {
    return (
      <div style={{ padding: 40, textAlign: 'center' }}>
        <LoadingSpinner />
        <p>กำลังโหลดข้อมูล...</p>
      </div>
    );
  }
  
  // Error
  if (status === 'error') {
    return (
      <ErrorDisplay
        message={`โหลดข้อมูลไม่สำเร็จ: ${error}`}
        onRetry={fetchData}
      />
    );
  }
  
  // Empty
  if (status === 'success' && (!data || (Array.isArray(data) && data.length === 0))) {
    return (
      <EmptyState 
        title={emptyMessage}
        action={<button onClick={fetchData}>🔄 Refresh</button>}
      />
    );
  }
  
  // Success
  if (status === 'success') {
    return (
      <div>
        {Array.isArray(data) 
          ? data.map((item, i) => <div key={item.id || i}>{renderItem(item)}</div>)
          : renderItem(data)
        }
      </div>
    );
  }
  
  // Idle
  return null;
}

// การใช้งาน
<AsyncContent
  url="/api/users"
  emptyMessage="ยังไม่มีผู้ใช้"
  renderItem={(user) => (
    <UserCard key={user.id} user={user} />
  )}
/>
```

---

## Step 161: Feature Flags Pattern {#step-161}

Feature Flags ใช้สำหรับเปิด/ปิด features โดยไม่ต้อง deploy ใหม่

```jsx
// Feature Flags configuration
const FEATURE_FLAGS = {
  newCheckout: true,
  darkMode: false,
  betaFeature: process.env.NODE_ENV === 'development',
  premiumFeature: false,
};

// Context สำหรับ Feature Flags
const FeatureFlagContext = React.createContext(FEATURE_FLAGS);

function FeatureFlagProvider({ children, overrides = {} }) {
  const flags = { ...FEATURE_FLAGS, ...overrides };
  
  return (
    <FeatureFlagContext.Provider value={flags}>
      {children}
    </FeatureFlagContext.Provider>
  );
}

// Hook สำหรับใช้ Feature Flag
function useFeatureFlag(flagName) {
  const flags = React.useContext(FeatureFlagContext);
  return flags[flagName] ?? false;
}

// Component ที่ใช้ Feature Flag
function Feature({ name, children, fallback = null }) {
  const isEnabled = useFeatureFlag(name);
  return isEnabled ? children : fallback;
}

// การใช้งาน
function CheckoutPage() {
  const hasNewCheckout = useFeatureFlag('newCheckout');
  
  return (
    <div>
      {/* วิธีที่ 1: ด้วย hook */}
      {hasNewCheckout ? <NewCheckoutFlow /> : <OldCheckoutFlow />}
      
      {/* วิธีที่ 2: ด้วย Feature component */}
      <Feature name="newCheckout" fallback={<OldCheckoutFlow />}>
        <NewCheckoutFlow />
      </Feature>
      
      {/* วิธีที่ 3: Show only when enabled */}
      <Feature name="betaFeature">
        <BetaBadge />
      </Feature>
    </div>
  );
}

// App setup
function App() {
  return (
    <FeatureFlagProvider overrides={{ darkMode: true }}>
      <Router>
        <CheckoutPage />
      </Router>
    </FeatureFlagProvider>
  );
}
```

---

## Step 162: Role-based Rendering {#step-162}

แสดง UI ตาม role ของผู้ใช้

```jsx
const ROLES = {
  ADMIN: 'admin',
  EDITOR: 'editor',
  VIEWER: 'viewer',
  GUEST: 'guest',
};

// Permissions map
const PERMISSIONS = {
  [ROLES.ADMIN]: ['view', 'create', 'edit', 'delete', 'manage_users'],
  [ROLES.EDITOR]: ['view', 'create', 'edit'],
  [ROLES.VIEWER]: ['view'],
  [ROLES.GUEST]: [],
};

function usePermission(permission) {
  const user = useContext(UserContext);
  const userPermissions = PERMISSIONS[user?.role] || [];
  return userPermissions.includes(permission);
}

// Permission Gate Component
function PermissionGate({ permission, children, fallback = null }) {
  const hasPermission = usePermission(permission);
  return hasPermission ? children : fallback;
}

// การใช้งาน
function ArticleActions({ article }) {
  const canEdit = usePermission('edit');
  const canDelete = usePermission('delete');
  const canManageUsers = usePermission('manage_users');
  
  return (
    <div>
      {/* ทุกคน view ได้ */}
      <button>อ่านบทความ</button>
      
      {/* เฉพาะผู้มีสิทธิ์ edit */}
      <PermissionGate permission="edit">
        <button>แก้ไข</button>
      </PermissionGate>
      
      {/* เฉพาะผู้มีสิทธิ์ delete */}
      <PermissionGate 
        permission="delete"
        fallback={<span style={{color:'#999'}}>ไม่มีสิทธิ์ลบ</span>}
      >
        <button style={{ color: 'red' }}>ลบ</button>
      </PermissionGate>
      
      {/* Conditional ตาม permission หลายอัน */}
      {canEdit && canDelete && (
        <button>Bulk Actions</button>
      )}
    </div>
  );
}
```

---

## Step 163: Skeleton Loading {#step-163}

Skeleton screen แสดงโครงร่างขณะโหลดข้อมูล ดีกว่า spinner

```jsx
// Skeleton Component
function Skeleton({ width = '100%', height = 20, borderRadius = 4, className }) {
  return (
    <div
      className={className}
      style={{
        width,
        height,
        borderRadius,
        backgroundColor: '#e0e0e0',
        backgroundImage: 'linear-gradient(90deg, #e0e0e0 25%, #f5f5f5 50%, #e0e0e0 75%)',
        backgroundSize: '200% 100%',
        animation: 'shimmer 1.5s infinite',
        display: 'inline-block'
      }}
    />
  );
}

// Skeleton for UserCard
function UserCardSkeleton() {
  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: 12, padding: 16 }}>
      {/* Avatar */}
      <Skeleton width={48} height={48} borderRadius="50%" />
      
      <div style={{ flex: 1 }}>
        {/* Name */}
        <Skeleton width="60%" height={16} style={{ marginBottom: 8 }} />
        {/* Email */}
        <Skeleton width="80%" height={12} />
      </div>
    </div>
  );
}

// Skeleton for Article
function ArticleSkeleton() {
  return (
    <div style={{ padding: 16 }}>
      <Skeleton width="100%" height={200} style={{ marginBottom: 16 }} />
      <Skeleton width="70%" height={24} style={{ marginBottom: 8 }} />
      <Skeleton width="40%" height={16} style={{ marginBottom: 16 }} />
      <Skeleton width="100%" height={16} style={{ marginBottom: 8 }} />
      <Skeleton width="100%" height={16} style={{ marginBottom: 8 }} />
      <Skeleton width="80%" height={16} />
    </div>
  );
}

// SkeletonList
function SkeletonList({ count = 3, Component = UserCardSkeleton }) {
  return (
    <>
      {Array.from({ length: count }, (_, i) => (
        <Component key={i} />
      ))}
    </>
  );
}

// การใช้งาน
function UserList() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  
  // ...fetch logic...
  
  if (isLoading) {
    return <SkeletonList count={5} Component={UserCardSkeleton} />;
  }
  
  return (
    <>
      {users.map(user => <UserCard key={user.id} user={user} />)}
    </>
  );
}
```

---

## Step 164: Guard Clauses Pattern {#step-164}

Guard Clauses ใช้ early return เพื่อลด nesting และทำให้ code อ่านง่ายขึ้น

```jsx
// ❌ Deep nesting (pyramid of doom)
function DeepNested({ user, data, config }) {
  if (user) {
    if (user.isActive) {
      if (data) {
        if (data.length > 0) {
          if (config?.showDetails) {
            return (
              <DetailView user={user} data={data} config={config} />
            );
          }
        }
      }
    }
  }
  return <FallbackView />;
}

// ✅ Guard Clauses - ง่ายต่อการอ่าน
function WithGuards({ user, data, config }) {
  // Guard 1: ต้องมี user
  if (!user) return <LoginPrompt />;
  
  // Guard 2: user ต้องใช้งานอยู่
  if (!user.isActive) return <InactiveUserMessage />;
  
  // Guard 3: ต้องมีข้อมูล
  if (!data) return <LoadingSpinner />;
  
  // Guard 4: ข้อมูลต้องไม่ว่าง
  if (data.length === 0) return <EmptyState />;
  
  // Happy path - ถึงตรงนี้ทุกอย่างพร้อมแล้ว
  return (
    <div>
      {config?.showDetails && <DetailHeader />}
      <DataList data={data} user={user} />
    </div>
  );
}
```

### Guard Pattern สำหรับ Forms

```jsx
function SubmitButton({ formData, isSubmitting, isValid }) {
  // Guards
  if (isSubmitting) {
    return <LoadingButton>กำลังส่ง...</LoadingButton>;
  }
  
  if (!isValid) {
    return <button disabled>กรอกข้อมูลให้ครบก่อน</button>;
  }
  
  return <button type="submit">ส่งข้อมูล</button>;
}
```

---

## Step 165: Best Practices และ Anti-patterns {#step-165}

### Best Practices

**1. ใช้ Early Return สำหรับ Guard Conditions**
```jsx
// ✅
function Component({ data }) {
  if (!data) return <Loading />;
  if (data.error) return <Error />;
  
  return <DataDisplay data={data} />;
}
```

**2. ตั้งชื่อ boolean ให้ชัดเจน**
```jsx
// ✅
const isLoading = status === 'loading';
const hasError = !!error;
const isEmpty = items.length === 0;
const isAuthenticated = !!user;
```

**3. สร้าง Component แยกสำหรับ conditional sections**
```jsx
// ✅ แยก component ทำให้ readable ขึ้น
function UserDashboard({ user }) {
  return (
    <div>
      <Header user={user} />
      {user.isPremium ? <PremiumDashboard /> : <FreeDashboard />}
      <UserStats userId={user.id} />
    </div>
  );
}
```

**4. ใช้ Object Map แทน nested ternary**
```jsx
// ✅
const VARIANTS = {
  primary: 'btn-primary',
  secondary: 'btn-secondary',
  danger: 'btn-danger',
};

function Button({ variant = 'primary', children }) {
  return (
    <button className={VARIANTS[variant] || VARIANTS.primary}>
      {children}
    </button>
  );
}
```

### Anti-patterns

**1. ❌ Nested Ternary มากเกินไป**
```jsx
// ❌ อ่านยากมาก
const element = a ? b : c ? d : e ? f : g;

// ✅ ใช้ function แทน
const getElement = () => {
  if (a) return b;
  if (c) return d;
  if (e) return f;
  return g;
};
```

**2. ❌ ใช้ number โดยตรงกับ &&**
```jsx
// ❌ อาจ render "0"
{count && <Counter />}

// ✅
{count > 0 && <Counter />}
```

**3. ❌ Logic ซับซ้อนใน JSX**
```jsx
// ❌ ยากต่อการอ่านและ test
{user && user.role === 'admin' && permissions.includes('edit') && !user.isSuspended && (
  <EditButton />
)}

// ✅ แยก logic ออกมา
const canEdit = user?.role === 'admin' 
  && permissions.includes('edit') 
  && !user?.isSuspended;

{canEdit && <EditButton />}
```

**4. ❌ Duplicate Conditional Logic**
```jsx
// ❌ เขียนซ้ำ
{isLoading && <div>Loading...</div>}
{!isLoading && data && <DataView />}
{!isLoading && !data && <EmptyState />}

// ✅ ชัดเจนกว่า
if (isLoading) return <Loading />;
if (!data) return <EmptyState />;
return <DataView data={data} />;
```

---

## Quiz และแบบฝึกหัด {#quiz}

### คำถาม

1. ความแตกต่างระหว่าง `&&`, `||`, `??` คืออะไร?
2. ทำไมถึงควรระวังการใช้ `0 && <Component />`?
3. Guard Clauses คืออะไร และใช้เมื่อไหร่?
4. Skeleton Loading ดีกว่า Spinner อย่างไร?
5. เมื่อไหรควรใช้ Object Map แทน switch?

### แบบฝึกหัด

**Exercise 1:** สร้าง `DataCard` component ที่:
- แสดง Skeleton ขณะ loading
- แสดง Error พร้อม retry button
- แสดง Empty State ถ้าไม่มีข้อมูล
- แสดงข้อมูลจริงเมื่อ success

**Exercise 2:** สร้าง `PermissionButton` ที่:
- รับ `requiredPermission` prop
- ซ่อนปุ่มถ้าไม่มีสิทธิ์
- แสดง tooltip บอกเหตุผลถ้า hover

**Exercise 3:** Refactor โค้ดต่อไปนี้โดยใช้ Guard Clauses:
```jsx
function ComplexComponent({ user, subscription, features }) {
  if (user) {
    if (!user.isBlocked) {
      if (subscription) {
        if (subscription.isActive) {
          if (features && features.length > 0) {
            return <FeatureList features={features} />;
          } else {
            return <NoFeatures />;
          }
        } else {
          return <ExpiredSubscription />;
        }
      } else {
        return <NoSubscription />;
      }
    } else {
      return <BlockedUser />;
    }
  }
  return <NotLoggedIn />;
}
```

---

## สรุป

ใน Part นี้เราได้เรียนรู้:
- Conditional Rendering ทุกวิธี: if/else, ternary, &&, ||, ??
- Short-circuit Evaluation และ gotchas
- Switch-based Rendering
- Object Map Pattern
- Loading, Error, Empty State Patterns
- Skeleton Loading
- Feature Flags Pattern
- Role-based Rendering
- Guard Clauses Pattern

---

## อ่านต่อ

➡️ **[Part 09: Lists และ Keys](./part-09-lists-and-keys.md)**

เราจะเรียนรู้การ render lists, key prop ที่ถูกต้อง, filter/sort, และ CRUD operations

---

*React Course - Part 08 | สร้างโดย React Course Thailand*
