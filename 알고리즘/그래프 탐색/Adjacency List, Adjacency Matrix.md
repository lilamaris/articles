정점 간 링크를 표현하는 방법 중 `Adjacency List`와 `Adjacency Matrix` 에 대한 개념

### Adjacency List
방향이 있는 그래프에서, 각 정점이 도달할 수 있는  이웃 정점을 Linked List처럼 연결하는 방법
BFS 에서, 시작 정점이 Queue에 Push 된  상태.
Queue에서 Pop한 정점이 도달할 수 있는 이웃 정점을 Queue에 다시 Push 하는 과정을 Queue가 비어있을 때까지 반복하는 과정으로 BFS 달성

![[AdjacencyList.png]]


### Adjacency Matrix
행의 좌표가 시작 정점, 열의 좌표가 도착 정점이 되는 행렬
방향이 있는 그래프와 달리, 방향이 없는 그래프의 경우 대칭 행렬의 성질을 갖게되므로, 대각 성분의 아래 또는 위 영역을 계산 영역에서 제외할 수 있다.

![[DirectedAdjacencyMatrix.png]]
`방향이 있는 그래프에서 Adjacency Matrix 구성`

![[UndirectedAdjacencyMatrix.png]]
`방향이 없는 그래프에서 Adjacency Matrix 구성`
