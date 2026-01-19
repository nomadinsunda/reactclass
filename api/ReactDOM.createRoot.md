`createRoot`는 **React 18부터 도입된 “새 렌더링 엔진 진입점(Entry Point)”** 입니다.
한 줄 요약하면:

> 👉 `ReactDOM.render` 시대를 끝내고, **Concurrent Rendering(동시성 렌더링)** 기능을 온전히 활용하기 위한 새로운 “루트 관리자”라고 보시면 됩니다.

---

## 1. `createRoot`는 정확히 무엇인가? 🧠

React 17까지는 이렇게 썼습니다:

```jsx
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(<App />, document.getElementById("root"));
```

React 18부터는 **새로운 Root API**를 사용해야 합니다:

```jsx
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
```

여기서 핵심은:

* 더 이상 `render()`를 **정적 함수**처럼 바로 호출하지 않고
* 먼저 `createRoot()`로 **“루트 객체(ReactDOMRoot)”를 생성한 뒤**
* 그 루트 객체가 가진 `render()` 메서드로 렌더링을 수행한다는 점입니다.

즉,

> 🔹 `createRoot()` → **React 애플리케이션의 최상위 컨테이너를 초기화**하는 함수
> 🔹 반환값인 `root` → **해당 컨테이너를 관리하는 컨트롤러 객체**

---

## 2. 왜 굳이 `createRoot`로 바꿨나? (ReactDOM.render의 한계) ⚙️

React 팀이 굳이 API를 바꾸면서까지 도입한 이유는 **Concurrent Rendering(동시성 렌더링)**입니다.

### 2-1. React 17까지의 문제점

기존 `ReactDOM.render` 방식은:

* 한 번 렌더링을 시작하면 → **중간에 끊거나 양보하기 어려움**
* 브라우저 메인 스레드를 오래 점유할 수 있음 → **큰 트리 렌더링 시 UI가 뚝뚝 끊기는 느낌**
* “업데이트 우선순위”를 섬세하게 관리하기 어려움
  (예: 입력 중인 텍스트 vs 화면 밖 리스트 렌더링)

### 2-2. React 18의 목표

`createRoot`가 여는 새로운 세계:

* 🧵 **Concurrent Rendering 지원**
  → React가 렌더링 작업을 잘게 쪼개어 나누고,
  → 중간에 **“잠깐, User가 입력했네?”** 하면서 브라우저에게 제어를 넘길 수 있음
* 🎯 **우선순위 기반 업데이트**
  → 사용자의 입력, 애니메이션처럼 중요한 작업을 먼저 처리
* 🔁 **자동 배치(Automatic Batching) 강화**
  → 여러 상태 업데이트를 하나로 묶어 효율적으로 렌더

이 모든 기능의 “출발점”이 바로 **`createRoot`로 생성된 Concurrent Root**입니다.

---

## 3. `createRoot`의 사용 패턴 🔧

### 3-1. 기본 사용

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const container = document.getElementById("root");

// ✅ Concurrent Root 생성
const root = ReactDOM.createRoot(container);

// ✅ React 엘리먼트 렌더링
root.render(<App />);

// (필요할 때)
// root.unmount();
```

여기서 `root`는 다음 같은 역할을 합니다:

* 이 컨테이너에 **어떤 React 트리가 연결되어 있는지** 관리
* 이후 `root.render(newTree)`로 **부분/전체 갱신** 가능
* 필요하면 `root.unmount()`로 **React 트리 전체 제거**

---

## 4. createRoot가 반환하는 “root 객체”의 정체 🤖

`createRoot(container)`는 내부적으로 **“Root Fiber”**를 생성하고, 그 Root Fiber와 DOM 컨테이너를 연결한 뒤, 이 모든 것을 다루는 **Root Controller 객체**를 반환합니다.

대표적인 메서드:

### 4-1. `root.render(element)`

* 현재 Root에 연결된 React 트리를 **주어진 엘리먼트 구조로 업데이트**
* 초기에 한 번 호출해도 되고,
  상태 관리 전략에 따라 **여러 번 호출** 가능
* React는 이전 트리와 새 트리를 비교하여 **필요한 부분만 DOM 업데이트** (Reconciliation)

### 4-2. `root.unmount()`

* 해당 컨테이너에서 React 트리를 완전히 제거
* 이벤트 핸들러, 상태, effect 등 깨끗하게 정리
* SPA에서 특정 시점에 React 앱을 떼어내야 할 때 사용 가능

---

## 5. createRoot와 React 18 기능의 연결고리 🔗

`createRoot`의 진짜 가치는 **React 18의 새로운 기능과 긴밀히 연결**된다는 점입니다.

### 5-1. Concurrent Rendering 🧵

Concurrent Root는 렌더링을 **“동기 한 방에 쭉”** 처리하지 않고:

* 시간이 오래 걸리는 렌더링 작업을 여러 청크로 나눕니다.
* 브라우저에게 **“잠깐 나도 숨 좀 고를게”** 라고 양보할 수 있습니다.
* 사용자 입력, 애니메이션과 같은 **고우선순위 작업**을 먼저 처리하고
  나머지 렌더는 뒤로 미룹니다.

결과적으로:

* 🖱 입력 지연 감소
* 📜 긴 리스트 스크롤 시 부드러움 향상
* ⚡ 느린 렌더링 컴포넌트가 있어도 전체 앱이 덜 끊김

> 이 모든 스케줄링/우선순위 조정은 **Concurrent Root 없이는 불가능**합니다.
> 즉, `createRoot`가 그것을 여는 “문”입니다.

---

## 6. `ReactDOM.render` vs `createRoot` 비교 표 📊

| 항목          | ReactDOM.render (Legacy) | ReactDOM.createRoot (Concurrent) |
| ----------- | ------------------------ | -------------------------------- |
| API 위치      | `react-dom`              | `react-dom/client`               |
| 렌더링 모드      | 동기 렌더링 중심                | 동시성 렌더링 지원                       |
| Root 관리 객체  | 없음 (정적 함수 호출)            | `root` 객체 반환                     |
| 자동 배치       | 일부 상황에서만                 | 대부분의 비동기 업데이트에 적용                |
| React 18 이후 | 점진적 제거 대상                | 권장/기본 진입점                        |

---

## 7. Hydration과 `hydrateRoot` 🌊

SSR(서버사이드 렌더링)까지 강의하신다면, `createRoot`와 쌍으로 등장하는 **`hydrateRoot`**도 함께 설명하시는 게 좋습니다.

### 7-1. 기존 방식 (React 17)

```jsx
ReactDOM.hydrate(<App />, document.getElementById("root"));
```

### 7-2. React 18 방식

```jsx
import { hydrateRoot } from "react-dom/client";

hydrateRoot(
  document.getElementById("root"),
  <App />
);
```

* `hydrateRoot` 역시 **Concurrent Hydration**을 지원합니다.
* 서버에서 미리 렌더링된 HTML과 **클라이언트의 React 트리**를 매칭하고
  이벤트 바인딩, 상태 복구 등을 수행합니다.

---

## 8. StrictMode와 createRoot의 관계 🔍

React 18에서 `createRoot` + `StrictMode` 조합을 사용하면, 개발 환경에서 다음이 두 번씩 실행됩니다:

* `useEffect` Cleanup / Setup
* 일부 라이프사이클

이유:

* Concurrent 모드에서 발생 가능한 문제를 **미리 조기에 탐지**하기 위해
* “Side-Effect가 없는 순수한 렌더링 로직”을 강제하기 위해

실제 코드:

```jsx
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

> **Concurrent Rendering 대비를 위한 안전장치** 💡

---

## 9. Vite + React 템플릿에서의 createRoot 구조 예시 🧪


```jsx
// main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");
const root = ReactDOM.createRoot(container);

// 여기서부터 React 트리의 “시작점”
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

포인트:

* `main.jsx`는 **“React 애플리케이션의 부트스트랩 파일”**
* 이 파일에서 **그냥 JSX를 렌더링하는 것이 아니라,**
* `createRoot`를 통해 **Root Fiber + DOM 컨테이너 연결**을 초기화하는 단계가 필요

---

## 10. 내부적으로 일어나는  🧩

`createRoot(container)` 호출 시 개념적 단계:

1. `container`(예: `<div id="root">`)를 아규먼트로 받음
2. React 내부에서 **Root Fiber 구조체 생성**
3. 해당 컨테이너 DOM 노드와 Root Fiber를 **1:1로 매핑**
4. 스케줄러(우선순위 관리), 업데이트 큐 등을 초기화
5. 이를 조종할 수 있는 **Root Controller 객체**(`root`)를 반환

이후 `root.render(<App />)` 호출 시:

1. `<App />`를 React Element 트리로 변환
2. 이전에 렌더된 트리와 비교 (Reconciliation)
3. 스케줄러가 작업을 여러 청크로 나누고, 브라우저와 협력해서 수행
4. 최종적으로 필요한 최소 DOM 변경만 적용




