# PART 3. `useSelector()` & `useDispatch()`

## React 컴포넌트는 Store를 어떻게 읽고 변경하는가?

PART 2에서는 React 컴포넌트가 Redux Store의 위치를 어떻게 알 수 있는지 살펴봤습니다.

핵심 구조는 다음과 같습니다.

```text
Redux Store
     │
     ▼
 <Provider>
     │
     │ React Redux Context
     ▼
React Component
```

`<Provider>`가 Redux Store를 React Redux Context를 통해 하위 컴포넌트 트리에 제공하기 때문에, 깊은 곳에 있는 컴포넌트도 Store에 접근할 수 있습니다.

이제 다음 문제가 남았습니다.

> **Store에 접근할 수 있다는 것은 알겠는데, 컴포넌트는 실제로 State를 어떻게 읽고 Action을 어떻게 전달할까?**

React Redux는 이를 위해 두 개의 핵심 Hook을 제공합니다.

```text
useSelector()
     ↓
Redux Store → React Component

useDispatch()
     ↓
React Component → Redux Store
```

이번 PART에서는 이 두 Hook을 중심으로 **React와 Redux Store 사이의 실제 데이터 흐름**을 완성합니다.

---

# 1. 먼저 전체 그림부터 보자

React Redux에서 가장 중요한 두 방향은 다음과 같습니다.

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

`useSelector()`와 `useDispatch()`의 역할은 서로 반대 방향입니다.

| Hook            | 방향            | 역할                            |
| --------------- | ------------- | ----------------------------- |
| `useSelector()` | Redux → React | Store의 State에서 필요한 값을 선택하고 구독 |
| `useDispatch()` | React → Redux | Action을 Store로 전달             |

예를 들어 다음 `Counter` 컴포넌트를 생각해봅시다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  const dispatch = useDispatch();

  return (
    <>
      <h1>{count}</h1>

      <button onClick={() => dispatch(increment())}>
        증가
      </button>
    </>
  );
}
```

이 짧은 코드 안에서 양방향 연결이 모두 일어납니다.

```text
                Counter
               /       \
              /         \
     useSelector()   useDispatch()
          │               │
          │ State         │ Action
          ▼               ▼
              Redux Store
```

이제 각각을 자세히 살펴보겠습니다.

---

# 2. `useSelector()`란?

`useSelector()`는 React Redux가 제공하는 Hook입니다.

```jsx
import { useSelector } from 'react-redux';
```

가장 기본적인 사용법은 다음과 같습니다.

```jsx
const count = useSelector(
  state => state.counter.value
);
```

한 문장으로 정의하면:

> **`useSelector()`는 Redux Store의 전체 State에서 현재 컴포넌트가 필요한 값을 선택하고, 그 값의 변경을 구독하는 Hook입니다.**

여기서 중요한 표현은 두 가지입니다.

```text
선택한다
+
구독한다
```

단순히 State를 한 번 읽어오는 함수가 아닙니다.

---

# 3. `selector`란?

다음 코드를 다시 보겠습니다.

```jsx
const count = useSelector(
  state => state.counter.value
);
```

`useSelector()`에 전달한 함수:

```javascript
state => state.counter.value
```

를 **selector 함수**라고 합니다.

Selector는 말 그대로:

> **전체 Redux State에서 필요한 값을 선택하는 함수**

입니다.

예를 들어 Redux State가 다음과 같다고 해보겠습니다.

```javascript
{
  counter: {
    value: 10
  },

  user: {
    name: 'Kim',
    age: 30
  },

  cart: {
    items: [...]
  }
}
```

전체 State를 트리로 표현하면:

```text
Root State
│
├── counter
│    └── value: 10
│
├── user
│    ├── name: "Kim"
│    └── age: 30
│
└── cart
     └── items
```

다음 selector는:

```javascript
state => state.counter.value
```

전체 State에서:

```text
Root State
    │
    └── counter
          │
          └── value
                │
                ▼
                10
```

을 선택합니다.

따라서:

```jsx
const count = useSelector(
  state => state.counter.value
);
```

의 `count`에는 `10`이 들어갑니다.

---

# 4. `useSelector()`의 `state`는 어디서 오는가?

초보자가 자주 궁금해하는 부분입니다.

```javascript
state => state.counter.value
```

여기 있는 `state`를 우리가 직접 전달한 적은 없습니다.

그렇다면 누가 전달하는 것일까요?

`useSelector()`가 현재 Provider에 연결된 Redux Store에 접근하고, 그 Store의 현재 State를 selector 함수에 전달합니다.

개념적으로 보면:

```text
<Provider store={store}>
          │
          ▼
      Redux Store
          │
          │ getState()
          ▼
       Root State
          │
          ▼
       selector
          │
state => state.counter.value
          │
          ▼
         10
```

즉 다음 코드에서:

```jsx
useSelector(state => state.counter.value);
```

`state`는 **현재 Redux Store의 전체 Root State**입니다.

이를 아주 단순화하면 다음과 같이 생각할 수 있습니다.

```javascript
const state = store.getState();

const selectedValue =
  selector(state);
```

실제 React Redux 내부 구현은 이것보다 훨씬 복잡하지만, selector의 역할을 이해하기에는 좋은 개념 모델입니다.

---

# 5. 왜 전체 State가 아니라 필요한 값만 선택하는가?

다음과 같이 전체 State를 가져올 수도 있다고 생각할 수 있습니다.

```jsx
const state = useSelector(state => state);
```

그리고:

```jsx
<h1>{state.counter.value}</h1>
```

처럼 사용할 수도 있습니다.

하지만 일반적으로 이렇게 작성하지 않습니다.

대신:

```jsx
const count = useSelector(
  state => state.counter.value
);
```

처럼 **컴포넌트가 실제로 필요한 값을 선택합니다.**

왜일까요?

`useSelector()`는 선택한 값의 변경 여부를 기준으로 컴포넌트의 리렌더링 필요성을 판단하기 때문입니다.

예를 들어:

```text
Redux State
│
├── counter.value
├── user.name
└── cart.items
```

`Counter` 컴포넌트가 필요한 것은:

```text
counter.value
```

뿐입니다.

따라서:

```jsx
const count = useSelector(
  state => state.counter.value
);
```

처럼 필요한 범위를 선택하는 것이 좋습니다.

핵심 원칙은:

> **컴포넌트가 실제로 필요한 State를 선택한다.**

입니다.

---

# 6. `useSelector()`는 단순히 State를 읽는 Hook이 아니다

여기서 `useSelector()`의 가장 중요한 특징이 등장합니다.

다음 코드를 생각해봅시다.

```jsx
const count = useSelector(
  state => state.counter.value
);
```

최초 렌더링 시:

```text
Redux Store
     │
     ▼
counter.value = 0
     │
     ▼
 selector
     │
     ▼
 count = 0
```

화면에는:

```text
0
```

이 표시됩니다.

그런데 다른 곳에서 다음 코드가 실행됐다고 해보겠습니다.

```javascript
dispatch(increment());
```

Redux Store의 State가:

```text
counter.value

0
↓
1
```

로 변경됩니다.

이때 `useSelector()`는 단순히 최초 렌더링 때 State를 한 번 읽고 끝나는 것이 아닙니다.

**Store의 변경을 구독하고 있기 때문에 State 변경을 감지할 수 있습니다.**

```text
Redux Store
     │
     │ State 변경
     ▼
Subscription
     │
     ▼
selector 다시 실행
```

이것이 `store.getState()`를 직접 호출하는 것과 `useSelector()`의 중요한 차이입니다.

---

# 7. Store Subscription

Redux Store에는 이미 `subscribe()`라는 기능이 있습니다.

```javascript
store.subscribe(listener);
```

Redux Store의 State가 변경되면 등록된 listener에게 변경 사실을 알릴 수 있습니다.

개념적으로:

```text
dispatch(action)
      │
      ▼
   Reducer
      │
      ▼
 New State
      │
      ▼
Redux Store
      │
      │ notify
      ▼
Subscribers
```

React Redux는 이 Store Subscription 메커니즘을 React의 렌더링 시스템과 연결합니다.

따라서 `useSelector()`를 사용하는 컴포넌트는 개념적으로 다음 흐름에 참여합니다.

```text
useSelector()
     │
     ▼
Store Subscription
     │
     ▼
Store Update 감지
     │
     ▼
selector 재실행
```

하지만 여기서 아주 중요한 점이 있습니다.

> **Store의 State가 변경되었다고 해서 `useSelector()`를 사용하는 모든 컴포넌트가 무조건 리렌더링되는 것은 아닙니다.**

---

# 8. Store가 변경되면 selector를 다시 평가한다

다음과 같은 컴포넌트가 있다고 해보겠습니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  return <h1>{count}</h1>;
}
```

현재:

```text
counter.value = 10
```

이라면 selector 결과도:

```text
10
```

입니다.

이후 Redux Store에서 `user.name`만 변경되었다고 해보겠습니다.

```text
Before

counter.value = 10
user.name = "Kim"


After

counter.value = 10
user.name = "Lee"
```

Store의 State 자체는 변경되었습니다.

React Redux는 Store Update를 감지하고 selector를 다시 평가할 수 있습니다.

```text
Store Update
     │
     ▼
selector 재실행
     │
     ▼
state.counter.value
     │
     ▼
     10
```

이전 selector 결과 역시 `10`이었습니다.

```text
Previous = 10
Current  = 10
```

선택한 값이 변하지 않았습니다.

따라서 `Counter`를 Redux Store 변경 때문에 다시 렌더링할 필요가 없습니다.

---

# 9. `useSelector()`와 비교

`useSelector()`는 selector가 반환한 **이전 결과와 새로운 결과를 비교**합니다.

기본적으로 중요한 비교는 reference equality, 즉 `===`입니다.

```text
Store Update
     │
     ▼
selector 실행
     │
     ▼
새로운 selected value
     │
     ▼
이전 결과 === 새로운 결과 ?
        │
     ┌──┴──┐
     │     │
    YES    NO
     │     │
     ▼     ▼
   유지   Re-render
```

예를 들어:

```jsx
const count = useSelector(
  state => state.counter.value
);
```

에서:

```text
이전 결과 = 10
새 결과   = 10

10 === 10
→ true
```

라면 Redux 변경으로 인한 리렌더링은 필요하지 않습니다.

반면:

```text
이전 결과 = 10
새 결과   = 11

10 === 11
→ false
```

이면 컴포넌트가 새로운 값을 반영할 수 있도록 리렌더링이 필요합니다.

따라서 중요한 흐름은:

```text
Redux State 변경
       ↓
모든 컴포넌트 Re-render
```

가 아니라:

```text
Redux State 변경
       ↓
Store Update 알림
       ↓
selector 재평가
       ↓
선택한 값 비교
       ↓
필요한 컴포넌트 Re-render
```

입니다.

> 여기서 말하는 것은 **Redux Store 변경으로 인해 `useSelector()`가 유발하는 리렌더링**입니다. 부모 컴포넌트의 리렌더링 등 다른 React 렌더링 원인까지 막아준다는 뜻은 아닙니다.

---

# 10. 객체를 반환하는 selector는 주의해야 한다

다음 코드를 보겠습니다.

```jsx
const user = useSelector(state => ({
  name: state.user.name,
  age: state.user.age
}));
```

selector가 실행될 때마다 새로운 객체를 생성합니다.

```javascript
{ name: 'Kim', age: 30 }
```

겉으로 보기에 값이 같더라도:

```javascript
const a = { name: 'Kim', age: 30 };
const b = { name: 'Kim', age: 30 };

a === b; // false
```

객체 reference가 다릅니다.

따라서 개념적으로:

```text
Previous
{ name: "Kim", age: 30 }
        │
        │ ===
        ▼
Current
{ name: "Kim", age: 30 }

→ false
```

가 될 수 있습니다.

이 때문에 selector에서 매번 새로운 객체나 배열을 만드는 것은 불필요한 리렌더링의 원인이 될 수 있습니다.

이 문제를 해결하는 방법인:

```text
여러 useSelector()
shallowEqual
memoized selector
```

등은 PART 4에서 자세히 다룹니다.

---

# 11. 여러 개의 `useSelector()`를 사용할 수 있다

컴포넌트에서 여러 State가 필요하다면 `useSelector()`를 여러 번 사용할 수도 있습니다.

```jsx
function UserProfile() {
  const name = useSelector(
    state => state.user.name
  );

  const age = useSelector(
    state => state.user.age
  );

  return (
    <>
      <h1>{name}</h1>
      <p>{age}</p>
    </>
  );
}
```

개념적으로:

```text
Redux Root State
      │
      ├── selector #1
      │      └── user.name
      │
      └── selector #2
             └── user.age
```

각 `useSelector()`가 필요한 값을 명확하게 선택합니다.

초급 단계에서는:

> **컴포넌트가 필요한 값을 가능한 명확하게 선택한다.**

라는 원칙부터 기억하면 충분합니다.

---

# 12. `useDispatch()`란?

이번에는 반대 방향을 살펴보겠습니다.

`useSelector()`가:

```text
Redux → React
```

방향이었다면 `useDispatch()`는:

```text
React → Redux
```

방향입니다.

먼저 import합니다.

```jsx
import { useDispatch } from 'react-redux';
```

그리고:

```jsx
const dispatch = useDispatch();
```

로 사용합니다.

한 문장으로 정의하면:

> **`useDispatch()`는 현재 `<Provider>`가 제공하는 Redux Store의 `dispatch` 함수를 React 컴포넌트에서 사용할 수 있게 해주는 Hook입니다.**

---

# 13. `useDispatch()`는 새로운 dispatch 함수를 만드는 것이 아니다

다음 코드를 보면:

```jsx
const dispatch = useDispatch();
```

React Redux가 새로운 `dispatch` 함수를 만들어 준다고 생각할 수 있습니다.

하지만 Redux Store에는 이미 `dispatch`가 존재합니다.

```javascript
store.dispatch(action);
```

PART 2에서 살펴본 것처럼 React Redux는 Provider와 Context를 통해 현재 Store에 접근할 수 있습니다.

따라서 개념적으로:

```text
<Provider store={store}>
          │
          ▼
React Redux Context
          │
          ▼
    useDispatch()
          │
          ▼
   store.dispatch
```

라고 이해할 수 있습니다.

즉:

```javascript
const dispatch = useDispatch();
```

는 개념적으로:

```text
현재 Provider가 제공하는
Redux Store의 dispatch를 사용한다.
```

라는 의미입니다.

---

# 14. `useDispatch()`로 Action 전달하기

Redux Toolkit의 Slice가 다음과 같다고 해보겠습니다.

```javascript
const counterSlice = createSlice({
  name: 'counter',

  initialState: {
    value: 0
  },

  reducers: {
    increment(state) {
      state.value++;
    }
  }
});

export const { increment } = counterSlice.actions;
```

React 컴포넌트에서는:

```jsx
function Counter() {
  const dispatch = useDispatch();

  const handleClick = () => {
    dispatch(increment());
  };

  return (
    <button onClick={handleClick}>
      증가
    </button>
  );
}
```

사용자가 버튼을 클릭하면:

```text
User Click
    │
    ▼
handleClick()
    │
    ▼
increment()
    │
    ▼
Action
    │
    ▼
dispatch(action)
    │
    ▼
Redux Store
```

이 흐름이 만들어집니다.

---

# 15. Action Creator와 dispatch를 구분하자

다음 코드는 초보자가 자주 혼동합니다.

```javascript
dispatch(increment());
```

여기에는 두 단계가 있습니다.

먼저:

```javascript
increment()
```

가 실행됩니다.

`increment`는 Redux Toolkit이 만들어 준 **Action Creator**입니다.

개념적으로 다음과 같은 Action 객체를 반환합니다.

```javascript
{
  type: 'counter/increment'
}
```

그 다음:

```javascript
dispatch(action);
```

이 실행됩니다.

따라서:

```javascript
dispatch(increment());
```

를 풀어 쓰면 개념적으로:

```javascript
const action = increment();

dispatch(action);
```

입니다.

흐름은:

```text
increment()
    │
    ▼
Action Object
    │
    ▼
dispatch()
    │
    ▼
Redux Store
```

입니다.

즉:

```text
increment()
→ Action 생성

dispatch()
→ Action을 Store에 전달
```

입니다.

---

# 16. `useDispatch()` 자체는 State를 변경하지 않는다

이 부분도 중요합니다.

다음 코드가 있다고 해서:

```javascript
dispatch(increment());
```

`dispatch` 함수가 직접 State를 변경하는 것은 아닙니다.

Redux의 기본 흐름은 이미 배운 것처럼:

```text
dispatch(action)
      │
      ▼
Redux Store
      │
      ▼
Reducer
      │
      ▼
New State
```

입니다.

따라서 React Redux의 `useDispatch()` 역할은 정확히:

```text
React Component
      │
      ▼
Redux Store의 dispatch에 접근
```

하는 것까지입니다.

State를 실제로 계산하는 책임은 Redux 쪽에 있습니다.

---

# 17. `useSelector()`와 `useDispatch()`를 합쳐보자

이제 두 Hook을 하나의 컴포넌트에서 사용해보겠습니다.

```jsx
import {
  useDispatch,
  useSelector
} from 'react-redux';

import { increment } from './counterSlice';

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

      <button onClick={handleClick}>
        증가
      </button>
    </>
  );
}

export default Counter;
```

이제 React와 Redux 사이에 완전한 순환 흐름이 만들어집니다.

---

# 18. 전체 흐름 ① — React → Redux

먼저 사용자가 버튼을 클릭합니다.

```text
User
 │
 │ click
 ▼
Button
 │
 ▼
handleClick()
 │
 ▼
dispatch(increment())
```

`increment()`가 Action을 생성합니다.

```text
increment()
    │
    ▼
{
  type: "counter/increment"
}
```

그 Action이 Store로 전달됩니다.

```text
React Component
      │
      │ dispatch(action)
      ▼
 Redux Store
      │
      ▼
   Reducer
```

이 방향이:

```text
React → Redux
```

입니다.

그리고 이 연결을 담당하는 React Redux Hook이:

```text
useDispatch()
```

입니다.

---

# 19. 전체 흐름 ② — Redux → React

Reducer 처리 결과 Redux State가 변경됩니다.

```text
counter.value

0
↓
1
```

Redux Store는 변경 사실을 subscriber들에게 알립니다.

```text
Redux Store
     │
     │ notify
     ▼
Subscription
```

React Redux는 이 변경을 감지하고 `useSelector()`의 selector를 다시 평가합니다.

```text
Subscription
     │
     ▼
selector 재평가
     │
     ▼
state.counter.value
```

이전 값:

```text
0
```

새로운 값:

```text
1
```

결과가 달라졌으므로 컴포넌트가 새로운 선택 값을 반영하도록 리렌더링됩니다.

```text
Previous = 0
Current  = 1
     │
     ▼
Different
     │
     ▼
Component Re-render
```

이 방향이:

```text
Redux → React
```

입니다.

그리고 이 연결을 담당하는 Hook이:

```text
useSelector()
```

입니다.

---

# 20. React Redux의 전체 데이터 흐름

이제 PART 1부터 배운 내용이 하나로 연결됩니다.

```text
                  User Event
                      │
                      ▼
              React Component
                      │
                      │ useDispatch()
                      ▼
               dispatch(action)
                      │
                      ▼
                Redux Store
                      │
                      ▼
                   Reducer
                      │
                      ▼
                  New State
                      │
                      ▼
               Store Update
                      │
                      ▼
                Subscription
                      │
                      ▼
              selector 재평가
                      │
                      ▼
             선택한 값 비교
                      │
               ┌──────┴──────┐
               │             │
             동일          변경됨
               │             │
               ▼             ▼
             유지       Re-render
                              │
                              ▼
                       React Component
```

이 흐름이 **React Redux의 가장 중요한 전체 흐름**입니다.

---

# 21. Provider까지 포함하면

PART 2에서 배운 Provider와 Context까지 포함하면 전체 구조는 다음과 같습니다.

```text
                  Redux Store
                       ▲
                       │
                 dispatch(action)
                       │
                  useDispatch()
                       ▲
                       │
               React Component
                       ▲
                       │
                  useSelector()
                       │
                       ▼
                  Selected State


그리고 이 모든 React Redux Hook은

        <Provider store={store}>
                   │
                   ▼
          React Redux Context
                   │
                   ▼
             Component Tree

를 통해 현재 Redux Store에 접근한다.
```

즉 역할을 정확히 나누면:

```text
Provider
   ↓
"어떤 Store를 사용할 것인가?"

useSelector
   ↓
"그 Store에서 어떤 값을 사용할 것인가?"

useDispatch
   ↓
"그 Store에 Action을 어떻게 전달할 것인가?"
```

라고 이해할 수 있습니다.

---

# 22. Redux State가 바뀌면 모든 컴포넌트가 리렌더링되는가?

아닙니다.

이 질문은 React Redux를 이해하는 데 매우 중요합니다.

예를 들어:

```text
Redux State
│
├── counter.value = 10
│
├── user.name = "Kim"
│
└── cart.items = [...]
```

세 개의 컴포넌트가 있다고 해보겠습니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  // ...
}
```

```jsx
function User() {
  const name = useSelector(
    state => state.user.name
  );

  // ...
}
```

```jsx
function Cart() {
  const items = useSelector(
    state => state.cart.items
  );

  // ...
}
```

`counter.value`만:

```text
10 → 11
```

로 변경되었다고 해보겠습니다.

개념적으로:

```text
Store Update
     │
     ├── Counter selector
     │       │
     │       └── 10 → 11
     │              ↓
     │          Re-render
     │
     ├── User selector
     │       │
     │       └── "Kim" → "Kim"
     │              ↓
     │             유지
     │
     └── Cart selector
             │
             └── same reference
                    ↓
                   유지
```

즉:

> **Redux Store가 변경되었다는 사실과 모든 React 컴포넌트가 리렌더링된다는 것은 같은 의미가 아닙니다.**

React Redux는 각 `useSelector()`가 선택한 결과의 변경 여부를 기준으로 Redux Store 업데이트에 따른 리렌더링 필요성을 판단합니다.

---

# 23. `useSelector()`가 있다고 부모 리렌더링까지 막아주는 것은 아니다

여기서는 Redux에 의한 리렌더링과 React 자체의 리렌더링을 구분해야 합니다.

예를 들어:

```text
Parent
  │
  └── Counter
       └── useSelector()
```

`Counter`가 선택한 Redux 값이 변하지 않았더라도 `Parent`가 다른 이유로 리렌더링되면 일반적인 React 렌더링 규칙에 따라 `Counter` 함수도 다시 호출될 수 있습니다.

따라서:

```text
useSelector()
     ↓
Redux Store 변경 때문에
불필요하게 리렌더링되는 것을 줄일 수 있음
```

이라고 이해해야 합니다.

다음처럼 이해하면 안 됩니다.

```text
useSelector()
     ↓
어떤 경우에도
컴포넌트 리렌더링 안 됨
```

React Redux의 최적화와 React 자체의 렌더링 최적화는 서로 관련되어 있지만 완전히 같은 문제는 아닙니다.

이 부분은 PART 4에서 `React.memo`와 함께 다시 살펴봅니다.

---

# 24. 가장 중요한 사고방식

React Redux를 단순히 다음처럼 외우면:

```text
useSelector = 값 가져오기

useDispatch = Action 보내기
```

사용법은 알 수 있지만 내부 흐름은 이해하기 어렵습니다.

조금 더 정확하게 이해해야 합니다.

### `useSelector()`

```text
Provider를 통해 Store 접근
        ↓
selector 실행
        ↓
필요한 값 선택
        ↓
Store 변경 구독
        ↓
selector 결과 비교
        ↓
필요하면 Re-render
```

### `useDispatch()`

```text
Provider를 통해 Store 접근
        ↓
Store의 dispatch 사용
        ↓
Action 전달
        ↓
Redux의 상태 변경 흐름 시작
```

두 흐름을 합치면:

```text
                  React
                    │
                    │ Action
                    ▼
              useDispatch()
                    │
                    ▼
                Redux Store
                    │
                    │ State Update
                    ▼
               Subscription
                    │
                    ▼
              useSelector()
                    │
                    │ Selected State
                    ▼
                  React
```

이것이 React Redux의 핵심입니다.

---

# 25. 그런데 내부에서는 어떻게 이것이 가능한가?

지금까지는 React Redux를 사용하는 관점에서 살펴봤습니다.

하지만 몇 가지 질문이 남아 있습니다.

첫 번째:

> **`useSelector()`는 React가 아닌 외부 Redux Store의 변경을 어떻게 React 렌더링 시스템과 안전하게 연결할까?**

두 번째:

> **왜 selector가 객체를 새로 만들면 불필요한 리렌더링이 발생할 수 있을까?**

세 번째:

> **여러 값을 객체로 선택하고 싶다면 어떻게 해야 할까?**

네 번째:

> **`shallowEqual`은 무엇이며 언제 사용하는가?**

다섯 번째:

> **`React.memo`와 `useSelector()`는 어떤 관계가 있는가?**

이 질문들은 단순한 API 사용법을 넘어 **React Redux의 내부 동작과 최적화**에 해당합니다.

다음 PART에서 이 내용을 살펴봅니다.

```text
PART 4

React Redux Internals & Optimization
              │
              ├── External Store
              │
              ├── Subscription
              │
              ├── useSyncExternalStore
              │
              ├── Equality Comparison
              │
              ├── shallowEqual
              │
              ├── Selector 설계
              │
              └── React.memo
```

---

# 핵심 정리

React Redux의 두 핵심 Hook은 다음과 같습니다.

```text
useSelector()
     ↓
Redux Store → React

useDispatch()
     ↓
React → Redux Store
```

`useSelector()`는:

> **Redux Store의 전체 State에서 필요한 값을 선택하고 Store 변경을 구독하며, 선택 결과가 변경되면 컴포넌트가 새로운 값을 반영하도록 React 렌더링과 연결합니다.**

`useDispatch()`는:

> **현재 Provider가 제공하는 Redux Store의 `dispatch` 함수를 React 컴포넌트에서 사용할 수 있게 합니다.**

그리고 두 Hook을 합치면 React Redux의 전체 흐름이 완성됩니다.

```text
User Event
    ↓
React Component
    ↓
useDispatch()
    ↓
dispatch(action)
    ↓
Redux Store
    ↓
Reducer
    ↓
New State
    ↓
Store Update
    ↓
Subscription
    ↓
useSelector()
    ↓
selector 재평가
    ↓
선택 결과 비교
    ↓
필요하면 Re-render
    ↓
React Component
```

여기서 반드시 기억해야 할 것은:

```text
Redux State 변경
      ≠
모든 React Component Re-render
```

이라는 점입니다.

정확한 흐름은:

```text
Store Update
     ↓
Subscription
     ↓
selector 재평가
     ↓
selected value 비교
     ↓
필요한 경우 Re-render
```

입니다.

다음 PART에서는 이 흐름의 내부로 한 단계 더 내려가 **React Redux가 외부 Redux Store의 변경을 React와 어떻게 연결하는지**, 그리고 selector와 리렌더링을 어떻게 최적화해야 하는지 살펴보겠습니다.
