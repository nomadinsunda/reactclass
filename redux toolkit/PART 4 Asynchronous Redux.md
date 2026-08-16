# PART 4. Asynchronous Redux

PART 2에서는 Redux Toolkit을 사용하여 React 애플리케이션의 동기적인 상태를 관리하는 방법을 학습했습니다.

기본적인 흐름은 다음과 같았습니다.

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

예를 들어 Counter의 값을 증가시키는 작업은 매우 단순합니다.

```javascript
dispatch(increment());
```

이 Action이 dispatch되면 Reducer가 즉시 State를 변경합니다.

하지만 실제 웹 애플리케이션에서는 대부분 서버와 통신해야 합니다.

```text
로그인
회원 정보 조회
상품 목록 조회
게시글 조회
주문 생성
파일 업로드
데이터 저장
```

이러한 작업은 즉시 결과가 반환되지 않습니다.

네트워크 요청을 보내고 서버의 응답을 기다려야 합니다.

즉, **Asynchronous Operation**이 필요합니다.

PART 3에서는 Redux에서 이러한 비동기 작업을 어떻게 처리하는지 살펴봅니다.

---

# 1. 동기적인 Redux와 비동기 작업의 차이

먼저 동기적인 Redux 작업을 다시 살펴봅시다.

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
State 변경
```

하지만 서버에서 상품 목록을 가져오는 경우는 다릅니다.

```javascript
const response = await fetch("/api/products");

const products = await response.json();
```

여기에는 시간이 필요합니다.

```text
Request
   ↓

대기...

   ↓

Response
```

따라서 단순히:

```text
Action
  ↓
Reducer
  ↓
State
```

만으로는 충분하지 않습니다.

---

# 2. 왜 Reducer에서 API를 호출하면 안 되는가?

다음 코드를 생각해봅시다.

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

Reducer는 기본적으로 다음 역할을 수행해야 합니다.

```text
Current State
     +
Action
     ↓
Reducer
     ↓
New State
```

즉 Reducer는 **State 계산**에 집중해야 합니다.

API 호출과 같은 작업은 State 계산 자체가 아니라 **Side Effect**입니다.

---

# 3. Side Effect란?

함수가 자신의 입력값을 이용하여 결과만 계산하는 것이 아니라 외부 세계에 영향을 주거나 외부 상태에 의존하는 작업을 Side Effect라고 합니다.

대표적인 예는 다음과 같습니다.

```text
HTTP Request

Database Access

File I/O

setTimeout()

console.log()

localStorage

현재 시간 읽기

Random 값 생성
```

예를 들어:

```javascript
fetch("/api/products");
```

는 네트워크와 상호작용합니다.

따라서 Side Effect입니다.

Redux Reducer는 이러한 Side Effect를 수행하는 위치로 사용하지 않습니다.

---

# 4. Reducer는 Pure Function을 기반으로 한다

Redux의 Reducer는 개념적으로 Pure Function이어야 합니다.

Pure Function은 같은 입력에 대해 같은 결과를 계산하며 외부 상태를 변경하지 않는 함수입니다.

예:

```javascript
function add(a, b) {
    return a + b;
}
```

```text
add(10, 20)
   ↓
30
```

언제 호출해도 같은 입력이라면 같은 결과입니다.

Reducer도 개념적으로:

```javascript
reducer(state, action)
```

이라는 입력을 받아 다음 State를 계산합니다.

```text
State + Action
     ↓
Reducer
     ↓
New State
```

반면:

```javascript
async function reducer(state, action) {

    const response =
        await fetch("/api/products");

}
```

처럼 작성하면 네트워크 상태라는 외부 요인에 의존하게 됩니다.

따라서 비동기 작업은 Reducer 밖에서 처리해야 합니다.

---

# 5. 그렇다면 비동기 작업은 어디에서 처리하는가?

Redux에는 Action이 Reducer에 도달하기 전에 중간에서 처리할 수 있는 구조가 있습니다.

이것이 **Middleware**입니다.

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

Redux Middleware는 Action이 Store에 dispatch된 이후 Reducer에 전달되는 과정 중간에 개입할 수 있습니다.

---

# 6. Middleware의 개념

Middleware라는 단어 자체를 보면:

```text
Middle
+
Software
```

즉 중간에 위치하는 소프트웨어라는 의미입니다.

Redux에서는:

```text
dispatch()
    ↓
Middleware
    ↓
Reducer
```

사이에 위치합니다.

Middleware에서는 다음과 같은 작업을 수행할 수 있습니다.

```text
Action Logging

비동기 처리

Error 처리

Analytics

API 호출

Action 변환
```

개념적으로:

```text
React
  ↓
dispatch(action)
  ↓
┌────────────────────┐
│     Middleware     │
│                    │
│ Logging            │
│ Async Operation    │
│ Side Effect        │
└─────────┬──────────┘
          ↓
       Reducer
          ↓
        State
```

---

# 7. Middleware가 필요한 이유

Reducer는 Side Effect를 수행하지 않습니다.

그러나 애플리케이션에는 Side Effect가 반드시 필요합니다.

그래서 역할을 분리합니다.

```text
Reducer
=
State 계산


Middleware
=
State 계산 전에 필요한 부가 작업
```

특히 비동기 Redux에서는 Middleware가 매우 중요합니다.

```text
Component
   ↓
dispatch
   ↓
Middleware
   ↓
API Request
   ↓
Response
   ↓
Action
   ↓
Reducer
   ↓
State
```

---

# 8. Redux Thunk

Redux에서 가장 대표적인 비동기 Middleware가 **Thunk Middleware**입니다.

Redux Toolkit의 `configureStore()`는 기본 설정에 Redux Thunk Middleware를 포함합니다.

따라서 일반적인 Redux Toolkit 프로젝트에서는 별도로 Thunk를 설치하거나 설정하지 않아도 사용할 수 있습니다.

Thunk를 이해하려면 먼저 일반 Redux Action을 생각해야 합니다.

일반 Action은 객체입니다.

```javascript
{
    type: "counter/increment"
}
```

즉:

```text
dispatch(
    Action Object
)
```

입니다.

Thunk Middleware가 존재하면 함수도 dispatch할 수 있습니다.

```text
dispatch(
    Function
)
```

이 함수가 바로 **Thunk Function**입니다.

---

# 9. Thunk란?

Thunk는 실행을 지연시키기 위해 감싸둔 함수라고 이해할 수 있습니다.

Redux에서는 다음과 같은 함수 형태를 많이 사용합니다.

```javascript
function fetchProducts() {

    return async function (dispatch, getState) {

        // asynchronous operation

    };

}
```

또는 Arrow Function으로:

```javascript
const fetchProducts = () =>
    async (dispatch, getState) => {

        // asynchronous operation

    };
```

이 함수는 일반 Action 객체가 아닙니다.

```text
Action

{
    type: "..."
}
```

가 아니라:

```text
Thunk

function(dispatch, getState) {
    ...
}
```

입니다.

---

# 10. Thunk Middleware가 하는 일

다음 코드가 실행되었다고 생각해봅시다.

```javascript
dispatch(fetchProducts());
```

먼저:

```javascript
fetchProducts()
```

가 실행되고 Thunk Function이 반환됩니다.

```javascript
async (dispatch, getState) => {
    ...
}
```

즉 `dispatch()`가 받는 것은 객체가 아니라 함수입니다.

```text
dispatch(
    async function(...)
)
```

Middleware가 없다면 일반 Redux는 이런 함수를 Reducer가 처리할 수 없습니다.

Thunk Middleware가 이를 가로챕니다.

```text
dispatch(thunkFunction)
        ↓
Thunk Middleware
        ↓
"함수인가?"
        ↓
YES
        ↓
thunkFunction(dispatch, getState)
```

즉 Thunk Middleware가 해당 함수를 실행합니다.

---

# 11. 일반 Action과 Thunk Action 비교

일반 Action:

```javascript
dispatch({
    type: "counter/increment"
});
```

흐름:

```text
Action Object
     ↓
dispatch
     ↓
Middleware
     ↓
Reducer
```

Thunk:

```javascript
dispatch(fetchProducts());
```

흐름:

```text
Thunk Function
      ↓
dispatch
      ↓
Thunk Middleware
      ↓
Function 실행
      ↓
Async Operation
```

그리고 Thunk 내부에서 다시 일반 Action을 dispatch할 수 있습니다.

```text
Thunk
  ↓
API Request
  ↓
dispatch(Action)
  ↓
Reducer
```

---

# 12. 직접 Thunk를 작성해보자

먼저 Slice를 만들어봅시다.

```javascript
import { createSlice } from "@reduxjs/toolkit";

const productSlice = createSlice({

    name: "products",

    initialState: {
        items: [],
        loading: false,
        error: null
    },

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

# 13. 직접 작성한 Thunk

```javascript
export const fetchProducts = () => {

    return async (dispatch) => {

        dispatch(fetchStart());

        try {

            const response =
                await fetch("/api/products");

            if (!response.ok) {
                throw new Error("Request failed");
            }

            const products =
                await response.json();

            dispatch(fetchSuccess(products));

        } catch (error) {

            dispatch(
                fetchFailure(error.message)
            );

        }

    };

};
```

이 코드에는 Redux의 비동기 처리 구조가 모두 들어 있습니다.

---

# 14. Thunk 실행 흐름

React Component에서:

```javascript
dispatch(fetchProducts());
```

가 실행됩니다.

전체 흐름은:

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

입니다.

---

# 15. 실패한 경우

서버 요청이 실패하면:

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

즉 비동기 요청 하나를 처리하기 위해 보통 세 가지 상태가 필요합니다.

```text
Request 시작

Request 성공

Request 실패
```

이를 상태로 표현하면:

```text
loading
data
error
```

입니다.

---

# 16. 비동기 요청의 세 가지 상태

대부분의 비동기 요청은 다음 세 단계 중 하나에 있습니다.

```text
Pending
Fulfilled
Rejected
```

Promise와 정확히 연결됩니다.

```text
Promise
│
├── Pending
│
├── Fulfilled
│
└── Rejected
```

Redux에서도 이 세 상태를 그대로 활용할 수 있습니다.

```text
API Request
   ↓
pending
   ↓

   ├── Success → fulfilled
   │
   └── Failure → rejected
```

이 구조가 뒤에서 `createAsyncThunk()`로 자동화됩니다.

---

# 17. JavaScript Promise와 Redux 연결

JavaScript에서 다음 코드를 생각해봅시다.

```javascript
fetch("/api/products")
    .then(response => response.json())
    .then(products => {
        console.log(products);
    })
    .catch(error => {
        console.error(error);
    });
```

Promise 상태는:

```text
fetch()
   ↓
Promise
   ↓
Pending
   ↓
┌───────────────┐
│               │
↓               ↓
Fulfilled     Rejected
```

Redux 비동기 상태도 동일한 구조를 사용할 수 있습니다.

```text
Promise State       Redux State

Pending          →  loading = true

Fulfilled        →  data 저장

Rejected         →  error 저장
```

따라서 Redux의 비동기 처리는 JavaScript Promise의 상태 흐름과 매우 밀접하게 연결됩니다.

---

# 18. 직접 Thunk를 작성하면 생기는 문제

직접 Thunk를 작성하면 동작은 하지만 반복 코드가 많아집니다.

```javascript
dispatch(fetchStart());

try {

    const response =
        await fetch(...);

    const data =
        await response.json();

    dispatch(fetchSuccess(data));

} catch (error) {

    dispatch(fetchFailure(error.message));

}
```

API마다 이런 코드가 필요합니다.

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

Action도 계속 늘어납니다.

```text
products/fetchStart
products/fetchSuccess
products/fetchFailure

users/fetchStart
users/fetchSuccess
users/fetchFailure
```

Redux Toolkit은 이 패턴을 자동화하는 API를 제공합니다.

그것이 **createAsyncThunk()**입니다.

---

# 19. `createAsyncThunk()`

`createAsyncThunk()`는 Promise 기반의 비동기 작업을 처리하기 위한 Redux Toolkit API입니다.

기본 형태는:

```javascript
createAsyncThunk(
    "actionTypePrefix",
    async () => {
        // async operation
    }
);
```

예:

```javascript
import { createAsyncThunk } from "@reduxjs/toolkit";

export const fetchProducts =
    createAsyncThunk(
        "products/fetchProducts",

        async () => {

            const response =
                await fetch("/api/products");

            const products =
                await response.json();

            return products;

        }
    );
```

---

# 20. `createAsyncThunk()`가 반환하는 것

다음 코드를 보면:

```javascript
export const fetchProducts =
    createAsyncThunk(...);
```

`fetchProducts`는 일반적인 Action Creator와 조금 다릅니다.

```javascript
fetchProducts()
```

를 호출하면 Thunk가 생성됩니다.

따라서:

```javascript
dispatch(fetchProducts());
```

형태로 사용합니다.

개념적으로:

```text
fetchProducts()
      ↓
Thunk 생성
      ↓
dispatch(thunk)
      ↓
Thunk Middleware
      ↓
Async Function 실행
```

입니다.

---

# 21. `createAsyncThunk()`가 자동 생성하는 Action

다음과 같이 작성했다고 생각해봅시다.

```javascript
createAsyncThunk(
    "products/fetchProducts",
    async () => {
        ...
    }
);
```

Redux Toolkit은 자동으로 세 종류의 Action Type을 생성합니다.

```text
products/fetchProducts/pending

products/fetchProducts/fulfilled

products/fetchProducts/rejected
```

즉:

```text
createAsyncThunk
       ↓
┌────────────────────────────┐
│ pending                    │
│ fulfilled                  │
│ rejected                   │
└────────────────────────────┘
```

입니다.

---

# 22. Pending Action

비동기 작업이 시작되면 자동으로:

```text
products/fetchProducts/pending
```

Action이 dispatch됩니다.

개념적으로:

```javascript
{
    type: "products/fetchProducts/pending"
}
```

입니다.

이 시점에서 보통:

```javascript
state.loading = true;
```

로 설정합니다.

---

# 23. Fulfilled Action

비동기 함수가 성공적으로 값을 반환하면:

```javascript
return products;
```

Redux Toolkit은 자동으로:

```text
products/fetchProducts/fulfilled
```

Action을 dispatch합니다.

그리고 반환값은 Action의 `payload`가 됩니다.

개념적으로:

```javascript
{
    type: "products/fetchProducts/fulfilled",
    payload: products
}
```

입니다.

따라서 Reducer에서는:

```javascript
action.payload
```

로 데이터를 받을 수 있습니다.

---

# 24. Rejected Action

비동기 작업이 실패하면:

```text
products/fetchProducts/rejected
```

Action이 dispatch됩니다.

개념적으로:

```javascript
{
    type: "products/fetchProducts/rejected",
    error: ...
}
```

가 됩니다.

따라서 실패 상태를 State에 저장할 수 있습니다.

---

# 25. `createAsyncThunk()` 전체 흐름

```text
dispatch(fetchProducts())
          ↓
      Thunk 실행
          ↓
Automatically Dispatch
          ↓
products/fetchProducts/pending
          ↓
      API Request
          ↓
       Promise
      ┌────┴────┐
      ↓         ↓
 Fulfilled    Rejected
      ↓         ↓
fulfilled    rejected
 Action       Action
```

이 흐름은 반드시 이해해야 합니다.

---

# 26. `extraReducers`

`createSlice()`의 `reducers`는 해당 Slice가 직접 정의한 Action에 대한 Reducer 로직을 작성할 때 사용합니다.

예:

```javascript
reducers: {

    addProduct(state, action) {
        ...
    }

}
```

그런데 `createAsyncThunk()`가 생성한:

```text
pending

fulfilled

rejected
```

Action은 `createSlice()`의 `reducers`에서 직접 만든 Action이 아닙니다.

이러한 외부 Action을 처리하기 위해 `extraReducers`를 사용할 수 있습니다.

---

# 27. `extraReducers` 기본 구조

```javascript
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
                (state) => {
                    state.loading = true;
                }
            )

            .addCase(
                fetchProducts.fulfilled,
                (state, action) => {
                    state.loading = false;
                    state.items = action.payload;
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
```

---

# 28. `builder.addCase()`

`builder.addCase()`는 특정 Action이 발생했을 때 실행할 Reducer 로직을 등록합니다.

예:

```javascript
builder.addCase(
    fetchProducts.pending,
    (state) => {
        state.loading = true;
    }
);
```

의 의미는:

```text
fetchProducts.pending Action 발생
        ↓
이 Reducer 실행
        ↓
loading = true
```

입니다.

---

# 29. Pending 처리

```javascript
.addCase(
    fetchProducts.pending,
    state => {

        state.loading = true;

        state.error = null;

    }
)
```

비동기 요청이 시작되었음을 State에 반영합니다.

State:

```javascript
{
    items: [],
    loading: true,
    error: null
}
```

React에서는 이를 사용하여:

```jsx
if (loading) {
    return <p>Loading...</p>;
}
```

같은 UI를 만들 수 있습니다.

---

# 30. Fulfilled 처리

```javascript
.addCase(
    fetchProducts.fulfilled,
    (state, action) => {

        state.loading = false;

        state.items =
            action.payload;

    }
)
```

`payload`에는 `createAsyncThunk()`의 비동기 함수가 반환한 값이 들어갑니다.

즉:

```javascript
return products;
```

↓

```javascript
action.payload === products
```

입니다.

---

# 31. Rejected 처리

```javascript
.addCase(
    fetchProducts.rejected,
    (state, action) => {

        state.loading = false;

        state.error =
            action.error.message;

    }
)
```

요청 실패 상태를 Redux State에 저장합니다.

React에서는:

```jsx
if (error) {
    return <p>{error}</p>;
}
```

와 같이 표현할 수 있습니다.

---

# 32. 전체 Product Slice

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
                await fetch("/api/products");

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

# 33. Store 등록

```javascript
import { configureStore }
    from "@reduxjs/toolkit";

import productReducer
    from "../features/products/productSlice";


const store = configureStore({

    reducer: {

        products: productReducer

    }

});


export default store;
```

Redux State:

```javascript
{
    products: {

        items: [],

        loading: false,

        error: null

    }
}
```

---

# 34. React Component에서 요청 실행

```jsx
import {
    useEffect
} from "react";

import {
    useDispatch,
    useSelector
} from "react-redux";

import {
    fetchProducts
} from "./productSlice";


function ProductList() {

    const dispatch =
        useDispatch();

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

---

# 35. Component 실행 과정

Component가 처음 렌더링됩니다.

```text
ProductList Render
       ↓
useEffect()
       ↓
dispatch(fetchProducts())
```

그 순간:

```text
fetchProducts.pending
       ↓
Reducer
       ↓
loading = true
```

State 변경으로 Component가 다시 렌더링됩니다.

```text
Loading...
```

서버 응답이 도착하면:

```text
fetchProducts.fulfilled
       ↓
Reducer
       ↓
items = products
loading = false
```

다시 렌더링됩니다.

```text
Product 1
Product 2
Product 3
```

---

# 36. 전체 흐름을 한 번에 보기

```text
┌────────────────────────────┐
│      ProductList.jsx       │
└──────────────┬─────────────┘
               │
               │ useEffect()
               ↓
┌────────────────────────────┐
│ dispatch(fetchProducts())  │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│      Thunk Middleware      │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│         pending            │
└──────────────┬─────────────┘
               ↓
          Reducer 실행
               ↓
         loading = true
               ↓
┌────────────────────────────┐
│      fetch("/api/...")     │
└──────────────┬─────────────┘
               ↓
            Promise
          ┌────┴────┐
          ↓         ↓
      resolve     reject
          ↓         ↓
     fulfilled   rejected
          ↓         ↓
       Reducer     Reducer
          ↓         ↓
        items      error
          │         │
          └────┬────┘
               ↓
          Redux Store
               ↓
          useSelector()
               ↓
         React Re-render
```

---

# 37. `reducers`와 `extraReducers` 차이

이 둘은 반드시 구분해야 합니다.

## `reducers`

해당 Slice가 직접 생성할 Action과 Reducer를 정의합니다.

```javascript
reducers: {

    clearProducts(state) {
        state.items = [];
    }

}
```

그러면 Action Creator도 자동 생성됩니다.

```javascript
clearProducts()
```

---

## `extraReducers`

다른 곳에서 생성된 Action을 처리합니다.

대표적으로:

```text
createAsyncThunk가 생성한 Action
```

입니다.

```javascript
extraReducers: builder => {

    builder.addCase(
        fetchProducts.fulfilled,
        ...
    );

}
```

따라서:

```text
reducers

→ Slice가 Action을 직접 생성


extraReducers

→ 외부 Action에 반응
```

이라고 이해할 수 있습니다.

---

# 38. `createAsyncThunk()`에 argument 전달

특정 상품을 조회한다고 생각해봅시다.

```text
GET /api/products/10
```

Thunk에 ID를 전달할 수 있습니다.

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

여기서:

```text
10
```

이 Payload Creator 함수의 파라미터로 전달됩니다.

```javascript
async (productId) => {
    ...
}
```

즉:

```text
fetchProduct(10)
      ↓
productId = 10
      ↓
GET /api/products/10
```

입니다.

---

# 39. Payload Creator

`createAsyncThunk()`의 두 번째 아규먼트 함수는 흔히 **Payload Creator**라고 합니다.

```javascript
createAsyncThunk(

    "products/fetchProduct",

    async (productId) => {

        ...

        return product;

    }

)
```

이 함수는 Promise를 반환합니다.

`async` 함수이므로 실제로는:

```text
Promise
```

를 반환합니다.

성공하면:

```javascript
return product;
```

값이 `fulfilled` Action의 payload가 됩니다.

---

# 40. Promise 관점에서 보는 createAsyncThunk

다음 코드:

```javascript
async (productId) => {

    const response =
        await fetch(...);

    return response.json();

}
```

는 결국 Promise를 반환합니다.

개념적으로:

```text
Payload Creator
      ↓
Promise
      ↓
Pending
      ↓
┌─────────────┐
↓             ↓
Fulfilled   Rejected
```

Redux Toolkit은 이 Promise 상태를 관찰하여:

```text
pending Action

fulfilled Action

rejected Action
```

을 자동으로 dispatch합니다.

따라서 `createAsyncThunk()`는 JavaScript Promise의 상태와 Redux Action을 연결해주는 도구라고 볼 수 있습니다.

---

# 41. `thunkAPI`

Payload Creator는 두 번째 파라미터로 Thunk API 객체를 받을 수 있습니다.

```javascript
createAsyncThunk(

    "products/fetchProducts",

    async (_, thunkAPI) => {

        // ...

    }

)
```

`thunkAPI`를 통해 여러 기능을 사용할 수 있습니다.

대표적으로:

```text
dispatch

getState

rejectWithValue

signal

requestId
```

등이 있습니다.

입문 단계에서는 특히:

```text
dispatch

getState

rejectWithValue
```

를 알아두는 것이 좋습니다.

---

# 42. `getState()`

현재 Redux State가 필요할 수 있습니다.

예를 들어 인증 토큰이 Redux State에 있다고 생각해봅시다.

```javascript
{
    auth: {
        token: "..."
    }
}
```

Thunk에서:

```javascript
const state =
    thunkAPI.getState();

const token =
    state.auth.token;
```

처럼 가져올 수 있습니다.

그리고:

```javascript
const response =
    await fetch("/api/users/me", {

        headers: {
            Authorization:
                `Bearer ${token}`
        }

    });
```

와 같이 사용할 수 있습니다.

---

# 43. `rejectWithValue()`

서버가 실패 응답을 반환했을 때 서버가 전달한 오류 데이터를 Redux에 전달하고 싶을 수 있습니다.

예:

```javascript
export const login =
    createAsyncThunk(

        "auth/login",

        async (
            loginRequest,
            thunkAPI
        ) => {

            const response =
                await fetch(
                    "/api/login",
                    {
                        method: "POST",

                        headers: {
                            "Content-Type":
                                "application/json"
                        },

                        body:
                            JSON.stringify(
                                loginRequest
                            )
                    }
                );

            const data =
                await response.json();

            if (!response.ok) {

                return thunkAPI.rejectWithValue(
                    data
                );

            }

            return data;

        }

    );
```

이 경우 rejected Action에서:

```javascript
action.payload
```

를 통해 서버가 전달한 오류 데이터를 받을 수 있습니다.

---

# 44. `action.error`와 `action.payload`

일반적인 예외:

```javascript
throw new Error("Network Error");
```

는 보통:

```javascript
action.error
```

에 관련 정보가 들어갑니다.

반면:

```javascript
return thunkAPI.rejectWithValue(data);
```

를 사용하면:

```javascript
action.payload
```

로 원하는 실패 데이터를 전달할 수 있습니다.

따라서 서버가 다음과 같은 에러 응답을 보낸다고 생각해봅시다.

```json
{
    "message": "Invalid username or password"
}
```

`rejectWithValue()`를 사용하면 이 데이터를 rejected Action의 payload로 다루기 편해집니다.

---

# 45. 로그인 예제

```javascript
export const login =
    createAsyncThunk(

        "auth/login",

        async (
            credentials,
            thunkAPI
        ) => {

            try {

                const response =
                    await fetch(
                        "/api/login",
                        {
                            method: "POST",

                            headers: {
                                "Content-Type":
                                    "application/json"
                            },

                            body:
                                JSON.stringify(
                                    credentials
                                )
                        }
                    );

                const data =
                    await response.json();

                if (!response.ok) {

                    return thunkAPI.rejectWithValue(
                        data.message
                    );

                }

                return data;

            } catch (error) {

                return thunkAPI.rejectWithValue(
                    "Network error"
                );

            }

        }

    );
```

---

# 46. 로그인 Slice

```javascript
const authSlice = createSlice({

    name: "auth",

    initialState: {

        user: null,

        token: null,

        loading: false,

        error: null

    },

    reducers: {

        logout(state) {

            state.user = null;
            state.token = null;

        }

    },

    extraReducers: builder => {

        builder

            .addCase(
                login.pending,

                state => {

                    state.loading = true;
                    state.error = null;

                }
            )

            .addCase(
                login.fulfilled,

                (state, action) => {

                    state.loading = false;

                    state.user =
                        action.payload.user;

                    state.token =
                        action.payload.token;

                }
            )

            .addCase(
                login.rejected,

                (state, action) => {

                    state.loading = false;

                    state.error =
                        action.payload
                        ?? action.error.message;

                }
            );

    }

});
```

---

# 47. 비동기 요청 State 설계

비동기 State는 보통 다음 형태로 설계할 수 있습니다.

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

상태 변화:

```text
Initial

loading = false
data    = null
error   = null


Pending

loading = true
data    = previous value
error   = null


Fulfilled

loading = false
data    = response
error   = null


Rejected

loading = false
data    = ...
error   = error
```

입니다.

---

# 48. 요청 상태를 문자열로 관리하는 방법

다음과 같이 관리할 수도 있습니다.

```javascript
{
    status: "idle",
    data: [],
    error: null
}
```

상태:

```text
idle

loading

succeeded

failed
```

예:

```javascript
.addCase(
    fetchProducts.pending,

    state => {
        state.status = "loading";
    }
)
```

```javascript
.addCase(
    fetchProducts.fulfilled,

    (state, action) => {

        state.status = "succeeded";

        state.items =
            action.payload;

    }
)
```

```javascript
.addCase(
    fetchProducts.rejected,

    (state, action) => {

        state.status = "failed";

        state.error =
            action.error.message;

    }
)
```

이 방식은 상태를 좀 더 명시적으로 표현할 수 있습니다.

---

# 49. 직접 Thunk와 `createAsyncThunk()` 비교

직접 Thunk:

```text
Thunk 직접 작성
       ↓
pending Action 직접 준비
       ↓
success Action 직접 준비
       ↓
failure Action 직접 준비
       ↓
try/catch
       ↓
각 Action dispatch
```

`createAsyncThunk()`:

```text
Payload Creator 작성
       ↓
Promise 실행
       ↓
pending 자동
       ↓
fulfilled 자동
       ↓
rejected 자동
```

즉 Redux Toolkit이 반복적인 Action 생성 부분을 자동화합니다.

---

# 50. 그렇다면 createAsyncThunk가 모든 비동기 문제를 해결하는가?

아닙니다.

`createAsyncThunk()`는 비동기 작업을 Redux에 통합하는 훌륭한 도구입니다.

하지만 서버 데이터를 많이 관리하기 시작하면 반복 코드가 다시 증가합니다.

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

각각에 대해:

```text
Thunk
pending
fulfilled
rejected
loading
error
data
```

를 관리해야 할 수 있습니다.

---

# 51. 서버 데이터에서 추가로 필요한 문제

서버 데이터는 단순히 요청해서 Redux State에 넣는 것으로 끝나지 않습니다.

다음 문제도 발생합니다.

```text
같은 API를 여러 Component가 호출하면?

이미 받은 데이터를 다시 요청해야 하나?

Cache는 얼마나 유지할 것인가?

POST 후 기존 GET 데이터는 어떻게 갱신할까?

Component가 사라졌을 때 Subscription은?

Loading 상태는 누가 관리할까?

Refetch는 언제 할까?

같은 요청을 중복으로 보내지 않을 방법은?
```

예를 들어:

```text
Header
   ↓
GET /users/1

ProfilePage
   ↓
GET /users/1

Sidebar
   ↓
GET /users/1
```

세 Component가 동일한 데이터를 요청할 수도 있습니다.

단순 `createAsyncThunk()`만 사용하면 이러한 Cache 정책을 직접 설계해야 할 수 있습니다.

---

# 52. Client State와 Server State

여기서 매우 중요한 구분이 등장합니다.

애플리케이션 State를 크게 두 종류로 생각해볼 수 있습니다.

```text
Application State
│
├── Client State
│
└── Server State
```

---

# 53. Client State

Client State는 클라이언트 애플리케이션 자체에서 생성하고 관리하는 상태입니다.

예:

```text
Modal 열림 여부

Sidebar 열림 여부

검색 조건

선택된 Tab

Wizard 진행 단계

클라이언트 전용 설정
```

이런 State는 `createSlice()`로 관리하기 적합합니다.

```text
Client State
     ↓
createSlice()
```

---

# 54. Server State

Server State는 원본 데이터가 서버에 존재합니다.

예:

```text
상품 목록

회원 정보

게시글

주문 목록

댓글

재고 정보
```

React 애플리케이션은 이 데이터의 복사본을 가져와 사용합니다.

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

Loading

Error

Subscription
```

---

# 55. createAsyncThunk 방식의 한계

`createAsyncThunk()`로 상품 목록을 관리하면:

```text
Component
    ↓
dispatch(fetchProducts())
    ↓
Thunk
    ↓
fetch()
    ↓
Promise
    ↓
fulfilled
    ↓
Reducer
    ↓
Store
```

입니다.

하지만 개발자가 직접 관리해야 하는 부분이 많습니다.

```text
items

loading

error

API 호출

중복 요청

cache

refetch

데이터 만료

POST 후 GET 갱신
```

서버 데이터가 많아질수록 이 작업은 반복됩니다.

---

# 56. 그래서 RTK Query가 등장한다

Redux Toolkit에는 서버 데이터의 fetching과 caching을 전문적으로 관리하기 위한 도구가 포함되어 있습니다.

그것이 **RTK Query**입니다.

RTK Query는:

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

등을 자동화해줍니다.

기존 방식:

```text
Component
   ↓
dispatch(thunk)
   ↓
Thunk
   ↓
fetch
   ↓
Promise
   ↓
Action
   ↓
Reducer
   ↓
Store
```

RTK Query:

```text
Component
   ↓
Generated Hook
   ↓
RTK Query
   ↓
Server
```

그리고 RTK Query 내부에서:

```text
Request

Cache

Loading

Error

Subscription

Refetch
```

를 관리합니다.

---

# 57. createAsyncThunk와 RTK Query의 역할 차이

둘 중 하나가 무조건 더 좋은 것은 아닙니다.

목적이 다릅니다.

```text
createAsyncThunk
=
일반적인 비동기 Redux Logic


RTK Query
=
Server State Fetching + Caching
```

예를 들어:

```text
복잡한 비동기 Workflow

여러 Action을 순차적으로 처리

Redux State를 읽으면서 로직 수행

특별한 Business Logic
```

등에서는 `createAsyncThunk()`가 적합할 수 있습니다.

반면:

```text
GET Products

GET Users

POST Order

Update Product

Delete Product
```

와 같이 서버 데이터를 가져오고 동기화하는 작업은 RTK Query가 매우 편리합니다.

---

# 58. PART 3의 전체 구조

지금까지의 흐름을 하나로 연결하면 다음과 같습니다.

```text
Reducer
│
│ Side Effect 금지
│
↓
비동기 작업은 어디서?
│
↓
Middleware
│
↓
Thunk Middleware
│
↓
Thunk Function
│
↓
Promise
│
├── pending
│
├── fulfilled
│
└── rejected
│
↓
createAsyncThunk()
│
↓
extraReducers
│
↓
Redux State
│
↓
React
```

그리고 서버 데이터 관리가 많아지면:

```text
createAsyncThunk()
       ↓
반복적인 Server State 관리
       ↓
Fetching
Caching
Invalidation
Refetch
Subscription
       ↓
RTK Query 필요
```

로 연결됩니다.

---

# 59. PART 2와 PART 3 연결

PART 2:

```text
createSlice
   ↓
dispatch
   ↓
Reducer
   ↓
State
```

PART 3:

```text
dispatch
   ↓
Middleware
   ↓
Async Operation
   ↓
Action
   ↓
Reducer
   ↓
State
```

즉 Middleware가 추가되었습니다.

전체 Redux 흐름은 이제 다음과 같습니다.

```text
React Component
      ↓
dispatch(...)
      ↓
Middleware
      ↓
Action Processing
      ↓
Reducer
      ↓
New State
      ↓
Store
      ↓
useSelector()
      ↓
React Re-render
```

---

# 60. 반드시 기억해야 할 핵심

## Reducer

```text
State 계산 담당
Side Effect 수행 X
```

## Middleware

```text
dispatch와 Reducer 사이에서
부가 작업 처리
```

## Thunk

```text
dispatch할 수 있는 함수
```

## Redux Thunk Middleware

```text
dispatch된 함수 감지
       ↓
함수 실행
       ↓
dispatch / getState 제공
```

## createAsyncThunk

```text
Promise 기반 비동기 작업
       ↓
pending
fulfilled
rejected
Action 자동 생성
```

## extraReducers

```text
Slice 외부에서 만들어진 Action 처리
```

---

# 61. 가장 중요한 실행 흐름

다음 코드를 보면:

```javascript
dispatch(fetchProducts());
```

머릿속에서 다음 과정이 보여야 합니다.

```text
fetchProducts()
      ↓
Thunk 생성
      ↓
dispatch(thunk)
      ↓
Thunk Middleware
      ↓
pending Action
      ↓
Reducer
      ↓
loading = true
      ↓
fetch()
      ↓
Promise
      ↓
┌───────────────┐
↓               ↓
fulfilled     rejected
↓               ↓
Action          Action
↓               ↓
Reducer        Reducer
↓               ↓
data           error
      ↓
Redux Store
      ↓
useSelector
      ↓
React Re-render
```

이 흐름을 이해하면 Redux의 비동기 처리 구조를 이해한 것입니다.

---

# 62. PART 3 최종 정리

Redux Reducer는 State를 계산하는 역할을 담당하기 때문에 API 호출과 같은 Side Effect를 직접 처리하지 않습니다.

Redux에서는 Middleware를 이용하여 이러한 작업을 Reducer 바깥에서 처리할 수 있습니다.

Thunk Middleware는 함수 형태의 값을 dispatch할 수 있게 하며, 그 함수 안에서 비동기 작업을 수행하고 다시 Action을 dispatch할 수 있습니다.

Redux Toolkit의 `createAsyncThunk()`는 이러한 Thunk 패턴을 표준화하고 다음 Action을 자동으로 생성합니다.

```text
pending

fulfilled

rejected
```

`createSlice()`의 `extraReducers`에서는 이러한 Action을 처리하여:

```text
loading

data

error
```

State를 관리할 수 있습니다.

그러나 서버 데이터가 많아지면 Fetching, Caching, Refetch, Subscription, Invalidation과 같은 문제를 직접 관리해야 합니다.

이 문제를 해결하기 위해 Redux Toolkit은 **RTK Query**를 제공합니다.

---

# 다음 단계 — PART 4

PART 4에서는 Redux Toolkit의 서버 상태 관리 도구인 **RTK Query**를 본격적으로 학습합니다.

전체 흐름은 다음과 같습니다.

```text
Client State
vs
Server State
      ↓
RTK Query 등장 이유
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
Cache Key
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
      ↓
Authentication
      ↓
Spring Boot REST API 연동
```

PART 4의 핵심 질문은 하나입니다.

> 서버 데이터를 가져오고 저장하는 것뿐 아니라, 그 데이터를 언제 다시 요청하고 언제 버리고 어떻게 여러 Component가 공유할 것인가?

이 문제를 RTK Query가 어떻게 해결하는지 살펴보게 됩니다.
