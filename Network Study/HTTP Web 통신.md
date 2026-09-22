[[>Security]]
## 1. Client-Server

클라이언트 —(프록시)요청—> 서버

클라이언트 <—(프록시)응답— 서버

- 클라이언크: 요청 보내는 쪽 (브라우저, 앱, …)
- 서버: 요청 받아서 처리, 응답 돌려줌
- 프록시: client와 server 사이에서 data를 중계하는 중간 서버
    - 접속 우회, IP 숨기기를 통한 익명성 확보, waf 통한 보안 강화, 캐싱 통한 성능 향상

서버 입장에서 클라이언트, text 뭉치가 들어온 것 뿐임.

그 text를 parsing 해서 의미를 해석해야 함.

→ 해석 과정에서 공격 발생 여지 생김 (text 조작, server 의도하지 않은 방식으로 해석됨)

WAF, text가 server에 도달하기 전 미리 걸러냄

클라이언트 - - WAF - - 요청—> 서버

## 2. HTTP protocol structure

### 2-1. HTTP Request structure

HTTP: client와 server 통신 시 사용하는 프로토콜

*WAF 공격 시, 4개 영역 중 아무 곳에나 악성 payload 숨길 수 있음.

- ex. header 의 cookie 에 `SQL injection`
- url 경로에 `path traversal`
- body에 `XSS script`

→ WAF rule, 4개 영역을 각각 따로 검사해야 함.

ex.

```jsx
// 1. request line
POST /login HTTP/1.1

// 2. headers (요청 부가정보)
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Content-Type: application/x-www-form-urlencoded
Content-Length: 29
Cookie: session=abc123xyz

// 3. 공백 line

// 4. body (본문, 실제 전송하는 데이터)
username=admin&password=1234
```

**① Request Line (요청 라인)** — 첫 번째 줄

- `POST`: HTTP 메서드 (어떤 동작을 하고 싶은지)
- `/login`: 요청 대상 경로 (URL의 경로 부분)
- `HTTP/1.1`: 사용하는 프로토콜 버전

**② Headers (헤더)** — 요청에 대한 부가 정보

key-value pair 로 이루어짐

- host: 어느 domain 으로 보내는 요청인지 (한 server, 여러 domain 가능)
- user-agent: client, 어떤 브라우저/프로그램인지 (client 가 조작가능, 신뢰x)
- content-type: body의 형식 (form, JSON, file,…)
- content-Length: body의 byte 길이
- cookie: client 가 server에게 다시 보내주는 값 (ex. session 정보)

**③ Body (본문)** — 실제 전송하는 데이터

### 2-2. HTTP Method

|메서드|의미|예시|CRUD|
|---|---|---|---|
|GET|데이터 조회 (읽기)|`GET /products?id=5`|Read|
|POST|데이터 생성/제출|로그인 폼 제출|Create|
|PUT|데이터 전체 수정|프로필 전체 업데이트|Update|
|PATCH|데이터 일부 수정|이름만 변경|Update|
|DELETE|데이터 삭제|게시글 삭제|Delete|

- 각 method 별로 parameter 가 log 에 남는지 여부가 다름
    
    (GET, 파라미터가 url 에 노출 `?id=5`
    
    POST, body 안에 data 있어서 url 노출x `POST /login HTTP/1.1`)
    

### 2-3. HTTP Response 구조

ex.

```jsx
// 1. status line: 버전, 상태 코드, 상태 메시지
HTTP/1.1 200 OK

// 2. Headers: response 에 대한 metadata
Content-Type: text/html; charset=UTF-8
Content-Length: 1256
Set-Cookie: session=abc123xyz; HttpOnly; Secure

// 3. 빈 줄

// 4. body: 실 응답 내용(html, json,...)
<html>
  <body>로그인 성공</body>
</html>
```

### 2-4. Status Code

|그룹|의미|예시|
|---|---|---|
|1xx|정보성 응답|100 Continue|
|2xx|성공|200 OK, 201 Created|
|3xx|리다이렉션|301 Moved Permanently, 302 Found|
|4xx|클라이언트 오류|400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found|
|5xx|서버 오류|500 Internal Server Error|

**보안 관점 포인트**: 공격자, 상태 코드로 서버 상태를 추측.

ex 1. SQL Injection 시도 — `500 Internal Server Error` —> 서버 내부의 쿼리 에러가 남 = SQL Injection 통할 가능성 있음

ex 2. `403 Forbidden` —> WAF 가 요청 차단 시 흔히 반환하는 코드

## 3. URL

```
<https://example.com:443/search?query=laptop&page=2#results>
└─┬─┘   └────┬────┘└┬┘└──┬───┘└──────┬───────────┘└──┬──┘
 스킴      호스트    포트  경로       쿼리 스트링      프래그먼트
```

|구성요소|설명|예시|
|---|---|---|
|Scheme|프로토콜 종류|`https`|
|Host|도메인/IP|`example.com`|
|Port|포트 번호 (생략 시 기본값: http=80, https=443)|`443`|
|Path|서버 내 자원 경로|`/search`|
|Query String|`?`로 시작, `key=value`를 `&`로 연결|`query=laptop&page=2`|
|Fragment|`#`로 시작, 서버로 전송되지 않고 브라우저에서만 사용|`#results`|

**보안 관점 포인트**:

- Query String: WAF 가 가장 많이 검사하는 영역 중 하나.
    - `?id=1' OR '1'='1` 같은 SQL Injection payload.
- Path: Path Traversal 공격 들어감 `/../../etc/passwd`
- Fragment `#` : 서버로 전송 x, WAF 아예 볼 수 없음.

## 4. Cookies and Session

HTTP, 기본적으로 무상태 (stateless) protocol.

서버, 현 요청 보낸 사람이 아까 로그인한 사람인지 기억 x

→ 쿠키/세션 사용

flow.

1. Client, send login request
2. Server, 인증 성공 시 session ID 발급. 응답 header 에 담아 보냄  
    `Set-Cookie: session=abc123xyz; HttpOnly; Secure`
3. 브라우저, 이 cookie 저장, 이후 같은 server로 보내는 모든 request 에 자동 첨부  
    `Cookie: session=abc123xyz`
4. Server, 이 session ID 로 로그인 완료한 사용자임을 판단.

Cookie 속성 (보안).

|속성|의미|
|---|---|
|`HttpOnly`|JavaScript로 쿠키에 접근 불가 → XSS로 쿠키 탈취를 어렵게 함|
|`Secure`|HTTPS 연결에서만 쿠키 전송|
|`SameSite`|다른 사이트에서 온 요청에 쿠키를 첨부할지 제어 → CSRF 방어와 직결|

## 5. HTTPS와 HTTP

```
HTTP:  [클라이언트] --- 평문 ---> [서버]   (도청 가능)
>> 중간에서 누구나 도청/변조 가능

HTTPS: [클라이언트] --- 암호문 ---> [서버]  (도청 불가, 서버에서 복호화)
>> HTTP + TLS(암호화 계층) → 통신 내용이 암호화되어 중간자가 못 봄
```

*HTTPS, “통신 구간” 을 암호화함. 서버 도달한 이후의 요청 “내용 자체” 의 안전여부 x

- SQL Injection 같은 payload가 HTTPS 로 암호화되어 전송—> server/WAF 에 도달해서 복호화되는 순간, 그 안에 있는 공격 payload 그대로 노출됨.
- WAF, TLS가 복호화된 이후 동작 (or TLS 를 직접 종료시키는 위치에서 동작)