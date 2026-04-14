# 入门篇综合练习

> 本练习集是入门篇（第 01-07 课）的补充训练，侧重于**场景化的实战问题**，需要你综合运用多个命令来完成任务。
>
> 建议先完成各课的课后练习和[入门综合测验](../beginner/08-beginner-quiz.md)后再来挑战。
>
> 📖 返回[课程目录](../README.md)

---

## 一、文件系统导航

### 练习 1：迷失在文件系统中

你刚打开终端，不知道自己在哪里。你需要：

1. 确认当前所在目录
2. 列出当前目录下的所有文件（包括隐藏文件）
3. 切换到 `/var/log` 目录
4. 用一条命令回到你刚才的目录（home 目录）
5. 确认你已经回到了 home 目录

<details>
<summary>查看答案</summary>

```bash
pwd
ls -la
cd /var/log
cd ~
pwd
```

也可以用 `cd -` 回到上一个目录，或 `cd` 不带参数直接回 home。

</details>

### 练习 2：探索项目结构

你拿到一个开源项目，需要快速了解它的目录结构：

1. 进入 `/tmp` 目录
2. 用一条命令创建这样的目录结构：`myproject/src`、`myproject/docs`、`myproject/tests`
3. 用 `ls` 递归查看 `myproject` 的完整树状结构
4. 进入 `myproject/src` 目录
5. 用相对路径列出 `docs` 目录的内容

<details>
<summary>查看答案</summary>

```bash
cd /tmp
mkdir -p myproject/{src,docs,tests}
ls -R myproject
cd myproject/src
ls ../docs
```

</details>

### 练习 3：路径高手

不切换目录，完成以下任务（假设你当前在 home 目录）：

1. 列出 `/etc` 目录下以 `.conf` 结尾的文件
2. 查看 `/usr/bin` 目录中文件的数量
3. 列出 `/tmp` 下最近修改的 5 个文件（按时间排序）

<details>
<summary>查看答案</summary>

```bash
ls /etc/*.conf
ls /usr/bin | wc -l
ls -lt /tmp | head -6
```

`head -6` 是因为 `ls -lt` 的第一行是 `total` 行，实际文件从第二行开始。也可以用 `ls -lt /tmp | tail -n +2 | head -5`。

</details>

---

## 二、文件与目录操作

### 练习 4：整理杂乱的下载目录

你的下载目录一片混乱，里面有各种文件。请模拟并整理：

1. 在 `/tmp` 下创建 `messy-downloads` 目录
2. 在其中创建以下空文件：`report.pdf`、`photo1.jpg`、`photo2.jpg`、`notes.txt`、`song.mp3`、`backup.tar.gz`、`readme.md`
3. 创建子目录：`documents`、`images`、`music`、`archives`
4. 把 `.pdf` 和 `.txt` 和 `.md` 文件移到 `documents`
5. 把 `.jpg` 文件移到 `images`
6. 把 `.mp3` 文件移到 `music`
7. 把 `.tar.gz` 文件移到 `archives`
8. 验证每个子目录中的文件

<details>
<summary>查看答案</summary>

```bash
mkdir /tmp/messy-downloads
cd /tmp/messy-downloads
touch report.pdf photo1.jpg photo2.jpg notes.txt song.mp3 backup.tar.gz readme.md
mkdir documents images music archives
mv report.pdf notes.txt readme.md documents/
mv photo1.jpg photo2.jpg images/
mv song.mp3 music/
mv backup.tar.gz archives/
ls documents images music archives
```

也可以用通配符：`mv *.jpg images/`

</details>

### 练习 5：项目模板生成器

你经常需要创建类似结构的项目。请用命令完成：

1. 在 `/tmp` 下创建 `webapp` 项目目录
2. 一次性创建以下结构：
   - `webapp/frontend/css/`
   - `webapp/frontend/js/`
   - `webapp/frontend/images/`
   - `webapp/backend/api/`
   - `webapp/backend/models/`
   - `webapp/config/`
3. 在 `frontend` 下创建 `index.html`
4. 在 `frontend/css` 下创建 `style.css`
5. 在 `frontend/js` 下创建 `app.js`
6. 在 `config` 下创建 `database.yml` 和 `settings.yml`
7. 递归查看整个项目结构

<details>
<summary>查看答案</summary>

```bash
mkdir -p /tmp/webapp/{frontend/{css,js,images},backend/{api,models},config}
touch /tmp/webapp/frontend/index.html
touch /tmp/webapp/frontend/css/style.css
touch /tmp/webapp/frontend/js/app.js
touch /tmp/webapp/config/{database.yml,settings.yml}
ls -R /tmp/webapp
```

</details>

### 练习 6：安全备份

你需要在修改重要配置文件前做备份：

1. 在 `/tmp` 下创建 `server-config` 目录，创建文件 `nginx.conf`、`app.conf`、`db.conf`
2. 给每个文件写入一行内容（用 `echo` 和重定向）
3. 创建 `backups` 目录
4. 把所有 `.conf` 文件**复制**到 `backups` 目录（不是移动）
5. 验证原文件还在，备份也存在
6. 给 `backups` 目录中的每个文件名加上 `.bak` 后缀

<details>
<summary>查看答案</summary>

```bash
mkdir /tmp/server-config
cd /tmp/server-config
echo "worker_processes 4;" > nginx.conf
echo "port = 8080" > app.conf
echo "host = localhost" > db.conf
mkdir backups
cp *.conf backups/
ls *.conf
ls backups/
cd backups
mv nginx.conf nginx.conf.bak
mv app.conf app.conf.bak
mv db.conf db.conf.bak
```

批量重命名也可以用循环：
```bash
for f in backups/*.conf; do mv "$f" "$f.bak"; done
```

</details>

---

## 三、查看文件内容

### 练习 7：日志文件初探

模拟查看一个应用日志：

1. 用以下命令生成模拟日志文件：
   ```bash
   for i in $(seq 1 100); do echo "$(date '+%Y-%m-%d %H:%M:%S') [INFO] Request $i processed" >> /tmp/app.log; done
   echo "2026-01-15 10:30:00 [ERROR] Database connection failed" >> /tmp/app.log
   echo "2026-01-15 10:30:01 [WARN] Retrying connection..." >> /tmp/app.log
   ```
2. 查看日志文件的总行数
3. 查看前 5 行
4. 查看最后 3 行
5. 用 `less` 打开文件，搜索 "ERROR"（写出搜索操作的按键）

<details>
<summary>查看答案</summary>

```bash
wc -l /tmp/app.log
head -5 /tmp/app.log
tail -3 /tmp/app.log
less /tmp/app.log
```

在 `less` 中搜索：输入 `/ERROR` 然后按 Enter。按 `n` 查找下一个匹配，按 `q` 退出。

</details>

### 练习 8：对比配置差异

你需要检查两个配置文件的内容：

1. 创建两个文件：
   ```bash
   printf "host=localhost\nport=3306\nuser=root\npassword=123456\n" > /tmp/config-dev.txt
   printf "host=prod-server\nport=3306\nuser=admin\npassword=s3cur3P@ss\n" > /tmp/config-prod.txt
   ```
2. 分别查看两个文件的内容
3. 用 `cat` 同时显示两个文件（加上文件头分隔）
4. 使用 `cat -n` 显示带行号的内容
5. 统计每个文件的行数、单词数、字符数

<details>
<summary>查看答案</summary>

```bash
cat /tmp/config-dev.txt
cat /tmp/config-prod.txt

# Display with file headers
echo "=== DEV ===" && cat /tmp/config-dev.txt && echo "=== PROD ===" && cat /tmp/config-prod.txt

cat -n /tmp/config-dev.txt
cat -n /tmp/config-prod.txt

wc /tmp/config-dev.txt
wc /tmp/config-prod.txt
```

</details>

### 练习 9：实时监控日志

你需要监控一个正在写入的日志文件：

1. 在后台持续写入日志：
   ```bash
   while true; do echo "$(date) - heartbeat" >> /tmp/monitor.log; sleep 2; done &
   ```
2. 用命令实时跟踪日志文件的新内容
3. 观察几秒后，停止跟踪（写出按键）
4. 停止后台写入进程

<details>
<summary>查看答案</summary>

```bash
while true; do echo "$(date) - heartbeat" >> /tmp/monitor.log; sleep 2; done &
tail -f /tmp/monitor.log
```

按 `Ctrl+C` 停止 `tail -f`。

```bash
# Stop the background process
kill %1
# Or find and kill
jobs
kill %1
```

</details>

---

## 四、文件权限

### 练习 10：部署 Web 文件

你需要为 Web 服务器设置正确的文件权限：

1. 在 `/tmp` 创建目录结构 `website/public`，并在其中创建 `index.html`、`style.css`、`app.js`
2. 创建 `website/scripts/deploy.sh`
3. 查看所有文件的当前权限
4. 设置目录权限为 `755`（所有者可读写执行，其他人可读和执行）
5. 设置 HTML/CSS/JS 文件权限为 `644`（所有者可读写，其他人只读）
6. 设置 `deploy.sh` 为可执行（`755`）
7. 验证权限设置

<details>
<summary>查看答案</summary>

```bash
mkdir -p /tmp/website/{public,scripts}
touch /tmp/website/public/{index.html,style.css,app.js}
touch /tmp/website/scripts/deploy.sh

ls -lR /tmp/website

chmod 755 /tmp/website /tmp/website/public /tmp/website/scripts
chmod 644 /tmp/website/public/*
chmod 755 /tmp/website/scripts/deploy.sh

ls -lR /tmp/website
```

也可以用符号模式：
```bash
chmod u+x /tmp/website/scripts/deploy.sh
```

</details>

### 练习 11：权限解读

查看以下权限并回答问题：

```
-rwxr-x--- 1 alice devteam 4096 Jan 10 09:00 build.sh
-rw-rw-r-- 1 alice devteam 2048 Jan 10 09:00 config.yml
drwxr-xr-x 2 root  root    4096 Jan 10 09:00 logs/
```

1. `build.sh` 的八进制权限是多少？谁可以执行它？
2. `config.yml` 的八进制权限是多少？组成员可以做什么？
3. `logs/` 的所有者是谁？其他用户可以做什么？
4. 如果用户 `bob`（属于 `devteam` 组）尝试修改 `build.sh`，会成功吗？
5. 如何让 `config.yml` 只有所有者能读写？

<details>
<summary>查看答案</summary>

1. `build.sh` 的八进制权限是 `750`。所有者 `alice` 和 `devteam` 组的成员可以执行，其他人不可以。
2. `config.yml` 的八进制权限是 `664`。组成员可以读和写。
3. `logs/` 的所有者是 `root`。其他用户可以读（`r`）和进入目录（`x`），但不能在其中创建或删除文件。
4. 不会成功。`bob` 属于 `devteam` 组，组权限是 `r-x`（只读和执行），没有写（`w`）权限。
5. `chmod 600 config.yml` 或 `chmod go-rw config.yml`

</details>

---

## 五、I/O Redirection 与 Pipe

### 练习 12：整理系统信息

你需要收集系统信息并保存为报告：

1. 把当前日期时间写入 `/tmp/system-report.txt`（覆盖）
2. 把系统主机名**追加**到同一个文件
3. 把当前登录用户名**追加**到同一个文件
4. 把系统正常运行时间（`uptime`）**追加**到同一个文件
5. 查看最终的报告文件

<details>
<summary>查看答案</summary>

```bash
date > /tmp/system-report.txt
hostname >> /tmp/system-report.txt
whoami >> /tmp/system-report.txt
uptime >> /tmp/system-report.txt
cat /tmp/system-report.txt
```

</details>

### 练习 13：Pipe 链式处理

使用 pipe 完成以下数据处理任务：

1. 列出 `/usr/bin` 中的所有文件，并统计总数
2. 列出 `/etc` 中的文件，按文件名排序，只显示前 10 个
3. 查看当前系统中的环境变量，找出包含 "PATH" 的行
4. 列出 `/usr/bin` 中的文件，筛选出包含 "zip" 的命令

<details>
<summary>查看答案</summary>

```bash
ls /usr/bin | wc -l
ls /etc | sort | head -10
env | grep PATH
ls /usr/bin | grep zip
```

</details>

### 练习 14：构建数据处理 Pipeline

有一个文件包含学生成绩，你需要处理它：

1. 创建成绩文件：
   ```bash
   cat > /tmp/scores.txt << 'EOF'
   Alice 85
   Bob 92
   Charlie 78
   Diana 95
   Eve 88
   Frank 72
   Grace 91
   Henry 83
   EOF
   ```
2. 按成绩从高到低排序
3. 只显示成绩最高的 3 个学生
4. 统计总共有多少个学生
5. 找出成绩包含 "9" 的学生（即 90 分以上）

<details>
<summary>查看答案</summary>

```bash
sort -k2 -n -r /tmp/scores.txt
sort -k2 -n -r /tmp/scores.txt | head -3
wc -l /tmp/scores.txt
grep "9[0-9]" /tmp/scores.txt
```

注意：`grep "9"` 会匹配任何包含 9 的行。如果要精确匹配 90+ 分，用 `awk '$2 >= 90' /tmp/scores.txt` 更准确。

</details>

---

## 六、获取帮助与综合应用

### 练习 15：自学新命令

你遇到了一个不熟悉的命令 `sort`，需要自己搞懂它：

1. 查看 `sort` 的简要帮助
2. 查看 `sort` 的手册页（写出搜索 "reverse" 的方法）
3. 创建一个文件 `/tmp/names.txt`，每行一个名字（至少 5 个）
4. 用 `sort` 按字母顺序排列
5. 用 `sort` 按字母逆序排列
6. 用 `sort` 去除重复行并排序

<details>
<summary>查看答案</summary>

```bash
sort --help

# Open man page and search
man sort
# Type /reverse then Enter to search

cat > /tmp/names.txt << 'EOF'
Charlie
Alice
Bob
Diana
Alice
Eve
Bob
EOF

sort /tmp/names.txt
sort -r /tmp/names.txt
sort -u /tmp/names.txt
```

</details>

### 练习 16：综合场景 —— 搭建实验环境

你是一名初级运维，主管要求你完成以下任务：

1. 在 `/tmp/lab` 下创建项目结构：`app/`、`logs/`、`config/`、`backups/`
2. 在 `config/` 下创建 `app.conf`，写入 3 行配置（自定义内容）
3. 把 `app.conf` 备份到 `backups/app.conf.bak`
4. 在 `logs/` 下创建 `access.log`，写入 5 行模拟日志
5. 设置 `logs/` 目录权限为 `755`，日志文件权限为 `644`
6. 设置 `config/` 目录权限为 `750`（只有所有者和同组可访问）
7. 查看最后 2 行日志
8. 把 `ls -lR /tmp/lab` 的输出保存到 `/tmp/lab/structure.txt`
9. 显示 `structure.txt` 的内容，确认一切就绪

<details>
<summary>查看答案</summary>

```bash
mkdir -p /tmp/lab/{app,logs,config,backups}

cat > /tmp/lab/config/app.conf << 'EOF'
server_port=8080
db_host=localhost
log_level=info
EOF

cp /tmp/lab/config/app.conf /tmp/lab/backups/app.conf.bak

cat > /tmp/lab/logs/access.log << 'EOF'
2026-01-15 10:00:01 GET /index.html 200
2026-01-15 10:00:02 GET /style.css 200
2026-01-15 10:00:03 POST /api/login 302
2026-01-15 10:00:05 GET /dashboard 200
2026-01-15 10:00:10 GET /api/data 500
EOF

chmod 755 /tmp/lab/logs
chmod 644 /tmp/lab/logs/access.log
chmod 750 /tmp/lab/config

tail -2 /tmp/lab/logs/access.log

ls -lR /tmp/lab > /tmp/lab/structure.txt
cat /tmp/lab/structure.txt
```

</details>

### 练习 17：综合场景 —— 从零开始的探索

你登录到一台全新的服务器，需要做一些基本的探索：

1. 确认当前用户名和所在目录
2. 查看系统中有哪些 shell 可用（提示：`/etc/shells`）
3. 查看 `/etc` 目录下有多少个 `.conf` 文件
4. 找到 `/var/log` 目录下最大的 3 个文件
5. 查看系统 hostname 和内核版本
6. 把以上所有信息汇总保存到 `~/server-info.txt`

<details>
<summary>查看答案</summary>

```bash
whoami
pwd

cat /etc/shells

ls /etc/*.conf 2>/dev/null | wc -l

ls -lS /var/log 2>/dev/null | head -4

hostname
uname -r

{
  echo "=== User & Directory ==="
  whoami
  pwd
  echo ""
  echo "=== Available Shells ==="
  cat /etc/shells
  echo ""
  echo "=== Config files in /etc ==="
  ls /etc/*.conf 2>/dev/null | wc -l
  echo ""
  echo "=== Largest files in /var/log ==="
  ls -lS /var/log 2>/dev/null | head -4
  echo ""
  echo "=== System Info ==="
  hostname
  uname -r
} > ~/server-info.txt

cat ~/server-info.txt
```

</details>

---

**完成所有练习了？太棒了！** 🎉

继续挑战 → [进阶篇综合练习](intermediate-exercises.md)
