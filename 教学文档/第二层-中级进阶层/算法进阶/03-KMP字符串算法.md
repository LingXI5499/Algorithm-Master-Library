# 03 - KMP 与字符串算法

## 1. KMP 算法

### 概念

KMP（Knuth-Morris-Pratt）算法用于**字符串模式匹配**：在文本串 `text` 中查找模式串 `pattern` 的所有出现位置。

**核心思想**：当发生失配时，利用模式串自身的前后缀信息（`next` 数组），将模式串向右滑动，而不是从头重新匹配。

时间复杂度：**O(n + m)**（n = 文本长度，m = 模式长度）

### next 数组定义

`next[i]` = 模式串 `pattern[0..i]` 中，**最长的相等真前缀与真后缀的长度**。

```
pattern = "aabaaab"
next  =  [0, 1, 0, 1, 2, 2, 3]
              ↑
          pattern[0..1]="aa" 最长相等前后缀="a"，长度1
```

### next 数组构建

```cpp
vector<int> buildNext(const string& p) {
    int m = p.size();
    vector<int> next(m, 0);
    int j = 0;  // 前缀指针
    for (int i = 1; i < m; i++) {
        while (j > 0 && p[i] != p[j]) j = next[j - 1];
        if (p[i] == p[j]) j++;
        next[i] = j;
    }
    return next;
}
```

### KMP 匹配

```cpp
vector<int> kmpSearch(const string& text, const string& pattern) {
    int n = text.size(), m = pattern.size();
    vector<int> next = buildNext(pattern);
    vector<int> result;

    int j = 0;  // 模式串指针
    for (int i = 0; i < n; i++) {
        while (j > 0 && text[i] != pattern[j]) j = next[j - 1];
        if (text[i] == pattern[j]) j++;
        if (j == m) {
            result.push_back(i - m + 1);  // 找到匹配，记录起始位置
            j = next[j - 1];              // 继续找下一个
        }
    }
    return result;
}
```

### 完整示例

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string text = "aabaabaabaab";
    string pat  = "aab";
    auto positions = kmpSearch(text, pat);
    for (int pos : positions)
        cout << "Found at index " << pos << "\n";
    // 输出：0, 3, 6, 9
    return 0;
}
```

### KMP 应用：最小循环节

字符串 `s` 的最小循环节长度 = `n - next[n-1]`（若能整除则是真循环节）。

```cpp
int minPeriod(const string& s) {
    int n = s.size();
    vector<int> next = buildNext(s);
    return n - next[n - 1];
}
```

---

## 2. Z 算法（扩展 KMP）

### 概念

Z 数组：`z[i]` = `s[i..n-1]` 与 `s[0..n-1]` 的**最长公共前缀（LCP）**的长度。

```
s = "aabxaa"
z = [0, 1, 0, 0, 2, 1]
        ↑           ↑
    s[1..] = "abxaa"  LCP="a"，长度1
    s[4..] = "aa"     LCP="aa"，长度2
```

### Z 数组构建

```cpp
vector<int> zFunction(const string& s) {
    int n = s.size();
    vector<int> z(n, 0);
    z[0] = n;
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = min(r - i, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

### Z 算法模式匹配

```cpp
// 在 text 中找 pattern 的所有位置
vector<int> zSearch(const string& text, const string& pattern) {
    string s = pattern + "#" + text;  // '#' 不在字母表中
    auto z = zFunction(s);
    int m = pattern.size();
    vector<int> result;
    for (int i = m + 1; i < (int)s.size(); i++)
        if (z[i] == m) result.push_back(i - m - 1);
    return result;
}
```

---

## 3. 字符串哈希

### 概念

将字符串映射为一个数值，用于 **O(1)** 比较任意子串是否相等。  
通常用**多项式滚动哈希**：

```
hash(s[0..n-1]) = s[0]*B^(n-1) + s[1]*B^(n-2) + ... + s[n-1]  (mod M)
```

### 实现（双哈希防碰撞）

```cpp
struct StringHash {
    const long long B1 = 131, M1 = 1e9+7;
    const long long B2 = 137, M2 = 1e9+9;
    int n;
    vector<long long> h1, h2, p1, p2;

    StringHash(const string& s) : n(s.size()), h1(n+1), h2(n+1), p1(n+1), p2(n+1) {
        p1[0] = p2[0] = 1;
        for (int i = 0; i < n; i++) {
            h1[i+1] = (h1[i] * B1 + s[i]) % M1;
            h2[i+1] = (h2[i] * B2 + s[i]) % M2;
            p1[i+1] = p1[i] * B1 % M1;
            p2[i+1] = p2[i] * B2 % M2;
        }
    }

    // 获取 s[l..r] 的哈希（0-indexed）
    pair<long long,long long> get(int l, int r) {
        long long v1 = (h1[r+1] - h1[l] * p1[r-l+1] % M1 + M1 * 2) % M1;
        long long v2 = (h2[r+1] - h2[l] * p2[r-l+1] % M2 + M2 * 2) % M2;
        return {v1, v2};
    }
};
```

### 应用：最长回文子串（O(n log n)）

```cpp
int longestPalindromeHash(const string& s) {
    string rev = s; reverse(rev.begin(), rev.end());
    StringHash h(s), hr(rev);
    int n = s.size(), ans = 1;

    auto isPalin = [&](int l, int r) {
        int rl = n - 1 - r, rr = n - 1 - l;
        return h.get(l, r) == hr.get(rl, rr);
    };

    // 枚举中心，二分长度
    for (int c = 0; c < n; c++) {
        // 奇数长度
        int lo = 0, hi = min(c, n-1-c);
        while (lo < hi) {
            int mid = lo + (hi - lo + 1) / 2;
            if (isPalin(c-mid, c+mid)) lo = mid;
            else hi = mid - 1;
        }
        ans = max(ans, 2*lo+1);
        // 偶数长度
        if (c+1 < n) {
            lo = 0; hi = min(c, n-2-c);
            while (lo < hi) {
                int mid = lo + (hi - lo + 1) / 2;
                if (isPalin(c-mid+1, c+mid)) lo = mid;
                else hi = mid - 1;
            }
            ans = max(ans, 2*lo);
        }
    }
    return ans;
}
```

---

## 4. Manacher 算法（最长回文子串，O(n)）

```cpp
string longestPalindrome(string s) {
    // 插入分隔符
    string t = "#";
    for (char c : s) { t += c; t += '#'; }
    int n = t.size();
    vector<int> p(n, 0);
    int c = 0, r = 0;

    for (int i = 0; i < n; i++) {
        if (i < r) p[i] = min(r - i, p[2*c - i]);
        while (i-p[i]-1 >= 0 && i+p[i]+1 < n && t[i-p[i]-1] == t[i+p[i]+1])
            p[i]++;
        if (i + p[i] > r) { c = i; r = i + p[i]; }
    }

    int maxLen = 0, center = 0;
    for (int i = 0; i < n; i++)
        if (p[i] > maxLen) { maxLen = p[i]; center = i; }

    return s.substr((center - maxLen) / 2, maxLen);
}
```

---

## 5. 总结

| 算法 | 功能 | 时间复杂度 |
|------|------|-----------|
| KMP | 单模式匹配 | O(n+m) |
| Z 算法 | 单模式匹配 | O(n+m) |
| 字符串哈希 | 子串比较 | 预处理 O(n)，查询 O(1) |
| Manacher | 最长回文子串 | O(n) |
| AC 自动机 | 多模式匹配 | O(n + Σ|patterns|) |
