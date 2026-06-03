앞의 내용을 하나 흐름으로 보면, **컴파일러가 의미적으로 맞는 프로그램인지 검사하는 방법**을 설명하는 부분이다. 앞에서 symbol table을 이용해서 이름, 타입, 스코프를 관리했다면, 이제는 그 정보를 이용해서 **정적 검사(static checking)** 를 수행하는 단계라고 보면 된다.

---
# Static Checking vs Dynamic Checking

컴파일러가 프로그램을 검사하는 방식은 크게 두 가지가 있다.

- static checking → compile time에 검사
- dynamic checking → run time에 검사

**Static checking**은 컴파일할 때 미리 확인할 수 있는 규칙을 검사하는 것이다.

예를 들어,

```c
int a;
float b;
a = b + 1;
```

이런 코드에서 `a`는 int이고 `b`는 float이라는 정보는 컴파일 타임에 알 수 있다. 그래서 대입이 가능한지, 자동 형 변환이 필요한지, 아니면 오류인지 컴파일러가 미리 판단할 수 있다.

반면 **dynamic checking**은 실행 중에야 알 수 있는 것을 검사하는 것이다.

예를 들어 배열 접근에서 `A[i]` 가 있을 때, `i` 값이 실제로 배열 범위 안에 있는지는 실행 중에 결정될 수 있다. 이 경우 컴파일러가 검사 코드를 넣어두고, 실행 중에 확인하게 만들 수 있다.

---
## Static Checking의 종류

정적 검사의 대표적인 예시는 네 가지다.
- Type checks
- Flow-of-control checks
- Uniqueness checks
- Name-related checks

이 네 가지는 전부 컴파일 타임에 어느 정도 확인 가능한 의미 규칙들이다.

## Type Checks, Overloading, Coercion, Polymorphism

여기는 타입 검사와 관련된 예시다.

```c
int op(int), op(float);
int f(float);
int a, c[10], d;
```

먼저 변수들의 타입을 보면, 아래와 같다.

```text
a → int
c → int array
d → int
```

#### `d = c + d; // FAIL`

`c`는 배열이고, `d`는 int다.

```c
d = c + d;
```

여기서 배열 `c` 자체와 정수 `d`를 더하는 것은 일반적으로 허용되지 않는다. 그래서 type error가 된다.

#### `*d = a; // FAIL`

`*d`는 d가 pointer일 때 가능한 표현이다. 그런데 `d`는 int로 선언되어 있다.

```c
int d;
*d = a;
```

즉, 정수를 pointer처럼 dereference하려고 한 것이므로 오류다.

#### `a = op(d); // OK: overloading`

`op`가 두 개 선언되어 있다.

```c
int op(int);
int op(float);
```

이런 경우 함수 이름은 같지만 parameter 타입이 다르다. 이것을 **overloading**이라고 한다.

`d`는 int이므로, `op(d)` 는 `op(int)`를 선택하면 된다. 그래서 이 코드는 가능하다.

#### `a = f(d); // OK: coercion`

`f`는 float을 받는다.

```c
int f(float);
```

그런데 `d`는 int다.

```c
a = f(d);
```

이 경우 int를 float으로 자동 변환해서 넘길 수 있다. 이런 자동 형 변환을 **coercion**이라고 한다.
 
즉, d는 int긴 하지만f float 으로 자동 변환되서 f 함수에 전달되는 것이다.

#### `vector<int> v; // OK: template instantiation`

`vector<int>`는 template에 타입 인자 `int`를 넣어서 실제 타입을 만드는 것이다.

이런 것을 **polymorphism** 또는 template instantiation과 연결해서 볼 수 있다. 하나의 일반화된 타입/함수 정의가 여러 타입에 대해 사용될 수 있다는 의미다.

---
# Flow-of-Control Checks

여기는 제어 흐름 관련 검사를 설명한다.

대표적으로 `break`가 어디서 사용될 수 있는지를 검사한다.

#### 잘못된 경우

```c
myfunc()
{
    ...
    break; // ERROR
}
```

`break`는 아무 곳에서나 쓸 수 없다. 보통 `while`, `for`, `switch` 안에서만 의미가 있다.
따라서 반복문이나 switch 밖에서 `break`를 쓰면 오류다.

#### 올바른 경우: while 내부

```c
while (n)
{
    ...
    if (i > 10)
        break; // OK
}
```

여기서 `break`는 while 반복문을 빠져나가는 의미가 있으므로 가능하다.

#### 올바른 경우: switch 내부

```c
switch (a)
{
    case 0:
        ...
        break; // OK
    case 1:
        ...
}
```

여기서도 `break`는 switch를 빠져나가는 의미가 있으므로 가능하다.

---
# Uniqueness Checks

여기는 같은 스코프 안에서 이름이 중복 선언되는지를 검사하는 내용이다.

#### 지역 변수 중복

```c
myfunc()
{
    int i, j, i; // ERROR
}
```

같은 함수 블록 안에서 `i`가 두 번 선언되었다.  
같은 스코프 안에서는 같은 이름의 변수를 중복 선언할 수 없으므로 오류다.

#### parameter 중복

```c
cnufym(int a, int a) // ERROR
{
    ...
}
```

함수 parameter 이름도 같은 parameter list 안에서는 중복될 수 없다.

#### struct 이름 중복

```c
struct myrec
{
    int name;
};

struct myrec // ERROR
{
    int id;
};
```

같은 스코프에 `struct myrec`가 이미 있는데 다시 같은 이름으로 선언하면 오류다.

즉, uniqueness check는 symbol table을 이용해서 현재 스코프에 이미 같은 이름이 있는지 확인하는 검사다.

---
# Name-Related Checks

여기는 이름과 이름의 연결 관계를 검사하는 예시다.

예시가 Java labeled loop이다.

```java
LoopA: for (int I = 0; I < n; I++)
{
    ...
    if (a[I] == 0)
        break LoopB; // Java labeled loop
    ...
}
```

여기서 `LoopA`는 label 이름이다. 그런데 안에서는 `break LoopB;`를 하고 있다.
이 경우 컴파일러는 `LoopB`라는 label이 실제로 존재하는지 확인해야 한다.

만약 `LoopB`가 선언되어 있지 않다면 오류다.

즉, name-related check는 단순 변수뿐 아니라 label, 함수, 타입 이름 등 여러 종류의 이름이 올바르게 사용되었는지 확인하는 것이다.

---
