---

---

---

#### Computer Hardware
CPU와 디바이스 컨트롤러는 **시스템 버스(System Bus)** 를 통해 연결되며, 시스템 버스는 컴퓨터의 주요 구성 요소 간 데이터 및 명령어 전송을 담당한다.  

시스템 버스는 **데이터 버스(Data Bus), 주소 버스(Address Bus), 제어 버스(Control Bus)** 로 나뉘며,

- **데이터 버스**는 데이터를 주고받고,
- **주소 버스**는 메모리 및 I/O 장치의 주소를 지정하며,
- **제어 버스**는 읽기(Read) 및 쓰기(Write) 등의 제어 신호를 전달한다.

Bus Arbiter(버스 중재기)
여러 개의 버스 요청이 있을 때, **충돌을 조정하고 우선순위를 결정**하는 역할을 한다.

Bus Master(버스 마스터) 
버스 요청 신호를 보내고, 버스 트랜잭션을 시작할 수 있는 장치**이다. (예: CPU, DMA 컨트롤러)

Bus Slave(버스 슬레이브) 
버스 마스터의 명령을 받아 해당 요청을 수행하는 장치이다. (예: 메모리, I/O 장치)

---
#### Interrupts Mechanism

#### interrupt 란?
하드웨어가 **Interrupt service routine (ISR)** 으로 제어를 넘기는 메커니즘으로,  **Interrupt Vector**를 통해 이루어진다.  

Interrupt Vector는 **모든 서비스 루틴의 주소를 저장한 테이블**로, 특정 Interrupt가 발생하면 해당 **서비스 루틴의 주소를 찾아 실행**하는 역할을 한다.

즉, Interrupt가 발생하면 CPU는 **Interrupt Vector Table** 을 참조하여 적절한 서비스 루틴을 실행하는 방식이다.

#### Trap or exception
소프트웨어에 의해 생성되는 **인터럽트(Interrupt)** 로, **오류(Error) 또는 사용자 요청(User Request)에 의해 발생**한다.

- **오류(Error)에 의한 트랩**: 0으로 나누기(Divide by Zero), 잘못된 메모리 접근(Segmentation Fault) 등
- **사용자 요청(User Request)에 의한 트랩**: 시스템 호출(System Call), 디버깅(Debugging Breakpoint) 등

즉, **트랩(Trap)과 예외(Exception)은 소프트웨어적으로 발생하는 인터럽트**이며, CPU는 이를 처리하기 위해 **인터럽트 핸들러(Interrupt Handler) 또는 예외 핸들러(Exception Handler)를 호출**한다.

---
#### Interrupt Handling
운영체제(OS)는 **컨텍스트 스위칭(Context Switching)** 이나 **인터럽트 처리** 시, **CPU의 상태와 프로그램 카운터(Program Counter, PC)를 저장**해야 한다. 이를 위해 **레지스터를 저장하는 방식**을 사용한다.

1. **현재 실행 중인 프로그램의 수행을 멈춘다.**
    - 현재 실행 중인 프로세스나 명령어가 인터럽트에 의해 일시적으로 중단됨.
    
2. **중지된 기능(프로그램)의 주소(현재 PC, 레지스터 값)를 저장한다.**
    - CPU는 **프로그램 카운터(PC)** 와 레지스터 상태를 **스택(Stack) 또는 PCB(Process Control Block)** 에 저장하여 이후 복원할 수 있도록 함.
    
3. **인터럽트 벡터 테이블(Interrupt Vector Table, IVT)을 사용하여 ISR(Interrupt Service Routine)의 주소를 가져온다.**
    - 인터럽트 요청(Interrupt Request Number, IRQ)에 해당하는 ISR의 주소를 찾음.
    - IVT에는 다양한 인터럽트 요청에 대한 **ISR의 주소 목록**이 저장되어 있음.
    
4. **ISR로 점프하여(이동하여) 실행한다.**
    - CPU가 **해당 ISR(인터럽트 서비스 루틴)으로 점프(jump)하여 인터럽트를 처리**함.

이후 ISR 실행이 끝나면, CPU는 저장된 **PC 및 레지스터 상태를 복원**한 후, 원래 수행 중이던 프로그램으로 돌아간다.

**인터럽트 처리 중 새로운 인터럽트가 발생하면, 현재 인터럽트가 정상적으로 처리되지 않을 수 있다.** **인터럽트가 중첩되면(CPU가 새로운 ISR로 계속 점프하면) 원래의 인터럽트 처리가 완료되지 않아 "Lost Interrupt(손실된 인터럽트)" 문제가 발생할 수 있다.** 이를 방지하기 위해, CPU는 **현재 인터럽트 처리 중에는 추가적인 인터럽트를 일시적으로 비활성화**한다.


![[Pasted image 20250312214906.png]]

- **I/O Request 발생**
	사용자 프로세스가 실행 중인 동안 **I/O 요청**이 발생한다. 이때, CPU는 계속 실행되지만, I/O 디바이스는 요청을 받고 데이터 전송을 시작한다.
    
- **I/O Interrupt 발생 및 ISR 실행**
	I/O 디바이스가 데이터 전송을 마치면 **인터럽트를 발생시켜 CPU에 알린다.** CPU는 현재 실행 중인 사용자 프로세스를 잠시 중단하고, **인터럽트 서비스 루틴(ISR, Interrupt Service Routine)** 을 실행한다.
    
- **ISR이 끝난 후 원래 작업 복귀**
    - 인터럽트 처리가 완료되면, CPU는 원래 실행 중이던 사용자 프로세스로 돌아간다.


여기서  ISR의 역할은 **I/O 완료 처리를 수행하고, 필요하면 추가적인 작업을 요청하는 것**이라고 할 수 있지!

---
#### OS Service
services provide fucntions that are helpful to the user
1. User interface 
2. Program execution
3. I/O operation
4. File-system manipulation
5. Communications
6. Error detection

other services exist for ensuring the efficient operation of the system itselt via resource sharing

1. Resource allocation
2. Accounting
3. Protection and security


---
#### System Calls
운영체제가 제공하는 서비스에 접근하는 프로그래밍 인터페이스는 주로 **고수준 API(High-Level API)를 통해 사용되며**, 시스템 호출(System Call)을 직접 사용하는 경우는 많지 않다.

대체적으로 각 **시스템 호출(System Call)** 에는 고유한 번호가 부여되며, **시스템 콜 인터페이스(System Call Interface)** 는 이러한 번호를 매핑한 테이블로 구성된다.

시스템 콜 인터페이스는 운영체제(OS) 커널에서 해당 시스템 호출을 실행하고, 그 결과와 상태 정보를 반환한다.

시스템 호출이 내부적으로 어떻게 동작하는지는 호출하는 프로그램(caller)이 알 필요가 없으며, 단순히 **API를 따르고, 운영체제가 시스템 호출을 통해 어떤 작업을 수행하는지만 이해하면 된다.**


---
#### User mode VS Kernel mode
운영체제는 하드웨어의 지원을 받아 최소한 두 가지 운영 모드를 구별하여 **시스템 보호 및 안정성을 보장**합니다.

1. User mode
	1. User에 의해 실행된다. 
2. Kernel mode
	1. OS에 의해 수행된다.
	2. kerne mode에서 수행하고 있을 때, OS는 kernel과 user의 메모리에 대한 접근의 제한을 푼다.

컴퓨터 하드웨어에는 **모드 비트(Mode Bit)** 가 추가되며, 특히 **프로세서 상태(Processor Status)** 에 포함된다.