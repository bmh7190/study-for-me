## Spawning and Joining Threads

스레드는 하나의 프로세스 안에서 동시에 여러 작업을 수행하기 위해 만들어진 실행 단위다. 프로그램이 실행되면 운영체제는 기본적으로 하나의 **메인 스레드(또는 마스터 스레드)** 를 생성한다. 우리가 `std::thread` 같은 방법으로 만드는 모든 스레드는 이 메인 스레드로부터 파생된 **자식 스레드**다.

같은 프로세스에 속한 모든 스레드는 **공유 자원**을 함께 사용한다. 이들은 코드 영역, 힙 영역, 전역 변수 공간 등을 공유하기 때문에 같은 메모리 공간에 접근할 수 있다. 이런 구조 덕분에 스레드 간 통신은 별도의 IPC(프로세스 간 통신) 없이 빠르게 이루어질 수 있으며, 낮은 지연 시간(latency)을 가진다는 장점이 있다. 

하지만 동시에 여러 스레드가 동일한 자원에 접근하면 **데이터 경쟁(data race)** 이 발생할 수 있으므로, 반드시 **뮤텍스(mutex)** 나 **조건 변수(condition variable)** 같은 동기화 도구를 사용해야 한다. 참고로 스택 영역은 스레드마다 독립적으로 할당되어 각 스레드의 지역 변수들이 서로 간섭하지 않는다.

![](Pasted%20image%2020251017141305.png)

스레드를 생성하면 메인 스레드와는 독립적으로 실행되지만, 메인 스레드가 그 스레드의 작업이 끝날 때까지 기다리도록 할 수도 있다. 이때 사용하는 함수가 `join()`이다. `join()`을 호출하면 메인 스레드는 해당 스레드가 종료될 때까지 블록 상태로 기다린다. 반대로 `detach()`를 호출하면 해당 스레드는 독립적으로 실행되며, 메인 스레드가 기다리지 않는다. 이 경우 스레드가 끝나면 운영체제가 자동으로 자원을 정리(clean up)한다.

하지만 주의할 점이 있다. 메인 스레드가 종료되면 그 프로세스 자체가 종료되기 때문에, 그 안에서 동작 중이던 모든 스레드도 함께 종료된다. 따라서 `detach()`된 스레드가 아직 작업을 마치지 않았더라도 프로세스가 끝나면 강제적으로 중단된다. 일반적으로는 이런 상황을 방지하기 위해 `join()`으로 모든 스레드의 종료를 기다리거나, 별도의 종료 신호를 통해 안전하게 작업을 마무리하도록 설계한다.

---
## Multithreaded C++11 

```c++
#include <cstdint.h> // uint64_t
#include <vector> // std::vector
#include <thread> // std::thread
// this function will be called by the threads ( should be void )
void say_hello(uint64_t id) {
	std::cout << "Hello from thread: " << id << std::endl;
}

int main(int argc, char* argv[]) { // this runs in the master thread
	const uint64_t num_threads = 4;
	std::vector<std::thread> threads;
	
	for (uint64_t id = 0; id<num_threads; id++) // for all threads
		threads.emplace_back(say_hello, id); // call say_hello with arg. id
	
	for (auto& thread : threads) //join each thread at the end
		thread.join();
}
```

- **쓰레드가 실행할 함수의 리턴 타입은 반드시 `void`** 여야 한다.  
    쓰레드 함수는 독립적으로 실행되며, 반환값을 직접 받을 수 없기 때문이다.
    
- **C++11부터 스레드는 객체로 관리된다.**  
    즉, `std::thread`는 하나의 실행 단위를 나타내는 **객체**이며, 이 객체를 통해 생성, 실행, 종료 대기(`join`) 등을 제어할 수 있다.

```c++
std::vector<std::thread> threads;
```

`std::thread` 타입의 객체를 담을 수 있는 `std::vector`를 만들어 여러 개의 스레드를 한꺼번에 관리할 수 있다.

```c
threads.emplace_back(say_hello, id);
```

위 코드는 `say_hello(id)`를 실행할 새로운 스레드를 생성하고, 그 스레드를 벡터에 추가한다.

마지막으로 메인 스레드는 모든 자식 스레드가 일을 마칠 때까지 `join()`을 호출하여 기다린다. 
이렇게 해야 프로그램이 메인 스레드 종료와 함께 강제 종료되지 않고, 모든 스레드가 안전하게 완료될 수 있다.

지금처럼 `std::vector`를 이용해 동적으로 관리하는 방법 외에도, 단순히 배열 형태로 스레드를 만들 수도 있다:

```c++
std::thread* threads = new std::thread[num_threads]; threads[i] = std::thread(say_hello, id);
```

이 방법은 동적 할당을 사용하여 스레드 배열을 만드는 방식이다.  
다만 이 경우에는 `delete[] threads;`로 메모리를 직접 해제해야 하며,  
예외 안전성이 떨어지기 때문에 일반적으로는 **`std::vector<std::thread>` 사용을 권장**한다.

---
### Join or Detach Threads

```c++
for (auto& thread : threads) thread.join();
for (auto& thread : threads) thread.detach();
```

쓰레드 벡터를 생성하고, 그 안에 새로운 쓰레드를 만들어 계속 넣어줬다.

- **join** : 생성된 쓰레드가 끝날 때까지 마스터 스레드는 기다린다.
- **detach** : 기다리지 않고, 마스터 스레드가 종료되면 함께 종료되도록 한다.
- **auto&** : 쓰레드를 복사하면 안 되기 때문에, `auto` 타입과 참조를 통해 벡터 안의 원본 쓰레드 객체에 직접 `join` 혹은 `detach`를 호출한다.

##### 주의점

- 우리는 쓰레드가 `join` 혹은 `detach` 되었는지 확인해야 하며, `detach`된 쓰레드는 다시 `join`할 수 없다. 

- 한 번 `join`되거나 `detach`된 쓰레드는 다시 사용할 수 없다. 이 말은, 쓰레드가 자신의 함수를 모두 실행하고 종료되면 그 쓰레드는 재사용이 불가능하다는 의미다.

- 모든 쓰레드의 `join` 또는 `detach` 호출은 항상 같은 영역(스코프) 안에서 수행되어야 한다.

---
### Handle Return Values

```c++
template <typename value_t, typename index_t>
value_t fibo(value_t n) {
	value_t a_0 = 0;
	value_t a_1 = 1;
	
	for (index_t index = 0; index < n; index++) {
		const value_t tmp = a_0;
		a_0 = a_1;
		a_1 += tmp;
	}
	
	return a_0;
}
```


위에서 다뤘듯이, 쓰레드가 실행할 함수의 리턴 타입은 반드시 `void` 여야 한다.  
하지만 쓰레드에서 실행한 함수의 **결과 값이 필요할 경우** 어떻게 해야 할까?

```c++

template <typename value_t, typename index_t>
void fibo(value_t n, value_t *result) {
// here we pass the address of result
	value_t a_0 = 0;
	value_t a_1 = 1;
	for (index_t index = 0; index < n; index++) {
		const value_t tmp = a_0; a_0 = a_1; a_1 += tmp;
	}
	*result = a_0;
}
int main(int argc, char* argv[]) { // this runs in the master thread
	const uint64_t num_threads = 32;
	
	std::vector<std::thread> threads;
	std::vector<uint64_t> results(num_threads, 0); 
	
	for (uint64_t id = 0; id < num_threads; id++)
		// specify template parameters and arguments
		threads.emplace_back(fibo<uint64_t, uint64_t>, id, &(results[id]));
		
	for (auto& thread : threads) thread.join(); // join the threads
	
	// print the result
	for (const auto& result : results) std::cout << result << std::endl;
}
```

이 방식의 핵심은, **함수 밖에 결과를 저장할 배열을 미리 만들어 두고**, 그 배열의 **주소를 인자로 전달하여 함수 내부에서 직접 값을 기록하는 것**이다. 이렇게 하면 스레드 함수는 `void`를 반환하지만, 각 스레드가 계산한 결과를 외부 메모리에 안전하게 저장할 수 있다.

#### Promises & Futures
그런데 이렇게 하면 `result` 배열에 직접 값을 기록하기 때문에 여러 스레드가 동시에 접근할 경우 **동기화 문제(race condition)** 가 발생할 수 있다.  

이 점은 **`std::promise`** 와 **`std::future`** 를 사용해서 해결할 수 있다.

![](Pasted%20image%2020251017143655.png)

메인 스레드가 새로운 스레드를 생성할 때마다 하나의 `promise` 객체를 만들고, 그 `promise`를 통해 `future`를 생성한다. 그리고 생성된 스레드에 `promise`를 전달하면, 스레드가 실행하는 함수는 계산된 결과를 `promise` 안에 채워넣게 된다. 메인 스레드는 해당 `future`를 통해 `promise`에 저장된 결과 값을 읽을 수 있으며, 이 과정에서 `future.get()`은 **block 상태**가 되므로 **race condition** 문제를 안전하게 해결할 수 있다.

```c++
#include <future> // std::promise/future
template <typename value_t, typename index_t>
void fibo(value_t n, std::promise<value_t>&& result) { // <- pass promise
	value_t a_0 = 0, a_1 = 1;
	for (index_t index = 0; index < n; index++) {
		const value_t tmp = a_0; a_0 = a_1; a_1 += tmp;
	}
	result.set_value(a_0); // <- fulfill promise
}

int main(int argc, char * argv[]) {
	const uint64_t num_threads = 32;
	std::vector<std::thread> threads;
	std::vector<std::future<uint64_t>> results; 
	
	for (uint64_t id = 0; id < num_threads; id++) { 
		std::promise<uint64_t> promise; 
		results.emplace_back(promise.get_future());
		threads.emplace_back(fibo<uint64_t, uint64_t>, id, std::move(promise));
	}
	
	for (auto& result : results) std::cout << result.get() << std::endl;
	for (auto& thread : threads) thread.join();
}
```

위의 `main` 함수를 보면, 먼저 `future` 타입의 결과 벡터를 생성한다.  

그 다음 각 스레드를 만들기 전에 `promise`를 생성하고,  
`promise.get_future()`로 얻은 `future`를 결과 벡터에 추가한다.

이후 생성된 `promise` 객체를 `fibo` 함수의 인자로 넘겨주면, 각 스레드는 `promise.set_value()`를 호출하여 자신이 계산한 값을 `promise` 내부에 저장한다. 메인 스레드는 `result.get()`을 통해 각 스레드가 채워 넣은 값을 확인하게 된다.

이때 `promise`는 한 번만 사용될 수 있는 단일 객체이므로 **복사가 아닌 이동(move)** 을 통해 전달해야 한다. 따라서 `std::move(promise)`를 사용해 스레드 함수로 넘겨주는 것이다.

결과적으로, `promise`와 `future`를 이용하면 여러 스레드가 동시에 실행되는 환경에서도  
값 전달과 동기화 문제를 안전하게 해결할 수 있다.