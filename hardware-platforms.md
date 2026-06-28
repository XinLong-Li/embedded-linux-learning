# 🔌 嵌入式 Linux 硬件平台全景指南

> **目标：** 全面了解全球主流嵌入式 Linux 芯片平台，为选型和求职提供参考
>
> 涵盖：芯片规格、适用场景、开发生态、招聘需求、薪资水平

---

## 📊 平台总览速查表

| 平台 | 架构 | 核心规格 | AI算力 | 难度 | 推荐度 |
|------|------|---------|--------|------|--------|
| **NXP i.MX 8M/9** | ARM Cortex-A | 1-6核A53/A55 | 0.5-2 TOPS | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **TI Sitara AM62x** | ARM Cortex-A | 1-4核A53 | 无 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Rockchip RK3588** | ARM big.LITTLE | 4×A76+4×A55 | 6 TOPS | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Raspberry Pi CM4/5** | ARM Cortex-A | 4×A72/A76 | 无 | ⭐ | ⭐⭐⭐⭐⭐ |
| **STM32MP2** | ARM Cortex-A35 | 1-2核A35 | ✓ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Allwinner T527/A523** | ARM Cortex-A55 | 8核A55 | 2 TOPS | ⭐⭐ | ⭐⭐⭐⭐ |
| **Nvidia Jetson Orin** | ARM Cortex-A78 | 8-12核A78 | 40-275 TOPS | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Qualcomm QCS/QRB** | ARM Kryo | 4-8核 | 3.5-12 TOPS | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **MediaTek Genio** | ARM v9 | 8核(含X925) | 6-50 TOPS | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **AMD Zynq MPSoC** | ARM+FPGA | 2-4核A53+R5 | 需FPGA加速 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Renesas RZ/G** | ARM/RISC-V | 1-4核A55 | ✓ (RZ/G3E) | ⭐⭐⭐ | ⭐⭐⭐ |
| **Microchip PolarFire** | RISC-V+FPGA | 4核RV64GC | 需FPGA加速 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Altera Agilex** | ARM+FPGA | 2核A55 | 2.5 TOPS | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

---

## 🔵 一、NXP i.MX 系列

> **定位：** 工业嵌入式 Linux 的"瑞士军刀"，覆盖面最广、资料最多
>
> **i.MX = innovative Multimedia eXtension**

### 家族演进

```
i.MX 6 (32位, ARMv7)       →  成熟稳定，存量市场巨大
    ↓
i.MX 8 (64位, ARMv8)       →  当前主力，四个子家族
    ↓
i.MX 9 (64位, ARMv8 + NPU) →  未来方向，AI + 安全
```

### 核心芯片对比

| 型号 | CPU | 协处理器 | GPU | NPU | 定位 |
|------|-----|---------|-----|-----|------|
| **i.MX 6ULL** | 1×A7 @ 900MHz | — | — | — | 入门级，成本极低 |
| **i.MX 8M Mini** | 4×A53 @ 1.8GHz | M4F | GC320/GLES | — | 多媒体处理 |
| **i.MX 8M Plus** | 4×A53 @ 1.8GHz | M7F | GC7000UL | **2.3 TOPS** | 工业 AI + 视觉 |
| **i.MX 91/91S** | 1×A55 | — | — | — | 最便宜 Linux MPU |
| **i.MX 93** | 2×A55 @ 1.7GHz | **M33** | — | **0.5 TOPS** | IoT + Edge AI |
| **i.MX 95** | 6×A55 | **M7 + M33** | ✓ | ✓ | 旗舰，汽车/机器人 |

### 开发生态
- **BSP：** Yocto / Buildroot / Debian
- **内核：** Linux 6.6 LTS（统一 BSP，i.MX 6/7/8/9 共用一个发布）
- **工具：** U-Boot + imx-mkimage + SCFW
- **启动安全：** HAB → EdgeLock Secure Enclave
- **资料：** NXP 官方文档齐全，社区活跃，中文资料丰富

### 开发者板推荐
- **FRDM i.MX 93** — NXP 官方入门板，性价比高
- **i.MX 8M Plus EVK** — 工业 AI 视觉评估

### 适用场景
工业 HMI、智能网关、医疗设备、汽车仪表、IoT 网关、边缘 AI 推理

---

## 🟠 二、Texas Instruments (TI) Sitara 系列

> **定位：** 工业控制之王，PRU 实时协处理器是独门绝技

### 核心芯片

| 型号 | CPU | 实时核心 | GPU | 定位 |
|------|-----|---------|-----|------|
| **AM62x** | 1-4×A53 @ 1.4GHz | M4F + **PRU** | 可选 | 入门工业控制 |
| **AM62Px** | 4×A53 | M4F + R5F | ✓ | 带显示的工业 HMI |
| **AM62A** | 4×A53 | M4F/R5F + C7x DSP | ✓ | 边缘 AI 视觉 |
| **AM64x** | 2×A53 | **R5F + PRU** (多核) | — | 实时工业通信 |
| **AM437x** | 1×A9 @ 1GHz | PRU-ICSS | ✓ | 经典老款，成本低 |

### PRU — TI 的杀手锏
- **PRU (Programmable Real-time Unit)**：可编程实时单元
- 独立于 ARM 核心运行，**确定性 I/O，周期级精度**
- 常用于：EtherCAT、PROFINET、电机控制、自定义协议
- **任何其他 ARM SoC 都无法做到同等精度的软件 I/O**

### 开发生态
- **SDK：** Processor SDK Linux（基于 Yocto/Arago）
- **内核：** Linux 5.10/6.1 LTS
- **社区：** TI E2E 论坛、BeagleBoard 社区
- **开发板：** BeaglePlay (AM625)、SK-AM62、BeagleBone Black (AM3358 经典)

### 适用场景
PLC、电机驱动、工业协议转换网关、传感器数据采集、电力自动化

### 招聘热度
工业自动化公司（西门子、ABB、汇川、台达）对 TI 平台有持续需求，稳定但不如消费电子波动大。

---

## 🟢 三、Rockchip 瑞芯微

> **定位：** 国产芯片性价比之王，AIoT 领域"卷王"
>
> **特点：** 规格拉满、价格激进、开源生态好

### 芯片矩阵

| 型号 | CPU | GPU | NPU | 视频 | 工艺 | 定位 |
|------|-----|-----|-----|------|------|------|
| **RK3562** | 4×A53 | Mali-G52 | 1 TOPS | 1080p | 22nm | 轻量 AIoT |
| **RK3568** | 4×A55 @ 2.0GHz | Mali-G52 | 1 TOPS | 4K60 | 22nm | 中端主力 |
| **RK3576** | 4×A72+4×A53 | Mali-G57 | **6 TOPS** | 8K30 | 8nm | AI 升级款 |
| **RK3588(S)** | 4×A76+4×A55 @ 2.4GHz | Mali-G610 | **6 TOPS** | 8K60 | 8nm | **旗舰** |

### RK3588 详细规格
- **CPU：** 4×Cortex-A76 (2.4GHz) + 4×Cortex-A55 (1.8GHz)
- **GPU：** Mali-G610 MP4 (Vulkan 1.2, OpenGL ES 3.2)
- **NPU：** 6 TOPS (INT8/INT4/FP16)，三核架构
- **视频：** 8K@60fps 解码 + 8K@30fps 编码
- **显示：** 四屏异显 (8K@60Hz + 2×4K@60Hz)
- **接口：** PCIe 3.0×4, USB 3.1×3, SATA 3.1, CAN-FD
- **功耗：** 满载 ~8W（需要散热片/风扇）

### 社区生态
- **Radxa ROCK 5B/5A** — RK3588 最活跃的开源板
- **Orange Pi 5** — RK3588S 性价比板
- **友善 NanoPi R6S/C** — RK3588 路由器/工控板
- **Firefly 系列** — 工业级 SOM 模组
- **内核主线化：** RK3588 已合入主线的部分较大，RK3568 完整度最高

### 适用场景
ARM PC/平板、8K 媒体播放器、边缘 AI 服务器、多路视频分析、NAS、高端广告机

---

## 🔴 四、STMicroelectronics STM32MP 系列

> **定位：** MCU 开发者"升级"到 Linux 的最平滑路径
>
> STM32 生态 + Cortex-M 协处理器 = 单片机工程师的"Linux 友好过渡"

### 家族对比

| 特性 | STM32MP1 (MP13/15) | STM32MP2 (MP21/23/25) |
|------|-------------------|----------------------|
| **A 核** | 1-2×Cortex-A7 (32位) | 1-4×Cortex-A35 (64位) |
| **最高主频** | ~800 MHz | ~1.5 GHz |
| **M 核** | Cortex-M4 | **Cortex-M33** + M0+ |
| **GPU** | 基础 | Vulkan 支持 |
| **NPU** | ❌ | ✅ |
| **安全** | TrustZone-A | TrustZone-A + TrustZone-M |
| **新接口** | 标准 | USB 3.0, PCIe, TSN 以太网 |

### MP2 的杀手锏：M33-TD 启动模式

传统模式：A 核先启动 → M 核被 Linux 唤醒（MP1 只能这样）

**MP2 新增的 M33-TD 模式：**
- M33 **先启动**，作为系统主控
- A35 可以完全关闭，**需要时才唤醒**
- 支持超低功耗待机（~1mW 级别）
- 类似"MCU 常驻 + MPU 按需"架构

### 开发生态
- **STM32CubeMX**：一站式芯片配置（引脚、时钟、设备树生成）
- **OpenSTLinux**：Yocto 生态，Linux 6.6 LTS
- **Bootlin 外部层**：Buildroot / OpenWrt 支持
- **开发板：** STM32MP157F-DK2、STM32MP257F-DK2
- **SoM 模组：** DH electronics DHCOS STM32MP2x（30×40mm）

### 适用场景
需要 Linux + 实时控制的物联网设备、工业 HMI、智能家电、医疗器械

---

## 🟣 五、AMD Xilinx Zynq 系列

> **定位：** FPGA + ARM 异构计算终极平台
>
> **特点：** 硬件可编程 + Linux，想做什么硬件加速都可以

### 家族概览

| 系列 | 定位 | ARM 核 | 代表芯片 |
|------|------|--------|---------|
| **Zynq-7000** | 入门 | 2×A9 @ 1GHz | XC7Z010/020/030 |
| **Zynq UltraScale+ MPSoC** | 主力 | 2-4×A53 + 2×R5F | ZU2~ZU19 |
| **Zynq RFSoC** | 射频 | 4×A53 + 2×R5F | RF-ADC/DAC 集成 |
| **Versal** | 下一代 | A72 + R5F | AI 引擎 |

### UltraScale+ MPSoC 详细规格

| 变体 | A 核 | GPU | 视频编解码 | 定位 |
|------|------|-----|-----------|------|
| **CG** | 2×A53 | ❌ | ❌ | 纯算力，无多媒体 |
| **EG** | 4×A53 | Mali-400 | ❌ | 通用 + GPU |
| **EV** | 4×A53 | Mali-400 | ✅ H.264/H.265 | 视觉 + 编解码 |

PL (Programmable Logic) 规模：8.1 万 ~ 114 万逻辑单元

### PS-PL 通信
- AXI4 总线互联（高带宽）
- AXI DMA（直接内存访问）
- 共享 DDR 内存

### 开发生态
- **PetaLinux**：Yocto 封装，一键构建 Linux
- **Vivado**：硬件设计（Verilog/VHDL/HLS）
- **Vitis**：应用软件开发
- **开发板：** ZCU102/104/106, ZCU111 (RFSoC), Trenz TE0813
- **学习曲线：** 最陡峭，需要同时懂 Linux 和 FPGA

### 适用场景
软件定义无线电、高速数据采集、实时图像处理、AI 推理加速、航空航天、军工

---

## ⚫ 六、Nvidia Jetson 系列

> **定位：** 边缘 AI 计算的绝对霸主
>
> **关键词：** CUDA、GPU 加速、ROS、自动驾驶

### 产品线

| 模组 | GPU | CPU | 内存 | AI 算力 | 功耗 | 定位 |
|------|-----|-----|------|---------|------|------|
| **Jetson Nano** | 128核 Maxwell | 4×A57 | 4GB | 472 GFLOPS | 5-10W | 入门（已停产）|
| **Jetson TX2 NX** | 256核 Pascal | 2×Denver+4×A57 | 4GB | 1.33 TFLOPS | 7.5-15W | 中端嵌入 |
| **Jetson Xavier NX** | 384核 Volta | 6×Carmel | 8/16GB | **21 TOPS** | 10-20W | AI 边缘 |
| **Jetson AGX Xavier** | 512核 Volta | 8×Carmel | 32GB | **32 TOPS** | 10-30W | 自动驾驶 |
| **Jetson Orin Nano** | 1024核 Ampere | 6×A78 | 4/8GB | **40 TOPS** | 7-15W | 新一代入门 |
| **Jetson Orin NX** | 1024核 Ampere | 8×A78 | 8/16GB | **100 TOPS** | 10-25W | AI 旗舰模组 |
| **Jetson AGX Orin** | 2048核 Ampere | 12×A78 | 32/64GB | **275 TOPS** | 15-60W | 数据中心级边缘 |

### 软件栈 — JetPack SDK
- **L4T (Linux for Tegra)**： Ubuntu 为基础的嵌入式 Linux
- **CUDA / cuDNN / TensorRT**：完整的 GPU AI 加速
- **DeepStream**：视频分析流水线
- **Isaac SDK**：机器人开发框架
- **ROS/ROS2**：原生支持

### 开发板/模组
- **Jetson Orin Nano Dev Kit**：$499，入门 AI 开发
- **Jetson AGX Orin Dev Kit**：$1999，旗舰开发

### 适用场景
自动驾驶感知、机器人 SLAM、多路视频 AI 分析、医学影像、智能零售

### 招聘热度
**极热。** 自动驾驶公司（小鹏、蔚来、百度 Apollo、Momenta、大疆车载）大量需要 Jetson 经验。

---

## 🟡 七、Qualcomm 高通 Dragonwing 系列

> **定位：** 从手机 SoC 降维打击嵌入式 IoT 市场
>
> **新任品牌 Dragonwing** 替代了之前混乱的 Snapdragon Q 命名

### 产品线

| 平台 | CPU | NPU AI | 定位 |
|------|-----|--------|------|
| **QCS2290 / QRB2210** | 4×Kryo 2.0GHz | — | 入门 IoT |
| **QCS4290 / QRB4210** | 8×Kryo | — | 中端 |
| **QCS5430** | 6nm, v9 | ✓ | 中端 AI (2025 新) |
| **QCS6490 / QCM6490** | 6nm, v9 | ✓ 3.5 TOPS | 主力（RB3 Gen 2） |
| **QCS8250 / QRB5165** | 7nm, v8 | 15 TOPS | 高端（RB5） |
| **QCS9100** | v9 | ✓ | 旗舰 AI 推理 |

### 开发板
- **RB3 Gen 2 Vision Kit**（QCS6490）：2025 年 ROS 2 主力平台
- **RB5**（QRB5165）：高端机器人开发

### 软件生态
- **Qualcomm Linux**：Yocto 构建（qcom-6.6.90-QLI 内核）
- **QIRP SDK**：机器人 SDK，ROS 2 Jazzy 原生支持
- **2D-Lidar-SLAM、Hand Detection 等 AI 示例**
- **Tria 模组**：SMARC/COM Express 工业模组

### 适用场景
ROS 机器人、5G IoT 网关、无人机、AI 相机、工业视觉

---

## 🔷 八、Raspberry Pi (Broadcom)

> **定位：** 嵌入式 Linux 的"Hello World"，人手一块
>
> **最重要的贡献：** 把嵌入式 Linux 的门槛降到最低

### 三代芯片对比

| 特性 | BCM2711 (Pi 4) | BCM2712 (Pi 5/CM5) |
|------|---------------|-------------------|
| **CPU** | 4×Cortex-A72 @ 1.5GHz | 4×**Cortex-A76** @ 2.4GHz |
| **GPU** | VideoCore VI | VideoCore VII |
| **RAM** | 1-8GB LPDDR4 | 1-**16GB** LPDDR4X (ECC) |
| **PCIe** | 1× Gen 2.0 | 1× Gen 2.0 |
| **USB3** | 2× (共享 5Gbps) | 2× (各自 5Gbps) |
| **eMMC** (CM) | CM4: 8-32GB | **CM5: 16-64GB** (2× 速度) |
| **温度** | -20~85°C | **-20~85°C** |
| **生命周期** | ~2028 | **到 2036 年！** |

### Compute Module 系列 — 嵌入式产品化的关键
- CM4/CM5 是 Raspberry Pi 基金会为工业/嵌入式专门设计的模组
- 55mm×40mm 标准尺寸，丰富的 SoM 生态
- **Kontron Pi-Tron CM5** (2025)：工业级 CM5 载板
- **Pi Terminal**：CM4/CM5 工控设备

### 限制
- ❌ 没有 NPU（无 AI 硬件加速）
- ❌ 没有工业实时总线（CAN 需外接）
- ❌ 没有 MIPI DSI/CSI 之外的摄像头/显示接口
- ✅ 但生态最强大，几乎所有嵌入式 Linux 教程都支持

### 适用场景
原型验证、教育、轻量工控、HMI 面板、数字标牌、NAS、实验室设备

---

## 🔶 九、Allwinner 全志科技

> **定位：** 国产中低端之王，平板/盒子/智能家居芯片
>
> **特点：** 种类繁多、成本极低、SDK 逐步开放

### 2025 年核心芯片

| 芯片 | CPU | GPU | NPU | 定位 |
|------|-----|-----|-----|------|
| **T113-S3/i** | 2×A7 @ 1.2GHz + HiFi4 | — | — | 超低成本工控（内置 DDR） |
| **V853** | A7 + RISC-V + AI-ISP | — | **1 TOPS** | AI 视觉（ADAS/安防） |
| **T527** | 8×A55 @ 1.8GHz + RISC-V E906 | Mali-G57 | **2 TOPS** | 车规 AI（AEC-Q100） |
| **A523** | 8×A55 | Mali | ✓ | 平板/教育 |
| **A733** | 8×A76 @ 2.0GHz | Mali-G610 | ✓ | 高性能平板/边缘服务器 |
| **D1-H** | RISC-V 玄铁 C906 | — | — | RISC-V Linux（IoT） |

### 2025 重要进展
- **A527/T527/A733 文档全面公开**，无需 NDA
- 数据手册 + 用户手册 + Linux SDK 在 Gitlab 上免费获取
- Linux 主线化推进中（A523 已有补丁提交）

### 开发板代表
- **Sipeed LicheeRV** (D1-H)：RISC-V Linux 入门
- **Sipeed LicheePi 4A** (TH1520, 4×C910)：RISC-V 开发板
- **Banana Pi / Orange Pi** 各种型号

### 适用场景
消费电子、车载中控、智能家居、工业 PLC、AI 安防摄像头

---

## 🟤 十、MediaTek Genio 系列

> **定位：** 手机芯片巨头"杀入"嵌入式 IoT
>
> **亮点：** 3nm 制程下放，2026 年旗舰功耗比逆天

### 2025-2026 产品线

| 平台 | 制程 | CPU | NPU | 温度 | 状态 |
|------|------|-----|-----|------|------|
| **Genio 360/360P** | — | — | 6/8.5 TOPS | — | 2025 |
| **Genio 420** | 6nm | 2×A78+6×A55 | **7.2 TOPS** | -20~95°C | 2026 Q2 |
| **Genio 520** | 6nm | 2×A78+6×A55 | <10 TOPS | -40~105°C | 2025 |
| **Genio 720** | 6nm | 2×A78+6×A55 | **10 TOPS** | -40~105°C | 2025 |
| **Genio Pro 5100** | **3nm** | 1×X925+3×X4+4×A720 | **50+ TOPS** | **-40~105°C** | **2026 Q3** |

### Genio Pro 5100 旗舰亮点
- **TSMC 3nm** — 嵌入式领域首次用上先进制程
- **7B LLM 推理 23 tokens/s** — 边缘运行大语言模型
- 2.5GbE TSN × 2, PCIe Gen4 × 3, Wi-Fi 7
- **10 年供货保证**

### 开发生态
- NeuroPilot SDK (PyTorch/ONNX/TFLite)
- Yocto / Debian / Ubuntu / ROS 2
- Pin 兼容（420/520/720 可互换）

### 适用场景
商用机器人、无人机、工业 AI 推理、智能零售、医疗影像

---

## 🟣 十一、Renesas 瑞萨 RZ 系列

> **定位：** 日系工业电子的"隐形冠军"
>
> **优势：** 工业温度范围广、长期供货保证、实时控制

### 产品矩阵

| 系列 | CPU | 特色 | 定位 |
|------|-----|------|------|
| **RZ/G2L** | 2×A55+M33 | 通用 Linux + 实时 | 工业 HMI |
| **RZ/G2UL** | 1×A55+M33 | 低成本 Linux | 网关 |
| **RZ/G3E** 🆕 | **4×A55+M33+** Ethos-U55 NPU | AI 加速，PCIe 3.0, USB 3.2 Gen2 | 2025 新旗舰 HMI |
| **RZ/V2H** | 4×A55+M33+2×R8 | DRP-AI3 **8 TOPS** | 视觉 AI |
| **RZ/V2L** | 2×A55+M33 | DRP-AI 轻量版 | 入门视觉 |
| **RZ/Five** | 1×RISC-V AX45MP | **RISC-V Linux** | RISC-V 替代 |

### RZ/G3E 特色（2025 年新品）
- Ethos-U55 microNPU 用于边缘 AI 推理
- 超低待机功耗（~50mW 待机 / ~1mW 深度睡眠）
- **15 年产品生命周期保证**

### 适用场景
工业 HMI、工厂自动化、视觉检测、医疗设备、楼宇自动化

---

## ⚪ 十二、Microchip PolarFire RISC-V SoC FPGA

> **定位：** RISC-V + FPGA 的开放替代方案（Xilinx 的挑战者）

### 架构

| 组件 | 规格 |
|------|------|
| **应用核** | 4×RV64GC @ 667 MHz |
| **监控核** | 1×RV64IMAC |
| **FPGA** | 25K ~ 500K 逻辑单元 |
| **内存** | DDR4/LPDDR4 with SECDED ECC |
| **安全** | SEU 免疫 (Flash FPGA)、安全启动 |

### 2025 关键里程碑
- ✅ **AEC-Q100 Grade 1** 认证（-40~125°C）
- 汽车电子正式入场

### 开发板
- **BeagleV-Fire**：$150 级别，BeagleBoard 合作（MPFS025T）
- **PolarFire SoC Icicle Kit**：$750，工业原型（MPFS250T）

### 适用场景
汽车网关、航空航天、工业实时控制+边缘计算、安全关键系统

---

## 🔘 十三、Altera (Intel) Agilex SoC FPGA

> **定位：** Intel 工艺加持的 FPGA + ARM 平台
>
> **2025 年：** Altera 从 Intel 独立，更灵活

### 产品线

| 系列 | 定位 | CPU | AI |
|------|------|-----|-----|
| **Agilex 3** | 低成本 | 2×A55 @ 800MHz | 2.54 TOPS |
| **Agilex 5** | 中端 | ARM HPS | ✓ DDR5/LPDDR5 |
| **Agilex 7** | 高端 | ARM HPS | ✓ 高性能 |
| **Agilex 9** | 旗舰 | ARM HPS | ✓ 数据中心 |

### Agilex 3 亮点（2025 年 4 月发布）
- Intel 7 制程，比上代功耗降低 **38%**
- AI Tensor Block
- 25K ~ 135K 逻辑单元
- FPGA Manager Linux 驱动已提交主线（kernel 6.19 预期）

### 开发工具
- Quartus Prime v25.3
- Visual Designer Studio（AI 辅助 IP 连线）
- Arm Development Studio

---

## 🟩 十四、Espressif 乐鑫 — 嵌入式 Linux？❌

> **重要声明：** 乐鑫的 ESP32 系列（包括 ESP32-P4）**不能运行 Linux**！
>
> 它们是 **MCU（微控制器）**，没有 MMU，只能跑 FreeRTOS/裸机。

### 为什么乐鑫值得了解

乐鑫在嵌入式领域极其重要，但它在 **MCU + RTOS** 赛道，不在 Linux 赛道：

| 芯片 | 架构 | 特点 |
|------|------|------|
| ESP32-S3 | Xtensa LX7 | WiFi+BLE, AI 加速 |
| ESP32-C6 | RISC-V | WiFi 6 + BLE 5 + 802.15.4 |
| **ESP32-P4** | RISC-V 双核 400MHz | H.264 编码, MIPI CSI/DSI, PPA |

### ESP32-P4 — 离 Linux 最近但仍不是 Linux
- 双核 RISC-V @ 400MHz，MIPI-CSI/DSI，H.264 编码器
- 看起来像 Linux SBC，但**仍无 MMU**
- 用 ESP-IDF 开发（FreeRTOS 变体）
- 可以跑 **NOMMU Linux**（实验性，非正式支持）

> **结论：** 如果你关心的是能跑 Linux 的芯片，乐鑫不是选项。但如果做 IoT 终端节点（传感器采集、WiFi 透传），ESP32 是必学。

---

## ⬜ 十五、其他值得关注的平台

### THC (太湖) 国产 RISC-V 平台
- 国产信创重点方向
- 多款 RISC-V Linux SBC 涌现

### StarFive 赛昉科技
- **JH7110**：4×SiFive U74 RISC-V @ 1.5GHz
- VisionFive 2 开发板（RISC-V Linux 主力板）

### SOPHGO 算能
- **SG2042**：64 核 RISC-V 服务器芯片
- **SG2044**：新一代 RISC-V
- Milk-V Pioneer 主板

### Broadcom (树莓派之外)
- BCM 系列在路由器/机顶盒中广泛应用
- 开源程度较低，不适合学习

---

## 💼 十六、嵌入式 Linux 就业市场全景

### 2025-2026 中国招聘行情

#### 各城市薪资范围（嵌入式 Linux）

| 经验 | 北京 | 上海 | 深圳 |
|------|------|------|------|
| 应届 | 11K-20K | 12K-22K | 12K-25K |
| 3-5年 | 20K-35K | 22K-38K | 20K-35K |
| 5-10年 | 30K-50K+ | 35K-60K+ | 30K-55K+ |
| 10年+ | 50K-100K+ | 50K-100K+ | 50K-80K+ |

#### 薪资溢价技能

| 技能 | 薪资增幅 | 备注 |
|------|---------|------|
| Linux 内核/驱动 | **+20~30%** | 芯片公司最看重 |
| AI 边缘计算 (NPU 适配) | **+30~50%** | 2025 最热门 |
| 汽车电子 (AUTOSAR) | **+20~35%** | 稳定增长 |
| RISC-V 架构 | **+15~25%** | 早期红利 |
| Yocto/Buildroot | **+10~15%** | 已经是标配 |
| Rust (内核开发) | **early stage** | 未来趋势 |

### 芯片公司招聘热度排行

```
🔥🔥🔥 最热：华为海思、地平线、寒武纪、Nvidia
🔥🔥   很热：大疆、NXP、TI、高通、瑞芯微
🔥    稳定：全志、ST、联发科、瑞萨、兆易创新
```

### 热门招聘方向

1. **自动驾驶/汽车电子** — CAN/车载以太网/AUTOSAR/功能安全
2. **AI 芯片 BSP/驱动** — 国产 GPU/AI 芯片公司的"抢人"大战
3. **机器人/ROS2** — 大疆、宇树、追觅、普渡
4. **工业 IoT/边缘计算** — MQTT/OPC UA/Modbus + Linux
5. **国产替代/信创** — 飞腾/鲲鹏/海思平台

---

## 📊 选型决策指南

### 按应用场景推荐

| 场景 | 首选 | 次选 | 原因 |
|------|------|------|------|
| **学习入门** | Raspberry Pi 5 | STM32MP157-DK2 | 生态最好/资料最多 |
| **工业 HMI** | i.MX 8M Plus | RZ/G2L | 显示丰富/工业级 |
| **机器人 ROS2** | Jetson Orin NX | Qualcomm RB3 Gen2 | GPU/NPU 算力 |
| **工业实时控制** | TI AM64x | STM32MP2 | PRU 实时单元 |
| **AI 边缘推理** | Jetson Orin | Genio 720 | GPU/NPU 算力 |
| **低成本 Linux** | i.MX 91/93 | RK3568 | 芯片便宜 |
| **视频分析** | RK3588 | T527 | 8K 解码/ISP |
| **FPGA + Linux** | Zynq MPSoC | PolarFire | 硬件加速 |
| **超低功耗 Linux** | STM32MP2 (M33-TD) | RZ/G3E | 待机功耗 |
| **国产化要求** | RK3588 | T527/A733 | 国产+生态好 |

### 按难度梯度推荐

```
入门（学习）  →  Raspberry Pi 5 → CM4/CM5 → STM32MP157
    ↓
进阶（工业）  →  i.MX 8M Plus → TI AM62x → RK3568/RK3588
    ↓
高级（AI）    →  Jetson Orin → Qualcomm RB3 → MediaTek Genio
    ↓
专家（FPGA）  →  Zynq MPSoC → PolarFire → Agilex
```

---

## 📚 资源链接

- **芯片选型工具：** [NXP Product Selector](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors:IMX_HOME)
- **TI Sitara 选型：** [TI Sitara Processors](https://www.ti.com/microcontrollers-mcus-processors/arm-based-processors/overview.html)
- **Rockchip Wiki：** [Rockchip Linux](https://opensource.rock-chips.com/)
- **Raspberry Pi 文档：** [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- **全志 Linux SDK：** [Allwinner Gitlab](https://gitlab.com/allwinner-zone)
- **Jetson 开发者：** [Nvidia Jetson](https://developer.nvidia.com/embedded-computing)
- **Buildroot 官方：** [Buildroot Manual](https://buildroot.org/downloads/manual/manual.html)
- **Yocto Project：** [Yocto Documentation](https://docs.yoctoproject.org/)

---

> 💡 **最后的建议：** 先精通一个平台（推荐 i.MX 或 Rockchip），再横向扩展。平台间原理相通，不要贪多。
>
> **如果你志在求职：** 重点学习 i.MX / Rockchip / STM32MP 之一，它们是招聘中提及频率最高的通用嵌入式 Linux 平台。
