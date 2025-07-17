## 📌 Bootstrap이란?

> **Bootstrap**은 Twitter에서 개발한 **오픈소스 프론트엔드 UI 컴포넌트 프레임워크**입니다.
> HTML, CSS, JavaScript로 작성된 UI 요소들을 빠르게 개발할 수 있도록 도와줍니다.

* **개발자명**: Mark Otto, Jacob Thornton (Twitter)
* **최초 릴리스**: 2011년
* **현재 버전**: 5.x (우리는 5.3.2 사용 중)
* **라이선스**: MIT

---

## 🎯 핵심 목적

* 일관된 **UI 디자인 시스템** 제공
* **반응형 웹** 디자인 자동 지원 (media queries 포함)
* CSS로 기본 스타일이 이미 정의되어 있어, **빠르게 UI 개발 가능**
* JavaScript 기반의 **동적 컴포넌트**도 기본 제공 (예: 모달, 드롭다운)

---

## 🧱 구성 요소

### ✅ 1. Layout 시스템 (Grid System)

* 12칸의 **Flexbox 기반 그리드** 제공
* 반응형(responsive) 웹 구현 용이

```html
<div class="container">
  <div class="row">
    <div class="col-md-6">왼쪽</div>
    <div class="col-md-6">오른쪽</div>
  </div>
</div>
```

### ✅ 2. Component (버튼, 카드, 네비게이션 등)

* `btn`, `card`, `navbar` 같은 CSS 클래스가 제공됨

```html
<button class="btn btn-primary">제출</button>
```

### ✅ 3. Utilities

* 여백(margin/padding), 색상, 폰트, 정렬 등의 유틸리티 클래스

```html
<div class="mt-3 text-center text-danger">중앙 정렬된 빨간 텍스트</div>
```

### ✅ 4. JavaScript 플러그인

* Modal, Carousel, Tooltip 등은 **JavaScript로 동작**
* Bootstrap 5는 **jQuery에 의존하지 않음** (Bootstrap 4까지는 jQuery 필요)

```html
<!-- Modal 트리거 -->
<button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#myModal">열기</button>

<!-- Modal -->
<div class="modal fade" id="myModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">...</div>
  </div>
</div>
```

---

## ⚙️ 설치 방식

* CDN (정적 HTML에서 사용 시)
* npm (`npm install bootstrap`)
* yarn (`yarn add bootstrap`)
* 직접 다운로드 (css/js 파일 포함)

> 💡 `package.json`에 `"bootstrap": "^5.3.2"` 가 있다면 **npm install 시 자동으로 `node_modules/bootstrap` 폴더에 다운로드됨**

---

## 🔌 React와 Bootstrap 통합

### 1. `index.js` 또는 `App.js`에 CSS 임포트

```js
import 'bootstrap/dist/css/bootstrap.min.css';
```

### 2. 컴포넌트에서 바로 사용

```jsx
function MyButton() {
  return <button className="btn btn-success">Success 버튼</button>;
}
```

### 3. (선택) React-Bootstrap 라이브러리 사용 가능

* Bootstrap 컴포넌트를 React 방식으로 wrapping한 라이브러리
* `react-bootstrap` 설치 후 사용

```bash
npm install react-bootstrap
```

```jsx
import { Button } from 'react-bootstrap';

function MyComponent() {
  return <Button variant="danger">삭제</Button>;
}
```

---

## ✅ 결론

| 항목           | 내용                                                        |
| ------------ | --------------------------------------------------------- |
| 무엇인가?        | Twitter가 만든 UI 프레임워크                                      |
| 무엇을 제공?      | CSS, JS 기반의 UI 컴포넌트와 유틸리티                                 |
| 어떤 방식?       | Grid + Component + JavaScript 플러그인                        |
| React에서 어떻게? | `import 'bootstrap/dist/css/bootstrap.min.css'`로 바로 사용 가능 |
| 장점           | 빠른 UI 개발, 반응형 지원, 일관성 유지                                  |
| 단점           | 커스터마이징이 어렵고 무거울 수 있음                                      |



