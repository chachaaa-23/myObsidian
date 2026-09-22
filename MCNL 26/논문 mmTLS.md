### 기존의 TLS와 TLS Middlebox
(성능 이슈) TLS MB, huge computational overhead
- MITM 형태로 동작하면서 암호화된 traffic 검사함.
- custom root certificate 을 client 에 설치하여 MB가 사이트를 impersonate(사칭) 할 수 있게 함. (client인것처럼 서버에게 사칭)

- 암호화 트래픽을 양 끝단에 **translate**(해석)하고 **relay**(중개)한다
  (translate and relay encryption traffic from one to the other)

기존 split-connection 구조:
```
client --(TLS connection 1)--> Middlebox --(TLS connection 2)--> server
```

middlebox 의 translate 과정:
```
TLS ciphertext -*복호화--> plaintext -DPI(Deep Packet *Inspection)--> -*re-encrypt--> TLS ciphertext(*forward)
```
.
	 *DPI: network packet의 header와 data payload를 동시에 들여다보며 application-layer 의 위험을 탐지하는 방법(ex. SQLi, XSS)
  
- 양 끝단의 TLS session throughput 성능이 떨어짐
  (throughput of end-to-end tls session 을 43%->73% 로 낮춤)
	- 두 TCP connection 사이에서 **traffic relay** 
	- encrypted content **translate** 
	- 각 network path segment마다 서로 다른 session key 사용하기 때문에 split connection 필요
	- TCP/TLS stack 을 두번 거치며 relay 과정에서 추가적인 processing/memory copy 비용 발생
- 최근, 보안 증가로 인한 TLS MB의 계산 비용도 나날히 증가

- TLS connection을 설정할 때 만들어진 session key, 
그 connection에서 여러 TLS record를 암호화 하는 데 사용함
- client-to-Middlebox(TLS connection 1), Middlebox-to-server(TLS connection 2) 와 같이 
	  다른 path segment 마다 서로 다른 session key 사용해야 하는 문제 발생. (두 개의 TLS handshake)
	-> mmTLS, 하나의 TLS connection 으로 양 종단이 통신하고, 하나의 TLS connection key 를 사용 및 전달함. 

### TLS 의 한계와 이를 극복하기 위한 기존 연구
- 트래픽 검사 성능 향상
	- re-encryption 비용, split-connection의 relay/translation 비용 절약
	- 
  (improves traffic inspection performance)
- Middlebox 가 TLS session key 전달받아서 가지고 있음 -- > client 의 encrypted packet을 middlebox가 decrypt 한 뒤, plaintext을  DPI 검사 함.

새로운 TLS session 을 만들때 **key delivery** 과정:
```
TLS session 생성 -> session key 생성 -> middlebox 에게 secure하게 전달(oob) -> 해당되는 session동안 여러 TLS record 검사
```

- TLS 이벤트 프로그래밍 라이브러리 제공, TLS MB 사용 편리
  (provides TLS event programming library which one can write TLS MB with ease)

- 트래픽 중개 비용 제거 - 보안 TLS 엔드포인트로 **단일 종단 TLS 세션** 유지
  (eliminates traffic relaying cost - single end-to-end TLS session by secure TLS endpoints)
	- 별도의 secure out-of-band TLS session 을 통해 하나의 공통 session key 전달
	- 하나의 TLS connection 으로 양 종단이 통신
	- MB, packet을 재암호화하지 않고, inspection 후에 원래 encrypted TLS record forward (**re-encryption 제거**)
	- MB, TLS connection을 terminate 하지 않음 (split connection 제거)
	- 안전한 **session key delivery** 를 위해 별도의 key-delivery channel 사용함(out-of-band TLS  session) 
		- endpoint authentication 보장
		--> 추가비용 발생! SmartNIC 등을 통해 최소화..
```
decrypt-> plaintext inspection-> 원래 packet 그대로 forwarding--> server
```

- Middlebox compromised(탈취) 방지를 위한 private tag
	- 두개의 key 도출: 
		- k1- 일반 TLS session key, MB도 가지고 있어서 ciphertext decrypt 가능
		- k2- private tag 생성을 위한 key. endpoint 들만 생성 가능. MB가 내용 변조한 것을 숨기지 못하도록
	- TLS record 안에 `ciphertext + original tag + private tag` 붙임.
	- MB가 session key 를 알고 있어도(key1) content를 임의로 수정한 뒤 endpoint 에게 변경여부를 숨기지 못하게 하기위한 두번째 authentication tag

- **private tag** 생성 및 인증 과정의 추가 오버헤드: 
	- first key 를 통해 도출함으로서 비용 최소화
	  (extra overhead for private tag generation and verification: first tag generation 을 통해 도출함으로서 cost minimize)
	- E2E authentication& **content integrity** 보호
- split connection mode 에 비해 성능 2~41 배 향상, 179Gbps traffic relaying throughput 달성

***content integrity**(무결성): 전송 내용 바뀌지 않았음 증명
**endpoint authentication**(종단 인증): 내가 통신하는 endpoint 가 진짜인지 인증
***confidentiality**(기밀성): 다른 사람이 plaintext를 못 봄. 암호화된 ciphertext 봄
--> mmTLS, confidentiality 포기. middlebox 가 traffic을 검사하려면 traffic 봐야하기 때문

#### 1,2 Introduction
- 기업용 TLS MB, 클라이언트가 방문하려는 사이트 impersonate(사칭) -- custom root CA 설치
- 정보를 빼돌리는지 어떤 트래픽을 보내는지 스트림 검사
```
client-> custom root CA 설치-> MB가 server imperonate-> Client가 MB가 만든 certificate 신뢰
```

client-side enterprise MB가 read-write 인 경우, compromised 되었을 때, security issue 발생!
=> READ-ONLY Middlebox 는 MB가 TLS endpoint가 될 필요가 없음!
	e2e TLS session을 그대로 유지하고, session key만 MB에게 전달, re-encryption없이 원래 packet forward. 

MB 가 endpoint impersonate: 
- endpoint authentication 깨짐 (client, 진짜 server 인증 불가)
- content integrity 깨짐 (중간에 content 수정해도 endpoint 모름)
  (all TLS middleboxed share the session keys ... compromised middlebox can modify content without notice)

### 3.1 mmTLS 아키텍처

1) client-MB: 별도의 TLS session
- session key delivery 를 위한 안전한 channel
- inspection bypass 시 null key 전송
2) client- server: 
- 실제 application traffic 을 위한 일반적인 e2e TLS session
- session key 생성

하나의 E2E TLS session 유지, session의 필요한 key material을 MB에 공유
traffic, endpoint 사이에서 수정 없이, re-encryption 없이 relay.

### 3.2 Private tag
#### end-to-end content integrity verification
>mmTLS 의 Middlebox 는 session key (key1)를 공유받아서 client 의 패킷 내용을 read-only로 inspect 하고 tag1을 생성할 수 있다. 하지만 양 종단만 아는 key2 는 공유받지 못했으므로 private tag(tag2)는 생성할 수 없으므로 MB가 compromised 되어서 message 가 modified 되어도 endpoint 에서 해당 태그를 검증하면 변조 사실을 확인할 수 있다.

##### private tag generation

tag =! endpoint authentication...
tag == content integrity/ data authentication

**key 1** : encrypt 와 tag generation 시 사용
**tag 1** : TLS 의 authentication tag 
- key, encryption과 authentication tag 생성에 모두 사용됨 (MB 탈취시 문제 발생)

**tag 2** : private tag
- modify 검증용 태그
- 전체 content 를 다시 hash 하여 tag2를 생성 X, 
- original tag1 에 대해 key2 를 이용한 추가 연산 -> tag2 생성. (private tag 생성/검증 오버헤드, 2-5%)
##### key delivery to the right CPU core using SmartNIC or dedicated CPU core


### Implementation

programming framework

session key forwarder

mmTLS clients

private tag generation


### 결론

18: **Middlebox를 추가했지만 E2E-TLS와 비교했을 때 웹페이지 로딩 시간이 거의 증가하지 않았다.**

19: **"웹페이지가 복잡해지고 요청해야 할 object가 많아져도 mmTLS는 E2E-TLS와 비슷하다."**

20: **“TLS Middlebox 뒤에 실제 DPI(Deep Packet Inspection) 애플리케이션을 붙여도 mmTLS의 성능 이점이 유지되는가?”**

21: 
**“현재 실제 웹사이트들이 어떤 TLS cipher suite와 TLS version을 사용하고 있는가?”**
mmTLS 논문, a, b인 AES_GCM 사용 (실제 사용 가능)


기존 연구:
- MITM MB의 보안 측면 개선 노력했지만,
- split-connection 방식 사용
- read-write 를 포함한 일반적인 middlebox 사용 사례를 함께 고려 
  ~read-only md 성능 개선(client 측 network 에서 traffic monitoring 을 위해)
- 성능 저하 (중복 TLS handshake, 두 handshake 콘텐츠 재조립과 relay, content re-encryption)

mmTLS:
- handshake 과정에 참여 x -> TLS handshake 성능 split-TLS 대비 41.2배 향상

방향성 및 계획
- mmTLS 코드 돌려보면서 실제 라이브러리를 사용하며 하나의 session에서 복호화-패킷검사-forward 흐름 확인해보기
- 기존 split connection 방식과 mmTLS 방식 비교, 얼만큼의 병목이 발생하는지 확인 (decrypt, re-encrypt, session key 전달, ...)

논문에서 SmartNIC 사용해 key를 올바른 CPU로 전달했었는데, 이 과정과 eBPF 연결해서 생각해보기
- eBPF/XDP, DPU 관련 논문 분석 및 실습
- 