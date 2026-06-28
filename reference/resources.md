# 📚 资源汇总

> 按阶段整理推荐资源，标 ⭐ 的为必读/首选

---

## 📖 书籍推荐

### Linux 基础
| 书名 | 说明 | 阶段 |
|------|------|------|
| ⭐ 《鸟哥的 Linux 私房菜》第四版 | Linux 入门经典，适合边看边操作 | 阶段一 |
| 《The Linux Command Line》 | 免费电子书，命令行为主线 | 阶段一 |

### C 语言
| 书名 | 说明 | 阶段 |
|------|------|------|
| ⭐ 《C 程序设计语言》(K&R) | C 语言圣经，常读常新 | 阶段二 |
| ⭐ 《C 和指针》 | 指针讲得最透的书 | 阶段二 |
| 《C 专家编程》 | 进阶必备，幽默深入 | 阶段二 |
| 《C 陷阱与缺陷》 | 薄而精，避坑指南 | 阶段二 |

### 计算机系统 / 操作系统
| 书名 | 说明 | 阶段 |
|------|------|------|
| ⭐ 《深入理解计算机系统》(CS:APP) | 程序员必读，建立系统观 | 阶段二/三 |
| ⭐ 《操作系统概念》(恐龙书) | 操作系统标准教材 | 阶段三 |
| 《现代操作系统》(Tanenbaum) | 另一本经典 OS 教材 | 阶段三 |

### Linux 内核
| 书名 | 说明 | 阶段 |
|------|------|------|
| ⭐ 《Linux 内核设计与实现》(Robert Love) | 内核入门最佳读物，短小精悍 | 阶段四 |
| ⭐ 《Linux 设备驱动程序》(LDD3) | 驱动开发经典，思想永不过时 | 阶段四 |
| 《深入 Linux 内核架构》 | 大部头，深入细节，进阶读 | 阶段四+ |
| 《Linux 内核观测技术 BPF》 | 现代内核调试/观测 | 阶段四+ |

### 嵌入式 Linux
| 书名 | 说明 | 阶段 |
|------|------|------|
| ⭐ 《Mastering Embedded Linux Programming》 | 嵌入式 Linux 最佳入门书 | 阶段五 |
| 《Embedded Linux Primer》 | 另一本优秀入门书 | 阶段五 |
| 《Building Embedded Linux Systems》 | 经典但略旧，第二版仍可参考 | 阶段五 |
| 《Exploring BeagleBone》 | 如果选 BeagleBone 开发板 | 阶段五 |

---

## 🌐 在线资源

### 教程与文档
| 资源 | 链接 | 说明 |
|------|------|------|
| ⭐ Linux 内核官方文档 | https://docs.kernel.org/ | 内核文档，日益完善 |
| ⭐ Bootlin 培训资料 | https://bootlin.com/docs/ | 免费的内核/嵌入式Linux培训幻灯片 |
| LKMPG | https://sysprog21.github.io/lkmpg/ | 内核模块编程指南，现代版 |
| eLinux.org | https://elinux.org/ | 嵌入式 Linux 百科全书 |
| OSDev Wiki | https://wiki.osdev.org/ | 写自己的 OS 圣经 |
| Buildroot Manual | https://buildroot.org/downloads/manual/manual.html | Buildroot 官方手册 |
| Linux From Scratch | https://linuxfromscratch.org/ | 从零构建 Linux 发行版 |
| Device Tree 规格 | https://devicetree-specification.readthedocs.io/ | 设备树官方规范 |

### 视频课程
| 课程 | 平台 | 说明 |
|------|------|------|
| 正点原子/野火 嵌入式 Linux 教程 | Bilibili | 中文视频，配合开发板，讲得细致 |
| 韦东山 嵌入式 Linux 教程 | Bilibili | 经典中文嵌入式 Linux 教程 |
| CS 162 (UC Berkeley) | YouTube | 操作系统课程，有实验 |
| Linux Kernel Programming (Free Electrons/Bootlin) | YouTube | 内核编程教程 |
| 6.S081 (MIT) | YouTube | MIT 操作系统工程，xv6 + risc-v |

### 实战练习
| 资源 | 链接 | 说明 |
|------|------|------|
| Bandit | https://overthewire.org/wargames/bandit/ | Linux 命令行闯关游戏 |
| LabEx | https://labex.io/ | Linux/C 在线实验环境 |
| The Eudyptula Challenge | (已关闭但题可搜到) | 内核编程挑战题 |
| Linux Kernel Selftests | 内核自带 | `tools/testing/selftests/` |

---

## 🛠️ 工具列表

### 必装工具
```bash
# 基础开发工具
sudo apt install build-essential gcc g++ gdb make cmake
sudo apt install git vim cscope ctags
sudo apt install valgrind cppcheck  # 内存检查/静态分析

# 交叉编译工具链（ARM 32位）
sudo apt install gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
# 交叉编译工具链（ARM 64位）
sudo apt install gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu

# 嵌入式 Linux 构建依赖
sudo apt install build-essential bc bison flex libssl-dev
sudo apt install libncurses-dev  # menuconfig 需要

# 调试/分析
sudo apt install strace ltrace
sudo apt install qemu-system-arm  # ARM 模拟器
sudo apt install minicom picocom   # 串口终端
sudo apt install device-tree-compiler  # dtc
```

### 内核开发
```bash
# 内核编译依赖
sudo apt install libelf-dev dwarves

# 内核代码浏览
# 推荐使用 VSCode + clangd + bear，或者在线看
# https://elixir.bootlin.com/linux/latest/source
# https://lxr.linux.no/
```

---

## 📐 学习路径速查

```
你的基础 → 推荐起点

Ubuntu 熟悉，命令行熟悉：
  ├── 快速过一遍 阶段一 第1周的目录结构和第4周的编译工具链
  ├── 直接进入 阶段二（C 语言深度进修）
  └── 正常推进后续阶段

有单片机经验：
  ├── 对寄存器操作、位操作有概念 → 阶段二第4周 会轻松很多
  ├── 对中断有概念 → 阶段三第1周 OS中断部分易理解
  └── 对 I2C/SPI 有使用经验 → 阶段五第9-10周 外设调试有基础

有 ROS 经验：
  ├── Linux 基础扎实 → 阶段一直接跳过
  ├── 可能有交叉编译经验 → 阶段五部分可加速
  └── 机器人项目可选为阶段六实战方向
```

---

## 🔗 社区与交流

- **Linux 内核邮件列表 (LKML)：** https://lore.kernel.org/lkml/
- **Stack Overflow：** linux-kernel, embedded-linux 标签
- **Reddit：** r/embedded, r/linuxdev
- **中文社区：** CSDN 嵌入式版、知乎嵌入式话题
- **GitHub：** 搜索 embedded-linux 项目学习

---

## 💾 推荐收藏的 GitHub 仓库

| 仓库 | 说明 |
|------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | Linux 内核主线 |
| [u-boot/u-boot](https://github.com/u-boot/u-boot) | U-Boot 主线 |
| [buildroot/buildroot](https://github.com/buildroot/buildroot) | Buildroot |
| [raspberrypi/linux](https://github.com/raspberrypi/linux) | 树莓派内核 |
| [STMicroelectronics/linux](https://github.com/STMicroelectronics/linux) | ST 的 Linux 内核 |
| [bootlin/](https://github.com/bootlin) | Bootlin 的培训资料和工具 |

---

## 🎓 认证（可选）

- **Linux Foundation：** LFCS (Linux Foundation Certified Sysadmin)
- **Linux Foundation：** LFCE (Linux Foundation Certified Engineer)
- **嵌入式相关认证暂无公认标准**，项目经验和 GitHub 更重要

---

> 💡 **最重要的资源是你的好奇心和实践。** 不要只读不练，不要怕出错，每个 bug 都是学习机会。
