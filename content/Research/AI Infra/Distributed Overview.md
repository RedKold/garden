# Basic
[Pytorch Distributed Overview](https://docs.pytorch.org/tutorials/beginner/dist_overview.html)

主要学习 `torch.distributed`  package. 这个包实际包含了一个并行模块 (parallelism modules) 的集合，一个 communications layer, 还有 infrastructure for launching and debugging large training jobs

## Parallelism APIs
- Distributed Data-Parallel (DDP)
- Fully Shared Data-Parallel Training (FSDP2)
- Tensor Parallel (TP)
	- #TODO 
- Pipeline Parallel

## Sharding primitives（分片原语）
> `Densor` and `DeviceMesh` are primitives used to build *parallelism* in terms of sharded or replicated tensors on N-dimensional group

- `DTensor`: 描述了 Tensor 的分布方式：分片 and/or 复制。并且自动的根据操作来分片
- `DeviceMesh`：将加速器设备通信器抽象为多维数组的形式。
	- 负责管理底层的 ProcessGroup 实例，以支持多维并行 (multi-dimensional parallelism) 中的集合通信 (collective communications)

## Communications APIs
PyTorch distributed communication layer (C10D) offers both collective communication APIs (`all_reduce` and `all_gather`) and P2P

[Writing Distributed Application with PyTorch](https://docs.pytorch.org/tutorials/intermediate/dist_tuto.html)

这是 PyTorch 实现 Communication（多机通信）的最底层 (C10D)
### Collective Communication


|                                                                                                                                                     |                                                                                                                                                               |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [![Scatter](https://docs.pytorch.org/tutorials/_images/scatter.png)](https://docs.pytorch.org/tutorials/_images/scatter.png)<br><br>Scatter         | [![Gather](https://docs.pytorch.org/tutorials/_images/gather.png)](https://docs.pytorch.org/tutorials/_images/gather.png)<br><br>Gather                       |
| [![Reduce](https://docs.pytorch.org/tutorials/_images/reduce.png)](https://docs.pytorch.org/tutorials/_images/reduce.png)<br><br>Reduce             | [![All-Reduce](https://docs.pytorch.org/tutorials/_images/all_reduce.png)](https://docs.pytorch.org/tutorials/_images/all_reduce.png)<br><br>All-Reduce       |
| [![Broadcast](https://docs.pytorch.org/tutorials/_images/broadcast.png)](https://docs.pytorch.org/tutorials/_images/broadcast.png)<br><br>Broadcast | <br>[![All-Gather](https://docs.pytorch.org/tutorials/_images/all_gather.png)](https://docs.pytorch.org/tutorials/_images/all_gather.png) <br><br> All-Gather |

## Applying Parallelism To Scale Your Model
### DP (Data Parallelism)
> [!note] Data Parallelism
> Data Parallelism is a widely adopted single-program multiple-data training *paradigm*(范例) where the model is *replicated* on every process, every model replica computes local gradients for a different set of **input data samples**, **gradients** are averaged within the data-parallel communicator group before each optimizer step

**总结**：模型在各个 process 复制，对不同的 input data samples 跑出 gradients，再在进 optimizer 之前取平均。
- 这样对数据的分布有要求吧 (?)

### Model Parallelism
> [!Note] Model Parallelism
> Model Parallelism techniques (or Sharded Data Parallelism) are required when a model doesn't fit in GPU, and can be combined together to form multi-dimensional (N-D) parallelism techniques.

**总结**：如果一个模型不 fit GPU，但是可以组在一起来做 multi-dimensional 的并行。每个 process 不持有完整的模型，但持有模型的某维度（Tensor Parallel）或某阶段 (Pipeline Parallel)
- 模型并行是将一个模型切分到多个设备上协同计算，区别于数据并行（每个设备持有完整模型副本）。

### How to choose from
1. Use `DistributedDataParallel(DDP)`, if your model fits in a single GPU but you want to easily scale up training using multiple GPUs.
	1. see also on [Getting Started with Distributed Data Parallel](https://docs.pytorch.org/tutorials/intermediate/ddp_tutorial.html)
2. Use `FullyShardedDataParallel(FSDP2)` when your model cannot fit on one GPU
	1. See also: [Getting Started with Fully Sharded Data Parallel (FSDP2)](https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html)
3. Use `Tensor Parallel(TP)` and/or `Pipeline Parallel(PP)` if you reach scaling limitation with FSDP2
	1. Try [Tensor Parallelism Tutorial](https://pytorch.org/tutorials/intermediate/TP_tutorial.html)


# Tensor Parallelism
- [Tensor Parallel APIs](https://pytorch.org/docs/stable/distributed.tensor.parallel.html)
- [Getting Started with DeviceMesh](https://pytorch.org/tutorials/recipes/distributed_device_mesh.html)
- [Getting Started with Fully Sharded Data Parallel](https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html)

#TODO  Read [Megatron-LM](https://arxiv.org/abs/1909.08053) paper

