# ⭐ ReactDOMRoot.render란?

`ReactDOMRoot.render()`는
**ReactDOM.createRoot()가 리턴한 Root 객체가 제공하는 렌더링 메서드**로서,

> **React 컴포넌트 트리를 실제 DOM에 렌더링하는 엔트리 포인트(entry point)**
> **(Concurrent Rendering 엔진을 통해 구현됨)**

을 의미합니다.

즉,

```js
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
```

여기서 `render()`가 실제로:

* Virtual DOM → Fiber 트리 생성
* Fiber 작업 스케줄링 (우선순위 기반 Concurrent Engine)
* DOM 업데이트(commit phase)
* 이벤트 시스템 바인딩
* hydration 옵션 처리(SSR + Hydration일 경우)

등을 수행하는 핵심 API입니다.

---

# ⭐ ReactDOMRoot.render의 핵심 역할 6가지

## 1️⃣ **React 컴포넌트 트리를 Root에 연결**

`render(element)`를 호출하면 `element`(JSX)는 Fiber Node로 변환되어
Root Fiber 트리에 연결됩니다.

## 2️⃣ **Concurrent Rendering(동시성 렌더링) 활성화**

createRoot로 생성된 root는 모든 렌더링을 React Fiber Scheduler가 관리합니다.

즉,

* 우선순위 기반 분할 작업(Time-Slicing)
* Interrupt 가능
* 백그라운드에서 부드러운 렌더링

등을 모두 render()에서 시작합니다.

## 3️⃣ **Initial mount 또는 update 렌더링 수행**

리액트는 render() 호출 시:

* **최초 호출**: mount
* **두 번째 이후 호출**: update(diff 렌더링)

을 자동으로 구분합니다.

## 4️⃣ **DOM을 실제로 업데이트(commit phase)**

Fiber Reconciliation 이후, DOM 조작은 commit phase에서 이루어지며
ReactDOMRoot.render가 이를 최종적으로 실행합니다.

## 5️⃣ **이벤트 시스템 등록**

최초 render 시 React의 “Event Delegation System”이 Root DOM에 바인딩됩니다.

예: `onClick`, `onChange`, `onSubmit` 등

## 6️⃣ **StrictMode 동작 반영**

Root 안에 `<React.StrictMode>`가 있다면
의도적인 double render(개발 모드)도 render() 단계에서 처리됩니다.

---

# ⭐ ReactDOMRoot.render는 리렌더링 API가 아니다

많은 분들이 헷갈리지만,

### ✔ render()는 "리렌더링 함수"가 **아닙니다**

→ **리렌더링은 state 변경으로 React가 자동 수행**
→ render()는 “Root 상태 업데이트 시작 신호”를 보내는 함수

정확한 정의는:

> **Root Fiber 트리에 새로운 React Element 트리를 연결해 렌더링 사이클을 트리거하는 함수**

입니다.

---

# ⭐ ReactDOMRoot.render의 내부 흐름

아주 간단하게 Fiber 스케줄러 흐름을 단계로 요약하면:

```
root.render(<App />)
   ↓
[Reconciliation - Render phase]
   - Fiber 트리 생성/업데이트
   - 우선순위 계산
   - concurrent scheduling
   - 작업을 pause/resume 가능
   ↓
[Commit phase]
   - DOM 변경(diff 반영)
   - 효과 부착 (useEffect)
   - 이벤트 등록
```

`render()`는 사실 이 전체 사이클을 시작하는 **트리거 역할**입니다.

---

# ⭐ ReactDOMRoot.render vs 이전 ReactDOM.render()

| 항목              | ReactDOM.render() (전통) | ReactDOMRoot.render() (React 18) |
| --------------- | ---------------------- | -------------------------------- |
| API             | 정적 API                 | Root 기반 API                      |
| 동시성(Concurrent) | ❌ 지원 안 됨               | ✅ 기본 활성화                         |
| 렌더링 엔진          | Legacy Stack Renderer  | Fiber Concurrent Renderer        |
| SSR Hydration   | 제한적                    | 개선된 hydrateRoot와 통합              |
| 리렌더링 방식         | 모든 호출이 전체 렌더링          | Root 트리에 기반하여 더 정교함              |

---

# ⭐ 요약 정리

> **ReactDOMRoot.render()는 React 앱을 DOM에 연결해 실제 렌더링을 시작하는 메서드이며, React Fiber Concurrent Engine의 전체 렌더링 사이클을 트리거하는 핵심 함수입니다.**


