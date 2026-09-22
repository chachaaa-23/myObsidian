**Error n Fix**
1. html tag 보였다, 안보였다 하게 하기
- display: none 이라는 css 를 담은 class 만들고, 이 클래스가 담긴 tag 요소를 getElementById 를 통해 가져온 뒤, 
`loadingContent.classList.add("invisible");`
처럼 add와 remove 하기

2. 큰 html div tag 안에 요소를 하나씩 넣는 경우 (submit 할 때 step 담는 div tag)
- 이전에 작성되어 있던 step들 뒤에 append 될 수 있다. 
	-> submit 보내기 전에 해당 div tag 안의 요소를 초기화하기.
	`document.getElementById(container).innerHTML = '';

