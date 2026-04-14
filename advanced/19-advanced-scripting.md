# 第 19 课：高级脚本技巧

> 上一课：[函数与脚本组织](18-functions.md) | 下一课：[Cron Job 与自动化](20-cron-and-automation.md)

## 学习目标

- 掌握 Bash 数组（索引数组与关联数组）的使用
- 学会字符串操作与参数展开技巧
- 理解命令替换与进程替换的区别
- 掌握 `trap`、`getopts` 和 Here Document 的用法

---

## 知识讲解

### 1. 数组

#### 索引数组（Indexed Array）

```bash
#!/bin/bash

# Declaration
fruits=("apple" "banana" "cherry" "date")

# Access elements (0-based index)
echo "First : ${fruits[0]}"
echo "Third : ${fruits[2]}"

# All elements
echo "All   : ${fruits[@]}"

# Array length
echo "Count : ${#fruits[@]}"

# Append
fruits+=("elderberry")

# Iterate
for fruit in "${fruits[@]}"; do
    echo "- $fruit"
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "  [$i] = ${fruits[$i]}"
done

# Slice: elements 1 through 3
echo "Slice : ${fruits[@]:1:3}"

# Delete element
unset 'fruits[2]'
```

#### 关联数组（Associative Array）

关联数组需要用 `declare -A` 声明（Bash 4+）：

```bash
#!/bin/bash

declare -A config

config[host]="192.168.1.100"
config[port]="8080"
config[env]="production"
config[workers]="4"

echo "Host: ${config[host]}:${config[port]}"
echo "Env : ${config[env]}"

# All keys
echo "Keys  : ${!config[@]}"

# All values
echo "Values: ${config[@]}"

# Check if key exists
if [[ -v config[host] ]]; then
    echo "host is set"
fi

# Iterate
for key in "${!config[@]}"; do
    echo "  $key = ${config[$key]}"
done
```

#### 实用示例：统计词频

```bash
#!/bin/bash

declare -A word_count

while IFS= read -r line; do
    for word in $line; do
        # Convert to lowercase
        word="${word,,}"
        # Remove punctuation
        word="${word//[^a-z]/}"
        [[ -n "$word" ]] && (( word_count[$word]++ ))
    done
done < "$1"

# Sort by frequency
for word in "${!word_count[@]}"; do
    echo "${word_count[$word]} $word"
done | sort -rn | head -20
```

### 2. 字符串操作

#### 长度

```bash
str="Hello, World!"
echo "Length: ${#str}"  # 13
```

#### 子串提取

```bash
str="Hello, World!"

echo "${str:0:5}"   # Hello
echo "${str:7}"     # World!
echo "${str: -6}"   # orld!  (space before - is required)
```

#### 替换

```bash
path="/home/user/docs/report.txt"

echo "${path/user/admin}"      # Replace first match
echo "${path//o/0}"            # Replace all matches

filename="photo.backup.tar.gz"
echo "${filename%.gz}"         # Remove shortest suffix: photo.backup.tar
echo "${filename%%.*}"         # Remove longest suffix: photo
echo "${filename#*.}"          # Remove shortest prefix: backup.tar.gz
echo "${filename##*.}"         # Remove longest prefix: gz
```

#### 大小写转换（Bash 4+）

```bash
str="Hello World"

echo "${str^^}"   # HELLO WORLD (all uppercase)
echo "${str,,}"   # hello world (all lowercase)
echo "${str^}"    # Hello World (first char uppercase)
```

### 3. 参数展开

参数展开用于为变量提供默认值或进行条件赋值：

| 语法 | 含义 |
|------|------|
| `${var:-default}` | 若 `var` 未定义或为空，返回 `default`（不修改 `var`） |
| `${var:=default}` | 若 `var` 未定义或为空，将 `var` 设为 `default` 并返回 |
| `${var:+alt}` | 若 `var` 已定义且非空，返回 `alt`；否则返回空 |
| `${var:?msg}` | 若 `var` 未定义或为空，打印 `msg` 并退出脚本 |

```bash
#!/bin/bash

# Use default if not set
echo "Shell: ${SHELL:-/bin/sh}"

# Set and use default
echo "Editor: ${EDITOR:=/usr/bin/vim}"
echo "EDITOR is now: $EDITOR"

# Provide alternative when set
name="Alice"
echo "${name:+Name is set: $name}"

# Require variable or fail
: "${DB_HOST:?DB_HOST must be set}"
```

实际应用——脚本配置：

```bash
#!/bin/bash

# Configurable with environment variables, sensible defaults
readonly LOG_DIR="${LOG_DIR:-/var/log/myapp}"
readonly MAX_RETRIES="${MAX_RETRIES:-3}"
readonly TIMEOUT="${TIMEOUT:-30}"

echo "LOG_DIR=$LOG_DIR, RETRIES=$MAX_RETRIES, TIMEOUT=$TIMEOUT"
# Override: LOG_DIR=/tmp MAX_RETRIES=5 ./script.sh
```

### 4. 命令替换

#### `$()` vs 反引号

```bash
# Recommended: $()
today=$(date +%Y-%m-%d)
files=$(ls -1 | wc -l)

# Legacy: backticks (avoid - hard to nest)
today=`date +%Y-%m-%d`
```

`$()` 的优势——可以嵌套：

```bash
# Nested command substitution
echo "Kernel config: $(cat /boot/config-$(uname -r) 2>/dev/null | wc -l) lines"
```

#### 算术运算

```bash
a=10; b=3

echo "Sum : $(( a + b ))"
echo "Mod : $(( a % b ))"
echo "Power: $(( a ** b ))"

# Increment
(( a++ ))
echo "a is now $a"
```

### 5. 进程替换

进程替换将命令的输出（或输入）当作临时文件来使用：

#### `<()`：将命令输出当作文件读取

```bash
# Compare output of two commands (like comparing two files)
diff <(ls /usr/bin) <(ls /usr/sbin)

# Compare sorted outputs
diff <(sort file1.txt) <(sort file2.txt)
```

#### `>()`：将文件写入重定向到命令

```bash
# Write to both file and stdout simultaneously
tee >(gzip > output.gz) < input.txt > /dev/null
```

#### 实用场景

```bash
# Join two data sources on a common field
join <(sort -k1,1 users.txt) <(sort -k1,1 orders.txt)

# Compare environment between two shells
diff <(env | sort) <(sudo -u www-data env | sort)

# Process multiple log files together
while IFS= read -r line; do
    echo "[combined] $line"
done < <(cat /var/log/syslog /var/log/auth.log | sort)
```

### 6. `trap` 信号处理

`trap` 用于捕获信号并执行指定命令：

| 信号 | 触发条件 |
|------|---------|
| `EXIT` | 脚本退出（正常或异常） |
| `INT` | Ctrl+C |
| `TERM` | `kill` 默认信号 |
| `HUP` | 终端关闭 |
| `ERR` | 命令失败（需要 `set -e`） |

```bash
#!/bin/bash
set -euo pipefail

tmpdir=$(mktemp -d)
logfile=$(mktemp)

cleanup() {
    echo "Cleaning up..."
    rm -rf "$tmpdir" "$logfile"
}

on_error() {
    local line=$1
    echo "Error on line $line" >&2
}

trap cleanup EXIT
trap 'on_error $LINENO' ERR

echo "Working directory: $tmpdir"
echo "Log file: $logfile"

# Even if something fails, cleanup runs
cp /some/file "$tmpdir/" 2>/dev/null || true
echo "Done"
```

### 7. `getopts`：选项解析

`getopts` 是解析命令行选项的标准工具：

```bash
#!/bin/bash

# Options: -v (verbose), -o <file> (output), -n <num> (count), -h (help)
verbose=false
output="/dev/stdout"
count=1

usage() {
    cat << EOF
Usage: $(basename "$0") [options] <input_file>

Options:
  -v          Verbose mode
  -o <file>   Output file (default: stdout)
  -n <num>    Repeat count (default: 1)
  -h          Show help
EOF
    exit 0
}

while getopts ":vo:n:h" opt; do
    case "$opt" in
        v) verbose=true ;;
        o) output="$OPTARG" ;;
        n) count="$OPTARG" ;;
        h) usage ;;
        :) echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
        *) echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))

# Remaining arguments
[[ $# -eq 0 ]] && { echo "Error: input file required" >&2; exit 1; }
input="$1"

$verbose && echo "Input: $input, Output: $output, Count: $count"

for ((i = 0; i < count; i++)); do
    cat "$input"
done > "$output"
```

使用方式：

```bash
./script.sh -v -n 3 -o result.txt input.txt
./script.sh -h
```

### 8. Here Document

Here Document 允许在脚本中嵌入多行文本：

#### 基本用法

```bash
cat << EOF
Hello, $(whoami)!
Today is $(date +%A).
Your home is $HOME.
EOF
```

#### 禁止变量展开

用引号包裹分隔符：

```bash
cat << 'EOF'
This is literal text.
$HOME will NOT be expanded.
$(date) will NOT run.
EOF
```

#### 去除前导 Tab

使用 `<<-`（仅去除 Tab，不去除空格）：

```bash
if true; then
	cat <<- EOF
	This text will not be indented in output.
	Tabs are stripped.
	EOF
fi
```

#### Here String

将单行字符串传给命令的标准输入：

```bash
# Instead of: echo "hello world" | tr ' ' '\n'
tr ' ' '\n' <<< "hello world"

# Useful with read
IFS=: read -r user _ uid _ <<< "root:x:0:0:root:/root:/bin/bash"
echo "User=$user, UID=$uid"
```

#### 实际应用

```bash
#!/bin/bash

# Generate a config file
cat > /tmp/nginx.conf << 'EOF'
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
}
EOF

# Send email (if mail is configured)
mail -s "Daily Report" admin@example.com << EOF
Server: $(hostname)
Date: $(date)
Disk Usage:
$(df -h / | tail -1)

Load Average: $(uptime | awk -F'load average:' '{print $2}')
EOF

# Generate SQL
mysql -u root << EOF
CREATE DATABASE IF NOT EXISTS myapp;
USE myapp;
CREATE TABLE IF NOT EXISTS logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
EOF
```

---

## 实战演练

### 练习 1：数组操作

```bash
#!/bin/bash

declare -a servers=("web01" "web02" "db01" "cache01" "web03")

echo "=== Server Inventory ==="
echo "Total: ${#servers[@]} servers"

# Filter web servers
echo -e "\nWeb servers:"
for s in "${servers[@]}"; do
    [[ "$s" == web* ]] && echo "  - $s"
done

# Add a server
servers+=("db02")
echo -e "\nAfter adding db02: ${servers[@]}"

# Remove by value
new_servers=()
for s in "${servers[@]}"; do
    [[ "$s" != "cache01" ]] && new_servers+=("$s")
done
servers=("${new_servers[@]}")
echo "After removing cache01: ${servers[@]}"
```

### 练习 2：日志分析器

```bash
cat > ~/log_analyzer.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

: "${1:?Usage: $0 <logfile>}"

logfile="$1"
[[ -f "$logfile" ]] || { echo "File not found: $logfile" >&2; exit 1; }

declare -A status_count
declare -A ip_count
total=0

while IFS= read -r line; do
    # Extract IP (first field)
    ip="${line%% *}"

    # Extract HTTP status code
    if [[ "$line" =~ \" [0-9]{3}\  ]]; then
        status="${BASH_REMATCH[0]//[^0-9]/}"
        (( status_count[$status]++ )) || true
    fi

    (( ip_count[$ip]++ )) || true
    (( total++ ))
done < "$logfile"

echo "===== Log Analysis: $(basename "$logfile") ====="
echo "Total requests: $total"

echo -e "\nStatus Code Distribution:"
for code in $(echo "${!status_count[@]}" | tr ' ' '\n' | sort); do
    printf "  %s: %d (%.1f%%)\n" "$code" "${status_count[$code]}" \
        "$(echo "scale=1; ${status_count[$code]} * 100 / $total" | bc)"
done

echo -e "\nTop 10 IPs:"
for ip in "${!ip_count[@]}"; do
    echo "${ip_count[$ip]} $ip"
done | sort -rn | head -10 | while read -r count ip; do
    printf "  %-15s %d requests\n" "$ip" "$count"
done
SCRIPT

chmod +x ~/log_analyzer.sh
```

### 练习 3：带 `getopts` 的完整工具

```bash
cat > ~/mkproject.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

# Defaults
project_type="basic"
with_git=true
verbose=false

usage() {
    cat << EOF
Usage: $(basename "$0") [-t type] [-G] [-v] [-h] <project_name>

Create a new project directory with standard structure.

Options:
  -t <type>   Project type: basic, web, python (default: basic)
  -G          Skip git init
  -v          Verbose output
  -h          Show help
EOF
    exit 0
}

log() { $verbose && echo "[INFO] $*"; }

while getopts ":t:Gvh" opt; do
    case "$opt" in
        t) project_type="$OPTARG" ;;
        G) with_git=false ;;
        v) verbose=true ;;
        h) usage ;;
        :) echo "Option -$OPTARG requires argument" >&2; exit 1 ;;
        *) echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))

[[ $# -eq 0 ]] && { echo "Error: project name required" >&2; usage; }

name="$1"
[[ -e "$name" ]] && { echo "Error: '$name' already exists" >&2; exit 1; }

log "Creating $project_type project: $name"
mkdir -p "$name"/{src,tests,docs}

case "$project_type" in
    web)
        mkdir -p "$name"/{public,static/{css,js,img},templates}
        log "Created web project directories"
        ;;
    python)
        touch "$name/src/__init__.py" "$name/tests/__init__.py"
        cat > "$name/requirements.txt" <<< "# Python dependencies"
        log "Created Python project files"
        ;;
    basic)
        log "Created basic project directories"
        ;;
    *)
        echo "Unknown project type: $project_type" >&2
        exit 1
        ;;
esac

cat > "$name/README.md" << EOF
# $name

Created: $(date +%Y-%m-%d)
Type: $project_type
EOF

if $with_git; then
    (cd "$name" && git init -q)
    log "Initialized git repository"
fi

echo "Project '$name' created successfully ($project_type)"
SCRIPT

chmod +x ~/mkproject.sh
```

### 练习 4：字符串操作实践

```bash
#!/bin/bash

# Parse a URL
url="https://user:pass@example.com:8080/path/to/page?q=search#section"

protocol="${url%%://*}"
remainder="${url#*://}"

userinfo="${remainder%%@*}"
remainder="${remainder#*@}"

hostport="${remainder%%/*}"
host="${hostport%%:*}"
port="${hostport##*:}"

path_query="${remainder#*/}"
path="/${path_query%%\?*}"
query="${path_query#*\?}"
query="${query%%#*}"
fragment="${path_query##*#}"

echo "Protocol : $protocol"
echo "User     : ${userinfo%%:*}"
echo "Host     : $host"
echo "Port     : $port"
echo "Path     : $path"
echo "Query    : $query"
echo "Fragment : $fragment"
```

---

## 小结

| 概念 | 语法 | 说明 |
|------|------|------|
| 索引数组 | `arr=("a" "b")` / `${arr[0]}` | 从 0 开始编号 |
| 关联数组 | `declare -A m` / `${m[key]}` | Bash 4+，键值对存储 |
| 数组长度 | `${#arr[@]}` | 元素个数 |
| 所有键 | `${!arr[@]}` | 遍历键名 |
| 字符串长度 | `${#str}` | 字符数 |
| 子串 | `${str:offset:length}` | 提取子字符串 |
| 替换 | `${str/old/new}` | 单次替换，`//` 全部替换 |
| 后缀删除 | `${str%pat}` / `${str%%pat}` | 最短 / 最长匹配 |
| 前缀删除 | `${str#pat}` / `${str##pat}` | 最短 / 最长匹配 |
| 默认值 | `${var:-default}` | 未定义时使用默认值 |
| 赋默认值 | `${var:=default}` | 未定义时赋值并使用 |
| 命令替换 | `$(command)` | 捕获命令输出 |
| 进程替换 | `<(cmd)` / `>(cmd)` | 将命令当作文件 |
| 信号捕获 | `trap 'cmd' SIGNAL` | 捕获并处理信号 |
| 选项解析 | `getopts "ab:c" opt` | 解析 `-a -b val` 形式选项 |
| Here Document | `<< EOF ... EOF` | 嵌入多行文本 |
| Here String | `<<< "string"` | 单行文本传入标准输入 |

---

## 练习

**1. 编写一个脚本，使用关联数组统计 `/etc/passwd` 中每种 Shell 的用户数量。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

declare -A shell_count

while IFS=: read -r _ _ _ _ _ _ shell; do
    (( shell_count[$shell]++ ))
done < /etc/passwd

echo "Shell usage in /etc/passwd:"
for shell in "${!shell_count[@]}"; do
    printf "  %-20s %d users\n" "$shell" "${shell_count[$shell]}"
done | sort -t' ' -k2 -rn
```

</details>

**2. 使用参数展开编写一个脚本，从文件路径中提取目录名、文件名、扩展名和不带扩展名的文件名（不使用 `dirname`/`basename` 命令）。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

filepath="${1:?Usage: $0 <filepath>}"

dir="${filepath%/*}"
filename="${filepath##*/}"
extension="${filename##*.}"
name_only="${filename%.*}"

# Handle files without extension
[[ "$extension" == "$filename" ]] && extension=""

echo "Full path : $filepath"
echo "Directory : $dir"
echo "Filename  : $filename"
echo "Name only : $name_only"
echo "Extension : ${extension:-<none>}"
```

</details>

**3. 使用 `getopts` 编写一个 `search.sh` 工具，支持以下选项：`-d <dir>` 指定搜索目录，`-t <type>` 指定文件类型（f/d/l），`-n <name>` 指定文件名模式，`-i` 不区分大小写。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
set -euo pipefail

search_dir="."
file_type=""
name_pattern=""
case_insensitive=false

while getopts ":d:t:n:ih" opt; do
    case "$opt" in
        d) search_dir="$OPTARG" ;;
        t) file_type="$OPTARG" ;;
        n) name_pattern="$OPTARG" ;;
        i) case_insensitive=true ;;
        h)
            echo "Usage: $0 [-d dir] [-t f|d|l] [-n pattern] [-i]"
            exit 0
            ;;
        :) echo "Option -$OPTARG requires argument" >&2; exit 1 ;;
        *) echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done

cmd=(find "$search_dir")
[[ -n "$file_type" ]] && cmd+=(-type "$file_type")

if [[ -n "$name_pattern" ]]; then
    if $case_insensitive; then
        cmd+=(-iname "$name_pattern")
    else
        cmd+=(-name "$name_pattern")
    fi
fi

"${cmd[@]}" 2>/dev/null
```

</details>

**4. 编写一个脚本，使用进程替换 `<()` 比较两个目录的文件列表差异。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
set -euo pipefail

dir1="${1:?Usage: $0 <dir1> <dir2>}"
dir2="${2:?Usage: $0 <dir1> <dir2>}"

[[ -d "$dir1" ]] || { echo "Not a directory: $dir1" >&2; exit 1; }
[[ -d "$dir2" ]] || { echo "Not a directory: $dir2" >&2; exit 1; }

echo "Comparing: $dir1 vs $dir2"
echo "---"

diff_output=$(diff \
    <(cd "$dir1" && find . -type f | sort) \
    <(cd "$dir2" && find . -type f | sort) \
) || true

if [[ -z "$diff_output" ]]; then
    echo "Directories have identical file lists"
else
    echo "Only in $dir1:"
    echo "$diff_output" | grep '^< ' | sed 's/^< /  /'

    echo "Only in $dir2:"
    echo "$diff_output" | grep '^> ' | sed 's/^> /  /'
fi
```

</details>

**5. 编写一个 Here Document 脚本，生成一个 HTML 报告页面，内容包含当前系统的 hostname、日期、磁盘使用率和前 5 个占用内存最多的进程。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash

report_file="/tmp/sysreport_$(date +%Y%m%d).html"

cat > "$report_file" << EOF
<!DOCTYPE html>
<html>
<head><title>System Report - $(hostname)</title>
<style>
  body { font-family: monospace; margin: 2em; }
  table { border-collapse: collapse; }
  th, td { border: 1px solid #ccc; padding: 4px 12px; text-align: left; }
  th { background: #eee; }
</style>
</head>
<body>
<h1>System Report</h1>
<p><strong>Host:</strong> $(hostname)</p>
<p><strong>Date:</strong> $(date)</p>

<h2>Disk Usage</h2>
<pre>$(df -h)</pre>

<h2>Top 5 Memory Consumers</h2>
<pre>$(ps aux --sort=-%mem 2>/dev/null | head -6 || ps aux | head -6)</pre>

</body>
</html>
EOF

echo "Report saved to: $report_file"
```

</details>
