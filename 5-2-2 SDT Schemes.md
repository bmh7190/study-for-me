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

전체 흐름은 이렇게 된다.

```
D → T { L.in := T.type } L     ↑        ↑          ↑   T 처리   L.in 설정    L 처리
```

예를 들어 입력이 `int a, b, c` 같은 선언이라면, 먼저 `int`를 보고 `T.type = integer`를 만든다. 그다음 `L.in = integer`로 설정한다. 그러면 `L`은 `a`, `b`, `c` 같은 identifier들에게 모두 integer 타입을 붙일 수 있다.

## Actions inside Productions

슬라이드의 `B → X {a} Y`는 action의 실행 시점을 일반화해서 보여주는 예시다.

```
B → X {a} Y
```

여기서 `{a}`는 `X`와 `Y` 사이에 들어 있는 semantic action이다.

이 action은 **왼쪽에 있는 symbol들이 모두 처리된 직후** 실행된다.

즉 `X`가 처리되고 나면 `{a}`를 실행하고, 그다음 `Y`를 처리한다.

이게 중요한 이유는, `{a}`가 보통 `X`에서 계산된 값을 이용해서 `Y`에 필요한 inherited attribute를 채워주는 역할을 하기 때문이다.

예를 들어:

```
D → T { L.in := T.type } L
```

여기서 `T`가 `X`, `{ L.in := T.type }`가 `{a}`, `L`이 `Y`라고 보면 된다.

## Bottom-up parsing에서는 언제 실행되는가

Bottom-up parser는 shift-reduce 방식으로 동작한다.

그래서

```
B → X {a} Y
```

에서 `{a}`는 `X`가 stack 위에 올라온 뒤 실행된다.

즉 `X`가 이미 인식되었고, 그 결과 attribute도 계산된 상태에서 action을 실행한다.

슬라이드 표현으로는:

```
Bottom-up parse:do {a} after X is on the top of the parsing stack
```

라고 되어 있다.

쉽게 말하면, bottom-up에서는 `X`를 reduce해서 stack 위에 올린 다음, 그 직후 action을 수행한다고 보면 된다.

## Top-down parsing에서는 언제 실행되는가

Top-down parser는 왼쪽에서 오른쪽으로 production을 확장해간다.

그래서

```
B → X {a} Y
```

에서는 `X`를 처리한 뒤, `Y`를 처리하기 직전에 `{a}`를 실행한다.

즉 `Y`가 nonterminal이면 `Y`를 expand하기 바로 전에 action을 실행하고, `Y`가 terminal이면 input에서 `Y`를 match하기 바로 전에 action을 실행한다.

이것도 결국 같은 의미다.

**Y를 처리하기 전에 필요한 정보를 미리 준비한다.**

## Marker nonterminal이 왜 나오는가

마지막 부분에 `M → ε`이 나온다.

이건 나중에 bottom-up parsing이나 LR parsing에서 embedded action을 처리하기 위해 사용하는 방법이다.

문제는 production 중간에 action이 들어가 있으면 parser 입장에서는 애매해질 수 있다.

예를 들어:

```
D → T { L.in := T.type } L
```

여기서 `{ L.in := T.type }`는 문법 symbol은 아니지만, parsing 중간에 실행되어야 한다.

그래서 이 action을 하나의 가짜 nonterminal로 바꿀 수 있다.

```
D → T M LM → ε { L.in := T.type }
```

여기서 `M`은 실제 input을 소비하지 않는다. `M → ε`이기 때문이다.

대신 `M`이 reduce되는 순간 semantic action을 실행한다.

즉 marker nonterminal은 **문법 중간에 있는 action을 parsing 과정에서 실행 가능하게 만들기 위한 장치**다.
top down parsing 에서는 쉽게 할 수 있음
bottom up parsing은 조금 까다로움

SDT schemes를 만든 이유는 parse tree를 만드는 과정에서 같이 하기 위함

---
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
