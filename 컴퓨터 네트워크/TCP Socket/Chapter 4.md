
## TCP 서버의 기본적인 함수 호출 순서

1. socket() : 소켓 생성
    
2. bind() : 소켓 주소 할당
    
3. listen() : 연결 요청 대기 상태
    
4. accept() : 연결 허용
    
5. read() / write() : 데이터 송수신
    

### 전체 코드

```c
int main(int argc, char *argv[])
{
    int serv_sock, clnt_sock;
    char message[BUF_SIZE];
    int str_len, i;

    struct sockaddr_in serv_adr;
    struct sockaddr_in clnt_adr;
    socklen_t clnt_adr_sz;

    if(argc!=2) {
        printf("Usage : %s <port>\n", argv[0]);
        exit(1);
    }

    serv_sock=socket(PF_INET, SOCK_STREAM, 0);
    if(serv_sock==-1)
        error_handling("socket() error");

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));

    if(bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr))==-1)
        error_handling("bind() error");

    if(listen(serv_sock, 5)==-1)
        error_handling("listen() error");

    clnt_adr_sz=sizeof(clnt_adr);

    for(i=0; i<5; i++)
    {
        clnt_sock=accept(serv_sock, (struct sockaddr*)&clnt_adr, &clnt_adr_sz);
        if(clnt_sock==-1)
            error_handling("accept() error");
        else
            printf("Connected client %d \n", i+1);

        while((str_len=read(clnt_sock, message, BUF_SIZE))!=0)
            write(clnt_sock, message, str_len);

        close(clnt_sock);
    }

    close(serv_sock);
    return 0;
}
```

---

### 연결 요청 대기 상태로의 진입

```c
serv_sock=socket(PF_INET, SOCK_STREAM, 0);
if(serv_sock==-1)
    error_handling("socket() error");

memset(&serv_adr, 0, sizeof(serv_adr));
serv_adr.sin_family=AF_INET;
serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
serv_adr.sin_port=htons(atoi(argv[1]));

if(bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr))==-1)
    error_handling("bind() error");

if(listen(serv_sock, 5)==-1)
    error_handling("listen() error");
```

`socket`에는 **프로토콜 체계**와 **데이터 전달 방식(소켓 타입)**을 지정해 소켓을 생성한다. 이때 **반환값은 파일 디스크립터**다.

생성된 소켓에 서버의 주소를 설정하기 위해 `sockaddr_in` 구조체에 **주소 체계**, **32비트 IP**, **16비트 포트**를 채워 주소 구조체를 완성한다.

그다음 `bind`로 **처음 만든 소켓(파일 디스크립터)**에 **주소 구조체**를 묶어준다. 함수 원형은 `struct sockaddr*`를 받도록 되어 있어 **여러 주소 체계**를 수용하려고 설계되었으므로, `sockaddr_in`을 *_(struct sockaddr_)로 형변환**해 전달한다.

```c
int listen(int sock, int backlog);
// 성공 시 0, 실패 시 -1 반환
```

이제 소켓을 **연결 요청을 받을 수 있는 상태**로 바꿔야 한다. `listen`은 서버 소켓을 **수신 대기 상태**로 전환하고, `backlog`는 **연결 요청 대기 큐의 크기(대기 가능한 최대 요청 수)**를 의미한다.

---

### 클라이언트의 연결 요청 수락

```c
for(i=0; i<5; i++)
{
    clnt_sock=accept(serv_sock, (struct sockaddr*)&clnt_adr, &clnt_adr_sz);
    if(clnt_sock==-1)
        error_handling("accept() error");
    else
        printf("Connected client %d \n", i+1);

    while((str_len=read(clnt_sock, message, BUF_SIZE))!=0)
        write(clnt_sock, message, str_len);

    close(clnt_sock);
}
```

이 예제는 **최대 5명의 클라이언트**와 차례대로 연결을 수락하고 통신한 뒤 종료한다.

```c
int accept(int sock, struct sockaddr* addr, socklen_t* addrlen);
// 성공 시 "새로 생성된" 소켓의 파일 디스크립터, 실패 시 -1
```

`accept`는 실제로 **연결을 수락**하면서 **데이터 송수신에 사용할 새로운 소켓**을 생성해 그 **파일 디스크립터**를 반환한다. 첫 번째 인자로는 **리스닝 소켓의 fd**를, 두 번째·세 번째 인자로는 **클라이언트 주소를 받기 위한 버퍼와 길이**를 넘긴다. (클라이언트 주소가 필요 없으면 `NULL`을 줄 수도 있다.)

---

## TCP 클라이언트의 기본적인 함수 호출 순서

1. socket()
    
2. connect()
    
3. read()/write()
    
4. close()
    

### 전체 코드

```c
int main(int argc, char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len;
    struct sockaddr_in serv_adr;

    if(argc!=3) {
        printf("Usage : %s <IP> <port>\n", argv[0]);
        exit(1);
    }

    sock=socket(PF_INET, SOCK_STREAM, 0);
    if(sock==-1)
        error_handling("socket() error");

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_adr.sin_port=htons(atoi(argv[2]));

    if(connect(sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr))==-1)
        error_handling("connect() error!");
    else
        puts("Connected...........");

    while(1)
    {
        fputs("Input message(Q to quit): ", stdout);
        fgets(message, BUF_SIZE, stdin);

        if(!strcmp(message,"q\n") || !strcmp(message,"Q\n"))
            break;

        write(sock, message, strlen(message));
        str_len=read(sock, message, BUF_SIZE-1);
        message[str_len]=0;
        printf("Message from server: %s", message);
    }

    close(sock);
    return 0;
}
```

---

클라이언트는 상대적으로 간단하다. **프로토콜 체계와 소켓 타입**을 지정해 소켓을 만들고, 그 소켓은 **연결 요청을 보낼 수 있는 상태**다.

연결 요청에는 `connect`를 사용한다.

```c
serv_adr.sin_family=AF_INET;
serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
serv_adr.sin_port=htons(atoi(argv[2]));

if(connect(sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr))==-1)
    error_handling("connect() error!");
else
    puts("Connected...........");
```

클라이언트는 보통 `bind`를 하지 않는다. **연결 시 커널이 자동으로 적절한 로컬 IP와 (에페메럴) 포트를 할당**해 주기 때문이다. `connect`에는 **연결할 소켓 fd**, **서버의 주소/포트**, **주소 길이**를 넘긴다. **성공하면 해당 소켓**은 바로 **송수신 가능**한 상태가 된다. 이후 `read`/`write`로 데이터를 주고받는다.

---

### (참고) 에코 클라이언트의 `read()` 처리

현재 에코 클라이언트는 `read()` 한 번으로 서버 메시지를 **전부** 받는 것처럼 구현되어 있지만, **TCP는 스트림**이므로 **데이터 경계가 없다**. 즉, 한 번에 다 올 수도 있고, 여러 번 나눠 올 수도 있다. 실제로는 **프로토콜(구분 방식)**을 정해 처리해야 한다. 예를 들어:

- **길이 프리픽스 방식**: 앞에 길이를 고정 크기로 붙이고, 그 길이만큼 반복해서 `read()`
    
- **구분자(델리미터) 방식**: `\n` 등 구분자를 만날 때까지 누적
    
- **고정 길이 레코드**: 메시지 길이가 항상 동일
    

이 중 하나로 **수신 루프**를 구성해야 안전하다.

---

필요하면 위 코드들을 **비차단 I/O**(논블로킹)나 `select/poll/epoll` 기반으로 발전시키는 버전도 정리해줄게.