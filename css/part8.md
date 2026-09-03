# PART 8. CSS Grid

## 1. 왜 CSS Grid가 필요한가?

PART 7에서는 **Flexbox**를 배웠습니다.

Flexbox의 핵심은 다음과 같습니다.

```text
Flex Container
        ↓
Main Axis를 중심으로
Flex Item 배치
```

예를 들어:

```text
┌─────────────────────────────────┐
│ [A]      [B]      [C]          │
└─────────────────────────────────┘
```

Flexbox는 한 방향을 중심으로 Item을 배치하고 정렬하는 데 매우 강력합니다.

하지만 다음과 같은 Layout을 생각해 보겠습니다.

```text
┌─────────┬─────────┬─────────┐
│    A    │    B    │    C    │
├─────────┼─────────┼─────────┤
│    D    │    E    │    F    │
└─────────┴─────────┴─────────┘
```

이번에는 단순히 한 방향으로 Item을 배치하는 것이 아니라:

```text
Column
  +

Row
```

두 방향을 동시에 제어해야 합니다.

이러한 **2차원 Layout**을 위해 만들어진 CSS Layout System이 **CSS Grid**입니다.

---

# 2. CSS Grid란?

CSS Grid는 **Row와 Column으로 이루어진 2차원 Grid를 만들고, 그 Grid 위에 Item을 배치하는 Layout System**입니다.

```text
              Columns

           1       2       3

        ┌───────┬───────┬───────┐
Row 1   │   A   │   B   │   C   │
        ├───────┼───────┼───────┤
Row 2   │   D   │   E   │   F   │
        └───────┴───────┴───────┘
```

Flexbox가:

```text
Main Axis 중심
```

이라면 Grid는:

```text
Row Axis
   +
Column Axis
```

를 함께 다룹니다.

따라서 Grid를 흔히:

> **2차원 Layout System**

이라고 합니다.

---

# 3. Flexbox와 Grid의 가장 중요한 차이

Flexbox:

```text
Main Axis
────────────────────────→

[A] [B] [C] [D]
```

한 방향을 중심으로 Item들의 관계를 제어합니다.

Grid:

```text
          Column

      1       2       3

   ┌───────┬───────┬───────┐
1  │   A   │   B   │   C   │
   ├───────┼───────┼───────┤
2  │   D   │   E   │   F   │
   └───────┴───────┴───────┘
          Row
```

행과 열을 함께 제어합니다.

단순화하면:

```text
Flexbox
→ 1차원 중심

Grid
→ 2차원 중심
```

입니다.

둘은 경쟁 관계가 아닙니다.

```text
Flexbox + Grid
```

를 함께 사용하는 것이 일반적입니다.

---

# 4. Grid Container 만들기

HTML:

```html
<div class="container">

  <div>A</div>
  <div>B</div>
  <div>C</div>

  <div>D</div>
  <div>E</div>
  <div>F</div>

</div>
```

CSS:

```css
.container {
  display: grid;
}
```

이 순간:

```text
.container
→ Grid Container

직계 자식
→ Grid Items
```

이 됩니다.

구조:

```text
Grid Container
│
├── Grid Item A
├── Grid Item B
├── Grid Item C
├── Grid Item D
├── Grid Item E
└── Grid Item F
```

Flexbox와 마찬가지로 **Grid Container의 직계 자식이 Grid Item**이 됩니다.

---

# 5. `display: grid`만 사용하면?

다음 코드만 작성했다고 하겠습니다.

```css
.container {
  display: grid;
}
```

아직 명시적인 Column을 만들지 않았습니다.

따라서 일반적인 경우 Item들은 자동 배치 알고리즘에 의해 하나의 Column 안에서 배치됩니다.

```text
┌─────────────────────┐
│ A                   │
├─────────────────────┤
│ B                   │
├─────────────────────┤
│ C                   │
├─────────────────────┤
│ D                   │
└─────────────────────┘
```

Grid의 진짜 힘은 **Grid Track을 정의하면서 시작됩니다.**

---

# 6. Grid Track이란?

Grid의 Row와 Column을 구성하는 각각의 영역을 **Grid Track**이라고 합니다.

예:

```text
Column Track
     ↓

┌────────┬────────┬────────┐
│        │        │        │
└────────┴────────┴────────┘
    ↑        ↑        ↑
 Track    Track    Track
```

Row 역시 Track입니다.

```text
┌─────────────────────────┐ ← Row Track
├─────────────────────────┤
└─────────────────────────┘ ← Row Track
```

즉:

```text
Grid Track

├── Row Track
└── Column Track
```

입니다.

---

# 7. `grid-template-columns`

Grid에서 가장 중요한 Property 중 하나입니다.

Column의 개수와 크기를 정의합니다.

```css
.container {
  display: grid;

  grid-template-columns:
    200px 200px 200px;
}
```

결과:

```text
       200px      200px      200px

     ┌────────┬────────┬────────┐
     │   A    │   B    │   C    │
     ├────────┼────────┼────────┤
     │   D    │   E    │   F    │
     └────────┴────────┴────────┘
```

즉:

```text
값 하나
→ Column 하나

값 세 개
→ Column 세 개
```

입니다.

---

# 8. 서로 다른 Column 크기

다음처럼 지정할 수도 있습니다.

```css
.container {
  display: grid;

  grid-template-columns:
    200px 400px 100px;
}
```

결과:

```text
  200px        400px          100px

┌────────┬────────────────┬──────┐
│   A    │       B        │  C   │
├────────┼────────────────┼──────┤
│   D    │       E        │  F   │
└────────┴────────────────┴──────┘
```

Grid는 Column마다 서로 다른 크기를 지정할 수 있습니다.

---

# 9. `grid-template-rows`

Row 크기는:

```css
grid-template-rows
```

로 정의합니다.

예:

```css
.container {
  display: grid;

  grid-template-columns:
    200px 200px 200px;

  grid-template-rows:
    100px 200px;
}
```

결과:

```text
             Columns

       200      200      200

    ┌────────┬────────┬────────┐
100 │   A    │   B    │   C    │
    ├────────┼────────┼────────┤
200 │   D    │   E    │   F    │
    └────────┴────────┴────────┘
```

이제 Row와 Column을 모두 제어하고 있습니다.

이것이 Grid가 **2차원 Layout**인 이유입니다.

---

# 10. `fr` 단위

Grid에서 매우 중요한 새로운 단위가 등장합니다.

```text
fr
```

`fr`은 **Grid Container에서 분배 가능한 공간의 비율**을 나타내는 Grid 전용 단위입니다.

예:

```css
.container {
  display: grid;

  grid-template-columns:
    1fr 1fr 1fr;
}
```

Container의 공간을 세 Column이 나눠 가집니다.

```text
┌──────────┬──────────┬──────────┐
│   1fr    │   1fr    │   1fr    │
└──────────┴──────────┴──────────┘
```

같은 비율이므로 세 Column의 크기가 동일합니다.

---

# 11. `1fr 2fr 1fr`

```css
grid-template-columns:
  1fr 2fr 1fr;
```

이라면:

```text
        1       2       1

┌─────────┬──────────────────┬─────────┐
│   1fr   │       2fr        │   1fr   │
└─────────┴──────────────────┴─────────┘
```

분배 가능한 공간을:

```text
1 : 2 : 1
```

비율로 나눕니다.

Flexbox의 `flex-grow`와 비슷한 느낌이 있지만 같은 개념은 아닙니다.

---

# 12. `fr`은 전체 width 비율이라는 뜻인가?

정확하게는 아닙니다.

다음처럼 생각하는 것이 좋습니다.

```text
Grid Container
      ↓
고정 크기 / gap / Content 등의
영향을 먼저 고려
      ↓
분배 가능한 공간
      ↓
fr 비율로 분배
```

예:

```css
grid-template-columns:
  200px 1fr 2fr;
```

이라면:

```text
Container
┌─────────┬────────────┬──────────────────┐
│ 200px   │    1fr     │       2fr        │
└─────────┴────────────┴──────────────────┘
```

200px 등을 고려한 뒤 남는 공간을 `1 : 2`로 나눕니다.

---

# 13. `repeat()`

같은 Track 정의를 반복하면 코드가 길어집니다.

```css
grid-template-columns:
  1fr 1fr 1fr 1fr;
```

Grid에서는 `repeat()`를 사용할 수 있습니다.

```css
grid-template-columns:
  repeat(4, 1fr);
```

의미:

```text
1fr을 4번 반복
```

결과:

```text
┌────────┬────────┬────────┬────────┐
│  1fr   │  1fr   │  1fr   │  1fr  │
└────────┴────────┴────────┴────────┘
```

---

# 14. `gap`

Flexbox에서 배웠던 `gap`을 Grid에서도 사용합니다.

```css
.container {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 20px;
}
```

결과:

```text
┌───────┐ 20px ┌───────┐ 20px ┌───────┐
│   A   │      │   B   │      │   C   │
└───────┘      └───────┘      └───────┘
   ↕
 20px
   ↕
┌───────┐      ┌───────┐      ┌───────┐
│   D   │      │   E   │      │   F   │
└───────┘      └───────┘      └───────┘
```

---

# 15. `row-gap`, `column-gap`

각 방향을 별도로 지정할 수도 있습니다.

```css
.container {
  row-gap: 20px;
  column-gap: 30px;
}
```

또는:

```css
gap: 20px 30px;
```

의미:

```text
gap:
row-gap column-gap
```

입니다.

---

# 16. Grid Line

Grid에서는 Item의 위치를 이해하기 위해 **Grid Line** 개념이 중요합니다.

3개의 Column을 만들면 Column Line은 4개입니다.

```text
Line       Line       Line       Line
 1           2          3          4
 │           │          │          │
 ▼           ▼          ▼          ▼

 ┌───────────┬──────────┬──────────┐
 │ Column 1  │ Column 2 │ Column 3 │
 └───────────┴──────────┴──────────┘
```

즉:

```text
Track 3개
→ Line 4개
```

입니다.

Row도 같은 원리입니다.

---

# 17. `grid-column-start` / `grid-column-end`

Grid Item이 어떤 Column Line에서 시작하고 끝날지 지정할 수 있습니다.

```css
.item {
  grid-column-start: 1;
  grid-column-end: 3;
}
```

의미:

```text
Line 1부터
Line 3까지
```

입니다.

```text
1            2            3            4
│            │            │            │
┌────────────┴────────────┬────────────┐
│          Item           │            │
└─────────────────────────┴────────────┘
```

Item이 두 Column을 차지합니다.

---

# 18. `grid-column`

Shorthand를 사용할 수 있습니다.

```css
.item {
  grid-column: 1 / 3;
}
```

즉:

```text
start / end
```

입니다.

```text
grid-column: 1 / 3
```

은:

```text
Column Line 1
      ↓
      ├───────────────┐
      │     Item      │
      └───────────────┤
                      ↑
                 Column Line 3
```

입니다.

---

# 19. `span`

몇 개의 Track을 차지할지 지정할 수도 있습니다.

```css
.item {
  grid-column: span 2;
}
```

의미:

```text
현재 위치에서
Column 2개 차지
```

입니다.

```text
┌───────────────┬────────┐
│     Item      │        │
│   span 2      │        │
└───────────────┴────────┘
```

---

# 20. `grid-row`

Row에서도 동일합니다.

```css
.item {
  grid-row: 1 / 3;
}
```

```text
Line 1
──────────────
│            │
│    Item    │
│            │
──────────────
Line 3
```

두 Row를 차지합니다.

---

# 21. Row와 Column을 동시에 지정

```css
.item {
  grid-column: 1 / 3;
  grid-row: 1 / 3;
}
```

그러면:

```text
┌───────────────┬───────┐
│               │       │
│               │   B   │
│       A       ├───────┤
│               │   C   │
│               │       │
└───────────────┴───────┘
```

처럼 여러 Row와 Column에 걸쳐 Item을 배치할 수 있습니다.

Grid의 강력한 특징입니다.

---

# 22. `grid-template-areas`

Grid에서는 Line 번호 대신 **영역에 이름을 붙여 Layout을 설계**할 수도 있습니다.

예를 들어 웹 페이지:

```text
┌─────────────────────────────┐
│           Header            │
├─────────┬───────────────────┤
│ Sidebar │       Main        │
├─────────┴───────────────────┤
│           Footer            │
└─────────────────────────────┘
```

다음처럼 표현할 수 있습니다.

```css
.layout {
  display: grid;

  grid-template-columns:
    250px 1fr;

  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}
```

---

# 23. `grid-area`

각 Item에 영역 이름을 지정합니다.

```css
.header {
  grid-area: header;
}

.sidebar {
  grid-area: sidebar;
}

.main {
  grid-area: main;
}

.footer {
  grid-area: footer;
}
```

HTML:

```html
<div class="layout">

  <header class="header">
    Header
  </header>

  <aside class="sidebar">
    Sidebar
  </aside>

  <main class="main">
    Main
  </main>

  <footer class="footer">
    Footer
  </footer>

</div>
```

Layout 구조가 CSS 코드 자체에 매우 명확하게 나타납니다.

---

# 24. Grid Area의 장점

다음 코드만 봐도:

```css
grid-template-areas:
  "header header"
  "sidebar main"
  "footer footer";
```

화면 구조를 어느 정도 읽을 수 있습니다.

```text
header | header

sidebar | main

footer | footer
```

따라서 복잡한 Page Layout을 표현할 때 매우 유용합니다.

---

# 25. Explicit Grid

다음처럼 직접 Track을 정의했다고 하겠습니다.

```css
.container {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  grid-template-rows:
    100px 100px;
}
```

우리가 명시적으로 정의한 Grid 영역을 **Explicit Grid**라고 합니다.

```text
Explicit Grid

┌───────┬───────┬───────┐
│       │       │       │
├───────┼───────┼───────┤
│       │       │       │
└───────┴───────┴───────┘
```

---

# 26. Implicit Grid

그런데 Item이 더 많다면 어떻게 될까요?

```text
A B C
D E F
G H I
```

우리가 Row를 2개만 정의했는데 세 번째 Row가 필요합니다.

브라우저는 필요한 Track을 자동으로 생성합니다.

이렇게 자동으로 생성되는 Grid를 **Implicit Grid**라고 합니다.

```text
Explicit
┌────┬────┬────┐
│ A  │ B  │ C  │
├────┼────┼────┤
│ D  │ E  │ F  │
└────┴────┴────┘

Implicit
┌────┬────┬────┐
│ G  │ H  │ I  │
└────┴────┴────┘
```

---

# 27. `grid-auto-rows`

Implicit Row의 크기를 지정할 수 있습니다.

```css
.container {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  grid-auto-rows: 150px;
}
```

자동으로 생성되는 Row가:

```text
150px
```

크기를 갖게 됩니다.

---

# 28. `grid-auto-columns`

Implicit Column도 마찬가지입니다.

```css
grid-auto-columns: 200px;
```

자동 생성되는 Column Track의 크기를 제어합니다.

---

# 29. `grid-auto-flow`

Grid Item의 자동 배치 방향을 제어합니다.

디폴트는 일반적으로:

```css
grid-auto-flow: row;
```

입니다.

개념적으로:

```text
A → B → C
        ↓
D → E → F
```

Row 방향으로 채워 나갑니다.

다음처럼 사용할 수도 있습니다.

```css
grid-auto-flow: column;
```

개념적으로:

```text
A   C   E
↓   ↓   ↓
B   D   F
```

처럼 Column 방향을 중심으로 자동 배치합니다.

---

# 30. `minmax()`

반응형 Grid에서 매우 중요한 함수입니다.

```css
grid-template-columns:
  repeat(3, minmax(200px, 1fr));
```

의미:

```text
각 Column은

최소 200px
최대 1fr
```

입니다.

즉:

```text
200px보다 너무 작아지지 않으면서
남는 공간이 있으면 확장
```

할 수 있습니다.

---

# 31. `auto-fit`

반응형 Grid에서 매우 많이 사용하는 패턴입니다.

```css
.container {
  display: grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(250px, 1fr)
    );

  gap: 20px;
}
```

Container가 넓으면:

```text
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Card 1│ │Card 2│ │Card 3│ │Card 4│
└──────┘ └──────┘ └──────┘ └──────┘
```

좁아지면:

```text
┌──────────┐ ┌──────────┐
│ Card 1   │ │ Card 2   │
└──────────┘ └──────────┘

┌──────────┐ ┌──────────┐
│ Card 3   │ │ Card 4   │
└──────────┘ └──────────┘
```

더 좁아지면:

```text
┌────────────────────┐
│ Card 1             │
└────────────────────┘

┌────────────────────┐
│ Card 2             │
└────────────────────┘
```

처럼 바뀔 수 있습니다.

Media Query 없이도 상당한 반응형 Layout을 만들 수 있습니다.

---

# 32. `auto-fit`과 `auto-fill`

둘은 비슷하지만 빈 Track을 처리하는 방식에서 차이가 있습니다.

단순화하면:

```text
auto-fill
→ 들어갈 수 있는 만큼
  Track을 생성하는 성격

auto-fit
→ 빈 Track을 접어서
  실제 Item들이 공간을
  더 활용할 수 있게 하는 성격
```

실무에서 Card Grid를 만들 때는:

```css
repeat(
  auto-fit,
  minmax(250px, 1fr)
)
```

패턴을 매우 자주 볼 수 있습니다.

---

# 33. Grid Item 정렬

Flexbox와 마찬가지로 Grid에서도 Item을 정렬할 수 있습니다.

대표적인 Property:

```text
justify-items
align-items
place-items
```

Grid에서는 두 축을 다음처럼 이해하면 좋습니다.

```text
Inline Axis
→ 일반적인 가로 방향

Block Axis
→ 일반적인 세로 방향
```

Writing Mode에 따라 실제 물리적 방향은 달라질 수 있습니다.

---

# 34. `justify-items`

Grid Cell 안에서 Grid Item을 **Inline Axis 방향**으로 정렬합니다.

```css
.container {
  justify-items: start;
}
```

```text
Cell
┌───────────────────┐
│ [Item]            │
└───────────────────┘
```

`center`:

```text
┌───────────────────┐
│      [Item]       │
└───────────────────┘
```

`end`:

```text
┌───────────────────┐
│            [Item] │
└───────────────────┘
```

---

# 35. `align-items`

Grid Cell 안에서 Grid Item을 **Block Axis 방향**으로 정렬합니다.

```css
.container {
  align-items: center;
}
```

```text
Cell
┌───────────────────┐
│                   │
│      [Item]       │
│                   │
└───────────────────┘
```

---

# 36. `place-items`

`align-items`와 `justify-items`를 한 번에 지정할 수 있습니다.

```css
.container {
  place-items: center;
}
```

개념적으로:

```css
align-items: center;
justify-items: center;
```

입니다.

따라서 Grid Item을 Cell 중앙에 쉽게 배치할 수 있습니다.

---

# 37. Grid 전체를 Container 안에서 정렬

Grid Item이 아니라 **Grid Tracks 전체**를 Container 안에서 정렬하고 싶을 수도 있습니다.

이때:

```text
justify-content
align-content
place-content
```

를 사용합니다.

Flexbox에서 배웠던 이름과 비슷하지만 Grid에서는 Grid Track 전체의 배치와 관련됩니다.

---

# 38. Items와 Content 구분

매우 중요합니다.

```text
justify-items
align-items

       ↓

각 Grid Cell 안에서
Item 정렬
```

반면:

```text
justify-content
align-content

       ↓

Grid Tracks 전체를
Container 안에서 정렬
```

입니다.

즉:

```text
items
→ Item

content
→ Grid 전체
```

라고 구분하면 이해하기 쉽습니다.

---

# 39. 개별 Item 정렬

특정 Grid Item만 다르게 정렬하려면:

```text
justify-self
align-self
```

를 사용합니다.

예:

```css
.item {
  justify-self: end;
  align-self: center;
}
```

즉:

```text
Container 전체 Item
→ justify-items
→ align-items

특정 Item
→ justify-self
→ align-self
```

입니다.

---

# 40. `place-self`

개별 Item의 두 축 정렬을 한 번에 지정할 수도 있습니다.

```css
.item {
  place-self: center;
}
```

개념적으로:

```css
align-self: center;
justify-self: center;
```

입니다.

---

# 41. Grid를 이용한 완전 중앙 정렬

Flexbox에서는:

```css
.container {
  display: flex;

  justify-content: center;
  align-items: center;
}
```

를 사용했습니다.

Grid에서는:

```css
.container {
  display: grid;

  place-items: center;
}
```

로 매우 간단하게 표현할 수 있습니다.

```text
┌─────────────────────────────┐
│                             │
│                             │
│           [Item]            │
│                             │
│                             │
└─────────────────────────────┘
```

---

# 42. 실전 예제 1 — Card Grid

HTML:

```html
<div class="cards">

  <article class="card">A</article>
  <article class="card">B</article>
  <article class="card">C</article>
  <article class="card">D</article>

</div>
```

CSS:

```css
.cards {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 20px;
}
```

결과:

```text
┌────────┐ ┌────────┐ ┌────────┐
│   A    │ │   B    │ │   C    │
└────────┘ └────────┘ └────────┘

┌────────┐
│   D    │
└────────┘
```

---

# 43. 실전 예제 2 — Responsive Card Grid

다음처럼 변경해 보겠습니다.

```css
.cards {
  display: grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(250px, 1fr)
    );

  gap: 20px;
}
```

이제 화면 크기에 따라 Column 개수가 자연스럽게 변경됩니다.

```text
Desktop

[A] [B] [C] [D]


Tablet

[A] [B]

[C] [D]


Mobile

[A]

[B]

[C]

[D]
```

Grid의 대표적인 반응형 패턴입니다.

---

# 44. 실전 예제 3 — 전체 Page Layout

HTML:

```html
<div class="layout">

  <header>Header</header>

  <aside>Sidebar</aside>

  <main>Main</main>

  <footer>Footer</footer>

</div>
```

CSS:

```css
.layout {
  min-height: 100vh;

  display: grid;

  grid-template-columns:
    250px 1fr;

  grid-template-rows:
    auto 1fr auto;

  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}

header {
  grid-area: header;
}

aside {
  grid-area: sidebar;
}

main {
  grid-area: main;
}

footer {
  grid-area: footer;
}
```

결과:

```text
┌────────────────────────────────┐
│            Header              │
├──────────┬─────────────────────┤
│          │                     │
│ Sidebar  │        Main         │
│          │                     │
├──────────┴─────────────────────┤
│            Footer              │
└────────────────────────────────┘
```

---

# 45. 실전 예제 4 — Dashboard

Grid는 Dashboard Layout에도 매우 적합합니다.

```text
┌───────────┬───────────┬───────────┐
│ Sales     │ Users     │ Orders    │
├───────────┴───────────┼───────────┤
│                       │           │
│       Chart           │ Recent    │
│                       │ Orders    │
└───────────────────────┴───────────┘
```

각 Widget이 여러 Row나 Column을 차지하도록 할 수 있습니다.

```css
.chart {
  grid-column: span 2;
}
```

처럼 구성할 수 있습니다.

---

# 46. Grid 안에 Flexbox 사용

Grid와 Flexbox를 하나만 선택해야 하는 것은 아닙니다.

예:

```text
Page
Grid
│
├── Header
│     ↓
│   Flexbox
│   Logo Menu Login
│
├── Sidebar
│
├── Main
│     ↓
│   Grid
│   Card Card Card
│
└── Footer
```

CSS:

```css
.page {
  display: grid;
}

.header {
  display: flex;
}

.cards {
  display: grid;
}
```

이것이 실무에서 매우 일반적인 패턴입니다.

---

# 47. Grid Item 안에 또 Grid 사용

Grid 역시 중첩할 수 있습니다.

```text
Grid Container
│
├── Item
│
└── Item
     │
     └── Grid Container
          ├── Item
          └── Item
```

즉 Layout System은 계층적으로 구성할 수 있습니다.

---

# 48. Flexbox를 사용할까 Grid를 사용할까?

다음 질문을 먼저 합니다.

### 한 방향을 중심으로 배치하는가?

```text
Logo   Menu   Login
```

Flexbox가 자연스럽습니다.

### Row와 Column 구조를 함께 설계하는가?

```text
┌─────┬─────┬─────┐
│     │     │     │
├─────┼─────┼─────┤
│     │     │     │
└─────┴─────┴─────┘
```

Grid가 자연스럽습니다.

---

# 49. Flexbox vs Grid

| 특징               | Flexbox         | Grid         |
| ---------------- | --------------- | ------------ |
| Layout 관점        | 1차원 중심          | 2차원 중심       |
| 핵심 개념            | Main/Cross Axis | Row/Column   |
| Item 배치          | Content 중심에 강함  | Track 중심에 강함 |
| Row/Column 동시 제어 | 제한적             | 강력           |
| Navigation       | 매우 적합           | 가능           |
| Card Grid        | 가능              | 매우 적합        |
| Page Layout      | 가능              | 매우 적합        |
| 내부 정렬            | 매우 강력           | 매우 강력        |

핵심:

```text
Flexbox
→ Item들의 관계

Grid
→ Layout 구조
```

라고 생각하면 선택하기 쉽습니다.

---

# 50. Position vs Flexbox vs Grid

PART 6과 PART 7까지 포함해서 비교해 보겠습니다.

```text
Position
→ 특정 Box의 위치 제어

Flexbox
→ 한 방향을 중심으로
  여러 Item의 관계 제어

Grid
→ Row + Column으로
  2차원 Layout 구조 제어
```

예:

```text
SALE Badge
→ Position


Navigation
Logo Menu Login
→ Flexbox


Dashboard
Row × Column
→ Grid
```

---

# 51. 자주 하는 실수 1

### Grid는 Flexbox의 업그레이드 버전이다.

아닙니다.

둘의 목적이 다릅니다.

```text
Flexbox
→ 1차원 관계형 Layout

Grid
→ 2차원 Track Layout
```

따라서 Grid가 Flexbox를 대체하는 것이 아닙니다.

---

# 52. 자주 하는 실수 2

### `1fr 1fr 1fr`은 항상 정확히 Container width의 1/3이다.

항상 그렇게 단순하게 생각하면 안 됩니다.

Grid Sizing에는:

```text
Content
Minimum Size
Gap
Fixed Track
```

등이 영향을 줄 수 있습니다.

따라서 `fr`은:

> **분배 가능한 공간을 나누는 단위**

라고 이해하는 것이 정확합니다.

---

# 53. 자주 하는 실수 3

### Grid Item은 모든 자손이다.

아닙니다.

Flexbox와 동일하게 Grid Container의 **직계 자식**이 Grid Item이 됩니다.

```text
Grid Container
│
├── Child A
│    ↑
│ Grid Item
│
└── Child B
     │
     └── Grandchild
          ↑
      Container의
      직접 Grid Item 아님
```

---

# 54. 자주 하는 실수 4

### Grid Line 번호 = Column 번호

아닙니다.

3개의 Column이 있다면:

```text
Column
   1        2        3

┌────────┬────────┬────────┐
│        │        │        │
└────────┴────────┴────────┘

↑        ↑        ↑        ↑
1        2        3        4
        Grid Lines
```

즉:

```text
3 Columns
→ 4 Column Lines
```

입니다.

---

# 55. 자주 하는 실수 5

### `justify-content`와 `justify-items`가 같다.

다릅니다.

```text
justify-items
       ↓
각 Grid Cell 내부에서
Item 정렬
```

```text
justify-content
       ↓
Grid Tracks 전체를
Container 내부에서 정렬
```

이 차이는 매우 중요합니다.

---

# 56. 자주 하는 실수 6 — Content Overflow

다음 Layout을 생각해 보겠습니다.

```css
grid-template-columns:
  250px 1fr;
```

Main 영역에 매우 긴 Content가 들어가면 예상과 다르게 Overflow가 발생할 수 있습니다.

이럴 때 다음 패턴이 유용합니다.

```css
grid-template-columns:
  250px minmax(0, 1fr);
```

또는 Grid Item에:

```css
.main {
  min-width: 0;
}
```

를 적용할 수 있습니다.

PART 7 Flexbox에서 배웠던 `min-width: 0` 문제와 연결되는 부분입니다.

---

# 57. Grid를 설계하는 사고 순서

Property를 무작정 외우기보다 다음 순서로 생각합니다.

```text
1
누가 Grid Container인가?
        ↓

2
누가 Grid Item인가?
        ↓

3
Column은 몇 개인가?
        ↓
grid-template-columns

4
Row를 명시적으로 정의해야 하는가?
        ↓
grid-template-rows

5
Track 크기는 어떻게 할 것인가?
        ↓
px / % / fr
minmax()

6
Track 사이 간격은?
        ↓
gap

7
Item이 몇 개의 Cell을 차지하는가?
        ↓
grid-column
grid-row

8
이름으로 영역을 설계할 것인가?
        ↓
grid-template-areas

9
Item 정렬은?
        ↓
justify-items
align-items

10
반응형인가?
        ↓
repeat()
auto-fit
minmax()
```

---

# 58. Grid Container Property 정리

대표적인 Grid Container Property:

```text
display

grid-template-columns
grid-template-rows

grid-template-areas

grid-auto-columns
grid-auto-rows
grid-auto-flow

gap
row-gap
column-gap

justify-items
align-items
place-items

justify-content
align-content
place-content
```

---

# 59. Grid Item Property 정리

대표적인 Grid Item Property:

```text
grid-column-start
grid-column-end
grid-column

grid-row-start
grid-row-end
grid-row

grid-area

justify-self
align-self
place-self
```

Container와 Item Property를 구분해서 이해하는 것이 중요합니다.

---

# 60. CSS Layout 전체 구조

지금까지 배운 내용을 연결해 보겠습니다.

```text
HTML Element
      ↓
CSS
      ↓
CSS Box
      ↓
Box Model
      ↓
Size & Units
      ↓
display
      ↓
Normal Flow
      │
      ├───────────────┐
      │               │
      ▼               ▼
Positioning        Layout Systems
                      │
                 ┌────┴────┐
                 │         │
                 ▼         ▼
              Flexbox     Grid
                 │         │
                 ▼         ▼
             1차원 중심   2차원 중심
```

이제 CSS Layout의 핵심 구조가 완성됩니다.

---

# 61. PART 7과 PART 8 연결

PART 7:

```text
Flex Container
      ↓
Main Axis
Cross Axis
      ↓
Flex Items
      ↓
정렬 / 공간 분배
```

PART 8:

```text
Grid Container
      ↓
Rows + Columns
      ↓
Grid Tracks
      ↓
Grid Cells
      ↓
Grid Items 배치
```

따라서:

```text
Flexbox
→ Item 중심 사고

Grid
→ Track 중심 사고
```

라고 구분하면 이해하기 쉽습니다.

---

# 62. PART 8 핵심 정리

CSS Grid는 **2차원 Layout System**입니다.

Grid 시작:

```css
.container {
  display: grid;
}
```

Column 정의:

```css
grid-template-columns:
  repeat(3, 1fr);
```

Row 정의:

```css
grid-template-rows:
  100px 200px;
```

간격:

```css
gap: 20px;
```

Item 영역:

```css
grid-column: 1 / 3;
grid-row: 1 / 3;
```

반응형 Grid:

```css
grid-template-columns:
  repeat(
    auto-fit,
    minmax(250px, 1fr)
  );
```

Page Layout:

```css
grid-template-areas:
  "header header"
  "sidebar main"
  "footer footer";
```

Item 중앙 정렬:

```css
place-items: center;
```

---

# PART 8의 가장 중요한 한 문장

> **CSS Grid의 핵심은 Item을 하나씩 좌표에 배치하는 것이 아니라, 먼저 Row와 Column으로 Grid Track을 설계하고 그 구조 위에 Grid Item을 배치하는 것이다.**

그리고 Flexbox와 Grid의 차이는 다음 한 문장으로 기억하면 됩니다.

> **Flexbox는 한 축을 중심으로 Item들의 관계를 제어하고, Grid는 Row와 Column이라는 두 축을 이용해 Layout 구조 자체를 설계한다.**
