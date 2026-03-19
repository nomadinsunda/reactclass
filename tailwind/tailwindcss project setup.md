**JavaScript(JS)** 기반의 React 프로젝트에서 **Tailwind CSS v4** 를 Vite와 함께 완벽하게 세팅하는 방법을 알려 드릴게요! 🚀

---

# ⚡️ Vite + React(JS) + Tailwind CSS v4 초스피드 세팅 가이드

최신 Tailwind v4는 **Rust 기반 엔진**을 사용하여 설정이 극도로 단순해졌습니다. JavaScript 프로젝트에서도 복잡한 설정 파일 없이 깔끔하게 시작할 수 있습니다. 

---

## 1️⃣ 프로젝트 생성 (Vite + React)
먼저 터미널을 열고 Vite를 이용해 새로운 React 프로젝트를 만듭니다.

```bash
# 1. 프로젝트 생성 (JavaScript 선택)
npm create vite@latest firsttailwindcssv4app

# 2. 프로젝트 폴더로 이동
cd firsttailwindcssv4app

# 3. 디폴트 패키지 설치
npm install
```

---

## 2️⃣ Tailwind CSS v4 및 플러그인 설치
v4부터는 `postcss`나 `autoprefixer`를 수동으로 설치할 필요가 없습니다. **Vite 전용 플러그인** 하나면 충분합니다.

```bash
# Tailwind v4 및 Vite 플러그인 설치
npm install tailwindcss @tailwindcss/vite
```
> **💡 여기서 잠깐!** `npx tailwindcss init` 같은 명령어는 이제 실행하지 않아도 됩니다. v4는 설정 파일(`tailwind.config.js`) 없이도 돌아가는 **Zero-config**를 지향하기 때문이죠.

---

## 3️⃣ Vite 설정 연결 (`vite.config.js`)
설치한 Tailwind 플러그인을 Vite가 인식하도록 설정 파일에 추가합니다.

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite' // 1. Tailwind 플러그인 불러오기

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(), // 2. 플러그인 목록에 추가
  ],
})
```

---

## 4️⃣ CSS 엔트리 파일 작성 (`src/index.css`)
Vite가 디폴트로 만들어준 `src/index.css` 파일의 내용을 **모두 지우고** 아래의 코드만 딱 입력하세요.

```css
/* src/index.css */
@import "tailwindcss";

/* 🎨 커스텀 테마 설정 (필요할 때만 추가) */
@theme {
  --color-brand: #3b82f6;
  --font-sans: "Pretendard", system-ui;
}
```
* **v4의 마법:** `@import "tailwindcss";` 한 줄이면 모든 기본 스타일과 유틸리티 클래스가 로드됩니다. 별도의 `@base`, `@utilities` 선언도 필요 없습니다.

---

## 5️⃣ 엔트리 포인트 확인 (`src/main.jsx`)
`src/main.jsx` 파일에서 방금 작성한 `index.css`가 제대로 임포트되어 있는지 확인합니다. (Vite 기본 템플릿에는 이미 포함되어 있을 거예요!)

```javascript
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css' // 👈 이 파일이 임포트되어 있어야 스타일이 적용됩니다!

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

---

## 6️⃣ src/App.css 내용 삭제
vite가 생성한 App.css 파일의 내용을 모두 삭제합니다

---

## 7️⃣ 클래스 사용 및 테스트 (`src/App.jsx`)
이제 모든 준비가 끝났습니다. `App.jsx`에서 Tailwind 클래스가 잘 작동하는지 테스트해 보세요.

```javascript
// src/App.jsx
function App() {
  return (
    <div className="flex min-h-screen items-center justify-center bg-zinc-900 text-white">
      <div className="text-center p-10 border border-zinc-700 rounded-2xl shadow-2xl bg-zinc-800">
        <h1 className="text-4xl font-black text-blue-400 mb-4">
          Tailwind CSS v4 + Vite 🚀
        </h1>
        <p className="text-zinc-400">
          JavaScript 기반 React 프로젝트 세팅 완료!
        </p>
        <button className="mt-6 px-6 py-2 bg-blue-600 hover:bg-blue-500 rounded-full transition-all">
          시작하기
        </button>
      </div>
    </div>
  )
}

export default App
```

---

## ✅ 마지막 체크리스트

* **파일 삭제:** 프로젝트 루트에 `tailwind.config.js`나 `postcss.config.js`가 없어도 정상입니다. (안심하세요! 😊)
* **자동 감지:** HTML, JS, JSX 등 어떤 파일이든 클래스를 쓰면 Tailwind가 알아서 찾아냅니다.
* **컴파일 속도:** Rust 기반 엔진 덕분에 저장하자마자 화면이 바뀌는 기적을 체험하실 수 있습니다.

---

**이렇게 세팅을 마친 후 `npm run dev`를 실행해 보세요!**
