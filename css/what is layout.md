## 1️⃣ normal flow의 정확한 정의

> **Normal flow란,
> CSS에서 별도의 레이아웃 알고리즘(flex, grid, table, positioned layout 등)을 적용하지 않았을 때
> 요소들이 배치되는 기본적인 흐름이다.**

즉,

```text
“아무 특별한 레이아웃을 쓰지 않았을 때의 기본 배치 규칙”
```

---

## 2️⃣ normal flow를 구성하는 두 가지 축

normal flow는 사실 두 개의 formatting context로 나뉩니다.

### ① Block formatting (세로 흐름)

* block-level 요소
* 부모의 content box 기준
* 위 → 아래로 쌓임
* width: auto → 부모의 content width
* margin collapsing 발생

```html
<div></div>
<div></div>
<div></div>
```

---

### ② Inline formatting (가로 흐름)

* inline-level 요소
* line box 안에서 좌 → 우 배치
* 줄바꿈 발생 가능
* 글자처럼 취급됨

```html
<span>A</span><span>B</span><span>C</span>
```

---

## 3️⃣ normal flow에 **포함되지 않는 것들**

아래는 **normal flow에서 벗어납니다**:

| 기능                   | 이유                      |
| -------------------- | ----------------------- |
| `position: absolute` | 흐름에서 제거                 |
| `position: fixed`    | 흐름에서 제거                 |
| `float`              | normal flow에서 분리됨       |
| `display: flex`      | flex formatting context |
| `display: grid`      | grid formatting context |
| `display: table`     | table layout            |

📌 하지만:

* float는 **완전히 제거는 아님**
* 주변 텍스트 흐름에 영향

---

## 4️⃣ normal flow의 핵심 규칙 요약

| 항목         | 규칙                        |
| ---------- | ------------------------- |
| 배치 방향      | block: 위→아래 / inline: 좌→우 |
| 줄바꿈        | block: 항상 / inline: 필요 시  |
| width 기본값  | block: auto(부모 폭)         |
| height 기본값 | 콘텐츠 기반                    |
| margin     | block 간 병합 가능             |

---

## 5️⃣ flow / flow-root와의 관계

| display-inside | normal flow와의 관계     |
| -------------- | -------------------- |
| `flow`         | normal flow 그대로      |
| `flow-root`    | normal flow + BFC 생성 |

---

## 6️⃣ 한 문장 요약 (정확)

> **Normal flow란,
> block-level 요소는 위에서 아래로,
> inline-level 요소는 줄 안에서 좌에서 우로 배치되는
> CSS의 기본 문서 흐름이다.**


---

## 1️⃣ flow 타입의 공식적 의미 (정의)

**`flow`는 요소가 자신의 자식들을 *normal flow* 규칙에 따라 배치하는
가장 기본적인 inner display type이다.**

형식적으로 말하면:

> **flow layout은 자식들을
> block-level box 또는 inline-level box로 생성하여
> normal flow에 따라 배치한다.**

---

## 2️⃣ flow 타입이 생성하는 레이아웃 컨텍스트

`display-inside: flow` 는 다음을 의미합니다:

* **Block Formatting Context(BFC)를 새로 만들지 않음**
* 기존 normal flow에 참여
* float, margin collapsing 등의 영향을 받음

즉:

```text
flow = "기본 문서 흐름"
```

---

## 3️⃣ flow 타입에서 자식이 배치되는 방식

부모가 `flow`일 때 자식은:

| 자식의 outer display | 배치 방식                |
| ----------------- | -------------------- |
| `block`           | 세로로 쌓임 (줄바꿈 발생)      |
| `inline`          | 같은 줄에 배치             |
| `inline-block`    | inline-level box로 배치 |
| `inline-flex`     | inline-level box로 배치 |
| `inline-grid`     | inline-level box로 배치 |

📌 **핵심은 block vs inline 구분뿐**

---

## 4️⃣ flow 타입의 특징 요약

flow 타입은 다음을 포함합니다:

* normal flow
* line box 생성
* inline formatting
* block formatting
* margin collapsing (block 요소 간)
* float의 영향 받음

---

## 5️⃣ flow와 다른 inner 타입과의 대비

| inner type  | 차이점           |
| ----------- | ------------- |
| `flow`      | 기본 흐름, BFC 없음 |
| `flow-root` | flow + BFC 생성 |
| `flex`      | 1차원 배치 알고리즘   |
| `grid`      | 2차원 배치 알고리즘   |

---

## 6️⃣ 한 문장으로 정확히 정의

> **flow 타입이란,
> 자식들을 block-level 또는 inline-level box로 생성하여
> normal document flow 규칙에 따라 배치하는 기본 레이아웃 방식이다.**

---

## 7️⃣ 가장 중요한 포인트 (시험용 문장)

> **`display: block`은 outer = block, inner = flow 를 의미한다.**
> **flow는 “자식 배치 알고리즘”이지, block 자체를 뜻하지 않는다.**





## 질문: flow 레이아웃에 두 자식이 있고, 하나는 block, 하나는 inline으로 설정가능?

> **“flow 레이아웃에 두 자식이 있고,
> 하나는 block, 하나는 inline이다”**
> → **문법적으로도, 개념적으로도 유효한 설명입니다.**

---

## 왜 유효한가?

부모가:

```css
.parent {
  display: block;   /* outer: block, inner: flow */
}
```

이면:

* 부모는 **flow formatting context**를 만듭니다.
* 자식들은 **outer display 값**에 따라:

  * `block` → block-level box
  * `inline` → inline-level box

즉,

```html
<div class="parent">
  <div class="child-block"></div>
  <span class="child-inline"></span>
</div>
```

```css
.child-block  { display: block; }
.child-inline { display: inline; }
```

👉 완전히 정상적인 상황입니다.

---

## 다만 “조금 더 정확한” 표현은

스펙 관점에서는 이렇게 말합니다:

> **flow 레이아웃에서
> 한 자식은 block-level box로,
> 다른 자식은 inline-level box로 참여한다.**

이 표현이 가장 정확합니다.

---

## 왜 혼동이 생기기 쉬운가?

### 이유 1️⃣ `block` / `inline`이라는 단어

* **속성 값**이기도 하고
* **box 타입**을 가리키는 용어이기도 함

그래서 문장에 그대로 쓰면 모호해질 수 있음

---

## 중요한 구분 (다시 한 번)

| 표현                         | 정확성      |
| -------------------------- | -------- |
| 자식이 block이다                | ⚠️ 문맥 필요 |
| 자식이 block-level box로 참여한다  | ✅ 정확     |
| 자식이 inline이다               | ⚠️ 문맥 필요 |
| 자식이 inline-level box로 참여한다 | ✅ 정확     |

---

## 한 문장 요약

> **flow 레이아웃에서는
> 자식이 block 또는 inline으로 설정되어 있다는 가정은 전혀 틀리지 않으며,
> 이는 각각 block-level box / inline-level box로 참여함을 의미한다.**





**위 질문 상황에서는, 두 개의 “라인(line)”이 만들어집니다.**
다만, **그 이유를 정확히 구분해서 이해하는 것이 중요**합니다.

---

## 상황 재정의 (정확한 조건)

* 부모: `display: block` → inner = `flow`
* 자식 2개:

  1. 첫 번째 자식 → `display: block` → block-level box
  2. 두 번째 자식 → `display: inline` → inline-level box

```html
<div class="parent">
  <div class="block-child">BLOCK</div>
  <span class="inline-child">INLINE</span>
</div>
```

---

## 1️⃣ 실제로 만들어지는 “라인”의 정체

### 🔹 block-level box

* **자기 전용 줄을 강제로 생성**
* 위·아래에 자동 줄바꿈
* 결과적으로 **하나의 독립된 line/row를 차지**

📌 이건 *inline formatting context의 line box* 와는 다릅니다.

---

### 🔹 inline-level box

* **line box 안에 배치됨**
* 부모의 flow 컨텍스트에서

  * 새로운 **line box**가 생성됨

---

## 2️⃣ 결과적으로 무슨 일이 벌어지나?

순서가 이렇습니다:

1. block-level box 배치
   → 부모 흐름에서 **한 줄(블록 줄)** 차지
2. 그 다음에 inline-level box 등장
   → **새로운 line box 생성**
   → 그 안에 inline 요소 배치

👉 **총 2개의 “줄”이 생김**

---

## 3️⃣ 중요한 용어 구분 (아주 중요)

| 용어       | 의미                          |
| -------- | --------------------------- |
| block 줄  | block-level box가 차지하는 세로 영역 |
| line box | inline 요소들이 들어가는 줄          |

📌 block 요소는 **line box 안에 들어가지 않습니다.**

---

## 4️⃣ 그림으로 보면

```
[ BLOCK child ]   ← block-level box (독립 줄)

[ INLINE child ]  ← line box (inline formatting)
```

---

## 5️⃣ 핵심 정리 (시험용 문장)

> **flow 레이아웃에서 block-level box는 독립된 줄을 차지하고,
> inline-level box는 line box 안에 배치되므로
> 결과적으로 두 개의 줄이 생성된다.**

---

## 6️⃣ 한 문장 요약

> **네, 이 경우 두 개의 줄이 만들어지며,
> 하나는 block-level box의 줄,
> 다른 하나는 inline formatting을 위한 line box이다.**




| 구분 | 의미 |
|---|---|
| 부모 레이아웃에 어떻게 배치될 것인지 | display-outside |
| 나의 아이템들이 어떻게 배치될 것인지 | display-inside |

| display 구성 | display-outside | display-inside |
|---|---|---|
| 예시 1 | block | flex |
| 예시 2 | inline | grid |



