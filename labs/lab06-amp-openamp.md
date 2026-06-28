# Lab 6：A7 + M4 异构多核通信 (OpenAMP)

> **目标：** 让 Cortex-M4 和 Cortex-A7 通过 OpenAMP/RPMsg 通信
>
> **时间：** 5-7 天
>
> **硬件：** STM32MP157F-DK2
>
> **先修：** Lab 5（能启动自己的 Linux）、了解 FreeRTOS 基础概念

---

## 架构总览

```
┌───────────────────────────────────────────────────────┐
│              STM32MP157F                                │
│                                                       │
│  Cortex-A7 (Linux)          Cortex-M4 (FreeRTOS/裸机)  │
│  ┌──────────────────┐       ┌─────────────────────┐   │
│  │ User App         │       │ 实时控制任务          │   │
│  │   open("/dev/    │       │  sensor_read()       │   │
│  │     ttyRPMSG0")  │       │  motor_control()     │   │
│  │   read/write     │       │  alarm_monitor()     │   │
│  ├──────────────────┤       ├─────────────────────┤   │
│  │ RPMsg 驱动        │  IPCC  │ OpenAMP 库           │   │
│  │   remoteproc     │◄══════►│   RPMsg Remote      │   │
│  │   rpmsg_tty      │ 中断   │   VirtIO Transport  │   │
│  ├──────────────────┤       ├─────────────────────┤   │
│  │ Linux 内核        │       │ FreeRTOS / 裸机      │   │
│  └──────────────────┘       └─────────────────────┘   │
│                                                       │
│          共享内存 (MCU SRAM / DDR 预留区域)              │
└───────────────────────────────────────────────────────┘
```

---

## Step 1：理解启动流程

在 STM32MP157 上，M4 有两种启动方式：

### 方式 A：A7 先启动 → 再加载 M4（本 Lab 用这个）

```
系统上电
  → TF-A (A7) → U-Boot → Linux
                            └→ remoteproc 加载 M4 固件
                              └→ M4 开始运行 → OpenAMP 建立通信
```

### 方式 B：M4 先启动（STM32CubeMX 中配置）

```
系统上电
  → M4 先跑（RTOS 控制电机/传感器）
  → 需要 Linux 时才唤醒 A7
  → 更复杂，本 Lab 不涉及
```

---

## Step 2：获取 STM32CubeMP1 和示例

```bash
cd ~/embedded-linux-lab/stm32mp1

# 官方 M4 固件示例仓库
git clone https://github.com/STMicroelectronics/STM32CubeMP1.git

cd STM32CubeMP1
# OpenAMP 示例在 Projects/STM32MP157F-DK2/Applications/OpenAMP/
ls Projects/STM32MP157F-DK2/Applications/OpenAMP/
# OpenAMP_TTY_echo/
# OpenAMP_raw/
```

---

## Step 3：M4 端固件（OpenAMP Echo 示例）

### 3.1 关键代码结构

```c
// M4 端核心流程 (简化版)

#include "openamp.h"

// RPMsg 端点回调 — A7 发来数据时被调用
static int rpmsg_rx_callback(struct rpmsg_endpoint *ept,
                              void *data, size_t len,
                              uint32_t src, void *priv)
{
    // 收到 A7 的数据，原样回传（echo）
    rpmsg_send(ept, data, len);
    return RPMSG_SUCCESS;
}

void main(void)
{
    // 1. 硬件初始化
    HAL_Init();
    SystemClock_Config();
    MX_IPCC_Init();         // 初始化 IPCC (Inter-Processor Communication Controller)
    MX_GPIO_Init();         // LED 等外设

    // 2. 初始化 OpenAMP
    MX_OpenAMP_Init(RPMSG_REMOTE, NULL);

    // 3. 创建 RPMsg 端点（创建一个通信通道）
    struct rpmsg_endpoint *ept;
    rpmsg_create_ept(&ept, "rpmsg-tty", RPMSG_ADDR_ANY,
                     RPMSG_ADDR_ANY, rpmsg_rx_callback, NULL);

    // 4. 主循环
    while (1) {
        // 定期处理 OpenAMP 消息
        OPENAMP_check_for_message();

        // 你的实时任务可以放在这里
        // 例如：读取传感器、控制电机、检测报警等
    }
}
```

### 3.2 编译 M4 固件

```bash
# 用 STM32CubeIDE 或 Makefile 编译
# 产物: OpenAMP_TTY_echo.elf

# 或者用 arm-none-eabi 工具链
sudo apt install gcc-arm-none-eabi
cd STM32CubeMP1/Projects/STM32MP157F-DK2/Applications/OpenAMP/OpenAMP_TTY_echo

# 编译
make -j$(nproc)
# 产物: build/OpenAMP_TTY_echo.elf
```

---

## Step 4：Linux 端配置

### 4.1 内核配置（Buildroot 中）

```bash
cd ~/embedded-linux-lab/buildroot-2025.02
make linux-menuconfig

# 必须启用的内核选项：
# Device Drivers --->
#   Remoteproc drivers --->
#     <*> STM32 remoteproc support
#   Rpmsg drivers --->
#     <*> RPMSG device interface
#     <*> RPMSG TTY driver
#   Mailbox Hardware Support --->
#     <*> STM32 IPCC mailbox

make -j$(nproc)
```

### 4.2 设备树配置

在 Buildroot 的设备树覆盖文件中添加 M4 和 IPCC 配置：

```dts
// 添加到 output/build/linux-*/arch/arm/boot/dts/stm32mp157f-dk2.dts
// 或通过 Buildroot 的 device tree overlay 机制

/ {
    reserved-memory {
        /* 为 M4 固件和 RPMsg vring 预留内存 */
        m4_fw_region: m4-fw@10000000 {
            reg = <0x10000000 0x40000>;   /* 256KB for M4 firmware */
            no-map;
        };
        m4_vdev0vring0: vdev0vring0@10040000 {
            reg = <0x10040000 0x1000>;
            no-map;
        };
        m4_vdev0vring1: vdev0vring1@10041000 {
            reg = <0x10041000 0x1000>;
            no-map;
        };
        m4_vdev0buffer: vdev0buffer@10042000 {
            reg = <0x10042000 0x100000>;   /* 1MB shared buffer */
            no-map;
        };
    };

    /* M4 remoteproc 节点 */
    m4_rproc: m4@10000000 {
        compatible = "st,stm32mp1-rproc";
        memory-region = <&m4_fw_region>, <&m4_vdev0buffer>,
                        <&m4_vdev0vring0>, <&m4_vdev0vring1>;
        mboxes = <&ipcc 0>, <&ipcc 1>;
        mbox-names = "vq0", "vq1";
    };
};
```

---

## Step 5：部署和运行

### 5.1 复制固件到板子

```bash
# 将 M4 固件复制到 Linux 能访问的位置
scp OpenAMP_TTY_echo.elf root@<board_ip>:/lib/firmware/
```

### 5.2 启动 M4 并建立通信

```bash
# 在板子的 Linux 中

# 1. 加载内核驱动
modprobe rpmsg_tty
modprobe stm32_rproc

# 2. 指定要加载的固件
echo OpenAMP_TTY_echo.elf > /sys/class/remoteproc/remoteproc0/firmware

# 3. 启动 M4 核心!
echo start > /sys/class/remoteproc/remoteproc0/state

# 4. 查看状态
cat /sys/class/remoteproc/remoteproc0/state
# running

# 5. 检查通信通道
dmesg | grep rpmsg
# virtio_rpmsg_bus virtio0: creating channel rpmsg-tty addr 0x400
# rpmsg_tty virtio0.rpmsg-tty.-1.0: new channel

# 6. 通信！向 M4 发数据并接收回显
echo "Hello M4!" > /dev/ttyRPMSG0
cat /dev/ttyRPMSG0
# Hello M4!    ← M4 回显的消息
```

---

## Step 6：实战 — 双向数据通信

### M4 端添加传感器模拟

```c
// M4 端：每 500ms 向 A7 发送模拟的温度数据
static void sensor_task(void *arg)
{
    struct rpmsg_endpoint *ept = (struct rpmsg_endpoint *)arg;
    char buf[64];
    float temp = 25.0;

    while (1) {
        // 模拟温度变化
        temp += ((float)(rand() % 100) / 100.0 - 0.5);

        snprintf(buf, sizeof(buf), "TEMP:%.1f", temp);
        rpmsg_send(ept, buf, strlen(buf) + 1);

        vTaskDelay(500 / portTICK_PERIOD_MS);
    }
}
```

### A7 Linux 端读取

```bash
# 在板子上持续读取
cat /dev/ttyRPMSG0
# TEMP:25.3
# TEMP:25.1
# TEMP:25.8
# TEMP:26.2
# ...
```

---

## 💡 核心概念回顾

| 组件 | 作用 | 在芯片上的物理位置 |
|------|------|-------------------|
| **IPCC** | 核间中断控制器，通知对方"有消息" | STM32MP157 的内置外设 |
| **共享内存** | 数据交换的"黑板" | DDR 或 MCU SRAM 中预留区域 |
| **remoteproc** | Linux 端管理 M4 的生命周期 | Linux 内核驱动 |
| **RPMsg** | 基于共享内存的消息传递协议 | Linux 驱动 + M4 库 |
| **VirtIO vring** | 环形缓冲区，实现无锁队列 | 共享内存中 |

### 通信延迟参考：

| 场景 | 典型延迟 |
|------|---------|
| A7 → M4 单次消息 | ~50-100μs |
| M4 → A7 单次消息 | ~50-100μs |
| 双向 RTT (echo) | ~100-200μs |
| M4 中断响应 | ~100ns |

---

## ⏭️ 下一步

A7+M4 通信跑通后，你就可以做更复杂的综合项目了。

➡️ [Lab 7: 综合项目](./lab07-final-project.md)
