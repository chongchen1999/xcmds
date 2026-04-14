# 第 12 课：Process 管理

> 上一课：[第 11 课：数据处理工具 awk](./11-awk.md) | 下一课：[第 13 课：网络工具](./13-networking.md)

---

## 学习目标

1. 掌握使用 `ps`、`top` 查看系统进程信息
2. 学会使用 `kill`、`killall`、`pkill` 终止进程
3. 理解前台 / 后台作业（job）管理机制
4. 掌握 `nohup`、`disown` 保持进程在终端关闭后继续运行
5. 了解进程优先级（`nice` / `renice`）和 `/proc` 文件系统

---

## 知识讲解

### 1. 查看进程：ps

`ps`（**P**rocess **S**tatus）用于查看当前系统进程的快照。

```bash
# current user's processes in current terminal
ps

# all processes, full format (BSD style)
ps aux

# all processes, full format (POSIX style)
ps -ef
```

`ps aux` 输出字段说明：

| 字段 | 含义 |
|------|------|
| `USER` | 进程所属用户 |
| `PID` | 进程 ID |
| `%CPU` | CPU 使用率 |
| `%MEM` | 内存使用率 |
| `VSZ` | 虚拟内存大小（KB） |
| `RSS` | 实际内存大小（KB） |
| `STAT` | 进程状态 |
| `START` | 启动时间 |
| `COMMAND` | 执行的命令 |

常见进程状态码：

| 状态 | 含义 |
|------|------|
| `R` | Running（运行中） |
| `S` | Sleeping（可中断休眠） |
| `D` | Uninterruptible sleep（不可中断休眠，通常是 I/O 等待） |
| `T` | Stopped（已暂停） |
| `Z` | Zombie（僵尸进程） |

实用组合：

```bash
# find a specific process
ps aux | grep nginx

# show process tree
ps auxf

# show threads
ps -eLf
```

### 2. 实时监控：top / htop

#### top

```bash
top
```

`top` 是一个实时的进程监控工具。常用交互快捷键：

| 按键 | 作用 |
|------|------|
| `q` | 退出 |
| `M` | 按内存排序 |
| `P` | 按 CPU 排序 |
| `k` | 输入 PID 终止进程 |
| `1` | 显示每个 CPU 核心的状态 |
| `h` | 帮助 |

```bash
# show specific user's processes
top -u www-data

# batch mode, run once (useful for scripts)
top -b -n 1
```

#### htop

`htop` 是 `top` 的增强版，提供彩色界面和鼠标支持（需要安装）：

```bash
# install
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # CentOS/RHEL

htop
```

`htop` 优势：支持鼠标操作、横向滚动、进程树视图、更直观的 CPU / 内存条形图。

### 3. 终止进程：kill / killall / pkill

#### kill —— 通过 PID 发送信号

```bash
# send SIGTERM (graceful termination, default)
kill 1234

# send SIGKILL (force kill, cannot be caught)
kill -9 1234

# send SIGHUP (reload config for many daemons)
kill -HUP 1234
```

常用信号：

| 信号 | 编号 | 作用 |
|------|------|------|
| `SIGHUP` | 1 | 挂起信号，常用于重载配置 |
| `SIGINT` | 2 | 中断（等同于 Ctrl+C） |
| `SIGKILL` | 9 | 强制终止（无法捕获） |
| `SIGTERM` | 15 | 优雅终止（默认） |
| `SIGSTOP` | 19 | 暂停进程（无法捕获） |
| `SIGCONT` | 18 | 恢复暂停的进程 |

```bash
# list all available signals
kill -l
```

#### killall —— 通过进程名终止

```bash
# kill all processes named "firefox"
killall firefox

# force kill
killall -9 python3
```

#### pkill —— 通过模式匹配终止

```bash
# kill processes matching pattern
pkill -f "python.*server"

# kill processes by user
pkill -u guest
```

> **推荐做法**：先用 `SIGTERM`（15）尝试优雅终止，等几秒后如果进程未退出再用 `SIGKILL`（9）。

### 4. 后台作业管理

#### 基本操作

```bash
# run command in background
long_task &

# list background jobs
jobs

# bring job to foreground
fg %1

# send running foreground job to background
# Step 1: press Ctrl+Z to suspend
# Step 2:
bg %1
```

#### 工作流示意

```
前台运行 ──Ctrl+Z──▶ 暂停（Stopped）──bg──▶ 后台运行
                                        ◀──fg──
```

#### jobs 输出解读

```bash
$ jobs
[1]+  Running    sleep 300 &
[2]-  Stopped    vim file.txt
```

- `[1]` —— job 编号，用 `%1` 引用
- `+` —— 当前默认 job（直接 `fg` 会恢复它）
- `-` —— 上一个 job

### 5. nohup 和 disown

当你关闭终端时，终端会向所有子进程发送 `SIGHUP`，导致进程终止。

#### nohup —— 启动时就忽略 SIGHUP

```bash
# output redirected to nohup.out by default
nohup long_task &

# redirect output explicitly
nohup long_task > output.log 2>&1 &
```

#### disown —— 事后将已运行的 job 脱离终端

```bash
# start a background task
long_task &

# remove it from shell's job table
disown %1

# disown all background jobs
disown -a
```

| 场景 | 推荐方法 |
|------|---------|
| 启动前就知道需要后台运行 | `nohup cmd &` |
| 已经运行了，临时需要脱离终端 | `Ctrl+Z` → `bg` → `disown` |
| 生产环境长期服务 | 使用 `systemd` service |

### 6. 进程优先级：nice / renice

Linux 进程的 nice 值范围为 **-20**（最高优先级）到 **19**（最低优先级），默认值为 **0**。

```bash
# start with lower priority (nice value 10)
nice -n 10 ./heavy_computation.sh

# start with higher priority (requires root)
sudo nice -n -5 ./important_task.sh

# change priority of running process
renice 15 -p 1234

# change priority for all processes of a user
sudo renice 10 -u guest
```

### 7. /proc 文件系统

`/proc` 是一个虚拟文件系统，内核通过它以文件形式暴露进程和系统信息。

```bash
# view process info (replace PID)
ls /proc/1234/

# process command line
cat /proc/1234/cmdline | tr '\0' ' '

# process environment variables
cat /proc/1234/environ | tr '\0' '\n'

# process status summary
cat /proc/1234/status

# memory map
cat /proc/1234/maps

# file descriptors
ls -l /proc/1234/fd/
```

系统级信息：

```bash
# CPU info
cat /proc/cpuinfo

# memory info
cat /proc/meminfo

# system uptime
cat /proc/uptime

# kernel version
cat /proc/version

# load average
cat /proc/loadavg
```

---

## 实战演练

### 步骤 1：查看进程

```bash
# list all processes
ps aux

# find shell processes
ps aux | grep bash

# show process tree
ps auxf | head -30
```

### 步骤 2：使用 top

```bash
# launch top, then:
#   press P to sort by CPU
#   press M to sort by memory
#   press 1 to see per-core CPU
#   press q to quit
top
```

### 步骤 3：后台作业实验

```bash
# start a long-running task in background
sleep 300 &

# check jobs
jobs

# start another task in foreground, then suspend
sleep 200
# press Ctrl+Z

# check jobs again
jobs

# resume the suspended job in background
bg %2

# bring first job to foreground
fg %1
# press Ctrl+C to stop it

# check remaining jobs
jobs
```

### 步骤 4：kill 实验

```bash
# start some test processes
sleep 1000 &
sleep 2000 &
sleep 3000 &

# list them
jobs
ps aux | grep "sleep [0-9]"

# kill by PID (replace with actual PID)
kill %1

# kill by name
killall sleep

# verify
jobs
```

### 步骤 5：nohup 实验

```bash
# start a process that survives terminal close
nohup bash -c 'for i in $(seq 1 10); do echo "tick $i"; sleep 2; done' > ~/tick.log 2>&1 &

# check it's running
jobs
cat ~/tick.log

# disown it
disown %1

# verify it's no longer in jobs list but still running
jobs
ps aux | grep tick

# clean up
kill %1 2>/dev/null; pkill -f "tick"; rm -f ~/tick.log ~/nohup.out
```

### 步骤 6：nice 实验

```bash
# start a low-priority process
nice -n 15 sleep 100 &

# check its priority in ps output (NI column)
ps -o pid,ni,comm -p $!

# change its priority
renice 5 -p $!
ps -o pid,ni,comm -p $!

# clean up
kill $!
```

### 步骤 7：探索 /proc

```bash
# get current shell's PID
echo $$

# explore its /proc entry
ls /proc/$$/
cat /proc/$$/status | head -10

# check system info
cat /proc/loadavg
cat /proc/meminfo | head -5
```

---

## 小结

| 命令 | 作用 |
|------|------|
| `ps aux` | 列出所有进程（BSD 格式） |
| `ps -ef` | 列出所有进程（POSIX 格式） |
| `top` / `htop` | 实时进程监控 |
| `kill PID` | 发送 SIGTERM 终止进程 |
| `kill -9 PID` | 强制终止进程 |
| `killall name` | 按名称终止进程 |
| `pkill pattern` | 按模式匹配终止进程 |
| `cmd &` | 后台运行命令 |
| `jobs` | 列出当前 shell 的后台作业 |
| `fg %n` / `bg %n` | 前台 / 后台恢复作业 |
| `Ctrl+Z` | 暂停前台进程 |
| `nohup cmd &` | 运行不受终端关闭影响的命令 |
| `disown %n` | 将作业脱离 shell 控制 |
| `nice -n N cmd` | 以指定优先级启动进程 |
| `renice N -p PID` | 修改运行中进程的优先级 |
| `/proc/PID/` | 查看进程的内核信息 |

---

## 练习

**1. 如何找到系统中占用 CPU 最高的 5 个进程？**

<details>
<summary>查看答案</summary>

```bash
ps aux --sort=-%cpu | head -6
```

`--sort=-%cpu` 按 CPU 使用率降序排列，`head -6` 取前 5 个（加上标题行）。

也可以用：

```bash
top -b -n 1 | head -12
```

</details>

**2. 如何优雅地终止一个叫 `node` 的进程？如果它不退出怎么办？**

<details>
<summary>查看答案</summary>

```bash
# Step 1: graceful termination
killall node

# Step 2: wait a few seconds, if still running, force kill
killall -9 node
```

先发 `SIGTERM` 让进程有机会做清理工作，如果超时未退出再用 `SIGKILL` 强制终止。

</details>

**3. 你在 SSH 远程服务器上启动了一个编译任务，突然需要断开 SSH，如何让任务继续运行？**

<details>
<summary>查看答案</summary>

如果任务已经在运行：

```bash
# suspend the foreground job
Ctrl+Z

# resume it in background
bg

# detach from shell
disown
```

如果还没开始，可以直接用：

```bash
nohup make -j4 > build.log 2>&1 &
```

</details>

**4. 如何查看 PID 为 5678 的进程的完整启动命令？**

<details>
<summary>查看答案</summary>

```bash
cat /proc/5678/cmdline | tr '\0' ' '
```

或者用 `ps`：

```bash
ps -p 5678 -o cmd
```

</details>

**5. 如何启动一个低优先级的备份脚本，使其不影响系统性能？**

<details>
<summary>查看答案</summary>

```bash
nice -n 19 ./backup.sh &
```

`-n 19` 设为最低优先级，`&` 放到后台执行。如果需要防止终端关闭导致中断：

```bash
nohup nice -n 19 ./backup.sh > backup.log 2>&1 &
```

</details>
