# 第 02 课：文件系统导航

> 上一课：[第 01 课：什么是 Terminal 与 Shell](./01-terminal-and-shell.md) | 下一课：[第 03 课：文件操作](./03-file-operations.md)

## 学习目标

- 理解 Linux 文件系统的层级结构（Filesystem Hierarchy）
- 掌握 `pwd`、`cd`、`ls`、`tree` 等导航命令
- 理解绝对路径（absolute path）与相对路径（relative path）的区别
- 熟练使用路径中的特殊符号：`~`、`-`、`..`、`.`

---

## 知识讲解

### 1. Linux 文件系统层级结构

Linux 的文件系统从根目录 `/` 开始，呈**树形结构**向下展开。所有文件和目录都挂载在这棵树上，包括硬盘、U盘甚至网络设备。

```
/                   # root directory
├── home/           # user home directories
│   ├── tourist/
│   └── alice/
├── etc/            # system configuration files
├── var/            # variable data (logs, cache, etc.)
│   └── log/
├── tmp/            # temporary files (cleared on reboot)
├── usr/            # user programs and shared resources
│   ├── bin/
│   └── lib/
├── bin/            # essential system binaries
├── sbin/           # system admin binaries
├── opt/            # optional/third-party software
├── dev/            # device files
├── proc/           # process and kernel info (virtual)
└── mnt/            # mount points for external filesystems
```

关键目录说明：

| 目录 | 说明 |
|------|------|
| `/` | 根目录，整个文件系统的起点 |
| `/home` | 普通用户的家目录，每个用户一个子目录 |
| `/etc` | 系统配置文件（如网络配置、用户信息等） |
| `/var` | 可变数据文件（日志、缓存、邮件等） |
| `/tmp` | 临时文件，重启后通常会被清空 |
| `/usr` | 用户级程序和共享资源（类似 Windows 的 Program Files） |
| `/opt` | 第三方软件的安装目录 |

### 2. `pwd` — Print Working Directory

显示当前所在的目录（绝对路径）：

```bash
pwd
```

输出示例：

```
/home/tourist
```

### 3. `cd` — Change Directory

切换当前工作目录：

```bash
cd /etc              # go to absolute path
cd Documents         # go to relative path (subdirectory)
cd ..                # go up one level
cd ../..             # go up two levels
cd ~                 # go to home directory
cd                   # same as cd ~ (go home)
cd -                 # go back to previous directory
```

特殊路径符号：

| 符号 | 含义 |
|------|------|
| `/` | 根目录 |
| `~` | 当前用户的家目录（等价于 `$HOME`） |
| `.` | 当前目录 |
| `..` | 上一级（父）目录 |
| `-` | 上一次所在的目录 |

### 4. `ls` — List Directory Contents

```bash
ls                   # list current directory
ls /etc              # list specified directory
ls -l                # long format (permissions, size, date)
ls -a                # show all files including hidden (dotfiles)
ls -lh               # long format with human-readable sizes
ls -R                # recursive listing (include subdirectories)
ls -lt               # sort by modification time (newest first)
ls -lS               # sort by file size (largest first)
```

`ls -l` 输出详解：

```
-rw-r--r--  1  tourist  staff  4096  Apr 14 10:00  notes.txt
──────┬───  ┬  ──┬────  ──┬──  ──┬─  ─────┬──────  ────┬────
  权限     链接  所有者   组    大小    修改时间      文件名
```

**隐藏文件**：Linux 中以 `.` 开头的文件或目录是隐藏的（如 `.bashrc`、`.ssh/`），必须用 `ls -a` 才能看到。

### 5. `tree` — 树形显示目录结构

```bash
tree                 # show full tree of current directory
tree -L 2            # limit depth to 2 levels
tree -d              # show only directories
tree -a              # include hidden files
tree /etc -L 1       # show /etc top level
```

> 如果系统没有安装 `tree`，可以用包管理器安装：
> - Ubuntu/Debian: `sudo apt install tree`
> - CentOS/Fedora: `sudo yum install tree`
> - macOS: `brew install tree`

### 6. 绝对路径 vs 相对路径

**绝对路径（Absolute Path）** 从根目录 `/` 开始，描述文件的完整位置：

```bash
cd /home/tourist/Documents
cat /etc/hostname
```

**相对路径（Relative Path）** 从当前目录出发，描述相对位置：

```bash
cd Documents              # relative to current directory
cat ../config.txt         # one level up, then config.txt
```

对比示例（假设当前在 `/home/tourist`）：

| 目标 | 绝对路径 | 相对路径 |
|------|----------|----------|
| 家目录下的 Documents | `/home/tourist/Documents` | `./Documents` 或 `Documents` |
| 系统配置目录 | `/etc` | `../../etc` |
| 上级目录 | `/home` | `..` |

**建议**：在脚本中使用绝对路径以避免歧义；在日常交互中使用相对路径以提高效率。

---

## 实战演练

跟随以下步骤完成一次完整的文件系统探索：

**步骤 1：确认当前位置**

```bash
pwd
```

**步骤 2：前往根目录并查看顶层结构**

```bash
cd /
ls -l
```

你会看到整个 Linux 文件系统的顶层目录。

**步骤 3：探索 `/etc` 目录**

```bash
cd /etc
pwd
ls | head -20
```

`head -20` 只显示前 20 个结果，避免输出过多。

**步骤 4：回到家目录**

```bash
cd ~
pwd
```

**步骤 5：查看家目录中的所有文件（包括隐藏文件）**

```bash
ls -la
```

注意以 `.` 开头的隐藏文件，如 `.bashrc`、`.profile` 等。

**步骤 6：创建练习目录并探索相对路径**

```bash
mkdir -p ~/linux-practice/level1/level2
cd ~/linux-practice
pwd

cd level1
pwd

cd level2
pwd

cd ../..
pwd
```

观察每次 `pwd` 的输出变化，理解 `..` 的作用。

**步骤 7：使用 `cd -` 快速切换**

```bash
cd /var/log
pwd
cd /etc
pwd
cd -
pwd
```

`cd -` 会把你带回 `/var/log`，在两个目录之间快速切换非常方便。

**步骤 8：用 `tree` 查看刚才创建的目录结构**

```bash
tree ~/linux-practice
```

输出类似：

```
/home/tourist/linux-practice
└── level1
    └── level2
```

**步骤 9：清理练习目录**

```bash
rm -r ~/linux-practice
```

---

## 小结

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `pwd` | 显示当前目录 | — |
| `cd <path>` | 切换目录 | `~`（家目录）、`-`（上一个目录）、`..`（上级目录） |
| `ls` | 列出目录内容 | `-l`（详细）、`-a`（含隐藏）、`-lh`（可读大小）、`-R`（递归） |
| `tree` | 树形显示目录 | `-L n`（限制深度）、`-d`（仅目录） |

路径速查：

| 符号 | 含义 | 示例 |
|------|------|------|
| `/` | 根目录 | `cd /` |
| `~` | 家目录 | `cd ~` |
| `.` | 当前目录 | `./script.sh` |
| `..` | 上一级目录 | `cd ..` |
| `-` | 前一个目录 | `cd -` |

---

## 练习

**1. 你当前在 `/home/tourist/projects` 目录，如何用相对路径进入 `/home/tourist/Documents`？**

<details>
<summary>查看答案</summary>

```bash
cd ../Documents
```

`..` 先回到上一级 `/home/tourist`，然后进入 `Documents`。

</details>

**2. `ls -la` 中的 `-a` 和 `-l` 分别代表什么？**

<details>
<summary>查看答案</summary>

- `-a`（all）：显示所有文件，包括以 `.` 开头的隐藏文件
- `-l`（long）：使用长格式输出，显示权限、所有者、大小、修改时间等详细信息

两个选项可以合并写成 `-la` 或 `-al`，效果相同。

</details>

**3. `cd` 和 `cd ~` 有什么区别？**

<details>
<summary>查看答案</summary>

没有区别。两者都是返回当前用户的家目录（home directory）。不带参数的 `cd` 默认行为就是 `cd ~`（即 `cd $HOME`）。

</details>

**4. 请解释以下路径中每个部分的含义：`/home/tourist/../alice/./Documents`**

<details>
<summary>查看答案</summary>

逐段解析：
- `/home/tourist` — 进入 tourist 的家目录
- `/..` — 返回上一级，回到 `/home`
- `/alice` — 进入 alice 的家目录
- `/.` — 当前目录（不变，仍在 alice）
- `/Documents` — 进入 Documents 子目录

最终等价于：`/home/alice/Documents`

</details>

**5. 如何用 `tree` 命令只显示 `/usr` 目录下两层的目录结构（不包含文件）？**

<details>
<summary>查看答案</summary>

```bash
tree -dL 2 /usr
```

- `-d` 只显示目录，不显示文件
- `-L 2` 限制深度为 2 层

</details>
