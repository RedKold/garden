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

LoRA 涉及到**低秩分解**



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



### Code

```python
class SparseMLPWithLoRA(nn.Module):
    """Sparse MLP module with LoRA adapters
    This is a GLU-style sparse MLP layer with LoRA adapters, \
        where the sparcity is implemented as Mixture of Experts (MoE), \
            and each expert is a dense MLP with LoRA adapters.
    """
    
    def __init__(self,
        hidden_size: int,
        ffh_size: int,
        activation_type: MLPActivationType = MLPActivationType.SILU,
        num_experts: int = 1,
        moe_topk: int = 1,
        rank: int = 0,
        world_size: int = 1,
        process_group: Optional[ProcessGroup] = None,
        init_mean: float = 0.0,
        init_std: float = 1.0,
        init_base_seed: int = 42,
        lora_rank: int = 0,
        lora_alpha: Optional[float] = None,
        lora_dropout_rate: float = 0.0,
        lora_dropout_seed: int = 42,
        lora_init_base_seed: int = 42,
        dtype: torch.dtype = torch.float32,
        device: str = "cpu",
    ):
        """Initialize Sparse MLP module with LoRA adapters
        
        Args:
            hidden_size(int): hidden dimension size
            ffh_size(int): hidden dimension size
            activation_type(MLPActivationType, default = MLPActivationType.SILU): activation type
            num_experts(int, default = 1): number of (global) experts, which can deduce expert_size = ffh_size // num_experts
            moe_topk(int, default = 1): topk-routing for MoE to control the sparcity
            rank(int, default = 0): rank
            world_size(int, default = 1): world size
            process_group(Optional[ProcessGroup], default = None): the process group (which will not be used for this simpler module yet)
            init_mean(float, default = 0.0): mean for the initialization
            init_std(float, default = 1.0): std for the initialization
            init_base_seed(int, default = 42): seed for the initialization
            lora_rank(int, default = 0): lora rank
            lora_alpha(Optional[float], default = None): lora alpha
            lora_dropout_rate(float, default = 0.0): lora dropout rate
            lora_dropout_seed(int, default = 42): lora dropout seed
            lora_init_base_seed(int, default = 42): seed for lora weight initialization
            dtype(torch.dtype, default = torch.float32): parameter dtype
            device(str, default = "cpu"): parameter device
        """
        super().__init__()

        self.hidden_size = hidden_size
        self.ffh_size = ffh_size
        self.activation_type = activation_type
        self.num_experts = num_experts
        self.moe_topk = moe_topk
        self.rank = rank
        self.world_size = world_size
        self.process_group = process_group
        self.init_mean = init_mean
        self.init_base_seed = init_base_seed
        self.init_std = init_std
        
        self.lora_rank = lora_rank
        self.lora_alpha = lora_alpha
        self.lora_dropout_rate = lora_dropout_rate
        self.lora_dropout_seed = lora_dropout_seed
        self.lora_init_base_seed = lora_init_base_seed

        self.dtype = dtype
        self.device = device

        
        # introduce additional linear gating layer G: [h, ne]
        assert ffh_size % num_experts == 0, \
            f"ffh_size({ffh_size}) cannot be divided by num_experts({ne})!"
        
        self.e = ffh_size // num_experts
        
        self.G = nn.Parameter(torch.empty(hidden_size, num_experts, device = device))   # G's dytpe is always float32. won't be affected by the parm dytpe

        # the amount of local experts in each process
        self.nle = num_experts // world_size


        # create the experts, model with a small DenseMLPWithLoRA, where ffh_size set to e
        self.experts = nn.ModuleList([
            DenseMLPWithLoRA(
                hidden_size=hidden_size,
                ffh_size=self.e,    # set to e: ffh_size // ne
                activation_type=activation_type,
                init_base_seed= init_base_seed + expert_idx,
                lora_rank=lora_rank,
                lora_alpha=lora_alpha,
                lora_dropout_rate=lora_dropout_rate,
                lora_dropout_seed=lora_dropout_seed + expert_idx,
                lora_init_base_seed=lora_init_base_seed + expert_idx,
                dtype=dtype,
                device=device
            )
            for i in range(self.nle)
            for expert_idx in [rank*self.nle + i]
        ])



        self.reset_parameters()

               
    def forward(self, input: torch.Tensor) -> torch.Tensor:
        """The forward pass of the Sparse MLP module with LoRA adapters
        
        Args:
            input(torch.Tensor): input tensor, with shape: [batch_size, seq_len, hidden_size]
            
        Returns:
            output(torch.Tensor): output tensor, with shape: [batch_size, seq_len, hidden_size]
        """

        # receive input X [b, s, h]
        # token t: [b, 1, h]

        # G: [h, ne]

        # calculated P_t
        x = input.to(dtype=self.G.dtype, device=self.G.device)  # [b, s, h]
        P_t = F.softmax(x @ self.G, dim=-1) # [b, s, ne], 

        # choose router
        topk_scores, topk_indices = torch.topk(P_t, k=self.moe_topk, dim=-1)
        # topk_indices: 

        # topk_scores: Q_t [b, s, k]
        # renormalization: Q_t / sum(Q_t), forall t
        routing_weights = topk_scores / topk_scores.sum(dim=-1,keepdim = True)

        # output has the same shape with input x
        output = torch.zeros_like(x)

        for i in range(self.nle):
            global_idx = self.rank * self.nle + i
            
            # get the expert who are one of topk_indices
            expert_mask = (topk_indices == global_idx)

            # .any: if any element in `input` is True
            if expert_mask.any():
                weight = (expert_mask * routing_weights).sum(dim=-1, keepdim= True) # [b, s, 1]

                expert_out = self.experts[i](x)
                output += expert_out * weight

        return output.to(dtype=input.dtype, device= input.device)




        # unrenormalization new Distribution Q
        
    def reset_parameters(self):
        """Initialize the weights of each local expert from its own distribution \
            and the gating layer from a normal distribution
        """

        # init the G
        g_G  = torch.Generator(device= self.G.device).manual_seed(self.init_base_seed)

        nn.init.normal_(
            self.G, 
            mean = self.init_mean, 
            std = self.init_std,
            generator=g_G
        )


        # reset_parameters has been called in __init__ of DenseMLPWith LoRA
        
```

*这里要注意一个问题*，`softmax` 要指定维度。否则可能不是对你要做的维度做
- **dim** ([_int_](https://docs.python.org/3/library/functions.html#int "(in Python v3.14)") _or_ [_tuple_](https://docs.python.org/3/library/stdtypes.html#tuple "(in Python v3.14)") _of_ _ints__,_ _optional_) – the dimension or dimensions to reduce. If `None`, all dimensions **are reduced.**

