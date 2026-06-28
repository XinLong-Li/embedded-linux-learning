# 阶段一：Linux 基础操作

> **时间：** 4-6 周 | **难度：** ⭐☆☆☆☆ | **目标：** 把 Linux 当作日常开发环境自如使用

---

## 📌 学习目标

- 安装 Ubuntu 系统，配置开发环境
- 熟练使用常用 Shell 命令（50+ 命令）
- 理解 Linux 文件系统结构
- 掌握 vim/编辑器基本操作
- 能编写 Shell 脚本自动化任务
- 理解用户、权限、进程概念
- 能用 gcc + make 编译 C 程序

---

## 📖 第1周：安装与基础操作

### Day 1-2：安装 Ubuntu
- [ ] 安装 VirtualBox 或 VMware
- [ ] 创建 Ubuntu 24.04 虚拟机（或安装双系统）
- [ ] 完成系统初始化配置（语言、时区、用户）
- [ ] 安装 VirtualBox Guest Additions（共享剪贴板、文件夹）

### Day 3-4：终端与基本命令
- [ ] 打开终端，理解 shell 是什么（bash/zsh）
- [ ] 必学命令（每个都动手敲）：
  ```bash
  pwd          # 当前目录
  ls           # 列出文件（-l, -a, -h 参数必学）
  cd           # 切换目录（., .., ~, - 的含义）
  mkdir        # 创建目录
  touch        # 创建空文件
  cp           # 复制（-r 递归）
  mv           # 移动/重命名
  rm           # 删除（小心！）
  cat          # 查看文件内容
  less / more  # 分页查看
  head / tail  # 查看头/尾（tail -f 很重要）
  echo         # 输出文本
  man          # 查看手册（养成习惯！）
  ```
- [ ] 理解 Linux 目录结构：
  ```
  /     根目录
  ├── /bin     基本命令
  ├── /boot    启动文件
  ├── /dev     设备文件
  ├── /etc     配置文件
  ├── /home    用户目录
  ├── /lib     库文件
  ├── /proc    进程信息（虚拟文件系统）
  ├── /sys     内核信息（虚拟文件系统）
  ├── /tmp     临时文件
  ├── /usr     用户程序
  └── /var     可变数据（日志等）
  ```

### Day 5-7：文件操作进阶
- [ ] 文件权限：
  ```bash
  chmod        # 修改权限（数字法和符号法）
  chown        # 修改所有者
  ls -l        # 理解 rwx 含义
  ```
- [ ] 管道与重定向：
  ```bash
  >            # 输出重定向
  >>           # 追加
  <            # 输入重定向
  |            # 管道
  ```
- [ ] 文本处理三剑客入门：
  ```bash
  grep         # 文本搜索
  sed          # 流编辑器（基础替换）
  awk          # 文本处理（基础用法）
  ```
- [ ] 其他实用命令：
  ```bash
  find         # 查找文件
  which        # 查找命令位置
  file         # 查看文件类型
  wc           # 统计行/字/字节
  sort         # 排序
  uniq         # 去重
  ```

### 📝 第1周练习
1. 创建如下目录结构：`~/projects/linux-learning/{notes,code,scripts}`
2. 用 `find` 找出 `/etc` 下所有 `.conf` 结尾的文件，存到列表文件
3. 用管道统计一个文本文件中的单词数

---

## 📖 第2周：用户、进程与软件管理

### Day 8-10：用户与权限深入
- [ ] 用户管理：
  ```bash
  whoami       # 我是谁
  id           # 用户ID信息
  su / sudo    # 切换用户/提权
  useradd      # 添加用户
  passwd       # 改密码
  ```
- [ ] 理解 `/etc/passwd`, `/etc/shadow`, `/etc/group`
- [ ] sudo 配置 (`/etc/sudoers`)
- [ ] 理解 root 用户和普通用户的区别
- [ ] 实操：创建新用户，配置 sudo 权限

### Day 11-12：进程管理
- [ ] 进程概念理解（PID, PPID）
- [ ] 进程命令：
  ```bash
  ps           # 查看进程（ps aux 必会）
  top / htop   # 动态查看进程
  kill         # 发送信号（kill -9 vs kill -15）
  pstree       # 进程树
  jobs         # 后台任务
  fg / bg      # 前后台切换
  nohup / &    # 后台运行
  ```
- [ ] 理解 `/proc` 文件系统
- [ ] 实操：写一个死循环程序，用各种方式终止它

### Day 13-14：软件包管理
- [ ] apt 包管理器：
  ```bash
  apt update           # 更新软件源
  apt upgrade          # 升级已安装的包
  apt install <pkg>    # 安装
  apt remove <pkg>     # 卸载
  apt search <keyword> # 搜索
  dpkg -l              # 列出已安装的包
  ```
- [ ] 理解软件源（`/etc/apt/sources.list`）
- [ ] 安装开发工具：
  ```bash
  sudo apt install build-essential gcc g++ gdb make
  sudo apt install git vim curl wget
  ```
- [ ] 从源码编译安装一个软件（理解 `./configure && make && make install`）

### 📝 第2周练习
1. 查看系统中占用内存最多的 5 个进程
2. 写一个脚本每5秒记录当前时间+CPU温度到日志（用到 cron 或 while 循环）
3. 从 GitHub 克隆任意 C 项目，手动编译并运行

---

## 📖 第3周：Shell 脚本编程

### Day 15-17：Shell 脚本基础
- [ ] 理解 shebang：`#!/bin/bash`
- [ ] 变量定义与使用：
  ```bash
  name="hello"
  echo $name
  echo ${name}
  ```
- [ ] 特殊变量：`$0`, `$1`, `$#`, `$@`, `$?`, `$$`
- [ ] 条件判断：
  ```bash
  if [ condition ]; then
      ...
  elif [ condition ]; then
      ...
  else
      ...
  fi
  ```
- [ ] 文件测试：`-f`, `-d`, `-e`, `-r`, `-w`, `-x`
- [ ] 字符串比较：`=`, `!=`, `-z`, `-n`
- [ ] 数值比较：`-eq`, `-ne`, `-lt`, `-gt`, `-le`, `-ge`

### Day 18-19：循环与函数
- [ ] for 循环：
  ```bash
  for i in {1..10}; do ... done
  for file in *.c; do ... done
  ```
- [ ] while 循环：
  ```bash
  while [ condition ]; do ... done
  ```
- [ ] 函数定义与调用：
  ```bash
  my_func() {
      local var="local"
      echo "Hello $1"
  }
  my_func "World"
  ```
- [ ] 数组操作

### Day 20-21：实用脚本实战
- [ ] 日志分析脚本
- [ ] 批量文件重命名脚本
- [ ] 系统监控脚本（CPU/内存/磁盘）
- [ ] 代码编译自动化脚本
- [ ] 理解环境变量（PATH, HOME, LD_LIBRARY_PATH 等）
- [ ] `.bashrc` / `.profile` 配置

### 📝 第3周练习：综合脚本项目
写一个 `dev-setup.sh` 脚本，自动完成：
1. 创建项目目录结构
2. 安装必要的开发包
3. 配置 Git 用户信息
4. 生成 SSH key（如不存在）
5. 输出系统信息摘要

---

## 📖 第4周：编辑器与开发工具链

### Day 22-24：编辑器（选一个深耕）
**推荐 vim（服务器环境必备）：**
- [ ] 基本模式：Normal, Insert, Visual, Command
- [ ] 移动：h, j, k, l, w, b, 0, $, gg, G
- [ ] 编辑：i, a, o, x, dd, yy, p, u, Ctrl+r
- [ ] 搜索替换：`/pattern`, `:s/old/new/g`
- [ ] 窗口分割：`:split`, `:vsplit`, Ctrl+w
- [ ] 配置 `.vimrc`

**或者 VS Code / VSCodium：**
- [ ] 远程开发插件 (Remote-SSH)
- [ ] C/C++ 插件配置

### Day 25-26：编译工具链
- [ ] 理解编译过程：预处理 → 编译 → 汇编 → 链接
  ```bash
  gcc -E hello.c -o hello.i    # 预处理
  gcc -S hello.i -o hello.s    # 编译
  gcc -c hello.s -o hello.o    # 汇编
  gcc hello.o -o hello         # 链接
  ```
- [ ] gcc 常用选项：`-Wall`, `-g`, `-O2`, `-I`, `-L`, `-l`, `-D`
- [ ] 静态库 vs 动态库：
  ```bash
  # 静态库 .a
  gcc -c foo.c -o foo.o
  ar rcs libfoo.a foo.o
  # 动态库 .so
  gcc -shared -fPIC foo.c -o libfoo.so
  ```

### Day 27-28：Makefile 基础
- [ ] 理解 Makefile 结构：目标、依赖、命令
  ```makefile
  target: dependencies
      command
  ```
- [ ] 变量、自动变量 (`$@`, `$^`, `$<`)
- [ ] 模式规则与通配符
- [ ] 伪目标 `.PHONY`
- [ ] 实操：为一个多文件 C 项目写 Makefile

### 📝 第4周练习：阶段一综合项目
项目：**Linux 开发环境自动化工具**
- 写一个完整的 Shell 脚本工具集
- 包含项目模板生成、代码编译、系统信息收集功能
- 用 Makefile 管理多个脚本的安装
- 写出清晰的使用文档

---

## ✅ 阶段一检查清单

完成以下所有项才算过关：
- [ ] 能在无图形界面的终端中完成所有日常操作
- [ ] 能独立写出 100 行以上的 Shell 脚本
- [ ] 理解 Linux 文件权限模型，能排查权限问题
- [ ] 能为多文件 C 项目手写 Makefile
- [ ] 知道如何查看日志、排查进程、管理服务
- [ ] 能用 gdb 调试简单的 C 程序

---

## 📚 阶段一推荐资源

- **书籍：** 《鸟哥的 Linux 私房菜 — 基础学习篇》第四版
- **在线：** [Linux Journey](https://linuxjourney.com/)
- **练习：** [Bandit](https://overthewire.org/wargames/bandit/) (Linux 闯关游戏)
- **参考：** `man` 命令是最佳参考资料
