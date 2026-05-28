# Procedure Activation and Lifetime

# Activaion Record Basics
Activation Record는 줄여서 **AR**이라고 부르고, 쉽게 말하면 **함수나 procedure가 실행되는 동안 필요한 정보를 저장하는 공간**이다.

함수가 호출되면 단순히 코드만 실행되는 것이 아니라, 그 함수가 실행되기 위해 필요한 여러 정보가 같이 필요하다. 예를 들어 매개변수 값, 지역 변수, 반환값, 함수가 끝난 뒤 돌아갈 주소, 이전 함수의 실행 상태 등이 필요하다. 이런 것들을 한 덩어리로 묶어서 저장한 것이 Activation Record이다.

**procedure가 호출될 때마다 AR이 하나씩 생긴다**. 여기서 중요한 점은 “함수 하나당 AR 하나”가 아니라, **호출 한 번당 AR 하나**라는 것이다.

예를 들어 `factorial(3)`이 실행되면서 `factorial(2)`, `factorial(1)`을 호출하면 함수 코드는 하나지만 호출은 여러 번 일어난다. 따라서 AR도 여러 개 생긴다.

```
factorial(3)의 ARfactorial(2)의 ARfactorial(1)의 AR
```

각 호출마다 매개변수 `n`의 값이 다르기 때문에 같은 함수라도 AR을 따로 가져야 한다.

---

![](../images/Pasted%20image%2020260528003344.png)
### Activation Record Pointer
왼쪽에 있는 **ARP**는 Activation Record Pointer이다. 

현재 실행 중인 Activation Record를 가리키는 포인터이다. 그림에서 ARP가 Activation Record의 중간쯤을 가리키고 있는데, 이것은 AR 안의 여러 정보에 접근할 때 기준점 역할을 한다.

예를 들어 지역 변수에 접근할 때는 ARP를 기준으로 아래쪽 offset을 사용하고, parameter에 접근할 때는 ARP를 기준으로 위쪽 offset을 사용할 수 있다.

컴파일러 입장에서는 변수 이름을 직접 찾는 것이 아니라,

```
ARP + offsetARP - offset
```

같은 방식으로 AR 안의 값을 찾는다.

---
### parameters
맨 위의 **parameters**는 현재 routine에 전달된 매개변수들이 저장되는 공간이다.

예를 들어 다음 코드가 있다고 하자.

```c
int add(int a, int b) {
    int result;
    result = a + b;
    return result;
}
```

`add(3, 5)`를 호출하면 `a = 3`, `b = 5`라는 정보가 필요하다. 이 값들이 AR의 parameters 영역에 저장된다. 즉 parameters 영역은 caller가 callee에게 넘겨준 값을 callee가 사용할 수 있도록 저장하는 공간이다.

---
### register save area
그다음 **register save area**는 register 값을 저장해 두는 공간이다.

함수를 호출하면 CPU register에 있던 값이 callee에 의해 바뀔 수 있다. 그런데 caller 입장에서는 함수 호출이 끝난 뒤에도 기존 register 값이 필요할 수 있다. 

그래서 함수 호출 전후로 필요한 register 값을 AR에 저장해 두었다가 나중에 복원한다. 쉽게 말하면 이 영역은 **함수 호출 때문에 잃어버리면 안 되는 CPU 상태를 잠시 저장하는 공간**이다.

---
### reture value
**return value**는 함수가 결과값을 반환할 때 사용하는 공간이다.

예를 들어 `add(3, 5)`의 결과가 `8`이라면, 이 `8`을 caller에게 넘겨야 한다.

```c
int x = add(3, 5);
```

여기서 `add`가 계산한 결과 `8`은 caller인 `main` 쪽의 `x`로 전달되어야 한다. 그 값을 담아두는 공간이 return value 영역이다.

단, 실제 시스템에서는 return value를 항상 AR에 저장하는 것은 아니고 register를 사용하는 경우도 많다. 하지만 개념적으로는 함수의 반환값이 caller에게 전달될 공간이 필요하다고 보면 된다.

---
### return address
**return address**는 함수가 끝난 뒤 다시 돌아갈 주소이다. 함수 호출은 현재 위치에서 callee로 실행 흐름이 이동하는 것이다. 그런데 callee가 끝나면 caller의 다음 명령어로 돌아와야 한다.

예를 들어 다음 코드에서,

```c
x = add(3, 5);
y = x + 1;
```

`add(3, 5)`를 실행하러 갔다가 함수가 끝나면 다시 `x = add(3, 5);` 이후 위치로 돌아와야 한다. 그래야 다음 줄인 `y = x + 1;`을 실행할 수 있다.

이때 “어디로 돌아가야 하는지”를 저장한 것이 return address이다.

즉 return address는 **caller의 실행을 재개할 주소**이다.

---
### access link
**access link**는 non-local variable에 접근하기 위한 link이다. 여기서 non-local variable은 현재 함수 안에서 선언된 변수는 아니지만, 바깥 scope에 있어서 접근 가능한 변수를 말한다.

예를 들어 nested procedure가 있다고 하자.

```c
procedure outer
    variable x

    procedure inner
        x 사용
```

`inner` 안에서 `x`를 사용하지만, `x`는 `inner`의 지역 변수가 아니라 `outer`의 변수이다. 이런 경우 `inner`의 AR만 보면 `x`를 찾을 수 없다. 그래서 `inner`의 AR에는 바깥 lexical scope인 `outer`의 AR을 가리키는 link가 필요하다. 그게 access link이다.

즉 access link는 **소스 코드상 바깥 scope의 AR로 이동하기 위한 포인터**이다.

---
### caller’s ARP
**caller’s ARP**는 caller의 Activation Record Pointer를 저장하는 공간이다.

함수가 호출되면 현재 실행 중인 caller에서 callee로 넘어간다. 그런데 callee가 끝나면 다시 caller의 AR로 돌아와야 한다. 그러려면 callee의 AR 안에 caller의 ARP를 저장해 두어야 한다.

예를 들어 `main`이 `add`를 호출했다면, `add`의 AR 안에는 `main`의 ARP가 저장된다. `add`가 끝나면 이 값을 이용해서 다시 `main`의 AR을 복원한다. 즉 caller’s ARP는 **return할 때 caller의 AR을 복원하기 위한 정보**이다.

이 부분은 **control link**라고도 볼 수 있다.

---
### local variables
맨 아래의 **local variables**는 현재 routine 안에서 선언된 지역 변수들이 저장되는 공간이다.

예를 들어 다음 코드에서,

```c
int add(int a, int b) {
    int result;
    result = a + b;
    return result;
}
```

`result`는 `add` 함수 안에서 선언된 local variable이다. 따라서 `add`의 AR 안에 저장된다.

“including spills”라고 되어 있는데, 여기서 spill은 register에 다 담지 못한 임시값을 memory로 내려놓는 것을 말한다. 즉 local variables 영역에는 진짜 지역 변수뿐 아니라, 계산 중 필요한 임시값도 들어갈 수 있다.

---
# Activation Records on the Stack
이번엔 procedure/function call이 일어날 때 stack에서 어떤 일이 발생하는지 보자.

아래 그림을 보면 stack 안에 Activation Record가 두 개 있다. 위쪽은 **Caller’s Activation Record**이고, 아래쪽은 **Callee’s Activation Record**이다.

![](../images/Pasted%20image%2020260528004302.png)

### What happens on a Call?
함수를 호출할 때 무슨 일이 발생할까? 크게 두 가지로 발생한다.

#### 1. Passing of Arguments
함수 호출이 일어나면 caller는 calle에게 argument를 넘겨야 한다. 

예를 들어 `add(3, 5)`를 호출하면 `3`과 `5`가 `add` 함수의 parameter인 `a`, `b`에 전달되어야 한다.

```c
add(3, 5)
```

여기서 `3`, `5`는 **actual arguments**이고, `add` 함수 정의의 `a`, `b`는 **formal parameters**이다.

```c
int add(int a, int b)
```

즉 호출 시에는 다음 대응이 만들어진다.

```
a ← 3
b ← 5
```

이 값들은 callee의 Activation Record 안에 있는 **parameters 영역**에 저장된다.

그래서 함수 호출이 일어날 때 caller는 단순히 “add로 이동해라”만 하는 것이 아니라, callee가 사용할 수 있도록 argument를 callee의 AR에 넣어줘야 한다.

---
#### 2. Transfer of Control
두 번째는 control transfer이다. 함수 호출은 실행 흐름이 caller에서 callee로 이동하는 것이다.

예를 들어 `main`이 실행되다가 `add(3, 5)`를 만나면, CPU는 `main`의 다음 줄을 계속 실행하는 것이 아니라 `add` 함수의 시작 주소로 이동한다.

즉 실행 흐름은 다음처럼 바뀐다.

```text
main 실행 중 → add 호출 → add 함수 코드로 이동 → add 실행
```

이때 중요한 점은 `add`가 끝나면 다시 `main`으로 돌아와야 한다는 것이다. 그래서 호출할 때 **return address**를 저장해 둔다. 

---
## What happens a a Return?
이번에는 반대로 함수가 반환될 때 무슨 일이 일어나는지 알아보자.

반환할 때도 크게 두 가지 일이 발생한다.

#### 1. Recovery of Results
함수가 결과를 반환하는 경우, caller는 그 결과를 받아야 한다.

예를 들어 아래 함수를 보자

```c
x = add(3, 5);
```

`add` 함수는 `8`을 반환한다. 그러면 caller인 `main`은 이 결과를 받아서 `x`에 저장해야 한다.

쉽게 말해, callee가 계산한 결과를 caller가 회수하는 것이다.

반환값은 AR 안의 return value 영역에 있을 수도 있고, 실제 시스템에서는 register에 들어 있을 수도 있다. 중요한 것은 callee가 만든 결과가 caller에게 전달되어야 한다는 점이다.

---
#### 2. Transfer of Control
함수가 끝나면 실행 흐름이 다시 caller로 돌아가야 한다. 이때 return address를 사용한다.

callee의 AR 안에는 return address가 저장되어 있다. callee가 끝날 때 이 주소를 꺼내서 해당 위치로 점프한다.

오른쪽 그림에서 **Return** 화살표가 위쪽으로 표시된 이유가 이것이다. callee의 AR이 제거되고, 다시 caller의 AR을 기준으로 실행이 이어진다.



함수를 호출하고, 반환하는 이 모든 과정에서  caller의 실행 상태를 잃지 않기 위해 ARP, PC, access link, register values 같은 execution context를 저장하고 복원해야 한다.

---
# Procedure Linkages
함수 호출을 실제로 구현하기 위해서 위의 과정들이 어떻게 적용되는지 직접 적을 살펴보자

![](../images/Pasted%20image%2020260528005842.png)

지금 위의 그림에서는 `p`가 실행되다가 `q`를 호출하면, `p` 안에서 **pre-call sequence**가 실행된다. 그 후 제어 흐름이 `q`로 넘어가고, `q`는 처음에 **prolog**를 실행한다. `q`의 본문이 끝나면 **epilog**를 실행하고, 다시 `p`로 돌아온다. 돌아온 직후 `p`는 **post-return sequence**를 실행한다.

중요한 점은 **procedure 자체가 가지는 코드**와 **call site마다 필요한 코드**가 다르다는 점이다. 

`prolog`와 `epilog`는 procedure마다 거의 표준적으로 존재한다. 즉 `q`라는 procedure가 있으면, `q`의 시작 부분에는 prolog가 있고 끝 부분에는 epilog가 있다.

반면 `pre-call`과 `post-return`은 **호출 위치마다** 존재한다. `p`가 `q`를 여러 번 호출하면, 각각의 call site마다 pre-call과 post-return이 필요하다.

----
### Pre-call Sequence
Pre-call sequence는 **caller가 callee를 호출하기 직전에 수행하는 작업**이다.

이 부분에서는 두 가지 역할을 수행한다.

1. Sets up Callee’s basic AR  
2. Helps preserve its own environments

쉽게 말하면, caller가 callee의 Activation Record를 만들기 위한 기본 준비를 하고, 동시에 자기 자신의 실행 상태도 보존하는 단계이다.

---
#### 1. Allocate Space for the Callee’s AR
먼저 callee인 `q`의 Activation Record 공간을 잡는다. 다만 callee의 AR 전체를 완전히 만드는 것은 아니고, 기본적인 부분을 준비한다고 보면 된다.

왜 local variable 공간은 제외될 수 있냐면, callee의 지역 변수 정보는 callee가 더 잘 알고 있기 때문이다. `q` 안에 어떤 local variable이 있고 얼마나 필요한지는 `q`의 코드에 해당한다. 그래서 local variable 공간 확보는 보통 callee의 prolog에서 처리된다.

----
#### 2. Evaluates each parameter & stores value or address
그다음 caller는 actual parameter를 계산한다.

예를 들어 호출이 이렇게 생겼다고 하자.

```c
q(x + 1, y * 2)
```

그러면 `x + 1`, `y * 2`를 먼저 계산해야 한다. 그 결과를 callee에게 넘긴다.

여기서 “stores value or address”가 중요하다.

Call-by-value라면 값을 저장한다.

```
a ← x + 1의 값
b ← y * 2의 값
```

Call-by-reference라면 값을 저장하는 것이 아니라 주소를 저장한다.

```
a ← x의 주소
b ← y의 주소
```

parameter passing 방식에 따라 callee의 AR에 들어가는 것이 값일 수도 있고 주소일 수도 있다.

---
#### 3. Saves return address, caller’s ARP into callee’s AR
함수를 호출하면 callee가 끝난 뒤 다시 caller로 돌아와야 한다. 그래서 return address를 callee의 AR에 저장한다.

예를 들어

```c
x = q(3, 5);
y = x + 1;
```

`q(3, 5)`가 끝나면 다시 `x = q(3, 5);`의 결과를 처리하고 다음 줄로 이어져야 한다. 이 복귀 지점이 return address이다.

그리고 caller’s ARP도 callee의 AR에 저장한다. ARP는 현재 Activation Record를 가리키는 포인터이다. `q`가 실행되는 동안에는 ARP가 `q`의 AR을 가리키지만, `q`가 끝나면 다시 `p`의 AR을 가리켜야 한다. 그래서 `q`의 AR 안에는 `p`의 ARP가 저장된다.

이 caller’s ARP는 control link처럼 생각하면 된다. 즉 caller의 실행 환경으로 돌아가기 위한 연결이다.

----
#### 4. If access links are used
Access link를 사용하는 언어라면, pre-call 단계에서 callee의 access link도 설정해야 한다.
이때 Access link는 non-local variable에 접근하기 위한 link이다.

예를 들어 nested procedure가 있다고 하자.

```c
procedure outer
    x 선언

    procedure inner
        x 사용
```

`inner`가 실행될 때 `x`는 `inner`의 local variable이 아니다. `outer`의 AR에 있다. 그래서 `inner`의 AR 안에는 `outer`의 AR을 가리키는 access link가 필요하다.

따라서 caller는 callee가 접근해야 할 lexical ancestor를 찾아서 callee의 AR에 복사해 둔다.
여기서 lexical ancestor는 소스 코드상 바깥 scope에 있는 procedure라고 생각하면 된다.

---
#### 5. Save any caller-save registers
함수 호출 중에 register 값이 바뀔 수 있다. 만약 caller가 어떤 register 값을 호출 후에도 계속 필요로 한다면, 호출 전에 저장해 두어야 한다. 이것이 caller-save register이다.

예를 들어 `p`가 어떤 값을 register에 들고 있었는데, `q`를 호출하면서 `q`가 그 register를 덮어쓸 수 있다. 그러면 `q`가 끝난 뒤 `p`는 원래 값을 잃어버린다.

그래서 caller가 책임지는 register는 pre-call sequence에서 저장한다.

슬라이드에는 “Save into space in caller’s AR”라고 되어 있다. 즉 caller-save register는 callee의 AR이 아니라 caller 자신의 AR에 저장할 수 있다.

---
#### 6. Jump to address of callee’s prolog code

마지막으로 callee의 prolog code 주소로 점프한다.
pre-call sequence가 끝나면 이제 caller의 준비는 끝났고, 실제 실행 흐름이 callee로 넘어간다.

여기까지가 pre-call sequence이다.

>Pre-call sequence는 caller가 callee 호출 전에 parameter, return address, caller ARP, access link, caller-save register 등을 준비하고 callee의 prolog로 점프하는 과정이다.

---
# Prolog Code
Prolog code는 **callee가 시작할 때 수행하는 작업**이다.

Pre-call이 caller가 하는 준비라면, prolog는 callee가 자기 실행 환경을 완성하는 단계이다.

여기서도 두 가지 역할을 수행한다.

1. Finish setting up the callee’s environment
2. Preserve parts of the caller’s environment that will be disturbed

즉 callee가 자기 body를 실행할 수 있도록 local data 공간 등을 준비하고, callee가 건드릴 수 있는 caller의 환경 일부를 보존한다.

---
#### 1. Preserve any callee-save registers
Register 저장 방식에는 caller-save와 callee-save가 있다.

- caller-save register는 caller가 호출 전에 저장한다.  
- callee-save register는 callee가 사용하기 전에 저장하고, 끝날 때 복원한다.

Prolog에서는 callee-save register를 저장한다.

예를 들어 `q`가 어떤 register를 사용해야 하는데, 호출 규칙상 그 register는 callee가 보존해야 하는 register라고 하자. 그러면 `q`는 prolog에서 그 register의 기존 값을 저장해 둔다. 나중에 epilog에서 다시 복원한다.

---
#### 2. If display is being used
Display를 사용하는 언어라면 prolog에서 display를 갱신한다.

Display는 lexical level별 Activation Record를 빠르게 찾기 위한 배열이다. 예를 들어 `display[2]`는 현재 lexical depth 2의 AR을 가리키는 식이다.

procedure가 새로 호출되면 현재 lexical level에 해당하는 display entry를 새 AR로 바꿔야 한다. 그런데 기존 display 값은 나중에 return할 때 복원해야 한다.

그래서 prolog에서 다음 작업을 한다.

1. 현재 lexical level의 기존 display entry를 새 AR 안에 저장
2. 현재 ARP를 display의 현재 lexical level entry에 저장

쉽게 말하면 기존 display 값을 백업하고,display가 현재 procedure의 AR을 가리키게 만든다.

display는 뒤쪽 슬라이드에서 더 자세히 나오지만, 여기서는 “non-local variable 접근을 빠르게 하기 위한 구조” 정도로 이해하면 된다.

---
#### 3. Allocate space for local data
callee의 local variable 공간을 확보한다.

예를 들어:

```c
int q(int a, int b) {
    int c;
    int temp;
    ...
}
```

`c`, `temp`는 `q`의 local variable이다. 이 변수들은 `q`의 AR 안에 저장되어야 한다.

Pre-call에서 callee의 기본 AR 공간은 준비되었지만, local variable 공간은 prolog에서 확보할 수 있다. 가장 단순한 방식은 AR을 확장해서 local data 영역을 만드는 것이다.

---
#### 4. Find any static data areas referenced in the callee
callee가 static data나 global data를 참조한다면, 그 위치를 확인하거나 접근 준비를 한다.

예를 들어 전역 변수나 static variable은 stack frame 안에 있지 않고 static/global data 영역에 있다. callee가 이런 데이터를 사용한다면, 해당 영역을 참조할 수 있어야 한다.

---
#### 5. Handle any local variable initializations

마지막으로 local variable 초기화를 처리한다.

예를 들어

```c
int q() {
    int x = 10;
    int y = 20;
}
```

`x`, `y`의 공간을 만드는 것만으로는 부족하고, `10`, `20`이라는 초기값을 넣어야 한다. 이런 초기화 코드가 prolog 근처에서 수행될 수 있다.

정리하면 prolog는 다음 역할이다.

1. callee-save register 저장
2. display 사용 시 display 갱신
3. local variable 공간 확보
4. static/global data 접근 준비
5. local variable 초기화

---
# Epilog Code
Epilog code는 **callee가 끝날 때 수행하는 작업**이다. Prolog가 callee의 시작 준비라면, epilog는 callee의 종료 정리이다.

1. Wind up the business of the callee
2. Start restoring the caller’s environment

즉 callee가 사용한 자원을 정리하고, caller의 환경을 복원하기 시작한다.

---
#### 1. Store return value?
return value 저장은 epilog에서 하는 것이 아니라, `return` 문을 실행할 때 이미 처리된다.

예를 들어서

```c
return c;
```

이 문장을 만나면 `c`의 값을 return value 위치나 return register에 저장한다. 그러고 나서 epilog로 넘어가서 정리 작업을 한다.

따라서 epilog의 주된 역할은 return value 계산이 아니라, **환경 정리와 복귀**이다.

---
#### 2. Restore callee-save registers
Prolog에서 callee-save register를 저장했다면, epilog에서 복원해야 한다.
이렇게 해야 caller 입장에서는 callee 호출 전후로 해당 register 값이 유지된 것처럼 보인다.

callee-save register는 다음처럼 처리된다.

- prolog: 기존 register 값 저장
- epilog: 기존 register 값 복원

---
#### 3. Free space for local data, if necessary
callee가 사용한 local data 공간을 정리한다.

대부분 stack 기반 AR에서는 함수가 return되면서 AR이 제거되므로 local variable 공간도 함께 사라진다. 하지만 local data를 heap에 따로 할당했거나, variable-length data를 별도 관리했다면 해제 작업이 필요할 수 있다.

---
#### 4. Load return address from AR
callee가 끝났으면 caller로 돌아가야 한다. 어디로 돌아갈지는 return address가 알려준다.
return address는 호출 시 callee의 AR에 저장되어 있었다. epilog에서는 이 return address를 AR에서 꺼낸다.

---
#### 5. Restore caller’s ARP
callee 실행 중에는 ARP가 callee의 AR을 가리킨다.
하지만 q가 끝나면 다시 caller인 p의 AR을 가리켜야 한다.
그래서 epilog에서 caller’s ARP를 복원한다.

---
#### 6. Jump to the return address
마지막으로 return address로 점프한다. 그러면 실행 흐름이 caller의 호출 다음 위치로 돌아간다.

한 문장으로 정리하면 Epilog code는 callee가 사용한 register와 local data를 정리하고, caller의 ARP와 return address를 이용해 caller로 돌아가는 코드이다.

---
# Post-return Sequence
Post-return sequence는 **callee가 return한 뒤 caller가 수행하는 작업**이다.
즉 callee의 epilog가 끝나고 return address로 돌아오면, caller는 아직 마무리해야 할 일이 있다.

여기서도 두 가지 역할이 나온다.

1. Finish restoring caller’s environment
2. Place any value back where it belongs

즉 caller의 환경 복원을 마무리하고, 반환값이나 parameter 결과를 적절한 위치에 넣는다.

---
#### 1. Copy return value from callee’s AR, if necessary
callee가 반환한 값이 callee의 AR에 있다면 caller가 그 값을 복사해야 한다.

예를 들어서

```c
x = q(3, 5);
```

`q`가 반환한 값을 `x`에 넣어야 한다.

다만 실제 시스템에서는 return value가 register에 들어오는 경우도 많다. 이 경우 callee의 AR에서 복사할 필요는 없고, return register의 값을 사용하면 된다.

---
#### 2. Free the callee’s AR
callee는 이미 끝났으므로 callee의 AR은 더 이상 필요 없다. 따라서 caller는 callee의 AR을 제거한다. stack에서는 보통 stack pointer를 조정해서 callee의 frame을 없앤다.

제거 후에는 다시 p의 AR만 남는다.

---
#### 3. Restore any caller-save registers
Pre-call에서 caller-save register를 저장했다면, post-return에서 복원한다.

이 흐름은 다음처럼 대응된다.

- pre-call: caller-save register 저장
- post-return: caller-save register 복원

callee-save register는 prolog/epilog에서 callee가 처리했고, caller-save register는 pre-call/post-return에서 caller가 처리한다.

---
#### 4. Restore any call-by-reference parameters to registers, if needed
Call-by-reference에서는 actual parameter의 주소가 callee에게 전달된다. callee가 그 값을 바꿨을 수 있다.

만약 caller가 해당 변수를 register에 캐싱해 두고 있었다면, memory의 값과 register 값이 달라질 수 있다. 그래서 필요하면 reference parameter와 관련된 register 값을 다시 맞춰야 한다.

또한 슬라이드에는 call-by-value/result parameter도 언급된다.

Call-by-value/result, 또는 copy-restore 방식은 호출할 때 값을 복사해서 callee에게 주고, 함수가 끝나면 callee의 formal parameter 값을 caller의 actual parameter에 다시 복사하는 방식이다.

즉 post-return에서 copy-back이 필요하다.

---
#### 5. Continue execution after the call

모든 정리가 끝나면 caller는 함수 호출 다음 명령어부터 계속 실행한다.

예를 들어서

```c
x = q(3, 5);
y = x + 1;
```

`q` 호출이 끝나고 post-return sequence까지 끝나면, 이제 `y = x + 1;`로 진행할 수 있다.

---
# Activation Record Details
Activation Record 는 실제 메모리에서 어디에 저장될까?

Activation Record는 procedure 호출 한 번에 필요한 실행 정보를 담는 구조였다. 그런데 이 AR을 항상 Stack에만 둘 수 있는 것은 아니다. 어떤 상황에서는 static 영역에 둘 수도 있고, 앞에서 언급했든 heap에서도 둘 수 있다.

### 1. Stack AR을 두는 경우
가장 일반적인 경우는 AR을 스택에 두는 경우이다. 
함수가 호출될 때 AR이 생기고, 함수가 끝날 때 AR이 사라져도 된다면 stack에 두면 된다.

예를 들어 다음 코드를 보자.

```c
int add(int a, int b) {
    int result;
    result = a + b;
    return result;
}

int main() {
    int x;
    x = add(3, 5);
}
```

`add(3, 5)`가 호출되면 `add`의 AR이 stack에 생긴다.

`add`가 끝나면 `add`의 매개변수 `a`, `b`, 지역 변수 `result`, return address 등은 더 이상 필요 없다. 그래서 `add`의 AR을 제거해도 된다.

이처럼 **호출이 시작될 때 생기고, 호출이 끝날 때 사라지는 정보**는 stack에 두는 것이 자연스럽다.

재귀 함수도 stack을 사용하면 잘 처리된다.

```c
int fact(int n) {
    if (n == 1) return 1;
    return n * fact(n - 1);
}
```

`fact(3)`이 `fact(2)`, `fact(1)`을 호출하면 stack에는 각각의 AR이 따로 쌓인다.

같은 함수라도 호출마다 `n` 값이 다르기 때문에 AR이 따로 필요하다. stack은 이런 구조와 잘 맞는다.

---
### 2. Heap AR을 두는 경우
그다음은 AR을 **heap**에 둬야 하는 경우이다.

만약에 **함수가 끝난 뒤에도 그 함수의 실행 상태가 필요하면 stack에 두면 안 된다.**
왜냐하면 stack에 있는 AR은 함수가 return하면 제거되기 때문이다.

예를 들어 어떤 언어에서 함수 안에 내부 함수를 만들고, 그 내부 함수를 밖으로 반환할 수 있다고 하자.

```c
function outer() {
    x = 10

    function inner() {
        return x
    }

    return inner
}
```

여기서 `inner`는 `outer` 안의 변수 `x`를 참조한다. 그런데 `outer`가 끝난 뒤에도 `inner`가 밖에서 호출될 수 있다면 문제가 생긴다.

`outer`의 AR을 stack에 두었다면, `outer`가 return하는 순간 AR이 사라진다. 그러면 `inner`가 나중에 `x`를 참조하려고 해도 `x`가 있던 공간이 없어져 버린다.

그래서 이런 경우에는 `outer`의 실행 상태를 heap에 둬야 한다. heap에 두면 함수가 return해도 그 데이터가 바로 사라지지 않기 때문이다.

---
### 3. Static AR을 두는 경우
아떤 procedure가 다른 함수를 호출하지 않고, 재귀도 하지 않으며, 동시에 여러 activation이 존재할 일이 없다면 AR을 static하게 둘 수 있다.

Static allocation은 AR의 위치를 실행 중에 새로 잡지 않고, 미리 고정해 두는 방식이다.

예를 들어 아주 단순한 함수가 있고, 그 함수가 동시에 여러 번 활성화될 일이 없다면, 그 함수의 지역 변수 공간을 고정된 메모리 위치에 둘 수 있다.

하지만 이 방식은 제약이 크다.

특히 재귀가 있으면 static allocation은 어렵다.

```
int fact(int n) {    if (n == 1) return 1;    return n * fact(n - 1);}
```

재귀 함수에서는 같은 procedure의 activation이 여러 개 동시에 존재한다.

```
fact(3)fact(2)fact(1)
```

이 셋은 모두 `n`이라는 지역 정보를 갖지만, 각각 값이 다르다. 만약 static 공간 하나만 사용하면 `n` 값이 서로 덮어써져서 제대로 실행할 수 없다.

그래서 static allocation은 빠르고 단순하지만, 일반적인 procedure call, 특히 recursion에는 적합하지 않다.