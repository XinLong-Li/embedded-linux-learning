# Ubuntu 常用 CLI 命令详解

> 按使用频率和功能分类，每个命令详解其选项、输出字段和实战用例。
> 目标读者：嵌入式 Linux 开发者，日常需要在终端完成编码、构建、调试、系统诊断。

---

## 频率分级

| 等级 | 含义 | 场景 |
|------|------|------|
| ⭐⭐⭐⭐⭐ | 每天必用 | 文件浏览、编辑、构建、包管理 |
| ⭐⭐⭐⭐ | 每周数次 | 权限、进程、压缩、网络 |
| ⭐⭐⭐ | 常用但间歇 | 磁盘、定时任务、用户管理 |
| ⭐⭐ | 需要时查阅 | 性能分析、抓包、内核日志 |
| ⭐ | 特殊场景 | 内核模块、SELinux/AppArmor |

---

## 1. 文件与目录操作

### 1.1 `ls` — 列出目录内容 ⭐⭐⭐⭐⭐

```bash
ls [OPTION]... [FILE]...
```

#### 常用选项

| 选项 | 长选项 | 作用 |
|------|--------|------|
| `-a` | `--all` | 列出所有条目，包括 `.` 开头隐藏文件 |
| `-A` | `--almost-all` | 同 `-a` 但不显示 `.` 和 `..` |
| `-l` | | 长格式：权限、链接数、所有者、大小、修改时间、文件名 |
| `-h` | `--human-readable` | 与 `-l` 配合，用 K/M/G 显示文件大小 |
| `-t` | | 按修改时间排序，最新的在前 |
| `-r` | `--reverse` | 反向排序 |
| `-S` | | 按文件大小排序，最大的在前 |
| `-R` | `--recursive` | 递归列出子目录 |
| `-d` | `--directory` | 列出目录本身而非其内容 |
| `-i` | `--inode` | 显示 inode 号 |
| `-1` | | 每行列一个文件（单列模式） |
| `--color=auto` | | 按文件类型着色（Ubuntu 默认 alias） |
| `-F` | `--classify` | 文件名后加指示符：`/`=目录, `*`=可执行, `@`=符号链接 |
| `-Z` | `--context` | 显示 SELinux 安全上下文 |

#### 长格式 (`ls -l`) 输出字段解读

```
-rwxr-xr-x 1 root root 12345 Jul 19 10:30 myapp*
|  |  |  |  |    |    |       |         |
|  |  |  |  |    |    |       |         +-- 文件名（* 表示可执行）
|  |  |  |  |    |    |       +-- 修改时间
|  |  |  |  |    |    +-- 文件大小（字节）
|  |  |  |  |    +-- 所属组
|  |  |  |  +-- 所有者
|  |  |  +-- 硬链接数
|  |  +-- 其他用户权限 (r=读 w=写 x=执行 -无)
|  +-- 组权限
+-- 文件类型 + 所有者权限
    - 普通文件  d 目录  l 符号链接
    c 字符设备  b 块设备  s socket  p 管道
```

#### 实战用例

```bash
# 最常用：按修改时间倒序，最近的文件在底部
ls -ltr

# 人可读大小 + 所有文件
ls -lah

# 按大小排序，快速找大文件
ls -lhS

# 只列目录本身
ls -ld */

# 递归 + 人可读，一层层展开看
ls -lRh

# 查看 inode 号（判断硬链接时有用）
ls -li

# 按扩展名分组列出
ls -l *.c
```

---

### 1.2 `cd` — 切换工作目录 ⭐⭐⭐⭐⭐

```bash
cd [DIR]
```

| 用法 | 作用 |
|------|------|
| `cd /absolute/path` | 切换到绝对路径 |
| `cd relative/path` | 切换到相对路径 |
| `cd` 或 `cd ~` | 切换到 home 目录 |
| `cd -` | 切换到上一个工作目录（利用 `$OLDPWD`） |
| `cd ..` | 切换到上级目录 |
| `cd ../..` | 上两级 |
| `cd !$` | 将上一条命令的最后一个参数作为目录（bash 历史扩展） |

```bash
# 常见组合：先 ls 看看，再 cd 进去
ls /opt/
cd /opt/toolchain

# 在两个目录间来回切换
cd /home/user/project
cd /var/log
cd -      # 回去 /home/user/project
cd -      # 又回到 /var/log

# 设置 CDPATH：让你在任意位置直接 cd 到常用目录
export CDPATH=.:~:~/projects:~/src
# 之后在任意位置 cd myproject 即可
```

---

### 1.3 `pwd` — 打印当前工作目录 ⭐⭐⭐⭐⭐

```bash
pwd [OPTION]
```

| 选项 | 作用 |
|------|------|
| `-L` | 显示逻辑路径（保留符号链接，默认） |
| `-P` | 显示物理路径（解析所有符号链接） |

```bash
# 当你通过符号链接进入目录时，区别很明显
ln -s /var/log /tmp/mylog
cd /tmp/mylog
pwd -L    # /tmp/mylog     —— 你"看到"的路径
pwd -P    # /var/log       —— 实际物理路径
```

---

### 1.4 `tree` — 树状列出目录结构 ⭐⭐⭐⭐

```bash
tree [OPTION]... [DIR]
```

| 选项 | 作用 |
|------|------|
| `-L N` | 限制显示深度为 N 层 |
| `-d` | 仅显示目录，不显示文件 |
| `-a` | 显示隐藏文件 |
| `-I pattern` | 排除匹配 pattern 的文件 |
| `-P pattern` | 仅列出匹配 pattern 的文件 |
| `-h` | 显示文件大小（人可读） |
| `-s` | 显示文件大小（字节） |
| `-f` | 显示每个文件的完整路径 |
| `-o file` | 输出到文件 |
| `--du` | 对每个目录累计显示其总大小 |
| `--charset=ascii` | 用 ASCII 字符绘制线条（兼容性更好） |

```bash
# 只看 2 层深
tree -L 2 ~/project

# 只看目录骨架
tree -d

# 排除 node_modules 和 .git
tree -I "node_modules|.git"

# 同时显示文件大小
tree -sh

# 生成项目结构文本
tree --charset=ascii -L 3 > project-structure.txt
```

---

### 1.5 `find` — 搜索文件 ⭐⭐⭐⭐⭐

这是 Linux 最强大的文件搜索工具，功能远不止 `-name`。

```bash
find [PATH]... [EXPRESSION]
```

#### 按名称/路径

| 表达式 | 作用 |
|--------|------|
| `-name "*.c"` | 文件名精确匹配（区分大小写），支持通配符 |
| `-iname "*.c"` | 同 `-name`，不区分大小写 |
| `-path "*/src/*"` | 路径匹配 |
| `-regex ".*\.\(c\|h\)$"` | 正则匹配路径 |

#### 按类型

| 表达式 | 作用 |
|--------|------|
| `-type f` | 普通文件 |
| `-type d` | 目录 |
| `-type l` | 符号链接 |
| `-type b` | 块设备 |
| `-type c` | 字符设备 |

#### 按时间（以下均为 find 特有语义）

| 表达式 | 含义 |
|--------|------|
| `-mtime +7` | 修改时间距今 **超过** 7 天 |
| `-mtime -7` | 修改时间距今 **不到** 7 天 |
| `-mtime 7` | 修改时间距今**恰好** 7 天（精度到天） |
| `-mmin -30` | 修改时间距今不到 30 **分钟** |
| `-atime N` | 同上但针对访问时间 |
| `-ctime N` | 同上但针对元数据变更时间 |
| `-newer file` | 比 file 更新 |
| `-newermt "2026-01-01"` | 比指定日期更新 |

#### 按大小

| 表达式 | 含义 |
|--------|------|
| `-size +100M` | 大于 100MB |
| `-size -10k` | 小于 10KB |
| `-size 0` | 空文件 |
| `-empty` | 空文件或空目录 |

#### 按权限/所有者

| 表达式 | 含义 |
|--------|------|
| `-user alice` | 属于 alice 的文件 |
| `-group dev` | 属于 dev 组的文件 |
| `-perm 644` | 权限精确为 644 |
| `-perm -u+x` | 所有者有执行权限 |
| `-perm /o+w` | 其他人有写权限 |

#### 动作（对匹配到的文件做什么）

| 表达式 | 含义 |
|--------|------|
| `-print` | 打印路径（默认） |
| `-print0` | 以 NUL 结尾打印（配合 `xargs -0` 处理含空格文件名） |
| `-delete` | 删除匹配文件 |
| `-exec cmd {} \;` | 对每个匹配执行 cmd，`{}` 是占位符，`\;` 结束 |
| `-exec cmd {} +` | 同上但批量传参（效率高，只启一次进程） |
| `-ls` | 以 `ls -dils` 格式输出 |

#### 逻辑组合

| 表达式 | 含义 |
|--------|------|
| `expr1 -a expr2` | 与（默认） |
| `expr1 -o expr2` | 或 |
| `! expr` 或 `-not expr` | 非 |
| `\( expr \)` | 分组（括号需要转义） |

#### 实战用例

```bash
# 查找所有 .c 文件并统计行数
find . -name "*.c" -exec wc -l {} + | tail -1

# 查找并删除 7 天前的 .log 文件
find /var/log -name "*.log" -mtime +7 -delete

# 查找大于 100MB 的文件，列出大小
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# 查找含空格文件名的文件（配合 xargs -0）
find . -name "*.txt" -print0 | xargs -0 grep "TODO"

# 查找空目录并删除
find /tmp -type d -empty -delete

# 查找最近 30 分钟修改过的 .c 或 .h 文件
find . -type f \( -name "*.c" -o -name "*.h" \) -mmin -30

# 查找有 SUID 位的文件（安全审计用）
find / -type f -perm -4000 -ls 2>/dev/null

# 排除指定目录
find . -type f -not -path "./.git/*" -not -path "./build/*"
```

---

### 1.6 `locate` — 快速文件定位 ⭐⭐⭐

```bash
locate [OPTION]... PATTERN
```

基于预先构建的文件名数据库（`/var/lib/mlocate/mlocate.db`），速度远超 `find`，但不实时。

| 选项 | 作用 |
|------|------|
| `-i` | 不区分大小写 |
| `-c` | 只输出匹配数量 |
| `-l N` | 限制输出 N 条 |
| `-r regex` | 使用正则表达式 |
| `-e` | 仅返回仍然存在的文件（数据库可能过期） |

```bash
# 使用前先更新数据库
sudo updatedb

# 搜索
locate libstdc++.so
locate -i kernel.img
locate -r '\.patch$'
```

⚠️ `updatedb` 通常由 cron 每天跑一次，刚创建的文件搜不到。

---

### 1.7 `which` — 显示命令的完整路径 ⭐⭐⭐⭐

```bash
which [OPTION] COMMAND
```

| 选项 | 作用 |
|------|------|
| `-a` | 显示所有匹配，不只是第一个 |

```bash
which gcc        # /usr/bin/gcc
which -a python  # 如果 PATH 中有多个 python，全部列出
```

---

### 1.8 `file` — 识别文件类型 ⭐⭐⭐⭐

```bash
file [OPTION]... FILE...
```

通过读取文件头（magic bytes）识别类型，不依赖扩展名。

| 选项 | 作用 |
|------|------|
| `-b` | 简洁输出（不显示文件名） |
| `-i` | 输出 MIME 类型 |
| `-z` | 尝试查看压缩文件内部 |
| `-s` | 对块/字符设备文件也做识别 |

```bash
file a.out              # ELF 64-bit LSB executable, x86-64...
file unknown.bin        # data, 或识别出具体格式
file -i index.html      # text/html; charset=utf-8
file -s /dev/sda        # 查看磁盘分区表类型
file firmware.bin       # 嵌入式固件分析第一步
```

---

### 1.9 `mkdir` — 创建目录 ⭐⭐⭐⭐⭐

```bash
mkdir [OPTION]... DIR...
```

| 选项 | 作用 |
|------|------|
| `-p` | 递归创建父目录，已存在也不报错 |
| `-m mode` | 直接设置权限，如 `-m 755` |
| `-v` | 打印每个创建的目录 |

```bash
# 一次创建深层路径
mkdir -p ~/project/{src,include,build,docs,tests}

# 创建并设权限
mkdir -m 700 ~/private

# 带时间戳的临时目录
mkdir -p /tmp/backup_$(date +%Y%m%d)
```

---

### 1.10 `touch` — 创建文件 / 更新时戳 ⭐⭐⭐⭐⭐

```bash
touch [OPTION]... FILE...
```

| 选项 | 作用 |
|------|------|
| `-a` | 仅修改访问时间（atime） |
| `-m` | 仅修改修改时间（mtime） |
| `-t STAMP` | 指定时间 `[[CC]YY]MMDDhhmm[.ss]` |
| `-d STRING` | 用自然语言指定时间 |
| `-c` | 文件不存在时不创建 |

```bash
touch newfile.txt                          # 创建空文件
touch -t 202601011200 file                 # 设为 2026-01-01 12:00
touch -d "3 days ago" file                 # 设为 3 天前
touch -d "2026-01-01 12:00:00" file

# Makefile 中常用：强制重新构建
touch configure.ac   # 下次 make 会重新跑 autoconf
```

---

### 1.11 `cp` — 复制文件与目录 ⭐⭐⭐⭐⭐

```bash
cp [OPTION]... SOURCE DEST
cp [OPTION]... SOURCE... DIR
```

| 选项 | 作用 |
|------|------|
| `-r` / `-R` | 递归复制目录 |
| `-a` | 归档模式 = `-dR --preserve=all`，保留所有属性、符号链接 |
| `-v` | 显示复制进度 |
| `-i` | 覆盖前询问 |
| `-n` | 不覆盖已存在文件 |
| `-u` | 仅当源比目标新时才复制（update） |
| `-p` | 保留时间戳、权限、所有者 |
| `-l` | 创建硬链接而非复制 |
| `-s` | 创建符号链接而非复制 |
| `-b` | 覆盖前创建备份 `file~` |
| `--parents` | 在目标中重建完整的源路径结构 |

```bash
# 安全复制：显示进度 + 保留属性
cp -av source_dir/ dest_dir/

# 只复制比目标更新的文件（同步用）
cp -ru src/ dst/

# 复制并保留完整路径层级
cp --parents src/subdir/file.txt /tmp/backup/
# 结果：/tmp/backup/src/subdir/file.txt

# 批量复制到目标目录
cp *.txt *.md ~/docs/
```

---

### 1.12 `mv` — 移动 / 重命名 ⭐⭐⭐⭐⭐

```bash
mv [OPTION]... SOURCE DEST
```

| 选项 | 作用 |
|------|------|
| `-v` | 显示操作详情 |
| `-i` | 覆盖前询问 |
| `-n` | 不覆盖已存在文件 |
| `-u` | 源比目标新才移动 |
| `-b` | 覆盖前备份目标 |

```bash
# 重命名
mv old_name.txt new_name.txt

# 批量改名（bash 参数扩展）
for f in *.jpeg; do mv "$f" "${f%.jpeg}.jpg"; done

# 移动全部文件到上级目录
mv * ..
```

---

### 1.13 `rm` — 删除文件 ⭐⭐⭐⭐⭐

```bash
rm [OPTION]... FILE...
```

| 选项 | 作用 |
|------|------|
| `-r` / `-R` | 递归删除目录 |
| `-f` | 强制删除，不提示，不报错（不存在也 OK） |
| `-i` | 每个文件都询问 |
| `-v` | 显示删除的文件 |
| `-d` | 删除空目录 |

```bash
# ⚠️ 极危险命令，永远在执行前多看一眼路径
rm -rf /tmp/build/     # OK —— 限定了 /tmp
rm -rf $BUILD_DIR/     # 务必确保变量非空！
rm -rf "${BUILD_DIR:?}"  # 安全写法：变量为空则报错退出

# 交互式删除（逐个确认）
rm -ri dir/

# 删除除了特定文件外的所有文件
shopt -s extglob
rm !(keep_this.txt|and_this.txt)
```

---

### 1.14 `ln` — 创建链接 ⭐⭐⭐⭐

```bash
ln [OPTION]... TARGET LINK_NAME   # 硬链接
ln -s [OPTION]... TARGET LINK_NAME  # 符号链接（软链接）
```

**硬链接 vs 符号链接：**

| 特性 | 硬链接 | 符号链接 |
|------|--------|----------|
| 跨文件系统 | ❌ | ✅ |
| 指向目录 | ❌ | ✅ |
| 删除原名后 | 仍然有效 | 断链（dangling） |
| inode | 相同 | 不同 |
| 占用空间 | 0（仅目录项） | 路径名大小 |

| 选项 | 作用 |
|------|------|
| `-s` | 创建符号链接（软链接） |
| `-f` | 强制覆盖已存在的目标 |
| `-n` | 将已存在的符号链接当普通文件处理 |
| `-v` | 显示操作 |

```bash
# 符号链接
ln -s /usr/bin/python3.12 /usr/local/bin/python
ln -s /opt/gcc-arm-13.2 /opt/gcc-arm     # 版本切换只需改链接

# 硬链接（相同 inode，节省空间，互为备份）
ln large_file.dat large_file_backup.dat

# 查看符号链接指向
readlink -f /usr/bin/python
ls -l /usr/bin/python       # -> python3.12
```

---

### 1.15 `cat` — 连接并输出文件内容 ⭐⭐⭐⭐⭐

```bash
cat [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-n` | 所有行加行号 |
| `-b` | 非空行加行号 |
| `-s` | 压缩连续空行为一行 |
| `-A` | 显示所有控制字符（`$`=行尾，`^I`=tab） |
| `-E` | 行尾显示 `$` |
| `-T` | tab 显示为 `^I` |
| `-v` | 显示不可见字符 |

```bash
# 拼接多个文件
cat file1.c file2.c > combined.c

# 创建多行文件（Ctrl+D 结束）
cat > newfile.txt << EOF
line 1
line 2
EOF

# 查看空格和 tab 分布
cat -A Makefile

# 反转文件内容
tac file.txt
```

---

### 1.16 `less` — 分页浏览 ⭐⭐⭐⭐⭐

```bash
less [OPTION]... [FILE]...
```

比 `more` 强大：支持前后翻页、搜索、高亮、跟随。

**浏览中快捷键：**

| 键 | 功能 |
|----|------|
| `Space / f` | 下一页 |
| `b` | 上一页 |
| `j / k` 或 `↑ / ↓` | 上下滚动一行 |
| `g` | 跳到开头 |
| `G` | 跳到末尾 |
| `/pattern` | 向下搜索 |
| `?pattern` | 向上搜索 |
| `n` | 下一个匹配 |
| `N` | 上一个匹配 |
| `F` | 进入 tail -f 模式（Ctrl+C 退出） |
| `v` | 用默认编辑器打开当前文件 |
| `h` | 帮助 |
| `q` | 退出 |

**命令行选项：**

| 选项 | 作用 |
|------|------|
| `-N` | 显示行号 |
| `-S` | 不折行（左右滚动查看长行） |
| `-F` | 持续读取（类似 `tail -f`） |
| `+N` | 从第 N 行开始 |
| `+/pattern` | 跳到第一个匹配 |

```bash
# 查看日志 + 实时跟踪
less -N +F /var/log/syslog

# 不换行的表格数据
less -S data.csv

# 从第 100 行开始看
less +100 file.txt

# 管道使用
ps aux | less -S
dmesg | less +G     # 直接跳到末尾
```

---

### 1.17 `head` — 输出文件开头 ⭐⭐⭐⭐⭐

```bash
head [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-n N` | 输出前 N 行（默认 10） |
| `-c N` | 输出前 N 字节 |
| `-q` | 多文件时不显示文件名头 |
| `-v` | 多文件时总是显示文件名头 |

```bash
head -n 20 /etc/passwd
head -c 1024 firmware.bin > header.bin   # 截取前 1024 字节
head -n 5 *.c                            # 查看多个文件的头部
```

---

### 1.18 `tail` — 输出文件末尾 ⭐⭐⭐⭐⭐

```bash
tail [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-n N` | 输出后 N 行（默认 10） |
| `-n +N` | 从第 N 行开始输出到末尾 |
| `-c N` | 输出后 N 字节 |
| `-f` | 持续跟踪文件追加内容 |
| `-F` | 同 `-f`，但文件被删除/转储后自动重新打开 |
| `--retry` | 文件不可用时持续重试 |
| `--pid=PID` | 与 `-f` 配合，指定进程退出后自动停止 |

```bash
# 实时跟踪日志（最常用）
tail -f /var/log/syslog

# 稳健跟踪：日志滚动（logrotate）后不丢
tail -F /var/log/nginx/access.log

# 跳过前 100 行，从 101 行开始输出
tail -n +101 file.txt

# 跟踪并在目标进程退出后自动结束
tail -f --pid=$(pgrep myapp) /var/log/myapp.log

# 查看文件最后 1024 字节
tail -c 1024 firmware.bin
```

---

### 1.19 `grep` — 文本搜索 ⭐⭐⭐⭐⭐

```bash
grep [OPTION]... PATTERN [FILE]...
```

#### 核心选项

| 选项 | 长选项 | 作用 |
|------|--------|------|
| `-i` | `--ignore-case` | 不区分大小写 |
| `-v` | `--invert-match` | 反向匹配（输出不匹配的行） |
| `-n` | `--line-number` | 显示行号 |
| `-r` / `-R` | `--recursive` | 递归搜索目录 |
| `-l` | `--files-with-matches` | 只显示文件名（不显示匹配行） |
| `-L` | `--files-without-match` | 只显示不匹配的文件名 |
| `-c` | `--count` | 显示每个文件的匹配次数 |
| `-w` | `--word-regexp` | 全词匹配（不是子串） |
| `-x` | `--line-regexp` | 整行匹配 |
| `-A N` | `--after-context=N` | 显示匹配行及其后 N 行 |
| `-B N` | `--before-context=N` | 显示匹配行及其前 N 行 |
| `-C N` | `--context=N` | 显示匹配行及其前后各 N 行 |
| `-e PAT` | `--regexp=PAT` | 指定多个 pattern |
| `-E` | `--extended-regexp` | 扩展正则（`+`, `?`, `|`, `()`） |
| `-F` | `--fixed-strings` | 固定字符串匹配（不解析正则，快） |
| `-o` | `--only-matching` | 只输出匹配到的部分（不输出整行） |
| `--color=auto` | | 高亮匹配部分 |
| `--include=GLOB` | | 只搜索匹配通配符的文件 |
| `--exclude=GLOB` | | 排除匹配通配符的文件 |
| `--exclude-dir=DIR` | | 排除目录 |

#### 实战用例

```bash
# 递归搜索源码中的 TODO
grep -rn "TODO" src/

# 搜索时显示上下文（前后各 3 行）
grep -rn -C 3 "error" /var/log/

# 不区分大小写 + 全词匹配
grep -iwn "main" *.c

# 排除 .git 和 build 目录
grep -rn "pattern" . --exclude-dir={.git,build,node_modules}

# 只在 .c 和 .h 文件中搜索
grep -rn "malloc" --include="*.c" --include="*.h" src/

# 反向搜索：找没有注释的行
grep -v '^\s*//' source.c

# 统计匹配次数
grep -rc "TODO" src/

# 提取 IP 地址
ifconfig | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'

# 列出所有包含 main 函数的 C 文件
grep -rl "int main" --include="*.c" .
```

#### grep 家族

| 命令 | 等价 | 说明 |
|------|------|------|
| `grep` | `grep -G` | 基本正则（BRE） |
| `egrep` | `grep -E` | 扩展正则（ERE，支持 `+?|()`） |
| `fgrep` | `grep -F` | 固定字符串（无正则，快） |
| `rgrep` | `grep -r` | 递归 |

---

### 1.20 `wc` — 统计行/词/字符/字节 ⭐⭐⭐⭐

```bash
wc [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-l` | 行数 |
| `-w` | 单词数 |
| `-m` | 字符数（含多字节字符） |
| `-c` | 字节数 |
| `-L` | 最长行的长度 |

```bash
# 统计源码规模
find . -name "*.c" -o -name "*.h" | xargs wc -l | tail -1
git ls-files | xargs wc -l | tail -1

# 统计进程数
ps aux | wc -l

# 统计文件数
find . -type f | wc -l
```

---

### 1.21 `sort` — 排序 ⭐⭐⭐⭐

```bash
sort [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-n` | 按数值排序（而非字典序） |
| `-h` | 按人可读大小排序（K/M/G） |
| `-r` | 反向排序 |
| `-u` | 去重输出 |
| `-k N` | 按第 N 列排序（从 1 开始） |
| `-t CHAR` | 指定分隔符（默认空白） |
| `-o FILE` | 输出到文件 |
| `-R` | 随机排序 |
| `-V` | 版本号排序（如 1.10 > 1.9） |

```bash
# 按内存排序进程
ps aux | sort -k4 -rn | head -10

# 按磁盘使用排序
du -sh * | sort -rh | head -20

# 版本号排序
printf "v1.2.0\nv1.10.0\nv2.0.0\nv1.2.1\n" | sort -V

# /etc/passwd 按 UID（第 3 列）排序
sort -t: -k3 -n /etc/passwd
```

---

### 1.22 `uniq` — 去重 ⭐⭐⭐

```bash
uniq [OPTION]... [INPUT [OUTPUT]]
```

⚠️ `uniq` 只合并相邻重复行，通常需要先 `sort`。

| 选项 | 作用 |
|------|------|
| `-c` | 前缀显示重复次数 |
| `-d` | 只输出重复行 |
| `-u` | 只输出不重复的行 |
| `-i` | 忽略大小写 |

```bash
# 统计访问日志中各 IP 的次数
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn | head

# 找出日志中重复的错误
grep "ERROR" app.log | sort | uniq -d
```

---

### 1.23 `cut` — 按列提取 ⭐⭐⭐

```bash
cut [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-d CHAR` | 指定分隔符（默认 tab） |
| `-f N` | 提取第 N 列（逗号分隔：`-f1,3`，范围 `-f1-5`） |
| `-c N` | 按字符位置提取 |
| `-b N` | 按字节位置提取 |
| `--complement` | 提取除了指定列以外的列 |

```bash
# 提取所有用户名
cut -d: -f1 /etc/passwd

# 提取 shell 及其用户数
cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn

# 按字符位置提取
echo "abcdefgh" | cut -c2-5   # bcde

# 提取 PATH 中的目录
echo $PATH | tr ':' '\n'
```

---

### 1.24 `diff` — 比较文件差异 ⭐⭐⭐⭐

```bash
diff [OPTION]... FILE1 FILE2
```

| 选项 | 作用 |
|------|------|
| `-u` | unified diff 格式（最常用，带上下文） |
| `-c` | context diff 格式 |
| `-r` | 递归比较目录 |
| `-q` | 只报告是否有差异 |
| `-w` | 忽略空白差异 |
| `-B` | 忽略空行差异 |
| `-i` | 忽略大小写 |
| `-y` | 并排显示 |
| `-W N` | 并排显示宽度 |
| `--color` | 着色输出 |

```bash
# unified 格式（最常用，patch 命令识别的格式）
diff -u old.c new.c

# 递归比较两个目录
diff -r project_v1/ project_v2/

# 只告诉你哪些文件不同
diff -rq build_v1/ build_v2/

# 并排显示（适合窄代码）
diff -y -W 120 old.c new.c | less -S

# 生成 patch 文件
diff -uN original/ modified/ > changes.patch
patch -p1 < changes.patch
```

---

## 2. 包管理

### 2.1 `apt` — APT 包管理 ⭐⭐⭐⭐⭐

```bash
apt [COMMAND] [PACKAGE]...
```

| 子命令 | 常用选项 | 说明 |
|--------|----------|------|
| `update` | | 更新本地包索引（不安装任何东西） |
| `upgrade` | `-y` | 升级所有已安装的包 |
| `full-upgrade` | `-y` | 升级，必要时移除冲突包 |
| `install` | `-y`, `--no-install-recommends` | 安装包 |
| `remove` | `--purge` | 卸载包（`--purge` 同时删除配置文件） |
| `autoremove` | `--purge` | 删除不再需要的依赖 |
| `search` | | 搜索包名和描述 |
| `show` | | 显示包详情（版本、大小、依赖、描述） |
| `list` | `--installed`, `--upgradable` | 列出包 |
| `edit-sources` | | 编辑 `/etc/apt/sources.list` |
| `policy` | | 显示包的版本策略（安装候选、已安装） |
| `depends` | | 列出包依赖 |
| `rdepends` | | 列出反向依赖 |

```bash
# 标准更新流程
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y

# 安装但不装"推荐"包（节省空间）
sudo apt install --no-install-recommends vim

# 搜索
apt search '^linux-image'

# 查看某包的详细信息和依赖
apt show nginx
apt depends nginx

# 查看哪些已安装的包需要升级
apt list --upgradable

# 查看某包从哪个源来、当前装了什么版本
apt policy gcc

# 锁定包版本（阻止升级）
sudo apt-mark hold linux-image-$(uname -r)
sudo apt-mark unhold linux-image-$(uname -r)
```

---

### 2.2 安装 `.deb` 包 — 三种方法 ⭐⭐⭐⭐

Ubuntu 安装本地 `.deb` 文件有三种方式，推荐优先级从高到低：

#### 方法一：`apt install ./package.deb`（推荐，Ubuntu 20.04+）

```bash
sudo apt install ./package.deb
# 或指定绝对路径
sudo apt install /path/to/package.deb
```

这是**现代推荐方式**，因为 `apt` 会自动解析并安装依赖：
- ✅ 自动从仓库下载缺失的依赖
- ✅ 语法统一，容易记
- ✅ 如果依赖无法满足，清晰报错

```bash
# 安装多个 .deb
sudo apt install ./pkg1.deb ./pkg2.deb ./pkg3.deb

# 通配符安装
sudo apt install ./*.deb

# 模拟安装（dry run，预览不实际安装）
sudo apt install --dry-run ./package.deb

# 安装时回答 yes
sudo apt install -y ./package.deb
```

#### 方法二：`dpkg -i`（传统方式，所有版本适用）

```bash
sudo dpkg -i package.deb
```

`dpkg` 是底层工具，**不会自动安装依赖**。如果缺少依赖，安装会失败，需要手动修复：

```bash
# 安装
sudo dpkg -i ./package.deb
# 如果报依赖错误，修复：
sudo apt install -f
# 或
sudo apt --fix-broken install
```

| 选项 | 作用 |
|------|------|
| `-i pkg.deb` | 安装包 |
| `--force-depends` | 忽略依赖错误强制安装（不推荐，可能破坏系统） |
| `--force-overwrite` | 覆盖其他包已有的文件 |
| `--dry-run` / `--no-act` | 模拟安装，不实际执行 |
| `-R DIR` / `--recursive DIR` | 递归安装目录下所有 `.deb` |
| `--unpack` | 仅解包不配置（`dpkg --configure -a` 后续配置） |
| `-B` / `--auto-deconfigure` | 自动反配置被替换的包 |

#### 方法三：`gdebi`（第三方工具，自动处理依赖）

```bash
sudo apt install gdebi
sudo gdebi package.deb
```

类似方法一，但适用于旧版 Ubuntu，且提供交互式界面。

---

### 2.3 查看 `.deb` 包内容（不安装）

```bash
# 列出 .deb 中包含的所有文件
dpkg -c package.deb
dpkg --contents package.deb

# 查看 .deb 的元信息（包名、版本、依赖、描述等）
dpkg -I package.deb
dpkg --info package.deb

# 提取文件到指定目录（不安装到系统）
dpkg -x package.deb /tmp/extracted/        # 提取文件
dpkg -e package.deb /tmp/extracted/DEBIAN/ # 提取控制文件（preinst, postinst 等）
```

---

### 2.4 `dpkg` — 已安装包的管理 ⭐⭐⭐⭐

```bash
dpkg [OPTION]... ACTION
```

| 选项/动作 | 说明 |
|-----------|------|
| `-r pkg` | 移除包（保留配置） |
| `-P pkg` | 清除包（删除配置） |
| `-l [pattern]` | 列出已安装的包 |
| `-L pkg` | 列出包安装的所有文件 |
| `-S path` | 查找哪个包提供了此文件 |
| `-s pkg` | 显示包状态 |
| `--get-selections` | 导出已安装包列表 |
| `--set-selections` | 批量恢复安装 |
| `--configure -a` | 配置所有未配置的包 |

```bash
# 查找 /usr/bin/gcc 属于哪个包
dpkg -S /usr/bin/gcc

# 导出已安装包列表 → 新机器恢复
dpkg --get-selections > packages.txt
# 在新机器上：
sudo dpkg --set-selections < packages.txt
sudo apt-get dselect-upgrade
```

---

### 2.5 `snap` — Snap 包管理 ⭐⭐⭐

```bash
snap [COMMAND]
```

| 子命令 | 说明 |
|--------|------|
| `find pkg` | 搜索 Snap 商店 |
| `info pkg` | 包详情 |
| `install pkg` | 安装（`--classic` 经典模式） |
| `remove pkg` | 卸载 |
| `list` | 列出已安装 |
| `refresh` | 更新所有 |
| `changes` | 查看最近的变更 |
| `disable / enable pkg` | 禁用/启用 |

```bash
snap find code
sudo snap install code --classic    # VS Code 需要经典模式
sudo snap install lxd
snap list
sudo snap refresh
```

---

## 3. 权限与用户

### 3.1 `chmod` — 修改文件权限 ⭐⭐⭐⭐⭐

```bash
chmod [OPTION]... MODE FILE...
```

**两种表示法：**

| 表示法 | 语法 | 示例 | 含义 |
|--------|------|------|------|
| 数字（八进制） | `0NNN` | `chmod 755` | 4=r, 2=w, 1=x，三组相加 |
| 符号 | `[ugoa][+-=][rwx]` | `chmod u+x` | 给 owner 加执行权限 |

**数字权限速查：**

| 数字 | 权限 | 典型用途 |
|------|------|----------|
| `755` | `rwxr-xr-x` | 目录、可执行文件 |
| `644` | `rw-r--r--` | 普通文件 |
| `600` | `rw-------` | SSH 私钥、敏感配置文件 |
| `700` | `rwx------` | 私有目录 |
| `777` | `rwxrwxrwx` | ⚠️ 任何人可读写执行，极少使用 |
| `400` | `r--------` | 只读（证书文件） |

**特殊权限位：**

| 位 | 数字前缀 | 符号 | 含义 |
|----|----------|------|------|
| SUID | `4` | `u+s` | 以文件所有者的身份执行 |
| SGID | `2` | `g+s` | 以文件所属组身份执行；目录中新文件继承组 |
| Sticky | `1` | `+t` | 只有所有者能删除（如 `/tmp`） |

```bash
# 常见组合
chmod +x script.sh                     # 加执行权限
chmod 755 ~/bin/                       # 设置目录权限
chmod 600 ~/.ssh/id_rsa                # 保护私钥
chmod -R 755 public_html/              # 递归设置
chmod go-w file.txt                    # 去掉组和其他人的写权限
chmod u+s /usr/bin/passwd              # SUID 示例（通常系统已设好）

# 查看特殊权限
ls -l /usr/bin/passwd   # -rwsr-xr-x  注意 s 替代了 x
ls -ld /tmp              # drwxrwxrwt  注意 t
```

---

### 3.2 `chown` — 修改所有者 ⭐⭐⭐⭐

```bash
chown [OPTION]... OWNER[:GROUP] FILE...
```

| 选项 | 作用 |
|------|------|
| `-R` | 递归 |
| `--reference=RFILE` | 参照 RFILE 的所有者设置 |

```bash
sudo chown alice file.txt
sudo chown alice:dev file.txt
sudo chown :dev file.txt            # 只改所属组
sudo chown -R alice:alice ~/project/
```

---

### 3.3 `sudo` — 以超级用户身份执行 ⭐⭐⭐⭐⭐

```bash
sudo [OPTION]... COMMAND
```

| 选项 | 作用 |
|------|------|
| `-u USER` | 以指定用户身份执行 |
| `-i` | 模拟初始登录（加载目标用户环境） |
| `-s` | 以目标用户的 shell 执行 |
| `-E` | 保留当前环境变量 |
| `-H` | 将 `$HOME` 设为目标用户的 home |

```bash
sudo apt install nginx
sudo -u postgres psql              # 以 postgres 用户执行
sudo -i                            # 进入 root shell（加载 root 环境）
sudo !!                            # 以 sudo 重新执行上一条命令
sudo visudo                        # 安全编辑 /etc/sudoers
```

---

### 3.4 用户与组管理 ⭐⭐

```bash
# 用户
id                                 # 显示 UID / GID / 所属组
whoami                             # 当前用户名
who                                # 当前登录的所有用户
w                                  # 更详细的登录用户信息

# 添加/修改用户
sudo useradd -m -s /bin/bash alice    # 创建用户（-m 创建 home）
sudo usermod -aG docker $USER         # 将用户加入附加组（务必 -a！）
sudo usermod -s /bin/zsh alice        # 修改登录 shell
sudo passwd alice                     # 修改密码
sudo userdel -r alice                 # 删除用户（-r 删除 home）

# 组
sudo groupadd dev                     # 创建组
sudo usermod -aG dev alice            # 将 alice 加入 dev 组
groups alice                          # 查看 alice 所属组

# 重新加载组（无需重新登录）
newgrp docker
# 或
exec su -l $USER
```

---

## 4. 进程管理

### 4.1 `ps` — 进程快照 ⭐⭐⭐⭐⭐

```bash
ps [OPTION]...
```

**最常用的两种格式：**

| 风格 | 示例 | 说明 |
|------|------|------|
| BSD (无 `-`) | `ps aux` | 显示所有用户的所有进程 |
| UNIX (`-`) | `ps -ef` | 同上，列略有不同 |
| UNIX (`-`) | `ps -eLf` | 显示所有线程 |

**`ps aux` 输出字段：**

```
USER   PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root     1   0.0  0.1 169416 13268 ?     Ss   Jul01   0:45 /sbin/init
|       |    |    |     |     |   |      |     |      |     |
|       |    |    |     |     |   |      |     |      |     +-- 命令
|       |    |    |     |     |   |      |     |      +-- 累计 CPU 时间
|       |    |    |     |     |   |      |     +-- 启动时间
|       |    |    |     |     |   |      +-- 状态码
|       |    |    |     |     |   +-- 终端（? 表示无终端）
|       |    |    |     |     +-- RSS：物理内存 (KB)
|       |    |    |     +-- VSZ：虚拟内存 (KB)
|       |    |    +-- 内存使用率
|       |    +-- CPU 使用率
|       +-- 进程 ID
+-- 用户名
```

**STAT 状态码：**

| 码 | 含义 |
|----|------|
| `R` | 正在运行或可运行 |
| `S` | 可中断睡眠（等待事件） |
| `D` | 不可中断睡眠（通常等待 I/O） |
| `Z` | 僵尸进程 |
| `T` | 被跟踪或停止 |
| `<` | 高优先级 |
| `N` | 低优先级 |
| `s` | 会话领导者 |
| `l` | 多线程 |
| `+` | 前台进程组 |

**主要选项：**

| 选项 | 作用 |
|------|------|
| `a` | 所有用户的进程（不仅是当前终端） |
| `u` | 以用户为中心的格式（%CPU, %MEM） |
| `x` | 包含无终端的进程（daemon） |
| `-e` | 所有进程 |
| `-f` | 完整格式 |
| `-L` | 显示线程 |
| `-o` | 自定义输出列 |
| `--sort=-%mem` | 按内存排序 |
| `-C CMD` | 按命令名过滤 |
| `-p PID` | 按 PID 过滤 |

```bash
# 最常用
ps aux | grep nginx

# 按内存占用 Top 20
ps aux --sort=-%mem | head -20

# 按 CPU 占用 Top 20
ps aux --sort=-%cpu | head -20

# 树状显示（父子进程关系）
ps auxf
# 或
ps -ejH

# 显示某进程的所有线程
ps -Lf -p $(pgrep firefox)

# 自定义输出格式
ps -eo pid,ppid,user,%cpu,%mem,comm --sort=-%cpu | head

# 按命令名查
ps -C sshd -o pid,user,%cpu,%mem,args
```

---

### 4.2 `top` / `htop` — 实时进程监控 ⭐⭐⭐⭐⭐

```bash
top [OPTION]
htop   # 需要单独安装 sudo apt install htop
```

`htop` 优势：彩色、鼠标可点击、可以直接 `F9` 杀进程、`F5` 树状显示、支持滚动。

**top 交互命令：**

| 键 | 功能 |
|----|------|
| `1` | 切换显示每个 CPU 核心 |
| `M` | 按内存使用排序 |
| `P` | 按 CPU 使用排序 |
| `T` | 按运行时间排序 |
| `k` | 杀死进程（输入 PID） |
| `r` | renice |
| `c` | 切换完整命令行 |
| `V` | 森林/树状显示 |
| `h` | 帮助 |
| `q` | 退出 |
| `u` | 过滤用户 |
| `L` | 搜索字符串 |

```bash
# 指定刷新间隔 2 秒
top -d 2

# 只显示某用户的进程
top -u alice

# 监控特定 PID
top -p 1234 -p 5678

# 批处理模式（非交互，输出到文件）
top -b -n 1 > top_snapshot.txt
```

---

### 4.3 `kill` / `pkill` / `killall` — 终止进程 ⭐⭐⭐⭐

```bash
kill [OPTION] PID...
kill -SIGNAL PID...
```

**常用信号：**

| 信号编号 | 信号名 | 作用 |
|----------|--------|------|
| `1` | `SIGHUP` | 挂断（daemon 常用来重新加载配置） |
| `2` | `SIGINT` | 中断（等同 Ctrl+C） |
| `3` | `SIGQUIT` | 退出并生成 core dump |
| `9` | `SIGKILL` | 强制终止（不可捕获，无法清理） |
| `15` | `SIGTERM` | 优雅终止（默认） |
| `18` | `SIGCONT` | 继续执行 |
| `19` | `SIGSTOP` | 停止（不可捕获） |

```bash
# 优雅终止
kill 1234
kill -TERM 1234

# 强制杀死
kill -9 1234

# 重新加载配置（nginx/sshd 等）
kill -HUP $(pgrep nginx)

# 按名称杀进程
pkill python
pkill -9 -f "python server.py"   # -f 匹配完整命令行

# 杀某个用户的所有进程
pkill -u alice

# 按名杀（精确匹配）
killall sshd

# 列出所有信号
kill -l
```

**何时用哪个：**

```
SIGTERM (15) → 先试这个，让进程有机会清理资源
SIGINT  (2)  → 等同 Ctrl+C
SIGKILL (9)  → 最后手段，进程没有机会清理
SIGHUP  (1)  → 对于 daemon，通常是"重新加载配置"
```

---

### 4.4 `nohup` — 忽略挂断信号 ⭐⭐⭐

```bash
nohup COMMAND [ARG]...
```

使进程在终端关闭后继续运行。需要配合 `&` 放到后台。

```bash
# 后台运行，不受终端关闭影响
nohup ./long_build.sh > build.log 2>&1 &

# 输出默认写入 nohup.out
nohup python train.py &
```

更好的替代方案：`screen` 或 `tmux`（可以重新 attach）。

---

### 4.5 前后台切换 ⭐⭐⭐

```bash
command &            # 后台启动
Ctrl+Z               # 挂起前台进程
bg                   # 将挂起进程放到后台继续
fg                   # 将后台进程拉到前台
fg %1                # 拉指定 job
jobs -l              # 列出所有后台 job 及其 PID
disown -h %1         # 使 job 脱离 shell（shell 退出后继续运行）
```

---

## 5. 磁盘与存储

### 5.1 `df` — 磁盘空间概览 ⭐⭐⭐⭐

```bash
df [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-h` | 人可读格式（K/M/G） |
| `-T` | 显示文件系统类型（ext4, xfs, tmpfs...） |
| `-t TYPE` | 只显示指定类型的文件系统 |
| `-x TYPE` | 排除指定类型 |
| `-i` | 显示 inode 使用情况 |
| `-l` | 只显示本地文件系统 |
| `--total` | 末尾加一行汇总 |

```bash
df -h                     # 日常查看
df -hT                    # 看文件系统类型
df -i                     # 检查 inode 耗尽（小文件太多）
df -h /home /var          # 看特定挂载点
df -h -x tmpfs -x devtmpfs # 排除临时文件系统
```

---

### 5.2 `du` — 目录空间分析 ⭐⭐⭐⭐

```bash
du [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-s` | 只显示汇总（summary） |
| `-h` | 人可读格式 |
| `-a` | 显示所有文件（不仅是目录） |
| `-c` | 末尾显示总计 |
| `--max-depth=N` | 限制递归深度 |
| `-d N` | 同上（GNU 风格） |
| `-b` | 以字节为单位 |
| `--time` | 显示修改时间 |
| `--exclude=PATTERN` | 排除匹配项 |
| `-x` | 不跨文件系统 |

```bash
# 当前目录下各子目录大小，排序
du -sh * | sort -rh

# 只看一层深（最有用的选项）
du -h --max-depth=1 ~/ | sort -rh

# 找出最大的 10 个目录
du -ah /var | sort -rh | head -10

# 排除缓存目录
du -sh --exclude=.git --exclude=node_modules project/

# 交互式探索（推荐替代 ncdu）
ncdu ~    # sudo apt install ncdu
```

---

### 5.3 `lsblk` — 列出块设备 ⭐⭐⭐

```bash
lsblk [OPTION]... [DEVICE]
```

| 选项 | 作用 |
|------|------|
| `-a` | 包括空设备 |
| `-f` | 显示文件系统和 UUID |
| `-o COLUMNS` | 自定义输出列 |
| `-m` | 显示权限/所有者 |
| `-t` | 树状显示拓扑 |

```bash
lsblk                     # 快速看所有块设备
lsblk -f                  # 显示 UUID 和文件系统类型
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,LABEL,UUID
```

---

### 5.4 `mount` / `umount` — 挂载与卸载 ⭐⭐⭐

```bash
mount [DEVICE] [MOUNTPOINT]
mount -t TYPE DEVICE MOUNTPOINT
umount DEVICE|MOUNTPOINT
```

| mount 选项 | 作用 |
|------------|------|
| `-t TYPE` | 指定文件系统类型 |
| `-o OPTIONS` | 挂载选项（ro=只读, rw, noexec, nosuid...） |
| `-o loop` | 挂载文件当块设备（.iso, .img） |
| `--bind` | 绑定挂载（将目录挂到另一个位置） |

```bash
# 挂载 U 盘
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb

# 挂载 ISO 镜像
sudo mount -o loop ubuntu.iso /mnt/iso

# 挂载 SD 卡镜像中的分区（嵌入式常用）
sudo mount -o loop,offset=$((START_SECTOR*512)) sdcard.img /mnt/part1

# 只读挂载，防止误写入
sudo mount -o ro /dev/sdb1 /mnt/readonly

# 强制卸载（设备忙时）
sudo umount -l /mnt/usb     # 懒卸载
sudo umount -f /mnt/usb     # 强制卸载
```

---

### 5.5 `dd` — 裸数据拷贝 ⭐⭐⭐

```bash
dd if=INPUT of=OUTPUT [OPTIONS]
```

| 选项 | 作用 |
|------|------|
| `bs=N` | 块大小（默认 512 字节，推荐 4M） |
| `count=N` | 拷贝块数 |
| `skip=N` | 跳过输入的前 N 块 |
| `seek=N` | 跳过输出的前 N 块 |
| `status=progress` | 显示进度 |
| `conv=fsync` | 完成后刷写缓存 |
| `conv=noerror,sync` | 读错误时继续并用零填充 |

```bash
# ⚠️ dd 是 "disk destroyer"，of= 参数要再三确认！

# 烧录镜像到 SD 卡
sudo dd if=rootfs.img of=/dev/sdb bs=4M status=progress conv=fsync

# 备份 MBR（前 512 字节）
sudo dd if=/dev/sda of=mbr.bak bs=512 count=1

# 生成随机文件
dd if=/dev/urandom of=random.bin bs=1M count=10

# 磁盘擦除（全写零）
sudo dd if=/dev/zero of=/dev/sdb bs=4M status=progress

# 只拷贝镜像中的特定分区（需要计算 offset）
fdisk -l image.img
dd if=image.img of=part1.img bs=512 skip=2048 count=131072
```

---

### 5.6 `fdisk` — 分区管理 ⭐⭐

```bash
sudo fdisk [OPTION] DEVICE
sudo fdisk -l [DEVICE]   # 列出分区表
```

| 子命令 | 作用 |
|--------|------|
| `p` | 打印分区表 |
| `n` | 新建分区 |
| `d` | 删除分区 |
| `w` | 写入并退出 |
| `q` | 不保存退出 |
| `t` | 修改分区类型 |

```bash
sudo fdisk -l                    # 列出所有磁盘分区
sudo fdisk -l /dev/sda           # 看指定磁盘
sudo fdisk -l image.img          # 看镜像的分区布局
```

---

## 6. 网络

### 6.1 `ping` — 测试连通性 ⭐⭐⭐⭐⭐

```bash
ping [OPTION] DESTINATION
```

| 选项 | 作用 |
|------|------|
| `-c N` | 发送 N 个包后停止 |
| `-i SEC` | 间隔秒数（默认 1，需要 sudo 才能 < 0.2） |
| `-s N` | 数据包大小（默认 56，不含头） |
| `-W SEC` | 超时等待时间 |
| `-I IFACE` | 指定网络接口 |
| `-f` | 洪泛模式（flood，需 sudo） |
| `-q` | 安静模式（只显示汇总） |
| `-4` / `-6` | 强制 IPv4 / IPv6 |

```bash
ping -c 4 8.8.8.8
ping -c 10 -i 0.5 -s 1024 192.168.1.1
ping -c 4 -W 2 example.com     # 2 秒超时
ping -c 100 -q 8.8.8.8          # 汇总统计
```

---

### 6.2 `curl` — 数据传输工具 ⭐⭐⭐⭐⭐

```bash
curl [OPTION]... [URL]...
```

支持 HTTP/HTTPS, FTP, SFTP, SCP, TFTP, TELNET, LDAP 等。

#### HTTP 常用选项

| 选项 | 作用 |
|------|------|
| `-X METHOD` | 指定方法（GET, POST, PUT, DELETE...），默认 GET |
| `-d DATA` | POST body（`-d '{"key":"val"}'`） |
| `-H HEADER` | 添加请求头（可多次使用） |
| `-o FILE` | 输出到文件 |
| `-O` | 输出到文件（用远程文件名） |
| `-I` | 只获取响应头（HEAD 请求） |
| `-i` | 输出包含响应头 |
| `-v` | 详细输出（含请求和响应头） |
| `-L` | 跟随重定向 |
| `-k` | 忽略 SSL 证书验证 |
| `-u USER:PASS` | 基本认证 |
| `-x PROXY` | 使用代理 |
| `-F field=value` | multipart form 上传 |
| `-w FORMAT` | 格式化输出（如 `%{http_code}`） |
| `-s` | 静默模式（不显示进度） |
| `-S` | 出错时显示错误（配合 `-s`） |
| `--connect-timeout SEC` | 连接超时 |
| `--max-time SEC` | 最大执行时间 |

```bash
# GET 请求
curl https://api.example.com/data

# 查看响应头
curl -I https://example.com

# POST JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"test"}' https://api.example.com/users

# 下载文件
curl -O https://example.com/file.tar.gz
curl -o myname.tar.gz https://example.com/file.tar.gz

# 静默 + 只输出 HTTP 状态码
curl -s -o /dev/null -w "%{http_code}" https://example.com

# 跟随重定向
curl -L https://short.link/abc

# 上传文件
curl -F "file=@image.png" https://upload.example.com

# API 测试（显示完整请求和响应）
curl -v https://api.example.com

# 不验证证书（内网自签证书用）
curl -k https://192.168.1.100/api
```

---

### 6.3 `wget` — 文件下载 ⭐⭐⭐⭐

```bash
wget [OPTION]... [URL]...
```

| 选项 | 作用 |
|------|------|
| `-c` | 断点续传 |
| `-O FILE` | 指定输出文件名 |
| `-P DIR` | 下载到指定目录 |
| `-r` | 递归下载 |
| `-np` | 不递归到上级目录 |
| `-nd` | 不创建目录层级 |
| `-A PATTERN` | 只接受匹配文件（`-A "*.pdf"`） |
| `-R PATTERN` | 排除匹配文件 |
| `--limit-rate=N` | 限速（如 `--limit-rate=200k`） |
| `-q` | 安静模式 |
| `-nv` | 非详细（简洁进度） |

```bash
wget -c https://example.com/large_file.iso   # 断点续传
wget -O output.html https://example.com
wget -r -np -nd -A "*.pdf" https://example.com/docs/
```

---

### 6.4 `ip` — 网络配置（替代 ifconfig） ⭐⭐⭐⭐

```bash
ip [OPTION] OBJECT COMMAND
# OBJECT: addr, link, route, neigh, rule, netns, ...
```

#### ip addr — IP 地址

```bash
ip a                           # 简写，显示所有接口 IP
ip addr show                   # 同上
ip addr show eth0              # 只看 eth0
ip -4 a                        # 只看 IPv4
ip -6 a                        # 只看 IPv6
ip -br a                       # 简洁单行输出
ip -c a                        # 彩色输出
```

#### ip link — 链路层

```bash
ip link                        # 所有接口状态
ip link show eth0
ip link set eth0 up            # 启用
ip link set eth0 down          # 禁用
ip link set eth0 mtu 9000      # 修改 MTU
```

#### ip route — 路由表

```bash
ip route                       # 查看路由表
ip route show default          # 看默认网关
ip route add 10.0.0.0/24 via 192.168.1.1 dev eth0
ip route del 10.0.0.0/24
```

#### 其他

```bash
ip neigh                       # ARP 表（替代 arp -a）
ip netns list                  # 网络命名空间
sudo ip netns add ns1          # 创建命名空间
```

---

### 6.5 `ss` — Socket 统计（替代 netstat） ⭐⭐⭐⭐

```bash
ss [OPTION]... [FILTER]
```

| 选项 | 作用 |
|------|------|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | 仅监听中的 sockets |
| `-a` | 所有 sockets（监听 + 已连接） |
| `-n` | 不解析服务名（显示端口号） |
| `-p` | 显示进程信息（需 sudo） |
| `-s` | 汇总统计 |
| `-4` / `-6` | 仅 IPv4 / IPv6 |
| `-e` | 扩展信息 |
| `-m` | 内存使用 |
| `state FILTER` | 按状态过滤 |

```bash
# 查看谁在监听哪些端口
ss -tlnp

# 所有 TCP 连接
ss -tan

# 已建立的 TCP 连接
ss -tn state established

# 看某端口的进程
ss -tlnp sport = :80

# 汇总统计
ss -s

# 比较：旧式 netstat 等价命令
# netstat -tlnp  →  ss -tlnp
# netstat -an    →  ss -tan
```

---

### 6.6 `ssh` — 安全 Shell ⭐⭐⭐⭐⭐

```bash
ssh [OPTION]... [USER@]HOST [COMMAND]
```

#### 常用选项

| 选项 | 作用 |
|------|------|
| `-p PORT` | 指定端口（默认 22） |
| `-i KEYFILE` | 指定私钥 |
| `-X` | 转发 X11 |
| `-Y` | 可信 X11 转发 |
| `-L [BIND:]PORT:HOST:PORT` | 本地端口转发 |
| `-R [BIND:]PORT:HOST:PORT` | 远程端口转发 |
| `-D [BIND:]PORT` | 动态端口转发（SOCKS 代理） |
| `-N` | 不执行远程命令（端口转发专用） |
| `-f` | SSH 后台运行 |
| `-C` | 压缩传输 |
| `-v` / `-vv` / `-vvv` | 详细调试输出 |
| `-o OPTION` | 自定义配置 |
| `-A` | 转发认证代理（慎用） |
| `-t` | 强制分配伪终端 |

```bash
# 基本连接
ssh alice@192.168.1.100
ssh -p 2222 alice@example.com

# 指定私钥
ssh -i ~/.ssh/id_ed25519 alice@host

# 执行远程命令（不登录）
ssh alice@host "ls -la /var/log"
ssh alice@host 'bash -s' < local_script.sh   # 远程执行本地脚本

# 端口转发
# 本地 8080 → 远程 localhost:80（访问本地 8080 等于访问远程 localhost:80）
ssh -L 8080:localhost:80 alice@host -N

# 远程 9090 → 本地 localhost:3000（让远程机器通过 9090 访问你本地的 3000）
ssh -R 9090:localhost:3000 alice@host -N

# SOCKS 代理（本地 1080 端口）
ssh -D 1080 alice@host -N

# 调试连接问题
ssh -vvv alice@host

# 跳过主机密钥检查（首次连接或临时）
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null alice@host

# 保持连接（防 timeout）
ssh -o ServerAliveInterval=60 alice@host
```

---

### 6.7 `scp` — 远程文件拷贝 ⭐⭐⭐⭐

```bash
scp [OPTION]... SOURCE... DEST
```

| 选项 | 作用 |
|------|------|
| `-P PORT` | 指定端口（注意大写） |
| `-r` | 递归拷贝目录 |
| `-p` | 保留修改时间和权限 |
| `-C` | 压缩传输 |
| `-i KEYFILE` | 指定私钥 |
| `-q` | 安静模式 |

```bash
# 上传
scp local_file.txt alice@host:/home/alice/

# 下载
scp alice@host:/var/log/syslog ./

# 递归目录
scp -r project/ alice@host:~/backup/

# 指定端口
scp -P 2222 file alice@host:~/
```

---

### 6.8 `rsync` — 高级同步 ⭐⭐⭐⭐

```bash
rsync [OPTION]... SOURCE... DEST
```

| 选项 | 作用 |
|------|------|
| `-a` | 归档模式 = `-rlptgoD`（保留所有属性） |
| `-v` | 详细输出 |
| `-z` | 压缩传输 |
| `-P` | `--partial --progress`（断点续传 + 进度） |
| `-n` | dry run（预览不执行） |
| `--delete` | 删除目标有但源没有的文件 |
| `--exclude=PATTERN` | 排除匹配项 |
| `--max-size=SIZE` | 限制最大文件大小 |
| `--bwlimit=RATE` | 限速 |
| `-e "ssh -p 2222"` | 指定 ssh 端口 |

```bash
# 最常用：同步目录
rsync -avz --progress src/ dst/

# 远程同步（推送）
rsync -avzP ~/project/ alice@host:~/project/

# 远程同步（拉取）
rsync -avzP alice@host:~/data/ ~/data/

# 模拟运行，看哪些文件会被传
rsync -avzn --delete src/ dst/

# 排除
rsync -avz --exclude=".git" --exclude="build/" project/ backup/

# 限速 1MB/s
rsync -avz --bwlimit=1000 large_data/ alice@host:~/data/
```

**`rsync` 尾部斜杠陷阱：**
```
rsync -a src/ dest/    # 拷贝 src 的内容到 dest/（不创建 src 目录）
rsync -a src dest/     # 拷贝 src 目录到 dest/ 下（创建 dest/src/）
```

---

### 6.9 DNS 查询 ⭐⭐⭐

```bash
# dig — 功能最全的 DNS 工具
dig example.com                           # 查询 A 记录
dig +short example.com                    # 只输出结果
dig example.com MX                        # 查询 MX 记录
dig -x 8.8.8.8                            # 反向查询
dig @1.1.1.1 example.com                  # 指定 DNS 服务器
dig +trace example.com                    # 跟踪 DNS 解析链

# nslookup — 更简单的 DNS 查询
nslookup example.com
nslookup example.com 8.8.8.8

# host — 最简 DNS
host example.com
host 8.8.8.8                              # 反向查询

# 日常检查解析
getent hosts example.com
getent ahosts example.com
```

---

### 6.10 `nc` (netcat) — 网络瑞士军刀 ⭐⭐⭐

```bash
nc [OPTION]... HOST PORT
```

| 选项 | 作用 |
|------|------|
| `-l` | 监听模式（服务端） |
| `-p PORT` | 本地端口 |
| `-v` / `-vv` | 详细输出 |
| `-z` | 端口扫描模式（不发送数据） |
| `-w SEC` | 超时 |
| `-u` | UDP 模式 |
| `-k` | 持续监听（client 断开后不退出） |
| `-n` | 不解析 DNS |

```bash
# 端口连通性测试
nc -zv 192.168.1.10 22
nc -zv 192.168.1.10 20-100     # 端口范围扫描

# 简单 TCP 服务端
nc -l 1234                       # 客户端 nc host 1234

# 简单文件传输
# 接收端
nc -l 1234 > received_file
# 发送端
nc host 1234 < file_to_send

# HTTP 请求测试
printf "GET / HTTP/1.0\r\n\r\n" | nc example.com 80

# UDP 测试
nc -u -l 1234                    # UDP 服务器
nc -u host 1234                  # UDP 客户端
```

---

## 7. 压缩与归档

### 7.1 `tar` — 归档 ⭐⭐⭐⭐⭐

```bash
tar [OPTION]... [FILE]...
```

#### 必需选项（每次选一个）

| 选项 | 长选项 | 作用 |
|------|--------|------|
| `-c` | `--create` | 创建归档 |
| `-x` | `--extract` | 解压 |
| `-t` | `--list` | 列出内容 |
| `-r` | `--append` | 追加到已有归档 |
| `-u` | `--update` | 只追加较新的文件 |
| `-A` | `--catenate` | 合并两个归档 |

#### 常用辅助选项

| 选项 | 作用 |
|------|------|
| `-v` | 详细输出（列出处理的文件） |
| `-f FILE` | 指定归档文件（`-f -` 表示 stdin/stdout） |
| `-C DIR` | 切换到 DIR 后再操作 |
| `-z` | 通过 gzip 压缩/解压 |
| `-j` | 通过 bzip2 压缩/解压 |
| `-J` | 通过 xz 压缩/解压 |
| `-a` | 根据扩展名自动选压缩方式 |

#### 其他常用选项

| 选项 | 作用 |
|------|------|
| `-p` | 保留文件权限 |
| `--strip-components=N` | 解压时去掉前 N 层目录 |
| `--exclude=PATTERN` | 排除文件 |
| `-k` | 解压时不覆盖已存在文件 |
| `--overwrite` | 解压时覆盖 |
| `--remove-files` | 归档后删除原文件 |
| `--transform` | 在归档/解压时重命名 |

```bash
# === 创建 ===
tar -czvf archive.tar.gz dir/          # gzip 压缩
tar -cjvf archive.tar.bz2 dir/         # bzip2 压缩
tar -cJvf archive.tar.xz dir/          # xz 压缩
tar -cavf archive.tar.gz dir/          # 自动选压缩方式 (GNU tar)

# === 解压 ===
tar -xzvf archive.tar.gz               # 解压 gzip
tar -xzvf archive.tar.gz -C /tmp       # 解压到指定目录
tar -xzvf archive.tar.gz --strip-components=1  # 去掉顶层目录

# === 查看 ===
tar -tzvf archive.tar.gz               # 不解压，列出内容
tar -tzvf archive.tar.gz | grep pattern

# === 高级用法 ===
# 解压单个文件
tar -xzvf archive.tar.gz path/to/specific/file.txt

# 排除某些文件
tar -czvf project.tar.gz --exclude=".git" --exclude="*.o" project/

# 管道用法：远程备份
tar -czvf - important_data/ | ssh user@host "cat > backup.tar.gz"

# 管道用法：不解压直接传输
ssh user@host "tar -czvf - /var/log" | tar -xzvf - -C /tmp/
```

---

### 7.2 `gzip` / `gunzip` ⭐⭐⭐

```bash
gzip [OPTION]... FILE...       # 压缩（替换原文件）
gunzip [OPTION]... FILE...     # 解压
zcat FILE.gz                   # 直接查看压缩文件内容
```

| 选项 | 作用 |
|------|------|
| `-k` | 保留原文件 |
| `-c` | 输出到 stdout |
| `-d` | 解压模式 |
| `-N` | 压缩级别（1 最快，9 最省，默认 6） |
| `-v` | 显示压缩比 |

```bash
gzip -k large_file.log       # 压缩但保留原文件
gzip -9 data.txt              # 最大压缩
zcat file.gz | grep "error"   # 不解压直接搜索
```

---

### 7.3 `zip` / `unzip` ⭐⭐⭐

```bash
zip [OPTION]... ZIPFILE FILE...
unzip [OPTION]... ZIPFILE
```

| zip 选项 | 作用 |
|----------|------|
| `-r` | 递归目录 |
| `-e` | 加密 |
| `-P PASS` | 密码（注意命令行可见） |
| `-x PATTERN` | 排除文件 |
| `-s SIZE` | 分卷 |

| unzip 选项 | 作用 |
|------------|------|
| `-l` | 列出内容 |
| `-d DIR` | 解压到指定目录 |
| `-o` | 覆盖不询问 |
| `-n` | 不覆盖 |

---

## 8. 系统信息与监控

### 8.1 系统概况

```bash
uname -a                    # 内核版本 + 架构
uname -r                    # 仅内核版本
lsb_release -a              # Ubuntu 发行版信息
cat /etc/os-release         # 系统信息（systemd 标准）
hostnamectl                 # 主机名 + 系统信息
uptime                      # 运行时间、平均负载
```

**`uptime` 输出解读：**
```
15:30:00 up 30 days,  2:15,  3 users,  load average: 0.15, 0.25, 0.30
                        |      |        |                |     |     |
                        运行30天 空闲2h15m 3个用户           1分钟  5分钟  15分钟
```
负载 < CPU 核心数 = 正常；负载持续 > 核心数 = 需要排查。

---

### 8.2 CPU 信息

```bash
lscpu                                  # CPU 架构详表
cat /proc/cpuinfo                      # 每个核心的详细参数
nproc                                  # CPU 核心数（make -j 常用）
lscpu | grep -E "Model name|Socket|Core|Thread|MHz"
```

---

### 8.3 内存

```bash
free -h                    # 人可读格式
free -h -s 2               # 每 2 秒刷新
free -h -w                 # 区分 buffers 和 cache（新版已默认分离）
```

**`free -h` 输出解读：**
```
               total    used    free    shared    buff/cache   available
Mem:           15Gi    4.2Gi   3.1Gi   1.0Gi     8.2Gi         10Gi
Swap:         2.0Gi    0.0Gi   2.0Gi
```
- `used`: 实际被进程使用的内存
- `buff/cache`: 内核缓存，需要时可释放
- `available`: 对新进程实际可用的量（比 `free` 更能反映真实可用内存）

---

### 8.4 设备列表

```bash
# USB 设备
lsusb                     # 列表
lsusb -t                  # 树状（看拓扑：哪个口挂了什么）
lsusb -v                  # 详细

# PCI/PCIe 设备
lspci                     # 列表
lspci -k                  # 显示驱动（看哪个内核模块驱动了该设备）
lspci -vv                 # 详细
lspci -nn                 # 含 vendor/device ID

# 块设备
lsblk                     # 见 5.3 节

# DMI (BIOS/主板/序列号)
sudo dmidecode -t system
sudo dmidecode -t memory
```

---

### 8.5 内核日志

```bash
dmesg                     # 全部内核环缓冲区日志
dmesg -w                  # 实时跟踪（插拔 USB 时有用）
dmesg --level=err,warn    # 只看错误和警告
dmesg -T                  # 显示人类可读时间戳
dmesg | grep -i "usb\|error\|fail"
dmesg | tail -50
```

---

### 8.6 systemd 日志

```bash
journalctl [OPTION]... [MATCHES]

# 核心选项
journalctl -u ssh.service                    # 看指定服务
journalctl -u ssh.service -f                 # 实时跟踪
journalctl -u ssh.service -n 50              # 最近 50 行
journalctl -u ssh.service --since "1 hour ago"
journalctl -u ssh.service --since "2026-07-19 08:00:00" --until "09:00:00"
journalctl -b                                # 本次启动以来的日志
journalctl -b -1                             # 上次启动的日志
journalctl -p err                            # 只看错误级别
journalctl -p err..emerg                     # 错误到紧急
journalctl _PID=1234                         # 按 PID 过滤
journalctl /usr/bin/nginx                    # 按可执行文件过滤
journalctl --no-pager                        # 不分页（管道时用）
journalctl --disk-usage                      # 日志占用空间
journalctl --vacuum-size=500M                # 清理日志至 500M
journalctl --vacuum-time=7d                  # 只保留 7 天

# 组合
journalctl -u nginx -p err --since today --no-pager
```

---

## 9. 文本处理

### 9.1 `sed` — 流编辑器 ⭐⭐⭐

```bash
sed [OPTION]... 'SCRIPT' [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-i` | 原地修改文件 |
| `-i.bak` | 原地修改，备份原文件为 `.bak` |
| `-n` | 安静模式（默认打印所有行） |
| `-e SCRIPT` | 多个脚本 |
| `-E` 或 `-r` | 扩展正则 |

**常用 script 模式：**

| 模式 | 示例 | 说明 |
|------|------|------|
| `s/old/new/` | `s/foo/bar/` | 替换（每行第一个） |
| `s/old/new/g` | `s/foo/bar/g` | 替换（每行所有） |
| `s/old/new/N` | `s/foo/bar/2` | 替换第 N 个匹配 |
| `/pattern/d` | `/^#/d` | 删除匹配行 |
| `/pattern/p` | `/error/p` | 打印匹配行 |
| `N,Md` | `1,5d` | 删除第 N 到 M 行 |
| `N,Ms/old/new/` | `10,20s/foo/bar/` | 第 N 到 M 行替换 |

```bash
# 替换（最常用）
sed 's/foo/bar/g' file.txt
sed -i 's/foo/bar/g' *.txt          # ⚠️ 原地修改多个文件

# 删除注释行
sed '/^#/d' config.conf

# 删除空行
sed '/^$/d' file.txt

# 提取第 5 到 10 行
sed -n '5,10p' file.txt

# 在每行前添加字符串
sed 's/^/PREFIX: /' file.txt

# 删除行尾空格
sed -i 's/[[:space:]]*$//' file.txt

# MAC 地址格式化：XX:XX:XX:XX:XX:XX → XX-XX-XX-XX-XX-XX
sed 's/:/-/g' macs.txt

# 多脚本
sed -e 's/foo/bar/' -e '/^#/d' file.txt
```

---

### 9.2 `awk` — 文本处理语言 ⭐⭐⭐

```bash
awk [OPTION]... 'PROGRAM' [FILE]...
```

`awk` 按行处理，自动将每行按分隔符拆分为 `$1, $2, ...`，`$0` 是整行。

| 选项 | 作用 |
|------|------|
| `-F FS` | 指定字段分隔符（默认空白） |
| `-v VAR=VAL` | 设置变量 |

**特殊变量：**

| 变量 | 含义 |
|------|------|
| `NR` | 当前行号 |
| `NF` | 当前行的字段数 |
| `FS` | 字段分隔符 |
| `OFS` | 输出字段分隔符 |
| `ORS` | 输出记录分隔符 |
| `FILENAME` | 当前文件名 |

**模式：**

```bash
# 打印第 1 和第 3 列
awk '{print $1, $3}' data.txt

# 指定分隔符
awk -F: '{print $1, $3, $7}' /etc/passwd

# 条件过滤
awk '$3 > 1000 {print $1}' /etc/passwd     # UID > 1000
awk '/error/ {print $0}' app.log            # 含 error 的行

# 统计
awk '{sum += $2} END {print sum}' data.txt   # 第 2 列求和
awk '{count++} END {print count}' file       # 行数

# 格式化输出
awk '{printf "%-20s %10d\n", $1, $2}' data.txt

# 按行号
awk 'NR>=10 && NR<=20' file.txt              # 第 10-20 行
awk 'NR==1{print}' file.txt                  # 第一行

# 重组字段
awk -F: -v OFS='|' '{print $1, $3, $7}' /etc/passwd
```

```bash
# 实战：Top 10 访问 IP
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# 实战：统计平均响应时间
awk '{sum+=$NF; count++} END {print sum/count " ms"}' response_times.txt

# 实战：合并多行（将每 3 行合并为一行）
awk 'NR%3{printf "%s ", $0; next} {print $0}' file.txt
```

---

### 9.3 `tee` — 分流输出 ⭐⭐⭐

```bash
tee [OPTION]... [FILE]...
```

| 选项 | 作用 |
|------|------|
| `-a` | 追加模式 |
| `-i` | 忽略中断信号 |

```bash
# 同时输出到终端和文件
make 2>&1 | tee build.log

# 追加到文件，同时在终端显示
./script.sh | tee -a log.txt

# 多个文件
./script.sh | tee log1.txt log2.txt

# 带 sudo 写保护文件
echo "new line" | sudo tee -a /etc/config.conf
```

---

### 9.4 `tr` — 字符替换/删除 ⭐⭐⭐

```bash
tr [OPTION]... SET1 [SET2]
```

| 选项 | 作用 |
|------|------|
| `-d` | 删除字符 |
| `-s` | 压缩重复字符 |
| `-c` | 取反（补集） |

```bash
# 大小写转换
echo "Hello World" | tr '[:upper:]' '[:lower:]'    # hello world
echo "Hello World" | tr 'A-Z' 'a-z'                # 同上

# 删除 Windows 换行符 \r
tr -d '\r' < dos_file.txt > unix_file.txt

# 压缩空行
cat file.txt | tr -s '\n'

# 将 PATH 按行输出
echo $PATH | tr ':' '\n'
```

---

## 10. Shell 内置与脚本

### 10.1 `echo` ⭐⭐⭐⭐⭐

```bash
echo [OPTION]... [STRING]...
```

| 选项 | 作用 |
|------|------|
| `-n` | 不输出末尾换行 |
| `-e` | 解析转义字符（`\n`, `\t`, `\033` 等） |
| `-E` | 不解析转义（默认） |

```bash
echo "Hello World"
echo -n "No newline"
echo -e "Line1\nLine2\nLine3"
echo -e "\033[1;31mRed Bold Text\033[0m"       # 彩色输出
echo "PATH is: $PATH"                            # 变量展开
echo "Files: $(ls | wc -l)"                      # 命令替换
```

---

### 10.2 `export` — 环境变量 ⭐⭐⭐⭐⭐

```bash
export NAME=VALUE
export NAME                # 仅标记为环境变量（值已设置）
```

```bash
# 永久生效：写入 ~/.bashrc 或 ~/.profile
export PATH=$PATH:/opt/gcc-arm/bin
export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm

# 临时：仅对本条命令有效
CROSS_COMPILE=arm-linux-gnueabihf- make
env VAR=value ./script.sh

# 查看所有环境变量
env | sort
printenv
```

---

### 10.3 `history` — 命令历史 ⭐⭐⭐⭐⭐

```bash
history [N]                # 显示最近 N 条
history -c                 # 清除历史
history -d N               # 删除第 N 条
```

**历史扩展（bang）：**

| 快捷键 | 作用 |
|--------|------|
| `!!` | 上一条命令 |
| `!$` | 上条命令的最后一个参数 |
| `!*` | 上条命令的所有参数 |
| `!grep` | 最近以 grep 开头的命令 |
| `!123` | 历史中编号 123 的命令 |
| `!-2` | 倒数第 2 条命令 |
| `^old^new^` | 替换上条命令中的 old 为 new |
| `Ctrl+R` | 反向交互搜索历史 |

```bash
# 搜索历史中包含某关键词的命令
history | grep "docker"

# 复用上条命令的参数
mkdir /very/long/path/name
cd !$                           # cd /very/long/path/name

# 快速修正
grep "eror" log.txt             # 手误
^eror^error^                    # 执行 grep "error" log.txt
```

---

### 10.4 `xargs` — 标准输入转命令行参数 ⭐⭐⭐

```bash
xargs [OPTION]... [COMMAND]
```

| 选项 | 作用 |
|------|------|
| `-n N` | 每次传 N 个参数 |
| `-I {}` | 用 `{}` 做占位符 |
| `-0` | 输入以 NUL 分隔（配合 `find -print0`） |
| `-P N` | 并行执行 N 个进程 |
| `-t` | 打印执行的命令 |
| `-p` | 打印并询问 |
| `-r` | 无输入时不执行命令 |

```bash
# 批量删除
find . -name "*.o" | xargs rm

# 处理含空格的文件名
find . -name "*.txt" -print0 | xargs -0 grep "TODO"

# 批量重命名（占位符）
ls *.jpeg | xargs -I {} mv {} {}.old

# 并行压缩
find . -name "*.log" -print0 | xargs -0 -P 4 -I {} gzip {}

# 批量创建目录
echo "dir1 dir2 dir3" | xargs mkdir -p

# 预览会执行的命令（安全先用）
find . -name "*.tmp" | xargs -t rm
```

---

### 10.5 `watch` — 周期性执行 ⭐⭐⭐

```bash
watch [OPTION]... COMMAND
```

| 选项 | 作用 |
|------|------|
| `-n SEC` | 刷新间隔（默认 2 秒） |
| `-d` | 高亮差异 |
| `-t` | 不显示标题头 |
| `-g` | 当输出变化时退出 |
| `-e` | 命令出错时退出 |

```bash
# 监控
watch -n 1 'free -h'
watch -n 1 'lsusb'                        # 插拔 USB 时
watch -n 2 'df -h'
watch -n 1 'ss -tnp'

# 组合多条命令
watch -n 1 'echo "=== Memory ===" && free -h && echo "=== Top CPU ===" && ps aux --sort=-%cpu | head -5'
```

---

### 10.6 `alias` / `unalias` ⭐⭐⭐⭐

```bash
alias NAME='command'       # 创建别名
unalias NAME               # 删除别名
alias                      # 列出所有别名
\command                   # 绕过别名执行原命令
```

```bash
# 常用推荐
alias ll='ls -alFh'
alias la='ls -Ah'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'

# 嵌入式开发实用别名
alias build='make -j$(nproc)'
alias flash='sudo dd if=output/image.img of=/dev/sdb bs=4M status=progress conv=fsync'

# 危险命令增加确认
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```

---

## 11. systemd 服务

### 11.1 `systemctl` ⭐⭐⭐⭐⭐

```bash
systemctl [COMMAND] [UNIT]
```

| 子命令 | 说明 |
|--------|------|
| `start UNIT` | 启动 |
| `stop UNIT` | 停止 |
| `restart UNIT` | 重启 |
| `reload UNIT` | 重载配置（不重启服务） |
| `status UNIT` | 查看状态 |
| `enable UNIT` | 开机自启 |
| `disable UNIT` | 禁止开机自启 |
| `is-enabled UNIT` | 是否已开启自启 |
| `is-active UNIT` | 是否正在运行 |
| `list-units` | 列出所有已加载的单元 |
| `list-units --state=failed` | 列出失败的单元 |
| `list-unit-files` | 列出所有已安装的单元文件 |
| `mask UNIT` | 屏蔽（彻底禁止启动） |
| `unmask UNIT` | 取消屏蔽 |
| `daemon-reload` | 重载 systemd 配置（修改 unit 文件后） |
| `edit UNIT --full` | 编辑 unit 文件 |
| `show UNIT` | 显示所有属性 |

```bash
# 标准操作
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx         # 不掉连接重载配置
systemctl status nginx
sudo systemctl enable nginx
sudo systemctl enable --now nginx   # 启用并立即启动

# 排查
systemctl status ssh --no-pager -l  # 完整日志
systemctl list-units --state=failed # 有哪些服务挂了
systemctl list-dependencies nginx   # 查看依赖
```

---

### 11.2 `systemd-analyze` — 启动分析 ⭐⭐

```bash
systemd-analyze                              # 总启动时间
systemd-analyze blame                        # 耗时排序
systemd-analyze critical-chain               # 关键路径
systemd-analyze plot > boot.svg              # 启动过程可视化图表
```

---

## 12. 开发工具链

### 12.1 `gcc` / `g++` — 编译 ⭐⭐⭐⭐⭐

```bash
gcc [OPTION]... FILE...
```

**编译流程四阶段：**

```
.c  ──预处理器(Preprocessor)──> .i  ──编译器(Compiler)──> .s  ──汇编器(Assembler)──> .o  ──链接器(Linker)──> ELF
    -E                            -S                            -c                            (默认)
```

| 选项 | 阶段 | 作用 |
|------|------|------|
| `-E` | 预处理 | 只预处理，输出到 stdout |
| `-S` | 编译 | 生成汇编 `.s` 文件 |
| `-c` | 汇编 | 生成目标 `.o` 文件，不链接 |
| `-o FILE` | | 指定输出文件名 |

**常用编译选项：**

| 选项 | 作用 |
|------|------|
| `-Wall` | 开启常见警告 |
| `-Wextra` | 额外警告 |
| `-Werror` | 警告当错误 |
| `-g` | 生成调试信息（gdb 用） |
| `-O0 / -O1 / -O2 / -O3 / -Os / -Og` | 优化级别 |
| `-std=c11 / -std=c++17` | 指定标准 |
| `-D NAME[=VALUE]` | 定义宏 |
| `-I DIR` | 头文件搜索路径 |
| `-L DIR` | 库搜索路径 |
| `-l NAME` | 链接 libNAME.so（`-lm` = libm, `-lpthread`） |
| `-static` | 静态链接 |
| `-shared` | 生成共享库 |
| `-fPIC` | 位置无关代码（共享库必需） |
| `-v` | 显示编译过程详情 |
| `-M` / `-MM` | 生成依赖关系 |
| `--sysroot=DIR` | 交叉编译 sysroot |
| `-march=ARCH` | 目标架构（`armv7-a`, `armv8-a`...） |
| `-mcpu=CPU` | 目标 CPU |
| `-mfpu=FPU` | 浮点单元（`neon`, `vfpv4`...） |

```bash
# 简单编译
gcc -o app main.c utils.c

# 分步编译
gcc -c main.c           # → main.o
gcc -c utils.c          # → utils.o
gcc -o app main.o utils.o

# 启用警告 + 调试
gcc -Wall -Wextra -g -O0 -o app main.c

# Release
gcc -Wall -O2 -o app main.c

# 链接数学库
gcc -o app main.c -lm

# 交叉编译 ARM
arm-linux-gnueabihf-gcc -mcpu=cortex-a9 -mfpu=neon -mfloat-abi=hard \
    --sysroot=/path/to/sysroot -o app main.c

# 查看预处理结果
gcc -E -dM main.c        # 列出所有预定义宏

# 生成依赖
gcc -MM main.c           # main.o: main.c header.h
```

---

### 12.2 `make` — 自动化构建 ⭐⭐⭐⭐⭐

```bash
make [OPTION]... [TARGET]...
```

| 选项 | 作用 |
|------|------|
| `-j N` | 并行 N 个 job |
| `-j$(nproc)` | 自动并行（用满 CPU） |
| `-C DIR` | 切换到 DIR 后执行 |
| `-f FILE` | 指定 Makefile |
| `-n` | dry run（只打印不执行） |
| `-B` | 无条件重新构建所有目标 |
| `-s` | 安静模式（不打印命令） |
| `-k` | 出错后继续构建其他目标 |
| `V=1` | 显示完整编译命令（Linux 内核等） |
| `-p` | 打印所有变量和规则 |

```bash
make                        # 构建默认目标
make -j$(nproc)             # 全速编译
make clean                  # 清理
make install                # 安装（通常需 sudo）
make -C build               # 在 build 目录执行
make V=1                    # 查看完整命令
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-

# 传递变量给 Makefile
make CFLAGS="-Wall -g" LDFLAGS="-lm"
```

---

### 12.3 `cmake` — 跨平台构建 ⭐⭐⭐⭐

```bash
# 配置
cmake -S . -B build                           # 源码目录 + 构建目录
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/opt/myapp
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake

# 构建
cmake --build build
cmake --build build -j$(nproc)
cmake --build build --target clean
cmake --build build --verbose                 # 显示完整编译命令

# 安装
cmake --install build
cmake --install build --prefix /opt/custom

# 列出所有可配置的选项
cmake -L build
cmake -LH build                               # 含帮助

# 交叉编译 toolchain 示例 (toolchain.cmake)
# set(CMAKE_SYSTEM_NAME Linux)
# set(CMAKE_SYSTEM_PROCESSOR arm)
# set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
# set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)
# set(CMAKE_SYSROOT /path/to/sysroot)
```

---

### 12.4 `gdb` — 调试器 ⭐⭐⭐⭐

```bash
gdb [OPTION]... [PROGRAM]
```

| 选项 | 作用 |
|------|------|
| `-q` | 安静模式 |
| `-ex CMD` | 启动时执行命令 |
| `-p PID` | attach 到运行中的进程 |
| `-c COREFILE` | 分析 core dump |
| `--args` | 后面跟程序和参数 |
| `-tui` | 文本用户界面 |

**核心调试命令：**

| 命令 | 简写 | 作用 |
|------|------|------|
| `run [args]` | `r` | 运行程序 |
| `break file:line` | `b` | 设断点 |
| `break func` | `b main` | 在函数入口设断点 |
| `delete N` | `d` | 删除断点 |
| `continue` | `c` | 继续执行 |
| `step` | `s` | 单步执行（进入函数） |
| `next` | `n` | 单步执行（不进入函数） |
| `finish` | `fin` | 执行到当前函数返回 |
| `print expr` | `p` | 打印表达式 |
| `info locals` | | 打印局部变量 |
| `info registers` | `i r` | 显示寄存器 |
| `backtrace` | `bt` | 调用栈 |
| `frame N` | `f` | 切换栈帧 |
| `list` | `l` | 显示源码 |
| `watch var` | | 观察变量变化 |
| `set var=val` | | 修改变量值 |
| `disassemble` | `disas` | 反汇编 |
| `x/NF ADDR` | `x` | 查看内存 |

```bash
# 启动调试
gdb ./a.out
gdb --args ./a.out --verbose --output=out.txt
gdb -p $(pgrep myapp)              # attach
gdb -tui ./a.out                   # TUI 模式（源码窗口）

# 常用启动脚本
gdb -ex "break main" -ex "run" ./a.out

# 分析 core dump
gdb ./a.out core
# 然后：bt, info registers, info locals, x/...

# 交叉调试（嵌入式）
gdb-multiarch ./firmware.elf
# (gdb) target remote localhost:1234
# (gdb) monitor reset
# (gdb) load
# (gdb) continue
```

**x (examine) 命令格式：** `x/NFU ADDR`
- N: 显示数量
- F: 格式 (x=hex, d=dec, s=string, i=指令)
- U: 单元 (b=byte, h=halfword, w=word, g=giant)

```bash
(gdb) x/16x $sp        # 查看栈顶 16 个字（十六进制）
(gdb) x/10i $pc        # 查看当前 PC 附近的 10 条指令
(gdb) x/s $r0          # 以字符串形式查看 r0 指向的内存
```

---

### 12.5 `git` — 版本控制 ⭐⭐⭐⭐⭐

这里只列高频子命令和选项，完整说明见 `reference/resources.md`。

```bash
# === 仓库操作 ===
git clone URL [DIR]                # 克隆
git init                           # 初始化

# === 查看状态和历史 ===
git status                         # 工作区状态
git diff                           # 未暂存的修改
git diff --staged                  # 已暂存的修改
git log --oneline --graph --all    # 最喜欢的日志视图
git log -p file.c                  # 查看某文件的修改历史
git blame file.c                   # 每行是谁改的
git show COMMIT                    # 查看某次提交的详情

# === 暂存与提交 ===
git add file                       # 暂存
git add -p                         # 交互式暂存（选择性地暂存代码块）
git commit -m "message"            # 提交
git commit --amend                 # 修改最后一次提交

# === 分支 ===
git branch                         # 列出本地分支
git branch -a                      # 列出所有分支（含远程）
git branch NAME                    # 创建分支
git switch NAME                    # 切换分支（新版，替代 checkout）
git switch -c NAME                 # 创建并切换
git merge NAME                     # 合并
git rebase main                    # 变基
git cherry-pick COMMIT             # 摘取提交

# === 远程 ===
git remote -v                      # 查看远程仓库
git fetch origin                   # 拉取不合并
git pull --rebase                  # 拉取 + 变基（推荐）
git push origin main               # 推送

# === 暂存与恢复 ===
git stash                          # 暂存当前修改
git stash pop                      # 恢复最近暂存
git stash list                     # 列出所有 stash
git checkout -- file               # 撤销文件修改
git reset HEAD file                # 取消暂存
git reset --hard HEAD              # ⚠️ 丢弃所有未提交修改
git revert COMMIT                  # 安全撤销（创建新 commit）

# === 查找 ===
git grep "TODO"                    # 在 git 管理的文件中搜索
git log --all --full-history -- "**/file.c"  # 查找文件的所有历史
git bisect start                   # 二分法定位 bug 引入点
```

---

### 12.6 ELF 分析与调试工具

```bash
# file — 快速识别文件类型
file a.out
file firmware.bin

# ldd — 查看动态库依赖
ldd a.out
ldd -v a.out                       # 详细（显示版本信息）
ldd -r a.out                       # 显示重定位信息

# readelf — 查看 ELF 结构
readelf -h a.out                   # ELF 头部
readelf -S a.out                   # Section 头部表
readelf -l a.out                   # Program 头部表（segment）
readelf -d a.out                   # Dynamic section
readelf -s a.out                   # 符号表
readelf -r a.out                   # 重定位表
readelf -A a.out                   # 架构特定属性（ARM 等）
readelf -a a.out                   # 全部信息

# objdump — 反汇编
objdump -d a.out                   # 反汇编所有代码段
objdump -d -S a.out                # 混排源码和汇编（需 -g 编译）
objdump -t a.out                   # 符号表
objdump -T a.out                   # 动态符号表
objdump -x a.out                   # 全部头部信息
objdump -s -j .rodata a.out        # 打印某 section 的原始内容

# nm — 列出符号
nm a.out                           # 所有符号
nm -D a.out                        # 动态符号
nm -u a.out                        # 未定义符号（需要外部提供）
nm -S a.out                        # 显示大小
nm --size-sort -S a.out            # 按大小排序

# size — 段大小统计
size a.out                         # text + data + bss
size -A a.out                      # 更详细的段大小

# strings — 提取可打印字符串
strings firmware.bin | grep -i "linux\|u-boot"
strings -n 8 firmware.bin          # 最小长度 8
strings -t x firmware.bin          # 显示十六进制偏移

# strip — 去除调试符号（减小体积）
strip a.out
strip --strip-debug a.out          # 仅去调试符号
file a.out                         # stripped 表示已去除

# hexdump / xxd — 十六进制查看
hexdump -C file.bin | head -50     # 正则十六进制 + ASCII
hexdump -C -n 256 file.bin         # 只看前 256 字节
xxd file.bin | head
xxd -r hexdump.txt > binary.bin    # hexdump 还原为二进制
```

---

### 12.7 系统调用与追踪

```bash
# strace — 跟踪系统调用
strace ./a.out                                    # 跟踪所有系统调用
strace -e open,read,write ./a.out                 # 只跟踪指定调用
strace -e trace=file ./a.out                       # 跟踪文件相关调用
strace -p PID                                      # attach 到运行中的进程
strace -c ./a.out                                  # 汇总统计
strace -o trace.log ./a.out                        # 输出到文件
strace -f ./a.out                                  # 跟踪子进程
strace -tt ./a.out                                 # 微秒时间戳
strace -s 1024 ./a.out                             # 打印更长字符串

# ltrace — 跟踪库函数调用
ltrace ./a.out
ltrace -e 'malloc+free' ./a.out

# perf — 性能分析（需要 linux-tools）
perf stat ./a.out                                  # 性能计数器
perf record ./a.out                                # 记录采样数据
perf report                                         # 查看报告
perf top                                            # 实时性能监控
```

---

## 13. I/O 重定向详解

### 文件描述符

| FD | 名称 | 默认指向 |
|----|------|----------|
| `0` | stdin | 键盘 |
| `1` | stdout | 终端 |
| `2` | stderr | 终端 |

### 基本重定向

```bash
cmd > file             # stdout 覆盖写入 file
cmd >> file            # stdout 追加写入 file
cmd 2> file            # stderr 写入 file
cmd &> file            # stdout + stderr 写入 file (bash)
cmd > file 2>&1        # stdout + stderr 写入 file (POSIX)
cmd > file 2>1         # ❌ 错误！这是把 stderr 写入名为 1 的文件
cmd < file             # 从 file 读取 stdin
cmd <<< "string"       # here-string (bash)
cmd << EOF             # here-document
...
EOF
cmd 2>&1 | tee log     # 同时看输出 + 存日志
cmd > /dev/null 2>&1   # 静默运行（丢弃所有输出）
```

### 常见模式

```bash
# 只保留 stderr
cmd 2> errors.log 1> /dev/null
# 等价于
cmd > /dev/null 2> errors.log      # 简写

# stdout 和 stderr 分开保存
cmd > stdout.log 2> stderr.log

# 同时写到终端和文件（管道过程中捕获）
make 2>&1 | tee build.log

# 只追加 stderr 到文件
cmd 2>> errors.log

# 交换 stdout 和 stderr
cmd 3>&1 1>&2 2>&3
```

---

## 14. 管道与组合

```bash
# 经典管道链
ps aux | grep python | grep -v grep | awk '{print $2}' | xargs kill

# 命令替换
gcc -o app $(find src -name "*.c")              # 所有 .c 文件编译
make -j$(nproc)                                   # CPU 核心数

# 逻辑运算符
cmd1 && cmd2          # cmd1 成功才执行 cmd2
cmd1 || cmd2          # cmd1 失败才执行 cmd2
cmd1 ; cmd2           # 依次执行（不管成败）
cmd1 | cmd2           # cmd1 的 stdout → cmd2 的 stdin

# 实用组合
make -j$(nproc) && ./run_tests || echo "Build or tests failed"
cd /path/to/project && git pull --rebase && make -j$(nproc)
[ -d build ] || mkdir build      # 目录不存在就创建
```

---

## 15. 快捷键速查

### Bash 行编辑 (Emacs 模式，默认)

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+A` | 跳到行首 |
| `Ctrl+E` | 跳到行尾 |
| `Ctrl+U` | 删除光标前到行首 |
| `Ctrl+K` | 删除光标后到行尾 |
| `Ctrl+W` | 删除光标前一个单词 |
| `Alt+D` | 删除光标后一个单词 |
| `Alt+Backspace` | 删除光标前一个单词 |
| `Ctrl+Y` | 粘贴最近删除的文本 |
| `Alt+.` | 粘贴上条命令最后一个参数 |
| `Ctrl+T` | 交换光标前后两个字符 |
| `Alt+T` | 交换光标前后两个单词 |
| `Ctrl+/` | 撤销 |
| `Ctrl+_` | 撤销（同 Ctrl+/） |

### 进程控制

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | SIGINT，终止前台进程 |
| `Ctrl+Z` | SIGSTOP，挂起前台进程 |
| `Ctrl+D` | EOF（退出 shell） |
| `Ctrl+L` | 清屏 |
| `Ctrl+S` | 暂停终端输出 |
| `Ctrl+Q` | 恢复终端输出 |

### 历史

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+R` | 反向搜索历史 |
| `Ctrl+G` | 退出历史搜索 |
| `Ctrl+P` / `↑` | 上一条命令 |
| `Ctrl+N` / `↓` | 下一条命令 |

---

## 附录 A：嵌入式开发专用软件包

```bash
# 基础工具
sudo apt install build-essential gcc g++ gdb gdb-multiarch
sudo apt install cmake ninja-build pkg-config

# ARM 交叉编译
sudo apt install gcc-arm-none-eabi      # bare-metal ARM
sudo apt install gcc-arm-linux-gnueabihf # ARM Linux (hard float)
sudo apt install gcc-aarch64-linux-gnu   # ARM64 Linux

# 内核 / U-Boot 编译
sudo apt install flex bison bc libncurses-dev libssl-dev
sudo apt install device-tree-compiler u-boot-tools

# QEMU 虚拟化
sudo apt install qemu-system-arm qemu-system-aarch64 qemu-system-x86
sudo apt install qemu-user-static           # 用户态模拟（chroot 用）

# 文件系统
sudo apt install genext2fs genimage mtd-utils squashfs-tools
sudo apt install f2fs-tools btrfs-progs

# 调试 / 分析
sudo apt install minicom picocom            # 串口终端
sudo apt install tio                        # 现代串口终端（推荐）
sudo apt install openocd                    # 片上调试器
sudo apt install binwalk                    # 固件分析
```

---

## 附录 B：实用 man page 速查方式

```bash
man command              # 查手册
man -k keyword           # 搜索手册 (apropos)
man -f command           # 显示简短描述 (whatis)
man N command            # 查第 N 章节 (1=命令, 2=系统调用, 3=库函数, 5=配置文件, 7=杂项, 8=系统管理)
info command             # GNU Info 文档（通常比 man 详细）

# 快速查看选项
command --help           # 大多数 GNU 工具支持
command -h               # 部分工具

# tldr — 社区驱动的简化手册（推荐安装）
sudo apt install tldr
tldr tar
tldr find
```
