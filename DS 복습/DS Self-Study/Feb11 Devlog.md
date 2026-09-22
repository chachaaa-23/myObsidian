	## 새 프로젝트 (Contextural Recommendation AI)
사용자의 새로운 입력 내용을 인식. 일정시간 뒤에 다음 내용을 추천해주는 ai 툴
gemini 사용
코디움인데, 웹뷰 없이 간단하고 가벼운 거

오른쪽 아래 작동상태 뜨도록. 클릭 시 작동 상태 표시, 끄기가능 -- 고양이 타닥타닥 앱 참고
wpm fighter 처럼, 입력되는 내용을 받아서 지속적으로 gemini 에게 전송

**흐름**
	1. ✅프로그램 설치
	2. ✅api key setting (at quickpick)
	3. 우측하단 박스에 설정 완료 알림, 자동으로 프로그램 실행.
	4. 사용자, text editor 상의 입력 감지 시 자동으로 gemini 에게 변화 전송 (initial input : 사용자가 보고있는 code 전체 / 실시간 input : 사용자 입력 코드 + 커서 위치)
	5. 사용자가 입력한 내용에 덧붙여서 나올만한 코드를 생성하도록 명령 
	6. 추천 코드 입력받아서 사용자 커서 옆에 회색으로 띄우기. 
	7. tab 누를 시, 사용자 코드에 반영. esc 누를 시, 사라지기
	8. 우측하단 상태창 클릭 시, 프로그램 종료 여부 나옴

---
Bongo cat을 통한 statusbar 에 text 띄우기
- createStatusBarItem
- 위 item에 text를 넣고, show()/hide() 여부 설정
- onTextCahnged 를 통해 document의 text 가 변경되었는지 확인.
- setTimeout 을 통해 사용자가 코드 입력하는지 시긴 여부 확인. 

---

참고링크:
	마소 completion sample
	https://github.com/microsoft/vscode-extension-samples/blob/main/completions-sample/src/extension.ts
	gemini cookbook
	https://github.com/google-gemini/cookbook?tab=readme-ov-file
	gemini api request guideline
	https://ai.google.dev/gemini-api/docs?hl=ko
	BongoCat 예제를 통한 statusbar
	https://github.com/kitgore/BongoCat/blob/main/src/extension.ts
	
