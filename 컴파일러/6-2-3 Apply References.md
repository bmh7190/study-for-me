앞에서는 `id`를 만나면 symbol table에서 찾아서 전역 변수면 이름으로 접근하고, 지역 변수면 `fp[offset]`으로 접근한다고 했다.

그런데 배열은 조금 다르다. 배열 이름 하나만 찾는 게 아니라, `A[i]`, `A[i1, i2]`처럼 **인덱스를 이용해서 실제 원소의 위치를 계산**해야 한다.

그래서 이 부분은 symbol table에 저장된 배열의 정보를 이용해서 배열 원소의 주소를 어떻게 계산하는지 설명하는 내용이다.

---
# Addressing Array Elements: One-Dimensional Arrays

예시가 다음과 같다.

```text
A : array [10..20] of integer;
```

이 말은 배열 `A`의 index가 10부터 20까지라는 뜻이다. 그리고 `integer`의 크기를 4바이트라고 하면, low = 10, w = 4 이다.

배열 원소 `A[i]`의 주소는 다음처럼 계산된다.

```text
A[i] = baseA + (i - low) * w
```

의미는 간단하다.

- `baseA`는 배열 `A`의 시작 주소다.  
- `low`는 배열 index가 시작하는 값
- `i - low`는 시작 index로부터 몇 번째 원소인지 나타낸다.  
- `w`는 원소 하나의 크기다.

예를 들어 `A[10]`은 첫 번째 원소이므로,

```text
baseA + (10 - 10) * 4 = baseA
```

가 된다.

`A[11]`은 두 번째 원소이므로,

```text
baseA + (11 - 10) * 4 = baseA + 4
```

가 된다.

즉, 배열은 index 값이 바로 offset이 아니라, **시작 index를 빼고 원소 크기를 곱해야 실제 offset이 나온다.**

---
## c를 미리 계산하는 이유

![](../images/Pasted%20image%2020260603093318.png)

예시에서는 원래 식을 아래와 같이 바꾼다.

```text
baseA + (i - low) * w
= i * w + c
```

여기서 `c = baseA - low * w` 이다.

즉,

```text
baseA + (i - low) * w
= baseA + i*w - low*w
= i*w + (baseA - low*w)
```

이렇게 정리한 것이다.

왜 이렇게 하냐면 `low`와 `w`는 컴파일 시간에 이미 아는 값인 경우가 많기 때문이다.  
예를 들어 여기서는 `low = 10`, `w = 4`다.

그래서 `c = baseA - 10 * 4` 를 미리 계산해 둘 수 있다.
그러면 실행 중에는 `i * 4 + c`만 계산하면 된다.

---
## Three-address code로 바꾸면

그 다음에 코드로 직접 바꾼 것을 보자

```text
t1 := c
t2 := i * 4
t3 := t1[t2]
... := t3
```

여기서 핵심은 `i * 4`를 계산해서 배열 원소의 offset을 만들고, `c`를 기준으로 실제 원소를 접근한다는 것이다.

조금 더 직관적으로 쓰면 이런 느낌이다.

```text
t2 := i * 4
t3 := A[t2 + c]
```

즉, `A[i]`는 단순히 `A`와 `i`를 붙여서 접근하는 게 아니라, 실제 메모리 주소 계산으로 바뀐다.

여기서 핵심 내용은 이미 알고 있는 내용을 잘 활용해서 실제 계산할 때 효율적으로 할 수 있다는 것이다.

---
# Multi-Dimensional Arrays

다음은 2차원 배열이다.

![](../images/Pasted%20image%2020260603093605.png)

```text
A : array [1..2, 1..3] of integer;
```

여기서 정보는 다음과 같다.

- low1 = 1
- low2 = 1
- n1 = 2 // 첫 번째 차원의 크기  
- n2 = 3 // 두 번째 차원의 크기  
- w = 4

즉 배열 원소는 총 2 × 3개다.

```text
A[1,1] A[1,2] A[1,3]
A[2,1] A[2,2] A[2,3]
```

그런데 메모리는 1차원으로 이어져 있다. 그래서 2차원 배열을 메모리에 어떤 순서로 저장할지 정해야 한다.

---
## Row-major와 Column-major

![](../images/Pasted%20image%2020260603093745.png)
### Row-major

Row-major는 행을 먼저 채우는 방식이다.

```text
A[1,1]
A[1,2]
A[1,3]
A[2,1]
A[2,2]
A[2,3]
```

C 언어 계열은 보통 row-major를 사용한다.

### Column-major
Column-major는 열을 먼저 채우는 방식이다.

```text
A[1,1]
A[2,1]
A[1,2]
A[2,2]
A[1,3]
A[2,3]
```

Fortran 계열에서 주로 사용하는 방식이다. 즉, 같은 `A[2,3]`이라도 row-major인지 column-major인지에 따라 메모리에서 계산되는 offset 공식이 달라진다.

---
## Row-major에서 2차원 배열 주소 계산

row-major 기준으로 예시를 보자

![](../images/Pasted%20image%2020260603093827.png)

```text
A : array [1..2, 1..3] of integer;
```

원소 `A[i1, i2]`의 주소는 다음과 같다.

```text
baseA + ((i1 - low1) * n2 + i2 - low2) * w
```

이 식의 의미는 이렇다.

```text
(i1 - low1) * n2
```

현재 행 앞에 몇 개의 원소가 있었는지 계산한다.

예를 들어 `A[2,1]`이면 첫 번째 행 `A[1,1]`, `A[1,2]`, `A[1,3]`이 앞에 있다.  
한 행에 원소가 `n2 = 3`개 있으므로, 두 번째 행으로 가려면 3개를 건너뛰어야 한다.

그다음

```text
i2 - low2
```

현재 행 안에서 몇 번째 열인지 계산한다.

그래서 전체 원소 번호는

```text
(i1 - low1) * n2 + (i2 - low2)
```

가 된다.

마지막으로 원소 하나의 크기 `w`를 곱하면 byte offset이 된다.

---

이 식을 다시 정리해보자

```text
baseA + ((i1 - low1) * n2 + i2 - low2) * w
= ((i1 * n2) + i2) * w + c
```

여기서

```text
c = baseA - ((low1 * n2) + low2) * w
```

이다.

예시 값은 다음과 같다.

```text
low1 = 1
low2 = 1
n2 = 3
w = 4
```

따라서

```text
c = baseA - ((1 * 3) + 1) * 4
  = baseA - 16
```

이 된다.

이렇게 상수 부분을 `c`로 미리 계산해두면, 실행 중에는 `i1`, `i2`에 대한 계산만 하면 된다.

---
## Three-address code

코드는 다음과 같다.

```text
t1 := i1 * 3
t1 := t1 + i2
t2 := c
t3 := t1 * 4
t4 := t2[t3]
... := t4
```

의미는 다음 순서다.

먼저 row-major 기준으로 몇 번째 원소인지 계산한다.

```text
t1 := i1 * 3
t1 := t1 + i2
```

그다음 byte offset을 만든다.

```text
t3 := t1 * 4
```

마지막으로 미리 계산된 기준값 `c`에서 offset만큼 떨어진 위치의 값을 가져온다.

```text
t4 := t2[t3]
```

즉, `A[i1, i2]`는 실제로 이런 계산을 거쳐 메모리 위치로 바뀐다.

---
# Multi-Dimensional Arrays in C/JAVA

앞에서 본 **2차원 배열 주소 계산식**을 C/JAVA 스타일로 정리해보자

![](../images/Pasted%20image%2020260603094132.png)

앞에서는 일반적인 배열처럼 lower bound가 있을 수 있다고 봤다.

```text
A : array [1..2, 1..3] of integer
```

그래서 `low1`, `low2`를 빼는 식이 들어갔다.

그런데 C/JAVA에서는 배열 index가 기본적으로 0부터 시작한다.

```c
A[0][0], A[0][1], A[0][2], ...
```

그래서 `low = 0`이라고 볼 수 있고, 식에서 `low`를 따로 빼지 않아도 된다.

---
## 1. 2차원 배열 주소 계산

```text
A[i1, i2]
= baseA + i1 * w1 + i2 * w2
```

- 여기서 `baseA`는 배열 시작 주소다.
- `i1`, `i2`는 각각 첫 번째 차원, 두 번째 차원의 index다.
- `w1`, `w2`는 각 index가 1 증가할 때 실제 주소가 얼마나 이동하는지를 의미한다.

2차원 배열이 row-major 방식이면, 한 행 전체를 지나야 다음 행으로 넘어간다. 그래서 `i1`이 1 증가하면 한 행의 크기만큼 이동한다.

예를 들어 `A[2][3]` 에서 첫 번째 index `2`는 “2번째 행으로 이동”이라는 뜻이고, 두 번째 index `3`은 “그 행 안에서 3번째 열로 이동”이라는 뜻이다.

---
## 2. row-major에서는 이렇게 정리된다

C/JAVA는 row-major 방식으로 저장한다고 보면 된다.

2차원 배열에서 한 행에 원소가 `n2`개 있고, 원소 하나의 크기가 `w`라면,

```text
w1 = n2 * w
w2 = w
```

이다.

그래서 주소식은 이렇게 바뀐다.

```text
baseA + i1 * w1 + i2 * w2
= baseA + i1 * (n2 * w) + i2 * w
= baseA + (i1 * n2 + i2) * w
```

즉, 핵심은 이 식이다.

```text
A[i1, i2] = baseA + (i1 * n2 + i2) * w
```

여기서 `i1 * n2 + i2`는 `A[i1][i2]`가 전체 배열을 1차원으로 펼쳤을 때 몇 번째 원소인지 계산하는 부분이다.

예를 들어 `n2 = 3`이면 한 행에 원소가 3개다.

```text
A[0][0] → 0 * 3 + 0 = 0번째
A[0][1] → 0 * 3 + 1 = 1번째
A[0][2] → 0 * 3 + 2 = 2번째
A[1][0] → 1 * 3 + 0 = 3번째
A[1][1] → 1 * 3 + 1 = 4번째
```

그다음 원소 하나가 4바이트라면, 여기에 `4`를 곱해서 byte offset을 만든다.

---
## 3. k차원 배열로 일반화

k차원 식은 2차원 배열을 더 일반화한 것이다.

```text
baseA + i1*w1 + i2*w2 + i3*w3 + ... + ik*wk
```

여기서 각 `wi`는 해당 index가 1 증가할 때 이동해야 하는 byte 수다.

row-major 방식에서는 뒤쪽 차원의 크기를 계속 곱하면서 계산한다.

```text
baseA + (...((i1*n2 + i2)*n3 + i3) ... * nk + ik) * w
```

이 식은 어렵게 보이지만 의미는 단순하다.

다차원 배열을 메모리에 저장할 때는 결국 1차원으로 쭉 펼쳐야 한다. 그래서 `i1, i2, ..., ik`를 이용해 “전체에서 몇 번째 원소인지”를 계산한 다음, 원소 크기 `w`를 곱한다.

---
## 4. three-address code

아래에는 `n2 = 3`인 2차원 배열 예시가 나온다.

```text
t1 := i1 * 3
t1 := t1 + i2
t2 := baseA
t3 := t1 * 4
t4 := t2[t3]
... := t4
```

이 코드는 `A[i1, i2]` 값을 읽는 과정이다.

먼저 전체 배열에서 몇 번째 원소인지 계산한다.

```text
t1 := i1 * 3
t1 := t1 + i2
```

여기서 `3`은 한 행의 원소 개수, 즉 `n2`다.

그다음 byte offset을 계산한다.

```text
t3 := t1 * 4
```

여기서 `4`는 integer 하나의 width다.

마지막으로 배열 시작 주소 `baseA`에서 offset `t3`만큼 떨어진 위치를 읽는다.

```text
t4 := t2[t3]
```

즉, `t4`에는 `A[i1, i2]`의 값이 들어간다.

---
# L → L[E] | id[E]  정리

여기서는 **배열 원소 접근을 three-address code로 어떻게 바꾸는지**를 설명한다.

앞에서 배열 주소 계산을 배웠다.

```text
A[i1, i2] = baseA + (i1 * n2 + i2) * w
```

이제 그 계산을 문법 규칙 안에서 실제로 수행하는 방법을 보는 것이다.  
즉, `A[i]`, `A[i][j]`, `A[i][j][k]` 같은 배열 접근을 처리한다.

## 1. L의 의미

여기서 `L`은 배열의 위치, 즉 **array element location**을 의미한다고 보면 된다.

```text
L → id [ E ]
L → L1 [ E ]
```

`L → id [ E ]`는 배열 접근이 처음 시작되는 경우다.

```text
A[i]
```

`L → L1 [ E ]`는 배열 접근이 이어지는 경우다.

```text
A[i][j]
A[i][j][k]
```

즉, `A[i][j]`는 한 번에 처리하는 게 아니라, `A[i]` 처리 하고 그 결과에 `[j]` 추가 처리하는 식으로 차원별로 처리한다.

---
## 2. L이 들고 다니는 정보
오른쪽에 나온 synthesized attribute들은 배열 주소 계산에 필요한 정보다.

![](../images/Pasted%20image%2020260603094942.png)

각각의 의미는 이렇다.

- L.array → 배열 이름에 대한 symbol table entry
- L.array.base → 배열의 시작 주소
- L.type → 현재 차원 이후의 타입
- L.addr → 지금까지 계산한 offset

예를 들어 `A[i][j]`에서 `A`는 symbol table에서 찾아야 한다.

```text
L.array = top.get(A)
```

그리고 `A`의 시작 주소가 필요하다.

```text
L.array.base = baseA
```

또한 현재 index를 처리할 때, 그 index에 곱해야 하는 width도 알아야 한다. 이때 `L.type.width`를 사용한다.

---
## 3. 배열 원소에 값을 저장하는 경우

먼저 일반 대입문은 이렇게 처리한다.

```text
S → id = E
{
  gen(top.get(id.lexeme) '=' E.addr);
}
```

예를 들어 `x = a + b` 이면 `x`를 symbol table에서 찾고, 오른쪽 표현식 결과 `E.addr`을 `x`에 저장한다.

배열 원소에 저장하는 경우는 다르다.

```text
S → L = E
{
  gen(L.array.base '[' L.addr ']' '=' E.addr);
}
```

예를 들어 `A[i] = x` 이면 `A[i]`의 주소를 계산한 뒤, 그 위치에 `x` 값을 저장한다.

```text
baseA[L.addr] = x
```

여기서 `L.addr`은 `i * width` 같은 계산을 통해 만들어진 offset이다.

---

## 4. 배열 원소를 읽는 경우

배열 원소가 표현식으로 사용될 수도 있다.

```text
E → L
{
  E.addr = new Temp();
  gen(E.addr '=' L.array.base '[' L.addr ']');
}
```

예를 들어 `x = A[i]` 이면 `A[i]` 값을 바로 `x`에 넣는 게 아니라, 먼저 임시 변수에 읽어온다.

```text
t1 = baseA[L.addr]
x = t1
```

즉, `L`이 왼쪽에 있으면 저장할 위치이고, 오른쪽 표현식에 있으면 값을 읽어올 위치다.

---
## 5. 첫 번째 index 처리: L → id [ E ]

가장 기본이 되는 규칙이다.

```text
L → id [ E ]
{
  L.array = top.get(id.lexeme);
  L.type = L.array.type.elem;
  L.addr = new Temp();
  gen(L.addr '=' E.addr '*' L.type.width);
}
```

예를 들어 `A[i] 를 처리한다고 하자.

먼저 배열 이름 `A`를 symbol table에서 찾는다.

```text
L.array = top.get(A)
```

그다음 `A`의 원소 타입을 가져온다.

```text
L.type = L.array.type.elem
```

예를 들어 `A`가 다음 타입이라고 하자.

```text
A : array [10] of integer
```

그러면 `A`의 원소 타입은 `integer`다.

```text
L.type = integer
L.type.width = 4
```

그래서 offset은 다음처럼 계산된다.

```text
L.addr = i * 4
```

즉, `A[i]`에서 `i`번째 원소로 가려면 원소 하나의 크기만큼 곱해야 한다.

---
## 6. 다음 index 처리: L → L1 [ E ]

2차원 이상에서는 이 규칙을 쓴다.

```text
L → L1 [ E ]
{
  L.array = L1.array;
  L.type = L1.type.elem;
  t = new Temp();
  L.addr = new Temp();
  gen(t '=' E.addr '*' L.type.width);
  gen(L.addr '=' L1.addr '+' t);
}
```

예를 들어 `A[i][j]` 를 보자.

먼저 `A[i]`까지 처리하면, 첫 번째 index에 대한 offset이 `L1.addr`에 들어 있다.
그다음 `[j]`를 처리하면서 두 번째 index에 대한 offset을 계산한다.

```text
t = j * L.type.width
```

그리고 이전 offset에 더한다.

```text
L.addr = L1.addr + t
```

즉, 배열 접근이 여러 차원으로 이어지면 offset을 계속 누적한다.

```text
첫 번째 index offset + 두 번째 index offset + ...
```

---
## 7. 예시: `A[i][j]`

배열이 다음과 같다고 하자.

```text
A : array [2] of array [3] of integer
```

이 타입은 2차원 배열처럼 볼 수 있다.

```text
A[i][j]
```

를 처리하면 먼저 `A[i]`에서 `i`에 곱해야 하는 값은 `array [3] of integer`의 width다.

```text
array [3] of integer의 width = 3 * 4 = 12
```

그래서 첫 번째 index는 다음처럼 계산된다.

```text
t1 = i * 12
```

그다음 `j`는 integer 하나의 width를 곱한다.

```text
t2 = j * 4
```

그리고 둘을 더한다.

```text
t3 = t1 + t2
```

최종적으로 `t3`가 `A[i][j]`의 offset이 된다.

값을 읽는다면, `t4 = baseA[t3]` 값을 저장한다면, `baseA[t3] = E.addr` 가 된다.

---
