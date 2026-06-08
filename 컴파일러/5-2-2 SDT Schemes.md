이전까지 정리한 **SDD**에서 한 단계 더 나아가서, **semantic rule을 실제 parsing 중에 언제 실행할 것인가**를 정리해보자

# SDD와 SDT schemes의 차이

>먼저 SDD는 문법 규칙에 sematic rule을 붙인 것이다.

예를 들어 앞에서 봤던 형태는 왼쪽과 같은 형태였다. 
이건 “어떤 attribute 값을 어떻게 계산해야 하는지”를 말해준다.

![](../images/Pasted%20image%2020260507103648.png)

그런데 여기에는 한 가지가 빠져 있다.

>**그 semantic rule을 정확히 언제 실행할 것인가?**

예를 들어 `D → T L`에서 `L.in := T.type`을 실행하려면, `T.type`이 먼저 계산되어 있어야 한다. 그리고 `L`을 처리하기 전에 `L.in`이 이미 준비되어 있어야 한다.

>그래서 SDD를 실제 parser가 실행 가능한 형태로 바꾼 것이 **SDT scheme**이다.

```
1. D → T L        L.in := T.type

2. D → T { L.in := T.type } L
```

1번을 2번처럼 바꾸게 되면 의미가 명확해진다.
`T`를 처리한 직후에 `{ L.in := T.type }`를 실행하고, 그다음 `L`을 처리하라는 뜻이다.

## SDT scheme이란?

>SDT scheme은 production rule 안에 실행 코드를 직접 넣은 형태다.

즉 문법 규칙과 semantic action이 섞여 있다.

```
D → T { L.in := T.type } L
T → int { T.type := 'integer' }
```

여기서 `{ ... }` 안에 들어간 부분이 semantic action이다.
이 action은 parsing 도중에 실제로 실행된다. 그래서 SDT는 단순한 정의가 아니라, **parser가 어떤 순서로 의미 동작을 수행할지까지 포함한 형태**라고 보면 된다.

---
#### 왜 `{ L.in := T.type }`가 T와 L 사이에 들어가는가

이게 제일 중요하다.

```
D → T L
```

에서 `L.in`은 inherited attribute다. 즉 `L`이 자기 처리를 시작하기 전에 외부에서 값을 받아야 한다. 그리고 그 값은 `T.type`에서 온다.

따라서 순서는 반드시 이렇게 되어야 한다.

1. T를 처리한다.
2. T.type을 계산한다.
3. L.in := T.type을 실행한다.4. L을 처리한다.

그래서 production 안에서는 action이 `T`와 `L` 사이에 들어간다.

```
D → T { L.in := T.type } L
```

이 말은 `L`을 처리하기 전에 `L.in`을 미리 채워두겠다는 뜻이다.

만약 action을 맨 뒤에 둔다면?

```
D → T L { L.in := T.type }
```

이렇게 되면 `L`을 이미 처리한 뒤에 `L.in`을 넣는 것이므로 늦다. inherited attribute는 자식 노드가 처리되기 전에 전달되어야 하니까, action 위치가 중요하다.

---
#### `T → int { T.type := 'integer' }`

이 규칙은 간단하다. `int`를 읽으면 `T.type`을 `integer`로 설정한다.

```
T → int { T.type := 'integer' }
```

즉 입력에서 `int`를 확인한 뒤, 이 `T`가 나타내는 타입 정보를 `integer`로 만든다.

그다음 상위 production인 `D → T { L.in := T.type } L`에서 이 값을 `L.in`으로 넘길 수 있다.

예를 들어 입력이 `int a, b, c` 같은 선언이라면, 먼저 `int`를 보고 `T.type = integer`를 만든다. 그다음 `L.in = integer`로 설정한다. 그러면 `L`은 `a`, `b`, `c` 같은 identifier들에게 모두 integer 타입을 붙일 수 있다.

---
### Bottom-up parsing에서는 언제 실행되는가
Bottom-up parser는 shift-reduce 방식으로 동작한다.

```
B → X {a} Y
```

위의 예시로 보면 `{a}`는 `X`가 stack 위에 올라온 뒤 실행된다. 즉 `X`가 이미 인식되었고, 그 결과 attribute도 계산된 상태에서 action을 실행한다.

```
1. 입력을 읽어서 X를 만든다.
2. X가 stack top에 올라온다.
3. 그때 {a}를 실행한다.
4. 이후 Y를 읽고 처리한다.
```
### Top-down parsing에서는 언제 실행되는가

Top-down parser는 왼쪽에서 오른쪽으로 production을 확장해간다.

```
B → X {a} Y
```

위의 예시에서는 `X`를 처리한 뒤, `Y`를 처리하기 직전에 `{a}`를 실행한다.

즉 `Y`가 nonterminal이면 `Y`를 expand하기 바로 전에 action을 실행하고, `Y`가 terminal이면 input에서 `Y`를 match하기 바로 전에 action을 실행한다.

```
1. B를 처리하기 위해 B → X {a} Y를 선택한다.
2. X를 처리한다.
3. X 처리가 끝나면 {a}를 실행한다.
4. 그다음 Y를 처리한다.
```

- Top-down은 **Y를 처리하러 내려가기 전에 action을 실행**하는 것이고, 
- bottom-up은 **X가 stack 위에서 완성된 뒤 action을 실행**하는 것이다.

둘 다 결과적으로는 `X 처리 후, Y 처리 전`이지만, top-down은 호출/확장 관점이고 bottom-up은 shift-reduce stack 관점이라고 보면 된다.

### Marker nonterminal이 왜 나오는가

마지막 부분에 `M → ε`이 나온다. 이건 나중에 bottom-up parsing이나 LR parsing에서 embedded action을 처리하기 위해 사용하는 방법이다.

문제는 production 중간에 action이 들어가 있으면 parser 입장에서는 애매해질 수 있다.

```
D → T { L.in := T.type } L
```

여기서 `{ L.in := T.type }`는 문법 symbol은 아니지만, parsing 중간에 실행되어야 한다. 그래서 이 action을 하나의 가짜 nonterminal로 바꿀 수 있다.

```
D → T M L
M → ε { L.in := T.type }
```

여기서 `M`은 실제 input을 소비하지 않는다. `M → ε`이기 때문이다.
대신 `M`이 reduce되는 순간 semantic action을 실행한다.

즉 marker nonterminal은 **문법 중간에 있는 action을 parsing 과정에서 실행 가능하게 만들기 위한 장치**다.

---
# SDT는 보통 parse tree 없이 구현한다

앞에서는 설명을 쉽게 하려고 parse tree를 그리고, 각 노드에 attribute를 붙인다고 했다.
그런데 실제 compiler가 매번 완전한 parse tree를 만든 다음 다시 순회하면 비효율적이다.

그래서 보통은 parsing 도중에 바로 semantic action을 실행한다.

parsing 하면서 production rule 이 확인되는 순간 sematic action이 실행하고, attribute 계산 / AST 생성 / 코드 생성 같은 일이 같이 일어난다.

그런데 parser의 방식에 따라 실행하기 편한 action 위치가 다르다. 

parser는 크게 방향이 다르다. LR parser은 bottom up 방식이고, LL parser는 top down 방식이다. 그리고 attribute 사이에서도 방향이 있다. synthesized attibute는 자식에서 부모로, inherited 는 부모에서 자식 혹은 자식 들 간에 attribute 이 들어간다.

- LR-parsable grammar and SDD is S-attributed
- LL-parsable grammar and SDD is L-attributed

그래서 parser의 진행 방향과 attribute의 흐름이 잘 맞으면 구현이 편하다.

---
### LR parser + S-attributed SDD

LR parser는 bottom-up 방식이다. 즉 leaf 쪽에서 시작해서 위로 reduce한다.
S-attributed SDD는 모든 attribute가 synthesized attribute다. 즉 자식에서 부모로 값이 올라간다.

둘 다 방향이 잘 맞는다.
그래서 LR parser에서는 S-attributed SDD를 구현하기 좋다.

```
E → E1 + T { E.val = E1.val + T.val }
```

이런 action은 `E1 + T`가 모두 stack에 올라와서 reduce될 때 실행하면 된다.
즉 reduce 시점에 자식들의 attribute가 이미 준비되어 있으니까, 부모 attribute를 계산하기 쉽다. 그래서 LR parser 같은 경우에 action 이 맨 끝에 있는 형태가 자연스럽다.

오른쪽 바디 즉 자식들 전체 처리 완료 하고, reduce 하면 서 action 실행하고 action을 통해 부모 attribute 계산하는 아주 자연스러운 흐름이 있다. 이걸 postfix translation schme 이라고 한다.

---
## Postfix Translation Scheme

Postfix translation scheme은 action을 production의 맨 오른쪽에 두는 방식이다.

```
E → E1 + T { print('+') }
```

여기서 `{ print('+') }`가 production body의 맨 끝에 있다.

이 방식은 bottom-up parsing과 잘 맞는다.

왜냐하면 bottom-up parser는 오른쪽 부분이 다 인식된 뒤 reduce하기 때문이다.

```
E1 + T를 모두 인식함
→ reduce E → E1 + T
→ 이때 action 실행
```

즉 postfix SDT는 **자식들이 모두 준비된 뒤 부모를 만들 때 action을 실행하는 구조**라고 보면 된다.

---
# Parser-Stack Implementation은 뭔가

LR parser는 stack을 쓴다. 그런데 stack에 grammar symbol만 있으면 semantic action을 실행할 수 없다.

예를 들어 stack에 이렇게만 있으면 `E.val`, `T.val`을 알 수 없다.

```
E + T
```

그래서 실제로는 symbol과 attribute를 같이 저장한다.

```
E, E.val = 9
+
T, T.val = 5
```

그러면 reduce할 때

```
E → E + T
```

를 적용하면서 stack에서 값을 꺼내 계산한다.

```
E.val = 9 + 5 = 14
```

그리고 새로 만들어진 `E`를 stack에 넣는다.

```
E, E.val = 14
```

이게 parser-stack implementation이다.

즉 의미는 간단하다. **LR parser의 stack에 문법 기호뿐 아니라 attribute 값도 같이 저장해두고, reduce할 때 semantic action을 실행한다.**

---
## LL parser + L-attributed SDD

LL parser는 top-down 방식이다. 즉 start symbol에서 시작해서 왼쪽에서 오른쪽으로 production을 펼친다.

L-attributed SDD는 inherited attribute가 있더라도, 정보 흐름이 왼쪽에서 오른쪽으로만 가도록 제한된 형태다.

그래서 LL parser와 잘 맞는다.

```
D → T { L.in = T.type } L
```

여기서 `T`를 먼저 처리하고, 그 결과인 `T.type`을 `L.in`으로 넘긴 다음 `L`을 처리한다.

이건 top-down parser에서 자연스럽게 구현할 수 있어.

```
D() {
    Ttype = T();
    Lin = Ttype;
    L(Lin);
}
```

즉 inherited attribute는 함수 인자처럼 넘기고, synthesized attribute는 return 값처럼 받을 수 있다.

---
## SDT with Actions inside Productions

지금까지 postfix는 action이 맨 끝에 있는 경우였다.

```
E → E1 + T { action }
```

그런데 inherited attribute는 action이 맨 끝에 있으면 안 되는 경우가 많다.

예를 들어 여기서 `L.in`은 `L`을 처리하기 전에 필요하다.

```
D → T L
L.in = T.type
```

그런데 action을 맨 끝에 두면 늦어지게 된다.

```
D → T L { L.in = T.type }
```

왜냐하면 이미 `L`을 처리한 뒤에 `L.in`을 넣는 것이기 때문이다.

그래서 action을 중간에 둔다.

```
D → T { L.in = T.type } L
```

이 말은 T 처리 후에 L 처리 전 L.in을 세팅한다 라는 뜻이다.

이걸 **SDT with actions inside productions**라고 한다. 즉 action이 production 맨 끝이 아니라, 필요한 위치에 들어간다.

>**LR parser에서는 reduce 시점에 자식들이 이미 준비되어 있어 action을 맨 뒤에서 실행하기 쉽지만, LL parser에서는 오른쪽 symbol을 처리하기 전에 inherited attribute를 넘겨야 하므로 action이 production 중간에 들어갈 수 있다.**

----
예시로 위의 방식들을 다뤄보자

![](../images/Pasted%20image%2020260601154836.png)

### Postfix Translation Schemes

왼쪽 위는 action이 production의 **맨 뒤**에 붙어 있다.

![](../images/Pasted%20image%2020260601155054.png)

이 방식은 **S-attributed SDD**에 잘 맞는다. 모든 값이 자식에서 부모로 올라가기 때문이다.

예를 들어 `E → E1 + T`를 보면, `E.val`을 계산하려면 `E1.val`과 `T.val`이 필요하다. 이 둘은 모두 오른쪽 body에 있는 자식들의 attribute다. 그래서 `E1 + T`가 전부 처리된 뒤에 action을 실행하면 된다.

이걸 postfix translation scheme이라고 부르는 이유는, semantic action이 production body의 뒤쪽에 오기 때문이다.

---
### SDT actions inside productions

오른쪽 위는 action이 production의 **중간 또는 앞쪽**에 들어가 있다.

![](../images/Pasted%20image%2020260601155136.png)

여기서는 계산값을 저장하는 것이 아니라, parsing 중간에 특정 시점에 바로 출력하는 예시다.

그런데 이 예시는 조금 주의해서 봐야 한다. 

postfix scheme은 `E.val` 같은 attribute를 계산하는 방식이고, 오른쪽은 `print()` action을 production 중간에 배치한 방식이다.

즉 목적이 약간 다르다.

- 왼쪽은 값을 계산해서 attribute에 저장
- 오른쪽은 parsing 중 특정 시점에 바로 출력

예를 들어 `E → { print('+'); } E1 + T`는 `E` production을 선택하는 순간 `+`를 먼저 출력한다.

이런 식으로 action을 production 중간이나 앞쪽에 넣으면, 단순히 reduce 시점에 실행하는 것보다 더 세밀하게 실행 시점을 조절할 수 있다.

다만 LR parser에서는 이런 중간 action을 그대로 처리하기 어렵기 때문에, 나중에 marker nonterminal 같은 방식으로 바꿔서 구현할 수 있다.

---
### Parser-Stack Implementation

LR parser에서 semantic attribute를 stack에 어떻게 저장하는지를 보여준다.

![](../images/Pasted%20image%2020260601155307.png)

LR parser는 stack을 사용한다. 그런데 단순히 grammar symbol만 저장하면 semantic action을 실행할 수 없다.

예를 들어 `E → E1 + T`에서 action은 다음과 같다.

```text
E.val = E1.val + T.val
```

이걸 계산하려면 stack 안에 `E1.val`과 `T.val`이 같이 있어야 한다.
그래서 symbol과 attribute를 한 쌍으로 저장한다.

---
#### `E → E1 + T`

슬라이드 아래쪽에 이런 action이 있다.

```text
E → E1 + T {
  stack[top - 2].val = stack[top - 2].val + stack[top].val;
  top = top - 2;
}
```

이걸 이해하려면 stack 상태를 이렇게 보면 된다.

```text
stack[top]     : T.val
stack[top - 1] : '+'
stack[top - 2] : E1.val
```

`E → E1 + T`로 reduce할 때, 실제로 필요한 값은 `E1.val`과 `T.val`이다.

그래서 `stack[top - 2].val = stack[top - 2].val + stack[top].val`
이 말은 `E.val = E1.val + T.val` 과 같은 의미다.

그리고 reduce 후에는 `E1 + T` 세 칸이 하나의 `E`로 줄어든다.
그래서 stack top 위치를 줄인다.

```text
top = top - 2
```

세 개가 하나로 줄어드니까 전체적으로 stack 크기가 2만큼 감소하는 것이다.

---
#### `T → T1 * F`도 똑같다

```text
T → T1 * F {
  stack[top - 2].val = stack[top - 2].val * stack[top].val;
  top = top - 2;
}
```

stack 상태는 대략 이렇다.

```text
stack[top]     : F.val
stack[top - 1] : '*'
stack[top - 2] : T1.val
```

그래서 `T.val = T1.val * F.val` 를 계산하고, 세 칸을 하나의 `T`로 줄인다.

---
#### `F → ( E )`는 왜 `top - 1`인가

```text
F → ( E ) {
  stack[top - 2].val = stack[top - 1].val;
  top = top - 2;
}
```

이 경우 stack에는 이렇게 있다.

```text
stack[top]     : ')'
stack[top - 1] : E.val
stack[top - 2] : '('
```

괄호 자체는 값이 없다. 실제 값은 가운데 있는 `E.val`이다.

그래서 `F.val = E.val` 이 되어야 한다.

이를 stack 기준으로 쓰면 `stack[top - 2].val = stack[top - 1].val` 이 된다.
그리고 `( E )` 세 개가 `F` 하나로 reduce되므로 역시 `top = top - 2`가 된다.

---
# Eliminating Left Recursions from SDT

왜 left recursion을 제거해야 하는지 회상해보자. 예를 들어 이런 문법이 있다고 하자.

```
E → E + T
E → T
```

이 문법은 left recursive하다. 왜냐하면 `E → E + T`처럼 오른쪽이 다시 `E`로 시작하기 때문이다.

Top-down parser, 특히 recursive descent parser에서는 이런 문법을 그대로 쓰면 문제가 생긴다.

```
E()
→ E()
→ E()
→ E()
...
```

계속 자기 자신을 먼저 호출해서 무한 재귀에 빠질 수 있다.

그래서 left recursion을 제거해서 보통 이런 형태로 바꾼다.

```
E → T R
R → + T R
R → ε
```

이제 `E`는 먼저 `T`를 읽고, 뒤에 `+ T`가 반복되는 구조가 된다.

---
## Simple case

원래 SDT가 이렇게 있었다고 하자. 이건 postfix 출력 예제다.

```text
E → E + T { print('+'); }
E → T
```

예를 들어 `a + b + c`를 처리하면, 연산자 `+`를 피연산자 뒤에 출력해서 postfix 형태를 만들 수 있다.

원래 문법은 left recursive하니까, 문법을 이렇게 바꾼다.

```text
E → T R
R → + T { print('+'); } R
R → ε
```

여기서 중요한 건 `{ print('+'); }`의 위치다. 단순히 문법만 바꾸면 안 되고, 원래 `+`가 출력되던 시점과 같은 의미가 되도록 action을 배치해야 한다.

```
R → + T { print('+'); } R
```

이렇게 둔 이유는 `+ T`를 처리한 뒤에 `+`를 출력해야 postfix 순서가 맞기 때문이다.

예를 들어 입력이 `a + b + c` 이면 postfix는 `a b + c +` 가 되어야 한다.

그래서 첫 번째 `+ T`에서 `b`까지 처리한 다음 `+`를 출력하고, 두 번째 `+ T`에서 `c`까지 처리한 다음 `+`를 출력한다.

흐름은 이렇게 된다.

```text
a 처리
b 처리
print('+')
c 처리
print('+')
```

즉 action을 terminal처럼 취급해서, 문법 변환 과정에서 같이 옮겨야 한다는 의미다.

---
### action을 terminal symbol처럼 취급해야 한다.

문법을 변환할 때 `{ print('+'); }` 같은 action도 하나의 symbol처럼 보고 위치를 유지하라는 뜻이다.

예를 들어 원래 오른쪽 body가 `E + T { print('+'); }` 였다면, `{ print('+'); }`도 그냥 body 안의 한 요소처럼 본다.

```text
E + T action
```

left recursion 제거를 할 때도 이 action이 원래 어떤 symbol 뒤에서 실행되어야 했는지 유지해야 한다. 이 경우 action은 `T` 뒤에 있어야 한다. 그래야 오른쪽 피연산자 `T`를 처리한 다음 `+`를 출력할 수 있다.

그래서 변환 후에도 `R → + T { print('+'); } R` 이 된다.

---
# complex case

두 번째 경우는 단순히 `print()`만 하는 게 아니라, attribute 값을 계산하는 경우다.

![](../images/Pasted%20image%2020260601160735.png)

이것도 left recursive하다. `A → A1 Y`에서 오른쪽이 다시 `A1`으로 시작하기 때문이다.


![](../images/Pasted%20image%2020260601160959.png)

처음에는 `X`에서 시작해서 값을 만든다.

```text
A.a = f(X.x)
```

그다음 `Y`가 하나씩 붙을 때마다 기존 값을 누적해서 갱신한다.

```text
A.a = g(이전 A.a, Y.y)
```

예를 들어 `X Y1 Y2`가 있다면 원래 의미는

```text
A.a = g(g(f(X.x), Y1.y), Y2.y)
```

가 되어야 한다. 즉 왼쪽에서 오른쪽으로 누적 계산하는 구조다.

---
### left recusion 제거하면?

left recursion을 제거하면 문법 자체는 보통 이렇게 바뀐다.

```text
A → X R
R → Y R
R → ε
```

그런데 여기서 문제는 attribute 계산이다.

원래는 `A.a`가 계속 누적되어야 한다.

```text
f(X.x)
→ g(f(X.x), Y1.y)
→ g(g(f(X.x), Y1.y), Y2.y)
```

이 누적값을 `R`에게 넘기면서 계산해야 한다.

그래서 `R`에 inherited attribute와 synthesized attribute를 둔다.

```text
R.i : 지금까지 누적된 값
R.s : 최종 결과값
```

---

# 변환된 SDT 해석

슬라이드의 변환 결과는 다음과 같다.

```text
A → X { R.i := f(X.x) } R { A.a := R.s }

R → Y { R1.i := g(R.i, Y.y) } R1 { R.s := R1.s }

R → ε { R.s := R.i }
```

#### `A → X { R.i := f(X.x) } R { A.a := R.s }`

먼저 `X`를 처리한다. 그러면 `X.x`가 생긴다.

원래 base case였던 `A → X { A.a := f(X.x) }` 의 의미를 살려서, 첫 누적값을 만든다.

```text
R.i := f(X.x)
```

그런 다음 `R`이 뒤에 오는 `Y`들을 처리한다. `R` 처리가 끝나면 최종 결과가 `R.s`에 들어 있다.

그래서 마지막에 `A.a := R.s` 로 전체 결과를 `A.a`에 넣는다.

---

#### `R → Y { R1.i := g(R.i, Y.y) } R1 { R.s := R1.s }`

이 규칙은 `Y`가 하나 더 붙은 경우다.

- 현재까지의 누적값은 `R.i`에 들어 있다.
- 새로 처리한 `Y`의 값은 `Y.y`다.

그러면 새로운 누적값은 `g(R.i, Y.y)` 가 된다.

이 값을 다음 `R1`에게 넘긴다.

```text
R1.i := g(R.i, Y.y)
```

즉 `R1`은 “여기까지 누적된 값”을 이어받아서 나머지 `Y`들을 계속 처리한다.

그리고 `R1`이 최종 결과를 계산해서 `R1.s`로 돌려주면, 현재 `R`도 그 값을 그대로 자신의 최종 결과로 삼는다.

```text
R.s := R1.s
```

---

#### `R → ε { R.s := R.i }`

이건 더 이상 `Y`가 없는 경우다. 즉 누적 계산이 끝났다.

그러면 현재까지 들고 있던 누적값 `R.i`가 최종 결과가 된다.

```text
R.s := R.i
```

그래서 `R.s`로 올려보낸다.

---

입력이 `X Y1 Y2`라고 해보자.

원래 left recursive 문법에서는 결과가 이렇게 되어야 한다.

```text
A.a = g(g(f(X.x), Y1.y), Y2.y)
```

변환된 문법에서는 이렇게 진행된다.

![](../images/Pasted%20image%2020260601162526.png)


```text
A → X R
R.i = f(X.x)
```

첫 번째 `Y1` 처리

```text
R1.i = g(R.i, Y1.y)
     = g(f(X.x), Y1.y)
```

두 번째 `Y2` 처리

```text
R2.i = g(R1.i, Y2.y)
     = g(g(f(X.x), Y1.y), Y2.y)
```

마지막 `R → ε`

```text
R2.s = R2.i
```



![](../images/Pasted%20image%2020260601162546.png)

```text
R1.s = R2.s
R.s = R1.s
A.a = R.s
```

다시 위로 올라오면서 결국 `A.a = g(g(f(X.x), Y1.y), Y2.y)` 가 된다.
원래 left recursive SDT와 같은 결과가 나온다.

>left recursion을 제거한 뒤에는 원래 `A.a`에 누적되던 값을 `R.i`로 전달하면서 아래로 내려간다. 각 `Y`를 처리할 때마다 `R.i`와 `Y.y`를 이용해 새로운 누적값을 만들고, 이를 다음 `R1.i`로 넘긴다. 마지막에 `R → ε`에 도달하면 더 이상 처리할 `Y`가 없으므로 현재 누적값 `R.i`를 `R.s`로 확정한다. 이후 `R.s`가 위로 전달되어 최종적으로 `A.a`에 저장된다.

---
# SDT's for L attributesd Definitions

앞에서 우리는 SDD와 SDT를 구분했다.

SDD는 production마다 semantic rule을 붙여서 **무엇을 계산할지**를 정의하는 방식이었다. 그런데 실제 parser가 동작할 때는 단순히 “계산해야 한다”만으로는 부족하다. 중요한 건 **언제 계산해야 하는가**이다.

특히 L-attributed SDD에서는 inherited attribute가 자주 등장한다. inherited attribute는 어떤 nonterminal이 처리되기 전에 미리 값이 들어가 있어야 한다. 그래서 semantic action을 아무 위치에나 둘 수 없다.

L-attributed SDD를 SDT로 바꿀 때 핵심 규칙은 두 가지다.

- **inherited attribute를 계산하는 action은 해당 nonterminal 바로 앞에 둔다**는 것이다.

- **synthesized attribute를 계산하는 action은 production 맨 끝에 둔다**는 것이다.

이 두 규칙은 attribute가 필요한 시점이 다르기 때문에 생긴다.

---
### inherited attribute action은 왜 앞에 두는가

Inherited attribute는 부모나 왼쪽 형제에게서 정보를 받아오는 attribute다.

예를 들어 다음 production을 보자.

```
D → T L
L.in := T.type
```

여기서 `L.in`은 `L`의 inherited attribute다.

`L`은 자기 내부를 처리할 때 `L.in` 값을 사용해야 한다. 그러면 `L`을 처리하기 전에 이미 `L.in`이 준비되어 있어야 한다.

그래서 SDT로 바꾸면 action을 `L` 앞에 둔다.

```
D → T { L.in := T.type } L
```


만약 action을 맨 뒤에 둔다면

```
D → T L { L.in := T.type }
```

이건 잘못된 위치다. 왜냐하면 이미 `L`을 처리한 뒤에 `L.in`을 넣는 것이기 때문이다. `L.in`은 `L`을 처리하는 도중에 필요한 값이므로, 처리 후에 넣으면 늦다.

그래서 inherited attribute를 계산하는 action은 **그 값을 받을 nonterminal 바로 앞**에 둔다.

---
## synthesized attribute action은 왜 맨 끝에 두는가

Synthesized attribute는 자식들의 값을 이용해서 부모의 값을 계산하는 attribute다.

예를 들어

```
E → E1 + T
E.val := E1.val + T.val
```

여기서 `E.val`은 부모 `E`의 synthesized attribute다.

이 값을 계산하려면 `E1.val`과 `T.val`이 먼저 필요하다. 즉 오른쪽 body에 있는 `E1`과 `T`가 모두 처리된 뒤에야 계산할 수 있다.

그래서 SDT에서는 action을 production 맨 끝에 둔다.

```
E → E1 + T { E.val := E1.val + T.val }
```

즉 synthesized attribute는 자식들이 다 처리된 후에 부모 값을 만들기 때문에, action이 맨 끝에 오는 것이 자연스럽다.

---
### L-attributed SDD와 연결해서 이해하기

L-attributed SDD는 정보 흐름이 왼쪽에서 오른쪽으로 가는 형태다.

예를 들어 production이 다음과 같다고 하자.

```
A → X1 X2 X3
```

여기서 `X3.in`을 계산해야 한다면, `X3`보다 왼쪽에 있는 `X1`, `X2`의 attribute나 부모 `A`의 attribute를 사용할 수 있다.

그러면 SDT에서는 `X3` 바로 앞에 action을 넣는다.

```
A → X1 X2 { X3.in := ... } X3
```

이렇게 해야 `X3`를 처리하기 전에 필요한 inherited attribute가 준비된다.

반대로 head인 `A`의 synthesized attribute를 계산한다면, 오른쪽 symbol들이 모두 처리된 뒤에 계산해야 한다.

```
A → X1 X2 X3 { A.s := ... }
```

그래서 synthesized attribute action은 production 맨 끝에 둔다.

---
# EX 5.19 S→while (C) S1

이번 예제는 위의 두 규칙을 `while` 문에 적용한 것이다.

여기서 `C`는 조건식이고, `S1`은 while 문 안에서 반복 실행될 statement다.

컴파일러는 이 while 문을 단순히 문법적으로 인식하는 것에서 끝나지 않고, 실제 실행 흐름을 표현하는 code를 만들어야 한다.

## Syntax-Directed Definition 의미

![](../images/Pasted%20image%2020260507104411.png)
#### `L1 = new();`

- `L1`은 while 문의 시작 지점 label이다. 
- while 문은 반복문이기 때문에, body 실행이 끝나면 다시 조건 검사 위치로 돌아가야 한다.
- 그래서 조건 검사를 시작하는 위치에 label이 필요하다.

```
L1:    조건 검사
```

즉 `L1`은 while 문의 맨 앞, 조건 검사 위치다.

---
#### `L2 = new();`

- `L2`는 조건이 true일 때 이동할 위치다.
- 조건 `C`가 참이면 while body인 `S1`을 실행해야 한다.
- 그래서 body 시작 위치에도 label이 필요하다.

```
L2:    S1.code
```

---
#### `S1.next = L1`

- `S1.next`는 `S1` 실행이 끝난 다음 어디로 가야 하는지를 의미한다.
- while 문에서는 body 실행이 끝나면 다시 조건 검사로 돌아가야 한다.
- 그래서 `S1.next`는 `L1`이 된다.

이 값은 `S1`이 code를 만들 때 필요하므로, `S1`을 처리하기 전에 미리 전달되어야 한다.
그래서 `S1.next`는 inherited attribute다.

---
#### `C.false = S.next`

- `C.false`는 조건 `C`가 false일 때 어디로 갈지를 의미한다.
- while 조건이 false면 반복문을 빠져나가야 한다.
- 그런데 while 문을 빠져나간 다음 위치는 `S.next`다.

즉 `S.next`는 현재 statement `S`가 끝난 뒤 다음에 실행될 위치다.

그래서 조건이 false이면 `C.false = S.next` 가 된다.

이 값은 조건식 `C`의 code를 만들 때 필요하다. 조건식 code는 true/false에 따라 jump를 생성해야 하기 때문이다. 따라서 `C.false`는 `C`를 처리하기 전에 미리 정해져 있어야 한다.

그래서 `C.false`도 inherited attribute다.

---
#### `C.true = L2`

- `C.true`는 조건 `C`가 true일 때 어디로 갈지를 의미한다.
- while 조건이 true이면 body를 실행해야 한다.

body 시작 label이 `L2`이므로 `C.true = L2` 가 된다.

이 값도 조건식 `C`의 code 생성에 필요하므로, `C`를 처리하기 전에 미리 전달되어야 한다.
따라서 `C.true`도 inherited attribute다.

---
#### `S.code = label || L1 || C.code || label || L2 || S1.code`

이건 while 문 전체 code를 조립하는 부분이다.

구조는 다음과 같다.

```
label L1
C.code
label L2
S1.code
```

조금 더 의미 중심으로 쓰면:

```
L1:
    C.code

L2:
    S1.code
```

여기서 `C.code` 안에는 조건이 true이면 `C.true`, false이면 `C.false`로 jump하는 코드가 들어간다고 보면 된다.

이미 앞에서 
```
C.true = L2
C.false = S.next
```

로 정했기 때문에, `C.code`는 조건 결과에 따라 적절한 위치로 이동할 수 있다.

그리고 `S1.next = L1`로 정했기 때문에, `S1.code`는 body가 끝난 뒤 다시 `L1`로 돌아가는 흐름을 만들 수 있다.

`S.code`는 `C.code`와 `S1.code`가 모두 만들어진 뒤에 조립할 수 있다.

그래서 `S.code`는 synthesized attribute다.

---
## SDD를 SDT로 바꾸는 과정

이제 중요한 부분은 아래쪽 SDT다.

![[Pasted image 20260601165651.png]]

원래 production은 `S → while ( C ) S1` 이었다.

그런데 inherited attribute들은 해당 nonterminal이 처리되기 전에 준비되어야 한다.

그래서 action을 중간에 배치한다.

---
#### 첫 번째 action

첫 번째 action은 `C` 바로 앞에 있다.

```
{ L1 = new(); L2 = new(); C.false = S.next; C.true = L2; }
```

왜 `C` 앞에 있냐면, 이 action이 `C`의 inherited attribute를 계산하기 때문이다.

`C`를 처리하기 전에 다음 값들이 필요하다.

```
C.false = S.nextC.true = L2
```

그래서 `C` 앞에 action을 둔다.

또 `C.true = L2`를 하려면 `L2`가 먼저 있어야 하고, body가 끝난 뒤 돌아갈 `L1`도 필요하므로 여기서 `L1`, `L2`를 함께 만든다.

여기서 `dummy`라고 적힌 이유는 `L1`, `L2`가 grammar symbol의 attribute라기보다는, code 생성을 위해 중간에 만들어두는 임시 label이기 때문이다.

즉 parsing symbol은 아니지만, 뒤의 semantic action들이 사용해야 하는 보조 값이다.

---
#### 두 번째 action

```
) { S1.next = L1; } S1
```

이 action은 `S1` 바로 앞에 있다.

왜냐하면 `S1.next`는 `S1`의 inherited attribute이기 때문이다.

`S1`이 자기 code를 만들 때, statement가 끝난 뒤 어디로 가야 하는지 알아야 한다.

while body가 끝나면 다시 조건 검사 위치로 가야 하므로 `S1.next = L1` 이다.

이 값은 `S1` 처리 전에 필요하기 때문에 `S1` 앞에 둔다.

---
#### 세 번째 action

```
S1 { S.code = label || L1 || C.code || label || L2 || S1.code; }
```

이 action은 production 맨 끝에 있다.

왜냐하면 `S.code`는 synthesized attribute이기 때문이다.

`S.code`를 만들려면 다음 값들이 모두 준비되어 있어야 한다.

```
L1L2C.codeS1.code
```

특히 `C.code`는 `C`를 처리한 뒤에 나오고, `S1.code`는 `S1`을 처리한 뒤에 나온다.

그래서 `S1`까지 모두 처리한 다음, 맨 마지막에 while 문 전체 code인 `S.code`를 조립한다.

---

# 이 예제의 흐름 정리

전체 실행 순서를 보면 이렇게 된다.

```
1. while ( 를 만난다.

2. L1, L2를 새로 만든다.
   L1 = while 조건 검사 위치
   L2 = while body 시작 위치

3. C를 처리하기 전에 C의 inherited attribute를 정한다.
   C.true = L2
   C.false = S.next

4. C를 처리한다.
   C.code 생성

5. S1을 처리하기 전에 S1의 inherited attribute를 정한다.
   S1.next = L1

6. S1을 처리한다.
   S1.code 생성

7. 마지막에 전체 while 문 code를 조립한다.
   S.code = label L1 || C.code || label L2 || S1.code
```

---

# 왜 이게 L-attributed SDT인가

이 예제에서 inherited attribute는 모두 해당 nonterminal 앞에서 계산된다.

```
C.true, C.false
→ C 앞에서 계산

S1.next
→ S1 앞에서 계산
```

그리고 synthesized attribute인 `S.code`는 production 맨 끝에서 계산된다.

```
S.code  
→ production 끝에서 계산
```

즉 앞 슬라이드의 규칙을 그대로 따른다.

**L-attributed SDD를 SDT로 바꾸면 inherited attribute 계산은 필요한 nonterminal 앞에 넣고, synthesized attribute 계산은 production 끝에 넣어서 parsing 중간중간 필요한 순서대로 semantic action을 실행한다.**

그래서 이 예제는 L-attributed SDD를 SDT로 바꾸는 대표적인 예시다.

