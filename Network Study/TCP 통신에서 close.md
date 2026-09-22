[[>Computer network]]
client측의 클라이언트 소켓을 close 하더라도
`close(sock);`

이와 연결되어있는 server측의 클라이언트 소켓 또한 close(or shutdown) 해줘야
client 의 연결이 정상적으로 종료된다. 
(클라이언트의 소켓만 닫아도 서버는 닫지 않은 상태라면 클라이언트 파일이 끝나지 않고 무한대기..)

