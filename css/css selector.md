

## 0. CSS 셀렉터란? 🎯

```css
p.highlight {
  color: red;
}
```

* `p.highlight` 👉 **셀렉터(selector)**
* `{ color: red; }` 👉 **선언 블록(declaration block)**

브라우저는 다음 순서로 동작합니다:

1. HTML 파싱 → DOM 트리 만들기
2. CSS 파싱 → 셀렉터/규칙 테이블 만들기
3. 각 DOM 요소에 대해 “이 셀렉터에 매칭되나?”를 검사
4. 매칭된 규칙의 스타일들을 모아서 **계산된 스타일(computed style)** 생성

결국 **“어떤 요소에 어떤 규칙을 적용할지”**를 결정하는 핵심이 **셀렉터**입니다.

---

## 1. 기본 셀렉터 4형제 🔤

### 1-1. 타입 셀렉터 (Type Selector)

**태그 이름으로 고르는 방식**

```css
p {
  line-height: 1.6;
}

h1 {
  font-size: 2rem;
}
```

* 모든 `<p>` 요소, `<h1>` 요소에 적용
* HTML 구조가 바뀌면 영향 범위가 커질 수 있으므로, **전역 스타일용**으로 사용

> 실무 팁 💡
> reset, base 스타일에서는 `body`, `h1`, `p`, `a` 같은 **타입 셀렉터**를 자주 사용합니다.

---

### 1-2. 클래스 셀렉터 (Class Selector)

```css
.highlight {
  background: yellow;
}

.btn-primary {
  padding: 0.5rem 1rem;
  border-radius: 4px;
}
```

* HTML:

```html
<p class="highlight">중요한 텍스트</p>
<button class="btn-primary">저장</button>
```

* 클래스는 **재사용 가능** / **여러 개 동시 사용 가능**

```html
<button class="btn btn-primary btn-lg">버튼</button>
```

> 실무에서는
>
> * **역할/의미 기반** 클래스: `.error-message`, `.card-header`
> * **유틸리티/도우미** 클래스: `.mt-4`, `.text-center` (Tailwind 유사)
>   두 가지를 섞어서 많이 씁니다.

---

### 1-3. 아이디 셀렉터 (ID Selector)

```css
#main-header {
  background: #222;
  color: white;
}
```

* HTML:

```html
<header id="main-header">...</header>
```

* 한 문서에서 **ID는 원칙적으로 1번만** 사용해야 합니다 (고유식별자)
* **특이점**: **특이성(specificity)**이 매우 높아서, 나중에 스타일 덮어쓰기 어려워짐

> 실무 팁 ⚠️
> “스타일링 용도로 ID 쓰지 마라”는 말 많이 들어보셨을 겁니다.
>
> * JS에서 특정 요소를 찾을 때: `document.getElementById(...)`
> * CSS에서는 **가능하면 class 위주**로 설계하는 게 유지보수에 유리합니다.

---

### 1-4. 전체 셀렉터 (Universal Selector)

```css
* {
  box-sizing: border-box;
}
```

* 모든 요소에 적용
* 보통 **reset / global 규칙**에 사용
* `*` 한 번 쓰면 진짜 모든 요소를 훑으니, **과도한 사용은 성능에 영향** 줄 수 있습니다.

---

## 2. 속성 셀렉터 (Attribute Selector) 🧩

**태그의 속성(attribute) 값으로 선택하는 고급 필터**

```css
/* type이 "email"인 input */
input[type="email"] {
  border-color: green;
}

/* data-role="dialog" 인 요소 */
div[data-role="dialog"] {
  position: fixed;
}
```

### 2-1. 정확히 같은 값

```css
input[type="text"] {
  /* type이 정확히 text인 input */
}
```

### 2-2. 값이 "어디에 포함되어 있는가" 계열

```css
/* 값이 공백으로 구분된 목록 중 "tag"가 포함된 경우 */
[class~="tag"] {
  /* class="tag red" / class="blue tag" 등 매칭 */
}

/* 값이 "btn-"으로 시작 */
[class^="btn-"] {
  /* btn-primary, btn-danger 등 */
}

/* 값이 "-icon"으로 끝나는 경우 */
[class$="-icon"] {
  /* user-icon, search-icon 등 */
}

/* 값에 "dialog" 문자열이 들어간 경우 */
[data-role*="dialog"] {
  /* modal-dialog, dialog-main 등 */
}
```

> 실무 예시 💼
> data-* 속성과 조합하면 JS와 CSS가 깔끔하게 연동됩니다.

```html
<div data-state="error">...</div>
<div data-state="success">...</div>
```

```css
[data-state="error"] { color: red; }
[data-state="success"] { color: green; }
```

---

## 3. 관계 셀렉터(Combinators) 🧬

DOM 트리 기준으로 **요소들 사이 관계**를 표현합니다.

### 3-1. 후손 셀렉터 (Descendant) – 공백

```css
/* .card 안에 있는 모든 p */
.card p {
  margin-bottom: 0.5rem;
}
```

* `.card` **안쪽 어느 깊이**에 있든 `<p>`면 매칭

---

### 3-2. 자식 셀렉터 (Child) – `>`

```css
.nav > li {
  list-style: none;
}
```

* `.nav`의 **직계 자식**인 `<li>`만 매칭

DOM 예시:

```html
<ul class="nav">
  <li>직계 1</li>          <!-- 매칭 O -->
  <li>
    <span>직계 2 안의 span</span> <!-- span은 매칭 X -->
  </li>
</ul>
```

---

### 3-3. 인접 형제 셀렉터 (Adjacent Sibling) – `+`

```css
h2 + p {
  margin-top: 0;
}
```

* `<h2>` 바로 **바로 뒤에 오는** `<p>`만 매칭

```html
<h2>제목</h2>
<p>여기는 붙어 있는 문단</p>  <!-- 매칭 O -->
<p>여기는 두 번째 문단</p>    <!-- 매칭 X -->
```

---

### 3-4. 일반 형제 셀렉터 (General Sibling) – `~`

```css
h2 ~ p {
  color: gray;
}
```

* 같은 부모를 가진 형제들 중에서
  **h2 뒤에 나오는 모든 p** 매칭

```html
<h2>제목</h2>
<p>p1</p>   <!-- 매칭 O -->
<div>div</div>
<p>p2</p>   <!-- 매칭 O -->
```

---

## 4. 가상 클래스 (Pseudo-classes) 😎

**요소의 “상태”**를 기반으로 선택합니다.

형식: `:something`

### 4-1. 대표적인 상호작용 상태

```css
a:hover {
  text-decoration: underline;
}

button:active {
  transform: scale(0.98);
}

input:focus {
  outline: 2px solid dodgerblue;
}
```

* `:hover` : 마우스가 올라간 상태
* `:active`: 클릭하는 순간
* `:focus`: 포커스를 가진 상태 (키보드 탭, 클릭 등)

---

### 4-2. 구조 기반 상태 셀렉터

#### `:first-child`, `:last-child`

```css
.list-item:first-child {
  border-top: none;
}

.list-item:last-child {
  border-bottom: none;
}
```

#### `:nth-child()`

```css
/* 짝수 번째 줄만 회색 */
tr:nth-child(even) {
  background: #f7f7f7;
}

/* 3n + 1 번째(1,4,7,...) */
li:nth-child(3n + 1) {
  font-weight: bold;
}
```

#### `:nth-of-type()`

```css
/* 같은 타입(h2) 안에서만 n번째 계산 */
h2:nth-of-type(2) {
  color: red;
}
```

> `:nth-child` vs `:nth-of-type` 차이 ⚔️
>
> * `:nth-child(2)` → “부모의 모든 자식 중 두 번째”
> * `:nth-of-type(2)` → “부모의 h2들 중 두 번째”

---

### 4-3. 상태 필터링 셀렉터

```css
/* 체크된 체크박스만 */
input[type="checkbox"]:checked {
  accent-color: green;
}

/* 비활성화된 input */
input:disabled {
  background: #eee;
}

/* 사용 가능한 input */
input:enabled {
  /* ... */
}
```

---

### 4-4. 논리 부정 :not()

```css
/* .btn 이면서 .btn-outline 이 아닌 버튼들 */
.btn:not(.btn-outline) {
  border: none;
}
```

* 특정 조건을 **제외**할 때 매우 유용

---

## 5. 가상 요소 (Pseudo-elements) ✂️

요소의 **일부분** 또는 **가상의 자식**을 선택하는 셀렉터.

형식: `::something` (옛날 문법은 `:` 하나)

### 5-1. `::before`, `::after`

```css
.button::before {
  content: "🔥";
  margin-right: 0.25rem;
}

.button::after {
  content: "→";
  margin-left: 0.25rem;
}
```

* 실제 DOM에 노드는 없지만,
  **“CSS로만 추가된 가짜 요소”**를 만들어 줍니다.
* `content:`가 **필수** (적어도 빈 문자열이라도)

---

### 5-2. `::first-line`, `::first-letter`

```css
p::first-line {
  font-weight: bold;
}

p::first-letter {
  font-size: 2rem;
}
```

* 신문/잡지 레이아웃처럼 **첫 글자**, **첫 줄**만 스타일링할 때 사용

---

### 5-3. `::selection`

```css
::selection {
  background: yellow;
  color: black;
}
```

* 사용자가 드래그로 선택한 텍스트의 스타일 지정

---

## 6. 셀렉터 그룹과 리스트 📚

여러 셀렉터에 **같은 스타일**을 한 번에 적용:

```css
h1, h2, h3 {
  font-family: "Pretendard", system-ui, sans-serif;
}
```

* `,` 로 구분
* 각 셀렉터는 **독립적으로 매칭**
  (하나라도 매칭하면 규칙 적용)

---

## 7. 특이성(Specificity)과 우선순위 ⚖️

**“충돌하는 스타일 중에 누가 이길까?”**를 결정하는 규칙입니다.

### 7-1. 특이성 계산 개념

일반적인 특이성 우선순위 (단순화 버전):

1. `!important` (가장 강력하지만 가능하면 피하기)
2. 인라인 스타일 (`style="..."`)
3. ID 셀렉터 (`#header`)
4. 클래스 / 속성 / 가상클래스 (`.btn`, `[type="text"]`, `:hover`)
5. 타입 / 가상요소 (`div`, `p`, `::before`)
6. 전체 셀렉터 (`*`)

예시:

```css
p { color: black; }           /* 타입 */
.content p { color: blue; }   /* 클래스 + 타입 */
#main p { color: red; }       /* ID + 타입 */
```

```html
<p>1</p>                     <!-- black -->
<div class="content">
  <p>2</p>                   <!-- blue -->
</div>
<div id="main">
  <p>3</p>                   <!-- red -->
</div>
```

> 실무 팁 💡
>
> * ID 셀렉터를 남발하면 **나중에 덮어쓰기가 지옥**
> * 가능하면 **class 기반 BEM, 유틸리티 클래스, Tailwind** 등으로 설계
> * `!important`는 "마지막 수단"으로만 사용

---

## 8. 현대 CSS 고급 셀렉터들 🚀

브라우저들이 점점 최신 기능을 잘 지원하면서, 꽤 강력한 셀렉터들이 등장했습니다.

### 8-1. :is()

여러 셀렉터를 **하나로 그룹화**하고, **특이성 계산도 깔끔하게** 해주는 도구.

```css
/* 전통적인 방식 */
.card h1,
.card h2,
.card h3 {
  margin-bottom: 0.5rem;
}

/* :is() 사용 */
.card :is(h1, h2, h3) {
  margin-bottom: 0.5rem;
}
```

* 가독성이 훨씬 좋아지고, 중복을 줄여줍니다.

---

### 8-2. :where()

`:is()`와 비슷하지만, **무조건 특이성이 0**인 특징이 있습니다.

```css
/* :where()는 특이성이 0이라
   나중에 더 구체적인 규칙이 쉽게 덮어쓸 수 있음 */
:where(.card p) {
  margin: 0;
}
```

* “기본값, 리셋 용도”로 아주 유용
* Tailwind도 내부적으로 :where를 많이 활용합니다.

---

### 8-3. :has() (부모를 조건으로 고르기) 💥

지원하는 브라우저에서 이제 **“부모 선택자”에 가까운 효과**를 낼 수 있습니다.

```css
/* 자식에 .error-message가 있는 .form-group에만 테두리 표시 */
.form-group:has(.error-message) {
  border-color: red;
}
```

* `:has()` 안에는 **"자식 / 후손 / 형제 조건"**을 넣을 수 있습니다.
* CSS 레벨에서 조건부 스타일링을 더 많이 할 수 있게 해줍니다.

> 주의 ⚠️
>
> * 오래된 브라우저에서는 지원이 안 될 수 있으므로, 필요시 폴백 고려

---

## 9. 실무 패턴: BEM, 유틸리티, 컴포넌트 구조 🧱

### 9-1. BEM 스타일 셀렉터

```css
.card {}
.card__header {}
.card__body {}
.card--primary {}
```

```html
<article class="card card--primary">
  <header class="card__header">...</header>
  <div class="card__body">...</div>
</article>
```

* **Block__Element--Modifier** 패턴
* 셀렉터만 봐도 구조와 역할이 바로 보입니다.

---

### 9-2. 유틸리티 클래스 패턴 (Tailwind 류)

```html
<div class="mt-4 text-center text-gray-600">
  유틸리티 클래스 기반 레이아웃
</div>
```

* 개별 속성을 각각의 클래스로 나누고,
  HTML에서 조합해서 사용
* 셀렉터 설계가 단순해지고, **“깊은 계층 구조 + 복잡한 관계 셀렉터”**가 줄어듭니다.

---

### 9-3. React/컴포넌트 기반과 셀렉터

리액트 같은 컴포넌트 기반 개발에서는:

* **컴포넌트 루트에 className 주고**
* 내부는 가능한 한 **local한 클래스**만 사용

```jsx
function Card({ title, children }) {
  return (
    <article className="card">
      <h2 className="card__title">{title}</h2>
      <div className="card__body">{children}</div>
    </article>
  )
}
```

```css
.card { /* ... */ }
.card__title { /* ... */ }
.card__body { /* ... */ }
```

이렇게 하면:

* 셀렉터가 단순해지고 (`.card__title` 정도),
* 깊은 조합(`.layout .sidebar .menu li a`) 같은 유지보수 어려운 패턴을 줄일 수 있습니다.

---

## 10. 작은 연습 문제들 ✍️

아래 HTML이 있을 때, 각 요구사항을 만족하는 셀렉터를 떠올려 보시면 좋습니다.

```html
<ul class="menu">
  <li class="menu-item"><a href="/" class="link">Home</a></li>
  <li class="menu-item menu-item--active"><a href="/about" class="link">About</a></li>
  <li class="menu-item"><a href="/contact" class="link">Contact</a></li>
</ul>
```

1. **활성 메뉴 아이템**만 빨간색 배경을 주고 싶다
2. **마지막 메뉴 아이템**만 오른쪽 마진을 없애고 싶다
3. `.menu` 안에 있는 **모든 a 태그**에 `text-decoration: none`을 주고 싶다
4. `href`가 `/`로 시작하는 링크만 색을 바꾸고 싶다

생각해보시고, 혹시 정답/해설이 필요하시면 말씀해 주세요. 🙂

---

## 마무리 🧩

정리해 보면, CSS 셀렉터는 대략 이렇게 분류할 수 있습니다:

* **기본 셀렉터**: `*`, `div`, `.class`, `#id`
* **속성 셀렉터**: `[type="text"]`, `[data-role^="dialog"]` …
* **관계 셀렉터**: `A B`, `A > B`, `A + B`, `A ~ B`
* **가상 클래스**: `:hover`, `:focus`, `:nth-child()`, `:not()` …
* **가상 요소**: `::before`, `::after`, `::first-line` …
* **현대 셀렉터**: `:is()`, `:where()`, `:has()` …

이제부터 CSS 코드를 보실 때
“**이 스타일은 어떤 셀렉터로, 왜 이 요소에 적용되는가?**”를 의식적으로 따라가 보시면
**React + Tailwind + 순수 CSS**를 모두 다룰 때 훨씬 감이 좋아집니다.

