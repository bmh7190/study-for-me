이전까지 정리한 **SDD**에서 한 단계 더 나아가서, **semantic rule을 실제 parsing 중에 언제 실행할 것인가**를 정리해보자

# SDD와 SDT schemes의 차이
먼저 SDD는 문법 규칙에 sematic rule을 붙인 것이다.

예를 들어 앞에서 봤던 형태는 왼쪽과 같은 형태였다. 
이건 “어떤 attribute 값을 어떻게 계산해야 하는지”를 말해준다.

![](../images/Pasted%20image%2020260507103648.png)

그런데 여기에는 한 가지가 빠져 있다.

>**그 semantic rule을 정확히 언제 실행할 것인가?**

예를 들어 `D → T L`에서 `L.in := T.type`을 실행하려면, `T.type`이 먼저 계산되어 있어야 한다. 그리고 `L`을 처리하기 전에 `L.in`이 이미 준비되어 있어야 한다.
그래서 SDD를 실제 parser가 실행 가능한 형태로 바꾼 것이 **SDT scheme**이다.

```
1. D → T L        L.in := T.type

2. D → T { L.in := T.type } L
```

1번을 2번처럼 바꾸게 되면 의미가 명확해진다.
`T`를 처리한 직후에 `{ L.in := T.type }`를 실행하고, 그다음 `L`을 처리하라는 뜻이다.

## SDT scheme이란?
SDT scheme은 production rule 안에 실행 코드를 직접 넣은 형태다.

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

Top-down은 **Y를 처리하러 내려가기 전에 action을 실행**하는 것이고, 
bottom-up은 **X가 stack 위에서 완성된 뒤 action을 실행**하는 것이다.

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

d오른쪽 바디 즉 자식들 전체 처리 완료 하고, reduce 하면 서 action 실행하고 action을 통해 부모 attribute 계산하는 아주 자연스러운 흐름이 있다. 이걸 postfix translation schme 이라고 한다.

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


---


---
## SDT with Actions inside Productions

앞에서 본 것처럼 action이 production 중간에 들어갈 수도 있다.

```
B → X {a} Y
```

이 경우 `{a}`는 `X`를 처리한 뒤, `Y`를 처리하기 전에 실행되어야 한다.

이게 필요한 이유는 보통 inherited attribute 때문이다.

예를 들어:

```
D → T { L.in = T.type } L
```

여기서 `{ L.in = T.type }`는 `L`을 처리하기 전에 반드시 실행되어야 한다.

그래서 action이 맨 끝에 있는 postfix 형태와 다르게, production 중간에 들어간다.










# Eliminating Left Recursions from SDT

실제로 값이 어떻게 전달되는지 체크해서 루틴을 만들 필요가 있음

---
## SDT's for L attributesd Definitions
Syn thesized 는 여전히 쉬움 하지만 inherited 는 어려움 

![](../images/Pasted%20image%2020260507104411.png)
while loop 는 condition 체크를 하고 만족하면 while 문 진행 그게 아니면 빠져나옴

가장 먼저해야할건 condition check 를 해야함 

S1 수행한 다음에는 while loop 으로 다시 돌아와야함
condition check 후 false라면 S1이 아니라 밖으로 나가야함

C가 false라면 S의 next가 될 것

S1.next 는 inherited attribute 
앞에서 일을 처리하고 전달을 받았기 때문

S.next 도 마찬가지로 inherited attribute 일 것이다.

C.code 는 자기 ㅅ자신이고
C의 true와 false는 부모에서 넘겨준것이기 때문에 inherited attribute임 

synsthesized attribute는 맨 뒤에 그리고 inheried attribute 는 필요한 곳 전에

그래서 c 전에 c.false c. true S1 전에 S1.next 그리고 맨 뒤에 synthesized

---
# Implemeting L-attributed SDD's

투 패스 컴파일러를 쓴다
중간 단계 파일을 dump 그걸 다시 읽어서 백엔드로 보낸다
조금 더 낫게 할 수 있는 방법은 중간중간에 계속 dump 하자
