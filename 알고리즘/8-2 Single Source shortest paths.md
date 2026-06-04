이번에는 특정 vertex `s`를 하나 고정해보자. 그리고 `s`로부터 그래프 안의 다른 모든 vertex로 가는 shortest path를 찾는 문제를 생각해볼 수 있다.

여기서 shortest path는 그래프의 종류에 따라 의미가 조금 달라진다.

먼저 weight이 없는 그래프에서는 shortest path가 단순하다. 이 경우에는 edge의 개수가 가장 적은 path를 shortest path라고 한다. 예를 들어 `s`에서 어떤 vertex `v`까지 가는 데 edge를 2개 거치는 경로와 4개 거치는 경로가 있다면, edge 2개를 거치는 경로가 더 짧은 경로이다.

하지만 weighted graph에서는 단순히 edge 개수가 적다고 해서 shortest path가 되는 것은 아니다. 각 edge마다 weight이 있기 때문에, path에 포함된 edge들의 weight 합이 가장 작은 path를 shortest path라고 한다. 이때는 shortest path를 **minimum-weight path**라고도 부른다.

예를 들어 `s`에서 `v`로 가는 두 경로가 있다고 해보자.

```
경로 1: s → A → v
edge weight 합 = 10 + 10 = 20

경로 2: s → B → C → v
edge weight 합 = 2 + 3 + 4 = 9
```

경로 1은 edge를 2개만 지나고, 경로 2는 edge를 3개 지나간다. 하지만 weighted graph에서는 edge 개수가 아니라 weight의 합을 기준으로 보기 때문에, 이 경우 shortest path는 경로 2가 된다. 

따라서 weighted graph에서 shortest path 문제는 다음과 같이 볼 수 있다.
>시작 vertex s가 주어졌을 때, s로부터 각 vertex까지 가는 path 중 edge weight의 합이 가장 작은 path를 찾는 문제

이러한 문제를 해결하는 대표적인 알고리즘이 **Dijkstra’s algorithm**이다

---
# Dijkstra's Algorithm

Dijkstra 알고리즘은 특정 시작 vertex `s`에서 다른 모든 vertex까지의 shortest path를 찾는 알고리즘이다.

처음에는 `s` 하나만 포함된 tree에서 시작한다. 그리고 현재 tree와 인접한 vertex들 중에서, **시작점 `s`로부터의 거리가 가장 짧은 vertex**를 하나 선택해서 tree에 추가한다.

여기서 중요한 점은 단순히 edge 하나의 weight만 보는 것이 아니라는 점이다.  
Dijkstra 알고리즘은 항상 s에서 해당 vertex까지 가는 전체 거리를 기준으로 판단한다.

예를 들어 현재 tree에 `A`, `B`가 들어 있고 시작점이 `A`라고 하자. 어떤 vertex `C`가 `B`와 edge weight 3으로 연결되어 있다면, `C`까지의 거리는 단순히 3이 아니다.

`A`에서 `B`까지의 거리와 `B-C` edge의 weight를 더해야 한다.

```
distance(A, C) = distance(A, B) + weight(B, C)
```

즉, Dijkstra는 현재 tree에서 바깥 vertex로 나가는 edge를 볼 때, 그 edge 자체의 weight만 보는 것이 아니라 **시작점에서 그 vertex까지 도달하는 누적 거리**를 본다.

---
## 진행 과정

Dijkstra 알고리즘은 다음과 같이 진행된다.

먼저 시작 vertex `s`를 tree에 넣고, `s`까지의 거리를 0으로 둔다.

```
distance(s) = 0
```

그리고 `s`와 인접한 vertex들에 대해 `s`로부터의 거리를 기록한다.

그 다음 현재 tree 밖에 있는 vertex 중에서, `s`로부터의 거리가 가장 짧은 vertex를 선택한다. 이 vertex는 더 이상 더 짧은 경로로 갱신될 수 없다고 보고 tree에 추가한다.

새로운 vertex가 tree에 들어오면, 그 vertex를 통해 다른 vertex로 가는 경로가 더 짧아질 수 있는지 확인한다. 만약 더 짧은 경로가 발견되면 거리 값을 갱신한다.

이 과정을 모든 vertex가 tree에 들어올 때까지 반복한다.

---
## 나중에 더 짧은 경로가 나올 수 있지 않을까?
결론은 불가능하다. 

Dijkstra는 항상 현재 가장 가까운 vertex부터 확정한다. 아직 확정되지 않은 vertex들은 이미 현재 선택된 vertex보다 멀리 있다. 그리고 edge weight이 음수가 아니기 때문에, 그 멀리 있는 vertex를 거쳐서 다시 돌아온다고 해도 거리가 줄어들 수 없다. 그래서 현재 가장 작은 거리 값을 가진 vertex는 나중에 더 작은 값으로 갱신될 가능성이 없다.


예시를 들어오면 현재 시작점 `A`로부터 거리 값이 이렇게 있다고 해보자.

```
B: 4
C: 7
D: 10
```

이 중 가장 작은 값은 `B = 4`이다. 그래서 `B`를 mark한다.

>혹시 나중에 `C`나 `D`를 거쳐서 `B`로 오면 더 짧아질 수 있을까?

`C`까지 가는 데 이미 7이 필요하다. 그런데 `C`에서 `B`로 가는 edge weight은 음수가 아니므로 최소 0 이상이다. 그러면 `C`를 거쳐서 `B`로 가는 거리는 최소한 `7 + 0 = 7` 이다. 이미 `B`까지의 거리는 4였으므로 더 짧아질 수 없다. `D`도 마찬가지다. `D`까지 이미 10이 필요하니까, `D`를 거쳐서 `B`로 가는 경로가 4보다 짧아질 수 없다. 그래서 `B = 4`는 확정해도 된다.

---
## 단 음수 weight은 없어야 한다
중요한 조건이 하나 있다. Dijkstra는 **edge weight이 모두 0 이상일 때만** 이 논리가 성립한다. 만약 음수 edge가 있으면 이야기가 달라진다.

예를 들어 현재는 `B = 4`, `C = 7`이라서 `B`를 먼저 확정했다고 하자.  

그런데 나중에 `C → B` edge weight이 `-5`라면,

```
A → C → B = 7 + (-5) = 2
```

가 된다.

그러면 원래 확정했던 `B = 4`보다 더 짧은 경로가 나와버린다. 그래서 Dijkstra 알고리즘은 **negative weight edge가 없는 그래프에서만 사용 가능**하다.

---
## 의사 코드

```c
dijkstraSSSP(G, n) // Weights are non-negative

Initialize all vertices as unseen.

Start the tree with the specified source vertex s;
reclassify it as tree;
define d(s, s) = 0.

Reclassify all vertices adjacent to s as fringe.

while there are fringe vertices:
    Select an edge between a tree vertex t and a fringe vertex v
    such that d(s, t) + W(tv) is minimum.

    Reclassify v as tree;
    add edge tv to the tree;
    define d(s, v) = d(s, t) + W(tv).

    Reclassify all unseen vertices adjacent to v as fringe.
```


> Dijkstra 알고리즘은 시작 vertex `s`로부터 다른 모든 vertex까지의 shortest path를 구하는 알고리즘이다. 처음에는 `s`만 tree에 넣고 `d(s,s)=0`으로 둔다. 이후 현재 tree와 인접한 fringe vertex들 중에서 `d(s,t)+W(t,v)` 값이 가장 작은 vertex `v`를 선택한다. 여기서 `d(s,t)`는 source에서 tree vertex `t`까지의 확정된 거리이고, `W(t,v)`는 `t`에서 `v`로 가는 edge weight이다. 선택된 vertex `v`는 tree에 추가되며, 이때 `d(s,v)`가 최단 거리로 확정된다. 이후 `v`와 인접한 unseen vertex들이 fringe가 되고, 기존 fringe vertex의 거리도 더 짧은 경로가 발견되면 갱신된다. 이 과정을 반복하면 source `s`에서 모든 vertex까지의 최단 거리를 구할 수 있다.

---
## 시간 복잡도
Dijkstra 알고리즘의 시간 복잡도는 배열 기반으로 구현하면 Prim 알고리즘과 거의 동일하게 분석할 수 있다.

Dijkstra 알고리즘은 시작 vertex `s`에서 출발해서, 다른 vertex들을 하나씩 tree에 추가한다. 이때 tree에 들어간 vertex는 `s`로부터의 최단 거리가 확정된 vertex이다. 그래프에 vertex가 `n`개 있다면, 최종적으로 모든 vertex가 tree에 들어가야 하므로 vertex는 총 `n`번 추가된다.

#### vertex 하나를 추가할 때의 비용
매번 새로운 vertex `v`를 tree에 추가할 때는 크게 두 가지 작업을 한다.

첫 번째는 아직 tree에 들어가지 않은 vertex 중에서 `s`로부터의 거리가 가장 짧은 vertex를 찾는 것이다. 배열 기반으로 구현하면 모든 vertex의 거리 값을 배열에 저장해두고, 아직 tree에 들어가지 않은 vertex들을 훑으면서 최소 거리 값을 가진 vertex를 찾는다. 이 작업은 최대 `n`개의 vertex를 확인해야 하므로 $O(n)$ 시간이 걸린다.

두 번째는 새로 tree에 들어온 vertex `v`에서 나가는 edge들을 확인하는 것이다. `v`와 인접한 vertex들에 대해, `v`를 거쳐 가는 경로가 기존에 알고 있던 거리보다 더 짧은지 확인한다.

즉, 인접한 vertex `u`에 대해 다음 값을 비교한다. $d(s,v) + W(v,u)$ 이 값이 기존의 `d(s,u)`보다 작다면 `d(s,u)`를 갱신한다. 이때 `v`와 인접한 vertex의 개수를 `|N(v)|`라고 하면, 이 갱신 작업에는 $O(|N(v)|)$ 시간이 걸린다. 따라서 vertex `v` 하나를 tree에 추가할 때 드는 시간은 $O(n + |N(v)|)$이다.

#### 전체 시간 복잡도
이 과정을 전체 vertex에 대해 반복한다. 먼저 최소 거리 vertex를 찾는 작업은 매번 `O(n)`이고, 이를 `n`번 반복하므로 $O(n^2)$ 시간이 걸린다.

다음으로 인접 edge를 확인하는 작업을 전체 vertex에 대해 모두 더하면, 결국 그래프의 모든 edge를 한 번씩 확인하는 것과 같다. 따라서 전체 edge 갱신 비용은 $O(m)$ 이다.

결국 배열 기반 Dijkstra 알고리즘의 전체 시간 복잡도는 $O(n^2 + m)$ 이다. 보통 단순 그래프에서는 `m ≤ n^2`이므로, 이를 간단히 $O(n^2)$ 로 정리하기도 한다.

---
### Heap 기반 구현
Dijkstra 알고리즘을 heap, 즉 priority queue를 이용해 구현하면 최소 거리 vertex를 더 빠르게 찾을 수 있다. 배열 기반에서는 매번 최소 거리 vertex를 찾기 위해 `O(n)` 시간이 걸렸지만, heap을 사용하면 최소 거리 vertex를 꺼내는 작업을 $O(\log n)$ 시간에 할 수 있다.

또한 edge를 확인하면서 거리 값이 더 작아지는 경우 priority queue의 값을 갱신해야 하므로, edge마다 최대 `O(log n)` 비용이 발생한다고 볼 수 있다. 따라서 heap 기반 Dijkstra 알고리즘의 시간 복잡도는 보통 $O(m \log n)$ 으로 정리한다.