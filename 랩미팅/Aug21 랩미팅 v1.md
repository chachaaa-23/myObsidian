보안할 점.
*AI에 내 생각, 온전히 의탁하지 마세요.
논문을 처음부터 끝까지 텍스트를 읽어보며, 상세하게 분석하세요*

- OWASP ZAP, 아주 상세히 분석
	전반적인 흐름
	공격 페이로드 생성 과정 
	공격 생성을 switch 해, 정상 payload 생성 가능할지 분석
	실제 서비스 돌려보기, 코드 뜯어보기
- 기존 연구 더 찾아보기
	SW testing, Web service (SW 공학 분야)
	web hacking, security
	http payload 생성
- 악성 payload 생성 -> 정상 payload generate 가능한지 알아보기
- 선행연구 상세하게 분석하기
	논문, 프로젝트, ...
	ModLearn 논문, dissect black box 논문, WAF booster
	OWASP ZAP 프로젝트

---

Pf comment.

**--owasp zap, 선행 연구 확실히 분석. how to work?** 
공격 페이로드 생성을 바탕으로 -> 정상 페이로드 만들 수 있는지, 실제 서비스 돌려보기

**기존 연구들을 바탕으로 비슷하게 , 조금 더 알아보라.** 
**sw testing, web serverice 에서도 비슷한 것들이 분명히 있을 것이다.** 
	**web hacking, 보안** 

**악성 payload 생성-- 정상 payloda generation 가능 여부?**
**ModLearn 외의 다양한 연구참고.** 
	**--akido 연구는 지양**


Co- comment.

dissect black box
	*http "dataset"* 사용하는 실 예시와 논문 분석 필요

WAF Booster
	waf 행동 보고, 판정행동 근사, 개선. tokenize and generate 
	signature producer
	payoad generator -- opersource? 어떤 방식으로 했는지? 

waf를 위한 crs 생성 -- 정상인 상황 collect

---



>page 1.

*"My goal: waf에 정상 traffic 흘려보내서, Policy Engine이 정상 구분하는 법 학습하도록 traffic 생성."*
	traffic: HTTP request + response(+ WAF 판정기록)

>page 2.

... waf에 의심 traffic 흘려보내서 학습하는 논문/연구 search
몇가지 논문과 cloudflare blog 참고해서 트래픽 생성 연구 탐색함. 

정상 트래픽 생성 과정- 각자 운영중인 실서비스를 활용함. 
**cluoudflare** 
**http2vec- '실서비스?' 여부 (방향 확인)**

>page3

우리는 아직 연구단계이고, 클라이언트 마다 다른 웹 구조를 가지므로 하나ㅡ이 데이터셋만으론 커버불가.

-> 직접적인 데이터 수집, 현 상황으로서는 고객사 실트래픽 수집 불가능. 
정상트래픽 자동 생성 -- cloudflare 데이터 증강, ...

따라서 traffic 생성 필요함. 
http request 와 응답을 묶어 하나의 traffic 파일을 생성할 예정. 

>page 4.

endpoint 별 parameter 타입에 맞는 현실적인 값 생성:

**HTTP request 예시 (input-1) -- 사용자의 http 요청**
**HTTP response 예시 (input-2) -- web Application으로부터 응답.** 
	-> status code (요청 성공/실패) + 응답 본문의 로그인 여부 (정규식, 세션유지 여부)
	- waf 통과 시 전달만 함 / block 시 웹엡까지 가지않고 403 에러 돌려줌
	- http status code 를 통해 성공/실패여부 확인
	- http response 통해 로그인 상태인지 체크
	**WAF 결과 (input-2-1) -- 사용자 요청이 실제로 waf에 의해 차단된건지 정보 확인** 

response 옆에 **"이 응답을 만드는 것은 웹 애플리케이션이다. WAF는 통과 시 전달만 하고, 차단 시에만 자신이 403을 생성한다"** 

>page 5.


traffic 생성: 
	http request -> waf->webApp -> response(or WAF 차단 응답) => **jsonl** (request+ waf 판정+ response).

실제 offline 학습: 
	추후 jsonl -> ai policy engince => rules.conf 룰셋 생성. 관리자 검토 후 WAF 수동적용

traffic 묶음 -> session 생성 (web app 에서 사용자가 움직이는 시나리오)

>page 6.

트래픽이 어디서 만들어져서 어디로 가는지,
생성한 http request 가 정상/악성일때의 각기다른 반응
세션 유지되면 계속해서 트래픽 생성

"생성한 트래픽이 진짜 정상인지 어떻게 보장하나?"
	생성 시점에 라벨 확정 + zen-internals 토크나이저로 자가검증 + 세션 유효성 확인 후 오염 레코드 폐기

"현실적이라는 걸 어떻게 증명하나?"
	생성 트래픽의 URL 접근 분포·세션 길이 분포를 실트래픽의 알려진 성질과 비교 (지표는 조사 예정)

>page 7.

- 오탐 유발가능한 정상 케이스, 데이터 증강을 통해 생성. 
	- **"오탐이 나는 곳은 바로 이런 값들이다. 이런 케이스를 의도적으로 만들어야 오탐을 발굴하고 줄일 수 있다."**


>page 8.

악성 트래픽 생성

이미 명시되어 있으므로 해당 방안을 참고할 예정

>page 9.

ZAP의 기능과 흐름 간단설명

- ZAP이 공격 요청을 서버에 쏴보고, 반응을 확인한 뒤, 취약점을 알려줌 (alert 단계별)
- zap, 클라이언트 위치에서 malicious request 보내서 공격해봄

공격 트래픽 생성과정
	**spider**(크롤러) 통해 **site 지도** 알아내기
	active scan 으로 공격 페이로드 주입
	1. crawl web application with spider - passively scan each page if finds
	2. use active scanner to attack all of the discovered pages, functionality, and parameters

어떤 기능을 가져다 슬것인지

~> ZAP 의 크롤러 통한 웹사이트 구조 알아내기 차용- 고객 사이트의 웹 구조 알아내기
~> web traffic scenario, login/session 유지 차용 (로그인 방식, logout 되는 url path, site 범위,...)
(+ 사용자의 실제 트래픽 모방하기 - ZAP에는 없음. )

>page 10.

ZEN의 기능과 흐름 간단설명

- 애플리케이션 런타임 내부에 삽입되는 in-app firewall. 
- `source`(입력 포착, context 파일에 넣음)-> `sinks` (위험함수 후킹)-> `vulnerabilities`(데이터 흐름 대조)

- 정규식이 아니라 SQL 토크나이징으로 판정.
- sink에 도달하는 공격만 판정 가능 (XSS는 불가) → 보조 심판이지 만능 심판이 아님.
- 라이선스가 AGPL이라 **제품 내장은 불가, 연구·검증 도구로만 사용**

어떤 기능을 가져다 슬것인지

| 활용                                             | 근거                                                                        |
| ---------------------------------------------- | ------------------------------------------------------------------------- |
| **① 입력 표면 체크리스트** — http 요청의 어느 자리에 값을 채워야 하는가 | `firewall-go` `Context` 구조체 (URL/Query/Headers/RouteParams/Body/Cookies…) |
| **② 라벨 자가검증** — 생성한 "정상" 값이 진짜 안전한지 확인         | `zen-internals` SQL 토크나이저 (반환코드 0=safe / 1=injection)                     |
| **③ 오탐·우회 판정 오라클** — WAF 판정과 대조                | Zen 탐지 전용 모드 (기본값, 계정 불필요)                                                |

나쁜 traffic 인지 자가검증 필요? 
.rjawmd 조ㅛ?


>page 11.

생성 파이프라인과 공부할 부분

- (2) 크롤러의 균등 분포로는 실사용자를 재현할 수 없어서
- (3) 오탐을 유발하는 값을 의도적으로 만들어야 해서
- (6) "현실적이다"를 주장이 아니라 측정으로 보여야 해서

마르코프 체인·think time·Zipf 같은 단어는 **한 번씩만 언급**. 

>page 12.

앞으로 공부해올 내용

1. 상태 전이 모델 = "지금 어디 있느냐에 따라 다음에 갈 곳의 확률이 정해진다"

	`마르코프 체인`의 쉬운 이름이야. 개념은 세 개뿐이야.
	장바구니에 아무것도 안 담고 결제부터 하고, 로그아웃한 뒤에 장바구니를 봐. **사람이 이렇게 안 다녀.**
	**상태 전이 모델은 빈도와 순서를 동시에 재현해.** "결제는 반드시 장바구니 다음에 온다"가 확률표에 들어있으니까.

- 마르코프 체인 전이 모델+ think time
- web workload characterization
- 데이터 증강
- ZAP spider 통한 endpoint app 자동추출

workload, traffic classification 관련 논문


