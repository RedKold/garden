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

- if you use `dig` command to analyze `api.deepseek.com`, you will find api request is not direct link to `api.deepseek.com` , it has mid-route
> [!Note] use `dig` to analyze | claude
> DNS 查询结果：
>   域名： api.deepseek.com
> 
>   - CNAME 记录：api.deepseek.com → api.deepseek.com.eo.dnse1.com
>   - A 记录（IP 地址）：
>     - 218.92.141.107
>     - 61.170.66.121
>   - TTL：3 秒（DNS 缓存时间很短）
>   - 查询状态：NOERROR（成功）
> 
>   该域名使用了 CDN 或智能 DNS 解析（通过 dnse1.com 服务），配置了两个 IP 地址，可能用于负载均衡或高可用


## “数据”是如何到达另一台机器的？

- **第一步：DNS 解析 - 域名-IP**
	-  `dig api.deepseek.com +short`
	- DNS 这一步已经有负载均衡了
- **第二步：packet forwarding - 逐跳到达**
	- 你的机器 -> 路由器 -> ISP -> ... -> 数据中心 -> 服务器
	- Gateway：默认网关

## 最终，到达实际的业务节点
- **业务节点收到来自本机的请求**
	- Header 中的 DEEPSEEK_API_KEY
	- Body 中的 JSON (model, messages, stream, ...)

- **麻烦的事情才真正开始**
	- 服务端要鉴权、计费、审计......
	- **后面还连着**数据库、缓存、消息队列
		- “操作系统 API”、“并发编程”——支撑了一切
		- OS labs -> Computer Science 的图景

## 数据中心：互联网 & AI 时代的幕后英雄
> Datacenter: "A network of computing and storage resources that enable the delivery of *shared* applications and data. " (CISCO)

- **上半场**：应用后端 (199X - now)
	- http -> Web 2.0 -> (iOS, Android)
	- 隐私..
- **下半场**： AI 推理 (2022 - 至今)


## **数据中心**： 一个产业链

- 市场的供需关系永远存在
	- 朝阳产业-**需要更多的人**
	- 夕阳产业-**陷入内卷**
- 技术总有一天会过时
	- “推导出这些技术”的能力 -> visions

**学校没有的“世界模型”**
- **产业结构**和**市场环境**
- 企业的本质、供应链、运转、股权、激励机制......（计金专场？！）
	- 进入社会——重新学习


## 数据中心中海量分布式数据处理

**实时的“小数据处理”**（CRUD）
- 订单事务、内容分发、用户鉴权、弹幕、计费、视频串流......

**半离线的“中数据处理”**
- 周期记账、备份、数据看板......

**离线的“大数据处理”**
- 内容索引、数据挖掘、流量分析......

**所有应用，无一例外**
- 豆包、千问、搜索、社交、支付、游戏..

---
Various using stages..


## The C10K Problem

**Dan Kegel, 1999**
> It's time for web servers to handle ten thousand clients simultaneously, don't you think? After all, the web is a big place now.


```c
while(true){
	Request *rq = get_request();
	pthread_create(&tid, NULL, handle_request, rq);
}
```

- C10M: 分布式系统

# 数据中心中的并发编程
- **高吞吐（QPS） & 低延迟的事件处理**
	- 处理事件可能需要读写持久存储或请求其他节点的服务
		- 计算机系统栈-一层一层
	- Fully locked hash table resize 就能出性能事故
- 数千/数万个请求同时到达服务器...
	- "Denial of Service, DoS"
		- **全国的小爱音箱**在小米汽车发布会上同步瘫痪

## 分布式系统难题
### CAP Theorem
- **不可能定理**
- 数据保持一致 (Consistency)、服务时刻可用（Availability）、容忍机器离线 (Partition tolerance) 不可兼得
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260515220751.png)
- e.g.
	- 不想让你妈知道出去玩，但是发朋友圈，如何做
	- 把妈妈拉黑
		- 本质是一个 http request，在系统上把朋友圈可见的数据抹掉（更改）
		- 假设到达了 **南京** 数据中心
	- 发朋友圈
		- 到达了另一个数据中心 **杭州**
- **问题**：
	- 如果妈妈想保证数据一致
		- 则在南京、杭州都刷朋友圈
		- 有可能南京发生拉黑的请求，会逐渐告诉杭州的数据中心。
	- 如果要容忍 Partition tolerance
		- 网络发生分区
	- 现在加上服务时刻可用呢？
		- 则不能做到！因为同步需要时间！
- 其他两两组合可以类比，**都不行**


## e.g. LLM Request
- Header: DEEPSEEK_API_KEY
	- 反复鉴权，billing
	- 做日志记录
- **同一个 key 还可以被多个机器使用（真正的并行）**
	- 用户可以随时 disable API key
	- 撞到了 CAP Theorem 的墙

This is very hard..


## 解决办法：重新设计系统接口
UNIX 的设计： open, read, write

### 数据 v.s. 计算
- UNIX：管道流式数据处理（**把数据带到计算**）
- 分布式系统：必须把计算分发到机器/线程（**把计算带到数据**）
	- MapReduce
### 容错和可靠性
- UNIX：假设机器和程序是可靠的
- 分布式系统：必须支持透明的网络延迟/容错

### 扩展 UNIX 的设计
- Google File System: 数据仍然是文件 (byte array)
- BigTable
	- key-value
- MapReduce
	- 计算可以 Scale out
	- 解决后台索引（OLAP, Analytical）

---

回归并行计算的本质——**描述一个计算图**
- MapReduce 也是如此
- 足够好的存储系统 + 描述数据上的计算图 = Serverless Computing
- Function as a Service: Write Once, Planet Scale

> [!Note] 幂等性 (idempotence)
幂等性指同一个操作无论执行一次还是多次，其结果和系统状态保持一致，不会产生副作用。



# 从 Token 到 Tensor
## 进入 SIMT 的世界
- `do_llm_inference()`
	- Next-token prediction
- [[OS-lab-M5|`gpt.c`]]

- 我们有各种各样新的 `tensors`（参数）
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260515224237.png)


## Attention is All You Need
**每一层 Transformer(head) 都是“一本书”**
- 每一个书都输出一个“改写后的版本”（去掉噪声）
- 看书=Q (书，要补全的内容)
- K：书的目录
- V：书的内容（具体的内容也是学出来的）


## 压榨极致的性能
- 混合精度 FMA
	- $D_{m\times n} +=A_{(m\times k)}\times B_{(k\times n)}$


- **矩阵乘法**的最大的麻烦
	- 行优先和列优先的组织方式
	- tile: 把矩阵分块

## AI token: 十万亿美元产业生态的起点


## 生成一个 Token
- $O(n^{2})$ 的 Full Attention -> sparsify, low-rank
生成一个 token 要做两件事
- Prefill：算出每一层所有的 attention “书库” (K 和 V)
	- 保存到 KV Cache
- Decode：生成输出 token
	- 读取完整 KV Cache
	- 取出 K/V ->根据当前 Q 计算“短时记忆”->生成下一个 token -> 更新 KV Cache
- **体系结构和操作系统的概念过时？**
	- 存储、互联、带宽、延迟都不一样了
- 