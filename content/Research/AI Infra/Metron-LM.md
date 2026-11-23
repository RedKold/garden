---
tags:
  - LLM
---
## Read papers

- *Abstract*
	- 在本工作中，我们采用层内模型并行实现了一种简单高效的模型并行方法。我们利用基于 Transformer 的语言模型的固有结构，构建了一个简单的模型并行实现，可在 PyTorch 中高效训练，且无需自定义 C++ 代码或编译器。该方法与 GPipe 等方法所倡导的基于流水线的模型并行正交


**Contribution** as:
- Implement a simple and efficient *model parallel approach* by making only a few targeted **modifications** to an existing PyTorch **transformer** implementation.
- We show that careful attention to the placement of *layer normalization in BERT-like* models is critical to achieving increased accuracies as the model grows.
- 我们表明，扩大模型规模能够提升 GPT-2（研究至 83 亿参数）和 BERT（研究至 39 亿参数）模型的准确率
- art results
	- test sets: 
		- perplexity on WikiText103
		- accuracy on LAMBADA
		- accuracy on RACE

[open source code](https://github.com/NVIDIA/Megatron-LM)


### Neural Language Model Pre-training
- Pre-trained language model have become an indispensable part of NLP researchers' tookits
- Pre-training cost much.


### Transformer Language Models and Multi-Head Attention
- Original transformer formulation:
	- a machine translation architecture, transforms an input sequence into another output sequence, using *Encoder* and *Decoder*
	- GPT-2: *Decoder-Only*
	- BERT: *Encoder-Only*


- The model used in this *paper*:
	- tips:
		- GPT-2 and BERT, both **GeLU** nonlinearities and layer normalization to the *input* of multi-head attention and feed forward layers, 
			- whereas the original transformer uses **ReLU**  nonlinearities and applies layer normalization to *outputs*


### Data and Model Parallelism in Deep Learning
Two central *paradigms* for scaling out deep neural network training to numerous hardware accelerators:
- data parallelism
	- a training mini-batch is split across multiple workers
- model parallelism
	- memory usage and computation of a model is distributed across multiple *workers*
However, large batch training introduces complications into the optimization process that can result in reduced accuracy or longer time to convergence.
- LM like BERT and GPT-2, are still increasing size and complexity. **MLE**

我们是怎么解决的？
- *Utilize* model parallelism to split the model across multiple accelerators.
- *alleviates* the memory pressure
- increases the amount of parallelism independently of the micro-batch size.

---

讨论一下 Model parallelism
- Two further paradigms
	- layer-wise *pipeline* parallelism
	- more general distributed tensor computation

- pipeline model parallelism
	- groups of operations are performed on one device
	- outputs are passed to the next device in the pipieline
		- a different group of operations are performed
	- Pipeline server?

**分布式张量计算**(distributed tensor computation)
- FlexFlow
- Mesh-TensorFlow
- **我们**：利用计算 Transformer Attention Head 时的并行性，来并行化我们的 Transformer 模型


**Our Approach**
- no require *any new compier* or *code re-writing*
- 对 PyTorch transformer implementations 做了一些修改。

- 看起来是很轻量化的

## Model Parallel Transformers
- Create a simple model parallel implementation by adding a few synchronization primitives.



### Detailing MLP block
MLP: multi-layer perceptron (MLP)


第一部分：
- GEMM followed by a **GeLU** non-linearity
$$
Y=\mathrm{GeLU}(XA)
$$
- 如何并行化 GEMM 呢？



- ONE OPTION
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20261025205112.png)
		- 这个例子可以说明，对 $X$ 列并行，对 $A$ 行并行，会产生部分和。所以需要一个 *synchronization point*
	- split the weight matrix $A$ along its *rows* and input $X$ along its *columns* as:
$$
X=[X_{1},X_{2}],A=[A_{1};A_{2}]
$$
	- This partitioning: $Y=\mathrm{GeLU}(X_{1}A_{1}+X_{2}A_{2})$,  Since **GeLU** is a nonlinear function, so $\mathrm{GeLU}(X_{1}A_{1}+X_{2}A_{2})\neq \mathrm{GeLU}(X_{1}A_{1})+\mathrm{GeLU}(X_{2}A_{2})$, and this approach require a *synchronization point* before the **GeLU** function

- ANOTHER *OPTION*
	- spilt $A$ along its *columns*, $A=[A_{1},A_{2}]$
	- This partitioning allows the GeLU nonlinearity to be independently applied to the output of each partitioned GEMM

$$
[Y_{1},Y_{2}]=[\mathrm{GeLU}(XA_{1}),\mathrm{GeLU}(XA_{2})]
$$
	- **优点**
		- removes a *synchronization point.*
		- Hence, we partition the first **GEMM** in this column parallel fashion and split the second **GEMM** along its rows so it takes the output of the **GeLU** layer directly without requiring any communication.

This approach splits both GEMMs in the MLP block across **GPUs** and requires only a single all-**reduce** operation in the forward pass ($g$ operator) and a single all-reduce in the backward pass ($f$ operator)


> [!question] What's indeed **GEMM**?
> **GEMM = GEneral Matrix Multiply（通用矩阵乘法）**，是 BLAS 标准库里的一个基础函数名，做的事就是：
> 
> 代码块
> 
> ```
> C = α·A·B + β·C
> 
> A: [M, K]      B: [K, N]      C: [M, N]
> ```
> 
> `α`、`β` 是两个标量系数（通常取 1 和 0，就退化成普通的 `C = A·B`）。名字里的 “General” 是相对于 BLAS 里更简单的 GEMV（矩阵×向量）、GER（向量外积）而言的。
> 
> ## 深度学习里为什么总提它
> 
> 因为**一个全连接层（去掉 bias）就是一个 GEMM**：
> 
> 代码块
> 
> ```
> Y = X·W + b
>     └──┬──┘
>       GEMM（bias 是一个 broadcast 加法，通常单独算或 fuse 掉）
> ```
> 
> 而 Transformer 基本上就是一堆 GEMM 串起来：





- 我们实现出来的大概就是这样:
 ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20261024183307.png)

上图第一个是 GeLU 操作，第二个是 Dropout 操作，具体逻辑如下：

1. MLP 的整个输入 X 通过 f 放置到每一块 GPU 之上。
2. 对于第一个全连接层：
    1. 使用列分割，把权重矩阵切分到两块 GPU 之上，得到 A1, A2。
    2. 在每一块 GPU 之上进行矩阵乘法得到第一个全连接层的输出 Y1 和 Y2。
3. 对于第二个全连接层：
    1. 使用行切分，把权重矩阵切分到两个 GPU 之上，得到 B1, B2。
    2. 前面输出 Y1 和 Y2 正好满足需求，直接可以和 B 的相关部分（B1, B2）做相关计算，不需要通信或者其他操作，就得到了 Z1, Z2。分别位于两个 GPU 之上。
4. Z1, Z2 通过 g 做 all-reduce（这是一个同步点），再通过 dropout 得到了最终的输出 Z。

然后在 GPU 之上，第二个 GEMM 的输出在传递到 dropout 层之前进行规约。这种方法将 MLP 块中的两个 GEMM 跨 GPU 进行拆分，并且只需要在前向过程中进行一次 all-reduce 操作（g 操作符）和在后向过程中进行一次 all-reduce 操作（f 操作符）。

```python
class f(torch.autograd.Function):
	def forward(ctx, x):
		return x
	def backward(ctx, gradient):
		all_reduce(gradient)
		return gradient
```

这是一个 $f$ operator 的实现。$g$ 是类似的：只需要在 forward 使用 all-reduce，在 backward 使用恒等。


| 位置                        | 切法             | 每张卡手里是什么                            | 通信              |
| ------------------------- | -------------- | ----------------------------------- | --------------- |
| MLP GEMM1 `X·A`，`A[h,4h]` | **列切**（切 n=4h） | `Y_i` 是 `Y` 沿 4h 维的**切片**           | 无               |
| ↳ GeLU                    | 本地逐元素          | 仍是切片                                | 无               |
| MLP GEMM2 `Y·B`，`B[4h,h]` | **行切**（切 k=4h） | `Z_i` 是 `Z` 的**部分和**                | all-reduce（`g`） |
| Attention QKV 投影          | **列切**（按头切）    | 每个头完整的 Q/K/V **切片**                 | 无               |
| ↳ softmax                 | 本地（头内）         | 仍是切片                                | 无               |
| Attention 输出投影            | **行切**         | **部分和**                             | all-reduce（`g`） |
| 输入 embedding `E[h,v]`     | **列切**（切词表 v）  | 该卡负责的 token 有完整向量，其余位置为 0 → **部分和** | all-reduce（`g`） |
| 输出 logits + fused CE      | **列切**（切词表 v）  | 各卡一段词表上的**部分 loss**                 | all-reduce 标量   |

### Self-Attention block

![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20261025194321.png)


We exploit inherent parallelism in the *multi-head* attention operation, partitioning the **GEMMs** associated with key ($K$), query $Q$, and value ($V$) in a **column** parallel fashion such that the matrix multiply corresponding to each attention head is done *locally* on one GPU.
- 仍然使用 column parallel
- 不需要 immediate communication 来完成 self-attention

在之后的 **GEMM** (from the output layer, after self attention) ，是 row-parallel 的
- 一样的道理，可以 takes the output of the parallel attention layer directly without requiring communication between the **GPUs**


Transformer 语言模型的输出嵌入维度为 hidden-size（H）乘以 vocabulary size（v）。由于现代语言模型的词表大小处于数万 token 的量级（例如 GPT-2 使用的词表大小为 50,257），因此并行化输出嵌入 GEMM 是有益的

```
H(hidden-size) * V(vocabulary-size)
```

然而，在 Transformer 语言模型中，**输出嵌入层与输入嵌入共享权重**(output embedding layer shares weights with the input embedding, requiring modifications to both)，因此二者都需要修改。我们沿词表维度按列并行化输入嵌入权重矩阵 $E_{H×v}$，即 $E = [E1, E2]$。(column-wise)

由于每个分区现在只包含嵌入表的一部分，因此在输入嵌入之后需要进行一次 all-reduce（g 算子）。

（图 4：Transformer 层中的通信操作。在单个模型并行 Transformer 层的前向和反向传播中，总共有 4 个通信操作。）

对于输出嵌入，一种方法是执行并行 GEMM $[Y1, Y2] = [XE1, XE2]$ 以获得 logits，加入 all-gather Y = all-gather ($[Y1, Y2]$)，并将结果送入**交叉熵损失函数**。然而，在这种情况下，all-gather 需要通信 b × s × v 个元素（b 为 batch-size，s 为序列长度），由于词表大小很大，这一通信量非常巨大。为减少通信量，我们将并行 GEMM 的输出 $[Y1, Y2]$ 与**交叉熵损失融合**，从而将维度降至 b × s。通信标量损失而非 logits 能大幅减少通信(Communicating scalar losses instead of logits)，并提升我们模型并行方法的效率


---
- Our Model Parallel approach:
	- reducing communication and keeping the GPUs compute bound
	- no *compute* part of the dropout, layer normalization, or residual connections, and *broadcast* the results to other GPUs
	- Duplicate the computation across **GPUs**
		- 问题：这样会重复多少？
		- **maintain** duplicate copies of layer normalization parameters on each GPU, and take the output of the model parallel region and run dropout and residual connection on these tensors before feeding them as input to the next model parallel regions


## Setup
预训练语言理解模型是自然语言处理与语言理解任务的核心。语言建模有多种形式。本文聚焦于 GPT-2（Radford 等，2019），一种从左到右的生成式 Transformer 语言模型，以及 BERT（Devlin 等，2018），一种基于语言模型掩码的双向 Transformer 模型。我们在下一节说明这些模型的配置，更多细节请参阅原始论文。


### Training Dataset

- To collect a large diverse training set with longterm dependencies, we aggregate several of the largest language modeling datasets.
	- 聚合。移除换行符 (CC-Stories, unnecessary newlines).
	- 合并数据集，过滤少于 128token 的文档，使用 LSH 对 Jaccard 相似度大于 0.7 的内容去重

- Resulting aggregate corpus:
	- contains **174GB** of deduplicated text


### Training Optimization and Hyper-parameters


## Conclusion

在这项工作中，我们仅对现有 PyTorch Transformer 实现做了少量修改，就实现了模型并行，从而成功突破了传统每个模型仅用单块 GPU 训练所带来的限制。我们利用 8 路模型并行，在 512 块 NVIDIA V100 GPU 上高效训练了参数量高达 83 亿的基于 Transformer 的模型，并在整个应用过程中持续达到最高 15.1 PetaFLOPs。

我们研究了模型规模对下游任务准确率的影响，在下游任务上取得了远优于已有方法的结果，并在 WikiText103、LAMBADA 和 RACE 数据集上建立了新的 SOTA。最后，我们开源了代码，以支持未来利用模型并行 Transformer 的工作。


**未来工作**：
- 继续扩大与训练规模
- 预训练不同的模型
- 在更难的上下游任务评估性能
- 知识蒸馏 (knowledge distillation), train small student models from these large pretrained teacher models.


