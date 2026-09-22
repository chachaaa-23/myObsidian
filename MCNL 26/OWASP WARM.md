[[<WAF (Web application Firewall)>]][[OWASP]][[🥽autoresearch (WAFPlanet)]]

(WAF Advanced Ruleset Management)
: waf ruleset 효율성 증대

### 1. background
주로 regular expression(정규표현식)으로 만들어지는 WAF ruleset
- protected application의 구조나 expected input type 고려하지 않는, 동일한 방식으로 적용됨 
	- 전자상거래 & 검색 서비스 application, 다른 traffic 특성 가짐
- ruleset author 의 경험 기반으로만 score 부여됨
- application의 context 고려 필요

=> 실 서비스 개시 전, FP 줄이기 위한 조정 필요
- disable overly sensitive rules
- adjust detection thersholds(임계점)
- reweight rule scores
~ 조정단계, 대부분 수동적, 시간소요 큼, 오류발생 가능성 큼. 
~ 오탐 줄이더라도 탐지기능 저하되어 balance 못이룰 수 있음
	accuracy 와 security 사이의 suboptimal tradeoff

=> ML 기반 tuning 프로세스 자동화
OWASP CRS 기반으로 ML 기술 적용.

규칙 삭제 x, 중요도만 변경
score 재계산, threshold 재조정
	detection achived through sum of the scores of matched rules, blocking incoming request if threshold is exceeded,

실제 데이터 사용해 simple linear model 학습.
- 모든 rule 조합, 탐색
- 실제 traffic & attack pattern에 따른 rule score 조정
- FP과 detection accuracy 사이의 최적의 균형 찾기

[owasp warm 웹페이지 원문](https://owasp.org/www-project-waf-advanced-ruleset-management/)
[owasp warm github](https://github.com/OWASP/www-project-waf-advanced-ruleset-management/blob/main/index.md)
	[ModSec-Learn: Boosting ModSecurity with Machine Learning - 24년 4월 연구](https://arxiv.org/abs/2406.13547)
	
