# 컴파일러

문법, LR 파싱, Syntax Directed Translation, 중간 코드 생성, 타입 검사, 활성 레코드를 정리한 노트입니다.

## 목차

### Foundations

- [0 Compiler Construction](<0 Compiler Construction.md>)
  - syntax와 grammar의 구성 요소인 terminal, non-terminal, production, start symbol을 정리합니다.

### Parsing

- [4-3-1 Bottom-up Parsing](<4-3-1 Bottom-up Parsing.md>)
  - handle, shift, reduce를 중심으로 bottom-up parsing과 shift-reduce conflict를 다룹니다.
- [4-3-2 LR(0) PARSING & SIMPLE LR](<4-3-2 LR(0) PARSING & SIMPLE LR.md>)
  - LR(0) item과 SLR parsing table 구성 흐름을 정리합니다.
- [4-4-1 More Powerful LR Parsers](<4-4-1 More Powerful LR Parsers.md>)
  - Canonical LR, LALR 같은 더 강력한 LR parser 기법을 다룹니다.
- [4-4-2 USING AMBIGUOUS GRAMMARS](<4-4-2 USING AMBIGUOUS GRAMMARS.md>)
  - ambiguous grammar를 다룰 때의 해석과 parser 설계 관점을 정리합니다.

### Syntax Directed Translation

- [5-1 Syntax Directed Translation](<5-1 Syntax Directed Translation.md>)
  - syntax directed definition과 translation의 기본 개념을 소개합니다.
- [5-2-1 Applications of SDT](<5-2-1 Applications of SDT.md>)
  - syntax directed translation이 실제 컴파일 과정에서 어떻게 활용되는지 정리합니다.
- [5-2-2 SDT Schemes](<5-2-2 SDT Schemes.md>)
  - semantic action을 grammar에 배치하는 SDT scheme 구성을 다룹니다.
- [5-2-3 Implementing L-Attributed SDD's](<5-2-3 Implementing L-Attributed SDD's.md>)
  - L-attributed SDD를 실제로 구현하는 방법을 다룹니다.

### Intermediate Code and Semantic Analysis

- [6-1-1 Intermediate Code Generation part 1](<6-1-1 Intermediate Code Generation part 1.md>)
  - 중간 코드 생성의 기본 개념과 표현 방식을 정리합니다.
- [6-1-2 Intermediate Code Generation part 1](<6-1-2 Intermediate Code Generation part 1.md>)
  - 중간 코드 생성의 추가 예시와 세부 구현 흐름을 다룹니다.
- [6-2-1 Intermediate Code Generation part 2](<6-2-1 Intermediate Code Generation part 2.md>)
  - 중간 코드 생성 심화 내용을 정리합니다.
- [6-2-2 Symbol Table](<6-2-2 Symbol Table.md>)
  - identifier 관리와 scope 처리를 위한 symbol table 구조를 다룹니다.
- [6-2-3 Apply References](<6-2-3 Apply References.md>)
  - 참조 적용과 값 전달 과정에서 필요한 의미 분석 요소를 정리합니다.
- [6-2-4 Type Checking](<6-2-4 Type Checking.md>)
  - 타입 규칙과 type checking 절차를 다룹니다.

### Run-Time Environment

- [7-2 Functions of Activation Record](<7-2 Functions of Activation Record.md>)
  - activation record의 역할과 함수 호출 시 메모리 구성을 정리합니다.
- [7-3 Handling Lexical Scope with AR](<7-3 Handling Lexical Scope with AR.md>)
  - activation record를 이용해 lexical scope를 처리하는 방식을 다룹니다.
