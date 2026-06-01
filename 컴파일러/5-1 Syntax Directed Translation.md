## Syntax-Directed Translation
Syntax-Directed Translation은 **context-free grammar를 기반으로 프로그램의 번역 과정을 정의하는 기법**이다. 즉, 문법 구조를 기준으로 프로그램을 해석하고, 그 구조에 따라 필요한 의미 처리나 코드 생성을 수행하는 방식이라고 볼 수 있다. 이 과정은 단순히 문법적으로 올바른 문장인지 확인하는 것에서 끝나지 않고, 프로그램의 구문 구조를 바탕으로 의미를 해석하기 때문에 **semantic processing**이라고도 한다.

예를 들어 parser가 `E → E1 + T`와 같은 문법 구조를 인식했다면, Syntax-Directed Translation에서는 이 구조를 이용해 덧셈 연산의 의미를 처리하거나, 해당 연산에 대한 중간 코드 또는 목적 코드를 생성할 수 있다.

## Syntax-Directed Translation의 목적
Syntax-Directed Translation의 목적은 크게 두 가지로 볼 수 있다. 

첫 번째는 **분석을 완성하는 것**이다. syntax analysis는 입력 프로그램이 문법적으로 올바른지만 판단하기 때문에, 문법만으로는 알 수 없는 정보가 존재한다. 예를 들어 `a = b / 0`과 같은 문장은 문법적으로는 올바를 수 있지만, 0으로 나누는 연산은 의미적으로 문제가 될 수 있다. 또한 `A = b + c`에서 `b`와 `c`가 서로 더할 수 없는 타입이라면, 이 역시 syntax analysis만으로는 판단하기 어렵다. 따라서 Syntax-Directed Translation은 이러한 **context-sensitive information**, 즉 문맥에 따라 결정되는 타입, 값, 선언 정보 등을 도출하여 분석 단계를 보완한다.

두 번째 목적은 **synthesis를 시작하는 것**이다. 여기서 synthesis는 프로그램을 실제로 번역하여 intermediate representation, 즉 IR이나 target code를 생성하는 과정을 의미한다. 컴파일러의 중요한 역할 중 하나는 소스 프로그램을 다른 형태의 코드로 변환하는 것이므로, Syntax-Directed Translation은 문법 구조에 따라 중간 코드나 목적 코드를 생성하는 출발점이 된다.

## Syntax-Directed Definition과 Semantic Rule
이러한 번역 과정을 형식적으로 정의하기 위해 사용하는 것이 **Syntax-Directed Definition**, 즉 SDD이다. SDD는 문법의 각 production에 semantic rule을 연결하여 attribute의 값을 어떻게 계산할지 정의한다. 여기서 production은 문법 규칙이고, semantic rule은 그 문법 규칙이 적용될 때 수행해야 할 의미 처리 규칙이다.

예를 들어 `E → E1 + T`는 production이고, `E.code → E1.code || T.code || '+'`는 semantic rule이다. 이 semantic rule은 `E1`과 `T`가 가지고 있는 code attribute를 이용해 `E`의 code attribute를 계산한다. 다시 말해, `E1`의 코드와 `T`의 코드를 먼저 생성한 뒤, 마지막에 덧셈 연산을 붙여 `E`의 코드를 만든다는 의미이다.

따라서 SDD는 단순한 문법 정의를 넘어서, 문법 구조에 따라 프로그램의 의미 정보나 번역 결과를 계산하는 방법을 함께 정의하는 개념이라고 정리할 수 있다.

---
# Syntax Directed Definition
Syntax-Directed Definition, 즉 SDD는 **context-free grammar에 attribute와 semantic rule을 함께 결합한 정의**이다. 일반적인 CFG가 프로그램의 문법 구조만 표현한다면, SDD는 그 문법 구조 위에 의미 정보를 추가하여 각 노드가 어떤 값을 가져야 하는지 정의한다. 즉, SDD는 단순히 “이 문장이 문법적으로 맞는가”를 판단하는 데 그치지 않고, parse tree를 따라가며 각 symbol의 의미 정보를 계산할 수 있게 해준다.

SDD에서 terminal과 nonterminal은 각각 attribute를 가질 수 있다. 여기서 attribute는 컴파일러가 처리 과정에서 필요로 하는 값이나 정보를 의미한다. 예를 들어 수식의 계산값, 변수의 타입, symbol table의 위치, 생성된 intermediate code 등이 attribute가 될 수 있다. 이러한 attribute의 값은 production에 연결된 semantic rule에 의해 결정된다. 

![](../images/Pasted%20image%2020260506151114.png)

SDD의 attribute 값은 parse tree를 순회하면서 계산된다. 일반적으로 parse tree를 depth-first traversal 방식으로 순회하며, 각 production에 연결된 semantic rule을 실행하여 attribute 값을 부여한다. 이 순회가 끝나면 parse tree의 각 노드에는 필요한 attribute 값이 채워지게 되고, 최종적으로 root node의 attribute에는 입력 프로그램을 번역한 결과나 의미 분석 결과가 담기게 된다. 따라서 SDD는 parse tree를 단순한 문법 구조가 아니라, 의미 정보가 포함된 annotated parse tree로 확장하는 역할을 한다.

![](../images/Pasted%20image%2020260506151211.png)

## Attribute의 종류
SDD에서 사용하는 attribute는 크게 **synthesized attribute**와 **inherited attribute**로 나눌 수 있다. Synthesized attribute는 어떤 nonterminal의 attribute 값이 해당 노드의 자식 노드들 또는 자기 자신의 정보로부터 계산되는 경우를 말한다. 즉, 정보가 parse tree의 아래쪽에서 위쪽으로 전달된다. 예를 들어 `E.val := E1.val + T.val`에서는 `E1.val`과 `T.val`을 이용해 부모 노드인 `E.val`을 계산하므로, `E.val`은 synthesized attribute이다.

반면 inherited attribute는 어떤 노드의 attribute 값이 부모 노드, 자기 자신, 또는 형제 노드의 정보로부터 전달되어 계산되는 경우를 말한다. 즉, 정보가 반드시 자식에서 부모로만 올라가는 것이 아니라, 부모에서 자식으로 내려오거나 왼쪽 형제에서 오른쪽 형제로 전달될 수 있다. 예를 들어 선언문에서 `int a, b, c`와 같이 하나의 타입 정보가 여러 identifier에 적용되어야 할 때, `int`라는 타입 정보를 뒤쪽의 identifier들에게 전달해야 한다. 이런 경우에 inherited attribute를 사용한다.

정리하면, synthesized attribute는 주로 계산 결과를 위로 올리는 데 사용되고, inherited attribute는 문맥 정보나 타입 정보처럼 주변 구조에서 전달받아야 하는 정보를 표현하는 데 사용된다. 따라서 SDD는 이 두 종류의 attribute를 이용하여 parse tree 위에서 프로그램의 의미 정보와 번역 결과를 체계적으로 계산할 수 있게 해준다.

---
# Bottom Up Evaluation

![](../images/Pasted%20image%2020260601000408.png)

Bottom-up parsing에서는 reduce가 일어나는 순간 semantic rule도 함께 실행할 수 있다. 

그 이유는 synthesized attribute가 production의 오른쪽 symbol, 즉 자식들의 attribute를 이용해서 왼쪽 nonterminal, 즉 부모의 attribute를 계산하는 방식이기 때문이다. 

Bottom-up parsing에서 reduce가 일어난다는 것은 이미 오른쪽 symbol들이 stack 위에 완성되어 있다는 뜻이므로, 그 symbol들의 attribute 값도 사용할 수 있다. 따라서 reduce 시점에 semantic rule을 실행하여 부모의 attribute 값을 바로 계산할 수 있다.

---
# Evaluating an SDD of Parse Tree
SDD를 평가해서 annotated parse tree를 만들려면 attribute 간 의존 관계를 파악해야 한다. 
여기서 annotated parse tree는 단순히 parse tree에 attribute 값  까지 표시한 트리이다.


#### attribute 계산에는 순서가 필요하다
attribute는 아무 순서로나 계산할 수 있는 게 아니다.

예를 들어 `E.val := E1.val + T.val` 이면 `E.val`을 계산하려면 먼저 `E1.val`, `T.val`이 있어야 한다. 그래서 계산 순서는 `E1.val, T.val → E.val` 순서로 해야한다.

즉, 어떤 attribute가 다른 attribute에 의존하면, **의존되는 값이 먼저 계산되어야 한다.**

#### synthesized attribute만 있으면 bottom-up 순서로 가능하다
synthesized attribute는 자식의 값을 이용해서 부모의 값을 계산한다.

```
부모.val := 자식1.val + 자식2.val
```

그래서 parse tree 아래쪽부터 위쪽으로 올라오면 된다. 즉, bottom-up order로 계산하면 항상 자연스럽게 맞는다.

#### inherited attribute까지 있으면 계산 순서가 복잡해진다
inherited attribute는 부모나 형제에게서 내려오거나 옆으로 전달되는 값이다.

예를 들어:

```
D → T L
L.in := T.type
```

이면 `L.in`을 계산하려면 `T.type`이 먼저 있어야 한다.
이 정도는 괜찮다. 왼쪽 형제 `T`를 먼저 계산하고 오른쪽 형제 `L`에게 넘기면 된다.

그런데 inherited와 synthesized가 섞이면, 어떤 값은 위에서 내려오고, 어떤 값은 아래에서 올라오고, 어떤 값은 형제에게 전달된다. 그래서 단순히 “항상 bottom-up” 또는 “항상 top-down”으로 계산할 수 없는 경우가 생긴다.

#### 순환 의존 문제 발생할 수도 있음
이런 production과 semantic rule이 있다.

```
A → BA.s = B.i
B.i = A.s + 1
```

이걸 보면 이상한 점이 있다. `A.s`를 계산하려면 `B.i`가 필요하다. 그런데 `B.i`를 계산하려면 `A.s`가 필요하다. 즉 서로가 서로를 필요로 한다.

이런 관계를 **circular dependency**, 즉 순환 의존이라고 한다.

---
# Inherited attributes Example

![](../images/Pasted%20image%2020260601002824.png)

#### `3 * 5` 계산 흐름
입력 `3 * 5`를 기준으로 보면 흐름은 이렇게 된다.

먼저 `3`을 읽어서

```
F.val = 3
```

그다음 `T → F T'`에서

```
T'.inh = F.val = 3
```

즉 `T'`에게 3을 넘긴다.

이제 `T' → * F T1'`에서 `5`를 읽는다.

```
F.val = 5
```

그리고

```
T1'.inh = T'.inh * F.val
        = 3 * 5
        = 15
```

이제 `T1' → ε`이므로 더 이상 곱셈이 없다.

```
T1'.syn = T1'.inh = 15
```

그 값이 다시 위로 올라간다.

```
T'.syn = T1'.syn = 15
T.val = T'.syn = 15
```

결국 전체 결과는

```
T.val = 15
```

---
# Evaluation Orders for SDD's
앞에서 synthesized attribute만 있을 때는 쉬웠다. 예를 들어 `E.val := E1.val + T.val`처럼 자식 값을 이용해서 부모 값을 계산하면, 그냥 parse tree 아래에서 위로 올라가면 된다. 그래서 bottom-up 순서로 계산하면 됐다.

그런데 inherited attribute가 섞이면 단순히 bottom-up만으로는 안 될 수 있다. 어떤 attribute는 부모에서 자식으로 내려가고, 어떤 attribute는 왼쪽 형제에서 오른쪽 형제로 전달되기 때문이다.

그래서 필요한 게 **evaluation order**, 즉 “attribute들을 어떤 순서로 계산해야 하는가”이다.

이 순서를 정하기 위해 사용하는 것이 **dependency graph**다.

## Dependency graph가 하는 일
Dependency graph는 parse tree 안에서 **attribute 값이 누구에게서 누구로 전달되는지**를 나타낸다. 쉽게 말하면 "어떤 값을 먼저 알아야 하는가?" 를 그래프로 표현하는 것이다.

semantic rule이 이렇게 있다면

```
A.b := X.c
```

`A.b`를 계산하려면 `X.c`가 먼저 필요하다.

그래서 dependency graph에서는

```
X.c → A.b
```

라고 표시한다.

화살표의 의미는 `먼저 필요한 값 → 그 값을 이용해서 계산되는 값`이다.

## Synthesized attribute의 경우

예를 들어 production이 `A → X` semantic rule이 `A.b := X.c` 이면 `A.b`는 synthesized attribute다. 왜냐하면 오른쪽 자식 `X`의 값을 이용해서 왼쪽 부모 `A`의 값을 계산하기 때문이다.

그래서 의존 관계는 `X.c → A.b` 이다.
즉, 자식 attribute가 먼저 계산되고, 그다음 부모 attribute가 계산된다.

## Inherited attribute의 경우

반대로 production이 `A → B` semantic rule이 `B.c := A.a` 이면 `B.c`는 inherited attribute다.
왜냐하면 오른쪽 자식 `B`가 부모 `A`의 정보를 전달받기 때문이다.

이때도 계산 순서는 동일한 방식으로 판단한다. 
`B.c`를 계산하려면 `A.a`가 먼저 필요하니까 `A.a → B.c` 가 된다.

---
# Acyclic Dependency Graphs for Attributed Parse Trees

앞에서 **dependency graph**는 “어떤 attribute가 어떤 attribute에 의존하는지”를 나타내는 그래프라고 했다. 이제 조금 더 구체적으로 보자.

![](../images/Pasted%20image%2020260601005109.png)

#### `A.a := f(X.x, Y.y)`

첫 번째는 이 규칙이다.

```
A.a := f(X.x, Y.y)
```

이 말은 `A.a`를 계산하려면 `X.x`와 `Y.y`가 필요하다는 뜻이다.

따라서 dependency graph는 아래와 같다.

```
X.x → A.a
Y.y → A.a
```

이건 자식 attribute를 이용해서 부모의 attribute를 계산하는 형태이다. 즉 A.a는 synthesized attribute이다. 

---
#### `X.x := f(A.a, Y.y)`

이 말은 `X.x`를 계산하려면 `A.a`와 `Y.y`가 필요하다는 뜻이다.

dependency graph는 아래와 같다.

```
A.a → X.x
Y.y → X.x
```

여기서 `X.x`는 오른쪽 symbol `X`의 attribute다. 오른쪽 자식 attribute가 부모 `A.a`와 형제 `Y.y`의 정보를 받아 계산되는 것이다. 즉, `X.x`는 inherited attribute다.

다만 이 형태는 조심해야 한다. `X`는 `Y`보다 왼쪽에 있는데, `X.x`를 계산하는 데 오른쪽 형제 `Y.y`가 필요하다. 이런 구조는 일반적인 left-to-right evaluation에서는 불편하다.

즉, dependency graph 자체는 cycle이 없을 수 있지만, L-attributed 조건에는 맞지 않을 수 있다.

---
#### `Y.y := f(A.a, X.x)`

이 말은 `Y.y`를 계산하려면 `A.a`와 `X.x`가 필요하다는 뜻이다.

dependency graph는 아래와 같다.

```
A.a → Y.y
X.x → Y.y
```

여기서 `Y.y`도 오른쪽 symbol `Y`의 attribute니까 inherited attribute다.

그런데 이번에는 `Y`가 오른쪽에 있고, 필요한 값이 부모 `A.a`와 왼쪽 형제 `X.x`다.

이건 자연스럽다. 왼쪽에서 오른쪽으로 계산할 때, `A.a`와 `X.x`를 먼저 알고 있으면 `Y.y`를 계산할 수 있기 때문이다.

그래서 이런 구조는 L-attributed definition에서 허용되는 전형적인 형태다.

---
# Ex 5.4

![](../images/Pasted%20image%2020260601005755.png)

`E.val = E1.val + T.val`은 오른쪽 자식들의 attribute를 이용해서 왼쪽 부모 `E`의 attribute를 계산하는 것이므로 synthesized attribute이다.

----
# Ex 5.5

![](../images/Pasted%20image%2020260601010546.png)

![](../images/Pasted%20image%2020260601010540.png)

---
# Ordering the Evaluation of Attributes

앞에서 dependency graph를 만들면 **어떤 attribute가 어떤 attribute에 의존하는지** 알 수 있었다.
그다음 필요한 건 이것이다.

> 의존 관계를 보고 실제로 어떤 순서로 attribute를 계산할 것인가?

즉, `A.a`를 계산하려면 `X.x`가 먼저 필요하고, `X.x`를 계산하려면 `Y.y`가 먼저 필요하다면, 아무 순서로나 계산하면 안 된다. 그래서 dependency graph의 edge를 따라 **계산 순서 evaluation order**를 정해야 한다.

### cycle이 없으면 topological sort 사용

dependency graph에 cycle이 없으면, 계산 순서를 만들 수 있다.
이때 사용하는 방법이 **topological sort**다.

쉽게 말하면

> 아직 계산되지 않은 attribute 중에서, 먼저 필요한 값이 없는 attribute부터 계산한다.

즉, 들어오는 화살표가 없는 노드를 먼저 계산한다.

예를 들어 어떤 노드에 들어오는 edge가 없다는 것은, 그 attribute를 계산하기 위해 먼저 필요한 다른 attribute가 없다는 뜻이다. 그래서 그 노드를 먼저 계산할 수 있다.

그다음 그 노드에서 나가는 edge를 제거하고, 다시 들어오는 edge가 없는 노드를 찾는다. 이런 식으로 순서를 만든다.

위 예시에서 `1, 3, 5, 2, 4, 6, 7, 8, 9`는 이런 방식으로 찾을 수 있는 가능한 evaluation order 중 하나다.

### cycle이 있으면 왜 문제인가

아래 예시를 보면 semantic rule이 이렇게 되어 있다.

```
A.a := f(X.x)
X.x := f(Y.y)
Y.y := f(A.a)
```

의존 관계는 이렇게 된다.

```
X.x → A.a
Y.y → X.x
A.a → Y.y
```

이걸 이어 보면 다시 자기 자신으로 돌아온다. 이 경우는 계산을 시작할 수 없다.

`A.a`를 계산하려면 `X.x`가 필요하고,  `X.x`를 계산하려면 `Y.y`가 필요하고,  `Y.y`를 계산하려면 다시 `A.a`가 필요하다. 즉 서로가 서로를 기다리는 상태가 된다.

그래서 cycle이 있으면 **cyclic dependence error**가 발생한다.

---
# Evaluation Order
Dependency graph에 cycle이 없다면, 이 그래프는 DAG가 되고 **topological sort**를 통해 attribute 계산 순서를 정할 수 있다. 

Topological sort는 방향 그래프의 노드들을 나열할 때, `mi → mj`라는 edge가 있으면 항상 `mi`가 `mj`보다 먼저 오도록 정렬하는 것이다.

즉, dependency graph에서 들어오는 edge가 없는 attribute부터 계산하고, 그 attribute에서 나가는 edge를 제거한 뒤 다시 계산 가능한 attribute를 찾는 방식으로 순서를 만든다. 들어오는 edge가 없다는 것은 그 attribute를 계산하기 위해 먼저 필요한 값이 없다는 뜻이므로 바로 계산할 수 있다.

따라서 dependency graph의 topological sort 결과는 semantic rule을 실행할 수 있는 올바른 evaluation order가 된다. 예를 들어 `1, 3, 5, 2, 4, 6, 7, 8, 9`와 같은 순서는 가능한 evaluation order 중 하나가 될 수 있다.

---
# Example Annotated Parse Tree

![](../images/Pasted%20image%2020260601011639.png)

먼저 `T → real` 때문에 `T.type = 'real'` 이 만들어진다.

그다음 `D → T L`에서 `L.in = T.type` 이므로 오른쪽 `L`은 다음 값을 받는다.

```
L.in = 'real'
```

이제 이 `L.in` 값이 변수 목록 안으로 계속 전달된다.


그림은 결국 이런 선언을 처리하는 과정이다.

```
real id1, id2, id3
```

컴파일러 입장에서는 `real`을 보고 타입을 만든 다음, symbol table에 각각의 identifier를 다음처럼 등록해야 한다.

```
id1 : real
id2 : real
id3 : real
```

그래서 semantic action은 대략 이런 느낌이 된다.

```
addtype(id1.entry, real)
addtype(id2.entry, real)
addtype(id3.entry, real)
```

## 왜 inherited attribute인가

`id1`, `id2`, `id3`는 자기 자신만 봐서는 타입을 모른다.
얘네만 보면 그냥 identifier일 뿐이고, 타입이 `int`인지 `real`인지 알 수 없다.

타입 정보는 왼쪽에 있는 `T.type = 'real'`에서 온다.

그래서 그 정보를 `L.in`이라는 inherited attribute로 아래쪽 `L`들에게 전달하는 것이다.

---
# Cycle free evaluation order

앞에서 dependency graph를 만들고, cycle이 없으면 topological sort로 evaluation order를 정할 수 있다고 했다. 그런데 문제는 매번 모든 parse tree에 대해 dependency graph를 만들고, cycle이 있는지 검사하는 것이 현실적으로 번거롭다는 점이다.

그래서 실제 컴파일러 설계에서는 **처음부터 cycle이 생기지 않는 형태의 SDD**를 사용하려고 한다.

그 대표적인 두 종류가 S-attributed SDD, L-attributed SDD 이다.

## S-attributed SDD란?
S-attributed SDD는 **모든 attribute가 synthesized attribute인 SDD**이다.
즉, 모든 semantic rule이 오른쪽 자식들의 값을 이용해서 왼쪽 부모의 attribute를 계산하는 형태다.

예를 들면 여기서 `E.val`은 자식 `E1.val`, `T.val`로부터 계산된다.

```
E → E1 + T
E.val = E1.val + T.val
```

#### 왜 cycle-free가 보장되는가
S-attributed SDD에서는 값이 항상 아래에서 위로만 올라간다. 
즉, 어떤 부모 attribute를 계산하려면 자식 attribute들이 먼저 필요하고, 자식 attribute들은 다시 그 아래 자식들로부터 계산된다.

이 구조에서는 값이 다시 아래로 내려가거나 옆으로 돌아가는 흐름이 없다. 그래서 dependency graph에 cycle이 생기지 않는다. 쉽게 말하면 `leaf → 내부 노드 → root` 방향으로만 계산이 진행된다.

#### 계산 순서
S-attributed SDD는 **bottom-up order**로 계산하면 된다. parse tree 기준으로는 **postorder traversal**과 잘 맞는다. Postorder traversal은 자식 먼저 방문 → 부모 나중에 방문하는 방식이다. 

이게 synthesized attribute 계산과 정확히 맞다.

예를 들어

```
E → E1 + T
E.val = E1.val + T.val
```

이면 `E.val`을 계산하기 전에 `E1.val`, `T.val`이 먼저 계산되어 있어야 한다.
Postorder traversal에서는 자식을 먼저 처리하므로 자연스럽게 이 순서가 만족된다.

#### 왜 LR parser에 적합한가
LR parser는 bottom-up parsing 방식이다. 즉, production의 오른쪽 body가 stack 위에 완성되면 그것을 왼쪽 nonterminal로 reduce한다.

S-attributed SDD에서는 reduce 시점에 오른쪽 symbol들의 attribute가 이미 준비되어 있다. 그래서 reduce와 동시에 semantic rule을 실행해서 왼쪽 nonterminal의 attribute를 계산할 수 있다.

예를 들어:

```
T → T * F
T.val = T1.val * F.val
```

LR parser가 `T * F`를 `T`로 reduce하는 순간, stack에는 이미 `T.val`과 `F.val`이 있다.
그래서 바로 `T.val = T1.val * F.val` 를 계산할 수 있다.

---
## L-attributed
L-attributed definition은 dependency graph의 정보 흐름이 **left to right**, 즉 왼쪽에서 오른쪽으로 진행되는 SDD다.

production이 다음과 같다고 하자.

```
A → X1 X2 ... Xn
```

여기서 어떤 오른쪽 symbol `Xi`의 inherited attribute `Xi.a`를 계산한다고 하면, `Xi.a`는 아무 attribute나 참조하면 안 된다.

허용되는 것은 두 가지다.

첫 번째, head인 `A`의 inherited attribute를 사용할 수 있다.

```
A의 attribute → Xi.a
```

두 번째, `Xi`보다 왼쪽에 있는 symbol들의 attribute를 사용할 수 있다.

```
X1, X2, ..., Xi-1의 attribute → Xi.a
```

즉 `Xi.a`를 계산할 때는 **부모 A의 정보** 또는 **자기보다 왼쪽에 있는 형제들의 정보**만 사용할 수 있다.

#### 왜 오른쪽 형제는 안 되는가

예를 들어 production이 `A → X1 X2` 라고 하자.

`X2.x`를 계산할 때 `A.a`나 `X1.x`를 쓰는 건 괜찮다.

```
X2.x := f(A.a, X1.x)
```

왜냐하면 왼쪽에서 오른쪽으로 처리하면 `A.a`와 `X1.x`를 먼저 알고 나서 `X2.x`를 계산할 수 있기 때문이다.

반대로 `X1.x`를 계산하는데 오른쪽 형제인 `X2.x`가 필요하면 문제가 된다.

```
X1.x := f(A.a, X2.x)
```

이러면 `X1`을 계산하려고 하는데 아직 오른쪽 `X2`를 보지도 않았거나 계산하지 않은 상태일 수 있다. 그래서 left-to-right 순서가 깨진다.

즉, L-attributed에서는 **오른쪽에서 왼쪽으로 값을 전달하는 의존성은 허용하지 않는다.**

#### Example

![](../images/Pasted%20image%2020260601014610.png)

이 예시는 L-attributed 조건을 만족한다.

`X.i`는 부모 `A.i`만 사용한다.

```
X.i := A.i
```

`Y.i`는 왼쪽 형제 `X.s`만 사용한다.

```
Y.i := X.s
```

`A.s`는 오른쪽 자식 `Y.s`를 사용해서 부모로 올라가는 synthesized attribute다.

```
A.s := Y.s
```

오른쪽 형제에서 왼쪽 형제로 가는 의존성이 없다. 즉 `X.i := Y.s` 같은 형태가 없다.
그래서 left-to-right 순서로 계산 가능하다.

#### S-attributed와 L-attributed 관계
S-attributed는 모든 attribute가 synthesized인 경우다. L-attributed는 synthesized attribute도 허용하고, inherited attribute도 제한적으로 허용한다.

그래서 관계를 보면

```
S-attributed ⊂ L-attributed
```

즉, 모든 S-attributed definition은 L-attributed definition이기도 하다.

왜냐하면 synthesized attribute는 자식에서 부모로 올라가는 값이라, L-attributed의 제한을 깨지 않기 때문이다.

----
# Using Translation Schemes for L-Attributed Definitions

앞에서 L-attributed definition은 **부모나 왼쪽 형제의 정보를 이용해서 inherited attribute를 계산할 수 있다**고 했다.

그런데 실제 parser가 동작할 때는 semantic rule을 아무 위치에서나 실행할 수 없다.  
특히 inherited attribute는 **해당 nonterminal을 처리하기 전에 미리 값이 준비되어 있어야 한다.**

그래서 SDD의 semantic rule을 production 안에 직접 끼워 넣어서, **언제 semantic action을 실행할지 명확하게 표시한 형태**가 필요하다.

이게 바로 **translation scheme**이다.

![](../images/Pasted%20image%2020260601015111.png)

Translation Scheme은 semantic rule을 production 내부에 `{ }`로 끼워 넣은 형태다.

```
D → T { L.in := T.type } L
```

이렇게 쓴 이유는 `L`을 처리하기 전에 `L.in` 값이 먼저 계산되어야 하기 때문이다.

흐름은 다음과 같다.

1. 먼저 `T`를 처리한다.
2. `T.type`이 계산된다.
3. `L`을 처리하기 전에 `{ L.in := T.type }`을 실행한다.
4. 그다음 `L`을 처리한다.

즉 `{ L.in := T.type }`의 위치가 중요하다.

만약 이 action을 `L` 뒤에 두면

```
D → T L { L.in := T.type }
```

이건 의미가 이상해진다. `L`을 처리할 때 이미 `L.in`이 필요했는데, `L` 처리가 끝난 뒤에야 값을 넣는 꼴이기 때문이다.

---
### 각 production 해석

#### 1. `D → T { L.in := T.type } L`

`T`에서 타입을 계산한 뒤, 그 타입을 `L`에게 넘긴다.

```
T.type → L.in
```

예를 들어 `T.type = integer`이면 `L.in = integer` 이 된다.

---
#### 2. `T → int { T.type := 'integer' }`

`int`라는 타입 키워드를 보면 `T.type`을 `'integer'`로 만든다.

```
T.type = integer
```

이건 synthesized attribute다. `int`를 보고 부모 `T`의 type을 계산하기 때문이다.

---
#### 3. `T → real { T.type := 'real' }`

`real`을 보면 `T.type`을 `'real'`로 만든다.

```
T.type = real
```

---
#### 4. `L → { L1.in := L.in } L1 , id { addtype(id.entry, L.in) }`

이게 조금 중요하다. `L → L1 , id`는 identifier 목록을 처리하는 production이다.

예를 들어 `id1, id2, id3` 같은 목록을 처리할 때 사용된다.

여기서 바깥쪽 `L`이 이미 타입 정보를 가지고 있다.

```
L.in = real
```

그런데 안쪽 `L1`도 같은 타입 정보를 알아야 한다.

그래서 `L1`을 처리하기 전에:

```
{ L1.in := L.in }
```

을 실행한다.

그다음 `L1`을 처리하고, 마지막 `id`에 대해서

```
addtype(id.entry, L.in)
```

을 실행한다. 즉 현재 id에도 같은 타입을 붙인다.

---
#### 5. `L → id { addtype(id.entry, L.in) }`

identifier가 하나만 있는 경우다.

예를 들어 `real id1` 이면 `L.in = real`을 이용해서 `addtype(id1.entry, real)` 을 실행한다.

----
# Ex 5.8 and 5.9

앞에서 **L-attributed definition**은 inherited attribute를 허용하지만, 조건이 있다고 했다.
production이 `A → X1 X2 ... Xn`일 때, 어떤 `Xi`의 inherited attribute는 다음 값만 사용할 수 있다.

1. 부모 A의 inherited attribute
2. Xi보다 왼쪽에 있는 symbol들의 attribute

즉, **오른쪽 형제의 값을 이용해서 왼쪽 symbol의 inherited attribute를 계산하면 안 된다.**

---
####  `T → F T'`

```
T → F T'
T'.inh = F.val
```

여기서 `T'.inh`는 오른쪽 symbol `T'`의 inherited attribute다.

`T'.inh`를 계산할 때 사용하는 값은 `F.val`이다.

```
F.val → T'.inh
```

`F`는 `T'`보다 왼쪽에 있는 symbol이다. 따라서 `T'.inh = F.val`은 L-attributed 조건을 만족한다.

즉, 앞에서 계산된 `F.val`을 오른쪽의 `T'`에게 넘기는 구조다.

---
#### `T' → * F T1'`

```
T' → * F T1'
T1'.inh = T'.inh × F.val
```

여기서 `T1'.inh`는 오른쪽 끝에 있는 `T1'`의 inherited attribute다.

이 값을 계산할 때 사용하는 값은 `T'.inh`, `F.val` 이다.

`T'.inh`는 부모 `T'`의 inherited attribute이고, `F.val`은 `T1'`보다 왼쪽에 있는 symbol의 attribute다. 따라서 이것도 L-attributed 조건을 만족한다.

즉, 지금까지 누적된 값 `T'.inh`와 현재 숫자 `F.val`을 곱해서 다음 `T1'`에게 넘기는 구조다.

---
#### `A → B C`

이 grammar가 L-attributed인가?

```
A → B C
A.s = B.b
B.i = f(C.c, A.s)
```

먼저 `A.s = B.b`는 synthesized attribute 계산이다.

```
B.b → A.s
```

자식 `B`의 값을 이용해서 부모 `A`의 값을 계산하므로 이 자체는 문제 없다.

그런데 문제는 `B.i = f(C.c, A.s)` 이다.

`B.i`는 오른쪽 symbol `B`의 inherited attribute다. 그런데 이 값을 계산하기 위해 `C.c`를 사용하고 있다. 여기서 `C`는 `B`보다 오른쪽에 있는 symbol이다. 즉, `B.i`를 계산하려면 오른쪽 형제 `C.c`가 필요하다.

이건 오른쪽에서 왼쪽으로 정보가 흐르는 구조다.

L-attributed에서는 inherited attribute를 계산할 때 오른쪽 형제의 attribute를 사용할 수 없기 때문에, 이 grammar는 **L-attributed가 아니다.**