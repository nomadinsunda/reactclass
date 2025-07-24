

## ✅ Virtual DOM은 무엇인가?

* **Virtual DOM은 "개념"이며**, 브라우저가 제공하는 기능이나 API가 아닙니다.
* 즉, `document.createElement()` 같은 DOM API처럼 **브라우저에 내장된 표준 기능이 아니라**,
  React(또는 다른 프레임워크)가 **자체적으로 구현한 내부 기술**입니다.
* React, Vue, Inferno, Svelte (일부), Preact 등에서 각각 **자체 방식으로 구현**합니다.

---

## 🔍 표준 DOM vs Virtual DOM

| 구분    | 표준 DOM                                        | Virtual DOM                                |
| ----- | --------------------------------------------- | ------------------------------------------ |
| 정의    | 브라우저가 제공하는 **공식 API와 객체 모델**                  | 라이브러리 내부에서 메모리에 구성된 **자바스크립트 객체 기반 가상 트리** |
| 표준 여부 | ✅ W3C 표준                                      | ❌ 비표준 (라이브러리마다 다름)                         |
| 사용 방법 | `document`, `createElement`, `appendChild`, 등 | `React.createElement`, JSX → React 내부 구조   |
| 위치    | 브라우저 메모리 상의 DOM 트리                            | 라이브러리 내부의 JS 객체                            |
| 목적    | 사용자에게 보이는 **실제 UI를 렌더링**                      | UI 변경을 **최소한으로 계산**하고 성능을 최적화              |

---

## 📦 React의 Virtual DOM 예시

```jsx
const element = <h1>Hello</h1>;
```

위 코드는 다음과 같이 내부 Virtual DOM 객체로 변환됩니다:

```js
const vdom = {
  type: 'h1',
  props: {
    children: 'Hello'
  }
};
```

그리고 React는 이 Virtual DOM을 기반으로 **이전 VDOM과 비교(diff)** 한 후,
**실제 DOM에 필요한 변경만 적용(patch)** 합니다.

---

## 💡 그럼 왜 존재하나?

브라우저의 DOM은 매우 무겁고 조작 비용이 큽니다.
→ 이를 **메모리상의 가벼운 객체로 대체해서 변경 전후를 비교하고**,
→ **오직 변경된 부분만 브라우저의 실제 DOM에 반영**하면 **성능이 비약적으로 향상**됩니다.

---

## 📌 결론

> Virtual DOM은 웹 표준이 아니라, **React 같은 라이브러리에서 자체적으로 만든 개념적인 최적화 기법**입니다.
> 브라우저가 지원하거나 명세한 것이 아니며, 다른 프레임워크는 **자체 구현 방식이 다릅니다.**


---

# 🌳 Virtual DOM 구현 원리와 주요 프레임워크 비교

## 🧠 1. Virtual DOM이란?

> \*\*Virtual DOM(V-DOM)\*\*은 **실제 브라우저 DOM을 추상화한 메모리 상의 객체 트리 구조**입니다.
> 변경이 발생하면 이 객체 트리를 통해 "차이점(diff)"을 계산하고, 최소한의 DOM 조작만 실제 브라우저에 적용합니다.

---

## 🧬 2. Virtual DOM의 구현 원리

### 📌 핵심 3단계

1. **렌더(Render)**
   컴포넌트 함수나 JSX를 기반으로 VDOM 트리 생성
   예:

   ```jsx
   const element = <h1>Hello</h1>;
   ```

   내부적으로는:

   ```js
   {
     type: 'h1',
     props: { children: 'Hello' }
   }
   ```

2. **디프(Diff)**
   변경 전의 Virtual DOM(old)과 변경 후의 Virtual DOM(new)을 비교
   → 어떤 노드가 변경되었는지 계산

   예:

   ```js
   old: <h1>Hello</h1>  
   new: <h1>Hi</h1>
   ```

   → 텍스트 노드 변경만 감지됨

3. **패치(Patch)**
   브라우저 실제 DOM에서 해당 변경만 적용 (innerText 등)

---

## ⚙️ 3. React의 Virtual DOM

* React는 **Fiber 구조**를 이용한 **비동기적 VDOM 비교 엔진**을 사용
* 주요 특징:

  * 트리를 DFS 순회하며 비교
  * 동일한 타입이면 props만 비교
  * 다른 타입이면 하위 노드를 모두 제거하고 다시 생성

```jsx
function App() {
  return <div><h1>Hello</h1></div>;
}
```

→ 내부적으로 React는 다음과 같은 VDOM 트리 생성:

```js
{
  type: 'div',
  props: {
    children: {
      type: 'h1',
      props: { children: 'Hello' }
    }
  }
}
```

* React는 이 구조를 기반으로 **reconciliation 알고리즘**으로 변경을 계산하고, 실제 DOM에 **batch update**를 적용

---

## 🧩 4. 다른 프레임워크들의 DOM 처리 방식

### ✅ 1. Vue.js (v2, v3)

* Vue 2: React와 유사한 VDOM 구현
* Vue 3: **Proxy 기반 반응형 시스템(Reactivity)** + **VDOM**
* React와 유사하게 template → render function → VDOM → Patch

> `v-if`, `v-for` 등을 통해 노드 변경 시 diffing 처리

---

### ✅ 2. Preact

* React의 **경량화 버전 (3kb)**
* 동일한 API (JSX 사용 가능), 내부 구현은 더 단순
* VDOM 사용 방식은 React와 거의 동일하지만, 일부 최적화 제거

> `h()`, `render()`, `diff()`, `commitRoot()` 함수로 구성된 간단한 아키텍처

---

### ✅ 3. Inferno.js

* React 스타일이지만, **VDOM diffing이 매우 빠름**
* JIT 컴파일 + 최적화된 patch 알고리즘

> 10만 개 이상의 노드를 diff해도 1ms 이내에 계산

---

### ✅ 4. Svelte

* ❌ Virtual DOM 사용 안 함
* **Compile-time에 DOM 조작 코드를 생성**
* 예: `<h1>{message}</h1>` → `document.querySelector('h1').textContent = message;`

> 즉, **컴파일러가 DOM 변경 코드를 만들어내므로 diffing 자체가 없음**

장점: 성능 최상 / 단점: 동적 구조는 제약 있음

---

### ✅ 5. Solid.js

* ❌ Virtual DOM 사용 안 함
* **Fine-grained reactivity**: signal이 바뀐 곳만 직접 업데이트

예:

```js
const [count, setCount] = createSignal(0);
```

> `count()`가 쓰인 DOM만 업데이트됨 (정확한 대상만 정밀 추적)

→ Reactivity Graph로 추적

---

## ⚖️ 5. 비교 요약

| 프레임워크      | VDOM 사용 | 처리 방식                 | 특징                    |
| ---------- | ------- | --------------------- | --------------------- |
| React      | ✅ 예     | VDOM diff → patch     | Fiber, JSX            |
| Vue (v2/3) | ✅ 예     | VDOM + reactivity     | Proxy (v3)            |
| Preact     | ✅ 예     | VDOM                  | React 호환, 경량          |
| Inferno    | ✅ 예     | VDOM 고속 diff          | JIT 최적화               |
| Svelte     | ❌ 아니요   | 컴파일 시 DOM 생성          | Virtual DOM 없음        |
| Solid.js   | ❌ 아니요   | signal 추적 → 직접 DOM 변경 | Fine-grained reactive |

---

## ✅ 결론

* Virtual DOM은 표준이 아닌 **프레임워크 내부 최적화 기법**
* 모든 프레임워크가 VDOM을 사용하는 것은 아님
* 최신 경향은 **VDOM 없이 더 정밀하고 빠르게 업데이트하는 방식(Svelte, Solid)** 으로 이동 중

