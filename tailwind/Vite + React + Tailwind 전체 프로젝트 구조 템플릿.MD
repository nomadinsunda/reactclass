## 1. 목표 🎯

이 글의 목표는:

> **“Vite + React + Tailwind CSS를 쓸 때, 팀/수업에서 재사용 가능한 기본 템플릿을 한 번에 잡아두자”**

입니다.

그래서 아래를 모두 다룹니다.

1. 프로젝트 생성 & 기본 세팅
2. 디렉터리 구조 설계
3. Tailwind 연동 (tailwind.config, postcss.config)
4. ESLint/Prettier 설정 (Vite 기본 + 약간의 튜닝)
5. `src` 구조 (components, pages, layouts, hooks 등)
6. 샘플 컴포넌트들 (Tailwind 맛보기)

---

## 2. Vite + React 프로젝트 생성 ⚙️

먼저 Vite 기반 React 프로젝트를 생성합니다.

```bash
# PNPM 예시 (npm/yarn 사용하셔도 됩니다)
pnpm create vite my-react-tailwind-app --template react

cd my-react-tailwind-app

# 의존성 설치
pnpm install
```

> 이미 Vite 기본 템플릿에는 `eslint.config.js`가 들어 있을 수 있습니다.
> 이 글에서는 **Vite 기본 eslint.config.js를 유지**하면서, Tailwind + Prettier만 추가하는 방향으로 갑니다.

---

## 3. Tailwind CSS 설치 및 연동 🌬️

Tailwind 공식 문서 기준(React + Vite)으로 세팅합니다.

### 3-1. Tailwind & PostCSS 패키지 설치

```bash
pnpm install -D tailwindcss postcss autoprefixer
```

초기화:

```bash
npx tailwindcss init -p
```

위 명령으로 두 파일이 생성됩니다.

* `tailwind.config.js`
* `postcss.config.js`

---

## 4. 전체 디렉터리 구조 설계 🗂️

강의나 팀 프로젝트에서 재사용하기 좋은, 살짝 구조화된 템플릿 예시입니다.

```bash
my-react-tailwind-app/
├─ .vscode/
│  └─ settings.json          # (선택) 에디터 Tailwind/Prettier 설정
├─ node_modules/
├─ public/
│  └─ favicon.ico
├─ src/
│  ├─ assets/
│  │  └─ react.svg
│  ├─ components/
│  │  ├─ layout/
│  │  │  ├─ AppLayout.jsx   # 공통 레이아웃
│  │  │  └─ Navbar.jsx
│  │  ├─ ui/
│  │  │  ├─ Button.jsx      # 공통 버튼
│  │  │  └─ Card.jsx
│  │  └─ common/
│  │     └─ ThemeToggle.jsx # 다크모드 토글 예시
│  ├─ pages/
│  │  ├─ Home.jsx
│  │  └─ About.jsx
│  ├─ hooks/
│  │  └─ useTheme.js        # 다크 모드 관리 예시
│  ├─ styles/
│  │  └─ index.css          # Tailwind 엔트리 + 커스텀 스타일
│  ├─ App.jsx
│  ├─ main.jsx
│  └─ vite-env.d.ts (ts일 때)
├─ .eslintrc.* / eslint.config.js  # (Vite 생성; 여기선 eslint.config.js 가정)
├─ .prettierrc.json                # Prettier 설정
├─ index.html
├─ package.json
├─ postcss.config.js
├─ tailwind.config.js
├─ vite.config.js
└─ README.md
```

이 구조의 의도:

* **components/layout**: 페이지 공통 레이아웃/헤더/푸터 등.
* **components/ui**: 디자인 시스템 수준의 공통 UI 컴포넌트(Button, Card 등).
* **components/common**: 여기저기 섞여 쓰이는 범용 컴포넌트.
* **pages**: 라우팅 또는 페이지 단위 화면.
* **hooks**: 재사용 가능한 커스텀 훅 (예: `useTheme`, `useFetch` 등).
* **styles/index.css**: Tailwind 지시자(`@tailwind base;` 등) + 공통 CSS.

---

## 5. Tailwind 설정 파일들 🔧

### 5-1. `tailwind.config.js`

Vite + React 전용 content 경로를 꼭 잡아야 합니다.

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{js,jsx,ts,tsx}', // src 내부 React 파일 전체 스캔
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: '#2563eb',
          light: '#60a5fa',
          dark: '#1d4ed8',
        },
      },
    },
  },
  plugins: [],
}
```

여기서 `theme.extend.colors.brand`는 나중에 `bg-brand`, `text-brand-dark` 같이 쓸 수 있는 **커스텀 색상**입니다.
강의할 때 “Tailwind의 theme 확장” 예제로 소개하기 좋습니다.

---

### 5-2. `postcss.config.js`

`npx tailwindcss init -p`로 만들어진 기본 형태를 그대로 사용하면 됩니다.

```js
// postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

Vite는 개발 서버/빌드 단계에서 이 설정을 사용해서 Tailwind를 처리합니다.

---

### 5-3. `src/styles/index.css`

Tailwind의 3대 지시자를 추가해야 합니다.

```css
/* src/styles/index.css */

/* Tailwind core layers */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 아래는 프로젝트 공통 스타일(선택) */
body {
  @apply bg-slate-50 text-slate-900;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

a {
  @apply text-brand hover:underline;
}
```

그리고 이 파일을 `main.jsx`에서 import해야 합니다.

---

## 6. Vite + React 엔트리 파일들 🧩

### 6-1. `main.jsx`

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './styles/index.css' // Tailwind 엔트리

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### 6-2. `App.jsx`

```jsx
// src/App.jsx
import AppLayout from './components/layout/AppLayout'
import Home from './pages/Home'

function App() {
  return (
    <AppLayout>
      <Home />
    </AppLayout>
  )
}

export default App
```

> 강의에서 “**레이아웃 컴포넌트 + 페이지 컴포넌트** 구조”를 강조할 수 있습니다.

---

## 7. 레이아웃 & UI 컴포넌트 구조 🧱

### 7-1. `AppLayout.jsx`

```jsx
// src/components/layout/AppLayout.jsx
import Navbar from './Navbar'

function AppLayout({ children }) {
  return (
    <div className="min-h-screen flex flex-col bg-slate-100">
      <Navbar />

      <main className="flex-1 container mx-auto px-4 py-6">
        {children}
      </main>

      <footer className="border-t bg-white py-4">
        <div className="container mx-auto px-4 text-sm text-slate-500">
          © {new Date().getFullYear()} My React Tailwind App
        </div>
      </footer>
    </div>
  )
}

export default AppLayout
```

Tailwind 포인트:

* `min-h-screen`: 화면 전체 높이.
* `flex flex-col`: 상하 구조.
* `container mx-auto px-4`: 반응형 컨테이너.

---

### 7-2. `Navbar.jsx`

```jsx
// src/components/layout/Navbar.jsx
import ThemeToggle from '../common/ThemeToggle'

function Navbar() {
  return (
    <header className="border-b bg-white/80 backdrop-blur sticky top-0 z-10">
      <nav className="container mx-auto px-4 py-3 flex items-center justify-between">
        <div className="flex items-center gap-2">
          <span className="inline-flex h-8 w-8 items-center justify-center rounded-lg bg-brand text-white font-bold">
            VT
          </span>
          <div>
            <div className="font-semibold text-slate-800">
              Vite + React + Tailwind
            </div>
            <div className="text-xs text-slate-500">
              Starter Template
            </div>
          </div>
        </div>

        <div className="flex items-center gap-4">
          <a href="#" className="text-sm text-slate-600 hover:text-slate-900">
            Home
          </a>
          <a href="#" className="text-sm text-slate-600 hover:text-slate-900">
            About
          </a>

          <ThemeToggle />
        </div>
      </nav>
    </header>
  )
}

export default Navbar
```

---

### 7-3. `Button.jsx` (UI 컴포넌트 예시)

```jsx
// src/components/ui/Button.jsx
const base =
  'inline-flex items-center justify-center rounded-md border text-sm font-medium transition focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed'

const variants = {
  solid: 'bg-brand text-white border-transparent hover:bg-brand-dark focus:ring-brand',
  outline:
    'bg-white text-brand border-brand hover:bg-brand/5 focus:ring-brand',
  ghost:
    'bg-transparent text-slate-700 border-transparent hover:bg-slate-100 focus:ring-slate-300',
}

function Button({ variant = 'solid', className = '', ...props }) {
  const variantClass = variants[variant] ?? variants.solid
  return (
    <button className={`${base} ${variantClass} ${className}`} {...props} />
  )
}

export default Button
```

> Tailwind로 간단한 “디자인 시스템 버튼”을 만드는 예시입니다.
> 나중에 학생들에게 variant 개념(Primary / Secondary / Ghost)을 설명하기에 좋습니다.

---

### 7-4. `Card.jsx`

```jsx
// src/components/ui/Card.jsx
function Card({ title, description, children }) {
  return (
    <div className="rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
      {title && (
        <h2 className="text-lg font-semibold text-slate-800 mb-1">
          {title}
        </h2>
      )}
      {description && (
        <p className="text-sm text-slate-500 mb-3">{description}</p>
      )}
      {children}
    </div>
  )
}

export default Card
```

---

## 8. 페이지 컴포넌트 예시 📄

### 8-1. `Home.jsx`

```jsx
// src/pages/Home.jsx
import Button from '../components/ui/Button'
import Card from '../components/ui/Card'

function Home() {
  return (
    <div className="space-y-6">
      <section className="text-center space-y-4">
        <p className="text-sm font-semibold text-brand uppercase tracking-wide">
          Vite + React + Tailwind
        </p>
        <h1 className="text-3xl md:text-4xl font-bold text-slate-900">
          Ready-to-use Starter Template
        </h1>
        <p className="text-sm md:text-base text-slate-600 max-w-xl mx-auto">
          This project gives you a clean structure with Vite, React, Tailwind
          CSS, ESLint, and Prettier so you can focus on building features, not
          configuring tools.
        </p>

        <div className="flex justify-center gap-3 pt-2">
          <Button>Get Started</Button>
          <Button variant="outline">View Docs</Button>
        </div>
      </section>

      <section className="grid md:grid-cols-3 gap-4">
        <Card
          title="Fast Dev Server"
          description="Vite provides lightning fast dev server and HMR."
        >
          <p className="text-sm text-slate-600">
            Save your file and see changes instantly. Perfect for React development.
          </p>
        </Card>

        <Card
          title="Utility-First Styling"
          description="Tailwind CSS keeps your styles close to your markup."
        >
          <p className="text-sm text-slate-600">
            Compose styles with small utility classes instead of jumping between
            many CSS files.
          </p>
        </Card>

        <Card
          title="Lint & Format"
          description="ESLint + Prettier help keep your code clean and consistent."
        >
          <p className="text-sm text-slate-600">
            Enforce best practices and automatic formatting across the whole team.
          </p>
        </Card>
      </section>
    </div>
  )
}

export default Home
```

---

## 9. 다크 모드 토글 + Hook 예시 🌙

### 9-1. `useTheme.js`

**Tailwind JIT + class 전략**으로 `dark` 클래스를 `<html>`에 붙이는 패턴입니다.

```js
// src/hooks/useTheme.js
import { useEffect, useState } from 'react'

const STORAGE_KEY = 'theme'

export function useTheme() {
  const [theme, setTheme] = useState(() => {
    if (typeof window === 'undefined') return 'light'
    return localStorage.getItem(STORAGE_KEY) || 'light'
  })

  useEffect(() => {
    const root = window.document.documentElement

    if (theme === 'dark') {
      root.classList.add('dark')
    } else {
      root.classList.remove('dark')
    }

    localStorage.setItem(STORAGE_KEY, theme)
  }, [theme])

  const toggleTheme = () =>
    setTheme((prev) => (prev === 'dark' ? 'light' : 'light' ? 'dark' : 'light'))

  return { theme, setTheme, toggleTheme }
}
```

*(※ 위의 삼항은 보기 좋게 `prev === 'dark' ? 'light' : 'dark'` 로 정리하셔도 됩니다. 일부러 학생들에게 “이상한 코드”로 보여주실 게 아니라면요.)*

---

### 9-2. `ThemeToggle.jsx`

```jsx
// src/components/common/ThemeToggle.jsx
import { useTheme } from '../../hooks/useTheme'

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme()

  return (
    <button
      type="button"
      onClick={toggleTheme}
      className="inline-flex items-center gap-1 rounded-full border border-slate-300 px-3 py-1 text-xs font-medium text-slate-700 hover:bg-slate-100"
    >
      <span
        className={`h-2 w-2 rounded-full ${
          theme === 'dark' ? 'bg-amber-400' : 'bg-sky-500'
        }`}
      />
      <span>{theme === 'dark' ? 'Dark' : 'Light'} mode</span>
    </button>
  )
}

export default ThemeToggle
```

다크 모드 사용을 위해서는 `tailwind.config.js`에 `darkMode: 'class'` 설정을 넣을 수 있습니다.

```js
// tailwind.config.js 일부
export default {
  darkMode: 'class',
  content: [/* ... */],
  theme: { /* ... */ },
  plugins: [],
}
```

---

## 10. ESLint & Prettier 설정 🧹

### 10-1. `eslint.config.js` (Vite 기본 + 약간의 튜닝)

질문에서 주셨던 Vite 기본 flat config 예시를 그대로 사용해도 됩니다.
여기서는 그것을 조금만 확장해 보겠습니다.

```js
// eslint.config.js
import js from '@eslint/js'
import globals from 'globals'
import reactHooks from 'eslint-plugin-react-hooks'
import reactRefresh from 'eslint-plugin-react-refresh'
import { defineConfig, globalIgnores } from 'eslint/config'

export default defineConfig([
  globalIgnores(['dist']),
  {
    files: ['**/*.{js,jsx}'],
    extends: [
      js.configs.recommended,
      reactHooks.configs.flat.recommended,
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
      parserOptions: {
        ecmaVersion: 'latest',
        ecmaFeatures: { jsx: true },
        sourceType: 'module',
      },
    },
    rules: {
      'no-unused-vars': ['error', { varsIgnorePattern: '^[A-Z_]' }],
      'no-console': 'off',      // 강의용이면 console 허용
    },
  },
])
```

> `.eslintrc.cjs`는 **옛날 형식**(RC 스타일)이고, `eslint.config.js`는 **flat config** 형식입니다.
> Vite가 flat config를 만들어 줬다면 **그것만 사용**하시는 게 맞습니다.
> 두 개를 동시에 쓰면 헷갈리니 `.eslintrc.*`는 굳이 추가하지 않는 게 좋습니다.

---

### 10-2. `.prettierrc.json`

질문에서 “`prettierrc.json` vs `.prettierrc.json`”을 물어보셨는데, **일반적으로는 앞에 점(`.`)을 붙이는** 것이 관례입니다.

* `.prettierrc`, `.prettierrc.json`, `.eslintrc`, `.gitignore` 처럼 **설정 파일/숨김 파일**을 나타낼 때 Unix 관례로 점을 붙입니다.
* Prettier는 다양한 파일명을 지원하지만, 점 있는 이름이 “표준적인 느낌” 이라 대부분 `.prettierrc` 또는 `.prettierrc.json`을 사용합니다.

예시:

```json
// .prettierrc.json
{
  "singleQuote": true,
  "semi": false,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "avoid"
}
```

> Vite + React + Tailwind 템플릿에서는 **팀/수업 기준 스타일**을 한 번 정해두고 `.prettierrc.json`으로 고정해 두면 좋습니다.

---

## 11. Vite 설정 (Tailwind는 자동 연동) ⚡

`vite.config.js`는 기본 생성된 형태면 충분합니다. Tailwind 전용 설정은 필요 없습니다.

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'

export default defineConfig({
  plugins: [react()],
})
```

Tailwind는 `postcss.config.js` + `tailwind.config.js` + `index.css` + `import './styles/index.css'` 흐름으로만 연동됩니다.
Vite는 이를 PostCSS 단계에서 처리해 주므로 별도의 plugin 설정이 필요 없습니다.

---

## 12. 마무리: 이 템플릿으로 무엇을 할 수 있나? ✅

이 템플릿만 있으면:

* **강의용**

  * “Vite dev 서버 구조”, “React 컴포넌트 구조”, “Tailwind 유틸리티 클래스”를 한 번에 보여줄 수 있습니다.
  * 레이아웃/페이지/UI 컴포넌트/커스텀 훅 구조를 학생들과 계속 재사용 가능.
* **실전 프로젝트용**

  * 바로 Router, Zustand/Redux, API 클라이언트(Axios/React Query) 등을 추가해서 확장할 수 있습니다.
* **팀 템플릿용**

  * ESLint + Prettier + Tailwind + 구조 통일 → 코드 스타일과 폴더 구조를 팀 표준으로 만들기 좋습니다.

