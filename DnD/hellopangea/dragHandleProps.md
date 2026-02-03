## 1️⃣ 왜 `dragHandleProps`가 따로 있을까? 🤔 (존재 이유)

### ❓ 질문

> draggableProps만 있으면 되는 거 아닌가?

### ❌ 그렇게 설계하면 생기는 문제

```jsx
<div {...provided.draggableProps}>
  <button>삭제</button>
  <input />
</div>
```

* 버튼 클릭하려다 드래그 시작
* input 포커스하려다 드래그 시작
* 모바일에서 터치 UX 최악

👉 **“카드 전체가 드래그 핸들”인 UX는 현실적으로 문제 많음**

그래서 DnD 라이브러리는 **“잡는 영역”을 분리**합니다.

---

## 2️⃣ `dragHandleProps`의 정체 🎯

```js
provided.dragHandleProps
```

* `@hello-pangea/dnd`가 **직접 생성**
* “드래그 시작을 감지할 수 있는 이벤트/속성 묶음”
* 이 props가 붙은 DOM에서만 **drag start 가능**

👉 즉,

> **드래그의 “시작점(Start Zone)”을 지정하는 역할**

---

## 3️⃣ `dragHandleProps` 안에는 뭐가 들어 있나? 🔍

구체적인 키는 내부 구현이지만, 성격은 명확합니다.

### 포함되는 것들

* pointer / mouse / touch 이벤트 핸들러
* keyboard drag 시작 이벤트
* 접근성(ARIA) 속성
* 포커스/탭 이동 관련 속성

대략 이런 느낌입니다:

```js
dragHandleProps = {
  tabIndex: 0,
  role: 'button',
  'aria-describedby': '...',

  onMouseDown: fn,
  onTouchStart: fn,
  onKeyDown: fn,
}
```

⚠️ 주의
**이걸 직접 읽거나 수정하면 안 됩니다.**
그냥 **DOM에 spread**만 하세요.

---

## 4️⃣ 어떻게 써야 하나? (패턴별 정리) 🧩

### ✅ 1. 전체 카드가 핸들인 경우 (간단)

```jsx
<div
  ref={provided.innerRef}
  {...provided.draggableProps}
  {...provided.dragHandleProps}
>
  {children}
</div>
```

* 카드 어디를 잡아도 드래그 가능
* 작은 데모/프로토타입에 적합

---

### ✅ 2. 핸들 분리 (실무에서 가장 흔함 ⭐)

```jsx
<div ref={provided.innerRef} {...provided.draggableProps}>
  <div {...provided.dragHandleProps} className="handle">
    ☰
  </div>
  <div className="content">
    카드 내용
  </div>
</div>
```

* 드래그는 ☰ 아이콘에서만 시작
* 버튼/입력 요소와 충돌 없음
* UX 최고

---

### ❌ 3. dragHandleProps를 안 붙이면?

```jsx
<div {...provided.draggableProps}>
```

* 드래그 **시작 자체가 안 됨**
* 키보드 드래그도 불가
* 접근성 깨짐

👉 draggable은 “존재”하지만 **잡을 수 없음**

---

## 5️⃣ draggableProps vs dragHandleProps (정확한 역할 분리) 🧠

| 구분                | 역할                         |
| ----------------- | -------------------------- |
| `draggableProps`  | “이 DOM은 **움직일 수 있는 대상**이다” |
| `dragHandleProps` | “이 DOM을 **잡으면 드래그를 시작한다**” |

👉 둘은 **완전히 다른 개념**입니다.

---

## 6️⃣ 왜 className이나 onMouseDown으로 대체하면 안 되나? 🚫

```jsx
<div onMouseDown={startDrag}>...</div> // ❌
```

이렇게 하면:

* 키보드 드래그 ❌
* 스크린 리더 ❌
* 모바일 터치 제스처 ❌
* 내부 상태 동기화 ❌

👉 `dragHandleProps`는 **접근성 + 멀티 입력 디바이스 + 내부 상태 관리**를 한 번에 해결합니다.

---

## 7️⃣ 실무에서 꼭 기억할 규칙 3가지 ✅

1. **드래그 가능한 모든 Draggable에는 반드시 dragHandleProps가 붙은 DOM이 있어야 한다**
2. **버튼/인풋이 많은 카드일수록 핸들을 분리하라**
3. **dragHandleProps는 절대 커스터마이징하지 말고 spread만 하라**

---

## 8️⃣ 한 문장 요약 🧩

> **`dragHandleProps`는
> 드래그 “시작 지점”을 정의하는 hello-pangea 전용 props 묶음이며,
> UX·접근성·멀티 디바이스 대응을 위해 반드시 사용해야 합니다.**

