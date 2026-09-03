# HTTP Headers

## HTTP Request와 Response를 이해

HTTP 통신을 공부할 때 많은 초보자가 다음과 같이 생각합니다.

```text
HTTP Request
=
URL + Method + Body
```

하지만 실제 HTTP 통신은 그렇지 않습니다.

```text
HTTP Message
│
├─ Start Line
├─ Headers
│
└─ Body
```

예를 들어 브라우저가 JSON 데이터를 요청한다면 실제 HTTP Request는 대략 다음과 같습니다.

```http
GET /api/products HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJ...
Origin: https://www.example.com
User-Agent: Mozilla/5.0 ...
```

서버는 다음과 같이 응답할 수 있습니다.

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 1358
Cache-Control: max-age=3600
ETag: "product-v21"
Access-Control-Allow-Origin: https://www.example.com

[
  ...
]
```

여기에서

```text
Content-Type
Authorization
Origin
User-Agent
Cache-Control
ETag
...
```

같은 값들이 바로 **HTTP Header Field**입니다.

HTTP Header는 단순한 부가 정보가 아닙니다.

> HTTP Header는 클라이언트와 서버가 HTTP Message 자체를 어떻게 해석하고 처리해야 하는지를 전달하는 메타데이터입니다.

---

# PART 1. HTTP Header의 기본 구조

## 1. Header Field의 기본 형태

HTTP Header는 기본적으로 다음 형태입니다.

```http
Header-Name: Header-Value
```

예:

```http
Content-Type: application/json
```

```text
Content-Type
      │
      └── Header Field Name

application/json
      │
      └── Header Field Value
```

HTTP 헤더 이름은 **case-insensitive**입니다.

따라서 프로토콜 의미상 다음은 동일하게 취급됩니다.

```http
Content-Type: application/json
```

```http
content-type: application/json
```

다만 문서와 코드에서는 일반적으로 표준 표기 형태를 사용합니다.

---

# PART 2. Request Header와 Response Header

Header는 사용 방향에 따라 크게 나눌 수 있습니다.

```text
Client
   │
   │ HTTP Request
   │
   │ Host
   │ Accept
   │ Authorization
   │ Cookie
   │ Origin
   │ User-Agent
   ▼
Server
```

반대 방향에서는:

```text
Server
   │
   │ HTTP Response
   │
   │ Content-Type
   │ Set-Cookie
   │ Location
   │ ETag
   │ Cache-Control
   │ Content-Security-Policy
   ▼
Client
```

하지만 모든 Header를 단순히 Request 또는 Response로만 나눌 수 있는 것은 아닙니다.

일부 헤더는 양쪽에서 사용됩니다.

대표적으로:

```text
Content-Type
Cache-Control
Connection
Upgrade
Sec-WebSocket-Protocol
```

등이 있습니다.

---

# PART 3. 표현과 콘텐츠 관련 Header

이번 강의에서 다룰 첫 번째 그룹입니다.

| Header                | 주요 방향              | 핵심 역할                    |
| --------------------- | ------------------ | ------------------------ |
| `Content-Type`        | Request / Response | Body의 Media Type         |
| `Content-Length`      | Request / Response | Body 길이                  |
| `Content-Disposition` | 주로 Response        | 콘텐츠 표시/다운로드 방식           |
| `Accept`              | Request            | 클라이언트가 원하는 응답 Media Type |

---

# 1. Content-Type

## 정의

`Content-Type`은 현재 HTTP Message Body에 들어 있는 데이터가 **어떤 형식인지** 나타냅니다.

예:

```http
Content-Type: application/json
```

의 의미는:

```text
HTTP Body 안의 데이터는 JSON이다.
```

입니다.

---

## JSON

```http
Content-Type: application/json
```

Body:

```json
{
  "id": 10,
  "name": "Keyboard"
}
```

---

## HTML

```http
Content-Type: text/html; charset=UTF-8
```

---

## 일반 텍스트

```http
Content-Type: text/plain; charset=UTF-8
```

---

## CSS

```http
Content-Type: text/css
```

---

## JavaScript

일반적으로:

```http
Content-Type: text/javascript
```

등이 사용됩니다.

---

## HTML Form

일반 `<form>` 전송에서는 다음과 같은 타입을 볼 수 있습니다.

```http
Content-Type: application/x-www-form-urlencoded
```

파일 업로드가 포함된 form에서는:

```http
Content-Type: multipart/form-data; boundary=----abc123
```

가 사용됩니다.

---

## 중요한 구분

`Content-Type`과 `Accept`는 서로 반대 개념에 가깝습니다.

```text
Accept
=
"나는 이런 형식으로 받고 싶다."

Content-Type
=
"내가 지금 보내는 데이터는 이 형식이다."
```

예:

```http
POST /users HTTP/1.1
Content-Type: application/json
Accept: application/json
```

의 의미:

```text
Request Body
→ JSON으로 보냄

Response Body
→ 가능하면 JSON으로 받고 싶음
```

---

# 2. Content-Length

`Content-Length`는 HTTP Message Body의 길이를 **byte 단위**로 나타냅니다.

```http
Content-Length: 348
```

의 의미:

```text
Body 크기 = 348 bytes
```

입니다.

예:

```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 5

Hello
```

HTTP Message를 읽는 쪽에서는 이 값을 이용해 Body의 경계를 판단할 수 있습니다.

다만 모든 HTTP Message에 반드시 `Content-Length`가 존재하는 것은 아닙니다.

HTTP/1.1에서는 message framing과 `Transfer-Encoding` 등의 규칙이 함께 작동하며, HTTP/2와 HTTP/3에서는 framing 자체가 HTTP/1.1과 다릅니다.

따라서:

> `Content-Length`는 Body의 크기를 표현하는 헤더이지, 모든 HTTP 통신에서 반드시 필요한 헤더는 아닙니다.

---

# 3. Content-Disposition

`Content-Disposition`은 콘텐츠를 브라우저에서 바로 표시할지, 파일로 다운로드하게 할지 등을 결정하는 데 사용됩니다.

대표적인 형태:

```http
Content-Disposition: inline
```

또는:

```http
Content-Disposition: attachment
```

파일명을 함께 지정할 수도 있습니다.

```http
Content-Disposition: attachment; filename="report.pdf"
```

의미:

```text
이 콘텐츠를 페이지 내부에서 표시하기보다는
report.pdf라는 이름의 파일로 다운로드하도록 처리
```

대표적인 파일 다운로드 응답:

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="report.pdf"
Content-Length: 238174
```

여기서:

```text
Content-Type
→ 데이터 자체가 무엇인가?

Content-Disposition
→ 그 데이터를 어떻게 제시할 것인가?
```

라는 차이가 있습니다.

---

# 4. Accept

`Accept`는 클라이언트가 서버에게:

> 어떤 Media Type의 Response를 받을 수 있거나 선호하는가?

를 알려주는 Request Header입니다.

예:

```http
Accept: application/json
```

의 의미:

```text
가능하면 JSON Response를 보내주세요.
```

여러 형식을 지정할 수도 있습니다.

```http
Accept: text/html, application/xhtml+xml, application/json
```

품질값 `q`를 사용할 수도 있습니다.

```http
Accept: text/html, application/json;q=0.9
```

개념적으로:

```text
text/html
우선순위 1.0

application/json
우선순위 0.9
```

입니다.

이러한 과정을 **Content Negotiation**이라고 합니다.

---

# PART 4. CORS 관련 Header

CORS는 브라우저의 Same-Origin Policy 때문에 필요한 HTTP 메커니즘입니다.

핵심 구조는:

```text
Browser
   │
   │ Origin
   ▼
Server
   │
   │ Access-Control-Allow-*
   ▼
Browser가 Response를 JavaScript에 공개할지 결정
```

입니다.

CORS는 서버가 Request 자체를 막는 기술이라고만 이해하면 안 됩니다.

> CORS의 핵심 집행 주체는 브라우저입니다.

MDN에서도 `Access-Control-Allow-Origin`을 서버가 특정 origin 또는 `*`에 대해 브라우저 접근을 허용하는 Response Header로 설명하며, origin별 동적 응답에서는 `Vary: Origin` 사용을 권장합니다. ([MDN Web Docs][1])

---

# 5. Origin

`Origin`은 요청이 **어느 origin에서 시작되었는지** 알려주는 Request Header입니다.

예:

```http
Origin: https://frontend.example.com
```

origin은 개념적으로:

```text
scheme + host + port
```

입니다.

예:

```text
https://example.com:443
```

Origin 비교에서는 단순 hostname만 비교하는 것이 아닙니다.

```text
http://example.com
https://example.com
```

은 scheme이 다르므로 다른 origin입니다.

```text
https://example.com:443
https://example.com:8443
```

도 port가 다르므로 다른 origin입니다.

---

# 6. Access-Control-Allow-Origin

서버가 특정 origin에서 온 브라우저 JavaScript에게 Response 접근을 허용한다는 것을 나타냅니다.

예:

```http
Access-Control-Allow-Origin: https://frontend.example.com
```

또는 credentials를 사용하지 않는 상황에서:

```http
Access-Control-Allow-Origin: *
```

를 사용할 수 있습니다.

중요한 규칙:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

조합을 credentials가 포함되는 CORS 요청에 사용할 수 있다고 생각하면 안 됩니다.

credentials를 허용한다면 일반적으로 명시적인 origin을 반환해야 합니다.

```http
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Credentials: true
```

---

# 7. Access-Control-Allow-Credentials

브라우저가 cross-origin 요청에서 credential을 포함하고 Response를 노출할 수 있도록 서버가 허용하는 Header입니다.

```http
Access-Control-Allow-Credentials: true
```

여기서 credentials에는 대표적으로:

```text
Cookie
HTTP Authentication credentials
TLS client certificate 관련 credential
```

등이 관련될 수 있습니다.

Fetch에서 쿠키를 cross-origin으로 보내려면 클라이언트 측 설정도 필요할 수 있습니다.

예:

```javascript
fetch("https://api.example.com/users", {
  credentials: "include"
})
```

서버:

```http
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Credentials: true
```

즉 서버 Header 하나만 설정한다고 끝나는 문제가 아닙니다.

---

# 8. Access-Control-Allow-Methods

Preflight Response에서 서버가 허용하는 HTTP Method를 브라우저에 알려줍니다.

```http
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

예를 들어 브라우저가:

```http
Access-Control-Request-Method: DELETE
```

라고 문의하면 서버는:

```http
Access-Control-Allow-Methods: GET, POST, DELETE
```

등으로 응답할 수 있습니다.

---

# 9. Access-Control-Allow-Headers

브라우저가 실제 CORS Request에서 사용할 수 있도록 서버가 허용하는 Request Header 목록입니다.

예:

```http
Access-Control-Allow-Headers: Authorization, Content-Type
```

클라이언트가:

```http
Authorization: Bearer ...
```

을 보내려고 하는 경우 Preflight가 발생할 수 있으며 서버는 해당 Header 사용을 허용해야 합니다.

---

# 10. Access-Control-Max-Age

Preflight 결과를 브라우저가 얼마 동안 캐시할 수 있는지를 지정합니다.

예:

```http
Access-Control-Max-Age: 3600
```

개념적으로:

```text
OPTIONS Preflight 성공
       │
       ▼
결과를 일정 시간 캐시
       │
       ▼
같은 조건의 요청마다
매번 OPTIONS를 보내는 비용 감소
```

입니다.

브라우저는 자체 상한을 적용할 수도 있으므로 서버가 매우 큰 값을 설정한다고 반드시 그대로 적용되는 것은 아닙니다.

---

# 11. Access-Control-Request-Method

Preflight Request에서 브라우저가:

> 실제 Request에서 이 HTTP Method를 사용하려고 한다.

라고 알려주는 Header입니다.

예:

```http
OPTIONS /api/users/1 HTTP/1.1
Origin: https://frontend.example.com
Access-Control-Request-Method: DELETE
```

---

# 12. Access-Control-Request-Headers

Preflight Request에서 실제 Request에 사용할 Header들을 서버에 알려줍니다.

예:

```http
Access-Control-Request-Headers: authorization, content-type
```

전체 흐름:

```text
Browser

OPTIONS /api/users
Origin: https://frontend.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type

                ↓

Server

Access-Control-Allow-Origin: https://frontend.example
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Authorization, Content-Type

                ↓

Browser

POST /api/users
Authorization: Bearer ...
Content-Type: application/json
```

---

# 13. Vary

`Vary`는 CORS 전용 Header가 아닙니다.

HTTP cache에게:

> 이 Response는 특정 Request Header 값에 따라 달라질 수 있다.

라고 알려주는 Response Header입니다.

예:

```http
Vary: Accept-Encoding
```

또는 CORS 환경에서:

```http
Vary: Origin
```

서버가 다음과 같이 origin에 따라 `Access-Control-Allow-Origin`을 동적으로 생성한다면:

```text
Origin A
→ ACAO: Origin A

Origin B
→ ACAO: Origin B
```

중간 cache가 두 Response를 같은 것으로 취급하면 문제가 발생할 수 있습니다.

그래서:

```http
Vary: Origin
```

을 지정합니다.

MDN 역시 특정 origin을 동적으로 반환하는 경우 `Vary: Origin`을 함께 보내야 cache가 origin별 Response 차이를 인식할 수 있다고 설명합니다. ([MDN Web Docs][1])

---

# PART 5. 인증과 상태 관리

| Header             | 방향       | 역할                 |
| ------------------ | -------- | ------------------ |
| `Cookie`           | Request  | 저장된 Cookie를 서버로 전달 |
| `Set-Cookie`       | Response | 브라우저에 Cookie 저장 요청 |
| `Authorization`    | Request  | 인증 Credential 전달   |
| `WWW-Authenticate` | Response | 인증 방식/Challenge 전달 |

---

# 14. Cookie

브라우저가 저장하고 있던 Cookie를 서버로 전달합니다.

```http
Cookie: sessionId=abc123
```

여러 Cookie:

```http
Cookie: sessionId=abc123; theme=dark
```

중요한 구조:

```text
Server
  │
  │ Set-Cookie
  ▼
Browser Cookie Storage

다음 Request

Browser
  │
  │ Cookie
  ▼
Server
```

즉:

```text
Set-Cookie
Server → Browser

Cookie
Browser → Server
```

입니다.

---

# 15. Set-Cookie

서버가 브라우저에게 Cookie를 저장하도록 지시합니다.

```http
Set-Cookie: sessionId=abc123
```

주요 속성:

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Lax
```

`HttpOnly`:

```text
JavaScript document.cookie 접근 제한
```

`Secure`:

```text
Secure context를 중심으로 전송
```

`SameSite`:

```text
cross-site 환경에서 Cookie 전송 정책 제어
```

대표 값:

```text
Strict
Lax
None
```

`SameSite=None`은 현대 브라우저 환경에서 일반적으로 `Secure`와 함께 사용됩니다.

---

# 16. Authorization

클라이언트가 서버에 인증 정보를 전달합니다.

Bearer Token:

```http
Authorization: Bearer eyJhbGciOi...
```

Basic Authentication:

```http
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

중요:

> Basic Authentication의 Base64는 암호화가 아닙니다.

따라서 HTTPS 없이 credential을 보호하는 수단으로 생각하면 안 됩니다.

JWT 기반 REST API에서는 흔히:

```text
Login
  ↓
Access Token 발급
  ↓
Authorization: Bearer Token
  ↓
Protected API
```

구조를 사용합니다.

---

# 17. WWW-Authenticate

서버가 클라이언트에게 필요한 Authentication scheme을 알려줍니다.

대표적으로 `401 Unauthorized` Response와 함께 볼 수 있습니다.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="admin"
```

또는 Bearer 기반 환경에서 challenge 정보를 전달할 수 있습니다.

개념적으로:

```text
Authorization
Client → Server
"이 credential로 인증합니다."

WWW-Authenticate
Server → Client
"이 인증 방식이 필요합니다."
```

---

# PART 6. Routing과 Proxy 관련 Header

---

# 18. Host

HTTP Request가 어느 host를 대상으로 하는지 나타냅니다.

```http
Host: www.example.com
```

port가 포함될 수도 있습니다.

```http
Host: example.com:8080
```

하나의 Web Server IP에서 여러 domain을 서비스하는 Virtual Host 환경에서 특히 중요합니다.

```text
203.0.113.10

Host: shop.example.com
        ↓
Shop Application

Host: blog.example.com
        ↓
Blog Application
```

HTTP/2와 HTTP/3에서는 `:authority` pseudo-header가 이 역할과 밀접하게 연결됩니다.

---

# 19. Location

`Location`은 주로 redirect 대상 URL을 전달하는 Response Header입니다.

```http
HTTP/1.1 302 Found
Location: /login
```

브라우저:

```text
GET /private
     ↓
302
Location: /login
     ↓
GET /login
```

Resource 생성에서도 사용할 수 있습니다.

```http
HTTP/1.1 201 Created
Location: /users/100
```

의미:

```text
새로 생성된 Resource 위치
→ /users/100
```

---

# 20. Referer

현재 요청이 어떤 페이지에서 유도되었는지에 관한 정보를 전달합니다.

```http
Referer: https://www.example.com/products
```

주의할 점은 철자입니다.

정상 영어 단어는:

```text
referrer
```

이지만 HTTP Header의 역사적인 표준 이름은:

```text
Referer
```

입니다.

따라서 Header 이름을 임의로:

```http
Referrer:
```

로 바꾸면 안 됩니다.

전송 범위는 `Referrer-Policy`의 영향을 받습니다.

---

# 21. X-Forwarded-For

Reverse Proxy를 거치는 환경에서 원래 Client IP와 Proxy chain 정보를 전달하기 위해 널리 사용되는 비표준 Header입니다.

예:

```http
X-Forwarded-For: 203.0.113.10
```

Proxy가 여러 개라면:

```http
X-Forwarded-For: 203.0.113.10, 10.0.0.4, 10.0.0.8
```

처럼 누적될 수 있습니다.

개념:

```text
Client
203.0.113.10
     ↓
Proxy A
     ↓
Proxy B
     ↓
Application

X-Forwarded-For:
203.0.113.10, Proxy-A-IP
```

중요한 보안 원칙:

> 외부 클라이언트가 직접 보낸 `X-Forwarded-For`를 무조건 신뢰해서는 안 됩니다.

신뢰할 수 있는 Reverse Proxy가 Header를 어떻게 생성·덮어쓰는지까지 함께 설정해야 합니다.

---

# 22. X-Forwarded-Proto

원래 Client가 사용한 protocol scheme을 전달합니다.

예:

```http
X-Forwarded-Proto: https
```

대표적인 구조:

```text
Browser
   │ HTTPS
   ▼
Nginx
   │ HTTP
   ▼
Spring Boot
```

Spring Boot에서 직접 연결만 보면:

```text
HTTP
```

처럼 보일 수 있습니다.

하지만:

```http
X-Forwarded-Proto: https
```

를 통해 원래 사용자가 HTTPS로 접속했다는 정보를 전달할 수 있습니다.

---

# 23. X-Forwarded-Host

원래 Client가 요청했던 Host 정보를 Proxy 뒤의 서버로 전달합니다.

```http
X-Forwarded-Host: www.example.com
```

Reverse Proxy 내부에서는 실제 backend host가:

```text
app:8080
```

일 수 있지만 사용자가 접근한 공개 Host는:

```text
www.example.com
```

일 수 있습니다.

이 차이를 전달하는 데 사용합니다.

---

# 24. X-Real-IP

`X-Real-IP`는 Nginx 등에서 많이 사용하는 비표준 Header입니다.

예:

```http
X-Real-IP: 203.0.113.10
```

`X-Forwarded-For`와 비교하면:

```text
X-Real-IP

203.0.113.10

보통 단일 IP
```

반면:

```text
X-Forwarded-For

203.0.113.10, 10.0.0.4, 10.0.0.8

Proxy를 거치며 chain을 표현할 수 있음
```

둘은 같은 Header가 아닙니다.

---

# PART 7. WebSocket Opening Handshake Header

WebSocket 연결은 처음부터 WebSocket frame으로 시작하는 것이 아닙니다.

초기 연결 과정에서는 **HTTP Opening Handshake**가 사용됩니다.

```text
Client
   │
   │ HTTP Request
   │ Upgrade: websocket
   ▼
Server
   │
   │ HTTP 101
   │ Upgrade: websocket
   ▼
WebSocket Connection
```

MDN도 WebSocket opening handshake가 HTTP 요청으로 시작되고 `Upgrade`, `Connection`, `Sec-WebSocket-Key` 등이 사용된다고 설명합니다. ([MDN Web Docs][2])

---

# 25. Upgrade

클라이언트가 현재 HTTP connection에서 다른 protocol로 전환하고 싶다는 의도를 전달하는 데 사용됩니다.

WebSocket:

```http
Upgrade: websocket
```

대표 handshake:

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
```

HTTP/1.1의 protocol upgrade mechanism과 관련된 Header라는 점이 중요합니다.

---

# 26. Connection

HTTP/1.1에서 현재 connection에만 적용되는 option을 지정합니다.

WebSocket handshake에서:

```http
Connection: Upgrade
```

는:

```text
Upgrade Header를
현재 connection의 upgrade 처리에 사용한다.
```

는 의미와 연결됩니다.

매우 중요한 현대 HTTP 주의점:

> `Connection`은 HTTP/2와 HTTP/3에서 사용하면 안 되는 connection-specific field입니다.

따라서 WebSocket의 전통적인:

```http
Connection: Upgrade
Upgrade: websocket
```

형태는 **HTTP/1.1 opening handshake 문맥**으로 이해하는 것이 좋습니다.

---

# 27. Sec-WebSocket-Key

Client가 WebSocket Opening Handshake에서 전송합니다.

예:

```http
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
```

브라우저가 임의 nonce를 생성하여 Base64 형태로 전달합니다.

MDN에 따르면 이 값은 브라우저가 생성하는 Base64 인코딩된 16-byte nonce이며, 보안을 위한 비밀키가 아니라 서버가 WebSocket handshake를 올바르게 처리했는지 검증하는 절차의 일부입니다. ([MDN Web Docs][2])

따라서 이름에 `Key`가 들어간다고:

```text
암호화 키
```

라고 이해하면 안 됩니다.

---

# 28. Sec-WebSocket-Accept

Server가 `Sec-WebSocket-Key`를 바탕으로 계산하여 Response에 보냅니다.

```http
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

계산 과정은 개념적으로:

```text
Sec-WebSocket-Key
        +
WebSocket GUID
        ↓
SHA-1
        ↓
Base64
        ↓
Sec-WebSocket-Accept
```

입니다.

표준 GUID:

```text
258EAFA5-E914-47DA-95CA-C5AB0DC85B11
```

MDN이 설명하는 공식 계산 방식도 `Sec-WebSocket-Key + GUID → SHA-1 → Base64`입니다. ([MDN Web Docs][3])

---

# 29. Sec-WebSocket-Version

Client가 사용하는 WebSocket protocol version을 전달합니다.

일반적으로:

```http
Sec-WebSocket-Version: 13
```

을 볼 수 있습니다.

현재 일반적인 WebSocket handshake에서 version 13이 사용됩니다.

서버가 해당 version을 지원하지 않는 경우 자신이 지원하는 version 정보를 응답에 제시하는 데도 관련됩니다. ([MDN Web Docs][4])

---

# 30. Sec-WebSocket-Extensions

Client와 Server가 WebSocket Extension을 협상합니다.

대표적인 예:

```http
Sec-WebSocket-Extensions: permessage-deflate
```

`permessage-deflate`는 WebSocket message compression에 사용될 수 있습니다.

개념:

```text
Client
"이 Extension을 지원합니다."

        ↓

Server
"그 Extension을 사용합시다."
```

---

# 31. Sec-WebSocket-Protocol

WebSocket의 **Subprotocol**을 협상합니다.

예:

```http
Sec-WebSocket-Protocol: chat, superchat
```

Client:

```text
chat
superchat
지원 가능
```

Server:

```http
Sec-WebSocket-Protocol: chat
```

즉:

```text
WebSocket
=
Transport protocol

STOMP
GraphQL protocol
Custom chat protocol
...
=
WebSocket 위에서 사용할 수 있는 상위 protocol
```

이라는 구조를 이해하는 것이 중요합니다.

MDN도 Request에서는 클라이언트가 지원하는 subprotocol을 선호 순서대로 제시하고 Response에서는 Server가 선택한 subprotocol을 반환한다고 설명합니다. ([MDN Web Docs][5])

---

# PART 8. WebSocket Handshake 전체 예제

Client:

```http
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat
```

Server:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

그 이후:

```text
HTTP Request / Response
          │
          │ 101 Switching Protocols
          ▼
────────────────────────
    WebSocket Frames
────────────────────────
```

로 전환됩니다.

---

# PART 9. Cache와 Conditional Request

이번 그룹은 반드시 흐름으로 이해해야 합니다.

```text
Cache-Control
      │
      ├─ freshness
      │
      ▼
Cache가 stale
      │
      ▼
Conditional Request
      │
      ├─ If-None-Match
      │       ↑
      │      ETag
      │
      └─ If-Modified-Since
              ↑
         Last-Modified
```

MDN 역시 stale response를 서버에 검증하는 과정을 validation/revalidation이라고 설명하고 `If-None-Match` 또는 `If-Modified-Since`를 사용한다고 설명합니다. ([MDN Web Docs][6])

---

# 32. Cache-Control

HTTP cache 동작을 지정합니다.

예:

```http
Cache-Control: max-age=3600
```

의미:

```text
Response를 최대 3600초 동안 fresh로 취급 가능
```

대표 directive:

```text
max-age
no-cache
no-store
private
public
must-revalidate
```

---

## max-age

```http
Cache-Control: max-age=3600
```

일정 시간 동안 fresh response로 재사용할 수 있습니다.

---

## no-cache

이름 때문에 매우 자주 오해합니다.

```http
Cache-Control: no-cache
```

는 단순히:

```text
절대로 저장하지 마라.
```

라는 뜻이 아닙니다.

핵심은:

```text
저장된 Response를 재사용하기 전에
origin server에 validation하라.
```

입니다.

---

## no-store

```http
Cache-Control: no-store
```

는 cache에 저장 자체를 하지 말라는 의미에 가깝습니다.

따라서:

```text
no-cache
≠
no-store
```

입니다.

---

## private

```http
Cache-Control: private
```

Shared Cache보다 개인 사용자에 해당하는 private cache에서 처리해야 하는 Response임을 나타냅니다.

---

## public

```http
Cache-Control: public
```

shared cache에서도 cache 가능한 Response임을 명시적으로 나타내는 데 사용할 수 있습니다.

---

# 33. ETag

Resource의 특정 representation/version을 식별하는 validator입니다.

```http
ETag: "v123"
```

또는:

```http
ETag: "33a64df5"
```

Server가 ETag를 어떤 방식으로 생성하는지는 구현에 따라 달라질 수 있습니다.

예:

```text
Resource Version
       ↓
ETag: "product-v21"
```

Client는 이후:

```http
If-None-Match: "product-v21"
```

를 보낼 수 있습니다.

---

# 34. If-None-Match

Client가 이전에 받은 ETag를 이용하여 조건부 Request를 보냅니다.

```http
If-None-Match: "product-v21"
```

Server:

```text
현재 ETag == product-v21
```

이라면 일반적인 GET/HEAD cache revalidation에서는:

```http
HTTP/1.1 304 Not Modified
```

를 반환할 수 있습니다.

Body를 다시 내려보낼 필요가 없습니다.

MDN도 `ETag`가 일치하는 경우 Server가 `304 Not Modified`를 보내고 Client가 기존 cached response를 계속 사용할 수 있다고 설명합니다. ([MDN Web Docs][7])

---

# 35. Last-Modified

Resource가 마지막으로 수정된 시각을 알려줍니다.

```http
Last-Modified: Tue, 01 Sep 2026 10:00:00 GMT
```

Client는 이 값을 저장했다가:

```http
If-Modified-Since: Tue, 01 Sep 2026 10:00:00 GMT
```

형태로 보낼 수 있습니다.

---

# 36. If-Modified-Since

특정 시점 이후 Resource가 변경되었는지를 확인하는 Conditional Request Header입니다.

```http
If-Modified-Since: Tue, 01 Sep 2026 10:00:00 GMT
```

Resource가 변경되지 않았다면:

```http
HTTP/1.1 304 Not Modified
```

변경되었다면:

```http
HTTP/1.1 200 OK
```

와 새로운 representation을 받을 수 있습니다.

`If-None-Match`와 `If-Modified-Since`가 함께 있는 경우 `If-None-Match`가 우선됩니다. MDN에서도 이 우선순위를 명시합니다. ([MDN Web Docs][6])

---

# PART 10. ETag Cache 전체 흐름

첫 Request:

```http
GET /products/1 HTTP/1.1
```

Server:

```http
HTTP/1.1 200 OK
ETag: "v10"
Cache-Control: max-age=60
Content-Type: application/json

{
  "id": 1,
  "name": "Keyboard"
}
```

Client Cache:

```text
/products/1

ETag = "v10"
Body = {...}
```

Cache가 stale이 된 후:

```http
GET /products/1 HTTP/1.1
If-None-Match: "v10"
```

Resource가 변경되지 않았다면:

```http
HTTP/1.1 304 Not Modified
ETag: "v10"
```

결과:

```text
Network Body Download
        ↓
     생략 가능

Cached Body
        ↓
      재사용
```

이것이 HTTP Conditional Request의 핵심입니다.

---

# PART 11. 진단과 비표준 Header

---

# 37. User-Agent

Client software에 대한 정보를 서버에 전달합니다.

대표적인 Browser Request:

```http
User-Agent: Mozilla/5.0 ...
```

전통적으로 다음과 같은 정보가 포함될 수 있었습니다.

```text
Browser
Browser Version
OS
Rendering Engine
Device 환경
```

하지만 `User-Agent` 문자열은 역사적으로 복잡해졌으며 Browser fingerprinting과 privacy 문제도 있습니다.

현대 Browser에서는 Client Hints가 보완 또는 대체 메커니즘으로 사용됩니다.

---

# 38. X-Requested-With

역사적으로 Ajax Request임을 구별하기 위해 많은 JavaScript framework/library에서 사용했던 비표준 Header입니다.

대표 값:

```http
X-Requested-With: XMLHttpRequest
```

과거 서버 코드에서:

```text
일반 Navigation Request인가?

Ajax Request인가?
```

를 구분하는 데 사용되었습니다.

하지만 Fetch API 시대에는 이것을 현대 HTTP의 필수 Header로 생각하면 안 됩니다.

또한 custom request header이므로 cross-origin 환경에서는 CORS 동작에도 영향을 줄 수 있습니다.

---

# PART 12. Web Security Response Header

웹 보안 Header들은 Browser에게:

```text
이 Response를 어떤 보안 정책으로 처리해야 하는가?
```

를 알려줍니다.

---

# 39. Strict-Transport-Security

보통 HSTS라고 부릅니다.

```http
Strict-Transport-Security: max-age=31536000
```

Browser에게:

```text
앞으로 일정 기간 이 host에 HTTP가 아니라
HTTPS로 접속해야 한다.
```

는 정책을 전달합니다.

예:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

HSTS는 HTTPS Response에서 의미 있게 처리되는 정책이라는 점이 중요합니다.

---

# 40. X-Content-Type-Options

대표 설정:

```http
X-Content-Type-Options: nosniff
```

Browser에게 Response의 MIME Type을 임의로 추측하는 MIME sniffing을 제한하도록 합니다.

예:

```http
Content-Type: text/plain
X-Content-Type-Options: nosniff
```

의 경우 Browser가:

```text
내용을 보니 JavaScript 같군.
JavaScript로 실행해보자.
```

와 같은 식의 해석을 하지 않도록 보안성을 높이는 데 도움을 줍니다.

---

# 41. X-Frame-Options

현재 페이지가 다른 페이지의 `<frame>`, `<iframe>` 등에 포함되는 것을 제한하여 Clickjacking 공격 완화에 사용됩니다.

대표 값:

```http
X-Frame-Options: DENY
```

또는:

```http
X-Frame-Options: SAMEORIGIN
```

`DENY`:

```text
어떤 사이트도 frame으로 포함 금지
```

`SAMEORIGIN`:

```text
동일 origin에서만 frame 포함 허용
```

현대 애플리케이션에서는 CSP의 `frame-ancestors`도 함께 알아두어야 합니다.

---

# 42. Content-Security-Policy

줄여서 **CSP**입니다.

Browser가 어떤 source에서 script, style, image, frame 등을 로드할 수 있는지를 제한합니다.

예:

```http
Content-Security-Policy: default-src 'self'
```

조금 더 구체적으로:

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.example.com;
  img-src 'self' https:;
```

개념:

```text
Browser

script 가져오려 함
        ↓
CSP 확인
        ↓
허용 Source?
  │
  ├─ YES → Load
  │
  └─ NO  → Block
```

XSS 공격 완화에 매우 중요한 방어 계층입니다.

하지만 CSP 하나만으로 XSS 문제 전체가 해결되는 것은 아닙니다.

---

# 43. Referrer-Policy

Browser가 Request의 `Referer` Header에 얼마나 많은 정보를 포함할지 제어합니다.

예:

```http
Referrer-Policy: no-referrer
```

또는:

```http
Referrer-Policy: strict-origin-when-cross-origin
```

즉:

```text
Referer
=
실제로 전달되는 정보

Referrer-Policy
=
얼마나 전달할지를 결정하는 정책
```

이라는 관계입니다.

---

# 44. Cross-Origin-Opener-Policy

줄여서 **COOP**입니다.

대표 설정:

```http
Cross-Origin-Opener-Policy: same-origin
```

Browser의 browsing context group 격리를 제어하여 cross-origin window 간 관계를 제한하는 보안 메커니즘입니다.

예를 들어 `window.open()` 등으로 연결되는 다른 browsing context와의 관계를 격리하는 데 중요한 역할을 합니다.

COOP는 다음과 같은 현대 Browser isolation 기술과 함께 공부하면 좋습니다.

```text
COOP
Cross-Origin-Opener-Policy

COEP
Cross-Origin-Embedder-Policy

CORP
Cross-Origin-Resource-Policy
```

이번 강의 범위에는 COOP만 포함되지만 세 정책은 고급 Web Security에서 함께 등장합니다.

---

# PART 13. Client Hints

원문에서 추가로 등장한:

```text
Sec-CH-UA-*
```

는 하나의 Header가 아니라 **Client Hints Header Family**입니다.

대표적으로 다음과 같은 형태가 있습니다.

```http
Sec-CH-UA:
Sec-CH-UA-Mobile:
Sec-CH-UA-Platform:
```

상황에 따라 더 세부적인 User-Agent Client Hints가 사용될 수 있습니다.

목적은 전통적인 거대한:

```http
User-Agent: Mozilla/5.0 ...
```

문자열 하나에 모든 정보를 몰아 넣는 방식을 개선하는 것입니다.

개념:

```text
Traditional

User-Agent
    │
    └─ 많은 환경 정보가 하나의 문자열
```

Client Hints:

```text
Sec-CH-UA
Sec-CH-UA-Mobile
Sec-CH-UA-Platform
...

필요한 정보를 구조화된 Header로 전달
```

단, privacy 및 fingerprinting 문제 때문에 모든 정보를 항상 자동으로 노출하는 구조는 아닙니다.

---

# PART 14. STOMP Frame Header는 HTTP Header가 아니다

WebSocket과 STOMP를 함께 공부할 때 매우 중요한 부분입니다.

다음과 같은 값:

```text
destination
subscription
message-id
ack
content-type
```

등은 STOMP Frame 안에서 사용될 수 있습니다.

예:

```text
MESSAGE
destination:/topic/chat
subscription:sub-0
message-id:007
content-type:application/json

{"message":"hello"}
```

여기서:

```text
destination
subscription
message-id
```

는 **HTTP Header가 아닙니다.**

계층을 구분해야 합니다.

```text
처음 연결

HTTP
│
├─ Upgrade
├─ Connection
├─ Sec-WebSocket-Key
└─ Sec-WebSocket-Version

            ↓

101 Switching Protocols

            ↓

WebSocket Protocol
│
└─ WebSocket Frames
        │
        └─ STOMP Frame
             │
             ├─ destination
             ├─ subscription
             └─ message-id
```

즉:

> WebSocket handshake Header와 STOMP Frame Header는 서로 다른 protocol layer의 Header입니다.

---

# PART 15. Reverse Proxy 실전

Nginx가 Client Request를 Spring Boot로 전달한다고 해보겠습니다.

```text
Internet

Browser
  │
  │ HTTPS
  ▼
Nginx
  │
  │ HTTP
  ▼
Spring Boot
```

Nginx 설정은 개념적으로 다음과 같은 Header를 전달할 수 있습니다.

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

Spring Boot가 보는 정보:

```text
Host
→ 사용자가 요청한 host

X-Real-IP
→ Client IP

X-Forwarded-For
→ Proxy chain 정보

X-Forwarded-Proto
→ 원래 Client protocol
```

이 정보는 다음과 같은 기능에 영향을 줄 수 있습니다.

```text
Redirect URL 생성
HTTPS 판단
Access Log
Client IP 확인
Security 정책
OAuth Redirect URL
```

---

# PART 16. CORS 전체 실전 흐름

React:

```text
https://frontend.example.com
```

Spring Boot:

```text
https://api.example.com
```

서로 다른 origin입니다.

Browser:

```http
OPTIONS /api/users HTTP/1.1
Origin: https://frontend.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type
```

Server:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 3600
Vary: Origin
```

Browser는 Preflight 결과를 확인한 뒤 실제 Request를 보냅니다.

```http
POST /api/users HTTP/1.1
Origin: https://frontend.example.com
Authorization: Bearer eyJ...
Content-Type: application/json
Cookie: SESSION=abc...

{
  "name": "Hong"
}
```

Server:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Credentials: true
Vary: Origin
Location: /api/users/100

{
  "id": 100,
  "name": "Hong"
}
```

이 하나의 흐름 안에서:

```text
Origin
Access-Control-Request-Method
Access-Control-Request-Headers
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Access-Control-Allow-Credentials
Access-Control-Max-Age
Vary
Authorization
Cookie
Content-Type
Location
```

등이 서로 연결됩니다.

---

# PART 17. 44개 HTTP Header 전체 인벤토리

| No. | Header                             | 주요 방향   | 핵심 역할                           |
| --: | ---------------------------------- | ------- | ------------------------------- |
|   1 | `Content-Type`                     | Req/Res | Body Media Type                 |
|   2 | `Content-Length`                   | Req/Res | Body byte 길이                    |
|   3 | `Content-Disposition`              | 주로 Res  | inline/download 처리              |
|   4 | `Accept`                           | Req     | 원하는 Response Media Type         |
|   5 | `Origin`                           | Req     | 요청이 시작된 origin                  |
|   6 | `Access-Control-Allow-Origin`      | Res     | 허용 origin                       |
|   7 | `Access-Control-Allow-Credentials` | Res     | credential 허용                   |
|   8 | `Access-Control-Allow-Methods`     | Res     | CORS 허용 method                  |
|   9 | `Access-Control-Allow-Headers`     | Res     | CORS 허용 header                  |
|  10 | `Access-Control-Max-Age`           | Res     | Preflight cache 시간              |
|  11 | `Access-Control-Request-Method`    | Req     | 실제 사용할 method                   |
|  12 | `Access-Control-Request-Headers`   | Req     | 실제 사용할 header                   |
|  13 | `Vary`                             | Res     | Request Header별 cache variation |
|  14 | `Cookie`                           | Req     | 저장된 Cookie 전송                   |
|  15 | `Set-Cookie`                       | Res     | Cookie 저장 지시                    |
|  16 | `Authorization`                    | Req     | 인증 credential                   |
|  17 | `WWW-Authenticate`                 | Res     | 인증 challenge                    |
|  18 | `Host`                             | Req     | 대상 host                         |
|  19 | `Location`                         | Res     | Redirect/Resource URI           |
|  20 | `Referer`                          | Req     | 이전 문서 정보                        |
|  21 | `X-Forwarded-For`                  | Req     | Proxy chain Client IP           |
|  22 | `X-Forwarded-Proto`                | Req     | 원래 protocol                     |
|  23 | `X-Forwarded-Host`                 | Req     | 원래 Host                         |
|  24 | `X-Real-IP`                        | Req     | 실제 Client IP                    |
|  25 | `Upgrade`                          | Req/Res | Protocol 전환                     |
|  26 | `Connection`                       | Req/Res | HTTP/1.1 connection option      |
|  27 | `Sec-WebSocket-Key`                | Req     | WebSocket handshake nonce       |
|  28 | `Sec-WebSocket-Accept`             | Res     | Server handshake 검증값            |
|  29 | `Sec-WebSocket-Version`            | 주로 Req  | WebSocket version               |
|  30 | `Sec-WebSocket-Extensions`         | Req/Res | Extension 협상                    |
|  31 | `Sec-WebSocket-Protocol`           | Req/Res | Subprotocol 협상                  |
|  32 | `Cache-Control`                    | Req/Res | Cache 정책                        |
|  33 | `ETag`                             | Res     | Resource validator              |
|  34 | `If-None-Match`                    | Req     | ETag 기반 조건부 요청                  |
|  35 | `Last-Modified`                    | Res     | 마지막 수정 시간                       |
|  36 | `If-Modified-Since`                | Req     | 날짜 기반 조건부 요청                    |
|  37 | `User-Agent`                       | Req     | Client software 정보              |
|  38 | `X-Requested-With`                 | Req     | 전통적 Ajax 식별                     |
|  39 | `Strict-Transport-Security`        | Res     | HTTPS 강제 정책                     |
|  40 | `X-Content-Type-Options`           | Res     | MIME sniffing 방지                |
|  41 | `X-Frame-Options`                  | Res     | Frame embedding 제한              |
|  42 | `Content-Security-Policy`          | Res     | Resource 실행/로드 정책               |
|  43 | `Referrer-Policy`                  | Res     | Referer 전송 정책                   |
|  44 | `Cross-Origin-Opener-Policy`       | Res     | Browsing context 격리             |

추가 Header Family:

```text
Sec-CH-UA-*
→ User-Agent Client Hints
```

HTTP Header가 아닌 것:

```text
destination
subscription
message-id
...
→ STOMP Frame Headers
```

---

# PART 18. 반드시 기억해야 하는 Header 관계

HTTP Header는 각각 따로 암기하는 것보다 다음 **관계**로 기억해야 합니다.

```text
Content Negotiation

Accept
    ↓
Server
    ↓
Content-Type
```

```text
Cookie

Set-Cookie
Server → Browser

Cookie
Browser → Server
```

```text
Authentication

Authorization
Client → Server

WWW-Authenticate
Server → Client
```

```text
CORS

Origin
        ↓
Access-Control-Allow-Origin
```

```text
CORS Preflight

Access-Control-Request-Method
        ↓
Access-Control-Allow-Methods

Access-Control-Request-Headers
        ↓
Access-Control-Allow-Headers
```

```text
Cache Validation

ETag
        ↓
If-None-Match

Last-Modified
        ↓
If-Modified-Since
```

```text
WebSocket

Sec-WebSocket-Key
        ↓
Sec-WebSocket-Accept
```

```text
Referrer

Referrer-Policy
        ↓
Referer
```

```text
Reverse Proxy

Client
 ↓
Nginx
 ↓
X-Forwarded-*
X-Real-IP
 ↓
Backend
```

---

# PART 19. HTTP Header를 공부할 때 가장 중요한 관점

44개의 Header를 단순 암기하면 금방 잊어버립니다.

다음 질문으로 접근하면 훨씬 쉽게 이해할 수 있습니다.

```text
① 누가 보내는가?

Client?
Server?
Proxy?
Browser?
```

```text
② 어느 방향인가?

Request?
Response?
둘 다?
```

```text
③ 어떤 계층인가?

HTTP?
WebSocket Handshake?
STOMP Frame?
```

```text
④ 상대방에게 무엇을 알려주는가?

Body 형식?
인증?
Cache?
Security Policy?
Routing?
CORS Permission?
```

```text
⑤ 다른 Header와 어떤 관계인가?

ETag ↔ If-None-Match

Last-Modified ↔ If-Modified-Since

Sec-WebSocket-Key ↔ Sec-WebSocket-Accept

Cookie ↔ Set-Cookie

Origin ↔ Access-Control-Allow-Origin
```

이 관점으로 보면 HTTP Header는 44개의 독립적인 지식이 아니라 서로 연결된 **몇 개의 HTTP 메커니즘**으로 정리됩니다.

---

# 최종 핵심 구조

```text
                         HTTP HEADERS
                              │
       ┌──────────────────────┼───────────────────────┐
       │                      │                       │
       ▼                      ▼                       ▼
 Representation             Security               Routing
       │                      │                       │
 Content-Type              Authorization         Host
 Content-Length            Cookie                Location
 Content-Disposition       Set-Cookie            Referer
 Accept                    CSP                   X-Forwarded-*
                           HSTS                  X-Real-IP
                              │
                              │
       ┌──────────────────────┼───────────────────────┐
       │                      │                       │
       ▼                      ▼                       ▼
      CORS                   Cache                WebSocket
       │                      │                       │
 Origin                   Cache-Control          Upgrade
 Access-Control-*         ETag                   Connection
 Vary                     If-None-Match          Sec-WebSocket-*
                          Last-Modified
                          If-Modified-Since
```

그리고 이 전체를 한 문장으로 정리하면 다음과 같습니다.

> **HTTP Header는 HTTP Request와 Response의 Body 자체가 아니라, 그 Message를 어떻게 해석하고 전달하고 인증하고 캐시하고 보안 처리하고 연결할 것인지를 클라이언트·서버·프록시가 서로 전달하기 위한 메타데이터입니다.**

이 원리를 먼저 이해하면 `Content-Type`, CORS, Cookie, JWT, Cache, Reverse Proxy, WebSocket, Security Header가 서로 전혀 다른 주제가 아니라 **HTTP Message를 제어하는 하나의 큰 시스템**으로 연결됩니다.

이 자료는 특히 **`no-cache ≠ no-store`**, **`Referer`의 역사적 철자**, **`X-Forwarded-For` 신뢰 문제**, **WebSocket의 `Sec-WebSocket-Key`가 암호화 키가 아니라는 점**, **HTTP/2·3에서는 `Connection` 같은 connection-specific header를 사용하지 않는다는 점**, **STOMP frame header와 HTTP header의 계층 차이**까지 포함해서 강의용으로 잡았습니다. Cache의 `ETag → If-None-Match → 304` 관계와 `Last-Modified → If-Modified-Since → 304` 관계 역시 HTTP 표준 동작에 맞춰 정리했습니다. ([MDN Web Docs][6])

현재 분량이라면 **한 파일에 전부 넣기보다는 실제 강의자료는 8~10개의 PART로 분리하는 편이 훨씬 좋습니다.** 특히 지금까지 만든 HTTP Header 이미지 세트와 결합한다면 `HTTP Header 기초 → 콘텐츠 → CORS → 인증/Cookie → Proxy → WebSocket → Cache → Security → 전체 실전 흐름` 순서가 가장 자연스럽습니다.

[1]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS?utm_source=chatgpt.com "Cross-Origin Resource Sharing (CORS) - HTTP | MDN"
[2]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-WebSocket-Key?utm_source=chatgpt.com "Sec-WebSocket-Key header - HTTP | MDN"
[3]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Sec-WebSocket-Accept?utm_source=chatgpt.com "Sec-WebSocket-Accept header - HTTP | MDN"
[4]: https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API?utm_source=chatgpt.com "WebSocket API (WebSockets) - Web APIs | MDN"
[5]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Protocol_upgrade_mechanism?utm_source=chatgpt.com "Protocol upgrade mechanism - HTTP | MDN"
[6]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching?utm_source=chatgpt.com "HTTP caching - HTTP | MDN"
[7]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/ETag?utm_source=chatgpt.com "ETag header - HTTP | MDN"
