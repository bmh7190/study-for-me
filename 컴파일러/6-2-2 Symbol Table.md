## Names and Scopes

앞에서 syntax-directed translation을 이용해서 three-address code를 생성하는 방법을 배웠다. 그런데 three-address code를 만들 때 `id`가 나오면, 컴파일러는 그 `id`가 단순히 어떤 문자열인지 보는 것에서 끝나면 안 된다.

예를 들어 다음과 같은 코드가 있다고 하자.

```c
a = b + c;
```

이 코드는 중간 코드로 만들면 대략 다음처럼 표현될 수 있다.

```text
t1 = b + c
a = t1
```

하지만 여기서 `a`, `b`, `c`가 정확히 무엇인지를 알아야 한다. `b`와 `c`가 선언된 변수인지, 두 값을 더할 수 있는 타입인지, 그리고 그 결과를 `a`에 대입할 수 있는지 확인해야 한다. 즉, 중간 코드 생성은 단순히 문법 구조만 보고 하는 것이 아니라, 이름이 의미하는 선언 정보까지 같이 확인하면서 진행된다.

그래서 필요한 것이 **name resolving**이다. name resolving은 프로그램 안에서 사용된 이름이 실제로 어떤 선언을 가리키는지 찾는 과정이다.

예를 들어 전역 변수와 지역 변수에 같은 이름이 있을 수 있다.

```c
int x;

void f() {
    int x;
    x = 10;
}
```

여기서 함수 안의 `x = 10;`에서 사용된 `x`는 전역 변수 `x`가 아니라 함수 내부에 선언된 지역 변수 `x`이다. 이름은 같지만, 스코프가 다르기 때문에 의미하는 대상이 달라진다. 따라서 컴파일러는 현재 위치에서 어떤 선언이 유효한지를 기준으로 이름을 해석해야 한다.

이 과정에서 **type checking**도 같이 필요하다. 이름을 찾으면 그 이름의 타입도 알 수 있기 때문이다.

예를 들어 `b + c`를 처리할 때 `b`가 `int`이고 `c`도 `int`라면 덧셈이 가능하다. 하지만 `b`가 배열이거나 함수 이름이거나, 덧셈이 불가능한 타입이라면 오류가 된다. 또한 `b + c`의 결과 타입이 `a`에 대입 가능한지도 확인해야 한다.

그래서 symbol table에는 보통 다음과 같은 정보가 저장된다.

- 이름
- 타입
- 선언된 위치
- 스코프
- 메모리 위치 또는 offset

이 정보가 있어야 parser나 semantic analyzer가 `id`를 만났을 때 그 이름이 올바른지 판단할 수 있다.

또 하나 중요한 이유는 backend 때문이다. 중간 코드에서는 `a`, `b`, `c`처럼 이름을 그대로 사용할 수 있지만, 실제 기계어로 갈 때는 이름이 그대로 남아 있지 않는다. 변수 이름은 결국 메모리 주소나 activation record 안의 offset 같은 실제 저장 위치로 바뀌어야 한다.

예를 들어 지역 변수라면 이런 식으로 바뀔 수 있다.

```text
a → fp - 4
b → fp - 8
c → fp - 12
```

즉, frontend에서는 이름과 타입을 확인하기 위해 symbol table이 필요하고, backend에서는 이름을 실제 메모리 위치로 바꾸기 위해 symbol table이 필요하다. 선언을 만날 때마다 그 정보를 symbol table에 저장해 두고, 나중에 그 이름이 사용될 때 찾아서 의미를 확인한다.

그리고 스코프가 있기 때문에 symbol table도 하나만 있으면 부족하다.

전역에 선언된 변수나 함수는 **global symbol table**에 저장된다.

```c
int g;
void func();
```

반면 함수 내부, 블록 내부, 구조체 내부에서 선언된 이름들은 **local symbol table**에 저장된다.

```c
void f() {
    int x;

    {
        int y;
    }
}
```

여기서 `x`는 함수 `f`의 local symbol table에 들어가고, `y`는 더 안쪽 블록의 local symbol table에 들어간다. 이렇게 해야 같은 이름이 여러 스코프에 존재해도 현재 위치에서 어떤 선언을 사용해야 하는지 결정할 수 있다.

---
# Symbol Tables for Scoping

앞에서 말한 **global symbol table**과 **local symbol table**이 실제 코드에서 어떻게 나뉘어 필요한지를 보여주는 예시다.

![](../images/Pasted%20image%2020260602235025.png)

핵심은 하나의 프로그램 안에도 이름이 선언되는 위치가 여러 종류라는 것이다. 전역 변수, 함수 이름, 함수의 parameter, 함수 내부 지역 변수, struct의 field가 모두 같은 방식으로 관리되지 않는다. 그래서 스코프마다 symbol table이 필요하다.

---
### 1. struct S의 field를 위한 symbol table

왼쪽 위에 이런 코드가 있다.

```c
struct S
{
    int a;
    int b;
} s;
```

여기서 `struct S`는 구조체 타입이고, 그 안에는 field `a`, `b`가 있다.

이때 `a`와 `b`는 전역 변수 `a`, `b`가 아니다. `struct S` 안에서만 의미를 갖는 field 이름이다. 따라서 구조체 내부에도 별도의 symbol table이 필요하다.

대략 이런 식으로 관리된다.

```text
struct S의 field table
a → int
b → int
```

그래야 나중에 `s.a`, `s.b` 같은 표현이 나왔을 때, 컴파일러가 `s`의 타입이 `struct S`인지 확인하고, 그 안에 `a`, `b`라는 field가 실제로 존재하는지 검사할 수 있다.

즉, `a`라는 이름은 단독으로 쓰인 변수 이름일 수도 있고, `s.a`처럼 구조체 field 이름일 수도 있다. 그래서 구조체 field는 일반 변수와 구분해서 관리해야 한다.

---
### 2. global variables and functions를 위한 symbol table

코드에는 전역 위치에 다음 선언들이 있다.

```c
struct S { ... } s;

void swap(int& a, int& b) { ... }

void somefunc() { ... }
```

여기서 `s`, `swap`, `somefunc`는 전역 스코프에 선언된 이름이다. 그래서 이들은 global symbol table에 저장된다.

```text
global symbol table
S        → struct type
s        → variable, type struct S
swap     → function
somefunc → function
```

이 정보가 있어야 `somefunc` 안에서 `swap(s.a, s.b);`를 만났을 때, 컴파일러가 `swap`이라는 함수가 실제로 선언되어 있는지 찾을 수 있다.

또 `s`도 global symbol table에서 찾아야 한다. `somefunc` 안에는 `s`가 지역 변수로 선언되어 있지 않기 때문에, local symbol table에서 못 찾으면 바깥 스코프인 global symbol table을 확인한다.

---
### 3. 함수마다 arguments와 locals를 위한 symbol table

다음으로 `swap` 함수 내부를 보자.

```c
void swap(int& a, int& b)
{
    int t;
    t = a;
    a = b;
    b = t;
}
```

여기서 `a`, `b`는 함수의 parameter이고, `t`는 함수 내부 지역 변수다. 이 이름들은 `swap` 함수 안에서만 의미가 있다.

따라서 `swap` 함수에는 별도의 local symbol table이 필요하다.

```text
swap의 local symbol table
a → int&, parameter
b → int&, parameter
t → int, local variable
```

이 symbol table이 있어야 함수 내부의 문장을 해석할 수 있다.

```c
t = a;
a = b;
b = t;
```

여기서 `a`, `b`, `t`가 각각 어떤 타입인지, parameter인지 local variable인지 알 수 있다.

특히 중요한 점은 `swap`의 parameter 이름 `a`, `b`와 `struct S`의 field 이름 `a`, `b`가 같다는 것이다. 하지만 이 둘은 완전히 다른 이름이다. 이걸 구분하려면 스코프별 symbol table이 반드시 필요하다.

---
### 4. somefunc에서 `swap(s.a, s.b)`를 검사하는 과정

마지막 부분이 핵심 예시다.

```c
void somefunc()
{
    ...
    swap(s.a, s.b);
    ...
}
```

컴파일러가 이 문장을 만나면 여러 symbol table을 사용해서 차례로 확인한다.

먼저 `swap`을 찾는다.

1. somefunc의 local symbol table에서 swap 찾기 → 없음
2. global symbol table에서 swap 찾기 → 있음, 함수

그다음 `s.a`를 확인한다.

- somefunc의 local symbol table에서 s 찾기 → 없음
- global symbol table에서 s 찾기 → 있음, type은 struct S 
- struct S의 field table에서 a 찾기 → 있음, int

`s.b`도 같은 방식이다.

- global symbol table에서 s 찾기 → struct S

이 과정을 통해 컴파일러는 다음을 알 수 있다.

```text
swap은 전역 함수다.
s는 전역 변수다.
s의 타입은 struct S다.
struct S에는 field a와 b가 있다.
s.a와 s.b는 접근 가능한 field다.
```

그래서 symbol table을 사용하면 `s`와 그 field에 접근하는 코드를 생성할 수 있다.

---
# Offset and Width for Runtime Allocation

앞에서 말한 symbol table 정보가 **실제 실행 시간 메모리 배치**와 어떻게 연결되는지를 보자.

앞에서는 symbol table이 이름, 타입, 스코프를 저장한다고 했다. 그런데 컴파일러가 실제 코드를 생성하려면 한 가지 정보가 더 필요하다.

```text
이 변수가 메모리에서 몇 번째 위치에 저장되는가?
```

이때 사용하는 개념이 **offset**과 **width**다.

---
### Offset은 시작 위치로부터의 거리다

예를 들어 구조체 `S`가 있다.

```c
struct S
{
    int a;
    int b;
} s;
```

`int`의 크기를 4바이트라고 하면, 구조체 안에서 `a`와 `b`는 다음처럼 배치된다.

```text
struct S
offset 0 : a
offset 4 : b
```

여기서 `a`의 offset은 0이고, `b`의 offset은 4다. 즉, offset은 **어떤 영역의 시작 주소로부터 얼마나 떨어져 있는지**를 의미한다.

만약 변수 `s`의 시작 주소가 `1000`이라고 하면, 아래와 같이  접근할 수 있다.

```text
s.a → 1000 + 0
s.b → 1000 + 4
```


---
### Width는 전체 크기다

`struct S`에는 `int a`, `int b`가 있다. `int` 하나가 4바이트라면 전체 크기는 다음과 같다.

```text
a: 4 bytes
b: 4 bytes
전체 width = 8 bytes
```

여기서 width는 해당 타입이나 메모리 영역이 차지하는 전체 크기다.

```text
int의 width = 4
struct S의 width = 8
```

이 정보가 있어야 컴파일러가 다음 field나 다음 변수를 어디에 배치할지 결정할 수 있다.

---
### 함수의 local symbol table도 offset을 가진다

이번에는 `swap` 함수를 보자.

```c
void swap(int& a, int& b)
{
    int t;
    t = a;
    a = b;
    b = t;
}
```

이 함수가 실행되면, `swap` 함수만의 실행 공간이 필요하다. 이 공간을 보통 **subroutine frame** 또는 **activation record**라고 한다.

여기에는 함수의 parameter와 local variable이 들어간다.

```text
parameter a
parameter b
local variable t
```

슬라이드에서는 이들이 frame 안에 다음 offset으로 저장된다고 설명한다.

```text
fp[0] : a
fp[4] : b
fp[8] : t
```

즉, `swap` 함수의 frame pointer를 기준으로,

```text
a → fp + 0
b → fp + 4
t → fp + 8
```

이렇게 접근할 수 있다.

여기서 `fp`는 frame pointer이고, 현재 함수의 실행 공간 시작 위치를 가리킨다고 보면 된다.

---
### Frame의 width는 12다

`swap` 함수의 frame에는 `a`, `b`, `t`가 들어간다. 각각 4바이트라고 하면,

```text
a: 4 bytes
b: 4 bytes
t: 4 bytes
```

따라서 전체 frame의 크기는 12바이트다.

```text
frame width = 4 + 4 + 4 = 12
```

이 정보는 런타임에 함수 호출이 일어났을 때, stack에 얼마만큼의 공간을 확보해야 하는지 결정하는 데 사용된다.

---
### Symbol table과 offset의 관계

여기서 중요한 건 symbol table이 단순히 이름과 타입만 저장하는 게 아니라는 점이다.

컴파일러는 symbol table에 이런 정보도 같이 저장한다.

```text
name: a
type: int
offset: 0
```

```text
name: b
type: int
offset: 4
```

```text
name: t
type: int
offset: 8
```

그래야 `t = a;` 같은 코드를 만났을 때 실제로는 이런 식의 접근 코드를 만들 수 있다.

```text
t = a
→ fp[8] = fp[0]
```

물론 실제 기계어는 더 복잡하지만, 핵심은 이름을 offset 기반 접근으로 바꾼다는 것이다.

---
### `swap(s.a, s.b)`와 offset

마지막으로 `somefunc` 안의 호출을 보자.

```c
swap(s.a, s.b);
```

여기서 컴파일러는 먼저 `s`를 global symbol table에서 찾는다.

```text
s → type struct S
```

그다음 `struct S`의 field table을 확인한다.

```text
a → offset 0
b → offset 4
```

그래서 `s.a`, `s.b`는 다음처럼 계산될 수 있다.

```text
s.a → s의 시작 주소 + 0
s.b → s의 시작 주소 + 4
```

그리고 `swap`은 `int& a`, `int& b`처럼 reference parameter를 받으므로, 실제로는 `s.a`와 `s.b`의 값 자체가 아니라 그 위치, 즉 주소가 전달된다.

즉, symbol table의 offset 정보가 있어야 `s.a`, `s.b`에 접근하는 코드를 만들 수 있다.

---
# Symbol Tables for Scoping
앞에서 설명한 내용을 실제 자료구조 형태로 보자.

![](../images/Pasted%20image%2020260602235650.png)

---
### Global symbol table

오른쪽 위의 `globals`가 전역 symbol table이다.

코드에서 전역 스코프에 있는 이름은 다음과 같다.

```c
struct S { int a; int b; } s;

void swap(int& a, int& b) { ... }

void foo() { ... }
```

그래서 global table에는 이런 이름들이 들어간다.

```text
s     → 전역 변수
swap  → 함수
foo   → 함수
```

여기서 `s`는 단순한 이름만 저장되는 것이 아니라, `s`의 타입 정보로 연결된다.

```text
s → Trec S
```

즉, `s`는 `struct S` 타입의 변수라는 뜻이다.

그리고 `swap`, `foo`도 각각 함수 타입 정보로 연결된다.

```text
swap → Tfun swap
foo  → Tfun foo
```

---
### struct S의 field table

`Trec S`는 `struct S` 타입을 나타내는 type node다.  
그런데 struct 타입은 내부에 field를 가진다.

```c
struct S {
    int a;
    int b;
};
```

그래서 `Trec S`는 별도의 symbol table을 가리킨다. 이 table 안에는 field `a`, `b`가 들어 있다.

```text
struct S의 field table
a → offset 0, type int
b → offset 4, type int
```

여기서 `a(0)`, `b(4)`라고 적힌 것은 각각 field의 offset이다.

즉, `s.a`를 접근하려면 `s`의 시작 주소에서 0만큼 떨어진 위치를 보면 되고, `s.b`는 4만큼 떨어진 위치를 보면 된다.

- s.a → base address of s + 0
- s.b → base address of s + 4

그리고 field table의 전체 width는 8이다.  `int a`가 4바이트, `int b`가 4바이트이기 때문이다.

---
### swap 함수의 local symbol table

`swap` 함수는 다음과 같다.

```c
void swap(int& a, int& b)
{
    int t;
    t = a;
    a = b;
    b = t;
}
```

`swap` 함수 안에는 parameter `a`, `b`와 local variable `t`가 있다.  
그래서 `swap` 함수도 자기만의 local symbol table을 가진다.

```text
swap의 local table
a → offset 0, type ref int
b → offset 4, type ref int
t → offset 8, type int
```

여기서 `a`, `b`는 `int&`이므로 그냥 `int`가 아니라 `Tref → Tint` 형태로 표현된다.  
즉, `a`와 `b`는 int 값을 직접 들고 있는 변수가 아니라, int를 가리키는 reference parameter다.

반면 `t`는 함수 안에서 선언된 일반 지역 변수이므로 `Tint`로 연결된다.

그리고 이 local table의 width는 12이다. 이 width는 나중에 `swap` 함수가 호출될 때 activation record, 즉 함수 실행 공간을 얼마나 잡아야 하는지 계산할 때 사용된다.

---
### foo 함수의 local symbol table

`foo` 함수는 다음과 같다.

```c
void foo()
{
    ...
    swap(s.a, s.b);
    ...
}
```

이 예시에서는 `foo` 안에 지역 변수가 따로 없기 때문에, `foo`의 local table은 거의 비어 있다.

그래서 그림에서 `Tfun foo`가 가리키는 table에는 `prev [0]` 정도만 보인다.  
여기서 `[0]`은 이 local table에서 필요한 local storage의 width가 0이라는 뜻으로 보면 된다.

---
### prev 포인터의 의미

각 symbol table에는 `prev`가 있다. `prev`는 바깥 스코프의 symbol table을 가리킨다.

전역 symbol table은 가장 바깥 스코프이므로 `prev = nil`이다.

반면 `swap` 함수의 local table은 바깥쪽 전역 스코프 안에 있으므로, `prev`가 global table을 가리킨다.

- swap local table → prev → globals
- foo local table  → prev → globals

이 구조 덕분에 이름을 찾을 때 현재 스코프부터 먼저 찾고, 없으면 바깥 스코프로 올라갈 수 있다.

예를 들어 `foo` 안에서 `swap(s.a, s.b)`를 만나면,

- foo local table에서 swap 찾기 → 없음

- prev를 따라 global table로 이동 → swap 있음

이런 식으로 찾는다.

`s`도 마찬가지다.

- foo local table에서 s 찾기 → 없음

- global table에서 s 찾기 → 있음, type은 struct S

그다음 `s.a`를 처리하려면 `s`의 타입인 `Trec S`로 이동하고, 그 안의 field table에서 `a`를 찾는다.

```text
s → Trec S
Trec S의 field table에서 a 찾기
→ offset 0, type int
```

`s.b`도 같은 방식이다.

```text
s → Trec S
Trec S의 field table에서 b 찾기
→ offset 4, type int
```

---
### Table node와 Type node

오른쪽 아래에 보면 두 종류의 박스가 나온다.

```text
Table nodes
type nodes
```

이 그림에서 symbol table 자체는 **table node**이고, 타입을 나타내는 것은 **type node**다.

예를 들어 global table의 `s` 항목은 `Trec S`라는 type node를 가리킨다.

```text
s → Trec S
```

`Trec S`는 struct 타입이고, 다시 field table을 가진다.

```text
Trec S → field table { a, b }
```

`swap`은 함수 타입이다.

```text
swap → Tfun swap
```

`Tfun swap`은 함수의 parameter와 local variable을 저장하는 table을 가리킨다.

```text
Tfun swap → swap local table
```

즉, symbol table과 type node가 서로 연결되면서 이름, 타입, field, parameter, local variable 정보를 모두 관리하는 구조가 된다.

---
# Hierachical Symbol Table Operation

여기는 앞에서 봤던 **스코프별 symbol table 구조를 실제로 어떻게 만들고 사용하는지**를 정리한 부분이다. 핵심은 symbol table이 하나만 있는 게 아니라, 전역 → 함수 → 블록처럼 **계층적으로 연결된다**는 것이다.

### mktable(previous)

새로운 symbol table을 만든다.

```text
mktable(previous)
```

여기서 `previous`는 바깥 스코프의 symbol table을 의미한다.

예를 들어 함수 안으로 들어가면 함수용 local symbol table을 새로 만든다. 그리고 그 table의 `prev`가 global table을 가리키게 한다.

```text
function local table → prev → global table
```

---
### enter(table, name, type, offset)

현재 symbol table에 새 이름을 등록한다.

```text
enter(table, name, type, offset)
```

예를 들어 다음 선언을 만나면,

```c
int t;
```

현재 함수의 local table에 `t`를 넣는다.

```text
t → type int, offset 8
```

즉, 선언된 변수나 field를 symbol table에 추가하는 연산이다.

---
### addwidth(table, width)

symbol table 안에 들어간 항목들의 전체 크기를 저장한다.

```text
addwidth(table, width)
```

예를 들어 `struct S`에 `int a`, `int b`가 있으면 각각 4바이트이므로 전체 width는 8이다.

```text
struct S width = 8
```

함수 frame도 마찬가지로 parameter와 local variable의 크기를 합쳐서 width를 계산한다.

---
### enterproc(table, name, newtable)

procedure나 function 이름을 현재 table에 등록하고, 그 함수의 local symbol table과 연결한다.

```text
enterproc(table, name, newtable)
```

예를 들어 `swap` 함수가 있으면 global table에는 `swap`이라는 함수 이름이 들어가고, 그 항목은 `swap` 함수의 local table을 가리킨다.

---
### lookup(table, name)

이름을 찾는 연산이다.

```text
lookup(table, name)
```

먼저 현재 table에서 찾고, 없으면 `prev`를 따라 바깥 table로 이동한다.

예를 들어 `foo` 안에서 `s`를 찾는다면,

- foo local table에서 s 검색 → 없음
- prev를 따라 global table 검색 → s 발견

이렇게 현재 스코프부터 바깥 스코프 순서로 이름을 찾는다.

---
# Syntax-Directed Translation of Delcarations in Scope

이 부분은 **선언문을 처리하면서 symbol table을 어떻게 만들고 관리하는지**를 SDD/SDT 형태로 보여주는 내용이다.

앞에서는 symbol table 연산을 개별적으로 봤다.

```text
mktable
enter
addwidth
enterproc
lookup
```

이번에는 그 연산들이 실제 문법 규칙 안에서 언제 실행되는지를 보여준다.

---

![](../images/Pasted%20image%2020260603000518.png)

왼쪽 문법을 보면 프로그램은 대략 이렇게 구성된다.

```text
P → D ; S
```

즉, 프로그램 `P`는 선언부 `D`와 실행문 `S`로 이루어진다.

```text
D → D ; D
  | id : T
  | proc id ; D ; S
```

`D`는 선언문이다.

`id : T`는 변수 선언이고, `x : integer`같은 형태라고 보면 된다.

`proc id ; D ; S`는 procedure 선언이다. procedure 안에도 다시 선언부 `D`와 실행문 `S`가 있다. 이 말은 procedure 내부에 새로운 스코프가 생긴다는 뜻이다.

---
### T는 타입을 만든다

```text
T → integer
  | real
  | array [ num ] of T
  | ^ T
  | record D end
```

`T`는 타입을 의미한다.

예를 들어 아래와 같은  타입들을 만들 수 있다.

- integer
- real
- array [10] of integer
- ^ integer
- record ... end

여기서 `T.type`과 `T.width`가 중요하다.

예를 들어 `integer`의 width가 4라면, T.type에는  integer type 이 들어가고 T.width는 4가 된다.

배열 타입이면 원소 타입의 width와 개수를 이용해서 전체 width를 계산할 수 있다.

```text
array [10] of integer
→ width = 10 * 4 = 40
```

---
### E는 표현식 처리를 위한 속성이다

오른쪽 문법에는 표현식 `E`도 나온다.

```text
E → E + E
  | E * E
  | - E
  | ( E )
  | id
  | E ^
  | & E
  | E . id
```

여기서 `E.addr`는 표현식 결과가 저장된 위치를 의미한다.

```text
E.addr → E의 값을 담고 있는 임시 변수 이름
```

예를 들어 `a + b`를 처리하면 중간 코드에서 임시 변수가 만들어질 수 있다.

```text
t1 = a + b
```

이때 `E.addr = t1`이 된다.

---
### tblptr와 offset

오른쪽에 전역 데이터가 두 개 나온다.

- tblptr
- offset

둘 다 stack이다.

### tblptr

`tblptr`는 현재 사용 중인 symbol table들을 저장하는 stack이다.

```text
tblptr = symbol table pointer stack
```

새로운 스코프에 들어가면 새 symbol table을 만들고 push한다.  
스코프가 끝나면 pop한다.

### offset

`offset`은 현재 스코프에서 변수들이 어느 위치에 배치되는지를 관리하는 stack이다.

```text
offset = offset value stack
```

새로운 스코프에 들어가면 offset을 0부터 시작한다.  
변수를 하나 선언할 때마다 그 변수의 width만큼 offset을 증가시킨다.

---

![](../images/Pasted%20image%2020260603000824.png)

### P → D ; S

```text
P →
{
  t := mktable(nil);
  push(t, tblptr);
  push(0, offset)
}
D ; S
```

프로그램이 시작되면 가장 먼저 global symbol table을 만든다.

```text
t := mktable(nil)
```

여기서 `nil`은 바깥 스코프가 없다는 뜻이다. 즉, global table의 `prev`는 없다.

그다음 이 table을 `tblptr`에 push한다.

```text
push(t, tblptr)
```

그리고 global scope의 offset을 0으로 시작한다.

```text
push(0, offset)
```

즉, 프로그램 시작 시에는 다음 상태가 된다.

```text
tblptr top → global symbol table
offset top → 0
```

---
### D → id : T

다음은 변수 선언을 처리하는 규칙이다.

```text
D → id : T
{
  enter(top(tblptr), id.name, T.type, top(offset));
  top(offset) := top(offset) + T.width
}
```

예를 들어 다음 선언을 보자.

```text
x : integer
```

컴파일러는 현재 symbol table에 `x`를 등록한다.

```text
enter(current table, x, integer type, current offset)
```

처음 offset이 0이면, `x → type integer, offset 0` 이 된다.

그다음 offset을 `T.width`만큼 증가시킨다.

```text
top(offset) := top(offset) + T.width
```

만약 integer의 width가 4라면, `offset = 0 + 4 = 4` 로 다음 변수가 선언되면 offset 4에 배치된다.

예를 들어

```text
x : integer;
y : real;
```

이면 대략 이렇게 된다.

```text
x → offset 0
y → offset 4
```

그리고 `y`의 width만큼 offset이 또 증가한다.

---
### D → proc id ; D1 ; S

가장 중요한 부분은 procedure 선언이다.

```text
D → proc id ;
{
  t := mktable(top(tblptr));
  push(t, tblptr);
  push(0, offset)
}
D1 ; S
{
  t := top(tblptr);
  addwidth(t, top(offset));
  pop(tblptr);
  pop(offset);
  enterproc(top(tblptr), id.name, t)
}
```

procedure가 나오면 새로운 스코프가 생긴다. 그래서 procedure 내부를 위한 새 symbol table을 만든다.

```text
t := mktable(top(tblptr))
```

여기서 `top(tblptr)`는 현재 바깥 스코프의 symbol table이다.  
즉, 새 table의 `prev`가 바깥 table을 가리키게 된다.

```text
procedure local table → prev → outer table
```

그다음 새 table을 현재 table로 사용하기 위해 push한다.

```text
push(t, tblptr)
```

그리고 procedure 내부 offset은 0부터 시작한다.

```text
push(0, offset)
```

이제 `D1 ; S`를 처리하면서 procedure 내부의 변수 선언과 문장을 처리한다.

---
### procedure 처리가 끝날 때

procedure 내부 선언과 문장을 다 처리한 뒤에는 이 semantic action이 실행된다.

```text
t := top(tblptr);
addwidth(t, top(offset));
pop(tblptr);
pop(offset);
enterproc(top(tblptr), id.name, t)
```

하나씩 보면 이렇다.

먼저 현재 table, 즉 procedure local table을 가져온다.

```text
t := top(tblptr)
```

그다음 현재 offset 값을 그 table의 width로 저장한다.

```text
addwidth(t, top(offset))
```

이 말은 procedure의 activation record가 얼마나 필요한지 저장한다는 뜻이다.

예를 들어 procedure 안에 지역 변수가 3개 있고 각각 4바이트라면,

```text
width = 12
```

가 된다.

그다음 procedure 스코프가 끝났으므로 현재 table과 offset을 pop한다.

```text
pop(tblptr)
pop(offset)
```

이제 다시 바깥 스코프의 table이 현재 table이 된다.

마지막으로 바깥 table에 procedure 이름을 등록한다.

```text
enterproc(top(tblptr), id.name, t)
```

즉, procedure 이름은 바깥 스코프에 등록되고, 그 이름은 procedure의 local symbol table을 가리킨다.

예를 들어

```text
proc swap;
  a : integer;
  b : integer;
  ...
```

라면 바깥 table에는 이렇게 들어간다.

```text
swap → procedure, local table pointer
```

그리고 `swap`의 local table에는 `a`, `b` 같은 내부 선언이 들어 있다.

---
### D → D1 ; D2

마지막 규칙은 선언이 여러 개 이어지는 경우다.

```text
D → D1 ; D2
```

이건 선언문을 순서대로 처리하겠다는 의미다.

예를 들어

```text
x : integer;
y : real;
```

이면 먼저 `D1`에서 `x`를 현재 table에 넣고 offset을 증가시킨다.  
그다음 `D2`에서 `y`를 현재 table에 넣고 offset을 또 증가시킨다.

---
## Type 선언에 대한 SDT

여기서는 앞에서 봤던 선언 처리 중에서, 특히 **타입 `T`를 만나면 `T.type`과 `T.width`를 어떻게 계산하는지**를 보여준다.

![](../images/Pasted%20image%2020260603001231.png)

여기서 핵심 attribute는 두 개다.

```text
T.type  → 타입 정보
T.width → 그 타입이 차지하는 byte 크기
```

---

### 기본 타입: integer, real

```text
T → integer
{
  T.type := 'integer';
  T.width := 4
}
```

`integer` 타입을 만나면 타입은 integer이고, 크기는 4바이트로 저장한다.

```text
integer → type integer, width 4
```

다음은 `real`이다.

```text
T → real
{
  T.type := 'real';
  T.width := 8
}
```

`real`은 8바이트라고 가정한다. 즉, 기본 타입은 바로 type과 width를 정할 수 있다.

---
### 배열 타입

```text
T → array [ num ] of T1
{
  T.type := array(num.val, T1.type);
  T.width := num.val * T1.width
}
```

배열 타입은 원소 타입 `T1`을 먼저 알아야 한다.

예를 들어 `array [10] of integer` 이면 `T1`은 `integer`다.

```text
T1.type = integer
T1.width = 4
```

배열 크기 `num.val`이 10이므로,

```text
T.type = array(10, integer)
T.width = 10 * 4 = 40
```

즉, 배열의 width는 `배열 길이 × 원소 타입의 width`이다.

배열 안에 배열이 들어가도 같은 방식으로 계산된다.

```text
array [2] of array [3] of integer
```

이면 안쪽부터 계산해서,

```text
array [3] of integer → width 12
array [2] of array[3] of integer → width 2 * 12 = 24
```

가 된다.

---
### 포인터 타입

```text
T → ^ T1
{
  T.type := pointer(T1.type);
  T.width := 4
}
```

`^T1`은 `T1`을 가리키는 포인터 타입이다.

예를 들어 `^ integer` 이면,

```text
T.type = pointer(integer)
T.width = 4
```

여기서 중요한 점은 포인터가 가리키는 대상 타입이 무엇이든, 포인터 자체의 크기는 `4바이트`로 둔다는 것이다.

```text
^integer → width 4
^real    → width 4
^array[...] → width 4
```

왜냐하면 포인터 변수에는 실제 값 전체가 들어가는 것이 아니라, 그 값을 가리키는 주소가 들어가기 때문이다.

---
### record 타입

가장 중요한 부분은 record다.

```text
T → record
{
  t := mktable(nil);
  push(t, tblptr);
  push(0, offset)
}
D end
{
  T.type := record(top(tblptr));
  T.width := top(offset);
  addwidth(top(tblptr), top(offset));
  pop(tblptr);
  pop(offset)
}
```

record는 C의 struct와 비슷하다고 보면 된다.

```text
record
  a : integer;
  b : real
end
```

record 안에는 field 선언들이 들어간다.  
그래서 record를 만나면 record 내부 field를 저장할 **새 symbol table**이 필요하다.

---
### record 시작 시점

record를 만나자마자 다음 action을 실행한다.

```text
t := mktable(nil);
push(t, tblptr);
push(0, offset)
```

이 의미는 다음과 같다.

- record field를 위한 새 symbol table 생성
- 그 table을 현재 table로 push
- record 내부 offset을 0부터 시작

즉, record 안의 field들은 기존 global/local table에 바로 들어가는 것이 아니라, record 전용 field table에 들어간다.

예를 들어

```text
record
  a : integer;
  b : real
end
```

이면 `a`, `b`는 record field table에 저장된다.

---
### record 내부 선언 D 처리

record 안의 `D`를 처리하면서 field들이 등록된다.

처리 방식은 앞에서 본 변수 선언과 같다.

```text
D → id : T
{
  enter(top(tblptr), id.name, T.type, top(offset));
  top(offset) := top(offset) + T.width
}
```

예를 들어

```text
a : integer;
b : real;
```

이면 처음 offset은 0이다.

```text
a → type integer, offset 0
offset = 0 + 4 = 4
```

그다음 `b`는 offset 4에 들어간다.

```text
b → type real, offset 4
offset = 4 + 8 = 12
```

따라서 record 전체 width는 12가 된다.

---
### record가 끝났을 때

`D end`까지 처리한 뒤에는 record 타입을 완성한다.

```text
T.type := record(top(tblptr));
T.width := top(offset);
addwidth(top(tblptr), top(offset));
pop(tblptr);
pop(offset)
```

먼저 현재 symbol table을 이용해서 record 타입을 만든다.

```text
T.type := record(top(tblptr))
```

즉, 이 record 타입은 자신의 field table을 가리키게 된다.

그다음 현재 offset을 record 전체 width로 사용한다.

```text
T.width := top(offset)
```

예를 들어 field들이 총 12바이트라면,

```text
T.width = 12
```

그리고 field table에도 전체 width를 저장한다.

```text
addwidth(top(tblptr), top(offset))
```

마지막으로 record 스코프가 끝났으므로 table과 offset을 pop한다.

```text
pop(tblptr)
pop(offset)
```

이제 다시 바깥 스코프의 symbol table로 돌아간다.

---
# Example

![](../images/Pasted%20image%2020260603001521.png)

---
# Syntax - Directed Translation of Statements in Scope

앞에서는 선언을 처리하면서 symbol table에 이름, 타입, offset, width를 저장했다.  
이제 **저장해 둔 symbol table 정보를 실제 문장과 표현식 번역에 어떻게 사용하는지**를 보여준다.

![](../images/Pasted%20image%2020260603001657.png)

---
## Statements in Scope

### S → S ; S

```text
S → S ; S
```

이건 문장이 여러 개 이어질 수 있다는 뜻이다.

예를 들어

```c
a = b;
c = d;
```

같은 구조를 처리하기 위한 규칙이다.

---
### S → id := E

핵심은 이 규칙이다.

```text
S → id := E
```

즉, 대입문이다.

예를 들어 `x := a + b` 를 처리한다고 생각하면 된다.

이때 오른쪽 표현식 `E`는 이미 계산되어서 결과 위치가 `E.addr`에 들어 있다고 본다.

예를 들어 `a + b`를 계산하면서 이런 중간 코드가 나왔다면,

```text
t1 := a + b
```

`E.addr`는 `t1`이 된다. 이제 해야 할 일은 `id`에 `E.addr` 값을 저장하는 것이다.

---
### lookup으로 id 찾기

먼저 symbol table에서 왼쪽 변수 이름을 찾는다.

```text
p := lookup(top(tblptr), id.name)
```

여기서 `top(tblptr)`는 현재 스코프의 symbol table이다. 즉, 현재 함수나 블록의 symbol table에서 먼저 찾고, 없으면 `prev`를 따라 global table까지 올라가면서 찾는다.

만약 못 찾으면, 선언되지 않은 변수를 사용한 것이므로 error를 낸다.

---
### 전역 변수와 지역 변수의 차이

찾은 entry `p`에는 그 이름이 전역 변수인지 지역 변수인지 알 수 있는 정보가 있다.

```text
if p.level = 0 then // global variable
```

`level = 0`이면 전역 변수라고 본다.

전역 변수는 이름 자체 또는 전역 주소로 접근할 수 있으므로, `emit(id.addr ':=' E.addr)` 처럼 코드를 만든다.

예를 들어 전역 변수 `x`에 대입하면, `x := t1` 같은 코드가 생성된다.

반면 지역 변수는 함수의 activation record 안에 있다.  
그래서 이름 자체가 아니라 frame pointer 기준 offset으로 접근해야 한다.

```text
emit(fp[p.offset] ':=' E.addr)
```

예를 들어 `t`가 현재 함수 frame의 offset 8에 있다면,

```text
fp[8] := E.addr
```

이렇게 번역된다.

---
# Syntax-Directed Translation of Expressions in Scope

이번에는 표현식 `E`를 번역하는 규칙이다.

표현식의 핵심 attribute는 이것이다.

```text
E.addr
```

`E.addr`는 표현식 결과가 저장된 위치를 의미한다.

---
### E → E1 + E2

```text
E → E1 + E2
{
  E.addr := newtemp();
  emit(E.addr ':=' E1.addr '+' E2.addr)
}
```

덧셈 표현식이다.

예를 들어 `a + b` 를 만나면, 먼저 결과를 저장할 임시 변수를 만든다.

```text
E.addr := newtemp()
```

예를 들어 `t1`이 만들어졌다고 하면, `t1 := a + b` 같은 three-address code를 생성한다.

정확히는 `a`, `b`도 각각 `E1.addr`, `E2.addr`로 표현된다.

```text
t1 := E1.addr + E2.addr
```

---
### E → E1 * E2

곱셈도 똑같다.

```text
E → E1 * E2
{
  E.addr := newtemp();
  emit(E.addr ':=' E1.addr '*' E2.addr)
}
```

예를 들어 `a * b` 이면, `t1 := a * b` 가 된다.

---
### E → -E1

단항 minus다.

```text
E → - E1
{
  E.addr := newtemp();
  emit(E.addr ':=' 'uminus' E1.addr)
}
```

예를 들어 `-a` 이면, `t1 := uminus a` 처럼 번역된다.

여기서 `uminus`는 이항 연산자 `-`와 구분하기 위한 단항 minus 연산이다.

---
### E → (E1)

```text
E → ( E1 )
{
  E.addr := E1.addr
}
```

괄호는 계산 결과를 바꾸지 않는다.   우선순위만 조정할 뿐이다.

그래서 새 임시 변수를 만들 필요 없이, 안쪽 표현식의 주소를 그대로 사용한다.

```text
E.addr = E1.addr
```

---
### E → id

가장 중요한 부분이다.

```text
E → id
{
  p := lookup(top(tblptr), id.name);
  if p = nil then error()
  else if p.level = 0 then
      E.addr := id.addr
  else
      E.addr := fp[p.offset]
}
```

표현식에서 `id`가 나오면, 이 이름이 어떤 변수인지 symbol table에서 찾아야 한다.

예를 들어 `x + t` 에서 `x`와 `t`를 각각 lookup한다.

---
### 전역 변수 id

만약 `p.level = 0`이면 전역 변수다.

```text
E.addr := id.addr
```

즉, 전역 변수는 이름 자체 또는 전역 주소를 `E.addr`로 사용한다.

예를 들어 전역 변수 `x`라면, `E.addr = x` 처럼 볼 수 있다.

---
### 지역 변수 id

반면 지역 변수라면 함수의 frame 안에 있다.

```text
E.addr := fp[p.offset]
```

예를 들어 `t`가 offset 8에 있으면, `E.addr = fp[8]` 이다.
즉, 표현식에서 `t`를 사용하면 실제로는 현재 함수 frame의 offset 8 위치에 있는 값을 읽는 것이다.

---
