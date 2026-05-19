앞에서는 SDT나 AST 같은 개념을 봤다면, 여기서는 그걸 실제로 **three-address code**로 바꾸는 방법을 본다. 즉, 문법을 분석하는 데서 끝나는 게 아니라, 컴파일러가 다음 단계에서 사용할 수 있는 중간 표현을 만드는 과정이다.

# SDT FOR IR GENERATION
여기서 핵심은 **SDT를 이용해서 IR을 생성한다**는 것이다.  
SDT는 문법 규칙에 의미 동작을 붙이는 방식인데, 이번에는 그 의미 동작이 단순 계산이 아니라 `t1 := a + b` 같은 중간 코드를 생성하는 역할을 한다.

![](../images/Pasted%20image%2020260519130622.png)

이 예시를 통해서 `E → E + E` 라는 문법 규칙이 적용되었을 때, 그 식을 three address code로 어떻게 바꾸는지를 보자

예를 들어 소스 코드에 이런 식  `a := b + c`  이 있다고 해보자 그럼 b+c 는 문법적으로 보면 `E → E + E` 규칙과 인식된다. 여기서 왼쪽 E는 b가 되고 오른쪽 E는 c가 되는 것이다. 

이때 컴파일러 그냥 "덧셈식이구나" 하고 끝나는게 아니라, 실제로 다음 단계에서 사용할 수 있는 중간 코드를 만들어야 한다. 

```
t1 := b + c
a := t1
```

이런 식으로 말이다.

### Synthesized Attribute
오른쪽에 Synthesized attribute가 정리되어 있다. `E.code` `E.addr` 이 중요하다. `E.code`는 해당 식을 계산하기 위해 필요한 three-address code다. `E.addr`은 그 식의 결과값이 저장된 위치 또는 이름을 말한다.

예를 들어 `b + x`를 계산해서 결과를 `t3`에 저장했다면 `E.addr = t3` 가 된다. 정리하면 `E.addr`는 "이 식의 결과가 어디에 들어 있냐?"를 나타내는 속성이다.


### E → E + E
빨간 박스에 있는 규칙은 E → E + E 인데, 이걸 좀 더 정확히 쓰면 보통 E → E1 + E2 이렇게 표현해서 왼쪽 피연산자와 오른쪽 피연산자를 구분한다.

예를 들어 `t1 + t2` 인 경우에는 아래와 같다.

```
E1.addr = t1
E2.addr = t2
```

이제 전체 식 `E`의 결과를 저장할 새 임시 변수 `E.addr = t3` 를 만든다.

### gen
그리고 코드를 생성한다.

```
gen(E.addr ':=' E1.addr '+' E2.addr)
gen(t3 := t1 + t2)
```

여기서 `gen`은 실제 코드를 출력하거나 중간 코드 리스트에 추가하는 함수라고 보면 된다. 쉽게 말해서 컴파일러 내부에서 `t3 := t1 + t2` 같은 three address code를 생성한다.

---
# Syntax-Directed Translation into Three-Address Code

![](../images/Pasted%20image%2020260519131848.png)
`S → id := E`이면 먼저 `E.code`를 생성하고, 그 결과 주소인 `E.addr`를 `id`에 대입한다.   예를 들어 `a := b + c`라면 먼저 `b + c`를 계산하는 코드가 나오고, 마지막에 `a := t1`이 나온다.

`E → E1 + E2`에서는 새 임시 변수 `newtemp()`를 만들고, `t := E1.addr + E2.addr` 형태의 코드를 생성한다. 곱셈도 같은 방식이고, 단항 마이너스도 `t := uminus E1.addr`처럼 처리한다.

---
# Example a = b + - c

![](../images/Pasted%20image%2020260519134835.png)

---
# Example While

![](../images/Pasted%20image%2020260519141850.png)

`while E do S1`은 단순 계산식보다 구조가 있다. 반복문의 시작 위치와 끝 위치가 필요하기 때문에 `S.begin`, `S.after` 같은 label 속성을 사용한다.

흐름은 이렇게 정리할 수 있다.

```
begin:
    E.code
    if E.addr = 0 goto after
    S1.code
    goto begin
after:
```

즉, 조건식을 먼저 계산하고, 조건이 거짓이면 반복문 뒤로 빠져나간다. 참이면 본문을 실행하고 다시 시작 label로 돌아간다.