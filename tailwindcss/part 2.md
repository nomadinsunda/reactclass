# PART 2. Tailwind CSS 설치와 개발환경 구성

## 1. 이번 PART의 목표

PART 1에서는 Tailwind CSS의 핵심 개념을 배웠습니다.

```text
CSS
 ↓
Tailwind Utility Class
 ↓
Utility 조합
 ↓
Variant
 ↓
React Component
 ↓
UI
```

이번 PART에서는 실제 React 프로젝트에 Tailwind CSS를 설치합니다.

최종적으로 다음 코드가 동작하는 환경을 만드는 것이 목표입니다.

```jsx
export default function App() {
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold text-blue-600">
        Hello Tailwind CSS
      </h1>
    </div>
  )
}
```

전체 과정은 다음과 같습니다.

```text
React + Vite 프로젝트
        ↓
Tailwind CSS 설치
        ↓
Vite Plugin 연결
        ↓
CSS에서 Tailwind import
        ↓
React에서 className 사용
        ↓
개발 서버 실행
        ↓
Browser
```

---

# 2. React + Vite 프로젝트 생성

먼저 Vite를 사용하여 React 프로젝트를 생성합니다.

```bash
npm create vite@latest tailwind-app
```

프로젝트 생성 과정에서 다음을 선택합니다.

```text
Framework
→ React

Variant
→ JavaScript
```

프로젝트 디렉터리로 이동합니다.

```bash
cd tailwind-app
```

dependency를 설치합니다.

```bash
npm install
```

개발 서버를 실행합니다.

```bash
npm run dev
```

브라우저에서 Vite + React 초기 화면이 나타나면 프로젝트 생성이 완료된 것입니다.

전체 구조는 대략 다음과 같습니다.

```text
tailwind-app/
│
├─ node_modules/
├─ public/
├─ src/
│  ├─ assets/
│  ├─ App.jsx
│  ├─ index.css
│  └─ main.jsx
│
├─ index.html
├─ package.json
└─ vite.config.js
```

---

# 3. Tailwind CSS 설치

이제 Tailwind CSS를 설치합니다.

Vite 프로젝트에서는 다음 패키지를 사용합니다.

```bash
npm install tailwindcss @tailwindcss/vite
```

두 패키지의 역할을 구분해서 이해해야 합니다.

```text
tailwindcss
     │
     └─ Tailwind CSS 본체


@tailwindcss/vite
     │
     └─ Tailwind를 Vite와 연결하는 Plugin
```

즉:

```text
React
   │
   ▼
Vite
   │
   ▼
@tailwindcss/vite
   │
   ▼
Tailwind CSS
```

라는 관계입니다.

---

# 4. Vite Plugin 설정

`vite.config.js`를 수정합니다.

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

여기서 중요한 부분은:

```js
import tailwindcss from '@tailwindcss/vite'
```

와:

```js
tailwindcss()
```

입니다.

기존에는:

```js
plugins: [
  react(),
]
```

이었다면:

```js
plugins: [
  react(),
  tailwindcss(),
]
```

가 됩니다.

즉 Vite의 build/dev 처리 과정에 Tailwind CSS가 연결됩니다.

```text
Vite
 │
 ├─ React Plugin
 │
 └─ Tailwind CSS Plugin
```

---

# 5. CSS에서 Tailwind CSS 불러오기

`src/index.css`를 엽니다.

기존 Vite의 CSS 내용을 제거하고 다음과 같이 작성합니다.

```css
@import "tailwindcss";
```

끝입니다.

이 한 줄이 중요합니다.

```text
src/index.css

@import "tailwindcss";
          │
          ▼
   Tailwind CSS 연결
```

Tailwind CSS v4에서는 기본적인 시작 구성이 매우 간단합니다.

---

# 6. `index.css`는 어디에서 사용되는가?

여기서 학생들이 자주 가지는 의문이 있습니다.

> `index.css`를 작성했는데 React가 어떻게 이 파일을 사용하는가?

`src/main.jsx`를 확인합니다.

보통 다음 코드가 있습니다.

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'

import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

여기서:

```js
import './index.css'
```

가 중요합니다.

연결 관계는:

```text
main.jsx
   │
   │ import './index.css'
   ▼
index.css
   │
   │ @import "tailwindcss"
   ▼
Tailwind CSS
```

입니다.

따라서 전체적으로:

```text
main.jsx
   │
   ├──── App.jsx
   │
   └──── index.css
             │
             ▼
       Tailwind CSS
```

구조가 됩니다.

---

# 7. 첫 번째 Tailwind Utility 사용

이제 `App.jsx`를 수정합니다.

```jsx
export default function App() {
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold text-blue-600">
        Hello Tailwind CSS
      </h1>
    </div>
  )
}
```

여기에는 네 개의 Utility가 사용되었습니다.

```text
p-8
│
└─ padding


text-3xl
│
└─ font-size


font-bold
│
└─ font-weight


text-blue-600
│
└─ text color
```

PART 1에서 배운 원리가 그대로 사용되고 있습니다.

```text
Utility
 +
Utility
 +
Utility
 +
Utility
 ↓
UI 스타일
```

---

# 8. React에서는 `className`

HTML에서는 다음과 같이 작성합니다.

```html
<h1 class="text-3xl font-bold">
  Hello
</h1>
```

React JSX에서는:

```jsx
<h1 className="text-3xl font-bold">
  Hello
</h1>
```

이라고 작성합니다.

차이는:

```text
HTML              React JSX

class             className
```

입니다.

Tailwind의 차이가 아닙니다.

React JSX의 속성 이름이 `className`이기 때문입니다.

따라서 앞으로 React + Tailwind에서는 매우 자주 다음 형태를 사용하게 됩니다.

```jsx
<div className="...">
```

```jsx
<button className="...">
```

```jsx
<span className="...">
```

---

# 9. 버튼 만들어보기

조금 더 실제적인 UI를 만들어 보겠습니다.

```jsx
export default function App() {
  return (
    <div className="p-8">
      <button
        className="
          px-4
          py-2
          bg-blue-500
          text-white
          font-semibold
          rounded-lg
          hover:bg-blue-600
        "
      >
        저장
      </button>
    </div>
  )
}
```

각 클래스의 역할은 다음과 같습니다.

```text
px-4
 └─ 좌우 padding

py-2
 └─ 상하 padding

bg-blue-500
 └─ 배경 색상

text-white
 └─ 글자 색상

font-semibold
 └─ 글자 두께

rounded-lg
 └─ 모서리 둥글게

hover:bg-blue-600
 └─ hover 상태에서 배경 색상 변경
```

결과적으로:

```text
px-4
+
py-2
+
bg-blue-500
+
text-white
+
font-semibold
+
rounded-lg
+
hover:bg-blue-600
        │
        ▼
   ┌───────────┐
   │    저장    │
   └───────────┘
```

하나의 버튼 UI가 완성됩니다.

---

# 10. Tailwind 클래스는 어떻게 CSS가 되는가?

다음 JSX가 있다고 하겠습니다.

```jsx
<button className="px-4 py-2 bg-blue-500 text-white">
  저장
</button>
```

우리가 직접 다음 CSS를 작성한 것은 아닙니다.

```css
.button {
  ...
}
```

대신 Tailwind가 프로젝트에서 사용된 Utility를 기반으로 필요한 CSS를 처리합니다.

개념적인 흐름은 다음과 같습니다.

```text
App.jsx

className="
  px-4
  py-2
  bg-blue-500
  text-white
"
        │
        ▼
Tailwind CSS
        │
        │ 사용된 Utility 처리
        ▼
필요한 CSS
        │
        ▼
Vite
        │
        ▼
Browser
```

즉 개발자는:

```text
어떤 CSS가 필요한가?
       ↓
그 CSS에 대응하는
Tailwind Utility는 무엇인가?
       ↓
className에 작성
```

하는 데 집중합니다.

---

# 11. 개발 중 클래스 변경

Vite 개발 서버를 실행해 둡니다.

```bash
npm run dev
```

예를 들어:

```jsx
className="bg-blue-500"
```

를:

```jsx
className="bg-red-500"
```

로 변경합니다.

또는:

```jsx
className="rounded-lg"
```

를:

```jsx
className="rounded-full"
```

로 변경합니다.

Vite의 개발환경에서 변경 결과를 빠르게 확인할 수 있습니다.

따라서 개발 과정은 매우 단순합니다.

```text
JSX 수정
   ↓
Utility 변경
   ↓
저장
   ↓
Browser에서 결과 확인
   ↓
다시 수정
```

---

# 12. Tailwind CSS v4에서 달라진 시작 방식

인터넷에서 Tailwind 설치 방법을 검색하면 다음과 같은 코드를 볼 수 있습니다.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

또는:

```js
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
}
```

이런 방식은 **이전 Tailwind 버전의 강의나 문서에서 많이 사용하던 구성**입니다.

현재 Tailwind CSS v4 + Vite의 기본적인 시작 방식에서는:

```css
@import "tailwindcss";
```

와 Vite Plugin을 사용하여 훨씬 간단하게 구성할 수 있습니다.

따라서 이번 강의에서는 다음 구성을 기준으로 합니다.

```text
Tailwind CSS v4
       +
Vite
       +
React
```

---

# 13. `tailwind.config.js`가 반드시 필요한가?

기본적인 Tailwind CSS 사용을 시작하기 위해 반드시 필요한 것은 아닙니다.

PART 1에서 사용했던:

```text
p-4
flex
grid
bg-blue-500
text-white
rounded-lg
hover:bg-blue-600
md:grid-cols-2
```

등을 사용하기 위해 처음부터 `tailwind.config.js`를 만들 필요는 없습니다.

기본 프로젝트는 매우 단순하게 시작할 수 있습니다.

```text
vite.config.js
      │
      └─ tailwindcss()

src/index.css
      │
      └─ @import "tailwindcss";

React Component
      │
      └─ className="..."
```

필요한 customization은 이후 PART에서 별도로 다룹니다.

---

# 14. 프로젝트 전체 연결 구조

지금까지의 내용을 하나로 연결해 보겠습니다.

```text
                React Project
                     │
                     ▼
                  main.jsx
                 /        \
                /          \
               ▼            ▼
           App.jsx       index.css
              │             │
              │             │
      className="..."       │
              │             │
              │       @import "tailwindcss"
              │             │
              └──────┬──────┘
                     ▼
               Tailwind CSS
                     │
                     ▼
                필요한 CSS
                     │
                     ▼
                    Vite
                     │
                     ▼
                  Browser
```

여기에서 역할을 구분하는 것이 중요합니다.

```text
React
→ UI Component 작성

className
→ Utility Class 지정

Tailwind CSS
→ Utility를 CSS로 처리

Vite
→ 개발 및 build 환경 제공

Browser
→ 최종 CSS를 적용하여 UI 렌더링
```

---

# 15. 자주 발생하는 실수

## `class`를 사용하는 경우

잘못된 예:

```jsx
<div class="p-4">
```

React에서는:

```jsx
<div className="p-4">
```

를 사용합니다.

---

## `index.css`에서 Tailwind를 import하지 않은 경우

다음 코드가 필요합니다.

```css
@import "tailwindcss";
```

---

## Vite Plugin을 등록하지 않은 경우

다음을 설치했다면:

```bash
npm install tailwindcss @tailwindcss/vite
```

`vite.config.js`에서도 Plugin을 등록해야 합니다.

```js
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

---

## Utility 이름을 임의로 만드는 경우

예를 들어 다음처럼 마음대로 작성할 수 있는 것은 아닙니다.

```text
padding-big
blue-background
big-font
```

Tailwind가 제공하는 Utility 문법을 사용해야 합니다.

```text
p-8
bg-blue-500
text-3xl
```

---

# 16. 설치 완료 체크

다음 항목이 모두 동작하면 Tailwind CSS 개발환경 구성이 완료된 것입니다.

```text
React + Vite 프로젝트
        ✓

tailwindcss 설치
        ✓

@tailwindcss/vite 설치
        ✓

Vite Plugin 등록
        ✓

@import "tailwindcss"
        ✓

className에 Utility 작성
        ✓

Browser에서 스타일 확인
        ✓
```

---

# 17. PART 2 핵심 정리

이번 PART에서 가장 중요한 흐름은 다음과 같습니다.

```text
① React + Vite
       ↓
② Tailwind CSS 설치
       ↓
③ @tailwindcss/vite 연결
       ↓
④ @import "tailwindcss"
       ↓
⑤ className에 Utility 작성
       ↓
⑥ Tailwind가 필요한 CSS 처리
       ↓
⑦ Browser에서 UI 렌더링
```

그리고 설정의 핵심만 다시 보면:

```bash
npm install tailwindcss @tailwindcss/vite
```

```js
// vite.config.js

import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

```css
/* src/index.css */

@import "tailwindcss";
```

```jsx
export default function App() {
  return (
    <button
      className="
        px-4 py-2
        bg-blue-500 text-white
        rounded-lg
        hover:bg-blue-600
      "
    >
      저장
    </button>
  )
}
```

이 네 부분이 연결되면 **React에서 Tailwind CSS를 사용할 준비가 끝난 것입니다.**

---

# 다음 PART

## PART 3. 기본 Utility Classes

다음 PART부터 본격적으로 Tailwind Utility를 사용합니다.

```text
Spacing
 ├─ p-
 ├─ m-
 └─ gap-

Size
 ├─ w-
 ├─ h-
 ├─ min-
 └─ max-

Color
 ├─ bg-
 ├─ text-
 └─ border-

Border
 ├─ border
 └─ rounded-

Layout
 ├─ block
 ├─ inline
 ├─ flex
 └─ grid
```

핵심은 Utility를 무작정 암기하는 것이 아니라 **기존 CSS Property와 Tailwind Utility를 연결해서 이해하는 것**입니다.
