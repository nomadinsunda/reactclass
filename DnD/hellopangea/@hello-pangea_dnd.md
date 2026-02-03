## 0) @hello-pangea/dnd 🧭

* 원래 유명했던 **react-beautiful-dnd**(Atlassian)가 사실상 유지보수가 중단/Deprecated 흐름을 타면서, 커뮤니티가 유지보수하는 “사실상 후계자” 성격의 라이브러리가 **@hello-pangea/dnd**입니다. ([GitHub][1])
* 컨셉은 **“리스트(수직/수평/리스트 간 이동) 기반 DnD에 최적화된 고수준 추상화”** 입니다. 즉, 범용 저수준(react-dnd)보다 **칸반/정렬/이동**에 특화되어 있어요. ([GitHub][2])
* TypeScript 기반이며 타입 정의가 번들로 포함되어 바로 사용할 수 있습니다. ([GitHub][3])
* 사용 예제/스토리북/샌드박스가 비교적 잘 갖춰져 있습니다. ([GitHub][4])

---

## 1) 이 라이브러리의 “정신적 모델” (중요) 🧠

**핵심은 한 줄입니다.**

> ✅ “드래그 앤 드롭은 DOM을 옮기는 게 아니라, **state(배열/트리)를 재정렬**하는 일이다.”

그래서 @hello-pangea/dnd는,

* 드래그 중에는 **시각적 위치/애니메이션을 라이브러리가 처리**하고
* 드래그가 끝나는 순간(`onDragEnd`)에 **state를 업데이트**해서
* 리렌더링 결과로 “진짜 위치”가 확정되게 만듭니다.

---

## 2) 3대 컴포넌트 구조 🔩

### (1) `DragDropContext` — “DnD 세션의 루트” 🌐

* DnD 전체 이벤트를 받고, 드래그 종료 시점에 **단 한 번** state를 갱신하는 곳입니다.

```jsx
import { DragDropContext } from "@hello-pangea/dnd";

<DragDropContext
  onDragEnd={handleDragEnd}
  onDragStart={handleDragStart} // 선택
  onDragUpdate={handleDragUpdate} // 선택
>
  ...
</DragDropContext>
```

### (2) `Droppable` — “드롭 가능한 영역” 🧺

* 리스트(컬럼) 같은 컨테이너.
* `droppableId`는 **고유해야** 하고, 드롭 영역의 DOM에 **필수 props**를 붙여야 합니다.

### (3) `Draggable` — “끌 수 있는 아이템” 🧲

* 카드/행(row) 같은 요소.
* `draggableId`는 **고유해야** 하고, `index`는 **리스트 내 순서**입니다.

---

## 3) `provided`와 `snapshot`을 ‘정확히’ 이해하기 🧩

이 부분이 @hello-pangea/dnd의 “문법”입니다. 익숙해지면 생산성이 확 올라갑니다.

---

### 3-1) `provided` = “DOM에 반드시 붙여야 하는 접착제(필수)” 🧷

`provided`는 라이브러리가 드래그를 가능하게 하려고 DOM에 주입해야 하는 것들을 모아둔 객체입니다.

#### ✅ Droppable의 provided

```jsx
<Droppable droppableId="todo">
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.droppableProps}
    >
      ...
      {provided.placeholder}
    </div>
  )}
</Droppable>
```

* `provided.innerRef`

  * 라이브러리가 **드롭 영역 DOM을 직접 참조**해야 해서 필요합니다.
* `provided.droppableProps`

  * 드래그 오버/드롭 판정에 필요한 이벤트/속성들이 들어있습니다.
* `provided.placeholder` (**매우 중요**)

  * 드래그 중 원래 자리의 “공간”을 유지하는 가짜 요소입니다.
  * 이게 없으면 **리스트가 드래그 중에 찌그러지거나 점프**하는 느낌이 생깁니다. (칸이 사라지니까요)

> 💡 결론: **Droppable은 `innerRef`, `droppableProps`, `placeholder`가 세트**입니다.

---

#### ✅ Draggable의 provided

```jsx
<Draggable draggableId={card.id} index={index}>
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.draggableProps}
      {...provided.dragHandleProps}
      style={{
        ...provided.draggableProps.style,
        // 필요하면 사용자 스타일 추가
      }}
    >
      ...
    </div>
  )}
</Draggable>
```

* `provided.innerRef` : 드래그 대상 DOM 참조
* `provided.draggableProps` : 드래그 이동/위치 계산/애니메이션 관련 props(여기에 style도 포함되는 경우가 많음) ([Velog][5])
* `provided.dragHandleProps` : “이 부분을 잡고 끌 수 있음”을 정의

  * 보통은 전체를 핸들로 두지만, “아이콘만 핸들”로도 가능

> ⚠️ 실무에서 가장 흔한 실수
>
> * `style={{ ...provided.draggableProps.style }}`를 **빼먹거나 덮어써서** 드래그 애니메이션이 깨짐
> * `dragHandleProps`를 **원하는 요소에만** 붙이지 않고 어중간하게 붙여 UX가 나빠짐

---

### 3-2) `snapshot` = “현재 드래그 상태를 알려주는 UI 힌트(선택)” 🎛️

`snapshot`은 “지금 드래그 중인가?”, “드롭 영역 위에 올라왔나?” 같은 상태를 줍니다.

대표적으로 이런 플래그를 UI에 씁니다:

* Droppable snapshot

  * `snapshot.isDraggingOver` : 드래그 대상이 현재 이 영역 위에 올라와 있는지
* Draggable snapshot

  * `snapshot.isDragging` : 지금 이 아이템이 드래그 중인지

예:

```jsx
const bg = snapshot.isDraggingOver ? "bg-blue-50" : "bg-white";
```

> ✅ `provided`는 필수(동작), `snapshot`은 선택(UI/스타일링)

---

## 4) `onDragEnd(result)` — “정답은 여기서 결정” 🧾

드래그가 끝나면 `handleDragEnd`가 호출되고, 여기서 **배열을 재배치**해야 합니다.

일반적으로 `result`는 이런 정보를 줍니다(핵심만):

* `result.source`: 어디서 시작했는지 `{ droppableId, index }`
* `result.destination`: 어디에 떨어졌는지 `{ droppableId, index }` (없을 수도!)
* `result.draggableId`: 어떤 아이템인지

> destination이 `null`이면 “유효한 드롭 영역 밖에 떨어뜨림”입니다. 그래서 보통 이렇게 가드합니다. ([Velog][6])

---

## 5) 실전 예제: “칸반(리스트 간 이동 + 리스트 내부 정렬)” 🧱✨

아래 코드는 “심플하지 않은” 실전형 구조입니다:

* **컬럼 2개 이상**
* **컬럼 내부 정렬 + 컬럼 간 이동**
* state 정규화까지는 아니지만 실무에서 바로 쓰는 형태

```jsx
import React, { useCallback, useMemo, useState } from "react";
import { DragDropContext, Droppable, Draggable } from "@hello-pangea/dnd";

// 재정렬 유틸
const reorder = (list, startIndex, endIndex) => {
  const result = Array.from(list);
  const [removed] = result.splice(startIndex, 1);
  result.splice(endIndex, 0, removed);
  return result;
};

// 리스트 간 이동 유틸
const move = (source, destination, droppableSource, droppableDestination) => {
  const sourceClone = Array.from(source);
  const destClone = Array.from(destination);

  const [removed] = sourceClone.splice(droppableSource.index, 1);
  destClone.splice(droppableDestination.index, 0, removed);

  return {
    [droppableSource.droppableId]: sourceClone,
    [droppableDestination.droppableId]: destClone,
  };
};

export default function KanbanDnd() {
  const [columns, setColumns] = useState({
    todo: [
      { id: "c1", title: "API 설계" },
      { id: "c2", title: "ERD 확정" },
      { id: "c3", title: "UI 와이어프레임" },
    ],
    doing: [
      { id: "c4", title: "DragDrop 적용" },
      { id: "c5", title: "성능 최적화" },
    ],
    done: [{ id: "c6", title: "프로젝트 셋업" }],
  });

  const columnOrder = useMemo(() => ["todo", "doing", "done"], []);

  const onDragEnd = useCallback((result) => {
    const { source, destination } = result;

    // 1) 유효한 드롭이 아니면 종료
    if (!destination) return;

    // 2) 같은 위치면 종료
    if (
      source.droppableId === destination.droppableId &&
      source.index === destination.index
    ) {
      return;
    }

    setColumns((prev) => {
      const srcList = prev[source.droppableId];
      const destList = prev[destination.droppableId];

      // 3) 같은 리스트 내부 정렬
      if (source.droppableId === destination.droppableId) {
        const reordered = reorder(srcList, source.index, destination.index);
        return { ...prev, [source.droppableId]: reordered };
      }

      // 4) 리스트 간 이동
      const moved = move(srcList, destList, source, destination);
      return { ...prev, ...moved };
    });
  }, []);

  return (
    <DragDropContext onDragEnd={onDragEnd}>
      <div style={{ display: "flex", gap: 16, alignItems: "flex-start" }}>
        {columnOrder.map((colId) => (
          <Droppable key={colId} droppableId={colId}>
            {(provided, snapshot) => (
              <div
                ref={provided.innerRef}
                {...provided.droppableProps}
                style={{
                  width: 280,
                  padding: 12,
                  borderRadius: 12,
                  border: "1px solid #ddd",
                  background: snapshot.isDraggingOver ? "#f0f8ff" : "#fafafa",
                  minHeight: 120,
                }}
              >
                <h3 style={{ margin: "0 0 12px 0" }}>{colId.toUpperCase()}</h3>

                {columns[colId].map((card, index) => (
                  <Draggable key={card.id} draggableId={card.id} index={index}>
                    {(provided, snapshot) => (
                      <div
                        ref={provided.innerRef}
                        {...provided.draggableProps}
                        // ✅ 핸들을 “제목줄”로 제한하고 싶으면 여기 대신 내부 요소에 dragHandleProps를 붙이세요
                        {...provided.dragHandleProps}
                        style={{
                          userSelect: "none",
                          padding: 12,
                          marginBottom: 10,
                          borderRadius: 10,
                          border: "1px solid #e5e5e5",
                          background: snapshot.isDragging ? "#fff" : "#ffffff",
                          boxShadow: snapshot.isDragging
                            ? "0 8px 24px rgba(0,0,0,0.12)"
                            : "none",
                          ...provided.draggableProps.style, // ✅ 이거 중요!
                        }}
                      >
                        <div style={{ fontWeight: 700 }}>{card.title}</div>
                        <div style={{ fontSize: 12, opacity: 0.7, marginTop: 6 }}>
                          id: {card.id}
                        </div>
                      </div>
                    )}
                  </Draggable>
                ))}

                {/* ✅ placeholder를 반드시 렌더링 */}
                {provided.placeholder}
              </div>
            )}
          </Droppable>
        ))}
      </div>
    </DragDropContext>
  );
}
```

---

## 6) 실무에서 자주 터지는 함정들 🔥 (여기서 많이 막힙니다)

### ✅ 1) `placeholder` 누락 → “드래그 중 레이아웃 점프”

* 드래그한 아이템이 빠지면 그 공간이 없어져서 리스트가 줄어듭니다.
* `provided.placeholder`가 그 공간을 유지해주는 핵심입니다.

### ✅ 2) `provided.draggableProps.style` 덮어쓰기 → “애니메이션/좌표 깨짐”

* 라이브러리가 계산한 transform/transition이 style에 들어옵니다.
* 커스텀 style을 쓰더라도 **마지막에 합쳐서 유지**해야 합니다. ([Velog][5])

### ✅ 3) `draggableId` / `droppableId`가 “렌더마다 바뀜”

* id는 **절대 안정적인 값**이어야 합니다.
* index를 id로 쓰면(특히 reorder에서) 지옥이 열릴 수 있습니다.

### ✅ 4) StrictMode / Portal / Modal 이슈

* 모달/포탈에서 드래그 위치가 튀는 케이스가 종종 있고, 포지셔닝/스크롤 컨테이너 구조에 따라 보정이 필요할 수 있습니다. (실무 트러블슈팅 글들도 꽤 있습니다.) ([HOCHI's blog][7])

---

## 7) 접근성(A11y) 관점에서 왜 “괜찮은 선택”인가요? ♿️

@hello-pangea/dnd는 애초에 “beautiful and accessible”을 표방하며 리스트형 DnD에 맞춘 패턴을 제공합니다. ([GitHub][2])
다만, 접근성은 “라이브러리만으로 끝”이 아니라 다음도 같이 챙기셔야 완성됩니다:

* 드래그 핸들에 대한 명확한 UI(아이콘/커서/설명)
* 키보드 사용자를 위한 포커스 스타일
* 드롭 가능 영역의 시각적 피드백(`snapshot.isDraggingOver`)

---

## 8) 언제 @hello-pangea/dnd를 쓰고, 언제 dnd-kit을 쓰나요? ⚖️

* ✅ **@hello-pangea/dnd**

  * “react-beautiful-dnd 스타일”의 **리스트 기반 칸반/정렬**을 빠르게 만들고 싶을 때
  * 문법이 정해져 있어서 팀 표준화가 쉬움
* ✅ **dnd-kit**

  * 더 현대적인 센서/충돌 감지/커스터마이징, 고급 성능 최적화가 필요할 때




[1]: https://github.com/atlassian/react-beautiful-dnd/issues/2672?utm_source=chatgpt.com "react-beautiful-dnd is now deprecated · Issue #2672"
[2]: https://github.com/hello-pangea/dnd?utm_source=chatgpt.com "hello-pangea/dnd: 💅 Beautiful and accessible drag ..."
[3]: https://github.com/hello-pangea/dnd/blob/main/docs/guides/types.md?utm_source=chatgpt.com "dnd/docs/guides/types.md at main · hello-pangea/dnd"
[4]: https://github.com/hello-pangea/dnd/blob/main/docs/about/examples.md?utm_source=chatgpt.com "dnd/docs/about/examples.md at main · hello-pangea/dnd"
[5]: https://velog.io/%40mikio/hello-pangeadnd?utm_source=chatgpt.com "@hello-pangea/dnd로 드래그 앤 드롭 구현하기 (feat. ..."
[6]: https://velog.io/%40ldlldl/hello-pangea-dnd?utm_source=chatgpt.com "[nextJS] @hello-pangea/dnd를 사용하여 Drag and Drop ..."
[7]: https://hochi-dev.tistory.com/20?utm_source=chatgpt.com "Modal에서 Drag & Drop 사용시 위치가 튕기는 현상 해결 ..."
