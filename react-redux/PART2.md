# PART 2. Provider & Context

## React 컴포넌트는 Redux Store의 위치를 어떻게 아는가?

PART 1에서 React Redux의 핵심 구조를 다음과 같이 정리했습니다.

```text
React
  │
  ▼
React Redux
  │
  ▼
Redux Store
```

이번 PART에서는 다음 질문에 답합니다.

> **깊은 곳에 있는 React 컴포넌트는 Redux Store의 위치를 어떻게 아는가?**

애플리케이션 최상단에서는 Store를 다음과 같이 등록합니다.

```jsx
<Provider store={store}>
  <App />
</Provider>
```

하지만 실제로 Store를 사용하는 컴포넌트는 깊은 곳에 있을 수 있습니다.

```text
Provider
   │
   └── App
        │
        └── Main
             │
             └── Dashboard
                    │
                    └── Counter
```

`Counter`에 `store`를 props로 전달하지 않아도 다음 코드는 동작합니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  return <h1>{count}</h1>;
}
```

그 이유는 React Redux의 `<Provider>`가 **React Context를 통해 Store를 하위 컴포넌트에 제공하기 때문**입니다.

---

# 1. Props Drilling 문제

Redux Store를 props로 직접 전달하면 다음과 같은 구조가 됩니다.

```jsx
<App store={store} />
```

```jsx
function App({ store }) {
  return <Main store={store} />;
}

function Main({ store }) {
  return <Dashboard store={store} />;
}

function Dashboard({ store }) {
  return <Counter store={store} />;
}
```

```text
App
│ store
▼
Main
│ store
▼
Dashboard
│ store
▼
Counter
```

`App`, `Main`, `Dashboard`가 Store를 직접 사용하지 않더라도 `Counter`에 전달하기 위해 계속 props를 넘겨야 합니다. 이것이 Props Drilling입니다.

React Context는 이 문제를 해결합니다.

```text
Context Provider
       │
       │ value
       ▼
      App
       │
      Main
       │
   Dashboard
       │
    Counter
       │
       └── Context에서 value 읽기
```

중간 컴포넌트가 값을 직접 전달하지 않아도 하위 컴포넌트가 Context를 통해 필요한 값을 읽을 수 있습니다.

---

# 2. `<Provider>`의 역할

Redux Store는 먼저 `configureStore()`로 생성합니다.

```javascript
// store.js

import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

`<Provider>`는 이 Store를 새로 만드는 것이 아니라, 이미 생성된 Store를 React 컴포넌트 트리에 제공합니다.

```jsx
import { Provider } from 'react-redux';
import { store } from './store';

createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

역할을 구분하면 다음과 같습니다.

| 역할                       | 담당                  |
| ------------------------ | ------------------- |
| Redux Store 생성           | `configureStore()`  |
| Store를 React 트리에 제공      | `<Provider>`        |
| 하위 컴포넌트가 Store에 접근하도록 연결 | React Redux Context |

개념적으로는 다음과 같습니다.

```text
configureStore()
      │
      ▼
Redux Store
      │
      │ store={store}
      ▼
 <Provider>
      │
      │ React Redux Context
      ▼
Component Tree
```

`Provider`가 `App`에 `store` props를 자동으로 전달하는 것은 아닙니다.

```jsx
function App(props) {
  console.log(props.store); // 자동으로 전달되지 않음
}
```

Store는 props chain이 아니라 Context를 통해 접근됩니다.

---

# 3. `useSelector()`와 `useDispatch()`의 연결

React Redux의 Hook은 Provider가 제공한 Context를 통해 현재 Store에 접근합니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  const dispatch = useDispatch();

  const handleClick = () => {
    dispatch(increment());
  };

  return (
    <>
      <h1>{count}</h1>
      <button onClick={handleClick}>증가</button>
    </>
  );
}
```

두 Hook의 역할은 다릅니다.

* `useSelector()`는 Store의 State에서 필요한 값을 선택하고 구독합니다.
* `useDispatch()`는 Store의 `dispatch` 함수를 가져와 Action을 전달합니다.

전체 흐름은 다음과 같습니다.

```text
Redux Store
     │
     ▼
 <Provider>
     │
     │ React Redux Context
     ▼
Component
     │
     ├── useSelector()
     │       └── State 선택 및 구독
     │
     └── useDispatch()
             └── Action 전달
```

따라서 다음 코드는:

```javascript
const count = useSelector(
  state => state.counter.value
);

const dispatch = useDispatch();
```

개념적으로 다음을 의미합니다.

```text
useSelector()
  → 현재 Provider의 Store에서 State를 읽고 구독

useDispatch()
  → 현재 Provider의 Store의 dispatch를 사용
```

---

# 4. Context는 Redux State를 관리하지 않는다

React Redux가 Context를 사용한다고 해서 Context가 Redux State를 저장하거나 관리하는 것은 아닙니다.

```text
Redux Store
   ↓
Redux State 관리
```

Context의 역할은 React 컴포넌트와 Store를 연결하는 것입니다.

```text
Redux Store
     │
     │ 접근에 필요한 정보 제공
     ▼
React Redux Context
     │
     ▼
React Components
```

즉:

> **Redux State는 Redux Store가 관리하고, Context는 컴포넌트가 Store에 접근할 수 있도록 연결합니다.**

---

# 5. Provider의 범위

React Redux Hook을 사용하는 컴포넌트는 반드시 해당 Store를 제공하는 `<Provider>`의 하위에 있어야 합니다.

정상적인 구조:

```text
Provider
   │
   └── App
        │
        └── Counter
             └── useSelector()  ✅
```

Provider 밖에서는 Store를 찾을 수 없습니다.

```text
App
 │
 └── Counter
      └── useSelector()  ❌
```

따라서 보통 애플리케이션의 최상단에 Provider를 배치합니다.

```jsx
createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

다만 Provider 아래에 있다고 해서 모든 컴포넌트가 Redux State를 자동으로 읽거나 구독하는 것은 아닙니다.

```text
Provider
│
└── App
    ├── Header       → Redux 사용 안 함
    ├── Main         → Redux 사용 안 함
    │    └── Counter → useSelector()
    └── Footer       → Redux 사용 안 함
```

Provider는 Store에 접근할 수 있는 환경만 제공하며, 실제로 State를 선택하고 구독하는 것은 `useSelector()`를 사용하는 컴포넌트입니다.

---

# 6. 전체 흐름

React Redux의 연결 구조를 하나로 정리하면 다음과 같습니다.

```text
┌────────────────────────────────────────────┐
│                Redux                       │
│                                            │
│              Redux Store                   │
└──────────────────┬─────────────────────────┘
                   │
                   │ store={store}
                   ▼
┌────────────────────────────────────────────┐
│              React Redux                   │
│                                            │
│               <Provider>                   │
│                    │                       │
│                    ▼                       │
│          React Redux Context               │
└──────────────────┬─────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────┐
│                 React                      │
│                                            │
│                  App                       │
│                   │                        │
│                Counter                     │
│              /         \                   │
│     useSelector()   useDispatch()          │
│          │               │                 │
│          ▼               ▼                 │
│      State 읽기       Action 전달          │
└────────────────────────────────────────────┘
```

핵심 역할은 다음과 같습니다.

| 요소                  | 역할                  |
| ------------------- | ------------------- |
| `configureStore()`  | Redux Store 생성      |
| `<Provider>`        | Store를 React 트리에 제공 |
| React Redux Context | 컴포넌트와 Store 연결      |
| `useSelector()`     | 필요한 State 선택 및 구독   |
| `useDispatch()`     | Store에 Action 전달    |

가장 중요한 문장은 다음과 같습니다.

> **`<Provider>`는 이미 생성된 Redux Store를 React Redux Context를 통해 하위 컴포넌트에 제공하고, `useSelector()`와 `useDispatch()`는 이 연결을 통해 Store에 접근합니다.**

이제 남은 질문은 다음과 같습니다.

> **Store의 State가 변경되었을 때 `useSelector()`는 어떻게 이를 감지하고 컴포넌트를 다시 렌더링하는가?**

다음 PART에서는 Store Subscription을 중심으로 이 흐름을 살펴봅니다.

```text
Store Update
     ↓
Subscription
     ↓
Selector 재실행
     ↓
선택한 값 비교
     ↓
Component Re-render
```
