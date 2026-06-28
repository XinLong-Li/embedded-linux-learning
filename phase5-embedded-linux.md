# 阶段五：嵌入式 Linux 实战

> **时间：** 8-12 周 | **难度：** ⭐⭐⭐⭐⭐ | **目标：** 从零构建一个嵌入式 Linux 系统，能裁剪移植到开发板

---

## 📌 学习目标

- 理解嵌入式 Linux 系统启动全流程
- 掌握交叉编译工具链的构建与使用
- 掌握 U-Boot 的配置与移植
- 掌握 Linux 内核的裁剪与配置
- 掌握根文件系统的构建（Buildroot / Yocto）
- 能移植一个完整的嵌入式 Linux 到开发板

---

## 🛠️ 硬件准备

**推荐开发板（选其一）：**

| 开发板 | SoC | 特点 | 推荐度 |
|--------|-----|------|--------|
| STM32MP157-DK2 | STM32MP157 (Cortex-A7×2 + M4) | 资料全、ST官方支持好 | ⭐⭐⭐⭐⭐ |
| i.MX6ULL (正点原子/野火) | i.MX6ULL (Cortex-A7) | 中文教程多、配套视频 | ⭐⭐⭐⭐⭐ |
| Raspberry Pi 4/5 | BCM2711/2712 (Cortex-A72/A76) | 社区大、入门友好 | ⭐⭐⭐⭐ |
| BeagleBone Black | AM3358 (Cortex-A8) | 经典、开源硬件 | ⭐⭐⭐ |

> **省钱建议：** 先用 QEMU 模拟 ARM 学习大部分内容，需要操作真实硬件时再买板子。

---

## 📖 第1周：嵌入式 Linux 启动全貌

### Day 1-3：启动流程总览
- [ ] 理解完整启动链路：
  ```
  ROM Boot → U-Boot SPL → U-Boot → Linux Kernel → init → RootFS
  ```
- [ ] 详细解释每一步：
  1. **ROM Boot**：芯片内置，加载 SPL 到 SRAM
  2. **U-Boot SPL**：初始化 DDR，加载 U-Boot 到 DRAM
  3. **U-Boot**：加载内核和设备树，传递启动参数
  4. **Linux Kernel**：初始化硬件，挂载根文件系统
  5. **init**：启动用户空间 init 进程（busybox/systemd）
  6. **RootFS**：完整的文件系统

### Day 4-5：交叉编译工具链
- [ ] 什么是交叉编译
- [ ] 三种获取方式：
  1. **预编译工具链**（推荐入门）：
     - ARM: `arm-linux-gnueabihf-`
     - AArch64: `aarch64-linux-gnu-`
     - `sudo apt install gcc-arm-linux-gnueabihf`
  2. **使用 Buildroot 构建**
  3. **使用 crosstool-NG 构建**（进阶）
- [ ] 测试交叉编译：
  ```bash
  arm-linux-gnueabihf-gcc hello.c -o hello_arm
  file hello_arm  # 确认是 ARM 可执行文件
  ```

### Day 6-7：QEMU 模拟 ARM 环境
- [ ] 用 QEMU 模拟 vexpress-a9 开发板：
  ```bash
  qemu-system-arm -M vexpress-a9 -m 256M \
    -kernel zImage -dtb vexpress-v2p-ca9.dtb \
    -sd rootfs.ext4 -nographic
  ```
- [ ] 理解 QEMU 中的设备模拟
- [ ] 用 QEMU 做后续实验（省钱、方便调试）

### 📝 第1周练习
1. 用交叉编译器编译一个 C 程序，用 `file` 和 `readelf` 确认架构
2. 在 QEMU 中启动 ARM Linux，运行自己编译的程序
3. 画出嵌入式 Linux 启动时序图

---

## 📖 第2-3周：U-Boot 深入

### Day 8-10：U-Boot 基础
- [ ] U-Boot 是什么，为什么需要它
- [ ] 获取 U-Boot 源码：
  ```bash
  git clone https://source.denx.de/u-boot/u-boot.git
  ```
- [ ] U-Boot 目录结构
- [ ] 配置 U-Boot（以开发板为例）：
  ```bash
  make <board>_defconfig
  make menuconfig
  ```
- [ ] 编译 U-Boot：
  ```bash
  make CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
  # 产出：u-boot.bin, u-boot.img, u-boot-spl.bin
  ```

### Day 11-14：U-Boot 命令与环境
- [ ] U-Boot 常用命令：
  ```
  printenv       # 打印环境变量
  setenv         # 设置环境变量
  saveenv        # 保存环境变量
  mmc / fatload  # 从 SD 卡加载
  tftp / nfs     # 网络加载
  bootm / bootz  # 启动内核
  md / mw        # 内存读写（调试用）
  ```
- [ ] bootargs 详解：
  ```
  setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rw rootwait
  ```
- [ ] 启动脚本（bootcmd）的编写
- [ ] FIT Image（Flattened Image Tree）

### Day 15-17：U-Boot 移植
- [ ] 理解 U-Boot 的板级支持（board support）
- [ ] Kconfig 中添加新板子
- [ ] 设备树在 U-Boot 中的使用
- [ ] SPL 的作用与配置
- [ ] 调试 U-Boot（LED/串口）

### 📝 第2-3周练习
1. 给虚拟/真实开发板编译 U-Boot
2. 自定义 U-Boot 启动画面和倒计时
3. 编写 bootcmd 脚本（从 SD 卡加载内核）
4. （可选）尝试为一块开发板添加 U-Boot 支持

---

## 📖 第4-5周：Linux 内核移植与裁剪

### Day 18-21：内核配置与裁剪 ⭐ 核心技能
- [ ] ARM 内核配置：
  ```bash
  make ARCH=arm multi_v7_defconfig  # 从通用配置开始
  make ARCH=arm menuconfig           # 精细裁剪
  ```
- [ ] 裁剪策略：
  - [ ] 去掉不需要的架构支持（只留 ARM）
  - [ ] 去掉不需要的文件系统（只留 ext4、tmpfs、proc、sysfs）
  - [ ] 去掉不需要的网络协议
  - [ ] 去掉不需要的驱动
  - [ ] 去除调试选项
  - [ ] 配置合适的调度器和抢占模型
- [ ] 编译 ARM 内核：
  ```bash
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
  ```
- [ ] 产物分析：`zImage` vs `uImage` vs `Image`
- [ ] 了解内核压缩/解压过程

### Day 22-24：设备树定制
- [ ] 为你的开发板编写/修改设备树：
  - 内存配置
  - 串口配置
  - 存储设备（MMC/NAND）
  - 网络接口
  - 自定义外设
- [ ] 编译设备树：
  ```bash
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
  ```
- [ ] 设备树覆盖 (Device Tree Overlay)
- [ ] 调试设备树问题

### Day 25-28：内核启动调试
- [ ] 理解内核启动日志（逐行分析）
- [ ] 内核启动参数 (bootargs)
- [ ] 常见启动问题排查：
  - 内核 panic（""Unable to mount rootfs""）
  - 设备树不匹配
  - 驱动 probe 失败
- [ ] `earlyprintk` / `earlycon` — 早期串口输出
- [ ] initramfs 作为过渡根文件系统

### 📝 第4-5周练习
1. 裁剪一个 ARM 内核，使镜像 < 3MB
2. 为自己（虚拟或真实）开发板编写 DTS
3. 记录内核启动日志，逐行解释
4. 故意制造一个启动失败，分析并修复

---

## 📖 第6-8周：根文件系统构建

### Day 29-32：BusyBox 构建最小 RootFS ⭐
- [ ] 什么是 BusyBox（嵌入式 Linux 的瑞士军刀）
- [ ] 配置与编译：
  ```bash
  git clone https://git.busybox.net/busybox
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
  # 选择静态链接
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
  make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- install
  ```
- [ ] 构建完整文件系统目录：
  ```bash
  mkdir -p rootfs/{bin,sbin,etc,proc,sys,dev,lib,usr,var,tmp,root}
  ```
- [ ] 复制 BusyBox 到 rootfs
- [ ] 复制 C 运行时库（libc）
- [ ] 创建设备节点：
  ```bash
  sudo mknod rootfs/dev/console c 5 1
  sudo mknod rootfs/dev/null c 1 3
  ```
- [ ] /etc/inittab 配置 init 行为
- [ ] /etc/fstab 配置挂载点
- [ ] /etc/init.d/rcS 启动脚本
- [ ] 制作文件系统镜像：
  ```bash
  dd if=/dev/zero of=rootfs.ext4 bs=1M count=128
  mkfs.ext4 rootfs.ext4
  sudo mount rootfs.ext4 /mnt
  sudo cp -a rootfs/* /mnt/
  sudo umount /mnt
  ```

### Day 33-36：Buildroot — 自动化构建
- [ ] Buildroot 是什么（一站式嵌入式 Linux 构建系统）
- [ ] 基本使用：
  ```bash
  git clone https://git.buildroot.net/buildroot
  make <board>_defconfig
  make menuconfig
  # 选择工具链、BusyBox、软件包
  make -j$(nproc)
  ```
- [ ] Buildroot 产出物：
  - 交叉编译工具链
  - 内核镜像
  - 根文件系统（各种格式）
  - U-Boot
- [ ] 自定义 Buildroot 配置（添加自己的包）
- [ ] `.config` 与 `defconfig` 的关系

### Day 37-38：Yocto 简介（了解）
- [ ] Yocto 是什么（企业级构建系统）
- [ ] Yocto 核心概念（layer, recipe, bitbake）
- [ ] 与 Buildroot 的对比与选择：
  - Buildroot：小团队、简单需求
  - Yocto：大团队、需要包管理、长期维护
- [ ] 快速体验 Yocto（编译一个最小镜像）

### Day 39-42：启动优化与安全
- [ ] 启动时间分析（`initcall_debug`）
- [ ] 内核启动优化技巧
- [ ] 快速启动方案（initramfs、ubifs）
- [ ] 了解 Secure Boot 和 TEE (OP-TEE)
- [ ] 文件系统加密（dm-crypt）

### 📝 第6-8周练习
1. 手工构建一个 < 10MB 的根文件系统
2. 用 Buildroot 为你的开发板生成完整镜像
3. 对比 Buildroot 和手工构建的区别
4. 写出启动全流程脚本（自动化烧录）

---

## 📖 第9-10周：系统移植与硬件调试

### Day 43-46：烧录与启动
- [ ] SD 卡分区与格式化
- [ ] 烧录流程：
  ```bash
  # SD 卡分区
  sudo fdisk /dev/sdb
  # 分区1：FAT32（存放 U-Boot、内核、设备树）
  # 分区2：ext4（根文件系统）
  # 烧录 U-Boot（通常需要特殊位置）
  sudo dd if=u-boot.imx of=/dev/sdb bs=512 seek=2
  ```
- [ ] 串口连接与调试（minicom/picocom）
- [ ] TFTP/NFS 网络启动（开发阶段推荐）

### Day 47-50：GPIO、I2C、SPI 实战
- [ ] GPIO 控制（通过 sysfs 或 libgpiod）
- [ ] I2C 设备通信（通过 i2c-tools）
- [ ] SPI 设备通信（通过 spidev）
- [ ] 编写一个用户空间的硬件控制程序

### Day 51-54：综合项目

**项目：从零构建你的嵌入式 Linux 系统**
1. 交叉编译工具链准备
2. 编译 U-Boot
3. 配置并裁剪内核（< 4MB）
4. 用 BusyBox 构建根文件系统
5. 设备树定制
6. SD 卡烧录
7. 系统启动成功，显示登录提示
8. 写一个开机自启动的应用程序（如 LED 闪烁）

### 📝 最终效果

```
U-Boot 2024.07 (Jan 01 2025 - 12:00:00)

Board: My Embedded Board
DRAM:  512 MiB
MMC:   mmc@2190000: 0
Loading from SD card...
Kernel image @ 0x80800000
Device tree @ 0x83000000
## Booting kernel...
Starting kernel...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 6.6.0-myboard (user@host)
...
[    1.234567] Freeing unused kernel memory: 1024K
Starting init: /sbin/init
Welcome to My Embedded Linux!
myboard login: root
# uname -a
Linux myboard 6.6.0-myboard #1 SMP armv7l
# cat /proc/cpuinfo
...
#
```

---

## ✅ 阶段五检查清单

- [ ] 能解释嵌入式 Linux 从加电到登录的完整流程
- [ ] 能使用交叉编译工具链编译 ARM/AArch64 程序
- [ ] 能配置和编译 U-Boot
- [ ] 能裁剪 Linux 内核（理解 menuconfig 中关键选项）
- [ ] 能手写/修改设备树
- [ ] 能手工构建 BusyBox 根文件系统
- [ ] 会使用 Buildroot 自动化构建
- [ ] **成品：开发板启动成功，进入 shell 环境**

---

## 📚 阶段五推荐资源

- **必读：** 《Mastering Embedded Linux Programming》(Chris Simmonds) — 嵌入式 Linux 圣经
- **必读：** 《Embedded Linux System Design and Development》
- **参考：** 《Building Embedded Linux Systems》(Karim Yaghmour)
- **参考：** [Bootlin Embedded Linux Training](https://bootlin.com/training/embedded-linux/) — 免费培训资料
- **参考：** [eLinux.org](https://elinux.org/) — 嵌入式 Linux Wiki
- **参考：** [Buildroot Manual](https://buildroot.org/downloads/manual/manual.html)
- **参考：** [U-Boot Documentation](https://u-boot.readthedocs.io/)
- **工具：** `tio` / `minicom` / `picocom` — 串口终端
