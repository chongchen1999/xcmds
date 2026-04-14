# 第 18 课：函数与脚本组织

> 上一课：[Shell Script 基础](17-scripting-fundamentals.md) | 下一课：[高级脚本技巧](19-advanced-scripting.md)

## 学习目标

- 掌握 Shell 函数的定义与调用方式
- 理解函数参数、返回值和局部变量
- 学会使用 `source` 命令复用代码
- 掌握脚本的错误处理与调试技巧

---

## 知识讲解

### 1. 函数定义

Bash 支持两种函数定义语法：

```bash
# Syntax 1: with 'function' keyword
function greet {
    echo "Hello, $1!"
}

# Syntax 2: POSIX-compatible (recommended)
greet() {
    echo "Hello, $1!"
}
```

调用函数时不需要括号，直接写函数名和参数：

```bash
greet "World"
greet "Linux"
```

> **注意**：函数必须在调用之前定义。

### 2. 函数参数

函数内部使用与脚本相同的位置参数变量：

| 变量 | 含义 |
|------|------|
| `$1` ~ `$N` | 第 N 个参数 |
| `$@` | 所有参数（各自独立） |
| `$*` | 所有参数（合为一个字符串） |
| `$#` | 参数个数 |

```bash
#!/bin/bash

show_args() {
    echo "Function received $# arguments:"
    local i=1
    for arg in "$@"; do
        echo "  \$$i = $arg"
        ((i++))
    done
}

show_args "hello" "world" "foo bar"
```

### 3. 返回值 vs `echo` 输出

#### `return`：设置 Exit Code（0~255）

```bash
is_even() {
    if [[ $(( $1 % 2 )) -eq 0 ]]; then
        return 0  # true
    else
        return 1  # false
    fi
}

if is_even 42; then
    echo "42 is even"
fi
```

#### `echo`：输出字符串，用命令替换捕获

```bash
get_username() {
    local uid="$1"
    # Extract username from /etc/passwd
    awk -F: -v id="$uid" '$3 == id {print $1}' /etc/passwd
}

root_name=$(get_username 0)
echo "UID 0 is: $root_name"
```

> **最佳实践**：用 `return` 表示成功/失败，用 `echo`/`printf` 返回数据。

### 4. 局部变量：`local`

不使用 `local` 的变量默认为全局变量，可能意外覆盖外部变量：

```bash
#!/bin/bash

result="global"

bad_function() {
    result="modified by function"  # Modifies global variable!
}

good_function() {
    local result="local only"     # Only visible inside function
    echo "Inside: $result"
}

bad_function
echo "After bad_function: $result"   # "modified by function"

good_function
echo "After good_function: $result"  # "modified by function" (unchanged)
```

> **规则**：函数内部的变量一律使用 `local` 声明，除非你明确需要修改全局状态。

### 5. `source` / `.` 命令

`source`（或 `.`）在当前 Shell 中执行另一个脚本，常用于加载配置和复用函数库：

```bash
# utils.sh - reusable function library
log_info() {
    echo "[INFO] $(date '+%H:%M:%S') $*"
}

log_error() {
    echo "[ERROR] $(date '+%H:%M:%S') $*" >&2
}

die() {
    log_error "$1"
    exit "${2:-1}"
}
```

```bash
#!/bin/bash
# main.sh - uses the function library

source ./utils.sh  # or: . ./utils.sh

log_info "Script started"
log_info "Processing data..."

[[ -f "data.csv" ]] || die "data.csv not found"

log_info "Done"
```

`source` vs 直接执行的区别：

| 方式 | 环境 | 变量/函数 |
|------|------|----------|
| `source script.sh` | 当前 Shell | 共享，可在当前 Shell 中使用 |
| `./script.sh` | 子 Shell | 隔离，不影响当前 Shell |

### 6. 脚本组织最佳实践

一个组织良好的脚本结构：

```bash
#!/bin/bash
#
# backup.sh - Backup specified directories to tar.gz
# Usage: backup.sh [-v] [-o output_dir] dir1 [dir2 ...]

set -euo pipefail

# --- Constants ---
readonly SCRIPT_NAME="$(basename "$0")"
readonly DEFAULT_OUTPUT="/tmp/backups"

# --- Global variables ---
verbose=false
output_dir="$DEFAULT_OUTPUT"

# --- Functions ---
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [-v] [-o output_dir] dir1 [dir2 ...]

Options:
  -v    Verbose output
  -o    Output directory (default: $DEFAULT_OUTPUT)
  -h    Show this help message
EOF
    exit 0
}

log() {
    [[ "$verbose" == true ]] && echo "[$(date '+%H:%M:%S')] $*"
}

do_backup() {
    local src="$1"
    local name
    name="$(basename "$src")"
    local dest="${output_dir}/${name}_$(date +%Y%m%d).tar.gz"

    log "Backing up $src -> $dest"
    tar -czf "$dest" -C "$(dirname "$src")" "$name"
    echo "Created: $dest"
}

# --- Argument parsing ---
while getopts ":vho:" opt; do
    case "$opt" in
        v) verbose=true ;;
        o) output_dir="$OPTARG" ;;
        h) usage ;;
        *) echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))

[[ $# -eq 0 ]] && { echo "Error: no directories specified" >&2; usage; }

# --- Main logic ---
mkdir -p "$output_dir"

for dir in "$@"; do
    [[ -d "$dir" ]] || { echo "Warning: '$dir' is not a directory, skipping" >&2; continue; }
    do_backup "$dir"
done

log "All backups complete"
```

### 7. 错误处理

#### `set -e`：遇错即停

```bash
set -e  # Exit on any command failure

cp important.conf /backup/
rm temp_file
echo "This won't run if any above command fails"
```

#### `set -u`：禁止未定义变量

```bash
set -u  # Exit on undefined variable

echo "$undefined_var"  # Error!
```

#### `set -o pipefail`：管道任意阶段失败即失败

```bash
set -o pipefail

# Without pipefail: only checks 'wc' exit code
# With pipefail: checks 'cat' exit code too
cat nonexistent.txt | wc -l
```

#### 推荐组合

```bash
#!/bin/bash
set -euo pipefail
```

#### `trap` 进行清理

```bash
#!/bin/bash
set -euo pipefail

tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT  # Clean up on exit (normal or error)

curl -s "https://example.com/data" > "$tmpfile"
wc -l "$tmpfile"
# tmpfile is auto-deleted when script exits
```

### 8. 调试

#### `set -x`：打印每条命令

```bash
#!/bin/bash
set -x  # Enable debug output

name="test"
echo "Hello $name"
```

输出：

```
+ name=test
+ echo 'Hello test'
Hello test
```

#### 局部调试

```bash
#!/bin/bash

echo "Normal output"

set -x
# Only debug this section
result=$(( 2 + 3 ))
echo "Result: $result"
set +x

echo "Back to normal"
```

#### `bash -x`：从外部调试

```bash
bash -x script.sh arg1 arg2
```

#### 自定义 debug 函数

```bash
#!/bin/bash

DEBUG=${DEBUG:-false}

debug() {
    [[ "$DEBUG" == true ]] && echo "[DEBUG] $*" >&2
}

debug "Starting processing"
# Run with: DEBUG=true ./script.sh
```

---

## 实战演练

### 练习 1：构建一个工具函数库

```bash
cat > ~/lib.sh << 'EOF'
#!/bin/bash
# Reusable utility library

confirm() {
    local prompt="${1:-Are you sure?}"
    read -p "$prompt [y/N] " answer
    [[ "$answer" =~ ^[Yy]$ ]]
}

require_command() {
    local cmd="$1"
    if ! command -v "$cmd" &> /dev/null; then
        echo "Error: '$cmd' is required but not installed" >&2
        return 1
    fi
}

file_age_seconds() {
    local file="$1"
    local now
    now=$(date +%s)
    local mod
    mod=$(stat -c %Y "$file" 2>/dev/null || stat -f %m "$file" 2>/dev/null)
    echo $(( now - mod ))
}
EOF
```

### 练习 2：使用函数库写主脚本

```bash
cat > ~/app.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

source ~/lib.sh

require_command "tar" || exit 1
require_command "gzip" || exit 1

echo "All dependencies satisfied"

if confirm "Proceed with backup?"; then
    echo "Starting backup..."
else
    echo "Cancelled"
    exit 0
fi
SCRIPT

chmod +x ~/app.sh
~/app.sh
```

### 练习 3：调试实践

```bash
cat > ~/debug_demo.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

add() {
    local a=$1 b=$2
    echo $(( a + b ))
}

multiply() {
    local a=$1 b=$2
    echo $(( a * b ))
}

x=5
y=3

sum=$(add $x $y)
product=$(multiply $x $y)

echo "$x + $y = $sum"
echo "$x * $y = $product"
SCRIPT

chmod +x ~/debug_demo.sh

# Normal run
~/debug_demo.sh

# Debug run
bash -x ~/debug_demo.sh
```

### 练习 4：带错误处理的完整脚本

```bash
cat > ~/safe_copy.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

die() { echo "$SCRIPT_NAME: Error: $1" >&2; exit "${2:-1}"; }

usage() {
    echo "Usage: $SCRIPT_NAME <source> <destination>"
    exit 0
}

[[ $# -eq 2 ]] || die "Expected 2 arguments, got $#"

src="$1"
dest="$2"

[[ -e "$src" ]] || die "'$src' does not exist"
[[ -d "$dest" ]] || die "'$dest' is not a directory"

# Create timestamped copy
base="$(basename "$src")"
timestamp="$(date +%Y%m%d_%H%M%S)"
target="${dest}/${base}.${timestamp}"

cp -a "$src" "$target"
echo "Copied: $src -> $target"
SCRIPT

chmod +x ~/safe_copy.sh
```

---

## 小结

| 概念 | 语法 / 命令 | 说明 |
|------|------------|------|
| 函数定义 | `func() { ... }` | POSIX 兼容写法 |
| 函数参数 | `$1` `$@` `$#` | 与脚本参数用法一致 |
| 返回状态 | `return 0` / `return 1` | 只能返回 0~255 |
| 返回数据 | `echo` + `$(func)` | 用命令替换捕获输出 |
| 局部变量 | `local var=value` | 避免污染全局作用域 |
| 加载脚本 | `source file.sh` | 在当前 Shell 中执行 |
| 遇错即停 | `set -e` | 命令失败时退出脚本 |
| 禁止未定义 | `set -u` | 引用未定义变量时报错 |
| 管道严格模式 | `set -o pipefail` | 管道任一环节失败即失败 |
| 调试模式 | `set -x` / `bash -x` | 打印每条执行的命令 |

---

## 练习

**1. 编写一个函数 `is_valid_ip`，判断传入的字符串是否为合法的 IPv4 地址。**

<details>
<summary>查看答案</summary>

```bash
is_valid_ip() {
    local ip="$1"
    local IFS='.'
    read -ra octets <<< "$ip"

    [[ ${#octets[@]} -ne 4 ]] && return 1

    for octet in "${octets[@]}"; do
        # Must be a number between 0-255
        [[ "$octet" =~ ^[0-9]+$ ]] || return 1
        (( octet < 0 || octet > 255 )) && return 1
    done
    return 0
}

# Test
for ip in "192.168.1.1" "256.1.2.3" "10.0.0" "abc.def.ghi.jkl"; do
    if is_valid_ip "$ip"; then
        echo "$ip -> valid"
    else
        echo "$ip -> invalid"
    fi
done
```

</details>

**2. 编写一个函数库文件 `math_lib.sh`，包含 `max`、`min`、`abs` 三个函数，然后在另一个脚本中 `source` 它并调用。**

<details>
<summary>查看答案</summary>

```bash
# math_lib.sh
max() { (( $1 > $2 )) && echo "$1" || echo "$2"; }
min() { (( $1 < $2 )) && echo "$1" || echo "$2"; }
abs() { local n=$1; (( n < 0 )) && echo $(( -n )) || echo "$n"; }
```

```bash
#!/bin/bash
# test_math.sh
source ./math_lib.sh

echo "max(3, 7) = $(max 3 7)"
echo "min(3, 7) = $(min 3 7)"
echo "abs(-5) = $(abs -5)"
```

</details>

**3. 解释 `set -euo pipefail` 中每个选项的作用，并说明为什么推荐在脚本开头使用。**

<details>
<summary>查看答案</summary>

- `set -e`：任何命令返回非零 Exit Code 时，立即退出脚本，防止错误被忽略后继续执行导致更大问题。
- `set -u`：引用未定义的变量时报错退出，防止因拼写错误导致变量为空而产生意外行为（例如 `rm -rf /$undefined_var` 变成 `rm -rf /`）。
- `set -o pipefail`：管道中任意一个命令失败时，整个管道的 Exit Code 为失败，而不是只看最后一个命令。例如 `cat nonexistent | wc -l` 在没有 pipefail 时会返回 0。

组合使用可以构成 Bash "严格模式"，让脚本更安全、更容易排查问题。

</details>

**4. 编写一个脚本，使用 `trap` 确保临时文件在脚本退出（包括被 Ctrl+C 中断）时被清理。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
set -euo pipefail

tmpdir=$(mktemp -d)
trap 'echo "Cleaning up $tmpdir"; rm -rf "$tmpdir"' EXIT

echo "Working in: $tmpdir"
echo "some data" > "$tmpdir/data.txt"
echo "more data" > "$tmpdir/log.txt"

echo "Files created:"
ls -la "$tmpdir"

echo "Press Ctrl+C or let it finish..."
sleep 3

echo "Normal exit"
# tmpdir is cleaned up automatically
```

</details>
