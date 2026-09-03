# PART 3. CSS Box Model

## 1. HTML Element와 Box

PART 2에서는 Selector를 이용하여 CSS를 적용할 HTML Element를 선택했습니다.

```text
HTML Element
      ↓
Selector Matching
      ↓
CSS 적용
```

이제 다음 질문이 생깁니다.

> **CSS가 적용된 HTML Element는 브라우저 화면에서 어떻게 다뤄지는가?**

CSS의 시각적 서식 모델에서 Element는 화면에 표시되면서 하나 이상의 **Box**를 생성할 수 있습니다.

입문 단계에서는 일반적인 Element 하나가 하나의 직사각형 Box로 표현된다고 생각하면 이해하기 쉽습니다.

예를 들어:

```html
<div class="card">
  Keyboard
</div>
```

CSS:

```css
.card {
  width: 300px;
  padding: 20px;
  border: 2px solid gray;
  margin: 30px;
}
```

브라우저는 이를 개념적으로 다음과 같은 Box로 다룹니다.

```text
┌────────────── margin ────────────────┐
│                                      │
│   ┌────────── border ────────────┐   │
│   │                              │   │
│   │   ┌────── padding ───────┐   │   │
│   │   │                       │   │   │
│   │   │       content         │   │   │
│   │   │       Keyboard        │   │   │
│   │   │                       │   │   │
│   │   └───────────────────────┘   │   │
│   │                              │   │
│   └──────────────────────────────┘   │
│                                      │
└──────────────────────────────────────┘
```

이 구조를 **CSS Box Model**이라고 합니다.

---

# 2. Box Model이란?

CSS Box Model은 Element가 생성하는 Box의 크기와 주변 영역을 이해하기 위한 기본 모델입니다.

Box는 안쪽에서 바깥쪽으로 다음 네 영역으로 구성됩니다.

```text
Content
   ↓
Padding
   ↓
Border
   ↓
Margin
```

그림으로 보면:

```text
┌──────────────── Margin ────────────────┐
│                                        │
│   ┌──────────── Border ────────────┐   │
│   │                                │   │
│   │   ┌──────── Padding ────────┐  │   │
│   │   │                          │  │   │
│   │   │        Content           │  │   │
│   │   │                          │  │   │
│   │   └──────────────────────────┘  │   │
│   │                                │   │
│   └────────────────────────────────┘   │
│                                        │
└────────────────────────────────────────┘
```

각 영역의 역할은 다음과 같습니다.

| 영역      | 의미                        |
| ------- | ------------------------- |
| Content | 실제 콘텐츠가 표시되는 영역           |
| Padding | Content와 Border 사이의 내부 여백 |
| Border  | Box의 테두리                  |
| Margin  | Border 바깥쪽의 외부 여백         |

Box Model은 앞으로 배울 **크기, 간격, Layout**의 기초가 됩니다.

---

# 3. Content

가장 안쪽에는 **Content Area**가 있습니다.

HTML:

```html
<div class="card">
  Keyboard
</div>
```

CSS:

```css
.card {
  width: 300px;
  height: 100px;
}
```

기본적인 `content-box` 모델에서 `width`와 `height`는 Content Box의 크기를 지정합니다.

```text
        width: 300px
     ←──────────────→

     ┌──────────────┐
     │              │
     │   Keyboard   │ ↑
     │              │ │ height
     │              │ │ 100px
     └──────────────┘ ↓

        Content
```

Content에는 텍스트뿐 아니라 이미지나 다른 Element 등이 들어갈 수 있습니다.

```html
<div class="card">
  <img src="keyboard.jpg" alt="Keyboard">
  <h2>Keyboard</h2>
  <p>50,000원</p>
</div>
```

---

# 4. Padding

**Padding**은 Content와 Border 사이의 내부 여백입니다.

```css
.card {
  padding: 20px;
}
```

구조:

```text
┌──────────── Border ────────────┐
│                                │
│       Padding 20px             │
│                                │
│       ┌──────────────┐         │
│       │   Content    │         │
│       └──────────────┘         │
│                                │
│       Padding 20px             │
│                                │
└────────────────────────────────┘
```

Padding을 사용하면 Content가 Border에 붙어 보이지 않도록 내부 공간을 만들 수 있습니다.

예를 들어 카드 UI에서:

```css
.card {
  padding: 20px;
}
```

를 사용하면 카드 내부의 콘텐츠와 테두리 사이에 공간이 생깁니다.

---

# 5. Padding의 방향

Padding은 네 방향을 개별적으로 지정할 수 있습니다.

```css
.card {
  padding-top: 10px;
  padding-right: 20px;
  padding-bottom: 30px;
  padding-left: 40px;
}
```

구조:

```text
              top
             10px
              ↓

       ┌─────────────┐
       │             │
 left  │   Content   │ right
 40px  │             │ 20px
       └─────────────┘

              ↑
             30px
            bottom
```

즉:

```text
padding-top
padding-right
padding-bottom
padding-left
```

가 있습니다.

---

# 6. Padding Shorthand

네 방향을 하나의 `padding` Property로 작성할 수 있습니다.

이를 **Shorthand Property**라고 합니다.

### 값 하나

```css
padding: 20px;
```

모든 방향:

```text
top    20px
right  20px
bottom 20px
left   20px
```

---

### 값 두 개

```css
padding: 10px 20px;
```

순서는:

```text
        10px
          ↑
          │
20px ← Content → 20px
          │
          ↓
        10px
```

즉:

```text
첫 번째 → top / bottom
두 번째 → left / right
```

입니다.

---

### 값 세 개

```css
padding: 10px 20px 30px;
```

순서는:

```text
top          10px

left/right   20px

bottom       30px
```

입니다.

---

### 값 네 개

```css
padding: 10px 20px 30px 40px;
```

순서는 시계 방향입니다.

```text
          10px
           TOP
            ↓

40px ←   Content   → 20px
LEFT                 RIGHT

            ↑
          30px
         BOTTOM
```

즉:

```text
top → right → bottom → left
```

입니다.

---

# 7. Border

**Border**는 Padding 바깥쪽에 위치하는 테두리입니다.

```css
.card {
  border: 2px solid gray;
}
```

`border`는 대표적인 Shorthand Property입니다.

```text
2px
│
└── border-width


solid
│
└── border-style


gray
│
└── border-color
```

따라서:

```css
border: 2px solid gray;
```

는 개념적으로:

```css
border-width: 2px;
border-style: solid;
border-color: gray;
```

와 같은 의미입니다.

---

# 8. Border Style

대표적인 Border Style에는 다음과 같은 것들이 있습니다.

```css
border-style: solid;
border-style: dashed;
border-style: dotted;
border-style: double;
border-style: none;
```

예:

```text
solid    ─────────────

dashed   ─── ─── ───

dotted   · · · · · ·

double   ═════════════
```

실제 UI에서는 `solid`가 가장 일반적으로 사용됩니다.

---

# 9. Border Radius

Box의 모서리를 둥글게 만들 수 있습니다.

```css
.card {
  border-radius: 10px;
}
```

기본 Box:

```text
┌──────────────┐
│              │
│     Card     │
│              │
└──────────────┘
```

`border-radius` 적용:

```text
╭──────────────╮
│              │
│     Card     │
│              │
╰──────────────╯
```

버튼, 카드, 입력창 등에서 매우 자주 사용됩니다.

```css
button {
  border-radius: 6px;
}
```

---

# 10. Margin

**Margin**은 Border 바깥쪽의 외부 여백입니다.

예를 들어 두 Box가 있다고 하겠습니다.

```text
┌─────────────┐
│   Card A    │
└─────────────┘
┌─────────────┐
│   Card B    │
└─────────────┘
```

Margin을 적용하면:

```css
.card {
  margin: 20px;
}
```

Box 바깥쪽에 공간이 만들어집니다.

```text
┌─────────────┐
│   Card A    │
└─────────────┘

      ↑
   외부 공간
   Margin

      ↓

┌─────────────┐
│   Card B    │
└─────────────┘
```

Padding과 Margin의 차이를 반드시 구분해야 합니다.

---

# 11. Padding과 Margin

두 개념은 초보자가 가장 많이 혼동합니다.

```text
                 Margin
                   ↓

       ┌───────────────────────┐
       │                       │
       │   ┌──── Border ────┐  │
       │   │                │  │
       │   │    Padding     │  │
       │   │       ↓        │  │
       │   │   ┌────────┐   │  │
       │   │   │Content │   │  │
       │   │   └────────┘   │  │
       │   │                │  │
       │   └────────────────┘  │
       │                       │
       └───────────────────────┘
```

핵심은:

```text
Padding
→ Border 안쪽의 공간


Margin
→ Border 바깥쪽의 공간
```

입니다.

---

# 12. Background는 어디까지 적용되는가?

Box Model을 이해할 때 중요한 부분입니다.

```css
.card {
  background-color: lightblue;
  padding: 20px;
  border: 5px solid navy;
  margin: 30px;
}
```

일반적으로 Element의 Background는:

```text
Content
+
Padding
+
Border 영역 아래까지
```

그려질 수 있습니다.

반면 Margin 영역은 Element의 Background가 칠해지는 영역이 아닙니다.

개념적으로:

```text
Margin             → Background 영역 아님

Border             → Background painting 영역에 포함 가능
Padding            → Background 표시
Content            → Background 표시
```

실제로 Background가 어느 Box까지 칠해지는지는 `background-clip`으로 제어할 수도 있습니다.

---

# 13. Margin의 방향과 Shorthand

Margin도 Padding과 동일한 방식으로 네 방향을 지정할 수 있습니다.

```css
margin-top: 10px;
margin-right: 20px;
margin-bottom: 30px;
margin-left: 40px;
```

Shorthand:

```css
margin: 10px 20px 30px 40px;
```

순서:

```text
top
 ↓
right
 ↓
bottom
 ↓
left
```

즉 시계 방향입니다.

```text
        10px
          ↑
          │
40px ←   Box   → 20px
          │
          ↓
        30px
```

---

# 14. `margin: auto`

Margin에는 `auto`를 사용할 수도 있습니다.

대표적인 예가 Block Box의 수평 중앙 정렬입니다.

```css
.container {
  width: 800px;
  margin-left: auto;
  margin-right: auto;
}
```

Shorthand로:

```css
.container {
  width: 800px;
  margin: 0 auto;
}
```

라고 많이 작성합니다.

개념적으로 사용 가능한 가로 공간이 남아 있을 때:

```text
Parent
┌───────────────────────────────────────┐
│                                       │
│ auto   ┌─────────────────────┐  auto  │
│←──────→│      Container      │←──────→│
│        └─────────────────────┘        │
│                                       │
└───────────────────────────────────────┘
```

처럼 좌우 `auto` Margin이 남는 공간을 나누어 가지면서 중앙에 배치될 수 있습니다.

단, `margin: 0 auto`가 **모든 Element를 무조건 중앙 정렬하는 마법의 코드**인 것은 아닙니다.

Box가 사용할 수 있는 공간과 크기, Formatting Context 등에 따라 결과가 달라집니다.

---

# 15. 가장 중요한 문제: `width`는 무엇의 크기인가?

다음 CSS를 보겠습니다.

```css
.box {
  width: 300px;
  padding: 20px;
  border: 10px solid black;
}
```

질문:

> 이 Box가 화면에서 차지하는 Border Box의 가로 크기는 300px일까요?

CSS의 디폴트 `box-sizing`은:

```css
box-sizing: content-box;
```

입니다.

따라서 여기서:

```css
width: 300px;
```

은 **Content Box의 width**를 지정합니다.

구조:

```text
             width: 300px
         ←────────────────→

         ┌────────────────┐
         │    Content     │
         └────────────────┘

      ↑                      ↑
 padding                  padding
  20px                     20px

   ↑                          ↑
 border                    border
  10px                       10px
```

따라서 Border Box의 실제 가로 크기는:

```text
300
+ 20 + 20
+ 10 + 10
───────────
= 360px
```

입니다.

---

# 16. `content-box`

디폴트 값입니다.

```css
box-sizing: content-box;
```

이 경우:

```css
.box {
  width: 300px;
  padding: 20px;
  border: 10px solid black;
}
```

에서:

```text
width
↓
Content Width
300px
```

입니다.

Border Box의 전체 너비는:

```text
Content
300px

+ Left Padding
20px

+ Right Padding
20px

+ Left Border
10px

+ Right Border
10px

────────────

Border Box Width
360px
```

입니다.

Margin은 Border Box 바깥의 공간이므로 이 `360px` 자체에는 포함되지 않습니다.

---

# 17. Height도 동일하다

세로 방향도 같은 원리가 적용됩니다.

```css
.box {
  height: 100px;

  padding-top: 20px;
  padding-bottom: 20px;

  border-top: 5px solid black;
  border-bottom: 5px solid black;
}
```

`content-box`라면:

```text
100
+ 20
+ 20
+ 5
+ 5
─────
150px
```

Border Box의 높이는 `150px`입니다.

---

# 18. `border-box`

실제 웹 개발에서는 다음 설정을 매우 많이 사용합니다.

```css
box-sizing: border-box;
```

이 경우:

```css
.box {
  box-sizing: border-box;

  width: 300px;
  padding: 20px;
  border: 10px solid black;
}
```

`width: 300px`는 **Border Box의 전체 width**를 지정합니다.

```text
       width: 300px
←────────────────────────→

┌─────────────────────────┐
│ Border                  │
│ ┌─────────────────────┐ │
│ │ Padding             │ │
│ │ ┌─────────────────┐ │ │
│ │ │     Content     │ │ │
│ │ └─────────────────┘ │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

따라서 Content Width는 내부적으로 줄어듭니다.

```text
전체 width
300px

- Left/Right Border
20px

- Left/Right Padding
40px

────────────

Content Width
240px
```

입니다.

---

# 19. `content-box`와 `border-box`

둘을 비교하면 Box Model의 핵심이 명확해집니다.

### content-box

```css
.box {
  box-sizing: content-box;

  width: 300px;
  padding: 20px;
  border: 10px solid;
}
```

```text
Content = 300px

Border Box
= 300 + 40 + 20
= 360px
```

### border-box

```css
.box {
  box-sizing: border-box;

  width: 300px;
  padding: 20px;
  border: 10px solid;
}
```

```text
Border Box = 300px

Content
= 300 - 40 - 20
= 240px
```

한눈에 비교하면:

```text
content-box

width
└── Content만 300px

결과 Border Box = 360px


border-box

width
└── Border까지 포함해서 300px

결과 Border Box = 300px
```

---

# 20. 왜 `border-box`를 많이 사용하는가?

다음과 같은 UI를 만든다고 하겠습니다.

```css
.card {
  width: 300px;
  padding: 20px;
  border: 1px solid gray;
}
```

개발자는 보통:

> "카드의 전체 너비를 300px로 만들고 싶다."

라고 생각합니다.

하지만 `content-box`에서는:

```text
300
+ 40 padding
+ 2 border
────────────
342px
```

가 됩니다.

`border-box`를 사용하면:

```css
.card {
  box-sizing: border-box;

  width: 300px;
  padding: 20px;
  border: 1px solid gray;
}
```

최종 Border Box width가:

```text
300px
```

가 됩니다.

따라서 Layout 크기를 계산하기가 훨씬 직관적입니다.

---

# 21. 많이 사용하는 전역 설정

그래서 실제 프로젝트에서는 다음과 같은 설정을 자주 볼 수 있습니다.

```css
* {
  box-sizing: border-box;
}
```

조금 더 명시적인 패턴으로는:

```css
html {
  box-sizing: border-box;
}

*,
*::before,
*::after {
  box-sizing: inherit;
}
```

처럼 작성하기도 합니다.

이렇게 하면 일반 Element뿐 아니라 `::before`, `::after`가 생성하는 Box에도 일관된 Box Sizing 전략을 적용할 수 있습니다.

---

# 22. Margin은 `width` 계산에 포함되는가?

다음 CSS를 보겠습니다.

```css
.box {
  box-sizing: border-box;

  width: 300px;
  margin: 20px;
}
```

`border-box`라고 하더라도 Margin은 `width: 300px` 안에 포함되지 않습니다.

```text
       Margin
         ↓

←20px→┌───────────────┐←20px→
      │               │
      │ Border Box    │
      │    300px      │
      │               │
      └───────────────┘
```

따라서 주변 공간까지 단순 합산하면:

```text
20 + 300 + 20
= 340px
```

의 수평 공간이 필요합니다.

핵심:

```text
border-box
→ Content + Padding + Border

Margin
→ Border Box 바깥
```

입니다.

---

# 23. Box 크기 계산 공식

`content-box`일 때 단순한 고정 길이 값만 고려하면 Border Box의 가로 크기는 다음과 같이 생각할 수 있습니다.

```text
Border Box Width
=
width
+ padding-left
+ padding-right
+ border-left-width
+ border-right-width
```

주변 Margin까지 포함하여 필요한 외부 공간을 단순 계산하면:

```text
Outer Space
=
margin-left
+ Border Box Width
+ margin-right
```

예:

```css
.box {
  width: 300px;

  padding: 20px;

  border: 5px solid;

  margin: 10px;
}
```

계산:

```text
Content
300

Padding
20 + 20

Border
5 + 5

────────────────

Border Box
350px


Margin
10 + 10

────────────────

주변 공간까지 단순 합산
370px
```

단, 실제 Layout에서는 `auto` Margin, Percentage, Margin Collapsing 등으로 인해 단순 합산만으로 설명할 수 없는 경우도 있습니다.

---

# 24. Margin Collapsing

Margin에는 Padding과 다른 중요한 특성이 있습니다.

Normal Flow의 Block Box들 사이에서 특정 조건이 충족되면 세로 Margin이 **Collapse(상쇄/병합)**될 수 있습니다.

예:

```html
<div class="box1">A</div>
<div class="box2">B</div>
```

CSS:

```css
.box1 {
  margin-bottom: 30px;
}

.box2 {
  margin-top: 20px;
}
```

처음에는 다음처럼 생각하기 쉽습니다.

```text
30px + 20px
= 50px
```

하지만 일반적인 인접 Block의 세로 Margin Collapsing 조건에서는 두 Margin이 합산되지 않고 더 큰 쪽이 사용될 수 있습니다.

```text
box1
┌─────────────┐
│      A      │
└─────────────┘

     30px

┌─────────────┐
│      B      │
└─────────────┘
box2
```

즉 이 경우 간격은 `50px`가 아니라 `30px`가 될 수 있습니다.

---

# 25. Margin Collapsing은 모든 Margin에서 발생하지 않는다

Margin Collapsing은 아무 Margin에서나 발생하는 것이 아닙니다.

대표적으로 **Normal Flow의 Block Box의 세로 Margin**에서 발생할 수 있습니다.

다음과 같은 경우에는 같은 방식으로 생각하면 안 됩니다.

```text
수평 margin
Flex Item의 margin
Grid Item의 margin
```

또한 부모와 첫 번째/마지막 자식 사이의 Margin 등에서도 조건에 따라 Collapsing이 발생할 수 있습니다.

입문 단계에서는:

> **일반적인 Block Layout에서 위아래 Margin은 경우에 따라 합쳐질 수 있다.**

정도로 먼저 이해하면 충분합니다.

---

# 26. `outline`은 Border와 다르다

Form이나 Focus 스타일을 다룰 때 `outline`을 자주 만나게 됩니다.

```css
input:focus {
  outline: 2px solid blue;
}
```

`outline`은 Border와 비슷해 보이지만 Box Model의 크기 계산에는 포함되지 않습니다.

```text
       Outline
    ┌───────────────┐
    │               │
    │ ┌───────────┐ │
    │ │  Border   │ │
    │ │  Content  │ │
    │ └───────────┘ │
    │               │
    └───────────────┘
```

즉:

```text
Content
Padding
Border
```

는 Box 크기 계산과 관계하지만:

```text
Outline
```

은 일반적인 Box Model 크기 계산에 추가되지 않습니다.

접근성 측면에서 Focus Outline을 무조건 제거하는 것도 피해야 합니다.

---

# 27. DevTools로 Box Model 확인하기

Box Model은 브라우저 개발자 도구에서 직접 확인하는 것이 가장 좋습니다.

예를 들어 Chrome DevTools에서 Element를 선택하면 다음과 같은 형태의 Box Model 정보를 볼 수 있습니다.

```text
┌──────── margin ────────┐
│                        │
│ ┌────── border ──────┐ │
│ │                    │ │
│ │ ┌── padding ─────┐ │ │
│ │ │                │ │ │
│ │ │    content     │ │ │
│ │ │    300 × 100   │ │ │
│ │ │                │ │ │
│ │ └────────────────┘ │ │
│ │                    │ │
│ └────────────────────┘ │
│                        │
└────────────────────────┘
```

CSS Layout 문제가 발생하면 가장 먼저 확인해야 할 것 중 하나가 Box Model입니다.

특히:

```text
width
height
padding
border
margin
box-sizing
```

을 확인합니다.

---

# 28. 실전 예제: Card 만들기

HTML:

```html
<div class="card">
  <h2>Keyboard</h2>
  <p>Mechanical Keyboard</p>
  <strong>50,000원</strong>
</div>
```

CSS:

```css
.card {
  box-sizing: border-box;

  width: 300px;

  padding: 20px;

  border: 1px solid #ccc;
  border-radius: 10px;

  margin: 20px;
}
```

Box Model로 분석하면:

```text
Margin
20px

┌────────────────────────────┐
│ Border                     │
│                            │
│   Padding 20px             │
│                            │
│   ┌────────────────────┐   │
│   │ Keyboard           │   │
│   │                    │   │
│   │ Mechanical         │   │
│   │ Keyboard           │   │
│   │                    │   │
│   │ 50,000원           │   │
│   └────────────────────┘   │
│                            │
└────────────────────────────┘

Border Box Width = 300px
```

이제 각각의 CSS Property가 무엇을 담당하는지 구분할 수 있습니다.

```text
width
→ Box 크기

padding
→ 내부 공간

border
→ 테두리

border-radius
→ 모서리

margin
→ 외부 공간
```

---

# 29. Box Model에서 자주 하는 실수

### 실수 1

```css
width: 300px;
padding: 20px;
```

를 작성하고 Box 전체가 무조건 `300px`이라고 생각하는 것.

디폴트 `content-box`에서는 그렇지 않습니다.

---

### 실수 2

Padding과 Margin을 같은 개념으로 생각하는 것.

```text
Padding → Border 안쪽

Margin → Border 바깥쪽
```

입니다.

---

### 실수 3

`border-box`에 Margin까지 포함된다고 생각하는 것.

포함되지 않습니다.

```text
border-box
=
Content
+ Padding
+ Border
```

Margin은 바깥쪽입니다.

---

### 실수 4

위아래 Margin을 항상 단순히 더하는 것.

Normal Flow의 Block Layout에서는 **Margin Collapsing**이 발생할 수 있습니다.

---

### 실수 5

HTML Element와 CSS Box를 완전히 동일한 개념으로 생각하는 것.

입문 단계에서는:

```text
Element → Box
```

로 이해해도 충분하지만 엄밀하게는 **Element가 CSS formatting 과정에서 Box를 생성한다**고 이해하는 것이 더 정확합니다.

---

# 30. HTML Element → Box → Layout

PART 3에서 가장 중요한 흐름입니다.

HTML:

```html
<div class="card">
  Card
</div>
```

DOM에는 Element가 존재합니다.

```text
DOM Tree

div.card
```

CSS가 적용되고 렌더링 과정에서 시각적 Box가 생성됩니다.

```text
HTML Element
      ↓
CSS Rules
      ↓
Computed Style
      ↓
CSS Box
```

Box는:

```text
Content
Padding
Border
Margin
```

이라는 영역을 가집니다.

그리고 브라우저는 다음 단계에서 이 Box들을 배치해야 합니다.

```text
Box A
Box B
Box C
```

바로 여기에서 **Layout** 개념이 본격적으로 시작됩니다.

---

# 31. Box Model과 Layout의 관계

Box Model은:

> **각 Box 자체의 크기와 내부/외부 영역을 이해하는 모델**

입니다.

Layout은:

> **그 Box들을 어디에 어떻게 배치할 것인가를 결정하는 과정**

입니다.

따라서:

```text
HTML Element
      ↓
CSS 적용
      ↓
Box 생성
      ↓
Box Model
크기와 영역
      ↓
Layout
Box의 위치와 배치
```

로 연결됩니다.

이 구분은 매우 중요합니다.

예를 들어:

```css
padding: 20px;
```

은 Box 내부 공간에 관한 것이고:

```css
display: flex;
```

는 자식 Box들의 배치 방식에 큰 영향을 주는 Layout 관련 설정입니다.

---

# 32. PART 3 핵심 정리

CSS를 이해할 때 HTML Element를 단순히 태그로만 생각하면 안 됩니다.

렌더링 과정에서 Element는 CSS formatting에 따라 Box를 생성합니다.

가장 기본적인 Box Model은:

```text
┌──────────── Margin ─────────────┐
│                                 │
│  ┌──────── Border ───────────┐  │
│  │                           │  │
│  │  ┌──── Padding ────────┐  │  │
│  │  │                     │  │  │
│  │  │      Content        │  │  │
│  │  │                     │  │  │
│  │  └─────────────────────┘  │  │
│  │                           │  │
│  └───────────────────────────┘  │
│                                 │
└─────────────────────────────────┘
```

입니다.

각 영역은:

```text
Content
→ 실제 콘텐츠

Padding
→ Content와 Border 사이의 내부 공간

Border
→ Box의 테두리

Margin
→ Border 바깥쪽의 외부 공간
```

을 의미합니다.

그리고 `box-sizing`은 `width`와 `height`가 어떤 Box를 기준으로 계산되는지를 결정합니다.

```text
content-box

width
  ↓
Content
+ Padding
+ Border
  ↓
실제 Border Box가 더 커질 수 있음
```

반면:

```text
border-box

width
  ↓
Content + Padding + Border
  ↓
Border Box 전체 크기
```

입니다.

따라서 실제 프로젝트에서는:

```css
* {
  box-sizing: border-box;
}
```

와 같은 설정을 자주 사용합니다.

---

# 33. 지금까지의 전체 흐름

PART 1부터 연결하면:

```text
PART 1
CSS Introduction
CSS는 무엇인가?
      ↓

PART 2
Selector & Cascade
어떤 Element에
어떤 스타일을 적용할 것인가?
      ↓

PART 3
Box Model
Element가 생성한 Box의
크기와 영역은 어떻게 구성되는가?
      ↓

PART 4
Size & Unit
크기를 어떤 단위와 기준으로
결정할 것인가?
      ↓

PART 5
Display & Normal Flow
Box는 어떤 특성을 가지고
어떻게 배치되는가?
```

CSS Layout을 이해하기 위한 핵심 흐름은 결국:

```text
HTML Element
      ↓
Selector
      ↓
Computed Style
      ↓
CSS Box
      ↓
Box Model
      ↓
Layout
      ↓
Screen
```

입니다.

**PART 3의 가장 중요한 한 문장**

> **Box Model은 HTML Element가 생성하는 CSS Box의 Content, Padding, Border, Margin 영역과 크기 계산 방식을 이해하기 위한 기본 모델이다.**

