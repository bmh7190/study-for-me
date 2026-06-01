
![](../images/Pasted%20image%2020260601113445.png)

parser가 parse tree를 만들면, 그 tree의 각 노드에 `val` 같은 attribute를 붙인다. 그리고 semantic rule에 따라 자식 노드의 값을 이용해서 부모 노드의 값을 계산한다. 이 예제에서는 모든 값이 아래에서 위로 올라가는 synthesized attribute이기 때문에, 숫자 leaf부터 시작해서 최종적으로 root 쪽에서 전체 수식의 값이 계산된다.

---
# Application of Syntax-Directed Translation
앞에서는 `9 + 5 + 2` 같은 수식을 보고 parse tree에 `val` 속성을 붙여서 최종 계산값을 구했다. 즉, 문법 구조를 따라가면서 attribute를 계산하는 방법을 봤다.

그런데 실제 컴파일러는 수식의 결과값만 계산하는 게 목적이 아니다. 컴파일러는 소스 코드를 분석한 뒤에, 이후 단계에서 사용하기 좋은 형태의 중간 표현을 만들어야 한다.

그래서 여기서 등장하는 게 **Abstract Syntax Tree**, 즉 AST다.

![](../images/Pasted%20image%2020260601113849.png)

### Concrete Syntax Tree란?

Concrete Syntax Tree는 보통 **parse tree**라고 보면 된다. parse tree는 문법 규칙이 실제로 어떻게 적용되었는지를 전부 보여준다. 그래서 문법의 nonterminal인 `E`, `T` 같은 노드가 그대로 등장한다.

왼쪽 그림을 보면 루트가 `E`이고, 그 아래에 다시 `E`, `+`, `T`가 있다. 그리고 `T` 아래에 `T * id`가 있는 식이다.

하지만 왼쪽 tree에는 실제 계산에 직접 필요하지 않은 문법 구조도 많이 들어 있다.

예를 들어 `E`, `T` 같은 노드는 연산 자체라기보다는 문법에서 우선순위와 결합 방향을 표현하기 위해 들어간 구조다. 컴파일러가 나중에 코드를 생성할 때는 “이게 E였는지 T였는지”보다 “어떤 연산을 어떤 피연산자에 적용하는지”가 더 중요하다.

그래서 parse tree는 문법 분석 과정을 정확하게 보여주지만, 컴파일러 내부에서 계속 사용하기에는 너무 자세하고 불필요한 정보가 많다.

### Abstract Syntax Tree란?

Abstract Syntax Tree는 parse tree에서 불필요한 문법 정보를 제거하고, 실제 의미 구조만 남긴 tree다.

오른쪽 그림을 보면 루트가 `+`이다. 왼쪽 자식은 `id`, 오른쪽 자식은 `*`이다. 그리고 `*`의 자식으로 `id`, `id`가 있다.

즉 AST는 이 수식을 이렇게 이해한다.

`+` 연산의 왼쪽 피연산자는 `id`이고, 오른쪽 피연산자는 `id * id`이다.

이 구조를 보면 연산 우선순위도 자연스럽게 드러난다. `*`가 `+`보다 아래에 있으므로, `id * id`가 먼저 계산되고 그 결과가 `+`의 오른쪽 피연산자가 된다.

---
# How to build AST?

그러면 **Concrete Syntax Tree를 AST로 어떻게 바꿀까?**

>**parse tree를 그대로 쓰지 말고, SDT의 semantic rule을 이용해서 필요한 AST 노드만 새로 만들어가자**는 것이다.

### AST를 어떻게 만들 것인가
AST를 만들 때는 노드를 크게 두 종류로 생각하면 된다.

하나는 **leaf node**이고, 다른 하나는 **interior node**다.

leaf node는 더 이상 자식이 없는 노드다. 예를 들어 `id`, `num` 같은 것이 leaf node가 된다. 이런 노드는 실제 lexical value를 가지고 있어야 한다.

예를 들어 `id`라면 그냥 “id”라는 종류만 있으면 부족하다. 실제로 어떤 변수 이름인지, symbol table에서 어디를 가리키는지 같은 정보가 필요하다. 그래서 leaf node에는 lexical analyzer가 넘겨준 값이나 symbol table entry 같은 정보가 추가로 저장된다.

그래서 leaf node는 보통 이런 식으로 만든다.

- `Leaf(op, val)` : 여기서 `op`는 노드의 종류이고, `val`은 실제 값이다.

예를 들어 변수 `x`를 나타내는 id라면 대략 이런 느낌이다.

- `Leaf(id, x의 symbol table entry)`

숫자 `10`이라면 이런 식이다.

- `Leaf(num, 10)`

즉 leaf node는 **실제 토큰 값을 들고 있는 끝 노드**라고 보면 된다.

## 내부 노드는 어떻게 만드는가
interior node는 자식이 있는 노드다. AST에서는 보통 연산자 노드가 interior node가 된다. 예를 들어 `id + id * id`에서 `+`는 두 개의 자식을 가진다. 왼쪽 자식은 `id`, 오른쪽 자식은 `*` 노드다.

`*` 노드도 두 개의 자식을 가진다. 왼쪽 자식은 `id`, 오른쪽 자식도 `id`다.

이런 내부 노드는 보통 이렇게 만든다.

`Node(op, c1, c2, ..., ck)` : 여기서 `op`는 연산자이고, `c1`, `c2`, ..., `ck`는 자식 노드들이다.

예를 들어 `id + id * id`의 AST는 대략 이렇게 만들어진다.

`Node('+', Leaf(id, ...), Node('*', Leaf(id, ...), Leaf(id, ...)))`

이렇게 만들면 parse tree에 있던 `E`, `T` 같은 문법용 노드는 사라지고, 실제 의미를 가진 `+`, `*`, `id`만 남는다.

## 왼쪽 parse tree에서 오른쪽 AST로 가는 과정

![](../images/Pasted%20image%2020260601114655.png)

AST를 만들 때는 `E`, `T` 자체가 중요한 게 아니다. 중요한 건 실제 연산 구조다.

그래서 먼저 가장 아래의 `id`들을 leaf node로 만든다.

- `id`  → `Leaf(id, id.entry)`

그다음 `id * id` 부분을 만나면 `*` 노드를 만든다.

- `T → T * id`  → `Node('*', 왼쪽 id 노드, 오른쪽 id 노드)`

그다음 전체 식 `id + id * id`를 만나면 `+` 노드를 만든다.

- `E → E + T`  → `Node('+', 왼쪽 id 노드, 오른쪽 * 노드)`

그래서 최종 AST의 루트는 `+`가 된다.

## 왜 이런 방식이 SDT인가
이 과정이 SDT인 이유는, 문법 규칙마다 AST를 만드는 semantic rule을 붙이기 때문이다.

예를 들어 이런 식이다.

```
E → E1 + T  
E.node = Node('+', E1.node, T.node)
```

이 말은 `E → E1 + T`라는 production이 적용되면, `+`를 루트로 하는 AST 노드를 만들고, 왼쪽 자식으로 `E1.node`, 오른쪽 자식으로 `T.node`를 붙이라는 뜻이다.

또 다른 규칙은 이렇게 된다.

```
T → T1 * F  
T.node = Node('*', T1.node, F.node)

F → id  
F.node = Leaf(id, id.entry)
```

즉 `val` attribute로 계산값을 만들었던 것처럼, 이번에는 `node` attribute로 AST 노드를 만든다. 앞 예제에서는 `E.val`, `T.val`, `F.val`을 계산했다면, 여기서는 `E.node`, `T.node`, `F.node`를 만든다고 보면 된다.

## 중요한 관점
중요한 건 AST를 직접 손으로 그리는 게 아니라, **parse tree를 순회하면서 AST 노드를 attribute로 만들어 올린다**는 점이다.

아래쪽 `id`에서 leaf node가 만들어지고, 그 leaf node들이 위로 전달된다. 그리고 `*`나 `+` 같은 production을 만날 때 내부 노드가 만들어진다.

그래서 이것도 기본적으로 synthesized attribute 방식이다.

자식 노드들의 AST가 먼저 만들어지고, 부모 노드는 그 자식 AST들을 이용해서 자신의 AST를 만든다.

정리하면 이런 흐름이다.

`id`를 만나면 leaf 생성  
`id * id`를 만나면 `*` node 생성  
`id + (id * id)`를 만나면 `+` node 생성  
최종적으로 `E.node`가 전체 AST의 root가 된다.

---
# Ex 5.11

앞에서는 parse tree와 AST의 차이를 봤다. parse tree는 `E`, `T` 같은 문법 구조를 그대로 가지고 있고, AST는 `+`, `-`, `id`, `num`처럼 실제 의미 있는 요소만 남긴다.

이번에는 거기서 한 단계 더 들어가서, **문법 규칙이 적용될 때마다 어떤 semantic rule을 실행해서 AST node를 만들 것인가** 를 예제로 확인해보자

![](../images/Pasted%20image%2020260601115244.png)



![](../images/Pasted%20image%2020260601115255.png)
