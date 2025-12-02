# OpenMP: Hello World

```c++
#include<iostream>

int main(){
	#pragma omp parallel
	std::cout<< "Hello world" <<std::endl;
}
```

OpenMP를 사용하면 위 코드에서처럼 `#pragma omp parallel` 지시문을 통해 병렬 영역을 만들 수 있다. 이 구문이 붙은 블록은 여러 개의 쓰레드가 동시에 실행하며, 기본적으로 CPU 코어 수만큼 쓰레드가 생성되기 때문에 “Hello world”가 여러 번 출력된다.

만약 실행 전에 쓰레드 수를 직접 지정하고 싶다면 환경 변수 **OMP_NUM_THREADS**를 설정해 원하는 쓰레드 개수로 실행할 수 있다. 다만 이는 프로그램 실행 순간에만 적용된다.

아래와 같이 코드 내부에서 직접 쓰레드 수를 지정할 수도 있다.

```c++
int main(){
	#pragma omp parallel num_threads(4)
	{
		int i = omp_get_thread_num();
		int n = omp_get_num_threads();
		std::cout << "Hello world from thread " 
		          << i << " of " << n << std::endl;
	}
}
```

`num_threads(4)`는 병렬 영역에서 4개의 쓰레드를 사용하겠다는 뜻이다.  
`omp_get_thread_num()`은 해당 코드를 실행하는 쓰레드의 ID를,  
`omp_get_num_threads()`는 전체 쓰레드의 수를 반환한다.

---

# parallel for Directive

우리가 원하는 것은 각 쓰레드가 서로 다른 일을 수행하여 하나의 작업을 병렬적으로 처리하는 것이다. OpenMP는 데이터 의존성이 없는 경우 `for` 반복문을 쉽게 병렬화할 수 있다.

```c
#pragma omp parallel
{
	#pragma omp for
	for(...) {
	}
}
```

또는 다음처럼 축약해 쓸 수도 있다.

```c
#pragma omp parallel for
for(...) {
}
```

`parallel for`를 사용하면 OpenMP가 전체 반복 범위를 쓰레드 수에 맞게 여러 블록으로 나누고, 각 쓰레드가 자신에게 할당된 구간을 병렬적으로 실행한다. 이를 기본적으로 **block distribution** 방식이라고 한다.

즉,

- `#pragma omp parallel` : 여러 쓰레드를 생성
    
- `#pragma omp for` : 반복문의 인덱스를 쓰레드에 나누어서 실행
    

이 두 단계가 합쳐져 하나의 for 루프를 병렬화하게 된다.

또한 OpenMP의 루프 병렬화는 컴파일러가 반복 횟수를 미리 계산해야 하므로 다음 조건들이 필요하다.

- 반복문의 인덱스는 루프 중간에 변경되면 안 된다.
    
- `goto`, `break` 등 반복문을 갑자기 빠져나오는 문장은 사용할 수 없다.
    
- 반복 범위는 컴파일 시간에 정적으로 계산할 수 있어야 한다.

---

# Vector Addition with OpenMP

```c
int main() {
	const uint64_t num_entries = 1UL << 30;
	std::vector<no_init_t<uint64_t>> x(num_entries); // memory allocation for the three vectors x,y,z
	std::vector<no_init_t<uint64_t>> y(num_entries); // using the no_init_t template as a wrapper for
	std::vector<no_init_t<uint64_t>> z(num_entries); // the actual type
	
	#pragma omp parallel for // manually initialize the input vectors x and y
	for (uint64_t i = 0; i < num_entries; i++) {
		x[i] = i;
		y[i] = num_entries - i;
	}
	
	for (uint64_t i = 0; i < num_entries; i++) // compute x + y = z sequentially
		z[i] = x[i] + y[i];
	
	#pragma omp parallel for // compute x + y = z in parallel
	for (uint64_t i = 0; i < num_entries; i++)
		z[i] = x[i] + y[i];
	
	#pragma omp parallel for // check if summation is correct
	for (uint64_t i = 0; i < num_entries; i++)
		if (z[i] - num_entries)
			std::cout << "error at position " << i << std::endl;
}
```



# Implicit Synchronization

```c
#pragma omp parallel
{ 
	// <- spawning of threads
	#pragma omp for
	for (uint64_t i = 0; i < num_entries; i++) {
		x[i] = i;
		y[i] = num_entries - i;
	}
	// <- implicit barrier
	
	#pragma omp for
	for (uint64_t i = 0; i < num_entries; i++)
		z[i] = x[i] + y[i];
	// <- another implicit barrier
	
	#pragma omp for
	for (uint64_t i = 0; i < num_entries; i++)
		if (z[i] - num_entries)
			std::cout << "error at position “ << i << std::endl;
		
} // <- joining of threads
// <- final barrier of the parallel scope
```

`#pragma omp parallel` 블록 안에 `#pragma omp for`가 여러 개 존재한다면, 각 `for` 구문 뒤에는 자동으로 **암묵적 동기화(implicit barrier)** 가 삽입된다.  

즉 첫 번째 반복문을 실행한 모든 쓰레드가 작업을 마칠 때까지 기다린 후에야 다음 반복문으로 넘어간다. 쓰레드 간 작업 순서를 보장하기 위한 기본 동작이다.

하지만 반복문 사이에 **데이터 의존성이 전혀 없다면**, 굳이 기다릴 필요 없이 바로 다음 작업으로 넘어가도 된다.  이럴 때 사용하는 옵션이 `nowait`이다.

```c++
#pragma omp parallel
{
	#pragma omp for nowait
	for (...) {
		// 첫 번째 반복문: 종료를 기다리지 않음
	}
	
	#pragma omp for
	for (...) {
		// 두 번째 반복문
	}
}
```

`nowait`을 붙이면 앞 반복문의 모든 쓰레드가 끝나기를 기다리지 않고 다음 구문으로 즉시 넘어간다. 단, 이렇게 동기화를 생략할 때는 **두 반복문이 서로 영향을 주지 않는 완전한 독립 작업**이어야 한다. 의존성이 있는 경우에는 반드시 기본 동작(implicit barrier)을 유지해야 한다.

아래는 **제목 그대로 유지하면서**, 사용자가 작성한 내용을 **더 자연스럽고 매끄럽게 정리한 버전**입니다.

---

# Variable Sharing and Privatization

```c++
#include <stdio.h>
int main() {
    #pragma omp parallel for
    for (int i = 0; i < 4; i++)
        for (int j = 0; j < 4; j++)
            printf("%d %d\n", i, j);
}
```

위 코드는 이중 for문이며, `#pragma omp parallel for`는 **가장 바깥 for문만 병렬화**한다. 따라서 바깥 인덱스 `i`가 여러 쓰레드에 나누어지고, 각 쓰레드는 자신이 맡은 `i`에 대해 내부 `j` 반복을 순차적으로 수행한다.

또 한 가지 중요한 점은 `i`와 `j`가 **병렬 영역 안에서 선언된 변수**임으로, 각 쓰레드는 서로 다른 `i`, `j`를 독립적으로 가진다. 즉, 이 경우에는 race condition이 발생하지 않는다.

---

# Declaring Private Variables

그렇다면 반복문에 사용되는 변수가 **병렬 영역 밖에서 선언되면** 어떻게 될까?

```c++
int i, j;
#pragma omp parallel for
for (i = 0; i < 4; i++)
    for (j = 0; j < 4; j++)
        printf("%d %d\n", i, j);
```

이때 변수 `i`와 `j`는 병렬 영역 밖에 있기 때문에 **쓰레드들이 동일한 변수에 동시에 접근**하게 된다. 이런 상황에서는 출력 결과가 섞이거나, 반복 인덱스가 엉키는 등의 race condition이 발생할 수 있다.

이를 해결하기 위해 OpenMP에서는 `private` 절을 제공한다.

```c++
int main() {
    int i, j; // C90 규칙 때문에 바깥에서 선언
    // #pragma omp parallel for만 사용하면 j에서 race condition 발생
    #pragma omp parallel for private(j)
    for (i = 0; i < 4; i++)
        for (j = 0; j < 4; j++)
            printf("%d %d\n", i, j);
}
```

`private(j)`를 사용하면, 비록 `j`가 병렬 영역 바깥에서 선언되었더라도 **각 쓰레드마다 독립적인 j 복사본**을 생성한다.  

즉, 공유 변수로 인한 경쟁 상태를 제거하고, 반복문의 올바른 동작을 보장할 수 있다.

---
# Initialization of Privatized Variables

`private()`을 사용하면 각 쓰레드는 해당 변수를 자신만의 로컬 변수로 관리하게 되지만, **초기화는 자동으로 이루어지지 않는다.**  

즉, private 변수의 초기값은 정의되지 않은 상태이며, 초기화를 하지 않으면 실행 시 예기치 못한 값이 들어 있어 문제가 발생할 수 있다.

OpenMP는 이러한 문제를 해결하기 위해 `firstprivate()`을 제공한다.

```c++
int main() {
    int i = 1;
    // each thread declares its own i
    // and sets it to i = 1
    #pragma omp parallel firstprivate(i)
    {
        // i == 1 in every thread
        printf("%d\n", i);
    }
}
```

`firstprivate(i)`는 공유 변수 `i`의 초기값을 기반으로 **각 쓰레드의 private i를 동일한 값으로 초기화**해준다.  

즉, 공유 변수를 로컬 변수로 복사하는 동시에 그 값으로 초기화하는 방식이다.

---

아래 예제를 보면 이를 더 명확히 이해할 수 있다.

```c++
int main() {
    const int num = omp_get_max_threads();
    int *aux = new int[num];

    int i = 1; // 초기값은 1
    #pragma omp parallel firstprivate(i) num_threads(num)
    {
        const int j = omp_get_thread_num();
        i += j;             // 각 쓰레드에서 i를 독립적으로 변경
        aux[j] = i;         // 변경된 값을 전역 배열로 다시 저장
    }
    delete[] aux; // aux에는 [1, 2, 3, ...]가 저장됨
}
```

- 각 쓰레드는 초기값 `i = 1`을 가지고 시작한다.
- 쓰레드 번호 `j`에 따라 `i += j` 연산을 수행하고,
- 그 결과를 전역 배열 `aux`에 저장한다.

이처럼 `firstprivate` 덕분에 쓰레드마다 동일한 초기 상태에서 독립적으로 계산을 시작할 수 있다.


또 한 가지 고민해볼 점은 **parallel for에서 반복 변수처럼 공유 변수를 업데이트하는 경우**이다.

```c++
int main() {
    int i;
    #pragma omp parallel for lastprivate(i) num_threads(16)
    for (int j = 0; j < 16; j++)
        i = j;
    // 이제 i는 마지막 반복에서의 값(15)을 가진다
}
```

기본적으로, 병렬 for 내에서 `i = j` 같은 코드를 쓰면 각 쓰레드가 서로 다른 순서로 `i`를 덮어쓰기 때문에 루프 이후 `i`가 어떤 값이 될지 예측할 수 없다.

이를 해결해주는 것이 `lastprivate(i)`이다.

반복문은 병렬로 수행하되, 반복문의 **마지막 반복(iteration)** 에서의 값을 병렬 영역 밖의 `i`에 복사해준다. 즉, 실행 순서와 관계없이 최종 반복에서의 값을 정확히 보존할 수 있다.
