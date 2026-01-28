CSS Flexbox는 **“한 줄 또는 여러 줄에 아이템들을 자동으로 정렬·나눠 담아주는 1차원 레이아웃 시스템”**입니다.
그런데 문서들을 보면 항상 **메인 축(main axis)**, **크로스 축(cross axis)**, **정렬** 얘기만 잔뜩 나와서 더 헷갈리죠. 😵‍💫

이번 글에서는:

* 왜 Flexbox가 생겼는지
* “축 2개”가 **정확히** 어떻게 동작하는지
* 브라우저가 flex 레이아웃을 계산하는 **실제 알고리즘 순서**
* 자주 헷갈리는 `justify-content` vs `align-items` vs `align-content`
* `flex: 1`, `flex-basis: 0` 같은 실전 속 개념들

까지 정리해 보겠습니다.

---

## 1. Flexbox가 왜 생겼나? (문제 정의) 🧩

옛날 CSS 레이아웃은 크게 4가지만 있었죠:

1. **Block/inline 레이아웃**:

   * `display: block` → 위에서 아래로 세로 쌓기
   * `display: inline` → 글자처럼 가로로 흐르기
   * 정교한 “정렬”, “유연한 폭 분배”가 거의 없음

2. **float 레이아웃**:

   * 이미지 둘러싸기 용도로 만들어졌는데, 억지로 레이아웃에 남용
   * `clear`, `clearfix` 같은 기괴한 패턴 남발

3. **position (absolute/fixed)**:

   * 화면 특정 좌표에 “박아버리는” 용도라서 반응형/유연 레이아웃에 취약

4. **table 레이아웃**:

   * HTML table처럼 동작 → 유연성도 떨어지고 의미론도 안 맞음

> 문제: “**가로 방향으로 여러 박스를 적당히 나눠 담고, 남는 공간도 자동으로 분배하면서, 필요하면 줄바꿈도 하고, 가운데 정렬도 하고, 끝 정렬도 하고…**”
> 을 깔끔하게 지원하는 레이아웃이 없었습니다.

그래서 나온 게 **Flexbox = Flexible Box Layout** 입니다.

* **주로 한 방향(가로 또는 세로)을 기준으로 정렬하고 배치**하는 레이아웃
* 나머지 방향(크로스 축)은 **정렬/정렬 옵션** 넣어주는 역할

> 👉 그래서 Flexbox를 **“1차원(1D) 레이아웃”**, CSS Grid를 **“2차원(2D) 레이아웃”**이라고 부릅니다.

---

## 2. 가장 중요한 개념: Flex 컨테이너와 Flex 아이템 🧱

### 2.1 flex 컨테이너 만들기

```css
.container {
  display: flex;       /* 또는 inline-flex */
}
```

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

* `.container` → **flex 컨테이너**
* 그 **직계 자식들** (`.item`) → **flex 아이템**

컨테이너에 `display: flex`가 걸리는 순간:

* 기존의 block formatting context 대신 **flex formatting context**가 생성됩니다.
* float, clear 같은 속성은 사실상 의미가 사라짐
* 자식들은 **자동으로 inline-block 같은 것 안 쓰고도 “가로로 줄 세우는” 상태**가 됩니다.

---

## 3. Flex의 두 축: main axis와 cross axis 🎯

Flexbox의 모든 혼란의 출발점이 바로 **축 2개**입니다.
하지만 이거 하나만 정확히 잡으면 전체가 풀립니다.

### 3.1 메인 축(main axis)과 크로스 축(cross axis)

* **메인 축(main axis)**:
  → flex 아이템이 **나란히 줄 서는 방향**
* **크로스 축(cross axis)**:
  → 메인 축에 **직각인 방향**

축의 방향은 **`flex-direction` 하나로 결정**됩니다.

```css
.container {
  display: flex;
  flex-direction: row; /* 기본값 */
}
```

#### `flex-direction` 값들

1. `row` (기본값)

   * 메인 축: **가로(왼→오)**
   * 크로스 축: **세로(위→아래)**

2. `row-reverse`

   * 메인 축: **가로(오→왼)**
   * 크로스 축: **세로(위→아래)**

3. `column`

   * 메인 축: **세로(위→아래)**
   * 크로스 축: **가로(왼→오)**

4. `column-reverse`

   * 메인 축: **세로(아래→위)**
   * 크로스 축: **가로(왼→오)**

대략적인 그림으로 보면:

```text
/* flex-direction: row (기본) */

Main axis:  --->  (가로)
Cross axis:  |
             v   (세로)

[ item ][ item ][ item ]
```

```text
/* flex-direction: column */

Main axis:
  |
  v  (세로)
Cross axis:  ---> (가로)

[ item ]
[ item ]
[ item ]
```

> 👉 **축 2개를 동시에 쓴다 =**
>
> * 메인 축은 “아이템 배치 방향 + 여분 공간 분배”
> * 크로스 축은 “아이템을 그 라인 안에서 어떻게 세워 놓을지(위/중앙/아래/늘이기)”

라고 생각하시면 됩니다.
**메인 축(흐름), 크로스 축(옆으로 밀어붙이기/정렬)** 이라고 외우셔도 좋아요.

---

## 4. flex-wrap: 한 줄 vs 여러 줄 📦

기본적으로 flex는 **한 줄에 다 욱여넣으려고** 합니다.

```css
.container {
  display: flex;
  flex-wrap: nowrap;  /* 기본값 */
}
```

* `nowrap` : 한 줄에 다 집어넣기. 공간이 부족하면 **줄이 잘려서 스크롤/넘침**.
* `wrap` : 공간이 부족하면 **다음 줄(또는 다음 열)로 자동 줄바꿈**.
* `wrap-reverse` : 줄바꿈은 하는데, 줄의 순서를 반대로 쌓음.

`flex-direction: row; flex-wrap: wrap;`이면 이런 느낌:

```text
[ item1 ][ item2 ][ item3 ][ item4 ]
[ item5 ][ item6 ][ item7 ] ...
```

> 이 “여러 줄” 각각을 **flex line**이라고 부릅니다.
> 크로스 축 정렬(`align-content`)은 이 **여러 줄 덩어리**를 어떻게 배치할지를 결정합니다.

---

## 5. 컨테이너에서 쓰는 속성들 정리 🧩

### 5.1 flex-flow (direction + wrap)

```css
.container {
  display: flex;
  flex-flow: row wrap; /* = flex-direction: row + flex-wrap: wrap */
}
```

---

### 5.2 justify-content: 메인 축 정렬

**메인 축 방향**으로 **남는 공간을 어떻게 배분할지**를 정하는 속성입니다.

```css
.container {
  display: flex;
  justify-content: center; /* 예시 */
}
```

주요 값:

* `flex-start` (기본값) : 시작 지점에 몰아서 배치
* `center` : 가운데 몰기
* `flex-end` : 끝 지점에 몰기
* `space-between` : 양 끝에 아이템을 붙이고, **아이템 사이 공간만 균등 분배**
* `space-around` : 각 아이템 양쪽에 동일한 여백 (양 끝은 절반)
* `space-evenly` : 양 끝 포함 **모든 간격을 동일하게**

예시 (가로 row 기준):

```text
justify-content: space-between

|item1     item2     item3|

justify-content: center

|    item1 item2 item3    |
```

---

### 5.3 align-items: 한 줄 안에서 크로스 축 정렬

**각 flex line 내부에서**, 아이템들이 **크로스 축 방향**으로 어떻게 정렬될지 결정합니다.

```css
.container {
  display: flex;
  align-items: center;
}
```

값들 (row 기준이면 세로 방향):

* `stretch` (기본값) : **아이템을 flex line 높이에 맞춰 늘림**
* `flex-start` : 위쪽 정렬
* `center` : 중앙 정렬
* `flex-end` : 아래쪽 정렬
* `baseline` : 텍스트 baseline 기준으로 정렬

row + 다양한 높이 아이템일 때:

```text
align-items: flex-start

item1
item2
item3     <-- 위쪽 정렬

align-items: center

 item1
 item2
 item3   <-- 세로 가운데 맞춤
```

> **핵심**: `justify-content`는 **메인 축(흐름 방향)**,
> `align-items`는 **크로스 축(직각 방향)** 입니다.
> 두 개가 **서로 다른 축**에 붙는다고 기억해 두세요.

---

### 5.4 align-content: 여러 줄(라인) 자체를 정렬

`flex-wrap: wrap`으로 여러 줄이 생겼을 때,
**그 줄 덩어리들**을 크로스 축 방향으로 어떻게 배치할지입니다.

```css
.container {
  display: flex;
  flex-wrap: wrap;
  align-content: space-between;
}
```

값은 `justify-content`랑 거의 비슷하지만 **대상이 다름**:

* `align-content`: **“줄들” 사이의 간격**
* `align-items`: **“줄 안의 아이템들”의 정렬**

아주 많이 헷갈리는 포인트입니다:

> * **한 줄만 있을 때는 `align-content`가 아무 효과가 없다.**
>   (여러 줄이 되어야 줄 덩어리를 움직일 수 있으니까)

---

### 5.5 gap (row-gap, column-gap)

Flex 컨테이너에서도 `gap`을 쓸 수 있습니다.

```css
.container {
  display: flex;
  gap: 1rem;           /* row + column 모두 */
  /* row-gap: 1rem;
     column-gap: 2rem; */
}
```

* 옛날에는 `margin-right`로 간격 만들고 마지막 아이템만 예외 처리하고… 그랬지만
* 요즘은 그냥 `gap` 하나로 끝나는 시대입니다.

---

## 6. Flex 아이템에서 쓰는 속성들 🧮

이제 개별 아이템 쪽 속성입니다. (여기서부터 진짜 “유연한 레이아웃”이 시작됩니다.)

### 6.1 flex-basis: 기본 크기(메인 축 기준)

```css
.item {
  flex-basis: 200px;
}
```

* 메인 축 방향의 “**기본 크기**”입니다.

  * `flex-direction: row` → 가로 폭 기준
  * `flex-direction: column` → 세로 높이 기준
* `flex-basis: auto` (기본값)
  → `width`/`height` 값(메인 축 방향의 크기)을 사용합니다.

### 6.2 flex-grow: 남는 공간을 어떻게 나눠 가질지 (확대 계수)

```css
.item {
  flex-grow: 1;
}
```

* **컨테이너의 메인 축 방향 공간이 남을 때**,
  이 값을 기준으로 **남는 공간을 비율로 나눠 가집니다.**
* 예:

```text
컨테이너 width: 600px
item1 기본 width: 100px (flex-grow: 1)
item2 기본 width: 100px (flex-grow: 2)
item3 기본 width: 100px (flex-grow: 1)

합: 300px → 남는 공간: 300px

grow 합계 = 1 + 2 + 1 = 4

item1: 300px * 1/4 = 75px → 최종 175px
item2: 300px * 2/4 = 150px → 최종 250px
item3: 300px * 1/4 = 75px → 최종 175px
```

> 그래서 실무에서 **`flex: 1`**은
> “**남는 공간을 나눠 가지는 아이템**”이라는 의미로 자주 쓰입니다.

### 6.3 flex-shrink: 공간이 부족할 때 얼마나 줄어들지 (축소 계수)

```css
.item {
  flex-shrink: 1; /* 기본값 */
}
```

* **컨테이너에 공간이 부족할 때**,
  이 값을 기준으로 아이템들이 **얼마나 줄어들지 비율**로 나눕니다.

가령:

```text
컨테이너 width: 300px
item1 기본 width: 200px (flex-shrink: 1)
item2 기본 width: 200px (flex-shrink: 1)
합: 400px → 100px 부족
→ 두 아이템이 50px씩 줄어들어 150px + 150px
```

> `flex-shrink: 0`으로 하면
> “절대 줄어들지 마! 대신 다른 놈들이 줄어라!” 라는 의미가 됩니다.

---

### 6.4 flex (shorthand)

```css
.item {
  flex: 1;         /* flex-grow: 1; flex-shrink: 1; flex-basis: 0; */
  flex: 0 0 auto;  /* flex-grow: 0; flex-shrink: 0; flex-basis: auto; */
}
```

자주 쓰는 패턴:

* `flex: 1`
  → 남는 공간 쫙 나눠 갖기, `flex-basis: 0` 이라서 **기본 크기는 무시하고 균등 분배 느낌**

* `flex: 0 0 auto`
  → 유연하지 않은 아이템. 기본 크기 그대로 유지, 줄어들지도/늘어나지도 않게.

---

### 6.5 align-self: 아이템 개별 정렬

`align-items`는 컨테이너 전체에 적용인데,
특정 아이템만 다르게 정렬하고 싶을 때 씁니다.

```css
.item.special {
  align-self: flex-end;
}
```

---

## 7. “브라우저는 flex를 어떻게 계산하나?” (축 2개 동시 사용 이해하기) 🧠

사용자 입장에서 제일 미친 부분이 이거죠:

> “메인 축이니 크로스 축이니… 두 축을 동시에 쓰면
> **먼저 메인 축 정렬하고, 또 크로스 축에서 다시 조정하고…
> 도대체 순서가 어떻게 되는 거냐???**”

간단하게, 브라우저는 대충 이런 순서로 처리합니다:

### 7.1 Step 1: 아이템들을 라인으로 나눈다

1. `flex-direction`을 보고 **메인 축 방향**을 결정
2. `flex-wrap`을 보고

   * `nowrap` → 한 줄에 다 넣기
   * `wrap` / `wrap-reverse` → **공간이 부족할 때 다음 줄로 넘기기**

이때, **“라인(line)”**이라는 단위가 생깁니다.

### 7.2 Step 2: 각 아이템의 “기본 크기”를 계산

각 아이템에 대해:

1. `flex-basis`를 우선 사용
2. `flex-basis: auto`이면, 메인 축 방향의 `width`/`height` 참고
3. 둘 다 없으면 콘텐츠 크기(내용의 자연 크기)

이걸로 **“가상의 기본 크기(outer base size)”**를 잡습니다.

### 7.3 Step 3: 메인 축에서 grow/shrink 적용

각 라인에 대해:

1. 라인 안 아이템 기본 크기를 다 더한 뒤,
2. 컨테이너 메인 축 크기와 비교해서

   * 남는 공간(+) → `flex-grow`로 나눠 줌
   * 모자란 공간(-) → `flex-shrink`로 줄여 줌
3. 이렇게 해서 각 아이템의 **최종 메인 크기**가 결정됩니다.

> 👉 여기까지가 “메인 축”에서 하는 일입니다.
> 아직 크로스 축(세로 정렬 등)은 건드리지 않았어요.

### 7.4 Step 4: 각 라인의 크로스 축 크기 결정

각 라인의 **높이(또는 너비)**를 결정합니다. (크로스 축 기준)

* `align-items: stretch`면
  → 라인 높이를 기준으로 아이템들을 늘려버림
* 아이템마다 크기가 다르면
  → 라인의 크로스 크기는 그 중 **가장 큰 아이템** 기준이 됩니다.

### 7.5 Step 5: 라인 안에서 아이템 크로스 축 정렬 (align-items/align-self)

각 라인 내부에서:

* `align-items` (또는 각 아이템의 `align-self`)에 따라
  아이템을 **위/가운데/아래/늘이기/baseline 기준**으로 배치합니다.

---

### 7.6 Step 6: 여러 라인 덩어리를 크로스 축으로 배치 (align-content)

마지막으로, **여러 flex line 덩어리**를 크로스 축 방향으로 배치합니다.

* `align-content: flex-start | center | flex-end | space-between | space-around | stretch ...`

---

> 🔁 정리하면:
>
> 1. 메인 축 방향으로 줄을 나눈다 (wrap)
> 2. 각 줄 안에서 아이템의 **메인 크기**를 grow/shrink로 계산한다
> 3. 각 줄의 크로스 크기를 정한다 (가장 큰 아이템 or stretch)
> 4. 줄 안에서 아이템들을 **크로스 축으로 정렬**한다 (align-items)
> 5. 여러 줄 전체를 **크로스 축으로 정렬**한다 (align-content)
>
> ⇒ 이렇게 해서 “두 축을 동시에 사용하는” 결과가 나오는 겁니다.

이걸 알고 나면
**“메인 축 정리 → 크로스 축 정리”**라는 흐름으로 머리 속에서 시뮬레이션할 수 있습니다.

---

## 8. 자주 헷갈리는 포인트 정리 ⚠️

### 8.1 justify-content vs align-items vs align-content

* `justify-content`

  * **메인 축 방향**에서 아이템 간 간격/정렬 (`row`면 가로, `column`이면 세로)
* `align-items`

  * **각 라인 안에서** 아이템의 크로스 축 정렬 (위/중앙/아래/늘이기)
* `align-content`

  * **여러 줄(라인)의 덩어리**를 크로스 축 방향으로 정렬

질문으로 바꿔보면:

* “아이템들을 **가로**로 가운데로 모으고 싶다”

  * `flex-direction: row; justify-content: center;`
* “아이템들을 **세로**로 가운데 정렬하고 싶다”

  * `flex-direction: row; align-items: center;`
* “여러 줄로 감긴 줄 덩어리 전체를 위/가운데/아래로 옮기고 싶다”

  * `flex-wrap: wrap; align-content: center;` 등

---

### 8.2 width/height vs flex-basis

* 메인 축 방향의 크기를 **정말** 제어하고 싶다면 → `flex-basis`를 우선 고려하세요.
* `flex-basis: auto`일 때만 `width`/`height`가 메인 크기에 영향.

실무에서 많이 쓰는 패턴:

```css
.item {
  flex: 0 0 200px;  /* 최소 기준 200px, grow/shrink 안 함 */
}
```

---

### 8.3 flex:1의 정확한 의미

```css
.item {
  flex: 1;  /* flex: 1 1 0; */
}
```

* `flex-grow: 1` : 남는 공간을 나눠 가짐
* `flex-shrink: 1` : 공간 부족할 때 줄어들기도 함
* `flex-basis: 0` : 기본 크기를 0으로 보고, **남는 공간만 기준으로 균등 분배**

그래서 “3개의 아이템에 모두 `flex: 1`” 하면:

* 각자 **동일한 비율로 컨테이너를 나눠 가짐**
* `width`를 따로 안 줘도, 자동으로 1/3씩 차지하는 UI가 됩니다.

---

## 9. Flex vs Grid: 언제 뭘 쓸까? 🧭

질문하신 것처럼,
“차라리 CSS Grid만 쓰고 싶다…”는 마음이 드는 것도 매우 자연스러운 반응입니다. 😅

* **Flexbox**

  * 한 방향(가로 또는 세로)을 기준으로
    아이템을 유연하게 늘였다 줄였다 하면서 배치하는 **1D 레이아웃**
  * 예:

    * 네비게이션 바
    * 버튼 그룹
    * 카드들을 가로로 쭉 나열하면서 반응형으로 줄 바꾸기
    * “한 줄 안”에서 중앙 정렬, 양끝 정렬 등

* **Grid**

  * 행(row) + 열(column) **둘 다**를 동시에 다루는 **2D 레이아웃**
  * 예:

    * 복잡한 대시보드
    * 사진 갤러리
    * 전체 페이지 레이아웃

> 실무 팁:
>
> * “아이템들이 그냥 한 줄로 쭉 나열되는데, 중간 중간 간격만 잘 나눠주고 싶다” → **Flex**
> * “행/열을 모두 의식하면서, 어떤 칸에 무엇을 놓을지까지 설계해야 한다” → **Grid**

---

## 10. 최소 예제로 보는 Flex 정신 🧪

### 10.1 가로 중앙 정렬 + 세로 중앙 정렬 (클래식 카드)

```html
<div class="container">
  <div class="box">Center</div>
</div>
```

```css
.container {
  display: flex;
  justify-content: center; /* 가로 중앙 (메인 축) */
  align-items: center;     /* 세로 중앙 (크로스 축) */
  height: 100vh;
}

.box {
  padding: 2rem;
  background: #fff;
}
```

* `flex-direction` 기본값이 `row`이므로

  * 메인 축: 가로
  * 크로스 축: 세로
* `justify-content: center` → **가로 중앙**
* `align-items: center` → **세로 중앙**

“축을 둘 다 쓰면 어떻게 되냐?”의 대표 예제죠.
브라우저는 먼저 메인 축(가로) 기준으로 위치 잡고,
그 다음 크로스 축(세로) 기준으로 한 번 더 정렬해 줍니다.

---

## 마무리 🧵

정리해 보면 Flexbox는:

1. `display: flex`로 **새로운 레이아웃 세계(flex formatting context)**를 열고
2. `flex-direction`으로 **메인 축/크로스 축을 결정**한 다음
3. 메인 축에서

   * 줄 나누기(`flex-wrap`)
   * 남는/부족한 공간 분배(`flex-grow`, `flex-shrink`, `flex-basis`)
4. 크로스 축에서

   * 라인 내부 정렬(`align-items` / `align-self`)
   * 여러 라인 덩어리 정렬(`align-content`)
5. 남는 간격은 `gap`으로 깔끔하게 처리

하는 **1차원 레이아웃 엔진**입니다.


