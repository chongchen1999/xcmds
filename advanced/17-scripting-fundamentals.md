# 第 17 课：Shell Script 基础

> 上一课：[Intermediate Quiz](../intermediate/16-intermediate-quiz.md) | 下一课：[函数与脚本组织](18-functions.md)

## 学习目标

- 理解 Shell Script 的基本结构与执行方式
- 掌握变量声明、用户输入和条件判断
- 学会使用循环和 `case` 语句控制流程
- 理解 Exit Code 的含义与使用

---

## 知识讲解

### 1. 什么是 Shell Script

Shell Script 是一系列 Shell 命令的集合，保存为文件后可一次性执行。它是 Linux 系统管理和自动化的核心工具。

### 2. Shebang 与脚本执行

每个脚本的第一行应指定解释器：

```bash
#!/bin/bash
```

创建并运行脚本的两种方式：

```bash
# Method 1: make it executable
chmod +x script.sh
./script.sh

# Method 2: run with interpreter directly
bash script.sh
```

### 3. 变量

#### 声明与引用

```bash
#!/bin/bash

name="Linux"
version=6

echo "System: $name"
echo "Version: ${version}.0"
```

> **注意**：等号两边不能有空格，`name = "Linux"` 会报错。

#### `readonly` 常量

```bash
readonly PI=3.14159
PI=3.14  # Error: PI is readonly
```

#### 特殊变量

| 变量 | 含义 |
|------|------|
| `$0` | 脚本文件名 |
| `$1` ~ `$9` | 位置参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数（独立字符串） |
| `$*` | 所有参数（单个字符串） |
| `$$` | 当前进程 PID |
| `$?` | 上一条命令的 Exit Code |

### 4. 用户输入：`read`

```bash
#!/bin/bash

read -p "Enter your name: " username
echo "Hello, $username!"

# Read with timeout (5 seconds)
read -t 5 -p "Quick! Enter a number: " num

# Read password (hidden input)
read -s -p "Password: " password
echo
```

### 5. 条件判断

#### `if / elif / else / fi`

```bash
#!/bin/bash

score=$1

if [[ $score -ge 90 ]]; then
    echo "Grade: A"
elif [[ $score -ge 70 ]]; then
    echo "Grade: B"
elif [[ $score -ge 60 ]]; then
    echo "Grade: C"
else
    echo "Grade: F"
fi
```

#### `test`、`[ ]` 与 `[[ ]]`

这三者功能类似，`[[ ]]` 是 Bash 增强版，推荐优先使用：

```bash
# All equivalent
test -f /etc/passwd
[ -f /etc/passwd ]
[[ -f /etc/passwd ]]
```

`[[ ]]` 的优势：
- 支持 `&&` 和 `||` 逻辑运算
- 支持正则匹配 `=~`
- 变量无需加引号也不会因空值报错

#### 数值比较运算符

| 运算符 | 含义 |
|--------|------|
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-lt` | 小于 |
| `-ge` | 大于等于 |
| `-le` | 小于等于 |

#### 字符串比较运算符

| 运算符 | 含义 |
|--------|------|
| `=` / `==` | 字符串相等 |
| `!=` | 字符串不等 |
| `-z` | 字符串为空 |
| `-n` | 字符串非空 |

```bash
#!/bin/bash

str="hello"

if [[ -z "$str" ]]; then
    echo "String is empty"
elif [[ "$str" == "hello" ]]; then
    echo "String is hello"
fi
```

#### 文件测试运算符

| 运算符 | 含义 |
|--------|------|
| `-f` | 是普通文件 |
| `-d` | 是目录 |
| `-e` | 文件/目录存在 |
| `-r` | 可读 |
| `-w` | 可写 |
| `-x` | 可执行 |
| `-s` | 文件非空 |

```bash
#!/bin/bash

file="/etc/passwd"

if [[ -f "$file" && -r "$file" ]]; then
    echo "$file exists and is readable"
    echo "Lines: $(wc -l < "$file")"
fi
```

### 6. 循环

#### `for` 循环

```bash
#!/bin/bash

# Iterate over a list
for fruit in apple banana cherry; do
    echo "I like $fruit"
done

# C-style for loop
for ((i = 1; i <= 5; i++)); do
    echo "Count: $i"
done

# Iterate over files
for file in /var/log/*.log; do
    echo "Log file: $file ($(du -h "$file" | cut -f1))"
done
```

#### `while` 循环

```bash
#!/bin/bash

count=1
while [[ $count -le 5 ]]; do
    echo "Iteration $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo ">> $line"
done < /etc/hostname
```

#### `until` 循环

`until` 在条件为**假**时循环，为真时停止（与 `while` 相反）：

```bash
#!/bin/bash

num=1
until [[ $num -gt 5 ]]; do
    echo "Number: $num"
    ((num++))
done
```

#### `break` 与 `continue`

```bash
#!/bin/bash

for i in {1..10}; do
    if [[ $i -eq 3 ]]; then
        continue  # Skip 3
    fi
    if [[ $i -eq 8 ]]; then
        break     # Stop at 8
    fi
    echo "$i"
done
```

### 7. `case` 语句

```bash
#!/bin/bash

read -p "Enter a command (start|stop|restart): " action

case "$action" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Unknown action: $action"
        echo "Usage: start|stop|restart"
        exit 1
        ;;
esac
```

`case` 支持通配符匹配：

```bash
#!/bin/bash

read -p "Enter a filename: " filename

case "$filename" in
    *.tar.gz | *.tgz)
        tar -xzf "$filename"
        ;;
    *.zip)
        unzip "$filename"
        ;;
    *.txt)
        cat "$filename"
        ;;
    *)
        echo "Unsupported file type"
        ;;
esac
```

### 8. Exit Code

每条命令执行后会返回一个 Exit Code（0~255），`0` 表示成功，非零表示失败：

```bash
#!/bin/bash

ls /etc/passwd > /dev/null 2>&1
echo "Exit code: $?"  # 0

ls /nonexistent 2> /dev/null
echo "Exit code: $?"  # 2 (not found)

# Use exit to set script's return code
check_root() {
    if [[ $EUID -ne 0 ]]; then
        echo "This script must be run as root"
        exit 1
    fi
}
```

在条件判断中利用 Exit Code：

```bash
if grep -q "root" /etc/passwd; then
    echo "root user found"
fi

# && and || as short-circuit operators
[[ -d /tmp ]] && echo "/tmp exists"
[[ -d /nope ]] || echo "/nope does not exist"
```

---

## 实战演练

### 练习 1：编写第一个脚本

```bash
# Create the script
cat > ~/hello.sh << 'EOF'
#!/bin/bash
echo "Hello, $(whoami)!"
echo "Today is $(date +%Y-%m-%d)"
echo "You are in: $(pwd)"
EOF

chmod +x ~/hello.sh
~/hello.sh
```

### 练习 2：系统信息脚本

```bash
cat > ~/sysinfo.sh << 'SCRIPT'
#!/bin/bash

echo "===== System Information ====="
echo "Hostname : $(hostname)"
echo "OS       : $(uname -s)"
echo "Kernel   : $(uname -r)"
echo "Uptime   : $(uptime -p 2>/dev/null || uptime)"
echo "User     : $(whoami)"
echo "Shell    : $SHELL"
echo "Disk (/) : $(df -h / | awk 'NR==2{print $5 " used"}')"
echo "Memory   : $(free -h 2>/dev/null | awk '/Mem/{print $3 "/" $2}')"
SCRIPT

chmod +x ~/sysinfo.sh
~/sysinfo.sh
```

### 练习 3：带参数的文件检查脚本

```bash
cat > ~/filecheck.sh << 'SCRIPT'
#!/bin/bash

if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <filepath>"
    exit 1
fi

target="$1"

if [[ ! -e "$target" ]]; then
    echo "'$target' does not exist"
    exit 1
fi

if [[ -f "$target" ]]; then
    echo "Type     : Regular file"
    echo "Size     : $(du -h "$target" | cut -f1)"
    echo "Lines    : $(wc -l < "$target")"
elif [[ -d "$target" ]]; then
    echo "Type     : Directory"
    echo "Contents : $(ls -1 "$target" | wc -l) items"
fi

echo "Readable : $([[ -r "$target" ]] && echo Yes || echo No)"
echo "Writable : $([[ -w "$target" ]] && echo Yes || echo No)"
echo "Executable: $([[ -x "$target" ]] && echo Yes || echo No)"
SCRIPT

chmod +x ~/filecheck.sh
~/filecheck.sh /etc/passwd
~/filecheck.sh /tmp
~/filecheck.sh /nonexistent
```

### 练习 4：批量重命名脚本

```bash
cat > ~/rename_ext.sh << 'SCRIPT'
#!/bin/bash

if [[ $# -ne 2 ]]; then
    echo "Usage: $0 <old_ext> <new_ext>"
    echo "Example: $0 txt md"
    exit 1
fi

old_ext="$1"
new_ext="$2"
count=0

for file in *."$old_ext"; do
    if [[ -f "$file" ]]; then
        base="${file%.$old_ext}"
        mv "$file" "${base}.${new_ext}"
        echo "Renamed: $file -> ${base}.${new_ext}"
        ((count++))
    fi
done

echo "Total: $count file(s) renamed"
SCRIPT

chmod +x ~/rename_ext.sh
```

---

## 小结

| 概念 | 语法 / 命令 | 说明 |
|------|------------|------|
| Shebang | `#!/bin/bash` | 指定脚本解释器 |
| 变量 | `var=value` / `$var` | 赋值不加空格，引用加 `$` |
| 只读变量 | `readonly var=value` | 不可修改 |
| 用户输入 | `read -p "prompt" var` | 从终端读取输入 |
| 条件判断 | `if [[ ... ]]; then ... fi` | 推荐使用 `[[ ]]` |
| 数值比较 | `-eq` `-ne` `-gt` `-lt` | 用于整数比较 |
| 字符串比较 | `=` `!=` `-z` `-n` | 用于字符串判断 |
| 文件测试 | `-f` `-d` `-e` `-r` `-w` `-x` | 判断文件属性 |
| `for` 循环 | `for x in list; do ... done` | 遍历列表 |
| `while` 循环 | `while [[ ... ]]; do ... done` | 条件为真时循环 |
| `until` 循环 | `until [[ ... ]]; do ... done` | 条件为假时循环 |
| `case` | `case $var in ... esac` | 多分支匹配 |
| Exit Code | `$?` / `exit N` | 0 成功，非零失败 |

---

## 练习

**1. 编写一个脚本，接受一个数字参数，判断它是奇数还是偶数。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <number>"
    exit 1
fi

if [[ $(( $1 % 2 )) -eq 0 ]]; then
    echo "$1 is even"
else
    echo "$1 is odd"
fi
```

</details>

**2. 编写一个脚本，使用 `for` 循环计算 1 到 100 的总和。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

sum=0
for ((i = 1; i <= 100; i++)); do
    ((sum += i))
done
echo "Sum of 1~100: $sum"
```

</details>

**3. 编写一个脚本，使用 `while` 循环读取 `/etc/shells`，统计共有多少种 Shell。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

count=0
while IFS= read -r line; do
    # Skip comments and empty lines
    [[ "$line" =~ ^#.*$ || -z "$line" ]] && continue
    echo "$line"
    ((count++))
done < /etc/shells

echo "Total shells: $count"
```

</details>

**4. 编写一个脚本，接受用户名作为参数，使用 `case` 判断该用户的角色（root=管理员，www-data/nginx=Web 服务，其他=普通用户）。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <username>"
    exit 1
fi

case "$1" in
    root)
        echo "$1 -> Administrator"
        ;;
    www-data | nginx | apache)
        echo "$1 -> Web Service Account"
        ;;
    nobody)
        echo "$1 -> System Daemon Account"
        ;;
    *)
        echo "$1 -> Regular User"
        ;;
esac
```

</details>

**5. 编写一个脚本，检查当前目录下是否存在 `.env` 文件。如果存在且可读，显示其内容（隐藏包含 `PASSWORD` 的行）；如果不存在，提示用户。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

envfile=".env"

if [[ ! -f "$envfile" ]]; then
    echo "No $envfile file found in current directory"
    exit 1
fi

if [[ ! -r "$envfile" ]]; then
    echo "$envfile exists but is not readable"
    exit 1
fi

echo "Contents of $envfile (passwords hidden):"
while IFS= read -r line; do
    if [[ "$line" == *PASSWORD* ]]; then
        key="${line%%=*}"
        echo "${key}=********"
    else
        echo "$line"
    fi
done < "$envfile"
```

</details>
