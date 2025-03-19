
1. 실행 중인 프로그램
2. 프로세스 상태과 관련된 실행 흐름 그 자체
	- 프로세스의 실행과 관련해서 영향을 주고 받는 모든 것들
기본적으로 프로세스는 하나만 실행된다.

execution stream + process state

---
### Process state

Memory context
프로세스가 사용한 메모리 ( code segment, data segment, stack segment, heap )
프로세스마다 다름

Hardware context
프로세스가 실행되면서 사용되는 하드웨어
Program counter : 현재 수행 중인 instruction 주소 
CPU register 

System context
프로세스 자체 관리
Process table, open file table, page table

프로그램은 디스크에 저장된 passive entity 지만, 프로세스는 active ( 실행 상태 ) 이다.
하나의 프로그램은 여러 개의 프로세스로 동작할 수 있다.

---
### Multiprogramming vs multiprocessing

1. Uniprogramming
	시스템에서 돌아가는 프로그램이 하나 -> 프로세스도 하나
2. Multiprogramming
	프로세스가 메모리에서 여러 개 동작
3. multiprocessing
	CPU 여러 개가 프로세스를 여러 개 수행
	CPU 관점 / CPU가 여러개
	multiprocessing이 되려면 multiprogramming이 되어야 하지만, 반대는 아님

---
### Process Control Block

Process control block, PCB
여러 프로세스를 테이블 형태로 모아두는 곳

Process state(위랑 다름 / 실제 상태 : 실행중 )
Program counter
Register
Scheduling information
Memory management information : 프로세스가 사용하는 메모리 공간
Accounting information : 사용 중인 정보
I/O status information 

Process table -> linked list로 관리

---
### Process State

![[Pasted image 20250313170002.png]]

new 
새로 만들어진 프로세스
running
프로세스 실행중
ready 
프로세스 준비 상태 상당수는 ready 상태임! 돌 수는 있지만 CPU 자리가 한정적이어서 대기 상태임
waiting : I/O 나 어떤 이벤트에 의해서 잠시 정지
terminate : 정지

---
### State Transition

ready queue 
모든 프로세스는 일단 여기로!

Device queue ( I/O waiting queues ) 
I/O request가 오면 여기로

Job queue
전체 프로세스 관리

---
### Schedulers
#### short term scheduler (CPU scheduler)
CPU에서 어떤 프로세스가 돌게 할 것 인가 고르기
생각보다 자주 깨어남 
한 자리를 여러 프로세스가 번갈아서 사용함 -> 굉장히 빠름
scheduler 자체는 굉장히 빨라야 함! 어

Long term scheduler (job scheduler)
어떤 프로세스를 job queue에 넣을 것인가?
너무 많은 프로세스들이 실행되면, 과부화 되기 때문에 일부러 격리!
메모리에 몇 개의 프로그램이 올라가는지 결정한다!
자주 호출되지 않기 때문에 느려도 됨

프로세스를 구분해서 실행
1. I/O-bound process  I/O 에 더 많은 시간을 사용하는 프로세스
2. CPU-bound process CPU에 남아서 오랫동안 실행되는 프로세스

Long term schedule은 둘의 적절한 조합을 찾아준다!

---
### CPU Switch From Process to Process

CPU가 다른 프로세스로 바꿀 때 현재 돌아가는 old process의 상태를 저장해야 한다. 저장을 하고 기존에 돌던(새로운) 프로세스에 대한 정보를 다시 불러온다. 이걸 Context switch 라고 한다. 

돌아가고 있던 프로세스 정보는 PCB 에 저장한다.

교체 과정 자체도 CPU에서 일어나기 때문에 빠르게 이루어져야 한다. 교체 과정 중에는 CPU가 할 일을 못 하잖아!

이 시간을 빠르게 하기 위해서 hardware에서 도와준다!

---
#### Threads
하나의 프로그램 안에 여러개 thread를 한다?
메모리에 올라온 것은 하나인데 여러 방향으로 실행?
execution stream이 여러 개로 분화시켜서 실행

---
### Process Creation and Termination


Operations on Processes
프로세스를 만들고, 종료를 하는 기능을 수행해야 한다. 

1. build from cratch 
	1. code, data 를 program file에서 읽어서  memory에 적재  - loading
	2. Create empty stack => memory context
	3. PCB 만들기
	4. 해당 PCB -> ready Q에 넣고 
	사실 이렇게 안 함.........
	
2. 복제 ( process )
	1. 현재 state 저장 ( PC, register )
	2. Memory context 복사
	3. PCB 복사 ( pid, parent, child만 변경 )
	4. 해당 PCB -> ready Q에 넣기
	그냥 그대로 가져와서 바꿔야 할 정보만 바꾸기!
	새로운 프로그램을 실행할 때 복제 방법은 적합하지 않음 
	처음에는 서로 다른 프로세스 간의 소통 창구로서 역할 수행

	loading을 한번 더함 

---
#### Process Creation
0번 프로세스도 있음 => 복제 X
init : build from scrath 
가장 먼저 생긴 프로세스는 아예 처음부터
그 다음부터는 복제해서 생성
복제 대상은 parent가 되고, 복제 결과물은 child가 된다.

Resource sharing이 가능해졌다! 

child 는 parent를 복제하기 때문에 부모의 메모리를 load한다.

parent는 fork () 를 통해 새로운 프로세스를 복제하고
child는 exec() 를 통해 loading new program을 한다!


Process Creation Example 

```C++
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
int main() {
	pid_t pid;
	/* fork a child process */
	pid = fork();
	if (pid < 0) { /* error occurred */
		fprintf(stderr, “fork failed”);
		return 1;
	}
	else if (pid == 0) { /* child process */
		execlp(“/bin/ls”, “ls”, NULL);
	}
	else { /* parent process */
		/* parent will wait for the child to complete */
		wait(NULL);
		printf(“child complete”);
	}
	return 0;
}
```


pid