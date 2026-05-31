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
