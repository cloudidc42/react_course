# Part 13: Styling in React

## Step 266-290 | ระดับ: กลาง

---

## สารบัญ

- [Step 266: วิธีจัดการ CSS ใน React](#step-266-วิธีจัดการ-css-ใน-react)
- [Step 267: Plain CSS ใน React](#step-267-plain-css-ใน-react)
- [Step 268: CSS Modules](#step-268-css-modules)
- [Step 269: Inline Styles](#step-269-inline-styles)
- [Step 270: CSS-in-JS ด้วย styled-components](#step-270-css-in-js-ด้วย-styled-components)
- [Step 271: Emotion CSS-in-JS](#step-271-emotion-css-in-js)
- [Step 272: Tailwind CSS ใน React](#step-272-tailwind-css-ใน-react)
- [Step 273: CSS Variables ใน React](#step-273-css-variables-ใน-react)
- [Step 274: Responsive Design](#step-274-responsive-design)
- [Step 275: Dark Mode](#step-275-dark-mode)
- [Step 276: Dynamic Styling](#step-276-dynamic-styling)
- [Step 277: Animation ใน React](#step-277-animation-ใน-react)
- [Step 278: clsx และ classnames Library](#step-278-clsx-และ-classnames-library)
- [Step 279: เมื่อไหร่ใช้ Styling แบบใด](#step-279-เมื่อไหร่ใช้-styling-แบบใด)
- [Step 280: Best Practices](#step-280-best-practices)
- [Quiz และแบบฝึกหัด](#quiz-และแบบฝึกหัด)

---

## Step 266: วิธีจัดการ CSS ใน React

React รองรับหลายวิธีในการ style components:

| วิธี | Library | ข้อดี | ข้อเสีย |
|---|---|---|---|
| Plain CSS | - | ง่าย, familiar | Global scope, naming conflicts |
| CSS Modules | built-in | Local scope | ต้องจัดการ class names |
| Inline Styles | - | Dynamic, no classes | ไม่รองรับ pseudo-classes |
| CSS-in-JS | styled-components, emotion | Component-scoped | Bundle size ใหญ่ขึ้น |
| Tailwind CSS | tailwindcss | Utility-first, fast | Class names ยาว |

---

## Step 267: Plain CSS ใน React

```jsx
// App.css
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
}

.button {
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.button-primary {
  background: #007bff;
  color: white;
  border: none;
}

.button-secondary {
  background: white;
  color: #007bff;
  border: 1px solid #007bff;
}
```

```jsx
// App.jsx
import './App.css';

function App() {
  return (
    <div className="container">
      <button className="button button-primary">Primary</button>
      <button className="button button-secondary">Secondary</button>
    </div>
  );
}
```

> **⚠️ ปัญหา**: CSS ทุกไฟล์อยู่ใน global scope อาจทำให้ class names ชนกัน

---

## Step 268: CSS Modules

CSS Modules สร้าง unique class names อัตโนมัติ ทำให้ styles เป็น local scope

### การสร้างไฟล์

```
src/
  components/
    Button/
      Button.module.css
      Button.jsx
```

```css
/* Button.module.css */
.button {
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.primary {
  background: #007bff;
  color: white;
  border: none;
}

.primary:hover {
  background: #0056b3;
}

.secondary {
  background: white;
  color: #007bff;
  border: 1px solid #007bff;
}

.secondary:hover {
  background: #f0f0f0;
}

.large {
  padding: 12px 24px;
  font-size: 16px;
}

.small {
  padding: 4px 8px;
  font-size: 12px;
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium',
  onClick,
  disabled 
}) {
  // รวม class names จาก CSS Modules
  const classNames = [
    styles.button,
    styles[variant],
    size !== 'medium' && styles[size]
  ].filter(Boolean).join(' ');

  return (
    <button 
      className={classNames}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

export default Button;
```

```jsx
// การใช้งาน
import Button from './components/Button/Button';

function App() {
  return (
    <div>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary" size="large">Large Secondary</Button>
      <Button variant="primary" size="small">Small Primary</Button>
    </div>
  );
}
```

### CSS Modules ที่ซับซ้อนกว่า

```css
/* Card.module.css */
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.cardImage {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.cardBody {
  padding: 16px;
}

.cardTitle {
  margin: 0 0 8px;
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

.cardDescription {
  margin: 0 0 16px;
  font-size: 14px;
  color: #666;
  line-height: 1.5;
}

.cardFooter {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-top: 1px solid #f0f0f0;
}

/* composes: ใช้ inherit styles จาก class อื่น */
.featuredCard {
  composes: card;
  border: 2px solid #007bff;
}
```

```jsx
// Card.jsx
import styles from './Card.module.css';

function Card({ image, title, description, footer, featured }) {
  return (
    <div className={featured ? styles.featuredCard : styles.card}>
      {image && (
        <img src={image} alt={title} className={styles.cardImage} />
      )}
      <div className={styles.cardBody}>
        <h3 className={styles.cardTitle}>{title}</h3>
        {description && (
          <p className={styles.cardDescription}>{description}</p>
        )}
      </div>
      {footer && (
        <div className={styles.cardFooter}>{footer}</div>
      )}
    </div>
  );
}
```

---

## Step 269: Inline Styles

Inline styles เหมาะสำหรับ dynamic styles ที่ขึ้นกับ state

```jsx
function DynamicBox() {
  const [color, setColor] = useState('#007bff');
  const [size, setSize] = useState(100);
  const [rotation, setRotation] = useState(0);

  const boxStyle = {
    width: `${size}px`,
    height: `${size}px`,
    backgroundColor: color,
    transform: `rotate(${rotation}deg)`,
    borderRadius: '8px',
    transition: 'all 0.3s ease',
    cursor: 'pointer',
    margin: '20px auto',
    display: 'block'
  };

  return (
    <div>
      <div style={boxStyle} onClick={() => setRotation(r => r + 45)} />
      <div>
        <label>Color: </label>
        <input 
          type="color" 
          value={color} 
          onChange={e => setColor(e.target.value)} 
        />
        <label> Size: {size}px </label>
        <input
          type="range"
          min="50"
          max="200"
          value={size}
          onChange={e => setSize(Number(e.target.value))}
        />
      </div>
    </div>
  );
}
```

### Inline Styles ข้อจำกัด

```jsx
// ❌ Inline styles ไม่รองรับ pseudo-classes และ media queries
const style = {
  ':hover': { color: 'red' },    // ❌ ไม่ทำงาน
  '@media (max-width: 768px)': { // ❌ ไม่ทำงาน
    fontSize: '14px'
  }
};

// ✅ ใช้ CSS classes สำหรับ hover/media queries
// ใช้ inline styles เฉพาะ dynamic values
function GoodInlineStyles({ isActive, progress }) {
  return (
    <div
      className="progress-container"  // ใช้ class สำหรับ static styles
      style={{
        // ใช้ inline สำหรับ dynamic values เท่านั้น
        '--progress': `${progress}%`,
        opacity: isActive ? 1 : 0.5,
      }}
    />
  );
}
```

---

## Step 270: CSS-in-JS ด้วย styled-components

```bash
npm install styled-components
```

```jsx
import styled from 'styled-components';

// สร้าง styled component
const Button = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
  
  /* Dynamic styles จาก props */
  background: ${props => props.variant === 'primary' ? '#007bff' : 'white'};
  color: ${props => props.variant === 'primary' ? 'white' : '#007bff'};
  border: 1px solid #007bff;
  
  &:hover {
    opacity: 0.9;
    transform: translateY(-1px);
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

const Card = styled.div`
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  
  &:hover {
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  }
`;

const Title = styled.h2`
  font-size: ${props => props.size || '24px'};
  color: ${props => props.color || '#1a1a1a'};
  margin: 0 0 16px;
`;

// การใช้งาน
function App() {
  return (
    <Card>
      <Title size="20px" color="#007bff">Hello Styled Components</Title>
      <Button variant="primary">Click Me</Button>
      <Button>Secondary</Button>
    </Card>
  );
}
```

### Theming ด้วย styled-components

```jsx
import styled, { ThemeProvider } from 'styled-components';

// กำหนด theme
const lightTheme = {
  colors: {
    primary: '#007bff',
    background: '#ffffff',
    text: '#1a1a1a',
    border: '#e0e0e0'
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px'
  },
  borderRadius: '4px'
};

const darkTheme = {
  colors: {
    primary: '#4da6ff',
    background: '#1a1a1a',
    text: '#ffffff',
    border: '#404040'
  },
  spacing: lightTheme.spacing,
  borderRadius: lightTheme.borderRadius
};

// Components ที่ใช้ theme
const ThemedButton = styled.button`
  background: ${({ theme }) => theme.colors.primary};
  color: white;
  padding: ${({ theme }) => `${theme.spacing.small} ${theme.spacing.medium}`};
  border-radius: ${({ theme }) => theme.borderRadius};
  border: none;
  cursor: pointer;
`;

const ThemedContainer = styled.div`
  background: ${({ theme }) => theme.colors.background};
  color: ${({ theme }) => theme.colors.text};
  min-height: 100vh;
  padding: ${({ theme }) => theme.spacing.large};
`;

function App() {
  const [isDark, setIsDark] = useState(false);
  
  return (
    <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
      <ThemedContainer>
        <h1>Themed App</h1>
        <ThemedButton onClick={() => setIsDark(d => !d)}>
          Toggle {isDark ? 'Light' : 'Dark'} Mode
        </ThemedButton>
      </ThemedContainer>
    </ThemeProvider>
  );
}
```

---

## Step 271: Emotion CSS-in-JS

```bash
npm install @emotion/react @emotion/styled
```

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';
import styled from '@emotion/styled';

// 1. css prop
const containerStyle = css`
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
`;

function App() {
  return (
    <div css={containerStyle}>
      <h1>Emotion Example</h1>
    </div>
  );
}

// 2. Dynamic css
function DynamicComponent({ isActive, color }) {
  return (
    <div
      css={css`
        background: ${isActive ? color : '#ccc'};
        padding: 16px;
        border-radius: 4px;
        transition: all 0.3s;
      `}
    >
      {isActive ? 'Active' : 'Inactive'}
    </div>
  );
}

// 3. Styled components (เหมือน styled-components)
const StyledCard = styled.div`
  background: white;
  border-radius: 8px;
  padding: ${props => props.compact ? '8px' : '16px'};
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
`;
```

---

## Step 272: Tailwind CSS ใน React

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```js
// tailwind.config.js
module.exports = {
  content: [
    './index.html',
    './src/**/*.{js,jsx,ts,tsx}'
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a'
        }
      }
    }
  },
  plugins: []
};
```

```css
/* index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```jsx
// Tailwind components
function TailwindButton({ children, variant = 'primary', size = 'md' }) {
  const baseClasses = 'rounded font-medium transition-all cursor-pointer';
  
  const variantClasses = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600 active:bg-blue-700',
    secondary: 'bg-white text-blue-500 border border-blue-500 hover:bg-blue-50',
    danger: 'bg-red-500 text-white hover:bg-red-600',
    ghost: 'text-gray-700 hover:bg-gray-100'
  };

  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };

  return (
    <button className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`}>
      {children}
    </button>
  );
}

// Card ด้วย Tailwind
function TailwindCard({ title, description, image, badge }) {
  return (
    <div className="bg-white rounded-xl shadow-md overflow-hidden hover:shadow-xl transition-shadow duration-300">
      {image && (
        <img 
          src={image} 
          alt={title}
          className="w-full h-48 object-cover"
        />
      )}
      <div className="p-6">
        {badge && (
          <span className="inline-block bg-blue-100 text-blue-600 text-xs font-semibold px-2.5 py-0.5 rounded-full mb-2">
            {badge}
          </span>
        )}
        <h3 className="text-xl font-bold text-gray-900 mb-2">{title}</h3>
        {description && (
          <p className="text-gray-600 text-sm leading-relaxed">{description}</p>
        )}
      </div>
    </div>
  );
}

// Responsive Grid
function ProductGrid({ products }) {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6 p-6">
      {products.map(product => (
        <TailwindCard
          key={product.id}
          title={product.title}
          description={product.description}
          image={product.image}
        />
      ))}
    </div>
  );
}
```

### Tailwind + Dynamic Classes

```jsx
// ⚠️ อย่า concatenate Tailwind classes แบบนี้
function BadTailwind({ color }) {
  return <div className={`bg-${color}-500`}>...</div>; // ❌ ไม่ work กับ purge
}

// ✅ ใช้ object mapping แทน
function GoodTailwind({ color }) {
  const colorMap = {
    blue: 'bg-blue-500',
    red: 'bg-red-500',
    green: 'bg-green-500'
  };
  
  return <div className={colorMap[color] || 'bg-gray-500'}>...</div>;
}
```

---

## Step 273: CSS Variables ใน React

```css
/* index.css */
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  
  --border-radius: 4px;
  --border-radius-lg: 8px;
  
  --font-size-sm: 12px;
  --font-size-base: 16px;
  --font-size-lg: 20px;
  --font-size-xl: 24px;
  
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.12);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
  
  /* Dark mode variables */
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --text-primary: #212529;
  --text-secondary: #6c757d;
}

[data-theme="dark"] {
  --bg-primary: #1a1a1a;
  --bg-secondary: #2d2d2d;
  --text-primary: #ffffff;
  --text-secondary: #adb5bd;
}
```

```jsx
// การใช้ CSS Variables ใน React
function ThemeManager() {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
  }, [theme]);

  return (
    <div style={{ background: 'var(--bg-primary)', color: 'var(--text-primary)', minHeight: '100vh' }}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}

// Dynamic CSS Variables
function ColorPicker() {
  const [primaryColor, setPrimaryColor] = useState('#007bff');
  const [borderRadius, setBorderRadius] = useState(4);

  useEffect(() => {
    document.documentElement.style.setProperty('--color-primary', primaryColor);
    document.documentElement.style.setProperty('--border-radius', `${borderRadius}px`);
  }, [primaryColor, borderRadius]);

  return (
    <div>
      <label>
        Primary Color:
        <input
          type="color"
          value={primaryColor}
          onChange={e => setPrimaryColor(e.target.value)}
        />
      </label>
      <label>
        Border Radius: {borderRadius}px
        <input
          type="range"
          min="0"
          max="20"
          value={borderRadius}
          onChange={e => setBorderRadius(Number(e.target.value))}
        />
      </label>
      <button style={{
        background: 'var(--color-primary)',
        color: 'white',
        padding: '8px 16px',
        borderRadius: 'var(--border-radius)',
        border: 'none',
        cursor: 'pointer'
      }}>
        Sample Button
      </button>
    </div>
  );
}
```

---

## Step 274: Responsive Design

```jsx
// Hook สำหรับ responsive design
function useMediaQuery(query) {
  const [matches, setMatches] = useState(
    () => window.matchMedia(query).matches
  );

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    
    const handleChange = (e) => setMatches(e.matches);
    
    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, [query]);

  return matches;
}

// Custom hook สำหรับ breakpoints
function useBreakpoint() {
  const isMobile = useMediaQuery('(max-width: 767px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1023px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  
  if (isMobile) return 'mobile';
  if (isTablet) return 'tablet';
  return 'desktop';
}

// การใช้งาน
function ResponsiveLayout() {
  const breakpoint = useBreakpoint();
  const isMobile = breakpoint === 'mobile';

  return (
    <div style={{
      display: 'flex',
      flexDirection: isMobile ? 'column' : 'row',
      gap: '20px'
    }}>
      <aside style={{
        width: isMobile ? '100%' : '250px',
        background: '#f0f0f0',
        padding: '20px'
      }}>
        <h3>Sidebar</h3>
        <p>Device: {breakpoint}</p>
      </aside>
      <main style={{ flex: 1, padding: '20px' }}>
        <h1>Main Content</h1>
      </main>
    </div>
  );
}
```

### CSS Grid Responsive

```css
/* responsive-grid.module.css */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
  padding: 20px;
}

.hero {
  display: grid;
  grid-template-columns: 1fr;
  gap: 30px;
  align-items: center;
}

@media (min-width: 768px) {
  .hero {
    grid-template-columns: 1fr 1fr;
  }
}

@media (min-width: 1024px) {
  .hero {
    grid-template-columns: 3fr 2fr;
  }
}
```

---

## Step 275: Dark Mode

```jsx
// Dark Mode Context
import { createContext, useContext, useState, useEffect } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [isDark, setIsDark] = useState(() => {
    // ตรวจสอบ preference จาก localStorage
    const saved = localStorage.getItem('theme');
    if (saved) return saved === 'dark';
    
    // ตรวจสอบ system preference
    return window.matchMedia('(prefers-color-scheme: dark)').matches;
  });

  useEffect(() => {
    // อัปเดต CSS class และ localStorage
    document.documentElement.classList.toggle('dark', isDark);
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  // Listen to system preference changes
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    
    const handleChange = (e) => {
      // อัปเดตเฉพาะถ้ายังไม่มี saved preference
      if (!localStorage.getItem('theme')) {
        setIsDark(e.matches);
      }
    };
    
    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return (
    <ThemeContext.Provider value={{ isDark, setIsDark, toggleDark: () => setIsDark(d => !d) }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

// Dark Mode Toggle Button
function DarkModeToggle() {
  const { isDark, toggleDark } = useTheme();
  
  return (
    <button
      onClick={toggleDark}
      style={{
        background: 'none',
        border: '1px solid currentColor',
        borderRadius: '20px',
        padding: '6px 12px',
        cursor: 'pointer',
        display: 'flex',
        alignItems: 'center',
        gap: '6px'
      }}
      aria-label={`Switch to ${isDark ? 'light' : 'dark'} mode`}
    >
      {isDark ? '☀️ Light' : '🌙 Dark'}
    </button>
  );
}
```

```css
/* Dark mode CSS */
:root {
  --bg: #ffffff;
  --text: #1a1a1a;
  --card-bg: #f8f9fa;
  --border: #e0e0e0;
}

.dark {
  --bg: #0f0f0f;
  --text: #e0e0e0;
  --card-bg: #1e1e1e;
  --border: #404040;
}

body {
  background-color: var(--bg);
  color: var(--text);
  transition: background-color 0.3s, color 0.3s;
}

.card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 16px;
}
```

---

## Step 276: Dynamic Styling

### Pattern 1: Conditional Classes

```jsx
import { useState } from 'react';
import styles from './Toggle.module.css';

function Toggle({ defaultOn = false, onChange, label }) {
  const [isOn, setIsOn] = useState(defaultOn);

  const handleToggle = () => {
    const newValue = !isOn;
    setIsOn(newValue);
    onChange?.(newValue);
  };

  return (
    <div className={styles.toggleContainer} onClick={handleToggle}>
      <div className={`${styles.track} ${isOn ? styles.trackOn : styles.trackOff}`}>
        <div className={`${styles.thumb} ${isOn ? styles.thumbOn : ''}`} />
      </div>
      {label && <span className={styles.label}>{label}</span>}
    </div>
  );
}
```

### Pattern 2: Style Objects

```jsx
function ProgressBar({ value, max = 100, color, showLabel = true }) {
  const percentage = Math.min(100, Math.max(0, (value / max) * 100));
  
  const getColor = () => {
    if (color) return color;
    if (percentage < 30) return '#dc3545';
    if (percentage < 70) return '#ffc107';
    return '#28a745';
  };

  const containerStyle = {
    background: '#e9ecef',
    borderRadius: '4px',
    height: '8px',
    overflow: 'hidden',
    position: 'relative'
  };

  const fillStyle = {
    width: `${percentage}%`,
    height: '100%',
    background: getColor(),
    transition: 'width 0.3s ease, background 0.3s ease',
    borderRadius: '4px'
  };

  return (
    <div>
      <div style={containerStyle}>
        <div style={fillStyle} role="progressbar" aria-valuenow={value} aria-valuemax={max} />
      </div>
      {showLabel && (
        <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: '4px', fontSize: '12px', color: '#666' }}>
          <span>{value}</span>
          <span>{percentage.toFixed(0)}%</span>
        </div>
      )}
    </div>
  );
}
```

### Pattern 3: CSS Custom Properties

```jsx
function RatingStars({ rating, maxRating = 5 }) {
  const percentage = (rating / maxRating) * 100;
  
  return (
    <div
      style={{
        '--rating': rating,
        '--max-rating': maxRating,
        '--percentage': `${percentage}%`
      }}
      className="star-rating"
    >
      {'★'.repeat(maxRating)}
    </div>
  );
}
```

```css
.star-rating {
  font-size: 24px;
  color: #ddd;
  position: relative;
  display: inline-block;
}

.star-rating::after {
  content: '★★★★★';
  position: absolute;
  left: 0;
  color: #ffc107;
  width: var(--percentage);
  overflow: hidden;
}
```

---

## Step 277: Animation ใน React

### CSS Transitions

```css
/* animations.module.css */
.fadeIn {
  animation: fadeIn 0.3s ease forwards;
}

.fadeOut {
  animation: fadeOut 0.3s ease forwards;
}

.slideIn {
  animation: slideIn 0.3s ease forwards;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes fadeOut {
  from { opacity: 1; transform: translateY(0); }
  to { opacity: 0; transform: translateY(-10px); }
}

@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.loadingPulse {
  animation: pulse 1.5s ease infinite;
}
```

```jsx
import styles from './animations.module.css';

function AnimatedAlert({ message, type = 'info' }) {
  const [isVisible, setIsVisible] = useState(true);
  const [isLeaving, setIsLeaving] = useState(false);

  const handleClose = () => {
    setIsLeaving(true);
    setTimeout(() => setIsVisible(false), 300);
  };

  if (!isVisible) return null;

  const alertColors = {
    info: { bg: '#cce5ff', text: '#004085', border: '#b8daff' },
    success: { bg: '#d4edda', text: '#155724', border: '#c3e6cb' },
    warning: { bg: '#fff3cd', text: '#856404', border: '#ffeeba' },
    error: { bg: '#f8d7da', text: '#721c24', border: '#f5c6cb' }
  };

  const colors = alertColors[type];

  return (
    <div
      className={isLeaving ? styles.fadeOut : styles.fadeIn}
      style={{
        background: colors.bg,
        color: colors.text,
        border: `1px solid ${colors.border}`,
        borderRadius: '4px',
        padding: '12px 16px',
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: '10px'
      }}
    >
      <span>{message}</span>
      <button
        onClick={handleClose}
        style={{
          background: 'none',
          border: 'none',
          cursor: 'pointer',
          fontSize: '18px',
          color: 'inherit',
          padding: '0 4px'
        }}
      >
        ×
      </button>
    </div>
  );
}
```

---

## Step 278: clsx และ classnames Library

```bash
npm install clsx
# หรือ
npm install classnames
```

```jsx
import clsx from 'clsx';

function Button({ 
  children, 
  variant = 'primary', 
  size = 'md',
  isLoading = false,
  isDisabled = false,
  fullWidth = false,
  className,
  ...props 
}) {
  const classes = clsx(
    // Base classes (always applied)
    'button',
    
    // Variant classes
    {
      'button--primary': variant === 'primary',
      'button--secondary': variant === 'secondary',
      'button--danger': variant === 'danger',
      'button--ghost': variant === 'ghost',
    },
    
    // Size classes
    {
      'button--sm': size === 'sm',
      'button--md': size === 'md',
      'button--lg': size === 'lg',
    },
    
    // State classes
    {
      'button--loading': isLoading,
      'button--disabled': isDisabled,
      'button--full-width': fullWidth,
    },
    
    // Additional classes from props
    className
  );

  return (
    <button
      className={classes}
      disabled={isDisabled || isLoading}
      {...props}
    >
      {isLoading ? (
        <span className="button__spinner">Loading...</span>
      ) : children}
    </button>
  );
}

// การใช้งาน
function App() {
  return (
    <div>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary" size="lg">Large Secondary</Button>
      <Button variant="danger" isLoading>Loading...</Button>
      <Button variant="ghost" isDisabled>Disabled</Button>
      <Button fullWidth className="my-custom-class">Full Width</Button>
    </div>
  );
}
```

---

## Step 279: เมื่อไหร่ใช้ Styling แบบใด

### Decision Guide

```
สถานการณ์                          แนะนำวิธี
────────────────────────────────────────────────────
โปรเจกต์ขนาดเล็ก/ทดสอบ            Plain CSS หรือ Inline Styles
Component library / Design system   CSS Modules
แอปขนาดใหญ่ที่ต้องการ Theming      styled-components / Emotion
ต้องการ Utility-first / DX ดี       Tailwind CSS
Enterprise / จำนวนนักพัฒนาเยอะ     CSS Modules + CSS Variables
Mobile-first / Performance          CSS Modules
```

### ตัวอย่างการเลือก

```jsx
// ✅ Inline styles: Dynamic values
<div style={{ 
  transform: `rotate(${angle}deg)`,  // Dynamic
  opacity: isVisible ? 1 : 0         // Dynamic
}} />

// ✅ CSS Modules: Component styles
import styles from './Card.module.css';
<div className={styles.card} />

// ✅ Tailwind: Layout & spacing
<div className="flex items-center gap-4 p-6 rounded-lg shadow-md" />

// ✅ CSS Variables: Theming
<div style={{ '--primary-color': brandColor }} />
```

---

## Step 280: Best Practices

### 1. Avoid Style Duplication

```jsx
// ❌ ซ้ำกัน
function Button() {
  const base = { padding: '8px 16px', borderRadius: '4px' };
  return <button style={base}>Click</button>;
}

function Input() {
  const base = { padding: '8px 16px', borderRadius: '4px' }; // ซ้ำ!
  return <input style={base} />;
}

// ✅ Share design tokens
const tokens = {
  padding: { sm: '4px 8px', md: '8px 16px', lg: '12px 24px' },
  borderRadius: '4px',
  colors: { primary: '#007bff' }
};
```

### 2. Semantic Class Names

```css
/* ❌ ไม่ดี: เปลี่ยนสีแล้วต้องเปลี่ยน class name ด้วย */
.red-button { background: red; }
.big-font { font-size: 24px; }

/* ✅ ดี: ชื่อบอก purpose ไม่ใช่ appearance */
.button-danger { background: red; }
.heading-primary { font-size: 24px; }
```

### 3. Performance Tips

```jsx
// ✅ ใช้ useMemo สำหรับ complex style calculations
function ExpensiveStyled({ data, isActive }) {
  const computedStyle = useMemo(() => ({
    background: computeGradient(data),
    transform: computeTransform(data),
    filter: isActive ? 'brightness(1.2)' : 'none'
  }), [data, isActive]);

  return <div style={computedStyle}>...</div>;
}

// ✅ อย่าสร้าง style object ใน render ถ้าไม่จำเป็น
// ❌ สร้าง object ใหม่ทุก render
function BadStyle() {
  return <div style={{ padding: '16px', margin: '8px' }}>...</div>;
}

// ✅ ใช้ CSS class แทน ถ้า style ไม่ dynamic
function GoodStyle() {
  return <div className={styles.container}>...</div>;
}
```

---

## สรุป

| วิธี | ใช้เมื่อ |
|---|---|
| Plain CSS | โปรเจกต์เล็ก, ทดสอบ |
| CSS Modules | Component scoping, ทีมใหญ่ |
| Inline Styles | Dynamic values, one-off styles |
| styled-components | Theming, component libraries |
| Tailwind CSS | Rapid development, utility-first |
| CSS Variables | Theming, design tokens |

---

## Quiz และแบบฝึกหัด

### Quiz

1. CSS Modules แก้ปัญหาอะไรของ Plain CSS?
2. ทำไม Tailwind ถึงไม่ควรใช้ string concatenation สำหรับ class names?
3. Inline styles มีข้อจำกัดอะไร?
4. CSS Variables ช่วยอะไรใน React?

### แบบฝึกหัด

**Exercise 1**: สร้าง `ThemeToggle` component ด้วย CSS Modules ที่มี animation

**Exercise 2**: แปลง component ด้วย Inline styles ไปใช้ Tailwind CSS

```jsx
// แปลงสิ่งนี้เป็น Tailwind:
const style = {
  display: 'flex',
  alignItems: 'center',
  padding: '16px',
  background: '#ffffff',
  borderRadius: '8px',
  boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
  gap: '12px'
};
```

**Exercise 3**: สร้าง Dark Mode feature ด้วย CSS Variables และ useEffect

---

➡️ ไปต่อ: [Part 14: Component Composition](./part-14-component-composition.md)
