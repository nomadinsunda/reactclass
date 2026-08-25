# React Router v6의 `useParams()`

`useParams()`는 React Router에서 **현재 매칭된 Route의 동적 Path Parameter를 읽기 위한 Hook**입니다.

React Router에서는 URL 경로의 일부를 고정된 문자열이 아니라 **동적으로 변하는 값**으로 정의할 수 있습니다.

예를 들어:

```text
/users/10
/users/20
/users/100
```

이 세 URL은 모두 다음 하나의 Route Pattern으로 표현할 수 있습니다.

```jsx
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

여기서:

```text
:userId
```

가 **동적 세그먼트(dynamic segment)**입니다.

현재 URL이:

```text
/users/10
```

이라면 React Router는:

```text
:userId
   │
   ▼
  "10"
```

으로 매칭합니다.

그리고 컴포넌트에서는:

```jsx
const { userId } = useParams();
```

를 통해 이 값을 읽을 수 있습니다.

---

# 1. `useParams()`를 한 문장으로 정의하면

> `useParams()`는 현재 React Router의 Route Match 결과에 포함된 동적 Path Parameter들을 객체 형태로 반환하는 Hook입니다.

가장 기본적인 흐름은 다음과 같습니다.

```text
Route Pattern
/users/:userId
        │
        │
        ▼

현재 URL
/users/10
        │
        ▼

Route Matching
        │
        ▼

params
{
  userId: "10"
}
        │
        ▼

useParams()
```

---

# 2. 왜 `useParams()`가 필요한가?

SPA에서 URL은 단순한 주소가 아니라 **현재 어떤 데이터를 보여줄 것인지 표현하는 역할**도 합니다.

예를 들어:

```text
/products/10
/products/11
/products/12
```

각 URL은 서로 다른 상품을 의미할 수 있습니다.

하지만 상품마다 Route를 따로 만들지는 않습니다.

```jsx
<Route path="/products/10" ... />
<Route path="/products/11" ... />
<Route path="/products/12" ... />
```

대신 다음과 같이 하나의 동적 Route를 정의합니다.

```jsx
<Route
  path="/products/:productId"
  element={<ProductDetail />}
/>
```

그리고:

```jsx
const { productId } = useParams();
```

로 현재 상품 ID를 읽습니다.

즉:

```text
/products/10
/products/11
/products/12
      │
      ▼
/products/:productId
      │
      ▼
useParams()
      │
      ▼
현재 productId 확인
```

이것이 동적 라우팅의 핵심입니다.

---

# 3. 기본 사용법

Route가 다음과 같다고 하겠습니다.

```jsx
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

현재 URL:

```text
/users/88
```

컴포넌트:

```jsx
import { useParams } from 'react-router-dom';

function UserDetail() {
  const { userId } = useParams();

  return (
    <h1>
      사용자 ID: {userId}
    </h1>
  );
}
```

화면에는:

```text
사용자 ID: 88
```

이 출력됩니다.

---

# 4. `params`는 어디에서 만들어지는가?

`useParams()`가 브라우저 URL 문자열을 직접 분석하는 것은 아닙니다.

React Router가 먼저 **현재 URL과 Route Pattern을 매칭**합니다.

예:

```jsx
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

현재 URL:

```text
/users/10
```

React Router는 개념적으로 다음과 같이 비교합니다.

```text
Route Pattern

/users/:userId
│      │
│      └── Dynamic Segment
│
└── Static Segment


현재 URL

/users/10
│      │
│      └── "10"
│
└── "users"
```

매칭 결과:

```js
{
  userId: '10'
}
```

이 값은 현재 Route Match 정보에 포함됩니다.

그리고:

```jsx
useParams()
```

는 그 Match 결과에 포함된 `params`를 읽어옵니다.

즉:

```text
현재 URL
   │
   ▼
Route Matching
   │
   ▼
Route Match
   │
   └── params
        │
        ▼
   useParams()
```

입니다.

---

# 5. 반환값

`useParams()`는 Parameter 이름과 값을 가진 객체를 반환합니다.

개념적인 타입은 다음과 같습니다.

```ts
{
  [key: string]: string | undefined
}
```

예:

```jsx
const params = useParams();
```

현재 매칭 결과가:

```text
/users/10/posts/300
```

이고 Route가:

```text
/users/:userId/posts/:postId
```

라면:

```js
params
```

는:

```js
{
  userId: '10',
  postId: '300'
}
```

이 됩니다.

---

# 6. Parameter 값은 문자열이다

매우 중요한 점입니다.

현재 URL이:

```text
/users/10
```

이라고 해도:

```jsx
const { userId } = useParams();
```

에서 `userId`는 숫자 `10`이 아니라:

```js
'10'
```

이라는 문자열입니다.

따라서:

```js
typeof userId
```

는:

```text
string
```

입니다.

숫자가 필요하다면 직접 변환해야 합니다.

```jsx
const { userId } = useParams();

const id = Number(userId);
```

또는:

```jsx
const id = parseInt(userId, 10);
```

처럼 사용할 수 있습니다.

---

# 7. 여러 개의 Parameter

Route Pattern에는 여러 개의 동적 세그먼트를 정의할 수 있습니다.

```jsx
<Route
  path="/users/:userId/posts/:postId"
  element={<PostDetail />}
/>
```

현재 URL:

```text
/users/88/posts/100
```

컴포넌트:

```jsx
function PostDetail() {
  const { userId, postId } = useParams();

  console.log(userId);
  console.log(postId);

  return <div>Post</div>;
}
```

결과:

```text
userId = "88"
postId = "100"
```

전체 매칭을 보면:

```text
/users/:userId/posts/:postId
        │             │
        ▼             ▼
       88            100
        │             │
        ▼             ▼

params.userId     params.postId
    "88"              "100"
```

---

# 8. Parameter 이름은 Route에 정의한 이름과 같다

다음 Route가 있다고 하겠습니다.

```jsx
<Route
  path="/products/:productId"
  element={<Product />}
/>
```

그러면 `useParams()`에서 읽어야 하는 이름은:

```jsx
const { productId } = useParams();
```

입니다.

다음은 잘못된 예입니다.

```jsx
const { id } = useParams();
```

Route에 `:id`가 없기 때문입니다.

즉:

```text
Route

/products/:productId
           │
           ▼

params
{
  productId: "10"
}
```

Parameter 이름은 개발자가 Route Pattern에서 직접 정합니다.

---

# 9. Nested Route에서의 `useParams()`

`useParams()`는 중첩 Route에서 특히 중요합니다.

다음과 같은 Route 구조를 생각해 봅시다.

```jsx
<Route path="users">
  <Route path=":userId">
    <Route
      path="posts/:postId"
      element={<UserPost />}
    />
  </Route>
</Route>
```

현재 URL:

```text
/users/10/posts/333
```

React Router의 매칭 구조는 개념적으로:

```text
users
  │
  └── :userId
         │
         └── posts/:postId
```

입니다.

각 동적 값은:

```text
:userId → "10"

:postId → "333"
```

으로 매칭됩니다.

따라서 `UserPost`에서:

```jsx
const params = useParams();
```

를 실행하면:

```js
{
  userId: '10',
  postId: '333'
}
```

을 얻을 수 있습니다.

---

# 10. 부모 Route의 params도 사용할 수 있다

중첩 Route에서는 현재 Route까지 매칭된 **부모 Route의 Parameter도 함께 사용할 수 있습니다.**

예:

```text
/users/:userId/posts/:postId
        │             │
        │             └── 자식 Route parameter
        │
        └──────────────── 부모 Route parameter
```

현재 URL:

```text
/users/10/posts/333
```

자식 컴포넌트에서:

```jsx
const { userId, postId } = useParams();
```

를 사용할 수 있습니다.

즉:

```text
Parent Match
:userId = "10"
      │
      ▼
Child Match
:postId = "333"
      │
      ▼
useParams()
      │
      ▼
{
  userId: "10",
  postId: "333"
}
```

라고 이해할 수 있습니다.

---

# 11. `useParams()`의 내부 흐름

다음 Route가 있다고 하겠습니다.

```jsx
<Route
  path="/products/:productId"
  element={<Product />}
/>
```

현재 URL:

```text
/products/30
```

개념적인 내부 흐름은 다음과 같습니다.

```text
Browser URL
/products/30
      │
      ▼
React Router
      │
      ▼
Route Tree와 비교
      │
      ▼
/products/:productId
      │
      │ Match
      ▼
productId = "30"
      │
      ▼
Route Match
      │
      └── params
           │
           ▼
      {
        productId: "30"
      }
           │
           ▼
       useParams()
           │
           ▼
     React Component
```

핵심은:

> `useParams()`가 직접 URL을 파싱하는 것이 아니라 React Router가 수행한 Route Matching 결과를 읽는다는 것입니다.

---

# 12. URL Parameter가 변경되면 어떻게 되는가?

현재 URL:

```text
/products/10
```

에서:

```text
/products/11
```

로 이동한다고 하겠습니다.

같은 Route Pattern이 사용될 수 있습니다.

```jsx
<Route
  path="/products/:productId"
  element={<ProductDetail />}
/>
```

변경 전:

```text
productId = "10"
```

변경 후:

```text
productId = "11"
```

흐름:

```text
/products/10
      │
      ▼
productId = "10"


URL 변경


/products/11
      │
      ▼
새 Route Match
      │
      ▼
productId = "11"
      │
      ▼
컴포넌트가 새로운 params로 렌더링
```

따라서 같은 컴포넌트가 유지되더라도 새로운 Parameter 값을 기준으로 다시 렌더링될 수 있습니다.

---

# 13. 실전 패턴 1: 상세 데이터 조회

가장 대표적인 패턴입니다.

Route:

```jsx
<Route
  path="/products/:productId"
  element={<ProductDetail />}
/>
```

컴포넌트:

```jsx
function ProductDetail() {
  const { productId } = useParams();

  useEffect(() => {
    fetch(`/api/products/${productId}`)
      .then(response => response.json())
      .then(data => {
        console.log(data);
      });
  }, [productId]);

  return <div>Product Detail</div>;
}
```

흐름:

```text
/products/10
      │
      ▼
useParams()
      │
      ▼
productId = "10"
      │
      ▼
API Request
      │
      ▼
GET /api/products/10
```

---

# 14. Parameter 변경과 데이터 재조회

다음 URL에서:

```text
/products/10
```

다음 URL로 이동한다고 하겠습니다.

```text
/products/11
```

`productId`가 변경됩니다.

```text
"10"
 ↓
"11"
```

Effect가 다음과 같이 작성되어 있다면:

```jsx
useEffect(() => {
  fetchProduct(productId);
}, [productId]);
```

흐름은:

```text
productId 변경
      │
      ▼
렌더링
      │
      ▼
useEffect
      │
      ▼
새 상품 데이터 요청
```

이 됩니다.

---

# 15. 실전 패턴 2: Parameter를 이용한 조건부 UI

Parameter 값에 따라 UI를 변경할 수도 있습니다.

Route:

```jsx
<Route
  path="/users/:type"
  element={<UserPage />}
/>
```

컴포넌트:

```jsx
function UserPage() {
  const { type } = useParams();

  if (type === 'admin') {
    return <AdminPage />;
  }

  return <NormalUserPage />;
}
```

현재 URL:

```text
/users/admin
```

이라면:

```text
type = "admin"
      │
      ▼
<AdminPage />
```

가 렌더링됩니다.

---

# 16. 실전 패턴 3: `useParams()` + `useNavigate()`

현재 Parameter를 이용하여 새로운 URL을 만들 수도 있습니다.

```jsx
function UserDetail() {
  const { userId } = useParams();
  const navigate = useNavigate();

  const handleEdit = () => {
    navigate(`/users/${userId}/edit`);
  };

  return (
    <button onClick={handleEdit}>
      수정
    </button>
  );
}
```

현재 URL:

```text
/users/10
```

에서 버튼을 클릭하면:

```text
userId = "10"
      │
      ▼
navigate("/users/10/edit")
      │
      ▼
/users/10/edit
```

로 이동합니다.

---

# 17. `useParams()`와 `useMatch()`의 차이

둘 다 Path Parameter를 다룰 수 있기 때문에 혼동하기 쉽습니다.

`useParams()`는:

> **현재 Route Match에 이미 존재하는 Parameter를 읽습니다.**

```jsx
const { userId } = useParams();
```

반면 `useMatch()`는:

> **개발자가 특정 Path Pattern을 직접 전달하고 현재 pathname이 그 Pattern과 일치하는지 검사합니다.**

```jsx
const match = useMatch('/users/:userId');
```

비교하면:

```text
useParams()

Route Matching이 이미 완료됨
        │
        ▼
현재 params 읽기


useMatch()

현재 pathname
      +
개발자가 지정한 Pattern
        │
        ▼
직접 Match 검사
```

| 구분            | `useParams()`      | `useMatch()`    |
| ------------- | ------------------ | --------------- |
| 목적            | 현재 Route params 읽기 | 특정 Pattern 검사   |
| Pattern 직접 전달 | X                  | O               |
| Match 여부 확인   | 직접 목적 아님           | O               |
| params 반환     | O                  | Match 객체 내부에서 O |

---

# 18. `useParams()`와 `useSearchParams()`의 차이

이 둘은 반드시 구분해야 합니다.

현재 URL:

```text
/products/10?page=2
```

구조를 보면:

```text
/products/10?page=2
│          │
│          └── Query Parameter
│
└───────────── Path
```

Route:

```jsx
<Route
  path="/products/:productId"
  element={<Product />}
 />
```

`useParams()`:

```jsx
const { productId } = useParams();
```

결과:

```text
"10"
```

`useSearchParams()`:

```jsx
const [searchParams] = useSearchParams();

searchParams.get('page');
```

결과:

```text
"2"
```

즉:

```text
/products/10?page=2
          │      │
          │      └── useSearchParams()
          │
          └───────── useParams()
```

입니다.

---

# 19. `useLocation()`과의 차이

`useLocation()`은 현재 Location 자체를 읽습니다.

```jsx
const location = useLocation();
```

예:

```js
{
  pathname: '/products/10',
  search: '?page=2',
  hash: '',
  state: null,
  key: 'abc'
}
```

반면 `useParams()`는:

```jsx
const params = useParams();
```

Route Pattern을 기준으로 추출된:

```js
{
  productId: '10'
}
```

을 반환합니다.

즉:

```text
useLocation()
      │
      ▼
현재 Location 정보


useParams()
      │
      ▼
현재 Route의 Dynamic Parameter
```

---

# 20. 없는 Parameter는 `undefined`일 수 있다

`useParams()`의 값은 타입상 `undefined`일 가능성을 고려해야 합니다.

예:

```jsx
const { userId } = useParams();
```

TypeScript에서는 상황에 따라:

```ts
string | undefined
```

로 취급될 수 있습니다.

따라서 안전하게 처리하려면:

```jsx
if (!userId) {
  return <div>잘못된 사용자 ID입니다.</div>;
}
```

처럼 검사할 수 있습니다.

또는 숫자로 변환한다면:

```jsx
const id = Number(userId);

if (Number.isNaN(id)) {
  return <div>올바르지 않은 ID입니다.</div>;
}
```

와 같은 검증이 필요할 수 있습니다.

---

# 21. Parameter는 중요한 입력값이다

다음처럼 URL에 값이 들어 있다고 해서:

```text
/products/10
```

`10`을 항상 신뢰해서는 안 됩니다.

사용자는 직접 주소창을 변경할 수도 있습니다.

```text
/products/abc
/products/-100
/products/999999999
```

따라서 서버 요청이나 비즈니스 로직에 사용할 경우 검증이 필요합니다.

```jsx
const { productId } = useParams();

const id = Number(productId);

if (!Number.isInteger(id) || id <= 0) {
  return <div>잘못된 상품 ID입니다.</div>;
}
```

즉:

```text
URL Parameter
      │
      ▼
useParams()
      │
      ▼
문자열
      │
      ▼
검증 / 변환
      │
      ▼
비즈니스 로직
```

순서로 생각하는 것이 좋습니다.

---

# 22. `useParams()`는 Query String을 읽지 않는다

현재 URL:

```text
/users/10?page=3
```

에서:

```jsx
const params = useParams();
```

로 얻을 수 있는 것은 Route Pattern에 정의된:

```text
:userId
```

뿐입니다.

예를 들어:

```jsx
<Route path="/users/:userId" />
```

라면:

```js
{
  userId: '10'
}
```

이 됩니다.

다음 값:

```text
?page=3
```

은 `useParams()`의 대상이 아닙니다.

Query String은:

```jsx
useSearchParams()
```

로 처리합니다.

---

# 23. `useParams()`는 Router Context 내부에서 사용해야 한다

`useParams()`는 React Router Hook이므로 Router 내부에서 사용해야 합니다.

예:

```jsx
<BrowserRouter>
  <App />
</BrowserRouter>
```

구조:

```text
BrowserRouter
      │
      ▼
Router Context
      │
      ▼
Routes
      │
      ▼
Route Match
      │
      ▼
Component
      │
      ▼
useParams()
```

즉, React Router의 현재 Route Match 정보가 존재해야 의미 있는 Parameter를 읽을 수 있습니다.

---

# 24. 전체 동작 구조

다음 Route를 다시 생각해 보겠습니다.

```jsx
<Route
  path="/users/:userId/posts/:postId"
  element={<Post />}
 />
```

현재 URL:

```text
/users/10/posts/300
```

전체 흐름은:

```text
Browser URL
/users/10/posts/300
        │
        ▼
React Router
        │
        ▼
Route Tree Matching
        │
        ▼
/users/:userId/posts/:postId
        │
        ├── userId = "10"
        └── postId = "300"
        │
        ▼
Route Match
        │
        └── params
             │
             ▼
{
  userId: "10",
  postId: "300"
}
             │
             ▼
        useParams()
             │
             ▼
      React Component
```

---

# 25. 관련 Hook 비교

| Hook                | 역할                                   |
| ------------------- | ------------------------------------ |
| `useParams()`       | 현재 Route Match의 동적 Path Parameter 읽기 |
| `useSearchParams()` | Query Parameter 읽기/변경                |
| `useLocation()`     | 현재 Location 객체 읽기                    |
| `useMatch()`        | 현재 pathname과 특정 Pattern 비교           |
| `useNavigate()`     | 코드에서 네비게이션 실행                        |
| `useResolvedPath()` | 상대 Path 해석                           |

목적별로 보면:

```text
/users/10?page=2
        │      │
        │      └── useSearchParams()
        │
        └───────── useParams()


현재 위치 전체
      ↓
useLocation()


현재 URL이 특정 Pattern과 맞는가?
      ↓
useMatch()


코드로 다른 페이지 이동
      ↓
useNavigate()
```

---

# 26. `useParams()`의 핵심

`useParams()`를 단순히:

> URL에서 값을 꺼내는 Hook

이라고 이해하면 조금 부족합니다.

더 정확하게는:

> **React Router가 현재 URL과 Route Tree를 매칭한 결과에서 `:paramName` 형태의 동적 Path Parameter들을 읽어오는 Hook**

입니다.

가장 압축하면:

```text
Route Pattern
/users/:userId
        │
        ▼

현재 URL
/users/10
        │
        ▼

React Router
Route Matching
        │
        ▼

params
{
  userId: "10"
}
        │
        ▼

useParams()
```

즉, `useParams()`의 본질은 **“URL 문자열을 직접 파싱하는 것”이 아니라 “React Router가 이미 계산한 Route Match의 Parameter를 읽는 것”**입니다.
