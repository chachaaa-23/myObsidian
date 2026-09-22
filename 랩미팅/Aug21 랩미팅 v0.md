*"My goal: waf에 정상 traffic 흘려보내서, Policy Engine이 정상 구분하는 법 학습하도록 traffic 생성."*
	traffic: HTTP request + response


## 논문 Research
... waf에 의심 traffic 흘려보내서 학습하는 논문 search

- ModSec-Learn
	https://github.com/pralab/modsec-learn-dataset
- HTTP2vec
	https://arxiv.org/pdf/2108.01763
	https://www.kaggle.com/datasets/ispangler/csic-2010-web-application-attacks


악성 트래픽 만들기

| ModSec-Learn    | SQLmap + tamper 스크립트 (페이로드 난독화), Kaggle SQLi 데이터셋, HTTP Params Dataset |
| --------------- | ---------------------------------------------------------------------- |
| HTTP2vec (UMP)  | **Arachni 스캐너**로 자체 앱 공격                                               |
| ModSec-AdvLearn | 적대적 변형(adversarial mutation)으로 우회 페이로드 생성                              |
| Cloudflare 블로그  | **sqli-fuzzer**, pseudorandom noise 샘플링                                |

정상 트래픽 만들기

| ModSec-Learn   | open-appsec 데이터셋 — **남이 만든 걸 가져다 씀**                    |
| -------------- | ------------------------------------------------------- |
| HTTP2vec (UMP) | "**collected** legitimate user traffic" — 실서비스에서 **수집** |
| CSIC 2010      | "자동 생성했다" (방법 설명 없음)                                    |
| Cloudflare     | 자사 네트워크의 **실트래픽**                                       |

-> 직접적인 데이터 수집, 현 상황으로서는 고객사 실트래픽 수집 불가능. 
정상트래픽 자동 생성 -- cloudflare 데이터 증강, ...

-- 

정상 트래픽 생성을 위한 행동 모델링& 값 생성에 대해서 추후 깊게 알아볼 예정. 
- 마르코프 체인 전이 모델+ think time
- web workload characterization
- 데이터 증강
- ZAP spider 통한 endpoint app 자동추출

더 알아볼 논문
1. Internet Web servers: workload characterization and performance implications 
   (https://ieeexplore.ieee.org/document/649565)
2. Optimizing HTTP Traffic Classification in Web Application Firewalls: A Comparative Analysis of Random Forest and SVM
   (https://ieeexplore.ieee.org/document/11181583/authors#authors)
3. Understanding Web Application Workloads and Their Applications: Systematic Literature Review and Characterization 
   (https://arxiv.org/abs/2409.12299)
4. Workload_Characterization_Issues_and_Methodologies 
   (https://www.researchgate.net/publication/221024511_Workload_Characterization_Issues_and_Methodologies)
5. HTTP2vec: Embedding of HTTP Requests for Detection of Anomalous Traffic 
   (https://arxiv.org/abs/2108.01763)


---

traffic 생성: 
	http request -> waf->webApp -> response(or WAF 차단 응답) => **jsonl** (request+ waf 판정+ response).

실제 offline 학습: 
	추후 jsonl -> ai policy engince => rules.conf 룰셋 생성. 관리자 검토 후 WAF 수동적용

traffic 묶음 -> session 생성 (web app 에서 사용자가 움직이는 시나리오)

```
━━━ 실시간 (실험 돌리는 동안) ━━━━━━━━━━━━━━━━━━━━━━━━
  [내 생성기] ──HTTP 요청──▶ [WAF/ModSec] ──▶ [웹앱]
       ▲                       │
       └────── 응답 ───────────┘
                               │
                               ├─▶ WAF 감사 로그 (걸린 룰, 점수)
                               │
                       [수집기가 요청+응답+WAF판정을 합쳐서]
                               │
                               ▼
                       📄 traffic.jsonl  ← 그냥 파일로 저장됨. 끝.


━━━ 나중에, 별도로 (오프라인 학습) ━━━━━━━━━━━━━━━━━━━
  📄 traffic.jsonl ──▶ [AI Policy Engine] ──▶ 📄 rules.conf
                          (팀원 A 담당)          (SecLang 룰셋)
                                                     │
                                                     ▼
                                            👤 관리자가 검토
                                                     │
                                                     ▼
                                            [WAF]에 수동 적용
```

HTTP request 예시 (input-1)
```http
POST /rest/user/login HTTP/1.1
Host: juice-shop:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...
Content-Type: application/json
Content-Length: 52
Cookie: language=ko; welcomebanner_status=dismiss

{"email":"alice@juice.sh","password":"Pa$$w0rd!23"}
```

-- 사용자의 http 요청

HTTP response 예시 (input-2)
```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: token=eyJhbGciOiJSUzI1NiIs...

{"authentication":{"token":"eyJhbGci...","umail":"alice@juice.sh"}}
```

-- web Application으로부터 응답. 
-> status code (요청 성공/실패) + 응답 본문의 로그인 여부 (정규식, 세션유지 여부)
- waf 통과 시 전달만 함 / block 시 웹엡까지 가지않고 403 에러 돌려줌
- http status code 를 통해 성공/실패여부 확인
- http response 통해 로그인 상태인지 체크

WAF 결과 (input-2-1)

-- 사용자 요청이 실제로 waf에 의해 차단된건지 정보 확인 
(https://github.com/owasp-modsecurity/ModSecurity/issues/1484)

```json
Jul 3 14:05:24 10.195.24.226 
{
	"transaction": 
	{
		"time":"03/Jul/2017:14:05:24 +0900",
		"transaction_id":"WVnQlArDGOIAAGaFyOYAAACQ",
		"remote_address":"10.10.10.20",
		"remote_port":51495,
		"local_address":"10.10.10.30",
		"local_port":80
	},
	"request":
	{
		"request_line":"POST / HTTP/1.1",
		"headers":
		{
			"Host":"http://10.10.10.30/index.html",
			"User-Agent":"curl/7.51.0",
			"Accept":"*/*",
			"Content-Length":"51",
			"Content-Type":"application/x-www-form-urlencoded"
		},
		"body":["{1}IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII"]
	},
	"response":
	{
		"protocol":"HTTP/1.1",
		"status":200,
		"headers":
		{
			"Last-Modified":"Wed, 28 Jun 2017 04:28:36 GMT",
			"ETag":"\"873-552fd9e21cba3\"",
			"Accept-Ranges":"bytes",
			"Content-Length":"2163",
			"Content-Type":"text/html"
		},
		"body":"<html><body><h1>It works!</h1></body></html>\n<h2><span style=\"color:red\"; >ModSecurity Test Page <span></h2>\n\na`````````````````````````````````````````````````````````````````````````````"
	},
	"audit_data":{}
}
```

json 기록 (output)
```JSON
{
  "transaction": {
    "client_ip": "172.19.0.4",
    "time_stamp": "Thu Aug 20 10:23:41 2026",
    "server_id": "8a1f2c9e4b7d",
    "client_port": 51492,
    "host_ip": "172.19.0.3",
    "host_port": 8080,
    "unique_id": "175880062112.884371",
    "is_interrupted": true,

    "request": {
      "method": "GET",
      "http_version": 1.1,
      "hostname": "waf",
      "uri": "/rest/products/search?q=O%27Brien",
      "body": "",
      "headers": {
        "Host": "waf:8080",
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
        "Accept-Language": "ko-KR,ko;q=0.9",
        "Cookie": "token=eyJhbGciOiJSUzI1NiIs..."
      }
    },

    "response": {
      "http_code": 403,
      "headers": { "Content-Type": "text/html" }
    },

    "producer": {
      "modsecurity": "ModSecurity v3.0.12 (Linux)",
      "connector": "ModSecurity-nginx v1.0.3",
      "secrules_engine": "Enabled",
      "components": ["OWASP_CRS/4.7.0\""]
    },

    "messages": [
      {
        "message": "SQL Injection Attack Detected via libinjection",
        "details": {
          "match": "detected SQLi using libinjection with fingerprint 's&sos'",
          "reference": "v51,8",
          "ruleId": "942100",
          "file": "/etc/modsecurity.d/owasp-crs/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf",
          "lineNumber": "45",
          "data": "Matched Data: s&sos found within ARGS:q: O'Brien",
          "severity": "2",
          "ver": "OWASP_CRS/4.7.0",
          "rev": "",
          "tags": [
            "application-multi", "language-multi", "platform-multi",
            "attack-sqli", "paranoia-level/1",
            "OWASP_CRS", "capec/1000/152/248/66"
          ],
          "maturity": "0",
          "accuracy": "0"
        }
      },
      }
    ]
  }
}
```

```
사용자 web application 구조 파악 → 정상 트래픽 생성

   (1) 구조 파악          : ZAP Client Spider + Data Driven Node
                            → 엔드포인트 맵 자동 추출

   (2) 행동 모델링         : 마르코프 체인 상태 전이 모델 + think time
                            (부하 테스트 도구의 VU/weight/think-time 개념 차용)
                            → 크롤러의 균등 분포가 아닌 Zipf 편중 재현

   (3) 값 생성            : 엔드포인트별 파라미터 타입에 맞는 현실적 값
                            + Cloudflare식 증강 (위험 키워드를 포함한 정상값)
                            → 오탐 유발 케이스 발굴

   (4) 편향 제거          : Zen Context 8개 필드 전부 변화
                            + 정상/악성 헤더 분포 동일화 (HTTP2vec 편향 사례 대응)

   (5) 라벨 검증          : Zen SQL 토크나이저(zen-internals)로 생성 시점 자가검증

   (6) 현실성 검증         : Zipf 적합도, 세션 길이/think time 분포 측정
```

---
## ZAP
penetration(침투) testing tool. 
MITM proxy

~> ZAP 의 크롤러 통한 웹사이트 구조 알아내기 차용- 고객 사이트의 웹 구조 알아내기
~> web traffic scenario, login/session 유지 차용 (로그인 방식, logout 되는 url path, site 범위,...)
(+ 사용자의 실제 트래픽 모방하기 - ZAP에는 없음. )

ZAP이 공격 요청을 서버에 쏴보고, 반응을 확인한 뒤, 취약점을 알려줌 (alert 단계별)
학습기능 없음. 기존에 있는 Scan Rule 목록을 서버에 대입해보는 방식 (고정된 규칙)

- zap, 클라이언트 위치에서 malicious request 보내서 공격해봄
	client (<-zap인증서- "zap" -원본 인증서->) -> WAF(nginx+modsec) -> server 
	- 도메인별 인증서, 즉석에서 만들어냄. 
	- *"self-generated Root CA certificates"*

1. crawl web application with spider - passively scan each page if finds
2. use active scanner to attack all of the discovered pages, functionality, and parameters
[Automation Framework Enviornment](https://www.zaproxy.org/docs/desktop/addons/automation-framework/environment/)
##### attack traffic 생성과정
**spider**(크롤러) 통해 **site 지도** 알아내기
active scan 으로 공격 페이로드 주입

traditional spider (빠름. JS based page, 못 봄)
1) 시작 url에 요청보냄
2) 돌아온 HTML 읽음
3) HTML 안에서 갈 수 있는 곳을 뽑아냄 (ex. `<a href="/login">`)
4) 발견한 곳들, enqueue
5) queue에서 하나 꺼내서 1번으로 돌아감. 반복.

client spider
- DOM event 실시간으로 받기가능. traditional보단 느림
- *"explores the web application by invoking(호출-) browsers which then follow the links that have been generated"*
+AJAX Spider (Crawljax 외부 라이브러리 사용, 효율 low 추천 x)
+traditional+ client spider 함께사용

context 
- web Application의 url들을 연결시킴 (웹 내에서 사용자의 url 움직임, 현재 로그인 유무)
- sites tree 내의 모든 url에 적용되는 regex 들임.
- session management method 통해 세션 다룸
	- cookie-bases, HTTP Authentication(`Authorization`), script-based
	- 추출(요청에서 세션 정보 꺼냄), 생성(빈 세션 만들기), 제거(요청에서 세션흔적 지움- 비로그인), 주입(요청에 세션 심음)
(https://www.zaproxy.org/docs/desktop/start/features/sessionmanagement/)

## zen firewall
(https://github.com/AikidoSec/firewall-go)
web application 프로세스 내부(런타임)에서  라이브러리로 삽입되어 동작

- app 프로세스 안에 사용자 요청이 들어왔을 때, 위험한 함수호출 하는지 판별, 실제 요청으로 넘기거나 error 처리. 
	- 나의 코드를 검사 (application runtime 내부 context.)
- **FP와 FN 탐지** 가능
- `source`(입력 포착)-> `sinks` (위험함수 후킹)-> `vulnerabilities`(데이터 흐름 대조)
- regex 아니라 SQL 토크나이징으로 문법구조 변경 판정 (별도 라이브러리)
	- SQLi, PT, command inj, ... (`library/vilnerabilities`)
	- sink 에 도달하는 공격만 판정가능 (XSS 같은 공격은 x)

~> traffic 수집 시, 편향없이 모든 변수에 값 잘 들어오는지 check
`context.go`  틀 참고해서 traffic 수집
```go
...
type Context struct {
	URL                string
	Path               string
	Method             string
	Query              map[string][]string
	Headers            map[string][]string
	RouteParams        map[string]string
	RemoteAddress      *string
	Body               any
	Cookies            map[string][]string
	Source             string
	Route              string
	executedMiddleware bool

	user           *aikido_types.User
	rateLimitGroup string

	deferredAttack *DeferredAttack
	...
```

~> traffic 파일 생성 시, 정상 traffic 인지 검증
	Zen SQL tokenizer
~> 다양한 traffic 시나리오 수집하도록 점검
	Zen sink 목록

## Cloudfare ML dataset blog
ML **WAF 학습**시킬 데이터, **데이터 증강과 sampling** 을 통해 생성
[Improving the accuracy of our machine learning WAF using data augmentation and sampling](https://blog.cloudflare.com/data-generation-and-sampling-strategies/#:~:text=Noisy%20labels.%20Label%20noise%20affects%20results%20a,studying%20statistical%20distribution%20of%20existing%20real%2Dworld%20data)

bening request (정상 요청 생성)
> data augmentation- 데이터 증강
> 	it means that we **mutate(변이-) benign conten**t in a variety of ways as the content will **remain benign** (this **isn’t** going to **accidentally turn** it into a **valid** payload, with probability 1). 
> 	For instance, we can **add random character noise**, **permute keywords, merge benign content** together from multiple sources, and so on. Alternatively, we can seed benign content with **‘dangerous’ keywords** or [ngrams](https://en.wikipedia.org/wiki/N-gram) frequently occuring in payloads - this results in a benign sample, but ideally will teach the model not to be too sensitive to the presence of malicious tokens lacking the proper semantics and structure.

malicious request (비정상 요청 생성법)
> use sqli-fuzzer, 
> psudorandom noise samples
> 	. This approach works by taking a collection of tokens and a probability distribution over these tokens, and independently sampling a stream of tokens from it to create our ‘sample’. Each sample length is selected from a separate discrete sample length distribution.
> 	. we can move towards much more ‘difficult’ noise examples, including elements such as: fragments of valid URIs, user agents, XML/XSLT content or even restricted language identifiers, or keywords.

