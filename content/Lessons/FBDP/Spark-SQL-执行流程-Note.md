Spark SQL 的执行过程是一个将 SQL 语句或 Dataset/DataFrame 操作转化为底层 RDD 算子的过程。其核心引擎是 **Catalyst 优化器**，它负责从逻辑计划到物理计划的转换与优化。

- ![image.png|](https://kold.oss-cn-shanghai.aliyuncs.com/20251218214138.png)
以下是 Spark SQL 执行的具体步骤：

---

### 1. 解析 (Analysis)

当你提交一条 SQL 语句时，Spark 首先通过 **Antlr 解析器** 将其转化为一棵 **未解析的逻辑计划树 (Unresolved Logical Plan)**。此时，Spark 只知道 SQL 的语法结构，并不清楚字段是否存在、表在哪里。

- **操作：** 访问 **Catalog**（元数据库）。
    
- **结果：** 确认表名、列名和数据类型。将“未解析”状态变为 **解析后的逻辑计划 (Resolved Logical Plan)**。
    

### 2. 逻辑计划优化 (Logical Optimization)

这是 **Catalyst** 引擎最核心的一步。它基于一套**基于规则的优化策略 (Rule-Based Optimization, RBO)** 对逻辑计划进行改写。

- **常见优化手段：**
    
    - **谓词下推 (Predicate Pushdown)：** 尽早过滤数据，减少读取量。
        
    - **列裁剪 (Column Pruning)：** 只读取需要的列，减少内存占用。
        
    - **常量累加 (Constant Folding)：** 提前计算 `1 + 1` 这种表达式。
        
- **结果：** 生成 **优化后的逻辑计划 (Optimized Logical Plan)**。
    

### 3. 物理计划阶段 (Physical Planning)

在这个阶段，Spark 将逻辑上的操作映射为可以在**集群上执行的物理策略。**

- **策略生成：** 一个逻辑计划可能会对应多个物理计划（例如 Join 操作可以采用 Broadcast Hash Join、Sort Merge Join 等）。
    
- **代价模型 (CBO)：** Spark 会利用 **基于代价的优化 (Cost-Based Optimization)**，通过统计信息（如表大小、选择度）来选择性能最优的一个物理计划。
	- 这里对应着上图的 Cost Model
- **结果：** 选定 **Selected Physical Plan**。

### 4. 代码生成与执行 (Code Generation & Execution)

这是最后的落地阶段。

- **全阶段代码生成 (Whole-Stage Code Generation)：** Spark 不会直接解释执行物理计划中的操作，而是将这些操作转换成紧凑的 Java 字节码。这种技术类似于把多个算子合在一起生成一段高效的 CPU 执行代码，消除了大量虚函数调用。
    
- **执行：** 最终生成的代码以 **RDD** 的形式在 Spark 集群上运行。
---

### 总结流程图

| **阶段**   | **输入**          | **处理核心**       | **输出**                 |
| -------- | --------------- | -------------- | ---------------------- |
| **解析**   | SQL / DataFrame | Catalog 校验     | Resolved Logical Plan  |
| **逻辑优化** | Resolved Plan   | RBO (谓词下推等)    | Optimized Logical Plan |
| **物理计划** | Optimized Plan  | CBO (代价评估)     | Selected Physical Plan |
| **执行**   | Physical Plan   | 代码生成 (Codegen) | **RDD 执行结果**           |