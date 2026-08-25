
# React Router의 `useHistory`

`useHistory`는 React Router v5에서 제공하던 Hook으로, **컴포넌트 코드 안에서 브라우저의 History를 조작하여 페이지를 이동할 수 있게 해주는 기능**입니다.

먼저 가장 중요한 점부터 짚고 가야 합니다.

> `useHistory`는 **React Router v5에서 사용하던 API**이며, **React Router v6부터는 `useNavigate`로 대체되었습니다.**

React Router v5:

```jsx
import { useHistory } from 'react-router-dom';

const history = useHistory();

history.push('/about');
```

React Router v6 이상:

```jsx
import { useNavigate } from 'react-router-dom';

const navigate = useNavigate();

navigate('/about');
```

따라서 `useHistory`는 현재 프로젝트에서 새롭게 사용하기 위한 API라기보다, **기존 React Router v5 코드를 이해하고 `useNavigate`가 왜 등장했는지 이해하기 위해 알아두어야 하는 API**라고 보는 것이 좋습니다.

---

# 1. `useHistory`는 왜 필요한가?

React Router에서 일반적인 페이지 이동은 `<Link>`를 사용합니다.

```jsx
<Link to="/about">
  About
</Link>
```

사용자가 링크를 클릭하면 `/about`으로 이동합니다.

하지만 모든 페이지 이동이 `<Link>` 클릭으로 발생하는 것은 아닙니다.

예를 들어 로그인 처리를 생각해 봅시다.

```text
로그인 버튼 클릭
      |
      v
서버에 로그인 요청
      |
      v
로그인 성공
      |
      v
메인 페이지로 이동
```

이 경우 페이지 이동은 JSX에 작성된 링크 자체가 아니라 **JavaScript 코드의 실행 결과에 의해 결정**됩니다.

이런 것을 흔히 **Programmatic Navigation**이라고 합니다.

> Programmatic Navigation이란 사용자가 링크를 직접 선택해서 이동하는 것이 아니라, 프로그램의 로직에 따라 코드에서 페이지 이동을 실행하는 것을 말합니다.

React Router v5에서는 이를 위해 `useHistory`를 사용했습니다.

```jsx
const history = useHistory();

history.push('/main');
```

---

# 2. 기본 사용 방법

React Router v5에서는 다음과 같이 사용합니다.

```jsx
import { useHistory } from 'react-router-dom';

function Login() {
  const history = useHistory();

  const handleLogin = () => {
    // 로그인 처리

    history.push('/main');
  };

  return (
    <button onClick={handleLogin}>
      Login
    </button>
  );
}
```

핵심 부분은 다음입니다.

```jsx
const history = useHistory();
```

`useHistory()`를 호출하면 React Router가 관리하는 **history 객체에 접근할 수 있습니다.**

그리고:

```jsx
history.push('/main');
```

을 호출하면 새로운 URL이 History Stack에 추가되고 해당 경로로 이동합니다.

---

# 3. `<Link>`와 `useHistory`의 차이

두 기능 모두 React Router에서 페이지 이동에 사용됩니다.

하지만 **누가 이동을 시작하는가**가 다릅니다.

`<Link>`:

```jsx
<Link to="/about">
  About
</Link>
```

흐름:

```text
사용자가 Link 클릭
       |
       v
React Router
       |
       v
/about 이동
```

`useHistory`:

```jsx
if (loginSuccess) {
  history.push('/main');
}
```

흐름:

```text
JavaScript 로직 실행
       |
       v
조건 만족
       |
       v
history.push('/main')
       |
       v
React Router
       |
       v
/main 이동
```

정리하면 다음과 같습니다.

| 기능               | `<Link>`   | `useHistory`         |
| ---------------- | ---------- | -------------------- |
| 이동 방식            | 선언적        | 명령적                  |
| 이동 시작            | 사용자의 링크 클릭 | JavaScript 코드        |
| 대표적인 사용          | 메뉴, 링크     | 로그인 성공, 저장 완료 등      |
| JSX 필요           | O          | X                    |
| React Router v5  | O          | O                    |
| React Router v6+ | O          | X (`useNavigate` 사용) |

---

# 4. `useHistory()`가 반환하는 것은 무엇인가?

`useHistory()`는 단순한 이동 함수 하나를 반환하는 것이 아닙니다.

React Router v5가 사용하는 **history 객체**를 반환합니다.

개념적으로 다음과 같은 형태입니다.

```js
const history = {
  push,
  replace,
  go,
  goBack,
  goForward,
  location,
  ...
};
```

따라서 다음과 같은 작업이 가능합니다.

```jsx
history.push('/about');
history.replace('/login');
history.go(-1);
history.goBack();
history.goForward();
```

즉:

> `useHistory`는 React 컴포넌트에서 React Router가 사용하는 History 객체에 접근하기 위한 Hook입니다.

---

# 5. `history.push()`

가장 많이 사용되는 메서드입니다.

```jsx
history.push('/about');
```

새로운 경로를 **History Stack에 추가**하면서 이동합니다.

브라우저 방문 기록을 단순화해서 표현하면:

```text
현재 History Stack

/home
  |
  v
현재 위치
```

여기서:

```jsx
history.push('/about');
```

을 실행하면:

```text
History Stack

/home
/about   <- 현재 위치
```

가 됩니다.

다시:

```jsx
history.push('/contact');
```

을 실행하면:

```text
History Stack

/home
/about
/contact   <- 현재 위치
```

가 됩니다.

따라서 브라우저의 뒤로 가기를 실행하면:

```text
/contact
    |
    | 뒤로 가기
    v
/about
```

으로 돌아갈 수 있습니다.

---

# 6. 브라우저의 `history.pushState()`와의 관계

여기서 중요한 점이 있습니다.

`history.push()`는 브라우저가 제공하는 `window.history.pushState()`와 이름이 비슷하지만 **동일한 API는 아닙니다.**

브라우저 API:

```js
window.history.pushState(
  state,
  '',
  '/about'
);
```

React Router v5:

```jsx
history.push('/about');
```

React Router는 내부적으로 브라우저의 History API를 기반으로 클라이언트 측 네비게이션을 구현합니다.

개념적인 관계는 다음과 같습니다.

```text
React Component
      |
      | history.push("/about")
      v
React Router
      |
      | History 관리
      v
Browser History API
      |
      | URL / History 변경
      v
Browser
```

따라서 `history.push()`를 단순히 `window.history.pushState()`의 별칭이라고 이해하면 안 됩니다.

> `history.push()`는 React Router가 제공하는 라우팅 추상화이고, 브라우저 History API는 그 아래에서 사용되는 웹 플랫폼 기능입니다.

---

# 7. `history.push()`를 호출하면 새로고침되는가?

일반적인 React Router의 클라이언트 측 네비게이션에서는 **문서 전체가 새로고침되지 않습니다.**

예를 들어:

```jsx
history.push('/about');
```

이 실행되었다고 하겠습니다.

전통적인 페이지 이동은 다음과 비슷합니다.

```text
/about 이동
   |
   v
서버에 새로운 Document 요청
   |
   v
HTML 다운로드
   |
   v
새 Document 생성
```

React Router의 SPA 네비게이션은 다릅니다.

```text
history.push('/about')
        |
        v
URL / History 변경
        |
        v
React Router가 location 변경 반영
        |
        v
Route 다시 매칭
        |
        v
필요한 React UI 변경
```

즉:

> 현재 Document를 유지하면서 URL과 React UI를 변경합니다.

이것이 SPA 라우팅의 핵심입니다.

---

# 8. `history.replace()`

`push()`와 함께 반드시 알아야 하는 메서드입니다.

```jsx
history.replace('/main');
```

`push()`는 새로운 History Entry를 추가하지만 `replace()`는 **현재 Entry를 새로운 Entry로 교체합니다.**

`push()`:

```text
/home
  |
  | push("/about")
  v
/home
/about   <- 현재
```

`replace()`:

```text
/home   <- 현재

replace("/about")

        v

/about  <- 현재
```

즉, 이전 `/home` 기록을 새로운 `/about` 기록으로 교체합니다.

---

# 9. `push()`와 `replace()`의 차이

| 메서드         | History 동작   | 뒤로 가기                 |
| ----------- | ------------ | --------------------- |
| `push()`    | 새로운 Entry 추가 | 이전 페이지로 돌아갈 수 있음      |
| `replace()` | 현재 Entry 교체  | 교체된 이전 Entry로 돌아가지 않음 |

예를 들어 로그인 페이지를 생각해 볼 수 있습니다.

```text
/login
   |
   | 로그인 성공
   v
/main
```

다음과 같이 작성하면:

```jsx
history.push('/main');
```

History는:

```text
/login
/main   <- 현재
```

이 됩니다.

따라서 뒤로 가기를 누르면 `/login`으로 돌아갈 수 있습니다.

반면:

```jsx
history.replace('/main');
```

을 사용하면 개념적으로:

```text
/main   <- 현재
```

가 되어 로그인 페이지로 돌아가는 것을 방지하는 패턴을 만들 수 있습니다.

---

# 10. 뒤로 가기

React Router v5에서는 다음과 같이 사용할 수 있습니다.

```jsx
history.goBack();
```

예:

```jsx
function Detail() {
  const history = useHistory();

  return (
    <button onClick={() => history.goBack()}>
      Back
    </button>
  );
}
```

History Stack이:

```text
/home
/products
/products/10   <- 현재
```

라면:

```jsx
history.goBack();
```

실행 후:

```text
/home
/products   <- 현재
/products/10
```

처럼 이전 History Entry로 이동합니다.

---

# 11. 앞으로 가기

반대로:

```jsx
history.goForward();
```

를 사용할 수도 있습니다.

```text
/home
/products   <- 현재
/products/10
```

여기서:

```jsx
history.goForward();
```

을 실행하면:

```text
/home
/products
/products/10   <- 현재
```

로 이동합니다.

---

# 12. `history.go()`

특정 History Entry만큼 이동할 수도 있습니다.

```jsx
history.go(-1);
```

한 단계 뒤로:

```text
현재
 |
 v
C
 |
 | go(-1)
 v
B
```

두 단계 뒤로:

```jsx
history.go(-2);
```

```text
A
B
C   <- 현재

go(-2)

A   <- 현재
B
C
```

반대로:

```jsx
history.go(1);
```

은 한 단계 앞으로 이동합니다.

---

# 13. 실전 예제: 로그인 성공 후 이동

`useHistory`가 가장 자연스럽게 사용되는 예제입니다.

```jsx
import { useHistory } from 'react-router-dom';

function Login() {
  const history = useHistory();

  const handleLogin = async () => {
    const success = await login();

    if (success) {
      history.replace('/main');
    }
  };

  return (
    <button onClick={handleLogin}>
      Login
    </button>
  );
}
```

전체 흐름은 다음과 같습니다.

```text
사용자
  |
  | Login 클릭
  v
handleLogin()
  |
  v
login()
  |
  | 성공
  v
history.replace("/main")
  |
  v
React Router
  |
  v
URL 변경
  |
  v
Route Matching
  |
  v
Main 컴포넌트 렌더링
```

여기서 중요한 것은 **페이지 이동이 사용자의 링크 클릭 자체가 아니라 로그인 로직의 결과로 발생했다는 것**입니다.

---

# 14. `useHistory`와 BrowserRouter의 관계

`useHistory()`는 아무 곳에서나 사용할 수 있는 Hook이 아닙니다.

React Router의 Router 내부에서 사용해야 합니다.

예를 들어:

```jsx
<BrowserRouter>
  <App />
</BrowserRouter>
```

구조가 있다면:

```text
BrowserRouter
     |
     | Router Context
     v
    App
     |
     v
   Login
     |
     v
 useHistory()
```

`useHistory()`는 Router가 제공하는 라우팅 정보를 사용합니다.

따라서 개념적으로:

```text
BrowserRouter
      |
      | History / Location 관리
      |
      | Context
      v
React Component
      |
      v
useHistory()
      |
      v
history 객체
```

라는 관계를 가집니다.

---

# 15. `useHistory`의 전체 동작 흐름

다음 코드가 실행된다고 생각해 봅시다.

```jsx
const history = useHistory();

history.push('/about');
```

전체 구조를 단순화하면:

```text
React Component
      |
      | useHistory()
      v
Router의 History 접근
      |
      v
history 객체
      |
      | push("/about")
      v
React Router
      |
      | History 변경
      v
Browser History API
      |
      v
URL 변경
      |
      v
Router가 새로운 location 반영
      |
      v
Route Matching
      |
      v
React 렌더링
      |
      v
화면 변경
```

핵심은 다음입니다.

> `history.push()`는 단순히 주소창의 문자열만 변경하는 것이 아니라, React Router의 네비게이션 흐름을 시작하여 새로운 URL에 맞는 UI가 렌더링되도록 합니다.

---

# 16. React Router v6에서 `useHistory`가 사라진 이유

React Router v6에서는 `useHistory` 대신 `useNavigate`를 사용합니다.

v5:

```jsx
const history = useHistory();

history.push('/about');
```

v6+:

```jsx
const navigate = useNavigate();

navigate('/about');
```

뒤로 가기도 변경되었습니다.

v5:

```jsx
history.goBack();
```

v6+:

```jsx
navigate(-1);
```

`replace()`도 옵션 형태로 변경할 수 있습니다.

v5:

```jsx
history.replace('/main');
```

v6+:

```jsx
navigate('/main', {
  replace: true
});
```

비교하면:

| React Router v5         | React Router v6+                    |
| ----------------------- | ----------------------------------- |
| `useHistory()`          | `useNavigate()`                     |
| `history.push('/a')`    | `navigate('/a')`                    |
| `history.replace('/a')` | `navigate('/a', { replace: true })` |
| `history.goBack()`      | `navigate(-1)`                      |
| `history.goForward()`   | `navigate(1)`                       |

---

# 17. 왜 `history` 객체에서 `navigate()` 함수로 바뀌었을까?

API의 관점에서 보면 v5에서는 개발자가 **History 객체 자체를 다루는 느낌**이 강했습니다.

```jsx
const history = useHistory();

history.push(...);
history.replace(...);
history.goBack();
```

반면 v6에서는 개발자가 실제로 원하는 행위인 **"네비게이션" 자체에 집중하도록 API가 단순화**되었습니다.

```jsx
const navigate = useNavigate();

navigate('/about');
navigate('/login', { replace: true });
navigate(-1);
```

개념적으로:

```text
React Router v5

useHistory()
     |
     v
history 객체
     |
 ┌───┼────────┐
push replace goBack


React Router v6+

useNavigate()
     |
     v
navigate()
     |
 ┌───┼─────────────┐
URL  replace       delta
```

따라서 현대 React Router를 학습한다면 `useHistory` 자체보다 **Programmatic Navigation이라는 개념을 이해한 뒤 `useNavigate`로 연결하는 것**이 중요합니다.

---

# 18. `<Link>`, `<NavLink>`, `useHistory`의 관계

세 가지를 함께 비교하면 React Router의 이동 방식을 이해하기 쉽습니다.

| 기능               | `<Link>` | `<NavLink>`   | `useHistory`      |
| ---------------- | -------- | ------------- | ----------------- |
| 기본 목적            | 링크 이동    | 링크 이동 + 활성 상태 | 코드 기반 이동          |
| 사용자 클릭           | 주로 필요    | 주로 필요         | 반드시 필요하지 않음       |
| JSX 컴포넌트         | O        | O             | X                 |
| 현재 경로 활성화        | X        | O             | X                 |
| 조건에 따른 이동        | 제한적      | 제한적           | 적합                |
| 대표 용도            | 일반 링크    | 메뉴 / 탭        | 로그인 / 저장 완료       |
| React Router v6+ | O        | O             | `useNavigate`로 대체 |

결국 세 가지는 다음처럼 구분할 수 있습니다.

```text
사용자가 링크를 눌러 이동
        |
        +---- 일반 링크 ----> <Link>
        |
        +---- 현재 메뉴 표시 -> <NavLink>


JavaScript 로직에 의해 이동
        |
        +---- React Router v5 ---> useHistory()
        |
        +---- React Router v6+ --> useNavigate()
```

---

# 19. 한 문장으로 정의하면

> `useHistory`는 React Router v5에서 함수 컴포넌트가 Router의 History 객체에 접근하여 `push`, `replace`, 뒤로 가기 등의 명령형 네비게이션을 수행할 수 있도록 제공했던 Hook이며, React Router v6부터는 `useNavigate`로 대체되었습니다.

---

# 핵심 정리

| 항목            | 설명                          |
| ------------- | --------------------------- |
| `useHistory`  | React Router v5의 Hook       |
| 목적            | 코드에서 페이지 이동                 |
| 반환값           | Router가 사용하는 history 객체     |
| `push()`      | 새로운 History Entry를 추가하고 이동  |
| `replace()`   | 현재 History Entry를 교체하고 이동   |
| `goBack()`    | 이전 History Entry로 이동        |
| `goForward()` | 다음 History Entry로 이동        |
| `go(n)`       | History Stack에서 상대적인 위치로 이동 |
| 페이지 새로고침      | 일반적인 SPA 네비게이션에서는 발생하지 않음   |
| v6 이후         | `useNavigate`로 대체           |

가장 중요한 흐름은 다음과 같습니다.

```text
JavaScript 로직
      |
      v
history.push("/about")
      |
      v
React Router
      |
      v
History / Location 변경
      |
      v
Route Matching
      |
      v
React UI 렌더링
```

따라서 `useHistory`의 핵심을 단순히 **"페이지를 이동시키는 Hook"**이라고만 기억하기보다는,

> **React Router v5에서 컴포넌트가 Router의 History에 접근하여 코드로 네비게이션을 명령할 수 있게 해주는 Hook**

이라고 이해하는 것이 더 정확합니다.
