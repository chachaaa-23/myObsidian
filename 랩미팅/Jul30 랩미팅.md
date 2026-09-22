# 1. Autoresearch Ruleset generate 결과

### 1) Summary
rule 추가없는 기본 OWASP CRS v4.28 에서 출발해, claude code 를 autonomous agent loop(자동 루프) 로 구동해 WAF Ruleset 을 자동 생성함.

약 1시간동안 9회의 experiment 수행한 결과, BA(Balanced Accuracy) 가 0.8186 -> 0.9752로 상승함.

- 원문의 v4.24.0 실험(0.808 -> 0.984)과 현재 v4.28.0 (0.818-> 0.9752) 은 baseline 과 최종 성능 모두 근접함. 
- 동일한 데이터셋과 평가 방식을 반복 -> 과적합 가능 유의

### 2) dataset

5354개의 실제 HTTP 요청으로 구성됨
- legitimate traffic 4500건 
	- 14개 카테고리, 실제 header, cookie, query,  parameter, POST body 포함)
	- openappsec project 2024 capture [https://github.com/openappsec/waf-comparison-project/tree/main]
> `{"method": "HEAD", "path": "/edgedl/release2/chrome_component/bu3mck7kwo76bex7b5uf2cfm7a_29.4/imefjhfbkmcmebodilednhmaccmincoa_29.4_win_ebycdg273y6cqvh4ecxhmxclbe.crx3", "headers": {"Connection": "Keep-Alive", "Accept": "*/*", "Accept-Encoding": "identity", "User-Agent": "Microsoft BITS/7.8"}, "body": null, "label": "legit", "category": "E-Commerce"}`

- malicious traffic 854건
	- 7개 공격 카테고리(sqli, xss, pt, ...) + WAF bypass payload
>`{"method": "GET", "path": "/?p=../../../../../../../usr/local/etc/apache2/conf/httpd.conf", "headers": {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0", "Connection": "close"}, "body": null, "label": "malicious", "category": "traversal"}`

### 3) Experiment

환경
- CRS version: v4.28.0 (Docker의 owasp/modsecurity-crs:v4.28-nginx 사용) 
- claude-sonnet-5, 비대화형 모드 (headless mode, bypassPermissions --print), experiment 1회당 1회 호출. 
- bash 로 loop 제어
- agent 에게 program.md (프롬프트) + results.tsv (결과) + config (rule& security setting)

과정
`requests.jsonl` 
-> python 코드를 통해 HTTP 로 실제 전송
-> nginx & modsecuricy container (localhost:8880) 로 실제 WAF가 판정
-> backend: nginx, 모든 요청에 200 반환 (exploit 공격 성립 측정 불가)

Agent, 두 개의 파일에 수정한 룰셋, 환경설정 저장
- `config/REQUEST-900-BEFORE.conf` (CRS 로드 전 변수설정)
	예시
```apache
# Raise inbound anomaly score threshold from stock 5 to 10 — stock PL1
# defaults produce a very high false-positive rate (TNR 72%) on real-world
# traffic; a higher threshold requires more accumulated signal before
# blocking, trading a small amount of detection for a large FP reduction.

SecAction \
"id:900510,\
phase:1,\
nolog,\
pass,\
t:none,\
setvar:tx.inbound_anomaly_score_threshold=10"
```

![[Screenshot 2026-07-28 at 10.45.26 AM 1.png]]

- `config/REQUEST-900-AFTER.conf` (crs 룰 조작 및 custom rule 추가)
예시 1 : rule 지우기
```conf
# -- [[ Rule removal: 942550 (JSON-Based SQL Injection) ]] --
#
# 942550 looks for a JSON-ish structure (`{...}` / `[...]`) inside any argument
# or cookie and scores 5 as CRITICAL. Every modern analytics SDK stores JSON in
# a cookie, so on this dataset it fires almost exclusively on legitimate
# traffic: mixpanel `mp_*_mixpanel` device blobs (`{"email":179535617}`),
# Zendesk `__za_cd_*` visit arrays (`[1660653883]`), Freewheel `fw_utm`
# (`{}`), Criteo `hadronId`, plus a few GraphQL `variables` arguments.
# 26 legit requests match it against 1 malicious one (an Oracle
# DBMS_PIPE.RECEIVE_MESSAGE probe that also trips 942100). The JSON body of a
# real SQLi payload is already covered by the libinjection rule 942100 and by
# custom rule 100030, so removing 942550 gives up essentially no detection.
SecRuleRemoveById 942550
```

예시 2 : custom rule 추가
```conf
# -- [[ Custom rule: shellshock function prologue / XXE entity declaration ]] --
#
# Two signatures the PL1 rules only score partially, so the payloads land under
# the anomaly threshold:
#
#   `() {`  - the Bash exported-function prologue every CVE-2014-6271 variant
#             in the dataset carries. Requiring a word/`:`/`_` character after
#             the brace keeps it off Adobe Analytics beacons, whose `oid` param
#             carries JS stubs like `functionuc(){}` and `removechild() { [native code] }`.
#   `<!ENTITY` - an XML entity declaration. Legitimate for a document to
#             contain, but not as a request parameter value.
SecRule REQUEST_URI|ARGS|ARGS_NAMES|REQUEST_BODY \
    "@rx (?:\(\s*\)\s*\{\s*[:_0-9a-z]|<!entity)" \
    "id:100020,\
     phase:2,\
     deny,\
     status:403,\
     t:none,t:urlDecodeUni,t:lowercase,\
     log,\
     msg:'Shellshock function prologue or XXE entity declaration',\
     severity:'CRITICAL'"
```

![[Screenshot 2026-07-28 at 8.24.00 AM.png]]

유의점: agent 생성 정규식, 검증 x (성능 issue 가능)

BA 결과

| #   | Commit    | BA         | TPR    | TNR    | FPR    | 상태          | 변경 내용                                           |
| --- | --------- | ---------- | ------ | ------ | ------ | ----------- | ----------------------------------------------- |
| 0   | `983e129` | 0.8186     | 0.9157 | 0.7216 | 27.84% | keep        | Baseline (stock CRS v4.28.0, PL1)               |
| 1   | `b683d54` | 0.8538     | 0.7646 | 0.9429 | 5.71%  | keep        | Inbound anomaly threshold 5 → 10                |
| 2   | `2db9cd4` | 0.9065     | 0.8700 | 0.9429 | 5.71%  | keep        | Custom rule 100010 — sensitive file target      |
| 3   | `f9d9309` | 0.9227     | 0.8700 | 0.9753 | 2.47%  | keep        | CRS rule 932270 제거                              |
| 4   | `17a2fdd` | 0.9297     | 0.8841 | 0.9753 | 2.47%  | keep        | Custom rule 100020 — Shellshock + XXE           |
| 5   | `6d33b79` | 0.9601     | 0.9450 | 0.9753 | 2.47%  | keep        | Custom rule 100030 — SQL injection semantics    |
| —   | `adfecc4` | 0.9482     | 0.9660 | 0.9304 | 6.96%  | **discard** | Custom rule 100040 초안 — 과도한 매칭                  |
| 6   | `03be67d` | 0.9701     | 0.9649 | 0.9753 | 2.47%  | keep        | Custom rule 100040 — OS command injection (개선판) |
| 7   | `c9048b6` | 0.9724     | 0.9649 | 0.9800 | 2.00%  | keep        | CRS rule 920420 제거                              |
| 8   | `069bc50` | **0.9752** | 0.9649 | 0.9856 | 1.44%  | keep        | CRS rule 942550 제거                              |
총 9회 실험 중 8회 keep, 1회 discard. 소요 시간 **50분 22초** (12:05:08 ~ 12:55:30).


# 2. 연구 동향 (논문 분석)

### 1) 해외 특허, Google patent : 
1. **AI**-based Web application attack **detection** and **defense** method and system
(CN 2026 Feb)
-  AI-based method and system for detecting and defending against web application attacks

> historical web **attack** traffic **data에서** **유효 집합** 추출, **clustering**
> -> **construct** micro-feature cluster **library**, 탐지 모델에 입력해 **탐지**. 
> 
	"when a malicious attack is detected, a blocking operation is performed"
	"when the malicious attack is a new and unknown attack, its features are updated in the attack micro-feature cluster library"

![[Screenshot 2026-07-30 at 4.04.40 PM.png]]

[https://patents.google.com/patent/CN121690871A/en?q=(WAF+ruleset+generater+ai)&oq=WAF+ruleset+generater+ai&sort=new]


2. Threat mitigation system and method
(US 2024 Feb)
- 과거의 suspect activity/event을 기반으로 활동 모니터링, 의심스러운 활동 포함여부 확인 -> 보안 이벤트 알림 설정
- generative AI/formatting script 사용해 **보안 이벤트 알림**, **반복적 처리** -> human-readable report 생성

>"Generating one or more detection rules that are indicative of a security event,"
>"notification includes a computer-readable language portion that defines one or more specifics of the security event"
>"iteratively processing the initial notification using a generative AI model and a formatting script to produce a summarized human-readable report"

![[Screenshot 2026-07-30 at 4.06.38 PM.png]]
[https://patents.google.com/patent/US12395522B2/en?q=(WAF+ruleset+generater+ai)&oq=WAF+ruleset+generater+ai]


3. Systems and methods for predictive analysis of potential attack patterns based on contextual security information
(US 2021 Dec)
- predicting, potential attack
- network~application stack

[https://patents.google.com/patent/US12166785B2/en?q=(WAF+ruleset+generater+ai)&oq=WAF+ruleset+generater+ai&page=1]

### 2) 논문

**GenSQLi**: A Generative Artificial Intelligence Framework for Automatically Securing Web Application Firewalls Against Structured Query Language Injection Attacks
(US UNC, 2024 Dec)

- generative AI Framework (GPT-4o)
- in-context learning 통한 공격 생성과 방어규칙 생성 pipeline 통합 
- 생성형 LLM으로 얻은 SQLi attacks, Target DB에 validate 걸차 거침
	-> hallucination 문제 완화
- SQLi 공격 한정

workflow
```
[LLM: SQLi 생성] → [DB 검증(환각 제거)] → [WAF에 투입해 우회 여부 테스트]
		    → [우회 성공 공격만 수집] → [ML 클러스터링] → [LLM: WAF 규칙 생성]
		    → [새 규칙 적용 후 재검증] → (RLHF로 반복 개선)
```

![[Pasted image 20260730154321.png]]

- 1) WAF를 뚫는 새로운 SQLi 공격 payload를 생성형 AI로 자동 생성
	- 선별된 예제 활용한 문맥 기반 학습, hallucination 완화
![[Pasted image 20260730151327.png]]

- 2) ML Clustering과 LLM 으로 방어 규칙 자동 생성 및 테스트
	- 생성한 payload, WAF 에 test. 우회 성공 시, ML algorithm 사용해 분류 (clustering 시 사용한 ML algorithm: TF-IDF + HAC, SequenceMatcher + DBSCAN)

LLM, Clustering Guidance
![[Screenshot 2026-07-30 at 3.14.34 PM.png]]

- 총 514개의 new SQLi payload 중 89%, OWASP CRS 적용 ModSec WAF 우회성공

Sample SQLi (generated by GPT-4o)
> `or (SELECT CASE WHEN (1 = 1 AND ‘a’ LIKE CONCAT(‘a’, ‘%’)) THEN 1 ELSE 0 END) = 1–`

Sample ModSecurity SecRules (generated by GPT-4o)
> `SecRule REQUEST_URI|ARGS ‘‘@rx (?i)(char\(|ord\(|ascii\(|concat\()|/\*.*?\*/|(\b1\s*=\s*1\b|(\w\s*=\s*\w))’’ \‘‘id:1001,phase:2,deny,status:403,msg:‘SQLi Pattern: Character manipulation, obfuscated comments, or~tautologies detected’’’`

*RLHF (Reinforcement Learning with Human Feedback)*
[https://www.mdpi.com/1999-5903/17/1/8]

