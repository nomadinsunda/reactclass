# React Router v6의 `useNavigate()`

`useNavigate()`는 React Router에서 **JavaScript 코드로 네비게이션을 실행할 수 있도록 해주는 Hook**입니다.

일반적인 링크 이동은 `<Link>`나 `<NavLink>`를 사용합니다.

```jsx
<Link to="/about">
  About
</Link>
```

하지만 모든 페이지 이동이 사용자의 링크 클릭으로 발생하는 것은 아닙니다.

예를 들어 다음과 같은 상황이 있습니다.

```text
로그인 요청
    │
    ▼
로그인 성공
    │
    ▼
메인 페이지로 이동
```

또는:

```text
폼 제출
   │
   ▼
서버 저장
   │
   ▼
저장 성공
   │
   ▼
상세 페이지로 이동
```

이처럼 **이벤트 처리 결과, 조건 판단, API 응답 등의 결과에 따라 코드에서 직접 이동해야 할 때** `useNavigate()`를 사용합니다.

---

# 1. `useNavigate()`란?

`useNavigate()`는 React Router가 제공하는 Hook으로, 호출하면 **네비게이션을 수행하는 `navigate` 함수**를 반환합니다.

```jsx
import { useNavigate } from 'react-router-dom';

function Example() {
  const navigate = useNavigate();

  // ...
}
```

반환된 `navigate` 함수를 사용하여 원하는 경로로 이동할 수 있습니다.

```jsx
navigate('/about');
```

핵심 구조는 다음과 같습니다.

```text
useNavigate()
      │
      ▼
 navigate 함수
      │
      │ navigate("/about")
      ▼
React Router Navigation
      │
      ▼
현재 Location 변경
      │
      ▼
새로운 Route Matching
      │
      ▼
React UI 업데이트
```

---

# 2. 한 문장으로 정의하면

> `useNavigate()`는 React Router의 Router Context에 접근하여 JavaScript 코드에서 새로운 경로로 이동하거나 History Entry를 이동할 수 있는 `navigate` 함수를 반환하는 Hook입니다.

---

# 3. 기본 사용법

```jsx
import { useNavigate } from 'react-router-dom';

function Example() {
  const navigate = useNavigate();

  const handleClick = () => {
    navigate('/about');
  };

  return (
    <button onClick={handleClick}>
      About으로 이동
    </button>
  );
}
```

사용자가 버튼을 클릭하면:

```text
button click
     │
     ▼
handleClick()
     │
     ▼
navigate("/about")
     │
     ▼
React Router
     │
     ▼
Location 변경
     │
     ▼
/about에 맞는 UI 렌더링
```

이 과정은 일반적인 SPA 네비게이션이므로 새로운 HTML Document를 서버에서 다시 받아오는 전체 페이지 이동과는 다릅니다.

---

# 4. `<Link>`와 `useNavigate()`의 차이

두 기능 모두 페이지를 이동할 수 있지만 사용 목적이 다릅니다.

`<Link>`:

```jsx
<Link to="/about">
  About
</Link>
```

사용자가 링크를 선택하면 이동합니다.

```text
사용자 클릭
    │
    ▼
<Link>
    │
    ▼
React Router Navigation
```

반면 `useNavigate()`는:

```jsx
if (success) {
  navigate('/complete');
}
```

처럼 프로그램의 실행 결과에 의해 이동합니다.

```text
프로그램 로직
    │
    ▼
조건 판단
    │
    ▼
navigate()
    │
    ▼
React Router Navigation
```

정리하면:

| 구분          | `<Link>`     | `useNavigate()` |
| ----------- | ------------ | --------------- |
| 방식          | 선언적 네비게이션    | 명령적 네비게이션       |
| 주된 시작점      | 사용자의 링크 선택   | JavaScript 로직   |
| JSX 컴포넌트    | O            | X               |
| 이벤트 후 이동    | 가능하지만 목적이 다름 | 적합              |
| API 응답 후 이동 | 부적합          | 적합              |
| 로그인 성공 후 이동 | 부적합          | 적합              |

따라서 기본 원칙은:

```text
사용자가 직접 이동할 링크
        ↓
      <Link>


코드 실행 결과에 따라 이동
        ↓
   useNavigate()
```

라고 생각하면 됩니다.

---

# 5. `navigate()` 함수

`useNavigate()`를 호출하면 `navigate` 함수가 반환됩니다.

```jsx
const navigate = useNavigate();
```

`navigate`는 크게 두 가지 방식으로 사용할 수 있습니다.

## 경로를 지정해서 이동

```jsx
navigate('/about');
```

## History에서 상대적인 위치로 이동

```jsx
navigate(-1);
```

따라서 개념적인 타입은 다음처럼 이해할 수 있습니다.

```ts
navigate(to, options?);

navigate(delta);
```

첫 번째 방식은 새로운 목적지를 지정하는 것이고:

```jsx
navigate('/products');
```

두 번째 방식은 현재 History Entry를 기준으로 이동하는 것입니다.

```jsx
navigate(-1);
navigate(1);
```

---

# 6. 기본 이동은 새로운 History Entry를 추가한다

다음 코드를 실행한다고 하겠습니다.

```jsx
navigate('/about');
```

일반적인 네비게이션에서는 새로운 History Entry가 추가됩니다.

현재 History가:

```text
/home   ← 현재
```

라면:

```jsx
navigate('/about');
```

실행 후:

```text
/home
/about   ← 현재
```

가 됩니다.

다시:

```jsx
navigate('/products');
```

를 실행하면:

```text
/home
/about
/products   ← 현재
```

가 됩니다.

따라서 브라우저의 뒤로 가기를 실행하면:

```text
/products
     │
     ▼
/about
```

으로 돌아갈 수 있습니다.

이것이 일반적으로 `push` 네비게이션이라고 부르는 동작입니다.

---

# 7. `replace: true`

새로운 History Entry를 추가하지 않고 **현재 Entry를 교체하고 싶다면** `replace` 옵션을 사용합니다.

```jsx
navigate('/main', {
  replace: true
});
```

현재 History가:

```text
/login   ← 현재
```

이라고 하겠습니다.

일반적인 이동:

```jsx
navigate('/main');
```

결과:

```text
/login
/main   ← 현재
```

따라서 뒤로 가기를 하면 `/login`으로 돌아갈 수 있습니다.

반면:

```jsx
navigate('/main', {
  replace: true
});
```

를 사용하면:

```text
Before

/login   ← 현재


replace


After

/main    ← 현재
```

처럼 현재 History Entry 자체가 교체됩니다.

---

# 8. `push`와 `replace`

두 동작의 차이는 매우 중요합니다.

```text
push

A
│
│ navigate("/B")
▼
A
B ← 현재


replace

A ← 현재
│
│ navigate("/B", { replace: true })
▼
B ← 현재
```

정리하면:

| 방식              | History 동작   | 뒤로 가기               |
| --------------- | ------------ | ------------------- |
| 기본 navigate     | 새로운 Entry 추가 | 이전 Entry로 이동 가능     |
| `replace: true` | 현재 Entry 교체  | 교체된 Entry로 돌아갈 수 없음 |

`replace`는 특히 로그인이나 인증 처리에서 자주 사용됩니다.

```text
/login
   │
   │ 로그인 성공
   ▼
/main
```

로그인 성공 후 다시 로그인 화면으로 돌아가는 것이 자연스럽지 않다면:

```jsx
navigate('/main', {
  replace: true
});
```

와 같은 패턴을 사용할 수 있습니다.

---

# 9. `state`를 이용한 데이터 전달

`navigate()`는 경로뿐만 아니라 추가적인 상태 데이터를 함께 전달할 수 있습니다.

```jsx
navigate('/detail/10', {
  state: {
    from: 'product-list',
    scrollY: 320
  }
});
```

이 데이터는 URL Path나 Query String에 포함되지 않습니다.

주소창에는:

```text
/detail/10
```

만 표시됩니다.

하지만 해당 Location에는 다음과 같은 상태가 연결될 수 있습니다.

```text
Location
│
├── pathname: "/detail/10"
│
└── state
    ├── from: "product-list"
    └── scrollY: 320
```

---

# 10. 전달한 `state` 읽기

전달받은 페이지에서는 `useLocation()`으로 읽습니다.

```jsx
import { useLocation } from 'react-router-dom';

function Detail() {
  const location = useLocation();

  console.log(location.state);

  // ...
}
```

또는 구조 분해를 사용할 수 있습니다.

```jsx
const { state } = useLocation();

console.log(state?.from);
console.log(state?.scrollY);
```

결과:

```text
product-list
320
```

즉:

```text
navigate()
   │
   │ state 전달
   ▼
새로운 Location
   │
   ▼
useLocation()
   │
   ▼
location.state
```

의 관계입니다.

---

# 11. `state`에 대한 중요한 주의점

다음과 같이 이해하면 안 됩니다.

```text
location.state는 새로고침하면 무조건 사라진다.
```

React Router의 Location State는 브라우저의 History Entry에 연결된 `history.state`를 기반으로 하기 때문에 같은 History Entry를 새로고침하는 경우 유지될 수 있습니다.

하지만 중요한 애플리케이션 데이터를 저장하는 용도로 사용해서는 안 됩니다.

이유는 다음과 같습니다.

```text
location.state

URL에 표현되지 않음
        │
        ├── URL 공유만으로 전달되지 않음
        ├── 직접 해당 URL로 접근하면 없을 수 있음
        ├── 서버가 URL을 통해 읽을 수 없음
        └── 영속적인 데이터 저장소가 아님
```

따라서:

```text
페이지 자체를 표현하는 핵심 정보
        ↓
Path / Query Parameter


이동 과정에서 필요한 부가 정보
        ↓
location.state
```

처럼 구분하는 것이 좋습니다.

---

# 12. 뒤로 가기

`navigate()`에 숫자를 전달하면 History Entry 사이를 이동할 수 있습니다.

```jsx
navigate(-1);
```

이는 한 단계 뒤로 가기를 의미합니다.

현재 History:

```text
/home
/products
/products/10   ← 현재
```

여기서:

```jsx
navigate(-1);
```

을 실행하면:

```text
/home
/products      ← 현재
/products/10
```

으로 이동합니다.

브라우저의 뒤로 가기와 유사한 동작입니다.

---

# 13. 앞으로 가기

한 단계 앞으로 이동하려면:

```jsx
navigate(1);
```

을 사용할 수 있습니다.

```text
/home
/products      ← 현재
/products/10
```

여기서:

```jsx
navigate(1);
```

을 실행하면:

```text
/home
/products
/products/10   ← 현재
```

가 됩니다.

---

# 14. 여러 단계 이동

숫자는 History에서 이동할 상대적인 거리입니다.

```jsx
navigate(-2);
```

두 단계 뒤로 이동합니다.

```text
A
B
C   ← 현재

navigate(-2)

A   ← 현재
B
C
```

반대로:

```jsx
navigate(2);
```

처럼 앞으로 여러 단계 이동할 수도 있습니다.

다만 앞으로 이동할 History Entry가 존재해야 실제 이동이 가능합니다.

---

# 15. 상대 경로 이동

`navigate()`는 절대 Path뿐만 아니라 상대 Path도 사용할 수 있습니다.

```jsx
navigate('settings');
```

또는:

```jsx
navigate('../settings');
```

와 같이 사용할 수 있습니다.

React Router에서 상대 경로는 단순한 문자열 결합이 아니라 **현재 Router의 라우팅 문맥을 기준으로 해석**됩니다.

예를 들어 다음과 같은 중첩 Route 구조가 있다고 하겠습니다.

```text
users
└── :userId
    ├── profile
    └── settings
```

현재 `profile` Route에서:

```jsx
navigate('../settings');
```

와 같이 상위 Route 관계를 기준으로 이동할 수 있습니다.

상대 네비게이션은 중첩 라우팅과 함께 사용할 때 특히 유용합니다.

---

# 16. Route-relative와 Path-relative

상대 경로를 정확하게 이해하려면 React Router의 Route 구조를 생각해야 합니다.

기본적인 상대 네비게이션은 **Route 계층 관계를 기준으로 해석**될 수 있습니다.

필요한 경우 `relative` 옵션을 사용할 수 있습니다.

```jsx
navigate('../settings', {
  relative: 'route'
});
```

또는:

```jsx
navigate('../settings', {
  relative: 'path'
});
```

개념적으로:

```text
relative: "route"
      ↓
Route 계층 관계 기준


relative: "path"
      ↓
URL Path 세그먼트 기준
```

따라서 상대 경로를 단순히:

```text
".." = URL 문자열 하나 제거
```

라고만 이해하면 중첩 Route에서 혼동할 수 있습니다.

---

# 17. 로그인 성공 후 이동

`useNavigate()`의 대표적인 사용 사례입니다.

```jsx
import { useNavigate } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();

  async function handleLogin() {
    const success = await loginAPI();

    if (success) {
      navigate('/', {
        replace: true
      });
    }
  }

  return (
    <button onClick={handleLogin}>
      Login
    </button>
  );
}
```

전체 흐름:

```text
사용자 Login 클릭
       │
       ▼
handleLogin()
       │
       ▼
loginAPI()
       │
       │ 성공
       ▼
navigate("/")
       │
       ▼
React Router
       │
       ▼
Location 변경
       │
       ▼
Route Matching
       │
       ▼
Home UI 렌더링
```

---

# 18. API 요청 성공 후 이동

폼 제출 후에도 자주 사용합니다.

```jsx
const navigate = useNavigate();

async function handleSubmit(data) {
  const result = await saveData(data);

  if (result.success) {
    navigate(`/items/${result.id}`);
  }
}
```

흐름:

```text
폼 제출
   │
   ▼
saveData()
   │
   ▼
서버 응답
   │
   │ success
   ▼
navigate("/items/10")
   │
   ▼
상세 페이지 이동
```

이것이 `useNavigate()`가 필요한 대표적인 이유입니다.

---

# 19. 렌더링 중 `navigate()`를 직접 호출하면 안 된다

다음 코드는 좋지 않은 패턴입니다.

```jsx
function MyPage() {
  const navigate = useNavigate();

  if (!user) {
    navigate('/login');
  }

  return <div>My Page</div>;
}
```

컴포넌트의 렌더링 과정에서 Side Effect인 네비게이션을 직접 실행하고 있기 때문입니다.

React 컴포넌트의 렌더링은 순수해야 합니다.

```text
Render
  │
  ├── JSX 계산
  ├── 값 계산
  │
  └── navigate() 직접 실행 X
```

이벤트의 결과로 이동한다면 이벤트 핸들러에서 실행합니다.

```jsx
const handleClick = () => {
  navigate('/login');
};
```

특정 렌더링 상태에 따라 선언적으로 다른 위치를 보여주어야 한다면 `<Navigate>`를 사용할 수도 있습니다.

```jsx
if (!user) {
  return <Navigate to="/login" replace />;
}
```

즉, 상황에 따라 네비게이션 방식을 구분해야 합니다.

---

# 20. `useNavigate()`와 `<Navigate>`

둘은 이름이 비슷하지만 목적이 다릅니다.

`useNavigate()`:

```jsx
const navigate = useNavigate();

const handleLogin = async () => {
  const success = await login();

  if (success) {
    navigate('/main');
  }
};
```

프로그램 로직에서 **명령적으로 이동**합니다.

반면 `<Navigate>`는:

```jsx
if (!user) {
  return <Navigate to="/login" replace />;
}
```

처럼 렌더링 결과로 다른 Location으로 이동하도록 표현합니다.

```text
이벤트 / 비동기 로직 결과
          ↓
     useNavigate()


렌더링 조건에 따른 이동
          ↓
      <Navigate>
```

---

# 21. `useNavigate()`와 `useLocation()`

두 Hook의 역할은 서로 반대 방향이라고 볼 수 있습니다.

```text
useLocation()
     │
     ▼
현재 Location 읽기


useNavigate()
     │
     ▼
새로운 Location으로 이동
```

예:

```jsx
function Example() {
  const location = useLocation();
  const navigate = useNavigate();

  return (
    <>
      <p>
        현재 위치: {location.pathname}
      </p>

      <button
        onClick={() => navigate('/about')}
      >
        이동
      </button>
    </>
  );
}
```

---

# 22. `useNavigate()`와 `useHistory()`

React Router v5에서는 Programmatic Navigation에 `useHistory()`를 사용했습니다.

React Router v5:

```jsx
const history = useHistory();

history.push('/about');
```

React Router v6에서는:

```jsx
const navigate = useNavigate();

navigate('/about');
```

로 변경되었습니다.

대표적인 API를 비교하면:

| React Router v5         | React Router v6                     |
| ----------------------- | ----------------------------------- |
| `useHistory()`          | `useNavigate()`                     |
| `history.push('/a')`    | `navigate('/a')`                    |
| `history.replace('/a')` | `navigate('/a', { replace: true })` |
| `history.goBack()`      | `navigate(-1)`                      |
| `history.goForward()`   | `navigate(1)`                       |
| `history.go(-2)`        | `navigate(-2)`                      |

v6에서는 History 객체를 직접 다루는 형태보다 **“어디로 이동할 것인가”라는 네비게이션 동작 중심의 API**로 단순화되었습니다.

---

# 23. 내부 동작을 어떻게 이해해야 하는가?

BrowserRouter 환경에서 다음 코드가 실행된다고 하겠습니다.

```jsx
navigate('/about');
```

개념적인 흐름은 다음과 같습니다.

```text
React Component
      │
      │ navigate("/about")
      ▼
React Router
      │
      │ Navigation 처리
      ▼
Browser History
      │
      ▼
현재 Location 변경
      │
      ▼
Router 상태 변경
      │
      ▼
Route Matching
      │
      ▼
React 렌더링
      │
      ▼
새로운 UI
```

React Router는 브라우저 환경에서 History API를 기반으로 네비게이션을 관리합니다.

하지만 다음처럼 지나치게 단순하게 이해해서는 안 됩니다.

```text
navigate()
   =
history.pushState()
```

`navigate()`는 단순한 브라우저 API의 별칭이 아닙니다.

> `navigate()`는 React Router의 네비게이션 시스템을 통해 새로운 Location으로 이동하도록 요청하는 고수준 API입니다.

브라우저 History API는 그 아래에서 사용되는 플랫폼 기능입니다.

---

# 24. SPA에서는 왜 새로고침하지 않는가?

전통적인 링크 이동은 새로운 Document 요청을 발생시킬 수 있습니다.

```text
URL 이동
   │
   ▼
Server Request
   │
   ▼
HTML Document 다운로드
   │
   ▼
새 Document 생성
```

React Router의 클라이언트 네비게이션은 일반적으로:

```text
navigate("/about")
        │
        ▼
History / Location 변경
        │
        ▼
React Router
        │
        ▼
Route Matching
        │
        ▼
React UI 변경
```

처럼 동작합니다.

즉:

> **현재 Document를 유지하면서 URL과 필요한 React UI를 변경하는 것이 SPA 네비게이션의 핵심입니다.**

---

# 25. 자주 사용하는 실전 패턴

`useNavigate()`는 특히 다음 상황에서 많이 사용합니다.

| 상황            | 예              |
| ------------- | -------------- |
| 로그인 성공        | `/main` 이동     |
| 로그아웃 완료       | `/login` 이동    |
| 회원가입 완료       | `/welcome` 이동  |
| 폼 저장 성공       | 상세 페이지 이동      |
| 상품 선택         | 상품 상세 이동       |
| 결제 완료         | 주문 완료 페이지 이동   |
| API 결과에 따른 분기 | 성공/실패 페이지 이동   |
| 뒤로 가기 버튼      | `navigate(-1)` |
| 특정 작업 후 이동    | 목록 페이지 복귀      |

핵심 공통점은 하나입니다.

> **사용자가 단순히 링크를 클릭해서 이동하는 것이 아니라 프로그램 로직의 결과로 이동이 결정된다.**

---

# 26. 관련 React Router API 비교

| API                 | 역할                          |
| ------------------- | --------------------------- |
| `<Link>`            | 사용자가 선택하는 일반 네비게이션 링크       |
| `<NavLink>`         | 현재 위치의 active 상태를 알 수 있는 링크 |
| `useNavigate()`     | 코드에서 네비게이션 실행               |
| `<Navigate>`        | 렌더링 결과로 다른 위치로 이동           |
| `useLocation()`     | 현재 Location 읽기              |
| `useParams()`       | 현재 Route의 Path Parameter 읽기 |
| `useSearchParams()` | Query Parameter 읽기/변경       |
| `useMatch()`        | 현재 pathname과 특정 Pattern 비교  |
| `useResolvedPath()` | 상대 Path 해석                  |

이를 목적별로 분류하면:

```text
사용자가 링크를 선택
        │
        ├── <Link>
        └── <NavLink>


코드 로직에서 이동
        │
        └── useNavigate()


렌더링 조건으로 이동
        │
        └── <Navigate>


현재 위치 읽기
        │
        └── useLocation()


현재 URL Pattern 검사
        │
        └── useMatch()
```

---

# 27. 전체 구조

`useNavigate()`의 전체 흐름을 정리하면 다음과 같습니다.

```text
React Component
       │
       │
       ▼
 useNavigate()
       │
       ▼
   navigate()
       │
       │
       ├── navigate("/about")
       ├── navigate("/login", { replace: true })
       ├── navigate(-1)
       └── navigate(1)
       │
       ▼
 React Router
       │
       ▼
History / Location 변경
       │
       ▼
  Route Matching
       │
       ▼
 React UI 업데이트
```

---

# 28. `useNavigate()`의 핵심

`useNavigate()`를 단순히:

> 페이지를 이동시키는 Hook

이라고만 이해하면 부족합니다.

더 정확하게는:

> **`useNavigate()`는 React Router의 네비게이션 시스템을 통해 JavaScript 코드에서 새로운 Location으로 이동하거나 현재 History를 기준으로 다른 Entry로 이동할 수 있도록 `navigate` 함수를 제공하는 Hook입니다.**

가장 압축하면:

```text
프로그램 로직
     │
     ▼
navigate(...)
     │
     ▼
React Router
     │
     ▼
Location 변경
     │
     ▼
Route Matching
     │
     ▼
UI 변경
```

따라서 `useNavigate()`의 본질은 **“URL 문자열을 변경하는 것”이 아니라 “React Router에게 새로운 네비게이션을 실행하도록 요청하는 것”**이라고 이해하는 것이 가장 정확합니다.
