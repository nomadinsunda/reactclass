# PART 2. Redux Fundamentals

Redux Toolkit을 제대로 이해하려면 `createSlice()`, `configureStore()` 같은 API부터 외우기보다 먼저 **Redux가 어떤 문제를 해결하기 위해 등장했고, 상태가 어떤 흐름으로 변경되는지**를 이해해야 합니다.

Redux의 핵심은 API가 아니라 다음 흐름에 있습니다.

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
UI
```

이 흐름을 이해하면 이후 Redux Toolkit에서 등장하는 `createSlice()`, `configureStore()`, `useSelector()`, `useDispatch()`도 훨씬 자연스럽게 연결됩니다.

---

# 1. 출발점: React의 State

React 애플리케이션에서 화면에 표시되는 데이터는 일반적으로 컴포넌트의 State로 관리합니다.

```jsx
import { useState } from "react";

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <>
            <h1>{count}</h1>

            <button onClick={() => setCount(count + 1)}>
                +1
            </button>
        </>
    );
}
```

여기서 `count`는 `Counter` 컴포넌트가 관리하는 State입니다.

```text
Counter Component
│
├── State
│    └── count = 0
│
├── setCount()
│
└── UI
     └── {count}
```

버튼을 클릭하면 다음 과정이 발생합니다.

```text
User Click
    ↓
Event Handler
    ↓
setCount(count + 1)
    ↓
State 변경
    ↓
Component Re-render
    ↓
변경된 UI
```

이 정도 규모라면 Redux가 필요하지 않습니다.

---

# 2. React State는 컴포넌트에 소속된다

React State를 이해할 때 중요한 특징이 있습니다.

> **State는 기본적으로 그 State를 선언한 컴포넌트에 소속됩니다.**

예를 들어:

```jsx
function App() {
    const [user, setUser] = useState(null);

    return (
        <>
            <Header user={user} />
            <Main user={user} />
            <Sidebar user={user} />
        </>
    );
}
```

`user` State는 `App`이 관리합니다.

하지만 여러 컴포넌트가 `user`를 필요로 한다면 상위 컴포넌트에서 하위 컴포넌트로 전달해야 합니다.

```text
App
│
├── Header
│    └── 사용자 이름
│
├── Main
│    └── 사용자 정보
│
└── Sidebar
     └── 사용자 권한
```

애플리케이션이 작다면 이것은 문제가 아닙니다.

문제는 컴포넌트 구조가 깊어질 때 나타납니다.

---

# 3. Props Drilling

다음과 같은 컴포넌트 트리를 생각해봅시다.

```text
App
 ↓
Layout
 ↓
Main
 ↓
Content
 ↓
UserProfile
```

`App`이 가지고 있는 `user`를 가장 아래의 `UserProfile`에서 사용해야 한다면 중간 컴포넌트들이 `user`를 사용하지 않더라도 계속 전달해야 할 수 있습니다.

```jsx
<Layout user={user} />
```

```jsx
<Main user={user} />
```

```jsx
<Content user={user} />
```

```jsx
<UserProfile user={user} />
```

결국 데이터가 다음과 같이 이동합니다.

```text
App
 │ user
 ↓
Layout
 │ user
 ↓
Main
 │ user
 ↓
Content
 │ user
 ↓
UserProfile
```

이러한 현상을 흔히 **Props Drilling**이라고 합니다.

중요한 점은 Props Drilling 자체가 잘못된 것은 아니라는 것입니다.

문제는 애플리케이션이 커지고 여러 컴포넌트가 동일한 상태를 공유하기 시작하면 **상태를 어디에 두고, 누가 변경하며, 어떻게 변경되었는지를 관리하기 어려워질 수 있다는 것**입니다.

---

# 4. Shared State

쇼핑몰 애플리케이션을 생각해봅시다.

애플리케이션 전체에서 다음과 같은 상태가 필요할 수 있습니다.

```text
로그인 사용자
장바구니
검색 / 필터 조건
UI 설정
```

특히 장바구니 State는 여러 컴포넌트가 함께 사용할 수 있습니다.

```text
Header
 └── 장바구니 상품 개수

ProductDetail
 └── 장바구니 상품 추가

CartPage
 └── 장바구니 목록

CheckoutPage
 └── 주문 상품
```

이처럼 여러 부분에서 함께 사용하는 상태를 **Shared State**라고 생각할 수 있습니다.

애플리케이션이 커질수록 다음 질문이 중요해집니다.

> 공유 State를 어디에 보관할 것인가?

> 누가 State를 변경할 수 있는가?

> 어떤 이유로 State가 변경되었는가?

> State 변경 과정을 어떻게 추적할 것인가?

Redux는 바로 이러한 문제를 **일관된 상태 변경 흐름**으로 관리하기 위한 도구입니다.

---

# 5. Redux란?

Redux는 JavaScript 애플리케이션의 **예측 가능한 상태 관리(Predictable State Management)**를 위한 라이브러리입니다.

Redux에서는 공유할 상태를 중앙의 **Store**에서 관리합니다.

```text
             Redux Store
        ┌──────────────────┐
        │      State       │
        │                  │
        │ user             │
        │ cart             │
        │ filter           │
        └──────────────────┘
             ↑        ↓
             │        │
       상태 변경 요청   상태 조회
             │        │
        React Components
```

하지만 Redux를 단순히 다음과 같이 이해하면 안 됩니다.

```text
Redux = 전역 변수를 만드는 도구
```

Redux에서 더 중요한 것은 **State를 어디에 저장하느냐보다 State가 변경되는 절차를 통제하는 것**입니다.

---

# 6. 왜 그냥 전역 객체를 사용하지 않는가?

다음과 같은 전역 객체가 있다고 생각해봅시다.

```javascript
const state = {
    count: 0
};
```

어디에서든 다음처럼 변경할 수 있다면:

```javascript
state.count++;
```

애플리케이션이 커졌을 때 문제가 발생합니다.

```text
Component A ──┐
Component B ──┼──→ state.count 직접 변경
Component C ──┤
Function D  ──┘
```

`count`가 예상하지 못한 값이 되었을 때 **누가, 언제, 왜 변경했는지 추적하기 어려워집니다.**

Redux는 State 변경을 이런 방식으로 허용하지 않습니다.

```text
직접 변경

Component ────────── X ─────────→ State
```

대신 정해진 흐름을 사용합니다.

```text
Component
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
```

이 **정해진 상태 변경 경로**가 Redux의 핵심입니다.

---

# 7. Redux의 핵심 구성 요소

Redux의 핵심 개념은 다음 여섯 가지입니다.

| 구성 요소              | 역할                             |
| ------------------ | ------------------------------ |
| **State**          | 현재 애플리케이션의 상태 데이터              |
| **Store**          | State와 Redux 처리 흐름을 관리         |
| **Action**         | 발생한 일을 표현하는 객체                 |
| **Action Creator** | Action 객체를 생성하는 함수             |
| **dispatch()**     | Action을 Store에 전달              |
| **Reducer**        | 현재 State와 Action으로 다음 State 계산 |

이 개념들을 각각 암기하기보다 **하나의 흐름으로 연결해서 이해하는 것**이 중요합니다.

```text
┌────────────────────┐
│  React Component   │
└─────────┬──────────┘
          │
          │ dispatch(action)
          ↓
┌────────────────────┐
│       Store        │
└─────────┬──────────┘
          │
          │ current state + action
          ↓
┌────────────────────┐
│      Reducer       │
└─────────┬──────────┘
          │
          │ new state
          ↓
┌────────────────────┐
│       Store        │
│     New State      │
└─────────┬──────────┘
          │
          ↓
┌────────────────────┐
│  React Component   │
└────────────────────┘
```

이제 각각의 역할을 살펴보겠습니다.

---

# 8. State

State는 **현재 애플리케이션의 상태를 표현하는 데이터**입니다.

예를 들어 쇼핑몰의 Redux State는 다음과 같을 수 있습니다.

```javascript
{
    user: {
        id: 1,
        name: "Alice"
    },

    cart: {
        items: []
    },

    filter: {
        category: "all"
    }
}
```

구조적으로 보면:

```text
Application State
│
├── user
│    ├── id
│    └── name
│
├── cart
│    └── items
│
└── filter
     └── category
```

Redux에서는 이 State를 **Store가 관리합니다.**

---

# 9. Store

**Store는 Redux 상태 관리의 중심 객체입니다.**

Store를 단순히 State를 저장하는 객체라고만 이해하면 부족합니다.

Store는 개념적으로 다음 역할을 담당합니다.

```text
Redux Store
│
├── State 보관
│
├── dispatch() 제공
│
├── Reducer 실행 흐름 관리
│
└── State 변경 알림
```

따라서 반드시 다음을 구분해야 합니다.

```text
Store ≠ State
```

* **State** → 관리되는 데이터
* **Store** → 그 State와 Redux 처리 흐름을 관리하는 객체

---

# 10. Action

Redux에서는 Component가 Redux State를 직접 변경하지 않습니다.

대신 **무슨 일이 발생했는지를 설명하는 객체**를 Store에 전달합니다.

```javascript
{
    type: "counter/increment"
}
```

이 객체를 **Action**이라고 합니다.

추가 데이터가 필요한 경우 일반적으로 `payload`를 사용합니다.

```javascript
{
    type: "cart/itemAdded",

    payload: {
        id: 10,
        name: "Keyboard"
    }
}
```

여기서:

```text
type
 └── 어떤 종류의 사건이 발생했는가?

payload
 └── 그 사건과 함께 전달할 데이터
```

라고 이해할 수 있습니다.

### Action은 명령이라기보다 사건이다

Action을 다음과 같이 이해하기 쉽습니다.

```text
"count를 증가시켜라."
```

하지만 Redux에서는 Action을 **상태 변경과 관련하여 무슨 일이 발생했는지를 설명하는 데이터**라고 이해하는 것이 더 좋습니다.

예:

```javascript
{
    type: "cart/itemAdded",
    payload: product
}
```

Action은 사건을 표현하고, **실제로 State를 어떻게 변경할지는 Reducer가 결정합니다.**

---

# 11. Action Creator

Action 객체를 매번 직접 작성할 수도 있습니다.

```javascript
store.dispatch({
    type: "counter/increment"
});
```

하지만 Action을 생성하는 함수를 만들 수도 있습니다.

```javascript
function increment() {
    return {
        type: "counter/increment"
    };
}
```

이러한 함수를 **Action Creator**라고 합니다.

```text
increment
   ↓
Action Creator

increment()
   ↓
{
    type: "counter/increment"
}
   ↓
Action
```

따라서 반드시 구분해야 합니다.

```text
Action Creator = 함수
Action         = 객체
```

Redux Toolkit에서는 이후 배우게 될 `createSlice()`가 이러한 Action Creator를 자동으로 생성해줍니다.

---

# 12. dispatch()

Action 객체를 만들었다고 해서 Redux가 자동으로 처리하는 것은 아닙니다.

Action을 **Store의 처리 흐름으로 전달**해야 합니다.

그 역할을 하는 것이 `dispatch()`입니다.

```javascript
store.dispatch({
    type: "counter/increment"
});
```

Action Creator를 사용하면:

```javascript
store.dispatch(increment());
```

이를 두 단계로 풀어보면 더욱 명확합니다.

```javascript
const action = increment();

store.dispatch(action);
```

즉:

```text
Action Creator
      ↓
    Action
      ↓
dispatch(action)
      ↓
    Store
```

여기서 중요한 점이 있습니다.

> **dispatch()가 State를 직접 변경하는 것은 아닙니다.**

`dispatch()`의 역할은 **Action을 Store의 Redux 처리 흐름으로 전달하는 것**입니다.

---

# 13. Reducer

Store가 Action을 받으면 **Reducer를 이용하여 다음 State를 계산합니다.**

Reducer는 개념적으로 다음과 같은 함수입니다.

```javascript
function reducer(state, action) {
    // 다음 State 계산
    return newState;
}
```

수학적인 함수처럼 표현하면:

```text
f(state, action) → newState
```

즉:

```text
Current State
      +
    Action
      ↓
   Reducer
      ↓
  New State
```

예를 들어:

```javascript
function counterReducer(state = { value: 0 }, action) {

    if (action.type === "counter/increment") {
        return {
            value: state.value + 1
        };
    }

    return state;
}
```

현재 State가:

```javascript
{
    value: 0
}
```

이고 Action이:

```javascript
{
    type: "counter/increment"
}
```

이라면 Reducer는:

```javascript
{
    value: 1
}
```

이라는 **다음 State**를 계산합니다.

---

# 14. Reducer는 누가 호출하는가?

Redux를 처음 배울 때 특히 중요한 부분입니다.

Component가 Reducer를 직접 호출하는 것이 아닙니다.

```text
잘못된 이해

Component
    ↓
 Reducer
```

실제 개념적 흐름은 다음과 같습니다.

```text
Component
    ↓
dispatch(action)
    ↓
  Store
    ↓
 Reducer
```

Store가 현재 State와 Action을 Reducer에게 전달합니다.

```text
                 current state
                      │
                      ↓
Store ─────────────→ Reducer
                      ↑
                      │
                    action
```

Reducer가 새로운 State를 반환하면 Store가 그것을 새로운 State로 보관합니다.

```text
Reducer
   │
   │ new state
   ↓
Store
   │
   └── State 갱신
```

따라서 **Redux 흐름의 중심은 Store**입니다.

---

# 15. Reducer와 Immutability

전통적인 Redux Reducer에서는 기존 State 객체를 직접 변경하지 않습니다.

다음과 같은 코드는 사용하지 않습니다.

```javascript
function reducer(state, action) {
    state.value++;

    return state;
}
```

대신 새로운 객체를 만들어 반환합니다.

```javascript
function reducer(state, action) {
    return {
        ...state,
        value: state.value + 1
    };
}
```

개념적으로:

```text
Old State
    │
    X 직접 수정
    │
    ↓
New State Object 생성
```

이라고 이해할 수 있습니다.

Redux Toolkit에서는 **Immer**를 사용하기 때문에 나중에는 다음처럼 작성할 수 있습니다.

```javascript
state.value++;
```

하지만 이것은 실제 Redux State를 무작정 직접 변경한다는 의미가 아닙니다.

이 부분은 Redux Toolkit에서 자세히 다룹니다.

지금은 다음 원칙에 집중하면 됩니다.

> **Reducer는 현재 State와 Action을 받아 다음 State를 계산한다.**

---

# 16. Redux 전체 Data Flow

이제 지금까지 배운 내용을 하나의 흐름으로 연결해봅시다.

초기 State:

```javascript
{
    value: 0
}
```

사용자가 `+1` 버튼을 클릭했다고 가정합니다.

```text
① User Event
      │
      │ Click
      ↓

② React Component
      │
      │ increment()
      ↓

③ Action Creator
      │
      │ returns
      ↓

④ Action
      │
      │ { type: "counter/increment" }
      ↓

⑤ dispatch(action)
      │
      ↓

⑥ Redux Store
      │
      │ current state + action
      ↓

⑦ Reducer
      │
      │ calculates
      ↓

⑧ New State
      │
      │ { value: 1 }
      ↓

⑨ Redux Store
      │
      │ State 갱신
      ↓

⑩ React
      │
      ↓

⑪ UI Re-render
```

이것이 Redux를 이해하는 데 가장 중요한 흐름입니다.

---

# 17. 코드로 연결해보기

먼저 초기 State와 Reducer를 정의합니다.

```javascript
const initialState = {
    count: 0
};

function reducer(state = initialState, action) {

    switch (action.type) {

        case "counter/increment":
            return {
                ...state,
                count: state.count + 1
            };

        case "counter/decrement":
            return {
                ...state,
                count: state.count - 1
            };

        default:
            return state;
    }
}
```

Action Creator:

```javascript
function increment() {
    return {
        type: "counter/increment"
    };
}
```

그리고 다음과 같이 dispatch합니다.

```javascript
store.dispatch(increment());
```

이 한 줄 안에서는 개념적으로 다음 일이 일어납니다.

```javascript
const action = increment();

store.dispatch(action);
```

`increment()`가 반환하는 Action은:

```javascript
{
    type: "counter/increment"
}
```

입니다.

Store는 현재 State와 Action을 Reducer에 전달합니다.

```javascript
const newState = reducer(currentState, action);
```

결국:

```text
currentState
{ count: 0 }

     +

action
{ type: "counter/increment" }

     ↓

reducer()

     ↓

newState
{ count: 1 }
```

이라는 흐름이 만들어집니다.

---

# 18. Redux의 단방향 데이터 흐름

Redux의 중요한 특징 중 하나는 **Unidirectional Data Flow**, 즉 **단방향 데이터 흐름**입니다.

```text
┌─────────────┐
│     UI      │
└──────┬──────┘
       │
       │ Action
       ↓
┌─────────────┐
│    Store    │
└──────┬──────┘
       │
       │ Reducer
       ↓
┌─────────────┐
│  New State  │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│     UI      │
└─────────────┘
```

State 변경 경로를 일정한 방향으로 제한하기 때문에 **상태가 어떤 과정을 통해 변경되었는지 이해하고 추적하기 쉬워집니다.**

---

# 19. Redux의 세 가지 기본 원칙

Redux의 구조는 전통적으로 세 가지 기본 원칙으로 설명할 수 있습니다.

## 19.1 Single Source of Truth

Redux에서 관리하는 애플리케이션 상태는 하나의 Store 안에서 관리됩니다.

```text
Redux Store
│
└── State
     ├── user
     ├── cart
     └── filter
```

하지만 이것이 **모든 React State를 Redux에 넣어야 한다는 뜻은 아닙니다.**

컴포넌트 내부에서만 사용하는 상태는 여전히 `useState()` 등으로 관리하는 것이 자연스럽습니다.

---

## 19.2 State is Read-Only

애플리케이션 코드에서 Redux State를 임의로 직접 변경하지 않습니다.

상태 변경이 필요한 사건을 **Action으로 표현하여 dispatch**합니다.

```text
state.count++       ❌

dispatch(action)    ✅
```

---

## 19.3 Changes are Made with Reducers

State가 어떻게 변경될지는 Reducer가 결정합니다.

```text
Current State
      +
    Action
      ↓
   Reducer
      ↓
  New State
```

이 세 가지 원칙이 Redux의 **예측 가능한 상태 관리 구조**를 만듭니다.

---

# 20. Redux가 해결하려는 진짜 문제

Redux를 다음과 같이 정의하는 것은 너무 단순합니다.

> Redux는 전역 State를 사용하기 위한 라이브러리다.

Redux의 더 중요한 가치는 **상태 변경 흐름의 체계화와 예측 가능성**에 있습니다.

상태를 어디에서나 직접 변경할 수 있다면:

```text
Component A ─────→ State
Component B ─────→ State
Component C ─────→ State
Function D  ─────→ State
Timer       ─────→ State
```

상태 변경의 원인을 추적하기 어려워질 수 있습니다.

Redux는 이러한 변경을 하나의 일관된 흐름으로 모읍니다.

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
```

따라서 Redux의 핵심을 한 문장으로 정의하면 다음과 같습니다.

> **Redux는 공유 State를 단순히 중앙에 저장하는 것이 아니라, State가 변경되는 경로를 Action → dispatch → Store → Reducer라는 일관된 흐름으로 통제하여 상태 변화를 예측하고 추적하기 쉽게 만드는 상태 관리 라이브러리입니다.**

---

# 21. 그런데 Vanilla Redux는 코드가 많다

Redux의 구조는 명확하지만 전통적인 방식으로 직접 작성하면 반복 코드가 많아집니다.

Action Type:

```javascript
const INCREMENT = "counter/increment";
const DECREMENT = "counter/decrement";
```

Action Creator:

```javascript
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
```

Reducer:

```javascript
function counterReducer(state = initialState, action) {

    switch (action.type) {

        case INCREMENT:
            return {
                ...state,
                count: state.count + 1
            };

        case DECREMENT:
            return {
                ...state,
                count: state.count - 1
            };

        default:
            return state;
    }
}
```

특히 중첩 객체의 불변성을 직접 관리하면 코드가 더욱 복잡해질 수 있습니다.

```javascript
return {
    ...state,

    user: {
        ...state.user,

        profile: {
            ...state.user.profile,
            name: action.payload
        }
    }
};
```

Redux의 핵심 개념이 복잡하다기보다 **이러한 반복적인 작성 방식이 Redux 코드를 장황하게 만들었습니다.**

---

# 22. Redux Toolkit의 등장

이러한 문제를 해결하기 위해 현대 Redux에서는 **Redux Toolkit(RTK)**을 표준적인 Redux 작성 방식으로 사용합니다.

중요한 것은:

```text
Redux Toolkit
      ≠
Redux와 다른 상태 관리 원리
```

라는 것입니다.

Redux Toolkit은 기존 Redux의 핵심 구조 위에 다음 기능을 제공합니다.

```text
Redux의 핵심 원리
       +
반복 코드 감소
       +
안전한 기본 설정
       +
편리한 API
```

따라서 Redux Toolkit을 사용하더라도 핵심 데이터 흐름은 그대로입니다.

```text
Component
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
Component
```

달라지는 것은 **이 구조를 얼마나 편리하게 작성하느냐**입니다.

---

# 23. Vanilla Redux vs Redux Toolkit

Vanilla Redux에서는:

```javascript
const INCREMENT = "counter/increment";

function increment() {
    return {
        type: INCREMENT
    };
}

function counterReducer(state = initialState, action) {

    switch (action.type) {

        case INCREMENT:
            return {
                ...state,
                value: state.value + 1
            };

        default:
            return state;
    }
}
```

Redux Toolkit에서는 나중에 다음과 같이 작성할 수 있습니다.

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

코드는 크게 줄어들지만 Redux의 개념이 사라진 것은 아닙니다.

`createSlice()`가 내부적으로 필요한 **Reducer 로직과 Action Creator를 구성**해주는 것입니다.

따라서 Redux Toolkit을 제대로 이해하려면 먼저 다음 관계를 이해해야 합니다.

```text
State
Action
Action Creator
dispatch()
Reducer
Store
```

---

# 24. 반드시 구분해야 할 개념

## Store vs State

```text
Store
  =
State와 Redux 처리 흐름을 관리하는 객체

State
  =
Store가 관리하는 데이터
```

---

## Action vs Action Creator

```text
Action
  =
발생한 일을 표현하는 JavaScript 객체

Action Creator
  =
Action을 만들어 반환하는 함수
```

---

## dispatch() vs Reducer

```text
dispatch()
  =
Action을 Store의 처리 흐름으로 전달

Reducer
  =
현재 State와 Action으로 다음 State 계산
```

---

## Redux vs Redux Toolkit

```text
Redux
  =
상태 관리 원리와 핵심 라이브러리

Redux Toolkit
  =
Redux를 현대적인 방식으로
편리하게 작성하기 위한 공식 도구 모음
```

---

# 25. Redux Fundamentals 최종 구조

Redux를 처음 배울 때 가장 먼저 머릿속에 만들어야 하는 그림은 다음과 같습니다.

```text
┌───────────────────────────┐
│      React Component      │
└─────────────┬─────────────┘
              │
              │ User Event
              ↓
┌───────────────────────────┐
│       Action Creator      │
└─────────────┬─────────────┘
              │
              │ Action
              ↓
┌───────────────────────────┐
│        dispatch()         │
└─────────────┬─────────────┘
              │
              ↓
┌───────────────────────────┐
│        Redux Store        │
│                           │
│       Current State       │
└─────────────┬─────────────┘
              │
              │ state + action
              ↓
┌───────────────────────────┐
│          Reducer          │
│                           │
│      (state, action)      │
│             ↓             │
│         new state         │
└─────────────┬─────────────┘
              │
              ↓
┌───────────────────────────┐
│        Redux Store        │
│                           │
│         New State         │
└─────────────┬─────────────┘
              │
              ↓
┌───────────────────────────┐
│      React Component      │
│         Re-render         │
└───────────────────────────┘
```

---

# 26. 최종 정리

Redux의 전체 동작을 한 문장으로 설명할 수 있어야 합니다.

> **React Component에서 상태 변경이 필요한 사건이 발생하면 Action을 `dispatch()`하고, Redux Store는 현재 State와 Action을 Reducer에 전달하여 다음 State를 계산한 뒤 새로운 State를 저장하며, React는 변경된 Redux State를 기반으로 필요한 UI를 다시 렌더링합니다.**

그리고 다음 흐름을 기억합니다.

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

이 흐름은 Redux Toolkit을 사용하더라도 사라지지 않습니다.

---

# 다음 단계 — Redux Toolkit

이제 Redux의 기본 구조를 이해했으므로 다음 단계에서는 이를 실제 React 애플리케이션에 연결합니다.

```text
Redux Fundamentals
        ↓
Redux Toolkit
        ↓
createSlice()
        ↓
Slice
        ↓
Immer
        ↓
configureStore()
        ↓
Provider
        ↓
useSelector()
        ↓
useDispatch()
        ↓
React + Redux Toolkit
```

여기서 중요한 것은 새로운 개념을 처음부터 다시 배우는 것이 아닙니다.

Redux Fundamentals에서 배운 개념들이 Redux Toolkit에서 **어떻게 구현되고 자동화되는지를 연결하는 것**입니다.

```text
Redux Fundamentals          Redux Toolkit

Action Creator       ───→   createSlice()가 생성
Reducer              ───→   createSlice()가 구성
Immutability         ───→   Immer가 지원
Store                ───→   configureStore()
State 조회           ───→   useSelector()
dispatch             ───→   useDispatch()
React 연결           ───→   Provider
```

즉,

> **Redux Toolkit을 배우는 것은 Redux를 버리고 새로운 상태 관리 방식을 배우는 것이 아니라, 지금까지 배운 Redux의 원리를 더 편리하고 안전하게 사용하는 방법을 배우는 것입니다.**
