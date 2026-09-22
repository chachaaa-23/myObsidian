CI 배포 (notion)
prompt design 
https://github.com/namdy0429/GILT?tab=readme-ov-file
---

### while 문 헷갈리는 경우..??

```typescript
switch (message.command) {

case "process":

// 사용자 코드 추가

let textDoc: vscode.TextDocument | null = null;

  

while (textDoc === null) {

textDoc = await pickOpenedDocument(this.context);

if (!textDoc) {

vscode.window.showErrorMessage("No document selected. Please select a document.");

return;

}

}

  

const messageToAppend = [{ role: "assistant", content: textDoc.getText() }];

const updatedMessage = [...message.value, ...messageToAppend];

  

//GPT API 호출

const gptResponse = await this.callGptApi(updatedMessage);

//웹뷰로 결과 전달
```

textDoc 변수를 만들어서, 이를 통해서 message를 주고받고 한다. 
craConfigmanager.ts에서 읽어오는 기능을 구현하고, 이를 webview.ts 에서 submit 버튼을 눌렀을 때 무조건 실행되도록 객체로 넣는다. 

원래는 비동기 신호 쪽에서 제어를 안해서 그런가 알아봤었는데, 그냥 while문을 잘못쓴거였다. 
https://squirmm.tistory.com/entry/ReactJavaScript-Promise-%EA%B0%92-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0