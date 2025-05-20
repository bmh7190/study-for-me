
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

지금 위의 그림은 **모니터(Monitor)** 의 구조를 나타낸 것이다.  모니터 내부에는 **한 번에 하나의 프로세스 또는 쓰레드만 진입할 수 있기 때문에**,  다른 프로세스가 동시에 접근을 시도할 경우, **먼저 들어간 프로세스가 작업을 마칠 때까지 기다려야 한다.**  이렇게 **진입을 대기하는 프로세스들이 줄 서 있는 곳을 `entry queue`라고 한다.**

또한, **모니터의 `initialization code`는 공유 데이터(shared data)를 초기화하는 역할**을 한다.



---

**모니터 내부에서 조건 변수(`condition variable`)는 순서 동기화를 위해 사용되며, 각각의 조건 변수는 하나의 대기 큐(waiting queue)를 가진다.**

예를 들어 다음과 같이 선언할 수 있다.

```c
codition x, y;
```

위 선언은 **`x`와 `y`라는 이름의 조건 변수(= 대기 큐)를 생성**하는 것이다.

- `x.wait()`  
    → 현재 실행 중인 프로세스(또는 쓰레드)는 **`x` 큐에 들어가 대기하게 된다.**  
    이 상태는 **다른 프로세스가 `x.signal()`을 호출할 때까지 계속된다.**
    
- `x.signal()`  
    → **`x` 큐에서 대기 중인 프로세스 중 하나를 꺼내 실행 상태로 만든다.**  
    즉, **대기 중인 프로세스를 깨워(monitor 내부로 복귀시켜)** 실행을 이어가게 한다.


![](../images/Pasted%20image%2020250520225021.png)

x, y라는 큐 자체도 공유변수기 대문에 한 명만 접근해야 한다. 따라서 모니터 내부에 존재하게 된다. 

일반적인 경우 
x.wait(숫자) 에서 숫자는 timeout을 의미한다. 예를 들어서, 숫자가 100이라면 100이라는 시간 동안 queue 안에서 대기하고, 100이 넘으면 알아서 해제하게 된다. 

---
## **Code Example**

```c
monitor ResourceAllocator {
	boolean busy;
	condition x;
	
	void acquire(int time) {
		if (busy)
			x.wait(time);
			busy = true;
	}
	
	void release() {
		busy = false;
		x.signal();
	}
	
	initialization_code() {
		busy = false;
	}
}
```

여기서 `busy` 공유 변수, 자원이 사용되고 있는지 확인하는 용도이다. 

```c
void acquire(int time) {
	if (busy)
		x.wait(time);
		busy = true;
}
```

time 만큼 자원을 할당 받아서 사용하는 함수인데, 지금 모니터 안에 있기 때문에 하나의 프로세스 밖에 실행할 수 밖에 없다. 만약 자원을 사용하고 있다면, `x.wait(time)`을 호출하여 x큐에 넣고, 누군가 `signal()`을 호출하면 깨워서 busy를 true로 바꿔서 내가 쓰고 있다는 것을 표시!

```c
void release() {
	busy = false;
	x.signal();
}
```

자원 해제를 위한 함수므로, busy를 false로 바꿔 내가 다 사용했다는 것을 알리고, x에 대기하고 있는, 프로세스를 깨워서 실행시킨다. 

여기서 busy 공유 변수가 따로 critical section problem을 해결 안 해도 되는 이유는 어차피 모니터 안에는 하나의 쓰레드