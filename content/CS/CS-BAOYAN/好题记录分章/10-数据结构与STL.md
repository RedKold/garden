# 数据结构与 STL（网易机考实战）

> [!summary] 类型总结
> STL 数据结构题考察"选对容器 + 边界细节"：需要动态有序集合时用 `set`，需要双向映射时用两个 `map`/`unordered_map`。
>
> - 守护者放置（网易机考）：`set` 维护已占位置（自动排序），每次在左边界、相邻空隙中点、右边界三处候选里选"最小距离最大"的位置；插入/删除要同步维护 id↔pos 两个映射。
>
> 通用思路：明确需要的操作（有序插入/删除/找相邻）→ 选容器 → 枚举所有候选位置取最优 → 注意边界（1 和 n）与空集合特判。

---
# 网易机考

## 守护者放置（STL 熟悉）

第一题：

放置守护者。
输入：
第一行 n, 我们守护者分布位置的坐标也是从 1-n
第二行 m
第 3 到第 m+2 行是 m 行数据，`t, id`

`1<=t<=2`, `1<=id<=n`

规则：守护者放置坐标，应该使得其到其他守护者的最小距离 **最大**

如果 t=1，就删除
t=2，就移除这个守护者，打印移除的 id


```cpp
#include <bits/stdc++.h>
using namespace std;

int n;
unordered_map<int, int> id2pos; // 使用map防止n过大
unordered_map<int, int> pos2id;
set<int> occupied_pos;          // 记录已被占据的位置，自动排序

void Solution(int t, int id) {
    if (t == 1) { // 放置守护者
        if (id2pos.count(id)) return;

        int best_pos = -1;
        int max_dist = -1;

        if (occupied_pos.empty()) {
            best_pos = 1;
        } else {
            // 1. 考虑左边界位置 1
            int first = *occupied_pos.begin();
            if (first > 1) {
                max_dist = first - 1;
                best_pos = 1;
            }

            // 2. 考虑中间空隙
            auto it = occupied_pos.begin();
            while (next(it) != occupied_pos.end()) {
                int l = *it;
                int r = *next(it);
                int d = (r - l) / 2;
                if (d > max_dist) {
                    max_dist = d;
                    best_pos = l + d;
                }
                it++;
            }

            // 3. 考虑右边界位置 n
            int last = *occupied_pos.rbegin();
            if (n - last > max_dist) {
                max_dist = n - last;
                best_pos = n;
            }
        }

        id2pos[id] = best_pos;
        pos2id[best_pos] = id;
        occupied_pos.insert(best_pos);
        cout << best_pos << endl;

    } else { // 移除守护者
		// unordered_map判断有没有元素的方式
        if (!id2pos.count(id)) return;
        int pos = id2pos[id];
        cout << pos << endl;
        
        occupied_pos.erase(pos);
        id2pos.erase(id);
        pos2id.erase(pos);
    }
}

int main() {
    ios::sync_with_stdio(false); // 优化IO
    cin.tie(NULL);
    if (!(cin >> n)) return 0;
    int m;
    cin >> m;
    while (m--) {
        int t, id;
        cin >> t >> id;
        Solution(t, id);
    }
    return 0;
}
```


测试样例：
- input:
```
7 9
1 1
1 2
1 3
2 1
1 4
2 2
1 5
1 6
1 7

```

- output
```
1
7
4
1
1
7
7
2
3
```



