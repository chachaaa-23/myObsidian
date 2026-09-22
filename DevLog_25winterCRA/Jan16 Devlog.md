### 1. zustand 를 이용한 변수 상태 관리
1. Messages 데이터 타입, interface를 통해 정의. (type / content) (파라미터와 반환타입 적기)
2. useMessagesStore에 deleteMessage 함수 정의 
3. useMessagesStore 를 사용하고픈 컴포넌트에 다음과 같이 함수 호출해서 메시지 관리 가능!
```tsx
const { messages, deleteMessage } = useMessagesStore();
deleteMessage(1);
```

##### deleteMessages - filter 구문 작동방식..
`state.messages.filter((_, i) => i !== index)`
- `filter` method : array method. 배열 순회하면서 조건을 만족하는 요소들만 모아 새로운 배열 반환.
- `.filter( (element, index) => 조건 )` : 배열의 각 요소를 순회하며 callback 함수의 조건을 만족하는 요소만 유지
- element 자리에 `_` 표기해 "사용하지 않는 변수" 나타냄

### 2. 자신 이벤트 전파 방지
`onClick={(e) => e.stopPropagation()}`는 **이벤트 전파 방지**를 위해 사용됩니다. 이를 설정하지 않으면, 자식 요소에서 발생한 클릭 이벤트가 부모 요소로 전파됩니다.

### 동작 차이

1. **설정한 경우 (`stopPropagation`)**:
    
    - 자식 요소(`message-box`)를 클릭하면 이벤트가 부모 요소(`messages-container` 또는 `chat-container`)로 전달되지 않습니다.
    - 즉, 자식 요소의 클릭 동작만 처리됩니다.
2. **설정하지 않은 경우**:
    
    - 자식 요소를 클릭했을 때, 부모 요소의 `onClick` 핸들러도 실행됩니다.
    - 이 경우, `handleOutsideClick`이 호출되어 `selectedIndex`가 `null`로 초기화되기 때문에 **클릭된 메시지 박스가 즉시 선택 해제**됩니다.

### 예시 시나리오

#### Without `stopPropagation`:

- `message-box` 클릭:
    1. `handleBoxClick(index)` 실행 → 메시지 박스 선택 (`selectedIndex` 업데이트).
    2. 이벤트가 부모로 전파.
    3. 부모의 `handleOutsideClick` 실행 → `selectedIndex` 초기화.
    4. 선택된 상태가 즉시 해제되므로 아무 효과도 보이지 않음.

#### With `stopPropagation`:

- `message-box` 클릭:
    1. `handleBoxClick(index)` 실행 → 메시지 박스 선택 (`selectedIndex` 업데이트).
    2. 이벤트가 부모로 전파되지 않음.
    3. 선택된 상태 유지 → 클릭된 메시지 박스에만 테두리와 아이콘 표시.

### 결론

`stopPropagation`을 설정하지 않으면, **메시지 박스를 클릭했을 때 바로 선택이 해제되는 문제**가 발생합니다.  
따라서 자식 요소의 클릭 이벤트를 독립적으로 처리하려면 반드시 `e.stopPropagation()`을 설정해야 합니다.

### 4. useState

`const [selectedBoxIndex, setSelectedBoxIndex] = useState<number | null >(null);`
다음과 같이 useState 에 제네릭 타입을 명시하지 않으면 초기값이 null 이기 때문자 ts에서 null type만 사용할 수 있다고 간주해버림. 

### 5. {condition && (jsx)} 문법
- JS의 && (AND) 연산자는 **왼쪽** 조건이 **참일** 경우에만 **오른쪽** 값을 **반환함**. 
- JSX, null이나 false가 반환 시, 화면에 아무것도 렌더링 x. 
- => condition이 거짓일 때, 코드 렌더링 x

### 6. 인라인 편집기능
`edit` 아이콘 클릭 시, 해당 `div` 블록에서 바로 텍스트를 수정할 수 있게 하려면, 기본적으로 **인라인 편집 기능**을 추가해야 합니다. 이를 위해 클릭 시 텍스트를 `contentEditable` 속성을 가진 입력창으로 변환할 수 있습니다.

### 7. Step 입력 버그
##### 1. **입력한 단어가 기존 단어 뒤에 써지고, 두 번째 단어부터 문장 앞에 써지는 문제**
1) contentEditable 의 기본 동작과 React 상태 업데이트 간의 충돌
- **`contentEditable`은 **DOM의 내용을 직접 수정**하는 방식으로 작동합니다. 그러나 **React는** **상태**(`state`)를 통해 UI를 재렌더링하려고 합니다.
- `onInput` 이벤트에서 `setEditedContent`로 상태를 업데이트하면, React는 상태 변경에 따라 다시 렌더링합니다. 이 과정에서 커서 위치가 리셋될 가능성이 있습니다.

2) innerText 와 커서위치 정보
- `e.target.innerText`를 사용해 콘텐츠를 가져올 때, React가 DOM을 재렌더링하면서 커서 위치 정보가 초기화될 수 있습니다.

> contentEditable에서 DOM 직접 수정하려 함 ( React와 상태 업데이트 하는 방식이 다름 )

> 값 입력 - setEditedContent(e.target.innerText) 로 업데이트 - 리액트 리렌더링 - 커서 위치 리셋

=> 해결 방법
- **커서 위치 저장 및 복원**  
    React가 DOM을 재렌더링하더라도 커서 위치가 유지되도록 해야 합니다.
- **`onInput`과 `onBlur`를 구분해서 사용**  
    실시간 입력 상태를 처리할 때 `onInput`은 필요하지만, 커서 위치 관리를 고려해야 합니다.