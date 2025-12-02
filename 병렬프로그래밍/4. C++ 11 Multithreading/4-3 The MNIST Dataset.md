MNIST 데이터셋은 손글씨 숫자(0~9) 이미지를 모은 대표적인 머신러닝 데이터셋이다.  
총 약 **65,000장(훈련용 60,000장 + 테스트용 10,000장)** 의 이미지로 구성되어 있으며,  
각 이미지는 **28 × 28 픽셀**의 흑백(grayscale) 이미지다.

![](../../images/Pasted%20image%2020251017155154.png)

이때 각 이미지를 행렬로 나타낼 수 있다.  
하나의 이미지를 `D_i`라 하면, 그 안의 `j`번째 픽셀 값을 `D_{ij}`로 표현할 수 있다.

여기서 `D_{ij}`는 **i번째 이미지의 j번째 픽셀 값**을 의미하며,  
픽셀 값은 보통 **0~255** 범위의 정수(밝기 값)를 가진다.

이렇게 하면 MNIST 전체 데이터셋은 `65,000 × 784` 크기의 행렬로 생각할 수 있으며,  
각 행이 하나의 이미지(벡터화된 형태)에 해당하고, 각 열이 픽셀 위치를 나타낸다고 볼 수 있다.

---
## All-Pairs Distance Matrix

이제 65,000장 중 임의의 두 이미지를 골랐을 때, “얼마나 같은가/다른가”를 **거리(distance)** 로 정의하자.

### Seqeuntial 한 방법

```c++
template <typename index_t, typename value_t>
void sequential_all_pairs(
	std::vector<value_t>& mnist, std::vector<value_t>& all_pair, index_t rows, index_t cols) {
	for (index_t i = 0; i < rows; i++) { 
		for (index_t I = 0; I <= i; I++) {
			value_t accum = value_t(0); // compute squared Euclidean distance
			
			for (index_t j = 0; j < cols; j++) {
				value_t residue = mnist[i*cols+j] - mnist[I*cols+j];
				accum += residue * residue;
			}
			
			all_pair[i*rows+I] = all_pair[I*rows+i] = accum; 
		}
	}
}

int main() {
	typedef no_init_t<float> value_t; 
	typedef uint64_t index_t;
	const index_t rows = 65000; 
	const index_t cols = 28 * 28;
	std::vector<value_t> mnist(rows*cols); 
	load_binary(mnist.data(), rows*cols,"./data/mnist_65000_28_28_32.bin");
	std::vector<value_t> all_pair(rows*rows);
	sequential_all_pairs(mnist, all_pair, rows, cols); 
```

`sequential_all_pairs`에서 `rows`는 **이미지 수**, `cols`는 **각 이미지의 픽셀 수(= 28×28)** 를 의미한다. 바깥 루프에서 `i`번째 이미지를 늘려가며, 안쪽 루프에서 **0번째부터 `i`번째까지**의 이미지와의 **제곱 유클리드 거리**를 계산한다. 계산 결과는 대칭이므로 `all_pair[i*rows + I]`와 `all_pair[I*rows + i]`에 **동시에 기록**한다.


---
### All-Pairs with Static Block-Cyclic Distribution

여기서 이제 **block-cyclic**으로 구현해보자. 스레드에게 **이미지의 인덱스 `i`** 를 나눠준다.

예를 들어 아래 그림처럼, **0번 스레드**는 (예시) **1번, 2번 이미지** 인덱스를 맡아 각각 비교를 수행하고, **1번 스레드**는 **3번, 4번 이미지** 인덱스를 맡아 3번과 비교, 4번과 비교를 수행한다.  

![](../../images/Pasted%20image%2020251017160331.png)

```c++
template <typename index_t,typename value_t>
void parallel_all_pairs(std::vector<value_t>& mnist, std::vector<value_t>& all_pair,
index_t rows, index_t cols, index_t num_threads = 64, index_t chunk_size = 64 / sizeof(value_t)) {
	
	auto block_cyclic = [&](const index_t& id) -> void {
		const index_t off = id*chunk_size;
		const index_t str = num_threads*chunk_size;
		
		for (index_t lower = off; lower<rows; lower+=str) {
			const index_t upper = std::min(lower + chunk_size, rows);
			
			for (index_t i = lower; i < upper; i++) { 
				for (index_t I = 0; I <= i; I++) {
					value_t accum = value_t(0); 
					
					for (index_t j = 0; j < cols; j++) {
						value_t residue = mnist[i*cols + j] - mnist[I*cols + j];
						accum += residue * residue;
					}
							
					all_pair[i*rows+I] = all_pair[I*rows+i] = accum;
				}
			}
		}
	};
	
	std::vector<std::thread> threads; // business as usual
	
	for (index_t id = 0; id < num_threads; id++) 
		threads.emplace_back(block_cyclic, id);
	
	for (auto& thread : threads) thread.join();
}
```

- **분할 방식**: `i` 인덱스를 `chunk_size` 행 단위 블록으로 나눠서, 각 스레드가 `off = id * chunk_size`에서 시작하고 `str = num_threads * chunk_size` 간격으로 **cyclic**하게 처리한다.
    
- **중복/경쟁 회피**: `I ≤ i`만 계산하고 `all_pair[i, I]`와 `all_pair[I, i]`에 동시에 기록하므로 서로 다른 스레드가 같은 위치를 쓰지 않는다.
    
- **캐시 친화성**: `chunk_size = 64 / sizeof(value_t)`처럼 캐시 라인(64B) 기준으로 맞추면 공간 지역성과 false sharing 측면에서 유리하다.

![](../../images/Pasted%20image%2020251017161142.png)

실행 시간을 보면 chunck size 크면 speedup은 떨어지고 있다. 

cuncksize가 작으면 loadbalancing 자체는 잘 되기 때문에 속도는 빨라지지 만 data locality가 감소하기 때문에 속도가 많이 늘어나진 않는다.

---

## Dynamic block-cyclic distribution

지금까지는 단순히 **스레드 번호 순서대로** 일을 나눠서 각 스레드가 고정된 구간만 처리하도록 했다.  하지만 이번에는 **작업을 더 빨리 끝내는 스레드에게** 남은 일을 **끝나는 순서대로 다시 배분하는 방식**을 사용할 수 있다.

![](../../images/Pasted%20image%2020251017161734.png)

이 방식을 구현하기 위해서는 몇 가지 고려할 점이 있다.

먼저, 기존에는 스레드 번호(`id`)를 기준으로 `lower` 값을 계산해 각 스레드가 맡을 시작 지점을 정했다. 하지만 이제는 **작업이 끝나는 순서에 따라** 새로운 일을 배분해야 하기 때문에, 각 스레드가 일을 마친 뒤 **전역 변수(global variable)** 를 통해 “다음 작업의 시작 위치”를 받아와야 한다.

문제는 이 전역 변수가 **모든 스레드에 의해 동시에 접근**된다는 점이다.  

여러 스레드가 동시에 값을 읽거나 수정하면 **race condition(경쟁 상태)** 이 발생할 수 있다. 따라서 이 공유 변수에 접근할 때는 반드시 **상호배제(mutex)** 나 **원자적 연산(atomic operation)** 을 사용하여 **접근 보장(synchronization)** 을 해줘야 한다.

### Mutex

```c++
#include <mutex>
std::mutex mutex;
// to be called by threads

void some_function(...) {
	mutex.lock()
	// this region is only processed by one thread at a time
	mutex.unlock();
	// this region is processed in parallel
}
```

만약에 공유변수에 대해 접근을 할 때는 다른 쓰레드들이 접근할 수 없도록, lock을 걸고 내가 일을 끝낼 때 다른 사람들이 들어올 수 있도록 unlock을 하면 된다.

근데 꼭 unlock을 해줘야 하는데 안 할 경우에는 다른 쓰레드들이 접근할 수 없는 문제가 발생한다. 특히 C++ 컴파일러는 이 점을 확인하지 않기 때문에 데드락에 걸릴 수도 있다.

### Avoiding Deadlocks

이 단점을 커버하기 위해서 C++은 lock_guard를 제공한다. 

간단히 말하면 이것을 원하는 영역에 사용하면 그 선언 시작에 lock을 걸고, 그 영역 맨 마지막에 unlock을 삽입해서 그 영역을 벗어나면 자동으로 unlock이 되는 것다.



```c++
#include <mutex>
std::mutex mutex;
// to be called by threads
void some_function(...) {

	{
	// here we acquire the lock
	std::lock_guard<std::mutex> lock_guard(mutex);
	// this region is locked by the mutex
	} // <- here we release the lock
	
// this region is processed in parallel
}
```

---
### All-Pairs with Dynamic Block-Cyclie Distribution

```c++
#include <mutex> 
template <typename index_t, typename value_t>
void dynamic_all_pairs(std::vector<value_t>& mnist, std::vector<value_t>& all_pair,
index_t rows, index_t cols, index_t num_threads = 64, index_t chunk_size = 64 / sizeof(value_t)) {
	std::mutex mutex;
	index_t global_lower = 0;
	
	auto dynamic_block_cyclic = [&]() -> void {
		index_t lower = 0; 
		while (lower < rows) {
		
			{ 
				std::lock_guard<std::mutex> lock_guard(mutex);
				lower = global_lower;
				global_lower += chunk_size;
			} 
		
			const index_t upper = std::min(lower+chunk_size, rows); 
			
			for (index_t i = lower; i < upper; i++) { 
			
				for (index_t I = 0; I <= i; I++) {
					value_t accum = value_t(0);
					
					for (index_t j = 0; j < cols; j++) {
						value_t residue = mnist[i*cols + j] - mnist[I*cols + j];
						accum += residue * residue;
					}
					
					all_pair[i*rows+I] = all_pair[I*rows+i] = accum; 
				}
			}
		}
	};
	
	std::vector<std::thread> threads;
	
	for (index_t id = 0; id < num_threads; id++) 
		threads.emplace_back(dynamic_block_cyclic);
		
	for (auto& thread : threads) thread.join();
}
```

![](../../images/Pasted%20image%2020251017162745.png)

static한 방식과 dynamic 방식을 비교해 보면, **성능 차이가 드라마틱하지 않을 때가 많다.**

- **chunk size가 매우 큰 경우**: 결국 각 스레드가 맡는 작업이 둘 다 거대해지므로, static이든 dynamic이든 **부하 분산 이점이 작아** 성능 개선이 거의 없다.
    
- **chunk size가 매우 작은 경우**: 모든 스레드가 **작은 단위 작업을 빈번히** 처리하므로, 두 방식 모두 유사하게 동작하고 **차이가 미미**해진다(오히려 관리/동기화 오버헤드가 눈에 띌 수 있다).
    

또, **작업량이 뒤로 갈수록 증가하는 불균형 패턴**에서는 dynamic 방식이 유용할 수 있지만, 작업량이 **동일**하다면 매번 **mutex(또는 atomic) 획득 비용** 때문에 성능이 떨어질 수 있다.

나아가 **작업량이 큰 블록부터 먼저 시작**하도록 정렬하면(예: greedy, longest-first) 더 나은 성능을 기대할 여지가 있다.

정리하면, **항상 dynamic block-cyclic이 정답은 아니다.**  
데이터 특성(작업량 분포), chunk 크기, 동기화 오버헤드 등을 고려해 **static / dynamic / hybrid(정렬+dynamic)** 중 상황에 맞는 방식을 선택해야 한다.