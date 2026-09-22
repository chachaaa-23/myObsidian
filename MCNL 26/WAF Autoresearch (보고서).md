[[🥽autoresearch (WAFPlanet)]][[<WAF (Web application Firewall)>]]
AI agent를 활용한 ModSecurity CRS의 자율적 최적화 연구 분석

## 1. 개요

AI agent가 ModSecurity CRS 설정을 자동으로 최적화하여 balanced accuracy를 최대화한다.

기존의 WAF는 수동으로 규칙을 튜닝해 시간이 많이 걸리고 zero-day 공격에 대응하기 어렵다는 한계가 존재한다. 

이러한 문제를 해결하기 위해 AI가 스스로 반복적인 실험을 수행하며 최적의 설정을 찾도록 하였다.

#### 프로젝트 구조

| Path                             | Job                                                             |
| -------------------------------- | --------------------------------------------------------------- |
| `scripts/evaluate.py`            | 현재 보안 수준 평가                                                     |
| `config/REQUEST-900-BEFORE.conf` | 보안수준 지정 (paranoia level, 변칙 thresholds, ...)                    |
| `config/REQUEST-900-AFTER.conf`  | 규칙 지우기(`SecRuleRemoveById`), 커스텀 규칙 추가(`SecRule`), 규칙 override |
| `./results.tsv`                  | 모든 실험 결과 저장                                                     |
| `./program.md`                  | AI agent 행동 지침                                                 |
| `dataset/requests.jsonl`         | 실제 HTTP request (4500 정상 요청 + 854 공격 요청)                        |
| `docker-compose.yml`             | ModSecurity CRS + nginx backend                                 |

## 2. 평가
연구 평가 지표로 Balanced_accuracy 사용한다.

>*Balanced Accuracy = (TPR+TNR) / 2*

TPR(True Positive Rate, 공격 탐지율) 과 TNR(True Negative Rate, 정상 요청 허용률) 의 균형이 중요하다. 
이때 simplicity criterion을 따라 최대한 단순한 규칙을 유지하도록 해야 한다.

---
## 3. 흐름

Autoresearch의 핵심은 AI의 반복적인 규칙 최적화 루프이다. 

본 프로젝트는 Claude Code와 같은 AI agent가 Git 저장소와 터미널 환경을 조작하면서 자율적으로 운영하도록 한다. 

Git을 활용, 안전하게 실험 반복 가능하다. 
1. tag 를 통해 실험공간 분리 (계속해서 keep / discard 반복 필요함)
2. 실험 결과 commit 통해 기록
3. 평가 결과에 따라 keep / rollback 실행. 

```
AI Agent
▼
[program.md, results.tsv] 지시사항& 현재 CRS설정 확인. 실험 결과 파악
▼
[config/] config 파일 수정을 통한 규칙 수정
	ex. paranoia level 변경, anomaly threshold 조정, 특정 규칙 제거, custom 규칙 추가, ...
▼
[evaluate.py] 평가 실행 (정상+악성 요청 전송)
▼
[results.tsv] 평가 결과 저장
	Balanced Accuracy, TPR, TNR, FPR, ...
▼
balanced_accuraty 향상 -> keep 
balanced_accuraty 감소 -> revert
```


### 실제 규칙 수정 과정

##### 예시 1. `REQUEST-900-BEFORE.conf` 의 설정값 변경

```shell
# === PARANOIA LEVEL ===
# Override paranoia level (1-4). Higher = more rules = better detection + more FPs.

SecAction \
	"id:900500,\
	phase:1,\
	nolog,\
	pass,\
	t:none,\
	setvar:tx.paranoia_level=1,\
	setvar:tx.executing_paranoia_level=1"
```

현재 `tx.paranoia_level=1` --> `tx.paranoia_level=2` 로 변경,
`evaluate.py` 로 결과 평가, 결과에 따라 keep / `git reset`.  

##### 예시 2. Bypass 분석을 통한 실제 규칙 수정

`missed_by_category` 의 bypass 수를 줄이기 위해 `requests.jsonl` 파일 분석 
![Screenshot 2026-06-03 at 1.32.43 PM.png](app://49ddf19ea6b00630c576e4343158da36f7f0/Users/jessicacha/Documents/Obsidian%20Vault/Screenshot%202026-06-03%20at%201.32.43%20PM.png?1780461168992)
-> 놓친 공격들의 공통점 찾기. 새로운 규칙 생성, `RESPONSE-999-AFTER.conf` 에 추가

![[Screenshot 2026-06-03 at 1.56.39 PM.png]]

##### 예시 3. FP 감소를 위한 실제 규칙 삭제

카테고리별 FP 확인, 원인 분석

![[Screenshot 2026-06-03 at 2.03.19 PM.png]]

-> AI agent, ModSecurity log 확인, 어떤 CRS Rule이 요청을 차단하는지 확인
-> FP 원인이라고 판단 시, 규칙 삭제 (ex. `SecRuleRemoveById 932270`)

![[Screenshot 2026-06-03 at 2.06.29 PM.png]]

---
## 4. 실험 결과
#### CRS v4.24.0 (19 experiments, ~4 hours)

| Metric              | Stock defaults | Optimized | Change |
| ------------------- | -------------- | --------- | ------ |
| Balanced Accuracy   | 80.8%          | 98.4%     | **+21.8%** |
| True Positive Rate  | 91.3%          | 98.5%     | **+7.9%**  |
| False Positive Rate | 29.7%          | 1.6%      | **-94.5%** |
| False Positives     | 1,336/4,500    | 74/4,500  | **-94.5%** |
##### 주요 개선점
>False Positive(오탐)이 1336 -> 74 건으로 감소함.

--> AI, 단순히 탐지 규칙을 추가하는 것 뿐만 아니라, **과도한 차단**을 유발하는 **규칙**을 **제거**하는 방향으로 최적화를 진행하였음. 

### 실제 분석 결과

`results.tsv` 와 `run.log` 분석하여, 최적화된 설정의 결과를 확인함.

`results.tsv`의 최적화 결과 

![[Screenshot 2026-06-03 at 1.13.45 PM.png]]

1. 규칙 추가보다 FP 일으키는 규칙 제거가 더 중요할 수 있다.
	가장 큰 성능 향상: 새로운 규칙 추가(X) -> 특정 CRS 규칙 제거 (O)
		 TNR 0.934 -> 0.969 -> 0.979
		 TFR 0.065 -> 0.030 -> 0.023 으로 성능 향상

이는 WAF에서 과도한 탐지 규칙이 오히려 BA 를 낮출 수 있음을 알 수 있다. 

`run.log`의 최적화 결과
![[Screenshot 2026-06-03 at 2.17.38 PM.png]]

#### 한계
시스템의 대부분은 공격을 탐지하지만 (TNR), 일부 우회공격(`bypass: 13`) 은 여전히 탐지하지 못하고 있다.

또한 남아있는 오탐(FP)의 상당수가 `E-Commerce` 관련 정상 트래픽에서 발생하고 있다. (`E-Commerce: 43`)

---
## 5. 결론

AI agent를 통한 규칙 수정, 평가, 유지 / 폐기의 반복을 통해 Balanced Accuracy를 80.8% 에서 98.4%까지 향상시켰으며, 특히 False Positive Rate 를 94.5% 감소시켰다. (CRS v4.24.0 결과)

분석 결과 AI 기반 WAF 최적화는 단순한 규칙 생성보다 기존 규칙 제거와 설정 조정의 균형을 통해 더욱 큰 효과를 얻을 수 있음을 확인할 수 있었다. 

향후 연구에서 남아있는 bypass 공격 탐지 문제와 FP(오탐) 문제를 해결해야 함을 알 수 있었다. 

