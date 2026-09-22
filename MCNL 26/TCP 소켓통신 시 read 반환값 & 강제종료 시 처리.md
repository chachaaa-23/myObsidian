### read 반환값
1. 정상: read 한 만큼의 byte 수 (>0)
2. EOF: 클라이언트, 정상 종료 시, 마지막임을 알려주는 EOF (== 0) 반환
3. error: 무언가 문제 발생 시 음수 반환. 구체적인 정보, `errno` 로 확인가능

-> ex. 
```c
if(read_cnt < 0){
	if(errno == ECONNRESET)   //
	...
```

### if 강제종료...?
server <-> client 간 통신을 통해 read/write 주고받음. 
이때 client 에서 강제 종료 (network 끊김, `ctrl c`  입력, ...) 로 인해 연결이 끊긴다면...

가능성1) SIGPIPE error 발생 (broken pipe)
	소켓 통신 중, client가 소켓 닫음 -> server 닫힌 소켓으로 데이터 전송 시
	>해결: server 의 main thread에서 SIGPIPE 에러 무시. 
	https://velog.io/@c4fiber/SIGPIPE-handle

가능성2) TCP RST 패킷 발생 
	포트 닫힘, 프로그램 강제 소켓 종료로 인한 app 오류, ...
	https://coconuts.tistory.com/1487
