# 🚦 `React.StrictMode`란?

## ✅ 정의

```jsx
<StrictMode>...</StrictMode>
```

는 React가 제공하는 **개발 모드 전용 도우미 컴포넌트**입니다.

> `StrictMode`는 앱의 **잠재적 문제를 미리 감지하고 경고**를 출력해 주는
> **개발 환경에서만 작동하는 안전 감시 모드**입니다.

---

## ✅ 어떤 일을 하나요?

`<StrictMode>`는 **다음과 같은 코딩 실수를 미리 감지**합니다:

| 기능                               | 설명                                                             |
| -------------------------------- | -------------------------------------------------------------- |
| 🚨 **사용 중단된(Deprecated) API 감지** | 예: `componentWillMount()` 같은 레거시 API                           |
| 🔁 **의도치 않은 부작용 감지**             | `useEffect`, `useState` 같은 Hook 내 **부작용(side effect)** 발생 시 경고 |
| 👀 **중복 렌더링 검사**                 | 일부 렌더링 로직이 잘못된 경우 `render()`를 **두 번 호출**해 이상 여부 확인             |
| 👶 **향후 React 기능과 호환 테스트**       | Concurrent Mode, 서버 컴포넌트 등을 대비한 테스트                            |

---

## 🔁 의도적인 **이중 호출 예제**

예를 들어 다음과 같은 컴포넌트가 있다고 가정합시다:

```jsx
useEffect(() => {
  console.log('Mounted');
}, []);
```

이때 `StrictMode` 안에 감싸면:

```jsx
<StrictMode>
  <App />
</StrictMode>
```

➡ 결과: 콘솔에 `"Mounted"`가 **두 번 출력됩니다.**

### 이유?

* **React는 개발 환경에서만** `useEffect`, `constructor`, `render`, `state` 초기화 등을 **두 번 실행**하여 \*\*"안정성 테스트"\*\*를 합니다.
* 하지만 **실제 렌더는 한 번만** 발생하므로, **성능 문제는 없습니다.**

---

## 🔍 실제 코드: `main.jsx` 또는 `index.jsx`

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

여기서 `StrictMode`는 `App`과 그 하위 컴포넌트 전체에 적용되며
**개발 환경에서만 활성화**되고,
**프로덕션 빌드 시에는 완전히 제거**됩니다.

---

## ✅ StrictMode는 다음을 **하지 않습니다**

| 항목              | 설명                                 |
| --------------- | ---------------------------------- |
| ❌ UI에 영향을 주지 않음 | `<StrictMode>`는 DOM에 아무것도 렌더링하지 않음 |
| ❌ 성능 저하 없음      | 오직 개발 중만 작동                        |
| ❌ 앱을 느리게 만들지 않음 | `React 18+`의 자동 이중 호출은 철저히 테스트 목적  |

---

## 📘 React 공식 설명 요약

> “StrictMode does not render any visible UI.
> It activates additional checks and warnings for its descendants.”

---

## ✅ 언제 사용하는가?

| 상황                       | 이유                           |
| ------------------------ | ---------------------------- |
| 새 프로젝트 시작 시              | 기본적으로 `<StrictMode>`로 감싸야 안전 |
| 팀 개발 시                   | 레거시 API 감지, Hook misuse 감지   |
| 마이그레이션 작업                | 문제 코드 사전 파악 가능               |
| `Concurrent Features` 대비 | 미래 기능 테스트 및 호환성 확보           |

---

## 📌 결론 요약

| 항목       | 설명                                         |
| -------- | ------------------------------------------ |
| 정체       | 개발 전용 React 감시 컴포넌트                        |
| 역할       | 레거시 API 감지, 부작용 검사, 이중 실행 테스트 등            |
| 프로덕션 영향  | ❌ 없음 (자동 제거됨)                              |
| 사용 권장 여부 | ✅ 항상 루트 컴포넌트를 `<StrictMode>`로 감싸는 것이 모범 사례 |
| 위치       | 보통 `main.jsx`, `index.jsx`의 `App` 주변에 사용   |

