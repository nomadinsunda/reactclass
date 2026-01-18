# 🚀 React.createElement란?

`React.createElement()`는 **React가 JSX를 실제 JavaScript 객체 형태의 요소(Element)로 변환하기 위해 사용하는 핵심 메서드**입니다.

즉, JSX는 단순한 문법 설탕(syntax sugar)일 뿐이며, **React.createElement()가 진짜 렌더링 단위를 생성하는 실체**입니다.

---

# 1️⃣ JSX → React Element 변환 과정

우리가 흔히 작성하는 JSX:

```jsx
const element = <h1 className="title">Hello React</h1>;
```

이 코드는 Babel이 다음의 순수 JS 코드로 변환합니다:

```js
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello React'
);
```

즉, JSX가 있든 없든 React는 내부적으로 **항상 createElement 함수를 호출함**으로써 React Element(정확히는 **React Fiber Node의 기초 데이터**)를 생성합니다.

---

# 2️⃣ React.createElement의 시그니처

```ts
React.createElement(
  type,
  props,
  ...children
)
```

### 🔍 각 인자의 의미

| 인자         | 설명                                |
| ---------- | --------------------------------- |
| `type`     | 태그 문자열(‘div’) 또는 컴포넌트 함수/클래스      |
| `props`    | 속성 객체 (className, id, onClick 등)  |
| `children` | 자식 요소(문자열, 숫자, 배열, 또 다른 React 요소) |

---

# 3️⃣ createElement가 반환하는 것은 "DOM"이 아님!

가장 중요한 사실:

## ❗ createElement는 **DOM을 생성하지 않습니다.**

DOM 렌더링은 ReactDOM이 합니다.

createElement의 반환값은 **React Element라는 단순한 JS 객체**입니다.

예시:

```js
const element = React.createElement('div', { id: 'box' }, 'Hello');
```

생성되는 실제 객체를 콘솔로 찍어보면 다음과 비슷합니다:

```js
{
  $$typeof: Symbol(react.element),
  type: "div",
  key: null,
  ref: null,
  props: {
    id: "box",
    children: "Hello"
  }
}
```

즉,

### ✔️ React Element = 가상 DOM(Virtual DOM)을 구성하는 JSON 구조

### ✔️ 브라우저 DOM이 아니라 React가 관리하는 "설계도"

---

# 4️⃣ createElement가 왜 중요한가?

React에서 다음의 핵심 기능은 모두 이 구조로 인해 가능합니다.

### ✔️ Virtual DOM 비교(Reconciliation)

### ✔️ Fiber 아키텍처 스케줄링

### ✔️ React.memo, PureComponent 최적화

### ✔️ 불변성 기반 렌더링 트리 관리

### ✔️ Hook 기반 렌더링 라이프사이클

즉, createElement로 만든 **React Element가 있어야 React가 요소 트리를 이해하고 최적화할 수 있습니다.**

---

# 5️⃣ JSX를 사용하지 않을 때의 예

강의에서 종종 JSX 없이 React를 설명할 때 사용합니다:

```js
const App = () => {
  return React.createElement(
    'div',
    { className: 'app' },
    React.createElement('h1', null, 'Hello')
  );
};
```

JSX를 쓰면 동일한 코드가 다음처럼 단순해지는 것뿐입니다:

```jsx
const App = () => {
  return (
    <div className="app">
      <h1>Hello</h1>
    </div>
  );
};
```

---

# 6️⃣ JSX = React.createElement의 문법 설탕

### ✔ JSX는 코드의 가독성을 위한 문법

### ✔ React.createElement()는 내부 작동 로직

### ✔ 둘은 1:1 대응

---

# 7️⃣ createElement를 직접 쓰는 경우는?

보통 개발자들은 JSX를 사용하지만, 특정 상황에서 createElement를 직접 쓰기도 합니다:

### 🔹 React를 컴파일러 없이 사용하는 환경 (CDN + 브라우저)

### 🔹 JSX 변환이 불가능한 동적 컴포넌트 생성

### 🔹 React 내부 동작 원리를 설명하는 강의자료 제작 시

국비 교육에서 React의 동작 원리를 설명할 때 자주 등장합니다.

---

# 🧠 개념 정리 (강의 슬라이드용)

| 항목                  | 설명                                        |
| ------------------- | ----------------------------------------- |
| JSX                 | 개발자가 보기 좋은 문법                             |
| React.createElement | JSX를 변환해 실제 React 요소를 만드는 실체              |
| 반환값                 | React Element(순수 JS 객체, UI의 설계도)          |
| DOM 생성              | ReactDOM의 역할 (createElement는 DOM을 만들지 않음) |

---

# 📌 결론

👉 **React.createElement()는 React가 화면을 그릴 때 사용하는 “요소 생성 함수”이며 JSX의 실체입니다.**
👉 **JSX는 문법 설탕이고, createElement가 진짜 렌더링 데이터를 생성합니다.**
👉 **React Element는 DOM이 아니라 React가 관리하는 가상 DOM 구조입니다.**

