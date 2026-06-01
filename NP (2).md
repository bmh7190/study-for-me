이번 장은 앞에서 배운 P, NP, NP-hard, NP-complete 개념을 바탕으로 실제 NP-complete 문제들을 보는 부분이다.

앞에서는 주로 개념들에 대해서 다뤘다.

- P: 다항시간 안에 풀 수 있는 문제
- NP: 답 후보가 주어지면 다항시간 안에 검증할 수 있는 문제
- NP-hard: 모든 NP 문제만큼 어렵거나 더 어려운 문제
- NP-complete: NP-hard이면서 동시에 NP에 속하는 문제

13장-2에서는 “그래서 실제로 어떤 문제가 NP-complete인가?”를 하나씩 확인한다.

# Satisfiability(SAT) Problem

최초의 NP-complete 문제로 **SAT**, 즉 Satisfiability 문제가 나온다.

SAT는 Boolean formula가 주어졌을 때, 각 변수에 0 또는 1을 잘 넣어서 전체 식을 참으로 만들 수 있는지 묻는 문제다.

예를 들어 식이 다음처럼 주어진다.

![](../images/Pasted%20image%2020260601172111.png)

여기서 전체 구조를 보면, 작은 OR 묶음들이 있고, 그 OR 묶음들이 AND로 연결되어 있다.

- variable: a, b, c, d, e 같은 변수
- literal: a, ¬b, c처럼 변수 또는 변수의 부정
- clause: (a ∨ ¬b ∨ c)처럼 literal들이 OR로 묶인 것
- Boolean formula: 여러 clause가 AND로 묶인 전체 식

즉, clause 하나는 그 안의 literal 중 하나라도 참이면 참이다. 그런데 전체 formula는 clause들이 AND로 묶여 있으므로, **모든 clause가 참이어야 전체 식이 참**이 된다.

---
## SAT Example

$$(a ∨ ¬b ∨ c) ∧ (¬a ∨ c ∨ ¬d ∨ ¬e) ∧ (b ∨ c ∨ d) ∧ (¬c ∨ e)$$

예를 들어 a, b, c, d, e = (1, 1, 0, 0, 1)이라고 하자.

그러면 각 clause를 확인한다.

첫 번째 clause: (a ∨ ¬b ∨ c)

- a = 1이므로 이미 참이다.
- 따라서 첫 번째 clause는 참이다.

두 번째 clause: (¬a ∨ c ∨ ¬d ∨ ¬e)

- ¬a = 0
- c = 0
- ¬d = 1
- 따라서 참이다.

이런 식으로 모든 clause가 최소 하나의 literal에서 참이 되면 전체 Boolean formula는 만족된다.

반대로 a, b, c, d, e = (0, 1, 0, 1, 1)이라면 첫 번째 clause가 문제가 된다.

첫 번째 clause: (a ∨ ¬b ∨ c)

- a = 0
- b = 1이므로 ¬b = 0
- c = 0

모두 0이므로 이 clause는 거짓이다. 전체 formula는 AND로 연결되어 있기 때문에 clause 하나만 거짓이어도 전체 식은 거짓이다.

즉, SAT는 “모든 clause를 동시에 참으로 만드는 변수 배정이 존재하는가?”를 묻는 문제다.

---
# SAT는 NP-complete 인가?
SAT 문제는 다음과 같이 정의된다.

> Boolean formula S가 주어졌을 때, 모든 clause를 만족시키는 0/1 assignment가 존재하는가?

먼저 SAT가 NP에 속하는 이유를 보자.

누군가가 “이 변수 배정이면 식이 만족된다”라고 답 후보를 줬다고 하자. 그러면 우리는 각 clause를 하나씩 검사하면 된다. 각 clause 안에 참인 literal이 하나라도 있는지 보면 되고, 모든 clause가 참인지 확인하면 된다.

이 검사는 식의 길이에 비례하는 시간, 즉 linear time 안에 가능하다. 그래서 SAT는 NP에 속한다.

그리고 Cook-Levin theorem에 의해 SAT는 NP-complete이다.

이 말은 굉장히 중요하다. SAT는 “최초로 NP-complete임이 증명된 문제”다. 그 전까지는 어떤 문제를 NP-complete라고 말하려면 모든 NP 문제로부터 환원됨을 보여야 했는데, Cook-Levin theorem이 SAT가 그 역할을 한다는 것을 증명한 것이다.

그래서 이후에는 SAT에서 다른 문제로 환원하면서 NP-complete 문제들을 늘려갈 수 있다.

---
# 3-SAT
3-SAT는 SAT의 특수한 형태다. SAT는 clause 안에 literal이 몇 개든 상관없다. 예를 들어 어떤 clause는 literal 2개, 어떤 clause는 4개, 어떤 clause는 10개일 수도 있다.

하지만 3-SAT는 각 clause가 정확히 3개의 literal로 구성된 Boolean formula의 만족 가능성을 묻는 문제다.

예를 들면 이런 형태다.

$$(a ∨ ¬b ∨ c) ∧ (¬a ∨ d ∨ e) ∧ (b ∨ ¬c ∨ d)$$

각 괄호 안에 literal이 정확히 3개씩 있다.

중요한 점은 3-SAT가 SAT보다 제한된 형태인데도 여전히 NP-complete라는 것이다. Karp가 SAT로부터 3-SAT로의 환원을 통해 3-SAT도 NP-complete임을 보였다.

>환원은 A의 입력을 그대로 B에 넣는 것이 아니라, A의 입력 X를 다항시간 안에 B의 입력 Y로 변환했을 때 yes/no 결과가 동일하게 유지되는 것을 말한다. 따라서 SAT ≤p 3SAT라는 것은 SAT 식 S를 3SAT 형식의 식 S'로 변환할 수 있고, S가 만족 가능하면 S'도 만족 가능하며, S가 만족 불가능하면 S'도 만족 불가능하다는 뜻이다. 즉, SAT의 답과 변환된 3SAT의 답이 항상 같아야 한다. 그리고 SAT는 이미 NP-complete이고, 3SAT는 assignment가 주어지면 각 clause를 검사해서 다항시간에 검증 가능하므로 NP에 속한다. 결국 SAT를 3SAT로 환원할 수 있으므로 3SAT는 SAT만큼 어렵고, 동시에 NP에도 속하기 때문에 3SAT는 NP-complete이다.

이게 왜 중요하냐면, 이후 많은 그래프 문제들을 증명할 때 일반 SAT보다 3-SAT에서 출발하는 경우가 많다. clause 크기가 3개로 고정되어 있어서 그래프로 변환하기가 더 편하기 때문이다.

---
# CLIQUE

Clique는 그래프에서 **서로 모두 연결되어 있는 꼭짓점들의 집합**이다.

예를 들어 어떤 그래프에서 정점 A, B, C가 있다고 하자. A-B, B-C, A-C가 모두 edge로 연결되어 있으면 {A, B, C}는 clique이다.

CLIQUE 문제는 다음과 같이 정의된다.

>그래프 G와 정수 k가 주어졌을 때, 크기가 최소 k인 clique가 존재하는가?

여기서 “최소 k”라는 말은 k개 이상인 clique가 있냐는 뜻이다.

CLIQUE가 NP에 속하는 이유는 간단하다. 누군가 정점 집합 S를 답 후보로 줬다고 하자. 그러면 S 안에 있는 모든 정점 쌍이 서로 edge로 연결되어 있는지 확인하면 된다. 정점이 k개라면 확인할 쌍은 대략 k²개이므로 O(k²) 시간 안에 검증 가능하다.

그리고 Karp가 3-SAT 또는 SAT로부터 CLIQUE로 환원해서 CLIQUE가 NP-complete임을 보였다.

직관적으로는 이런 느낌이다.

3-SAT의 각 clause에서 literal 하나씩 고른다고 생각한다. 모든 clause에서 하나씩 골랐을 때 서로 모순되지 않는 선택이면 전체 식을 만족시킬 수 있다. 이 구조를 그래프로 바꾸면, “각 clause에서 하나씩 고르면서 서로 충돌하지 않는 선택”이 clique를 찾는 문제로 바뀐다.

즉, SAT의 “만족 가능한 변수 선택” 문제가 CLIQUE의 “서로 연결된 정점 선택” 문제로 변환되는 것이다.

---
# INDEPENDENT SET

Independent set은 clique와 반대 느낌이다.

Clique는 집합 안의 정점들이 서로 모두 연결되어 있어야 한다. 반면 independent set은 집합 안의 정점들이 서로 하나도 연결되어 있지 않아야 한다. 
즉, independent set은 **서로 모두 이웃하지 않는 꼭짓점들의 집합**이다.

INDEPENDENT SET 문제는 다음과 같다.

>그래프 G와 정수 k가 주어졌을 때, 크기가 최소 k인 independent set이 존재하는가?

이 문제도 NP에 속한다. 누군가 정점 집합 S를 주면, S 안의 모든 정점 쌍 사이에 edge가 없는지 확인하면 된다. 있으면 independent set이 아니고, 없으면 independent set이다.

중요한 연결은 CLIQUE에서 INDEPENDENT SET으로의 환원이다.

여기서 핵심은 **complement graph**다.
원래 그래프 G에서 edge가 있던 곳은 없애고, edge가 없던 정점 쌍에는 edge를 추가한 그래프를 complement graph라고 한다.

그러면 원래 그래프 G에서 clique였던 정점 집합은 complement graph에서는 independent set이 된다. 왜냐하면 원래 서로 모두 연결되어 있던 정점들이 complement graph에서는 서로 연결되지 않기 때문이다.

따라서 CLIQUE 문제를 INDEPENDENT SET 문제로 바꿀 수 있다.

G에 크기 k인 clique가 존재한다 ↔ complement graph에 크기 k인 independent set이 존재한다.

그래서 INDEPENDENT SET은 CLIQUE로부터 환원되어 NP-complete이다.

---
# VERTEX COVER

Vertex cover는 그래프의 모든 edge를 “덮는” 정점 집합이다.

정점 집합 S가 vertex cover라는 것은, 그래프의 모든 edge에 대해 그 edge의 양 끝점 중 적어도 하나가 S에 포함된다는 뜻이다.

예를 들어 edge (u, v)가 있다면, u 또는 v 중 하나는 vertex cover 안에 있어야 한다.

VERTEX COVER 문제는 다음과 같다.

>그래프 G와 정수 k가 주어졌을 때, 크기가 최대 k인 vertex cover가 존재하는가?

여기서는 앞 문제들과 다르게 “최소 k”가 아니라 “최대 k”다. 왜냐하면 vertex cover는 보통 가능한 적은 수의 정점으로 모든 edge를 덮고 싶기 때문이다.

이 문제도 NP에 속한다. 누군가 정점 집합 S를 주면, S를 제거했을 때 남은 그래프에 edge가 있는지 확인하면 된다. 또는 모든 edge를 하나씩 보면서 양 끝점 중 하나가 S에 있는지 확인하면 된다.

VERTEX COVER가 NP-complete인 이유는 INDEPENDENT SET으로부터 환원되기 때문이다.

핵심 관계는 이거다.

그래프 G의 정점 수가 n일 때,  
G에 크기 k인 independent set이 존재한다  ↔ G에 크기 n-k인 vertex cover가 존재한다.

왜냐하면 independent set에 포함되지 않은 나머지 정점들이 모든 edge를 덮기 때문이다.

만약 independent set S 안에 edge가 있다면 independent set이 아니다. 따라서 모든 edge는 적어도 한쪽 끝점이 S 밖에 있어야 한다. 그러면 V-S가 모든 edge를 덮는 vertex cover가 된다.

그래서 INDEPENDENT SET이 어렵다면 VERTEX COVER도 어렵고, VERTEX COVER는 NP에 속하므로 NP-complete이다.

---
#  SET-COVER와 SUBSET-SUM

여기서는 두 가지 문제가 나온다.

## SET-COVER

SET-COVER 문제는 다음과 같다.

>m개의 set들이 주어졌을 때, 그중 k개의 set만 골라서 전체 원소를 모두 덮을 수 있는가?

예를 들어 전체 원소가 {1, 2, 3, 4, 5}라고 하자.

집합들이 다음처럼 있다.

S1 = {1, 2}  
S2 = {2, 3}  
S3 = {4, 5}  
S4 = {1, 3, 5}

이때 k = 2라면, 두 개의 set만 골라 전체 {1, 2, 3, 4, 5}를 모두 포함할 수 있는지 묻는 것이다.

SET-COVER는 VERTEX COVER로부터 환원되어 NP-complete이다.

직관적으로 보면 vertex cover도 사실 “모든 edge를 덮는 정점 집합”을 찾는 문제다. 각 정점이 자신과 연결된 edge들의 집합을 덮는다고 생각하면, vertex cover는 set cover의 특수한 형태처럼 볼 수 있다.

즉, edge들을 전체 원소로 보고, 각 vertex를 “그 vertex가 덮는 edge들의 set”으로 보면 vertex cover 문제가 set cover 문제로 바뀐다.

## SUBSET-SUM

SUBSET-SUM 문제는 다음과 같다.

>정수들의 집합과 목표값 k가 주어졌을 때, 
>일부 정수들을 골라 합이 정확히 k가 되게 할 수 있는가?

예를 들어 숫자들이 {3, 5, 7, 10}이고 k = 15라면, 5 + 10 = 15이므로 yes다.

SUBSET-SUM도 NP-complete 문제다. 슬라이드에서는 VERTEX-COVER로부터 환원된다고 되어 있다.

이 문제의 핵심은 “선택한다 / 선택하지 않는다” 구조다. 각 숫자를 subset에 넣을지 말지 결정하는 것이고, 그 결과 합이 정확히 목표값이 되는지 본다. 이런 선택 구조 때문에 다른 NP-complete 문제를 숫자 선택 문제로 변환할 수 있다.

---
# KNAPSACK과 HAMILTONIAN CYCLE

## 0/1 KNAPSACK

0/1 KNAPSACK 문제는 다음과 같다.

>각 item마다 weight와 benefit이 있을 때, 
>전체 weight이 W 이하이면서 benefit이 k 이상이 되도록 item 일부를 고를 수 있는가?

여기서 0/1이라는 말은 각 item을 “넣거나 / 안 넣거나” 둘 중 하나만 선택한다는 뜻이다. 같은 item을 여러 번 넣을 수 없다.

예를 들어 배낭 용량 W = 10이고, 목표 benefit k = 20이라고 하자. item들이 여러 개 있을 때, 무게 합은 10 이하로 유지하면서 benefit 합이 20 이상이 되도록 고를 수 있는지 묻는다.

이 문제는 SUBSET-SUM으로부터 환원되어 NP-complete이다.

직관적으로 SUBSET-SUM은 숫자들을 골라 정확히 k를 만드는 문제다. 이것을 knapsack으로 바꾸면, 각 숫자를 item의 weight 또는 benefit으로 생각할 수 있다. 그래서 subset을 고르는 문제가 knapsack에서 item을 고르는 문제로 바뀐다.

주의할 점은 우리가 알고 있는 “DP로 푸는 knapsack”과 NP-complete가 충돌해 보일 수 있다는 점이다.

0/1 knapsack은 W에 대해 O(nW) 같은 DP 알고리즘이 있다. 하지만 이건 입력 크기에 대한 진짜 다항시간이라고 보기 어렵다. 왜냐하면 W가 이진수로 입력되면 W의 값은 입력 길이에 비해 매우 커질 수 있기 때문이다. 그래서 decision version의 0/1 knapsack은 NP-complete로 분류된다.

## HAMILTONIAN CYCLE

Hamiltonian cycle 문제는 다음과 같다.

>그래프 G가 주어졌을 때, 
>모든 vertex를 정확히 한 번씩 방문하고 다시 시작점으로 돌아오는 cycle이 존재하는가?

이 문제는 “모든 정점을 한 번씩 방문하는 순환 경로”를 찾는 문제다.

DFS나 BFS에서 말하는 단순 경로와 다르게, 모든 정점을 정확히 한 번씩 방문해야 하기 때문에 훨씬 어렵다.

이 문제는 VERTEX COVER로부터 환원되어 NP-complete이다.

여기서 헷갈리면 안 되는 점이 있다.

Euler cycle은 모든 edge를 한 번씩 방문하는 cycle이고, polynomial time에 판별할 수 있다.  
Hamiltonian cycle은 모든 vertex를 한 번씩 방문하는 cycle이고, NP-complete이다.

둘은 비슷해 보이지만 난이도가 완전히 다르다.

---
# DIRECTED HAMILTONIAN CYCLE과 TSP

## DIRECTED HAMILTONIAN CYCLE

Directed Hamiltonian cycle은 방향 그래프에서 Hamiltonian cycle이 존재하는지 묻는 문제다.

즉, 각 edge에 방향이 있고, 그 방향을 따라가면서 모든 vertex를 정확히 한 번씩 방문한 뒤 시작점으로 돌아와야 한다.

이 문제는 일반 HAMILTONIAN CYCLE로부터 환원되어 NP-complete이다.

무방향 그래프의 edge를 방향 edge 구조로 바꾸어 directed graph 문제로 만들 수 있기 때문이다.

### TSP

TSP는 Travelling Salesman Problem, 즉 외판원 문제다.

슬라이드에서는 결정 문제 형태로 정의한다.

Complete edge-weighted graph G가 주어졌을 때, weight 합이 k 이하인 Hamiltonian cycle이 존재하는가?

여기서 complete graph라는 것은 모든 정점 쌍 사이에 edge가 있는 그래프다. 그리고 각 edge에는 비용 또는 거리가 있다.

TSP는 보통 “모든 도시를 한 번씩 방문하고 출발 도시로 돌아올 때 최소 비용 경로를 찾아라”라는 최적화 문제로 많이 알려져 있다.

그런데 NP-complete를 말할 때는 결정 문제로 바꾼다.

최적화 버전: 가장 짧은 순회 경로의 비용은 얼마인가?  
결정 버전: 비용이 k 이하인 순회 경로가 존재하는가?

TSP의 결정 버전은 HAMILTONIAN CYCLE로부터 환원되어 NP-complete이다.

직관적으로는 이렇게 생각하면 된다.

Hamiltonian cycle 문제는 “모든 정점을 한 번씩 방문하는 cycle이 있냐?”를 묻는다.  
TSP는 “모든 정점을 한 번씩 방문하는 cycle 중 weight이 k 이하인 것이 있냐?”를 묻는다.

Hamiltonian cycle을 TSP로 바꿀 때는 원래 그래프에 있던 edge는 작은 weight, 없던 edge는 큰 weight를 주는 식으로 만들 수 있다. 그러면 weight이 일정 기준 이하인 TSP 경로가 존재한다는 것은 원래 그래프에 Hamiltonian cycle이 존재한다는 뜻이 된다.

---
# TSP 예시

마지막 슬라이드는 작은 complete graph에서 “weight at most 75인 Hamiltonian cycle이 있는가?”를 묻는 예시다.

정점은 1, 2, 3, 4가 있고, 각 edge마다 weight이 있다.

Hamiltonian cycle은 모든 정점을 정확히 한 번씩 방문하고 다시 시작점으로 돌아와야 한다.

정점이 4개라면 가능한 cycle 중 하나는 다음과 같다.

1 → 2 → 3 → 4 → 1

또 다른 cycle은

1 → 3 → 2 → 4 → 1

이런 식으로 여러 순회 경로가 있을 수 있다.

문제는 그중에서 edge weight 합이 75 이하인 cycle이 존재하는지 확인하는 것이다.

여기서 중요한 건 TSP가 “최단 경로 하나 찾기” 문제가 아니라, NP-complete 맥락에서는 **기준 k 이하의 Hamiltonian cycle이 존재하는지 묻는 결정 문제**라는 점이다.

즉, 이 슬라이드는 TSP를 다음처럼 이해하라는 예시다.

“모든 정점을 한 번씩 방문하는 cycle들 중에서, 총 비용이 75 이하인 것이 있나?”