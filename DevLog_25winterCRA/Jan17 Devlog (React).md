## 1. 사용자's input, edit bug
##### 1) Logic Flow
*수정 중 적은 로직..*
에딧 버튼 누르면
[handleEditClick]
- 현재 인덱스가 에딧할 인덱스로 바뀜 
- 현재 context가 에딧할 context로 바뀜

사용자, 새로운 입력
[handleContentChange]
- 입력이 들어와서 changeEvent 감지 시
- 변경된 innerText로 새롭게 set함

다른 창을 눌러서 수정을 완료할 시,
[handleBlur]
- updateMessage로 메시지 업데이트
- 현재 바꾼 index를 비움

문제점: 
- 리렌더링이 너무 많이 됨. 새롭게 값을 입력받을 때마다 렌더링됨. 이에 따라 커서 위치가 계속 변동됨. 
- 여러가지 Handle 함수들이 얽혀 있음. 단순하게, 이해하기 쉽게 코드 짜기! 
##### 2) Detail usage
그냥 칸을 누를 시 = handleBoxClick 이 되었을 때, 
	그때의 index 가져오기
	바로 텍스트 눌러서 수정 가능하도록 수정
		contentEditable만 수정되도록 삼항연산자 ㄱㄱ

**onBlur 처리**, selectedBoxInput 이 아니면 실행되도록 인덱스 인식하는 로직 바꾸기. 

##### 3) `.join()` 매서드
- js에서 배열의 모든 요소를 하나의 문자열로 합치는 역할!
- 각 배열 요소 사이에 삽입할 구분자, 괄호 속 인수로 전달가능 (기본값은 쉼표 `,`)

## 2. React 특이사항
#### 1. **div에 contentEditable속성 사용하기**
- `div`를 사용해 `div`를 누르면 값이 수정되게 해주고 싶었다.
처음에는 `input` 태그를 사용했다.  
하지만 그러면 부모 `div`의 스타일을 그대로 가져갈 수 없었다.

- div 안의 innerText의 변경을 감지하려면 `onChange()`가 아닌 `onInput()`을 사용해야한다.  
왜냐하면 해당 함수는 `input`, `textarea`, `select`와 같은 태그에서만 작동하기 때문이다.

#### 2. 👀 onChange

`<input>` `<textarea>` `<select>` 와 같은 폼(Form) 엘리먼트는 사용자의 입력값을 제어하는 데 사용된다.

React에서는 이러한 변경될 수 있는 입력값을 일반적으로 컴포넌트의 state로 관리하고 업데이트한다.

`onChange` 이벤트가 발생하면 `e.target.value` 를 통해 이벤트 객체에 담겨있는 `input` 값을 읽어올 수 있다.

컴포넌트 `return`문 안의 `input` 태그에 `value` 와 `onChange` 를 작성하고,

`onChange` 는 `input` 의 텍스트가 바뀔 때마다 발생하는 이벤트로, 이벤트가 발생하면 `handleChange` 함수가 작동하며, 이벤트 객체에 담긴 `input` 값을 `setState` 를 통해 새로운 state 로 갱신한다.
#### 3. JavaScript data-set 사용법
- JS, DOM 생성 시점에 `data-` 로 시작하는 속성들을 하나로 모아 dataset Map 을 만들어 관리. 

https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-HTML-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8Bdata-%EC%86%8D%EC%84%B1 << 참고..

---
## 4. React  Propts (TypeScript)
#### 1. props 는 뭐지? 
- React에서 하나의 컴포넌트에서 다른 컴포넌트로 데이터를 전달할 때 쓰임. 
	(ex. 부모 컴포넌트에서 자식 컴포넌트로. ) 데이터 흐름을 동적으로 만들고 싶을 때 유용.
	코드를 다시 쓰지 않아도 컴포넌트의 데이터를 동적으로 재사용. 

#### 2. props 사용법
```jsx
function Tool(props) {
	const name = props.name;
	const food = props.food;
	return (
		<div>
			<h1>My name is {name}.</h1>
			<p>My favorite food is {food}.</p>
		</div>
	);
}

export default Tool;
```
1. props를 인자로 전달
2. props를 변수로 선언
3. JSX 템플릿에 변수 사용
4. App 컴포넌트의 props에 데이터 전달

#### 3. TypeScript 에서의 object Type Props 넘기기

##### 부모 컴포넌트
```tsx
import React, { useState } from 'react'
import User from './User'

// 자식 컴포넌트로 넘겨주기 위해 export.
export type UserType = {
	id: number
	name: string
	age: number
	position: string
}

function userList (): React.ReactElement {
// userList 배열 state에 제네릭 타입(userListType[]) 지정.
	const [ userList, setUserList ] = useState<UserListType[]>([
	{
		id: 0,
		name: "chacha",
		age: 22,
		position: "front-end"
	},
	{
		id: 1,
		name: "sangeun",
		age: 21,
		position: "back-end"
	},])

	return(
	<div>
		{userList.map(user => <User key={user.id} user={user} /> )}
	</div>
	)
}
```

##### 자식 컴포넌트
```tsx
import React from 'react'
import UserType from './UserList'

type UserProps = {
	user: UserType // 부모컴포넌트에서 import 해온 `user` type 재사용. 연결시키기!
}

export default function User({ user }: UserProps): React.ReactElement{
// ^부모 컴포넌트에서 선언한 type인 Usertype, 자식 컴포넌트에서 별도로 type을 사용해 !연결!
// ^자식 컴포넌트에서 연결한 UserProps 속의 요소들, 사용할 function의 props 속으로 전달해주기! 
	const { name, age, position } = user
	// ^사용할 props 변수들, argument 로 받아온 props 뭉탱이에 연결
	
	return(
		<div>
			<p>이름: {name}</p>
			<p>나이: {age}</p>
			<p>포지션: {position}</p>
		</div>
	)
}
	
```

**=> 부모 컴포넌트에서 정의한 `user` 객체 타입을 자식 컴포넌트에서 import 하여 재사용.( 객체 타입과 props 타입이 같을 경우)**

**=> 타입이 다를 경우, `type` or `interface` 를 사용해 type 작성한 뒤, props에 타입을 정의(연결시키기)**

##### <요약>
1. 타입 정의(type 선언) -- 부모 
`export type UserType = { id: number; name: string; age: number; position: string; };
`
2. 데이터 준비 및 상태 관리(전달할 데이터를 부모 컴포넌트에서 준비. 실제 값 넣기, 타입은 제네릭으로 설정 -- state, props, data 등) -- 부모 컴포넌트
`const [userList, setUserList] = useState<UserType[]>([/* 데이터 */]);
`
3. props로 전달 (부모 컴포넌트의 데이터 전달) -- jsx에서 자식 컴포넌트를 호출할 때
`{userList.map((user) => <User key={user.id} user={user} />)}
`
4. props 타입 정의(부모가 넘긴 user 객체의 type 설정) -- 자식 컴포넌트
`type UserProps = { user: UserType; };`

5. props 사용 및 구조 분해 할당 (전달받은 데이터, 자식 컴포넌트에서 사용하도록 선언) -- 자식 컴포넌트
`const { name, age, position } = user;`

---
#### 1. Component 가 뭐지?
- like JS function. 
- accept arbitrary inputs (called "props") props 같은 임의의 입력값 허용
- return React elements describing what should appear on the screen (화면에 뭐가 떠야할지 설명하는 것)
- 예시 1 ) 함수형 컴포넌트
```jsx
function Welcome(props) {
	return <h1>Hello, {props.name}</h1>;
}
```
-> 하나의 props object argument 허용, React element 반환. 
- 예시 2) 클래스형 컴포넌트 (예시 생략)

#### 2. Component rendering
- 예시 코드 ) 
```jsx
const element = <Welcome name="Chacha" />;
```
	- user-defined component.
	- it passes JSX attributes and children to this component as a single object ( called props )  

- 실제 사용 예시 코드 
```jsx
function Welcome(props) {
	return <h1>Hello, {props.name}</ h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root));
const element = <Welcome name= "Chacha" />;
	
root.render(element);
```
참고 >> 
https://legacy.reactjs.org/docs/components-and-props.html 
https://www.freecodecamp.org/korean/news/how-to-use-props-in-react/

### 5.  React Hook
#### 1. useEffect
- 리액트 컴포넌트가 렌더링 될 때마다 특정 작업을 실행할 수 있도록 하는 Hook. 
- 컴포넌트에서도 생명주기 메소드를 사용할 수 있음

**`useEffect(function, deps)`**

- function: 수행하고자 하는 작업
- deps: 배열 형태, 배열 안에는 검사하고자 하는 특정 값 or 빈 배열
1. component가 mount 됐을 때(처음 나타났을 때)
	: 컴포넌트가 화면에 가장 처음 렌더링 될 때, 한 번만 실행하고 싶을 때 -> 빈 배열. (배열을 생략한다면 리렌더링 될 때마다 실행)
```jsx
useEffect(()) => { console.log("마운트 될 때만 실행됨");}, []); 
```

2. component가 update 될 때 (특정 props, state가 바뀔 때)
	: 특정값이 업데이트 될 때 실행하고 싶을 때는, deps 위치의 배열 안에 검사하고 싶은 값을 넣어준다. (의존값이 들어있는 배열 deps 라고도 함. dependency)
```jsx
useEffect(()) => { console.log("업데이트 될 때만 실행됨");}, [name]); 
```

3. component가 unmount 될 때(사라질 때) & update 되기 직전에
	cleanup 함수 반환 (return 뒤에 나오는 함수, useEffect에 대한 뒷정리 함수. )
	- unmount 될때만 실행 : 두번째 파라미터로 빈 배열!
	- 특정값 update 되기 직전에 cleanup 함수 실행: deps 배열 안에 검사하고 싶은 값!
```jsx
useEffect(() => {
	console.log('effect');
	console.log(name);
	return () => {
		console.log('cleanup');
		console.log(name);
	};
}, []);
```
 
 *useEffect 안에서 사용하는 상태나 props가 있다면, useEffect 의 deps 에 넣어줘야 함! 
https://xiubindev.tistory.com/100 << 참고..

### 2. useState
- 특정 컴포넌트의 데이터 (변수, 객체, 등) 의 '코드 상에서' 변화된 값을 실제 DOM에도 반영되게 하기 위해 사용
- 일종의 < setter 가 포함된 변수 선언 > 
- 초기값도 할당 가능
- 예시 : 
`const [name, setName] = useState("Chacha")`
