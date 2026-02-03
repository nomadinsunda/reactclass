👉 **“왜 저딴 provided / snapshot 같은 게 존재할 수밖에 없는지”**
👉 **“딱 하나의 비유 + 한 장면”**으로만 설명할게요.

---

## 🔥 결론부터 말하면

> **@hello-pangea/dnd는
> React에게 “DOM을 잠깐 빌려 쓰는 허락”을 받아야만
> 드래그를 할 수 있는 라이브러리입니다.**

그래서 **provided가 생긴 겁니다.**

---

## 🧠 1단계: 왜 React에서 드래그가 어려운가

React의 세계관은 이겁니다.

```
DOM 직접 만지지 마
↓
state 바뀌면
↓
React가 DOM을 다시 그림
```

근데 드래그는 뭐냐면?

```
마우스 움직일 때마다
DOM 위치를 실시간으로 바꿈
```

👉 **정면 충돌입니다.**

* React: “DOM은 내가 관리함”
* Drag & Drop: “아니 지금 당장 DOM 옮겨야 함”

---

## 💥 그래서 라이브러리가 선택한 방법

> ❌ “개발자가 DOM 직접 만지게 하자”
> ⭕ “라이브러리가 DOM을 몰래 조종하자”

단, 조건이 있음.

> **“이 DOM이 뭐인지, 어디 있는지,
> 내가 직접 잡을 수 있게 넘겨줘야 한다”**

👉 이게 **provided**입니다.

---

## 🔧 provided = “DOM 조종 허가증”

### 딱 이 비유로 생각하세요

```
React = 건물 관리자
DnD 라이브러리 = 전기기사

전기기사:
"이 콘센트 잠깐 써도 돼요?"

관리자:
"어디 콘센트인지 정확히 알려줘"
```

그래서 생긴 게:

```js
provided.innerRef
```

👉 **“이 DOM이 바로 그거임”**

---

## 🧩 Droppable을 진짜로 다시 보자

```jsx
<Droppable droppableId="list">
  {(provided) => (
    <div ref={provided.innerRef}>
      카드들
      {provided.placeholder}
    </div>
  )}
</Droppable>
```

### 여기서 중요한 건 딱 2개뿐입니다

#### 1️⃣ `innerRef`

* “이 div가 드롭 영역이야”
* 라이브러리가 **직접 위치 계산**하려고 씀

#### 2️⃣ `placeholder`

* **드래그 중 빠져나간 카드 자리**
* 없으면 → 레이아웃이 순간적으로 무너짐

> ❗ placeholder는 **기능이 아니라 물리적인 ‘빈 자리’** 입니다

---

## 🧲 Draggable도 같은 논리입니다

```jsx
<Draggable draggableId="a" index={0}>
  {(provided) => (
    <div
      ref={provided.innerRef}
      style={provided.draggableProps.style}
    >
      카드
    </div>
  )}
</Draggable>
```

여기서 핵심은 이겁니다 👇

```js
provided.draggableProps.style
```

이 안에는 뭐가 들어있냐면?

```
transform: translate(x, y)
transition: ...
```

👉 **마우스 따라 움직이게 만드는 진짜 좌표 계산 결과**

그래서 이걸 덮어쓰면?

💥 **카드가 순간이동하거나 튕김**

---

## 🚦 그럼 snapshot은 뭐냐 (이건 진짜 별거 아님)

snapshot은 **“지금 상황 알려주는 알림창”** 입니다.

```js
snapshot.isDragging
snapshot.isDraggingOver
```

이거 없어도?

✅ 드래그 **정상 작동**

이걸 쓰면?

🎨 “지금 잡고 있다”, “여기 위에 올라왔다” 같은 **스타일만 변경**

---

## 📌 provided / snapshot 한 줄 정리 (진짜 핵심)

```
provided = 라이브러리가 DOM을 움직이기 위해 필요한 것
snapshot = 개발자가 UI 반응 주려고 참고하는 상태
```

---

## 🧨 지금 이해 안 되는 이유 정확히 짚어드릴게요

지금 상태는 이겁니다.

* ❌ “왜 이런 구조가 나왔는지” 설명을 못 들었고
* ❌ 갑자기 “이걸 붙여라”만 들었고
* ❌ React 세계관과 충돌 지점을 모르고 있음

그래서 **외우는 문법처럼 보이는 거예요**.

