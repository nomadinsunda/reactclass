CSS `float` 레이아웃은 **옛날 레이아웃의 주력 기술**이었고, 지금은 `flex`, `grid`가 있지만, 여전히 이해해 두면 좋습니다.
옛날 코드 유지보수, 블로그/게시판 테마 분석, 면접/시험 문제에도 종종 나오거든요. 😈

아예 “옛날 신문 레이아웃 엔진”이라고 생각하면 이해가 좀 쉽습니다.

---

## 1. `float`의 원래 목적: 텍스트에 그림 둘러싸기 📰

처음에 `float`는 **레이아웃용이 아니라**:

> *“이미지를 왼쪽에 붙이고, 오른쪽으로 텍스트가 흘러가게 하고 싶다”*

이런 용도로 만들어졌습니다.

```html
<p>
  <img src="photo.jpg" class="float-left" alt="사진" />
  Lorem ipsum dolor sit amet, consectetur adipiscing elit...
</p>
```

```css
.float-left {
  float: left;
  margin: 0 16px 16px 0; /* 오른쪽/아래쪽 여백 */
}
```

이렇게 하면:

* 사진은 **왼쪽에 달라붙고**,
* 텍스트는 사진의 오른쪽과 아래쪽으로 **둘러싸여 흐릅니다.**

이 “둘러싼다”는 개념이 `float`의 본질입니다.

---

## 2. float 기본 동작 정리 🧱

### 2.1 float 가능한 값

```css
float: left;
float: right;
float: none;   /* 기본값 */
float: inline-start; /* 논리 속성(잘 안 씀) */
float: inline-end;
```

현실적으로는 `left`, `right` 두 개가 메인입니다.

---

### 2.2 float가 걸리면 무슨 일이 생기나?

어떤 요소에 `float: left`를 주면:

1. 그 요소는 **일반적인 문서 흐름(normal flow)** 에서 빠져나옵니다.
2. 그리고 가능한 한 **왼쪽**(또는 `right`면 오른쪽)으로 붙습니다.
3. 뒤따르는 **텍스트나 인라인 요소**는 그 `float` 요소 주변을 **감싸며 흐릅니다**.
4. 하지만 **블록 요소**는 float을 감싸지 않고, float 요소의 아래로 내려가려 합니다.
   (단, 상황에 따라 float 옆으로도 올 수 있습니다)

즉, `float`는 **자기 자리에서 빠져나와 한쪽으로 붙어버리는 박스**입니다.

---

## 3. 고전 float 레이아웃 패턴: 2컬럼, 3컬럼 📚

`flex`/`grid`가 등장하기 전에는 `float`로 컬럼 레이아웃을 만들었습니다.

### 3.1 2컬럼 레이아웃 예시

```html
<div class="container">
  <div class="sidebar">Sidebar</div>
  <div class="content">Main Content</div>
</div>
```

```css
.container {
  border: 1px solid #ccc;
}

.sidebar {
  float: left;
  width: 200px;
  background: #f2f2f2;
}

.content {
  margin-left: 200px; /* 혹은 float: right; width: calc(100% - 200px); */
  background: #fff;
}
```

특징:

* `.sidebar`에 `float: left`로 왼쪽 컬럼.
* `.content`는 float 옆에 배치되도록 마진 또는 또 다른 float로 처리.
* 부모 `.container`는 **자식이 float이면 높이가 0이 되는 문제**가 생깁니다 (이건 아래에서).

---

## 4. float 레이아웃의 대표적인 문제들 💣

float 레이아웃이 욕을 먹는 가장 큰 이유는 **부모 높이 붕괴, clearfix, 예측하기 힘든 흐름** 때문입니다.

### 4.1 부모 높이 붕괴 문제

```html
<div class="container">
  <div class="box">A</div>
  <div class="box">B</div>
</div>
<p>다음 콘텐츠</p>
```

```css
.container {
  border: 2px solid red;
}

.box {
  float: left;
  width: 100px;
  height: 100px;
  background: lightblue;
}
```

이 경우:

* `.box` 두 개는 float 상태라 **normal flow에서 빠짐**.
* `.container`는 “내 안에 normal flow에 속한 애가 없네?” → **높이가 0으로 계산**.
* 결과적으로 `.container`의 빨간 border가 **내용을 감싸지 않고**,
  아래 `<p>`가 위로 말려 올라와 붙는 것처럼 보입니다.

#### 왜 이런 일이?

* float 요소는 **부모의 높이 계산에서 제외**되기 때문입니다.
* `position: absolute`도 비슷하게 부모 높이를 무시합니다.

---

## 5. 이 문제를 해결하는 고전 기술: `clear`와 clearfix 🧹

### 5.1 `clear` 속성

`clear`는 “이 요소는 **어느 방향의 float 옆에 붙지 않겠다**”는 선언입니다.

```css
clear: left;
clear: right;
clear: both; /* 왼쪽, 오른쪽 float 모두 회피 */
```

예시:

```html
<div class="container">
  <div class="float-box">float</div>
  <p class="clear-both">이 문단은 float 아래에서 시작합니다.</p>
</div>
```

```css
.float-box {
  float: left;
  width: 100px;
  height: 100px;
  background: lightblue;
}

.clear-both {
  clear: both;
}
```

* `.clear-both`는 “왼쪽이나 오른쪽에 float이 있으면 그 밑으로 내려가겠다” → float 아래로 밀려남.

하지만 이걸로 **부모 높이 문제**는 안 끝납니다.

---

### 5.2 clearfix 패턴 (부모가 float 자식을 “감싸게” 만들기)

float 레이아웃에서 거의 의식처럼 쓰던 코드입니다.

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

그 다음 부모에 `clearfix` 클래스를 붙입니다.

```html
<div class="container clearfix">
  <div class="box">A</div>
  <div class="box">B</div>
</div>
```

이렇게 하면:

* `.container`의 **가상 요소** `::after`가 마지막에 하나 생김.
* 그 가상 요소는 `clear: both` 상태라, **위의 float 박스들 아래로 내려가면서 자기 위치를 잡음**.
* 이 가상 요소도 normal flow에 속하므로, `.container`의 높이를 **떠받치게** 됨.
* 결과적으로 부모 컨테이너의 높이가 정상적으로 **float 자식을 감싸게** 됩니다.

요약하면:

> float 자식들을 감싸도록 부모에 **보이지 않는 clear 요소를 하나 자동으로 꽂아 넣는 트릭**이 `clearfix`.

---

## 6. BFC(Block Formatting Context)와 float의 관계 🧠

float 문제를 해결할 때 `overflow: hidden` / `auto` 같은 속성을 써서 **BFC**를 만드는 방법도 많이 사용합니다.

```css
.container {
  overflow: hidden; /* 또는 auto */
  border: 2px solid red;
}
```

이렇게 하면:

* `.container`는 새로운 **Block Formatting Context**(BFC)를 형성.
* BFC는 자신의 영역 안에 있는 float 요소들의 높이를 **레이아웃 계산에 포함**합니다.
* 그래서 `clearfix` 없이도 부모가 자식 float의 높이를 감싸게 됩니다.

단점:

* 진짜로 `overflow`가 잘리는 효과도 같이 생길 수 있으니 주의 (`box-shadow`나 툴팁 등).

---

## 7. float 레이아웃을 이해하기 위한 시각적인 흐름 정리 🧩

텍스트 + 이미지 예를 그림처럼 생각해 보면:

1. 먼저 텍스트가 **왼쪽에서 오른쪽으로, 위에서 아래로** 그려진다고 가정.
2. 중간에 `float: left` 이미지를 만나면:

   * 그 이미지는 현재 줄에서 **왼쪽 끝 근처**로 달라붙고,
   * 텍스트는 이미지의 오른쪽 빈 공간에서부터 다시 줄을 그립니다.
3. 텍스트가 이미지의 아래까지 내려가면,

   * 그 다음 줄부터는 이미지가 더 이상 흐름을 막지 않으므로,
     다시 전체 너비를 사용하게 됩니다.

여기에 블록 요소까지 얽히면:

* 블록 요소는 보통 **float 아래로 내려가려고** 함.
* 하지만 충분한 공간과 특정 상황에서는 **float 옆으로** 끼어들기도 해서 헷갈립니다.
* 그래서 실무에서는 float로 복잡한 레이아웃을 짜는 걸 **지양**하게 된 거죠.

---

## 8. float vs flex vs grid: 언제 뭘 써야 하나? ⚔️

### 8.1 float

* 원래 목적: **텍스트 주위에 이미지/박스 떠 있게 하기**.
* 레이아웃에 쓰면:

  * 부모 높이 붕괴, clearfix, BFC 등 **트릭이 많이 필요**.
  * 수직 정렬, 동적 중간 정렬 같은 건 매우 번거롭고 제한적.
* 요즘 권장 사용처:

  * 신문/블로그 텍스트에 작은 이미지 떠 있게 할 때,
  * 레거시 코드 유지보수 시 이해용.

### 8.2 flexbox

* 일차원(1D) 레이아웃: 한 축 기준으로 아이템을 나열하고 **정렬/간격** 제어에 특화.
* `justify-content`, `align-items`, `gap` 등으로 중간 정렬, 양끝 정렬 등 쉽게 가능.
* float처럼 정상 흐름에서 빠지는 게 아니라, **레이아웃 시스템 안에서 계산**되므로 예측 가능.

```css
.container {
  display: flex;
}
```

### 8.3 grid

* 이차원(2D) 레이아웃: **행(row)** 과 **열(column)** 을 동시에 설계.
* “첫 번째 행, 두 번째 열에 이 컴포넌트”처럼 명시적 배치 가능.
* 복잡한 레이아웃(대시보드, 잡지식 레이아웃 등)에 최적화.

---

## 9. 아직도 float를 봐야 하는 이유 🧐

현대 코드에서는 거의 `flex`, `grid`가 표준이지만:

1. **옛날 테마/템플릿** – 워드프레스, XE, 오래된 게시판 스킨, 옛날 회사 내부 시스템 등은 float 레이아웃 투성이.
2. **면접/시험** – “float과 clear가 뭐냐”, “clearfix 설명해봐라” 같은 질문은 종종 나옴.
3. **브라우저 동작 이해** – float을 이해하면 **브라우저의 레이아웃 엔진이 블록과 인라인을 어떻게 배치하는지** 감이 좋아집니다.

당장 새로 만드는 프로젝트에서 레이아웃을 float로 짜는 건 **절대 비추**지만,
**읽고 이해하고 디버깅할 줄 아는 것**은 프론트엔드 개발자로서 꽤 중요한 스킬입니다.

---

## 10. 정리 ✍️

핵심만 요약하면:

* `float`는 원래 **텍스트에 박스를 떠 있게 하는 용도**였다.
* `float`를 쓰면 요소가 **normal flow에서 빠져나와** 왼쪽/오른쪽으로 붙고, 주변 텍스트가 이를 둘러싼다.
* 레이아웃에 쓰면:

  * 부모 높이가 0이 되는 **높이 붕괴 문제**가 생김.
  * 이를 해결하기 위해 `clear` + **clearfix**, 또는 `overflow: hidden/auto`로 BFC 생성 같은 트릭이 필요하다.
* 요즘 레이아웃의 주인공은 `flex`와 `grid`지만,

  * 옛 코드 분석, 시험/면접, 브라우저 레이아웃 이해를 위해 float 개념은 알아두는 게 좋다.

