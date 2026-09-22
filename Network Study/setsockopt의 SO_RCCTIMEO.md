260723
[[>Computer network]]

`setsockopt()` 의 `SO_RCCTIMEO` 를 통한 socket timeout 설정

return value: success completion일 시, -1이 아닌 다른 값 반환. error 시 `-1` . 

사용 시, 
```c
int 변수 = setsockopt();
```
로 쓰면 안됨! 이 친구는 단순히 연결된 소켓이 특정 조건을 만족하는지 여부를 단독으로 평가하는 일종의 함수. 

use like this ...
```c
socktype = recvfrom(serv_sock, recv_pkt, sizeof(*recv_pkt), 0, (struct sockaddr*)&clnt_adr, &clnt_adr_sz); //receive ACK.
//시간 내에 receive 실패 시, recvfrom 에서 -1 반환 (NOT setsockopt)

setsockopt(serv_sock, SOL_SOCKET, SO_RCVTIMEO, &optVal, optLen);  
```



https://m.blog.naver.com/yenuzzang/222207594193
