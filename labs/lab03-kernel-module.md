# Lab 3：第一个内核模块

> **目标：** 编写、编译、加载一个 Linux 内核模块
>
> **时间：** 2-3 天
>
> **学到的概念：** 内核模块结构、printk、insmod/rmmod、内核日志、Makefile 构建

---

## 为什么内核模块是嵌入式 Linux 的核心技能？

```
用户空间                   内核空间
────────                   ────────
应用程序                    内核模块 (.ko)
    │                          │
    │  open/read/write/ioctl   │
    ├─────────────────────────►│
    │                          ├── 操作硬件寄存器
    │                          ├── 响应中断
    │                          ├── 在内核日志输出
    │                          └── 暴露 /dev 或 /sys 接口
```

嵌入式 Linux 的驱动开发 = 写内核模块。

---

## Step 1：准备内核源码

```bash
cd ~/embedded-linux-lab/modules
mkdir hello-module && cd hello-module

# 从 Lab 1 的内核源码开始（确保是同一份内核）
# 或者从 Buildroot 的 output/build/ 中找到内核源码
KERNEL_DIR=~/embedded-linux-lab/kernel/linux-5.15.180
```

---

## Step 2：写模块代码

```c
// hello_module.c
#include <linux/module.h>    // 所有模块都需要
#include <linux/kernel.h>    // printk, KERN_INFO 等
#include <linux/init.h>      // __init, __exit 宏

// 模块加载时的函数
static int __init hello_init(void)
{
    printk(KERN_INFO "Hello, kernel! Module loaded.\n");
    printk(KERN_INFO "Current process: %s (PID: %d)\n",
           current->comm, current->pid);
    return 0;  // 返回 0 表示加载成功
}

// 模块卸载时的函数
static void __exit hello_exit(void)
{
    printk(KERN_INFO "Goodbye, kernel! Module unloaded.\n");
}

// 注册模块的入口和出口
module_init(hello_init);
module_exit(hello_exit);

// 模块元信息（必须）
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("My first kernel module");
```

---

## Step 3：写 Makefile

```makefile
# Makefile for hello_module
# 内核模块需要用内核构建系统来编译

# 指明要编译的模块目标
obj-m := hello_module.o

# 内核源码路径（根据你的实际情况修改）
KDIR := $(HOME)/embedded-linux-lab/kernel/linux-5.15.180

# 如果内核源码不在当前架构下，需要指定架构和编译器
ARCH := arm
CROSS_COMPILE := arm-linux-gnueabihf-

# 当前目录
PWD := $(shell pwd)

all:
    $(MAKE) -C $(KDIR) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) \
        M=$(PWD) modules

clean:
    $(MAKE) -C $(KDIR) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) \
        M=$(PWD) clean
```

---

## Step 4：编译模块

```bash
# 先确保内核已经编译过（模块编译依赖内核的 Module.symvers）
cd ~/embedded-linux-lab/kernel/linux-5.15.180

# 如果之前只编译了 zImage，需要补编译模块
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules_prepare

# 回到模块目录编译
cd ~/embedded-linux-lab/modules/hello-module
make
```

**产物：**
```
hello_module.ko    ← 这就是你的内核模块！
hello_module.mod.c
hello_module.mod.o
hello_module.o
modules.order
Module.symvers
```

```bash
# 检查模块信息
file hello_module.ko
# hello_module.ko: ELF 32-bit LSB relocatable, ARM, version 1 (SYSV)

modinfo hello_module.ko
# filename:       hello_module.ko
# license:        GPL
# author:         Your Name
# description:    My first kernel module
# depends:
# name:           hello_module
# vermagic:       5.15.180 SMP mod_unload ARMv7 p2v8
```

---

## Step 5：在 QEMU 中测试

### 5.1 把模块传进 QEMU

**方法一：直接打包进 rootfs**

```bash
# 假设你的 rootfs 在 ~/embedded-linux-lab/rootfs/
cd ~/embedded-linux-lab/rootfs
mkdir -p rootfs/root/modules
cp ~/embedded-linux-lab/modules/hello-module/hello_module.ko rootfs/root/modules/

# 重新制作 rootfs.img
dd if=/dev/zero of=rootfs.img bs=1M count=64
mkfs.ext4 rootfs.img
mkdir -p /tmp/mnt
sudo mount rootfs.img /tmp/mnt
sudo cp -a rootfs/* /tmp/mnt/
sudo umount /tmp/mnt
```

**方法二：用 scp（如果 QEMU 有网络）**

```bash
# 启动 QEMU 时加网络转发
# -net nic,model=lan9118 -net user,hostfwd=tcp::2222-:22

# 然后在宿主机上
scp -P 2222 hello_module.ko root@localhost:/root/
```

### 5.2 加载和测试

```bash
# 在 QEMU 的 shell 中：

# 加载模块
insmod /root/modules/hello_module.ko

# 查看内核日志
dmesg | tail
# [  123.456] Hello, kernel! Module loaded.
# [  123.456] Current process: insmod (PID: 123)

# 查看已加载的模块
lsmod | grep hello
# hello_module          16384  0

# 查看模块信息
cat /proc/modules | grep hello

# 卸载模块
rmmod hello_module

# 再看日志
dmesg | tail
# [  234.567] Goodbye, kernel! Module unloaded.
```

---

## Step 6：添加模块参数

```c
// hello_param.c — 带参数的模块
#include <linux/module.h>
#include <linux/moduleparam.h>
#include <linux/kernel.h>

// 定义模块参数
static char *who = "world";       // 字符串参数
static int times = 1;             // 整数参数
static bool verbose = false;      // 布尔参数

// 注册参数（使其可以从命令行传入）
module_param(who, charp, 0644);
MODULE_PARM_DESC(who, "Name to greet");
module_param(times, int, 0644);
MODULE_PARM_DESC(times, "Number of greetings");
module_param(verbose, bool, 0644);
MODULE_PARM_DESC(verbose, "Enable verbose output");

static int __init hello_init(void)
{
    int i;
    for (i = 0; i < times; i++) {
        printk(KERN_INFO "Hello, %s! (%d/%d)\n", who, i+1, times);
    }
    if (verbose) {
        printk(KERN_INFO "Verbose mode: module loaded at %pS\n",
               hello_init);
    }
    return 0;
}

static void __exit hello_exit(void)
{
    printk(KERN_INFO "Goodbye, %s!\n", who);
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A kernel module with parameters");
```

测试：
```bash
# 不带参数
insmod hello_param.ko
# → Hello, world! (1/1)

# 带参数
insmod hello_param.ko who="embedded" times=5 verbose=1
# → Hello, embedded! (1/5)
#   Hello, embedded! (2/5)
#   Hello, embedded! (3/5)
#   Hello, embedded! (4/5)
#   Hello, embedded! (5/5)
#   Verbose mode: module loaded at ...

# 也支持在 /sys 下动态修改
cat /sys/module/hello_param/parameters/who
# world
echo "linux" > /sys/module/hello_param/parameters/who
cat /sys/module/hello_param/parameters/who
# linux
```

---

## Step 7：创建 /proc 文件接口

```c
// hello_proc.c — 通过 /proc 暴露信息的模块
#include <linux/module.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>

static int module_count = 0;

// seq_file 读取回调
static int hello_proc_show(struct seq_file *m, void *v)
{
    seq_printf(m, "Hello from kernel module!\n");
    seq_printf(m, "Module loaded: %d times\n", ++module_count);
    seq_printf(m, "Current jiffies: %lu\n", jiffies);
    return 0;
}

static int hello_proc_open(struct inode *inode, struct file *file)
{
    return single_open(file, hello_proc_show, NULL);
}

static const struct proc_ops hello_proc_ops = {
    .proc_open = hello_proc_open,
    .proc_read = seq_read,
    .proc_release = single_release,
};

static struct proc_dir_entry *entry;

static int __init hello_init(void)
{
    entry = proc_create("hello_module", 0444, NULL, &hello_proc_ops);
    if (!entry) {
        pr_err("Failed to create /proc/hello_module\n");
        return -ENOMEM;
    }
    pr_info("Created /proc/hello_module\n");
    return 0;
}

static void __exit hello_exit(void)
{
    proc_remove(entry);
    pr_info("Removed /proc/hello_module\n");
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
```

测试：
```bash
insmod hello_proc.ko
cat /proc/hello_module
# Hello from kernel module!
# Module loaded: 1 times
# Current jiffies: 4294967295

cat /proc/hello_module
# Hello from kernel module!
# Module loaded: 2 times     ← 每次读取计数增加
# Current jiffies: 4294969295
```

---

## 💡 你学到了什么

| 概念 | 对应代码 |
|------|---------|
| 模块加载/卸载 | `module_init()` / `module_exit()` |
| 内核日志 | `printk(KERN_INFO ...)` |
| 模块参数 | `module_param()` |
| /proc 接口 | `proc_create()` + `seq_file` |
| 内核构建系统 | `obj-m := xxx.o` + `make -C $(KDIR) M=$(PWD) modules` |

---

## 🐞 常见问题

| 问题 | 解决 |
|------|------|
| `Invalid module format` | 模块和内核版本不匹配，检查 `vermagic` |
| `Unknown symbol` | 依赖的其他模块未加载 |
| `insmod: can't insert` | 看 `dmesg` 里的具体错误信息 |
| 编译报 `build` 目录不存在 | 先 `make modules_prepare` |

---

## ⏭️ 下一步

你已经会写内核模块了。接下来写一个真正的**字符设备驱动**——让用户空间程序能 `open/read/write` 你的内核模块。

➡️ [Lab 4: 字符设备驱动](./lab04-char-driver.md)
