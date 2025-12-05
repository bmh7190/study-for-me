여기서는 시계열(time series) 데이터를 예시로 활용해보려고 한다. 시계열 데이터란 말 그대로 시간의 흐름에 따라 변화하는 값을 기록한 데이터로, 음성 신호, 센서 기록, 주가, 생체신호 등 다양한 분야에서 등장한다. 우리는 여러 개의 시계열 데이터가 주어졌을 때, **서로 얼마나 유사한지 비교해야 하는 상황**을 가정해볼 것이다.

![](../../images/Pasted%20image%2020251204151653.png)

예를 들어 위 그림은 “exact”라는 단어를 미국인과 영국인이 발음한 음성 파형을 각각 시계열로 나타낸 것이다. 두 사람은 같은 단어를 발음했기 때문에 전체적인 파형의 모양은 비슷하다. 지금까지 이미지에서는 단순히 유클리디안 거리(Euclidean distance)를 이용해 두 데이터가 비슷한지 판별할 수 있었다. 이미지의 경우 각 픽셀이 고정된 위치를 갖고 있기 때문에 같은 위치의 값을 비교하면 의미가 있기 때문이다.

하지만 시계열 데이터는 상황이 다르다. 같은 파형이라도 **언제 피크가 발생했는지**, 즉 **시간이 얼마나 빨리 혹은 늦게 흐르는지**에 따라 모양이 어긋날 수 있다. 예를 들어 두 사람이 같은 단어를 말하더라도 한 사람은 조금 더 천천히 말하고, 다른 사람은 더 빠르게 말할 수 있다. 이럴 때 단순히 같은 시점(time index)의 값을 빼서 거리를 계산해버리면, 실제로는 유사한 두 신호가 서로 다르게 보일 수 있다.

## Dynamic Time Warping (DTW)
그래서 이번에는 이러한 시간 차이를 보정해가며 유사도를 계산할 수 있는 방법, 즉 **Dynamic Time Warping(DTW)** 을 적용해볼 것이다. 쉽게 말하면, DTW는 두 시계열의 특정 시점들이 꼭 일대일로 대응하지 않더라도, **시간 축을 자유롭게 늘였다 줄였다 하면서 가장 잘 맞는 정렬을 찾아주는 방법**이다. 다시 말해, “이 두 시계열이 서로 얼마나 비슷하게 생겼는가?”를 **시간의 왜곡을 허용한 상태에서 계산**할 수 있게 해주는 알고리즘이다.

DTW를 이해하기 위해서는 먼저, 두 시계열을 어떻게 “정렬”하는지를 그래프로 표현한 DAG(Directed Acyclic Graph) 구조를 살펴볼 필요가 있다. 아래 그림은 길이가 각각 4인 두 시계열에 대해 DTW 거리 계산 과정에서 등장하는 DAG의 한 예시이다. 이 그래프의 각 노드는 두 시계열의 특정 시점 조합 $(i,j)$을 나타내며, DTW는 이 노드들을 연결하는 경로 중 비용이 최소가 되는 경로를 찾는 과정이라고 볼 수 있다.

![](../../images/Pasted%20image%2020251204152921.png)


이 그래프에는 다음과 같은 핵심 제약 조건들이 존재한다.

1. **Continuity(연속성)**  
    노드 간 이동은 항상 한 칸씩만 가능하며, 이동 방향은 가로, 세로, 대각선의 세 가지뿐이다.  
    즉, $(i, j) \rightarrow (i+1, j), (i, j+1), (i+1, j+1)$ 형태로만 이동할 수 있다.  
    이를 통해 warping path가 불연속적으로 점프하지 않도록 제한한다.
    
2. **Monotony(단조성)**  
    이동할 때마다 적어도 하나의 인덱스가 증가해야 한다.즉, 시간축을 거꾸로 되돌아가거나 감소하는 방향으로 움직일 수 없다. 이는 시계열의 시간 흐름을 보존한다는 뜻이다.
    
3. **Bounding(경계 조건)**  
    최적의 warping path는 **왼쪽 위(0,0)** 에서 시작하여 **오른쪽 아래(n−1, m−1)** 에서 끝나야 한다. 다시 말해, 두 시계열의 처음과 끝이 반드시 서로 매칭되어야 한다.
    

이 세 가지 조건이 만족되는 경로들 중에서, 각 노드에서의 시계열 간 “거리”(예: |x[i] − y[j]|)를 누적했을 때 총 비용이 가장 작은 경로가 바로 **DTW가 말하는 최적 warping path**이다.  

그 경로 위의 노드들이 두 시계열이 시간 왜곡을 허용한 상태에서 어떻게 서로 “최적으로 정렬되는지”를 보여준다.

---
![](../../images/Pasted%20image%2020251204153822.png)

DTW 계산은 크게 두 단계로 나눌 수 있다. 먼저, 두 시계열의 모든 시점 쌍을 서로 비교하여 **local cost**, 즉 해당 지점 간의 거리(weight)를 계산한다. 이 단계는 일종의 weighting function을 적용하는 과정이라고 볼 수 있다. 가장 단순한 경우에는 두 시계열 값의 차이를 제곱한 값 $(x_i - y_j)^2$을 local cost로 사용한다. 즉, query 시계열과 subject 시계열의 각 포인트 간 유사도를 전체적으로 계산해 cost matrix를 채우는 것이다.

그 다음 단계에서는 이 local cost들을 기반으로, 시계열의 처음 지점에서 끝 지점까지 이동하는 **누적 비용**을 계산하여 최적 경로를 찾는다. 이때 DTW는 반드시 세 가지 조건, 즉 **연속성**, **단조성** , **경계 조건을 만족해야 한다. 이를 반영하면 현재 (i, j) 위치에서 다음 위치로 이동할 수 있는 방향은 오직 세 가지뿐이다:

- 오른쪽 방향 (i, j+1)
- 아래 방향 (i+1, j)
- 오른쪽 아래 대각선 (i+1, j+1)

즉, grid 상에서 “뒤로 가는” 것 또는 큰 점프는 허용되지 않는다.

누적 비용을 채울 때는 각 셀 (i, j)에 대해, local cost(i, j)에 더해 **왼쪽, 위쪽, 대각선 위** 이 세 셀 중 누적 비용이 가장 작은 값을 선택하면 된다. DTW는 동적 계획법(DP)을 사용하기 때문에, 이전까지의 비용이 이미 누적되어 있고, 현재 위치에서는 세 방향 중 가장 비용이 작은 경로를 확장하는 것이 전체 경로의 최적 해를 보장한다.

이 과정을 시작점인 (0,0)부터 끝점인 (n−1, m−1)까지 반복하면, 마지막 셀에 도달했을 때 저장된 값이 두 시계열 사이의 **최소 warping cost**, 즉 DTW 거리이다. 이 값이 가장 낮은 정렬 경로가 두 시계열이 시간 왜곡을 허용한 상태에서 가장 비슷하게 매칭되는 형태를 나타낸다.

---
## Dynamic Programming Formulation of DTW

이제 각 과정을 자세하게 알아보자.

![](../../images/Pasted%20image%2020251204154615.png)

왼쪽 그림은 DTW 알고리즘의 첫 번째 단계인 **weight matrix**, 즉 두 시계열의 각 시점에서 값을 빼고 제곱하여 얻은 local cost들을 모아놓은 행렬을 나타낸 것이다. 시계열 길이가 4라면 이 행렬은 4×4 크기가 된다. 그 다음 단계에서는 이 weight matrix를 기반으로 실제 최단 경로를 찾기 위한 **DP matrix(누적 비용 행렬)** 을 구성하는데, 이 DP 행렬의 크기가 4×4가 아닌 5×5인 이유는 경계 조건을 처리하기 위해 padding을 한 줄씩 추가하기 때문이다. 이렇게 하면 DP를 채우는 과정에서 인덱스 예외 처리를 하지 않아도 되고, 시작점과 초기값을 명확히 정의할 수 있다.

![](../../images/Pasted%20image%2020251204155115.png)

DP matrix를 채울 때는 DTW의 이동 규칙을 따라, 특정 칸에 도달하기 위해 가능한 세 가지 이전 위치—왼쪽, 위쪽, 왼쪽 위 대각선—중 누적 비용이 가장 작은 값에 현재 weight 값을 더해 적는다. 즉, 실제 이동 방향은 오른쪽, 아래, 대각선 아래로 “전진”하지만, 비용 계산은 그 반대 방향을 참조하여 “어디서 왔을 때 가장 비용이 적은지”를 판단하는 방식으로 이루어진다. 이 과정을 테이블 끝까지 반복하면, 마지막 셀(오른쪽 아래 모서리)에 기록된 값이 두 시계열 사이의 최소 누적 거리, 즉 DTW 거리가 된다.

---
## DTW: Naive Sequential Implementation

가장 먼저, 완전히 직관적인 방식으로 DTW를 구현한 **순차(sequential) 버전** 코드부터 보자.

### 전체 코드

```c
template <typename index_t, typename value_t> __host__
value_t plain_dtw(value_t * query, value_t * subject, index_t num_features) {
	const index_t lane = num_features+1;
	value_t * penalty = new value_t[lane*lane];
	
	for (index_t index = 1; index < lane; index++) // initialize the matrix M
		penalty[index] = penalty[index*lane] = INFINITY;
		
	penalty[0] = 0;
	
	for (index_t row=1; row<lane; row++) { // traverse graph in row-major order
		const value_t q_value = query[row-1];
		for (index_t col=1; col<lane; col++) {
			// determine contribution from incoming edges
			const value_t diag = penalty[(row-1)*lane+col-1];
			const value_t abve = penalty[(row-1)*lane+col+0];
			const value_t left = penalty[(row+0)*lane+col-1];
			// compute residue between query and subject
			const value_t residue = q_value - subject[col-1];
			// relax node by greedily picking minimum edge
			penalty[row*lane + col] = residue*residue + min(diag, min(abve, left));
		}
	}
	
	const value_t result = penalty[lane*lane-1]; // report the lower right cell
	delete[] penalty;
	return result;
}
```

---

### 1) DP 행렬 크기 설정

```c
const index_t lane = num_features+1;
value_t * penalty = new value_t[lane*lane];
```

먼저 `lane`이라는 변수를 `num_features + 1`로 잡고 있다.  
여기서 `num_features`는 시계열의 길이(예: 4라면 4개 포인트)이고, 우리는 DTW를 위해 **padding을 한 줄 추가한 (n+1)×(n+1) DP 행렬**을 만들고 싶다. 그래서 가로·세로 길이를 모두 `num_features + 1`로 잡아서, 전체 `lane * lane` 크기의 `penalty` 배열을 동적으로 할당한다.

이 `penalty` 행렬이 곧 우리가 말했던 **DP matrix**, 즉 누적 비용을 저장하는 행렬이다. 여기 안에 weight 정보와 최단 경로를 따라왔을 때의 누적 비용이 모두 녹아들게 된다.

---

### 2) 테두리 초기화

```c++
for (index_t index = 1; index < lane; index++) // initialize the matrix M
	penalty[index] = penalty[index*lane] = INFINITY;

penalty[0] = 0;
```

그 다음으로는 DP 행렬의 **테두리를 초기화**하는 단계다.

`index`를 1부터 `lane-1`까지 돌면서,

- `penalty[index]` → 첫 번째 행(0번째 row)의 나머지 칸들
    
- `penalty[index*lane]` → 첫 번째 열(0번째 col)의 나머지 칸들
    

을 모두 `INFINITY`로 채워 넣는다. 이렇게 하면 DP 테이블의 첫 행과 첫 열(0번 row, 0번 col)은 전부 “갈 수 없는 길”처럼 아주 큰 값으로 채워진다.

그리고 딱 하나 예외가 되는 위치가

```c++
penalty[0] = 0;
```

여기다.  

왼쪽 위 모서리 `(0,0)`은 **시작점**이기 때문에, 누적 비용을 0으로 설정해둔다.  
그래서 나중에 DP를 채울 때, `(1,1)`은 `(0,0)`에서만 진입할 수 있고, 그 비용 위에 local cost만 더해지게 된다.

즉, 이 초기화 과정을 통해서 시작점은 0이 되고, 첫 행, 첫 열은 무한대로 채워서 사실상 “벽”이다,

---

### 3) DP 테이블 채우기 (row-major 순서로 순회)

이제 본격적으로 `penalty` 행렬을 채우는 단계다.

```c++
for (index_t row=1; row<lane; row++) { // traverse graph in row-major order
	const value_t q_value = query[row-1];
	for (index_t col=1; col<lane; col++) {
		// determine contribution from incoming edges
		const value_t diag = penalty[(row-1)*lane+col-1];
		const value_t abve = penalty[(row-1)*lane+col+0];
		const value_t left = penalty[(row+0)*lane+col-1];
		// compute residue between query and subject
		const value_t residue = q_value - subject[col-1];
		// relax node by greedily picking minimum edge
		penalty[row*lane + col] = residue*residue + min(diag, min(abve, left));
	}
}
```

바깥쪽 `row` 루프는 1행부터 마지막 행까지, 안쪽 `col` 루프는 1열부터 마지막 열까지,  
즉 (1,1)에서 (n,n)까지 **행 우선(row-major)** 순서로 DP 테이블을 채운다.

`row`가 1 이상이므로, `query[row-1]`는 실제 query 시계열의 `row-1`번째 원소에 해당한다.

```c++
const value_t q_value = query[row-1];
```

그리고 각 칸 `(row, col)`에서, 이전 누적 비용 후보 3개를 꺼내온다:

```c++
const value_t diag = penalty[(row-1)*lane+col-1];
const value_t abve = penalty[(row-1)*lane+col+0];
const value_t left = penalty[(row+0)*lane+col-1];
```

- `diag` : 왼쪽 위 대각선 `(row-1, col-1)`에서 온 경우
    
- `abve` : 바로 위 `(row-1, col)`에서 온 경우
    
- `left` : 바로 왼쪽 `(row, col-1)`에서 온 경우
    

이게 바로 DTW에서 말한 세 가지 이동 방향(↖, ↑, ←)에 대응된다.

그 다음, 현재 위치의 local cost를 계산한다.

```c++
const value_t residue = q_value - subject[col-1];
```

이는 query의 `row-1`번째 값과 subject의 `col-1`번째 값의 차이를 나타낸다.  
실제 DP에 들어가는 local cost는 이 차이를 제곱한 값이다.

마지막으로, 세 방향 중 최소 누적 비용을 선택하고, 현재 local cost를 더해 새 값을 만든다.

```c++
penalty[row*lane + col] = residue*residue + min(diag, min(abve, left));
```

> `현재 칸 = (query와 subject의 차이 제곱) + (대각선/위/왼쪽 중 최소 누적 비용)`

이라는 DTW DP 공식이 그대로 코드에 구현된 것이다.

---

### 4) 최종 답 꺼내기

```c++
const value_t result = penalty[lane*lane-1]; // report the lower right cell
delete[] penalty;
return result;
```

`lane*lane-1`은 1차원 배열로 평탄화된 `penalty`에서 **맨 마지막 원소**,  
즉, 2D 관점에서는 `(lane-1, lane-1)` → (n, n) 위치에 해당한다.

여기에는 시작점 `(0,0)`에서 끝점 `(n,n)`까지  
DTW 제약(연속성/단조성/경계 조건)을 모두 만족하는 경로들 중  
**누적 비용이 최소인 값**, 즉 **DTW 거리**가 저장되어 있다.

마지막으로 동적 할당한 `penalty`를 `delete[]`로 해제해주고, 이 값을 결과로 반환하면 naive sequential DTW 구현이 끝난다.

---

## DTW: Naive Multi threaded OpenMP

다음은 이 코드르를 멀티쓰레드로 나타낼건데 사용될 것은 바로 전에 다뤘던 openmp이다. 

### host_dtw
```c
#include <omp.h>
template <
typename index_t, typename value_t> __host__
void host_dtw(
	value_t * query, // pointer to query
	value_t * subject, // pointer to data matrix
	value_t * dist, // pointer to distance array
	index_t num_entries, // number of entries (m)
	index_t num_features) { // number of time ticks (n)
	
	# pragma omp parallel for
	for (index_t entry=0; entry<num_entries; entry++) {
		const index_t off = entry*num_features;
		dist[entry] = optimized_dtw(query, subject+off, num_features);
	}
}
```

먼저 입력값을 보면, `num_entries`는 **subject 시계열 데이터의 총 개수**라고 이해하면 된다. 즉, 각각의 entry가 하나의 독립된 시계열 데이터 셋이다. 반면 `query`는 이 모든 subject와 비교될 **하나의 기준 시계열**이다. 따라서 전체 구조는 “하나의 query에 대해 수만 개의 subject를 각각 DTW로 비교하는” 형태가 된다. 또 `num_features`는 **시계열 하나가 갖고 있는 시간 축의 길이**, 즉 각 시계열 데이터의 총 time tick 개수다.

이제 OpenMP를 적용한 `host_dtw`는 뒤에서 구현할 `optimized_dtw()` 함수를 병렬로 실행하는데, 병렬화의 단위는 바로 `num_entries`이다. 즉, 한 개의 subject와 DTW를 계산하는 작업은 각각 독립적이므로 서로 간섭이 없다. 따라서 OpenMP는 전체 `num_entries`를 스레드들 사이에서 _block distribution_ 방식으로 나누어 처리한다. 쉽게 말하면, 여러 개의 CPU 스레드가 20만 개 정도 되는 시계열 데이터들을 나눠서 맡아서 실행하는 것이다.

## optimized_dtw

여기서는 맨 위에서 설명한 방식과 달리, 메모리 사용을 줄이기 위한 최적화를 하나 적용했다.  
생각해 보면 DP 테이블에서 어떤 한 지점을 계산할 때 필요한 값은 **좌상단, 왼쪽, 위쪽** 이렇게 세 방향뿐이다. 이 세 값의 위치 관계를 잘 보면, 사실 전체 행이 아니라 **위 행과 현재 행, 즉 2개 행만 있으면 계산이 가능하다.**

그래서 DP 테이블 전체를 저장하지 않고, **오직 두 줄만 유지**하면서 이 두 줄의 역할을 번갈아가며 사용하는 방식으로 구현했다.

1. 처음에는 첫 번째 줄을 `source row`, 그 다음 줄을 `target row`로 둔다.
    
2. `source row`를 참조하여 `target row`의 각 칸에 cost를 채워 넣으면서 한 행을 모두 계산한다.
    
3. 한 행 계산이 끝나면, 두 줄의 역할을 바꾼다.
    
    - 기존의 `target row`가 이제 다음 계산을 위한 `source row`가 되고,
        
    - 기존의 `source row`는 새로운 `target row` 역할을 맡는다.
        
4. 예를 들어, 지금 2행의 정보를 가지고 3행을 계산하는 시점에서는, **1행의 정보는 더 이상 필요하지 않다.** 따라서 1행이 들어 있던 버퍼(줄)에 3행 값을 **그냥 덮어써도** 전혀 문제가 없다.
    
이런 식으로 매 단계마다 두 줄의 역할만 교대로 바꿔 주면, 전체 DP 테이블을 모두 들고 있지 않아도 되고, **항상 2개의 행만으로 모든 계산을 수행**할 수 있게 된다.

```c++
template <typename index_t, typename value_t> __host__
value_t optimized_dtw(value_t * query, value_t * subject, index_t num_features) {
	const index_t lane = num_features+1;
	value_t * penalty = new value_t[2*lane];
	
	// initialization is slightly different to the quadratic case
	for (index_t index=0; index<lane; index++) penalty[index+1] = INFINITY;
	
	penalty[0] = 0;
		
	for (index_t row=1; row<lane; row++) { 
		const value_t q_value = query[row-1];
		
		const index_t target_row = row & 1;
		const index_t source_row = !target_row;
		
		// this is crucial to reset the zero from row zero to inf
		if (row == 2) penalty[target_row*lane] = INFINITY;
		
		for (index_t col=1; col<lane; col++) { // now everything as usual
			const value_t diag = penalty[source_row*lane+col-1];
			const value_t abve = penalty[source_row*lane+col+0];
			const value_t left = penalty[target_row*lane+col-1];
			const value_t residue = q_value - subject[col-1];
			penalty[target_row*lane + col] 
				= residue*residue + min(diag,min(abve, left)); // relax cell
		}
	}
	
	const index_t last_row = num_features & 1;
	const value_t result = penalty[last_row*lane + num_features];
	delete[] penalty;
	return result;
}

```

이 코드에서는 기존 DTW 구현과 대부분 동일하지만, 메모리를 절약하기 위해 **penalty 테이블을 전체 (n+1)×(n+1) 크기로 만들지 않고, 단 2개의 행만 유지하도록 최적화**하였다.

먼저 다음과 같이 penalty 버퍼를 2행만 갖도록 할당한다.

```c++
const index_t lane = num_features + 1;
value_t * penalty = new value_t[2 * lane];
```

이제 DP 테이블은 실제로는 여러 행이 존재하지만, 메모리에서는 **두 행을 번갈아가며 재사용**하는 방식으로 처리하게 된다.

초기화 방식도 약간 달라졌다.  

왼쪽 첫 열(col = 0)은 DTW 특성상 항상 무한대(INF)로 시작해야 하는데, 두 행만 번갈아 사용하는 구조에서는 매번 전체를 초기화할 필요가 없다. 따라서 **맨 처음 한 번만** 다음과 같이 첫 행을 초기화한다.

```c++
for (index_t index = 0; index < lane; index++)
    penalty[index + 1] = INFINITY;

penalty[0] = 0;
```

즉, 첫 번째 행은 `[0, INF, INF, ..., INF]`로 설정되며 이는 기존 DTW 초기 조건과 동일하다.

이후부터는 실제 DP 행(row)을 1부터 순회하며 두 버퍼 행의 역할을 교대로 바꿔 가면서 계산한다.

```c++
for (index_t row = 1; row < lane; row++) { 
    const value_t q_value = query[row - 1];
		
    const index_t target_row = row & 1;   // 이번에 값을 쓸 행
    const index_t source_row = !target_row; // 이전 값을 가지고 있는 행
```

이때 `&` 연산자는 비트 AND 연산자로, `row & 1` 을 수행하면 **row 의 마지막 비트(LSB)가 1인지 0인지**를 확인하게 된다.

정수에서 **마지막 비트가 1이면 홀수**, 0이면 짝수를 의미하므로 `row & 1` 은 row 값이 홀수일 때 1, 짝수일 때 0을 반환한다.

따라서 row가 1부터 시작하면, 첫 번째 반복(row=1)에서는 `row & 1 = 1` 이 되어 **target_row가 1**이 되고, `source_row` 는 그 반대 값이므로 0이 된다. 그 이후 row 값이 증가할 때마다 `row & 1` 결과가 **0, 1, 0, 1…** 형태로 번갈아 나오므로, `target_row` 와 `source_row` 의 역할도 **두 버퍼 사이에서 계속 교대로 바뀌게** 된다.

여기서 중요한 점은 `row == 2`일 때이다. 이 시점에서 `target_row`는 다시 0번 버퍼가 되는데, 이 버퍼의 `(0,0)` 위치에는 초기화 때 넣어 두었던 `0`이 그대로 남아 있다. 그러나 DP[2][0]은 무조건 `INF`여야 하므로, 이 값을 반드시 다시 초기화해 주어야 한다.

```c++
    if (row == 2)
        penalty[target_row * lane] = INFINITY;
```

이렇게 해두면 이후부터는 두 버퍼 행이 모두 일반 DP 행처럼 사용되며,  
불필요한 0이 남아 있어 계산을 방해하는 일이 없어진다.

이후 각 열(col)에 대해 좌상단, 위, 왼쪽 값을 이용한 전형적인 DTW 점화식을 적용하여  
현재 행(target row)에 값을 채워 넣는다.

```c++
    for (index_t col = 1; col < lane; col++) {
        const value_t diag = penalty[source_row * lane + col - 1];
        const value_t abve = penalty[source_row * lane + col];
        const value_t left = penalty[target_row * lane + col - 1];
        const value_t residue = q_value - subject[col - 1];

        penalty[target_row * lane + col] =
            residue * residue + min(diag, min(abve, left));
    }
}
```

---
### main

위 코드는 메인 함수 부분으로, DTW를 테스트하기 위한 전체 흐름은 다음과 같다.

```c++
typedef uint64_t index_t; // 인덱스에 사용할 정수 타입
typedef uint8_t  label_t; // 라벨(C,B,F)을 0,1,2로 인코딩
typedef float    value_t; // 실제 데이터 값(시계열) 타입

int main() {
    constexpr index_t num_features = 128;        // 각 시계열의 길이
    constexpr index_t num_entries  = 1UL << 20;  // 시계열 개수 (약 100만 개)
	
    value_t * data   = nullptr;
    value_t * dist   = nullptr;
    label_t * labels = nullptr; // C, B, F를 0, 1, 2로 인코딩한 라벨
	
    // CUDA에서 제공하는 cudaMallocHost를 사용해 페이지락(pinned) 메모리 할당
    cudaMallocHost(&data,   sizeof(value_t) * num_features * num_entries);
    cudaMallocHost(&dist,   sizeof(value_t) * num_entries);
    cudaMallocHost(&labels, sizeof(label_t) * num_entries);

    // CBF 데이터셋을 생성해서 data와 labels를 채워 넣는다.
    generate_cbf(data, labels, num_entries, num_features);
	
    // CPU에서 DTW를 수행하고, 결과를 dist 배열에 저장한다.
    host_dtw(data, data, dist, num_entries, num_features); 
	
    // 앞쪽 몇 개 샘플에 대해 라벨과 DTW 결과를 출력해 비교해 본다.
    for (index_t index = 0; index < 9; index++) 
        std::cout << index_t(labels[index]) << " " << dist[index] << std::endl;
		
    // cudaMallocHost로 할당한 메모리는 cudaFreeHost로 해제
    cudaFreeHost(labels);
    cudaFreeHost(data);
    cudaFreeHost(dist);
}

```

정리하자면,

- `cudaMallocHost`를 사용해서 **호스트 메모리 영역을 CUDA용으로 고정(pinned)** 해 두고,
    
- `generate_cbf`로 CBF 데이터와 라벨을 생성한 뒤,
    
- `host_dtw`를 호출해 DTW 결과를 `dist` 배열에 채운 다음,
    
- 일부 샘플에 대해 **라벨과 DTW 거리 값을 나란히 출력하면서 결과를 확인**하는 구조다.

---
## DTW: Naive CUDA Kernel
우리가 앞에서 구현한 DTW를 이제 CUDA 커널로 옮겨서,  
**각 time series 쌍에 대한 DTW 계산을 GPU의 각 스레드가 하나씩 담당**하게 만든 코드이다.

```c
template <typename index_t, typename value_t> __global__
void DTW_naive_kernel(
    value_t * Query,   // pointer to the query time series
    value_t * Subject, // pointer to the subject / database time series
    value_t * Dist,    // pointer to the output distance array
    value_t * Cache,   // auxiliary memory for DP matrices (2 rows per thread)
    index_t num_entries,   // number of time series (m)
    index_t num_features)  // length of each time series (n)
{
    // 전역 스레드 ID, DP 행 길이, 각 스레드가 처리할 시계열의 시작 위치 계산
    const index_t thid = blockDim.x * blockIdx.x + threadIdx.x;
    const index_t lane = num_features + 1;
    const index_t base = thid * num_features;  // 시계열의 시작 오프셋

    if (thid < num_entries) { // 유효한 인덱스인지 체크해서 out-of-bounds 접근 방지
        // 이 스레드를 위한 penalty 버퍼 주소 설정 (2 x (n+1) 크기)
        value_t * penalty = Cache + thid * 2 * lane;

        // penalty 행렬 초기화: 첫 행을 [0, INF, INF, ..., INF]로 설정
        penalty[0] = 0;
        for (index_t index = 0; index < lane; index++)
            penalty[index + 1] = INFINITY;

        // DP 테이블을 행(row) 기준으로 순회하면서 완화(relax) 수행
        for (index_t row = 1; row < lane; row++) {
            // 이 스레드가 담당하는 query에서 현재 행에 해당하는 값
            const value_t q_value = Query[base + row - 1];

            // 2행 롤링 버퍼: 홀짝을 이용해 target/source 행 결정
            const index_t target_row = row & 1;
            const index_t source_row = !target_row;

            // 두 번째 행부터는 (0,0)에 남아 있던 0을 INF로 리셋해줘야 함
            if (row == 2)
                penalty[target_row * lane] = INFINITY;

            const index_t src_off = source_row * lane;
            const index_t trg_off = target_row * lane;

            // 열(col)을 순회하면서 DTW 점화식 적용
            for (index_t col = 1; col < lane; col++) {
                const value_t diag = penalty[src_off + col - 1]; // 좌상단
                const value_t abve = penalty[src_off + col];     // 위
                const value_t left = penalty[trg_off + col - 1]; // 왼쪽

                const value_t s_value = Subject[base + col - 1]; 
                const value_t residue = q_value - s_value;

                penalty[trg_off + col] =
                    residue * residue + min(diag, min(abve, left));
            }
        }

        // 마지막 행 인덱스(짝수/홀수)에 따라 결과 위치 선택 후 Dist에 기록
        const index_t last_row = num_features & 1;
        Dist[thid] = penalty[last_row * lane + num_features];
    }
}

```

---
CUDA 버전의 DTW는 기본적인 계산 방식은 OpenMP 구현과 동일하지만, GPU에서는 각 스레드가 처리해야 할 데이터를 직접 지정해야 한다는 차이가 있다. 이를 위해 먼저 전역 스레드 ID를 계산하는데, 다음 코드처럼 블록 크기와 블록 인덱스, 그리고 스레드 인덱스를 조합해 `thid` 값을 얻는다.

```c++
const index_t thid = blockDim.x * blockIdx.x + threadIdx.x;
```

이 값은 전체 쓰레드 중에서 이 쓰레드가 몇 번째인지 나타내며, 이후 이 번호를 기준으로 어떤 시계열 데이터를 처리할지 결정된다. 

```c
const index_t lane = num_features + 1;
const index_t base = thid * num_features;  // 시계열의 시작 오프셋
```

DTW에서 DP 테이블의 열 개수는 패딩을 포함해 `num_features + 1` 이므로 이를 lane이라 정의하며, 여러 개의 시계열이 하나의 긴 배열에 붙어 저장되어 있으므로 쓰레드마다 자신의 입력 데이터가 시작하는 위치를 알아야 한다. 이를 위해 `thid * num_features` 만큼 건너뛰어 offset을 계산하고, 이제 `base` 위치부터 해당 쓰레드가 담당할 시계열 데이터가 존재하게 된다.

추가로, GPU에서는 각 스레드가 사용할 penalty 버퍼가 필요하다. 이 버퍼는 `Cache` 라는 큰 메모리 안에 연속적으로 저장되어 있으며, 쓰레드는 자신의 ID를 이용해 다음과 같이 penalty 영역의 시작 위치를 얻는다.

```c++
value_t * penalty = Cache + thid * 2 * lane;
```

즉, Cache는 전체 쓰레드의 penalty 공간을 global 메모리에 모아둔 영역이고, 각 쓰레드는 `thid` 값만큼 떨어진 위치에 자신만의 `2 x (n+1)` 크기의 penalty 배열을 할당받은 셈이다. 이런 구조 덕분에 CUDA에서는 OpenMP와 동일한 DP 계산 로직을 유지하면서도, 쓰레드별로 독립적인 메모리 영역을 사용해 안전하게 병렬 처리를 수행할 수 있다.

---
나머지 DTW 커널 내부 로직은 앞에서 작성했던 OpenMP 버전과 동일하고, CUDA에서는 이를 다음과 같이 커널로 호출한다.

```c
const uint64_t threads = 32; DTW_naive_kernel<<<SDIV(num_entries, threads), threads>>>(     Data, Data, Dist, Cache, num_entries, num_features );
```

여기서 `threads`는 한 블록당 스레드 수를 의미하고, `SDIV(num_entries, threads)`는 `num_entries`를 32로 나눈 뒤 올림(ceiling)한 값을 block 개수로 사용한다. 

즉, 전체적으로 보면 **총 `num_entries`개의 시계열에 대해, 32개 스레드씩 묶인 여러 개의 블록이 실행되는 구조**가 된다. 각 스레드는 커널 내부에서 자신의 전역 스레드 ID(`thid`)를 계산한 뒤, 그 ID를 기준으로 하나의 데이터셋(하나의 time series)을 전담해서 DTW를 수행한다. 쉽게 말해서, **“스레드 하나가 시계열 하나를 맡는다”**라고 생각하면 된다.

---
### 결과 보기

실행 결과를 보니 Titan X 기준으로 약 30초가 걸렸고, 이는 32코어에서 돌린 OpenMP 코드에 비해 대략 10배 정도 느린 성능이다. 왜 이렇게 느려졌는지 생각해 보면, 가장 큰 이유는 **메모리 접근이 coalescing(병합)되지 않았기 때문**이다.

현재 구현에서는 **각 스레드가 하나의 데이터 셋(하나의 시계열)** 을 통째로 담당하도록 구성되어 있다. 하지만 실제로 GPU에서는 스레드가 개별로 움직이는 것이 아니라, **32개 스레드가 하나의 warp 단위로 묶여 동시에 실행**된다. 즉, 한 번에 실행되는 단위는 “스레드 1개”가 아니라 “warp에 속한 32개 스레드 묶음”이다.

그런데 지금 구조에서는 warp 안의 32개 스레드가 각각 서로 다른 데이터 셋을 들고 있기 때문에, 예를 들어 Query나 Subject를 읽을 때:

- 스레드 0은 데이터셋 0의 `subject[0]`
- 스레드 1은 데이터셋 1의 `subject[0]`
- 스레드 2는 데이터셋 2의 `subject[0]`
- …

이런 식으로 **서로 멀리 떨어진 위치**를 동시에 읽게 된다. 즉, 메모리 주소가 `num_features` 간격으로 점프하는 **stride access**가 되고, 이 때문에 warp 입장에서는 원래 한 번에 인접한 데이터를 쫙 가져올 수 있는 coalesced access를 제대로 활용하지 못한다. 겉으로 보면 각 스레드가 자기 데이터에 “정상적으로” 접근하는 것 같지만, 하드웨어 관점에서는 32개 스레드가 전부 떨어진 주소를 찔러대는 셈이라 매우 비효율적인 접근 패턴이 되는 것이다.

![](../../images/Pasted%20image%2020251205160713.png)

그래서 이 비효율을 줄이기 위해 데이터 저장 방식을 **의도적으로 바꾼다**. 즉, thread들이 동시에 읽는 값들이 **메모리에서도 서로 인접하도록 layout을 재배치하는 것**이다. 이렇게 하면 warp에서 thread들이 parallel하게 접근할 때 하드웨어는 필요한 데이터를 한 번에 연속적으로 가져올 수 있고, 결과적으로 메모리 접근이 병합(coalesced)되어 성능이 크게 개선된다. 요약하면, “thread는 dataset 단위로 계산하지만, GPU 실행 단위가 warp라는 점 때문에 여러 thread가 동시에 접근하는 위치가 서로 붙어 있도록 데이터 저장 구조를 바꾼다”는 개념이다.

---
## DTW: CUDA Kernel with Thread-Local Memory

이번에는 이전 naive CUDA 커널에서 `Cache`라는 글로벌 메모리 버퍼를 쓰던 부분을 없애고,  
각 쓰레드가 **자기만의 penalty 배열을 로컬(스레드 전용) 배열로 들고 있게** 만드는 버전이다.

### 커널 코드

먼저 커널 전체를 채워 보면 이렇게 된다:

```c++
template <typename index_t, typename value_t, index_t const_num_features>
__global__ void DTW_static_kernel(
    value_t * Query,   // pointer to the query
    value_t * Subject, // pointer to the database
    value_t * Dist,    // pointer to the distance
    index_t num_entries,   // number of time series (m)
    index_t num_features)  // number of time ticks (n)
{
    // compute global thread identifier, lane length and offset
    const index_t thid = blockDim.x * blockIdx.x + threadIdx.x;
    const index_t lane = num_features + 1;
    const index_t base = thid * num_features;

    if (thid < num_entries) {
        // 스레드 로컬 penalty 버퍼: 2 x (const_num_features+1) 크기
        // (각 스레드마다 독립된 DP 2-row 버퍼를 갖게 됨)
        value_t penalty[2 * (const_num_features + 1)];

        // --- 초기화: 첫 행을 [0, INF, INF, ..., INF]로 세팅 ---
        penalty[0] = 0;
        
        for (index_t index = 0; index < lane; ++index) {
            penalty[index + 1] = INFINITY;
        }

        // --- DP 완화(relaxation): 행(row) 기준 순회 ---
        for (index_t row = 1; row < lane; ++row) {
            // 이 스레드가 담당하는 query의 row-1 위치 값
            const value_t q_value = Query[base + row - 1];

            // 2줄 롤링 버퍼: 홀짝을 이용해 source/target 결정
            const index_t target_row = row & 1;
            const index_t source_row = !target_row;

            // 두 번째 행부터는 (0,0)에 남아 있던 0을 INF로 리셋
            if (row == 2) {
                penalty[target_row * lane] = INFINITY;
            }

            const index_t src_off = source_row * lane;
            const index_t trg_off = target_row * lane;

            // 열(col)을 순회하며 전형적인 DTW 점화식 적용
            for (index_t col = 1; col < lane; ++col) {
                const value_t diag = penalty[src_off + col - 1]; // 좌상단
                const value_t abve = penalty[src_off + col];     // 위
                const value_t left = penalty[trg_off + col - 1]; // 왼쪽

                const value_t s_value = Subject[base + col - 1]; // subject 값
                const value_t residue = q_value - s_value;

                penalty[trg_off + col] =
                    residue * residue + min(diag, min(abve, left));
            }
        }

        // 마지막 행 인덱스(짝/홀)에 따라 결과 위치 선택 후 Dist에 기록
        const index_t last_row = num_features & 1;
        Dist[thid] = penalty[last_row * lane + num_features];
    }
}


```

```c
value_t penalty[2 * (const_num_features + 1)];
```

쉽게 말하면 이번 최적화의 핵심은 **DTW 계산 중 사용되는 penalty 테이블을 기존처럼 global memory에 두지 않고, thread-local memory로 옮겨 사용하는 것**이다. thread-local 메모리는 각 스레드가 독립적으로 가지는 전용 공간이며, 물리적으로는 여전히 global memory 위에 존재한다. 그렇다면 물리 주소가 동일한데 왜 이것이 최적화가 될까?

그 이유는 **컴파일러와 하드웨어가 이 공간을 다루는 방식이 달라지기 때문**이다. 이전 구현에서는 penalty 배열이 “큰 global 배열에 대한 포인터 연산” 형태였기 때문에, 컴파일러 입장에서는 이 데이터가 언제 어떻게 바뀔지 확신할 수 없었다. 그래서 penalty 접근을 **절대 register에 올릴 수 없다고 판단했으며**, 결국 DP 연산 시 penalty 값을 읽고 쓰는 모든 연산이 **매번 global memory 왕복**이 발생하게 되었다.

반면 thread-local memory에 penalty를 선언하면 상황이 달라진다. 이 공간은 **각 스레드만 접근하고 다른 스레드가 건드릴 수 없다는 것이 명확히 보장되기 때문에**, 컴파일러는 이 데이터를 안전하게 **레지스터나 L1 캐시에 올려둘 수 있다.** 그 결과 penalty 접근 비용이 global memory 접근 대신 **register/cache hit 수준으로 떨어지고**, DTW의 반복 계산 부분이 훨씬 빨라지게 된다.

요약하면, **물리적인 저장 위치는 global memory 기반이라 동일하지만, 접근 방식이 thread-private 구조로 바뀌면서 컴파일러가 register-level 최적화를 할 수 있게 된 것이 성능 향상의 진짜 원인**이라고 볼 수 있다.

이제 최적화한 커널을 실제로 실행해보면, 코드는 다음과 같이 호출된다.

```c++
const uint64_t threads = 32;
DTW_static_kernel<uint64_t, float, num_features>
<<<SDIV(num_entries,threads),threads>>>(Data,Data,Dist,num_entries,num_features); 
```

한 블록당 32개의 스레드를 사용하고, `SDIV(num_entries, threads)`를 통해 전체 데이터 개수를 32로 나눈 뒤 올림값을 블록 개수로 사용한다. 여전히 **스레드 하나가 시계열 하나를 맡는 구조**는 그대로 유지된다.

이렇게 thread-local penalty를 사용하는 static 커널을 Pascal 기반 Titan X에서 돌려 보면, 실행 시간은 약 **2.7초**로 측정되며, 이는 이전 `DTW_naive_kernel` 대비 약 **11배 정도 빠른 속도**이다. 즉, penalty를 global memory 대신 thread-local로 옮겨서 레지스터/캐시를 적극 활용하게 만든 최적화 효과는 꽤 크게 나타난 셈이다.

다만, 이 속도는 여전히 **32코어 OpenMP CPU 코드와 비교하면 ‘조금 빠르거나 비슷한 수준’** 에 그친다. 그 이유는 아직도 `Query`와 `Subject`에 대한 접근 패턴이 warp 입장에서 **non-coalesced** 인 상태로 남아 있기 때문이다. 

---
## Thread and Memory Hierarchy

non-coalesced 문제를 이해하려면 먼저 GPU의 스레드 구조와 메모리 계층을 정리할 필요가 있다.

![](../../images/Pasted%20image%2020251205162441.png)

각 쓰레드는 **자신만의 register와 local memory를** 가지고 있으며, 이 영역은 쓰레드 외부에서 접근할 수 없는 전용 공간이다.  

그 위 계층에는 **shared memory가 있는데, 이는 thread block 내부에서 여러 스레드가 함께 사용하는 빠른 저장 공간**이다. 마지막으로 **global memory는 GPU 전체 스레드가 접근 가능한 가장 큰 메모리 공간**으로, 속도는 상대적으로 느리다. 이 구조 때문에 **shared memory는 메모리 병목을 줄이는 데 매우 효과적**이며, 특히 데이터 접근 패턴을 스레드들이 재사용하거나 접근 순서를 조정할 때 성능을 크게 향상시킨다. 

정리하면, **local memory는 논리적으로는 각 thread의 고유 공간이지만 실제로는 global memory 위에 위치하기 때문에 절대적인 접근 속도는 느린 편**이다. 반면 **shared memory는 한 block 내부의 여러 thread가 함께 사용하는 공간이지만, 접근 대상이 block 단위로 제한되기 때문에 global memory보다 훨씬 빠르게 동작한다.**

즉, local memory는 thread 전용이라는 점에서 안전성과 독립성을 제공하지만 속도면에서는 global memory 특성을 그대로 가지는 반면, shared memory는 범위가 좁은 대신 **많은 thread가 빠르게 데이터를 재사용하고 협력할 수 있도록 설계된 고속 저장 공간**이라고 이해할 수 있다.

---
## DTW: CUDA Kernel with Shared Memory

shared memory 를 사용한 버전이다. 전체 DTW 로직 자체는 이전과 동일하고, 여기서는 **메모리 공간을 어떻게 할당해서 쓰는지**에 집중해서 보자.

```c++
template <typename index_t,typename value_t> __global__
void DTW_shared_kernel(value_t * Query, value_t * Subject, value_t * Dist,
                       index_t num_entries, index_t num_features) {

    const index_t thid = blockDim.x * blockIdx.x + threadIdx.x; 
    const index_t lane = num_features + 1;      // DP 테이블의 열 개수
    const index_t base = thid * num_features;   // 이 스레드가 담당하는 시계열 시작 위치
    
    extern __shared__ value_t Cache[];          // 동적 shared memory 선언
    
    if (thid < num_entries) {
        // 이 블록의 shared memory(Cache) 중에서
        // threadIdx.x 에 해당하는 스레드가 쓸 구간을 penalty로 사용
        value_t * penalty = Cache + threadIdx.x * (2 * lane); 
        
        // DP 테이블 초기화: 첫 행 [0, INF, INF, ...]
        penalty[0] = 0;
        for (index_t index = 0; index < lane; index++)
            penalty[index + 1] = INFINITY;
        
        // 나머지 로직은 이전과 동일한 DTW 점화식
        for (index_t row = 1; row < lane; row++) { 
            const value_t q_value = Query[base + row - 1];
            const index_t target_row = row & 1;
            const index_t source_row = !target_row;
            
            if (row == 2) 
                penalty[target_row * lane] = INFINITY;
            
            const index_t src_off = source_row * lane;
            const index_t trg_off = target_row * lane;
            
            for (index_t col = 1; col < lane; col++) {
                const value_t diag = penalty[src_off + col - 1];
                const value_t abve = penalty[src_off + col];
                const value_t left = penalty[trg_off + col - 1];
                const value_t s_value = Subject[base + col - 1];
                const value_t residue = q_value - s_value;
                
                penalty[trg_off + col] =
                    residue * residue + min(diag, min(abve, left));
            }
        }
        
        const index_t last_row = num_features & 1;
        Dist[thid] = penalty[last_row * lane + num_features];
    }
}
```

여기서 핵심은 이 한 줄이다:

```c++
extern __shared__ value_t Cache[]; 
```

`Cache`라는 이름의 배열을 선언했는데, `__shared__` 키워드가 붙어 있으므로 이 공간은 **shared memory 영역**에 위치한다. 그리고 `extern` 으로 선언했기 때문에, 크기를 컴파일 시점이 아니라 **커널을 호출할 때 동적으로 지정하겠다**는 의미가 된다. 즉, “shared memory 를 쓸 건데, 정확한 크기는 나중에 런치(launch)할 때 알려줄게”라는 선언이다.

실제로 커널 호출부에서는 다음과 같이 shared memory 크기를 함께 넘겨준다.

```c++
uint64_t threads = 32;
uint64_t sh_mem = 2 * (num_features + 1) * threads * sizeof(float);

DTW_shared_kernel<uint64_t, float>
    <<<SDIV(num_entries, threads), threads, sh_mem>>>(
        Data, Data, Dist, num_entries, num_features
    );
```

여기서 `threads`는 한 블록당 스레드 수(32), `lane = num_features + 1`, 
각 스레드가 필요한 penalty 버퍼 크기는 `2 * lane` 이고,  
이걸 `threads` 개만큼 써야 하므로 전체 shared memory 크기는:

> `2 * (num_features + 1) * threads * sizeof(float)`

가 된다. 이 값을 `<<<... , threads, sh_mem>>>` 의 세 번째 인자로 넘겨주면, 그 크기만큼의 shared memory 가 **블록마다 동적으로 할당**되고, 커널 내부에서 `extern __shared__ value_t Cache[];` 로 그 공간을 사용할 수 있게 되는 것이다.

정리하면 이렇게 **penalty 테이블을 global / local 이 아니라 shared memory에 올려서**,  
같은 블록 안의 스레드들이 빠른 on-chip 메모리를 활용해 DTW 연산을 수행하도록 최적화한 버전이라고 볼 수 있다.

### 결과 정리

Titan X에서 이 shared memory 버전의 실행 시간은 약 **0.63초**로 나타났으며, 이는 이전 `DTW_static_kernel` 대비 약 **4.3배 빠른 성능**이다. 즉, penalty 테이블을 shared memory로 옮기고 on-chip 메모리를 적극적으로 활용함으로써 메모리 병목이 크게 완화된 결과라고 볼 수 있다.

하지만 이 방식에는 분명한 단점이 존재한다.

### shared memory의 크기는 한정적이다

shared memory는 빠른 대신 공간이 매우 작기 때문에 시계열 길이가 길어지면 공유 메모리의 한계를 넘어버린다. CUDA에서는 하나의 SM에 block을 배치할 때 shared memory도 함께 할당되는데, 이 공간이 부족하면 block 자체가 실행되지 못하거나, 한 SM에서 동시에 실행 가능한 block의 수가 줄어 occupancy가 떨어지는 문제가 발생한다.

### Non-coalesced 접근 문제는 여전히 남아 있다

이 shared memory 버전 역시 Query와 Subject를 global memory에서 읽는 방식은 그대로 유지된다. 따라서 warp 내부 thread들이 서로 떨어진 주소를 동시에 읽는 **non-coalesced 접근 문제는 여전히 존재**한다. 하지만 shared memory는 on-chip SRAM이라 global memory처럼 32B 혹은 128B cache-line 정렬 제약을 받지 않고, thread마다 비연속적인 인덱스를 접근해도 sub-transaction 비용이 거의 발생하지 않는다. 즉, shared memory는 **비연속 접근 성능 저하가 거의 없는 메모리 계층**이며, coalescing 요구가 사실상 없다고 볼 수 있다.

따라서 Query/Subject는 여전히 non-coalesced 방식으로 global memory를 읽지만, DTW 연산의 핵심 부분인 penalty 테이블 갱신은 대부분 shared memory에서 수행되므로 coalescing 문제의 영향이 크게 줄어든다. 결과적으로 shared memory 버전은 non-coalesced 문제를 해결한 것은 아니지만, penalty 계산을 global memory 대신 shared memory에서 처리함으로써 전체 성능을 의미 있게 끌어올린 형태라고 정리할 수 있다.

---
## Wavefront Relaxtion

지금까지 의존성은 좌대각선 위 왼쪽에 있다고 해서 2줄만 사용해서 역할을 번갈아가도록 배정해서 사용 공간을 줄였다. 근데 조금더 생각해보면 좌우 대각선 사이의 값들은 의존성이 없다. 어

![](../../images/Pasted%20image%2020251205164822.png)
