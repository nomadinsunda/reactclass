# PART 5. RTK Query

## — Server State Fetching, Caching, Synchronization

RTK Query를 처음 접하면 다음과 같은 용어들이 한꺼번에 등장합니다.

```text
createApi
fetchBaseQuery
Query
Mutation
Cache
Cache Key
Subscription
Tag
Invalidation
Refetch
...
```

하나씩 보면 어려운 개념은 아니지만, 처음부터 내부 구조를 모두 이해하려고 하면 RTK Query가 상당히 복잡하게 느껴질 수 있습니다.

따라서 이번 PART에서는 **사용법부터 시작해서 점차 내부 구조로 들어가는 방식**으로 학습합니다.

```text
[1단계: 초급]

fetch + useEffect의 불편함
        ↓
RTK Query가 왜 필요한가?
        ↓
가장 간단한 Query 사용


[2단계: 핵심]

Server State
        ↓
Query Cache
        ↓
Subscription
        ↓
Mutation
        ↓
Cache Invalidation


[3단계: 내부 원리]

Cache Key
Request Deduplication
Cache Lifetime
Reducer / Middleware
Refetch


[4단계: 실전 / 중급]

Lazy Query
transformResponse
injectEndpoints
Custom baseQuery
onQueryStarted
queryFulfilled
apiSlice.util
...
```

> **처음 읽을 때 모든 것을 암기할 필요는 없습니다.**
>
> 먼저 **"RTK Query는 왜 필요한가?"**와
> **"Component가 Server Data를 직접 관리하지 않고 RTK Query의 Cache를 사용한다."**
>
> 이 두 가지를 이해하는 것이 가장 중요합니다.

---

# CHAPTER 1. 왜 RTK Query가 필요한가?

## 1. 우리가 지금까지 Server Data를 가져오던 방법

React 애플리케이션에서 상품 목록을 Server로부터 가져온다고 생각해봅시다.

가장 직접적인 방법은 `fetch()`와 `useEffect()`를 사용하는 것입니다.

```jsx
import { useEffect, useState } from "react";

function ProductList() {

    const [products, setProducts] = useState([]);
    const [isLoading, setIsLoading] = useState(false);
    const [error, setError] = useState(null);

    useEffect(() => {

        const fetchProducts = async () => {

            try {

                setIsLoading(true);

                const response =
                    await fetch("/api/products");

                const data =
                    await response.json();

                setProducts(data);

            } catch (error) {

                setError(error);

            } finally {

                setIsLoading(false);
            }
        };

        fetchProducts();

    }, []);

    // ...
}
```

상품 목록 하나를 가져오기 위해 벌써 다음 상태들을 직접 관리하고 있습니다.

```text
products
isLoading
error
```

그리고 다음 작업도 직접 해야 합니다.

```text
Component Mount
      ↓
useEffect()
      ↓
fetch()
      ↓
HTTP Request
      ↓
Response
      ↓
setProducts()
      ↓
Component Re-render
```

여기까지는 괜찮아 보입니다.

하지만 애플리케이션이 커지기 시작하면 문제가 달라집니다.

---

# 2. Server Data가 많아지면?

실제 애플리케이션에는 상품만 존재하지 않습니다.

```text
Products
Users
Orders
Reviews
Cart
Search Results
Notifications
...
```

각 데이터마다 다음을 반복해야 한다면 어떨까요?

```text
Request

Loading State

Error State

Response 처리

State 저장
```

더 큰 문제는 여기서 끝나지 않습니다.

예를 들어 상품 목록을 이미 가져왔다고 해봅시다.

```text
Server
   ↓
GET /products
   ↓
Browser

[A, B, C]
```

다른 Component에서도 똑같은 상품 목록이 필요하다면 다시 Request를 보내야 할까요?

```text
Component A
     ↓
GET /products

Component B
     ↓
GET /products

Component C
     ↓
GET /products
```

그리고 이미 가져온 데이터는 얼마나 유지해야 할까요?

```text
방금 받은 데이터인데
다시 Server에서 가져와야 하나?

기존 데이터를 재사용하면 안 되나?
```

더 어려운 문제도 있습니다.

상품을 하나 추가했다고 생각해봅시다.

```text
POST /products
```

Server에는 새로운 상품이 생겼습니다.

```text
Server

A
B
C
D   ← 새 상품
```

그런데 Browser가 가지고 있는 상품 목록은 여전히:

```text
Cache

A
B
C
```

일 수 있습니다.

즉,

```text
Server State
     ≠
Client가 가지고 있는 Server Data
```

가 되어버립니다.

따라서 Server Data를 관리할 때는 단순히 **"HTTP Request를 어떻게 보낼까?"**만 생각해서는 안 됩니다.

우리는 결국 다음 문제를 해결해야 합니다.

```text
Fetching

Loading / Error

Caching

Cache 공유

중복 Request 방지

Cache Lifetime

Refetch

Cache Invalidation

Server ↔ Client Synchronization
```

바로 이 문제를 해결하기 위해 Redux Toolkit이 제공하는 도구가 **RTK Query**입니다.

---

# 3. RTK Query란?

RTK Query를 처음에는 다음과 같이 이해하면 충분합니다.

> **RTK Query는 Server에서 데이터를 가져오고, 가져온 데이터를 Cache하고, React Component들이 그 데이터를 사용할 수 있도록 관리해주는 Redux Toolkit의 Server State 관리 도구입니다.**

즉 단순히:

```javascript
fetch("/api/products");
```

를 편하게 만들어주는 Wrapper가 아닙니다.

RTK Query가 실제로 해결하려는 문제는:

```text
             Server
                ↕
             Fetching
                ↕
          RTK Query Cache
                ↕
        React Components
```

입니다.

---

# 4. PART 4의 `createAsyncThunk()`와 무엇이 다른가?

PART 4에서는 Redux의 비동기 작업을 다음과 같이 처리했습니다.

```text
React Component
      ↓
dispatch(fetchProducts())
      ↓
Thunk Middleware
      ↓
createAsyncThunk()
      ↓
API Request
      ↓
Promise
      ↓
pending / fulfilled / rejected
      ↓
extraReducers
      ↓
Redux Store
      ↓
React Re-render
```

`createAsyncThunk()`는 여전히 매우 유용합니다.

하지만 Server State를 관리하려면 단순한 비동기 Workflow 외에도:

```text
Cache

Cache Key

Subscription

Request Deduplication

Cache Lifetime

Refetch

Invalidation
```

같은 문제가 추가됩니다.

따라서 역할을 크게 구분하면:

```text
일반적인 비동기 Redux Workflow
             ↓
     createAsyncThunk()


Server State
Fetching + Caching + Synchronization
             ↓
          RTK Query
```

라고 볼 수 있습니다.

---

# CHAPTER 2. 가장 간단한 RTK Query부터 시작하기

# 5. 먼저 전체 코드를 보자

RTK Query를 처음부터 내부 구조로 이해하려고 하지 말고 먼저 실제 사용 모습을 살펴봅시다.

```javascript
import {
    createApi,
    fetchBaseQuery
} from "@reduxjs/toolkit/query/react";

export const apiSlice = createApi({

    reducerPath: "api",

    baseQuery: fetchBaseQuery({
        baseUrl: "/api"
    }),

    endpoints: builder => ({

        getProducts: builder.query({

            query: () => "/products"

        })

    })

});

export const {
    useGetProductsQuery
} = apiSlice;
```

Component에서는:

```jsx
function ProductList() {

    const {
        data: products,
        isLoading,
        error
    } = useGetProductsQuery();

    if (isLoading) {
        return <p>Loading...</p>;
    }

    if (error) {
        return <p>Error</p>;
    }

    return (
        <ul>
            {products?.map(product => (
                <li key={product.id}>
                    {product.name}
                </li>
            ))}
        </ul>
    );
}
```

여기서 가장 먼저 주목해야 할 부분은 이것입니다.

```javascript
useGetProductsQuery();
```

기존에는 우리가 직접:

```text
useEffect
fetch
data state
loading state
error state
```

를 관리했습니다.

RTK Query에서는 Query Hook을 통해 이 작업을 훨씬 선언적으로 처리할 수 있습니다.

---

# 6. `createApi()`

RTK Query의 출발점은 `createApi()`입니다.

```javascript
const apiSlice = createApi({
    // ...
});
```

쉽게 말하면:

> **"우리 애플리케이션에서 Server API를 RTK Query로 어떻게 관리할 것인지 정의하는 중심 설정"**

이라고 생각하면 됩니다.

기본 구조는:

```text
createApi()

├── reducerPath
├── baseQuery
├── tagTypes
└── endpoints
```

입니다.

하지만 처음부터 네 가지를 모두 이해할 필요는 없습니다.

먼저:

```text
createApi
   │
   ├── baseQuery
   │
   └── endpoints
```

만 이해하면 됩니다.

---

# 7. `fetchBaseQuery()`

`fetchBaseQuery()`는 RTK Query가 HTTP Request를 수행할 때 사용할 수 있는 기본 Base Query입니다.

```javascript
baseQuery: fetchBaseQuery({
    baseUrl: "/api"
})
```

그리고 Endpoint에서:

```javascript
query: () => "/products"
```

라고 작성하면:

```text
baseUrl
/api

   +

Endpoint URL
/products

   ↓

/api/products
```

가 됩니다.

전체적으로는:

```text
Endpoint
   ↓
baseQuery
   ↓
fetchBaseQuery
   ↓
fetch()
   ↓
HTTP Request
   ↓
Server
```

라고 이해하면 됩니다.

---

# 8. Endpoint란?

Server와 어떤 작업을 할 것인지 정의하는 부분입니다.

```javascript
endpoints: builder => ({

    getProducts: builder.query({

        query: () => "/products"

    })

})
```

RTK Query의 Endpoint는 크게 두 종류입니다.

```text
Endpoint
   │
   ├── Query
   │
   └── Mutation
```

지금은 먼저 Query부터 살펴보겠습니다.

---

# 9. Query

Query는 Server State를 **조회하고 Cache하기 위한 Endpoint**입니다.

```javascript
getProducts: builder.query({

    query: () => "/products"

})
```

의미는 간단합니다.

```text
getProducts
     ↓
GET /api/products
     ↓
상품 데이터 조회
     ↓
RTK Query Cache
```

여기서 중요한 단어가 처음 등장합니다.

**Cache**입니다.

RTK Query는 Response를 단순히 Component에 전달하고 버리는 것이 아니라 Query 결과를 관리합니다.

이것이 뒤에서 매우 중요한 역할을 합니다.

---

# 10. Generated Hook

`@reduxjs/toolkit/query/react`의 `createApi()`를 사용하면 Endpoint 정의를 기반으로 React Hook을 생성할 수 있습니다.

예를 들어:

```javascript
getProducts: builder.query(...)
```

가 있으면:

```javascript
useGetProductsQuery()
```

를 사용할 수 있습니다.

```text
getProducts
     ↓
Generated Hook
     ↓
useGetProductsQuery()
```

그래서 Component에서는:

```javascript
const {
    data,
    isLoading,
    error
} = useGetProductsQuery();
```

처럼 사용할 수 있습니다.

---

# 11. Query Hook을 호출하면 무슨 일이 일어나는가?

처음에는 다음 정도로 이해하면 충분합니다.

```text
Component
    ↓
useGetProductsQuery()
    ↓
RTK Query
    ↓
필요하면 Server Request
    ↓
Response
    ↓
Data 관리
    ↓
Component Re-render
```

하지만 이것은 입문용 설명입니다.

실제로는 중간에 매우 중요한 것이 있습니다.

```text
Query Cache
```

조금 더 정확하게 표현하면:

```text
Component
    ↓
useGetProductsQuery()
    ↓
RTK Query
    ↓
Cache 확인
    ↓
필요한 경우 Request
    ↓
Response
    ↓
Cache 저장
    ↓
Component가 Cache 사용
    ↓
Component Re-render
```

이제부터 RTK Query의 진짜 핵심으로 들어갑니다.

---

# CHAPTER 3. RTK Query의 핵심 — Server State와 Query Cache

# 12. Client State와 Server State

RTK Query를 제대로 이해하려면 두 종류의 State를 구분해야 합니다.

```text
Application State
       │
       ├── Client State
       │
       └── Server State
```

## Client State

Client State는 애플리케이션 자체가 소유하는 상태입니다.

예:

```text
Modal 열림 여부

Sidebar 열림 여부

현재 선택된 Tab

검색 입력값

Theme

Wizard 진행 단계
```

이러한 State는:

```text
Local Client State
       ↓
    useState()
```

또는:

```text
Shared Client State
       ↓
   createSlice()
```

로 관리하기 좋습니다.

---

# 13. Server State

Server State는 원본 데이터가 Browser가 아니라 **Server에 존재하는 State**입니다.

예:

```text
상품 정보
회원 정보
주문
리뷰
게시글
검색 결과
알림
```

실제 원본은:

```text
Database
    ↓
Backend Server
```

에 존재합니다.

Browser가 가지고 있는 것은 그 데이터의 복사본입니다.

```text
Database
    ↓
Backend Server
    ↓
HTTP
    ↓
Browser
    ↓
Cached Server Data
```

따라서 Browser의 Server Data에는 특별한 문제가 발생합니다.

> **"내가 가지고 있는 이 데이터가 아직 Server의 최신 데이터와 같은가?"**

이 문제가 RTK Query에서 Cache와 Synchronization이 중요한 이유입니다.

---

# 14. 가장 중요한 Mental Model

RTK Query를 처음 배우면 다음처럼 생각하기 쉽습니다.

```text
Component
    ↓
Server Request
    ↓
Response
    ↓
Component Data
```

하지만 RTK Query에서는 다음과 같이 생각하는 것이 더 정확합니다.

```text
                 Server
                   ↕
                RTK Query
                   ↓
               Query Cache
                   ↑
              Subscription
                   ↑
               Component
```

핵심은 이것입니다.

> **Component가 Server Data 자체를 소유하는 것이 아니라, RTK Query가 관리하는 Query Cache를 Component가 사용한다.**

그리고 Query Hook은 단순히 Request를 보내는 함수가 아닙니다.

```javascript
useGetProductsQuery();
```

는 개념적으로:

```text
"상품 데이터를 주세요"
        +
"이 Query 결과를 사용하겠습니다"
```

라는 의미를 동시에 가집니다.

이 Mental Model이 이해되면 이후의:

```text
Cache Key
Subscription
Deduplication
Cache Lifetime
Tag
Invalidation
Refetch
```

가 하나의 시스템으로 연결됩니다.

---

# 15. Query Argument와 Cache Key

특정 상품 하나를 조회한다고 해봅시다.

```javascript
useGetProductQuery(10);
```

여기서 `10`은 **Query Argument**입니다.

RTK Query는 개념적으로:

```text
Endpoint
getProduct

    +

Argument
10

    ↓

Query Cache Entry
```

를 구성합니다.

따라서:

```javascript
useGetProductQuery(10);

useGetProductQuery(20);
```

은 서로 다른 Cache Entry를 사용합니다.

```text
getProduct(10)
      ↓
Cache Entry A


getProduct(20)
      ↓
Cache Entry B
```

즉 Cache는 단순히 Endpoint 이름만으로 구분되는 것이 아닙니다.

> **Endpoint + Query Argument의 직렬화된 결과가 Query Cache를 식별하는 핵심이 됩니다.**

---

# 16. Cache와 Subscription

여러 Component가 동일한 Query를 사용한다고 해봅시다.

```text
Component A ───┐
               │
Component B ───┼──→ Query Cache Entry
               │
Component C ───┘
```

세 Component가 모두:

```javascript
useGetProductQuery(10);
```

을 사용한다면 동일한 Query Cache Entry를 사용할 수 있습니다.

각 Component가 각각:

```text
fetch
data state
loading state
error state
```

를 독립적으로 만드는 것이 아닙니다.

---

# 17. Subscription이란?

Component에서 Query Hook을 사용하면 해당 Query Cache Entry를 **구독(Subscription)**하게 됩니다.

```text
Component
    ↓
Query Hook
    ↓
Subscription
    ↓
Query Cache Entry
```

쉽게 표현하면 Component가 RTK Query에게:

> **"나는 이 Query 결과를 사용하고 있어."**

라고 알려주는 관계라고 생각할 수 있습니다.

Component가 Unmount되면:

```text
Component Unmount
       ↓
Subscription 제거
```

가 됩니다.

하지만 중요한 점이 있습니다.

> **Subscription이 사라졌다고 Cache가 반드시 즉시 삭제되는 것은 아닙니다.**

---

# 18. Request Deduplication

세 Component가 거의 동시에 같은 상품을 필요로 한다고 생각해봅시다.

```text
Component A ─┐
Component B ─┼──→ getProduct(10)
Component C ─┘
                  ↓
             Cache Entry
```

RTK Query가 Query Cache를 중심으로 Server State를 공유하기 때문에 동일한 Query에 대한 불필요한 중복 Request를 줄일 수 있습니다.

이를 **Request Deduplication**이라고 합니다.

즉:

```text
Component A → GET /products/10
Component B → GET /products/10
Component C → GET /products/10
```

처럼 무조건 세 번 요청하는 구조로 생각해서는 안 됩니다.

---

# 19. Cache Lifetime

마지막 Subscription이 사라졌다고 Cache를 바로 제거하면 어떨까요?

사용자가 상품 상세 화면에서 다른 화면으로 이동했다가 바로 돌아왔는데 다시 Server Request가 필요할 수 있습니다.

그래서 RTK Query는 사용되지 않는 Cache도 일정 시간 유지할 수 있습니다.

```text
Last Subscription 제거
          ↓
      Cache 유지
          ↓
  keepUnusedDataFor
          ↓
       시간 경과
          ↓
      Cache 제거
```

예를 들어:

```text
상품 상세
    ↓
다른 화면
    ↓
잠시 후 상품 상세로 복귀
```

같은 상황에서 기존 Cache를 재사용할 수 있습니다.

---

# CHAPTER 4. Server State를 변경해보자 — Mutation

# 20. Query와 Mutation

Endpoint에는 두 가지 중요한 종류가 있습니다.

```text
Endpoint
   │
   ├── Query
   │
   └── Mutation
```

Query는 Server State를 **조회**합니다.

```javascript
getProducts: builder.query({

    query: () => "/products"

})
```

Mutation은 Server State를 **변경**합니다.

```javascript
createProduct: builder.mutation({

    query: product => ({

        url: "/products",

        method: "POST",

        body: product

    })

})
```

가장 기본적인 구분은:

```text
Query
=
Server State 조회 + Cache


Mutation
=
Server State 변경
```

입니다.

---

# 21. Mutation Hook

Mutation Endpoint를 만들면 Hook도 생성됩니다.

```text
createProduct
      ↓
Generated Hook
      ↓
useCreateProductMutation()
```

사용:

```javascript
const [
    createProduct,
    {
        isLoading,
        error
    }
] = useCreateProductMutation();
```

그리고 사용자 이벤트에서:

```javascript
createProduct({
    name: "Keyboard"
});
```

처럼 실행할 수 있습니다.

Query와 비교하면 중요한 차이가 있습니다.

```text
Query

Component가 데이터를 필요로 함
        ↓
자동 Query + Subscription


Mutation

사용자가 Server Data를 변경
        ↓
Mutation Trigger 호출
```

---

# 22. Cache가 있기 때문에 새로운 문제가 생긴다

Cache는 매우 유용하지만 새로운 문제를 만듭니다.

처음 Server가:

```text
Server

A
B
```

이고 Query 결과가 Cache되었다고 합시다.

```text
Server          Cache

A               A
B               B
```

이제 Mutation으로 상품 `C`를 추가합니다.

```text
POST /products
```

Server는:

```text
Server

A
B
C
```

가 됩니다.

하지만 기존 Cache는:

```text
Cache

A
B
```

입니다.

즉:

```text
Server State
     ≠
Cached Server State
```

가 되어버렸습니다.

이 문제를 해결하기 위해 **Cache Invalidation**이 필요합니다.

---

# CHAPTER 5. Cache와 Server를 동기화하자

# 23. Tag는 왜 필요한가?

RTK Query는 Query Cache와 Mutation 사이의 관계를 표현하기 위해 **Tag**를 사용할 수 있습니다.

먼저:

```javascript
export const apiSlice = createApi({

    tagTypes: [
        "Product"
    ],

    // ...
});
```

Tag를 처음에는 아주 단순하게 생각하면 됩니다.

> **"이 Cache가 어떤 종류의 Server State와 관련되어 있는가?"를 표시하는 Label**

입니다.

---

# 24. `providesTags`

상품 목록 Query가:

```javascript
getProducts: builder.query({

    query: () => "/products",

    providesTags: [
        {
            type: "Product",
            id: "LIST"
        }
    ]

})
```

라고 되어 있다고 해봅시다.

개념적으로:

```text
getProducts Query
       ↓
Query Cache Entry
       ↓
   provides
       ↓
 Product:LIST
```

입니다.

즉:

> **"이 Query Cache는 Product 목록과 관련된 Cache입니다."**

라고 표시하는 것입니다.

---

# 25. `invalidatesTags`

상품을 생성하면 Server의 Product 목록이 변경됩니다.

```javascript
createProduct: builder.mutation({

    query: product => ({

        url: "/products",

        method: "POST",

        body: product

    }),

    invalidatesTags: [
        {
            type: "Product",
            id: "LIST"
        }
    ]

})
```

의미는:

```text
createProduct
      ↓
Server State 변경
      ↓
Product:LIST
      ↓
Invalidate
```

입니다.

쉽게 말하면:

> **"상품 목록과 관련된 기존 Cache를 더 이상 최신이라고 믿지 마라."**

라는 의미입니다.

---

# 26. Cache Invalidation → Automatic Refetch

이제 `providesTags`와 `invalidatesTags`가 연결됩니다.

```text
getProducts Query
       ↓
Cache 생성
       ↓
providesTags
Product:LIST


createProduct Mutation
       ↓
Server 변경
       ↓
invalidatesTags
Product:LIST
       ↓
관련 Cache Invalid
       ↓
Active Subscription 존재
       ↓
Refetch
       ↓
새 Response
       ↓
Cache Update
       ↓
Component Update
```

이것이 RTK Query에서 가장 중요한 흐름 중 하나입니다.

즉:

```text
Mutation
   ↓
Server 변경
   ↓
Cache Invalidation
   ↓
Refetch
   ↓
Cache Update
   ↓
Component Update
```

를 통해 Server State와 Client Cache의 동기화를 도와줍니다.

---

# 27. Entity Tag와 `LIST` Tag

Tag는 목록 전체뿐 아니라 개별 Entity에도 붙일 수 있습니다.

```text
Product:10
Product:20
Product:30
Product:LIST
```

예:

```javascript
providesTags: result =>

    result

        ? [
            ...result.map(product => ({
                type: "Product",
                id: product.id
            })),

            {
                type: "Product",
                id: "LIST"
            }
        ]

        : [
            {
                type: "Product",
                id: "LIST"
            }
        ]
```

여기서 `"LIST"`는 Database의 Primary Key가 아닙니다.

> **Tag의 `id`는 반드시 Entity Primary Key일 필요가 없습니다.**

따라서:

```text
Product:LIST
User:PROFILE
Order:HISTORY
Review:MINE
```

처럼 의미 기반 ID를 사용해 Cache Group을 표현할 수도 있습니다.

---

# CHAPTER 6. Query를 조금 더 편리하게 사용하기

# 28. 조건부 Query — `skip`

모든 Query가 Component Mount 즉시 실행되어야 하는 것은 아닙니다.

예를 들어 `productId`가 존재할 때만 조회하고 싶다면:

```javascript
const { data } =
    useGetProductQuery(productId, {

        skip: !productId

    });
```

로 작성할 수 있습니다.

```text
productId 존재?
      │
   ┌──┴──┐
   ↓     ↓
  NO    YES
   ↓     ↓
 skip   Query
 true   실행
```

대표적인 상황은:

```text
Route Parameter가 아직 없음

User ID가 아직 없음

선택 항목이 아직 없음

특정 UI가 아직 열리지 않음
```

등입니다.

---

# 29. Lazy Query

일반 Query Hook은 Component가 데이터를 필요로 할 때 Query와 Subscription을 관리합니다.

```javascript
useGetProductQuery(id);
```

하지만 사용자의 행동이 발생했을 때만 조회하고 싶을 수도 있습니다.

예를 들어 검색 버튼을 눌렀을 때 검색한다고 해봅시다.

```javascript
const [
    triggerSearch,
    {
        data,
        isFetching
    }
] = useLazySearchProductsQuery();
```

원하는 시점에:

```javascript
triggerSearch(keyword);
```

를 호출합니다.

```text
useLazySearchProductsQuery()
          ↓
      Trigger 획득
          ↓
      아직 Query X


사용자 검색
     ↓
triggerSearch(keyword)
     ↓
Query 시작
     ↓
Server Request
```

---

# 30. Query vs Lazy Query vs Mutation

세 가지를 구분하면:

| 종류         | 의미                    | 대표 상황    |
| ---------- | --------------------- | -------- |
| Query      | Component가 데이터를 필요로 함 | 상품 목록    |
| Lazy Query | 원하는 시점에 조회            | 검색       |
| Mutation   | Server State 변경       | 생성·수정·삭제 |

Mental Model:

```text
Query
  ↓
데이터가 필요함
  ↓
자동 Query + Subscription


Lazy Query
  ↓
특정 시점에 조회
  ↓
trigger()


Mutation
  ↓
Server State 변경
  ↓
Mutation Trigger
```

---

# 31. `transformResponse()`

Server Response와 Component가 원하는 데이터 구조가 항상 같지는 않습니다.

Server가:

```javascript
{
    status: "SUCCESS",

    data: {
        id: 10,
        name: "Keyboard"
    }
}
```

를 반환한다고 해봅시다.

Component에서는 실제 `data`만 필요할 수 있습니다.

```javascript
getProduct: builder.query({

    query: id =>
        `/products/${id}`,

    transformResponse: response =>
        response.data

})
```

흐름은:

```text
HTTP Response
      ↓
transformResponse()
      ↓
Client가 사용할 Data
      ↓
RTK Query Cache
      ↓
Component
```

여기서 중요한 점은:

> **RTK Query Cache에는 `transformResponse()`가 반환한 값이 저장됩니다.**

따라서 Server Response 구조와 Client에서 사용할 Data Model을 분리하는 데 활용할 수 있습니다.

---

# 32. `isLoading`과 `isFetching`

두 값은 비슷해 보이지만 차이가 있습니다.

## `isLoading`

아직 사용할 Data가 없는 최초 요청 상태를 표현할 때 중요합니다.

```text
Cache Data 없음
      ↓
Request
      ↓
isLoading
```

따라서:

```text
Loading Screen
Skeleton UI
```

등을 보여줄 수 있습니다.

## `isFetching`

이미 Data가 있어도 새로운 Request가 진행 중일 수 있습니다.

```text
Existing Data
      +
New Request
      ↓
isFetching
```

따라서 기존 화면은 유지하면서:

```text
Refreshing...
```

같은 UI를 보여줄 수 있습니다.

---

# 33. `refetch()`

Query를 명시적으로 다시 요청할 수도 있습니다.

```javascript
const {
    data,
    refetch
} = useGetProductsQuery();
```

사용:

```jsx
<button onClick={refetch}>
    새로고침
</button>
```

이 방식은:

```text
Tag Invalidation
      ↓
Automatic Refetch
```

와 달리 Component가 명시적으로 Query를 다시 실행하는 방법입니다.

---

# CHAPTER 7. RTK Query는 Redux 위에서 어떻게 동작하는가?

지금까지는 RTK Query를 사용하는 입장에서 살펴봤습니다.

이제 한 단계 더 깊이 들어가겠습니다.

---

# 34. Redux Store에 RTK Query 등록

API Slice를 만들었다고 모든 설정이 끝나는 것은 아닙니다.

Redux Store에 Reducer와 Middleware를 등록해야 합니다.

```javascript
const store = configureStore({

    reducer: {

        [apiSlice.reducerPath]:
            apiSlice.reducer

    },

    middleware:
        getDefaultMiddleware =>

            getDefaultMiddleware()
                .concat(
                    apiSlice.middleware
                )

});
```

두 요소가 중요합니다.

```text
apiSlice.reducer

apiSlice.middleware
```

---

# 35. `reducerPath`

`reducerPath`는 RTK Query의 State가 Redux Store에서 저장될 위치를 결정합니다.

```javascript
reducerPath: "api"
```

개념적으로:

```text
Redux Store
│
├── auth
├── ui
└── api
     │
     ├── queries
     ├── mutations
     └── subscriptions
```

와 같은 구조를 생각할 수 있습니다.

---

# 36. `apiSlice.reducer`

RTK Query는 Query와 Mutation 관련 State를 Redux Store에서 관리합니다.

개념적으로:

```text
Redux Store
│
└── api
     │
     ├── Queries
     ├── Mutations
     ├── Cache-related State
     └── Subscription-related State
```

따라서 중요한 사실은:

> **RTK Query는 Redux와 별개로 동작하는 외부 Cache Library가 아닙니다.**

Redux Toolkit 위에서 동작하는 Server State 관리 시스템입니다.

---

# 37. `apiSlice.middleware`

RTK Query Middleware는 여러 Runtime 동작을 담당합니다.

```text
dispatch
    ↓
Redux Middleware Pipeline
    ↓
apiSlice.middleware
    ↓
RTK Query Runtime
```

여기에서:

```text
Request Lifecycle

Cache 관리

Subscription 관리

Invalidation

Polling

Refetch
```

등과 관련된 동작이 처리됩니다.

따라서 RTK Query가 자동으로 많은 일을 해주는 것처럼 보여도 내부에는 여전히 Redux의:

```text
Store
Reducer
Middleware
Action
dispatch
```

개념이 존재합니다.

---

# 38. `setupListeners()`

RTK Query는 Browser의 Focus나 Network 상태 변화와 Query Refetch를 연결할 수도 있습니다.

```javascript
setupListeners(store.dispatch);
```

대표적인 기능은:

```text
refetchOnFocus

refetchOnReconnect
```

입니다.

---

# 39. `refetchOnFocus`

사용자가 다른 Browser Tab으로 이동했다가 다시 돌아왔다고 생각해봅시다.

그동안 Server State가 변경되었을 수 있습니다.

```text
다른 Tab으로 이동
      ↓
   시간 경과
      ↓
현재 Tab으로 복귀
      ↓
Window Focus
      ↓
RTK Query
      ↓
필요한 Query Refetch
```

이를 통해 Cache와 Server State의 Synchronization을 강화할 수 있습니다.

---

# 40. `refetchOnReconnect`

Network가 끊겼다가 다시 연결되는 상황도 있습니다.

```text
Online
  ↓
Offline
  ↓
Network Reconnect
  ↓
RTK Query
  ↓
Query Refetch
```

이 역시 RTK Query가 단순 Fetching Library가 아니라 **Server State Synchronization System**이라는 점을 보여줍니다.

---

# CHAPTER 8. Query와 Mutation의 내부 실행 흐름

# 41. Query의 전체 실행 흐름

이제 다음 한 줄을 내부 관점에서 살펴봅시다.

```javascript
useGetProductQuery(10);
```

지금까지 배운 내용을 모두 연결하면:

```text
React Component
      ↓
Generated Query Hook
      ↓
Endpoint + Argument
      ↓
Cache Key 결정
      ↓
Cache Entry 확인
      ↓
┌────────────────────┐
│ 사용 가능한 Cache? │
└─────────┬──────────┘
          │
      ┌───┴───┐
      ↓       ↓
     YES      NO
      ↓       ↓
 Cache 사용   baseQuery
      │       ↓
      │   HTTP Request
      │       ↓
      │     Server
      │       ↓
      │    Response
      │       ↓
      │ transformResponse
      │       ↓
      └────→ Cache
              ↑
          Subscription
              ↑
           Component
```

처음에 단순하게:

```text
Hook → Request → Response
```

라고 배웠던 구조 안에 사실 이 많은 과정이 들어 있었던 것입니다.

---

# 42. Mutation의 전체 실행 흐름

Mutation은 다음과 같이 이해할 수 있습니다.

```text
User Event
    ↓
Mutation Trigger
    ↓
Endpoint
    ↓
baseQuery
    ↓
HTTP Request
    ↓
Server State 변경
    ↓
Response
    ↓
Mutation Success
    ↓
invalidatesTags
    ↓
관련 Query Cache Invalid
    ↓
Active Subscription 존재?
    ↓
Refetch
    ↓
새로운 Server State
    ↓
Cache Update
    ↓
Component Update
```

여기에 필요하면:

```text
onQueryStarted
queryFulfilled
```

같은 Lifecycle Logic도 추가할 수 있습니다.

이제부터는 이러한 고급 기능을 살펴봅니다.

---

# CHAPTER 9. 실전 — 애플리케이션 규모가 커졌을 때

# 43. `injectEndpoints()`

작은 애플리케이션에서는 하나의 파일에 Endpoint를 모두 정의해도 됩니다.

```javascript
createApi({

    endpoints: builder => ({

        getProducts: ...,
        createProduct: ...,
        getOrders: ...,
        createOrder: ...,
        searchProducts: ...

    })

});
```

하지만 Endpoint가 많아지면 파일이 지나치게 커집니다.

그렇다고 기능마다 무조건:

```text
productApi = createApi()

orderApi = createApi()

searchApi = createApi()
```

를 만드는 것이 최선은 아닙니다.

하나의 Base API Slice를 만들 수 있습니다.

```javascript
export const apiSlice = createApi({

    reducerPath: "api",

    baseQuery: fetchBaseQuery({
        baseUrl: "/api"
    }),

    tagTypes: [
        "Product",
        "Order",
        "User"
    ],

    endpoints: () => ({})

});
```

그리고 기능별 파일에서 Endpoint를 주입합니다.

```javascript
export const productApi =
    apiSlice.injectEndpoints({

        endpoints: builder => ({

            getProducts:
                builder.query({

                    query: () =>
                        "/products"

                })

        })

    });
```

---

# 44. `injectEndpoints()`의 Mental Model

`injectEndpoints()`는 새로운 RTK Query Store를 만드는 것이 아닙니다.

```text
                 createApi()
                     ↓
                  apiSlice
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
    Products       Orders       Search
        ↓            ↓            ↓
 injectEndpoints injectEndpoints injectEndpoints
        │            │            │
        └────────────┼────────────┘
                     ↓
              하나의 API Slice
                     ↓
              하나의 Cache System
```

즉:

> **코드는 기능별로 분리하면서 RTK Query Cache와 Middleware는 하나의 API Slice를 중심으로 공유할 수 있습니다.**

---

# CHAPTER 10. 실전 — Request 공통 처리

# 45. Custom `baseQuery`는 왜 필요한가?

기본적인 경우:

```javascript
fetchBaseQuery({
    baseUrl: "/api"
})
```

만으로 충분합니다.

하지만 실제 애플리케이션에서는 모든 Request에 공통 처리가 필요할 수 있습니다.

```text
Authorization Header

Cookie

CSRF Token

공통 Error Handling

401 처리

Token Refresh

Logging
```

이때 `fetchBaseQuery()`를 감싸 **Custom baseQuery**를 만들 수 있습니다.

```text
Endpoint
    ↓
Custom baseQuery
    ↓
fetchBaseQuery
    ↓
HTTP Request
    ↓
Server
    ↓
Response
    ↓
Custom Response Handling
```

---

# 46. `prepareHeaders()`

Request를 보내기 전에 공통 Header를 설정할 수 있습니다.

```javascript
const rawBaseQuery = fetchBaseQuery({

    baseUrl: "/api",

    prepareHeaders: headers => {

        const token =
            localStorage.getItem("token");

        if (token) {

            headers.set(
                "Authorization",
                `Bearer ${token}`
            );

        }

        return headers;
    }

});
```

흐름:

```text
Endpoint
    ↓
fetchBaseQuery
    ↓
prepareHeaders()
    ↓
Header 추가
    ↓
HTTP Request
```

입니다.

---

# 47. `credentials`

Cookie 기반 인증을 사용하는 경우 `fetch`의 `credentials` 옵션을 설정할 수도 있습니다.

```javascript
fetchBaseQuery({

    baseUrl: "/api",

    credentials: "include"

})
```

개념적으로:

```text
Browser
   │
   │ Cookie
   ↓
HTTP Request
   ↓
Server
```

입니다.

이것은 RTK Query 고유의 인증 방식이 아니라 내부적으로 사용하는 `fetch`의 Request 정책과 연결된 설정입니다.

---

# 48. Custom `baseQuery`와 공통 Error Handling

모든 Request에서 `401 Unauthorized`를 공통 처리한다고 해봅시다.

```javascript
const rawBaseQuery =
    fetchBaseQuery({
        baseUrl: "/api"
    });

const baseQuery =
    async (args, api, extraOptions) => {

        const result =
            await rawBaseQuery(
                args,
                api,
                extraOptions
            );

        if (result.error?.status === 401) {

            api.dispatch(
                logout()
            );

        }

        return result;
    };
```

흐름은:

```text
Endpoint
    ↓
Custom baseQuery
    ↓
rawBaseQuery
    ↓
HTTP Request
    ↓
Server
    ↓
Response
    ↓
   401 ?
 ┌───┴───┐
 ↓       ↓
YES      NO
 ↓        ↓
logout   result
dispatch return
```

입니다.

Custom Base Query의 `api`를 통해:

```javascript
api.dispatch(...)

api.getState()
```

등에 접근할 수 있습니다.

따라서 RTK Query의 Request Lifecycle과 일반 Redux Logic을 연결할 수 있습니다.

---

# CHAPTER 11. 실전 — Query Lifecycle

# 49. `onQueryStarted()`

RTK Query는 Query 또는 Mutation이 시작될 때 추가적인 Lifecycle Logic을 실행할 수 있습니다.

```javascript
updateProduct: builder.mutation({

    query: product => ({

        url: `/products/${product.id}`,

        method: "PUT",

        body: product

    }),

    async onQueryStarted(
        arg,
        {
            dispatch,
            queryFulfilled
        }
    ) {

        // Request Lifecycle Logic

    }

})
```

Mental Model:

```text
Endpoint 시작
     ↓
onQueryStarted()
     ↓
Request Lifecycle Logic
```

입니다.

---

# 50. `queryFulfilled`

`onQueryStarted()`에서 사용할 수 있는 `queryFulfilled`는 해당 Query 또는 Mutation의 요청 결과와 연결된 Promise입니다.

```text
Endpoint 시작
      ↓
onQueryStarted()
      ↓
queryFulfilled
      ↓
    Promise
   ↙      ↘
Success  Failure
```

따라서:

```javascript
try {

    const { data } =
        await queryFulfilled;

    // 성공 이후 처리

} catch (error) {

    // 실패 처리

}
```

와 같은 Workflow를 만들 수 있습니다.

---

# CHAPTER 12. 실전 — Hook 없이 RTK Query 사용하기

# 51. `endpoint.initiate()`

React Generated Hook을 사용하지 않고 Redux `dispatch()`를 통해 Endpoint를 직접 시작할 수도 있습니다.

```javascript
dispatch(

    apiSlice.endpoints
        .getCurrentUser
        .initiate()

);
```

즉 Endpoint를 시작하는 방법은:

```text
React Component
      ↓
Generated Hook


또는


Redux Logic
      ↓
dispatch(
  endpoint.initiate()
)
      ↓
RTK Query
```

가 될 수 있습니다.

일반적인 React Component에서는 Generated Hook이 훨씬 편리합니다.

하지만 특정 Lifecycle이나 Redux Logic에서 Query를 직접 시작해야 하는 경우 `initiate()`를 사용할 수 있습니다.

---

# 52. `apiSlice.util`

RTK Query는 Cache를 Programmatic하게 조작할 수 있는 Utility API도 제공합니다.

대표적으로:

```javascript
apiSlice.util.invalidateTags(...)
```

와:

```javascript
apiSlice.util.upsertQueryData(...)
```

등이 있습니다.

---

# 53. `invalidateTags()`

특정 Tag를 코드에서 직접 무효화할 수 있습니다.

```javascript
dispatch(

    apiSlice.util.invalidateTags([

        {
            type: "Product",
            id: "LIST"
        }

    ])

);
```

흐름:

```text
Redux Logic
     ↓
invalidateTags()
     ↓
RTK Query
     ↓
관련 Cache Invalid
     ↓
필요한 Query Refetch
```

일반적인 Mutation에서는 `invalidatesTags`를 선언적으로 사용하는 것이 자연스럽습니다.

하지만 Mutation 외부의 Logic에서 Cache를 무효화해야 할 때 사용할 수 있습니다.

---

# 54. `upsertQueryData()`

특정 Query Cache Entry에 데이터를 직접 넣거나 업데이트할 수도 있습니다.

```javascript
dispatch(

    apiSlice.util.upsertQueryData(

        "getCurrentUser",

        undefined,

        user

    )

);
```

개념적으로:

```text
Redux Logic
     ↓
upsertQueryData()
     ↓
Query Cache Entry
     ↓
Data Update
```

입니다.

즉 반드시 새로운 Server Request를 수행해야만 Cache를 변경할 수 있는 것은 아닙니다.

---

# CHAPTER 13. 지금까지 배운 Redux 도구들의 역할

# 55. `useState`, `createSlice`, `createAsyncThunk`, RTK Query

지금까지 여러 State 관리 도구를 배웠습니다.

각각의 역할을 구분해봅시다.

```text
Local UI State
      ↓
useState()
```

예:

```text
Modal
Input
Toggle
```

---

```text
Shared Client State
      ↓
createSlice()
```

예:

```text
인증 상태
Theme
전역 UI State
```

---

```text
일반적인 비동기 Redux Workflow
      ↓
createAsyncThunk()
```

---

```text
Server State

Fetching
+
Caching
+
Synchronization

      ↓

RTK Query
```

하지만 실제 애플리케이션에서는 이들이 완전히 독립적인 것은 아닙니다.

예를 들어:

```text
RTK Query
    ↓
onQueryStarted
    ↓
dispatch()
    ↓
createSlice State 변경
```

처럼 서로 연결할 수도 있습니다.

---

# 56. RTK Query가 Redux를 없애는 것은 아니다

RTK Query 내부에도 여전히 Redux의 핵심 개념이 존재합니다.

```text
Redux Store

Reducer

Middleware

Action

dispatch
```

따라서 다음 학습 순서는 의미가 있습니다.

```text
Redux Fundamentals
       ↓
Redux Toolkit
       ↓
Middleware / Thunk
       ↓
createAsyncThunk
       ↓
RTK Query
```

RTK Query는 Redux를 대체하는 별개의 기술이 아닙니다.

> **Redux Toolkit 위에서 Server State의 Fetching, Caching, Synchronization 문제를 해결하도록 설계된 도구입니다.**

---

# 57. 최종 Mental Model

이제 RTK Query 전체를 하나의 그림으로 연결해봅시다.

```text
                         Server
                           ↕
                       baseQuery
                           ↕
                        Endpoint
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
            Query                    Mutation
              │                         │
              ↓                         ↓
         Query Cache              Server 변경
              │                         │
              │                  invalidatesTags
              │                         │
              │                   Cache Invalid
              │                         │
              │                      Refetch
              │                         │
              └────────────┬────────────┘
                           ↓
                    Query Cache Entry
                           ↑
                      Subscription
                           ↑
                    React Component
```

이 그림에서 가장 먼저 볼 것은 세 가지입니다.

```text
Query
   ↓
Cache
   ↓
Component
```

그리고 Mutation이 발생하면:

```text
Mutation
   ↓
Server State 변경
   ↓
Invalidation
   ↓
Refetch
   ↓
Cache Update
   ↓
Component Update
```

가 연결됩니다.

---

# 58. 애플리케이션이 커지면

규모가 커지면:

```text
                       createApi()
                           ↓
                        apiSlice
                           ↓
                    injectEndpoints()
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Product           Order           Search
      Endpoints        Endpoints        Endpoints
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                    하나의 API Slice
                           ↓
                    하나의 Cache System
                           ↓
                      Redux Store
```

형태로 확장할 수 있습니다.

---

# 59. RTK Query를 한 문장으로 정의한다면

RTK Query를 단순히:

```text
API 호출을 쉽게 해주는 Library
```

라고 기억하면 핵심을 놓치게 됩니다.

조금 더 정확하게 정의하면:

> **RTK Query는 Redux Toolkit 위에서 Server State를 가져오고(Fetching), Query 결과를 저장하고 공유하며(Caching), Server의 변경과 Client Cache를 다시 맞추는(Synchronization) 작업을 관리하는 Server State 관리 시스템입니다.**

따라서 RTK Query의 핵심은:

```text
Server
   ↕
Fetching
   ↕
Query Cache
   ↕
Subscription
   ↕
React Component
```

그리고 Server State가 변경되었을 때:

```text
Mutation
   ↓
Invalidation
   ↓
Refetch
   ↓
Cache Update
   ↓
Component Update
```

라는 구조에 있습니다.

---

# 60. 처음 공부할 때 반드시 기억해야 할 것

이 문서에 등장한 모든 API를 한 번에 암기할 필요는 없습니다.

## 1단계 — 반드시 이해

```text
RTK Query가 왜 필요한가?

Query

Mutation

Generated Hook

Server State

Query Cache
```

## 2단계 — 핵심 원리

```text
Cache Key

Subscription

Request Deduplication

Cache Lifetime

Tag

Invalidation

Refetch
```

## 3단계 — 내부 구조

```text
apiSlice.reducer

apiSlice.middleware

setupListeners

Query 전체 실행 흐름

Mutation 전체 실행 흐름
```

## 4단계 — 실전에서 필요할 때

```text
skip

Lazy Query

transformResponse

injectEndpoints

Custom baseQuery

prepareHeaders

onQueryStarted

queryFulfilled

endpoint.initiate

apiSlice.util
```

따라서 처음 학습할 때 가장 중요한 흐름은 이것입니다.

```text
왜 RTK Query가 필요한가?
          ↓
       Query
          ↓
        Cache
          ↓
     Component
          ↓
      Mutation
          ↓
Server State 변경
          ↓
   Cache가 오래됨
          ↓
    Invalidation
          ↓
       Refetch
          ↓
     Cache Update
```

이 흐름이 머릿속에 잡히면 RTK Query의 나머지 기능들은 더 이상 서로 떨어져 있는 API들의 모음이 아니라, **Server State를 관리하기 위한 하나의 시스템**으로 보이기 시작합니다.
