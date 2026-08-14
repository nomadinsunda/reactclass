# PART 2. Redux Fundamentals

Redux Toolkit을 제대로 이해하려면 먼저 Redux가 어떤 문제를 해결하기 위해 등장했으며, Redux 내부에서 상태가 어떤 흐름으로 변경되는지를 이해해야 합니다.

Redux Toolkit의 `createSlice()`, `configureStore()` 같은 API부터 외우기 시작하면 `dispatch`, `action`, `reducer`, `store`의 관계가 모호해지기 쉽습니다.

따라서 이 PART에서는 Redux Toolkit을 사용하기 전에 **Redux의 핵심 동작 원리**부터 이해합니다.

---

# 1. React의 State

React 애플리케이션에서 화면에 표시되는 데이터는 일반적으로 컴포넌트의 State로 관리할 수 있습니다.

```javascript
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

이 경우 `count`는 `Counter` 컴포넌트가 관리하는 State입니다.

```text
Counter Component
│
├── State
│     └── count = 0
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

이 정도의 애플리케이션에서는 Redux가 필요하지 않습니다.

---

# 2. React State의 범위

중요한 것은 React의 State가 기본적으로 **해당 State를 선언한 컴포넌트에 소속된다**는 것입니다.

```javascript
function App() {

    const [user, setUser] = useState(null);

    return (
        <>
            <Header />
            <Main />
            <Sidebar />
        </>
    );
}
```

`user`는 `App`이 관리합니다.

그런데 다음 컴포넌트들이 모두 사용자 정보를 필요로 한다고 가정해봅시다.

```text
App
│
├── Header
│     └── 사용자 이름
│
├── Main
│     └── 사용자 정보
│
└── Sidebar
      └── 사용자 권한
```

그러면 상위 컴포넌트에서 하위 컴포넌트로 데이터를 전달해야 합니다.

```jsx
<Header user={user} />
<Main user={user} />
<Sidebar user={user} />
```

애플리케이션이 작다면 문제가 되지 않습니다.

하지만 컴포넌트 구조가 깊어지면 문제가 발생할 수 있습니다.

---

# 3. Props Drilling

다음 구조를 생각해봅시다.

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

`App`이 가지고 있는 `user`를 `UserProfile`에서 사용해야 한다면 중간 컴포넌트들이 `user`를 사용하지 않더라도 전달해야 할 수 있습니다.

```jsx
<Layout user={user}>
```

```jsx
<Main user={user}>
```

```jsx
<Content user={user}>
```

```jsx
<UserProfile user={user}>
```

즉,

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

와 같은 구조가 됩니다.

이러한 상황을 흔히 **Props Drilling**이라고 합니다.

Props Drilling 자체가 잘못된 것은 아닙니다.

문제는 애플리케이션이 커지고 여러 컴포넌트가 동일한 상태를 공유하면서 상태의 전달과 변경 흐름이 복잡해질 수 있다는 것입니다.

---

# 4. 공유 상태(Shared State)

쇼핑몰 애플리케이션을 생각해봅시다.

여러 컴포넌트에서 다음과 같은 상태가 필요할 수 있습니다.

```text
로그인 사용자
장바구니
UI 설정
검색/필터 조건
```

예를 들어 장바구니 상태는 다음 컴포넌트에서 동시에 필요할 수 있습니다.

```text
Header
 └── 장바구니 상품 개수

ProductDetail
 └── 장바구니 추가

CartPage
 └── 장바구니 목록

CheckoutPage
 └── 주문 상품
```

이처럼 여러 컴포넌트가 함께 사용하는 상태를 **Shared State**라고 생각할 수 있습니다.

애플리케이션이 커질수록 다음 문제가 중요해집니다.

> 공유 상태를 어디에 보관할 것인가?

> 상태를 누가 변경할 수 있는가?

> 상태가 어떻게 변경되었는가?

> 상태가 변경되면 어떤 컴포넌트가 영향을 받는가?

Redux는 이러한 상태 관리 문제를 체계적으로 해결하기 위해 사용할 수 있는 도구입니다.

---

# 5. Redux란?

Redux는 JavaScript 애플리케이션의 **예측 가능한 상태 관리(Predictable State Management)**를 위한 라이브러리입니다.

Redux의 핵심 아이디어는 단순합니다.

애플리케이션에서 공유해야 하는 상태를 중앙의 **Store**에서 관리합니다.

```text
             Redux Store
          ┌───────────────┐
          │     State     │
          │               │
          │ user          │
          │ cart          │
          │ filter        │
          └───────────────┘
             ↑         ↓
             │         │
       상태 변경 요청   상태 조회
             │         │
       React Components
```

하지만 Redux는 단순히 "전역 변수를 만들어주는 라이브러리"가 아닙니다.

Redux에서 중요한 것은 **상태 변경 절차를 명확하게 통제한다는 것**입니다.

---

# 6. Redux를 전역 변수로 이해하면 안 되는 이유

다음과 같은 전역 객체가 있다고 생각해봅시다.

```javascript
const state = {
    count: 0
};
```

어디에서든 다음과 같이 변경할 수 있다면:

```javascript
state.count++;
```

애플리케이션 규모가 커졌을 때 문제가 발생합니다.

```text
Component A ──┐
Component B ──┼──→ state.count 직접 변경
Component C ──┤
Function D  ──┘
```

`count`가 예상하지 못한 값이 되었을 때 어떤 코드가 상태를 변경했는지 추적하기 어려워집니다.

Redux는 이런 방식으로 상태를 관리하지 않습니다.

Redux에서는 상태 변경에 **정해진 흐름**이 존재합니다.

```text
상태를 직접 변경

Component ───────────X──────────→ State


Redux 방식

Component
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
```

이 흐름이 Redux를 이해하는 핵심입니다.

---

# 7. Redux의 핵심 구성 요소

Redux를 이해하려면 다음 개념의 관계를 이해해야 합니다.

```text
Store
State
Action
Action Creator
dispatch
Reducer
```

각각을 따로 암기하기보다 하나의 데이터 흐름으로 이해해야 합니다.

전체 구조를 먼저 보면 다음과 같습니다.

```text
          ┌──────────────────┐
          │ React Component  │
          └────────┬─────────┘
                   │
                   │ dispatch(action)
                   ↓
          ┌──────────────────┐
          │      Store       │
          └────────┬─────────┘
                   │
                   │ current state
                   │ + action
                   ↓
          ┌──────────────────┐
          │     Reducer      │
          └────────┬─────────┘
                   │
                   │ new state
                   ↓
          ┌──────────────────┐
          │      Store       │
          │   New State      │
          └────────┬─────────┘
                   │
                   ↓
          ┌──────────────────┐
          │ React Component  │
          └──────────────────┘
```

이제 각각을 살펴보겠습니다.

---

# 8. State

State는 **현재 애플리케이션의 상태를 표현하는 데이터**입니다.

예를 들어 쇼핑몰의 Redux State가 다음과 같다고 생각할 수 있습니다.

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

개념적으로는:

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

Redux Store는 이러한 State를 보관합니다.

---

# 9. Store

**Store는 Redux 상태 관리의 중심 객체입니다.**

Store는 단순히 State만 저장하는 객체가 아닙니다.

Store는 Redux의 전체 상태 관리 흐름을 관리합니다.

개념적으로 Store는 다음 역할을 수행합니다.

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

따라서 다음과 같이 이해하는 것이 좋습니다.

```text
Store ≠ State

Store
 └── State를 관리하는 객체
```

State는 **데이터**이고 Store는 그 데이터를 **관리하는 Redux 객체**입니다.

---

# 10. Action

Redux에서는 Component가 State를 직접 변경하지 않습니다.

대신 다음과 같은 객체를 Redux에게 전달합니다.

```javascript
{
    type: "counter/increment"
}
```

이 객체를 **Action**이라고 합니다.

Action은 쉽게 말하면:

> "어떤 일이 발생했다."

또는

> "이러한 상태 변경이 필요하다."

라는 정보를 표현하는 일반 JavaScript 객체입니다.

예를 들어:

```javascript
{
    type: "cart/addItem",
    payload: {
        id: 10,
        name: "Keyboard"
    }
}
```

여기서:

```text
type
```

은 어떤 종류의 Action인지 나타냅니다.

```text
payload
```

는 Action과 함께 전달할 추가 데이터를 담는 데 일반적으로 사용합니다.

---

# 11. Action은 명령이라기보다 사건을 표현한다

Action을 처음 배울 때 다음과 같이 생각하기 쉽습니다.

```text
"count를 증가시켜라."
```

하지만 Redux의 Action은 보통 **무슨 일이 발생했는지를 표현하는 데이터**라고 이해하는 것이 좋습니다.

예:

```javascript
{
    type: "cart/itemAdded",
    payload: product
}
```

즉,

```text
Action
=
상태 변경과 관련하여 발생한 일을 설명하는 데이터
```

입니다.

실제 State를 어떻게 변경할지는 **Reducer가 결정합니다.**

---

# 12. Action Creator

매번 Action 객체를 직접 작성할 수도 있습니다.

```javascript
dispatch({
    type: "counter/increment"
});
```

하지만 Action 객체를 생성하는 함수를 만들 수도 있습니다.

```javascript
function increment() {
    return {
        type: "counter/increment"
    };
}
```

이러한 함수를 **Action Creator**라고 합니다.

```text
Action Creator
      ↓
Action 생성
```

즉:

```javascript
const action = increment();
```

결과:

```javascript
{
    type: "counter/increment"
}
```

중요한 차이입니다.

```text
increment
    ↓
Action Creator

increment()
    ↓
Action 반환

{
   type: "counter/increment"
}
    ↓
Action
```

Redux Toolkit에서는 나중에 배울 `createSlice()`가 이러한 Action Creator를 자동으로 만들어줍니다.

---

# 13. dispatch()

Action을 만들었다고 Redux가 자동으로 처리하는 것은 아닙니다.

Action을 Store에 전달해야 합니다.

이때 사용하는 것이:

```javascript
dispatch()
```

입니다.

예:

```javascript
store.dispatch({
    type: "counter/increment"
});
```

또는 Action Creator를 사용하면:

```javascript
store.dispatch(increment());
```

입니다.

이를 분해해서 보면:

```javascript
const action = increment();

store.dispatch(action);
```

입니다.

따라서 `dispatch()`의 핵심 역할은:

> Action을 Redux Store의 처리 흐름으로 전달하는 것

입니다.

```text
Component
    │
    │ Action 생성
    ↓
Action
    │
    │ dispatch(action)
    ↓
Store
```

`dispatch()`가 State를 직접 변경하는 것은 아닙니다.

---

# 14. Reducer

Store가 Action을 받으면 **Reducer**를 통해 다음 State를 계산합니다.

Reducer는 개념적으로 다음 형태의 함수입니다.

```javascript
function reducer(state, action) {

    // ...

    return newState;
}
```

즉:

```text
Reducer

(Current State, Action)
          ↓
      New State
```

수학적인 함수처럼 표현하면:

```text
f(state, action) → newState
```

입니다.

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

이고 다음 Action이 전달되었다면:

```javascript
{
    type: "counter/increment"
}
```

Reducer는:

```javascript
{
    value: 1
}
```

이라는 다음 State를 계산합니다.

---

# 15. Reducer는 Store가 호출한다

이 부분은 매우 중요합니다.

Component가 Reducer를 직접 호출하는 구조가 아닙니다.

잘못된 이해:

```text
Component
    ↓
Reducer
```

Redux의 실제 개념적 흐름:

```text
Component
    ↓
dispatch(action)
    ↓
Store
    ↓
Reducer
```

즉 Store가 Redux 흐름의 중심입니다.

Store가 현재 State와 Action을 Reducer에 전달합니다.

```text
                 current state
                       │
                       ↓
Store ───────────→ Reducer
                       ↑
                       │
                     action
```

Reducer가 다음 State를 반환하면 Store가 그것을 새로운 State로 보관합니다.

```text
Reducer
   │
   │ new state
   ↓
Store
   │
   └── State 갱신
```

---

# 16. Reducer와 불변성(Immutability)

전통적인 Redux Reducer에서는 기존 State 객체를 직접 변경하면 안 됩니다.

다음 코드는 잘못된 방식입니다.

```javascript
function reducer(state, action) {

    state.value++;

    return state;
}
```

기존 객체를 직접 변경하고 있기 때문입니다.

전통적인 Redux에서는 새로운 객체를 만들어 반환합니다.

```javascript
function reducer(state, action) {

    return {
        ...state,
        value: state.value + 1
    };
}
```

즉:

```text
Old State
   │
   X 직접 수정
   │
   ↓
New State Object 생성
```

Redux Toolkit에서는 이후 **Immer**를 통해 이 부분을 훨씬 편하게 작성할 수 있습니다.

따라서 지금은 다음 원칙만 기억하면 됩니다.

> Reducer는 현재 State와 Action을 받아 다음 State를 계산한다.

---

# 17. Redux의 전체 Data Flow

이제 지금까지의 개념을 하나로 연결해봅시다.

초기 상태:

```javascript
{
    value: 0
}
```

사용자가 버튼을 클릭합니다.

```text
1. User
   │
   │ Click
   ↓

2. React Component
   │
   │ increment()
   ↓

3. Action Creator
   │
   │ returns
   ↓

4. Action
   │
   │ { type: "counter/increment" }
   ↓

5. dispatch(action)
   │
   ↓

6. Redux Store
   │
   │ current state + action
   ↓

7. Reducer
   │
   │ calculates
   ↓

8. New State
   │
   │ { value: 1 }
   ↓

9. Redux Store
   │
   │ state updated
   ↓

10. React
    │
    ↓

11. UI Re-render
```

이것이 Redux의 가장 중요한 흐름입니다.

---

# 18. 코드로 전체 흐름 이해하기

Redux Toolkit을 사용하기 전에 개념적인 Redux 코드를 살펴보겠습니다.

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

호출:

```javascript
store.dispatch(increment());
```

이를 내부 흐름으로 풀어보면:

```javascript
const action = increment();

store.dispatch(action);
```

Action은:

```javascript
{
    type: "counter/increment"
}
```

Store는 개념적으로:

```javascript
const newState = reducer(currentState, action);
```

와 같은 작업을 수행합니다.

그리고:

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

이 됩니다.

---

# 19. Redux의 단방향 데이터 흐름

Redux의 중요한 특징 중 하나는 **Unidirectional Data Flow**, 즉 단방향 데이터 흐름입니다.

```text
          ┌─────────────┐
          │     UI      │
          └──────┬──────┘
                 │
              Action
                 │
                 ↓
          ┌─────────────┐
          │    Store    │
          └──────┬──────┘
                 │
              Reducer
                 │
                 ↓
          ┌─────────────┐
          │ New State   │
          └──────┬──────┘
                 │
                 ↓
          ┌─────────────┐
          │     UI      │
          └─────────────┘
```

상태 변경 흐름을 한 방향으로 제한함으로써 상태가 어떻게 변경되었는지를 이해하고 추적하기 쉬워집니다.

---

# 20. Redux의 세 가지 기본 원칙

전통적으로 Redux는 세 가지 기본 원칙으로 설명할 수 있습니다.

## 20.1 Single Source of Truth

애플리케이션의 Redux 상태는 하나의 Store 안에서 관리됩니다.

```text
Redux Store
│
└── State
     ├── user
     ├── cart
     └── filter
```

이 말이 모든 React State를 무조건 Redux Store에 넣으라는 뜻은 아닙니다.

컴포넌트 내부에서만 필요한 상태는 여전히 `useState()` 등으로 관리하는 것이 자연스럽습니다.

---

## 20.2 State is Read-Only

애플리케이션 코드가 Redux State를 임의로 직접 변경하는 것이 아니라 **Action을 dispatch하여 상태 변경을 요청**합니다.

```text
Component

state.count++       X

dispatch(action)    O
```

---

## 20.3 Changes are Made with Reducers

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

이 세 원칙이 Redux의 예측 가능한 상태 관리 구조를 만듭니다.

---

# 21. Redux에서 각 구성 요소의 책임

Redux를 이해할 때 각 요소의 책임을 분리하면 훨씬 쉽습니다.

| 구성 요소          | 책임                          |
| -------------- | --------------------------- |
| State          | 현재 애플리케이션 상태                |
| Store          | State와 Redux 처리 흐름 관리       |
| Action         | 발생한 일을 표현하는 데이터             |
| Action Creator | Action 객체 생성                |
| `dispatch()`   | Action을 Store에 전달           |
| Reducer        | State와 Action으로 다음 State 계산 |

이를 한 문장으로 연결하면 다음과 같습니다.

> Component에서 Action을 만들고 `dispatch()`를 통해 Store에 전달하면, Store는 Reducer를 이용하여 현재 State와 Action으로 다음 State를 계산하고 새로운 State를 저장한다.

이 문장을 이해하면 Redux의 핵심을 이해한 것입니다.

---

# 22. Redux가 해결하는 진짜 문제

Redux를 단순히:

> 전역 State를 사용하기 위한 라이브러리

라고 정의하면 Redux의 핵심을 놓치게 됩니다.

Redux가 제공하는 중요한 가치는 **상태 변경 흐름의 체계화와 예측 가능성**입니다.

Redux가 없다면 상태 변경이 다음처럼 흩어질 수 있습니다.

```text
Component A ──────→ State
Component B ──────→ State
Component C ──────→ State
Function D  ──────→ State
Timer       ──────→ State
```

Redux에서는 이를:

```text
             Action
               ↓
             Store
               ↓
             Reducer
               ↓
              State
```

라는 일관된 흐름으로 관리합니다.

---

# 23. 그런데 Redux를 직접 사용하면 코드가 많다

Redux의 구조는 명확하지만 직접 구현하다 보면 반복 코드가 많아집니다.

예를 들어 Action Type:

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

상태가 많아지면 이러한 코드가 계속 증가합니다.

또한 불변성을 지키기 위한 코드도 반복됩니다.

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

Redux의 개념 자체보다 이러한 반복 코드가 개발을 복잡하게 만들었습니다.

---

# 24. Redux Toolkit의 등장

이 문제를 해결하기 위해 현재 Redux에서는 **Redux Toolkit(RTK)**을 표준적인 Redux 작성 방식으로 사용합니다.

Redux Toolkit은 Redux를 없앤 새로운 상태 관리 라이브러리가 아닙니다.

중요합니다.

```text
Redux Toolkit
      ≠
Redux와 다른 상태 관리 원리
```

Redux Toolkit은:

```text
Redux의 핵심 원리
      +
반복 코드 감소
      +
안전한 기본 설정
      +
편리한 API
```

를 제공하는 도구입니다.

따라서 Redux Toolkit을 사용해도 내부의 기본 흐름은 그대로입니다.

```text
Component
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
Component
```

달라지는 것은 이 구조를 **얼마나 편리하게 작성하느냐**입니다.

---

# 25. Vanilla Redux와 Redux Toolkit 비교

기존 방식에서는:

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

Redux Toolkit에서는 나중에 다음과 같은 형태로 작성할 수 있습니다.

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

코드는 크게 줄어들지만 내부 개념은 사라지지 않습니다.

`createSlice()`는 자동으로 필요한 Action Creator와 Reducer 로직을 구성합니다.

즉 Redux Toolkit을 제대로 이해하려면 결국:

```text
State
Action
Action Creator
dispatch
Reducer
Store
```

를 이해해야 합니다.

---

# 26. PART 1 핵심 구조

Redux를 처음 접했을 때 가장 먼저 머릿속에 만들어야 하는 그림은 이것입니다.

```text
┌───────────────────────────┐
│      React Component      │
└─────────────┬─────────────┘
              │
              │ User Event
              ↓
┌───────────────────────────┐
│      Action Creator       │
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
│ (state, action)           │
│          ↓                │
│      new state            │
└─────────────┬─────────────┘
              │
              ↓
┌───────────────────────────┐
│        Redux Store        │
│                           │
│        New State          │
└─────────────┬─────────────┘
              │
              ↓
┌───────────────────────────┐
│      React Component      │
│         Re-render         │
└───────────────────────────┘
```

---

# 27. 반드시 구분해야 할 것

## Store와 State

```text
Store
=
State를 관리하는 Redux 객체

State
=
Store가 관리하는 데이터
```

---

## Action과 Action Creator

```text
Action
=
JavaScript 객체

Action Creator
=
Action을 만들어 반환하는 함수
```

---

## dispatch와 Reducer

```text
dispatch
=
Action을 Store의 처리 흐름으로 전달

Reducer
=
현재 State와 Action으로 다음 State 계산
```

---

## Redux와 Redux Toolkit

```text
Redux
=
상태 관리 원리와 핵심 라이브러리

Redux Toolkit
=
Redux를 현대적인 방식으로 편리하게 작성하기 위한 공식 도구 모음
```

---

# 28. PART 1 최종 정리

Redux의 전체 동작을 한 문장으로 설명할 수 있어야 합니다.

> React Component에서 상태 변경이 필요한 사건이 발생하면 Action을 `dispatch()`하고, Redux Store는 현재 State와 Action을 Reducer에 전달하여 다음 State를 계산한 뒤 새로운 State를 저장하며, React는 변경된 Redux State를 기반으로 필요한 UI를 다시 렌더링한다.

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

이 흐름은 이후 Redux Toolkit을 사용하더라도 바뀌지 않습니다.

---

# 다음 단계

PART 2에서는 이 Redux 구조를 실제 React 애플리케이션에 적용하면서 다음 내용을 학습합니다.

```text
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

PART 1에서 배운 `State`, `Action`, `Action Creator`, `dispatch`, `Reducer`, `Store`가 Redux Toolkit에서 각각 어떻게 구현되고 자동화되는지를 연결하는 것이 PART 2의 핵심입니다.
