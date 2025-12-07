이번 예시는 앞에서 계산해둔 평균 이미지를 활용하여, **모든 이미지의 각 픽셀에서 해당 픽셀의 평균값을 빼주는(mean correction)** 과정이다. 즉, 데이터 전체를 “평균이 0이 되도록 정규화(zero-centering)”하는 단계라고 볼 수 있다. 

![](../../images/Pasted%20image%2020251203221001.png)

이를 CUDA에서 구현한 커널이 아래 코드이다.

```c++
template <typename index_t, typename value_t> __global__
void correction_kernel(value_t * Data, value_t * Mean, index_t num_entries, index_t num_features) {
	const auto thid = blockDim.x*blockIdx.x + threadIdx.x;
		if (thid < num_features) {
		const value_t value = Mean[thid];
	for (index_t entry = 0; entry < num_entries; entry++)
		Data[entry*num_features + thid] -= value;
	}
}
```

커널의 구조는 앞서 평균을 계산했던 커널과 거의 동일하다. 스레드는 `thid`를 이용하여 **하나의 픽셀 위치**를 담당하고, 그 픽셀에 해당하는 평균값을 `value = Mean[thid]`로 레지스터에 저장한다. 이후 `entry` 반복문을 돌면서 전체 이미지에 걸쳐 같은 위치의 픽셀 값을 하나씩 불러와 그 평균값을 빼준다.

즉, 전체 202,599장의 이미지에 대해

- 첫 번째 스레드는 모든 이미지의 (0번 픽셀) 값을 평균만큼 감소시키고
    
- 두 번째 스레드는 (1번 픽셀)을 수정하고
    
- …
    
- 마지막 스레드는 (마지막 픽셀)을 수정한다
    
```c++
corretion_kernel<<<SDIV(rows*cols, 32),32>>>(Data, Mean, imgs, rows*cols)
```

이 방식으로 모든 픽셀의 “평균 제거(mean centering)”가 병렬로 수행되는 구조이다. GPU는 스레드 수만큼 픽셀 위치를 동시에 처리하므로, 이 연산 또한 CPU 기반 반복문에 비해 매우 빠르게 수행된다.

---
앞선 예시에서는 쓰레드 하나가 **픽셀 인덱스(thid) 하나**를 담당하여, 모든 이미지의 해당 픽셀에서 평균값을 빼주는 방식이었다. 하지만 이번 버전의 커널은 병렬화 방향을 바꾸어, **이미지 단위로 병렬화**를 수행한다. 즉, 쓰레드 하나가 이미지 한 장 전체를 담당하는 방식이다. CUDA 커널은 아래와 같다.

```c++
template < typename index_t, typename value_t> __global__
void correction_kernel_ortho(value_t * Data, value_t * Mean, index_t num_entries, index_t num_features) {
	const auto thid = blockDim.x*blockIdx.x + threadIdx.x;
	
		if (thid < num_entries) {
		for (index_t feat = 0; feat < num_features; feat++)
			Data[thid*num_features + feat] -= Mean[feat];
	}
}
```

여기서 `thid`는 **이미지 번호**(0번 이미지, 1번 이미지 …)를 뜻한다. 즉 스레드 하나가 이미지 한 장을 맡는다. 그리고 내부 반복문에서 `feat`를 증가시키며 해당 이미지의 모든 픽셀을 순회한 다음, 그 픽셀 값에서 미리 계산한 평균 이미지의 동일 위치 값 `Mean[feat]`을 빼준다.

이 커널의 흐름을 정리하면 다음과 같다.

- 쓰레드 ID(`thid`)는 이미지 인덱스와 대응된다.
    
- 쓰레드 하나는 **이미지 한 장 전체(2,475개 픽셀)** 를 처리한다.
    
- 내부 반복에서 모든 픽셀을 순회하면서 `Data[thid * num_features + feat] -= Mean[feat]` 를 수행하여 평균 값을 빼준다.
    
- 결과적으로 전체 이미지가 **한 번의 커널 실행으로 모두 zero-centering 처리**된다.
    
```c++
correction_kernel_ortho<<<SDIV(images, 32),32>>>(Data, Mean, imgs, rows*cols)
```

이 방식은 앞선 버전(픽셀 기준 병렬화)과는 반대로, **이미지 기준 병렬화**를 선택한 것이고, 데이터 접근 패턴 또한 더 연속적인 형태가 되므로 어떤 GPU에서는 이 방식이 더 효율적일 수도 있다.

---
## Coalesced Memory Access

위의 두 방식이 메모리 접근을 어떻게 하고 있는지 살펴보자

먼저 **thread 하나가 픽셀 하나를 담당하는 경우**를 생각해보면, 커널 내부의 루프는 다음과 같다.
```c++
for (index_t entry = 0; entry < num_entries; entry++)
	Data[entry*num_features + thid] -= value;
```

여기서 `thid`는 픽셀 인덱스를 의미하고, `entry`는 이미지 인덱스를 의미한다. 따라서 이 코드는 “여러 이미지에 대해, 항상 같은 위치의 픽셀을 하나의 스레드가 순회하면서 평균 값을 빼주는” 형태가 된다.

![](../../images/Pasted%20image%2020251203222328.png)

이걸 쓰레드 하나만 놓고 보면, 메모리에서 `num_features` 간격으로 멀찍이 떨어진 위치들을 계속 접근하고 있으니, 언뜻 보기에는 메모리 접근 효율이 좋지 않아 보인다. 일종의 “뛰엄뛰엄(strided) 접근"처럼 보이기 때문이다.

하지만 실제 실행 단위를 CUDA 관점에서 다시 보면 상황이 달라진다. 하나의 SM에서는 **warp 단위(32개 스레드)** 로 명령을 실행한다. 즉, thread 0, 1, 2, …, 31이 같은 시점에 같은 명령을 수행하게 된다. 이때 모든 스레드가 같은 `entry`에 대해 위 코드를 실행한다고 하면, 이들이 접근하는 주소는

```text
Data[entry*num_features + 0]
Data[entry*num_features + 1]
Data[entry*num_features + 2]
...
Data[entry*num_features + 31]
```
처럼 **연속된 인덱스 구간**이 된다. 결국 warp 기준으로 보면, 서로 인접한 픽셀들을 한 번에 읽고/쓰는 형태가 되기 때문에, 글로벌 메모리 입장에서는 **coalesced access(연속 접근)** 이 이루어진다.

두 번째는 하나의 thread가 하나의 이미지를 담당하는 경우이다.

```c
for (index_t feat = 0; feat < num_features; feat++)
	Data[thid*num_features + feat] -= Mean[feat];
```
![](../../images/Pasted%20image%2020251203223012.png)

이 방식에서는 쓰레드 하나가 이미지 전체의 픽셀을 순회하며 평균값을 빼주기 때문에, 스레드 단독으로만 보면 데이터가 **연속적인 메모리 구간**을 차례대로 읽게 된다. 즉, 한 스레드 입장에서 보면 이 방식이 훨씬 효율적으로 보일 수 있다.

하지만 실제 GPU에서는 스레드가 개별적으로 실행되는 것이 아니라 **warp(32개 스레드) 단위로 동시에 실행**된다. 따라서 스레드 하나만 보고 판단하면 GPU의 실제 메모리 접근 패턴을 잘못 이해하게 된다.

아래를 생각해보자:

- thread 0 → 이미지 0의 모든 픽셀을 순서대로 접근
    
- thread 1 → 이미지 1의 모든 픽셀을 순서대로 접근
    
- thread 2 → 이미지 2의 모든 픽셀을 순서대로 접근
    
- … 이런 식으로 스레드 하나당 이미지 하나씩 배정됨
    

그런데 warp 내부에서는 **이 스레드들이 같은 명령을 동시에 실행**하게 되므로, 실제 메모리 접근은 다음과 같이 나타난다.


```c
Data[0번 이미지의 첫 픽셀],
Data[1번 이미지의 첫 픽셀],
Data[2번 이미지의 첫 픽셀],
...
Data[31번 이미지의 첫 픽셀]
```

즉, **서로 완전히 떨어진 위치에 있는 데이터**를 warp 안의 쓰레드들이 동시에 접근하게 된다. 이렇게 되면 메모리 접근이 인접한 주소를 묶어서 가져오는 coalescing이 깨지고, 메모리 트랜잭션이 여러 번 나뉘어 불필요한 데이터까지 로드하게 된다.

결과적으로는 다음과 같은 비효율이 발생한다.

> 쓰레드 하나로 보면 연속 접근이라 좋아 보이지만, warp 전체를 보면  
> **32개의 서로 멀리 떨어진 이미지 구간을 동시에 읽어야 하므로 오히려 비효율적이다.


- **픽셀 기준 병렬화(스레드 = 픽셀)** : warp 단위에서는 연속된 주소로 접근하므로 매우 효율적
    
- **이미지 기준 병렬화(스레드 = 이미지)** : warp 단위에서는 멀리 떨어진 주소로 접근하므로 비효율적

이라는 결론이 나온다.

---
## The Tale of two Memory Access Patterns

두 방법의 메모리 접근을 시각화한 이미지이다. 

![](../../images/Pasted%20image%2020251203223354.png)

왼쪽 그림은 **픽셀 단위 병렬화(thid = pixel index)** 방식으로, warp 내부의 스레드들이 서로 인접한 픽셀 인덱스를 담당하기 때문에 메모리에서도 **연속된 주소를 한 번에 읽을 수 있는(coalesced memory access)** 형태가 된다. 즉, warp의 32개 스레드가 같은 `entry`에 대해 동시에 실행될 때, `Data[entry*num_features + thid]`는 연속된 메모리 구간을 접근하게 되므로 메모리 대역폭을 가장 효율적으로 사용할 수 있다.

반면 오른쪽 그림은 **이미지 단위 병렬화(thid = image index)** 방식이다. 이 경우 warp의 스레드들은 이미지 0, 이미지 1, 이미지 2 … 처럼 서로 멀리 떨어진 메모리 구역을 동시에 접근하게 된다. 따라서 실제로는 각 이미지의 첫 번째 픽셀만 필요한데, GPU는 주변의 연속된 메모리까지 함께 불러오기 때문에 사용되지 않는 데이터가 섞여 들어오고, 메모리 트랜잭션도 여러 번 발생해 **non-coalesced memory access**, 즉 비효율적인 접근이 되어버린다.