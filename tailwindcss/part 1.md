# PART 1. Tailwind CSS 시작하기

## 1. Tailwind CSS란?

Tailwind CSS는 웹 UI를 빠르게 만들기 위한 **Utility-First CSS Framework**입니다.

일반적인 CSS에서는 먼저 클래스 이름을 만들고, CSS 파일에서 해당 클래스에 스타일을 정의합니다.

```html
<button class="submit-button">
  저장
</button>
```

```css
.submit-button {
  padding: 8px 16px;
  background-color: blue;
  color: white;
  border-radius: 8px;
}
```

Tailwind CSS에서는 별도의 `.submit-button` 스타일을 작성하는 대신, 미리 준비된 **Utility Class**를 HTML 또는 JSX에 직접 조합합니다.

```jsx
<button className="px-4 py-2 bg-blue-500 text-white rounded-lg">
  저장
</button>
```

핵심 차이는 다음과 같습니다.

```text
일반 CSS

HTML / JSX
    │
    │ class="submit-button"
    ▼
CSS
    │
    │ .submit-button { ... }
    ▼
스타일 적용


Tailwind CSS

HTML / JSX
    │
    │ px-4 py-2 bg-blue-500 ...
    ▼
Utility Classes
    │
    ▼
스타일 적용
```

Tailwind CSS를 이해할 때 가장 중요한 것은 다음 한 문장입니다.

> **Tailwind CSS는 작은 Utility Class들을 조합하여 UI를 만드는 CSS Framework이다.**

---

# 2. Utility란?

`utility`는 일반적으로 **특정한 기능이나 용도를 제공하는 작은 도구**라는 의미입니다.

Tailwind CSS에서 Utility Class는 보통 하나의 작은 스타일 역할을 담당합니다.

예를 들어:

```text
p-4             padding
mt-4            margin-top
w-full          width
text-center     text-align
font-bold       font-weight
bg-blue-500     background-color
rounded-lg      border-radius
flex            display
```

따라서 다음 코드는:

```jsx
<div className="p-4 bg-blue-500 text-white rounded-lg">
  Hello Tailwind
</div>
```

여러 개의 작은 스타일을 조합한 것입니다.

개념적으로 보면:

```text
p-4
+
bg-blue-500
+
text-white
+
rounded-lg
        │
        ▼
┌─────────────────────┐
│   Hello Tailwind    │
└─────────────────────┘
```

이러한 방식을 **Utility-First**라고 합니다.

---

# 3. Tailwind CSS도 결국 CSS다

Tailwind CSS를 처음 배우는 학생들이 가장 많이 하는 오해가 있습니다.

> Tailwind CSS는 CSS와 다른 새로운 스타일 언어인가?

아닙니다.

Tailwind의 Utility Class는 결국 CSS 속성에 대응합니다.

예를 들어:

```html
<div class="flex">
```

에서 `flex`는 개념적으로 다음 CSS에 대응합니다.

```css
display: flex;
```

다음 Tailwind 코드도:

```html
<div class="justify-center">
```

CSS로 생각하면:

```css
justify-content: center;
```

입니다.

몇 가지를 비교하면 다음과 같습니다.

| CSS                       | Tailwind CSS     |
| ------------------------- | ---------------- |
| `display: flex`           | `flex`           |
| `display: grid`           | `grid`           |
| `justify-content: center` | `justify-center` |
| `align-items: center`     | `items-center`   |
| `width: 100%`             | `w-full`         |
| `font-weight: 700`        | `font-bold`      |
| `text-align: center`      | `text-center`    |
| `border-radius: 0.5rem`   | `rounded-lg`     |

따라서 Tailwind를 배운다는 것은 완전히 새로운 스타일 시스템을 배우는 것이 아닙니다.

기존 CSS 지식을 다음과 같이 표현하는 방법을 배우는 것입니다.

```text
CSS 개념
   │
   ▼
CSS Property / Value
   │
   ▼
Tailwind Utility Class
```

예를 들어 CSS Flexbox를 알고 있다면:

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

Tailwind에서는:

```html
<div class="flex justify-center items-center">
```

라고 표현할 수 있습니다.

즉,

> **CSS를 알고 있으면 Tailwind CSS는 훨씬 쉽게 배울 수 있습니다.**

---

# 4. 일반 CSS 방식

다음과 같은 버튼을 만든다고 생각해 봅시다.

```html
<button class="primary-button">
  저장
</button>
```

CSS를 작성합니다.

```css
.primary-button {
  padding: 8px 16px;
  background-color: #3b82f6;
  color: white;
  border-radius: 8px;
  font-weight: 600;
}
```

구조는 다음과 같습니다.

```text
HTML

<button class="primary-button">
        │
        ▼
CSS Selector

.primary-button
        │
        ▼
CSS Properties

padding
background-color
color
border-radius
font-weight
```

이 방식에서는 `primary-button`이라는 **의미 있는 클래스 이름**을 먼저 만들고 그 클래스가 어떤 스타일을 가질지 CSS 파일에서 정의합니다.

---

# 5. Tailwind CSS 방식

같은 버튼을 Tailwind CSS로 작성하면 다음과 같습니다.

```jsx
<button
  className="
    px-4
    py-2
    bg-blue-500
    text-white
    rounded-lg
    font-semibold
  "
>
  저장
</button>
```

각 Utility Class가 하나의 역할을 담당합니다.

```text
px-4
 └─ 좌우 padding

py-2
 └─ 상하 padding

bg-blue-500
 └─ background color

text-white
 └─ text color

rounded-lg
 └─ border radius

font-semibold
 └─ font weight
```

따라서 기존 방식처럼:

```text
primary-button
       │
       ▼
CSS 파일을 찾아감
       │
       ▼
스타일 확인
```

하는 대신 Tailwind에서는:

```text
HTML / JSX

px-4 py-2 bg-blue-500 text-white rounded-lg
 │     │        │          │          │
 ▼     ▼        ▼          ▼          ▼
padding padding background color    radius
```

처럼 **스타일을 사용하는 위치에서 바로 확인할 수 있습니다.**

---

# 6. Utility-First란?

Tailwind CSS를 설명할 때 가장 중요한 용어가 **Utility-First**입니다.

일반 CSS에서는 보통 UI의 의미를 기준으로 클래스를 만듭니다.

```css
.card {
  ...
}

.login-button {
  ...
}

.product-title {
  ...
}
```

이를 하나의 큰 스타일 묶음이라고 생각할 수 있습니다.

```text
.login-button
     │
     ├─ padding
     ├─ background
     ├─ color
     ├─ border-radius
     └─ font-weight
```

Tailwind는 반대로 작은 Utility들을 준비하고 필요한 것들을 조합합니다.

```text
px-4
py-2
bg-blue-500
text-white
rounded-lg
font-semibold
       │
       ▼
   하나의 버튼
```

그래서 이름이 **Utility-First**입니다.

---

# 7. Tailwind CSS는 Inline Style과 같은 것인가?

처음 보면 Tailwind는 Inline Style과 비슷해 보일 수 있습니다.

Inline Style:

```jsx
<button
  style={{
    padding: "8px 16px",
    backgroundColor: "blue",
    color: "white"
  }}
>
  저장
</button>
```

Tailwind:

```jsx
<button className="px-4 py-2 bg-blue-500 text-white">
  저장
</button>
```

둘 다 컴포넌트 근처에서 스타일을 확인할 수 있다는 공통점이 있습니다.

하지만 동일한 방식은 아닙니다.

Tailwind는 **CSS Class 기반**입니다.

```html
class="bg-blue-500"
```

브라우저에는 실제 CSS 규칙이 적용됩니다.

개념적으로:

```css
.bg-blue-500 {
  background-color: ...;
}
```

따라서 Tailwind에서는 CSS의 여러 기능을 자연스럽게 사용할 수 있습니다.

예를 들어 `hover` 상태를:

```jsx
<button className="bg-blue-500 hover:bg-blue-600">
  저장
</button>
```

처럼 표현할 수 있습니다.

Responsive Design 역시:

```jsx
<div className="text-sm md:text-lg">
```

처럼 표현할 수 있습니다.

즉, Tailwind는 Inline Style 문법이 아니라 **CSS Class를 생성하고 조합해서 사용하는 시스템**입니다.

---

# 8. Tailwind CSS를 사용하면 CSS를 몰라도 되는가?

아닙니다.

Tailwind를 제대로 사용하려면 CSS의 핵심 개념을 알고 있어야 합니다.

특히 다음 내용은 중요합니다.

```text
CSS
 │
 ├─ Box Model
 │    ├─ width / height
 │    ├─ margin
 │    ├─ padding
 │    └─ border
 │
 ├─ Typography
 │
 ├─ Position
 │
 ├─ Flexbox
 │
 ├─ Grid
 │
 └─ Responsive Design
```

Tailwind는 이러한 CSS 개념을 없애는 것이 아니라 **다른 문법으로 편리하게 표현할 수 있게 해줍니다.**

예를 들어:

```html
<div class="flex justify-between items-center">
```

를 이해하려면 다음 CSS 개념을 알고 있어야 합니다.

```css
display: flex;
justify-content: space-between;
align-items: center;
```

따라서:

> **CSS를 모르고 Tailwind를 배우는 것보다 CSS를 이해한 상태에서 Tailwind 표현법을 배우는 것이 훨씬 중요합니다.**

---

# 9. Tailwind CSS의 장점

## 빠른 UI 개발

별도의 CSS 파일로 이동하지 않고 JSX에서 바로 스타일을 작성할 수 있습니다.

```jsx
<div className="p-6 bg-white rounded-xl shadow">
```

따라서:

```text
JSX 작성
   ↓
CSS 파일 이동
   ↓
클래스 작성
   ↓
JSX로 이동
```

하는 작업을 줄일 수 있습니다.

---

## 일관된 Design System

개발자가 임의의 값을 계속 만드는 상황을 줄일 수 있습니다.

예를 들어 spacing을:

```text
p-1
p-2
p-3
p-4
p-6
p-8
```

처럼 정해진 scale을 중심으로 사용할 수 있습니다.

색상 역시:

```text
blue-100
blue-200
blue-300
...
blue-900
```

처럼 일관된 체계를 사용할 수 있습니다.

---

## 클래스 이름 고민 감소

일반 CSS에서는 다음과 같은 고민이 자주 발생합니다.

```text
이 클래스 이름을 뭐라고 하지?

product-card?
product-item?
product-box?
product-container?
```

Tailwind에서는 스타일 자체를 Utility Class로 표현하므로 이러한 고민이 크게 줄어듭니다.

---

## Responsive Design을 쉽게 표현

다음처럼 breakpoint prefix를 붙일 수 있습니다.

```html
<div class="w-full md:w-1/2 lg:w-1/3">
```

의미는 대략 다음과 같습니다.

```text
작은 화면
w-full
   │
   ▼
중간 화면 이상
md:w-1/2
   │
   ▼
큰 화면 이상
lg:w-1/3
```

Responsive Design은 이후 PART에서 자세히 다룹니다.

---

## 상태 스타일을 쉽게 표현

버튼의 상태도 클래스 조합으로 표현할 수 있습니다.

```jsx
<button
  className="
    bg-blue-500
    hover:bg-blue-600
    focus:ring-2
    disabled:opacity-50
  "
>
  저장
</button>
```

```text
기본 상태      bg-blue-500
hover          hover:bg-blue-600
focus          focus:ring-2
disabled       disabled:opacity-50
```

---

# 10. Tailwind CSS의 단점

Tailwind CSS가 모든 상황에서 완벽한 것은 아닙니다.

가장 먼저 느끼는 단점은 HTML 또는 JSX의 클래스가 길어진다는 것입니다.

```jsx
<button
  className="
    inline-flex
    items-center
    justify-center
    px-4
    py-2
    bg-blue-500
    text-white
    font-semibold
    rounded-lg
    shadow
    hover:bg-blue-600
  "
>
  저장
</button>
```

처음 보는 학생에게는 일반 CSS보다 복잡해 보일 수 있습니다.

또한 다음과 같은 Tailwind 표현법을 익혀야 합니다.

```text
p-4
px-4
gap-4
w-full
max-w-xl
items-center
justify-between
rounded-lg
```

하지만 CSS 개념과 연결해서 학습하면 대부분의 Utility Class를 어렵지 않게 이해할 수 있습니다.

---

# 11. React와 Tailwind CSS

React에서는 HTML의 `class` 대신 `className`을 사용합니다.

HTML:

```html
<div class="p-4 bg-gray-100">
  Hello
</div>
```

React JSX:

```jsx
<div className="p-4 bg-gray-100">
  Hello
</div>
```

Tailwind 자체의 차이가 아니라 **JSX 문법 때문**입니다.

React 컴포넌트에서는 다음과 같이 사용할 수 있습니다.

```jsx
function ProductCard() {
  return (
    <div className="p-6 bg-white rounded-xl shadow">
      <h2 className="text-xl font-bold">
        Keyboard
      </h2>

      <p className="mt-2 text-gray-600">
        Mechanical Keyboard
      </p>

      <button className="mt-4 px-4 py-2 bg-blue-500 text-white rounded-lg">
        구매
      </button>
    </div>
  )
}
```

구조와 스타일이 한 컴포넌트 안에서 함께 보입니다.

```text
ProductCard
 │
 ├─ div
 │   └─ Card Layout
 │
 ├─ h2
 │   └─ Typography
 │
 ├─ p
 │   └─ Description Style
 │
 └─ button
     └─ Button Style
```

이러한 특성 때문에 Tailwind CSS는 **Component 기반 UI 개발과 잘 어울립니다.**

---

# 12. Tailwind CSS를 공부하는 방법

Tailwind Utility Class를 전부 암기할 필요는 없습니다.

다음 방식으로 공부하는 것이 좋습니다.

```text
CSS 개념 이해
      ↓
CSS Property 확인
      ↓
대응하는 Tailwind Utility 확인
      ↓
직접 UI 작성
      ↓
반복 사용
      ↓
자연스럽게 익숙해짐
```

예를 들어 Flexbox를 알고 있다면:

```css
display: flex;
justify-content: space-between;
align-items: center;
gap: 16px;
```

Tailwind 표현을 연결해서 학습합니다.

```html
<div class="flex justify-between items-center gap-4">
```

결국 중요한 것은 클래스 이름 암기가 아니라:

```text
무엇을 만들 것인가?
       ↓
어떤 CSS가 필요한가?
       ↓
Tailwind에서는 어떻게 표현하는가?
```

의 사고 과정입니다.

---

# 13. 핵심 정리

Tailwind CSS의 핵심 구조를 정리하면 다음과 같습니다.

```text
                Tailwind CSS
                     │
                     ▼
              Utility-First
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Spacing       Layout       Visual
   p-4, m-4      flex, grid   bg, border
        │            │            │
        └────────────┼────────────┘
                     ▼
             Utility 조합
                     │
                     ▼
               UI Component
```

가장 중요한 것은 다음 세 가지입니다.

**첫째**, Tailwind CSS는 CSS를 대체하는 새로운 스타일 언어가 아닙니다.

**둘째**, 기존 CSS의 Property와 Value를 Utility Class 형태로 표현합니다.

**셋째**, 작은 Utility Class들을 조합해서 UI를 만드는 방식을 **Utility-First**라고 합니다.

따라서 Tailwind CSS 학습의 핵심은:

```text
CSS를 버리고 Tailwind를 배운다
              X

CSS를 이해하고
그 CSS를 Tailwind로 표현하는 방법을 배운다
              O
```

입니다.

---

# 다음 PART

**PART 2. Tailwind CSS 설치와 개발환경 구성**

다음 PART에서는 Vite + React 프로젝트를 기준으로 Tailwind CSS를 설치하고 실제 Utility Class가 브라우저에 적용되는 과정까지 살펴봅니다.

```text
Vite + React
     ↓
Tailwind CSS 설치
     ↓
CSS 연결
     ↓
JSX에서 className 사용
     ↓
Tailwind CSS 생성
     ↓
Browser Rendering
```
