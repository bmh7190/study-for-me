## Syntax-Directed Translation
Syntax-Directed Translation은 **context-free grammar를 기반으로 프로그램의 번역 과정을 정의하는 기법**이다. 즉, 문법 구조를 기준으로 프로그램을 해석하고, 그 구조에 따라 필요한 의미 처리나 코드 생성을 수행하는 방식이라고 볼 수 있다. 이 과정은 단순히 문법적으로 올바른 문장인지 확인하는 것에서 끝나지 않고, 프로그램의 구문 구조를 바탕으로 의미를 해석하기 때문에 **semantic processing**이라고도 한다.

예를 들어 parser가 `E → E1 + T`와 같은 문법 구조를 인식했다면, Syntax-Directed Translation에서는 이 구조를 이용해 덧셈 연산의 의미를 처리하거나, 해당 연산에 대한 중간 코드 또는 목적 코드를 생성할 수 있다.

## Syntax-Directed Translation의 목적
Syntax-Directed Translation의 목적은 크게 두 가지로 볼 수 있다. 첫 번째는 **분석을 완성하는 것**이다. syntax analysis는 입력 프로그램이 문법적으로 올바른지만 판단하기 때문에, 문법만으로는 알 수 없는 정보가 존재한다. 예를 들어 `a = b / 0`과 같은 문장은 문법적으로는 올바를 수 있지만, 0으로 나누는 연산은 의미적으로 문제가 될 수 있다. 또한 `A = b + c`에서 `b`와 `c`가 서로 더할 수 없는 타입이라면, 이 역시 syntax analysis만으로는 판단하기 어렵다. 따라서 Syntax-Directed Translation은 이러한 **context-sensitive information**, 즉 문맥에 따라 결정되는 타입, 값, 선언 정보 등을 도출하여 분석 단계를 보완한다.

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