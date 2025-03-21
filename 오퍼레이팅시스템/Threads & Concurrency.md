---

---

---
# Motivation

![[Pasted image 20250321143558.png]]

request 가 도착하면 main thread가 감지하다 child thread를 만든다!

process의 메모리를 포함한 자원들은 공유하면서 execution stream만 여러 개로 복제 한다.  

---
# Benefits of Multihreaded Process

process 가 막혔을 때, 다른 흐름을 돌게 할 수 있다.

멀티프로세스의 경우 새로운 프로세스를 만들 때 자원 비용이 크고, 프로세스 간 소통을 하는데 비용이 든다. 하지만 thread 같은경우에는 자원을 공유하기 때문에 , 소통 필요하지 않음
프로세스를 만드는거보다 thread 만드는게 더 저렴
thread 제거하는데도 빠름
같은 프로세스 안에서 두 개의 쓰레드 사이의 switch 가 쉽다
더 적은 자원을 사용한다. 

---
# Multicore Programming
mulicore  또 multiprocessor system의 경우 다음과 같은 문제점을 가지고 있다.
- Dividing activites
- Balance
- Data splitting
- Data dependency
- Tesing and debugging

병렬성(Parallelism) 은 시스템이 아나 이상의 일을 동시적으로 수행할 수 있도록 해준다!

Concurrency는 프로세스 만드는 하나 이상의 job을 도와줄 수 있다! 비슷한 개념일 수 있지만 병렬성과 다르게 하나의 프로세스에서 이뤄진다는 점!

- Concurrency

	![[Pasted image 20250321165752.png]]

- Parallelism

	![[Pasted image 20250321165838.png]]

---
# Parallelism

병렬성은 크게 **Data Parallelism**과 **Task Parallelism**으로 나뉜다.

- **Data Parallelism**:  
    여러 개의 코어에서 **같은 작업을 서로 다른 데이터 조각에 대해 동시에 수행**하는 방식이다.  
    즉, 데이터를 나누어 각 코어에 분산시키고 동일한 연산을 병렬로 실행한다.
    
- **Task Parallelism**:  
    **서로 다른 작업을 각 스레드에 나누어 코어에 분산**시키는 방식이다.  
    각 스레드는 고유한 작업을 수행하며 병렬로 실행된다.
    

---
# Traditional Process Model
### ✅ 하나의 프로세스가 가지는 두 가지 주요 특성

1. **리소스 소유의 단위 (Unit of Resource Ownership)**
    
    - 프로세스는 고유한 **가상 주소 공간(Virtual Address Space)** 을 할당받아, 자신의 **프로세스 이미지**(코드, 데이터, 스택 등)를 저장함.
    - 파일, I/O 장치 등 **일정한 자원에 대한 제어권(Control of resources)** 을 가진다.
    
2. **실행 흐름의 단위 (Unit of Execution Stream)**
    
    - **하나의 스레드(Thread of Control)** 를 가지고 실행된다.
    - **실행 상태(Execution State)** 와 **디스패치 우선순위(Dispatching Priority)** 를 가진다.
    - 여러 프로세스의 실행은 **인터리브(interleave)** 되어 실행될 수 있다.
    - **멀티코어 또는 멀티프로세서 시스템**에서는 **진정한 병렬성(Parallelism)** 이 가능하며, 이는 단순한 동시성(Concurrency)과는 다르다.


---
# Multithreaded Process

위의 두 특징을 분리해보면 어떨까?
대부분 현대의 OS 들은 이 두 가지 특징을 독립적으로 다룬다. 

리소스 소유는 보통 process 또는 task로 불리고, dispatching은 보통 thread 혹은 lightwegiht process 라고 불린다. 

![[Pasted image 20250321172725.png]]

## Characteristics of threads

실행 상태를 가지고 있다. running, ready, stopped 
thread context를 동작하지 않을 때 저장한다. 