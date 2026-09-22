(논문 사이트- 24년 6월)[https://arxiv.org/abs/2406.13547]
(오픈소스 코드)[https://github.com/pralab/modsec-learn]
(오픈소스 데이터셋)[https://github.com/pralab/modsec-learn-dataset]

**기존 autoResearch:** AI Agent가 rule을 수정하는 방식
**ModSec Learn:** ML이 rule의 rule weight(중요도)를 학습하는 방식
(SQLi attack 만을 대상으로 함)

### 결과
feature selection with ℓ1(sparse) regularization을 통해 CRS rule의 30% 이상을 제거하면서 성능 유지.

기존 ModSecurity 탐지율(TPR)을 45% 이상 향상. (1% FPR오탐율 기준)

높은 PL(paranoia level)에서도 TRP 향상

![[Screenshot 2026-06-08 at 6.25.33 AM.png]]


### flow
```정적 OWASP CRS 
Request
-> ModSecurity CRS
-> 어떤 Rule이 trigger 되었는지 추출
-> ML Model 입력
-> 최종 분류
```

### scoring
기존 CRS
```
942100 발생
점수 +5

941100 발생
점수 +3
```

ModSec-Learn
1)각각의 **CRS rule** (ex. `Rule 942100`)을 **input feature** vector 로 만든다.
```
Request A

942100 = 1
941100 = 1
932270 = 0
920420 = 0
...
```
2)ML model, 어떤 rule조합이 실제 공격인지 학습.
	`Detection rules, Paranoia, Level, Anomaly Scoring`
3)발생한 공격에 따라 rule의 weight 조정

#### dataset
`legitimate/`, `malicious/` 분리,
`X_train, X_test` 직접 구성 가능함
-> 다양한 tampering script 적용, 탐지 회피형 payload 생성

### ML
-> 2개의 linear model (SVM, Support Vector Machine 과 LR, Logistic Regression)