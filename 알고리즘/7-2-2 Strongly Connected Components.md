어떤 방향 그래프에서 strongly connected라는 것은 서로 다른 두 정점 v와 w를 선택했을 때, v에서 w로 가는 path가 존재하고, w에서 v로 가는 path도 존재하는 경우를 말한다.

Component는 이러한 연결 관계를 만족하는 정점들의 묶음 중에서 더 이상 확장할 수 없는 최대 크기의 부분 그래프를 의미한다.

따라서 strongly connected component는 방향 그래프 안에서 모든 정점들이 서로 도달 가능한 최대 크기의 정점 집합이라고 할 수 있다.

![](../images/Pasted%20image%2020260604134237.png)

왼쪽 방향 그래프를 strongly connected components로 나누면 오른쪽과 같이 분리된다.

A, B, D, F는 서로 도달 가능하므로 하나의 SCC가 된다. 예를 들어 A에서 B, B에서 D, D에서 A로 갈 수 있고, F도 A와 서로 오갈 수 있는 경로가 있으므로 같은 component에 포함된다.

C는 다른 정점에서 C로 들어오는 edge는 있지만, C에서 다시 다른 정점으로 나가는 경로가 없기 때문에 혼자 하나의 SCC가 된다.

E와 G는 서로 오갈 수 있는 edge가 있으므로 하나의 SCC가 된다.

---
# Kosaraju

> 어떻게 하면 이 strongly connected component를 찾을 수 있을까?

앞에서 다룬 DFS를 이용하면 strongly connected component를 찾을 수 있다. 대표적인 알고리즘이 Kosaraju 알고리즘이다.

먼저 방향 그래프 $G$가 있을 때, $G$의 모든 edge 방향을 반대로 뒤집은 그래프를 $G^T$ 라고 한다. 이를 transpose graph라고 부른다.

Kosaraju 알고리즘은 크게 두 번의 DFS로 수행된다.

1. 첫 번째로 원래 그래프 G에서 DFS를 수행한다. 이때 어떤 정점에서 더 이상 탐색할 수 있는 인접 정점이 없으면, 즉 그 정점의 탐색이 완전히 끝나면 stack에 넣는다. 따라서 stack에는 finish time이 늦은 정점이 위쪽에 오게 된다.

2. 두 번째로 모든 edge 방향을 뒤집은 그래프 $G^T$에서 DFS를 수행한다. 이때는 첫 번째 DFS에서 만든 stack에서 정점을 하나씩 pop하면서, 아직 방문하지 않은 정점에 대해 DFS를 시작한다.

$G^T$ 에서 한 번의 DFS로 방문되는 정점들의 묶음이 하나의 strongly connected component가 된다.

---
##  예시로 알아보기 
이렇게 하면 이해가 안된다. 예시를 통해 알아보자

### 첫 번쨰 DFS 수행

먼저 원래 그래프 G에서 DFS를 수행하면서 finish time 순서대로 stack을 채운다.

![](../images/Pasted%20image%2020260604130449.png)

A에서 시작한다고 하면, DFS는 A → B → C 순서로 깊게 들어간다. 
C에서는 더 이상 방문할 수 있는 인접 정점이 없으므로 C의 탐색이 끝나고, C를 stack에 넣는다.

다시 B로 돌아오면 아직 방문하지 않은 인접 정점 D가 있으므로 D를 방문한다. 
D의 인접 정점은 A인데, A는 이미 방문된 상태이므로 더 이상 진행하지 않는다. 
따라서 D의 탐색이 끝나고 D를 stack에 넣는다.

다시 B로 돌아오면 B의 인접 정점 C와 D는 모두 이미 방문된 상태이다. 
따라서 B의 탐색도 끝났으므로 B를 stack에 넣는다.

그다음 A로 돌아온다. A의 인접 정점 B, C, F 중에서 B와 C는 이미 방문되었고, F는 아직 방문되지 않았으므로 F를 방문한다. 
F에서도 더 이상 새롭게 방문할 정점이 없으므로 F를 stack에 넣는다.

이제 A의 모든 인접 정점을 확인했으므로 A도 stack에 넣는다.

아직 방문하지 않은 정점 중 E를 시작점으로 DFS를 수행한다고 하자.

E에서 G로 갈 수 있으므로 G를 방문한다. 
G의 인접 정점 중 D는 이미 방문된 상태이므로 더 이상 진행할 수 없다. 
따라서 G의 탐색이 끝나고 G를 stack에 넣는다.

다시 E로 돌아오면 E도 더 이상 방문할 정점이 없으므로 E를 stack에 넣는다.

### 두 번째 DFS 수행

이제 모든 edge 방향을 뒤집은 그래프 $G^T$ 에서 DFS를 다시 수행한다. 이때 시작 정점은 임의로 고르는 것이 아니라, 첫 번째 DFS에서 만든 stack에서 pop한 순서를 따른다.

![](../images/Pasted%20image%2020260604140038.png)

먼저 stack에서 E를 pop한다. $G^T$ 에서 E를 시작점으로 DFS를 수행하면 E와 G가 함께 방문된다. 따라서 {E, G}가 하나의 strongly connected component가 된다. 그다음 stack에서 G가 pop되지만, G는 이미 E에서 시작한 DFS 과정에서 방문되었으므로 넘어간다.

다음으로 A를 pop한다. $G^T$ 에서 A를 시작점으로 DFS를 수행하면 A, F, B, D가 함께 방문된다. 
따라서 {A, F, B, D}가 하나의 strongly connected component가 된다.
그다음 stack에서 F, B, D가 차례대로 pop되더라도 이미 방문된 정점들이므로 넘어간다.

마지막으로 C를 pop한다. C는 아직 방문되지 않았으므로 $G^T$ 에서 C를 시작점으로 DFS를 수행한다. 이때 C만 방문되므로 {C}가 하나의 strongly connected component가 된다.

최종 SCC는 이렇게 된다.

```text
{E, G}
{A, B, D, F}
{C}
```


정리하면 Kosaraju 알고리즘은 먼저 원래 그래프 G에서 DFS를 수행하여 각 정점의 탐색이 끝나는 순서대로 stack에 저장한다. 그다음 모든 edge 방향을 뒤집은 $G^T$ 에서 stack의 top부터 정점을 pop하면서 DFS를 수행한다. 이때 한 번의 DFS로 방문되는 정점들의 집합이 하나의 strongly connected component가 된다.

>주의할 점은 **첫 번째 DFS의 stack은 방문 순서가 아니라 종료 순서**라는 것이다.

즉, 먼저 방문한 정점이 먼저 들어가는 게 아니라, **더 이상 탐색할 곳이 없어 탐색이 끝난 정점부터 stack에 들어간다.**