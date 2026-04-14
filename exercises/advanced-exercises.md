# 高级篇综合练习

> 本练习集是高级篇（第 17-23 课）的补充训练，侧重于 **Shell 脚本编写、系统管理与性能调优** 的综合实战。
> 部分练习需要编写完整脚本，建议在真实环境中运行和调试。
>
> 建议先完成各课的课后练习和[高级综合实战项目](../advanced/24-capstone-project.md)后再来挑战。
>
> 📖 返回[课程目录](../README.md)

---

## 一、Shell Script 基础

### 练习 1：参数化备份脚本

编写脚本 `backup.sh`，实现以下功能：

- 接受两个参数：源目录和备份目标目录
- 如果参数不足，打印用法说明并退出
- 检查源目录是否存在
- 在目标目录下创建以当前日期命名的备份文件夹（如 `backup-2026-01-15`）
- 将源目录的内容复制到备份文件夹
- 打印备份完成的摘要（文件数量、总大小）

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Parameterized directory backup script

if [ $# -lt 2 ]; then
    echo "Usage: $0 <source_dir> <backup_dir>"
    exit 1
fi

SRC="$1"
DEST="$2"
DATE=$(date +%Y-%m-%d)
BACKUP_PATH="${DEST}/backup-${DATE}"

if [ ! -d "$SRC" ]; then
    echo "Error: source directory '$SRC' does not exist"
    exit 1
fi

mkdir -p "$BACKUP_PATH"
cp -r "$SRC"/* "$BACKUP_PATH"/ 2>/dev/null

FILE_COUNT=$(find "$BACKUP_PATH" -type f | wc -l)
TOTAL_SIZE=$(du -sh "$BACKUP_PATH" | awk '{print $1}')

echo "Backup complete:"
echo "  Source:      $SRC"
echo "  Destination: $BACKUP_PATH"
echo "  Files:       $FILE_COUNT"
echo "  Total size:  $TOTAL_SIZE"
```

测试：
```bash
mkdir -p /tmp/test-src && touch /tmp/test-src/{a,b,c}.txt
bash backup.sh /tmp/test-src /tmp/backups
```

</details>

### 练习 2：交互式菜单系统

编写脚本 `system-menu.sh`，显示一个系统信息菜单：

- 选项 1：显示系统信息（hostname、kernel、uptime）
- 选项 2：显示磁盘使用情况
- 选项 3：显示内存使用情况
- 选项 4：显示当前登录用户
- 选项 5：退出
- 选择无效选项时提示重新选择
- 每次操作完成后回到菜单（循环）

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Interactive system info menu

while true; do
    echo ""
    echo "===== System Info Menu ====="
    echo "1) System overview"
    echo "2) Disk usage"
    echo "3) Memory usage"
    echo "4) Logged-in users"
    echo "5) Exit"
    echo "============================"
    read -p "Choose [1-5]: " choice

    case $choice in
        1)
            echo "--- System Overview ---"
            echo "Hostname: $(hostname)"
            echo "Kernel:   $(uname -r)"
            echo "Uptime:   $(uptime -p 2>/dev/null || uptime)"
            ;;
        2)
            echo "--- Disk Usage ---"
            df -h | grep -v tmpfs
            ;;
        3)
            echo "--- Memory Usage ---"
            free -h 2>/dev/null || vm_stat 2>/dev/null
            ;;
        4)
            echo "--- Logged-in Users ---"
            who
            ;;
        5)
            echo "Bye!"
            exit 0
            ;;
        *)
            echo "Invalid option. Please choose 1-5."
            ;;
    esac
done
```

</details>

### 练习 3：批量文件重命名

编写脚本 `rename-files.sh`，实现批量重命名功能：

- 接受参数：目录路径、旧后缀、新后缀
- 预览模式：先显示即将进行的重命名操作（不实际执行）
- 用户确认后才实际执行
- 统计并报告重命名的文件数量

测试场景：将一个目录下所有 `.txt` 文件改为 `.md`

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Batch file rename with preview

if [ $# -lt 3 ]; then
    echo "Usage: $0 <directory> <old_ext> <new_ext>"
    echo "Example: $0 /tmp/docs txt md"
    exit 1
fi

DIR="$1"
OLD_EXT="$2"
NEW_EXT="$3"
COUNT=0

if [ ! -d "$DIR" ]; then
    echo "Error: directory '$DIR' does not exist"
    exit 1
fi

echo "Preview: the following files will be renamed:"
echo ""

for file in "$DIR"/*."$OLD_EXT"; do
    [ -f "$file" ] || continue
    newname="${file%.$OLD_EXT}.$NEW_EXT"
    echo "  $file -> $newname"
    COUNT=$((COUNT + 1))
done

if [ $COUNT -eq 0 ]; then
    echo "No .$OLD_EXT files found in $DIR"
    exit 0
fi

echo ""
read -p "Rename $COUNT file(s)? [y/N]: " confirm
if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
    echo "Cancelled."
    exit 0
fi

for file in "$DIR"/*."$OLD_EXT"; do
    [ -f "$file" ] || continue
    mv "$file" "${file%.$OLD_EXT}.$NEW_EXT"
done

echo "Done. Renamed $COUNT file(s)."
```

测试：
```bash
mkdir -p /tmp/docs
touch /tmp/docs/{readme,notes,guide}.txt
bash rename-files.sh /tmp/docs txt md
ls /tmp/docs
```

</details>

---

## 二、函数与脚本组织

### 练习 4：日志工具库

创建一个可复用的日志函数库 `log-lib.sh`，然后在主脚本中 `source` 引用它：

**log-lib.sh** 需包含：
- `log_info` 函数：打印 `[INFO]` 级别日志（绿色）
- `log_warn` 函数：打印 `[WARN]` 级别日志（黄色）
- `log_error` 函数：打印 `[ERROR]` 级别日志（红色）
- 每条日志自动带时间戳
- 可选：将日志同时写入文件

然后写主脚本 `deploy.sh`，模拟一个部署流程，调用日志函数记录每一步。

<details>
<summary>查看答案</summary>

**log-lib.sh:**
```bash
#!/bin/bash
# Reusable logging library with color output and optional file logging

LOG_FILE="${LOG_FILE:-}"

_log() {
    local level="$1" color="$2" msg="$3"
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local entry="[$timestamp] [$level] $msg"

    echo -e "${color}${entry}\033[0m"

    if [ -n "$LOG_FILE" ]; then
        echo "$entry" >> "$LOG_FILE"
    fi
}

log_info()  { _log "INFO"  "\033[32m" "$1"; }
log_warn()  { _log "WARN"  "\033[33m" "$1"; }
log_error() { _log "ERROR" "\033[31m" "$1"; }
```

**deploy.sh:**
```bash
#!/bin/bash
# Simulated deployment script using log library

source ./log-lib.sh
export LOG_FILE="/tmp/deploy.log"

log_info "Starting deployment..."

log_info "Step 1: Pulling latest code"
sleep 1

log_info "Step 2: Installing dependencies"
sleep 1

log_warn "Step 3: Cache directory not found, creating..."
mkdir -p /tmp/app-cache

log_info "Step 4: Running tests"
sleep 1

if [ ! -f "/tmp/fake-binary" ]; then
    log_error "Build artifact not found!"
    log_warn "Skipping deploy, please build first"
    exit 1
fi

log_info "Deployment finished successfully"
```

测试：
```bash
bash deploy.sh
cat /tmp/deploy.log
```

</details>

### 练习 5：模块化系统检查工具

编写一个模块化的系统健康检查工具，包含：

- `checks/check-disk.sh` —— 检查磁盘使用率，超过阈值则告警
- `checks/check-memory.sh` —— 检查内存使用率
- `checks/check-process.sh` —— 检查关键进程是否运行
- `health-check.sh` —— 主脚本，依次调用所有检查模块，汇总结果

每个检查模块返回 0 表示正常，1 表示异常。主脚本根据返回值统计结果。

<details>
<summary>查看答案</summary>

**checks/check-disk.sh:**
```bash
#!/bin/bash
# Check disk usage against threshold
THRESHOLD="${DISK_THRESHOLD:-80}"

max_usage=$(df -h | awk 'NR>1 {gsub(/%/,"",$5); if($5+0 > max) max=$5} END {print max}')

if [ "$max_usage" -gt "$THRESHOLD" ] 2>/dev/null; then
    echo "FAIL: Disk usage ${max_usage}% exceeds threshold ${THRESHOLD}%"
    return 1 2>/dev/null || exit 1
else
    echo "OK: Disk usage ${max_usage}% within threshold ${THRESHOLD}%"
    return 0 2>/dev/null || exit 0
fi
```

**checks/check-memory.sh:**
```bash
#!/bin/bash
# Check memory usage
if command -v free &>/dev/null; then
    usage=$(free | awk '/Mem:/ {printf "%.0f", $3/$2*100}')
    echo "OK: Memory usage ${usage}%"
else
    echo "SKIP: 'free' command not available"
fi
return 0 2>/dev/null || exit 0
```

**checks/check-process.sh:**
```bash
#!/bin/bash
# Check if critical processes are running
CRITICAL_PROCS="${CRITICAL_PROCS:-sshd cron}"
FAILED=0

for proc in $CRITICAL_PROCS; do
    if pgrep -x "$proc" &>/dev/null; then
        echo "OK: $proc is running"
    else
        echo "FAIL: $proc is NOT running"
        FAILED=1
    fi
done

return $FAILED 2>/dev/null || exit $FAILED
```

**health-check.sh:**
```bash
#!/bin/bash
# Main health check script, loads and runs all check modules

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
CHECK_DIR="$SCRIPT_DIR/checks"
PASS=0
FAIL=0

echo "===== System Health Check ====="
echo "Date: $(date)"
echo ""

for check in "$CHECK_DIR"/check-*.sh; do
    [ -f "$check" ] || continue
    name=$(basename "$check" .sh)
    echo "--- Running: $name ---"

    source "$check"
    if [ $? -eq 0 ]; then
        PASS=$((PASS + 1))
    else
        FAIL=$((FAIL + 1))
    fi
    echo ""
done

echo "===== Summary ====="
echo "Passed: $PASS"
echo "Failed: $FAIL"

[ $FAIL -eq 0 ] && echo "Overall: HEALTHY" || echo "Overall: UNHEALTHY"
exit $FAIL
```

</details>

---

## 三、高级脚本技巧

### 练习 6：带选项的命令行工具

编写脚本 `logparser.sh`，使用 `getopts` 处理命令行选项：

```
Usage: logparser.sh [-l LEVEL] [-n COUNT] [-o OUTPUT] [-h] <logfile>
  -l LEVEL   Filter by log level (INFO, WARN, ERROR)
  -n COUNT   Show only the last COUNT matching lines
  -o OUTPUT  Save results to OUTPUT file
  -h         Show this help message
```

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Log parser with command-line options via getopts

usage() {
    echo "Usage: $0 [-l LEVEL] [-n COUNT] [-o OUTPUT] [-h] <logfile>"
    echo "  -l LEVEL   Filter by log level (INFO, WARN, ERROR)"
    echo "  -n COUNT   Show only the last COUNT matching lines"
    echo "  -o OUTPUT  Save results to OUTPUT file"
    echo "  -h         Show this help message"
    exit 1
}

LEVEL=""
COUNT=""
OUTPUT=""

while getopts "l:n:o:h" opt; do
    case $opt in
        l) LEVEL="$OPTARG" ;;
        n) COUNT="$OPTARG" ;;
        o) OUTPUT="$OPTARG" ;;
        h) usage ;;
        *) usage ;;
    esac
done

shift $((OPTIND - 1))

if [ $# -lt 1 ]; then
    echo "Error: logfile is required"
    usage
fi

LOGFILE="$1"
if [ ! -f "$LOGFILE" ]; then
    echo "Error: file '$LOGFILE' not found"
    exit 1
fi

RESULT=$(cat "$LOGFILE")

if [ -n "$LEVEL" ]; then
    RESULT=$(echo "$RESULT" | grep "\[$LEVEL\]")
fi

if [ -n "$COUNT" ]; then
    RESULT=$(echo "$RESULT" | tail -n "$COUNT")
fi

if [ -n "$OUTPUT" ]; then
    echo "$RESULT" > "$OUTPUT"
    echo "Results saved to $OUTPUT ($(echo "$RESULT" | wc -l) lines)"
else
    echo "$RESULT"
fi
```

测试：
```bash
# Using the log file from intermediate exercise 13
bash logparser.sh -l ERROR /tmp/webapp.log
bash logparser.sh -l INFO -n 3 /tmp/webapp.log
bash logparser.sh -l WARN -o /tmp/warnings.txt /tmp/webapp.log
```

</details>

### 练习 7：安全的临时文件与 trap

编写脚本 `safe-process.sh`，演示 `trap` 的正确使用：

- 创建临时目录存放中间文件
- 设置 `trap`，确保脚本在正常退出、被中断（Ctrl+C）、或遇到错误时都能清理临时文件
- 模拟一个多步骤处理流程
- 每一步创建临时文件
- 最后汇总结果并清理

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Demonstrates safe temp file handling with trap

TMPDIR=$(mktemp -d /tmp/safe-process.XXXXXX)
echo "Working directory: $TMPDIR"

cleanup() {
    echo ""
    echo "Cleaning up temp files in $TMPDIR ..."
    rm -rf "$TMPDIR"
    echo "Cleanup complete."
}

trap cleanup EXIT INT TERM

echo "Step 1: Generating data..."
seq 1 1000 > "$TMPDIR/raw-data.txt"
echo "  Created raw-data.txt ($(wc -l < "$TMPDIR/raw-data.txt") lines)"

echo "Step 2: Filtering even numbers..."
awk '$1 % 2 == 0' "$TMPDIR/raw-data.txt" > "$TMPDIR/even.txt"
echo "  Created even.txt ($(wc -l < "$TMPDIR/even.txt") lines)"

echo "Step 3: Calculating sum..."
SUM=$(awk '{s+=$1} END {print s}' "$TMPDIR/even.txt")
echo "$SUM" > "$TMPDIR/result.txt"

echo ""
echo "Result: Sum of even numbers from 1-1000 = $SUM"
echo ""
echo "Temp files before cleanup:"
ls -la "$TMPDIR"

# cleanup happens automatically via trap on EXIT
```

测试：
```bash
bash safe-process.sh

# Test interrupt handling: run and press Ctrl+C during execution
bash safe-process.sh  # press Ctrl+C quickly
ls /tmp/safe-process.*  # should be cleaned up
```

</details>

### 练习 8：数组与关联数组

编写脚本 `inventory.sh`，使用数组管理服务器清单：

- 用普通数组存储服务器列表
- 用关联数组存储每台服务器的角色（web/db/cache）
- 实现功能：添加服务器、列出所有服务器、按角色筛选、删除服务器
- 格式化输出服务器列表

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Server inventory manager using arrays (requires bash 4+)

declare -a SERVERS=()
declare -A ROLES=()

add_server() {
    local name="$1" role="$2"
    SERVERS+=("$name")
    ROLES["$name"]="$role"
    echo "Added: $name ($role)"
}

list_servers() {
    if [ ${#SERVERS[@]} -eq 0 ]; then
        echo "No servers in inventory."
        return
    fi
    printf "%-15s %-10s\n" "SERVER" "ROLE"
    printf "%-15s %-10s\n" "------" "----"
    for srv in "${SERVERS[@]}"; do
        printf "%-15s %-10s\n" "$srv" "${ROLES[$srv]}"
    done
}

filter_by_role() {
    local target="$1"
    echo "Servers with role '$target':"
    for srv in "${SERVERS[@]}"; do
        if [ "${ROLES[$srv]}" = "$target" ]; then
            echo "  - $srv"
        fi
    done
}

remove_server() {
    local target="$1"
    local new_servers=()
    for srv in "${SERVERS[@]}"; do
        if [ "$srv" != "$target" ]; then
            new_servers+=("$srv")
        fi
    done
    SERVERS=("${new_servers[@]}")
    unset ROLES["$target"]
    echo "Removed: $target"
}

# Demo
add_server "web-01" "web"
add_server "web-02" "web"
add_server "db-01" "db"
add_server "db-02" "db"
add_server "cache-01" "cache"
echo ""

list_servers
echo ""

filter_by_role "web"
echo ""

remove_server "web-02"
echo ""

list_servers
```

注意：关联数组需要 Bash 4+。macOS 自带的 bash 是 3.x，需要 `brew install bash` 或改用 zsh。

</details>

---

## 四、Cron 与自动化

### 练习 9：自动化日志轮转

编写一套日志轮转脚本和对应的 cron 配置：

- 脚本 `rotate-logs.sh`：
  - 检查 `/var/log/myapp/` 下的日志文件
  - 超过 10MB 的文件，压缩并添加日期后缀
  - 删除 30 天前的压缩日志
  - 记录轮转操作日志
- 写出对应的 crontab 条目（每天凌晨 2 点执行）

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Log rotation script

LOG_DIR="/var/log/myapp"
MAX_SIZE=$((10 * 1024 * 1024))  # 10MB in bytes
RETENTION_DAYS=30
ROTATE_LOG="/var/log/myapp/rotation.log"
DATE=$(date +%Y%m%d)

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$ROTATE_LOG"
}

if [ ! -d "$LOG_DIR" ]; then
    echo "Error: $LOG_DIR does not exist"
    exit 1
fi

log "Starting log rotation"

find "$LOG_DIR" -name "*.log" ! -name "rotation.log" -type f | while read -r file; do
    size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
    if [ "$size" -gt "$MAX_SIZE" ]; then
        archive="${file}.${DATE}.gz"
        gzip -c "$file" > "$archive"
        : > "$file"  # Truncate original
        log "Rotated: $file -> $archive (was ${size} bytes)"
    fi
done

# Clean old archives
deleted=$(find "$LOG_DIR" -name "*.gz" -mtime +${RETENTION_DAYS} -delete -print | wc -l)
if [ "$deleted" -gt 0 ]; then
    log "Deleted $deleted archive(s) older than ${RETENTION_DAYS} days"
fi

log "Log rotation complete"
```

Crontab 条目：
```
# Run log rotation daily at 2:00 AM
0 2 * * * /opt/scripts/rotate-logs.sh >> /var/log/cron-rotate.log 2>&1
```

安装 crontab：
```bash
crontab -e
# Add the line above, save and exit
crontab -l  # verify
```

</details>

### 练习 10：系统监控告警脚本

编写 `monitor-alert.sh`，配合 cron 定期检查系统状态：

- 检查磁盘使用率（阈值 90%）
- 检查内存使用率（阈值 85%）
- 检查关键服务是否运行（sshd 等）
- 如果任何指标超标，将告警信息写入 `/tmp/alerts.log`
- 写出 crontab 条目（每 5 分钟执行一次）

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# System monitor with alerting

ALERT_LOG="/tmp/alerts.log"
DISK_THRESHOLD=90
MEM_THRESHOLD=85
SERVICES="sshd cron"
ALERT=0

timestamp() {
    date '+%Y-%m-%d %H:%M:%S'
}

alert() {
    echo "[$(timestamp)] ALERT: $1" >> "$ALERT_LOG"
    ALERT=1
}

# Check disk
while read -r usage mount; do
    pct=${usage%\%}
    if [ "$pct" -gt "$DISK_THRESHOLD" ] 2>/dev/null; then
        alert "Disk usage on $mount is ${usage} (threshold: ${DISK_THRESHOLD}%)"
    fi
done < <(df -h | awk 'NR>1 {print $5, $6}')

# Check memory (Linux)
if command -v free &>/dev/null; then
    mem_pct=$(free | awk '/Mem:/ {printf "%.0f", $3/$2*100}')
    if [ "$mem_pct" -gt "$MEM_THRESHOLD" ]; then
        alert "Memory usage is ${mem_pct}% (threshold: ${MEM_THRESHOLD}%)"
    fi
fi

# Check services
for svc in $SERVICES; do
    if ! pgrep -x "$svc" &>/dev/null; then
        alert "Service '$svc' is not running!"
    fi
done

if [ $ALERT -eq 0 ]; then
    echo "[$(timestamp)] OK: All checks passed" >> "$ALERT_LOG"
fi
```

Crontab 条目：
```
# System health check every 5 minutes
*/5 * * * * /opt/scripts/monitor-alert.sh
```

</details>

---

## 五、系统管理

### 练习 11：用户与权限管理自动化

编写脚本 `manage-users.sh`，从 CSV 文件批量创建用户：

输入文件格式：
```
username,group,shell
alice,developers,/bin/bash
bob,developers,/bin/bash
charlie,ops,/bin/zsh
diana,developers,/bin/bash
```

脚本需要：
- 读取 CSV 文件（跳过标题行）
- 如果用户组不存在，先创建
- 创建用户、设置组和 shell
- 生成随机初始密码
- 将用户名和密码输出到 `/tmp/credentials.txt`
- 干运行模式（`-d` 参数）：只显示将执行的命令，不实际执行

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Batch user creation from CSV

DRY_RUN=false

while getopts "d" opt; do
    case $opt in
        d) DRY_RUN=true ;;
        *) echo "Usage: $0 [-d] <users.csv>"; exit 1 ;;
    esac
done
shift $((OPTIND - 1))

CSV_FILE="${1:?Usage: $0 [-d] <users.csv>}"
CRED_FILE="/tmp/credentials.txt"

if [ ! -f "$CSV_FILE" ]; then
    echo "Error: file '$CSV_FILE' not found"
    exit 1
fi

run_cmd() {
    if $DRY_RUN; then
        echo "[DRY RUN] $*"
    else
        "$@"
    fi
}

generate_password() {
    # Generate 12-char random password
    tr -dc 'A-Za-z0-9!@#$%' < /dev/urandom | head -c 12
}

[ -f "$CRED_FILE" ] && : > "$CRED_FILE"

tail -n +2 "$CSV_FILE" | while IFS=',' read -r username group shell; do
    # Trim whitespace
    username=$(echo "$username" | xargs)
    group=$(echo "$group" | xargs)
    shell=$(echo "$shell" | xargs)

    echo "--- Processing user: $username ---"

    # Create group if needed
    if ! getent group "$group" &>/dev/null; then
        echo "Creating group: $group"
        run_cmd sudo groupadd "$group"
    fi

    # Create user
    if getent passwd "$username" &>/dev/null; then
        echo "User $username already exists, skipping."
        continue
    fi

    password=$(generate_password)
    echo "Creating user: $username (group: $group, shell: $shell)"
    run_cmd sudo useradd -m -g "$group" -s "$shell" "$username"

    if ! $DRY_RUN; then
        echo "$username:$password" | sudo chpasswd
        echo "$username,$password" >> "$CRED_FILE"
    else
        echo "[DRY RUN] Would set password for $username"
        echo "[DRY RUN] $username,<random_pass>" >> "$CRED_FILE"
    fi
done

echo ""
echo "Credentials saved to $CRED_FILE"
$DRY_RUN && echo "(Dry run mode - no actual changes made)"
```

测试（先用干运行模式）：
```bash
cat > /tmp/users.csv << 'EOF'
username,group,shell
alice,developers,/bin/bash
bob,developers,/bin/bash
charlie,ops,/bin/zsh
EOF

bash manage-users.sh -d /tmp/users.csv
```

</details>

### 练习 12：Systemd 服务管理

为一个 Node.js 应用编写完整的 systemd 服务配置和管理脚本：

1. 编写 systemd unit 文件 `myapp.service`
2. 编写管理脚本 `service-ctl.sh`，支持：`start`、`stop`、`restart`、`status`、`logs`
3. 脚本需要检查 unit 文件是否已安装，未安装则提示安装命令

<details>
<summary>查看答案</summary>

**/etc/systemd/system/myapp.service:**
```ini
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production
Environment=PORT=3000
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**service-ctl.sh:**
```bash
#!/bin/bash
# Service management wrapper

SERVICE="myapp"
UNIT_FILE="/etc/systemd/system/${SERVICE}.service"

if [ ! -f "$UNIT_FILE" ]; then
    echo "Error: $UNIT_FILE not found."
    echo "Install with:"
    echo "  sudo cp ${SERVICE}.service $UNIT_FILE"
    echo "  sudo systemctl daemon-reload"
    echo "  sudo systemctl enable ${SERVICE}"
    exit 1
fi

case "${1:-}" in
    start)
        echo "Starting $SERVICE..."
        sudo systemctl start "$SERVICE"
        sudo systemctl status "$SERVICE" --no-pager
        ;;
    stop)
        echo "Stopping $SERVICE..."
        sudo systemctl stop "$SERVICE"
        echo "$SERVICE stopped."
        ;;
    restart)
        echo "Restarting $SERVICE..."
        sudo systemctl restart "$SERVICE"
        sudo systemctl status "$SERVICE" --no-pager
        ;;
    status)
        sudo systemctl status "$SERVICE" --no-pager
        ;;
    logs)
        shift
        sudo journalctl -u "$SERVICE" --no-pager "${@:--n 50}"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status|logs}"
        echo ""
        echo "  logs [-f]     Show logs (-f for follow mode)"
        echo "  logs -n 100   Show last 100 log entries"
        exit 1
        ;;
esac
```

</details>

---

## 六、磁盘与存储管理

### 练习 13：磁盘空间清理

编写 `disk-cleanup.sh`，智能清理磁盘空间：

- 显示当前磁盘使用情况
- 查找并列出大于 100MB 的文件（Top 20）
- 查找 30 天未访问的文件（仅列出，不删除）
- 查找并统计各目录的大小（`/var/log`、`/tmp`、`/home`）
- 清理 `/tmp` 下超过 7 天的文件（需要用户确认）
- 清理已删除但被进程占用的文件（提供 `lsof` 检查方法）

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Intelligent disk cleanup script

echo "===== Disk Usage Overview ====="
df -h | grep -v tmpfs
echo ""

echo "===== Top 20 Largest Files ====="
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | \
    awk '{print $5, $NF}' | sort -rh | head -20
echo ""

echo "===== Files Not Accessed in 30 Days (under /tmp) ====="
find /tmp -type f -atime +30 2>/dev/null | head -20
echo ""

echo "===== Directory Sizes ====="
for dir in /var/log /tmp /home; do
    if [ -d "$dir" ]; then
        size=$(du -sh "$dir" 2>/dev/null | awk '{print $1}')
        echo "  $dir: $size"
    fi
done
echo ""

echo "===== Old Files in /tmp (>7 days) ====="
old_files=$(find /tmp -type f -mtime +7 2>/dev/null)
count=$(echo "$old_files" | grep -c '^' 2>/dev/null)
echo "Found $count file(s) older than 7 days in /tmp"

if [ "$count" -gt 0 ]; then
    echo "$old_files" | head -10
    echo ""
    read -p "Delete these files? [y/N]: " confirm
    if [ "$confirm" = "y" ]; then
        find /tmp -type f -mtime +7 -delete 2>/dev/null
        echo "Cleaned."
    else
        echo "Skipped."
    fi
fi
echo ""

echo "===== Deleted but Open Files ====="
echo "Run: sudo lsof +L1"
echo "These are files deleted from disk but still held open by processes."
echo "Restarting those processes will free the space."
```

</details>

### 练习 14：存储使用趋势报告

编写脚本 `storage-trend.sh`，记录并报告存储使用趋势：

- 每次运行时，将当前磁盘使用率追加到 CSV 文件
- 使用 awk 分析 CSV 数据，计算：最近 7 天的平均使用率、使用率变化趋势
- 如果使用率连续上升超过 3 次，发出告警

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Storage usage trend tracker

DATA_FILE="/tmp/storage-trend.csv"

# Initialize CSV if not exists
if [ ! -f "$DATA_FILE" ]; then
    echo "timestamp,mount,usage_pct" > "$DATA_FILE"
fi

# Record current usage
while read -r usage mount; do
    pct=${usage%\%}
    echo "$(date '+%Y-%m-%d %H:%M:%S'),$mount,$pct" >> "$DATA_FILE"
done < <(df -h | awk 'NR>1 && $5 ~ /%/ {print $5, $6}')

echo "Current usage recorded."
echo ""

# Analyze root partition trend
echo "===== Trend Analysis (root partition) ====="
ROOT_DATA=$(grep ',/$' "$DATA_FILE" | tail -7)

if [ "$(echo "$ROOT_DATA" | wc -l)" -lt 2 ]; then
    echo "Not enough data points. Run this script over multiple days."
    exit 0
fi

echo "$ROOT_DATA" | awk -F',' '
{
    readings[NR] = $3
    count = NR
}
END {
    # Average
    sum = 0
    for (i=1; i<=count; i++) sum += readings[i]
    avg = sum / count
    printf "  Average usage: %.1f%%\n", avg
    printf "  Data points:   %d\n", count

    # Trend detection
    increasing = 0
    for (i=2; i<=count; i++) {
        if (readings[i] > readings[i-1]) {
            increasing++
        } else {
            increasing = 0
        }
    }
    printf "  Consecutive increases: %d\n", increasing

    if (increasing >= 3) {
        print ""
        print "  *** WARNING: Usage has been increasing for " increasing " consecutive readings! ***"
    }
}'
```

配合 cron 每日运行：
```
# Record storage trend daily at 8:00 AM
0 8 * * * /opt/scripts/storage-trend.sh >> /var/log/storage-trend.log 2>&1
```

</details>

---

## 七、性能调优与监控

### 练习 15：性能基线采集

编写 `perf-baseline.sh`，采集系统性能基线数据：

- 采集 CPU 使用率（5 秒间隔，3 次采样）
- 采集内存使用情况
- 采集磁盘 I/O 统计
- 采集网络连接统计
- 所有数据保存为结构化报告

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Performance baseline data collector

REPORT="/tmp/perf-baseline-$(date +%Y%m%d-%H%M%S).txt"

{
    echo "===== Performance Baseline Report ====="
    echo "Date: $(date)"
    echo "Hostname: $(hostname)"
    echo "Kernel: $(uname -r)"
    echo ""

    echo "===== CPU Usage (3 samples, 5s interval) ====="
    if command -v vmstat &>/dev/null; then
        vmstat 5 4
    elif command -v top &>/dev/null; then
        top -bn3 | grep '%Cpu' | tail -3
    fi
    echo ""

    echo "===== Memory ====="
    if command -v free &>/dev/null; then
        free -h
    else
        vm_stat 2>/dev/null
    fi
    echo ""

    echo "===== Disk I/O ====="
    if command -v iostat &>/dev/null; then
        iostat -x 1 2
    else
        echo "(iostat not available, showing disk usage)"
        df -h
    fi
    echo ""

    echo "===== Network Connections ====="
    if command -v ss &>/dev/null; then
        echo "Connection states:"
        ss -s
        echo ""
        echo "Listening ports:"
        ss -tlnp 2>/dev/null || ss -tln
    else
        netstat -an | awk '/^tcp/ {print $6}' | sort | uniq -c | sort -rn
    fi
    echo ""

    echo "===== Top 10 Processes by CPU ====="
    ps aux --sort=-%cpu 2>/dev/null | head -11 || ps aux | head -11
    echo ""

    echo "===== Top 10 Processes by Memory ====="
    ps aux --sort=-%mem 2>/dev/null | head -11 || ps aux | head -11
    echo ""

    echo "===== Load Average ====="
    uptime

} > "$REPORT" 2>&1

echo "Report saved to: $REPORT"
echo "Size: $(du -h "$REPORT" | awk '{print $1}')"
```

</details>

### 练习 16：进程 CPU 使用率追踪器

编写 `cpu-tracker.sh`，持续追踪指定进程的 CPU 使用率：

- 接受进程名或 PID 作为参数
- 每秒采样一次，记录到 CSV 文件
- 支持 `Ctrl+C` 优雅退出，并输出统计摘要（平均值、最大值、最小值）
- 使用 `trap` 处理信号

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# Track CPU usage of a specific process over time

if [ $# -lt 1 ]; then
    echo "Usage: $0 <process_name_or_PID> [interval_seconds]"
    exit 1
fi

TARGET="$1"
INTERVAL="${2:-1}"
CSV="/tmp/cpu-track-${TARGET}-$(date +%H%M%S).csv"
SAMPLES=0
TOTAL_CPU=0
MAX_CPU=0
MIN_CPU=999

echo "timestamp,pid,cpu_pct" > "$CSV"

show_summary() {
    echo ""
    echo "===== Tracking Summary ====="
    if [ $SAMPLES -gt 0 ]; then
        AVG=$(awk "BEGIN {printf \"%.1f\", $TOTAL_CPU / $SAMPLES}")
        echo "  Samples:  $SAMPLES"
        echo "  Average:  ${AVG}%"
        echo "  Maximum:  ${MAX_CPU}%"
        echo "  Minimum:  ${MIN_CPU}%"
    else
        echo "  No samples collected."
    fi
    echo "  Data file: $CSV"
    exit 0
}

trap show_summary INT TERM

echo "Tracking '$TARGET' every ${INTERVAL}s (Ctrl+C to stop)..."
echo ""

while true; do
    if [[ "$TARGET" =~ ^[0-9]+$ ]]; then
        line=$(ps -p "$TARGET" -o pid=,pcpu= 2>/dev/null)
    else
        line=$(ps -C "$TARGET" -o pid=,pcpu= 2>/dev/null | head -1)
    fi

    if [ -z "$line" ]; then
        echo "Process '$TARGET' not found. Waiting..."
        sleep "$INTERVAL"
        continue
    fi

    pid=$(echo "$line" | awk '{print $1}')
    cpu=$(echo "$line" | awk '{print $2}')

    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp,$pid,$cpu" >> "$CSV"
    printf "%s  PID: %-8s  CPU: %s%%\n" "$timestamp" "$pid" "$cpu"

    # Update stats (integer comparison via awk)
    SAMPLES=$((SAMPLES + 1))
    TOTAL_CPU=$(awk "BEGIN {print $TOTAL_CPU + $cpu}")
    MAX_CPU=$(awk "BEGIN {print ($cpu > $MAX_CPU) ? $cpu : $MAX_CPU}")
    MIN_CPU=$(awk "BEGIN {print ($cpu < $MIN_CPU) ? $cpu : $MIN_CPU}")

    sleep "$INTERVAL"
done
```

测试：
```bash
# In one terminal, start a process
while true; do :; done &
PID=$!

# In another terminal, track it
bash cpu-tracker.sh $PID 2

# Press Ctrl+C to see summary
kill $PID
```

</details>

### 练习 17：综合场景 —— 生产事故排查

场景：周一早上你收到告警，Web 服务器响应变慢。请写出你的排查步骤（每一步给出命令）：

1. 快速查看系统整体状态（load average、uptime）
2. 检查 CPU 和内存的使用情况
3. 找出占用 CPU 最多的进程
4. 检查磁盘空间是否充足
5. 检查磁盘 I/O 是否有瓶颈
6. 查看网络连接数，是否有异常大量连接
7. 检查应用日志中的错误
8. 检查系统日志中是否有硬件错误
9. 如果发现是某个进程占用资源过多，如何处理
10. 写一个一键采集以上所有信息的脚本

<details>
<summary>查看答案</summary>

```bash
# 1. System overview
uptime
w

# 2. CPU and memory
top -bn1 | head -15
free -h
vmstat 1 5

# 3. Top CPU consumers
ps aux --sort=-%cpu | head -10

# 4. Disk space
df -h

# 5. Disk I/O
iostat -x 1 3
# Check I/O wait in vmstat output (%wa column)

# 6. Network connections
ss -s
ss -tn state established | wc -l
ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10

# 7. Application logs
tail -100 /var/log/myapp/error.log
grep -c "ERROR" /var/log/myapp/error.log
grep "ERROR" /var/log/myapp/error.log | tail -20

# 8. System/hardware errors
dmesg | tail -30
journalctl -p err --since "1 hour ago"

# 9. Handle rogue process
# Reduce priority:
renice +10 -p <PID>
# Or limit CPU:
cpulimit -p <PID> -l 50
# Last resort - kill:
kill <PID>
kill -9 <PID>
```

**一键采集脚本 `incident-report.sh`：**

```bash
#!/bin/bash
# Incident data collector for quick triage

REPORT="/tmp/incident-$(date +%Y%m%d-%H%M%S).txt"

collect() {
    echo "===== $1 =====" >> "$REPORT"
    eval "$2" >> "$REPORT" 2>&1
    echo "" >> "$REPORT"
}

echo "Collecting system data..."

collect "Timestamp" "date"
collect "Uptime & Load" "uptime"
collect "Memory" "free -h"
collect "CPU - Top Processes" "ps aux --sort=-%cpu | head -15"
collect "Memory - Top Processes" "ps aux --sort=-%mem | head -15"
collect "Disk Space" "df -h"
collect "Disk I/O" "iostat -x 1 2 2>/dev/null || echo 'iostat not available'"
collect "Network Connections Summary" "ss -s 2>/dev/null || netstat -s"
collect "Top Connection Sources" "ss -tn 2>/dev/null | awk 'NR>1{print \$5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10"
collect "Recent System Errors" "dmesg 2>/dev/null | tail -20"
collect "Recent Journal Errors" "journalctl -p err --since '1 hour ago' --no-pager 2>/dev/null || echo 'journalctl not available'"

echo "Report saved to: $REPORT"
echo "Size: $(du -h "$REPORT" | awk '{print $1}')"
```

</details>

---

**完成所有练习了？太棒了！** 🎉

你已经完成了整个 xcmds 教程的所有综合练习。建议回顾你觉得困难的部分，并在实际工作中不断练习。

📖 返回[课程目录](../README.md)
