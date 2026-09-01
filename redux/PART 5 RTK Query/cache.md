RTK Query의 **cache**는 단순히 “서버에서 받아온 데이터를 잠깐 저장하는 공간” 정도가 아닙니다. 정확히는 **서버에서 가져온 데이터와 그 요청 상태를 Redux Store 안에 저장하고, 같은 요청을 여러 컴포넌트가 공유하게 하며, 언제 다시 요청할지·언제 삭제할지·언제 무효화할지까지 관리하는 시스템**입니다. ([Redux Toolkit][1])

핵심부터 보면 이렇습니다.

```text
Server
   ↓ HTTP
RTK Query
   ↓
Redux Store 안의 RTK Query Cache
   ↓
React Hook
   ↓
Component
```

예를 들어:

```js
const { data } = useGetProductsQuery();
```

를 호출했을 때 RTK Query는 단순히 `fetch("/products")`만 하는 것이 아니라,

```text
1. 이 요청에 해당하는 Cache가 이미 있는가?
2. 있으면 새 HTTP 요청이 필요한가?
3. 현재 이 Cache를 몇 개의 컴포넌트가 사용 중인가?
4. 요청 상태는 pending / fulfilled / rejected 중 무엇인가?
5. 응답 데이터는 무엇인가?
6. 언제 마지막으로 성공했는가?
7. 언제 Cache를 삭제할 것인가?
```

까지 관리합니다.

---

## 1. RTK Query Cache는 어디에 저장되는가?

중요합니다.

RTK Query의 cache는 브라우저의:

```text
localStorage
sessionStorage
IndexedDB
```

같은 곳에 기본적으로 저장되는 것이 아닙니다.

**Redux Store 안에 저장됩니다.**

예를 들어:

```js
export const productApi = createApi({
  reducerPath: "productApi",

  baseQuery: fetchBaseQuery({
    baseUrl: "/api"
  }),

  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => "/products"
    })
  })
});
```

그리고:

```js
configureStore({
  reducer: {
    [productApi.reducerPath]:
      productApi.reducer
  },

  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(
      productApi.middleware
    )
});
```

라고 하면 Store의 구조를 개념적으로 이렇게 볼 수 있습니다.

```text
Redux Store
│
├─ auth
│    └─ ...
│
├─ cart
│    └─ ...
│
└─ productApi
     │
     ├─ queries
     ├─ mutations
     ├─ provided
     ├─ subscriptions 관련 상태
     └─ config
```

즉 RTK Query cache 역시 **Redux State의 일부**입니다.

다만 개발자가 직접 reducer를 작성해서 관리하는 것이 아니라:

```text
productApi.reducer
+
productApi.middleware
```

가 관리합니다.

---

# 2. 무엇을 Cache하는가?

보통 “cache”라고 하면 `data`만 생각하기 쉽습니다.

하지만 실제 query cache entry에는 단순 응답 데이터 외에 **요청 상태에 필요한 메타데이터**도 함께 관리됩니다.

예를 들어:

```js
useGetProductQuery(10);
```

을 실행했다고 하겠습니다.

서버 응답:

```js
{
  id: 10,
  name: "Keyboard",
  price: 50000
}
```

RTK Query 내부에서는 개념적으로:

```js
{
  status: "fulfilled",

  endpointName: "getProduct",

  originalArgs: 10,

  data: {
    id: 10,
    name: "Keyboard",
    price: 50000
  },

  requestId: "...",

  startedTimeStamp: ...,

  fulfilledTimeStamp: ...
}
```

같은 정보를 관리한다고 보면 됩니다.

그래서 Hook에서:

```js
const {
  data,
  error,
  isLoading,
  isFetching,
  isSuccess,
  isError,
} = useGetProductQuery(10);
```

같은 값을 바로 받을 수 있는 것입니다.

즉:

```text
RTK Query Cache
        │
        ├─ 실제 서버 데이터
        │
        └─ 요청 상태 정보
                ↓
            Query Hook
                ↓
data
isLoading
isFetching
isSuccess
error
...
```

입니다. ([Redux Toolkit][2])

---

# 3. Cache의 핵심은 `endpoint + argument`

이 부분이 RTK Query cache에서 가장 중요합니다.

다음 endpoint가 있다고 하겠습니다.

```js
getProduct: builder.query({
  query: (id) => `/products/${id}`
})
```

그리고:

```js
useGetProductQuery(10);
```

을 호출합니다.

그러면 RTK Query는 대략:

```text
endpoint
getProduct

+

argument
10
```

을 이용해서 cache key를 만듭니다.

개념적으로:

```text
getProduct(10)
```

이라는 Cache가 만들어진다고 이해하면 됩니다.

다른 컴포넌트에서:

```js
useGetProductQuery(20);
```

을 호출하면:

```text
getProduct(20)
```

이라는 별도의 Cache가 만들어집니다.

따라서:

```text
RTK Query Cache

getProduct(10)
├─ id: 10
├─ name: Keyboard
└─ price: 50000


getProduct(20)
├─ id: 20
├─ name: Mouse
└─ price: 30000
```

처럼 됩니다.

공식적으로 RTK Query는 **endpoint 이름과 직렬화된 query argument를 이용해서 `queryCacheKey`를 만듭니다.** 같은 endpoint와 같은 argument가 같은 cache key를 만들어내면 요청은 dedupe되고 같은 캐시 데이터를 공유합니다. ([Redux Toolkit][1])

---

# 4. `queryCacheKey`란?

예를 들어:

```js
useGetProductQuery(10);
```

RTK Query 내부에서는 argument를 직렬화하여 cache key를 만듭니다.

개념적으로:

```text
getProduct + 10
       ↓
queryCacheKey
       ↓
getProduct(10)
```

정확한 내부 문자열 표현을 애플리케이션 코드에서 의존할 필요는 없지만 개념은 중요합니다.

객체 argument도 마찬가지입니다.

```js
useSearchProductsQuery({
  category: "keyboard",
  page: 1
});
```

그러면:

```text
endpoint:
searchProducts

argument:
{
  category: "keyboard",
  page: 1
}
```

을 직렬화해서 cache key를 만듭니다.

RTK Query의 기본 직렬화는 객체 key 순서도 정리하기 때문에, 의미상 같은 객체는 같은 cache key를 만들도록 처리합니다. ([Redux Toolkit][3])

예를 들어 개념적으로:

```js
{
  page: 1,
  category: "keyboard"
}
```

와:

```js
{
  category: "keyboard",
  page: 1
}
```

은 같은 query argument로 취급될 수 있도록 기본 serialization이 처리합니다.

---

# 5. 같은 요청을 두 컴포넌트가 하면 어떻게 될까?

여기가 RTK Query cache의 강력한 부분입니다.

```jsx
function Header() {
  const { data } =
    useGetProductQuery(10);

  ...
}
```

그리고:

```jsx
function ProductPage() {
  const { data } =
    useGetProductQuery(10);

  ...
}
```

두 컴포넌트가 있습니다.

둘 다:

```text
getProduct(10)
```

을 요청합니다.

그러면:

```text
Header
   │
   └── useGetProductQuery(10)
                 │
                 ▼
           getProduct(10)
              Cache
                 ▲
                 │
ProductPage      │
   └── useGetProductQuery(10)
```

가 됩니다.

서버 요청을 두 번 보내는 것이 아니라 일반적으로 **하나의 cache entry를 공유**합니다.

```text
GET /products/10
       ↓
한 번 요청
       ↓
getProduct(10) Cache
       ↓
┌───────────────┐
│               │
Header       ProductPage
```

같은 `queryCacheKey`를 생성하는 요청은 dedupe되며 캐시 데이터와 업데이트를 공유합니다. ([Redux Toolkit][1])

---

# 6. 이것이 Request Deduplication이다

예를 들어 동시에:

```js
Component A
useGetProductQuery(10);

Component B
useGetProductQuery(10);

Component C
useGetProductQuery(10);
```

이 호출됐다고 하겠습니다.

세 요청 모두:

```text
endpoint = getProduct
arg = 10
```

이므로 같은:

```text
queryCacheKey
```

를 가집니다.

따라서:

```text
Component A ─┐
Component B ─┼─→ getProduct(10)
Component C ─┘
                    ↓
              Cache 존재 확인
                    ↓
              하나의 요청 공유
```

하게 됩니다.

이 기능을 **deduplication**이라고 합니다.

---

# 7. argument가 다르면 Cache도 다르다

반면:

```js
useGetProductQuery(10);
useGetProductQuery(20);
useGetProductQuery(30);
```

이면:

```text
getProduct(10)
getProduct(20)
getProduct(30)
```

이라는 서로 다른 cache key가 만들어집니다.

따라서 HTTP 요청도 각각 필요할 수 있습니다.

```text
GET /products/10
GET /products/20
GET /products/30
```

즉:

> **endpoint가 같다고 같은 Cache가 아닙니다.**

정확히는:

> **endpoint + serialized argument가 같아야 같은 Cache입니다.**

---

# 8. Cache가 있으면 서버 요청을 안 한다

예를 들어 첫 번째 컴포넌트가:

```js
useGetProductQuery(10);
```

을 호출합니다.

Cache 없음:

```text
getProduct(10)
     ↓
Cache 없음
     ↓
GET /products/10
     ↓
Server
     ↓
Response
     ↓
Cache 저장
```

이후 다른 컴포넌트가:

```js
useGetProductQuery(10);
```

을 호출합니다.

이미 Cache가 있다면 기본적인 상황에서는:

```text
getProduct(10)
     ↓
Cache 존재
     ↓
Cache 사용
     ↓
HTTP 요청 생략
```

하게 됩니다. ([Redux Toolkit][1])

이것이 RTK Query가 불필요한 네트워크 요청을 줄이는 핵심입니다.

---

# 9. Subscription이 매우 중요하다

RTK Query Cache를 이해하려면 **subscription**을 반드시 알아야 합니다.

컴포넌트가:

```js
useGetProductQuery(10);
```

을 호출하면 단순히 데이터를 한번 읽는 것이 아닙니다.

해당 cache entry를 **구독(subscribe)**합니다.

```text
Component
      ↓
useGetProductQuery(10)
      ↓
getProduct(10) Cache 구독
```

즉:

```text
“getProduct(10)의 데이터가 바뀌면
 나에게 알려줘.”
```

라는 의미입니다.

그래서 cache가 바뀌면:

```text
RTK Query Cache 변경
        ↓
Hook 감지
        ↓
Component 리렌더링
```

됩니다.

---

# 10. Subscription Reference Count

RTK Query는 같은 cache entry를 몇 군데에서 사용하고 있는지도 셉니다.

예를 들어:

```text
Component A
useGetProductQuery(10)

Component B
useGetProductQuery(10)

Component C
useGetProductQuery(20)
```

이면:

```text
getProduct(10)
subscription count = 2

getProduct(20)
subscription count = 1
```

입니다.

공식 문서에서도 subscription은 reference-count 방식으로 관리된다고 설명합니다. ([Redux Toolkit][1])

---

# 11. 컴포넌트가 unmount되면?

Component A가 사라지면:

```text
getProduct(10)

2 subscribers
      ↓
1 subscriber
```

가 됩니다.

하지만 Component B가 여전히 사용하고 있으므로 cache는 그대로 유지됩니다.

```text
Component B
      ↓
getProduct(10)
subscription = 1
      ↓
Cache 유지
```

이게 중요합니다.

**컴포넌트 하나가 unmount했다고 Cache가 바로 삭제되는 것이 아닙니다.**

---

# 12. 마지막 subscriber가 사라지면?

Component B까지 unmount하면:

```text
getProduct(10)
subscription count
1
↓
0
```

이 됩니다.

여기서도 cache가 즉시 삭제되지는 않습니다.

기본적으로 RTK Query는 **마지막 subscriber가 사라진 후 60초 동안 Cache를 유지**합니다. ([Redux Toolkit][1])

```text
subscription = 0
      ↓
60초 timer 시작
      ↓
Cache 유지
      ↓
60초 동안 새 subscriber 없음
      ↓
Cache 삭제
```

---

# 13. 왜 바로 삭제하지 않을까?

예를 들어:

```text
ProductList
     ↓
ProductDetail
     ↓
뒤로 가기
     ↓
ProductList
```

처럼 사용자가 화면을 이동한다고 해보겠습니다.

ProductList에서:

```js
useGetProductsQuery();
```

를 사용했습니다.

상세 페이지로 이동:

```text
ProductList unmount
     ↓
subscription = 0
```

이 됐다고 Cache를 즉시 삭제하면,

뒤로 왔을 때:

```text
GET /products
```

를 또 보내야 합니다.

하지만 Cache를 일정 시간 유지하면:

```text
ProductList unmount
       ↓
Cache 60초 유지
       ↓
30초 후 다시 ProductList mount
       ↓
기존 Cache 사용
       ↓
화면 즉시 표시
```

할 수 있습니다.

그래서 사용자 경험도 빨라지고 서버 요청도 줄어듭니다.

---

# 14. `keepUnusedDataFor`

이 Cache 유지 시간을 설정하는 것이:

```js
keepUnusedDataFor
```

입니다.

기본값은:

```text
60초
```

입니다. ([Redux Toolkit][1])

예를 들어:

```js
const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/api"
  }),

  keepUnusedDataFor: 30,

  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => "/products"
    })
  })
});
```

그러면:

```text
마지막 subscriber 제거
       ↓
30초
       ↓
Cache 삭제
```

가 됩니다.

---

# 15. endpoint별 설정도 가능하다

```js
getProducts: builder.query({
  query: () => "/products",

  keepUnusedDataFor: 120
})
```

이렇게 하면 이 endpoint의 Cache만:

```text
120초
```

동안 유지됩니다.

endpoint 설정이 API 전체 설정을 override할 수 있습니다. ([Redux Toolkit][3])

---

# 16. `keepUnusedDataFor`는 “데이터 유효기간”이 아니다

이 부분은 학생들이 많이 헷갈립니다.

```js
keepUnusedDataFor: 60
```

을 보고:

> “데이터는 60초 동안만 유효하다.”

라고 생각하면 틀립니다.

정확한 의미는:

> **마지막 subscriber가 사라진 후 얼마 동안 Cache entry를 보관할 것인가**

입니다.

즉 active subscriber가 계속 있다면:

```text
subscription > 0
```

Cache는 60초가 지났다고 자동 삭제되지 않습니다.

```text
Component 계속 mount
      ↓
subscription = 1
      ↓
10분 경과
      ↓
Cache 유지
```

입니다.

---

# 17. Cache와 Refetch는 다른 문제다

이것도 중요합니다.

Cache가 존재한다고 해서:

```text
절대로 다시 서버 요청하지 않는다
```

는 뜻은 아닙니다.

RTK Query에는 refetch를 결정하는 별도 옵션들이 있습니다.

예:

```text
refetchOnMountOrArgChange
refetchOnFocus
refetchOnReconnect
refetch()
tag invalidation
```

등입니다.

즉:

```text
Cache 존재 여부
```

와:

```text
새 HTTP 요청이 필요한가?
```

는 관련은 있지만 완전히 같은 개념은 아닙니다.

---

# 18. `refetchOnMountOrArgChange`

예를 들어:

```js
useGetProductsQuery(undefined, {
  refetchOnMountOrArgChange: true
});
```

라고 하면 컴포넌트가 mount될 때 기존 cache가 있어도 refetch할 수 있습니다.

또 숫자를 지정할 수도 있습니다.

```js
refetchOnMountOrArgChange: 30
```

이면 기존 fulfilled 결과가 일정 시간보다 오래됐을 때 새 요청을 수행하는 식으로 동작합니다. ([Redux Toolkit][1])

개념적으로:

```text
Component mount
       ↓
Cache 있음
       ↓
마지막 성공 시각 확인
       ↓
30초 이상 경과?
   ↙          ↘
 YES          NO
  ↓            ↓
refetch      Cache 사용
```

입니다.

---

# 19. Cache가 변경되는 경우

RTK Query Cache는 여러 가지 이유로 변경될 수 있습니다.

가장 대표적으로:

```text
① Query 성공

② Query refetch 성공

③ Optimistic Update

④ Pessimistic Update

⑤ updateQueryData()

⑥ upsertQueryData()

⑦ Mutation → tag invalidation → refetch

⑧ Cache entry 제거
```

등이 있습니다.

---

# 20. Mutation 응답은 Query Cache와 성격이 다르다

Query는:

```js
builder.query()
```

이고 주요 목적이:

```text
서버 데이터 조회
+
Cache
+
subscription
```

입니다.

Mutation은:

```js
builder.mutation()
```

이고 주요 목적이:

```text
서버 상태 변경
```

입니다.

예:

```js
addProduct
updateProduct
deleteProduct
```

Mutation 자체도 결과 상태를 관리하지만, 일반적으로 Query처럼 **같은 argument를 기반으로 장기간 공유하는 조회 Cache**라는 관점으로 생각하면 안 됩니다.

그래서 Mutation이 성공하면 흔히:

```text
Tag invalidation
        ↓
관련 Query Cache 무효화
        ↓
필요하면 Refetch
```

를 사용합니다.

---

# 21. Tag는 Cache 자체가 아니다

중요합니다.

```js
providesTags: ["Product"]
```

라고 했다고:

```text
Product tag = Cache
```

가 아닙니다.

Tag는 Cache를 **식별하고 무효화하기 위한 메타데이터**입니다.

예를 들어:

```js
getProducts: builder.query({
  query: () => "/products",

  providesTags: ["Product"]
})
```

Mutation:

```js
addProduct: builder.mutation({
  query: (product) => ({
    url: "/products",
    method: "POST",
    body: product
  }),

  invalidatesTags: ["Product"]
})
```

그러면:

```text
getProducts Cache
      │
      └── provides "Product"

addProduct Mutation
      │
      └── invalidates "Product"
                  ↓
          관련 Cache invalid
                  ↓
       active subscriber 있으면
                  ↓
              refetch
```

입니다. ([Redux Toolkit][4])

---

# 22. Tag invalidation 시 Cache가 무조건 삭제되는 것도 아니다

더 정확히 말하면 mutation이 tag를 invalidate하면 **해당 tag를 제공한 query cache entry가 stale/invalid 대상으로 처리**됩니다.

그리고 active subscription이 있다면 refetch가 발생합니다. ([Redux Toolkit][4])

```text
Mutation 성공
      ↓
invalidateTags
      ↓
관련 Query Cache 찾음
      ↓
active subscription 있음?
      │
      ├─ YES → refetch
      │
      └─ NO  → 상황에 맞게 Cache 정리
```

이 구조로 이해하는 것이 좋습니다.

---

# 23. RTK Query는 전역 normalized cache가 아니다

이것도 상당히 중요한 특징입니다.

예를 들어:

```js
useGetProductsQuery();
```

결과:

```js
[
  { id: 10, name: "Keyboard" },
  { id: 20, name: "Mouse" }
]
```

그리고:

```js
useGetProductQuery(10);
```

결과:

```js
{
  id: 10,
  name: "Keyboard"
}
```

이라고 하겠습니다.

RTK Query는 기본적으로 이것을:

```text
하나의 Product id=10 객체
```

로 완전히 합쳐서 모든 query가 같은 객체 하나를 참조하도록 만드는 전역 normalized cache가 아닙니다.

대략:

```text
getProducts(undefined)
   ↓
[
  Product 10,
  Product 20
]

getProduct(10)
   ↓
Product 10
```

이라는 **서로 다른 query cache entry**가 존재할 수 있습니다.

공식 문서에서도 RTK Query는 fully normalized/shared-across-queries cache가 아니라고 설명하며, 필요하면 tags, `transformResponse`, `createEntityAdapter`, `selectFromResult` 등을 조합할 수 있습니다. ([Redux Toolkit][1])

---

# 24. 그래서 같은 Product가 여러 Cache에 있을 수 있다

예:

```text
getProducts()
   └─ Product 10

getProduct(10)
   └─ Product 10

getFeaturedProducts()
   └─ Product 10
```

즉:

```text
Product 10
```

이 세 cache entry에 각각 존재할 수 있습니다.

따라서 product 10이 수정되었을 때 consistency를 유지하기 위해:

```text
tag invalidation
```

이 중요해집니다.

예:

```js
{ type: "Product", id: 10 }
```

tag를 사용하면 관련 query들을 refetch하도록 만들 수 있습니다.

---

# 25. Optimistic Update와 Cache

앞에서 이야기했던:

```js
api.util.updateQueryData()
```

도 바로 이 Cache를 수정합니다.

예:

```js
dispatch(
  productApi.util.updateQueryData(
    "getProduct",
    10,
    (draft) => {
      draft.name =
        "Mechanical Keyboard";
    }
  )
);
```

그러면:

```text
getProduct(10) Cache

Before
Keyboard

      ↓ updateQueryData

After
Mechanical Keyboard
```

가 됩니다.

그리고:

```js
useGetProductQuery(10);
```

을 사용하는 컴포넌트는 해당 cache entry를 구독하고 있으므로 즉시 리렌더링됩니다.

---

# 26. `updateQueryData()`에서 endpoint + arg가 중요한 이유

```js
updateQueryData(
  "getProduct",
  10,
  ...
)
```

여기서:

```text
"getProduct"
+
10
```

이 바로 어떤 cache entry를 수정할지를 결정합니다.

즉 RTK Query cache 구조를 이해하면 `updateQueryData()`도 바로 이해됩니다.

```text
endpointName
+
argument
      ↓
queryCacheKey
      ↓
특정 Cache Entry
      ↓
update
```

---

# 27. Cache lifecycle 전체 흐름

전체를 하나로 연결해 보겠습니다.

```text
React Component mount
        ↓
useGetProductQuery(10)
        ↓
queryCacheKey 생성
getProduct(10)
        ↓
Cache 존재?
   ↙           ↘
 NO            YES
 ↓              ↓
HTTP 요청       Cache 사용
 ↓
Server
 ↓
Response
 ↓
Cache 저장
        ↓
Component가 Cache 구독
        ↓
Cache 변경 시 리렌더링
        ↓
Component unmount
        ↓
subscription count 감소
        ↓
마지막 subscriber?
   ↙           ↘
 NO            YES
 ↓              ↓
Cache 유지     count = 0
                  ↓
           keepUnusedDataFor
               timer
                  ↓
          새 subscriber 없음
                  ↓
              Cache 삭제
```

이게 RTK Query cache의 기본 lifecycle입니다.

---

# 28. 네 컴포넌트 예제로 보면 훨씬 명확하다

다음이 있다고 하겠습니다.

```js
A → useGetUserQuery(1)

B → useGetUserQuery(2)

C → useGetUserQuery(3)

D → useGetUserQuery(3)
```

컴포넌트는 4개이지만 cache entry는:

```text
getUser(1)
getUser(2)
getUser(3)
```

총 3개입니다.

subscription count는:

```text
getUser(1) → 1
getUser(2) → 1
getUser(3) → 2
```

입니다.

따라서 HTTP 요청도 기본적으로 3개입니다. 공식 문서가 Cache lifetime을 설명할 때 바로 이와 같은 형태의 예제를 사용합니다. ([Redux Toolkit][1])

C가 unmount:

```text
getUser(3)
2 → 1
```

Cache 유지.

D도 unmount:

```text
getUser(3)
1 → 0
```

그제야:

```text
keepUnusedDataFor timer 시작
```

입니다.

---

# 29. `selectFromResult`도 Cache와 관련 있다

예를 들어:

```js
const { product } =
  useGetProductsQuery(undefined, {
    selectFromResult: ({ data }) => ({
      product:
        data?.find(p => p.id === 10)
    })
  });
```

이렇게 하면 전체 query cache를 공유하면서도 필요한 부분만 선택할 수 있습니다.

중요한 점은:

```text
새 HTTP 요청을 만드는 것이 아니라
기존 Query Cache에서 일부만 선택
```

한다는 것입니다.

그리고 선택된 결과가 바뀌지 않으면 불필요한 component rerender도 줄일 수 있습니다. ([Redux Toolkit][2])

---

# 30. Cache는 “서버 State의 클라이언트 복사본”

이 관점이 가장 중요합니다.

일반 Redux state:

```text
modalOpen
selectedTab
theme
form input
```

같은 것은:

> **Client State**

입니다.

RTK Query cache:

```text
products
users
orders
posts
```

같은 것은:

> **Server State의 Client-side Cache**

입니다.

즉:

```text
Server가 원본
       ↓
RTK Query Cache는 복사본
```

입니다.

그래서 Cache는 항상 이런 문제가 있습니다.

```text
Server 데이터 변경
       ↓
Client Cache는 옛날 데이터일 수 있음
```

바로 이 문제 때문에:

```text
refetch
tag invalidation
optimistic update
pessimistic update
polling
refetchOnFocus
refetchOnReconnect
```

같은 기능들이 존재합니다.

---

# 31. Cache와 Redux Store 관계를 한 번 더 정리하면

```text
Redux Store
│
├─ 일반 Redux State
│
│   ├─ auth
│   ├─ cart
│   └─ ui
│
└─ RTK Query State
    │
    └─ productApi
        │
        ├─ Query Cache
        │   ├─ getProducts(undefined)
        │   ├─ getProduct(10)
        │   └─ getProduct(20)
        │
        ├─ Mutation State
        │
        ├─ subscription 정보
        │
        └─ invalidation 관련 정보
```

즉 RTK Query가 Redux와 별개의 cache 시스템인 것이 아닙니다.

**Redux Store 위에서 동작하는 서버 상태 관리 시스템**입니다.

---

# 핵심을 압축하면

RTK Query cache는 다음 7개를 기억하면 거의 완전히 이해한 것입니다.

1. **Query 결과는 Redux Store 안에 Cache된다.**
2. **Cache의 식별 기준은 기본적으로 `endpoint + serialized argument`이다.**
3. **같은 cache key를 사용하는 컴포넌트들은 하나의 cache entry를 공유하고 요청도 dedupe된다.**
4. **Query Hook을 사용하는 컴포넌트는 해당 cache entry를 subscribe한다.**
5. **마지막 subscriber가 사라져도 기본적으로 60초 동안 cache가 유지된다.**
6. **Mutation의 tag invalidation, refetch, optimistic update 등으로 cache를 최신 상태에 맞춘다.**
7. **RTK Query는 기본적으로 전역 normalized entity cache가 아니라 query 결과별 cache를 관리한다.** ([Redux Toolkit][1])

그리고 전체를 한 문장으로 정리하면:

> **RTK Query Cache는 `endpoint + argument`를 기반으로 서버 응답을 Redux Store에 저장하고, 동일한 요청을 여러 컴포넌트가 공유하도록 하며, subscription count·cache lifetime·refetch·tag invalidation·수동 cache update 등을 이용해 서버 데이터와 React UI 사이의 동기화를 관리하는 시스템입니다.**

[1]: https://redux-toolkit.js.org/rtk-query/usage/cache-behavior?utm_source=chatgpt.com "Cache Behavior | Redux Toolkit"
[2]: https://redux-toolkit.js.org/rtk-query/usage/queries?utm_source=chatgpt.com "Queries | Redux Toolkit"
[3]: https://redux-toolkit.js.org/rtk-query/api/createApi?utm_source=chatgpt.com "createApi | Redux Toolkit"
[4]: https://redux-toolkit.js.org/rtk-query/usage/automated-refetching?utm_source=chatgpt.com "Automated Re-fetching | Redux Toolkit"
