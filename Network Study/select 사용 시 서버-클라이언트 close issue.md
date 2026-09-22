[[>Computer network]]
260724

`FD_SET` 통해 readable/writable 신호를 등록했다고 가정한다.

서버-클라이언트 간의 상호작용 후, 연결된 read/write 신호를 둘 다 `FD_CLR` 해줘야지 연결된 fd들이 정상적으로 fd_set에서 종료되고, `close(i)` 가 가능하다. 

*pair 맞추기 필수!*
*else, keep blocking...*

---

`write(i, buf, read_cnt);`
select() 를 통해 멀티플렉싱 서버 구축 시, 클라이언트에게 read/write 할 때엔 `clnt_sd` 가 아니라 i 라는 sd 를 통해서 전달해야 함. 