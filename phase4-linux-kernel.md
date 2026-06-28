# 阶段四：Linux 内核初步

> **时间：** 8-10 周 | **难度：** ⭐⭐⭐⭐☆ | **目标：** 理解内核架构，能写字符设备驱动

---

## 📌 学习目标

- 理解 Linux 内核的整体架构
- 掌握内核模块开发
- 理解字符设备驱动框架
- 理解设备树 (Device Tree)
- 能编写简单的平台驱动
- 知道如何配置和编译内核

---

## 📖 第1周：内核概览与源码探索

### Day 1-3：内核架构总览
- [ ] 内核是什么，不是什么（不是完整的 OS）
- [ ] 整体架构图：
  ```
  ┌──────────────────────────┐
  │    用户空间应用程序        │
  ├──────────────────────────┤
  │    系统调用接口 (SCI)      │
  ├──────────────────────────┤
  │  进程管理  │  内存管理     │
  │  (调度)   │  (虚拟内存)   │
  ├───────────┼──────────────┤
  │  文件系统  │  网络栈       │
  │  (VFS)   │  (TCP/IP)    │
  ├───────────┼──────────────┤
  │    设备驱动 (字符/块/网络)  │
  ├──────────────────────────┤
  │  Arch 相关代码 (ARM/x86/.) │
  ├──────────────────────────┤
  │       硬件 (CPU/内存/外设)  │
  └──────────────────────────┘
  ```
- [ ] 内核源码目录结构概览：
  ```
  kernel/    — 进程调度、信号等核心代码
  mm/        — 内存管理
  fs/        — 文件系统 (VFS + 各文件系统实现)
  drivers/   — 设备驱动（最大目录）
  arch/      — 架构相关代码 (arch/arm/, arch/x86/)
  include/   — 头文件
  net/       — 网络协议栈
  block/     — 块设备层
  ipc/       — 进程间通信
  init/      — 内核初始化
  lib/       — 内核库函数
  scripts/   — 编译辅助脚本
  ```

### Day 4-5：内核编译
- [ ] 获取内核源码：
  ```bash
  git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
  # 或
  wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.tar.xz
  ```
- [ ] 配置内核（x86 上练习）：
  ```bash
  make x86_64_defconfig  # 从默认配置开始
  make menuconfig         # 图形化配置界面
  ```
- [ ] 编译内核：
  ```bash
  make -j$(nproc)        # 并行编译
  ```
- [ ] 在 QEMU 中启动自己编译的内核：
  ```bash
  qemu-system-x86_64 -kernel arch/x86/boot/bzImage \
    -initrd initramfs.img -nographic
  ```
- [ ] 理解内核配置系统（Kconfig 语法）

### Day 6-7：内核日志与调试
- [ ] `printk` 与日志级别：
  ```c
  printk(KERN_INFO "Hello from kernel\n");
  printk(KERN_ERR "Error: %d\n", err);
  ```
- [ ] `dmesg` / `journalctl -k` 查看内核日志
- [ ] `/proc` 和 `/sys` 文件系统
- [ ] `ftrace` 基础使用
- [ ] 了解 `kgdb` / `kdb`
- [ ] 看懂内核 Oops 信息

### 📝 第1周练习
1. 在 QEMU 里启动自己编译的内核
2. 修改内核版本字符串，重新编译验证
3. 通过 `/proc` 和 `/sys` 探索内核数据结构

---

## 📖 第2周：内核模块

### Day 8-10：模块基础
- [ ] Hello World 内核模块：
  ```c
  #include <linux/module.h>
  #include <linux/kernel.h>

  static int __init hello_init(void) {
      printk(KERN_INFO "Hello, kernel!\n");
      return 0;
  }

  static void __exit hello_exit(void) {
      printk(KERN_INFO "Goodbye, kernel!\n");
  }

  module_init(hello_init);
  module_exit(hello_exit);

  MODULE_LICENSE("GPL");
  MODULE_AUTHOR("Your Name");
  MODULE_DESCRIPTION("A simple hello world module");
  ```
- [ ] Makefile 写法
- [ ] 编译、加载、卸载模块：
  ```bash
  make
  sudo insmod hello.ko     # 插入模块
  lsmod | grep hello        # 查看模块
  sudo rmmod hello          # 移除模块
  dmesg | tail              # 查看输出
  ```
- [ ] `modinfo hello.ko` 查看模块信息
- [ ] 模块参数传递

### Day 11-12：模块进阶
- [ ] 模块依赖关系
- [ ] 导出符号 (`EXPORT_SYMBOL`)
- [ ] 模块许可证（GPL vs 非 GPL）
- [ ] 创建一个多文件的模块
- [ ] `/proc` 文件创建 (`proc_create`)

### Day 13-14：字符设备驱动框架

> 这是驱动开发的起点！

- [ ] 设备号（主设备号、次设备号）：
  ```c
  dev_t dev = MKDEV(major, minor);
  register_chrdev_region / alloc_chrdev_region
  ```
- [ ] `file_operations` 结构体：
  ```c
  static struct file_operations fops = {
      .owner = THIS_MODULE,
      .open = my_open,
      .release = my_release,
      .read = my_read,
      .write = my_write,
      .unlocked_ioctl = my_ioctl,
  };
  ```
- [ ] 创建设备类和设备节点：
  ```c
  class_create / device_create
  ```
- [ ] 用户空间与内核空间的数据交换：
  ```c
  copy_to_user / copy_from_user
  ```

### 📝 第2周练习
1. 写一个能记录调用次数的简单字符设备
2. 写一个 `/proc` 文件，显示模块内部状态
3. 模块间通信（一个模块导出函数，另一个调用）

---

## 📖 第3周：字符设备驱动实战

### Day 15-18：实现一个完整的字符设备驱动

**项目：内核缓冲区设备**
- [ ] open/release：模块引用计数管理
- [ ] read：从内核缓冲区读数据到用户空间
- [ ] write：从用户空间写数据到内核缓冲区
- [ ] ioctl：支持多种控制命令
- [ ] lseek：支持文件位置移动
- [ ] 并发保护：用 mutex/spinlock 保护共享数据
- [ ] 测试程序：写用户空间程序测试所有功能

### Day 19-21：高级驱动话题
- [ ] 阻塞 I/O（等待队列）：
  ```c
  wait_event_interruptible / wake_up_interruptible
  ```
- [ ] 非阻塞 I/O
- [ ] poll 支持
- [ ] 异步通知 (fasync)
- [ ] `mmap` 实现（将内核内存映射到用户空间）

### 📝 第3周练习
1. 实现生产者-消费者管道设备（有数据才能读，有空间才能写）
2. 给之前的字符设备增加 mmap 支持
3. 测试多进程同时读写同一设备的并发正确性

---

## 📖 第4周：平台驱动与设备树

### Day 22-24：Linux 设备模型
- [ ] 设备 (device)、驱动 (driver) 、总线 (bus) 三要素
- [ ] platform bus（平台总线）：
  - `platform_device`（设备端）
  - `platform_driver`（驱动端）
  - probe/remove 机制
- [ ] 设备与驱动的匹配（compatible 字符串、id_table）
- [ ] 理解 driver core 的 sysfs 体现

### Day 25-26：设备树 (Device Tree) ⭐ 嵌入式核心

- [ ] 为什么需要设备树（ARM 没有 PCI 那样的自枚举总线）
- [ ] DTS（设备树源文件）语法：
  ```dts
  /dts-v1/;

  / {
      model = "My Board";
      compatible = "myboard";

      memory {
          device_type = "memory";
          reg = <0x80000000 0x10000000>;  /* 256MB */
      };

      mydevice: mydevice@40000000 {
          compatible = "mycompany,mydevice";
          reg = <0x40000000 0x1000>;
          interrupts = <42>;
      };
  };
  ```
- [ ] DTC（设备树编译器）
- [ ] DTB（设备树二进制）的生成与加载
- [ ] 驱动中解析设备树：
  ```c
  // 获取 compatible 匹配
  of_match_table
  // 读取属性
  of_property_read_u32 / of_property_read_string
  // 获取 reg 资源
  platform_get_resource
  ```
- [ ] 查看设备树（`/sys/firmware/fdt` 或 `/proc/device-tree`）

### Day 27-28：中断处理
- [ ] 中断上下文 vs 进程上下文
- [ ] 注册中断处理函数：
  ```c
  request_irq / devm_request_irq
  ```
- [ ] 顶半部 (Top Half) vs 底半部 (Bottom Half)
- [ ] 底半部机制：
  - tasklet（简单，传统）
  - workqueue（可以睡眠）
  - threaded IRQ（现在推荐）
- [ ] 中断共享
- [ ] 写一个按键中断驱动（用 GPIO 中断）

### 📝 第4周练习
1. 写一个平台驱动框架（platform driver + device 匹配）
2. 解析设备树中的自定义属性
3. 写一个模拟的中断处理驱动（用 tasklet/workqueue 做底半部）

---

## 📖 第5-6周：内核深入与驱动实战

### Day 29-34：内核同步机制再深入
- [ ] 自旋锁 (spinlock) — 不可睡眠的锁
- [ ] 互斥锁 (mutex) — 可睡眠的锁
- [ ] 读写锁 (rwlock / rwsem)
- [ ] RCU (Read-Copy-Update) — 内核中广泛使用
- [ ] 原子操作 (atomic_t)
- [ ] 内存屏障 (`mb()`, `rmb()`, `wmb()`)
- [ ] 理解 preempt_disable / local_irq_save
- [ ] 选择一个机制时：该在什么上下文中？能否睡眠？

### Day 35-38：内核定时器与时间管理
- [ ] jiffies 与 HZ
- [ ] 内核定时器 (`timer_list`)
- [ ] 高精度定时器 (hrtimer)
- [ ] 内核中的延时：`mdelay`, `udelay`, `msleep`, `schedule_timeout`

### Day 39-42：I2C/SPI 驱动框架
- [ ] I2C 驱动模型（adaptor, client, driver）
- [ ] SPI 驱动模型（master, slave, driver）
- [ ] 写一个 I2C 设备驱动（如 EEPROM、温度传感器）
- [ ] 理解 regmap 抽象层

### 📝 第5-6周练习：阶段四综合项目
**项目：多功能虚拟设备驱动**
- 支持字符设备接口
- 支持 I/O 控制（ioctl）
- 使用平台驱动 + 设备树模型
- 实现阻塞/非阻塞读写
- 使用内核定时器模拟硬件行为
- 使用锁机制保证并发安全
- 写完整的用户空间测试程序
- 代码符合 Linux 内核编码规范（`scripts/checkpatch.pl` 检查通过）

---

## ✅ 阶段四检查清单

- [ ] 能自己编译内核并在 QEMU 中启动
- [ ] 能编写、加载、调试内核模块
- [ ] 理解 file_operations 中每个函数指针的作用
- [ ] 能独立写字符设备驱动
- [ ] 理解设备树的作用，能读写 DTS 文件
- [ ] 理解平台总线 (platform bus) 的匹配机制
- [ ] 知道什么时候用哪种内核同步机制
- [ ] 通过 checkpatch.pl 检查代码

---

## 📚 阶段四推荐资源

- **必读：** 《Linux 设备驱动程序》(LDD3) — 虽然是 2.6 内核的书，但核心思想不变
- **必读：** 《Linux 内核设计与实现》(Robert Love) — 内核入门最佳读物
- **参考：** 《深入 Linux 内核架构》(Wolfgang Mauerer)
- **参考：** [Linux 内核文档](https://www.kernel.org/doc/html/latest/)
- **参考：** [The Linux Kernel Module Programming Guide](https://sysprog21.github.io/lkmpg/)
- **参考：** [Bootlin 的 Linux 内核与驱动培训资料](https://bootlin.com/doc/training/linux-kernel/)
- **工具：** `scripts/checkpatch.pl` — 代码风格检查
- **工具：** `sparse` — 静态分析工具
