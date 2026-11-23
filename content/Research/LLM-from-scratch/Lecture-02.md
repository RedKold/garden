## Preview

use `forward` to describe model calculation:

```python
class Model(nn.Module):
	def __init__(self):
		super().__init__()
		self.layer = ...
		
	def forward(self, x):
		h = self.layer(x)
		y = ...
		return y			# output tensor

model = Model()
y = model(x)				# trigger forward
```


> [!note] 从数据中学习表示（Learn Representation）
> - 设定任务目标
> - 收集样本与标签
> - 将现实对象表示为特征
> - 学习从输入表示到目标表示的变换
> - $Y=f(X;\theta)$

Where $\theta$ is the parameters that need *model* to learn 

前向过程就是一个对输入张量进行计算的程序

## Linear Layer: Transformation
$$
Y=XW+b
$$
- $X$: input features
- $W,b$: model parameters
	- $b$ 是一个 bias，偏置
- $Y$: *new* feature representation
- $D\to E$: the **transformation** happened in *last* feature dimension

	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260904104006.png)


## 用形状描述前向计算
$$
X\in \mathbb{R}^{B\times H\times D},W\in \mathbb{R}^{D\times E},Y=XW\in \mathbb{R}^{B\times H\times E}
$$


| Dim     | Meaning         |
| ------- | --------------- |
| $B$     | batch: 同时处理的样本数 |
| $H$     | 额外结构维：位置、对象或序列  |
| $D$     | 输入特征维度          |
| $E$<br> | 输出特征维度          |
|         |                 |

- 每个位置不同，但使用同一个 $W$
	- weight 矩阵。同样的变换。

### 偏置 $b$ 引出的另一种“复用”
$$
Y=XW+b
$$
```python
xw = x @ w	# [B, H, E]
b  = ...	# [E]
y  = xw + b	# [B, H, E]
```

广播机制，PyTorch 让同一个 $b[E]$ 沿 $B,H$ 维度参与每个位置的加法。这就是*broadcasting*

## 矩阵乘：从线性变换考虑
$$
y_{bhe}=\sum_{d=1}^{D}x_{bhd}w_{de}
$$

- 每个输出元素是一个 dot-product
- 总计算量约为 $O(BHDE)$
- 最主要的计算来源

## Tenor 同时携带数据和计算属性
```python
x = torch.randn(
	2,3,4,
	dtype=torch.float32,
	device="cpu",
)
```

- `shape`：数据如何组织
- `dtype`：每个元素如何表示
- `device`：计算在哪里执行
- `requires_grad`：是否需要追踪梯度

后续所有模型组件，输入、输出和参数最终都是 Tensor。


## einsum
$$
Y_{bhe}=\sum_{d}X_{bhd}W_{de}
$$

```python
B, H, D, E = 2,3,4,5
x = torch.rand(B,H,D)
w = torch.randn(D, E)

y = torch.einsum("bhd, de->bhe", x, w)
assert y.shape == (B, H, E)
```

`b` 、`h` 、`e` 被保留，`d` 只在输入中出现，因此沿 `d` **求和**


## Parameter is also Tensor
**But managed by model**

```python
w = nn.Parameter(torch.empty(D,E))
b = nn.Parameter(torch.zeros(E))
```

普通 Tensor 变成 `Parameter` 后：

- 会被模块自动注册
- 会出现在 `model.parameters()` 中
- 会随 `model.to(device)` 移动
- 会进入 `state_dict()`，从而保存和加载


## A Module Example
```python
class FeatureTransform(nn.Module):
	def __init__(self, in_dim, out_dim):
		super().__init__()
		self.weight = nn.Parameter(
			torch.empty(in_dim, out_dim)
		)
		self.bias = nn.Parameter(torch.zeros(out_dim))
		nn.init_xavier_uniform(self.weight)	
	
	def forward(self, x):
		return x @ self.weight + self.bias
```

**Module**定义了“状态 + 前向规则”

> 一个 PyTorch 模型本质上由多个 `nn.Module` 递归组合而成

