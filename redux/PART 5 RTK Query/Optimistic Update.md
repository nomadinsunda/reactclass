RTK Query에서 **낙관적 업데이트(Optimistic Update)** 는 서버 응답을 기다리지 않고 **“성공할 것이라고 가정하고 먼저 캐시/UI를 바꿔버리는 방식”**입니다.

예를 들어 사용자가 상품 이름을 수정했다고 하겠습니다. 일반적인 방식이라면 `PATCH /products/10` 요청을 보내고 서버가 성공 응답을 준 뒤 화면을 바꿉니다. 반면 낙관적 업데이트는 버튼을 누르는 순간 RTK Query 캐시를 먼저 수정해서 화면에 즉시 반영하고, 그 뒤 서버 요청 결과를 기다립니다. 서버 요청이 성공하면 그대로 유지하고, 실패하면 원래 상태로 되돌립니다. RTK Query에서는 보통 mutation의 `onQueryStarted`와 `api.util.updateQueryData()`를 조합해서 구현합니다. ([Redux Toolkit][1])

```text
사용자 수정
   ↓
Mutation 시작
   ↓
캐시를 먼저 변경
   ↓
UI 즉시 변경
   ↓
HTTP 요청 진행
   ↓
┌───────────────┐
│ 서버 성공     │ → 그대로 유지
└───────────────┘

┌───────────────┐
│ 서버 실패     │ → 캐시 rollback
└───────────────┘
```

가장 중요한 특징은 **UI가 서버 응답을 기다리지 않는다는 것**입니다.

---

예를 들어 먼저 상품 조회 endpoint가 있다고 해보겠습니다.

```js
getProduct: builder.query({
  query: (id) => `/products/${id}`,
}),
```

캐시에 다음 데이터가 들어 있다고 가정하겠습니다.

```js
{
  id: 10,
  name: "Keyboard",
  price: 50000
}
```

React에서는:

```js
const { data } = useGetProductQuery(10);
```

으로 이 캐시 데이터를 보고 있습니다.

이제 상품 이름을 `"Keyboard"`에서 `"Mechanical Keyboard"`로 변경한다고 하겠습니다.

mutation을 다음과 같이 정의할 수 있습니다.

```js
updateProduct: builder.mutation({
  query: ({ id, ...patch }) => ({
    url: `/products/${id}`,
    method: "PATCH",
    body: patch,
  }),

  async onQueryStarted(
    { id, ...patch },
    { dispatch, queryFulfilled }
  ) {

    const patchResult = dispatch(
      productApi.util.updateQueryData(
        "getProduct",
        id,
        (draft) => {
          Object.assign(draft, patch);
        }
      )
    );

    try {
      await queryFulfilled;
    } catch {
      patchResult.undo();
    }
  },
}),
```

이 코드가 낙관적 업데이트의 핵심 형태입니다. 공식 문서도 같은 패턴을 권장 예제로 보여줍니다. ([Redux Toolkit][1])

여기서 각각을 뜯어보겠습니다.

## `onQueryStarted`가 핵심 출발점

```js
async onQueryStarted(
  { id, ...patch },
  { dispatch, queryFulfilled }
) {
```

`onQueryStarted`는 이름 그대로 **query 또는 mutation이 시작되는 순간 실행되는 lifecycle callback**입니다. 즉 서버 응답이 도착한 후가 아니라 요청이 시작될 때 바로 실행됩니다. 두 번째 아규먼트에는 `dispatch`, `getState`, `queryFulfilled`, `requestId`, `getCacheEntry` 등의 lifecycle API가 전달됩니다. ([Redux Toolkit][2])

흐름은:

```text
updateProduct(...)
      ↓
Mutation 시작
      ↓
onQueryStarted() 즉시 실행
      ↓
HTTP 요청은 아직 진행 중
```

입니다.

그래서 낙관적 업데이트를 수행하기에 적합합니다.

---

## `updateQueryData()`는 무엇을 하는가?

핵심 코드는 이것입니다.

```js
dispatch(
  productApi.util.updateQueryData(
    "getProduct",
    id,
    (draft) => {
      Object.assign(draft, patch);
    }
  )
);
```

`updateQueryData()`는 **이미 존재하는 RTK Query 캐시 데이터를 직접 수정하기 위한 thunk action creator**입니다. dispatch되면 현재 캐시 엔트리에 patch를 적용합니다. ([Redux Toolkit][1])

구조를 보면:

```js
updateQueryData(
  endpointName,
  arg,
  recipe
)
```

입니다.

우리 예제에서는:

```js
updateQueryData(
  "getProduct",  // 어떤 query endpoint의 캐시인가?
  10,            // 그 query를 호출했을 때의 아규먼트
  (draft) => {   // 캐시를 어떻게 바꿀 것인가?
    Object.assign(draft, patch);
  }
)
```

즉 정확히 어떤 캐시를 수정할지 찾기 위해:

```text
endpointName + query argument
```

를 사용합니다.

예를 들어:

```js
useGetProductQuery(10);
```

의 캐시는 개념적으로:

```text
getProduct(10)
```

이고,

```js
useGetProductQuery(20);
```

은 별개의 캐시입니다.

```text
RTK Query Cache

getProduct(10)
{
  id: 10,
  name: "Keyboard"
}

getProduct(20)
{
  id: 20,
  name: "Mouse"
}
```

따라서:

```js
updateQueryData(
  "getProduct",
  10,
  ...
)
```

은 **`getProduct(10)` 캐시만 수정**합니다.

여기서 매우 중요한 점은 `updateQueryData()`는 **이미 존재하는 cache entry를 수정하는 용도**라는 것입니다. 해당 `endpointName + arg` 조합의 캐시가 존재하지 않으면 새 캐시를 만들어 주지 않습니다. 새 캐시 생성이나 교체에는 `upsertQueryData()`가 사용됩니다. ([Redux Toolkit][1])

---

## `draft`는 무엇인가?

여기:

```js
(draft) => {
  Object.assign(draft, patch);
}
```

의 `draft`는 현재 캐시 데이터의 **Immer draft**입니다.

따라서 현재 캐시가:

```js
{
  id: 10,
  name: "Keyboard",
  price: 50000
}
```

이고:

```js
patch = {
  name: "Mechanical Keyboard"
}
```

라면:

```js
Object.assign(draft, patch);
```

실행 후 캐시는:

```js
{
  id: 10,
  name: "Mechanical Keyboard",
  price: 50000
}
```

가 됩니다.

코드는 마치 직접 객체를 변경하는 것처럼 작성하지만, RTK Query가 내부적으로 안전한 immutable update를 처리합니다.

```text
기존 Cache
    ↓
Immer Draft
    ↓
draft.name 변경
    ↓
새 Cache State 생성
```

입니다.

---

## 여기서 UI가 왜 즉시 바뀌는가?

React 컴포넌트가:

```js
const { data } =
  useGetProductQuery(10);
```

을 통해 RTK Query cache를 구독하고 있기 때문입니다.

`updateQueryData()`가 캐시를 변경하면:

```text
dispatch(updateQueryData(...))
        ↓
RTK Query Cache 변경
        ↓
Redux Store 변경
        ↓
useGetProductQuery(10)
새 Cache State 감지
        ↓
Component 리렌더링
        ↓
새 이름 즉시 표시
```

가 됩니다.

서버 응답을 아직 받지 않았는데도:

```text
Keyboard
```

가 곧바로:

```text
Mechanical Keyboard
```

로 보입니다.

이게 바로 **Optimistic**이라는 이름의 이유입니다.

> “서버에서도 성공할 거라고 낙관적으로 가정하고 화면부터 바꾼다.”

---

## 그런데 서버 요청은 언제 실행되나?

캐시 변경과 별개로 mutation의 `query()`도 실행됩니다.

```js
query: ({ id, ...patch }) => ({
  url: `/products/${id}`,
  method: "PATCH",
  body: patch,
})
```

따라서 전체적으로는 거의 동시에 두 흐름이 진행됩니다.

```text
updateProduct({
  id: 10,
  name: "Mechanical Keyboard"
})

               ↓

       Mutation 시작
        ↙             ↘

onQueryStarted()       query()
      ↓                  ↓
Cache 먼저 변경       PATCH /products/10
      ↓                  ↓
UI 즉시 변경          Server 처리
```

즉 **캐시 업데이트 때문에 서버 요청이 없어지는 것은 아닙니다.**

서버 요청은 정상적으로 진행됩니다.

---

## `queryFulfilled`가 매우 중요하다

`onQueryStarted`의 두 번째 아규먼트에서:

```js
{
  dispatch,
  queryFulfilled
}
```

를 받고 있죠.

`queryFulfilled`는 **현재 요청이 성공하거나 실패하는 것을 기다릴 수 있는 Promise**입니다. 성공하면 resolve되고 실패하면 reject됩니다. ([Redux Toolkit][2])

따라서:

```js
try {
  await queryFulfilled;
} catch {
  ...
}
```

라는 코드를 사용할 수 있습니다.

흐름은:

```text
HTTP 요청
    ↓
queryFulfilled
    ↓

성공
Promise resolve

실패
Promise reject
```

입니다.

---

## 성공하면 무엇을 하는가?

사실 아무것도 하지 않아도 됩니다.

```js
try {
  await queryFulfilled;
}
```

에서 요청이 성공했다면 이미 우리가 캐시를 먼저 변경해 놓았기 때문입니다.

```text
캐시
Keyboard
   ↓
Optimistic Update
Mechanical Keyboard
   ↓
서버 요청 성공
   ↓
그대로 유지
```

입니다.

그래서 성공 처리 코드가 없어 보이는 것입니다.

---

## 실패하면 어떻게 하나?

바로 이 부분입니다.

```js
catch {
  patchResult.undo();
}
```

`dispatch(updateQueryData())`의 반환값에는 **적용한 변경을 되돌릴 수 있는 `undo()`**가 포함됩니다. 공식 API에서 `updateQueryData()`는 `patches`, `inversePatches`, `undo()`를 가진 patch collection을 반환합니다. ([Redux Toolkit][3])

그래서:

```js
const patchResult = dispatch(
  productApi.util.updateQueryData(...)
);
```

에서 `patchResult`를 저장해 둡니다.

서버 요청 실패:

```js
catch {
  patchResult.undo();
}
```

하면 이전 캐시 상태로 rollback 됩니다.

```text
원래 Cache
Keyboard
    ↓
Optimistic Update
Mechanical Keyboard
    ↓
UI도 즉시 변경
    ↓
PATCH 요청 실패
    ↓
patchResult.undo()
    ↓
Keyboard
    ↓
UI도 다시 원래 상태
```

이 **rollback**이 낙관적 업데이트에서 굉장히 중요합니다.

---

## 왜 `undo()`가 가능한가?

`updateQueryData()`가 단순히 값을 덮어쓰는 것만 하는 게 아니라 내부적으로 변경 patch와 반대 patch를 만들어 두기 때문입니다.

개념적으로:

```text
기존 데이터
name: "Keyboard"

        ↓ update

새 데이터
name: "Mechanical Keyboard"

동시에 내부적으로

patch
Keyboard
→ Mechanical Keyboard

inversePatch
Mechanical Keyboard
→ Keyboard
```

를 가지고 있다고 생각하면 됩니다.

그래서:

```js
patchResult.undo();
```

를 호출하면 inverse patch가 적용됩니다.

---

## 전체 코드를 다시 보면

```js
updateProduct: builder.mutation({
  query: ({ id, ...patch }) => ({
    url: `/products/${id}`,
    method: "PATCH",
    body: patch,
  }),

  async onQueryStarted(
    { id, ...patch },
    { dispatch, queryFulfilled }
  ) {

    // ① 서버 응답 전에 캐시부터 수정
    const patchResult = dispatch(
      productApi.util.updateQueryData(
        "getProduct",
        id,
        (draft) => {
          Object.assign(draft, patch);
        }
      )
    );

    try {
      // ② 실제 서버 요청 결과 기다림
      await queryFulfilled;

      // ③ 성공 → 이미 Cache가 변경됐으므로 유지
    } catch {

      // ④ 실패 → 원래 Cache로 rollback
      patchResult.undo();
    }
  },
}),
```

이 코드의 흐름은:

```text
Mutation Trigger
      ↓
onQueryStarted()
      ↓
updateQueryData()
      ↓
Cache 즉시 변경
      ↓
UI 즉시 변경
      ↓
await queryFulfilled
      ↓
 ┌──────────────┐
 │              │
성공            실패
 │               │
유지          undo()
                 ↓
              rollback
```

입니다.

---

## 일반 Mutation과 비교하면 차이가 명확합니다

보통 tag invalidation을 사용하면:

```js
updateProduct: builder.mutation({
  query: ...,

  invalidatesTags: ["Product"]
})
```

흐름이 대략:

```text
Mutation
   ↓
Server 요청
   ↓
성공
   ↓
Tag invalidate
   ↓
Query refetch
   ↓
새 데이터
   ↓
UI 변경
```

입니다.

즉 서버 round trip을 기다립니다.

반면 낙관적 업데이트는:

```text
Mutation
   ↓
Cache 즉시 변경
   ↓
UI 즉시 변경
   │
   └─────────────┐
                 ↓
             Server 요청
                 ↓
           성공 / 실패 확인
```

입니다.

RTK Query 공식 문서는 일반적으로 최신 데이터를 유지할 때 자동 tag invalidation과 refetch를 우선적인 방법으로 권장하지만, 즉각적인 사용자 피드백 등이 필요한 경우 manual cache update, 특히 optimistic update가 적합하다고 설명합니다. ([Redux Toolkit][1])

---

## 어디에 쓰면 좋은가?

대표적인 예는 이런 UI입니다.

* 좋아요 버튼
* 즐겨찾기
* 체크박스
* Todo 완료 상태
* 상품 수량 `+ / -`
* 간단한 이름/제목 수정
* 게시물 상태 변경

예를 들어 좋아요 버튼을 눌렀는데 서버 응답까지 500ms를 기다린 다음 하트가 바뀐다면 UI가 답답하게 느껴집니다.

```text
클릭
 ↓
♡ → ♥
 ↓
서버 요청
```

처럼 바로 보여주는 것이 훨씬 자연스럽습니다.

실패했을 때만:

```text
♥
↓
서버 실패
↓
♡
```

로 되돌리는 것입니다.

---

## 낙관적 업데이트와 비관적 업데이트의 차이

RTK Query에는 **Pessimistic Update** 패턴도 있습니다. 공식 문서에서는 pessimistic update는 먼저 `queryFulfilled`를 기다린 뒤 서버 응답을 이용하여 캐시를 수정하는 방식으로 설명합니다. ([Redux Toolkit][1])

```text
Optimistic

Mutation
   ↓
Cache 먼저 변경
   ↓
UI 변경
   ↓
Server 응답
   ↓
실패하면 rollback
```

반대로:

```text
Pessimistic

Mutation
   ↓
Server 응답 기다림
   ↓
성공
   ↓
Cache 변경
   ↓
UI 변경
```

즉 핵심 차이는 **캐시를 언제 변경하느냐**입니다.

```text
Optimistic
Server 응답 전

Pessimistic
Server 응답 후
```

---

## 중요한 주의점: 여러 mutation이 겹치면?

예를 들어 사용자가 아주 빠르게:

```text
좋아요
취소
좋아요
취소
```

를 누른다고 해보겠습니다.

여러 mutation이 동시에 진행되면 각 요청의 patch와 rollback 순서가 꼬일 수 있습니다.

```text
Mutation A
Mutation B
Mutation C
       ↓
응답 순서는
B → A → C
```

처럼 올 수도 있습니다.

RTK Query 공식 문서도 **여러 mutation이 짧은 시간에 겹치는 경우 `.undo()` 기반 rollback에서 race condition이 발생할 수 있다**고 경고합니다. 이런 경우에는 실패 시 직접 undo하기보다 관련 tag를 invalidate해서 서버로부터 다시 fetch하는 것이 더 단순하고 안전할 수 있습니다. ([Redux Toolkit][1])

예를 들어:

```js
catch {
  dispatch(
    productApi.util.invalidateTags(["Product"])
  );
}
```

처럼 처리하는 것입니다.

즉:

```text
단순한 mutation
→ undo() 사용 가능

여러 mutation이 겹칠 가능성이 큼
→ 실패 시 invalidateTags()
→ 서버 데이터 다시 fetch
```

가 더 안전할 수 있습니다.

---

그리고 지금까지 보신 RTK Query 구조와 연결하면 낙관적 업데이트는 정확히 여기 들어갑니다.

```text
React Component
      ↓
useUpdateProductMutation()
      ↓
trigger
updateProduct(...)
      ↓
Mutation endpoint
      ↓
onQueryStarted()
      │
      ├─ dispatch(
      │    api.util.updateQueryData(...)
      │  )
      │       ↓
      │    Cache 즉시 변경
      │       ↓
      │    Component 즉시 업데이트
      │
      └─ 실제 HTTP 요청
              ↓
        queryFulfilled
          ↙        ↘
       성공          실패
        │             │
      유지          undo()
                       ↓
                    rollback
```

이 구조가 **RTK Query 낙관적 업데이트의 핵심 메커니즘**입니다.

한 줄로 정리하면:

> **RTK Query의 낙관적 업데이트는 mutation이 시작되는 순간 `onQueryStarted`에서 `updateQueryData()`를 dispatch해 캐시를 먼저 변경하고, 서버 요청이 성공하면 그대로 유지하며, 실패하면 `undo()` 또는 tag invalidation으로 원래 상태를 복구하는 패턴입니다.** ([Redux Toolkit][1])

[1]: https://redux-toolkit.js.org/rtk-query/usage/manual-cache-updates?utm_source=chatgpt.com "Manual Cache Updates | Redux Toolkit"
[2]: https://redux-toolkit.js.org/rtk-query/api/createApi?utm_source=chatgpt.com "createApi | Redux Toolkit"
[3]: https://redux-toolkit.js.org/rtk-query/api/created-api/api-slice-utils?utm_source=chatgpt.com "API Slices: Utilities | Redux Toolkit"
