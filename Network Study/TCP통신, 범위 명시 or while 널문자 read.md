20260719 c language, Computer Network study

### read 방법 1
`while(read(clnt_sock, filename, 10) != 0);`
-> 널문자가 나오기 전까지, clnt_sock으로부터 10byte 씩 데이터를 read 한 뒤, filename 이라는 변수에 넣는다. 


### read 방법 2
`read(clnt_sock, filename, 100);`
-> clnt_sock 으로부터 100byte 의 데이터를 read 한 뒤, filename 이라는 변수에 넣는다. 

