Bottom-up parsing 방식의 대표적인 기법으로 **LR(k) 파싱**이 있으며, 일반적으로 특별한 언급이 없으면 **lookahead가 1인 LR(1)** 구조가 많이 사용된다. 

LR 파서의 장점으로는 **대부분의 Context-Free Grammar(CFG)로 표현되는 프로그래밍 언어를 처리할 수 있다는 점**, **백트래킹이 필요 없다는 점**, 그리고 **구문 오류(syntactic error)를 가능한 한 빠르게 탐지할 수 있다는 점**이 있다. 또한 **LL 파서로 파싱 가능한 모든 언어를 LR 파서로도 처리할 수 있다.** 

다만, **파서를 설계하고 파싱 테이블을 생성하는 과정이 복잡하여 많은 노력이 필요하다는 단점**이 있다.

---
# User push down automata
LR Parser는 Pushdown Automata(PDA)를 기반으로 동작하며, 스택을 이용하여 문맥 자유 문법(Context-Free Grammar, CFG)을 효율적으로 파싱한다. 

파서는 다양한 상태(state)를 정의하고, 이러한 상태들을 바탕으로 parsing table을 구성한다. 즉, 현재 상태와 다음 입력 심볼에 따라 어떤 동작을 수행할지 결정하게 된다. 

이때 parsing table은 Action table과 Goto table로 구성되는데, Action table은 현재 상태와 다음 입력 심볼이 주어졌을 때 shift, reduce, accept, error 중 어떤 동작을 수행할지를 정의하며, Goto table은 reduction 이후 특정 비단말 기호에 대해 어떤 상태로 전이해야 하는지를 나타낸다. 

이러한 구조를 통해 LR 파서는 하나의 상태 기계로 동작한다. 비록 CFG 자체는 Finite Automata로 직접 변환될 수 없지만, LR 파싱에서는 handle을 포함하는 viable prefix를 인식하는 DFA를 구성할 수 있다. 

---
# Viable Prefixes
여기서 말하는 Viable prefix란 right-sentential form의 접두사 중에서, 앞으로 reduce될 handle의 오른쪽 끝을 넘지 않는 부분 문자열로, LR 파싱 과정에서 스택에 나타날 수 있는 모든 유효한 문자열을 의미한다.

## The DFA using viable prefix

![](../images/Pasted%20image%2020260417000715.png)

---
# Model of an LR Parser

![](../images/Pasted%20image%2020260417000851.png)

LR Parser가 동작하는 전체 모델은 다음과 같이 설명할 수 있다. 입력 스트림(input stream)에서 토큰을 하나씩 읽어 LR Parsing Program(Driver)으로 전달하며, 이 과정에서 스택을 이용하여 파싱이 진행된다. 파서는 현재 스택의 상태와 다음 입력 심볼을 기반으로 parsing table을 참조하여 어떤 동작을 수행할지 결정한다. 이때 수행 가능한 동작은 **shift**, **reduce**, **accept**, **error**의 네 가지이다. Shift는 다음 입력 심볼을 스택에 추가하는 동작이며, Reduce는 스택의 상단에 있는 심볼들이 특정 생산 규칙의 오른쪽 부분과 일치할 때 이를 왼쪽 부분의 비단말 기호로 치환하는 동작이다. Accept는 입력이 문법적으로 올바를 때 파싱을 종료하는 것을 의미하며, Error는 문법 오류가 발생했음을 나타낸다.

또한, LR 파서는 **Action table**과 **Goto table**을 사용하여 상태 전이를 결정한다. Action table은 현재 상태와 다음 입력 심볼에 따라 수행할 동작을 정의하며, Goto table은 Reduce 이후 비단말 기호에 대해 어떤 상태로 이동할지를 나타낸다. 이러한 상태 전이는 LR(0), SLR, LR(1), LALR(1) 방법을 통해 구성된 DFA(Deterministic Finite Automaton)를 기반으로 이루어진다.

스택에는 단순히 문법 기호만 저장되는 것이 아니라, 각 기호가 스택에 추가될 때의 **상태(state)** 도 함께 저장된다. 실제 구현에서는 상태 번호만 저장해도 되지만, 설명의 편의를 위해 문법 기호와 상태를 함께 표현하기도 한다. 이러한 구조를 통해 LR Parser는 현재 상태와 입력 심볼만으로 다음 동작을 결정할 수 있으며, 백트래킹 없이 효율적인 구문 분석을 수행할 수 있다.

---
# LR Parsing Driver
그림에서 나타난 **driver**는 LR 파서의 핵심 구성 요소로, 파싱 과정을 실제로 수행하는 제어 프로그램을 의미한다.

![](../images/Pasted%20image%2020260417001216.png)

Driver는 입력 스트림, 스택, 그리고 parsing table(Action과 Goto)을 이용하여 파싱을 진행한다. LR 파서의 현재 상태는 스택의 내용과 남아 있는 입력으로 표현되며, 일반적으로 `(s₀ X₁ s₁ X₂ s₂ … Xₘ sₘ, aᵢ aᵢ₊₁ … aₙ $)`와 같은 형태의 configuration으로 나타낸다. 여기서 스택에는 문법 기호와 상태가 번갈아 저장되며, 실제 구현에서는 상태 번호만 저장해도 되지만 설명의 편의를 위해 두 정보를 함께 표현한다. 입력은 아직 처리되지 않은 토큰들의 나열과 끝을 의미하는 `$`로 구성된다.

Driver는 스택의 최상단 상태 `sₘ`과 다음 입력 심볼 `aᵢ`를 이용하여 `action[sₘ, aᵢ]` 값을 참조하고, 그 결과에 따라 동작을 결정한다. 만약 `action[sₘ, aᵢ] = shift s`라면, 입력 심볼 `aᵢ`와 새로운 상태 `s`를 스택에 push하고 입력 포인터를 다음 심볼로 이동시킨다. `action[sₘ, aᵢ] = reduce A → β`인 경우에는 `β`의 길이를 `r = |β|`라 할 때 스택에서 `2r`개의 항목을 pop한 후, 비단말 기호 `A`를 push하고 `goto[sₘ₋ᵣ, A]`를 참조하여 얻은 새로운 상태 `s`를 다시 스택에 push한다. `action[sₘ, aᵢ] = accept`이면 입력 문자열이 문법적으로 올바르다는 의미이므로 파싱을 종료하고, `action[sₘ, aᵢ] = error`인 경우에는 구문 오류가 발생했음을 의미하며 오류 복구 절차를 시도하게 된다.

결과적으로 driver는 현재 상태와 다음 입력 심볼을 기반으로 shift, reduce, accept, error의 네 가지 동작을 결정하며, Action과 Goto 테이블을 이용해 상태 전이를 수행하는 LR 파서의 제어 장치라고 할 수 있다. 이를 통해 LR 파서는 백트래킹 없이 결정적인 방식으로 효율적인 구문 분석을 수행하게 된다.

---
# Example LR(0) parsing table
사실 이렇게 보면 이해가 안될 수 있기 때문에 예시를 통해 알아보자




