[[<WAF (Web application Firewall)>]]
26-1 w14, 6월 1주차 스터디

**기초 용어**
- Payload& Signature: 해커가 보낸 데이터(Payload) 에서 공격자임을 알 수 있는 Signature 를 찾기
- FP(오탐): 일반 고객까지 차단
- FN(미탐): 해커 놓치는 경우

**연구 배경**
- 기존 WAF, 일일히 규칙 써줘야 함
	- LLM, 분석 시간 느림

=> **VibeWAF** (Hybrid 방식)
: 빠른 규칙 엔진이 대부분 처리, 처음보는 규칙만 LLM 탐지, 바로 새 규칙 만들어 엔진에 전달

**VibeWAF 매커니즘**
**Stage 1. Matching**
: ModSecurity, 기존 규칙(SecRule)으로 트래픽 필터링. 
- (빠른 시간)

**Stage 2. Analysis& Generation**
: 규칙에 없는, 매칭되지 않은 트래픽만 LLM 분석. 새로운 규칙 생성, 이를 시스템에 등록(Feedback Loop)
	LLM, 공격으로 판단 시(ex. SQL Injection) 즉시 이에 맞는 ModSecurity SecRule 문법의 규칙 생성. engine에 update

-> Hit Rate 점점 향상. 
-> Latency, LLM 단일 모델에 비해 빠름. 
-> Recall(탐지율), OWASP 규칙셋보다 많이 잡아냄

-  장점: zero-day vulnerability에도 수 초만에 대응규칙 만들어냄 [[Zero-day vulnerability (제로데이 취약가능성)]]
- 한계: 
	- 규칙이 너무 많이 생성됨, 정리필요 (5000건당 750개 규칙생성됨. 규칙 생명주기 관리)
	- white-list 함정: 기존 white-list 규칙이 신규 공격을 정상으로 오인. 공격 통과시키는 문제.

=> LLM, 실시간 탐지에 쓰기보다, asynchronous ruleset 생성기로 사용하는게 현실적
=> 기존의 OWASP CRS 을 기본, AI가 보완하는 방식이 훨씬 안정적. 

