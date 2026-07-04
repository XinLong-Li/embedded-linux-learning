# 🔬 实验手册：做中学嵌入式 Linux

> **原则：** 每个概念都有对应的动手实验。先跑通，再理解。
>
> **路线：** QEMU 模拟器（免费入门）→ 真实开发板（进阶实战）
>
> 📖 **QEMU 还不熟悉？** 先阅读 [QEMU 完全指南](../reference/qemu-guide.md) 全面了解 QEMU 的原理与用法。

---

## 🎯 推荐硬件

### 首推：STM32MP157F-DK2（约 ¥800-1200）

```\
┌─────────────────────────────────────────┐
│          STM32MP157F-DK2                 │
│                                          │
│  SoC: STM32MP157 (Cortex-A7×2 + M4)     │
│  RAM: 512MB DDR3                         │
│  Storage: MicroSD 卡槽                    │
│  Display: 4" TFT 触摸屏 (480×800)        │
│  Network: 千兆以太网                      │
│  USB: USB-C 供电 + USB Host              │
│  Debug: 板载 ST-LINK (JTAG/UART)         │
│  Expand: Arduino ×2 + Raspberry Pi 40pin │
│  Audio: 音频编解码器 + 耳机/扬声器          │
└─────────────────────────────────────────┘
```

**为什么是它？**

| 理由 | 说明 |
|------|------|
| **文档最全** | ST 官方 Wiki (`wiki.st.com`) 从零教起，每一步都有截图 |
| **完整的嵌入式 Linux 体验** | TF-A → U-Boot → Linux 启动链全暴露，不隐藏任何细节 |
| **异构多核实战** | Cortex-A7 (Linux) + Cortex-M4 (RTOS)，OpenAMP 官方示例直接跑 |
| **板载 ST-LINK** | 不需要额外买调试器，JTAG 调试 + 串口一应俱全 |
| **STM32CubeMX 集成** | 引脚配置 + 时钟树 + 设备树一键生成 |
| **工业级基因** | 不是玩具板，学到的东西直接用于产品开发 |

**需要额外购买：**
- MicroSD 卡（16GB+，Class 10）
- USB-C 电源适配器（5V/3A）
- （可选）USB-TTL 串口模块（CH340）

### 备选：树莓派 4/5（约 ¥300-500）

如果预算紧张，树莓派也可以入门，但会"屏蔽"一些嵌入式特有概念（启动流程被封装、没有 M 核）。

---

## 🗺️ 实验路线图

```
Lab 0: 环境搭建           ██░░░░░░░░  1天     安装 Ubuntu + 工具链
Lab 1: QEMU 最小系统      ███░░░░░░░  2-3天   BusyBox + 内核 + 手动构建
Lab 2: Buildroot 自动化    ██░░░░░░░░  1-2天   用 Buildroot 一键生成系统
Lab 3: 内核模块 HelloWorld  ███░░░░░░░  2-3天   第一个内核模块，加载/卸载
Lab 4: 字符设备驱动        █████░░░░░  3-5天   完整的读写驱动 + 测试程序
Lab 5: 用真实板子启动       ████░░░░░░  3-5天   STM32MP157 从零启动
Lab 6: 异构多核通信        ██████░░░░  5-7天   A7 + M4 OpenAMP 通信
Lab 7: 综合项目            █████████░  2-3周   自选项目
```

---

## ⚡ 快速开始（最短路径）

```bash
# 1. 确保你有 Ubuntu 22.04/24.04
lsb_release -a

# 2. 一键安装所有依赖
sudo apt update && sudo apt install -y \
  build-essential gcc g++ gdb make cmake git \
  vim curl wget tree htop \
  qemu-system-arm qemu-system-aarch64 \
  gcc-arm-linux-gnueabihf gcc-aarch64-linux-gnu \
  flex bison libssl-dev libncurses-dev bc rsync \
  device-tree-compiler picocom minicom \
  libelf-dev dwarves cpio unzip

# 3. 进入 Lab 1
```

---

## 📂 目录结构

```
labs/
├── README.md                  ← 本文件
├── environment-setup.md       ← 详细环境配置
├── lab01-qemu-minimal.md      ← Lab 1: QEMU 最小系统
├── lab02-buildroot.md         ← Lab 2: Buildroot 自动化
├── lab03-kernel-module.md     ← Lab 3: 内核模块
├── lab04-char-driver.md       ← Lab 4: 字符设备驱动
├── lab05-real-board.md        ← Lab 5: STM32MP157 真机启动
├── lab06-amp-openamp.md       ← Lab 6: 异构多核通信
└── lab07-final-project/       ← Lab 7: 综合项目
```
