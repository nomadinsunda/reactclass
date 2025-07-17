## ✅ 1. 개발 시: `react-scripts start` → 내부 **개발 서버(dev server)** 사용

### ✅ 어떤 서버?

* `react-scripts` 내부의 **Webpack Dev Server**
* 디폴트로 **Node.js 기반 메모리 서버**
* 브라우저 자동 새로고침(HMR: Hot Module Replacement), 에러 오버레이, 프록시 설정 지원

```bash
npm start
```

🔽 실행 시:

* `localhost:3000`에서 React 앱이 서비스됨
* **정적 파일이 실제로 디스크에 쓰이지 않고 메모리에서 서빙됨**

---

## ✅ 2. 운영 환경(배포 시): 정적 파일을 웹 서버에서 호스팅

React 앱은 `npm run build`를 실행하면 다음처럼 정적 파일이 생성됩니다:

```
build/
├── index.html
├── static/
│   ├── js/
│   └── css/
```

이 정적 파일을 **서빙할 웹 서버**가 필요한데, 아래 중 하나를 선택합니다:

---

## ✅ 3. 운영 시 일반적으로 많이 쓰는 웹 서버

| 서버 종류                    | 설명 및 특징                                       |
| ------------------------ | --------------------------------------------- |
| **Nginx**                | 가장 많이 사용됨. 빠르고 안정적인 정적 웹 서버                   |
| **Apache HTTP Server**   | 전통적인 웹 서버. 설정 유연함                             |
| **Express.js (Node.js)** | 백엔드 API + 정적 리소스 서빙을 한 서버에서 구성 가능             |
| **Vercel**               | JAMStack + SPA 배포에 최적화된 플랫폼 (자동 HTTPS, 캐시 포함) |
| **Netlify**              | 정적 사이트 호스팅 전문. React 빌드 결과 바로 배포 가능           |
| **Firebase Hosting**     | Google 기반 서버리스 정적 호스팅. 빠름 + SSL 내장            |
| **GitHub Pages**         | 간단한 정적 파일 호스팅. custom domain도 설정 가능           |
| **S3 + CloudFront**      | AWS에서 SPA 정적 파일 서빙 시 사용 (엔터프라이즈 환경)           |

---

## ✅ Express.js 서버에서 호스팅하는 예시

```js
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'build'))); // build/ 정적 파일 제공

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'build', 'index.html')); // SPA 라우팅 대응
});

app.listen(8080, () => console.log('Server on http://localhost:8080'));
```

---

## ✅ 요약 정리

| 상황                 | 웹 서버                                                              |
| ------------------ | ----------------------------------------------------------------- |
| 개발 시 (`npm start`) | Webpack Dev Server (Node.js)                                      |
| 운영 시 (배포 후)        | Nginx, Express, Firebase Hosting, Netlify, Vercel, GitHub Pages 등 |
| SPA 라우팅 대응         | 대부분의 서버에서 `index.html`로 fallback 설정 필요 (`rewrite`)                |

---

## ✅ 보너스: 운영 시 Nginx 설정 예시

```nginx
server {
  listen 80;
  server_name example.com;

  root /var/www/html/build;
  index index.html;

  location / {
    try_files $uri /index.html;  # SPA 라우팅 대응
  }
}
```



## ✅ `npm start`의 정체

```bash
npm start
```

➡️ 이는 내부적으로 다음을 실행합니다:

```bash
react-scripts start
```

➡️ 그리고 `react-scripts start`는 **Webpack Dev Server**를 실행합니다.

---

## ✅ Webpack Dev Server의 역할

| 역할             | 설명                                            |
| -------------- | --------------------------------------------- |
| 🛰️ HTTP 서버 역할 | React 앱을 브라우저에 서빙 (기본 포트: `localhost:3000`)   |
| 🔁 핫 리로딩(HMR)  | 파일을 저장하면 브라우저가 자동 새로고침됨                       |
| 🧠 메모리 기반 번들   | 실제 디스크에 쓰지 않고 메모리에서 번들 실행                     |
| 🔄 프록시 처리      | `package.json`의 `"proxy"`를 통해 백엔드 요청을 포워딩     |
| 📦 번들링 자동 수행   | Webpack + Babel로 JSX, ES6 등을 브라우저가 이해할 코드로 변환 |

---

## ✅ 흐름 정리

```
npm start
↓
react-scripts start
↓
Webpack Dev Server 실행
↓
브라우저 열림 (http://localhost:3000)
↓
React SPA가 메모리 번들로 실시간 제공됨
```

---

## ✅ 결과

> React 기반 SPA가 **Webpack Dev Server에 의해 자동으로 실행되고**,
> 브라우저가 해당 앱을 **localhost:3000**에서 로딩하게 되는 것입니다.
