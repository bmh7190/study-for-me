# 알고리즘

그래프 탐색, 그래프 알고리즘, 문자열 매칭, 계산 복잡도와 NP 관련 개념을 정리한 노트입니다.

## 목차

### Graph Traversal

- [7-2-1 DFS ( Depth First Search )](<7-2-1 DFS ( Depth First Search ).md>)
  - 깊이 우선 탐색의 진행 방식과 DFS tree, back edge, cross edge 같은 간선 분류를 정리합니다.
- [7-2-2 BFS ( Breath First Search )](<7-2-2 BFS ( Breath First Search ).md>)
  - 너비 우선 탐색의 레벨 기반 탐색 흐름과 BFS에서의 간선 특성을 설명합니다.
- [7-2-2 Strongly Connected Components](<7-2-2 Strongly Connected Components.md>)
  - strongly connected component의 정의와 Kosaraju 알고리즘으로 SCC를 찾는 과정을 다룹니다.

### Graph Algorithms

- [8-1 Minimum Spanning Tree](<8-1 Minimum Spanning Tree.md>)
  - Prim과 Kruskal 알고리즘을 중심으로 minimum spanning tree를 정리합니다.
- [8-2 Single Source shortest paths](<8-2 Single Source shortest paths.md>)
  - 하나의 시작 정점에서 모든 정점까지의 최단 경로 문제를 다룹니다.
- [8-3 All pairs shortest paths](<8-3 All pairs shortest paths.md>)
  - 모든 정점 쌍 사이의 최단 경로를 구하는 알고리즘을 정리합니다.
- [9 Transitive Closure](<9 Transitive Closure.md>)
  - 그래프에서 도달 가능성(reachability)을 계산하는 방법을 다룹니다.

### String Matching

- [11-1 Knuth-Morris-Pratt(KMP) algorithm](<11-1 Knuth-Morris-Pratt(KMP) algorithm.md>)
  - prefix function을 이용해 문자열 매칭을 효율적으로 수행하는 KMP 알고리즘을 정리합니다.
- [11-2 Boyer-Moore-Horspool alogrithm](<11-2 Boyer-Moore-Horspool alogrithm.md>)
  - skip table을 이용해 문자열 탐색을 빠르게 수행하는 Boyer-Moore-Horspool 알고리즘을 다룹니다.

### Complexity and NP

- [13 NP (1)](<13 NP (1).md>)
  - NP의 기본 정의와 polynomial time, verification 관점을 소개합니다.
- [13 NP (2)](<13 NP (2).md>)
  - NP-complete, NP-hard 등 계산 복잡도 이론의 확장 개념을 다룹니다.
