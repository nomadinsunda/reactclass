# PART 1. React Redux Fundamentals

## React와 Redux Store는 어떻게 연결되는가?

Redux를 학습했다면 이미 다음 구조에 익숙할 것입니다.

```text
Action
  ↓
dispatch(action)
  ↓
Reducer
  ↓
State 변경
  ↓
Redux Store
```

Redux는 애플리케이션의 **상태(State)를 중앙에서 관리하는 독립적인 상태 관리 라이브러리**입니다.

그런데 React 애플리케이션에서 Redux를 사용하려고 하면 한 가지 중요한 문제가 생깁니다.

> **React 컴포넌트는 Redux Store와 어떻게 연결되는가?**

Redux Store가 존재한다고 해서 React 컴포넌트가 자동으로 Store의 State를 읽거나, Store의 변경을 감지하거나, Action을 dispatch할 수 있는 것은 아닙니다.

이 문제를 해결하는 라이브러리가 바로 **React Redux**입니다.

---

# 1. React Redux란?

React Redux는 한 문장으로 정의하면 다음과 같습니다.

> **React Redux는 React 컴포넌트와 Redux Store를 연결해 주는 공식 React 바인딩 라이브러리입니다.**

전체 관계를 단순화하면 다음과 같습니다.

```text
┌───────────────────────┐
│        React          │
│                       │
│   React Components    │
└───────────┬───────────┘
            │
            │ React Redux
            │
┌───────────▼───────────┐
│        Redux          │
│                       │
│     Redux Store       │
└───────────────────────┘
```

여기서 중요한 것은 **Redux와 React Redux는 서로 다른 라이브러리**라는 점입니다.

```text
Redux
→ 상태를 관리한다.

React Redux
→ React가 Redux Store를 사용할 수 있도록 연결한다.
```

즉,

> **Redux가 "상태 관리 시스템"이라면, React Redux는 "React와 Redux 사이의 연결 계층"입니다.**

---

# 2. Redux와 React Redux는 다르다

처음 Redux를 배우는 학생들이 가장 많이 혼동하는 부분입니다.

Redux 자체는 React 전용 라이브러리가 아닙니다.

Redux는 React가 없어도 사용할 수 있습니다.

예를 들어 Redux Store가 있다고 해보겠습니다.

```javascript
const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});

console.log(store.getState());

store.dispatch(increment());
```

이 코드에는 React가 전혀 없습니다.

Redux Store는 독립적으로 다음 작업을 수행할 수 있습니다.

```text
getState()
dispatch()
subscribe()
```

따라서 구조적으로 보면 다음과 같습니다.

```text
Redux
│
├── Store
├── State
├── Action
├── Reducer
└── Dispatch
```

반면 React Redux는 React 애플리케이션이 이 Redux Store를 편리하고 안전하게 사용할 수 있도록 연결합니다.

```text
React Redux
│
├── <Provider>
├── useSelector()
└── useDispatch()
```

대표적인 역할을 비교하면 다음과 같습니다.

| Redux      | React Redux             |
| ---------- | ----------------------- |
| State 관리   | React와 Redux 연결         |
| Store 생성   | Store를 React 트리에 제공     |
| Reducer 실행 | 컴포넌트에서 State 사용         |
| Action 처리  | 컴포넌트에서 dispatch         |
| State 변경   | State 변경을 React 렌더링과 연결 |

따라서 다음 두 패키지도 구분해야 합니다.

```bash
npm install @reduxjs/toolkit react-redux
```

`@reduxjs/toolkit`은 현대적인 Redux 로직을 작성하기 위한 도구이고,

`react-redux`는 그 Redux Store를 **React에서 사용하기 위한 연결 라이브러리**입니다.

---

# 3. Redux Store는 React 바깥에 존재한다

React Redux를 제대로 이해하려면 가장 먼저 이 사실을 이해해야 합니다.

> **Redux Store는 React 컴포넌트가 아닙니다.**

예를 들어 다음과 같이 Store를 만들었다고 해보겠습니다.

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

생성된 `store`는 단순한 JavaScript 객체입니다.

개념적으로 보면 다음과 같습니다.

```text
JavaScript Application
│
├── Redux Store
│
│    ├── getState()
│    ├── dispatch()
│    └── subscribe()
│
└── React
     │
     └── Component Tree
```

즉, Redux Store와 React Component Tree는 처음부터 하나로 연결되어 있는 것이 아닙니다.

```text
Redux Store                    React Component Tree

┌──────────────┐               ┌──────────────┐
│    Store     │               │     App      │
│              │               │      │       │
│    State     │               │    Main      │
│              │               │      │       │
│   dispatch   │               │   Counter    │
└──────────────┘               └──────────────┘

       서로 자동으로 연결되어 있지 않음
```

바로 이 지점에서 React Redux가 필요합니다.

---

# 4. React 컴포넌트에서 Redux를 직접 사용하면 안 되는가?

기술적으로는 가능합니다.

Redux Store는 일반 JavaScript 객체이므로 직접 import할 수 있습니다.

```jsx
import { store } from './store';

function Counter() {
  const state = store.getState();

  return <h1>{state.counter.value}</h1>;
}
```

처음 보면 아무 문제가 없어 보입니다.

Store의 State도 가져왔습니다.

그런데 중요한 문제가 있습니다.

---

## 4.1 `getState()`는 현재 값을 읽을 뿐이다

```javascript
store.getState();
```

`getState()`는 호출한 순간의 State를 반환합니다.

하지만 React에게 다음 사실을 알려주지는 않습니다.

> "Redux State가 변경되었으니 이 컴포넌트를 다시 렌더링하세요."

예를 들어:

```text
Counter Component
      │
      │ store.getState()
      ▼
Redux Store
      │
      └── count = 0
```

이후 다른 곳에서:

```javascript
store.dispatch(increment());
```

하여 State가 변경되었다고 해보겠습니다.

```text
count = 0

   ↓ dispatch

count = 1
```

Redux Store의 값은 변경되었습니다.

하지만 단순히 `getState()`를 호출했던 React 컴포넌트가 **자동으로 다시 렌더링되는 것은 아닙니다.**

React와 Redux의 변경 감지가 연결되어 있지 않기 때문입니다.

---

# 5. 그렇다면 `subscribe()`를 사용하면 되지 않을까?

Redux에는 Store의 변경을 감지할 수 있는 `subscribe()`가 있습니다.

```javascript
store.subscribe(() => {
  console.log(store.getState());
});
```

따라서 이론적으로 React 컴포넌트에서 직접 Store를 구독하는 코드를 만들 수도 있습니다.

개념적으로는 다음과 같습니다.

```text
React Component
      │
      ├── store.getState()
      │
      ├── store.subscribe()
      │
      └── store.dispatch()
              │
              ▼
          Redux Store
```

하지만 이렇게 하면 각 React 컴포넌트가 직접 다음 작업을 처리해야 합니다.

```text
1. Store를 가져온다.

2. Store를 구독한다.

3. State를 읽는다.

4. 필요한 State를 선택한다.

5. State가 변경되었는지 판단한다.

6. 필요한 경우 React를 다시 렌더링한다.

7. 컴포넌트가 사라질 때 구독을 해제한다.
```

React 애플리케이션의 컴포넌트가 수십 개, 수백 개가 되면 이런 코드를 직접 관리하는 것은 현실적이지 않습니다.

그리고 React의 렌더링 시스템과 외부 Store의 변경을 안전하게 연결하는 문제도 생각보다 단순하지 않습니다.

---

# 6. React Redux가 이 연결을 담당한다

그래서 React Redux가 등장합니다.

React Redux는 React 컴포넌트가 Redux Store를 사용할 때 필요한 연결 작업을 대신 처리합니다.

```text
React Component
       │
       ▼
┌────────────────────────┐
│      React Redux       │
│                        │
│  Store 접근            │
│  Store 구독            │
│  State 선택            │
│  dispatch 제공         │
│  Re-render 연결        │
└───────────┬────────────┘
            │
            ▼
       Redux Store
```

즉 React 컴포넌트가 직접

```javascript
store.getState();
store.subscribe(...);
store.dispatch(...);
```

를 관리하는 대신 React Redux가 React에 적합한 방식으로 이 작업들을 연결합니다.

---

# 7. React Redux의 핵심 역할

React Redux의 역할은 크게 세 가지로 생각하면 쉽습니다.

## 7.1 React 애플리케이션에 Store를 제공한다

대표적인 것이 `<Provider>`입니다.

```jsx
<Provider store={store}>
  <App />
</Provider>
```

개념적으로:

```text
Redux Store
     │
     ▼
 <Provider>
     │
     ▼
    App
     │
 ┌───┴────┐
 │        │
Header   Main
          │
       Counter
```

이렇게 하면 하위 React 컴포넌트들이 React Redux를 통해 Store에 접근할 수 있습니다.

`<Provider>`가 실제로 Store를 어떻게 전달하는지는 **PART 2**에서 자세히 살펴봅니다.

---

## 7.2 Redux State를 React 컴포넌트에서 사용할 수 있게 한다

React Redux는 `useSelector()`를 제공합니다.

```jsx
const count = useSelector(
  state => state.counter.value
);
```

개념적으로:

```text
Redux Store
     │
     │ State
     ▼
 React Redux
     │
     │ useSelector()
     ▼
React Component
```

즉,

> **`useSelector()`는 Redux Store의 State 중 컴포넌트가 필요한 값을 선택해서 사용할 수 있게 해줍니다.**

하지만 `useSelector()`의 중요한 역할은 단순히 값을 읽는 것에서 끝나지 않습니다.

Store의 변경과 React 컴포넌트의 **re-render를 연결하는 역할**도 합니다.

이 부분은 PART 3에서 자세히 다룹니다.

---

## 7.3 React 컴포넌트에서 Action을 dispatch할 수 있게 한다

반대 방향도 필요합니다.

사용자가 버튼을 클릭했다고 해보겠습니다.

```jsx
<button onClick={handleClick}>
  증가
</button>
```

React 컴포넌트는 Redux Store에 Action을 전달해야 합니다.

React Redux에서는 `useDispatch()`를 사용합니다.

```jsx
const dispatch = useDispatch();

dispatch(increment());
```

흐름은 다음과 같습니다.

```text
React Component
      │
      │ useDispatch()
      ▼
   dispatch
      │
      │ action
      ▼
 Redux Store
```

따라서 `useSelector()`와 `useDispatch()`는 방향을 기준으로 이해하면 매우 쉽습니다.

```text
Redux → React
useSelector()

React → Redux
useDispatch()
```

---

# 8. React Redux의 전체 역할을 한 번에 보기

지금까지의 내용을 하나로 연결하면 다음과 같습니다.

```text
┌──────────────────────────────────────────┐
│              React                      │
│                                          │
│          React Component                 │
│             │       ▲                    │
│             │       │                    │
│   useDispatch()   useSelector()          │
│             │       │                    │
└─────────────│───────│────────────────────┘
              │       │
              ▼       │
┌──────────────────────────────────────────┐
│            React Redux                   │
│                                          │
│   React와 Redux Store 사이의 연결 계층   │
│                                          │
│   • Store 제공                           │
│   • Store 접근                           │
│   • Store 구독                           │
│   • State 선택                           │
│   • dispatch 제공                        │
│   • Re-render 연결                       │
└─────────────│────────────────────────────┘
              │
              ▼
┌──────────────────────────────────────────┐
│               Redux                     │
│                                          │
│             Redux Store                  │
│                                          │
│        State / dispatch / subscribe      │
└──────────────────────────────────────────┘
```

이 구조가 **React Redux 전체 강의의 출발점**입니다.

---

# 9. React Redux에서 가장 중요한 세 가지

React Redux를 처음 배울 때는 우선 다음 세 가지를 기억하면 됩니다.

```text
React Redux
│
├── <Provider>
│      │
│      └── Store를 React 애플리케이션에 제공
│
├── useSelector()
│      │
│      └── Store → Component
│
└── useDispatch()
       │
       └── Component → Store
```

조금 더 직관적으로 표현하면:

```text
                  Redux Store
                 ▲           │
                 │           │
          Action │           │ State
                 │           ▼
          useDispatch()   useSelector()
                 ▲           │
                 │           ▼
               React Component
```

그리고 이 모든 것이 동작할 수 있도록 가장 바깥에서 Store와 React를 연결하는 것이 `<Provider>`입니다.

---

# 10. React Redux를 단순한 Hook 모음으로 보면 안 된다

React Redux를 처음 배우면 다음 두 Hook에만 관심을 가지기 쉽습니다.

```javascript
useSelector()
useDispatch()
```

그러나 React Redux의 본질은 Hook 자체가 아닙니다.

더 중요한 것은:

> **외부에 존재하는 Redux Store와 React의 렌더링 시스템을 연결하는 것**

입니다.

Redux Store는 React와 독립적으로 존재합니다.

```text
Redux Store
```

React는 자신만의 컴포넌트 트리와 렌더링 시스템을 가지고 있습니다.

```text
React Component Tree
```

React Redux가 이 둘 사이에 위치합니다.

```text
React
  │
  ▼
React Redux
  │
  ▼
Redux
```

따라서 React Redux를 공부할 때는 단순히

```text
useSelector 사용법
useDispatch 사용법
```

만 외우는 것이 아니라,

```text
Store는 어디에 존재하는가?

        ↓

React는 Store의 존재를 어떻게 아는가?

        ↓

컴포넌트는 Store에 어떻게 접근하는가?

        ↓

컴포넌트는 State를 어떻게 읽는가?

        ↓

State가 변경된 것을 어떻게 감지하는가?

        ↓

그 변경이 어떻게 React의 re-render로 연결되는가?
```

라는 흐름으로 이해해야 합니다.

---

# 11. 앞으로 배울 내용

이제 React Redux가 왜 필요한지는 알았습니다.

하지만 아직 중요한 질문이 남아 있습니다.

예를 들어 다음 코드가 있다고 해보겠습니다.

```jsx
import { Provider } from 'react-redux';
import { store } from './store';

<Provider store={store}>
  <App />
</Provider>
```

그리고 아주 깊숙한 곳에 다음 컴포넌트가 있다고 해보겠습니다.

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

`Counter`는 `store`를 직접 전달받지 않았습니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  // ...
}
```

그런데도 `useSelector()`는 Redux Store에 접근할 수 있습니다.

그렇다면 가장 중요한 질문이 생깁니다.

> **Counter 컴포넌트는 Redux Store가 어디에 있는지 어떻게 알고 있을까요?**

그 비밀은 React Redux의 **`<Provider>`와 React의 Context API**에 있습니다.

다음 PART에서는 바로 이 연결 구조를 내부 원리까지 살펴봅니다.

```text
PART 2

Redux Store
     │
     ▼
 <Provider>
     │
     ▼
React Context
     │
     ▼
Component Tree
     │
     ▼
useSelector / useDispatch
```

---

# 핵심 정리

React Redux를 한 문장으로 다시 정의하면:

> **React Redux는 독립적으로 존재하는 Redux Store와 React 컴포넌트 트리를 연결하여, 컴포넌트가 Store의 State를 구독하고 Action을 dispatch할 수 있도록 해주는 공식 React 바인딩 라이브러리입니다.**

그리고 가장 중요한 구조는 다음과 같습니다.

```text
                 React
                   │
                   ▼
             React Redux
          ┌────────┼────────┐
          │        │        │
      Provider  useSelector useDispatch
          │        │        │
          └────────┼────────┘
                   │
                   ▼
              Redux Store
```

다음 PART에서 이 중 가장 먼저 동작하는 **`<Provider>`가 Redux Store를 React 컴포넌트 트리에 어떻게 제공하는지**, 그리고 그 과정에서 **React Context가 어떤 역할을 하는지** 자세히 살펴보겠습니다.
