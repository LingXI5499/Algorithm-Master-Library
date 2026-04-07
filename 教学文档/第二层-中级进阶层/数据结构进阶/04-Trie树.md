# 04 - Trie 树（字典树）

## 1. 概念解释

Trie（前缀树）是一棵多叉树，每条从根到节点的路径对应一个字符串的前缀。

```
插入 "apple", "app", "ban", "band"：

        root
       /    \
      a      b
      |      |
      p      a
      |      |
      p      n
     / \      \
    #   l      d
        |      #
        e
        |
        #    （# 表示单词结束标记）
```

**核心操作**：
- **Insert**：O(|s|) 插入字符串
- **Search**：O(|s|) 查找字符串
- **StartsWith**：O(|p|) 查找前缀

---

## 2. 基本实现（数组版，竞赛常用）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e6 + 5;
int trie[MAXN][26];  // trie[node][c] = 子节点编号
bool isEnd[MAXN];
int tot = 1;  // 根节点编号为 1，0 表示空

void insert(const string& s) {
    int node = 1;
    for (char c : s) {
        int ch = c - 'a';
        if (!trie[node][ch]) trie[node][ch] = ++tot;
        node = trie[node][ch];
    }
    isEnd[node] = true;
}

bool search(const string& s) {
    int node = 1;
    for (char c : s) {
        int ch = c - 'a';
        if (!trie[node][ch]) return false;
        node = trie[node][ch];
    }
    return isEnd[node];
}

bool startsWith(const string& prefix) {
    int node = 1;
    for (char c : prefix) {
        int ch = c - 'a';
        if (!trie[node][ch]) return false;
        node = trie[node][ch];
    }
    return true;
}
```

---

## 3. 面向对象版

```cpp
class Trie {
    struct Node {
        int children[26];
        bool isEnd;
        Node() : isEnd(false) { fill(children, children+26, -1); }
    };
    vector<Node> nodes;

public:
    Trie() { nodes.emplace_back(); }

    void insert(const string& s) {
        int cur = 0;
        for (char c : s) {
            int ch = c - 'a';
            if (nodes[cur].children[ch] == -1) {
                nodes[cur].children[ch] = nodes.size();
                nodes.emplace_back();
            }
            cur = nodes[cur].children[ch];
        }
        nodes[cur].isEnd = true;
    }

    bool search(const string& s) {
        int cur = 0;
        for (char c : s) {
            int ch = c - 'a';
            if (nodes[cur].children[ch] == -1) return false;
            cur = nodes[cur].children[ch];
        }
        return nodes[cur].isEnd;
    }

    bool startsWith(const string& prefix) {
        int cur = 0;
        for (char c : prefix) {
            int ch = c - 'a';
            if (nodes[cur].children[ch] == -1) return false;
            cur = nodes[cur].children[ch];
        }
        return true;
    }
};
```

---

## 4. 0-1 Trie（XOR 最大值）

每个数按**二进制位**从高到低插入 Trie，可在 O(30) 内找到与给定数 XOR 最大的数。

```cpp
const int BITS = 30;
int trie01[MAXN * 31][2];
int tot01 = 1;

void insert01(int x) {
    int node = 1;
    for (int i = BITS; i >= 0; i--) {
        int bit = (x >> i) & 1;
        if (!trie01[node][bit]) trie01[node][bit] = ++tot01;
        node = trie01[node][bit];
    }
}

// 查询与 x XOR 的最大值
int maxXOR(int x) {
    int node = 1, res = 0;
    for (int i = BITS; i >= 0; i--) {
        int bit = (x >> i) & 1;
        int want = 1 - bit;  // 希望对应位为 1 以最大化 XOR
        if (trie01[node][want]) {
            res |= (1 << i);
            node = trie01[node][want];
        } else {
            node = trie01[node][bit];
        }
    }
    return res;
}
```

**例题：数组中两个数 XOR 的最大值（LeetCode 421）**

```cpp
int findMaximumXOR(vector<int>& nums) {
    memset(trie01, 0, sizeof(trie01));
    tot01 = 1;
    for (int x : nums) insert01(x);
    int ans = 0;
    for (int x : nums) ans = max(ans, maxXOR(x));
    return ans;
}
```

---

## 5. 统计前缀出现次数

```cpp
int cnt[MAXN];  // cnt[node]：经过该节点的单词数

void insertWithCount(const string& s) {
    int node = 1;
    for (char c : s) {
        int ch = c - 'a';
        if (!trie[node][ch]) trie[node][ch] = ++tot;
        node = trie[node][ch];
        cnt[node]++;
    }
}

// 前缀 prefix 出现的次数
int countPrefix(const string& prefix) {
    int node = 1;
    for (char c : prefix) {
        int ch = c - 'a';
        if (!trie[node][ch]) return 0;
        node = trie[node][ch];
    }
    return cnt[node];
}
```

---

## 6. 例题：单词搜索 II（LeetCode 212）

在矩阵中搜索单词列表中所有出现的单词，用 Trie 代替逐词 DFS：

```cpp
struct TrieNode {
    TrieNode* ch[26] = {};
    string word;
};

class Solution {
    void dfs(vector<vector<char>>& board, TrieNode* node, int r, int c, vector<string>& res) {
        char cc = board[r][c];
        if (cc == '#' || !node->ch[cc-'a']) return;
        node = node->ch[cc-'a'];
        if (!node->word.empty()) { res.push_back(node->word); node->word.clear(); }
        board[r][c] = '#';
        int dx[] = {0,0,1,-1}, dy[] = {1,-1,0,0};
        for (int d = 0; d < 4; d++) {
            int nr = r+dx[d], nc = c+dy[d];
            if (nr>=0 && nr<(int)board.size() && nc>=0 && nc<(int)board[0].size())
                dfs(board, node, nr, nc, res);
        }
        board[r][c] = cc;
    }
public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (auto& w : words) {
            TrieNode* cur = root;
            for (char c : w) {
                if (!cur->ch[c-'a']) cur->ch[c-'a'] = new TrieNode();
                cur = cur->ch[c-'a'];
            }
            cur->word = w;
        }
        vector<string> res;
        for (int i = 0; i < (int)board.size(); i++)
            for (int j = 0; j < (int)board[0].size(); j++)
                dfs(board, root, i, j, res);
        return res;
    }
};
```

---

## 7. 总结

| 应用 | 特点 |
|------|------|
| 字符串前缀查询 | O(|s|) 插入/查找 |
| 自动补全 | 前缀遍历 |
| 最长公共前缀 | 找分叉节点 |
| XOR 最大值 | 0-1 Trie，O(30) 查询 |
| 多模式匹配 | 配合 AC 自动机 |
