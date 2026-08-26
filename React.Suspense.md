# React `<Suspense>` — 준비되지 않은 UI를 다루는 렌더링 경계

React에서 `<Suspense>`를 단순히 **"로딩 화면을 보여주는 컴포넌트"**라고 설명하면 핵심을 놓치기 쉽습니다.

`Suspense`의 본질은 로딩 스피너 자체가 아닙니다.

핵심은:

> **React가 현재 렌더링하려는 UI의 일부가 아직 준비되지 않았을 때, 그 부분의 렌더링을 잠시 보류하고 대신 `fallback` UI를 보여줄 수 있도록 만드는 경계(boundary)입니다.**

가장 기본적인 모습은 다음과 같습니다.

```jsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

개념적으로:

```text
SomeComponent 렌더링
        │
        ▼
   준비되었는가?
        │
    ┌───┴────┐
   YES       NO
    │         │
    ▼         ▼
실제 UI     Suspend
              │
              ▼
         가장 가까운
         <Suspense>
              │
              ▼
           fallback
```

그리고 준비가 끝나면 React가 다시 렌더링을 시도하여 실제 UI를 보여줍니다.

---

# 1. 왜 Suspense가 필요한가?

React 애플리케이션에서는 렌더링하려는 UI가 즉시 준비되지 않는 상황이 발생할 수 있습니다.

대표적인 예가:

```text
Code Splitting된 Component
데이터를 기다리는 Component
Streaming SSR의 일부 UI
```

입니다.

예를 들어:

```jsx
const Profile = lazy(() => import("./Profile"));
```

`Profile` 코드가 아직 브라우저에 다운로드되지 않았다고 해보겠습니다.

React는:

```jsx
<Profile />
```

을 렌더링하려고 하지만 필요한 JavaScript 모듈이 아직 없습니다.

즉:

```text
React
  │
  │ <Profile /> 렌더링 요청
  ▼
Profile 코드가 있는가?
  │
  └── NO
       │
       ▼
   지금은 렌더링 불가능
```

합니다.

이때 React에게 필요한 것이:

> **"이 UI가 준비될 때까지 무엇을 보여줄 것인가?"**

입니다.

그 역할을 맡는 것이 `<Suspense>`입니다.

```jsx
<Suspense fallback={<p>프로필 로딩 중...</p>}>
  <Profile />
</Suspense>
```

---

# 2. Suspense는 "로딩 상태 변수"가 아니다

처음 Suspense를 배우면 다음과 비슷하다고 생각하기 쉽습니다.

```jsx
if (loading) {
  return <Loading />;
}

return <Profile />;
```

하지만 Suspense의 정신 모델은 다릅니다.

일반적인 로딩 상태 처리는 개발자가 직접:

```text
loading = true
      │
      ▼
Loading UI
      │
      ▼
loading = false
      │
      ▼
실제 UI
```

를 관리합니다.

Suspense에서는:

```text
React Rendering
      │
      ▼
자식 UI가 Suspend
      │
      ▼
가장 가까운 Suspense Boundary
      │
      ▼
fallback 표시
      │
      ▼
준비 완료
      │
      ▼
React가 다시 렌더링
      │
      ▼
실제 UI
```

의 형태가 됩니다.

즉 Suspense는 개발자가 별도의 `loading` state를 하나씩 관리하는 API라기보다 **React의 렌더링 시스템에서 "아직 준비되지 않은 UI"를 처리하는 구조**입니다.

---

# 3. `fallback`은 무엇인가?

`fallback`은 자식 UI가 아직 준비되지 않았을 때 대신 보여줄 React Node입니다.

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <Profile />
</Suspense>
```

다음처럼 사용할 수도 있습니다.

```jsx
<Suspense fallback={<Spinner />}>
  <Profile />
</Suspense>
```

또는:

```jsx
<Suspense fallback={<ProfileSkeleton />}>
  <Profile />
</Suspense>
```

화면 흐름은:

```text
Profile 준비 전

┌───────────────────────┐
│ ProfileSkeleton       │
│                       │
│ ▓▓▓▓▓                 │
│ ▓▓▓▓▓▓▓▓              │
└───────────────────────┘


        ↓ 준비 완료


┌───────────────────────┐
│ Alice                 │
│ alice@example.com     │
└───────────────────────┘
```

입니다.

따라서 `fallback`은 보통:

* Loading Spinner
* Skeleton UI
* Placeholder
* 간단한 Loading Message

등을 사용합니다.

---

# 4. 가장 이해하기 쉬운 예제 — `React.lazy()`

Suspense의 동작을 처음 이해하기에는 `React.lazy()`와 함께 보는 것이 가장 좋습니다.

```jsx
import { lazy, Suspense } from "react";

const Profile = lazy(() => import("./Profile"));

export default function App() {
  return (
    <Suspense fallback={<p>프로필 로딩 중...</p>}>
      <Profile />
    </Suspense>
  );
}
```

여기에는 역할이 세 개 있습니다.

```text
import()
   │
   └── Profile Module을 비동기로 로드


lazy()
   │
   └── 비동기 Module Loading을
       React Component 렌더링과 연결


Suspense
   │
   └── Profile이 준비되지 않은 동안
       fallback을 표시
```

이 역할을 섞지 않는 것이 중요합니다.

---

# 5. `React.lazy()` + Suspense의 실제 흐름

다음 코드를 기준으로 보겠습니다.

```jsx
const Profile = lazy(
  () => import("./Profile")
);
```

그리고:

```jsx
<Suspense fallback={<Loading />}>
  <Profile />
</Suspense>
```

React가 `<Profile />`을 렌더링하려는 순간:

```text
① React 렌더링 시작
       │
       ▼
② <Profile /> 발견
       │
       ▼
③ Profile Module 확인
       │
       ▼
④ 아직 준비되지 않음
       │
       ▼
⑤ Module Loading 시작
       │
       ▼
⑥ 현재 렌더링 Suspend
       │
       ▼
⑦ 가장 가까운 <Suspense> 탐색
       │
       ▼
⑧ fallback 렌더링
```

동시에 브라우저에서는:

```text
import("./Profile")
       │
       ▼
Profile Chunk 요청
       │
       ▼
Network
       │
       ▼
Chunk Download
       │
       ▼
Module 준비 완료
```

가 진행됩니다.

그리고 준비가 끝나면:

```text
Module 준비 완료
       │
       ▼
React가 렌더링 재시도
       │
       ▼
<Profile /> 렌더링 가능
       │
       ▼
fallback 제거
       │
       ▼
실제 Profile UI 표시
```

가 됩니다.

전체를 합치면:

```text
                    <Profile />
                         │
                         ▼
                  아직 준비되지 않음
                         │
               ┌─────────┴──────────┐
               │                    │
               ▼                    ▼
         React Suspend         Module Loading
               │                    │
               ▼                    ▼
          <Suspense>            Profile Chunk
               │                    │
               ▼                    ▼
          fallback 표시          다운로드 완료
               │                    │
               └─────────┬──────────┘
                         ▼
                   React 재렌더링
                         │
                         ▼
                    <Profile />
```

---

# 6. Suspense Boundary라는 개념

`<Suspense>`에서 매우 중요한 단어가 **Boundary(경계)**입니다.

```jsx
<Suspense fallback={<Loading />}>
  <A />
  <B />
  <C />
</Suspense>
```

이 Suspense는:

```text
Suspense Boundary
┌─────────────────────┐
│                     │
│ A                   │
│ B                   │
│ C                   │
│                     │
└─────────────────────┘
```

라는 하나의 렌더링 경계를 만듭니다.

이 안에서 UI가 suspend하면 React는 가장 가까운 Suspense boundary를 사용합니다.

핵심 규칙:

> **Suspend한 컴포넌트에서 위쪽으로 올라가며 가장 가까운 Suspense Boundary가 해당 로딩 상태를 처리합니다.**

---

# 7. Boundary의 크기가 중요한 이유

다음 구조를 보겠습니다.

```jsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Sidebar />
  <Dashboard />
</Suspense>
```

`Dashboard`만 준비되지 않았다고 해도 경계가 페이지 전체를 감싸고 있기 때문에 넓은 영역이 fallback으로 전환될 수 있습니다.

```text
Before

┌──────────────────────────┐
│ Header                   │
├────────┬─────────────────┤
│Sidebar │ Dashboard       │
└────────┴─────────────────┘


Dashboard Suspend


┌──────────────────────────┐
│                          │
│      PageSkeleton        │
│                          │
└──────────────────────────┘
```

따라서 상황에 따라 Boundary를 더 작게 만들 수 있습니다.

```jsx
<>
  <Header />

  <Sidebar />

  <Suspense fallback={<DashboardSkeleton />}>
    <Dashboard />
  </Suspense>
</>
```

그러면:

```text
┌──────────────────────────┐
│ Header                   │
├────────┬─────────────────┤
│Sidebar │                 │
│        │ Dashboard       │
│        │ Skeleton        │
└────────┴─────────────────┘
```

처럼 특정 부분만 fallback으로 대체할 수 있습니다.

---

# 8. 여러 Suspense Boundary를 사용할 수도 있다

대시보드가 다음 세 영역으로 구성되어 있다고 하겠습니다.

```text
Dashboard
│
├── Summary
├── Chart
└── Activity
```

각 영역의 준비 시간이 다르다면:

```jsx
function Dashboard() {
  return (
    <>
      <Suspense fallback={<SummarySkeleton />}>
        <Summary />
      </Suspense>

      <Suspense fallback={<ChartSkeleton />}>
        <Chart />
      </Suspense>

      <Suspense fallback={<ActivitySkeleton />}>
        <Activity />
      </Suspense>
    </>
  );
}
```

처럼 나눌 수 있습니다.

그러면:

```text
Summary    → 준비됨
Chart      → 아직 준비 안 됨
Activity   → 준비됨
```

일 때:

```text
┌────────────────────────┐
│ Summary                │
│ 실제 내용              │
├────────────────────────┤
│ ChartSkeleton          │
│ ▓▓▓▓▓▓▓                │
├────────────────────────┤
│ Activity               │
│ 실제 내용              │
└────────────────────────┘
```

처럼 일부 영역만 fallback을 표시할 수 있습니다.

이것이 Suspense Boundary를 설계하는 중요한 이유입니다.

---

# 9. Nested Suspense

Suspense Boundary는 중첩할 수도 있습니다.

```jsx
<Suspense fallback={<PageSkeleton />}>
  <Page>

    <Suspense fallback={<SidebarSkeleton />}>
      <Sidebar />
    </Suspense>

    <MainContent />

  </Page>
</Suspense>
```

구조:

```text
Outer Suspense
│
└── Page
    │
    ├── Inner Suspense
    │       │
    │       └── Sidebar
    │
    └── MainContent
```

`Sidebar`가 suspend하면 가장 가까운 boundary는:

```text
Inner Suspense
```

입니다.

따라서:

```text
Page
├── SidebarSkeleton
└── MainContent
```

처럼 처리할 수 있습니다.

하지만 Inner Boundary 밖의 어떤 컴포넌트가 suspend하면 Outer Boundary가 사용될 수 있습니다.

핵심은:

> **React는 suspend한 위치에서 가장 가까운 상위 Suspense Boundary를 찾습니다.**

---

# 10. Suspense는 모든 비동기 작업을 자동으로 감지하는 것이 아니다

여기서 매우 중요한 오해가 있습니다.

다음 코드를 Suspense가 자동으로 처리한다고 생각하면 안 됩니다.

```jsx
useEffect(() => {
  fetch("/api/users")
    .then(...)
}, []);
```

`fetch()`가 존재한다고 해서 Suspense가 자동으로:

```text
fetch 시작
   ↓
Suspense fallback
```

으로 바뀌는 것은 아닙니다.

Suspense는 **Suspense와 통합된 방식으로 렌더링이 suspend될 때** 동작합니다.

대표적으로:

```text
React.lazy()

Suspense-aware Framework

Suspense-aware Data Source

React의 use(Promise)
```

등과 연결될 수 있습니다.

따라서:

> **Suspense는 "모든 Promise를 자동으로 감시하는 컴포넌트"가 아닙니다.**

이 점이 매우 중요합니다.

---

# 11. React 19에서는 `use(Promise)`도 Suspense와 연결된다

React 19에서는 `use()`를 사용하여 Promise의 결과를 렌더링 과정에서 읽을 수 있습니다.

개념적인 예:

```jsx
import { use, Suspense } from "react";

function User({ userPromise }) {
  const user = use(userPromise);

  return <h2>{user.name}</h2>;
}

function Page({ userPromise }) {
  return (
    <Suspense fallback={<p>사용자 로딩 중...</p>}>
      <User userPromise={userPromise} />
    </Suspense>
  );
}
```

`User`가:

```jsx
use(userPromise)
```

를 실행했는데 Promise가 아직 pending이면:

```text
User Rendering
     │
     ▼
use(userPromise)
     │
     ▼
Promise Pending
     │
     ▼
Suspend
     │
     ▼
<Suspense>
     │
     ▼
fallback
```

이 됩니다.

Promise가 resolve되면:

```text
Promise Resolve
      │
      ▼
React 재렌더링
      │
      ▼
use(userPromise)
      │
      ▼
user 반환
      │
      ▼
실제 UI
```

가 됩니다.

따라서 현재 Suspense를 설명할 때는:

```text
React.lazy()
+
React 19 use(Promise)
+
Suspense-aware Framework/Data Source
```

라는 그림을 함께 알고 있는 것이 좋습니다.

다만 Client Component에서 렌더링할 때마다 새 Promise를 만드는 방식은 피해야 하며, 실제 데이터 패칭에서는 Suspense를 지원하는 프레임워크나 라이브러리의 캐싱 메커니즘을 사용하는 것이 일반적입니다.

---

# 12. Promise를 "throw한다"는 말은 어떻게 이해해야 할까?

Suspense 내부 원리를 설명할 때 흔히 다음 표현을 사용합니다.

> **"컴포넌트가 Promise를 throw하면 Suspense가 잡는다."**

정신 모델을 이해하는 데는 유용하지만, 입문 단계에서는 이것을 일반적인 JavaScript 예외 처리와 완전히 같은 개념으로 받아들이지 않는 것이 좋습니다.

핵심적으로 이해해야 할 것은:

```text
렌더링 시작
   │
   ▼
현재 UI를 완성할 수 없음
   │
   ▼
React에게
"이 렌더링은 아직 준비되지 않았다"
알려짐
   │
   ▼
가장 가까운 Suspense Boundary
   │
   ▼
fallback
```

입니다.

즉 입문 단계에서는:

> **"렌더링에 필요한 리소스가 준비되지 않으면 해당 렌더링이 suspend되고, 가장 가까운 Suspense가 fallback을 보여준다."**

라고 이해하는 것이 더 좋습니다.

`Promise throw`는 그 동작을 이해하기 위한 **내부 구현에 가까운 정신 모델**로 뒤에서 배우면 됩니다.

---

# 13. Suspense와 Error Boundary의 차이

둘은 모두 "Boundary"라는 공통점이 있지만 목적이 다릅니다.

| 구분    | Suspense            | Error Boundary |
| ----- | ------------------- | -------------- |
| 목적    | 아직 준비되지 않은 UI 처리    | 실패한 UI 처리      |
| 상태    | Pending / Suspended | Error          |
| 화면    | Loading / Skeleton  | Error UI       |
| 대표 상황 | Chunk 또는 데이터 준비 중   | 렌더링/로딩 실패      |

개념적으로:

```text
Component
   │
   ├── 아직 준비 안 됨
   │        │
   │        ▼
   │     Suspense
   │        │
   │        ▼
   │     Loading UI
   │
   └── Error 발생
            │
            ▼
       Error Boundary
            │
            ▼
         Error UI
```

실전에서는 두 개를 함께 사용할 수도 있습니다.

```jsx
<ErrorBoundary fallback={<ErrorPage />}>
  <Suspense fallback={<Loading />}>
    <Profile />
  </Suspense>
</ErrorBoundary>
```

그러면:

```text
Profile 준비 중
     ↓
Loading


Profile 준비 완료
     ↓
Profile UI


Profile 로딩/렌더링 실패
     ↓
ErrorPage
```

처럼 역할이 명확하게 나뉩니다.

---

# 14. `startTransition`과 Suspense

Suspense의 중요한 문제 중 하나는 **이미 화면에 보이던 UI가 다시 fallback으로 바뀌는 상황**입니다.

예를 들어 사용자가 페이지 A를 보고 있다고 하겠습니다.

```text
Page A
```

그리고 Page B로 이동하는 업데이트가 발생했는데 Page B가 아직 준비되지 않았습니다.

단순하게 처리하면:

```text
Page A
   ↓
Loading...
   ↓
Page B
```

처럼 이미 보이던 UI가 갑자기 사라질 수 있습니다.

`startTransition()`은 어떤 업데이트를 **긴급하지 않은 UI 전환**으로 표시하는 데 사용할 수 있습니다.

```jsx
startTransition(() => {
  setPage("B");
});
```

개념적으로:

```text
현재 Page A 표시
      │
      ▼
Page B로 Transition 시작
      │
      ▼
Page B가 Suspend
      │
      ▼
가능하면 기존 Page A를 유지
      │
      ▼
Page B 준비
      │
      ▼
Page B로 전환
```

즉 `startTransition()` 자체가 데이터를 로드하는 것도 아니고 Suspense를 만드는 것도 아닙니다.

역할은:

```text
Suspense
→ 준비되지 않은 UI의 경계 처리


startTransition
→ 이 업데이트는 긴급하지 않은 전환이라고 React에 알림
```

입니다.

두 기능이 함께 사용되면 갑작스럽게 기존 UI 전체를 fallback으로 바꾸는 현상을 줄이는 데 도움이 됩니다.

---

# 15. `useDeferredValue`와 Suspense

검색창을 생각해보겠습니다.

```text
사용자 입력
react
react r
react ro
react rou
...
```

텍스트 입력은 즉각적으로 반응하는 것이 좋습니다.

하지만 검색 결과 영역은 조금 늦게 업데이트되어도 괜찮을 수 있습니다.

```jsx
const deferredQuery = useDeferredValue(query);
```

그러면 개념적으로:

```text
query
│
└── 사용자의 최신 입력


deferredQuery
│
└── 조금 뒤따라가는 값
```

으로 사용할 수 있습니다.

```jsx
<input
  value={query}
  onChange={e => setQuery(e.target.value)}
/>

<Suspense fallback={<SearchSkeleton />}>
  <SearchResults query={deferredQuery} />
</Suspense>
```

역할을 구분하면:

```text
useDeferredValue
→ 느려도 되는 값을 지연시킴

Suspense
→ 결과 UI가 suspend하는 경우
   그 영역의 렌더링 경계를 처리
```

입니다.

---

# 16. Suspense와 Streaming SSR

Suspense는 클라이언트 코드 스플리팅에서만 사용하는 개념이 아닙니다.

Streaming SSR에서도 중요한 역할을 합니다.

기존 SSR을 아주 단순하게 생각하면:

```text
Server
  │
  │ 전체 HTML 준비
  ▼
Browser
```

처럼 전체 HTML을 준비한 다음 보내는 모델을 떠올릴 수 있습니다.

Streaming SSR에서는:

```text
Server
  │
  ├── 준비된 HTML 먼저 전송
  │
  ├── 준비 안 된 영역은 fallback
  │
  └── 나중에 준비된 내용을 추가 전송
  ▼
Browser
```

가 가능합니다.

화면 관점에서는:

```text
①

┌──────────────────────┐
│ Header               │
├──────────────────────┤
│ Loading...           │
└──────────────────────┘


②

┌──────────────────────┐
│ Header               │
├──────────────────────┤
│ 실제 Content         │
└──────────────────────┘
```

처럼 페이지가 점진적으로 채워지는 UX를 만들 수 있습니다.

Suspense Boundary는 서버에서도:

> **"이 부분은 아직 준비되지 않았으므로 우선 fallback을 제공하자."**

라는 렌더링 경계 역할을 할 수 있습니다.

---

# 17. Suspense Boundary를 어떻게 설계해야 할까?

Suspense를 많이 넣는 것이 무조건 좋은 것은 아닙니다.

너무 크게 잡으면:

```text
작은 Component 하나 Suspend
        ↓
페이지 전체 fallback
```

이 될 수 있습니다.

너무 작게 잡으면:

```text
Loading
Loading
Loading
Loading
```

처럼 여러 fallback이 짧게 반복되어 화면이 산만해질 수 있습니다.

따라서 Boundary는 **사용자가 하나의 UI 단위로 인식하는 영역**을 기준으로 설계하는 것이 좋습니다.

예:

```text
Dashboard
│
├── Header
│
├── Summary      ← Suspense Boundary
│
├── Chart        ← Suspense Boundary
│
└── Activity     ← Suspense Boundary
```

또는:

```text
App
│
└── Route Page   ← Suspense Boundary
```

처럼 사용할 수 있습니다.

---

# 18. 실전에서 기억해야 할 점

첫째, Suspense를 **Loading State 관리 라이브러리**라고 생각하지 않는 것이 좋습니다.

Suspense는:

```text
렌더링 가능한가?
```

를 중심으로 동작하는 **렌더링 경계**입니다.

둘째, `fetch()`나 Promise가 있다고 자동으로 Suspense가 작동하는 것은 아닙니다.

```text
일반 fetch
≠
자동 Suspense
```

Suspense와 통합된 데이터 소스가 필요합니다.

셋째, Boundary의 위치가 UX를 결정합니다.

```text
Boundary가 크다
→ 넓은 영역 fallback

Boundary가 작다
→ 일부 영역 fallback
```

넷째, 로딩과 실패는 구분해야 합니다.

```text
Pending
→ Suspense

Error
→ Error Boundary
```

다섯째, 이미 표시된 화면을 새로운 fallback으로 덮는 것이 불편하다면:

```text
startTransition
useDeferredValue
```

같은 API와 Suspense의 관계를 함께 생각할 수 있습니다.

---

# 19. 전체 동작을 한 번에 이해하기

다음 코드가 있다고 하겠습니다.

```jsx
const Profile = lazy(
  () => import("./Profile")
);

function App() {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <Profile />
    </Suspense>
  );
}
```

전체 흐름:

```text
React Rendering
      │
      ▼
 <Profile />
      │
      ▼
Profile Module 필요
      │
      ▼
아직 준비 안 됨
      │
      ├─────────────────────┐
      │                     │
      ▼                     ▼
렌더링 Suspend        Module Loading
      │                     │
      ▼                     ▼
가장 가까운             Profile Chunk
<Suspense>                 │
      │                     ▼
      ▼                 준비 완료
ProfileSkeleton             │
      │                     │
      └──────────┬──────────┘
                 ▼
           React 렌더링 재시도
                 │
                 ▼
             <Profile />
                 │
                 ▼
              실제 UI
```

이 그림이 Suspense의 핵심입니다.

---

# 20. 최종 정리

`Suspense`를 가장 간단하게 설명하면:

> **`<Suspense>`는 자식 UI가 아직 렌더링할 준비가 되지 않아 suspend할 때, 가장 가까운 경계에서 대신 `fallback`을 보여주고 준비가 완료되면 실제 UI 렌더링을 다시 시도하도록 하는 React의 렌더링 Boundary입니다.**

더 짧게 말하면:

> **"아직 준비되지 않은 UI를 다루는 렌더링 경계"**

입니다.

전체 구조를 기억하면:

```text
               React Rendering
                      │
                      ▼
               Component Tree
                      │
                      ▼
              준비되지 않은 UI
                      │
                      ▼
                   Suspend
                      │
                      ▼
          가장 가까운 <Suspense>
                      │
                      ▼
                   fallback
                      │
              준비 완료 │
                      ▼
                React Retry
                      │
                      ▼
                  실제 UI
```

그리고 다른 API와 역할을 구분하면 더욱 명확합니다.

```text
React.lazy()
→ Component Code를 지연 로딩과 연결

use(Promise)
→ Promise 결과를 렌더링에서 읽고
  필요하면 suspend

Suspense
→ Suspend된 UI의 Boundary와 fallback 담당

startTransition()
→ 긴급하지 않은 UI 전환 표시

useDeferredValue()
→ 값의 업데이트를 지연

ErrorBoundary
→ Error 처리
```

따라서 Suspense의 본질은 **"로딩 화면"이 아니라 "렌더링 경계"**입니다.

React가 UI를 그리다가:

> **"이 부분은 아직 준비되지 않았다."**

라는 상황을 만나면,

> **"그럼 이 경계까지만 fallback으로 보여주고, 준비되면 다시 렌더링하자."**

라고 처리할 수 있게 만드는 것이 `<Suspense>`입니다.
