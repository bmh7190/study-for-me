
조건 변수(`std::condition_variable`)를 사용하면 **조건이 만족될 때까지 잠들었다가**, 조건이 만족되면 **깨워서 순서를 제어**할 수 있다.

```c++
#include <mutex>
std::mutex mutex;
std::unique_lock<std::mutex> lock(mutex);

// wait example
while (!condition) {
	cv.wait(lock); // sleep until condition is true
}
// proceed when condition is satisfied

// notify example
{
	std::lock_guard<std::mutex> lock(mutex);
	condition = true; // update shared state
}

cv.notify_one(); // wake up one waiting thread
```

여기서 주의 깊게 볼 부분은 **`unique_lock`** 이다. `lock_guard`와 비슷하게 동작하지만, 결정적인 차이가 있다.`lock_guard`는 생성된 구역 전체에서 뮤텍스를 잠그고(`lock`) 스코프가 끝날 때 자동으로 해제한다(`unlock`). 그래서 만약 `lock_guard`로 락을 잡은 상태에서 `wait`에 들어가면,  
스레드가 잠든 동안에도 락이 계속 걸려 있기 때문에 **다른 스레드가 절대 접근할 수 없다.**

반면 `unique_lock`은 조금 더 유연하다. `wait`에 들어갈 때 잠시 **락을 해제(unlock)** 하고, 깨울 때 다시 **락을 획득(lock)** 하는 구조이기 때문에 **다른 스레드가 공유 변수에 접근하고 상태를 바꿀 수 있다.**

##### 예시로 알아보자!

이게 왜 필요한지 예시로 생각해보자.

스레드 A가 먼저 실행되었다고 하자. 조건을 확인했는데 아직 **공유 변수에 접근할 수 없는 상태**라서 `wait`에 들어갔다. 그럼 이제 A는 잠들면서 **락을 해제**한다.  

이제 스레드 B가 실행되어 같은 조건을 확인하고, 공유 변수를 바꾼 뒤 `notify_one()`을 호출하면 A를 깨울 수 있다.

만약 `lock_guard`를 썼다면 A가 잠들 때도 락을 쥐고 있어서, B는 락을 얻지 못하고 **조건을 변경하거나 notify를 호출할 수 없게 된다.** 결과적으로 프로그램이 **교착 상태(deadlock)** 에 빠지게 되는 것이다.

결론적으로 `unique_lock`은 **잠들 때(lock 해제)**, **깨서 실행 재개할 때(lock 획득)** 를 모두 지원하기 때문에 조건 변수(`condition_variable`)과 함께 쓸 때 필수적인 도구다.

즉, `unique_lock`을 사용해야 **조건 검사–대기–재시작**의 전체 과정이 올바르게 동작한다.

---
## Modeling ad Sleeping Student

![](../../images/Pasted%20image%2020251017164444.png)

아침 먹을 시간에 시간에 일어나고 싶은 상황이다.

```c++
int main() {
	std::mutex mutex;
	std::condition_variable cv;
	bool time_for_breakfast = false; // globally shared state
	
	auto student = [&]() -> void { // to be called by thread
		{ // this is the scope of the lock
			std::unique_lock<std::mutex> unique_lock(mutex);
			while (!time_for_breakfast) // check the globally shared state
				cv.wait(unique_lock); // lock is released during wait
		// alternatively, you can specify the predicate directly using a closure
		// cv.wait(unique_lock,[&](){ return time_for_break_fast; });
		
		} // lock is finally released
		std::cout << "Time to make some coffee!" << std::endl;
	};
	
	// create the waiting thread and wait for 2s
	std::thread my_thread(student);
	std::this_thread::sleep_for(2s);
	{ // prepare the alarm clock
		std::lock_guard<std::mutex> lock_guard(mutex);
		time_for_breakfast = true;
	} // here the lock is released
	
	cv.notify_one(); // ring the alarm clock
	my_thread.join(); // wait until breakfast is fini
```

thread가 `student` 함수를 실행하면 `unique_lock`으로 먼저 `mutex`를 잡는다. 그다음 조건을 확인한다. **아침 먹을 시간이 아니면**(= `time_for_breakfast == false`) `while` 안으로 들어가 `cv.wait(unique_lock)`로 **잠든다**. 이때 `unique_lock`을 넘겨주기 때문에 **wait에 들어가는 순간 lock을 풀고**, 깨울 때 **다시 lock을 잡은 뒤** `wait`가 반환된다. (스푸리어스 웨이크업 대비해서 지금처럼 `while`로 재확인하거나 `cv.wait(lock, pred)`를 쓰는 게 정석)

> 주의: `cv`는 “wait되었음을 알려주는 **상태 변수**”가 아니다.  
> **상태는 `time_for_breakfast`** 같은 공유 변수에 있고, **`cv`는 ‘상태가 바뀌었어!’라고 알리는 신호(벨)** 역할이다.

메인 스레드는 2초 기다렸다가(`sleep_for(2s)`), `lock_guard`로 `mutex`를 잠시 잡고 **`time_for_breakfast = true`로 상태를 갱신**한다. 그다음 lock을 **풀고 나서** `cv.notify_one()`으로 **대기 중인 스레드 하나를 깨운다**. (상태 갱신 → unlock → notify 순서는 깨어난 스레드가 곧바로 lock을 잡고 진행하기 좋아서 흔히 쓰는 패턴)

깨워진 `student` 스레드는 `wait`가 **lock을 다시 획득한 상태로** 반환되고, `while` 조건을 재확인해서 참이면 루프를 빠져나온다. 그 시점에 `unique_lock` 스코프가 끝나면서 lock이 풀리고, `"Time to make some coffee!"`를 출력한다. 마지막에 `join()`으로 스레드가 끝날 때까지 기다리면 깔끔하게 종료된다.


---
## Playing Ping-Pong with Condition Variables

![](../../images/Pasted%20image%2020251017164750.png)

```c++

int main() {
	std::mutex mutex;
	std::condition_variable cv;
	bool is_ping = true; // globally shared state
	
	auto ping = [&]() -> void {
		while (true) {
			std::unique_lock<std::mutex> unique_lock(mutex); // wait to be signaled
			cv.wait(unique_lock, [&](){return is_ping;});
			
			std::this_thread::sleep_for(1s); // print "ping" to the command line
			std::cout << "ping" << std::endl;
			
			is_ping = !is_ping; // alter state and notify other thread
			cv.notify_one();
		}
	};
	
	auto pong = [&]() -> void {
		while (true) {
			std::unique_lock<std::mutex> unique_lock(mutex); // wait to be signaled
			cv.wait(unique_lock, [&](){return !is_ping;});
			
			std::this_thread::sleep_for(1s); // print "pong" to the command line
			std::cout << "pong" << std::endl;
			
			is_ping = !is_ping; // alter state and notify other thread
			cv.notify_one();
		}
	};
	
	std::thread ping_thread(ping); std::thread pong_thread(pong);
	ping_thread.join(); pong_thread.join();
}
```

기존 방식에서는 이렇게 썼다.

```c++
while (!is_ping) { 	cv.wait(unique_lock); }
```

즉, 조건이 거짓이면 계속 `wait` 상태에 들어가고, 깨워져도 다시 조건을 확인해서  
거짓이면 다시 잠드는 구조였다.

하지만 이렇게 한 줄로 표현할 수도 있다.

```c++
cv.wait(unique_lock, [&](){return is_ping;});
```

`is_ping`이 `true`일 때만  `wait`을 빠져나와 이후 코드 실행한다.


### 개선 사항
상태 확인 따로 work 따로 공유 변수 즉 상태 변수 수정은 lockguard로!

```c++
while (true) {
	
	{
		std::unique_lock<std::mutex> unique_lock(mutex); // wait to be signaled
		cv.wait(unique_lock, [&](){return is_ping;});
	}
	
	std::this_thread::sleep_for(1s); // print "ping" to the command line
	std::cout << "ping" << std::endl;
	
	{
		std::lock_guard<std::mutex> lock_guard(mutex);
		is_ping = !is_ping; // alter state and notify other thread
	}
	
	cv.notify_one();

}

```