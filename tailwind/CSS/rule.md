CSS에서 말하는 **“rule(규칙)”** 은
👉 **“어떤 요소를 어떻게 꾸밀지에 대한 한 줄 짜리 약속서”** 라고 보시면 됩니다.

이걸 제대로 이해하면
셀렉터, 선언, 우선순위(캐스캐이딩)까지 한 번에 머릿속에 구조가 잡혀요.
차근차근 “엔진 분해”하듯이 살펴보겠습니다. 🛠️

---

## 1. CSS Rule이란? 🧾

CSS 파일 안에는 이런 코드들이 잔뜩 있죠:

```css
p {
  color: red;
  font-size: 16px;
}
```

이 **전체 블록 하나**를 바로 **CSS 규칙(rule)** 이라고 부릅니다.

* “어떤 요소를” → `p` (셀렉터)
* “어떻게 꾸민다” → `{ color: red; font-size: 16px; }` (선언 블록)

즉,

> **CSS Rule = Selector + Declaration Block**

으로 이루어진 구조입니다.

---

## 2. CSS Rule의 기본 구조 🔧

가장 기본적인 규칙의 형태는 다음과 같습니다.

```css
selector {
  property: value;
  property2: value2;
}
```

예를 들어:

```css
.btn-primary {
  background-color: blue;
  color: white;
  padding: 0.5rem 1rem;
}
```

여기서 각각의 의미는:

1. **Selector(셀렉터)** – `.btn-primary`

   * “어떤 HTML 요소들에 이 스타일을 적용할지”를 지정합니다.
   * 예: `p`, `.btn`, `#header`, `ul li a:hover` 등

2. **Declaration Block(선언 블록)** – `{ ... }`

   * `{`와 `}` 사이에 들어있는 전체 내용을 말합니다.

3. **Declaration(선언)** – `background-color: blue;` 같은 한 줄

   * `property: value;` 형식을 따르는 한 줄이 하나의 선언입니다.

4. **Property(속성)** – `background-color`, `color`, `padding` …

   * “무엇을 바꿀 것인가”

5. **Value(값)** – `blue`, `white`, `0.5rem 1rem` …

   * “어떻게 바꿀 것인가”

한 줄씩 분해해 보면 이런 느낌입니다:

```css
.btn-primary {              /* 🔹 rule 전체 */
  background-color: blue;   /* 🔸 declaration (property + value) */
  color: white;             /* 🔸 declaration */
}
```

---

## 3. Style Rule vs At-Rule(일반 규칙 vs @규칙) ⚖️

CSS의 “rule”이라고 할 때는 크게 두 종류로 나눌 수 있습니다.

1. **Style Rule (스타일 규칙)** – 우리가 가장 많이 쓰는 형태

   ```css
   h1 {
     font-size: 2rem;
     color: #333;
   }
   ```

   * 셀렉터 + 선언 블록
   * HTML 요소에 직접 스타일을 적용

2. **At-Rule (@규칙)** – `@`로 시작하는 특별한 규칙

   ```css
   @import url('reset.css');

   @media (max-width: 768px) {
     .sidebar {
       display: none;
     }
   }

   @keyframes fade-in {
     from { opacity: 0; }
     to { opacity: 1; }
   }
   ```

   대표적인 @규칙들:

   * `@import` – 다른 CSS 파일 가져오기
   * `@media` – 반응형 조건부 규칙
   * `@font-face` – 웹 폰트 정의
   * `@keyframes` – 애니메이션 정의
   * `@supports` – 브라우저가 특정 기능을 지원하는지 체크

> 정리하면:
> **Style Rule**은 “요소를 꾸미는 규칙”이고,
> **At-Rule**은 “CSS 전체 동작을 제어하는 메타 규칙”에 가깝습니다.

---

## 4. 하나의 Rule 안에 여러 선언: 선언(Declaration)들 🧱

하나의 규칙(rule)은 **여러 줄의 선언**을 가질 수 있습니다.

```css
.card {
  width: 300px;
  height: 200px;
  padding: 16px;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}
```

여기서 `.card` 규칙은 다음의 선언들을 포함합니다:

* `width: 300px;`
* `height: 200px;`
* `padding: 16px;`
* `border-radius: 8px;`
* `background-color: #fff;`
* `box-shadow: ...;`

각 선언은 **세미콜론(`;`)으로 끝나야** 하며,
브라우저는 이 선언들을 차례대로 읽고 적용합니다.

> 💡 마지막 줄에도 세미콜론을 붙이는 습관을 들이면
> 나중에 선언을 추가/삭제할 때 에러가 줄어듭니다.

---

## 5. 그룹 셀렉터: 여러 요소에 한 번에 규칙 적용하기 🎯

하나의 규칙(rule)에 **여러 셀렉터**를 동시에 걸 수도 있습니다.
이때 셀렉터들을 `,`(콤마)로 구분합니다.

```css
h1,
h2,
h3 {
  font-family: 'Pretendard', system-ui, sans-serif;
  color: #222;
}
```

의미:

* `h1`, `h2`, `h3` **모두** 같은 규칙을 공유
* 중복 스타일을 줄이고, 관리하기 쉬워짐

---

## 6. 셀렉터 조합에 따른 Rule 예시 🔍

같은 “규칙” 구조라도 셀렉터를 어떻게 쓰느냐에 따라 의미가 달라집니다.

```css
/* 태그 셀렉터 */
p {
  line-height: 1.6;
}

/* 클래스 셀렉터 */
.btn {
  padding: 8px 12px;
}

/* 아이디 셀렉터 */
#main-header {
  background-color: #fafafa;
}

/* 자식 결합자 */
nav > a {
  margin-right: 12px;
}

/* 후손 결합자 */
nav a {
  text-decoration: none;
}

/* 가상 클래스 */
a:hover {
  text-decoration: underline;
}

/* 가상 요소 */
p::first-line {
  font-weight: bold;
}
```

**규칙의 형태는 모두 동일**하지만,
셀렉터가 달라지면 “어떤 요소에 적용되느냐”가 달라질 뿐입니다.

---

## 7. !important: 규칙의 “우선권”을 강제로 올리기 🚨

각 선언에는 `!important`를 붙여 **우선순위를 강제로 올릴 수** 있습니다.

```css
.alert {
  color: red !important;
}
```

의미:

* 이 규칙보다 나중에 나오는, 더 구체적인 규칙이라도
  `color`를 덮어쓰지 못하게 만듭니다.

하지만…

> ⚠️ **주의:** `!important`는 “최후의 수단”으로만 사용하셔야 합니다.
> 너무 남발하면 스타일 구조가 꼬여서 디버깅이 지옥이 됩니다.

---

## 8. 브라우저는 Rule들을 어떻게 처리할까? (캐스케이딩) 🌊

CSS = **Cascading Style Sheets**
→ “규칙들이 *폭포처럼* 위에서 아래로 떨어지며 합쳐진다”는 뜻입니다.

브라우저는 대략 이런 순서로 규칙을 처리합니다:

1. **모든 CSS 파일과 `<style>` 태그, `style=""` 속성을 모은다.**
2. 각각을 **Rule 단위로 파싱**한다.
3. HTML의 각 요소마다 **어떤 규칙의 셀렉터와 매칭되는지** 검사한다.
4. 매칭된 규칙들을 **우선순위에 따라 정렬**한다.

   * 브라우저 기본 스타일 < 외부 CSS < `<style>` 내부 < `style=""` 인라인
   * `!important`가 붙은 선언은 가장 높은 우선순위
   * 같은 속성끼리는 **특정성(specificity)** + **나중에 나온 규칙**이 이김
5. 최종적으로 결정된 **property → value**를 렌더링에 사용한다.

여기서 포인트는:

> **각 CSS Rule이 결국 “후보 중 하나”로 들어가고,
> 캐스케이딩 규칙에 의해 누가 최종 승자가 될지가 결정된다**는 것입니다.

---

## 9. 잘못된 Rule / 선언은 어떻게 될까? (에러 처리) 🧨

CSS는 **꽤 관대한 언어**입니다.
규칙 일부가 잘못되어도, 가능한 부분은 최대한 적용하려고 합니다.

예:

```css
.box {
  color: red;
  foo-bar: 123;      /* ❌ 존재하지 않는 속성 → 브라우저가 무시 */
  width: 100px;
  background: blu;   /* ❌ 잘못된 값 → 해당 선언만 무시 */
}
```

* **알 수 없는 속성 이름** → 그 선언만 무시
* **알 수 없는 값** → 그 선언만 무시
* 나머지 선언들은 정상적으로 적용됩니다.

> 즉, 하나의 Rule 안에서 일부 선언이 잘못되더라도
> “나머지”는 살아남아서 적용됩니다.

---

## 10. Shorthand vs Longhand: 한 Rule 안에서 요약/상세 섞기 ✂️

같은 Rule 안에서 어떤 속성은 **축약형(shorthand)**,
어떤 속성은 **개별 속성(longhand)**으로 쓸 수 있습니다.

```css
.box {
  margin: 10px 20px;      /* shorthand */
  margin-top: 5px;        /* longhand */
}
```

이 경우:

* `margin: 10px 20px;` → top/bottom = 10px, left/right = 20px
* `margin-top: 5px;` → 나중에 선언되었으므로, top은 5px로 덮어씀

속성 간 우선순위도 **“나중에 나온 선언이 이긴다”** 원칙을 따릅니다.

---

## 11. Rule을 어떻게 정리하면 좋을까? (베스트 프랙티스) ✅

### 11-1. 관련 속성끼리 묶기

```css
.card {
  /* 레이아웃 관련 */
  display: flex;
  align-items: center;
  justify-content: space-between;

  /* 박스 크기 관련 */
  width: 320px;
  padding: 16px;

  /* 스타일 관련 */
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}
```

같은 Rule 안에서도 성격이 비슷한 속성들을 묶어 정렬해두면
나중에 수정할 때 훨씬 이해하기 쉽습니다.

---

### 11-2. 주석으로 Rule의 의도를 분명히 하기 💬

```css
/* 메인 네비게이션 바 */
.navbar {
  position: sticky;
  top: 0;
  z-index: 1000; /* 항상 상단에 보이게 */
  background-color: white;
  border-bottom: 1px solid #eee;
}
```

* “이 Rule은 어떤 역할을 하는지”
* “왜 이런 속성 값이 들어갔는지”

를 꼭 적어두시면,
몇 달 후의 ‘미래의 나’가 정말 고마워합니다. 😂

---

### 11-3. 지나치게 거대한 Rule은 나누기

```css
/* ❌ 안 좋은 예: 하나의 규칙에 너무 많은 역할을 때려넣은 경우 */
.card {
  /* 레이아웃 + 색상 + 사이즈 + 상태 + 반응형까지… */
}
```

vs

```css
/* ✅ 좋은 예: 역할별로 나눈 클래스 + 규칙 */
.card {
  /* 기본 박스 스타일 */
}

.card--primary {
  /* 색상 변형 */
}

.card--hoverable:hover {
  /* hover 상태 */
}
```

CSS Rule도 함수처럼 **단일 책임(Single Responsibility)**에 가깝게 설계하면
재사용, 유지보수가 훨씬 쉬워집니다.

---

## 12. 전체 그림 요약 🧠

* **CSS Rule = 셀렉터 + 선언 블록**
* 선언 블록은 **`property: value;`** 형식의 **선언들(declarations)** 묶음
* CSS Rule에는

  * **일반 스타일 규칙 (Style Rule)** 과
  * `@media`, `@keyframes` 같은 **At-Rule**이 있음
* 브라우저는

  1. 모든 Rule을 읽고
  2. 각 요소와 매칭되는 규칙을 찾고
  3. 우선순위(특정성, !important, 선언 순서 등)에 따라
  4. 최종 스타일을 결정
* 잘못된 선언이 있어도, **그 줄만 무시**하고 나머지는 정상 적용
* Rule을 쓸 때는

  * 관련 속성끼리 묶고
  * 주석을 잘 달고
  * 역할별로 클래스를 나눠 관리하면
  * 나중에 “내가 쓴 CSS”를 내가 이해할 수 있게 됩니다 😅

