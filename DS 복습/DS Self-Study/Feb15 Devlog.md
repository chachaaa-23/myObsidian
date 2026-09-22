## Create React App 개발 도중 problem-solving
1. 추가 질문(additional question), openai 에게 보내기
- 기존 로직을 쓰면 messages[] 에 저장되있는 string 들에 전부 index 가 붙게 됨(no limit) 
	<문제> definition과 step 뿐만 아니라 gpt 의 답변에까지 index 가 붙음.
	-> messages[] 저장할 때, data 와 state 를 따로 나누어서 저장한다.
	-> definition과 step의 갯수를 특정 변수에 저장한 뒤, 해당 길이만큼만 인덱싱 후, 나머지는 no indexing. 

---

If message.type === “result” 
결과 잘 출력하고,
밑에 다음과 같은 div 태그 추가

<div>

<div onClick=“입력창에 궁금한 내용을 적어 전송해주세요.”  > 추가로 궁금한 점이 있나요?

placeholder 에 예시로 물어볼만한 질문 적기.

<div> 새로운 질문을 하시겠나요? 

<div onClick=새로운 질문으로 넘어가시겠습니까? (이전 기록은 ㅣog 에 저장됨. ) yes/no 버튼 띄우기. >

**Yes 를 눌렀을 때, log 가 저장되고, 새로운 timestamp 로 넘어가도록.** 

message 초기화를 했을 때에는

**Step 입력 시 문제점**

  

- 처음 edit 창 누르고 입력 시, 단어 1개만 기존 단어 뒤에 써지고 2번째 단어부터는 문장 앞에 써짐
- 한글 입력 시 조립되는 게 아니라, 흩어져서 입력 됨. 

  

추가할 내용

- Edit 버튼 눌렀을 때, text 창에 focus 가도록. (바로 타이핑 가능하도록)
- 엔터 입력 시, handleBlur 실향

  

- 맨 처음 definition, 한번 입력된 이상 삭제되지 않도록. (문제정의 쓰고 step 썻을 때, definition을 지우면 step 1 이 definition으로 들어가게 됨)

  

- 제출 입력 후, 자동으로 다음 제출 창에 focus가 가도록  설정

**문제점**

  

2. 한글 입력시, 맨 마지막 글자 두번 입력됨 (한글 외의 다른 문자를 입력하거나, 화살표키 등으로 정렬하면 해결. IME 문제일듯? )

-> IME 인식을 위한 flag 변수 만들고, 이에 관련된 html windows event 사용.

  

3. 맨 처음 들어가있는 파일을 인식하지 못함 (다른 파일 클릭했다가 다시 돌아와야 함 )

-> 맨 처음 vscode를 킬 때, 로딩이 완전히 이루어진 뒤 파일을 가져오도록 setTimeout 설정.

  

4. 사용자 입력 언어로 결과가 출력되도록 작업 처리하기. (결과만 번역시키기)

-> gpt에게 결과를 받아서, 이를 사용자가 입력한 언어로 변환시켜서 사용자에게 출력 (message 저장소에는 영어 저장 - 토큰 수 절약)