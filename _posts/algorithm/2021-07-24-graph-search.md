---
title:  "[알고리즘] BFS/DFS"

categories:
  - Algorithm
tags:
  - [algorithm, BFS, DFS]
toc: true
toc_sticky: true
---

> ### 그래프 탐색

###  BFS 와 DFS 란?

대표적인 그래프 탐색 알고리즘

* 너비 우선 탐색 (Breadth First Search) : 정점들과 같은 레벨에 있는 노드들 (형제 노드들)을 먼저 탐색하는 방식

* 깊이 우선 탐색 (Depth First Search) : 정점의 자식들을 먼저 탐색하는 방식



- BFS 방식: A - B - C - D - G - H - I - E - F - J
  - 한 단계씩 내려가면서, 해당 노드와 같은 레벨에 있는 노드들 (형제 노드들)을 먼저 순회함
- DFS 방식: A - B - D - E - F - C - G - H - I - J
  - 한 노드의 자식을 타고 끝까지 순회한 후, 다시 돌아와서 다른 형제들의 자식을 타고 내려가며 순화함

![img](https://www.fun-coding.org/00_Images/BFSDFS.png)

> ### BFS 알고리즘 구현

- 자료구조 큐를 활용함
  - need_visit 큐와 visited 큐, 두 개의 큐를 생성

```python
from collections import deque

def bfs(graph, start_node):
    visited = list()
    need_visit = deque()
    
    need_visit.append(start_node) # 첫 노드를 방문할 큐에 추가
    
    while need_visit:
        current = need_visit.popleft() # 방문할 큐에서 다음 노드를 꺼내옴
        if current not in visited: # 방문한 적이 없으면
            visited.append(current) # 방문한 노드에 추가
            need_visit.extend(graph[current]) # 인접한 노드들을 방문할 큐에 추가
    
    return visited
```

* 시간 복잡도 : O(V + E)

> ### DFS 알고리즘 구현

- 자료구조 스택과 큐를 활용함
  - need_visit 스택과 visited 큐, 두 개의 자료 구조를 생성

```python
def dfs(graph, start_node):
    visited, need_visit = list(), list()
    need_visit.append(start_node)
    
    while need_visit:
        current = need_visit.pop()
        if current not in visited:
            visited.append(current)
            need_visit.extend(graph[current])
    
    return visited
```

* 시간 복잡도 : O(V + E)