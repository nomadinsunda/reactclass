# 🧩 `React.Suspense`란?

> `React.Suspense`는 **지연(lazy) 로딩되는 컴포넌트나 비동기 데이터가 준비될 때까지의 UI를 제어하는 React의 내장 컴포넌트**입니다.

즉,

* 컴포넌트가 **아직 준비되지 않았을 때**
* 사용자에게 보여줄 **"대기 상태의 UI (fallback)"** 를 지정하는 용도입니다.

---

## ✅ 주요 목적 2가지

| 목적                         | 설명                                             |
| -------------------------- | ---------------------------------------------- |
| 1️⃣ Lazy-loaded 컴포넌트 대기    | `React.lazy()`로 불러오는 컴포넌트가 로드되기 전 보여줄 UI 제공    |
| 2️⃣ 비동기 데이터 대기 (React 18+) | `use`, `await` 같은 Suspense-aware 데이터 로딩과 함께 작동 |

---

# 🧪 기본 사용 예시 (React.lazy와 함께)

```jsx
import React, { Suspense, lazy } from 'react';

const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Profile />
    </Suspense>
  );
}
```

### 📌 위 동작 설명:

1. `Profile` 컴포넌트는 `import()`로 분리되어 별도 청크로 로딩됩니다.
2. 해당 파일이 네트워크에서 로딩되기 전까지는 렌더링 불가능.
3. 그 동안은 `<div>Loading...</div>`이 대신 보여짐.
4. 로딩 완료 시, 자동으로 `<Profile />`로 교체됩니다.

---

# 🛠 작동 메커니즘

```jsx
<Suspense fallback={<FallbackUI />}>
  <LazyOrAsyncComponent />
</Suspense>
```

* 내부에서 Promise가 `pending` 상태인 경우:

  * React는 \*\*렌더링을 "중단"\*\*하고 `fallback` UI만 렌더링
* Promise가 `resolved` 되면:

  * 원래 컴포넌트를 **다시 렌더링**

---

## 🔧 Suspense + Lazy: 코드 스플리팅 용도

```jsx
const Chart = lazy(() => import('./Chart'));

<Suspense fallback={<SkeletonChart />}>
  <Chart />
</Suspense>
```

이 방식은 코드 스플리팅을 통해 **초기 번들 크기를 줄이고**,
컴포넌트는 **필요할 때만 로드되도록** 할 수 있습니다.

---

## 🌐 Suspense + 비동기 데이터 (React 18+)

React 18부터는 서버 컴포넌트/streaming을 활용하여 비동기 데이터도 Suspense로 처리할 수 있게 되었습니다.

```jsx
<Suspense fallback={<LoadingUser />}>
  <UserInfo userId={1} />
</Suspense>
```

그리고 내부적으로 `UserInfo`는 비동기적으로 데이터를 fetch한 후 Promise를 throw하는 방식으로 Suspense와 동기화됩니다.

> ⚠️ 단, 이 기능은 `use()`, `createFromFetch()`, React Server Components 등과 연계된 최신 API에서만 작동합니다.

---

# ⚠️ 사용 시 주의할 점

| 항목                              | 설명                                                             |
| ------------------------------- | -------------------------------------------------------------- |
| 반드시 fallback 필요                 | fallback을 지정하지 않으면 React가 경고 출력                                |
| 내부에 lazy 컴포넌트 or 비동기 컴포넌트 있어야 함 | 정적 컴포넌트만 있을 경우 Suspense는 무의미                                   |
| 중첩 가능                           | 여러 Suspense를 중첩하여 "스켈레톤 뷰" 구조 설계 가능                            |
| 에러는 처리 못 함                      | Suspense는 에러 처리가 아니라 loading UI 제어용, 에러는 ErrorBoundary로 처리해야 함 |

---

## 🧠 실전 설계 팁

| 전략                             | 설명                                    |
| ------------------------------ | ------------------------------------- |
| 🧩 페이지 단위 Suspense             | 라우터에서 페이지 컴포넌트를 lazy load             |
| 🪟 UI 영역별 Suspense 분리          | 탭, 사이드바, 차트 등 개별 Suspense fallback 적용 |
| 💣 ErrorBoundary + Suspense 조합 | 로딩 중 실패 상황도 제어 가능                     |

```jsx
<ErrorBoundary>
  <Suspense fallback={<Loading />}>
    <AsyncComponent />
  </Suspense>
</ErrorBoundary>
```

---

## 🧾 요약

| 항목 | 설명                                            |
| -- | --------------------------------------------- |
| 정의 | 로딩 중인 컴포넌트를 대체하는 fallback UI 지정               |
| 연동 | `React.lazy()`, React 18의 async data fetching |
| 효과 | 사용자에게 스켈레톤 or 스피너 제공, UX 향상                   |
| 제약 | 반드시 fallback 필요, 에러 처리는 별도로                   |
| 확장 | 중첩 사용 가능, 서버 스트리밍과 연계 가능 (RSC)                |

---

## 📌 실제로 언제 쓰나요?

* 코드 스플리팅할 때 (`React.lazy`)
* 데이터가 비동기일 때 (`use`, `defer`, server components)
* 사용자에게 **"빈 화면" 대신 뭔가 표시**하고 싶을 때


