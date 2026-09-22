채팅창에서는 내가 메세지를 보낼 때마다, 또는 상대가 나에게 메세지를 보낼 때마다 스크롤이 맨 아래로 내려가도록 해야한다.

생각보다 간단하지만 그래도 정리하고 넘어가려고 한다!

1. 먼저 메세지 맨 아래에 div를 하나 만든다. 콘텐츠는 비운 채로 둘 것이다.

```jsx
<MessagesContainer>
  {messages.map((message) => (
    <Message />
  )}
  <div></div> // <-- 이 위치로 스크롤이 내려오게 할 것이다.
</MessagesContainer>
```

---

2. useRef로 ref 객체를 하나 만들고, 아까 만든 div에 ref 값으로 설정한다.

```tsx
const messageEndRef = useRef<HTMLDivElement | null>(null);

...

<MessagesContainer>
  {messages.map((message) => (
    <Message />
  )}
  <div ref={messageEndRef}></div> // <--
</MessagesContainer>
```

---

1. useEffect에 messages를 종속성 배열에 넣고, 아래처럼 작성해준다.  
    `scrollIntoView()` 메소드는 자신이 호출된 요소가 사용자에게 표시되도록 상위 컨테이너를 스크롤한다.

```tsx
useEffect(() => {
  messageEndRef.current.scrollIntoView({ behavior: 'smooth' });
}, [messages]);
```

---

enter가 있는 padding 을 조정해서 text 가려지지 않게 설정.
`padding-right: 28px; /* 버튼과의 여백을 확보 */`

---

