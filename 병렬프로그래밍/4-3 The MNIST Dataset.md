MNIST 데이터셋은 손글씨 숫자(0~9) 이미지를 모은 대표적인 머신러닝 데이터셋이다.  
총 약 **65,000장(훈련용 60,000장 + 테스트용 10,000장)** 의 이미지로 구성되어 있으며,  
각 이미지는 **28 × 28 픽셀**의 흑백(grayscale) 이미지다.

![](../images/Pasted%20image%2020251017155154.png)

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

sequential_all_pairs 를 보면 이때 rows는 이미지고 cols는 실제 컬럼이다. 

0부터 이미지를 하나씩 늘리고, 그 전까지 이미지의 거리를 계산된다. 