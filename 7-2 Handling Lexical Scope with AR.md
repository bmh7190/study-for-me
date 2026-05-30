# Mapping Names to Values


# Scope Rules
같은 이름의 변수가 여러 번 등장할 때, 그 이름이 정확히 무엇을 가리키는지 어떻게 확인할까?

프로그램 안에서는 같은 이름이 여러 곳에서 쓰일 수 있다. 예를 들어 이 예시에서는 `x`라는 이름이 두 번 나온다.

![](../images/Pasted%20image%2020260529180310.png)

하나는 바깥쪽에 선언된 **function x**이고, 다른 하나는 `procedure p` 안에 선언된 **local variable x**이다.

---
# Scoping Rules

### Scoping
Scoping은 쉽게 말하면 **이 이름이 정확히 어떤 선언을 가리키는지 정하는 규칙**이다.

프로그램 안에는 같은 이름이 여러 번 나올 수 있다. 예를 들어 `x`라는 이름이 전역 변수로도 있고, 함수 이름으로도 있고, 어떤 procedure 안의 지역 변수로도 있을 수 있다.

그런데 코드에서 그냥 `x`라고 쓰면 컴파일러는 이걸 보고 판단해야 한다.

“이 `x`는 어느 `x`인가?”

이걸 결정하는 것이 **scope**이고, 그 규칙이 **scoping rule**이다.

즉 scoping은 단순히 “변수가 보인다 / 안 보인다” 정도가 아니라, 더 정확히는 **각 name이 어떤 instance를 참조하는지 정의하는 것**이다. 여기서 instance라는 말은 “같은 이름을 가진 여러 선언 중 실제로 선택되는 하나”라고 보면 된다.

예를 들어 바깥에 `x`가 있고 안쪽 블록에도 `x`가 있으면, 안쪽 블록에서 `x`를 사용할 때는 보통 안쪽 `x`를 가리킨다. 이때 안쪽 `x`가 선택되는 것이 scoping rule에 의해 결정되는 것이다.

---
### Lexicla Scoping
Lexical scoping은 **소스 코드의 구조를 보고 이름을 결정하는 방식**이다.
즉 프로그램을 실제로 실행해보지 않아도, 코드가 어떻게 중첩되어 있는지만 보면 어떤 이름이 어떤 선언을 가리키는지 알 수 있다.

예를 들어 이런 구조가 있다고 해보자.

```c++
procedure outer;
  var x : integer;

  procedure inner;
  begin
    x := 1;
  end;
```

`inner` 안에서 `x := 1;`을 보면, `inner` 안에는 `x`가 없다. 그러면 lexical scoping에서는 소스 코드상 바깥쪽으로 올라가면서 `x`를 찾는다. 바로 바깥인 `outer`에 `x`가 있으니까, 이 `x`는 `outer`의 지역 변수 `x`를 의미한다.

중요한 건 **호출 순서가 아니라 코드의 중첩 구조를 본다**는 점이다.

그래서 lexical scoping에서는 컴파일러가 미리 판단할 수 있다.

“현재 procedure는 lexical depth가 몇이고, 이 변수가 선언된 depth는 몇이니까 access link를 몇 번 따라가면 되겠다.”

이런 식으로 compile time에 이름 해석이 가능하다.

---
### Dynamic Scoping
Dynamic scoping은 lexical scoping과 반대로 **실행 중 호출 관계를 보고 이름을 결정하는 방식**이다. 즉 소스 코드상 어디에 선언되어 있는지를 보는 것이 아니라, 프로그램이 실행될 때 현재 함수가 누구에게 호출되었는지, 그 호출 스택을 따라가면서 같은 이름을 찾는다.

예를 들어 이런 상황을 생각해보자.

```
A가 B를 호출하고,
B 안에서 x를 사용한다.
```

Dynamic scoping에서는 `B`의 코드상 바깥 scope를 보는 것이 아니라, **B를 호출한 A의 실행 환경**을 볼 수 있다. 그래서 A에 `x`가 있으면 B 안의 `x`가 A의 `x`를 가리킬 수도 있다. 즉 dynamic scoping에서는 같은 함수 `B`라도 누가 호출했는지에 따라 `x`가 다른 변수를 가리킬 수 있다.

이게 lexical scoping과 가장 큰 차이다.
Lexical scoping에서는 `B` 안의 `x`가 무엇인지는 코드 구조만 보면 정해진다.  
Dynamic scoping에서는 `B` 안의 `x`가 무엇인지는 실행 흐름을 봐야 정해진다.

```c
int x = 1;

void A() {
    int x = 2;
    B();
}

void B() {
    print(x);
}
```

여기서 `B()` 안의 `x`가 무엇인지 보자.

**Lexical scoping**이면 `B`가 코드상 어디에 정의되어 있는지를 본다.  
`B`는 전역 위치에 정의되어 있으니까, `B` 안에서 보이는 `x`는 전역 변수 `x = 1`이다.
즉 lexical scoping에서는 결과가 `1`이 된다.

그런데 **dynamic scoping**이면 `B`를 누가 호출했는지를 본다.  
실행 흐름은 `A()`가 `B()`를 호출한 상황이다. 그리고 `A` 안에는 `x = 2`가 있다.
그래서 dynamic scoping에서는 `B` 안의 `x`가 `A`의 `x`를 가리키게 되어 결과가 `2`가 될 수 있다.

---
# Lexical Scoping Example

![](../images/Pasted%20image%2020260530140839.png)

이 예시에서는 여러 procedure와 function이 바깥 scope에 선언된 변수를 사용하고 있다. 이러한 변수 접근은 실행 중 호출 관계가 아니라, 코드가 작성된 중첩 구조를 기준으로 결정되기 때문에 **lexical scoping**으로 이해할 수 있다.

### Lexical VS Dynmaic Scoping
특히 헷갈릴 수 있는 부분은 `exchange`이다. `exchange`는 코드상으로는 `sort` 바로 아래에 선언되어 있지만, 실제 실행 중에는 `partition` 안에서 호출된다. 하지만 lexical scoping에서는 “누가 호출했는가”가 아니라 “어디에 선언되어 있는가”를 기준으로 이름을 찾는다. 따라서 `exchange` 안에서 사용하는 `a`나 `x`는 호출자인 `partition`의 변수가 아니라, `exchange`를 감싸고 있는 `sort`의 변수로 해석된다.

만약 dynamic scoping이라면 이야기가 달라질 수 있다. Dynamic scoping에서는 `exchange`가 코드상 어디에 정의되어 있는지를 보는 것이 아니라, 실행 중에 **누가 `exchange`를 호출했는지**를 기준으로 이름을 찾는다. 오른쪽 call tree에서는 `partition`이 `exchange`를 호출하고 있으므로, `exchange` 안에서 이름을 찾을 때 현재 `exchange`의 환경에서 먼저 찾고, 없으면 호출자인 `partition`의 환경을 확인한다. 그다음에는 `partition`을 호출한 `quicksort`, 그 위의 `quicksort`, 마지막으로 `sort` 순서로 call stack을 따라 올라가며 같은 이름의 변수를 찾게 된다.

### Symbol table 을 사용한다.
또 하나 중요한 점은 `partition` 안에서 사용되는 `v`이다. `v`는 `partition` 내부에서 선언된 변수가 아니라, 바깥쪽 procedure인 `quicksort`에서 선언된 지역 변수이다. 따라서 `quicksort` 입장에서는 `v`가 지역 변수이고, `partition` 입장에서는 바깥 scope에 있는 **non-local variable**이다.

이처럼 같은 이름이나 변수 참조가 어떤 선언에 연결되는지는 각 scope의 **symbol table**을 통해 확인한다. 컴파일러는 현재 scope의 symbol table에서 먼저 이름을 찾고, 없으면 lexical parent의 symbol table로 올라가면서 해당 이름을 찾는다. 이 과정을 통해 `partition`의 `v`는 `quicksort`의 `v`로, `exchange`의 `a`와 `x`는 `sort`의 `a`와 `x`로 결정된다.

---
# Nested Procedures & Symbol Tables
앞에서 본 lexical scoping을 **컴파일러가 symbol table로 어떻게 관리하는지**를 보여준다.

앞에서는 `partition` 안에서 `v`를 쓰면 `quicksort`의 `v`를 의미하고, `exchange` 안에서 `a`, `x`를 쓰면 `sort`의 `a`, `x`를 의미한다고 했다.  

그 판단을 컴파일러가 어떻게 할까?

**각 procedure/function마다 자기 scope에 해당하는 symbol table이 있고, 그 symbol table들이 lexical nesting 구조에 맞게 연결되어 있다.**

![](../images/Pasted%20image%2020260530142039.png)

각 procedure는 자기 scope 안에서 선언된 변수, 매개변수, 함수 정보를 **symbol table**로 관리한다. Symbol table에는 보통 이름, 종류, 크기, 타입, offset 같은 정보가 저장된다. 여기서 offset은 나중에 activation record 안에서 해당 변수를 찾기 위해 사용된다.

그리고 procedure 안에 또 다른 procedure가 선언되는 **nested procedure** 구조에서는 symbol table도 중첩 구조를 가진다. 즉 내부 procedure의 symbol table은 자기 바깥 scope에 해당하는 procedure의 symbol table과 연결된다.

예를 들어 `partition`은 `quicksort` 안에 선언되어 있으므로, `partition`의 symbol table은 `quicksort`의 symbol table을 가리킨다. 그리고 `quicksort`는 `sort` 안에 선언되어 있으므로, `quicksort`의 symbol table은 `sort`의 symbol table과 연결된다.

이렇게 연결해두면 컴파일러는 어떤 이름을 찾을 때 현재 procedure의 symbol table에서 먼저 찾고, 없으면 바깥 procedure의 symbol table로 올라가면서 찾을 수 있다. 이 과정을 통해 `partition` 안에서 사용된 `v`는 `quicksort`의 지역 변수로, `a`는 `sort`의 변수로 해석된다.


---
# Static Allocation

Static allocation은 procedure의 지역 변수 저장공간을 **컴파일 시점에 고정된 위치로 미리 배정하는 방식**이다. 즉 함수가 호출될 때마다 새로운 공간을 만드는 것이 아니라, `A`는 항상 `A`용 저장공간을 쓰고, `B`는 항상 `B`용 저장공간을 사용한다.

![](../images/Pasted%20image%2020260530142559.png)

예를 들어 `subroutine A`에는 `i`, `a(100)`, `b(10,10)`이 있고, `subroutine B`에는 `i`, `c(10)`이 있다. Static allocation에서는 이 변수들의 위치가 미리 정해져 있기 때문에, 컴파일러는 변수 접근 코드를 단순하게 만들 수 있다. 그래서 **code generation이 단순하다**는 장점이 있다.

하지만 한계도 분명하다. 변수 크기가 컴파일 시점에 정해져 있어야 하므로 **고정 크기 변수만 다루기 쉽고**, 같은 procedure가 동시에 여러 번 실행되는 **recursion을 지원하기 어렵다**. 재귀 호출에서는 호출마다 서로 다른 지역 변수 공간이 필요하지만, static allocation은 procedure마다 저장공간이 하나뿐이기 때문이다.

정리하면, static allocation은 **저장 위치를 미리 고정해서 구현은 단순하지만, 재귀나 동적 메모리처럼 실행 중에 유연한 저장공간이 필요한 경우에는 적합하지 않은 방식**이다.

---
# Lexical Scopes Without Nested Procedures
중첩 procedure가 없다고 생각하면 변수 위치를 찾는 일은 비교적 단순하다.

변수는 보통 두 가지 중 하나다.

- 첫째, **local variable**이다.  
	현재 실행 중인 procedure 안에서 선언된 변수라면, 현재 activation record 안에서 찾으면 된다. 즉 AR pointer를 기준으로 정해진 offset만큼 이동하면 해당 변수의 저장공간을 찾을 수 있다.

- 둘째, **global variable**이다.  
	procedure 바깥, 즉 전역 영역에 선언된 변수라면 global data 영역의 정해진 offset에서 찾으면 된다.

그래서 nested procedure가 없을 때는 변수 접근이 단순하다.

- local variable → 현재 AR 안에서 찾기  
- global variable → global area의 고정 offset에서 찾기`

이 정도로 처리할 수 있다.

그런데 여기서 질문이 생긴다.

### **그럼 왜 stack이 필요한가?**
이유는 procedure가 호출될 때마다 새로운 실행 상태가 필요하기 때문이다. 같은 procedure라도 여러 번 호출될 수 있고, 특히 recursion이 있으면 같은 procedure의 activation이 동시에 여러 개 존재할 수 있다. 그러면 각 호출마다 local variable, parameter, return address 등을 따로 저장해야 한다. 그래서 activation record를 stack에 쌓아 관리한다.

즉, nested procedure가 없으면 **변수 위치 자체는 찾기 쉽지만**, procedure 호출마다 독립적인 실행 상태를 유지하기 위해 stack은 여전히 필요하다.

---
### Lexical Scopes Without Nested Procedures - Problem
앞의 설명은 nested procedure가 없을 때의 단순한 경우였다. 그런데 procedure 안에 또 다른 procedure가 정의되면 문제가 생긴다.

예제에서 `sort` 안에는 `a`, `x`가 있고, `quicksort` 안에는 `k`, `v`가 있다. 그리고 `partition`은 `quicksort` 안에 정의되어 있다.

이때 `partition` 안에서 `v`를 사용하면, `v`는 현재 `partition`의 지역 변수가 아니다. 바깥쪽 procedure인 `quicksort`의 지역 변수다. 또 `partition` 안에서 `a`를 사용하면, `a`는 `partition`에도 없고 `quicksort`에도 없다. 더 바깥쪽인 `sort`에 선언된 변수다.

그래서 단순히 “현재 AR 아니면 global”로는 부족하다.  
중간 scope에 있는 변수도 찾아야 하기 때문이다.

### Static Nesting Depth
이를 해결하려면 각 procedure가 코드상 몇 번째 깊이에 있는지, 즉 **static nesting depth**를 알아야 한다.

예제에서는 다음처럼 볼 수 있다.

- `sort`는 depth 1  
- `readarray`, `quicksort`는 depth 2  
- `partition`은 depth 3

그러면 `partition`에서 `v`를 찾을 때는 depth 3에서 depth 2로 한 단계 올라가면 되고, `a`를 찾을 때는 depth 3에서 depth 1로 두 단계 올라가면 된다.

이 depth 정보는 컴파일 시점에 알 수 있다. 왜냐하면 lexical scoping은 실행 흐름이 아니라 코드의 중첩 구조를 기준으로 하기 때문이다. 실행 중에는 이 depth 차이를 이용해서 activation record 안의 link를 따라간다. 

이것을 **link chasing in AR**이라고 보면 된다. 즉 현재 AR에서 시작해서 lexical parent의 AR을 따라 올라가며 필요한 변수가 선언된 scope의 AR을 찾는 것이다.

정리하면, nested procedure가 없으면 변수는 local 또는 global로 쉽게 찾을 수 있다. 하지만 nested procedure가 있으면 `partition`이 `quicksort`의 `v`, `sort`의 `a`처럼 바깥 procedure의 지역 변수를 접근해야 하므로, lexical nesting depth와 AR link를 이용해 정확한 저장공간을 찾아야 한다.

---
# Activation Records on the Stack 
함수가 호출될 때마다 해당 함수의 실행 상태를 담는 **activation record**가 stack에 쌓인다. 여기서는 `sort → quicksort → quicksort → partition → exchange` 순서로 호출이 진행되면서 AR이 어떻게 쌓이는지 보여준다.

![](../images/Pasted%20image%2020260530143545.png)

stack에는 **실행 중 호출 순서대로 AR이 쌓이지만**, access link는 **코드상 lexical nesting 구조를 따라 연결된다**.  control flow는 `partition → exchange`일 수 있지만, lexical scope 기준으로 `exchange`의 바깥 scope는 `sort`이기 때문에 access link는 `sort`를 가리킨다.

근데 이 access link는 어떻게 연결 짓는가? 

----
# Access Links and How to use Them

Access link는 **현재 activation record에서 lexical parent의 activation record로 이동하기 위한 포인터**다. 즉 현재 procedure 안에 없는 변수를 찾을 때, 실행 중 호출 관계를 따라가는 것이 아니라 **코드상 바깥 scope의 AR로 이동하기 위해 사용**한다.

기본 규칙은 다음과 같다.

현재 procedure의 lexical depth를 `np`라고 하고, 접근하려는 변수가 선언된 depth를 `nq`라고 하면, `np - nq`번 access link를 따라가면 된다. 그 다음 도착한 AR 안에서 해당 변수의 **offset** 위치를 접근하면 된다.

예를 들어 `partition`은 depth 3이다.

```
sort        depth 1
quicksort   depth 2
partition   depth 3
```

`partition` 안에서 `v`를 사용한다고 해보자. `v`는 `partition` 안에 선언된 것이 아니라 `quicksort` 안에 선언되어 있다.  
즉 `v`는 depth 2에 있다.

그러면 계산은 다음과 같다.

`np - nq = 3 - 2 = 1`

따라서 `partition`의 AR에서 access link를 **1번** 따라가면 `quicksort`의 AR에 도착한다. 그 AR 안에서 `v`의 offset 위치를 접근하면 된다.

이번에는 `partition` 안에서 `a`를 사용한다고 해보자.  

`a`는 `sort`에 선언되어 있으므로 depth 1에 있다.

`np - nq = 3 - 1 = 2`

따라서 `partition`의 AR에서 access link를 **2번** 따라가야 한다.

```
partition AR→ quicksort AR→ sort AR
```

도착한 `sort`의 AR에서 `a`의 offset 위치를 접근하면 된다.

여기서 중요한 점은 `np - nq` 값은 **compile time에 계산 가능하다**는 것이다.  
왜냐하면 lexical scoping은 코드의 중첩 구조를 기준으로 하고, 각 procedure의 depth는 컴파일 시점에 이미 알 수 있기 때문이다.

그래서 컴파일러는 `partition` 안의 `a`를 보고 다음과 같은 코드를 만들 수 있다.

```
access link 2번 따라가기→ 도착한 AR에서 a의 offset 접근
```


마지막으로 `nq > np`인 경우도 생각할 수 있다.  
예를 들어 바깥 procedure가 안쪽 procedure의 지역 변수를 접근하려는 상황이다.

하지만 lexical scoping에서는 안쪽 scope의 변수는 바깥쪽에서 보이지 않는다.  
따라서 이런 경우는 컴파일러가 허용하지 않는다.

정리하면, access link는 **non-local variable을 찾기 위해 lexical nesting 구조를 따라 AR을 올라가는 장치**다. 현재 depth와 변수 선언 depth의 차이만큼 access link를 따라가고, 도착한 AR 안에서 offset으로 변수를 찾는다.

---
## How to Set Up Access Links?

Access link는 **callee의 AR 안에 저장되는 포인터**이고, 그 포인터는 callee가 접근해야 할 **lexical parent의 AR**을 가리켜야 한다.

즉 procedure `p`가 procedure `q`를 호출할 때, 단순히 `q`의 AR만 만들면 끝이 아니라, `q`의 access link를 어디로 연결할지도 정해야 한다.

여기서는 두 경우로 나눈다.

## 1. `np < nq`인 경우
`np`는 caller `p`의 lexical depth이고, `nq`는 callee `q`의 lexical depth이다.
`np < nq`라는 것은 호출되는 `q`가 `p`보다 더 안쪽에 선언된 procedure라는 뜻이다.

![](../images/Pasted%20image%2020260530144925.png)

`quicksort`가 `partition`을 호출하면,

`np = 2`  
`nq = 3`

즉 `np < nq`이다.

이 경우 `partition`은 코드상 `quicksort` 안에 선언되어 있다. 따라서 `partition`의 lexical parent는 바로 `quicksort`다. 그래서 `partition`의 access link는 **caller인 quicksort의 AR**을 가리키면 된다.

caller의 ARP를 callee의 access link에 복사하는 것이다.

---
## 2. `np >= nq`인 경우

이 경우는 caller가 callee와 같은 depth이거나, caller가 callee보다 더 안쪽 depth에 있는 경우다. 이때 중요한 점은 **callee의 access link가 caller를 무조건 가리키는 것이 아니라, callee를 코드상 감싸는 lexical parent의 AR을 가리켜야 한다**는 것이다.

공식은 다음과 같다.

`np - nq + 1`번 access link를 따라간다.

---
#### 예시 1: quicksort가 quicksort를 호출하는 경우

![](../images/Pasted%20image%2020260530145244.png)

먼저 재귀 호출 상황을 보면, `quicksort(1,9)`가 `quicksort(1,3)`을 호출한다.

caller는 `quicksort(1,9)`이고, callee는 `quicksort(1,3)`이다.

`quicksort(1,9)`의 depth = 2  
`quicksort(1,3)`의 depth = 2

따라서 `np = 2`, `nq = 2`이고, `np >= nq`인 경우다.

이때 새로 호출된 `quicksort(1,3)`의 access link가 caller인 `quicksort(1,9)`를 가리키면 안 된다. 두 quicksort는 같은 procedure의 서로 다른 activation일 뿐이고, lexical parent 관계가 아니기 때문이다.

`quicksort`는 코드상 `sort` 안에 선언되어 있으므로, `quicksort`의 lexical parent는 항상 `sort`다. 따라서 `quicksort(1,3)`의 access link는 `sort`의 AR을 가리켜야 한다.

공식에 대입하면,

`np - nq + 1 = 2 - 2 + 1 = 1`

따라서 caller인 `quicksort(1,9)`의 AR에서 access link를 1번 따라간다.

도착한 `sort AR`을 새로 만들어진 `quicksort(1,3)`의 access link로 설정한다.

즉 stack에는 `quicksort(1,9)`와 `quicksort(1,3)`이 연속으로 쌓이지만, 두 AR의 access link는 모두 `sort`를 가리킨다.

---
#### 예시 2: partition이 exchange를 호출하는 경우
이번에는 `partition` 안에서 `exchange(i, j)`를 호출하는 경우다.

코드 구조는 다음과 같다.

```text
sort depth 1
├─ exchange depth 2
└─ quicksort depth 2
   └─ partition depth 3
```

실행 중 호출 관계는 다음과 같다.

```text
partition
→ exchange
```

하지만 lexical scoping에서는 실행 중 누가 호출했는지가 아니라, **코드상 어디에 선언되어 있는지**를 본다.

caller는 `partition`이고, callee는 `exchange`이다.

`partition`의 depth = 3  
`exchange`의 depth = 2

따라서 `np = 3`, `nq = 2`이고, 역시 `np >= nq`인 경우다.

이때 `exchange`의 access link가 caller인 `partition`을 가리키면 안 된다. `exchange`는 `partition` 안에 선언된 procedure가 아니라, `sort` 안에 선언된 procedure이기 때문이다.

`exchange`의 lexical parent는 `sort`다. 따라서 `exchange`의 access link는 `sort`의 AR을 가리켜야 한다.

공식에 대입하면,

`np - nq + 1 = 3 - 2 + 1 = 2`

따라서 caller인 `partition`의 AR에서 access link를 2번 따라간다.

```text
partition AR
→ quicksort AR
→ sort AR
```

도착한 `sort AR`을 새로 만들어진 `exchange`의 access link로 설정한다.

이렇게 해야 `exchange` 안에서 `a`나 `x`를 사용할 때, `partition`이나 `quicksort`가 아니라 `sort`의 `a`, `x`에 접근할 수 있다.

---
# Display
access link가 여러 개로 연결되고, 많아질텐데 점프를 하기 위해서 three address code 관점에서 중간 단계 코드가 너무 많아지낟. 성능 높이기 위해서 displayf라는 것을 사용한다.