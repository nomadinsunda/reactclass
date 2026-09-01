# RTK Query 완벽 이해

## Redux에서 서버 데이터를 어떻게 관리할 것인가?

RTK Query는 **Redux Toolkit에 포함된 서버 상태(Server State) 데이터 패칭 및 캐싱 도구**입니다.

```text
Redux Toolkit
│
├─ configureStore
├─ createSlice
├─ createAsyncThunk
│
└─ RTK Query
    ├─ createApi
    ├─ fetchBaseQuery
    ├─ Query
    ├─ Mutation
    ├─ Cache
    ├─ Subscription
    ├─ Tag Invalidation
    └─ React Hooks
```

RTK Query를 단순히 다음과 같이 이해해서는 부족합니다.

```text
RTK Query
= fetch를 편하게 해주는 도구
```

더 정확하게는 다음과 같습니다.

```text
RTK Query
=
서버 요청
+ Redux Store 기반 캐시
+ 요청 상태 관리
+ 중복 요청 제거
+ Subscription 관리
+ Cache Lifecycle 관리
+ Cache Invalidation
+ 자동 Re-fetch
+ React Hook 생성
```

즉 RTK Query의 핵심은 **“서버에서 데이터를 가져오는 것” 자체보다 “가져온 서버 데이터를 어떻게 캐시하고 동기화할 것인가”** 에 있습니다.

---

# 1. Client State와 Server State

RTK Query를 이해하려면 먼저 Redux가 다루는 상태를 구분할 필요가 있습니다.

### Client State

애플리케이션 내부에서 만들어지고 관리되는 상태입니다.

```text
로그인 모달 열림 여부
현재 선택한 탭
사이드바 열림 여부
테마
로컬 필터 값
```

예를 들어:

```js
const initialState = {
  sidebarOpen: false,
};
```

이런 데이터는 일반적인 `createSlice()`로 관리하기 적합합니다.

반면 다음과 같은 데이터가 있습니다.

```text
상품 목록
회원 정보
게시글
댓글
주문 내역
서버 검색 결과
```

이 데이터의 **실제 소유자는 프론트엔드가 아니라 서버**입니다.

이를 Server State라고 생각할 수 있습니다.

```text
Server
   │
   │ HTTP
   ▼
Client
```

클라이언트가 가지고 있는 것은 서버 데이터의 **복사본 또는 캐시**일 뿐입니다.

따라서 Server State를 관리하려면 단순히 값을 저장하는 것 이상의 문제가 발생합니다.

```text
언제 가져올 것인가?
이미 가져온 데이터가 있는가?
다른 컴포넌트도 같은 데이터를 요청하고 있는가?
얼마 동안 보관할 것인가?
서버 데이터가 변경되면 어떻게 갱신할 것인가?
요청 중인가?
실패했는가?
다시 요청해야 하는가?
```

RTK Query는 바로 이 문제들을 해결하기 위해 만들어졌습니다.

---

# 2. 기존 Redux 방식의 문제

예를 들어 서버에서 게시글 목록을 가져온다고 하겠습니다.

```http
GET /posts
```

`createAsyncThunk()`를 사용한다면 보통 다음과 같은 상태가 필요합니다.

```js
const initialState = {
  posts: [],
  loading: false,
  error: null,
};
```

Thunk를 정의합니다.

```js
export const fetchPosts = createAsyncThunk(
  'posts/fetchPosts',

  async () => {
    const response = await fetch('/posts');
    return response.json();
  }
);
```

그리고 `extraReducers`에서 요청의 생명주기를 처리합니다.

```js
extraReducers: (builder) => {
  builder
    .addCase(fetchPosts.pending, (state) => {
      state.loading = true;
    })

    .addCase(fetchPosts.fulfilled, (state, action) => {
      state.loading = false;
      state.posts = action.payload;
    })

    .addCase(fetchPosts.rejected, (state, action) => {
      state.loading = false;
      state.error = action.error;
    });
}
```

여기까지는 가능하지만 실제 애플리케이션에서는 추가 문제가 생깁니다.

```text
컴포넌트 A → GET /posts
컴포넌트 B → GET /posts
```

둘이 같은 데이터를 요청하면?

```text
중복 요청을 할 것인가?
기존 데이터를 재사용할 것인가?
```

게시글을 수정했다면?

```text
PUT /posts/3
```

기존 `/posts` 데이터는 이제 오래된 데이터가 될 수 있습니다.

```text
GET /posts cache
       │
       │ POST 수정
       ▼
stale data
```

따라서 다시 가져와야 할 수도 있습니다.

이 모든 작업을 직접 구현하면 코드가 빠르게 복잡해집니다.

RTK Query는 이러한 **서버 데이터 관리 패턴 자체를 추상화한 도구**입니다.

---

# 3. RTK Query의 전체 구조

RTK Query를 사용할 때 가장 중요한 함수는 `createApi()`입니다.

```js
const api = createApi({
  reducerPath: 'api',

  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
  }),

  endpoints: (builder) => ({
    ...
  }),
});
```

전체 구조를 먼저 보면 다음과 같습니다.

```text
createApi()
│
├─ baseQuery
│
│    HTTP 요청 실행 방법
│
├─ endpoints
│
│    ├─ query
│
│    └─ mutation
│
├─ reducer
│
│    서버 데이터 Cache 관리
│
├─ middleware
│
│    요청 / Subscription / Cache Lifecycle 관리
│
├─ selectors
│
├─ thunks
│
└─ React Hooks
```

중요한 점은 `createApi()`가 단순히 Hook만 만드는 함수가 아니라는 것입니다.

`createApi()`는 하나의 **API Slice**를 생성합니다.

공식 문서에 따르면 생성된 API slice에는 캐시를 관리하는 reducer, subscription과 cache lifetime을 관리하는 middleware, endpoint별 selector와 thunk가 포함되며 React용 entry point를 사용하면 React Hooks도 자동 생성됩니다.

---

# 4. createApi()

기본 코드는 다음과 같습니다.

```js
import {
  createApi,
  fetchBaseQuery,
} from '@reduxjs/toolkit/query/react';

export const postApi = createApi({
  reducerPath: 'postApi',

  baseQuery: fetchBaseQuery({
    baseUrl: 'https://example.com/',
  }),

  tagTypes: ['Post'],

  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => 'posts',
    }),
  }),
});
```

여기에서:

```js
createApi({...})
```

를 실행하면 RTK Query가 API와 관련된 Redux 로직을 생성합니다.

개념적으로는 다음과 같습니다.

```text
postApi
│
├─ reducerPath
├─ reducer
├─ middleware
├─ endpoints
│
│   └─ getPosts
│
├─ util
│
└─ React Hooks
    └─ useGetPostsQuery
```

따라서 나중에 Store에 다음 두 가지를 등록합니다.

```js
postApi.reducer
postApi.middleware
```

---

# 5. reducerPath

```js
reducerPath: 'postApi'
```

는 RTK Query의 상태가 Redux Store에서 저장될 위치의 키입니다.

Store를 다음처럼 구성하면:

```js
configureStore({
  reducer: {
    [postApi.reducerPath]: postApi.reducer,
  },
});
```

개념적인 Redux State는 다음과 같은 구조를 가지게 됩니다.

```text
Redux Store
│
└─ state
   │
   └─ postApi
       │
       ├─ queries
       ├─ mutations
       ├─ provided
       ├─ subscriptions
       └─ config
```

일반적인 slice와 비교하면:

```js
reducer: {
  counter: counterReducer,
}
```

가

```text
state.counter
```

를 만든다면,

```js
reducer: {
  [postApi.reducerPath]: postApi.reducer,
}
```

는 일반적으로

```text
state.postApi
```

아래에 RTK Query의 상태를 저장하게 됩니다.

우리가 `posts`, `loading`, `error`를 직접 관리하지 않아도 되는 이유가 여기에 있습니다.

RTK Query reducer가 이 정보를 관리합니다.

---

# 6. fetchBaseQuery()

```js
baseQuery: fetchBaseQuery({
  baseUrl: 'https://example.com/',
})
```

`fetchBaseQuery()`는 브라우저의 `fetch()`를 기반으로 만들어진 RTK Query용 경량 HTTP 요청 함수입니다.

예를 들어:

```js
getPosts: builder.query({
  query: () => 'posts',
});
```

라면 최종 요청 URL은 대략 다음과 같이 만들어집니다.

```text
baseUrl
https://example.com/

        +

query()
posts

        ↓

https://example.com/posts
```

인증 토큰도 공통으로 설정할 수 있습니다.

```js
baseQuery: fetchBaseQuery({
  baseUrl: '/api',

  prepareHeaders: (headers, { getState }) => {
    const token = getState().auth.token;

    if (token) {
      headers.set(
        'authorization',
        `Bearer ${token}`
      );
    }

    return headers;
  },
});
```

그러면 각 endpoint마다 Authorization Header를 반복해서 작성하지 않아도 됩니다.

---

# 7. endpoints란?

```js
endpoints: (builder) => ({
  ...
})
```

는 서버와 어떤 작업을 할 것인지 정의하는 부분입니다.

여기서 `builder`는 우리가 만드는 객체가 아닙니다.

```js
(builder) => ({
```

이 callback을 RTK Query가 호출하면서 **builder 객체를 아규먼트로 전달합니다.**

그 객체에는 대표적으로 다음 메서드가 있습니다.

```js
builder.query()
builder.mutation()
```

따라서:

```text
builder
│
├─ query()
│
│   서버 데이터 조회
│
└─ mutation()
    서버 데이터 변경
```

이라는 구조입니다.

---

# 8. Query

Query는 일반적으로 **서버 데이터를 조회**할 때 사용합니다.

```text
GET /posts
GET /posts/3
GET /users
GET /products?page=2
```

예:

```js
getPosts: builder.query({
  query: () => 'posts',
});
```

그리고:

```js
getPostById: builder.query({
  query: (id) => `posts/${id}`,
});
```

React integration을 사용하면 endpoint 이름을 기반으로 Hook이 자동 생성됩니다.

```text
getPosts
    ↓
useGetPostsQuery

getPostById
    ↓
useGetPostByIdQuery
```

따라서:

```js
const {
  data,
  error,
  isLoading,
} = useGetPostsQuery();
```

처럼 사용할 수 있습니다.

---

# 9. Mutation

Mutation은 일반적으로 서버 데이터를 변경하는 작업입니다.

```text
POST
PUT
PATCH
DELETE
```

예:

```js
addPost: builder.mutation({
  query: (newPost) => ({
    url: 'posts',
    method: 'POST',
    body: newPost,
  }),
});
```

자동 생성되는 Hook은:

```js
useAddPostMutation()
```

입니다.

Query Hook과 Mutation Hook의 사용 방법은 다릅니다.

Query는 Hook을 호출하면 보통 자동으로 요청합니다.

```js
const { data } = useGetPostsQuery();
```

Mutation은 실행 함수를 반환합니다.

```js
const [addPost, result] =
  useAddPostMutation();
```

따라서 실제 요청은:

```js
addPost(newPost);
```

를 호출해야 발생합니다.

---

# 10. Query Hook의 핵심

```js
const {
  data,
  error,
  isLoading,
  isFetching,
  isSuccess,
  isError,
  refetch,
} = useGetPostsQuery();
```

여기서 특히 많이 혼동하는 것이:

```text
isLoading
vs
isFetching
```

입니다.

`isLoading`은 보통 현재 query에 대한 데이터가 아직 없는 최초 요청 상황을 표현하는 데 적합합니다.

반면 `isFetching`은 현재 네트워크 요청이 진행 중인지 나타냅니다.

따라서 이미 데이터가 화면에 있어도 새로운 요청이 진행 중이라면:

```text
data 있음
+
isFetching = true
```

가 가능합니다.

예를 들면:

```text
첫 요청

data 없음
isLoading = true
isFetching = true
```

이후 데이터가 도착합니다.

```text
data 있음
isLoading = false
isFetching = false
```

그리고 background refetch가 발생하면:

```text
기존 data 있음
isLoading = false
isFetching = true
```

가 될 수 있습니다.

이 차이는 로딩 UI를 설계할 때 매우 중요합니다.

---

# 11. RTK Query의 진짜 핵심: Cache

다음 두 컴포넌트를 생각해 봅시다.

```jsx
function Header() {
  const { data } = useGetPostByIdQuery(1);
}

function Main() {
  const { data } = useGetPostByIdQuery(1);
}
```

두 컴포넌트 모두:

```js
useGetPostByIdQuery(1)
```

을 호출합니다.

하지만 이것이 반드시 서버 요청 두 번을 의미하지는 않습니다.

RTK Query는 기본적으로 다음 조합을 기반으로 캐시를 구분합니다.

```text
endpoint
+
serialized query argument
```

예:

```text
getPostById + 1
```

이 조합에서 내부적인 `queryCacheKey`가 생성됩니다.

개념적으로:

```text
getPostById(1)
        │
        ▼
queryCacheKey
        │
        ▼
Cache Entry
```

따라서 Header와 Main이 같은 endpoint와 같은 아규먼트를 사용하면 같은 cache entry를 공유할 수 있습니다.

```text
Header
  │
  └──────┐
         ▼
 getPostById(1)
         │
      Cache
         │
 getPostById(1)
         ▲
  ┌──────┘
Main
```

이것이 RTK Query의 **Deduplication**입니다.

---

# 12. 아규먼트가 다르면 다른 Cache

다음 요청들은 서로 다릅니다.

```js
useGetPostByIdQuery(1);

useGetPostByIdQuery(2);

useGetPostByIdQuery(3);
```

개념적으로 각각 다른 cache entry가 만들어집니다.

```text
getPostById(1)
     ↓
Cache A

getPostById(2)
     ↓
Cache B

getPostById(3)
     ↓
Cache C
```

따라서 RTK Query의 캐시는 단순히:

```text
endpoint마다 하나
```

가 아닙니다.

더 정확히는:

```text
endpoint + serialized arguments
```

조합을 기준으로 생성됩니다.

---

# 13. Subscription

RTK Query에서 매우 중요한 개념이 **Subscription**입니다.

컴포넌트가:

```js
useGetPostsQuery();
```

를 호출하면 단순히 데이터를 요청하는 것만이 아닙니다.

개념적으로 해당 cache entry에 대해:

```text
"이 컴포넌트가 이 데이터를 사용하고 있습니다."
```

라는 subscription이 만들어집니다.

예를 들어:

```text
Component A
useGetPostsQuery()
        │
        ▼

getPosts cache
subscription count = 1
```

Component B도 같은 Query를 사용하면:

```text
Component A ─┐
             │
             ▼
        getPosts cache
             ▲
             │
Component B ─┘

subscription count = 2
```

Component A가 unmount되면:

```text
subscription count
2 → 1
```

Component B까지 unmount되면:

```text
subscription count
1 → 0
```

이 reference count가 cache lifetime 관리에서 중요합니다.

---

# 14. Cache는 컴포넌트가 사라지면 즉시 삭제되는가?

아닙니다.

기본적으로 마지막 subscription이 제거되어 reference count가 0이 된 뒤에도 cache data는 잠시 유지됩니다.

RTK Query의 기본 `keepUnusedDataFor` 값은 **60초**입니다.

```text
Component mounted
       │
       ▼
subscription = 1
       │
       │ unmount
       ▼
subscription = 0
       │
       │
       │ 60 seconds
       ▼
Cache 제거
```

그 시간 안에 같은 Query를 사용하는 컴포넌트가 다시 나타나면 기존 cache data를 사용할 수 있습니다.

설정도 가능합니다.

```js
createApi({
  keepUnusedDataFor: 120,
});
```

또는 endpoint별로 설정할 수 있습니다.

```js
getPosts: builder.query({
  query: () => 'posts',
  keepUnusedDataFor: 300,
});
```

---

# 15. useGetPostsQuery() 내부 흐름

이제 가장 중요한 전체 흐름입니다.

컴포넌트에서:

```js
const { data } =
  useGetPostsQuery();
```

를 실행했다고 하겠습니다.

전체 흐름을 개념적으로 보면:

```text
React Component
        │
        │ useGetPostsQuery()
        ▼
RTK Query Hook
        │
        ▼
getPosts endpoint
        │
        ▼
queryCacheKey 생성
        │
        ▼
Redux Store Cache 확인
        │
        ├──────────────┐
        │              │
   Cache 없음       Cache 있음
        │              │
        ▼              ▼
 HTTP Request       기존 Cache 사용
        │
        ▼
 fetchBaseQuery
        │
        ▼
      Server
        │
        ▼
    Response
        │
        ▼
RTK Query reducer
        │
        ▼
 Redux Store Cache
        │
        ▼
React Component
```

이 구조가 RTK Query의 핵심입니다.

---

# 16. Redux Store에는 무엇이 저장되는가?

기존 Redux 방식에서는 우리가 직접:

```js
{
  posts: [],
  loading: false,
  error: null
}
```

를 정의했습니다.

RTK Query에서는 이러한 서버 요청 상태를 RTK Query가 관리합니다.

개념적으로:

```text
state.postApi
│
├─ queries
│   │
│   └─ getPosts(...)
│       ├─ status
│       ├─ data
│       ├─ error
│       └─ timestamps
│
├─ mutations
│
├─ provided
│
├─ subscriptions
│
└─ config
```

즉:

```js
const { data, isLoading, error }
```

값들이 마법처럼 생기는 것이 아닙니다.

RTK Query가 Redux Store에 저장하고 관리하는 요청 상태를 Hook이 읽어서 React 컴포넌트에 제공하는 것입니다.

---

# 17. RTK Query reducer의 역할

Store에는 반드시:

```js
[postApi.reducerPath]: postApi.reducer
```

가 필요합니다.

```js
export const store = configureStore({
  reducer: {
    [postApi.reducerPath]: postApi.reducer,
  },
});
```

이 reducer는 RTK Query가 관리하는:

```text
Query 결과
Mutation 상태
Cache 상태
Tag 관련 상태
요청 상태
```

등을 Redux Store에 반영합니다.

즉:

```text
postApi.reducer
=
RTK Query 상태를 Redux Store에 저장하는 reducer
```

라고 이해하면 됩니다.

---

# 18. middleware는 왜 필요한가?

RTK Query에서 reducer만 등록해서는 충분하지 않습니다.

```js
middleware: (getDefaultMiddleware) =>
  getDefaultMiddleware()
    .concat(postApi.middleware)
```

가 필요합니다.

RTK Query middleware는 단순 HTTP middleware가 아닙니다.

RTK Query의 동작 과정에서 필요한 **비동기 요청과 cache lifecycle, subscription 기반 동작, invalidation/refetch 등의 orchestration**에 관여합니다.

큰 구조로 보면:

```text
Component
    │
    ▼
RTK Query Hook
    │
    ▼
dispatch
    │
    ▼
Redux Middleware Chain
    │
    ├─ Redux Toolkit default middleware
    │
    └─ postApi.middleware
              │
              ├─ Query lifecycle
              ├─ Subscription
              ├─ Cache lifecycle
              ├─ Invalidation
              └─ Re-fetch 관련 처리
```

따라서 Store에는 두 가지가 모두 필요합니다.

```text
postApi.reducer
        +
postApi.middleware
```

---

# 19. 전체 Store 설정

```js
import { configureStore } from '@reduxjs/toolkit';
import { postApi } from './postApi';

export const store = configureStore({
  reducer: {
    [postApi.reducerPath]: postApi.reducer,
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(postApi.middleware),
});
```

그리고 React Redux의 Provider는 그대로 사용합니다.

```jsx
import { Provider } from 'react-redux';
import { store } from './store';

<Provider store={store}>
  <App />
</Provider>
```

전체 관계는:

```text
<Provider store={store}>
        │
        ▼
    Redux Store
        │
        ├─ ordinary reducers
        │
        └─ RTK Query reducer
                 ▲
                 │
         RTK Query middleware
                 ▲
                 │
       generated query hooks
                 ▲
                 │
          React Components
```

입니다.

---

# 20. Tags를 정확하게 이해하기

다음 코드를 보겠습니다.

```js
providesTags: ['Post']
```

이를:

```text
이 Query는 Post Tag를 구독한다
```

라고 설명하면 정확하지 않습니다.

**Subscription과 Tag는 다른 개념입니다.**

Subscription은:

```text
어떤 컴포넌트가 어떤 cache entry를 사용하고 있는가?
```

를 관리하는 개념입니다.

Tag는:

```text
어떤 cache data가 어떤 종류의 데이터인가?
```

를 표시하기 위한 **cache invalidation label**입니다.

따라서:

```js
providesTags: ['Post']
```

의 의미는:

```text
"이 Query가 만든 cache data는
Post라는 Tag를 제공한다."
```

에 가깝습니다.

---

# 21. Cache Invalidation

게시글 목록을 가져왔다고 하겠습니다.

```js
getPosts: builder.query({
  query: () => 'posts',
  providesTags: ['Post'],
});
```

캐시는 개념적으로:

```text
getPosts Cache
      │
      └─ Tag: Post
```

가 됩니다.

이후 게시글을 추가합니다.

```js
addPost: builder.mutation({
  query: (post) => ({
    url: 'posts',
    method: 'POST',
    body: post,
  }),

  invalidatesTags: ['Post'],
});
```

Mutation 성공 후:

```text
addPost()
   │
   ▼
POST /posts
   │
   ▼
Success
   │
   ▼
invalidate "Post"
   │
   ▼
Post Tag를 제공한 cache 탐색
   │
   ▼
getPosts cache invalid
```

현재 해당 Query를 사용하는 active subscription이 있다면 관련 Query가 다시 fetch될 수 있습니다.

```text
Mutation 성공
      │
      ▼
Post Tag invalidation
      │
      ▼
getPosts cache stale
      │
      ▼
active subscription 존재
      │
      ▼
Automatic Re-fetch
      │
      ▼
GET /posts
```

이것이 RTK Query의 가장 강력한 기능 중 하나입니다.

---

# 22. Tag를 ID 단위로 관리하기

모든 Post를 무효화하는 방식은 편하지만 범위가 큽니다.

```js
invalidatesTags: ['Post']
```

게시글 하나만 수정했는데 모든 Post 관련 Query를 다시 요청할 수도 있기 때문입니다.

더 세밀하게 관리할 수 있습니다.

```js
getPostById: builder.query({
  query: (id) => `posts/${id}`,

  providesTags: (result, error, id) => [
    { type: 'Post', id },
  ],
});
```

그러면:

```text
getPostById(1)
→ { type: 'Post', id: 1 }

getPostById(2)
→ { type: 'Post', id: 2 }
```

처럼 구분할 수 있습니다.

수정 Mutation도:

```js
updatePost: builder.mutation({
  query: ({ id, ...patch }) => ({
    url: `posts/${id}`,
    method: 'PATCH',
    body: patch,
  }),

  invalidatesTags: (result, error, arg) => [
    { type: 'Post', id: arg.id },
  ],
});
```

로 만들 수 있습니다.

그러면:

```text
Post 3 수정
       │
       ▼
{ type: "Post", id: 3 }
invalidate
       │
       ▼
Post 3과 관련된 cache
```

를 정밀하게 무효화할 수 있습니다.

---

# 23. LIST Tag 패턴

실무에서는 목록과 개별 데이터를 구분하기 위해 `'LIST'` 같은 추상 ID를 많이 활용할 수 있습니다.

```js
getPosts: builder.query({
  query: () => 'posts',

  providesTags: (result) =>
    result
      ? [
          ...result.map(({ id }) => ({
            type: 'Post',
            id,
          })),

          {
            type: 'Post',
            id: 'LIST',
          },
        ]
      : [
          {
            type: 'Post',
            id: 'LIST',
          },
        ],
});
```

예를 들어 다음 데이터가 있다면:

```text
Posts List Cache

Post / 1
Post / 2
Post / 3
Post / LIST
```

새 게시글 추가 시:

```js
invalidatesTags: [
  { type: 'Post', id: 'LIST' },
]
```

만 무효화할 수 있습니다.

반면 Post 3 수정은:

```js
invalidatesTags: [
  { type: 'Post', id: 3 },
]
```

처럼 처리할 수 있습니다.

이렇게 하면 불필요한 refetch를 줄일 수 있습니다.

---

# 24. Query와 Mutation의 전체 관계

RTK Query의 구조를 한 장으로 생각하면 다음과 같습니다.

```text
                    ┌─────────────────┐
                    │     Server      │
                    └────────┬────────┘
                             │
                 HTTP Request│Response
                             │
                    ┌────────▼────────┐
                    │ fetchBaseQuery  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │     createApi   │
                    │                 │
                    │   endpoints     │
                    │                 │
                    │ query mutation  │
                    └───────┬─────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
          Query Cache                  Mutation
              │                           │
              │                    invalidatesTags
              │                           │
              ◀──── providesTags ─────────┘
              │
              ▼
          Redux Store
              │
              ▼
        generated Hook
              │
              ▼
      React Component
```

---

# 25. 자동 생성 Hook

다음 endpoint가 있다면:

```js
endpoints: (builder) => ({
  getPosts: builder.query({...}),
  getPostById: builder.query({...}),
  addPost: builder.mutation({...}),
})
```

React용 `createApi`는 endpoint 이름을 기반으로 Hook을 생성합니다.

```text
getPosts
→ useGetPostsQuery

getPostById
→ useGetPostByIdQuery

addPost
→ useAddPostMutation
```

따라서:

```js
export const {
  useGetPostsQuery,
  useGetPostByIdQuery,
  useAddPostMutation,
} = postApi;
```

처럼 사용할 수 있습니다.

---

# 26. 완성된 API Slice

```js
import {
  createApi,
  fetchBaseQuery,
} from '@reduxjs/toolkit/query/react';

export const postApi = createApi({
  reducerPath: 'postApi',

  baseQuery: fetchBaseQuery({
    baseUrl: 'https://example.com/api/',
  }),

  tagTypes: ['Post'],

  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => 'posts',

      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({
                type: 'Post',
                id,
              })),
              {
                type: 'Post',
                id: 'LIST',
              },
            ]
          : [
              {
                type: 'Post',
                id: 'LIST',
              },
            ],
    }),

    getPostById: builder.query({
      query: (id) => `posts/${id}`,

      providesTags: (result, error, id) => [
        {
          type: 'Post',
          id,
        },
      ],
    }),

    addPost: builder.mutation({
      query: (newPost) => ({
        url: 'posts',
        method: 'POST',
        body: newPost,
      }),

      invalidatesTags: [
        {
          type: 'Post',
          id: 'LIST',
        },
      ],
    }),

    updatePost: builder.mutation({
      query: ({ id, ...patch }) => ({
        url: `posts/${id}`,
        method: 'PATCH',
        body: patch,
      }),

      invalidatesTags: (
        result,
        error,
        { id }
      ) => [
        {
          type: 'Post',
          id,
        },
      ],
    }),
  }),
});

export const {
  useGetPostsQuery,
  useGetPostByIdQuery,
  useAddPostMutation,
  useUpdatePostMutation,
} = postApi;
```

---

# 27. Mutation과 unwrap()

Mutation Hook은 다음처럼 사용합니다.

```js
const [addPost, {
  isLoading,
  error,
}] = useAddPostMutation();
```

그리고:

```js
addPost(newPost);
```

를 호출합니다.

Mutation trigger의 반환값은 `.unwrap()`을 사용할 수 있습니다.

```js
try {
  const result = await addPost({
    title: 'Redux',
    body: 'RTK Query',
  }).unwrap();

  console.log(result);
} catch (error) {
  console.error(error);
}
```

`.unwrap()`을 사용하면 성공 payload를 직접 얻거나 실패를 `catch`에서 처리하기 편합니다.

---

# 28. refetch()

Query Hook은 `refetch`도 제공합니다.

```js
const {
  data,
  refetch,
} = useGetPostsQuery();
```

직접 다시 요청하고 싶다면:

```jsx
<button onClick={refetch}>
  다시 가져오기
</button>
```

처럼 사용할 수 있습니다.

하지만 RTK Query에서는 가능한 경우 매번 수동 `refetch()`를 호출하기보다 **Tag Invalidation을 통해 데이터 관계를 선언하는 방식**이 더 자연스러운 경우가 많습니다.

---

# 29. Polling

주기적으로 서버 데이터를 갱신할 수도 있습니다.

```js
useGetPostsQuery(undefined, {
  pollingInterval: 3000,
});
```

개념적으로:

```text
GET /posts
    │
  3 sec
    │
GET /posts
    │
  3 sec
    │
GET /posts
```

형태가 됩니다.

실시간 대시보드나 지속적인 상태 확인 같은 경우에 활용할 수 있습니다.

---

# 30. refetchOnFocus와 refetchOnReconnect

사용자가 다른 탭으로 이동했다가 다시 돌아왔을 때 데이터를 새로 가져오거나, 네트워크 연결이 복구되었을 때 다시 요청하도록 구성할 수도 있습니다.

Store 설정에서 보통:

```js
import { setupListeners } from '@reduxjs/toolkit/query';

setupListeners(store.dispatch);
```

를 추가하고 Query 옵션을 사용할 수 있습니다.

```js
useGetPostsQuery(undefined, {
  refetchOnFocus: true,
  refetchOnReconnect: true,
});
```

이 기능은 오래된 cache를 자연스럽게 최신 상태로 유지하는 데 유용합니다.

---

# 31. skip

아직 Query를 실행하면 안 되는 상황도 있습니다.

예를 들어 로그인한 사용자의 ID가 아직 없다면:

```js
const { data } =
  useGetUserQuery(userId, {
    skip: !userId,
  });
```

`userId`가 없는 동안 Query를 실행하지 않습니다.

---

# 32. transformResponse

서버의 응답 구조가 다음과 같다고 하겠습니다.

```json
{
  "success": true,
  "result": {
    "posts": []
  }
}
```

컴포넌트가 필요한 것은 `posts`뿐일 수 있습니다.

```js
getPosts: builder.query({
  query: () => 'posts',

  transformResponse: (response) =>
    response.result.posts,
});
```

그러면 cache에는 컴포넌트가 사용하기 편하게 변환된 데이터를 저장할 수 있습니다.

---

# 33. Optimistic Update

일반적인 Mutation은:

```text
사용자 클릭
   ↓
Server Request
   ↓
Server Response
   ↓
UI Update
```

순서입니다.

따라서 서버가 느리면 UI 반응도 느껴질 수 있습니다.

Optimistic Update는 순서를 바꿉니다.

```text
사용자 클릭
   ↓
UI 즉시 변경
   ↓
Server Request
   │
   ├─ 성공 → 유지
   │
   └─ 실패 → rollback
```

RTK Query에서는 `onQueryStarted`와 cache update utility 등을 이용하여 구현할 수 있습니다.

개념적으로:

```js
async onQueryStarted(
  arg,
  { dispatch, queryFulfilled }
) {
  const patchResult = dispatch(
    postApi.util.updateQueryData(
      'getPosts',
      undefined,
      (draft) => {
        // cache를 먼저 변경
      }
    )
  );

  try {
    await queryFulfilled;
  } catch {
    patchResult.undo();
  }
}
```

처럼 처리할 수 있습니다.

---

# 34. Prefetch

사용자가 실제 페이지로 이동하기 전에 데이터를 미리 요청할 수도 있습니다.

예를 들어:

```text
사용자가 게시글 링크에 mouse hover
           │
           ▼
        Prefetch
           │
           ▼
      GET /posts/3
           │
           ▼
        Cache 저장
           │
           ▼
사용자가 게시글 페이지 이동
           │
           ▼
      Cache 즉시 사용
```

이렇게 하면 페이지 전환 체감 속도를 개선할 수 있습니다.

---

# 35. RTK Query의 전체 실행 흐름

마지막으로 전체 흐름을 정리하면 다음과 같습니다.

```text
① React Component render

        ↓

② useGetPostsQuery()

        ↓

③ RTK Query endpoint 호출

        ↓

④ endpoint + argument
   → queryCacheKey 생성

        ↓

⑤ Redux Store Cache 확인

        ↓

   ┌───────────────┐
   │ Cache 존재?   │
   └───────┬───────┘
           │
      ┌────┴────┐
      │         │
     YES       NO
      │         │
      ▼         ▼
Cache 사용   Query 시작
                │
                ▼
        RTK Query middleware
                │
                ▼
          fetchBaseQuery
                │
                ▼
             Server
                │
                ▼
            Response
                │
                ▼
         RTK Query reducer
                │
                ▼
          Redux Store Cache
                │
                ▼
        React Component
          re-render
```

여기에 Subscription까지 포함하면:

```text
Component mount
       │
       ▼
useQuery()
       │
       ▼
Cache Entry 구독
       │
       ▼
subscription +1

...

Component unmount
       │
       ▼
subscription -1
       │
       ▼
subscription = 0
       │
       ▼
keepUnusedDataFor timer
       │
       ▼
Cache 제거
```

가 됩니다.

---

# 36. Mutation까지 포함한 최종 데이터 흐름

```text
                  SERVER
                    ▲
                    │
              HTTP Request
                    │
             fetchBaseQuery
                    ▲
                    │
             RTK Query API
          ┌─────────┴─────────┐
          │                   │
        Query              Mutation
          │                   │
          │              invalidatesTags
          │                   │
          ▼                   │
       Cache ◀────────────────┘
          │
     providesTags
          │
          ▼
      Redux Store
          │
          ▼
Generated React Hook
          │
          ▼
React Component
```

이 흐름을 이해하는 것이 RTK Query 학습의 핵심입니다.

---

# 37. createAsyncThunk와 RTK Query의 차이

`createAsyncThunk()`와 RTK Query는 둘 다 비동기 요청에 사용할 수 있지만 목적이 다릅니다.

| 영역                 | createAsyncThunk | RTK Query   |
| ------------------ | ---------------- | ----------- |
| 비동기 작업             | 직접 정의            | Endpoint 선언 |
| pending            | reducer에서 처리     | 자동 관리       |
| fulfilled          | reducer에서 처리     | 자동 관리       |
| rejected           | reducer에서 처리     | 자동 관리       |
| 서버 데이터 Cache       | 직접 구현            | 자동 관리       |
| 중복 Query 제거        | 직접 구현            | 지원          |
| Subscription       | 직접 구현 필요         | 자동 관리       |
| Cache Lifetime     | 직접 구현            | 지원          |
| 자동 Refetch         | 직접 구현            | 지원          |
| Cache Invalidation | 직접 구현            | Tags 제공     |
| React Hook         | 직접 연결            | 자동 생성 가능    |
| Optimistic Update  | 직접 구현            | 관련 API 제공   |
| Polling            | 직접 구현            | 지원          |
| Prefetch           | 직접 구현            | 지원          |

따라서 단순히:

```text
RTK Query
=
createAsyncThunk의 짧은 버전
```

이라고 이해해서는 안 됩니다.

더 정확하게는:

```text
createAsyncThunk
→ 범용적인 비동기 Redux 로직

RTK Query
→ 서버 데이터 fetching + caching 문제에 특화된 시스템
```

입니다.

실제로 RTK Query 자체도 내부적으로 Redux Toolkit의 `createSlice`와 `createAsyncThunk` 기반 기능을 활용해 구성되어 있습니다.

---

# 38. RTK Query와 TanStack Query

둘 모두 서버 상태를 관리하기 위한 강력한 도구이지만 생태계와 설계 방향이 다릅니다.

| 항목               | RTK Query             | TanStack Query            |
| ---------------- | --------------------- | ------------------------- |
| 생태계              | Redux Toolkit         | 독립적                       |
| Redux 필요성        | Redux Store 사용        | Redux 불필요                 |
| Cache 저장         | Redux 기반              | 자체 Query Cache            |
| API 정의           | endpoint 중심           | query hook/options 중심     |
| DevTools         | Redux DevTools와 통합 가능 | 전용 DevTools               |
| Tag Invalidation | 지원                    | Query Key invalidation 방식 |
| Generated Hooks  | 지원                    | 일반적으로 직접 hooks/options 구성 |
| Redux 앱과 통합      | 매우 자연스러움              | 별도 서버 상태 계층               |

따라서:

```text
이미 Redux Toolkit 중심 애플리케이션
          │
          ▼
RTK Query가 자연스러운 선택
```

이 될 수 있습니다.

반대로:

```text
Redux를 사용하지 않는 애플리케이션
          │
          ▼
TanStack Query가 자연스러운 선택
```

인 경우가 많습니다.

다만 어느 한쪽이 항상 우월하다는 의미는 아닙니다.

애플리케이션 아키텍처와 팀의 상태 관리 전략에 따라 선택하는 것이 적절합니다.

---

# 39. 가장 중요한 개념 관계

RTK Query를 공부할 때 다음 개념을 서로 혼동하지 않는 것이 중요합니다.

```text
Query
│
│ 서버 데이터 조회
▼
Cache Entry
│
│ endpoint + argument로 식별
▼
queryCacheKey


Subscription
│
│ 누가 이 cache를 사용 중인가?
▼
reference count


Tag
│
│ 어떤 cache를 무효화할 것인가?
▼
Cache Invalidation


Mutation
│
│ 서버 데이터 변경
▼
invalidatesTags
│
▼
관련 Cache invalid
│
▼
Active Subscription 존재
│
▼
Automatic Re-fetch
```

즉 세 개는 서로 다른 개념입니다.

```text
Cache Key
= 캐시의 "주소"

Subscription
= 캐시를 사용하는 "사용자 수"

Tag
= 캐시 무효화를 위한 "라벨"
```

이 세 가지를 구분하면 RTK Query의 구조가 훨씬 명확해집니다.

---

# 40. RTK Query 한 문장 정의

RTK Query는 단순한 HTTP 요청 라이브러리가 아닙니다.

> **RTK Query는 API endpoint를 선언하면 서버 요청부터 Redux Store 기반의 캐싱, 요청 상태, subscription, cache lifetime, invalidation, refetch까지 서버 상태의 전체 생명주기를 관리해 주는 Redux Toolkit의 데이터 패칭 및 캐싱 시스템입니다.**

그리고 가장 핵심적인 흐름은 다음 하나로 정리할 수 있습니다.

```text
React Component
      │
      ▼
Generated Hook
      │
      ▼
RTK Query Endpoint
      │
      ▼
queryCacheKey
      │
      ▼
Redux Cache
      │
      ├── Cache HIT ─────────────┐
      │                          │
      └── Cache MISS             │
              │                  │
              ▼                  │
           Request               │
              │                  │
              ▼                  │
            Server               │
              │                  │
              ▼                  │
           Response              │
              │                  │
              ▼                  │
          Redux Cache ◀──────────┘
              │
              ▼
        React Component
```

여기에 Mutation이 발생하면:

```text
Mutation
   │
   ▼
Server 변경
   │
   ▼
invalidatesTags
   │
   ▼
관련 Query Cache invalid
   │
   ▼
Active Subscription
   │
   ▼
Automatic Re-fetch
```

가 추가됩니다.

이 구조가 RTK Query의 본질입니다.
