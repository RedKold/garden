# 一个 Token 的旅程

## 回顾 “并发编程”

- 从 `spawn(T_worker)` 和 join 开始
    - 一切的基础：**计算图模型**
    - 共享内存、互斥锁、条件变量、信号量的底层机制
- 面向 “异构” 复杂 T_worker 的改进
    - 关键问题：管理状态、同步、复杂性
        - Coroutine, goroutine
        - Promise, async/await
- 面向 “同构” 小而短 T_worker 的改进
    - 关键问题：最大化计算密度和能效比
        - SIMD (数据并行)
        - 从图形处理器到到 CUDA/SIMT


## 用户的视角：

一个 LLM Request
 - 通过 Request 库请求 DeepSeek API endpoint，就能启动和 AI 的聊天
 - 新时代的基础设施


## “数据”是如何到达另一台机器的？

- **第一步：DNS 解析 - 域名-IP**
	-  `dig api.deepseek.com +short`
	- DNS 这一步已经有负载均衡了
- **第二步：packet forwarding - 逐跳到达**
	- 你的机器 -> 路由器 -> ISP -> ... -> 数据中心 -> 服务器
	- Gateway：默认网关
	- 