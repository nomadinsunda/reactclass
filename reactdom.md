React의 DOM 처리 방식은 **전통적인 직접 DOM 조작 방식과 완전히 다릅니다.**
React는 성능 최적화와 선언적 UI 구성을 위해 **Virtual DOM (가상 DOM)** 을 중심으로 작동하며,
이를 통해 실제 DOM 조작을 **최소화**하고 **효율적인 상태 기반 렌더링**을 가능하게 합니다.

다음은 React의 DOM 처리 방식을 전문가 수준으로 상세히 설명한 내용입니다.

---

## 🧠 1. 기본 철학: 선언형 UI + 상태 기반 DOM 렌더링

React는 다음 철학을 바탕으로 DOM을 처리합니다:

* **“DOM은 상태(state)의 함수이다.”**
* `UI = f(state)`로, 상태가 바뀌면 전체 UI를 다시 그리는 것처럼 동작
* 하지만 실제로는 모든 DOM을 새로 그리는 것이 아니라 **Virtual DOM 기반 diffing**으로 최소 변경만 수행

---

## 🧱 2. Virtual DOM: 가상 DOM 트리

### ✅ Virtual DOM이란?

* 브라우저의 실제 DOM을 **메모리 상에서 JavaScript 객체 형태로 추상화한 트리 구조**
* React 엘리먼트(`React.createElement` 또는 JSX)로 생성됨
* 실제 DOM과 동일한 구조를 가지되, **실제 브라우저 DOM이 아님**

### ✅ 예시:

```jsx
const element = <h1>Hello, world</h1>;
```

➡ React는 내부적으로 다음처럼 구성:

```js
const virtualDom = {
  type: 'h1',
  props: {
    children: 'Hello, world'
  }
};
```

---

## 🔁 3. Reconciliation (조정): Virtual DOM 비교 알고리즘

### 상태가 변경되면:

1. 새로운 Virtual DOM 생성
2. 이전 Virtual DOM과 **Diffing Algorithm**으로 비교
3. 차이를 계산
4. 최소한의 실제 DOM 조작 수행 (patching)

### 이 과정을 React는 **Reconciliation**이라고 부릅니다.

---

## ⚙️ 4. Diffing Algorithm: 효율적인 변경 감지

React는 다음과 같은 규칙을 이용해 diffing 속도를 높입니다:

* **노드 타입이 다르면** → 해당 노드를 통째로 교체
* **같은 타입이면** → 속성(props) 비교 후 필요한 속성만 변경
* **Key 기반 list 비교** → 효율적인 반복 요소 업데이트 지원

---

## 🛠️ 5. 실제 DOM 반영: React DOM Renderer

React는 플랫폼에 따라 다른 **Renderer**를 사용합니다:

| Renderer            | 설명                    |
| ------------------- | --------------------- |
| `react-dom`         | 브라우저용 실제 DOM 조작 수행    |
| `react-native`      | 네이티브 컴포넌트로 렌더링        |
| `react-three-fiber` | WebGL 기반 Three.js 엔진용 |
| `ink`               | 터미널 기반 CLI UI를 위한 렌더러 |

### 예: react-dom

```js
ReactDOM.render(<App />, document.getElementById('root'));
```

* `App` → Virtual DOM 생성
* Virtual DOM diff → 실제 DOM 변경

---

## 🚀 6. Batching & Fiber Architecture

### 🔹 Batching (일괄 처리)

* 여러 상태 변화가 동시에 일어나면 **한 번의 렌더링으로 묶어서 처리**
* 불필요한 리렌더링 방지

### 🔹 React Fiber

React 16 이후 도입된 **새로운 Reconciliation 엔진**

* Virtual DOM을 **단위 작업(Unit of Work)** 으로 나눔
* **우선순위 기반 작업 스케줄링** 가능 (High priority: 사용자 입력, Low: 로딩)
* **중단 가능한 렌더링**을 지원하여 UX 개선 (ex. 스크롤 중 멈추지 않음)

---

## 🧩 7. React DOM 처리 최적화 기법

| 기법                       | 설명                                 |
| ------------------------ | ---------------------------------- |
| `React.memo`             | 컴포넌트 props가 변하지 않으면 리렌더링 방지        |
| `useMemo`, `useCallback` | 함수나 값을 메모이제이션하여 불필요한 연산 제거         |
| Key 설정                   | 리스트 항목에 고유한 key를 설정해 diffing 효율 향상 |
| Lazy Loading             | 동적 import로 필요한 시점에만 컴포넌트 렌더링       |

---

## 📉 8. 전통적인 DOM 처리와의 비교

| 항목        | 전통 방식 (Vanilla JS / jQuery)                        | React                                  |
| --------- | -------------------------------------------------- | -------------------------------------- |
| DOM 조작    | 직접 DOM API (`document.createElement`, `innerHTML`) | 가상 DOM 기반으로 최소한의 변경만 실제 DOM에 반영        |
| 이벤트       | 이벤트 리스너 직접 바인딩                                     | Synthetic Event 시스템 (버블링 최적화)          |
| 상태 기반 렌더링 | 수작업으로 DOM 업데이트                                     | 상태 변경 → Virtual DOM diff → 실제 DOM 업데이트 |
| 성능        | 빈번한 DOM 접근은 느림                                     | 효율적인 diff + batching으로 고성능 유지          |

---

## ✅ 결론

> React는 **Virtual DOM + Reconciliation + Fiber 아키텍처**를 통해 DOM을 **선언적으로, 효율적으로, 비동기적으로 처리**합니다.

* 전통 DOM 방식은 **명령형(Imperative)**, React는 **선언형(Declarative)**
* 실제 DOM은 **결과물일 뿐**, 모든 UI 상태 변화는 Virtual DOM의 변화를 통해 간접적으로 처리됨

---

## 📌 추가로 설명 가능한 주제

* DOM 트리와 Shadow DOM의 비교
* React의 SyntheticEvent 시스템
* SSR에서의 DOM 처리 방식 (Next.js, hydration 등)
* React 18의 Concurrent Mode와 DOM 처리 영향

