# PART 2. CSS Selector & Cascade

## 1. Selector란?

CSS는 HTML Element에 스타일을 적용합니다.

그런데 브라우저는 먼저 다음 질문에 답할 수 있어야 합니다.

> **이 CSS Rule을 어떤 HTML Element에 적용할 것인가?**

이때 사용하는 것이 **Selector(선택자)**입니다.

```css
p {
  color: blue;
}
```

여기서:

```text
p             → Selector

color: blue;  → Declaration
```

입니다.

HTML이 다음과 같다면:

```html
<h1>CSS 강의</h1>
<p>첫 번째 문단입니다.</p>
<p>두 번째 문단입니다.</p>
```

`p` Selector는 두 개의 `<p>` Element를 모두 선택합니다.

```text
HTML

<h1>CSS 강의</h1>

<p>첫 번째 문단입니다.</p>  ◀──┐
                             │
<p>두 번째 문단입니다.</p>  ◀──┤
                             │
CSS                          │
                             │
p { ─────────────────────────┘
  color: blue;
}
```

즉:

> **Selector = CSS를 적용할 HTML Element를 선택하는 문법**

입니다.

---

# 2. 기본 Selector

가장 먼저 알아야 할 Selector는 다음 세 가지입니다.

```text
Element Selector
Class Selector
ID Selector
```

---

# 3. Element Selector

Element의 태그 이름으로 선택합니다.

```css
p {
  color: blue;
}
```

HTML:

```html
<p>Hello</p>
<p>CSS</p>
```

두 `<p>`가 모두 선택됩니다.

```text
p
│
├── <p>Hello</p>
│
└── <p>CSS</p>
```

다른 Element도 동일합니다.

```css
h1 {
  font-size: 40px;
}

button {
  background-color: blue;
}
```

Element Selector는 해당 종류의 Element 전체에 공통 스타일을 적용할 때 적합합니다.

---

# 4. Class Selector

실제 CSS 작성에서 매우 많이 사용하는 Selector입니다.

HTML의 `class` Attribute를 이용합니다.

```html
<h1 class="title">CSS 강의</h1>
```

CSS에서는 `.`을 사용합니다.

```css
.title {
  color: blue;
}
```

관계를 보면:

```text
HTML

class="title"
       │
       ▼
     title
       ▲
       │
CSS    │

.title {
  color: blue;
}
```

중요한 점은 CSS에서 `.`이 class 이름 자체에 포함되는 것이 아니라 **Class Selector를 나타내는 문법**이라는 것입니다.

HTML:

```html
class="title"
```

CSS:

```css
.title
```

---

# 5. 하나의 Class는 여러 Element에서 사용할 수 있다

Class의 중요한 특징입니다.

```html
<h2 class="important">공지사항</h2>

<p class="important">
  반드시 확인하세요.
</p>
```

CSS:

```css
.important {
  color: red;
}
```

결과:

```text
.important
     │
     ├── <h2 class="important">
     │
     └── <p class="important">
```

두 Element 모두 선택됩니다.

따라서 Class는:

> **여러 Element에 동일한 스타일을 재사용하기 위한 핵심 수단**

입니다.

---

# 6. 하나의 Element는 여러 Class를 가질 수 있다

반대도 가능합니다.

```html
<button class="btn primary">
  저장
</button>
```

여기에는 두 개의 Class가 있습니다.

```text
class="btn primary"
       │     │
       │     └── primary
       │
       └── btn
```

CSS:

```css
.btn {
  padding: 10px 20px;
  border-radius: 6px;
}

.primary {
  background-color: blue;
  color: white;
}
```

해당 `<button>`에는 두 Rule의 스타일이 모두 적용될 수 있습니다.

```text
.btn
 padding
 border-radius
       │
       ▼

   <button>
      저장

       ▲
       │
.primary
 background-color
 color
```

이러한 방식은 실제 웹 개발에서 매우 많이 사용됩니다.

---

# 7. ID Selector

HTML의 `id` Attribute를 이용하여 Element를 선택합니다.

HTML:

```html
<header id="main-header">
  Header
</header>
```

CSS:

```css
#main-header {
  background-color: black;
  color: white;
}
```

ID Selector는 `#`을 사용합니다.

```text
HTML

id="main-header"
       │
       ▼
   main-header
       ▲
       │
CSS    │

#main-header
```

정리하면:

```text
Element Selector     p

Class Selector       .title

ID Selector          #header
```

입니다.

---

# 8. Class와 ID의 차이

초보자가 자주 혼동하는 부분입니다.

HTML:

```html
<div id="header"></div>

<div class="card"></div>
<div class="card"></div>
<div class="card"></div>
```

개념적으로:

```text
ID

#header
   │
   └── 특정 Element 식별


Class

.card
   │
   ├── Element
   ├── Element
   └── Element
```

`id` 값은 문서 안에서 고유해야 합니다.

반면 Class는 여러 Element에서 반복해서 사용할 수 있습니다.

CSS 스타일링에서는 일반적으로 **Class 중심으로 작성하는 것이 관리하기 좋습니다.**

---

# 9. Universal Selector

모든 Element를 선택하려면 `*`을 사용합니다.

```css
* {
  box-sizing: border-box;
}
```

`*`를 **Universal Selector**라고 합니다.

개념적으로:

```text
*
│
├── html
├── body
├── h1
├── div
├── p
├── button
└── ...
```

처럼 모든 Element를 대상으로 합니다.

실제 CSS에서는 다음 코드가 자주 사용됩니다.

```css
* {
  box-sizing: border-box;
}
```

`box-sizing`은 Box Model PART에서 자세히 다룹니다.

---

# 10. Grouping Selector

여러 Selector에 같은 스타일을 적용하고 싶을 수 있습니다.

다음처럼 반복할 수도 있습니다.

```css
h1 {
  color: blue;
}

h2 {
  color: blue;
}

h3 {
  color: blue;
}
```

하지만 `,`를 이용하여 묶을 수 있습니다.

```css
h1,
h2,
h3 {
  color: blue;
}
```

이를 **Selector List**라고 합니다.

```text
h1 ──┐
h2 ──┼──→ color: blue
h3 ──┘
```

---

# 11. Selector를 조합할 수 있다

CSS Selector는 하나만 사용하는 것이 아닙니다.

HTML:

```html
<p class="message">Hello</p>
```

다음 Selector를 사용할 수 있습니다.

```css
.message {
  color: blue;
}
```

또는:

```css
p.message {
  color: blue;
}
```

`p.message`는:

> **p Element이면서 동시에 message Class를 가진 Element**

를 의미합니다.

```text
p.message

p
AND
class="message"
```

HTML:

```html
<p class="message">선택됨</p>

<div class="message">선택되지 않음</div>
```

`p.message`는 첫 번째 Element만 선택합니다.

---

# 12. Descendant Combinator

HTML은 부모와 자식 관계를 가진 Tree 구조입니다.

```html
<div class="card">
  <h2>Keyboard</h2>

  <div>
    <p>가격: 50,000원</p>
  </div>
</div>
```

구조:

```text
.card
  │
  ├── h2
  │
  └── div
       │
       └── p
```

`.card` 내부에 존재하는 모든 `p`를 선택하려면:

```css
.card p {
  color: gray;
}
```

처럼 공백을 사용합니다.

```text
.card p
 │    │
 │    └── 후손 중 p
 │
 └── .card 내부
```

여기서 중요한 것은 **직접적인 자식만 의미하는 것이 아니라 후손(Descendant) 전체를 대상으로 한다는 것**입니다.

---

# 13. Child Combinator

직접적인 자식만 선택하려면 `>`를 사용합니다.

```css
.card > p {
  color: blue;
}
```

HTML:

```html
<div class="card">

  <p>A</p>

  <div>
    <p>B</p>
  </div>

</div>
```

Tree:

```text
.card
  │
  ├── p A       ← 선택
  │
  └── div
       │
       └── p B  ← 선택되지 않음
```

따라서:

```text
.card p
```

와:

```text
.card > p
```

는 다릅니다.

```text
.card p
→ 모든 후손 p


.card > p
→ 직접적인 자식 p
```

---

# 14. Sibling Combinator

같은 부모를 가진 Element를 **Sibling(형제)**이라고 합니다.

```html
<h2>제목</h2>
<p>첫 번째 문단</p>
<p>두 번째 문단</p>
```

Tree:

```text
Parent
 │
 ├── h2
 ├── p
 └── p
```

모두 같은 부모의 자식이므로 서로 Sibling입니다.

---

# 15. Next-Sibling Combinator

`+`는 특정 Element 바로 다음에 있는 Sibling을 선택합니다.

```css
h2 + p {
  color: blue;
}
```

HTML:

```html
<h2>제목</h2>

<p>첫 번째 문단</p>

<p>두 번째 문단</p>
```

결과:

```text
h2
 │
 └── 바로 다음 p
        ↑
      선택
```

첫 번째 `<p>`만 선택됩니다.

---

# 16. Subsequent-Sibling Combinator

`~`는 앞의 Element 뒤에 나오는 같은 부모의 해당 Sibling들을 선택합니다.

```css
h2 ~ p {
  color: blue;
}
```

HTML:

```html
<h2>제목</h2>

<p>첫 번째 문단</p>
<p>두 번째 문단</p>
```

두 `<p>` 모두 선택됩니다.

```text
h2
 │
 ├── p ← 선택
 │
 └── p ← 선택
```

정리하면:

```text
A B
→ A의 후손 B

A > B
→ A의 직접 자식 B

A + B
→ A 바로 다음 형제 B

A ~ B
→ A 뒤에 나오는 형제 B
```

---

# 17. Attribute Selector

HTML Attribute를 이용해서도 Element를 선택할 수 있습니다.

HTML:

```html
<input type="text">
<input type="password">
```

CSS:

```css
input[type="text"] {
  border: 1px solid blue;
}
```

의미는:

```text
input
AND
type="text"
```

입니다.

즉:

```html
<input type="text">
```

만 선택됩니다.

기본적인 형태는 다음과 같습니다.

```css
[disabled] {
}

[type="text"] {
}

input[type="email"] {
}
```

---

# 18. Pseudo-class

Element의 **상태나 구조적 조건**을 기준으로 선택할 수도 있습니다.

대표적인 예가 `:hover`입니다.

```css
button:hover {
  background-color: blue;
}
```

의미:

```text
button
  │
  │ 마우스가 올라감
  ▼
button:hover
```

평상시:

```text
[ 저장 ]
```

마우스를 올리면:

```text
[ 저장 ] ← background 변경
```

다른 대표적인 Pseudo-class에는 다음과 같은 것들이 있습니다.

```css
a:hover {
}

input:focus {
}

button:disabled {
}

li:first-child {
}

li:last-child {
}

li:nth-child(2) {
}
```

Pseudo-class는 `:` 하나를 사용합니다.

---

# 19. Pseudo-element

Pseudo-element는 Element의 **특정 부분이나 가상으로 생성되는 부분**을 선택합니다.

대표적으로:

```css
p::first-letter {
  font-size: 30px;
}
```

첫 글자에 스타일을 적용할 수 있습니다.

또한:

```css
.title::before {
  content: "★ ";
}
```

```css
.title::after {
  content: " !";
}
```

처럼 사용할 수도 있습니다.

개념적으로:

```text
.title

       CSS가 생성
           ↓

::before  실제 콘텐츠  ::after
```

대표적인 Pseudo-element:

```css
::before
::after
::first-letter
::first-line
::selection
```

Pseudo-element는 일반적으로 `::`를 사용합니다.

---

# 20. Pseudo-class와 Pseudo-element

둘을 구분해야 합니다.

```text
Pseudo-class
:
Element의 상태 또는 조건

:hover
:focus
:first-child


Pseudo-element
::
Element의 특정 부분 또는 생성된 부분

::before
::after
::first-letter
```

예:

```css
button:hover {
}
```

```css
.title::before {
}
```

---

# 21. 하나의 Element에 여러 Rule이 적용될 수 있다

HTML:

```html
<h1 id="main-title" class="title">
  CSS
</h1>
```

CSS:

```css
h1 {
  color: red;
}

.title {
  color: blue;
}

#main-title {
  color: green;
}
```

세 Selector 모두 같은 `<h1>`을 선택합니다.

```text
h1 ───────────┐
              │
.title ───────┼──→ <h1 id="main-title" class="title">
              │
#main-title ──┘
```

그런데 세 Rule 모두 `color`를 지정하고 있습니다.

```text
red
blue
green
```

그렇다면 브라우저는 어떤 값을 사용해야 할까요?

여기서 **Cascade**가 필요합니다.

---

# 22. Cascade란?

CSS의 이름 자체가:

> **Cascading Style Sheets**

입니다.

여러 CSS Declaration이 같은 Element의 같은 Property에 영향을 줄 때 브라우저는 **Cascade 알고리즘**을 이용하여 최종적으로 적용될 Declaration을 결정합니다.

개념적으로:

```text
여러 CSS Declaration
        │
        ▼
      Cascade
        │
        ▼
우선할 Declaration 결정
        │
        ▼
   최종 스타일
```

단순히 **“아래에 작성한 CSS가 무조건 이긴다”**라고 이해하면 안 됩니다.

브라우저는 여러 조건을 고려합니다.

입문 단계에서는 다음 요소들을 기억하면 됩니다.

```text
Origin / Importance
        ↓
Specificity
        ↓
Scope proximity 등
        ↓
Source Order
        ↓
최종 Declaration
```

실제 Cascade에는 cascade layer나 scoping proximity 같은 요소도 존재하지만, 처음 CSS를 학습할 때는 **Importance → Specificity → Source Order**를 중심으로 이해하면 충분합니다.

---

# 23. Specificity란?

**Specificity(명시도)**는 여러 Selector가 같은 Element에 적용될 때 **Selector가 대상을 얼마나 구체적으로 지정하는지**를 비교하기 위한 개념입니다.

예를 들어:

```html
<h1 id="main-title" class="title">
  CSS
</h1>
```

다음 세 Selector가 모두 이 Element를 선택합니다.

```css
h1 {
  color: red;
}

.title {
  color: blue;
}

#main-title {
  color: green;
}
```

일반적인 우선 관계는:

```text
ID Selector
     >
Class / Attribute / Pseudo-class
     >
Element / Pseudo-element
```

입니다.

따라서 위 예제에서는:

```css
#main-title {
  color: green;
}
```

이 더 높은 Specificity를 가집니다.

---

# 24. Specificity를 숫자로 이해하기

교육 목적으로 다음처럼 생각하면 이해하기 쉽습니다.

```text
ID | CLASS | TYPE
```

예를 들어:

```css
p
```

는:

```text
0 | 0 | 1
```

`.message`는:

```text
0 | 1 | 0
```

`#header`는:

```text
1 | 0 | 0
```

입니다.

조합하면:

```css
.card p
```

```text
0 | 1 | 1
```

이고:

```css
#main .card p
```

는:

```text
1 | 1 | 1
```

입니다.

각 열을 **독립적으로 왼쪽부터 비교**한다고 이해하는 것이 좋습니다.

주의할 점은 이를 단순한 십진수 점수처럼 계산하는 규칙으로 생각하면 안 된다는 것입니다.

---

# 25. Source Order

Specificity까지 같다면 뒤에 선언된 Rule이 우선할 수 있습니다.

```css
.title {
  color: red;
}

.title {
  color: blue;
}
```

두 Selector의 Specificity가 같습니다.

따라서 뒤에 있는:

```css
.title {
  color: blue;
}
```

가 적용됩니다.

```text
Specificity 동일
       │
       ▼
Source Order 비교
       │
       ▼
뒤의 Declaration
```

따라서:

> **“나중에 작성한 CSS가 이긴다”는 것은 앞선 Cascade 조건들이 동일할 때의 이야기**

입니다.

---

# 26. Inline Style

Inline Style도 Cascade에 영향을 줍니다.

```html
<h1
  id="title"
  class="title"
  style="color: purple;"
>
  CSS
</h1>
```

작성자 스타일 내부의 일반적인 Rule과 비교하면 Inline Style은 매우 높은 우선순위를 가집니다.

```css
#title {
  color: green;
}
```

이 있어도 일반적인 상황에서는 Inline Style의:

```css
color: purple;
```

이 적용됩니다.

그래서 스타일을 Inline으로 남발하면 CSS를 관리하고 Override하기 어려워질 수 있습니다.

---

# 27. `!important`

CSS Declaration 뒤에:

```css
!important
```

를 붙일 수도 있습니다.

```css
.title {
  color: red !important;
}
```

하지만 이것을 단순히:

> "`!important`가 있으면 무조건 모든 CSS를 이긴다"

라고 이해하면 정확하지 않습니다.

`!important`는 Cascade에서 **Importance를 변경하는 기능**이며, 중요한 Declaration들 사이에서도 Origin, Layer, Specificity 등의 규칙에 따라 다시 비교가 이루어집니다.

따라서 일반적인 애플리케이션 CSS에서는 남발하지 않는 것이 좋습니다.

```text
!important 남발
      ↓
Override 어려움
      ↓
CSS 유지보수 어려움
```

---

# 28. Inheritance

CSS에는 **Inheritance(상속)**라는 개념도 있습니다.

HTML:

```html
<div class="container">
  <p>Hello CSS</p>
</div>
```

CSS:

```css
.container {
  color: blue;
}
```

`<p>`에 직접 `color`를 지정하지 않았지만 글자가 파란색으로 표시될 수 있습니다.

```text
div.container
 color: blue
      │
      │ inheritance
      ▼
      p
   color: blue
```

부모 Element의 일부 CSS Property 값이 자식 Element로 전달되기 때문입니다.

하지만 **모든 Property가 상속되는 것은 아닙니다.**

---

# 29. 상속되는 Property와 그렇지 않은 Property

대표적으로 텍스트와 관련된 많은 Property는 상속됩니다.

```css
color
font-family
font-size
font-weight
line-height
```

반면 Box와 Layout에 관련된 많은 Property는 기본적으로 상속되지 않습니다.

```css
width
height
margin
padding
border
```

예를 들어:

```css
.parent {
  border: 5px solid red;
}
```

라고 했다고 해서 자식 Element에 동일한 `border`가 자동으로 생기지는 않습니다.

개념적으로:

```text
color
font
  │
  └── 상속되는 경우가 많음


margin
padding
border
width
  │
  └── 기본적으로 상속되지 않음
```

Property마다 상속 여부가 정해져 있으므로 필요할 때 해당 Property의 정의를 확인하는 습관이 중요합니다.

---

# 30. Cascade와 Inheritance의 차이

두 개념을 혼동하기 쉽습니다.

**Cascade**는:

> 여러 Declaration 후보 중 어떤 Declaration이 이 Element에 적용될지를 결정하는 과정

입니다.

**Inheritance**는:

> 부모의 계산된 값이 상속 가능한 Property에 대해 자식에게 전달되는 메커니즘

입니다.

```text
Cascade
──────────────
여러 스타일 후보
      ↓
Element에 적용할 값 결정


Inheritance
──────────────
Parent의 값
      ↓
Child로 전달
```

둘은 서로 다른 개념입니다.

---

# 31. 실전 예제

HTML:

```html
<div class="card">
  <h2 class="title">Keyboard</h2>
  <p class="price">50,000원</p>
  <button>구매</button>
</div>
```

CSS:

```css
.card {
  width: 300px;
  padding: 20px;
}

.card .title {
  color: navy;
}

.card .price {
  font-weight: bold;
}

.card button {
  padding: 10px 20px;
}

.card button:hover {
  background-color: navy;
  color: white;
}
```

Selector를 분석해 보겠습니다.

```text
.card
→ card Class를 가진 Element


.card .title
→ .card 내부의 .title


.card .price
→ .card 내부의 .price


.card button
→ .card 내부의 button


.card button:hover
→ .card 내부의 button 중
  현재 :hover 상태인 Element
```

이처럼 실제 CSS에서는 여러 Selector를 조합하여 원하는 Element를 정확하게 선택합니다.

---

# 32. Selector 전체 구조 정리

지금까지 배운 내용을 한 번에 정리하면 다음과 같습니다.

```text
CSS Selector
│
├── Basic
│    ├── *
│    ├── p
│    ├── .class
│    └── #id
│
├── Combination
│    ├── p.message
│    └── h1, h2, h3
│
├── Combinator
│    ├── A B
│    ├── A > B
│    ├── A + B
│    └── A ~ B
│
├── Attribute
│    └── input[type="text"]
│
├── Pseudo-class
│    ├── :hover
│    ├── :focus
│    └── :nth-child()
│
└── Pseudo-element
     ├── ::before
     ├── ::after
     └── ::first-letter
```

---

# 33. CSS가 적용되는 전체 흐름

PART 1과 PART 2를 연결하면 CSS를 다음과 같이 이해할 수 있습니다.

```text
HTML Elements
      │
      ▼
Selector
어떤 Element인가?
      │
      ▼
Matching Rules
적용 가능한 CSS Rule 수집
      │
      ▼
Cascade
충돌하는 Declaration 결정
      │
      ▼
Inheritance
필요한 상속 값 처리
      │
      ▼
Computed Style
Element의 스타일 결정
      │
      ▼
Layout / Paint
```

즉 Selector만 안다고 CSS 적용 과정 전체를 이해한 것은 아닙니다.

핵심은:

> **Selector로 Element를 찾고 → Cascade로 충돌을 해결하고 → Inheritance를 고려하여 최종 스타일이 결정된다.**

입니다.

---

# 34. PART 2 핵심 정리

Selector는:

> **CSS Rule을 적용할 HTML Element를 선택하는 문법**

입니다.

가장 기본적인 Selector는:

```text
p            Element Selector

.title       Class Selector

#header      ID Selector
```

입니다.

관계에 따라:

```text
A B          Descendant

A > B        Child

A + B        Next Sibling

A ~ B        Subsequent Sibling
```

을 사용할 수 있습니다.

상태나 특정 부분을 대상으로 할 때는:

```text
:hover       Pseudo-class

::before     Pseudo-element
```

등을 사용합니다.

그러나 Selector의 역할은 **대상을 선택하는 것**까지입니다.

같은 Element의 같은 Property를 여러 Declaration이 대상으로 하면 **Cascade**가 최종 Declaration을 결정합니다.

```text
Selector
어디에 적용할까?
      ↓
Cascade
충돌하면 무엇을 사용할까?
      ↓
Inheritance
부모에게서 받을 값은 있는가?
      ↓
Computed Style
최종 스타일
```

그리고 CSS를 이해하는 전체 흐름에서 현재 위치는 다음과 같습니다.

```text
PART 1
CSS Introduction
      ↓
PART 2
Selector & Cascade
      ↓
PART 3
Box Model
      ↓
PART 4
Size & Unit
      ↓
PART 5
Display & Normal Flow
      ↓
Flexbox / Grid / Responsive
```

다음 PART에서는 Selector를 통해 선택된 HTML Element가 화면에서 어떤 **Box**로 다뤄지는지 알아봅니다.

```text
HTML Element
      ↓
Selector
      ↓
CSS 적용
      ↓
┌─────────────────┐
│      margin     │
│  ┌───────────┐  │
│  │  border   │  │
│  │ ┌───────┐ │  │
│  │ │padding│ │  │
│  │ │content│ │  │
│  │ └───────┘ │  │
│  └───────────┘  │
└─────────────────┘
      ↓
PART 3. Box Model
```
