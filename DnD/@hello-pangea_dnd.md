# 🧲 @hello-pangea/dnd

> react-beautiful-dnd의 정통 후계자

---

## 🧠 @hello-pangea/dnd란?

`@hello-pangea/dnd`는
👉 **react-beautiful-dnd의 커뮤니티 포크(fork)** 프로젝트입니다.

```text
react-beautiful-dnd (Atlassian)
        ↓ (유지보수 중단)
@hello-pangea/dnd (Community Driven)
```

즉,

> ❝ react-beautiful-dnd의 철학, API, 사용법을
> **그대로 계승하면서 계속 살아있는 프로젝트** ❞

---

## ❓ 왜 react-beautiful-dnd가 사라졌나?

Atlassian은 공식적으로 이렇게 말했습니다.

* 내부 제품(Jira 등)에서는 더 이상 사용하지 않음
* 유지보수 비용 대비 효율 문제
* React 생태계 변화 대응 어려움

👉 하지만 문제는…

> **너무 잘 만든 API라서 대체제가 바로 나오지 않음**

그래서 커뮤니티가 나섰고,
그 결과가 **@hello-pangea/dnd**입니다 🔥

---

## ✨ @hello-pangea/dnd의 핵심 특징

| 항목                    | 설명                 |
| --------------------- | ------------------ |
| 🧩 선언적 API            | DragDropContext 중심 |
| 📦 Batteries Included | 정렬, 애니메이션, 접근성 내장  |
| 🧠 레이아웃 인식            | DOM 위치 기반 계산       |
| 🎯 사용 난이도             | 낮음 (진입 장벽 ↓)       |
| 🧱 제약                 | 레이아웃 자유도 제한        |

---

## 🧠 철학 한 줄 요약

> ❝ **Drag & Drop을 “UI 문제”로 본다** ❞

이게 dnd-kit과의 가장 큰 차이입니다.

---

## 🆚 dnd-kit vs @hello-pangea/dnd (한눈 비교)

| 항목     | @hello-pangea/dnd | dnd-kit       |
| ------ | ----------------- | ------------- |
| 설계 철학  | UI 중심             | 상태 중심         |
| DOM 이동 | O (측정 기반)         | X (transform) |
| 자유도    | 중                 | 매우 높음         |
| 러닝커브   | 낮음                | 높음            |
| 모바일    | 제한적               | 매우 좋음         |
| 추천 용도  | 리스트/칸반            | 커스텀 DnD       |

---

## 🧩 기본 구조 이해하기

```text
DragDropContext
 └── Droppable
      └── Draggable
```

![Image](https://user-images.githubusercontent.com/7050871/31140337-f8575a6e-a828-11e7-835c-deadd9bd7068.gif)

![Image](https://codesandbox.io/api/v1/sandboxes/wvjklk/screenshot.png)

![Image](https://images.viblo.asia/f85d4f3c-0ced-4a3f-aac1-fc302d326a78.png)

---

## 🧠 핵심 개념 1 — DragDropContext

### 📌 모든 드래그의 시작점

```tsx
import { DragDropContext } from '@hello-pangea/dnd'

<DragDropContext
  onDragStart={onDragStart}
  onDragUpdate={onDragUpdate}
  onDragEnd={onDragEnd}
>
  {children}
</DragDropContext>
```

### DragDropContext의 책임

* 전체 드래그 상태 관리
* 현재 드래그 중인 아이템 추적
* 충돌 계산 & reorder 타이밍 제어

---

## 🧠 핵심 개념 2 — Droppable

> ❝ 여기는 떨어질 수 있는 영역이다 ❞

```tsx
<Droppable droppableId="todo-list">
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.droppableProps}
      className={`p-4 rounded ${
        snapshot.isDraggingOver ? 'bg-blue-100' : 'bg-gray-100'
      }`}
    >
      {children}
      {provided.placeholder}
    </div>
  )}
</Droppable>
```

### ❗ placeholder의 의미 (중요)

* 드래그 중 레이아웃 붕괴 방지
* DOM 재정렬 시 높이 보존

👉 **이게 DOM 기반 DnD의 핵심 트릭입니다**

---

## 🧠 핵심 개념 3 — Draggable

> ❝ 이 녀석은 움직일 수 있다 ❞

```tsx
<Draggable draggableId={card.id} index={index}>
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.draggableProps}
      {...provided.dragHandleProps}
      className={`
        p-3 mb-2 rounded shadow
        ${snapshot.isDragging ? 'bg-yellow-100' : 'bg-white'}
      `}
    >
      <h4 className="font-bold">{card.title}</h4>
      <p className="text-sm text-gray-500">{card.description}</p>
    </div>
  )}
</Draggable>
```

### snapshot으로 알 수 있는 것들

* `isDragging`
* `isDropAnimating`
* `draggingOver`

👉 **UI 제어에 최적화된 설계**

---

## 🧪 실전 예제 — Kanban 스타일 리스트 이동 (심플 ❌)

### 📦 상태 구조

```ts
const initialState = {
  todo: [
    { id: 't1', title: 'React Router 복습' },
    { id: 't2', title: 'DnD 개념 정리' },
  ],
  doing: [
    { id: 'd1', title: 'Kanban UI 구현' },
  ],
}
```

---

### 🧠 onDragEnd 로직 (핵심 중의 핵심)

```ts
const onDragEnd = (result) => {
  const { source, destination } = result

  if (!destination) return

  if (
    source.droppableId === destination.droppableId &&
    source.index === destination.index
  ) {
    return
  }

  setState((prev) => {
    const sourceList = [...prev[source.droppableId]]
    const [moved] = sourceList.splice(source.index, 1)

    const destList =
      source.droppableId === destination.droppableId
        ? sourceList
        : [...prev[destination.droppableId]]

    destList.splice(destination.index, 0, moved)

    return {
      ...prev,
      [source.droppableId]: sourceList,
      [destination.droppableId]: destList,
    }
  })
}
```

📌 **이 라이브러리의 본질은 결국 “배열 재정렬”**

---

## ⚠️ @hello-pangea/dnd의 한계 (중요)

### ❌ 1. 모바일 UX가 완벽하지 않음

* 터치 드래그 불안정
* long-press UX 필요

### ❌ 2. 레이아웃 제약

* position: absolute
* transform 중첩
* overflow hidden

### ❌ 3. 커스텀 충돌 전략 불가

* 내부 계산 로직 블랙박스

---

## 🎯 언제 @hello-pangea/dnd를 써야 할까?

✅ Trello / Jira 스타일 UI
✅ 세로/가로 정렬 리스트
✅ 빠른 구현이 필요한 교육/프로토타입
✅ 팀원 숙련도가 낮을 때

❌ 완전 커스텀 DnD
❌ 모바일 중심 서비스
❌ Canvas / 게임 UI

---

## 🧠 dnd-kit vs @hello-pangea/dnd — 철학 차이 요약

```text
@hello-pangea/dnd
→ “DOM이 움직여야 사용자가 이해한다”

dnd-kit
→ “상태가 바뀌면 UI는 따라온다”
```

---

## 📌 마무리 요약

* `@hello-pangea/dnd`는 **아직도 충분히 강력**
* API 직관성은 여전히 최고 수준
* 단, **확장성과 자유도는 dnd-kit이 우위**
* **교육 / 빠른 실무 적용엔 매우 적합**


