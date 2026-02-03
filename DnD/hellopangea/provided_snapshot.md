# 🎯 @hello-pangea/dnd : `provided` & `snapshot` 

---

## 0️⃣ 한 줄 요약

> **provided = 반드시 DOM에 붙여야 하는 “기계 부품”**
> **snapshot = 현재 드래그 상태를 알려주는 “상태 신호등”**

```
DnD 동작 여부  ← provided (필수)
UI 반응/스타일 ← snapshot (선택)
```

---

## 1️⃣ 전체 구조를 한 눈에 👀 (Big Picture)

![Image](https://user-images.githubusercontent.com/2182637/53607406-c8f3a780-3c12-11e9-979c-7f3b5bd1bfbd.gif)

![Image](https://cdn.sanity.io/images/3do82whm/next/51476ebe49fecd191631aafbb3ab437ebbc0aaad-3600x2000.png?auto=format\&fit=clip\&q=75\&w=3840)

![Image](https://repository-images.githubusercontent.com/196048036/5fae96d6-a1e5-43bc-8556-4ab9d83d4ff2)

```
DragDropContext
   ├─ Droppable
   │     ├─ provided  → DOM 연결 (필수)
   │     └─ snapshot  → isDraggingOver
   │
   └─ Draggable
         ├─ provided  → DOM 이동/애니메이션 (필수)
         └─ snapshot  → isDragging
```

👉 **중요 포인트**

* 라이브러리는 **DOM을 직접 제어해야** 합니다
* 그래서 `provided`가 필요합니다
* 하지만 “어떻게 보일지”는 개발자가 결정 → `snapshot`

---

## 2️⃣ `provided`란 무엇인가? 🔧

### 📌 정의

> **`provided`는 라이브러리가 DOM을 제어하기 위해
> 반드시 주입해야 하는 ref + props 묶음 객체**

즉,

* “이 DOM이 드래그 대상이다”
* “이 DOM이 드롭 영역이다”
* “이 DOM은 지금 이동 중이다”

라는 **기계적인 사실**을 알려주는 역할입니다.

---

## 3️⃣ Droppable의 `provided` 도식화 🧺

![Image](https://i.sstatic.net/CCNfb.png)

![Image](https://user-images.githubusercontent.com/51338185/114188114-750b2f00-9951-11eb-88b2-02f57c9a55f0.png)

### 🔹 Droppable provided 구성

```
provided = {
  innerRef        → 드롭 영역 DOM 참조
  droppableProps  → 드롭 판정 이벤트/속성
  placeholder     → 드래그 중 비워진 공간
}
```

### 🔹 시각적 사고

```
[ 카드 A ][ 카드 B ][ 카드 C ]
                ↑
          이 카드가 드래그됨

placeholder ↓

[ 카드 A ][  (빈 공간)  ][ 카드 C ]
```

👉 `placeholder`가 없으면?

* 카드 B가 빠진 순간
* 리스트 높이가 줄어들고
* **드래그 중 레이아웃이 덜컹거림**

---

### 🔹 Droppable 코드

```jsx
<Droppable droppableId="list">
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}           // 🔴 필수
      {...provided.droppableProps}      // 🔴 필수
      className={
        snapshot.isDraggingOver
          ? "bg-blue-50"
          : "bg-white"
      }
    >
      {children}

      {provided.placeholder}            // 🔴 필수
    </div>
  )}
</Droppable>
```

> ✅ **Droppable = 3종 세트**
>
> * `innerRef`
> * `droppableProps`
> * `placeholder`

---

## 4️⃣ Draggable의 `provided` 도식화 🧲

![Image](https://miro.medium.com/1%2A9lg87_E573b7molx8AKpmw.jpeg)

![Image](https://user-images.githubusercontent.com/2182637/53614150-efbed780-3c2c-11e9-9204-a5d2e746faca.gif)

### 🔹 Draggable provided 구성

```
provided = {
  innerRef
  draggableProps      → 위치, transform, transition 포함
  dragHandleProps     → “잡는 손잡이”
}
```

### 🔹 시각적 사고

```
┌────────────────────┐
│   카드 전체 영역   │  ← draggableProps
│   ┌──────────┐     │
│   │  ☰ 아이콘 │     │  ← dragHandleProps
│   └──────────┘     │
└────────────────────┘
```

---

### 🔹 Draggable 코드

```jsx
<Draggable draggableId={id} index={index}>
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}              // 🔴 필수
      {...provided.draggableProps}         // 🔴 필수
      {...provided.dragHandleProps}        // 🔴 필수
      style={{
        ...provided.draggableProps.style, // 🔴 절대 제거 금지
        boxShadow: snapshot.isDragging
          ? "0 8px 20px rgba(0,0,0,0.15)"
          : "none",
      }}
    >
      카드 내용
    </div>
  )}
</Draggable>
```

⚠️ **강조 포인트**

* `provided.draggableProps.style`을 덮어쓰면
  → **이동 애니메이션이 깨집니다**
* 반드시 **spread 후 커스터마이징**

---

## 5️⃣ `snapshot`이란 무엇인가? (상태 신호등 🚦)

### 📌 정의

> **`snapshot`은 “지금 드래그가 어떤 상태인가?”를 알려주는
> 읽기 전용 상태 객체**

즉,

* 동작에는 관여 ❌
* UI 반응에만 사용 ⭕

---

## 6️⃣ snapshot 주요 속성 (강의 표)

### 🔹 Droppable snapshot

| 속성               | 의미                      |
| ---------------- | ----------------------- |
| `isDraggingOver` | 드래그 중인 아이템이 이 영역 위에 있는가 |

```jsx
snapshot.isDraggingOver
  ? "드롭 가능 강조"
  : "기본 상태"
```

---

### 🔹 Draggable snapshot

| 속성           | 의미                |
| ------------ | ----------------- |
| `isDragging` | 지금 이 아이템이 드래그 중인가 |

```jsx
snapshot.isDragging
  ? "떠 있는 카드 효과"
  : "기본 카드"
```

---

## 7️⃣ provided vs snapshot — 결정적 비교 슬라이드 ⚖️

| 구분     | provided | snapshot   |
| ------ | -------- | ---------- |
| 목적     | DOM 제어   | 상태 표시      |
| 필수 여부  | ✅ 필수     | ⭕ 선택       |
| ref 포함 | ✅        | ❌          |
| 이벤트/좌표 | ✅        | ❌          |
| UI 스타일 | ❌        | ✅          |
| 없으면?   | ❌ 작동 안 함 | 😐 스타일만 없음 |

---

## 8️⃣ 핵심 멘트

> 🔹 **“provided는 동작을 위한 부품입니다.”**
> 🔹 **“snapshot은 UI를 위한 상태입니다.”**
> 🔹 **“provided 없으면 드래그 자체가 안 됩니다.”**
> 🔹 **“snapshot 없어도 드래그는 됩니다.”**

---

## 9️⃣ 가장 헷갈려하는 부분 정리 ❓

### Q1. snapshot으로 드래그를 제어하면 안 되나요?

➡️ **안 됩니다**
→ snapshot은 **읽기 전용**입니다

### Q2. provided를 일부만 써도 되나요?

➡️ **안 됩니다**
→ 특히 `innerRef`, `style`, `placeholder` 누락은 치명적입니다

### Q3. 왜 이렇게 복잡한가요?

➡️ 이유는 단 하나입니다

> **“DOM을 직접 조작하지 않고도
> React 방식(state 기반)으로
> 부드러운 드래그를 구현하기 위해서”**

---

## 🔚 마무리 요약 슬라이드

```
DnD = UI 이벤트 ❌
DnD = 상태 재정렬 ⭕
```

* `provided` → **기계**
* `snapshot` → **신호**
* `onDragEnd` → **정답**

