
## 프로세스 기반 다중접속 서버 모델

지금까지는 하나의 클라이언트와만 연결되는 서버를 다루었기 때문에, **하나의 프로세스에서 모든 처리가 가능**했다. 하지만 실제 서버 환경에서는 **여러 클라이언트가 동시에 접속**하기 때문에, 부모 프로세스는 **연결 요청을 받기만 하고**, 연결을 수락할 때마다 **자식 프로세스를 생성(fork)** 하여 해당 클라이언트와의 통신을 담당하도록 해야 한다.

![](Pasted%20image%2020251017111219.png)



```c
while(1)
{
	adr_sz=sizeof(clnt_adr);
	clnt_sock=accept(serv_sock, (struct sockaddr*)&clnt_adr, &adr_sz);
	if(clnt_sock==-1)
		continue;
	else
		puts("new client connected...");
	pid=fork();
	if(pid==-1)
	{
		close(clnt_sock);
		continue;
	}
	if(pid==0)
	{
		close(serv_sock);
		while((str_len=read(clnt_sock, buf, BUF_SIZE))!=0)
			write(clnt_sock, buf, str_len);
		
		close(clnt_sock);
		puts("client disconnected...");
		return 0;
	}
	else
		close(clnt_sock);
}
close(serv_sock);
```

### 동작 설명

1. `accept()`를 호출하면 클라이언트의 연결 요청을 수락하고,  
    해당 클라이언트와의 통신을 위한 **새로운 소켓을 생성**한다.
    
2. 이후 `fork()`를 호출하여 **자식 프로세스를 생성**한다.
    
    - 부모 프로세스는 다음 클라이언트의 연결 요청을 기다림 (`accept()` 반복)
        
    - 자식 프로세스는 현재 연결된 클라이언트와의 통신을 담당함
        
3. 즉, **부모는 연결 담당**, **자식은 송수신 담당**으로 역할이 나뉜다.

여기서 주의해야 할 점은 `fork()`를 호출하면 **프로세스의 주소 공간 전체가 복사**되기 때문에, 파일 디스크립터(File Descriptor, FD)도 복사된다. 

즉, 부모와 자식은 **같은 소켓을 가리키는 FD를 각각 하나씩 갖게 된다.**

![](Pasted%20image%2020251017130527.png)

`close()`는 단순히 “소켓을 삭제한다”는 뜻이 아니다. 
운영체제는 각 소켓에 대해 **참조 카운트(reference count)** 를 유지한다.

- `fork()` 전: count = 1
    
- `fork()` 후: count = 2 (부모, 자식 각각이 같은 소켓을 참조)
    

`close()`가 호출될 때마다 이 count가 1 감소하고, **count가 0이 되었을 때** 커널이 실제로 소켓을 완전히 삭제한다.

### 올바른 소켓 닫기 흐름

- **자식 프로세스:**  
    부모의 서버 소켓(`serv_sock`)은 필요 없으므로 `close(serv_sock)`을 호출한다.
    
- **부모 프로세스:**  
    `accept()`로 새로 생성된 클라이언트 소켓(`clnt_sock`)은 자식이 처리하므로  
    부모에서는 `close(clnt_sock)`을 호출해 FD 참조를 줄인다.
    

이렇게 해야 각 프로세스가 담당하지 않는 소켓의 FD를 정리할 수 있고, 최종적으로 count가 0이 되어 소켓이 완전히 닫힌다.

---
## TCP의 입출력 루틴 분할

TCP 소켓은 **양방향 통신(Full-Duplex)** 이 가능하다.  

즉, 하나의 소켓으로 **데이터를 보내고(receive/write)** **받는(send/read)** 일이 동시에 일어날 수 있다.

하지만 지금까지의 에코 서버에서는 `read()`와 `write()`를 **순차적으로 수행**했기 때문에, 입력과 출력을 동시에 처리할 수는 없었다.

이 문제는 **멀티프로세스 구조**를 사용하면 간단히 해결된다.  

`fork()`를 이용해 **부모와 자식 프로세스가 각각 읽기와 쓰기를 담당**하도록 분리할 수 있다.

```c
if(connect(sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr))==-1)
	error_handling("connect() error!");

pid=fork();
if(pid==0)
	write_routine(sock, buf);
else 
	read_routine(sock, buf);

close(sock);
return 0;
```

### 동작 설명

1. **클라이언트가 서버와 연결**을 맺는다.  
    → `connect()`가 성공하면 이제 소켓은 송수신이 모두 가능한 상태다.
    
2. **`fork()`를 호출**해 프로세스를 둘로 나눈다.
    
    - **자식 프로세스:** 사용자 입력을 받아 서버로 전송 (`write_routine`)
        
    - **부모 프로세스:** 서버로부터 오는 데이터를 수신 (`read_routine`)
        
3. 소켓을 두 프로세스가 공유하면서 **동시에 읽기/쓰기 작업을 수행할 수 있게 된다.**
    
4. 양쪽 프로세스는 같은 소켓 FD를 복사해서 쓰기 때문에,  
    네트워크 연결은 하나지만, 실제 동작은 **비동기적 I/O처럼** 병렬로 이루어진다.