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
	- 在 Spark Streaming 中，如果我们需要修改流程序的代码，在修改代码重新提交任务时，是不能从checkpoint中恢复数据的（程序就跑不起来），是因为Spark不认识修改后的程序了。
	- 在 Structured Streaming中，对于指定的代码修改操作，**是不影响修改后从 checkpoint中恢复数据的**。
- **总结**:
	- Structured Streaming有更简洁的API、更完善的流功能、更适用于流处理。而Spark Streaming更适用于偏批处理的场景



