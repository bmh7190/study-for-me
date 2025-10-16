소켓은 `socket` 함수를 호출함으로써 생성된다.

데이터를 **보내는 소켓**과 **받는 소켓**이 구분되어 있다.

```c
int socket(int domain, int type, int protocol);
```

소켓을 생성하면 처음에는 아무런 정보가 없는 상태이다. 따라서 IP 주소와 포트 번호를 설정해주어야 한다. `bind` 함수를 사용하면 소켓에 주소 정보를 부여할 수 있다.

```c
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
```

이렇게 만들어진 소켓은 **연결 요청을 보내기만 가능한 상태**이다.  
따라서 연결 요청을 **수락 가능한 상태**로 변경해야 한다.

```c
int listen(int sockfd, int backlog);
```

이제 소켓은 연결 요청을 **수락할 수 있는 상태**가 되었다.  
즉, “요청을 받을 준비”가 된 것이지 실제로 요청을 수락한 것은 아니다.  
연결 요청이 들어오면, **요청을 수락하는 함수**를 통해 연결을 수락할 수 있다.

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

`accept` 함수를 통해 연결 요청을 수락하면, 서버는 기존의 서버 소켓을 사용하지 않고,  
**실제 데이터 송수신에 사용할 새로운 소켓**을 생성하여 반환한다.

---

정리하자면,

1. 소켓을 생성(`socket`)
    
2. IP와 포트 번호를 할당(`bind`)
    
3. 연결 요청을 받을 수 있도록 설정(`listen`)
    
4. 요청이 들어오면 연결을 수락(`accept`)
    

연결이 수락되면, 해당 클라이언트와의 데이터 송수신을 위한 **전용 소켓**이 새로 생성된다.

---

클라이언트 쪽에서 **연결 요청을 보내는 소켓을 만드는 과정**은 훨씬 단순하다.  
소켓을 만들고(`socket`) 바로 연결 요청(`connect`)을 보내면 된다.

```c
int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen);
```

