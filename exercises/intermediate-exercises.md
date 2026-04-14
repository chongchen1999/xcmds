# 进阶篇综合练习

> 本练习集是进阶篇（第 09-15 课）的补充训练，侧重于**多命令组合的场景化问题**。
> 你将处理日志分析、文本处理、进程管理、网络调试等真实运维场景。
>
> 建议先完成各课的课后练习和[进阶综合测验](../intermediate/16-intermediate-quiz.md)后再来挑战。
>
> 📖 返回[课程目录](../README.md)

---

## 一、grep 与正则表达式

### 练习 1：Web 服务器日志分析

先创建模拟日志：

```bash
cat > /tmp/access.log << 'EOF'
192.168.1.10 - - [15/Jan/2026:10:00:01] "GET /index.html HTTP/1.1" 200 1024
192.168.1.15 - - [15/Jan/2026:10:00:02] "GET /style.css HTTP/1.1" 200 512
10.0.0.5 - - [15/Jan/2026:10:00:03] "POST /api/login HTTP/1.1" 401 128
192.168.1.10 - - [15/Jan/2026:10:00:04] "GET /dashboard HTTP/1.1" 200 2048
10.0.0.5 - - [15/Jan/2026:10:00:05] "POST /api/login HTTP/1.1" 401 128
10.0.0.5 - - [15/Jan/2026:10:00:06] "POST /api/login HTTP/1.1" 200 256
172.16.0.20 - - [15/Jan/2026:10:00:07] "GET /admin HTTP/1.1" 403 64
192.168.1.10 - - [15/Jan/2026:10:00:08] "GET /api/data HTTP/1.1" 500 32
192.168.1.15 - - [15/Jan/2026:10:00:09] "GET /images/logo.png HTTP/1.1" 200 4096
10.0.0.5 - - [15/Jan/2026:10:00:10] "DELETE /api/users/5 HTTP/1.1" 403 64
EOF
```

完成以下任务：

1. 找出所有失败的请求（状态码非 200）
2. 找出所有来自 `10.0.0.5` 的请求
3. 统计每个 IP 地址出现了多少次
4. 找出所有 POST 请求
5. 找出所有 4xx 错误（401、403 等）
6. 找出访问 `/api/` 路径的所有请求

<details>
<summary>查看答案</summary>

```bash
# Non-200 requests
grep -v '" 200 ' /tmp/access.log

# Requests from 10.0.0.5
grep "^10\.0\.0\.5" /tmp/access.log

# Count per IP
awk '{print $1}' /tmp/access.log | sort | uniq -c | sort -rn

# POST requests
grep '"POST ' /tmp/access.log

# 4xx errors
grep '" 4[0-9][0-9] ' /tmp/access.log

# Requests to /api/
grep '/api/' /tmp/access.log
```

</details>

### 练习 2：代码审查助手

先创建模拟源代码：

```bash
cat > /tmp/app.py << 'EOF'
import os
import sys
# TODO: add logging module
password = "hardcoded123"
DB_HOST = "localhost"
DB_PASSWORD = "admin123"

def connect_db():
    # TODO: use connection pool
    print("DEBUG: connecting to database")
    pass

def handle_request(req):
    print("DEBUG: processing request")
    eval(req.body)  # dangerous!
    pass

API_KEY = "sk-abc123secret"
# FIXME: remove before production
print("DEBUG: app started")
EOF
```

你需要做安全代码审查：

1. 找出所有 TODO 和 FIXME 注释
2. 找出所有硬编码的密码或密钥（包含 `password`、`secret`、`key` 等，忽略大小写）
3. 找出所有 `print("DEBUG` 语句，并显示行号
4. 找出潜在危险函数调用（`eval`、`exec`）
5. 统计文件中有多少行 TODO/FIXME

<details>
<summary>查看答案</summary>

```bash
grep -n "TODO\|FIXME" /tmp/app.py

grep -in "password\|secret\|key" /tmp/app.py

grep -n 'print("DEBUG' /tmp/app.py

grep -n "eval\|exec" /tmp/app.py

grep -c "TODO\|FIXME" /tmp/app.py
```

</details>

---

## 二、sed 文本处理

### 练习 3：批量修改配置文件

准备配置文件：

```bash
cat > /tmp/app.ini << 'EOF'
[server]
host = localhost
port = 8080
debug = true
log_level = DEBUG

[database]
host = localhost
port = 3306
name = myapp_dev
user = root
password = dev123

[cache]
host = localhost
port = 6379
ttl = 300
EOF
```

你需要把开发配置转换为生产配置：

1. 把所有 `localhost` 替换为 `prod-server.example.com`
2. 把 `debug = true` 改为 `debug = false`
3. 把 `log_level = DEBUG` 改为 `log_level = WARN`
4. 把数据库名从 `myapp_dev` 改为 `myapp_prod`
5. 把 `password = dev123` 改为 `password = CHANGE_ME`
6. 将以上修改保存到 `/tmp/app-prod.ini`

<details>
<summary>查看答案</summary>

```bash
sed -e 's/localhost/prod-server.example.com/g' \
    -e 's/debug = true/debug = false/' \
    -e 's/log_level = DEBUG/log_level = WARN/' \
    -e 's/myapp_dev/myapp_prod/' \
    -e 's/password = dev123/password = CHANGE_ME/' \
    /tmp/app.ini > /tmp/app-prod.ini

cat /tmp/app-prod.ini
```

</details>

### 练习 4：清洗数据文件

准备脏数据：

```bash
cat > /tmp/raw-data.csv << 'EOF'
# This is a header comment
# Generated on 2026-01-15
Name,  Email,   Phone
Alice,  alice@example.com,  555-0101
Bob,bob@test.com,   555-0102

Charlie,  charlie@example.com,555-0103
  Diana,diana@test.com,  555-0104

Eve,  eve@example.com,  555-0105
EOF
```

你需要清洗这个数据文件：

1. 删除所有以 `#` 开头的注释行
2. 删除所有空行
3. 去掉逗号后面的多余空格
4. 去掉行首的多余空格
5. 将结果保存到 `/tmp/clean-data.csv`

<details>
<summary>查看答案</summary>

```bash
sed -e '/^#/d' \
    -e '/^$/d' \
    -e 's/,  */,/g' \
    -e 's/^  *//' \
    /tmp/raw-data.csv > /tmp/clean-data.csv

cat /tmp/clean-data.csv
```

预期输出：
```
Name,Email,Phone
Alice,alice@example.com,555-0101
Bob,bob@test.com,555-0102
Charlie,charlie@example.com,555-0103
Diana,diana@test.com,555-0104
Eve,eve@example.com,555-0105
```

</details>

---

## 三、awk 数据分析

### 练习 5：服务器资源报告

准备数据：

```bash
cat > /tmp/servers.txt << 'EOF'
hostname cpu_usage mem_usage disk_usage status
web-01 45 62 70 running
web-02 78 85 65 running
db-01 92 90 88 warning
db-02 30 45 50 running
cache-01 15 80 30 running
app-01 88 75 60 running
app-02 95 92 95 critical
monitor 10 20 15 running
EOF
```

用 `awk` 完成以下分析：

1. 只显示主机名和状态（第 1 和第 5 列）
2. 找出 CPU 使用率超过 80% 的服务器
3. 计算所有服务器的平均 CPU 使用率
4. 找出状态为 `warning` 或 `critical` 的服务器
5. 找出任意资源使用率（cpu/mem/disk）超过 90 的服务器
6. 按内存使用率从高到低排序输出

<details>
<summary>查看答案</summary>

```bash
# Show hostname and status
awk 'NR>1 {print $1, $5}' /tmp/servers.txt

# CPU > 80%
awk 'NR>1 && $2 > 80 {print $1, "CPU:"$2"%"}' /tmp/servers.txt

# Average CPU
awk 'NR>1 {sum+=$2; count++} END {printf "Average CPU: %.1f%%\n", sum/count}' /tmp/servers.txt

# Warning or critical
awk '$5 == "warning" || $5 == "critical"' /tmp/servers.txt

# Any resource > 90
awk 'NR>1 && ($2>90 || $3>90 || $4>90) {print $1, "cpu:"$2, "mem:"$3, "disk:"$4}' /tmp/servers.txt

# Sort by memory usage descending
awk 'NR>1' /tmp/servers.txt | sort -k3 -rn
```

</details>

### 练习 6：销售数据汇总

准备数据：

```bash
cat > /tmp/sales.csv << 'EOF'
date,product,quantity,price
2026-01-01,Widget-A,10,29.99
2026-01-01,Widget-B,5,49.99
2026-01-02,Widget-A,8,29.99
2026-01-02,Widget-C,3,99.99
2026-01-03,Widget-B,12,49.99
2026-01-03,Widget-A,15,29.99
2026-01-03,Widget-C,7,99.99
EOF
```

1. 计算每行的销售额（数量 × 单价），附加为新列输出
2. 计算总销售额
3. 按产品分组统计各产品的总销售额
4. 找出单笔销售额最高的记录

<details>
<summary>查看答案</summary>

```bash
# Add revenue column
awk -F',' 'NR==1 {print $0",revenue"} NR>1 {printf "%s,%.2f\n", $0, $3*$4}' /tmp/sales.csv

# Total revenue
awk -F',' 'NR>1 {total += $3*$4} END {printf "Total: $%.2f\n", total}' /tmp/sales.csv

# Revenue by product
awk -F',' 'NR>1 {rev[$2] += $3*$4} END {for (p in rev) printf "%s: $%.2f\n", p, rev[p]}' /tmp/sales.csv

# Highest single sale
awk -F',' 'NR>1 {rev=$3*$4; if(rev>max){max=rev; line=$0}} END {print line, "-> $"max}' /tmp/sales.csv
```

</details>

---

## 四、进程管理

### 练习 7：排查失控进程

模拟场景：系统变得很慢，你需要找出原因。

1. 查看当前系统中 CPU 占用最高的 5 个进程
2. 查看当前系统中内存占用最高的 5 个进程
3. 查看当前用户运行的所有进程
4. 在后台启动一个模拟进程：`sleep 3600 &`
5. 查看这个后台进程的 PID
6. 用 `kill` 优雅地终止它
7. 确认进程已经被终止

<details>
<summary>查看答案</summary>

```bash
ps aux --sort=-%cpu | head -6

ps aux --sort=-%mem | head -6

ps ux

sleep 3600 &

jobs -l
# Or
ps aux | grep "sleep 3600"

kill %1
# Or use the PID: kill <PID>

jobs
# Or
ps aux | grep "sleep 3600"
```

</details>

### 练习 8：后台任务管理

完成一套完整的后台任务管理操作：

1. 启动一个前台命令 `sleep 600`
2. 将它暂停（挂起）——写出按键
3. 查看后台任务列表
4. 将它恢复为后台运行
5. 再启动另一个后台任务 `sleep 601 &`
6. 查看所有后台任务的状态
7. 终止所有后台任务

<details>
<summary>查看答案</summary>

```bash
sleep 600
# Press Ctrl+Z to suspend

jobs

bg %1

sleep 601 &

jobs

kill %1 %2
# Verify
jobs
```

</details>

---

## 五、网络工具

### 练习 9：API 接口测试

使用 `curl` 测试一个公开 API（httpbin.org）：

1. 发送一个 GET 请求到 `https://httpbin.org/get`，只查看 HTTP 响应头
2. 发送一个 GET 请求并只显示 HTTP 状态码
3. 发送一个 POST 请求到 `https://httpbin.org/post`，body 为 `{"name":"test"}`
4. 发送请求并将响应保存到文件
5. 下载 `https://httpbin.org/image/png` 并保存为 `/tmp/test-image.png`

<details>
<summary>查看答案</summary>

```bash
curl -I https://httpbin.org/get

curl -s -o /dev/null -w "%{http_code}" https://httpbin.org/get

curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}'

curl -s https://httpbin.org/get > /tmp/api-response.json

curl -s -o /tmp/test-image.png https://httpbin.org/image/png
```

</details>

### 练习 10：网络连通性排查

你发现应用无法连接某个服务，需要逐步排查：

1. 检查能否 ping 通 `8.8.8.8`（只发 3 个包）
2. 检查 DNS 是否正常：`ping google.com`（只发 2 个包）
3. 查看本机的网络接口和 IP 地址
4. 查看当前系统的 DNS 配置
5. 查看当前系统中所有监听的端口
6. 检查某个远程端口是否可达（如 `google.com:443`）

<details>
<summary>查看答案</summary>

```bash
ping -c 3 8.8.8.8

ping -c 2 google.com

ip addr
# Or on macOS:
ifconfig

cat /etc/resolv.conf

# Linux:
ss -tlnp
# Or:
netstat -tlnp

# Check remote port
curl -s -o /dev/null -w "%{http_code}" https://google.com
# Or use nc (netcat):
nc -zv google.com 443
```

</details>

---

## 六、包管理与环境配置

### 练习 11：新机器初始化

你拿到一台全新的 Ubuntu 服务器，需要安装开发环境：

1. 先更新包索引
2. 安装 `git`、`curl`、`vim`（一条命令）
3. 验证 `git` 安装成功（查看版本）
4. 查看 `git` 包的详细信息
5. 搜索是否有 `nodejs` 相关的包可用
6. 列出系统中已安装的包中包含 "python" 的包

<details>
<summary>查看答案</summary>

```bash
sudo apt update

sudo apt install -y git curl vim

git --version

apt show git

apt search nodejs

apt list --installed 2>/dev/null | grep python
```

如果是 CentOS/RHEL 系统，对应命令：
```bash
sudo yum update
sudo yum install -y git curl vim
yum info git
yum search nodejs
yum list installed | grep python
```

</details>

### 练习 12：环境变量配置

你需要为开发环境设置环境变量：

1. 查看当前所有环境变量中与 `HOME` 相关的变量
2. 查看当前的 `PATH` 变量值，并用换行符分隔每个路径方便阅读
3. 临时设置一个变量 `APP_ENV=development` 并验证
4. 将 `/opt/myapp/bin` 添加到 `PATH`（临时）
5. 验证新路径已添加
6. 写出如何将这些设置永久化（写入哪个文件，用什么命令）

<details>
<summary>查看答案</summary>

```bash
env | grep HOME

echo $PATH | tr ':' '\n'

export APP_ENV=development
echo $APP_ENV

export PATH=$PATH:/opt/myapp/bin

echo $PATH | tr ':' '\n' | grep myapp
```

永久化设置——追加到 `~/.bashrc`（bash）或 `~/.zshrc`（zsh）：
```bash
echo 'export APP_ENV=development' >> ~/.bashrc
echo 'export PATH=$PATH:/opt/myapp/bin' >> ~/.bashrc
source ~/.bashrc
```

</details>

---

## 七、综合实战

### 练习 13：日志分析报告

这是一个综合练习，需要组合 grep、sed、awk、sort、pipe 完成一个完整的日志分析任务。

准备日志：

```bash
cat > /tmp/webapp.log << 'EOF'
2026-01-15 08:00:01 [INFO] Server started on port 8080
2026-01-15 08:01:15 [INFO] User alice logged in from 192.168.1.10
2026-01-15 08:05:32 [WARN] Slow query detected: 2500ms
2026-01-15 08:10:44 [INFO] User bob logged in from 10.0.0.5
2026-01-15 08:15:00 [ERROR] Database connection timeout
2026-01-15 08:15:01 [ERROR] Failed to process request: DB unavailable
2026-01-15 08:15:05 [INFO] Retrying database connection
2026-01-15 08:15:06 [INFO] Database reconnected
2026-01-15 08:20:00 [INFO] User charlie logged in from 192.168.1.10
2026-01-15 08:25:30 [WARN] Memory usage above 80%: 82%
2026-01-15 08:30:00 [INFO] User alice logged out
2026-01-15 08:35:15 [ERROR] OutOfMemoryError in worker thread
2026-01-15 08:35:16 [ERROR] Worker thread crashed, restarting
2026-01-15 08:40:00 [INFO] User bob logged out
2026-01-15 09:00:00 [INFO] Scheduled cleanup completed
EOF
```

完成以下分析任务：

1. 统计各日志级别（INFO/WARN/ERROR）的出现次数
2. 提取所有登录的用户名和 IP 地址
3. 找出所有 ERROR 级别日志的时间和消息内容
4. 找出从 `192.168.1.10` 登录的所有用户
5. 生成一份简要报告，保存到 `/tmp/log-report.txt`，包含：错误数、警告数、登录用户列表

<details>
<summary>查看答案</summary>

```bash
# Count by level
grep -oP '\[(INFO|WARN|ERROR)\]' /tmp/webapp.log | sort | uniq -c | sort -rn

# Extract login users and IPs
grep "logged in" /tmp/webapp.log | awk '{print $5, "from", $NF}'

# ERROR details
grep '\[ERROR\]' /tmp/webapp.log | awk '{print $2, substr($0, index($0, "[ERROR]") + 8)}'

# Users from 192.168.1.10
grep "logged in from 192.168.1.10" /tmp/webapp.log | awk '{print $5}'

# Generate report
{
  echo "=== Log Analysis Report ==="
  echo ""
  echo "Error count: $(grep -c '\[ERROR\]' /tmp/webapp.log)"
  echo "Warning count: $(grep -c '\[WARN\]' /tmp/webapp.log)"
  echo ""
  echo "Logged in users:"
  grep "logged in" /tmp/webapp.log | awk '{print "  - " $5 " (from " $NF ")"}'
} > /tmp/log-report.txt

cat /tmp/log-report.txt
```

</details>

### 练习 14：进程与资源监控

综合运用进程管理和文本处理工具：

1. 启动 3 个后台进程：
   ```bash
   sleep 1000 &
   sleep 1001 &
   sleep 1002 &
   ```
2. 用 `ps` 列出这 3 个 sleep 进程并提取它们的 PID
3. 将这些 PID 保存到 `/tmp/pids.txt`（每行一个）
4. 用一条命令终止所有这些进程（提示：结合 `xargs`）
5. 验证所有进程已终止
6. 查看当前系统负载（`uptime`）并用 `awk` 提取 load average 值

<details>
<summary>查看答案</summary>

```bash
sleep 1000 &
sleep 1001 &
sleep 1002 &

ps aux | grep "sleep 100[0-2]"

ps aux | grep "sleep 100[0-2]" | awk '{print $2}' > /tmp/pids.txt
cat /tmp/pids.txt

cat /tmp/pids.txt | xargs kill
# Or:
xargs kill < /tmp/pids.txt

ps aux | grep "sleep 100[0-2]"

uptime | awk -F'load average: ' '{print $2}'
```

</details>

### 练习 15：配置文件大改造

你需要对多个配置文件做批量修改，综合运用 sed、grep、pipe：

准备文件：

```bash
mkdir -p /tmp/configs
for svc in web api worker; do
cat > /tmp/configs/${svc}.conf << EOF
service_name = ${svc}
port = 8080
environment = development
log_level = DEBUG
max_connections = 100
timeout = 30
enable_ssl = false
ssl_cert = /etc/ssl/dev-cert.pem
EOF
done
```

任务：

1. 查看所有配置文件中当前的 `environment` 设置
2. 将所有文件的 `environment` 从 `development` 改为 `production`
3. 将所有文件的 `log_level` 从 `DEBUG` 改为 `ERROR`
4. 将所有文件的 `enable_ssl` 改为 `true`
5. 将所有文件的 `ssl_cert` 路径改为 `/etc/ssl/prod-cert.pem`
6. 给 `web.conf` 的 `port` 改为 `443`，给 `api.conf` 改为 `8443`
7. 验证所有修改

<details>
<summary>查看答案</summary>

```bash
grep "environment" /tmp/configs/*.conf

sed -i 's/environment = development/environment = production/' /tmp/configs/*.conf

sed -i 's/log_level = DEBUG/log_level = ERROR/' /tmp/configs/*.conf

sed -i 's/enable_ssl = false/enable_ssl = true/' /tmp/configs/*.conf

sed -i 's|ssl_cert = /etc/ssl/dev-cert.pem|ssl_cert = /etc/ssl/prod-cert.pem|' /tmp/configs/*.conf

sed -i 's/port = 8080/port = 443/' /tmp/configs/web.conf
sed -i 's/port = 8080/port = 8443/' /tmp/configs/api.conf

echo "=== web.conf ===" && cat /tmp/configs/web.conf
echo "=== api.conf ===" && cat /tmp/configs/api.conf
echo "=== worker.conf ===" && cat /tmp/configs/worker.conf
```

注意：macOS 上的 `sed -i` 需要加空字符串参数：`sed -i '' 's/old/new/' file`

</details>

---

**完成所有练习了？太棒了！** 🎉

继续挑战 → [高级篇综合练习](advanced-exercises.md)
