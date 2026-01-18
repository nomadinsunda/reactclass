

# 🚀 React StrictMode란?

**React.StrictMode**는 개발 단계에서만 작동하는 **“개발용 진단 및 안전 점검 도구(Development Diagnostics Tool)”**입니다.
운영(프로덕션) 환경에서는 **절대 동작하지 않으며 성능에도 영향을 주지 않습니다.**

> ✅ 요약
> StrictMode는 React 앱에서 **잠재적 문제를 조기에 발견하기 위한 안전장치**입니다.
> 특히 React 18부터는 **Concurrent Rendering 시대에 맞춰 더욱 강력한 검사 기능**이 강화되었습니다.

---

# 🧠 1. StrictMode의 핵심 목적

StrictMode는 다음 네 가지 문제를 미리 감지합니다.

### 1) ❗ **Unsafe한 라이프사이클 메서드 감지 (클래스 컴포넌트)**

* `componentWillMount`
* `componentWillReceiveProps`
* `componentWillUpdate`
  이러한 메서드들은 비동기 렌더링 환경(Concurrent Mode)에서 안전하지 않기 때문에
  StrictMode는 이를 사용하면 경고를 출력합니다.

---

### 2) 🔁 **부작용(Side Effect) 감지: Effect Clean-up 반복 실행**

React 18의 StrictMode에서 가장 많이 겪는 현상:

> **`useEffect`, `useLayoutEffect`가 개발 환경에서 “두 번 실행”되는 이유**

이는 버그가 아니라,
**Effect가 안전하게 작성되었는지 검사하기 위해 React가 일부 로직을 의도적으로 다시 실행하기 때문**입니다.

---

### 3) 🧵 **비동기 렌더링(Concurrent Rendering)에서 문제될 코드 사전 검출**

React 18은 Concurrent Rendering 기반입니다.
StrictMode는 다음 같은 패턴이 안전한지 검사합니다:

* 같은 컴포넌트가 여러 번 렌더되어도 잘 동작하는가?
* 이벤트 핸들러나 effect가 idempotent(멱등성)를 가지는가?
* setTimeout / 외부 API 호출이 두 번 실행됐을 때 문제가 없는가?

즉, **하나의 컴포넌트 로직이 ‘멱등성’을 가져야 하는지 테스트하는 환경**이라고 보시면 됩니다.

---

### 4) 🧹 **구식 API 감지**

다음과 같은 레거시 API도 StrictMode에서 경고합니다:

* `findDOMNode()`
* legacy Context API
* 문자열 ref (`ref="text"`)
* 오래된 ref 사용 패턴 등

---

# 💡 2. StrictMode가 실제로 하는 일

StrictMode는 **특정 기능을 강제로 한 번 더 실행**하여 부작용을 탐지합니다.

| 검사 항목        | 동작                                        |
| ------------ | ----------------------------------------- |
| 렌더링 검사       | 컴포넌트 렌더링을 **두 번 실행**                      |
| Effect 검사    | `useEffect` → cleanup → setup을 **두 번 실행** |
| State 초기화 검사 | 컴포넌트 초기화 로직을 **여러 번 실행**                  |
| Legacy 코드 검사 | 위험한 패턴 감지 후 콘솔 경고 출력                      |

---

# 🔎 3. 예시 코드

```jsx
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

const root = createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

StrictMode는 **앱 전체** 또는 **일부 컴포넌트만** 감싸도 됩니다.

```jsx
<React.StrictMode>
  <UserForm />
</React.StrictMode>
```

---

# 🧪 4. 왜 useEffect가 두 번 실행될까?

React 18 StrictMode는 다음 순서로 effect를 검사합니다:

1. 컴포넌트 렌더
2. effect 실행
3. **effect cleanup 실행**
4. **다시 effect 실행**
5. 실제 렌더 트리에 반영됨

이 과정을 통해 다음을 검사할 수 있습니다:

* cleanup이 정확히 구현되었는가?
* effect가 안전하게 반복 실행 가능한가?
* 외부 API 호출, 구독, 이벤트 리스너 등에 부작용이 없는가?

---

# 🕹 5. 실제 시나리오로 이해하기 (React 18의 Concurrent Mode 기준)

### 📍 예: 서버에서 데이터 fetch

```jsx
useEffect(() => {
  fetch('/api/user').then(...)
}, []);
```

만약 StrictMode가 없다면:

* 앱 부트 시 1번만 호출됨

StrictMode가 있다면:

* 개발 환경에서는 fetch가 2번 호출됨
  → 의도적으로 문제가 있는지를 테스트하기 위함
  → 프로덕션 빌드에서는 **딱 1번** 호출됩니다

---

# 🔥 6. StrictMode는 프로덕션에서는 어떻게 동작할까?

**프로덕션(빌드 결과)에서는 완전히 비활성화됩니다.**

* 두 번 렌더링 없음
* effect 두 번 실행 없음
* 성능 오버헤드 없음
* 단지 개발 중에만 작동하는 도구

---

# 📚 7. 왜 React 팀은 StrictMode를 강제할까?

React 18 이후 렌더링 모델은 **Concurrent Rendering** 입니다.
이 렌더링 모델의 특징:

* 어떤 컴포넌트는 여러 번 렌더될 수 있음
* 오래 걸리는 렌더링 작업은 중단될 수 있음
* 재시작될 수 있음 (interruptible rendering)

즉,

> **React는 더 이상 “한 번 렌더 → DOM 반영” 패턴이 아님**

따라서 StrictMode는 다음을 사전에 검증합니다:

* 컴포넌트가 **idempotent**한가
* effect 정리가 완벽하게 구현되어 있는가
* 외부 API 접근/구독(cleanup)이 안정적인가
* 자식 컴포넌트가 여러 번 마운트되어도 문제없는가

---

# 🧭 8. StrictMode가 감지하는 패턴 예시

### 🚫 잘못된 사례

```jsx
useEffect(() => {
  socket.connect(); // cleanup 없음
}, []);
```

StrictMode로 감싸면:

* connect → cleanup 없음 → connect 재실행 → 연결 중복 발생
  → 콘솔에 경고 발생
  → 개발자가 문제를 빨리 발견할 수 있음

---

# 🏁 9. 왜 StrictMode를 꼭 써야 하는가?

현대 React 앱에서 StrictMode는 **사실상 필수**입니다.

* React가 미래에 도입할 더 강력한 동시성 기능과 호환성 확보
* 버그를 일찍 잡아내고 안정성 확보
* 팀 프로젝트 시 예측 가능한 렌더링 패턴 유지
* 오래된 코드 사용 방지
* 강의/교육 시 학생에게 “React 최신 렌더링 모델”을 이해시키는 데 도움

---

# 🎯 10. 결론

> 🔹 StrictMode = 개발자에게 잠재적 문제를 조기에 알려주는 안전 모드
> 🔹 렌더링 & effect를 일부러 두 번 실행
> 🔹 Concurrent Rendering 기반에서 문제가 될 수 있는 패턴을 사전에 감지
> 🔹 프로덕션에서는 완전히 비활성화
> 🔹 React 18 이후 기본적으로 사용하는 것이 추천

