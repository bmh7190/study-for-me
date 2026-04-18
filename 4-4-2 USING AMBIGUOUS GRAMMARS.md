
# Dealing with Ambiguous Grammars

![](../images/Pasted%20image%2020260418153735.png)

지금 문법은 E → E + E | id  라서 본질적으로 모호하다. 같은 문자열 `id+id+id`에 대해 두 가지 해석이 가능하다. 하나는 `(id+id)+id`이고, 다른 하나는 `id+(id+id)`이다.

이 모호함이 그대로 테이블에 드러난다. 표에서 state 4를 보면 `action[4, +]`에 `s3 / r2`가 같이 들어가 있다. 이게 바로 shift/reduce conflict다. 즉, 현재 상태에서 다음 입력이 `+`일 때 두 가지 선택지가 동시에 가능하다는 의미다. 하나는 shift해서 뒤를 더 읽는 것이고, 다른 하나는 이미 본 것을 reduce하는 것이다.

이 선택이 곧 결합 방향(associativity)을 결정한다. shift를 선택하면 오른쪽을 더 읽고 나중에 묶기 때문에 `id+(id+id)`처럼 right associativity가 되고, reduce를 선택하면 먼저 묶고 나가기 때문에 `(id+id)+id`처럼 left associativity가 된다. 슬라이드 오른쪽이 이걸 보여준다.

그래서 중요한 포인트는 이거다. 문법이 모호하면 파싱 테이블에서도 conflict가 생기고, 그 conflict는 “어떻게 해석할지 결정이 안 된 상태”를 의미한다. LR parser는 원래 deterministic해야 하는데, 여기서는 하나의 입력에 대해 두 행동이 가능해지니까 문제가 되는 거다.

---
# Using Associativity and Precedence to Resolve Conflicts

실제 컴파일러에서는 이걸 그냥 두지 않는다. 보통 연산자에 precedence와 associativity를 부여해서 강제로 하나를 선택하게 만든다. 예를 들어 `+`를 left-associative로 정의하면 reduce를 선택하도록 해서 `(id+id)+id`로 고정한다.

#### Precedence
먼저 **precedence(우선순위)** 는 어떤 연산자를 먼저 계산할지 정하는 규칙이다. 예를 들어 `+`와 `*`가 같이 있으면 보통 `*`가 먼저 계산된다. 그래서 `id + id * id`는 `(id + (id * id))`로 해석된다. 이건 `*`의 precedence가 `+`보다 높기 때문이다.

#### Associativity
다음으로 **associativity(결합 방향)** 는 같은 우선순위의 연산자가 여러 개 있을 때, 어느 쪽부터 묶을지를 정하는 규칙이다. 예를 들어 `+`는 보통 left-associative라서 `id + id + id`는 `(id + id) + id`로 묶는다. 반대로 지수 연산자 같은 건 right-associative라서 `a ^ b ^ c`를 `a ^ (b ^ c)`로 묶는다.

이걸 LR 파싱 관점에서 보면 더 명확해진다. 지금 슬라이드처럼 `E → E + E` 같은 모호한 문법에서는 `+`를 만났을 때 shift를 할지 reduce를 할지 결정이 안 된다. 이때 규칙을 이렇게 준다:

- 연산자가 **left-associative**이면 → reduce 선택
- 연산자가 **right-associative**이면 → shift 선택
- 연산자 우선순위가 높으면 → shift (더 읽어서 먼저 계산)
- 우선순위가 낮으면 → reduce (지금까지 먼저 계산)

그래서 `+`를 left-associative로 정의하면 reduce를 선택하게 되고, 결국 `(id+id)+id`로 고정된다. 반대로 right-associative로 정의하면 shift를 선택해서 `id+(id+id)`가 된다.

![](../images/Pasted%20image%2020260418154136.png)

---
# Error Detection in LR Parsing

Canonical LR(1) 파서는 각 item에 정확한 lookahead 정보를 가지고 있기 때문에, reduction을 수행할 때도 매우 엄격하게 판단한다. 따라서 입력에 문법 오류가 있을 경우, 잘못된 상황에서 섣불리 reduction을 진행하지 않고, 가능한 한 빠른 시점에서 오류를 인식한다. 즉, 오류를 발견하기 전에 불필요한 reduction을 수행하는 일이 없다.

반면 SLR이나 LALR 파서는 lookahead 정보가 상대적으로 덜 정밀하다. SLR은 FOLLOW 집합을 기반으로, LALR은 상태 병합 과정에서 합쳐진 lookahead를 기반으로 reduction을 결정하기 때문에, 실제로는 잘못된 입력임에도 불구하고 일시적으로 올바른 것처럼 판단하여 reduction을 먼저 수행할 수 있다. 그 결과 오류를 조금 늦게 발견하는 경우가 발생한다.

하지만 LR 계열 파서의 공통적인 특징은, 잘못된 입력 토큰을 그대로 shift하여 스택에 올리는 일은 없다는 점이다. 결국 LR(1)은 가장 정확하고 빠르게 오류를 탐지하는 방식이고, SLR과 LALR은 효율성을 위해 일부 정확도를 희생하지만 실무에서는 충분히 안정적으로 사용되는 방식이라고 볼 수 있다.

---
# Error Recovery in LR Parsing

#### Panic mode Recovery
먼저 **Panic mode recovery**는 가장 단순하고 안전한 방법이다. 파서가 에러를 만나면, 스택을 계속 pop하면서 “어떤 비단말 A로 다시 시작할 수 있는 상태”를 찾는다. 그런 다음 A를 스택에 넣고, 입력에서는 의미 없는 토큰들을 버리다가 `FOLLOW(A)`에 해당하는 토큰(예: `;`, `}` 같은 구문 경계)을 만날 때까지 건너뛴다. 결국 에러가 발생한 구문 덩어리를 통째로 버리고, 그 다음 정상적인 지점에서 다시 파싱을 이어가는 방식이다. 구현은 쉽고 안정적이지만, 중간 내용이 많이 날아갈 수 있다는 단점이 있다.

##### Pharse-level Recovery
다음으로 **Phrase-level recovery**는 좀 더 정교한 방식이다. 파싱 테이블의 error 위치마다 “어떻게 고칠지”를 미리 정의해두고, 상황에 맞게 입력이나 스택을 조금 수정한다. 예를 들어 연산자가 빠졌다면 하나를 삽입하거나, 불필요한 토큰을 삭제하는 식이다. 즉, 전체를 버리는 대신 “국소적으로 고쳐서 계속 진행”하려는 접근이다. 다만 어떤 수정이 맞는지 판단하는 기준이 필요해서 구현이 복잡해진다.

##### Error Productions
마지막으로 **Error productions**는 문법 자체에 “에러 상황을 허용하는 규칙”을 추가하는 방법이다. 특정한 잘못된 패턴을 하나의 production으로 정의해두고, 그걸 통해 파싱을 이어간다. 이 경우에도 필요하면 스택을 조정하고 입력을 일부 버리면서 진행한다. Yacc 같은 parser generator에서 자주 사용하는 방식이다. 특정 오류를 의도적으로 처리할 수 있다는 장점이 있지만, 문법 설계가 더 복잡해진다.

정리하면, Panic mode는 “버리고 다시 시작”, Phrase-level은 “조금 고쳐서 계속”, Error production은 “문법에 에러 처리까지 포함”이라고 보면 된다.

---
# Example : Pharse-level  Recovery
이 표는 **phrase-level recovery를 실제 LR 테이블에 어떻게 넣는지** 보여주는 예제다. 핵심은 간단하다. 원래는 error가 나야 할 칸에, “어떻게 고칠지”를 미리 정의해둔 것이다.

![](../images/Pasted%20image%2020260418154528.png)


표에서 빨간색 `e1`, `e2`가 바로 그 에러 처리 루틴이다.

먼저 `e1`은 **operand(피연산자)가 없는 경우**다. 예를 들어 `+ +`, `* +` 같은 상황이다. 이 경우 파서는 “여기에는 id 같은 값이 하나 있어야 정상인데 없네”라고 판단한다. 그래서 실제로는 입력에 없는 `id`를 하나 넣은 것처럼 처리한다. 슬라이드에 있는 “push state 2 (add some id)”가 그 의미다. 즉, 스택에 `id`를 넣고 진행을 계속한다.

반대로 `e2`는 **operator(연산자)가 없는 경우**다. 예를 들어 `id id`처럼 값이 연속으로 나오는 상황이다. 원래라면 `id + id`처럼 중간에 연산자가 있어야 한다. 그래서 파서는 “여기에는 + 같은 연산자가 빠졌네”라고 보고, `+`를 하나 넣은 것처럼 처리한다. 그래서 “push state 3 (add +)”가 된다.

결국 이 테이블은 이렇게 동작한다. 원래라면 error로 멈춰야 하는 상황에서, 단순히 멈추는 대신 “사람이 흔히 하는 실수”를 가정해서 입력을 조금 고쳐주고 계속 진행한다. 즉, 전체를 버리는 panic mode와 달리, **문장을 최대한 살리면서 복구하려는 방식**이다.