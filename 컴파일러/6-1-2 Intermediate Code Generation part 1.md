# Types and Declarations
이제 중간 코드 생성에서 타입과 선언을 다룬다.

타입 검사는 프로그램이 실행될 때 올바른 연산이 수행될 수 있는지를 미리 확인하는 과정이다.

예를 들어,

```
int a;
double b;
a + b;
```

이 경우 `int`와 `double`을 더할 수 있는지, 더한다면 결과 타입은 무엇인지 판단해야 한다.

또는 다음과 같은 코드는 문제가 될 수 있다.

```
int a;
char *p;
a + p;
```

언어 규칙에 따라 허용될 수도 있고, 오류일 수도 있다. 컴파일러는 이런 의미적 규칙을 확인해야 한다.

타입 정보는 중간 코드 생성에도 필요하다. 왜냐하면 타입에 따라 필요한 저장 공간과 사용할 명령어가 달라지기 때문이다.

예를 들어 `int` 덧셈과 `float` 덧셈은 실제 기계어 수준에서 다른 명령어를 사용할 수 있다. 따라서 컴파일러는 IR을 만들 때 타입 정보를 함께 고려해야 한다.

---
# Type Expression
타입은 단순히 `int`, `float` 같은 기본 타입만 있는 것이 아니다. 배열, 포인터, 함수, 구조체처럼 복합 타입도 있다.

이런 타입을 표현하기 위해 **type expression**을 사용한다.

기본 타입에는 다음과 같은 것들이 있다.

```
boolean
char
int
float
void
```

여기서 `void`는 값이 없음을 나타내는 타입이다. 예를 들어 반환값이 없는 함수의 타입으로 사용된다.

복합 타입은 기본 타입에 type constructor를 적용해서 만든다. 예를 들어 배열 타입, 포인터 타입, 함수 타입 등이 있다.

## Type Constructor ( 배열, 레코드, 포인터, 함수 ) 

### 배열
배열 타입은 다음처럼 표현할 수 있다.

```
int p[10];
```

이 타입은 다음처럼 표현된다.

```
array(10, integer)
```

즉 “integer 타입 원소 10개로 이루어진 배열”이라는 뜻이다.

다차원 배열도 중첩된 배열로 표현한다.

```
int p[3][2];
```

는 다음처럼 표현할 수 있다.

```
array(3, array(2, integer))
```

### 레코드

구조체는 record 타입으로 표현한다.

```
struct {
    int p;
    char q;
} data;
```

는 대략 다음처럼 표현된다.

```
record((p × integer) × (q × char))
```


### 포인터

포인터는 다음처럼 표현한다.

```
int *p;
```

```
pointer(integer)
```


### 함수
함수 타입은 domain에서 range로 가는 mapping으로 표현한다.

```
int foo(int p, char q) {
    return 2;
}
```

이 함수는 `int`와 `char`를 입력으로 받고 `int`를 반환한다. 따라서 타입은 다음처럼 표현할 수 있다.

```
integer × char → integer
```

즉 함수 타입은 다음 구조로 본다.

```
입력 타입들 → 반환 타입
```

---
# Type Equivalence
타입 검사의 핵심 중 하나는 두 타입이 같은지 판단하는 것이다. 이를 **type equivalence**라고 한다.

예를 들어 두 값이 모두 `int`이면 같은 타입이다. 그런데 배열이나 포인터처럼 복합 타입이면 단순히 이름만 비교하면 부족하다.

두 타입이 동등하다고 판단하는 기준은 보통 다음과 같다.

1. 같은 basic type이면 동등하다.
2. 같은 type constructor로 만들어졌고 내부 타입도 동등하면 동등하다.
3. 하나가 다른 타입 이름을 가리키는 경우에도 동등할 수 있다.

예를 들어,

```
array(10, integer)
array(10, integer)
```

는 같은 타입으로 볼 수 있다.

하지만,

```
array(10, integer)
array(20, integer)
```

는 크기가 다르므로 다른 타입이다.

또,

```
pointer(integer)
pointer(float)
```

도 내부 타입이 다르기 때문에 다른 타입이다.

타입 동등성 검사가 중요한 이유는, 올바른 연산인지 확인하고 그에 맞는 기계어 명령어를 선택해야 하기 때문이다.

---
# Declaration and Storage Layout
변수가 선언되면 컴파일러는 그 변수의 타입을 보고 필요한 저장 공간을 계산해야 한다.

예를 들어,

```
int a;
double b;
int c[10];
```

이렇게 선언되어 있다고 하자.

각 타입의 크기가 다음과 같다고 하면,

```
int    = 4 bytes
double = 8 bytes
```

저장 공간은 다음처럼 계산할 수 있다.

```
a : int             → 4 bytes
b : double          → 8 bytes
c : array(10, int)  → 40 bytes
```

컴파일러는 이 정보를 symbol table에 저장한다.

symbol table에는 보통 다음 정보가 들어간다.

```
name
type
width
offset
```

여기서 offset은 해당 변수가 현재 activation record나 메모리 영역에서 어느 위치에 배치되는지를 나타낸다.

예를 들어 다음처럼 배치할 수 있다.

```
Name   Type             Width   Offset
a      int              4       0
b      double           8       4
c      array(10,int)    40      12
```

그리고 다음 변수를 넣을 때는 offset을 계속 증가시킨다.

```
offset = offset + width
```

즉 선언을 처리한다는 것은 단순히 “변수가 있다”를 기록하는 것이 아니라, **이 변수의 타입이 무엇이고, 메모리를 얼마나 차지하며, 어디에 배치되는지 결정하는 과정**이다.

---
# Sequence of Declarations

선언문을 처리할 때는 문법 규칙에 semantic action을 붙여 symbol table을 채운다.

예를 들어 문법이 다음과 같다고 하자.

```
P → D
D → T id ; D
D → ε
```

이는 선언들이 연속해서 나오는 구조를 의미한다.

```
int a;
double b;
int c[10];
```

이런 코드를 처리하려면 먼저 symbol table을 만들고, offset을 0으로 초기화한다.

```
mktable()
offset = 0
```

그다음 선언 하나를 만날 때마다 다음 작업을 한다.

```
put(id.lexeme, T.type, offset)
offset = offset + T.width
```

즉 `int a;`를 만나면 symbol table에 `a`를 넣고 offset을 4 증가시킨다. `double b;`를 만나면 `b`를 넣고 offset을 8 증가시킨다. `int c[10];`을 만나면 `c`를 넣고 offset을 40 증가시킨다.

이 과정은 저번에 배운 SDT와 연결된다. 문법 규칙에 semantic action을 붙여서, parsing 과정 중에 symbol table을 구성하는 것이다.
