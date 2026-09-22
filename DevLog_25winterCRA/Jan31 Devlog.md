#### 1. `onCompositionStart`와 `onCompositionEnd`
**한글 입력 과정(Composition)** 에서 keydown 이벤트가 중복 발생하는 이유는:

1. OS가 keydown 이벤트를 감지하고 처리함
2. 브라우저도 keydown 이벤트를 감지하고 처리함  
    → **결과적으로 같은 keydown 이벤트가 두 번 실행됨**

- 브라우저에서 IME(입력기) 로 조합형 문자를 입력할 떄 발생하는 이벤트. HTML 요소에서 기본적으로 감지 가능
- 리액트에서 `keyboardEvent` 객체를 보면 `isComposing` 속성이 제공되지 않고, compoistion 단계의 시작과 끝을 위한 이벤트가 별도로 존재
```js
const [isComposing, setIsComposing] = useState(false); 
const handleClickEvent = (event) => {
  if(isComposing) return; 
  
  // 키보드 이벤트 ..
}

<AutoComplete
	onCompositionStart={()=>setIsComposing(true)}
    onCompositionEnd={()=>setIsComposing(false)}
    onKeydown={hanleClickEvent}
/>
```

--> 이런 식으로 isComposing 상태를 별도로 관리하여, 각각의 composition 이벤트가 발생할 때 상태의 값을 변경하면 한글 입력에 따른 keydown 이벤트의 중복을 막을 수 있다.

---

*js에서 이런 문제가 발생하는 경우...*
###### 🔍 해결 방법: `isComposing` 속성 활용

Web API에서는 `KeyboardEvent.isComposing` 속성을 제공하여 현재 입력이 **Composition(조합) 과정 중인지 여부**를 확인할 수 있습니다.  
이 속성은 `true` 또는 `false` 값을 반환합니다.

✅ **즉, 한글 입력 과정 중이면 `isComposing === true`이므로, keydown 이벤트를 무시하면 된다.**

```
const keyboardEventHandler = (event) => {   if (event.isComposing) return;  // IME 조합 중이면 keydown 이벤트 무시      // 나머지 키보드 이벤트 처리 로직 };
```

💡 **주의:** `keyCode`를 이용해 처리하는 방법도 있지만, `keyCode`는 **더 이상 사용되지 않는(deprecated)** 속성이므로 피하는 것이 좋다.


https://velog.io/@dosomething/React-%ED%95%9C%EA%B8%80-%EC%9E%85%EB%A0%A5%EC%8B%9C-keydown-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%A4%91%EB%B3%B5-%EB%B0%9C%EC%83%9D-%ED%98%84%EC%83%81
^참고