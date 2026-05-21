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

시작 vertex s가 주어졌을 때,s로부터 각 vertex까지 가는 path 중edge weight의 합이 가장 작은 path를 찾는 문제

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

