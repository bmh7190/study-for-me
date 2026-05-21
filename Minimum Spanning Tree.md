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
Prim 알고리즘은 minimum spanning tree를 만들기 위해 vertex를 하나씩 tree에 추가한다. 그래프에 vertex가 `n`개 있다면, 최종적으로 모든 vertex가 tree에 포함되어야 하므로 vertex는 총 `n`번 추가된다.

처음에는 임의의 시작 vertex 하나를 tree에 넣고, 이후에는 매 단계마다 fringe vertex 중에서 현재 tree와 가장 작은 가중치로 연결되는 vertex를 하나 선택한다.

### vertex 하나를 추가할 때 하는 일
어떤 vertex `v`가 새롭게 tree에 추가되었다고 하자. 그러면 `v`와 연결된 다른 vertex들을 확인해야 한다. 왜냐하면 `v`가 tree에 들어오면서, 바깥 vertex로 가는 더 싼 edge가 새롭게 발견될 수 있기 때문이다.

예를 들어 어떤 vertex `E`가 원래는 가중치 10짜리 edge로 tree와 연결될 수 있었다고 하자. 그런데 새로 추가된 vertex `v`를 통해 `E`로 가는 edge의 가중치가 4라면, `E`의 최소 연결 비용을 10에서 4로 갱신해야 한다.

즉, vertex `v`를 tree에 넣을 때는 `v`에서 나가는 edge들을 확인하면서 fringe vertex들의 정보를 갱신한다.

이때 `v`에서 나가는 edge의 개수를 `|N(v)|`라고 하면, 이 edge들을 확인하는 데 걸리는 시간은 $O(|N(v)|)$ 이다.

### 그런데 왜 O(n + |N(v)|)인가?
배열 기반 구현에서는 매번 다음에 tree에 넣을 vertex를 찾기 위해 fringe vertex들의 배열을 훑어야 한다. 즉, 현재 tree와 연결될 수 있는 후보 vertex들 중에서 최소 가중치를 가진 vertex를 찾아야 한다.

이 과정을 단순 배열로 처리하면, 매번 최대 `n`개의 vertex를 확인해야 한다.

그래서 vertex 하나를 추가할 때 드는 시간은 크게 두 부분이다.

1. 최소 가중치를 가진 fringe vertex 찾기: O(n)
2. 새로 추가된 vertex v의 인접 edge 갱신하기: O(|N(v)|)

따라서 한 번의 반복에서 드는 시간은 $O(n + |N(v)|)$이다.

### 전체 시간 복잡도
이 과정을 모든 vertex에 대해 반복한다. 먼저, 최소 fringe vertex를 찾는 작업은 매번 `O(n)`이고, vertex를 총 `n`번 추가하므로 $O(n) \times n = O(n^2)$ 이다.

다음으로, 각 vertex가 tree에 추가될 때마다 그 vertex에서 나가는 edge들을 확인한다.

모든 vertex에 대해 `|N(v)|`를 다 더하면 전체 edge 수와 관련된다.
무방향 그래프라면 각 edge가 양쪽 vertex의 인접 리스트에 한 번씩 들어가므로

$$  
\sum_v |N(v)| = 2m  
$$

이다.

그래서 edge 갱신 전체 비용은 $O(m)$으로 볼 수 있다. 정확히는 무방향 그래프에서 `O(2m)`이지만, 상수는 생략하므로 `O(m)`이다. 따라서 전체 시간 복잡도는 $O(n^2 + m)$ 이다.

---
# Kruskal’s algorithm
Kruskal 알고리즘도 **minimum spanning tree**를 구하는 greedy algorithm이다. Prim 알고리즘이 하나의 tree를 점점 확장해 나가는 방식이라면, Kruskal 알고리즘은 **가중치가 작은 edge부터 하나씩 선택하면서 여러 개의 작은 tree들을 점점 합쳐 나가는 방식**이다.

처음에는 모든 vertex가 각각 독립된 tree라고 생각한다. 즉, vertex가 `n`개라면 처음에는 tree도 `n`개 있는 상태이다.

그 다음 그래프의 모든 edge를 가중치가 작은 순서대로 확인한다. 이때 어떤 edge를 선택했을 때 cycle이 생기지 않으면 그 edge를 MST에 추가한다. 반대로 그 edge를 선택했을 때 cycle이 생기면 추가하지 않고 넘어간다.

#### 왜 cycle을 피해야 할까?
Spanning tree는 모든 vertex를 연결해야 하지만, tree이기 때문에 cycle이 있으면 안 된다.

예를 들어 이미 `A`, `B`, `C`가 하나의 tree로 연결되어 있다고 하자. 이 상태에서 `A-C` edge를 추가하면 `A-B-C-A`처럼 cycle이 생길 수 있다.

이런 edge는 추가해도 새로운 vertex를 연결하는 데 도움이 되지 않는다. 이미 같은 tree 안에 있는 vertex끼리를 다시 연결하는 것이기 때문이다. 따라서 Kruskal 알고리즘은 edge를 추가하기 전에, 해당 edge의 양 끝 vertex가 **이미 같은 tree에 속해 있는지** 확인한다.

#### Disjoint Set을 사용하는 이유
Kruskal 알고리즘에서는 여러 개의 tree가 만들어지고, edge를 선택할 때마다 두 tree가 하나로 합쳐질 수 있다. 이때 각각의 tree를 하나의 집합으로 생각할 수 있다.

예를 들어 처음에는 다음과 같이 모든 vertex가 따로 있다.

```
{A}, {B}, {C}, {D}, {E}
```

만약 edge `A-B`를 선택하면 `A`와 `B`가 하나의 tree가 된다.

```
{A, B}, {C}, {D}, {E}
```

그 다음 edge `C-D`를 선택하면 다음과 같다.

```
{A, B}, {C, D}, {E}
```

이후 edge `B-C`를 선택하면 `{A, B}`와 `{C, D}`가 합쳐진다.

```
{A, B, C, D}, {E}
```

이처럼 Kruskal 알고리즘에서는 계속해서 **서로 다른 집합을 합치는 과정**이 발생한다.  
그래서 Disjoint Set 자료구조를 사용한다.

#### find와 union
Disjoint Set에서는 주로 `find`와 `union` 연산을 사용한다.

```
find(u)
```

`find(u)`는 vertex `u`가 현재 어떤 집합에 속해 있는지 알려준다.  
즉, `u`가 어떤 tree에 포함되어 있는지 확인하는 연산이다.

```
union(u, v)
```

`union(u, v)`는 `u`가 속한 집합과 `v`가 속한 집합을 하나로 합치는 연산이다.  
즉, 서로 다른 두 tree를 하나의 tree로 합치는 과정이라고 보면 된다.

#### Edge를 선택할 때 판단 기준
어떤 edge `(u, v)`를 확인한다고 하자. 먼저 `find(u)`와 `find(v)`를 실행한다.

만약 `find(u) = find(v)` 라면 `u`와 `v`는 이미 같은 tree에 속해 있다는 뜻이다. 이 edge를 추가하면 cycle이 생기므로 선택하지 않는다.

반대로 `find(u) ≠ find(v)` 라면 `u`와 `v`는 서로 다른 tree에 속해 있다는 뜻이다. 이 edge를 추가해도 cycle이 생기지 않으므로 MST에 추가한다. 그리고 두 tree를 하나로 합치기 위해 `union(u, v)` 를 수행한다.

---
## 의사 코드

```c
KruskalMST(G, n)

R = E
F = ∅

while R is not empty
    Remove the lightest edge vw from R

    if vw does not make a cycle in F
        Add vw to F

return F
```

> Kruskal 알고리즘은 모든 edge를 가중치가 작은 순서대로 확인하면서, cycle을 만들지 않는 edge만 선택하는 알고리즘이다. 처음에는 선택된 edge가 없으므로 각 vertex가 하나의 독립된 tree이고, 전체 구조는 forest로 볼 수 있다. 가장 작은 edge를 하나씩 확인하면서, 그 edge의 양 끝 vertex가 서로 다른 tree에 속해 있으면 edge를 추가하고 두 tree를 합친다. 반대로 이미 같은 tree에 속해 있다면 그 edge를 추가했을 때 cycle이 생기므로 선택하지 않는다. 이 과정을 반복하면 최종적으로 모든 vertex를 연결하면서 전체 edge weight의 합이 최소인 minimum spanning tree를 얻을 수 있다.

---
## 시간 복잡도
Kruskal 알고리즘은 모든 edge를 가중치가 작은 순서대로 확인하면서 MST를 만든다. 따라서 가장 먼저 해야 하는 일은 edge들을 weight 기준으로 정렬하는 것이다.

그래프의 edge 개수를 `m`이라고 하면, edge를 정렬하는 데 걸리는 시간은 다음과 같다.

$$O(m \log m)$$

정렬이 끝난 뒤에는 edge를 작은 것부터 하나씩 확인한다. 각 edge `uv`에 대해 `u`와 `v`가 이미 같은 tree에 속해 있는지 확인해야 한다.

이때 사용하는 자료구조가 **Union-Find**, 즉 **Disjoint Set**이다.

각 edge마다 수행하는 작업은 다음과 같다.

```
find(u)
find(v)
if find(u) != find(v)
    union(u, v)
```

`find(u)`와 `find(v)`를 통해 두 vertex가 같은 set에 있는지 확인한다. 만약 같은 set에 있다면 이미 같은 tree 안에 있다는 뜻이므로, 그 edge를 추가하면 cycle이 생긴다. 따라서 선택하지 않는다.

반대로 서로 다른 set에 있다면, 그 edge를 추가해도 cycle이 생기지 않는다. 그래서 edge를 MST에 넣고, `union(u, v)`를 통해 두 tree를 하나로 합친다.

Union-Find를 효율적으로 구현하면 `find`와 `union`은 거의 상수 시간에 가깝게 처리된다. 그래서 전체 edge를 한 번씩 확인하는 비용은 대략 $O(m)$ 정도로 볼 수 있다.

따라서 Kruskal 알고리즘의 전체 시간 복잡도는 $O(m \log m) + O(m)$ 이고, 더 큰 항만 남기면 $O(m \log m)$ 이 된다. 여기서 `m`은 edge의 개수이다.

---

Prim 알고리즘과 비교하면 그래프가 sparse한지 dense한지에 따라 차이가 생긴다.

Prim 알고리즘을 배열 기반으로 구현하면 시간 복잡도는 보통 $O(n^2 + m)$ 이고, dense graph에서는 보통 $O(n^2)$ 로 본다.

반면 Kruskal 알고리즘은 $O(m \log m)$ 이다. 만약 그래프가 sparse graph라면, 즉 edge 수 `m`이 vertex 수 `n`에 비해 훨씬 적다면 Kruskal이 유리할 수 있다.

예를 들어 $m = o(n^2)$ 이면 edge가 가능한 최대 개수인 `n^2`보다 훨씬 적다는 뜻이다. 이 경우 `m log m`이 `n^2`보다 작아질 수 있으므로, Kruskal 알고리즘이 Prim 알고리즘보다 빠를 수 있다.

반대로 dense graph라면 edge 수가 거의 최대에 가깝다 $m = \Theta(n^2)$ 이면 Kruskal의 시간 복잡도는 $O(m \log m) = O(n^2 \log n^2)$ 이다.

그런데 $\log n^2 = 2 \log n$ 이므로, $O(n^2 \log n^2) = O(n^2 \log n)$ 이 된다.

이때 배열 기반 Prim 알고리즘은 $O(n^2)$ 이므로, dense graph에서는 Kruskal이 Prim보다 대략 `log n`만큼 더 느릴 수 있다.