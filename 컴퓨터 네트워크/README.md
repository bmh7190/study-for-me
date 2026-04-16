# 컴퓨터 네트워크

네트워크 계층 구조, 주소 체계, 전달과 포워딩, IP, ARP, ICMP, 라우팅, TCP, TCP Socket을 정리한 노트입니다.

## 목차

### 7 Layers

- [2-2. The OSI Model](<7 Layers/2-2 The OSI Model.md>)
  - OSI 7계층의 역할과 hop-to-hop, source-to-destination 관점의 데이터 전달 방식을 정리합니다.
- [2-3. TCP IP Protocol suite](<7 Layers/2-3 TCP IP Protocol suite.md>)
  - TCP/IP protocol suite의 계층별 통신 방식과 encapsulation 흐름을 다룹니다.
- [2-4. Addressing](<7 Layers/2-4 Addressing.md>)
  - logical address, port number 등 계층별 주소 체계와 라우터를 지날 때 주소가 어떻게 쓰이는지 정리합니다.

### IP Address

- [5-4. Special Address](<IP Address/5-4 Special Address.md>)
  - DHCP, broadcast, loopback, private network address 등 특수 IP 주소의 용도를 다룹니다.
- [5-5. NAT](<IP Address/5-5 NAT.md>)
  - NAT가 private address와 public address를 변환하는 방식과 네트워크에서 쓰이는 이유를 정리합니다.

### Delivery and Forwarding

- [6-1. Delivery](<Delivery Forwarding/6-1 Delivery.md>)
  - direct delivery와 indirect delivery를 통해 패킷이 목적지까지 전달되는 기본 흐름을 다룹니다.
- [6-2. Forwarding](<Delivery Forwarding/6-2 Forwarding.md>)
  - routing table을 기반으로 next hop을 결정하고 packet을 forwarding하는 과정을 정리합니다.
- [6-3. MPLS](<Delivery Forwarding/6-3 MPLS.md>)
  - MPLS의 label 기반 forwarding 개념과 기존 IP forwarding과의 차이를 다룹니다.

### IP

- [7-2. Datagrams](<IP/7-2 Datagrams.md>)
  - IP datagram의 구조, TTL, multiplexing, Ethernet frame 안에서의 encapsulation을 정리합니다.
- [7-3. Fragmentation](<IP/7-3 Fragmentation.md>)
  - MTU, flag, fragment offset을 중심으로 IP datagram이 분할되고 재조립되는 방식을 다룹니다.
- [7-4. Options](<IP/7-4 Options.md>)
  - record route, source route, timestamp 등 IP option field의 종류와 목적을 정리합니다.
- [7-5. Checksum](<IP/7-5 Checksum.md>)
  - IP header checksum이 오류를 검출하는 방식과 계산 과정을 다룹니다.
- [7-8. IP Package](<IP/7-8 IP Package.md>)
  - IP 계층 구현에서 필요한 package 구조와 datagram 처리 흐름을 정리합니다.

### ARP

- [8-1. Address Mapping](<ARP/8-1 Address Mapping.md>)
  - IP 주소를 물리 주소로 매핑해야 하는 이유와 ARP의 기본 동작을 다룹니다.
- [8-3. ARP Package](<ARP/8-3 ARP Package.md>)
  - ARP cache table, input/output module, cache control module 등 ARP package 내부 구성을 정리합니다.

### ICMP

- [9-1. ICMP](<ICMP/9-1 ICMP.md>)
  - ICMP의 역할과 IP 계층에서 오류 보고와 진단 메시지를 제공하는 이유를 정리합니다.
- [9-2. Messages](<ICMP/9-2 Messages.md>)
  - destination unreachable, time exceeded, redirect, echo, timestamp, traceroute 관련 ICMP 메시지를 다룹니다.

### Unicast Routing Protocols

- [11-2. Inter And Intra Domain Routing](<Unicast Routing Protocols/11-2 Inter And Intra Domain Routing.md>)
  - inter-domain과 intra-domain routing의 차이와 대표 routing protocol의 분류를 정리합니다.
- [11-3. Distance Vector Routing](<Unicast Routing Protocols/11-3 Distance Vector Routing.md>)
  - Bellman-Ford 기반 distance vector routing, table update, instability 문제와 완화 기법을 다룹니다.
- [11-5. Link State Routing](<Unicast Routing Protocols/11-5 Link State Routing.md>)
  - Dijkstra algorithm을 기반으로 link state routing에서 shortest path tree를 구성하는 과정을 정리합니다.

### TCP

- [15-1. TCP Services](<TCP/15-1 TCP Services.md>)
  - TCP가 제공하는 connection-oriented, reliable byte stream 서비스와 송수신 buffer 개념을 다룹니다.
- [15-2. TCP Features](<TCP/15-2 TCP Features.md>)
  - TCP의 번호 체계, ACK 의미, full-duplex 통신 등 핵심 특징을 정리합니다.
- [15-3. TCP Segment](<TCP/15-3 TCP Segment.md>)
  - TCP segment header의 port, sequence number, ACK number, control field, checksum을 다룹니다.
- [15-4. A TCP Connection](<TCP/15-4 A TCP Connection.md>)
  - three-way handshake, data transfer, connection termination, half-close 흐름을 정리합니다.
- [15-5. State Transition Diagram](<TCP/15-5 State Transition Diagram.md>)
  - TCP connection이 SYN, ESTABLISHED, FIN 계열 상태를 거쳐 변하는 과정을 다룹니다.
- [15-6. Windows In TCP](<TCP/15-6 Windows In TCP.md>)
  - TCP window의 의미와 sliding window를 통해 송수신량을 제어하는 방식을 정리합니다.
- [15-7. Flow Control](<TCP/15-7 Flow Control.md>)
  - receiver window를 이용해 수신 측 buffer overflow를 막는 TCP flow control을 다룹니다.
- [15-7-1. Silly Window Syndrome](<TCP/15-7-1 Silly Window Syndrome.md>)
  - 작은 window update나 작은 segment 전송으로 효율이 떨어지는 silly window syndrome을 정리합니다.
- [15-7-2. SYN Flooding](<TCP/15-7-2 SYN Flooding.md>)
  - TCP handshake 특성을 악용한 SYN flooding 공격과 그 배경을 다룹니다.
- [15-8. Error Control](<TCP/15-8 Error Control.md>)
  - ACK 규칙, fast retransmission, ACK 유실 등 TCP의 오류 제어와 재전송 동작을 정리합니다.
- [15-9. Congestion Control](<TCP/15-9 Congestion Control.md>)
  - slow start, congestion avoidance, fairness, UDP와 TCP의 혼잡 제어 차이를 다룹니다.
- [15-10. TCP Timer](<TCP/15-10 TCP Timer.md>)
  - persistence timer, keepalive timer, RTO 계산, Karn's algorithm 등 TCP timer 동작을 정리합니다.
- [15-11. Options](<TCP/15-11 Options.md>)
  - MSS, window scale, timestamp, SACK 등 TCP option field의 종류와 역할을 다룹니다.

### TCP Socket

- [Chapter 1](<TCP Socket/Chapter 1.md>)
  - socket, bind, listen, accept, connect를 통해 TCP 서버와 클라이언트의 기본 흐름을 정리합니다.
- [Chapter 2](<TCP Socket/Chapter 2.md>)
  - protocol family와 socket type을 중심으로 소켓 생성 시 선택해야 하는 옵션을 다룹니다.
- [Chapter 3](<TCP Socket/Chapter 3.md>)
  - IPv4 주소 구조체, port number, network byte order와 host byte order의 차이를 정리합니다.
- [Chapter 4](<TCP Socket/Chapter 4.md>)
  - TCP echo server/client 예제를 통해 서버와 클라이언트의 함수 호출 순서를 다룹니다.
- [Chapter 10-1](<TCP Socket/Chapter 10-1.md>)
  - multiprocess server에서 zombie process가 생기는 이유와 wait, waitpid, signal 처리를 정리합니다.
- [Chapter 10-2](<TCP Socket/Chapter 10-2.md>)
  - process 기반 다중 접속 서버와 TCP 입출력 루틴 분할을 통해 동시 접속 처리 방식을 다룹니다.

### Appendix

- [HTTP](<부록/HTTP.md>)
  - HTTP connection, persistent HTTP, HTTP/2 등 웹 통신의 기본 동작을 정리한 부록입니다.
