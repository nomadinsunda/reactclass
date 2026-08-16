# PART 5. RTK Query

PART 3에서는 Redux에서 비동기 작업을 처리하기 위해 다음 구조를 학습했습니다.

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

`createAsyncThunk()`는 일반적인 비동기 Redux 로직을 작성할 때 매우 유용합니다.

하지만 실제 웹 애플리케이션에서는 서버에서 가져오는 데이터가 많아집니다.

```text
GET    /products
GET    /products/10
POST   /products
PUT    /products/10
DELETE /products/10

GET    /members
GET    /orders
GET    /categories
```

이러한 Server State를 모두 `createAsyncThunk()`로 관리하면 다음 작업을 반복해서 작성해야 합니다.

```text
API Request
loading
error
data
pending
fulfilled
rejected
cache
refetch
중복 요청 관리
데이터 무효화
```

Redux Toolkit은 이러한 **Data Fetching과 Caching 문제를 해결하기 위한 RTK Query**를 제공합니다. RTK Query는 Redux Toolkit에 포함된 데이터 fetching/caching 도구이며, 일반적인 서버 데이터 로딩 로직을 직접 작성해야 하는 양을 크게 줄이는 것을 목적으로 합니다. ([리덕스 툴킷][1])

---

# 1. Client State와 Server State

RTK Query를 이해하려면 먼저 Client State와 Server State를 구분해야 합니다.

애플리케이션의 State를 크게 다음과 같이 생각할 수 있습니다.

```text
Application State
│
├── Client State
│
└── Server State
```

## Client State

Client State는 브라우저 애플리케이션 자체에서 만들어지고 관리되는 상태입니다.

예:

```text
Sidebar 열림 여부

Modal 표시 여부

현재 선택한 Tab

검색 조건

Wizard 단계

UI Theme
```

이런 상태는 일반적으로 `createSlice()`로 관리하기 적합합니다.

```text
Client State
     ↓
createSlice()
```

---

# 2. Server State

Server State는 원본 데이터가 서버에 존재합니다.

예:

```text
상품

회원

게시글

주문

댓글

재고
```

React 애플리케이션이 가지고 있는 것은 서버 데이터의 클라이언트 측 복사본이라고 볼 수 있습니다.

```text
Database
    ↓
Server
    ↓
HTTP Response
    ↓
Browser
    ↓
Cached Server Data
```

따라서 Server State에는 Client State와 다른 문제가 존재합니다.

```text
언제 요청할 것인가?

같은 데이터를 다시 요청해야 하는가?

이미 받은 데이터를 재사용할 수 있는가?

서버 데이터가 변경되면 Cache는 어떻게 할 것인가?

Component가 없어지면 Cache를 언제 제거할 것인가?

여러 Component가 동일한 데이터를 요청하면 어떻게 할 것인가?
```

이 문제들을 RTK Query가 관리합니다.

---

# 3. `createAsyncThunk()` 방식의 Server State 관리

상품 목록을 가져온다고 가정해봅시다.

```javascript
export const fetchProducts =
    createAsyncThunk(
        "products/fetchProducts",

        async () => {

            const response =
                await fetch("/api/products");

            return response.json();
        }
    );
```

그리고 Slice:

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
                state => {
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

동작에는 문제가 없습니다.

하지만 API가 30개, 50개로 증가한다면 반복 코드가 많아집니다.

---

# 4. RTK Query가 해결하려는 문제

RTK Query에서는 Server State 관리와 관련된 여러 기능을 하나의 시스템으로 제공합니다. 공식 문서는 RTK Query를 데이터 fetching과 caching을 단순화하기 위한 도구로 설명합니다. ([리덕스 툴킷][1])

개념적으로:

```text
RTK Query
│
├── API Request
├── Response 처리
├── Loading State
├── Error State
├── Cache
├── Cache Key
├── Subscription
├── Request Deduplication
├── Refetch
└── Cache Invalidation
```

를 담당합니다.

따라서 Component 입장에서는:

```text
데이터 필요
    ↓
Generated Hook 호출
    ↓
RTK Query
    ↓
필요한 데이터 반환
```

이라는 훨씬 단순한 구조를 사용할 수 있습니다.

---

# 5. RTK Query의 핵심 구조

가장 먼저 전체 구조를 보겠습니다.

```text
React Component
       ↓
Generated Hook
       ↓
┌──────────────────────┐
│      RTK Query       │
│                      │
│ Endpoint             │
│ Request              │
│ Cache                │
│ Subscription         │
│ Invalidation         │
└──────────┬───────────┘
           ↓
       REST API
           ↓
      Spring Boot
```

RTK Query에서 핵심적으로 학습해야 하는 것은 다음입니다.

```text
createApi()

fetchBaseQuery()

endpoints

builder.query()

builder.mutation()

Generated Hooks

Cache

Subscription

Tags

Invalidation
```

---

# 6. `createApi()`

RTK Query의 중심 API는 `createApi()`입니다.

`createApi()`는 Backend API에서 데이터를 어떻게 가져오고 변경할 것인지 정의하는 endpoint 집합을 구성하고, fetching/caching에 필요한 Redux 로직과 React 사용 시 Hook까지 생성하는 핵심 API입니다. ([리덕스 툴킷][2])

기본적인 형태는 다음과 같습니다.

```javascript
import {
    createApi,
    fetchBaseQuery
} from "@reduxjs/toolkit/query/react";


export const productApi = createApi({

    reducerPath: "productApi",

    baseQuery: fetchBaseQuery({
        baseUrl: "/api"
    }),

    endpoints: builder => ({

        // endpoints

    })

});
```

구조를 보면:

```text
createApi()
│
├── reducerPath
├── baseQuery
└── endpoints
```

입니다.

---

# 7. 왜 `/query/react`에서 import 하는가?

React 애플리케이션에서 Generated Hook 기능까지 사용하려면 일반적으로 다음 경로에서 import합니다.

```javascript
import {
    createApi,
    fetchBaseQuery
} from "@reduxjs/toolkit/query/react";
```

이를 통해 endpoint를 기반으로 React Hook을 생성할 수 있습니다.

예를 들어:

```text
getProducts
       ↓
useGetProductsQuery()
```

가 만들어집니다.

---

# 8. `reducerPath`

```javascript
reducerPath: "productApi"
```

는 RTK Query가 Redux Store 내부에서 자신의 상태를 저장할 위치의 key를 정의합니다.

개념적으로 Redux State 내부에는:

```javascript
{
    productApi: {
        queries: {
            // ...
        },
        mutations: {
            // ...
        }
    }
}
```

와 같은 RTK Query 관리 영역이 만들어집니다.

따라서 RTK Query 역시 Redux Store를 사용합니다.

중요합니다.

```text
RTK Query

Redux와 별개의 상태 관리 시스템
        X

Redux Toolkit 위에 구성된
Server State 관리 시스템
        O
```

---

# 9. `fetchBaseQuery()`

`fetchBaseQuery()`는 RTK Query에서 HTTP 요청을 쉽게 수행할 수 있도록 제공되는 작은 fetch 기반 wrapper입니다. 공식 API에 따르면 `baseUrl`, `prepareHeaders`, 표준 `RequestInit` 옵션 등을 사용할 수 있습니다. ([리덕스 툴킷][3])

예:

```javascript
baseQuery: fetchBaseQuery({

    baseUrl:
        "http://localhost:8080/api"

})
```

이후 endpoint에서:

```javascript
query: () => "/products"
```

라고 작성하면 최종 요청 URL은:

```text
http://localhost:8080/api/products
```

가 됩니다.

즉:

```text
baseUrl

http://localhost:8080/api

        +

endpoint

/products

        ↓

http://localhost:8080/api/products
```

입니다.

---

# 10. Endpoint란?

Endpoint는 애플리케이션이 서버와 수행할 하나의 API 작업을 정의합니다.

예를 들어 Spring Boot에 다음 REST API가 있다고 생각해봅시다.

```text
GET    /api/products
GET    /api/products/{id}

POST   /api/products

PUT    /api/products/{id}

DELETE /api/products/{id}
```

RTK Query에서는 각각을 endpoint로 정의할 수 있습니다.

```text
getProducts

getProduct

createProduct

updateProduct

deleteProduct
```

즉:

```text
REST API Operation
       ↓
RTK Query Endpoint
```

이라고 볼 수 있습니다.

---

# 11. `endpoints`

Endpoint는 다음처럼 작성합니다.

```javascript
endpoints: builder => ({

    getProducts: builder.query({

        query: () => "/products"

    })

})
```

여기서:

```text
builder
   ↓
query()
mutation()
```

두 가지가 매우 중요합니다.

---

# 12. `builder.query()`

`query`는 주로 **Server State를 조회하여 Cache할 때** 사용합니다. 공식 RTK Query 문서도 query를 서버에서 데이터를 가져와 client cache에 저장하는 작업으로 설명하며, 데이터 변경이 목적이라면 mutation을 사용하도록 안내합니다. ([리덕스 툴킷][4])

예:

```javascript
getProducts: builder.query({

    query: () => "/products"

})
```

이 endpoint는:

```text
GET /products
```

를 수행합니다.

---

# 13. Product 목록 Query

전체 코드는:

```javascript
export const productApi = createApi({

    reducerPath: "productApi",

    baseQuery: fetchBaseQuery({
        baseUrl:
            "http://localhost:8080/api"
    }),

    endpoints: builder => ({

        getProducts:
            builder.query({

                query: () =>
                    "/products"

            })

    })

});
```

이제 RTK Query가 React Hook을 자동으로 만들어줍니다.

```javascript
useGetProductsQuery
```

---

# 14. Generated Hook

다음 endpoint 이름:

```javascript
getProducts
```

를 기반으로:

```javascript
useGetProductsQuery()
```

Hook이 만들어집니다.

Export:

```javascript
export const {
    useGetProductsQuery
} = productApi;
```

Component에서는:

```jsx
function ProductList() {

    const {
        data,
        isLoading,
        error
    } = useGetProductsQuery();

}
```

와 같이 사용합니다.

React용 query hook은 fetch를 트리거하고, Component를 해당 cache data에 subscribe시키며, 요청 상태와 cached data를 읽도록 합니다. ([리덕스 툴킷][5])

---

# 15. `createAsyncThunk()`와 비교

PART 3에서는 Component가:

```javascript
dispatch(fetchProducts());
```

를 직접 실행했습니다.

그리고:

```javascript
const {
    items,
    loading,
    error
} = useSelector(
    state => state.products
);
```

가 필요했습니다.

RTK Query에서는:

```javascript
const {
    data,
    isLoading,
    error
} =
    useGetProductsQuery();
```

로 상당 부분 통합됩니다.

비교하면:

```text
createAsyncThunk

useEffect()
   ↓
dispatch()
   ↓
Thunk
   ↓
pending / fulfilled / rejected
   ↓
Reducer
   ↓
useSelector()


RTK Query

useGetProductsQuery()
   ↓
RTK Query
   ↓
Request + Cache + State 관리
```

입니다.

---

# 16. `useEffect()`가 필요하지 않다

일반적인 RTK Query Hook 사용에서는:

```javascript
useEffect(() => {

    dispatch(fetchProducts());

}, []);
```

처럼 요청을 직접 시작할 필요가 없습니다.

```javascript
const {
    data,
    isLoading,
    error
} =
    useGetProductsQuery();
```

Hook 사용 자체가 필요한 query 수행 및 subscription과 연결됩니다. ([리덕스 툴킷][5])

---

# 17. ProductList Component

```jsx
import {
    useGetProductsQuery
} from "./productApi";


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


export default ProductList;
```

Component에서 직접 관리하던:

```text
fetch

Promise

loading State

error State

Redux dispatch
```

가 크게 줄어들었습니다.

---

# 18. Query Argument

특정 상품을 조회하고 싶다고 합시다.

```text
GET /products/10
```

endpoint:

```javascript
getProduct:
    builder.query({

        query: id =>
            `/products/${id}`

    })
```

Component:

```javascript
const {
    data,
    isLoading,
    error
} =
    useGetProductQuery(10);
```

여기서:

```text
10
```

이 Query Argument입니다.

---

# 19. Query Argument와 Cache Key

Query Argument는 단순히 URL 생성에만 사용되는 것이 아닙니다.

RTK Query는 endpoint와 query argument를 기준으로 cache entry를 구분합니다. React query hooks에서도 query argument가 cache key 결정에 사용됩니다. ([리덕스 툴킷][5])

예:

```javascript
useGetProductQuery(10);
```

개념적으로:

```text
Endpoint

getProduct

    +

Argument

10

    ↓

Query Cache Entry
```

다른 요청:

```javascript
useGetProductQuery(20);
```

은 별도의 Cache Entry가 됩니다.

---

# 20. Cache

RTK Query에서 Query의 중요한 목적은 데이터를 가져오는 것뿐 아니라 **그 결과를 Cache하는 것**입니다. ([리덕스 툴킷][4])

예:

```text
GET /products/10
       ↓
Server Response
       ↓
RTK Query Cache
       ↓
Component
```

한 번 데이터를 받았다면 상황에 따라 기존 Cache를 재사용할 수 있습니다.

---

# 21. 같은 Query를 여러 Component가 사용한다면?

다음 두 Component가 있다고 가정해봅시다.

```text
ProductHeader

ProductDetail
```

둘 다:

```javascript
useGetProductQuery(10);
```

을 호출합니다.

개념적으로:

```text
ProductHeader ──────┐
                    │
                    ↓
             getProduct(10)
                  Cache
                    ↑
                    │
ProductDetail ──────┘
```

RTK Query는 동일한 endpoint + argument에 대응하는 cache entry와 subscription을 관리하기 때문에 각 Component가 항상 완전히 독립된 서버 요청을 직접 관리하는 구조가 아닙니다. ([리덕스 툴킷][6])

---

# 22. Request Deduplication

예를 들어:

```javascript
useGetProductQuery(10);
```

을 동일한 조건으로 사용하는 여러 Component가 있을 수 있습니다.

RTK Query에서는 같은 cache entry를 공유할 수 있기 때문에 동일한 데이터를 사용하는 Component마다 직접 별도의 fetching 로직과 state를 만들 필요가 없습니다.

개념적으로:

```text
Component A ─┐
             │
Component B ─┼──→ getProduct(10)
             │          ↓
Component C ─┘      Cache Entry
```

이 구조가 Server State 공유를 훨씬 쉽게 만듭니다. ([리덕스 툴킷][6])

---

# 23. Subscription

RTK Query를 이해할 때 Cache와 함께 반드시 알아야 하는 개념이 **Subscription**입니다.

React query hook을 사용하는 Component는 해당 query cache entry를 사용하도록 subscription을 형성합니다. 공식 문서에서도 query hook이 fetch와 cache subscription을 함께 처리한다고 설명합니다. ([리덕스 툴킷][7])

예:

```text
Component A
     │
     │ subscribe
     ↓

getProduct(10)
Cache Entry
```

Component B도 같은 Query를 사용하면:

```text
Component A ───┐
               │
               ↓
         getProduct(10)
            Cache
               ↑
               │
Component B ───┘
```

가 됩니다.

---

# 24. Component가 Unmount되면?

Component가 사라지면 해당 Component의 subscription도 사라집니다.

```text
Component
    ↓
Unmount
    ↓
Subscription 제거
```

하지만 subscription이 없어졌다고 Cache가 반드시 즉시 삭제되는 것은 아닙니다.

RTK Query는 사용되지 않는 cache data를 일정 기간 보관할 수 있으며 `keepUnusedDataFor`를 통해 이 시간을 설정할 수 있습니다. ([리덕스 툴킷][6])

개념적으로:

```text
Last Subscriber 제거
        ↓
Cache 즉시 삭제 X
        ↓
일정 시간 유지
        ↓
다시 사용되지 않으면 제거
```

이렇게 하면 화면을 잠깐 이동했다가 다시 돌아왔을 때 기존 데이터를 재활용할 수 있습니다.

---

# 25. Query와 Mutation

RTK Query의 Endpoint는 크게 두 종류로 생각하면 됩니다.

```text
Endpoint
│
├── Query
│
└── Mutation
```

## Query

주로 데이터를 조회합니다.

```text
GET
```

예:

```javascript
getProducts:
    builder.query(...)
```

## Mutation

서버 데이터를 변경하는 작업에 사용합니다.

```text
POST
PUT
PATCH
DELETE
```

예:

```javascript
createProduct:
    builder.mutation(...)
```

RTK Query 공식 문서도 데이터를 가져오는 작업에는 query를, 서버 데이터를 변경하거나 cache invalidation과 연결되는 작업에는 mutation을 사용하도록 설명합니다. ([리덕스 툴킷][4])

---

# 26. Product 생성 Mutation

```javascript
createProduct:
    builder.mutation({

        query: product => ({

            url: "/products",

            method: "POST",

            body: product

        })

    })
```

전체 API:

```javascript
export const productApi = createApi({

    reducerPath: "productApi",

    baseQuery: fetchBaseQuery({
        baseUrl: "/api"
    }),

    endpoints: builder => ({

        getProducts:
            builder.query({

                query: () =>
                    "/products"

            }),

        createProduct:
            builder.mutation({

                query: product => ({

                    url: "/products",

                    method: "POST",

                    body: product

                })

            })

    })

});
```

---

# 27. Mutation Hook

`createProduct` endpoint에서는 다음 Hook이 생성됩니다.

```javascript
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
] =
    useCreateProductMutation();
```

실행:

```javascript
await createProduct({
    name: "Keyboard",
    price: 50000
});
```

---

# 28. Query Hook과 Mutation Hook 사용 방식 차이

Query:

```javascript
const {
    data,
    isLoading
} =
    useGetProductsQuery();
```

Query는 Component가 데이터를 필요로 하는 동안 자동적인 fetching/subscription 흐름과 연결됩니다.

Mutation:

```javascript
const [
    createProduct,
    result
] =
    useCreateProductMutation();
```

Mutation은 일반적으로 사용자가 특정 작업을 수행했을 때 Trigger Function을 호출합니다.

예:

```jsx
<button
    onClick={() =>
        createProduct(product)
    }
>
    등록
</button>
```

---

# 29. 문제: POST 이후 기존 GET Cache는 어떻게 되는가?

현재 Product 목록이 다음과 같다고 합시다.

```text
RTK Query Cache

GET /products

[
    Product A,
    Product B
]
```

이제:

```text
POST /products
```

로 Product C를 추가했습니다.

Server:

```text
Product A
Product B
Product C
```

그런데 기존 Cache에는:

```text
Product A
Product B
```

만 있을 수 있습니다.

즉:

```text
Server State
       ≠
Cached Server State
```

가 됩니다.

이 문제를 해결하기 위해 **Cache Invalidation**이 필요합니다.

---

# 30. Tag

RTK Query에서는 Cache 사이의 관계를 표현하기 위해 Tag를 사용할 수 있습니다.

먼저 API에 tag type을 정의합니다.

```javascript
export const productApi = createApi({

    reducerPath: "productApi",

    baseQuery: fetchBaseQuery({
        baseUrl: "/api"
    }),

    tagTypes: [
        "Product"
    ],

    endpoints: builder => ({
        // ...
    })

});
```

---

# 31. `providesTags`

상품 목록 Query가 Product 데이터를 제공한다고 표시할 수 있습니다.

```javascript
getProducts:
    builder.query({

        query: () =>
            "/products",

        providesTags: [
            "Product"
        ]

    })
```

개념적으로:

```text
GET /products
      ↓
Cache Entry
      ↓
Tag
      ↓
Product
```

입니다.

`providesTags`는 해당 query result가 어떤 cache tag와 연결되는지를 표현합니다. ([리덕스 툴킷][8])

---

# 32. `invalidatesTags`

Product를 생성하는 Mutation:

```javascript
createProduct:
    builder.mutation({

        query: product => ({

            url: "/products",

            method: "POST",

            body: product

        }),

        invalidatesTags: [
            "Product"
        ]

    })
```

의미:

```text
Product 생성 성공
       ↓
Product Tag 무효화
```

입니다.

---

# 33. Cache Invalidation

전체 흐름을 보면:

```text
GET /products
      ↓
Cache 생성
      ↓
providesTags: Product


POST /products
      ↓
Mutation 성공
      ↓
invalidatesTags: Product
      ↓
Product 관련 Cache Invalid
```

RTK Query는 mutation이 특정 tag를 invalidate하도록 정의할 수 있으며, 해당 tag를 제공하는 query cache가 active subscription을 가지고 있다면 다시 fetch하고, 사용 중이 아니라면 cache data 제거 대상으로 처리합니다. ([리덕스 툴킷][8])

---

# 34. Automatic Refetch

가장 중요한 흐름입니다.

```text
ProductList Component
       ↓
useGetProductsQuery()
       ↓
GET /products
       ↓
Cache
       ↓
providesTags: Product


사용자가 상품 등록
       ↓
createProduct()
       ↓
POST /products
       ↓
Success
       ↓
invalidatesTags: Product
       ↓
Product Cache Invalid
       ↓
Active Subscription 존재
       ↓
GET /products Refetch
       ↓
새로운 Server Data
       ↓
Cache Update
       ↓
Component Update
```

이것이 RTK Query의 가장 강력한 기능 중 하나입니다. ([리덕스 툴킷][8])

---

# 35. RTK Query의 핵심은 단순 fetch가 아니다

RTK Query를:

> `fetch()`를 편하게 호출하는 라이브러리

라고 이해하면 핵심을 놓치게 됩니다.

더 중요한 역할은:

```text
Server State
     ↕
Client Cache
```

를 관리하는 것입니다.

즉:

```text
Fetch
+
Cache
+
Subscription
+
Invalidation
+
Refetch
```

이 전체 시스템을 제공한다는 것이 중요합니다. ([리덕스 툴킷][1])

---

# 36. 세밀한 Tag 관리

상품 하나를 수정했다고 모든 Product Query를 무조건 무효화할 필요는 없을 수 있습니다.

Tag에 ID를 사용할 수도 있습니다.

개념적으로:

```javascript
providesTags: (result, error, id) => [
    {
        type: "Product",
        id
    }
]
```

예:

```text
Product:10
Product:20
Product:30
```

그러면 Product 10을 수정했을 때:

```text
Product:10
```

과 관련된 cache를 선택적으로 invalidate하는 구조를 만들 수 있습니다.

RTK Query는 tag type과 optional ID를 이용한 세밀한 invalidation 패턴도 지원합니다. ([리덕스 툴킷][8])

---

# 37. `isLoading`과 `isFetching`

Query 결과에서는 요청 상태를 나타내는 여러 값을 사용할 수 있습니다.

특히 개념적으로 구분할 필요가 있는 것이:

```text
isLoading

isFetching
```

입니다.

처음 데이터를 가져오는 상황:

```text
Cache Data 없음
     ↓
Request
     ↓
isLoading
```

이미 데이터가 존재하지만 다시 요청하는 상황에서는 기존 데이터를 표시하면서 새로운 request가 진행될 수 있습니다.

이런 상태를 구분하면 Skeleton UI와 background refetch UI를 다르게 표현할 수 있습니다.

---

# 38. Manual Refetch

필요하다면 Query Hook에서 `refetch`를 사용할 수도 있습니다.

```javascript
const {
    data,
    refetch
} =
    useGetProductsQuery();
```

사용:

```jsx
<button onClick={refetch}>
    새로고침
</button>
```

자동 invalidation 외에도 명시적으로 데이터를 다시 요청해야 하는 상황에서 사용할 수 있습니다.

---

# 39. Store에 RTK Query 등록

`createApi()`만 만들었다고 RTK Query가 완성되는 것은 아닙니다.

API Slice가 생성한 Reducer와 Middleware를 Redux Store에 등록해야 합니다.

```javascript
import {
    configureStore
} from "@reduxjs/toolkit";

import {
    productApi
} from "../features/products/productApi";


export const store =
    configureStore({

        reducer: {

            [productApi.reducerPath]:
                productApi.reducer

        },

        middleware:
            getDefaultMiddleware =>

                getDefaultMiddleware()
                    .concat(
                        productApi.middleware
                    )

    });
```

---

# 40. 왜 Reducer를 등록하는가?

RTK Query는 내부적으로 Cache와 요청 상태를 Redux Store에서 관리합니다.

따라서:

```javascript
productApi.reducer
```

를 Store에 등록해야 합니다.

```text
RTK Query
   ↓
Query Cache
Mutation State
Subscription 관련 상태
   ↓
Redux Store
```

---

# 41. 왜 Middleware를 등록하는가?

RTK Query Middleware는 API 요청 수명 주기와 cache 관련 동작 등을 처리하는 데 사용됩니다.

따라서:

```javascript
productApi.middleware
```

도 Store middleware에 추가합니다.

```text
dispatch
   ↓
Redux Middleware
   ↓
RTK Query Middleware
   ↓
Request / Cache 관리
```

---

# 42. 일반 Slice와 RTK Query를 함께 사용

실제 애플리케이션에서는 다음처럼 사용할 수 있습니다.

```javascript
const store =
    configureStore({

        reducer: {

            auth:
                authReducer,

            ui:
                uiReducer,

            [productApi.reducerPath]:
                productApi.reducer

        },

        middleware:
            getDefaultMiddleware =>

                getDefaultMiddleware()
                    .concat(
                        productApi.middleware
                    )

    });
```

즉:

```text
Redux Store
│
├── auth
│     ↓
│   createSlice
│
├── ui
│     ↓
│   createSlice
│
└── productApi
      ↓
    RTK Query
```

와 같이 Client State와 Server State 관리 코드를 함께 사용할 수 있습니다.

---

# 43. Client State와 Server State의 역할 분리

교육 단계에서는 다음 기준으로 이해하면 좋습니다.

```text
Client State
     ↓
createSlice()


General Async Workflow
     ↓
createAsyncThunk()


Server State Fetching / Caching
     ↓
RTK Query
```

예:

```text
Modal Open
      ↓
createSlice


로그인 이후 여러 비동기 Business Logic
      ↓
createAsyncThunk 가능


Product 목록 조회 및 Cache
      ↓
RTK Query
```

각 도구의 책임을 구분하는 것이 중요합니다.

---

# 44. Authentication과 RTK Query

Spring Security + JWT 환경에서는 API 요청에 Access Token을 넣어야 할 수 있습니다.

예:

```text
Authorization:
Bearer eyJ...
```

`fetchBaseQuery()`의 `prepareHeaders`를 이용하여 요청 Header를 구성할 수 있습니다. 공식 RTK Query 예제에서도 Redux State에서 JWT를 가져와 `prepareHeaders`에서 Authorization header에 넣는 방식을 소개합니다. ([리덕스 툴킷][9])

예:

```javascript
baseQuery: fetchBaseQuery({

    baseUrl: "/api",

    prepareHeaders:
        (headers, { getState }) => {

            const token =
                getState().auth.token;

            if (token) {

                headers.set(
                    "Authorization",
                    `Bearer ${token}`
                );

            }

            return headers;

        }

})
```

---

# 45. 인증 요청 흐름

```text
Redux Store

auth.token
    ↓

prepareHeaders()
    ↓

Authorization Header 추가
    ↓

HTTP Request
    ↓

Spring Security
    ↓

JWT Authentication Filter
    ↓

Controller
```

즉 기존 Redux 인증 State와 RTK Query를 연결할 수 있습니다.

---

# 46. Spring Boot REST API 예제

Spring Boot Controller가 다음과 같다고 합시다.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public List<ProductResponse> getProducts() {
        // ...
    }

    @GetMapping("/{id}")
    public ProductResponse getProduct(
            @PathVariable Long id) {

        // ...

    }

    @PostMapping
    public ProductResponse createProduct(
            @RequestBody ProductRequest request) {

        // ...

    }

    @PutMapping("/{id}")
    public ProductResponse updateProduct(
            @PathVariable Long id,
            @RequestBody ProductRequest request) {

        // ...

    }

    @DeleteMapping("/{id}")
    public void deleteProduct(
            @PathVariable Long id) {

        // ...

    }
}
```

이 API를 RTK Query로 연결할 수 있습니다.

---

# 47. 전체 Product API Slice

```javascript
import {
    createApi,
    fetchBaseQuery
} from "@reduxjs/toolkit/query/react";


export const productApi = createApi({

    reducerPath:
        "productApi",

    baseQuery:
        fetchBaseQuery({

            baseUrl:
                "http://localhost:8080/api"

        }),

    tagTypes: [
        "Product"
    ],

    endpoints:
        builder => ({

            getProducts:
                builder.query({

                    query: () =>
                        "/products",

                    providesTags: [
                        "Product"
                    ]

                }),


            getProduct:
                builder.query({

                    query: id =>
                        `/products/${id}`

                }),


            createProduct:
                builder.mutation({

                    query: product => ({

                        url:
                            "/products",

                        method:
                            "POST",

                        body:
                            product

                    }),

                    invalidatesTags: [
                        "Product"
                    ]

                }),


            updateProduct:
                builder.mutation({

                    query:
                        ({ id, ...product }) => ({

                            url:
                                `/products/${id}`,

                            method:
                                "PUT",

                            body:
                                product

                        }),

                    invalidatesTags: [
                        "Product"
                    ]

                }),


            deleteProduct:
                builder.mutation({

                    query: id => ({

                        url:
                            `/products/${id}`,

                        method:
                            "DELETE"

                    }),

                    invalidatesTags: [
                        "Product"
                    ]

                })

        })

});


export const {

    useGetProductsQuery,

    useGetProductQuery,

    useCreateProductMutation,

    useUpdateProductMutation,

    useDeleteProductMutation

} = productApi;
```

---

# 48. 이 파일 하나에서 만들어지는 것

`createApi()` 하나로 다음 구조가 만들어집니다.

```text
productApi
│
├── reducer
│
├── middleware
│
├── endpoints
│
│    ├── getProducts
│
│    ├── getProduct
│
│    ├── createProduct
│
│    ├── updateProduct
│
│    └── deleteProduct
│
└── React Hooks
     │
     ├── useGetProductsQuery()
     │
     ├── useGetProductQuery()
     │
     ├── useCreateProductMutation()
     │
     ├── useUpdateProductMutation()
     │
     └── useDeleteProductMutation()
```

`createApi()`가 API slice와 fetching/caching 관련 Redux 로직을 생성한다는 점이 RTK Query의 핵심입니다. ([리덕스 툴킷][2])

---

# 49. `createSlice()`와 `createApi()` 비교

이 둘은 이름도 비슷하고 둘 다 Redux Toolkit에 포함되어 있어 혼동하기 쉽습니다.

## `createSlice()`

주로 Client State 관리:

```javascript
createSlice({
    name: "cart",

    initialState: {
        items: []
    },

    reducers: {
        // ...
    }
});
```

## `createApi()`

주로 Server State fetching/caching:

```javascript
createApi({

    baseQuery:
        fetchBaseQuery(...),

    endpoints:
        builder => ({
            // ...
        })

});
```

따라서:

```text
createSlice()
      ↓
State Mutation Logic


createApi()
      ↓
Server Communication
+
Server Data Cache Management
```

라고 구분할 수 있습니다.

---

# 50. RTK Query 내부 흐름

Component에서 다음 코드가 실행되었다고 생각해봅시다.

```javascript
useGetProductsQuery();
```

개념적인 전체 흐름은 다음과 같습니다.

```text
React Component
      ↓
useGetProductsQuery()
      ↓
RTK Query
      ↓
Cache 확인
      ↓
┌─────────────────────┐
│ Cache 사용 가능?    │
└──────────┬──────────┘
           │
     ┌─────┴─────┐
     ↓           ↓
    YES          NO
     ↓           ↓
Cache 사용    HTTP Request
                 ↓
            Spring Boot
                 ↓
              Response
                 ↓
               Cache
                 ↓
            Component
```

세부 동작은 설정과 현재 cache 상태에 따라 달라질 수 있지만, 이 mental model은 RTK Query의 역할을 이해하기 좋습니다. ([리덕스 툴킷][6])

---

# 51. Mutation 이후 흐름

```javascript
createProduct({
    name: "Keyboard"
});
```

를 실행합니다.

```text
React Component
      ↓
createProduct()
      ↓
Mutation Endpoint
      ↓
POST /products
      ↓
Spring Boot
      ↓
Database INSERT
      ↓
Response
      ↓
Mutation Success
      ↓
invalidatesTags: Product
      ↓
Product Cache Invalid
      ↓
Active Query 존재
      ↓
GET /products
      ↓
Cache Update
      ↓
React Component Update
```

이 흐름을 이해하면 RTK Query의 Cache Invalidation을 이해한 것입니다. ([리덕스 툴킷][8])

---

# 52. `createAsyncThunk()`와 RTK Query 최종 비교

## createAsyncThunk

```text
Component
   ↓
dispatch()
   ↓
Thunk
   ↓
API
   ↓
Promise
   ↓
pending / fulfilled / rejected
   ↓
extraReducers
   ↓
Redux State
```

개발자가 직접 관리하는 부분이 상대적으로 많습니다.

```text
loading
error
data
Reducer
State 구조
```

---

## RTK Query

```text
Component
   ↓
Generated Hook
   ↓
RTK Query
   ↓
API
   ↓
Cache
   ↓
Component
```

그리고 RTK Query가:

```text
Request State

Cache

Subscription

Refetch

Invalidation
```

등을 관리합니다. ([리덕스 툴킷][1])

---

# 53. RTK Query가 Redux를 없애는 것은 아니다

RTK Query를 사용한다고 Redux가 필요 없어지는 것이 아닙니다.

실제로 RTK Query API Slice는:

```text
Reducer

Middleware

Actions

Redux Store State
```

와 긴밀하게 동작합니다.

따라서 지금까지 배운 Redux 흐름을 알고 있으면 RTK Query 내부 구조를 훨씬 쉽게 이해할 수 있습니다.

```text
Redux Fundamentals
       ↓
Redux Toolkit
       ↓
Middleware / Thunk
       ↓
RTK Query
```

이 순서로 학습한 이유가 여기에 있습니다.

---

# 54. RTK Query를 사용할 때 특히 중요한 Mental Model

RTK Query를 배울 때 가장 중요한 생각은:

> Component가 서버에서 데이터를 직접 가져와 자신의 State에 저장한다.

가 아닙니다.

다음처럼 생각하는 것이 좋습니다.

```text
Server
   ↓
RTK Query Cache
   ↓
Component가 Cache를 구독
```

즉:

```text
Component
      ↓
Server Data 요청
      ↓
RTK Query가 Cache 관리
      ↓
Component는 Cache Data 사용
```

입니다.

---

# 55. Query는 단순 GET 함수가 아니다

다음 코드를:

```javascript
useGetProductsQuery();
```

단순히:

```javascript
fetch("/products");
```

를 편하게 호출한다고 생각하면 부족합니다.

실제로 개념적으로는:

```text
Query
│
├── Request
├── Cache Entry
├── Query Argument
├── Subscription
├── Loading State
├── Error State
├── Refetch
└── Tag Relation
```

을 함께 생각해야 합니다.

---

# 56. Mutation도 단순 POST 함수가 아니다

마찬가지로:

```javascript
useCreateProductMutation();
```

은 단순히 POST 요청만 담당한다고 이해하면 부족합니다.

Mutation은:

```text
Server State 변경
       ↓
관련 Cache 관계 확인
       ↓
Tag Invalid
       ↓
필요한 Query Refetch
```

와 연결할 수 있습니다. ([리덕스 툴킷][10])

이것이 RTK Query가 단순 HTTP Client와 다른 중요한 이유입니다.

---

# 57. Redux Toolkit 전체 그림

이제 PART 1부터 PART 4까지 모든 내용을 연결할 수 있습니다.

```text
                  Redux Toolkit
                       │
          ┌────────────┴────────────┐
          │                         │
          ↓                         ↓
      createSlice()             createApi()
          │                         │
      Client State              Server State
          │                         │
          ↓                         ↓
        Reducer                  RTK Query
          │                         │
          │               ┌─────────┼──────────┐
          │               ↓         ↓          ↓
          │             Query    Mutation     Cache
          │               │         │          │
          └───────────────┴─────────┴──────────┘
                          │
                          ↓
                     Redux Store
                          │
                          ↓
                       React
```

---

# 58. PART 1 ~ PART 4 전체 학습 흐름

## PART 1 — Redux Fundamentals

```text
State
Action
Action Creator
dispatch
Reducer
Store
```

↓

## PART 2 — Redux Toolkit

```text
createSlice
Immer
configureStore
Provider
useSelector
useDispatch
```

↓

## PART 3 — Asynchronous Redux

```text
Middleware
Thunk
Promise
createAsyncThunk
pending
fulfilled
rejected
extraReducers
```

↓

## PART 4 — RTK Query

```text
Server State
createApi
fetchBaseQuery
Endpoint
Query
Mutation
Generated Hook
Cache
Cache Key
Subscription
Tag
Invalidation
Refetch
```

---

# 59. PART 4 핵심 실행 흐름

다음 코드:

```javascript
const {
    data,
    isLoading,
    error
} =
    useGetProductsQuery();
```

를 보면 머릿속에서 다음 구조가 보여야 합니다.

```text
Component
   ↓
useGetProductsQuery()
   ↓
Query Subscription
   ↓
RTK Query
   ↓
Cache 확인
   ↓
필요한 경우 Request
   ↓
Spring Boot REST API
   ↓
Response
   ↓
RTK Query Cache
   ↓
Query Result
   ↓
Component
```

그리고:

```javascript
createProduct(product);
```

를 보면:

```text
Mutation
   ↓
POST /products
   ↓
Server State 변경
   ↓
Product Tag Invalid
   ↓
관련 Cache 확인
   ↓
Active Query Refetch
   ↓
Cache Update
   ↓
Component Update
```

가 보여야 합니다. ([리덕스 툴킷][8])

---

# 60. 최종 정리

RTK Query는 Redux Toolkit에 포함된 **Server State Data Fetching + Caching 시스템**입니다. 단순히 `fetch()`를 대신하는 것이 아니라 query result cache, subscription, mutation, invalidation, refetch 등의 서버 데이터 생명주기를 관리합니다. ([리덕스 툴킷][1])

`createApi()`는 Backend API와의 통신 방법을 endpoint 단위로 정의하고 fetching/caching 로직 및 React Hook을 생성하는 RTK Query의 중심 API입니다. ([리덕스 툴킷][2])

```text
createApi
   ↓
API Slice
   ↓
Endpoints
   │
   ├── Query
   │      ↓
   │    Fetch
   │      ↓
   │    Cache
   │
   └── Mutation
          ↓
       Server 변경
          ↓
       Cache Invalid
          ↓
         Refetch
```

따라서 Redux Toolkit을 사용하는 React 애플리케이션을 큰 관점에서 보면 다음과 같이 역할을 분리할 수 있습니다.

```text
Local UI State
      ↓
useState


Shared Client State
      ↓
createSlice


일반적인 복잡한 비동기 Redux Logic
      ↓
createAsyncThunk


Server State
Fetching + Caching
      ↓
RTK Query
```

그리고 지금까지 배운 모든 Redux 개념은 결국 하나의 흐름으로 연결됩니다.

```text
React
  ↓
Redux / Redux Toolkit
  ↓
Client State
+
Server State
  ↓
Predictable State Management
```

RTK Query까지 이해하면 Redux Toolkit을 단순히 `createSlice()`를 사용하는 라이브러리가 아니라, **React 애플리케이션의 Client State와 Server State를 서로 다른 책임으로 관리할 수 있는 종합적인 Redux 도구 모음**으로 이해할 수 있습니다.

[1]: https://redux-toolkit.js.org/rtk-query/overview?utm_source=chatgpt.com "RTK Query Overview"
[2]: https://redux-toolkit.js.org/rtk-query/api/createApi?utm_source=chatgpt.com "createApi"
[3]: https://redux-toolkit.js.org/rtk-query/api/fetchBaseQuery?utm_source=chatgpt.com "fetchBaseQuery"
[4]: https://redux-toolkit.js.org/rtk-query/usage/queries?utm_source=chatgpt.com "Queries"
[5]: https://redux-toolkit.js.org/rtk-query/api/created-api/hooks?utm_source=chatgpt.com "API Slices: React Hooks"
[6]: https://redux-toolkit.js.org/rtk-query/usage/cache-behavior?utm_source=chatgpt.com "Cache Behavior"
[7]: https://redux-toolkit.js.org/rtk-query/usage/usage-without-react-hooks?utm_source=chatgpt.com "Usage Without React Hooks"
[8]: https://redux-toolkit.js.org/rtk-query/usage/automated-refetching?utm_source=chatgpt.com "Automated Re-fetching"
[9]: https://redux-toolkit.js.org/rtk-query/usage/examples?utm_source=chatgpt.com "RTK Query Examples"
[10]: https://redux-toolkit.js.org/rtk-query/usage/mutations?utm_source=chatgpt.com "Mutations"
