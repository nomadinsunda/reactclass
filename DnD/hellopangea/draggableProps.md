**`provided.draggableProps`는 전적으로 `@hello-pangea/dnd`가 제공하는 객체**입니다.
개발자가 만드는 게 아니라, **라이브러리가 내부 DnD 엔진 상태를 기반으로 “주입해 주는 값”** 이에요.

아래에서 **어디서, 언제, 왜 생기는지**를 정확히 짚어드리겠습니다.

---

## 1️⃣ 누가 `provided.draggableProps`를 만들까? 🧠

```jsx
<Draggable draggableId="card-1" index={0}>
  {(provided) => (
    <div {...provided.draggableProps} />
  )}
</Draggable>
```

이 구조에서:

* `Draggable` 컴포넌트는 **@hello-pangea/dnd 내부 로직**
* `provided` 객체는 **Draggable가 render props 함수에 넘겨주는 아규먼트**
* 그 안의 `draggableProps`는 **라이브러리가 생성해서 주입**

👉 **100% 라이브러리 소유 객체**입니다.

---

## 2️⃣ `provided.draggableProps` 안에는 뭐가 들어 있나? 🔍

공식 문서 기준 + 실제 DOM 기준으로 보면 대략 이런 성격의 값들이 들어 있습니다:

```js
provided.draggableProps = {
  style: {
    transform: 'translate(0px, 120px)',
    transition: 'transform 200ms cubic-bezier(...)',
  },
  'data-rfd-draggable-id': 'card-1',
  'data-rfd-draggable-context-id': '0',
  role: 'button',
  tabIndex: 0,
  onTransitionEnd: fn,
  // 접근성/키보드 드래그 관련 속성들
}
```

⚠️ 주의
이 구조는 **공식 API가 아니라 내부 구현 세부사항**이기 때문에
“정확히 어떤 key가 있다”에 의존하면 안 됩니다.

**우리가 보장받는 건 단 하나**입니다:

> 👉 **이 객체를 root DOM에 spread 해야 Draggable이 정상 동작한다**

---

## 3️⃣ 왜 `draggableProps`를 따로 제공할까? 🤔

### 이유 1️⃣ DOM 구조를 강제하지 않기 위해

라이브러리가 이런 식이었다면:

```jsx
<Draggable>
  <div className="fixed-structure">...</div>
</Draggable>
```

* 개발자는 레이아웃/마크업을 바꿀 수 없습니다.

하지만 render props 패턴으로:

```jsx
{(provided) => (
  <section>
    <article>
      <div {...provided.draggableProps} />
    </article>
  </section>
)}
```

👉 **어디가 draggable root인지 개발자가 결정**할 수 있습니다.

---

### 이유 2️⃣ React 이벤트 시스템과 자연스럽게 결합

DnD는:

* pointer events
* keyboard navigation
* focus 관리
* ARIA role

같은 걸 다룹니다.

이를 **props 형태로 노출**하면:

* React의 합성 이벤트 시스템 안에서 동작
* 사용자 정의 이벤트와 충돌 최소화
* SSR / Strict Mode 대응이 쉬워짐

---

### 이유 3️⃣ 스타일 제어를 개발자에게 열어두기 위해

`draggableProps.style`을 보면:

* 이동(transform)
* 애니메이션(transition)

같은 **핵심 제어 값**이 들어 있습니다.

개발자가:

```jsx
style={{
  ...provided.draggableProps.style,
  opacity: 0.8,
}}
```

처럼 **덮어쓰기/확장**할 수 있게 하려는 설계입니다.

---

## 4️⃣ 개발자가 “하면 안 되는 것” 🚫

### ❌ `provided.draggableProps`를 직접 수정

```js
provided.draggableProps.style.opacity = 0.5 // ❌
```

* 라이브러리 내부 상태와 어긋날 수 있습니다.

### ❌ 일부만 골라 쓰기

```jsx
<div style={provided.draggableProps.style} /> // ❌
```

* data-*, role, 이벤트 핸들러가 빠짐

### ❌ spread 순서 뒤집기

```jsx
<div
  style={{ opacity: 0.8 }}
  {...provided.draggableProps} // ❌ (style 덮어써짐)
>
```

* 라이브러리가 의도한 스타일이 사라질 수 있습니다.

---

## 5️⃣ 한 줄로 정리하면 🧩

> **`provided.draggableProps`는 `@hello-pangea/dnd`가 내부 DnD 엔진 상태를 바탕으로 생성해 주는 “드래그 가능한 DOM에 반드시 붙여야 하는 속성 묶음”입니다.**



원하시면 이 두 설계 포인트도 내부 구조 관점에서 바로 이어서 설명드리겠습니다.
