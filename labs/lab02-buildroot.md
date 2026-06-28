# Lab 2：Buildroot 自动化构建

> **目标：** 用 Buildroot 一键生成完整的嵌入式 Linux 系统（工具链+内核+rootfs）
>
> **时间：** 1-2 天（首次编译 ~40分钟）
>
> **学到的概念：** Buildroot 架构、交叉工具链自动构建、defconfig、包管理

---

## 🤔 Buildroot 是什么？

```
手工方式 (Lab 1):                     Buildroot 方式 (Lab 2):
                                   
下载内核源码 → 配置 → 编译            make qemu_arm_vexpress_defconfig
下载 BusyBox → 配置 → 编译      vs    make menuconfig    (微调)
手工建目录 → 手工做镜像 → 启动          make -j$(nproc)    (全自动)
                                    ./output/images/start-qemu.sh
😫 30+ 步操作                        🤩 3 步搞定
```

Buildroot 帮你管理：
1. **交叉编译工具链**（自动下载/构建）
2. **Linux 内核**（下载、配置、编译）
3. **根文件系统**（BusyBox + 你选的软件包）
4. **启动脚本**（QEMU 一键启动）

---

## Step 1：获取 Buildroot

```bash
cd ~/embedded-linux-lab

git clone --depth=1 --branch 2025.02.x \
    https://gitlab.com/buildroot.org/buildroot.git buildroot-2025.02

cd buildroot-2025.02
```

> 📦 国内网络慢？用镜像：
> `git clone --depth=1 --branch 2025.02.x https://gitee.com/mirrors/buildroot.git`

---

## Step 2：配置

```bash
# 查看 QEMU 相关的预设配置
ls configs/ | grep qemu
# qemu_aarch64_ebbr_defconfig
# qemu_aarch64_sbsa_defconfig
# qemu_aarch64_virt_defconfig
# qemu_arm_vexpress_defconfig       ← 我们要用的
# qemu_mips32r2el_malta_defconfig
# qemu_riscv64_virt_defconfig
# ...

# 使用 ARM vexpress 配置（和 Lab 1 同一个平台）
make qemu_arm_vexpress_defconfig

# 微调配置
make menuconfig
```

### menuconfig 中建议的修改

```
Toolchain --->
    *** 选择 glibc 而不是 uClibc（更好的兼容性） ***
    C library (glibc)

System configuration --->
    *** 设置 root 密码 ***
    (myroot123) Root password
    *** 给系统起个名字 ***
    (my-embedded) System hostname

Target packages --->
    *** 添加一些有用的工具 ***
    Networking applications --->
        [*] openssh          ← SSH 远程登录
        [*]   openssh-server
    Debugging, profiling and benchmark --->
        [*] strace           ← 调试利器
        [*] ltrace
```

---

## Step 3：构建

```bash
# ⚠️ 首次编译会下载源码包，耗时 20-40 分钟
# 确保网络通畅！

make -j$(nproc)
```

**Buildroot 会自动：**
1. 下载并构建交叉工具链（arm-linux-gnueabihf）
2. 下载 Linux 内核源码 → 配置 → 编译
3. 下载 BusyBox → 配置 → 编译
4. 下载你选的所有软件包 → 编译
5. 制作根文件系统镜像
6. 生成 QEMU 启动脚本

**产物在 `output/images/`：**
```
output/images/
├── zImage                       # 内核镜像
├── vexpress-v2p-ca9.dtb        # 设备树
├── rootfs.ext2                 # 根文件系统镜像
├── start-qemu.sh               # 启动脚本 (一键启动!)
└── ...
```

---

## Step 4：启动

```bash
# Buildroot 已经帮你生成好了启动脚本
./output/images/start-qemu.sh
```

**或者手动启动：**
```bash
qemu-system-arm \
    -M vexpress-a9 \
    -m 256M \
    -kernel output/images/zImage \
    -dtb output/images/vexpress-v2p-ca9.dtb \
    -drive file=output/images/rootfs.ext2,if=sd,format=raw \
    -append "console=ttyAMA0,115200 root=/dev/mmcblk0 rw" \
    -net nic,model=lan9118 \
    -net user,hostfwd=tcp::2222-:22 \
    -nographic
```

**用 SSH 登录（如果你在 menuconfig 中启用了 openssh）：**
```bash
# 从宿主机 SSH 到 QEMU
ssh -p 2222 root@localhost
# 输入你设置的 root 密码
```

---

## Step 5：探索 Buildroot 的工程结构

```
buildroot-2025.02/
├── Config.in              # 顶层配置入口
├── Makefile               # 顶层 Makefile
├── package/               # 软件包描述（每个包一个目录）
│   ├── busybox/
│   │   ├── Config.in      # 包的配置选项
│   │   └── busybox.mk     # 包的 Makefile（下载/编译/安装逻辑）
│   ├── openssh/
│   └── ...
├── configs/               # 预设配置文件
│   └── qemu_arm_vexpress_defconfig
├── board/                 # 板级支持
│   └── qemu/
├── output/
│   ├── build/             # 各源码包的构建目录
│   ├── host/              # 主机工具（交叉工具链在这里）
│   ├── target/            # 目标文件系统（未打包的 rootfs）
│   └── images/            # 最终产物
└── dl/                    # 下载的源码包缓存
```

---

## Step 6：添加你自己的程序

### 6.1 创建你自己的 package

```bash
cd ~/embedded-linux-lab/buildroot-2025.02
mkdir -p package/myhello
```

```bash
# package/myhello/Config.in
cat > package/myhello/Config.in << 'EOF'
config BR2_PACKAGE_MYHELLO
    bool "myhello"
    help
        A simple hello world application for learning Buildroot.
EOF
```

```bash
# package/myhello/myhello.mk
cat > package/myhello/myhello.mk << 'EOF'
################################################################################
#
# myhello
#
################################################################################

MYHELLO_VERSION = 1.0
MYHELLO_SITE = ./package/myhello/src
MYHELLO_SITE_METHOD = local

define MYHELLO_BUILD_CMDS
    $(TARGET_CC) $(@D)/hello.c -o $(@D)/hello
endef

define MYHELLO_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/hello $(TARGET_DIR)/usr/bin/hello
endef

$(eval $(generic-package))
EOF
```

```bash
# package/myhello/src/hello.c
mkdir -p package/myhello/src
cat > package/myhello/src/hello.c << 'EOF'
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Hello from my custom Buildroot package!\n");
    printf("This is embedded Linux!\n");
    return 0;
}
EOF
```

### 6.2 注册到 Buildroot

在 `package/Config.in` 中添加一行（放在末尾）：

```bash
echo 'menu "My Custom Packages"' >> package/Config.in
echo 'source "package/myhello/Config.in"' >> package/Config.in
echo 'endmenu' >> package/Config.in
```

### 6.3 构建并测试

```bash
make menuconfig
# Target packages → My Custom Packages → [*] myhello

make -j$(nproc)

# 启动后在 QEMU 中运行
./output/images/start-qemu.sh

/ # hello
Hello from my custom Buildroot package!
This is embedded Linux!
```

---

## 📊 Buildroot vs 手工构建

| | 手工方式 (Lab 1) | Buildroot (Lab 2) |
|------|----------------|-------------------|
| 学习价值 | ⭐⭐⭐⭐⭐ 理解每个细节 | ⭐⭐⭐ 理解自动化原理 |
| 效率 | ⭐ 慢，重复劳动 | ⭐⭐⭐⭐⭐ 一键生成 |
| 可复现性 | ⭐⭐ 容易出错 | ⭐⭐⭐⭐⭐ .config 文件即可 |
| 适合场景 | 学习原理 | 实际项目开发 |

---

## 💡 关键概念

| Buildroot 概念 | 对应实际意义 |
|----------------|------------|
| `defconfig` | 一个项目的基础配置（版本可控） |
| `output/build/` | 每个软件包的独立构建目录 |
| `output/target/` | 组装中的根文件系统 |
| `package/*/*.mk` | 描述如何下载/编译/安装一个包 |
| `board/*/` | 板级特定的补丁和配置 |

---

## ⏭️ 下一步

有了 Buildroot 这个高效的构建系统，接下来学习内核模块——这是嵌入式 Linux 驱动开发的基础。

➡️ [Lab 3: 内核模块 HelloWorld](./lab03-kernel-module.md)
