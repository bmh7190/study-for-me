
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

![[Pasted image 20250319144624.png]]

✅ **0번 프로세스는 복제되지 않음**

- 최초의 프로세스는 **복제(Fork)로 생성되지 않고, 처음부터(build from scratch) 생성됨.**
- 대표적으로 `init` 프로세스가 여기에 해당.

✅ **가장 먼저 생긴 프로세스(init)는 처음부터 생성되고, 이후 프로세스는 복제를 통해 생성됨**

- `init` 프로세스 이후부터는 **부모 프로세스(Parent)를 복제하여 자식 프로세스(Child)를 생성**.
- 복제된 프로세스는 **부모의 메모리를 로드**하여 동일한 상태를 가짐.
- 부모는 새로운 프로세스를 생성하는 역할을 수행.

✅ **리소스 공유 가능 (Resource Sharing Enabled)**

- 부모와 자식 프로세스는 **일부 리소스를 공유**할 수 있음.
- 예: 파일 디스크립터, 메모리 공간, 환경 변수 등.

✅ **프로세스 생성 메커니즘**

1. **부모 프로세스(Parent Process) → `fork()` 호출**
    
    - `fork()`를 호출하면 **부모 프로세스의 복사본(Child Process)이 생성됨.**
    - 자식 프로세스는 부모와 **거의 동일한 상태**로 시작됨.
2. **자식 프로세스(Child Process) → `exec()` 호출**
    
    - `exec()` 를 호출하면 **새로운 프로그램이 메모리에 로드됨.**
    - 즉, `exec()` 를 실행한 이후에는 **기존 부모의 코드가 아닌 새로운 프로그램이 실행됨.**


#### Process Creation Example 

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


---
### Process Termination

✅ **프로세스가 종료될 때 수행되는 작업**

1. 프로세스는 **마지막 명령을 실행한 후**, `exit()` 시스템 호출을 통해 운영체제에 **삭제 요청**을 보냄.
2. 자식 프로세스(Child)는 **`wait()` 시스템 호출을 통해 부모에게 종료 상태(Status Data)를 반환**.
3. 운영체제는 **프로세스가 사용하던 자원(메모리, 파일 디스크립터 등)을 해제(Deallocate)**.

✅ **부모 프로세스가 자식 프로세스를 강제 종료할 수 있음**

- 부모 프로세스는 필요에 따라 자식 프로세스를 종료할 수 있음.
- **Linux**: `kill()`
- **Windows**: `TerminateProcess()`

✅ **부모가 자식 프로세스를 종료하는 이유**

1. **자식 프로세스가 할당된 자원을 초과하여 사용함** (예: 메모리, CPU 시간 초과).
2. **자식 프로세스가 수행해야 할 작업이 더 이상 필요하지 않음**.
3. **부모 프로세스가 종료될 경우, 일부 운영체제는 자식 프로세스를 계속 실행하는 것을 허용하지 않음**.

---


✅ **일부 운영체제에서는 부모 프로세스가 종료되지 않는 한 자식 프로세스도 존재할 수 없음.**

- 만약 부모 프로세스가 종료되면, **그에 속한 모든 자식 프로세스도 함께 종료되어야 함.**
- 이를 **"Cascading Termination(연쇄 종료)"** 라고 하며, **부모뿐만 아니라 모든 자식 프로세스와 그들의 하위 프로세스도 제거됨.**
- 이 종료는 **운영체제(OS)에 의해 자동으로 수행됨.**

✅ **부모가 `wait()` 시스템 콜을 사용하여 자식 프로세스의 종료를 기다릴 수 있음.**

- 부모가 `wait()` 호출 시, 자식 프로세스가 종료될 때까지 기다림.
- 종료된 자식 프로세스의 **PID(Process ID)와 상태 정보를 반환**받음.

```c
pid = wait(&status);
```

✅ **좀비 프로세스(Zombie Process)**

- **부모가 `wait()`을 호출하지 않으면, 자식 프로세스는 좀비 상태가 됨.**
- 좀비 프로세스는 **실행되지 않지만, 프로세스 테이블에서 정보가 남아있는 상태**로 자원을 차지함.

✅ **고아 프로세스(Orphan Process)**

- **부모가 `wait()`을 호출하지 않고 먼저 종료되면, 자식 프로세스는 고아 프로세스가 됨.**
- 대부분의 운영체제에서는 **고아 프로세스를 `init` 프로세스가 자동으로 관리하여 제거**함.


---
### Multiprogram Architecture Example (Chrome)

✅ **일반적인 웹 브라우저**

- 대부분의 웹 브라우저는 **싱글 프로세스(Single Process) 모델**을 사용하여 동작.
- 이 경우, **하나의 웹사이트에서 문제가 발생하면 전체 브라우저가 멈추거나 충돌(Hang or Crash)할 가능성이 높음.**

✅ **구글 크롬의 멀티 프로세스 모델**  
구글 크롬은 **3가지 주요 프로세스**로 동작하여 안정성과 보안을 강화함.

1. **브라우저 프로세스(Browser Process)**
    
    - **사용자 인터페이스(UI), 디스크 I/O, 네트워크 I/O** 등을 관리함.
    - 크롬의 전체 프레임워크를 제어하는 **중앙 프로세스** 역할.
2. **렌더러 프로세스(Renderer Process)**
    
    - 웹 페이지를 **렌더링(HTML, CSS, JavaScript 처리)** 하는 역할.
    - **각 웹사이트(탭)마다 새로운 렌더러 프로세스가 생성**됨.
    - 웹사이트 충돌이 발생해도 **다른 웹사이트(탭)에는 영향을 주지 않도록 격리됨 (Sandboxing).**
3. **플러그인 프로세스(Plug-in Process)**
    
    - 플러그인(예: Flash, PDF 뷰어 등)이 **독립적인 프로세스로 실행**됨.
    - 특정 플러그인이 충돌해도 브라우저 전체에는 영향을 미치지 않음.

---
### Inter - process Communication

✅ **시스템 내의 프로세스는 두 가지 유형으로 분류될 수 있음**

1. **독립적인 프로세스 (Independent Process)**
    
    - 다른 프로세스에 영향을 주거나 받지 않음.
    - 예: 개별 실행 중인 텍스트 에디터, 미디어 플레이어 등.
2. **협력하는 프로세스 (Cooperating Process)**
    
    - **공유 데이터**를 포함하여 다른 프로세스와 상호작용할 수 있음.
    - 예: 웹 브라우저(렌더러 & 네트워크 프로세스), 클라이언트-서버 모델 등.

✅ **협력하는 프로세스가 필요한 이유 (Reasons for Cooperating Processes)**

1. **정보 공유 (Information Sharing)**
    
    - 여러 프로세스가 동일한 데이터를 사용해야 할 경우 필요.
    - 예: 데이터베이스, 파일 공유 시스템.
2. **연산 속도 향상 (Computation Speedup)**
    
    - 작업을 여러 프로세스로 나누어 병렬 처리하면 속도가 향상됨.
    - 예: 멀티스레딩, 병렬 연산.
3. **모듈화 (Modularity)**
    
    - 프로그램을 여러 개의 독립적인 프로세스로 분리하여 관리가 용이함.
    - 예: 운영체제의 커널 모듈, 웹 브라우저의 멀티 프로세스 아키텍처.
4. **편의성 (Convenience)**
    
    - 특정 작업을 백그라운드에서 실행하여 사용자 경험을 향상.
    - 예: 오디오/비디오 스트리밍, 프린터 스풀링.

✅ **프로세스 간 통신 (Inter-Process Communication, IPC)**  
협력하는 프로세스는 **프로세스 간 통신(IPC)** 이 필요함.  
대표적인 두 가지 방식:

1. **공유 메모리 (Shared Memory)**
    
    - 프로세스들이 같은 메모리 공간을 공유하여 데이터를 주고받음.
    - 빠른 속도지만 동기화(예: 뮤텍스, 세마포어) 필요.
2. **메시지 전달 (Message Passing)**
    
    - 프로세스 간 직접 데이터를 공유하지 않고, **메시지를 교환하여 통신**.
    - 운영체제 커널이 메시지를 중재하여 데이터 동기화 문제를 줄임.

![[Pasted image 20250319150329.png]]
##### **(a) 메시지 전달(Message Passing) 방식**

- `msgsnd()`, `msgrcv()` 와 같은 시스템 호출을 사용하여 프로세스 간 메시지를 주고받음.
- **데이터를 복사하여 전달하므로** 대용량 데이터를 주고받을 때 **비효율적**일 수 있음.
- 하지만 **운영체제가 메시지 전달을 관리하므로 동기화 문제가 발생하지 않음**.

##### **(b) 공유 메모리(Shared Memory) 방식**

- 공유 메모리 영역을 생성하고, **프로세스들이 동일한 메모리 공간을 참조**하여 데이터를 주고받음.
- 데이터를 **복사하지 않고 직접 접근**하므로, 대용량 데이터 전송에 유리함.
- 하지만 **여러 프로세스가 동시에 접근할 경우 동기화 관리가 필요함** (예: Mutex, Semaphore 사용).

|IPC 방식|데이터 전달 방식|속도|동기화 필요 여부|대용량 데이터 처리|
|---|---|---|---|---|
|**메시지 전달**|데이터 복사|느림|❌ 필요 없음|❌ 비효율적|
|**공유 메모리**|포인터 제공|빠름|✅ 필요함|✅ 효율적|

---
### 