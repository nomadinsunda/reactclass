# PART 3. Redux Toolkit

PART 2에서는 Redux의 핵심 데이터 흐름을 학습했습니다.

```text
Event
  ↓
Action
  ↓
dispatch()
  ↓
Store
  ↓
Reducer
  ↓
New State
  ↓
Store
  ↓
React
```

Redux의 핵심 원리는 비교적 단순합니다.

하지만 실제 애플리케이션에서 이 구조를 직접 구현하려고 하면 다음과 같은 질문이 생깁니다.

> **Action Type, Action Creator, Reducer, Store를 실제 React 프로젝트에서는 어떻게 구성해야 할까?**

현대 Redux 애플리케이션에서는 일반적으로 **Redux Toolkit(RTK)**을 사용합니다.

Redux Toolkit은 Redux의 원리를 다른 것으로 바꾸는 라이브러리가 아닙니다.

> **Redux Toolkit은 기존 Redux의 구조를 유지하면서 반복적인 코드를 줄이고, 안전한 기본 설정을 제공하는 Redux 공식 도구 모음입니다.**

이번 PART에서는 다음 흐름을 완성하는 것이 목표입니다.

```text
createSlice()
     ↓
Slice Reducer + Action Creator
     ↓
configureStore()
     ↓
Redux Store
     ↓
Provider
     ↓
React Component
   ↙          ↘
useSelector   useDispatch
```

---

# 1. 왜 Redux Toolkit이 필요한가?

먼저 Vanilla Redux로 간단한 Counter를 구현한다고 생각해봅시다.

```javascript
const initialState = {
    value: 0
};

const INCREMENT = "counter/increment";
const DECREMENT = "counter/decrement";

function increment() {
    return {
        type: INCREMENT
    };
}

function decrement() {
    return {
        type: DECREMENT
    };
}

function counterReducer(state = initialState, action) {
    switch (action.type) {
        case INCREMENT:
            return {
                ...state,
                value: state.value + 1
            };

        case DECREMENT:
            return {
                ...state,
                value: state.value - 1
            };

        default:
            return state;
    }
}
```

Counter 하나를 만들었을 뿐인데 상당한 코드가 필요합니다.

```text
Action Type 선언
      ↓
Action Creator 작성
      ↓
Reducer 작성
      ↓
switch 작성
      ↓
불변성 처리
```

애플리케이션의 기능이 증가하면 이러한 코드도 계속 증가합니다.

이처럼 실제 기능보다 반복적으로 작성해야 하는 구조적인 코드를 흔히 **Boilerplate Code**라고 합니다.

Redux Toolkit은 이러한 반복을 줄이기 위해 등장했습니다.

---

# 2. Redux Toolkit이란?

Redux Toolkit은 Redux 애플리케이션을 작성하기 위한 공식 도구 모음입니다.

대표적인 API에는 다음과 같은 것들이 있습니다.

```text
Redux Toolkit
│
├── configureStore()
├── createSlice()
├── createReducer()
├── createAction()
├── createAsyncThunk()
└── createApi()        ← RTK Query
```

이번 PART에서는 먼저 **동기적인 Redux 상태 관리**에 집중합니다.

핵심적으로 사용할 Redux Toolkit API는 다음 두 가지입니다.

```text
createSlice()
configureStore()
```

그리고 React와 Redux를 연결하기 위해 `react-redux`의 다음 기능을 사용합니다.

```text
Provider
useSelector()
useDispatch()
```

전체 관계를 먼저 보면 다음과 같습니다.

```text
              Redux Toolkit
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
   createSlice()        configureStore()
        │                     │
        │                 Redux Store
        │                     │
        └──────────┬──────────┘
                   │
                   ↓
              React-Redux
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    Provider   useSelector useDispatch
```

---

# 3. `@reduxjs/toolkit`과 `react-redux`는 다르다

React에서 Redux Toolkit을 사용할 때 보통 두 패키지를 함께 설치합니다.

```bash
npm install @reduxjs/toolkit react-redux
```

두 패키지는 역할이 다릅니다.

## `@reduxjs/toolkit`

Redux 자체를 구성하고 관리하기 위한 도구를 제공합니다.

```text
@reduxjs/toolkit
│
├── createSlice()
├── configureStore()
├── createAsyncThunk()
└── RTK Query
```

즉,

> **Redux를 구성하는 도구**

입니다.

## `react-redux`

React와 Redux Store를 연결합니다.

```text
react-redux
│
├── Provider
├── useSelector()
└── useDispatch()
```

즉,

> **React Component가 Redux Store를 사용할 수 있도록 연결하는 라이브러리**

입니다.

따라서 다음처럼 구분하면 됩니다.

```text
@reduxjs/toolkit
        ↓
Redux 구성 및 상태 관리


react-redux
        ↓
React ↔ Redux 연결
```

이 차이는 반드시 이해해야 합니다.

---

# 4. Slice란?

Redux 애플리케이션의 전체 State가 다음과 같다고 생각해봅시다.

```javascript
{
    counter: {
        value: 0
    },

    user: {
        name: "Alice",
        loggedIn: true
    },

    cart: {
        items: []
    }
}
```

전체 Redux State를 기능 또는 도메인 단위로 나누어 볼 수 있습니다.

```text
Redux State
│
├── counter
│    └── value
│
├── user
│    ├── name
│    └── loggedIn
│
└── cart
     └── items
```

Redux Toolkit에서는 특정 기능에 관련된 상태와 로직을 하나의 단위로 묶어 관리할 수 있습니다.

예를 들어 Counter 기능에는 다음이 필요합니다.

```text
Counter
│
├── State
├── Reducer Logic
├── Action Creator
└── Action Type
```

Redux Toolkit에서는 이렇게 **특정 기능에 관련된 Redux 로직을 하나의 단위로 구성한 것**을 **Slice**라고 합니다.

```text
Counter Slice
│
├── State
│    └── value
│
├── Reducers
│    ├── increment
│    └── decrement
│
├── Action Creators
│    ├── increment()
│    └── decrement()
│
└── Action Types
     ├── counter/increment
     └── counter/decrement
```

---

# 5. `createSlice()`

Slice를 만드는 대표적인 API가 `createSlice()`입니다.

```javascript
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
    name: "counter",

    initialState: {
        value: 0
    },

    reducers: {
        increment(state) {
            state.value++;
        },

        decrement(state) {
            state.value--;
        }
    }
});
```

핵심 구조는 세 부분입니다.

```text
createSlice({
│
├── name
├── initialState
└── reducers
})
```

각각의 의미를 살펴보겠습니다.

---

# 6. `name`

```javascript
name: "counter"
```

`name`은 Slice의 이름입니다.

특히 `createSlice()`가 자동으로 생성하는 **Action Type의 prefix**로 사용됩니다.

예를 들어:

```javascript
reducers: {
    increment(state) {
        state.value++;
    }
}
```

라고 정의하면 Redux Toolkit은 개념적으로 다음 Action Type을 생성합니다.

```text
name
 ↓
counter

reducer name
 ↓
increment

        ↓

counter/increment
```

따라서 `increment()` Action Creator를 호출하면 다음과 같은 Action이 만들어집니다.

```javascript
{
    type: "counter/increment"
}
```

### 중요한 점

`createSlice()`의 `name`이 Redux Store State의 key를 직접 결정하는 것은 아닙니다.

```javascript
const counterSlice = createSlice({
    name: "counter",
    // ...
});
```

그리고 Store를 다음처럼 구성할 수도 있습니다.

```javascript
const store = configureStore({
    reducer: {
        myCounter: counterSlice.reducer
    }
});
```

이 경우 State는:

```javascript
{
    myCounter: {
        value: 0
    }
}
```

가 됩니다.

즉,

```text
createSlice name
"counter"
    ↓
Action Type Prefix
counter/increment


configureStore reducer key
myCounter
    ↓
State Address
state.myCounter
```

따라서:

> **Slice의 `name`과 Store State의 key는 서로 다른 역할을 한다.**

실제 프로젝트에서는 이해하기 쉽도록 같은 이름을 사용하는 경우가 많을 뿐입니다.

---

# 7. `initialState`

```javascript
initialState: {
    value: 0
}
```

`initialState`는 해당 Slice가 처음 사용할 State입니다.

Counter Slice의 초기 State는:

```javascript
{
    value: 0
}
```

입니다.

이 Reducer를 Store에 다음처럼 등록하면:

```javascript
reducer: {
    counter: counterReducer
}
```

전체 Redux State는:

```javascript
{
    counter: {
        value: 0
    }
}
```

가 됩니다.

---

# 8. `reducers`

```javascript
reducers: {
    increment(state) {
        state.value++;
    },

    decrement(state) {
        state.value--;
    }
}
```

`reducers`에는 해당 Slice의 State를 변경하기 위한 Reducer 로직을 정의합니다.

Redux의 기본 원리는 그대로입니다.

```text
Current State + Action
          ↓
       Reducer
          ↓
      New State
```

차이점은 `createSlice()`가 이 Reducer 정의를 바탕으로 **Reducer와 Action Creator를 함께 구성해준다**는 것입니다.

---

# 9. `createSlice()`가 자동으로 만들어주는 것

다음 코드를 보겠습니다.

```javascript
const counterSlice = createSlice({
    name: "counter",

    initialState: {
        value: 0
    },

    reducers: {
        increment(state) {
            state.value++;
        }
    }
});
```

Redux Toolkit은 이 정의를 바탕으로 다음 요소들을 생성합니다.

```text
               createSlice()
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Reducer   Actions   Action Types
                    │
                    ↓
              Action Creators
```

따라서:

```javascript
counterSlice.reducer
```

를 통해 Slice Reducer를 얻을 수 있고,

```javascript
counterSlice.actions
```

를 통해 자동 생성된 Action Creator를 얻을 수 있습니다.

---

# 10. `counterSlice.actions`

다음처럼 Action Creator를 꺼낼 수 있습니다.

```javascript
export const {
    increment,
    decrement
} = counterSlice.actions;
```

이제:

```javascript
increment()
```

를 호출하면 Action 객체가 생성됩니다.

```javascript
{
    type: "counter/increment"
}
```

즉 Vanilla Redux에서 직접 작성했던:

```javascript
function increment() {
    return {
        type: "counter/increment"
    };
}
```

를 Redux Toolkit이 자동으로 만들어준 것입니다.

---

# 11. `payload`가 필요한 Action

Action을 통해 값을 전달해야 하는 경우도 있습니다.

```javascript
reducers: {
    incrementByAmount(state, action) {
        state.value += action.payload;
    }
}
```

사용할 때는:

```javascript
dispatch(incrementByAmount(10));
```

이라고 작성합니다.

먼저:

```javascript
incrementByAmount(10)
```

이 실행되면서 개념적으로 다음 Action이 생성됩니다.

```javascript
{
    type: "counter/incrementByAmount",
    payload: 10
}
```

그리고 Reducer는:

```javascript
action.payload
```

를 통해 전달된 `10`을 사용합니다.

전체 흐름은:

```text
incrementByAmount(10)
        ↓
Action Creator
        ↓
{
    type: "counter/incrementByAmount",
    payload: 10
}
        ↓
dispatch()
        ↓
Reducer
        ↓
state.value += action.payload
```

입니다.

---

# 12. `state.value++`는 State 직접 수정 아닌가?

여기서 중요한 의문이 생깁니다.

Redux Reducer는 기존 State를 직접 변경하면 안 됩니다.

그런데 `createSlice()`에서는 다음처럼 작성합니다.

```javascript
increment(state) {
    state.value++;
}
```

겉으로 보면 기존 State를 직접 수정하는 것처럼 보입니다.

하지만 Redux Toolkit에서는 이것이 가능합니다.

그 이유가 **Immer**입니다.

---

# 13. Immer

Redux Toolkit은 내부적으로 Immer를 사용하여 Reducer의 불변성 처리를 쉽게 해줍니다.

개발자는 다음처럼 mutation 형태의 코드를 작성할 수 있습니다.

```javascript
state.value++;
```

하지만 실제 기존 State 객체를 그대로 수정하는 것이 아니라 Immer가 변경 내용을 추적하여 **새로운 immutable State**를 만들어냅니다.

개념적으로:

```text
개발자가 작성

state.value++
     ↓
  Immer
     ↓
새로운 불변 State 생성
```

예를 들어:

```text
Old State
{
    value: 0
}
    ↓

state.value++

    ↓
  Immer
    ↓

New State
{
    value: 1
}
```

따라서 `createSlice()`의 Reducer 안에서는 다음과 같은 코드를 자연스럽게 사용할 수 있습니다.

```javascript
state.items.push(product);
state.user.name = "Alice";
state.value++;
```

하지만 이를:

> Redux에서는 State를 직접 수정해도 된다.

라고 이해하면 안 됩니다.

정확하게는:

> **Redux Toolkit의 Immer 기반 Reducer에서는 mutation처럼 코드를 작성할 수 있으며, Immer가 이를 안전한 immutable update 결과로 변환한다.**

라고 이해해야 합니다.

---

# 14. `configureStore()`

Slice를 만들었다면 이제 Redux Store를 구성해야 합니다.

Redux Toolkit에서는 `configureStore()`를 사용합니다.

```javascript
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

const store = configureStore({
    reducer: {
        counter: counterReducer
    }
});

export default store;
```

전체 관계는:

```text
counterSlice
     ↓
counterSlice.reducer
     ↓
counterReducer
     ↓
configureStore()
     ↓
Redux Store
```

입니다.

여러 Slice도 하나의 Store에 등록할 수 있습니다.

```javascript
const store = configureStore({
    reducer: {
        counter: counterReducer,
        user: userReducer,
        cart: cartReducer
    }
});
```

그러면 Redux State는:

```javascript
{
    counter: { ... },
    user: { ... },
    cart: { ... }
}
```

형태가 됩니다.

즉 `reducer` 객체에 등록한 **key가 Redux State의 구조를 결정합니다.**

---

# 15. `configureStore()`가 하는 일

`configureStore()`는 단순히 Reducer를 합치는 함수가 아닙니다.

Redux Store를 실무에 적합한 기본 설정과 함께 구성합니다.

```text
configureStore()
│
├── Reducer 구성
├── Store 생성
├── 기본 Middleware 구성
├── Redux DevTools 지원
└── 개발 단계 안전성 검사
```

따라서 기존 Redux의 저수준 Store 설정보다 훨씬 간단하게 Redux Store를 구성할 수 있습니다.

여기까지 진행하면 Redux Store 자체는 완성되었습니다.

```text
createSlice()
     ↓
Reducer
     ↓
configureStore()
     ↓
Redux Store
```

하지만 아직 한 가지 중요한 문제가 남아 있습니다.

> **React Component는 이 Store가 어디에 있는지 어떻게 알 수 있을까요?**

여기서 `react-redux`가 등장합니다.

---

# 16. Redux Store와 React 연결

현재 상태를 구조적으로 보면 다음과 같습니다.

```text
Redux Store


React Application
```

Redux Store와 React Component Tree는 별개의 구조입니다.

따라서 둘 사이를 연결해야 합니다.

```text
Redux Store
     ↕
React Application
```

이 역할을 `react-redux`가 담당합니다.

핵심 도구는 다음 세 가지입니다.

```text
Provider
useSelector()
useDispatch()
```

---

# 17. `Provider`

React 애플리케이션이 Redux Store를 사용할 수 있도록 앱 최상단에서 `Provider`로 감쌉니다.

Vite + React 프로젝트라면 일반적으로 `main.jsx`에서 설정합니다.

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { Provider } from "react-redux";

import App from "./App.jsx";
import store from "./app/store.js";

createRoot(document.getElementById("root")).render(
    <StrictMode>
        <Provider store={store}>
            <App />
        </Provider>
    </StrictMode>
);
```

핵심은 다음 코드입니다.

```jsx
<Provider store={store}>
    <App />
</Provider>
```

구조적으로 보면:

```text
Provider
│
│ store={store}
│
└── App
     │
     ├── Header
     ├── Main
     └── Footer
```

`Provider` 아래에 위치한 컴포넌트들은 React-Redux API를 통해 Redux Store를 사용할 수 있게 됩니다.

그런데 여기서 한 단계 더 들어가야 합니다.

> **Provider 아래에 있다는 이유만으로 자식 컴포넌트가 어떻게 Store를 찾을 수 있을까요?**

그 핵심이 **React Context API**입니다.

---

# 18. `Provider`와 React Context API

컴포넌트가 Redux Store의 위치를 알 수 있는 이유는 앱 최상단의 `Provider`와 React의 **Context API** 덕분입니다.

`Provider`는 전달받은 Store를 React Context를 이용하여 하위 Component Tree에서 접근할 수 있도록 제공합니다.

```jsx
<Provider store={store}>
    <App />
</Provider>
```

개념적으로는 다음과 같은 구조입니다.

```text
                 Redux Store
                      │
                      │ store={store}
                      ▼
              ┌──────────────┐
              │   Provider   │
              └──────┬───────┘
                     │
               React Context
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Header        Main        Footer
                     │
                     ▼
                  Counter
```

즉 Store를 각 컴포넌트가 직접 import하는 것이 아닙니다.

```javascript
// 이런 방식으로 사용하는 것이 핵심 구조가 아니다.
import store from "./store";
```

대신 앱 최상단에서 한 번:

```jsx
<Provider store={store}>
    <App />
</Provider>
```

로 Store를 제공하고, 하위 컴포넌트에서는:

```javascript
useSelector()
useDispatch()
```

를 사용합니다.

전체 관계는 다음과 같습니다.

```text
Redux Store
    │
    ▼
Provider
    │
    ▼
React Context
    │
    ├───────────────┐
    ▼               ▼
useSelector()   useDispatch()
    │               │
    ▼               ▼
State 읽기      dispatch 사용
```

따라서 `Provider`는 단순히 `<App />`을 감싸는 장식용 Component가 아닙니다.

> **Provider는 Redux Store를 React Component Tree에 연결하는 진입점입니다.**

그리고 `useSelector()`와 `useDispatch()`는 이 연결을 이용하여 현재 Provider가 제공하는 Store와 상호작용합니다.

---

# 19. `useSelector()`

`useSelector()`는 Redux Store의 State에서 컴포넌트가 필요한 값을 선택하여 읽을 때 사용합니다.

```jsx
import { useSelector } from "react-redux";

function Counter() {
    const count = useSelector(
        state => state.counter.value
    );

    return <h1>{count}</h1>;
}
```

Redux State가:

```javascript
{
    counter: {
        value: 10
    }
}
```

이라면:

```javascript
state => state.counter.value
```

의 결과는 `10`입니다.

이처럼 State에서 필요한 값을 선택하는 함수를 **Selector**라고 합니다.

```text
Redux State
{
    counter: {
        value: 10
    },

    user: {
        name: "Alice"
    }
}
        │
        │ Selector
        ▼
state => state.counter.value
        │
        ▼
       10
```

따라서:

```javascript
const count = useSelector(
    state => state.counter.value
);
```

는:

> **Redux State에서 `counter.value`를 선택하여 현재 React Component에서 사용한다.**

는 의미입니다.

---

# 20. `useSelector()`와 Re-render

`useSelector()`는 State를 한 번 읽고 끝나는 함수가 아닙니다.

React-Redux는 Store의 변경과 컴포넌트를 연결하고, Selector가 선택한 결과가 변경되면 해당 컴포넌트가 새로운 값을 사용하도록 다시 렌더링합니다.

```text
Redux Store
value = 0
    │
    ▼
useSelector()
    │
    ▼
Counter
    │
    ▼
0 표시


dispatch(increment())
    │
    ▼
Redux Store
value = 1
    │
    ▼
Selector 결과 변경
    │
    ▼
Counter Re-render
    │
    ▼
1 표시
```

이 연결 덕분에 Redux State와 React UI가 동기화됩니다.

---

# 21. `useDispatch()`

State를 읽는 것이 `useSelector()`라면 Action을 Store로 보내기 위해서는 `useDispatch()`를 사용합니다.

```jsx
import { useDispatch } from "react-redux";

function Counter() {
    const dispatch = useDispatch();

    // ...
}
```

`useDispatch()`는 현재 Provider가 제공하는 Redux Store의 `dispatch` 함수를 사용할 수 있도록 해줍니다.

따라서:

```javascript
dispatch(increment());
```

와 같이 사용할 수 있습니다.

---

# 22. `dispatch(increment())`를 정확히 이해하자

다음 코드는 Redux Toolkit을 이해하는 데 매우 중요합니다.

```javascript
dispatch(increment());
```

처음 보면 `increment()`가 State를 직접 증가시키는 것처럼 보일 수 있습니다.

하지만 그렇지 않습니다.

`increment()`는 **Action Creator**입니다.

먼저:

```javascript
increment()
```

가 실행되어:

```javascript
{
    type: "counter/increment"
}
```

라는 Action 객체를 만듭니다.

그 Action 객체가 `dispatch()`로 전달됩니다.

따라서:

```javascript
dispatch(increment());
```

를 풀어쓰면:

```javascript
const action = increment();

dispatch(action);
```

입니다.

전체 흐름은:

```text
increment()
    ↓
Action Creator 실행
    ↓
{
    type: "counter/increment"
}
    ↓
dispatch(action)
    ↓
Redux Store
    ↓
Reducer
```

입니다.

---

# 23. `useSelector()`와 `useDispatch()`의 관계

두 Hook을 함께 보면 React와 Redux의 관계가 명확해집니다.

```text
                 React Component
                ┌───────────────┐
                │               │
          State │               │ Action
           읽기 │               │ 전달
                │               │
                ↑               ↓
          useSelector()    useDispatch()
                ↑               │
                │               │
                │        dispatch(action)
                │               ↓
             ┌──────────────────────┐
             │     Redux Store      │
             │                      │
             │        State         │
             └──────────┬───────────┘
                        │
                        │ state + action
                        ▼
                   ┌─────────┐
                   │ Reducer │
                   └────┬────┘
                        │
                        │ new state
                        ▼
                   Redux Store
```

쉽게 기억하면:

```text
useSelector()
=
Redux → React


useDispatch()
=
React → Redux
```

입니다.

---

# 24. Counter 전체 구현

이제 지금까지 배운 내용을 하나의 프로젝트로 연결해보겠습니다.

```text
src
│
├── app
│    └── store.js
│
├── features
│    └── counter
│         ├── counterSlice.js
│         └── Counter.jsx
│
├── App.jsx
└── main.jsx
```

Redux Toolkit에서는 이렇게 기능 중심으로 관련 코드를 묶는 구조를 사용할 수 있습니다.

## `counterSlice.js`

```javascript
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
    value: 0
};

const counterSlice = createSlice({
    name: "counter",
    initialState,

    reducers: {
        increment(state) {
            state.value++;
        },

        decrement(state) {
            state.value--;
        },

        incrementByAmount(state, action) {
            state.value += action.payload;
        }
    }
});

export const {
    increment,
    decrement,
    incrementByAmount
} = counterSlice.actions;

export default counterSlice.reducer;
```

## `store.js`

```javascript
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

const store = configureStore({
    reducer: {
        counter: counterReducer
    }
});

export default store;
```

## `main.jsx`

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { Provider } from "react-redux";

import App from "./App.jsx";
import store from "./app/store.js";

createRoot(document.getElementById("root")).render(
    <StrictMode>
        <Provider store={store}>
            <App />
        </Provider>
    </StrictMode>
);
```

## `Counter.jsx`

```jsx
import {
    useDispatch,
    useSelector
} from "react-redux";

import {
    increment,
    decrement,
    incrementByAmount
} from "./counterSlice";

function Counter() {
    const count = useSelector(
        state => state.counter.value
    );

    const dispatch = useDispatch();

    return (
        <div>
            <h1>{count}</h1>

            <button onClick={() => dispatch(increment())}>
                +1
            </button>

            <button onClick={() => dispatch(decrement())}>
                -1
            </button>

            <button
                onClick={() =>
                    dispatch(incrementByAmount(10))
                }
            >
                +10
            </button>
        </div>
    );
}

export default Counter;
```

## `App.jsx`

```jsx
import Counter from "./features/counter/Counter";

function App() {
    return (
        <main>
            <Counter />
        </main>
    );
}

export default App;
```

이제 Redux Toolkit과 React-Redux를 이용한 기본적인 Redux 애플리케이션이 완성되었습니다.

---

# 25. +1 버튼을 클릭하면 실제로 무슨 일이 일어나는가?

이제 지금까지 배운 모든 개념을 실제 실행 흐름으로 연결해보겠습니다.

사용자가 `+1` 버튼을 클릭합니다.

```jsx
<button onClick={() => dispatch(increment())}>
    +1
</button>
```

## STEP 1. Event Handler 실행

클릭 이벤트에 의해 다음 코드가 실행됩니다.

```javascript
dispatch(increment());
```

## STEP 2. Action Creator 실행

먼저:

```javascript
increment()
```

가 실행됩니다.

결과:

```javascript
{
    type: "counter/increment"
}
```

## STEP 3. Action Dispatch

생성된 Action이 `dispatch()`로 전달됩니다.

```javascript
dispatch({
    type: "counter/increment"
});
```

## STEP 4. Store가 Action 처리

Action이 Redux Store의 처리 흐름으로 들어갑니다.

```text
dispatch(action)
      ↓
Redux Store
      ↓
Reducer
```

## STEP 5. Reducer 실행

현재 State와 Action을 이용하여 해당 Reducer 로직이 실행됩니다.

```javascript
state.value++;
```

## STEP 6. Immer가 새로운 State 생성

개발자는 mutation 형태로 작성했지만 Immer가 immutable update를 처리합니다.

```text
value: 0
   ↓
state.value++
   ↓
Immer
   ↓
value: 1
```

## STEP 7. Redux Store State 변경

Redux Store는 새로운 State를 갖게 됩니다.

```javascript
{
    counter: {
        value: 1
    }
}
```

## STEP 8. Selector 결과 변경

`Counter` 컴포넌트가 사용하는 Selector:

```javascript
state => state.counter.value
```

의 결과가:

```text
0 → 1
```

로 변경됩니다.

## STEP 9. Counter Re-render

`Counter` Component가 새로운 State 값을 사용하여 다시 렌더링됩니다.

```text
0
↓
1
```

---

# 26. 전체 실행 흐름

지금까지의 모든 과정을 하나로 연결하면 다음과 같습니다.

```text
┌────────────────────────────┐
│       User clicks +1       │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│        increment()         │
│       Action Creator       │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│           Action           │
│                            │
│ type: counter/increment    │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│      dispatch(action)      │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│        Redux Store         │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│       counterReducer       │
│                            │
│       state.value++        │
└─────────────┬──────────────┘
              ↓
          Immer 처리
              ↓
┌────────────────────────────┐
│         New State          │
│                            │
│         value: 1           │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│        Redux Store         │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│       useSelector()        │
│                            │
│ state.counter.value        │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│     Counter Re-render      │
│                            │
│             1              │
└────────────────────────────┘
```

이 흐름은 이번 PART에서 가장 중요합니다.

---

# 27. Redux Toolkit과 기존 Redux의 관계

Redux Toolkit을 사용한다고 해서 PART 1에서 배운 Redux의 구성 요소가 사라지는 것은 아닙니다.

| Redux 개념       | Redux Toolkit / React-Redux |
| -------------- | --------------------------- |
| State          | `initialState`              |
| Reducer        | `createSlice().reducer`     |
| Action Type    | `createSlice()`가 생성         |
| Action Creator | `slice.actions`             |
| Store          | `configureStore()`          |
| dispatch       | `useDispatch()`를 통해 사용      |
| State 읽기       | `useSelector()`             |
| React 연결       | `Provider`                  |

즉,

> **Redux Toolkit은 Redux의 개념을 없애는 것이 아니라 반복적인 구현을 상당 부분 자동화한다.**

라고 이해하는 것이 정확합니다.

---

# 28. Redux State와 Local State를 구분하자

Redux를 배웠다고 해서 모든 State를 Redux에 넣어야 하는 것은 아닙니다.

예를 들어:

```javascript
const [open, setOpen] = useState(false);
```

처럼 특정 Component에서만 필요한 Modal 상태라면 Local State로 충분할 수 있습니다.

반면 다음과 같은 데이터는 Redux State의 후보가 될 수 있습니다.

```text
여러 화면에서 공유되는 로그인 정보

여러 Component에서 사용하는 장바구니

애플리케이션 여러 영역에서 공유되는 상태
```

개념적으로:

```text
State
│
├── Local State
│      ↓
│   useState()
│
└── Shared / Application State
       ↓
     Redux
```

라고 생각할 수 있습니다.

Redux의 목적은 모든 State를 무조건 중앙화하는 것이 아닙니다.

> **여러 영역에서 공유되고 중앙 관리할 가치가 있는 State를 체계적으로 관리하는 것이 Redux의 핵심 목적입니다.**

---

# 29. 이번 PART 핵심 정리

전체 구조를 마지막으로 한 번 정리해봅시다.

```text
                   createSlice()
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
           Reducer          Action Creator
              │                   │
              ↓                   ↓
       configureStore()          Action
              │                   │
              ↓                   │
         Redux Store ←────── dispatch()
              │                   ↑
              │                   │
              ↓                   │
         useSelector()       useDispatch()
              │                   ↑
              ↓                   │
             React Component
```

그리고 React와 Redux Store의 연결까지 포함하면:

```text
Redux Store
    │
    ▼
Provider
    │
    ▼
React Context
    │
    ▼
React Component Tree
    │
    ├── useSelector()
    │       ↓
    │   State 읽기
    │
    └── useDispatch()
            ↓
        Action 전달
```

핵심 API를 한 문장씩 정리하면 다음과 같습니다.

```text
createSlice()
=
State + Reducer Logic을 정의하고
Reducer와 Action Creator를 생성


configureStore()
=
Reducer들을 이용하여
Redux Store를 구성


Provider
=
Redux Store를
React Component Tree에 제공


useSelector()
=
Redux State → React


useDispatch()
=
React → Redux Action Dispatch


Immer
=
Reducer에서 mutation처럼 작성한 코드를
안전한 immutable update로 처리
```

Redux Toolkit을 사용하더라도 Redux의 기본 원리는 변하지 않습니다.

```text
Event
  ↓
Action
  ↓
dispatch
  ↓
Store
  ↓
Reducer
  ↓
New State
  ↓
Store
  ↓
React
```

---

# 30. 다음 단계 — Asynchronous Redux

여기까지 학습하면 Redux Toolkit을 이용한 **동기적인 상태 관리**의 기본 구조가 완성됩니다.

하지만 실제 웹 애플리케이션에서는 서버와 통신해야 합니다.

예를 들어:

```text
로그인
상품 목록 조회
회원 정보 조회
상품 등록
주문 생성
```

등은 REST API 요청이 필요합니다.

```javascript
const response = await fetch("/api/products");
const products = await response.json();
```

그런데 Reducer의 역할은 State를 계산하는 것입니다.

```text
Reducer
│
├── State 계산       O
├── Action 처리      O
└── API Request      X
```

즉 Reducer 안에서 API 요청과 같은 비동기 Side Effect를 직접 처리하는 구조로 만들면 안 됩니다.

그러면 새로운 질문이 생깁니다.

> **Redux 애플리케이션에서 비동기 작업은 어디에서 처리해야 할까?**

이 질문이 다음 PART의 시작입니다.

```text
Reducer와 Side Effect
        ↓
Middleware
        ↓
Thunk
        ↓
Redux Thunk
        ↓
Promise
        ↓
createAsyncThunk()
        ↓
pending
fulfilled
rejected
        ↓
extraReducers
        ↓
React + Redux + REST API
```

다음 PART에서는 JavaScript의 `Promise`, `async/await`, `fetch()`가 Redux의 비동기 처리와 어떻게 연결되는지 살펴봅니다.

그리고 마지막에는 직접 API 요청 상태를 관리하는 방식이 왜 복잡해지는지 확인하면서 **RTK Query가 왜 필요한가**로 자연스럽게 연결합니다.
