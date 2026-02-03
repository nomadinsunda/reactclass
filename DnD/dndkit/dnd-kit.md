# 🧲 dnd-kit

> React에서 가장 현대적인 Drag & Drop 라이브러리

---

## ✨ dnd-kit이란?

**dnd-kit**은 React를 위한 **Headless Drag & Drop 프레임워크**입니다.

> ❌ “미리 만들어진 UI”
> ✅ “드래그 동작만 제공하고, UI는 개발자가 직접 구현”

즉,
**스타일·레이아웃에 전혀 간섭하지 않는 DnD 엔진**이라고 보시면 됩니다.

---

## 🔥 왜 dnd-kit인가?

기존 Drag & Drop 라이브러리의 문제점부터 짚어보겠습니다.

### ❌ 기존 라이브러리들의 한계

| 라이브러리               | 문제점                |
| ------------------- | ------------------ |
| react-beautiful-dnd | 유지보수 중단, 무겁고 제약 많음 |
| HTML5 Drag API      | 모바일 지원 지옥 😇       |
| react-dnd           | 개념이 복잡, 보일러플레이트 과다 |

---

### ✅ dnd-kit의 핵심 장점

| 특징           | 설명                          |
| ------------ | --------------------------- |
| 🎯 Headless  | UI/스타일 완전 자유                |
| 📱 모바일 지원    | Pointer / Touch 센서 기본 제공    |
| ⚡ 성능         | DOM 이동 ❌ → transform 기반     |
| 🧠 수학적 충돌 감지 | Rect 기반 collision detection |
| 🔌 확장성       | 정렬, 스냅, 드롭 규칙 커스터마이징        |

---

## 🧠 dnd-kit의 전체 구조

```text
사용자 입력 (마우스 / 터치)
        ↓
      Sensor
        ↓
  Drag Context (DndContext)
        ↓
Collision Detection
        ↓
Sortable / Droppable / Draggable
```

![Image](https://docs.dndkit.com/~gitbook/image?dpr=3\&quality=100\&sign=a73ed3c\&sv=2\&url=https%3A%2F%2F3633755066-files.gitbook.io%2F~%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MMujhzqaYbBEEmDxnZO%252Fuploads%252FrFnN48FwW1TuQlqZmp58%252Fconcepts-illustration-large.svg%3Falt%3Dmedia%26token%3D451a9922-8aba-426b-bd91-fa4721a71ef7\&width=768)

![Image](https://repository-images.githubusercontent.com/316086701/11437f7f-14e3-44d7-8641-2e9654fd80eb)

---

## 🧩 핵심 개념 1 — DndContext

### 🔑 모든 드래그의 시작점

```tsx
<DndContext
  sensors={sensors}
  collisionDetection={closestCenter}
  onDragStart={handleDragStart}
  onDragEnd={handleDragEnd}
>
  {children}
</DndContext>
```

📌 **DndContext 역할**

* 드래그 이벤트 수집
* 현재 활성 아이템 관리
* 충돌 감지 전략 실행
* 상태 전달 (context 기반)

> ⚠️ **React Context API 기반**
> → 불필요한 리렌더링 최소화 설계

---

## 🧲 핵심 개념 2 — Sensor 시스템

### 🎮 Sensor = “입력 장치 추상화”

```ts
import {
  useSensor,
  useSensors,
  PointerSensor,
  KeyboardSensor,
} from '@dnd-kit/core'

const sensors = useSensors(
  useSensor(PointerSensor, {
    activationConstraint: {
      distance: 8, // 8px 이동해야 드래그 시작
    },
  }),
  useSensor(KeyboardSensor)
)
```

### Sensor 종류

| Sensor         | 설명                 |
| -------------- | ------------------ |
| PointerSensor  | 마우스 + 터치           |
| TouchSensor    | 터치 전용              |
| KeyboardSensor | 접근성(Accessibility) |
| Custom Sensor  | 직접 구현 가능           |

👉 **HTML5 Drag API를 사용하지 않습니다!**

---

## 🧠 핵심 개념 3 — Collision Detection

### “지금 어디 위에 있는가?” 🤔

```ts
import { closestCenter } from '@dnd-kit/core'
```

#### 대표 전략들

| 전략               | 설명     |
| ---------------- | ------ |
| closestCenter    | 중심점 기준 |
| rectIntersection | 사각형 교차 |
| pointerWithin    | 포인터 위치 |
| custom           | 직접 구현  |

📌 **실무에서는 `closestCenter` + 정렬 UI 조합이 가장 안정적**

---

## 🔗 핵심 개념 4 — useDraggable / useDroppable

### 🎯 완전 Headless API

```ts
const {
  attributes,
  listeners,
  setNodeRef,
  transform,
  isDragging,
} = useDraggable({
  id: 'card-1',
})
```

```ts
const style = {
  transform: CSS.Translate.toString(transform),
  opacity: isDragging ? 0.5 : 1,
}
```

👉 **DOM 이동 ❌**
👉 **CSS transform 사용 ✅ (GPU 가속)**

---

## 🔄 핵심 개념 5 — Sortable 시스템

> 실무에서 dnd-kit의 꽃 🌸

```tsx
import {
  SortableContext,
  useSortable,
  arrayMove,
  verticalListSortingStrategy,
} from '@dnd-kit/sortable'
```

---

## 🧪 실습 예제 (심플 ❌ / 실전형 ✅)

### 📁 상태 구조 (엔티티 패턴)

```ts
{
  columns: {
    todo: ['a', 'b'],
    doing: ['c'],
  },
  cards: {
    a: { id: 'a', title: 'React 공부' },
    b: { id: 'b', title: 'Redux 정리' },
    c: { id: 'c', title: 'DnD 구현' },
  }
}
```

---

### 🧩 Sortable Card 컴포넌트

```tsx
function SortableCard({ id, title }) {
  const {
    setNodeRef,
    attributes,
    listeners,
    transform,
    transition,
    isDragging,
  } = useSortable({ id })

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.4 : 1,
  }

  return (
    <div
      ref={setNodeRef}
      style={style}
      {...attributes}
      {...listeners}
      className="p-3 bg-white shadow rounded cursor-grab"
    >
      {title}
    </div>
  )
}
```

---

### 🧠 Drag End 로직 (핵심!)

```ts
const handleDragEnd = (event) => {
  const { active, over } = event
  if (!over || active.id === over.id) return

  setCards((items) => {
    const oldIndex = items.indexOf(active.id)
    const newIndex = items.indexOf(over.id)
    return arrayMove(items, oldIndex, newIndex)
  })
}
```

📌 **DnD는 UI가 아니라 “상태 재배치” 문제입니다**

---

## 🧠 dnd-kit의 철학 (중요)

> ❝ Drag & Drop은 DOM 문제가 아니라
> **State Transition 문제다** ❞

| 항목     | dnd-kit |
| ------ | ------- |
| DOM 이동 | ❌       |
| 상태 기반  | ✅       |
| 선언적    | ✅       |
| 접근성    | 기본 제공   |

---

## ⚠️ 실무에서 자주 겪는 함정

### ❗ 1. id는 반드시 **stable** 해야 함

```ts
❌ index
✅ uuid / 고유 키
```

### ❗ 2. transform을 직접 style로 적용해야 함

### ❗ 3. CSS overflow + transform 충돌 주의

---

## 🎯 언제 dnd-kit을 써야 할까?

✅ Trello / Kanban
✅ 정렬 가능한 리스트
✅ 대시보드 위젯
✅ 커스텀 UI DnD
❌ 단순 HTML Drag만 필요한 경우

---

## 📌 마무리 요약

* dnd-kit은 **UI 없는 Drag 엔진**
* 핵심은 **Sensor + Collision + State**
* DOM을 옮기지 않고 **상태를 이동**
* React 철학에 가장 잘 맞는 DnD


