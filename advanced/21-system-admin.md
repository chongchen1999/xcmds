# 第 21 课：系统管理

> 上一课：[Cron Job 与自动化](20-cron-and-automation.md) | 下一课：[磁盘与存储管理](22-disk-and-storage.md)

## 学习目标

- 掌握 Linux 用户与组的管理方法
- 理解 `/etc/passwd`、`/etc/shadow`、`/etc/group` 的结构
- 熟练使用 `sudo` 及 `visudo` 配置权限
- 掌握 `systemctl` 管理 systemd 服务
- 了解 systemd unit 文件的基本结构
- 学会使用 `journalctl` 和系统日志排查问题
- 熟悉常用系统信息查询命令

---

## 知识讲解

### 1. 用户管理

Linux 是多用户系统，每个用户有唯一的 UID（User ID）。

#### 创建用户

```bash
# Create user with home directory and default shell
sudo useradd -m -s /bin/bash alice

# Create user with specific UID and group
sudo useradd -m -u 1500 -g developers -s /bin/bash bob

# Create system user (no home, no login)
sudo useradd -r -s /usr/sbin/nologin appuser
```

#### 修改用户

```bash
# Change default shell
sudo usermod -s /bin/zsh alice

# Add user to supplementary group (append, don't replace)
sudo usermod -aG docker alice
sudo usermod -aG sudo alice

# Lock / unlock account
sudo usermod -L alice
sudo usermod -U alice

# Change username
sudo usermod -l alice_new alice
```

#### 删除用户

```bash
# Delete user only
sudo userdel bob

# Delete user and home directory
sudo userdel -r bob
```

#### 密码管理

```bash
# Set password interactively
sudo passwd alice

# Force password change on next login
sudo passwd -e alice

# View password aging info
sudo chage -l alice

# Set password to expire in 90 days
sudo chage -M 90 alice
```

#### 查看用户信息

```bash
# Current user
whoami
id

# Specific user
id alice
groups alice

# All logged-in users
who
w
```

### 2. 关键配置文件

#### /etc/passwd

每行一个用户，7 个字段以 `:` 分隔：

```
username:x:UID:GID:comment:home_dir:shell
```

```bash
# View a user's entry
grep alice /etc/passwd
# alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
```

| 字段 | 说明 |
|------|------|
| `username` | 用户名 |
| `x` | 密码占位符（实际存于 shadow） |
| `UID` | 用户 ID（0=root，1-999=系统，1000+=普通） |
| `GID` | 主组 ID |
| `comment` | 注释（通常为全名） |
| `home_dir` | 家目录路径 |
| `shell` | 登录 Shell |

#### /etc/shadow

存储加密密码和密码策略，仅 root 可读：

```
username:$encrypted_password:lastchange:min:max:warn:inactive:expire:
```

```bash
sudo cat /etc/shadow | grep alice
# alice:$6$xyz...:19500:0:99999:7:::
```

#### /etc/group

```
groupname:x:GID:member_list
```

```bash
# Create and manage groups
sudo groupadd developers
sudo groupdel developers
sudo gpasswd -a alice developers   # Add member
sudo gpasswd -d alice developers   # Remove member
```

### 3. sudo 与权限提升

`sudo` 允许授权用户以其他用户（通常是 root）身份执行命令。

#### 基本用法

```bash
# Run single command as root
sudo apt update

# Run as specific user
sudo -u postgres psql

# Open root shell
sudo -i

# Preserve environment
sudo -E command

# Edit file with sudo
sudo -e /etc/hosts   # uses $EDITOR
```

#### /etc/sudoers 与 visudo

**永远使用 `visudo` 编辑 sudoers 文件**——它会在保存前做语法检查，避免把自己锁在系统外面。

```bash
# Edit main sudoers file
sudo visudo

# Edit drop-in file (recommended)
sudo visudo -f /etc/sudoers.d/developers
```

sudoers 语法：

```
# Format: who where=(as_whom) what
# Allow alice to run all commands as any user
alice ALL=(ALL:ALL) ALL

# Allow developers group to restart nginx without password
%developers ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx

# Allow deploy user to run specific commands
deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp, \
                            /usr/bin/systemctl status myapp
```

```bash
# Check your sudo privileges
sudo -l
```

### 4. 服务管理：systemctl

`systemctl` 是管理 systemd 服务的核心命令。

#### 常用操作

```bash
# Start / stop / restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload config without restarting
sudo systemctl reload nginx

# Enable / disable at boot
sudo systemctl enable nginx
sudo systemctl disable nginx

# Enable and start in one step
sudo systemctl enable --now nginx

# Check status
systemctl status nginx

# Check if active/enabled
systemctl is-active nginx
systemctl is-enabled nginx
```

#### 查看服务列表

```bash
# All loaded services
systemctl list-units --type=service

# All enabled services
systemctl list-unit-files --type=service --state=enabled

# Failed services
systemctl --failed
```

#### 服务依赖

```bash
# Show dependencies
systemctl list-dependencies nginx

# Show reverse dependencies
systemctl list-dependencies --reverse nginx
```

### 5. systemd Unit 文件基础

Unit 文件通常位于 `/etc/systemd/system/`（自定义）或 `/lib/systemd/system/`（包管理器）。

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application Server
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server --config /etc/myapp/config.yaml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

| 段落 | 关键字段 | 说明 |
|------|----------|------|
| `[Unit]` | `After` | 在指定服务之后启动 |
| | `Wants` | 软依赖（依赖失败不影响自身） |
| | `Requires` | 硬依赖（依赖失败则自身也失败） |
| `[Service]` | `Type` | `simple`（默认）、`forking`、`oneshot` |
| | `ExecStart` | 启动命令 |
| | `Restart` | 重启策略：`on-failure`、`always`、`no` |
| `[Install]` | `WantedBy` | `enable` 时的目标 |

```bash
# After editing unit files
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

### 6. 日志管理

#### journalctl

`journalctl` 查询 systemd 日志（journal）：

```bash
# View all logs (newest last)
journalctl

# Follow new entries (like tail -f)
journalctl -f

# Logs for a specific service
journalctl -u nginx
journalctl -u nginx --since "1 hour ago"

# Logs since boot
journalctl -b

# Previous boot
journalctl -b -1

# Time range
journalctl --since "2026-04-14 08:00:00" --until "2026-04-14 12:00:00"

# Filter by priority (0=emerg to 7=debug)
journalctl -p err
journalctl -p warning..err

# Show kernel messages only
journalctl -k

# Output as JSON
journalctl -u nginx -o json-pretty --no-pager -n 5

# Disk usage of journal
journalctl --disk-usage

# Clean old journal entries
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=500M
```

#### 传统日志文件

| 文件 | 内容 |
|------|------|
| `/var/log/syslog` | 通用系统日志（Debian/Ubuntu） |
| `/var/log/messages` | 通用系统日志（RHEL/CentOS） |
| `/var/log/auth.log` | 认证日志（登录、sudo） |
| `/var/log/kern.log` | 内核日志 |
| `/var/log/dmesg` | 启动时内核消息 |
| `/var/log/apt/` | APT 包管理日志 |
| `/var/log/nginx/` | Nginx 访问/错误日志 |

```bash
# Kernel ring buffer
dmesg
dmesg | tail -20
dmesg -T   # Human-readable timestamps
dmesg -l err,warn
```

### 7. 系统信息

```bash
# Kernel and OS info
uname -a          # All info
uname -r          # Kernel version
uname -m          # Architecture

# Hostname
hostnamectl
sudo hostnamectl set-hostname myserver

# Distribution info
cat /etc/os-release
lsb_release -a    # If available

# Uptime and load
uptime
# Output: 14:23:05 up 45 days,  3:12,  2 users,  load average: 0.52, 0.38, 0.41

# Hardware info
lscpu             # CPU details
lsmem             # Memory layout
lsblk             # Block devices
lspci             # PCI devices
lsusb             # USB devices
```

`load average` 的三个数字分别是 1 分钟、5 分钟、15 分钟的平均负载。作为参考，值应低于 CPU 核心数：

```bash
# Check number of CPU cores
nproc
# If load average consistently exceeds nproc, system is overloaded
```

---

## 实战演练

### 练习 1：用户和组管理

```bash
# Create a group
sudo groupadd webteam

# Create user with webteam as supplementary group
sudo useradd -m -s /bin/bash -G webteam webdev1

# Verify
id webdev1
grep webdev1 /etc/passwd
grep webteam /etc/group

# Add existing user to group
sudo usermod -aG webteam $(whoami)

# Verify group membership
groups $(whoami)

# Clean up
sudo userdel -r webdev1
sudo groupdel webteam
```

### 练习 2：配置 sudo 权限

```bash
# Create a drop-in sudoers file
sudo visudo -f /etc/sudoers.d/webteam

# Add this line:
# %webteam ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx

# Verify syntax
sudo visudo -c

# Check effective permissions
sudo -l -U webdev1

# Clean up
sudo rm /etc/sudoers.d/webteam
```

### 练习 3：服务管理

```bash
# List all running services
systemctl list-units --type=service --state=running

# Check a service status
systemctl status sshd

# View service's unit file
systemctl cat sshd

# View service properties
systemctl show sshd --property=MainPID,ActiveState,SubState

# Check for failed services
systemctl --failed
```

### 练习 4：日志查询

```bash
# View recent system errors
journalctl -p err --since "today"

# Follow SSH logs
journalctl -u sshd -f

# Find failed login attempts
journalctl -u sshd | grep "Failed password"

# Check kernel messages since boot
journalctl -k -b

# View journal disk usage
journalctl --disk-usage
```

### 练习 5：系统信息快速报告

```bash
echo "=== System Info ==="
uname -a
echo ""
echo "=== Hostname ==="
hostnamectl
echo ""
echo "=== Uptime ==="
uptime
echo ""
echo "=== CPU ==="
nproc
lscpu | grep "Model name"
echo ""
echo "=== Memory ==="
free -h
echo ""
echo "=== OS Release ==="
cat /etc/os-release | head -4
```

---

## 小结

| 概念 | 命令 | 说明 |
|------|------|------|
| 创建用户 | `useradd -m -s /bin/bash user` | 创建用户并指定 Shell |
| 修改用户 | `usermod -aG group user` | 追加用户到组 |
| 删除用户 | `userdel -r user` | 删除用户及家目录 |
| 设置密码 | `passwd user` | 交互式设置密码 |
| 编辑 sudoers | `visudo` | 安全编辑 sudo 配置 |
| 启动服务 | `systemctl start svc` | 启动 systemd 服务 |
| 开机自启 | `systemctl enable svc` | 设置开机自动启动 |
| 查看服务状态 | `systemctl status svc` | 查看运行状态和最近日志 |
| 查看日志 | `journalctl -u svc` | 查看指定服务的日志 |
| 实时日志 | `journalctl -f` | 实时跟踪新日志 |
| 内核消息 | `dmesg -T` | 带时间戳查看内核日志 |
| 系统信息 | `uname -a` | 内核和系统信息 |
| 主机信息 | `hostnamectl` | 主机名和系统详情 |
| 运行时间 | `uptime` | 运行时间和负载 |

---

## 练习

**1. 解释 `/etc/passwd` 中这行的含义：`www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin`**

<details>
<summary>查看答案</summary>

- 用户名：`www-data`
- `x`：密码存储在 `/etc/shadow`
- UID：33
- GID：33
- 注释字段：`www-data`
- 家目录：`/var/www`
- Shell：`/usr/sbin/nologin`（禁止登录）

这是一个系统用户（UID < 1000），通常用于运行 Web 服务器（如 Nginx、Apache），`nologin` 确保该用户无法通过 SSH 或终端登录。

</details>

**2. 你需要让用户 `deploy` 能够无密码重启 `myapp` 服务，但不能做其他任何 sudo 操作。写出 sudoers 配置。**

<details>
<summary>查看答案</summary>

使用 `visudo -f /etc/sudoers.d/deploy` 创建配置文件：

```
deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp
```

说明：
- `deploy`：用户名
- `ALL`：适用于所有主机
- `(root)`：以 root 身份执行
- `NOPASSWD`：无需输入密码
- 仅限 `systemctl restart myapp` 这一条命令

如果还需要查看状态：

```
deploy ALL=(root) NOPASSWD: /usr/bin/systemctl restart myapp, \
                            /usr/bin/systemctl status myapp
```

</details>

**3. 一个服务启动失败，请写出排查步骤。**

<details>
<summary>查看答案</summary>

```bash
# 1. Check service status and error message
systemctl status myapp

# 2. View detailed logs
journalctl -u myapp --no-pager -n 50

# 3. Verify unit file syntax
systemd-analyze verify /etc/systemd/system/myapp.service

# 4. Check the executable path and permissions
ls -la /opt/myapp/bin/server
file /opt/myapp/bin/server

# 5. Try running the command manually as the service user
sudo -u appuser /opt/myapp/bin/server --config /etc/myapp/config.yaml

# 6. Check for port conflicts
sudo ss -tlnp | grep :8080

# 7. Check SELinux or AppArmor if applicable
sudo ausearch -m AVC -ts recent   # SELinux
sudo journalctl | grep apparmor   # AppArmor
```

</details>

**4. 写一条 `journalctl` 命令，查看今天所有 `sshd` 服务中优先级为 warning 或更高的日志。**

<details>
<summary>查看答案</summary>

```bash
journalctl -u sshd -p warning --since "today"
```

说明：
- `-u sshd`：筛选 sshd 服务
- `-p warning`：优先级 warning 及以上（包括 err、crit、alert、emerg）
- `--since "today"`：从今天 0 点开始

</details>

**5. `systemctl enable` 和 `systemctl start` 有什么区别？`enable --now` 又做了什么？**

<details>
<summary>查看答案</summary>

- `systemctl start svc`：**立即启动**服务，但重启后不会自动运行。
- `systemctl enable svc`：创建 symlink 使服务**开机自启**，但不会立即启动。
- `systemctl enable --now svc`：同时做两件事——设置开机自启 **并** 立即启动。

技术细节：`enable` 实际上是在 `/etc/systemd/system/multi-user.target.wants/` 下创建了指向 unit 文件的符号链接，系统启动到 `multi-user.target` 时会自动启动所有该目录下的服务。

</details>
