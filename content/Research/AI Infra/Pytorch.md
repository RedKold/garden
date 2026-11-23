# Tensors

> [!Note] What's tensors
> Tensors are a specialized data structure that are very similar to arrays and matrices.

Tensors can be run on GPUs or other specialized hardware to accelerate computing.

## Tensor initialization

- Directly from data:
```python
data = [[1,2], [3,4]]
x_data = torch.tensor(data)
```

- from a NumPy array

```python
np_array = np.array(data)
x_np = torch.from_numpy(np_array)
```

- Also, tensor can be initialized by other tensors
- retains the properties (shape, datatype) of the argument tensor

```python
x_ones = torch.ones_like(x_data) # retains the properties of x_data
print(f"Ones Tensor: \n {x_ones} \n")

x_rand = torch.rand_like(x_data, dtype=torch.float) # overrides the datatype of x_data
print(f"Random Tensor: \n {x_rand} \n")

x_zero = torch.zeros_like(x_data)
```



- With **random** or **constant** value

```python
shape = (2,3,)
rand_tensor = torch.rand(shape)
ones_tensor = torch.ones(shape)
zeros_tensor = torch.zeros(shape)
```

## Tensor Attributes

```python
tensor.shape
tensor.dtype
tensor.device
```

```output
>>> print(f"Dtype of tensor is : {tensor.dtype}")
Dtype of tensor is : torch.float32
```

## Tensor Operations

each of operation can be run on the GPU

- **Standard** numpy-like indexing and slicing
```python
tensor = torch.ones(4,4)
tensor[:,1] = 0
print(tensor)
```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260718153140.png)

- **Joining tensors**

```python
t1 = torch.cat([tensor, tensor, tensor], dim=1)
print(t1)
```

you can use `torch.cat` to concatenate a sequence of tensors along a given dimension.


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260718153531.png)


- **Multiplying tensors**
元素乘法
element-wise product

```python
# This computes the element-like product
print(f"tensor.mul(tensor) \n {tensor.mul(tensor)} \n") 
# Alternative syntax:
print(f"tensor * tenosr \n {tensor * tensor}")
```


this computes the matrix multiplication between two tensors

```python
tensor.matmul(tenosor.T)
# or
tensor @ tenosr.T
```


## More operations

[torch — PyTorch 2.13 documentation](https://docs.pytorch.org/docs/2.13/torch.html)


### `torch.bmm`
[torch.bmm — PyTorch 2.13 documentation](https://docs.pytorch.org/docs/2.13/generated/torch.bmm.html)

`torch.bmm` (_input_, _mat2_, _*_, _out=None_) → [Tensor](https://docs.pytorch.org/docs/2.13/tensors.html#torch.Tensor "torch. Tensor")


If `input` is a (b×n×m)(b×n×m) tensor, `mat2` is a (b×m×p)(b×m×p) tensor, `out` will be a (b×n×p)(b×n×p) tensor.

Performs a batch matrix-matrix product of matrices stored in `input` and `mat2`

### `torch.permute`

```python
torch.permute(input, dims) -> Tensor
```

Parameters:
- **input** ([_Tensor_](https://docs.pytorch.org/docs/2.13/tensors.html#torch.Tensor "torch. Tensor")) – the input tensor.
- **dims** ([_torch. Size_](https://docs.pytorch.org/docs/2.13/size.html#torch.Size "torch. Size")_,_ [_tuple_](https://docs.python.org/3/library/stdtypes.html#tuple " (in Python v3.14)") _of_ [_int_](https://docs.python.org/3/library/functions.html#int " (in Python v3.14)") _or_ [_list_](https://docs.python.org/3/library/stdtypes.html#list " (in Python v3.14)") _of_ [_int_](https://docs.python.org/3/library/functions.html#int " (in Python v3.14)")) – the desired ordering of dimensions.

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260718162519.png)

交换维度的作用。


### `torch.einsum`

[torch.einsum — PyTorch 2.13 documentation](https://docs.pytorch.org/docs/2.13/generated/torch.einsum.html)
需要熟悉爱因斯坦求和符号


### `torch.Tensor.scatter`

```python
Tensor.scatter_(dim, index, src, *, reduce=None) -> Tensor
```

Write all values from the tensor `src` into `self` at the indices specified in the `index` tensor. For each value in `src`, its output index is specified by its index in `src` for `dimension != dim` and by the corresponding value in `index` for `dimension = dim`

这个函数可以把在 `dim` 维度，将 `self` 在该维度的分量在 `index` 处的值替换为 `src` 的值

- `dim=0` 时： `self[index[i][j][k]][j][k] = src[i][j][k]`
- `dim=1` 时： `self[i][index[i][j][k]][k] = src[i][j][k]`
- `dim=2` 时： `self[i][j][index[i][j][k]] = src[i][j][k]`


# `nn`

## `Parameter`

```python
class torch.nn.parameter.Parameter(data=None, requires_grad=True)
```

A kind of Tensor that is to be considered a module parameter.

*Parameters* are Tensor subclasses, that have a very special property when used with s.
- they're assigned as Module attributes they are automatically added to the list of its parameters, and will appear e.g. in iterator.


## `init`


