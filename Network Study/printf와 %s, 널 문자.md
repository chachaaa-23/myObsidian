
[[>Computer network]]
20260719 c language

`read_cnt = fread((void*)buf, 1, BUF_SIZE, fp);`
fread 를 통해 문자열을 읽어올 때, buf 배열에는 문자열의 끝을 알리는 NULL 문자 `\0` 가 없음. 

`printf("%s\n", buf);`
따라서 printf 로 문자열을 출력해도 `%s` 컴퓨터는 메모리에서 NULL 문자 `\0` 를 만날때까지 계속해서 char 를 읽기만 함. 

-> 대안: 출력할 문자열 최대 길이를 명시적으로 지정
`printf("\n2>> %.*s\n",read_cnt, buf);`
-> 대안2: string이 아니라 char 로 출력 (not sure..)


