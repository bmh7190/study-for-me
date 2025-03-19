---

---

---
### 🏛️ Instruction Set

**Instruction Set**은 컴퓨터가 수행할 수 있는 기능들의 집합이다.  
각 컴퓨터마다 고유한 **Instruction Set**을 가지지만, 기본적인 동작 원리는 유사하다.  
초기 컴퓨터들은 매우 단순한 Instruction Set을 가지고 있었으며,  
오늘날의 많은 현대 컴퓨터들도 여전히 간결한 Instruction Set을 유지하고 있다.

---

### 🔢 Arithmetic Operations

**덧셈(Add)과 뺄셈(Subtract)** 연산은 **3개의 피연산자(Operands)**를 사용한다.

- **두 개(Source)**는 연산할 값이고,
- **하나(Destination)**는 결과를 저장할 레지스터이다.

assembly

복사편집

`add a, b, c  # a ← b + c sub a, b, c  # a ← b - c`

모든 산술 연산(Arithmetic Operations)은 이와 같은 형식을 따른다.

> **📌 Design Principle 1**: 단순한 연산일수록 규칙적인 형식을 따르는 것이 좋다.

---

### 📌 Register Operands

산술 연산(Arithmetic Instructions)은 **레지스터(Register)**를 사용하여 데이터를 처리한다.  
MIPS 프로세서는 **32개의 32비트 레지스터 파일(Register File)**을 제공하며,  
이는 **자주 사용되는 데이터**를 저장하는 용도로 활용된다.

- 레지스터는 **0번부터 31번까지 번호가 부여**되며,
- 32비트 데이터 단위를 **워드(Word)**라고 부른다.

💾 **어셈블러(Assembler)에서 사용하는 레지스터 명칭**

- `$t0, $t1, ... , $t9` → **임시 값(Temporary Values) 저장**
- `$s0, $s1, ... , $s7` → **저장된 변수(Saved Variables) 저장**

> **📌 Design Principle 2**: **레지스터 크기가 작을수록 연산 속도는 더 빨라진다.**

---
### 🖥️ **Main Memory for Composite Data**

산술 연산(Arithmetic Operations)을 적용하기 위해, **메인 메모리(Main Memory)**를 사용하여 데이터를 불러오거나 저장할 수 있다.

#### **📌 메모리에서 데이터 로드 및 저장**

```assembly
lw $t0, 8($s1)  # 메모리에서 값을 불러와 $t0에 저장 (Load Word) 
sw $t0, 8($s1)  # $t0의 값을 메모리에 저장 (Store Word)`
```

- `lw (Load Word)` : 메모리에서 데이터를 불러와 레지스터에 저장
- `sw (Store Word)` : 레지스터 값을 다시 메모리에 저장

---

### 📌 **메모리 주소와 데이터 정렬**

✅ **MIPS의 메모리 특성**

- **메모리는 바이트(Byte) 단위로 주소가 할당됨**
- **각 주소(Address)는 8비트(1바이트) 데이터를 가리킴**
- **워드(Word)는 메모리에 정렬(Aligned)되어 저장됨**
- **주소는 항상 4의 배수(Multiple of 4)여야 함**

✅ **MIPS는 Big Endian 방식을 사용**

- 워드(Word) 내에서 **가장 큰 값(Most Significant Byte, MSB)**이 **가장 작은 주소(Least Address)**에 저장됨

📌 **정리하면:**

- 메모리는 **바이트 주소로 관리**되며,
- **4바이트 정렬**을 유지해야 하며,
- **MIPS는 빅 엔디언(Big Endian) 방식**을 따른다.

---
### Littele vs Big endian

Example
- vaiable x has 4-byte value of 0x01234567
- Address given by &x is 0x100

![[Pasted image 20250319102729.png]]

- **Big Endian**: 주소가 작은 곳부터 **가장 큰 값**이 저장되고, 점점 작은 값이 뒤에 배치된다.
- **Little Endian**: 주소가 작은 곳부터 **가장 작은 값**이 저장되고, 점점 큰 값이 뒤에 배치된다.


---

### Memory Operand Example 1

#### C Code

```c
g = h + A[8];
```

- `g` → `$s1`
- `h` → `$s2`
- `A` 배열의 **베이스 주소** → `$s3`

#### Compiled MIPS Code

```assembly

# 배열 A의 인덱스 8은 오프셋 32를 필요로 함 
# (4바이트 * 8 = 32) 
lw $t0, 32($s3)   # A[8]의 값을 로드하여 $t0에 저장 
add $s1, $s2, $t0 # g = h + A[8]

```

- 배열은 **4바이트(Word) 단위**로 저장되므로,  
- 인덱스 8에 해당하는 메모리 오프셋은 `8 × 4 = 32` 바이트가 됨.
- `lw` 명령어를 사용해 `A[8]`의 값을 `$t0`에 로드한 후,
- `add` 명령어로 `h`와 `A[8]`을 더해 `g`에 저장.

---
### Memory Operand Example2

#### C Code

```c
A[12] = h + A[8]
```

- `h` → `$s2`
- `A` 배열의 **베이스 주소** → `$s3`

#### Compiled MIPS Code

```assembly

# 배열 A의 인덱스 8은 오프셋 32를 필요로 함 
# (4바이트 * 8 = 32) 

lw $t0, 32($s3)   # A[8]의 값을 로드하여 $t0에 저장 
add $$t0, $s2, $t0
sw $t0, 48($s3)

```

- 배열은 **4바이트(Word) 단위**로 저장되므로,  
- 인덱스 8에 해당하는 메모리 오프셋은 `8 × 4 = 32` 바이트가 됨.
- `lw` 명령어를 사용해 `A[8]`의 값을 `$t0`에 로드한 후,
- `add` 명령어로 `h`와 `A[8]`을 더해 `$t0`에 저장.
- 인덱스 12에 해당하는 메모리 오프셋은 `12 × 4 = 48` 바이트가 됨.
- `sw` 명령어를 사용해 `A[12]`에 `$t0`를 저장.


---
