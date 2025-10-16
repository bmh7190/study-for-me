## 인터넷 주소(Internet Address)

인터넷 주소는 4byte 주소 체계인 ipv4와 16yte 주소체계인 ipv6가 존재한다. 소켓을 생성할 때 protocol family에 PF_INET, PF_INET6으로 지정한게 그것이다.


## 포트 번호
ip는 컴퓨터를 구분하는 용도로 사용되며,PORT 번호는 소켓을 구분하는 용도로 사용된다. 한아ㅢ 프로그램 내에서는 둘 이사응 ㅣ소켓이 존재할 수 있으므로, 둘 이상의 PORT가 하나의프로그램에 의해 할당될 수 있다. 

PORT 번호는 16비트로 0이상 65535 이하로 표현할 있고 0 ~1023은 잘 알려진 PORT라 해서 이미 용도가 정해져있다.

---
## IPv4 기반의 주소 표현을 위한 구조체 

```c
struct sockaddr_in
{
	sa_family_t sin_family //주소체계
	uint16_t sin_port; //port번호
	struct in_addr; sin_addr; //32비트 IP 주소 체계
	char sin_zero[8]; // 사용되지 않음
}
```

- sin family : 주소 체계 정보 저장 AF_INET / AF_INET6
- sin_port: 16비트 PORT 번호 저장 네트워크 바이트순서로 저장
- sin_addr: 32비트 IP 주소 정보로 저장 / 네트워크 바이트 수넛로 저장

```c
struct in_addr
{
	in_addr_t s_addr;
}
```


----
예시)

```c
struct sockaddr_in serv_addr;
bind(serv_sock, (struct sockarr *) &serv_addr, sizeof(serv_addr));
```

bind 매개 변수가 sockaddr 타입이기 때문에 sockaddr_in을 scokaddr로 강제 형변환 해야한다.

```c
struct sockaddr
{
	sa_family_t sin_family;
	char sa_data[14];
}
```
sockaddr은 다양한 주소체계의 주소 정보를 담을 수 있도록 정의 되었다. 그래서 ipv4의 주소 체계를 담기가 불편해서 동일한 바이트 열을 구성하는 구조체 sockaddr_in을 통해 쉽게 ipv4의 주소 정보를 담을 수 있도록 한 것!

---
## 네트워크 바이트 VS 호스트 바이트 순서

네트워크는 바이트는 빅엔디안으로 되어있고, 호스트 바이트는 스몰엔디안 방식으로 작성됨 그래서 형변환이 반드시 필요함


```c
unsigned short hton(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```

여기서ㅓ h는 호스트 n은 네트워크 s는 리턴 값이 short, l은 리턴 값이 long

