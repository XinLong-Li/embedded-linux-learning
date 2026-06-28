# 🧩 异构多核 AMP 架构深度解析

> **核心问题：** 芯片上有 Cortex-A + Cortex-M/Cortex-R/RISC-V 多种核，A 核跑 Linux，小核跑 RTOS/裸机，它们如何通信协作？
>
> **关键词：** AMP、OpenAMP、RPMsg、remoteproc、VirtIO、Mailbox/IPCC、共享内存

---

## 📌 一、什么是异构多核 AMP 架构？

### 1.1 SMP 与 AMP 的区别

```
SMP (对称多处理)                    AMP (非对称多处理)
─────────────────                    ─────────────────
   Linux                            Linux       RTOS
  ┌───┬───┐                        ┌───┐      ┌───┐
  │A53│A53│                        │A53│      │M33│
  │   │   │    同一个 OS             │   │      │   │    不同 OS
  └───┴───┘                        └───┘      └───┘
    共享内存    统一管理               共享内存    各管各的
                                     └── OpenAMP ──┘
```

**AMP (Asymmetric Multi-Processing)**：不同架构的 CPU 核心运行**不同的操作系统**，各自管理自己的资源，通过标准化的通信框架交换数据。

### 1.2 为什么需要 AMP？

| 需求 | Linux (A 核) 的劣势 | RTOS/裸机 (M/R 核) 的优势 |
|------|-------------------|--------------------------|
| **硬实时响应** | 中断延迟 ~10-100μs | 中断延迟 **<1μs ~ 100ns** |
| **确定性执行** | 调度器会抢占，不可预测 | 确定性的任务执行时间 |
| **低功耗待机** | Linux 唤醒慢（秒级） | M 核可独立运行，功耗 μW 级 |
| **功能安全** | Linux 未认证 | M/R 核可达 ASIL-D/SIL4 |
| **复杂业务** | ✅ 网络/文件系统/GUI/AI | ❌ 资源受限 |

**典型场景：**
- 机器人：A 核跑 ROS2 + AI 视觉，M 核做电机实时控制
- 汽车：A 核跑 IVI 车载系统，R 核做 CAN/底盘安全控制
- 工业：A 核跑 HMI + 云连接，M 核做 PLC 逻辑 + EtherCAT

---

## 📐 二、通用通信架构：OpenAMP 标准栈

### 2.1 四层模型

```
┌──────────────────────────────────────────────────────┐
│                   应用层                              │
│   Linux: /dev/ttyRPMSGx, /dev/rpmsgx, socket         │
│   RTOS:  rpmsg_send() / rpmsg_receive()             │
├──────────────────────────────────────────────────────┤
│              RPMsg (消息总线层)                        │
│   端点(endpoint)管理、服务通告(name service)           │
│   异步非阻塞消息传递                                   │
├──────────────────────────────────────────────────────┤
│           VirtIO (虚拟化传输层)                        │
│   vring 环形缓冲区管理 (TX/RX 单向对)                  │
│   零拷贝 shared memory 传输                           │
├──────────────────────────────────────────────────────┤
│       remoteproc (生命周期管理层)                       │
│   Linux 侧：加载/启动/停止/监控远程核固件                │
│   RTOS 侧：Resource Table (声明需要的资源)             │
├──────────────────────────────────────────────────────┤
│          硬件层 (Mailbox/IPCC/IPI + 共享内存)           │
│   IPCC: 中断通知对方 "有新数据"                        │
│   Shared DDR/OCRAM: 数据交换载体                      │
└──────────────────────────────────────────────────────┘
```

### 2.2 通信流程（M核 → A核 发送数据为例）

```
M核应用
  │
  ├─ rpmsg_send(data, len)
  │    │
  │    └─ 写入 data 到共享内存的 vring TX buffer
  │       └─ 更新 vring 描述符
  │          └─ virtqueue_kick()
  │             └─ 写 IPCC 寄存器 → 触发 A核硬件中断
  │
  └─ 立即返回（异步！）

                    [IPCC 中断]

A核收到中断
  │
  └─ IPCC ISR
     └─ virtio 解析 vring 描述符
        └─ RPMsg 总线将消息路由到对应端点
           └─ Linux 用户空间 read() 获取数据
```

**核心优势：完全异步！** M 核发完消息即可继续采集数据，不等待 A 核响应。

---

## 🏭 三、各大厂商异构 AMP 实现对比

### 3.1 总览对比表

| 厂商 | 芯片系列 | A 核 (Linux) | 从核 (RTOS/裸机) | 通信框架 | 独有特性 |
|------|---------|-------------|------------------|---------|---------|
| **ST** | STM32MP15x | 1-2×A7 @800MHz | **Cortex-M4** @209MHz | OpenAMP | MCU→MPU 最平滑过渡 |
| **ST** | STM32MP25x | 1-2×A35 @1.5GHz | **Cortex-M33** @400MHz | OpenAMP | **M33-TD 先启动模式** |
| **NXP** | i.MX 8M Plus | 4×A53 @1.8GHz | **Cortex-M7** @800MHz | RPMsg-lite | AN14120 官方指南 |
| **NXP** | i.MX 93 | 2×A55 @1.7GHz | **Cortex-M33** @250MHz | OpenAMP | EdgeLock 安全 + Ethos-U65 NPU |
| **NXP** | i.MX 95 | 6×A55 | **Cortex-M7+M33** | OpenAMP | 3 种核共存 |
| **TI** | AM62x | 1-4×A53 @1.4GHz | **Cortex-M4F** + PRU | OpenAMP / IPC | **PRU 周期级确定性 I/O** |
| **TI** | AM64x | 2×A53 @1.0GHz | **Cortex-R5F**×4 + PRU | OpenAMP / IPC | 工业协议硬件加速 |
| **AMD** | Zynq-7000 | 2×A9 @1GHz | (无 M/R 核) | OpenAMP | FPGA 协处理 |
| **AMD** | ZynqMP | 2-4×A53 @1.5GHz | **Cortex-R5F**×2 | OpenAMP | **FPGA 硬件加速 + R5 实时** |
| **全志** | T113-i | 2×A7 @1.2GHz | **玄铁 C906 RISC-V** @1GHz + **HiFi4 DSP** | OpenAMP | 三核异构，**79元起** |
| **全志** | V853 | 1×A7 @900MHz | **玄铁 E907 RISC-V** @600MHz | OpenAMP/MSGBOX | AI-ISP + NPU + RISC-V |
| **瑞萨** | RZ/G2L | 2×A55 @1.2GHz | **Cortex-M33** | OpenAMP | 15年供货保证 |
| **瑞萨** | RZ/V2H | 4×A55 @1.8GHz | **Cortex-M33** + **Cortex-R8**×2 | - | DRP-AI 8 TOPS |

---

### 3.2 STM32MP2 — M33-TD 模式（独有杀手锏）

这是 STM32MP2 最独特的特性：

```
传统 AMP (MP1, 其他厂商)：          MP2 的 M33-TD 模式：
                                    
  A7 先启动                          M33 先启动！
  │                                  │
  ├─ ROM Boot                       ├─ ROM Boot  
  ├─ TF-A                           ├─ MCUboot (安全启动)
  ├─ U-Boot                         ├─ RTOS/裸机 初始化
  ├─ Linux ← remoteproc 加载 M4      ├─ 实时任务运行中...
  └─ M4 在 Linux 启动后才开始工作       │
                                      ├─ "需要 Linux 了" → 唤醒 A35
                                      ├─ TF-A → U-Boot → Linux
                                      └─ A35 启动后，M33 已工作完毕！

  状态：M4 永远是"从"                状态：M33 可以是"主"！
```

**价值：**
- 设备上电后 **毫秒级** 即可开始实时控制（不等 Linux 启动）
- 紧急事件可由 M33 独立处理并记录，A 核故障不影响
- 低功耗场景：平时 A35 完全断电，M33 值守，需要时唤醒

---

### 3.3 TI Sitara — PRU 实时协处理器（独有杀手锏）

TI 的 PRU (Programmable Real-time Unit) 是**嵌入式世界最独特的实时解决方案**：

```
AM64x 芯片
├── Cortex-A53 (Linux)    ← HMI、网络、云连接
├── Cortex-R5F ×4 (RTOS)  ← 实时控制逻辑
├── PRU-ICSSG ×2          ← 👑 周期级确定性 I/O
│   ├── 可编程定制协议
│   ├── EtherCAT Slave 硬件实现
│   ├── PROFINET
│   └── 自定义高速 GPIO 时序
└── 共享内存 + Mailbox
```

**为什么 PRU 无可替代？**
- ARM 核的 GPIO 翻转 **受中断延迟影响**（~1-10μs 抖动）
- PRU 的 GPIO 翻转是**周期级确定性**的（无抖动）
- 可以在 Linux 完全不参与的情况下，以纳秒精度生成/捕获时序信号
- 能实现软件定义的 EtherCAT 从站、Σ-Δ ADC 接口等

> **一句话：** 需要工业高速实时 I/O 而你不想外挂 FPGA → TI AM64x 是不二之选。

---

### 3.4 AMD Zynq UltraScale+ — A53 + R5 + FPGA 三异构

ZynqMP 是异构架构的"终极形态"：

```
┌──────────────────────────────────────────────┐
│           Zynq UltraScale+ MPSoC              │
│                                              │
│  PS (Processing System)     PL (Programmable Logic)
│  ┌────────────────────┐    ┌─────────────────┐
│  │ Cortex-A53 ×4      │    │  FPGA 逻辑       │
│  │  (Linux/PetaLinux) │◄──►│  (AXI 总线互联)   │
│  │                    │    │                  │
│  │ Cortex-R5F ×2      │    │  自定义 IP:      │
│  │  (FreeRTOS/裸机)    │    │  - AI 加速器     │
│  │                    │    │  - 视频处理流水线  │
│  │ GPU Mali-400       │    │  - 自定义协议     │
│  │                    │    │  - DSP 协处理    │
│  └────────────────────┘    └─────────────────┘
│                                              │
│  R5 ↔ A53: OpenAMP/RPMsg                     │
│  PL ↔ PS:  AXI4 + AXI DMA (最高带宽)          │
└──────────────────────────────────────────────┘
```

**三种异构层次：**
1. **A53 ↔ R5F**：标准的 OpenAMP AMP 通信（Linux ↔ RTOS）
2. **A53 ↔ FPGA**：AXI 总线 + DMA（硬件加速器）
3. **R5F ↔ FPGA**：低延迟实时控制 + 硬件加速

**R5 的两种运行模式：**
- **Split 模式**：两个 R5 独立运行（各司其职）
- **Lockstep 模式**：两个 R5 执行相同代码，硬件比对结果（功能安全 ASIL-D）

---

### 3.5 NXP i.MX — 丰富的 RPMsg 方案

i.MX 系列提供了最多的异构组合：

```
i.MX 8M Plus                     i.MX 93                      i.MX 95
┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│ A53 ×4 (Linux)   │         │ A55 ×2 (Linux)   │         │ A55 ×6 (Linux)   │
│       ↕          │         │       ↕          │         │       ↕          │
│ M7 ×1 (RTOS)     │         │ M33 ×1 (RTOS)    │         │ M7 ×1 (RTOS)     │
│       ↕          │         │                  │         │       ↕          │
│ (HiFi4 DSP 可选)  │         │ Ethos-U65 NPU   │         │ M33 ×1 (安全协处理)│
└──────────────────┘         └──────────────────┘         └──────────────────┘

通信方式：                    通信方式：                      通信方式：
- RPMsg-lite (轻量级)         - OpenAMP 标准栈                - OpenAMP + SCFW
- RPMsg TTY 驱动              - RPMsg Char 驱动               - Vendor resource entries
- CAN-FD 备用通道              - Hypervisor (Jailhouse)       - Address translation
```

**NXP 的独有特性：RPMsg-lite**
- 是 RPMsg 的极简实现，去掉了 VirtIO 的完整抽象
- 代码更小、延迟更低
- 适用于不需要 VirtIO 全部特性的场景
- 官方指南：**AN14120** 应用笔记

**i.MX 95 的独特能力：**
- 三种核共存：A55(Linux) + M7(硬实时) + M33(安全协处理)
- **Energy Flex** 架构：每核独立电源域，精细化功耗管理
- M7 可访问 NPU 进行实时 AI 推理

---

### 3.6 全志 T113 — 国产三核异构（性价比之王）

T113-i 是目前**最便宜的 ARM + RISC-V + DSP 三核异构芯片**：

```
T113-i 芯片                       价格：核心板含税 ~79元
┌────────────────────────────────────────┐
│                                        │
│  Cortex-A7 ×2 (1.2GHz)                 │
│  ├─ Linux (Buildroot/Yocto)            │
│  ├─ HMI, 网络, 文件系统                  │
│  └─ OpenAMP Master                     │
│              ↕ RPMsg + RPBuf           │
│  玄铁 C906 RISC-V (1.0GHz)              │
│  ├─ FreeRTOS / RT-Thread / 裸机         │
│  ├─ 实时采集、电机控制                    │
│  └─ OpenAMP Remote + MSGBOX            │
│              ↕                         │
│  HiFi4 DSP (600MHz)                     │
│  └─ 音频处理、数字信号处理                │
│                                        │
│  共享内存: 内置 128MB DDR3 (T113-S3)     │
└────────────────────────────────────────┘
```

**实测性能数据 (T113-i RPMsg)：**
| 方向 | 数据量 | 耗时 | 速率 |
|------|--------|------|------|
| RISC-V → ARM (发) | 496KB | 20ms | **24.79 Mb/s** |
| ARM → RISC-V (收) | 496KB | 9980ms | 0.05 Mb/s |

> ⚠️ 接收性能偏低，实际项目中需优化 Linux 端读路径（改用轮询、增大缓冲区等）

**为什么备受青睐？**
- **成本极低**：核心板 79 元，比 STM32H7 还便宜
- **ARM+RISC-V+DSP**：三种架构覆盖几乎所有场景
- **汽车级温度**：-40°C ~ +85°C
- 已有近 **2000 家企业**批量使用

---

## 🔧 四、开发实操：以 STM32MP2 为例

### 4.1 总体步骤

```
1. 准备 M33 固件 (FreeRTOS/Zephyr/裸机)
   └─ .elf 文件，必须包含 .resource_table 段

2. 配置 Linux 设备树
   ├─ 预留共享内存区域 (reserved-memory)
   ├─ 配置 IPCC/Mailbox 节点
   └─ 配置 remoteproc 节点

3. 编译 Linux 内核
   ├─ CONFIG_REMOTEPROC=y
   ├─ CONFIG_RPMSG_CHAR=y
   └─ CONFIG_RPMSG_TTY=y

4. 部署 & 运行
   ├─ 将 M33 固件放到 /lib/firmware/
   ├─ echo firmware.elf > /sys/class/remoteproc/remoteproc0/firmware
   └─ echo start > /sys/class/remoteproc/remoteproc0/state
```

### 4.2 设备树配置要点

```dts
/ {
    reserved-memory {
        /* M33 固件代码/数据区 */
        m33_fw: m33-fw@80000000 {
            reg = <0x80000000 0x400000>;   /* 4MB */
            no-map;                          /* 不用内核的 cache 管理 */
        };

        /* RPMsg vring 和共享缓冲区 */
        vdev0vring0: vdev0vring0@80040000 {
            reg = <0x80040000 0x1000>;      /* TX vring */
            no-map;
        };
        vdev0vring1: vdev0vring1@80041000 {
            reg = <0x80041000 0x1000>;      /* RX vring */
            no-map;
        };
        vdev0buffer: vdev0buffer@80042000 {
            reg = <0x80042000 0x100000>;    /* 共享消息缓冲 */
            no-map;
        };
    };

    /* RemoteProc 节点 */
    m33_rproc: remoteproc@0 {
        compatible = "st,stm32mp2-rproc";
        memory-region = <&m33_fw>, <&vdev0buffer>,
                        <&vdev0vring0>, <&vdev0vring1>;
        mboxes = <&ipcc 0>, <&ipcc 1>;
        mbox-names = "vq0", "vq1";
    };
};
```

### 4.3 M33 固件关键代码

```c
// 1. Resource Table — 必须！Linux 靠它识别通信参数
__section(".resource_table")  // 放在指定段
struct resource_table {
    uint32_t ver = 1;
    uint32_t num = 1;
    uint32_t reserved[2] = {0};
    uint32_t offset[1] = {offsetof(resource_table, vdev)};

    // 声明 virtio 设备
    struct fw_rsc_vdev vdev = {
        .type = RSC_VDEV,
        .id = VIRTIO_ID_RPMSG,
        .num_of_vrings = 2,     // TX + RX
    };

    // vring0 — M33 → A35 (TX)
    struct fw_rsc_vdev_vring vring0 = {
        .da = 0x80040000,       // 必须匹配设备树！
        .align = 4096,
        .num = 256,             // 缓冲区数量
    };

    // vring1 — A35 → M33 (RX)
    struct fw_rsc_vdev_vring vring1 = {
        .da = 0x80041000,
        .align = 4096,
        .num = 256,
    };
} resource_table;

// 2. 初始化
void main(void) {
    // 初始化 IPCC 硬件
    MX_IPCC_Init();

    // 初始化 OpenAMP 框架
    MX_OpenAMP_Init(RPMSG_REMOTE, NULL);

    // 创建 RPMsg 端点
    struct rpmsg_endpoint *ept;
    rpmsg_create_ept(&ept, "my-channel", 
                      RPMSG_ADDR_ANY, RPMSG_ADDR_ANY,
                      my_rx_callback, NULL);

    // 等待远程处理器就绪
    while (!rpmsg_is_ready()) {
        MX_OpenAMP_Process();  // 处理 OpenAMP 事件
    }

    // 发送第一条消息！
    rpmsg_send(ept, "Hello from M33!", 17);
}

// 3. 接收回调
int my_rx_callback(struct rpmsg_endpoint *ept, 
                   void *data, size_t len,
                   uint32_t src, void *priv) {
    // 处理从 Linux 发来的消息
    process_command(data, len);
    return RPMSG_SUCCESS;
}
```

### 4.4 Linux 侧测试

```bash
# 加载驱动
modprobe rpmsg_char && modprobe rpmsg_ctrl

# 启动 M33 固件
echo stm32mp2_m33_fw.elf > /sys/class/remoteproc/remoteproc0/firmware
echo start > /sys/class/remoteproc/remoteproc0/state

# 检查状态
cat /sys/class/remoteproc/remoteproc0/state
# 输出: running

# 确认 RPMsg 通道建立
dmesg | grep rpmsg
# virtio_rpmsg_bus virtio0: creating channel my-channel addr 0x400

# 用户空间通信
cat /dev/rpmsg0          # 接收 M33 消息
echo "hello" > /dev/rpmsg0  # 向 M33 发送
```

---

## ⚡ 五、通信方案选型决策树

```
需要实时核与 Linux 通信？
│
├── 需要 ns 级确定性 I/O？(工业协议)
│   └── YES → TI AM64x/AM62x (PRU 无人能敌)
│
├── 需要 FPGA 硬件加速？
│   ├── 高性能 → AMD Zynq UltraScale+ (A53 + R5F + FPGA)
│   └── RISC-V 开放 → Microchip PolarFire SoC
│
├── 超低功耗？（A 核可完全断电）
│   └── → STM32MP2 (M33-TD 模式独有)
│
├── 成本极敏感？(<100元)
│   └── → 全志 T113-i (ARM+RISC-V+DSP, 79元)  
│
├── 需要功能安全认证？(ASIL-D/SIL4)
│   ├── 汽车 → STM32MP2 / TI TDA4 / NXP S32
│   └── 工业 → ZynqMP Lockstep / TI Hercules
│
├── 简单实时任务？（用 RPMsg-lite 足够）
│   └── → NXP i.MX 8M Plus (AN14120 指南完善)
│
└── 选型不确定？先学通用方案
    └── → STM32MP157 (资料最多，社区最大)
```

---

## 🔮 六、2025-2026 异构架构趋势

### 6.1 从双核到三核/四核异构

不再只是 A+M 双核，越来越多芯片采用 **A + M + R + DSP** 或 **A + M + NPU** 组合：
- i.MX 95：A55 + M7 + M33（三种 ARM 核！）
- T113-i：ARM + RISC-V + DSP
- RZ/V2H：A55 + M33 + R8×2 + DRP-AI

### 6.2 M 核先启动成为新趋势

STM32MP2 的 M33-TD 模式可能引领潮流。越来越多的场景需要：
- 设备上电即工作（不等 Linux 启动）
- A 核故障不影响安全功能
- MCU 常驻 + MPU 按需

### 6.3 RISC-V 从核遍地开花

玄铁 C906/E907 作为从核出现在大量国产芯片中，跨 ISA 的 AMP（ARM Linux + RISC-V RTOS）成为常态。

### 6.4 Zephyr RTOS 强势崛起

NXP、Toradex、ST 都在推动 Zephyr 作为 M/R 核首选 RTOS，替代 FreeRTOS 的趋势明显。

### 6.5 OpenAMP 标准化持续演进

2025 年关键增强：
- Address Translation（不同核看到不同物理地址时自动转换）
- Vendor Resource Extensions（厂商自定义资源）
- Hypervisor 支持（Jailhouse, Xen）

---

## 📚 七、学习路径建议

```
入门 (1-2周)
  ├─ 理论：理解 AMP 概念、OpenAMP 四层模型
  ├─ 动手：STM32MP157 + 官方 OpenAMP 示例
  └─ 目标：跑通 echo_test

进阶 (2-4周)  
  ├─ 深入：读懂 Resource Table、设备树 reserved-memory
  ├─ 动手：自定义 RPMsg 通道，实现双向数据传输
  ├─ 调试：用逻辑分析仪观察 IPCC 中断时序
  └─ 目标：从零搭建自己的 AMP 通信系统

高级 (4-8周)
  ├─ 深水区：Cache 一致性、零拷贝优化、实时性测量
  ├─ 动手：M 核实时控制电机 + A 核 ROS2 导航
  ├─ 扩展：尝试 RISC-V 从核（全志 T113/D1）
  └─ 目标：设计一个多核实时系统架构
```

---

## 📖 参考资源

| 资源 | 链接 |
|------|------|
| OpenAMP 官方文档 | https://openamp.readthedocs.io/ |
| OpenAMP GitHub | https://github.com/OpenAMP/open-amp |
| libmetal | https://github.com/OpenAMP/libmetal |
| ST Wiki - Cortex-M 协处理器管理 | https://wiki.st.com/stm32mpu/wiki/Cortex-M_coprocessor_management_overview |
| NXP AN14120 (i.MX8M Plus M7开发) | NXP 官网搜索 |
| Xilinx UG1186 (OpenAMP 用户指南) | AMD 官网 |
| 全志 T113 OpenAMP (米尔电子) | https://www.myirtech.com/ |
| 全志 V85x E907 RTOS SDK | Yuzuki Lizard GitHub |
| RT-Thread RISC-V AMP | https://www.rt-thread.org/ |

---

> 💡 **一句话总结：** 异构 AMP = A 核做"大脑"（Linux，复杂决策）+ M/R 核做"小脑"（RTOS，实时反射）+ OpenAMP 做"神经"（通信）。这是嵌入式 Linux 的高级技能，也是你区别于普通嵌入式工程师的核心竞争力。
