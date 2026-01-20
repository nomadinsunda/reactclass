# 🎯 이벤트 캡처링(Event Capturing)

브라우저 이벤트 모델의 첫 단계, 그리고 많은 개발자가 잘 이해하지 못하는 영역!

---

## 1. 이벤트 캡처링이란? 🕵️‍♂️

이벤트 캡처링(Capturing Phase)이란,

> **이벤트가 실제 타겟 요소에 도달하기 전에**
> **DOM 트리의 최상단에서부터 타겟 요소까지 내려오면서 실행되는 단계**

를 말합니다.

📌 다시 말해,
**밖 → 안(Capture)** → 타겟 → **안 → 밖(Bubble)** 순서 중,
바로 **밖 → 안** 구간입니다.

---

## 2. 이벤트 흐름 전체 그림 🧭

브라우저 이벤트 흐름은 3단계입니다:

```
1) Capturing Phase (캡처 단계)
2) Target Phase (타겟 단계)
3) Bubbling Phase (버블 단계)
```

➡ 흐름을 그림으로 보면:

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

## 3. 이벤트 캡처링을 사용하는 이유는? 🤔

일반적으로 개발자들은 **버블링** 이벤트만 사용하는 경우가 많습니다.

그런데 캡처링은 다음과 같은 상황에서 매우 중요합니다.

### ✅ 1) 이벤트를 “타겟 도달 전에” 가로채고 싶을 때

예: 모달 바깥 클릭이 아닌 **전체 페이지 차단**,
혹은 특정 로직을 우선 실행해야 하는 경우

### ✅ 2) 이벤트 위임(Delegation)에서 특별한 상황 제어

버블링으로 위임하는 구조에서
**하위 요소가 이벤트를 먼저 처리하기 전에 상위 요소에서 처리하고 싶을 때**

### ✅ 3) 보안/추적/로깅 등 글로벌 이벤트 선처리

전역 클릭 로깅, 사용자 접근 제어 등

### ✅ 4) 일부 UI 프레임워크 내부 구현

React, Vue, Angular 모두
**DOM 이벤트는 캡처링 + 버블링 조합으로 제어**합니다.

---

## 4. 캡처링을 활성화하는 문법 📌

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

## 5. 캡처링 vs 버블링 실행 순서 비교 🔥

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

➡ 캡처링은 **항상 버블링보다 먼저 실행**됩니다.

---

## 6. 캡처링에서 전파(stopPropagation) 하면 무슨 일이? 🧱

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

## 7. 캡처링 단계에서 preventDefault() 사용 가능? 🧐

정답: **가능하지만 상황 따라 다르다**

* 캡처 단계에서도 **기본 동작을 막을 수 있다**
* 그러나 `event.cancelable === true`인 이벤트에만 가능
* 또한 리스너가 `passive: true`이면 불가능

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

## 8. 언제 캡처링을 사용하는가? (실전 사례) 🧪

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

➡ 모달 바깥 클릭만 정확히 감지 가능
➡ 내부 로직보다 **항상 먼저 실행됨**

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

## 9. 캡처링 + React의 관계 🔥 (중요!)

React는 DOM 이벤트를 다음 순서로 처리합니다.

1. 캡처링 이벤트(Synthetic Event Capture)
2. 타겟 이벤트
3. 버블링 이벤트(Synthetic Event Bubble)
4. native DOM 이벤트 버블링

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

➡ DOM 이벤트 흐름과 완전히 동일
➡ React 내부 구현을 이해하려면 꼭 알아야 하는 부분

---

## 10. 흔히 하는 실수 ⚠️

### ❌ 버블링을 막으면 캡처링도 막힌다고 착각

반대입니다!

* **캡처링 단계 → 타겟 → 버블링**
* 버블링에서 stopPropagation하면 **캡처링은 이미 끝난 뒤**

### ❌ return false로 캡처링/버블링 제어 가능하다고 착각

순수 DOM에서는 불가능.

### ❌ 잘못된 이벤트 위임으로 인해 클릭 이벤트가 이상하게 작동

캡처링 단계에서 전파를 막아서 생기는 문제

---

## 11. 한눈에 보는 캡처링 요약 📝

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

# 📌 마무리

이벤트 캡처링은 UI 이벤트 처리에서
“가장 먼저 이벤트를 잡는 필터” 같은 역할을 합니다.

대부분의 초급 개발자는 버블링만 사용하지만,
**캡처링을 이해하면 고급 UI 제어, 복잡한 상호작용, 프레임워크 동작까지 정확히 파악**할 수 있습니다.

