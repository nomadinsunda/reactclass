# React Router v6의 `useResolvedPath()`

`useResolvedPath()`는 React Router에서 **주어진 경로를 현재 라우팅 문맥을 기준으로 해석하여 최종 `Path` 객체를 계산하는 Hook**입니다.

예를 들어 다음과 같은 상대 경로가 있다고 하겠습니다.

```text id="4hz55e"
profile
../settings
../../dashboard
```

이 문자열만 보면 실제로 어느 URL을 의미하는지 알 수 없습니다.

현재 어떤 Route 안에 있는지에 따라 결과가 달라질 수 있기 때문입니다.

`useResolvedPath()`는 React Router가 가지고 있는 현재 Route Context와 Location 정보를 이용하여 이러한 상대 경로를 실제로 사용할 수 있는 Path로 계산합니다.

```text id="q0snyn"
상대 경로
"../settings"
      │
      ▼
useResolvedPath()
      │
      │ 현재 Router Context에서 해석
      ▼
{
  pathname: "...",
  search: "",
  hash: ""
}
```

---

# 1. `useResolvedPath()`를 한 문장으로 정의하면

> `useResolvedPath()`는 React Router의 현재 라우팅 문맥을 기준으로 전달된 `to` 값을 해석하여 최종 `pathname`, `search`, `hash`를 가진 `Path` 객체를 반환하는 Hook입니다.

즉, 단순히 문자열을 이어 붙이는 함수가 아닙니다.

```text id="5v3sdv"
to
"../settings"
      │
      ▼
현재 Route Context
      +
현재 Location
      │
      ▼
useResolvedPath()
      │
      ▼
Resolved Path
```

---

# 2. 왜 필요한가?

절대 경로는 그 자체만으로 목적지가 명확합니다.

```text id="zhfkq1"
/users
/users/10
/users/10/profile
```

하지만 상대 경로는 다릅니다.

```text id="ii1jyn"
profile
../settings
..
```

예를 들어:

```jsx id="z1vyfb"
<Link to="../settings">
  Settings
</Link>
```

의 `../settings`가 실제로 어느 URL을 의미하는지는 **현재 Route의 위치와 Route 계층 구조**에 따라 결정됩니다.

React Router는 `<Link>`, `<NavLink>`, `navigate()` 등을 사용할 때 이러한 상대 경로를 자동으로 해석합니다.

`useResolvedPath()`를 사용하면 그 계산 결과를 직접 얻을 수 있습니다.

---

# 3. 기본 사용법

```jsx id="usde21"
import { useResolvedPath } from 'react-router-dom';

function Example() {
  const path = useResolvedPath('profile');

  console.log(path);

  return <div>Example</div>;
}
```

반환값은 다음과 같은 `Path` 객체입니다.

```js id="bxdp0p"
{
  pathname: '/users/10/profile',
  search: '',
  hash: ''
}
```

즉:

```text id="q1z3ry"
useResolvedPath("profile")
          │
          ▼
{
  pathname,
  search,
  hash
}
```

형태입니다.

---

# 4. 반환값

`useResolvedPath()`의 반환값은 개념적으로 다음과 같습니다.

```ts id="ba2eak"
{
  pathname: string;
  search: string;
  hash: string;
}
```

예를 들어:

```jsx id="qe5w0j"
const resolved = useResolvedPath(
  'profile?tab=info#account'
);
```

결과는 상황에 따라 다음과 같은 형태가 될 수 있습니다.

```js id="c2xu80"
{
  pathname: '/users/10/profile',
  search: '?tab=info',
  hash: '#account'
}
```

따라서 `useResolvedPath()`는 단순히 `pathname`만 계산하는 Hook이 아닙니다.

```text id="10n1jk"
Resolved Path
│
├── pathname
├── search
└── hash
```

를 반환합니다.

---

# 5. `useLocation()`의 반환값과 차이

두 Hook의 반환 객체는 일부 구조가 비슷합니다.

`useLocation()`:

```js id="11z225"
{
  pathname,
  search,
  hash,
  state,
  key
}
```

`useResolvedPath()`:

```js id="m93ceo"
{
  pathname,
  search,
  hash
}
```

하지만 목적은 완전히 다릅니다.

```text id="rlorco"
useLocation()
      │
      ▼
현재 위치를 읽는다


useResolvedPath()
      │
      ▼
어떤 목적지가
현재 문맥에서 어디인지 계산한다
```

즉:

> `useLocation()`은 현재 위치이고, `useResolvedPath()`는 목적지 후보를 해석한 결과입니다.

---

# 6. 절대 경로도 사용할 수 있다

`useResolvedPath()`에는 상대 경로뿐 아니라 절대 경로도 전달할 수 있습니다.

```jsx id="11f9yx"
const path = useResolvedPath('/about');
```

결과:

```js id="q815xd"
{
  pathname: '/about',
  search: '',
  hash: ''
}
```

따라서 `useResolvedPath()`의 본질을:

> 상대 경로를 절대 경로로 변환한다.

라고만 설명하면 조금 부족합니다.

더 정확하게는:

> **전달된 `to` 값을 React Router의 현재 라우팅 문맥에서 해석한다.**

입니다.

---

# 7. 상대 경로

`useResolvedPath()`가 특히 중요한 경우는 상대 경로입니다.

예:

```jsx id="nqpj0n"
const path = useResolvedPath('profile');
```

또는:

```jsx id="edg885"
const path = useResolvedPath('../settings');
```

상대 경로는 다음과 같은 React Router 기능에서 공통적으로 사용됩니다.

```jsx id="b2rvdp"
<Link to="../settings" />

<NavLink to="../settings" />

navigate('../settings');

useResolvedPath('../settings');
```

즉, React Router에서 상대 경로 해석은 특정 Hook 하나에만 존재하는 개념이 아닙니다.

```text id="q7g44e"
Relative To
   "../settings"
         │
         ▼
React Router의
Path Resolution
         │
   ┌─────┼──────────┐
   ▼     ▼          ▼
 Link  navigate  useResolvedPath
```

라고 이해할 수 있습니다.

---

# 8. 가장 중요한 개념: 상대 경로의 기준

여기서 가장 주의해야 할 부분입니다.

다음과 같이 단순하게 설명하면 정확하지 않습니다.

```text id="1jdwbq"
상대 경로는 현재 브라우저 URL을 기준으로 계산한다.
```

React Router의 상대 경로는 기본적으로 **현재 Route 계층 관계를 기준으로 해석**됩니다.

즉:

```text id="2whpbj"
Browser URL만 보는 것이 아니라

현재 매칭된 Route hierarchy
          │
          ▼
    상대 경로 해석
```

이 핵심입니다.

---

# 9. Route hierarchy란?

다음과 같은 Route를 생각해 봅시다.

```jsx id="56rxqr"
<Route path="users">
  <Route path=":userId">
    <Route path="profile" element={<Profile />} />
    <Route path="settings" element={<Settings />} />
  </Route>
</Route>
```

Route Tree는 다음과 같습니다.

```text id="0rjvhh"
users
  │
  └── :userId
        │
        ├── profile
        │
        └── settings
```

현재 `profile` Route가 매칭되어 있다고 하겠습니다.

```text id="19oixm"
/users/10/profile
```

이 상태에서:

```jsx id="c54ruv"
useResolvedPath('../settings');
```

와 같은 상대 경로를 사용하면 React Router가 현재 Route 관계를 이용하여 목적지를 해석합니다.

개념적으로:

```text id="l3shrk"
profile Route
      │
      │ ..
      ▼
:userId Route
      │
      │ settings
      ▼
settings Route
```

와 같은 관점으로 이해하는 것이 좋습니다.

---

# 10. `..`는 항상 URL 세그먼트 하나를 제거한다는 뜻이 아니다

이 부분도 중요합니다.

일반 파일 시스템에서는:

```text id="j91ff6"
a/b/c
  │
  │ ..
  ▼
a/b
```

처럼 생각할 수 있습니다.

하지만 React Router의 기본 상대 네비게이션에서는 `..`를 단순히:

> URL 문자열에서 세그먼트 하나 삭제

라고만 이해하면 안 됩니다.

기본적으로는 **Route 계층에서 한 단계 위로 이동한다는 의미**가 더 중요합니다.

```text id="rm7xk3"
Route A
  │
  └── Route B
        │
        └── Route C ← 현재
               │
               │ ..
               ▼
             Route B
```

따라서 URL 구조와 Route 구조가 항상 1:1인 것은 아니라는 점을 기억해야 합니다.

---

# 11. `relative: "route"`

`useResolvedPath()`는 상대 경로를 어떻게 해석할지 옵션으로 지정할 수 있습니다.

기본적인 방식은 Route 계층을 기준으로 하는 것입니다.

```jsx id="lyjmzg"
const path = useResolvedPath(
  '../settings',
  {
    relative: 'route'
  }
);
```

개념적으로:

```text id="uwvbcf"
relative: "route"
        │
        ▼
현재 Route hierarchy 기준
        │
        ▼
상대 경로 해석
```

입니다.

이 방식은 Nested Routing과 자연스럽게 연결됩니다.

---

# 12. `relative: "path"`

URL Path 세그먼트 자체를 기준으로 상대 경로를 해석하고 싶다면:

```jsx id="vh9hx5"
const path = useResolvedPath(
  '../settings',
  {
    relative: 'path'
  }
);
```

를 사용할 수 있습니다.

개념적으로:

```text id="5jrsnq"
relative: "path"
       │
       ▼
현재 URL pathname의
Path 세그먼트 관계 기준
       │
       ▼
상대 경로 해석
```

입니다.

따라서 두 개념을 비교하면:

```text id="g6zslx"
relative: "route"

Route Tree
   │
   ▼
상대 경로 해석


relative: "path"

URL Path
   │
   ▼
상대 경로 해석
```

입니다.

---

# 13. `route`와 `path`의 차이

| 옵션                  | 기준                    |
| ------------------- | --------------------- |
| `relative: "route"` | 매칭된 Route hierarchy   |
| `relative: "path"`  | URL pathname의 Path 관계 |

기본적인 React Router 사용에서는 Route-relative 방식이 자연스럽습니다.

특히 Nested Route를 설계할 때:

```text id="vlsfcy"
Parent Route
     │
     └── Child Route
            │
            └── Grandchild Route
```

와 같은 관계를 그대로 상대 네비게이션에 활용할 수 있습니다.

---

# 14. `useResolvedPath()`는 실제로 이동하지 않는다

매우 중요한 특징입니다.

```jsx id="jhwohm"
const target =
  useResolvedPath('../settings');
```

이 코드를 실행한다고 해서 브라우저 URL이 변경되는 것은 아닙니다.

`useResolvedPath()`는 **경로를 계산할 뿐**입니다.

```text id="klra4n"
useResolvedPath()
      │
      ▼
Path 계산
      │
      ▼
{
  pathname,
  search,
  hash
}

URL 변경 X
```

실제 네비게이션은 다른 API가 담당합니다.

```text id="8pjpf0"
경로 계산
   ↓
useResolvedPath()


실제 이동
   ↓
<Link>
<NavLink>
useNavigate()
```

---

# 15. `useResolvedPath()`와 `useNavigate()`

두 Hook의 관계를 보면 이해하기 쉽습니다.

```jsx id="8yosyd"
const target =
  useResolvedPath('../edit');

const navigate =
  useNavigate();
```

`target`은 목적지를 계산합니다.

```text id="ns1jdz"
useResolvedPath()
       │
       ▼
목적지 계산
```

`navigate()`는 실제로 이동합니다.

```text id="dhbi04"
navigate()
    │
    ▼
실제 Navigation
```

따라서:

```text id="5f59ft"
"어디인가?"
     ↓
useResolvedPath()


"그곳으로 이동하라"
     ↓
useNavigate()
```

라고 구분할 수 있습니다.

---

# 16. 실전 예제

```jsx id="jvw60r"
import {
  useNavigate,
  useResolvedPath
} from 'react-router-dom';

function MenuButton() {
  const target =
    useResolvedPath('../edit');

  const navigate =
    useNavigate();

  const handleClick = () => {
    navigate(target);
  };

  return (
    <button onClick={handleClick}>
      Edit
    </button>
  );
}
```

전체적인 흐름은:

```text id="tot7xq"
"../edit"
    │
    ▼
useResolvedPath()
    │
    ▼
Resolved Path
    │
    ▼
navigate()
    │
    ▼
React Router Navigation
```

입니다.

다만 단순히 이동만 할 목적이라면 굳이 두 Hook을 함께 사용할 필요는 없습니다.

다음처럼 직접 사용할 수 있습니다.

```jsx id="ywd94i"
navigate('../edit');
```

따라서 `useResolvedPath()`는 **해석된 목적지 자체가 필요한 경우**에 더 의미가 있습니다.

---

# 17. 언제 `useResolvedPath()`가 특히 유용한가?

예를 들어 직접 링크와 비슷한 컴포넌트를 만든다고 하겠습니다.

```jsx id="5md4yk"
function CustomLink({ to, children }) {
  const resolved = useResolvedPath(to);

  // resolved를 이용한 추가 로직

  return (
    <Link to={to}>
      {children}
    </Link>
  );
}
```

이 경우:

```text id="i42k69"
to
 │
 ▼
useResolvedPath()
 │
 ├── 실제 목적지 분석
 ├── active 여부 계산
 └── 사용자 정의 로직
```

등에 활용할 수 있습니다.

실제로 `useResolvedPath()`는 일반 애플리케이션의 단순 이동보다는 **라우팅 관련 사용자 정의 컴포넌트를 만들 때 더 의미가 큰 Hook**입니다.

---

# 18. `useResolvedPath()`와 `<Link>`

다음 코드를 보겠습니다.

```jsx id="xdb8kx"
<Link to="../settings">
  Settings
</Link>
```

개발자는 최종 절대 경로를 직접 계산하지 않습니다.

React Router가 상대 `to`를 현재 라우팅 문맥에서 해석합니다.

```text id="irtoid"
<Link
  to="../settings"
>
       │
       ▼
React Router
Path Resolution
       │
       ▼
최종 목적지
```

`useResolvedPath()`는 이와 같은 **Path Resolution 결과를 컴포넌트에서 직접 얻어야 할 때 사용하는 API**라고 이해하면 좋습니다.

---

# 19. `useResolvedPath()`와 `useMatch()` 조합

두 Hook은 사용자 정의 Navigation 컴포넌트를 만들 때 함께 사용할 수 있습니다.

```jsx id="1k73fo"
const resolved =
  useResolvedPath(to);

const match =
  useMatch({
    path: resolved.pathname,
    end: true
  });
```

흐름:

```text id="me119e"
상대경로 to
     │
     ▼
useResolvedPath()
     │
     ▼
절대적으로 해석된 pathname
     │
     ▼
useMatch()
     │
     ▼
현재 URL과 비교
     │
     ▼
Active 여부
```

즉:

```text id="bb6e6o"
useResolvedPath()
      ↓
"이 링크가 실제로 어디를 가리키는가?"


useMatch()
      ↓
"현재 위치가 그 경로와 일치하는가?"
```

라고 구분할 수 있습니다.

---

# 20. `useLocation()`과 비교

`useLocation()`:

```jsx id="l8lb3g"
const location = useLocation();
```

현재 위치를 읽습니다.

```text id="0rbpxa"
현재 위치
/users/10/profile
```

`useResolvedPath()`:

```jsx id="fp2ilm"
const target =
  useResolvedPath('../settings');
```

목적지를 계산합니다.

```text id="tfkt2v"
목적지 후보
"../settings"
      │
      ▼
Resolved Path
```

즉:

```text id="zjllnc"
현재 어디에 있는가?
        ↓
   useLocation()


이 경로는 어디를 의미하는가?
        ↓
 useResolvedPath()
```

입니다.

---

# 21. `useMatch()`와 비교

`useMatch()`는 Path를 계산하지 않습니다.

```jsx id="ws4j5d"
const match =
  useMatch('/users/:id');
```

현재 `pathname`이 Pattern과 맞는지 검사합니다.

반면:

```jsx id="v6pfgm"
const path =
  useResolvedPath('../profile');
```

은 주어진 목적지가 현재 문맥에서 어떤 Path가 되는지를 계산합니다.

```text id="tkwvmp"
useResolvedPath()
      │
      ▼
Path Resolution


useMatch()
      │
      ▼
Path Matching
```

`Resolution`과 `Matching`을 구분하는 것이 중요합니다.

---

# 22. `useResolvedPath()`와 `useParams()`의 차이

`useParams()`는 현재 Route Match에서 Parameter 값을 읽습니다.

```text id="qzy4i0"
/users/10
       │
       ▼
useParams()
       │
       ▼
{
  id: "10"
}
```

`useResolvedPath()`는 목적지를 계산합니다.

```text id="r6u5ay"
"../settings"
       │
       ▼
useResolvedPath()
       │
       ▼
Resolved Path
```

역할이 완전히 다릅니다.

---

# 23. Node.js `path.resolve()`와 동일한 것은 아니다

`useResolvedPath()`를 설명할 때 Node.js의 `path.resolve()`를 비유로 사용할 수는 있습니다.

둘 다 상대적인 경로를 어떤 기준에 따라 해석한다는 점에서는 비슷합니다.

하지만 실제 동작 기반은 다릅니다.

Node.js:

```text id="j2yn5i"
File System Path
      │
      ▼
path.resolve()
```

React Router:

```text id="z4tpcn"
Route Context
+
Location
+
to
 │
 ▼
useResolvedPath()
```

즉, `useResolvedPath()`는 파일 시스템을 다루는 것이 아니라 **React Router의 Route 구조와 URL Path를 다루는 API**입니다.

따라서:

> 개념적 비유는 가능하지만 같은 알고리즘이라고 생각하면 안 됩니다.

---

# 24. 직접 문자열을 조합하지 않는 이유

다음과 같은 코드는 피하는 것이 좋습니다.

```jsx id="zq8ivf"
const url =
  location.pathname + '/../settings';
```

또는:

```jsx id="2vz06m"
const url =
  `${location.pathname}/profile`;
```

이 방식은 Nested Route, trailing slash, 상대 경로 등의 상황에서 문제가 발생하기 쉽습니다.

React Router에서는:

```jsx id="zjvgy6"
useResolvedPath('../settings');
```

처럼 Router의 Path Resolution 규칙을 사용하는 편이 안전합니다.

즉:

```text id="l82bvd"
직접 문자열 계산
      X

       ↓

React Router
Path Resolution
      O
```

라고 생각하면 됩니다.

---

# 25. 관련 Hook 비교

| Hook                | 역할                                |
| ------------------- | --------------------------------- |
| `useResolvedPath()` | `to`를 현재 라우팅 문맥에서 Path로 해석        |
| `useLocation()`     | 현재 Location 읽기                    |
| `useMatch()`        | 현재 pathname과 특정 Pattern 비교        |
| `useParams()`       | 현재 Route Match의 Path Parameter 읽기 |
| `useNavigate()`     | 실제 네비게이션 실행                       |
| `useSearchParams()` | Query Parameter 읽기/변경             |

목적을 질문으로 바꾸면 더 쉽게 구분할 수 있습니다.

```text id="l7h4lu"
현재 어디에 있는가?
        ↓
useLocation()


현재 URL의 :id는 무엇인가?
        ↓
useParams()


현재 URL이 이 패턴과 맞는가?
        ↓
useMatch()


"../settings"는 실제 어디인가?
        ↓
useResolvedPath()


그곳으로 이동하고 싶은가?
        ↓
useNavigate()
```

---

# 26. 전체 동작 구조

`useResolvedPath()`의 동작을 전체적으로 정리하면 다음과 같습니다.

```text id="zxg4v2"
현재 Route Context
        │
        │
현재 Location
        │
        │
        ├─────────────┐
        │             │
        ▼             │
   상대 경로 to       │
  "../settings"       │
        │             │
        └──────┬──────┘
               ▼
       useResolvedPath()
               │
               ▼
       Path Resolution
               │
               ▼
┌──────────────────────────┐
│       Resolved Path      │
│                          │
│ pathname                 │
│ search                   │
│ hash                     │
└──────────────────────────┘
```

중요한 것은 여기까지는 **Navigation이 아니라 Path 계산**이라는 점입니다.

```text id="j6yah3"
useResolvedPath()
      │
      ▼
Path 계산 완료
      │
      X
URL은 아직 변경되지 않음
```

---

# 27. `useResolvedPath()`의 핵심

`useResolvedPath()`를 단순히:

> 상대 경로를 절대 경로로 바꾸는 Hook

이라고 설명할 수는 있지만, 이것만으로는 React Router의 동작을 충분히 설명하지 못합니다.

더 정확하게는:

> **`useResolvedPath()`는 전달된 `to` 값을 현재 React Router의 Route Context와 Location을 기준으로 해석하여 최종 `pathname`, `search`, `hash`를 가진 `Path` 객체로 계산하는 Hook입니다.**

가장 압축하면:

```text id="p7c7cc"
Relative To
"../settings"
      │
      +
Route Context
      │
      +
Location
      │
      ▼
useResolvedPath()
      │
      ▼
Resolved Path
```

따라서 `useResolvedPath()`의 본질은 **“URL을 이동시키는 것”이 아니라 “React Router의 현재 문맥에서 이 `to`가 실제로 어느 경로를 의미하는지 계산하는 것”**입니다.
