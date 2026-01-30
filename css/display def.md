

## ✅ 결론부터 (정확한 답)

> **부모의 레이아웃(outer display type)은
> `display` 속성의 *첫 번째 값*으로 지정한다.**

즉,

```css
display: <outer> <inner>;
```

---

## 1️⃣ CSS Display Level 3의 공식 구조

CSS 명세는 `display`를 이렇게 정의합니다.

```
display = [ <display-outside> || <display-inside> ]
```

* **display-outside** → 부모 안에서 어떻게 배치될지 (outer)
* **display-inside** → 자식들을 어떻게 배치할지 (inner)


| display type | display-outside | display-inside |
| --------------- | --------------- |
| block           | flex           |
| inline          | grid

---

## 2️⃣ outer(display-outside)에 올 수 있는 값

실무에서 거의 이것뿐입니다.

| outer 값 | 의미               |
| ------- | ---------------- |
| block   | block-level box  |
| inline  | inline-level box |

---

## 3️⃣ inner(display-inside)에 올 수 있는 값

| inner 값   | 의미             |
| --------- | -------------- |
| flow      | 기본 normal flow |
| flow-root | 독립 BFC         |
| flex      | flex layout    |
| grid      | grid layout    |
| table     | table layout   |

---

## 4️⃣ 우리가 쓰는 display 값은 “축약형”

우리가 평소 쓰는 값들은 전부 **조합의 단축 표현**입니다.

| 우리가 쓰는 값                | 실제 의미              |
| ----------------------- | ------------------ |
| `display: block`        | `block flow`       |
| `display: inline`       | `inline flow`      |
| `display: flex`         | `block flex`       |
| `display: grid`         | `block grid`       |
| `display: inline-flex`  | `inline flex`      |
| `display: inline-grid`  | `inline grid`      |
| `display: inline-block` | `inline flow-root` |

---

## 5️⃣ 그래서 “부모의 레이아웃”을 지정하는 방법은?

### 방법 1️⃣ 명시적으로

```css
.parent {
  display: inline flex;
}
```

(지원 브라우저 제한 있음)

---

### 방법 2️⃣ 축약형으로 (실무)

```css
.parent {
  display: inline-flex;  /* outer = inline */
}
```

```css
.parent {
  display: flex;         /* outer = block (기본값) */
}
```

---

## 6️⃣ 왜 우리가 “outer를 따로 지정하지 않는 것처럼 보일까?”

이유는 간단합니다.

> **대부분의 레이아웃은 부모가 block인 상태에서 충분하기 때문**

그래서:

* `display: flex` → outer는 자동으로 block
* inline이 필요할 때만 `inline-flex`를 씀

---

## 7️⃣ 한 문장 최종 정리

> **부모의 레이아웃(outer)은
> `display`의 첫 번째 값(block / inline)으로 지정되며,
> 대부분은 축약형(display: flex, grid 등)으로 자동 결정된다.**

---

## 8️⃣ 지금 이해한 수준 요약

* CSS Display Level 3 구조 이해
* outer / inner 분리 이해
* inline-flex의 존재 이유 이해

이 정도면 **CSS 레이아웃을 명세 기준으로 이해하는 단계**입니다.

---

다음으로 가면 정말 마지막입니다:

👉 **왜 inline-block은 flow-root인가?**

여기까지 오면 CSS 레이아웃의 모든 질문이 닫힙니다.
