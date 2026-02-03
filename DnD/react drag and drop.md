# React Drag & Drop 🧲🖱️



React에서 Drag & Drop(DnD)을 한다는 건, 단순히 “마우스로 끌어다 놓기”가 아니라 **사용자 입력(마우스/터치/키보드) → 드래그 상태 관리 → 충돌 판정 → 시각적 피드백 → 데이터 재정렬/이동**을 안정적으로 연결하는 작업입니다. 🧠

아래는 실무에서 가장 많이 쓰는 방식 3가지입니다. ✅

1. **Native HTML5 Drag & Drop API** (가벼움, 하지만 제약 많음) 🧱
2. **dnd-kit** (현대적/유연/성능 좋음) 🧩⚡
3. **@hello-pangea/dnd** (react-beautiful-dnd 계열, “칸반 보드”에 특화) 🧰📌
   (+ **react-dnd**: 복잡한 드롭 규칙/다양한 “드롭 타겟”을 다루는 고전 강자)

---

## 1) React에서 DnD의 “본질” 🔥

DnD는 크게 5단계 파이프라인으로 이해하면 깔끔합니다. 🧵

### (1) 입력 감지 🖱️📱⌨️

* Pointer(마우스), Touch(터치), Keyboard(키보드) 등에서 “드래그 시작”을 판단
* 예: `mousedown` → 일정 거리 이상 이동하면 drag start

### (2) 드래그 상태(Drag State) 저장 🧠

* “지금 뭐를 들고 있는지(active)”
* “어디 위에 올라가 있는지(over)”
* “원래 위치/현재 위치”

### (3) 충돌 판정(Collision Detection) 🎯

* 커서/아이템 기준으로 **어떤 droppable 위에 있는지** 계산
* 예: `closestCenter`, `rectIntersection` 같은 알고리즘

### (4) 시각적 피드백 🪄

* 드래그 중인 아이템을 떠 있는 것처럼 보이게(Transform)
* 대상 영역 하이라이트
* placeholder, overlay, z-index, opacity 등

### (5) 데이터 업데이트(진짜 핵심) 🧱➡️🧱

* UI가 아닌 **상태(배열/맵)** 를 이동/재정렬
* React에선 **불변 업데이트**로 재구성

> 결론: DnD는 “DOM을 옮기는 기술”이 아니라, **상태를 재배치하고 그 결과를 렌더링**하는 패턴입니다. 🔁

---

## 2) Native HTML5 Drag & Drop (짧고 간단하지만… 😅)

### 장점 👍

* 라이브러리 없이 즉시 가능
* 기본 이벤트(`dragstart`, `dragover`, `drop`) 제공

### 단점 👎 (실무에서 자주 부딪힘)

* **모바일 터치 지원이 애매**하거나 브라우저별 차이 큼 📱
* 드래그 중 커스텀 UI/정교한 충돌판정이 어렵다
* 리스트 재정렬 같은 UX는 구현 난이도 상승

### 아주 간단한 예시 🧪

```jsx
function NativeDnD() {
  const [items, setItems] = React.useState(["A", "B", "C"])
  const dragIndexRef = React.useRef(null)

  return (
    <ul>
      {items.map((it, idx) => (
        <li
          key={it}
          draggable
          onDragStart={() => (dragIndexRef.current = idx)}
          onDragOver={(e) => e.preventDefault()} // drop 허용
          onDrop={() => {
            const from = dragIndexRef.current
            const to = idx
            if (from === null || from === to) return

            const next = [...items]
            const [moved] = next.splice(from, 1)
            next.splice(to, 0, moved)
            setItems(next)
            dragIndexRef.current = null
          }}
          style={{ padding: 12, border: "1px solid #ddd", marginBottom: 8 }}
        >
          {it}
        </li>
      ))}
    </ul>
  )
}
```

> “학습용/초간단”으로는 OK ✅
> “칸반/정교한 UX/모바일/접근성”이면 라이브러리를 추천합니다. 💡

---

## 3) dnd-kit (요즘 실무에서 가장 ‘엔진다운’ 선택 🧩⚡)

dnd-kit은 React DnD를 **센서(sensor) + 충돌판정 + 정렬전략(strategy)** 으로 분해해 제공합니다.
즉, “원하는 UX를 조립”하기 좋습니다. 🛠️

### 핵심 개념 4개 🧠

* **DndContext**: DnD의 최상위 컨텍스트 🌍
* **Sensors**: 마우스/터치/키보드 입력을 감지 🎛️
* **Droppable / Draggable**: 드롭 영역/드래그 대상 🧲
* **Sortable**: “리스트 재정렬”을 위한 상위 추상화 📦↕️

---

### ✅ dnd-kit Sortable 예시 (실무형: 드래그 핸들 + 오버레이 + 재정렬) 🧷✨

> 아래 예시는 “카드 리스트 정렬”을 꽤 실전처럼 구성했습니다.

```jsx
import React from "react"
import {
  DndContext,
  PointerSensor,
  KeyboardSensor,
  useSensor,
  useSensors,
  closestCenter,
} from "@dnd-kit/core"
import {
  SortableContext,
  useSortable,
  arrayMove,
  verticalListSortingStrategy,
  sortableKeyboardCoordinates,
} from "@dnd-kit/sortable"
import { CSS } from "@dnd-kit/utilities"
import { DragOverlay } from "@dnd-kit/core"

function SortableItem({ id, title }) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
  } = useSortable({ id })

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.3 : 1,
    border: "1px solid #ddd",
    padding: 12,
    borderRadius: 10,
    background: "white",
    display: "flex",
    gap: 10,
    alignItems: "center",
  }

  return (
    <div ref={setNodeRef} style={style}>
      {/* ✅ Drag Handle */}
      <button
        {...listeners}
        {...attributes}
        style={{
          cursor: "grab",
          padding: "6px 10px",
          borderRadius: 8,
          border: "1px solid #ccc",
          background: "#f7f7f7",
        }}
        aria-label="drag handle"
      >
        ⠿
      </button>

      <div style={{ flex: 1 }}>
        <div style={{ fontWeight: 700 }}>{title}</div>
        <div style={{ fontSize: 12, color: "#666" }}>id: {id}</div>
      </div>
    </div>
  )
}

export default function DndKitSortableDemo() {
  const [items, setItems] = React.useState([
    { id: "c1", title: "카드 1" },
    { id: "c2", title: "카드 2" },
    { id: "c3", title: "카드 3" },
    { id: "c4", title: "카드 4" },
  ])

  const [activeId, setActiveId] = React.useState(null)

  // ✅ Sensors: 포인터 + 키보드
  const sensors = useSensors(
    useSensor(PointerSensor, {
      activationConstraint: { distance: 8 }, // 실수 클릭 방지 🧯
    }),
    useSensor(KeyboardSensor, {
      coordinateGetter: sortableKeyboardCoordinates,
    })
  )

  const ids = items.map((it) => it.id)
  const activeItem = items.find((it) => it.id === activeId)

  function handleDragStart(event) {
    setActiveId(event.active.id)
  }

  function handleDragEnd(event) {
    const { active, over } = event
    setActiveId(null)

    if (!over) return
    if (active.id === over.id) return

    const oldIndex = ids.indexOf(active.id)
    const newIndex = ids.indexOf(over.id)
    setItems((prev) => arrayMove(prev, oldIndex, newIndex))
  }

  function handleDragCancel() {
    setActiveId(null)
  }

  return (
    <div style={{ maxWidth: 420, margin: "40px auto" }}>
      <h2 style={{ fontSize: 20, fontWeight: 800, marginBottom: 12 }}>
        dnd-kit Sortable 리스트 🧩
      </h2>

      <DndContext
        sensors={sensors}
        collisionDetection={closestCenter}
        onDragStart={handleDragStart}
        onDragEnd={handleDragEnd}
        onDragCancel={handleDragCancel}
      >
        <SortableContext items={ids} strategy={verticalListSortingStrategy}>
          <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
            {items.map((it) => (
              <SortableItem key={it.id} id={it.id} title={it.title} />
            ))}
          </div>
        </SortableContext>

        {/* ✅ DragOverlay: 드래그 중 “떠있는 카드” */}
        <DragOverlay>
          {activeItem ? (
            <div
              style={{
                border: "1px solid #bbb",
                padding: 12,
                borderRadius: 10,
                background: "white",
                boxShadow: "0 10px 30px rgba(0,0,0,0.15)",
                width: 380,
              }}
            >
              <div style={{ fontWeight: 800 }}>{activeItem.title}</div>
              <div style={{ fontSize: 12, color: "#666" }}>
                드래그 중… ✨
              </div>
            </div>
          ) : null}
        </DragOverlay>
      </DndContext>
    </div>
  )
}
```

#### 이 코드가 “실무 감성”인 포인트 🧠✨

* **activationConstraint(distance: 8)** → 클릭과 드래그를 구분 (실수 방지) 🧯
* **DragOverlay** → 드래그 시 시각적 퀄리티 상승 🎨
* **KeyboardSensor** → 접근성(키보드 DnD)까지 고려 ⌨️
* 상태 업데이트는 `arrayMove`로 **불변 재정렬** 🔁

---

## 4) @hello-pangea/dnd (칸반 보드에 특화 📌🧰)

이 계열은 “Trello 같은 칸반”을 만들 때 UX가 이미 잘 잡혀 있습니다.
**Droppable / Draggable + provided / snapshot** 패턴이 특징이에요. 🧱

### 핵심 개념 🧠

* `DragDropContext`: 전체 컨텍스트
* `Droppable`: 드롭 가능한 영역(리스트)
* `Draggable`: 드래그 가능한 카드
* `provided`: ref/props 묶음(라이브러리 연결용) 🔌
* `snapshot`: 현재 상태(드래그 중인지, 위에 올라갔는지) 👀

### “provided / snapshot”이 왜 중요하냐면요? 🔍

* **provided**는 “이 DOM에 드래그 기능을 붙이기 위한 연결 케이블”입니다.

  * `provided.innerRef` → DOM 참조 연결
  * `provided.draggableProps` / `provided.dragHandleProps` → 이벤트/속성 연결
* **snapshot**은 “현재 드래그 상황을 알려주는 실시간 텔레메트리”입니다.

  * `snapshot.isDragging` / `snapshot.isDraggingOver` 등을 활용해 스타일 변경

> 즉, `provided`는 **동작 연결**, `snapshot`은 **상태 기반 UI 반응** 🎛️🎨

---

## 5) 성능 & 구조 설계 팁 (실무에서 진짜 갈리는 부분 ⚡🧱)

### ✅ 상태 구조는 이렇게 잡으면 편합니다 🧠

칸반이라면 보통:

* `listsById: { [listId]: { id, title, cardIds: [] } }`
* `cardsById: { [cardId]: { id, title, ... } }`

즉 **정규화(normalized)** 구조가 DnD에 강합니다. 💪
왜냐하면 “재정렬”은 결국 `cardIds` 배열만 바꾸면 되니까요. 🔁

---

### ✅ 리렌더 최소화 팁 🧊

* 드래그 중에는 state를 과도하게 바꾸지 말기 (특히 `onDragMove`) 🚫
* 카드 컴포넌트는 `React.memo` + 안정적인 props 유지 🧷
* 큰 리스트면 **가상화(virtualization)** 고려 📦 (단, DnD와 결합 난이도 있음)

---

### ✅ 접근성(A11y)은 선택이 아니라 기본입니다 ♿✨

* 키보드로도 이동 가능해야 함 (`KeyboardSensor` 같은 것)
* 포커스 관리(드래그 시작/종료 후 어디에 포커스를 둘지)
* 스크린리더 안내(“이동됨”, “몇 번째 위치” 등)

---

## 6) 라이브러리 선택 가이드 🧭

### dnd-kit 추천 ✅

* 커스텀 UX/정교한 충돌판정/성능/확장성 중요 🧩⚡
* “내 앱에 맞는 드래그 경험”을 만들고 싶다

### @hello-pangea/dnd 추천 ✅

* “칸반 보드 중심”이고, 정형화된 UX가 편하다 📌
* provided/snapshot 패턴이 맞는다

### Native 추천 ✅

* 아주 단순한 드롭(파일 업로드 영역 같은) 📤
* 학습/프로토타입/의존성 최소


