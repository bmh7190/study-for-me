Digraph `G`의 transitive closure `G*`란, `G`에서 어떤 vertex `v`로부터 vertex `w`로 가는 directed path가 존재할 때 arc `vw`를 추가한 그래프이다. 즉, 원래 그래프에서 직접 연결되어 있지 않더라도 여러 edge를 거쳐 도달할 수 있다면, transitive closure에서는 그 도달 가능성을 하나의 arc로 표현한다. 따라서 `G*`는 그래프 안의 모든 reachability 정보를 담고 있는 그래프라고 볼 수 있다.

![](../images/Pasted%20image%2020260521181351.png)

---
## Transitive Closure를 구하는 가장 쉬운 방법
Transitive closure `G*`를 구하는 가장 단순한 방법은 **각 vertex마다 DFS 또는 BFS를 한 번씩 수행하는 것**이다.

Transitive closure는 어떤 vertex `v`에서 다른 vertex `w`로 갈 수 있으면, `G*`에 arc `vw`를 추가한 그래프이다. 따라서 핵심은 각 vertex `v`에 대해 **v로부터 도달 가능한 모든 vertex를 찾는 것**이다.

이때 DFS나 BFS를 사용하면 된다. DFS와 BFS는 시작 vertex로부터 reachable한 모든 vertex를 탐색하는 알고리즘이기 때문이다.

예를 들어 vertex `A`에서 DFS를 시작했을 때 `B`, `C`, `D`를 방문할 수 있었다고 하자. 그러면 원래 그래프에서 `A`가 `B`, `C`, `D`에 도달 가능하다는 뜻이다. 따라서 transitive closure `G*`에는 다음 arc들을 추가한다.

```text
A → B
A → C
A → D
```

이 과정을 모든 vertex에 대해 반복한다.

---
## 시간 복잡도
시간 복잡도는 vertex가 `n`개, edge가 `m`개일 때 DFS/BFS 한 번이 `O(n + m)`이고, 이를 모든 vertex에 대해 수행하므로 $O(n(n+m))$이다. 

Dense graph처럼 `m = O(n^2)`이면 보통 $O(n^3)$ 으로 볼 수 있다.