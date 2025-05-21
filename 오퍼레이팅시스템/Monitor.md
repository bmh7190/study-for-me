
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

`busy`는 **자원이 현재 사용 중인지 여부를 나타내는 공유 변수**이다.
`x`는 **조건 변수(condition variable)** 로, 자원을 기다리는 프로세스들을 관리하는 **대기 큐** 역할을 한다.

```c
void acquire(int time) {
	if (busy)
		x.wait(time);
		busy = true;
}
```

이 함수는 **자원을 할당받기 위한 함수**로, 현재 자원이 사용 중(`busy == true`)이라면, `x.wait(time)`을 호출하여 **해당 프로세스를 `x` 큐에 넣고 대기**시킨다. 그렇지 않다면 즉시 `busy = true`로 설정하여 **자원을 점유**한다.

> ⚠ **주의**: 이 함수는 모니터 내부에 있기 때문에 **동시에 하나의 프로세스만 실행 가능하다.**  
> 따라서 `busy` 변수에 대한 접근은 **자동적으로 상호 배제(mutual exclusion)** 가 보장된다.  
> 즉, 별도로 임계 구역(critical section)을 지정하거나 락(lock)을 사용할 필요가 없다.

```c
void release() {
	busy = false;
	x.signal();
}
```

이 함수는 **자원을 해제하는 함수**로, `busy = false`로 설정하여 **자원이 사용 중이 아님을 표시**한다.`x.signal()`을 호출하여 **`x` 큐에서 대기 중인 프로세스 중 하나를 깨운다.**  깨워진 프로세스는 다시 모니터에 진입하여 자원을 사용할 수 있게 된다.

---
## **Condition vairable choices**

만약 Q가 `x.wait()` 상태에 있고, P가 `x.signal()`을 호출하면 어떤 일이 발생할까?

Q와 P는 동시에 병렬적으로 실행될 수 없다. 만약 Q가 깨어나 실행된다면, **P는 반드시 대기해야 한다.**  
이는 모니터 내부에는 **오직 하나의 프로세스만 들어올 수 있기 때문**이다.

사실 `wait()` 상태가 된다는 것은, **해당 프로세스를 condition queue, 즉 대기 큐에 넣고 모니터 밖에서 기다리게 한다는 의미**이다.  만약 `wait()` 상태를 모니터 내부에서 유지한다면, **모니터에 하나의 쓰레드만 들어올 수 있다는 제약** 때문에,  다른 어떤 쓰레드도 모니터에 진입할 수 없게 되어버린다. 결과적으로 **모든 프로세스가 교착상태에 빠지게 된다.**


#### Signal and Wait

P가 Q를 깨웠을 때, **P는 잠시 대기 상태로 전환되고 Q가 모니터에 들어가 실행**된다.  Q의 실행이 끝난 후에야 **P가 다시 모니터에 들어가 실행을 이어간다.**  또한, 만약 Q가 실행 도중 다시 `wait()`을 호출하게 된다면,  P는 계속 대기 중이므로, 이를 위해 **잠시 대기할 별도의 큐(Urgent Queue)** 가 필요하다.

#### Signal and Continue

P가 Q를 깨운 후, **P가 계속 실행을 마친다.**  Q는 **P가 모니터를 나간 뒤에야 들어와서 실행**된다.  
혹은, P가 실행 도중 다시 `wait()`을 호출하게 되면,  그 시점에 Q가 모니터에 들어와 실행을 이어가게 된다.

---
## **Monitor Implemetation( Signal and Wait )**

```c
semaphore monitor_lock; // initially = 1
semaphore sig_lock; // initially = 0
int sig_lock_count = 0;
```

모니터 바깥에는 두 개의 큐가 존재한다.

- **entry queue**: `monitor_lock`을 얻지 못한 일반 프로세스들이 대기
- **signaler queue**: `signal-and-wait` 방식에서, 신호를 보내고 잠시 대기 중인 프로세스들이 대기

`monitor_lock`과 `sig_lock`은 각각의 큐 접근과 실행 제어를 위한 **세마포어**이며,  `sig_lock_count`는 현재 **signaler queue에서 대기 중인 프로세스의 수**를 나타낸다.

```c
wait(monitor_lock);
	…
	body of F;
	…
if (sig_lock_count > 0)
	signal(sig_lock);
else
	signal(monitor_lock);
```

###### **`wait(monitor_lock)`**
세마포어 기반의 진입 제어로 키를 얻은 프로세스만 모니터에 진입 가능하며,  **못 얻은 프로세스는 entry queue에서 대기**한다.
###### **함수 `F` 본체 실행**
실제 공유 데이터를 다루는 로직이 이 위치에 들어간다. 언어 수준에서 모니터를 지원하는 경우, 컴파일러가 이 진입(Entry Section)과 퇴장(Exit Section) 코드를 자동 삽입해준다.
###### **모니터 퇴장 시점**
`signal()` 호출 여부에 따라,  다음 실행 주체를 **signaler queue > entry queue** 우선순위로 결정한다.

**`sig_lock_count > 0`** 이면 signaler queue에 대기 중인 프로세스가 있다는 뜻이므로,  `sig_lock`에 signal을 보내어 **그 프로세스를 깨운다.** 그렇지 않으면, `monitor_lock`을 signal하여 **entry queue에 있는 프로세스를 진입시킨다.**

> [!note] 왜 signaler queue에 우선순위를 주는가?
> 
> `signaler queue`에 있는 프로세스는 이미 모니터 내부에서 `signal()`을 호출하고  **잠시 나가 있던 중인 프로세스**이다.  즉, **작업을 완료하지 못한 상태**에서 기다리고 있으므로  `entry queue`에서 처음 진입하려는 프로세스보다 우선권을 주는 것이 올바른 동작이다.

---
 
 
```c
 int x count; / / x에서 기다리는 프로세스 수
 semaphore x_sem; // x에 대한 semaphore
```

```c
/* x.wait */
x_count++;
if (sig_lock_count > 0)
	signal(sig_lock);
else
	signal(monitor_lock);
wait(x_sem);
x_count--;
```

x.wait() 은 모니터 안에 들어와 있는 상태에서 실행, 

x_count 를 통해서 잘거라고 알리고, 누군가 깨운다음, wait(x_sem)을 해서 잠든다? semaphore 1 증가 해주고, signal(x_sem)을 해주면 1 해주면서 깨어나겠지
