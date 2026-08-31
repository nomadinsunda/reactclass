# PART 4-3. createAsyncThunk() + extraReducers 완전 이해

## 1. 이번 PART에서 이해해야 할 핵심

앞에서 우리는 Redux에서 비동기 작업을 처리하기 위해 Thunk를 사용한다는 것을 배웠습니다.

직접 Thunk를 작성하면 대략 다음과 같은 형태가 됩니다.

```js
const fetchUser = (userId) => {
  return async (dispatch, getState) => {
    // 요청 시작
    dispatch(fetchStart());

    try {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();

      // 성공
      dispatch(fetchSuccess(data));
    } catch (error) {
      // 실패
      dispatch(fetchFailure(error.message));
    }
  };
};
```

이 방식의 핵심은 간단합니다.

```text
비동기 작업 수행
        ↓
결과를 Action으로 변환
        ↓
dispatch(Action)
        ↓
Reducer
        ↓
State 변경
```

문제는 비동기 작업을 만들 때마다 `Start`, `Success`, `Failure` Action을 반복해서 만들어야 한다는 것입니다.

Redux Toolkit은 이 반복 구조를 `createAsyncThunk()`로 자동화합니다.

그리고 여기서 생성된 Action에 Slice가 반응하도록 만드는 것이 바로 `extraReducers`입니다.

---

# 2. 핵심 원리: Thunk와 Reducer의 역할을 분리한다

이미지의 첫 번째 영역에서 가장 중요한 내용입니다.

Redux의 Reducer는 다음 역할만 담당합니다.

```text
(currentState, action)
        ↓
     newState
```

즉 Reducer는 **현재 State와 Action을 보고 새로운 State를 계산하는 역할**에 집중합니다.

반면 다음과 같은 작업은 Reducer의 책임이 아닙니다.

```text
API Request
Timer
Logging
파일 접근
외부 시스템 접근
```

이러한 작업은 Side Effect입니다.

예를 들어 Reducer 내부에서 다음과 같이 작성하면 안 됩니다.

```js
function reducer(state, action) {
  const response = await fetch("/api/users");

  // ...
}
```

비동기 네트워크 작업은 실행 시간과 결과를 Reducer가 예측할 수 없기 때문입니다.

따라서 Redux에서는 역할을 분리합니다.

```text
Thunk
비동기 작업 + Side Effect
        ↓
      Action
        ↓
Reducer
State 계산
```

핵심은 다음 한 문장입니다.

> **Thunk는 비동기 작업을 수행하고, Reducer는 그 결과를 Action으로 받아 State만 계산합니다.**

이 역할 분리가 이후 `createAsyncThunk()`와 `extraReducers` 구조를 이해하는 출발점입니다.

---

# 3. createAsyncThunk()는 무엇을 자동화하는가?

직접 Thunk를 작성하면 보통 세 가지 Action이 필요합니다.

```text
요청 시작
요청 성공
요청 실패
```

예를 들면:

```js
dispatch(fetchStart());

dispatch(fetchSuccess(data));

dispatch(fetchFailure(error));
```

`createAsyncThunk()`는 이 패턴을 자동화합니다.

```js
export const fetchUserById = createAsyncThunk(
  "user/fetchById",

  async (userId, thunkAPI) => {
    const response =
      await fetch(`/api/users/${userId}`);

    if (!response.ok) {
      return thunkAPI.rejectWithValue(
        await response.text()
      );
    }

    return response.json();
  }
);
```

첫 번째 아규먼트:

```js
"user/fetchById"
```

는 Action Type Prefix입니다.

두 번째 아규먼트:

```js
async (userId, thunkAPI) => {
  ...
}
```

는 실제 비동기 작업을 수행하는 **Payload Creator**입니다.

---

# 4. createAsyncThunk()가 자동으로 만드는 3개의 Action

다음 Thunk를 만들었다고 하겠습니다.

```js
const fetchUserById = createAsyncThunk(
  "user/fetchById",
  async (userId) => {
    // ...
  }
);
```

Redux Toolkit은 자동으로 다음 세 종류의 Action Creator를 제공합니다.

```js
fetchUserById.pending

fetchUserById.fulfilled

fetchUserById.rejected
```

각각의 Action Type은 개념적으로 다음과 같습니다.

```text
user/fetchById/pending

user/fetchById/fulfilled

user/fetchById/rejected
```

즉 개발자가 직접 다음 코드를 만들 필요가 없습니다.

```js
fetchStart()
fetchSuccess()
fetchFailure()
```

`createAsyncThunk()`가 비동기 작업의 생명주기에 맞춰 자동으로 처리합니다.

---

# 5. pending은 언제 발생하는가?

컴포넌트에서 다음 코드를 실행했다고 하겠습니다.

```js
dispatch(fetchUserById(1));
```

먼저 Thunk Middleware가 Thunk Function을 실행합니다.

그리고 Payload Creator의 비동기 작업을 시작하기 전에 `pending` Action이 자동으로 dispatch됩니다.

```text
dispatch(fetchUserById(1))
          ↓
Thunk Middleware
          ↓
pending Action
          ↓
Payload Creator 실행
          ↓
API Request
```

따라서 `pending`은 일반적으로 다음 상태를 만드는 데 사용합니다.

```js
state.loading = true;
state.error = null;
```

UI 관점에서는:

```text
"요청을 시작했으므로 로딩 화면을 보여라."
```

라는 의미입니다.

---

# 6. Payload Creator가 실제 비동기 작업을 수행한다

`pending` Action이 발생한 뒤 Payload Creator가 실행됩니다.

```js
async (userId, thunkAPI) => {
  const response =
    await fetch(`/api/users/${userId}`);

  if (!response.ok) {
    return thunkAPI.rejectWithValue(
      await response.text()
    );
  }

  return response.json();
}
```

여기가 실제 Side Effect가 발생하는 영역입니다.

```text
Payload Creator
      ↓
fetch()
      ↓
서버 요청
      ↓
Promise 대기
```

그리고 작업 결과에 따라 두 방향으로 나뉩니다.

```text
              Payload Creator
                    ↓
              비동기 작업
                    ↓
             Promise 결과
              /         \
           성공           실패
            ↓              ↓
       fulfilled        rejected
```

여기서 중요한 점은 **`fulfilled`와 `rejected`가 순차적으로 실행되는 것이 아니라 둘 중 하나만 발생한다는 것**입니다.

---

# 7. 성공하면 fulfilled Action이 발생한다

Payload Creator에서 정상적으로 값을 반환했다고 하겠습니다.

```js
return data;
```

그러면 `createAsyncThunk()`가 자동으로 `fulfilled` Action을 dispatch합니다.

그리고 반환값은:

```js
action.payload
```

에 저장됩니다.

즉:

```text
return data
    ↓
fulfilled Action
    ↓
action.payload = data
```

가 됩니다.

따라서 Reducer에서는 다음처럼 사용할 수 있습니다.

```js
.addCase(
  fetchUserById.fulfilled,
  (state, action) => {
    state.loading = false;
    state.data = action.payload;
    state.error = null;
  }
)
```

---

# 8. 실패하면 rejected Action이 발생한다

실패는 크게 두 가지 방법으로 발생할 수 있습니다.

일반적인 예외를 발생시키는 방법:

```js
throw new Error("Network Error");
```

이 경우 에러 정보는 주로:

```js
action.error
```

에 들어갑니다.

반면:

```js
return thunkAPI.rejectWithValue(errorData);
```

를 사용하면 개발자가 전달한 값이:

```js
action.payload
```

에 들어갑니다.

따라서 두 방식은 다음과 같이 구분할 수 있습니다.

```text
throw Error
    ↓
rejected
    ↓
action.error


rejectWithValue(value)
    ↓
rejected
    ↓
action.payload
```

그래서 `rejected` Reducer를 다음처럼 작성하는 경우가 많습니다.

```js
.addCase(
  fetchUserById.rejected,
  (state, action) => {
    state.loading = false;

    state.error =
      action.payload ??
      action.error.message;
  }
);
```

---

# 9. fetch()의 중요한 특징

여기서 반드시 알아야 할 JavaScript의 특징이 있습니다.

`fetch()`는 서버가 HTTP 404나 500을 반환했다고 해서 Promise를 자동으로 reject하지 않습니다.

예를 들어:

```js
const response =
  await fetch("/api/users/999");
```

서버가:

```text
404 Not Found
```

를 반환해도 HTTP 응답 자체를 정상적으로 받았다면 `fetch()`의 Promise는 이행될 수 있습니다.

따라서 직접 확인해야 합니다.

```js
if (!response.ok) {
  throw new Error("사용자 조회 실패");
}
```

또는 서버에서 받은 에러 정보를 Redux로 전달하려면:

```js
if (!response.ok) {
  return thunkAPI.rejectWithValue(
    await response.text()
  );
}
```

처럼 처리할 수 있습니다.

---

# 10. thunkAPI는 무엇인가?

Payload Creator의 두 번째 파라미터를 다시 보겠습니다.

```js
async (userId, thunkAPI) => {
  ...
}
```

`thunkAPI`는 `createAsyncThunk()`가 Payload Creator에게 제공하는 객체입니다.

여러 기능이 있지만 처음에는 다음 세 가지를 이해하면 충분합니다.

```text
thunkAPI.dispatch
thunkAPI.getState
thunkAPI.rejectWithValue
```

### `thunkAPI.dispatch(action)`

Thunk 안에서 다른 Action을 dispatch할 수 있습니다.

```js
thunkAPI.dispatch(logRequest());
```

### `thunkAPI.getState()`

현재 Redux Store의 State를 조회합니다.

```js
const state = thunkAPI.getState();

const token = state.auth.token;
```

예를 들어 API 요청 전에 현재 로그인 토큰을 가져올 수 있습니다.

```js
const state = thunkAPI.getState();

const response = await fetch(
  `/api/users/${userId}`,
  {
    headers: {
      Authorization:
        `Bearer ${state.auth.token}`,
    },
  }
);
```

### `thunkAPI.rejectWithValue(value)`

개발자가 원하는 실패 정보를 `rejected` Action의 `payload`로 전달합니다.

```js
return thunkAPI.rejectWithValue(
  "사용자를 찾을 수 없습니다."
);
```

---

# 11. 이제 extraReducers가 필요한 이유

여기까지 이해하면 자연스럽게 다음 질문이 생깁니다.

> `createAsyncThunk()`가 `pending`, `fulfilled`, `rejected` Action을 만들었다면 누가 이 Action을 받아 State를 변경하는가?

바로 `extraReducers`입니다.

예를 들어:

```js
const userSlice = createSlice({
  name: "user",

  initialState: {
    data: null,
    loading: false,
    error: null,
  },

  reducers: {
    resetUser(state) {
      state.data = null;
    },
  },

  extraReducers: (builder) => {
    // ...
  },
});
```

여기서 `extraReducers`는 **`createSlice()`에 전달하는 JavaScript 객체의 프로퍼티**입니다.

그리고 역할은 명확합니다.

> **이 Slice가 직접 생성하지 않은 외부 Action에 이 Slice가 어떻게 반응할지를 정의합니다.**

---

# 12. reducers와 extraReducers의 결정적인 차이

둘의 차이를 단순히 "동기 vs 비동기"라고 외우면 정확하지 않습니다.

더 본질적인 차이는 **Action을 누가 생성하느냐**입니다.

### reducers

```js
reducers: {
  resetUser(state) {
    state.data = null;
  }
}
```

`createSlice()`는 여기에서 Reducer뿐 아니라 Action Creator도 자동 생성합니다.

```js
userSlice.actions.resetUser
```

즉:

```text
reducers
    ↓
Case Reducer 정의
    +
Action Creator 생성
```

입니다.

### extraReducers

```js
extraReducers: (builder) => {
  builder.addCase(
    fetchUserById.pending,
    (state) => {
      state.loading = true;
    }
  );
}
```

`extraReducers`는 새로운 Action Creator를 만들지 않습니다.

이미 존재하는 Action을 받아 처리합니다.

```text
외부에서 만들어진 Action
          ↓
     extraReducers
          ↓
   State 변경 로직 실행
```

따라서 가장 중요한 차이는 다음과 같습니다.

| 구분                | `reducers`   | `extraReducers`           |
| ----------------- | ------------ | ------------------------- |
| Case Reducer 정의   | O            | O                         |
| Action Creator 생성 | O            | X                         |
| Action 출처         | Slice가 직접 생성 | 이미 존재하는 외부 Action         |
| 대표적인 사용           | `resetUser`  | `fetchUserById.fulfilled` |

`extraReducers`가 비동기 전용 기능인 것은 아닙니다.

다른 Slice가 만든 일반 Action에도 반응할 수 있습니다.

---

# 13. builder.addCase()는 무엇을 하는가?

`extraReducers`에서는 보통 다음과 같이 작성합니다.

```js
extraReducers: (builder) => {
  builder
    .addCase(
      fetchUserById.pending,
      (state) => {
        state.loading = true;
        state.error = null;
      }
    )
    .addCase(
      fetchUserById.fulfilled,
      (state, action) => {
        state.loading = false;
        state.data = action.payload;
        state.error = null;
      }
    )
    .addCase(
      fetchUserById.rejected,
      (state, action) => {
        state.loading = false;
        state.error =
          action.payload ??
          action.error.message;
      }
    );
}
```

`addCase()`의 의미는 매우 단순합니다.

```text
builder.addCase(
    어떤 Action,
    실행할 Reducer
)
```

즉:

> **"이 Action이 dispatch되면 이 Reducer를 실행하라."**

라는 등록 작업입니다.

예를 들어:

```js
builder.addCase(
  fetchUserById.fulfilled,
  reducerFn
);
```

은 개념적으로:

```text
fetchUserById.fulfilled
Action 발생
        ↓
reducerFn 실행
        ↓
State 변경
```

입니다.

---

# 14. extraReducers는 Action을 만드는 곳이 아니다

이 부분은 반드시 구분해야 합니다.

```js
extraReducers: (builder) => {
  builder.addCase(
    fetchUserById.fulfilled,
    ...
  );
}
```

이 코드가:

```text
fulfilled Action을 생성한다
```

는 뜻이 아닙니다.

Action은 이미 `createAsyncThunk()`가 만들었습니다.

```text
createAsyncThunk()
        ↓
pending
fulfilled
rejected
Action Creator 생성
```

`extraReducers`는:

```text
이미 만들어진 Action
        ↓
어떻게 State를 변경할 것인가?
```

를 정의합니다.

따라서 다음처럼 기억하면 됩니다.

> **`createAsyncThunk()`는 Action을 만들고, `extraReducers`는 그 Action에 반응합니다.**

---

# 15. 비동기 State는 어떻게 모델링하는가?

서버 데이터를 가져오는 화면은 단순히 `data` 하나만으로 상태를 표현하기 어렵습니다.

일반적으로 다음 세 가지가 필요합니다.

```js
const initialState = {
  data: null,
  loading: false,
  error: null,
};
```

각 필드의 역할은 다음과 같습니다.

```text
data
서버에서 받은 실제 데이터

loading
현재 요청이 진행 중인가?

error
요청이 실패했는가?
```

이 상태들은 `pending / fulfilled / rejected`와 자연스럽게 대응됩니다.

---

# 16. 3단계 State 전이

### pending

```js
state.loading = true;
state.error = null;
```

상태는:

```text
요청 시작
loading = true
error = null
```

이 됩니다.

기존 `data`를 반드시 삭제할 필요는 없습니다.

---

### fulfilled

```js
state.loading = false;
state.data = action.payload;
state.error = null;
```

상태는:

```text
요청 성공
loading = false
data = 서버 데이터
error = null
```

이 됩니다.

---

### rejected

```js
state.loading = false;

state.error =
  action.payload ??
  action.error.message;
```

상태는:

```text
요청 실패
loading = false
error = 에러 정보
```

가 됩니다.

실패했을 때 기존 `data`를 유지할지 `null`로 변경할지는 애플리케이션 정책에 따라 결정합니다.

---

# 17. 직접 State를 변경하는 것처럼 보이는 이유

다음 코드를 보면 Redux 규칙에 위배되는 것처럼 보일 수 있습니다.

```js
state.loading = true;
```

Redux에서는 원래 State를 직접 변경하면 안 됩니다.

하지만 `createSlice()`의 Case Reducer에서 전달받는 `state`는 Immer가 관리하는 **draft State**입니다.

따라서 개발자는:

```js
state.loading = true;
```

처럼 작성할 수 있습니다.

개념적으로는:

```text
state.loading = true
        ↓
Immer가 변경 내용 추적
        ↓
불변성을 유지한
새로운 State 생성
```

입니다.

즉 코드 작성 방식은 mutation처럼 보이지만 Redux Store의 기존 State를 그대로 파괴하는 방식은 아닙니다.

---

# 18. 전체 실행 흐름

이제 이미지의 전체 흐름을 연결해 보겠습니다.

```text
1. React Component
        ↓
2. dispatch(fetchUserById())
        ↓
3. Thunk Middleware
        ↓
4. pending Action 자동 dispatch
        ↓
5. Payload Creator 실행
        ↓
6. API Request / Promise 대기
        ↓
        결과
       /    \
    성공      실패
     ↓         ↓
7-A fulfilled  7-B rejected
       \       /
        \     /
         ↓   ↓
8. extraReducers
   builder.addCase()
        ↓
9. Redux Store
   State 변경
        ↓
10. React-Redux / useSelector
    변경 감지
        ↓
11. Component Re-render
```

여기서 특히 중요한 것은:

```text
fulfilled
```

와

```text
rejected
```

가 연속해서 발생하는 것이 아니라는 점입니다.

둘은 **성공/실패에 따른 분기**입니다.

```text
             Promise 결과
              /        \
           성공          실패
            ↓             ↓
       fulfilled       rejected
              \        /
               ↓      ↓
             Reducer
```

---

# 19. 컴포넌트에서 결과를 직접 처리하고 싶다면 unwrap()

`createAsyncThunk()`로 만든 Thunk를 dispatch하면 Promise 형태의 결과를 받을 수 있습니다.

```js
const resultAction =
  await dispatch(fetchUserById(1));
```

하지만 컴포넌트에서 일반적인 Promise처럼:

```js
try {
  ...
} catch {
  ...
}
```

형태로 성공과 실패를 처리하고 싶다면 `.unwrap()`이 유용합니다.

```js
try {
  const user =
    await dispatch(
      fetchUserById(1)
    ).unwrap();

  console.log(user);
} catch (error) {
  console.error(error);
}
```

성공하면:

```text
fulfilled
    ↓
action.payload
    ↓
unwrap()
    ↓
payload 반환
```

실패하면:

```text
rejected
    ↓
unwrap()
    ↓
rejectWithValue의 payload
또는 serialized error를 throw
```

가 됩니다.

따라서 컴포넌트에서 후속 처리가 필요할 때 특히 유용합니다.

```js
try {
  await dispatch(saveUser(user)).unwrap();

  navigate("/users");
} catch (error) {
  alert(error);
}
```

---

# 20. 전체 구조를 세 문장으로 정리

첫 번째:

> **Thunk는 API 요청과 같은 Side Effect를 담당하고 Reducer는 State 계산에 집중합니다.**

두 번째:

> **`createAsyncThunk()`는 비동기 작업의 진행 결과를 `pending / fulfilled / rejected` Action으로 자동 변환합니다.**

세 번째:

> **`extraReducers`는 `createSlice()` 객체의 프로퍼티로서, Slice가 직접 생성하지 않은 외부 Action에 어떻게 반응하여 자신의 State를 변경할지를 정의합니다.**

따라서 전체 구조는 결국 다음과 같습니다.

```text
createAsyncThunk()
       │
       │ 비동기 작업
       ▼
pending / fulfilled / rejected
       │
       │ Action
       ▼
extraReducers
       │
       │ builder.addCase()
       ▼
Case Reducer
       │
       ▼
Redux State
       │
       ▼
React UI
```

## 최종 핵심

`createAsyncThunk()`와 `extraReducers`를 서로 별개의 문법으로 외우면 구조가 복잡해 보입니다.

두 기능의 관계를 **생산자와 소비자** 관점에서 보면 훨씬 단순합니다.

```text
createAsyncThunk
    =
비동기 작업 수행
+
생명주기 Action 생성


extraReducers
    =
그 Action을 감지
+
State 변경
```

즉,

> **`createAsyncThunk()`는 비동기 작업을 Redux가 이해할 수 있는 Action으로 변환하고, `extraReducers`는 그 Action을 State 변화로 변환합니다.**

이 문장이 이번 PART의 핵심입니다.
