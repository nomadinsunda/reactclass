# React Router v6의 `useLocation()`

`useLocation()`은 React Router가 관리하고 있는 **현재 `Location` 객체를 읽어오는 Hook**입니다.

```jsx
import { useLocation } from 'react-router-dom';

const location = useLocation();
```

여기서 말하는 `Location`은 단순히 주소창의 URL 문자열 하나가 아닙니다.

현재 라우팅 위치를 표현하는 다음과 같은 정보가 포함되어 있습니다.

```text
Location
├── pathname
├── search
├── hash
├── state
└── key
```

따라서 `useLocation()`을 사용하면 현재 경로뿐 아니라 쿼리스트링, 해시, 네비게이션 과정에서 전달된 `state` 등의 정보를 컴포넌트에서 읽을 수 있습니다.

---

# 1. `useLocation()`을 한 문장으로 정의하면

> `useLocation()`은 React Router가 현재 네비게이션 위치를 표현하기 위해 관리하는 `Location` 객체를 컴포넌트에서 읽을 수 있도록 해주는 Hook입니다.

예를 들어 브라우저의 현재 주소가 다음과 같다고 하겠습니다.

```text
https://example.com/users/10?page=2#profile
```

React Router에서:

```jsx
const location = useLocation();

console.log(location);
```

하면 개념적으로 다음과 같은 객체를 얻습니다.

```js
{
  pathname: '/users/10',
  search: '?page=2',
  hash: '#profile',
  state: null,
  key: 'abc123'
}
```

즉:

```text
/users/10?page=2#profile
│        │       │
│        │       └── hash
│        │
│        └────────── search
│
└─────────────────── pathname
```

---

# 2. `useLocation()`이 반환하는 값

`useLocation()`은 현재 `Location` 객체를 반환합니다.

개념적으로 다음과 같은 구조입니다.

```ts
{
  pathname: string;
  search: string;
  hash: string;
  state: any;
  key: string;
}
```

각 프로퍼티의 역할은 다음과 같습니다.

| 프로퍼티       | 의미                         | 예                  |
| ---------- | -------------------------- | ------------------ |
| `pathname` | 현재 URL의 경로                 | `/users/10`        |
| `search`   | 쿼리스트링                      | `?page=2`          |
| `hash`     | URL의 hash 부분               | `#profile`         |
| `state`    | History Entry에 연결된 사용자 데이터 | `{ from: 'home' }` |
| `key`      | 현재 Location Entry를 식별하는 키  | `"abc123"`         |

---

# 3. `pathname`

`pathname`은 현재 URL에서 **경로 부분**을 나타냅니다.

예를 들어:

```text
https://example.com/users/10?page=2#profile
                    └───────┘
                     pathname
```

값은:

```js
location.pathname
```

```text
/users/10
```

입니다.

다른 예:

```text
URL                              pathname

https://example.com/             /
https://example.com/about        /about
https://example.com/users/10     /users/10
https://example.com/products/1   /products/1
```

사용 예:

```jsx
const location = useLocation();

console.log(location.pathname);
```

---

# 4. `search`

`search`는 URL에서 `?`로 시작하는 **쿼리스트링 전체 문자열**입니다.

예:

```text
/users?page=2&sort=name
      └────────────────┘
             search
```

```jsx
const location = useLocation();

console.log(location.search);
```

결과:

```text
?page=2&sort=name
```

중요한 점은 `search`가 파싱된 객체가 아니라 **문자열**이라는 것입니다.

따라서:

```js
location.search.page
```

처럼 사용할 수 없습니다.

React Router에서는 쿼리 파라미터를 다룰 때 일반적으로 `useSearchParams()`를 사용합니다.

```jsx
const [searchParams] = useSearchParams();

const page = searchParams.get('page');
```

---

# 5. `hash`

`hash`는 URL의 `#` 이후 부분을 나타냅니다.

예를 들어:

```text
/docs/react#hooks
           └────┘
             hash
```

`location.hash`의 값은:

```text
#hooks
```

입니다.

사용 예:

```jsx
const location = useLocation();

console.log(location.hash);
```

Hash는 특정 섹션을 식별하거나, UI 상태를 URL에 표현하는 등의 목적으로 사용할 수 있습니다.

다만 한 가지 주의해야 합니다.

> `location.hash`가 있다고 해서 React Router가 항상 해당 DOM 엘리먼트까지 자동으로 스크롤해 주는 것은 아닙니다.

필요한 경우 애플리케이션에서 별도의 스크롤 처리를 구현할 수 있습니다.

---

# 6. `state`

`state`는 `pathname`, `search`, `hash`와 조금 성격이 다릅니다.

URL 문자열에 포함되지 않는 **네비게이션 상태 데이터**입니다.

예를 들어:

```jsx
navigate('/product/10', {
  state: {
    from: 'search',
    keyword: 'keyboard'
  }
});
```

이렇게 이동했다면 `/product/10` 컴포넌트에서는:

```jsx
const location = useLocation();

console.log(location.state);
```

다음 데이터를 읽을 수 있습니다.

```js
{
  from: 'search',
  keyword: 'keyboard'
}
```

브라우저 주소창에는:

```text
/product/10
```

만 나타납니다.

즉:

```text
URL
/product/10
     │
     └── state는 URL에 보이지 않음
```

개념적으로는:

```text
History Entry
│
├── URL
│   └── /product/10
│
└── state
    ├── from: "search"
    └── keyword: "keyboard"
```

와 같습니다.

---

# 7. `state`는 어디에 저장되는가?

React Router의 `location.state`는 브라우저의 **History Entry에 연결된 `history.state`를 기반으로 관리**됩니다.

따라서:

```jsx
navigate('/detail/10', {
  state: {
    from: 'list'
  }
});
```

는 개념적으로:

```text
Browser History
       │
       ▼
History Entry
├── URL   : /detail/10
└── state : { from: "list" }
```

와 같은 구조를 가집니다.

중요한 점은:

> `state`는 React 컴포넌트의 일반적인 `useState` 상태가 아닙니다.

그리고 URL에도 직접 포함되지 않습니다.

---

# 8. `state`와 새로고침에 대한 주의

다음 설명은 정확하지 않습니다.

```text
F5를 누르면 location.state는 무조건 사라진다.
```

`location.state`는 브라우저의 `history.state`를 기반으로 하기 때문에 **같은 History Entry를 새로고침하는 경우 유지될 수 있습니다.**

그러나 그렇다고 중요한 데이터를 `location.state`에 저장하는 것도 적절하지 않습니다.

왜냐하면:

```text
location.state
      │
      ├── URL에 포함되지 않음
      ├── 다른 사람에게 URL만 전달할 수 없음
      ├── 직접 URL로 진입하면 존재하지 않을 수 있음
      └── 서버에서는 접근할 수 없음
```

때문입니다.

따라서:

```text
페이지를 식별하는 중요한 데이터
        ↓
URL Path / Query


네비게이션 과정의 부가 정보
        ↓
location.state
```

처럼 구분하는 것이 좋습니다.

---

# 9. `key`

`key`는 현재 Location Entry를 식별하기 위한 값입니다.

예를 들어:

```js
{
  pathname: '/about',
  search: '',
  hash: '',
  state: null,
  key: 'x93ks2'
}
```

여기서:

```text
key = "x93ks2"
```

입니다.

라우팅이 발생하여 새로운 History Entry가 생성되면 새로운 `key`가 부여될 수 있습니다.

```text
History Stack

/home
key = "a1"

      ↓ push

/about
key = "b7"

      ↓ push

/contact
key = "c3"
```

따라서 같은 URL이라도 서로 다른 History Entry라면 다른 `key`를 가질 수 있습니다.

일반적인 애플리케이션 코드에서 직접 사용할 일은 많지 않지만, 페이지 전환 효과나 Location Entry를 구분해야 하는 경우 활용할 수 있습니다.

---

# 10. `useLocation()`은 BrowserRouter와 어떤 관계인가?

`useLocation()`을 이해하려면 `BrowserRouter`와의 관계를 보는 것이 중요합니다.

일반적인 애플리케이션 구조는 다음과 같습니다.

```jsx
<BrowserRouter>
  <App />
</BrowserRouter>
```

`BrowserRouter`는 현재 location을 관리하고 Router Context를 통해 하위 컴포넌트에 제공합니다.

개념적으로:

```text
Browser
   │
   │ URL / History
   ▼
BrowserRouter
   │
   │ 현재 Location 관리
   ▼
Router Context
   │
   ▼
React Component
   │
   ▼
useLocation()
   │
   ▼
현재 Location 객체
```

따라서 `useLocation()`은 브라우저 주소창을 직접 읽어 오는 함수라기보다,

> **React Router가 관리하고 Context를 통해 제공하는 현재 Location을 읽는 Hook**

이라고 이해하는 것이 더 정확합니다.

---

# 11. URL이 변경되면 어떤 일이 일어나는가?

예를 들어 현재 경로가:

```text
/home
```

이라고 하겠습니다.

사용자가 다음 링크를 클릭합니다.

```jsx
<Link to="/about">
  About
</Link>
```

전체적인 흐름은 다음과 같습니다.

```text
사용자가 <Link> 클릭
        │
        ▼
React Router Navigation
        │
        ▼
Browser History 변경
        │
        ▼
현재 Location 변경
        │
        ▼
Router Context의 값 변경
        │
        ▼
새로운 경로에 맞게 Route Matching
        │
        ▼
관련 React UI 렌더링
```

그리고 `useLocation()`을 사용하는 컴포넌트에서는 새로운 `Location` 값을 읽을 수 있습니다.

```text
Before

location.pathname
        ↓
      /home


<Link to="/about">

        ↓


After

location.pathname
        ↓
      /about
```

---

# 12. 뒤로 가기와 `useLocation()`

`useLocation()`은 `<Link>`나 `navigate()`에 의한 이동뿐 아니라 브라우저의 History 이동에도 반응합니다.

예를 들어 History가:

```text
/home
/about
/products   ← 현재
```

라고 하겠습니다.

사용자가 브라우저의 뒤로 가기를 실행합니다.

```text
/home
/about      ← 현재
/products
```

React Router는 History 변경을 감지하고 현재 Location을 갱신합니다.

```text
Browser 뒤로 가기
       │
       ▼
History Entry 변경
       │
       ▼
React Router
       │
       ▼
현재 Location 변경
       │
       ▼
useLocation()
       │
       ▼
pathname = "/about"
```

---

# 13. `useLocation()`의 내부 흐름

전체적인 구조를 단순화하면 다음과 같습니다.

```text
Browser
│
│ URL / History
│
▼
BrowserRouter
│
│ 현재 Location 관리
│
▼
Router Context
│
│
▼
useLocation()
│
▼
Location
├── pathname
├── search
├── hash
├── state
└── key
│
▼
React Component
```

따라서 핵심은:

```text
Browser의 현재 Navigation 위치
              │
              ▼
         React Router
              │
              ▼
          Location
              │
              ▼
        useLocation()
              │
              ▼
         Component
```

입니다.

---

# 14. 기본 사용 예제

```jsx
import { useLocation } from 'react-router-dom';

function LocationDemo() {
  const location = useLocation();

  return (
    <div>
      <p>pathname: {location.pathname}</p>
      <p>search: {location.search}</p>
      <p>hash: {location.hash}</p>
      <p>state: {JSON.stringify(location.state)}</p>
      <p>key: {location.key}</p>
    </div>
  );
}

export default LocationDemo;
```

현재 주소가:

```text
/users/10?page=2#profile
```

이라면 대략 다음과 같이 나타납니다.

```text
pathname : /users/10
search   : ?page=2
hash     : #profile
state    : null
key      : xxxxx
```

---

# 15. 활용 패턴 1: 경로에 따라 UI 변경

현재 경로에 따라 다른 UI를 보여줄 수 있습니다.

```jsx
const { pathname } = useLocation();

return (
  <div>
    {pathname.startsWith('/admin')
      ? <AdminNav />
      : <MainNav />}
  </div>
);
```

개념적으로:

```text
location.pathname
       │
       ├── /admin/...  → <AdminNav />
       │
       └── 그 외       → <MainNav />
```

다만 Route 자체를 구분하기 위해 지나치게 많은 `pathname` 조건문을 작성한다면 `<Routes>`와 `<Route>` 구조로 해결할 수 있는지 먼저 검토하는 것이 좋습니다.

---

# 16. 활용 패턴 2: 이전 페이지 정보 전달

목록에서 상세 페이지로 이동한다고 하겠습니다.

```jsx
navigate('/product/10', {
  state: {
    from: '/products'
  }
});
```

상세 페이지:

```jsx
const location = useLocation();

console.log(location.state?.from);
```

결과:

```text
/products
```

이를 이용하면:

```text
상품 목록
    │
    │ state = { from: "/products" }
    ▼
상품 상세
    │
    │ location.state.from
    ▼
어디에서 들어왔는지 확인
```

과 같은 패턴을 구현할 수 있습니다.

---

# 17. 활용 패턴 3: 이전 검색 조건 전달

검색 결과에서 상세 페이지로 이동한다고 하겠습니다.

```jsx
navigate('/product/10', {
  state: {
    from: 'search',
    keyword: 'keyboard'
  }
});
```

상세 페이지:

```jsx
const { state } = useLocation();

console.log(state?.from);
console.log(state?.keyword);
```

결과:

```text
search
keyboard
```

이처럼 `state`는 URL에 표현할 필요가 없는 **네비게이션 과정의 부가 정보**를 전달하는 데 사용할 수 있습니다.

---

# 18. 활용 패턴 4: Location 변경 감지

`useLocation()`은 현재 위치가 변경될 때 Side Effect를 실행하는 용도로도 사용할 수 있습니다.

```jsx
const location = useLocation();

useEffect(() => {
  console.log('페이지 이동:', location.pathname);
}, [location]);
```

흐름:

```text
Location 변경
      │
      ▼
Component 렌더링
      │
      ▼
useEffect 실행
      │
      ▼
페이지 이동 관련 작업
```

대표적으로 페이지뷰 분석이나 스크롤 처리 등을 구현할 때 활용할 수 있습니다.

---

# 19. 활용 패턴 5: 페이지 전환 애니메이션

Location을 페이지 전환 애니메이션의 기준으로 사용할 수도 있습니다.

예:

```jsx
const location = useLocation();

<AnimatePresence mode="wait">
  <motion.div key={location.pathname}>
    <Routes location={location}>
      {/* routes */}
    </Routes>
  </motion.div>
</AnimatePresence>
```

경로가 변경되면:

```text
/home
  │
  │ pathname 변경
  ▼
/about
```

`key`가 달라지고 React가 서로 다른 엘리먼트로 판단할 수 있으므로 페이지 전환 애니메이션을 구성할 수 있습니다.

상황에 따라 `pathname` 대신 `location.key`를 사용할 수도 있습니다.

---

# 20. `useLocation()`은 URL을 변경하는 Hook이 아니다

매우 중요한 구분입니다.

`useLocation()`은 **읽기용 Hook**입니다.

```jsx
const location = useLocation();
```

현재 위치 정보를 읽는 것이 목적입니다.

따라서 다음처럼 생각하면 안 됩니다.

```jsx
location.pathname = '/about';
```

이런 방식으로 라우팅을 변경하는 것이 아닙니다.

URL을 변경하려면:

```jsx
const navigate = useNavigate();

navigate('/about');
```

또는:

```jsx
<Link to="/about">
  About
</Link>
```

을 사용합니다.

정리하면:

```text
현재 Location 읽기
       ↓
 useLocation()


Location 변경
       ↓
 useNavigate()
 <Link>
 <NavLink>
```

---

# 21. `useLocation()`과 `useNavigate()`의 차이

두 Hook을 함께 이해하면 React Router의 구조가 훨씬 명확해집니다.

```text
useLocation()
     ↓
현재 위치 읽기


useNavigate()
     ↓
새 위치로 이동
```

| Hook                | 역할                          |
| ------------------- | --------------------------- |
| `useLocation()`     | 현재 Location 읽기              |
| `useNavigate()`     | 새로운 Location으로 이동           |
| `useSearchParams()` | 쿼리 파라미터 읽기/변경               |
| `useParams()`       | Route의 동적 Path Parameter 읽기 |

예를 들어:

```jsx
function Example() {
  const location = useLocation();
  const navigate = useNavigate();

  return (
    <>
      <p>{location.pathname}</p>

      <button onClick={() => navigate('/about')}>
        About
      </button>
    </>
  );
}
```

---

# 22. `useLocation()`과 `useSearchParams()` 비교

둘은 비슷해 보이지만 역할이 다릅니다.

| 기능            | `useLocation()` | `useSearchParams()` |
| ------------- | --------------- | ------------------- |
| `pathname` 읽기 | 가능              | 불가능                 |
| `search` 읽기   | 문자열로 가능         | 파싱된 형태로 사용 가능       |
| `hash` 읽기     | 가능              | 불가능                 |
| `state` 읽기    | 가능              | 불가능                 |
| `key` 읽기      | 가능              | 불가능                 |
| 쿼리 파라미터 수정    | 직접 목적 아님        | 가능                  |

예:

```text
/users?page=2&sort=name
```

`useLocation()`:

```jsx
const location = useLocation();

location.search;
```

결과:

```text
?page=2&sort=name
```

`useSearchParams()`:

```jsx
const [searchParams] = useSearchParams();

searchParams.get('page');
```

결과:

```text
2
```

즉:

```text
useLocation()
     ↓
현재 Location 전체 구조 확인


useSearchParams()
     ↓
Query Parameter 전문 처리
```

---

# 23. `useLocation()`과 `useParams()`도 다르다

예를 들어 Route가:

```jsx
<Route
  path="/users/:id"
  element={<User />}
/>
```

이고 현재 URL이:

```text
/users/10
```

이라면:

```jsx
const location = useLocation();
```

결과:

```js
location.pathname === '/users/10'
```

반면:

```jsx
const params = useParams();
```

결과:

```js
{
  id: '10'
}
```

즉:

```text
/users/10
│      │
│      └── useParams() → id = "10"
│
└───────── useLocation() → pathname = "/users/10"
```

역할이 완전히 다릅니다.

---

# 24. `window.location`과 `useLocation()`의 차이

이 부분도 매우 중요합니다.

브라우저에는 원래:

```js
window.location
```

객체가 존재합니다.

그렇다면 왜 React Router의 `useLocation()`을 사용할까요?

`window.location`은 **브라우저가 관리하는 Location**입니다.

반면:

```jsx
useLocation()
```

은 **React Router가 현재 라우팅 상태로 관리하고 있는 Location을 React 컴포넌트에서 읽기 위한 API**입니다.

개념적으로:

```text
Browser
   │
   └── window.location


React Router
   │
   └── Location
          │
          └── useLocation()
```

React Router 기반 컴포넌트에서는 라우팅 상태에 반응하여 UI를 렌더링해야 하므로 일반적으로 `useLocation()`을 사용합니다.

---

# 25. 리렌더링에 대한 정확한 이해

다음과 같이 단순하게 이해하면 약간 부정확합니다.

```text
URL이 바뀌면 모든 컴포넌트가 다시 렌더링된다.
```

React Router에서 Location이 변경되면 Router가 새로운 라우팅 상태를 제공하고, 이에 따라 **관련 라우트와 해당 Context 값을 사용하는 컴포넌트들이 새로운 상태를 기준으로 렌더링될 수 있습니다.**

따라서 핵심은:

```text
Location 변경
      │
      ▼
Router 상태 변경
      │
      ▼
새 Route Matching
      │
      ▼
관련 React UI 업데이트
```

입니다.

`useLocation()`은 이러한 Router의 Location Context를 읽기 때문에 현재 위치가 변경되면 새로운 Location 값을 받게 됩니다.

---

# 26. 전체 구조

지금까지의 내용을 하나의 흐름으로 정리하면 다음과 같습니다.

```text
Browser URL / History
          │
          ▼
    BrowserRouter
          │
          │ 현재 Location 관리
          ▼
    Router Context
          │
          ▼
     useLocation()
          │
          ▼
┌─────────────────────────┐
│        Location         │
│                         │
│ pathname                │
│ search                  │
│ hash                    │
│ state                   │
│ key                     │
└────────────┬────────────┘
             │
             ▼
      React Component
             │
             ▼
        UI Rendering
```

---

# 27. `useLocation()`을 이해할 때 가장 중요한 개념

`useLocation()`을 단순히:

> 현재 URL을 가져오는 Hook

이라고 기억하는 것보다 다음과 같이 이해하는 것이 더 정확합니다.

> `useLocation()`은 React Router가 현재 네비게이션 위치를 나타내기 위해 관리하고 있는 `Location` 객체를 React 컴포넌트에서 읽을 수 있도록 해주는 Hook입니다.

즉:

```text
URL
 ↓
Browser History
 ↓
React Router
 ↓
Location
 ↓
useLocation()
 ↓
React Component
```

라는 관계입니다.

---

# 핵심 정리

| 항목              | 설명                                         |
| --------------- | ------------------------------------------ |
| `useLocation()` | 현재 React Router `Location` 객체를 반환          |
| `pathname`      | 현재 URL의 경로                                 |
| `search`        | `?`부터 시작하는 쿼리스트링                           |
| `hash`          | `#`부터 시작하는 Hash                            |
| `state`         | 현재 History Entry에 연결된 사용자 정의 상태            |
| `key`           | Location Entry를 구분하는 식별값                   |
| 읽기/쓰기           | 기본적으로 Location을 읽는 용도                      |
| URL 변경          | `useNavigate`, `<Link>`, `<NavLink>` 등을 사용 |
| Router 관계       | Router Context를 통해 현재 Location을 얻음         |

가장 압축해서 표현하면:

```text
Browser
   │
   │ URL / History
   ▼
React Router
   │
   │ Location 관리
   ▼
useLocation()
   │
   ▼
현재 Location 읽기
   │
   ├── pathname
   ├── search
   ├── hash
   ├── state
   └── key
```

`useLocation()`의 핵심은 **“주소창의 문자열을 읽는다”가 아니라 “React Router가 현재 위치로 관리하고 있는 Location 상태를 읽는다”**는 것입니다.
