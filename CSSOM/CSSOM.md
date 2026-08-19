CSSOM(CSS Object Model)은 DOM과 함께 브라우저 렌더링을 이해할 때 핵심 개념입니다. 다만 **CSSOM을 단순히 "CSS로 만든 트리"라고만 설명하면 정확하지 않습니다.** CSSOM은 본래 JavaScript가 CSS 스타일시트와 규칙을 객체로 다룰 수 있게 하는 객체 모델이고, 렌더링 과정에서는 이 스타일 정보가 DOM 요소에 적용되어 최종 스타일 계산에 사용됩니다.

## 1. CSSOM이란?

CSSOM은 **CSS Object Model**의 약자입니다.

예를 들어 브라우저가 다음 HTML과 CSS를 로드한다고 해보겠습니다.

```html
<h1 class="title">Hello</h1>
<p>Welcome</p>
```

```css
body {
    margin: 0;
}

.title {
    color: blue;
    font-size: 32px;
}

p {
    color: gray;
}
```

브라우저는 HTML을 파싱하여 DOM을 구성하고, CSS는 파싱하여 브라우저 내부에서 **스타일시트와 CSS 규칙을 구조화된 객체 형태로 표현**합니다.

개념적으로 보면:

```text
CSS
 │
 │ parsing
 ▼
CSSOM
 │
 ├── CSSStyleSheet
 │
 ├── CSSStyleRule
 │
 ├── CSSStyleDeclaration
 │
 └── ...
```

즉 CSSOM의 핵심은:

> **CSS 스타일시트와 CSS 규칙을 객체로 표현하여 브라우저와 JavaScript가 CSS를 구조적으로 다룰 수 있도록 하는 객체 모델**

입니다.

---

# 2. DOM과 CSSOM의 차이

DOM은 HTML 문서를 객체화합니다.

```html
<body>
    <h1>Hello</h1>
    <p>Welcome</p>
</body>
```

개념적으로:

```text
Document
   │
  html
   │
  body
  ├── h1
  └── p
```

반면 CSSOM은 CSS 스타일시트의 규칙을 객체로 다룹니다.

```css
body {
    margin: 0;
}

h1 {
    color: blue;
}
```

개념적으로:

```text
StyleSheet
 │
 ├── Rule
 │    ├── selector: body
 │    └── declaration
 │         └── margin: 0
 │
 └── Rule
      ├── selector: h1
      └── declaration
           └── color: blue
```

따라서 둘을 구분하면:

|         | DOM                        | CSSOM                      |
| ------- | -------------------------- | -------------------------- |
| 대상      | HTML/XML 문서                | CSS 스타일시트                  |
| 입력      | HTML                       | CSS                        |
| 핵심 객체   | `Document`, `Element`      | `CSSStyleSheet`, `CSSRule` |
| 목적      | 문서 구조 표현/조작                | CSS 규칙 표현/조작               |
| JS 접근 예 | `document.querySelector()` | `document.styleSheets`     |

---

# 3. 실제 CSSOM 객체를 확인해보자

브라우저에서 다음 CSS가 있다고 해봅시다.

```css
.title {
    color: blue;
    font-size: 32px;
}
```

JavaScript에서는:

```javascript
console.log(document.styleSheets);
```

를 통해 문서와 연결된 스타일시트들을 확인할 수 있습니다.

첫 번째 스타일시트를 가져오면:

```javascript
const sheet = document.styleSheets[0];

console.log(sheet);
```

일반적으로 `CSSStyleSheet` 객체입니다.

그리고:

```javascript
console.log(sheet.cssRules);
```

를 통해 CSS 규칙들을 볼 수 있습니다.

개념적으로:

```text
document
   │
   └── styleSheets
          │
          └── CSSStyleSheet
                  │
                  └── cssRules
                        │
                        └── CSSStyleRule
                              │
                              ├── selectorText
                              │
                              └── style
```

---

# 4. CSSStyleSheet

하나의 스타일시트를 표현하는 객체입니다.

예를 들어:

```html
<link rel="stylesheet" href="style.css">
```

또는:

```html
<style>
    h1 {
        color: red;
    }
</style>
```

에서 만들어지는 스타일시트가 `CSSStyleSheet`로 표현됩니다.

JavaScript:

```javascript
const sheet = document.styleSheets[0];
```

그리고:

```javascript
sheet.cssRules
```

를 통해 내부 규칙에 접근할 수 있습니다.

---

# 5. CSSStyleRule

다음 CSS를 생각해봅시다.

```css
.title {
    color: blue;
    font-size: 32px;
}
```

이 하나의 스타일 규칙은 CSSOM에서 `CSSStyleRule`로 표현됩니다.

개념적으로:

```text
CSSStyleRule
│
├── selectorText
│      ".title"
│
└── style
       │
       ├── color: "blue"
       └── font-size: "32px"
```

JavaScript로:

```javascript
const rule = document.styleSheets[0].cssRules[0];

console.log(rule.selectorText);
```

결과:

```text
.title
```

그리고:

```javascript
console.log(rule.style.color);
```

결과:

```text
blue
```

입니다.

---

# 6. CSSStyleDeclaration

CSS 선언부:

```css
{
    color: blue;
    font-size: 32px;
}
```

를 다루는 대표적인 객체가 `CSSStyleDeclaration`입니다.

예를 들어:

```javascript
const rule = document.styleSheets[0].cssRules[0];

console.log(rule.style);
```

여기서:

```javascript
rule.style
```

이 `CSSStyleDeclaration`입니다.

따라서:

```javascript
rule.style.color = "red";
```

처럼 CSS 규칙 자체를 변경할 수도 있습니다.

---

# 7. CSSOM과 `element.style`의 관계

HTML 요소에서도:

```javascript
const title = document.querySelector(".title");

console.log(title.style);
```

를 사용할 수 있습니다.

여기서도 `style`은 `CSSStyleDeclaration` 객체입니다.

하지만 중요한 차이가 있습니다.

```css
.title {
    color: blue;
}
```

```javascript
const title = document.querySelector(".title");

console.log(title.style.color);
```

이 경우 결과가 `"blue"`일 것 같지만 일반적으로:

```text
""
```

입니다.

왜냐하면:

```javascript
element.style
```

은 **해당 요소의 inline style 선언**을 주로 나타내기 때문입니다.

즉:

```html
<h1 class="title" style="color:red">
```

라면:

```javascript
title.style.color
```

결과는:

```text
red
```

입니다.

---

# 8. 그러면 최종 적용된 CSS는 어떻게 확인할까?

여기서 중요한 API가:

```javascript
getComputedStyle()
```

입니다.

```css
.title {
    color: blue;
    font-size: 32px;
}
```

```html
<h1 class="title">Hello</h1>
```

이라면:

```javascript
const title = document.querySelector(".title");

const style = getComputedStyle(title);

console.log(style.color);
console.log(style.fontSize);
```

와 같이 사용할 수 있습니다.

`getComputedStyle()`은 단순히 `.title` 규칙 하나를 읽는 것이 아닙니다.

브라우저는 여러 스타일 소스를 고려해야 합니다.

```text
Browser default style
        +
External CSS
        +
<style>
        +
inline style
        +
selector specificity
        +
!important
        +
inheritance
        +
cascade
        ↓
Computed Style
```

즉 해당 요소에 **계단식 규칙(Cascade)을 적용한 뒤 계산된 스타일**을 조회하는 것입니다.

---

# 9. 여기서 흔히 말하는 "CSSOM Tree"를 조심해야 한다

웹 성능이나 렌더링 설명에서 흔히 다음과 같은 그림을 봅니다.

```text
HTML → DOM Tree

CSS → CSSOM Tree

DOM + CSSOM
      ↓
 Render Tree
```

교육적으로 매우 유용한 모델입니다.

하지만 이것을 너무 문자 그대로 받아들이면 문제가 있습니다.

CSSOM의 본질은:

```text
CSS StyleSheet
       ↓
CSSStyleSheet
       ↓
CSSRule
       ↓
CSSStyleDeclaration
```

같은 **CSS 객체 모델**입니다.

그리고 렌더링 과정에서는 CSS 규칙을 DOM 요소에 매칭하고 cascade, inheritance 등을 적용해 각 요소의 스타일을 계산합니다.

따라서 좀 더 정확하게 표현하면:

```text
HTML
 │
 ▼
DOM
 │
 │
 ├──────────────┐
 │              │
CSS             │
 │              │
 ▼              │
CSSOM           │
 │              │
 └──────┬───────┘
        ▼
 Style Calculation
        │
        ▼
 styled elements
        │
        ▼
 Render Tree / Layout
```

라고 이해하는 것이 좋습니다.

---

# 10. CSS가 적용되는 과정

다음 예제를 보겠습니다.

```html
<body>
    <h1 class="title">Hello</h1>
</body>
```

CSS:

```css
body {
    color: black;
}

.title {
    color: blue;
    font-size: 32px;
}
```

브라우저는 먼저 CSS를 파싱합니다.

```text
CSS source
   │
   ▼
Tokenizer / Parser
   │
   ▼
CSS Rules
```

그리고 DOM 요소에 어떤 규칙이 적용되는지 판단합니다.

```text
DOM

body
 │
 └── h1.title
```

CSS:

```text
body
 → color:black

.title
 → color:blue
 → font-size:32px
```

그 후 selector matching이 이루어집니다.

```text
h1.title
    │
    ├── body로부터 상속 가능한 스타일 검토
    │
    └── .title 규칙 match
```

그리고 Cascade가 적용됩니다.

---

# 11. Cascade가 중요한 이유

같은 요소에 여러 CSS 규칙이 적용될 수 있습니다.

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

HTML:

```html
<h1 id="main-title" class="title">
    Hello
</h1>
```

브라우저는 단순히 "마지막 CSS 하나"를 선택하는 것이 아닙니다.

여러 조건을 고려합니다.

```text
Origin / Importance
        ↓
Cascade layers 등
        ↓
Specificity
        ↓
Scoping proximity 등
        ↓
Order of appearance
```

그 결과 최종적으로 해당 요소의 속성 값이 결정됩니다.

즉 CSSOM은 **단순 CSS 문자열 저장소가 아닙니다.**

CSS의 규칙들이 구조화되어 있고, 브라우저는 이를 DOM과 결합해 스타일을 계산합니다.

---

# 12. 상속(Inheritance)도 적용된다

예를 들어:

```css
body {
    color: blue;
}
```

HTML:

```html
<body>
    <div>
        <p>Hello</p>
    </div>
</body>
```

`color`는 상속되는 속성이므로 `<p>`에 직접:

```css
p {
    color: blue;
}
```

가 없어도 결과적으로 파란색 글자가 될 수 있습니다.

개념적으로:

```text
body
color: blue
   │
   │ inheritance
   ▼
div
color: blue
   │
   ▼
p
color: blue
```

반면 모든 CSS 속성이 상속되는 것은 아닙니다.

예를 들어 `margin`은 일반적으로 부모에서 자식으로 상속되지 않습니다.

---

# 13. DOM + CSSOM → Render Tree

이제 브라우저 렌더링 과정으로 연결됩니다.

HTML:

```html
<body>
    <h1>Hello</h1>
    <p>Welcome</p>
</body>
```

CSS:

```css
h1 {
    color: blue;
}

p {
    display: none;
}
```

DOM에는:

```text
body
├── h1
└── p
```

가 존재합니다.

하지만 `p`는:

```css
display: none;
```

이므로 일반적인 렌더링 구조에서는 박스를 생성하지 않습니다.

개념적으로:

```text
DOM + Style Information
        │
        ▼
Rendering structure

body
 │
 └── h1
```

가 됩니다.

따라서 **DOM Tree와 화면에 그려지는 구조는 동일하지 않습니다.**

---

# 14. `visibility: hidden`과 `display: none`도 차이가 있다

```css
p {
    visibility: hidden;
}
```

이면 요소가 보이지 않더라도 일반적으로 레이아웃 공간은 유지됩니다.

반면:

```css
p {
    display: none;
}
```

이면 해당 요소는 레이아웃 박스를 만들지 않습니다.

그래서:

```text
display: none
    ↓
layout box 없음

visibility: hidden
    ↓
layout box 존재
하지만 painting에서 보이지 않음
```

이라는 중요한 차이가 있습니다.

---

# 15. 이후 Layout과 Paint가 진행된다

전체 흐름을 연결하면 다음과 같습니다.

```text
HTML
 │
 ▼
Parsing
 │
 ▼
DOM
 │
 ├────────────────┐
 │                │
 │               CSS
 │                │
 │                ▼
 │              Parsing
 │                │
 │                ▼
 │              CSSOM
 │                │
 └───────┬────────┘
         ▼
   Style Calculation
         │
         ▼
 Rendering structure
         │
         ▼
       Layout
         │
         ▼
       Paint
         │
         ▼
     Compositing
         │
         ▼
       Screen
```

여기에서 역할을 구분하는 것이 중요합니다.

**Style Calculation**

```text
이 요소에 어떤 CSS 값이 적용되는가?
```

**Layout**

```text
이 요소의 크기와 위치는 어디인가?
```

**Paint**

```text
텍스트, 배경, 테두리 등을 어떻게 그릴 것인가?
```

**Compositing**

```text
여러 레이어를 어떤 순서로 합성해서 최종 화면을 만들 것인가?
```

입니다.

---

# 16. CSS는 왜 렌더링을 막을 수 있는가?

HTML에 다음이 있다고 해보겠습니다.

```html
<head>
    <link rel="stylesheet" href="style.css">
</head>
```

브라우저 입장에서는 CSS가 아직 로딩되지 않았다면:

```text
<h1>Hello</h1>
```

가

```css
h1 {
    display: none;
}
```

일 수도 있고:

```css
h1 {
    font-size: 100px;
}
```

일 수도 있습니다.

따라서 필요한 스타일 정보를 확보하고 계산하지 않고 무작정 화면을 그리면 잘못된 화면을 먼저 보여줄 수 있습니다.

이런 이유 때문에 CSS는 일반적으로 **render-blocking resource**라고 설명합니다.

---

# 17. JavaScript에서 CSSOM을 조작할 수도 있다

CSSOM은 Object Model이므로 JavaScript에서 접근하고 변경할 수 있습니다.

예를 들어:

```javascript
const sheet = document.styleSheets[0];

sheet.insertRule(
    ".warning { color: red; font-weight: bold; }",
    sheet.cssRules.length
);
```

새 CSS 규칙을 추가할 수 있습니다.

삭제도 가능합니다.

```javascript
sheet.deleteRule(0);
```

기존 규칙을 수정할 수도 있습니다.

```javascript
const rule = sheet.cssRules[0];

rule.style.color = "green";
```

따라서:

```text
CSS file
   ↓
CSSOM
   ↕
JavaScript
```

라는 관계가 성립합니다.

---

# 18. DOM 조작과 CSSOM 조작의 차이

JavaScript에서:

```javascript
const title = document.querySelector(".title");

title.textContent = "Hello";
```

는 DOM을 변경합니다.

반면:

```javascript
title.style.color = "red";
```

는 해당 요소의 inline style 선언을 변경합니다.

또:

```javascript
document.styleSheets[0]
        .cssRules[0]
        .style.color = "red";
```

처럼 스타일시트의 CSS 규칙 자체를 변경할 수도 있습니다.

개념적으로:

```text
JavaScript
    │
    ├── DOM API
    │     ↓
    │   HTML 구조/내용 변경
    │
    └── CSSOM API
          ↓
        CSS 규칙/스타일 변경
```

입니다.

---

# 19. CSSOM을 window와 연결하면

앞서 설명한 `window` 객체와도 연결할 수 있습니다.

```text
window
 │
 └── document
       │
       ├── DOM
       │    │
       │    └── Element
       │
       └── styleSheets
              │
              └── CSSStyleSheet
                    │
                    └── CSSRule
```

그리고:

```javascript
window.getComputedStyle(element)
```

을 통해 특정 요소에 계산된 스타일을 조회할 수 있습니다.

보통 `window.`를 생략해서:

```javascript
getComputedStyle(element)
```

이라고 작성합니다.

---

# 20. 학생들에게는 이 세 개를 반드시 구분시키는 것이 좋다

### DOM

> HTML 문서를 객체 모델로 표현한다.

```text
HTML
 ↓
DOM
```

### CSSOM

> CSS 스타일시트와 CSS 규칙을 객체 모델로 표현한다.

```text
CSS
 ↓
CSSOM
```

### Render Tree / 렌더링 구조

> DOM 요소와 계산된 스타일 정보를 이용해 화면 렌더링에 필요한 구조를 만든다.

```text
DOM ──────┐
          ├─→ Style Calculation
CSSOM ────┘
                 ↓
          Rendering structure
                 ↓
              Layout
                 ↓
              Paint
```

이 셋을 모두 "`Tree`"라고 뭉뚱그려 설명하면 오히려 학생들이 혼동하기 쉽습니다.

---

# 21. 최종적으로 기억할 구조

CSSOM을 브라우저 전체 흐름 안에 넣으면 다음 그림이 핵심입니다.

```text
                    WEB BROWSER
                         │
           ┌─────────────┴─────────────┐
           │                           │
         HTML                         CSS
           │                           │
         Parser                      Parser
           │                           │
           ▼                           ▼
          DOM                        CSSOM
           │                           │
           └─────────────┬─────────────┘
                         │
                         ▼
                Style Calculation
                         │
                         ▼
               Rendering Structure
                         │
                         ▼
                       Layout
                  위치 / 크기 계산
                         │
                         ▼
                       Paint
                  실제 그리기 정보
                         │
                         ▼
                    Compositing
                         │
                         ▼
                       Screen
```

### 한 문장으로 정의한다면

> **CSSOM(CSS Object Model)은 브라우저가 CSS 스타일시트와 그 안의 CSS 규칙을 객체 형태로 표현하고 JavaScript가 이를 조회·조작할 수 있도록 정의한 객체 모델이며, 브라우저는 이 스타일 정보를 DOM 요소와 결합하여 각 요소의 최종 스타일을 계산하고 렌더링에 사용한다.**

특히 **`CSS → CSSOM → DOM과 규칙 매칭 → Cascade/Inheritance → Computed Style → Layout → Paint`**라는 흐름으로 이해하면 CSSOM뿐 아니라 이후 브라우저 렌더링 파이프라인까지 한꺼번에 연결됩니다.
