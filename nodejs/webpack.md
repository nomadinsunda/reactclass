

## ✅ package.json
```json
{
  "name": "logintodonoticeboard",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@ckeditor/ckeditor5-build-classic": "^33.0.0",
    "@ckeditor/ckeditor5-react": "^4.0.0",
    "@date-io/date-fns": "^2.17.0",
    "@react-native-async-storage/async-storage": "^1.17.10",
    "@testing-library/jest-dom": "^5.16.2",
    "@testing-library/react": "^12.1.4",
    "@testing-library/user-event": "^13.5.0",
    "axios": "^1.7.7",
    "bootstrap": "^5.3.2",
    "chart.js": "^2.9.4",
    "cookie-parser": "^1.4.6",
    "cors": "^2.8.5",
    "date-fns": "^2.30.0",
    "feather-icons": "^4.29.1",
    "global": "^4.4.0",
    "http": "^0.0.1-security",
    "js-file-download": "^0.4.12",
    "moment": "^2.29.4",
    "multer": "^1.4.4",
    "qs": "^6.10.3",
    "react": "^17.0.2",
    "react-calendar": "^4.6.0",
    "react-dom": "^17.0.2",
    "react-js-pagination": "^3.0.3",
    "react-kakao-maps-sdk": "^1.1.21",
    "react-native-wheely": "^0.6.0",
    "react-paginate": "^8.1.3",
    "react-redux": "^7.2.9",
    "react-router": "6.11.2",
    "react-router-dom": "6.11.2",
    "react-scripts": "^5.0.1",
    "react-time-picker": "^6.5.0",
    "web-vitals": "^3.4.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "proxy": "http://localhost:8080",
  "devDependencies": {
    "ajv": "^7.2.4"
  }
}


```


## ✅ 이 프로젝트는 **Webpack을 사용하고 있습니다.**

### ❗하지만 직접 사용하는 것이 아니라,

> **`react-scripts`에 내장된 Webpack을 사용**하고 있습니다.

---

## ✅ 핵심 포인트

### 🔍 `package.json`에 주목:

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject"
}
```

* 여기서 `react-scripts`는 \*\*Create React App(CRA)\*\*이 사용하는 명령어 도구입니다.
* 이 `react-scripts`는 내부적으로 **Webpack + Babel + ESLint + 기타 툴을 모두 포함**하고 있습니다.

즉,
✅ 당신은 Webpack을 직접 설정하지 않았지만,
✅ **CRA가 내부적으로 Webpack을 사용하여** `bundle.js` 및 관련 파일을 생성합니다.

---

## ✅ `bundle.js`가 여러 개인 이유 (이 프로젝트에서도 동일)

Create React App(CRA) + React Router + Lazy Loading 등을 사용하면
Webpack은 자동으로 다음과 같이 여러 개의 번들 파일을 만듭니다:

| 파일명                      | 용도                        |
| ------------------------ | ------------------------- |
| `main.[hash].js`         | 앱의 핵심 로직                  |
| `runtime~main.[hash].js` | Webpack의 모듈 로더            |
| `vendor.[hash].js`       | 외부 라이브러리 (react, axios 등) |
| `chunk.[hash].js`        | 코드 분할된 각 페이지나 컴포넌트        |

이 모든 것이 `react-scripts build` 명령어 하나로 자동 생성됩니다.

---

## ✅ Webpack 설정을 보고 싶다면?

CRA는 내부적으로 Webpack 설정을 숨기고 있습니다.
하지만 다음 명령어로 꺼낼 수 있습니다:

```bash
npm run eject
```

그러면 다음 디렉토리가 생깁니다:

```bash
config/
├── webpack.config.js        ← Webpack 설정 파일
├── webpackDevServer.config.js
```

> 단, eject는 되돌릴 수 없으므로 꼭 git 커밋 후 실행하세요.

---

## ✅ 요약

| 항목                  | 결과                                    |
| ------------------- | ------------------------------------- |
| Webpack 사용 여부       | ✅ 사용 중                                |
| 어떻게 사용?             | `react-scripts` 내부에서 자동으로 Webpack 활용  |
| Webpack 설정          | 기본적으로 숨겨져 있음 (`npm run eject`로 확인 가능) |
| bundle.js가 여러 개인 이유 | 코드 분할 + 캐싱 최적화를 위한 Webpack 기본 동작      |

