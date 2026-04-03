# Chapter 1 Dialog
**三个部分**：

- **虚拟化 virtualization**
- **并发 concurrency**
- **持久性 persistence**


# Chapter 2 操作系统介绍

> 有一类软件专门负责 **让程序运行变得容易**（甚至允许你同时运行多个程序），允许程序共享内存，让程序能够与设备交互，以及其他类似的有趣的工作。这些软件称为操作系统 (Operating System, OS)

## 虚拟化 virtualization
**Virtualization**
- 做到以上的主要技术
-  操作系统将物理 (physical)资源（如处理器，内存或磁盘）转换为更通用，更强大且更易于使用的虚拟形式。
- 操作系统->虚拟机

**操作系统**在干什么？可以从几个别名窥见一二
- virtual machine
	- 操作系统做虚拟化
- standard library
	- 提供 system call，让应用程序调用
- resource manager
	- 操作系统可以管理资源，共享内存、共享磁盘、共享 CPU 的处理

## 并发 concurrency
```c
**
 * 线程函数1：没有使用互斥锁，会产生竞争条件
 */
void* thread_func_no_mutex(void* arg) {
    int thread_id = *(int*)arg;
    int iterations = 100000;

    printf("线程 %d 开始 (无互斥锁)\n", thread_id);

    for (int i = 0; i < iterations; i++) {
        // 这是一个非原子操作，包含读取-修改-写入三个步骤
        shared_counter++;
    }

    printf("线程 %d 结束\n", thread_id);
    return NULL;
}

/**
 * 线程函数2：使用互斥锁保护共享变量
 */
void* thread_func_with_mutex(void* arg) {
    int thread_id = *(int*)arg;
    int iterations = 100000;

    printf("线程 %d 开始 (有互斥锁)\n", thread_id);

    for (int i = 0; i < iterations; i++) {
        // 使用互斥锁保护临界区
        pthread_mutex_lock(&mutex);
        shared_counter_with_mutex++;
        pthread_mutex_unlock(&mutex);
    }

    printf("线程 %d 结束\n", thread_id);
    return NULL;
}

/**
 * 演示并发执行的线程 - 展示线程交替执行
 */
void* thread_func_print(void* arg) {
    int thread_id = *(int*)arg;

    for (int i = 0; i < 5; i++) {
        printf("线程 %d: 执行步骤 %d\n", thread_id, i);
        // 让当前线程休眠，让其他线程有机会执行
        usleep(100000); // 休眠 100ms
    }

    return NULL;
}

int main() {
    pthread_t threads[4];
    int thread_ids[4] = {1, 2, 3, 4};
	
    for (int i = 0; i < 4; i++) {
        pthread_create(&threads[i], NULL, thread_func_print, &thread_ids[i]);
    }

    for (int i = 0; i < 4; i++) {
        pthread_join(threads[i], NULL);
    }


    shared_counter = 0;

    pthread_create(&threads[0], NULL, thread_func_no_mutex, &thread_ids[0]);
    pthread_create(&threads[1], NULL, thread_func_no_mutex, &thread_ids[1]);

    pthread_join(threads[0], NULL);
    pthread_join(threads[1], NULL);

    return 0;
}
```

理论上这个并发程序的结果应该是 `200,000`，因为两个进程各自 iterations 为 `100,000` 并加了两次。对共享变量加了。

但是实际结果并非如此。
- **这是因为**这个操作并不是 atomically 执行的（所有的指令一次性执行）。
	- load the value from memory to register
	- Do increment 
	- save it back to memory

## 持久性 persistence

DRAM：volatile（易失的）存储数值
硬件以Input/Output, I/O 的形式出现.

**操作系统**管理磁盘的软件：file system。

操作系统不会为每个应用程序创建专用的虚拟磁盘。相反，塔假设用户经常需要共享 (share) 文件中的信息
- 一个经典的 C 语言编写测试过程
	- 创建和编辑 C 文件
	- 编译器转换为可执行文件
	- 运行可执行文件 e.g.`./main`
- **访问设备**是一件困难的事情，索性 OS 提供了 API 


## 一个历史的回顾
**多道程序时代**在小型机 (minicomputer) 后变得普遍

