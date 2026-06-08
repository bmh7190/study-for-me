이번 내용은 **IR Generation 이후에 만들어진 intermediate representation(IR)을 그대로 기계어로 보내지 않고, 더 좋은 형태로 다듬는 과정**을 이해하려고 나온 내용이다. 앞에서 우리는 source code를 token, parse tree, semantic analysis, IR generation 순서로 바꿨다. 그런데 IR은 보통 기계적으로 생성되기 때문에 불필요한 계산, 중복된 임시 변수, 반복문 안에서 매번 다시 계산되는 값 같은 비효율이 생긴다. 그래서 이번 장에서는 “생성된 IR을 어떻게 더 효율적으로 바꿀 수 있는가?”를 보는 것이다.

---
## Intermediate Code Optimization이 필요한 이유

컴파일러 흐름에서 지금 위치는 **IR Generation 다음, Code Generation 이전**이다. 즉, 아직 특정 CPU 명령어로 바꾸기 전이다. 이 시점에서 최적화를 하면 특정 기계에 의존하지 않고, 프로그램 구조 자체를 개선할 수 있다.

여기서 말하는 optimization은 사실 “완벽한 최적 코드 찾기”라기보다는 **intermediate code improvement**에 가깝다. 왜냐하면 어떤 프로그램에 대해 항상 최적의 코드를 찾는 것은 일반적으로 undecidable한 문제이기 때문이다. 그래서 현실적인 목표는 “프로그램의 의미는 바꾸지 않으면서, 더 나은 IR로 고치는 것”이다.

---
## 왜 IR에는 중복이 생기는가

IR generation은 보통 high-level code를 정해진 규칙에 따라 기계적으로 번역한다. 그래서 사람이 보면 같은 계산인데도, IR에서는 여러 번 따로 계산되는 경우가 많다.

예를 들어 이런 코드가 있다고 하자.

```c
b1 = x + x < y;
b2 = x + x == y;
b3 = x + x > y;
```

단순히 IR로 바꾸면 `x + x`와 `y`를 매번 다시 임시 변수에 넣을 수 있다.

```text
_t0 = x + x;
_t1 = y;
b1 = _t0 < _t1;

_t2 = x + x;
_t3 = y;
b2 = _t2 == _t3;

_t4 = x + x;
_t5 = y;
b3 = _t5 < _t4;
```

근데 사실 `x + x`는 세 번 다 같은 값이고, `y`도 그대로다. 그러면 한 번만 계산해서 공유할 수 있다.

```text
_t0 = x + x;
_t1 = y;
b1 = _t0 < _t1;
b2 = _t0 == _t1;
b3 = _t0 > _t1;
```

즉, 여기서 보려는 핵심은 **IR은 정직하게 만들어지지만, 똑똑하게 만들어지는 것은 아니다**라는 점이다. 그래서 나중에 optimizer가 중복 계산을 찾아 제거하거나 공유하게 된다.

----
## 반복문에서의 비효율

반복문에서는 최적화 효과가 더 커진다.

```c
while (x < y + z) {
    x = x - y;
}
```

단순 IR은 반복할 때마다 `y + z`를 다시 계산할 수 있다.

```text
_L0:
_t0 = y + z;
_t1 = x < _t0;
IfZ _t1 Goto _L1;
x = x - y;
Goto _L0;
_L1:
```

그런데 반복문 안에서 `y`와 `z`가 변하지 않는다면 `y + z`는 매번 다시 계산할 필요가 없다. 반복문 밖에서 한 번만 계산해도 된다.

```text
_t0 = y + z;
_L0:
_t1 = x < _t0;
IfZ _t1 Goto _L1;
x = x - y;
Goto _L0;
_L1:
```

이런 걸 보면 optimization은 단순히 코드 줄 수를 줄이는 게 아니라, **실제로 실행되는 계산 횟수를 줄이는 것**이라고 이해하면 된다. 특히 loop 안의 코드는 여러 번 실행되기 때문에, loop 밖으로 뺄 수 있으면 성능 차이가 커진다.

---
## 좋은 optimizer의 조건

좋은 optimizer는 세 가지를 만족해야 한다.

첫째, **observable behavior를 바꾸면 안 된다.**  
즉, 프로그램 실행 결과가 달라지면 안 된다. 아무리 빨라져도 의미가 바뀌면 compiler bug다.

둘째, **가능하면 효율적인 IR을 만들어야 한다.**  
중복 계산 제거, 불필요한 대입 제거, 상수 계산 미리 처리 같은 작업이 여기에 들어간다.

셋째, **optimizer 자체가 너무 오래 걸리면 안 된다.**  
컴파일 시간이 너무 길어지면 실용성이 떨어진다. 그래서 모든 가능한 최적화를 완벽하게 찾기보다는, 제한된 시간 안에서 안전하고 효과적인 개선을 수행한다.

----
## 무엇을 최적화하는가

최적화 대상은 하나만 있는 게 아니다.

보통 가장 많이 생각하는 건 **runtime**이다. 프로그램을 더 빠르게 실행시키는 것이다.

하지만 경우에 따라서는 **memory usage**를 줄이는 것이 더 중요할 수도 있고, 임베디드 시스템에서는 **power consumption**을 줄이는 것이 중요할 수도 있다.

다만 여기서 중요한 점은 compiler optimization이 보통 알고리즘의 Big-O를 바꾸지는 않는다는 것이다. 예를 들어 O(n²) 알고리즘을 O(n log n)으로 바꿔주는 건 보통 programmer의 역할이다. compiler는 주로 같은 알고리즘 안에서 **상수 시간 비용을 줄이는 방향**으로 개선한다.

그래도 상수 계수도 실제 실행에서는 매우 중요하다. 반복문 안에서 곱셈 하나를 없애거나, 중복 메모리 접근을 줄이는 것만으로도 성능 차이가 크게 날 수 있다.

----
## Intermediate Code Optimization vs Code Optimization

여기서 구분해야 하는 게 있다.

**Intermediate code optimization**은 특정 machine에 의존하지 않는 최적화다. 예를 들어 중복 계산 제거, copy propagation, dead code elimination 같은 것은 어떤 CPU에서도 의미가 있다.

반면 **code optimization**은 target machine에 맞춘 최적화다. 예를 들어 특정 CPU에서는 어떤 명령어가 더 빠른지, register를 어떻게 배치할지, cache를 어떻게 활용할지 같은 내용이 여기에 들어간다.

그래서 이번 내용은 주로 **machine-independent optimization**에 가깝다.

---
## Program analysis와 soundness

최적화를 하려면 compiler가 프로그램에 대해 어떤 사실을 알아야 한다.

예를 들어 이런 코드가 있다고 하자.

```c
int x;
int y;

if (y < 5)
    x = 137;
else
    x = 42;

print(x);
```

`print(x)` 지점에서 확실히 말할 수 있는 사실은:

```text
x is either 137 or 42
```

이다.

그런데 compiler가

```text
x is 137
```

이라고 판단하면 틀릴 수 있다. `else`로 가면 `x = 42`이기 때문이다. 이런 분석은 **unsound**하다.

sound analysis는 “틀린 사실을 맞다고 주장하지 않는 분석”이다. compiler optimization에서는 이게 매우 중요하다. 최적화는 프로그램 의미를 바꾸면 안 되기 때문에, 확실하지 않은 정보를 확실하다고 가정하면 위험하다.

---
## Semantics-preserving optimization

이번 장에서 다루는 최적화는 모두 **semantics-preserving**이어야 한다.

즉, 프로그램의 의미는 그대로 두고 IR만 더 효율적으로 바꾸는 것이다.

예를 들어 다음은 semantics-preserving이다.

```text
불필요한 임시 변수 제거
컴파일 타임에 알 수 있는 상수 계산 미리 수행
loop 안에서 변하지 않는 계산을 loop 밖으로 이동
```

반면 bubble sort를 quicksort로 바꾸는 것은 일반적인 compiler optimization으로 보기 어렵다. 결과는 같을 수 있지만 알고리즘 자체를 바꾸는 것이고, 안정 정렬 여부나 side effect 같은 문제도 생길 수 있다.

---
## Basic Block과 Control-Flow Graph

최적화를 하려면 IR을 그냥 한 줄씩 보는 것만으로는 부족하다. 프로그램의 흐름을 구조화해서 봐야 한다.

그래서 나온 개념이 **basic block**이다.

basic block은 쉽게 말하면 **한 번 들어가면 중간에서 빠져나가지 않고, 끝까지 순서대로 실행되는 IR instruction 묶음**이다.

조건문이나 goto가 있으면 control flow가 갈라지기 때문에 block이 나뉜다.

그리고 basic block들을 node로 만들고, block 사이의 실행 흐름을 edge로 연결한 것이 **Control-Flow Graph, CFG**다.

여기서 CFG는 앞에서 배운 Context-Free Grammar가 아니라 **Control-Flow Graph**를 의미한다.

CFG를 만들면 compiler는 “어떤 block 다음에 어떤 block이 실행될 수 있는지”를 알 수 있다. 이게 나중에 global optimization의 기반이 된다.

---
## Optimization의 종류

최적화는 적용 범위에 따라 나눌 수 있다.

**Peephole optimization**은 아주 가까운 몇 개의 instruction만 보고 더 짧거나 빠른 instruction으로 바꾸는 것이다. 말 그대로 작은 구멍으로 코드를 보는 느낌이다.

**Local optimization**은 하나의 basic block 안에서만 최적화한다. control flow를 복잡하게 볼 필요가 없어서 상대적으로 쉽다.

**Global optimization**은 function 전체 CFG를 대상으로 한다. 여러 basic block을 넘나들며 분석하므로 더 강력하지만 더 어렵다.

**Inter-procedural optimization**은 여러 function 사이까지 분석한다. 함수 호출 관계까지 봐야 하므로 더 복잡하다.

---
## Common Subexpression Elimination

Common subexpression elimination은 같은 계산을 여러 번 하지 않도록 하는 최적화다.

예를 들어:

```c
a = b * c + g;
d = b * c * e;
```

여기서 `b * c`가 중복된다.

그래서 다음처럼 바꿀 수 있다.

```c
tmp = b * c;
a = tmp + g;
d = tmp * e;
```

핵심 조건은 중간에 `b`나 `c` 값이 바뀌지 않아야 한다는 것이다. 값이 바뀌지 않았다면 같은 계산 결과를 다시 사용할 수 있다.

이 최적화는 단순히 계산을 줄이는 것뿐 아니라, 뒤의 다른 최적화가 더 잘 적용되도록 도와준다.

---
## Copy Propagation

Copy propagation은 복사된 값을 실제 값으로 대체하는 최적화다.

예를 들어:

```c
int a = 4;
int b = a;
int c = a;
```

`a`가 계속 4라면 다음처럼 바꿀 수 있다.

```c
int a = 4;
int b = 4;
int c = 4;
```

조금 더 일반적으로 보면:

```text
v1 = v2
a = ... v1 ...
```

이런 상황에서 `v1`과 `v2`가 바뀌지 않았다면 `v1` 대신 `v2`를 사용할 수 있다.

이게 중요한 이유는 dead code elimination으로 이어질 수 있기 때문이다. 복사만 하고 실제로 필요 없는 변수가 드러날 수 있다.

---
## Dead Code Elimination

Dead code elimination은 결과가 어디에서도 사용되지 않는 코드를 제거하는 것이다.

예를 들어:

```c
int dead_code(int input) {
    int i = 4;
    return input;
}
```

여기서 `i = 4`는 아무 데도 쓰이지 않는다. 그래서 제거해도 프로그램 의미가 바뀌지 않는다.

```c
int dead_code(int input) {
    return input;
}
```

즉, 어떤 assignment가 dead하다는 것은 **그 assignment로 저장된 값이 이후에 절대 읽히지 않는다**는 뜻이다.

## Local optimization은 여러 번 반복해야 한다

중요한 점은 최적화 하나를 한 번 적용한다고 끝나는 게 아니라는 것이다.

예를 들어 common subexpression elimination을 하면 copy propagation이 가능해지고, copy propagation을 하면 dead code가 드러날 수 있다. dead code를 제거하면 다시 다른 common subexpression이 보일 수도 있다.

즉, 최적화들은 서로 연결되어 있다.

대략 흐름은 이런 느낌이다.

```text
중복 계산 제거
→ 복사 관계가 단순해짐
→ 사용되지 않는 값이 드러남
→ dead code 제거
→ 다시 최적화 가능성 발생
```

그래서 optimizer는 보통 여러 optimization pass를 반복적으로 적용한다.

## Arithmetic simplification과 Constant folding

Arithmetic simplification은 더 쉬운 연산으로 바꾸는 것이다.

예를 들어:

```text
x = 4 * a
```

를

```text
x = a << 2
```

처럼 바꿀 수 있다. 4를 곱하는 것은 왼쪽으로 2비트 shift하는 것과 같기 때문이다.

Constant folding은 컴파일 타임에 이미 알 수 있는 상수 계산을 미리 해버리는 것이다.

```text
x = 4 * 5
```

는 실행 중에 계산할 필요 없이

```text
x = 20
```

으로 바꿀 수 있다.

## Global optimization이 필요한 이유

Local optimization은 basic block 하나 안에서만 본다. 그래서 간단하고 안전하지만, block을 넘어가는 정보는 활용하지 못한다.

Global optimization은 CFG 전체를 본다. 그래서 더 강력하다.

예를 들어 어떤 값이 여러 block에서 동일하게 유지된다면, local analysis만으로는 알기 어렵다. 하지만 global analysis를 하면 function 전체 흐름을 보고 판단할 수 있다.

대표적인 global optimization은 다음과 같다.

```text
code motion
global dead code elimination
global constant propagation
partial redundancy elimination
```

특히 code motion은 반복문 안에서 변하지 않는 코드를 밖으로 빼는 것처럼, block 사이로 코드를 이동시키는 최적화다.

## Local analysis와 Global analysis의 차이

local analysis에서는 statement의 predecessor가 보통 하나다. 그냥 앞 문장만 보면 된다.

하지만 global analysis에서는 어떤 basic block에 여러 경로로 들어올 수 있다. 그러면 각 predecessor에서 오는 정보를 합쳐야 한다.

예를 들어 어떤 block에 들어오기 전에 한 경로에서는 `x = 3`, 다른 경로에서는 `x = 5`였다면, 해당 block 시작점에서 `x = 3`이라고 단정할 수 없다. 이런 식으로 여러 경로의 정보를 안전하게 결합해야 한다.

또 하나의 차이는 CFG에는 loop가 있을 수 있다는 점이다. loop가 있으면 정보가 다시 돌아오므로, 분석 결과를 여러 번 갱신해야 할 수 있다. 이때 무한히 반복되지 않도록 조심해야 한다.

## 정리

이번 내용의 핵심은 이거다.

IR은 이전 단계에서 기계적으로 생성되기 때문에 중복 계산, 불필요한 변수, 반복문 안의 낭비 같은 비효율이 생긴다. Intermediate code optimization은 이런 IR을 프로그램 의미는 유지한 채 더 효율적으로 바꾸는 과정이다.

이를 위해 compiler는 program analysis를 수행하고, 이 분석은 sound해야 한다. 그리고 최적화는 basic block, control-flow graph 같은 구조 위에서 수행된다.

처음에는 local optimization부터 본다. common subexpression elimination, copy propagation, dead code elimination, constant folding 같은 것들이 여기에 해당한다. 이후에는 CFG 전체를 대상으로 하는 global optimization으로 확장된다.