#  `<NavLink>`란?

`<NavLink>`는 React Router가 제공하는 **네비게이션 전용 링크 컴포넌트**입니다.

기본적인 페이지 이동 기능은 `<Link>`와 동일하지만, 한 가지 중요한 기능이 추가되어 있습니다.

> **현재 URL이 자신이 가리키는 경로와 일치하는지를 판단하여 `isActive` 상태를 제공한다.**

즉,

```text
<Link>
경로 이동

<NavLink>
경로 이동 + 현재 경로와의 일치 여부 감지
```

따라서 `<NavLink>`는 **현재 사용자가 어느 메뉴에 있는지 시각적으로 표시해야 하는 UI**에서 특히 유용합니다.

```jsx
import { NavLink } from 'react-router-dom';

<NavLink to="/about">
  About
</NavLink>
```

---

# 1. `<NavLink>`는 왜 필요한가?

다음과 같은 네비게이션 메뉴가 있다고 생각해 봅시다.

```text
Home    About    Contact
         ↑
      현재 위치
```

현재 URL이 `/about`이라면 사용자는 **About 메뉴가 현재 선택된 메뉴라는 것을 알아야 합니다.**

예를 들어 다음과 같이 표현할 수 있습니다.

```text
Home    [ About ]    Contact
          ACTIVE
```

일반적인 `<Link>`는 URL을 변경하는 역할을 하지만, **현재 URL과 자신의 경로가 일치하는지를 이용한 UI 상태 표현은 직접 처리해야 합니다.**

반면 `<NavLink>`는 React Router의 현재 location을 기준으로 자신의 `to`와 비교하여 **`isActive` 상태를 제공**합니다.

```text
현재 URL
   │
   │ /about
   ▼
React Router
   │
   │ 현재 URL과 to 비교
   ▼
<NavLink to="/about">
   │
   └── isActive = true
```

그래서 네비게이션 바, 탭, 사이드바처럼 **"현재 위치"를 표현해야 하는 링크**에 적합합니다.

---

# 2. 기본 사용법

```jsx
<NavLink to="/home">
  Home
</NavLink>
```

브라우저에서는 최종적으로 `<a>` 엘리먼트 기반의 링크로 렌더링됩니다.

개념적으로 보면 다음과 같습니다.

```html
<a href="/home">
  Home
</a>
```

하지만 일반 `<a>`와 달리 클릭 시 React Router가 네비게이션을 처리하므로 일반적인 SPA 이동에서는 **문서 전체를 새로 요청하지 않고 클라이언트 측 라우팅**이 이루어집니다.

그리고 `<NavLink>`는 현재 URL과 `to`의 경로를 비교하여 활성 상태를 판단합니다.

```text
현재 URL: /home

<NavLink to="/home">
          └──────┘
             ↓
          경로 일치
             ↓
      isActive = true
```

---

# 3. 핵심: `isActive`

`<NavLink>`를 이해할 때 가장 중요한 값이 `isActive`입니다.

```jsx
<NavLink
  to="/about"
  className={({ isActive }) =>
    isActive ? 'nav-link active' : 'nav-link'
  }
>
  About
</NavLink>
```

여기서 `className`에 문자열이 아니라 **함수**를 전달했습니다.

React Router는 이 함수를 호출하면서 현재 링크의 상태를 전달합니다.

```jsx
({ isActive }) => ...
```

현재 URL이 `/about`이라면:

```text
현재 URL = /about
to       = /about

        ↓ 비교

isActive = true
```

따라서 다음 표현식이 실행됩니다.

```jsx
isActive
  ? 'nav-link active'
  : 'nav-link'
```

결과:

```text
nav-link active
```

반대로 현재 URL이 `/contact`라면:

```text
현재 URL = /contact
to       = /about

        ↓ 불일치

isActive = false
```

결과:

```text
nav-link
```

---

# 4. `className`을 함수로 사용하기

가장 일반적인 사용 방법입니다.

```jsx
<NavLink
  to="/about"
  className={({ isActive }) =>
    isActive ? 'nav-link active' : 'nav-link'
  }
>
  About
</NavLink>
```

CSS는 다음처럼 작성할 수 있습니다.

```css
.nav-link {
  color: gray;
  text-decoration: none;
}

.nav-link.active {
  color: blue;
  font-weight: bold;
}
```

현재 URL이 `/about`이면 `isActive`가 `true`가 되고 `active` 클래스가 추가됩니다.

```text
/about

   ↓

isActive = true

   ↓

className="nav-link active"
```

---

# 5. `style`을 함수로 사용하기

`style` 역시 함수를 전달할 수 있습니다.

```jsx
<NavLink
  to="/about"
  style={({ isActive }) => ({
    fontWeight: isActive ? 'bold' : 'normal',
    color: isActive ? 'red' : 'black',
  })}
>
  About
</NavLink>
```

현재 URL이 `/about`이면:

```jsx
{
  fontWeight: 'bold',
  color: 'red'
}
```

다른 URL이라면:

```jsx
{
  fontWeight: 'normal',
  color: 'black'
}
```

즉, `<NavLink>`는 단순히 active 여부를 알려줄 뿐이고, **실제로 어떤 스타일을 적용할지는 개발자가 결정합니다.**

---

# 6. `isActive`와 `isPending`

`className`이나 `style` 함수에서는 라우팅 상태에 관한 값을 받을 수 있습니다.

대표적으로 다음 두 가지가 중요합니다.

| 상태          | 의미                      |
| ----------- | ----------------------- |
| `isActive`  | 현재 URL이 이 링크의 경로와 일치하는가 |
| `isPending` | 해당 링크로의 네비게이션이 진행 중인가   |

예를 들어:

```jsx
<NavLink
  to="/users"
  className={({ isActive, isPending }) => {
    if (isPending) return 'pending';
    if (isActive) return 'active';

    return '';
  }}
>
  Users
</NavLink>
```

개념적으로는 다음과 같습니다.

```text
현재 위치
   │
   ├── 현재 이 경로인가?
   │       └── isActive
   │
   └── 이 경로로 이동 중인가?
           └── isPending
```

`isPending`은 특히 Data Router 계열의 라우팅과 데이터 로딩을 함께 사용하는 경우 유용합니다.

---

# 7. `<NavLink>`의 내부 흐름

예를 들어 다음 코드가 있다고 하겠습니다.

```jsx
<NavLink to="/about">
  About
</NavLink>
```

현재 URL이 `/about`이라면 전체 흐름은 개념적으로 다음과 같습니다.

```text
Browser URL
   │
   │ /about
   ▼
BrowserRouter / Router
   │
   │ 현재 location 관리
   ▼
NavLink
   │
   │ location.pathname과
   │ 자신의 to="/about" 비교
   ▼
경로 일치 판단
   │
   ├── 일치
   │     └── isActive = true
   │
   └── 불일치
         └── isActive = false
   │
   ▼
className / style 함수
   │
   ▼
<a> 엘리먼트 렌더링
```

즉, 핵심 흐름은 다음과 같습니다.

> **현재 location → `to`와 비교 → `isActive` 결정 → 스타일 결정 → `<a>` 렌더링**

---

# 8. `end`가 필요한 이유

`<NavLink>`에서 매우 중요한 속성 중 하나가 `end`입니다.

다음 링크가 있다고 하겠습니다.

```jsx
<NavLink to="/about">
  About
</NavLink>
```

기본적으로 `/about`뿐만 아니라 `/about` 아래의 경로에서도 활성 상태가 될 수 있습니다.

```text
/about
/about/team
/about/company
```

즉, 기본 매칭은 하위 경로까지 고려합니다.

```text
to="/about"

URL
├── /about          → Active
├── /about/team     → Active
└── /about/company  → Active
```

정확하게 `/about`일 때만 활성화하고 싶다면 `end`를 사용합니다.

```jsx
<NavLink to="/about" end>
  About
</NavLink>
```

이제 다음과 같이 동작합니다.

```text
to="/about" + end

URL
├── /about          → Active
├── /about/team     → Not Active
└── /about/company  → Not Active
```

따라서 `end`는 쉽게 말하면:

> **"여기서 경로 비교를 끝내라."**

라는 의미로 이해할 수 있습니다.

---

# 9. 특히 `/`에서 `end`가 중요한 이유

다음과 같은 메뉴를 생각해 봅시다.

```jsx
<nav>
  <NavLink to="/">Home</NavLink>
  <NavLink to="/about">About</NavLink>
  <NavLink to="/contact">Contact</NavLink>
</nav>
```

루트 경로 `/`는 모든 URL의 시작점이기 때문에 경로 매칭을 설명할 때 주의가 필요합니다.

실전에서는 Home 링크를 다음과 같이 작성하는 패턴을 자주 사용합니다.

```jsx
<NavLink to="/" end>
  Home
</NavLink>
```

의도를 명확하게 표현하면:

```text
/          → Home Active
/about     → Home Not Active
/contact   → Home Not Active
```

즉, Home 메뉴는 **정확히 `/`일 때만 활성화**되도록 만드는 것입니다.

---

# 10. `<Link>`와 `<NavLink>`의 차이

두 컴포넌트 모두 React Router에서 **클라이언트 측 네비게이션**을 수행합니다.

차이는 현재 경로에 대한 상태가 필요한가에 있습니다.

| 항목        | `<Link>` | `<NavLink>`      |
| --------- | -------- | ---------------- |
| 기본 목적     | 경로 이동    | 경로 이동 + 현재 위치 표현 |
| SPA 네비게이션 | O        | O                |
| 현재 경로 감지  | 직접 처리    | `isActive` 제공    |
| 동적 스타일    | 직접 처리    | 상태 기반 처리 용이      |
| 대표 용도     | 일반 링크    | 메뉴, 탭, 사이드바      |

따라서 선택 기준은 간단합니다.

```text
단순히 이동만 필요
      ↓
    <Link>


이동 + 현재 메뉴 표시 필요
      ↓
   <NavLink>
```

---

# 11. 실전 예제: 네비게이션 메뉴

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink
        to="/"
        end
        className={({ isActive }) =>
          isActive ? 'active' : ''
        }
      >
        Home
      </NavLink>

      <NavLink
        to="/about"
        className={({ isActive }) =>
          isActive ? 'active' : ''
        }
      >
        About
      </NavLink>

      <NavLink
        to="/contact"
        className={({ isActive }) =>
          isActive ? 'active' : ''
        }
      >
        Contact
      </NavLink>
    </nav>
  );
}

export default Navigation;
```

현재 URL이 `/about`이라면 개념적으로 다음과 같은 상태가 됩니다.

```text
Home       About       Contact
  │           │            │
false       true          false
  │           │            │
  ▼           ▼            ▼
normal      ACTIVE        normal
```

따라서 사용자는 현재 자신이 **About 페이지에 있다는 것을 메뉴를 통해 바로 확인할 수 있습니다.**

---

# 12. `<NavLink>`를 한 문장으로 정의하면

> **`<NavLink>`는 `<Link>`의 네비게이션 기능에 현재 URL과 자신의 목적지 경로가 일치하는지를 판단하는 기능을 더하여, 메뉴·탭·사이드바 등의 활성 상태 UI를 쉽게 구현할 수 있도록 만든 React Router의 링크 컴포넌트입니다.**

---

# 핵심 정리

| 항목          | 설명                              |
| ----------- | ------------------------------- |
| `<NavLink>` | 현재 경로를 인식할 수 있는 React Router 링크 |
| `to`        | 이동할 목적지                         |
| `isActive`  | 현재 경로와 링크가 일치하는지 나타냄            |
| `isPending` | 해당 경로로 네비게이션이 진행 중인지 나타냄        |
| `className` | 함수를 전달하여 상태에 따라 클래스 결정 가능       |
| `style`     | 함수를 전달하여 상태에 따라 인라인 스타일 결정 가능   |
| `end`       | 하위 경로가 아닌 지정한 경로까지 정확하게 매칭      |
| 최종 렌더링      | `<a>` 엘리먼트 기반                   |
| 주요 용도       | Navbar, Sidebar, Tab 등 현재 위치 표시 |

핵심 구조만 압축하면 다음과 같습니다.

```text
현재 URL(location)
        │
        ▼
     <NavLink>
        │
        │ 현재 URL ↔ to 비교
        ▼
 ┌───────────────┐
 │   isActive    │
 │ true / false  │
 └───────┬───────┘
         │
         ▼
 className / style
         │
         ▼
   활성 메뉴 표현
```

`<NavLink>`의 본질은 **"스타일이 있는 `<Link>`"라기보다 "현재 경로와의 관계를 알고 있는 `<Link>`"**라고 이해하는 것이 가장 정확합니다.
