# 컴퓨터보안

암호학 기초, 대칭키/공개키 암호, 해시와 서명, TLS, 웹 보안, 사이버 보안을 정리한 노트입니다.

## 목차

### Foundations

- [1. Cryptography](<1. Cryptography.md>)
  - 암호학의 목적, 보안 목표, 보안 서비스와 메커니즘 같은 기초 개념을 다룹니다.
- [3. Number Theory](<3. Number Theory.md>)
  - divisibility, Euclidean algorithm, modular arithmetic 등 암호학에 필요한 정수론 기초를 다룹니다.

### Symmetric Cryptography

- [4. DES](<4. DES.md>)
  - DES의 Feistel 구조, round function, key schedule 등 고전 블록 암호의 동작 원리를 정리합니다.
- [5. AES](<5. AES.md>)
  - AES의 state 구조와 SubBytes, ShiftRows, MixColumns, AddRoundKey 과정을 중심으로 현대 블록 암호를 다룹니다.
- [6. Block Cipher Operation](<6. Block Cipher Operation.md>)
  - ECB, CBC, CFB, OFB, CTR 등 블록 암호 운영 모드와 사용 시 주의점을 정리합니다.
- [7. Random Bit Generator](<7. Random Bit Generator.md>)
  - randomness, unpredictability, TRNG, PRNG, seed 요구사항과 randomness test를 다룹니다.

### Public Key and Integrity

- [8. Public Key Cryptography](<8. Public Key Cryptography.md>)
  - 공개키 암호의 개념, 키 교환, RSA 같은 비대칭키 방식의 핵심 원리를 다룹니다.
- [9. Hash Digital Signature](<9. Hash Digital Signature.md>)
  - 해시 함수, 메시지 무결성, 디지털 서명과 인증 개념을 정리합니다.

### Network Security

- [10. SSL TLS Protocol (1)](<10. SSL TLS Protocol (1).md>)
  - SSL/TLS의 기본 구조, handshake 흐름, 인증서 기반 보안 통신을 소개합니다.
- [11. SSL TLS Protocol (2)](<11. SSL TLS Protocol (2).md>)
  - TLS 세부 동작, 키 교환, record protection 등 심화 내용을 다룹니다.

### Application and Cyber Security

- [12. Cyber Security](<12. Cyber Security.md>)
  - 사이버 보안 전반의 개념과 위협 모델, 보안 관리 관점을 정리합니다.
- [13. Web Security](<13. Web Security.md>)
  - HTTP stateless, cookie, SQL Injection, XSS, JWT, SSRF, CSRF 등 웹 보안 주제를 다룹니다.
