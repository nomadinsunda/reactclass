`package.json`은 Node.js 기반 프로젝트의 **설정 파일이자 프로젝트의 뇌**입니다.
npm이나 yarn은 이 파일을 보고 패키지를 설치하고, 프로젝트를 빌드하거나 실행합니다.


---

## ✅ 1. package.json 예제

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "나의 첫 Node.js 애플리케이션",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "keywords": ["node", "express", "api"],
  "author": "성원 서",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.7.7"
  },
  "devDependencies": {
    "nodemon": "^3.0.3"
  },
  "type": "module"
}
```

---

## ✅ 2. 항목별 해석

| 항목                  | 설명                                                   |
| ------------------- | ---------------------------------------------------- |
| `"name"`            | 패키지 이름. 소문자, 하이픈 사용 가능. npm 배포 시 고유해야 함              |
| `"version"`         | 버전 번호 (Semantic Versioning: `주.부.수` 예: `1.0.0`)      |
| `"description"`     | 프로젝트 설명. npm 사이트나 README로도 활용                        |
| `"main"`            | 진입점 파일. `require('my-app')` 시 참조되는 기본 파일             |
| `"scripts"`         | 커맨드라인 명령어 정의. `npm run start`처럼 실행 가능                |
| `"keywords"`        | 검색 키워드. npm 레지스트리에서 검색될 때 참조됨                        |
| `"author"`          | 작성자 이름                                               |
| `"license"`         | 라이선스 종류 (`MIT`, `ISC`, `Apache-2.0`, `UNLICENSED` 등) |
| `"dependencies"`    | 애플리케이션 실행에 **필수인 외부 패키지** 목록                         |
| `"devDependencies"` | 개발 중에만 필요한 패키지. 배포 시에는 포함 안 됨                        |
| `"type"`            | `commonjs`(기본값) 또는 `module` (ESM 사용 시 명시 필요)         |

---

## ✅ 3. Scripts 상세 예

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js",
  "test": "jest",
  "build": "webpack"
}
```

| 명령어             | 실행 결과              |
| --------------- | ------------------ |
| `npm run start` | `node index.js` 실행 |
| `npm run dev`   | `nodemon`으로 실시간 반영 |
| `npm run test`  | `jest` 테스트 수행      |
| `npm run build` | Webpack으로 번들링      |

👉 자주 쓰는 스크립트를 자동화할 수 있어 **빌드 도구처럼 사용 가능**

---

## ✅ 4. dependencies vs devDependencies

| 구분              | 예시                           | 설명                    |
| --------------- | ---------------------------- | --------------------- |
| dependencies    | `express`, `axios`           | 서버 실행 중 반드시 필요한 라이브러리 |
| devDependencies | `nodemon`, `jest`, `webpack` | 개발, 테스트, 빌드용 도구       |

> `npm install --save` → dependencies
> `npm install --save-dev` → devDependencies

---

## ✅ 5. 버전 표기법

| 표기         | 의미                                              |
| ---------- | ----------------------------------------------- |
| `"^1.2.3"` | **주버전 고정**, 부버전/수버전 자동 업그레이드 (`>=1.2.3 <2.0.0`) |
| `"~1.2.3"` | **부버전 고정**, 수버전만 업그레이드 (`>=1.2.3 <1.3.0`)       |
| `"1.2.3"`  | 정확히 그 버전만 사용                                    |
| `"*"`      | 어떤 버전이든 허용 (권장하지 않음)                            |

---

## ✅ 6. package.json 생성 방법

```bash
npm init        # 질문 기반으로 생성
npm init -y     # 기본값으로 빠르게 생성
```

---

## ✅ 7. 숨겨진 고급 항목들

| 항목                   | 설명                                      |
| -------------------- | --------------------------------------- |
| `"engines"`          | Node.js, npm 버전 요구사항 지정                 |
| `"peerDependencies"` | 다른 패키지의 특정 버전에 의존할 때 사용 (라이브러리 제작 시 중요) |
| `"files"`            | npm 배포 시 포함할 파일 목록 지정                   |
| `"private": true`    | npm에 **배포를 방지**하기 위한 안전장치               |

---

## ✅ 마무리 요약

| 목적         | package.json의 역할         |
| ---------- | ------------------------ |
| 📦 의존성 관리  | 어떤 패키지가 필요한지 자동 기록       |
| ⚙️ 프로젝트 설정 | 진입 파일, 실행 명령어, 버전 등 지정   |
| 🛠️ 개발 자동화 | `scripts`로 빌드/테스트 자동화 가능 |
| 🚀 배포 가능   | npm에 배포 시 사용될 메타데이터 저장소  |

