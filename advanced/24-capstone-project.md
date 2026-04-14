# 第 24 课：高级综合实战项目

> 上一课：[性能调优与监控](23-performance.md) | 下一课：[返回目录](../README.md)

## 前言

恭喜你来到高级篇的最终课！本课不是传统的讲解课，而是一个 **综合实战项目**。你将运用第 17-23 课学到的所有知识，从零构建一个完整的服务器监控与告警脚本。

项目完成后，还有一份覆盖整个高级篇的综合测验，检验你的学习成果。

---

## 项目：构建服务器监控与告警系统

### 需求规格

构建一个 Bash 脚本，实现以下功能：

1. **采集系统信息**：CPU 使用率、内存使用率、磁盘使用率、系统负载
2. **日志解析与分析**：分析系统日志中的错误和告警
3. **阈值告警**：超过设定阈值时触发告警
4. **告警输出**：将告警写入文件，可选发送邮件
5. **自动调度**：通过 Cron 定期执行
6. **日志轮转**：管理自身产生的日志

---

### 第一步：项目结构

创建项目目录：

```bash
mkdir -p ~/server-monitor/{config,logs,reports}
touch ~/server-monitor/monitor.sh
touch ~/server-monitor/config/thresholds.conf
touch ~/server-monitor/config/alert.conf
chmod +x ~/server-monitor/monitor.sh
```

目录结构：

```
~/server-monitor/
├── monitor.sh              # Main script
├── config/
│   ├── thresholds.conf     # Alert thresholds
│   └── alert.conf          # Alert output configuration
├── logs/
│   └── monitor.log         # Runtime logs
└── reports/
    └── report_YYYYMMDD.txt # Daily reports
```

### 第二步：配置文件

#### thresholds.conf

```bash
# Alert thresholds (percentage)
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90
LOAD_THRESHOLD_MULTIPLIER=2    # Alert when load > (cores * multiplier)
SWAP_THRESHOLD=50
```

#### alert.conf

```bash
# Alert configuration
ALERT_FILE="/home/user/server-monitor/logs/alerts.log"
REPORT_DIR="/home/user/server-monitor/reports"
LOG_FILE="/home/user/server-monitor/logs/monitor.log"

# Email settings (optional, requires mailutils)
ENABLE_EMAIL=false
EMAIL_TO="admin@example.com"
EMAIL_SUBJECT_PREFIX="[Server Alert]"

# Log retention (days)
LOG_RETENTION=30
REPORT_RETENTION=90
```

### 第三步：系统信息采集模块

```bash
#!/bin/bash
# monitor.sh - Server monitoring and alert system
set -euo pipefail

readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

# Load configuration
source "${SCRIPT_DIR}/config/thresholds.conf"
source "${SCRIPT_DIR}/config/alert.conf"

readonly TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
readonly DATE_TAG=$(date '+%Y%m%d')

# ============================================================
# Logging
# ============================================================
log() {
    local level="$1"
    shift
    echo "${TIMESTAMP} [${level}] $*" >> "$LOG_FILE"
}

# ============================================================
# System info collection
# ============================================================
get_cpu_usage() {
    # Sample CPU over 1 second for accuracy
    local idle
    idle=$(mpstat 1 1 2>/dev/null | awk '/Average/ {print $NF}')
    if [[ -z "$idle" ]]; then
        idle=$(top -bn2 -d1 | grep "Cpu(s)" | tail -1 | awk '{print $8}')
    fi
    echo "scale=1; 100 - ${idle:-0}" | bc
}

get_memory_usage() {
    free | awk '/^Mem:/ {printf "%.1f", $3/$2 * 100}'
}

get_swap_usage() {
    local total used
    total=$(free | awk '/^Swap:/ {print $2}')
    used=$(free | awk '/^Swap:/ {print $3}')
    if [[ "$total" -eq 0 ]]; then
        echo "0.0"
    else
        echo "scale=1; ${used} * 100 / ${total}" | bc
    fi
}

get_disk_usage() {
    # Returns the highest usage percentage among all mounted partitions
    df -h --output=pcent,target -x tmpfs -x devtmpfs 2>/dev/null \
        | tail -n +2 \
        | sed 's/%//' \
        | sort -rn \
        | head -1 \
        | awk '{print $1}'
}

get_disk_details() {
    df -h --output=source,size,used,avail,pcent,target -x tmpfs -x devtmpfs 2>/dev/null
}

get_load_average() {
    awk '{print $1}' /proc/loadavg
}

get_cpu_cores() {
    nproc
}

get_uptime() {
    uptime -p 2>/dev/null || uptime
}

get_top_processes() {
    local sort_by="$1"
    local count="${2:-5}"
    ps aux --sort="-${sort_by}" | head -$((count + 1))
}
```

### 第四步：日志解析模块

```bash
# ============================================================
# Log analysis
# ============================================================
analyze_system_logs() {
    local since="${1:-1 hour ago}"
    local result=""

    # Count errors and warnings in journal
    local error_count=0
    local warn_count=0

    if command -v journalctl &>/dev/null; then
        error_count=$(journalctl --since "$since" -p err --no-pager -q 2>/dev/null | wc -l)
        warn_count=$(journalctl --since "$since" -p warning --no-pager -q 2>/dev/null | wc -l)
    fi

    echo "errors=${error_count},warnings=${warn_count}"
}

check_failed_services() {
    if command -v systemctl &>/dev/null; then
        systemctl --failed --no-pager --no-legend 2>/dev/null
    fi
}

check_oom_events() {
    # Check for OOM killer events in the last hour
    if command -v journalctl &>/dev/null; then
        journalctl -k --since "1 hour ago" --no-pager -q 2>/dev/null \
            | grep -i "out of memory" || true
    fi
}
```

### 第五步：告警模块

```bash
# ============================================================
# Alert handling
# ============================================================
ALERTS=()

add_alert() {
    local severity="$1"
    local message="$2"
    ALERTS+=("[${severity}] ${message}")
    log "$severity" "$message"
}

check_thresholds() {
    local cpu_usage mem_usage disk_usage load_avg swap_usage cores max_load

    cpu_usage=$(get_cpu_usage)
    mem_usage=$(get_memory_usage)
    disk_usage=$(get_disk_usage)
    load_avg=$(get_load_average)
    swap_usage=$(get_swap_usage)
    cores=$(get_cpu_cores)
    max_load=$(echo "${cores} * ${LOAD_THRESHOLD_MULTIPLIER}" | bc)

    log "INFO" "CPU=${cpu_usage}% MEM=${mem_usage}% DISK=${disk_usage}% LOAD=${load_avg} SWAP=${swap_usage}%"

    # CPU check
    if (( $(echo "$cpu_usage > $CPU_THRESHOLD" | bc -l) )); then
        add_alert "CRITICAL" "CPU usage at ${cpu_usage}% (threshold: ${CPU_THRESHOLD}%)"
    fi

    # Memory check
    if (( $(echo "$mem_usage > $MEMORY_THRESHOLD" | bc -l) )); then
        add_alert "CRITICAL" "Memory usage at ${mem_usage}% (threshold: ${MEMORY_THRESHOLD}%)"
    fi

    # Disk check
    if (( $(echo "$disk_usage > $DISK_THRESHOLD" | bc -l) )); then
        add_alert "CRITICAL" "Disk usage at ${disk_usage}% (threshold: ${DISK_THRESHOLD}%)"
    fi

    # Load check
    if (( $(echo "$load_avg > $max_load" | bc -l) )); then
        add_alert "WARNING" "Load average ${load_avg} exceeds ${max_load} (${cores} cores x ${LOAD_THRESHOLD_MULTIPLIER})"
    fi

    # Swap check
    if (( $(echo "$swap_usage > $SWAP_THRESHOLD" | bc -l) )); then
        add_alert "WARNING" "Swap usage at ${swap_usage}% (threshold: ${SWAP_THRESHOLD}%)"
    fi

    # Failed services
    local failed_svcs
    failed_svcs=$(check_failed_services)
    if [[ -n "$failed_svcs" ]]; then
        add_alert "WARNING" "Failed services detected: $(echo "$failed_svcs" | awk '{print $2}' | tr '\n' ' ')"
    fi

    # OOM events
    local oom_events
    oom_events=$(check_oom_events)
    if [[ -n "$oom_events" ]]; then
        add_alert "CRITICAL" "OOM killer events detected in the last hour"
    fi

    # Log analysis
    local log_stats
    log_stats=$(analyze_system_logs "1 hour ago")
    local err_count warn_count
    err_count=$(echo "$log_stats" | cut -d',' -f1 | cut -d'=' -f2)
    warn_count=$(echo "$log_stats" | cut -d',' -f2 | cut -d'=' -f2)
    if (( err_count > 50 )); then
        add_alert "WARNING" "High error rate in system logs: ${err_count} errors in last hour"
    fi
}
```

### 第六步：输出与告警发送

```bash
# ============================================================
# Alert output
# ============================================================
send_alerts() {
    if [[ ${#ALERTS[@]} -eq 0 ]]; then
        log "INFO" "All checks passed, no alerts"
        return 0
    fi

    local alert_msg=""
    alert_msg+="Server Alert Report - ${TIMESTAMP}\n"
    alert_msg+="Host: $(hostname)\n"
    alert_msg+="================================\n\n"

    for alert in "${ALERTS[@]}"; do
        alert_msg+="${alert}\n"
    done

    alert_msg+="\n--- Top CPU Processes ---\n"
    alert_msg+="$(get_top_processes %cpu 5)\n"
    alert_msg+="\n--- Top Memory Processes ---\n"
    alert_msg+="$(get_top_processes %mem 5)\n"
    alert_msg+="\n--- Disk Usage ---\n"
    alert_msg+="$(get_disk_details)\n"

    # Write to alert file
    echo -e "$alert_msg" >> "$ALERT_FILE"

    # Send email if enabled
    if [[ "$ENABLE_EMAIL" == "true" ]] && command -v mail &>/dev/null; then
        echo -e "$alert_msg" | mail -s "${EMAIL_SUBJECT_PREFIX} ${#ALERTS[@]} alert(s) on $(hostname)" "$EMAIL_TO"
        log "INFO" "Alert email sent to ${EMAIL_TO}"
    fi

    log "INFO" "${#ALERTS[@]} alert(s) generated"
}

generate_daily_report() {
    local report_file="${REPORT_DIR}/report_${DATE_TAG}.txt"

    {
        echo "======================================"
        echo "  Daily System Report"
        echo "  Host: $(hostname)"
        echo "  Date: ${TIMESTAMP}"
        echo "  Uptime: $(get_uptime)"
        echo "======================================"
        echo ""
        echo "--- System Metrics ---"
        echo "CPU Usage:    $(get_cpu_usage)%"
        echo "Memory Usage: $(get_memory_usage)%"
        echo "Swap Usage:   $(get_swap_usage)%"
        echo "Disk Usage:   $(get_disk_usage)%"
        echo "Load Average: $(cat /proc/loadavg)"
        echo "CPU Cores:    $(get_cpu_cores)"
        echo ""
        echo "--- Memory Details ---"
        free -h
        echo ""
        echo "--- Disk Details ---"
        get_disk_details
        echo ""
        echo "--- Top 10 CPU Processes ---"
        get_top_processes %cpu 10
        echo ""
        echo "--- Top 10 Memory Processes ---"
        get_top_processes %mem 10
        echo ""
        echo "--- Failed Services ---"
        local failed
        failed=$(check_failed_services)
        echo "${failed:-None}"
        echo ""
        echo "--- Log Summary (last 24h) ---"
        local stats
        stats=$(analyze_system_logs "24 hours ago")
        echo "Errors:   $(echo "$stats" | cut -d',' -f1 | cut -d'=' -f2)"
        echo "Warnings: $(echo "$stats" | cut -d',' -f2 | cut -d'=' -f2)"
    } > "$report_file"

    log "INFO" "Daily report generated: ${report_file}"
}
```

### 第七步：日志轮转模块

```bash
# ============================================================
# Log rotation
# ============================================================
rotate_logs() {
    # Rotate monitor log if > 10MB
    if [[ -f "$LOG_FILE" ]]; then
        local size
        size=$(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE" 2>/dev/null || echo 0)
        if (( size > 10485760 )); then
            mv "$LOG_FILE" "${LOG_FILE}.$(date +%Y%m%d%H%M%S)"
            gzip "${LOG_FILE}."* 2>/dev/null || true
            log "INFO" "Log rotated"
        fi
    fi

    # Clean old logs
    find "$(dirname "$LOG_FILE")" -name "*.gz" -mtime +"$LOG_RETENTION" -delete 2>/dev/null || true

    # Clean old reports
    find "$REPORT_DIR" -name "report_*.txt" -mtime +"$REPORT_RETENTION" -delete 2>/dev/null || true

    # Clean old alerts
    if [[ -f "$ALERT_FILE" ]]; then
        local asize
        asize=$(stat -f%z "$ALERT_FILE" 2>/dev/null || stat -c%s "$ALERT_FILE" 2>/dev/null || echo 0)
        if (( asize > 52428800 )); then
            mv "$ALERT_FILE" "${ALERT_FILE}.old"
            gzip "${ALERT_FILE}.old"
        fi
    fi
}
```

### 第八步：主函数与入口

```bash
# ============================================================
# Main entry point
# ============================================================
usage() {
    cat <<EOF
Usage: $(basename "$0") [OPTIONS]

Options:
    -c, --check       Run health check and send alerts (default)
    -r, --report      Generate daily report
    -s, --status      Show current system status
    -h, --help        Show this help message
EOF
}

cmd_status() {
    echo "=== System Status ==="
    echo "CPU:    $(get_cpu_usage)%"
    echo "Memory: $(get_memory_usage)%"
    echo "Swap:   $(get_swap_usage)%"
    echo "Disk:   $(get_disk_usage)%"
    echo "Load:   $(get_load_average) ($(get_cpu_cores) cores)"
    echo "Uptime: $(get_uptime)"
}

cmd_check() {
    log "INFO" "=== Health check started ==="
    rotate_logs
    check_thresholds
    send_alerts
    log "INFO" "=== Health check completed ==="
}

cmd_report() {
    log "INFO" "=== Report generation started ==="
    generate_daily_report
    log "INFO" "=== Report generation completed ==="
}

# Parse arguments
ACTION="check"
while [[ $# -gt 0 ]]; do
    case "$1" in
        -c|--check)   ACTION="check";  shift ;;
        -r|--report)  ACTION="report"; shift ;;
        -s|--status)  ACTION="status"; shift ;;
        -h|--help)    usage; exit 0 ;;
        *)            echo "Unknown option: $1"; usage; exit 1 ;;
    esac
done

# Ensure directories exist
mkdir -p "$(dirname "$LOG_FILE")" "$REPORT_DIR"

case "$ACTION" in
    check)  cmd_check  ;;
    report) cmd_report ;;
    status) cmd_status ;;
esac
```

### 第九步：Cron 集成

```bash
# Health check every 5 minutes
*/5 * * * * /home/user/server-monitor/monitor.sh --check

# Daily report at 7:00 AM
0 7 * * * /home/user/server-monitor/monitor.sh --report
```

安装到 crontab：

```bash
(crontab -l 2>/dev/null; cat <<'CRON'
# Server monitoring
*/5 * * * * /home/user/server-monitor/monitor.sh --check >> /dev/null 2>&1
0 7 * * * /home/user/server-monitor/monitor.sh --report >> /dev/null 2>&1
CRON
) | crontab -

# Verify
crontab -l
```

### 第十步：测试

```bash
# Test status output
~/server-monitor/monitor.sh --status

# Test health check
~/server-monitor/monitor.sh --check
cat ~/server-monitor/logs/monitor.log

# Test report generation
~/server-monitor/monitor.sh --report
cat ~/server-monitor/reports/report_$(date +%Y%m%d).txt

# Simulate high threshold to test alerts
# Temporarily set CPU_THRESHOLD=1 in thresholds.conf
# Then run --check and verify alerts.log
```

---

## 扩展挑战

完成基本项目后，尝试以下进阶任务：

1. **添加网络监控**：监控网络接口流量、丢包率和连接数（使用 `ss`、`sar -n DEV`）
2. **添加进程监控**：检查关键进程是否在运行，自动重启崩溃的服务
3. **HTML 报告**：将日报生成为 HTML 格式，可以用浏览器查看
4. **告警去重**：相同告警在冷却时间内不重复发送（使用状态文件）
5. **Telegram/Webhook 通知**：用 `curl` 发送告警到 Telegram Bot 或 Slack Webhook
6. **仪表盘**：用 `whiptail` 或 `dialog` 创建终端 UI 的实时仪表盘
7. **用 systemd Timer 替换 Cron**：创建 `.service` 和 `.timer` 文件

---

## 高级篇综合测验

以下测验覆盖第 17-23 课的所有内容。

**1. 下面的脚本有什么问题？如何修复？**

```bash
#!/bin/bash
files=$(ls /tmp/*.log)
for f in $files; do
    rm $f
done
```

<details>
<summary>查看答案</summary>

**问题：**
1. 解析 `ls` 输出——文件名含空格或特殊字符会出错
2. 变量 `$f` 没有加引号——空格会导致 word splitting
3. 如果没有匹配文件，`ls` 会报错

**修复：**

```bash
#!/bin/bash
for f in /tmp/*.log; do
    [[ -f "$f" ]] || continue
    rm "$f"
done
```

或更简洁：`find /tmp -maxdepth 1 -name "*.log" -delete`

</details>

**2. 解释 `set -euo pipefail` 的每个选项。**

<details>
<summary>查看答案</summary>

- `set -e`（errexit）：任何命令返回非零状态时立即退出脚本
- `set -u`（nounset）：引用未定义的变量时报错并退出（防止拼写错误）
- `set -o pipefail`：管道中任何命令失败则整个管道返回非零（默认只看最后一个命令的状态）

组合使用可以尽早发现错误，是编写健壮脚本的最佳实践。

</details>

**3. 写一个函数，接受目录路径参数，返回该目录下最大的 5 个文件（名称和大小）。**

<details>
<summary>查看答案</summary>

```bash
top_files() {
    local dir="${1:-.}"

    if [[ ! -d "$dir" ]]; then
        echo "Error: $dir is not a directory" >&2
        return 1
    fi

    find "$dir" -maxdepth 1 -type f -printf '%s\t%p\n' \
        | sort -rn \
        | head -5 \
        | while IFS=$'\t' read -r size file; do
            printf "%10s  %s\n" "$(numfmt --to=iec "$size")" "$file"
        done
}
```

</details>

**4. Cron 任务 `0 2 * * * /opt/backup.sh` 没有执行。列出排查步骤。**

<details>
<summary>查看答案</summary>

```bash
# 1. Verify crontab is loaded
crontab -l | grep backup

# 2. Check cron service is running
systemctl status cron    # or crond on RHEL

# 3. Check script permissions and path
ls -la /opt/backup.sh
file /opt/backup.sh      # Verify it's a script, not binary

# 4. Test script manually
/opt/backup.sh

# 5. Check cron logs
grep CRON /var/log/syslog | tail -20
journalctl -u cron --since "02:00"

# 6. Check for PATH issues in script
head -5 /opt/backup.sh   # Does it use absolute paths?

# 7. Check if cron has MAILTO and there are error emails
# 8. Add logging: 0 2 * * * /opt/backup.sh >> /tmp/backup.log 2>&1
```

</details>

**5. 解释 `local` 和全局变量的区别。为什么函数中应该使用 `local`？**

<details>
<summary>查看答案</summary>

- **全局变量**：在整个脚本中可见，函数内修改会影响函数外的值
- **`local` 变量**：只在函数内部可见，函数返回后自动销毁

```bash
name="global"

change_name() {
    name="modified"       # Modifies the global variable!
}

safe_change() {
    local name="local"    # Independent, does not affect global
}
```

**为什么要用 `local`：**
1. 避免意外修改全局状态（尤其是循环变量 `i`、临时变量 `result`）
2. 防止函数之间的变量冲突
3. 使函数成为独立单元，便于测试和复用

</details>

**6. `useradd` 和 `adduser` 有什么区别？**

<details>
<summary>查看答案</summary>

- **`useradd`**：低级命令，所有发行版都有。需要手动指定选项（`-m` 创建家目录、`-s` 指定 Shell），否则可能不创建家目录。
- **`adduser`**：
  - 在 **Debian/Ubuntu** 上是高级交互式脚本，会自动创建家目录、设置密码、复制 skel 文件
  - 在 **RHEL/CentOS** 上只是 `useradd` 的符号链接

最佳实践：在脚本中使用 `useradd`（行为可预测）；交互式创建用户时在 Debian 系可以使用 `adduser`。

</details>

**7. 一个服务器的 `iostat` 显示 `%util` 持续在 95% 以上，`await` 超过 50ms。分析原因和解决方案。**

<details>
<summary>查看答案</summary>

**分析：**
- `%util` 95%：磁盘几乎 100% 繁忙，I/O 已成为瓶颈
- `await` 50ms：平均 I/O 等待时间很高（SSD 正常应 < 5ms，HDD < 20ms）

**排查：**

```bash
# Find I/O heavy processes
sudo iotop -o

# Check for swap activity (causes heavy I/O)
vmstat 1 5    # Look at si/so columns

# Check if it's read or write
iostat -xz 1 5   # Compare r/s vs w/s, rkB/s vs wkB/s
```

**解决方案：**
1. **找出并优化 I/O 密集进程**（数据库查询优化、减少日志写入）
2. **增加内存**（减少 swap，让更多数据缓存在内存中）
3. **升级到 SSD**（如果还在用 HDD）
4. **使用 I/O 调度器优化**：`echo deadline > /sys/block/sda/queue/scheduler`
5. **分散 I/O**：将日志和数据放到不同磁盘
6. **使用 `ionice` 降低非关键任务的 I/O 优先级**

</details>

**8. 写一条命令：找出 `/var/log` 下所有超过 7 天、大于 10MB 的 `.log` 文件，压缩它们。**

<details>
<summary>查看答案</summary>

```bash
find /var/log -name "*.log" -mtime +7 -size +10M -exec gzip {} \;
```

更安全的版本（先预览）：

```bash
# Preview
find /var/log -name "*.log" -mtime +7 -size +10M -ls

# Execute
find /var/log -name "*.log" -mtime +7 -size +10M -exec gzip -v {} \;
```

</details>

**9. 解释 LVM 中如何在不停机的情况下扩展一个正在使用的 ext4 文件系统。**

<details>
<summary>查看答案</summary>

```bash
# 1. Check current LV and VG status
sudo lvs
sudo vgs

# 2. If VG has free space, extend LV
sudo lvextend -L +20G /dev/vg_name/lv_name

# 3. Resize ext4 filesystem online
sudo resize2fs /dev/vg_name/lv_name

# Or combine steps 2-3:
sudo lvextend -L +20G --resizefs /dev/vg_name/lv_name

# 4. Verify
df -h /mount/point
```

关键点：ext4 支持在线扩展（无需卸载），`lvextend` + `resize2fs` 都可以在挂载状态下执行。

如果 VG 空间不足，需先添加新物理卷：

```bash
sudo pvcreate /dev/sdX
sudo vgextend vg_name /dev/sdX
```

</details>

**10. `free -h` 显示 available 内存只有 200MB，swap 使用了 1.5GB。写出排查和处理步骤。**

<details>
<summary>查看答案</summary>

```bash
# 1. Identify memory-hungry processes
ps aux --sort=-%mem | head -10

# 2. Check for memory leaks (RSS growing over time)
while true; do ps aux --sort=-%mem | head -5; sleep 60; done

# 3. Check OOM killer history
journalctl -k | grep -i oom

# 4. Detailed memory breakdown
cat /proc/meminfo

# 5. Per-process memory map (for suspect process)
sudo pmap -x <PID> | tail -5

# 6. Check for large tmpfs
df -h | grep tmpfs
```

**处理方案：**
1. **短期**：终止或重启内存泄漏的进程
2. **中期**：增加 swap 空间作为缓冲 `fallocate -l 4G /swapfile2`
3. **长期**：
   - 增加物理内存
   - 优化应用内存使用
   - 用 systemd 的 `MemoryMax=` 限制进程内存
   - 降低 `swappiness` 值（如 10）优先使用物理内存

</details>

**11. 写一个 `systemd` service 文件，用于运行一个 Node.js 应用，要求：以 `appuser` 用户运行、失败后自动重启、依赖网络就绪。**

<details>
<summary>查看答案</summary>

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```

启用：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemctl status myapp
journalctl -u myapp -f
```

</details>

**12. 以下 `strace` 输出说明了什么？**

```
openat(AT_FDCWD, "/etc/myapp/config.yaml", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/home/appuser/.myapp/config.yaml", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "./config.yaml", O_RDONLY) = 3
```

<details>
<summary>查看答案</summary>

这显示程序按顺序查找配置文件：

1. 先找 `/etc/myapp/config.yaml` → 不存在（`ENOENT`）
2. 再找 `/home/appuser/.myapp/config.yaml` → 不存在
3. 最后找 `./config.yaml`（当前目录下）→ 成功打开（返回文件描述符 3）

**启示：**
- 程序有 3 个配置文件搜索路径，优先级从高到低
- 当前使用的是当前目录下的 `config.yaml`
- 如果在 Cron 中运行，当前目录会变化，可能找不到配置文件
- 生产环境应将配置放在 `/etc/myapp/config.yaml`

</details>

**13. `sar -r` 输出中 `%memused` 持续增长、`kbcached` 持续下降意味着什么？**

<details>
<summary>查看答案</summary>

这表明 **真实内存使用在增长**（不是缓存增长）：

- `%memused` 增长：总内存使用率上升
- `kbcached` 下降：内核正在回收缓存页面来满足应用的内存需求

这是**内存不足的早期信号**。当缓存被完全回收后，系统将开始使用 swap，性能会急剧下降。

**应对措施：**
1. 立即用 `ps aux --sort=-%mem` 找出内存消耗最大的进程
2. 检查是否有内存泄漏
3. 考虑增加物理内存

</details>

---

## 评分指南

| 得分 | 等级 | 说明 |
|------|------|------|
| 12-13 | ⭐⭐⭐ 优秀 | 高级 Linux 管理能力扎实 |
| 9-11 | ⭐⭐ 良好 | 掌握了核心知识，部分细节需加强 |
| 6-8 | ⭐ 合格 | 基础掌握，建议回顾薄弱章节 |
| < 6 | 需要复习 | 建议重新学习第 17-23 课 |

---

## 课程总结

恭喜你完成了整个高级篇的学习！回顾一下你掌握的技能：

| 课程 | 核心技能 |
|------|----------|
| 第 17 课 | Shell 脚本基础：变量、条件、循环、参数处理 |
| 第 18 课 | 函数与脚本组织：模块化、返回值、作用域 |
| 第 19 课 | 高级脚本技巧：数组、正则、信号处理、调试 |
| 第 20 课 | Cron 与自动化：定时任务、at、systemd Timer |
| 第 21 课 | 系统管理：用户、sudo、服务、日志 |
| 第 22 课 | 磁盘与存储：df/du、LVM、fstab、swap |
| 第 23 课 | 性能调优：top、vmstat、iostat、strace、lsof |
| 第 24 课 | 综合实战：监控系统项目 + 综合测验 |

### 推荐的下一步

1. **实践**：在真实服务器或虚拟机上部署你的监控脚本
2. **深入学习**：
   - 容器技术：Docker、Kubernetes
   - 配置管理：Ansible、Terraform
   - 监控平台：Prometheus + Grafana、Zabbix
   - CI/CD：GitHub Actions、Jenkins
3. **认证考试**：
   - LPIC-1 / LPIC-2（Linux Professional Institute）
   - RHCSA / RHCE（Red Hat）
   - CompTIA Linux+
4. **开源贡献**：参与 Linux 相关开源项目，在实战中提升技能

🎉 **你已经具备了 Linux 系统管理的核心技能。继续练习，保持好奇心，成为真正的 Linux 专家！**
