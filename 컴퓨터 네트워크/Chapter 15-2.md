
## 프로세스 기반 다중접속 서버 모델

![](../images/Pasted%20image%2020251017111219.png)

지금까지는 하나의 클라이언트와 연결되었기 때문에 하나의 프로세스에서 처리가 가능했다. 하지만 클라이언트가 여러개라면 부모 프로세스에서는 연결 요청을 받기만 하고, 연결을 수락하면 부모프로세스에서 소켓만 만드는게 아니라 해당 클라이언트와 소통을 할 자식 프로세스를 fork하고 거기서 해당 클라이언트 전용 소켓을 이용한다.

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

우선 accpet를 하면 연결 요청을 수락하게 되고, 소켓을 만든다. 그 다음 fork()를 통해 자식 프로세스를 만들고, 그 자식 프로세스에게 해당 클라이언트에 대한 데이터송수신을 하도록 한다. 

여기서 조금 유의할 점은 fork()하면 해당 프로세스가 가지고 잇던 값도 복사되기 때문에 서버 소켓 클라이언트 소켓 파일디스크립터 값도 복사된다.