### 1. conversation log를 적을 폴더와 파일 만들기
현재 열려 있는 폴더의 디렉토리를 확인하려면 `vscode.workspace.workspaceFolders`를 사용하면 됩니다. 이 API는 사용자가 현재 작업 중인 워크스페이스(또는 폴더)에 대한 정보를 제공합니다.

##### 1. **워크스페이스 폴더 확인 코드**
다음 코드를 사용하면 현재 열려 있는 워크스페이스 폴더 경로를 확인할 수 있습니다.

```typescript
vscode.commands.registerCommand('extension.getWorkspacePath', () => {
    const workspaceFolders = vscode.workspace.workspaceFolders;

    if (workspaceFolders && workspaceFolders.length > 0) {
      const workspacePath = workspaceFolders[0].uri.fsPath;
      vscode.window.showInformationMessage(`Workspace Path: ${workspacePath}`);
    } ...
```

##### **`workspaceFolders`가 반환하는 값**
- 열려 있는 모든 워크스페이스 폴더를 배열로 반환합니다.
- 각 폴더는 `WorkspaceFolder` 객체입니다:
  - `uri`: 폴더의 URI (`vscode.Uri` 타입).
  - `name`: 폴더 이름.
  - `index`: 워크스페이스 목록에서 폴더의 순서.

1. **다중 루트 워크스페이스 지원**
   만약 VS Code에서 여러 폴더가 열려 있다면, 배열의 모든 요소를 순회하여 처리해야 합니다.

   ```typescript
   if (workspaceFolders) {
     workspaceFolders.forEach(folder => {
       console.log(`Folder Name: ${folder.name}, Path: ${folder.uri.fsPath}`);
     });
   }
   ```

2. **URI에서 파일 시스템 경로 추출**
   `workspaceFolders[0].uri.fsPath`를 사용하면 URI에서 파일 시스템 경로를 얻을 수 있습니다.

---
### **2. 에러들**

[Error - 11:36:45 AM] An unexpected error occurred:

[Error - 11:36:45 AM] Error: Could not find config file.

    at assertConfigurationExists (/Users/jessicacha/.nvm/versions/node/v23.1.0/lib/node_modules/eslint/lib/config/config-loader.js:84:23)

    at LegacyConfigLoader.loadConfigArrayForFile (/Users/jessicacha/.nvm/versions/node/v23.1.0/lib/node_modules/eslint/lib/config/config-loader.js:341:9)

    at async ESLint.lintText (/Users/jessicacha/.nvm/versions/node/v23.1.0/lib/node_modules/eslint/lib/eslint/eslint.js:914:25)

    at async /Users/jessicacha/.vscode/extensions/dbaeumer.vscode-eslint-3.0.10/server/out/eslintServer.js:1:26981

    at async M (/Users/jessicacha/.vscode/extensions/dbaeumer.vscode-eslint-3.0.10/server/out/eslintServer.js:1:19807)

    at async /Users/jessicacha/.vscode/extensions/dbaeumer.vscode-eslint-3.0.10/server/out/eslintServer.js:1:234554

    at async /Users/jessicacha/.vscode/extensions/dbaeumer.vscode-eslint-3.0.10/server/out/eslintServer.js:1:63886

—> read only directory에 파일을 만들려고 해서 발생하는 에러. 다른 파일에 만들거나 파일 경로를 vs code api 매서드인 ~uri 를 사용하기. 

---

```
Argument of type '{ role: string; content: string; }[]' is not assignable to parameter of type 'string | ArrayBufferView<ArrayBufferLike>'.

Type '{ role: string; content: string; }[]' is missing the following properties from type 'Float64Array<ArrayBufferLike>': BYTES_PER_ELEMENT, buffer, byteLength, byteOffset, and 3 more.ts(2345)
```

—> argument passing error. **writeFile**의 data 는 string type만 처리한다. 배열은 처리불가. JSON.stringify 로 처리해야 함. 

---
vscode에서는 path 를 단순히 / 를 사용해서 나타낼 수 없다. (아마 보안?) 다음과 같은 함수를 사용해야 한다.
--> const conversationPath = path.join(context.extensionPath, "conversation");

---

한번 입력한 뒤, 새로운 창을 켰을 때 secret 오류 발생 

--> 메인 파일인 extension.ts에 명령어를 직접 입력해야 하는 

vscode.commands.registerCommand("openAI.setAPIKey", setAPIKey),

에서 함수 부분에 **argument로 context**를 넣어주지 않았기 때문에 직접 명령어를 입력하는 곳에서는 제대로 인식하지 않음. (**context의 정보가 있어야지 secrets 에 올라갈 수 있다.**)

ex. craConfigManager.ts 의 setAPIKey(context) 함수
```
await context.secrets.store("OPENAI_API_KEY", apiKey);
```

수정 이후 코드 : 
```
vscode.commands.registerCommand("openAI.setAPIKey", async () => {

await setAPIKey(context);

}),
```
=> *함수의 passing argument를 잘 확인하자!! *

---

### **3.  fs (node.js) 통한 파일 생성**

var fs = require('fs');
--> fs 선언.
  
fs.writeFile('파일경로/파일명','파일에들어갈내용',function(err){
```
    if (err === null) {
        console.log('success');
    } else {
        console.log('fail');
    }
});
```

- fs.writeFile 의 첫번째인자는 파일명 , 두번째 인자는 파일의 내용 , 세번째는 콜백함수 이다.
- 콜백함수엔 err을 인자로 줄수있으며 err은 에러가 낫을경우이다.
https://dydals5678.tistory.com/96
https://www.daleseo.com/js-node-fs/
---
#### 4.1 OpenAI 에게 question 보내는 형식 (예시)
```javascript
const response = await openai.chat.completions.create({
	"max_tokens": 1024,
	"temperature": 0.69,
	"top_p": 1,
	"frequency_penalty": 0.4,
	"presence_penalty": 0.6,
	"model": "gpt-4",
	"messages": [
		{
		"role": "system",
		"content": " <INSTRUCTION> ",
		
		"role": "user",
		"content": " <QUESTION> "
		}
	]
}
```
---
#### 5. **`push(log)`와 `push(...log)`의 차이**
##### `push(log)`
- `log` 배열 전체를 **하나의 요소**로 추가합니다.
- 이미 있는 배열에 중첩으로 배열이 들어감.. 
##### `push(...log)`
- `log` 배열의 각 요소를 **개별 요소**로 추가합니다.
- `...log`를 사용하면 배열을 풀어서 개별 요소로 다룰 수 있습니다.
- 데이터를 관리할 때 배열 내부에 중첩 배열이 생기지 않도록 방지합니다.
- 새로운 대화 기록(`log`)의 개별 항목을 기존 데이터(`existingData`)에 추가할 때 굿

