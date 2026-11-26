
## zhang yuchen

LoRA 低秩矩阵适应, Low-Rank Adaption
- As a PEFT method

- Introduce Loquetier
	- 整合了基于 *LoRA* 的 LLM 微调和推理服务的统一
- **虚拟模块**
	- 将 LoRA 改动与基座模型隔离
- 统一训推任务
	- 同时输入和微调
- 可变负载和真实负载模拟


## liang mingyu

*CalibCache*

- Background
	- 噪声去噪，还原图像
- Solution
	- 少步蒸馏
	- 特征缓存

- **预测式缓存**
	- 跳过某几步，假设第四步和第五步的差距不大，即连续可导，可以泰勒展开来估算。


- 先缓存，再纠错
	- **离线**：对齐二阶变化
	- **在线**：修正缓存残差

- TaylorSeer, HiCache, CalibCache

- **推理步数**比较低的时候，Taylor 的假设就不太成立了。


## zhang jiakun
- 逐步证明: 模型决策与 Lean 反馈

一次状态反馈。

组件消融


## Xushuo
- Rethinking Supervision
- Lean 证明


从片段边界构造监督样本。

- How to compare?

**开个并行化**。验证单机代码和分布式代码的一致性
- static analysis.



## 杂言杂语
震惊瘫坐了，曾经北大招 IMO math 的金牌做 lean 语言数据标注，4000yuan/day
