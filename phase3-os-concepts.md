# 阶段三：操作系统核心概念

> **时间：** 6-8 周 | **难度：** ⭐⭐⭐☆☆ | **目标：** 理解操作系统在"硬件之上、应用之下"做了什么

---

## 📌 学习目标

- 理解操作系统的四大核心模块
- 能解释一个程序从启动到退出的全过程
- 理解并发与同步问题
- **动手写一个 Mini-OS**（最重要！）

---

## 📖 第1周：操作系统概览与进程管理

### Day 1-2：操作系统是什么
- [ ] OS 的职责：资源管理 + 抽象 + 隔离
- [ ] OS 发展简史（批处理 → 多道程序 → 分时 → 现代 OS）
- [ ] 内核态 vs 用户态
- [ ] 系统调用是什么（用户态进入内核态的唯一入口）
- [ ] 中断与异常

### Day 3-5：进程管理
- [ ] 进程概念：运行中的程序 = 代码 + 数据 + 栈 + PCB
- [ ] 进程控制块 (PCB / task_struct)
- [ ] 进程状态转换图：
  ```
  New → Ready ⇄ Running → Terminated
           ↕
        Waiting/Blocked
  ```
- [ ] 进程创建：fork() 深入理解
  ```c
  pid_t pid = fork();
  if (pid == 0) {
      // 子进程
  } else if (pid > 0) {
      // 父进程
  }
  ```
- [ ] exec 系列函数（替换进程映像）
- [ ] 孤儿进程、僵尸进程、守护进程
- [ ] 写代码观察 fork 的写时复制 (COW) 行为

### Day 6-7：进程调度
- [ ] 调度时机
- [ ] 调度算法：
  - FCFS（先来先服务）
  - SJF（短作业优先）
  - 优先级调度
  - 时间片轮转 (Round Robin)
  - 多级反馈队列
- [ ] Linux CFS（完全公平调度器）基本思想
- [ ] 上下文切换的开销

### 📝 第1周练习
1. 用 fork 创建进程树，观察父子关系
2. 故意制造僵尸进程并清理
3. 用 C 写一个多进程并发下载模拟程序

---

## 📖 第2周：线程与并发

### Day 8-10：线程基础
- [ ] 线程 vs 进程（共享什么，独有什么）
- [ ] POSIX 线程 (pthread)：
  ```c
  pthread_create, pthread_join, pthread_exit
  ```
- [ ] 用户线程 vs 内核线程
- [ ] Linux 中线程的实现（task_struct，轻量级进程）

### Day 11-14：同步与互斥 ⭐ 重点
- [ ] 竞态条件 (Race Condition)
- [ ] 临界区问题
- [ ] 互斥锁 (mutex)：
  ```c
  pthread_mutex_lock / unlock
  ```
- [ ] 信号量 (semaphore)：
  ```c
  sem_wait / sem_post
  ```
- [ ] 条件变量 (condition variable)：
  ```c
  pthread_cond_wait / signal
  ```
- [ ] 自旋锁 (spinlock) — 内核中大量使用
- [ ] 死锁：产生条件、预防、检测
- [ ] **读者-写者问题**（经典同步问题）
- [ ] **哲学家就餐问题**

### 📝 第2周练习
1. 多线程归并排序
2. 生产者-消费者模型（用 mutex + cond）
3. 哲学家就餐的实现与分析
4. 故意制造死锁，然后修复

---

## 📖 第3周：内存管理

### Day 15-17：内存管理基础
- [ ] 逻辑地址 vs 物理地址
- [ ] 内存分配方式：
  - 连续分配（首次适配、最佳适配）
  - 分页 (Paging)
  - 分段 (Segmentation)
- [ ] 页表与 TLB（快表）
- [ ] 多级页表

### Day 18-19：虚拟内存
- [ ] 虚拟内存的概念与动机
- [ ] 地址翻译过程（MMU 硬件）
- [ ] 缺页中断 (Page Fault)
- [ ] 页面置换算法：
  - FIFO, LRU, Clock（二次机会）
- [ ] 写时复制 (Copy-on-Write)
- [ ] Linux 中的伙伴系统 (Buddy System) 和 slab 分配器

### Day 20-21：实际动手
- [ ] 查看 `/proc/meminfo`
- [ ] 查看 `/proc/<pid>/maps` — 进程内存映射
- [ ] 用 `mmap` 做文件映射
- [ ] 理解 `malloc` 底层发生了什么（brk vs mmap）

### 📝 第3周练习
1. 用 C 实现一个简单的内存分配器（malloc/free）
2. 实现 LRU 页面置换算法的模拟器
3. 分析一个程序的内存使用（VSS, RSS, PSS）

---

## 📖 第4周：文件系统与I/O

### Day 22-24：文件系统
- [ ] 文件系统概念：文件、目录、inode
- [ ] 文件分配方式（连续、链接、索引）
- [ ] 空闲空间管理（位图、链表）
- [ ] VFS（虚拟文件系统交换）— Linux 的精妙抽象
- [ ] 理解 inode、dentry、superblock、file 结构体
- [ ] 硬链接 vs 软链接（符号链接）
- [ ] 查看 inode 信息：`stat`, `df -i`

### Day 25-26：I/O 系统
- [ ] I/O 方式：
  - 轮询 (Polling)
  - 中断驱动 (Interrupt-driven)
  - DMA (Direct Memory Access)
- [ ] 阻塞 I/O vs 非阻塞 I/O
- [ ] select / poll / epoll（I/O 多路复用）
- [ ] 了解 Linux 中 epoll 的高效实现

### Day 27-28：设备管理
- [ ] 块设备 vs 字符设备
- [ ] 设备驱动程序模型
- [ ] 设备文件 (`/dev/`)

### 📝 第4周练习
1. 实现一个玩具文件系统（FUSE）
2. 写一个 epoll 实现的简易 HTTP 服务器
3. 查看并理解 `strace ls` 的输出

---

## 📖 第5-6周：⚡ Mini-OS 项目（阶段三核心）

> **这是整个学习路线中最重要的项目之一。** 写一个能从裸机启动的微型内核。

### 项目目标
- 从引导程序 (bootloader) 启动
- 进入保护模式 / 长模式
- 设置 GDT / IDT
- 实现中断处理
- 实现简单的内存管理（物理页分配）
- 实现简单的进程切换（上下文切换）
- 实现系统调用接口
- 能在屏幕上打印信息

### 实施步骤

#### Step 1：环境搭建
- [ ] 安装交叉编译工具链（i686-elf-gcc）
- [ ] 安装 QEMU 模拟器
- [ ] 安装 NASM 汇编器
- [ ] 搭建 Makefile 编译系统

#### Step 2：启动
- [ ] 写引导扇区（Boot Sector，512 字节）
- [ ] 从实模式切换到保护模式
- [ ] 加载内核到内存
- [ ] 跳转到内核入口

#### Step 3：基础输出
- [ ] 操作 VGA 文本模式显存（0xB8000）
- [ ] 实现 print 函数（打印字符串、数字、十六进制）

#### Step 4：中断
- [ ] 设置 IDT（中断描述符表）
- [ ] 实现中断处理函数
- [ ] 重映射 PIC（可编程中断控制器）
- [ ] 实现键盘中断处理
- [ ] 实现简单的 shell（读取键盘输入）

#### Step 5：内存管理
- [ ] 物理内存探测（通过 BIOS / multiboot）
- [ ] 实现物理页帧分配器
- [ ] （可选）实现简单的虚拟内存和分页

#### Step 6：多任务
- [ ] 实现 task_struct
- [ ] 实现上下文切换（保存/恢复寄存器）
- [ ] 实现协作式多任务调度

### 📝 Mini-OS 最终效果
```
Booting MyOS...
[OK] GDT initialized
[OK] IDT initialized
[OK] Physical memory: 128 MB
[OK] Paging enabled
[OK] Multitasking started

MyOS> help
Available commands: help, meminfo, tasks, reboot
MyOS> meminfo
Total: 128 MB, Used: 2 MB, Free: 126 MB
MyOS>
```

---

## ✅ 阶段三检查清单

- [ ] 能画出进程状态转换图并解释每种状态
- [ ] 能手写生产者-消费者模型（多线程）
- [ ] 能解释虚拟地址到物理地址的转换过程
- [ ] 理解 inode 是什么，硬链接和软链接的区别
- [ ] **完成了 Mini-OS 项目！**（最重要的成果）
- [ ] 能解释 Linux 中 read() 从应用到内核的完整流程

---

## 📚 阶段三推荐资源

- **必读：** 《操作系统概念》(Silberschatz) — 恐龙书
- **参考：** 《现代操作系统》(Tanenbaum)
- **参考：** 《深入理解计算机系统》(CS:APP) — 第8章
- **Mini-OS 教程：**
  - [OSDev Wiki](https://wiki.osdev.org/) — 圣经级参考资料
  - [Writing a Simple Operating System from Scratch](https://www.cs.bham.ac.uk/~exr/lectures/opsys/10_11/lectures/os-dev.pdf) (Nick Blundell)
  - [The little book about OS development](https://littleosbook.github.io/)
