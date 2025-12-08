기존 방식에서는 캐시(cache)가 있긴 했지만, 우리가 직접 “이 데이터를 캐시에 넣어두자”라고 관리할 수 있는 구조는 아니었다. 캐시는 하드웨어가 알아서 관리하는 영역이라, **재사용 가능한 데이터가 있어도, 그걸 얼마나 잘 활용할지는 GPU와 캐시 정책에 달려 있고, 프로그래머가 세밀하게 통제하기는 어렵다.**

그래서 CUDA에서는 **사용자가 직접 관리할 수 있는 빠른 메모리 영역**을 하나 더 제공하는데, 그게 바로 **shared memory**다. 이건 캐시와는 느낌이 조금 다르다. 캐시는 “하드웨어가 자동으로 쓰는 고속 버퍼”라면, shared memory는 **프로그래머가 직접 선언하고, 직접 읽고 쓰고, 직접 재사용 패턴을 설계하는 메모리**다.

![](../../images/Pasted%20image%2020251204134901.png)

shared memory는 **global memory까지 가지 않고 블록 내부에서만 데이터를 돌려 쓸 수 있기 때문에 훨씬 빠르다.** 다만 “레지스터처럼 공짜”는 아니고, 어디까지나 메모리이기 때문에 용량도 제한적이고, 잘못 쓰면 충돌(bank conflict)도 날 수 있다. GPU 쪽에서는 보통 **shared memory**라고 부르지만, 조금 더 일반적인 용어로는 **scratchpad memory**라고도 부른다. 말 그대로 프로그래머가 직접 쓰는 “작업용 임시 메모리”인 셈이다.

![](../../images/Pasted%20image%2020251204134934.png)

shared memory의 사용 방법은 매우 단순하다. 커널 내부에서 변수를 선언할 때 앞에 `__shared__` 키워드만 붙여 주면 된다. 예를 들어 공분산 계산처럼 두 개의 타일(tile)을 캐시에 올려놓고 여러 스레드가 재사용해야 하는 경우, 다음과 같이 2차원 배열 형태의 shared memory를 선언할 수 있다.

```c
__shared__ value_t cache_x[chunk_size][chunk_size]; __shared__ value_t cache_y[chunk_size][chunk_size];
```

이렇게 선언된 `cache_x`와 `cache_y`는 **블록 내부의 모든 스레드가 함께 접근하고 재사용할 수 있는 고속 메모리 공간**이 된다. global memory에서 동일한 데이터를 반복해서 불러오는 대신, 한 번 shared memory로 가져와 여러 스레드가 공동으로 쓰기 때문에 메모리 대역폭을 크게 절약할 수 있고 연산 속도도 훨씬 빨라진다.

---

공분산 행렬 $C = \frac{1}{m}D^T D$ 을 계산하는 과정은, 크기가 **2475 × 200,000**인 mean-centered 데이터 행렬 $D$의 컬럼들끼리 서로 곱해 최종적으로 **2475 × 2475** 크기의 공분산 행렬을 만드는 작업이다.

예를 들어 공분산 행렬에서 행 범위가 $j \sim j+7$, 열 범위가 $J \sim J+7$ 에 해당하는 **8×8 블록** 하나를 계산한다고 하자.  

![](../../images/Pasted%20image%2020251204140448.png)

이 블록의 64개 원소는 각각 $D$의 **j~j+7 번째 픽셀 열**, $D$의 **J~J+7 번째 픽셀 열**이 만들어내는 모든 조합 $8×8 = 64개$의 내적이다. 즉, 이 작은 블록을 계산하는 동안 **같은 열 데이터가 여러 번 반복해서 사용된다**는 점이 자연스럽게 드러난다.

이 중복 접근을 줄이기 위해, CUDA에서는 **shared memory**를 사용하여  
자주 재사용되는 열 데이터들을 **한 번만 global memory에서 읽어와**  
쓰레드 블록 내부에서 함께 공유할 수 있다.

그림처럼  j~j+7 범위에 해당하는 열 타일 / J~J+7 범위에 해당하는 열 타일을 각각 shared memory에 위한 공간 두 개를 만들어 두고:

```c
__shared__ value_t cache_x[chunk_size][chunk_size];   // j-side tile
__shared__ value_t cache_y[chunk_size][chunk_size];   // J-side tile
```

global memory에서 chunk 단위로 데이터 일부를 가져와 두 shared memory 타일에 각각 저장하고 블록 내부의 8×8 스레드가 이 shared memory 타일들을 재사용하며 4개의 조합을 동시에 계산한다.

즉, 필요한 열 데이터들을 **shared memory 타일에 일시 저장한 뒤**, 그 타일 안에서 모든 스레드가 재사용하는 방식으로 global memory 접근 횟수를 크게 줄일 수 있다.

shared memory의 용량은 제한적이기 때문에 전체 200,000행을 한 번에 담을 수는 없다.  
그래서 세로 방향을 일정 크기의 **chunk 단위로 잘라(tile)** chunk를 shared에 로드하고 부분 계산 다음 chunk 로드하고고 누적을 반복하여 전체 합을 만든다.

---
## Tiling Example

또 다른 예시를 통해 shared memory의 필요성을 다시 살펴보자.  

예를 들어, 16×16 크기의 데이터 배열이 있다고 하자. 

공분산 행렬을 계산하려면 이 배열의 값들을 여러 조합으로 곱해주어야 한다. 원래 naive 방식이라면, 각 스레드가 자신에게 할당된 픽셀 데이터를 곱하기 위해 필요할 때마다 **global memory에서 직접 데이터를 읽어와야 한다.**

물론 GPU에는 L1/L2 cache가 있어서 어느 정도는 캐싱을 해주지만, 이 캐시는 **하드웨어가 자동으로 관리**하기 때문에 개발자가 어떤 데이터를 재사용하고 싶은지 명시적으로 제어할 수 없다.  
즉, *“잘 캐싱되기만을 바라는 수동적인 방식”*이 된다.

![](../../images/Pasted%20image%2020251204141853.png)

그래서 CUDA에서는 더 능동적으로 메모리를 제어하기 위해 **shared memory**를 활용한다.

shared memory는 같은 SM에 있는 쓰레드들(block 내 쓰레드들)이 함께 접근하고 재사용할 수 있는, 개발자 관리형의 고속 메모리다. 한 번 shared memory에 데이터를 올려두면, 블록 내 모든 스레드가 추가적인 global 접근 없이 같은 데이터를 읽고 사용할 수 있다.

이 과정은 다음과 같이 진행된다.

1. 먼저 global memory에서 일정 크기의 **chunk(타일)** 만큼 데이터를 읽어온다.  
    이 chunk 크기는 shared memory 크기에 맞게 잘라낸 것이다.
    
2. 읽어온 데이터를 shared memory에 복사한 뒤 블록 내의 모든 쓰레드는 이 shared memory에서 데이터를 가져와 연산을 수행한다.
    
3. 모든 쓰레드가 해당 chunk에 대한 계산을 마치면, 다음 chunk를 global memory에서 shared memory로 불러오고 다시 연산을 반복한다.

**global memory 접근은 chunk 단위로 딱 한 번씩**, **연산 중 쓰레드들은 shared memory만 접근**,
**캐시를 예측하거나 기다릴 필요 없이 재사용 필터를 직접 관리**할 수 있다.

이 방식 덕분에 global memory 왕복 횟수가 대폭 줄어들고 스레드들이 동일 데이터를 효율적으로 공유할 수 있어서 공분산 계산 같은 대규모 행렬 연산의 성능을 크게 끌어올릴 수 있게 된다.

---
## CUDA Cov Matrix

![](../../images/Pasted%20image%2020251204142511.png)

### 전체 코드

```c++
template <typename index_t, typename value_t, uint32_t chunk_size = 8>
__global__ void shared_covariance_kernel(
value_t * Data, value_t * Cov, index_t num_entries, index_t num_features) {

	// first index in a window of width chunk_size
	const index_t base_x = blockIdx.x*chunk_size;
	const index_t base_y = blockIdx.y*chunk_size;
	
	// local thread identifiers
	const index_t thid_y = threadIdx.y;
	const index_t thid_x = threadIdx.x;
	
	// global thread identifiers
	const index_t x = base_x + thid_x;
	const index_t y = base_y + thid_y;
	
	// optional early exit for tiles above the diagonal
	if (base_x >= base_y + chunck) return;
	
	// allocate shared memory
	__shared__ value_t cache_x[chunk_size][chunk_size];
	__shared__ value_t cache_y[chunk_size][chunk_size];
	
	// compute the number of chunks to be computed
	const index_t num_chunks = SDIV(num_entries, chunk_size);
	value_t accum = 0; // accumulated value of scalar product
	
	//Start of the main loop for each chunk
	for (index_t chunk = 0; chunk < num_chunks; chunk++) {
	
		// assign thread IDs to rows and columns
		const index_t row = thid_y + chunk*chunk_size;
		const index_t col_x = thid_x + base_x;
		const index_t col_y = thid_x + base_y;
		
		// check if valid row or column indices
		const bool valid_row = row < num_entries;
		const bool valid_col_x = col_x < num_features;
		const bool valid_col_y = col_y < num_features;
		
		cache_x[thid_y][thid_x] 
			= valid_row*valid_col_x ? Data[row*num_features + col_x] : 0;
		cache_y[thid_y][thid_x] 
			= valid_row*valid_col_y ? Data[row*num_features + col_y] : 0;
			
		__syncthreads(); 
	
		if (x <= y) // optional early exit
			for (index_t k = 0; k < chunk_size; k++) 
				accum += cache_y[k][thid_y] * cache_x[k][thid_x];
			
		__syncthreads();
	
	} // end for loop over chunk entries
	
	if (y < num_features && x <= y) 
		Cov[y*num_features+x] = Cov[x*num_features+y] = accum / num_entries;
}
```



---

```c++
template <typename index_t, typename value_t, uint32_t chunk_size = 8>
__global__ void shared_covariance_kernel(
    value_t * Data, value_t * Cov, index_t num_entries, index_t num_features) {
```

- `index_t`, `value_t` : 인덱스와 값 타입을 템플릿으로 받음 (예: `int`, `float`).
- `chunk_size` : 한 번에 처리할 타일의 크기(기본 8).
- `Data` : mean-centered 데이터 행렬 (D) (크기 m × n).
- `Cov` : 공분산 행렬 (C) (크기 n × n).
- `num_entries` = m (이미지 개수), `num_features` = n (픽셀 개수).
    

이 커널의 목표는
**shared memory(타일링)를 이용해서 공분산 행렬 $D^T D / m$을 효율적으로 계산.**

---

### 1. 타일(윈도우)의 시작 위치 계산

```c++
// first index in a window of width chunk_size
const index_t base_x = blockIdx.x*chunk_size;
const index_t base_y = blockIdx.y*chunk_size;
```

- 그리드가 2차원 블록으로 깔려 있고, 각 블록은 공분산 행렬의 **chunk_size × chunk_size** 타일 하나를 담당.
    
- `base_x` : 이 블록이 담당하는 타일의 열 방향 시작 인덱스 (J).
- `base_y` : 이 블록이 담당하는 타일의 행 방향 시작 인덱스 (j).
    

즉, 이 블록은 공분산 행렬의  
행: `base_y ~ base_y + chunk_size - 1`  
열: `base_x ~ base_x + chunk_size - 1`  
에 해당하는 작은 정사각형 타일을 계산한다.

---

### 2. 블록 내부에서 thread ID

```c++
// local thread identifiers
const index_t thid_y = threadIdx.y;
const index_t thid_x = threadIdx.x;

// global thread identifiers
const index_t x = base_x + thid_x;
const index_t y = base_y + thid_y;
```

- `thid_x`, `thid_y` : 블록 내부에서 스레드 위치 (0 ~ chunk_size-1).
- `x`, `y` : 공분산 행렬 전체 기준에서 이 스레드가 담당하는 **열(x), 행(y)** 위치.

→ 이 스레드는 결국 `Cov[y][x]`(대칭성 이용하면 위/아래 둘 다)에 대응하는 원소를 계산한다.

---
### 3. 대각선 위 타일은 바로 스킵 (대칭성 이용)

```c++
// optional early exit for tiles above the diagonal
if (base_x >= base_y + chunk_size) return;
```

- 공분산 행렬은 대칭이므로,  
    **대각선 위쪽 타일(열 > 행)** 은 계산할 필요가 없다.
    
- `base_x >= base_y + chunk_size` 인 타일은 “완전히 대각선 위”에 있으므로,  
    이런 타일들은 **아예 커널 초반에서 리턴**해서 연산을 건너뜀.

---

### 4. shared memory 타일 선언

```c++
// allocate shared memory
__shared__ value_t cache_x[chunk_size][chunk_size];
__shared__ value_t cache_y[chunk_size][chunk_size];
```

- `cache_x` : 열 방향으로 `base_x` 주변 컬럼들을 담는 타일.
- `cache_y` : 열 방향으로 `base_y` 주변 컬럼들을 담는 타일.
    

두 타일 모두 크기는 `chunk_size × chunk_size`.  
각 chunk 루프마다 global memory → shared memory로 데이터를 옮겨와,  
이 타일 안에서 여러 스레드가 재사용한다.

---

### 5. 전체 chunk 개수와 누적 변수

```c++
// compute the number of chunks to be computed
const index_t num_chunks = SDIV(num_entries, chunk_size);
value_t accum = 0; // accumulated value of scalar product
```

- `num_chunks` : 세로 방향(이미지 개수 m)을 chunk_size씩 나누었을 때 몇 번 반복해야 하는지. 
    (예: m=200,000, chunk_size=8 → 대략 25,000번)

- `accum` : 이 스레드가 담당하는 `(y, x)` 위치에 대해 $\sum_i D_i(y) D_i(x)$을 누적할 레지스터 변수.
    
---

### 6. 각 chunk(타일 조각)마다 반복

```c++
// Start of the main loop for each chunk
for (index_t chunk = 0; chunk < num_chunks; chunk++) {
```

각 chunk 루프에서 하는 일:

1. global memory에서 “현재 chunk에 해당하는 행 구간”을 shared memory로 로드.
    
2. 그 chunk 내부에서 부분 스칼라곱을 계산해 `accum`에 더함.
    
3. 다음 chunk로 넘어가서 반복.
    

---

### 7. 이 chunk에서 사용할 행/열 인덱스 계산

```c++
// assign thread IDs to rows and columns
const index_t row   = thid_y + chunk*chunk_size;
const index_t col_x = thid_x + base_x;
const index_t col_y = thid_x + base_y;
```

- `row` : 현재 chunk에서 이 스레드가 담당해서 가져올 **데이터 행(i)** 인덱스.
    
    - chunk가 0일 때: 0~7
        
    - chunk가 1일 때: 8~15 … 이런 식으로 증가.
        
- `col_x` : x방향(열) 쪽 타일에서 실제 feature 인덱스 (J~J+7).
- `col_y` : y방향(열) 쪽 타일에서 실제 feature 인덱스 (j~j+7).  

즉, 이 스레드는 데이터 행: `row` + 열 두 개: `col_x`, `col_y`에 해당하는 값을 shared memory에 옮겨오는 역할을 한다.

---

### 8. 인덱스 유효성 검사 + shared memory 로딩

```c++
// check if valid row or column indices
const bool valid_row   = row   < num_entries;
const bool valid_col_x = col_x < num_features;
const bool valid_col_y = col_y < num_features;

cache_x[thid_y][thid_x] 
    = valid_row*valid_col_x ? Data[row*num_features + col_x] : 0;
cache_y[thid_y][thid_x] 
    = valid_row*valid_col_y ? Data[row*num_features + col_y] : 0;

__syncthreads(); 
```

- 끝 chunk에서는 `row`가 범위를 넘을 수 있기 때문에, 유효한 인덱스인지 체크해서 out-of-bounds를 막는다.
    
- `cache_x[thid_y][thid_x]` 에는 `Data[row][col_x]` 값 (x쪽 컬럼),
    
- `cache_y[thid_y][thid_x]` 에는 `Data[row][col_y]` 값 (y쪽 컬럼)을 저장.

-  유효하지 않은 경우에는 0을 넣어서 계산에 영향 없게 처리.


`__syncthreads()` : 블록 내 모든 쓰레드가 shared memory에 작성을 끝낼 때까지 기다린 뒤, 그 다음 단계(부분합 계산)로 넘어감.

---

### 9. 이 chunk에서 스칼라곱 부분 계산

```c++
if (x <= y) // optional early exit
    for (index_t k = 0; k < chunk_size; k++) 
        accum += cache_y[k][thid_y] * cache_x[k][thid_x];

__syncthreads(); 
} // end for loop over chunk entries
```

- `if (x <= y)` : 공분산 행렬의 **아래 삼각형(대각 포함)** 만 계산.  
    (위쪽 삼각형은 대칭으로 복사하기 때문에 계산 안 함)
    
- `for (k = 0; k < chunk_size; k++)`  
	이 chunk 안에서 `k` 방향으로 쭉 돌면서 `cache_y[k][thid_y]` 와 `cache_x[k][thid_x]` 를 곱해서 `accum`에 더한다. 즉, 한 chunk(행 8개)에 대해서 부분 스칼라곱을 계산하는 것.
    
- 두 번째 `__syncthreads()`  
    이 chunk에 대한 연산이 모두 끝났으니, 다음 chunk에서 shared memory를 다시 덮어써도 괜찮도록 스레드들을 동기화.

이 과정을 모든 chunk에 대해 반복하면,  
결국 `accum`에는 $\sum_{i=0}^{m-1} D_i(y) D_i(x)$ 전체가 누적된다.

---

### 10. 공분산 결과를 global memory에 기록

```c++
if (y < num_features && x <= y)
    Cov[y*num_features+x] = Cov[x*num_features+y] = accum / num_entries;
}
```

- `y < num_features` : 행 인덱스가 feature 범위 안인지 체크.
    
- `x <= y` : 아래 삼각형만 직접 계산. 위쪽은 복사로 채울 것이므로 여기서는 쓰지 않음.
    
- `Cov[y*num_features + x]` 에 결과를 쓰고, 동시에 `Cov[x*num_features + y]` 에도 같은 값을 써서 **C = Cᵀ(대칭)** 을 만족시키도록 한다.


결과적으로 shared memory 타일을 이용해 global memory 접근을 줄이고, chunk 단위로 데이터를 나눠 처리하면서, 대칭성까지 활용해 연산량을 절반 정도로 줄인 공분산 커널이다.
    
---
## Performance Considerations

chunk size(타일 크기)가 커지면 커질수록 성능이 좋아진다.  

예를 들어, chunk_size = 8이면 하나의 블록에서 8×8 = 64개의 스레드가 동작하고,  
각 스레드는 shared memory에 데이터를 한 번만 로드한 뒤 여러 번 재활용할 수 있기 때문이다.  
즉, chunk가 클수록 **재사용되는 데이터의 양이 많아지고**, 그만큼 global memory 접근이 줄어들어 더 빠른 성능을 낼 수 있다.

### chunk_size = 2

- 각 블록의 쓰레드 수: **2 × 2 = 4 threads**
    
- 각 블록의 global memory load 수: **4 × 2 = 8 loads = 32 bytes**
    
- 각 블록이 수행하는 연산량: **2 × 4 = 8 mul/add = 8 FLOPs**
    
- CGMA:

$$\text{CGMA} = \frac{8\ \text{FLOPs}}{8 \times 4\text{byte}} = 0.25 \text{ flop/byte}$$

즉, global memory 접근 대비 연산량이 적어서 효율이 낮다.

####  chunk_size = 8일 때

- 각 블록의 쓰레드 수: **8 × 8 = 64 threads**
    
- 각 블록의 global memory load 수: **2 × 64 = 128loads = 512 bytes**
    
- 각 블록이 수행하는 연산 수: 
	- **스레드당 연산 = 8 FLOPs**
	- 전체 스레드 64개 = **64 × 8 = 512 FLOPs**
    
$$\text{CGMA} = \frac{512\ \text{FLOPs}}{128 \times 4\text{byte}} = 1 \text{ flop/byte}$$

chunk_size=2일 때보다 **4배 이상 더 높은 연산 효율**을 가지게 된다.  

chunk_size를 크게 설정하면 한 번 global memory에서 가져온 데이터를 shared memory에 올려 두고 더 많이 재활용할 수 있기 때문에, 연산 대비 글로벌 메모리 접근 비율(CGMA)이 높아지고 성능도 크게 향상된다. 예를 들어 chunk_size를 2에서 8로 늘리면 CGMA가 0.25에서 1.0으로 증가하며, 같은 메모리 대역폭으로 더 많은 연산을 수행할 수 있게 된다. 그래서 처음 보면 “chunk_size는 클수록 무조건 좋은 것 아닌가?”라는 생각이 들 수 있다.

하지만 여기에는 중요한 제약이 하나 있다. shared memory는 SM(SM당 64KB 등) 전체가 공유하는 자원이기 때문에, 블록 하나가 사용하는 shared memory 크기가 커질수록 하나의 SM에 동시에 배치할 수 있는 block의 수가 줄어든다는 점이다. SM에서는 여러 block이 동시에 올라가 있어야 충분한 수의 warp를 유지할 수 있고, 그래야만 어떤 warp가 global memory 접근으로 대기 중일 때 다른 warp를 실행시키며 레이턴시를 숨길 수 있다. 그런데 chunk_size가 너무 커져 shared memory를 많이 차지하게 되면, 한 SM에 단 하나의 block만 올라가는 상황이 발생할 수 있고, 이렇게 되면 warp 수도 급격히 줄어들면서 오히려 전체 성능이 떨어질 수 있다.

즉, chunk_size를 키우는 과정은 CGMA 향상(=데이터 재사용 증가)과 SM occupancy 감소(=활성 warp 감소) 사이에서 균형을 맞추는 작업이다. shared memory 용량 자체는 64KB 정도로 꽤 넉넉해 보이지만, 실제로는 여러 block이 동시에 SM을 공유해야 한다는 점 때문에 chunk_size를 무작정 크게 잡을 수는 없다. 결국 최적의 chunk_size는 shared memory 사용량과 occupancy를 함께 고려하여, **데이터 재사용 효율이 높으면서도 충분한 수의 warp를 유지할 수 있는 지점**을 찾는 것이 중요하다.