[[>Computer network]]
20260720

`while((read_cnt = recvfrom(sock, recv_pkt, sizeof(*recv_pkt), 0, (struct sockaddr*)&from_adr, &adr_sz)) != 0){`

UDP 통신, 단순히 recvfrom 으로 읽어들인 read_cnt 로만 while 문의 읽기 조건을 탈출할 수 없음
-> 따로 packet 에 break mode 를 추가하기
`if(recv_pkt->mode == 3) break;`

*keep blocking...*