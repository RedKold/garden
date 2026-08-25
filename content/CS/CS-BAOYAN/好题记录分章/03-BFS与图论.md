# BFS 与图论

> [!summary] 类型总结
> BFS 适合求**无权图的最短路/最少步数**：从起点逐层扩展，第一次到达某点即为最短。关键是**建图**与**去重**。
>
> - 3629 质数传送：把"相邻下标"和"质数可跳到的下标"建成图，用埃氏筛预处理每个数的质因数，BFS 按层计数；访问后立即清空该质数分组，避免 O(n²) 的重复检查。
>
> 通用思路：明确节点与边 → 预处理邻接关系（筛法/分组）→ BFS 逐层扩展，到达终点即返回层数 → 用 visited 防止重复入队。

---
### BFS 板子题 [3629. 通过质数传送到达终点的最少跳跃次数](https://leetcode.cn/problems/minimum-jumps-to-reach-end-via-prime-teleportation/)
给你一个长度为 `n` 的整数数组 `nums`。
你从下标 0 开始，目标是到达下标 `n - 1`。

在任何下标 `i` 处，你可以执行以下操作之一：

- **移动到相邻格子**：跳到下标 `i + 1` 或 `i - 1`，如果该下标在边界内。
- **质数传送**：如果 `nums[i]` 是一个**质数** `p`，你可以立即跳到任何满足 `nums[j] % p == 0` 的下标 `j` 处，且下标 `j != i` 。

返回到达下标 `n - 1` 所需的 **最少** 跳跃次数。

**质数** 是一个大于 1 的自然数，只有两个因子，1 和它本身。

- 这个题的关键是构造出这样一个无向图：
	- 左右挨着的下标连通
	- 按照质数规则，`nums[j] % p == 0` 的下标 `j`，且 `j != i` 的也可以连通
- 我们要返回的答案即 BFS 的层数..


#### Code
```cpp
const int MX = 1'000'001;
vector<int> prime_factors[MX];

int init = [] {
    for (int i = 2; i < MX; i++) {
        if (prime_factors[i].empty()) { // i 是prime
            for (int j = i; j < MX; j += i) {
                // insert prime
                prime_factors[j].push_back(i);
            }
        }
    }
    return 0;
}();

class Solution {
public:
    int minJumps(vector<int>& nums) {
        int n = nums.size();
        unordered_map<int, vector<int>> groups;
        // init the graph
        for (int i = 0; i < n; i++) {
            for (int p : prime_factors[nums[i]]) {
                // 质数i可以跳转到其因子的下标处
                groups[p].push_back(i);
            }
        }

        // form the bfs
        queue<int> q;
        vector<bool> vis(n, false);
        vis[0] = true;
        q.push(0);
        int ans = 0;

        while (!q.empty()) {
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                int u = q.front();

                q.pop();

                if (u == n - 1) {
                    return ans;
                }
                auto& idx = groups[nums[u]];
                idx.push_back(u + 1);
                if (u > 0) {
                    idx.push_back(u - 1);
                }
                // update queue
                for (auto j : idx) {
                    if (!vis[j]) {
                        vis[j] = true;
                        q.push(j);
                    }
                }
                
                groups[nums[u]].clear();
            }
            ans++;
        }
        return ans;
    }
};
```

#### After words..
细节：
- 我们用埃氏筛法获得可跳转的列表（建图）
- 为了不超时，你每次应该清除 `group[nums[u]]`，即 `group[nums[u]].clear()`，这样我们就不会发生重复的检查是否 `vis[j]`（因为质因子的特殊性，我们总是在第一次就访问了所有因为质因子关系的**边**，后面再访问必然不是最短路径）
- 这会使得时间复杂度从 $O(n^{2})$ 到 $O(n)$



