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
## stack에 오른쪽부터 push하는 이유
LL parser는 production의 오른쪽 부분을 왼쪽부터 처리해야 한다.

```
while → ( → Action1 → C → ) → Action2 → S1
```

하지만 stack은 나중에 넣은 것이 먼저 나오는 LIFO 구조다. 따라서 실제로 stack에 넣을 때는 오른쪽에서 왼쪽 순서로 push한다.

```
push S1
push Action2
push )
push C
push Action1
push (
push while
```

그래야 stack top에는 `while`이 올라오고, parser는 top부터 하나씩 처리하면서 production을 왼쪽에서 오른쪽 순서로 수행할 수 있다.

---
## 첫 번째 Action record

`while`과 `(`가 처리되고 나면 `Action1`이 stack top에 온다.

이때 stack 안에는 앞으로 처리될 `C`, `)`, `Action2`, `S1`이 이미 정해진 순서로 놓여 있다. 그래서 `Action1`은 현재 top을 기준으로 상대 위치를 이용해 다른 record에 attribute 값을 넣을 수 있다.

첫 번째 action의 내용은 다음과 같다.

```
L1 = new();
L2 = new();

stack[top - 1].false = snext;
stack[top - 1].true = L2;

stack[top - 3].al1 = L1;
stack[top - 3].al2 = L2;

print("label", L1);
```

여기서 `snext`는 원래 `S.next` 값이다. 즉 while 문이 끝난 뒤 이동할 위치다.

`stack[top - 1]`은 곧 처리될 `C` record를 가리킨다. 따라서 다음 코드는

```
stack[top - 1].false = snext;
stack[top - 1].true = L2;
```

결국 다음 의미와 같다.

```
C.false = S.next;
C.true = L2;
```

즉 조건식이 false이면 while 문 밖으로 나가고, true이면 body 시작 위치인 `L2`로 이동하도록 만드는 것이다.

또한 `L1`, `L2`는 나중에 실행될 `Action2`에서도 필요하다. 그래서 첫 번째 action은 `stack[top - 3]`, 즉 두 번째 action record에 이 label들을 미리 저장한다.

```
Action2.al1 = L1;
Action2.al2 = L2;
```

마지막으로 `print("label", L1)`은 조건 검사 시작 위치를 중간 코드에 바로 출력하는 것이다.

---
## 두 번째 Action record

`C`와 `)`가 처리된 뒤에는 `Action2`가 실행된다.

`Action2`에는 앞에서 저장해둔 `L1`, `L2`가 들어 있다.

```
al1 = L1
al2 = L2
```

두 번째 action의 내용은 다음과 같다.

```
stack[top - 1].next = al1;
print("label", al2);
```

여기서 `stack[top - 1]`은 곧 처리될 `S1` record를 가리킨다. 따라서

```
stack[top - 1].next = al1;
```

은 다음 의미와 같다.

```
S1.next = L1;
```

즉 while body가 끝나면 다시 조건 검사 위치로 돌아가게 한다.

그리고

```
print("label", al2);
```

는 body 시작 label인 `L2`를 중간 코드에 출력하는 것이다.

----
# Implementing L-Attributed Definitions in Bottom-UP Parsers

앞에서는 LL parser, recursive descent parser 처럼 top-down 방식에서 L-attributed SDD를 구현하는 방법을 보았다.

```text
inherited attribute → 함수 argument
synthesized attribute → 함수 return value
```

또는 stack 기반 LL parser에서는 필요한 attribute를 nonterminal record에 미리 넣어줬다. 그런데 이제 bottom-up parser, 즉 LR parser 같은 방식에서 L-attributed SDD를 구현하려고 하면 문제가 생긴다.

---
### 왜 Bottom-Up Parser에서는 더 어려운가

Bottom-up parser는 기본적으로 **오른쪽 body가 다 인식된 뒤 reduce**한다.

예를 들어 `A → X Y` 이면 `X Y`가 모두 stack에 올라온 뒤에야 `A`로 reduce한다.
이 방식은 synthesized attribute와는 잘 맞는다.

```text
A.val = X.val + Y.val
```

처럼 자식 값이 다 준비된 뒤 부모 값을 계산하면 되기 때문이다.

하지만 L-attributed SDD에는 inherited attribute가 있다.
예를 들어 `A → X { Y.in = X.val } Y` 이런 경우 `Y`를 처리하기 전에 `Y.in`이 미리 필요하다.

그런데 bottom-up parser는 `Y`까지 다 읽고 나서 reduce하는 방식이라, production 중간에 있는 action을 바로 실행하기가 어렵다. 그래서 **Bottom-Up Parser에서 L-attributed definition을 구현하는 것은 더 어렵고, translation scheme으로 다시 작성해야 한다**.

---
### 중간 action을 marker nonterminal로 바꾸는 이유

L-attributed SDT에서는 action이 production 중간에 들어갈 수 있었다.

```text
A → X { actions } Y
```

여기서 `{ actions }`는 `X`를 처리한 뒤, `Y`를 처리하기 전에 실행되어야 한다.

하지만 bottom-up parser는 production 중간에 있는 action을 문법 symbol처럼 자연스럽게 처리하지 못한다.

그래서 이 action을 하나의 가짜 nonterminal로 바꾼다.

```text
A → X N Y
N → ε { actions }
```

여기서 `N`이 **marker nonterminal**이다.

`N → ε`이므로 실제 입력 token은 소비하지 않는다.  
대신 parser가 `N → ε`로 reduce하는 순간 `{ actions }`를 실행한다.

즉 원래는 production 중간에 있던 action을, 문법적으로는 nonterminal `N`으로 바꿔서 bottom-up parser가 처리할 수 있게 만드는 것이다.

---
## 예시로 보면

원래 SDT가 다음과 같다고 하자.

```text
A → X { actions } Y
```

X를 처리하고, action을 실행한 다음 Y를 처리한다.

이걸 bottom-up parser에서 구현하기 위해 이렇게 바꾼다.

```text
A → X N Y
N → ε { actions }
```

이제 parser 입장에서는 `N`도 하나의 nonterminal이다.

흐름은 이렇게 된다.

- X가 stack에 올라온다.
- N → ε로 reduce하면서 actions를 실행한다.
- 그 다음 Y를 처리한다.
- 마지막에 X N Y를 A로 reduce한다.

즉 marker nonterminal `N`은 **입력은 소비하지 않지만, action을 실행하기 위한 위치 표시자**라고 보면 된다.

---
## 왜 conflict가 생길 수 있나

marker nonterminal `N`은 `N → ε`이다. 즉 아무 입력도 읽지 않고 바로 reduce될 수 있다.

Bottom-up parser에서는 어떤 시점에 다음 token을 shift할지, 아니면 `N → ε`로 reduce할지 결정해야 한다.

그런데 `N → ε` 같은 규칙을 추가하면 parser table에서 이런 갈등이 생길 수 있다.

```text
지금 N → ε로 reduce해야 하나?
아니면 다음 input token을 shift해야 하나?
```

이게 shift/reduce conflict로 이어질 수 있다. 그래서 marker nonterminal은 중간 action을 bottom-up parser에서 구현하기 위한 방법이지만, 항상 안전한 것은 아니다. parse table에 새로운 conflict를 만들 수 있다는 점을 조심해야 한다.

---
# Bottom-Up parsing of L-Attributed SDDs

### M이 action에 필요한 값을 어떻게 아는가?

원래 action `{ a }`는 `A`의 attribute나 `α`에 있는 symbol들의 attribute를 사용할 수 있다.

예를 들어 `A → α { a } β` 에서 `{ a }`가 다음과 같은 값을 필요로 한다고 하자.

```
A.in
α.s
```

그러면 marker nonterminal `M`이 이 값들을 알아야 한다.

>**원래 action이 필요로 하던 값들을 M의 inherited attribute로 복사해 둔다.**

즉 `M`이 실행될 때 필요한 재료를 미리 들고 있게 만드는 것이다.

---
### M이 계산한 결과는 synthesized attribute로 만든다

원래 action `{ a }`는 뒤쪽 symbol `β`에게 넘길 inherited attribute를 계산할 수 있다.

예를 들어

```
A → α { B.in = f(A.in, α.s) } B
```

여기서 action은 `B.in`을 계산한다.
이걸 marker nonterminal로 바꾸면, `M`이 먼저 그 값을 계산하게 한다.

```
A → α M B
M → ε { M.s = f(M.i, α.s) }
```

그리고 이후 `B.in`은 `M.s`를 이용해 채운다고 보면 된다.

원래 action `{ a }`가 계산하던 값을 그대로 계산하되, 그 결과를 바로 뒤쪽 symbol에 직접 넣는 게 아니라 **M의 synthesized attribute로 만들어 둔다**는 뜻이다.

즉 흐름은 이렇게 바뀐다.

- 기존 : A 또는 α의 정보 → action a → β.in

- 변환 후 : 
	A 또는 α의 정보 → M의 inherited attribute
	M에서 action 실행 → M의 synthesized attribute
	M.s를 이용해 β 쪽으로 전달

---
## 왜 이렇게 복잡하게 하나?

bottom-up parser에서는 reduce 시점이 중요하다. `M → ε`은 입력을 소비하지 않고 reduce될 수 있으므로, `M`이 reduce되는 순간 action을 실행할 수 있다.

그런데 이때 `M`은 자기만의 attribute를 가져야 한다.

그래야 stack 기반으로 다음과 같이 처리할 수 있다.

```
α까지 처리됨
→ M → ε reduce
→ M이 필요한 값을 받아 action 실행
→ 결과를 M.s에 저장
→ 이후 β 처리에 사용
```

즉 marker nonterminal `M`은 단순한 표시자가 아니라, **중간 action의 입력과 출력을 attribute 형태로 들고 있는 record 역할**을 한다.

---
# Translation Schemes using Marker Nonterminals

**직접 stack을 쓰던 것을 marker nonterminal의 synthesized attribute로 대체**하는 예시를 보자

![](../images/Pasted%20image%2020260602181145.png)

원래 방식은 이렇게 되어 있다.

```
S → if E { 
        emit(iconst_0);
        push(pc);
        emit(if_icmpeq, 0);
    }
    then S { 
        backpatch(top(), pc - top());
        pop();
    }
```

여기서 첫 번째 action은 `E`가 끝난 직후 실행된다. 이때 조건식 `E`의 결과가 stack 위에 있다고 보고, `0`과 비교해서 false이면 `then S`를 건너뛰도록 jump 명령을 만든다.

문제는 이 jump 명령의 목적지를 아직 모른다는 것이다.  
왜냐하면 `then S`가 아직 생성되지 않았기 때문이다.

그래서 일단 현재 명령어 위치 `pc`를 저장해 둔다.

```
push(pc);
emit(if_icmpeq, 0);
```

그리고 `then S`까지 다 처리한 뒤에야 현재 `pc`가 “if문이 끝나는 위치”가 된다.  
그때 저장해 둔 위치를 꺼내서 jump 거리를 채운다.

```
backpatch(top(), pc - top());pop();
```

---

이걸 marker nonterminal로 바꾸면 이렇게 된다.

```
S → if E M then S { backpatch(M.loc, pc - M.loc) }

M → ε { 
        emit(iconst_0);
        M.loc := pc;
        emit(if_icmpeq, 0);
    }
```

여기서 `M`은 실제 입력 토큰을 소비하지 않는다. `M → ε` 이기 때문에 문법적으로는 아무것도 안 읽지만, **그 위치에서 action을 실행하기 위해 끼워 넣은 가짜 nonterminal**이다.

즉 `M`의 역할은 이거다. E를 처리한 직후,then S를 처리하기 전에,미리 실행해야 하는 action을 수행하고,나중에 필요한 값 pc를 M.loc에 저장한다.


> marker nonterminal이 추가되는 건 그 다음에 나올 것들을 처리하기 전에 필요한 action을 먼저 하고, 그 결과를 M 자체에 저장한 다음, 나중 action에서 넘겨받아 쓰는 것이다.

이렇게 이해하면 된다.

>**M이 “다음 것들의 정보”를 미리 아는 것은 아니다.**  

`M`은 아직 `then S`의 결과를 모른다. 대신 나중에 `then S`가 끝났을 때 필요한 과거 정보, 즉 “jump 명령이 있던 위치”를 저장해 둔다.



즉 흐름은 이렇게 된다.

```
if E
```

여기까지 처리하면 조건식 코드가 만들어져 있다.

```
M → ε action 실행
```

여기서 false jump 명령을 만들고, 그 위치를 `M.loc`에 저장한다.

```
then S
```

then 안의 문장 코드를 생성한다.

```
마지막 action 실행
```

이제 `pc`가 then문 끝 위치를 가리키므로,

```
backpatch(M.loc, pc - M.loc)
```

으로 jump 목적지를 채운다.

---
# Ex 5.25

### 위를 아래와 같이 바꾸는 것이 타당한가?

```
A → { B.i = f(A.i); } B C
```

```
A → M B C
M → ε { M.i = A.i; M.s = f(M.i); }
```


>결론부터 말하면 **그냥 이렇게 바꾸면 legal하지 않다**고 보는 게 맞다..

원래 의도는 아래와 같다.

```
A → { B.i = f(A.i); } B C
```

A의 inherited attribute A.i를 이용해서 B의 inherited attribute B.i를 미리 계산한다.
즉 `B`를 parsing하기 전에 `B.i`가 필요하니까, action을 `B` 앞에 둔 거다.

그런데 marker로 바꾸면서 이렇게 했다.

```
A → M B C
M → ε { M.i = A.i; M.s = f(M.i); }
```

여기서 문제가 생긴다.

`M → ε`의 semantic action 안에서 `A.i`를 쓰고 있다.

```
M.i = A.i
```

그런데 `M → ε`라는 production만 보면, 이 production에 등장하는 symbol은 `M`뿐이다.

```
M → ε
```

즉 이 production의 semantic action에서 직접 접근할 수 있는 건 기본적으로 `M`의 attribute뿐이다.  그런데 갑자기 바깥 production의 head인 `A.i`를 참조하고 있다.

그래서 문법적으로 보면 이건 이상한 형태다.

```
M → ε { M.i = A.i; }
```

여기서 `A`는 `M → ε` production 안에 존재하지 않는다.

---

LR parser 관점에서도 문제가 있다.

`A → M B C`를 parsing할 때, `M → ε`는 제일 먼저 reduce된다.

그 순간 stack 상태는 대략 이렇다.

```
... 
```

아직 `A`는 만들어지지 않았다.  왜냐하면 `A`는 나중에 `M B C` 전체가 완성되어야 reduce되기 때문이다. 즉 `M → ε` action이 실행되는 시점에는 `A.i` 를 어디서 가져와야 하는지가 명확하지 않다.

그래서 이 변환은 그냥 두면 legal하지 않다.

>marker는 LR parser의 reduce-action 방식에 맞추기 위한 수단이다. 하지만 그 수단을 쓰려면, reduce 시점에 필요한 데이터가 stack이나 전역 상태에 준비되어 있어야 한다.
>이 예시는 A.i가 준비되어 있지 않아서 문제가 된다.

---
# Ex 5.26

![](../images/Pasted%20image%2020260602183542.png)

여기서 action이 두 번 중간에 들어간다.

첫 번째 action은 `C`를 처리하기 전에 실행되어야 한다.

```
L1 = new()
L2 = new()
C.true = L2
C.false = S.next
print(label L1)
```

왜냐하면 조건식 `C`를 번역할 때 이미 `C.true`, `C.false`가 필요하기 때문이다.

두 번째 action은 `S1`을 처리하기 전에 실행되어야 한다.

```
S1.next = L1
print(label L2)
```

왜냐하면 while문의 body인 `S1`이 끝나면 다시 while문의 시작 위치 `L1`로 돌아가야 하기 때문이다.

---
그래서 marker를 넣어서 이렇게 바꾼다.

![](../images/Pasted%20image%2020260602183640.png)

여기서 `M`은 첫 번째 중간 action을 대신한다.
`N`은 S1 가기 전에 action을 대신한다.

![](../images/Pasted%20image%2020260602183823.png)


LR parser는 입력을 왼쪽에서 오른쪽으로 shift하면서 stack에 쌓고,완성된 부분을 reduce하면서 bottom-up으로 올라간다.

- ?에는 S.next 같은 inherited attribute가 이미 저장되어 있다고 본다.

- while shift
- ( shift

- M → ε reduce
	이때 첫 번째 중간 action 실행
	L1, L2를 만들고, C.true, C.false를 설정한다.
	C.false는 바깥 S.next를 stack에서 가져온다.

- 그다음 C를 parsing/reduce하면서 C.code를 만든다.
	이때 C.true, C.false는 M이 준비해 둔 값을 사용한다.

- ) shift

- N → ε reduce
	이때 M.L1을 가져와서 S1.next에 넣는다. S1이 끝나면 while 시작 위치 L1로 돌아가게 한다.

- 그다음 body 부분을 parsing/reduce해서 S1.code를 만든다.

- 마지막으로 S → while ( M C ) N S1을 reduce
	M.L1, C.code, M.L2, S1.code를 조합해서 S.code를 만든다.

![](../images/Pasted%20image%2020260602214559.png)

---
# Rewriting a Grammer to Avoid Inherited Attributed

marker를 사용하면 중간 action을 `M → ε` 형태의 reduce action으로 바꿀 수 있지만, marker action에서 필요한 값이 reduce 시점에 stack에 존재하지 않으면 문제가 생길 수 있다.

그래서 이번에는 아예 문법을 바꿔 inherited attribute가 필요하지 않도록 만드는 방법을 본다.

![](../images/Pasted%20image%2020260602214856.png)

예를 들어 입력이 다음과 같다고 하자.

```
id1, id2, id3 : int
```

여기서 타입 정보는 `T → int`를 보고 나서야 알 수 있다.  
그런데 타입을 붙여야 하는 `id1`, `id2`, `id3`는 왼쪽의 `L` 안에 있다.

즉 정보 흐름이 이렇게 된다.

```
T.type → L 안의 id들
```

오른쪽에서 얻은 정보를 왼쪽으로 넘겨야 하므로 inherited attribute가 필요하다.

---

그래서 문법을 이렇게 바꾼다.

```
D → id L  
T → int  
T → real  
L → , id L1  
L → : T
```

이제 `id`들을 먼저 읽다가, 마지막에 `: T`를 만난다.

```
id1, id2, id3 : int
```

이 구조에서는 가장 안쪽의 `: T`에서 타입이 결정된다.

```
L → : TL.type = T.type
```

그다음 reduce가 일어나면서 타입 정보가 위로 올라온다.  
올라오는 과정에서 각 `id`에 타입을 붙인다.

```
L → , id L1
addtype(id.entry, L1.type)

D → id L
addtype(id.entry, L.type)
```

즉 `T.type`을 아래로 내려보내는 게 아니라, `T.type`이 만들어진 뒤 reduce 과정에서 위로 올라오게 만든 것이다.

---
# Replacing Inherited Attributes with Syntheiszed Lists
앞에서는 **문법 자체를 바꿔서 inherited attribute를 피하는 방법**이었다면,  

이번에는 **문법은 거의 유지하되, inherited attribute 대신 list를 synthesized attribute로 만들어서 처리하는 방법**이다.

![](../images/Pasted%20image%2020260602215315.png)

원래 선언문은 이런 형태라고 보면 된다.

```
real id1, id2, id3
```

```
D → T L  
T → int  
T → real  
L → L1 , id  
L → id
```

여기서 해야 하는 일은 아래와 같다.

```
id1의 type = real
id2의 type = real
id3의 type = real
```

그런데 `T.type`은 `T → real`에서 만들어지고, 이 타입을 `L` 안의 여러 `id`들에게 전달해야 한다.

여기서 `L.type`은 위에서 아래로 전달되는 값이다. 즉 inherited attribute다.

흐름은 이렇게 된다.

```
T.type = real
↓
L.type = real
↓
L1.type = real
↓
각 id에 addtype 적용
```

그래서 그림에서도 `T`에서 나온 type 정보가 `L` 쪽으로 내려가는 화살표로 표현되어 있다.

---

두 번째 슬라이드는 발상을 바꾼다. 기존 방식은 **타입을 id들에게 내려보내자** 였다.
그런데 새 방식은 다음과 같다.

>id들을 먼저 list로 모아 올리고,나중에 type을 한 번에 적용하자

그래서 `L`이 더 이상 `type`을 inherited attribute로 받지 않는다.  
대신 `L`은 자기 안에 있는 id들을 모아서 `L.list`라는 synthesized attribute로 만든다.

```
L → id
L.list = [id]

L → L1 , id
L.list = L1.list + [id]
```

예를 들어 `id1`, `id2`, `id3이면` reduce 과정에서

```
L.list = [id1]
L.list = [id1, id2]
L.list = [id1, id2, id3]
```

이렇게 id 목록이 위로 올라간다.

![](../images/Pasted%20image%2020260602215645.png)

이제 `D → T L`을 reduce할 때는 두 값이 모두 준비되어 있다.

```
T.type = real
L.list = [id1, id2, id3]
```

그래서 이때 한 번에 처리한다.

```
D → T L {
    for all id ∈ L.list:
        addtype(id.entry, T.type)
}
```

즉 `L` 내부로 type을 미리 내려보내지 않는다.  
대신 `L`이 id 목록을 위로 올려주고, `D`에서 `T.type`과 `L.list`를 함께 사용한다.