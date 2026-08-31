# PART 4. Asynchronous Redux

## Redux에서 비동기 작업은 어떻게 처리하는가?

앞에서 배운 Redux의 기본 데이터 흐름은 다음과 같습니다.

```text
Action
  ↓
dispatch()
  ↓
Reducer
  ↓
New State
  ↓
UI 업데이트
```

이 구조는 동기적인 State 변경을 이해하기에는 충분합니다.

예를 들어:

```js
dispatch(increment());
```

를 실행하면 Action이 Reducer에 전달되고 Reducer는 새로운 State를 계산합니다.

하지만 실제 애플리케이션에서는 다음과 같은 작업이 필요합니다.

```text
로그인
상품 목록 조회
회원 정보 조회
게시글 조회
주문 생성
파일 업로드
```

이러한 작업은 대부분 서버와 통신해야 합니다.

즉,

```text
Request
   ↓
대기
   ↓
Response
```

라는 시간이 필요한 비동기 작업입니다.

여기서 문제가 발생합니다.

> Reducer는 State를 계산하는 역할을 담당하는데 API 요청은 어디에서 처리해야 할까?

이 질문에서 Redux의 **Middleware와 Thunk**가 시작됩니다.

---

# PART 4-1. Asynchronous Redux

## 왜 비동기 작업을 위해 Middleware와 Thunk가 필요한가?

### 1. Reducer의 역할

Reducer는 다음과 같은 함수입니다.

```js
(state, action) => newState
```

즉,

> 현재 State와 Action을 이용하여 새로운 State를 계산합니다.

Reducer는 State 계산에 집중해야 합니다.

따라서 다음과 같은 Side Effect를 Reducer 내부에서 수행하지 않습니다.

```text
API Request
Timer
Logging
파일 처리
외부 시스템 접근
```

예를 들어 다음과 같은 코드를 Reducer에 넣는 구조는 적절하지 않습니다.

```js
function reducer(state, action) {
  fetch("/api/products");

  return state;
}
```

API 호출은 State 계산이 아니라 외부 시스템과의 상호작용입니다.

따라서 다음처럼 역할을 분리해야 합니다.

```text
비동기 작업
API Request
     ↓
결과를 Action으로 변환
     ↓
Reducer
     ↓
State 계산
```

---

## 2. 일반 Action의 동작

일반적인 Action Creator를 생각해 보겠습니다.

```js
const increment = () => {
  return {
    type: "counter/increment"
  };
};
```

컴포넌트에서는 다음처럼 dispatch합니다.

```js
dispatch(increment());
```

실제로는 다음 과정이 발생합니다.

```text
① Action Creator 호출
        ↓
② Action Object 생성
        ↓
③ dispatch(action)
        ↓
④ Reducer 실행
        ↓
⑤ State 업데이트
```

Action 객체는 즉시 만들어지므로 이 과정에는 기다려야 하는 작업이 없습니다.

---

## 3. 비동기 작업은 다르다

상품 목록을 서버에서 가져온다고 생각해 보겠습니다.

```js
fetch("/api/products");
```

`fetch()`를 호출했다고 상품 데이터가 즉시 반환되는 것은 아닙니다.

```text
Request
클라이언트 → 서버

        ↓

Pending
응답 대기

        ↓

Response
서버 → 클라이언트
```

Promise 관점에서는 다음과 같습니다.

```text
Promise
   ↓
Pending
   ↓
┌──────────────┐
↓              ↓
Fulfilled    Rejected
```

따라서 비동기 작업에서는 API 요청을 실행하고 결과가 나올 때까지 기다릴 수 있는 별도의 로직이 필요합니다.

---

## 4. Middleware

Redux는 이러한 기능을 추가할 수 있도록 Middleware 구조를 제공합니다.

Middleware는 `dispatch()` 과정에 개입합니다.

```text
dispatch(...)
      ↓
Middleware Chain
      ↓
Reducer
```

Middleware가 여러 개라면 다음과 같은 체인이 만들어집니다.

```text
dispatch(...)
      ↓
Middleware
      ↓
Middleware
      ↓
Middleware
      ↓
Reducer
```

Middleware는 전달된 값을 검사하거나 로깅하거나 특정 작업을 수행한 뒤 다음 Middleware로 전달할 수 있습니다.

---

## 5. Thunk Middleware

Thunk Middleware도 이러한 Redux Middleware 중 하나입니다.

Thunk Middleware의 중요한 특징은 다음과 같습니다.

> dispatch된 값이 함수이면 그 함수를 실행합니다.

일반 Action 객체라면:

```text
dispatch(Action)
      ↓
Thunk Middleware
      ↓
Reducer
```

함수가 들어오면:

```text
dispatch(Thunk Function)
      ↓
Thunk Middleware
      ↓
Thunk Function 실행
```

따라서 Redux에서 비동기 작업을 수행할 공간을 만들 수 있습니다.

---

## 6. Thunk Function

Thunk는 `dispatch`할 수 있는 함수입니다.

대표적인 형태는 다음과 같습니다.

```js
const fetchProducts = () => {
  return async (dispatch, getState) => {
    // 비동기 작업
  };
};
```

외부에서는:

```js
dispatch(fetchProducts());
```

를 실행합니다.

`fetchProducts()`의 반환값은 Action 객체가 아니라 함수입니다.

```text
fetchProducts()
      ↓
Thunk Function
      ↓
dispatch(thunk)
      ↓
Thunk Middleware
      ↓
Thunk Function 실행
```

---

## 7. dispatch와 getState

Thunk Function에는 중요한 두 아규먼트가 전달됩니다.

```js
(dispatch, getState) => {
  // ...
}
```

`dispatch`는 새로운 Action이나 Thunk를 Redux에 전달할 때 사용합니다.

```js
dispatch(fetchSuccess(products));
```

`getState`는 현재 Redux Store의 State를 읽을 때 사용합니다.

```js
const state = getState();
```

예를 들어 인증 토큰이 Redux에 저장되어 있다면:

```js
const fetchProducts = () => {
  return async (dispatch, getState) => {
    const state = getState();
    const token = state.auth.token;

    const response = await fetch("/api/products", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    const products = await response.json();

    dispatch(fetchSuccess(products));
  };
};
```

따라서 두 함수의 역할은 다음과 같이 구분할 수 있습니다.

| 함수           | 역할                 |
| ------------ | ------------------ |
| `dispatch()` | Action 또는 Thunk 전달 |
| `getState()` | 현재 Redux State 조회  |

중요한 점은 Thunk가 직접 State를 변경하지 않는다는 것입니다.

```text
Thunk
 ↓
비동기 작업
 ↓
dispatch(Action)
 ↓
Reducer
 ↓
State 변경
```

---

# PART 4-2. Thunk의 실제 동작

## 직접 작성한 Thunk + Promise

이제 실제 Thunk를 작성해 보겠습니다.

상품 목록을 서버에서 가져온다고 가정합니다.

먼저 세 가지 Action이 있다고 하겠습니다.

```text
fetchStart()
fetchSuccess(products)
fetchFailure(error)
```

각 Action의 의미는 다음과 같습니다.

```text
fetchStart
→ 요청 시작

fetchSuccess
→ 요청 성공

fetchFailure
→ 요청 실패
```

---

## 1. 직접 작성한 Thunk

```js
export const fetchProducts = () => {
  return async (dispatch, getState) => {
    dispatch(fetchStart());

    try {
      const response = await fetch("/api/products");

      if (!response.ok) {
        throw new Error("Request failed");
      }

      const products = await response.json();

      dispatch(fetchSuccess(products));
    } catch (error) {
      dispatch(fetchFailure(error.message));
    }
  };
};
```

컴포넌트에서는 다음과 같이 실행할 수 있습니다.

```js
dispatch(fetchProducts());
```

---

## 2. 실제로 무엇이 일어나는가?

먼저:

```js
fetchProducts()
```

가 호출됩니다.

이 함수는 다음 함수를 반환합니다.

```js
async (dispatch, getState) => {
  // ...
}
```

따라서:

```js
dispatch(fetchProducts());
```

는 사실상 Thunk Function을 dispatch하는 것입니다.

```text
dispatch(Thunk Function)
        ↓
Thunk Middleware
        ↓
Thunk Function 실행
```

---

## 3. 요청 시작

Thunk가 실행되면 가장 먼저:

```js
dispatch(fetchStart());
```

가 실행됩니다.

`fetchStart()`는 일반 Action을 반환합니다.

따라서:

```text
dispatch(fetchStart())
        ↓
Reducer
        ↓
loading = true
error = null
```

가 됩니다.

여기서 중요한 사실이 하나 있습니다.

> 비동기 작업을 수행하는 동안에도 실제 Redux State 변경은 일반 Action과 Reducer를 통해 이루어집니다.

---

## 4. API Request

다음 코드가 실행됩니다.

```js
const response = await fetch("/api/products");
```

이 시점에서 Promise는 Pending 상태입니다.

```text
fetch()
   ↓
Promise Pending
   ↓
서버 응답 대기
```

서버의 응답이 도착하면 `await` 다음 코드로 진행합니다.

---

## 5. HTTP 오류 처리

여기서 `fetch()`의 중요한 특성을 알아야 합니다.

HTTP `404`, `500` 등의 응답이 왔다고 해서 `fetch()` Promise가 자동으로 reject되는 것은 아닙니다.

서버로부터 HTTP Response를 정상적으로 받았다면 Promise는 일반적으로 fulfilled됩니다.

따라서 직접 확인해야 합니다.

```js
if (!response.ok) {
  throw new Error("Request failed");
}
```

그러면 `catch`로 이동합니다.

```text
response.ok === false
        ↓
throw Error
        ↓
catch(error)
        ↓
dispatch(fetchFailure(...))
```

반대로 네트워크 자체에 문제가 발생하면 `fetch()` Promise 자체가 reject될 수 있으며 이 경우에도 `catch`로 이동합니다.

---

## 6. JSON 파싱도 비동기 작업이다

다음 코드도 Promise를 반환합니다.

```js
const products = await response.json();
```

따라서 개념적으로는:

```text
fetch()
 ↓
Promise Pending
 ↓
Response
 ↓
response.json()
 ↓
두 번째 Promise Pending
 ↓
JSON Parsing 완료
 ↓
products
```

의 과정입니다.

---

## 7. 성공

성공하면:

```js
dispatch(fetchSuccess(products));
```

를 실행합니다.

그러면:

```text
fetchSuccess(products)
        ↓
Action Object
        ↓
dispatch(Action)
        ↓
Reducer
        ↓
items = products
loading = false
```

가 됩니다.

---

## 8. 실패

실패하면:

```js
catch (error) {
  dispatch(fetchFailure(error.message));
}
```

가 실행됩니다.

```text
오류 발생
   ↓
catch(error)
   ↓
dispatch(fetchFailure(...))
   ↓
Reducer
   ↓
error 저장
loading = false
```

---

## 9. Promise와 Redux State는 같은 것이 아니다

여기서 중요한 개념이 있습니다.

Promise State:

```text
Pending
Fulfilled
Rejected
```

Redux State:

```js
{
  loading: false,
  data: [],
  error: null
}
```

둘은 서로 다른 상태입니다.

Promise가 Redux State를 직접 변경하지 않습니다.

중간에 Thunk와 Action이 존재합니다.

```text
Promise 결과
     ↓
Thunk
     ↓
dispatch(Action)
     ↓
Reducer
     ↓
Redux State
```

따라서 다음과 같이 이해하는 것이 정확합니다.

```text
요청 시작
→ dispatch(fetchStart())

요청 성공
→ dispatch(fetchSuccess(data))

요청 실패
→ dispatch(fetchFailure(error))
```

---


# PART 4-3. createAsyncThunk() + extraReducers

직접 Thunk를 작성하면 원리를 이해하기에는 좋지만 반복되는 코드가 많습니다.

앞의 코드에서는 우리가 직접:

```js
dispatch(fetchStart());
dispatch(fetchSuccess(products));
dispatch(fetchFailure(error));
```

를 작성했습니다.

Redux Toolkit은 이러한 전형적인 비동기 흐름을 자동화하는 `createAsyncThunk()`를 제공합니다.

---

# 1. createAsyncThunk()

기본 구조는 다음과 같습니다.

```js
export const fetchProducts = createAsyncThunk(
  "products/fetchProducts",
  async (_, thunkAPI) => {
    const response = await fetch("/api/products");

    if (!response.ok) {
      return thunkAPI.rejectWithValue("요청 실패");
    }

    return response.json();
  }
);
```

첫 번째 아규먼트:

```js
"products/fetchProducts"
```

는 **Action Type Prefix**입니다.

두 번째 아규먼트:

```js
async (_, thunkAPI) => {
  // ...
}
```

는 **Payload Creator**입니다.

실제 비동기 작업을 수행하는 함수입니다.

---

# 2. 자동 생성되는 3가지 Action

`createAsyncThunk()`의 핵심은 비동기 작업의 생명주기에 맞춰 세 가지 Action을 자동으로 생성한다는 것입니다.

```text
products/fetchProducts/pending

products/fetchProducts/fulfilled

products/fetchProducts/rejected
```

따라서 직접:

```js
fetchStart
fetchSuccess
fetchFailure
```

를 만드는 작업을 상당 부분 자동화할 수 있습니다.

전체적인 관계는 다음과 같습니다.

```text
dispatch(fetchProducts())
           ↓
       pending
           ↓
   Payload Creator
           ↓
     Promise 결과
        ↙     ↘
 fulfilled   rejected
```

---

# 3. pending

`dispatch(fetchProducts())`가 실행되면 먼저 `pending` Action이 자동 dispatch됩니다.

```text
products/fetchProducts/pending
```

Reducer에서는 일반적으로:

```js
state.loading = true;
state.error = null;
```

로 처리합니다.

---

# 4. fulfilled

Payload Creator가 정상적으로 값을 반환하면:

```js
return data;
```

`fulfilled` Action이 자동 dispatch됩니다.

반환값은:

```js
action.payload
```

에 들어갑니다.

즉:

```text
return data
    ↓
fulfilled Action
    ↓
action.payload = data
```

입니다.

---

# 5. rejected

실패 처리에는 중요한 두 가지 경우가 있습니다.

### rejectWithValue()

```js
return thunkAPI.rejectWithValue(data);
```

를 사용하면:

```text
rejected Action
      ↓
action.payload = data
```

가 됩니다.

서버가 전달한 에러 응답 등을 애플리케이션의 실패 데이터로 전달할 때 유용합니다.

### throw

반대로:

```js
throw new Error("Network Error");
```

처럼 예외를 발생시키면 주로:

```text
rejected Action
      ↓
action.error
```

를 통해 오류 정보가 전달됩니다.

따라서:

```text
정상 return
     ↓
action.payload
     ↓
fulfilled


rejectWithValue(value)
     ↓
action.payload
     ↓
rejected


throw Error
     ↓
action.error
     ↓
rejected
```

로 구분할 수 있습니다.

---

# 6. thunkAPI란?

Payload Creator의 두 번째 파라미터로 전달되는:

```js
thunkAPI
```

는 Redux Toolkit이 제공하는 객체입니다.

실무에서 특히 중요한 기능은 다음 세 가지입니다.

| 기능                           | 역할                                  |
| ---------------------------- | ----------------------------------- |
| `thunkAPI.dispatch()`        | 다른 Action 또는 Thunk dispatch         |
| `thunkAPI.getState()`        | 현재 Redux Store State 조회             |
| `thunkAPI.rejectWithValue()` | 사용자 정의 실패 데이터를 `action.payload`로 전달 |

예를 들어:

```js
async (_, thunkAPI) => {
  const state = thunkAPI.getState();

  thunkAPI.dispatch(logRequest());

  const response = await fetch("/api/products");

  if (!response.ok) {
    return thunkAPI.rejectWithValue("요청 실패");
  }

  return response.json();
}
```

앞에서 직접 작성한 Thunk의:

```js
(dispatch, getState)
```

와 연결해서 이해하면 됩니다.

---

# 7. extraReducers

`createAsyncThunk()`가 Action을 자동으로 만들어도 이 Action을 받았을 때 State를 어떻게 변경할지는 Reducer가 정의해야 합니다.

여기서 `extraReducers`를 사용합니다.

```js
extraReducers: (builder) => {
  builder
    .addCase(fetchProducts.pending, (state) => {
      state.loading = true;
      state.error = null;
    })

    .addCase(fetchProducts.fulfilled, (state, action) => {
      state.loading = false;
      state.items = action.payload;
    })

    .addCase(fetchProducts.rejected, (state, action) => {
      state.loading = false;
      state.error =
        action.payload || action.error.message;
    });
}
```

흐름은 다음과 같습니다.

```text
pending
   ↓
extraReducers
   ↓
loading = true


fulfilled
   ↓
extraReducers
   ↓
items = action.payload
loading = false


rejected
   ↓
extraReducers
   ↓
error 저장
loading = false
```

---

# 8. reducers와 extraReducers

두 개의 차이를 이해해야 합니다.

`reducers`는 해당 Slice가 직접 정의하고 생성하는 Action을 처리합니다.

```js
reducers: {
  clearProducts(state) {
    state.items = [];
  }
}
```

반면 `extraReducers`는 다른 곳에서 만들어진 Action에도 반응할 수 있습니다.

대표적인 예가:

```text
fetchProducts.pending
fetchProducts.fulfilled
fetchProducts.rejected
```

입니다.

즉:

```text
reducers
→ 이 Slice에서 정의한 Action


extraReducers
→ 외부에서 생성된 Action에도 반응
```

이라고 이해하면 됩니다.

---

# 9. dispatch(fetchProducts())의 반환값

여기에는 실무에서 매우 중요한 특징이 있습니다.

```js
const result = await dispatch(fetchProducts());
```

`createAsyncThunk`로 생성한 Thunk를 dispatch하면 결과 Promise는 일반적으로 **reject되지 않고 resolve**되며, 최종 `fulfilled` 또는 `rejected` Action 객체를 결과로 제공합니다.

따라서 다음 코드의 `catch`가 단순히 `rejected Action` 때문에 실행되는 것은 아닙니다.

```js
try {
  await dispatch(fetchProducts());
} catch (error) {
  // rejected Action이라고 자동으로 여기로 오지 않음
}
```

---

# 10. unwrap()

컴포넌트에서 일반적인 Promise처럼 성공과 실패를 처리하고 싶다면 `.unwrap()`을 사용할 수 있습니다.

```js
try {
  const products =
    await dispatch(fetchProducts()).unwrap();

  console.log(products);
} catch (error) {
  console.error(error);
}
```

성공하면 fulfilled Action의 payload를 반환합니다.

```text
fulfilled
   ↓
unwrap()
   ↓
action.payload 반환
```

실패하면 오류 값을 throw합니다.

```text
rejected
   ↓
unwrap()
   ↓
rejectWithValue의 payload
또는 error
   ↓
throw
   ↓
catch
```

따라서 컴포넌트에서 요청 성공 후 페이지 이동 등의 명령형 로직이 필요할 때 유용합니다.

---

# 11. createAsyncThunk 전체 실행 흐름

지금까지를 하나로 연결해 보겠습니다.

```text
① dispatch(fetchProducts())
           ↓
② Thunk 생성
           ↓
③ Thunk Middleware
           ↓
④ pending Action dispatch
           ↓
⑤ Payload Creator 실행
           ↓
⑥ Promise Pending
           ↓
⑦ Promise 결과
        ↙       ↘
      성공       실패
       ↓          ↓
⑧-A fulfilled  ⑧-B rejected
       ↘          ↙
        ⑨ Reducer
            ↓
       ⑩ New State
```

`fulfilled`와 `rejected`는 순서대로 실행되는 것이 아닙니다.

**성공하면 fulfilled, 실패하면 rejected 중 하나가 선택됩니다.**


---

# PART 4-4. React에서의 전체 실행 흐름

이제 마지막으로 지금까지 배운 내용을 React와 연결해 보겠습니다.

상품 목록 컴포넌트가 있다고 하겠습니다.

```js
function ProductList() {
  const dispatch = useDispatch();

  const { items, loading, error } = useSelector(
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
      {items.map(p => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

이 짧은 코드 뒤에서는 상당히 많은 작업이 일어납니다.

---

# 1. 최초 Render

React가 `ProductList`를 렌더링합니다.

```text
ProductList()
     ↓
최초 Render
```

아직 서버 데이터가 없을 수 있습니다.

---

# 2. useEffect 실행

컴포넌트가 마운트된 후 Effect가 실행됩니다.

```js
useEffect(() => {
  dispatch(fetchProducts());
}, [dispatch]);
```

여기서 Redux 비동기 흐름이 시작됩니다.

---

# 3. Thunk dispatch

```js
dispatch(fetchProducts());
```

가 실행됩니다.

```text
React Component
      ↓
dispatch(fetchProducts())
      ↓
Thunk Middleware
```

Thunk Middleware가 Thunk Function을 실행합니다.

---

# 4. pending Action

`createAsyncThunk()`는 먼저 `pending` Action을 dispatch합니다.

```text
fetchProducts.pending
```

`extraReducers`가 이를 처리합니다.

```js
state.loading = true;
state.error = null;
```

Store의 State가 변경됩니다.

---

# 5. React-Redux가 Store 변경을 전달한다

Store가 변경되면 React-Redux의 subscription 구조를 통해 관련 컴포넌트에 업데이트가 전달됩니다.

`useSelector()`는 selector 결과를 기준으로 필요한 리렌더링을 결정합니다.

따라서:

```text
Redux Store 변경
      ↓
React-Redux Subscription
      ↓
selector 결과 확인
      ↓
관련 컴포넌트 Re-render
```

가 됩니다.

화면에는:

```text
Loading...
```

이 나타납니다.

---

# 6. 서버 응답

그동안 Payload Creator에서는 API 요청이 진행되고 있습니다.

```js
const response = await fetch("/api/products");
```

서버 응답이 도착하고 데이터 처리가 성공하면:

```text
fulfilled Action
```

이 자동 dispatch됩니다.

---

# 7. fulfilled Action 처리

`extraReducers`가:

```js
fetchProducts.fulfilled
```

를 처리합니다.

```js
state.items = action.payload;
state.loading = false;
```

그러면 Redux Store가 다시 변경됩니다.

---

# 8. React 화면 업데이트

React-Redux가 Store 변경을 구독 컴포넌트에 전달하고 selector 결과가 변경되면 `ProductList`가 다시 렌더링됩니다.

```text
fulfilled
   ↓
extraReducers
   ↓
Redux Store
   ↓
React-Redux Subscription
   ↓
useSelector 결과 변경
   ↓
Component Re-render
   ↓
상품 목록 표시
```

이것이 **React → Redux → 비동기 작업 → Redux → React**의 전체 연결입니다.

---

# 9. React 18 StrictMode 주의

개발 환경에서 React의 `StrictMode`를 사용하면 Effect의 문제를 발견하기 위해 setup/cleanup 사이클이 추가로 실행될 수 있습니다.

따라서 다음처럼 Effect에서 직접 요청을 시작하면:

```js
useEffect(() => {
  dispatch(fetchProducts());
}, [dispatch]);
```

개발 과정에서 동일한 요청이 반복되는 것처럼 보일 수 있습니다.

중요한 것은 이것을:

> React에서는 API가 항상 두 번 호출된다.

라고 이해하면 안 된다는 것입니다.

`StrictMode`의 개발 환경 검사와 관련된 동작이며 프로덕션에서 같은 이유로 두 번 실행되는 것은 아닙니다.

하지만 여기서 더 근본적인 문제가 드러납니다.

서버 데이터를 직접 관리하기 시작하면 단순한 API 호출 이외에도 다음 문제들을 고려해야 합니다.

```text
Caching
Request Deduplication
Synchronization
Cache Invalidation
Refetch
Subscription
Loading / Error
```

그리고 이것이 **RTK Query가 등장하는 이유**와 연결됩니다.

---

# 10. Client State와 Server State

애플리케이션의 State를 크게 두 종류로 나누어 생각해 볼 수 있습니다.

## Client State

클라이언트 자체가 소유하는 상태입니다.

예:

```text
모달 열림 여부
사이드바 상태
선택된 탭
검색 조건
Wizard 단계
폼 입력값
```

이러한 State는 UI에서 만들어지고 UI가 직접 관리합니다.

Redux Toolkit에서는 일반적으로:

```js
createSlice()
```

로 관리하기 좋습니다.

---

## Server State

서버가 원본을 가지고 있는 데이터입니다.

예:

```text
상품 목록
회원 정보
주문 정보
게시글
댓글
재고 정보
카테고리 정보
```

클라이언트에 있는 것은 서버 데이터의 복사본 또는 캐시라고 볼 수 있습니다.

```text
Database / Server
       ↕
      HTTP
       ↕
     Client
       ↓
Cached Server Data
```

따라서 Server State는 Client State와 다른 문제가 발생합니다.

---

# 11. Server State가 복잡한 이유

상품 목록 하나를 가져오는 것만 생각하면:

```js
fetch("/api/products");
```

로 간단해 보입니다.

하지만 실제 애플리케이션에서는 다음을 관리해야 합니다.

```text
Fetching
Caching
Request Deduplication
Synchronization
Cache Invalidation
Refetch
Subscription
Loading
Error
```

예를 들어 이미 상품 데이터를 가져왔다면:

> 다시 서버에 요청해야 하는가?

다른 컴포넌트도 같은 데이터를 요청한다면:

> 요청을 두 번 보내야 하는가?

상품을 수정했다면:

> 기존 캐시는 언제 무효화해야 하는가?

사용자가 화면으로 돌아왔다면:

> 서버 데이터를 다시 가져와야 하는가?

이러한 문제까지 모두 직접 처리하면 `createAsyncThunk()` 코드가 빠르게 복잡해질 수 있습니다.

---

# 12. createAsyncThunk가 적합한 경우

그렇다고 `createAsyncThunk()`가 필요 없어지는 것은 아닙니다.

복잡한 비즈니스 로직이나 여러 단계의 비동기 Workflow를 직접 제어해야 할 때 매우 유용합니다.

예를 들어:

```text
주문 생성
 ↓
결제 요청
 ↓
결제 성공 확인
 ↓
재고 처리
 ↓
주문 상태 변경
 ↓
페이지 이동
```

처럼 여러 작업을 순서대로 제어해야 한다면 `createAsyncThunk()`가 적합할 수 있습니다.

---

# 13. Server State와 RTK Query

반대로 핵심 문제가:

```text
서버 데이터 조회
캐싱
동일 요청 중복 방지
데이터 재사용
자동 Refetch
Cache Invalidation
Subscription
Loading / Error
```

이라면 RTK Query가 훨씬 적합합니다.

RTK Query는 Server State 관리에 필요한 기능을 제공합니다.

```text
Automatic Caching

Request Deduplication

Cache Invalidation

Background Refetch

Subscription Management

Loading / Error Handling
```

따라서:

```text
Client State
→ createSlice()


복잡한 비동기 Workflow
→ createAsyncThunk()


Server State
→ RTK Query
```

라는 큰 그림을 잡을 수 있습니다.

이것은 절대적인 규칙이라기보다 각 도구가 특히 잘 해결하는 문제 영역을 이해하기 위한 구분입니다.

---




# 최종 핵심

PART 4 전체를 한 문장으로 압축하면 다음과 같습니다.

> **Redux에서 Thunk는 Reducer 밖에서 비동기 작업을 수행하고 그 결과를 Action으로 Redux 흐름에 다시 연결하며, `createAsyncThunk()`는 이 패턴을 `pending / fulfilled / rejected` Action으로 자동화하고, Server State 관리가 중심이 되면 RTK Query를 사용할 수 있습니다.**

그리고 반드시 역할을 구분해서 기억해야 합니다.

| 구성 요소              | 핵심 역할                                            |
| ------------------ | ------------------------------------------------ |
| `Reducer`          | State 계산                                         |
| `Middleware`       | dispatch 과정 확장                                   |
| `Thunk Middleware` | dispatch된 함수 실행                                  |
| `Thunk Function`   | 비동기 작업과 Side Effect 수행                           |
| `dispatch`         | Action 또는 Thunk 전달                               |
| `getState`         | 현재 Redux State 조회                                |
| `createAsyncThunk` | 비동기 생명주기 Action 자동 생성                            |
| `extraReducers`    | 외부에서 생성된 Action에 반응하여 State 변경                   |
| `React-Redux`      | Redux Store와 React 연결                            |
| `RTK Query`        | Server State fetching/caching/synchronization 관리 |

이 흐름을 이해했다면 다음 **PART 5. RTK Query**에서는 단순히 새로운 API를 배우는 것이 아니라, **PART 4에서 직접 관리했던 Server State 관련 반복 작업을 RTK Query가 어떻게 대신해 주는가**를 중심으로 들어가는 것이 가장 자연스럽습니다.
