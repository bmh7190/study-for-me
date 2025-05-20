
**모니터(Monitor)는 동기화를 위한 편리하고 효율적인 고수준 추상화(high-level abstraction) 메커니즘이다.**

모니터는 **추상 데이터 타입(ADT)** 처럼, 내부 변수들은 오직 **모니터 내부에 정의된 procedure**를 통해서만 접근 가능하고 외부에서는 직접 접근할 수 없다.

모든 동기화 문제를 전부 해결할 수 없다.

**공유된 데이터 구조**, **공유 데이터를 다루는 procedure**, **이러한 procedure를 동시에 호출하는 concurrent thread들 간의 동기화**를 캡슐화하여 동기화된 접근을 보장한다. 
    
```c
monitor monitor-name {
	// shared variable declarations
	…
	procedure P1 (…) { … }
	…
	procedure Pn (…) { … }
	Initialization code (…) { … }
}
```

---

## **Schematic view of a monitor**

![](../images/Pasted%20image%2020250520224413.png)

지금 위의 그림은 모니터를 나타낸 것이다. 모니터 안에는 하나의 프로세스, 하나의 쓰레드만 들어갈 수 있기 때문에, 동시에 요청할 경우 이미 들어간 프로세스가 끝날 때까지 기다려야 한다. 기다리는 곳을 entry queue라고 보면 된다. 또 initialization code를 통해 shared data 초기화를 한다.

순서 동기화를 위한 condtion value는 이 그림에는 안 나와있다. 

---

모니터 안에서 condition varaibles ( waiting queues )

codtiton x, y; 
x 라는 이름의 큐를 만든다. 
x.wait() 은 x.signal()이라는 함수가 호출될 때까지 x 라는 큐에서 기다리겠다. 
x.siganl() 은 x 큐에서 하나 꺼내서 실행시키겠다!
