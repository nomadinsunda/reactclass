# HTTP Header 상세 설명 — CodeCrews

이 문서는 **frontend**(React 19 + Vite + RTK Query)와 **noticeBoard**(Spring Boot 3 / Spring Security 6) 두 프로젝트에서 실제로 **사용되거나 언급된 모든 HTTP header**를 다룬다.

각 header는 먼저 **한 줄 정의 사전**에서 전체 역할을 파악한 뒤, 상세 절에서 다음 네 가지를 분리해 읽도록 구성한다.

| 구분 | 뜻 |
| --- | --- |
| **스펙** | RFC 9110 / Fetch Standard가 정의한 원래 의미 |
| **방향·주체** | 요청/응답 중 어디에 실리며 브라우저·클라이언트·서버 중 누가 만드는가 |
| **이 프로젝트** | 어느 파일 어느 줄에서 어떻게 쓰이는가 (근거 있는 사실만) |
| **주의** | 시니어 관점에서 이 코드가 갖는 실제 문제 |

문서 끝에는 wire dump 수준의 시나리오별 흐름과, **현재 코드에 실제로 존재하는 header 관련 결함 목록**을 붙였다.

---

## 0. 먼저: 이 프로젝트의 요청 경로 지도

header를 이해하려면 요청이 어느 hop을 지나는지부터 알아야 한다. dev와 prod에서 header가 달라지는 이유가 전부 여기서 나온다.

### 개발 환경 (현재 설정)

```
브라우저 (http://localhost:5173)
   │
   │  ① same-origin 요청.  fetch('/api/posts')
   │     → CORS 없음. preflight 없음. Origin header는 POST/PUT 등에서만 붙음.
   ▼
Vite dev server (localhost:5173)   ← frontend/vite.config.js
   │
   │  ② proxy: '/api' → http://localhost:8080, changeOrigin: true
   │     changeOrigin 은 **Host** header 를 target(localhost:8080)으로 바꾼다.
   │     Origin header 는 그대로 전달된다.
   ▼
Spring Boot (localhost:8080)
   │
   ├─ CorsFilter                (@Order(HIGHEST_PRECEDENCE), 서블릿 필터)
   ├─ FilterExceptionHandler    (Security chain)
   ├─ SingleVisitInterceptor    (Security chain) — User-Agent 를 읽음
   ├─ JwtAuthenticationFilter   (Security chain) — Cookie 를 읽음
   └─ Controller                (@CookieValue accessToken)
```

핵심 결과: **개발 환경에서는 브라우저 입장에서 모든 API 호출이 same-origin이다.** 그래서 `CorsFilter`가 붙이는 `Access-Control-*` header는 브라우저가 아예 검사하지 않는다. 아래 4장의 CORS 결함들이 dev에서 안 터지는 이유가 이것이다.

### 운영 환경 (리버스 프록시를 쓰지 않을 경우)

```
브라우저 (https://app.example.com)
   │  ③ cross-origin.  → preflight(OPTIONS) + Origin header 필수
   │     쿠키를 보내려면 credentials: 'include' + SameSite=None; Secure
   ▼
Spring Boot (https://api.example.com)
```

`frontend/src/api/baseApi.js:5`의 주석이 "운영 리버스 프록시도 같은 규칙이어야 한다"고 못박은 것은, ②를 유지해서 same-origin을 그대로 가져가라는 뜻이다. 그렇게 하면 CORS 자체가 사라진다.

---

## 1. HTTP Header 기초

HTTP 메시지는 `start-line` + `header fields` + 빈 줄 + `body`로 이루어진다. header는 body를 **어떻게 해석할지**, **누가 보냈는지**, **인증 상태가 무엇인지**를 알려주는 메타데이터다.

```http
POST /logins HTTP/1.1
Host: localhost:8080
Origin: http://localhost:5173
Content-Type: application/json
Content-Length: 42
Cookie: accessToken=eyJhbGciOi...

{"id":"user1","password":"1234"}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: accessToken=eyJ...; Path=/; Max-Age=1800000; Secure; HttpOnly; SameSite=None
Set-Cookie: refreshToken=eyJ...; Path=/; Max-Age=1209600000; Secure; HttpOnly; SameSite=None

{"code":"...","message":"로그인에 성공했습니다."}
```

알아둘 규칙 세 가지:

1. **header 이름은 대소문자를 구분하지 않는다.** `Content-Type`과 `content-type`은 같다. HTTP/2·HTTP/3에서는 소문자로 전송된다.
2. **같은 이름의 header가 여러 번 나올 수 있다.** `Set-Cookie`가 대표적이며, 쿠키 하나당 한 줄이어야 한다. (`CookieSupport.setCookieFromJwt`가 `addHeader`를 세 번 호출하는 이유.)
3. **브라우저 JS가 직접 설정할 수 없는 header가 있다.** `Origin`, `Host`, `Cookie`, `Referer`, `Content-Length`, `Connection`, `Sec-*` 등은 [forbidden request header](https://fetch.spec.whatwg.org/#forbidden-request-header)라서 브라우저가 강제로 관리한다. 이것이 4.6절의 `Access-Control-Allow-Headers` 문제와 직결된다.

### 1.1 이 문서에 등장하는 HTTP Header 정의 사전

아래 표는 사용 내역을 보기 전에 반드시 알아야 할 **각 header의 정의**다.  
`요청`은 클라이언트가 서버에 보내는 header, `응답`은 서버가 클라이언트에 보내는 header다. `둘 다`는 요청과 응답에서 모두 사용할 수 있다는 뜻이다.

#### 표현과 콘텐츠

| Header | 방향 | 정의 | 대표 예 |
| --- | --- | --- | --- |
| `Content-Type` | 둘 다 | 메시지 body 또는 multipart 각 파트의 **데이터 형식(Media Type)**과 처리 방식을 알린다. | `application/json`, `multipart/form-data; boundary=...` |
| `Content-Length` | 둘 다 | 메시지 body의 크기를 **바이트(옥텟)** 단위로 나타낸다. | `Content-Length: 348` |
| `Content-Disposition` | 주로 응답·multipart 파트 | 콘텐츠를 브라우저에 바로 표시할지 다운로드할지, 또는 multipart 파트의 필드명·파일명을 설명한다. | `attachment; filename="report.pdf"` |
| `Accept` | 요청 | 클라이언트가 응답으로 받을 수 있거나 선호하는 미디어 타입을 서버에 알린다. | `Accept: application/json` |

`Content-Type`은 **지금 보내는 body의 형식**, `Accept`는 **돌려받고 싶은 응답의 형식**이다. 둘은 반대 방향의 요구를 표현한다.

#### CORS

| Header | 방향 | 정의 | 누가 사용하는가 |
| --- | --- | --- | --- |
| `Origin` | 요청 | 현재 요청을 시작한 문서의 origin(`scheme + host + port`)을 나타낸다. 경로는 포함하지 않는다. | 브라우저가 생성하고 서버가 허용 여부를 판단한다. |
| `Access-Control-Allow-Origin` | 응답 | 해당 응답을 JavaScript가 읽도록 허용할 origin을 지정한다. | 서버가 보내고 브라우저가 검사한다. |
| `Access-Control-Allow-Credentials` | 응답 | credentials mode가 `include`인 CORS 응답을 브라우저가 JavaScript에 공개해도 되는지 나타낸다. 유효한 값은 `true`다. | 서버가 보내고 브라우저가 검사한다. |
| `Access-Control-Allow-Methods` | preflight 응답 | 이어질 실제 CORS 요청에서 허용할 HTTP 메서드를 나열한다. | 서버가 보내고 브라우저가 검사한다. |
| `Access-Control-Allow-Headers` | preflight 응답 | 이어질 실제 CORS 요청에서 허용할 non-safelisted 요청 header 이름을 나열한다. | 서버가 보내고 브라우저가 검사한다. |
| `Access-Control-Max-Age` | preflight 응답 | preflight 허용 결과를 브라우저가 캐시할 수 있는 시간을 초 단위로 알린다. | 서버가 보내고 브라우저가 캐시한다. |
| `Access-Control-Request-Method` | preflight 요청 | 실제 요청에서 사용할 HTTP 메서드를 미리 알린다. | 브라우저가 자동 생성한다. |
| `Access-Control-Request-Headers` | preflight 요청 | 실제 요청에서 사용할 non-safelisted header 이름들을 미리 알린다. | 브라우저가 자동 생성한다. |
| `Vary` | 응답 | 나열된 요청 header의 값에 따라 응답 표현이 달라진다는 사실을 캐시에 알린다. | `Vary: Origin`은 origin별 CORS 응답의 캐시 혼용을 막는다. |

중요한 구분: CORS 응답 header는 서버 요청 자체를 차단하는 방화벽이 아니다. 브라우저가 **응답을 프론트 JavaScript에 공개할지** 판정하는 규칙이다.

#### 인증과 상태

| Header | 방향 | 정의 | 대표 예 |
| --- | --- | --- | --- |
| `Cookie` | 요청 | 브라우저가 현재 URL의 쿠키 규칙에 맞는 `name=value` 쌍을 서버로 보낸다. | `Cookie: accessToken=...; refreshToken=...` |
| `Set-Cookie` | 응답 | 서버가 브라우저에 쿠키의 생성·변경·삭제를 지시한다. | `Set-Cookie: accessToken=...; HttpOnly; Secure` |
| `Authorization` | 요청 | 선택한 인증 scheme과 credentials를 서버에 전달한다. | `Authorization: Bearer <token>` |
| `WWW-Authenticate` | 응답 | 서버가 클라이언트에 사용할 인증 scheme과 인증 범위를 알린다. 보통 `401`과 함께 사용한다. | `WWW-Authenticate: Bearer realm="api"` |

`Set-Cookie`는 **저장 지시**, `Cookie`는 저장된 쿠키의 **후속 전송**이다. `HttpOnly`, `Secure`, `SameSite`, `Path` 같은 속성은 `Set-Cookie`에만 있고 이후 `Cookie` 요청에는 포함되지 않는다.

#### 라우팅과 리다이렉트

| Header | 방향 | 정의 | 대표 예 |
| --- | --- | --- | --- |
| `Host` | 요청 | 클라이언트가 접속하려는 호스트와 포트를 나타낸다. HTTP/2·HTTP/3에서는 주로 `:authority`가 같은 역할을 한다. | `Host: api.example.com` |
| `Location` | 응답 | 리다이렉트할 URL 또는 새로 생성된 리소스의 URL을 알려준다. | `Location: /posts/42` |
| `Referer` | 요청 | 현재 요청을 유발한 이전 페이지의 주소를 전달한다. 노출 범위는 `Referrer-Policy`가 제어한다. | `Referer: https://app.example.com/posts` |
| `X-Forwarded-For` | 요청(프록시가 추가) | 프록시를 거치기 전 원래 클라이언트 IP의 전달 이력을 나타낸다. | `X-Forwarded-For: 203.0.113.7` |
| `X-Forwarded-Proto` | 요청(프록시가 추가) | 클라이언트가 프록시에 접속할 때 사용한 원래 scheme을 알린다. | `X-Forwarded-Proto: https` |
| `X-Forwarded-Host` | 요청(프록시가 추가) | 클라이언트가 요청한 원래 `Host` 값을 알린다. | `X-Forwarded-Host: app.example.com` |

`X-Forwarded-*`는 인터넷 클라이언트가 보낸 값을 그대로 신뢰하면 안 된다. **신뢰할 수 있는 리버스 프록시가 기존 값을 제거하거나 검증한 뒤 추가한다는 전제**에서만 사용한다.

#### WebSocket opening handshake

| Header | 방향 | 정의 |
| --- | --- | --- |
| `Upgrade` | 요청·응답 | 현재 HTTP 연결에서 다른 프로토콜로 전환할 것을 요청하거나 수락한다. WebSocket에서는 값이 `websocket`이다. |
| `Connection` | 요청·응답 | 현재 연결에만 적용할 connection option을 지정한다. WebSocket HTTP/1.1 handshake에서는 `Upgrade`를 지목한다. |
| `Sec-WebSocket-Key` | handshake 요청 | 클라이언트가 생성한 nonce를 base64로 보내 서버 응답 검증의 입력으로 사용한다. 인증 토큰은 아니다. |
| `Sec-WebSocket-Accept` | handshake 응답 | 서버가 `Sec-WebSocket-Key`를 규칙대로 처리했음을 증명해 WebSocket 전환을 수락한다. |
| `Sec-WebSocket-Version` | handshake 요청 | 클라이언트가 사용할 WebSocket 프로토콜 버전을 알린다. 현재 표준 버전은 `13`이다. |
| `Sec-WebSocket-Extensions` | handshake 요청·응답 | `permessage-deflate` 같은 WebSocket 확장 기능을 제안하고 협상한다. |
| `Sec-WebSocket-Protocol` | handshake 요청·응답 | 애플리케이션 subprotocol을 제안하고 선택한다. |

이 `Sec-WebSocket-*` header는 **WebSocket 연결을 시작하는 HTTP handshake에만** 속한다. 연결 후 STOMP frame 안에 나타나는 `destination`, `content-type`, `Authorization` 등은 HTTP header가 아니다.

#### 캐시·조건부 요청·진단·보안

| Header | 방향 | 정의 |
| --- | --- | --- |
| `User-Agent` | 요청 | 요청을 만든 클라이언트 소프트웨어의 식별 정보를 전달한다. 사용자가 바꿀 수 있으므로 신원 증명 수단은 아니다. |
| `X-Requested-With` | 요청 | 과거 Ajax 요청임을 표시하던 비표준 관습 header다. 보통 값은 `XMLHttpRequest`다. |
| `Cache-Control` | 둘 다 | 캐시 저장·재검증·유효 기간에 관한 지시자를 전달한다. |
| `ETag` | 응답 | 특정 표현 버전을 식별하는 opaque validator다. 반드시 파일의 MD5를 뜻하지는 않는다. |
| `If-None-Match` | 요청 | 보유한 `ETag`를 보내 현재 표현과 같으면 body 대신 `304`를 받도록 조건을 건다. |
| `Last-Modified` | 응답 | 서버가 알고 있는 리소스의 마지막 수정 시각을 알린다. |
| `If-Modified-Since` | 요청 | 지정 시각 이후 변경된 경우에만 표현을 보내 달라는 조건을 건다. |
| `Strict-Transport-Security` | 응답 | 일정 기간 해당 호스트에는 HTTPS로만 접속하도록 브라우저에 지시한다. |
| `X-Content-Type-Options` | 응답 | `nosniff` 값으로 선언된 MIME 타입을 브라우저가 임의로 추측하지 못하게 한다. |
| `X-Frame-Options` | 응답 | 다른 문서의 frame 안에 현재 문서를 표시할 수 있는 범위를 제한한다. |
| `Content-Security-Policy` | 응답 | 스크립트·스타일·이미지·frame 등 리소스 로드와 실행이 허용되는 출처를 정책으로 제한한다. |
| `Referrer-Policy` | 응답 | 이후 요청의 `Referer`에 어느 수준의 정보를 포함할지 정한다. |
| `Cross-Origin-Opener-Policy` | 응답 | top-level 문서의 browsing context group 분리를 제어해 다른 origin 창과의 참조 관계를 제한한다. |

이제부터의 인벤토리와 상세 절은 위 정의를 프로젝트 코드에 대입해 **어디에서 생성되고, 어디에서 읽히며, 현재 구현에 어떤 문제가 있는지** 설명한다.

---

## 2. 전체 Header 인벤토리

이 프로젝트에서 **명시적으로 코드에 등장하는** header와, **동작상 반드시 흐르지만 코드에 나타나지 않는** header를 나눠서 정리했다.

### 2.1 코드에 명시적으로 등장하는 header

| Header | 방향 | 위치 | 값 |
| --- | --- | --- | --- |
| `Access-Control-Allow-Origin` | 응답 | `cors/CorsFilter.java:26` | `${client.url}` = `http://localhost:5173` |
| `Access-Control-Allow-Credentials` | 응답 | `cors/CorsFilter.java:27` | `true` |
| `Access-Control-Allow-Methods` | 응답 | `cors/CorsFilter.java:28` | `*` ⚠️ |
| `Access-Control-Max-Age` | 응답 | `cors/CorsFilter.java:29` | `3600` |
| `Access-Control-Allow-Headers` | 응답 | `cors/CorsFilter.java:30` | `Origin, X-Requested-With, Content-Type, Accept, Authorization` |
| `Set-Cookie` | 응답 | `CookieSupport.java:40,41,52`, `JwtService.java:53` | accessToken / refreshToken / JSESSIONID 삭제 |
| `Cookie` | 요청 | `JwtAuthenticationFilter.java:49`, `JwtService.java:66`, `JwtHandshakeInterceptor.java:26`, `AuthHandshakeInterceptor.java:33`, 각 컨트롤러 `@CookieValue` (28곳) | `accessToken=…; refreshToken=…` |
| `User-Agent` | 요청 | `SingleVisitInterceptor.java:28` | Redis 방문자 집계 키의 값 |
| `Content-Type` | 요청·응답 | `FilterExceptionHandler.java:58,69`, `S3Service.java:37,52`, `PostController.java:25`, `MeetingController.java:31` | `application/json`, `multipart/form-data`, 업로드 파일의 MIME |
| `Authorization` | 요청 | `CustomOAuth2UserService.java:71` (아웃바운드), `JwtChannelInterceptor.java:23` (STOMP native) | `Bearer <token>` |
| `Accept` | 요청 | `CustomOAuth2UserService.java:72` (아웃바운드) | `application/json` |
| `Content-Length` | 응답(S3 오브젝트 메타데이터) | `S3Service.java:38,53` | 업로드 파일 바이트 수 |
| `Origin` | — | `CorsFilter.java:30`에서 **이름만** 언급 | (브라우저가 자동 생성) |
| `X-Requested-With` | — | `CorsFilter.java:30`에서 **이름만** 언급 | 실제로는 아무도 보내지 않음 |

### 2.2 코드에는 없지만 반드시 흐르는 header

| Header | 언제 | 누가 만드는가 |
| --- | --- | --- |
| `Host` | 모든 요청 | 브라우저 → Vite proxy가 `changeOrigin: true`로 재작성 |
| `Origin` | cross-origin 요청, 그리고 same-origin의 POST/PUT/PATCH/DELETE | 브라우저 |
| `Content-Type: multipart/form-data; boundary=…` | 파일 업로드 | 브라우저 (`FormData` 사용 시 자동, `lib/file.js:39-40`) |
| `Location` | OAuth2 리다이렉트, 302 응답 | `OAuth2AuthenticationSuccessHandler`, `CustomAuthenticationFailureHandler`의 `sendRedirect` |
| `Upgrade` / `Connection` / `Sec-WebSocket-*` | WebSocket handshake | 브라우저 (SockJS가 WebSocket transport를 고를 때) |
| `Access-Control-Request-Method` / `Access-Control-Request-Headers` | preflight(OPTIONS) | 브라우저 |
| `Referer` | 링크 이동, 리다이렉트 | 브라우저 |
| `Cache-Control` / `ETag` / `Last-Modified` | 정적 자산, S3 응답 | Vite dev server, S3 |
| `Vary` | 이상적으로는 CORS 응답에 | **현재 아무도 설정하지 않음** ⚠️ |

### 2.3 HTTP header가 아닌 것 (헷갈리기 쉬움)

| 항목 | 실체 |
| --- | --- |
| STOMP `destination`, `content-type`, `message`, `subscription` | **STOMP frame header.** WebSocket payload 안에 들어가는 텍스트지 HTTP header가 아니다. |
| `JwtChannelInterceptor`의 `Authorization` | STOMP CONNECT frame의 **native header**. HTTP `Authorization`과 이름만 같다. |
| `MailUtil`의 `<meta http-equiv="Content-Type" ...>` | HTML 메타 태그. SMTP로 나가는 메일 본문 안이라 HTTP와 무관. |
| Kafka `group-id: noticeboard-chat-group` | Kafka consumer 설정. |

---

## 3. 표현(Representation) Header

### 3.1 `Content-Type`

**스펙.** body의 미디어 타입을 나타낸다. `type/subtype; parameter=value` 형식이며, `charset`과 multipart의 `boundary`가 대표적인 파라미터다. body가 없는 메시지(GET 요청, 204 응답)에는 붙지 않는다.

**이 프로젝트.** 세 가지 방향으로 쓰인다.

**(1) 서버 → 클라이언트, 에러/성공 봉투**

```java
// common/exception/FilterExceptionHandler.java:56-58
response.setStatus(status);
response.setCharacterEncoding("utf-8");
response.setContentType(MediaType.APPLICATION_JSON_VALUE);   // "application/json"
```

`setCharacterEncoding("utf-8")`을 `setContentType` **앞에** 호출했기 때문에 최종 응답은 `Content-Type: application/json;charset=utf-8`이 된다. 순서가 반대였다면 `setContentType`이 인코딩을 덮어써서 한글 메시지가 깨졌을 것이다. 이 순서는 우연이 아니라 서블릿 스펙상 지켜야 하는 규칙이다.

**(2) 클라이언트 → 서버, JSON vs multipart 이중 수용**

```java
// post/post/controller/PostController.java:25
@PostMapping(consumes = {MediaType.APPLICATION_JSON_VALUE, MediaType.MULTIPART_FORM_DATA_VALUE})
public ResponseEntity<ResponseMessage> addPost(
        @RequestPart PostRequest postRequest,
        @RequestPart(required = false) List<MultipartFile> multipartFiles,
        @CookieValue String accessToken)
```

`consumes`는 **`Content-Type` 기반 라우팅 조건**이다. 요청의 `Content-Type`이 둘 중 하나가 아니면 Spring이 컨트롤러를 호출하지 않고 `415 Unsupported Media Type`을 낸다.

`@RequestPart`는 각 파트의 `Content-Type`을 따로 본다. 프론트가 이걸 정확히 맞추고 있다.

```js
// frontend/src/lib/file.js:38-40
const form = new FormData()
form.append(jsonPartName, new Blob([JSON.stringify(jsonPayload)], { type: 'application/json' }))
```

`Blob`에 `type: 'application/json'`을 준 것이 핵심이다. 그냥 문자열을 `append`하면 그 파트의 `Content-Type`이 `text/plain`이 되고, `@RequestPart PostRequest`가 JSON으로 역직렬화하지 못해 415가 난다. 실제 전송되는 바디는 이런 모양이다.

```http
POST /posts HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryAbC123

------WebKitFormBoundaryAbC123
Content-Disposition: form-data; name="postRequest"; filename="blob"
Content-Type: application/json

{"title":"제목","content":"내용"}
------WebKitFormBoundaryAbC123
Content-Disposition: form-data; name="multipartFiles"; filename="report.pdf"
Content-Type: application/pdf

%PDF-1.4...
------WebKitFormBoundaryAbC123--
```

**(3) 서버 → S3, 오브젝트 메타데이터**

```java
// s3/service/S3Service.java:36-39
ObjectMetadata metadata = new ObjectMetadata();
metadata.setContentType(file.getContentType());       // 브라우저가 준 값을 그대로 신뢰
metadata.setContentLength(file.getInputStream().available());
amazonS3Client.putObject(bucket, uuid, file.getInputStream(), metadata);
```

여기 저장한 `Content-Type`이 나중에 S3가 그 오브젝트를 서빙할 때 응답 `Content-Type`이 된다.

**주의.**

- **`boundary`를 직접 만들지 마라.** RTK Query의 `fetchBaseQuery`는 `body`가 `FormData`면 `Content-Type`을 **설정하지 않고 브라우저에 맡긴다.** 브라우저만이 실제 `boundary` 값을 알기 때문이다. 코드에서 `headers: { 'Content-Type': 'multipart/form-data' }`를 손으로 붙이면 boundary가 빠져서 서버가 파싱에 실패한다. 현재 코드는 어디에서도 `Content-Type`을 손으로 설정하지 않는다 — 이게 맞다.
- **`file.getContentType()`은 사용자 입력이다.** `.exe`를 `image/png`로 위장해서 올릴 수 있다. S3가 그 값 그대로 서빙하므로, 사용자가 올린 HTML/SVG가 버킷 도메인에서 실행될 수 있다. 매직 넘버 검증 또는 `Content-Type: application/octet-stream` 강제 + `Content-Disposition: attachment`가 정석이다.
- `S3Service.isValidFile`의 조건 `if(!extension.equals("exe") || !extension.equals("bat"))`은 **항상 참**이다(어떤 문자열도 두 값과 동시에 같을 수 없으므로 둘 중 하나는 반드시 `!equals` = true). 즉 `uploadFileToS3`는 모든 파일을 `INVALID_FILE`로 거절한다. 실질 방어선은 프론트의 `lib/file.js` 뿐이다.

### 3.2 `Content-Length`

**스펙.** body의 옥텟 수. `Transfer-Encoding: chunked`와는 상호 배타적이다.

**이 프로젝트.** `S3Service.java:38,53`에서 S3 오브젝트 메타데이터로 설정한다. HTTP 응답의 `Content-Length`는 Tomcat이 자동 계산한다.

**주의.** `file.getInputStream().available()`은 **파일 크기를 보장하지 않는다.** `available()`은 "블로킹 없이 읽을 수 있는 바이트 수"의 추정치다. Spring의 `MultipartFile`은 임계값을 넘으면 디스크에 스풀하는데, 이때 `available()`이 실제 크기와 다를 수 있다. 정확한 값은 `file.getSize()`다. 값이 어긋나면 S3가 업로드를 잘라내거나 거부한다.

### 3.3 `Content-Disposition`

**스펙.** 두 가지 문맥에서 쓰인다.

- multipart body의 각 파트: `Content-Disposition: form-data; name="field"; filename="a.pdf"`
- 응답 전체: `Content-Disposition: attachment; filename="a.pdf"` — 브라우저에게 인라인 렌더링 대신 다운로드하라고 지시

**이 프로젝트.** 코드에 직접 등장하지 않는다. multipart 파트의 `Content-Disposition`은 브라우저의 `FormData`가 자동 생성한다. **응답 쪽 `Content-Disposition`은 아무 데도 없다.**

첨부파일 다운로드는 서버 header 대신 순수 클라이언트 기법으로 구현되어 있다.

```js
// frontend/src/features/post/postApi.js:113-130
downloadAttachment: builder.mutation({
  async queryFn({ url, fileName }) {
    const response = await fetch(url)              // S3 URL, credentials 없이
    const objectUrl = URL.createObjectURL(await response.blob())
    const anchor = document.createElement('a')
    anchor.href = objectUrl
    anchor.download = fileName                     // ← 여기서 파일명이 결정된다
    anchor.click()
    URL.revokeObjectURL(objectUrl)
  },
})
```

**이 구현은 의도적으로 옳다.** `<a download>` 속성은 **cross-origin URL에서는 브라우저가 무시한다.** S3 URL을 그대로 `href`에 넣었다면 다운로드 대신 브라우저 탭에서 열려버리고 파일명도 UUID가 됐을 것이다. 먼저 `fetch` → `blob` → `blob:` object URL로 바꾸면 same-origin이 되어 `download` 속성이 살아난다. `frontend/CLAUDE.md:119`가 이 엔드포인트만 `queryFn`으로 baseQuery를 우회하는 이유도 같은 맥락이다 — `/api` 접두사와 `credentials: 'include'`가 외부 origin에 붙으면 안 되기 때문.

**주의.** 이 방식은 **S3 버킷에 CORS 설정이 있어야만 동작한다.** `fetch(url)` + `.blob()`은 응답 body를 읽는 것이므로 S3가 `Access-Control-Allow-Origin`을 내려주지 않으면 CORS 에러로 실패한다. 필요한 버킷 CORS 규칙:

```json
[{
  "AllowedOrigins": ["https://app.example.com"],
  "AllowedMethods": ["GET"],
  "AllowedHeaders": [],
  "ExposeHeaders": ["Content-Length", "Content-Type"],
  "MaxAgeSeconds": 3600
}]
```

`credentials`를 안 쓰므로 `AllowedOrigins: ["*"]`도 가능하다.

### 3.4 `Accept`

**스펙.** 클라이언트가 받고 싶은 미디어 타입. `q` 파라미터로 선호도를 매긴다. 서버는 이걸 근거로 content negotiation을 한다.

**이 프로젝트.** 유일한 명시적 사용은 **아웃바운드**다.

```java
// oauth/service/CustomOAuth2UserService.java:70-72
HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth(accessToken.getTokenValue());
headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
```

Spring 서버가 **클라이언트가 되어** naver/google/kakao의 UserInfo 엔드포인트를 호출할 때 붙인다. 프론트 → 백엔드 방향에서는 `fetch`가 디폴트값 `*/*`를 보내고, 백엔드는 `@RestController`라 항상 JSON으로만 응답하므로 negotiation이 일어나지 않는다.

`Accept`가 `CorsFilter`의 `Access-Control-Allow-Headers` 목록에 들어 있지만, 이건 CORS-safelisted header라서 나열할 필요가 없다(4.6절 참고).

> HTML의 `<input accept="image/png,image/jpeg">`(`RegisterPage.jsx:244`, `MeetingEditorPage.jsx:160`, `ProfilePage.jsx:152`)는 **HTTP `Accept` header가 아니다.** 파일 선택 다이얼로그의 필터일 뿐이고, 사용자가 "모든 파일"로 바꾸면 우회된다. 실제 검증은 `lib/file.js`의 `validateImage`가 한다.

---

## 4. CORS Header

CORS(Cross-Origin Resource Sharing)는 브라우저가 다른 origin의 응답을 JS에 넘겨줄지 결정하는 규칙이다. **origin = scheme + host + port** 삼중항이며, 하나라도 다르면 cross-origin이다.

이 프로젝트의 CORS는 전부 한 파일에 있다.

```java
// cors/CorsFilter.java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class CorsFilter implements Filter {
    @Value("${client.url}") private String clientUrl;   // http://localhost:5173

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
        response.setHeader("Access-Control-Allow-Origin", clientUrl);
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setHeader("Access-Control-Allow-Methods", "*");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers",
                "Origin, X-Requested-With, Content-Type, Accept, Authorization");

        if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);   // 여기서 끊는다
        } else {
            chain.doFilter(req, res);
        }
    }
}
```

`@Order(Ordered.HIGHEST_PRECEDENCE)`라서 Spring Security 필터 체인보다 **먼저** 실행된다. 이건 의도적으로 옳다 — preflight OPTIONS는 인증 정보(쿠키)를 싣지 않으므로, Security가 먼저 보면 401을 내고 CORS가 깨진다.

`SecurityConfig`는 `.cors()`를 호출하지 않는다. 즉 Spring Security의 `CorsConfigurationSource` 경로는 아예 쓰이지 않고, 이 필터가 CORS의 전부다.

### 4.1 `Origin`

**스펙.** 요청을 보낸 문서의 origin. **scheme + host + port만** 담고 경로는 담지 않는다. 브라우저가 강제로 붙이며 JS가 위조할 수 없다(forbidden header).

붙는 경우:

- 모든 cross-origin fetch/XHR
- **same-origin이라도 POST / PUT / PATCH / DELETE** (Chrome 기준)
- `<form>` 제출, WebSocket handshake
- GET / HEAD same-origin에는 붙지 않음

`Origin: null`이 되는 경우도 있다 — `file://`, sandboxed iframe, 일부 리다이렉트. 문자열 `"null"`을 유효한 origin으로 화이트리스트에 넣으면 안 된다.

**이 프로젝트.** `CorsFilter.java:30`에서 이름만 언급된다. 서버 코드는 `request.getHeader("Origin")`을 **읽지 않는다.** 허용 origin이 `client.url` 설정값 하나로 고정되어 있기 때문이다.

`index.html`의 `<link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin />`에서 `crossorigin` 속성이 `Origin` header를 붙게 만든다. 폰트 파일은 CORS 모드로 받아야 하므로 이 속성이 필요하다.

**주의.** 허용 origin이 하나로 고정된 것은 **보안상 오히려 좋다.** 흔한 취약점인 "`Origin`을 그대로 에코백"(`response.setHeader("ACAO", request.getHeader("Origin"))`)을 피했다. 다만 스테이징·프리뷰 도메인이 늘어나면 화이트리스트 검증 로직이 필요해지고, 그 순간 `Vary: Origin`이 필수가 된다(4.9절).

### 4.2 `Access-Control-Allow-Origin` (ACAO)

**스펙.** 응답 header. 이 응답을 읽어도 되는 origin을 지정한다. 값은 정확히 하나의 origin이거나 `*`다. **콤마로 여러 개를 나열할 수 없다.**

**이 프로젝트.** `http://localhost:5173` 고정 (`CorsFilter.java:26` ← `application.yaml`의 `client.url`).

이 값은 `frontend/vite.config.js`의 `server.port: 5173` + `strictPort: true`와 짝을 이룬다. vite.config.js 주석이 명시하듯 "백엔드 application.yaml의 client.url과 반드시 같아야 한다". `strictPort: true`가 있어서, 5173이 점유되면 Vite가 5174로 조용히 넘어가는 대신 **실패한다.** OAuth 리다이렉트가 소리 없이 깨지는 사고를 막는 좋은 설정이다.

**주의.** `Access-Control-Allow-Credentials: true`와 함께 쓸 때 ACAO가 `*`면 브라우저가 응답을 **무조건 거부한다.** 현재는 구체적 origin이라 이 함정은 피했다.

### 4.3 `Access-Control-Allow-Credentials`

**스펙.** 값은 대소문자를 구분하는 문자열 `true`만 유효하다(`false`를 보내는 것과 header를 안 보내는 것이 같다). 요청에 쿠키를 실을지는 클라이언트의 credentials mode와 브라우저 쿠키 정책이 결정한다. 이 header는 credentials mode가 `include`인 CORS 응답을 브라우저가 JS에 공개하도록 허용하는 응답 측 조건이다.

**이 프로젝트.** `true` (`CorsFilter.java:27`). 프론트 쪽 짝은 이것이다.

```js
// frontend/src/api/baseApi.js:7-10
const rawBaseQuery = fetchBaseQuery({
  baseUrl: '/api',
  credentials: 'include',      // 인증이 HttpOnly 쿠키 기반이다.
})
```

**응답을 JavaScript에서 사용하려면 양쪽 설정이 맞아야 한다.** 클라이언트의 `credentials: 'include'`로 쿠키가 전송되더라도 서버에 이 header가 없으면 브라우저는 CORS 응답을 프론트 코드에 공개하지 않는다. 반대로 서버가 이 header를 보내도 클라이언트가 credentials를 포함하도록 요청하지 않았다면 그것만으로 쿠키가 전송되지는 않는다.

**주의 (중요).** `Access-Control-Allow-Credentials: true`는 **CSRF 노출과 직결된다.**

- `SecurityConfig`가 `.csrf().disable()`을 호출한다.
- 인증이 쿠키 기반이고 `SameSite=None`이다.
- 즉 공격자 사이트에서 `<form method="POST" action="https://api.example.com/posts">`를 제출하면 브라우저가 쿠키를 붙여서 보낸다.
- CORS는 **응답을 읽는 것**만 막는다. **요청이 도달해서 서버 상태를 바꾸는 것**은 막지 못한다.

`<form>` 제출은 preflight를 유발하지 않는 simple request이므로 CORS 검사조차 거치지 않는다. 실질 방어는 다음 중 하나여야 한다.

1. `SameSite=Lax` (dev/prod 모두 same-site로 배치 — 리버스 프록시 방식)
2. CSRF 토큰 (double-submit cookie)
3. 상태 변경 엔드포인트에서 `Origin` header 검증

현재 코드는 셋 다 없다.

### 4.4 `Access-Control-Allow-Methods` ⚠️

**스펙.** preflight 응답에서 허용 메서드를 나열한다. **`*`는 credentials가 없는 요청에서만 와일드카드로 동작한다.** credentials가 포함된 요청에서는 `*`가 "`*`라는 이름의 메서드"라는 **리터럴**로 해석되어, 사실상 아무 메서드도 허용하지 않는다.

> "It has this meaning only for requests without credentials... In requests with credentials, it is treated as the literal method name `*` without special semantics." — MDN

**이 프로젝트.** `*` (`CorsFilter.java:28`). 그런데 같은 필터가 `Access-Control-Allow-Credentials: true`를 함께 보낸다.

**이건 버그다.** 진짜 cross-origin 배포로 넘어가는 순간, `PUT /profiles`, `DELETE /posts/1`, `PATCH /password` 같은 preflight가 필요한 요청이 **전부 실패한다.** dev에서 안 터지는 유일한 이유는 Vite proxy가 same-origin으로 만들어 preflight 자체가 없기 때문이다.

수정:

```java
response.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE, OPTIONS");
```

### 4.5 `Access-Control-Max-Age`

**스펙.** preflight 결과를 캐시할 초 수. 같은 (origin, URL, 메서드, header 집합) 조합에 대해 이 시간 동안 OPTIONS를 생략한다.

**이 프로젝트.** `3600` (1시간, `CorsFilter.java:29`).

**주의.** 브라우저마다 상한이 있다. Chrome은 **최대 2시간(7200초)**, Firefox는 **24시간(86400초)**으로 클램프한다. 3600은 안전한 값이다.

캐시 무효화 수단이 없다는 점은 알아두자 — CORS 정책을 바꿔도 브라우저는 최대 1시간 동안 옛 결과를 쓴다. 정책 변경 배포 직후 원인 모를 CORS 에러가 나면 이걸 의심하면 된다.

### 4.6 `Access-Control-Allow-Headers` ⚠️

**스펙.** preflight 응답에서, 실제 요청이 보내도 되는 **커스텀 요청 header**를 나열한다. 다음 네 개는 **CORS-safelisted**라서 나열할 필요가 없다.

`Accept`, `Accept-Language`, `Content-Language`, `Content-Type`(값이 `text/plain` / `multipart/form-data` / `application/x-www-form-urlencoded`일 때만)

**이 프로젝트.** `Origin, X-Requested-With, Content-Type, Accept, Authorization` (`CorsFilter.java:30`)

다섯 개를 하나씩 뜯어보면:

| 값 | 판정 |
| --- | --- |
| `Origin` | **무의미.** forbidden request header라서 JS가 설정할 수 없고, preflight의 `Access-Control-Request-Headers`에 절대 나타나지 않는다. |
| `X-Requested-With` | **불필요.** 이 프로젝트의 어떤 코드도 이 header를 보내지 않는다. jQuery 시대의 잔재. |
| `Accept` | **불필요.** safelisted. |
| `Content-Type` | **필요.** `application/json`은 safelisted 값이 아니라서 preflight를 유발하고, 여기 나열되어야 한다. |
| `Authorization` | **현재는 불필요.** 프론트가 HttpOnly 쿠키만 쓰고 `Authorization` header를 보내지 않는다. STOMP의 `Authorization`은 HTTP header가 아니라 frame native header다. Bearer 방식으로 전환할 때를 위한 대비로만 유효. |

즉 실제로 필요한 값은 `Content-Type` 하나다. 나머지는 해롭진 않지만, 이 목록을 읽는 다음 개발자에게 "이 프로젝트는 `X-Requested-With`를 쓰는구나"라는 잘못된 인상을 준다.

### 4.7 `Access-Control-Request-Method` / `Access-Control-Request-Headers`

**스펙.** **preflight 요청**에만 붙는 요청 header. 브라우저가 "본 요청은 이 메서드와 이 header들을 쓸 건데 괜찮냐"고 묻는 것이다.

```http
OPTIONS /posts/1 HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: content-type
```

**이 프로젝트.** 코드에서 읽지 않는다. `CorsFilter`가 요청 내용과 무관하게 항상 같은 정적 응답을 준다. 그래서 4.4의 `*` 문제가 조용히 통과하는 것이다 — 필터가 `Access-Control-Request-Method: DELETE`를 검사하지 않으니 서버는 문제를 감지하지 못하고, 판정은 전적으로 브라우저 몫이 된다.

### 4.8 preflight가 언제 발생하는가

이 구분이 실무에서 제일 중요하다.

**simple request (preflight 없음):**

- 메서드가 `GET` / `HEAD` / `POST`
- 커스텀 header 없음 (safelisted만)
- `POST`의 `Content-Type`이 `application/x-www-form-urlencoded` / `multipart/form-data` / `text/plain`

**preflight 발생:**

- `PUT` / `PATCH` / `DELETE` 등
- `Content-Type: application/json` ← **RTK Query가 객체 `body`를 보낼 때 붙이는 값**
- 커스텀 header 사용

이 프로젝트의 API를 대입하면:

| 요청 | Content-Type | preflight? |
| --- | --- | --- |
| `GET /posts` | 없음 | ❌ |
| `POST /logins` (`authApi.js:13`) | `application/json` | ✅ |
| `POST /registers` (`authApi.js:23`, FormData) | `multipart/form-data` | ❌ (simple) |
| `PATCH /profile-image` (`profileApi.js:24`, FormData) | `multipart/form-data` | ✅ (PATCH이므로) |
| `PUT /profiles` (`profileApi.js:18`) | `application/json` | ✅ |
| `DELETE /posts/{id}` | 없음 | ✅ (DELETE이므로) |

즉 `POST /registers`를 제외한 거의 모든 쓰기 요청이 preflight를 탄다. 4.4의 버그가 cross-origin 배포에서 치명적인 이유다.

### 4.9 `Vary` — 없는 header ⚠️

**스펙.** 응답 header. "이 응답은 나열된 요청 header 값에 따라 달라진다"고 캐시에 알린다. `Origin`에 따라 ACAO를 다르게 주는 서버는 `Vary: Origin`이 필수다. 없으면 공유 캐시가 A origin용 응답을 B origin에 재사용해 CORS가 무작위로 깨진다.

**이 프로젝트.** 없다.

**주의.** 현재는 ACAO가 설정 상수 하나로 고정이라 **당장은 문제가 없다.** 하지만 화이트리스트 방식(요청 `Origin`을 검사해 에코백)으로 바꾸는 순간 `Vary: Origin`을 함께 넣어야 한다. 이건 지금 고칠 버그가 아니라 리팩터링 시 잊으면 안 되는 항목이다.

---

## 5. 인증·상태 Header

### 5.1 `Cookie`

**스펙.** 요청 header. `name=value` 쌍을 `; `로 이어 붙인다. **속성(`Path`, `Secure`, `HttpOnly`, `SameSite`)은 전송되지 않는다** — 그것들은 브라우저가 "언제 보낼지" 판단하는 데만 쓰고 서버로 넘어오지 않는다. 서버는 어떤 쿠키가 `HttpOnly`였는지 알 수 없다.

**이 프로젝트.** 인증의 전부다. 네 계층에서 읽는다.

**(1) 서블릿 필터 — 액세스 토큰 추출**

```java
// security/jwt/support/JwtAuthenticationFilter.java:47-57
public String getAccessTokenFromHeader(ServletRequest request) {
    Cookie cookies[] = ((HttpServletRequest) request).getCookies();
    if (cookies != null && cookies.length != 0) {
        return Arrays.stream(cookies)
                .filter(c -> c.getName().equals("accessToken")).findFirst()
                .map(Cookie::getValue).orElse(null);
    }
    return null;
}
```

메서드 이름이 `...FromHeader`지만 실제로는 `Cookie` header를 파싱한다. `Authorization`이 아니다.

**(2) 리프레시 흐름**

```java
// security/jwt/support/JwtAuthenticationFilter.java:32-42
if (token != null && jwtTokenProvider.validateAccessToken(token)) {
    SecurityContextHolder.getContext().setAuthentication(...);
} else if (token != null) {
    jwtService.validateRefreshToken(request, response);      // 새 accessToken 을 Set-Cookie
    setSuccessResponse(response, ResponseCode.CREATE_ACCESS_TOKEN);
    return;                                                   // ← 원래 요청은 처리하지 않는다
}
```

액세스 토큰이 만료되면 **원래 요청을 처리하지 않고** 새 쿠키만 심은 200 응답을 돌려준다. 프론트가 이 규약을 알고 재시도한다.

```js
// frontend/src/api/baseApi.js:25-28
let result = await rawBaseQuery(args, api, extraOptions)
if (result.data?.data === '4') {          // ResponseCode.CREATE_ACCESS_TOKEN
  result = await rawBaseQuery(args, api, extraOptions)   // 한 번만 재시도
}
```

이건 사실상 손으로 구현한 토큰 리프레시 인터셉터다. 동작은 하지만 두 가지 냄새가 있다. (a) 성공 봉투에 매직 문자열 `'4'`를 심어 프론트·백엔드가 결합됐다. `401` + `WWW-Authenticate`가 HTTP다운 표현이다. (b) 동시에 여러 요청이 나가면 각각 리프레시를 유발해 refresh token race가 생긴다. RTK Query 진영의 표준 해법은 `async-mutex`로 리프레시를 직렬화하는 것이다.

**(3) 컨트롤러 파라미터 — 28곳**

```java
// post/post/controller/PostController.java:27
@CookieValue String accessToken
```

`@CookieValue`는 디폴트가 `required = true`다. 쿠키가 없으면 Spring이 `MissingRequestCookieException` → **400 Bad Request**를 낸다. 인증 실패의 올바른 코드는 401인데 400이 나가므로, 프론트의 `STATUS_MESSAGE[400]`("요청 값을 확인해주세요.")이 뜬다. 실제로는 "로그인이 필요합니다"여야 한다.

또한 이건 **컨트롤러가 인증 컨텍스트를 직접 다룬다**는 뜻이다. `SecurityContextHolder`에 이미 `Authentication`이 채워져 있으므로 `@AuthenticationPrincipal`로 받는 게 정석이다. 28곳에 `@CookieValue String accessToken`이 반복되는 것은 인증 방식을 바꿀 때 28곳을 고쳐야 한다는 뜻이기도 하다.

**(4) WebSocket handshake**

```java
// meeting/chat/interceptor/JwtHandshakeInterceptor.java:24-40
Cookie[] cookies = req.getCookies();
for (Cookie c : cookies) {
    if ("accessToken".equals(c.getName())) {
        if (jwtTokenProvider.validateAccessToken(c.getValue())) {
            attributes.put("userId", jwtTokenProvider.getUserPk(c.getValue()));
            return true;
        }
    }
}
return false;   // handshake 거부
```

WebSocket handshake는 일반 HTTP GET으로 시작하므로 `Cookie` header가 그대로 실린다. 이것이 프론트가 STOMP `connectHeaders`를 쓰지 않고도 인증되는 이유다.

### 5.2 `Set-Cookie`

**스펙.** 응답 header. 쿠키 하나당 한 줄이어야 하며, 콤마로 합칠 수 없다.

```
Set-Cookie: <name>=<value>; Domain=<d>; Path=<p>; Max-Age=<s>; Expires=<date>; Secure; HttpOnly; SameSite=<Strict|Lax|None>
```

속성별 의미:

| 속성 | 의미 |
| --- | --- |
| `Domain` | 생략하면 **host-only** — 정확히 그 호스트에만 전송(서브도메인 제외). 지정하면 서브도메인 포함. |
| `Path` | 이 경로 접두사에만 전송. |
| `Max-Age` | **초** 단위 수명. `Expires`보다 우선. `0` 또는 음수면 즉시 삭제. |
| `Secure` | HTTPS에서만 전송. **단 `http://localhost`는 trustworthy origin으로 취급되어 예외.** |
| `HttpOnly` | `document.cookie`로 읽을 수 없음. XSS로 토큰 탈취 방어. |
| `SameSite=Strict` | cross-site 요청에 절대 안 붙음. |
| `SameSite=Lax` | 디폴트값. top-level GET 네비게이션에만 붙음. |
| `SameSite=None` | 항상 붙음. **`Secure` 필수.** |

**이 프로젝트.**

```java
// security/jwt/support/CookieSupport.java:17-26
public static ResponseCookie createAccessToken(String access) {
    return ResponseCookie.from("accessToken", access)
            .path("/")
            .maxAge(30 * 60 * 1000)          // ⚠️
            .secure(true)
            .domain(DOMAIN_URL)              // ⚠️
            .httpOnly(true)
            .sameSite("none")
            .build();
}
```

```java
// security/jwt/support/CookieSupport.java:39-52
public static void setCookieFromJwt(Token token, HttpServletResponse response) {
    response.addHeader("Set-Cookie", createAccessToken(token.getAccessToken()).toString());
    response.addHeader("Set-Cookie", createRefreshToken(token.getRefreshToken()).toString());

    ResponseCookie deleteSessionCookie = ResponseCookie.from("JSESSIONID", "")
            .path("/").httpOnly(true).secure(false).sameSite("Lax").maxAge(0).build();
    response.addHeader("Set-Cookie", deleteSessionCookie.toString());
}
```

`setHeader`가 아니라 `addHeader`를 쓴 것이 정확하다. `setHeader`였다면 세 번째 호출이 앞의 둘을 덮어써서 쿠키가 하나만 남았을 것이다.

세 번째 쿠키가 `JSESSIONID`를 `Max-Age=0`으로 지우는 이유는, OAuth2 인증 과정에서 Spring Security가 임시 세션을 만들기 때문이다. JWT로 전환한 뒤에는 그 세션이 필요 없다. `SessionCreationPolicy.STATELESS`와 짝을 이루는 정리 작업이다.

**주의 A — `maxAge` 단위 버그.** `ResponseCookie.ResponseCookieBuilder.maxAge(long)`은 **초**를 받는다. 그런데 코드는 밀리초 계산식을 넣었다.

| 쿠키 | 코드 값 | 의도 | 실제 |
| --- | --- | --- | --- |
| accessToken | `30 * 60 * 1000` = 1,800,000 | 30분 | **약 20.8일** |
| refreshToken | `14 * 24 * 60 * 60 * 1000` = 1,209,600,000 | 14일 | **약 38년** |

액세스 토큰 쿠키가 20일간 브라우저에 남는다. JWT 자체의 `exp`는 여전히 30분일 테니 서버는 만료를 감지하지만, **쿠키 수명과 토큰 수명이 어긋난다.** 결과적으로 만료된 토큰이 20일 내내 매 요청에 실려 나가고, `JwtAuthenticationFilter`가 매번 리프레시 경로로 빠지며, 로그아웃 후에도 브라우저에 죽은 토큰이 남는다.

```java
// 수정
.maxAge(30 * 60)                 // 1800초 = 30분
.maxAge(14 * 24 * 60 * 60)       // 1209600초 = 14일
// 또는 명시적으로
.maxAge(Duration.ofMinutes(30))
```

**주의 B — `@Value`가 static 필드에 안 먹는다.**

```java
@Value("${server.url}")
private static String DOMAIN_URL;      // ← 항상 null
```

Spring은 static 필드에 값을 주입하지 않는다. 애초에 `CookieSupport`는 `@Component`도 아니라서 컨테이너가 손대지 않는다. `DOMAIN_URL`은 **영원히 `null`**이다.

`ResponseCookie.domain(null)`은 `Domain` 속성을 생략한다. 결과적으로 쿠키는 host-only가 된다 — 이 경우엔 **우연히 원하는 동작이다.** 왜냐면 `server.url` 값이 `http://localhost:8080`인데, 이건 유효한 `Domain` 값이 아니기 때문이다(`Domain` 속성은 scheme과 port를 포함할 수 없다). 만약 주입이 성공했다면 브라우저가 쿠키 전체를 거부했을 것이다.

즉 **버그 두 개가 서로를 상쇄하고 있다.** 하나만 고치면 인증이 깨진다. 올바른 수정은 static을 버리고 `@Component` + 인스턴스 필드로 만들면서, 값도 `example.com` 같은 순수 호스트명(또는 로컬에서는 미설정)으로 바꾸는 것이다.

**주의 C — `SameSite=None` + `Secure` + `http://localhost`.**

`SameSite=None`은 `Secure` 없이는 브라우저가 쿠키를 거부한다. 여기선 `Secure`가 있으니 조합 자체는 유효하다. 그리고 Chrome/Edge/Firefox는 `http://localhost`를 [trustworthy origin](https://w3c.github.io/webappsec-secure-contexts/)으로 취급하므로 HTTP인데도 `Secure` 쿠키를 저장한다. 그래서 dev에서 동작한다.

하지만 **`http://192.168.0.10:5173` 같은 LAN 주소로 팀원이 접속하면 쿠키가 조용히 사라진다.** localhost가 아니면 trustworthy origin이 아니기 때문이다. 에러도 안 나고 그냥 로그인이 안 되는 것처럼 보인다. `Dockerfile_Noway`가 `npm run dev -- --host`로 외부 바인딩을 하므로 실제로 마주칠 수 있는 시나리오다.

**주의 D — `deleteJwtTokenInCookie`의 속성 불일치.**

```java
// CookieSupport.java:55-68
Cookie accessToken = new Cookie("accessToken", null);
accessToken.setPath("/");
accessToken.setMaxAge(0);
accessToken.setDomain(DOMAIN_URL);        // null
response.addCookie(accessToken);
```

쿠키 삭제는 **`Name` + `Domain` + `Path`가 정확히 일치**해야 성립한다. 생성 시에는 `SameSite=None; Secure`가 붙었는데 삭제 시에는 `jakarta.servlet.http.Cookie`를 써서 `SameSite`도 `Secure`도 붙지 않는다.

`Name`/`Domain`/`Path`는 일치하므로 **삭제 자체는 동작한다** (`SameSite`와 `Secure`는 매칭 키가 아니다). 다만 삭제 지시를 담은 이 응답이 cross-site 문맥에서 오면, `SameSite` 디폴트값(Lax)에 걸려 브라우저가 이 `Set-Cookie`를 무시할 수 있다. 생성과 삭제는 **같은 빌더로 같은 속성 집합**을 쓰는 것이 안전하다.

```java
// 권장
ResponseCookie.from("accessToken", "")
    .path("/").maxAge(0).secure(true).httpOnly(true).sameSite("none").build();
```

### 5.3 `Authorization`

**스펙.** `Authorization: <scheme> <credentials>`. 흔한 scheme은 `Basic`, `Bearer`(RFC 6750), `Digest`.

**이 프로젝트.** 세 군데에 이름이 등장하는데, **셋 다 성격이 다르다.**

**(1) 아웃바운드 HTTP — 실제로 동작하는 유일한 사용처**

```java
// oauth/service/CustomOAuth2UserService.java:70-71
HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth(accessToken.getTokenValue());   // Authorization: Bearer ya29.a0...
```

Spring이 클라이언트로서 OAuth2 provider의 UserInfo 엔드포인트를 호출할 때 붙인다. 이 프로젝트에서 `Authorization` HTTP header가 실제로 나가는 유일한 지점이다.

**(2) STOMP frame native header — HTTP header가 아님**

```java
// meeting/chat/interceptor/JwtChannelInterceptor.java:23
if (StompCommand.CONNECT.equals(acc.getCommand())) {
    String bearer = acc.getFirstNativeHeader("Authorization");
    if (bearer != null && bearer.startsWith("Bearer ")) {
        token = bearer.substring(7);
    }
    ...
}
```

`getFirstNativeHeader`는 **STOMP CONNECT frame** 안의 header를 읽는다. WebSocket payload에 들어 있는 텍스트지, HTTP 요청 header가 아니다. frame은 이렇게 생겼다.

```
CONNECT
accept-version:1.2
heart-beat:10000,10000
Authorization:Bearer eyJ...

^@
```

**그리고 프론트는 이 header를 보내지 않는다.**

```js
// frontend/src/features/chat/stompClient.js:12-15
const client = new Client({
  webSocketFactory: () => new SockJS('/ws'),
  reconnectDelay: 5000,
})       // ← connectHeaders 없음
```

따라서 실제로는 `JwtChannelInterceptor`의 fallback 경로만 동작한다. `JwtHandshakeInterceptor`가 handshake의 `Cookie` header에서 토큰을 뽑아 `attributes.put("userId", ...)`로 심어두고, `JwtChannelInterceptor`가 그걸 읽는다. 두 인터셉터는 이렇게 연결되어 있다.

```
브라우저 ──HTTP GET /ws (Cookie: accessToken=...)──▶ JwtHandshakeInterceptor
                                                        │ attributes["userId"] = ...
                                                        ▼
브라우저 ──STOMP CONNECT (native header 없음)──────▶ JwtChannelInterceptor
                                                        │ sessionAttributes["userId"] 로 fallback
                                                        ▼
                                                    acc.setUser(StompPrincipal)
```

**(3) `Access-Control-Allow-Headers` 목록** — 4.6절 참고. 프론트가 보내지 않으므로 현재는 불필요하다.

**주의.** `AuthHandshakeInterceptor.java`는 `JwtHandshakeInterceptor`와 거의 동일한 코드인데, `StompWebSocketConfig`에 **등록되어 있지 않다.** 죽은 코드다. 삭제하는 게 맞다. 남겨두면 다음 사람이 "handshake 인증이 두 군데 있나?"로 30분을 쓴다.

두 클래스의 유일한 실질 차이는 거부 방식이다.

- `JwtHandshakeInterceptor`: `return false` — Spring이 **403**을 낸다
- `AuthHandshakeInterceptor`: `response.setStatusCode(HttpStatus.FORBIDDEN)` 후 `return false` — 명시적이지만 결과는 같다

### 5.4 `WWW-Authenticate` — 없는 header

**스펙.** `401 Unauthorized` 응답에 **필수**다(RFC 9110 §11.6.1). 클라이언트에게 어떤 인증 scheme을 쓰라고 알려준다.

**이 프로젝트.** 없다. `FilterExceptionHandler.setErrorResponse`가 401을 내지만 이 header를 붙이지 않는다.

**주의.** `SecurityConfig`가 `.httpBasic().disable()`을 호출한 것은 이 맥락에서 **옳은 선택이다.** httpBasic이 켜져 있으면 401 응답에 `WWW-Authenticate: Basic realm="Realm"`이 붙고, 브라우저가 **네이티브 로그인 팝업**을 띄운다. SPA에서는 최악의 UX다. 끈 것이 맞다.

다만 그 결과 401에 `WWW-Authenticate`가 아예 없어 스펙 위반 상태가 됐다. 엄격한 API 클라이언트를 붙일 계획이면 `WWW-Authenticate: Bearer realm="api"`를 명시하는 게 정석이다.

---

## 6. 라우팅·리다이렉트 Header

### 6.1 `Host`

**스펙.** HTTP/1.1 요청에 **필수**. 요청 대상의 호스트와 포트. 하나의 IP에서 여러 도메인을 서빙하는 virtual hosting의 근거다. HTTP/2에서는 `:authority` 의사 header로 대체된다.

**이 프로젝트.** 코드에서 읽지 않지만 **Vite proxy가 재작성한다.**

```js
// frontend/vite.config.js
'/api': { target: BACKEND, changeOrigin: true, rewrite: (p) => p.replace(/^\/api/, '') }
```

`changeOrigin: true`는 프록시가 target으로 요청을 전달할 때 `Host` header를 target의 것(`localhost:8080`)으로 바꾼다. 끄면 `Host: localhost:5173`이 그대로 전달되어, 호스트 기반 라우팅이 있는 백엔드에서 문제가 된다.

**`Origin` header는 재작성하지 않는다.** 그래서 브라우저가 `Origin: http://localhost:5173`을 붙인 POST 요청은 Spring에도 그 값 그대로 도착한다.

**주의 (운영).** 리버스 프록시 뒤에 배포할 때 Spring이 원래 호스트/스킴을 알려면 `X-Forwarded-*` 계열이 필요하다.

```nginx
proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host  $host;
```

```yaml
# application.yaml
server:
  forward-headers-strategy: framework
```

이게 없으면 두 가지가 깨진다.

1. **OAuth2 리다이렉트 URI**가 `http://내부IP:8080/...`으로 생성되어 provider 검증에 실패한다.
2. **`SingleVisitInterceptor`의 방문자 집계가 전부 하나로 뭉친다.** `request.getRemoteAddr()`(`SingleVisitInterceptor.java:27`)가 프록시 IP를 돌려주기 때문이다. `X-Forwarded-For`의 첫 항목을 읽어야 한다.

### 6.2 `Location`

**스펙.** 3xx 리다이렉트 응답에서 이동할 URL, 201 Created에서 생성된 리소스의 URL.

**이 프로젝트.** OAuth2 흐름에서만 나온다. `sendRedirect`가 302 + `Location`을 만든다.

```java
// oauth/support/OAuth2AuthenticationSuccessHandler.java:85-86
setCookieFromJwt(token, response);
getRedirectStrategy().sendRedirect(request, response, createRedirectUrl(clientUrl));
// → Location: http://localhost:5173
```

```java
// oauth/support/OAuth2AuthenticationSuccessHandler.java:73
// 이메일을 못 얻은 경우
sendRedirect(..., clientUrl + "/oauth2/disallowance");
```

```java
// oauth/support/CustomAuthenticationFailureHandler.java:31
sendRedirect(..., clientUrl + "/oauth/error?message=" + URLEncoder.encode(msg, "UTF-8"));
```

**중요한 설계 포인트.** 성공 핸들러는 `Set-Cookie`와 `Location`을 **같은 302 응답에** 담는다.

```http
HTTP/1.1 302 Found
Location: http://localhost:5173
Set-Cookie: accessToken=eyJ...; Path=/; Secure; HttpOnly; SameSite=None
Set-Cookie: refreshToken=eyJ...; Path=/; Secure; HttpOnly; SameSite=None
Set-Cookie: JSESSIONID=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
```

브라우저는 리다이렉트를 따라가기 **전에** `Set-Cookie`를 처리한다. 그래서 프론트로 돌아왔을 때 이미 로그인 상태다. `SameSite=None`이 여기서 결정적이다 — provider 도메인에서 시작된 cross-site 네비게이션이므로 `Strict`였다면 쿠키가 저장되지 않는다.

**주의.** `frontend/CLAUDE.md:142`가 못박는 대로, `/oauth2/disallowance`와 `/oauth/error`는 **프론트 라우트**다. 그래서 `vite.config.js`가 `/oauth2` 전체가 아니라 `/oauth2/authorization`만 프록시한다. `/oauth2` 전체를 넘기면 백엔드가 `/oauth2/disallowance`를 404로 돌려주고 리다이렉트가 죽는다. 이건 header 자체가 아니라 `Location` 값이 가리키는 경로의 소유권 문제다.

**주의 2.** `CustomAuthenticationFailureHandler`는 `setErrorResponse(response, 401, ...)`로 **응답 본문을 먼저 쓰고**(`:28`) 그 다음 `sendRedirect`를 호출한다(`:31`). 두 동작은 양립하지 않는다.

- 응답이 아직 커밋되지 않았다면(Tomcat 디폴트 버퍼 8KB — 짧은 JSON이면 보통 여기) `sendRedirect`가 버퍼를 리셋하고 302 + `Location`을 낸다. **`setErrorResponse`가 쓴 401 JSON은 통째로 버려진다.**
- 버퍼를 넘겨 이미 커밋됐다면 `sendRedirect`가 `IllegalStateException`을 던진다.

즉 어느 쪽이든 의도한 대로 동작하지 않는다. API 클라이언트용 401 JSON을 줄지, 브라우저용 302를 줄지 하나만 정해야 한다. 이 흐름은 브라우저 전체 페이지 이동에서 오므로 **302 쪽이 맞다** — `setErrorResponse` 호출을 지우면 된다.

### 6.3 `Referer`

**스펙.** 이 요청을 유발한 페이지의 URL. 철자가 `Referrer`가 아니라 `Referer`인 것은 원 스펙의 오타가 굳어진 것이다. `Referrer-Policy` 응답 header로 노출 수준을 제어한다.

**이 프로젝트.** 코드에서 읽지 않는다. OAuth 리다이렉트 체인에서 브라우저가 자동으로 붙인다.

**주의.** `CustomAuthenticationFailureHandler`가 에러 메시지를 **쿼리 스트링**에 담는다(`/oauth/error?message=...`). 그 페이지에서 나가는 모든 요청의 `Referer`에 이 메시지가 실린다. 지금은 민감하지 않지만, 에러 메시지에 이메일이나 내부 식별자가 들어가면 유출 경로가 된다. `Referrer-Policy: strict-origin-when-cross-origin`을 디폴트로 두는 게 좋다.

---

## 7. WebSocket / SockJS Header

### 7.1 handshake header 일습

WebSocket 연결은 **일반 HTTP GET으로 시작**해서 `101 Switching Protocols`로 프로토콜을 바꾼다. 이 GET에는 쿠키가 그대로 실린다 — `JwtHandshakeInterceptor`가 동작하는 근거다.

**요청:**

```http
GET /ws/123/abcdefg/websocket HTTP/1.1
Host: localhost:8080
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
Origin: http://localhost:5173
Cookie: accessToken=eyJ...; refreshToken=eyJ...
```

**응답:**

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

| Header | 역할 |
| --- | --- |
| `Upgrade: websocket` | 프로토콜 전환 요청 |
| `Connection: Upgrade` | hop-by-hop header. **프록시가 반드시 전달해야 한다.** |
| `Sec-WebSocket-Key` | 클라이언트가 만든 16바이트 랜덤의 base64. 보안용이 아니라 캐시/프록시가 실수로 응답하는 것을 막는 용도. |
| `Sec-WebSocket-Accept` | `Key` + 고정 GUID를 SHA-1 → base64. 서버가 handshake를 이해했다는 증명. |
| `Sec-WebSocket-Version` | 항상 `13` (RFC 6455). |
| `Sec-WebSocket-Extensions` | `permessage-deflate` 등 압축 협상. |
| `Sec-WebSocket-Protocol` | 서브프로토콜 협상. 이 프로젝트는 쓰지 않는다(STOMP를 payload로 얹으므로). |

`Sec-*` 접두사 header는 **JS가 설정할 수 없다.** 브라우저만 만든다. 그래서 `WebSocket` 생성자에 커스텀 header를 넣을 방법이 없고, 이것이 STOMP가 `Authorization`을 **frame native header**로 나르는 이유다.

### 7.2 SockJS가 개입하면

```java
// meeting/chat/config/StompWebSocketConfig.java:22-25
registry.addEndpoint("/ws")
        .addInterceptors(new JwtHandshakeInterceptor(jwtTokenProvider))
        .setAllowedOriginPatterns("*")
        .withSockJS();
```

```js
// frontend/src/features/chat/stompClient.js:13
webSocketFactory: () => new SockJS('/ws'),
```

`.withSockJS()`가 켜져 있으면 순수 WebSocket이 아니다. SockJS는 이런 순서를 밟는다.

1. **`GET /ws/info`** — 일반 HTTP. 서버 능력과 엔트로피를 받는다.
2. transport 선택 — `websocket` → `xhr-streaming` → `xhr-polling` 순으로 폴백
3. 선택된 transport로 `/ws/{serverId}/{sessionId}/{transport}` 연결

**핵심:** WebSocket이 아닌 폴백 transport에서는 `Upgrade`/`Sec-WebSocket-*`가 아예 없고, **평범한 XHR long-polling**이 된다. 이 경우에도 `Cookie` header는 실리므로 `JwtHandshakeInterceptor`는 여전히 동작한다. 다만 cross-origin 배포에서는 이 XHR들이 **CORS 대상**이 되어 4장의 결함이 그대로 적용된다.

`vite.config.js`의 `'/ws': { target: BACKEND, changeOrigin: true, ws: true }`에서 `ws: true`가 `Upgrade`/`Connection` header를 프록시가 전달하도록 한다. 이게 없으면 handshake가 101 대신 200으로 떨어지고 SockJS가 조용히 XHR 폴백으로 내려간다 — "채팅이 되긴 하는데 느리다"의 전형적 원인이다.

**주의.** `setAllowedOriginPatterns("*")`는 WebSocket handshake에 대해 **모든 origin을 허용**한다. `CorsFilter`가 HTTP API에는 `http://localhost:5173`만 허용하는 것과 어긋난다. WebSocket은 same-origin 정책의 적용을 받지 않으므로(브라우저가 cross-origin WebSocket을 막지 않는다) 이건 실제 위험이다 — 공격자 페이지가 피해자의 쿠키로 채팅 소켓을 열 수 있다(Cross-Site WebSocket Hijacking). `setAllowedOrigins(clientUrl)`로 좁혀야 한다.

### 7.3 STOMP frame header (HTTP 아님)

WebSocket이 열린 뒤 오가는 것은 STOMP frame이다. 형식이 HTTP와 비슷해서 헷갈리지만 **완전히 다른 레이어**다.

```
SEND
destination:/app/chat.send.42
content-type:application/json
content-length:58

{"sender":"u1","meetingId":42,"messageType":"SEND"}^@
```

```
MESSAGE
destination:/topic/chat.42
subscription:sub-0
message-id:abc-123
content-type:application/json

{"sender":"u1","message":"안녕"}^@
```

프로젝트에서 쓰는 destination:

| destination | 방향 | 근거 |
| --- | --- | --- |
| `/app/chat.send.{meetingId}` | 클라 → 서버 | `stompClient.js:34`, `@MessageMapping` |
| `/topic/chat.{meetingId}` | 서버 → 클라 (구독) | `chatApi.js:28`, `enableSimpleBroker("/topic")` |
| `/user/...` | 개별 전송 | `setUserDestinationPrefix("/user")` |

`registry.setApplicationDestinationPrefixes("/app")`, `enableSimpleBroker("/topic")`가 이 prefix들을 정의한다(`StompWebSocketConfig.java:31-35`).

---

## 8. 캐시·진단 Header

### 8.1 `User-Agent`

**스펙.** 클라이언트 소프트웨어 식별 문자열. 역사적 호환성 때문에 거의 모든 브라우저가 "Mozilla/5.0 (...)"로 시작하는 위장 문자열을 보낸다. 신뢰할 수 없는 값이며, [Client Hints](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Client_hints)(`Sec-CH-UA-*`)로 대체되는 추세다.

**이 프로젝트.** 방문자 집계에 쓴다.

```java
// admin/visitant/util/SingleVisitInterceptor.java:27-35
String userIp = request.getRemoteAddr();
String userAgent = ((HttpServletRequest) request).getHeader("User-Agent");
String today = LocalDate.now().toString();
String key = userIp + "_" + today;

if (!valueOperations.getOperations().hasKey(key)) {
    valueOperations.set(key, userAgent);
}
```

**주의 세 가지.**

1. **키가 IP+날짜라서 UA는 값으로만 저장된다.** NAT 뒤 여러 사용자는 1명으로 집계되고, 모바일 IP 로밍은 여러 명으로 집계된다. UA를 키에 넣지 않았으므로 같은 IP의 서로 다른 기기도 1명이다.
2. **TTL이 없다.** `valueOperations.set(key, userAgent)`에 만료가 없어 Redis 키가 무한 누적된다. `VisitantScheduler`가 별도로 정리하는지 확인이 필요하지만, 최소한 `set(key, ua, Duration.ofDays(2))`로 안전망을 두는 게 맞다.
3. **`getRemoteAddr()`는 프록시 뒤에서 무의미하다** — 6.1절 참고.
4. **모든 요청에 대해 실행된다.** 이 필터는 정적 자산, preflight, `/ws/info`에도 걸린다. 요청당 Redis 왕복이 최소 1회 추가된다.

### 8.2 `X-Requested-With`

**스펙.** 표준이 아니다. jQuery/Prototype 시절 XHR임을 표시하려고 `XMLHttpRequest` 값을 붙이던 관습. 커스텀 header이므로 cross-origin에서는 preflight를 유발한다.

**이 프로젝트.** `CorsFilter.java:30`의 허용 목록에 이름만 있다. **어떤 코드도 이 header를 보내지 않는다.**

**주의.** 과거 CSRF 방어로 쓰이기도 했다(`<form>` 제출은 커스텀 header를 붙일 수 없으므로). 하지만 이 프로젝트는 아무도 보내지 않고 서버도 검사하지 않으므로 아무 효과가 없다. 목록에서 빼는 게 정직하다.

### 8.3 `Cache-Control` / `ETag` / `If-None-Match` / `Last-Modified`

**스펙.**

| Header | 의미 |
| --- | --- |
| `Cache-Control: no-store` | 어디에도 저장 금지 |
| `Cache-Control: no-cache` | 저장은 하되 매번 재검증 |
| `Cache-Control: max-age=31536000, immutable` | 1년 캐시, 재검증 불필요 |
| `ETag` | 리소스 버전 식별자 |
| `If-None-Match` | 클라이언트가 가진 ETag. 같으면 서버가 `304 Not Modified` |
| `Last-Modified` / `If-Modified-Since` | 타임스탬프 기반 재검증 |

**이 프로젝트.** 애플리케이션 코드에서 설정하지 않는다. 실제로는 세 곳에서 흐른다.

1. **Vite dev server** — 소스 모듈에 `Cache-Control: no-cache`, 의존성 사전 번들에 `max-age=31536000, immutable`
2. **Vite 프로덕션 빌드** — 파일명에 콘텐츠 해시가 들어가므로(`assets/index-a1b2c3.js`) 정적 서버가 `immutable`을 줄 수 있다. `index.html`만 `no-cache`여야 한다.
3. **S3** — 첨부파일 응답에 `ETag`(대개 MD5)와 `Last-Modified`를 자동으로 붙인다.

**주의 (보안).** 인증 정보를 담은 응답에는 `Cache-Control: no-store`가 필요하다. 현재 `/profiles`, `/admin/**` 같은 응답에 아무 캐시 지시자가 없어서, 중간 프록시나 브라우저 back/forward 캐시에 개인정보가 남을 수 있다.

```java
// SecurityConfig 에 추가 — Spring Security의 디폴트 캐시 방어를 켠다
http.headers(headers -> headers
    .cacheControl(Customizer.withDefaults())   // no-cache, no-store, max-age=0, must-revalidate
);
```

### 8.4 없는 보안 응답 header ⚠️

`SecurityConfig`가 `http.headers(...)`를 전혀 설정하지 않는다. Spring Security는 디폴트로 여러 보안 header를 넣어주지만, 이 프로젝트는 `HttpSecurity`를 명시적으로 구성하면서 그 부분을 건드리지 않았다. 확인 및 추가가 필요한 목록:

| Header | 값 | 막는 것 |
| --- | --- | --- |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | HTTPS 다운그레이드 |
| `X-Content-Type-Options` | `nosniff` | MIME 스니핑. **S3에 사용자 파일을 그대로 서빙하는 이 프로젝트에 특히 중요.** |
| `X-Frame-Options` / `frame-ancestors` | `DENY` | 클릭재킹 |
| `Content-Security-Policy` | 최소 `default-src 'self'` | XSS. `RichTextEditor.jsx`가 있으므로 실제 위협. |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | URL 유출 (6.3절) |
| `Cross-Origin-Opener-Policy` | `same-origin` | cross-origin 윈도우 참조 |

---

## 9. 시나리오별 전체 흐름

### 9.1 폼 로그인 — `POST /logins`

```http
POST /api/logins HTTP/1.1                        ← 브라우저 → Vite(5173)
Host: localhost:5173
Origin: http://localhost:5173                    ← same-origin POST 라도 붙는다
Content-Type: application/json
Content-Length: 32

{"id":"user1","password":"1234"}
```

Vite proxy가 `/api`를 떼고 `Host`를 바꿔 전달:

```http
POST /logins HTTP/1.1                            ← Vite → Spring(8080)
Host: localhost:8080                             ← changeOrigin: true
Origin: http://localhost:5173                    ← 그대로
Content-Type: application/json
```

Spring 응답:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://localhost:5173      ← CorsFilter (dev 에선 무시됨)
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: *
Access-Control-Max-Age: 3600
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization
Content-Type: application/json;charset=UTF-8
Set-Cookie: accessToken=eyJhbGciOiJIUzI1NiJ9...; Path=/; Max-Age=1800000; Secure; HttpOnly; SameSite=None
Set-Cookie: refreshToken=eyJhbGciOiJIUzI1NiJ9...; Path=/; Max-Age=1209600000; Secure; HttpOnly; SameSite=None
Set-Cookie: JSESSIONID=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax

{"code":"...","message":"로그인에 성공했습니다."}
```

관련 코드: `LoginController.java:44` → `LoginService.login(request, response)` → `CookieSupport.setCookieFromJwt`

### 9.2 인증된 요청 + 액세스 토큰 만료 재발급

**1차 요청 — 토큰이 만료된 상태:**

```http
GET /api/profiles HTTP/1.1
Cookie: accessToken=<expired>; refreshToken=<valid>
```

`JwtAuthenticationFilter`가 `validateAccessToken` 실패 → `else if` 분기 → `JwtService.validateRefreshToken`:

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Set-Cookie: accessToken=<new>; Path=/; Max-Age=1800000; Secure; HttpOnly; SameSite=None

{"data":"4","message":"..."}          ← ResponseCode.CREATE_ACCESS_TOKEN
```

**원래 요청은 처리되지 않았다.** 프론트가 봉투를 보고 재시도:

```http
GET /api/profiles HTTP/1.1
Cookie: accessToken=<new>; refreshToken=<valid>
```

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{"data":{...실제 프로필...},"message":"..."}
```

관련 코드: `JwtAuthenticationFilter.java:32-42` ↔ `baseApi.js:25-28`

### 9.3 multipart 게시글 작성 — `POST /posts`

```http
POST /api/posts HTTP/1.1
Origin: http://localhost:5173
Cookie: accessToken=eyJ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWx
Content-Length: 284531

------WebKitFormBoundary7MA4YWx
Content-Disposition: form-data; name="postRequest"; filename="blob"
Content-Type: application/json

{"title":"제목","content":"<p>본문</p>","tags":["java"]}
------WebKitFormBoundary7MA4YWx
Content-Disposition: form-data; name="multipartFiles"; filename="spec.pdf"
Content-Type: application/pdf

%PDF-1.7 ...
------WebKitFormBoundary7MA4YWx--
```

```http
HTTP/1.1 201 Created
Content-Type: application/json;charset=UTF-8

{"code":"...","message":"게시글이 등록되었습니다."}
```

**cross-origin이라면** 앞에 preflight가 붙는다. `Content-Type: multipart/form-data`는 safelisted 값이라 preflight를 유발하지 않지만, 메서드가 `POST`이므로 simple request다 — **preflight 없음**. 반면 `PUT /posts/{id}`(같은 multipart)는 메서드 때문에 preflight를 탄다.

관련 코드: `lib/file.js:38-48` → `postApi.js` → `PostController.java:25-31`

### 9.4 CORS preflight (cross-origin 배포 시)

```http
OPTIONS /posts/42 HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: content-type
```

현재 `CorsFilter`의 응답:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: *                    ← ⚠️ 여기서 실패한다
Access-Control-Max-Age: 3600
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization
```

**브라우저 판정:** `Access-Control-Allow-Credentials: true`이므로 `*`는 리터럴로 해석된다. 요청 메서드 `DELETE`가 허용 목록(`"*"`라는 이름의 메서드 하나)에 없다 →

```
Access to fetch at 'https://api.example.com/posts/42' from origin 'https://app.example.com'
has been blocked by CORS policy: Method DELETE is not allowed by
Access-Control-Allow-Methods in preflight response.
```

### 9.5 OAuth2 소셜 로그인 전체

```
① 사용자가 "구글로 로그인" 클릭
   LoginPage.jsx:18 — window.location.href = '/oauth2/authorization/google'
   (SPA 네비게이션이 아니라 전체 페이지 이동. CLAUDE.md:141 이 유일한 예외라고 명시)

② GET /oauth2/authorization/google  →  Vite proxy → Spring
   Spring Security 의 OAuth2AuthorizationRequestRedirectFilter 가 응답:

   HTTP/1.1 302 Found
   Location: https://accounts.google.com/o/oauth2/v2/auth?
             client_id=...&redirect_uri=http://localhost:8080/login/oauth2/code/google
             &scope=email%20profile&state=<csrf>&response_type=code
   Set-Cookie: JSESSIONID=...; Path=/; HttpOnly       ← state 보관용 임시 세션

③ 사용자가 구글에서 동의

④ GET http://localhost:8080/login/oauth2/code/google?code=4/0Ade...&state=<csrf>
   Cookie: JSESSIONID=...                             ← state 검증에 필요

⑤ Spring → Google 토큰 엔드포인트 (서버 간 통신, 브라우저 무관)
   POST https://oauth2.googleapis.com/token
   Content-Type: application/x-www-form-urlencoded

⑥ Spring → Google UserInfo — CustomOAuth2UserService.java:70-75
   GET https://openidconnect.googleapis.com/v1/userinfo
   Authorization: Bearer ya29.a0AfH6...                ← headers.setBearerAuth
   Accept: application/json                            ← headers.setAccept

⑦ OAuth2AuthenticationSuccessHandler
   HTTP/1.1 302 Found
   Location: http://localhost:5173
   Set-Cookie: accessToken=eyJ...; Secure; HttpOnly; SameSite=None
   Set-Cookie: refreshToken=eyJ...; Secure; HttpOnly; SameSite=None
   Set-Cookie: JSESSIONID=; Max-Age=0                  ← 임시 세션 정리

⑧ 브라우저가 Set-Cookie 를 먼저 적용한 뒤 Location 을 따라간다
   → 프론트 도착 시 이미 로그인 상태
```

⑦의 `SameSite=None`이 필수인 이유: 이 302는 백엔드(`localhost:8080`)에서 나오고 브라우저는 `localhost:5173`으로 이동한다. 포트가 다르지만 **쿠키의 same-site 판정은 registrable domain 기준이라 포트는 무시**되므로 localhost끼리는 same-site다. 운영에서 `api.example.com` → `app.example.com`이라면 여전히 same-site(같은 등록 도메인). 하지만 완전히 다른 도메인이면 `None`이 반드시 필요하다.

### 9.6 SockJS + STOMP 채팅 연결

```
① SockJS 정보 조회
   GET /ws/info?t=1724567890123
   Cookie: accessToken=eyJ...
   ← JwtHandshakeInterceptor 는 아직 안 탄다(/info 는 handshake 가 아님)

   HTTP/1.1 200 OK
   Content-Type: application/json
   {"entropy":123456,"origins":["*:*"],"cookie_needed":true,"websocket":true}

② WebSocket handshake
   GET /ws/482/k2m9x1qp/websocket
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
   Sec-WebSocket-Version: 13
   Origin: http://localhost:5173
   Cookie: accessToken=eyJ...                    ← 여기서 인증된다

   → JwtHandshakeInterceptor.beforeHandshake
     · Cookie 에서 accessToken 추출
     · validateAccessToken 통과 → attributes["userId"] = userPk
     · return true

   HTTP/1.1 101 Switching Protocols
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=

③ STOMP CONNECT frame (WebSocket payload, HTTP 아님)
   CONNECT
   accept-version:1.2
   heart-beat:10000,10000
   (Authorization native header 없음 — 프론트가 connectHeaders 를 안 준다)

   → JwtChannelInterceptor.preSend
     · getFirstNativeHeader("Authorization") → null
     · sessionAttributes["userId"] fallback → acc.setUser(StompPrincipal)

   CONNECTED
   version:1.2
   heart-beat:10000,10000

④ 구독 — chatApi.js:28
   SUBSCRIBE
   id:sub-0
   destination:/topic/chat.42

⑤ 발행 — stompClient.js:33-36
   SEND
   destination:/app/chat.send.42
   content-length:64

   {"sender":"u1","meetingId":42,"message":"안녕","messageType":"SEND"}

⑥ 브로드캐스트
   MESSAGE
   destination:/topic/chat.42
   subscription:sub-0
   message-id:d-0-7
   content-type:application/json

   {"sender":"u1","message":"안녕",...}
```

### 9.7 S3 첨부파일 다운로드

```http
GET https://noticeboardbucket.s3.ap-northeast-2.amazonaws.com/<uuid> HTTP/1.1
Origin: http://localhost:5173
                                    ← Cookie 없음. queryFn 이 baseQuery 를 우회했으므로
                                      credentials: 'include' 가 붙지 않는다
```

```http
HTTP/1.1 200 OK
Content-Type: application/pdf              ← S3Service.java:37 에서 저장한 값
Content-Length: 284531                     ← S3Service.java:38
ETag: "9b2cf5a1e0d4..."
Last-Modified: Sun, 24 Aug 2026 10:00:00 GMT
Access-Control-Allow-Origin: http://localhost:5173     ← 버킷 CORS 설정이 있어야 함
```

프론트는 blob으로 받아 `blob:` URL을 만들고 `<a download>`로 저장한다(`postApi.js:121-127`). **응답에 `Content-Disposition`이 없어도 되는 이유**가 바로 이 우회 기법이다.

---

## 10. 종합: header 관련 결함 목록

| # | 위치 | 문제 | 심각도 | 수정 |
| --- | --- | --- | --- | --- |
| 1 | `CorsFilter.java:28` | `Access-Control-Allow-Methods: *` + `Allow-Credentials: true` → 스펙상 `*`가 리터럴로 해석되어 preflight가 필요한 모든 요청 실패 | **높음** (cross-origin 배포 시) | `"GET, POST, PUT, PATCH, DELETE, OPTIONS"` 명시 |
| 2 | `CookieSupport.java:20,31` | `maxAge`가 초 단위인데 밀리초 값 전달 → accessToken 쿠키 20.8일, refreshToken 38년 | **높음** | `30*60`, `14*24*60*60` 또는 `Duration.ofMinutes(30)` |
| 3 | `CookieSupport.java:14-15` | `@Value`를 static 필드에 → `DOMAIN_URL`이 항상 null | **중간** (현재는 다른 버그와 상쇄 중) | `@Component` + 인스턴스 필드, 값도 순수 호스트명으로 |
| 4 | `StompWebSocketConfig.java:24` | `setAllowedOriginPatterns("*")` → Cross-Site WebSocket Hijacking 가능 | **높음** | `setAllowedOrigins(clientUrl)` |
| 5 | `SecurityConfig.java:43` | `.csrf().disable()` + 쿠키 인증 + `SameSite=None` → CSRF 무방비 | **높음** | `SameSite=Lax` (same-site 배치) 또는 CSRF 토큰 |
| 6 | `SecurityConfig` 전체 | `http.headers(...)` 미설정 → HSTS / nosniff / CSP / Referrer-Policy 부재 | **중간** | 8.4절 표 참고 |
| 7 | `AuthHandshakeInterceptor.java` | 등록되지 않은 죽은 코드. `JwtHandshakeInterceptor`와 중복 | 낮음 (유지보수) | 삭제 |
| 8 | `CustomAuthenticationFailureHandler.java:28-31` | 401 JSON을 쓴 뒤 `sendRedirect` → JSON이 버려지거나(미커밋) `IllegalStateException`(커밋됨) | **중간** | `setErrorResponse` 호출 제거, 302만 남기기 |
| 9 | `@CookieValue String accessToken` × 28 | 쿠키 없으면 401이 아니라 **400** | **중간** | `@AuthenticationPrincipal`로 교체 |
| 10 | `S3Service.java:37,52` | `file.getContentType()`을 검증 없이 S3에 저장 → 사용자 파일이 버킷 도메인에서 실행 가능 | **중간** | `application/octet-stream` 강제 + `Content-Disposition: attachment` |
| 11 | `S3Service.java:38,53` | `available()`은 파일 크기를 보장하지 않음 | **중간** | `file.getSize()` |
| 12 | `S3Service.java:80-86` | `if(!ext.equals("exe") \|\| !ext.equals("bat"))` — 항상 참 → 모든 첨부 거부 | **높음** | `if (BLOCKED.contains(ext))` |
| 13 | `CorsFilter.java:30` | `Allow-Headers`에 `Origin`(forbidden), `X-Requested-With`(미사용), `Accept`(safelisted) 나열 | 낮음 | `"Content-Type"`만 남기기 |
| 14 | `SingleVisitInterceptor.java:27` | `getRemoteAddr()`가 프록시 뒤에서 프록시 IP → 집계 붕괴 | **중간** (운영) | `X-Forwarded-For` + `forward-headers-strategy: framework` |
| 15 | `SingleVisitInterceptor.java:35` | Redis 키에 TTL 없음 → 무한 누적 | **중간** | `set(key, ua, Duration.ofDays(2))` |
| 16 | `baseApi.js:25-28` | 매직 문자열 `'4'` 기반 리프레시. 동시 요청 시 race | **중간** | `401` + mutex 직렬화 |
| 17 | `CookieSupport.java:55-68` | 삭제 쿠키가 생성 쿠키와 다른 빌더·다른 속성 | 낮음 | `ResponseCookie`로 통일, 동일 속성 |
| 18 | 전역 | `Vary: Origin` 없음 | 낮음 (현재), **높음** (화이트리스트 전환 시) | ACAO를 동적으로 만들면 반드시 추가 |

---

## 11. 한 줄 요약표

| Header | 방향 | 이 프로젝트에서의 역할 |
| --- | --- | --- |
| `Host` | 요청 | Vite proxy가 `changeOrigin`으로 재작성. 운영에선 `X-Forwarded-Host` 필요 |
| `Origin` | 요청 | 브라우저 자동 생성. 서버는 읽지 않고 `client.url` 고정값만 사용 |
| `Content-Type` | 양방향 | JSON 봉투 / multipart 라우팅(`consumes`) / S3 오브젝트 메타데이터 |
| `Content-Length` | 양방향 | Tomcat 자동. S3 업로드 시 명시(단, `available()` 버그) |
| `Content-Disposition` | 응답 | **미사용.** 다운로드는 blob + `<a download>`로 우회 |
| `Accept` | 요청 | OAuth2 UserInfo 호출에서만 명시 |
| `Cookie` | 요청 | **인증의 전부.** accessToken / refreshToken. 필터·컨트롤러·WS handshake 모두 여기서 읽음 |
| `Set-Cookie` | 응답 | 로그인·리프레시·OAuth 성공 시 발급. `Secure; HttpOnly; SameSite=None` |
| `Authorization` | 요청 | OAuth2 provider 호출(아웃바운드)에만 실사용. STOMP 쪽은 frame header이고 프론트가 안 보냄 |
| `WWW-Authenticate` | 응답 | **미사용.** `httpBasic` 비활성이라 팝업은 안 뜨지만 401 스펙 위반 |
| `Access-Control-Allow-Origin` | 응답 | `http://localhost:5173` 고정 |
| `Access-Control-Allow-Credentials` | 응답 | `true`. 프론트 `credentials: 'include'`와 짝 |
| `Access-Control-Allow-Methods` | 응답 | `*` ⚠️ credentials와 함께 쓰면 무효 |
| `Access-Control-Allow-Headers` | 응답 | 5개 나열, 실제 필요한 건 `Content-Type` 하나 |
| `Access-Control-Max-Age` | 응답 | `3600`. Chrome 상한 7200 이내라 안전 |
| `Access-Control-Request-Method/Headers` | 요청 | 브라우저가 preflight에 붙임. 서버는 읽지 않음 |
| `Vary` | 응답 | **미사용.** ACAO 고정이라 현재는 무해 |
| `Location` | 응답 | OAuth2 302 리다이렉트. `Set-Cookie`와 같은 응답에 실림 |
| `Referer` | 요청 | 읽지 않음. `/oauth/error?message=`가 유출 경로가 될 수 있음 |
| `Upgrade` / `Connection` | 양방향 | WebSocket 101 전환. Vite `ws: true`가 전달 보장 |
| `Sec-WebSocket-Key/Accept/Version/Extensions` | 양방향 | 브라우저·서버 자동. JS가 설정 불가 → STOMP가 토큰을 frame으로 나르는 이유 |
| `User-Agent` | 요청 | Redis 방문자 집계의 값 (`SingleVisitInterceptor`) |
| `X-Requested-With` | — | 허용 목록에 이름만. 실사용 없음 |
| `Cache-Control` / `ETag` / `Last-Modified` | 응답 | Vite·S3가 자동 생성. 앱 코드는 미설정 (보안상 `no-store` 필요) |
| `X-Forwarded-For/Proto/Host` | 요청 | **운영 필수.** 현재 미설정 → IP 집계·OAuth 리다이렉트 붕괴 위험 |
| `Strict-Transport-Security` / `X-Content-Type-Options` / `CSP` / `Referrer-Policy` | 응답 | **전부 미설정** |

---

## 부록: 운영 리버스 프록시 예시

`frontend/CLAUDE.md:208`의 "운영 리버스 프록시도 같은 rewrite 규칙이어야 한다"를 구현하면 CORS 자체가 사라지고 10장의 #1, #5, #18이 동시에 해결된다.

```nginx
server {
    listen 443 ssl http2;
    server_name app.example.com;

    # SPA 정적 파일
    location / {
        root /srv/frontend/dist;
        try_files $uri /index.html;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
    location = /index.html {
        root /srv/frontend/dist;
        add_header Cache-Control "no-cache";
    }

    # API — vite.config.js 의 프록시 규칙과 동일하게
    location /api/ {
        proxy_pass         http://backend:8080/;      # 끝 슬래시로 /api 제거
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   X-Forwarded-Host  $host;
    }

    # SockJS / WebSocket
    location /ws/ {
        proxy_pass         http://backend:8080/ws/;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade    $http_upgrade;    # ← 필수
        proxy_set_header   Connection "upgrade";        # ← 필수
        proxy_set_header   Host       $host;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_read_timeout 3600s;                       # 유휴 소켓 유지
    }

    # OAuth 시작점만 (/oauth2 전체를 넘기면 안 된다 — CLAUDE.md:142)
    location /oauth2/authorization/ {
        proxy_pass       http://backend:8080/oauth2/authorization/;
        proxy_set_header Host              $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

이 구성이면 브라우저 입장에서 모든 것이 `https://app.example.com` 하나의 origin이다.

- `CorsFilter`의 `Access-Control-*`가 전부 무의미해진다 → 필터를 지워도 된다
- 쿠키를 `SameSite=Lax`로 좁힐 수 있다 → CSRF 표면이 사라진다
- preflight가 없다 → `Allow-Methods: *` 버그가 소멸한다
- `X-Forwarded-*`가 붙는다 → `SingleVisitInterceptor`의 IP 집계와 OAuth 리다이렉트가 정상화된다

`application.yaml`에 다음을 추가해야 Spring이 `X-Forwarded-*`를 신뢰한다.

```yaml
server:
  forward-headers-strategy: framework
```

---

## 참고

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 6265bis — Cookies (SameSite 포함)](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis)
- [Fetch Standard — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol)
- [MDN — Access-Control-Allow-Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Access-Control-Allow-Methods)
- [Spring — ResponseCookie.ResponseCookieBuilder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseCookie.ResponseCookieBuilder.html)
- [RFC 6455 — The WebSocket Protocol](https://www.rfc-editor.org/rfc/rfc6455.html)
- [STOMP 1.2 Protocol Specification](https://stomp.github.io/stomp-specification-1.2.html)
