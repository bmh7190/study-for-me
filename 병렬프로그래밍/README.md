# 병렬프로그래밍

병렬 프로그래밍의 이론적 배경, C++11 멀티스레딩, OpenMP, CUDA를 정리한 노트입니다.

## 목차

### Theoretical Background

- [1. Theroretical Background](<2. Theroretical Background/1. Theroretical Background.md>)
  - 병렬화가 필요한 이유와 성능 분석에 필요한 기본 이론을 정리합니다.

### C++ 11 Multithreading

- [4-1. C++11 Multithreading](<4. C++ 11 Multithreading/4-1 C++11 Multithreading.md>)
  - C++11 thread API와 thread 생성, join, detach 등 기본적인 멀티스레딩 사용법을 다룹니다.
- [4-2. Matrix-Vector Multiplicaton](<4. C++ 11 Multithreading/4-2 Matrix-Vector Multiplicaton.md>)
  - 행렬-벡터 곱을 예제로 block, cyclic, block-cyclic 분할 방식의 병렬화를 정리합니다.
- [4-3. The MNIST Dataset](<4. C++ 11 Multithreading/4-3 The MNIST Dataset.md>)
  - MNIST 데이터셋을 이용해 all-pairs distance matrix와 동적 작업 분배 방식을 다룹니다.
- [4-4. Condition Variable in C++11](<4. C++ 11 Multithreading/4-4 Condition Variable in C++11.md>)
  - condition variable을 이용한 thread 대기와 통지, 생산자-소비자형 동기화 패턴을 정리합니다.

### OpenMP

- [1. OpenMP](<6. OpenMP/1. OpenMP.md>)
  - OpenMP의 기본 directive와 병렬 영역, loop 병렬화 개념을 소개합니다.
- [2. The MNIST Dataset](<6. OpenMP/2. The MNIST Dataset.md>)
  - OpenMP 실습에 사용할 MNIST 데이터셋의 구조와 처리 흐름을 정리합니다.
- [3. Exercise](<6. OpenMP/3. Exercise.md>)
  - OpenMP를 적용해 병렬화 연습을 진행하는 실습 중심 노트입니다.
- [4. Scheduling Loops](<6. OpenMP/4. Scheduling Loops.md>)
  - static, dynamic, guided, runtime scheduling과 reduction을 통해 loop workload 분배 방식을 다룹니다.
- [5. Softmax Regression Classifier over MNIST](<6. OpenMP/5. Softmax Regression Classifier over MNIST.md>)
  - MNIST softmax regression을 OpenMP로 병렬화하고 accuracy 평가와 gradient descent 최적화를 정리합니다.

### CUDA

- [7-1. CUDA 알기](<7. CUDA/7-1 CUDA 알기.md>)
  - CUDA programming model, GPU 실행 구조, kernel 호출의 기본 개념을 소개합니다.
- [7-2. Mean Computation](<7. CUDA/7-2 Mean Computation.md>)
  - 평균 계산을 예제로 CUDA thread와 block을 이용한 병렬 reduction 흐름을 다룹니다.
- [7-3. Warp](<7. CUDA/7-3 Warp.md>)
  - thread block이 warp로 나뉘는 방식, warp scheduling, barrier synchronization을 정리합니다.
- [7-4. Centered Data Matrix](<7. CUDA/7-4 Centered Data Matrix.md>)
  - 데이터 행렬을 중심화하는 계산을 CUDA로 병렬화하는 방법을 다룹니다.
- [7-5. Computation of the Covariance Matrix](<7. CUDA/7-5 Computation of the Covariance Matrix.md>)
  - covariance matrix 계산을 GPU 병렬 연산으로 구성하는 과정을 정리합니다.
- [7-6. Tiling](<7. CUDA/7-6 Tiling.md>)
  - shared memory와 tiling 기법을 통해 메모리 접근 비용을 줄이는 최적화 방법을 다룹니다.
- [7-7. Time Series](<7. CUDA/7-7 Time Series.md>)
  - time series 데이터를 대상으로 CUDA 병렬 처리 흐름을 적용하는 내용을 정리합니다.
