[[>Computer network]]
260729

stdin 입력받고 `del` 키 컨트롤 시, 
ASCII code `127` 만 조건에 넣고, `\b` 키는 넣으면 안됨! (일반 입력도 조건에 걸려버리게 됨)

---

canonical mode: `enter` 키를 누를때까지 키를 입력받은 뒤 치리하는 mode. 
-> non-canonical mode 를 위해 `ICANON` 비활성화 필요

`ECHO` : 화면에 사용자 입력을 echo 하는 macro
`ICANON` : canonical mode 활성화 macro 

https://m.blog.naver.com/tipsware/221009514492
http://blog.dolba.net/k2club/2554
https://chungcode.tistory.com/105
https://unix.stackexchange.com/questions/382790/how-to-allow-backspaces-in-unbuffered-non-canonical-mode
