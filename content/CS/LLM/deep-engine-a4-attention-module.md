# Task 1: Offline Sliding-Window Attention
**Multi-head Attention** module is a important part of *Transformer*
it hold 3 tensor as *input*
- Query tensor , $\mathbf{Q}$, $\mathbf{Q}\in \mathbb{R}^{\text{batch\_size}\times \text{seq\_len\_q}\times \text{num\_head}\times\text{head\_dim}}$. noted as `[b, sq, hq, hd]`
- Key tensor and Value tensor, noted as $\mathbf{K,V}$, they have the *same* shape, hold $\mathbf{K,V} \in \mathbb{R}^{\text{batch\_size}\times \text{seq\_len\_kv}\times\text{num\_head\_kv}\times\text{head\_dim}}$, noted as `[b, skv, hkv, hd]`

> [!Note] The *batch-like* dimension of Multi-head Attention
> $batch\_size$ and $num\_head$, they are all *batch-like* dim

for `seq_len`, every tensor $\mathbf{q}_{i}\in \mathbf{Q}$, can be seen as the $i$ -th `token` 's *embedded latent query message*.

Every tensor $\mathbf{v_{j}}$ can be seen as the `j` -th `token` 's *embedded latent knowledge*

Every $\mathbf{v_{j}}$ is to a embedded latent key tensor $\mathbf{k_{j}}$, 
so we can calculate the dot-product $\mathbf{q_{i}k_{j}^{T}}$ to get the *similarity* score.

The gathered result of each $\mathbf{q_{i}}$ is $\mathbf{o_{i}}$, they express the weighted-sum of $\mathbf{v_{j}}\in \mathbf{V}$ .

$$
\mathbf{o_{i}}=\sum_{j}\mathbf{a}_{j}^{(i)}\mathbf{v_{j}}
$$
where $\mathbf{a}^{(i)}$ is the *normalized dot-product similarity* of each $\mathbf{q_{i}}$ and all $\mathbf{k_{j}}$
- how to normalized? most usually- `softma`

$$
\mathrm{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})=\mathbf{A}\times \mathbf{V} 
$$

where $\mathbf{A}=\mathrm{softmax_{row-wise}(scale \cdot \mathbf{P})}\in \mathbb{R}^{\text{sq}\times \text{skv}}$
$$
\mathbf{P}=\mathbf{Q\times K^{|}}+\mathbf{M}\in \mathbb{R}^{\text{sq}\times\text{skv}}
$$

- $\mathbf{M}$ is a two-value `Attention Mask` , every element of it $\in \{ -\infty , 0 \}$
	- if two $(\mathbf{q_{i},k_{j}})$ is un-relevant, `Mask` is set to $-\infty$, so the `softmax` will set near 0
	- else :, set to `0`

![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20260722144741.png)

- `softmax` is sensitive to value change.
- some methods to maintain 
	1. scale $\mathbf{P}$, that is $scale=\frac{1}{\sqrt{ hd }}$
	2. Softmax Temperature, form: $\frac{\mathbf{P}}{temp}$, where `temp` is a *super-parameter* ranging $(0,+\infty)$.
		1. if  $temp = 1.0$, `softmax` is the original distribution
		2. if $temp\to 0.0$, `softmax` would be more *sharp*, (one-hot)
		3. if $temp \to +\infty$, then `softmax` would be more smooth (uniform)
	3. **Softmax Capping**, $\mathrm{cap}\cdot \mathrm{\tanh}\left( \frac{\mathbf{P}}{cap} \right)$, `cap` is a large positive *number*.
		1. it and softmax temperature only one need to be used
	4. **Softmax Clipping**, to restrain the *outliers* increase too much, we can do *softmax Clipping*, it's to $\mathbf{A}_{\text{clipped}}=\mathrm{clip}((r-l)\cdot A+l,0,1)$
		1. $\mathbf{A}$ is the original `softmax` output, ranging from $[0,1]$
		2. $[l,r]$ is a super-range, $l\leq{0}, r\geq {1}$
		3. map $\mathbf{A}$ from $[0,1]$ to $[l,r]$, then *clip* back to $[0,1]$
	5. **QK layer Normalization**, to improve the *robustness* of attention *weight* of $\mathbf{A}$, we can apply `softmax dropout` to $\mathbf{A}$, form: $\mathbf{A}_{dropout}=\mathrm{dropout}_{p}(\mathbf{A})$
		- $p \in[0,1]$
		- will set partial weight to 0 in $\mathbf{A}$, and scale remaining part to keep the same **sum**
		-  use a2's `GroupRMSNorm`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260722150106.png)

here we define a *enumerate type* `AttnQKVPackFormat`

