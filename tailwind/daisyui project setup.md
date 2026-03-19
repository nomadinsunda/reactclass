# 🌼 Vite + React + Tailwind v4 + daisyUI: 최신 스택 구축 가이드

이제 JavaScript 설정 파일(`tailwind.config.js`) 없이, 오직 **CSS와 Vite 플러그인**만으로 강력한 UI 컴포넌트 라이브러리인 daisyUI를 세팅해 보겠습니다.

---

## 1️⃣ 프로젝트 생성 및 폴더 이동
가장 먼저 Vite를 사용하여 새로운 React 프로젝트를 생성합니다.

```bash
# 1. Vite 프로젝트 생성 (React + JavaScript)
npm create vite@latest firstdaisyapp

# 2. 폴더 이동 및 기본 패키지 설치
cd firstdaisyapp
npm install
```

---

## 2️⃣ 필수 패키지 설치 (Tailwind v4 & daisyUI)
v4 엔진과 Vite 전용 플러그인, 그리고 daisyUI를 한꺼번에 설치합니다.

```bash
# Tailwind v4, Vite 플러그인, daisyUI 설치
npm install tailwindcss @tailwindcss/vite daisyui@latest
```
> **💡 포인트:** v4는 `postcss`나 `autoprefixer`를 별도로 관리할 필요가 없습니다. `@tailwindcss/vite`가 이 모든 최적화를 Rust 엔진으로 대신 수행합니다.

---

## 3️⃣ Vite 설정 파일 업데이트 (`vite.config.js`)
Vite 빌드 파이프라인에 Tailwind 엔진을 심어줍니다. 이 작업만으로 모든 준비가 끝납니다.

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite' // 1. 임포트

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(), // 2. 플러그인 등록
  ],
})
```

---

## 4️⃣ CSS 엔트리 파일 작성 (`src/index.css`)
기존 내용을 모두 지우고, Tailwind와 daisyUI를 **CSS 표준 문법**으로 불러옵니다. v4에서 가장 크게 바뀐 부분입니다.

```css
/* src/index.css */
@import "tailwindcss";
@plugin "daisyui"; /* 👈 반드시 tailwindcss 다음에 위치해야 합니다. */

/* (선택 사항) 나만의 브랜드 컬러 설정 */
@theme {
  --color-primary: #570df8;
  --color-secondary: #f000b8;
}
```



---

## 5️⃣ HTML 테마 설정 (`index.html`)
daisyUI의 다양한 테마(dark, light, retro 등)를 사용하려면 `<html>` 태그에 속성을 추가합니다.

```html
<!doctype html>
<html lang="ko" data-theme="dark"> <head>... </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

---

## 6️⃣ 컴포넌트에서 확인하기 (`src/App.jsx`)
이제 daisyUI의 클래스들이 잘 작동하는지 확인해 볼 차례입니다.

```javascript
// src/App.jsx
function App() {
  return (
    <div className="min-h-screen bg-base-200 flex flex-col items-center justify-center gap-6 p-4">
      <h1 className="text-3xl font-black text-primary">Hello, daisyUI v4!</h1>
      
      {/* daisyUI 버튼 그룹 */}
      <div className="flex gap-2">
        <button className="btn btn-primary shadow-lg">주요 버튼</button>
        <button className="btn btn-secondary">보조 버튼</button>
        <button className="btn btn-outline btn-accent">라인 버튼</button>
      </div>

      {/* daisyUI 배지 및 통계 컴포넌트 */}
      <div className="stats shadow bg-base-100">
        <div className="stat">
          <div className="stat-title">방문자 수</div>
          <div className="stat-value text-primary">25.6K</div>
          <div className="stat-desc">지난 달 대비 21% 증가</div>
        </div>
      </div>
    </div>
  )
}

export default App
```

---

## 7️⃣ 프로젝트 실행
모든 설정이 완료되었습니다! 터미널에서 개발 서버를 구동하세요.

```bash
npm run dev
```

---

## 🏁 요약 및 장점

* **Zero-Config:** 더 이상 루트 폴더에 복잡한 `.js` 설정 파일들이 굴러다니지 않습니다.
* **초고속 빌드:** Rust 기반의 **Oxide 엔진**이 daisyUI의 방대한 스타일을 순식간에 빌드합니다.
* **유지보수 용이:** 모든 테마와 플러그인 설정이 `src/index.css` 한곳에 모여 있어 관리가 매우 직관적입니다.

