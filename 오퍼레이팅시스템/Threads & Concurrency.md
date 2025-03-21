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
