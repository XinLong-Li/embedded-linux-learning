# Lab 4：字符设备驱动

> **目标：** 写一个完整的字符设备驱动，用户空间程序可以 open/read/write/ioctl
>
> **时间：** 3-5 天
>
> **核心产出：** 一个支持多进程并发读写的内核缓冲区设备 + 用户空间测试程序

---

## 驱动架构总览

```
用户空间程序 (test_char.c)
    │
    │  fd = open("/dev/mybuf0", O_RDWR)
    │  read(fd, buf, size)
    │  write(fd, buf, size)
    │  ioctl(fd, CMD, arg)
    │  close(fd)
    │
────┼── 系统调用 ──────────────────────────────
    │
    ▼
内核模块 (mybuf.ko)
    │
    ├── mybuf_open()     ← 分配资源, 引用计数+1
    ├── mybuf_read()     ← copy_to_user() 传数据到用户空间
    ├── mybuf_write()    ← copy_from_user() 从用户空间取数据
    ├── mybuf_ioctl()    ← 控制命令
    └── mybuf_release()  ← 释放资源, 引用计数-1
    │
    ▼
内核缓冲区 (kmalloc 分配的内存)
```

---

## Step 1：设备号注册

内核通过**主设备号 (major)** 找到驱动，通过**次设备号 (minor)** 区分同一驱动的不同设备实例。

```c
// mybuf.c — 完整字符设备驱动
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/fs.h>         // file_operations, 设备号注册
#include <linux/cdev.h>       // cdev 结构体
#include <linux/device.h>     // class_create, device_create
#include <linux/uaccess.h>    // copy_to_user, copy_from_user
#include <linux/slab.h>       // kmalloc, kfree
#include <linux/mutex.h>      // mutex 锁
#include <linux/ioctl.h>      // ioctl 命令定义

#define DEVICE_NAME  "mybuf"
#define CLASS_NAME   "mybuf_class"
#define BUFFER_SIZE  4096
#define DEVICE_COUNT 2        // 创建 mybuf0 和 mybuf1 两个设备

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple character device driver");

// ==================== 设备数据结构 ====================
struct mybuf_dev {
    struct cdev cdev;         // 内核字符设备结构体
    char *buffer;             // 内核缓冲区
    size_t size;              // 缓冲区当前数据大小
    struct mutex lock;        // 并发保护锁
    int dev_id;               // 设备编号
};

static struct mybuf_dev *devices;      // 设备数组
static dev_t dev_num;                  // 设备号
static struct class *dev_class;        // 设备类
static int major;                      // 主设备号

// ==================== 文件操作 ====================
static int mybuf_open(struct inode *inode, struct file *filp)
{
    struct mybuf_dev *dev;

    // 从 inode 中获取设备实例（这是标准做法）
    dev = container_of(inode->i_cdev, struct mybuf_dev, cdev);
    filp->private_data = dev;  // 保存设备指针

    pr_info("mybuf%d: opened\n", dev->dev_id);
    return 0;
}

static ssize_t mybuf_read(struct file *filp, char __user *buf,
                           size_t count, loff_t *f_pos)
{
    struct mybuf_dev *dev = filp->private_data;
    ssize_t retval = 0;

    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;

    // 检查是否有数据可读
    if (*f_pos >= dev->size)
        goto out;  // EOF

    // 不能读超过实际数据量
    if (*f_pos + count > dev->size)
        count = dev->size - *f_pos;

    // 核心操作：内核空间 → 用户空间
    if (copy_to_user(buf, dev->buffer + *f_pos, count)) {
        retval = -EFAULT;
        goto out;
    }

    *f_pos += count;
    retval = count;

    pr_info("mybuf%d: read %zu bytes (pos=%lld)\n",
            dev->dev_id, count, *f_pos);

out:
    mutex_unlock(&dev->lock);
    return retval;
}

static ssize_t mybuf_write(struct file *filp, const char __user *buf,
                            size_t count, loff_t *f_pos)
{
    struct mybuf_dev *dev = filp->private_data;
    ssize_t retval = -ENOMEM;

    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;

    // 缓冲区已满（简化处理，不扩展）
    if (*f_pos >= BUFFER_SIZE)
        goto out;

    if (*f_pos + count > BUFFER_SIZE)
        count = BUFFER_SIZE - *f_pos;

    // 核心操作：用户空间 → 内核空间
    if (copy_from_user(dev->buffer + *f_pos, buf, count)) {
        retval = -EFAULT;
        goto out;
    }

    *f_pos += count;
    dev->size = max(dev->size, (size_t)*f_pos);
    retval = count;

    pr_info("mybuf%d: wrote %zu bytes (total=%zu)\n",
            dev->dev_id, count, dev->size);

out:
    mutex_unlock(&dev->lock);
    return retval;
}

// ioctl 命令定义（标准的 Linux ioctl 编码方式）
#define MYBUF_MAGIC 'k'
#define MYBUF_CLEAR    _IO(MYBUF_MAGIC, 0)   // 清空缓冲区
#define MYBUF_GETSIZE  _IOR(MYBUF_MAGIC, 1, size_t)  // 获取数据量
#define MYBUF_RESIZE   _IOW(MYBUF_MAGIC, 2, size_t)  // 预留（未实现）

static long mybuf_ioctl(struct file *filp, unsigned int cmd,
                         unsigned long arg)
{
    struct mybuf_dev *dev = filp->private_data;

    if (_IOC_TYPE(cmd) != MYBUF_MAGIC)
        return -ENOTTY;

    switch (cmd) {
    case MYBUF_CLEAR:
        mutex_lock(&dev->lock);
        memset(dev->buffer, 0, BUFFER_SIZE);
        dev->size = 0;
        mutex_unlock(&dev->lock);
        pr_info("mybuf%d: buffer cleared\n", dev->dev_id);
        return 0;

    case MYBUF_GETSIZE:
        return put_user(dev->size, (size_t __user *)arg);

    default:
        return -ENOTTY;
    }
}

static int mybuf_release(struct inode *inode, struct file *filp)
{
    struct mybuf_dev *dev = filp->private_data;
    pr_info("mybuf%d: released\n", dev->dev_id);
    return 0;
}

// 文件操作表 — 这是驱动和 VFS 之间的桥梁
static struct file_operations mybuf_fops = {
    .owner          = THIS_MODULE,
    .open           = mybuf_open,
    .read           = mybuf_read,
    .write          = mybuf_write,
    .unlocked_ioctl = mybuf_ioctl,
    .release        = mybuf_release,
};

// ==================== 模块初始化 ====================
static int __init mybuf_init(void)
{
    int i, ret;

    // 1. 动态分配设备号
    ret = alloc_chrdev_region(&dev_num, 0, DEVICE_COUNT, DEVICE_NAME);
    if (ret) {
        pr_err("Failed to allocate device number\n");
        return ret;
    }
    major = MAJOR(dev_num);
    pr_info("Allocated major=%d, minor range 0-%d\n", major, DEVICE_COUNT-1);

    // 2. 创建设备类（在 /sys/class/ 下出现）
    dev_class = class_create(CLASS_NAME);
    if (IS_ERR(dev_class)) {
        unregister_chrdev_region(dev_num, DEVICE_COUNT);
        return PTR_ERR(dev_class);
    }

    // 3. 为每个设备实例分配内存
    devices = kmalloc_array(DEVICE_COUNT, sizeof(*devices), GFP_KERNEL);
    if (!devices) {
        class_destroy(dev_class);
        unregister_chrdev_region(dev_num, DEVICE_COUNT);
        return -ENOMEM;
    }

    // 4. 初始化每个设备
    for (i = 0; i < DEVICE_COUNT; i++) {
        struct mybuf_dev *dev = &devices[i];

        // 分配缓冲区
        dev->buffer = kmalloc(BUFFER_SIZE, GFP_KERNEL);
        memset(dev->buffer, 0, BUFFER_SIZE);
        dev->size = 0;
        dev->dev_id = i;
        mutex_init(&dev->lock);

        // 初始化 cdev 并添加到内核
        cdev_init(&dev->cdev, &mybuf_fops);
        dev->cdev.owner = THIS_MODULE;
        ret = cdev_add(&dev->cdev, MKDEV(major, i), 1);
        if (ret) {
            pr_err("Failed to add cdev for device %d\n", i);
            goto cleanup;
        }

        // 在 /dev 下创建设备节点
        device_create(dev_class, NULL, MKDEV(major, i),
                      NULL, "mybuf%d", i);

        pr_info("Created /dev/mybuf%d\n", i);
    }

    pr_info("mybuf driver loaded (%d devices)\n", DEVICE_COUNT);
    return 0;

cleanup:
    while (--i >= 0) {
        device_destroy(dev_class, MKDEV(major, i));
        cdev_del(&devices[i].cdev);
        kfree(devices[i].buffer);
    }
    kfree(devices);
    class_destroy(dev_class);
    unregister_chrdev_region(dev_num, DEVICE_COUNT);
    return ret;
}

// ==================== 模块清理 ====================
static void __exit mybuf_exit(void)
{
    int i;
    for (i = 0; i < DEVICE_COUNT; i++) {
        device_destroy(dev_class, MKDEV(major, i));
        cdev_del(&devices[i].cdev);
        kfree(devices[i].buffer);
    }
    kfree(devices);
    class_destroy(dev_class);
    unregister_chrdev_region(dev_num, DEVICE_COUNT);
    pr_info("mybuf driver unloaded\n");
}

module_init(mybuf_init);
module_exit(mybuf_exit);
```

---

## Step 2：编译与部署

```makefile
obj-m := mybuf.o
KDIR := $(HOME)/embedded-linux-lab/kernel/linux-5.15.180
ARCH := arm
CROSS_COMPILE := arm-linux-gnueabihf-
PWD := $(shell pwd)

all:
    $(MAKE) -C $(KDIR) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) M=$(PWD) modules

clean:
    $(MAKE) -C $(KDIR) M=$(PWD) clean
```

---

## Step 3：编写用户空间测试程序

```c
// test_mybuf.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

#define MYBUF_MAGIC 'k'
#define MYBUF_CLEAR   _IO(MYBUF_MAGIC, 0)
#define MYBUF_GETSIZE _IOR(MYBUF_MAGIC, 1, size_t)

int main(int argc, char *argv[])
{
    int fd, ret;
    char write_buf[] = "Hello from userspace! This is a test of the character device driver.";
    char read_buf[256] = {0};
    size_t size;

    // 1. 打开设备
    fd = open("/dev/mybuf0", O_RDWR);
    if (fd < 0) {
        perror("open");
        return 1;
    }
    printf("[1] Opened /dev/mybuf0 (fd=%d)\n", fd);

    // 2. 写入数据
    ret = write(fd, write_buf, strlen(write_buf));
    printf("[2] Wrote %d bytes: \"%s\"\n", ret, write_buf);

    // 3. 通过 ioctl 查询数据大小
    ioctl(fd, MYBUF_GETSIZE, &size);
    printf("[3] Buffer size (via ioctl): %zu bytes\n", size);

    // 4. 从头读取数据
    lseek(fd, 0, SEEK_SET);
    ret = read(fd, read_buf, sizeof(read_buf) - 1);
    read_buf[ret] = '\0';
    printf("[4] Read %d bytes: \"%s\"\n", ret, read_buf);

    // 5. 验证数据一致性
    if (strcmp(write_buf, read_buf) == 0) {
        printf("[5] ✅ Data integrity check PASSED\n");
    } else {
        printf("[5] ❌ Data integrity check FAILED\n");
    }

    // 6. 清空缓冲区
    ioctl(fd, MYBUF_CLEAR);
    ioctl(fd, MYBUF_GETSIZE, &size);
    printf("[6] After clear, buffer size: %zu bytes\n", size);

    close(fd);
    return 0;
}
```

用 ARM 交叉编译器编译：
```bash
arm-linux-gnueabihf-gcc -static test_mybuf.c -o test_mybuf
```

---

## Step 4：在 QEMU 中运行

```bash
# 在 QEMU 中
insmod /root/modules/mybuf.ko
# dmesg 输出:
# Allocated major=248, minor range 0-1
# Created /dev/mybuf0
# Created /dev/mybuf1
# mybuf driver loaded (2 devices)

ls -l /dev/mybuf*
# crw------- 1 root root 248, 0 ... /dev/mybuf0
# crw------- 1 root root 248, 1 ... /dev/mybuf1

# 运行测试程序
/root/test_mybuf
# [1] Opened /dev/mybuf0 (fd=3)
# [2] Wrote 64 bytes: "Hello from userspace! ..."
# [3] Buffer size (via ioctl): 64 bytes
# [4] Read 64 bytes: "Hello from userspace! ..."
# [5] ✅ Data integrity check PASSED
# [6] After clear, buffer size: 0 bytes

# 查看驱动日志
dmesg | tail -20
# mybuf0: opened
# mybuf0: wrote 64 bytes (total=64)
# mybuf0: read 64 bytes (pos=64)
# mybuf0: buffer cleared
# mybuf0: released
```

---

## 💡 关键概念总结

| 概念 | 代码位置 | 作用 |
|------|---------|------|
| `file_operations` | 驱动中的 `.open/.read/.write/.ioctl` | 告诉内核"这些函数处理用户请求" |
| `cdev_add()` | `mybuf_init()` | 把驱动注册到 VFS |
| `device_create()` | `mybuf_init()` | 在 `/dev` 下创建节点 |
| `copy_to_user()` | `mybuf_read()` | **安全地**将数据从内核传到用户空间 |
| `copy_from_user()` | `mybuf_write()` | **安全地**将数据从用户空间传到内核 |
| `container_of()` | `mybuf_open()` | 从 `cdev` 结构体反推出整个设备结构体指针 |
| `mutex_lock()` | read/write 函数 | 防止多进程同时访问导致数据错乱 |

---

## 🔧 进阶挑战

1. **支持阻塞读取**：数据为空时 `read` 休眠等待，有数据后被唤醒（用到 `wait_queue`）
2. **支持 poll/select**：让用户程序可以用 `poll()` 检测是否有数据可读
3. **支持 mmap**：把内核缓冲区映射到用户空间，实现零拷贝
4. **压力测试**：同时启动 10 个进程读写 `/dev/mybuf0`，验证并发安全

---

## ⏭️ 下一步

你已经会写完整的内核驱动了。接下来用真实的 STM32MP157 开发板跑一遍！

➡️ [Lab 5: STM32MP157 真机启动](./lab05-real-board.md)
