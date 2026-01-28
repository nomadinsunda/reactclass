CSS `display`는 **“이 요소를 레이아웃 엔진이 *어떤 방식으로* 취급할 것인가?”**를 결정하는, 레이아웃의 핵심 스위치입니다.
(박스 모델이 “상자 구조”라면, `display`는 “상자를 *어떻게 배치할지* 결정하는 모드 전환 레버”라고 보시면 됩니다.) 💡


---

## 1. `display` 한 줄 정의 🧩

> **display = “이 요소가 레이아웃에 참여하는 방식” + “자식들을 어떤 레이아웃 규칙으로 배치할지”**

조금 전문적으로 말하면:

* **바깥쪽(outer) 타입**

  * `block`, `inline`, `none` 처럼
  * “이 요소 자신이 주변 요소들과 *어떻게* 줄을 서는지”를 결정
* **안쪽(inner) 타입**

  * `flow`, `flex`, `grid`, `table` 처럼
  * “이 요소의 자식들을 *어떤 레이아웃 시스템*으로 배치할지”를 결정

현실 코드에서는 보통 이렇게 합쳐서 씁니다:

* `display: block`  → block + flow
* `display: inline` → inline + flow
* `display: flex`   → block + flex
* `display: grid`   → block + grid

“바깥쪽이 block/inline, 안쪽이 flex/grid/flow”라고 대략 기억해 두시면 됩니다. 👍

---

## 2. 가장 기초: block vs inline vs inline-block 📏

### 2-1. `display: block` – 한 줄을 혼자 먹는 박스 🧱

```html
<div class="box"></div>
```

```css
.box {
  display: block;
  width: 300px;
  height: 100px;
  background: tomato;
}
```

특징:

* **항상 새 줄에서 시작** (앞뒤로 줄바꿈)
* **가능하면 가로로 꽉 채우려 함** (width를 안 주면 부모 너비 100%)
* `width`, `height`, `margin`, `padding` 모두 정상 동작
* 대표 태그: `div`, `p`, `h1`~`h6`, `ul`, `ol`, `section`…

Tailwind: `block` 클래스 = `display: block;`

---

### 2-2. `display: inline` – 텍스트처럼 흘러가는 인라인 요소 ✍️

```html
<span class="tag">Hello</span>
<span class="tag">World</span>
```

```css
.tag {
  display: inline;
  background: gold;
  padding: 4px;
}
```

특징:

* **텍스트 줄 안에서 흐름(flow)에 따라 배치**
* **새 줄을 차지하지 않음** (옆에 계속 이어 붙음)
* `width`, `height`를 직접 줘도 **거의 무시**됨
* `margin-top`, `margin-bottom`도 사실상 반영 안 됨
* 대표 태그: `span`, `a`, `strong`, `em`, `img(특수케이스)` …

Tailwind: `inline` 클래스 = `display: inline;`

---

### 2-3. `display: inline-block` – 줄은 같이 쓰고, 크기는 갖고 싶다 😎

```html
<button class="btn">OK</button>
<button class="btn">Cancel</button>
```

```css
.btn {
  display: inline-block;
  width: 100px;
  height: 40px;
  background: royalblue;
}
```

특징:

* **한 줄에 여러 개 나란히 배치** (inline처럼)
* **width / height 설정 가능** (block처럼)
* 버튼, 탭, 라벨 같은 “작은 박스 UI” 만들 때 유용

Tailwind: `inline-block` 클래스 = `display: inline-block;`

---

## 3. 현대 레이아웃의 핵심: flex & grid 🧭

이제부터는 **“컨테이너의 display”**가 포인트입니다.

### 3-1. `display: flex` – 1차원(가로 또는 세로) 레이아웃 🧵

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

```css
.container {
  display: flex;          /* 여기! */
  gap: 8px;
}

.item {
  padding: 8px 16px;
  background: #eee;
}
```

* 이 **컨테이너**는 “flex formatting context”를 생성
* 자식들은 **“플렉스 아이템”**이 됨
* `flex-direction`, `justify-content`, `align-items` 같은 속성으로
  가로/세로 정렬, 간격, 정렬, 늘이기/줄이기 등을 제어

Tailwind와 연결하면:

* `flex` → `display: flex;`
* `flex-row` / `flex-col`
* `justify-center` / `items-center`
  등등이 전부 `display: flex;`에서 파생된 속성들입니다.

---

### 3-2. `display: grid` – 2차원 레이아웃 (행 + 열) 🧊

```html
<div class="grid-container">
  <div class="box">1</div>
  <div class="box">2</div>
  <div class="box">3</div>
</div>
```

```css
.grid-container {
  display: grid;                          /* 여기! */
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}
```

* 컨테이너가 **그리드 레이아웃 컨텍스트**를 생성
* 자식들이 **행(row) / 열(column)** 좌표로 배치
* `grid-template-columns`, `grid-template-rows`, `grid-area` 등으로
  레이아웃을 2차원으로 정교하게 설계

Tailwind:

* `grid` → `display: grid;`
* `grid-cols-3`, `gap-2` 등

> **요약**
>
> * `display: flex`  → 1차원 (주로 한 축 기준)
> * `display: grid`  → 2차원 (행/열 기준)

---

## 4. 숨기기 / 제거: `display: none` vs `visibility: hidden` 🙈

### 4-1. `display: none`

```css
.box {
  display: none;
}
```

* **“레이아웃에서 완전히 제거”**
* 해당 요소는 **존재하지 않는 것처럼** 취급
* 공간도 차지하지 않음
* 스크린 리더에서도 기본적으로 제외되는 경우가 많음(접근성 영향)

JavaScript와 함께 많이 쓰죠:

```js
element.style.display = 'none'
element.style.display = 'block'
```

Tailwind: `hidden` = `display: none;`

---

### 4-2. `visibility: hidden` 과의 차이

```css
.box {
  visibility: hidden;
}
```

* 요소는 **여전히 레이아웃에 참여** → 공간을 차지
* 하지만 눈에만 안 보임
* 접근성, 포커스, 이벤트 처리 측면에서 `display: none`과 다르게 동작

> 기억 포인트 ✏️
>
> * **`display: none`** → *“아예 없는 셈”*
> * **`visibility: hidden`** → *“투명 망토 두름 (자리만 있음)”*

---

## 5. 그 외 자주 헷갈리는 `display` 값들 🧪

### 5-1. `display: contents` – 자기 박스만 제거하고, 자식만 남기기

```html
<div class="wrapper">
  <div class="inner">
    <span>Text 1</span>
    <span>Text 2</span>
  </div>
</div>
```

```css
.inner {
  display: contents;
}
```

* `.inner` 요소의 **자기 박스는 날려 버리고**, 자식만 남긴 것처럼 레이아웃
* DOM 구조는 유지하지만, 레이아웃 상에서는 `.inner`의 박스가 없음
* 접근성/포커스, `position`, `overflow` 등에서 이슈가 있을 수 있어 **조심해서 사용**

Tailwind: `contents` 클래스 = `display: contents;`

---

### 5-2. `display: list-item` – 리스트 마커(●, 1. 등) 추가

```html
<div class="li-like">항목</div>
```

```css
.li-like {
  display: list-item;
  list-style-type: disc;
}
```

* `li` 같은 느낌으로 마커가 붙음
* `ul` 밖에서도 리스트 UI처럼 만들 수 있음

---

### 5-3. `display: table` 계열 – 옛날 테이블 레이아웃 흉내

```css
.table {
  display: table;
}
.row {
  display: table-row;
}
.cell {
  display: table-cell;
}
```

* HTML `<table>`, `<tr>`, `<td>`처럼 동작하게 만드는 값들
* 요즘은 **flex/grid로 대부분 대체** 가능, 그래도 레거시 코드에서 종종 등장

---

### 5-4. `display: flow-root` – 새로운 블록 포맷팅 컨텍스트 만들기

```css
.card {
  display: flow-root;
}
```

* float 정리, margin collapsing 등 레이아웃 문제를 해결하는 데 사용
* 예전에는 `overflow: hidden;` 같은 트릭으로 만들던 “새로운 레이아웃 컨텍스트”를
  좀 더 명시적으로 생성하는 방식

Tailwind: `flow-root` 클래스 = `display: flow-root;`

---

## 6. “display의 디폴트값이 뭐냐?” 질문 정리 🧠

이 부분 때문에 헷갈리셨을 것 같아서 **정리 존**을 하나 파겠습니다.

### 6-1. CSS 스펙 상 “초깃값(initial value)”

CSS 표준 문서 기준으로 **`display`의 initial value는 `inline`** 입니다.

```css
/* 아무 스타일도 안 주면, 속성 자체의 초기값은 inline */
.some-element {
  display: inline; /* 스펙 상 초깃값 */
}
```

하지만… 실제 HTML 렌더링은 이렇게 되어 있지 않죠. `div`는 block이고, `span`은 inline입니다.

### 6-2. 그럼 왜 `<div>`는 기본이 block인가요? 🤔

브라우저는 HTML을 렌더링할 때 **“UA(User Agent) 스타일시트”**를 기본으로 적용합니다.

대충 이런 느낌의 CSS를 브라우저가 내부적으로 가지고 있다고 보시면 됩니다:

```css
/* 브라우저 안에 내장된 기본 CSS (컨셉) */
div { display: block; }
p   { display: block; }
h1  { display: block; }
span { display: inline; }
a    { display: inline; }
img  { display: inline; }
button { display: inline-block; }
ul, ol { display: block; }
li { display: list-item; }
```

그래서 정리하면:

* **CSS 스펙 상 초깃값**: `display: inline;`
* **HTML 요소마다의 기본값**: 브라우저 UA 스타일시트에 정의된 값

  * `<div>` → `display: block;`
  * `<span>` → `display: inline;`
  * `<li>` → `display: list-item;`
  * `<button>` → 보통 `inline-block` 느낌

> 질문에 대한 한 줄 답변 ✍️
>
> * “`display` 속성의 **디폴트값**은?”
>
>   * *스펙 관점*: `inline`
>   * *HTML 태그 관점*: 태그마다 다름 (`div=block`, `span=inline` 등)

---

## 7. `display`와 박스 모델, 그리고 다른 속성들과의 관계 🧷

### 7-1. `display` vs 박스 모델(box model)

* 박스 모델: **“한 박스가 내부적으로 어떻게 생겼느냐”**

  * `content`, `padding`, `border`, `margin`, `box-sizing` 등
* display: **“이 박스를 이웃들과 어떻게 배치할 것이냐”**

  * block/inline/flex/grid/none …

둘은 **서로 다른 층(layer)** 입니다.

* `display`를 바꿔도 `padding`, `border` 값은 그대로
* 다만 **어떤 값들이 실제로 적용되느냐**는 `display`에 영향을 받음

  * `inline` 요소는 `width`, `height`가 사실상 무시
  * `block`, `flex`, `grid`에서는 잘 적용됨

---

### 7-2. `display` vs `position` vs `opacity`

자주 헷갈리는 삼총사입니다. 😅

* `display: none`

  * 레이아웃에서 제거, 공간 없음, 이벤트도 안 잡힘
* `position: absolute`

  * 레이아웃 흐름에서 빼지만, **보이긴 보임**, 공간도 안 차지
* `opacity: 0`

  * **투명해질 뿐** → 공간 차지, 이벤트는 잡힘 (클릭 가능)

모두 “보이는/안 보이는 것”과 관련 있지만, **레이아웃 참여 방식**을 바꾸는 것은
**오직 `display`와 `position`(일부 값)** 입니다.

---

## 8. 개발하면서 자주 겪는 `display` 관련 버그들 🧨

### 8-1. width/height가 안 먹어요

```css
span {
  width: 200px;
  height: 100px;
  background: tomato;
}
```

→ 안 먹는 이유는 단순합니다. **`span`의 기본 display가 `inline`** 이라서 그래요.

해결:

```css
span {
  display: inline-block; /* 혹은 block, flex, grid 등 */
  width: 200px;
  height: 100px;
}
```

---

### 8-2. margin-top / margin-bottom이 안 먹어요

```css
span {
  display: inline;
  margin-top: 20px;
  margin-bottom: 20px;
}
```

* 인라인 요소는 **수직 방향 margin이 제대로 반영되지 않습니다.**
* 줄 간격은 `line-height`가 관여

해결:

```css
span {
  display: inline-block; /* 또는 block */
  margin-top: 20px;
  margin-bottom: 20px;
}
```

---

### 8-3. flex가 안 먹어요 (`justify-center`, `items-center` 안 먹는 느낌)

Tailwind/Vite 프로젝트에서 아주 자주 나오는 케이스입니다.

```html
<div class="justify-center items-center">
  <!-- ... -->
</div>
```

→ 안 됩니다. 왜냐하면 **`justify-center`, `items-center`는 “flex 컨테이너”일 때만 의미가 있기 때문**입니다.

필수:

```html
<div class="flex justify-center items-center">
  <!-- ... -->
</div>
```

* `flex` → `display: flex;` 를 만들어 줌
* 그 위에 `justify-*/items-*`가 얹혀서 동작

즉, **정렬이 안 되면 가장 먼저 `display`를 의심**해야 합니다.

---

## 9. 요약 차트 📚

| 값              | 역할/특징                       | 자주 쓰는 용도                        |
| -------------- | --------------------------- | ------------------------------- |
| `block`        | 새 줄 차지, 가로 꽉 채우기            | 레이아웃 기본 박스(div, section 등)      |
| `inline`       | 텍스트와 함께 흐름, width/height 무시 | 글자 강조(span, a 등)                |
| `inline-block` | 한 줄에 여러 개 + 크기 지정 가능        | 버튼, 작은 박스 UI                    |
| `none`         | 레이아웃에서 제거, 숨김               | 조건부 렌더링, 토글 UI                  |
| `flex`         | 1차원(가로/세로) 레이아웃 컨테이너        | 수평/수직 정렬, 간단한 레이아웃              |
| `inline-flex`  | 인라인 요소지만 내부는 flex           | 텍스트 줄 안에서 플렉스 박스                |
| `grid`         | 2차원 레이아웃 (행+열)              | 대칭/복잡한 레이아웃                     |
| `contents`     | 부모 박스 제거, 자식만 남기기           | 드문 특수 용도                        |
| `list-item`    | 목록 마커(●, 1. 등) 추가           | 커스텀 리스트 UI                      |
| `table-*`      | 테이블 레이아웃 흉내                 | 레거시 레이아웃, 특수 정렬                 |
| `flow-root`    | 새로운 블록 포맷팅 컨텍스트             | float 문제 해결, margin collapse 방지 |

---

## 마무리 🧵

정리해 보면:

1. `display`는 **레이아웃 모드 스위치**

   * 바깥쪽: block/inline/none
   * 안쪽: flow/flex/grid/table …
2. “디폴트값” 질문은

   * 스펙: `inline`
   * HTML 태그: UA 스타일시트 때문에 태그마다 다름 (`div=block`, `span=inline` 등)
3. flex/grid/Tailwind 정렬이 안 먹으면

   * **가장 먼저 `display`를 확인**해야 함 (`flex`, `grid`, `block`, `inline-block` 등)
4. 숨길 때는

   * `display: none`(완전 제거) vs `visibility: hidden`(자리만 차지)

