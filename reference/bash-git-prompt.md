# Ubuntu 22.04 Bash 终端显示 Git 分支

## 问题

Ubuntu 22.04 默认的 bash 提示符（PS1）不显示当前 git 仓库的分支信息。进入 git 仓库后，终端提示符没有任何变化。

## 原因

1. Git 自带了 `__git_ps1` 函数（位于 `/usr/lib/git-core/git-sh-prompt`），但默认 `~/.bashrc` 没有 source 这个文件
2. 默认的 PS1 没有调用 `$(__git_ps1)` 来显示分支名

## 解决方法

编辑 `~/.bashrc`，做两个修改：

### 1. 加载 git-prompt 脚本

在 `~/.bashrc` 末尾或合适位置添加：

```bash
# --- Git prompt (show branch in PS1) ---
if [ -f /usr/lib/git-core/git-sh-prompt ]; then
    . /usr/lib/git-core/git-sh-prompt
    export GIT_PS1_SHOWDIRTYSTATE=1
    export GIT_PS1_SHOWSTASHSTATE=1
    export GIT_PS1_SHOWUNTRACKEDFILES=1
    export GIT_PS1_SHOWUPSTREAM="auto"
fi
```

各选项含义：

| 选项 | 作用 |
|------|------|
| `GIT_PS1_SHOWDIRTYSTATE=1` | 有未暂存的修改时显示 `*`，有已暂存的修改时显示 `+` |
| `GIT_PS1_SHOWSTASHSTATE=1` | 有 stash 时显示 `$` |
| `GIT_PS1_SHOWUNTRACKEDFILES=1` | 有未跟踪文件时显示 `%` |
| `GIT_PS1_SHOWUPSTREAM="auto"` | 显示与远程分支的同步状态 |

### 2. 修改 PS1 加入 git 分支

找到彩色的 PS1 行，在 `\$` 前加入 `$(__git_ps1 " (%s)")`：

```bash
# 修改前
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# 修改后
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\[\033[01;33m\]$(__git_ps1 " (%s)")\[\033[00m\]\$ '
```

其中 `\[\033[01;33m\]` 将分支名设为黄色，`\[\033[00m\]` 恢复默认颜色。

### 3. 重新加载

```bash
source ~/.bashrc
```

## 效果

进入 git 仓库后，提示符变为：

```
user@host:~/path/to/repo (main *%)$
```

## 上游同步状态符号

| 符号 | 含义 |
|------|------|
| `=` | 本地与远程同步 |
| `<` | 本地落后于远程 |
| `>` | 本地领先于远程 |
| `<>` | 本地与远程分叉（diverged） |

## 仓库状态符号

| 符号 | 含义 |
|------|------|
| `*` | 有未暂存的修改（unstaged changes） |
| `+` | 有已暂存的修改（staged changes） |
| `%` | 有未跟踪的文件（untracked files） |
| `$` | 有 stash 保存的内容 |
