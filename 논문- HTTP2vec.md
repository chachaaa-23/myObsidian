최신 unsupervised language representation model을 활용해- HTTP request 를 embedding 후- traffic 내의 이상 현상 분류에 사용함
- 임베딩 공간 생성
- 모델, 해석가능
- NLP 분야의 방법론 가져옴
- HTTP message의 진정한 의미 포착

HTTP 같은 text protocol 관점에서, pattern matching 은 expert knowledge 와 attack를 알려주는 set of rules(patterns) 이 바탕이 되어야 한다. 

feature construction
- ex) bag-of-words, tf-idf or bag-of-n-grams
- , Doc2Vec, fasText, ELMo or BERT

text vectorization, feature learning
HTTP 요청의 vector 표현 얻기. 
- 벡터 표현을 통해 HTTP traffic 분석, 이상 현상에서 특정 토큰의 패턴 식별

--datasets
CSIC2010
CSE-CIC-ID2018
https://registry.opendata.aws/cse-cic-ids2018/
- anomalous traffic 생성하기 위해, DVWA application that was hosted on a single machine 사용
UMP: 지도 제공 및 경로계획 세우는 웹 어플리케이션
- Arachni5 통해 비정상적인 요청 생성
- 