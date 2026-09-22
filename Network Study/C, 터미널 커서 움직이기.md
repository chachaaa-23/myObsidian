[[>Computer network]]
260729

`\r` 통해 터미널 상에서 caret (커서) 맨 오른쪽으로 움직이고,
다시 printf 해서 덮어써도
caret 위치만 옮겼을 뿐, 기존에 print 한 바탕 문자들은 안 지워지고 남아있다. 
만약 새로 입력한 문자열이 남아있던 문자열보다 짧다면, 잔상처럼 남아서 괴롭힌다!!
*caridge return

-> 공백으로 지우거나 (너무 복잡함. 잘 안됨;;)
-> 아예 터미널을 clear 하듯, 내 눈에 안보이도록 `\n` 한 뒤, 새롭게 printf 한다. (사용자 입장에서는 별 이상 없게 보임. )
`printf("\033[H\033[2J");`

https://rottk.tistory.com/entry/%EC%BD%98%EC%86%94-%EB%8B%A4%EB%A3%A8%EA%B8%B0-Escape-Sequence-C
https://m.blog.naver.com/tipsware/221009514492
https://blog.naver.com/math717/223773429993
--
https://unix.stackexchange.com/questions/730751/how-to-know-if-the-terminal-understands-0332j-033h
https://student.cs.uwaterloo.ca/~cs452/terminal.html