# PART 9. Theme & Design Token

## Tailwind CSS v4로 프로젝트의 Design System 만들기

지금까지 우리는 Tailwind가 제공하는 Utility를 사용했습니다.

예:

```jsx
bg-blue-600
text-gray-900
rounded-xl
p-4
max-w-7xl
```

하지만 실제 프로젝트에서는 다음과 같은 문제가 생깁니다.

```text
Button A
→ bg-blue-600

Button B
→ bg-indigo-600

Button C
→ bg-sky-600

Card A
→ rounded-lg

Card B
→ rounded-xl

Card C
→ rounded-2xl
```

모두 기술적으로는 동작하지만 프로젝트의 **Design Rule이 불명확**합니다.

따라서 다음 단계가 필요합니다.

```text
Utility
   ↓
Design Token
   ↓
Theme
   ↓
Design System
```

이번 PART에서는 Tailwind CSS v4의 `@theme`을 중심으로 프로젝트 전체의 디자인 규칙을 만드는 방법을 학습합니다.

---

# 1. Design Token이란?

Design Token은 디자인에서 반복되는 값을 **이름을 가진 규칙**으로 정의한 것입니다.

예를 들어 프로젝트에서 브랜드 색상을 다음처럼 결정했다고 하겠습니다.

```text
Primary Color
→ #2563eb

Success Color
→ #16a34a

Danger Color
→ #dc2626
```

이 값들을 Component마다 직접 작성하면:

```jsx
<button className="bg-[#2563eb]">
```

```jsx
<a className="text-[#2563eb]">
```

```jsx
<div className="border-[#2563eb]">
```

처럼 동일한 값이 반복됩니다.

대신:

```text
#2563eb
   ↓
Primary
```

라는 의미를 부여할 수 있습니다.

즉:

```text
Raw Value
      ↓
Design Token
      ↓
Semantic Meaning
```

입니다.

---

# 2. 왜 Design Token이 필요한가?

Design Token이 없으면:

```text
Component
├─ #2563eb
├─ #1d4ed8
├─ 12px
├─ 14px
├─ 18px
├─ 10px radius
└─ ...
```

같은 값들이 코드 곳곳에 흩어집니다.

Design Token을 사용하면:

```text
Design System
│
├─ Color
│   ├─ Primary
│   ├─ Secondary
│   ├─ Success
│   └─ Danger
│
├─ Typography
│   ├─ Body
│   ├─ Heading
│   └─ Caption
│
├─ Spacing
│
├─ Radius
│
└─ Breakpoint
```

형태로 관리할 수 있습니다.

장점:

```text
일관성
유지보수성
재사용성
Theme 변경 용이
Designer와 Developer 간 공통 언어
```

---

# 3. Tailwind CSS v4의 CSS-first 설정

Tailwind CSS v4에서는 CSS 파일에서 직접 Tailwind를 import하고 theme을 정의하는 방식이 핵심입니다.

기본:

```css
@import "tailwindcss";
```

그리고:

```css
@theme {
  ...
}
```

을 사용합니다.

예:

```css
@import "tailwindcss";

@theme {
  --color-brand-500: #3b82f6;
}
```

그러면 Tailwind Utility에서:

```jsx
<div className="bg-brand-500">
```

처럼 사용할 수 있습니다.

Tailwind 공식 문서에서도 color palette, spacing scale, typography, breakpoint 등을 `@theme`으로 커스터마이징하는 방식을 안내합니다.

---

# 4. `@theme`의 핵심 원리

예:

```css
@theme {
  --color-brand-500: #3b82f6;
}
```

구조:

```text
@theme
  │
  ▼

--color-brand-500
       │
       ▼

Theme Variable
       │
       ▼

bg-brand-500
text-brand-500
border-brand-500
ring-brand-500
```

즉 Tailwind는 theme variable의 **namespace**를 기반으로 관련 Utility를 생성합니다.

```text
--color-*
→ Color Utility

--font-*
→ Font Family

--breakpoint-*
→ Responsive Variant

--radius-*
→ Border Radius

--spacing-*
→ Spacing 기반 Utility
```

이 개념이 PART 9의 핵심입니다.

---

# 5. Custom Color 만들기

예:

```css
@theme {
  --color-brand-50: #eff6ff;
  --color-brand-100: #dbeafe;
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;
}
```

이제:

```jsx
<button
  className="
    bg-brand-600
    text-white

    hover:bg-brand-700
  "
>
  구매하기
</button>
```

처럼 사용할 수 있습니다.

또:

```jsx
<p className="text-brand-600">
```

```jsx
<div className="border-brand-500">
```

```jsx
<input className="focus-visible:ring-brand-500">
```

처럼 여러 CSS property에 같은 색상 token을 활용할 수 있습니다.

---

# 6. Color Palette를 만드는 이유

단일 색상 하나만 정의하면:

```text
brand
```

상태별 표현이 어렵습니다.

실제 UI에서는 보통:

```text
brand-50
brand-100
brand-500
brand-600
brand-700
```

처럼 단계가 필요합니다.

예:

```text
Background
→ brand-50

Border
→ brand-200

Primary
→ brand-600

Hover
→ brand-700
```

이렇게 하면 Component 전체에서 일관된 hierarchy를 만들 수 있습니다.

---

# 7. Raw Token과 Semantic Token

여기서 한 단계 더 나아가야 합니다.

예:

```text
blue-600
```

은 색상 값 자체의 이름입니다.

하지만:

```text
primary
```

는 **역할**의 이름입니다.

비교:

```text
Raw Token

blue-50
blue-100
blue-600
blue-700


Semantic Token

surface
primary
primary-hover
danger
muted
```

실제 Design System에서는 semantic token이 매우 중요합니다.

---

# 8. Semantic Color 개념

예를 들어:

```text
Primary
→ 브랜드의 주요 Action

Danger
→ 삭제/오류

Success
→ 성공

Surface
→ Card / Panel Background

Muted
→ 보조 정보
```

처럼 역할을 정의합니다.

개념:

```text
Raw Color
   ↓
Semantic Role
   ↓
Component
```

예:

```text
blue-600
   ↓
primary
   ↓
Button
Link
Focus Ring
```

이렇게 하면 나중에 브랜드 색상이 바뀌어도 Component 의미는 그대로 유지됩니다.

---

# 9. Typography Theme

Font Family도 Theme으로 정의할 수 있습니다.

```css
@theme {
  --font-sans: "Pretendard", sans-serif;
  --font-display: "Inter", sans-serif;
}
```

이후:

```jsx
<body className="font-sans">
```

```jsx
<h1 className="font-display">
```

처럼 사용할 수 있습니다.

공식 Tailwind v4 문서에서도 `--font-*` namespace를 이용한 theme customization을 지원합니다.

---

# 10. Font Token 설계

예:

```text
Body
→ font-sans

Code
→ font-mono

Marketing Heading
→ font-display
```

중요한 것은 Component마다:

```jsx
style={{
  fontFamily: "..."
}}
```

를 반복하는 것이 아니라:

```text
Theme
  ↓
Font Token
  ↓
Utility
```

로 연결하는 것입니다.

---

# 11. Spacing Theme

Spacing도 Design System의 핵심입니다.

예를 들어 프로젝트에서:

```text
4
8
12
16
24
32
```

같은 일정한 spacing scale을 사용한다고 하겠습니다.

이 spacing rule은:

```text
Padding
Margin
Gap
Width
Height
```

등 다양한 곳에서 반복됩니다.

즉:

```text
Spacing Token
      ↓
p-*
m-*
gap-*
w-*
h-*
```

의 관계가 만들어집니다.

Tailwind의 theme variable은 이러한 spacing 기반 Utility들과 연결됩니다.

---

# 12. Radius Token

Card와 Button의 radius도 일관성을 가져야 합니다.

예:

```text
Button
→ rounded-lg

Input
→ rounded-lg

Card
→ rounded-xl

Modal
→ rounded-2xl
```

Theme을 이용하면 radius scale 자체를 프로젝트 규칙으로 관리할 수 있습니다.

개념:

```text
Radius Scale
│
├─ sm
├─ md
├─ lg
├─ xl
└─ 2xl
```

그리고 Component는 이 규칙 안에서 선택합니다.

---

# 13. Breakpoint 커스터마이징

Tailwind v4에서는 breakpoint 역시 theme variable로 정의할 수 있습니다.

예:

```css
@theme {
  --breakpoint-sm: 40rem;
  --breakpoint-md: 48rem;
  --breakpoint-lg: 64rem;
}
```

그리고:

```jsx
<div className="md:grid-cols-2">
```

처럼 사용합니다.

`--breakpoint-*` theme variable은 Responsive Variant와 연결됩니다.

중요:

> Breakpoint는 기기 이름을 의미하는 것이 아니라 **Layout이 바뀌어야 하는 min-width 조건**입니다.

---

# 14. 새로운 Breakpoint 추가

예:

```css
@theme {
  --breakpoint-3xl: 120rem;
}
```

이후:

```jsx
<div className="3xl:grid-cols-6">
```

같은 Variant를 사용할 수 있습니다.

즉:

```text
Theme Variable
--breakpoint-3xl
        ↓
Variant
3xl:
```

입니다.

---

# 15. 디폴트 Theme 확장

일반적으로 기존 Tailwind theme을 그대로 사용하면서 프로젝트 값만 추가할 수 있습니다.

예:

```css
@theme {
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;

  --font-display: "Inter", sans-serif;

  --breakpoint-3xl: 120rem;
}
```

그러면 기존:

```text
bg-blue-500
text-gray-900
rounded-xl
```

도 사용할 수 있고:

```text
bg-brand-500
font-display
3xl:grid-cols-6
```

도 사용할 수 있습니다.

---

# 16. 특정 Theme Namespace 완전히 교체하기

Tailwind v4에서는 namespace를 `initial`로 설정해서 해당 디폴트 theme을 제거하고 직접 정의할 수도 있습니다.

예:

```css
@theme {
  --color-*: initial;

  --color-white: #ffffff;
  --color-brand: #2563eb;
  --color-danger: #dc2626;
}
```

이 경우 디폴트 color namespace 대신 직접 정의한 색상만 사용하도록 제한할 수 있습니다.

이 방식은 강력하지만 프로젝트 전체에 영향을 주므로 신중하게 사용해야 합니다.

---

# 17. 전체 Theme을 직접 정의하기

Theme 전체를 직접 관리하려는 경우 global theme namespace를 초기화한 뒤 필요한 token을 직접 정의할 수도 있습니다.

개념적으로:

```text
Tailwind Default Theme
        ↓
     제거
        ↓
Project Theme
        ↓
직접 정의
```

이 방식은 강한 Design System 제어가 필요한 프로젝트에서 고려할 수 있습니다.

초급 프로젝트에서는 보통 **기존 theme을 확장하는 방식**부터 시작하는 것이 좋습니다.

---

# 18. Theme Variable과 CSS Variable의 관계

`@theme`에서 정의한 값은 CSS custom property 기반입니다.

예:

```css
@theme {
  --color-brand-600: #2563eb;
}
```

즉 브라우저 관점에서는:

```text
CSS Variable
      +
Tailwind Theme Namespace
      ↓
Utility 생성
```

입니다.

이 부분이 Tailwind CSS v4의 CSS-first 철학에서 매우 중요합니다.

---

# 19. Theme Variable 직접 사용하기

Theme variable은 CSS에서도 활용할 수 있습니다.

예:

```css
.custom-card {
  border-color: var(--color-brand-600);
}
```

즉 Tailwind Utility만 사용하는 것이 아니라:

```text
Theme Token
    │
    ├─ Tailwind Utility
    └─ Custom CSS
```

양쪽에서 공유할 수 있습니다.

---

# 20. Arbitrary Value vs Theme Token

PART 7에서 다음을 배웠습니다.

```jsx
w-[420px]
bg-[#2563eb]
mt-[18px]
```

이런 arbitrary value는 필요할 때 유용합니다.

하지만 같은 값이 반복된다면:

```text
arbitrary value 반복
        ↓
Token 후보
        ↓
Theme 등록
```

을 생각해야 합니다.

예:

```jsx
bg-[#2563eb]
bg-[#2563eb]
bg-[#2563eb]
```

보다:

```css
@theme {
  --color-brand-600: #2563eb;
}
```

후:

```jsx
bg-brand-600
```

가 더 일관적입니다.

---

# 21. 언제 Arbitrary Value를 사용할까?

좋은 기준:

```text
한 번만 필요한 특수 값
        ↓
Arbitrary Value


프로젝트에서 반복되는 값
        ↓
Theme Token
```

예:

```jsx
top-[117px]
```

처럼 매우 특수한 위치값은 arbitrary value가 자연스럽습니다.

Tailwind 공식 문서도 arbitrary value를 theme의 제약을 벗어나야 하는 경우에 사용할 수 있도록 제공합니다.

---

# 22. Custom Utility

Tailwind에 없는 Utility가 프로젝트에서 반복적으로 필요할 수도 있습니다.

Tailwind CSS v4에서는 `@utility`를 사용해서 Custom Utility를 등록할 수 있습니다.

예:

```css
@utility content-auto {
  content-visibility: auto;
}
```

이제:

```jsx
<div className="content-auto">
```

로 사용할 수 있습니다.

`@utility`로 등록한 Utility는 `hover:`, `focus:`, `lg:` 같은 Variant와도 조합할 수 있습니다.

---

# 23. Custom Utility + Variant

예:

```jsx
<div className="hover:content-auto">
```

또는:

```jsx
<div className="lg:content-auto">
```

처럼 사용할 수 있습니다.

즉:

```text
Custom Utility
       +
Tailwind Variant
       ↓
일반 Utility와 동일한 사용 경험
```

을 제공합니다.

---

# 24. `@layer components`

복잡한 Component 스타일을 CSS class로 묶고 싶을 수도 있습니다.

예:

```css
@layer components {
  .card {
    background-color: white;
    border-radius: var(--radius-xl);
    padding: 1.5rem;
  }
}
```

사용:

```jsx
<div className="card">
```

Tailwind 공식 문서에서도 `@layer components`를 이용해 Card, Button 같은 component class를 정의할 수 있음을 설명합니다. 다만 이런 abstraction을 과도하게 만들 필요는 없다고 안내합니다.

---

# 25. Utility-first와 Component Class의 균형

다음처럼:

```jsx
<button
  className="
    rounded-lg
    bg-brand-600
    px-4
    py-2
    text-white
  "
>
```

Utility를 직접 조합하는 것이 Tailwind의 기본 방식입니다.

하지만 완전히 반복되는 패턴이 많다면:

```text
React Component
CSS Component Layer
Custom Utility
```

등을 고려할 수 있습니다.

핵심은:

> **중복이 있다는 이유만으로 모든 Utility 조합을 CSS class 하나로 숨기지 않습니다.**

---

# 26. React Component가 더 자연스러운 경우

예:

```jsx
function Button({
  children,
  variant = 'primary',
}) {
  ...
}
```

UI 구조와 behavior까지 함께 재사용한다면 CSS `.btn`보다 **React Component abstraction**이 더 자연스러울 수 있습니다.

```text
Visual style만 반복
→ Utility / Custom CSS 고려

Markup + Behavior + Style 반복
→ React Component 고려
```

입니다.

---

# 27. Semantic Component Variant

예:

```jsx
<Button variant="primary">
  저장
</Button>

<Button variant="danger">
  삭제
</Button>
```

내부:

```jsx
const variants = {
  primary:
    'bg-brand-600 text-white hover:bg-brand-700',

  danger:
    'bg-red-600 text-white hover:bg-red-700',
}
```

이렇게 하면:

```text
Design Token
      ↓
Utility
      ↓
Component Variant
      ↓
Application UI
```

구조가 만들어집니다.

---

# 28. Theme과 State Variant

Theme Token도 기존 State Variant와 동일하게 사용할 수 있습니다.

예:

```jsx
<button
  className="
    bg-brand-600
    text-white

    hover:bg-brand-700

    focus-visible:ring-2
    focus-visible:ring-brand-500
  "
>
```

즉:

```text
Theme
+
State Variant
+
Utility
```

가 그대로 조합됩니다.

---

# 29. Dark Mode와 Theme

Tailwind에서는 `dark:` Variant를 이용해 dark color scheme용 Utility를 적용할 수 있습니다.

예:

```jsx
<div
  className="
    bg-white
    text-gray-900

    dark:bg-gray-900
    dark:text-white
  "
>
```

하지만 실제 Design System에서는:

```text
Light Theme
   ↓
Surface / Text / Border

Dark Theme
   ↓
Surface / Text / Border
```

전체 semantic role을 함께 설계해야 합니다.

---

# 30. Dark Mode는 색 반전이 아니다

잘못된 생각:

```text
white ↔ black
```

만 바꾸면 Dark Mode 완성.

실제로는:

```text
Background
Surface
Elevated Surface
Border
Primary Text
Secondary Text
Muted Text
Focus
Hover
Disabled
```

등을 모두 다시 확인해야 합니다.

특히 Contrast와 readability가 중요합니다.

---

# 31. Custom Variant

Tailwind CSS v4에서는 `@custom-variant`를 이용해 프로젝트 자체 Variant를 정의할 수도 있습니다.

예:

```css
@custom-variant theme-midnight (
  &:where([data-theme="midnight"] *)
);
```

이후:

```jsx
<div className="theme-midnight:bg-black">
```

같은 형태를 사용할 수 있습니다.

이 기능을 이용하면 특정 data attribute 기반 theme 같은 패턴도 구성할 수 있습니다.

---

# 32. Theme Switching 구조

예:

```html
<html data-theme="midnight">
```

그리고:

```text
DOM Attribute
data-theme="midnight"
        ↓
Custom Variant
        ↓
theme-midnight:
        ↓
Utility 적용
```

이렇게 연결할 수 있습니다.

다만 초급 프로젝트에서는 우선 `dark:`부터 이해한 후 확장하는 것이 좋습니다.

---

# 33. Design Token Layer를 나누기

실무에서는 Token을 한 단계로만 구성하지 않을 수도 있습니다.

예:

```text
Primitive Token
│
├─ blue-600
├─ gray-50
├─ gray-900
└─ red-600

        ↓

Semantic Token
│
├─ primary
├─ surface
├─ foreground
├─ muted
└─ danger

        ↓

Component Token
│
├─ button-primary-bg
├─ card-bg
└─ input-border
```

작은 프로젝트에서는 이렇게까지 나눌 필요가 없지만, 규모가 커질수록 역할 분리가 중요해질 수 있습니다.

---

# 34. Primitive Token

Primitive Token은 값 그 자체에 가까운 이름입니다.

```text
blue-600
gray-100
red-500
spacing-4
radius-lg
```

장점:

```text
재사용 가능
Color Scale 구성 쉬움
```

하지만 Component의 의미는 잘 드러나지 않습니다.

---

# 35. Semantic Token

Semantic Token은 역할을 표현합니다.

```text
primary
danger
surface
foreground
muted
border
```

이 방식은 Theme 변경에서 특히 강합니다.

예:

```text
Light
primary → blue-600

Dark
primary → blue-400
```

Component에서는 계속:

```text
primary
```

라는 의미만 사용합니다.

---

# 36. Design Token의 핵심 목적

Token의 목적은 단순히 변수로 값을 치환하는 것이 아닙니다.

```text
Bad

#2563eb
   ↓
--blue


Better

#2563eb
   ↓
--primary
   ↓
Primary Action
```

즉:

> **값에 의미를 부여하는 것**

이 중요합니다.

---

# 37. 프로젝트 Theme 예제

예를 들어 쇼핑몰 프로젝트 Theme을 만든다고 하겠습니다.

```css
@import "tailwindcss";

@theme {
  --color-brand-50: #eff6ff;
  --color-brand-100: #dbeafe;
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;

  --color-danger-500: #ef4444;
  --color-danger-600: #dc2626;

  --font-sans: "Pretendard", sans-serif;

  --breakpoint-3xl: 120rem;
}
```

이제 React에서는:

```jsx
<button
  className="
    rounded-lg
    bg-brand-600
    px-4
    py-2
    font-semibold
    text-white

    hover:bg-brand-700
  "
>
  구매하기
</button>
```

처럼 사용합니다.

---

# 38. Theme 적용 전

```jsx
<button
  className="
    bg-[#2563eb]
    hover:bg-[#1d4ed8]
    text-white
  "
>
```

문제:

```text
색상 의미를 알기 어려움
반복
일관성 관리 어려움
브랜드 변경 어려움
```

---

# 39. Theme 적용 후

```jsx
<button
  className="
    bg-brand-600
    hover:bg-brand-700
    text-white
  "
>
```

개선:

```text
의미가 명확
Theme에서 일괄 관리
재사용 가능
Visual consistency 증가
```

---

# 40. Theme Token을 어디까지 만들까?

모든 값에 Token을 만들 필요는 없습니다.

잘못된 예:

```text
--card-left-padding
--card-title-margin-top
--product-name-line-height
--button-icon-gap
```

프로젝트 전용 token이 지나치게 많아지면 오히려 복잡해집니다.

좋은 기준:

> **여러 곳에서 반복되고, 프로젝트의 시각적 규칙을 형성하는 값인가?**

그렇다면 Token 후보입니다.

---

# 41. Token 후보 판단 기준

```text
반복되는가?
      │
     YES
      ↓

디자인 의미가 있는가?
      │
     YES
      ↓

프로젝트 전체에서
일관되게 관리할 필요가 있는가?
      │
     YES
      ↓

Theme Token
```

반대로:

```text
한 번만 사용하는 특수 값
      ↓
Arbitrary Value
```

일 수 있습니다.

---

# 42. Custom CSS는 실패가 아니다

Tailwind를 사용한다고 해서 모든 CSS를 Utility로만 작성해야 하는 것은 아닙니다.

공식 문서도 필요한 경우 일반 CSS, base styles, component layer, custom utility 등을 사용할 수 있도록 지원합니다.

따라서:

```text
Tailwind Utility
+
Custom CSS
```

는 정상적인 조합입니다.

중요한 것은 **어떤 방식이 의도를 가장 명확하게 표현하는가**입니다.

---

# 43. `@layer base`

HTML element의 공통 기본 스타일을 정의할 수도 있습니다.

예:

```css
@layer base {
  h1 {
    font-size: var(--text-2xl);
  }

  h2 {
    font-size: var(--text-xl);
  }
}
```

Tailwind 공식 문서에서도 `@layer base`를 이용한 기본 element style 확장을 지원합니다.

다만 모든 element를 무조건 global style로 고정하기보다는 프로젝트 정책에 맞춰 사용해야 합니다.

---

# 44. `@layer components`

예:

```css
@layer components {
  .card {
    background: white;
    border-radius: var(--radius-xl);
  }
}
```

사용:

```jsx
<div className="card">
```

Utility로 override할 수도 있습니다.

```jsx
<div className="card rounded-none">
```

이런 Component Layer는 third-party component styling이나 반복되는 복잡한 visual class에 사용할 수 있습니다.

---

# 45. `@utility`

Tailwind v4에서 custom utility를 등록할 때 사용합니다.

```css
@utility scrollbar-hidden {
  &::-webkit-scrollbar {
    display: none;
  }
}
```

사용:

```jsx
<div className="scrollbar-hidden">
```

그리고:

```jsx
<div className="lg:scrollbar-hidden">
```

같이 Variant와 조합할 수 있습니다.

---

# 46. `@variant`

Custom CSS 안에서 Tailwind Variant를 사용할 수도 있습니다.

예:

```css
.my-element {
  background: white;

  @variant dark {
    background: black;
  }
}
```

공식 Tailwind v4 문서에서 `@variant`는 CSS 안에서 built-in variant를 적용하는 기능으로 제공됩니다.

---

# 47. Design System 전체 구조

```text
Design System
│
├─ Theme
│   ├─ Color
│   ├─ Typography
│   ├─ Spacing
│   ├─ Radius
│   └─ Breakpoint
│
├─ Tailwind Utility
│
├─ Variant
│   ├─ Responsive
│   ├─ State
│   └─ Theme
│
├─ Component
│   ├─ Button
│   ├─ Input
│   ├─ Card
│   └─ Badge
│
└─ Application
```

모든 계층이 연결됩니다.

---

# 48. Button Design System 예제

Theme:

```css
@theme {
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;
}
```

Component:

```jsx
function PrimaryButton({ children }) {
  return (
    <button
      className="
        rounded-lg
        bg-brand-600
        px-4
        py-2

        font-semibold
        text-white

        hover:bg-brand-700

        focus-visible:outline-none
        focus-visible:ring-2
        focus-visible:ring-brand-500
      "
    >
      {children}
    </button>
  )
}
```

Application:

```jsx
<PrimaryButton>
  구매하기
</PrimaryButton>
```

구조:

```text
Theme Token
    ↓
Utility
    ↓
React Component
    ↓
Application
```

---

# 49. Product Card Design System

Theme:

```text
Color
Radius
Typography
Spacing
```

Component:

```jsx
<ProductCard />
```

Application:

```text
Product Grid
```

이전 PART에서 각각 따로 학습한 요소들이 이제 하나의 시스템으로 연결됩니다.

---

# 50. PART 9 핵심 정리

## Tailwind CSS v4 Theme

```text
@theme
│
├─ --color-*
├─ --font-*
├─ --breakpoint-*
├─ --radius-*
└─ 기타 Theme Variable
```

Theme variable은 관련 Tailwind Utility와 Variant를 생성하거나 확장하는 기반이 됩니다.

---

# 51. Customization 수단

```text
Theme Token
→ @theme

한 번 사용하는 특수 값
→ Arbitrary Value

반복되는 Custom Utility
→ @utility

Base Element Style
→ @layer base

Component CSS
→ @layer components

Custom Condition
→ @custom-variant

Custom CSS 내부 Variant
→ @variant
```

Tailwind v4는 이 모든 확장 방법을 CSS 중심으로 제공합니다.

---

# 52. 가장 중요한 판단 기준

```text
값이 한 번만 필요한가?
      ↓
Arbitrary Value


프로젝트 디자인 규칙인가?
      ↓
Theme Token


새로운 반복 Utility인가?
      ↓
@utility


Markup + Behavior까지 반복되는가?
      ↓
React Component
```

이 기준을 가지면 무분별한 abstraction을 피할 수 있습니다.

---

# 53. PART 9 최종 핵심

```text
PART 8

bg-blue-600
text-gray-900
rounded-xl
p-4

          ↓

PART 9

Color
Typography
Spacing
Radius
Breakpoint

          ↓

Design Token

          ↓

@theme

          ↓

Project Design System
```

> **Tailwind의 진짜 확장성은 Utility를 많이 외우는 데 있는 것이 아니라, 프로젝트의 디자인 규칙을 Theme과 Token으로 정의하고 모든 Component가 같은 규칙을 공유하도록 만드는 데 있습니다.**

---

# 다음 PART

## PART 10. 실전 프로젝트 & Best Practices

마지막 PART에서는 지금까지 배운 모든 내용을 하나의 React 프로젝트 구조로 통합합니다.

```text
PART 1~4
Tailwind 기본 + Responsive

PART 5
State Variant

PART 6
Flexbox / Grid

PART 7
Sizing / Spacing

PART 8
Typography / Color / Visual

PART 9
Theme / Design Token

          ↓

PART 10

실전 React UI
+
Component 설계
+
Responsive
+
Design System
+
Best Practices
```

---

# PART 9 이미지 구성 — 총 8장

```text
1/8
Theme & Design Token 전체 개념
+ Raw Value → Token → Utility → Component

2/8
Tailwind CSS v4 @theme
+ Theme Variable namespace
+ Utility 생성 원리

3/8
Custom Color / Font / Radius
+ Brand Theme 만들기

4/8
Breakpoint / Spacing Theme
+ 기존 Theme 확장
+ namespace 교체

5/8
Primitive Token vs Semantic Token
+ 실제 Design System 구조

6/8
Arbitrary Value vs Theme Token
+ @utility
+ Custom CSS

7/8
@layer / @variant / @custom-variant
+ Dark Theme
+ React Component와 연결

8/8
쇼핑몰 Design System 종합 실전
+ Theme
+ Button
+ Input
+ Card
+ Dark Mode 개념
+ PART 9 전체 정리
```
