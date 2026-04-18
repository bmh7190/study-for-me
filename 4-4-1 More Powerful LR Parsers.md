# SLR and Ambiguity
SLR과 모호성의 관계를 보면, 모든 SLR 문법은 모호하지 않다. 즉, SLR parsing table이 충돌 없이 구성되는 문법은 반드시 unambiguous하다. 그러나 그 반대는 성립하지 않는다. 즉, 모호하지 않은 문법이라도 SLR 방식으로는 parsing table을 만들 때 충돌이 발생할 수 있다.

예시를 통해서 알아보자

![](../images/Pasted%20image%2020260417141117.png)

이 문법을 보면 **FOLLOW(R)** 에 `=`가 포함되어 있다. 따라서 SLR의 규칙에 따르면, 상태에 `R → L •`이 존재할 때 입력 기호가 `=`이라도 **reduce를 수행하는 것이 허용된다.**

하지만 동시에 같은 상태에 `S → L • = R`이 존재하므로, 입력이 `=`일 경우에는 `=`를 읽고 다음 상태로 이동하는 **shift 동작도 가능하다.**

즉, 입력이 `=`일 때 **shift와 reduce가 모두 가능한 상황**이 발생하게 되며, SLR 파서는 이 둘 중 어느 것을 선택해야 할지 결정할 수 없다. 이로 인해 **shift/reduce conflict**가 발생한다.

직관적으로 보면, 현재의 `L`이 이미 하나의 `R`로 완성된 것이라고 볼 수도 있고(reduce), 아직 `L = R` 형태의 더 큰 구조로 확장될 수도 있기 때문에(shift), 두 가지 해석이 동시에 가능해 보이는 상태라고 이해할 수 있다.

---
# LR(1) Grammars
LR(1) 문법은 SLR에서 사용한 LR(0) automata를 확장한 방식이다. SLR에서는 FOLLOW 집합을 이용해 reduce 여부를 판단했지만, 이 과정에서 불필요한 충돌이 발생할 수 있었다. 이를 해결하기 위해 LR(1)에서는 **lookahead symbol(다음 입력 기호)** 을 추가로 사용하여 보다 정확하게 parsing 동작을 결정한다.

즉, LR(1) parsing에서는 단순히 현재 상태뿐만 아니라, **다음에 올 입력 기호까지 함께 고려**하여 shift와 reduce를 구분하게 된다. 이로 인해 SLR에서 발생하던 불필요한 충돌을 대부분 제거할 수 있다.

LR(1) item은 기존의 LR(0) item에 lookahead를 추가한 형태로 표현된다.  
LR(0) item이 `[A → α • β]`였다면, LR(1) item은 `[A → α • β, a]`와 같이 **lookahead 기호 a가 함께 포함된다.**


또한 일반적으로 “LR parser”라고 하면 **canonical LR(1) parser**를 의미하며, 이는 가장 강력한 형태의 LR parser이다. 다만 LR(1)은 상태 수가 매우 많아질 수 있기 때문에, 이를 줄이기 위해 LR(0) 기반 구조를 유지하면서 상태 수를 줄인 **LALR parser**가 실제로 많이 사용된다.

---
# What is a problem?
lookahead symbol의 목적은 parsing 과정에서 **여러 개의 reduce 후보가 있을 때, 어떤 것을 선택해야 하는지를 결정하기 위한 기준을 제공하는 것**이다. 즉, 단순히 현재 상태만 보는 것이 아니라, **다음에 들어올 입력 기호를 함께 고려하여 올바른 reduce를 선택할 수 있게 한다.**

LR(1) item에서 lookahead는 항상 중요한 역할을 하는 것은 아니며, 점이 아직 오른쪽 끝에 도달하지 않은 경우에는 단순히 정보를 저장하는 역할만 한다. 예를 들어 `[A → X • Y Z, a]`와 같은 상태에서는 아직 reduce 단계가 아니므로, lookahead `a`는 직접적으로 사용되지 않는다. 반면 `[A → X Y Z • , a]`처럼 점이 끝에 있는 경우에는, 입력이 `a`일 때만 reduce를 수행할 수 있으므로 lookahead가 실제로 중요한 역할을 하게 된다.

또한 lookahead를 사용하면, 동일한 형태의 우변을 가지는 여러 생산 규칙이 존재하더라도, **다음 입력 기호를 기준으로 어떤 비단말로 reduce해야 하는지를 구분할 수 있다.**

---
# SLR VS LR(1)
SLR에서는 하나의 상태 안에 여러 개의 item이 함께 존재하면서, 동일한 입력 기호에 대해 shift와 reduce가 동시에 가능한 상황이 발생할 수 있었다. 이를 해결하기 위해 LR(1)에서는 각 item에 lookahead를 추가하여, 하나의 상태를 더 세분화하여 나누게 된다.

즉, 기존 SLR의 하나의 상태를 그대로 사용하는 것이 아니라, **각 item이 가지는 lookahead에 따라 상태를 분리(split)** 하여 충돌을 제거한다.

예를 들어, 기존 상태 $I_2$에는 다음 두 item이 함께 존재한다:

S → L • = R  
R → L •

SLR에서는 이 두 item이 같은 상태에 있기 때문에, 입력이 `=`일 때 shift와 reduce가 동시에 가능해져 충돌이 발생한다.

하지만 LR(1)에서는 이 상태를 다음과 같이 나눈다:

- `S → L • = R` → lookahead와 관계없이 `=`에서 shift
- `R → L • , $` → lookahead가 `$`일 때만 reduce

즉, `R → L •`의 경우 lookahead가 `$`일 때만 reduce가 허용되므로, 입력이 `=`일 때는 reduce가 불가능해지고, 자연스럽게 shift만 남게 되어 충돌이 사라진다.

또한 그림에서처럼:

action[2, =] = s6  
action[2, $] = r5

와 같이 **입력 기호에 따라 동작이 명확히 분리된다.**

---
# Building LR(1) Automata
LR(1) item `[A → α • β, a]`는 현재까지 α를 읽은 상태이며, 앞으로 β를 더 읽어야 하고, 그 이후에는 입력 기호 a가 올 것으로 기대된다는 의미를 가진다. 즉, 단순히 현재 어디까지 파싱했는지를 나타내는 것뿐만 아니라, **앞으로 어떤 입력이 이어질지에 대한 정보까지 함께 포함하는 상태 표현**이다.

이때 lookahead 기호 a는 항상 직접적으로 사용되는 것은 아니다. β가 아직 남아 있는 경우, 즉 `[A → α • β, a]` 형태에서는 아직 파싱이 진행 중이므로 lookahead는 단순히 정보를 유지하는 역할만 한다. 실제로 중요한 역할을 하는 경우는 β가 없는 상태, 즉 `[A → α • , a]`처럼 점이 오른쪽 끝에 도달했을 때이다.

이 경우에는 α까지 모두 읽은 상태이므로 reduce를 수행할 수 있는데, 이때 lookahead a를 확인하여 **다음 입력이 a일 때만 A → α로 reduce를 수행하도록 제한**한다. 이를 통해 불필요한 reduce를 방지하고, 보다 정확한 parsing을 가능하게 한다.

---
# The Closure Operation for LR(1) Items
LR(1) closure 연산은 단순히 nonterminal을 확장하는 것이 아니라, **lookahead까지 함께 계산하여 확장하는 과정**이다.

기본적으로 어떤 item이 다음과 같은 형태일 때:

```
[A → α • B β, a]
```

이는 현재 α까지 읽었고, 앞으로 B를 처리한 뒤 β가 나오고, 그 이후에는 a가 올 것으로 예상된다는 의미이다.

이때 점 뒤에 nonterminal B가 있으므로, B의 모든 production을 확장해야 한다. 
즉, B → γ 형태의 모든 규칙에 대해 새로운 item을 추가하게 된다.

하지만 LR(1)에서는 단순히 `[B → • γ]`를 추가하는 것이 아니라, **lookahead를 함께 계산해서 붙여야 한다.** 이 lookahead는 다음과 같이 계산된다:

```
b ∈ FIRST(βa)
```

즉, B 뒤에 오는 β와 기존 lookahead a를 이어붙인 문자열 βa에서, **가장 앞에 올 수 있는 terminal들을 구한 것**이 새로운 lookahead가 된다.

따라서 closure에 추가되는 item은 다음과 같은 형태가 된다:

```
[B → • γ, b]
```

이 과정을 더 이상 새로운 item이 추가되지 않을 때까지 반복한다.

---
# The Goto Operation for LR(1) Items
LR(1)에서 `goto(I, X)`는 상태 III에서 기호 XXX를 읽었을 때 이동하는 새로운 상태를 만드는 연산이다.

어떤 상태 $I$에 다음과 같은 item이 있다고 하자:

```
[A → α • X β, a]
```

이는 현재 α까지 읽었고, 다음에 X를 읽을 수 있는 상태라는 의미이다.

이때 $X$를 실제로 하나 읽으면, 점이 오른쪽으로 이동하게 된다:

```
[A → α X • β, a]
```
이렇게 점을 이동시킨 item들을 모은 뒤, 그 결과에 대해 다시 closure를 적용한 것이 바로 `goto(I, X)`가 된다.

즉, 전체 과정은 다음과 같다:

```
goto(I, X) = closure({ [A → α X • β, a] })
```

---
# Constructing the set of LR(1) Items of a Grammar
LR(1) 파서는 결국 **DFA(상태 그래프)** 를 만드는 과정이고, 이 DFA의 각 상태가 바로 **LR(1) item 집합**이다.

#### Step 1. 문법 확장 (Augment)

```
S' → S
```

- 시작 상태를 명확하게 만들기 위한 것  
- accept 조건 만들려고 필요

#### Step 2. 시작 상태 만들기

```
I₀ = closure({ [S' → • S, $] })
```

- 아직 아무것도 안 읽음
- 마지막 입력은 `$` (끝)
- 이게 DFA 시작 상태

#### Step 3. 모든 이동 경우 만들기

각 상태 $I$에 대해 가능한 모든 기호 $X$ (terminal + nonterminal)에 대해 계산한다.

```
goto(I, X)
```

이때 goto(I,X) 이게 값이 있다면 새로운 상태로 추가한다.

#### Step 4. 반복
새로운 상태가 계속 생기니까 더 이상 상태가 안 생길 때까지 반복한다.

---
# Example Grammar and LR(1) Items

![](../images/Pasted%20image%2020260417151654.png)
![](../images/Pasted%20image%2020260417151710.png)

---
# LALR(1) Grammars
LR(1) 의 경우에는 production의 진행위치, lookahead로 다음에 올거 까지 같이 들고 있다. 그래서 겉보기에는 같은 상황이어도 loockahdead가 다르면 서로 다른 stae로 분리된다. 그러다 보니 conflict 를 잘 피할 수느 있지만 관리해야 하는 state가 많아지게 된다.

그래서 이를 해결하기 위해서 조금 state를 더 적게 관리할수 없을까?

LALR(1) 의 경우에는 LR(1) state 중에서 비슷한거 끼리 묶는다. 그니까 production rule도 같고 진행 위치 dot 도 같은데, lookahead 가 다른 경우를 하나로 치환하는 것이ㅏㄷ.

예를 들어

- $[L \to * \bullet R, =]$
- $[L \to * \bullet R, \$]$

이 둘은 dot 위치와 production이 같고 lookahead만 다르다.
이런 것들을 한 state로 묶어서

- $[L \to * \bullet R, =/\$]$

처럼 합쳐버린다.  그래서 **state 수가 크게 줄어든다.**

---
# Constructing LALR(1) Parsing Tables

사실 LALR(1) parsing table을 만드는 과정은 비교적 단순하다. 먼저 canonical LR(1) item set을 모두 생성한 뒤, 각 상태에서 **production과 dot 위치가 동일한 item들(즉, 같은 core를 가지는 상태들)**을 하나로 묶는다. 그리고 이때 서로 다른 상태에 존재하던 **lookahead 집합은 합집합으로 병합**한다.

![](../images/Pasted%20image%2020260418152608.png)

----
# Example LALR(1) Grammar

다음과 같은 produciton rule을 가지는 예시가 있다.

- S → L = R | R
- L → * R | id
- R → L


![](../images/Pasted%20image%2020260418152922.png)

이 예시에서 네모 칸을 보면 원래는 저 상태에서 2개의 item 이 있었는데, 이걸 하나로 합친 것을 볼 수 있다.

![](../images/Pasted%20image%2020260418153135.png)


---
# LR SLR LALR Summary

![](../images/Pasted%20image%2020260418153328.png)

LL 파싱 테이블과 LR 파싱 테이블의 차이는 “무엇을 기준으로 다음 행동을 결정하느냐”에 있다. LL은 비단말과 다음 입력 토큰을 기준으로 어떤 production을 사용할지를 바로 결정하는 방식이다. 그래서 테이블의 형태도 “Nonterminal × Terminal → Production”이 되고, 이를 채우기 위해 FIRST와 FOLLOW를 사용한다. 즉, 문법 자체의 성질을 분석해서 테이블을 구성하는 구조다.

반면 LR은 상태 기반이다. 여기서 상태는 LR item들의 집합으로 이루어진 DFA의 한 노드라고 보면 된다. LR 테이블은 두 부분으로 나뉘는데, ACTION과 GOTO다. ACTION은 “현재 상태와 입력 토큰”을 보고 shift할지, reduce할지, accept할지를 결정하고, GOTO는 reduce 이후 비단말로 이동할 다음 상태를 정한다. 그래서 구조는 “State × Terminal → action”, “State × Nonterminal → next state”가 된다. LL이 문법 규칙 선택 문제라면, LR은 상태 전이 기반의 자동자 실행 문제라고 보면 정확하다.

마지막으로 문법의 성질도 테이블로 판단한다. LL(1) 문법이라는 것은 LL 테이블을 만들었을 때 어떤 셀에도 두 개 이상의 production이 들어가지 않는 경우를 말한다. 마찬가지로 SLR, LALR(1), LR(1)도 각각 해당 방식으로 테이블을 만들었을 때 conflict가 없으면 그 문법에 속한다고 한다. 여기서 conflict라는 것은 같은 입력 상황에서 두 가지 이상의 행동을 선택해야 하는 경우를 의미한다.