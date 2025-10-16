
## 프로토콜 체계 (Protocol Family)

프로토콜은 그 종류에 따라 여러 부류로 나뉘며, 이러한 부류를 **프로토콜 체계(Protocol Family)** 라고 한다.

- PF_INET
- PF_INET6
- PF_LOCAL
- PF_PACKET
- PF_IPX

---
## 소켓의 타입

소켓을 생성할 때는 **데이터 전송 방식**에 따라 소켓의 타입도 함께 결정된다. 이 중에서 프로토콜 체계 `PF_INET`(IPv4)에서 자주 사용하는 대표적인 두 가지 소켓 타입을 살펴보자.

#### 연결 지향형 소켓 (SOCK_STREAM)

`SOCK_STREAM`은 **TCP 소켓**을 의미한다.  
데이터가 중간에 손실되지 않으며, **전송한 순서대로 수신**된다.  
또한 **데이터의 경계가 존재하지 않으며**,  
소켓 간 연결은 **항상 1대 1 구조**를 가진다.



#### 비연결 지향형 소켓 (SOCK_DGRAM)

`SOCK_DGRAM`은 **UDP에서 사용되는 소켓**으로, 데이터 손실이나 파손이 발생할 수 있다.  
하지만 연결 과정을 거치지 않기 때문에 속도가 빠르며, 간단한 데이터 송수신에 자주 사용된다.


```c
int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
```

위 코드는 **PF_INET 프로토콜 체계**에서  
**SOCK_STREAM 방식**을 사용하는, 즉 **TCP 소켓**을 생성한다는 의미이다.

사실 `SOCK_STREAM` 타입을 지정하면 이미 TCP가 결정되기 때문에,  
세 번째 인자에는 `0`을 전달해도 동일하게 동작한다.


```c
int udp_socket = socket(PF_INET, SOCK_DGRAM, IPPROTO_UDP);
```

이 코드는 **PF_INET 프로토콜 체계**에서  
**SOCK_DGRAM 방식**, 즉 **UDP 프로토콜**을 사용하는 소켓을 생성한다.
