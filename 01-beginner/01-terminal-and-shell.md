# 第 01 课：什么是 Terminal 与 Shell

> 上一课：[README](../README.md) | 下一课：[第 02 课：文件系统导航](./02-navigating-filesystem.md)

## 学习目标

- 理解 Terminal（终端）和 Shell（命令解释器）的概念与区别
- 了解常见的 Shell 类型：bash、zsh、sh、fish
- 学会在不同操作系统中打开终端
- 掌握第一批基础命令：`echo`、`whoami`、`date`、`clear`
- 理解 CLI（命令行界面）与 GUI（图形界面）的区别

---

## 知识讲解

### 1. Terminal 与 Shell 的区别

**Terminal（终端）** 是一个窗口程序，它提供了一个文本输入/输出的界面。你可以把它想象成一个"容器"或"窗口"。

**Shell（命令解释器）** 是运行在 Terminal 内部的程序，负责接收你输入的命令、解释并执行它们。Shell 是你与操作系统内核之间的"翻译官"。

简单来说：

| 概念 | 角色 | 类比 |
|------|------|------|
| Terminal | 提供界面的窗口 | 电视机屏幕 |
| Shell | 解释和执行命令的程序 | 电视台（提供内容） |

### 2. 常见的 Shell 类型

| Shell | 说明 |
|-------|------|
| `bash` | **Bourne Again Shell**，最广泛使用的 Shell，大多数 Linux 发行版的默认 Shell |
| `zsh` | **Z Shell**，功能更丰富，macOS Catalina 及之后的默认 Shell |
| `sh` | **Bourne Shell**，最古老的 Shell，语法是其他 Shell 的基础 |
| `fish` | **Friendly Interactive Shell**，语法更友好，自动补全强大，但与 POSIX 不完全兼容 |

查看当前使用的 Shell：

```bash
echo $SHELL
```

查看系统中可用的所有 Shell：

```bash
cat /etc/shells
```

### 3. 如何打开终端

| 操作系统 | 方法 |
|----------|------|
| **Ubuntu/Debian** | `Ctrl + Alt + T`，或在应用菜单搜索 "Terminal" |
| **CentOS/Fedora** | 应用菜单 → 系统工具 → Terminal |
| **macOS** | `Cmd + Space` 搜索 "Terminal"，或打开 Launchpad → 其他 → 终端 |
| **Windows (WSL)** | 安装 WSL 后，搜索 "Ubuntu" 或 "Windows Terminal" |

### 4. Shell Prompt（命令提示符）解析

打开终端后，你会看到类似这样的提示符：

```
tourist@myhost:~$
```

各部分含义：

```
tourist  @  myhost  :  ~        $
──┬───   ─  ──┬───  ─  ┬        ┬
  │            │        │        │
 用户名      主机名   当前目录   普通用户标识
```

- `~` 代表当前用户的家目录（home directory）
- `$` 表示当前是普通用户（root 用户显示 `#`）

### 5. CLI 与 GUI 的区别

| 特性 | CLI（命令行界面） | GUI（图形界面） |
|------|-------------------|-----------------|
| 交互方式 | 键盘输入文本命令 | 鼠标点击图标和菜单 |
| 学习曲线 | 较陡，需要记命令 | 较平缓，直观易上手 |
| 效率 | 熟练后极高，支持批量操作和脚本自动化 | 适合简单、偶尔的操作 |
| 资源占用 | 极低 | 较高 |
| 远程操作 | 通过 SSH 即可，带宽需求极低 | 需要图形化远程桌面，带宽需求高 |
| 自动化 | 天然支持脚本化 | 难以自动化 |

### 6. 第一批命令

#### `echo` — 输出文本

```bash
echo "Hello, Linux!"
echo "当前 Shell 是: $SHELL"
```

#### `whoami` — 显示当前用户名

```bash
whoami
```

#### `date` — 显示当前日期和时间

```bash
date
date "+%Y-%m-%d %H:%M:%S"    # custom format
```

#### `clear` — 清屏

```bash
clear
```

也可以使用快捷键 `Ctrl + L` 达到同样效果。

---

## 实战演练

打开终端，按顺序执行以下步骤：

**步骤 1：确认你的 Shell**

```bash
echo $SHELL
```

你应该看到类似 `/bin/bash` 或 `/bin/zsh` 的输出。

**步骤 2：查看你是谁**

```bash
whoami
```

输出你的用户名，例如 `tourist`。

**步骤 3：查看当前日期**

```bash
date
```

输出类似：`Mon Apr 14 10:30:00 CST 2026`

**步骤 4：用自定义格式显示日期**

```bash
date "+%Y年%m月%d日 %H:%M"
```

输出类似：`2026年04月14日 10:30`

**步骤 5：用 echo 输出一段话**

```bash
echo "我正在学习 Linux 命令行！"
echo "我的用户名是: $(whoami)"
```

第二条命令使用了 **命令替换**（command substitution），`$(whoami)` 会先执行 `whoami`，然后把结果嵌入到 `echo` 的输出中。

**步骤 6：清屏**

```bash
clear
```

屏幕被清空，但之前的命令历史并没有消失，你可以向上滚动查看。

**步骤 7：查看系统可用的 Shell**

```bash
cat /etc/shells
```

你会看到系统中安装的所有 Shell 路径列表。

---

## 小结

| 命令 | 作用 | 示例 |
|------|------|------|
| `echo` | 输出文本到终端 | `echo "Hello"` |
| `whoami` | 显示当前用户名 | `whoami` |
| `date` | 显示日期和时间 | `date "+%Y-%m-%d"` |
| `clear` | 清屏 | `clear` 或 `Ctrl + L` |
| `echo $SHELL` | 查看当前 Shell | `echo $SHELL` |
| `cat /etc/shells` | 查看可用 Shell 列表 | `cat /etc/shells` |

---

## 练习

**1. Terminal 和 Shell 分别是什么？它们的关系是什么？**

<details>
<summary>查看答案</summary>

Terminal（终端）是提供文本输入输出界面的窗口程序；Shell（命令解释器）是运行在 Terminal 内部、负责解释和执行命令的程序。Terminal 是外壳（显示界面），Shell 是内核（处理逻辑）。一个 Terminal 可以运行不同的 Shell。

</details>

**2. 如何查看当前系统使用的是哪种 Shell？**

<details>
<summary>查看答案</summary>

```bash
echo $SHELL
```

该命令会输出当前默认 Shell 的路径，例如 `/bin/bash` 或 `/bin/zsh`。

</details>

**3. 请写出一条命令，输出格式为 `2026-04-14` 的当前日期。**

<details>
<summary>查看答案</summary>

```bash
date "+%Y-%m-%d"
```

其中 `%Y` 是四位年份，`%m` 是两位月份，`%d` 是两位日期。

</details>

**4. Shell Prompt 中 `$` 和 `#` 分别代表什么？**

<details>
<summary>查看答案</summary>

- `$` 表示当前是**普通用户**
- `#` 表示当前是 **root 用户**（超级管理员）

看到 `#` 时要格外小心，因为 root 拥有系统最高权限，误操作可能导致严重后果。

</details>

**5. CLI 相比 GUI 有哪些优势？请至少列出三点。**

<details>
<summary>查看答案</summary>

1. **效率高**：熟练后操作速度远超鼠标点击
2. **支持自动化**：可以编写脚本批量处理任务
3. **资源占用低**：不需要图形界面，适合服务器环境
4. **远程操作方便**：通过 SSH 即可远程管理，带宽需求极低
5. **可重复性强**：命令可以记录、分享、重复执行

</details>
