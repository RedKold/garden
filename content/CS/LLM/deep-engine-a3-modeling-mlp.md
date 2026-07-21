## Task 1 Dense MLP With LoRA

Don't know why: test in local toy test, failure. but test on ref, pass.

*Multi-Layer Perceptron*.
- widely used in LLMs based on Transformer

$$
\text{MLP}(\mathbf{X}) = (\phi(\mathbf{X} \times \mathbf{W}_{gate}) \odot (\mathbf{X} \times \mathbf{W}_{up})) \times \mathbf{W}_{down}
$$
- $\mathbf{X}$: input `hidden states`, where $\mathbf{X}\in \mathbb{R}^{\text{batch\_size} \times\text{seq\_len}\times\text{hidden\_size}}$, noted as `[b,s,d]`
- $\mathbf{W}_{up}$:  上投影矩阵，满足 $\mathbf{W}_{up}\in \mathbb{R}^{\text{hidden\_size}\times\text{ffh}}$, 把 $\mathbf{X}$ 从 `h` 维映射到 `ffh` 维度，记为 `[d, ffh]`
- $\mathbf{W}_{down}$ ：下投影矩阵，满足 $\mathbf{W}_{down}\in \mathbb{R}^{\text{ffh}\times\text{hidden\_size}}$, 将 $\mathbf{X}$ 从 `ffh` 维映射回 `h` 维，记为 `[ffh, d]
- $\mathbf{W}_{gate}$ 表示门控投影矩阵，满足 $\mathbf{W}_{gate}\in \mathbb{R}^{\text{hidden\_size}\times \text{ffh}}$. 引入非线性变换。
	- 配合**激活函数**形成门控项 $\phi(\mathbf{X}\times \mathbf{W}_{gate})$

- $\odot$ 表示 element-wise 乘

LoRA 涉及到低秩分解



## Task 2: Sparse MLP
所谓 *dense*，指的是标准结构。先把 `hidden_states` $\mathbf{X}$ 从 `h` 维上投影 (up-project) 到更高的 `ffh` 维度，再通过 `gating` 机制下投影 (down-project) 回原始维度

*Sparse*：
- 类似于 multi-head 机制

将投影矩阵的 `ffh` 维度划分为 `ne` 个大小相等的 `shard`（分片）
- each `shard` size is `e = ffh // ne`, responding to one 'Expert'
- in `SparseMLPWithLoRA`, in  `hidden_states` $\mathbf{X}$ , each `token` only use a *routing mechanism* to map to `k` experts.
- each expert only responsible for deal with specific `e` -dim sub-space.
	- in this module, you can simply model each *expert* as a small *DenseMLPWithLoRA* module, where `ffh_size` param is set as `e`
	- each `token` 's final output is weighted-sum of `k`  *experts*

	- 并行子空间学习能力


> [!Question] How to model routing mechanism? 
> How can we choose `k` specific experts for each token?


> [!Question] How to identify *weight* $\mathbf{W}$
> the weighted-sum need a *weight* matrix

---

Introduce a added linear `gating` layer $\mathbf{G}$, fit $\mathbf{G}\in \mathbb{R}^{h\times ne}$, for each token $t$, hold $t\in \mathbb{R}^{b\times {1}\times h}$, use $\mathbf{G}$ to project `hidden_states` to a `ne` -dim `logits`, do softmax for that `logits`, get a `ne` -dim routing Prob distribution $\mathbf{P}_{t}$, where $\mathbf{P}_{t}[i]$ express the prob that `token` is routed to expert $E_{i}$

$$
\mathbf{P_{t}}= \mathrm{softmax}(\mathbf{X}_{t} \times \mathbf{G}),where\;\mathbf{G}\in \mathbb{R}^{h\times ne},\forall t
$$

- [ ] Based $\mathbf{P}_{t}$, now we find the `k`  **experts** with highest prob, form a new set $\mathbf{I}_{t}$, as the **router** of that `token`. These `k` prob value will form a new k-dim not normalization distribution $\mathbf{Q}_{t}$

- [ ] Do **renormalization**(here we do it by  simply divide the sum ) for $\mathbf{Q}$, define a new `k` -dim routing Prob Distribution, so we can get the $\mathbf{W}_{t}$ about that `token` 's each expert, and the output of each expert $\mathbf{E}_{i}$

$$
\mathbf{W}_{t}={\frac{\mathbf{Q}_{t}}{sum(\mathbf{Q}_{t})},\forall t}
$$

$$
\mathbf{O}_{t}'[i]=E_{i}(\mathbf{X}_{i}),\forall i\in \mathbf{I}_{t},\forall t
$$

To simulate the *distributed* environment, we add two similar parameters: `rank` and `world_size` .(check a2).
You need to instantize `nle` local *experts*, the index range of `expert` is $R=[rank \cdot nle,(rank+1)nle]$, where `nle = ne // world_size`.
- `nle` is the amount of local experts of each *process*

So for each token `t`, `SparseMLPWithLoRA` only output a *partial* sum.


the final *completed* output is to gather all output of each `rank`, but in this lab we leave out this.

$$
\mathbf{O}_{t}=\begin{cases}
\sum_{i\in \mathbf{I}'_{t}}\mathbf{W _{t}}[i]\mathbf{O}_{t}'[i],\; & \mathbf{I_{t}' \neq \emptyset} \\
\vec{\mathbf{0}}, & \mathbf{I_{t}' \neq \emptyset}
\end{cases}
where\; \mathbf{I'_{t}=I_{t}\cap }R,\forall t
$$

