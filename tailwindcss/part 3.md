# PART 3. 핵심 Utility Classes

## 1. 이번 PART의 목표

PART 2까지 React 프로젝트에서 Tailwind CSS를 사용할 준비를 마쳤습니다.

이제부터 실제 UI를 만들기 위해 가장 많이 사용하는 Utility를 배웁니다.

이번 PART의 핵심은 Utility 이름을 무작정 암기하는 것이 아닙니다.

> **기존 CSS 속성과 Tailwind Utility의 관계를 이해하는 것**

입니다.

예를 들어 CSS에서:

```css
.card {
  padding: 1rem;
  margin-top: 2rem;
  width: 100%;
  background-color: white;
  border-radius: 0.5rem;
}
```

처럼 작성했다면 Tailwind에서는:

```jsx
<div className="
  p-4
  mt-8
  w-full
  bg-white
  rounded-lg
">
```

처럼 표현할 수 있습니다.

전체적인 관계는:

```text
CSS Property
     ↓
Tailwind Utility
     ↓
className에 조합
     ↓
UI
```

입니다.

---

# 2. Utility를 카테고리로 이해하자

Tailwind에는 매우 많은 Utility가 있습니다.

전부 외우려고 하면 어렵습니다.

대신 CSS와 마찬가지로 **카테고리**로 나누어 이해하면 쉽습니다.

```text
Core Utility
│
├─ Spacing
│   ├─ padding
│   ├─ margin
│   └─ gap
│
├─ Sizing
│   ├─ width
│   ├─ height
│   ├─ min-width
│   └─ max-width
│
├─ Typography
│   ├─ font-size
│   ├─ font-weight
│   ├─ line-height
│   └─ text-align
│
├─ Colors
│   ├─ background
│   ├─ text
│   └─ border
│
├─ Borders
│   ├─ width
│   ├─ color
│   └─ radius
│
└─ Effects
    ├─ shadow
    └─ opacity
```

이번 PART에서는 이 여섯 영역을 중심으로 살펴봅니다.

Flexbox와 Grid 같은 Layout은 이후 PART에서 별도로 다룹니다.

---

# 3. Spacing — 여백

UI를 만들 때 가장 많이 사용하는 Utility 중 하나가 Spacing입니다.

CSS에서는 크게:

```text
padding
margin
gap
```

이 있습니다.

Tailwind에서는:

```text
p-
m-
gap-
```

으로 시작합니다.

---

# 4. Padding

CSS:

```css
padding: 1rem;
```

Tailwind:

```html
p-4
```

예:

```jsx
<div className="p-4">
  Content
</div>
```

`p`는 padding을 의미합니다.

방향을 지정할 수도 있습니다.

```text
p-4     모든 방향

pt-4    top
pr-4    right
pb-4    bottom
pl-4    left

px-4    left + right
py-4    top + bottom
```

그림으로 보면:

```text
                 pt-4
                  ↓
        ┌─────────────────┐
        │                 │
 pl-4 → │     Content     │ ← pr-4
        │                 │
        └─────────────────┘
                  ↑
                 pb-4
```

그리고:

```text
px-4
→ 좌우 padding

py-4
→ 상하 padding
```

입니다.

실무에서는:

```jsx
<button className="px-4 py-2">
  저장
</button>
```

같은 형태를 매우 자주 사용합니다.

---

# 5. Margin

Margin도 동일한 규칙을 사용합니다.

```text
m-4

mt-4
mr-4
mb-4
ml-4

mx-4
my-4
```

예:

```jsx
<h1 className="mb-4">
  상품 목록
</h1>
```

의미는:

```text
mb-4
││
│└─ spacing 값
│
└── margin-bottom
```

입니다.

또한:

```html
mx-auto
```

도 매우 자주 사용합니다.

예:

```jsx
<div className="max-w-4xl mx-auto">
```

개념적으로:

```text
max-w-4xl
→ 최대 너비 제한

mx-auto
→ 좌우 margin auto
```

이므로 고정된 최대 너비를 가진 컨테이너를 가운데 정렬할 때 자주 사용합니다.

---

# 6. Gap

Flexbox나 Grid의 자식 요소 사이 간격은 `gap`을 사용합니다.

```jsx
<div className="flex gap-4">
  <div>A</div>
  <div>B</div>
  <div>C</div>
</div>
```

개념적으로:

```text
┌─────┐    ┌─────┐    ┌─────┐
│  A  │    │  B  │    │  C  │
└─────┘    └─────┘    └─────┘
        ↑           ↑
       gap-4       gap-4
```

방향별로:

```text
gap-4
gap-x-4
gap-y-4
```

를 사용할 수 있습니다.

---

# 7. Spacing Scale

Tailwind에서는 일정한 spacing scale을 사용합니다.

대표적인 예:

```text
p-1
p-2
p-3
p-4
p-6
p-8
```

디폴트 theme에서 대표적으로:

```text
1 → 0.25rem
2 → 0.5rem
4 → 1rem
6 → 1.5rem
8 → 2rem
```

입니다.

따라서:

```text
p-4
```

는 디폴트 theme에서:

```css
padding: 1rem;
```

에 해당합니다.

중요한 점은 숫자를 전부 외우는 것이 아니라 **일관된 spacing scale을 사용한다는 것**입니다.

---

# 8. Sizing — 크기

다음은 width와 height입니다.

CSS:

```css
width: 100%;
height: 4rem;
```

Tailwind에서는:

```text
w-full
h-16
```

처럼 표현합니다.

---

# 9. Width

대표적인 Width Utility:

```text
w-4
w-8
w-16
w-32

w-full
w-screen
w-auto
```

예:

```jsx
<input className="w-full" />
```

`w-full`은 부모가 제공하는 사용 가능한 너비를 기준으로 `width: 100%`를 적용합니다.

---

# 10. Height

Height도 같은 방식입니다.

```text
h-4
h-8
h-16
h-32

h-full
h-screen
h-auto
```

예:

```jsx
<div className="h-screen">
  ...
</div>
```

개념적으로:

```text
h-screen
    ↓
viewport 높이만큼
```

입니다.

---

# 11. Min / Max Size

실제 웹 페이지에서는 최대 너비를 제한하는 경우가 많습니다.

예:

```jsx
<div className="max-w-4xl mx-auto">
```

구조:

```text
Browser
┌───────────────────────────────────────┐
│                                       │
│      ┌─────────────────────────┐      │
│      │                         │      │
│      │       max-w-4xl         │      │
│      │                         │      │
│      └─────────────────────────┘      │
│                                       │
└───────────────────────────────────────┘
                ↑
              mx-auto
```

대표 Utility:

```text
min-w-*
max-w-*

min-h-*
max-h-*
```

가 있습니다.

---

# 12. Typography — 텍스트

Typography에서는 주로 다음을 사용합니다.

```text
font-size
font-weight
text-align
line-height
letter-spacing
```

Tailwind에서는:

```text
text-
font-
leading-
tracking-
```

등의 Utility를 사용합니다.

---

# 13. Font Size

대표적인 크기:

```text
text-xs
text-sm
text-base
text-lg
text-xl
text-2xl
text-3xl
text-4xl
```

예:

```jsx
<h1 className="text-3xl">
  상품 목록
</h1>
```

중요한 점은 `text-*`가 두 가지 문맥에서 사용된다는 것입니다.

```text
text-3xl
→ font-size

text-blue-600
→ text color

text-center
→ text-align
```

즉 `text-`만 보고 판단하는 것이 아니라 **뒤의 값까지 함께 읽어야 합니다.**

---

# 14. Font Weight

CSS:

```css
font-weight: 700;
```

Tailwind:

```text
font-bold
```

대표적인 Utility:

```text
font-light
font-normal
font-medium
font-semibold
font-bold
font-extrabold
```

예:

```jsx
<h2 className="text-xl font-bold">
  Mechanical Keyboard
</h2>
```

---

# 15. Text Alignment

```text
text-left
text-center
text-right
text-justify
```

예:

```jsx
<h1 className="text-center">
  Login
</h1>
```

---

# 16. Line Height

Line Height는:

```text
leading-
```

을 사용합니다.

예:

```text
leading-none
leading-tight
leading-normal
leading-relaxed
leading-loose
```

```jsx
<p className="leading-relaxed">
  Tailwind CSS를 사용하면 Utility를 조합하여
  빠르게 UI를 만들 수 있습니다.
</p>
```

---

# 17. Colors

Tailwind에서는 색상을 다양한 Utility에서 동일한 패턴으로 사용합니다.

예:

```text
blue-50
blue-100
blue-200
...
blue-500
blue-600
...
blue-950
```

이를:

```text
background
text
border
```

등에 적용합니다.

---

# 18. Background Color

```text
bg-blue-500
bg-red-500
bg-green-500
bg-gray-100
bg-white
bg-black
```

예:

```jsx
<div className="bg-blue-500">
```

구조를 보면:

```text
bg - blue - 500
│      │      │
│      │      └─ shade
│      └──────── color
└─────────────── background
```

입니다.

---

# 19. Text Color

```text
text-blue-600
text-gray-700
text-red-500
text-white
text-black
```

예:

```jsx
<p className="text-gray-600">
  상품 설명입니다.
</p>
```

---

# 20. Border Color

```text
border-gray-300
border-blue-500
border-red-500
```

예:

```jsx
<input className="border border-gray-300" />
```

여기서:

```text
border
→ border width

border-gray-300
→ border color
```

라는 역할 차이가 있습니다.

---

# 21. Border

가장 기본적인 Border는:

```text
border
```

입니다.

두께를 조절할 수도 있습니다.

```text
border
border-2
border-4
```

방향별 Utility도 있습니다.

```text
border-t
border-r
border-b
border-l
```

예:

```jsx
<div className="border-b border-gray-200">
```

---

# 22. Border Radius

모서리를 둥글게 만들 때:

```text
rounded
```

를 사용합니다.

대표적인 Utility:

```text
rounded-sm
rounded
rounded-md
rounded-lg
rounded-xl
rounded-2xl
rounded-full
```

예:

```jsx
<button className="rounded-lg">
```

또는 원형 Avatar:

```jsx
<img
  className="w-12 h-12 rounded-full"
  src={profileImage}
/>
```

---

# 23. Shadow

그림자는:

```text
shadow-
```

를 사용합니다.

대표적인 예:

```text
shadow-sm
shadow
shadow-md
shadow-lg
shadow-xl
```

예:

```jsx
<div className="shadow-md">
```

Card에서 매우 자주 사용합니다.

```jsx
<div className="
  bg-white
  rounded-xl
  shadow-md
">
```

---

# 24. Opacity

투명도는:

```text
opacity-
```

를 사용합니다.

예:

```text
opacity-0
opacity-25
opacity-50
opacity-75
opacity-100
```

```jsx
<div className="opacity-50">
  Disabled Content
</div>
```

---

# 25. 실제 Card에 Utility 조합하기

지금까지 배운 Utility를 하나의 Component에 적용해 보겠습니다.

```jsx
export default function ProductCard() {
  return (
    <div
      className="
        max-w-sm
        mx-auto
        p-6
        bg-white
        border
        border-gray-200
        rounded-xl
        shadow-md
      "
    >
      <h2
        className="
          text-xl
          font-bold
          text-gray-900
        "
      >
        Mechanical Keyboard
      </h2>

      <p
        className="
          mt-2
          text-sm
          text-gray-600
          leading-relaxed
        "
      >
        개발자를 위한 기계식 키보드입니다.
      </p>

      <div className="mt-6">
        <span
          className="
            text-2xl
            font-bold
            text-blue-600
          "
        >
          129,000원
        </span>
      </div>
    </div>
  )
}
```

여기에 사용된 Utility를 카테고리로 분류하면:

```text
Spacing
├─ p-6
├─ mt-2
└─ mt-6

Sizing
└─ max-w-sm

Typography
├─ text-xl
├─ text-sm
├─ text-2xl
├─ font-bold
└─ leading-relaxed

Colors
├─ bg-white
├─ text-gray-900
├─ text-gray-600
└─ text-blue-600

Border
├─ border
├─ border-gray-200
└─ rounded-xl

Effect
└─ shadow-md
```

입니다.

이것이 Tailwind의 기본적인 사용 방식입니다.

> **작은 Utility를 조합해서 하나의 Component 디자인을 만든다.**

---

# 26. CSS와 Tailwind를 비교해 보기

기존 CSS:

```css
.card {
  max-width: 24rem;
  margin-left: auto;
  margin-right: auto;
  padding: 1.5rem;
  background-color: white;
  border: 1px solid;
  border-radius: 0.75rem;
}
```

React:

```jsx
<div className="card">
```

Tailwind에서는 별도의 `.card` 클래스를 만들지 않고:

```jsx
<div
  className="
    max-w-sm
    mx-auto
    p-6
    bg-white
    border
    rounded-xl
  "
>
```

처럼 필요한 Utility를 조합할 수 있습니다.

즉:

```text
일반 CSS

.card
  ↓
여러 CSS Property 정의
  ↓
className="card"


Tailwind

여러 Utility
  ↓
className="
  max-w-sm
  mx-auto
  p-6
  bg-white
  border
  rounded-xl
"
```

입니다.

---

# 27. Utility를 읽는 방법

처음부터 Utility를 모두 외우려고 하지 마십시오.

다음 순서로 읽는 것이 좋습니다.

예:

```text
px-4
```

먼저:

```text
p
→ padding
```

그리고:

```text
x
→ horizontal
→ left + right
```

마지막:

```text
4
→ spacing scale
```

따라서:

```text
px-4
 ↓
좌우 padding
```

입니다.

다른 예:

```text
text-blue-600
```

는:

```text
text
   +
blue
   +
600
 ↓
text color
```

입니다.

또:

```text
rounded-lg
```

는:

```text
rounded
   +
lg
 ↓
border-radius
```

입니다.

이렇게 **Utility 이름을 분해해서 읽는 습관**을 들이는 것이 중요합니다.

---

# 28. 자주 사용하는 조합

실무에서는 특정 조합이 반복해서 등장합니다.

버튼:

```text
px-4 py-2
bg-blue-500
text-white
font-semibold
rounded-lg
```

입력창:

```text
w-full
px-4 py-2
border
border-gray-300
rounded-lg
```

Card:

```text
p-6
bg-white
border
rounded-xl
shadow-md
```

페이지 Container:

```text
max-w-7xl
mx-auto
px-4
```

제목:

```text
text-3xl
font-bold
text-gray-900
```

본문:

```text
text-base
text-gray-600
leading-relaxed
```

Utility를 하나씩 외우는 것보다 이러한 **조합 패턴**을 익히는 것이 실제 개발에서는 더 중요합니다.

---

# 29. 이번 PART에서 아직 다루지 않는 것

다음과 같은 Utility도 매우 중요하지만 이후 PART에서 따로 다룹니다.

```text
Responsive

sm:
md:
lg:
xl:
2xl:
```

```text
State

hover:
focus:
active:
disabled:
```

```text
Layout

flex
grid
items-center
justify-between
grid-cols-*
```

따라서 지금은:

```text
Spacing
Sizing
Typography
Colors
Borders
Effects
```

에 집중하면 됩니다.

---

# 30. PART 3 핵심 정리

Tailwind Utility를 다음과 같이 기억하면 됩니다.

```text
                    Core Utility
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ▼                 ▼                 ▼
    Spacing           Sizing          Typography
 p / m / gap        w / h / max-w     text / font
       │                 │                 │
       └──────────┬──────┴──────────┬──────┘
                  │                 │
                  ▼                 ▼
                Colors           Borders
              bg / text         border /
                                rounded
                  │                 │
                  └────────┬────────┘
                           ▼
                         Effects
                         shadow
                         opacity
```

그리고 실제 사용은:

```text
CSS에서 필요한 스타일 생각
           ↓
해당 Tailwind Utility 선택
           ↓
className에 여러 Utility 조합
           ↓
Component 완성
```

예:

```jsx
<div
  className="
    max-w-sm
    mx-auto
    p-6
    bg-white
    border
    border-gray-200
    rounded-xl
    shadow-md
  "
>
```

핵심은 이것입니다.

> **Tailwind를 잘한다는 것은 수백 개의 클래스를 외우는 것이 아니라, CSS 속성을 보고 적절한 Utility를 찾고 조합할 수 있다는 뜻입니다.**

---

# 다음 PART

## PART 4. 반응형 디자인 — Responsive

다음 PART에서는 동일한 UI를 화면 크기에 따라 변화시키는 방법을 배웁니다.

```text
Mobile
   ↓
sm:
   ↓
md:
   ↓
lg:
   ↓
xl:
   ↓
2xl:
```

예:

```jsx
<div
  className="
    grid
    grid-cols-1
    md:grid-cols-2
    lg:grid-cols-3
  "
>
```

핵심은 Tailwind의 **Mobile-First Responsive Design**입니다.
