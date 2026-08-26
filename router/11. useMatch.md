# React Router v6의 `useMatch()`

`useMatch()`는 React Router가 제공하는 Hook으로,

> **현재 URL의 `pathname`이 특정 Path Pattern과 일치하는지 검사하고, 일치하면 매칭 정보와 Path Parameter를 반환합니다.**

쉽게 말하면 다음 두 가지를 확인하는 Hook입니다.

```text
1. 현재 URL이 이 패턴과 일치하는가?
2. 일치한다면 :param 값은 무엇인가?
```

예를 들어:

```jsx
const match = useMatch('/users/:id');
```

현재 URL이:

```text
/users/10
```

이라면 `match`는 `null`이 아니라 매칭 정보를 담은 객체가 됩니다.

```js
{
  params: {
    id: '10'
  },
  pathname: '/users/10',
  pathnameBase: '/users/10',
  pattern: {
    path: '/users/:id',
    caseSensitive: false,
    end: true
  }
}
```

반대로 현재 URL이:

```text
/products/10
```

이라면:

```js
match === null
```

이 됩니다.

---

# 1. `useMatch()`를 한 문장으로 정의하면

> `useMatch()`는 현재 Location의 `pathname`과 개발자가 지정한 Path Pattern을 비교하여, 일치하면 `PathMatch` 객체를 반환하고 일치하지 않으면 `null`을 반환하는 React Router Hook입니다.

핵심 흐름은 다음과 같습니다.

```text
현재 location.pathname
         │
         │
         ▼
     useMatch()
         │
         │ Path Pattern과 비교
         ▼
┌─────────────────────┐
│      Match ?        │
└─────────┬───────────┘
          │
      ┌───┴───┐
      │       │
     Yes      No
      │       │
      ▼       ▼
 PathMatch   null
   객체
```

---

# 2. 기본 사용법

```jsx
import { useMatch } from 'react-router-dom';

function ProductPage() {
  const match = useMatch('/products/:productId');

  console.log(match);

  return <div>Product</div>;
}
```

현재 URL이:

```text
/products/3
```

이라면:

```js
match.params.productId
```

값은:

```text
3
```

입니다.

반대로 현재 URL이:

```text
/products
```

라면:

```js
match === null
```

입니다.

---

# 3. 반환값: `PathMatch` 객체

`useMatch()`는 매칭에 성공하면 개념적으로 다음과 같은 객체를 반환합니다.

```ts
{
  params: {
    [key: string]: string | undefined
  };

  pathname: string;

  pathnameBase: string;

  pattern: {
    path: string;
    caseSensitive?: boolean;
    end?: boolean;
  };
}
```

매칭에 실패하면:

```js
null
```

을 반환합니다.

따라서 일반적으로 다음과 같이 사용합니다.

```jsx
const match = useMatch('/users/:id');

if (match) {
  console.log(match.params.id);
}
```

---

# 4. `params`

`params`에는 Path Pattern에서 `:parameter`로 선언한 값이 들어갑니다.

예:

```jsx
const match = useMatch('/users/:userId');
```

현재 URL:

```text
/users/100
```

결과:

```js
match.params
```

```js
{
  userId: '100'
}
```

즉:

```text
Pattern

/users/:userId
       └──────┘
          │
          ▼

URL

/users/100
       └─┘
        │
        ▼

params.userId = "100"
```

---

# 5. 여러 개의 Path Parameter

Path Parameter가 여러 개여도 동일하게 처리됩니다.

```jsx
const match = useMatch(
  '/users/:userId/posts/:postId'
);
```

현재 URL:

```text
/users/10/posts/300
```

결과:

```js
match.params
```

```js
{
  userId: '10',
  postId: '300'
}
```

흐름으로 보면:

```text
/users/:userId/posts/:postId
        │            │
        │            │
        ▼            ▼
       10           300

        │            │
        ▼            ▼

params.userId     params.postId
    "10"              "300"
```

---

# 6. `pathname`

`pathname`은 실제로 Pattern과 매칭된 URL 경로를 나타냅니다.

예:

```jsx
const match = useMatch('/users/:id');
```

현재 URL:

```text
/users/10
```

그러면:

```js
match.pathname
```

은:

```text
/users/10
```

입니다.

즉:

```text
Pattern
/users/:id

      ↓ match

URL
/users/10

      ↓

pathname
/users/10
```

---

# 7. `pathnameBase`

`pathnameBase`는 매칭 결과에서 **wildcard(`*`) 부분을 제외한 기본 경로**를 나타낼 때 특히 중요합니다.

예:

```jsx
const match = useMatch('/files/*');
```

현재 URL:

```text
/files/images/logo.png
```

개념적으로:

```text
pathname

/files/images/logo.png
```

반면:

```text
pathnameBase

/files
```

처럼 기본 매칭 영역을 나타낼 수 있습니다.

즉:

```text
/files/images/logo.png
└────┘ └─────────────┘
 base      splat
```

`pathnameBase`는 중첩 경로나 wildcard 기반 라우팅을 처리할 때 유용합니다.

---

# 8. `pattern`

`pattern`에는 `useMatch()`에 전달한 Path Pattern 정보가 들어 있습니다.

예:

```jsx
const match = useMatch({
  path: '/users/:id',
  caseSensitive: false,
  end: true
});
```

매칭에 성공하면:

```js
match.pattern
```

은 개념적으로 다음과 같습니다.

```js
{
  path: '/users/:id',
  caseSensitive: false,
  end: true
}
```

즉, **어떤 규칙으로 URL을 검사했는지**를 나타냅니다.

---

# 9. 문자열 Pattern과 객체 Pattern

`useMatch()`에는 문자열을 바로 전달할 수 있습니다.

```jsx
useMatch('/users/:id');
```

또는 좀 더 세밀하게 제어하려면 객체를 전달할 수 있습니다.

```jsx
useMatch({
  path: '/users/:id',
  caseSensitive: false,
  end: true
});
```

객체 Pattern의 대표적인 옵션은 다음과 같습니다.

| 옵션              | 의미                  |
| --------------- | ------------------- |
| `path`          | 검사할 경로 Pattern      |
| `caseSensitive` | 대소문자를 구분할지 여부       |
| `end`           | URL 끝까지 일치해야 하는지 여부 |

---

# 10. `end` 옵션

`end`는 **Path Pattern의 끝까지 정확하게 일치해야 하는지**를 결정합니다.

예:

```jsx
const match = useMatch({
  path: '/about',
  end: true
});
```

현재 URL이:

```text
/about
```

이라면:

```text
match
```

됩니다.

하지만:

```text
/about/team
```

이라면:

```text
null
```

입니다.

개념적으로:

```text
path="/about"
end=true

/about
└─────┘
   │
   └── Match


/about/team
└─────┘─────
   │
   └── 뒤에 경로가 더 존재
       → Not Match
```

반대로:

```jsx
useMatch({
  path: '/about',
  end: false
});
```

라면 하위 경로까지 매칭될 수 있습니다.

```text
/about
/about/team
/about/company
```

---

# 11. `caseSensitive`

기본적으로 React Router의 Path Matching은 대소문자를 구분하지 않습니다.

예:

```jsx
useMatch({
  path: '/users',
  caseSensitive: false
});
```

다음 URL은 모두 매칭될 수 있습니다.

```text
/users
/Users
/USERS
```

대소문자를 구분하고 싶다면:

```jsx
useMatch({
  path: '/users',
  caseSensitive: true
});
```

라고 설정할 수 있습니다.

이 경우:

```text
/users     → Match
/Users     → Not Match
```

처럼 동작합니다.

---

# 12. wildcard `*`

`useMatch()`는 wildcard 경로도 사용할 수 있습니다.

예:

```jsx
const match = useMatch('/dashboard/*');
```

다음과 같은 URL이 매칭될 수 있습니다.

```text
/dashboard
/dashboard/stats
/dashboard/users
/dashboard/settings/profile
```

예를 들어 현재 URL이:

```text
/dashboard/users
```

라면:

```js
match.params['*']
```

에는 wildcard가 소비한 나머지 경로가 들어갈 수 있습니다.

```text
users
```

개념적으로:

```text
Pattern

/dashboard/*
           │
           ▼

URL

/dashboard/users/profile
           └───────────┘
                 │
                 ▼

params["*"]
"users/profile"
```

---

# 13. 실전 예제 1: 현재 메뉴 활성화

```jsx
import { useMatch } from 'react-router-dom';

function DashboardMenu() {
  const match = useMatch('/dashboard/*');

  return (
    <div className={match ? 'active' : ''}>
      Dashboard
    </div>
  );
}
```

현재 URL이:

```text
/dashboard
```

또는:

```text
/dashboard/stats
/dashboard/users
```

등이라면 `match`가 존재합니다.

따라서:

```text
match !== null
       │
       ▼
className="active"
```

가 됩니다.

다만 일반적인 네비게이션 메뉴의 active 처리가 목적이라면 `<NavLink>`가 더 직접적인 선택입니다.

```jsx
<NavLink to="/dashboard">
  Dashboard
</NavLink>
```

`useMatch()`는 단순 메뉴 스타일링보다 **컴포넌트 내부에서 직접 특정 Pattern을 검사해야 할 때** 더 유용합니다.

---

# 14. 실전 예제 2: 특정 페이지인지 검사

```jsx
const match = useMatch('/users/:id');

if (match) {
  console.log('User Detail Page');
  console.log(match.params.id);
}
```

현재 URL:

```text
/users/10
```

결과:

```text
User Detail Page
10
```

흐름:

```text
현재 pathname
/users/10
      │
      ▼
Pattern
/users/:id
      │
      ▼
Match
      │
      ▼
params.id = "10"
```

---

# 15. 실전 예제 3: 조건부 UI 렌더링

```jsx
function Header() {
  const adminMatch = useMatch('/admin/*');

  return (
    <>
      {adminMatch
        ? <AdminHeader />
        : <MainHeader />}
    </>
  );
}
```

현재 URL이:

```text
/admin/users
```

이면:

```text
useMatch("/admin/*")
        │
        ▼
      Match
        │
        ▼
<AdminHeader />
```

그 외라면:

```text
null
 │
 ▼
<MainHeader />
```

가 됩니다.

---

# 16. `useMatch()`와 `useParams()`의 차이

둘 다 Path Parameter와 관련이 있지만 역할은 다릅니다.

`useParams()`는 **현재 매칭된 Route가 가진 params를 읽는 Hook**입니다.

예:

```jsx
<Route
  path="/users/:id"
  element={<User />}
/>
```

`User` 컴포넌트:

```jsx
const params = useParams();

console.log(params.id);
```

현재 URL이:

```text
/users/10
```

이라면:

```js
{
  id: '10'
}
```

을 얻을 수 있습니다.

반면 `useMatch()`는 개발자가 직접 Pattern을 전달합니다.

```jsx
const match = useMatch('/users/:id');
```

즉:

```text
useParams()

현재 매칭된 Route
       │
       ▼
    params 읽기


useMatch()

현재 pathname
       │
       │ 특정 Pattern과 비교
       ▼
 Match 여부 + params
```

---

# 17. `useMatch()`가 Route Tree와 완전히 무관한 것은 아니다

`useMatch()`를 설명할 때 다음과 같이 표현하면 조금 과도합니다.

```text
useMatch는 React Router의 라우트 구조와 완전히 무관하다.
```

보다 정확하게는:

> `useMatch()`는 현재 컴포넌트가 어떤 `<Route>`의 `path`를 가지고 있는지를 직접 사용하는 대신, 개발자가 전달한 별도의 Path Pattern을 현재 Location의 `pathname`과 비교합니다.

즉, 현재 Route에서 Parameter를 가져오는 `useParams()`와 달리:

```jsx
useMatch('/app/products/:id');
```

처럼 **원하는 Pattern을 직접 지정할 수 있다는 것**이 핵심입니다.

단, `useMatch()` 역시 Router Context 내부에서 사용해야 합니다.

---

# 18. `useParams()`와 비교 예제

현재 Route 구조:

```jsx
<Route path="/app">
  <Route
    path="products/:productId"
    element={<Product />}
  />
</Route>
```

현재 URL:

```text
/app/products/3
```

`Product` 내부에서는:

```jsx
const params = useParams();

console.log(params.productId);
```

결과:

```text
3
```

입니다.

반면 다른 컴포넌트에서 현재 URL이 특정 Pattern과 일치하는지 직접 확인하고 싶다면:

```jsx
const match =
  useMatch('/app/products/:productId');
```

라고 작성할 수 있습니다.

```text
현재 pathname
/app/products/3

       │
       ▼

Pattern
/app/products/:productId

       │
       ▼

Match

       │
       ▼

params.productId = "3"
```

---

# 19. `useLocation()`과 `useMatch()`의 차이

`useLocation()`은 현재 Location 정보를 그대로 읽습니다.

```jsx
const location = useLocation();

console.log(location.pathname);
```

결과:

```text
/users/10
```

하지만 이 URL이 `/users/:id` 패턴과 맞는지는 직접 확인해야 합니다.

반면:

```jsx
const match = useMatch('/users/:id');
```

를 사용하면 React Router의 Path Matching 기능을 이용할 수 있습니다.

정리하면:

```text
useLocation()
      │
      ▼
현재 위치를 읽는다


useMatch()
      │
      ▼
현재 pathname을
특정 Pattern과 비교한다
```

---

# 20. `useMatch()`는 Query String을 검사하지 않는다

매우 중요한 특징입니다.

현재 URL이:

```text
/users/10?page=3
```

이라고 하겠습니다.

```jsx
useMatch('/users/:id');
```

는:

```text
/users/10
```

이라는 `pathname`을 기준으로 매칭합니다.

즉:

```text
URL

/users/10?page=3
└───────┘ └──────┘
 pathname   search
    │
    ▼
 useMatch()
```

`?page=3`은 Path Matching 대상이 아닙니다.

Query Parameter를 읽으려면:

```jsx
useSearchParams();
```

를 사용합니다.

---

# 21. hash 역시 Path Matching 대상이 아니다

현재 URL:

```text
/users/10#profile
```

에서도 `useMatch()`가 검사하는 핵심 대상은:

```text
/users/10
```

입니다.

```text
/users/10#profile
└───────┘└───────┘
 pathname   hash
    │
    ▼
 useMatch()
```

Hash를 읽으려면:

```jsx
const location = useLocation();

console.log(location.hash);
```

를 사용할 수 있습니다.

---

# 22. 내부 동작을 개념적으로 보면

다음 코드가 있다고 하겠습니다.

```jsx
const match = useMatch('/users/:id');
```

현재 URL:

```text
/users/10
```

React Router의 동작을 개념적으로 단순화하면 다음과 같습니다.

```text
현재 Location
     │
     ▼
pathname 추출
     │
     ▼
 /users/10
     │
     │
     │ Pattern
     │ /users/:id
     ▼
Path Matching
     │
     ├── "users" 비교
     │
     └── ":id" → "10"
              │
              ▼
        params 생성
              │
              ▼
         PathMatch
```

결과:

```js
{
  params: {
    id: '10'
  },
  pathname: '/users/10',
  pathnameBase: '/users/10',
  pattern: {
    path: '/users/:id',
    caseSensitive: false,
    end: true
  }
}
```

---

# 23. 내부적으로 단순 정규식 비교만 하는 것은 아니다

`useMatch()`를 설명하면서:

```text
path-to-regexp 스타일의 정규식 엔진
```

이라고 설명할 수는 있지만, React Router가 `path-to-regexp` 라이브러리를 그대로 사용한다고 이해하면 안 됩니다.

React Router는 자체적인 Path Matching 로직을 가지고 있으며 Pattern을 해석하여 현재 `pathname`과 비교합니다.

개념적으로는:

```text
Path Pattern
/users/:id
      │
      ▼
Pattern 해석
      │
      ▼
pathname과 비교
      │
      ▼
Parameter 추출
      │
      ▼
PathMatch 반환
```

정도로 이해하는 것이 적절합니다.

---

# 24. `useMatch()`는 Router 내부에서 사용해야 한다

다음과 같은 구조가 있어야 합니다.

```jsx
<BrowserRouter>
  <App />
</BrowserRouter>
```

그리고 하위 컴포넌트에서:

```jsx
function App() {
  const match = useMatch('/users/:id');

  // ...
}
```

처럼 사용할 수 있습니다.

개념적으로:

```text
BrowserRouter
      │
      │ Router Context
      ▼
     App
      │
      ▼
  useMatch()
```

Router Context 밖에서 `useMatch()`를 사용하는 것은 올바른 사용 방식이 아닙니다.

---

# 25. `useMatch()`와 `<NavLink>`의 관계

`<NavLink>`도 내부적으로 현재 URL과 자신의 경로를 비교하여 active 상태를 판단합니다.

따라서 개념적으로 두 기능은 관련이 있습니다.

```text
현재 pathname
      │
      ├───────────────┐
      ▼               ▼
   useMatch()      <NavLink>
      │               │
Pattern 직접 검사   자신의 to와 비교
      │               │
      ▼               ▼
PathMatch/null      isActive
```

따라서:

```text
네비게이션 메뉴 active 표시
        ↓
     <NavLink>


컴포넌트 로직에서
특정 Path Pattern 검사
        ↓
     useMatch()
```

처럼 구분하면 좋습니다.

---

# 26. `useResolvedPath()`와도 역할이 다르다

`useResolvedPath()`는 Path Matching을 하는 Hook이 아닙니다.

예를 들어 현재 위치가:

```text
/users/10
```

이고 상대경로:

```text
settings
```

가 있다면:

```jsx
const path =
  useResolvedPath('settings');
```

를 사용하여 상대 경로를 현재 라우팅 문맥에 맞는 Path로 해석할 수 있습니다.

반면:

```jsx
useMatch('/users/:id');
```

는 현재 URL이 특정 Pattern과 일치하는지를 검사합니다.

```text
useResolvedPath()
       │
       ▼
 상대 경로 해석


useMatch()
       │
       ▼
 Path Pattern 비교
```

---

# 27. 관련 Hook 비교

| Hook                | 역할                                       |
| ------------------- | ---------------------------------------- |
| `useMatch()`        | 현재 `pathname`이 특정 Path Pattern과 일치하는지 검사 |
| `useParams()`       | 현재 매칭된 Route의 Path Parameter 읽기          |
| `useLocation()`     | 현재 `Location` 객체 읽기                      |
| `useSearchParams()` | Query Parameter 읽기/변경                    |
| `useNavigate()`     | Programmatic Navigation 수행               |
| `useResolvedPath()` | 상대 Path를 현재 Router 문맥에 맞게 해석             |

조금 더 직관적으로 보면:

```text
현재 위치 자체가 궁금하다
        ↓
   useLocation()


현재 Route의 :param이 궁금하다
        ↓
     useParams()


현재 URL이 특정 패턴과 맞는가?
        ↓
      useMatch()


쿼리스트링을 다루고 싶다
        ↓
 useSearchParams()


코드로 이동하고 싶다
        ↓
    useNavigate()
```

---

# 28. 주의할 점

첫째, `useMatch()`는 URL 전체 문자열을 비교하는 Hook이 아닙니다.

핵심 비교 대상은 현재 Location의 **`pathname`**입니다.

따라서:

```text
/users/10?page=3#profile
```

에서:

```text
/users/10
```

부분을 Path Pattern과 비교합니다.

둘째, 메뉴의 active 스타일링만 필요하다면 `useMatch()`를 직접 사용하는 것보다 `<NavLink>`가 더 적합할 수 있습니다.

셋째, 현재 Route의 Parameter만 필요한 것이라면 `useParams()`가 더 직접적인 API입니다.

`useMatch()`는 **특정 Pattern과 현재 URL의 관계 자체를 검사해야 하는 상황**에서 사용하는 것이 가장 자연스럽습니다.

---

# 29. 전체 구조

`useMatch()`의 전체 동작을 정리하면 다음과 같습니다.

```text
Browser URL
     │
     ▼
React Router
     │
     ▼
Current Location
     │
     ▼
location.pathname
     │
     │
     │   Path Pattern
     │  "/users/:id"
     │
     ▼
   useMatch()
     │
     ▼
 Path Matching
     │
 ┌───┴───────────┐
 │               │
Match          No Match
 │               │
 ▼               ▼
PathMatch        null
 │
 ├── params
 ├── pathname
 ├── pathnameBase
 └── pattern
```

---

# 30. `useMatch()`의 핵심

`useMatch()`를 단순히:

> URL이 같은지 확인하는 Hook

이라고 이해하면 부족합니다.

더 정확하게는:

> **현재 React Router Location의 `pathname`과 개발자가 지정한 Path Pattern을 React Router의 Path Matching 규칙으로 비교하고, 일치하면 매칭된 경로와 Path Parameter 정보를 담은 `PathMatch` 객체를 반환하는 Hook**

입니다.

가장 압축하면:

```text
현재 pathname
      +
 Path Pattern
      │
      ▼
   useMatch()
      │
      ▼
 match ?
   │
   ├── Yes → PathMatch
   │          └── params
   │
   └── No  → null
```

즉, `useMatch()`의 본질은 **“현재 URL을 읽는 것”이 아니라 “현재 URL의 경로를 특정 라우팅 패턴과 비교하는 것”**입니다.
