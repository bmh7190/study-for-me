앞에서는 L-attributed SDD를 SDT로 바꾸는 규칙을 배웠다.

예를 들어 inherited attribute는 이렇게 필요한 nonterminal 앞에 넣었다.

```
D → T { L.in := T.type } L
```

그리고 synthesized attribute는 production 맨 끝에 넣었다.

```
E → E1 + T { E.val := E1.val + T.val }
```

여기까지는 “semantic action을 어디에 배치해야 하는가”에 대한 내용이었다.

그런데 실제 컴파일러는 이 action들을 진짜로 실행해야 한다. 그래서 이제 질문이 바뀐다.

>이 action들을 실제 parser 코드에서는 어떻게 처리할 것인가?

이걸 설명하기 위해 나온 단원이 **Implementing L-attributed SDD’s**다.

특히 Recursive-Descent Parser에서는 nonterminal마다 함수가 하나씩 생긴다. 그래서 L-attributed SDD의 attribute 전달을 함수의 **argument**와 **return value**로 구현할 수 있다는 것을 설명한다.

---
# During Recursive -Descent Parsing

Recursive-Descent Parser는 각 nonterminal을 함수로 만든다.

예를 들어 nonterminal `A`가 있으면 parser 안에는 `A()`라는 함수가 있다고 보면 된다.

이제 이 `A()` 함수가 단순히 문법만 검사하는 것이 아니라, attribute도 같이 처리하도록 확장한다.

그래서 슬라이드에서 말하는 핵심은 이거다.

- function A의 argument→ nonterminal A의 inherited attribute
- function A의 return value→ nonterminal A의 synthesized attribute

즉 inherited attribute는 바깥에서 안쪽으로 전달되는 값이므로 함수 인자로 받는다. 반대로 synthesized attribute는 `A`를 처리한 결과로 만들어지는 값이므로 함수의 return 값으로 돌려준다.

예를 들어 `A`가 어떤 inherited attribute를 받아야 한다면 함수는 이런 느낌이 된다.

```
A(inherited 값)
```

그리고 `A`를 처리한 뒤 synthesized attribute를 만들어서 반환한다.

```
result = A(inherited 값)
```

쉽게 말하면 들어오는 값으로 inherited attribute를 사용하고, 나가는 값으로 synthesized attribute을 사용한다는 것이다

---
## Function A가 해야 하는 일

첫 번째로, `A`를 어떤 production으로 전개할지 결정해야 한다.

예를 들어 `A`에 여러 production이 있으면, 현재 입력 token을 보고 어떤 rule을 사용할지 고른다.

```
A → X Y
A → Z
```

이런 경우 현재 입력을 보고 `A → X Y`를 쓸지, `A → Z`를 쓸지 결정해야 한다.


두 번째로, 필요한 terminal이 실제 입력에 있는지 확인해야 한다.

예를 들어 production에 `int`가 필요하면, 현재 입력 token이 정말 `int`인지 확인한다.
맞으면 다음 token으로 넘어가고, 아니면 syntax error가 된다.


세 번째로, attribute 변수들을 보존해야 한다.

parsing 중간에 계산된 attribute는 나중에 다른 nonterminal에게 넘겨야 할 수 있다.

예를 들어 왼쪽 symbol에서 계산한 값을 오른쪽 symbol의 inherited attribute로 넘겨야 한다면, 그 값을 중간 변수에 저장해두어야 한다.


네 번째로, production body 안에 있는 nonterminal에 해당하는 함수를 호출해야 한다.

예를 들어 production이 `A → X Y` 라면 `A()` 함수 안에서는 `X()`를 호출하고, 그다음 `Y()`를 호출한다. 만약 `Y`가 inherited attribute를 필요로 한다면, `X()`에서 얻은 값을 `Y()`의 인자로 넘길 수 있다.

---
## Implementing L-attributed Definitions in Top Down Parsers

앞에서 말한걸 예시로 알아보자

![](../images/Pasted%20image%2020260602140741.png)

### 왼쪽 SDT 해석

왼쪽에는 이런 translation scheme이 있다.

```
D → T { L.in := T.type } L
T → int  { T.type := 'integer' }
T → real { T.type := 'real' }
```

여기서 `D → T { L.in := T.type } L`을 보면, `T`를 먼저 처리한 뒤 `T.type`을 `L.in`으로 넘긴다.

이게 L-attributed 방식과 잘 맞는 이유는, 왼쪽 `T`에서 얻은 정보를 오른쪽 `L`로 넘기기 때문이다.

---
### 오른쪽 코드 해석: `void D()`

오른쪽 코드의 `D()` 함수는 production `D → T L`을 구현한 것이다.

```
void D()
{
    Type Ttype = T();
    Type Lin = Ttype;
    L(Lin);
}
```

이 코드는 왼쪽 SDT와 정확히 대응된다.

먼저 `Type Ttype = T();` 이 부분은 `T`를 처리하는 부분이다. `T()` 함수는 `T.type`을 계산해서 return한다. 그래서 `Ttype`에는 `integer` 또는 `real` 같은 타입 정보가 들어간다.

그다음 `Type Lin = Ttype;` 이 부분이 바로 semantic action이다.

```
L.in := T.type
```

즉 `T.type`을 `L.in`으로 넘기기 위해 `Lin`이라는 변수에 저장한다.

마지막으로 `L(Lin);` 을 통해 L을 처리한다. `L.in`은 inherited attribute이므로 `L()` 함수의 argument로 전달된다.

---
### 오른쪽 코드 해석: `Type T()`

이제 `T()` 함수를 보자.

```
Type T()
{
    Type Ttype;

    if (lookahead == INT)
    {
        Ttype = TYPE_INT;
        match(INT);
    }
    else if (lookahead == REAL)
    {
        Ttype = TYPE_REAL;
        match(REAL);
    }
    else error();

    return Ttype;
}
```

이 함수는 production 두 개를 처리한다.

```
T → int
T → real
```

현재 입력 token이 `INT`이면 `T → int`를 선택한다.

```
Ttype = TYPE_INT;
match(INT);
```

즉 `int`를 읽고, `T.type`을 `TYPE_INT`로 만든다.

현재 입력 token이 `REAL`이면 `T → real`을 선택한다.

```
Ttype = TYPE_REAL;match(REAL);
```

즉 `real`을 읽고, `T.type`을 `TYPE_REAL`로 만든다.

마지막에 `return Ttype;` 을 한다. 이 return 값이 바로 `T`의 synthesized attribute다.

---
### 오른쪽 코드 해석: `void L(Type Lin)`

아래에는 이런 함수가 있다.

```
void L(Type Lin)
{
    ...
}
```

여기서 `Lin`이 바로 `L.in`이다. `L.in`은 inherited attribute였다.
그래서 함수 안에서 계산해서 return하는 것이 아니라, 바깥에서 인자로 받아온다.

예를 들어 입력이 `int a, b`라면 `T()`가 `TYPE_INT`를 return하고, `D()`는 그 값을 `L(TYPE_INT)`로 넘긴다. 그러면 `L()`은 `a`, `b` 같은 identifier들에게 integer 타입을 붙일 수 있다.

>정리해보면, 지금까지 배운 `SDD → SDT` 과정은 이론적으로 attribute 계산 순서와 semantic action의 위치를 정하는 과정이라고 볼 수 있다.
>
>이를 실제 recursive-descent parser 코드로 구현하면 더 단순하게 이해할 수 있다. 각 nonterminal은 하나의 함수로 대응되고, synthesized attribute는 해당 함수를 처리한 결과이므로 return 값으로 전달된다. 반대로 inherited attribute는 해당 nonterminal을 처리하기 전에 외부에서 받아야 하는 값이므로 함수의 argument로 전달된다.
>
>그리고 semantic action은 특별한 구조가 아니라, 실제 코드 안에서 수행되는 일반적인 대입문이나 함수 호출로 구현하면 된다.

---
# Ex 5.20  `while (C) S1`

![](../images/Pasted%20image%2020260602141555.png)

- `S.next`는 while 문이 끝난 뒤 이동할 위치다.

- `C.false = S.next`는 조건이 false이면 while 문 밖으로 나가라는 뜻이다.

- `C.true = L2`는 조건이 true이면 while body 시작 위치로 가라는 뜻이다.

- `S1.next = L1`은 while body 실행이 끝나면 다시 조건 검사 위치로 돌아가라는 뜻이다.

- 마지막 `S.code`는 `C.code`와 `S1.code`를 합쳐서 while 문 전체 코드를 만드는 것이다.

---

![](../images/Pasted%20image%2020260602141631.png)


```text
string S(label next)
```

여기서 `next`가 바로 `S.next`다.

`S.next`는 inherited attribute였다. 즉 `S`가 자기 내부에서 만드는 값이 아니라, 바깥 문맥에서 “이 statement가 끝나면 어디로 갈지”를 받아오는 값이다.

그래서 함수의 argument로 들어간다.

### local variable: `Scode`, `Ccode`, `L1`, `L2`

함수 안에는 이런 local variable이 있다.

```text
string Scode, Ccode;
label L1, L2;
```

- `Ccode`는 조건식 `C`가 만든 코드 조각이다.

- `Scode`는 body statement `S1`이 만든 코드 조각이다.

- `L1`, `L2`는 while 문을 만들기 위해 새로 생성하는 label이다.
	- 여기서 `L1`은 조건 검사 위치이고, `L2`는 while body 시작 위치다.

---
### `if (current input == token while)`

이 부분은 recursive-descent parser의 기본 역할이다.

현재 입력이 `while`이면 이 production을 선택한다.

```text
S → while ( C ) S1
```

즉 parser가 “이번 statement는 while 문이구나”라고 판단하는 부분이다.

그다음 `while`을 소비하고, `(`도 확인한다.

```text
advance input;
check '(' is next on the input, and advance;
```

---
### `L1 = new(); L2 = new();`

이 부분은 SDT의 첫 번째 action에 해당한다.

```c
L1 = new(); // 조건 검사 위치
L2 = new(); // body 시작 위치
```

while 문 코드 생성을 위해 label 두 개를 새로 만든다.

---
### `Ccode = C(next, L2);`

이 부분이 제일 중요하다.

원래 SDT에서는 이렇게 되어 있었다.

```c
C.false = S.next;
C.true = L2;
```

그런데 코드에서는 `S.next`가 함수 인자 `next`로 들어와 있다.

그래서 `C`를 호출할 때 다음처럼 넘긴다.

```c
Ccode = C(next, L2);
```

즉 이건 의미상

```c
C.false = next
C.true = L2
```

를 넘기는 것이다.

여기서 `next`와 `L2`는 `C`의 inherited attribute로 들어간다. 그리고 `C()`는 조건식 코드를 만들어서 return한다. 그래서 `Ccode`는 `C.code`에 해당한다.

---
### `Scode = S(L1);`

이 부분은 while body인 `S1`을 처리하는 부분이다.

원래 SDT에서는 `S1.next = L1;` 이었다. 즉 body 실행이 끝나면 다시 조건 검사 위치 `L1`로 돌아가야 한다.

코드에서는 이것을 다음처럼 구현한다.

```c
Scode = S(L1);
```

여기서 `L1`은 body statement `S1`의 inherited attribute로 전달된다.

그리고 `S(L1)`은 body code를 만들어서 return한다.

그래서 `Scode`는 `S1.code`에 해당한다.

---
### `return ...`

마지막 부분은 synthesized attribute를 return하는 부분이다.

```c
return("label" || L1 || Ccode || "label" || L2 || Scode);
```

이건 원래 SDT의 마지막 action과 같다.

```c
S.code = label || L1 || C.code || label || L2 || S1.code;
```

즉 while 문 전체 코드를 조립해서 return한다. 여기서 `S.code`는 synthesized attribute다.
왜냐하면 `S.code`는 `Ccode`, `Scode` 같은 자식 결과를 이용해서 부모 `S`에서 만들어지는 값이기 때문이다. 따라서 코드에서는 return 값으로 구현된다.

---
# On the fly code generation

앞에서는 `while` 문을 recursive-descent parser 코드로 구현하면서, `Ccode`, `Scode` 같은 문자열을 return 받아 마지막에 합쳤다.

```text
Ccode = C(next, L2);
Scode = S(L1);

return "label" || L1 || Ccode || "label" || L2 || Scode;
```

이 방식은 개념을 이해하기에는 좋다. `C.code`, `S1.code`, `S.code` 같은 synthesized attribute가 실제 코드에서는 return 값으로 표현된다는 것을 보여주기 때문이다.

그런데 실제 구현 관점에서는 문제가 있다.

`Ccode`와 `Scode`가 짧은 문자열이면 괜찮지만, 실제 프로그램에서는 조건식이나 statement가 길어질 수 있다. 그러면 `Ccode`, `Scode`, `S.code` 같은 코드 조각도 길어진다. 이 긴 문자열들을 계속 return하고, 복사하고, `||`로 이어 붙이면 비효율적이다.

그래서 **on-the-fly code generation**이 나온다.

---
on-the-fly code generation은 `S.code`라는 긴 문자열을 다 만든 뒤 return하는 방식이 아니라, parsing 중에 코드가 필요한 순간마다 바로 생성하는 방식이다.

예를 들어 앞에서는 while 문 전체 코드를 마지막에 이렇게 합쳤다.

```text
S.code = label || L1 || C.code || label || L2 || S1.code
```

하지만 on-the-fly 방식에서는 굳이 `S.code` 전체를 문자열로 만들지 않는다.

대신 parsing 중에 순서대로 바로 emit한다.

```text
emit(label L1)
C를 처리하면서 C.code에 해당하는 코드 emit
emit(label L2)
S1을 처리하면서 S1.code에 해당하는 코드 emit
```

즉 이전 방식이 “코드 조각을 문자열로 들고 있다가 마지막에 합치기”라면, on-the-fly 방식은 “코드가 나와야 하는 시점에 바로 출력하거나 저장하기”라고 보면 된다.

이렇게 하면 긴 문자열을 계속 복사하지 않아도 된다.

---

다만 아무 경우에나 on-the-fly로 바꿀 수 있는 것은 아니다. 코드가 생성되는 순서가 parser가 production body를 처리하는 순서와 잘 맞아야 한다.

슬라이드에서 말하는 `main attribute`는 어떤 nonterminal의 대표 결과 attribute를 의미한다. 예를 들어 statement `S`의 대표 결과는 보통 `S.code`다. 조건식 `C`도 `C.code`를 대표 attribute로 볼 수 있다.

이 main attribute가 synthesized attribute여야 한다는 말은, 그 결과가 자식들을 처리한 뒤 만들어지는 값이어야 한다는 뜻이다.

예를 들어 while 문에서는:

```text
S.code = label L1 || C.code || label L2 || S1.code
```

`S.code`는 `C.code`와 `S1.code`를 이용해 만들어진다. 즉 자식의 결과를 부모가 조립하는 synthesized attribute다.

그리고 중요한 조건은, 이 조립 순서가 production body의 순서와 맞아야 한다는 것이다.

while 문 production은 다음과 같다.

```text
S → while ( C ) S1
```

여기서 `C`가 먼저 나오고, 그 다음 `S1`이 나온다.

코드 생성 순서도 다음과 같다.

```text
label L1
C.code
label L2
S1.code
```

즉 `C.code`가 먼저 나오고, `S1.code`가 뒤에 나온다. production에서 `C`가 `S1`보다 앞에 있는 것과 코드 생성 순서가 맞는다.

그래서 이런 경우에는 on-the-fly code generation이 가능하다.

반대로 만약 production에서는 `C`를 먼저 처리해야 하는데, 최종 코드에서는 `S1.code`가 `C.code`보다 먼저 나와야 한다면 바로 emit하기 어렵다. 그 경우에는 결국 코드 조각을 임시로 저장해두거나 나중에 재배치해야 한다.

---
# EX 5.22

![](../images/Pasted%20image%2020260602143340.png)


```
print("label", L1);
```

이 부분이 on-the-fly code generation의 핵심이다.

이전에는 마지막에

```
"label" || L1 || Ccode || "label" || L2 || Scode
```

로 합쳤다.

그런데 지금은 `L1` label이 필요한 순간에 바로 출력한다.

즉 중간 코드에 바로 이런 줄을 생성하는 것이다.

```
label L1
```

---

```
C(next, L2);
```

조건식 `C`를 처리한다. 여기서 인자로 `next`, `L2`를 넘긴다.
조건이 true이면 body 시작 label인 `L2`로 가고, false이면 while 문 밖인 `next`로 간다.


```
Ccode = C(next, L2);
```

이전 방식에서는 위처럼 `C`가 만든 코드를 문자열로 받아왔다.
하지만 지금은 return을 받지 않는다.

왜냐하면 `C()` 내부에서 조건식에 해당하는 중간 코드를 직접 출력하기 때문이다.

예를 들어 `C`가 `i < 10`이라면 `C()` 안에서 이런 코드가 바로 출력될 수 있다.

```
if i < 10 goto L2
goto next
```

---
```
print("label", L2);
```

이제 while body가 시작되는 위치를 출력한다.

중간 코드에 이런 줄을 생성하는 것이다.

```
label L2
```

---

```
S(L1);
```

while body인 `S1`을 처리한다.

여기서 `L1`을 인자로 넘기는 이유는 body 실행이 끝나면 다시 조건 검사 위치 `L1`로 돌아가야 하기 때문이다.

```
Scode = S(L1);
```

여기서도 이전 방식에서는 위처럼 body code를 문자열로 return 받았다.
하지만 지금은 `S(L1)` 안에서 body에 해당하는 코드가 바로 출력된다.

---
# L-attributed SDD's and LL parsing

앞에서는 nonterminal을 함수로 보고 아래와 같이 처리했다.

- inherited attribute  → 함수 argument
- synthesized attribute → 함수 return value

그런데 모든 top-down parser가 recursive function (함수) 형태로만 구현되는 것은 아니다. LL parser는 **parser stack**을 사용해서도 구현할 수 있다. 이 슬라이드는 바로 그 경우에 attribute와 semantic action을 stack에 어떻게 넣을 것인지 설명한다.

## L-attributed SDD is based on an LL-grammar

L-attributed SDD는 LL parser와 잘 맞는다.

왜냐하면 LL parser는 production을 왼쪽에서 오른쪽으로 처리하고, L-attributed SDD도 attribute 정보가 왼쪽에서 오른쪽으로 흐르기 때문이다.

예를 들어

```text
D → T { L.in := T.type } L
```

여기서 `T`를 먼저 처리하고, 그 결과인 `T.type`을 이용해 `L.in`을 만든 다음 `L`을 처리한다.

이런 흐름은 LL parser의 처리 순서와 잘 맞는다.

즉 이 말은 **L-attributed SDD는 LL parser가 처리하는 순서대로 inherited attribute를 넘기고 synthesized attribute를 받을 수 있는 구조다.**

### We can embed actions into productions

앞에서 SDT를 만들 때 semantic action을 production 안에 넣었다.

```text
D → T { L.in := T.type } L
```

이런 식으로 action을 production 중간에 넣을 수 있다.

LL parser에서도 이 action을 실제 parsing 과정 중에 실행해야 한다. 그런데 LL parser가 stack 기반으로 구현되어 있다면, stack에는 원래 grammar symbol만 들어간다.

예를 들어

```text
D → T { action } L
```

을 처리하려면 stack에 단순히 `T`, `L`만 넣으면 안 된다. 중간에 `{ action }`도 실행해야 하니까, parser stack이 action도 저장할 수 있어야 한다.

즉 LL parser stack을 확장해서 다음 것들을 같이 저장한다.

```text
grammar symbol
semantic action
attribute data
```

### However extra information should be kept

stack에 symbol만 넣으면 parsing은 가능하지만, attribute 계산은 어렵다.

예를 들어 `L.in := T.type`을 실행하려면 `T.type` 값을 알고 있어야 하고, 그 값을 `L`에게 전달할 공간도 필요하다.

그래서 parser stack에는 추가 정보가 필요하다.

단순히 `T`, `L` 만 저장하는 것이 아니라,

```text
T의 synthesized attribute 저장 위치
L의 inherited attribute 저장 위치
action record
```

같은 정보도 함께 관리해야 한다. 여기서 핵심은 **LL parser stack을 attribute 계산까지 가능한 구조로 확장해야 한다**는 것이다.

---
## Managing stack to attribute handling

이제 stack에서 inherited attribute와 synthesized attribute를 어떻게 저장하는지가 나온다.

### inherited attribute는 nonterminal과 함께 stack에 둔다
`A`라는 nonterminal을 처리할 때 필요한 inherited attribute를 `A`와 같이 stack에 넣는다.

예를 들어 `L.in`이 필요하면, stack에 `L`을 넣을 때 `L.in` 값도 같이 넣어둔다.

```text
L, L.in
```

왜냐하면 `L`을 expand할 때 `L.in` 값을 바로 써야 하기 때문이다.

`A`를 처리하기 전에 실행해야 하는 action이 있다면, 그 action record를 `A`보다 위쪽에 두어야 한다. LL parser stack은 top에서부터 처리되니까, `A` 위에 action이 있으면 `A`를 처리하기 전에 action이 먼저 실행된다.

즉 inherited attribute를 계산하는 action은 해당 nonterminal 전에 실행되어야 하므로, stack에서도 그 순서가 유지되도록 배치한다.

### synthesized attribute는 별도 synthesize-record에 둔다

synthesized attribute는 자식들을 처리한 뒤 부모로 올라가는 값이다. 그래서 `A`를 처리한 결과를 저장할 공간이 필요하다.

이것을 separate synthesize-record라고 한다. 즉 `A`가 처리된 뒤 만들어질 synthesized attribute를 저장할 record를 `A` 아래쪽에 따로 둔다.

```text
synthesize-record
A
action-record
```

이런 식으로 stack에 관련 정보를 배치해서, `A`가 처리된 뒤 결과값을 synthesize-record에 저장할 수 있게 한다.

---
# Ex 5.23 Prdictive LL parser with Stack 

우리가 다루는 production은 while 문이다.

![](../images/Pasted%20image%2020260602144904.png)

여기서 필요한 attribute는 다음과 같다.

- S.next   : while 문이 끝난 뒤 이동할 위치
- C.false  : 조건이 false일 때 이동할 위치
- C.true   : 조건이 true일 때 이동할 위치
- S1.next  : body 실행 후 이동할 위치


그리고 label은 다음과 같다.

- L1 : 조건 검사 시작 위치
- L2 : while body 시작 위치

----
![](../images/Pasted%20image%2020260602145059.png)

### (a) 처음 stack 상태

왼쪽 위 그림을 보면 stack top에 `S`가 있고, 그 아래에 `next = x`가 있다. 이 말은 현재 parser가 nonterminal `S`를 처리하려는 상황이고, `S.next` 값이 이미 `x`로 들어와 있다는 뜻이다.

여기서 `x`는 while 문이 끝났을 때 이동할 label이다.

```
S.next = x
```

이 값은 inherited attribute다. 왜냐하면 `S`가 자기 내부에서 만든 값이 아니라, 바깥 문맥이 `S`에게 넘겨준 값이기 때문이다.

---
## S를 production으로 확장한 상태

이제 LL parser가 `S → while ( C ) S1` production을 선택하면 stack에 production body를 넣는다. 그런데 단순히 문법 symbol만 넣는 게 아니라, 중간 action과 attribute record도 같이 넣는다.

그래서 그림처럼 stack에 다음 요소들이 생긴다.

![](../images/Pasted%20image%2020260602170904.png)

여기서 중요한 건 `Action` record가 두 개 들어간다는 점이다.

첫 번째 action은 `C`를 처리하기 전에 실행되어야 한다.
왜냐하면 `C.false`, `C.true`를 미리 정해줘야 `C`가 조건식 코드를 만들 수 있기 때문이다.

두 번째 action은 `S1`을 처리하기 전에 실행되어야 한다.
왜냐하면 `S1.next`를 미리 정해줘야 `S1`이 body 코드를 만들 수 있기 때문이다.

---
## 첫 번째 Action record

첫 번째 action record에는 이런 정보가 있다.

```
snext = x
L1 = ?
L2 = ?
```

여기서 `snext = x`는 원래 `S.next = x`였던 값을 action record가 복사해서 들고 있는 것이다.
왜 복사하냐면, 이 action이 실행될 때 `S.next` 값을 사용해야 하기 때문이다.

첫 번째 action이 하는 일은 아래 박스에 적혀 있다.

```
L1 = new();
L2 = new();

stack[top - 1].false = snext;
stack[top - 1].true = L2;

stack[top - 3].al1 = L1;
stack[top - 3].al2 = L2;

print("label", L1);
```

LL parser는 production의 오른쪽 부분을 왼쪽부터 처리해야 한다. 하지만 stack은 나중에 넣은 것이 먼저 나오는 LIFO 구조이므로, production RHS를 stack에 넣을 때는 오른쪽에서 왼쪽 순서로 push한다. 그래야 가장 왼쪽 symbol이 stack top에 올라와 먼저 처리된다.

예를 들어 다음 production이 있다고 하면,

```
S → while ( Action1 C ) Action2 S1
```

처리 순서는 다음과 같아야 한다.

```
while → ( → Action1 → C → ) → Action2 → S1
```

따라서 stack에는 반대로 넣는다.

그러면 stack top에는 `while`이 올라오고, parser는 top부터 하나씩 처리하면서 왼쪽에서 오른쪽 순서로 production을 수행할 수 있다.

이 구조를 이해하면 `top-1`, `top-3` 같은 상대 위치 접근도 이해된다. parser는 `S → while ( Action1 C ) Action2 S1` production으로 확장할 때, stack에 들어가는 symbol들의 순서를 이미 알고 있다. 그래서 `while`과 `(`가 처리된 뒤 `Action1`이 실행되는 시점에는, 현재 top이 `Action1`이고 그 아래쪽에 앞으로 처리될 `C`, `)`, `Action2`, `S1`이 정해진 순서로 놓여 있다.

따라서 첫 번째 action은 현재 action record를 기준으로 상대 위치를 이용해 값을 넣을 수 있다. 예를 들어 `top-1`에는 곧 처리될 `C` record가 있으므로 `C.false`, `C.true` 값을 채울 수 있다. 또한 `top-3`에는 나중에 실행될 두 번째 action record가 있으므로, 그 action이 사용할 `L1`, `L2` 값을 미리 저장할 수 있다.

---
핵심만 보자

첫째, `C`가 처리되기 전에 `C.false`, `C.true`를 미리 채운다. 이는 조건식 `C`가 중간 코드를 만들 때 true/false일 때 이동할 label을 알아야 하기 때문이다.

둘째, `L1`, `L2`는 나중에 `Action2`에서도 필요하므로 두 번째 action record에 저장해둔다. `Action2`는 이후 `S1`을 처리하기 전에 실행된다.

```
Action2 실행 내용S1.next = L1;print("label", L2);
```

`S1.next = L1`은 while body가 끝난 뒤 다시 조건 검사 위치로 돌아가게 하기 위한 것이다. 그리고 `label L2`는 while body가 시작되는 위치를 출력하는 것이다.