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

### main
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

```c++
template <typename index_t, typename value_t> __host__
value_t optimized_dtw(value_t * query, value_t * subject, index_t num_features) {
const index_t lane = num_features+1; // allocate two rows of penalty matrix of shape 2x(n+1)
value_t * penalty = new value_t[2*lane];
// initialization is slightly different to the quadratic case
for (index_t index=0; index<lane; index++) penalty[index+1] = INFINITY;
penalty[0] = 0;
for (index_t row=1; row<lane; row++) { // traverse graph in topologically sorted order
const value_t q_value = query[row-1]; // compute cyclic indices (0,1,0,1,0,...)
const index_t target_row = row & 1;
const index_t source_row = !target_row;
// this is crucial to reset the zero from row zero to inf
if (row == 2) penalty[target_row*lane] = INFINITY;
for (index_t col=1; col<lane; col++) { // now everything as usual
const value_t diag = penalty[source_row*lane+col-1]; // cyclic indices for score matrix
const value_t abve = penalty[source_row*lane+col+0];
const value_t left = penalty[target_row*lane+col-1];
const value_t residue = q_value - subject[col-1]; // traditional indices for time series
penalty[target_row*lane + col] = residue*residue + min(diag,min(abve, left)); // relax cell
}
}
const index_t last_row = num_features & 1; // compute the index of the last row
const value_t result = penalty[last_row*lane + num_features];
delete[] penalty;
return result;
}

```

여기서는 맨 위의 과정과 다르게 최적화 포인트를 적용한게 있다. 생각해보면 dp 테이블에서 한의 지점을 얻기 위해서 필요한 데이터는 좌상단 좌 상 이렇게 3방향인데, 이거 위치를 보면 사실상 2행만 있으면 계산할 수 있다. 그래서 2줄만 사용하되, 2줄의 역할을 번갈아가면서 지정해준다.

맨 처음에는 source row 그 다음줄을 target row로 지정해줘서 source row를 통해 target row 에 cost를 입력해주고 한 행이 끝나면 이제 역할을 바꾼다. 그럼 2번째 줄이 source row가 되고, 첫 번째 줄은 target row가 된다. 이 시점은 지금 2행읉 통해서 3행을 계산하기 때문에 지금 target row에 잇는 첫번째 행 정보는 필요없고, 여기에 덮어씌워서 3행을 계산한거다. 그래서 2줄 사이에 역할을 번갈아가면서 할 수 있는 것이다. 