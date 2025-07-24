
## ✅ 1. JSX란?

\*\*JSX (JavaScript XML)\*\*은 리액트에서 사용하는 **JavaScript 확장 문법**으로, XML/HTML과 비슷한 구조로 UI를 정의할 수 있게 해줍니다.

예시:

```jsx
const element = <h1>Hello, React!</h1>;
```

---

## ✅ 2. JSX는 브라우저가 이해할 수 없음

브라우저는 **JSX 문법을 직접 이해하지 못합니다.** 브라우저는 오직 **표준 JavaScript만 해석**할 수 있으므로, JSX는 반드시 \*\*JavaScript 코드로 변환(트랜스파일)\*\*되어야 합니다.

이때 사용하는 도구가 바로 **Babel**입니다.

---

## ✅ 3. JSX → JavaScript 코드로 변환되는 이유

JSX는 Babel을 통해 아래와 같이 변환됩니다:

```jsx
const element = <h1>Hello, React!</h1>;
```

⬇️ Babel에 의해 변환됨

```js
const element = React.createElement('h1', null, 'Hello, React!');
```

이처럼 JSX는 실제로 **`React.createElement()`** 함수 호출로 바뀝니다.

---

## ✅ 4. 이 변환이 Virtual DOM과 어떤 관련이 있나?

### 🔸 React.createElement()의 역할:

* `React.createElement()`는 \*\*JavaScript 객체 (Virtual DOM 노드)\*\*를 생성합니다.
* 이 객체는 **실제 브라우저 DOM이 아니라, 메모리 상의 Virtual DOM** 트리에 사용됩니다.

예:

```js
{
  type: 'h1',
  props: {
    children: 'Hello, React!'
  }
}
```

이런 객체가 리액트 내부에서 **Virtual DOM 트리를 구성**하며, 실제 DOM과의 **차이(diff)를 비교**한 후 최소한의 변경만 실제 DOM에 적용합니다 (reconciliation).

---

## ✅ 정리: JSX → JS 변환 이유와 Virtual DOM의 연관성

| 구분               | 설명                                                              |
| ---------------- | --------------------------------------------------------------- |
| JSX 변환 이유        | 브라우저는 JSX를 이해하지 못하기 때문에 표준 JS 코드로 변환해야 함                        |
| 변환 결과            | JSX는 `React.createElement()`로 바뀜                                |
| Virtual DOM과의 연관 | `React.createElement()`가 생성하는 객체가 바로 Virtual DOM 트리를 구성함        |
| 핵심 포인트           | JSX는 Virtual DOM을 만드는 데 사용되는 표현식이고, 실제로는 자바스크립트 함수 호출로 변환되어 사용됨 |

---

## ✅ 시각화 요약

```plaintext
JSX
 ↓  (Babel 트랜스파일)
React.createElement()
 ↓
Virtual DOM 객체
 ↓  (diffing 알고리즘)
실제 DOM 갱신
```

