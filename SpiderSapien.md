:Client-Centric Web Scanner

#### 0. Abstract

기존의 low-level HTTP 중신 black-box scanner,
실제 사용자가 브라우저에서 상호작용하는 방식에 가깝도록 개선한 client-centric web crawler. 

- **blackbox** **clawling** loop 내에서 몰입형 상호작용을 달성하기 위해 Web application 으로부터 사용자 중심의 고수준 피드백 채널을 결합
- client-side 분석
- 현재 state -> interactable element 발견-> 어떤 action 이 필요한지 확인 후 interaction -> ...

기존 문제점과 개선안.
1.. interactable element 놓침 -> runtime 기반 interactable element detection
- 기존: low level 정보, URL, HTTP Request, parameter, html tag, JS event listener
- 사용자에게 노출되는 browser-level feedback이용해 interaction 판단
- `<a>, <button>` 같은 element 보다 `<label>` 같이 semantic 하지 않은 element 가 실제 버튼처럼 동작 가능함

2.. 상호작용 순서 잘못 선택 -> 상태 고려한 상호작용 ordering
- 커서 특성, text 커서, visibility 등을 종합해서 실제로 interactable 한 element 인지 판단
- 현재 application 상태에서 어떤 interaction이 가능한지 고려gkdu tnstj wjdgka
	- 우선순위: new actions - active elements- input elements

3.. 복잡한 form 채우기 어려움 -> LLM-Based form solving
- ex. 다양한 form input 존재, 입력된 내용이 실제로 존재하는지 등의 디테일한 검증이 들어감
- LLM에게 내용 넘겨서 form 대신 채우도록 함. 

평가 내용.
- server-side code coverage 
- XSS vulnerabilities
- state depth

ZAP 이 SpiderSapien 보다 약간 높은 coverage 기록한 part 있음 (TinyFileManager)
- 이유: ZAP은 모든 HTTP parameter fuzzing 시도-> invalid csrf token 생성-> error handling code 실행함. 
<-> SpiderSapide: valid 한 내용만 생성하도록 함. 

#### ZAP 과 SpiderSapien 비교
**ZAP.**
MITM proxy. 사용자의 browser나 테스트가 zap을 통과하면, http traffic 관찰하여, modified traffic 생성함. 
- crawler 가 application을 탐색하는 과정에서, traffic 이 부수적으로 생성됨 -> 발견된 attack surface 를 대상으로 의도적으로 변형 traffic 생성
- client spider 를 통해 DOM 접근, 실제 브라우저에서 발생한 HTTP message 확인 가능

**SpiderSapien.**
새로운 resource/url 을 발견하는 것
- 크롤링 전략: user-facing interaction 과 application state 탐색 중심

---

피드백 채널: 
1) 상호작용 가능한 요소 감지, EI 상호작용 순서를 합리적으로 정렬
2) form 해결을 위해 LLM을 직교적으로 사용

#### 1.Introduction
**기존방식**
단일 path 작업 해결 접근방식 
- EvoCrawl: UI 상호작용 순서 지정 개선
- YuraScanner: LLM이 어떻게 작업 지향적 스캐닝과 폼 해결을 지원할 수 있는지 보여줌

새로 노출된 상태를 반복적으로 발견, 작업, 재방문 해야할 때 제한적일 수 있음 
	section C

**SpiderSapien**
- 사용자 대면 브라우저 피드백을 중심으로, 블랙박스 크롤링 루프 재설계
- 프레임워크에 구애받지 않는 상호작용 요소 발견, 상호작용 순서 지정, LLM 기반 폼 해결을 반복적인 크롤러 내에 결합함. 
-> 새로운 app 상태 발견, 심층적인 기능 도달 가능
ex) 이전 상호작용 이후에만 후속 작업이 노출되는 application
	<실제 예시~~ >

- 렌더링된 DOM에서 브라우저가 사용자에게 보여주는 "대상 요소의 상호작용 가능성" 신호를 기반으로, 비의미론적 요소까지 포착
	ex. 커서 모양, tabindex 같은 runtime 속성을 이용해, 상호작용 가능 여부 판단

Evaluation. 
ablation studies 제거 연구:
- SS-LLM+ ELM 제거
- SS-ELM+ LLM 제거

#### 2. Challenges

목표: 자동화 스캐너, 웹 어플리케이션 상호작용 추상화 수준을 실제 사용자와 동일한 수준으로 높이기
mimicking normal web application users
-> black box 크롤러의 상호작용 전략 개선

**Interaction Strategy.**
기존: 구성 요소만 수정
- low level URL, network 요청, JS 이벤트 리스너
- 특정 HTML 태그에 의존하는 크롤러로는 발견 못하는 요소들 있음

현 논문: high-level avlid 클라이언트 측 동작 사용
- application 과 상호작용하는 방법
- input 은 다양한 방식으로 구현될 수 있지만, 고수준 속성을 통해 상호작용 가능성 나타냄
브라우저 내의 feedback channel을 기반으로, 클라이언트 측 동작을 선택

1)**상호작용 가능한 요소 탐지.**

단순한 HTML semantic tag 감지 `<a>, <button>`

2)**상호작용 순서.**
`<label> <\label>`
어떤 요소랑 상호작용 가능한지?
상호작용 유형을 올바르게 모델링했는지? 
ex. modal overlay HTML popup 으로 폼 여는 버튼, 누르면 폼 뒤의 가려진 요소하고는 상호작용 불가함
- HTML과 DOM 을 단순 분석하는 스캐너의 경우, 상호작용 순서 인식 불가..

**3)LLM-based Form Solving**

ex. valid email, 숫자, URL, 우편번호 ...
HTML form, 아주 다양하고 그 구조와 유효성 검사 방식도 제각각임. 
-> scanner, 자동으로 추론하기 어려움

Scanner 가 볼 수 없는 server-side code 가 input 에 대한 data validation 검사 수행할 수 있음
-> HTML semantic(의미론)이 지원하지 않는 유효성 검사 강제할 수 있음 

html만 보고 input value의 의미를 알기 어려운 경우 존재함.
-> LLM 에게 form 과 문맥을 전달, 적절한 입력값 생성하도록 해여 form submit.

#### 3. Method

**A. Motivating Example**
기존.
- low-level abstraction
- interactable element 발견, 순서 선택, form input 제공에 실패함
문제점:
1) 실제 HTML element 가 아니라 label 에 button 존재
2) label에 event listender 직접적으로 연결 x. global event handler 달기
3) 차단된 element behind the modal에 상호작용하려고 시도

SpiderSapien.
- interactable element 감지, UI 상호작용 순서 지정
- +YuraScanner 의 방식 사용- LLM 사용해 form 해결

**B. Client-side Crawling**
<동기부여 예제>

identify interactable elements
prioritizing element interactions
using LLM to fill in forms in the scanner

1) Interactable Element Discovery (상호작용 가능한 요소 탐색)
기존 스캐너의 interactable 가능한 요소 식별 가능한지
ex 1) HTML semantic HTML input types
ex 2) event listener가 연결된 요소 식별, hooking

**문제점.**
1)사용자 정의 요소 유형을 포함 x
2)event listener 를 특정 요소에 적절하게 연결 x
3)다른 라이브러리에 대한 framework 별 정적 분석을 구현x

**SpiderSapien.**
semantic HTML input 유형의 초기목록 사용+ runtime interactable
button 역할인 label, 올바르게 식별 (상호작용 가능한 요소 식별)

ex. 요소가 클릭 가능한지 확인하기 위해 pointer value나 요소가 아닌,
tabindex 값 확인

특정 tag에 대한 HTML parsing 이나 JS event listen
![[Screenshot 2026-10-06 at 3.12.17 PM.png]]

2) Crawling Strategy
유효한 interactable element 발견한 뒤, 
해당 요소와 상호작용할 순서 선택하기. 
- client측 상호작용 우선순위 지정해야 함
- 무작위성으로 scanner unstuck

app 상태에서 가능한 동작을 방향성 그래프로 추적
- edge, node: 동작, 상태 나타냄

step 1. 짧고 얕은 scan 수행
- 발견된 link의 임계값에 도달/ 새로운 링크 찾을 수 없을 때까지 새로운 열견 페이지 찾는데 우선순위 둠
step 2. client 측 상호작용과 payload 주입이 우선시되는 main 단계로 이동
- 새로운 작업
- 현 페이지의 활성 요소
- 입력 요소
	를 통해 우선순위 지정

3) LLM-based Input Generation
form= app에 복잡한 user input 제공 가능함. 
form 내용 검증 시, 정규식이 아니라
사용자 정의 JS 코드로. 

**LLM.** 
LLM single/ few- shot 을 통해 수용 가능한 form 입력 제공,
form 의 중요한 요소 요약

**Implementation.**
form 발견 -> 해당 form의 HTML, LLM form 해결 모듈에 제공
-> LLM, 어떤 요소에 어떤 값을 입력할지를 나타내는 `(selector, value)` paired list 생성
-> scanner, 위 output으로 form 제출

4) SpiderSapien 구현
Black window scanner 확장, python
langChain 사용해 LLM 관련코드 구현
- gemini 2.5 Flash (temp 1.7)

4. Evaluation
next week

#### 5. Analysis/Discussion

현재의 web scanner, 의도되지 않은 경로에 너무 집중하고 있음.
 app의 기능을 탐색하지 못하고 취약점 놓치게 된다

ex. ZAP 의 TinyFileManager 에서의 전체 커버리지, SpiderSapien 과 유사함
zap의 일부 커버리지, 잘못된 CSRF token 과 관련한 오류 처리 코드
-> ZAP이 HTTP 수준에서 모든 요청 매개변수를 fuzzing 하기 때문
=> 유효한 토큰으로 form을 성공적으로 제출하도록 의도. (의도치 않은 일부 경로를 회피)






