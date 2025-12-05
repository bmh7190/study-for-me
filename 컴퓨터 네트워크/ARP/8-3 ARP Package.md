ARP가 실제로 어떻게 구현되었는지 살펴보자.

![](../../images/Pasted%20image%2020251205133449.png)

ARP 는 3개의 module로 구성되어 있는데 Output module input module cache control modul 그리고 Cache table과 Queue 도 포함한다. 여기서 Cache table이 지금까지 말햇던 ARP 테이블을 말한다. 각 모듈을 쉽게 설명하면 Output 모듈은 MAC 주소가 나각 모듈에서 어떤 일을 하는지 pseudo 코드를 통해 알아봐자


