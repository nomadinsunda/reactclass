# 🎨 SASS (Syntactically Awesome Style Sheets) 완전 정복 가이드

> CSS를 **프로그래밍 언어처럼 다루게 해주는 전처리기**
> 👉 대규모 프론트엔드 프로젝트에서 **생산성과 유지보수성**을 폭발적으로 높여줍니다.

---

# 🚀 1. SASS란 무엇인가?

**SASS (Syntactically Awesome Style Sheets)**는
기존 CSS를 확장한 **CSS 전처리기(preprocessor)**입니다.

👉 핵심 개념:

```text
SASS 코드 → 컴파일 → CSS
```

즉, 브라우저는 SASS를 이해 못하기 때문에
**반드시 CSS로 변환되어야 합니다.**

---

# 🔥 2. 왜 SASS를 사용하는가?

## ❌ 기존 CSS의 문제점

```css
.button {
  background: blue;
}
.button:hover {
  background: darkblue;
}
.button.active {
  background: navy;
}
```

👉 문제

* 반복 많음 😡
* 구조 파악 어려움 😡
* 유지보수 지옥 😡

---

## ✅ SASS 사용 시

```scss
.button {
  background: blue;

  &:hover {
    background: darkblue;
  }

  &.active {
    background: navy;
  }
}
```

👉 장점

* 구조적 👀
* 가독성 ↑
* 유지보수 ↑

---

# 🧠 3. SASS 핵심 문법 (실무 필수)

---

## 🎯 3.1 변수 (Variables)

```scss
$primary-color: #3498db;
$padding: 16px;

.button {
  background: $primary-color;
  padding: $padding;
}
```

👉 **디자인 토큰 관리 가능**

---

## 🎯 3.2 중첩 (Nesting)

```scss
.nav {
  ul {
    margin: 0;
  }

  li {
    list-style: none;
  }

  a {
    text-decoration: none;
  }
}
```

👉 HTML 구조 그대로 표현 가능

---

## 🎯 3.3 & (부모 선택자 참조)

```scss
.button {
  &-primary {
    color: white;
  }

  &:hover {
    opacity: 0.8;
  }
}
```

👉 BEM 스타일 작성에 매우 유용

---

## 🎯 3.4 믹스인 (Mixins) 💡

> 재사용 가능한 스타일 함수

```scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  @include flex-center;
}
```

👉 **반복 제거의 핵심**

---

## 🎯 3.5 함수 (Functions)

```scss
@function px-to-rem($px) {
  @return $px / 16 * 1rem;
}

h1 {
  font-size: px-to-rem(32);
}
```

👉 계산 로직까지 가능

---

## 🎯 3.6 상속 (Extend)

```scss
%button-base {
  padding: 10px;
  border-radius: 5px;
}

.btn-primary {
  @extend %button-base;
  background: blue;
}
```

👉 공통 스타일 재사용

---

## 🎯 3.7 조건문 & 반복문 🔁

```scss
$theme: dark;

body {
  @if $theme == dark {
    background: black;
    color: white;
  }
}
```

```scss
@for $i from 1 through 3 {
  .col-#{$i} {
    width: 100% / $i;
  }
}
```

👉 CSS에 없는 **프로그래밍 능력 제공**

---

# ⚙️ 4. SCSS vs SASS 문법 차이

## 1️⃣ SCSS (권장 👍)

```scss
.button {
  color: red;
}
```

👉 CSS와 거의 동일 (가장 많이 사용됨)

---

## 2️⃣ SASS (들여쓰기 기반)

```sass
.button
  color: red
```

👉 Python 스타일

---

# 🏗️ 5. 프로젝트 구조 (실무 패턴)

```bash
styles/
 ├── base/
 │   ├── _reset.scss
 │   ├── _typography.scss
 │
 ├── components/
 │   ├── _button.scss
 │   ├── _card.scss
 │
 ├── layout/
 │   ├── _header.scss
 │   ├── _footer.scss
 │
 ├── utils/
 │   ├── _variables.scss
 │   ├── _mixins.scss
 │
 └── main.scss
```

👉 `_파일명.scss` = partial (컴파일 제외)

---

# 🔗 6. Import / Use 시스템

## ❌ 구 방식 (deprecated)

```scss
@import "variables";
```

---

## ✅ 최신 방식 (권장)

```scss
@use "variables" as v;

.button {
  color: v.$primary-color;
}
```

👉 네임스페이스 관리 가능

---

# ⚡ 7. 컴파일 방식

## 1️⃣ CLI

```bash
sass input.scss output.css
```

---

## 2️⃣ Vite / Webpack

👉 자동 컴파일됨

```bash
npm install sass
```

---

# 💥 8. SASS vs Tailwind vs CSS

| 구분       | 특징       |
| -------- | -------- |
| CSS      | 기본       |
| SASS     | 구조 + 재사용 |
| Tailwind | 유틸리티 기반  |

👉 요즘 트렌드

```text
SASS → 점점 감소
Tailwind → 급성장 🚀
```

하지만

👉 **대형 프로젝트 / 디자인 시스템**
→ 아직도 SASS 많이 사용됨

---

# 🧩 9. 언제 SASS를 써야 하는가?

## ✅ 추천 상황

* 디자인 시스템 구축
* 반복 스타일 많을 때
* 팀 협업 (규칙 필요)

## ❌ 비추천

* 작은 프로젝트
* Tailwind 사용하는 경우

---

# 🧠 10. 핵심 요약

✔ CSS를 **프로그래밍화**한 것
✔ 변수 / 함수 / 반복문 제공
✔ 유지보수성 극대화
✔ SCSS 문법이 사실상 표준

---

# 🔥 한 줄 정리

> 💡 **SASS는 "CSS를 개발자스럽게 만든 언어"이다**

