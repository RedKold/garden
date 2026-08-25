# LogTrick（特殊性质的区间技巧）

> [!summary] 类型总结
> LogTrick 指一类利用**区间某种运算值域增长极快**的性质来剪枝的题：乘积、按位与/或、gcd 等在 O(log V) 次内就会收敛或爆炸，因此不必枚举全部 O(n²) 个区间。
>
> - 2622 和与乘积：只有 `>1` 的元素会让乘积快速增长，先单独提取它们；用前缀和 O(1) 求区间和，计算 `dis = 乘积 - 和`，再借助两侧连续 1 的个数判断能补多少个 1 并统计方案数；乘积一旦超过全局和立即 break 剪枝。
>
> 通用思路：识别"增长很快的运算"→ 提取特殊元素 → 用前缀和/前缀积 O(1) 求区间值 → 用"缺口 dis"剪枝并计数。

---
## LogTrick
[知乎-LogTrick](https://zhuanlan.zhihu.com/p/1933215367158830792z)

这是一个利用特殊性质化简迭代的问题大类。

### [蓝桥杯 2622 和与乘积 ](https://www.dotcpp.com/oj/problem2622.html)
#### 题面
给定一个数列` A = (a1, a2, · · · , an)`，问有多少个区间 `[L, R]` 满足区间内元素的乘积等于他们的和，即` aL · aL+1 · · · aR = aL + aL+1 + · · · + aR `。

##### 输入格式

输入第一行包含一个整数 `n`，表示数列的长度。  
第二行包含 `n` 个整数，依次表示数列中的数 `a1, a2, · · · , an`。  

##### 输出格式

输出仅一行，包含一个整数表示满足如上条件的区间的个数。

#### 思路

如果元素都是 `>1` 的，容易知道乘法的增长速度远大于加法。
> [!Note] 解读
> 可严格证明：如果 `[L, R]` 满足 `sum_lr < product_lr`, 则 `[L, R+k], for some k`, 都 `sum_lk < product_lk`


> 但是本题有 `1` 元素，我们单独处理。把 `>1` 的元素单挑出来。记录其下标。这样可以减少溢出


> [!Question] 怎么处理 `1` 元素呢？
> `1` 元素有个有趣的性质：在序列和 `sum_ij`、序列乘积 `product_ij` 中，**有且仅有** `1` 元素加入序列， `sum_ij` 可以单调增加 `1` 而 `product_ij` 不变。
> 这给了一个 `insight`，我们可以计算 `int dis = product_ij - sum_ij`, 即 `product` 高于 `sum` 的部分，并提前计算好每个非 `1` 的元素，**向左**有多少连续的 `1` (`L1_Count)`，**向右**则是 `R1_Count`
> - 计算连续的 `1` 的数量可以考虑用**动态规划**，在 `O(n)` 内完成。
> - 如果左右两边的 `1` 的总数不够弥补 `dis`，则可以直接 `break`，因为此段以 `i` 开头的序列无法通过左右 `+1` 来构成答案。
> - 如果可以，则利用**滑动窗口**确定滑动的边界。右边的 `1` 最多用 `r = min(rones, dis)` 个，左边的最多用 `l=min(lones, dis)` 个，总体来说方案数是滑动的，`l + r - dis +1` 个
> 	- 代码中用的是一个更严谨的方法。通过左边用多少个 ：`k`，来构造不等式来说的。道理是一样的。



```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <algorithm>
#include <cmath>
#include <cstring> // For memset, though std::vector is preferred
#define endl "\n"

using namespace std;

// N 应该略大于最大可能的元素数量
const int N_MAX = 200005;

// a 数组为原始数据数组
int a[N_MAX];
// sum 数组记录前缀和数组 (使用 long long 防止溢出，尽管元素值较小)
long long sum[N_MAX];

// b 数组为 a 中值不为 1 的元素所构成的数组
int b[N_MAX];
// id[i] 表示在 b 数组中下标为 i 的元素原始下标 (在 a 数组中的下标)
int id[N_MAX];

// L1[i] 存储紧邻 i 左侧的连续 '1' 的数量
int L1_count[N_MAX];
// R1[i] 存储紧邻 i 右侧的连续 '1' 的数量
int R1_count[N_MAX];

int n;
long long ans = 0;
int blen = 0;

void solve() {
    // 1. 输入和预处理前缀和
    if (!(cin >> n)) return;
    ans += n; // 每个单独元素 (i=j) 都是一个答案 (因为 a[i] == a[i])

    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        sum[i] = sum[i - 1] + a[i];

        if (a[i] != 1) {
            // 把不为 1 的元素放到 b 数组中去
            b[++blen] = a[i];
            id[blen] = i; // 记录原始的下标
        }
    }
    
    // 2. 预处理 L1_count 和 R1_count
    // L1_count[i]: 紧邻 a[i] 左侧连续 '1' 的数量
    for (int i = 1; i <= n; i++) {
        if (a[i] == 1) {
            // 如果 a[i] 是 1, 它的左边连续 1 的数量是它左边那个元素的数量 + 1
            L1_count[i] = L1_count[i - 1] + 1;
        } else {
            // 如果 a[i] 不是 1, 紧邻其左侧的 1 数量为 0
            L1_count[i] = 0;
        }
    }
    
    // R1_count[i]: 紧邻 a[i] 右侧连续 '1' 的数量
    for (int i = n; i >= 1; i--) {
        if (a[i] == 1) {
            // 如果 a[i] 是 1, 它的右边连续 1 的数量是它右边那个元素的数量 + 1
            R1_count[i] = R1_count[i + 1] + 1;
        } else {
            // 如果 a[i] 不是 1, 紧邻其右侧的 1 数量为 0
            R1_count[i] = 0;
        }
    }

    // 3. 遍历由非 '1' 元素构成的子数组 B[i...j]
    for (int i = 1; i <= blen; i++) {
        long long current_multi = 1;
        int idx1 = id[i]; // 左起点的原始下标

        for (int j = i; j <= blen; j++) {
            // 计算 B[i...j] 的乘积
            current_multi *= b[j];

            // 剪枝: 乘积增长过快，直接大于所有元素的最大和，后续不可能再相等
            if (current_multi > sum[n]) {
                break;
            }
            
            // 跳过 i=j (单独一个元素的子数组已在 ans+=n 中计算)
            if (i == j) continue;

            int idx2 = id[j]; // 右边界限点的原始下标

            // 计算 A[idx1...idx2] 的实际和
            long long current_sum = sum[idx2] - sum[idx1 - 1];

            // dis 是我们需要通过两侧 '1' 元素弥补的差值
            long long dis = current_multi - current_sum;

            if (dis < 0) {
                // 和大于乘积，且数组元素都 >= 1，随着 j 增大，乘积会增长更快，dis 可能会变大
                continue;
            }

            // 计算方案数
            // 需要 Dis 个 '1'，左侧有 X 个可用，右侧有 Y 个可用

            // 注意：我们取的是 idx1 左边紧邻的 1 块，和 idx2 右边紧邻的 1 块
            long long X = L1_count[idx1 - 1]; // idx1 左边紧邻的 1 块数量
            long long Y = R1_count[idx2 + 1]; // idx2 右边紧邻的 1 块数量
            
            if (X + Y < dis) {
                // 两侧 '1' 的总数不够所需差值，条件不可能成立
                continue;
            } else {
                // 有足够的 '1' 来弥补差值 Dis
                
                // k 是从左侧 X 个 '1' 中选取的数量
                /[2536. 子矩阵元素加 1](https://leetcode.cn/problems/increment-submatrices-by-one/)/ k 必须满足:
                // 1. 0 <= k <= X
                // 2. 0 <= Dis - k <= Y  => Dis - Y <= k <= Dis
                
                // k 的下限 (L_bound)
                long long L_bound = max(0LL, dis - Y);
                // k 的上限 (R_bound)
                long long R_bound = min(X, dis);
                // 实际是一个滑动窗口
                
                // 方案数 = R_bound - L_bound + 1
                // 必须保证 R_bound >= L_bound
                if (R_bound >= L_bound) {
                    ans += R_bound - L_bound + 1;
                }
            }
        }
    }

    cout << ans << endl;
}

int main() {
    // 提高 I/O 效率
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    solve();
    return 0;
}
```


