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