# PART 8. Typography, Color & Visual Styling

## Tailwind CSS로 UI의 시각적 표현 완성하기

지금까지는 UI의 구조와 공간을 만들었습니다.

```text
PART 6
Flexbox / Grid
→ Layout

PART 7
Sizing / Spacing
→ 크기와 공간

PART 8
Typography / Color / Visual Styling
→ 시각적 표현
```

이번 PART에서는 다음 질문에 답합니다.

> 같은 Layout이라도 어떻게 하면 정보의 중요도가 명확하고, 읽기 쉽고, 일관된 UI로 만들 수 있을까?

핵심 영역은 다음과 같습니다.

```text
Typography
├─ font-size
├─ font-weight
├─ line-height
├─ letter-spacing
├─ text-align
└─ text decoration

Color
├─ text color
├─ background
└─ opacity

Visual
├─ border
├─ border-radius
├─ shadow
├─ ring
└─ divide
```

---

# 1. Typography의 역할

Typography는 단순히 글자 크기를 변경하는 작업이 아닙니다.

UI에서는 Typography를 통해 **정보의 계층(Hierarchy)**을 표현합니다.

예:

```text
상품 상세

무선 헤드폰                    ← 제목
편안한 착용감과 뛰어난 음질     ← 설명
129,000원                     ← 핵심 정보
무료배송                       ← 보조 정보
```

모든 글자의 크기와 굵기가 같다면:

```text
무선 헤드폰
편안한 착용감과 뛰어난 음질
129,000원
무료배송
```

정보의 중요도를 빠르게 구분하기 어렵습니다.

따라서:

```text
Typography
      ↓
Visual Hierarchy
      ↓
정보의 중요도 전달
```

이라는 관점으로 접근해야 합니다.

---

# 2. `text-*` — Font Size

Tailwind에서는 `text-*` Utility로 font size를 설정합니다.

```jsx
<h1 className="text-4xl">
  상품 목록
</h1>
```

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
...
```

일반적인 관계:

```text
작음

text-xs
   ↓
text-sm
   ↓
text-base
   ↓
text-lg
   ↓
text-xl
   ↓
text-2xl
   ↓
text-4xl

큼
```

중요한 것은 숫자를 암기하는 것이 아니라 **정보 계층에 맞는 size scale을 일관되게 사용하는 것**입니다.

---

# 3. Typography Hierarchy

예를 들어 상품 Card를 만들겠습니다.

```jsx
<article>
  <h3 className="text-lg font-semibold">
    무선 헤드폰
  </h3>

  <p className="text-sm text-gray-500">
    프리미엄 노이즈 캔슬링
  </p>

  <p className="text-xl font-bold">
    129,000원
  </p>
</article>
```

구조:

```text
무선 헤드폰
↑
text-lg + font-semibold

프리미엄 노이즈 캔슬링
↑
text-sm + muted color

129,000원
↑
text-xl + font-bold
```

즉 Typography hierarchy는 단순히 font-size 하나로 만들지 않습니다.

```text
Size
+
Weight
+
Color
+
Spacing
```

을 함께 사용합니다.

---

# 4. `font-*` — Font Weight

대표 Utility:

```text
font-thin
font-light
font-normal
font-medium
font-semibold
font-bold
font-extrabold
```

예:

```jsx
<h2 className="font-bold">
  인기 상품
</h2>
```

```jsx
<p className="font-normal">
  상품 설명입니다.
</p>
```

```jsx
<span className="font-semibold">
  129,000원
</span>
```

모든 중요한 정보를 `font-bold`로 만들면 오히려 hierarchy가 사라질 수 있습니다.

```text
제목
→ semibold / bold

본문
→ normal

중요한 값
→ semibold / bold

보조 정보
→ normal + muted color
```

처럼 역할에 따라 구분하는 것이 좋습니다.

---

# 5. Font Family

Tailwind에서는 font family도 Utility로 지정할 수 있습니다.

대표적으로:

```text
font-sans
font-serif
font-mono
```

예:

```jsx
<body className="font-sans">
```

코드 표현:

```jsx
<code className="font-mono">
  npm run dev
</code>
```

실제 프로젝트에서는 PART 9에서 배울 Theme 설정을 통해 프로젝트의 기본 font family를 Design Token으로 정의하는 경우가 많습니다.

---

# 6. Line Height — `leading-*`

Line height는 여러 줄 텍스트의 읽기 편의성에 큰 영향을 줍니다.

Tailwind:

```text
leading-none
leading-tight
leading-snug
leading-normal
leading-relaxed
leading-loose
```

예:

```jsx
<p className="leading-relaxed">
  Tailwind CSS는 Utility-First 방식으로
  UI를 빠르게 구성할 수 있습니다.
</p>
```

비교:

```text
leading-tight

첫 번째 줄
두 번째 줄
세 번째 줄


leading-relaxed

첫 번째 줄

두 번째 줄

세 번째 줄
```

제목은 비교적 촘촘하게:

```jsx
<h1 className="text-4xl font-bold leading-tight">
```

본문은 조금 여유롭게:

```jsx
<p className="leading-relaxed">
```

사용하는 패턴이 일반적입니다.

---

# 7. Letter Spacing — `tracking-*`

글자 사이의 간격은 `tracking-*`으로 조절합니다.

대표:

```text
tracking-tighter
tracking-tight
tracking-normal
tracking-wide
tracking-wider
tracking-widest
```

예:

```jsx
<h1 className="tracking-tight">
  Better Products
</h1>
```

또는 작은 Label:

```jsx
<span
  className="
    text-xs
    font-semibold
    uppercase
    tracking-wider
  "
>
  NEW PRODUCT
</span>
```

Letter spacing 역시 지나치게 사용하면 가독성이 떨어질 수 있습니다.

---

# 8. Text Alignment

대표 Utility:

```text
text-left
text-center
text-right
text-justify
```

예:

```jsx
<h1 className="text-center">
  인기 상품
</h1>
```

Responsive Variant와 조합할 수도 있습니다.

```jsx
<h1
  className="
    text-center
    md:text-left
  "
>
```

```text
base
→ 가운데 정렬

md 이상
→ 왼쪽 정렬
```

---

# 9. Text Transform

대표 Utility:

```text
uppercase
lowercase
capitalize
normal-case
```

예:

```jsx
<span className="uppercase">
  new
</span>
```

결과:

```text
NEW
```

작은 Badge나 Category Label에서 사용할 수 있습니다.

---

# 10. Text Decoration

대표:

```text
underline
overline
line-through
no-underline
```

예:

```jsx
<a
  className="
    underline
    underline-offset-4
  "
>
  자세히 보기
</a>
```

할인 가격:

```jsx
<span className="text-gray-400 line-through">
  159,000원
</span>

<span className="font-bold text-red-600">
  129,000원
</span>
```

결과:

```text
159,000원   129,000원
─────────
 이전 가격     현재 가격
```

---

# 11. Text Overflow

PART 7에서 배운 Sizing과 Typography가 연결되는 부분입니다.

한 줄 줄임:

```jsx
<p className="truncate">
  매우 길고 긴 상품 이름입니다...
</p>
```

여러 줄 제한이 필요한 경우 프로젝트 환경과 Tailwind 버전에서 제공되는 line-clamp Utility를 사용할 수 있습니다.

예:

```jsx
<p className="line-clamp-2">
  상품에 대한 매우 긴 설명...
</p>
```

결과:

```text
이 상품은 최신 기술을 적용하여
뛰어난 성능과 편의성을...
```

Typography는 Container의 width와 함께 생각해야 합니다.

---

# 12. Tailwind Color System

Tailwind에서는 Color Utility를 다양한 CSS property에 일관된 형태로 적용합니다.

예:

```text
text-blue-600

bg-blue-600

border-blue-600

ring-blue-600
```

즉:

```text
Color Token
    │
 blue-600
    │
    ├─ text-*
    ├─ bg-*
    ├─ border-*
    └─ ring-*
```

처럼 이해할 수 있습니다.

---

# 13. Color Scale

색상은 일반적으로 여러 단계의 shade를 제공합니다.

개념적으로:

```text
blue-50
   ↓
blue-100
   ↓
blue-200
   ↓
...
blue-500
   ↓
blue-600
   ↓
...
blue-900
   ↓
blue-950
```

대체로 숫자가 커질수록 더 진한 shade가 됩니다.

예:

```jsx
<div className="bg-blue-50">
```

```jsx
<button className="bg-blue-600">
```

```jsx
<p className="text-blue-900">
```

---

# 14. Text Color

```jsx
<h1 className="text-gray-900">
  상품 목록
</h1>
```

```jsx
<p className="text-gray-500">
  총 24개의 상품
</p>
```

```jsx
<span className="text-red-600">
  품절
</span>
```

중요:

> Color는 단순한 장식이 아니라 정보의 역할과 상태를 전달합니다.

예:

```text
Primary text
→ 높은 contrast

Secondary text
→ 낮은 visual emphasis

Error
→ 위험/실패

Success
→ 성공/완료
```

다만 **색상만으로 상태를 전달하면 안 됩니다.**

예:

```text
잘못된 접근

● 빨강
● 초록

색만 다름
```

보다는:

```text
오류
✓ 완료
```

처럼 text/icon 등의 추가 signal을 함께 사용해야 합니다.

---

# 15. Background Color

```jsx
<div className="bg-gray-50">
```

```jsx
<button className="bg-blue-600 text-white">
```

```jsx
<span className="bg-red-50 text-red-700">
  품절
</span>
```

Background와 Text Color는 항상 함께 생각해야 합니다.

```text
Background
     +
Text
     ↓
Contrast
     ↓
Readability
```

---

# 16. Color + State Variant

PART 5에서 배운 Variant와 연결합니다.

```jsx
<button
  className="
    bg-blue-600
    text-white

    hover:bg-blue-700
    active:bg-blue-800

    focus-visible:ring-2
    focus-visible:ring-blue-500
    focus-visible:ring-offset-2

    disabled:bg-gray-300
    disabled:text-gray-500
  "
>
  구매하기
</button>
```

상태:

```text
Default
blue-600

   ↓ hover

blue-700

   ↓ active

blue-800
```

즉:

```text
Color Utility
      +
State Variant
      ↓
Interactive Feedback
```

입니다.

---

# 17. Opacity

요소 전체의 투명도를 조절할 수 있습니다.

```text
opacity-0
opacity-25
opacity-50
opacity-75
opacity-100
```

예:

```jsx
<button
  disabled
  className="
    disabled:opacity-50
  "
>
```

하지만 중요한 점:

> `opacity-*`는 해당 Element와 그 렌더링된 내용 전체의 투명도에 영향을 줍니다.

따라서 단순히 background만 투명하게 만들고 싶은 경우와는 구분해야 합니다.

---

# 18. Color Opacity 문법

Tailwind에서는 색상에 alpha 값을 함께 지정할 수 있습니다.

예:

```jsx
<div className="bg-black/50">
```

의미:

```text
black
+
50% alpha
```

Overlay에 매우 자주 사용됩니다.

```jsx
<div className="absolute inset-0 bg-black/50" />
```

Text에도:

```jsx
<p className="text-white/80">
```

처럼 사용할 수 있습니다.

---

# 19. Border

대표 Utility:

```text
border
border-0
border-2
border-4

border-t
border-r
border-b
border-l
```

예:

```jsx
<div className="border">
```

색상:

```jsx
<div className="border border-gray-200">
```

특정 방향:

```jsx
<header className="border-b">
```

```jsx
<aside className="border-r">
```

PART 6에서 만든 Layout에서도 자주 사용했던 패턴입니다.

---

# 20. Border Radius

대표:

```text
rounded-none
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
<div className="rounded-xl">
```

Avatar:

```jsx
<img className="size-12 rounded-full">
```

Button:

```jsx
<button className="rounded-lg">
```

Card:

```jsx
<article className="rounded-xl">
```

중요한 것은 모든 요소에 서로 다른 radius를 무작위로 사용하는 것이 아니라 **Design System에서 일관된 radius scale을 사용하는 것**입니다.

---

# 21. 특정 Corner만 Radius 적용

예:

```text
rounded-t-xl
rounded-b-xl

rounded-l-lg
rounded-r-lg
```

Card Image:

```jsx
<img
  className="
    rounded-t-xl
    object-cover
  "
/>
```

처럼 특정 영역에만 적용할 수도 있습니다.

---

# 22. Shadow

Shadow는 Element의 elevation과 grouping을 표현하는 데 사용할 수 있습니다.

대표:

```text
shadow-sm
shadow
shadow-md
shadow-lg
shadow-xl
shadow-2xl
shadow-none
```

예:

```jsx
<article
  className="
    rounded-xl
    border
    shadow-sm
  "
>
```

Hover:

```jsx
<article
  className="
    shadow-sm
    transition-shadow
    hover:shadow-lg
  "
>
```

개념:

```text
Default
┌─────────────┐
│    Card     │
└─────────────┘
   shadow-sm

      ↓ hover

┌─────────────┐
│    Card     │
└─────────────┘
   shadow-lg
```

Shadow를 너무 강하게 사용하면 화면의 시각적 hierarchy가 복잡해질 수 있습니다.

---

# 23. Border vs Shadow

Card를 구분하는 방법은 여러 가지입니다.

```text
Border
→ 명확한 경계

Shadow
→ elevation / 깊이

Background
→ 영역 구분

Spacing
→ 그룹 구분
```

항상:

```jsx
border shadow-xl
```

을 동시에 사용할 필요는 없습니다.

UI의 목적에 따라 선택합니다.

---

# 24. Ring

Tailwind의 `ring-*` Utility는 특히 focus indicator에서 유용합니다.

```jsx
<input
  className="
    border
    focus:outline-none
    focus:ring-2
    focus:ring-blue-500
  "
/>
```

이번 강의에서는 접근성을 고려해 `focus-visible`과 함께 사용하는 패턴도 중요합니다.

```jsx
<button
  className="
    focus-visible:outline-none
    focus-visible:ring-2
    focus-visible:ring-blue-500
    focus-visible:ring-offset-2
  "
>
```

구조:

```text
        Ring
    ┌─────────────┐
    │ ┌─────────┐ │
    │ │ Button  │ │
    │ └─────────┘ │
    └─────────────┘
```

Ring은 단순 장식보다 **Keyboard focus를 명확하게 전달하는 용도**로 이해하는 것이 중요합니다.

---

# 25. Divide

반복되는 자식 Element 사이에 구분선을 넣을 수 있습니다.

```jsx
<ul className="divide-y divide-gray-200">
  <li className="py-4">상품 A</li>
  <li className="py-4">상품 B</li>
  <li className="py-4">상품 C</li>
</ul>
```

결과:

```text
상품 A
────────────────
상품 B
────────────────
상품 C
```

List, Table-like UI, Setting Menu 등에 유용합니다.

---

# 26. Gradient

Tailwind에서는 Gradient도 Utility 조합으로 만들 수 있습니다.

예:

```jsx
<div
  className="
    bg-linear-to-r
    from-blue-600
    to-purple-600
  "
>
```

개념:

```text
Blue ─────────────────→ Purple
```

중간 색상:

```jsx
<div
  className="
    bg-linear-to-r
    from-blue-600
    via-purple-600
    to-pink-600
  "
>
```

Gradient는 강조 영역이나 Hero 등에 사용할 수 있지만, 가독성과 전체 Design System을 해치지 않도록 제한적으로 사용하는 것이 좋습니다.

---

# 27. Aspect Ratio

이미지나 Media UI의 비율을 일정하게 유지할 때 사용합니다.

```text
aspect-square
aspect-video
```

예:

```jsx
<img
  className="
    aspect-square
    w-full
    object-cover
  "
/>
```

상품 Card에서 이미지 크기가 달라도:

```text
┌─────────┐
│ Image   │
│   1:1   │
└─────────┘

┌─────────┐
│ Image   │
│   1:1   │
└─────────┘
```

처럼 일관된 Card Layout을 만들 수 있습니다.

---

# 28. `object-fit`

이미지를 정해진 영역에 어떻게 맞출지 결정합니다.

대표:

```text
object-cover
object-contain
object-fill
object-none
object-scale-down
```

상품 이미지:

```jsx
<img
  className="
    size-full
    object-cover
  "
/>
```

`object-cover`:

```text
Container를 채움
+
비율 유지
+
일부 영역이 잘릴 수 있음
```

`object-contain`:

```text
전체 Image 표시
+
비율 유지
+
빈 공간이 생길 수 있음
```

상품 종류에 따라 적절한 방식을 선택합니다.

---

# 29. Visual Hierarchy

지금까지 배운 Utility들은 독립적으로 존재하지 않습니다.

예:

```jsx
<article
  className="
    overflow-hidden
    rounded-xl
    border
    border-gray-200
    bg-white
    shadow-sm
  "
>
  <img
    className="
      aspect-square
      w-full
      object-cover
    "
  />

  <div className="p-4">
    <p
      className="
        text-sm
        text-gray-500
      "
    >
      Audio
    </p>

    <h3
      className="
        mt-1
        text-lg
        font-semibold
        text-gray-900
      "
    >
      무선 헤드폰
    </h3>

    <p
      className="
        mt-3
        text-xl
        font-bold
        text-blue-600
      "
    >
      129,000원
    </p>
  </div>
</article>
```

여기에는:

```text
Typography
Color
Spacing
Border
Radius
Shadow
Image Styling
```

이 모두 사용됩니다.

---

# 30. 실전 — Primary Button

```jsx
<button
  className="
    rounded-lg
    bg-blue-600
    px-4
    py-2

    text-sm
    font-semibold
    text-white

    shadow-sm
    transition

    hover:bg-blue-700
    hover:shadow-md

    active:bg-blue-800

    focus-visible:outline-none
    focus-visible:ring-2
    focus-visible:ring-blue-500
    focus-visible:ring-offset-2

    disabled:cursor-not-allowed
    disabled:opacity-50
  "
>
  구매하기
</button>
```

Button 하나에도 여러 개념이 연결됩니다.

```text
Shape
→ rounded-lg

Color
→ bg / text

Spacing
→ px / py

Typography
→ text-sm / font-semibold

State
→ hover / active / disabled

Accessibility
→ focus-visible
```

---

# 31. 실전 — Badge

```jsx
<span
  className="
    inline-flex
    items-center

    rounded-full
    bg-green-50

    px-2.5
    py-1

    text-xs
    font-medium
    text-green-700
  "
>
  배송 가능
</span>
```

Badge에서는:

```text
작은 Typography
+
Background
+
Text Color
+
Rounded Shape
+
작은 Padding
```

의 조합이 핵심입니다.

---

# 32. 실전 — Alert

```jsx
<div
  className="
    rounded-lg
    border
    border-red-200
    bg-red-50
    p-4
    text-red-800
  "
>
  상품 정보를 불러오지 못했습니다.
</div>
```

여기서 Color는 단순한 장식이 아니라 **상태의 의미**를 전달합니다.

하지만 실제 접근성에서는:

```text
색상
+
Icon
+
Text
```

를 함께 사용하는 것이 좋습니다.

---

# 33. 실전 — Input

```jsx
<input
  type="text"
  placeholder="상품 검색"
  className="
    w-full
    rounded-lg
    border
    border-gray-300
    bg-white
    px-4
    py-2

    text-gray-900
    placeholder:text-gray-400

    focus:border-blue-500
    focus:outline-none
    focus:ring-2
    focus:ring-blue-500/20
  "
/>
```

Input에서는:

```text
Default Border
      ↓ focus
Focus Border
+
Focus Ring
```

으로 사용자의 현재 위치를 명확하게 보여주는 것이 중요합니다.

---

# 34. Dark Mode 개념

Tailwind에서는 `dark:` Variant를 이용해 dark color scheme에 대응할 수 있습니다.

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

개념:

```text
Light

bg-white
text-gray-900

       ↓ dark

Dark

bg-gray-900
text-white
```

다만 **dark mode는 단순히 white와 black을 뒤집는 작업이 아닙니다.**

Background, Surface, Border, Text, Muted Text, Interactive State의 contrast를 함께 설계해야 합니다.

이번 강의자료의 시각 디자인 자체는 밝은 톤을 유지하고, dark mode는 개념과 코드 중심으로만 설명합니다.

---

# 35. Responsive Typography

Typography에도 Responsive Variant를 사용할 수 있습니다.

```jsx
<h1
  className="
    text-3xl
    font-bold
    tracking-tight

    md:text-4xl
    lg:text-5xl
  "
>
  Better Products
</h1>
```

```text
base
text-3xl

   ↓ md

text-4xl

   ↓ lg

text-5xl
```

하지만 화면이 커진다고 모든 text를 크게 만들 필요는 없습니다.

특히 Body Text는 **가독성**을 우선해야 합니다.

---

# 36. Responsive Visual Styling

Visual Styling도 breakpoint에 따라 변경할 수 있습니다.

```jsx
<article
  className="
    rounded-lg
    shadow-sm

    md:rounded-xl
    md:shadow-md
  "
>
```

기술적으로 가능하지만, 단순히 breakpoint가 바뀐다는 이유만으로 모든 visual property를 바꿀 필요는 없습니다.

Responsive Variant는 **실제 디자인 요구사항이 있을 때** 사용합니다.

---

# 37. 자주 하는 실수 ① 모든 Text를 `text-gray-*`로 임의 지정

프로젝트 전체에서:

```text
text-gray-900
text-gray-700
text-gray-600
text-gray-500
text-gray-400
```

를 컴포넌트마다 임의로 선택하면 일관성이 무너질 수 있습니다.

실제 프로젝트에서는:

```text
Primary Text
Secondary Text
Muted Text
Disabled Text
```

같은 semantic role을 Design System으로 정의하는 것이 좋습니다.

PART 9의 Theme/Design Token에서 이 문제를 해결합니다.

---

# 38. 자주 하는 실수 ② 모든 Card에 강한 Shadow

```jsx
shadow-2xl
```

을 모든 Card에 사용하면 모든 Element가 중요해 보입니다.

Visual Hierarchy에서는:

```text
Normal Surface
→ border 또는 매우 약한 shadow

Interactive Card
→ hover 시 shadow 증가

Modal / Floating Panel
→ 상대적으로 높은 elevation
```

처럼 단계가 필요합니다.

---

# 39. 자주 하는 실수 ③ 색상만으로 상태 전달

예:

```text
●
```

빨간색인지 초록색인지에 따라서만 상태를 구분하면 접근성이 떨어집니다.

대신:

```text
✓ 결제 완료

! 결제 실패
```

처럼:

```text
Color
+
Icon
+
Text
```

를 함께 사용합니다.

---

# 40. 자주 하는 실수 ④ Focus 제거

다음처럼:

```jsx
className="outline-none"
```

만 사용하고 다른 focus indicator를 제공하지 않으면 keyboard 사용자가 현재 focus 위치를 알기 어렵습니다.

따라서:

```jsx
focus-visible:outline-none
focus-visible:ring-2
focus-visible:ring-blue-500
```

처럼 **대체 focus indicator를 반드시 제공**해야 합니다.

---

# 41. 자주 하는 실수 ⑤ Utility를 디자인 기준 없이 선택

예:

```text
Card A → rounded-lg
Card B → rounded-2xl
Card C → rounded-md

Button A → blue-500
Button B → blue-700
Button C → indigo-600
```

기술적으로 모두 가능하지만 UI의 일관성이 떨어집니다.

따라서:

```text
Color
Typography
Radius
Shadow
Spacing
```

을 프로젝트 전체에서 일정한 규칙으로 사용하는 것이 중요합니다.

이것이 다음 PART에서 배울 **Design Token**으로 연결됩니다.

---

# 42. Visual Styling 사고 순서

UI를 만들 때 바로 `shadow-xl`, `text-blue-600`부터 선택하지 않습니다.

```text
① 정보의 역할은 무엇인가?
        ↓
Typography hierarchy

② 상태나 의미를 Color로 표현해야 하는가?
        ↓
Semantic Color

③ 영역을 어떻게 구분할 것인가?
        ↓
Spacing / Background / Border

④ Shape가 필요한가?
        ↓
Radius

⑤ Elevation이 필요한가?
        ↓
Shadow

⑥ Interactive Element인가?
        ↓
Hover / Active / Focus / Disabled
```

이 순서로 생각하면 불필요한 styling을 줄일 수 있습니다.

---

# 43. Product Card 최종 예제

```jsx
<article
  className="
    group
    overflow-hidden
    rounded-xl
    border
    border-gray-200
    bg-white
    shadow-sm

    transition-shadow
    hover:shadow-lg
  "
>
  <div className="overflow-hidden">
    <img
      src={product.image}
      alt={product.name}
      className="
        aspect-square
        w-full
        object-cover

        transition-transform
        group-hover:scale-105
      "
    />
  </div>

  <div className="p-4">
    <span
      className="
        text-xs
        font-medium
        uppercase
        tracking-wide
        text-gray-500
      "
    >
      {product.category}
    </span>

    <h3
      className="
        mt-1
        truncate
        text-lg
        font-semibold
        text-gray-900
      "
    >
      {product.name}
    </h3>

    <p
      className="
        mt-3
        text-xl
        font-bold
        text-blue-600
      "
    >
      {product.price.toLocaleString()}원
    </p>

    <button
      className="
        mt-4
        w-full
        rounded-lg
        bg-blue-600
        px-4
        py-2.5

        text-sm
        font-semibold
        text-white

        transition

        hover:bg-blue-700
        active:bg-blue-800

        focus-visible:outline-none
        focus-visible:ring-2
        focus-visible:ring-blue-500
        focus-visible:ring-offset-2
      "
    >
      장바구니 담기
    </button>
  </div>
</article>
```

지금까지의 PART가 모두 연결됩니다.

```text
PART 5
State Variant
       │
       ▼
hover / focus-visible

PART 6
Layout
       │
       ▼
Flex / Grid

PART 7
Sizing & Spacing
       │
       ▼
w / p / m / gap

PART 8
Visual Styling
       │
       ▼
Typography / Color
Border / Radius / Shadow
```

---

# 44. PART 8 핵심 정리

## Typography

```text
text-*
font-*
leading-*
tracking-*

text-left
text-center
text-right

uppercase
underline
truncate
line-clamp-*
```

## Color

```text
text-*
bg-*
border-*
ring-*

color/alpha
```

## Visual Styling

```text
border-*
rounded-*
shadow-*
ring-*
divide-*

aspect-*
object-*
opacity-*
```

그리고 State Variant와 결합합니다.

```text
hover:
active:
focus:
focus-visible:
disabled:
dark:
```

---

# 가장 중요한 원칙

```text
좋은 UI
   ≠
Utility를 많이 사용한 UI

좋은 UI
   =
명확한 정보 계층
+
일관된 Color
+
적절한 Contrast
+
일관된 Radius / Shadow
+
명확한 Interactive Feedback
```

Tailwind의 Utility는 Design을 대신해 주는 것이 아닙니다.

> **Tailwind는 디자인 결정을 빠르고 일관되게 코드로 표현할 수 있도록 도와주는 도구입니다.**

다음 PART에서는 지금까지 반복해서 사용했던 Color, Typography, Spacing 등의 값을 프로젝트 전체에서 일관되게 관리하는 방법을 배웁니다.

```text
PART 9
Theme & Design Token

Color
Typography
Spacing
Radius
...
      ↓
프로젝트의 Design System
```

# PART 8 이미지 구성 — 총 8장

```text
1/8
Typography 전체 개념
+ Visual Hierarchy
+ text-* / font-*

2/8
Line Height / Letter Spacing
+ Alignment / Decoration
+ truncate / line-clamp

3/8
Tailwind Color System
+ Text / Background
+ Color Scale / Contrast

4/8
Color + State Variant
+ Opacity / Alpha
+ Button 상태

5/8
Border / Radius / Divide
+ 일관된 Shape 설계

6/8
Shadow / Ring
+ Elevation
+ Focus Accessibility

7/8
Image Styling
+ aspect-ratio / object-fit
+ Gradient
+ Dark Mode 개념

8/8
Product Card 종합 실전
+ Typography
+ Color
+ Border / Radius / Shadow
+ State
+ PART 8 전체 정리
```

이렇게 **8장**으로 구성하면 PART 6·7과 동일한 이미지 시리즈 구조를 유지하면서, PART 9의 **Theme / Design Token**으로 자연스럽게 연결할 수 있습니다.
