# 第 23 课：性能调优与监控

> 上一课：[磁盘与存储管理](22-disk-and-storage.md) | 下一课：[高级综合实战项目](24-capstone-project.md)

## 学习目标

- 理解 Linux 性能监控的核心指标（CPU、内存、I/O、网络）
- 熟练使用 `top`/`htop` 分析系统负载
- 掌握 `vmstat`、`iostat`、`free`、`sar` 等专项监控工具
- 学会使用 `strace` 追踪系统调用
- 学会使用 `lsof` 查看打开文件
- 建立系统性的性能排查思路

---

## 知识讲解

### 1. 性能监控概述

性能问题通常归结为四个资源瓶颈：

| 资源 | 核心指标 | 主要工具 |
|------|----------|----------|
| **CPU** | load average、%user、%system、%iowait | `top`、`mpstat`、`sar` |
| **内存** | used、available、swap usage | `free`、`vmstat`、`top` |
| **磁盘 I/O** | IOPS、吞吐量、await | `iostat`、`iotop` |
| **网络** | 带宽、连接数、丢包率 | `ss`、`iftop`、`sar -n` |

排查的基本策略：先用综合工具（`top`）确定大方向，再用专项工具深入分析。

### 2. top / htop 深入解析

#### top

`top` 是最常用的实时系统监控工具：

```bash
top
```

输出分为两部分——**概要区域**和**进程列表**：

```
top - 14:23:05 up 45 days,  3:12,  2 users,  load average: 2.52, 1.38, 0.91
Tasks: 287 total,   2 running, 285 sleeping,   0 stopped,   0 zombie
%Cpu(s): 25.3 us,  5.1 sy,  0.0 ni, 65.2 id,  3.8 wa,  0.0 hi,  0.6 si,  0.0 st
MiB Mem :  15921.4 total,   1234.5 free,   8456.2 used,   6230.7 buff/cache
MiB Swap:   2048.0 total,   1890.3 free,    157.7 used.   6012.1 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 3421 mysql     20   0 2.5g  1.2g  12340 S  45.2   7.7  12:34.56 mysqld
 8765 www-data  20   0  856m  450m   8900 S  12.3   2.8   5:43.21 php-fpm
```

##### Load Average 详解

`load average: 2.52, 1.38, 0.91` 表示过去 1、5、15 分钟的平均负载。

负载值 = 正在运行 + 等待运行的进程数（包括等待 I/O 的进程）。

```bash
# Check CPU cores to interpret load average
nproc
# If 4 cores: load 4.0 = fully utilized, >4.0 = overloaded
```

- 1 分钟 > 5 分钟 > 15 分钟：负载正在**上升**
- 1 分钟 < 5 分钟 < 15 分钟：负载正在**下降**
- 三个值接近：负载**稳定**

##### CPU 状态

| 缩写 | 全称 | 说明 |
|------|------|------|
| `us` | user | 用户空间（应用代码） |
| `sy` | system | 内核空间（系统调用） |
| `ni` | nice | 低优先级用户进程 |
| `id` | idle | 空闲 |
| `wa` | iowait | 等待 I/O 完成 |
| `hi` | hardware interrupt | 硬件中断 |
| `si` | software interrupt | 软件中断 |
| `st` | steal | 虚拟机被宿主机占用的时间 |

**关键判断：**
- `wa` 高 → 磁盘 I/O 瓶颈
- `us` 高 → 应用层 CPU 密集
- `sy` 高 → 大量系统调用（可能是进程创建、上下文切换过多）
- `st` 高 → 虚拟机资源不足

##### top 交互命令

| 按键 | 功能 |
|------|------|
| `1` | 显示每个 CPU 核心的状态 |
| `M` | 按内存排序 |
| `P` | 按 CPU 排序 |
| `T` | 按运行时间排序 |
| `k` | Kill 进程 |
| `r` | Renice 进程 |
| `f` | 选择显示的字段 |
| `c` | 显示完整命令行 |
| `H` | 显示线程 |
| `q` | 退出 |

#### htop

`htop` 是 `top` 的增强版，提供彩色界面、鼠标支持和更直观的操作：

```bash
htop

# Filter by user
htop -u www-data

# Show tree view
htop -t
```

`htop` 优势：
- 彩色 CPU/内存柱状图
- 鼠标点击排序
- 树形视图（`F5`）显示进程父子关系
- 搜索（`F3`）和过滤（`F4`）
- 直接发送信号（`F9`）

### 3. vmstat — 虚拟内存统计

`vmstat` 提供 CPU、内存、I/O、进程调度的综合快照：

```bash
# Single snapshot
vmstat

# Report every 2 seconds, 10 times
vmstat 2 10
```

输出解读：

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0  15736 123456  45678 678901    0    0    12    56  234  567 25  5 65  4  0
```

| 列 | 说明 | 关注点 |
|------|------|--------|
| `r` | 等待 CPU 的进程数 | 持续 > CPU 核数则过载 |
| `b` | 不可中断睡眠（等 I/O） | 持续 > 0 则有 I/O 瓶颈 |
| `swpd` | 使用的 swap（KB） | 持续增长说明内存不足 |
| `si/so` | swap in/out（KB/s） | > 0 表示正在使用 swap |
| `bi/bo` | 块设备 read/write（KB/s） | 磁盘 I/O 量 |
| `in` | 中断数/秒 | — |
| `cs` | 上下文切换数/秒 | 过高可能有性能问题 |
| `wa` | CPU 等待 I/O 的百分比 | 高则 I/O 瓶颈 |

```bash
# Focus on memory with timestamps
vmstat -t -S M 1 5
```

### 4. iostat — I/O 统计

`iostat`（来自 `sysstat` 包）监控磁盘 I/O 性能：

```bash
# Basic output
iostat

# Extended stats, every 2 seconds
iostat -xz 2

# Human-readable, specific device
iostat -xh /dev/sda 1
```

关键指标：

```
Device   r/s    w/s   rkB/s   wkB/s  rrqm/s  wrqm/s  %rrqm  %wrqm  r_await  w_await  aqu-sz  rareq-sz  wareq-sz  svctm  %util
sda     45.00  120.00 1800.0  9600.0   2.00   30.00   4.26   20.00    1.20     2.50    0.45    40.00     80.00     1.50   24.75
```

| 指标 | 说明 | 告警阈值 |
|------|------|----------|
| `r/s` / `w/s` | 每秒读/写请求数 | 取决于磁盘类型 |
| `r_await` / `w_await` | 平均读/写延迟（ms） | HDD > 20ms，SSD > 5ms |
| `aqu-sz` | 平均请求队列长度 | 持续 > 1 可能有瓶颈 |
| `%util` | 设备繁忙百分比 | 持续 > 80% 需关注 |

### 5. free — 内存使用

```bash
# Human-readable
free -h

# Output:
#               total        used        free      shared  buff/cache   available
# Mem:           15Gi       8.0Gi       1.2Gi       256Mi       6.1Gi       5.9Gi
# Swap:         2.0Gi       154Mi       1.8Gi
```

**理解 Linux 内存管理：**

- `free` 不等于"浪费"——Linux 会尽量用空闲内存做磁盘缓存
- **`available`** 才是真正能给新进程使用的内存（free + 可回收的 cache）
- 判断是否内存不足看 `available`，不要看 `free`

```bash
# Continuous monitoring
free -h -s 2   # Every 2 seconds

# Wide format (separate buffers and cache)
free -hw
```

### 6. sar — 系统活动报告

`sar`（来自 `sysstat` 包）是最全面的历史性能数据工具，可以回看过去的系统状态：

```bash
# CPU usage (current, every 2 seconds, 5 times)
sar -u 2 5

# CPU usage for today
sar -u

# CPU usage for a specific date
sar -u -f /var/log/sysstat/sa14

# Memory usage
sar -r 2 5

# Swap usage
sar -S

# Disk I/O
sar -d 2 5

# Network interface stats
sar -n DEV 2 5

# Network errors
sar -n EDEV

# Context switches and interrupts
sar -w 2 5

# Load average
sar -q 2 5
```

`sar` 的数据由 `sysstat` 的 cron 任务每 10 分钟自动采集，存储在 `/var/log/sysstat/` 目录：

```bash
# Ensure sysstat is collecting data
sudo systemctl enable --now sysstat
```

### 7. strace — 追踪系统调用

`strace` 追踪进程的系统调用，是排查程序异常的利器：

```bash
# Trace a command
strace ls /tmp

# Trace a running process
sudo strace -p <PID>

# Trace with timestamps
strace -t ls /tmp

# Summary only (count and time per syscall)
strace -c ls /tmp
# Example output:
# % time     seconds  usecs/call     calls    errors syscall
# ------ ----------- ----------- --------- --------- ----------------
#  35.12    0.000234          23        10           openat
#  22.45    0.000150          15        10           read
#  18.67    0.000125          12        10         5 stat

# Trace only file-related calls
strace -e trace=file ls /tmp

# Trace only network calls
strace -e trace=network curl example.com

# Follow child processes
strace -f bash -c "ls | grep test"

# Output to file
strace -o /tmp/trace.log -p <PID>
```

常见用途：
- 找出程序在读取哪些配置文件
- 排查 "file not found" 但不确定哪个文件
- 分析程序为何卡住（看它在等什么系统调用）
- 找出程序的网络连接目标

### 8. lsof — 列出打开的文件

Linux 中"一切皆文件"，`lsof` 可以显示进程打开的所有文件（包括网络连接、设备等）：

```bash
# All open files (very long list)
sudo lsof | head -20

# Files opened by a specific process
sudo lsof -p <PID>

# Files opened by a specific user
sudo lsof -u www-data

# Who is using a specific file
sudo lsof /var/log/syslog

# Who is using a specific directory
sudo lsof +D /var/log/

# Network connections (like ss/netstat)
sudo lsof -i

# Specific port
sudo lsof -i :80
sudo lsof -i :443

# TCP connections only
sudo lsof -i TCP

# Find deleted but still open files (occupying disk space)
sudo lsof | grep '(deleted)'

# Who is using a specific filesystem (useful before umount)
sudo lsof +f -- /mnt/data
```

**实用场景：**

```bash
# "Device busy" when trying to umount? Find who's using it:
sudo lsof +f -- /mnt/usb

# Port already in use? Find the process:
sudo lsof -i :8080

# Disk space not freed after deleting files? Find deleted-but-open files:
sudo lsof | grep deleted | sort -k7 -rn | head -10
```

### 9. 性能排查方法论

当系统变慢时，按照以下步骤系统化排查：

#### 第一步：概览

```bash
# Quick system overview
uptime          # Load average
free -h         # Memory
df -h           # Disk space
```

#### 第二步：定位瓶颈

```bash
# Is it CPU, memory, I/O, or network?
top             # Interactive overview
vmstat 1 5      # Check r, b, wa, si/so
iostat -xz 1 3  # Disk I/O detail
```

**判断逻辑：**

```
系统变慢
├── load average 高？
│   ├── %wa 高 → 磁盘 I/O 问题 → iostat, iotop
│   ├── %us 高 → CPU 密集进程 → top (sort by CPU)
│   └── %sy 高 → 内核/系统调用过多 → strace
├── available 内存低？
│   ├── swap 使用多？ → 内存不足，需要加内存或找出内存泄漏进程
│   └── OOM killer 日志？ → journalctl -k | grep -i oom
├── 磁盘 %util 高？
│   ├── await 高 → 磁盘慢 → 检查硬件或升级 SSD
│   └── 找出 I/O 密集进程 → iotop
└── 都正常？
    └── 检查网络 → ss, ping, sar -n DEV
```

#### 第三步：深入分析

```bash
# Find the problematic process
top -c                     # Full command line
ps aux --sort=-%cpu | head # Top CPU consumers
ps aux --sort=-%mem | head # Top memory consumers

# Trace the suspect process
strace -c -p <PID>         # Syscall summary
lsof -p <PID>              # What files is it using

# Check logs
journalctl -u <service> --since "30 min ago"
dmesg -T | tail -50
```

---

## 实战演练

### 练习 1：系统负载分析

```bash
# Check current load
uptime

# Monitor with vmstat (every 1 second, 10 iterations)
vmstat 1 10

# Check per-CPU usage
mpstat -P ALL 1 3

# Generate some load for testing (use with caution)
# CPU stress (run in background, kill after testing)
dd if=/dev/urandom of=/dev/null bs=1M &
PID_STRESS=$!

# Observe the change
top -bn1 | head -5

# Clean up
kill $PID_STRESS
```

### 练习 2：内存分析

```bash
# Detailed memory info
free -h
cat /proc/meminfo | head -20

# Top memory consumers
ps aux --sort=-%mem | head -10

# Monitor memory over time
vmstat -S M 1 5

# Check for OOM events
sudo journalctl -k | grep -i "out of memory"
sudo dmesg | grep -i oom
```

### 练习 3：I/O 分析

```bash
# Disk I/O stats
iostat -xz 1 5

# If available: interactive I/O monitor
sudo iotop -o   # Show only processes doing I/O

# Check which processes use the most I/O
sudo iotop -b -n 3 -o
```

### 练习 4：使用 strace 排查

```bash
# Trace file access of a command
strace -e trace=openat cat /etc/hostname 2>&1 | head -20

# Count syscalls
strace -c ls /usr 2>&1

# Trace network connections
strace -e trace=network curl -s -o /dev/null https://example.com 2>&1 | head -20
```

### 练习 5：使用 lsof

```bash
# Find what's listening on ports
sudo lsof -i -P -n | grep LISTEN

# Find open files in /var/log
sudo lsof +D /var/log 2>/dev/null | head -20

# Count open files per process
sudo lsof 2>/dev/null | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

---

## 小结

| 概念 | 命令 | 说明 |
|------|------|------|
| 综合监控 | `top` / `htop` | 实时查看 CPU、内存、进程 |
| 负载查看 | `uptime` | load average |
| 内存监控 | `free -h` | 内存和 swap 使用 |
| 虚拟内存 | `vmstat 1` | CPU、内存、I/O 综合快照 |
| 磁盘 I/O | `iostat -xz 1` | 磁盘性能详细指标 |
| I/O 进程 | `iotop` | 按进程查看 I/O |
| 历史数据 | `sar` | 回看历史性能数据 |
| 系统调用 | `strace -p PID` | 追踪进程的系统调用 |
| 打开文件 | `lsof` | 查看进程打开的文件/网络 |
| 端口占用 | `lsof -i :PORT` | 查看端口被哪个进程占用 |
| CPU 进程 | `ps aux --sort=-%cpu` | 按 CPU 排序的进程列表 |
| 内存进程 | `ps aux --sort=-%mem` | 按内存排序的进程列表 |
| 内核日志 | `dmesg -T` | 带时间戳的内核消息 |

---

## 练习

**1. 服务器 load average 显示 `8.50, 6.20, 2.10`，CPU 为 4 核。分析这个负载趋势并说明该怎么做。**

<details>
<summary>查看答案</summary>

**分析：**
- 15 分钟平均：2.10（正常，低于 4 核）
- 5 分钟平均：6.20（偏高，超过 4 核 55%）
- 1 分钟平均：8.50（严重过载，是核数的 2 倍多）

**趋势：** 负载正在急剧上升（1min > 5min > 15min），说明是近期发生的问题。

**排查步骤：**

```bash
# 1. Check if it's CPU or I/O
top   # Look at %wa (iowait)

# 2. If %wa is low (CPU-bound):
ps aux --sort=-%cpu | head -5

# 3. If %wa is high (I/O-bound):
iostat -xz 1 3
sudo iotop -o

# 4. Check for recently started processes
ps aux --sort=-start_time | head -10
```

如果是某个进程突然占用大量资源，可以用 `kill`/`renice` 处理；如果是正常业务增长导致，则需要考虑扩容。

</details>

**2. `free -h` 显示 free 只有 200MB，但 available 有 6GB。系统是否内存不足？为什么？**

<details>
<summary>查看答案</summary>

**系统内存充足**，不存在不足问题。

**原因：** Linux 内核会尽量利用空闲内存做磁盘缓存（buff/cache），加速文件读写。这些缓存在需要时可以立即释放给应用使用。

- `free`（200MB）：完全未被使用的内存
- `buff/cache`：被内核用作缓存的内存，可回收
- `available`（6GB）：`free` + 可回收的缓存 = 新进程实际可用的内存

**判断标准：**
- 看 `available`，不看 `free`
- 只有当 `available` 也很低且 swap 大量使用时，才表示内存不足

</details>

**3. 如何找出占用 8080 端口的进程并查看它的详细信息？**

<details>
<summary>查看答案</summary>

```bash
# Method 1: lsof
sudo lsof -i :8080
# Output shows PID, user, and command

# Method 2: ss
sudo ss -tlnp | grep 8080
# Output shows the process in the last column

# Get detailed info about the process
# Assuming PID is 12345:
ps aux | grep 12345
cat /proc/12345/cmdline | tr '\0' ' '
ls -la /proc/12345/exe       # Actual binary
cat /proc/12345/status       # Memory, threads, etc.
sudo lsof -p 12345           # All open files
```

</details>

**4. 写一个命令序列，快速生成当前系统的性能快照（one-liner 或短脚本）。**

<details>
<summary>查看答案</summary>

```bash
#!/bin/bash
# perf_snapshot.sh - Quick system performance snapshot

echo "=== $(date) ==="
echo ""
echo "--- Load & Uptime ---"
uptime
echo ""
echo "--- CPU ---"
mpstat 1 1 2>/dev/null || top -bn1 | head -3
echo ""
echo "--- Memory ---"
free -h
echo ""
echo "--- Swap ---"
swapon --show 2>/dev/null
echo ""
echo "--- Disk I/O ---"
iostat -xz 1 1 2>/dev/null || echo "iostat not available"
echo ""
echo "--- Disk Space ---"
df -h -x tmpfs -x devtmpfs
echo ""
echo "--- Top 5 CPU Processes ---"
ps aux --sort=-%cpu | head -6
echo ""
echo "--- Top 5 Memory Processes ---"
ps aux --sort=-%mem | head -6
echo ""
echo "--- Network Connections ---"
ss -s
echo ""
echo "--- Failed Services ---"
systemctl --failed --no-pager 2>/dev/null
```

</details>

**5. 一个 Java 应用运行一段时间后越来越慢，最终被 OOM killer 杀掉。用哪些工具排查？写出排查流程。**

<details>
<summary>查看答案</summary>

**排查流程：**

```bash
# 1. Confirm OOM event
sudo journalctl -k | grep -i "out of memory"
sudo dmesg | grep -i "oom"
# Look for: "Out of memory: Kill process XXXX (java)"

# 2. Monitor memory trend (before it crashes again)
# Run vmstat and watch swpd/si/so columns
vmstat 5

# 3. Watch the Java process memory over time
while true; do
    ps -p <PID> -o pid,rss,vsz,pmem,comm --no-headers
    sleep 10
done

# 4. Check if memory grows continuously (memory leak indicator)
# rss (Resident Set Size) should stabilize; if it keeps growing → leak

# 5. Analyze with strace
strace -c -p <PID>   # Which syscalls dominate?

# 6. Check open files (file descriptor leak?)
lsof -p <PID> | wc -l   # Run periodically, see if count grows

# 7. Application-level diagnostics
# For Java specifically:
jstat -gc <PID> 1000     # GC stats
jmap -heap <PID>         # Heap summary
jmap -histo <PID>        # Object histogram

# 8. Prevention: set memory limits
# In systemd service file:
# MemoryMax=2G
# Or JVM flags: -Xmx2g -Xms1g
```

**总结：** 这是典型的内存泄漏场景。关键是用 `vmstat`/`free` 确认内存趋势，用 `ps` 跟踪进程 RSS 增长，然后结合应用层工具定位泄漏源。

</details>
