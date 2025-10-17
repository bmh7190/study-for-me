
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

##### 예시로 알아보자

이게 왜 필요한지 예시로 생각해보자.

스레드 A가 먼저 실행되었다고 하자. 조건을 확인했는데 아직 **공유 변수에 접근할 수 없는 상태**라서 `wait`에 들어갔다. 그럼 이제 A는 잠들면서 **락을 해제**한다.  

이제 스레드 B가 실행되어 같은 조건을 확인하고, 공유 변수를 바꾼 뒤 `notify_one()`을 호출하면 A를 깨울 수 있다.

만약 `lock_guard`를 썼다면 A가 잠들 때도 락을 쥐고 있어서, B는 락을 얻지 못하고 **조건을 변경하거나 notify를 호출할 수 없게 된다.** 결과적으로 프로그램이 **교착 상태(deadlock)** 에 빠지게 되는 것이다.

결론적으로 `unique_lock`은 **잠들 때(lock 해제)**, **깨서 실행 재개할 때(lock 획득)** 를 모두 지원하기 때문에 조건 변수(`condition_variable`)과 함께 쓸 때 필수적인 도구다.

즉, `unique_lock`을 사용해야 **조건 검사–대기–재시작**의 전체 과정이 올바르게 동작한다.

---
## Modeling ad Sleeping Student

![](../images/Pasted%20image%2020251017164444.png)

아침 먹을 시간에 일어나고 싶어 하는 상황이다.

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


---
## Playing Ping-Pong with Condition Variables

![](../images/Pasted%20image%2020251017164750.png)

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