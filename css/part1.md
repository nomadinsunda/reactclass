# PART 1. CSS Introduction

## 1. CSS란?

**CSS(Cascading Style Sheets)**는 HTML로 작성된 문서의 **모양(Style)과 배치(Layout)**를 정의하는 언어입니다.

HTML이 웹 페이지의 **구조와 의미**를 담당한다면, CSS는 그 구조를 화면에 **어떻게 보여줄 것인지** 담당합니다.

```text
HTML
웹 페이지의 구조와 의미
        │
        ▼
CSS
모양과 레이아웃
        │
        ▼
Browser
화면에 렌더링
```

예를 들어 다음 HTML이 있다고 하겠습니다.

```html
<h1>Hello CSS</h1>
<p>CSS를 공부합니다.</p>
```

HTML만 사용하면 브라우저의 디폴트 스타일로 화면에 표시됩니다.

여기에 CSS를 적용할 수 있습니다.

```css
h1 {
  color: blue;
  font-size: 40px;
}

p {
  color: gray;
}
```

HTML의 구조는 그대로이지만 화면에 표시되는 모습은 달라집니다.

즉,

> **HTML = 무엇을 표시할 것인가**
> **CSS = 그것을 어떻게 표시할 것인가**

라고 이해할 수 있습니다.

---

# 2. HTML과 CSS의 역할

HTML과 CSS의 역할을 조금 더 정확하게 구분해 보겠습니다.

HTML:

```html
<h1>상품 목록</h1>

<div class="product">
  <h2>Keyboard</h2>
  <p>가격: 50,000원</p>
</div>
```

HTML은 다음과 같은 **문서 구조**를 만듭니다.

```text
상품 목록
   │
   └── 상품
        ├── 상품명
        └── 가격
```

CSS는 이 구조에 시각적인 표현을 적용합니다.

```css
.product {
  width: 300px;
  padding: 20px;
  border: 1px solid gray;
  border-radius: 10px;
}
```

역할을 정리하면 다음과 같습니다.

| 기술         | 주요 역할                             |
| ---------- | --------------------------------- |
| HTML       | 구조(Structure), 의미(Semantics), 콘텐츠 |
| CSS        | 스타일(Style), 크기, 간격, 배치(Layout)    |
| JavaScript | 동작(Behavior), 이벤트, 데이터 처리         |

따라서 웹 페이지를 크게 보면:

```text
HTML        CSS          JavaScript
구조         표현           동작
 │            │             │
 └────────────┼─────────────┘
              ▼
          Web Page
```

라고 생각할 수 있습니다.

---

# 3. CSS는 무엇을 변경할 수 있는가?

CSS는 단순히 글자의 색상을 변경하는 언어가 아닙니다.

CSS를 사용하면 크게 다음과 같은 것들을 제어할 수 있습니다.

### 텍스트

```css
.title {
  color: blue;
  font-size: 32px;
  font-weight: bold;
  text-align: center;
}
```

### 배경

```css
.card {
  background-color: lightgray;
}
```

### 크기

```css
.card {
  width: 300px;
  height: 200px;
}
```

### 간격

```css
.card {
  margin: 20px;
  padding: 20px;
}
```

### 테두리

```css
.card {
  border: 1px solid gray;
  border-radius: 10px;
}
```

### 레이아웃

```css
.container {
  display: flex;
  gap: 20px;
}
```

또는:

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}
```

따라서 CSS는 크게:

```text
CSS
 │
 ├── Style
 │    ├── color
 │    ├── background
 │    ├── font
 │    └── border
 │
 ├── Size / Spacing
 │    ├── width / height
 │    ├── margin
 │    └── padding
 │
 └── Layout
      ├── display
      ├── position
      ├── Flexbox
      └── Grid
```

를 담당한다고 볼 수 있습니다.

---

# 4. CSS 기본 문법

CSS의 가장 기본적인 형태는 다음과 같습니다.

```css
h1 {
  color: blue;
  font-size: 40px;
}
```

구조를 분해하면:

```text
h1 {
  color: blue;
}
│     │    │
│     │    └── Value
│     │
│     └── Property
│
└── Selector
```

정확히는 다음과 같습니다.

```text
h1                → Selector
color             → Property
blue              → Value
color: blue;      → Declaration
{ ... }           → Declaration Block
```

전체:

```css
h1 {
  color: blue;
  font-size: 40px;
}
```

를 하나의 **CSS Rule(Ruleset)**이라고 부릅니다.

---

# 5. Selector

Selector는 CSS를 적용할 **HTML Element를 선택하는 부분**입니다.

예를 들어:

```html
<h1>Hello</h1>
<p>CSS</p>
```

다음 CSS에서:

```css
h1 {
  color: red;
}
```

`h1`이 Selector입니다.

```text
HTML

<h1>Hello</h1>
<p>CSS</p>

       ▲
       │ 선택
       │

CSS

h1 {
  color: red;
}
```

따라서 `h1` Element에만 `color: red`가 적용됩니다.

Selector에는 다양한 종류가 있습니다.

```css
p {
}
```

Element Selector

```css
.title {
}
```

Class Selector

```css
#header {
}
```

ID Selector

Selector는 CSS에서 매우 중요한 주제이므로 **PART 2에서 자세히 다룹니다.**

---

# 6. Property와 Value

CSS Declaration은 일반적으로 다음 형태를 사용합니다.

```text
Property : Value ;
```

예를 들어:

```css
color: red;
```

여기서:

```text
color → Property
red   → Value
```

입니다.

다른 예를 보면:

```css
font-size: 20px;
```

```text
font-size → Property
20px      → Value
```

또한 하나의 Rule에는 여러 Declaration을 작성할 수 있습니다.

```css
.card {
  width: 300px;
  background-color: white;
  border: 1px solid gray;
}
```

각 Declaration은 `;`으로 구분합니다.

```text
Selector
   │
   ▼

.card {
  width: 300px;
  ─────────────
  Declaration

  background-color: white;
  ────────────────────────
  Declaration

  border: 1px solid gray;
  ───────────────────────
  Declaration
}
```

---

# 7. CSS를 HTML에 적용하는 세 가지 방법

CSS를 HTML에 적용하는 방법은 크게 세 가지입니다.

```text
CSS 적용 방법
     │
     ├── Inline CSS
     ├── Internal CSS
     └── External CSS
```

---

## 7.1 Inline CSS

HTML Element의 `style` Attribute에 직접 CSS를 작성합니다.

```html
<h1 style="color: blue;">Hello CSS</h1>
```

구조는:

```text
HTML Element
     │
     └── style Attribute
              │
              └── CSS Declaration
```

입니다.

간단한 테스트에는 사용할 수 있지만 일반적인 웹 애플리케이션의 스타일 관리 방법으로는 적합하지 않습니다.

---

# 8. Internal CSS

HTML 문서 내부의 `<style>` Element에 CSS를 작성합니다.

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    h1 {
      color: blue;
    }

    p {
      color: gray;
    }
  </style>
</head>
<body>

  <h1>Hello CSS</h1>
  <p>CSS를 공부합니다.</p>

</body>
</html>
```

HTML과 CSS가 하나의 파일에 존재합니다.

작은 예제나 간단한 실습에서는 편리하지만 프로젝트가 커지면 CSS 관리가 어려워집니다.

---

# 9. External CSS

CSS를 별도의 파일로 작성하는 방법입니다.

예를 들어:

```text
project/
│
├── index.html
└── style.css
```

`style.css`:

```css
h1 {
  color: blue;
}

p {
  color: gray;
}
```

HTML에서는 `<link>`를 사용하여 CSS 파일을 연결합니다.

```html
<head>
  <link rel="stylesheet" href="style.css">
</head>
```

관계를 보면:

```text
index.html
     │
     │ <link>
     ▼
style.css
     │
     │ CSS Rules
     ▼
HTML Elements에 스타일 적용
```

실제 웹 개발에서는 일반적으로 CSS를 별도의 파일이나 스타일 시스템으로 관리합니다.

---

# 10. `<link>` Element

External CSS에서 사용하는:

```html
<link rel="stylesheet" href="style.css">
```

를 살펴보겠습니다.

`href`는 가져올 CSS 파일의 위치를 지정합니다.

```html
href="style.css"
```

`rel`은 현재 HTML 문서와 연결된 Resource의 관계(Relationship)를 나타냅니다.

```html
rel="stylesheet"
```

즉,

```text
현재 HTML Document
        │
        │ relationship
        ▼
     stylesheet
        │
        ▼
    style.css
```

라는 의미입니다.

보통 `<head>` 내부에 작성합니다.

```html
<head>
  <meta charset="UTF-8">
  <title>CSS Example</title>

  <link rel="stylesheet" href="style.css">
</head>
```

---

# 11. CSS 주석

CSS 주석은 다음 문법을 사용합니다.

```css
/* CSS 주석 */
```

예:

```css
/* 제목 스타일 */
.title {
  color: blue;
}
```

여러 줄도 가능합니다.

```css
/*
  상품 카드
  공통 스타일
*/
.card {
  width: 300px;
}
```

JavaScript처럼 다음 문법을 사용하면 안 됩니다.

```css
// CSS 주석
```

CSS의 표준 주석 문법은:

```css
/* ... */
```

입니다.

---

# 12. 하나의 HTML Element에 여러 CSS가 적용될 수 있다

CSS를 배우면서 매우 중요한 사실이 있습니다.

하나의 Element에는 하나의 CSS Rule만 적용되는 것이 아닙니다.

예를 들어:

```html
<h1 class="title">Hello CSS</h1>
```

다음 CSS가 있다고 하겠습니다.

```css
h1 {
  font-size: 40px;
}

.title {
  color: blue;
}
```

`<h1>`에는 두 Rule의 스타일이 모두 적용될 수 있습니다.

```text
h1 Rule
font-size: 40px
      │
      ├──────────┐
      │          ▼
      │     <h1 class="title">
      │        Hello CSS
      │
      └──────────▲
                 │
.title Rule ─────┘
color: blue
```

결과적으로 해당 Element는:

```text
font-size: 40px
color: blue
```

를 모두 가지게 됩니다.

그런데 여러 Rule이 **같은 Property에 서로 다른 Value를 지정하면** 어떻게 될까요?

```css
h1 {
  color: red;
}

.title {
  color: blue;
}
```

이 문제를 해결하기 위해 CSS에는 **Cascade**라는 규칙이 존재합니다.

---

# 13. CSS의 C는 Cascading

CSS는:

> **Cascading Style Sheets**

의 약자입니다.

여기서 **Cascading**이 매우 중요합니다.

하나의 Element에 여러 스타일 규칙이 적용될 수 있기 때문에 브라우저는 어떤 스타일을 최종적으로 사용할 것인지 결정해야 합니다.

개념적으로:

```text
여러 CSS Rules
      │
      ▼
같은 Element를 대상으로 함
      │
      ▼
Cascade
      │
      ▼
충돌하는 Declaration 판단
      │
      ▼
최종 스타일 결정
```

예를 들어:

```css
p {
  color: red;
}

.text {
  color: blue;
}
```

```html
<p class="text">Hello</p>
```

두 Rule 모두 같은 `<p>`를 대상으로 할 수 있습니다.

브라우저는 Cascade 규칙에 따라 최종적으로 사용할 `color`를 결정합니다.

이 과정에는 **Origin, Importance, Specificity, Source Order** 등의 개념이 관계합니다.

이 내용은 Selector를 배운 후 자세히 이해하는 것이 좋으므로 **PART 2에서 Cascade와 Specificity를 함께 다룹니다.**

---

# 14. 브라우저는 CSS를 어떻게 사용하는가?

브라우저는 HTML 파일을 단순히 위에서 아래로 화면에 복사하는 것이 아닙니다.

HTML을 분석하여 **DOM(Document Object Model)**을 만들고 CSS를 분석하여 스타일 규칙을 처리합니다.

입문 단계에서는 다음 정도의 흐름으로 이해하면 충분합니다.

```text
HTML
 │
 │ Parsing
 ▼
DOM Tree
 │
 ├───────────────┐
 │               │
 │             CSS
 │               │
 │          Parsing / Style Rules
 │               │
 └───────┬───────┘
         ▼
   Style Calculation
         │
         ▼
      Layout
         │
         ▼
      Painting
         │
         ▼
       Screen
```

즉 CSS는 단순히 색상 정보만 제공하는 것이 아니라 **Element의 크기와 배치가 결정되는 과정에도 직접 관여**합니다.

---

# 15. CSS를 이해하는 핵심 관점

앞으로 CSS를 공부할 때 가장 중요한 관점이 있습니다.

HTML:

```html
<div class="card">
  <h2>Keyboard</h2>
  <p>50,000원</p>
</div>
```

브라우저는 이러한 Element들을 화면에 배치하기 위한 **Box**로 다룹니다.

개념적으로:

```text
HTML Element
      │
      ▼
     Box
      │
      ├── 크기
      ├── 간격
      ├── 테두리
      └── 위치
```

그리고 여러 Box가 존재하면:

```text
Box
Box
Box
Box
```

브라우저는 이 Box들을 **어떻게 배치할 것인가**를 결정해야 합니다.

여기에서 CSS Layout이 시작됩니다.

```text
HTML Element
      ↓
     Box
      ↓
  Box Model
      ↓
Normal Flow
      ↓
   display
      ↓
 ┌────┴─────┐
 ↓          ↓
Flexbox    Grid
```

따라서 앞으로 CSS를 단순히:

> "글자색이나 배경색을 변경하는 문법"

으로 생각하면 안 됩니다.

더 중요한 관점은:

> **CSS는 HTML Element가 화면에서 어떤 Box가 되고, 그 Box들이 어떻게 배치되고 표현되는지를 결정하는 언어이다.**

입니다.

---

# 16. PART 1 핵심 정리

CSS는 **Cascading Style Sheets**의 약자이며 HTML 문서의 표현과 레이아웃을 정의합니다.

```text
HTML
구조와 의미
   │
   ▼
CSS
표현과 레이아웃
   │
   ▼
Browser
화면 렌더링
```

CSS Rule의 기본 구조는:

```css
selector {
  property: value;
}
```

입니다.

CSS 적용 방법은:

```text
Inline CSS
Internal CSS
External CSS
```

가 있으며 실제 프로젝트에서는 주로 CSS를 HTML과 분리하여 관리합니다.

그리고 CSS를 제대로 이해하기 위한 가장 중요한 흐름은:

```text
HTML Element
      ↓
     Box
      ↓
Box의 크기와 간격
      ↓
Box의 배치
      ↓
    Layout
```

입니다.

다음 PART에서는 CSS가 **어떤 HTML Element에 적용될 것인지 결정하는 Selector**를 자세히 알아봅니다.

```text
PART 1
CSS란 무엇인가?
      │
      ▼
PART 2
어떤 Element에 적용할 것인가?
      │
      ▼
CSS Selector
```
