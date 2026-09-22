### HTTP request 생성 측면

1) spider나 실제 App 사용을 통해 정상 HTTP request 확보 
	   spider 직접탐색/ 브라우저.웹에서 발생하는 HTTP traffic 을 proxy로 관찰, site tree 생성
2) Active Scan, 확보한 request를 기준으로 공격 가능한 input vector(parameter) 선택 (정상 request들의 url 분석해 공격할만한 parameter 찾음. request에서 변형 가능한 요소 찾기)
3) 변형 파라미터 생성 및 서버로 전송 — scan policy통해 어떤 rule을 사용해 active scan 할지 지정함. parameter 에 들어갈 수 있는 공격을 시도함 (input vector)
4) 원래 요청의 응답과 변형 페이로드 응답 비교, alert 결정

[Scan Policy_ZAP](https://www.zaproxy.org/docs/desktop/start/features/scanpolicy/?utm_source=chatgpt.com)
- `getBaseMsg()` - 새 요청 만들기 - parameter 변경(`setParameter()`) - 전송, 서버의 변화 확인
- => ZAP 이 미리 확보한 HTTP message 를 scanner 가 사용할 기준 메시지로 삼고, 그 메시지를 복제 및 변형해가며 공격용 message 를 만든다. 필요하다면 원본이나 정상 입력 결과와 공격의 결과를 비교해서 alert 를 결정한다. 

- 정상 traffic: spider 가 만든 request / 실제 브라우저가 만듬 request
  https://github.com/zaproxy/zap-extensions/blob/edcf27b91d2caff35d041eb95dfeb9e396c80b52/addOns/spider/src/main/java/org/zaproxy/addon/spider/SpiderScan.java#L51
- 공격 traffic: 확보한 http 메시지에서 parameter, payload 선택, 변형 파라미터 생성

```
[원본 request]
	Active Scan infrastructure
      ↓
[어떤 parameter를 공격할지]
	Input Vector
      ↓
[어떤 공격을 할지]
Scan Policy 의 - Active Scan Rule
      ↓
[payload]
	기존 정상 메시지 복제 및 변형, 전송
	getNewMsg()
	setParameter()
	sendAndReceive()
      ↓
[response 비교]
	Alert
```
#### 개념
SW security testing
취약점 분석, 발견, 악용하려고 시도.
(discover vulnerability, attempt exploitation of vulnerability)

penetration testing tool (침투 테스트 도구)
- search vulnerabilities, be addressed (취약점 탐색 -> 해결되도록)
- mitm proxy

intercept- 요청을 가로채 정밀한 분석 (로그인 흐름, 복잡한 요청 재현)
repeater- 요청 반복 재생성

browser - server 사이의 트래픽 들여다봄. 요청 가로채고 수정해 재전송.
- HTTP request/ response 분석
- cookie, header, token 확인. 인증흐름 검증. 입력값 변조, 반복 테스트, ...

API 를 활용한 스캔 자동화 가능, CI/CD pipeline에 붙이기 유용

##### 특징
plugin architecture. 
새로운 기능은 add-ons 에 구현되어있음
-> ex. SQLi 의 검사 방법 자체, ZAP 본체가 아니라 Actice Scanner Rules add-on의 `ascanrules` 에 들어있음

**Passive Scan = 지나가는 HTTP 요청/응답을 보고 분석**  
> *"Scanning is also performed in a background thread to not slow down exploration. 
> Passive scanning is good at finding some vulnerabilities and as a way to get a feel for the basic security state of a web application and locate where more investigation may be warranted."*
**Active Scan = 새로운 공격용 HTTP 요청을 만들어 서버에 보내고 반응을 분석**

---

### GUI 설명

##### 흐름
정상 request -> parameter 선택-> payload 삽입-> 전송

```text
ZAP이 접근한 사이트(site tree) &spider로 브라우저 탐색.
URL/ request/ parameter 발견
->
알고있는 HTTP 요청을 대상으로 공격할만한 parameter/input 찾음 (active scan)
->
active scan rule, 여러 scanner rule을 통해 target 취약점 검사(각 공격 방법과 payload를 이용해 input 변형)
-> 
변형한 요청을 서버로 전송 (parameter에 공격 payload 테스트)
->
서버 응답 변화 분석
->
취약점의 증거 발견 시, Alert 생성
```

- proxy 를 통과하는 request/ response 를 passive scan 대상으로 삼음
	-> 프록시 요청/url이 많을수록 공격 대상 많아짐

- *payload list에 각 공격별로 여러 테스트용 값들 존재*
- url과 parameter 입력값을 확인하고 공격대상 선정 
- 취약점과 관련된 오류 발견 시, alert 띄움 (not exactly)
##### spider
- 전통적 크롤러
- 도달 가능한 링크 수 `URLs Found`
- `Max Depth` zap이 설정한 depth limit 에 도달한 경우

AJAX Splider
- js 인식 크롤러 (사용x, old fashioned)
##### Client Spider
- 최신 js 인식 크롤러

#### Active Scan
*attemps to find potential vulnerabilities by using known attacks against the selected targets*
- spider들이 찾은 URLs <- 실제 공격 페이로드 주입해서 취약점 탐지
- 대상 서버에 실제 부하/변조 일으킴

새로운 정상 request를 처음부터 생성 x,
이미 선택된 request를 대상으로 공격을 수행하는 기능 (starting point url 부터 확장)

공격할 parameter = 사용자가 값을 입력할 수 있는 부분 
공격 대상= input vector (active scan input vectors. ex. url query string, post data, plaintext ...)

active scan 을 진행과정: 
add-one 안의 scan policy가 여러가지 공격 rule을 선택
-> parameter 에 오류 테스트 값 넣어
-> ZAP 내부함수 `sendAndReceive()` 로 요청 전송, 응답 확인

스캐닝 유의점!
- *can only find certain types of vulnerabilities*
- *logical vulnerabilities, such as broken access control won't be found bu any active/automated vulnerability scanning*
-> 모든 취약점 찾기 위해서는 manual penetration testion 수행해야 함

- 실행되는 규칙들 scan policies 로 구성됨, 원하는 만큼 설정 가능
(https://www.zaproxy.org/docs/desktop/ui/dialogs/scanpolicymgr/)

모드에 따른 공격 범위 제한
- safe, standard, attack mode ...

공격 테스트
1. (forced browsing) 파일 탐색 게열 스캔 규칙
	- ex. backup/hidden file disclosure
	- 동작원리: spider가 이미 진짜 파일 발견 -> active scan 이 또다른 확장자를(.bak, .old, .pyc, ...) 붙여서 존재하는지 GET 으로 열어봄
2. 밝혀진 url, request/response 를 바탕으로 active scan

--paraneter: 공격된 parameter
--attack: active scan rule이 실제로 공급한 payload
--risk: 취약점의 상대적인 위험도
--confidence: ZAP이 alert 결과를 얼마나 확신하는지 (FP~ confirmed)

##### flow
1. spider 를 통해 실제 Application을 탐색
2. 탐색한 내용을 바탕으로 정상적인 HTTP 요청을 확보함
3. 확보한 요청에서 공격할 수 있는 입력값(parameter) 찾기
4. ZAP 이 미리 정의해 둔 공격용 값(payload)을 parameter에 넣어, 변형된 요청을 생성 후 전송
5. 기존 정상 요청의 응답과 공격 요청의 응답을 비교, 취약점 여부를 판단

`sqlInjectionScanRule.java` : `SQL_CHECK_ERR` 목록에 있는 공격할 파라미터 예시들을 임의로 넣어본다. 
[ZAP Active Scan Rules](https://www.zaproxy.org/docs/desktop/addons/active-scan-rules/?utm_source=chatgpt.com)

-> 정상 요청을 어디에서 가져오는지?
-> 정상 요청의 parameter 는 어떻게 찾는지?
-> baseline URL 부터 구체적인 node 로 어떻게 확장되는지? (어떻게 request 를 학습하는지?)
	=> spider 를 통한 탐색, site tree 에 추가. 브라우저도 사이트를 직접 사용한 경우
-> payload 를 어떤 방식으로 끼워 넣고, http 요청을 만드는지?
	=> 확보된 요청에서 공격할 수 있는 입력값 (parameter) 찾아서 수정. 

##### 예시
1. 정상 입력값 확인
![[Screenshot 2026-09-05 at 8.39.16 PM.png]]

2. ZAP, 공격을 위해 '( 넣음 (500 에러 발생)
![[Screenshot 2026-09-05 at 8.30.08 PM.png]]

3. 정상 value 로 재확인 (S4feV4lu3)
![[Screenshot 2026-09-05 at 8.29.16 PM.png]]

`SqlInjectionScanRule.java` 예시

```java
                HttpMessage msg1 = getNewMsg();
                String sqlErrValue =
                        prefixStrings[prefixIndex] + SQL_CHECK_ERR[sqlErrorStringIndex];
                setParameter(msg1, param, sqlErrValue);

                // send the message with the modified parameters
                try {
                    sendAndReceive(msg1, false); // do not follow redirects
                } catch (SocketException ex) {
                    LOGGER.debug(
                            "Caught {} {} when accessing: {}",
                            ex.getClass().getName(),
                            ex.getMessage(),
                            msg1.getRequestHeader().getURI());
                    continue; // Continue to the next prefixString
                }
                countErrorBasedRequests++;

                if (msg1.getResponseHeader().getStatusCode() == 500
                        && this.getBaseMsg().getResponseHeader().getStatusCode() != 500) {
                    // Double check that the service doesn't respond with a 500 for all invalid
                    // values
                    HttpMessage msgSafe = getNewMsg();
                    setParameter(msgSafe, param, "S4feV4lu3");

                    try {
                        sendAndReceive(msgSafe, false);
                    } catch (SocketException ex) {
                        LOGGER.debug(
                                "Caught {} {} when accessing: {}",
                                ex.getClass().getName(),
                                ex.getMessage(),
                                msgSafe.getRequestHeader().getURI());
                    }
                    if (msgSafe.isResponseFromTargetHost()
                            && msgSafe.getResponseHeader().getStatusCode() != 500) {
                        // Internal Server Error only when its an SQLi attack, a good enough
                        // indication in this case
                        sqlInjectionFoundForUrl = true;
                        sqlInjectionAttack = sqlErrValue;

                        newAlert()
                                .setConfidence(Alert.CONFIDENCE_LOW)
                                .setName(getName())
                                .setParam(param)
                                .setAttack(sqlInjectionAttack)
                                .setEvidence(msg1.getResponseHeader().getPrimeHeader())
                                .setMessage(msg1)
                                .raise();
                        continue;
                    }
                }

```