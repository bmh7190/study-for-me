
![](../images/Pasted%20image%2020251017144941.png)

```c++
for (int i = 0; i<m; i++){
	b[i] = 0.0;
	
	for(int j =0; j<n; j++)
		b[i] = A[i][j] + x[j];
}
```

```c++
// the sequential DMV product Ax = b (A of size mxn, b of size m, x of size n)
template <typename value_t, typename index_t>
void sequential_mult(
	std::vector<value_t>& A, std::vector<value_t>& x, std::vector<value_t>& b, index_t m, index_t n) {
	
	for (index_t row = 0; row < m; row++) {
		value_t accum = value_t(0);
	
	for (index_t col = 0; col < n; col++)
		accum += A[row*n + col] * x[col];
		b[row] = accum;
	}
}

int main(int argc, char* argv[]) {
	const uint64_t n = 1UL << 15;
	const uint64_t m = 1UL << 15;
	
	std::vector<uint64_t> A(m*n); // alloc
	std::vector<uint64_t> x(n);
	std::vector<uint64_t> b(m);
	
	init(A, x, m, n); // init
	sequential_mult(A, x, b, m, n); // mult
}
```

## Multithread Matrix-Vector Multiplication

위의 코드는 벡터 곱을 그냥 반복문을 통해서 구하고 있다, 근데 이거 잘 생각해보면, 똑같은 연산을 모든 행과 열에서 수행하고 있으므로 병렬적으로 할 수 있다.

그래서 병렬적으로 벡터 곱을 계산하기 위해서 두 가지 방법을 예로 들 수 있다.

#### Block distribution of threads

첫번째 방법은 블록 단위로 쓰레드들에게 할 일을 나눠주는 것이다. 간단하게 생각해보면 똑같은 일을 하고 있기 때문에 그냥 전체 행을 thread 수로 나눠서 각 쓰레드들에게 배정하면 된다.

![](../images/Pasted%20image%2020251017145638.png)

#### Cyclic distribution of threads

다음 방법은 처음부터 할당하지 말고 그냥 쓰레드들에게 순환적으로 배분하는 것이다.

![](../images/Pasted%20image%2020251017145834.png)

이제 각 방법들이 c++11 multithread를 통해 어떻게 구현할 수 있는지 살펴보자

---
## Multithread MV with Block Distribution

```c++
#define SDIV(x, y) (((x) + (y) - 1) / (y))

template <typename value_t, typename index_t>
void block_parallel_mult(
std::vector<value_t>& A, std::vector<value_t>& x, std::vector<value_t>& b,
index_t m, n, index_t num_threads = 8) {

	auto block = [&](const index_t& id) -> void {
		const index_t chunk = SDIV(m, num_threads); 
		const index_t lower = id*chunk; 
		const index_t upper = std::min(lower + chunk, m);
		
		for (index_t row = lower; row < upper; row++) { 
			value_t accum = value_t(0); 
			
			for (index_t col = 0; col < n; col++)
				accum += A[row*n + col] * x[col];
				
			b[row] = accum;
		}
	}
	
	std::vector<std::thread> threads;
	
	for (index_t id = 0; id < num_threads; id++) threads.emplace_back(block, id);
	
	for (auto& thread : threads) thread.join();
}
```

**Block distribution**은 전체 작업을 쓰레드의 개수만큼 **큰 블록 단위로 나누는 방식**이다.  
간단히 말해서 전체 해야 할 일의 양을 **쓰레드 개수로 나눈 만큼** 각 쓰레드에 맡기는 구조다.

##### 일 나누기

```c++
#define SDIV(x, y) (((x) + (y) - 1) / (y))
```

하지만 단순히 `m / num_threads`로 나누면 정수가 나오지 않을 수 있기 때문에, `SDIV(x, y)` 매크로를 사용해 **나머지가 생길 경우 올림 처리**를 한다.  

즉, 각 블록의 크기(`chunk`)가 정수가 되도록 보장한다.

##### 어디서부터 어디까지 계산할지

이제 각 쓰레드가 얼마만큼 계산할지(`chunk`)를 정했다면, 각 쓰레드가 **어디서부터 어디까지 계산할지**를 정해야 한다.

이 로직은 `block` 함수 안에서 처리된다. 반복문으로 쓰레드를 생성할 때 `id` 값을 함께 넘기는데, 이 `id`는 각 쓰레드의 번호 역할을 한다.

- 0번 쓰레드의 시작 지점(`lower`)은 `0 * chunk`, 즉 행렬의 맨 처음이다.  
    계산 범위는 `chunk`만큼이므로, 끝 지점(`upper`)은 `lower + chunk`가 된다.
    
- 1번 쓰레드는 `1 * chunk`부터 시작하므로, 0번이 맡은 구간 다음부터 계산을 시작한다.
    
- 이런 식으로 모든 쓰레드가 자신에게 할당된 구간만큼 계산을 수행한다.
    
- 마지막 쓰레드는 할당된 구간이 전체 행의 개수(`m`)를 초과할 수 있으므로,  
    `std::min(lower + chunk, m)`을 사용해 **실제 행의 개수를 넘지 않게 조정**한다.

##### 장단점

우선 전체를 블록으로 나눠서 쓰레드 별로 배분하기 때문에, data locality가 높다. 따라서 cache 사용성이 좋고, false sharing 도 적을 것이다.

하지만 마 `min(lower + chunk, m)` 에서 알 수 있듯이 맨 마지막 쓰레드는 무조건 다른 쓰레드들보다 작은 일을 하게 될 것이다. 따라서 load balancing이 완벽하지 않다. 

---
## Multithreaded MV with Cyclic Distribution

다음은 **cyclic**한 방식이다. 여기서는 쓰레드를 **순환**시키면서 일을 배분한다.

```c++
template <typename value_t, typename index_t>
void block_parallel_mult(
std::vector<value_t>& A, std::vector<value_t>& x, std::vector<value_t>& b,
index_t m, n, index_t num_threads = 8) {

	auto cyclic = [&](const index_t& id) -> void {
		for (index_t row = id; row < m; row+=num_threads) {
			value_t accum = value_t(0);
		
		for (index_t col = 0; col < n; col++)
			accum += A[row*n + col] * x[col];
			
		b[row] = accum;
		
		}
	}
	
	std::vector<std::thread> threads;
	for (index_t id = 0; id < num_threads; id++) threads.emplace_back(cyclic, id);
	for (auto& thread : threads) thread.join();
}
```

그냥 **행 단위로** 각 쓰레드에 배분한다. 구현도 단순하다. 계속 순환하면서 스레드에게 일을 주므로 **load balancing**이 비교적 좋다. 다만 하나의 스레드가 인접한 행의 데이터를 연속적으로 계산하지 않기 때문에 **false sharing**이 늘어날 수 있다.

그럼에도 더 생각해볼 수 있는 점은, 결과 벡터인 `b`에 **바로 쓰지 않고** 각 스레드의 **임시 버퍼**에 먼저 저장한 뒤, 마지막에 한꺼번에 `b`로 복사하는 방법을 쓰면 false sharing을 어느 정도 줄일 수 있다는 것이다.

---
## Static Block-Cyclic Distribution

잠깐 생각해보면 두 방법을 섞어서 쓸 수 있다. **block 단위로 나눈 뒤**, 그 블록들을 **cyclic하게 배분**하는 방법이다.

![](../images/Pasted%20image%2020251017152026.png)

이 방법은 한 단위에 포함된 **block 크기/개수**를 어떻게 잡느냐에 따라 균형이 달라진다. 블록이 **너무 크면** 균형(load balancing)이 나빠질 수 있고, **너무 작으면** 스레드 간 쓰기 위치가 자주 엇갈리며 **false sharing** 가능성이 높아진다. 따라서 **캐시 친화적(cache-friendly)** 으로 블록을 정하는 것이 중요하다. 예를 들어 **캐시 라인이 64 byte**라면(일반적), `double` 기준 **8개(8×8B=64B)** 단위로 맞추거나, 그 배수로 블록 크기를 설정해 라인 경계를 잘 맞춰주는 식의 튜닝이 효과적이다.


---
```c++
template <typename value_t, typename index_t>
void block_parallel_mult(
	std::vector<value_t>& A, 
	std::vector<value_t>& x, 
	std::vector<value_t>& b,
	index_t m, n, index_t num_threads = 8,
	index_t chunk_size=64/sizeof(value_t)) 
{
	auto block_cyclic = [&] (const index_t& id) -> void {
		
		const index_t offset = id*chunk_size;
		const index_t stride = num_threads*chunk_size;

		for (index_t lower = offset; lower < m; lower += stride) {
			const index_t upper = std::min(lower + chunk_size, m); block
			
			for (index_t row = lower; row < upper; row++) { 
				value_t accum = value_t(0);
				
				for (index_t col = 0; col < n; col++)
					accum += A[row*n + col] * x[col];
				
				b[row] = accum;
			}
		}
	}
	
	std::vector<std::thread> threads;
	
	for (index_t id = 0; id < num_threads; id++) 
		threads.emplace_back(block_cyclic, id);
	
	for (auto& thread : threads) thread.join();
	
}
```

##### 일 나누기
일 자체는 64/8 해서 8개 씩 돌아서 하기로 했다.

##### 어디서부터 어디까지 할지

자 여기서 chunk size 즉 실제 thread 가 할 일이 chunck size 부터 되지 않는다. 이게 무슨 말이냐면 1번 쓰레가 100 을 일하면 다음 쓰레드는 100부터 200이 아니라는 것이다 왜냐면 cyclic 하게 배분 중이니까! 그래서 stride를 도입했는데, chunck  * num_threads 를 해서 내가 다음에 일 할 곳은 어딘지를 계산한다.