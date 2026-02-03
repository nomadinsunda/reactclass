# React Drag & Drop (Native + dnd-kit + @hello-pangea/dnd + react-dnd) 🧲🖱️📦

> “드래그 UI”가 아니라 **입력(마우스/터치/키보드) → 충돌 판정 → 시각 피드백 → 상태(데이터) 재배치**를 설계하는 일입니다. 🧠✨

---

## 0) React에서 DnD의 본질 🔥

DnD는 크게 5단계 파이프라인으로 이해하면 “갑자기 쉬워집니다.” 🧵

1. **입력 감지** 🖱️📱⌨️
2. **드래그 상태(무엇을 잡아 들고 있는가 / 어디 위인가)** 🧠
3. **충돌 판정(어느 droppable 위인가)** 🎯
4. **시각적 피드백(overlay, placeholder, transform)** 🪄
5. **데이터 업데이트(배열/정규화 모델 재배치)** 🔁

> 핵심: DnD는 DOM을 옮기는 게 아니라, **상태를 바꾸고 React가 다시 그리게 하는 패턴**입니다. ✅

---

## 1) 4가지 접근 방식 한눈에 보기 🧭

### ✅ Native HTML5 DnD

* 의존성 0, 빠르게 가능 🧱
* 하지만 모바일/정교한 UX/접근성에서 제약이 큼 ⚠️

### ✅ dnd-kit

* 현대적/확장성/성능/접근성 중심 🧩⚡
* “원하는 UX를 조립”하는 느낌 (센서/충돌판정/전략) ([GitHub][1])

### ✅ @hello-pangea/dnd

* Trello 같은 **칸반 UX에 특화** 🧰📌
* 드래그 중 placeholder/레이아웃 처리 감각이 좋음
* npm 기준 주간 다운로드도 매우 큼 ([NPM][2])

### ✅ react-dnd

* “정렬”보다 **드래그 객체 ↔ 드롭 대상 간 규칙 모델링**에 강함 🧲🧱
* 타입 기반(ITEM TYPE) + drop 규칙 + monitor 관찰
* 주간 다운로드도 매우 큼 ([NPM][3])

---

## 2) Native HTML5 Drag & Drop (가볍지만 ‘벽’이 있음) 🧱😅

### 장점 👍

* 설치 없이 바로 사용 가능
* 브라우저 기본 이벤트 제공

### 단점 👎

* 모바일 터치 UX가 불안정/브라우저 차이
* 정교한 충돌 판정/리스트 재정렬 UX는 직접 구현 부담 ↑
* 접근성(키보드 DnD)까지 고려하면 난이도 상승

### 예시: “리스트 재정렬” 최소 구현 🧪

```jsx
import React from "react"

export default function NativeSortable() {
  const [items, setItems] = React.useState(["A", "B", "C", "D"])
  const dragIndexRef = React.useRef(null)

  return (
    <ul style={{ maxWidth: 360, margin: "24px auto", padding: 0 }}>
      {items.map((it, idx) => (
        <li
          key={it}
          draggable
          onDragStart={() => (dragIndexRef.current = idx)}
          onDragOver={(e) => e.preventDefault()} // drop 허용
          onDrop={() => {
            const from = dragIndexRef.current
            const to = idx
            if (from == null || from === to) return

            const next = [...items]
            const [moved] = next.splice(from, 1)
            next.splice(to, 0, moved)
            setItems(next)

            dragIndexRef.current = null
          }}
          style={{
            listStyle: "none",
            padding: 12,
            border: "1px solid #ddd",
            borderRadius: 10,
            marginBottom: 10,
            background: "white",
            cursor: "grab",
          }}
        >
          {it}
        </li>
      ))}
    </ul>
  )
}
```

> 학습/프로토타입에는 충분 ✅
> 칸반/모바일/접근성/정교한 UX면 라이브러리 쪽이 현실적입니다. 💡

---

## 3) dnd-kit (엔진 분해형: 센서 + 충돌판정 + 전략) 🧩⚡

dnd-kit의 방향성은 한 문장으로 이렇습니다:

* **센서(Sensors)**: 입력(포인터/터치/키보드)을 분리
* **Collision detection**: 어떤 대상 위인지 계산
* **Sortable**: “정렬”을 위한 상위 추상화 제공 ([GitHub][1])

또한 `@dnd-kit/core` 최신 버전 정보는 npm에 공개되어 있고(예: 6.3.1) ([NPM][4]), 실무에서 널리 사용됩니다.

### 예시: Sortable + DragOverlay + DragHandle (실무형) 🧷✨

```jsx
import React from "react"
import {
  DndContext,
  PointerSensor,
  KeyboardSensor,
  useSensor,
  useSensors,
  closestCenter,
  DragOverlay,
} from "@dnd-kit/core"
import {
  SortableContext,
  useSortable,
  arrayMove,
  verticalListSortingStrategy,
  sortableKeyboardCoordinates,
} from "@dnd-kit/sortable"
import { CSS } from "@dnd-kit/utilities"

function SortableItem({ id, title }) {
  const { setNodeRef, attributes, listeners, transform, transition, isDragging } =
    useSortable({ id })

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.25 : 1,
    border: "1px solid #ddd",
    borderRadius: 12,
    padding: 12,
    background: "white",
    display: "flex",
    alignItems: "center",
    gap: 10,
  }

  return (
    <div ref={setNodeRef} style={style}>
      {/* ✅ Drag Handle */}
      <button
        {...listeners}
        {...attributes}
        style={{
          cursor: "grab",
          border: "1px solid #ccc",
          borderRadius: 10,
          padding: "6px 10px",
          background: "#f7f7f7",
        }}
        aria-label="drag handle"
      >
        ⠿
      </button>
      <div style={{ flex: 1 }}>
        <div style={{ fontWeight: 800 }}>{title}</div>
        <div style={{ fontSize: 12, color: "#666" }}>id: {id}</div>
      </div>
    </div>
  )
}

export default function DndKitSortable() {
  const [items, setItems] = React.useState([
    { id: "c1", title: "카드 1" },
    { id: "c2", title: "카드 2" },
    { id: "c3", title: "카드 3" },
    { id: "c4", title: "카드 4" },
  ])

  const [activeId, setActiveId] = React.useState(null)
  const ids = items.map((x) => x.id)
  const activeItem = items.find((x) => x.id === activeId)

  const sensors = useSensors(
    useSensor(PointerSensor, { activationConstraint: { distance: 8 } }), // 클릭-드래그 오작동 방지 🧯
    useSensor(KeyboardSensor, { coordinateGetter: sortableKeyboardCoordinates })
  )

  return (
    <div style={{ maxWidth: 420, margin: "40px auto" }}>
      <h2 style={{ fontSize: 20, fontWeight: 900, marginBottom: 12 }}>
        dnd-kit Sortable 🧩
      </h2>

      <DndContext
        sensors={sensors}
        collisionDetection={closestCenter}
        onDragStart={(e) => setActiveId(e.active.id)}
        onDragCancel={() => setActiveId(null)}
        onDragEnd={(e) => {
          const { active, over } = e
          setActiveId(null)
          if (!over || active.id === over.id) return

          const oldIndex = ids.indexOf(active.id)
          const newIndex = ids.indexOf(over.id)
          setItems((prev) => arrayMove(prev, oldIndex, newIndex))
        }}
      >
        <SortableContext items={ids} strategy={verticalListSortingStrategy}>
          <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
            {items.map((it) => (
              <SortableItem key={it.id} id={it.id} title={it.title} />
            ))}
          </div>
        </SortableContext>

        {/* ✅ DragOverlay: “떠 있는 카드” */}
        <DragOverlay>
          {activeItem ? (
            <div
              style={{
                width: 380,
                border: "1px solid #bbb",
                borderRadius: 12,
                padding: 12,
                background: "white",
                boxShadow: "0 10px 30px rgba(0,0,0,0.15)",
              }}
            >
              <div style={{ fontWeight: 900 }}>{activeItem.title}</div>
              <div style={{ fontSize: 12, color: "#666" }}>드래그 중… ✨</div>
            </div>
          ) : null}
        </DragOverlay>
      </DndContext>
    </div>
  )
}
```

**dnd-kit이 특히 좋은 경우** ✅

* “칸반도 하되” 커스텀 UX가 많다
* 키보드/센서/충돌판정/전략을 튜닝해야 한다
* 성능/확장성을 길게 보고 간다

---

## 4) @hello-pangea/dnd (칸반 UX 특화: provided/snapshot) 🧰📌

@hello-pangea/dnd는 react-beautiful-dnd 계열의 “칸반 감성”을 유지한 라이브러리입니다.
npm 기준 버전/주간 다운로드도 매우 활발한 편입니다. ([NPM][2])

### provided / snapshot의 의미가 핵심 🧠

* **provided**: “DOM에 DnD 기능을 연결하는 케이블” 🔌

  * `provided.innerRef`, `provided.droppableProps`, `provided.draggableProps`, `provided.dragHandleProps`
* **snapshot**: “현재 드래그 상태 텔레메트리(실시간 UI 반응)” 👀

  * `snapshot.isDragging`, `snapshot.isDraggingOver` 등

### 예시: (미니) 칸반 스타일로 리스트/카드 드래그 📌

```jsx
import React from "react"
import { DragDropContext, Droppable, Draggable } from "@hello-pangea/dnd"

const initial = {
  todo: [{ id: "c1", title: "할 일 1" }, { id: "c2", title: "할 일 2" }],
  doing: [{ id: "c3", title: "진행 1" }],
}

function moveBetweenColumns(state, source, destination) {
  const src = Array.from(state[source.droppableId])
  const dst = Array.from(state[destination.droppableId])
  const [moved] = src.splice(source.index, 1)
  dst.splice(destination.index, 0, moved)

  return {
    ...state,
    [source.droppableId]: src,
    [destination.droppableId]: dst,
  }
}

export default function PangeaMiniKanban() {
  const [cols, setCols] = React.useState(initial)

  return (
    <div style={{ display: "flex", gap: 12, padding: 24 }}>
      <DragDropContext
        onDragEnd={(result) => {
          const { source, destination } = result
          if (!destination) return

          // 같은 컬럼 내 reorder
          if (source.droppableId === destination.droppableId) {
            const next = Array.from(cols[source.droppableId])
            const [moved] = next.splice(source.index, 1)
            next.splice(destination.index, 0, moved)
            setCols((prev) => ({ ...prev, [source.droppableId]: next }))
            return
          }

          // 컬럼 간 이동
          setCols((prev) => moveBetweenColumns(prev, source, destination))
        }}
      >
        {Object.entries(cols).map(([colId, cards]) => (
          <Droppable key={colId} droppableId={colId}>
            {(provided, snapshot) => (
              <div
                ref={provided.innerRef}
                {...provided.droppableProps}
                style={{
                  width: 260,
                  padding: 12,
                  borderRadius: 14,
                  background: snapshot.isDraggingOver ? "#e0f2fe" : "#f1f5f9",
                }}
              >
                <h3 style={{ fontWeight: 900, marginBottom: 10 }}>{colId}</h3>

                {cards.map((card, index) => (
                  <Draggable key={card.id} draggableId={card.id} index={index}>
                    {(provided, snapshot) => (
                      <div
                        ref={provided.innerRef}
                        {...provided.draggableProps}
                        {...provided.dragHandleProps}
                        style={{
                          ...provided.draggableProps.style,
                          padding: 12,
                          borderRadius: 12,
                          border: "1px solid #ddd",
                          background: "white",
                          marginBottom: 10,
                          opacity: snapshot.isDragging ? 0.6 : 1,
                        }}
                      >
                        {card.title}
                      </div>
                    )}
                  </Draggable>
                ))}

                {/* ✅ placeholder: 드래그 중 레이아웃 안정화 */}
                {provided.placeholder}
              </div>
            )}
          </Droppable>
        ))}
      </DragDropContext>
    </div>
  )
}
```

**@hello-pangea/dnd가 특히 좋은 경우** ✅

* Trello 같은 “칸반 UX”를 빠르게 안정적으로
* placeholder/레이아웃 안정화가 중요
* 정형화된 UX에 맞추는 게 편하다

---

## 5) react-dnd (정렬보다 “규칙/타입/상호작용” 엔진) 🧲🧱

react-dnd는 관점이 완전히 다릅니다.

* dnd-kit / pangea: “정렬/칸반 UX”에 초점
* **react-dnd**: “드래그 아이템(데이터 객체) ↔ 드롭 타겟(규칙)”의 상호작용에 초점

npm 기준으로도 널리 사용되며(주간 다운로드/버전 정보) ([NPM][3]),
“대시보드 위젯 배치, 폼 빌더, 에디터” 같은 **도메인 규칙이 강한 DnD**에서 빛납니다. 🔥

### 핵심 개념 🧠

* **DndProvider + Backend**: HTML5Backend 등
* **Drag Item**: “드래그하는 데이터 객체” 📦
* **useDrag**: 이 컴포넌트는 드래그 가능 🖱️
* **useDrop**: 이 컴포넌트는 드롭 가능 🎯
* **monitor**: 현재 드래그 상태 관찰(수집/조건/효과) 👀

### 예시: “CARD 타입만 받고, 조건(canDrop)으로 거르기” 🎯

```jsx
import React from "react"
import { DndProvider, useDrag, useDrop } from "react-dnd"
import { HTML5Backend } from "react-dnd-html5-backend"

const ItemTypes = { CARD: "CARD" }

function Card({ id, title, listId }) {
  const [{ isDragging }, dragRef] = useDrag(() => ({
    type: ItemTypes.CARD,
    item: { id, listId },
    collect: (monitor) => ({ isDragging: monitor.isDragging() }),
  }))

  return (
    <div
      ref={dragRef}
      style={{
        padding: 12,
        borderRadius: 12,
        border: "1px solid #ddd",
        background: "white",
        marginBottom: 10,
        opacity: isDragging ? 0.4 : 1,
        cursor: "grab",
      }}
    >
      {title}
    </div>
  )
}

function List({ id, title, onMoveCard }) {
  const [{ isOver, canDrop }, dropRef] = useDrop(() => ({
    accept: ItemTypes.CARD,
    canDrop: (item) => item.listId !== id, // ✅ 같은 리스트로는 drop 금지
    drop: (item) => onMoveCard(item.id, item.listId, id),
    collect: (monitor) => ({
      isOver: monitor.isOver(),
      canDrop: monitor.canDrop(),
    }),
  }))

  return (
    <div
      ref={dropRef}
      style={{
        width: 280,
        padding: 12,
        borderRadius: 14,
        background: isOver && canDrop ? "#dcfce7" : "#f1f5f9",
      }}
    >
      <h3 style={{ fontWeight: 900, marginBottom: 10 }}>{title}</h3>
      <div style={{ fontSize: 12, color: "#64748b" }}>
        {canDrop ? "여기로 드롭 가능 ✅" : "드롭 불가 ❌"}
      </div>
    </div>
  )
}

export default function ReactDndMini() {
  const [state, setState] = React.useState({
    L1: [{ id: "c1", title: "카드 1" }, { id: "c2", title: "카드 2" }],
    L2: [{ id: "c3", title: "카드 3" }],
  })

  function onMoveCard(cardId, fromListId, toListId) {
    if (fromListId === toListId) return

    setState((prev) => {
      const from = [...prev[fromListId]]
      const to = [...prev[toListId]]
      const idx = from.findIndex((c) => c.id === cardId)
      const [moved] = from.splice(idx, 1)
      to.unshift(moved) // 예: 대상 리스트 맨 위로 📌
      return { ...prev, [fromListId]: from, [toListId]: to }
    })
  }

  return (
    <DndProvider backend={HTML5Backend}>
      <div style={{ display: "flex", gap: 12, padding: 24 }}>
        <div style={{ width: 280 }}>
          <List id="L1" title="List 1" onMoveCard={onMoveCard} />
          <div style={{ marginTop: 10 }}>
            {state.L1.map((c) => (
              <Card key={c.id} id={c.id} title={c.title} listId="L1" />
            ))}
          </div>
        </div>

        <div style={{ width: 280 }}>
          <List id="L2" title="List 2" onMoveCard={onMoveCard} />
          <div style={{ marginTop: 10 }}>
            {state.L2.map((c) => (
              <Card key={c.id} id={c.id} title={c.title} listId="L2" />
            ))}
          </div>
        </div>
      </div>
    </DndProvider>
  )
}
```

**react-dnd가 특히 좋은 경우** ✅

* 드래그 타입이 여러 개(CARD, COLUMN, WIDGET…)
* “A는 B에만 drop 가능” 같은 규칙이 많다
* 대시보드/에디터/빌더처럼 “정렬”보다 “상호작용 모델링”이 핵심

---

## 6) 실무에서 제일 중요한 “상태 설계” 팁 🧠🧱

칸반/보드 구조에서는 **정규화(normalized)** 가 DnD의 난이도를 확 줄입니다.
> 정규화 상태란,
> “중첩된 UI 구조를 그대로 저장하지 않고,
> 엔티티를 ID 기준으로 분리하고
> ‘관계’만 배열로 관리하는 방식” 입니다.

### ❌ 비정규화된 상태 (DnD 지옥문 😵‍💫)
```
const board = {
  id: 'board-1',
  lists: [
    {
      id: 'list-1',
      title: 'Todo',
      cards: [
        { id: 'c1', title: '할 일 1' },
        { id: 'c2', title: '할 일 2' },
      ],
    },
    {
      id: 'list-2',
      title: 'Doing',
      cards: [
        { id: 'c3', title: '진행 1' },
      ],
    },
  ],
}
```

### ✅ 정규화된 상태 (DnD가 쉬워지는 구조 😌)
```
const state = {
  listsById: {
    'list-1': {
      id: 'list-1',
      title: 'Todo',
      cardIds: ['c1', 'c2'],
    },
    'list-2': {
      id: 'list-2',
      title: 'Doing',
      cardIds: ['c3'],
    },
  },

  cardsById: {
    c1: { id: 'c1', title: '할 일 1' },
    c2: { id: 'c2', title: '할 일 2' },
    c3: { id: 'c3', title: '진행 1' },
  },

  listOrder: ['list-1', 'list-2'],
}

```

---

✅ 추천 형태

* `listsById: { [listId]: { id, title, cardIds: [] } }`
* `cardsById: { [cardId]: { id, title, ... } }`

👉 카드 이동/정렬은 결국 `cardIds` 배열만 바꾸면 끝이라서,
UI가 커져도 제어가 쉬워집니다. 🔁

---

## 7) 성능 & UX 체크리스트 ⚡✅

* 드래그 중에 `onDragMove`로 state를 계속 바꾸지 마세요 (프레임 드랍 💥)
* `React.memo` + 안정적인 props로 카드 리렌더 최소화 🧊
* 드래그 핸들(⠿)을 분리하면 클릭/스크롤 UX가 좋아짐 🧷
* 모바일이면 “드래그 시작 거리(distance)” 같은 제약으로 오작동 방지 🧯
* 접근성: 키보드 센서/포커스/스크린리더 메시지 고려 ♿⌨️

---

## 8) 그래서 뭘 쓰면 되나요? 최종 선택 가이드 🧭✨

* **리스트 재정렬/커스텀 UX/성능/확장성** → **dnd-kit** 🧩⚡ ([GitHub][1])
* **칸반(Trello) UX를 빠르게/안정적으로** → **@hello-pangea/dnd** 🧰📌 ([NPM][2])
* **타입/규칙/상호작용(에디터/빌더/위젯 배치)** → **react-dnd** 🧲🧱 ([NPM][3])
* **아주 단순한 드롭/학습용** → Native 🧱

[1]: https://github.com/clauderic/dnd-kit?utm_source=chatgpt.com "clauderic/dnd-kit"
[2]: https://www.npmjs.com/package/%40hello-pangea/dnd?utm_source=chatgpt.com "hello-pangea/dnd"
[3]: https://www.npmjs.com/package/react-dnd?activeTab=dependents&utm_source=chatgpt.com "react-dnd"
[4]: https://www.npmjs.com/package/%40dnd-kit/core?activeTab=dependents&utm_source=chatgpt.com "dnd-kit/core"
