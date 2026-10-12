:Client-Centric Web Scanner
*semantic tag: 사람이 이해하기 쉽도록 태그 이름만 봐도 역할을 알 수 있는 태그
	ex. `header, section, footer, ...`
*non-semantic tag: 태그 자체에 역할 x, 영역 나누거나 스타일 적용을 위한 컨테이너 역할 
	ex. js 동적생성, `div, span`

## 0. Abstract

1. immersive interaction
scanner, deeply explore modern web application
	-> immersive interaction (몰입형 상호작용) 구현을 위해 
	client-centric (사용자 중심)의 고수준 피드백 채널 결합
2. crawling loop 구성
	- 1) 상호작용 가능 요소 감지
	- 2) UI 상호작용 순서 정렬
	- 3) form 해결을 위한 LLM 사용

## 1.Introduction

**기존방식**.
기존 crawler, HTML parsing 해 static link 따라가는 데 집중 (+ request parameter, form input 과 같은 기본 상호작용)

**문제점.**
Scanner, 다단계 상태 흐름 탐색, 
실제 app에서 다단계 user step으로 발생가능한 backend service input 구성 필요해짐

-> 사용자에게 노출되는 것과 다른 abstraction 수준에서 app을 관찰, 조작함
~ immersive interaction 의 중요성 대두
	-> client가 실제로 웹 사용하는 것과 비슷하게 feedback channel 활용해, crawling loop 설계

**SpiderSapien**.

1. detecting interactable elements
2. ordering UI interactions
3. complex form solving using LLMs

**기존연구와 차별점.**
blackbox crawling loop around user-facing browser feedback 재설계

**Evaluation.** 
기존 방식에 비해 더 많은 웹 탐색 가능, 더 많은 웹 취약점 찾기가능

## 2. Challenges

**Interaction Strategy.**
블랙박스 크롤러와 탐색할 요소 간의 interaction(상호작용) 전략

1)**상호작용 가능한 요소 탐지.**
Detecting Interactable Elements.

client-side JS code 로 인해서 현대 웹 앱, 복잡해짐
	- ex1. html semantic 적게쓰고 non-semantic 많이 쓰게 됨. 이해 어려워짐
	- ex2. js event listener 통해서 element 와 상호작용 처리 x, 부모 element나 전역 handler 에서 처리. 이해 어려워짐

-> 렌더링된 DOM의 runtime 신호 활용
- cursor 상태, tabindex, 시각적 활성화

2)**상호작용 순서.**

now interactable form과 non-interactable elements 간의 차이를 인지 x
-> form을 조기에 닫아버림, 더 상호작용할 기회 없어짐

JS 설정을 안 막아둬서 scanner 가 숨겨놓은 button까지 클릭해버림
-> interactable element 를 찾는것과 동시에,
현재 상호작용 가능한 요소가 주어진 client-side state 에서 무엇인지도 분석해야함
- random 요소 일부 넣어서 무작위성 보장, helps to not stuck in state.

**3)LLM-based Form Solving**

단순 버튼 클릭만으로는 deep state 진입 어려운 현대 web app.
-> 텍스트 입력이 필요한 UI form 도 존재함. 
	- html semantic attribute 통해 scanner 입력 도움을 줄수 있긴 함 (but optional, 제한적)
server-side code 에서 input value 에 대해 validation check 진행할텐데, value를 막 생성하는 것 만으로는 상호작용 알아내기 어려움

## 3. Method

flow.
seed URL -> 현재 page에서 user 가 인식 가능한 signal만 대상으로, browser와 상호작용 가능한 element 찾기

**A. Motivating Example**
문제 1) `button` HTML element 가 아니라 `label` element 사용
	button 을 눌러서 다음페이지로 넘어가야 하는데, non-sementic element 사용
문제 2) `label` 에  직접 event listener 등록 x
	JQuery libarary 의 global event handler 사용
문제 3) 버튼 클릭 후, form 에서 새롭게 보이고 상호작용 가능한 요소 prioritize
	else, scanner, modal 뒤의 차단된 element들과 interact 하려고 시도함
	, not trigger their event listeners, form disappear
	, prevent future form interaction
문제 4) form 제출 단계에서 특정 format의 input value 입력해야 함
	data validation challenges 추가

**B. Client-side Crawling**

**1. Interactable Element Discovery (상호작용 가능한 요소 탐색)**
- 상호작용 가능한 element를 얼마나 식별 가능한지에 따라 scanner 성능 달라짐

기본적인 semantic HTML input type 사용 (`a, button, form, input, ...`)
**+runtime 에 상호작용 가능성을 나타내는 element 지속적으로 추가하기**
-> A 예제의 step1 과정에서 non-semantic element `label` 을 button 으로 인식할 수 있게 해줌

app 개발자들이 실제 인간 유저에게 web app을 어떻게 사용해야하는지 hint 를 주는것에서 착안 (but not always generalized...)
=> HTML 유형이나 framework 관계없이 (non-semantic element, 비표준 DOM 구조, JS framework 달라도),

**element의 `cursor` 동작과 `tabindex` 검사해서, interactable element 추가 식별**
- ex1) element click 가능한지 확인 -> non-semantic element 인 user interface element 위에 마우스 가져갔을 시, cursor 모양으로 바뀌는지 확인
- ex2) element가 click 가능한지 확인 -> pointer 값이나 음수가 아닌 tabindex 값 확인 (`div` tag 인데  `tabindex=1`)
- ex3) React 같은 framework 사용한 SPA(Single Page Application) 에서 `a` tag가 아닌 `div`가 click 이벤트 받는 경우 -> 해당 `div` 에 click event listener 가 직접 연결 x 어도, cursor= pointer 이고 tabindex 존재한다면, interactable 후보로 포함. 
	렌더링된 browser DOM 상태를 바탕으로 interactable 확장
- ex4) `span` 으로 구현된 버튼, 실제 HTML tag는 `button` 이 아닌경우 -> pointer cursor 여부, tabindex 존재여부 확인
- ex5) `div` 로 edit 기능 영역 구현되있는 경우 -> cursor=text, tabindex >=0, 실제 입력 가능한 영역으로 표시되는지 확인 - input flow 구축

런타임 가시성 검사해, list of interactable elements 정리-> prioritize active element
- interactable element set - 현재 표시/활성화된 요소만!
- 다른 요소 아래 숨겨진 요소, 클릭/상호작용 불가함

최적화.
현재 DOM에 있는 element 우선순위 지정
- 전체 app에 걸친 얕은 crawling x,
- client 측의 깊은 crawling 수행가능

-> DOM 모델링할 필요 x, 브라우저에 의해 렌더링된 그대로, 직접 사용

**2. Crawling Strategy**
2단계 crawling 접근방식 사용
step 1, 2 순환해가며 test coverage 달성. 
탐색 다양성 유지를 위해 randomization 
Prioritize active elements ~> DFS 전략 유사

step 1) short, shallow scan
- 새로운 link 찾기
- state graph 따라가면서 app의 새로운 page 나 state 찾기
- 임계치까지 진행
step 2) main phase
- 사용자가 할 법한 행동을 우선순위에 따라서 선택, 행동. 필요시 payload 삽입
- ex) link 따라가기, client-side의 interactable element 클릭, form 입력, payload 삽입
	- 우선순위: new actions, 현 페이지의 active elements, input elements

**C. LLM-based Input Generation**


**D. Implementation**
blackbox crawling loop 개선,
Black window scanner 확장, 
python, 
langChain 사용해 LLM 관련코드 구현
- gemini 2.5 Flash (temp 1.7)

## 4. Evaluation


## 5. Analysis/Discussion





