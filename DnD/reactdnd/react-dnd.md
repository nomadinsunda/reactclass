# React DnD 완전 정복: “드래그앤드롭을 ‘프레임워크’로 설계하는” 라이브러리 🧩🖱️

**react-dnd**는 “드래그 가능한 것(Drag Source)”과 “드롭 가능한 곳(Drop Target)”을 **완전히 분리**해서, 복잡한 DnD UX(칸반, 트리, 빌더, 에디터 등)를 **구성요소처럼 조립**하게 해주는 라이브러리입니다. 공식 문서도 “복잡한 drag & drop 인터페이스를 만들되, 컴포넌트를 decoupled(결합도 낮게) 유지”하는 것을 목표로 합니다. ([react-dnd.github.io][1])

> ✅ 핵심 느낌:
> **“정해진 UI(리스트 정렬)만 제공”**하는 게 아니라,
> **“DnD 상태 머신 + 이벤트 라우팅 + 백엔드 추상화”**를 제공해서 여러분이 UI를 설계하도록 돕습니다. 🧠⚙️

---

## 1) react-dnd가 ‘어려워 보이는’ 진짜 이유 😵‍💫➡️😎

react-dnd는 사용자를 대신해 “정렬 알고리즘”을 다 해주는 스타일이 아니라, 아래 3가지를 여러분이 **명확히 정의**하게 만듭니다.

1. **무엇을 드래그하는가?** (item: 드래그 데이터)
2. **어디에 드롭할 수 있는가?** (accept: 허용 타입)
3. **드래그 중/드롭 시 어떤 규칙으로 상태를 바꿀 것인가?** (hover/drop/end 로직)

그래서 처음엔 “코드가 많고 추상적”으로 느껴지지만, 한 번 모델이 잡히면 **DnD를 원하는 형태로 확장**하기가 정말 강합니다. 💪

---

## 2) 전체 구조 한 장으로 보기 🗺️

react-dnd는 크게 이렇게 돌아갑니다:

* **`<DndProvider backend={...}>`**: DnD “런타임”을 앱에 주입
* **Backend**: 브라우저의 실제 드래그 이벤트를 사용 (HTML5 / Touch / Test 등) ([react-dnd.github.io][2])
* **Hooks (`useDrag`, `useDrop`)**: 컴포넌트를 “드래그/드롭 유닛”으로 등록
* **Monitor**: 지금 드래그 상태가 어떤지(드래그 중인가? 어떤 아이템인가? 어디 위에 있나?)를 조회 ([react-dnd.github.io][3])

---

## 3) 설치 & 기본 세팅 🧰

HTML5 기반이 기본 백엔드입니다. ([react-dnd.github.io][2])

```bash
npm i react-dnd react-dnd-html5-backend
```

```jsx
// main.jsx (또는 App.jsx 상단)
import { DndProvider } from 'react-dnd'
import { HTML5Backend } from 'react-dnd-html5-backend'

export default function App() {
  return (
    <DndProvider backend={HTML5Backend}>
      {/* DnD가 필요한 모든 UI */}
      <Board />
    </DndProvider>
  )
}
```

> 참고: react-dnd는 GitHub 기준 **v16.0.0**(2022-04-05)이 최신 릴리즈로 표시됩니다. ([GitHub][4])
> (실무에서는 안정적으로 쓰이지만, “최신 React 기능에 맞춘 활발한 릴리즈” 스타일은 아니라는 점도 판단 포인트입니다.)

---

## 4) 가장 중요한 개념 4개 🔥 (여기서 감이 잡힙니다)

### (1) **Type(타입)**: “이 드래그는 어떤 종류냐?” 🏷️

react-dnd는 드래그를 문자열 타입으로 구분합니다.

```js
const ItemTypes = {
  CARD: 'CARD',
  LIST: 'LIST',
}
```

드롭 타겟은 `accept: ItemTypes.CARD`처럼 **받을 수 있는 타입**을 선언합니다.

---

### (2) **Item(드래그 데이터)**: “드래그 중에 들고 다니는 payload” 🎒

`useDrag`의 `item`은 드래그 동안 전달되는 데이터입니다.

```js
item: { id, listId, index }
```

이게 있어야 “어느 카드가 어디서 왔는지”를 알 수 있어요.

---

### (3) **Collect/Monitor**: “드래그 상태를 구독해서 UI를 바꾼다” 👀

Monitor는 드래그 상태를 조회합니다. 예: `isDragging`, `canDrop`, `isOver` 등 ([react-dnd.github.io][3])

```jsx
const [{ isDragging }, dragRef] = useDrag(() => ({
  type: ItemTypes.CARD,
  item: { id, index, listId },
  collect: (monitor) => ({
    isDragging: monitor.isDragging(),
  }),
}))
```

* `collect`는 **“DnD 상태 → 리렌더링할 props로 변환”**하는 함수라고 보시면 됩니다. 🎛️

---

### (4) **hover / drop**: “룰이 들어가는 지점” 🧠

* `hover`: 드래그 중에 대상 위를 지나갈 때 계속 호출 (정렬/미리보기/스왑)
* `drop`: 실제 드롭됐을 때 호출 (최종 확정)

---

## 5) 실전 예제: 칸반 카드 “리스트 내 재정렬 + 리스트 간 이동” 🧱➡️🧱

아래 예제는 react-dnd의 전형적인 패턴(특히 **Sortable**)을 담았습니다.

### ✅ 데이터 형태

```js
const initial = {
  lists: {
    todo: { id: 'todo', title: 'TODO', cardIds: ['c1', 'c2'] },
    doing: { id: 'doing', title: 'DOING', cardIds: ['c3'] },
  },
  cards: {
    c1: { id: 'c1', title: 'Write spec' },
    c2: { id: 'c2', title: 'Design UI' },
    c3: { id: 'c3', title: 'Implement' },
  },
}
```

---

### ✅ Board: 상태 업데이트 함수들

```jsx
import { useCallback, useState } from 'react'
import List from './List'

export default function Board() {
  const [state, setState] = useState(initial)

  const moveCard = useCallback(({ fromListId, toListId, fromIndex, toIndex, cardId }) => {
    setState(prev => {
      const next = structuredClone(prev)

      // 1) from에서 제거
      next.lists[fromListId].cardIds.splice(fromIndex, 1)

      // 2) to에 삽입
      next.lists[toListId].cardIds.splice(toIndex, 0, cardId)

      return next
    })
  }, [])

  return (
    <div style={{ display: 'flex', gap: 16 }}>
      {Object.values(state.lists).map(list => (
        <List
          key={list.id}
          list={list}
          cards={list.cardIds.map(id => state.cards[id])}
          moveCard={moveCard}
        />
      ))}
    </div>
  )
}
```

---

### ✅ List: 드롭 “컨테이너” (카드가 떨어질 수 있는 영역)

```jsx
import { useDrop } from 'react-dnd'
import Card from './Card'

const ItemTypes = { CARD: 'CARD' }

export default function List({ list, cards, moveCard }) {
  const [{ isOver, canDrop }, dropRef] = useDrop(() => ({
    accept: ItemTypes.CARD,
    collect: (monitor) => ({
      isOver: monitor.isOver({ shallow: true }),
      canDrop: monitor.canDrop(),
    }),
  }))

  return (
    <div
      ref={dropRef}
      style={{
        width: 280,
        padding: 12,
        borderRadius: 12,
        background: '#f5f5f5',
        outline: isOver && canDrop ? '2px solid #333' : 'none',
      }}
    >
      <h3>{list.title}</h3>

      <div style={{ display: 'flex', flexDirection: 'column', gap: 8 }}>
        {cards.map((card, index) => (
          <Card
            key={card.id}
            card={card}
            index={index}
            listId={list.id}
            moveCard={moveCard}
          />
        ))}
      </div>
    </div>
  )
}
```

---

### ✅ Card: `useDrag + useDrop`을 “같은 DOM”에 합치기 (가장 핵심 패턴) 🎯

```jsx
import { useRef } from 'react'
import { useDrag, useDrop } from 'react-dnd'

const ItemTypes = { CARD: 'CARD' }

export default function Card({ card, index, listId, moveCard }) {
  const ref = useRef(null)

  // 1) Drop: 다른 카드가 이 카드 위로 hover 될 때, 순서를 바꾼다
  const [, dropRef] = useDrop(() => ({
    accept: ItemTypes.CARD,
    hover: (dragItem, monitor) => {
      if (!ref.current) return

      const fromListId = dragItem.listId
      const toListId = listId

      const fromIndex = dragItem.index
      const toIndex = index

      // ✅ 같은 자리면 아무 것도 안 함
      if (fromListId === toListId && fromIndex === toIndex) return

      // (선택) 마우스 위치 기반 “반쯤 넘어오면 교체” 같은 정교한 룰도 여기서 구현합니다.

      moveCard({
        fromListId,
        toListId,
        fromIndex,
        toIndex,
        cardId: dragItem.id,
      })

      // ✅ 중요: dragItem의 index/listId를 업데이트해서
      // hover가 반복될 때 “이전 위치 기준”으로 계속 꼬이지 않게 함
      dragItem.index = toIndex
      dragItem.listId = toListId
    },
  }))

  // 2) Drag: 내가 드래그 가능한 카드임을 선언
  const [{ isDragging }, dragRef] = useDrag(() => ({
    type: ItemTypes.CARD,
    item: { id: card.id, index, listId },
    collect: (monitor) => ({
      isDragging: monitor.isDragging(),
    }),
  }))

  // 3) dragRef와 dropRef를 같은 DOM에 합성
  dragRef(dropRef(ref))

  return (
    <div
      ref={ref}
      style={{
        padding: 12,
        borderRadius: 10,
        background: 'white',
        cursor: 'grab',
        opacity: isDragging ? 0.4 : 1,
        boxShadow: '0 1px 4px rgba(0,0,0,0.12)',
      }}
    >
      {card.title}
    </div>
  )
}
```

### 여기서 제일 중요한 포인트 ✅

* `dragRef(dropRef(ref))` 이 한 줄이 **“이 DOM은 드래그도 되고, 드롭 타겟도 된다”**를 의미합니다.
* `hover`에서 `moveCard`를 호출하면 **드래그 중에 실시간으로 정렬이 바뀌는 UX**가 구현됩니다. 🧲

---

## 6) Drop 결과를 Drag 쪽에서 받기 🎁 (dropResult)

드롭 타겟이 `drop()`에서 객체를 반환하면, 드래그 소스의 `end()`에서 `monitor.getDropResult()`로 받을 수 있습니다. ([Stack Overflow][5])

```jsx
const [{}, dragRef] = useDrag(() => ({
  type: ItemTypes.CARD,
  item: { id: card.id },
  end: (item, monitor) => {
    const result = monitor.getDropResult()
    if (result) {
      console.log('Dropped into:', result.listId)
    }
  },
}))
```

```jsx
const [, dropRef] = useDrop(() => ({
  accept: ItemTypes.CARD,
  drop: () => ({ listId }), // ✅ 이 값이 drag end에서 getDropResult로 옴
}))
```

---

## 7) 모바일/터치까지 고려하면? 📱🤏 (백엔드 이야기)

react-dnd는 백엔드를 바꿀 수 있습니다. 문서에도 HTML5 / Touch / Test backend가 분리되어 있습니다. ([react-dnd.github.io][2])
실무에서 “데스크톱은 HTML5, 모바일은 Touch” 같이 섞고 싶으면 **multi-backend** 계열을 붙이기도 합니다. (예: `react-dnd-multi-backend`) ([npm][6])

> 다만 터치/HTML5 동시 지원은 케이스별 이슈가 있어, 프로젝트 환경(스크롤/제스처/브라우저)에 따라 튜닝이 필요합니다. ([GitHub][7])

---

## 8) react-dnd를 “잘 쓰는” 사람들의 공통 팁 🧠✨

### ✅ 1) 드래그 아이템(item)을 **최소 데이터**로 유지하세요

* `{ id, index, listId }` 정도면 대부분 충분합니다.
* 객체를 크게 만들면 hover가 자주 불릴 때 부담이 커집니다. 🏋️

### ✅ 2) hover에서 상태 업데이트는 “필요할 때만”

* 같은 인덱스면 return
* (가능하면) 마우스가 카드의 절반을 넘었을 때만 reorder 같은 룰 추가

### ✅ 3) UI 피드백은 `collect(monitor)`에서

* `isDragging`, `isOver`, `canDrop` 같은 값을 UI에 바로 반영하면 UX가 좋아집니다. ([react-dnd.github.io][3])

---

## 9) react-dnd는 언제 “최적”이고, 언제 “과한”가요? ⚖️

### 👍 react-dnd가 특히 강한 상황

* 노션/피그마/폼 빌더 같은 **커스텀 상호작용**
* 드래그 중 **복잡한 규칙**(허용 조건, 미리보기, 다중 타겟, 중첩 구조)
* “정렬”을 넘어서는 **DnD 기반 편집기**

### 👀 더 단순한 대안이 나을 때

* “리스트 정렬” 중심이면 **@hello-pangea/dnd** 같은 고수준 라이브러리가 더 빠를 수 있음
* 센서/충돌감지/정렬이 주 목표면 **dnd-kit**이 더 현대적인 DX를 주는 경우도 많음

---


[1]: https://react-dnd.github.io/react-dnd/?utm_source=chatgpt.com "React DnD"
[2]: https://react-dnd.github.io/react-dnd/docs/backends/html5/?utm_source=chatgpt.com "React DnD HTML5 Backend"
[3]: https://react-dnd.github.io/react-dnd/docs/api/drag-source-monitor/?utm_source=chatgpt.com "DragSourceMonitor"
[4]: https://github.com/react-dnd/react-dnd?utm_source=chatgpt.com "react-dnd/react-dnd: Drag and Drop for React"
[5]: https://stackoverflow.com/questions/72996625/react-dnd-typescript-how-to-use-getdropresult?utm_source=chatgpt.com "React DnD / TypeScript: How to use getDropResult?"
[6]: https://www.npmjs.com/react-dnd-multi-backend?utm_source=chatgpt.com "react-dnd-multi-backend"
[7]: https://github.com/react-dnd/react-dnd/issues/3483?utm_source=chatgpt.com "Simultaneous html5 and touch backend · Issue #3483"
