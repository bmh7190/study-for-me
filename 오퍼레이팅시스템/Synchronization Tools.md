
**프로세스는 동시(concurrently)하게 실행될 수 있다.**

프로세스는 **언제든지 중단될 수 있으며** (interrupt), **부분적으로 실행이 완료된 상태로 남을 수 있다**.

- **자원 공유** : 여러 프로세스가 **하나의 컴퓨터**에서 실행되고, **다양한 자원을 공유**할 필요가 있다.  
        
- **빠른 작업 처리** : **하나의 일을 여러 개의 서브 작업**(task)으로 나누어 **병렬 처리**함으로써 더 빠르게 작업을 완료하려는 목적이 있다.  
        
- **모듈화** : **시스템을 모듈화하여 구성**하고, 하나의 프로세스의 **출력**이 다른 프로세스의 **입력**으로 사용될 수 있다.  예를 들어, **파이프라인** 방식으로 여러 프로세스를 연결하여, 하나의 프로세스가 **처리한 데이터를 다른 프로세스가** 계속해서 **처리할 수 있도록** 하는 방식이다.


공유된 데이터에 동시 접근은 데이터 불일치를 불러올 수 있다. 

데이터 일치를 유지하기 위해서는 프로세스 협력하는 중에 순서대로 실행을 보장하는 메커니즘이 필요하다.

---
## 예시 : Consumer - Producer 문제 

**정수 카운터**는 버퍼에 채워진 항목의 수를 추적합니다.
    
초기에는 카운터가 0으로 설정됩니다.
    
생산자는 새로운 항목을 생성할 때 카운터를 증가시키고, 소비자는 항목을 소비할 때 카운터를 감소시킵니다.

```C++
while (true) {
	/* produce an item in next produced */ 

	/* do nothing */ 
	buffer[in] = next_produced; 
	in = (in + 1) % BUFFER_SIZE; 
	counter++; 
} 
```

```c++
while (counter == BUFFER_SIZE) ; 
```

만약 `counter`가 버퍼 사이즈와 동일하면 버퍼에 더 넣을 수 없으니까 자리가 날 때까지 기다려야 한다.


```c++
while (true) {
	while (counter == 0) 
	; /* do nothing */ 
	next_consumed = buffer[out]; 
	out = (out + 1) % BUFFER_SIZE; 
	counter--; 
	/* consume the item in next consumed */ 
}
```

```c
while (counter == 0) 
```

만약 버퍼에 들어온 `Consumer`가 없다면? 들어올 때까지 기다려야 한다. 


- **카운터 증가 (counter++)**는 다음과 같이 구현될 수 있다:
    
    - `register1 = counter`
        
    - `register1 = register1 + 1`
        
    - `counter = register1`
        
- **카운터 감소 (counter--)**는 다음과 같이 구현될 수 있다:
    
    - `register2 = counter`
        
    - `register2 = register2 - 1`
        
    - `counter = register2`
        

##### **실행 순서 (counter = 5에서 시작)**

1. 생산자(producer)가 `register1 = counter`를 실행 (register1 = 5)
    
2. 생산자가 `register1 = register1 + 1`을 실행 (register1 = 6)
    
3. 소비자(consumer)가 `register2 = counter`를 실행 (register2 = 5)
    
4. 소비자가 `register2 = register2 - 1`을 실행 (register2 = 4)
    
5. 생산자가 `counter = register1`을 실행 (counter = 6)
    
6. 소비자가 `counter = register2`를 실행 (counter = 4)


현재 설명에서, **생산자**가 **레지스터**에서 값을 증가시키고, **`counter` 변수에 최종 값을 반영**하는 방식으로 진행되고 있다. 그런데 이 과정에서 **레지스터만 증가**시키고 `counter`에 값을 반영하기 전에 **소비자**가 `counter`를 건드리면 **이상한 값**이 들어가게 된다.


---
## **Critical Section Problem**

