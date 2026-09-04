# PART 2. daisyUI Class 조합과 Modifier

## 1. 이번 PART에서 배울 내용

PART 1에서는 다음과 같은 코드를 사용했습니다.

```html
<button class="btn btn-primary">
  저장
</button>
```

이번에는 여기서 한 단계 더 나아가 다음 코드를 정확하게 읽어보겠습니다.

```html
<button class="btn btn-primary btn-lg btn-wide">
  저장
</button>
```

각 Class에는 서로 다른 역할이 있습니다.

```text
btn
│
├─ btn-primary
├─ btn-lg
└─ btn-wide
```

이번 PART에서는 다음 내용을 학습합니다.

* Component Class
* Modifier
* Color Modifier
* Size Modifier
* Style Modifier
* State Modifier
* 여러 Modifier의 조합
* Tailwind CSS Utility Class와의 조합

이번 PART의 핵심은 다음 한 문장입니다.

> **daisyUI는 Component Class를 중심으로 필요한 Modifier를 조합하여 UI를 구성합니다.**

---

# 2. 가장 먼저 Component Class를 찾는다

다음 코드를 보겠습니다.

```html
<button class="btn btn-primary">
  저장
</button>
```

가장 중요한 Class는:

```text
btn
```

입니다.

`btn`은 이 Element를 **Button UI로 스타일링하는 Component Class**입니다.

```text
<button>
   │
   ▼
  btn
   │
   ▼
daisyUI Button
```

즉:

```html
<button>
  저장
</button>
```

은 일반 HTML Button이고,

```html
<button class="btn">
  저장
</button>
```

은 daisyUI의 Button 스타일을 사용하는 Element가 됩니다.

---

# 3. Component Class란?

daisyUI에는 여러 Component Class가 있습니다.

예를 들어:

```text
btn
card
input
select
badge
alert
navbar
modal
```

각 Class는 특정 UI Component를 표현합니다.

```text
btn
   ↓
Button

card
   ↓
Card

input
   ↓
Input

alert
   ↓
Alert
```

따라서 daisyUI 코드를 볼 때는 **가장 먼저 Component Class가 무엇인지 찾는 습관**을 들이는 것이 좋습니다.

예:

```html
<div class="alert alert-success">
  저장되었습니다.
</div>
```

여기서 Component Class는:

```text
alert
```

입니다.

그리고:

```text
alert-success
```

는 `alert`를 변경하는 Modifier입니다.

---

# 4. Modifier란?

Modifier는 **Component의 기본적인 표현을 변경하는 Class**입니다.

예를 들어:

```html
<button class="btn">
  Default
</button>
```

기본 Button에:

```text
btn-primary
```

를 추가하면:

```html
<button class="btn btn-primary">
  Primary
</button>
```

Primary Variant가 됩니다.

다시:

```text
btn-lg
```

를 추가하면:

```html
<button class="btn btn-primary btn-lg">
  Primary
</button>
```

큰 Button이 됩니다.

개념적으로:

```text
Component Class
      │
      ▼
     btn
      │
      ├── btn-primary
      │      └─ Color
      │
      └── btn-lg
             └─ Size
```

라고 볼 수 있습니다.

---

# 5. Component Class + Modifier

daisyUI에서 자주 볼 수 있는 패턴입니다.

```text
Component Class
      +
Modifier
```

예:

```html
<button class="btn btn-primary">
```

```text
btn
│
└─ Component Class

btn-primary
│
└─ Modifier
```

또 다른 예:

```html
<div class="alert alert-success">
```

```text
alert
│
└─ Component Class

alert-success
│
└─ Modifier
```

Badge도 같습니다.

```html
<div class="badge badge-error">
  Error
</div>
```

```text
badge
│
└─ Component Class

badge-error
│
└─ Modifier
```

이 패턴을 이해하면 daisyUI의 Class 이름을 훨씬 쉽게 읽을 수 있습니다.

---

# 6. Color Modifier

가장 자주 사용하는 Modifier 중 하나가 **Color Modifier**입니다.

Button을 예로 들어보겠습니다.

```html
<button class="btn btn-primary">
  Primary
</button>

<button class="btn btn-secondary">
  Secondary
</button>

<button class="btn btn-accent">
  Accent
</button>

<button class="btn btn-neutral">
  Neutral
</button>
```

또한 의미 기반 상태 색상도 사용할 수 있습니다.

```html
<button class="btn btn-info">
  Info
</button>

<button class="btn btn-success">
  Success
</button>

<button class="btn btn-warning">
  Warning
</button>

<button class="btn btn-error">
  Error
</button>
```

구조는 동일합니다.

```text
btn
 │
 ├─ btn-primary
 ├─ btn-secondary
 ├─ btn-accent
 ├─ btn-neutral
 ├─ btn-info
 ├─ btn-success
 ├─ btn-warning
 └─ btn-error
```

여기서 중요한 점은 이 색상들이 단순히 특정 HEX 값을 의미하는 것이 아니라 **Theme의 Semantic Color와 연결되어 있다는 것**입니다.

```text
Theme
  │
  ├─ primary
  ├─ secondary
  ├─ accent
  ├─ success
  ├─ warning
  └─ error
        │
        ▼
Component Modifier
```

따라서 Theme이 바뀌면 같은:

```html
<button class="btn btn-primary">
```

라도 표현되는 색상이 달라질 수 있습니다.

---

# 7. Size Modifier

Component의 크기를 변경하는 Modifier도 있습니다.

Button을 예로 들면:

```html
<button class="btn btn-xs">
  Xsmall
</button>

<button class="btn btn-sm">
  Small
</button>

<button class="btn btn-md">
  Medium
</button>

<button class="btn btn-lg">
  Large
</button>

<button class="btn btn-xl">
  Xlarge
</button>
```

개념적으로:

```text
작음                               큼

btn-xs
   ↓
btn-sm
   ↓
btn-md
   ↓
btn-lg
   ↓
btn-xl
```

처럼 생각할 수 있습니다.

Size Modifier는 Button에만 있는 개념이 아닙니다.

Component에 따라 다음과 같은 형태가 존재할 수 있습니다.

```text
input-sm
input-lg

select-sm
select-lg

badge-sm
badge-lg
```

중요한 것은:

> **모든 Component가 모든 Modifier를 지원하는 것은 아닙니다.**

각 Component가 어떤 Modifier를 지원하는지는 daisyUI 문서에서 확인해야 합니다.

---

# 8. Style Modifier

색상과 크기 외에도 Component의 **시각적 표현 방식**을 변경할 수 있습니다.

Button에서는 대표적으로 다음과 같은 Variant를 사용할 수 있습니다.

```html
<button class="btn btn-primary">
  Primary
</button>

<button class="btn btn-outline btn-primary">
  Outline
</button>

<button class="btn btn-ghost">
  Ghost
</button>

<button class="btn btn-link">
  Link
</button>
```

개념적으로:

```text
btn
 │
 ├─ 기본 Button
 │
 ├─ btn-outline
 │     └─ Outline 스타일
 │
 ├─ btn-ghost
 │     └─ Ghost 스타일
 │
 └─ btn-link
       └─ Link 스타일
```

Color와 Style을 함께 조합할 수도 있습니다.

```html
<button class="btn btn-outline btn-primary">
  저장
</button>
```

이를 분해하면:

```text
btn
│
├─ btn-outline
│      └─ Style
│
└─ btn-primary
       └─ Color
```

입니다.

---

# 9. Shape와 Layout 관련 Modifier

Button에는 모양이나 크기 특성을 바꾸는 Class도 사용할 수 있습니다.

예:

```html
<button class="btn btn-wide">
  Wide Button
</button>
```

```html
<button class="btn btn-square">
  X
</button>
```

```html
<button class="btn btn-circle">
  ♥
</button>
```

개념적으로:

```text
btn-wide
   ↓
넓은 Button

btn-square
   ↓
정사각형 Button

btn-circle
   ↓
원형 Button
```

이런 Modifier 역시 기본 `btn`에 추가하여 사용합니다.

---

# 10. State Modifier

UI Component는 현재 상태를 표현해야 할 때도 있습니다.

예를 들어 Button을 비활성화할 수 있습니다.

```html
<button class="btn" disabled>
  저장
</button>
```

여기서 `disabled`는 daisyUI Class가 아니라 **HTML 속성(attribute)**입니다.

이 차이가 중요합니다.

```text
class="btn"
     ↓
daisyUI

disabled
     ↓
HTML attribute
```

Component에 따라 상태를 표현하는 Class가 제공되기도 합니다.

따라서 상태를 볼 때는:

```text
HTML 자체의 상태인가?

또는

daisyUI가 제공하는 State Class인가?
```

를 구분해야 합니다.

---

# 11. Modifier를 여러 개 조합하기

이제 다음 코드를 읽어보겠습니다.

```html
<button
  class="btn btn-primary btn-outline btn-lg btn-wide"
>
  저장
</button>
```

처음 보면 Class가 많아 보이지만 역할별로 나누면 간단합니다.

```text
btn
│
├─ Component
│
├─ btn-primary
│      └─ Color
│
├─ btn-outline
│      └─ Style
│
├─ btn-lg
│      └─ Size
│
└─ btn-wide
       └─ Shape / Size characteristic
```

즉 daisyUI 코드는 다음처럼 읽으면 됩니다.

```text
무엇인가?
→ Button

어떤 색인가?
→ Primary

어떤 스타일인가?
→ Outline

어떤 크기인가?
→ Large

어떤 형태인가?
→ Wide
```

---

# 12. Class 순서가 의미를 결정하는가?

다음 두 코드를 보겠습니다.

```html
<button class="btn btn-primary btn-lg">
```

```html
<button class="btn-lg btn-primary btn">
```

HTML의 `class` 속성 안에서 Class 이름을 적는 순서 자체가 일반적으로 **“먼저 적용하고 나중에 덮어쓴다”라는 CSS 규칙을 직접 결정하는 것은 아닙니다.**

실제 우선순위는 생성된 CSS의 규칙, specificity, cascade 등에 의해 결정됩니다.

하지만 강의와 실무에서는 **사람이 읽기 좋은 일정한 순서**를 사용하는 것을 추천합니다.

예:

```text
Component
   ↓
Color / Style
   ↓
Size / Shape
   ↓
State
   ↓
Tailwind Utility
```

예:

```html
<button
  class="btn btn-primary btn-outline btn-lg w-full mt-4"
>
```

이렇게 작성하면 코드를 읽기가 쉽습니다.

---

# 13. Tailwind CSS Utility Class와 함께 사용하기

이제 가장 중요한 실전 패턴입니다.

```html
<button
  class="btn btn-primary w-full mt-4"
>
  로그인
</button>
```

여기에는 두 시스템의 Class가 섞여 있습니다.

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

역할을 분리하면:

```text
daisyUI
────────────────
Component의 공통 디자인

Tailwind CSS
────────────────
레이아웃과 세부 스타일 조정
```

입니다.

---

# 14. Card에서도 같은 원리가 적용된다

Button만 특별한 것이 아닙니다.

다음 Card를 보겠습니다.

```html
<div class="card bg-base-100 w-96 shadow-xl">
  <div class="card-body">
    <h2 class="card-title">
      Product
    </h2>

    <p>
      Mechanical Keyboard
    </p>

    <div class="card-actions justify-end">
      <button class="btn btn-primary">
        구매하기
      </button>
    </div>
  </div>
</div>
```

Class를 구분해 보면:

```text
daisyUI
────────────────
card
card-body
card-title
card-actions
btn
btn-primary
bg-base-100


Tailwind CSS
────────────────
w-96
shadow-xl
justify-end
```

즉 실제 프로젝트에서는 daisyUI와 Tailwind CSS를 자연스럽게 섞어서 사용합니다.

---

# 15. Input에서도 같은 패턴

```html
<input
  type="email"
  placeholder="Email"
  class="input w-full"
/>
```

여기서:

```text
input
   ↓
daisyUI Component Class

w-full
   ↓
Tailwind CSS Utility Class
```

크기를 변경하고 싶다면 Component가 지원하는 Modifier를 추가할 수 있습니다.

```html
<input
  type="email"
  class="input input-lg w-full"
/>
```

분해하면:

```text
input
│
├─ Component
│
├─ input-lg
│     └─ Size
│
└─ w-full
      └─ Tailwind Utility
```

---

# 16. Alert에서도 같은 규칙

```html
<div class="alert alert-success mt-4">
  저장되었습니다.
</div>
```

분해하면:

```text
alert
│
├─ Component
│
├─ alert-success
│      └─ Color / Semantic State
│
└─ mt-4
       └─ Tailwind Utility
```

즉 Button에서 배운 원리가 거의 그대로 반복됩니다.

---

# 17. daisyUI Class 이름을 읽는 방법

처음 보는 코드가 있다고 하겠습니다.

```html
<button
  class="btn btn-error btn-outline btn-sm w-full"
>
  삭제
</button>
```

한 번에 읽으려고 하지 말고 다음 순서로 읽습니다.

```text
① Component 찾기

btn
↓
Button


② Color 찾기

btn-error
↓
Error


③ Style 찾기

btn-outline
↓
Outline


④ Size 찾기

btn-sm
↓
Small


⑤ Tailwind Utility 찾기

w-full
↓
Full Width
```

결과적으로:

> **전체 너비를 사용하는 Small 크기의 Error Outline Button**

이라고 이해할 수 있습니다.

---

# 18. Modifier는 Component마다 다르다

이 부분은 매우 중요합니다.

다음과 같이 생각하면 안 됩니다.

```text
btn-primary가 있으니까

card-primary
input-primary
navbar-primary

도 모두 있겠지?
```

반드시 그렇지는 않습니다.

daisyUI의 각 Component는 자신이 지원하는 Class와 Modifier가 있습니다.

예를 들어 개념적으로:

```text
Button
├─ Color
├─ Size
├─ Style
└─ Shape

Input
├─ Color
└─ Size

Alert
├─ Color
└─ Style

Badge
├─ Color
├─ Size
└─ Style
```

처럼 Component마다 지원 범위가 다를 수 있습니다.

따라서 새로운 Component를 사용할 때는 공식 문서에서 해당 Component의 Class 목록을 확인하는 습관이 중요합니다.

---

# 19. daisyUI Class와 Tailwind Utility를 구분하는 방법

처음에는 다음 코드가 헷갈릴 수 있습니다.

```html
<div
  class="
    card
    bg-base-100
    shadow-xl
    w-96
    mx-auto
    mt-10
  "
>
```

어떤 것이 daisyUI이고 어떤 것이 Tailwind CSS일까요?

대표적으로:

```text
card
bg-base-100
    ↓
daisyUI의 Component/Theme 관련 Class


shadow-xl
w-96
mx-auto
mt-10
    ↓
Tailwind CSS Utility
```

하지만 모든 Class를 이름만 보고 완벽하게 구분하려고 할 필요는 없습니다.

중요한 것은:

> **Component의 구조와 Variant는 daisyUI 문서를 확인하고, 레이아웃 및 세부 조정에는 Tailwind Utility를 적극적으로 활용한다.**

는 개발 패턴을 이해하는 것입니다.

---

# 20. React에서는 `className`

지금까지 HTML 중심으로 설명했지만 React에서는 `class` 대신 `className`을 사용합니다.

HTML:

```html
<button class="btn btn-primary">
  저장
</button>
```

React:

```jsx
<button className="btn btn-primary">
  저장
</button>
```

Modifier를 추가하는 방법은 같습니다.

```jsx
<button
  className="btn btn-primary btn-lg w-full"
>
  저장
</button>
```

구조:

```text
btn
   → Component

btn-primary
   → Color

btn-lg
   → Size

w-full
   → Tailwind Utility
```

---

# 21. React에서 조건부 Class 사용하기

실제 React 애플리케이션에서는 상태에 따라 Modifier를 변경하는 경우가 많습니다.

예:

```jsx
function SaveButton({ saving }) {
  return (
    <button
      className="btn btn-primary"
      disabled={saving}
    >
      {saving ? '저장 중...' : '저장'}
    </button>
  )
}
```

여기서는:

```text
daisyUI
→ Button 디자인

React
→ saving 상태 관리

HTML
→ disabled 상태
```

로 역할이 나뉩니다.

조금 더 발전시키면 조건에 따라 Class를 변경할 수도 있습니다.

```jsx
function StatusBadge({ success }) {
  return (
    <span
      className={
        success
          ? 'badge badge-success'
          : 'badge badge-error'
      }
    >
      {success ? 'Success' : 'Error'}
    </span>
  )
}
```

흐름은:

```text
React State / Props
        ↓
조건 판단
        ↓
Modifier 선택
        ↓
badge-success
또는
badge-error
        ↓
UI 변경
```

입니다.

---

# 22. 실전 예제 — 로그인 Form

지금까지 배운 내용을 하나로 합쳐보겠습니다.

```jsx
function LoginForm() {
  return (
    <div className="card bg-base-100 w-full max-w-md shadow-xl">
      <div className="card-body">

        <h2 className="card-title">
          로그인
        </h2>

        <input
          type="email"
          placeholder="Email"
          className="input w-full"
        />

        <input
          type="password"
          placeholder="Password"
          className="input w-full"
        />

        <button
          className="btn btn-primary w-full mt-4"
        >
          로그인
        </button>

      </div>
    </div>
  )
}
```

이 코드를 역할별로 나누면:

```text
Component
────────────────
card
card-body
card-title
input
btn


Modifier / Theme
────────────────
btn-primary
bg-base-100


Tailwind Utility
────────────────
w-full
max-w-md
shadow-xl
mt-4
```

이것이 실제 daisyUI 코드의 기본적인 모습입니다.

---

# 23. Class를 작성할 때 추천 순서

강의에서는 다음 규칙으로 통일하면 코드를 읽기 편합니다.

```text
① Component
② Color
③ Style
④ Size / Shape
⑤ State
⑥ Tailwind Utility
```

예:

```html
<button
  class="
    btn
    btn-primary
    btn-outline
    btn-lg
    w-full
    mt-4
  "
>
  저장
</button>
```

한 줄로 작성하면:

```html
<button class="btn btn-primary btn-outline btn-lg w-full mt-4">
  저장
</button>
```

이 순서는 CSS의 동작을 위한 절대적인 문법 규칙이 아니라 **코드의 가독성과 일관성을 위한 작성 규칙**입니다.

---

# 24. 전체 구조 한눈에 보기

```text
                    HTML Element
                         │
                         ▼
                  Component Class
                         │
                        btn
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
        Color          Style           Size
     btn-primary    btn-outline       btn-lg
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                Tailwind Utility
                    w-full
                     mt-4
                         │
                         ▼
                    Final UI
```

코드로 표현하면:

```html
<button
  class="btn btn-primary btn-outline btn-lg w-full mt-4"
>
  저장
</button>
```

---

# 25. PART 2 핵심 정리

daisyUI Class를 볼 때 다음 순서로 생각하면 됩니다.

```text
1. 어떤 Component인가?
        ↓
      btn

2. 어떤 Color인가?
        ↓
   btn-primary

3. 어떤 Style인가?
        ↓
   btn-outline

4. 어떤 Size인가?
        ↓
     btn-lg

5. 추가적인 Layout 조정은?
        ↓
   w-full mt-4
        ↓
 Tailwind CSS
```

결국 daisyUI의 기본 패턴은 다음과 같습니다.

```text
Component Class
       +
   Modifier
       +
Tailwind Utility
       ↓
    Final UI
```

예:

```html
<button class="btn btn-primary btn-lg w-full">
  저장
</button>
```

여기서 가장 중요한 것은 Class 이름을 외우는 것이 아닙니다.

> **Component Class를 중심으로 Color, Style, Size 등의 Modifier를 조합하고, 필요한 세부 레이아웃과 스타일은 Tailwind CSS Utility Class로 조정한다.**

이 원리를 이해하면 Button뿐 아니라 Input, Badge, Alert, Card 등 다른 daisyUI Component도 훨씬 쉽게 배울 수 있습니다.

---

# 다음 PART

## PART 3. daisyUI Color와 Theme 시스템

다음 PART에서는 PART 2에서 사용한:

```text
primary
secondary
accent
neutral
info
success
warning
error
```

가 실제로 어디에서 오는지 살펴봅니다.

핵심 흐름은 다음과 같습니다.

```text
Theme
   ↓
Semantic Color
   ↓
primary / secondary / accent / ...
   ↓
Component Modifier
   ↓
btn-primary
badge-secondary
alert-success
```

즉 다음 PART에서는 **daisyUI의 Component와 Theme이 어떻게 연결되는지**를 자세히 학습합니다.
