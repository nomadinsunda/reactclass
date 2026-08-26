# `<nav>` — HTML의 Navigation Landmark

## 사용자가 어디로 이동할 수 있는지를 의미적으로 표현하는 시맨틱 요소

`<nav>`는 메뉴를 예쁘게 만들기 위한 태그가 아닙니다.

CSS 스타일을 적용하지 않으면 `<nav>`와 `<div>`는 화면에서 거의 차이가 없어 보일 수도 있습니다.

하지만 HTML 문서의 의미 구조에서는 큰 차이가 있습니다.

```html
<div>
  ...
</div>
```

`<div>`는 특별한 의미가 없는 범용 컨테이너입니다.

반면:

```html
<nav>
  ...
</nav>
```

`<nav>`는 브라우저와 보조기술에게 다음과 같은 의미를 전달합니다.

> **"이 영역은 사용자가 문서나 사이트의 주요 위치로 이동하기 위한 Navigation 영역입니다."**

---

# 1. `<nav>`란?

`<nav>`는 현재 문서 내부 또는 다른 문서로 이동할 수 있는 **주요 navigation 링크들의 section**을 나타내는 HTML 시맨틱 요소입니다.

대표적으로 다음과 같은 곳에서 사용합니다.

```text
사이트 메인 메뉴
카테고리 메뉴
사이드바 Navigation
페이지 내부 목차
푸터의 주요 Navigation
```

가장 기본적인 형태는 다음과 같습니다.

```html
<nav>
  <ul>
    <li><a href="/">홈</a></li>
    <li><a href="/about">소개</a></li>
    <li><a href="/products">상품</a></li>
  </ul>
</nav>
```

구조적으로 보면:

```text
<nav>
  │
  └── <ul>
       │
       ├── <li>
       │    └── <a> 홈
       │
       ├── <li>
       │    └── <a> 소개
       │
       └── <li>
            └── <a> 상품
```

각 요소는 서로 다른 역할을 담당합니다.

| 요소      | 역할                     |
| ------- | ---------------------- |
| `<nav>` | Navigation 영역이라는 의미 부여 |
| `<ul>`  | Navigation 항목들의 목록     |
| `<li>`  | 개별 Navigation 항목       |
| `<a>`   | 실제 이동 가능한 링크           |

따라서:

> **`<nav>` = Navigation의 의미적 영역**

이라고 이해하면 됩니다.

---

# 2. `<nav>`는 링크 자체가 아니다

초보자가 자주 혼동하는 부분입니다.

`<nav>` 자체가 사용자를 다른 페이지로 이동시키는 것은 아닙니다.

실제 이동은 `<a>`가 담당합니다.

```html
<nav>
  <a href="/about">About</a>
</nav>
```

역할을 분리하면:

```text
<nav>
  │
  │ "이 영역은 Navigation이다"
  │
  ▼
<a href="/about">
  │
  │ "사용자를 /about으로 이동시킨다"
  ▼
Navigation
```

즉:

```text
<nav> = 영역의 의미

<a>   = 실제 링크
```

입니다.

이 구분은 React Router를 배울 때도 중요합니다.

---

# 3. 왜 `<div>` 대신 `<nav>`를 사용하는가?

다음 두 코드를 비교해 보겠습니다.

## `<div>` 사용

```html
<div class="menu">
  <a href="/">홈</a>
  <a href="/about">소개</a>
  <a href="/products">상품</a>
</div>
```

## `<nav>` 사용

```html
<nav>
  <a href="/">홈</a>
  <a href="/about">소개</a>
  <a href="/products">상품</a>
</nav>
```

화면은 CSS에 따라 동일하게 만들 수 있습니다.

하지만 문서 구조는 다릅니다.

```text
<div>
   │
   ▼
일반적인 컨테이너


<nav>
   │
   ▼
Navigation Section
   │
   ▼
Navigation Landmark
```

`<nav>`를 사용하면 HTML 자체에 **영역의 목적**이 포함됩니다.

---

# 4. Semantic HTML의 관점

HTML에는 여러 시맨틱 요소가 있습니다.

```text
<header>
<nav>
<main>
<article>
<section>
<aside>
<footer>
```

이 요소들은 단순히 화면을 나누기 위한 박스가 아닙니다.

각 영역이 **무엇을 의미하는지** 표현합니다.

예를 들어:

```html
<body>

  <header>
    ...
  </header>

  <nav>
    ...
  </nav>

  <main>
    ...
  </main>

  <aside>
    ...
  </aside>

  <footer>
    ...
  </footer>

</body>
```

문서 구조를 개념적으로 표현하면:

```text
Document
│
├── Header
│
├── Navigation
│
├── Main Content
│
├── Complementary Content
│
└── Footer
```

이렇게 HTML 코드 자체가 문서의 의미 구조를 설명하게 됩니다.

---

# 5. `<nav>`와 접근성

`<nav>`의 중요한 목적 중 하나는 접근성입니다.

`<nav>`는 일반적으로 접근성 트리에서 **navigation landmark**로 노출됩니다.

개념적으로:

```text
HTML

<nav>
  ...
</nav>

      │
      ▼

Accessibility Tree

Navigation Landmark
```

스크린리더 등의 보조기술은 이러한 landmark를 이용하여 페이지의 주요 영역을 파악할 수 있습니다.

예를 들어 페이지가:

```text
Banner
Navigation
Main
Complementary
Contentinfo
```

와 같은 landmark 구조를 가지고 있다면 사용자는 페이지의 모든 내용을 처음부터 순서대로 탐색하지 않고 주요 영역을 기준으로 탐색할 수 있습니다.

따라서 `<nav>`의 중요한 장점은:

> **Navigation 영역을 시각적으로만 표현하는 것이 아니라 접근성 구조에도 Navigation 영역으로 전달한다는 것**

입니다.

---

# 6. `role="navigation"`과 `<nav>`

`<nav>`에는 navigation landmark 의미가 이미 존재합니다.

따라서 일반적으로:

```html
<nav role="navigation">
  ...
</nav>
```

처럼 `role="navigation"`을 반복해서 작성할 필요가 없습니다.

그냥:

```html
<nav>
  ...
</nav>
```

이면 충분합니다.

반면 다음처럼 `<div>`를 사용한다면:

```html
<div role="navigation">
  ...
</div>
```

ARIA를 통해 Navigation 역할을 부여할 수 있습니다.

하지만 HTML에 이미 의미가 있는 요소가 존재한다면:

```html
<nav>
```

를 사용하는 편이 더 자연스럽습니다.

따라서 일반적인 원칙은:

```text
가능하면 Native HTML Semantic Element 사용

              ↓

          <nav>

              ↓

필요할 때 ARIA로 추가 정보 제공
```

입니다.

---

# 7. 모든 링크를 `<nav>`로 감싸는 것은 아니다

`<nav>`는 페이지의 모든 링크를 위한 컨테이너가 아닙니다.

예를 들어:

```html
<p>
  자세한 내용은
  <a href="/docs">공식 문서</a>를 참고하세요.
</p>
```

여기 있는 링크를:

```html
<nav>
  <a href="/docs">공식 문서</a>
</nav>
```

로 만들 필요는 없습니다.

왜냐하면 이 링크는 문장 속의 참고 링크이지 페이지의 주요 Navigation section이 아니기 때문입니다.

따라서:

```text
주요 Navigation 링크 모음
          │
          ▼
        <nav>


본문 속 일반 링크
          │
          ▼
         <a>
```

라고 구분하면 됩니다.

---

# 8. `<nav>`를 사용하는 대표적인 위치

## 사이트 메인 Navigation

```html
<header>
  <h1>My Blog</h1>

  <nav aria-label="주 메뉴">
    <ul>
      <li><a href="/">홈</a></li>
      <li><a href="/posts">글 목록</a></li>
      <li><a href="/tags">태그</a></li>
      <li><a href="/about">소개</a></li>
    </ul>
  </nav>
</header>
```

구조:

```text
Header
│
├── Logo / Title
│
└── Navigation
     │
     ├── Home
     ├── Posts
     ├── Tags
     └── About
```

---

# 9. 페이지 내부 Navigation

`<nav>`는 다른 페이지로 이동할 때만 사용하는 것이 아닙니다.

현재 페이지의 중요한 section으로 이동하는 목차에도 사용할 수 있습니다.

```html
<nav aria-label="문서 목차">
  <ol>
    <li><a href="#intro">소개</a></li>
    <li><a href="#setup">설치</a></li>
    <li><a href="#usage">사용법</a></li>
  </ol>
</nav>
```

예를 들어:

```text
현재 Document

#intro
#setup
#usage
```

에 대해:

```text
<nav>
 │
 ├── href="#intro"
 ├── href="#setup"
 └── href="#usage"
```

와 같은 Navigation을 만들 수 있습니다.

즉 `<nav>`의 Navigation은:

```text
다른 Document로 이동

또는

현재 Document의 주요 section으로 이동
```

모두 포함할 수 있습니다.

---

# 10. `<nav>`는 여러 개 사용할 수 있다

한 페이지에는 여러 종류의 Navigation이 존재할 수 있습니다.

예를 들어:

```html
<header>
  <nav aria-label="주 메뉴">
    ...
  </nav>
</header>

<aside>
  <nav aria-label="카테고리">
    ...
  </nav>
</aside>

<footer>
  <nav aria-label="푸터 메뉴">
    ...
  </nav>
</footer>
```

문서 구조는:

```text
Document
│
├── Header
│    └── Navigation
│         "주 메뉴"
│
├── Main
│
├── Aside
│    └── Navigation
│         "카테고리"
│
└── Footer
     └── Navigation
          "푸터 메뉴"
```

처럼 구성될 수 있습니다.

HTML에서 `<nav>`를 한 번만 사용해야 한다는 규칙은 없습니다.

---

# 11. 여러 `<nav>`와 접근 가능한 이름

페이지에 Navigation landmark가 하나뿐이라면:

```html
<nav>
  ...
</nav>
```

만으로도 충분한 경우가 많습니다.

하지만 여러 Navigation landmark가 존재한다면 사용자가 각각을 구분할 수 있도록 이름을 제공하는 것이 중요합니다.

```html
<nav aria-label="주 메뉴">
  ...
</nav>

<nav aria-label="카테고리">
  ...
</nav>

<nav aria-label="푸터 메뉴">
  ...
</nav>
```

보조기술에서는 개념적으로:

```text
Navigation — 주 메뉴

Navigation — 카테고리

Navigation — 푸터 메뉴
```

처럼 구분할 수 있습니다.

따라서 `aria-label`은:

> **여러 Navigation landmark를 서로 구별할 수 있는 접근 가능한 이름을 제공하는 방법**

으로 이해하면 좋습니다.

---

# 12. `<nav>`와 `<header>`

두 요소는 역할이 다릅니다.

### `<header>`

문서나 section의 소개 영역을 나타냅니다.

예:

```text
Logo
Title
Heading
Search
Intro
Navigation
```

등을 포함할 수 있습니다.

### `<nav>`

Navigation 영역 자체를 의미합니다.

따라서 다음 구조가 자연스럽습니다.

```html
<header>

  <h1>My Site</h1>

  <nav>
    ...
  </nav>

</header>
```

구조:

```text
<header>
   │
   ├── Site Title
   │
   └── <nav>
         └── Navigation Links
```

하지만 `<nav>`가 반드시 `<header>` 안에 있어야 하는 것은 아닙니다.

```html
<header>
  ...
</header>

<nav>
  ...
</nav>

<main>
  ...
</main>
```

도 올바른 구조입니다.

---

# 13. `<nav>`와 `<footer>`

`<footer>` 안에도 Navigation이 존재할 수 있습니다.

```html
<footer>

  <nav aria-label="푸터 메뉴">
    <ul>
      <li><a href="/terms">이용약관</a></li>
      <li><a href="/privacy">개인정보 처리방침</a></li>
      <li><a href="/contact">문의하기</a></li>
    </ul>
  </nav>

  <p>© 2026 My Company</p>

</footer>
```

구조:

```text
Footer
│
├── Navigation
│    ├── Terms
│    ├── Privacy
│    └── Contact
│
└── Copyright
```

여기에서도:

```text
<footer> = Footer 영역

<nav> = 그 안의 Navigation 영역
```

이라는 역할 분리가 존재합니다.

---

# 14. `<nav>`와 `<aside>`

`<aside>`는 본문의 주요 흐름과 간접적으로 관련된 보조 콘텐츠를 나타냅니다.

예를 들어 Sidebar가 있다고 하겠습니다.

```html
<aside>

  <h2>Categories</h2>

  <nav aria-label="카테고리">
    ...
  </nav>

</aside>
```

역할은:

```text
<aside>
   │
   │ 보조 콘텐츠 영역
   │
   └── <nav>
         │
         └── Navigation
```

입니다.

`aside`와 `nav`는 서로 경쟁하는 태그가 아니라 **서로 다른 의미를 표현하기 때문에 중첩될 수 있습니다.**

---

# 15. `<nav>`와 `<ul>`

이 둘 역시 역할이 다릅니다.

```html
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
  </ul>
</nav>
```

역할:

```text
<nav>
Navigation 영역
     │
     ▼
<ul>
항목들의 List
     │
     ▼
<li>
개별 List Item
     │
     ▼
<a>
Navigation Link
```

`<nav>`가 `<ul>`을 반드시 요구하는 것은 아닙니다.

예를 들어:

```html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>
```

도 가능합니다.

다만 여러 Navigation 항목이 논리적인 목록을 형성한다면 list markup을 사용하는 것이 구조를 더 명확하게 표현할 수 있습니다.

---

# 16. Skip Link와 Navigation

키보드 사용자는 페이지 상단에 많은 Navigation 링크가 있으면 본문에 도달하기 위해 여러 번 Tab을 눌러야 할 수 있습니다.

```text
Logo
 ↓
Menu 1
 ↓
Menu 2
 ↓
Menu 3
 ↓
Menu 4
 ↓
...
 ↓
Main Content
```

이를 개선하는 대표적인 접근성 패턴이 Skip Link입니다.

```html
<a href="#main" class="skip-link">
  본문 바로가기
</a>

<header>
  <nav aria-label="주 메뉴">
    ...
  </nav>
</header>

<main id="main">
  ...
</main>
```

동작:

```text
Tab
 │
 ▼
본문 바로가기
 │
 │ Enter
 ▼
#main
 │
 ▼
Main Content
```

사용자가 반복적인 Navigation을 건너뛰고 주요 콘텐츠로 바로 이동할 수 있도록 도와줍니다.

---

# 17. Skip Link CSS

일반적으로 Skip Link는 평소에는 시각적으로 숨겨 두었다가 키보드 focus를 받았을 때 나타나도록 구현합니다.

```css
.skip-link {
  position: absolute;
  left: -9999px;
}

.skip-link:focus {
  left: 0;
  top: 0;
  padding: 1rem;
  background: black;
  color: white;
  z-index: 1000;
}
```

핵심은:

```text
평상시
   │
   ▼
시각적으로 숨김

Tab으로 Focus
   │
   ▼
화면에 표시

Enter
   │
   ▼
Main Content로 이동
```

입니다.

---

# 18. 반응형 Navigation과 `<nav>`

`<nav>` 자체는 메뉴를 열거나 닫는 기능을 제공하지 않습니다.

다음과 같은 모바일 메뉴가 있다고 하겠습니다.

```text
[ My Site ]       [ ☰ ]
```

햄버거 버튼을 누르면:

```text
Home
Products
Cart
My Page
```

가 나타납니다.

여기서 역할을 분리해야 합니다.

```text
<nav>
    │
    └── Navigation의 의미


<button>
    │
    └── 메뉴 열기 / 닫기


CSS
    │
    └── 표시 방식


JavaScript / React
    │
    └── 상태 및 동작
```

즉 `<nav>`는 Navigation의 **의미**를 담당하지 메뉴 UI의 동작 자체를 담당하지 않습니다.

---

# 19. 햄버거 메뉴의 접근성

예를 들어:

```html
<button
  aria-expanded="false"
  aria-controls="main-menu"
>
  메뉴
</button>

<nav
  id="main-menu"
  aria-label="주 메뉴"
>
  ...
</nav>
```

여기서:

```text
aria-controls="main-menu"
          │
          ▼
어떤 영역을 제어하는가?


aria-expanded="false"
          │
          ▼
현재 메뉴가 펼쳐져 있는가?
```

를 표현합니다.

메뉴가 열리면 JavaScript 또는 React에서:

```html
aria-expanded="true"
```

로 변경할 수 있습니다.

---

# 20. React에서 `<nav>`

React에서도 `<nav>`의 의미는 HTML과 동일합니다.

```jsx
function Navbar() {
  return (
    <nav aria-label="메인 내비게이션">
      <ul>
        <li>
          <a href="/">홈</a>
        </li>

        <li>
          <a href="/about">소개</a>
        </li>
      </ul>
    </nav>
  );
}
```

JSX로 작성되었지만 브라우저에서는 실제 HTML `<nav>` element가 만들어집니다.

```text
React JSX

<nav>
  ...
</nav>

     │
     ▼

React Rendering

     │
     ▼

DOM

HTMLNavElement
```

따라서 React를 사용한다고 해서 HTML의 시맨틱 의미가 사라지는 것은 아닙니다.

---

# 21. React Router의 `<Link>`와 `<nav>`

React Router를 사용하면 Navigation 구조와 Navigation 동작을 명확하게 분리할 수 있습니다.

```jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav aria-label="메인 내비게이션">
      <ul>

        <li>
          <Link to="/">
            홈
          </Link>
        </li>

        <li>
          <Link to="/about">
            소개
          </Link>
        </li>

        <li>
          <Link to="/posts">
            게시글
          </Link>
        </li>

      </ul>
    </nav>
  );
}
```

여기에서 역할은 완전히 다릅니다.

```text
<nav>
   │
   │ Navigation 영역이라는
   │ 시맨틱 의미 제공
   ▼

<Link>
   │
   │ React Router
   │ Navigation 실행
   ▼

URL 변경
   │
   ▼
Route Matching
   │
   ▼
React UI 변경
```

즉:

> **`<nav>`는 "여기가 Navigation 영역이다"를 표현하고, `<Link>`는 "어디로 이동할 것인가"와 실제 Router Navigation을 담당합니다.**

---

# 22. `<nav>`와 React Router의 역할을 혼동하지 말자

다음 코드를 보겠습니다.

```jsx
<nav>
  <Link to="/">Home</Link>
  <Link to="/about">About</Link>
</nav>
```

각 요소의 책임은 다음과 같습니다.

```text
<nav>
   │
   └── Semantic Navigation Region


<Link to="/">
   │
   └── Router Navigation


<Link to="/about">
   │
   └── Router Navigation
```

`<nav>`를 제거하더라도:

```jsx
<div>
  <Link to="/">Home</Link>
  <Link to="/about">About</Link>
</div>
```

`Link`의 Router Navigation은 여전히 동작합니다.

하지만 Navigation landmark라는 **HTML 의미가 사라집니다.**

반대로:

```jsx
<nav>
  <a href="/">Home</a>
</nav>
```

React Router가 없어도 `<nav>`의 시맨틱 의미는 그대로 존재합니다.

따라서 두 개념은 서로 독립적입니다.

---

# 23. 흔한 실수 1 — 모든 링크를 `<nav>`에 넣기

잘못된 사고방식:

```text
링크가 있다
    ↓
무조건 nav
```

올바른 사고방식:

```text
링크가 있다
    │
    ▼
이 링크 그룹이
주요 Navigation을 구성하는가?
    │
 ┌──┴──┐
YES    NO
 │      │
 ▼      ▼
nav     일반 링크
```

`<nav>`는 링크가 있다는 이유만으로 사용하는 태그가 아닙니다.

---

# 24. 흔한 실수 2 — Navigation인데 `<div>`만 사용

```html
<div class="nav">
  <a href="/">Home</a>
  <a href="/about">About</a>
</div>
```

CSS 관점에서는 아무 문제가 없을 수 있습니다.

하지만 의미 구조는:

```text
<div>
  ↓
Generic Container
```

입니다.

Navigation이 명확하다면:

```html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>
```

처럼 표현하는 것이 의미를 더 잘 전달합니다.

---

# 25. 흔한 실수 3 — 여러 Navigation을 구분하지 않음

다음과 같이:

```html
<nav>...</nav>

<nav>...</nav>

<nav>...</nav>
```

여러 Navigation landmark가 존재한다면 사용자가 각각의 목적을 구분하기 어려울 수 있습니다.

필요하다면:

```html
<nav aria-label="주 메뉴">
  ...
</nav>

<nav aria-label="카테고리">
  ...
</nav>

<nav aria-label="푸터 메뉴">
  ...
</nav>
```

처럼 접근 가능한 이름을 제공하는 것이 좋습니다.

---

# 26. `<nav>`의 전체 구조

이제 `<nav>`의 역할을 하나의 흐름으로 정리해보겠습니다.

```text
HTML Document
      │
      ▼
    <nav>
      │
      │ Semantic Meaning
      ▼
Navigation Section
      │
      ├───────────────┐
      │               │
      ▼               ▼
Browser / DOM    Accessibility Tree
                      │
                      ▼
               Navigation Landmark
      │
      ▼
Navigation Links
      │
      ▼
<a> 또는 <Link>
      │
      ▼
사용자 Navigation
```

즉 `<nav>`의 핵심 역할은 **이동 자체를 수행하는 것보다 Navigation 영역의 의미를 정의하는 것**입니다.

---

# 27. React Router까지 연결한 전체 구조

앞에서 배운 React Router와 연결하면 더욱 명확합니다.

```text
<nav>
 │
 │ "여기는 Navigation 영역"
 │
 ▼
<Link to="/about">
 │
 │ 사용자 클릭
 ▼
React Router Navigation
 │
 ▼
Router Location 변경
 │
 ▼
<Routes>
 │
 │ Route Matching
 ▼
<Route path="/about">
 │
 ▼
<About />
 │
 ▼
React Rendering
 │
 ▼
UI
```

여기서:

```text
HTML Semantic Layer
        │
        ▼
      <nav>


Navigation Layer
        │
        ▼
      <Link>


Routing Layer
        │
        ▼
      <Routes>
      <Route>


Rendering Layer
        │
        ▼
     React UI
```

로 책임이 명확하게 분리됩니다.

---

# 최종 정의

`<nav>`를 가장 정확하게 한 문장으로 정리하면 다음과 같습니다.

> **`<nav>`는 현재 문서 또는 다른 문서의 주요 위치로 이동하기 위한 Navigation 링크들의 section을 의미적으로 표현하고, 보조기술에 Navigation landmark를 제공하는 HTML 시맨틱 요소입니다.**

조금 더 쉽게 표현하면:

> **`<nav>` = "이 영역은 사용자가 어디로 이동할지 선택하는 Navigation 영역이다"라는 의미를 HTML 문서에 부여하는 요소**

React Router와 함께 사용할 때는 다음 구분이 특히 중요합니다.

```text
<nav>
"여기는 Navigation 영역이다."
        │
        ▼
<Link>
"이곳으로 이동한다."
        │
        ▼
<Routes>
"현재 URL과 맞는 Route를 찾는다."
        │
        ▼
<Route>
"이 URL에는 이 UI를 사용한다."
        │
        ▼
React UI
```

따라서 `<nav>`는 React Router의 기능이 아니라 **HTML의 시맨틱 구조를 담당하는 요소**이며, `<Link>`, `<Routes>`, `<Route>`와 결합했을 때 의미 구조와 Navigation 동작이 명확하게 분리된 React 애플리케이션을 만들 수 있습니다.


