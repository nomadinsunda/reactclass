# 🌼 DaisyUI

Tailwind CSS를 진짜 UI 프레임워크로 바꿔주는 마법

---

## 1. 🧐 DaisyUI란 무엇인가?

DaisyUI는 Tailwind CSS의 **플러그인(plugin)** 으로, Tailwind의 유틸리티 클래스 위에 **고급 UI 컴포넌트 세트**를 직접 제공해주는 UI 라이브러리다.

즉…

### ❌ Tailwind만으로 UI 구성할 때:

* 매 버튼, 카드, 모달을 **순수 유틸리티 클래스**로 일일이 조합해야 한다
* 미세한 스타일링이 강점이지만, **반복되는 UI 작업**이 너무 많다
* 디자인 일관성을 유지하기 어렵다

### ✅ DaisyUI를 사용하면:

* **Button**, **Card**, **Modal**, **Navbar**, **Tabs**, **Badge**, **Alert**, **Dropdown**, **Menu** 등
  ▶ 이미 디자인된 UI 컴포넌트를 **className 한 줄**로 바로 사용
* Tailwind의 장점을 그대로 유지
* 테마(theme)를 자동으로 관리
* CSS 파일 추가 X
* 반응형 포함
* 다크모드 테마 자동 구성

👉 **"Tailwind로 Bootstrap 같은 경험을 하게 해주는 라이브러리"**
이라고 생각하면 됨.

---

## 2. ⭐ DaisyUI를 사용하는 이유

### 2.1 Tailwind의 한계를 정확히 찌른다 💥

Tailwind는 유틸리티 중심이라 다음과 같은 문제가 있다:

```html
<button class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg shadow">SAVE</button>
```

✖ 매번 스타일을 조합해야 한다
✖ 일관성 유지 어려움
✖ UI 로직과 스타일이 뒤섞임
✖ 반복 작업 폭발

### DaisyUI는 다음과 같이 해결한다:

```html
<button class="btn btn-primary">SAVE</button>
```

끝. 😎

---

## 3. 📦 설치 및 설정 (Vite + React 기준)

### 3.1 설치 (필수)

```bash
npm i daisyui
```

### 3.2 tailwind.config.js 수정

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: { extend: {} },
  plugins: [
    import('@tailwindcss/typography'),
    import('@tailwindcss/forms'),
    import('daisyui')
  ]
}
```

### 3.3 DaisyUI 기본 동작 방식

* 테마 라는 개념을 자동 활성화
* 사용하는 HTML 요소에 DaisyUI 클래스명을 입력하면 적용
* Tailwind JIT 엔진으로 즉시 렌더링

---

## 4. 🌈 DaisyUI Theme (테마 시스템)

DaisyUI의 가장 강력한 기능 중 하나는 **테마(theme) 시스템**이다.

### 4.1 테마란?

전체 사이트 색상을 한번에 바꾸는 기능!

* `light`
* `dark`
* `cupcake`
* `bumblebee`
* `corporate`
* `synthwave`
* `retro`
* `cyberpunk`
* `valentine`
* `halloween`
* `forest`
* `luxury`
* `lofi`
* `pastel`
  … 등 30개가 넘는 테마 제공.

### 4.2 테마 적용 방법

```html
<html data-theme="forest">
```

또는 React라면:

```jsx
<div data-theme="retro">
  <App />
</div>
```

### 4.3 다크 모드 자동 전환

```html
<html data-theme="dark">
```

또는 JS로 토글 가능:

```jsx
document.documentElement.setAttribute("data-theme", "dark")
```

### 4.4 테마 토글 컴포넌트 (완성 형태)

```jsx
<button
  className="btn"
  onClick={() => {
    const theme =
      document.documentElement.getAttribute("data-theme") === "dark"
        ? "light"
        : "dark";
    document.documentElement.setAttribute("data-theme", theme);
  }}
>
  Toggle Theme
</button>
```

👉 강의 시 학생들이 매우 좋아하는 데모 주제!

---

## 5. 🎨 DaisyUI 컴포넌트 종류 (핵심 정리)

DaisyUI는 컴포넌트를 크게 아래처럼 나눌 수 있음:

### 5.1 Button 컴포넌트

```html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-accent">Accent</button>
<button class="btn btn-outline">Outline</button>
```

### 5.2 Card

```html
<div class="card w-96 bg-base-100 shadow-xl">
  <figure><img src="/img.jpg" alt="Shoes" /></figure>
  <div class="card-body">
    <h2 class="card-title">Shoes!</h2>
    <p>If a dog chews shoes whose shoes does he choose?</p>
    <div class="card-actions justify-end">
      <button class="btn btn-primary">BUY NOW</button>
    </div>
  </div>
</div>
```

### 5.3 Modal

```html
<button class="btn" onclick="my_modal.showModal()">open modal</button>
<dialog id="my_modal" class="modal">
  <form method="dialog" class="modal-box">
    <p>hello!</p>
  </form>
</dialog>
```

### 5.4 Navbar

```html
<div class="navbar bg-base-200">
  <a class="btn btn-ghost text-xl">daisyUI</a>
</div>
```

### 5.5 Tabs

```html
<div role="tablist" class="tabs tabs-boxed">
  <a role="tab" class="tab tab-active">Tab 1</a>
  <a role="tab" class="tab">Tab 2</a>
  <a role="tab" class="tab">Tab 3</a>
</div>
```

### 5.6 Dropdown

```html
<div class="dropdown">
  <button class="btn">Menu</button>
  <ul class="dropdown-content menu p-2 bg-base-100 rounded-box shadow">
    <li><a>Item 1</a></li>
    <li><a>Item 2</a></li>
  </ul>
</div>
```

총 50종이 넘는 컴포넌트가 있음.

---

## 6. 🔧 DaisyUI는 어떻게 동작하는가? (내부 구조)

### ✔ Tailwind의 플러그인 방식으로 동작한다

* DaisyUI는 Tailwind의 plugin system을 사용
* 컴포넌트 정의를 **UI 컴포넌트 → CSS Class** 로 변환
* JIT 엔진으로 필요할 때만 CSS 생성

### ✔ 실제 CSS를 컴파일하는 것이 아니라

> “Tailwind 클래스 확장 + Utility Class 조합의 템플릿을 제공한다”

### ✔ 커스텀 Theme는 CSS 변수 기반으로 관리된다

DaisyUI의 테마는 `:root` 또는 `[data-theme]` 수준에 CSS 변수를 정의해 놓고
각 컴포넌트가 이 변수를 사용한다.

예:

```css
[data-theme=light] {
  --p: 0 100% 50%;
  --s: 180 100% 50%;
}
```

컴포넌트는 다음처럼 변수를 사용:

```css
.btn-primary {
  background-color: hsl(var(--p));
}
```

👉 강력하고 확장성 높은 방법

---

## 7. 🧪 React + DaisyUI 실제 예제

```jsx
export default function App() {
  return (
    <div className="p-10 space-y-4">
      <button className="btn btn-primary">Primary</button>
      <button className="btn btn-secondary">Secondary</button>
      <div className="card w-96 bg-base-100 shadow-xl">
        <div className="card-body">
          <h2 className="card-title">Hello DaisyUI!</h2>
          <p>React + Tailwind + DaisyUI</p>
          <div className="card-actions justify-end">
            <button className="btn btn-accent">Go</button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 8. 📌 DaisyUI의 장점 요약

| 항목                | 설명                                            |
| ----------------- | --------------------------------------------- |
| 🎨 빠른 UI 구성       | 버튼 하나를 만들기 위해 10~20줄의 Tailwind 클래스를 만들 필요가 없음 |
| 🎯 테마 시스템 제공      | Light/Dark 테마 제공, 커스텀도 가능                     |
| 🧱 일관된 디자인        | 50+ 컴포넌트가 동일한 스타일 가이드 기반                      |
| ⚡ JIT 최적화         | 필요할 때만 CSS 생성                                 |
| 🔌 Tailwind 철학 유지 | Utility-first 방식과 100% 호환                     |

---

## 9. ❗ 주의할 점

* 클래스명이 직관적이지만, **Tailwind 순수 스타일링보다 통제력이 떨어질 수 있음**
* UI 디자인을 완전 커스터마이징하려면 추가 CSS 필요
* 대규모 디자인 시스템에는 조금 가벼움

그러나 **국비 과정**, **교육용 프로젝트**, **스타트업 초기 MVP**, **개인 프로젝트**에서는
👍 **가장 효율적인 UI 라이브러리**라고 해도 과언이 아니다.

---

# 🎉 결론

DaisyUI는 단순한 Tailwind 플러그인이 아니다.
**Tailwind의 생산성을 10배 올려주는 실전 UI 프레임워크**다.

* 버튼
* 카드
* 모달
* 네비게이션
* 테마
* 반응형
* 다크모드

모든 걸 클라스 하나로 해결할 수 있다.

React + Tailwind를 완성된 UI 라이브러리처럼 사용하고 싶다면
DaisyUI는 사실상 **정답**이다. 🌼✨

---

참고 사항:

✅ DaisyUI 전체 컴포넌트 사용 예제(React 버전)
✅ DaisyUI 테마 토글 예제 (Dark ↔ Light)
✅ DaisyUI 기반 Vite + React 프로젝트 템플릿 전체 구조
✅ DaisyUI를 커스터마이징하는 고급 테크닉 소개 블로그


