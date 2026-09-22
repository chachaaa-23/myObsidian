#### **1. [error]GPT api 와 통신이 왜 안되지? 

이전 버전에서 통신이 잘 되었던 이유는:

1. **`acquireVsCodeApi`가 호출되었기 때문**입니다. 이를 통해 웹뷰와 확장 프로그램이 메시지를 주고받을 수 있었습니다.
2. **`window.addEventListener("message")`로 데이터를 수신하는 코드가 포함**되어 있었고, 이를 통해 확장 프로그램에서 전송한 데이터를 처리할 수 있었습니다.

현재 버전에서 통신을 복구하려면:

1. `webviewscript.js`에 `acquireVsCodeApi` 호출을 추가하세요.
2. 메시지 수신 처리를 위한 `window.addEventListener("message")`를 포함하세요.

<공식문서>
	**웹 뷰에서 익스텐션으로 메세지 패싱**
	웹 뷰에서 익스텐션으로 메세지를 보내는 것 또한 가능합니다. 이는 웹 뷰 내부에서 특별한 VS Code API 오브젝트의 `postMessage` 함수를 통해 가능합니다. 이 VS Code API 오브젝트에 접근하기 위해, 웹 뷰 내부에서 `acquireVsCodeApi`를 호출 하십시오. 이 함수는 오직 세션별로 한번씩 실행 가능합니다. 이 메소드에서 리턴된 VS Code API 인스턴스를 사용하여 다른 사용하고자 하는 함수에 사용하십시오.

---
#### **2. [error] vscode 상에서 데이터 주고받기**
**`vscode.postMessage`는 웹뷰에서 확장 프로그램으로 데이터를 전송**합니다.
**`webview.onDidReceiveMessage`는 확장 프로그램이 웹뷰로부터 데이터를 수신**하는 이벤트 핸들러입니다.

현재 `postMessage`로 데이터를 웹뷰로 전달하고 있지만, 웹뷰에서 해당 메시지를 처리하는 이벤트 리스너(`message` 이벤트 리스너)가 없기 때문에 데이터를 화면에 띄우지 못하고 있습니다. 이 문제를 해결하려면, **웹뷰의 JavaScript 파일(webviewscript.js)**에 `window.addEventListener('message', callback)`를 추가하여 확장 프로그램에서 전달된 메시지를 처리해야 합니다.

- `vscode.postMessage`와 `onDidReceiveMessage`는 **실시간으로 데이터를 전달**합니다.
- 데이터가 특정 스토리지에 저장되거나 버퍼링되어 전달되는 것이 아니라, 메시지가 발생하면 즉시 이벤트가 트리거됩니다.
- 따라서:
    - `vscode.postMessage`를 호출하면,
    - `onDidReceiveMessage`가 즉시 트리거되어 해당 데이터를 전달받습니다.

---

`postMessage`로 전달된 데이터를 처리하려면, `window.addEventListener('message', callback)`를 설정해야 합니다.
``` javascript
// 웹뷰에서 메시지 수신 이벤트 처리 
window.addEventListener("message", (event) => {   const message = event.data; 
// 확장 프로그램에서 전달된 메시지 데이터    
if (message.command === "setData") {
```

데이터 흐름 정리**

1. **사용자가 데이터를 입력하고 제출**:
    
    - 웹뷰에서 `vscode.postMessage`를 호출하여 확장 프로그램으로 데이터를 보냅니다.
2. **확장 프로그램에서 GPT API 호출**:
    
    - `onDidReceiveMessage` 이벤트 핸들러에서 데이터를 받아 GPT API로 요청을 보냅니다.
3. **GPT 응답을 웹뷰로 전달**:
    
    - 확장 프로그램에서 `webview.postMessage`를 통해 GPT의 응답 데이터를 웹뷰로 전달합니다.
4. **웹뷰에서 데이터 수신 및 출력**:
    
    - `webviewscript.js`의 `window.addEventListener("message")`를 통해 메시지를 수신하고, 데이터를 화면에 출력합니다.

---
#### **3. 이전 커밋으로 돌아가기**
##### 1. **해당 커밋으로 이동만 하려면** (`detached HEAD` 상태)
다음 명령어를 사용하세요:

`git checkout d910576`

이렇게 하면 해당 커밋 상태로 워킹 디렉토리를 변경하지만, 새로운 커밋을 만들면 브랜치와 연결되지 않는 `detached HEAD` 상태가 됩니다.

---
