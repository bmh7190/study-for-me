
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

자 block distribution은 쓰레드 별로 하나의 큰 블록 덩어리를 나눠주는 방식인데, 간단히 생각해보면 전체 할 일을 / thread 수 로 나누면 된다. 

```c++
#define SDIV(x, y) (((x) + (y) - 1) / (y))
```

근데 정수가 안 나올 수 있기 때문에 SIDV(x, y) 를 통해 무조건 정수가 나오도록 한다.
