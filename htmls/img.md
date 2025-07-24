
# 🖼️ `<img>` 태그: 이미지 삽입 태그

## ✅ 1. 정의

```html
<img src="path/to/image.jpg" alt="설명 텍스트">
```

* `<img>`는 HTML 문서에 **이미지를 삽입**하기 위한 **빈 요소 (void element)** 입니다.
* **닫는 태그가 없으며 self-closing(`<img />`)이 가능**합니다.
* 화면에 시각적 정보를 표시하거나, 링크, 버튼, 썸네일, 배너, 아이콘 등 다양한 UI 요소로 활용됩니다.

---

## 🔧 2. 주요 속성

| 속성                | 필수      | 설명                                                      |
| ----------------- | ------- | ------------------------------------------------------- |
| `src`             | ✅       | 이미지 경로 (URL 또는 경로). 웹에서 이미지 리소스를 가져오는 주소                |
| `alt`             | ✅ (접근성) | 이미지가 **로딩되지 않거나 시각장애인**을 위한 대체 텍스트                      |
| `width`, `height` | ❌       | 이미지의 렌더링 크기(px, %, auto 등)                              |
| `title`           | ❌       | 마우스를 올렸을 때 툴팁 텍스트                                       |
| `loading`         | ❌       | 지연 로딩 설정 (`lazy`, `eager`, `auto`)                      |
| `decoding`        | ❌       | 브라우저의 이미지 디코딩 전략 (`sync`, `async`, `auto`)              |
| `referrerpolicy`  | ❌       | 이미지 요청 시 referrer 헤더 전송 정책                              |
| `crossorigin`     | ❌       | 외부 이미지 요청 시 CORS 정책 적용 (`anonymous`, `use-credentials`) |

---

## 📌 3. 핵심 속성 설명

### ✅ `src` (source)

* 이미지를 불러올 **경로** 또는 **URL**
* 절대 경로, 상대 경로, CDN 주소, base64 등 다양하게 사용 가능

```html
<img src="/images/logo.png" />
<img src="https://cdn.example.com/banner.jpg" />
<img src="data:image/png;base64,iVBORw0K..." />
```

### ✅ `alt` (alternative text)

* **접근성 필수 요소**
  → 시각 장애인 사용자 또는 이미지 로딩 실패 시 대체 설명 제공
* SEO에도 긍정적인 영향
* 단순 장식용 이미지인 경우 `alt=""`로 설정 (의미 없는 이미지임을 명시)

```html
<img src="dog.jpg" alt="강아지가 잔디밭에서 뛰어노는 모습" />
<img src="icon.svg" alt="" /> <!-- purely decorative -->
```

---

### ✅ `width` / `height`

* HTML 레벨에서 이미지의 표시 크기를 정의
* 숫자만 입력하면 단위는 픽셀(px)
* CSS로 조절하는 것이 더 유연

```html
<img src="photo.jpg" width="300" height="200" />
```

---

### ✅ `loading` (Lazy Loading)

* 이미지가 **뷰포트에 들어올 때까지 로딩을 지연**
* 성능 최적화에 매우 중요 (특히 긴 페이지나 썸네일 갤러리 등)

| 값       | 설명                 |
| ------- | ------------------ |
| `lazy`  | 지연 로딩. 화면에 보일 때 로딩 |
| `eager` | 즉시 로딩 (기본)         |
| `auto`  | 브라우저가 판단           |

```html
<img src="profile.jpg" loading="lazy" />
```

---

### ✅ `decoding`

* 이미지 디코딩 방식 제어

| 값       | 설명                  |
| ------- | ------------------- |
| `async` | 렌더링 최적화 위해 비동기로 디코딩 |
| `sync`  | 디코딩이 끝날 때까지 기다림     |
| `auto`  | 브라우저가 자동 결정 (기본)    |

---

### ✅ `referrerpolicy`

* 이미지 요청 시 `Referer` 헤더를 어떻게 전송할지 제어

```html
<img src="https://cdn.example.com/img.jpg" referrerpolicy="no-referrer" />
```

---

### ✅ `crossorigin`

* 외부 도메인의 이미지를 사용할 때 **CORS** 설정

| 값                 | 설명                         |
| ----------------- | -------------------------- |
| `anonymous`       | 쿠키/인증 정보 없이 요청             |
| `use-credentials` | 쿠키 포함 요청 (서버가 CORS 허용해야 함) |

```html
<img src="https://example.com/image.png" crossorigin="anonymous" />
```

---

## 🧠 4. 렌더링 시 주의점

* 이미지는 **비동기적으로 로딩**됨
* `<img>`는 문서의 흐름에서 **레이아웃을 차지**함 → `display: inline` 기본 속성
* 콘텐츠 시프트(레이아웃 밀림) 방지를 위해 **width/height 또는 aspect-ratio 지정** 권장

---

## ♿ 5. 접근성과 SEO

| 항목                   | 권장          | 이유                     |
| -------------------- | ----------- | ---------------------- |
| `alt`                | ✅           | 스크린 리더와 SEO를 위해 반드시 제공 |
| `title`              | ❌           | 접근성 보조용이 아님. 툴팁용       |
| `aria-hidden="true"` | 장식용 이미지일 경우 | 스크린 리더 무시 처리           |

---

## 🧩 6. React에서의 `<img>` 태그

* React(JSX)에서는 일반 HTML과 유사하지만, 다음 차이가 있음:

```jsx
<img src={viteLogo} alt="Vite logo" className="logo" />
```

| 항목                  | 설명                                   |
| ------------------- | ------------------------------------ |
| `{viteLogo}`        | import된 이미지 변수 (Webpack, Vite에서 처리됨) |
| `className`         | HTML의 `class` 대신 사용                  |
| `onLoad`, `onError` | 이미지 로딩 상태 감지 가능                      |

---

## 🔐 7. 보안 관련 고려

* 외부 이미지가 트래킹에 사용될 수 있음 → `referrerpolicy`, `crossorigin` 설정 중요
* 이미지 URL이 신뢰할 수 있는 출처인지 확인 필요

---

## ✅ 요약 정리

| 항목        | 내용                                                                        |
| --------- | ------------------------------------------------------------------------- |
| 용도        | HTML 문서에 이미지를 표시                                                          |
| 필수 속성     | `src`, `alt`                                                              |
| 중요한 추가 속성 | `loading`, `decoding`, `width`, `height`, `referrerpolicy`, `crossorigin` |
| 접근성       | `alt` 필수, 장식용은 `alt=""`                                                   |
| 성능 최적화    | `loading="lazy"` 필수 고려                                                    |
| React와 연계 | JSX에서는 변수 사용, `className`, 이벤트 핸들러 사용 가능                                  |


