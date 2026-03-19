**Vite 기반 React 프로젝트에서 Tailwind CSS를 사용하려면 `tailwind.config.js` 파일이 “필수”입니다.**
이 파일은 Tailwind가 어떤 파일을 스캔할지, 어떤 확장 기능을 적용할지, 커스텀 색상·폰트 등을 어떻게 설정할지를 정의하는 핵심 설정 파일입니다.

아래에서 **왜 필요한지 → 기본 생성 파일 구조 → Vite 기준 설치 절차 → 예시 코드**까지 정리해드리겠습니다.

---

# ✅ 1. `tailwind.config.js`가 필요한 이유

Tailwind는 사용되지 않는 CSS를 제거하기 위해 **Content Scanning(트리 셰이킹)** 을 수행합니다.
즉, Tailwind가 어떤 JSX/HTML에서 어떤 Utility Class를 사용하는지 알아야 합니다.

이를 위해 Tailwind는 항상 아래 설정을 필요로 합니다:

```js
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [],
}
```

➡️ 이 파일이 없으면 Tailwind는 어떤 파일에서 클래스를 찾아야 하는지 알 수 없습니다.
➡️ 결과적으로 **CSS가 전혀 생성되지 않거나**, Tailwind 적용이 안 됩니다.

---

# ✅ 2. Vite + React에서 Tailwind 설치 공식 절차

아래는 Tailwind 공식 문서 기준 설치 과정입니다.

---

## 📌 ① Tailwind 패키지 설치

```
npm install -D tailwindcss postcss autoprefixer
```

---

## 📌 ② Tailwind 초기화

```
npx tailwindcss init -p
```

이 명령은 두 파일을 생성합니다:

✔ `tailwind.config.js`
✔ `postcss.config.js`

> 즉, Tailwind를 프로젝트에서 사용하려면 config 파일을 자동 생성하는 것이 **정석**입니다.

---

# 📌 ③ `tailwind.config.js` 구성

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

---

# 📌 ④ `src/index.css`에 Tailwind 지시어 추가

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

# 📌 ⑤ `main.jsx`에서 index.css 임포트

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

---

# 🎉 Tailwind 정상 적용 확인하기

`App.jsx`에서 테스트:

```jsx
export default function App() {
  return (
    <div className="text-3xl font-bold text-blue-600">
      Hello Tailwind!
    </div>
  )
}
```

---

# 🔍 결론: tailwind.config.js는 꼭 필요합니다

✔ Tailwind의 핵심 설정 파일
✔ Content scanning을 위한 필수 요소
✔ 테마 확장 및 플러그인 적용의 중심
✔ Vite + React에서는 `npx tailwindcss init -p` 실행 시 자동 생성됨


