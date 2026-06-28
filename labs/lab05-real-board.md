# Lab 5：STM32MP157 真机启动

> **目标：** 在真实开发板上从 SD 卡启动自己构建的嵌入式 Linux
>
> **时间：** 3-5 天
>
> **硬件：** STM32MP157F-DK2（约 ¥800-1200）

---

## 🛒 采购清单

| 物品 | 说明 | 参考价格 |
|------|------|---------|
| STM32MP157F-DK2 | 官方 Discovery Kit | ¥800-1200 |
| MicroSD 卡 | 16GB+, Class 10/U1 | ¥30 |
| USB-C 线 | 供电 | 随板附赠 |
| （可选）USB-TTL | CH340 模块 | ¥10 |

板子到手后插上 USB-C（通过 ST-LINK 接口），系统即上电。

---

## Step 1：了解你的板子

### STM32MP157F-DK2 硬件资源

```
                           STM32MP157F-DK2
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ┌─────────────────────────────────┐   ┌──────────────┐ │
│  │   STM32MP157F                   │   │  PMIC        │ │
│  │   ├── Cortex-A7 ×2 @ 800MHz     │   │  STPMIC1     │ │
│  │   ├── Cortex-M4   @ 200MHz      │   └──────────────┘ │
│  │   ├── GPU: Vivante GC2000       │                    │
│  │   └── 3D GPU @ 533MHz          │   ┌──────────────┐ │
│  └─────────────────────────────────┘   │  512MB DDR3  │ │
│                                        └──────────────┘ │
│  ┌───────────┐  ┌──────────┐  ┌─────────────────────┐   │
│  │ ST-LINK   │  │ 4" LCD   │  │ Ethernet (RJ45)      │   │
│  │ (JTAG+    │  │ Touch    │  │ USB-C OTG            │   │
│  │  UART)    │  │ 480×800  │  │ 2× Arduino Header    │   │
│  └───────────┘  └──────────┘  │ 40-pin RPi Header    │   │
│                               │ Audio Codec           │   │
│  ┌──────────────────────────┐ │ microSD Slot          │   │
│  │  Boot Mode Switch (SW1)  │ └─────────────────────┘   │
│  │  SD 卡启动: SW1=1 0 0     │                          │
│  └──────────────────────────┘                           │
└─────────────────────────────────────────────────────────┘
```

---

## Step 2：下载 STM32MP1 官方工具

### 2.1 STM32CubeProgrammer

用于烧录镜像到 SD 卡 / eMMC：

```bash
# 官网下载 Linux 版本
# https://www.st.com/en/development-tools/stm32cubeprog.html
# 解压后
unzip en.stm32cubeprg-lin.zip
cd SetupSTM32CubeProgrammer-*
chmod +x SetupSTM32CubeProgrammer-*.linux
./SetupSTM32CubeProgrammer-*.linux
```

安装完成后：
```bash
export PATH=$PATH:$HOME/STMicroelectronics/STM32Cube/STM32CubeProgrammer/bin
STM32_Programmer_CLI --version
```

### 2.2 下载 STM32MP1 官方 BSP

```bash
cd ~/embedded-linux-lab
mkdir stm32mp1 && cd stm32mp1

# ST 官方 OpenSTLinux BSP 下载页面
# https://www.st.com/en/embedded-software/stm32mp1dev.html
# 或者直接从 ST 的 GitHub 获取
git clone https://github.com/STMicroelectronics/meta-st-stm32mp.git
```

---

## Step 3：用 Buildroot 构建你自己的系统（更快的方法）

```bash
cd ~/embedded-linux-lab/buildroot-2025.02

# STM32MP157 有官方 defconfig
make stm32mp157f_dk2_defconfig

make menuconfig
# 检查：
# Target options → ARM (little endian) → Cortex-A7
# 确认工具链、内核版本

make -j$(nproc)
```

**产物位置：**
```
output/images/
├── stm32mp157f-dk2.dtb            # 设备树
├── zImage                         # 内核
├── u-boot.stm32                   # U-Boot (STM32 格式)
├── tf-a.stm32                     # TF-A (ARM 可信固件)
├── rootfs.ext4                    # 根文件系统
├── sdcard.img                     # 🎉 SD 卡完整镜像！
└── ...
```

---

## Step 4：烧录到 SD 卡

### 方法一：直接写 sdcard.img（推荐）

```bash
# 插入 SD 卡，确认设备名
lsblk
# 假设 SD 卡是 /dev/sdb

# ⚠️ 确认设备名正确！写错会毁掉你的硬盘！

# 烧录
sudo dd if=output/images/sdcard.img of=/dev/sdb bs=8M status=progress conv=fdatasync

# 等待写入完成，拔出 SD 卡
```

### 方法二：用 STM32CubeProgrammer

```bash
STM32_Programmer_CLI -c port=usb1 -w output/images/sdcard.img
```

### 方法三：手动分区（了解底层）

```bash
# 1. 分区
sudo fdisk /dev/sdb
#   o (新建 DOS 分区表)
#   n p 1 2048 +128M  (boot 分区, FAT)
#   n p 2 ... ...      (rootfs 分区, ext4)
#   t 1 c              (类型改为 W95 FAT32)
#   w (写入)

# 2. 格式化
sudo mkfs.vfat /dev/sdb1
sudo mkfs.ext4 /dev/sdb2

# 3. 挂载并复制文件
sudo mount /dev/sdb1 /mnt/boot
sudo cp output/images/tf-a.stm32 /mnt/boot/
sudo cp output/images/u-boot.stm32 /mnt/boot/
sudo cp output/images/zImage /mnt/boot/
sudo cp output/images/stm32mp157f-dk2.dtb /mnt/boot/
sudo umount /mnt/boot

sudo mount /dev/sdb2 /mnt/rootfs
sudo tar xf output/images/rootfs.tar -C /mnt/rootfs/
sudo umount /mnt/rootfs
```

---

## Step 5：启动！

### 5.1 硬件连接

```
1. 设置 SW1 启动开关为 SD 卡启动：1=ON, 2=OFF, 3=OFF
2. 插入 SD 卡
3. USB-C 线连接 ① ST-LINK 口 到电脑（供电 + 调试串口）
4. （可选）连接网线
```

### 5.2 串口连接

```bash
# ST-LINK 会虚拟一个串口
ls /dev/ttyACM*
# /dev/ttyACM0

# 连接
picocom -b 115200 /dev/ttyACM0
# 或
minicom -D /dev/ttyACM0 -b 115200
```

### 5.3 应该看到的启动日志

```
NOTICE:  CPU: STM32MP157FAC Rev.Z
NOTICE:  Model: STMicroelectronics STM32MP157F-DK2 Discovery Board
NOTICE:  BL2: v2.8.0 (debug): ...

U-Boot 2024.01-stm32mp-...

Hit any key to stop autoboot:  0

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 6.6.0 ...
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7)
[    0.000000] Machine model: STMicroelectronics STM32MP157F-DK2 Discovery Board
...
[    1.234567] Freeing unused kernel memory: 1024K
Starting init: /sbin/init

Welcome to Buildroot!
stm32mp1 login: root
#
```

---

## Step 6：在板子上运行你的驱动

```bash
# 复制 Lab 4 的驱动到 SD 卡 rootfs 中
# 重新制作 rootfs 或通过 scp 复制

# 在板子上
insmod /root/modules/mybuf.ko
ls -l /dev/mybuf*
# crw------- 1 root root 248, 0 ... /dev/mybuf0
# crw------- 1 root root 248, 1 ... /dev/mybuf1

# 运行测试程序
/root/test_mybuf
# ✅ 和 QEMU 中表现完全一样！
```

---

## Step 7：点亮一个 LED！⚡

STM32MP157F-DK2 的 LED 连在 GPIO 上，可以通过 sysfs 控制：

```bash
# 板子上有一个蓝色的用户 LED (LD7, 接在 PA13 上)

# 导出 GPIO
echo 13 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio13/direction

# 点亮
echo 1 > /sys/class/gpio/gpio13/value
sleep 1
# 熄灭
echo 0 > /sys/class/gpio/gpio13/value

# 闪烁脚本
while true; do
    echo 1 > /sys/class/gpio/gpio13/value
    sleep 0.5
    echo 0 > /sys/class/gpio/gpio13/value
    sleep 0.5
done
```

---

## 🎉 恭喜！

你已经有了一块正在运行你自己构建的 Linux 的 ARM 开发板。

---

## ⏭️ 下一步

这块板子还有一个 **Cortex-M4** 核心没有利用。接下来让它和 Linux（A7）通过 OpenAMP 通信。

➡️ [Lab 6: A7 + M4 异构多核通信](./lab06-amp-openamp.md)
