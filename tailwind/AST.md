# 🌳 AST (Abstract Syntax Tree)

## 👉 PostCSS / 컴파일러 / React까지 연결되는 “핵심 개념”

---

# 🎯 1. AST란 무엇인가?

> 💡 **AST = 코드의 “구조를 트리 형태로 표현한 것”**
> 💡 AST는 “코드를 이해하고 조작하기 위해 반드시 필요한 중간 표현”이다

우리가 작성하는 코드는 사실 단순 문자열입니다.

```css
.button {
  color: red;
}
```

👉 하지만 컴퓨터는 이걸 이렇게 이해하지 않습니다 ❌
👉 대신 “구조”로 해석합니다 ⭕

---

## 🔍 AST로 보면 이렇게 된다

```text
Root
 └── Rule (.button)
      └── Declaration (color: red)
```

👉 즉,

* 문자열 ❌
* 구조화된 트리 ⭕

---

# 🧠 2. 왜 AST가 중요한가?

## ❌ 문자열 처리의 문제

```js
css.replace("red", "blue");
```

👉 위험합니다 😱

* 다른 위치까지 바뀜
* 의도하지 않은 변경 발생

---

## ✅ AST 기반 처리

👉 “정확한 위치”만 수정 가능

```text
Rule(.button)
 └── Declaration(color: red) → blue
```

👉 구조 기반이라 안전함 👍

---

# ⚙️ 3. AST 생성 과정 (컴파일 파이프라인)

PostCSS / Babel / TypeScript / React 모두 동일한 흐름입니다.

```text
Code (문자열)
   ↓
Tokenizer (토큰 분리)
   ↓
Parser (문법 분석)
   ↓
AST 생성 🌳
   ↓
Transform (변환)
   ↓
Code 출력
```

---

## 🎯 예제: CSS 파싱 흐름

```css
.button {
  display: flex;
}
```

### 1️⃣ 토큰화

```text
.selector(.button)
.brace_open
.property(display)
.value(flex)
.brace_close
```

---

### 2️⃣ AST 생성

```text
Root
 └── Rule (.button)
      └── Declaration (display: flex)
```

---

# 🧩 4. PostCSS에서의 AST 구조

PostCSS는 CSS를 다음과 같은 노드로 구성합니다.

---

## 🏗️ 주요 노드

### 1️⃣ Root

전체 CSS

```text
Root
```

---

### 2️⃣ Rule

```css
.button {}
```

👉 selector 기반 블록

---

### 3️⃣ Declaration

```css
color: red;
```

👉 실제 스타일

---

### 4️⃣ AtRule

```css
@media (max-width: 600px)
```

👉 특수 규칙

---

## 🎯 실제 구조

```css
.button {
  color: red;
}
```

👉 AST

```text
Root
 └── Rule (.button)
      └── Declaration (color: red)
```

---

# 🔌 5. PostCSS 플러그인은 어떻게 AST를 쓰는가?

👉 핵심: **AST를 순회하면서 수정**

---

## 💡 예제: color red → blue 변환

```js
module.exports = () => {
  return {
    postcssPlugin: 'change-color',
    Declaration(decl) {
      if (decl.prop === 'color' && decl.value === 'red') {
        decl.value = 'blue';
      }
    }
  };
};
```

---

## 🔍 동작 과정

```text
AST 순회
 → Declaration 발견
 → 조건 검사
 → 값 변경
```

👉 이게 PostCSS의 본질입니다

---

# ⚡ 6. AST 조작 패턴 (핵심 3가지)

## 1️⃣ Find (탐색)

```js
Declaration(decl)
```

👉 특정 노드 찾기

---

## 2️⃣ Transform (변환)

```js
decl.value = 'blue';
```

---

## 3️⃣ Insert / Remove (구조 변경)

```js
rule.append({ prop: 'margin', value: '0' });
```

👉 새로운 스타일 추가

---

# 🧬 7. AST vs DOM (헷갈리는 포인트)

| 구분    | AST   | DOM      |
| ----- | ----- | -------- |
| 대상    | 코드    | HTML     |
| 생성 시점 | 빌드 시점 | 브라우저 런타임 |
| 목적    | 코드 변환 | UI 렌더링   |

---

👉 핵심 차이

> AST = 컴파일 단계
> DOM = 실행 단계

---

# 🚀 8. AST는 CSS만 쓰는 게 아니다

## 🎯 JavaScript (Babel)

```js
const a = 10;
```

👉 AST

```text
VariableDeclaration
 └── Identifier(a)
 └── Literal(10)
```

---

## 🎯 React (JSX → AST → JS)

```jsx
<div>Hello</div>
```

👉 AST → JS 변환

```js
React.createElement("div", null, "Hello");
```

---

👉 즉,

> 💡 AST는 “모든 컴파일러의 핵심”

---

# 🧠 9. PostCSS AST의 강력함

## 🔥 가능한 작업들

* CSS 자동 변환
* vendor prefix 추가
* Tailwind class 생성
* CSS 최적화 (minify)
* lint 검사
* design token 변환

---

👉 이 모든 것의 기반 = AST

---

# 🧨 10. AST를 이해하면 생기는 변화

## BEFORE 😵

* CSS는 그냥 문자열
* 빌드 도구는 블랙박스

---

## AFTER 😎

* CSS = 구조화된 트리
* PostCSS = AST 변환 엔진
* Tailwind = AST 생성기
* Autoprefixer = AST 변환기

---

# 🏆 11. 핵심 요약

✔ AST = 코드의 구조 트리 🌳
✔ 문자열이 아니라 “구조”로 다룸
✔ PostCSS는 AST를 변환하는 엔진
✔ 플러그인은 AST를 탐색/수정

---

# 🔥 한 줄 정리

> 💡 **AST는 “코드를 데이터처럼 다루기 위한 구조화된 표현”이다**

