# Part 16: useContext Hook ขั้นสูง

## Step 346-375 | ระดับ: กลาง-สูง

---

## สารบัญ

- [Step 346: useContext ทบทวน](#step-346-usecontext-ทบทวน)
- [Step 347: Context + useReducer Pattern](#step-347-context--usereducer-pattern)
- [Step 348: Custom Provider Hooks](#step-348-custom-provider-hooks)
- [Step 349: Context Optimization ขั้นสูง](#step-349-context-optimization-ขั้นสูง)
- [Step 350: เมื่อไหรควรใช้ Context](#step-350-เมื่อไหรควรใช้-context)
- [Step 351: ตัวอย่าง Shopping Cart Context สมบูรณ์](#step-351-ตัวอย่าง-shopping-cart-context-สมบูรณ์)
- [Step 352: Context กับ Server-Side Rendering](#step-352-context-กับ-server-side-rendering)
- [Step 353: Testing Context](#step-353-testing-context)
- [Step 354: Context Pattern ขั้นสูง](#step-354-context-pattern-ขั้นสูง)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 346: useContext ทบทวน

```jsx
import { createContext, useContext, useState, useMemo, useReducer, useCallback } from 'react';

// Pattern พื้นฐาน
const CounterContext = createContext(null);

function CounterProvider({ children, initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);
  
  const value = useMemo(() => ({
    count,
    increment: () => setCount(c => c + 1),
    decrement: () => setCount(c => c - 1),
    reset: () => setCount(initialCount)
  }), [count, initialCount]);
  
  return (
    <CounterContext.Provider value={value}>
      {children}
    </CounterContext.Provider>
  );
}

function useCounter() {
  const ctx = useContext(CounterContext);
  if (!ctx) throw new Error('useCounter must be used within CounterProvider');
  return ctx;
}

// Components
function CountDisplay() {
  const { count } = useCounter();
  return <span style={{ fontSize: '2rem', fontWeight: 'bold' }}>{count}</span>;
}

function CountControls() {
  const { increment, decrement, reset } = useCounter();
  return (
    <div style={{ display: 'flex', gap: '8px' }}>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
      <button onClick={increment}>+</button>
    </div>
  );
}

function CounterApp() {
  return (
    <CounterProvider initialCount={10}>
      <div style={{ textAlign: 'center', padding: '20px' }}>
        <CountDisplay />
        <CountControls />
      </div>
    </CounterProvider>
  );
}
```

---

## Step 347: Context + useReducer Pattern

useReducer + Context เหมาะสำหรับ state ที่ซับซ้อน

```jsx
// 1. กำหนด types
const ACTIONS = {
  SET_USER: 'SET_USER',
  UPDATE_PROFILE: 'UPDATE_PROFILE',
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR',
  CLEAR_ERROR: 'CLEAR_ERROR',
  LOGOUT: 'LOGOUT'
};

// 2. กำหนด initial state
const initialState = {
  user: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
  profile: null
};

// 3. สร้าง reducer
function authReducer(state, action) {
  switch (action.type) {
    case ACTIONS.SET_USER:
      return {
        ...state,
        user: action.payload,
        isAuthenticated: !!action.payload,
        isLoading: false,
        error: null
      };
      
    case ACTIONS.UPDATE_PROFILE:
      return {
        ...state,
        user: state.user ? { ...state.user, ...action.payload } : state.user
      };
      
    case ACTIONS.SET_LOADING:
      return { ...state, isLoading: action.payload };
      
    case ACTIONS.SET_ERROR:
      return { ...state, error: action.payload, isLoading: false };
      
    case ACTIONS.CLEAR_ERROR:
      return { ...state, error: null };
      
    case ACTIONS.LOGOUT:
      return { ...initialState };
      
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// 4. สร้าง Contexts แยก (state vs dispatch)
const AuthStateContext = createContext(null);
const AuthDispatchContext = createContext(null);

// 5. สร้าง Provider
function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, initialState);
  
  // Load user จาก localStorage เมื่อ mount
  useEffect(() => {
    const savedUser = localStorage.getItem('auth_user');
    if (savedUser) {
      dispatch({ type: ACTIONS.SET_USER, payload: JSON.parse(savedUser) });
    }
  }, []);
  
  return (
    <AuthStateContext.Provider value={state}>
      <AuthDispatchContext.Provider value={dispatch}>
        {children}
      </AuthDispatchContext.Provider>
    </AuthStateContext.Provider>
  );
}

// 6. Custom hooks
function useAuthState() {
  const ctx = useContext(AuthStateContext);
  if (!ctx) throw new Error('useAuthState must be used within AuthProvider');
  return ctx;
}

function useAuthDispatch() {
  const ctx = useContext(AuthDispatchContext);
  if (!ctx) throw new Error('useAuthDispatch must be used within AuthProvider');
  return ctx;
}

// 7. Action creators (ทำให้ component ไม่ต้องรู้ action structure)
function useAuthActions() {
  const dispatch = useAuthDispatch();
  
  const login = useCallback(async (email, password) => {
    dispatch({ type: ACTIONS.SET_LOADING, payload: true });
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      if (!response.ok) throw new Error('Login failed');
      
      const user = await response.json();
      localStorage.setItem('auth_user', JSON.stringify(user));
      dispatch({ type: ACTIONS.SET_USER, payload: user });
      return true;
    } catch (error) {
      dispatch({ type: ACTIONS.SET_ERROR, payload: error.message });
      return false;
    }
  }, [dispatch]);
  
  const logout = useCallback(() => {
    localStorage.removeItem('auth_user');
    dispatch({ type: ACTIONS.LOGOUT });
  }, [dispatch]);
  
  const updateProfile = useCallback((updates) => {
    dispatch({ type: ACTIONS.UPDATE_PROFILE, payload: updates });
  }, [dispatch]);
  
  const clearError = useCallback(() => {
    dispatch({ type: ACTIONS.CLEAR_ERROR });
  }, [dispatch]);
  
  return { login, logout, updateProfile, clearError };
}

// การใช้งาน
function LoginButton() {
  const { isAuthenticated, isLoading } = useAuthState();
  const { login, logout } = useAuthActions();
  
  if (isAuthenticated) {
    return <button onClick={logout}>Logout</button>;
  }
  
  return (
    <button 
      onClick={() => login('user@test.com', 'password')}
      disabled={isLoading}
    >
      {isLoading ? 'Loading...' : 'Login'}
    </button>
  );
}

function UserInfo() {
  // สนใจแค่ state ไม่ต้อง re-render เมื่อ dispatch
  const { user, isAuthenticated } = useAuthState();
  
  if (!isAuthenticated) return <p>Not logged in</p>;
  
  return (
    <div>
      <p>Welcome, {user.name}!</p>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

---

## Step 348: Custom Provider Hooks

```jsx
// Pattern: สร้าง complete Provider module

// notifications-context.js
const NotificationContext = createContext(null);

const notificationReducer = (state, action) => {
  switch (action.type) {
    case 'ADD':
      return [...state, { 
        ...action.payload, 
        id: Date.now() + Math.random(),
        createdAt: new Date()
      }];
    case 'REMOVE':
      return state.filter(n => n.id !== action.payload);
    case 'CLEAR':
      return [];
    default:
      return state;
  }
};

function NotificationProvider({ children }) {
  const [notifications, dispatch] = useReducer(notificationReducer, []);
  
  const addNotification = useCallback(({ message, type = 'info', duration = 3000 }) => {
    const id = Date.now() + Math.random();
    dispatch({ type: 'ADD', payload: { message, type, id } });
    
    if (duration > 0) {
      setTimeout(() => {
        dispatch({ type: 'REMOVE', payload: id });
      }, duration);
    }
    
    return id;
  }, []);
  
  const removeNotification = useCallback((id) => {
    dispatch({ type: 'REMOVE', payload: id });
  }, []);
  
  const clearAll = useCallback(() => {
    dispatch({ type: 'CLEAR' });
  }, []);
  
  // Convenience methods
  const success = useCallback((message, opts) => 
    addNotification({ message, type: 'success', ...opts }), [addNotification]);
    
  const error = useCallback((message, opts) => 
    addNotification({ message, type: 'error', duration: 5000, ...opts }), [addNotification]);
    
  const warning = useCallback((message, opts) => 
    addNotification({ message, type: 'warning', ...opts }), [addNotification]);
    
  const info = useCallback((message, opts) => 
    addNotification({ message, type: 'info', ...opts }), [addNotification]);
  
  const value = useMemo(() => ({
    notifications,
    addNotification,
    removeNotification,
    clearAll,
    success,
    error,
    warning,
    info
  }), [notifications, addNotification, removeNotification, clearAll, success, error, warning, info]);
  
  return (
    <NotificationContext.Provider value={value}>
      {children}
      <NotificationList />
    </NotificationContext.Provider>
  );
}

function useNotification() {
  const ctx = useContext(NotificationContext);
  if (!ctx) throw new Error('useNotification must be used within NotificationProvider');
  return ctx;
}

// Notification Display Component
function NotificationList() {
  const { notifications, removeNotification } = useContext(NotificationContext);
  
  const typeStyles = {
    success: { bg: '#d4edda', color: '#155724', border: '#c3e6cb' },
    error: { bg: '#f8d7da', color: '#721c24', border: '#f5c6cb' },
    warning: { bg: '#fff3cd', color: '#856404', border: '#ffeeba' },
    info: { bg: '#cce5ff', color: '#004085', border: '#b8daff' }
  };
  
  return (
    <div style={{
      position: 'fixed',
      top: '16px',
      right: '16px',
      zIndex: 9999,
      display: 'flex',
      flexDirection: 'column',
      gap: '8px',
      maxWidth: '360px',
      width: '100%'
    }}>
      {notifications.map(notification => {
        const styles = typeStyles[notification.type] || typeStyles.info;
        return (
          <div
            key={notification.id}
            style={{
              background: styles.bg,
              color: styles.color,
              border: `1px solid ${styles.border}`,
              borderRadius: '6px',
              padding: '12px 16px',
              display: 'flex',
              justifyContent: 'space-between',
              alignItems: 'flex-start',
              boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
              animation: 'slideInRight 0.3s ease'
            }}
          >
            <span style={{ flex: 1, marginRight: '8px' }}>
              {notification.message}
            </span>
            <button
              onClick={() => removeNotification(notification.id)}
              style={{
                background: 'none',
                border: 'none',
                cursor: 'pointer',
                fontSize: '16px',
                color: 'inherit',
                padding: '0',
                lineHeight: 1
              }}
            >
              ×
            </button>
          </div>
        );
      })}
    </div>
  );
}

// การใช้งาน
function ProductCard({ product }) {
  const { success, error } = useNotification();
  const { addItem } = useCart();
  
  const handleAddToCart = async () => {
    try {
      await addItem(product);
      success(`เพิ่ม "${product.name}" ลงตะกร้าแล้ว`);
    } catch (e) {
      error('ไม่สามารถเพิ่มสินค้าได้ กรุณาลองใหม่');
    }
  };
  
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={handleAddToCart}>เพิ่มลงตะกร้า</button>
    </div>
  );
}
```

---

## Step 349: Context Optimization ขั้นสูง

### Context Selector Pattern

```jsx
// ปัญหา: consumer re-render แม้ส่วนที่ตัวเองสนใจไม่เปลี่ยน
const UserContext = createContext(null);

// Component นี้ re-render ทุกครั้งที่ user.notifications เปลี่ยน
// แม้ว่าจะสนใจแค่ user.name
function UserName() {
  const { user } = useContext(UserContext); // ❌ re-render ทุกครั้ง
  return <span>{user.name}</span>;
}

// ✅ แก้ไขด้วย React.memo
const UserNameMemo = React.memo(function({ name }) {
  return <span>{name}</span>;
});

function UserNameWrapper() {
  const { user } = useContext(UserContext);
  return <UserNameMemo name={user.name} />;
}

// ✅ หรือแยก Context ตาม concern
const UserNameContext = createContext('');
const UserNotificationsContext = createContext([]);

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <UserNameContext.Provider value={user?.name || ''}>
      <UserNotificationsContext.Provider value={user?.notifications || []}>
        {children}
      </UserNotificationsContext.Provider>
    </UserNameContext.Provider>
  );
}

// Component นี้ re-render เฉพาะเมื่อ name เปลี่ยน
function UserNameOptimized() {
  const name = useContext(UserNameContext); // ✅ แค่ name
  return <span>{name}</span>;
}
```

### Context Batching

```jsx
function OptimizedCartProvider({ children }) {
  const [items, dispatch] = useReducer(cartReducer, []);
  
  // ✅ Compute derived values ด้วย useMemo
  const stats = useMemo(() => ({
    count: items.reduce((sum, item) => sum + item.quantity, 0),
    total: items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    isEmpty: items.length === 0
  }), [items]);
  
  // ✅ Memoize actions
  const actions = useMemo(() => ({
    addItem: (product) => dispatch({ type: 'ADD_ITEM', payload: product }),
    removeItem: (id) => dispatch({ type: 'REMOVE_ITEM', payload: id }),
    updateQuantity: (id, qty) => dispatch({ type: 'UPDATE_QTY', payload: { id, qty } }),
    clearCart: () => dispatch({ type: 'CLEAR' })
  }), []); // dispatch ไม่เปลี่ยน ดังนั้น actions ก็ไม่เปลี่ยน
  
  // แยก Context: items เปลี่ยนบ่อย, actions ไม่เปลี่ยน
  return (
    <CartItemsContext.Provider value={items}>
      <CartStatsContext.Provider value={stats}>
        <CartActionsContext.Provider value={actions}>
          {children}
        </CartActionsContext.Provider>
      </CartStatsContext.Provider>
    </CartItemsContext.Provider>
  );
}

const CartItemsContext = createContext([]);
const CartStatsContext = createContext({ count: 0, total: 0, isEmpty: true });
const CartActionsContext = createContext(null);

// Hooks ที่แยกตาม concern
function useCartItems() { return useContext(CartItemsContext); }
function useCartStats() { return useContext(CartStatsContext); }
function useCartActions() { return useContext(CartActionsContext); }
```

---

## Step 350: เมื่อไหรควรใช้ Context

### Decision Matrix

```
State ที่ใช้ใน:     1-2 components    หลาย components    ทั้งแอป
Solution:            useState/props    Context API         State Mgmt Library
                     (+ lift state)    (+ Context)         (Redux/Zustand)

ตัวอย่าง:
- Form input state  → useState
- Modal open/close  → useState
- Current user      → Context
- Theme             → Context  
- Cart              → Context หรือ Zustand
- Complex dashboard → Redux/Zustand
```

```jsx
// ตัวอย่างการตัดสินใจ

// 1. useState (ส่วนตัวของ component)
function Counter() {
  const [count, setCount] = useState(0); // ✅ local state
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// 2. Props (ส่งลง 1-2 levels)
function ProductCard({ name, price, onAddToCart }) { // ✅ props
  return (
    <div>
      <p>{name}: {price}</p>
      <button onClick={onAddToCart}>Add to Cart</button>
    </div>
  );
}

// 3. Context (global/shared state)
function useUserPreferences() {
  return useContext(PreferencesContext); // ✅ context
}

// 4. External library (complex state)
// ใช้ Redux, Zustand, Jotai ถ้า state complex มาก
```

---

## Step 351: ตัวอย่าง Shopping Cart Context สมบูรณ์

```jsx
// cart-context.jsx

const CartContext = createContext(null);

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingIndex = state.items.findIndex(
        item => item.id === action.payload.id
      );
      
      if (existingIndex >= 0) {
        const updatedItems = state.items.map((item, index) => 
          index === existingIndex
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
        return { ...state, items: updatedItems };
      }
      
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    }
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
      
    case 'UPDATE_QUANTITY': {
      if (action.payload.quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(item => item.id !== action.payload.id)
        };
      }
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
    }
    
    case 'CLEAR_CART':
      return { ...state, items: [] };
      
    case 'APPLY_COUPON':
      return {
        ...state,
        coupon: action.payload,
        discount: action.payload ? action.payload.discountPercent : 0
      };
      
    default:
      return state;
  }
};

const initialCartState = {
  items: [],
  coupon: null,
  discount: 0
};

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(
    cartReducer,
    initialCartState,
    // Initializer: โหลดจาก localStorage
    (initial) => {
      try {
        const saved = localStorage.getItem('cart');
        return saved ? JSON.parse(saved) : initial;
      } catch {
        return initial;
      }
    }
  );
  
  // Save ลง localStorage เมื่อ state เปลี่ยน
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(state));
  }, [state]);
  
  // Computed values
  const subtotal = useMemo(() => 
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [state.items]
  );
  
  const discountAmount = useMemo(() => 
    subtotal * (state.discount / 100),
    [subtotal, state.discount]
  );
  
  const total = useMemo(() => 
    subtotal - discountAmount,
    [subtotal, discountAmount]
  );
  
  const itemCount = useMemo(() => 
    state.items.reduce((sum, item) => sum + item.quantity, 0),
    [state.items]
  );
  
  // Actions
  const actions = useMemo(() => ({
    addItem: (product) => dispatch({ type: 'ADD_ITEM', payload: product }),
    removeItem: (id) => dispatch({ type: 'REMOVE_ITEM', payload: id }),
    updateQuantity: (id, quantity) => dispatch({ 
      type: 'UPDATE_QUANTITY', 
      payload: { id, quantity } 
    }),
    clearCart: () => dispatch({ type: 'CLEAR_CART' }),
    applyCoupon: (coupon) => dispatch({ type: 'APPLY_COUPON', payload: coupon })
  }), []);
  
  const value = useMemo(() => ({
    items: state.items,
    coupon: state.coupon,
    discount: state.discount,
    subtotal,
    discountAmount,
    total,
    itemCount,
    isEmpty: state.items.length === 0,
    ...actions
  }), [state, subtotal, discountAmount, total, itemCount, actions]);
  
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

export function useCart() {
  const ctx = useContext(CartContext);
  if (!ctx) throw new Error('useCart must be used within CartProvider');
  return ctx;
}

// Cart Badge
function CartBadge() {
  const { itemCount } = useCart();
  
  return (
    <div style={{ position: 'relative', display: 'inline-block' }}>
      🛒
      {itemCount > 0 && (
        <span style={{
          position: 'absolute',
          top: '-8px',
          right: '-8px',
          background: '#dc3545',
          color: 'white',
          borderRadius: '50%',
          width: '20px',
          height: '20px',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: '12px',
          fontWeight: 'bold'
        }}>
          {itemCount > 99 ? '99+' : itemCount}
        </span>
      )}
    </div>
  );
}

// Cart Sidebar
function CartSidebar({ isOpen, onClose }) {
  const { items, subtotal, total, discount, discountAmount, updateQuantity, removeItem, clearCart } = useCart();
  
  if (!isOpen) return null;
  
  return (
    <div style={{
      position: 'fixed', top: 0, right: 0, bottom: 0,
      width: '380px', background: 'white',
      boxShadow: '-4px 0 20px rgba(0,0,0,0.15)',
      zIndex: 1000, display: 'flex', flexDirection: 'column'
    }}>
      <div style={{ padding: '20px', borderBottom: '1px solid #eee', display: 'flex', justifyContent: 'space-between' }}>
        <h2 style={{ margin: 0 }}>ตะกร้าสินค้า ({items.length})</h2>
        <button onClick={onClose} style={{ background: 'none', border: 'none', fontSize: '20px', cursor: 'pointer' }}>×</button>
      </div>
      
      <div style={{ flex: 1, overflowY: 'auto', padding: '16px' }}>
        {items.length === 0 ? (
          <div style={{ textAlign: 'center', color: '#999', marginTop: '40px' }}>
            <p style={{ fontSize: '40px' }}>🛒</p>
            <p>ตะกร้าว่างเปล่า</p>
          </div>
        ) : (
          items.map(item => (
            <div key={item.id} style={{ display: 'flex', gap: '12px', marginBottom: '16px', padding: '12px', border: '1px solid #eee', borderRadius: '8px' }}>
              <img src={item.image} alt={item.name} style={{ width: '60px', height: '60px', objectFit: 'cover', borderRadius: '4px' }} />
              <div style={{ flex: 1 }}>
                <p style={{ margin: '0 0 4px', fontWeight: '500' }}>{item.name}</p>
                <p style={{ margin: '0 0 8px', color: '#007bff' }}>฿{item.price.toLocaleString()}</p>
                <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
                  <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>-</button>
                  <span>{item.quantity}</span>
                  <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>+</button>
                  <button 
                    onClick={() => removeItem(item.id)}
                    style={{ marginLeft: 'auto', color: 'red', background: 'none', border: 'none', cursor: 'pointer' }}
                  >
                    🗑️
                  </button>
                </div>
              </div>
            </div>
          ))
        )}
      </div>
      
      {items.length > 0 && (
        <div style={{ padding: '16px 20px', borderTop: '1px solid #eee' }}>
          <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '8px' }}>
            <span>ราคาสินค้า:</span>
            <span>฿{subtotal.toLocaleString()}</span>
          </div>
          {discount > 0 && (
            <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '8px', color: '#28a745' }}>
              <span>ส่วนลด ({discount}%):</span>
              <span>-฿{discountAmount.toLocaleString()}</span>
            </div>
          )}
          <div style={{ display: 'flex', justifyContent: 'space-between', fontWeight: 'bold', fontSize: '18px', marginBottom: '16px' }}>
            <span>รวมทั้งหมด:</span>
            <span>฿{total.toLocaleString()}</span>
          </div>
          <button style={{ width: '100%', padding: '12px', background: '#007bff', color: 'white', border: 'none', borderRadius: '6px', fontSize: '16px', cursor: 'pointer' }}>
            ชำระเงิน
          </button>
          <button 
            onClick={clearCart}
            style={{ width: '100%', marginTop: '8px', padding: '8px', background: 'none', border: '1px solid #ccc', borderRadius: '6px', cursor: 'pointer', color: '#666' }}
          >
            ล้างตะกร้า
          </button>
        </div>
      )}
    </div>
  );
}
```

---

## Step 352: Context กับ Server-Side Rendering

```jsx
// ใน Next.js / SSR Context
function ServerSafeProvider({ children, initialData }) {
  // ใช้ lazy initialization เพื่อหลีกเลี่ยง SSR mismatch
  const [state, setState] = useState(() => {
    // ใน SSR: ใช้ initialData จาก server
    // ใน Browser: ลองดู localStorage
    if (typeof window !== 'undefined') {
      try {
        const saved = localStorage.getItem('cart');
        return saved ? JSON.parse(saved) : initialData;
      } catch {
        return initialData;
      }
    }
    return initialData;
  });
  
  return (
    <MyContext.Provider value={state}>
      {children}
    </MyContext.Provider>
  );
}
```

---

## Step 353: Testing Context

```jsx
// utils/test-utils.jsx
import { render } from '@testing-library/react';

function AllProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

function customRender(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}

export * from '@testing-library/react';
export { customRender as render };

// __tests__/CartBadge.test.jsx
import { render, screen } from '../utils/test-utils';
import { CartBadge } from '../components/CartBadge';

describe('CartBadge', () => {
  it('shows item count when cart has items', () => {
    // Mock cart context
    const mockCartValue = { itemCount: 3 };
    
    render(
      <CartContext.Provider value={mockCartValue}>
        <CartBadge />
      </CartContext.Provider>
    );
    
    expect(screen.getByText('3')).toBeInTheDocument();
  });
  
  it('does not show badge when cart is empty', () => {
    const mockCartValue = { itemCount: 0 };
    
    render(
      <CartContext.Provider value={mockCartValue}>
        <CartBadge />
      </CartContext.Provider>
    );
    
    expect(screen.queryByText('0')).not.toBeInTheDocument();
  });
});
```

---

## Step 354: Context Pattern ขั้นสูง

### Context Factory Pattern

```jsx
// ✅ Factory function สร้าง context + hooks อัตโนมัติ
function createRequiredContext(displayName) {
  const Context = createContext(null);
  Context.displayName = displayName;
  
  function useRequiredContext() {
    const ctx = useContext(Context);
    if (ctx === null) {
      throw new Error(`${displayName} context not found. Wrap component in ${displayName}Provider.`);
    }
    return ctx;
  }
  
  return [Context, useRequiredContext];
}

// สร้าง contexts
const [ThemeContext, useTheme] = createRequiredContext('Theme');
const [AuthContext, useAuth] = createRequiredContext('Auth');
const [CartContext, useCart] = createRequiredContext('Cart');
```

### Compound Context (Context + Compound Component)

```jsx
// Form Context ที่ทำงานกับ Compound Components
const FormContext = createContext(null);

function Form({ children, onSubmit, defaultValues = {} }) {
  const [values, setValues] = useState(defaultValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const setValue = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  }, [errors]);
  
  const setTouched_ = useCallback((name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
  }, []);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(values);
  };
  
  return (
    <FormContext.Provider value={{ values, errors, touched, setValue, setTouched: setTouched_ }}>
      <form onSubmit={handleSubmit}>{children}</form>
    </FormContext.Provider>
  );
}

function useFormField(name) {
  const { values, errors, touched, setValue, setTouched } = useContext(FormContext);
  
  return {
    value: values[name] || '',
    error: touched[name] ? errors[name] : undefined,
    onChange: (e) => setValue(name, e.target.value),
    onBlur: () => setTouched(name)
  };
}

function FormField({ name, label, type = 'text', ...props }) {
  const { value, error, onChange, onBlur } = useFormField(name);
  
  return (
    <div style={{ marginBottom: '16px' }}>
      <label>{label}</label>
      <input
        type={type}
        value={value}
        onChange={onChange}
        onBlur={onBlur}
        style={{ 
          display: 'block', 
          width: '100%', 
          padding: '8px',
          border: `1px solid ${error ? 'red' : '#ccc'}`,
          borderRadius: '4px'
        }}
        {...props}
      />
      {error && <p style={{ color: 'red', fontSize: '12px', margin: '4px 0 0' }}>{error}</p>}
    </div>
  );
}

Form.Field = FormField;

// การใช้งาน
function ContactForm() {
  const handleSubmit = (values) => {
    console.log('Form submitted:', values);
  };
  
  return (
    <Form onSubmit={handleSubmit} defaultValues={{ name: '', email: '' }}>
      <Form.Field name="name" label="ชื่อ" />
      <Form.Field name="email" label="Email" type="email" />
      <Form.Field name="message" label="ข้อความ" />
      <button type="submit">ส่ง</button>
    </Form>
  );
}
```

---

## สรุป useContext ขั้นสูง

| Pattern | ใช้เมื่อ |
|---|---|
| Context + useReducer | Complex state transitions |
| Split State/Dispatch | Optimize re-renders |
| Custom Provider Hooks | Reusable provider logic |
| Context Factory | สร้าง context หลายตัวที่มี pattern เหมือนกัน |
| Compound Context | Context + Compound Component |

---

## Quiz และแบบฝึกหัด

### Quiz

1. ทำไม Context + useReducer ดีกว่า Context + useState ในกรณีที่ state ซับซ้อน?
2. การแยก State Context กับ Dispatch Context ช่วยอะไร?
3. `useMemo` ใน Provider ช่วยอย่างไร?

### แบบฝึกหัด

**Exercise 1**: สร้าง Preferences Context ด้วย useReducer

```jsx
// TODO: สร้าง PreferencesProvider ที่มี:
// - language: 'th' | 'en'
// - currency: 'THB' | 'USD' | 'EUR'
// - notifications: boolean
// - emailUpdates: boolean
// - actions: updatePreferences, resetPreferences
// - Persist ลง localStorage
function PreferencesProvider({ children }) {
  // implement here...
}
```

**Exercise 2**: สร้าง Multi-step Form Context

```jsx
// TODO: สร้าง WizardContext ที่มี:
// - currentStep: number
// - totalSteps: number
// - data: object (accumulated form data)
// - nextStep(stepData): void
// - prevStep(): void
// - goToStep(step): void
// - isFirstStep: boolean
// - isLastStep: boolean
function WizardProvider({ steps, children }) {
  // implement here...
}
```

---

➡️ ไปต่อ: [Part 17: useReducer Hook](./part-17-useReducer-hook.md)
