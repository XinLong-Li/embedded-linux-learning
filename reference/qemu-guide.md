# 🐧 QEMU 完全指南：嵌入式 Linux 的虚拟实验室

> **目标:** 全面理解 QEMU 的原理、架构和在嵌入式 Linux 开发中的使用方式
> **适用阶段:** Phase 1 ~ Phase 6 全阶段
> **前置知识:** 基本的 Linux 命令行操作

---

## 📖 什么是 QEMU？

**QEMU**（Quick Emulator）是一个开源的通用机器模拟器和虚拟化器。它可以在一种架构的宿主机上运行另一种架构的程序（模拟），也可以利用硬件虚拟化技术（如 KVM）以接近原生性能运行虚拟机。

在嵌入式 Linux 学习中，QEMU 扮演着至关重要的角色——**它让你无需购买任何硬件开发板，就能在 PC 上运行和调试完整的嵌入式 Linux 系统**。

> 💡 **核心价值：** 零成本入门，无限"开发板"——一台 PC + QEMU = 一块可编程的 ARM 开发板。

---

## 🏗️ QEMU 架构概览

```
┌─────────────────────────────────────────────────────┐
│                    用户空间应用程序                      │
│           (BusyBox, 自定义应用, 测试程序)                 │
├─────────────────────────────────────────────────────┤
│                    Linux Kernel                       │
│         (arch/arm, drivers, filesystems...)           │
├──────────────────────────┬──────────────────────────┤
│                          │                          │
│    System Mode (系统模式)  │  User Mode (用户模式)      │
│    ────────────────────  │  ────────────────────     │
│    • 模拟完整机器          │  • 只模拟 CPU + 系统调用    │
│    • CPU + 内存 + 外设     │  • 直接使用宿主机内核       │
│    • 运行完整 OS           │  • 运行单个交叉编译程序      │
│    • qemu-system-*        │  • qemu-* (如 qemu-arm)   │
│                          │                          │
├──────────────────────────┴──────────────────────────┤
│              TCG (Tiny Code Generator)                │
│         动态二进制翻译：Guest ISA → Host ISA             │
├─────────────────────────────────────────────────────┤
│                   宿主机操作系统                         │
│              (macOS / Linux / Windows)                │
├─────────────────────────────────────────────────────┤
│             硬件加速 (可选): KVM / HAXM / HVF            │
└─────────────────────────────────────────────────────┘
```

### 两种工作模式

| 特性 | System Mode (系统模式) | User Mode (用户模式) |
|------|----------------------|---------------------|
| **命令** | `qemu-system-<arch>` | `qemu-<arch>` |
| **模拟范围** | 完整机器 (CPU/内存/外设/总线) | 仅 CPU + Linux 系统调用 |
| **运行内容** | 完整的 Guest OS + 应用 | 单个交叉编译的可执行文件 |
| **内核** | Guest 自带内核 | 使用宿主机内核 (通过 binfmt_misc) |
| **文件系统** | 独立的 rootfs 镜像 | 宿主机文件系统 (chroot 语义) |
| **启动速度** | 较慢 (需完整引导) | 极快 (瞬间启动) |
| **适用场景** | 嵌入式系统开发、内核调试 | 交叉编译测试、CI/CD |
| **本仓库用法** | 🎯 Lab 全部实验 | 快速测试交叉编译工具链 |

---

## 🎯 QEMU 支持的架构

QEMU 支持超过 **20 种 CPU 架构**，以下是嵌入式开发中最常用的：

| 架构 | System 命令 | User 命令 | 典型开发板 | 本仓库使用 |
|------|-----------|----------|-----------|----------|
| **ARM (32-bit)** | `qemu-system-arm` | `qemu-arm` | vexpress-a9, versatilepb | ✅ Lab 1~7 |
| **AArch64 (64-bit)** | `qemu-system-aarch64` | `qemu-aarch64` | virt, sbsa-ref | ✅ 部分实验 |
| **RISC-V** | `qemu-system-riscv64` | `qemu-riscv64` | virt, sifive_u | 📅 计划中 |
| **MicroBlaze** | `qemu-system-microblaze` | `qemu-microblaze` | petalogix-s3adsp1800 | — |
| **MIPS** | `qemu-system-mips` | `qemu-mips` | malta | — |
| **x86/x86_64** | `qemu-system-x86_64` | `qemu-x86_64` | pc, q35 | — |

> 💡 **为什么选 ARM？** ARM 是嵌入式领域的主流架构 (手机、IoT、汽车电子)，学习 ARM 嵌入式 Linux 具有最高的性价比和就业价值。

---

## 📦 安装 QEMU

### macOS (本仓库主要开发环境)

```bash
# 安装完整 QEMU (含所有架构支持)
brew install qemu

# 验证安装
qemu-system-arm --version
qemu-system-aarch64 --version
```

### Ubuntu / Debian

```bash
# 安装 ARM 相关 QEMU
sudo apt update
sudo apt install qemu-system-arm qemu-system-aarch64 qemu-user-static

# 验证
qemu-system-arm --version
```

### 从源码编译 (需要定制功能时)

```bash
git clone https://gitlab.com/qemu-project/qemu.git
cd qemu
./configure --target-list=arm-softmmu,aarch64-softmmu --enable-kvm
make -j$(nproc)
sudo make install
```

---

## 🔧 QEMU System Mode 详解

System Mode 是本仓库实验的核心使用方式。它模拟一台完整的 ARM 开发板。

### 核心组件

```
qemu-system-arm 启动需要:
┌──────────────────────────────────────────────┐
│  1. Kernel Image                              │
│     ├── zImage (压缩内核)                      │
│     ├── uImage (U-Boot 格式)                   │
│     └── vmlinux (ELF 格式, 用于调试)           │
│                                               │
│  2. Root Filesystem (rootfs)                  │
│     ├── initramfs (内嵌到内核)                  │
│     ├── SD 卡镜像 (.img/.ext4)                 │
│     └── NFS (网络文件系统)                      │
│                                               │
│  3. Device Tree (可选但推荐)                   │
│     └── .dtb 文件描述硬件拓扑                   │
│                                               │
│  4. Bootloader (可选)                         │
│     └── U-Boot (真实开发板流程)                 │
└──────────────────────────────────────────────┘
```

### 常用命令行参数速查表

| 参数 | 说明 | 示例 |
|------|------|------|
| `-M` | 指定模拟的机器型号 | `-M vexpress-a9` |
| `-cpu` | 指定 CPU 型号 | `-cpu cortex-a9` |
| `-m` | 分配内存大小 | `-m 256M` |
| `-kernel` | 指定内核镜像 | `-kernel zImage` |
| `-dtb` | 指定设备树文件 | `-dtb vexpress-v2p-ca9.dtb` |
| `-drive` / `-sd` | 挂载磁盘/SD 镜像 | `-sd rootfs.ext4` |
| `-append` | 内核启动参数 | `-append "root=/dev/mmcblk0 console=ttyAMA0"` |
| `-nographic` | 无图形窗口, 串口输出到终端 | `-nographic` |
| `-serial` | 重定向串口 | `-serial stdio` |
| `-net` | 网络配置 | `-net nic -net user` |
| `-netdev` + `-device` | 新式网络配置 | `-netdev user,id=n0 -device virtio-net,netdev=n0` |
| `-gdb` | 开启 GDB 调试服务 | `-gdb tcp::1234` |
| `-S` | 启动时暂停, 等待 GDB 连接 | `-S` |
| `-smp` | 模拟多核 CPU | `-smp 2` |

---

## 🧪 实战示例

### 示例 1: 最简启动 (BusyBox + 自制 rootfs)

这是本仓库 Lab 1 的核心流程:

```bash
# 1. 编译内核
make ARCH=arm vexpress_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j$(nproc)

# 2. 制作 initramfs
mkdir -p rootfs/{bin,dev,etc,proc,sys,tmp}
# ... 放入 busybox + init 脚本 ...

# 3. 启动 QEMU
qemu-system-arm \
    -M vexpress-a9 \
    -m 256M \
    -kernel arch/arm/boot/zImage \
    -dtb arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb \
    -initrd rootfs.cpio.gz \
    -append "console=ttyAMA0 rdinit=/init" \
    -nographic
```

### 示例 2: Buildroot 一键生成

```bash
# Buildroot 自动生成 kernel + rootfs + 启动脚本
make qemu_arm_vexpress_defconfig
make
# 自动生成 output/images/start-qemu.sh
./output/images/start-qemu.sh
```

### 示例 3: GDB 调试内核

```bash
# 终端 1: 启动 QEMU, 挂起等待 GDB
qemu-system-arm -M vexpress-a9 -kernel zImage -nographic \
    -gdb tcp::1234 -S

# 终端 2: 连接 GDB
arm-linux-gnueabi-gdb vmlinux
(gdb) target remote :1234
(gdb) b start_kernel
(gdb) c
```

### 示例 4: 用户模式快速测试

```bash
# 无需 rootfs, 直接运行交叉编译的程序
arm-linux-gnueabi-gcc -static hello.c -o hello
qemu-arm ./hello
# 输出: Hello from ARM!
```

---

## 🌐 QEMU 网络配置

嵌入式开发中，网络用于传输文件、远程调试和访问外部服务。

### 三种网络模式

```
┌──────────────────────────────────────────────────────┐
│  1. User Mode (SLIRP) — 最常用, 无需 root 权限         │
│     Guest: 10.0.2.15                                 │
│     Host:   10.0.2.2   (可访问宿主机)                  │
│     DNS:    10.0.2.3   (内置 DNS 代理)                 │
│     宿主机 → Guest: 需端口转发 (hostfwd)               │
│                                                      │
│  2. TAP / Bridge — 需要 root, Guest 获得独立 IP        │
│     Guest ↔ Host ↔ LAN 完全互通                       │
│                                                      │
│  3. Socket — 多个 QEMU 实例间互联                      │
└──────────────────────────────────────────────────────┘
```

### 常用网络配置命令

```bash
# User Mode + 端口转发 (将 Guest 22 端口映射到 Host 2222)
qemu-system-arm ... \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-device,netdev=net0

# 现在可以在宿主机通过 SSH 连接 QEMU
ssh -p 2222 root@localhost
```

---

## 🔄 QEMU 在嵌入式 Linux 工作流中的位置

```
┌─────────────────────────────────────────────────────────┐
│                   嵌入式 Linux 开发流程                     │
│                                                         │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌───────┐ │
│  │ Bootloader│ → │  Kernel   │ → │  Rootfs  │ → │  App  │ │
│  │  U-Boot   │   │  Linux    │   │  BusyBox │   │ 自定义 │ │
│  └──────────┘   └──────────┘   └──────────┘   └───────┘ │
│       │              │              │              │     │
│       ▼              ▼              ▼              ▼     │
│  ┌───────────────────────────────────────────────────┐  │
│  │                 🐧 QEMU 虚拟开发板                    │  │
│  │  • 快速迭代: 编译 → 启动 → 测试 (秒级)               │  │
│  │  • 内核调试: GDB + printk + ftrace                  │  │
│  │  • 无惧"变砖": reset 即可恢复                        │  │
│  └───────────────────────────────────────────────────┘  │
│                         │                               │
│                         ▼ (验证通过后)                    │
│  ┌───────────────────────────────────────────────────┐  │
│  │                 📟 真实硬件开发板                     │  │
│  │  • STM32MP157F-DK2 / Raspberry Pi / ZCU104        │  │
│  │  • 真实外设: GPIO, I2C, SPI, Camera...             │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

> 💡 **最佳实践:** "QEMU 先行, 硬件验证"——在 QEMU 中完成 80% 的内核和应用调试, 最后在真实硬件上验证外设和性能。这大大缩短了开发周期。

---

## 📊 QEMU vs 真实硬件 vs 其他方案

| 维度 | QEMU | 真实开发板 | Docker/容器 |
|------|------|----------|-----------|
| **成本** | 🟢 免费 | 🔴 ¥300~5000+ | 🟢 免费 |
| **架构模拟** | 🟢 多种架构 | 🟡 固定架构 | 🔴 仅宿主机架构 |
| **内核调试** | 🟢 GDB + 无惧崩溃 | 🟡 JTAG/SWD 调试器 | 🔴 共享宿主机内核 |
| **硬件外设** | 🟡 有限模拟 | 🟢 真实外设 | 🔴 无硬件访问 |
| **启动速度** | 🟢 秒级 | 🟡 分钟级 | 🟢 毫秒级 |
| **"变砖" 风险** | 🟢 零风险 | 🔴 可能存在 | 🟢 零风险 |
| **性能准确性** | 🟡 近似 | 🟢 真实 | 🟢 原生性能 |
| **学习启动流程** | 🟢 完整流程 | 🟢 完整流程 | 🔴 跳过启动 |
| **适用阶段** | Phase 1~5 学习 | Phase 5~6 实战 | ❌ 不适合嵌入式 |

> ⚠️ **QEMU 的局限:** QEMU 不能完全替代真实硬件。它无法模拟特定的外设时序、DMA 延迟、中断响应时间等硬件特性。当你的项目涉及真实传感器、电机控制、硬件加速时，必须上真机。

---

## 🛠️ QEMU 与构建系统的集成

### Buildroot

```bash
# Buildroot 内置 20+ QEMU 板级配置
make list-defconfigs | grep qemu

# 常用配置:
# qemu_arm_vexpress_defconfig     — ARM vexpress-a9
# qemu_aarch64_virt_defconfig     — AArch64 virt
# qemu_riscv64_virt_defconfig     — RISC-V virt
```

### Yocto Project

```bash
# Yocto 通过 runqemu 脚本集成
MACHINE=qemuarm64 bitbake core-image-minimal
runqemu qemuarm64 nographic
```

### 手动集成 (本仓库 Lab 1 方式)

```bash
# 完全手动控制每个环节
# 适合深入理解系统组成, 但不适合生产项目
```

---

## 🐛 常见问题与排查

| 现象 | 可能原因 | 解决方法 |
|------|---------|---------|
| `Kernel panic - not syncing: VFS` | rootfs 未正确挂载 | 检查 `-initrd` / `-sd` 参数和 `root=` 内核参数 |
| `qemu-system-arm: command not found` | QEMU 未安装或未在 PATH | `brew install qemu` 或 `apt install qemu-system-arm` |
| 启动后无输出 | 串口配置不匹配 | 确保 `console=ttyAMA0` 与 QEMU `-serial stdio` 对应 |
| `-initrd` 太大 | initramfs 超过默认内存 | 增加 `-m 512M` 或更大内存 |
| 网络不通 | 未配置网络设备 | 添加 `-netdev user,id=n0 -device virtio-net-device,netdev=n0` |
| GDB 连接被拒 | QEMU 未开启 GDB stub | 添加 `-gdb tcp::1234` 参数 |
| macOS 上 KVM 不可用 | macOS 不支持 KVM | 使用 HVF 加速: `-accel hvf` (仅同架构) |

---

## 📚 深入学习资源

### 官方资源
- [QEMU 官方文档](https://www.qemu.org/docs/master/)
- [QEMU ARM 支持的机器列表](https://www.qemu.org/docs/master/system/target-arm.html)
- [QEMU Wiki](https://wiki.qemu.org/)

### 推荐阅读
- ⭐ *Mastering Embedded Linux Programming* — Chapter 4: All About Bootloaders (含 QEMU 实操)
- ⭐ QEMU 源码中的 `docs/system/` 目录

### 本仓库相关文档
- 🧪 [Lab 1: QEMU 最小嵌入式 Linux 系统](../labs/lab01-qemu-minimal.md) — 从零搭建
- 🧪 [Lab 2: Buildroot 构建系统](../labs/lab02-buildroot.md) — QEMU 自动化构建
- 🧪 [Lab 3: 内核模块开发](../labs/lab03-kernel-module.md) — QEMU 中测试驱动
- 🧪 [Lab 4: 字符设备驱动](../labs/lab04-char-driver.md) — QEMU 中调试驱动
- 🧪 [Lab 5: 真实开发板](../labs/lab05-real-board.md) — 从 QEMU 迁移到硬件
- 📖 [环境配置指南](../labs/environment-setup.md) — QEMU 安装步骤

---

> 🎯 **下一步:** 安装了 QEMU 了吗？马上开始 [Lab 1: QEMU 最小嵌入式 Linux 系统](../labs/lab01-qemu-minimal.md)，亲手构建你的第一个虚拟嵌入式 Linux！
