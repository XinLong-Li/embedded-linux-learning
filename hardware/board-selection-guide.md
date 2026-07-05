# 🧭 学习板选型指南：用你手头的硬件开始学习

> **目标：** 评估你已有的开发板是否适合嵌入式 Linux 学习，最大化利用现有资源
>
> **核心原则：** 不是非要买推荐板才能学 —— 先看懂手头板子的"优劣势"，扬长避短

---

## 📋 快速自测：你的板子适合学习吗？

从嵌入式 Linux **系统学习**的角度，一块理想的开发板应该满足：

| 理想条件 | 为什么重要 | 学习阶段 |
|---------|-----------|---------|
| ✅ 支持 Mainline 内核 | 能自己编译、裁剪、调试内核 | Phase 4 |
| ✅ 标准 U-Boot 启动 | 理解通用 bootloader 工作流 | Phase 5 |
| ✅ `qemu_*_defconfig` 有对应 | Buildroot/Yocto 一键构建 | Phase 5 |
| ✅ 公开 SoC 数据手册 | 深入理解外设寄存器 | Phase 4~5 |
| ✅ 板载调试器 (JTAG/SWD) | 内核调试、单步跟踪 | Phase 4 |
| ✅ 异构多核 (A+M) | OpenAMP / RPMsg 实战 | Lab 6 |
| ✅ GPIO / I2C / SPI 引出 | 外设驱动开发 | Phase 5 |

现实中几乎没有板子能全部满足（推荐板 STM32MP157F-DK2 除外）。关键是**知道缺了什么，用其他方式补上**。

---

## 🍓 案例一：Raspberry Pi 3 Model B V1.2

> **结论：适合！** 能完成本仓库 80% 的 Lab 实验，是手头三块板里最合适的学习板。

### 规格速览

| 项目 | 详情 |
|------|------|
| **SoC** | Broadcom BCM2837 |
| **CPU** | 4×Cortex-A53 @ 1.2GHz (ARMv8 64-bit) |
| **RAM** | 1GB LPDDR2 |
| **存储** | MicroSD 卡槽 |
| **网络** | 10/100 以太网 + WiFi + BLE |
| **GPIO** | 40-pin (I2C, SPI, UART, GPIO) |
| **显示** | HDMI, MIPI DSI |
| **摄像头** | MIPI CSI |

### 学习匹配度评估

| 学习环节 | RPi 3B 表现 | 说明 |
|---------|:---:|------|
| **内核编译** | ✅ 优秀 | `bcmrpi3_defconfig` 直接编译，Mainline 支持良好 |
| **Buildroot** | ✅ 优秀 | `raspberrypi3_defconfig` 一键构建完整系统 |
| **启动流程** | 🟡 一般 | GPU 先启动（闭源固件），非标准 TF-A→U-Boot 流程 |
| **U-Boot** | ✅ 支持 | 有上游 U-Boot 支持，可替代默认启动 |
| **设备驱动** | ✅ 优秀 | 40-pin GPIO 满足 I2C/SPI 驱动实验 |
| **SoC 文档** | 🔴 不足 | BCM 不公开完整数据手册，深入外设受限 |
| **调试器** | 🔴 不足 | 无板载 JTAG，需外接调试器 |
| **异构多核** | 🔴 不支持 | 无 M 核，Lab 6 无法完成 |
| **成本** | 🟢 极低 | 二手 ~¥150 |
| **社区教程** | 🟢 海量 | 网上几乎任何问题都能搜到 |

### 核心优势：生态碾压级

RPi 3B 最大的优势不在于硬件本身，而在于**生态**：

```bash
# Buildroot 内置 Pi 3 配置，一行命令出完整系统
make raspberrypi3_defconfig
make
# 把 output/images/sdcard.img 写入 TF 卡 → 插入 → 上电 → 启动

# 内核编译也有标准流程
make ARCH=arm64 bcmrpi3_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

几乎所有嵌入式 Linux 工具链、教程、书籍都会以树莓派作为示例平台。

### 主要硬伤：启动流程不标准

```
标准嵌入式 Linux 启动流程:              树莓派的实际启动流程:

BootROM                                 BootROM (闭源)
  ↓                                        ↓
TF-A (ARM Trusted Firmware)             GPU 固件 (闭源, bootcode.bin)
  ↓                                        ↓
U-Boot (开源 bootloader)                GPU 读 config.txt, 初始化硬件
  ↓                                        ↓
Linux Kernel                            加载 ARM 固件 → 启动 ARM 核
                                             ↓
                                        U-Boot / Kernel 被加载
```

这导致你学不到通用的 bootloader 工作流 —— 而这是嵌入式 Linux 面试和实际工作中的核心知识点。不过可以用 U-Boot 覆盖默认启动方式，部分弥补。

> 💡 **补救方案：** 给 RPi 3B 刷入上游 U-Boot，体验标准启动流程。详见 [U-Boot for Raspberry Pi](https://u-boot.readthedocs.io/en/latest/board/broadcom/raspberrypi.html)。

### 使用建议

```
RPi 3B 最适合:
├── Lab 2: Buildroot 一键构建系统      ✅ 原生支持
├── Lab 3: 内核模块 Hello World        ✅ 编译 → scp → insmod
├── Lab 4: 字符设备驱动               ✅ GPIO/I2C/SPI 外设实操
├── Lab 5: 真实板子启动               ✅ TF 卡烧写、uboot 调试
└── Lab 6: OpenAMP 异构多核           ❌ 没 M 核，做不了

替代方案: Lab 6 用 QEMU 模拟 xlnx-zcu102（Cortex-A53 + Cortex-R5）来学习 OpenAMP
```

---

## 🚀 案例二：NVIDIA Jetson TX2

> **结论：不太适合系统层学习，但 Phase 6 AI 项目时是王牌。**

### 规格速览

| 项目 | 详情 |
|------|------|
| **SoC** | NVIDIA Tegra X2 (Parker) |
| **CPU** | 2×Denver2 + 4×Cortex-A57 (ARMv8 64-bit) |
| **GPU** | Pascal 架构, 256 CUDA 核心 |
| **RAM** | 8GB LPDDR4 (128-bit) |
| **存储** | 32GB eMMC + SD 卡槽 |
| **网络** | 千兆以太网 + WiFi |
| **视频** | HDMI 2.0, 4K 编解码 |
| **扩展** | GPIO, I2C, SPI, UART, PCIe |

### 学习匹配度评估

| 学习环节 | Jetson TX2 表现 | 说明 |
|---------|:---:|------|
| **内核编译** | 🟡 一般 | 需用 NVIDIA L4T 内核，不能直接用 Mainline |
| **启动流程** | 🔴 很差 | 私有 cboot + 安全启动，完全非标准 |
| **rootfs 构建** | 🔴 很差 | JetPack SDK 一键刷写，屏蔽构建细节 |
| **Buildroot/Yocto** | 🔴 不支持 | 没有官方配置，需大量移植工作 |
| **设备驱动** | 🟢 可行 | 有 GPIO/I2C/SPI，但文档少 |
| **SoC 文档** | 🔴 闭源 | NVIDIA 不公开 Tegra 完整手册 |
| **GPU/CUDA** | 🟢 独有优势 | 边缘 AI 推理，其他学习板做不到 |
| **成本** | 🟡 中等 | 二手 ~¥500 |

### 核心问题：JetPack SDK 屏蔽了学习关键环节

NVIDIA 提供的 JetPack SDK 是一套"全家桶" —— 内核、设备树、rootfs、驱动全部预构建，一个脚本刷入：

```
学习嵌入式 Linux 需要的流程:           Jetson TX2 给你的实际体验:

TF-A (ARM Trusted Firmware)  →    NVIDIA 私有安全固件（闭源, 不可修改）
U-Boot 配置与编译              →    cboot（NVIDIA 私有, 不可替换）  
手动配置内核 .config            →    预配置的 L4T 内核, 直接刷写
手动裁剪 rootfs (BusyBox)       →    预装的 Ubuntu 桌面系统
自制 init 脚本                  →    systemd 直接接管
```

你在 Jetson 上做实验时，大部分时间是在 **Ubuntu 系统里写应用层代码**，而非做嵌入式 Linux 系统级构建。

### 独有优势：边缘 AI 的王牌

但 Jetson TX2 有一个其他学习板无法替代的优势 —— **CUDA GPU**：

```bash
# 在 Jetson TX2 上运行 AI 推理（其他学习板做不到）
import tensorrt as trt
# 256 CUDA 核心加速 YOLO/ResNet 等模型推理

# ROS + CUDA 机器人 SLAM
roslaunch orb_slam2 jetson_tx2.launch
```

### 使用建议

```
Jetson TX2 最适合:
├── Phase 6 / Lab 7: AI 边缘计算综合项目    ✅ GPU 推理 + ROS
├── 多路视频 AI 分析                       ✅ 硬件编码器 + CUDA
├── 机器人 SLAM / 自主导航                  ✅ ROS + GPU 加速
├── Lab 1~4: 内核编译、驱动开发             ❌ 被 SDK 屏蔽, 用 QEMU 代替
└── Lab 6: OpenAMP 异构多核                ❌ 做不了

最佳定位: 学习用 QEMU + RPi 3B, 做项目用 Jetson TX2
```

---

## 📊 三块板子的综合对比

| 维度 | QEMU (虚拟) | RPi 3B | Jetson TX2 | STM32MP157F-DK2 (推荐) |
|------|:---:|:---:|:---:|:---:|
| **成本** | 免费 | ~¥150 | ~¥500 | ~¥800-1200 |
| **内核编译** | ✅ 全架构 | ✅ Mainline | 🟡 L4T 定制 | ✅ Mainline |
| **启动流程** | ✅ 标准 | 🟡 GPU 先启 | 🔴 私有 | ✅ TF-A→U-Boot |
| **Buildroot** | ✅ 多板卡 | ✅ 官方配置 | 🔴 不支持 | ✅ 官方配置 |
| **设备驱动** | 🟡 有限模拟 | ✅ GPIO 40-pin | 🟡 可用 | ✅ 丰富外设 |
| **SoC 文档** | N/A | 🔴 BCM 闭源 | 🔴 NVIDIA 闭源 | ✅ ST 公开 |
| **板载调试** | GDB 内建 | 🔴 需外接 | 🔴 需外接 | ✅ ST-LINK |
| **异构多核** | 🟡 部分支持 | 🔴 无 | 🔴 无 | ✅ A7+M4 |
| **AI/GPU** | ❌ | ❌ | ✅ CUDA | ❌ |
| **社区教程** | 🟢 通用 | 🟢 海量 | 🟢 AI 方向 | 🟢 ST Wiki |

---

## 🗺️ 推荐学习策略：三块板子各司其职

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  学习阶段                主力工具           验证/项目         │
│                                                             │
│  Phase 1~3              🐧 QEMU            仅需 PC           │
│  (Linux/C/OS 基础)                                        │
│                                                             │
│  Phase 4                🐧 QEMU            🍓 RPi 3B 验证    │
│  (内核 + 驱动)           编译调试主力         GPIO 外设实战     │
│                                                             │
│  Phase 5                🐧 QEMU            🍓 RPi 3B 真机    │
│  (嵌入式 Linux 实战)     Buildroot 快速迭代   烧录启动验证      │
│                                                             │
│  Lab 6 (异构多核)        🐧 QEMU 模拟        RPi/TX2 无此能力 │
│                         xlnx-zcu102                        │
│                                                             │
│  Phase 6 / Lab 7        🍓 RPi 3B          🚀 Jetson TX2    │
│  (综合项目)              通用嵌入式项目       AI 边缘计算项目    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> 💡 **一句话总结：** QEMU 学原理，RPi 3B 跑流程，Jetson TX2 做 AI 项目。三块板子互补，不用再额外买硬件就能覆盖完整学习路线。等学到 Lab 6 异构多核时，先用 QEMU 模拟跑通，再决定是否入手 STM32MP157F-DK2。

---

## ⚡ 如果你要新买一块板

如果你未来打算购买一块专门用于学习的开发板：

| 预算 | 推荐 | 理由 |
|------|------|------|
| **¥800-1200** | ⭐ **STM32MP157F-DK2** | 最完整的嵌入式 Linux 学习体验，TF-A→U-Boot→Linux 全暴露，板载 ST-LINK |
| **¥300-500** | Raspberry Pi 4/5 | 如果已有 RPi 3B 就不用换；买 4/5 内存大一点，跑编译更快 |
| **¥200-400** | BeagleBone Black | TI AM3358, 开源程度高, 有 PRU, 也是经典学习板 |

> ⚠️ **现在不用买。** 你手头的 RPi 3B 足够开始 Lab 1~5 的绝大部分内容。先用起来，学到瓶颈了再考虑添置。

---

## 📚 相关文档

- 📖 [硬件平台全景指南](./hardware-platforms.md) — 全球主流嵌入式 Linux 芯片平台概览
- 🐧 [QEMU 完全指南](../reference/qemu-guide.md) — 零成本虚拟开发板
- 🔬 [实验手册](../labs/README.md) — 7 个动手实验
- 🧪 [Lab 1: QEMU 最小系统](../labs/lab01-qemu-minimal.md)
- 🧪 [Lab 5: 真实开发板](../labs/lab05-real-board.md)
