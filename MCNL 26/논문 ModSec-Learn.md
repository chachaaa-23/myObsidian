
새로운 rule을 생성하는 것 x,
기존에 있던 OWASP CRS Rule의 severity를 -> 각 rule에 대해 ML이 도출한 weight로 처리 (각 네트워크에 따른 룰 가중치 계산)
CRS rule을 최적으로 결합, 중복되는 rule discard (중복되는 룰 제거)

-> ML 에게 어떤 input feature 를 주어서 룰 가중치를 학습시키는지? 
	 legitimate/malicious request 를 통해서 ML 기법 학습. 

Dataset.
- open-appsec dataset (논문 작성 기준, 508,529개)
	legitimate samples + 458개의 SQLi payloads

부족한 의심 트래픽, 3가지 소스를 통해 open-appsec 데이터셋을 증강시킴
- HTTP Params dataset
- SQLi dataset available on Kaggle
- SQLmap 으로 생성한 새로운 SQLi payload set (SQLmap, 페이로드 난독화를 위해 설계된 탬퍼링 스크립트로 진행)

https://github.com/pralab/modsec-learn
https://github.com/pralab/modsec-learn-dataset

---

데이터셋:
1. Open-appsec dataset
Legitimate Dataset
- 12개 카테고리, 185개의 실 웹사이트에서 수집
- 실제 웹사이트 방문해서 사이트 내의 다양한 작업을 수행함으로써 기록됨. (ex. sign-up, 상품 선택, 장바구니 담기, ...)
- 1,040,242 건의 HTTP request
	https://github.com/openappsec/waf-comparison-project

Malicious Dataset
- 73,924 건의 payloads
- mgm security partners GmbH 로부터 수집됨
	- secLists "Fuzzing", Foospidy 의 payloads, PayloadsAllTheThings, ...
	https://github.com/openappsec/mgm-web-attack-payloads

2. HttpParamsDataset
- bening-anomaly values
- anomaly value- SQLi, XSS, Commandi, PT
- 여러 공개소스를 활용해서 생성함
	- CSIC2010 의 정상 payload, 
	- **sqlmap** 을 통한 SQLi 샘플, xssya **통한** 샘플, **vega scanner** 통한 Commandi와 PT 샘플, **FuzzDB repository** 통한 XSS, Commandi, PT 샘플 생성
https://github.com/Morzeux/HttpParamsDataset

2. Kaggle 의 SQLi Dataset
	https://www.kaggle.com/datasets/sajid576/sql-injection-dataset/data

3. sqlmap 사용
