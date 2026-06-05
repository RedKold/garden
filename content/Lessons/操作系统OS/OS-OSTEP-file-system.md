> [!Question] 关键问题
> 如何构建一个简单的文件系统？磁盘上需要什么结构？它们需要记录什么？它们如何访问？


# 思考方式
主要有两个要思考的：
- Data Structure
	- 文件系统在磁盘上使用哪些类型的数据来组织其数据和元数据？
- Access Method
	- 如何将进程发出的调用，如 `open(), read(), write()` 等映射到它的结构上？
		- 在执行特定系统调用期间读取*哪些*结构？
		- 改写哪些结构 ？
		- 所有这些步骤的执行效率如何？


# 整体组织
**VSFS**

- SUPER BLOCK
- data bitmap
- inode bitmap
	- 我们需要某种 allocation structure 来记录某一位数据或者 inode 是否被分配
- 数据区域
## 文件组织：inode

inode 中实际上是所有关于文件的信息
- **最重要的决定**：如何引用数据块的位置
	- 简单的方法：存一个或多个直接指针。
	- **局限性**：对大文件支持不好。没有那么多直接指针。（比如：大于 `4KB * 直接指针数`）
- trade-off：多级索引 (multi level index)
	- double indirect pointer：指向一个包含**间接块指针**的块
		- **每个间接块**都包含指向数据块的指针。
		- `4KB = 1024*指针(32bit)`，故可以使用额外的 `1024*1024` 个 `4KB` 块来增长文件。支持 4GB 大小的文件。
	- want more? **triple indirect pointer**

因为大多数文件可能很小，所以我们 trade-off 的结果是提供 12 个直接指针，这样<48KB 的文件我们可以直接在 inode 中一步获取


## 目录组织 

VSFS：目录的组织很简单。
一个目录基本上只包含一个二元组（条目名称，inode 号）的列表。
> 感觉上和这个差不多。
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260605162107.png)



[[OS-OSTEP-locality-and-fast-file-system]]