# 线性

## 以i结尾的状态

* [P2196 [NOIP1996 提高组\] 挖地雷 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2196)
* [P1115 最大子段和 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1115)

* [P1020 [NOIP1999 提高组\] 导弹拦截 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1020)
  * 最长递增子序列，有贪心的做法
* [P2285 [HNOI2004\] 打鼹鼠 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2285)
  * 枚举最后一个打的鼹鼠，转移的条件是时间足够
* [P1725 琪露诺 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1725)
  * 枚举最后一个格子$i$，转移的条件是$j\in[i-R,i-L]$
  * 单调队列优化，需要注意的是由于区间不连续，不能使用堆。
  * 需要多枚举几个，因为超过n都算到达终点
* [P4933 大师 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4933)
  * 枚举公差再DP能拿80
* [P1091 [NOIP2004 提高组\] 合唱队形 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1091)
  * 最长递增子序列的变体
* [P4310 绝世好题 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4310)
* [P1070 [NOIP2009 普及组\] 道路游戏 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1070)
  * 涉及到时间的一般都是1...t的状态
  * 刚开始设的是`dp[i][j]`表示前i时间停在j的最大值。但是后来发现其实用完步数之后选开始位置没有限制，所以就可以把j这一维消掉了。


# 二维

## 分别以i，j结尾的状态

* [P2758 编辑距离 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2758)

  * 相同就不改，否则枚举改哪个。注意状态初始化。

* [P1439 【模板】最长公共子序列 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1439)

  * 既然是全排列，可以离散化之后做最长递增子序列。

* [P2679 [NOIP2015 提高组\] 子串 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2679)

  * B的最后一位到底要不要跟A匹配？因此去掉了`dp[i][j-1]`的转移

  * ```
    k表示用了多少段，0/1表示是否用了A[i]，这可以决定是否分段
    dp[i][j][k][0] = dp[i-1][j][k][0] + dp[i-1][j][k][1]
    dp[i][j][k][1] =
    	if A[i]==B[j]: dp[i-1][j-1][k-1][0] + dp[i-1][j-1][k-1][1] + dp[i-1][j-1][k][1]
    	else 0
    ```

* [P1854 花店橱窗布置 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1854)

  * 花到底要不要跟盆匹配？因此去掉了削减花但不削减花盆的转移。
  * 记录路径：记录每个状态的前一个状态即可。

* [P1140 相似基因 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1140)

  * 同最长公共子序列

## 区间

* [P1435 [IOI2000\] 回文字串 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1435)
  * 同回文子序列思路
* [P1775 石子合并（弱化版） - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1775)
  * `假设[i,k]和[k+1,j]已经处理完毕，那么只需要将这两大堆合并，利用前缀和即可`。
* [Zuma - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/CF607B)
  * 需要注意的是，当两侧相等时，可以不费额外代价进行转移
* [P3205 [HNOI2010\] 合唱队 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P3205)
  * 由于当前状态的前置状态取决于前一个入队的人的大小，那么就需要额外设置状态区分前一个人是最左还是最右
* [P4170 [CQOI2007\] 涂色 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4170)
  * 两侧相等时有特殊处理。但不要直接从`dp[i+1][j-1]`转移到`dp[i][j]`
* [P4290 [HAOI2008\] 玩具取名 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4290)
  * `dp[i][j]`表示一个合法的集合，转移就是集合的笛卡尔积。
* [P1040 [NOIP2003 提高组\] 加分二叉树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1040)
  * 枚举根节点


## 环形

* [P1880 [NOI1995\] 石子合并 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1880)
  * 断环成链，枚举`dp[i][j]`的时候注意`j<min(2*n,i+n)`
* [P1063 [NOIP2006 提高组\] 能量项链 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1063)

# 背包

## 01

* [P1048 [NOIP2005 普及组\] 采药 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1048)

* [P1802 5 倍经验日 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1802)

* [P1049 [NOIP2001 普及组\] 装箱问题 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1049)

  * value和cost相同。本质上是怎么挑才能使总和更接近某个值

* [P1164 小A点菜 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1164)

* [P2392 kkksc03考前临时抱佛脚 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2392)

  * 把n个数分成两堆使得两边之差最小。假设sum为总和。
  * 假设挑出来的数为a(a<sum/2)，那么就是使得(sum-a)-a = sum-2a最小，即使得a尽量接近sum/2，那么做法同P1049

* [P1874 快速求和 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1874)

  * 对于每个空位，value为后面的数字，cost为1

* [P2340 [USACO03FALL\] Cow Exhibition G - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2340)

  * ```
    有一个比较逆天的想法：
    dp[i][j][k]表示前i个能否组成情商为j且智商为k
    dp[i][j][k] = dp[i-1][j+eq[i]][k+eq[j]] || dp[i-1][j][k]
    ```

  * value为智商，cost为情商

    ```
    dp[i][j]表示情商为j时前i个的最大智商：
    dp[i][j] = dp[i-1][j-cost]+iq[i]
    ```

  * 不需要关心正负，因为实际上是从0开始转移的。
  
* [P1441 砝码称重 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1441)

  * 先枚举所有状态，然后`dp[i][j]`表示前i个能不能凑出j
  

## 完全

* [P1616 疯狂的采药 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1616)

## 多重

* [P1077 [NOIP2012 普及组\] 摆花 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1077)
  * 枚举每种物品拿多少个
* [P1833 樱花 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1833)
* [P1541 [NOIP2010 提高组\] 乌龟棋 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1541)
  * 本质上是每一步到底选哪张牌的问题。

## 依赖

### 树形

* [P2015 二叉苹果树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2015)
* [P2014 [CTSC1997\] 选课 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2014)

### 不需要树形

* [P1064 [NOIP2006 提高组\] 金明的预算方案 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1064)
  * 对于每个树根，枚举子树的形态

# 矩阵

## 记忆化搜索

* [P1434 [SHOI2002\] 滑雪 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1434)

## 限定转移方向

* [P1002 [NOIP2002 普及组\] 过河卒 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1002)
* [P3842 [TJOI2007\] 线段 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P3842)
  * 从线段的哪个端点转移的
* [P1004 [NOIP2000 提高组\] 方格取数 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1004)
  * 同时转移两次即可。

# 图上DP

## Floyd

* [P1613 跑路 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1613)
  * `dp[i][j][o]`表示i到j有2^k的路径
  * `dp[i][j][o] |= dp[i][k][o-1]&&dp[k][j][o-1]`
  * 若`dp[i][j][any o]`那么两点之间就有边。在新图上跑最短路。

## DAG

* [P4017 最大食物链计数 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4017)
  * topo+dp
* [P4316 绿豆蛙的归宿 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P4316)
  * 反向建图，按topo序dp


# 树上DP

## 子树之间没有约束

* [P1352 没有上司的舞会 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1352)
  * 容易想到`dp[0/1][i]`表示以i为根选不选i的最大值
  * 转移`dp[0][i]`时会想到枚举子树的状态，到底选还是不选
  * 但是实际上，由于子树状态之间没有依赖关系，因此`dp[0][u]=sum{max(dp[1][v],dp[0][v])}`即可，不需要枚举子树的状态
* [P1122 最大子树和 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1122)
  * 如果子树的dp值为正就选出来。
  
* [P2016 战略游戏 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2016)
  * `dp[u][0/1]`表示u上放不放士兵的最小值
  

## 子树之间有约束

* [P2015 二叉苹果树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2015)
  * 由于是二叉树，枚举子树状态很方便，可以直接用`dp[u][i]`表示u为根使用i个树枝的最大值
  * `dp[u][i] = max(dp[v1][k]+dp[v2][i-k-2],dp[v1][i-1],dp[v2][i-1])`，注意k的范围
  * 记忆化搜索可能更好写。
* [P2014 [CTSC1997\] 选课 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2014)
  * 由于是多叉树，不太好枚举子树的状态。所以上一题的做法行不通。
  * `dp[u][i][j]`表示根为u，前i个节点中选择j门课的最大价值。
  * `dp[u][i][j] = max(dp[u][i-1][j],dp[u][i-1][k]+dp[v][sizeof(v)][j-k])`，其中`j>=k>1`，因为根节点必须选。
  * 可以把i这一维压掉，注意倒序枚举j
* [P2585 [ZJOI2006\] 三色二叉树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2585)
  * 由于是二叉树，可以枚举子树的状态
* [P1273 有线电视网 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1273)
  * `dp[u][i][j]`表示u为根前i个结点服务j个叶子的最大价值
  * 对于每个叶子`dp[u][1][1]=value[u]`
  * `dp[u][j]=max(dp[u][j],dp[u][k]+dp[v][j-k]-cost(u,v))`

## 换根

* [P2986 [USACO10MAR\] Great Cow Gathering G - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2986)

  * 随便找一个根，比如0.那么`dp[i]`表示在这种有向树下，以i为根的代价。

  * ```
    dp[u] = sum{dp[v]+cnt[v]*w}
    cnt[u] = sum{cnt[v]}
    ```

  * 从0开始进行dfs，对于每一个邻居进行换根

# 状压

## 矩阵

将每一行的状态压缩为int

* [P1896 [SCOI2005\] 互不侵犯 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1896)
  * `dp[i][s]`表示考虑前i行并且第i行的状态为s的最大值
  * `dp[i][s] = max{dp[i-1][valid s]}+getnum(s)`
* [P1879 [USACO06NOV\] Corn Fields G - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1879)
* [P2704 [NOI2001\] 炮兵阵地 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2704)
  * 状态需要存两列。`dp[i][s1][s2]`表示i-1行状态为s1，i行为s2

## 图

* [P2622 关灯问题II - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2622)
  * 状压之后求最短路，可以不显式存图
* [P2761 软件补丁问题 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P2761)
  * 状压之后求最短路，显式存图超空间