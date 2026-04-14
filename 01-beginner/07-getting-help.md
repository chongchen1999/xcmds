# 第 07 课：获取帮助

> 上一课：[第 06 课：I/O Redirection 与 Pipe](./06-redirection-and-pipes.md) | 下一课：[第 08 课：入门综合测验](./08-beginner-quiz.md)

## 学习目标

- 掌握 `man` 手册的使用方法，理解 man page 的章节分类
- 掌握 `--help` 快速查看命令用法
- 了解 `info` 文档系统
- 掌握 `whatis` 和 `apropos` 快速查找命令
- 了解 `tldr` 社区驱动的简化手册
- 了解常用在线资源

---

## 知识讲解

### 1. `man` — Manual Pages

`man` 是 Linux 中最权威、最完整的命令参考文档系统。

```bash
man ls          # open the manual for ls
man cp          # open the manual for cp
man man         # the manual about man itself
```

#### man page 导航

在 man page 中（使用 `less` 作为分页器），操作方式与 `less` 相同：

| 按键 | 操作 |
|------|------|
| `Space` / `f` | 向下翻页 |
| `b` | 向上翻页 |
| `j` / `↓` | 向下一行 |
| `k` / `↑` | 向上一行 |
| `/pattern` | 搜索关键词 |
| `n` / `N` | 下一个/上一个搜索结果 |
| `q` | 退出 |

#### man page 的内容结构

一个典型的 man page 包含以下部分：

| 段落 | 内容 |
|------|------|
| **NAME** | 命令名称和简短描述 |
| **SYNOPSIS** | 命令语法格式 |
| **DESCRIPTION** | 详细描述 |
| **OPTIONS** | 所有可用选项说明 |
| **EXAMPLES** | 使用示例（不是所有 man page 都有） |
| **SEE ALSO** | 相关命令和手册的引用 |
| **EXIT STATUS** | 返回值说明 |

#### SYNOPSIS 语法约定

```
ls [OPTION]... [FILE]...
```

- `[...]` — 方括号内为可选参数
- `...` — 可以指定多个
- **粗体** / 下划线 — 需要按原样输入的关键字
- 无括号 — 必需参数

#### man page 章节（Sections）

Linux 的 man page 分为 8 个章节：

| 章节 | 内容 | 示例 |
|------|------|------|
| 1 | 用户命令 | `ls`, `grep`, `chmod` |
| 2 | 系统调用（System Calls） | `open()`, `read()`, `fork()` |
| 3 | 库函数（Library Functions） | `printf()`, `malloc()` |
| 4 | 特殊文件和设备 | `/dev/null`, `/dev/sda` |
| 5 | 文件格式和配置文件 | `/etc/passwd`, `/etc/fstab` |
| 6 | 游戏 | — |
| 7 | 杂项（协议、规范等） | `ascii`, `regex`, `signal` |
| 8 | 系统管理命令 | `mount`, `iptables`, `useradd` |

有些名称同时存在于多个章节中，需要指定章节号：

```bash
man passwd          # default: section 1 (the command)
man 5 passwd        # section 5: /etc/passwd file format
man 3 printf        # section 3: C library function
man 2 open          # section 2: system call
```

查看所有章节中的同名条目：

```bash
man -a passwd       # show all sections sequentially (press q to go to next)
man -k passwd       # search all man pages related to "passwd"
```

### 2. `--help` — 快速参考

几乎所有命令都支持 `--help`（或 `-h`）选项，输出比 man page 更简洁的用法说明：

```bash
ls --help
grep --help
chmod --help
```

`--help` 的优势：
- 速度快，直接输出到终端
- 信息简洁，快速查看选项列表
- 适合已经知道命令但忘了某个选项的场景

```bash
# quickly check tar options
tar --help | grep "extract"

# check rsync compression option
rsync --help | grep "compress"
```

> 当你只需要查一个选项时用 `--help`；需要完整理解一个命令时用 `man`。

### 3. `info` — GNU Info 文档

`info` 是 GNU 项目的文档系统，某些 GNU 工具的 `info` 文档比 man page 更详细（特别是 `coreutils`、`bash`、`gawk` 等）。

```bash
info coreutils       # GNU core utilities documentation
info bash            # detailed bash manual
info grep            # grep documentation
```

`info` 阅读器导航：

| 按键 | 操作 |
|------|------|
| `Space` | 向下翻页 |
| `Backspace` | 向上翻页 |
| `Tab` | 跳到下一个超链接 |
| `Enter` | 进入超链接 |
| `u` | 回到上级节点 |
| `n` / `p` | 下一节 / 上一节 |
| `q` | 退出 |

> 实际使用中，大多数人更习惯使用 `man`。`info` 更像是扩展阅读。

### 4. `whatis` 和 `apropos` — 快速搜索

#### `whatis` — 快速查看命令简介

```bash
whatis ls
# ls (1)               - list directory contents

whatis chmod
# chmod (1)            - change file mode bits

whatis passwd
# passwd (1)           - change user password
# passwd (5)           - the password file
```

`whatis` 显示命令的 one-line description 和所在的 man page 章节。

#### `apropos` — 按关键词搜索命令

当你知道想做什么，但不知道用哪个命令时：

```bash
apropos "copy file"
# cp (1)               - copy files and directories
# install (1)          - copy files and set attributes

apropos "disk usage"
# df (1)               - report file system disk space usage
# du (1)               - estimate file space usage

apropos "compress"
# gzip (1)             - compress or expand files
# bzip2 (1)            - a block-sorting file compressor
# zip (1)              - package and compress files
```

```bash
# equivalent to apropos
man -k "keyword"
```

> 如果 `apropos` 报错 "nothing appropriate"，可能需要运行 `sudo mandb` 重建 man page 索引数据库。

### 5. `tldr` — 社区简化手册

`tldr`（Too Long; Didn't Read）是社区维护的简化命令参考，提供最常用的示例，比 man page 更容易上手。

#### 安装

```bash
# Node.js
npm install -g tldr

# Python
pip install tldr

# Homebrew (macOS)
brew install tldr

# Ubuntu/Debian
sudo apt install tldr
```

#### 使用

```bash
tldr tar
```

输出示例：

```
tar
Archiving utility.

- Create an archive from files:
  tar cf target.tar file1 file2 file3

- Create a gzipped archive:
  tar czf target.tar.gz file1 file2 file3

- Extract an archive in the current directory:
  tar xf source.tar

- Extract a gzipped archive:
  tar xzf source.tar.gz

- Extract to a specific directory:
  tar xf source.tar -C /path/to/directory
```

`tldr` 的优势：
- 只展示最常用的用法和示例
- 直接给出可运行的命令，无需阅读大量文档
- 社区持续更新维护

> `tldr` 适合快速查阅；深入理解还是要看 `man`。两者配合使用效果最好。

### 6. 在线资源

除了本地文档工具，以下在线资源也非常有价值：

| 资源 | 网址 | 特点 |
|------|------|------|
| **explainshell.com** | https://explainshell.com | 粘贴完整命令，逐词解析含义 |
| **tldr.sh** | https://tldr.sh | tldr 的在线版本 |
| **man7.org** | https://man7.org/linux/man-pages/ | Linux man pages 在线版 |
| **ArchWiki** | https://wiki.archlinux.org | 极其详细的 Linux 百科 |
| **Stack Overflow** | https://stackoverflow.com | 搜索具体问题 |

#### `type` 和 `which` — 了解命令本身

在查手册之前，先确认命令的类型：

```bash
type cd
# cd is a shell builtin

type ls
# ls is aliased to 'ls --color=auto'

type grep
# grep is /usr/bin/grep

which python3
# /usr/bin/python3
```

- **builtin** 命令的帮助在 `man bash` 或 `help cd` 中
- **外部命令** 的帮助用 `man` 查看
- **alias** 需要先了解原始命令

```bash
help cd              # help for shell builtins
help for
help if
```

---

## 实战演练

**步骤 1：使用 `man` 查看命令手册**

```bash
man ls
```

进入后练习：
1. 按 `Space` 翻页
2. 按 `/` 输入 `-h` 搜索 human-readable 选项
3. 按 `n` 跳到下一个匹配
4. 按 `q` 退出

**步骤 2：查看不同章节的同名 man page**

```bash
whatis passwd
```

注意输出有多个章节，分别查看：

```bash
man 1 passwd         # the passwd command
man 5 passwd         # the /etc/passwd file format
```

**步骤 3：使用 `--help` 快速查询**

```bash
grep --help | head -20     # quick overview of grep options
tar --help | grep "create" # find the create option
```

**步骤 4：使用 `apropos` 搜索命令**

```bash
apropos "list directory"
apropos "change password"
apropos "disk space"
```

**步骤 5：判断命令类型**

```bash
type echo       # builtin
type ls         # may be aliased
type grep       # external command
type cd         # builtin
which python3   # show path
```

**步骤 6：查看 Shell Builtin 的帮助**

```bash
help cd
help echo
help type
```

**步骤 7：（可选）安装并使用 `tldr`**

```bash
# install if not available
# npm install -g tldr   or   pip install tldr

tldr find
tldr chmod
tldr rsync
```

对比 `man find` 和 `tldr find` 的信息量差异。

---

## 小结

| 工具 | 作用 | 适合场景 |
|------|------|----------|
| `man` | 完整的官方手册 | 深入理解命令的所有选项和行为 |
| `--help` | 命令内置的简短帮助 | 快速查看选项列表 |
| `info` | GNU 扩展文档 | GNU 工具的详细教程 |
| `whatis` | 一行命令简介 | 快速了解某个命令是干什么的 |
| `apropos` | 按关键词搜索命令 | 知道目标但不知道命令名 |
| `tldr` | 社区简化手册 | 快速查看常用示例 |
| `type` / `which` | 判断命令类型和路径 | 确认命令是 builtin、alias 还是外部程序 |
| `help` | Shell Builtin 帮助 | 查看 `cd`、`echo` 等内建命令 |

---

## 练习

**1. 如何查看 `/etc/passwd` 文件格式的说明（而不是 `passwd` 命令的用法）？**

<details>
<summary>查看答案</summary>

```bash
man 5 passwd
```

`passwd` 存在于 man page 的第 1 章节（用户命令）和第 5 章节（文件格式）。默认 `man passwd` 显示第 1 章节，要查看文件格式需要指定章节号 `5`。

可以先用 `whatis passwd` 查看所有可用章节：

```bash
whatis passwd
# passwd (1)           - change user password
# passwd (5)           - the password file
```

</details>

**2. 你想压缩文件但不记得用什么命令，如何搜索？**

<details>
<summary>查看答案</summary>

```bash
apropos compress
```

或者：

```bash
man -k compress
```

输出会列出所有与 "compress" 相关的命令，如 `gzip`、`bzip2`、`xz`、`zip` 等。

也可以缩小范围：

```bash
apropos "compress file"
```

</details>

**3. `type cd` 显示 "cd is a shell builtin"，如何查看 `cd` 的帮助信息？**

<details>
<summary>查看答案</summary>

Shell Builtin 命令不是独立的程序，没有单独的 man page（有些系统有 `man cd`，但内容较少）。正确的查看方式：

```bash
help cd              # bash built-in help system
```

或者查看 bash 手册中 builtin 部分：

```bash
man bash             # then search /cd with /    cd\b
```

`help` 命令只能用于 Shell Builtin，对外部命令无效。

</details>

**4. `man` 和 `tldr` 各适合什么场景？**

<details>
<summary>查看答案</summary>

- **`man`** 适合：
  - 需要完整了解一个命令的所有选项和行为
  - 查看系统调用或文件格式的详细规范
  - 作为权威参考（永远是最准确的文档来源）

- **`tldr`** 适合：
  - 快速查看常用示例（"我怎么用 `tar` 解压？"）
  - 第一次接触一个命令，想快速上手
  - 记不住复杂命令的常用参数组合

最佳实践：先用 `tldr` 快速了解常用用法，需要深入理解时再查 `man`。

</details>

**5. 如何找到系统中所有与 "network" 相关的命令？**

<details>
<summary>查看答案</summary>

```bash
apropos network
```

如果结果太多，可以配合 `grep` 过滤章节 1（用户命令）和章节 8（管理命令）：

```bash
apropos network | grep "(1)\|(8)"
```

还可以搜索更具体的关键词：

```bash
apropos "network interface"
apropos "network configuration"
```

如果 `apropos` 返回 "nothing appropriate"，需要更新 man page 数据库：

```bash
sudo mandb
```

</details>
