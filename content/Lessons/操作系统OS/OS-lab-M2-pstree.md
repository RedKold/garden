## 数据结构设计
在这个实验中，我们要实现“打印进程树”的功能

先来看一下 pstree 的标准实现长什么样：

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260408142646.png)

这是输入了 `pstree` 之后打印的本机的进程树

可见这个结构体要维护进程树之间的父子关系。
