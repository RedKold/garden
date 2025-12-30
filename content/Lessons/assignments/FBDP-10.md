
1. 简述RDD、DataFrame、DataSet的区别  
2. 简述Spark SQL的执行流程
3. 简述Spark ML的流水线含义
4. 简述Spark Streaming和Spark Structured Streaming的区别  
5. 简述GraphX的弹性分布式属性图的设计思想

### 简述 RDD、DataFrame、DataSet 的区别
- **备忘**：讲义中的对比图
- ![image.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/20251222151729.png)

- **RDD** (Resilient Distributed Dataset)
	- Spark 的基本抽象，提供了不可变的分布式的数据集合，可以并行操作
	- 特点
		-  分布式的内存抽象，允许在大型集群上执行基于内存的计算（In-Memory Computing），同时还保持了MapReduce等数据流模型的容错特性
		- 支持各种数据类型，但无法根据数据结构进行编程。**更适合非结构化数据**，*不适合大规模结构化数据*
		- 可以精细控制 rdd 的转换
		- 可设置checkpoint（设置检查点，用于容错）
	
- **DataFrame**
	- `DataFrame` 是一个列名组织的数据集 (Dataset)，等价于一个关系数据库中的表。
	- 特点：
		- 有优秀的的优化
		- 提供了丰富的接口
		- 支持广泛的多种数据源 (a wide array of sources)
			- 结构化数据，对非结构化数据支持不太好
			- *结构化数据规模处理好*
		- API 不够灵活
- **DataSet**
	- 是 DataFrameAPI 的一个扩展。
	- 特点
		- 提供了编译时类型检查
		- 支持各种数据类型
		- 结合了 RDDs 和 DataFrame 的优点：对结构化和非结构化数据处理都很好
		- 强类型 (Strong typing) ，支持 lambda 函数，SQL 优化执行 


## 简述Spark SQL的执行流程

**执行流程示意图**
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251222153952.png)
- Spark SQL 执行的具体步骤如下

#### 解析（Analysis）
Spark 通过 Antlr 解析器将SQL 语句转化为未解析的逻辑计划树（**Unresolved** Logical Plan），此时Spark 不知道字段和表的具体信息。
之后，其通过访问元数据库 *Catalog* ，确认表名、列名和数据类型。表转化为  *Resolved Logical Plan*

#### 逻辑计划优化 (Logical Optimization)
这是Catalyst 优化器的核心一步。它基于一套基于规则的优化策略(Rule-Based Optimization, RBO)改写逻辑计划, 生成 *Optimized Logical Plan*

#### 物理计划阶段 (Physical Planning)
这个阶段
- *Spark* mapping the logical operation to physical strategy that can be done on cluster (映射逻辑到物理计划操作)
然后, 通过一个Cost Model，来选定一个性能最优的物理计划。
结果是选定一个 *Selected Physical Plan*

#### 执行阶段
这是最后的落地阶段
- Spark **将最终生成的代码以 RDD**的形式在Spark Cluster 上运行


##  简述Spark ML的流水线含义
Spark ML 把执行过程整个抽象为了Pipeline
- **数据基础**：采用DataFrame
- 实现了以下的API
	- *Transformer*
		- 实现一个算法，把一个DataFrame 转化为另一个DataFrame
		- 可以用来实现*预处理*等工作
	- *Estimator*
		- fit a *DataFrame*, 算法生成另一个 *Transformer*
		- 即拟合一个数据集 (*DataFrame*) 并生成另一个**模型**（训练过程）
		- `fit()`
	- *Pipeline*
		- 连接指定的 *Transformer*和 *Estimator*，形成工作流
	- *Parameter*：全部的Transformers和Estimators共享一个指定Parameter的通用API。

一个流水线，被指定为由一系列 *Transformer*和 *Estimator* （Stage）构成的工作流。这些Stage 按顺序对 *DataFrame* 进行操作
 - ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222163836.png)

Spark Pipeline 完成了对机器学习各阶段的高阶抽象，使得代码更简单。



## 简述Spark Streaming和Spark Structured Streaming的区别  
- Spark Streaming
- **数据模型上**
	- Spark Streaming 使用**微批**的形式处理，实现了基于 RDDs 的 DStream API，每个时间间隔上的数据为一个 RDD，源源不断对 RDD 进行处理来实现
	- Spark Structured Streaming 使用**无界表**的概念计算，流数据相当于不断的新加行
- **API**
	- Spark Streaming DStream interface: RDD
	- Structured Streaming: 使用 DataFrame, Dataset 的编程接口，还可以使用 Spark SQL 的方法
- **Process Time vs. Event Time**
	- Process Time: 流处理引擎接收到数据的时间
	- Event Time：时间真正发生的时间
	- Spark Streaming 由于微批的概念，将一段时间内接收到的数据放入一个批内，划分批的时间是 Process Time 而不是 Event Time，没有提供对 Event Time 的支持。
	- Structured Streaming 提供了基于事件时间处理数据的功能，如果数据包含事件的时间戳，就可以基于事件时间进行处理。开发者可以很方便处理**乱序**到达的数据等复杂情况
	- Structured Streaming 的 continuous mode 提供了实时处理。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222171721.png)
- **可靠性处理**
	- 两者在可靠性保证方面都是使用了 *checkpoint* 机制。checkpoint 通过设置检查点，将数据保存到文件系统，在出现故障的时候进行数据恢复。
	- 在 Spark Streaming 中，如果我们需要修改流程序的代码，在修改代码重新提交任务时，是不能从 checkpoint 中恢复数据的（程序就跑不起来），是因为 Spark 不认识修改后的程序了。
	- 在 Structured Streaming 中，对于指定的代码修改操作，**是不影响修改后从 checkpoint 中恢复数据的**。
- **总结**:
	- Structured Streaming 有更简洁的 API、更完善的流功能、更适用于流处理。而 Spark Streaming 更适用于偏批处理的场景

## 简述GraphX的弹性分布式属性图的设计思想

GraphX 的弹性分布式属性图的设计思想可以说是：**亦图亦表**，**图就是表**
- GraphX 在 RDD 的基础上抽象出了 Resilient Distributed Property Graph, 在同一份物理存储基础上实现了两个视图：Table & Graph
- 对 Graph 视图的所有操作，最终转化为对其关联的 Table 视图的 RDD 操作完成。这样对一个图的计算，最终在逻辑上，等价于一系列 RDD 的转换过程。因此，Graph 最终具备了 RDD 的三个关键特性：
	- Immutable 不变性
	- Distributed 分布性
	- Fault-Tolerant 容错性
- 逻辑上，所有图的转换和操作都产生了一个*新图*
- 二者视图共用物理数据：
	- `RDD[VertexPartition]`, `RDD[EdgePartitoin]` 组成
	- 是一个内部存储的带索引结构的分片数据块

> this ends the assignment 10;