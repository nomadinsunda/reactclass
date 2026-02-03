**“포인터 이벤트”는 마우스/터치/펜을 하나로 통합한 이벤트 모델**입니다.
이걸 이해하면 **왜 DnD 라이브러리가 포인터 이벤트를 쓰는지**가 바로 보입니다.

---

## 1️⃣ 한 줄 정의부터 📌

> **포인터 이벤트(pointer events)** 란
> **마우스, 터치, 스타일러스(펜) 입력을 “하나의 이벤트 체계”로 다루는 웹 표준 이벤트**입니다.

---

## 2️⃣ 왜 굳이 “포인터”라는 개념이 생겼을까? 🤔

과거에는 입력 방식마다 이벤트가 따로 있었습니다.

### ❌ 예전 방식 (분리됨)

| 입력 장치 | 이벤트                                   |
| ----- | ------------------------------------- |
| 마우스   | `mousedown`, `mousemove`, `mouseup`   |
| 터치    | `touchstart`, `touchmove`, `touchend` |
| 펜     | vendor-specific                       |

👉 문제점:

* 같은 로직을 **3번 구현**
* 모바일/태블릿 대응 지옥
* DnD 같은 기능은 유지보수 악몽

---

## 3️⃣ 포인터 이벤트의 핵심 아이디어 💡

> **“어차피 사용자는 ‘어딘가를 누르고 움직인다’ → 이걸 하나로 묶자”**

그래서 나온 게 **Pointer Events API**입니다.

---

## 4️⃣ 포인터 이벤트의 기본 이벤트들 🎯

| 이벤트                             | 의미                       |
| ------------------------------- | ------------------------ |
| `pointerdown`                   | 포인터가 눌림 (마우스 클릭 / 터치 시작) |
| `pointermove`                   | 포인터가 이동                  |
| `pointerup`                     | 포인터가 떼어짐                 |
| `pointercancel`                 | 시스템이 입력을 취소              |
| `pointerenter` / `pointerleave` | 영역 진입/이탈                 |

👉 **마우스/터치/펜 모두 동일한 이벤트 이름**을 씁니다.

---

## 5️⃣ 포인터 이벤트가 “통합”이라는 증거 🧠

포인터 이벤트 객체에는 이런 정보가 들어 있습니다.

```js
event.pointerType // 'mouse' | 'touch' | 'pen'
event.clientX
event.clientY
event.isPrimary
```

즉 코드에서는:

```js
if (event.pointerType === 'touch') {
  // 터치
}
```

같은 분기조차 **선택 사항**입니다.

---

## 6️⃣ DnD에서 포인터 이벤트가 중요한 이유 🎯

지금 사용 중인 `@hello-pangea/dnd` 같은 라이브러리는:

* 데스크톱 (마우스)
* 모바일 (터치)
* 태블릿 (펜)

을 **모두 동일한 드래그 로직으로 처리**해야 합니다.

그래서 내부적으로는:

```txt
pointerdown → drag start
pointermove → dragging
pointerup → drop
```

이라는 **하나의 흐름**만 유지합니다.

👉 만약 마우스/터치를 나눴다면:

* 코드 2배
* 버그 2배
* 테스트 2배

---

## 7️⃣ pointer events vs CSS `pointer-events` (중요한 구분 ⚠️)

이름 때문에 많이 헷갈립니다.

### ❌ 이건 CSS 속성

```css
pointer-events: none;
```

* “마우스/터치 이벤트를 무시할지 여부”
* 클릭 막기용

### ✅ 이게 우리가 말하는 것

```js
pointerdown / pointermove / pointerup
```

* **입력 이벤트 시스템**
* 완전히 다른 개념입니다

---

## 8️⃣ React에서 포인터 이벤트는 이렇게 씁니다 🧩

```jsx
<div
  onPointerDown={handleDown}
  onPointerMove={handleMove}
  onPointerUp={handleUp}
/>
```

React는 내부적으로 브라우저 pointer events를 감싸서 제공합니다.

---

## 9️⃣ 왜 최신 라이브러리들은 포인터 이벤트를 “기본”으로 쓰나?

| 이유         | 설명              |
| ---------- | --------------- |
| 📱 멀티 디바이스 | 모바일/데스크톱/태블릿 통합 |
| ♿ 접근성      | 키보드/보조기기 연동 용이  |
| 🧠 단순화     | 로직 1세트          |
| 🚀 성능      | 이벤트 중복 처리 감소    |

👉 그래서 **DnD / 드로잉 / 캔버스 / 제스처 라이브러리**는 거의 다 포인터 이벤트 기반입니다.

---

## 🔟 한 문장 요약 🧩

> **포인터 이벤트는
> 마우스·터치·펜 입력을 하나의 이벤트 흐름(`pointerdown → move → up`)으로 통합한 웹 표준이며,
> 현대 DnD 라이브러리의 핵심 기반입니다.**

---



# 1️⃣ 포인터 캡처(pointer capture)란?

## 한 줄 정의 📌

> **포인터 캡처(pointer capture)**란
> **한 요소가 “포인터가 떼어질 때까지” 모든 포인터 이벤트를 독점해서 받는 기능**입니다.

---

## 2️⃣ 포인터 캡처 없으면 무슨 일이 생길까? 😱

### 상황 가정

1. 카드에서 `pointerdown`
2. 드래그 시작
3. 마우스를 빠르게 움직임
4. 포인터가 카드 DOM 영역을 벗어남

### ❌ 포인터 캡처가 없으면

* 이후 `pointermove` 이벤트는

  * **포인터가 위치한 다른 요소**에게 전달됨
* 카드 컴포넌트는:

  * 이벤트를 더 이상 못 받음
  * 드래그 중단 / 튕김 / 끊김 발생

👉 **“요소 밖으로 나가면 드래그가 끊기는 현상”**

---

## 3️⃣ 포인터 캡처가 있으면 어떻게 달라질까? 🔒

```js
element.setPointerCapture(event.pointerId)
```

이 순간부터:

* 포인터가 어디로 이동하든
* 화면 밖으로 나가든
* 다른 요소 위에 있든

👉 **모든 `pointermove / pointerup` 이벤트가
캡처한 요소로 강제 전달**

즉:

```txt
pointerdown (카드)
↓ capture
pointermove (문서 어디든)
pointerup (어디서든)
```

👉 **드래그가 절대 끊기지 않음**

---

## 4️⃣ DnD에서 포인터 캡처가 “필수”인 이유 🎯

DnD는 본질적으로 이 흐름입니다:

```txt
pointerdown → pointermove(반복) → pointerup
```

그리고 드래그 중에는:

* 마우스는 수백 px 이동
* 터치는 화면 전체를 가로질러 이동
* 스크롤/오버플로우 영역도 넘나듦

👉 포인터 캡처 없으면 **정상적인 드래그는 불가능**합니다.

그래서 `@hello-pangea/dnd`, `dnd-kit`, `react-dnd` 같은 라이브러리는
**모두 내부적으로 pointer capture를 사용**합니다.

---

# 2️⃣ pointerdown vs mousedown 차이점

이제 두 번째 질문으로 넘어가죠.

---

## 1️⃣ 이벤트 계보부터 다릅니다 🧬

| 구분      | pointerdown    | mousedown    |
| ------- | -------------- | ------------ |
| 이벤트 시스템 | Pointer Events | Mouse Events |
| 입력 통합   | 마우스·터치·펜       | 마우스 전용       |
| 모바일 지원  | ✅              | ❌ (터치 안 됨)   |
| 압력/펜 정보 | 있음             | 없음           |
| 포인터 캡처  | 있음             | ❌            |

👉 **완전히 다른 이벤트 시스템**입니다.

---

## 2️⃣ 모바일에서의 결정적 차이 📱

### `mousedown`

* 터치 디바이스에서는:

  * 아예 발생하지 않거나
  * `touchstart`로 대체됨

### `pointerdown`

* 터치 시에도 **그대로 발생**
* `pointerType === 'touch'`

👉 **하나의 코드로 모바일/데스크톱 통합**

---

## 3️⃣ 드래그 관점에서의 차이 (핵심) 🔥

### ❌ mousedown 기반 드래그

```txt
mousedown → mousemove → mouseup
```

문제:

* 터치 불가
* 펜 불가
* pointer capture 불가
* 스크롤/iframe/오버레이에서 끊김

---

### ✅ pointerdown 기반 드래그

```txt
pointerdown → setPointerCapture
→ pointermove → pointerup
```

장점:

* 모든 입력 통합
* 안정적인 드래그
* 화면 전체 이동 가능
* 모바일 대응

👉 **DnD 라이브러리가 pointerdown을 쓰는 이유**

---

## 4️⃣ 실제 차이가 드러나는 예시 💥

### 예시 1: 요소 밖으로 나갔을 때

| 상황        | mousedown | pointerdown |
| --------- | --------- | ----------- |
| 카드 밖으로 이동 | 드래그 끊김 가능 | 유지          |
| 스크롤 영역 통과 | 불안정       | 안정          |
| iframe 근처 | 자주 끊김     | 안정          |

---

### 예시 2: 입력 장치

| 입력  | mousedown | pointerdown |
| --- | --------- | ----------- |
| 마우스 | ✅         | ✅           |
| 터치  | ❌         | ✅           |
| 펜   | ❌         | ✅           |

---

## 5️⃣ React + DnD에서의 실제 흐름 🧩

`dragHandleProps` 내부에서는 실제로:

```txt
onPointerDown
  → setPointerCapture
  → drag start
```

이런 흐름이 숨어 있습니다.

👉 우리가 직접 안 써도 되는 이유는
👉 **라이브러리가 다 해주기 때문**입니다.

---

## 6️⃣ 한 문장씩 요약 🧩

### ✔ 포인터 캡처

> **드래그가 시작된 요소가 포인터가 떼어질 때까지 모든 포인터 이벤트를 독점하는 메커니즘**

### ✔ pointerdown vs mousedown

> **pointerdown은 모든 입력을 통합하고 캡처 가능한 현대 이벤트이며,
> mousedown은 마우스 전용의 레거시 이벤트**




