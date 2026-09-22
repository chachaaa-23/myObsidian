[[>Computer network]]
20260717

TCP 소켓 통신 시, server의 입출력버퍼 - client의 입출력버퍼 에서 데이터를 받아오는 것과
server/client 의 application level의 버퍼 (ex. `char buf[BUF_SIZE];`)  에 데이터를 넣어주고 빼는 것을 잘 연결해야 함. 

문자열을 넣기 전, 후로 
`memset(buf, 0, sizeof(buf));` 등을 통해 쓰레기값이 없도록 관리 필요함. 

