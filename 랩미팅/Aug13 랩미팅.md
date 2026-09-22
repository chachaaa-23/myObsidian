### 1. Autoresearch, AI Rule 수정 / 제외 기준

**요약**: 반복 최적화 알고리즘
: 카테고리별 진단 정보를 근거로 방향을 정하는 방식.

```
목표 함수: maximize balanced_accuracy(config)
제약: config {REQUEST-900-BEFORE.conf, RESPONSE-999-AFTER.conf}
탐색 연산자: {threshold 조정, CRS 룰 제거, 커스텀 룰 추가, content-type 조정}
선택: keep (개선 시) / revert (악화·동일 시)
탐색 우선순위: missed_by_category, false_positives_by_category 기반 그리디 선택
종료 조건: 연속 실패 N회, 또는 외부 시간 제한
```


**흐름**:
1. docker 로 Modsec CRS + backend 가동
2. `watchdog.sh` 로 `run-agent.sh`  반복 호출
-> `run-agent.sh`가 claude CLI를 매 실험마다 새 프로세스로 호출

```
1. docker compose up -d          # ModSecurity CRS + backend 컨테이너 기동
2. bash scripts/watchdog.sh <branch>   # 에이전트 루프 감시 프로세스 시작
      └─ watchdog.sh가 scripts/run-agent.sh를 반복 호출
            └─ run-agent.sh가 claude CLI를 매 실험(experiment)마다 새 프로세스로 호출
```

##### `run-agent.sh`
- 매번 claude 를 새롭게 호출. 이전 기억 없음.
- 간단한 프롬프트와 기존 평가기록, 참고할 파일목록 넘겨줌
	- workflow prompt 파일 (program.md), 현재까지의 진행 결과 기록(git log)과 최고 기록(results.tsv), ...
- 한번에 한 변화(experiment) 만 하도록 제약.

	```
	claude --model claude-sonnet-5 --permission-mode bypassPermissions --print You are running experiment 2 in the WAF autoresearch loop.\012Branch: autoresearch/v4-mac-run1 | Commit: e61b44f | Total results: 29\012Best so far: 1e4a6c2\0110.989970\0110.998829\0110.981111\0110.018889\011keep\011remove CRS rule 921130 (HTTP Response Splitting Attack) — audit-log sole-cause analysis of E-Commerce FPs found RUM/perf-beacon libraries (Boomerang, Akamai mPulse, Cloudflare) report negotiated protocol as literal ARGS value (nt_protocol=http/1.1, ak.proto=http/1.1, json.timingsV2.nextHopProtocol=http/1.1), colliding with the \bhttp/\d branch of the regex; dataset has no header-injection/response-splitting attack category at all so rule is entirely non-load-bearing, zero TPR loss, FPs 91->85, new best\012\012Read program.md for full instructions. The config files are large now.\012\012EFFICIENT WORKFLOW (avoid timeouts):\0121. Read results.tsv to see what's been tried\0122. Run: python3 scripts/evaluate.py > run.log 2>&1 to see current missed/FP breakdown\0123. Based on run.log, pick ONE targeted change (don't re-read full config unless needed)\0124. Make the change, git add -A && git commit -m 'experiment: <description>'\0125. Run evaluate again, read run.log\0126. Append to results.tsv: commit<TAB>balanced_accuracy<TAB>tpr<TAB>tnr<TAB>fpr<TAB>keep|discard<TAB>description\0127. If worse than best: git revert --no-edit HEAD\0128. git add results.tsv && git commit -m 'results: <description>'\012\012Do exactly ONE experiment. Focus on either removing a high-FP CRS rule or adding a targeted detection rule for remaining misses.
	```


```sh
"You are running experiment $EXPERIMENT in the WAF autoresearch loop.

Branch: autoresearch/$BRANCH | Commit: $CURRENT_COMMIT | Total results: $((RESULTS_COUNT - 1))
Best so far: $BEST

Read program.md for full instructions. The config files are large now.

EFFICIENT WORKFLOW (avoid timeouts):

1. Read results.tsv to see what's been tried
2. Run: python3 scripts/evaluate.py > run.log 2>&1 to see current missed/FP breakdown
3. Based on run.log, pick ONE targeted change (don't re-read full config unless needed)
4. Make the change, git add -A && git commit -m 'experiment: <description>'
5. Run evaluate again, read run.log
6. Append to results.tsv: commit<TAB>balanced_accuracy<TAB>tpr<TAB>tnr<TAB>fpr<TAB>keep|discard<TAB>description
7. If worse than best: git revert --no-edit HEAD
8. git add results.tsv && git commit -m 'results: <description>'

Do exactly ONE experiment. Focus on either removing a high-FP CRS rule or adding a targeted detection rule for remaining misses." 
```


##### `program.md`
- claude 가 매 실험마다 참고하는 system prompt.
- 목표
	1) BA 최대화: TPR(공격 막기)과 TNP(오탐 없음) 동일 가중치
	2) Simplicity criterion: 동일한 성능 개선일 시 더 단순한 변경 우선 (rule 제거 > rule 추가.)
	3) FP와 TN, TP와 FN 함께 개선: "공격 탐지를 늘리려고 정상 트래픽을 더 막는 것"과 "오탐을 줄이려고 탐지력을 희생하는 것" 모두 금지

 - 탐색 방향 제시 (starting directions / strategy tips)
    - paranoia level 조정, anomaly score threshold 튜닝, 특정 CRS 룰 제거, 카테고리별 커스텀 룰 추가, content-type 허용 범위 조정 등
    - 전략: broad(전역 설정) → specific(개별 룰) 순으로 탐색, 무작정 룰을 지우기보다 threshold 조정을 먼저 시도
    - `false_positives_by_category`와 `missed_by_category`를 근거로 다음 행동을 결정하도록 명시

- 파일 구조
	1. 현재 상태 확인 후 (result.tsv) 
	2. 목표 및 제약 규정. 
		- BA 지수 올리기. 단순한 config 작성. simplicity criterion- 공격 막되 정상 트래픽을 더 막으면 안됨(vice versa)
		- config files 통한 BA 파악, 수정 가능한 파일과 불가능한 파일 파악, output format 규정
	3. loop 순서 규정
	4. 탐색 시 idea (starting directions) 과 strategy tips 제공
		- paranoia lever sweep, anomaly score thresholds, 특정 카테고리용 rule removal & add custom rules, content-type 튜닝 등 다양한 전략 제공
		- broad -> specific 하게 탐색. 단순히 rule 지우지 말고 threshold 와 같은 세팅 조절부터 시도 권장. `false_positives_by_category` 와 `missed_by_category` 파악을 통해 rule 수정 

**`request.jsonl` :** 
```json
{"method": "...", "path": "...", "headers": {...}, "body": null, "label": "malicious", "category": "sqli"}
```

- 실험 loop flow:
	1. 현 상태 파악 (results.tsv, .conf)
	2. 탐지 실패한 공격유형과 카테고리 (`missed_by_category`, `false_positives_by_category`) 파악, 다음 행동 결정
		`2. Decide what to try next. Use the 'missed_by_category' and 'false_positives_by_category' from the last run to guide your decisions.`
	3. 실제 설정 수정 (.conf)
	4. git commit 후 재평가
	5. 기존 최고 BA와 비교, 개선 시 브랜치 유지. 개선 없으면 브랜치 reset
	6. 결과 기록 (results.tsv)

=> 오탐 원인 규명 후 제거
FP 최다 category 확인 (`false_positives_by_category`) -> 원인 rule 특정 -> rule 제거 / 설정 완화 / custom rule 추가 -> 변화 후 TPR (`missed_by_category`) 체크, 이익일 시 keep

##### 판단 패턴 유형

패턴 A — CRS 내장 룰 제거 (threshold/policy 완화)

> `502eaba` — CRS rule 932270 (Unix Shell Expression) 제거. 
> 	E-Commerce 트래픽에서 쿼리스트링이 shell 메타문자 패턴과 우연히 유사해 오탐 발생. FP 1253→848, TPR 손실 없음 → keep.

쿼리스트링/url이 shell meta문자 패턴과 유사, e-commerce category FP 최다원인 파악, rule 제거. (`932270`)

패턴 B — 커스텀 룰 신규 추가 (missed_by_category 대응)

> `f2574a4` — custom rule 1000001 (난독화된 OS 파일 경로 순회 탐지) 추가. 
> 	`missed_by_category`에서 traversal 30건이 CRS 930100/930110/930120의 디코딩 로직을 우회하는 것을 확인, 리터럴 ASCII 앵커로 직접 매칭하는 규칙 신설. traversal 미탐 30건 전량 해결, 신규 오탐 0건.

`missed_by_category` 에 특정 공격유형이 몰려있는 것 파악시, 해당 카테고리 전용 정규식 rule 추가 (`100026`)

패턴 C — 실험 후 기각 (discard, 트레이드오프 판단)

> `2928914` — CRS rule 942100 (libinjection 기반 SQLi) 제거 시도. 
> 	E-Commerce FP의 최다 원인(42/144)이었으나, 제거 시 TPR이 0.916→0.881로 하락하며 순손실 발생 → discard.

threshold 조정했더니 TP 손실이 더 크고 FP 감소가 적음. discard. (`ea9bed7`)

```
1. false_positives_by_category 1위 카테고리 확인 → 어떤 CRS rule ID가 원인인지 로그로 특정
2. 그 rule 제거 or content-type/threshold 완화 시도
3. missed_by_category가 늘면(TPR 손실) → 되돌림, 안 늘면 → keep
4. missed_by_category가 특정 공격 유형에 몰리면 → 해당 유형만 잡는 좁은 정규식 custom rule 추가 (넓게 잡으면 FP 유발 → 나중에 tighten)
5. 동일 balanced_accuracy면 더 단순한 쪽(rule 삭제 > rule 추가) 채택 — program.md의 simplicity criterion을 실제로 지킴
```


### 2. Autoresearch AI Ruleset Guidance (내부 logic, tuning 과정)

1) **이중 가이드 구조**

| 구분    | program.md                   | run-agent.sh 프롬프트           |
| ----- | ---------------------------- | --------------------------- |
| 성격    | 고정된 규칙서 (매 실험 공통)            | 매 실험마다 동적으로 생성되는 컨텍스트       |
| 담는 내용 | 목표 정의, 제약, 전체 알고리즘, 탐색 전략 힌트 | 지금 실험 번호, 현재 커밋, 지금까지 최고 기록 |
| 역할    | "이 프로젝트의 규칙은?"               | "지금 이 순간 무엇을 수정해야 하는지?"     |

run-agent.sh 가 매 호출마다 program.md 를 프롬프트로 재주입.

2) **목표와 제약 명시**
- BA 단일 최적화 목표
- simplicity criterion
- 탐색 공간 제약: 수정 가능 파일 명시. 점수 임의조작 차단

3) 탐색 순서 우선순위 부여
`program.md` 통해 탐색 순서 및 strategy tips 제시

- broad -> specific 순서로 수정 진행 (전역 설정 -> 개별 rule)
- 다음 룰셋 수정의 근거로 놓친 카테고리들(공격 기법) 사용 (`missed_by_category`, `false_positives_by_category`)
- rule을 지우기 전에 threshold 조정 먼저 하도록 지침
	`Don't just disable rules — try adjusting their severity or threshold first.`


### 3. Experiment 결과 및 분석

결과
	*원문 보고서: 19 experiments, 4 hours, 17 kept 2 discarded (CRS v4.24)*
- 8월 9일-11일, 총 172번의 experiment.
- claude 세션 대기시간 제외, 실제 활동시간: 1.1h
- BA=1.0 수렴 전 실험: 17 experiement
- 수렴 후: 96 experiement (84% 가 BA에 전혀 영향없는 활동)

|                           | 수렴 전 (17개 실험) | 수렴 후 (96개 실험) |
| ------------------------- | ------------- | ------------- |
| CRS 룰 제거                  | 17            | 10            |
| 룰 narrowing               | 11            | 1             |
| 커스텀 룰 추가                  | 8             | 0             |
| **simplify(****무변화****)** | 0             | **50**        |
| revert                    | 0             | 12            |
| 문서화                       | 0             | 10            |
| dead-weight 재검증(무변화)      | 0             | 9             |

현재 score:
```
balanced_accuracy: 1.000000
tpr:              1.000000
tnr:              1.000000
fpr:              0.000000
true_positives:   854
false_negatives:  0
true_negatives:   4500
false_positives:  0
total_requests:   5354
evaluation_seconds: 36.1
```

분석
1. 과적합 발생: 
- train, test 데이터셋 분리 x. 같은 데이터셋으로 측정.
- -> 데이터셋 외워버림 (악성 페이로드가 파라미터 이름 p에만 들어있는 것 발견- 공격 탐지를 p가 아닌 다른 파라미터에서 끔)
```
46f89c2: generalize ARGS:p-only narrowing ... to whole attack-category tags 
(attack-xss, attack-rce, attack-lfi, attack-injection-php, attack-sqli)
...confirmed all 15 FP-causing rules across 5 tags fire 100% on ARGS:p for 
real malicious traffic, zero hits on any other ARGS name
```

2. AI rule 수정 흐름:
step 1) 저비용 오탐 제거
	CRS 내장 rule을 통째로 제거 (`SecRuleRemoveById`)

step 2) scope 좁히기
	rule 일부 수정하며 특정 필드 (쿠키, 파라미터명) 만 제외. 오탐만 제거

step 3) custom rule로 miss된 공격 탐지
	`missed_by_category` 확인, 이를 탐지하기 위해 custom rule 생성

step 4) rule 일반화 (과적합)
	개별 rule 단위 탐지 -> 일반화 (과적합)


#### 과적합 수정 test

dataset: 기존 `requests.jsonl` 을 label, category 기준으로 stratified split. 
	train- 3728 (legit 3150, malicious 598, bypass 74), test- 1606 (legit 1350, malicious 256, bypass 32) 로 train과 test 진행. (약 7:3)

실험 횟수: 14회, 1h

핵심 관찰:
- TPR은 baseline부터 현재까지 0.911371로 내려가지 않음
	- 공격 탐지력 손실 없이 순수 FP만 걷어내는 중
- discard/revert는 아직 0건
	- 모든 실험이 FP 감소 + TPR 유지를 정확히 맞춰서 전부 keep됨
- 오탐 원인 패턴이 거의 다 "서드파티 쿠키/트래킹 값이 우연히 공격 페이로드 모양(쉘 연산자, JSON, SQL 주석 등)과 겹침"  

![[Pasted image 20260813150120.png]]