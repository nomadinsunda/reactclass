CSS Grid Layout은 **“2D 레이아웃 전용 엔진”**이라고 생각하시면 편합니다.
Flexbox가 한 축(가로 *또는* 세로)을 중심으로 아이템을 정렬한다면, Grid는 **가로 + 세로 두 축을 한 번에 설계**하는 시스템이에요.


---

## 1. Grid를 왜 쓰는가? – Flexbox랑 뭐가 달라? 🤔

### Flexbox

* “메인 축 하나 정해서”
  → `flex-direction: row`면 가로로 1줄 배치, 넘치면 줄바꿈(`flex-wrap`)
* 카드 리스트, 버튼 그룹, 네비게이션 바 같이 **일렬로 나열**하는 데 최적화

### Grid

* **행(row) + 열(column)을 동시에 설계**
* “3열 레이아웃인데, 첫 행은 200px, 나머지는 자동으로 늘어나고…” 같은 **표 형태 레이아웃**에 최적화
* **수평/수직을 동시에 고려한 레이아웃** → 웹페이지 전체 틀, 대시보드, 카드 그리드, 갤러리처럼 2D 구조에 강함

> 한 줄 요약
>
> * Flexbox: 줄 세우는 전문가 (1D)
> * Grid: 표(레이아웃) 짜는 전문가 (2D)

---

## 2. 기본 개념 정리 🧱

### 2.1 Grid Container와 Grid Item

```css
.container {
  display: grid; /* 또는 inline-grid */
}
```

* `display: grid`를 선언한 요소 → **Grid Container**
* 그 안에 직접 들어 있는 자식 요소들 → **Grid Item**

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

여기서 `.container`가 **컨테이너**, `.item`들이 **아이템**입니다.

---

### 2.2 트랙(track)과 셀(cell), 그리고 라인(line) 📏

Grid는 **눈에 안 보이는 표**라고 생각하시면 됩니다.

* **열(column track)**: 세로 줄 하나
* **행(row track)**: 가로 줄 하나
* **셀(cell)**: 행 + 열이 교차하는 칸 하나
* **라인(line)**: 칸을 나누는 경계선 (선 번호로 아이템 위치를 지정할 수 있음)

예를 들어 3열 그리드를 만들면:

```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px; /* 3개의 column track */
}
```

* 열 라인 번호: `1 | 2 | 3 | 4` (항상 “트랙 개수 + 1”개)
* 나중에 `grid-column: 1 / 3` → “라인 1에서 시작해서 라인 3 직전까지(1~2열)를 차지해라” 같은 식으로 사용

---

## 3. Grid의 핵심 속성들 ⚙️

### 3.1 `grid-template-columns` / `grid-template-rows`

Grid의 **행/열 구조를 정의**합니다.

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 1fr;
  grid-template-rows: 100px auto;
}
```

* 열:

  * 첫 번째 열: `200px`
  * 두 번째/세 번째 열: `1fr`, `1fr` → 남은 공간을 비율로 나눔
* 행:

  * 첫 번째 행: `100px`
  * 두 번째 행: `auto` → 콘텐츠 높이에 따라 결정

---

### 3.2 `fr` 단위 – Grid 전용 유닛 💧

`fr`은 **fraction(비율)**의 약자입니다.

```css
grid-template-columns: 1fr 2fr 1fr;
```

* 전체를 1+2+1 = 4 “조각”으로 나누고

  * 첫 번째 열: 1/4
  * 두 번째 열: 2/4
  * 세 번째 열: 1/4

이렇게 **여유 공간을 비율로 나누기 때문에**, “양쪽 여백 빼고 가운데 컬럼만 크게” 같은 구조를 매우 쉽게 만들 수 있습니다.

---

### 3.3 `repeat()` 함수 – 반복 줄이기 🔁

```css
grid-template-columns: repeat(3, 1fr); /* 1fr 1fr 1fr 과 동일 */
grid-template-rows: repeat(2, 200px);  /* 200px 200px */
```

* `repeat(개수, 크기)` 형태
* `repeat(3, minmax(200px, 1fr))`처럼 복잡한 패턴도 반복 가능

---

### 3.4 `minmax()` – 최소/최대 범위 지정 📐

```css
grid-template-columns: repeat(3, minmax(200px, 1fr));
```

* 각 열은

  * 최소 200px
  * 최대 1fr (남은 공간 비율)
* 창이 좁아지면 200px 이하로는 줄어들지 않음
* 반응형 레이아웃에서 굉장히 자주 쓰입니다.

---

### 3.5 `auto-fill` / `auto-fit` – 자동으로 칸 수 맞추기 🧩

**카드 그리드, 갤러리** 같은 곳에서 필수템입니다.

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}
```

* 컨테이너의 가로 폭 안에

  * 200px 이상 되는 칸을 가능한 한 많이 채워 넣음
* 화면이 넓으면 칸 수가 많아지고, 좁으면 자동으로 줄어듦 → **반응형 카드 그리드 패턴**

`auto-fit`과 `auto-fill`의 차이:

* `auto-fill`: “공간을 채울 수 있는 만큼 칸을 잡아두고, 비어 있는 칸도 유지”
* `auto-fit`: “실제로 내용이 있는 칸 위주로, 빈 칸은 접어서(축소) 꽉 맞게”

실무에서는 보통 카드 그리드에서 `auto-fit`을 더 많이 씁니다.

```css
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
```

---

### 3.6 `gap`, `row-gap`, `column-gap` – 간격 설정 🧱🧱

```css
.container {
  display: grid;
  gap: 16px;          /* row + column 둘 다 16px */
  row-gap: 8px;       /* 행 간격 */
  column-gap: 24px;   /* 열 간격 */
}
```

Flexbox의 `gap`과 동일한 느낌이지만, Grid에서는 특히 **표처럼 보이는 레이아웃에 간격을 적용**할 때 자주 사용합니다.

---

## 4. Grid Item 배치하기 🎯

### 4.1 기본 배치 – 자동 배치(auto placement)

아무 속성도 지정하지 않으면, Grid는 아래 규칙으로 아이템을 채웁니다.

1. 첫 번째 행의 왼쪽부터 오른쪽으로 차례대로
2. 꽉 차면 다음 행으로 넘어감

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}
```

```html
<div class="container">
  <div>A</div> <!-- 1행 1열 -->
  <div>B</div> <!-- 1행 2열 -->
  <div>C</div> <!-- 1행 3열 -->
  <div>D</div> <!-- 2행 1열 -->
  <div>E</div> <!-- 2행 2열 -->
</div>
```

---

### 4.2 라인 번호로 위치 지정 – `grid-column`, `grid-row` 📌

```css
.item1 {
  grid-column: 1 / 3; /* 1~2열 차지 */
  grid-row: 1 / 2;    /* 1행만 차지 */
}

.item2 {
  grid-column: 3 / 4; /* 3열 */
  grid-row: 1 / 3;    /* 1~2행 세로로 길게 */
}
```

* `grid-column: 시작라인 / 끝라인`
* `grid-row: 시작라인 / 끝라인`
* `span` 키워드로 “몇 칸을 늘려라”도 가능

```css
.item3 {
  grid-column: span 2; /* 현재 위치에서 오른쪽으로 2칸 차지 */
  grid-row: span 1;
}
```

---

### 4.3 `grid-area` – 한 번에 지정하기

```css
.item1 {
  grid-area: 1 / 1 / 2 / 3;
  /* grid-row-start / grid-column-start / grid-row-end / grid-column-end */
}
```

* 가독성은 떨어질 수 있지만, 면적을 한 번에 표현할 수 있어요.

---

## 5. `grid-template-areas`로 레이아웃 설계하기 🗺️

Grid의 강력한 기능 중 하나가 **“영역 이름” 기반 레이아웃입니다.**

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: 80px 1fr 60px;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  gap: 8px;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

```html
<div class="container">
  <header class="header">Header</header>
  <aside  class="sidebar">Sidebar</aside>
  <main   class="main">Main</main>
  <footer class="footer">Footer</footer>
</div>
```

장점:

* **레이아웃이 코드만 봐도 눈에 보입니다.**

```css
grid-template-areas:
  "header header"
  "sidebar main"
  "footer footer";
```

이 부분만 봐도 “위에 헤더, 왼쪽 사이드바, 오른쪽 메인, 아래 푸터” 레이아웃이 바로 떠오르죠.

---

## 6. 정렬 속성 – 컨텐츠/아이템 정렬 ✨

Grid도 Flexbox처럼 정렬 관련 속성이 있습니다.

### 6.1 전체 그리드 영역 내에서 트랙 정렬

* `justify-content` : **수평 방향(인라인 축)**으로 트랙 전체를 정렬
* `align-content` : **수직 방향(블록 축)**으로 트랙 전체를 정렬
* `place-content: align justify`의 단축

예를 들어, 컨테이너보다 트랙 전체 너비가 작을 때:

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(2, 100px);
  justify-content: center; /* 수평 중앙 정렬 */
  align-content: center;   /* 수직 중앙 정렬 */
}
```

---

### 6.2 각 셀 내부에서 아이템 정렬

* `justify-items`: 각 셀 안에서 **수평 정렬**
* `align-items`: 각 셀 안에서 **수직 정렬**
* `place-items: align justify` 단축

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  justify-items: center; /* 각 칸 내부에서 가운데 정렬 */
  align-items: center;
}
```

* 개별 아이템에만 다르게 적용하고 싶다면:

  * `justify-self`, `align-self` 사용

```css
.item-special {
  justify-self: end;   /* 오른쪽 정렬 */
  align-self: start;   /* 위쪽 정렬 */
}
```

---

## 7. 암시적(implicit) 트랙 – 자동으로 생기는 행/열 🕳️

정의한 `grid-template-rows`/`columns`보다 아이템이 더 많으면, Grid는 **자동으로 새로운 행이나 열을 추가**합니다.

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  /* grid-template-rows는 1줄만 정의했다고 가정 */
  grid-auto-rows: 150px; /* 암시적 행의 높이를 지정 */
}
```

* 아이템이 많아져 2행, 3행이 필요해지면

  * 그 행들의 높이는 `grid-auto-rows`에 의해 150px로 결정

비슷하게 열에 대해서는 `grid-auto-columns`가 있습니다.

---

## 8. 실전 패턴 1 – 카드 그리드 레이아웃 🎴

### 목표

* 화면 넓으면 4칸
* 중간이면 3칸
* 더 좁으면 2칸, 1칸으로 줄어드는 카드 그리드

```css
.cards {
  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
}
.card {
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
}
```

```html
<section class="cards">
  <article class="card">Card 1</article>
  <article class="card">Card 2</article>
  <article class="card">Card 3</article>
  <article class="card">Card 4</article>
  <article class="card">Card 5</article>
</section>
```

* 핵심은 `grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));`
* 이 패턴 하나만 알아도 **반응형 카드/갤러리는 거의 다 해결**됩니다.

---

## 9. 실전 패턴 2 – “Holy Grail Layout” 📚

클래식한 3단 레이아웃:

* 위에 헤더
* 가운데: 왼쪽 사이드바 + 메인 콘텐츠 + 오른쪽 사이드바
* 아래 푸터

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
}

.header  { grid-area: header;  background: #eee; }
.sidebar { grid-area: sidebar; background: #f4f4f4; }
.main    { grid-area: main;    background: #fff; }
.aside   { grid-area: aside;   background: #f4f4f4; }
.footer  { grid-area: footer;  background: #eee; }
```

```html
<div class="layout">
  <header class="header">Header</header>
  <nav class="sidebar">Left Sidebar</nav>
  <main class="main">Main Content</main>
  <aside class="aside">Right Sidebar</aside>
  <footer class="footer">Footer</footer>
</div>
```

이걸 Flexbox로 구현하려면 꽤 머리 아픈데,
Grid에서는 **layout의 표 구조를 그대로 코드로 옮겨놓는 느낌**이라 이해하기 더 쉽습니다.

---

## 10. Grid + Tailwind CSS를 함께 쓸 때 (보너스) 🐬

지금 사용 중이신 Vite + React + Tailwind 환경을 기준으로 하면:

```jsx
<div className="grid grid-cols-3 gap-4">
  <div className="bg-slate-100 p-4">A</div>
  <div className="bg-slate-100 p-4">B</div>
  <div className="bg-slate-100 p-4">C</div>
  <div className="bg-slate-100 p-4">D</div>
</div>
```

Tailwind의 grid 관련 유틸:

* `grid` / `inline-grid`
* `grid-cols-3`, `grid-cols-4`, `grid-rows-2` …
* `grid-cols-[200px_1fr]`처럼 임의 값도 가능
* `auto-cols-fr`, `auto-rows-min` 등
* `gap-4`, `gap-x-2`, `gap-y-4`
* `place-items-center`, `justify-items-center`, `items-start` …

Tailwind는 결국 **CSS Grid 프로퍼티들을 클래스로 쪼개놓은 것**일 뿐이어서,
지금 설명드린 개념(CSS Grid 원본)을 이해하고 나면 Tailwind 유틸이 훨씬 자연스럽게 보일 거예요.

---

## 11. 정리 🧾

CSS Grid Layout의 핵심은:

1. **2D 레이아웃(행 + 열)을 한 번에 설계**하는 시스템
2. `grid-template-columns` / `rows`, `fr`, `repeat()`, `minmax()`로 “표 구조”를 먼저 잡는다
3. `gap`, `justify-items`, `align-items`, `place-*`로 정렬과 간격을 다듬는다
4. `grid-template-areas`를 쓰면 레이아웃이 눈에 보이듯이 표현된다
5. `auto-fit` + `minmax()` 패턴은 반응형 카드/갤러리의 핵심
6. Tailwind를 쓰더라도 **기본은 CSS Grid 개념**이기 때문에, 원리를 이해하면 어떤 프레임워크에서도 응용 가능

