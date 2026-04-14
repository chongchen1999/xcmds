# 第 13 课：网络工具

> 上一课：[第 12 课：Process 管理](./12-process-management.md) | 下一课：[第 14 课：包管理器](./14-package-management.md)

---

## 学习目标

1. 掌握 `ping`、`curl`、`wget` 等常用网络诊断与数据传输工具
2. 学会使用 `ssh`、`scp`、`rsync` 进行远程登录与文件同步
3. 理解 `netstat` / `ss` 查看网络连接状态
4. 掌握 `ip` / `ifconfig` 查看与配置网络接口
5. 学会使用 `dig` / `nslookup` 进行 DNS 查询

---

## 知识讲解

### 1. 测试网络连通性：ping

`ping` 通过发送 ICMP Echo 请求来检测目标主机是否可达以及网络延迟。

```bash
# basic connectivity test
ping google.com

# send only 5 packets
ping -c 5 google.com

# set interval to 0.2 seconds (requires root)
sudo ping -i 0.2 google.com

# set packet size
ping -s 1024 -c 3 google.com
```

输出关键指标：

| 字段 | 含义 |
|------|------|
| `ttl` | Time To Live，经过的路由跳数 |
| `time` | 往返延迟（毫秒） |
| `packet loss` | 丢包率 |

> **提示**：如果 `ping` 不通，不一定代表主机离线——很多服务器会禁用 ICMP 响应。

### 2. HTTP 请求工具：curl

`curl`（**C**lient **URL**）是功能强大的命令行 HTTP 客户端。

#### 基本请求

```bash
# GET request
curl https://api.github.com

# show response headers
curl -I https://api.github.com

# show headers + body
curl -i https://api.github.com

# verbose output (debug)
curl -v https://api.github.com
```

#### POST 请求

```bash
# POST with JSON body
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name": "linux", "level": "intermediate"}'

# POST form data
curl -X POST https://httpbin.org/post \
  -d "username=admin&password=secret"
```

#### 自定义 Header

```bash
# add custom headers
curl -H "Authorization: Bearer TOKEN123" \
     -H "Accept: application/json" \
     https://api.example.com/data
```

#### 下载文件

```bash
# save with remote filename
curl -O https://example.com/file.tar.gz

# save with custom filename
curl -o myfile.tar.gz https://example.com/file.tar.gz

# resume interrupted download
curl -C - -O https://example.com/largefile.iso

# follow redirects
curl -L -O https://example.com/redirect-to-file

# silent mode (hide progress bar)
curl -sO https://example.com/file.tar.gz
```

#### 实用技巧

```bash
# download and pipe to bash (use with caution!)
curl -fsSL https://get.docker.com | bash

# test API response time
curl -o /dev/null -s -w "Total time: %{time_total}s\n" https://google.com

# upload a file
curl -F "file=@/path/to/document.pdf" https://example.com/upload
```

### 3. 下载工具：wget

`wget` 专注于文件下载，支持断点续传和递归下载。

```bash
# basic download
wget https://example.com/file.tar.gz

# save with custom filename
wget -O custom-name.tar.gz https://example.com/file.tar.gz

# resume interrupted download
wget -c https://example.com/largefile.iso

# download in background
wget -b https://example.com/largefile.iso

# recursive download (mirror a site)
wget -r -l 2 -np https://example.com/docs/

# limit download speed
wget --limit-rate=500k https://example.com/largefile.iso

# download with authentication
wget --user=admin --password=secret https://example.com/private/file.zip
```

`wget -r` 常用参数：

| 参数 | 作用 |
|------|------|
| `-r` | 递归下载 |
| `-l N` | 递归深度限制为 N 层 |
| `-np` | 不追溯到父目录 |
| `-k` | 转换链接为本地链接 |
| `-p` | 下载页面所需的所有资源 |

#### curl vs wget 对比

| 特性 | `curl` | `wget` |
|------|--------|--------|
| 协议支持 | 极广（HTTP, FTP, SMTP 等） | HTTP, HTTPS, FTP |
| 递归下载 | 不支持 | 支持 |
| 断点续传 | `-C -` | `-c` |
| 管道输出 | 默认输出到 stdout | 默认保存为文件 |
| API 测试 | 非常适合 | 不太适合 |

### 4. 远程登录：ssh

`ssh`（**S**ecure **Sh**ell）用于安全地远程登录服务器。

#### 基本连接

```bash
# connect to remote host
ssh user@192.168.1.100

# specify port
ssh -p 2222 user@hostname

# execute a single command remotely
ssh user@host "df -h && free -m"
```

#### SSH 密钥认证

密钥认证比密码更安全、更方便：

```bash
# generate SSH key pair
ssh-keygen -t ed25519 -C "your_email@example.com"

# copy public key to server
ssh-copy-id user@host

# now login without password
ssh user@host
```

#### SSH Config 文件

编辑 `~/.ssh/config` 简化连接：

```
Host myserver
    HostName 192.168.1.100
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host jump
    HostName 10.0.0.1
    User admin

Host internal
    HostName 10.0.0.50
    User developer
    ProxyJump jump
```

配置后可以直接使用别名连接：

```bash
# instead of: ssh -p 2222 deploy@192.168.1.100
ssh myserver

# connect through jump host
ssh internal
```

#### SSH 端口转发

```bash
# local port forwarding: access remote service through local port
ssh -L 8080:localhost:80 user@host

# remote port forwarding: expose local service to remote
ssh -R 9090:localhost:3000 user@host

# SOCKS proxy
ssh -D 1080 user@host
```

### 5. 远程文件复制：scp

```bash
# copy local file to remote
scp file.txt user@host:/home/user/

# copy remote file to local
scp user@host:/var/log/app.log ./

# copy directory recursively
scp -r ./project user@host:/opt/

# specify port
scp -P 2222 file.txt user@host:/tmp/

# copy between two remote hosts
scp user1@host1:/data/file.csv user2@host2:/backup/
```

### 6. 高效同步：rsync

`rsync` 只传输差异部分，比 `scp` 更高效。

```bash
# sync local directory to remote
rsync -avz ./project/ user@host:/opt/project/

# sync remote to local
rsync -avz user@host:/opt/project/ ./project/

# dry run (preview changes)
rsync -avzn ./project/ user@host:/opt/project/

# exclude files
rsync -avz --exclude='*.log' --exclude='node_modules' \
  ./project/ user@host:/opt/project/

# delete files on destination that don't exist in source
rsync -avz --delete ./project/ user@host:/opt/project/

# show progress
rsync -avz --progress ./largefile.iso user@host:/data/

# use custom SSH port
rsync -avz -e "ssh -p 2222" ./data/ user@host:/backup/
```

`rsync` 常用参数：

| 参数 | 作用 |
|------|------|
| `-a` | 归档模式（保留权限、时间戳等） |
| `-v` | 详细输出 |
| `-z` | 传输时压缩 |
| `-n` | 模拟运行（dry run） |
| `--delete` | 删除目标端多余的文件 |
| `--exclude` | 排除匹配的文件 |
| `--progress` | 显示传输进度 |

> **注意**：`rsync` 的源路径末尾加 `/` 表示同步目录内容；不加 `/` 表示同步整个目录。

### 7. 查看网络连接：netstat / ss

#### ss（推荐，替代 netstat）

```bash
# show all TCP connections
ss -t

# show all listening ports
ss -tlnp

# show all UDP sockets
ss -ulnp

# show all connections with process info
ss -tunap

# filter by port
ss -tlnp | grep :80

# show connection statistics
ss -s
```

#### netstat（较旧，部分系统已移除）

```bash
# show all listening ports with process info
netstat -tlnp

# show all connections
netstat -an

# show routing table
netstat -r
```

`ss` 参数速查：

| 参数 | 含义 |
|------|------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | 仅显示 listening 状态 |
| `-n` | 显示端口号而非服务名 |
| `-p` | 显示进程信息 |
| `-a` | 显示所有连接 |

### 8. 网络接口配置：ip / ifconfig

#### ip（现代工具，推荐）

```bash
# show all interfaces and IP addresses
ip addr show

# show specific interface
ip addr show eth0

# show routing table
ip route show

# show default gateway
ip route | grep default

# show link status
ip link show

# show ARP table
ip neigh show
```

#### ifconfig（传统工具）

```bash
# show all interfaces
ifconfig

# show specific interface
ifconfig eth0
```

> **推荐**：优先使用 `ip` 命令，`ifconfig` 已在很多新发行版中被弃用。

### 9. DNS 查询：dig / nslookup

#### dig（推荐）

```bash
# basic DNS lookup
dig example.com

# query specific record type
dig example.com A          # IPv4 address
dig example.com AAAA       # IPv6 address
dig example.com MX         # mail server
dig example.com NS         # name servers
dig example.com TXT        # TXT records
dig example.com CNAME      # canonical name

# short answer only
dig +short example.com

# use specific DNS server
dig @8.8.8.8 example.com

# reverse DNS lookup
dig -x 8.8.8.8

# trace DNS resolution path
dig +trace example.com
```

#### nslookup

```bash
# basic lookup
nslookup example.com

# use specific DNS server
nslookup example.com 8.8.8.8

# query MX records
nslookup -type=MX example.com
```

---

## 实战演练

### 步骤 1：测试网络连通性

```bash
# ping a well-known host
ping -c 4 google.com

# test local network
ping -c 2 192.168.1.1
```

### 步骤 2：使用 curl 进行 HTTP 请求

```bash
# GET request and pretty-print JSON
curl -s https://api.github.com | head -20

# check HTTP status code
curl -o /dev/null -s -w "%{http_code}\n" https://google.com

# POST JSON to a test API
curl -s -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"test": "hello"}' | head -20

# measure response time
curl -o /dev/null -s -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTotal: %{time_total}s\n" https://google.com
```

### 步骤 3：使用 wget 下载

```bash
# download a small test file
wget -O /tmp/test-robots.txt https://www.google.com/robots.txt

# check the file
cat /tmp/test-robots.txt | head -10

# clean up
rm /tmp/test-robots.txt
```

### 步骤 4：查看网络连接

```bash
# show listening TCP ports
ss -tlnp

# show all established connections
ss -tn state established

# show connection statistics
ss -s
```

### 步骤 5：查看网络接口

```bash
# show all interfaces
ip addr show

# show routing table
ip route show

# show DNS configuration
cat /etc/resolv.conf
```

### 步骤 6：DNS 查询

```bash
# look up a domain
dig +short google.com

# look up MX records
dig +short google.com MX

# reverse lookup
dig -x 8.8.8.8 +short

# full query with trace
dig +trace google.com | tail -20
```

### 步骤 7：SSH 配置（可选，需要远程服务器）

```bash
# generate key if you don't have one
ls ~/.ssh/id_ed25519 || ssh-keygen -t ed25519

# view your public key
cat ~/.ssh/id_ed25519.pub

# create or edit SSH config
cat ~/.ssh/config 2>/dev/null || echo "No SSH config yet"
```

---

## 小结

| 命令 | 作用 |
|------|------|
| `ping -c N host` | 发送 N 个 ICMP 包测试连通性 |
| `curl URL` | 发送 HTTP 请求 |
| `curl -O URL` | 下载文件（保留远程文件名） |
| `curl -X POST -d DATA` | 发送 POST 请求 |
| `wget URL` | 下载文件 |
| `wget -c URL` | 断点续传 |
| `wget -r URL` | 递归下载 |
| `ssh user@host` | 远程登录 |
| `ssh-keygen -t ed25519` | 生成 SSH 密钥对 |
| `scp src user@host:dst` | 远程复制文件 |
| `rsync -avz src dst` | 高效同步文件 |
| `ss -tlnp` | 查看 TCP 监听端口 |
| `ip addr show` | 查看网络接口与 IP |
| `ip route show` | 查看路由表 |
| `dig domain` | DNS 查询 |
| `dig +short domain` | DNS 简短查询 |

---

## 练习

**1. 如何用 `curl` 发送一个带有自定义 Header 的 POST 请求，并将响应保存到文件？**

<details>
<summary>查看答案</summary>

```bash
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer mytoken" \
  -d '{"key": "value"}' \
  -o response.json
```

`-H` 添加 Header，`-d` 指定请求体，`-o` 将响应保存到文件。

</details>

**2. 你需要把本地的 `~/project` 目录同步到远程服务器，但排除 `node_modules` 和 `.git` 目录，应该怎么做？**

<details>
<summary>查看答案</summary>

```bash
rsync -avz --exclude='node_modules' --exclude='.git' \
  ~/project/ user@host:/opt/project/
```

注意源路径末尾的 `/`：加了 `/` 表示同步目录内容（不会在远程创建 `project` 外层目录）。

</details>

**3. 如何查看本机 80 端口被哪个进程占用？**

<details>
<summary>查看答案</summary>

```bash
ss -tlnp | grep :80
```

或者：

```bash
sudo lsof -i :80
```

`ss -tlnp` 中的 `-p` 参数会显示占用端口的进程名和 PID。

</details>

**4. 如何配置 SSH 使得输入 `ssh prod` 就能连接到 `deploy@192.168.1.50:2222`？**

<details>
<summary>查看答案</summary>

在 `~/.ssh/config` 中添加：

```
Host prod
    HostName 192.168.1.50
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

之后直接执行 `ssh prod` 即可连接。

</details>

**5. 如何用 `dig` 查询 `example.com` 的邮件服务器记录，并只显示简短结果？**

<details>
<summary>查看答案</summary>

```bash
dig +short example.com MX
```

`MX` 指定查询邮件交换记录，`+short` 只显示结果值，不显示完整的 DNS 响应。

</details>
