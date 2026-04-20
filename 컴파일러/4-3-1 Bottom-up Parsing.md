
Bottom-up parsing은 입력 스트림 $w$ 와 문법 $G$ 가 주어졌을 때, parse tree를 leaf node에서 시작해서 root 방향으로 구성하는 방법이다. 

이 과정은 rightmost derivation을 역으로 복원하는 방식이다. LR parsing은 입력을 left to right로 읽으면서 rightmost derivation in reverse를 수행하는 방법이며, 대표적인 방식으로 shift-reduce parsing, SLR, Canonical LR, LALR이 있다. 

Bottom-up parsing에서는 reduction이라는 개념이 핵심인데, 이는 현재 입력으로부터 얻은 문자열을 생산 규칙의 오른쪽 부분과 대응시켜 점점 nonterminal로 줄여 가는 과정이다. 이때 아무 오른쪽 부분이나 reduce하는 것이 아니라, rightmost derivation의 역과정에서 올바른 **handle**에 해당할 때만 reduce해야 한다. 입력을 모두 읽고 최종적으로 시작 심볼까지 reduce되면 parsing이 성공한다. 또한 parser는 현재까지의 stack 상태와 다음 입력 심볼을 보고 shift할지 reduce할지를 결정한다.

---
## Example

![](../images/Pasted%20image%2020260416161255.png)

기존의 파싱 방식인 **LL 파싱**은 입력을 **왼쪽에서 오른쪽으로(left-to-right)** 읽으면서, **Leftmost Derivation**을 사용하여 **가장 왼쪽의 nonterminal부터 확장**하는 **Top-Down 방식**으로 진행된다. 즉, 시작 심볼에서 출발하여 입력 문자열을 생성해 나가는 방식이다.

반면, **LR 파싱**(SLR, Canonical LR, LALR 등)은 입력을 동일하게 **왼쪽에서 오른쪽으로 읽지만**, **Rightmost Derivation의 역과정(rightmost derivation in reverse)** 을 사용한다. 이 방식은 **Bottom-Up 방식**으로, 완성된 입력 문자열에서 시작하여 **handle을 찾아 reduction을 반복함으로써** 최종적으로 시작 심볼에 도달한다.

따라서 LR 파싱은 LL 파싱을 단순히 역으로 수행하는 것이 아니라, **Rightmost Derivation을 거꾸로 복원하는 과정**이라고 이해하는 것이 더 정확하다.

### Handle 

![](../images/Pasted%20image%2020260416162009.png)


여기서 **핸들(handle)** 이라는 개념이 등장한다. 핸들은 **오른쪽 문장형(right-sentential form)** 에서 **생산 규칙의 오른쪽 부분(RHS)** 과 일치하는 부분 문자열로, 이를 해당 생산 규칙의 **왼쪽 부분(LHS)** 으로 축약(reduction)하면 **rightmost derivation의 이전 단계로 되돌릴 수 있는 문자열**을 의미한다.

즉, 그림에서 볼 수 있듯이 전체 문자열 중에서 **그래머의 생산 규칙을 적용하여 하나의 non terminal 기호로 치환할 수 있는 부분 문자열**을 핸들이라고 한다. 이러한 핸들을 순차적으로 축약해 나가면, 입력 문자열을 시작 기호로 환원할 수 있으며, 이는 **bottom-up parsing(하향식 파싱)**의 핵심 개념이다.

---
# Shift and Reduce on Bottom up Parsing
Bottom-up parsing에서는 다음 입력 심볼(lookahead)을 기반으로 두 가지 동작 중 하나를 결정한다.

#### Shift

- **정의**: 다음 입력 심볼을 스택(stack)의 top에 추가하는 동작.
- **사용 시점**: 아직 스택의 top이 어떤 생산 규칙의 RHS와 일치하지 않을 때.
- **목적**: 더 많은 입력을 읽어 handle을 형성하기 위함.

#### Reduce

- **정의**: 스택의 top에 있는 심볼들이 어떤 생산 규칙 `A → β`의 RHS인 `β`와 일치할 때, 이를 `A`로 치환하는 동작.
- **사용 시점**: 스택의 top이 **handle**을 형성했을 때.
- **목적**: rightmost derivation의 이전 단계로 되돌아가기 위함.

#### 기타 동작

- **Accept**: 입력이 모두 처리되고 스택에 시작 기호만 남았을 때.
- **Error**: 어떤 규칙도 적용할 수 없는 경우.

### Bottom-Up Parsing 과정 요약

1. **입력 문자열을 왼쪽에서 오른쪽으로 읽는다.**
2. **스택의 top이 handle을 형성하면 Reduce를 수행한다.**
3. **handle이 형성되지 않으면 Shift를 수행한다.**
4. 이 과정을 반복하여 **시작 기호만 남으면 Accept**한다.

---
## Examples - shift reduce parser 

우선 예시를 통해 알아보기 전에 정리를 해보면,

일단 스택을 초기화하고 `$`를 넣는다. 이는 스택이 비어 있는 상태를 의미하는 경계 표시자이다.

그 다음, 스택의 top이 목표한 심볼인 시작 기호(Start Symbol)가 되고 동시에 입력 토큰이 `$`가 될 때까지 다음 과정을 반복한다.

- 먼저 handle을 찾는다. 스택의 top에 handle이 없는 경우에는 다음 입력 심볼을 스택에 넣는 **shift** 동작을 수행한다.

- 반대로 handle이 존재하는 경우에는 **handle pruning**, 즉 **reduce**를 수행한다. 이때 handle에 해당하는 심볼들을 스택에서 pop한 후, 그 대신 해당 생산 규칙의 왼쪽 부분에 해당하는 **nonterminal**을 스택에 push한다.

---
## Examples 1  

![](../images/Pasted%20image%2020260416164136.png)

첫 번째 예시로 `id * id`가 input stream에 들어왔다고 해보자. 이때 스택은 `$`만 들어 있는 초기화 상태이다. 처음 action은 당연히 **shift**를 수행하여 입력 심볼 `id`를 스택에 넣는다.

그 다음에는 **reduce**를 수행할 수 있는지 확인한다. 주어진 그래머에 `F → id`라는 생산 규칙이 있으므로, 스택의 `id`를 pop하고 대신 `F`를 push한다. 이어서 `T → F`라는 생산 규칙을 적용할 수 있으므로 `F`를 pop하고 `T`를 push한다.

이후 이 예제에서는 **shift**를 수행하여 `*`를 스택에 넣는다. 다시 handle이 존재하는지 확인하지만 아직 reduce할 수 없으므로 한 번 더 **shift**를 수행하여 다음 입력 심볼 `id`를 스택에 넣는다. 그러면 다시 `F → id` 규칙을 이용해 `id`를 `F`로 reduce하고, 이어서 `T → T * F` 규칙을 적용하여 `T * F`를 `T`로 reduce한다. 마지막으로 `E → T` 규칙을 적용하면 시작 심볼인 `E`가 되어 **accept** 상태에 도달한다.

대충 어떤 흐름인지 이해할 수 있을 것이다. 여기서 한 가지 의문이 생길 수 있는데, 사실 중간 단계에서 `T`를 `E`로 먼저 reduce할 수도 있었다. 그러나 그렇게 하지 않고 `shift`를 통해 `T * F` 형태가 완성된 이후에 reduce를 수행하였다. 이러한 **shift와 reduce 사이의 선택 문제**는 이후에 SLR, LR, LALR과 같은 방법을 통해 해결할 수 있으며, 지금은 bottom-up parsing의 동작 방식에 집중하도록 하자.

---

## Example 2 

![](../images/Pasted%20image%2020260416164758.png)

이 예시에서도 동일하다. 처음에는 `$`로 초기화된 스택에서 시작하여, 입력 스트림에서 심볼을 하나씩 꺼내면서 **reduce**를 수행할 수 있는지, 즉 **handle**이 존재하는지를 확인한다. 만약 handle을 찾을 수 없다면 **shift**를 수행하여 입력 심볼을 스택에 넣고, 이 과정을 반복한다. 이러한 과정을 통해 입력 문자열을 점차적으로 축약해 나가며, 최종적으로 입력의 끝을 나타내는 `$`가 도달하고 스택의 top이 시작 기호(Start Symbol)가 되면 **accept** 상태에 도달하게 된다.

---
# Issues in Shift Reduce Parsing
예시를 통해 생각해보면, 이 **shift-reduce parsing**에는 여러 가지 문제점이 존재한다.

#### 오래 걸릴 수 있다.
먼저, **linear time** 에 수행하기 어렵다는 점이다. 스택에는 handle이 하나만 존재하는 것이 아니라 여러 개가 될 수 있기 때문에, 어떤 부분이 실제로 축약되어야 하는 handle인지 판단하기 위해 **스택 전체를 탐색해야 할 수도 있다**. 이 과정에서 가능한 모든 경우를 고려해야 하므로 **backtracking**이 발생할 수 있으며, 이는 파싱의 효율성을 크게 저하시킨다.

#### 충돌이 발생할 수 있다.
**Shift/Reduce Conflict**  
다음 동작으로 shift를 해야 할지, reduce를 해야 할지 결정할 수 없는 경우를 의미한다. 대표적인 예로는 _dangling else_ 문제인 `if E then if E then S else S`와 같은 문장이 있다. 이 경우 `else`를 어떤 `if`와 결합해야 하는지 모호해진다.

**Reduce/Reduce Conflict**  
두 개 이상의 생산 규칙 중 어떤 규칙으로 reduce해야 할지 결정할 수 없는 경우이다. 
예를 들어, Fortran에서 `if(id)`는 **배열 변수 참조**로 해석될 수도 있고 **if 문**으로 해석될 수도 있어 모호성이 발생한다.

이러한 문제들을 해결하기 위해 **SLR, Canonical LR, LALR**과 같은 보다 강력한 LR 파싱 기법이 사용되며, 이들은 상태 정보와 lookahead를 활용하여 충돌을 효과적으로 해결한다.

---
## Shift Reduce Parsing : Shift Reduce Conflicts
이 그림은 **shift/reduce conflict**의 대표적인 예인 _dangling else problem_을 설명한다. 

주어진 문법은 다음과 같다.

S → if E then S  
  | if E then S else S  
  | other


![](../images/Pasted%20image%2020260416165425.png)


파싱 과정에서 스택에 `if E then S`가 쌓여 있고, 다음 입력 심볼이 `else`인 상황을 생각해 보자. 이때 파서는 두 가지 선택지 중 하나를 결정해야 한다.

첫 번째는 **reduce**를 수행하는 경우이다. 즉, `if E then S`를 하나의 `S`로 축약할 수 있다. 그러나 이렇게 하면 이후에 등장하는 `else`가 앞의 `if`와 연결되지 못하게 되어 문장의 의도와 다른 해석이 될 수 있다.

두 번째는 **shift**를 수행하는 경우이다. 즉, `else`를 스택에 추가하여 `if E then S else S` 형태로 확장한 뒤, 이후에 하나의 `S`로 축약하는 방식이다. 이 경우 `else`는 가장 가까운 `if`와 결합하게 된다.

따라서 이 상황에서의 핵심적인 의문은, `else`가 등장했을 때 **이미 완성된 것처럼 보이는 `if E then S`를 reduce해야 하는지**, 아니면 **앞에 또 다른 `else`가 이어질 가능성을 고려하여 shift해야 하는지**에 대한 것이다. 이러한 모호성 때문에 **shift/reduce conflict**가 발생한다.

일반적으로 LR 파서에서는 이 문제를 해결하기 위해 **shift를 우선적으로 선택**한다. 그 결과, `else`는 가장 가까운 `if`와 결합하게 되며, 이는 대부분의 프로그래밍 언어에서 채택하고 있는 해석 방식이다.

정리하면, 이 그림은 `if E then S` 뒤에 `else`가 등장했을 때 **reduce와 shift 중 어떤 동작을 선택해야 하는지 결정할 수 없는 상황**, 즉 **shift/reduce conflict**를 나타내며, 이를 해결하기 위해 보통 **shift를 선택하여 else가 가장 가까운 if와 매칭되도록 한다**는 점을 보여준다.

---
## Shift-Reduce Parsing: Reduce-Reduce Conflicts

![](../images/Pasted%20image%2020260416165658.png)

이 예시에서 보면 `a`가 스택에 들어왔을 때, 이 `a`와 관련된 그래머가 `A → a`와 `B → a` 두 가지가 존재할 수 있다. 따라서 이 두 생산 규칙 중 어떤 규칙을 선택하여 **reduce**를 수행해야 하는지 결정하기 어려운 문제가 발생한다.

즉, 스택의 top에 있는 `a`는 두 비단말 기호 `A`와 `B` 모두로 축약될 수 있으므로, 파서는 어떤 규칙을 적용해야 할지 판단할 수 없게 된다. 이러한 상황을 **reduce/reduce conflict**라고 하며, 두 개 이상의 생산 규칙이 동일한 입력에 대해 적용 가능할 때 발생한다.

이러한 충돌은 문법이 모호하거나, 파싱 테이블을 구성할 때 충분한 문맥 정보가 제공되지 않을 경우에 발생한다. 이를 해결하기 위해서는 더 강력한 파싱 기법인 **SLR**, **Canonical LR**, 또는 **LALR** 파서를 사용하여 **lookahead** 정보를 기반으로 올바른 축약을 선택할 수 있도록 해야 한다.

---
