All-Pairs Shortest Paths는 graph 또는 digraph에서 **모든 vertex 쌍 사이의 최단 거리**를 계산하는 문제이다. 즉, 특정 source `s` 하나만 고정하는 것이 아니라, 모든 vertex를 한 번씩 source로 잡고 최단 거리를 구하는 것이다.

예를 들어 vertex가 다음과 같이 있다고 하자.

```text
A, B, C, D
```

그러면 구해야 하는 것은 다음과 같다.

```text
A → B, A → C, A → D
B → A, B → C, B → D
C → A, C → B, C → D
D → A, D → B, D → C
```

즉, 모든 출발점과 모든 도착점 사이의 shortest path를 구해야 한다.

---
# 가장 쉬운 방법
가장 직관적인 방법은 모든 vertex를 한 번씩 source로 잡고, single-source shortest path 알고리즘을 반복하는 것이다. 즉, 각 vertex마다 Dijkstra 또는 Bellman-Ford 알고리즘을 수행하면 된다.

```text
for each vertex s:
    run Dijkstra or Bellman-Ford from s
```

Dijkstra는 edge weight이 음수가 없을 때 사용할 수 있고, Bellman-Ford는 음수 edge가 있어도 사용할 수 있다. 단, negative cycle이 있으면 shortest path 자체가 제대로 정의되지 않는다.

### Dijkstra를 반복하는 경우
Dijkstra 알고리즘을 heap 기반으로 구현하면 한 번 수행하는 데 보통 $O(m \log n)$ 시간이 걸린다. 그런데 이것을 모든 vertex에 대해 수행해야 하므로 총 `n`번 반복한다.

따라서 전체 시간 복잡도는 $O(n \cdot m \log n)$ 이 된다.
### Bellman-Ford를 반복하는 경우
Bellman-Ford 알고리즘은 한 source에서 다른 모든 vertex까지의 shortest path를 구하는 데 $O(nm)$ 시간이 걸린다.

이를 모든 vertex에 대해 `n`번 수행하면, $O(n \cdot nm) = O(n^2m)$ 이 된다.

## Dense graph에서는 왜 느릴까?
Dense graph는 edge가 매우 많은 그래프이다. vertex가 `n`개일 때 가능한 edge 수가 거의 최대에 가까우면, $m = \Theta(n^2)$ 로 볼 수 있다.

이때 Dijkstra를 모든 vertex에서 수행하면, $O(nm \log n)$ 이고, 여기에 $m = \Theta(n^2)$ 를 대입하면, $O(n \cdot n^2 \log n) = O(n^3 \log n)$ 이 된다.

즉, dense graph에서는 Dijkstra 반복 방식이 $O(n^3)$ 보다 `log n`만큼 더 느리다.

Bellman-Ford를 반복하는 경우는 더 느리다.
$O(n^2m)$ 에 $m = \Theta(n^2)$ 를 대입하면, $O(n^2 \cdot n^2) = O(n^4)$ 이 된다.

따라서 dense graph에서는 두 방식 모두 $O(n^3)$ 알고리즘보다 느리다.

---
# Floyd-Warshall Algorithm
Floyd-Warshall 알고리즘은 **모든 vertex 쌍 사이의 최단 거리**를 구하는 알고리즘이다. 특히 dense graph에 대해서도 항상 $O(n^3)$ 시간을 보장한다.

Dijkstra를 모든 vertex에 대해 반복하면 dense graph에서 $O(n^3 \log n)$이 될 수 있고, Bellman-Ford를 반복하면 $O(n^4)$까지 커질 수 있다. 반면 Floyd-Warshall 알고리즘은 세 겹 반복문을 사용하므로 항상 $O(n^3)$ 시간에 모든 쌍 최단 거리를 계산할 수 있다.