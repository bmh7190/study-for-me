이번 예시는 **픽셀 간의 상관관계를 계산하는 공분산(covariance) 계산 예시**이다. 

이전 단계에서 우리는 모든 이미지에 대해 픽셀 평균을 제거하여, 각 이미지가 0-평균이 되도록 정규화한 **mean centered data matrix**를 만들어 두었다. 이제 이 정규화된 데이터 행렬을 이용해, 서로 다른 픽셀들 간에 어떤 상관성이 있는지를 나타내는 공분산 행렬을 계산하는 것이 이번 단계의 목표이다.

공분산 행렬은 두 픽셀 $j$와 $j'$가 여러 이미지에서 동시에 어떻게 변화하는지를 수치적으로 표현해주는 것으로, mean centered된 데이터 $D$에 대해 $C = \frac{1}{m} D^T D$ 형태로 계산된다. 결과적으로 $2475 \times 2475$ 크기의 공분산 행렬이 만들어지며, 이는 모든 픽셀 간 관계를 포함하는 매우 중요한 통계적 정보가 된다.

![](../../images/Pasted%20image%2020251203224754.png)

---

## Navie Covariance Matrix Computation

```c++
//CUDA kernel performing naive covariance matrix computation
template <typename index_t, typename value_t> __global__
void covariance_kernel(
value_t * Data, // centered data matrix
value_t * Cov, // covariance matrix
index_t num_entries, // number of images (m)
index_t num_features) { // number of pixels (n)

// determine row and column indices of Cov (j and j' <-> J)
const auto J = blockDim.x*blockIdx.x + threadIdx.x;
const auto j = blockDim.y*blockIdx.y + threadIdx.y;
	if (j < num_features && J < num_features) { // check range of indices
		value_t accum = 0; // store scalar product in a register
		
		// accumulate contribution over images (entry <-> i)
		for (index_t entry = 0; entry < num_entries; entry++)
			accum += Data[entry*num_features + j] * Data[entry*num_features + J];
		Cov[j*num_features + J] = accum / num_entries;
	}
}
```

```c++
const auto J = blockDim.x*blockIdx.x + threadIdx.x;
const auto j = blockDim.y*blockIdx.y + threadIdx.y;
```

이 두 줄은 **2차원 블록과 2차원 스레드 구조**에서 각각의 스레드가 공분산 행렬의 `(j, J)` 위치를 담당하도록 하기 위해, 전역 인덱스를 계산하는 과정이다.

먼저 `J`를 보면,

- `blockDim.x` → 블록 한 줄(가로) 안에 있는 스레드 개수
- `blockIdx.x` → 현재 블록이 그리드 전체에서 몇 번째 가로 줄(block column)인지
- `threadIdx.x` → 블록 내부에서 스레드가 가로로 몇 번째인지

따라서 **J는 “그리드 전체를 가로 방향으로 펼쳤을 때, 내가 몇 번째 쓰레드인지”에 해당한다.**

즉, `blockDim.x * blockIdx.x` 로 “이 블록이 시작하는 전역 x-좌표”가 정해지고 거기에 `threadIdx.x` 를 더해서 블록 내부에서 자신의 위치를 반영하여 최종적으로 전체 좌표계에서 이 쓰레드의 **열 인덱스 J**가 된다.

`j` 역시 같은 원리다.

- `blockDim.y` → 블록의 세로 방향 스레드 개수
- `blockIdx.y` → 그리드에서 이 블록이 세로로 몇 번째인지
- `threadIdx.y` → 블록 내부에서 스레드의 세로 위치

그래서 **j는 공분산 행렬에서 스레드가 담당하는 행(row)의 전역 인덱스가 된다.**

```c++
// CUDA kernel performing naive covariance matrix computation
template <typename index_t, typename value_t> __global__
void covariance_kernel(
    value_t * Data,    // centered data matrix
    value_t * Cov,     // covariance matrix
    index_t num_entries,   // number of images (m)
    index_t num_features)  // number of pixels (n)
{
    // determine row and column indices of Cov (j and j' <-> J)
    const auto J = blockDim.x*blockIdx.x + threadIdx.x;
    const auto j = blockDim.y*blockIdx.y + threadIdx.y;

    if (j < num_features && J < num_features) { // check range of indices
        value_t accum = 0; // store scalar product in a register
        
        // accumulate contribution over images (entry <-> i)
        for (index_t entry = 0; entry < num_entries; entry++)
            accum += Data[entry*num_features + j] * Data[entry*num_features + J];

        Cov[j*num_features + J] = accum / num_entries;
    }
}
```

이 커널은 **공분산 행렬 $C$의 한 원소 $C_{jJ}$ 를 하나의 스레드가 계산하는 구조다.  
그래서 스레드마다 `(j, J)`라는 두 개의 인덱스를 갖게 되는데, 이걸 위해 **2차원 그리드/블록** 을 사용한다.

- `J = blockDim.x * blockIdx.x + threadIdx.x`  
    → 공분산 행렬에서 **열 인덱스** 역할
    
- `j = blockDim.y * blockIdx.y + threadIdx.y`  
    → 공분산 행렬에서 **행 인덱스** 역할
    

즉, 공분산 행렬 (Cov)의 `(j, J)` 위치 하나가 스레드 하나에 대응된다.  
그래서 전체 그리드를 2D로 깔면, 각 스레드가 행렬 요소 하나씩을 맡아 채우는 셈이다.


```c++
if (j < num_features && J < num_features)
```

쓰레드 수가 `num_features × num_features`보다 조금 더 많을 수도 있기 때문에, 인덱스가 범위를 넘지 않도록 막아주는 안전장치다.

그 안에서 실제 계산은 다음 반복문이 담당한다.

```c++
for (index_t entry = 0; entry < num_entries; entry++)
    accum += Data[entry*num_features + j] * Data[entry*num_features + J];
```

여기서 `entry`는 이미지 인덱스 (i)에 해당하고,  
`Data[entry*num_features + j]`는 “i번째 이미지의 j번째 픽셀 값”,  
`Data[entry*num_features + J]`는 “i번째 이미지의 J번째 픽셀 값”이다.

즉, i를 0부터 m-1까지 돌면서

$$\sum_i \bar{v}_j^{(i)} \bar{v}_J^{(i)}  $$
를 그대로 구현한 것이다. 

그리고 마지막에

```c++
Cov[j*num_features + J] = accum / num_entries;
```

를 통해 이 합을 `num_entries = m`으로 나눠서 $C_{jJ} = \frac{1}{m} \sum_i \bar{v}_j^{(i)} \bar{v}_J^{(i)}$를 저장한다.  
여기서도 `Cov`는 1차원 배열이므로, `(j, J)` 위치를 `j*num_features + J`라는 인덱스로 평탄화해서 접근하고 있다.

```c++
dim3 blocks(SDIV(rows*cols,8), SDIV(rows*cols,8));
dim3 threads(8,8);
covariance_kernel<<<blocks,threads>>>(Data,Cov,imgs,rows*cols);
```

하나의 block 안에는 **가로 8개(`threadIdx.x = 0~7`)**, **세로 8개(`threadIdx.y = 0~7`)**,  
즉 총 **64개의 스레드(8×8)** 가 배치된다. 각각의 스레드는 이 8×8 작은 타일에서 자신만의 좌표를 갖고, 공분산 행렬의 한 위치를 담당하게 된다.

그리고 전체 블록(grid)의 개수는 **전체 픽셀 수(rows * cols)를 8로 나눈 값**, 즉 `ceil((rows*cols)/8)` 만큼 가로 방향, 세로 방향으로 배치하여 2차원적으로 구성된다. 이렇게 하면 공분산 행렬의 전체 크기(n × n)를 커버하기 위해 필요한 블록 수가 정확히 만들어지고, 전체 스레드가 공분산 행렬의 모든 원소를 계산할 수 있게 된다.

---
## Sysmmetric Covariance Matrix Computation

공분산은 결국 같은 행렬 $D$와 그 전치 행렬 $D^T$를 곱해서 $C = \frac{1}{m} D^T D$ 를 구하는 것이라서, 결과로 나오는 공분산 행렬 $C$는 항상 **대칭 행렬**이 된다.  

즉, $C_{jJ} = C_{Jj}$ 이고, 위·아래 삼각형에 **똑같은 값이 두 번씩** 들어가는 구조다.  
그래서 굳이 모든 원소를 다 따로 계산하면 계산량을 절반은 그냥 버리는 셈이 된다.

이 대칭성을 이용한 커널이 바로 아래 코드다.

```c++
//CUDA kernel performing symmetric covariance matrix computation
template <typename index_t, typename value_t> __global__
void symmetric_covariance_kernel(
value_t * Data, value_t * Cov, index_t num_entries, index_t num_features) {
	
	// indices as before
	const auto J = blockDim.x*blockIdx.x + threadIdx.x;
	const auto j = blockDim.y*blockIdx.y + threadIdx.y;
	
	// execute only entries below the diagonal since C = C^T
	if (j < num_features && J <= j) {
		value_t accum = 0;
		for (index_t entry = 0; entry < num_entries; entry++)
			accum += Data[entry*num_features + j] * Data[entry*num_features + J];
		
		// exploit symmetry
		Cov[j*num_features + J] = Cov[J*num_features + j] = accum / num_entries;
	}
}
```

여기서 핵심은 바로 이 **조건문**이다.

```c++
if (j < num_features && J < num_features)
```

나이브한 버전에서는 `j`와 `J`가 각각 전체 픽셀 수(`num_features`) 범위 안에만 들어오면,  
공분산 행렬의 **모든 위치 (j, J)** 에 대해 값을 계산했다.  

즉, 대각선 위·아래를 가리지 않고 **행렬 전체를 다 채우는 방식**이었다.

반면, 대칭성을 이용한 버전에서는 조건을 이렇게 바꾼다.

```c++
if (j < num_features && J <= j)
```

이번에는 `j`와 `J`가 유효 범위 안에 있는 것뿐 아니라, **`J <= j`일 때만 실행**되도록 했다.

- `j` : 행 인덱스
- `J` : 열 인덱스

라고 보면, `J <= j`인 위치는 **대각선과 그 아래 삼각형 영역**에 해당한다.  

즉, 공분산 행렬의 **아랫부분(포함 대각)** 만 직접 계산하고, 위쪽 삼각형은 아예 루프를 돌지 않으므로 연산을 건너뛰게 된다.

연산 자체(누적하고 나눠주는 계산)는 나이브 버전과 완전히 동일하다. 다만 결과를 행렬에 쓸 때 이렇게 한 번에 처리한다.

```c++
Cov[j*num_features + J] = Cov[J*num_features + j] = accum / num_entries;
```

즉, `(j, J)` 위치에 값을 쓰고, 동시에 `(J, j)` 위치에도 똑같은 값을 복사해서 넣는다. 이렇게 해서 **한 번의 계산으로 대칭 위치 두 곳을 동시에 채우기 때문에**, 계산해야 할 (j, J) 쌍의 수가 거의 **절반으로 줄어드는 효과**를 얻을 수 있다.