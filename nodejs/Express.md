## 1. Express란 무엇인가? 🤔

### 1.1 한 줄 정의

> **Express는 Node.js 위에서 동작하는 최소주의(Minimalist) 웹 애플리케이션 프레임워크**입니다.

* **HTTP 서버를 쉽게 만들도록 도와주는 라이브러리**
* 라우팅, 미들웨어, 요청/응답 처리 등 **웹 서버에 필요한 기본 기능을 제공**
* 그 외 나머지(ORM, 템플릿 엔진, 인증 등)는 **개발자가 필요한 것을 골라 붙이는 구조**

즉, Spring Boot처럼 **“올인원 프레임워크”** 가 아니라,
**“얇은 웹 서버 레이어 + 자유롭게 조합하는 생태계”** 라고 보시면 이해가 쉽습니다.

---

## 2. 왜 Express가 Node.js 세계에서 그렇게 자주 등장할까? 🌍

Node.js만으로도 `http` 모듈을 사용해 HTTP 서버를 만들 수 있습니다:

```js
// 순수 Node.js http 서버 예시
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello World");
  } else if (req.url === "/users" && req.method === "GET") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify([{ id: 1, name: "Alice" }]));
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("Not Found");
  }
});

server.listen(3000);
```

이 정도는 괜찮지만, 기능이 조금만 늘어나면 바로 지옥이 시작됩니다. 🔥

* URL, 메서드에 따라 라우팅 분기 직접 구현해야 함
* 요청 바디(JSON, form-data) 파싱 직접 해야 함
* 쿠키, 세션, 인증, 로깅, 에러 처리 등도 직접 구현…

그래서 나온 것이 **Express**입니다.

Express를 사용하면 위 코드는 이렇게 바뀝니다:

```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.get("/users", (req, res) => {
  res.json([{ id: 1, name: "Alice" }]);
});

app.use((req, res) => {
  res.status(404).send("Not Found");
});

app.listen(3000, () => {
  console.log("Server listening on port 3000");
});
```

### 정리하자면, Express의 장점 💡

* **간결한 라우팅 API**: `app.get("/path", handler)`
* **미들웨어 기반 구조**: 로깅, 인증, 에러 처리 등 공통 로직을 깔끔하게 분리
* **풍부한 생태계**: 수많은 미들웨어/라이브러리와 잘 통합
* **프레임워크가 강제하는 것이 적다**: 자유도가 높음

---

## 3. Express의 핵심 개념 3가지 🧠

Express를 이해하려면 **이 세 가지 키워드**를 잡으시면 됩니다.

1. **Application 객체 (`app`)**
2. **라우팅 (Routing)**
3. **미들웨어 (Middleware)**

---

### 3.1 Application 객체 (`app`) – 서버의 본체 🏛️

```js
const express = require("express");
const app = express();
```

`app`은 Express 애플리케이션 인스턴스입니다. 주요 역할은:

* 라우트 등록: `app.get`, `app.post`, `app.put`, ...
* 미들웨어 등록: `app.use(...)`
* 서버 시작: `app.listen(port, callback)`
* 전역 설정: `app.set("view engine", "ejs")` 같은 설정들

한마디로, **애플리케이션의 컨테이너이자 설정 허브**입니다.

---

### 3.2 라우팅 (Routing) – “어떤 URL에 어떤 코드를 실행할 것인가” 🗺️

라우팅은 단순히 말해서:

> **"특정 URL + HTTP 메서드(GET/POST/PUT/DELETE 등)에 대해 어떤 함수를 실행할 것인지"를 연결하는 작업**

```js
app.get("/users", (req, res) => {
  // GET /users 요청 처리
});

app.post("/users", (req, res) => {
  // POST /users 요청 처리
});

app.get("/users/:id", (req, res) => {
  // GET /users/10 같은 요청 처리
  const id = req.params.id;
  res.send(`User ID: ${id}`);
});
```

#### 라우트 핸들러의 시그니처

```js
(req, res, next) => { ... }
```

* `req`: 요청 정보 (URL, 헤더, 바디, 파라미터 등)
* `res`: 응답을 보내기 위한 객체
* `next`: 다음 미들웨어/핸들러로 제어를 넘기는 함수

---

### 3.3 미들웨어 (Middleware) – Express의 진짜 힘 💪

Express의 설계를 이해하는 핵심은 **“모든 것이 미들웨어”**라는 관점입니다.

> **미들웨어란?**
> 들어온 요청(`req`)과 나가는 응답(`res`) 사이에서
> **공통 기능을 수행하고, 필요하면 다음 단계로 넘기는 함수**입니다.

형식은 다음과 같습니다:

```js
function middleware(req, res, next) {
  // 공통 작업 수행 (로그, 인증, 바디 파싱 등)
  next(); // 다음 미들웨어로 제어 넘김
}
```

#### 전역 미들웨어 예시

```js
// 모든 요청에 대해 시간/메서드/URL 로깅
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});
```

#### 특정 경로에만 적용되는 미들웨어

```js
// /admin 아래 경로에만 인증 미들웨어 적용
function authMiddleware(req, res, next) {
  const isAuthenticated = false; // 예시

  if (!isAuthenticated) {
    return res.status(401).send("Unauthorized");
  }
  next();
}

app.use("/admin", authMiddleware);

app.get("/admin/dashboard", (req, res) => {
  res.send("Admin Dashboard");
});
```

#### 에러 처리 미들웨어

에러 처리 미들웨어는 **인자가 4개**입니다: `(err, req, res, next)`

```js
// 일반 라우트에서 에러 발생
app.get("/error", (req, res, next) => {
  try {
    throw new Error("Something went wrong");
  } catch (err) {
    next(err); // 에러를 에러 핸들러로 전달
  }
});

// 에러 처리 미들웨어
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send("Internal Server Error");
});
```

Express 내부적으로도:

* `body-parser`, `cookie-parser`, `express.json()` 같은 것들이 **전부 미들웨어**입니다.

---

## 4. 요청 → 응답까지의 흐름 (Lifecycle) 🔁

Express 서버에 요청이 들어오면 내부적으로는 대략 이런 흐름을 거칩니다.

1. Node.js의 `http` 서버가 소켓 연결 수락
2. 요청이 Express의 **미들웨어 체인**으로 들어감
3. `app.use(...)`로 등록된 미들웨어들이 순서대로 실행
4. 조건에 맞는 라우트 핸들러(`app.get("/path", ...)`) 실행
5. 응답이 `res.send` / `res.json` / `res.end` 등으로 클라이언트에게 전송
6. 만약 핸들러에서 에러가 발생하면, 에러 미들웨어가 처리

포인트는:

> **Express는 “미들웨어 체인 위에 라우팅을 올려 놓은 구조”** 라는 것

그래서 **미들웨어의 등록 순서가 매우 중요**합니다. 👀

---

## 5. 자주 쓰는 내장 미들웨어 & 유용한 기능들 🧰

### 5.1 JSON 바디 파싱

예전에는 `body-parser` 패키지를 따로 설치했지만,
지금은 Express에 내장되어 있습니다.

```js
app.use(express.json());          // JSON 요청 바디 파싱
app.use(express.urlencoded({      // x-www-form-urlencoded 파싱
  extended: true
}));
```

이걸 등록해 두면, 이후 라우트 핸들러에서:

```js
app.post("/users", (req, res) => {
  const { name, email } = req.body; // 여기에 파싱된 데이터가 들어옴
  res.json({ created: true, name, email });
});
```

---

### 5.2 정적 파일 제공 (Static Files)

프론트엔드 정적 자원(HTML, CSS, JS, 이미지)을 제공할 때 사용합니다.

```js
app.use(express.static("public"));
```

구조 예:

```
project/
  app.js
  public/
    index.html
    styles.css
    app.js
```

이렇게 설정하면:

* `GET /` → `public/index.html` (브라우저 기본 요청 처리 가능)
* `GET /styles.css` → `public/styles.css`
* `GET /app.js` → `public/app.js`

---

### 5.3 Router로 모듈화하기 (express.Router) 🧩

규모가 커지면 모든 라우트를 `app.js`에 때려 넣으면 관리가 힘들어집니다.
그래서 **Router**라는 개념을 사용해 **기능별 라우트 파일**로 분리합니다.

```js
// routes/userRoutes.js
const express = require("express");
const router = express.Router();

router.get("/", (req, res) => {
  res.send("User List");
});

router.post("/", (req, res) => {
  res.send("Create User");
});

router.get("/:id", (req, res) => {
  res.send(`User Detail: ${req.params.id}`);
});

module.exports = router;
```

```js
// app.js
const express = require("express");
const app = express();
const userRoutes = require("./routes/userRoutes");

app.use("/users", userRoutes); // /users로 시작하는 요청에 userRoutes 연결

app.listen(3000);
```

이렇게 하면:

* `GET /users` → User List
* `POST /users` → Create User
* `GET /users/10` → User Detail: 10

**장점**

* 기능/도메인 단위로 라우트 분리 (User, Article, Auth 등)
* 팀 개발 시 코드 충돌/관리 용이

---

## 6. 실무에서 자주 보게 되는 Express 패턴들 🧱

### 6.1 MVC-ish 패턴

Express 자체가 MVC 프레임워크는 아니지만, 다음과 같이 구조를 나누는 것이 일반적입니다.

```text
project/
  app.js
  routes/
    userRoutes.js
  controllers/
    userController.js
  models/
    userModel.js  // Mongoose, Sequelize, Prisma 등과 통합
```

* **Route**: URL → Controller 함수 연결
* **Controller**: 요청 처리 로직, 서비스 호출, 응답 구성
* **Model**: DB 연동 로직

### 6.2 미들웨어로 횡단 관심사 처리 (Cross-cutting Concerns)

* 로깅 미들웨어
* 인증/인가 미들웨어 (JWT, 세션)
* 요청 유효성 검사 (Validation)
* Rate limiting (DDOS, 과도한 호출 방지)
* 공통 에러 처리

이 모든 것을 **미들웨어 체인**에 걸어두고,
핵심 비즈니스 로직은 라우트/컨트롤러에서 깔끔하게 유지하는 방식입니다.

---

## 7. Express의 철학: “작고, 단순하고, 확장 가능하게” 🧬

Spring Boot, NestJS 같은 프레임워크에 비해 Express는 상당히 **얇고 단순**합니다.

* 디렉터리 구조 강제 ❌
* 의존성 주입 컨테이너 내장 ❌
* ORM 내장 ❌
* 인증/보안/유효성 검사 내장 ❌

대신:

* **필수적인 웹 서버 기능만** 제공
* 나머지는 **커뮤니티 생태계 패키지들로 조합**
* 자유도가 크지만, **아키텍처를 개발자가 직접 설계**해야 함

그래서:

* **학습곡선은 빠른 편**이지만
* 일정 규모 이상부터는 **개발자의 설계 능력이 품질을 결정**합니다.

---

## 8. 간단한 Express 샘플 프로젝트 구조 예시 🧪

마지막으로, 아주 작은 API 서버 예시를 하나 묶어서 보여드리겠습니다.

```text
project/
  app.js
  routes/
    userRoutes.js
  middlewares/
    logger.js
    errorHandler.js
```

```js
// middlewares/logger.js
module.exports = function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();
};
```

```js
// middlewares/errorHandler.js
module.exports = function errorHandler(err, req, res, next) {
  console.error("Error:", err.message);
  res.status(500).json({ error: "Internal Server Error" });
};
```

```js
// routes/userRoutes.js
const express = require("express");
const router = express.Router();

const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];

router.get("/", (req, res) => {
  res.json(users);
});

router.get("/:id", (req, res, next) => {
  const user = users.find(u => u.id === Number(req.params.id));
  if (!user) {
    return res.status(404).json({ error: "User Not Found" });
  }
  res.json(user);
});

module.exports = router;
```

```js
// app.js
const express = require("express");
const logger = require("./middlewares/logger");
const errorHandler = require("./middlewares/errorHandler");
const userRoutes = require("./routes/userRoutes");

const app = express();

// 공통 미들웨어
app.use(logger);
app.use(express.json());

// 라우트
app.use("/users", userRoutes);

// 404 처리
app.use((req, res) => {
  res.status(404).json({ error: "Not Found" });
});

// 에러 처리 미들웨어 (맨 마지막에)
app.use(errorHandler);

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

이 정도 구조면:

* 미들웨어 체인
* 라우트 모듈화
* 에러 처리
  를 모두 한 번에 체험해 볼 수 있습니다. 👍

---

## 9. 정리 ✨

Express는 Node.js 생태계에서 다음과 같은 이유로 **기본 레퍼런스**처럼 자주 언급됩니다.

* Node.js의 http 서버를 **현실적인 수준으로 사용 가능하게 만드는 최소 프레임워크**
* **미들웨어 체인** 기반의 매우 유연한 구조
* 라우팅, 바디 파싱, 정적 파일 제공 등 **웹 서버 기본 기능을 간결하게 제공**
* “아키텍처를 강제하지 않고, 개발자가 직접 설계해 조립하는 스타일”


