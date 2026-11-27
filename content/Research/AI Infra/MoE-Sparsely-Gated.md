
在 Sparsely-Gated MoE 里，每个 token 的隐藏状态进入一个可训练的路由器（gate），路由器选出得分最高的 k 个专家。模型只运行这些专家，再对它们的输出做加权求和。两个 token 可以执行同样数量的专家，却走不同的计算路径

---

Why need this:
- capacity of a neural network to absorb information is limited by its number of parameters.
- but more parameters face more challenge.

- Introduce a Sparse-Gated Mixture-of-Experts layer (MoE)


- **条件计算** (Conditional computation)
	- as a way to increase model capacity without a proportional increase in computational costs
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20261127162047.png)
	- 划分给其他专家做计算

- **解决的问题**
	- Computing devices (especially GPUs) are much faster at arithmetic that at branching.
		- conditional computation definitely need *branching*
	- Large batch size are critical for performance
		- *amortize the costs* of parameter transfers and updates
		- *conditional computation* reduces that
	- *Network* bandwidth can be a bottleneck
	- Model capacity is most critical for very large data sets


## The Sparsely-Gated Mixture of Experts Layer

focus on language modeling and machine translation tasks

2017 年论文啊
- 把 MoE 作为通用神经网络组件
- 将 MoE 应用于卷积 (convolutional application of the MoE)
	- allows for different gating decisions at each position in the text.



- Structure of *MoE* layer
	- a set of $n$ "expert networks", $E_{1},\dots,E_{n}$
		- itself a neural networks.
	- gating network $G$
		- output: a sparse $n$ -dimensional vector
	- output $y$ of the MoE module can be written as follows:
$$
y=\sum_{i=1}^{n}G(x)_{i}E_{i}(x)
$$
- 如果 experts 太多可以用 two-level hierarchical MoE。
	- primary gating network chooses a *sparse* weighted combination of "experts"


### Gating Network
- traditional
	- Softmax Gating
$$
Ga_{\sigma}(x) = Softmax(s\cdot W_{g})
$$

- Noisy Top-K Gating
	- add 2 components to the Softmax gating network
		- *sparsity and noise*
	- keep only the *top k values*
		- setting the rest to $-\infty$, which causes the corresponding gate values to equal 0
$$
\begin{align}
G(x)=Softmax(\text{KeeptopK}(H(x),k)) \\
H(x)_{i}=(x\cdot W_{g})_{i}+ \text{StandardNormal}()\cdot Softplus((x\cdot W_{noise})_{i}) \\
KeepTopK(v,k)_{i}=  
\begin{cases}
v_{i},\;&\text{if $v_{i}$ is in the top $k$ elements of $v$}\\
-\infty,&\;\text{otherwise}
\end{cases}
\end{align}
$$
- **The sparsity serves to save computation**


## Addressing Performance Challenges

