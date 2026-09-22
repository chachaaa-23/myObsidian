- fork system call 은 Linux와 Unix system에서 새로운 프로세스 (child process)를 생성할때 쓰인다. 이는 fork() call을 한 parent process 와 동시에 실행된다. 
- 새로운 child process 가 생성된 후, 두 프로세스들은 fork() 명령어 뒤에 이어지는 다음 명령을 실행한다. 
- child process 는 동일한 pc(program counter), 같은 CPU register 를 사용한다. 
- It takes no parameters and returns an integer value. 

*(**program counter**: processor register that holds the memory address of the next instruction to be executed in a program. sequential program execution 에서 CPU가 PC를 통해 instruction 들을 메모리에서 fetch 하기 때문이다. 명령어 실행 순서를 담당한다.)*

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(){
	fork();
	fork();
	fork();
	printf("hello\n");
	return 0;
}
```

![[Screenshot 2025-11-02 at 11.10.52 AM.png]]



(출처: https://www.geeksforgeeks.org/c/fork-system-call/)