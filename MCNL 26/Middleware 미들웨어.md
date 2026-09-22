[[<WAF (Web application Firewall)>]] [[🐢Akamai WAAP 보호 (26-1 w13)]]

: 서로 다른 OS, HW, Application 간에 원활한 통신과 데이터 교환을 위해 중간 매개역할 하는 software
	- **양쪽**을 **연결**하여 **데이터**를 **주고받을 수** 있도록 **중간**에서 **매개** 역할을 하는 software
	- Network를 통해서 연결된 여러 개의 컴퓨터에 있는 많은 process들에게 어떤 서비스를 사용할 수 있도록 연결해 주는 software
	- 3계층 client/server 구조에서 middleware 존재함
		- 통상적으로 middleware = WAS 서버
		- ex. Apache, Nginx

---

3-tier
: 3개의 논리 계층으로 분할된 아키텍처 패턴.
	관리 포인트를 3개로 쪼갠 것
	Client -- Server -- DB
		서버: Web Server (front-end 의 정적 페이지를 전용 처리) , WAS (Web Application Server, back-end 의 동적페이지 정용 처리)