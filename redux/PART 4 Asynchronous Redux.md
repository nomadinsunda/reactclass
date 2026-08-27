# PART 4. Asynchronous Redux

## — Middleware, Thunk, Promise, `createAsyncThunk()`까지

PART 3에서는 Redux Toolkit을 사용하여 React 애플리케이션의 **동기적인 State**를 관리하는 방법을 학습했습니다.

기본 흐름은 다음과 같았습니다.

```text
React Component
      ↓
Action Creator
      ↓
Action
      ↓
dispatch()
      ↓
Redux Store
      ↓
Reducer
      ↓
New State
      ↓
useSelector()
      ↓
React Re-render
```

예를 들어 Counter 값을 증가시키는 작업은 단순합니다.

```javascript
dispatch(increment());
```

Action이 dispatch되면 Reducer가 새로운 State를 계산합니다.

하지만 실제 웹 애플리케이션에서는 서버와 통신해야 하는 경우가 많습니다.

```text
로그인
회원 정보 조회
상품 목록 조회
게시글 조회
주문 생성
파일 업로드
데이터 저장
```

이러한 작업은 즉시 끝나지 않습니다.

```text
Request
   ↓
대기...
   ↓
Response
```

즉 **Asynchronous Operation**이 필요합니다.

이번 PART의 핵심 질문은 이것입니다.

> **Reducer는 State를 계산하는 함수인데, API Request 같은 비동기 작업은 Redux의 어디에서 처리해야 할까?**

이 질문에서 Redux의 **Middleware와 Thunk**가 등장합니다.

---

# 1. 동기적인 Redux와 비동기 작업

동기적인 Redux 작업을 먼저 다시 생각해봅시다.

```javascript
dispatch(increment());
```

Action:

```javascript
{
    type: "counter/increment"
}
```

Reducer:

```javascript
increment(state) {
    state.value++;
}
```

전체 흐름은 매우 짧습니다.

```text
dispatch(action)
      ↓
Reducer
      ↓
New State
```

반면 서버에서 상품 목록을 가져오는 작업은 다릅니다.

```javascript
const response =
    await fetch("/api/products");

const products =
    await response.json();
```

중간에 서버의 응답을 기다려야 합니다.

```text
Request
   ↓
Promise Pending
   ↓
Response
```

따라서 단순한:

```text
Action
  ↓
Reducer
  ↓
State
```

구조만으로 비동기 작업 자체를 표현하기는 어렵습니다.

---

# 2. 왜 Reducer에서 API를 호출하면 안 되는가?

다음과 같은 코드를 작성하면 어떨까요?

```javascript
const productSlice = createSlice({
    name: "products",

    initialState: {
        items: []
    },

    reducers: {
        loadProducts: async (state) => {
            const response =
                await fetch("/api/products");

            const products =
                await response.json();

            state.items = products;
        }
    }
});
```

이 코드는 Redux Reducer의 역할에 맞지 않습니다.

Reducer의 기본 역할은:

```text
Current State
     +
   Action
     ↓
  Reducer
     ↓
 New State
```

입니다.

즉 Reducer는 **State 계산**에 집중해야 합니다.

API Request와 같은 작업은 State 계산 자체가 아니라 **Side Effect**입니다.

---

# 3. Side Effect와 Pure Reducer

Side Effect는 함수가 단순히 입력을 이용해 결과를 계산하는 것을 넘어 **외부 세계와 상호작용하거나 외부 상태에 의존하는 작업**을 의미합니다.

대표적으로:

```text
HTTP Request
Database Access
File I/O
setTimeout()
localStorage
현재 시간 읽기
Random 값 생성
```

등이 있습니다.

예를 들어:

```javascript
fetch("/api/products");
```

는 네트워크라는 외부 세계와 상호작용합니다.

따라서 Side Effect입니다.

반면 Redux Reducer는 개념적으로 **Pure Function**을 기반으로 합니다.

```javascript
reducer(state, action)
```

즉:

```text
State + Action
     ↓
  Reducer
     ↓
 New State
```

라는 계산에 집중합니다.

따라서 역할을 다음과 같이 분리해야 합니다.

```text
Reducer
=
State 계산


Side Effect
=
Reducer 외부에서 처리
```

그렇다면 새로운 질문이 생깁니다.

> **Reducer 밖의 어디에서 비동기 작업을 처리할까?**

---

# 4. Middleware

Redux에는 Action이 Reducer로 전달되는 과정에 개입할 수 있는 구조가 있습니다.

바로 **Middleware**입니다.

기존 흐름:

```text
dispatch(action)
      ↓
Reducer
```

Middleware가 존재하면:

```text
dispatch(action)
      ↓
Middleware
      ↓
Reducer
```

가 됩니다.

Middleware에서는 다음과 같은 작업을 수행할 수 있습니다.

```text
Logging
Async Operation
Error Handling
Analytics
Action 변환
Side Effect
```

따라서 구조적으로 보면:

```text
React Component
      ↓
dispatch(...)
      ↓
┌─────────────────────┐
│     Middleware      │
│                     │
│ Logging             │
│ Async Operation     │
│ Side Effect         │
└──────────┬──────────┘
           ↓
        Reducer
           ↓
         State
```

입니다.

핵심 역할을 구분하면:

```text
Reducer
=
State 계산


Middleware
=
dispatch와 Reducer 사이에서
추가적인 로직을 처리
```

입니다.

---

# 5. Redux Thunk

Redux에서 가장 대표적인 비동기 Middleware가 **Thunk Middleware**입니다.

Redux Toolkit의 `configureStore()`에는 기본적으로 Thunk Middleware가 포함되어 있습니다.

따라서 일반적인 Redux Toolkit 프로젝트에서는 별도로 설치하지 않아도 사용할 수 있습니다.

일반적인 Redux Action은 객체입니다.

```javascript
{
    type: "counter/increment"
}
```

따라서 일반적인 dispatch는:

```text
dispatch(
    Action Object
)
```

입니다.

그런데 Thunk Middleware가 존재하면 **함수도 dispatch할 수 있습니다.**

```text
dispatch(
    Function
)
```

이 함수가 **Thunk Function**입니다.

---

# 6. Thunk Function이란?

Redux에서 Thunk는 일반적으로 다음 형태를 가집니다.

```javascript
function fetchProducts() {

    return async function (
        dispatch,
        getState
    ) {

        // asynchronous operation
    };
}
```

Arrow Function으로 작성하면:

```javascript
const fetchProducts = () =>
    async (dispatch, getState) => {

        // asynchronous operation
    };
```

즉 일반 Action과 형태부터 다릅니다.

```text
일반 Action

{
    type: "..."
}
```

반면:

```text
Thunk

function(dispatch, getState) {
    ...
}
```

입니다.

---

# 7. Thunk Middleware는 무엇을 하는가?

다음 코드가 실행되었다고 생각해봅시다.

```javascript
dispatch(fetchProducts());
```

먼저:

```javascript
fetchProducts()
```

가 실행됩니다.

그리고 다음과 같은 Thunk Function을 반환합니다.

```javascript
async (dispatch, getState) => {
    ...
}
```

결국 실제 형태는:

```text
dispatch(
    async function(...)
)
```

이 됩니다.

Thunk Middleware는 dispatch된 값을 검사합니다.

```text
dispatch(value)
      ↓
Thunk Middleware
      ↓
함수인가?
  ↙       ↘
YES       NO
 ↓         ↓
실행      next(action)
```

함수라면 개념적으로:

```javascript
thunkFunction(dispatch, getState);
```

를 실행합니다.

그래서 Thunk 안에서는:

```javascript
dispatch(...)
```

를 다시 사용할 수 있고,

```javascript
getState()
```

를 통해 현재 Redux State도 읽을 수 있습니다.

---

# 8. 일반 Action과 Thunk의 차이

일반 Action:

```text
Action Object
     ↓
dispatch
     ↓
Middleware
     ↓
Reducer
     ↓
State
```

Thunk:

```text
Thunk Function
      ↓
dispatch
      ↓
Thunk Middleware
      ↓
Thunk 실행
      ↓
Async Operation
      ↓
dispatch(Action)
      ↓
Reducer
      ↓
State
```

여기서 매우 중요한 점이 있습니다.

> **Thunk가 State를 직접 변경하는 것이 아닙니다.**

Thunk는 비동기 작업을 수행하고 **Action을 dispatch**합니다.

State 변경은 여전히 Reducer가 담당합니다.

```text
Thunk
  ↓
Side Effect
  ↓
dispatch(Action)
  ↓
Reducer
  ↓
State
```

이 원칙은 Redux에서 매우 중요합니다.

---

# 9. 직접 Thunk 작성하기

상품 목록을 가져오는 기능을 만들어봅시다.

먼저 State를 다음과 같이 구성합니다.

```javascript
const initialState = {
    items: [],
    loading: false,
    error: null
};
```

비동기 요청에는 일반적으로 세 가지 결과가 필요합니다.

```text
요청 시작
요청 성공
요청 실패
```

따라서 Slice에 다음 Reducer를 정의할 수 있습니다.

```javascript
const productSlice = createSlice({
    name: "products",

    initialState,

    reducers: {
        fetchStart(state) {
            state.loading = true;
            state.error = null;
        },

        fetchSuccess(state, action) {
            state.loading = false;
            state.items = action.payload;
        },

        fetchFailure(state, action) {
            state.loading = false;
            state.error = action.payload;
        }
    }
});
```

Action Creator를 꺼냅니다.

```javascript
export const {
    fetchStart,
    fetchSuccess,
    fetchFailure
} = productSlice.actions;
```

---

# 10. 직접 작성한 Thunk

이제 실제 API Request를 수행하는 Thunk를 작성합니다.

```javascript
export const fetchProducts = () => {

    return async (dispatch) => {

        dispatch(fetchStart());

        try {
            const response =
                await fetch("/api/products");

            if (!response.ok) {
                throw new Error(
                    "Request failed"
                );
            }

            const products =
                await response.json();

            dispatch(
                fetchSuccess(products)
            );

        } catch (error) {

            dispatch(
                fetchFailure(error.message)
            );
        }
    };
};
```

React Component에서는:

```javascript
dispatch(fetchProducts());
```

로 실행합니다.

---

# 11. 직접 작성한 Thunk의 전체 실행 흐름

이 코드는 다음과 같이 동작합니다.

```text
React Component
      ↓
fetchProducts()
      ↓
Thunk Function 반환
      ↓
dispatch(thunk)
      ↓
Thunk Middleware
      ↓
Thunk Function 실행
      ↓
dispatch(fetchStart())
      ↓
Reducer
      ↓
loading = true
      ↓
fetch("/api/products")
      ↓
Promise Pending
      ↓
Server Response
      ↓
response.json()
      ↓
Promise Fulfilled
      ↓
dispatch(fetchSuccess(products))
      ↓
Reducer
      ↓
items = products
loading = false
```

실패하면:

```text
fetch()
   ↓
Error
   ↓
catch
   ↓
dispatch(fetchFailure(error))
   ↓
Reducer
   ↓
error 저장
loading = false
```

가 됩니다.

---

# 12. Promise와 Redux 비동기 State

여기서 JavaScript Promise와 Redux State가 연결됩니다.

Promise는 다음 세 가지 상태를 가집니다.

```text
Promise
│
├── Pending
├── Fulfilled
└── Rejected
```

Redux의 비동기 요청도 같은 구조로 생각할 수 있습니다.

```text
Promise State        Redux State

Pending        →     loading = true

Fulfilled      →     data 저장

Rejected       →     error 저장
```

즉:

```text
API Request
     ↓
  Pending
   ↙   ↘
Success Failure
  ↓      ↓
Fulfilled Rejected
```

라는 구조입니다.

이 패턴은 거의 모든 API마다 반복됩니다.

---

# 13. 직접 Thunk의 문제점

직접 Thunk를 작성하는 방식은 정상적으로 동작합니다.

하지만 반복 코드가 많습니다.

```javascript
dispatch(fetchStart());

try {

    const response =
        await fetch(...);

    const data =
        await response.json();

    dispatch(fetchSuccess(data));

} catch (error) {

    dispatch(
        fetchFailure(error.message)
    );
}
```

API가 증가하면:

```text
Products API
    ↓
start / success / failure

Users API
    ↓
start / success / failure

Orders API
    ↓
start / success / failure
```

와 같은 패턴이 계속 반복됩니다.

Redux Toolkit은 이 반복적인 비동기 Action 처리 패턴을 자동화하는 API를 제공합니다.

바로:

```javascript
createAsyncThunk()
```

입니다.

---

# 14. `createAsyncThunk()`

`createAsyncThunk()`는 **Promise 기반 비동기 작업을 Redux의 Action 흐름과 연결하기 위한 Redux Toolkit API**입니다.

기본 구조는:

```javascript
createAsyncThunk(
    "actionTypePrefix",

    async () => {
        // asynchronous operation
    }
);
```

예:

```javascript
import {
    createAsyncThunk
} from "@reduxjs/toolkit";

export const fetchProducts =
    createAsyncThunk(
        "products/fetchProducts",

        async () => {

            const response =
                await fetch(
                    "/api/products"
                );

            if (!response.ok) {
                throw new Error(
                    "Failed to fetch products"
                );
            }

            return response.json();
        }
    );
```

사용할 때는:

```javascript
dispatch(fetchProducts());
```

라고 작성합니다.

---

# 15. `createAsyncThunk()`는 무엇을 자동화하는가?

`createAsyncThunk()`의 핵심은 비동기 작업의 세 상태를 자동으로 Action으로 표현해준다는 것입니다.

```text
products/fetchProducts/pending

products/fetchProducts/fulfilled

products/fetchProducts/rejected
```

즉:

```text
createAsyncThunk()
       ↓
Payload Creator 실행
       ↓
Promise
       ↓
┌───────────────┐
│               │
↓               ↓
Fulfilled     Rejected
```

이 과정에서 Redux Toolkit은:

```text
시작
 ↓
pending Action

성공
 ↓
fulfilled Action

실패
 ↓
rejected Action
```

을 자동으로 dispatch합니다.

---

# 16. `pending`

비동기 작업이 시작되면:

```text
products/fetchProducts/pending
```

Action이 자동으로 dispatch됩니다.

이때 보통:

```javascript
state.loading = true;
state.error = null;
```

로 설정합니다.

---

# 17. `fulfilled`

Payload Creator가 값을 반환하면:

```javascript
return products;
```

`fulfilled` Action이 자동으로 dispatch됩니다.

개념적으로:

```javascript
{
    type:
        "products/fetchProducts/fulfilled",

    payload: products
}
```

입니다.

따라서:

```javascript
action.payload
```

를 통해 서버 데이터를 받을 수 있습니다.

---

# 18. `rejected`

비동기 작업이 실패하면:

```text
products/fetchProducts/rejected
```

Action이 dispatch됩니다.

일반적인 예외 정보는:

```javascript
action.error
```

를 통해 확인할 수 있습니다.

---

# 19. `createAsyncThunk()` 전체 흐름

이 부분은 PART 4에서 가장 중요한 흐름입니다.

```text
dispatch(fetchProducts())
          ↓
fetchProducts()
          ↓
Thunk 생성
          ↓
Thunk Middleware
          ↓
pending Action
          ↓
Reducer
          ↓
loading = true
          ↓
Payload Creator 실행
          ↓
fetch()
          ↓
Promise
      ┌───┴───┐
      ↓       ↓
 Fulfilled  Rejected
      ↓       ↓
fulfilled  rejected
 Action     Action
```

여기서 중요한 것은:

> **`createAsyncThunk()`가 Promise의 상태를 Redux Action으로 연결해준다.**

는 것입니다.

---

# 20. `extraReducers`

이제 자동 생성된:

```text
pending
fulfilled
rejected
```

Action을 처리해야 합니다.

`createSlice()`의 `reducers`는 Slice가 **직접 생성하는 Action과 Reducer**를 정의합니다.

```javascript
reducers: {
    clearProducts(state) {
        state.items = [];
    }
}
```

이 경우:

```javascript
clearProducts()
```

라는 Action Creator도 생성됩니다.

반면 `createAsyncThunk()`가 만든 Action은 Slice의 `reducers`가 직접 만든 Action이 아닙니다.

이런 Action에 반응하기 위해 `extraReducers`를 사용할 수 있습니다.

---

# 21. `reducers`와 `extraReducers`

둘의 차이는 다음처럼 기억하면 됩니다.

```text
reducers
=
이 Slice가 Action을 직접 정의하고 생성


extraReducers
=
다른 곳에서 생성된 Action에 반응
```

대표적인 `extraReducers` 대상이:

```text
fetchProducts.pending
fetchProducts.fulfilled
fetchProducts.rejected
```

입니다.

---

# 22. `builder.addCase()`

`builder.addCase()`는 특정 Action이 발생했을 때 실행할 Reducer 로직을 등록합니다.

```javascript
extraReducers: builder => {

    builder
        .addCase(
            fetchProducts.pending,
            state => {
                state.loading = true;
                state.error = null;
            }
        )

        .addCase(
            fetchProducts.fulfilled,
            (state, action) => {
                state.loading = false;
                state.items =
                    action.payload;
            }
        )

        .addCase(
            fetchProducts.rejected,
            (state, action) => {
                state.loading = false;
                state.error =
                    action.error.message;
            }
        );
}
```

결과적으로:

```text
pending
   ↓
loading = true


fulfilled
   ↓
loading = false
items = action.payload


rejected
   ↓
loading = false
error = ...
```

가 됩니다.

---

# 23. 전체 Product Slice

```javascript
import {
    createAsyncThunk,
    createSlice
} from "@reduxjs/toolkit";


export const fetchProducts =
    createAsyncThunk(
        "products/fetchProducts",

        async () => {

            const response =
                await fetch(
                    "/api/products"
                );

            if (!response.ok) {
                throw new Error(
                    "Failed to fetch products"
                );
            }

            return response.json();
        }
    );


const productSlice = createSlice({
    name: "products",

    initialState: {
        items: [],
        loading: false,
        error: null
    },

    reducers: {},

    extraReducers: builder => {

        builder
            .addCase(
                fetchProducts.pending,
                state => {
                    state.loading = true;
                    state.error = null;
                }
            )

            .addCase(
                fetchProducts.fulfilled,
                (state, action) => {
                    state.loading = false;
                    state.items =
                        action.payload;
                }
            )

            .addCase(
                fetchProducts.rejected,
                (state, action) => {
                    state.loading = false;
                    state.error =
                        action.error.message;
                }
            );
    }
});


export default productSlice.reducer;
```

---

# 24. React Component에서 사용하기

```jsx
import { useEffect } from "react";

import {
    useDispatch,
    useSelector
} from "react-redux";

import {
    fetchProducts
} from "./productSlice";


function ProductList() {

    const dispatch = useDispatch();

    const {
        items,
        loading,
        error
    } = useSelector(
        state => state.products
    );

    useEffect(() => {
        dispatch(fetchProducts());
    }, [dispatch]);


    if (loading) {
        return <p>Loading...</p>;
    }

    if (error) {
        return <p>{error}</p>;
    }

    return (
        <ul>
            {items.map(product => (
                <li key={product.id}>
                    {product.name}
                </li>
            ))}
        </ul>
    );
}

export default ProductList;
```

Component의 실행 과정은 다음과 같습니다.

```text
ProductList Render
       ↓
useEffect()
       ↓
dispatch(fetchProducts())
       ↓
pending
       ↓
loading = true
       ↓
Re-render
       ↓
Loading...
       ↓
Server Response
       ↓
fulfilled
       ↓
items = products
loading = false
       ↓
Re-render
       ↓
상품 목록 표시
```

---

# 25. `createAsyncThunk()`에 아규먼트 전달하기

특정 상품 하나를 조회한다고 생각해봅시다.

```text
GET /api/products/10
```

`createAsyncThunk()`의 Payload Creator에 값을 전달할 수 있습니다.

```javascript
export const fetchProduct =
    createAsyncThunk(
        "products/fetchProduct",

        async (productId) => {

            const response =
                await fetch(
                    `/api/products/${productId}`
                );

            return response.json();
        }
    );
```

호출:

```javascript
dispatch(fetchProduct(10));
```

흐름은:

```text
fetchProduct(10)
      ↓
productId = 10
      ↓
Payload Creator
      ↓
GET /api/products/10
```

입니다.

여기서 `10`은 `fetchProduct()` 호출 시 전달한 **아규먼트**이고, Payload Creator에서는 `productId`라는 **파라미터**로 받습니다.

---

# 26. Payload Creator와 `thunkAPI`

`createAsyncThunk()`의 두 번째 아규먼트로 전달하는 함수를 **Payload Creator**라고 합니다.

```javascript
createAsyncThunk(
    "products/fetchProduct",

    async (productId, thunkAPI) => {
        // ...
    }
);
```

Payload Creator는 일반적으로 Promise를 반환합니다.

성공적으로 반환한 값은:

```javascript
return product;
```

`fulfilled` Action의:

```javascript
action.payload
```

가 됩니다.

두 번째 파라미터인 `thunkAPI`를 통해서는 대표적으로 다음 기능을 사용할 수 있습니다.

```text
dispatch
getState
rejectWithValue
signal
requestId
```

입문 단계에서는 특히:

```text
dispatch
getState
rejectWithValue
```

를 알아두면 좋습니다.

---

# 27. `getState()`

Thunk 내부에서 현재 Redux State가 필요한 경우가 있습니다.

예를 들어 인증 토큰이:

```javascript
{
    auth: {
        token: "..."
    }
}
```

에 있다면:

```javascript
const state =
    thunkAPI.getState();

const token =
    state.auth.token;
```

으로 가져올 수 있습니다.

그리고:

```javascript
await fetch("/api/users/me", {
    headers: {
        Authorization:
            `Bearer ${token}`
    }
});
```

처럼 사용할 수 있습니다.

---

# 28. `rejectWithValue()`

서버가 반환한 오류 데이터를 rejected Action에 직접 전달하고 싶을 수 있습니다.

```javascript
if (!response.ok) {

    return thunkAPI.rejectWithValue(
        data
    );
}
```

이 경우 실패 데이터는:

```javascript
action.payload
```

에서 확인할 수 있습니다.

반면 일반적인 예외:

```javascript
throw new Error("Network Error");
```

의 정보는 일반적으로:

```javascript
action.error
```

를 통해 확인합니다.

따라서:

```text
throw Error
    ↓
action.error


rejectWithValue(data)
    ↓
action.payload
```

라고 구분할 수 있습니다.

---

# 29. 비동기 State 설계

비동기 State는 흔히 다음과 같이 설계할 수 있습니다.

```javascript
{
    data: null,
    loading: false,
    error: null
}
```

목록이라면:

```javascript
{
    items: [],
    loading: false,
    error: null
}
```

또는 상태를 문자열로 더 명확하게 표현할 수도 있습니다.

```javascript
{
    items: [],
    status: "idle",
    error: null
}
```

상태 흐름:

```text
idle
 ↓
loading
 ↓
┌──────────────┐
↓              ↓
succeeded    failed
```

두 방식 모두 가능하며 애플리케이션의 요구에 따라 선택할 수 있습니다.

---

# 30. `createAsyncThunk()`의 한계와 RTK Query

`createAsyncThunk()`는 Redux에서 비동기 로직을 처리하는 강력한 도구입니다.

하지만 서버 데이터가 많아지면 또 다른 문제가 생깁니다.

예를 들어:

```text
GET /products
GET /products/10
POST /products
PUT /products/10
DELETE /products/10

GET /users
GET /orders
```

각 서버 데이터마다 다음을 직접 관리해야 할 수 있습니다.

```text
data
loading
error

fetch
cache
refetch
중복 요청
데이터 만료
동기화
```

여기서 **Client State와 Server State**의 차이가 중요해집니다.

```text
Application State
│
├── Client State
│
└── Server State
```

### Client State

클라이언트 애플리케이션 자체가 소유하는 상태입니다.

```text
Modal 열림 여부
Sidebar 상태
선택된 Tab
검색 조건
Wizard 단계
```

이런 상태는 `createSlice()`로 관리하기 좋습니다.

### Server State

원본 데이터가 서버에 존재하는 상태입니다.

```text
상품
회원
게시글
주문
댓글
재고
```

클라이언트는 서버 데이터의 복사본을 가져와 사용합니다.

```text
Database / Server
       ↓
      HTTP
       ↓
     Client
       ↓
Cached Server Data
```

따라서 Server State에는 추가적인 문제가 있습니다.

```text
Fetching
Caching
Synchronization
Invalidation
Refetch
Request Deduplication
Subscription
Loading / Error
```

원문에서도 이 부분이 매우 잘 잡혀 있습니다. Server State를 단순한 `data/loading/error` 문제에서 끝내지 않고 **Caching, Synchronization, Invalidation, Subscription** 문제로 확장한 것이 RTK Query로 넘어가는 좋은 연결점입니다. 

---

# 31. 그래서 RTK Query가 필요하다

Redux Toolkit은 Server State의 **Fetching과 Caching을 전문적으로 관리하는 도구**를 제공합니다.

바로 **RTK Query**입니다.

RTK Query는 다음과 같은 문제를 다루도록 설계되어 있습니다.

```text
API Request
Loading
Error
Cache
Request Deduplication
Subscription
Refetch
Cache Invalidation
```

따라서:

```text
createAsyncThunk
=
일반적인 비동기 Redux Logic


RTK Query
=
Server State Fetching + Caching
```

정도로 역할을 구분하면 좋습니다.

`createAsyncThunk()`가 무조건 구식이고 RTK Query가 무조건 더 좋은 것은 아닙니다.

복잡한 비동기 Workflow나 Redux State와 긴밀하게 결합된 Business Logic에서는 `createAsyncThunk()`가 적합할 수 있고, 서버 데이터를 가져오고 Cache하고 동기화하는 것이 핵심이라면 RTK Query가 더 적합할 수 있습니다. 원문도 이 역할 차이를 명확하게 구분하고 있습니다. 

---

# 32. PART 4 전체 실행 흐름

이번 PART에서 가장 중요하게 기억해야 할 흐름입니다.

```text
React Component
      ↓
dispatch(fetchProducts())
      ↓
fetchProducts()
      ↓
Thunk Function 생성
      ↓
Thunk Middleware
      ↓
pending Action
      ↓
Reducer
      ↓
loading = true
      ↓
Payload Creator
      ↓
fetch()
      ↓
Promise
   ┌──┴──┐
   ↓     ↓
resolve reject
   ↓     ↓
fulfilled rejected
 Action    Action
   ↓        ↓
Reducer   Reducer
   ↓        ↓
 data     error
   └───┬────┘
       ↓
  Redux Store
       ↓
  useSelector()
       ↓
React Re-render
```

이 흐름이 머릿속에 보인다면 Redux의 기본적인 비동기 처리 구조를 이해한 것입니다.

---

# 33. PART 4 핵심 정리

이번 PART의 개념을 하나의 구조로 연결하면 다음과 같습니다.

```text
Reducer
   │
   │ Side Effect 처리 X
   ↓
Middleware
   ↓
Thunk Middleware
   ↓
Thunk Function
   ↓
Async Operation
   ↓
Promise
   │
   ├── Pending
   ├── Fulfilled
   └── Rejected
   ↓
createAsyncThunk()
   ↓
pending / fulfilled / rejected
Action 자동 생성
   ↓
extraReducers
   ↓
Redux State
   ↓
React
```

각 개념을 한 문장으로 정리하면:

| 개념                   | 역할                                          |
| -------------------- | ------------------------------------------- |
| Reducer              | State를 계산한다                                 |
| Middleware           | dispatch와 Reducer 사이의 처리 과정에 개입한다           |
| Thunk                | 비동기 로직 등을 담을 수 있는 함수                        |
| Thunk Middleware     | dispatch된 Thunk Function을 실행한다              |
| Promise              | 비동기 작업의 Pending/Fulfilled/Rejected 상태를 표현한다 |
| `createAsyncThunk()` | Promise 기반 비동기 작업과 Redux Action을 연결한다       |
| `extraReducers`      | Slice 외부에서 만들어진 Action에 반응한다                |
| RTK Query            | Server State의 Fetching과 Caching을 전문적으로 관리한다 |

---

# 다음 단계 — PART 5. RTK Query

서버 데이터가 많아지면 단순히 데이터를 가져오는 것만으로 끝나지 않습니다.

```text
이미 받은 데이터를 다시 요청해야 하는가?

같은 API를 여러 Component가 사용하면?

Cache는 언제 제거할 것인가?

POST 이후 기존 GET 결과는 어떻게 갱신할 것인가?

중복 Request는 어떻게 막을 것인가?

Component가 사라지면 Subscription은 어떻게 할 것인가?
```

바로 이 문제를 해결하기 위해 **RTK Query**가 등장합니다.

다음 PART에서는:

```text
Client State
vs
Server State
      ↓
RTK Query
      ↓
createApi()
      ↓
fetchBaseQuery()
      ↓
endpoints
      ↓
builder.query()
builder.mutation()
      ↓
Generated Hooks
      ↓
Query Cache
      ↓
Subscription
      ↓
Request Deduplication
      ↓
providesTags
invalidatesTags
      ↓
Cache Invalidation
      ↓
Automatic Refetch
```

의 흐름으로 학습하면 자연스럽습니다.

