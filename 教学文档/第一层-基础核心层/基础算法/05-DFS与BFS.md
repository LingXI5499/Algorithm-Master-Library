# 05 - DFS 与 BFS

## 1. 深度优先搜索（DFS）

### 概念

DFS 沿着一条路径尽可能深入，直到无法继续时**回溯**，再尝试其他路径。

```
图：1-2, 1-3, 2-4, 2-5
DFS 从 1 出发：1 → 2 → 4 → (回溯) → 5 → (回溯) → 3
```

**特点**：
- 使用**栈**（或递归的系统栈）
- 空间复杂度：O(树高 h) 或 O(V) 最坏
- 适合：路径存在性、连通性、拓扑排序、回溯

### 图的 DFS 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e5 + 5;
vector<int> adj[MAXN];
bool visited[MAXN];

void dfs(int u) {
    visited[u] = true;
    // 处理节点 u 的逻辑
    for (int v : adj[u]) {
        if (!visited[v]) dfs(v);
    }
}
```

### 网格（矩阵）DFS

```cpp
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

void dfsGrid(vector<vector<char>>& grid, int r, int c) {
    int n = grid.size(), m = grid[0].size();
    if (r < 0 || r >= n || c < 0 || c >= m) return;
    if (grid[r][c] != '1') return;
    grid[r][c] = '0';  // 标记已访问（原地修改）
    for (int d = 0; d < 4; d++)
        dfsGrid(grid, r + dx[d], c + dy[d]);
}

// 岛屿数量（LeetCode 200）
int numIslands(vector<vector<char>>& grid) {
    int ans = 0;
    for (int i = 0; i < (int)grid.size(); i++)
        for (int j = 0; j < (int)grid[0].size(); j++)
            if (grid[i][j] == '1') {
                dfsGrid(grid, i, j);
                ans++;
            }
    return ans;
}
```

---

## 2. 回溯法

### 概念

回溯是 DFS 的延伸，在搜索时**做选择 → 深入 → 撤销选择**，枚举所有可能的解。

**模板**：

```cpp
void backtrack(参数) {
    if (终止条件) {
        // 保存结果
        return;
    }
    for (每一个选择) {
        做选择;
        backtrack(下一层);
        撤销选择;
    }
}
```

### 全排列（LeetCode 46）

```cpp
vector<vector<int>> permute(vector<int>& nums) {
    vector<vector<int>> res;
    vector<bool> used(nums.size(), false);
    vector<int> path;

    function<void()> bt = [&]() {
        if (path.size() == nums.size()) { res.push_back(path); return; }
        for (int i = 0; i < (int)nums.size(); i++) {
            if (used[i]) continue;
            used[i] = true; path.push_back(nums[i]);
            bt();
            path.pop_back(); used[i] = false;
        }
    };
    bt();
    return res;
}
```

### 子集（LeetCode 78）

```cpp
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> res;
    vector<int> path;

    function<void(int)> bt = [&](int start) {
        res.push_back(path);
        for (int i = start; i < (int)nums.size(); i++) {
            path.push_back(nums[i]);
            bt(i + 1);
            path.pop_back();
        }
    };
    bt(0);
    return res;
}
```

### N 皇后（LeetCode 51）

```cpp
vector<vector<string>> solveNQueens(int n) {
    vector<vector<string>> res;
    vector<string> board(n, string(n, '.'));
    vector<bool> col(n), diag1(2*n), diag2(2*n);

    function<void(int)> bt = [&](int row) {
        if (row == n) { res.push_back(board); return; }
        for (int c = 0; c < n; c++) {
            if (col[c] || diag1[row-c+n] || diag2[row+c]) continue;
            board[row][c] = 'Q';
            col[c] = diag1[row-c+n] = diag2[row+c] = true;
            bt(row + 1);
            board[row][c] = '.';
            col[c] = diag1[row-c+n] = diag2[row+c] = false;
        }
    };
    bt(0);
    return res;
}
```

---

## 3. 广度优先搜索（BFS）

### 概念

BFS 按**层**扩展，先访问距起点近的节点，适合求**最短路径**（无权图）。

**特点**：
- 使用**队列**
- 空间复杂度：O(V)
- 适合：最短路、最少步数、最小操作次数

### 图的 BFS 模板

```cpp
void bfs(int start, vector<int> adj[]) {
    vector<int> dist(MAXN, -1);
    queue<int> q;
    dist[start] = 0;
    q.push(start);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
}
```

### 网格最短路（LeetCode 1091）

```cpp
int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
    int n = grid.size();
    if (grid[0][0] || grid[n-1][n-1]) return -1;
    int dirs[8][2] = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    queue<pair<int,int>> q;
    q.push({0, 0});
    grid[0][0] = 1;
    int steps = 1;
    while (!q.empty()) {
        int sz = q.size();
        while (sz--) {
            auto [r, c] = q.front(); q.pop();
            if (r == n-1 && c == n-1) return steps;
            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0) {
                    grid[nr][nc] = 1;
                    q.push({nr, nc});
                }
            }
        }
        steps++;
    }
    return -1;
}
```

### 多源 BFS（LeetCode 542，01 矩阵）

同时从所有 0 出发，计算每个格子到最近 0 的距离：

```cpp
vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    vector<vector<int>> dist(n, vector<int>(m, INT_MAX));
    queue<pair<int,int>> q;
    // 多源：将所有 0 同时入队
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            if (mat[i][j] == 0) { dist[i][j] = 0; q.push({i, j}); }

    int dx[] = {0,0,1,-1}, dy[] = {1,-1,0,0};
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        for (int d = 0; d < 4; d++) {
            int nr = r + dx[d], nc = c + dy[d];
            if (nr>=0 && nr<n && nc>=0 && nc<m && dist[nr][nc] > dist[r][c] + 1) {
                dist[nr][nc] = dist[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
    return dist;
}
```

---

## 4. DFS vs BFS 对比

| 维度 | DFS | BFS |
|------|-----|-----|
| 数据结构 | 栈（递归）| 队列 |
| 空间复杂度 | O(h) | O(宽度) |
| 最短路 | 无法保证 | 可以（无权图）|
| 连通性 | 可以 | 可以 |
| 适合 | 枚举所有路径、回溯 | 最短步数、层次遍历 |

> **经验**：求最少操作次数 → BFS；枚举所有方案 → DFS+回溯
