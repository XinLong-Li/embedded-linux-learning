# Lab 0：环境搭建

> **目标：** 搭好所有工具，验证交叉编译链可用
>
> **时间：** ~2 小时
>
> **系统要求：** Ubuntu 22.04 / 24.04（实体机或虚拟机）

---

## 1. 宿主机要求

| 项目 | 最低配置 | 推荐配置 |
|------|---------|---------|
| CPU | 2 核 | 4+ 核 |
| RAM | 4GB | 8GB+ |
| 磁盘 | 40GB | 100GB (编译内核/Buildroot 很吃空间) |
| 系统 | Ubuntu 22.04 | Ubuntu 24.04 |
| 虚拟化 | VirtualBox/VMware | 双系统最佳 |

> ⚠️ **不推荐 WSL** — 涉及 USB 设备、串口、交叉编译时会有各种奇怪问题

---

## 2. 一键安装脚本

```bash
#!/bin/bash
# 文件名: setup-tools.sh
# 运行: bash setup-tools.sh

set -e

echo "=== 更新软件源 ==="
sudo apt update

echo "=== 基础开发工具 ==="
sudo apt install -y build-essential gcc g++ gdb make cmake
sudo apt install -y git vim curl wget tree htop
sudo apt install -y flex bison bc rsync cpio unzip
sudo apt install -y python3 python3-pip

echo "=== 交叉编译工具链 ==="
# ARM 32位 (Cortex-A7/A9)
sudo apt install -y gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
# ARM 64位 (Cortex-A53/A55)
sudo apt install -y gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu

echo "=== QEMU 模拟器 ==="
sudo apt install -y qemu-system-arm qemu-system-aarch64 qemu-user-static

echo "=== 内核编译依赖 ==="
sudo apt install -y libssl-dev libncurses-dev libelf-dev dwarves
sudo apt install -y device-tree-compiler

echo "=== 串口工具 ==="
sudo apt install -y picocom minicom

echo "=== Git 配置 ==="
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

echo ""
echo "=== 验证安装 ==="

echo -n "ARM32 交叉编译器: "
arm-linux-gnueabihf-gcc --version | head -1 || echo "❌ 未安装"

echo -n "ARM64 交叉编译器: "
aarch64-linux-gnu-gcc --version | head -1 || echo "❌ 未安装"

echo -n "QEMU ARM: "
qemu-system-arm --version | head -1 || echo "❌ 未安装"

echo -n "QEMU AArch64: "
qemu-system-aarch64 --version | head -1 || echo "❌ 未安装"

echo -n "dtc: "
dtc --version | head -1 || echo "❌ 未安装"

echo ""
echo "=== 全部完成！ ==="
```

---

## 3. 验证交叉编译

```bash
# 创建测试文件
cat > hello.c << 'EOF'
#include <stdio.h>
int main() {
    printf("Hello from ARM!\n");
    return 0;
}
EOF

# ARM32 编译
arm-linux-gnueabihf-gcc -static hello.c -o hello_arm32
file hello_arm32
# 输出: hello_arm32: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked...

# ARM64 编译
aarch64-linux-gnu-gcc -static hello.c -o hello_arm64
file hello_arm64
# 输出: hello_arm64: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked...

# 用 QEMU 运行 ARM32 程序
qemu-arm ./hello_arm32
# 输出: Hello from ARM!

# 用 QEMU 运行 ARM64 程序
qemu-aarch64 ./hello_arm64
# 输出: Hello from ARM!
```

---

## 4. 创建工作目录

```bash
mkdir -p ~/embedded-linux-lab
cd ~/embedded-linux-lab
mkdir -p {kernel,buildroot,rootfs,modules,scripts,output}
tree -L 2
```

预期结构:
```
~/embedded-linux-lab/
├── kernel/          # 内核源码
├── buildroot/       # Buildroot
├── rootfs/          # 手工构建的根文件系统
├── modules/         # 内核模块源码
├── scripts/         # 启动/构建脚本
└── output/          # 编译产物
```

---

## 5. 可选：配置开发环境加速

### 用国内镜像源（如果 apt 慢）

```bash
sudo sed -i 's/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sudo apt update
```

### 配置 ccache（加速重复编译）

```bash
sudo apt install -y ccache
echo 'export PATH="/usr/lib/ccache:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 安装 VS Code 插件（推荐）

- **C/C++** (Microsoft)
- **DeviceTree** (Trung Le)
- **ARM Assembly** (dan-c-underwood)
- **Remote - SSH** (如果远程开发)

---

## 6. 检查清单

完成以下所有项后才能进入 Lab 1：

- [ ] `arm-linux-gnueabihf-gcc --version` 正常输出
- [ ] `aarch64-linux-gnu-gcc --version` 正常输出
- [ ] `qemu-system-arm -M help | grep vexpress` 有输出
- [ ] 交叉编译的 `hello_arm32` 能用 `qemu-arm` 运行
- [ ] 工作目录 `~/embedded-linux-lab/` 已创建
- [ ] `git` 可用

---

**完成本 Lab 后，进入 [Lab 1: QEMU 最小嵌入式 Linux 系统](./lab01-qemu-minimal.md)**
