# Part 15: Context API

## Step 316-345 | ระดับ: กลาง

---

## สารบัญ

- [Step 316: Context API คืออะไร?](#step-316-context-api-คืออะไร)
- [Step 317: ปัญหา Props Drilling](#step-317-ปัญหา-props-drilling)
- [Step 318: createContext และ Provider](#step-318-createcontext-และ-provider)
- [Step 319: Consumer Component](#step-319-consumer-component)
- [Step 320: useContext Hook](#step-320-usecontext-hook)
- [Step 321: Default Context Value](#step-321-default-context-value)
- [Step 322: Multiple Contexts](#step-322-multiple-contexts)
- [Step 323: Context Performance](#step-323-context-performance)
- [Step 324: ตัวอย่าง Theme Context](#step-324-ตัวอย่าง-theme-context)
- [Step 325: ตัวอย่าง Auth Context](#step-325-ตัวอย่าง-auth-context)
- [Step 326: ตัวอย่าง Language/i18n Context](#step-326-ตัวอย่าง-languagei18n-context)
- [Step 327: Context vs Props Drilling](#step-327-context-vs-props-drilling)
- [Step 328: Context กับ TypeScript](#step-328-context-กับ-typescript)
- [Step 329: Anti-patterns ใน Context](#step-329-anti-patterns-ใน-context)
- [Step 330: Best Practices](#step-330-best-practices)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 316: Context API คืออะไร?

**Context API** ช่วยให้ส่งข้อมูลผ่าน component tree โดยไม่ต้องส่ง props ทุก level

```
Without Context (Props Drilling):
App → Header → UserMenu → Avatar → Tooltip
 ↓      ↓         ↓         ↓        ↓
user   user      user      user     user  (ต้องส่งตลอดทาง)

With Context:
App (Context Provider - user)
 ├── Header
 │    └── UserMenu
 │         └── Avatar ← useContext → user (ดึงโดยตรง!)
 └── Sidebar
      └── UserProfile ← useContext → user (ดึงโดยตรง!)
```

### เมื่อไหร่ควรใช้ Context?

- **Theme** (dark/light mode)
- **Authentication** (current user)
- **Language** (localization)
- **Global configuration**
- **Shopping cart** (global state)

### เมื่อไหร่ไม่ควรใช้ Context?

- Props ส่งผ่านแค่ 1-2 level ก็พอ
- State ที่เปลี่ยนบ่อยมาก (ทำให้ re-render เยอะ)
- ใช้ state management library แทน (Redux, Zustand)

---

## Step 317: ปัญหา Props Drilling

```jsx
// ❌ Props Drilling - ยุ่งยากมาก
function App() {
  const [user, setUser] = useState({ name: 'Alice', role: 'admin' });
  const [theme, setTheme] = useState('light');
  
  return (
    <div>
      {/* ต้องส่ง user และ theme ทุก level */}
      <Header user={user} theme={theme} setTheme={setTheme} />
      <Main user={user} theme={theme} />
      <Footer user={user} theme={theme} />
    </div>
  );
}

function Header({ user, theme, setTheme }) {
  return (
    <header>
      {/* ต้องส่งต่อ แม้จะไม่ใช้โดยตรง */}
      <Nav user={user} theme={theme} setTheme={setTheme} />
    </header>
  );
}

function Nav({ user, theme, setTheme }) {
  return (
    <nav>
      {/* ต้องส่งต่ออีก */}
      <UserMenu user={user} theme={theme} setTheme={setTheme} />
    </nav>
  );
}

function UserMenu({ user, theme, setTheme }) {
  return (
    <div>
      {/* ต้องส่งต่ออีก */}
      <Avatar user={user} theme={theme} />
      <ThemeToggle theme={theme} setTheme={setTheme} />
    </div>
  );
}
```

---

## Step 318: createContext และ Provider

```jsx
import { createContext, useState } from 'react';

// 1. สร้าง Context
const ThemeContext = createContext('light'); // default value

// 2. สร้าง Provider
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = {
    theme,
    setTheme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light')
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Wrap app ด้วย Provider
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
      <Footer />
    </ThemeProvider>
  );
}
```

---

## Step 319: Consumer Component

Consumer เป็นวิธีเก่าในการ consume Context (ก่อน useContext hook)

```jsx
// Consumer (วิธีเก่า - ไม่แนะนำสำหรับ function components)
function ThemeButton() {
  return (
    <ThemeContext.Consumer>
      {({ theme, toggleTheme }) => (
        <button
          onClick={toggleTheme}
          style={{
            background: theme === 'dark' ? '#333' : '#fff',
            color: theme === 'dark' ? '#fff' : '#333'
          }}
        >
          Current: {theme}
        </button>
      )}
    </ThemeContext.Consumer>
  );
}
```

---

## Step 320: useContext Hook

```jsx
import { useContext } from 'react';

// ✅ useContext - วิธีแนะนำสำหรับ function components
function ThemeButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333',
        padding: '8px 16px',
        border: '1px solid currentColor',
        borderRadius: '4px',
        cursor: 'pointer'
      }}
    >
      {theme === 'dark' ? '☀️ Light Mode' : '🌙 Dark Mode'}
    </button>
  );
}

// Component ที่อยู่ลึกมากก็ useContext ได้เลย
function DeepNestedComponent() {
  const { theme } = useContext(ThemeContext);
  
  return (
    <div style={{ 
      background: theme === 'dark' ? '#1a1a1a' : '#fff',
      color: theme === 'dark' ? '#fff' : '#000',
      padding: '16px'
    }}>
      I'm deeply nested but have access to theme!
    </div>
  );
}
```

### Custom Hook สำหรับ Context

```jsx
// ✅ Best practice: สร้าง custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// การใช้งาน
function MyComponent() {
  const { theme, toggleTheme } = useTheme(); // ✅ ใช้ custom hook
  // ...
}
```

---

## Step 321: Default Context Value

```jsx
// Default value ใช้เมื่อไม่มี Provider ครอบ
const UserContext = createContext({
  user: null,
  isAuthenticated: false,
  login: () => {},
  logout: () => {}
});

// ถ้าไม่มี Provider จะใช้ default value
function ComponentWithoutProvider() {
  const { user, isAuthenticated } = useContext(UserContext);
  
  // user = null, isAuthenticated = false (default values)
  return <div>{isAuthenticated ? user.name : 'Guest'}</div>;
}

// ตัวอย่างที่ดีกว่า: throw error ถ้าไม่มี Provider
const StrictUserContext = createContext(null);

function useUser() {
  const context = useContext(StrictUserContext);
  if (context === null) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}
```

---

## Step 322: Multiple Contexts

```jsx
// หลาย Contexts ทำงานร่วมกัน
const ThemeContext = createContext(null);
const AuthContext = createContext(null);
const CartContext = createContext(null);

function AppProviders({ children }) {
  return (
    <ThemeContext.Provider value={/* theme */}>
      <AuthContext.Provider value={/* auth */}>
        <CartContext.Provider value={/* cart */}>
          {children}
        </CartContext.Provider>
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}

// หรือ compose providers
function composeProviders(...providers) {
  return ({ children }) => {
    return providers.reduceRight((child, Provider) => {
      return <Provider>{child}</Provider>;
    }, children);
  };
}

const AllProviders = composeProviders(
  ThemeProvider,
  AuthProvider,
  CartProvider
);

function App() {
  return (
    <AllProviders>
      <Main />
    </AllProviders>
  );
}
```

---

## Step 323: Context Performance

Context ทำให้ทุก consumer re-render เมื่อ value เปลี่ยน

```jsx
// ❌ Performance problem
function BadProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [cart, setCart] = useState([]);

  // ❌ Object ใหม่ทุก render → ทุก consumer re-render
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme, cart, setCart }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ แยก Contexts ตาม concern
function GoodProviders({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// ✅ ใช้ useMemo สำหรับ context value
function OptimizedProvider({ children }) {
  const [user, setUser] = useState(null);

  const value = useMemo(() => ({
    user,
    setUser,
    logout: () => setUser(null)
  }), [user]); // สร้าง object ใหม่เฉพาะเมื่อ user เปลี่ยน

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// ✅ แยก state กับ dispatch
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function SplitProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Components ที่ต้องการแค่ dispatch ไม่ re-render เมื่อ state เปลี่ยน
function ActionButton() {
  const dispatch = useContext(DispatchContext); // ✅ ไม่ re-render เมื่อ state เปลี่ยน
  return <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>;
}
```

---

## Step 324: ตัวอย่าง Theme Context

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

// Types of theme values
const themes = {
  light: {
    name: 'light',
    colors: {
      primary: '#007bff',
      background: '#ffffff',
      surface: '#f8f9fa',
      text: '#212529',
      textSecondary: '#6c757d',
      border: '#dee2e6',
      shadow: 'rgba(0, 0, 0, 0.1)'
    }
  },
  dark: {
    name: 'dark',
    colors: {
      primary: '#4da6ff',
      background: '#121212',
      surface: '#1e1e1e',
      text: '#e0e0e0',
      textSecondary: '#9e9e9e',
      border: '#333333',
      shadow: 'rgba(0, 0, 0, 0.3)'
    }
  }
};

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [themeName, setThemeName] = useState(() => {
    return localStorage.getItem('theme') || 
           (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  });

  const theme = themes[themeName];

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', themeName);
    localStorage.setItem('theme', themeName);
    
    // Apply CSS variables
    Object.entries(theme.colors).forEach(([key, value]) => {
      document.documentElement.style.setProperty(`--color-${key}`, value);
    });
  }, [themeName, theme]);

  const toggleTheme = () => {
    setThemeName(current => current === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, themeName, toggleTheme, setThemeName }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

// Component ที่ใช้ Theme
function ThemedCard({ title, children }) {
  const { theme } = useTheme();
  const { colors } = theme;
  
  return (
    <div style={{
      background: colors.surface,
      color: colors.text,
      border: `1px solid ${colors.border}`,
      borderRadius: '8px',
      padding: '20px',
      boxShadow: `0 2px 8px ${colors.shadow}`
    }}>
      <h3 style={{ color: colors.primary, marginBottom: '12px' }}>{title}</h3>
      {children}
    </div>
  );
}

function ThemeToggle() {
  const { themeName, toggleTheme } = useTheme();
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        display: 'flex',
        alignItems: 'center',
        gap: '8px',
        padding: '8px 16px',
        background: 'var(--color-primary)',
        color: 'white',
        border: 'none',
        borderRadius: '20px',
        cursor: 'pointer'
      }}
    >
      {themeName === 'dark' ? '☀️' : '🌙'}
      {themeName === 'dark' ? 'Light Mode' : 'Dark Mode'}
    </button>
  );
}

// App ที่ใช้ ThemeProvider
function App() {
  return (
    <ThemeProvider>
      <div style={{ minHeight: '100vh', background: 'var(--color-background)', padding: '20px' }}>
        <ThemeToggle />
        <div style={{ maxWidth: '800px', margin: '40px auto', display: 'grid', gap: '20px' }}>
          <ThemedCard title="Card One">
            <p>This card adapts to the theme automatically.</p>
          </ThemedCard>
          <ThemedCard title="Card Two">
            <p>All themed components update together!</p>
          </ThemedCard>
        </div>
      </div>
    </ThemeProvider>
  );
}
```

---

## Step 325: ตัวอย่าง Auth Context

```jsx
import { createContext, useContext, useState, useCallback } from 'react';

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const login = useCallback(async (email, password) => {
    setIsLoading(true);
    setError(null);
    
    try {
      // จำลอง API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      if (email === 'admin@test.com' && password === 'password') {
        const userData = {
          id: 1,
          email,
          name: 'Admin User',
          role: 'admin',
          avatar: `https://ui-avatars.com/api/?name=Admin+User`
        };
        setUser(userData);
        localStorage.setItem('user', JSON.stringify(userData));
        return { success: true };
      } else {
        throw new Error('Invalid email or password');
      }
    } catch (err) {
      setError(err.message);
      return { success: false, error: err.message };
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem('user');
  }, []);

  const updateProfile = useCallback(async (updates) => {
    setUser(prev => ({ ...prev, ...updates }));
  }, []);

  // Restore session จาก localStorage
  const [isInitialized, setIsInitialized] = useState(false);
  
  useEffect(() => {
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      try {
        setUser(JSON.parse(savedUser));
      } catch (e) {
        localStorage.removeItem('user');
      }
    }
    setIsInitialized(true);
  }, []);

  if (!isInitialized) {
    return <div>Initializing...</div>;
  }

  const value = {
    user,
    isAuthenticated: !!user,
    isLoading,
    error,
    login,
    logout,
    updateProfile
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}

// Protected Route
function ProtectedRoute({ children, requiredRole }) {
  const { isAuthenticated, user } = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  if (requiredRole && user.role !== requiredRole) {
    return <div>Access denied. Required role: {requiredRole}</div>;
  }
  
  return children;
}

// Login Form
function LoginForm() {
  const { login, isLoading, error } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    const result = await login(email, password);
    if (result.success) {
      console.log('Logged in!');
    }
  };

  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: '400px', margin: '0 auto' }}>
      <h2>Login</h2>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <input
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Email"
        style={{ display: 'block', width: '100%', marginBottom: '12px', padding: '8px' }}
      />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        placeholder="Password"
        style={{ display: 'block', width: '100%', marginBottom: '12px', padding: '8px' }}
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
      <p style={{ color: '#666', fontSize: '12px' }}>
        Test: admin@test.com / password
      </p>
    </form>
  );
}

// User Avatar with Auth
function UserAvatar() {
  const { user, isAuthenticated, logout } = useAuth();
  
  if (!isAuthenticated) {
    return <a href="/login">Login</a>;
  }
  
  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <img 
        src={user.avatar} 
        alt={user.name}
        style={{ width: '32px', height: '32px', borderRadius: '50%' }}
      />
      <span>{user.name}</span>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

## Step 326: ตัวอย่าง Language/i18n Context

```jsx
const translations = {
  th: {
    welcome: 'ยินดีต้อนรับ',
    login: 'เข้าสู่ระบบ',
    logout: 'ออกจากระบบ',
    settings: 'การตั้งค่า',
    products: 'สินค้า',
    cart: 'ตะกร้า',
    checkout: 'ชำระเงิน',
    total: 'ยอดรวม',
    search: 'ค้นหา',
    loading: 'กำลังโหลด...',
    error: 'เกิดข้อผิดพลาด'
  },
  en: {
    welcome: 'Welcome',
    login: 'Login',
    logout: 'Logout',
    settings: 'Settings',
    products: 'Products',
    cart: 'Cart',
    checkout: 'Checkout',
    total: 'Total',
    search: 'Search',
    loading: 'Loading...',
    error: 'An error occurred'
  },
  ja: {
    welcome: 'ようこそ',
    login: 'ログイン',
    logout: 'ログアウト',
    settings: '設定',
    products: '商品',
    cart: 'カート',
    checkout: '精算',
    total: '合計',
    search: '検索',
    loading: '読み込み中...',
    error: 'エラーが発生しました'
  }
};

const LanguageContext = createContext(null);

function LanguageProvider({ children, defaultLanguage = 'th' }) {
  const [language, setLanguage] = useState(() => {
    return localStorage.getItem('language') || defaultLanguage;
  });

  useEffect(() => {
    localStorage.setItem('language', language);
    document.documentElement.setAttribute('lang', language);
  }, [language]);

  const t = useCallback((key) => {
    return translations[language]?.[key] || translations['en']?.[key] || key;
  }, [language]);

  return (
    <LanguageContext.Provider value={{ language, setLanguage, t }}>
      {children}
    </LanguageContext.Provider>
  );
}

function useLanguage() {
  const context = useContext(LanguageContext);
  if (!context) throw new Error('useLanguage must be used within LanguageProvider');
  return context;
}

// LanguageSwitcher
function LanguageSwitcher() {
  const { language, setLanguage } = useLanguage();
  
  const languages = [
    { code: 'th', label: '🇹🇭 ไทย' },
    { code: 'en', label: '🇺🇸 English' },
    { code: 'ja', label: '🇯🇵 日本語' }
  ];
  
  return (
    <select
      value={language}
      onChange={e => setLanguage(e.target.value)}
      style={{ padding: '4px 8px', borderRadius: '4px' }}
    >
      {languages.map(lang => (
        <option key={lang.code} value={lang.code}>
          {lang.label}
        </option>
      ))}
    </select>
  );
}

// Component ที่ใช้ Translation
function Navbar() {
  const { t } = useLanguage();
  
  return (
    <nav style={{ display: 'flex', justifyContent: 'space-between', padding: '16px' }}>
      <div>
        <a href="/">{t('products')}</a>
        <a href="/cart">{t('cart')}</a>
        <a href="/settings">{t('settings')}</a>
      </div>
      <LanguageSwitcher />
    </nav>
  );
}
```

---

## Step 327: Context vs Props Drilling

```jsx
// เมื่อไหร่ใช้ Context?

// ✅ ใช้ Context: ข้อมูลที่ใช้หลาย levels ลึก
function ThemeDisplay() {
  const { theme } = useTheme(); // ดึงโดยตรง ไม่ต้องส่ง props ลงมา
  return <div>Current theme: {theme}</div>;
}

// ✅ ใช้ Props: ข้อมูลที่ส่งแค่ 1-2 levels
function ProductCard({ title, price, image }) { // Props ปกติสำหรับ simple case
  return (
    <div>
      <img src={image} alt={title} />
      <h3>{title}</h3>
      <p>{price}</p>
    </div>
  );
}

// ✅ Composition pattern แทน Props Drilling
// แทนที่จะส่ง user ผ่านทุก level:
function App() {
  const user = useUser();
  
  return (
    // ส่ง rendered component แทน data
    <Layout
      header={<UserAvatar user={user} />} // UserAvatar รับ user โดยตรง
    >
      <Dashboard />
    </Layout>
  );
}
```

---

## Step 328: Context กับ TypeScript

```typescript
import { createContext, useContext, ReactNode } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface AuthContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<boolean>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string): Promise<boolean> => {
    // implementation
    return true;
  };

  const logout = () => {
    setUser(null);
  };

  const value: AuthContextType = {
    user,
    isAuthenticated: !!user,
    login,
    logout
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

---

## Step 329: Anti-patterns ใน Context

### Anti-pattern 1: ใส่ทุกอย่างใน Context เดียว

```jsx
// ❌ God Context - ทุกอย่างรวมกัน
const AppContext = createContext({
  user: null,
  theme: 'light',
  cart: [],
  notifications: [],
  settings: {},
  // ... ทุกอย่าง
});

// ✅ แยก Contexts
const UserContext = createContext(null);
const ThemeContext = createContext(null);
const CartContext = createContext(null);
```

### Anti-pattern 2: Context สำหรับ state ที่เปลี่ยนเร็ว

```jsx
// ❌ ไม่ดี: ใส่ mouse position ใน Context → re-render ทุก component ที่ consume
const MouseContext = createContext({ x: 0, y: 0 });

function MouseProvider({ children }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    window.addEventListener('mousemove', e => setPos({ x: e.clientX, y: e.clientY }));
  }, []);
  
  return <MouseContext.Provider value={pos}>{children}</MouseContext.Provider>;
}

// ✅ ดีกว่า: ใช้ Custom Hook แทน (ไม่ใช้ Context)
function useMousePosition() {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handler = e => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);
  
  return pos;
}
```

---

## Step 330: Best Practices

### 1. Custom Hook สำหรับ Context

```jsx
// ✅ Always export custom hook, not the context directly
const ThemeContext = createContext(null); // ไม่ export ตรงๆ

export function useTheme() { // Export custom hook
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

### 2. Memoize Context Value

```jsx
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  // ✅ useMemo ป้องกัน re-render ที่ไม่จำเป็น
  const value = useMemo(() => ({
    user,
    setUser,
    isAdmin: user?.role === 'admin'
  }), [user]);
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}
```

### 3. แยก Read และ Write Contexts

```jsx
// ✅ Component ที่ไม่ต้อง dispatch ไม่ต้อง re-render
const CountStateContext = createContext(null);
const CountDispatchContext = createContext(null);

function CountProvider({ children }) {
  const [count, setCount] = useState(0);
  
  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={setCount}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// Component ที่แสดงเท่านั้น → re-render เมื่อ count เปลี่ยน
function CountDisplay() {
  const count = useContext(CountStateContext);
  return <p>Count: {count}</p>;
}

// Component ที่ action เท่านั้น → ไม่ re-render เมื่อ count เปลี่ยน
function CountButton() {
  const setCount = useContext(CountDispatchContext);
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

---

## สรุป Context API

```jsx
// Complete Pattern
const MyContext = createContext(null);

// Provider
function MyProvider({ children }) {
  const [state, setState] = useState(initialState);
  
  const value = useMemo(() => ({
    ...state,
    actions: { doSomething: () => setState(/* ... */) }
  }), [state]);
  
  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
}

// Custom Hook
function useMyContext() {
  const ctx = useContext(MyContext);
  if (!ctx) throw new Error('Must use within MyProvider');
  return ctx;
}

// Export
export { MyProvider, useMyContext };
```

---

## Quiz และแบบฝึกหัด

### Quiz

1. Context API แก้ปัญหา Props Drilling อย่างไร?
2. เมื่อไหร่ไม่ควรใช้ Context?
3. ทำไมควรแยก Context หลายตัวแทนที่จะใส่ทุกอย่างใน Context เดียว?
4. `useMemo` ช่วยกับ Context Performance อย่างไร?

### แบบฝึกหัด

**Exercise 1**: สร้าง Toast Notification Context

```jsx
// TODO: สร้าง ToastContext ที่:
// - มี toasts array (id, message, type: 'success'|'error'|'info')
// - มี addToast(message, type) function
// - มี removeToast(id) function
// - Auto-remove toast หลังจาก 3 วินาที
// - แสดง Toasts ใน fixed position

function ToastProvider({ children }) {
  // implement here...
}

function useToast() {
  // implement here...
}
```

**Exercise 2**: สร้าง Shopping Cart Context

```jsx
// TODO: สร้าง CartContext ที่มี:
// - items: CartItem[]
// - addItem(product): void
// - removeItem(productId): void
// - updateQuantity(productId, quantity): void
// - clearCart(): void
// - totalItems: number
// - totalPrice: number

function CartProvider({ children }) {
  // implement here...
}
```

---

➡️ ไปต่อ: [Part 16: useContext Hook ขั้นสูง](./part-16-useContext-hook.md)
