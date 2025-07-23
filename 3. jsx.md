
# 📘 JSX란?

## ✅ 정의

**JSX (JavaScript XML)** 는 **JavaScript 문법 안에서 HTML 태그 문법을 작성할 수 있게 해주는 확장 문법(Syntax Extension)** 입니다.

* React의 UI 선언을 더 **직관적이고 선언적**으로 작성할 수 있게 돕습니다.
* XML/HTML 문법처럼 보이지만, **브라우저가 직접 해석하지 않습니다.**
* Babel 등의 **트랜스파일러에 의해 `React.createElement()` 호출로 변환**됩니다.

---

## 💡 예시로 보는 JSX

### 📌 JSX 문법

```jsx
const element = <h1>Hello, JSX!</h1>;
```

> 이 코드는 실제로 아래와 같은 **JavaScript 코드**로 변환됩니다:

```js
const element = React.createElement("h1", null, "Hello, JSX!");
```

즉, `React.createElement(type, props, ...children)` 형식으로 **함수 호출 형태로 바뀌는 것**입니다.

---

## 🧬 JSX 변환 원리

JSX는 Babel 등을 통해 **React.createElement 호출로 트랜스파일됨**:

### 원래 코드:

```jsx
const greeting = <h1 className="title">Hello, JSX!</h1>;
```

### 변환 결과 (Babel):

```js
const greeting = React.createElement(
  'h1',
  { className: 'title' },
  'Hello, JSX!'
);
```

이렇게 생성된 `greeting` 객체는 다음과 같은 **가상의 React Element 구조체**입니다:

```js
{
  type: 'h1',
  props: {
    className: 'title',
    children: 'Hello, JSX!'
  }
}
```

> 이 객체는 **Virtual DOM 트리의 구성 요소**로 활용됩니다.

---

## 🎯 JSX의 특징

| 특징        | 설명                                     |
| --------- | -------------------------------------- |
| 선언형       | `<h1>Hello</h1>`처럼 HTML 유사 문법으로 UI를 선언 |
| 타입 안전     | 문법 오류 시 컴파일 단계에서 감지 가능                 |
| 자바스크립트 통합 | `{}` 내부에 JS 표현식 사용 가능                  |
| XML 유사 문법 | 닫는 태그 필수 (`<br />`)                    |

---

## 🧪 JSX 내부에 JavaScript 쓰기

```jsx
const name = "intheeast";
const element = <h1>Hello, {name}!</h1>; // Hello, intheeast!
```

`{}` 내부에 JavaScript 표현식을 자유롭게 쓸 수 있습니다:

```jsx
const now = new Date();
const element = <p>{now.toLocaleTimeString()}</p>;
```

---

## ⚠️ JSX에서 주의할 점

* 모든 태그는 **닫혀 있어야 함**: `<br> ❌` → `<br /> ✅`
* **속성 이름은 camelCase** 사용: `class → className`, `for → htmlFor`
* 여러 요소는 반드시 하나의 부모로 감싸야 함 (또는 Fragment 사용)

```jsx
// ❌ 에러
return (
  <h1>Hi</h1>
  <p>Hello</p>
)

// ✅ 가능
return (
  <div>
    <h1>Hi</h1>
    <p>Hello</p>
  </div>
)

// ✅ 또는
return (
  <>
    <h1>Hi</h1>
    <p>Hello</p>
  </>
)
```

---

## 🧱 JSX가 필요한 이유

| 기존 방식 (`React.createElement`) | JSX 방식                  |
| ----------------------------- | ----------------------- |
| 가독성이 낮음                       | 가독성 높음                  |
| 중첩 복잡도 증가                     | 트리 구조 표현에 직관적           |
| UI 구조 파악 어려움                  | HTML-like 표현으로 구조 파악 쉬움 |

---

## 🛠 JSX를 사용하는 기술 스택

* **React**
* **Preact** (React 호환)
* **Solid.js** (JSX 파싱 후 DOM 조작 코드 생성)
* **Babel** (JSX를 JS로 변환)

JSX는 **브라우저가 인식하지 못하기 때문에**, 반드시 `Babel` 등의 트랜스파일러가 필요합니다.

---

## 📌 결론

* JSX는 **React 컴포넌트의 UI를 선언적이고 직관적으로 표현**하게 해주는 **문법 확장**입니다.
* 브라우저가 이해하는 JS 코드로 **React.createElement() 호출 형태로 변환**되어 실행됩니다.
* JSX는 문법이 아니라, **React 생태계의 코드 작성 패러다임**이라 이해하는 것이 더 정확합니다.
