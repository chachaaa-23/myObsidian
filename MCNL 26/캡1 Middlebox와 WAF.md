Q. WAF 는 뭐고, Middlebox 는 무엇을 하는가? (사용 이유, 특징과 한계점)
A. WAF: application 레이어에서 작동, App 을 보호하는 역할. HTTP traffic, payloads, cookies 와 같은 것들을 검사
Middlebox: network와 transport 레이어에서 작동. 네트워크 트래픽을 관리하는 역할. IP 패킷과 TCP/UDP 헤더 등을 검사

Q. 왜 MB가 WAF 를 위한 것이어야 하는가?
Q. WAF 만 server side 에서 돌리면 안되는 이유는 무엇인가? (WAF+MB인 이유)
A. 서버에서 직접 WAF 를 실행해도 트래픽 '차단' 은 가능하지만, 악성 트래픽이 이미 서버 네트워크 스택에 도달한 후에야 필터링됨. 서버 리소스 소모와 취약점 노출 문제 존재. 
~ WAF를 위한 MB: 트래픽이 서버에 도달하기 전 검사 및 차단. rever proxy mode 로 inline 배치될 때 심층 검사 가능. 

Q. Client-side/ Server-side WAF 실행 차이는?
A. client-side WAF 의 검증은 응답성 향상에 유리하나, 쉽게 우회 가능함. 
보안 측면에서 server-side 검증이 필요함.

##### mmTLS

기존 TLS 들과의 차이점 
1) 일반 TLS Middlebox: 한 endpoint에서 다른 endpoint로 암호화된 트래픽을 중계하는 과정에서 계산 오버헤드 발생. 
2) TLS-extension: middlebox를 key 교환 과정에 참여.
    split connection.  read only, read-write only 로 구분. 
	~mb 손상 시, endpoint 는 손상 여부를 알 수 없음 (client 측에서 read-write MB 사용하는 것 위험하다고 주장..)

- TLS와 MB, 서로간 충돌 발생 (트래픽 내용을 확인해야 하므로 custom root CA 설치, impersonation, 다른 endpoint 인증 불가)


mmTLS 란 무엇인지, 특징
- client 층 MB
- connection termination 없이 end-to-end 통신 지원
- MB read-only. content 수정없이 패킷검사 수행 (수정할 필요없이, 재암호화 없이 패킷 단순히 전달)
- 문제: TLS session 관리, key sharing 오버헤드 -> SmartNIC 를 통해 전달함으로 오버헤드 최소화

traffic 검사 성능 향상 (성능, TLS 속성 보장)
- (성능) **session key 공유**를 통해 traffic relaying cost 제거. (하나의 end-to-end TLS session에서 작동)
- (TLS 속성) endpoint 간의 두개의 키 교환, **private tag** 생성, TLS record modification 보장 (보안)
	- endpoint 끼리만 알고있는 key로 private tag 생성 (콘텐츠 불법 수정 탐지)
	- overhead, key1 을 통해서 tag2 도출, 비용 최소화. (최저비용)
	- 발생하는 overhead 또한 split-connection 에 비하면 감당할만한 수준임

=>  기존 split-connection 대비 향상된 처리량
ephemeral 연결 시 발생하는 TLS handshake overhead 제거. 응답시간 단축
key1 를 통해 private tag도출, 추가에 따른 계산 오버헤드 최소화




