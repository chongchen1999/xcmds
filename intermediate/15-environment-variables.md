# 第 15 课：Environment Variables 与 Shell 配置

> 上一课：[第 14 课：包管理器](./14-package-management.md) | 下一课：[第 16 课：进阶综合测验](./16-intermediate-quiz.md)

---

## 学习目标

1. 理解环境变量的概念及其在 Linux 系统中的作用
2. 掌握常见环境变量：`PATH`、`HOME`、`USER`、`SHELL`、`LANG`、`EDITOR`
3. 学会使用 `export` 设置和管理环境变量
4. 理解 Shell 配置文件的加载顺序（`.bashrc`、`.bash_profile`、`.zshrc`、`.profile`）
5. 学会自定义 `PS1` 提示符、设置 `alias` 别名以及修改 `PATH`

---

## 知识讲解

### 1. 什么是环境变量

环境变量是操作系统用来存储配置信息的**键值对**。它们影响 Shell 和程序的行为方式。

```bash
# view all environment variables
env

# view a specific variable
echo $HOME

# show all variables (including shell-local variables)
set | head -30
```

环境变量与 Shell 变量的区别：

| 类型 | 作用域 | 设置方式 |
|------|--------|---------|
| Shell 变量 | 仅当前 Shell 进程可见 | `VAR=value` |
| 环境变量 | 当前进程及所有子进程可见 | `export VAR=value` |

```bash
# shell variable (only visible in current shell)
MY_VAR="hello"
echo $MY_VAR       # works
bash -c 'echo $MY_VAR'  # empty — child process can't see it

# environment variable (inherited by child processes)
export MY_VAR="hello"
bash -c 'echo $MY_VAR'  # works — child inherits it
```

### 2. 常见环境变量

| 变量 | 含义 | 示例值 |
|------|------|--------|
| `PATH` | 可执行文件搜索路径 | `/usr/local/bin:/usr/bin:/bin` |
| `HOME` | 当前用户主目录 | `/home/username` |
| `USER` | 当前用户名 | `username` |
| `SHELL` | 当前使用的 Shell | `/bin/bash` |
| `LANG` | 系统语言/编码 | `en_US.UTF-8` |
| `EDITOR` | 默认文本编辑器 | `vim` |
| `TERM` | 终端类型 | `xterm-256color` |
| `PWD` | 当前工作目录 | `/home/username/projects` |
| `OLDPWD` | 上一个工作目录 | `/home/username` |
| `HOSTNAME` | 主机名 | `myserver` |
| `UID` | 当前用户 ID | `1000` |
| `LOGNAME` | 登录用户名 | `username` |

```bash
# view common variables
echo "User: $USER"
echo "Home: $HOME"
echo "Shell: $SHELL"
echo "PATH: $PATH"
echo "Editor: $EDITOR"
echo "Language: $LANG"
```

#### PATH 详解

`PATH` 是最重要的环境变量之一。当你输入一个命令时，Shell 会按照 `PATH` 中列出的目录**从左到右**依次查找该命令的可执行文件。

```bash
# show PATH, one directory per line
echo $PATH | tr ':' '\n'

# find which executable is used
which python3
type python3

# show all matches
which -a python3
```

### 3. export 命令

```bash
# set and export in one step
export EDITOR="vim"

# set first, export later
MY_APP_ENV="production"
export MY_APP_ENV

# unset (remove) a variable
unset MY_APP_ENV

# temporarily set variable for a single command
LANG=C sort file.txt
DEBUG=1 ./my_script.sh

# export multiple variables
export VAR1="value1" VAR2="value2"
```

`export -p` 可以查看当前所有已导出的环境变量：

```bash
export -p | head -20
```

### 4. Shell 配置文件与加载顺序

不同的 Shell 在不同场景下读取不同的配置文件。理解加载顺序非常重要。

#### Bash 配置文件

```
Login Shell 加载顺序：
/etc/profile
  └── ~/.bash_profile (如果存在)
        └── 通常在内部 source ~/.bashrc
      或 ~/.bash_login (如果 .bash_profile 不存在)
      或 ~/.profile (如果前两者都不存在)

Non-login Interactive Shell 加载顺序：
~/.bashrc
```

| 文件 | 何时加载 | 典型用途 |
|------|---------|---------|
| `/etc/profile` | Login shell | 系统全局配置 |
| `~/.bash_profile` | Login shell | 用户登录配置（SSH 登录、tty 登录） |
| `~/.bashrc` | Non-login interactive shell | 每次打开新终端窗口时执行 |
| `~/.profile` | Login shell（当 `.bash_profile` 不存在时） | 通用 shell 配置 |
| `~/.bash_logout` | 退出 login shell 时 | 清理操作 |

> **Login Shell vs Non-login Shell**：
> - **Login shell**：通过 SSH 登录、在 tty 登录、`su - user`
> - **Non-login shell**：在桌面环境下打开终端模拟器、运行 `bash` 子 shell

**最佳实践**：将所有配置放在 `~/.bashrc` 中，然后在 `~/.bash_profile` 中 source 它：

```bash
# ~/.bash_profile
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

#### Zsh 配置文件

```
加载顺序：
/etc/zshenv       → 所有 Zsh 实例
~/.zshenv         → 所有 Zsh 实例
/etc/zprofile     → Login shell
~/.zprofile       → Login shell
/etc/zshrc        → Interactive shell
~/.zshrc          → Interactive shell
/etc/zlogin       → Login shell
~/.zlogin         → Login shell
```

对于 Zsh 用户，大部分配置放在 `~/.zshrc` 中即可。

### 5. 自定义 PS1 提示符

`PS1` 变量控制命令行提示符的外观。

#### 常用转义序列

| 转义符 | 含义 |
|--------|------|
| `\u` | 用户名 |
| `\h` | 主机名（短） |
| `\H` | 主机名（完整） |
| `\w` | 当前工作目录（完整路径） |
| `\W` | 当前工作目录（仅目录名） |
| `\d` | 日期 |
| `\t` | 时间（24 小时制） |
| `\T` | 时间（12 小时制） |
| `\n` | 换行 |
| `\$` | `$`（普通用户）或 `#`（root） |

#### 示例

```bash
# default style
PS1='\u@\h:\w\$ '
# output: user@hostname:/home/user$

# minimal
PS1='\W \$ '
# output: projects $

# with time
PS1='[\t] \u@\h:\w\$ '
# output: [14:30:05] user@hostname:/home/user$

# colorful prompt
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# two-line prompt
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\n\$ '
```

常用 ANSI 颜色代码：

| 代码 | 颜色 |
|------|------|
| `\033[0;30m` | 黑 |
| `\033[0;31m` | 红 |
| `\033[0;32m` | 绿 |
| `\033[0;33m` | 黄 |
| `\033[0;34m` | 蓝 |
| `\033[0;35m` | 紫 |
| `\033[0;36m` | 青 |
| `\033[01;32m` | 亮绿（粗体） |
| `\033[00m` | 重置颜色 |

> **注意**：在 `PS1` 中使用颜色代码时，必须用 `\[` 和 `\]` 包裹不可见字符，否则会导致终端换行错位。

### 6. alias 命令别名

`alias` 用来创建命令的快捷方式。

```bash
# create alias
alias ll='ls -alh'
alias la='ls -A'
alias gs='git status'
alias gp='git push'
alias dc='docker compose'

# view all aliases
alias

# view a specific alias
alias ll

# remove an alias
unalias ll

# bypass alias (use original command)
\ls
command ls
```

**持久化别名**：在终端中直接输入的 `alias` 仅对当前会话有效。要永久保存，需要写入配置文件：

```bash
# add to ~/.bashrc (or ~/.zshrc)
echo "alias ll='ls -alh'" >> ~/.bashrc

# reload config
source ~/.bashrc
```

实用别名推荐：

```bash
# safety aliases
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ~='cd ~'

# shortcuts
alias h='history'
alias c='clear'
alias ports='ss -tlnp'
alias myip='curl -s ifconfig.me'

# directory sizes
alias duh='du -h --max-depth=1 | sort -hr'
```

### 7. 修改 PATH

#### 临时修改（当前会话）

```bash
# prepend a directory (higher priority)
export PATH="/opt/myapp/bin:$PATH"

# append a directory (lower priority)
export PATH="$PATH:/opt/myapp/bin"

# add current user's bin
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"
```

#### 永久修改

将 `export PATH=...` 写入 Shell 配置文件：

```bash
# for Bash users — add to ~/.bashrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# for Zsh users — add to ~/.zshrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### 检查 PATH 中的重复项

```bash
echo $PATH | tr ':' '\n' | sort | uniq -d
```

> **建议**：在配置文件中修改 `PATH` 时，先检查目录是否已存在，避免重复添加：

```bash
# add to PATH only if not already present
if [[ ":$PATH:" != *":/opt/myapp/bin:"* ]]; then
    export PATH="/opt/myapp/bin:$PATH"
fi
```

---

## 实战演练

### 步骤 1：查看当前环境变量

```bash
# view common environment variables
echo "USER: $USER"
echo "HOME: $HOME"
echo "SHELL: $SHELL"
echo "LANG: $LANG"
echo "EDITOR: $EDITOR"

# view PATH, one per line
echo $PATH | tr ':' '\n'

# count total environment variables
env | wc -l
```

### 步骤 2：理解 Shell 变量 vs 环境变量

```bash
# set a shell variable
TEST_VAR="I am a shell variable"
echo $TEST_VAR

# child process cannot see it
bash -c 'echo "Child sees: [$TEST_VAR]"'

# export it
export TEST_VAR

# now child can see it
bash -c 'echo "Child sees: [$TEST_VAR]"'

# clean up
unset TEST_VAR
```

### 步骤 3：临时为命令设置变量

```bash
# set LANG for a single command
LANG=C ls --help | head -5

# set multiple variables
FOO=bar BAZ=qux bash -c 'echo "FOO=$FOO, BAZ=$BAZ"'
```

### 步骤 4：查看配置文件

```bash
# check which config files exist
ls -la ~/.bashrc ~/.bash_profile ~/.profile ~/.zshrc 2>/dev/null

# view .bashrc content
cat ~/.bashrc | head -30
```

### 步骤 5：自定义 PS1

```bash
# save current PS1
OLD_PS1="$PS1"

# try minimal prompt
PS1='\W \$ '

# try prompt with time
PS1='[\t] \u:\W\$ '

# try colorful prompt
PS1='\[\033[01;32m\]\u\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ '

# restore original
PS1="$OLD_PS1"
```

### 步骤 6：设置 alias

```bash
# create some aliases
alias ll='ls -alh'
alias gs='git status'
alias myip='curl -s ifconfig.me'

# test them
ll /tmp
myip

# view all aliases
alias

# remove
unalias myip
```

### 步骤 7：修改 PATH

```bash
# create a personal bin directory
mkdir -p ~/bin

# create a test script
cat > ~/bin/hello << 'SCRIPT'
#!/bin/bash
echo "Hello from ~/bin!"
SCRIPT
chmod +x ~/bin/hello

# try running it (may fail if ~/bin not in PATH)
hello 2>/dev/null || echo "not found in PATH"

# add ~/bin to PATH
export PATH="$HOME/bin:$PATH"

# now it works
hello

# verify
which hello

# clean up
rm ~/bin/hello
```

---

## 小结

| 命令 / 概念 | 作用 |
|-------------|------|
| `echo $VAR` | 查看变量的值 |
| `env` | 列出所有环境变量 |
| `export VAR=value` | 设置并导出环境变量 |
| `unset VAR` | 删除变量 |
| `VAR=val cmd` | 临时为某个命令设置变量 |
| `source ~/.bashrc` | 重新加载配置文件 |
| `~/.bashrc` | Bash 交互式 shell 配置文件 |
| `~/.bash_profile` | Bash login shell 配置文件 |
| `~/.zshrc` | Zsh 交互式 shell 配置文件 |
| `PS1='...'` | 自定义命令行提示符 |
| `alias name='cmd'` | 创建命令别名 |
| `unalias name` | 删除别名 |
| `export PATH="dir:$PATH"` | 在 PATH 前面添加目录 |

---

## 练习

**1. Shell 变量和环境变量有什么区别？如何将一个 Shell 变量变为环境变量？**

<details>
<summary>查看答案</summary>

- **Shell 变量**：仅在当前 Shell 进程中可见，子进程无法继承
- **环境变量**：当前进程及所有子进程都可以访问

将 Shell 变量转为环境变量：

```bash
MY_VAR="hello"      # shell variable
export MY_VAR       # now it's an environment variable
```

或者一步完成：

```bash
export MY_VAR="hello"
```

</details>

**2. 为什么修改了 `~/.bashrc` 之后，需要执行 `source ~/.bashrc` 才能生效？有没有其他方式？**

<details>
<summary>查看答案</summary>

`~/.bashrc` 仅在新的交互式 Shell 启动时自动加载。修改文件本身不会影响当前正在运行的 Shell。

使更改生效的方法：

```bash
# method 1: source
source ~/.bashrc

# method 2: dot command (equivalent to source)
. ~/.bashrc

# method 3: open a new terminal window/tab
```

</details>

**3. 你安装了一个程序到 `/opt/tools/bin`，但执行时提示 "command not found"。如何永久解决？**

<details>
<summary>查看答案</summary>

在 `~/.bashrc`（或 `~/.zshrc`）中添加：

```bash
export PATH="/opt/tools/bin:$PATH"
```

然后重新加载：

```bash
source ~/.bashrc
```

放在 `$PATH` 前面是为了让 `/opt/tools/bin` 中的命令优先于系统中同名命令。

</details>

**4. 如何创建一个别名 `update`，使其一键执行 `sudo apt update && sudo apt upgrade -y`？如何让它永久生效？**

<details>
<summary>查看答案</summary>

```bash
alias update='sudo apt update && sudo apt upgrade -y'
```

永久生效：将上面这行写入 `~/.bashrc`：

```bash
echo "alias update='sudo apt update && sudo apt upgrade -y'" >> ~/.bashrc
source ~/.bashrc
```

</details>

**5. 解释以下 PS1 设置的效果：`PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '`**

<details>
<summary>查看答案</summary>

拆解分析：

| 部分 | 含义 |
|------|------|
| `\[\033[01;32m\]` | 开始亮绿色 |
| `\u` | 当前用户名 |
| `@` | 字面 `@` 符号 |
| `\h` | 主机名 |
| `\[\033[00m\]` | 重置颜色 |
| `:` | 字面冒号 |
| `\[\033[01;34m\]` | 开始亮蓝色 |
| `\w` | 当前完整路径 |
| `\[\033[00m\]` | 重置颜色 |
| `\$` | `$`（普通用户）或 `#`（root） |

最终效果：`user@hostname:/home/user$`，其中用户名和主机名是绿色，路径是蓝色。这实际上就是 Ubuntu 默认的终端提示符样式。

</details>
