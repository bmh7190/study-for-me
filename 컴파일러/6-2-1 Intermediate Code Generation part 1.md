중간 코드 생성은 컴파일러가 소스 코드를 바로 기계어로 바꾸기 전에, **컴파일러가 다루기 쉬운 중간 형태로 바꾸는 과정**이다.

앞에서 배운 내용을 연결해서 보면, lexical analyzer는 문자를 token으로 만들고, parser는 token을 이용해서 syntax tree를 만든다. 그리고 static checker는 타입이나 의미적으로 문제가 없는지 확인한다. 그다음 단계가 바로 **Intermediate Code Generator**이다.

즉 전체 흐름은 이렇게 볼 수 있다.

![](../images/Pasted%20image%2020260518164629.png)

여기서 intermediate representation, 즉 **IR**은 소스 코드와 기계어 사이에 있는 표현이다. 사람이 작성하는 고급 언어보다는 단순하고, 실제 기계어보다는 추상적이다.

이렇게 중간 표현을 두는 이유는 컴파일러 구조를 나누기 위해서이다. Front end는 소스 코드를 분석해서 IR을 만들고, back end는 IR을 실제 기계어로 바꾼다. 그러면 언어가 달라져도 IR 이후의 최적화나 코드 생성 과정을 어느 정도 공통적으로 사용할 수 있다.

---
## Intermediate Representation
IR은 컴파일러가 프로그램에 대해 알아낸 정보를 담고 있는 내부 표현이다.

Front end는 소스 코드를 읽고 문법, 의미, 타입 등을 분석한 뒤 IR을 만든다. Middle end는 이 IR을 더 효율적인 IR로 바꾼다. Back end는 IR을 실제 target machine code로 변환한다.

즉 역할을 나누면 다음과 같다.

![](../images/Pasted%20image%2020260518164531.png)

IR이 중요한 이유는 **최적화의 기준이 되는 표현**이기 때문이다. 예를 들어 어떤 계산이 반복되는지, 어떤 변수가 더 이상 사용되지 않는지, 어떤 코드가 불필요한지 등을 소스 코드 상태에서 바로 분석하기는 어렵다. 하지만 IR로 바꾸면 연산이 더 단순한 형태로 나뉘기 때문에 분석이 쉬워진다.

그래서 IR은 단순한 임시 코드가 아니라, 컴파일러가 프로그램을 이해한 결과라고 볼 수 있다.

---
# Concrete and Abstract Syntax Trees

파싱을 하면 **parse tree**가 만들어진다. 이 parse tree는 문법 규칙을 그대로 반영하기 때문에 **concrete syntax tree**라고도 한다.

그런데 parse tree는 너무 자세하다. 예를 들어 수식 하나를 표현할 때도 `E`, `T`, `F` 같은 문법용 nonterminal이 많이 들어간다. 이런 정보는 문법 분석에는 필요하지만, 실제 코드 생성에는 불필요한 경우가 많다.

그래서 컴파일러는 보통 **AST, Abstract Syntax Tree**를 사용한다. AST는 문법 규칙의 세부 구조보다는 실제 의미 있는 연산 구조만 남긴 트리이다.

예를 들어 다음 식이 있다고 하자.

```
a + b * c
```

이 식의 핵심은 `b * c`가 먼저 계산되고, 그 결과가 `a`와 더해진다는 것이다. AST는 이를 다음처럼 표현한다.

```        
      +
    /   \
   a     *
        / \
       b   c
```

즉 AST는 “문법적으로 어떻게 유도되었는가”보다 “실제로 어떤 계산 구조를 가지는가”에 집중한다.

정리하면, parse tree는 문법 규칙을 자세히 반영한 트리고 AST는 코드 생성과 의미 분석에 필요한 구조만 남긴 트리라고 보면 된다.

---
# Variants of Syntax Tree
AST는 트리 구조이기 때문에 같은 계산이 여러 번 등장해도 각각 별도의 노드로 표현될 수 있다. 그런데 실제로 같은 계산이 반복된다면, 컴파일러 입장에서는 이를 한 번만 계산하고 재사용하는 것이 더 효율적이다.

이때 사용하는 표현이 **DAG, Directed Acyclic Graph**이다. DAG는 방향이 있고 cycle이 없는 그래프이다. AST와 비슷하지만, 같은 값을 나타내는 노드는 하나만 만들고 여러 곳에서 공유할 수 있다.

예를 들어 다음 식을 보자.

![](../images/Pasted%20image%2020260518165251.png)

```
a + a * (b - c) + (b - c) * d
```

여기서 `(b - c)`가 두 번 등장한다. AST에서는 `(b - c)`를 두 번 만들 수 있지만, DAG에서는 `(b - c)` 노드를 하나만 만들고 두 위치에서 공유한다.

이렇게 하면 컴파일러는 다음 사실을 알 수 있다.

```
(b - c)는 같은 계산이므로 한 번만 계산해도 된다.
```

그래서 DAG는 공통 부분식 제거 같은 최적화와 관련이 있다. 핵심은 **중복 계산을 명시적으로 드러내는 표현**이라는 점이다

---
# Producing DAG
DAG를 만들 때는 보통 `Leaf`와 `Node` 같은 생성 함수를 사용한다.

Leaf는 변수나 상수처럼 더 이상 쪼갤 수 없는 값을 만드는 함수이고, Node는 연산자와 자식 노드를 묶어서 새로운 연산 노드를 만드는 함수라고 보면 된다.

예를 들어,

```
b - c
```

는 다음처럼 만들 수 있다.

```
Node('-', Leaf(b), Leaf(c))
```

그런데 DAG에서는 무조건 새 노드를 만들지 않는다. 새 노드를 만들기 전에 **이미 같은 노드가 존재하는지 확인**한다.

즉 과정은 다음과 같다.

1. 만들려는 노드의 형태를 확인한다.
2. 같은 연산자와 같은 자식을 가진 노드가 이미 있는지 찾는다.
3. 있으면 기존 노드를 사용한다.
4. 없으면 새 노드를 만든다.

이 방식 덕분에 같은 표현식이 여러 번 등장해도 하나의 노드를 공유할 수 있다.

---
# Construction of DAG
DAG를 효율적으로 만들기 위해 **value numbering**이라는 방법을 사용한다.

![](../images/Pasted%20image%2020260518165752.png)

각 노드에 고유한 번호를 붙이는데, 이 번호를 **value number**라고 한다. 같은 값을 나타내는 표현식은 같은 번호를 갖게 된다.

예를 들어,

```
b - c
```

라는 계산이 처음 나오면 새 노드를 만들고 value number를 부여한다.

```
1: b
2: c
3: b - c
```

그다음 또 `b - c`가 나오면 새로 만들지 않고, 기존 value number 3을 재사용한다.

이때 노드를 비교하기 위해 **signature**를 사용한다. signature는 대략 다음 형태이다.

```
<op, left, right>
```

예를 들어 `b - c`는 다음처럼 표현할 수 있다.

```
<-, b, c>
```

같은 signature가 이미 있으면 같은 계산이라고 판단한다. 그리고 이를 빠르게 찾기 위해 hash table을 사용한다.

즉 value numbering은 “이 계산을 전에 한 적이 있는가?”를 효율적으로 확인하는 방법이다.

---
# Three address code
중간 표현의 대표적인 형태가 **three-address code**이다. 

![](../images/Pasted%20image%2020260518170021.png)

Three-address code는 하나의 명령어가 최대 세 개의 주소를 사용한다. 일반적인 형태는 다음과 같다.

```
x = y op z
```

여기서 `x`는 결과가 저장되는 곳이고, `y`, `z`는 피연산자이다. `op`는 연산자이다.


예를 들어 다음 식을 보자.

```
x + y * z
```

이 식은 `y * z`를 먼저 계산한 다음 `x`와 더해야 한다. three-address code로 바꾸면 다음처럼 된다.

```
t1 = y * z
t2 = x + t1
```

여기서 `t1`, `t2`는 컴파일러가 만든 임시 변수이다. 원래 소스 코드에 있던 이름이 아니라, 복잡한 연산을 단순한 단계로 나누기 위해 컴파일러가 만들어낸 이름이다.

Three-address code는 AST나 DAG를 일렬로 펼친 표현이라고 볼 수 있다. 트리 구조는 계층적이지만, 실제 실행은 순서가 필요하다. 그래서 컴파일러는 트리나 그래프 형태의 표현을 명령어 나열 형태로 바꾼다.


## Three-Address Code 장점
Three-address code의 장점은 크게 세 가지이다.

첫째, 실제 기계어와 어느 정도 비슷하다. 많은 기계어 명령어도 보통 몇 개의 피연산자를 가지고 계산을 수행한다. 그래서 three-address code는 이후 target code로 바꾸기 쉽다.

둘째, 복잡한 표현식을 단순한 연산 단위로 나눈다. 예를 들어 `a + b * c - d` 같은 식도 여러 개의 임시 변수와 단순 연산으로 분해할 수 있다.

```
t1 = b * c
t2 = a + t1
t3 = t2 - d
```

셋째, 최적화하기 쉽다. 연산이 한 줄씩 나뉘어 있기 때문에 중복 계산, 불필요한 대입, 사용되지 않는 값 등을 분석하기 좋다.

즉 three-address code는 **소스 코드의 복잡한 구조를 컴파일러가 다루기 쉬운 단순 명령어 형태로 바꾼 것**이다.

---
# Address and Instruction
Three-address code에서 address는 여러 종류가 될 수 있다.

첫 번째는 소스 프로그램에 등장하는 변수 이름이다.

```
a, b, c, x, y
```

이런 이름들은 보통 symbol table의 entry를 가리키는 방식으로 구현될 수 있다.

두 번째는 상수이다.

```
1, 2, 3.14
```

세 번째는 컴파일러가 생성한 temporary variable이다.

```
t1, t2, t3
```

그리고 three-address instruction에는 여러 형태가 있다.

```
x = y op z          // 이항 연산
x = op y            // 단항 연산
x = y               // 복사
goto L              // 무조건 점프
if x relop y goto L // 조건 점프
param x             // 함수 인자 전달
call p              // 함수 호출
x = y[i]            // 배열 접근
x[i] = y            // 배열 저장
```

즉 three-address code는 단순 산술 연산뿐 아니라 조건문, 반복문, 배열, 함수 호출까지 표현할 수 있다.

---
# Three-Address Code Example

![](../images/Pasted%20image%2020260518170338.png)

이 코드는 먼저 `i = i + 1`을 실행하고, 그다음 `a[i] < v`가 참이면 다시 반복한다.

이를 three-address code로 표현하면 다음과 같다.

```
L:
    t1 = i + 1
    i = t1
    t2 = i * 8
    t3 = a[t2]
    if t3 < v goto L
```

여기서 `L`은 반복문 시작 위치를 나타내는 label이다.

`i = i + 1`은 바로 한 줄로 표현할 수도 있지만, three-address code에서는 보통 임시 변수를 사용해서 다음처럼 나눌 수 있다.

```
t1 = i + 1
i = t1
```

그리고 `a[i]`에 접근하기 위해 `t2 = i * 8`이 나온다. 이는 배열 원소 하나의 크기가 8바이트라고 가정한 것이다. 배열 접근은 내부적으로 단순히 `a[i]`가 아니라, 다음과 같은 주소 계산이 필요하다.

```
배열 원소 주소 = 배열 시작 주소 + i * 원소 크기
```

그래서 `i * 8`을 계산한 뒤, 그 위치의 값을 `t3`에 넣는다.

마지막으로 조건을 검사한다.

```
if t3 < v goto L
```

조건이 참이면 다시 반복문의 시작으로 이동한다.

---
# Three Address Code: Quadruple
Three-address code를 실제로 저장하는 방법 중 하나가 **quadruple**이다.

![](../images/Pasted%20image%2020260518170914.png)

Quadruple은 하나의 three-address instruction을 네 개의 필드로 저장한다.

```
(op, arg1, arg2, result)
```

예를 들어,

```
t1 = y * z
```

는 다음처럼 표현할 수 있다.

```
(*, y, z, t1)
```

그리고

```
t2 = x + t1
```

는 다음처럼 표현한다.

```
(+, x, t1, t2)
```

Quadruple의 장점은 결과 이름이 명시적으로 존재한다는 것이다. `t1`, `t2` 같은 임시 변수 이름이 분명하게 적혀 있기 때문에 코드 순서를 바꾸거나 최적화할 때 다루기 쉽다.

단점은 공간을 조금 더 많이 쓴다는 것이다. 매 명령어마다 네 개의 필드를 저장해야 하기 때문이다.

---
# Three Address Code: Triple
Triple은 quadruple보다 더 compact한 표현 방식이다.

![](../images/Pasted%20image%2020260518171106.png)

Quadruple은 결과를 저장할 임시 변수 이름을 명시적으로 쓴다. 반면 triple은 결과 이름을 따로 저장하지 않고, **명령어의 index를 결과 이름처럼 사용**한다.

예를 들어 quadruple에서는 다음과 같이 쓴다.

```
t1 = y * z
t2 = x + t1
```

Triple에서는 `t1`, `t2`를 따로 쓰지 않고, 명령어 번호를 참조한다.

```
0: (*, y, z)
1: (+, x, (0))
```

여기서 `(0)`은 0번 명령어의 결과를 의미한다.

Triple의 장점은 임시 변수 이름을 따로 저장하지 않아도 되기 때문에 공간을 덜 쓴다는 것이다. 하지만 단점도 있다. 명령어 순서를 바꾸면 index가 달라질 수 있기 때문에 코드 재배치가 어렵다.

즉 triple은 compact하지만 manipulation이 어렵다.

---
# Three Address Code: Indirect Triples
Indirect triple은 triple의 단점을 보완하기 위한 방식이다.

![](../images/Pasted%20image%2020260518171317.png)

Triple은 명령어 index를 결과 이름처럼 사용하기 때문에 명령어 순서를 바꾸기 어렵다. Indirect triple은 실제 triple table은 그대로 두고, 별도의 pointer list를 사용해서 실행 순서를 관리한다.

즉 실제 명령어는 triple table에 있고, 실행 순서는 별도의 list가 가리킨다.

이렇게 하면 명령어 자체를 직접 옮기지 않고, list의 순서만 바꾸면 된다. 그래서 triple보다 reorder가 쉬워진다.

정리하면 다음과 같다.

> [!note]
> 
> - Triple           : 공간은 덜 쓰지만 순서 변경이 어렵다.
> - Indirect Triple  : triple보다 순서 변경이 쉽다.
> - Quadruple        : 공간은 더 쓰지만 다루기 쉽다.

결국 핵심 trade-off는 **공간 효율성 vs 조작 편의성**이다.

---
# Static Single Assingment Form ( SSA )

SSA의 핵심 규칙은 간단하다.

> 각 변수 이름은 정확히 한 번만 정의된다.

예를 들어 다음 코드가 있다고 하자.

```
p = a + bq = p - cp = q * dp = e - pq = p + q
```

여기서 `p`와 `q`가 여러 번 다시 정의된다. 일반 코드에서는 자연스럽지만, 컴파일러 최적화 입장에서는 변수 값이 언제 바뀌는지 추적하기가 어렵다.

SSA에서는 이를 다음처럼 바꾼다.

```
p1 = a + b
q1 = p1 - c
p2 = q1 * d
p3 = e - p2
q2 = p3 + q1
```

이제 `p1`, `p2`, `p3`는 각각 한 번만 정의된다. 이렇게 하면 데이터 흐름 분석이 쉬워진다. 어떤 값이 어디서 만들어졌고 어디서 사용되는지 명확해지기 때문이다.

## Phi Function
SSA에서 어려운 부분은 if문처럼 제어 흐름이 갈라지는 경우이다.

예를 들어 다음 코드가 있다고 하자.

```
if (flag)
    x = -1;
else
    x = 1;
    
y = x * a;
```

일반 코드에서는 if문 이후에 `x`를 사용한다. 그런데 SSA에서는 각 변수는 한 번만 정의되어야 하므로 다음처럼 나뉜다.

```
if (flag)
    x1 = -1;
else
    x2 = 1;
```

그러면 if문 이후의 `x`는 `x1`일 수도 있고 `x2`일 수도 있다. 이때 사용하는 것이 **Φ-function**, 즉 phi function이다.

```
x3 = Φ(x1, x2)
y1 = x3 * a
```

`Φ(x1, x2)`는 제어 흐름에 따라 들어온 값을 선택한다는 의미이다. 즉 flag가 참인 경로로 왔다면 `x1`, 거짓인 경로로 왔다면 `x2`를 선택한다.

따라서 phi function은 SSA에서 여러 제어 흐름이 다시 합쳐질 때 필요한 장치이다.

---
# Two Address Code
Three-address code와 비교되는 표현으로 **two-address code**가 있다.

Two-address code는 다음과 같은 형태를 가진다.

```
x = x op y
```

즉 결과가 저장되는 위치와 피연산자 하나가 같은 이름을 공유한다.

예를 들어,

```
x = x + y
```

처럼 쓴다.

이 방식은 compact하다는 장점이 있다. 이름을 적게 사용하기 때문이다. 하지만 문제가 있다. 기존 `x` 값을 덮어쓰는 destructive operation이기 때문에 값 관리가 어려울 수 있다.

Three-address code는 다음처럼 결과를 별도의 임시 변수에 둘 수 있다.

```
t1 = x + y
```

이렇게 하면 기존 `x` 값을 유지하면서 새 결과를 만들 수 있다. 그래서 현대 컴파일러에서는 분석과 최적화에 더 편한 three-address form이 중요하게 사용된다.

---
# Control Flow Graph
프로그램은 항상 위에서 아래로만 실행되지 않는다. if문, while문, 함수 호출, goto, return 같은 구조가 있으면 실행 흐름이 바뀐다.

이 흐름을 그래프로 표현한 것이 **Control-Flow Graph**, 줄여서 CFG이다.

![](../images/Pasted%20image%2020260518172417.png)

여기서 주의할 점은 이 CFG는 앞에서 배운 Context-Free Grammar의 CFG와 이름이 같지만 다른 개념이다. 여기서는 **Control-Flow Graph**이다.

Control-flow graph에서 node는 보통 **basic block**이고, edge는 제어 흐름을 나타낸다.

예를 들어 조건문이 있으면 조건이 참일 때 가는 edge와 거짓일 때 가는 edge가 생긴다. 반복문은 다시 위로 돌아가는 edge가 생긴다.

즉 control-flow graph는 “이 코드 다음에 어떤 코드가 실행될 수 있는가?”를 보여주는 그래프이다.

---
# Basic Blocks
Basic block은 중간 코드 명령어들의 연속된 묶음이다.

정확히는 다음 조건을 만족하는 코드 조각이다.

1. 시작 지점으로만 들어온다.
2. 중간에는 밖으로 나가지 않는다.
3. 마지막에서만 분기하거나 다음 블록으로 넘어간다.

쉽게 말하면, 한 번 basic block에 들어오면 중간에 끊기지 않고 끝까지 실행되는 코드 묶음이다.

예를 들어 다음 코드는 하나의 basic block이 될 수 있다.

```
t1 = i + 1
i = t1
t2 = i * 8
t3 = a[t2]
```

하지만 다음 명령어는 조건 분기이므로 block의 끝이 된다.

```
if t3 < v goto L
```

Basic block은 local optimization의 단위가 된다. 즉 컴파일러는 한 basic block 안에서 불필요한 계산을 줄이거나, 임시 변수를 정리하거나, 연산 순서를 바꾸는 최적화를 수행할 수 있다.

---
