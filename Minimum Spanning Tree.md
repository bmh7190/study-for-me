## Spanning Tree란?
Spanning tree는 그래프의 **모든 정점을 포함하면서, 모든 정점이 서로 연결되도록 만든 tree**를 말한다. 이때 tree이기 때문에 cycle은 존재하지 않는다. 즉, 원래 그래프에서 일부 edge만 선택해서 모든 정점을 연결하되, 불필요한 cycle은 제거한 형태라고 볼 수 있다.

만약 그래프의 edge에 가중치가 주어져 있다면, 여러 spanning tree 중에서 edge 가중치의 합이 가장 작은 tree를 찾을 수 있다. 이를 **minimum spanning tree**, 즉 **최소 신장 트리**라고 한다.

쉽게 말하면, minimum spanning tree는 모든 정점을 연결하는 방법 중에서 **전체 연결 비용이 가장 작은 방법**을 찾는 것이다.

예를 들어 여러 도시를 도로로 연결해야 한다고 해보자. 모든 도시가 서로 연결되도록 도로를 만들되, 도로 건설 비용의 총합이 가장 작아지도록 선택한 연결 구조가 minimum spanning tree라고 볼 수 있다.

이 Minimum Spanning Tree를 찾는 알고리즘으로 2가지 정도있다.

- Prim 알고리즘 : Vertex 기반 그리디 알고리즘
- Kruskal 알고리즘 : Edge 기반 그리디 알고리즘

---
# Prim's Algorithm
Prim 알고리즘은 **minimum spanning tree**를 구하는 대표적인 greedy algorithm이다.  
이 알고리즘은 하나의 vertex에서 시작해서, 현재 만들어진 tree를 조금씩 확장해 나가는 방식으로 동작한다.

중요한 점은 Prim 알고리즘이 **특정 출발점에서 특정 도착점까지 가는 최단 경로를 찾는 알고리즘이 아니라는 것**이다. 즉, 어떤 지점까지 가는 비용이 최소가 되는 경로를 찾는 알고리즘이 아니다. 그런 문제는 Dijkstra 알고리즘에 가깝다.

Prim 알고리즘의 목적은 그래프의 모든 vertex를 연결하되, 선택된 edge들의 가중치 합이 최소가 되도록 하는 것이다. 따라서 핵심은 “한 지점까지 가장 싸게 가는 것”이 아니라, **전체 그래프를 가장 적은 비용으로 연결하는 것**이다.

Prim 알고리즘은 다음과 같이 진행된다.

1. 임의의 vertex 하나를 선택해서 tree를 만든다.
2. 현재 tree에 포함된 vertex들과, 아직 tree에 포함되지 않은 vertex들을 연결하는 edge들을 확인한다.
3. 그중 가중치가 가장 작은 edge를 선택한다.
4. 선택된 edge와 연결된 새로운 vertex를 tree에 추가한다.
5. 모든 vertex가 tree에 포함될 때까지 이 과정을 반복한다.

쉽게 말하면, Prim 알고리즘은 현재까지 만든 tree에서 바깥으로 뻗어 나갈 수 있는 edge 중 가장 비용이 작은 것을 계속 선택하는 방식이다.

---
![](../images/Pasted%20image%2020260521141908.png)

이 그림을 예로 들어서, 시작 vertex를 `A`로 잡았다고 해보자.

처음에는 tree에 포함된 vertex가 `A` 하나뿐이다. 이때 `A`와 연결된 vertex는 `B`, `G`, `F`가 있다. Prim 알고리즘은 현재 tree와 바깥 vertex를 연결하는 edge 중에서 가중치가 가장 작은 edge를 선택한다. 따라서 `A-B`, `A-G`, `A-F` 중 가장 가중치가 작은 edge를 고르고, 그 결과 `B`가 tree에 추가된다고 해보자.

그러면 현재 tree는 `A, B`로 확장된다.

이제 후보 edge는 단순히 `A`와 연결된 edge만 보는 것이 아니다. 현재 tree에 포함된 `A` 또는 `B`와, 아직 tree에 포함되지 않은 vertex를 연결하는 edge들을 모두 확인한다. 즉, 기존 후보였던 `G`, `F`에 더해, `B`와 연결된 `C`도 후보에 들어오게 된다.

이 후보들 중에서 다시 가장 가중치가 작은 edge를 선택한다. 만약 그 edge가 `G`와 연결되어 있다면, 이번에는 `G`가 tree에 추가된다. 그러면 현재 tree는 `A, B, G`가 된다.

`G`가 tree에 포함되면 다시 후보 edge의 범위가 넓어진다. `G`와 연결된 `I`, `H` 같은 vertex들이 새롭게 후보에 포함될 수 있다. 이때 후보 edge들 중 가장 가중치가 작은 것이 `I`와 연결된 edge라면, `I`가 다음으로 tree에 추가된다.

여기서 중요한 점은 새로운 vertex가 추가될 때마다 **후보군이 계속 갱신된다**는 것이다. 예를 들어 `I`가 추가되면 `E`처럼 이전에는 직접 고려하지 않았던 vertex가 새롭게 후보로 들어올 수 있다. 또한 이미 후보에 있던 vertex라 하더라도, 새로 추가된 vertex를 통해 더 작은 가중치의 edge로 연결될 수 있다면 그 연결 비용이 갱신될 수 있다.

즉, Prim 알고리즘은 현재까지 만든 tree를 기준으로, 바깥 vertex들과 연결되는 가장 싼 edge를 계속 선택하면서 tree를 확장해 나간다. 이 과정을 모든 vertex가 tree에 포함될 때까지 반복하면, 전체 vertex를 연결하는 edge들의 가중치 합이 최소가 되는 minimum spanning tree를 만들 수 있다.

---
## 의사 코드

```c
PrimMST(G, n)

Initialize all vertices as unseen.

Select an arbitrary vertex s to start the tree;
reclassify it as tree.

Reclassify all vertices adjacent to s as fringe.

While there are fringe vertices:
    Select an edge of minimum weight between
    a tree vertex t and a fringe vertex v.

    Reclassify v as tree;
    add edge tv to the tree.

    Reclassify all unseen vertices adjacent to v as fringe.
```

> Prim 알고리즘은 vertex를 `unseen`, `fringe`, `tree` 상태로 나누어 MST를 만드는 알고리즘이다. 처음에는 임의의 vertex 하나를 `tree`로 선택하고, 그와 인접한 vertex들을 `fringe`로 둔다. 이후 현재 tree와 fringe vertex를 연결하는 edge 중 가장 가중치가 작은 edge를 선택하고, 해당 fringe vertex를 tree에 추가한다. 새 vertex가 tree에 들어오면 그 주변의 unseen vertex들이 새롭게 fringe가 되면서 후보 영역이 확장된다. 이 과정을 fringe vertex가 없어질 때까지 반복하면 모든 vertex를 최소 비용으로 연결하는 minimum spanning tree를 얻을 수 있다.

---
## 시간 복잡도
Prim 알고리즘은 minimum spanning tree를 만들기 위해 vertex를 하나씩 tree에 추가한다.

그래프에 vertex가 `n`개 있다면, 최종적으로 모든 vertex가 tree에 포함되어야 하므로 vertex는 총 `n`번 추가된다.

처음에는 임의의 시작 vertex 하나를 tree에 넣고, 이후에는 매 단계마다 fringe vertex 중에서 현재 tree와 가장 작은 가중치로 연결되는 vertex를 하나 선택한다.

### vertex 하나를 추가할 때 하는 일
어떤 vertex `v`가 새롭게 tree에 추가되었다고 하자.

그러면 `v`와 연결된 다른 vertex들을 확인해야 한다. 왜냐하면 `v`가 tree에 들어오면서, 바깥 vertex로 가는 더 싼 edge가 새롭게 발견될 수 있기 때문이다.

예를 들어 어떤 vertex `E`가 원래는 가중치 10짜리 edge로 tree와 연결될 수 있었다고 하자. 그런데 새로 추가된 vertex `v`를 통해 `E`로 가는 edge의 가중치가 4라면, `E`의 최소 연결 비용을 10에서 4로 갱신해야 한다.

즉, vertex `v`를 tree에 넣을 때는 `v`에서 나가는 edge들을 확인하면서 fringe vertex들의 정보를 갱신한다.

이때 `v`에서 나가는 edge의 개수를 `|N(v)|`라고 하면, 이 edge들을 확인하는 데 걸리는 시간은 $O(|N(v)|)$ 이다.

### 그런데 왜 O(n + |N(v)|)인가?
배열 기반 구현에서는 매번 다음에 tree에 넣을 vertex를 찾기 위해 fringe vertex들의 배열을 훑어야 한다. 즉, 현재 tree와 연결될 수 있는 후보 vertex들 중에서 최소 가중치를 가진 vertex를 찾아야 한다.

이 과정을 단순 배열로 처리하면, 매번 최대 `n`개의 vertex를 확인해야 한다.

그래서 vertex 하나를 추가할 때 드는 시간은 크게 두 부분이다.

```text
1. 최소 가중치를 가진 fringe vertex 찾기: O(n)
2. 새로 추가된 vertex v의 인접 edge 갱신하기: O(|N(v)|)
```

따라서 한 번의 반복에서 드는 시간은 $O(n + |N(v)|)$이다.

### 전체 시간 복잡도
이 과정을 모든 vertex에 대해 반복한다.

먼저, 최소 fringe vertex를 찾는 작업은 매번 `O(n)`이고, vertex를 총 `n`번 추가하므로

$$  
O(n) \times n = O(n^2)  
$$

이다.

다음으로, 각 vertex가 tree에 추가될 때마다 그 vertex에서 나가는 edge들을 확인한다.

모든 vertex에 대해 `|N(v)|`를 다 더하면 전체 edge 수와 관련된다.

무방향 그래프라면 각 edge가 양쪽 vertex의 인접 리스트에 한 번씩 들어가므로

$$  
\sum_v |N(v)| = 2m  
$$

이다.

그래서 edge 갱신 전체 비용은

$$  
O(m)  
$$

으로 볼 수 있다. 정확히는 무방향 그래프에서 `O(2m)`이지만, 상수는 생략하므로 `O(m)`이다.

따라서 전체 시간 복잡도는 $O(n^2 + m)$ 이다.

---

정리하면 이렇게 쓰면 돼.

> Prim 알고리즘은 MST를 만들기 위해 vertex를 총 `n`번 tree에 추가한다. 배열 기반 구현에서는 매 단계마다 fringe vertex 중 최소 가중치로 연결되는 vertex를 찾기 위해 최대 `n`개의 vertex를 확인하므로 `O(n)` 시간이 걸린다. 또한 새로 tree에 추가된 vertex `v`와 인접한 edge들을 확인하면서 fringe 정보를 갱신해야 하므로 `O(|N(v)|)` 시간이 추가된다. 따라서 vertex 하나를 추가할 때 `O(n + |N(v)|)` 시간이 걸리고, 이를 전체 vertex에 대해 반복하면 `O(n^2 + m)` 시간이 된다.

조금 더 짧게 말하면,

```text
최소 후보 선택 비용: n번 × O(n) = O(n²)
인접 edge 갱신 비용: 전체 edge를 한 번씩 확인 = O(m)

따라서 Prim 알고리즘의 시간 복잡도 = O(n² + m)
```

보통 단순 배열 기반 Prim 알고리즘에서는 `m ≤ n²`이므로

$$  
O(n^2 + m) = O(n^2)  
$$

으로 정리하기도 한다.