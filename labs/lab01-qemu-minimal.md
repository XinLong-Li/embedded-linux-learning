# Lab 1：QEMU 最小嵌入式 Linux 系统

> **目标：** 从源码编译内核 + BusyBox，在 QEMU 中启动一个命令行 Linux
>
> **时间：** 2-3 天（首次），熟练后 30 分钟
>
> **学到的概念：** 内核编译、设备树、根文件系统、init 进程、交叉编译

---

## 📐 架构总览

```
你构建的东西:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  QEMU (模拟 ARM vexpress-a9 开发板)                       │
│  ┌───────────────────────────────────────────────────┐  │
│  │  根文件系统 (rootfs.img)                            │  │
│  │  ├── /bin/busybox   ← 所有命令 (ls, cat, sh, ...)   │  │
│  │  ├── /etc/inittab   ← init 配置                    │  │
│  │  ├── /etc/init.d/rcS ← 启动脚本                    │  │
│  │  └── /dev/*         ← 设备节点                     │  │
│  ├───────────────────────────────────────────────────┤  │
│  │  Linux 内核 (zImage + dtb)                         │  │
│  │  ├── 进程调度、内存管理、文件系统                      │  │
│  │  └── 设备驱动 (串口、块设备、网络)                     │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 1：编译 Linux 内核

### 1.1 获取源码

```bash
cd ~/embedded-linux-lab/kernel

# 推荐 Linux 5.15 LTS（稳定性最好）
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.180.tar.xz
tar xf linux-5.15.180.tar.xz
cd linux-5.15.180
```

> 💡 你也可以用 `git clone --depth=1 --branch v5.15.180 git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git`，但国内下载 `.tar.xz` 更快。

### 1.2 配置

```bash
# 使用 vexpress 默认配置（ARM 32位）
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_defconfig

# 可选：进入菜单微调
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

**menuconfig 中务必检查：**
```
General setup --->
    [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
        (initramfs.cpio.gz) Initramfs source file(s)  ← 可以留空

Kernel Features --->
    [*] Use the ARM EABI to compile the kernel

Device Drivers --->
    Character devices --->
        [*] Enable TTY
        Serial drivers --->
            <*> ARM AMBA PL011 serial port support
            [*]   Support for console on AMBA serial port
```

### 1.3 编译

```bash
# -j$(nproc) 使用所有 CPU 核心并行编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

**产物位置：**
- 内核镜像: `arch/arm/boot/zImage`（压缩内核）
- 设备树: `arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb`

---

## Step 2：编译 BusyBox

### 2.1 获取源码

```bash
cd ~/embedded-linux-lab
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
```

### 2.2 配置与编译

```bash
# 使用默认配置
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig

# 配置为静态链接（不需要外部 .so 库，省事）
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
# Settings → [*] Build static binary (no shared libs)

# 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)

# 安装到 _install 目录
make install
# _install/ 下有 bin/ sbin/ usr/ linuxrc
```

---

## Step 3：制作根文件系统

### 3.1 创建目录结构

```bash
cd ~/embedded-linux-lab/rootfs

# 复制 BusyBox 产物
mkdir -p rootfs
cp -r ../busybox-1.36.1/_install/* rootfs/

# 创建必要的目录
mkdir -p rootfs/{etc,proc,sys,dev,tmp,var,root,lib,usr/lib}

# 创建 /lib 下的必要文件
# 如果 BusyBox 是静态编译的，这步不需要
# 如果是动态编译：
# cp /usr/arm-linux-gnueabihf/lib/*.so* rootfs/lib/
```

### 3.2 创建设备节点

```bash
cd rootfs/dev
sudo mknod -m 666 console c 5 1
sudo mknod -m 666 null    c 1 3
sudo mknod -m 666 tty1    c 4 1
sudo mknod -m 666 tty2    c 4 2
sudo mknod -m 666 ttyAMA0 c 204 16
cd ../..
```

### 3.3 init 配置

```bash
# /etc/inittab — init 进程的配置文件
cat > rootfs/etc/inittab << 'EOF'
# 系统初始化
::sysinit:/etc/init.d/rcS

# 在串口控制台启动 shell（自动登录 root）
console::respawn:-/bin/sh

# Ctrl+Alt+Del 重启
::ctrlaltdel:/sbin/reboot

# 关机时卸载文件系统
::shutdown:/bin/umount -a -r
EOF

# /etc/init.d/rcS — 系统初始化脚本
mkdir -p rootfs/etc/init.d
cat > rootfs/etc/init.d/rcS << 'EOF'
#!/bin/sh
echo "=== Starting init ==="

# 挂载虚拟文件系统
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp

# 创建设备节点（BusyBox 的 mdev 会做这个）
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s

echo "=== Init done, starting shell ==="
EOF
chmod +x rootfs/etc/init.d/rcS

# /etc/fstab — 文件系统挂载表
cat > rootfs/etc/fstab << 'EOF'
proc    /proc   proc    defaults    0   0
sysfs   /sys    sysfs   defaults    0   0
tmpfs   /tmp    tmpfs   defaults    0   0
EOF
```

### 3.4 制作镜像文件

```bash
cd ~/embedded-linux-lab/rootfs

# 创建一个 64MB 的空白文件
dd if=/dev/zero of=rootfs.img bs=1M count=64

# 格式化为 ext4
mkfs.ext4 rootfs.img

# 挂载并复制
mkdir -p /tmp/mnt
sudo mount -o loop rootfs.img /tmp/mnt
sudo cp -a rootfs/* /tmp/mnt/
sudo umount /tmp/mnt
```

---

## Step 4：启动！

### 4.1 启动脚本

```bash
cat > ~/embedded-linux-lab/scripts/qemu-start.sh << 'EOF'
#!/bin/bash

KERNEL_DIR=../kernel/linux-5.15.180
ROOTFS_DIR=../rootfs

qemu-system-arm \
    -M vexpress-a9 \
    -m 256M \
    -kernel ${KERNEL_DIR}/arch/arm/boot/zImage \
    -dtb ${KERNEL_DIR}/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb \
    -sd ${ROOTFS_DIR}/rootfs.img \
    -append "console=ttyAMA0,115200 root=/dev/mmcblk0 rw rootwait" \
    -nographic
EOF

chmod +x ~/embedded-linux-lab/scripts/qemu-start.sh
```

### 4.2 运行

```bash
cd ~/embedded-linux-lab/scripts
./qemu-start.sh
```

### 4.3 你应该看到

```
Booting Linux on physical CPU 0x0
Linux version 5.15.180 (user@ubuntu) ...
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
...
VFS: Mounted root (ext4 filesystem) on device 179:0.
Freeing unused kernel memory: 1024K
=== Starting init ===
=== Init done, starting shell ===

/ # ls
bin      etc      linuxrc  proc     sbin     usr
dev      lib      mnt      root     sys      var

/ # cat /proc/cpuinfo
processor       : 0
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 1024.00
Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpd32
...

/ # uname -a
Linux (none) 5.15.180 #1 SMP ...

/ # echo "Hello from my first embedded Linux!"
Hello from my first embedded Linux!
```

### 4.4 关闭

```bash
# 在 QEMU 的 shell 中：
/ # poweroff

# 或直接 Ctrl+A 然后按 X（退出 QEMU）
```

---

## 🔍 理解这一步做了什么

| 你做的 | 对应的概念 |
|--------|-----------|
| `make vexpress_defconfig` | 内核配置——选择要编译哪些驱动/功能 |
| `make -j$(nproc)` | 交叉编译——在 x86 上生成 ARM 指令 |
| 设备树 `.dtb` | 硬件描述——告诉内核板子有什么外设 |
| BusyBox | 根文件系统——提供 `ls`/`cat`/`sh` 等基本命令 |
| `/etc/inittab` | init 进程——内核启动后执行的第一个用户程序 |
| `-M vexpress-a9` | QEMU 的机器模型——模拟 ARM 开发板 |
| `-append "console=ttyAMA0..."` | 内核启动参数——指定串口控制台和根设备 |

---

## 🐞 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| `Kernel panic - not syncing: VFS: Unable to mount root fs` | 根文件系统挂载失败 | 检查 `root=` 参数和设备树 |
| 启动后没有 shell | inittab 配置不对 | 检查 `console::respawn:-/bin/sh` |
| `-sh: xxx: not found` | BusyBox 动态链接但没复制 lib | 复制 `.so` 或静态编译 |
| `qemu-system-arm: command not found` | QEMU 未安装 | `sudo apt install qemu-system-arm` |

---

## ⏭️ 下一步

你已经有了一个能启动的嵌入式 Linux 系统。但手工构建太繁琐了——接下来用 **Buildroot** 自动化这个流程。

➡️ [Lab 2: Buildroot 自动化构建](./lab02-buildroot.md)
