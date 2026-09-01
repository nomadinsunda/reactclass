> **Tag는 "이 Cache가 어떤 데이터와 관련 있는지 붙여놓은 이름표"이고, Invalidation은 "그 이름표가 붙은 Cache는 이제 믿지 마"라는 신호이며, Refetch는 "그러면 서버에서 최신 데이터를 다시 가져오자"는 실제 동작입니다.**

예를 들어 상품 목록부터 보겠습니다.

```js
getProducts: builder.query({
  query: () => "/products",

  providesTags: ["Product"],
})
```

서버에서:

```text
GET /products
```

를 해서 다음 데이터를 받았습니다.

```js
[
  { id: 1, name: "Keyboard" },
  { id: 2, name: "Mouse" }
]
```

그러면 RTK Query에는 대략 이런 Cache가 생깁니다.

```text
getProducts(undefined)

┌─────────────────────────────┐
│ Cache                       │
│                             │
│ Keyboard                    │
│ Mouse                       │
│                             │
│ Tag: "Product"              │
└─────────────────────────────┘
```

여기서:

```js
providesTags: ["Product"]
```

의 의미는

> **"`getProducts`의 Cache는 `Product`라는 데이터와 관련 있습니다."**

라고 RTK Query에게 알려주는 것입니다.

즉 Tag는 **Cache에 붙이는 이름표**라고 생각하면 됩니다.

---

그런데 사용자가 새로운 상품을 추가했다고 해보겠습니다.

```js
addProduct: builder.mutation({
  query: (product) => ({
    url: "/products",
    method: "POST",
    body: product,
  }),

  invalidatesTags: ["Product"],
})
```

사용자가:

```js
addProduct({
  name: "Monitor"
});
```

를 호출합니다.

서버에서는 이제:

```text
Server

Keyboard
Mouse
Monitor       ← 새로 추가됨
```

이 되었습니다.

그런데 클라이언트의 기존 Cache는 아직:

```text
RTK Query Cache

Keyboard
Mouse
```

입니다.

즉 **Server와 Cache가 달라졌습니다.**

```text
Server                  Cache

Keyboard                Keyboard
Mouse                   Mouse
Monitor        ← 없음!
```

바로 이 문제를 해결하기 위해 `invalidatesTags`가 등장합니다.

```js
invalidatesTags: ["Product"]
```

이 말은:

> **"`Product`라는 Tag와 관련된 Cache는 이제 최신이라고 보장할 수 없다."**

라는 뜻입니다.

여기서 중요한 점은 **Invalidation이 데이터를 수정한다는 뜻이 아니라는 것**입니다.

```text
invalidate
     ↓
Cache 데이터 직접 수정 ❌

invalidate
     ↓
"이 Cache는 이제 최신이 아닐 수 있음" ⭕
```

---

그럼 RTK Query가 `"Product"`라는 Tag와 관련된 Query Cache를 찾아봅니다.

앞에서:

```js
getProducts: builder.query({
  ...
  providesTags: ["Product"]
})
```

라고 했죠.

그래서 연결됩니다.

```text
getProducts
    │
    └─ providesTags: ["Product"]
                ↑
                │
        같은 Tag 이름
                │
                ↓
addProduct
    │
    └─ invalidatesTags: ["Product"]
```

즉:

```text
getProducts
"나는 Product 데이터를 제공해"
        │
        │
        │ Product
        │
        ↓
addProduct
"Product 데이터가 변경됐어!"
```

라고 생각하면 됩니다.

이것이 `providesTags`와 `invalidatesTags`의 관계입니다.

---

### 그러면 Refetch는 언제 나오는가?

이제 RTK Query는 생각합니다.

```text
Product Tag가 invalidate 됐다.
        ↓
getProducts Cache가 Product Tag를 가지고 있다.
        ↓
그런데 현재 Component가
getProducts를 사용하고 있다.
        ↓
그럼 최신 데이터가 필요하겠네.
        ↓
Refetch!
```

그래서 다시:

```http
GET /products
```

를 보냅니다.

서버에서:

```js
[
  { id: 1, name: "Keyboard" },
  { id: 2, name: "Mouse" },
  { id: 3, name: "Monitor" }
]
```

를 받아옵니다.

그리고 기존 Cache를 최신 결과로 갱신합니다.

```text
기존 Cache

Keyboard
Mouse

        ↓ Refetch

새 Cache

Keyboard
Mouse
Monitor
```

Component는 이 Cache를 구독하고 있으므로:

```text
Cache 변경
    ↓
Hook이 변경 감지
    ↓
Component 리렌더링
    ↓
Monitor도 화면에 표시
```

됩니다.

---

## 전체를 한 번에 보면

이 흐름을 정확히 잡으시면 됩니다.

```text
① 최초 조회

useGetProductsQuery()
        ↓
GET /products
        ↓
Server
        ↓
Keyboard
Mouse
        ↓
RTK Query Cache
        ↓
Tag: "Product" 부착
(providesTags)
```

그리고:

```text
② 상품 추가

addProduct()
        ↓
POST /products
        ↓
Server

Keyboard
Mouse
Monitor
```

Mutation이 성공하면:

```text
③ invalidatesTags

invalidatesTags: ["Product"]

        ↓

"Product Tag와 관련된 Cache는
이제 최신이 아닐 수 있다."
```

RTK Query가 관련 Cache를 찾습니다.

```text
④ Tag Matching

"Product"
    │
    ├──── getProducts Cache
    │       providesTags: ["Product"]
    │
    └──── 일치!
```

현재 이 Query를 Component가 사용하고 있다면:

```text
⑤ Refetch

GET /products
      ↓
Server
      ↓
Keyboard
Mouse
Monitor
      ↓
Cache 갱신
      ↓
Component 리렌더링
```

따라서 전체 핵심은:

```text
Mutation
   ↓
Server 데이터 변경
   ↓
invalidatesTags
   ↓
관련 Tag를 가진 Query Cache 찾기
   ↓
active subscription 존재
   ↓
Refetch
   ↓
Server의 최신 데이터
   ↓
Cache 갱신
   ↓
UI 갱신
```

입니다.

---

### `providesTags`라는 이름도 이해하면 쉽습니다

`provide`는 **제공하다**라는 뜻이죠.

```js
providesTags: ["Product"]
```

즉 이 Query가:

> **"내 Cache는 Product라는 데이터를 제공하고 있습니다."**

라고 표시하는 것입니다.

```text
getProducts Query
       ↓
Cache 생성
       ↓
"나는 Product 관련 Cache야"
       ↓
providesTags: ["Product"]
```

반면 `invalidate`는 **무효화하다**라는 뜻입니다.

```js
invalidatesTags: ["Product"]
```

Mutation이:

> **"Product 데이터가 변경됐으니 Product와 관련된 기존 Cache를 더 이상 최신이라고 믿지 마."**

라고 알려주는 것입니다.

그래서 이름 자체로 보면:

```text
providesTags
     ↓
Query가 자신의 Cache에
"나는 어떤 데이터인가?" 표시


invalidatesTags
     ↓
Mutation이
"어떤 데이터가 변경됐는가?" 표시
```

입니다.

---

## Tag는 Cache가 아닙니다

여기가 특히 중요합니다.

```text
Tag ≠ Cache
```

Tag에는 실제 상품 데이터가 들어 있는 게 아닙니다.

```text
Cache
┌───────────────────────┐
│ Keyboard              │
│ Mouse                 │
│                       │
│ Tag → "Product"       │
└───────────────────────┘
```

즉 Tag는 **Cache와 Mutation을 연결하기 위한 식별 정보**에 가깝습니다.

그래서 저는 학생들에게 Tag를 **"Cache에 붙이는 포스트잇"**이라고 설명하는 것을 추천합니다.

```text
┌─────────────────────────┐
│ getProducts Cache       │
│                         │
│ Keyboard                │
│ Mouse                   │
│                         │
│       ┌─────────────┐   │
│       │ 🏷 Product  │   │
│       └─────────────┘   │
└─────────────────────────┘
```

Mutation이:

```text
"Product가 변경됐어!"
```

라고 하면 RTK Query가:

```text
Product 포스트잇 붙어 있는 Cache가 어디 있지?
```

를 찾는 것입니다.

---

## 그런데 왜 그냥 Mutation 결과로 Cache를 수정하지 않을까?

좋은 질문이 여기서 나옵니다.

상품을 추가했으면 그냥:

```text
기존 Cache + 새 Product
```

하면 될 것 같죠.

그렇게 할 수도 있습니다. 앞에서 배운:

```js
updateQueryData()
```

가 바로 그런 **수동 Cache Update** 방법입니다.

하지만 Tag 방식은 다른 전략입니다.

```text
Tag Invalidation 방식

"서버 데이터가 변경됐다."
        ↓
"기존 Cache를 직접 고치지 말고"
        ↓
"서버에서 다시 받아오자."
```

즉:

```text
Tag Invalidation
        ↓
Refetch
        ↓
Server를 Source of Truth로 사용
```

하는 방식입니다.

그래서 구현이 매우 간단합니다.

```js
getProducts: builder.query({
  query: () => "/products",

  providesTags: ["Product"],
}),

addProduct: builder.mutation({
  query: (product) => ({
    url: "/products",
    method: "POST",
    body: product,
  }),

  invalidatesTags: ["Product"],
}),
```

이것만으로:

```text
POST 성공
   ↓
Product invalidation
   ↓
GET /products 자동 refetch
   ↓
Cache 최신화
```

를 RTK Query가 처리합니다.

---

## Optimistic Update와 비교하면 더 명확합니다

앞에서 배운 낙관적 업데이트는:

```text
Mutation
   ↓
updateQueryData()
   ↓
Cache 직접 수정
   ↓
UI 즉시 변경
   ↓
Server 요청
```

입니다.

Tag Invalidation은:

```text
Mutation
   ↓
Server 성공
   ↓
invalidateTags
   ↓
Query refetch
   ↓
Cache 교체
   ↓
UI 변경
```

입니다.

그래서:

| 방식                  | Cache를 최신화하는 방법         |
| ------------------- | ----------------------- |
| Tag Invalidation    | 서버에서 다시 가져온다            |
| `updateQueryData()` | Cache를 직접 수정한다          |
| Optimistic Update   | 서버 응답 전에 Cache를 먼저 수정한다 |

이 차이를 이해하면 앞에서 공부한 내용들이 하나로 연결됩니다.

---

그리고 나중에는 단순히 `"Product"` 하나만 쓰지 않고 **ID까지 붙이는 Tag**를 배우게 됩니다.

```js
{ type: "Product", id: 10 }
```

그러면:

```text
전체 Product Cache를 다시 요청
```

하는 대신:

```text
Product 10과 관련된 Cache만
정확하게 invalidation
```

할 수 있습니다.

하지만 지금 단계에서는 이것부터 확실히 잡으시면 됩니다.

> **Query의 `providesTags`는 "이 Cache가 어떤 서버 데이터와 관련 있는지 표시"하고, Mutation의 `invalidatesTags`는 "그 서버 데이터가 변경되었다고 알리며", RTK Query는 해당 Tag와 연결된 활성 Query를 `refetch`하여 Cache를 서버의 최신 데이터와 다시 맞춥니다.**

가장 짧게 외우면 **`Tag = 이름표 → Invalidate = 낡았다고 표시 → Refetch = 서버에서 다시 가져오기`**입니다.
