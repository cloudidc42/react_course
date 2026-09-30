# Part 02: ติดตั้งและตั้งค่า Development Environment
## Step 11-25 | ระดับ: พื้นฐาน

---

## 📋 สิ่งที่จะได้เรียนรู้ใน Part นี้

- ติดตั้ง Node.js และ npm
- ทำความรู้จัก Package Manager (npm, yarn, pnpm)
- สร้าง React Project ด้วย Vite
- โครงสร้างไฟล์ที่ได้หลังสร้าง Project
- ทำความรู้จัก VS Code Extensions ที่จำเป็น
- คำสั่ง npm พื้นฐาน
- รัน Development Server ครั้งแรก
- เข้าใจ package.json

---

## Step 11: ติดตั้ง Node.js

### Node.js คืออะไร?

**Node.js** คือ JavaScript Runtime ที่ทำให้เราสามารถรัน JavaScript นอก Browser ได้ React ต้องการ Node.js เพื่อ:
- รัน Build Tools (Vite, Webpack)
- จัดการ Dependencies (npm)
- รัน Development Server

### ดาวน์โหลดและติดตั้ง Node.js

**วิธีที่ 1: ดาวน์โหลดโดยตรง (แนะนำสำหรับผู้เริ่มต้น)**
```
1. ไปที่ https://nodejs.org
2. ดาวน์โหลด LTS Version (Long Term Support)
3. รัน Installer และทำตามขั้นตอน
4. Restart Terminal/Command Prompt
```

**วิธีที่ 2: ใช้ NVM (Node Version Manager) - แนะนำสำหรับนักพัฒนา**
```bash
# macOS/Linux: ติดตั้ง NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# หรือใช้ Homebrew (macOS)
brew install nvm

# Windows: ดาวน์โหลด nvm-windows จาก GitHub
# https://github.com/coreybutler/nvm-windows

# ติดตั้ง Node.js เวอร์ชันล่าสุด LTS
nvm install --lts

# ใช้งาน Node.js เวอร์ชัน LTS
nvm use --lts

# แสดง Node.js เวอร์ชันที่ติดตั้ง
nvm list
```

### ตรวจสอบการติดตั้ง

```bash
# ตรวจสอบ Node.js version
node --version
# ควรแสดง: v20.x.x หรือ v22.x.x

# ตรวจสอบ npm version
npm --version
# ควรแสดง: 10.x.x

# ทดสอบรัน JavaScript
node -e "console.log('Hello Node.js!')"
# ควรแสดง: Hello Node.js!
```

---

## Step 12: ทำความรู้จัก Package Manager

### npm (Node Package Manager)

**npm** ติดมากับ Node.js โดยอัตโนมัติ เป็น Package Manager ที่ใช้จัดการ JavaScript Libraries

```bash
# คำสั่ง npm พื้นฐาน

# ติดตั้ง Package
npm install package-name
npm install package-name@version   # ระบุ version
npm install package-name --save-dev # สำหรับ Dev เท่านั้น

# ถอนการติดตั้ง Package
npm uninstall package-name

# อัปเดต Package
npm update package-name
npm update               # อัปเดตทุก Package

# ดู Package ที่ติดตั้ง
npm list
npm list --depth=0      # แสดงแค่ Top level

# ดูข้อมูล Package
npm info package-name

# ค้นหา Package
npm search keyword

# รัน Script ใน package.json
npm run script-name
npm start               # รัน start script
npm test                # รัน test script
npm run build           # รัน build script
```

### yarn (Yet Another Resource Negotiator)

```bash
# ติดตั้ง yarn
npm install -g yarn

# คำสั่ง yarn
yarn add package-name         # ติดตั้ง Package
yarn add package-name --dev   # Dev dependency
yarn remove package-name      # ถอนการติดตั้ง
yarn install                  # ติดตั้งจาก yarn.lock
yarn upgrade                  # อัปเดต
yarn run script-name          # รัน Script
```

### pnpm (Performant npm)

```bash
# ติดตั้ง pnpm
npm install -g pnpm

# คำสั่ง pnpm
pnpm add package-name         # ติดตั้ง Package
pnpm add -D package-name      # Dev dependency
pnpm remove package-name      # ถอนการติดตั้ง
pnpm install                  # ติดตั้งจาก lock file
pnpm run script-name          # รัน Script
```

### เปรียบเทียบ Package Managers

| คุณสมบัติ | npm | yarn | pnpm |
|-----------|-----|------|------|
| ความเร็ว | ปานกลาง | เร็ว | เร็วที่สุด |
| Disk Space | ปกติ | ปกติ | ประหยัดสูงสุด |
| Lock File | package-lock.json | yarn.lock | pnpm-lock.yaml |
| Workspace | npm workspaces | yarn workspaces | pnpm workspaces |
| ความนิยม | สูงสุด | สูง | กำลังเพิ่ม |

> 💡 **แนะนำ**: สำหรับมือใหม่ใช้ **npm** เพราะติดมากับ Node.js และเป็นมาตรฐาน สำหรับ Production Project แนะนำ **pnpm**

---

## Step 13: สร้าง React Project ด้วย Vite

### Vite คืออะไร?

**Vite** (อ่านว่า "วีท" แปลว่า "เร็ว" ในภาษาฝรั่งเศส) คือ Build Tool รุ่นใหม่ที่:
- รวดเร็วกว่า Create React App มาก
- ใช้ ES Modules แบบ Native
- Hot Module Replacement (HMR) ที่เร็วมาก
- ปัจจุบันเป็น มาตรฐานใหม่ สำหรับ React

### สร้าง React Project

```bash
# วิธีที่ 1: ใช้ npm
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev

# วิธีที่ 2: ใช้ yarn
yarn create vite my-react-app --template react
cd my-react-app
yarn install
yarn dev

# วิธีที่ 3: ใช้ pnpm
pnpm create vite my-react-app --template react
cd my-react-app
pnpm install
pnpm dev

# วิธีที่ 4: Interactive Mode (ถามทีละขั้น)
npm create vite@latest
# ? Project name: my-react-app
# ? Select a framework: React
# ? Select a variant: JavaScript (หรือ TypeScript)
```

### Templates ที่ Vite รองรับ

```bash
# React + JavaScript
npm create vite@latest -- --template react

# React + TypeScript (แนะนำสำหรับ Production)
npm create vite@latest -- --template react-ts

# React + TypeScript + SWC (เร็วกว่า Babel)
npm create vite@latest -- --template react-swc-ts
```

### เปรียบเทียบ Create React App vs Vite

| | Create React App | Vite |
|--|-----------------|------|
| เวลาสร้าง Project | 2-5 นาที | 5-10 วินาที |
| Cold Start | 10-30 วินาที | < 1 วินาที |
| HMR | 1-5 วินาที | < 50ms |
| Bundle Tool | Webpack | Rollup |
| Status | Deprecated | แนะนำ ✅ |

---

## Step 14: ทำความเข้าใจโครงสร้างไฟล์

หลังจากรัน `npm create vite@latest my-react-app -- --template react` จะได้โครงสร้างนี้:

```
my-react-app/
├── 📁 public/
│   └── vite.svg              ← Icon สำหรับ Vite
├── 📁 src/
│   ├── 📁 assets/
│   │   └── react.svg         ← Logo React
│   ├── App.css               ← CSS ของ App Component
│   ├── App.jsx               ← Root Component
│   ├── index.css             ← Global CSS
│   └── main.jsx              ← Entry Point
├── .eslintrc.cjs             ← ESLint Configuration
├── .gitignore                ← Git Ignore Rules
├── index.html                ← HTML Template หลัก
├── package.json              ← Project Config & Dependencies
├── vite.config.js            ← Vite Configuration
└── README.md                 ← Project Documentation
```

### อ่านแต่ละไฟล์สำคัญ

**`package.json`** - หัวใจของ Project
```json
{
  "name": "my-react-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",              // รัน Dev Server
    "build": "vite build",      // สร้าง Production Build
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"   // Preview Production Build
  },
  "dependencies": {
    "react": "^18.2.0",         // React Core
    "react-dom": "^18.2.0"      // React DOM Renderer
  },
  "devDependencies": {
    "@types/react": "^18.2.66",
    "@types/react-dom": "^18.2.22",
    "@vitejs/plugin-react": "^4.2.1",  // Vite Plugin สำหรับ React
    "eslint": "^8.57.0",
    "eslint-plugin-react": "^7.34.1",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.6",
    "vite": "^5.2.0"            // Vite Build Tool
  }
}
```

**`vite.config.js`** - การตั้งค่า Vite
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  // สามารถเพิ่ม config อื่นๆ ได้
  server: {
    port: 3000,          // เปลี่ยน Port
    open: true,          // เปิด Browser อัตโนมัติ
  },
})
```

**`index.html`** - HTML Template หลัก
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React</title>
  </head>
  <body>
    <!-- React App จะ Render ที่นี่ -->
    <div id="root"></div>
    <!-- Entry Point -->
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

---

## Step 15: ติดตั้งและตั้งค่า VS Code

### ดาวน์โหลด VS Code

```
ไปที่: https://code.visualstudio.com
ดาวน์โหลดและติดตั้ง VS Code สำหรับ OS ของคุณ
```

### Extensions ที่จำเป็นสำหรับ React

**1. ES7+ React/Redux/React-Native snippets**
```
ID: dsznajder.es7-react-js-snippets
ใช้สำหรับ: Shortcut สร้าง Component template เร็วๆ

Shortcuts สำคัญ:
  rafce  → React Arrow Function Component (Export)
  rfce   → React Function Component (Export)
  rce    → React Class Export Component
  useState snippet, useEffect snippet
```

**2. Prettier - Code formatter**
```
ID: esbenp.prettier-vscode
ใช้สำหรับ: จัดรูปแบบโค้ดอัตโนมัติ

ตั้งค่าใน settings.json:
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

**3. ESLint**
```
ID: dbaeumer.vscode-eslint
ใช้สำหรับ: ตรวจสอบคุณภาพโค้ดแบบ Real-time
```

**4. Auto Import - ES6, TS, JSX, TSX**
```
ID: steoates.autoimport
ใช้สำหรับ: Auto Import Modules อัตโนมัติ
```

**5. Path Intellisense**
```
ID: christian-kohler.path-intellisense
ใช้สำหรับ: Auto-complete ชื่อไฟล์และ Path
```

**6. GitLens**
```
ID: eamodio.gitlens
ใช้สำหรับ: เพิ่มความสามารถ Git ใน VS Code
```

**7. Tailwind CSS IntelliSense**
```
ID: bradlc.vscode-tailwindcss
ใช้สำหรับ: Auto-complete Tailwind CSS Classes
```

**8. React Developer Tools (Browser Extension)**
```
Chrome: ค้นหา "React Developer Tools" ใน Chrome Web Store
Firefox: ค้นหาใน Firefox Add-ons
ใช้สำหรับ: Debug React Components, State, Props ใน Browser
```

### ตั้งค่า VS Code สำหรับ React

สร้างไฟล์ `.vscode/settings.json` ใน Project:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on",
  "editor.minimap.enabled": false,
  "emmet.includeLanguages": {
    "javascript": "javascriptreact",
    "typescript": "typescriptreact"
  },
  "javascript.updateImportsOnFileMove.enabled": "always",
  "typescript.updateImportsOnFileMove.enabled": "always",
  "files.exclude": {
    "**/node_modules": true,
    "**/.git": true,
    "**/dist": true
  }
}
```

สร้างไฟล์ `.prettierrc` ใน Root:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "jsxSingleQuote": false,
  "jsxBracketSameLine": false
}
```

---

## Step 16: รัน Development Server ครั้งแรก

### คำสั่งพื้นฐาน

```bash
# เข้าไปใน Project Directory
cd my-react-app

# ติดตั้ง Dependencies
npm install

# รัน Development Server
npm run dev
```

เมื่อรันสำเร็จจะเห็น:
```
  VITE v5.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

เปิด Browser ไปที่ `http://localhost:5173/` จะเห็น React App ครั้งแรก!

### คำสั่งอื่นๆ ที่สำคัญ

```bash
# Build สำหรับ Production
npm run build
# สร้างโฟลเดอร์ dist/ ที่มีไฟล์ที่ Optimize แล้ว

# Preview Production Build
npm run preview
# รัน Server เพื่อดู Production Build

# Lint Code
npm run lint
# ตรวจสอบ Code Quality ด้วย ESLint

# ติดตั้ง Package ใหม่
npm install axios
# ติดตั้ง axios เพื่อใช้ทำ HTTP Requests
```

---

## Step 17: ทำความรู้จัก Hot Module Replacement (HMR)

### HMR คืออะไร?

**HMR** คือฟีเจอร์ที่ทำให้ Browser อัปเดตเฉพาะส่วนที่เปลี่ยนแปลงโดย **ไม่ต้อง Reload** ทั้งหน้า

```
คุณแก้ไขโค้ด → Vite ตรวจพบ → อัปเดต Browser ทันที (< 50ms)
```

### ทดลอง HMR

1. เปิดไฟล์ `src/App.jsx`
2. เปลี่ยนข้อความ
3. บันทึกไฟล์
4. ดูว่า Browser อัปเดตเองทันที!

```jsx
// src/App.jsx - ลองเปลี่ยน text
function App() {
  return (
    <div>
      {/* ลองเปลี่ยนข้อความนี้และบันทึก */}
      <h1>สวัสดี React! 🎉</h1>
    </div>
  )
}

export default App
```

---

## Step 18: โครงสร้าง Project ที่ดี

### แนะนำโครงสร้าง Folder สำหรับ Project จริง

```
my-react-app/
├── 📁 public/
│   ├── favicon.ico
│   └── og-image.png
├── 📁 src/
│   ├── 📁 components/       ← Reusable Components
│   │   ├── 📁 ui/           ← Basic UI Components
│   │   │   ├── Button.jsx
│   │   │   ├── Input.jsx
│   │   │   └── Modal.jsx
│   │   └── 📁 layout/       ← Layout Components
│   │       ├── Navbar.jsx
│   │       ├── Sidebar.jsx
│   │       └── Footer.jsx
│   ├── 📁 pages/            ← Page Components
│   │   ├── Home.jsx
│   │   ├── About.jsx
│   │   └── Contact.jsx
│   ├── 📁 hooks/            ← Custom React Hooks
│   │   ├── useAuth.js
│   │   └── useFetch.js
│   ├── 📁 context/          ← React Context
│   │   └── AuthContext.jsx
│   ├── 📁 store/            ← State Management
│   │   └── authStore.js
│   ├── 📁 services/         ← API Services
│   │   ├── api.js
│   │   └── authService.js
│   ├── 📁 utils/            ← Helper Functions
│   │   ├── formatDate.js
│   │   └── validators.js
│   ├── 📁 types/            ← TypeScript Types (ถ้าใช้)
│   │   └── index.ts
│   ├── 📁 constants/        ← Constants
│   │   └── routes.js
│   ├── 📁 styles/           ← Global Styles
│   │   └── globals.css
│   ├── App.jsx
│   └── main.jsx
├── .env                     ← Environment Variables
├── .env.example             ← Example ENV
├── .eslintrc.cjs
├── .gitignore
├── .prettierrc
├── index.html
├── package.json
└── vite.config.js
```

---

## Step 19: ตั้งค่า Git

### เริ่มต้น Git Repository

```bash
# เริ่มต้น Git (ถ้ายังไม่มี)
git init

# ตั้งค่า User
git config --global user.name "ชื่อของคุณ"
git config --global user.email "email@example.com"

# เพิ่มไฟล์ทั้งหมด
git add .

# Commit ครั้งแรก
git commit -m "Initial commit: React project setup"
```

### ไฟล์ `.gitignore` ที่ดี

```gitignore
# Dependencies
node_modules/
.pnp
.pnp.js

# Build outputs
dist/
dist-ssr/
build/
*.tsbuildinfo

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

# Editor
.vscode/
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
.idea/

# OS
.DS_Store
Thumbs.db

# Testing
coverage/

# Cache
.eslintcache
.stylelintcache
*.lcov
.cache/
```

---

## Step 20: ทดสอบสร้าง Component แรก

```bash
# สร้างไฟล์ components/HelloWorld.jsx
```

```jsx
// src/components/HelloWorld.jsx
function HelloWorld({ name = "นักเรียน" }) {
  return (
    <div style={{
      padding: '20px',
      border: '2px solid #61dafb',
      borderRadius: '8px',
      textAlign: 'center',
      fontFamily: 'Arial, sans-serif'
    }}>
      <h1>🎉 สวัสดี, {name}!</h1>
      <p>ยินดีต้อนรับสู่โลกของ React</p>
      <p>นี่คือ Component แรกของคุณ</p>
    </div>
  );
}

export default HelloWorld;
```

```jsx
// src/App.jsx - อัปเดตให้ใช้ Component
import HelloWorld from './components/HelloWorld';
import './App.css';

function App() {
  return (
    <div style={{ 
      display: 'flex', 
      justifyContent: 'center', 
      alignItems: 'center',
      minHeight: '100vh'
    }}>
      <HelloWorld name="สมชาย" />
    </div>
  );
}

export default App;
```

---

## Step 21: ทำความรู้จัก Node Modules

### `node_modules/` คืออะไร?

```
node_modules/ คือโฟลเดอร์ที่เก็บ Dependencies ทั้งหมด
ขนาดอาจใหญ่ถึง 200MB+ สำหรับ Project ทั่วไป
ห้าม Commit ขึ้น Git!
```

```bash
# ดูขนาด node_modules
du -sh node_modules/
# อาจแสดง: 150M  node_modules/

# นับจำนวน Packages
ls node_modules/ | wc -l
# อาจแสดง: 500+

# เหตุผลที่ใหญ่มาก:
# react ต้องการ dependency A
# dependency A ต้องการ dependency B
# dependency B ต้องการ dependency C
# ... ต้นไม้ dependency อาจลึกมาก
```

### `package-lock.json` คืออะไร?

```json
// package-lock.json บันทึก exact version ของทุก dependency
{
  "name": "my-react-app",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "dependencies": {
        "react": "^18.2.0"
      }
    },
    "node_modules/react": {
      "version": "18.2.0",  // Exact version ที่ใช้
      "resolved": "https://registry.npmjs.org/react/-/react-18.2.0.tgz",
      "integrity": "sha512-..."
    }
  }
}
```

> 💡 **ควร Commit** `package-lock.json` เข้า Git เพื่อให้ทีมทุกคนได้ Dependency เวอร์ชันเดียวกัน

---

## Step 22: Environment Variables

### .env Files

```bash
# .env - สำหรับ Production
VITE_API_URL=https://api.production.com
VITE_APP_NAME=MyApp

# .env.development - สำหรับ Development
VITE_API_URL=http://localhost:3001
VITE_DEBUG=true

# .env.local - Local Override (ไม่ต้อง Commit)
VITE_API_KEY=your-secret-key
```

> ⚠️ **สำคัญ**: ใน Vite ต้องขึ้นต้นด้วย `VITE_` เท่านั้นถึงจะเข้าถึงจาก Code ได้

```jsx
// การใช้ Environment Variables ใน React
function App() {
  const apiUrl = import.meta.env.VITE_API_URL;
  const appName = import.meta.env.VITE_APP_NAME;
  
  console.log('API URL:', apiUrl);
  
  return <h1>{appName}</h1>;
}
```

---

## Step 23: ติดตั้ง Library พื้นฐาน

```bash
# Routing
npm install react-router-dom

# HTTP Requests
npm install axios

# Icons
npm install react-icons

# Date Handling
npm install date-fns

# หลังติดตั้งตรวจสอบที่ package.json
cat package.json
```

```jsx
// ทดสอบการ import
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import axios from 'axios';
import { FaReact } from 'react-icons/fa';
import { format } from 'date-fns';

console.log('Libraries loaded!');
```

---

## Step 24: Debug ด้วย Browser DevTools

### Chrome DevTools สำหรับ React

```
F12 หรือ Right Click → Inspect → เปิด DevTools

Tab ที่สำคัญ:
1. Console  → ดู console.log และ Errors
2. Elements → ดู DOM Structure
3. Network  → ดู API Requests
4. Sources  → Debug JavaScript
5. Components → (ต้องติดตั้ง React DevTools Extension)
6. Profiler → (ต้องติดตั้ง React DevTools Extension)
```

### React Developer Tools

หลังติดตั้ง Extension:
```
1. Components Tab
   → ดู Component Tree
   → เห็น Props และ State ของแต่ละ Component
   → สามารถแก้ไข State ได้ Real-time

2. Profiler Tab
   → วัด Performance ของ Render
   → ดูว่า Component ไหน Re-render บ่อยเกินไป
   → หา Performance Bottleneck
```

### ใช้ console.log สำหรับ Debug

```jsx
function DebugExample({ data }) {
  // ดู data ที่ได้รับ
  console.log('data:', data);
  
  // ดูรูปแบบสวยงาม
  console.table(data);
  
  // ดูเป็น Tree
  console.dir(data);
  
  // วัดเวลา
  console.time('render');
  // ... โค้ดที่ต้องการวัด
  console.timeEnd('render');
  
  return <div>{JSON.stringify(data, null, 2)}</div>;
}
```

---

## Step 25: สรุปและ Checklist

### สิ่งที่ต้องมีก่อนเริ่ม Part ถัดไป

```bash
# ตรวจสอบ Tools ทั้งหมด
node --version          # v18+ หรือ v20+
npm --version           # v8+
git --version           # ใดก็ได้

# ตรวจสอบ Project รันได้
cd my-react-app
npm run dev             # ต้องเห็น localhost:5173
```

### Checklist

- [ ] ติดตั้ง Node.js v18+ แล้ว
- [ ] สร้าง React Project ด้วย Vite แล้ว
- [ ] รัน `npm run dev` แล้วเห็นหน้าเว็บ
- [ ] ติดตั้ง VS Code Extensions แล้ว
- [ ] เข้าใจโครงสร้าง Project เบื้องต้น
- [ ] เข้าใจ `package.json` เบื้องต้น
- [ ] มี React Developer Tools ใน Browser แล้ว

### คำสั่งสรุป

```bash
# สร้าง Project ใหม่
npm create vite@latest app-name -- --template react

# เข้า Project
cd app-name

# ติดตั้ง Dependencies
npm install

# รัน Dev Server
npm run dev

# Build Production
npm run build

# Preview Production
npm run preview
```

### ตัวอย่าง Package ที่ใช้บ่อยใน React Projects

```bash
# Routing
npm install react-router-dom@6

# State Management
npm install @reduxjs/toolkit react-redux
npm install zustand

# UI Library
npm install @mui/material @emotion/react @emotion/styled
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion

# Forms
npm install react-hook-form
npm install zod @hookform/resolvers

# HTTP
npm install axios
npm install @tanstack/react-query

# Styling
npm install tailwindcss postcss autoprefixer

# Icons
npm install react-icons

# Date
npm install date-fns

# Testing
npm install --save-dev jest @testing-library/react @testing-library/jest-dom
npm install --save-dev @playwright/test
```

---

## ➡️ Part ถัดไป

**[Part 03: JSX - JavaScript XML พื้นฐาน →](./part-03-jsx-fundamentals.md)**

ใน Part 03 เราจะเรียนรู้:
- JSX คืออะไร และทำงานอย่างไร
- กฎของ JSX
- การใช้ JavaScript ใน JSX
- JSX vs HTML ต่างกันอย่างไร
- Fragments

---

*Part 02 จบแล้ว! 🎉 Step 11-25 ผ่านไปแล้ว*
