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
```



- With **random** or **constant** value

```python
shape = (2,3,)
rand_tensor = torch.rand(shape)
ones_tensor = torch.ones(shape)
zeros_tensor = torch.zeros(shape)
```

