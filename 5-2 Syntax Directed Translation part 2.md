
SDT schemes

parse tree 를 만들면서 언제 semantic rule을 실행해야하는가

![](../images/Pasted%20image%2020260507103648.png)

top down parsing 에서는 쉽게 할 수 있음
bottom up parsing은 조금 까다로움

SDT schemes를 만든 이유는 parse tree를 만드는 과정에서 같이 하기 위함

---
# Eliminating Left Recursions from SDT

실제로 값이 어떻게 전달되는지 체크해서 루틴을 만들 필요가 있음

---
## SDT's for L attributesd Definitions
Syn thesized 는 여전히 쉬움 하지만 inherited 는 어려움 

![](../images/Pasted%20image%2020260507104411.png)
while loop 는 condition 체크를 하고 만족하면 while 문 진행 그게 아니면 빠져나옴

가장 먼저해야할건 condition check 를 해야함 

S1 수행한 다음에는 while loop 으로 다시 돌아와야함
condition check 후 false라면 S1이 아니라 밖으로 나가야함

C가 false라면 S의 next가 될 것

S1.next 는 inherited attribute 
앞에서 일을 처리하고 전달을 받았기 때문

S.next 도 마찬가지로 inherited attribute 일 것이다.

C.code 는 자기 ㅅ자신이고
C의 true와 false는 부모에서 넘겨준것이기 때문에 inherited attribute임 

synsthesized attribute는 맨 뒤에 그리고 inheried attribute 는 필요한 곳 전에

그래서 c 전에 c.false c. true S1 전에 S1.next 그리고 맨 뒤에 synthesized

---
# Implemeting L-attributed SDD's

투 패스 컴파일러를 쓴다
중간 단계 파일을 dump 그걸 다시 읽어서 백엔드로 보낸다
조금 더 낫게 할 수 있는 방법은 중간중간에 계속 dump 하자
