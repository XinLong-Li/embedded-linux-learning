# 🐧 嵌入式 Linux 学习路线图

> **目标：** 从零基础到能够裁剪 Linux 系统并移植到常见嵌入式芯片（ARM Cortex-A 系列等）
>
> **前置基础：** C 语言基础、单片机基础、对操作系统有初步了解
>
> **预计总学时：** 6-12 个月（业余时间学习）

---

## 📋 学习阶段总览

```
阶段一：Linux 基础操作        ████████░░░░░░░░  1 周（快速review）
阶段二：C 语言深度进修        ████████████████  6-8 周  ← 🔥 你从这里开始
阶段三：操作系统核心概念      ████████████████  6-8 周
阶段四：Linux 内核初步        ████████████████  8-10 周
阶段五：嵌入式 Linux 实战     ████████████████  8-12 周
阶段六：综合项目实战          ████████████████  持续进行
```

> 💡 **你的起点：** 鉴于你已有 ROS 开发经验、熟悉 Ubuntu 和命令行操作，**阶段一只需花 1 周快速回顾**（重点看 Makefile/编译工具链和 Shell 脚本），直接进入**阶段二 C 语言深度进修**。

## 🗺️ 路线图

| 阶段 | 内容 | 目标 | 关键产出 |
|------|------|------|----------|
| [阶段一](./phases/phase1-linux-basics.md) | Linux 基础操作 | 熟练使用 Linux 作为日常开发环境 | Shell 脚本工具箱 |
| [阶段二](./phases/phase2-c-deep-dive.md) | C 语言深度进修 | 掌握嵌入式 C 编程核心技能 | 数据结构库 / 小型项目 |
| [阶段三](./phases/phase3-os-concepts.md) | 操作系统核心概念 | 理解 OS 运行原理 | Mini-OS 玩具内核 |
| [阶段四](./phases/phase4-linux-kernel.md) | Linux 内核初步 | 理解内核架构与驱动模型 | 字符设备驱动 |
| [阶段五](./phases/phase5-embedded-linux.md) | 嵌入式 Linux 实战 | 掌握移植与裁剪全流程 | 自制嵌入式 Linux 系统 |
| [阶段六](./phases/phase6-projects.md) | 综合项目实战 | 独立完成项目 | 完整嵌入式产品原型 |

---

## 🎯 学习原则

1. **实践优先** — 每个概念都要动手验证，不要只看书
2. **循序渐进** — 不要跳阶段，基础不牢后面会非常痛苦
3. **记录笔记** — 用 Markdown 记录学习笔记，建立自己的知识库
4. **阅读源码** — 尽早养成阅读源码的习惯
5. **项目驱动** — 每个阶段结束时做一个综合小项目

## 🛠️ 推荐实验环境

### 硬件
- **开发板（阶段五购买）：** STM32MP157 或 i.MX6ULL 或 Raspberry Pi 4
- **USB转串口模块：** CH340 / CP2102
- **可选：** 逻辑分析仪（调试 I2C/SPI 用）

### 软件
- **宿主机 OS：** Ubuntu 22.04/24.04（虚拟机或双系统）
- **不建议用 WSL**（涉及 USB 设备和交叉编译时会有麻烦）
- **建议：** 在 VirtualBox/VMware 中安装 Ubuntu，或直接安装双系统

## 📚 核心推荐资源

详见 [resources.md](./reference/resources.md) | 🐧 [QEMU 完全指南](./reference/qemu-guide.md) | 🧭 [学习板选型指南](./hardware/board-selection-guide.md) | 🎯 [初学者买板指南](./hardware/beginner-board-guide.md) | 🔬 [动手实验](./labs/README.md) | 硬件平台：[平台总览](./hardware/hardware-platforms.md) | 异构多核：[AMP架构详解](./hardware/heterogeneous-amp-architecture.md) | 实战案例：[ZCU104 Sobel HLS 加速](./hardware/zcu104-sobel-hls-case-study.md)

### 必读书籍
1. 《鸟哥的 Linux 私房菜 — 基础学习篇》
2. 《C 程序设计语言》(K&R) + 《C 和指针》
3. 《操作系统概念》(恐龙书) 或 《现代操作系统》
4. 《Linux 设备驱动程序》(LDD3)
5. 《嵌入式 Linux 基础教程》(第2版)
6. 《Mastering Embedded Linux Programming》

---

## 📅 建议学习节奏

- **工作日：** 每天 1-2 小时
- **周末：** 每天 3-4 小时
- **每完成一个阶段：** 花 1-2 天复习和做阶段项目
- **每周日：** 回顾本周所学，整理笔记

---

**开始学习吧！快速 review [阶段一](./phases/phase1-linux-basics.md) 的编译工具链部分，然后进入 [阶段二：C 语言深度进修](./phases/phase2-c-deep-dive.md) 🚀**
