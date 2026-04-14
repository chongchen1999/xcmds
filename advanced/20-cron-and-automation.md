# 第 20 课：Cron Job 与自动化

> 上一课：[高级脚本技巧](19-advanced-scripting.md) | 下一课：[系统管理](21-system-admin.md)

## 学习目标

- 掌握 Crontab 的语法与常用调度模式
- 理解 Cron 的环境限制及排错方法
- 学会使用 `at` 命令进行一次性任务调度
- 了解 systemd Timer 作为现代替代方案
- 能编写实用的自动化脚本（备份、日志轮转）

---

## 知识讲解

### 1. Crontab 语法

Crontab 的每一行由 5 个时间字段 + 命令组成：

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * * command
```

#### 特殊字符

| 字符 | 含义 | 示例 |
|------|------|------|
| `*` | 任意值 | `* * * * *`（每分钟） |
| `,` | 列举多个值 | `1,15,30 * * * *` |
| `-` | 范围 | `1-5`（周一至周五） |
| `/` | 步长 | `*/5 * * * *`（每 5 分钟） |

#### 预定义调度符

| 符号 | 等价于 | 含义 |
|------|--------|------|
| `@reboot` | — | 系统启动时执行一次 |
| `@hourly` | `0 * * * *` | 每小时 |
| `@daily` | `0 0 * * *` | 每天午夜 |
| `@weekly` | `0 0 * * 0` | 每周日午夜 |
| `@monthly` | `0 0 1 * *` | 每月 1 号午夜 |
| `@yearly` | `0 0 1 1 *` | 每年 1 月 1 日 |

### 2. Crontab 命令

```bash
# Edit current user's crontab
crontab -e

# List current user's crontab
crontab -l

# Remove current user's crontab (use with caution!)
crontab -r

# Edit another user's crontab (root only)
sudo crontab -u www-data -e
```

### 3. 常用 Cron 模式

```bash
# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Every hour at :30
30 * * * * /path/to/script.sh

# Every day at 2:00 AM
0 2 * * * /path/to/script.sh

# Weekdays at 9:00 AM
0 9 * * 1-5 /path/to/script.sh

# Every Monday and Thursday at 6:00 PM
0 18 * * 1,4 /path/to/script.sh

# 1st and 15th of each month at midnight
0 0 1,15 * * /path/to/script.sh

# Every 15 minutes during business hours (9-17) on weekdays
*/15 9-17 * * 1-5 /path/to/script.sh

# First Monday of every month at 3:00 AM
0 3 1-7 * 1 /path/to/script.sh
```

### 4. Cron 环境陷阱

Cron 执行命令时的环境与交互式 Shell 差异很大，这是最常见的问题来源：

#### PATH 问题

Cron 的默认 PATH 很有限（通常只有 `/usr/bin:/bin`）：

```bash
# Bad: command may not be found
* * * * * my_script.sh

# Good: use absolute paths
* * * * * /home/user/bin/my_script.sh

# Good: set PATH explicitly
* * * * * PATH=/usr/local/bin:/usr/bin:/bin /home/user/bin/my_script.sh
```

也可以在 crontab 开头设置全局环境：

```bash
# Set at top of crontab
PATH=/usr/local/bin:/usr/bin:/bin:/home/user/bin
SHELL=/bin/bash
MAILTO=admin@example.com

# Jobs below inherit these settings
0 2 * * * backup.sh
0 3 * * * cleanup.sh
```

#### 无终端环境

Cron 中没有终端（no TTY），某些命令会出错：

```bash
# Bad: interactive commands will fail
* * * * * read -p "input: " var

# Bad: programs that need a terminal
* * * * * vim /etc/config

# Good: ensure non-interactive
* * * * * DEBIAN_FRONTEND=noninteractive apt-get update -y
```

#### 工作目录

Cron 任务的工作目录是用户的 home 目录，不要依赖相对路径：

```bash
# Bad: depends on working directory
0 2 * * * cd project && ./run.sh

# Good: use absolute paths everywhere
0 2 * * * /home/user/project/run.sh
```

### 5. 日志记录

Cron 默认将输出通过邮件发送。实际环境中通常重定向到日志文件：

```bash
# Redirect stdout and stderr to log
0 2 * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Discard all output
0 2 * * * /path/to/script.sh > /dev/null 2>&1

# Log with timestamp
0 2 * * * /path/to/script.sh 2>&1 | while IFS= read -r line; do echo "$(date '+\%Y-\%m-\%d \%H:\%M:\%S') $line"; done >> /var/log/myjob.log

# Use logger to write to syslog
0 2 * * * /path/to/script.sh 2>&1 | logger -t myjob
```

在脚本内部实现日志记录更加可靠：

```bash
#!/bin/bash
LOGFILE="/var/log/myapp/backup.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" >> "$LOGFILE"
}

log "INFO" "Backup started"
# ... do work ...
log "INFO" "Backup completed"
```

### 6. `at` 命令：一次性调度

`at` 用于安排一次性任务，执行后自动删除：

```bash
# Run at specific time
echo "/path/to/script.sh" | at 14:30

# Run at specific date and time
echo "/path/to/script.sh" | at 2:00 AM 2026-04-20

# Run after delay
echo "/path/to/script.sh" | at now + 30 minutes
echo "/path/to/script.sh" | at now + 2 hours
echo "/path/to/script.sh" | at now + 1 day

# Interactive mode
at 10:00 tomorrow << 'EOF'
/home/user/backup.sh
echo "Backup done" | mail -s "Backup" admin@example.com
EOF
```

#### 管理 `at` 任务

```bash
# List pending jobs
atq

# View job details
at -c <job_number>

# Remove a job
atrm <job_number>
```

#### 访问控制

- `/etc/at.allow`：允许使用 `at` 的用户
- `/etc/at.deny`：禁止使用 `at` 的用户

### 7. systemd Timer：现代替代方案

systemd Timer 是 Cron 的现代替代，提供更精确的控制和更好的日志集成。

#### 创建一个 Timer

需要两个文件：一个 `.service` 单元和一个 `.timer` 单元。

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily backup job

[Service]
Type=oneshot
ExecStart=/home/user/scripts/backup.sh
User=user
```

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily at 2am

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

#### 管理 Timer

```bash
# Enable and start
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Check status
systemctl status backup.timer
systemctl list-timers --all

# View logs
journalctl -u backup.service

# Run manually
sudo systemctl start backup.service

# Disable
sudo systemctl disable backup.timer
```

#### OnCalendar 语法

```bash
# Every day at 3:00 AM
OnCalendar=*-*-* 03:00:00

# Weekdays at 9:00 AM
OnCalendar=Mon..Fri *-*-* 09:00:00

# Every 15 minutes
OnCalendar=*:0/15

# First day of month
OnCalendar=*-*-01 00:00:00

# Test calendar expressions
systemd-analyze calendar "Mon..Fri *-*-* 09:00:00"
```

#### Cron vs systemd Timer

| 特性 | Cron | systemd Timer |
|------|------|---------------|
| 配置方式 | 单行表达式 | 需要两个文件 |
| 日志 | 自行处理 | 自动集成 journalctl |
| 错过执行 | 跳过 | `Persistent=true` 补执行 |
| 依赖管理 | 无 | 可依赖其他 service |
| 精度 | 分钟级 | 秒级 |
| 资源控制 | 无 | 支持 cgroup 限制 |

### 8. 实战：自动化备份脚本

```bash
#!/bin/bash
set -euo pipefail

# --- Configuration ---
readonly BACKUP_DIRS=("/etc" "/home" "/var/www")
readonly BACKUP_DEST="/backup"
readonly RETENTION_DAYS=30
readonly DATE_STAMP=$(date +%Y%m%d_%H%M%S)
readonly HOSTNAME=$(hostname)
readonly LOGFILE="/var/log/backup.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') [$1] $2" | tee -a "$LOGFILE"
}

log "INFO" "=== Backup started ==="

mkdir -p "$BACKUP_DEST"

# Perform backup
for dir in "${BACKUP_DIRS[@]}"; do
    if [[ ! -d "$dir" ]]; then
        log "WARN" "Skipping $dir (not found)"
        continue
    fi

    name="$(echo "$dir" | tr '/' '_' | sed 's/^_//')"
    archive="${BACKUP_DEST}/${HOSTNAME}_${name}_${DATE_STAMP}.tar.gz"

    log "INFO" "Backing up $dir -> $archive"
    if tar -czf "$archive" "$dir" 2>> "$LOGFILE"; then
        size=$(du -h "$archive" | cut -f1)
        log "INFO" "Success: $archive ($size)"
    else
        log "ERROR" "Failed to backup $dir"
    fi
done

# Clean old backups
log "INFO" "Removing backups older than $RETENTION_DAYS days"
find "$BACKUP_DEST" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete -print \
    | while read -r f; do log "INFO" "Deleted: $f"; done

log "INFO" "=== Backup completed ==="
```

安装到 Crontab：

```bash
# Run daily at 2:00 AM
crontab -l 2>/dev/null; echo "0 2 * * * /home/user/scripts/backup.sh >> /var/log/backup_cron.log 2>&1"
```

### 9. 实战：日志轮转脚本

```bash
#!/bin/bash
set -euo pipefail

# --- Configuration ---
readonly LOG_DIR="/var/log/myapp"
readonly MAX_FILES=7        # Keep 7 rotated copies
readonly COMPRESS_AFTER=1   # Compress files older than 1 rotation

rotate_log() {
    local logfile="$1"
    local base
    base="$(basename "$logfile")"

    [[ -f "$logfile" ]] || return 0

    # Remove oldest
    [[ -f "${logfile}.${MAX_FILES}.gz" ]] && rm -f "${logfile}.${MAX_FILES}.gz"

    # Shift existing rotations
    for (( i = MAX_FILES - 1; i >= 1; i-- )); do
        local next=$(( i + 1 ))
        [[ -f "${logfile}.${i}.gz" ]] && mv "${logfile}.${i}.gz" "${logfile}.${next}.gz"
        [[ -f "${logfile}.${i}" ]]    && mv "${logfile}.${i}" "${logfile}.${next}"
    done

    # Rotate current file
    cp "$logfile" "${logfile}.1"
    truncate -s 0 "$logfile"

    # Compress older rotations
    for (( i = COMPRESS_AFTER + 1; i <= MAX_FILES; i++ )); do
        [[ -f "${logfile}.${i}" ]] && gzip "${logfile}.${i}"
    done

    echo "Rotated: $base"
}

# Rotate all .log files in the directory
for logfile in "$LOG_DIR"/*.log; do
    [[ -f "$logfile" ]] || continue
    rotate_log "$logfile"
done
```

---

## 实战演练

### 练习 1：查看和编辑 Crontab

```bash
# List current crontab
crontab -l

# Add a simple test job
(crontab -l 2>/dev/null; echo "*/1 * * * * echo 'cron test' >> /tmp/crontest.log") | crontab -

# Verify
crontab -l

# Wait 2 minutes, check output
sleep 120
cat /tmp/crontest.log

# Clean up
crontab -l | grep -v 'crontest' | crontab -
rm -f /tmp/crontest.log
```

### 练习 2：编写 Cron-safe 脚本

```bash
cat > ~/cron_safe.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

# Ensure full PATH
export PATH="/usr/local/bin:/usr/bin:/bin:$PATH"

# Use absolute paths
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly LOGFILE="/tmp/cron_safe_$(date +%Y%m%d).log"

# Prevent concurrent execution with lock file
readonly LOCKFILE="/tmp/$(basename "$0").lock"

cleanup() {
    rm -f "$LOCKFILE"
}
trap cleanup EXIT

if ! mkdir "$LOCKFILE" 2>/dev/null; then
    echo "Another instance is running" >&2
    exit 1
fi

# Actual work
{
    echo "=== $(date) ==="
    echo "PATH: $PATH"
    echo "PWD: $(pwd)"
    echo "USER: $(whoami)"
    echo "Script dir: $SCRIPT_DIR"
    df -h /
    echo "---"
} >> "$LOGFILE"
SCRIPT

chmod +x ~/cron_safe.sh
~/cron_safe.sh
cat /tmp/cron_safe_$(date +%Y%m%d).log
```

### 练习 3：使用 `at` 调度

```bash
# Schedule a job for 1 minute from now
echo 'echo "at job executed at $(date)" >> /tmp/at_test.log' | at now + 1 minute

# List pending jobs
atq

# Wait and check
sleep 70
cat /tmp/at_test.log

rm -f /tmp/at_test.log
```

### 练习 4：创建 systemd Timer

```bash
# Create service file
sudo tee /etc/systemd/system/cleanup.service << 'EOF'
[Unit]
Description=Clean up temp files

[Service]
Type=oneshot
ExecStart=/usr/bin/find /tmp -type f -name "*.tmp" -mtime +7 -delete
EOF

# Create timer file
sudo tee /etc/systemd/system/cleanup.timer << 'EOF'
[Unit]
Description=Run cleanup daily

[Timer]
OnCalendar=*-*-* 04:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable cleanup.timer
sudo systemctl start cleanup.timer

# Check status
systemctl status cleanup.timer
systemctl list-timers | grep cleanup
```

---

## 小结

| 概念 | 命令 / 语法 | 说明 |
|------|------------|------|
| 编辑 Crontab | `crontab -e` | 编辑当前用户的定时任务 |
| 查看 Crontab | `crontab -l` | 列出所有定时任务 |
| 删除 Crontab | `crontab -r` | 删除所有定时任务 |
| 时间格式 | `分 时 日 月 周` | 5 个字段 |
| 每 N 分钟 | `*/N * * * *` | 步长表达式 |
| 输出日志 | `>> log 2>&1` | 重定向 stdout 和 stderr |
| 一次性调度 | `echo cmd \| at TIME` | 执行一次后自动删除 |
| 查看 at 队列 | `atq` | 列出待执行的一次性任务 |
| systemd Timer | `.timer` + `.service` | 现代定时任务方案 |
| 查看 Timer | `systemctl list-timers` | 列出所有 Timer |

---

## 练习

**1. 写出以下 Cron 表达式的含义：**
- `30 4 * * 1-5`
- `0 */2 * * *`
- `15 0 1 * *`
- `0 9,18 * * *`

<details>
<summary>查看答案</summary>

- `30 4 * * 1-5`：每个工作日（周一到周五）凌晨 4:30 执行
- `0 */2 * * *`：每 2 小时的整点执行（0:00, 2:00, 4:00, ...）
- `15 0 1 * *`：每月 1 号 0:15 执行
- `0 9,18 * * *`：每天 9:00 和 18:00 执行

</details>

**2. 写一个 Cron 表达式：每周一到周五的上午 8 点到下午 6 点之间，每 30 分钟执行一次健康检查脚本。**

<details>
<summary>查看答案</summary>

```
*/30 8-18 * * 1-5 /path/to/healthcheck.sh >> /var/log/healthcheck.log 2>&1
```

注意：`8-18` 表示 8 点到 18 点（含两端），`*/30` 表示每 30 分钟，`1-5` 表示周一到周五。

</details>

**3. 你的 Cron 任务 `* * * * * python3 myscript.py` 没有执行，列出 3 个可能的原因和排查方法。**

<details>
<summary>查看答案</summary>

1. **PATH 问题**：Cron 环境的 PATH 不包含 `python3` 所在路径。
   - 排查：使用绝对路径 `/usr/bin/python3`，或在 crontab 开头设置 `PATH=...`
   - 验证：`which python3` 确认路径

2. **相对路径问题**：`myscript.py` 使用了相对路径，Cron 的工作目录可能不是你期望的位置。
   - 排查：使用绝对路径 `/home/user/myscript.py`
   - 验证：在脚本中添加 `import os; print(os.getcwd())` 查看工作目录

3. **权限问题**：脚本或相关文件没有正确的权限。
   - 排查：`chmod +x myscript.py`，检查输出文件的写入权限
   - 验证：查看 `/var/log/syslog` 或 `/var/log/cron` 中的错误信息

额外排查方法：添加日志输出 `* * * * * /usr/bin/python3 /path/to/myscript.py >> /tmp/cron_debug.log 2>&1`

</details>

**4. 编写一个完整的自动化脚本，实现以下功能：每天检查磁盘使用率，超过 80% 时发送告警（写入日志文件），并通过 Crontab 调度。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# disk_alert.sh - Alert when disk usage exceeds threshold
set -euo pipefail

readonly THRESHOLD=80
readonly LOGFILE="/var/log/disk_alert.log"
readonly ALERT_FILE="/tmp/disk_alert_sent"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOGFILE"
}

usage=$(df / | awk 'NR==2 {gsub(/%/,""); print $5}')

if (( usage >= THRESHOLD )); then
    if [[ ! -f "$ALERT_FILE" ]]; then
        log "ALERT: Disk usage at ${usage}% (threshold: ${THRESHOLD}%)"
        log "Disk details:"
        df -h / >> "$LOGFILE"
        touch "$ALERT_FILE"
    fi
else
    # Clear alert flag when usage drops
    rm -f "$ALERT_FILE"
    log "OK: Disk usage at ${usage}%"
fi
```

Crontab 配置：

```bash
# Check disk every hour
0 * * * * /home/user/scripts/disk_alert.sh
```

</details>

**5. 比较 Cron 和 systemd Timer，在什么场景下你会选择 systemd Timer？**

<details>
<summary>查看答案</summary>

选择 systemd Timer 的场景：

1. **需要补执行错过的任务**：`Persistent=true` 可以在系统恢复后补执行停机期间错过的任务，Cron 会直接跳过。

2. **需要精确到秒的调度**：Cron 的最小精度是分钟，systemd Timer 支持秒级精度（如 `OnCalendar=*:*:0/30` 每 30 秒执行一次）。

3. **需要查看执行日志**：systemd Timer 自动集成 `journalctl`，可以方便地查看历史执行记录和错误信息，无需手动配置日志。

4. **需要依赖管理**：可以声明 Timer 依赖于网络就绪、某个挂载点可用等条件（`After=network-online.target`）。

5. **需要资源限制**：可以通过 Service 文件限制 CPU、内存使用（`MemoryMax=512M`、`CPUQuota=50%`），防止定时任务耗尽系统资源。

</details>
