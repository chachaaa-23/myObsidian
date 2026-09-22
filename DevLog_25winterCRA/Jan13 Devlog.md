### TypeScript compilation fails with "TS1479: The current file is a CommonJS module" #2778

-> // @ts-expect-error

## Vite란?

공식문서에 따르면 프랑스어로 빠르다를 의미하고, 빠르고 간결한 모던 웹 프로젝트 개발 경험에 초점을 맞춰 탄생한 빌드 도구이다.

- 개발 시 네이티브 ES Module을 넘어 더욱 다양한 기능을 제공한다.
- 번들링 시, Rollup 기반의 다양한 빌드 커맨드를 사용할 수 있습니다. 이는 높은 수준으로 최적화된 정적(static) 리소스들을 배포할 수 있게끔 하며, 미리 정의된 설정(Pre-configured)을 제공합니다.

#### Create React App대신 사용하는 이유

CRA는 JavaScript로 구성된 Webpack을 사용하는데 속도가 느린편입니다. 평소에는 못느낄 수 있지만 처리해야 할 코드의 양이 많아질 수록 느린 속도를 채감할 수 있습니다.  
위와 같은 단점을 해결하기 위해 **Esbuild**를 기반으로 만들어진 빌드툴인 Vite를 사용하게 됩니다.

> ***Esbuild: Go 언어로 작성된 JavaScript 빌드툴로 속도가 빠르다.**

#### 템플릿 생성하기

React로 작성을 할 것이기 때문에 템플릿 명으로 react를 작성하면 됩니다.  
TypeScript의 경우 템플릿-ts를 붙여작성하게 됩니다.

```shell
# npm 6.x
npm create vite@latest ${디렉터리 명} --template ${템플릿 명}

# npm 7+
npm create vite@latest ${디렉터리 명} -- --template ${템플릿 명}

# JavaScript react 템플릿 생성
npm create vite@latest vite-test -- --template react

# TypeScript react-ts 템플릿 생성
npm create vite@latest {디렉토리 명} -- --template react-ts
```

# React Vite 로 넘어가기
**1. extension.ts 역할**
- vs code extension의 진입점 역할
- extension activate/ deactivate 담당
- vscode api 사용, webviewPanel 생성 ...

 **2. webview.ui 파일 (react-vite code)**
 - webview content 만 담당
 - vs code api와 직접 접근 불가. 
 - vscode.postMessage / window.addEventListener 등을 통해 통신 가능 (webviewscript.js 에서 통신 할때와 같은 방식)
 - 