# PART 4. React Redux Internals & Optimization

## — Store의 변경은 어떻게 React 렌더링으로 연결되는가?

PART 3에서는 React Redux의 두 핵심 Hook을 중심으로 전체 데이터 흐름을 살펴봤습니다.

```text
useDispatch()
     ↓
React → Redux Store

useSelector()
     ↓
Redux Store → React
```

그리고 Store가 변경되었을 때 다음과 같은 흐름이 일어난다고 설명했습니다.

```text
Store Update
     ↓
Subscription
     ↓
selector 재평가
     ↓
selected value 비교
     ↓
필요하면 Re-render
```

이번 PART에서는 이 흐름을 한 단계 더 깊게 살펴봅니다.

핵심 질문은 다음과 같습니다.

> **React Redux는 React 바깥에 존재하는 Redux Store의 변경을 어떻게 React 렌더링 시스템과 연결하는가?**

그리고 이어서:

```text
왜 객체 selector가 문제가 될 수 있는가?

왜 === 비교가 중요한가?

shallowEqual은 언제 사용하는가?

memoized selector는 왜 필요한가?

React.memo와 useSelector는 어떤 관계인가?
```

를 살펴봅니다.

---

# 1. Redux Store는 React 외부의 상태 저장소다

가장 먼저 다시 확인해야 할 사실이 있습니다.

Redux Store는 React State가 아닙니다.

```text
React
│
├── useState
├── useReducer
└── Component Rendering


Redux
│
└── Redux Store
     ├── getState()
     ├── dispatch()
     └── subscribe()
```

즉 Redux Store는 React와 독립적으로 존재하는 **External Store**입니다.

예를 들어:

```javascript
const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

여기서 만들어진 `store`는 React 컴포넌트 안에서 생성되는 State가 아닙니다.

개념적으로:

```text
JavaScript Runtime
│
├── Redux Store
│    └── State
│
└── React
     └── Component Tree
```

따라서 중요한 문제가 생깁니다.

> Redux Store의 State가 변경되었을 때 React는 그 사실을 어떻게 알 수 있을까?

React 자체는 Redux Store를 자동으로 감시하지 않습니다.

이 연결을 React Redux가 담당합니다.

---

# 2. Redux의 `subscribe()`

Redux Store는 외부에서 State 변경을 감지할 수 있도록 `subscribe()`를 제공합니다.

```javascript
const unsubscribe = store.subscribe(() => {
  console.log(store.getState());
});
```

Action이 dispatch되고 State가 변경되면:

```text
dispatch(action)
      ↓
   Reducer
      ↓
 New State
      ↓
Redux Store
      ↓
 subscribers에게 알림
```

이 흐름이 일어납니다.

즉 Redux Store 자체는 이미 다음 기능을 가지고 있습니다.

```text
"State가 변경되었다."
        ↓
subscriber에게 알린다.
```

하지만 이 기능만으로 React 렌더링과 안전하게 연결되는 것은 아닙니다.

React Redux가 이 외부 구독 시스템과 React 사이를 연결합니다.

---

# 3. `useSelector()`의 실제 역할을 다시 보자

PART 3에서 다음 코드를 사용했습니다.

```jsx
const count = useSelector(
  state => state.counter.value
);
```

초급 수준에서는 다음처럼 이해할 수 있었습니다.

```text
Redux Store
     ↓
useSelector()
     ↓
필요한 State 선택
```

하지만 실제 역할은 조금 더 많습니다.

개념적으로:

```text
useSelector()
│
├── 현재 Store에 접근
├── 현재 State 읽기
├── selector 실행
├── Store 변경 구독
├── Store 변경 시 selector 재평가
├── 이전 선택 결과와 비교
└── 필요하면 React Re-render 연결
```

즉 `useSelector()`는 단순한 State 조회 Hook이 아닙니다.

> **External Store의 State와 React 렌더링을 연결하는 Hook**

이라고 이해하는 것이 더 정확합니다.

---

# 4. External Store와 React를 연결할 때 생기는 문제

단순하게 생각하면 다음과 같이 구현하고 싶을 수 있습니다.

```javascript
store.subscribe(() => {
  forceRender();
});
```

하지만 현대 React의 렌더링 시스템에서는 외부 Store와의 연결을 단순히 이런 방식으로 처리하면 문제가 생길 수 있습니다.

React는 렌더링을 다음처럼 단순한 한 번의 동기 작업으로만 처리하지 않습니다.

```text
render
  ↓
commit
```

현대 React는 렌더링 작업을 중단하거나 다시 시도할 수도 있고, 여러 렌더링 작업을 조정할 수도 있습니다.

따라서 외부 State를 읽을 때는:

```text
현재 Render가 보고 있는 값

그리고

Commit 시점에 실제 Store가 가지고 있는 값
```

사이에 일관성이 유지되어야 합니다.

이 문제를 안전하게 처리하기 위해 React는 외부 Store를 구독하기 위한 공식적인 메커니즘을 제공합니다.

그 핵심이 `useSyncExternalStore`입니다.

---

# 5. `useSyncExternalStore`란?

React의 `useSyncExternalStore`는 한 문장으로 정의하면:

> **React 외부에 존재하는 Store를 React 렌더링 시스템과 안전하게 구독하기 위한 Hook입니다.**

개념적인 형태는 다음과 같습니다.

```javascript
useSyncExternalStore(
  subscribe,
  getSnapshot
);
```

두 가지 핵심 정보가 필요합니다.

```text
subscribe
   ↓
Store가 변경되었음을 어떻게 알 것인가?

getSnapshot
   ↓
현재 Store의 값을 어떻게 읽을 것인가?
```

예를 들면 개념적으로:

```javascript
useSyncExternalStore(
  store.subscribe,
  store.getState
);
```

라고 생각할 수 있습니다.

실제 React Redux의 구현은 selector 처리, Context, subscription 관리, 개발 모드 검사 등 여러 계층이 추가되어 있지만 핵심 아이디어는 비슷합니다.

---

# 6. `useSelector()`와 `useSyncExternalStore`

현대 React Redux에서 `useSelector()`는 외부 Redux Store와 React를 연결하는 과정에서 React의 외부 Store 구독 메커니즘을 활용합니다.

개념적인 흐름은:

```text
Redux Store
     │
     ├── subscribe()
     │
     └── getState()
             │
             ▼
External Store Subscription
             │
             ▼
React Redux
             │
             ▼
useSelector()
             │
             ▼
React Rendering
```

조금 더 자세히 보면:

```text
Redux Store
    │
    │ State 변경
    ▼
Subscription 알림
    │
    ▼
현재 Store Snapshot 확인
    │
    ▼
selector 실행
    │
    ▼
selected value 계산
    │
    ▼
이전 selected value와 비교
    │
    ▼
필요하면 React에 Update 요청
```

여기서 중요한 것은:

> React Redux가 직접 무작정 컴포넌트를 다시 그리는 것이 아니라, React가 외부 Store를 안전하게 다룰 수 있는 방식으로 Store 변경을 React에 연결한다는 점입니다.

---

# 7. Snapshot이란?

`useSyncExternalStore`를 이해할 때 `snapshot`이라는 표현이 자주 나옵니다.

Snapshot은 어렵게 생각할 필요가 없습니다.

> **특정 시점에서 외부 Store를 읽었을 때 얻을 수 있는 현재 상태 값**

이라고 보면 됩니다.

Redux Store에서는 개념적으로:

```javascript
store.getState();
```

의 반환값이 현재 Store 상태의 snapshot 역할을 합니다.

예를 들어:

```text
Redux Store

counter.value = 10
user.name = "Kim"
```

현재 snapshot은:

```javascript
{
  counter: {
    value: 10
  },

  user: {
    name: 'Kim'
  }
}
```

입니다.

Action이 dispatch되면:

```text
Before Snapshot
      ↓
dispatch
      ↓
After Snapshot
```

으로 바뀝니다.

React Redux는 이런 Store snapshot과 selector 결과를 이용해 컴포넌트가 실제로 새로운 값을 필요로 하는지 판단합니다.

---

# 8. Store가 변경되었다고 바로 Re-render하지 않는다

이 부분은 React Redux 최적화의 핵심입니다.

다음 컴포넌트가 있다고 해보겠습니다.

```jsx
function Counter() {
  const count = useSelector(
    state => state.counter.value
  );

  return <h1>{count}</h1>;
}
```

현재 Redux State:

```javascript
{
  counter: {
    value: 10
  },

  user: {
    name: 'Kim'
  }
}
```

selector 결과:

```text
10
```

이제 `user.name`만 변경됩니다.

```text
Kim
 ↓
Lee
```

Store 전체 State 객체는 바뀝니다.

```text
Previous Root State
       !==
New Root State
```

하지만 `Counter`의 selector 결과는 그대로입니다.

```text
Previous selected value = 10
New selected value      = 10
```

따라서:

```text
10 === 10
```

이고, Redux Store 변경 때문에 `Counter`를 다시 렌더링할 필요가 없습니다.

즉:

```text
Store State 변경 여부
```

가 아니라:

```text
현재 컴포넌트가 선택한 값의 변경 여부
```

가 중요합니다.

---

# 9. 기본 비교는 `===`

`useSelector()`는 기본적으로 selector 결과가 변경되었는지 판단할 때 strict equality를 사용합니다.

즉:

```javascript
previousSelected === currentSelected
```

를 기준으로 생각하면 됩니다.

흐름은:

```text
Store Update
     ↓
selector 재실행
     ↓
previousSelected
        │
        │ ===
        ▼
currentSelected
     │
 ┌───┴────┐
 │        │
same    different
 │        │
 ▼        ▼
skip    Re-render
```

Primitive 값은 이해하기 쉽습니다.

```javascript
10 === 10
// true
```

```javascript
10 === 11
// false
```

문제는 객체와 배열입니다.

---

# 10. 객체와 배열은 Reference가 중요하다

JavaScript에서 객체는 값 자체가 아니라 reference를 기준으로 `===` 비교됩니다.

```javascript
const a = {
  name: 'Kim'
};

const b = {
  name: 'Kim'
};

console.log(a === b);
// false
```

내용은 같아 보이지만 서로 다른 객체입니다.

```text
a ──▶ Object A

b ──▶ Object B
```

따라서:

```javascript
a === b
```

는 `false`입니다.

이 특성이 `useSelector()` 최적화에서 매우 중요합니다.

---

# 11. 잘못된 Selector 예제

다음 selector를 보겠습니다.

```jsx
const user = useSelector(state => ({
  name: state.user.name,
  age: state.user.age
}));
```

Store Update가 발생할 때마다 selector가 다시 실행될 수 있습니다.

그리고 매번:

```javascript
{
  name: state.user.name,
  age: state.user.age
}
```

라는 **새로운 객체**를 생성합니다.

예를 들어 실제 값은 변하지 않았다고 해도:

```text
Previous

{
  name: "Kim",
  age: 30
}


Current

{
  name: "Kim",
  age: 30
}
```

두 객체의 reference는 다를 수 있습니다.

```javascript
previous === current
// false
```

따라서 불필요한 Redux Store 업데이트에도 컴포넌트가 리렌더링될 수 있습니다.

---

# 12. 해결 방법 1 — 필요한 값을 각각 선택한다

가장 단순한 방법은 필요한 값을 각각 선택하는 것입니다.

```jsx
const name = useSelector(
  state => state.user.name
);

const age = useSelector(
  state => state.user.age
);
```

이 경우:

```text
selector #1
   ↓
"Kim"

selector #2
   ↓
30
```

처럼 각각의 결과를 비교할 수 있습니다.

```text
"Kim" === "Kim"
30 === 30
```

값이 같으면 Redux 변경으로 인한 리렌더링 필요성이 생기지 않습니다.

초급 및 일반적인 코드에서는 이 방식이 매우 읽기 쉽고 안전합니다.

---

# 13. 해결 방법 2 — 이미 존재하는 객체를 선택한다

다음처럼 Store 안에 이미 존재하는 객체 reference를 그대로 선택할 수도 있습니다.

```jsx
const user = useSelector(
  state => state.user
);
```

이 경우 Redux 불변성 규칙이 제대로 지켜지고 있다면 `user`와 관계없는 다른 State만 변경되었을 때 기존 `user` reference가 유지될 수 있습니다.

예를 들어:

```text
Root State
│
├── counter  ← 변경됨
│
└── user     ← 기존 reference 유지
```

라면:

```javascript
previousUser === currentUser
// true
```

가 될 수 있습니다.

따라서 해당 컴포넌트의 Redux Store 업데이트로 인한 리렌더링을 피할 수 있습니다.

이것이 Redux에서 **불변성 유지와 reference identity가 중요한 또 하나의 이유**입니다.

---

# 14. 해결 방법 3 — `shallowEqual`

여러 값을 하나의 객체 형태로 가져오고 싶을 수도 있습니다.

```jsx
const userInfo = useSelector(
  state => ({
    name: state.user.name,
    age: state.user.age
  })
);
```

이 경우 React Redux의 `shallowEqual`을 사용할 수 있습니다.

```jsx
import {
  useSelector,
  shallowEqual
} from 'react-redux';

const userInfo = useSelector(
  state => ({
    name: state.user.name,
    age: state.user.age
  }),
  shallowEqual
);
```

`shallowEqual`은 객체 reference 자체만 비교하지 않고 **객체의 1-depth 프로퍼티 값을 비교**합니다.

예를 들어:

```javascript
{
  name: 'Kim',
  age: 30
}
```

과:

```javascript
{
  name: 'Kim',
  age: 30
}
```

가 서로 다른 객체여도:

```text
name === name
age  === age
```

라면 shallow comparison 결과는 같다고 판단할 수 있습니다.

---

# 15. Shallow Comparison이란?

Shallow Comparison은 이름 그대로 **얕은 비교**입니다.

다음 객체를 생각해봅시다.

```javascript
const a = {
  name: 'Kim',
  age: 30
};

const b = {
  name: 'Kim',
  age: 30
};
```

일반 `===` 비교는:

```javascript
a === b
// false
```

입니다.

하지만 shallow comparison은 프로퍼티 단위로 비교합니다.

```text
a.name === b.name
a.age  === b.age
```

모두 같다면 동일한 결과로 취급할 수 있습니다.

---

# 16. Shallow Comparison은 깊은 비교가 아니다

다음과 같은 객체를 보겠습니다.

```javascript
const a = {
  user: {
    name: 'Kim'
  }
};

const b = {
  user: {
    name: 'Kim'
  }
};
```

1-depth에서 비교하면:

```javascript
a.user === b.user
```

가 됩니다.

두 `user` 객체가 새로 만들어졌다면:

```javascript
a.user === b.user
// false
```

입니다.

즉 `shallowEqual`은 내부 깊숙한 값까지 재귀적으로 비교하는 Deep Equality가 아닙니다.

```text
Shallow Equality

Object
│
├── property #1 비교
├── property #2 비교
└── property #3 비교

여기까지만
```

따라서 중첩 객체를 매번 새로 만드는 구조를 `shallowEqual` 하나로 모두 해결할 수 있는 것은 아닙니다.

---

# 17. Memoized Selector란?

조금 더 복잡한 경우에는 **Memoized Selector**를 사용할 수 있습니다.

예를 들어 selector 안에서 계산을 수행한다고 해보겠습니다.

```javascript
state => state.products.filter(
  product => product.price >= 10000
)
```

`filter()`는 실행할 때마다 새로운 배열을 생성합니다.

```text
Store Update
     ↓
selector 실행
     ↓
filter()
     ↓
새 Array
```

실제 `products`가 바뀌지 않아도 selector가 다시 실행되면 새로운 배열 reference가 생성될 수 있습니다.

이런 경우 memoization을 사용할 수 있습니다.

핵심 아이디어는:

```text
입력이 같다
   ↓
이전에 계산했던 결과를 재사용
```

입니다.

---

# 18. `createSelector()`

Redux Toolkit에는 memoized selector를 만들 수 있는 `createSelector()`가 포함되어 있습니다.

예를 들어:

```javascript
import { createSelector } from '@reduxjs/toolkit';

const selectProducts = state => state.products;

const selectExpensiveProducts = createSelector(
  [selectProducts],

  products =>
    products.filter(
      product => product.price >= 10000
    )
);
```

컴포넌트에서는:

```jsx
const products = useSelector(
  selectExpensiveProducts
);
```

처럼 사용할 수 있습니다.

개념적인 동작은:

```text
selectProducts
     ↓
입력 products 확인
     ↓
이전 입력과 동일?
    /      \
  YES      NO
   │        │
   ▼        ▼
기존 결과   다시 계산
재사용      ↓
          결과 저장
```

입니다.

---

# 19. Memoized Selector가 유용한 경우

모든 selector를 memoized selector로 만들 필요는 없습니다.

다음처럼 단순한 접근은:

```javascript
state => state.counter.value
```

memoization이 거의 필요하지 않습니다.

하지만 다음과 같은 경우에는 유용할 수 있습니다.

```text
배열 filter

배열 map

배열 sort

복잡한 계산

여러 State 조합

매번 객체/배열을 새로 생성하는 selector
```

예를 들어:

```javascript
state =>
  state.users
    .filter(user => user.active)
    .sort((a, b) =>
      a.name.localeCompare(b.name)
    )
```

와 같은 계산은 memoized selector를 고려할 수 있습니다.

---

# 20. Selector는 가능한 한 Pure 해야 한다

Selector는 State를 받아 값을 계산하는 함수입니다.

```javascript
state => state.user.name
```

따라서 selector 자체는 가능한 한 **Pure Function**으로 작성하는 것이 중요합니다.

좋은 selector:

```javascript
state => state.counter.value
```

좋은 selector:

```javascript
state => state.user.name
```

좋은 selector:

```javascript
state => state.products.length
```

반면 selector 안에서 다음과 같은 Side Effect를 수행하면 안 됩니다.

```javascript
state => {
  fetch('/api/users');

  return state.users;
}
```

또는:

```javascript
state => {
  console.log('외부 상태 변경');

  someGlobalValue++;

  return state.user;
}
```

selector는 여러 상황에서 반복 실행될 수 있기 때문에 **언제 몇 번 호출되는지에 의존하는 코드를 넣으면 안 됩니다.**

---

# 21. Selector는 Render 중에도 실행될 수 있다

`useSelector()`는 컴포넌트 렌더링과 연결되어 있기 때문에 selector는 React 렌더 과정에서 실행될 수 있습니다.

따라서 selector는:

```text
빠르고

예측 가능하고

Side Effect가 없고

동일한 입력에 동일한 결과를 주는
```

형태가 좋습니다.

즉:

> selector는 "State를 조회하고 계산하는 함수"이지, 외부 작업을 수행하는 장소가 아닙니다.

---

# 22. React Redux와 `React.memo`

이제 자주 혼동하는 두 가지 최적화를 구분해 보겠습니다.

```text
useSelector()
```

와:

```text
React.memo()
```

는 서로 다른 문제를 해결합니다.

---

# 23. `useSelector()`가 다루는 문제

`useSelector()`는 주로 다음 방향의 변경을 다룹니다.

```text
Redux Store
      ↓
Component
```

즉 Redux Store가 업데이트되었을 때 현재 컴포넌트가 선택한 값이 바뀌었는지 판단합니다.

```text
Store Update
     ↓
Selector
     ↓
Selected Value Changed?
     ↓
Re-render 여부 판단
```

---

# 24. `React.memo()`가 다루는 문제

`React.memo()`는 주로 부모 컴포넌트의 리렌더링으로 인해 자식 컴포넌트가 다시 호출되는 문제를 줄이는 데 사용됩니다.

```text
Parent Re-render
      ↓
Child
```

기본적으로 부모가 리렌더링되면 자식 함수 컴포넌트도 다시 실행될 수 있습니다.

```jsx
const Child = React.memo(function Child({
  name
}) {
  return <div>{name}</div>;
});
```

`React.memo()`는 props가 이전과 같다면 부모 리렌더링에 따른 자식 렌더링을 생략할 수 있도록 돕습니다.

---

# 25. `useSelector()`와 `React.memo()`는 서로 대체 관계가 아니다

다음 구조를 생각해봅시다.

```text
Redux Store
      │
      ▼
   Counter
      ▲
      │
    Parent
```

`Counter`는 두 방향에서 렌더링 영향을 받을 수 있습니다.

```text
Redux Store Update
       ↓
useSelector()


Parent Re-render
       ↓
React Rendering
```

따라서:

```text
useSelector
→ Redux Store 변경과 관련된 최적화

React.memo
→ 부모 props 기반 리렌더링과 관련된 최적화
```

라고 구분하면 이해하기 쉽습니다.

둘은 필요에 따라 함께 사용할 수도 있습니다.

---

# 26. `useSelector()`가 `React.memo()` 역할까지 해주는 것은 아니다

예를 들어:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button
        onClick={() => setCount(count + 1)}
      >
        Parent Update
      </button>

      <Child />
    </>
  );
}
```

그리고:

```jsx
function Child() {
  const name = useSelector(
    state => state.user.name
  );

  return <h1>{name}</h1>;
}
```

Redux의 `user.name`이 변경되지 않았더라도 `Parent`가 리렌더링되면 일반적인 React 렌더링 규칙에 따라 `Child`가 다시 호출될 수 있습니다.

`useSelector()`가 이를 자동으로 차단하는 것은 아닙니다.

필요한 경우:

```jsx
const Child = React.memo(function Child() {
  const name = useSelector(
    state => state.user.name
  );

  return <h1>{name}</h1>;
});
```

처럼 별도의 React 렌더링 최적화를 고려할 수 있습니다.

다만 모든 컴포넌트에 `React.memo()`를 붙이는 것은 권장되는 접근이 아닙니다.

실제 성능 문제가 있는 곳에서 사용하는 것이 좋습니다.

---

# 27. React Redux 최적화의 핵심은 "선택 범위"

가장 기본적이고 효과적인 최적화는 복잡한 API를 사용하는 것이 아닙니다.

먼저 selector를 적절하게 작성하는 것입니다.

예를 들어:

```jsx
const state = useSelector(
  state => state
);
```

보다:

```jsx
const count = useSelector(
  state => state.counter.value
);
```

가 더 명확합니다.

또:

```jsx
const user = useSelector(
  state => ({
    name: state.user.name,
    age: state.user.age
  })
);
```

를 무조건 사용하는 것보다:

```jsx
const name = useSelector(
  state => state.user.name
);

const age = useSelector(
  state => state.user.age
);
```

처럼 필요한 값만 선택하는 것이 더 단순할 수 있습니다.

핵심은:

> **컴포넌트가 필요한 State 범위만 구독하도록 selector를 설계하는 것**

입니다.

---

# 28. Redux State 구조와 Re-render 범위를 연결해서 생각하자

Redux State가 다음과 같다고 해보겠습니다.

```text
Root State
│
├── auth
│    ├── user
│    └── token
│
├── products
│    └── items
│
└── cart
     └── items
```

컴포넌트가:

```jsx
const root = useSelector(
  state => state
);
```

를 사용하면 너무 넓은 범위를 선택합니다.

반면:

```jsx
const cartItems = useSelector(
  state => state.cart.items
);
```

를 사용하면 해당 컴포넌트가 관심 있는 값이 훨씬 명확해집니다.

```text
Root State
│
├── auth
│
├── products
│
└── cart
     │
     └── items ← 이 값에 관심
```

이런 selector 설계가 React Redux 성능 최적화의 가장 기본적인 출발점입니다.

---

# 29. 최적화를 너무 일찍 하지 말자

React Redux를 배우면 다음 키워드를 빠르게 만나게 됩니다.

```text
shallowEqual

createSelector

React.memo

useMemo

useCallback
```

하지만 이들을 무조건 사용한다고 성능이 좋아지는 것은 아닙니다.

오히려 코드 복잡도가 높아질 수 있습니다.

우선순위는 다음과 같이 잡는 것이 좋습니다.

```text
1. 필요한 State만 선택한다.
        ↓
2. 불필요하게 새로운 객체/배열을 만들지 않는다.
        ↓
3. 실제 성능 문제가 있는지 확인한다.
        ↓
4. 필요한 곳에 shallowEqual / memoized selector 사용
        ↓
5. React 렌더링 문제라면 React.memo 등 검토
```

즉 최적화 도구보다 **올바른 State 선택과 구조 설계가 먼저**입니다.

---

# 30. React Redux 내부 흐름을 한 번에 보기

이제 전체 내부 흐름을 하나로 합쳐보겠습니다.

```text
┌─────────────────────────────────────┐
│            Redux Store              │
│                                     │
│  getState()                         │
│  dispatch()                         │
│  subscribe()                        │
└────────────────┬────────────────────┘
                 │
                 │ State Update
                 ▼
          Subscription Notify
                 │
                 ▼
┌─────────────────────────────────────┐
│           React Redux               │
│                                     │
│  External Store 연결                │
│  Snapshot 확인                      │
│  Selector 실행                      │
│  Equality 비교                      │
└────────────────┬────────────────────┘
                 │
                 ▼
        selected value 비교
                 │
          ┌──────┴───────┐
          │              │
        동일           변경됨
          │              │
          ▼              ▼
        유지       React Update 요청
                         │
                         ▼
┌─────────────────────────────────────┐
│              React                  │
│                                     │
│       Component Re-render           │
└─────────────────────────────────────┘
```

이것이 React Redux가 Redux Store와 React 렌더링 시스템을 연결하는 핵심 구조입니다.

---

# 31. Provider까지 포함한 전체 구조

PART 2의 Provider까지 포함하면 전체 구조는 다음과 같습니다.

```text
                    Redux Store
                         │
                         │ store
                         ▼
                    <Provider>
                         │
                         ▼
                React Redux Context
                         │
                         ▼
                 React Component
                   │           │
                   │           │
              useDispatch   useSelector
                   │           │
                   │           ▼
                   │       Selector
                   │           │
                   │           ▼
                   │    Selected Value
                   │           │
                   │           ▼
                   │     Equality Check
                   │           │
                   │           ▼
                   │      Re-render?
                   │
                   ▼
             dispatch(action)
                   │
                   ▼
                Redux Store
```

이 구조 안에서 각 요소의 역할은 명확합니다.

---

# 32. 네 개 PART 전체 연결

지금까지 React Redux를 네 단계로 살펴봤습니다.

```text
PART 1
React Redux Fundamentals
        │
        ▼
React와 Redux 사이에
왜 연결 계층이 필요한가?
```

```text
PART 2
Provider & Context
        │
        ▼
컴포넌트는 Store가
어디 있는지 어떻게 아는가?
```

```text
PART 3
useSelector & useDispatch
        │
        ▼
State를 어떻게 읽고
Action을 어떻게 보내는가?
```

```text
PART 4
Internals & Optimization
        │
        ▼
Store 변경이 어떻게
React Re-render로 연결되며
어떻게 불필요한 렌더링을 줄이는가?
```

전체 흐름은:

```text
Redux Store
     │
     ▼
Provider
     │
     ▼
Context
     │
     ▼
React Component
     │
     ├── useDispatch()
     │        ↓
     │   Action 전달
     │        ↓
     │   Redux Store Update
     │
     └── useSelector()
              ↓
        Store Subscription
              ↓
        Selector 재평가
              ↓
        Equality Comparison
              ↓
        필요한 경우 Re-render
```

입니다.

---

# 핵심 정리

React Redux의 내부 동작을 한 문장으로 정리하면:

> **React Redux는 Provider와 Context를 통해 Redux Store에 접근하고, 외부 Store 구독 메커니즘을 React 렌더링 시스템과 연결하여 Store 변경 시 selector를 다시 평가한 뒤 선택 결과가 실제로 변경된 컴포넌트가 새로운 값을 반영하도록 합니다.**

핵심 흐름은 다음과 같습니다.

```text
Store Update
     ↓
Subscription
     ↓
Snapshot 확인
     ↓
Selector 실행
     ↓
Selected Value
     ↓
Equality Comparison
     ↓
필요한 경우 Re-render
```

그리고 최적화에서 가장 중요한 원칙은:

```text
필요한 State만 선택한다.
```

입니다.

그 다음 필요에 따라:

```text
shallowEqual
     ↓
객체의 1-depth 값 비교

createSelector
     ↓
계산 결과 memoization

React.memo
     ↓
부모 리렌더링에 따른
자식 렌더링 최적화
```

를 사용할 수 있습니다.

마지막으로 반드시 구분해야 합니다.

```text
useSelector
→ Redux Store Update와
  선택한 State 변화의 연결

React.memo
→ Parent Re-render와
  Child Re-render의 연결
```

둘은 서로 다른 문제를 해결합니다.

---

# React Redux 전체 최종 흐름

```text
                  User Event
                      │
                      ▼
                React Component
                      │
                useDispatch()
                      │
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
                Store Notify
                      │
                      ▼
              External Store
               Subscription
                      │
                      ▼
                  Selector
                      │
                      ▼
               Selected Value
                      │
                      ▼
             Equality Comparison
                      │
              ┌───────┴────────┐
              │                │
            동일             변경됨
              │                │
              ▼                ▼
            유지          React Update
                               │
                               ▼
                      Component Re-render
```

이 흐름을 이해하면 React Redux를 단순히:

```text
Provider
useSelector
useDispatch
```

라는 API 세 개로 외우는 수준을 넘어,

> **Redux Store와 React Rendering System을 연결하는 구조**

로 이해할 수 있습니다.
