 "compilerOptions": {
		"module": "Node16",
-> ESNext 를 사용해서 더욱 포괄적으로 모듈을 사용 가능. 

 "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "Node",
    "esModuleInterop": true,

  // for the documentation about the extensions.json format
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "connor4312.esbuild-problem-matchers",
    "ms-vscode.extension-test-runner",
    "tobermory.es6-string-html"
  ]

-> vscode.extension.json. 다음 파일을 통해 추천 등록하기. 

"dependsOn": ["build-react"
-> vscoe/task.json. 자동으로 react build 되도록 설정.

