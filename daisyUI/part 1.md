# PART 1. daisyUI란 무엇인가?

## 1. 이번 PART에서 배울 내용

이번 PART에서는 daisyUI의 세부 컴포넌트를 배우기 전에 다음 질문에 답합니다.

* daisyUI란 무엇인가?
* 왜 Tailwind CSS와 함께 사용하는가?
* Tailwind CSS만 사용할 때와 무엇이 다른가?
* Utility Class와 Component Class의 차이는 무엇인가?
* daisyUI를 사용하면 무엇이 편리해지는가?
* daisyUI는 React Component Library인가?
* daisyUI를 프로젝트에 어떻게 적용하는가?

이번 PART의 핵심은 다음 한 문장입니다.

> **daisyUI는 Tailwind CSS 위에서 동작하며, 여러 Tailwind 스타일을 의미 있는 Component Class로 사용할 수 있도록 해주는 UI Component Library입니다.**

---

# 2. Tailwind CSS로 버튼 만들기

Tailwind CSS에서는 Utility Class를 조합해서 UI를 만듭니다.

예를 들어 버튼 하나를 만들어 보겠습니다.

```html
<button
  class="
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
```

Tailwind CSS를 배웠다면 각각의 클래스가 무엇을 의미하는지 알 수 있습니다.

```text
px-4
 └─ 좌우 padding

py-2
 └─ 상하 padding

bg-blue-500
 └─ 배경색

text-white
 └─ 글자색

font-semibold
 └─ 글자 굵기

rounded-lg
 └─ border-radius

hover:bg-blue-600
 └─ hover 상태의 배경색
```

즉 Tailwind CSS의 기본적인 스타일링 방식은 다음과 같습니다.

```text
HTML Element
      │
      ▼
여러 Utility Class 조합
      │
      ▼
최종 UI
```

버튼 하나를 만드는 데도 여러 Utility Class가 필요합니다.

---

# 3. 버튼이 많아지면?

버튼 하나만 있다면 큰 문제가 없습니다.

하지만 애플리케이션에는 수많은 버튼이 존재합니다.

```text
로그인
회원가입
저장
수정
삭제
취소
확인
검색
장바구니
구매하기
```

이때 매번 다음과 같이 작성한다면 코드가 반복됩니다.

```html
<button
  class="px-4 py-2 bg-blue-500 text-white font-semibold rounded-lg hover:bg-blue-600"
>
  저장
</button>
```

```html
<button
  class="px-4 py-2 bg-blue-500 text-white font-semibold rounded-lg hover:bg-blue-600"
>
  수정
</button>
```

```html
<button
  class="px-4 py-2 bg-blue-500 text-white font-semibold rounded-lg hover:bg-blue-600"
>
  확인
</button>
```

여기서 문제가 발생합니다.

```text
Utility Class 증가
        ↓
class가 길어짐
        ↓
같은 스타일 반복
        ↓
일관된 디자인 관리가 어려워짐
```

물론 React Component로 추상화하거나 별도의 CSS를 작성하는 방법도 있습니다.

하지만 버튼뿐만 아니라 다음과 같은 UI를 모두 직접 설계해야 한다면 상당한 작업이 필요합니다.

```text
Button
Card
Input
Select
Checkbox
Badge
Alert
Modal
Navbar
Menu
Tabs
Table
Pagination
...
```

이 문제를 해결하는 방법 중 하나가 **daisyUI**입니다.

---

# 4. daisyUI란?

daisyUI는 **Tailwind CSS 기반의 UI Component Library**입니다.

Tailwind CSS가 다음과 같은 Utility Class를 제공한다면,

```text
px-4
py-2
rounded-lg
font-semibold
bg-blue-500
```

daisyUI는 다음과 같은 Component Class를 제공합니다.

```text
btn
card
input
select
alert
badge
modal
navbar
menu
tabs
```

예를 들어 버튼은 다음처럼 만들 수 있습니다.

```html
<button class="btn">
  저장
</button>
```

색상까지 지정하려면:

```html
<button class="btn btn-primary">
  저장
</button>
```

코드가 훨씬 단순해집니다.

---

# 5. Tailwind CSS와 daisyUI의 관계

여기서 가장 중요한 부분입니다.

daisyUI는 Tailwind CSS와 경쟁하는 별개의 CSS 프레임워크라고 이해하면 안 됩니다.

구조적으로는 다음과 같이 생각하는 것이 좋습니다.

```text
┌──────────────────────────────┐
│          Application         │
│                              │
│   btn   card   modal   ...   │
├──────────────────────────────┤
│            daisyUI           │
│                              │
│      Component Classes       │
├──────────────────────────────┤
│         Tailwind CSS         │
│                              │
│        Utility System        │
└──────────────────────────────┘
```

즉,

```text
Tailwind CSS
      ↓
Utility 기반 스타일링 시스템 제공

daisyUI
      ↓
그 위에서 사용할 수 있는
Component Class와 Theme 시스템 제공
```

이라고 이해하면 됩니다.

---

# 6. Utility Class와 Component Class

두 개념의 차이를 정확하게 이해해야 합니다.

## Tailwind CSS

Tailwind CSS에서는 주로 **어떤 스타일을 적용할 것인가**를 직접 표현합니다.

```html
<button class="px-4 py-2 rounded-lg">
```

클래스를 보면 스타일 속성이 드러납니다.

```text
px-4
→ 좌우 padding

py-2
→ 상하 padding

rounded-lg
→ 둥근 모서리
```

이를 **Utility Class**라고 합니다.

---

## daisyUI

daisyUI에서는 다음과 같이 작성할 수 있습니다.

```html
<button class="btn">
```

`btn`은 padding이나 border-radius 같은 개별 CSS 속성을 직접 표현하지 않습니다.

대신,

```text
btn
 ↓
"이 Element는 Button UI다"
```

라는 의미를 표현합니다.

따라서 `btn`은 **Component Class**입니다.

---

# 7. 같은 버튼 비교하기

Tailwind CSS만 사용하는 경우를 보겠습니다.

```html
<button
  class="
    px-4
    py-2
    rounded-lg
    font-semibold
    bg-blue-500
    text-white
    hover:bg-blue-600
  "
>
  저장
</button>
```

daisyUI를 사용하면:

```html
<button class="btn btn-primary">
  저장
</button>
```

개념적으로 비교하면 다음과 같습니다.

```text
Tailwind CSS

<button>
   │
   ├─ px-4
   ├─ py-2
   ├─ rounded-lg
   ├─ font-semibold
   ├─ bg-blue-500
   ├─ text-white
   └─ hover:bg-blue-600


daisyUI

<button>
   │
   ├─ btn
   └─ btn-primary
```

daisyUI는 여러 스타일 규칙을 **의미 있는 Component Class**로 사용할 수 있도록 합니다.

---

# 8. 그렇다면 Tailwind CSS는 더 이상 사용하지 않는가?

아닙니다.

이 부분을 반드시 이해해야 합니다.

daisyUI를 사용해도 Tailwind CSS Utility Class는 계속 사용합니다.

예를 들어:

```html
<button class="btn btn-primary w-full mt-4">
  로그인
</button>
```

여기에는 두 종류의 클래스가 함께 사용되고 있습니다.

```text
btn
btn-primary
     │
     └─ daisyUI

w-full
mt-4
     │
     └─ Tailwind CSS
```

역할을 나누면:

```text
daisyUI
   │
   └─ Button의 기본적인 디자인과 상태

Tailwind CSS
   │
   └─ width, margin 등 세부 Layout 조정
```

즉 둘은 함께 사용합니다.

---

# 9. 실제 개발에서는 이렇게 사용한다

예를 들어 로그인 화면을 만든다고 하겠습니다.

```html
<div class="max-w-md mx-auto mt-20">
  <div class="card bg-base-100 shadow-xl">
    <div class="card-body">

      <h2 class="card-title">
        로그인
      </h2>

      <input
        type="email"
        placeholder="Email"
        class="input input-bordered w-full"
      />

      <input
        type="password"
        placeholder="Password"
        class="input input-bordered w-full"
      />

      <button class="btn btn-primary w-full">
        로그인
      </button>

    </div>
  </div>
</div>
```

여기에서도 두 시스템이 함께 사용됩니다.

```text
daisyUI
────────────────────
card
card-body
card-title
input
input-bordered
btn
btn-primary
bg-base-100


Tailwind CSS
────────────────────
max-w-md
mx-auto
mt-20
shadow-xl
w-full
```

따라서 실제 개발 구조는 다음과 같습니다.

```text
daisyUI
   +
Tailwind CSS
   ↓
Application UI
```

---

# 10. daisyUI가 제공하는 것

daisyUI는 Button 하나만 제공하는 것이 아닙니다.

대표적으로 다음과 같은 UI Component Class를 제공합니다.

```text
Actions
├─ Button
├─ Dropdown
├─ Modal
├─ Swap
└─ Theme Controller

Data Display
├─ Accordion
├─ Avatar
├─ Badge
├─ Card
├─ Carousel
├─ Chat Bubble
├─ Collapse
├─ Countdown
├─ Kbd
├─ Stat
└─ Table

Navigation
├─ Breadcrumbs
├─ Dock
├─ Link
├─ Menu
├─ Navbar
├─ Pagination
├─ Steps
└─ Tabs

Feedback
├─ Alert
├─ Loading
├─ Progress
├─ Radial Progress
├─ Skeleton
├─ Status
└─ Toast

Data Input
├─ Checkbox
├─ Fieldset
├─ File Input
├─ Filter
├─ Label
├─ Radio
├─ Range
├─ Rating
├─ Select
├─ Text Input
├─ Textarea
└─ Toggle

Layout
├─ Divider
├─ Drawer
├─ Footer
├─ Hero
├─ Indicator
├─ Join
├─ Mask
└─ Stack
```

따라서 일반적인 웹 애플리케이션에서 자주 사용하는 UI를 빠르게 구성할 수 있습니다.

---

# 11. daisyUI는 React Component Library인가?

여기서 많이 하는 오해가 있습니다.

daisyUI는 React 전용 Component Library가 아닙니다.

예를 들어 다음 코드를 보면:

```jsx
<button className="btn btn-primary">
  저장
</button>
```

`Button`이라는 React Component를 import하지 않았습니다.

다음과 같은 코드가 아닙니다.

```jsx
import { Button } from "some-library"

<Button>저장</Button>
```

daisyUI에서는 일반적인 HTML Element를 사용합니다.

```jsx
<button>
```

그리고 class를 적용합니다.

```jsx
className="btn btn-primary"
```

즉 핵심은 다음과 같습니다.

```text
React Component를 제공
        X

HTML Element
     +
daisyUI Class
        O
```

따라서 daisyUI는 React뿐만 아니라 Tailwind CSS를 사용하는 다양한 환경에서 사용할 수 있습니다.

---

# 12. JavaScript 동작도 제공하는가?

이것도 중요한 특징입니다.

daisyUI는 UI의 **스타일과 상태 표현**에 집중합니다.

예를 들어 Modal을 사용할 때 실제 애플리케이션에서는 React state와 함께 사용할 수 있습니다.

```jsx
const [open, setOpen] = useState(false)
```

```text
React
 │
 ├─ 상태 관리
 ├─ Event 처리
 └─ UI 렌더링

daisyUI
 │
 └─ UI 스타일
```

즉,

```text
동작
→ React / JavaScript

디자인
→ Tailwind CSS + daisyUI
```

라는 역할 분리가 가능합니다.

이 개념은 이후 Modal, Drawer, Dropdown 등을 배울 때 매우 중요합니다.

---

# 13. daisyUI의 Theme

daisyUI의 또 다른 중요한 특징은 **Theme 시스템**입니다.

예를 들어:

```html
<html data-theme="light">
```

또는:

```html
<html data-theme="dark">
```

처럼 Theme을 적용할 수 있습니다.

그리고 다음과 같은 Semantic Color를 사용합니다.

```text
primary
secondary
accent
neutral

base-100
base-200
base-300

info
success
warning
error
```

따라서:

```html
<button class="btn btn-primary">
```

에서 `primary`는 특정한 `blue-500` 같은 물리적인 색상 자체를 의미하기보다는 **현재 Theme에서 primary 역할을 담당하는 색상**을 의미합니다.

이 내용은 PART 3에서 자세히 다룹니다.

---

# 14. 프로젝트에 daisyUI 설치하기

Tailwind CSS가 구성된 프로젝트에서 daisyUI를 설치합니다.

```bash
npm install daisyui
```

설치 후 현재 사용하는 Tailwind CSS 및 daisyUI 버전에 맞게 프로젝트 CSS에서 daisyUI를 등록합니다.

예를 들어 최신 Tailwind CSS 기반 설정에서는 다음과 같은 형태를 사용할 수 있습니다.

```css
@import "tailwindcss";
@plugin "daisyui";
```

이후 React Component에서 바로 daisyUI class를 사용할 수 있습니다.

```jsx
function App() {
  return (
    <button className="btn btn-primary">
      Hello daisyUI
    </button>
  )
}

export default App
```

---

# 15. 첫 번째 daisyUI 화면

간단한 Card를 만들어 보겠습니다.

```jsx
function App() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="card w-96 bg-base-100 shadow-xl">
        <div className="card-body">
          <h2 className="card-title">
            daisyUI
          </h2>

          <p>
            Tailwind CSS와 daisyUI를 함께 사용합니다.
          </p>

          <div className="card-actions justify-end">
            <button className="btn btn-primary">
              시작하기
            </button>
          </div>
        </div>
      </div>
    </div>
  )
}
```

여기서도 역할을 구분해 봅시다.

```text
Tailwind CSS
────────────────
min-h-screen
flex
items-center
justify-center
w-96
justify-end
shadow-xl


daisyUI
────────────────
card
bg-base-100
card-body
card-title
card-actions
btn
btn-primary
```

둘 중 하나만 사용하는 것이 아니라 **서로 역할을 나누어 함께 사용**합니다.

---

# 16. daisyUI를 사용하는 이유

daisyUI의 장점을 정리하면 다음과 같습니다.

### ① 코드가 짧아진다

```html
<button class="btn btn-primary">
```

### ② UI 디자인의 일관성을 유지하기 쉽다

모든 버튼에서 공통된 Component Class를 사용할 수 있습니다.

```text
btn
 ├─ btn-primary
 ├─ btn-secondary
 ├─ btn-success
 └─ btn-error
```

### ③ 개발 속도가 빨라진다

Button, Card, Modal, Navbar 등의 기본 UI를 처음부터 설계할 필요가 줄어듭니다.

### ④ Tailwind CSS와 함께 사용할 수 있다

```html
<button class="btn btn-primary w-full mt-4">
```

### ⑤ Theme을 이용하기 쉽다

```text
light
dark
cupcake
business
...
```

같은 Theme 시스템을 활용할 수 있습니다.

---

# 17. 가장 중요한 오해 정리

### 오해 1

```text
daisyUI를 사용하면
Tailwind CSS는 필요 없다.
```

잘못된 이해입니다.

```text
Tailwind CSS
      +
daisyUI
      ↓
Application UI
```

---

### 오해 2

```text
daisyUI는 React Component Library다.
```

정확하지 않습니다.

daisyUI는 다음과 같이 class를 중심으로 사용합니다.

```jsx
<button className="btn">
```

---

### 오해 3

```text
daisyUI가 애플리케이션 상태를 관리한다.
```

아닙니다.

```text
React
→ State / Event / Rendering

Tailwind CSS
→ Utility Styling

daisyUI
→ Component Styling / Theme
```

---

# 18. 전체 구조 한눈에 보기

```text
                  React Application
                         │
          ┌──────────────┴──────────────┐
          │                             │
     Application Logic                  UI
          │                             │
   State / Event / API          ┌───────┴───────┐
          │                     │               │
        React                daisyUI       Tailwind CSS
                                │               │
                         Component Class   Utility Class
                                │               │
                                └───────┬───────┘
                                        │
                                        ▼
                                  Final Design
```

---

# 19. PART 1 핵심 정리

```text
Tailwind CSS
    │
    ├─ Utility Class
    │
    ├─ flex
    ├─ grid
    ├─ p-4
    ├─ mt-4
    └─ w-full
           │
           │
           ▼
        daisyUI
           │
           ├─ Component Class
           │
           ├─ btn
           ├─ card
           ├─ input
           ├─ alert
           ├─ modal
           └─ navbar
                  │
                  ▼
             Application UI
```

가장 중요한 문장은 이것입니다.

> **Tailwind CSS는 UI를 구성하는 Utility를 제공하고, daisyUI는 그 위에서 자주 사용하는 UI 패턴을 Component Class와 Theme으로 제공합니다.**

따라서 daisyUI를 배운다는 것은 Tailwind CSS를 버리고 새로운 CSS 시스템으로 넘어가는 것이 아닙니다.

**이미 배운 Tailwind CSS 위에 한 단계 더 높은 수준의 UI Component 시스템을 추가하는 것**이라고 이해하면 됩니다.

---

# 다음 PART

**PART 2. daisyUI Class 조합과 Modifier**

다음 PART에서는 다음 코드를 정확하게 읽는 방법을 배웁니다.

```html
<button class="btn btn-primary btn-lg">
  저장
</button>
```

```text
btn
 │
 ├─ Component
 │
 ├─ btn-primary → Color
 │
 └─ btn-lg      → Size
```

그리고 daisyUI Component Class와 Tailwind Utility Class를 어떤 기준으로 조합해야 하는지 자세히 살펴봅니다.
