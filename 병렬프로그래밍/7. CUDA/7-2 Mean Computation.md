첫 번째 CUDA 예제는 **여러 장의 이미지를 대상으로 각 픽셀의 평균값을 구하는 작업**이다.  
하나의 이미지는 가로 45픽셀, 세로 55픽셀로 이루어져 있으며, 전체 픽셀 수는 45 × 55 = 2,475개다. 이런 이미지가 **총 202,599장** 주어져 있다고 하자.

우리가 하고 싶은 일은 간단하다.  

![](../../images/Pasted%20image%2020251203194402.png)

각 이미지에서 **같은 위치에 있는 픽셀들끼리 값을 모두 더한 뒤, 그 평균을 구해 하나의 ‘평균 이미지’를 만드는 것**이다. 예를 들어, 모든 이미지의 (0,0) 위치 픽셀 값들을 모아 평균을 내고, (0,1) 위치도 같은 방식으로 평균을 내는 식이다. 이렇게 하면 최종적으로 원본과 동일한 크기(45×55)를 가지는 평균 이미지 한 장을 얻을 수 있다.

---

### Header
먼저 코드의 헤더 부분을 살펴보면 다음과 같은 선언을 확인할 수 있다.

```c
#include "../include/hpc_helpers.hpp" // timers, error macros
#include "../include/bitmap_IO.hpp" // write images
#include "../include/binary_IO.hpp" // load data
template <
	typename index_t, // data type for indices
	typename value_t> __global__ // data type for values
	void compute_mean_kernel(
	value_t * Data, // device pointer to data
	value_t * Mean, // device pointer to mean
	index_t num_entries, // number of images (m)
	index_t num_features); // number of pixels (n)
```

이 코드를 통해 가장 먼저 파악할 수 있는 점은 `compute_mean_kernel` 함수가 **`__global__` 키워드로 선언된 CUDA 커널**이라는 사실이다.  

`__global__`은 **해당 함수가 CPU에서 호출되지만 실제 실행은 GPU에서 이루어진다**는 의미다. 즉, 이 함수 안의 모든 연산은 GPU의 스레드들에 의해 병렬적으로 수행된다.

---
### compute mean kernel
이제 실제 평균 이미지를 계산하는 `compute_mean_kernel` 함수가 어떻게 구현되어 있는지 살펴보자.

```c++
// CUDA kernel implementing the mean face computation
template <typename index_t, typename value_t> __global__
void compute_mean_kernel(
value_t * Data, value_t * Mean, index_t num_entries, index_t num_features) {
	// compute global thread identifier
	const auto thid = blockDim.x*blockIdx.x + threadIdx.x;
	
	// prevent memory access violations
	if (thid < num_features) {
	
		// accumulate in a fast register, not in slow global memory
		value_t accum = 0;
		
		for (index_t entry = 0; entry < num_entries; entry++)
			accum += Data[entry*num_features + thid];
		
		// write the register once to global memory
		Mean[thid] = accum / num_entries;
	}
}
```

커널이 시작되면 가장 먼저 각 쓰레드는  자신이 담당해야 할 픽셀이 무엇인지 결정해야 한다. 이를 위해 GPU에서는 **글로벌 스레드 ID(global thread id)** 를 계산하는데, 코드의 첫 줄에서 `thid`가 바로 이 역할을 한다. 계산식은 다음과 같다.

```c++
thid = blockDim.x * blockIdx.x + threadIdx.x
```

여기서 `blockDim.x`는 하나의 블록 안에 존재하는 쓰레드 개수이고, `blockIdx.x`는 현재 스레드가 속한 블록의 인덱스이며, `threadIdx.x`는 그 블록 내에서 자신의 위치를 가리킨다. 즉,

> 전체 Grid를 1차원으로 펼쳐보았을 때,  
> **현재 스레드가 전체 스레드 중 몇 번째인지 나타내는 값**이 `thid`이다.

이렇게 계산된 `thid`는 **곧 "내가 맡아야 할 픽셀의 인덱스"** 가 된다. 전체 픽셀 개수는 `num_features = rows * cols` 이므로, 조건문에서 `thid < num_features`를 체크하여 범위를 벗어난 스레드가 잘못된 접근을 하지 않도록 보호하고 있다.

각 스레드는 자신에게 배정된 픽셀 위치 하나를 기준으로 202,599장의 이미지를 순회한다. 이때 반복문에서는

`accum += Data[entry*num_features + thid];`

이와 같이 모든 이미지의 동일한 픽셀 위치 값들을 더해 누적 합을 만든다. 누적 과정에서 `accum`이 로컬 레지스터에 저장된다는 점도 중요하다. 레지스터는 GPU에서 가장 빠른 메모리이기 때문에, 글로벌 메모리에 직접 접근하는 것보다 훨씬 효율적으로 반복 연산을 수행할 수 있다.

마지막으로 모든 이미지를 다 더한 뒤, 스레드는 그 합을 이미지 개수로 나누어 평균을 구하고 이를 `Mean[thid]`에 기록한다. 여기서 `Mean`은 GPU global memory에 있는 결과 버퍼이며, 모든 스레드가 병렬로 서로 다른 픽셀에 대한 평균값을 채워 넣게 된다.


### 효율적인가?
`Data` 배열 안에는 모든 이미지의 픽셀이 일렬로 저장되어 있다. 한 이미지는 `num_features = rows * cols`개의 픽셀을 가지고 있고, 그 다음 이미지도 같은 길이만큼 바로 이어서 붙어 있는 구조다.

커널에서 한 스레드는 하나의 픽셀 인덱스 `thid`를 맡고, 루프를 돌면서

```c++
accum += Data[entry * num_features + thid];
```
와 같이 **모든 이미지의 같은 위치 픽셀**을 차례대로 읽어온다. 

![](../../images/Pasted%20image%2020251203201539.png)

이걸 한 쓰레드만 놓고 보면, 빨간 동그라미처럼 메모리에서 일정 간격(num_features)씩 떨어진 값들을 계속 건너뛰면서 읽는 “스트라이드 접근”이 된다. 겉으로만 보면 캐시 효율이 안 좋아 보이고, 매번 멀리 떨어진 주소를 들락날락하니 비효율적일 것처럼 느껴질 수 있다.

그런데 실제로는 CUDA가 **warp 단위(보통 32개의 스레드)** 로 명령을 실행한다는 점이 중요하다. 같은 warp 안의 쓰레드들은  같은 `entry`에 대해 동시에 실행되므로, 그 순간 이들이 접근하는 주소는

```c++
entry * num_features + 0,
entry * num_features + 1,
entry * num_features + 2,
...
entry * num_features + 31
```

처럼 서로 인접한 값들이 된다. 즉, 파란색으로 표시한 것처럼 **연속된 메모리 구간을 한 번에 읽는 “coalesced access” 패턴**이 만들어진다. 글로벌 메모리는 보통 일정한 크기의 덩어리(여러 바이트)를 한 번에 가져오는데, 이렇게 warp에 속한 스레드들이 나란한 주소를 요구하면, 한두 번의 메모리 트랜잭션으로 32개 스레드의 데이터를 한꺼번에 가져올 수 있다.

정리하면, 개별 스레드만 떼어 놓고 보면 멀리 떨어진 주소를 건너뛰면서 접근하는 것처럼 보이지만, **warp 기준으로 보면 같은 시점에 읽는 데이터가 서로 연속적이기 때문에 오히려 메모리 대역폭을 효율적으로 쓰는 구조**다. 그래서 이 커널의 메모리 접근 패턴은 처음 인상과 달리 충분히 성능이 잘 나올 수 있는 형태라고 볼 수 있다.

---
### Main function
다음은 위의 헤더를 구현한 메인 함수이다.

```c
int main(int argc, char * argv[]) {
	cudaSetDevice(0);
	
	// 202599 grayscale images each of shape 55 x 45
	constexpr uint64_t imgs = 202599, rows = 55, cols = 45;
	
	// pointer for data matrix and mean vector on CPU
	float *data = nullptr, *mean = nullptr;
	cudaMallocHost(&data, sizeof(float)*imgs*rows*cols);
	cudaMallocHost(&mean, sizeof(float)*rows*cols);
	
	// allocate storage on GPU
	float *Data = nullptr, *Mean = nullptr;
	cudaMalloc(&Data, sizeof(float)*imgs*rows*cols);
	cudaMalloc(&Mean, sizeof(float)*rows*cols);
	
	// load data matrix from disk
	std::string file_name = "./data/celebA_gray_lowres.202599_55_45_32.bin";
	load_binary(data, imgs*rows*cols, file_name);
	
	// Memory transfers and kernel invocation
	// Copy data to device and reset Mean
	cudaMemcpy(Data, data, sizeof(float)*imgs*rows*cols, H2D);
	cudaMemset(Mean, 0, sizeof(float)*rows*cols);
	
	// Compute mean
	compute_mean_kernel <<<SDIV(rows*cols,32), 32 >>>(Data, Mean, imgs, rows*cols);
	
	// Transfer mean back to host
	cudaMemcpy(mean, Mean, sizeof(float)*rows*cols, D2H);
	
	// Writing the result and freeing memory
	dump_bitmap(mean, rows, cols, "./imgs/celebA_mean.bmp");
	cudaFreeHost(data); cudaFreeHost(mean); // get rid of the memory
	cudaFree(Data); cudaFree(Mean);
}
```

프로그램은 먼저 CPU와 GPU 양쪽에 이미지 데이터를 저장할 공간을 준비한다. 아래 코드에서 볼 수 있듯이, `cudaMallocHost`를 사용해 CPU(Host) 메모리에 `data`와 `mean` 버퍼를 할당하고 있다. 이때 `data`는 전체 이미지(202,599장 × 2,475픽셀)를 모두 저장할 공간이며, `mean`은 최종 평균 이미지를 담기 위한 배열이다.

```c++
	// pointer for data matrix and mean vector on CPU
	float *data = nullptr, *mean = nullptr;
	cudaMallocHost(&data, sizeof(float)*imgs*rows*cols);
	cudaMallocHost(&mean, sizeof(float)*rows*cols);
	
	// allocate storage on GPU
	float *Data = nullptr, *Mean = nullptr;
	cudaMalloc(&Data, sizeof(float)*imgs*rows*cols);
	cudaMalloc(&Mean, sizeof(float)*rows*cols);
```

여기서 `cudaMallocHost`를 사용한 이유는 CPU 메모리를 페이지 락(pinned) 상태로 확보해 GPU와의 전송 속도를 높이기 위함이다. 이어서 같은 크기의 메모리를 `cudaMalloc`으로 GPU(Device) 쪽에도 할당하는데, 이는 앞으로 CUDA 커널이 실제 연산을 수행할 공간이다.

메모리 준비가 끝나면 CPU는 디스크에 저장된 이미지 파일을 읽어와, 앞서 확보해 둔 Host 메모리(`data`)에 전체 이미지를 일렬로 로딩한다.

```c++
	// load data matrix from disk
	std::string file_name = "./data/celebA_gray_lowres.202599_55_45_32.bin";
	load_binary(data, imgs*rows*cols, file_name);
```

이 과정에서 디스크 → CPU 메모리로 이미지가 들어오게 된다.

```c++
	// Memory transfers and kernel invocation
	// Copy data to device and reset Mean
	cudaMemcpy(Data, data, sizeof(float)*imgs*rows*cols, H2D);
	cudaMemset(Mean, 0, sizeof(float)*rows*cols);
```

그 다음 단계에서는 CPU에 적재된 데이터를 GPU 메모리로 옮기고, 결과가 저장될 `Mean` 버퍼를 0으로 초기화하여 커널 실행을 위한 준비를 완료한다.

`cudaMemcpy`는 Host에서 Device로의 복사를 수행하며(매크로 `H2D`), 이렇게 GPU 메모리가 이미지로 채워지면 커널이 병렬 연산을 수행할 준비가 된다. 또한 `cudaMemset`으로 평균 결과 버퍼를 0으로 만들어 두는 것은, 이후 커널이 각 스레드마다 누적합을 더해 나가기 위한 초기 상태를 설정하는 과정이라고 볼 수 있다.

이제 Host에서 모든 준비를 마친 뒤, 실제 연산을 수행할 CUDA 커널을 호출한다. 커널 호출부는 다음과 같다.

```c
compute_mean_kernel <<<SDIV(rows*cols,32), 32 >>>(Data, Mean, imgs, rows*cols);
```

여기서 `compute_mean_kernel`은 앞서 `__global__`로 정의했던 GPU 전용 함수이며, `rows * cols` 즉 전체 픽셀 수를 32로 나눈 값을 블록 개수로 사용하고, 각 블록당 32개의 스레드를 배치하도록 설정하고 있다. 이 구성은 결국 전체 스레드 수가 `rows * cols` 이상 되도록 배치하여, **각 픽셀 위치마다 정확히 하나의 스레드가 담당하도록 하기 위한 구조**이다. 즉, 이미지의 한 픽셀 위치를 하나의 스레드가 맡아서 202,599장의 이미지에서 해당 위치의 픽셀 값을 모두 더하고 평균을 내는 방식이다. 

커널 실행이 끝나면 평균 계산 결과는 GPU 메모리의 `Mean` 버퍼에 저장되어 있다. 하지만 이 값은 GPU의 VRAM에 위치하므로, 프로그램이 실제로 결과를 활용하거나 파일로 저장하기 위해서는 다시 CPU 메모리로 가져와야 한다. 이를 위해 다음과 같이 `cudaMemcpy`를 사용해 Device에서 Host로 데이터를 복사한다.

```c++
	// Transfer mean back to host
	cudaMemcpy(mean, Mean, sizeof(float)*rows*cols, D2H);
```


여기서 사용된 매크로 `D2H`는 Device to Host를 의미하며, GPU에서 CPU로 데이터를 옮기기 위한 모드이다. 이 한 줄을 통해 GPU에서 계산된 평균 이미지가 Host 메모리의 `mean` 배열에 전달되고, 이제 CPU는 이 값을 기반으로 후처리나 파일 저장을 수행할 수 있게 된다.

이제 결과를 실제 이미지 파일로 출력하고, 프로그램이 할당했던 모든 메모리를 정리하는 단계가 이어진다.


```c++
	// Writing the result and freeing memory
	dump_bitmap(mean, rows, cols, "./imgs/celebA_mean.bmp");
	cudaFreeHost(data); cudaFreeHost(mean); // get rid of the memory
	cudaFree(Data); cudaFree(Mean);
```

`dump_bitmap`은 Host 메모리에 있는 평균 이미지를 BMP 파일 형식으로 저장하는 함수이다. 이후 메모리 정리를 위해 Host에서 `cudaMallocHost`로 할당했던 `data`와 `mean`은 `cudaFreeHost`로 해제하고, Device 쪽에서 `cudaMalloc`으로 확보했던 `Data`와 `Mean`은 `cudaFree`로 각각 해제한다. 이렇게 Host와 Device 양쪽에서 사용한 메모리를 모두 정리해줌으로써 프로그램은 실행 과정에서 할당했던 모든 자원을 깨끗하게 반환하게 된다.

---
### 결과 분석

![](../../images/Pasted%20image%2020251203202251.png)

실행 시간을 보면, 먼저 **CPU에서 GPU로 데이터를 전송하는 데 약 170.136ms**가 소요된다. 이는 전체 이미지 데이터가 매우 크기 때문에 가장 시간이 많이 걸리는 단계다. 그 다음 실제 GPU가 평균을 계산하는 **커널 실행 시간은 약 8.82ms**이며, 마지막으로 GPU에서 계산된 평균 이미지를 다시 **CPU로 가져오는 시간은 약 0.03ms**로 매우 짧다.

커널이 하는 일은 단순하다. 모든 픽셀 값을 정확히 한 번씩 읽고 그때마다 한 번의 덧셈을 수행해 전체 합을 만든 뒤 평균을 내는 과정이다. 이미지가 약 20만 장이고 픽셀이 약 5억 개 정도 되기 때문에, 이 모든 연산을 9ms 안에 처리했다는 것은 GPU가 상당히 높은 처리량을 보였다는 의미다. 이 시간을 기준으로 계산하면, GPU는 초당 약 208GB 정도의 글로벌 메모리 대역폭을 사용한 셈이며, 연산량으로 환산하면 약 52GFLOPS 수준의 성능을 낸 것이다.