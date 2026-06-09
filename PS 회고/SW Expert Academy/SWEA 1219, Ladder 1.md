---
published: 2026-06-10
tags:
  - swea
  - cpp
---
[문제 링크](https://swexpertacademy.com/main/code/problem/problemDetail.do?problemLevel=4&problemLevel=5&contestProbId=AV14ABYKADACFAYh&categoryId=AV14ABYKADACFAYh&categoryType=CODE&problemTitle=&orderBy=FIRST_REG_DATETIME&selectCodeLang=ALL&select-1=5&pageSize=10&pageIndex=1)
---
## 내 접근

1. 사다리 경로 중 목표한 좌표에 도달할 수 있는 사다리 시작 좌표를 찾는 문제.
2. 모든 사다리 경로를 전부 탐색할 필요는 없는 것 같다. 목표 좌표로부터 거꾸로 올라가서 시작 좌표를 찾는 방식이면 한번만 탐색해도 되니까.
3. BFS와 비슷한 구조가 될 것 같다. 가로 방향으로 나아갈 수 있을 때를 제외하고는 세로 방향 탐색이 이뤄져야한다.
4. 문제 제약 사항에서 한 사다리에서 뻗은 가로 막대가 또 다른 세로 막대를 가로지르는 경우는 없다고 명시했으므로, 가로 방향으로 탐색할 때 좌,우측 모두 탐색 가능한 경우는 없다.
5. 이 경우, 탐색이 종료되는 조건은 시작 열을 방문하는 시점인 것 같다. 탐색한 경로를 바탕으로 시작 열에 방문한 좌표가 있으면 해당 좌표에 사다리가 목표 좌표에 도달하는 사다리다.

```cpp
#include <iostream>
#include <queue>
#define IO std::cin.tie(NULL), std::ios_base::sync_with_stdio(false)

#define GOAL 2
#define SIZE 100

int T = 10;

int board[SIZE][SIZE];
bool visited[SIZE][SIZE];

int dc[2] = {1, -1};

void bfs(std::pair<int, int> end) {
  std::queue<std::pair<int, int>> Q;
  Q.push(end);
  visited[end.first][end.second] = true;

  while (!Q.empty()) {
    auto [qr, qc] = Q.front();
    Q.pop();

    if (qr == 0)
      break;

    bool hasCross = false;

    for (int d = 0; d < 2; d++) {
      int nc = qc + dc[d];

      if (nc < 0 || SIZE <= nc)
        continue;

      if (visited[qr][nc])
        continue;

      if (board[qr][nc] != 1)
        continue;

      hasCross = true;
      visited[qr][nc] = true;
      Q.push({qr, nc});
    }

    if (!hasCross) {
      int nr = qr - 1;

      if (nr < 0 || SIZE <= nr)
        continue;

      if (visited[nr][qc])
        continue;

      if (board[nr][qc] != 1)
        continue;

      visited[nr][qc] = true;
      Q.push({nr, qc});
    }
  }
}

void solve() {
  int t, x;
  std::pair<int, int> end;

  while (T--) {
    std::cin >> t;

    for (int row = 0; row < SIZE; row++) {
      std::fill(visited[row], visited[row] + SIZE, false);
    }

    for (int row = 0; row < SIZE; row++) {
      for (int col = 0; col < SIZE; col++) {
        std::cin >> x;

        board[row][col] = x;
        if (x == GOAL)
          end = {row, col};
      }
    }

    bfs(end);

    std::cout << "#" << t << " ";
    for (int col = 0; col < SIZE; col++) {
      if (visited[0][col]) {
        std::cout << col << '\n';
      }
    }
  }
}

int main() {
  IO;
  solve();
  return 0;
}
```

## 개선점

### 큐랑 방문 배열은 필요 없다.

- 문제에서는 여러 경로를 탐색할 필요가 없고, 탐색 우선 순위가 있는 단일 경로 추적임.
- 좌,우로 모두 탐색 가능한 경우는 없으므로 분기가 생기지 않는다.

아래 수도 코드와 같은 방식이면 BFS 구조를 사용하면서 불러올 수 있는 오해를 줄이고, 로직의 의도를 더 명확하게 나타낼 수 있을 것이다.

```
while row > 0:
	if col - 1 >= 0 and board[row][col - 1] == 1:
		while col - 1 >= 0 and board[row][col - 1] == 1:
			col -= 1
		row -= 1
		
	else if c + 1 < SIZE and board[row][col + 1] == 1:
		while col + 1 < SIZE and board[row][col + 1] == 1:
			col += 1
		row -= 1
	
	else:
		row -= 1

return col;
```