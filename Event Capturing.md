# 이벤트 캡처링(Event Capturing)

브라우저 이벤트 모델의 첫 단계, 그리고 많은 개발자가 잘 이해하지 못하는 영역!

---

## 1. 이벤트 캡처링이란?

이벤트 캡처링(Capturing Phase)이란,

> **이벤트가 실제 타겟 요소에 도달하기 전에**
> **DOM 트리의 최상단에서부터 타겟 요소까지 내려오면서 실행되는 단계**

를 말합니다.

 다시 말해,
**밖 → 안(Capture)** → 타겟 → **안 → 밖(Bubble)** 순서 중,
바로 **밖 → 안** 구간입니다.

---

## 2. 이벤트 흐름 전체 그림

브라우저 이벤트 흐름은 3단계입니다:

```
1) Capturing Phase (캡처 단계)
2) Target Phase (타겟 단계)
3) Bubbling Phase (버블 단계)
```

→ 흐름을 그림으로 보면:

```
window
  ↓ (capture)
document
  ↓ (capture)
html
  ↓ (capture)
body
  ↓ (capture)
div
  ↓ (capture)
button ← 타겟
  ↑ (bubble)
div
  ↑ (bubble)
body
  ↑ (bubble)
html
  ↑ (bubble)
document
  ↑ (bubble)
window
```

즉:

* **캡처 단계 = 상위 요소 → 하위 요소로 내려오며 실행**
* **버블 단계 = 하위 요소 → 상위 요소로 올라가며 실행**

따라서 캡처링은 이벤트의 “진입 과정”이라고 볼 수 있습니다.

---

## 3. 이벤트 캡처링을 사용하는 이유는?

일반적으로 개발자들은 **버블링** 이벤트만 사용하는 경우가 많습니다.

그런데 캡처링은 다음과 같은 상황에서 매우 중요합니다.

### O 1) 이벤트를 “타겟 도달 전에” 가로채고 싶을 때

예: 모달 바깥 클릭이 아닌 **전체 페이지 차단**,
혹은 특정 로직을 우선 실행해야 하는 경우

### O 2) 이벤트 위임(Delegation)에서 특별한 상황 제어

버블링으로 위임하는 구조에서
**하위 요소가 이벤트를 먼저 처리하기 전에 상위 요소에서 처리하고 싶을 때**

### O 3) 보안/추적/로깅 등 글로벌 이벤트 선처리

전역 클릭 로깅, 사용자 접근 제어 등

### O 4) 일부 UI 프레임워크 내부 구현

React, Vue, Angular 모두
**DOM 이벤트는 캡처링 + 버블링 조합으로 제어**합니다.

---

## 4. 캡처링을 활성화하는 문법

### 기본 addEventListener:

```js
element.addEventListener('click', handler, { capture: true });
```

혹은

```js
element.addEventListener('click', handler, true); // 옛날 방식
```

이렇게 등록하면 **캡처 단계**에서 handler가 실행됩니다.

---

## 5. 캡처링 vs 버블링 실행 순서 비교

아래 구조를 보세요:

```html
<div id="outer">
  <div id="inner">Click me</div>
</div>
```

이벤트 등록:

```js
outer.addEventListener('click', () => console.log('outer capture'), { capture: true });
outer.addEventListener('click', () => console.log('outer bubble'));

inner.addEventListener('click', () => console.log('inner capture'), { capture: true });
inner.addEventListener('click', () => console.log('inner bubble'));
```

`inner`를 클릭했을 때 실행 순서:

```
1. outer capture
2. inner capture
3. inner bubble
4. outer bubble
```

→ **조상 요소에서는** 캡처링이 항상 버블링보다 먼저 실행됩니다.

 단, **타겟 요소 자신은 예외**입니다.
`inner`는 타겟이므로 2번·3번은 캡처/버블 단계가 아니라 **타겟 단계(`eventPhase === 2`)** 에서 실행되며,
그 순서는 `capture` 옵션이 아니라 **등록한 순서**로 결정됩니다.

```js
// 등록 순서를 바꾸면 출력 순서도 바뀝니다
inner.addEventListener('click', () => console.log('inner bubble'));                    // 먼저 등록
inner.addEventListener('click', () => console.log('inner capture'), { capture: true });

// 결과: outer capture → inner bubble → inner capture → outer bubble
```

> 참고로 React의 `onClickCapture`/`onClick`은 이 예외가 없습니다.
> React는 캡처 목록과 버블 목록을 따로 만들어 실행하므로, **타겟에서도 Capture가 항상 먼저** 호출됩니다.

---

## 6. 캡처링에서 전파(stopPropagation) 하면 무슨 일이?

```js
outer.addEventListener('click', (e) => {
  console.log('outer capture');
  e.stopPropagation();
}, { capture: true });

inner.addEventListener('click', () => console.log('inner bubble'));
```

`inner`을 클릭 시:

```
outer capture
```

그리고 끝.

* `inner bubble`은 실행되지 않음
* **왜냐하면 캡처 단계에서 전파가 이미 차단됐기 때문**

즉,

> **캡처링 단계에서 전파를 막으면 타겟에 도달하지도 않는다.**

이 특성은 매우 강력한 제어 도구입니다.

---

## 7. 캡처링 단계에서 preventDefault() 사용 가능?

정답: **가능하지만 상황 따라 다르다**

* 캡처 단계에서도 **기본 동작을 막을 수 있다** (기본 동작은 전파가 모두 끝난 뒤에 실행되므로, 어느 단계에서 막든 유효)
* 그러나 `event.cancelable === true`인 이벤트에만 가능
* 또한 리스너가 `passive: true`이면 불가능 — 호출해도 **무시되고 콘솔 경고만** 뜹니다
* `window` / `document` / `document.body`에 `wheel`, `touchstart`, `touchmove`를 등록하면
  **옵션을 생략해도 `passive`가 기본 `true`** 입니다. 이 경우 `{ capture: true, passive: false }`처럼 명시해야 막힙니다

예:

```js
window.addEventListener(
  'click',
  (e) => {
    e.preventDefault(); // 링크 클릭 이동 막기 가능
  },
  { capture: true }
);
```

---

## 8. 언제 캡처링을 사용하는가? (실전 사례)

### 8-1. 모달 외부 클릭으로 닫기 방지

버블링에서 처리하면 내부 클릭도 외부로 전파될 수 있음 → 문제 발생

캡처링으로 처리하면:

```js
document.addEventListener(
  'click',
  (e) => {
    if (!modal.contains(e.target)) {
      closeModal();
    }
  },
  { capture: true }
);
```

→ 모달 바깥 클릭만 정확히 감지 가능
→ 내부 로직보다 **항상 먼저 실행됨**

 React와 함께 쓸 때 주의할 점

* `document`에 캡처로 건 리스너는 **React의 모든 핸들러(`onClickCapture` 포함)보다 먼저** 실행됩니다.
  (React는 루트 컨테이너에 위임하는데, `document`가 그 조상이기 때문)
  그래서 “버튼을 눌러 드롭다운을 여는 `onClick`”이 실행되기도 전에 이 리스너가 먼저 닫아버리는 버그가 흔합니다.
  → 토글 버튼 자체를 `if (toggleRef.current?.contains(e.target)) return;`으로 제외해 주세요.
* 자식 쪽에서 `stopPropagation()`을 해도 **캡처 리스너는 이미 실행된 뒤**라 막을 수 없습니다.
* 제거할 때는 `capture` 플래그를 맞춰야 합니다: `document.removeEventListener('click', fn, { capture: true })`
  (또는 `AbortController`의 `signal`로 일괄 해제)
* 클릭보다 `pointerdown` 캡처가 더 자연스러운 경우가 많습니다(드래그·터치 대응).

---

### 8-2. 글로벌 클릭 로깅

대규모 서비스에서 유저 행동 추적 시:

```js
document.addEventListener(
  'click',
  (e) => console.log('전역 로깅'),
  { capture: true }
);
```

* 어떤 요소를 클릭하든 모두 먼저 기록 가능
* UI 로직보다 먼저 실행되므로 부작용 없음

---

### 8-3. 이벤트 위임과의 조합

ex) 특정 자식 요소에서 이벤트를 막는 경우에도
부모가 캡처링으로 먼저 받아 처리 가능.

---

### 8-4. 중요 시스템 UI 보호

버튼 스팸 방지, 특정 영역 클릭 우선 처리 등

예:

```js
document.addEventListener(
  'click',
  (e) => {
    if (isSystemMenuOpen) e.stopPropagation();
  },
  { capture: true }
);
```

---

## 9. 캡처링 + React의 관계 (중요!)

React(17 이후, 18·19 포함)는 **`document`가 아니라 `createRoot`에 넘긴 루트 컨테이너**에
캡처용·버블용 리스너를 각각 하나씩 걸어두고, 그 안에서 합성 이벤트를 직접 전파시킵니다.

그래서 네이티브 리스너와 섞였을 때 실제 순서는 이렇게 됩니다.

```
1) window / document 의 캡처 리스너            (네이티브)
2) 루트 컨테이너의 캡처 리스너                     → React의 onClickCapture 들 (위 → 아래)
3) 루트 안쪽 DOM 노드들의 네이티브 리스너            (캡처 → 타겟 → 버블)
4) 루트 컨테이너의 버블 리스너                     → React의 onClick 들 (아래 → 위)
5) document / window 의 버블 리스너            (네이티브)
```

* 즉 “**합성 캡처 → 합성 버블 → 네이티브 버블**” 순이 아니라,
  루트 안쪽 노드에 직접 건 네이티브 리스너는 **합성 캡처와 합성 버블 사이**에 끼어듭니다.
* React 핸들러의 `e.stopPropagation()`은 **합성 전파만** 멈춥니다.
  `document`/`window`에 직접 건 리스너까지 막으려면 `e.nativeEvent.stopImmediatePropagation()`이 필요합니다.

즉, React의 `onClickCapture`가 바로 캡처링입니다.

```jsx
<div onClickCapture={() => console.log('capture')}>
  <button onClick={() => console.log('bubble')}>Click</button>
</div>
```

출력:

```
capture
bubble
```

→ 개발자가 보는 흐름(캡처 → 타겟 → 버블)은 DOM과 동일
→ 다만 그 흐름을 **실제로 만들어내는 주체는 브라우저가 아니라 React**(루트 컨테이너에서의 이벤트 위임)라는 점이 핵심입니다

---

## 10. 흔히 하는 실수

### X 버블링을 막으면 캡처링도 막힌다고 착각

반대입니다!

* **캡처링 단계 → 타겟 → 버블링**
* 버블링에서 stopPropagation하면 **캡처링은 이미 끝난 뒤**

### X return false로 캡처링/버블링 제어 가능하다고 착각

순수 DOM에서는 불가능.

### X 잘못된 이벤트 위임으로 인해 클릭 이벤트가 이상하게 작동

캡처링 단계에서 전파를 막아서 생기는 문제

---

## 11. 한눈에 보는 캡처링 요약

| 구분             | 설명                                                   |
| -------------- | ---------------------------------------------------- |
| 역할             | 이벤트가 타겟에 도달하기 전, 상위 → 하위로 내려오며 실행                    |
| 실행 시점          | 버블링보다 먼저                                             |
| 활성화            | `addEventListener(type, handler, { capture: true })` |
| 전파 중단          | `stopPropagation()`을 쓰면 타겟에도 도달 X                    |
| preventDefault | cancelable 이벤트에 한하여 가능                               |
| 실전 활용          | 전역 로깅, 모달/드롭다운, 시스템 우선 처리, 이벤트 위임                    |
| React와의 관계     | `onClickCapture`로 동일한 흐름 구현                          |

---

# 마무리

이벤트 캡처링은 UI 이벤트 처리에서
“가장 먼저 이벤트를 잡는 필터” 같은 역할을 합니다.

대부분의 초급 개발자는 버블링만 사용하지만,
**캡처링을 이해하면 고급 UI 제어, 복잡한 상호작용, 프레임워크 동작까지 정확히 파악**할 수 있습니다.

