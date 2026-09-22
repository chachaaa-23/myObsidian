[[<WAF (Web application Firewall)>]]
26-1 w13, 5월 5주차 스터디

**>WAP**
- 암호화 x / 암호화된 HTTP 트래픽 검사, 규칙 목록에 정의된 공격 여부 확인
- DDos 공격, 봇 완화, API 보호, 다중 벡터 공격 방어필요
- 최종 사용자 ~~ WAF ~~ Web server 사이에 존재하는 검사 flow

문제점
- 부정확한 탐지, 높은 오탐률
- 수동 검토 및 유지, 정적인 ruleset 의존
	→ FP(False Positive) 발생, 일반 유저들까지 전부차단 / FN (False Negative) 발생, 보안망 느슨해 방어뚫림
	
- 웹 기반 공격 보호 (XSS, SQL Injection, DDos 공격)


**>>Adaptive Security Engine (적응형 보안 엔진)**

- ML통한 고유 트래픽 및 공격 패턴 학습, 요청 실시간 분석 통한 미래의 위험 차단.
- self-tuning 통한 규칙 최적화
- backdoor API 찾아내고 데이터 분석
- good bot, bad bot 골라내기

**>Padding Evasion (패딩 회피)** [[Padding Evasion 패딩 회피]]

- 해커, 악성 페이로드 앞에 대량의 junk data 붙임, WAF 검사 buffer(8-128KB) 넘기게 만듬
    - WAF, 성능 유지 위해 용량이 큰 요청, 검사없이 통과시키는 취약점 활용(Fail Open)

**>>Behavioral DDoS Engine (행동 기반 DDoS 엔진)**

- 고정 Threshold(임계값) 없이, 평상시 traffic 의 Baseline(기초선) 학습. 이상 직후 즉각 감지 및 차단

**>Edge Platform**

- 전세계에 있는 서버, 공격 시작되자마자 즉시 차단
- 멀웨어 차단 [[Malware 멀웨어]]
	: 수상한 파일 업로드 시, 서버에 닿기 전, network edge 에서 미리 검사, 바이러스 걸러냄 