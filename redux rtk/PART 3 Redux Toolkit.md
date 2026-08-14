# PART 3. Redux Toolkit

PART 1에서는 Redux의 핵심 데이터 흐름을 학습했습니다.

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

이제 중요한 질문이 생깁니다.

> 이 구조를 실제 React 애플리케이션에서는 어떻게 구현할 것인가?

현재 Redux 애플리케이션에서는 일반적으로 **Redux Toolkit(RTK)**을 사용합니다.

Redux Toolkit은 Redux의 동작 원리를 바꾸는 것이 아닙니다.

PART 1에서 배운 Redux 구조를 더 적은 코드와 안전한 기본 설정으로 구현할 수 있도록 도와주는 공식 도구입니다.

---

# 1. Vanilla Redux의 문제

PART 1에서 Counter를 Redux 방식으로 구현하면 다음과 같은 코드가 필요했습니다.

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

기능은 단순한데 작성해야 하는 코드가 많습니다.

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

상태와 기능이 많아지면 이러한 코드도 계속 증가합니다.

이를 흔히 **Boilerplate Code**가 많다고 표현합니다.

---

# 2. Redux Toolkit이란?

Redux Toolkit은 Redux 애플리케이션을 작성하기 위한 공식적인 도구 모음입니다.

대표적으로 다음 API를 제공합니다.

```text
Redux Toolkit
│
├── configureStore()
├── createSlice()
├── createReducer()
├── createAction()
├── createAsyncThunk()
└── createApi()      ← RTK Query
```

이번 PART에서는 먼저 동기적인 Redux 상태 관리에 집중합니다.

핵심 API는 두 개입니다.

```text
createSlice()
configureStore()
```

그리고 React에서 Redux Store를 사용하기 위해 `react-redux`가 제공하는 다음 기능을 사용합니다.

```text
Provider
useSelector()
useDispatch()
```

전체 구조는 다음과 같습니다.

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
               React Redux
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Provider  useSelector useDispatch
```

---

# 3. `@reduxjs/toolkit`과 `react-redux`

React에서 Redux Toolkit을 사용할 때 보통 두 패키지를 설치합니다.

```bash
npm install @reduxjs/toolkit react-redux
```

두 라이브러리의 역할은 다릅니다.

## `@reduxjs/toolkit`

Redux 자체를 구성하고 상태를 관리하기 위한 도구를 제공합니다.

```text
@reduxjs/toolkit
│
├── createSlice()
├── configureStore()
├── createAsyncThunk()
└── RTK Query
```

## `react-redux`

React와 Redux Store를 연결합니다.

```text
react-redux
│
├── Provider
├── useSelector()
└── useDispatch()
```

따라서 다음과 같이 구분할 수 있습니다.

```text
@reduxjs/toolkit

Redux를 구성하고 관리


react-redux

React가 Redux를 사용할 수 있도록 연결
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

전체 State를 기능 또는 도메인별로 나누어 생각할 수 있습니다.

```text
Redux State
│
├── counter
│
│    └── value
│
├── user
│
│    ├── name
│
│    └── loggedIn
│
└── cart
     └── items
```

Redux Toolkit에서는 특정 기능과 관련된:

```text
State
+
Reducer Logic
+
Action Creator
+
Action Type
```

을 하나의 단위로 구성할 수 있습니다.

이 단위를 **Slice**라고 합니다.

예를 들어:

```text
Counter Slice
│
├── State
│     └── value
│
├── Reducers
│     ├── increment
│     └── decrement
│
├── Action Creators
│     ├── increment()
│     └── decrement()
│
└── Action Types
      ├── counter/increment
      └── counter/decrement
```

입니다.

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

구조를 분해하면:

```text
createSlice({
│
├── name
│
├── initialState
│
└── reducers
})
```

입니다.

---

# 6. `name`

```javascript
name: "counter"
```

`name`은 Slice의 이름입니다.

특히 생성되는 Action Type의 prefix로 사용됩니다.

예를 들어:

```javascript
increment(state) {
    state.value++;
}
```

라는 Reducer가 있으면 Redux Toolkit은 개념적으로 다음 Action Type을 만듭니다.

```text
counter/increment
```

즉:

```text
name
 ↓
counter

reducer name
 ↓
increment

결과
 ↓
counter/increment
```

입니다.

여기서 매우 중요한 점이 있습니다.

> `createSlice()`의 `name`이 Redux Store State의 key를 직접 결정하는 것은 아닙니다.

Store의 State key는 뒤에서 `configureStore()`의 `reducer` 설정을 통해 결정됩니다.

---

# 7. `initialState`

```javascript
initialState: {
    value: 0
}
```

해당 Slice가 처음 사용할 State입니다.

따라서 Counter Slice의 초기 State는:

```javascript
{
    value: 0
}
```

입니다.

나중에 Store에 다음과 같이 등록하면:

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

형태가 됩니다.

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

`reducers`에는 이 Slice의 State를 변경하기 위한 Reducer 함수들을 정의합니다.

PART 1에서 Reducer는 다음과 같이 설명했습니다.

```text
Current State + Action
          ↓
       Reducer
          ↓
      New State
```

Redux Toolkit에서도 이 원리는 동일합니다.

다만 `createSlice()`가 Reducer와 Action을 함께 구성해주는 것입니다.

---

# 9. `createSlice()`가 자동으로 만들어주는 것

다음 코드를 작성했다고 생각해봅시다.

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

Redux Toolkit은 이 정의를 기반으로 필요한 Redux 구성 요소를 생성합니다.

개념적으로:

```text
createSlice()
     │
     ├── Reducer
     │
     ├── Action Creator
     │
     └── Action Type
```

을 만들어줍니다.

따라서:

```javascript
counterSlice.reducer
```

를 통해 Slice Reducer를 얻을 수 있고,

```javascript
counterSlice.actions
```

를 통해 생성된 Action Creator들을 얻을 수 있습니다.

---

# 10. `counterSlice.actions`

다음과 같이 Action Creator를 꺼낼 수 있습니다.

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

개념적으로:

```javascript
increment()
```

결과는:

```javascript
{
    type: "counter/increment"
}
```

입니다.

즉 PART 1에서 직접 만들었던:

```javascript
function increment() {
    return {
        type: "counter/increment"
    };
}
```

를 Redux Toolkit이 자동으로 생성한 것입니다.

---

# 11. `payload`가 필요한 Action

값을 함께 전달해야 할 수도 있습니다.

예를 들어:

```javascript
reducers: {

    incrementByAmount(state, action) {
        state.value += action.payload;
    }

}
```

사용할 때:

```javascript
dispatch(incrementByAmount(10));
```

`incrementByAmount(10)`은 개념적으로 다음 Action을 만듭니다.

```javascript
{
    type: "counter/incrementByAmount",
    payload: 10
}
```

Reducer에서는:

```javascript
action.payload
```

를 통해 `10`을 사용할 수 있습니다.

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

# 12. `counterSlice.reducer`

`createSlice()`는 해당 Slice의 Reducer도 만들어줍니다.

```javascript
export default counterSlice.reducer;
```

보통 파일은 다음처럼 작성합니다.

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

이 파일 하나에서:

```text
State
Reducer
Action Creator
Action Type
```

가 함께 관리됩니다.

이것이 Slice의 중요한 장점입니다.

---

# 13. 그런데 `state.value++`는 State 직접 수정 아닌가?

PART 1에서는 Reducer에서 기존 State를 직접 수정하면 안 된다고 배웠습니다.

전통적인 Redux에서는 다음 코드가 문제가 됩니다.

```javascript
state.value++;
```

그런데 `createSlice()`에서는:

```javascript
increment(state) {
    state.value++;
}
```

라고 작성합니다.

모순처럼 보입니다.

Redux Toolkit에서는 이것이 가능합니다.

그 이유가 **Immer**입니다.

---

# 14. Immer

Redux Toolkit은 내부적으로 Immer를 사용하여 Reducer의 불변성 처리를 쉽게 해줍니다.

개발자는 마치 State를 직접 수정하는 것처럼 작성할 수 있습니다.

```javascript
state.value++;
```

하지만 Immer가 이를 기반으로 새로운 불변 State를 생성합니다.

개념적으로:

```text
개발자가 작성

state.value++;

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

따라서 `createSlice()`의 Reducer 안에서는 이러한 형태의 코드를 사용할 수 있습니다.

```javascript
state.items.push(product);

state.user.name = "Alice";

state.value++;
```

하지만 이것을:

> Redux에서는 State를 직접 수정해도 된다.

라고 이해하면 안 됩니다.

정확한 의미는:

> Redux Toolkit의 Immer 기반 Reducer 안에서는 mutation 형태의 코드를 작성할 수 있고, Immer가 안전한 immutable update 결과를 생성한다.

입니다.

---

# 15. `configureStore()`

Slice를 만들었으면 이제 Redux Store를 만들어야 합니다.

Redux Toolkit에서는 `configureStore()`를 사용합니다.

```javascript
import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
    reducer: {
        counter: counterReducer
    }
});

export default store;
```

전체 구조는:

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

---

# 16. 여러 Slice를 Store에 등록

실제 애플리케이션에는 여러 Slice가 있을 수 있습니다.

```javascript
const store = configureStore({

    reducer: {

        counter: counterReducer,

        user: userReducer,

        cart: cartReducer

    }

});
```

그러면 Redux State 구조는:

```javascript
{
    counter: {
        ...
    },

    user: {
        ...
    },

    cart: {
        ...
    }
}
```

가 됩니다.

즉:

```text
configureStore

reducer: {
    counter: counterReducer,
    user: userReducer,
    cart: cartReducer
}

             ↓

Redux State

{
    counter: ...,
    user: ...,
    cart: ...
}
```

입니다.

---

# 17. Store State의 Key는 누가 결정하는가?

이 부분은 매우 중요합니다.

다음 Slice가 있다고 합시다.

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

그리고 Store를 다음처럼 구성합니다.

```javascript
const store = configureStore({

    reducer: {
        myCounter: counterSlice.reducer
    }

});
```

그러면 State는:

```javascript
{
    myCounter: {
        value: 0
    }
}
```

입니다.

즉:

```text
createSlice name

"counter"

      ↓

Action Type Prefix

counter/increment
```

반면:

```text
configureStore reducer key

myCounter

      ↓

State Address

state.myCounter
```

입니다.

따라서:

```text
Slice name
≠
반드시 Store State Key
```

입니다.

우연히 같은 이름을 사용하는 경우가 많을 뿐입니다.

---

# 18. `configureStore()`가 하는 일

`configureStore()`는 단순히 Reducer를 합치는 함수가 아닙니다.

Redux Store를 좋은 기본 설정과 함께 구성해줍니다.

개념적으로:

```text
configureStore()
│
├── Reducer 구성
├── Store 생성
├── 기본 Middleware 구성
├── Redux DevTools 지원
└── 개발 단계의 안전성 검사
```

따라서 기존 Redux의 저수준 Store 설정보다 훨씬 간단합니다.

비동기 Redux에서 사용할 Thunk Middleware도 이 기본 설정과 관련되어 있습니다.

이 부분은 PART 3에서 자세히 다룹니다.

---

# 19. 현재까지의 구조

여기까지 만들면 Redux 자체는 존재하지만 아직 React가 Store를 사용할 수 있도록 연결하지 않았습니다.

```text
counterSlice
      ↓
counterReducer
      ↓
configureStore()
      ↓
Redux Store


React Application
```

이 둘을 연결해야 합니다.

```text
Redux Store
      ↕
React Application
```

이 역할을 `react-redux`가 담당합니다.

---

# 20. `Provider`

React 애플리케이션이 Redux Store에 접근할 수 있도록 `Provider`를 사용합니다.

일반적인 Vite + React 프로젝트에서는 `main.jsx`에서 설정할 수 있습니다.

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

import { Provider } from "react-redux";

import App from "./App.jsx";
import store from "./store.js";

createRoot(document.getElementById("root")).render(
    <StrictMode>

        <Provider store={store}>
            <App />
        </Provider>

    </StrictMode>
);
```

핵심은:

```jsx
<Provider store={store}>
    <App />
</Provider>
```

입니다.

구조적으로:

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

Provider 하위의 React 컴포넌트들은 React-Redux API를 통해 Store에 접근할 수 있습니다.

---

# 21. Provider의 역할

Provider는 React 컴포넌트 트리에 Redux Store를 제공하는 역할을 합니다.

```text
             Redux Store
                  │
                  ↓
              Provider
                  │
                  ↓
                 App
          ┌───────┼───────┐
          ↓       ↓       ↓
       Header    Main    Footer
```

따라서 React 컴포넌트마다 직접:

```javascript
import store from "./store";
```

해서 사용하는 방식이 아닙니다.

React에서는 `Provider`와 React-Redux Hooks를 이용합니다.

대표적인 Hook이:

```text
useSelector()
useDispatch()
```

입니다.

---

# 22. `useSelector()`

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

State가:

```javascript
{
    counter: {
        value: 10
    }
}
```

이라면:

```javascript
state.counter.value
```

는:

```text
10
```

입니다.

---

# 23. Selector란?

다음 함수:

```javascript
state => state.counter.value
```

는 State에서 필요한 값을 선택합니다.

이러한 함수를 **Selector**라고 합니다.

개념적으로:

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

       ↓ Selector

state => state.counter.value

       ↓

10
```

입니다.

따라서:

```javascript
const count = useSelector(
    state => state.counter.value
);
```

는:

> Redux State에서 `counter.value`를 선택하여 React Component에서 사용한다.

는 의미입니다.

---

# 24. `useSelector()`와 Re-render

`useSelector()`는 단순히 한 번 State를 읽고 끝나는 함수가 아닙니다.

React-Redux는 Store의 변경을 확인하고 해당 Selector의 결과가 변경되면 필요한 React Component가 다시 렌더링될 수 있도록 연결합니다.

개념적으로:

```text
Redux Store

value = 0
   │
   ↓
useSelector()
   │
   ↓
Counter
   │
   ↓
0 표시


dispatch(increment())
   ↓

Redux Store

value = 1
   │
   ↓
Selector 결과 변경
   │
   ↓
Counter Re-render
   │
   ↓
1 표시
```

이 연결 덕분에 Redux State와 React UI가 동기화됩니다.

---

# 25. `useDispatch()`

State를 읽는 것이 `useSelector()`라면 Action을 Store에 전달하기 위해서는 `useDispatch()`를 사용합니다.

```jsx
import { useDispatch } from "react-redux";

function Counter() {

    const dispatch = useDispatch();

    // ...

}
```

`useDispatch()`는 Redux Store의 `dispatch` 함수를 사용할 수 있게 합니다.

이후:

```javascript
dispatch(increment());
```

와 같이 사용할 수 있습니다.

---

# 26. `dispatch(increment())`를 정확하게 이해하기

다음 코드는 매우 중요합니다.

```javascript
dispatch(increment());
```

처음 보면 `increment()`가 State를 증가시키는 것처럼 보입니다.

하지만 그렇지 않습니다.

`increment()`는 **Action Creator**입니다.

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

그 Action 객체가:

```javascript
dispatch(...)
```

에 전달됩니다.

따라서 다음 코드를:

```javascript
dispatch(increment());
```

개념적으로 풀어쓰면:

```javascript
const action = increment();

dispatch(action);
```

입니다.

흐름은:

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

# 27. `useSelector()`와 `useDispatch()`의 관계

두 Hook을 함께 보면 React와 Redux의 관계가 명확해집니다.

```text
                  React Component
                 ┌───────────────┐
                 │               │
           State │               │ Action
            읽기 │               │ 전달
                 │               │
                 ↑               ↓
          useSelector()     useDispatch()
                 ↑               │
                 │               │
                 │         dispatch(action)
                 │               ↓
              ┌──────────────────────┐
              │     Redux Store      │
              │                      │
              │        State         │
              └──────────┬───────────┘
                         │
                         │ state + action
                         ↓
                    ┌─────────┐
                    │ Reducer │
                    └────┬────┘
                         │
                         │ new state
                         ↓
                    Redux Store
```

쉽게 기억하면:

```text
useSelector
=
Redux → React


useDispatch
=
React → Redux
```

입니다.

---

# 28. Counter 전체 구현

이제 하나의 완성된 예제를 만들어봅시다.

프로젝트 구조:

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

Redux Toolkit에서는 기능 중심으로 관련 코드를 묶는 구조를 많이 사용할 수 있습니다.

---

# 29. `counterSlice.js`

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

이 파일에서 생성되는 것을 정리하면:

```text
counterSlice
│
├── reducer
│
└── actions
     │
     ├── increment()
     ├── decrement()
     └── incrementByAmount()
```

입니다.

---

# 30. `store.js`

```javascript
import { configureStore } from "@reduxjs/toolkit";

import counterReducer
    from "../features/counter/counterSlice";

const store = configureStore({

    reducer: {
        counter: counterReducer
    }

});

export default store;
```

이 설정으로 Store State는:

```javascript
{
    counter: {
        value: 0
    }
}
```

구조가 됩니다.

---

# 31. `main.jsx`

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

이제 `App` 이하의 컴포넌트에서 Redux Store를 사용할 수 있습니다.

---

# 32. `Counter.jsx`

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

            <button
                onClick={() => dispatch(increment())}
            >
                +1
            </button>

            <button
                onClick={() => dispatch(decrement())}
            >
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

---

# 33. `App.jsx`

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

이제 전체 애플리케이션이 완성되었습니다.

---

# 34. +1 버튼을 클릭하면 실제로 무슨 일이 일어나는가?

이제 가장 중요한 부분입니다.

사용자가:

```text
+1
```

버튼을 클릭합니다.

다음 코드가 실행됩니다.

```javascript
dispatch(increment());
```

### STEP 1

`increment()` Action Creator가 실행됩니다.

```javascript
increment()
```

↓

```javascript
{
    type: "counter/increment"
}
```

### STEP 2

Action이 `dispatch()`에 전달됩니다.

```javascript
dispatch({
    type: "counter/increment"
});
```

### STEP 3

Action이 Redux Store의 처리 흐름으로 들어갑니다.

```text
dispatch(action)
      ↓
Redux Store
```

### STEP 4

Store는 Reducer를 통해 Action을 처리합니다.

```text
Current State

{
   counter: {
      value: 0
   }
}

+

Action

{
   type: "counter/increment"
}

        ↓

counterReducer
```

### STEP 5

`increment` Reducer 로직이 실행됩니다.

```javascript
state.value++;
```

Immer가 새로운 State를 생성합니다.

```text
value: 0
   ↓
value: 1
```

### STEP 6

Redux Store가 새로운 State를 갖게 됩니다.

```javascript
{
    counter: {
        value: 1
    }
}
```

### STEP 7

`useSelector()`가 선택한 값이 변경됩니다.

```javascript
state => state.counter.value
```

결과:

```text
0 → 1
```

### STEP 8

`Counter` Component가 새로운 값을 사용하여 다시 렌더링됩니다.

화면:

```text
0
```

↓

```text
1
```

---

# 35. 전체 실행 흐름

지금까지의 모든 내용을 하나로 연결하면 다음과 같습니다.

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
│   dispatch(increment())    │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│        Redux Store         │
└─────────────┬──────────────┘
              ↓
┌────────────────────────────┐
│      counterReducer        │
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
│      Counter Re-render     │
│                            │
│             1              │
└────────────────────────────┘
```

이 그림은 PART 2에서 가장 중요한 그림입니다.

---

# 36. PART 1과 PART 2 연결

PART 1에서 배운 Redux 구성 요소는 Redux Toolkit에서도 사라지지 않았습니다.

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

즉 Redux Toolkit은 Redux의 개념을 감추는 것이 아니라 상당 부분 **자동화**합니다.

---

# 37. `createSlice()`의 역할을 다시 정리

`createSlice()`를 단순히:

> Reducer를 만드는 함수

라고 이해하면 부족합니다.

다음과 같이 이해하는 것이 좋습니다.

```text
             createSlice()
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Reducer    Actions   Action Types
                  │
                  ↓
            Action Creators
```

예:

```javascript
const counterSlice = createSlice({
    name: "counter",
    initialState,
    reducers: {
        increment(state) {
            state.value++;
        }
    }
});
```

결과적으로:

```text
counterSlice.reducer

counterSlice.actions.increment()

"counter/increment"
```

등이 만들어집니다.

---

# 38. `configureStore()`의 역할을 다시 정리

`configureStore()`는 여러 Reducer를 하나의 Redux Store에 구성합니다.

```text
counterReducer ──┐
                 │
userReducer ─────┼──→ configureStore()
                 │          ↓
cartReducer ─────┘      Redux Store
```

그리고 등록한 key가 State 구조를 결정합니다.

```javascript
reducer: {

    counter: counterReducer,

    user: userReducer

}
```

↓

```javascript
{
    counter: ...,
    user: ...
}
```

입니다.

---

# 39. Provider / useSelector / useDispatch 관계

React와 Redux 사이에는 `react-redux`가 위치합니다.

```text
                 React
                   │
        ┌──────────┼──────────┐
        │          │          │
    Provider  useSelector useDispatch
        │          ↑          ↓
        └──────────┼──────────┘
                   │
              Redux Store
```

각각의 역할은:

```text
Provider
→ Store를 React Component Tree에 제공

useSelector
→ Store State를 선택해서 읽음

useDispatch
→ Store에 Action을 dispatch
```

입니다.

---

# 40. Redux State와 Local State를 구분하자

Redux를 배웠다고 모든 State를 Redux에 넣는 것은 좋지 않습니다.

예를 들어:

```javascript
const [open, setOpen] = useState(false);
```

처럼 특정 Component에서만 필요한 Modal 상태라면 Local State로 충분할 수 있습니다.

반면:

```text
여러 화면에서 공유되는 로그인 정보

여러 Component에서 사용하는 장바구니

애플리케이션 전역에서 공유하는 설정
```

등은 Redux State 후보가 될 수 있습니다.

따라서:

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

Redux의 목적은 모든 State를 중앙화하는 것이 아니라 **중앙 관리가 필요한 State를 체계적으로 관리하는 것**입니다.

---

# 41. 아직 해결되지 않은 문제

여기까지의 Redux Toolkit은 동기적인 State 변경에는 매우 편리합니다.

예:

```javascript
dispatch(increment());

dispatch(addItem(product));

dispatch(logout());
```

그런데 실제 웹 애플리케이션에서는 서버와 통신해야 합니다.

예를 들어:

```text
로그인

상품 목록 조회

회원 정보 조회

상품 등록

주문 생성
```

등은 REST API 호출이 필요합니다.

예를 들어 다음과 같은 코드를 생각해볼 수 있습니다.

```javascript
const response =
    await fetch("/api/products");

const products =
    await response.json();
```

문제가 있습니다.

Reducer는 State를 계산하는 역할을 담당해야 합니다.

Reducer 안에서 이런 비동기 작업을 직접 수행하는 구조로 만들 수는 없습니다.

```text
Reducer
│
├── State 계산       O
│
├── Action 처리      O
│
└── API Request      X
```

그렇다면 다음 질문이 생깁니다.

> Redux 애플리케이션에서 비동기 작업은 어디에서 처리해야 하는가?

이 질문이 PART 3의 시작입니다.

---

# 42. PART 2 핵심 정리

Redux Toolkit을 사용한 React 애플리케이션의 기본 구조는 다음과 같습니다.

```text
                   createSlice()
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
          Reducer            Action Creator
             │                     │
             ↓                     ↓
       configureStore()        Action
             │                     │
             ↓                     │
        Redux Store ←──── dispatch()
             │                  ↑
             │                  │
             ↓                  │
        useSelector()      useDispatch()
             │                  ↑
             ↓                  │
             React Component
```

반드시 기억해야 할 핵심은 다음과 같습니다.

```text
createSlice()
=
State + Reducer Logic을 정의하고
Reducer와 Action Creator를 생성


configureStore()
=
Reducer들을 이용하여
Redux Store 구성


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

그리고 Redux Toolkit을 사용하더라도 PART 1에서 배운 기본 원리는 변하지 않습니다.

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

# 다음 단계 — PART 3

이제 Redux의 동기적인 상태 관리는 완성되었습니다.

다음 PART에서는 실제 웹 애플리케이션에서 매우 중요한 **Asynchronous Redux**를 다룹니다.

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

특히 JavaScript의 `Promise`, `async/await`, `fetch()`가 Redux의 비동기 처리와 어떻게 연결되는지를 중심으로 설명합니다.

그리고 PART 3 마지막에서는 직접 API 요청을 관리하는 방식이 왜 복잡해지는지를 확인한 뒤, **PART 4의 RTK Query가 왜 필요한지** 자연스럽게 연결합니다.
