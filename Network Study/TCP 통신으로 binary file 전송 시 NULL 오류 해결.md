260724
[[>Computer network]]

기존: message 버퍼에 저장 후, strcpy 에 옮김 
*>>!문제발생! binary data 를 string copy 로 옮기려니까 NULL 로 바뀜!!

~> 해결방안:
1) `memcpy` 로 binary data 통째로 옮기기
2) fread 시 message 대신 직접 `send_pkt->msg` 로 받기

```c
read_cnt = fread(message, 1, BUF_SIZE, fp); 
// strcpy(send_pkt->msg, message);

memcpy(send_pkt->msg, message, read_cnt);
```

