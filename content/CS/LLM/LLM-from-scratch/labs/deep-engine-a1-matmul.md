## Task 1: MalMul with multi-head variant


done

## Task 2: MalMul with importance

在 multi-head 矩阵乘法的基础上，引入一个表示“重要性”的概率张量 $P$，形状 $[b,s]$
- 表示 $A_{1}$ 中对应位置元素的重要程度。
- 我们目标：对每个序列中的“重要元素”执行矩阵乘法运算
	- 有 `total_important_seq_len` 个，简记 `t`
	- 计算结果收集到输出张量 `O3` 中，形状为 `[t, nh, e]`

- 重要元素的范围：
	- `top_p`：取值为 `[0., 1.]`。概率大于等于 `top_p` -> important element.
	- `top_k`: range `[1, ..., seq_len]`, 对于 batch 中的每个 seq，只把概率最高的 `top_k` 个元素视为重要元素
- **必须同时满足**


```python
def matmul_with_importance(
    input: torch.Tensor,
    weight: torch.Tensor,
    probs: torch.Tensor,
    grad_output: Optional[torch.Tensor] = None,
    num_heads: int = 1,
    top_p: float = 1.0,
    top_k: Optional[int] = None,
) -> Tuple[torch.Tensor, Optional[torch.Tensor], Optional[torch.Tensor]]:
    """matmul input and weight and return output (with optional grad_input, grad_weight whenever grad_output is given)
    where only the important elements of the input tensor can be computed and gathered to the output tensor
    decided by the importance probability tensor, tuned by top_p and top_k

    Args:
        input (torch.Tensor): input tensor in the range of [-1, 1], with shape: [batch_size, seq_len, hidden_size]
        weight (torch.Tensor): weight tensor in the range of [-1, 1], with shape: [hidden_size, embed_size]
        probs (torch.Tensor): probability tensor in the range of [0, 1], with shape: [batch_size, seq_len]
        grad_output (Optional[torch.Tensor], optional): gradient for the output tensor, with shape: [t, hidden_size]. Defaults to None. Used in Task3.
        num_heads (int): number of heads to split hidden_size
        top_p (float, [0., 1.]): only the elements with the probability equal or higher than top_p are important ones
        top_k (int, [1, ..., seq_len], optional): only the elements with the top_k highest probability are important ones

    Returns:
        output (torch.Tensor): output tensor, with shape: [t, num_heads, embed_size]
        grad_input (torch.Tensor, optional): gradient for the input tensor if grad_output is given, otherwise None.
            Return None in Task2.
        grad_weight (torch.Tensor, optional): gradient for the weight tensor if grad_output is given, otherwise None
            Return None in Task2.
    """

    b, s, h = input.shape
    if top_k is None:
        top_k = s

    # Step 1: Build the importance mask [b, s]
    mask_p = probs >= top_p
    _, topk_idx = torch.topk(probs, k=top_k, dim=1)
    
    mask_k = torch.zeros_like(probs, dtype=torch.bool)
    mask_k.scatter_(1, topk_idx, True)
    important_mask = mask_p & mask_k

    # Step 2: Extract important elements -> A3 [t, h]
    
    # split A1_flat into h sample
    A1_flat = input.reshape(-1, h)
    mask_flat = important_mask.reshape(-1)
    A3 = A1_flat[mask_flat]
    
    # After our boolean indexing, then the first dim is `t`: the number of important seq
    t = A3.shape[0]

    # Step 3: Multi-head matmul (same as Task 1)
    hd = h // num_heads
    A3_reshaped = A3.reshape(t, num_heads, hd)
    W2 = weight.reshape(num_heads, hd, -1)
    O3 = torch.einsum("tnh, nhe->tne", A3_reshaped, W2)

    return O3, None, None
```


## Task 3: MalMul with grad
如果提供了输出张量的可选梯度（记为 `dO3`，其形状与 `O3` 相同），需要计算输入向量的梯度
`dA1` 和权重向量的梯度 `dW1`

这里我们采用**分母布局**，即梯度张亮的 shape 总是和变量（输入）保持一致


反向传播的代码：

```python
def backward(self, grad_output):  
       # grad_output: [b, h, e]  
       # dL/dW = X^T @ grad_output  
       b, h, d = self.input.shape  
       # [b,h,d] -> [b*h, d]  
       x_flat = self.input.reshape(-1, d)  
       # [b,h,e] -> [b*h, e]  
       grad_out_flat = grad_output.reshape(-1, self.W.shape[1])  
       # [d, b*h] @ [b*h, e] -> [d, e]  
       self.W_grad = x_flat.T @ grad_out_flat  
       # dL/dx = grad_output @ W^T  
       # [b,h,e] @ [e,d] -> [b,h,d]  
       grad_input = grad_output @ self.W.T  
       return grad_input
```

