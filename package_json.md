
`package.json`은 **Node.js 프로젝트의 메타데이터와 의존성 정보를 담고 있는 핵심 파일**입니다.
React, Express, Vue, Nest.js 등 **npm 기반 프로젝트라면 무조건 존재**합니다.

---

## ✅ 핵심 개념

| 항목      | 설명                          |
| ------- | --------------------------- |
| 프로젝트 정보 | 이름, 버전, 설명 등                |
| 스크립트    | `npm run`으로 실행할 수 있는 명령 정의  |
| 의존성     | 프로젝트가 사용하는 npm 패키지 목록       |
| 기타 설정   | 라이선스, 리포지토리 주소, 실행 엔트리포인트 등 |

---

## 📦 예시: package.json 기본 구조

```json
{
  "name": "my-react-app",
  "version": "1.0.0",
  "description": "나의 React 애플리케이션",
  "main": "index.js",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^7.0.0"
  },
  "keywords": ["react", "vite", "frontend"],
  "author": "홍길동",
  "license": "MIT"
}
```

---

## 🧠 주요 속성 설명

| 속성                | 의미                                          |
| ----------------- | ------------------------------------------- |
| `name`            | 프로젝트 이름 (npm 패키지 이름으로도 사용 가능)               |
| `version`         | 프로젝트 버전 (`semver` 형식 권장: major.minor.patch) |
| `scripts`         | 커맨드 라인 명령어. `npm run [스크립트명]` 으로 실행         |
| `dependencies`    | 운영 시 필요한 패키지 (예: `express`)                 |
| `devDependencies` | 개발 시만 필요한 패키지 (예: `jest`, `nodemon`)        |
| `main`            | 시작 파일 경로 (보통 `index.js`)                    |
| `license`         | 오픈소스 라이선스 종류 (예: MIT)                       |

---

## 🔧 생성 방법

터미널에서 아래 명령 실행:

```bash
npm init
```

또는 빠르게 생성:

```bash
npm init -y
```

> `-y`는 모든 디폴트 값을 자동으로 채워줍니다.

---

## 💡 실전 팁

* **패키지 설치 시 자동으로 업데이트됨**

  ```bash
  npm install axios
  ```

  → 자동으로 `dependencies`에 추가됨

* **npm 스크립트를 활용한 자동화**

```json
 {
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

---

## 📝 요약

| 항목  | 설명                                 |
| --- | ---------------------------------- |
| 용도  | Node.js 프로젝트의 메타데이터, 의존성, 스크립트를 관리 |
| 생성  | `npm init` 명령어로 생성                 |
| 필수성 | npm 기반 프로젝트의 핵심, 없으면 npm 명령어 실행 불가 |

