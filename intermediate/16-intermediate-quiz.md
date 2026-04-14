# 第 16 课：进阶综合测验

> 上一课：[第 15 课：Environment Variables 与 Shell 配置](./15-environment-variables.md) | 下一课：[第 17 课：Shell 脚本基础](../advanced/17-scripting-fundamentals.md)

---

## 学习目标

本课是中级阶段（第 09–15 课）的综合测验，涵盖以下主题：

- 第 09 课：文本搜索 grep 与 Regex
- 第 10 课：流编辑器 sed
- 第 11 课：数据处理工具 awk
- 第 12 课：Process 管理
- 第 13 课：网络工具
- 第 14 课：包管理器
- 第 15 课：Environment Variables 与 Shell 配置

---

## 评分标准

| 等级 | 分数 | 说明 |
|------|------|------|
| 🏆 精通 | 18–20 | 中级知识已完全掌握，可以进入高级阶段 |
| ✅ 良好 | 14–17 | 基础扎实，建议复习薄弱环节后继续 |
| 📖 需要复习 | 10–13 | 回顾相关课程后再次测试 |
| 🔄 重修 | 0–9 | 建议重新学习中级阶段各课程 |

每题 1 分，共 20 题。

---

## 测验题目

### grep 与 Regex（第 09 课）

**1. 填空题：写出命令，在 `/var/log/syslog` 中搜索包含 "error" 或 "Error" 或 "ERROR" 的行。**

<details>
<summary>查看答案</summary>

```bash
grep -i "error" /var/log/syslog
```

`-i` 表示忽略大小写。也可以使用扩展正则：

```bash
grep -E "[Ee][Rr][Rr][Oo][Rr]" /var/log/syslog
```

</details>

**2. 选择题：以下哪个正则表达式能匹配一个合法的 IPv4 地址格式（如 `192.168.1.1`）？**

A. `[0-9]+.[0-9]+.[0-9]+.[0-9]+`
B. `[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+`
C. `\d{3}\.\d{3}\.\d{3}\.\d{3}`
D. `[0-9]*\.[0-9]*\.[0-9]*\.[0-9]*`

<details>
<summary>查看答案</summary>

**B**。`.` 在正则中是通配符，匹配任意字符，必须用 `\.` 来匹配字面的点号。`+` 表示一个或多个数字。

- A 错误：`.` 没有转义，会匹配任意字符
- C 错误：`\d` 在基本正则中不一定被支持，且 IP 地址每段不一定是 3 位数
- D 错误：`*` 表示零个或多个，会匹配空字符串

</details>

**3. 填空题：使用 `grep` 递归搜索 `~/project` 目录下所有 `.py` 文件中包含 `import requests` 的行，并显示行号。**

<details>
<summary>查看答案</summary>

```bash
grep -rn "import requests" ~/project --include="*.py"
```

- `-r`：递归搜索
- `-n`：显示行号
- `--include="*.py"`：仅搜索 `.py` 文件

</details>

### sed（第 10 课）

**4. 填空题：使用 `sed` 将文件 `config.txt` 中所有的 `localhost` 替换为 `192.168.1.100`（原地修改）。**

<details>
<summary>查看答案</summary>

```bash
sed -i 's/localhost/192.168.1.100/g' config.txt
```

- `-i`：原地编辑
- `s/old/new/g`：全局替换（`g` 表示每行所有匹配项都替换）

</details>

**5. 选择题：`sed -n '10,20p' file.txt` 的作用是什么？**

A. 删除第 10 到 20 行
B. 在第 10 行到 20 行后追加内容
C. 只打印第 10 到 20 行
D. 将第 10 到 20 行写入新文件

<details>
<summary>查看答案</summary>

**C**。`-n` 抑制默认输出，`10,20p` 表示打印第 10 到 20 行。

</details>

**6. 填空题：使用 `sed` 删除文件中所有空行。**

<details>
<summary>查看答案</summary>

```bash
sed '/^$/d' file.txt
```

- `^$`：匹配空行（行首直接接行尾）
- `d`：删除匹配的行

</details>

### awk（第 11 课）

**7. 填空题：使用 `awk` 打印 `/etc/passwd` 中第 1 列（用户名）和第 7 列（Shell），以 Tab 分隔。**

<details>
<summary>查看答案</summary>

```bash
awk -F: '{print $1 "\t" $7}' /etc/passwd
```

或使用 OFS：

```bash
awk -F: 'BEGIN{OFS="\t"} {print $1, $7}' /etc/passwd
```

`-F:` 指定冒号为分隔符。

</details>

**8. 场景题：你有一个 CSV 日志文件 `access.log`，每行格式为 `IP,时间,URL,状态码`。写出命令统计每个 IP 的访问次数，并按次数从大到小排序。**

<details>
<summary>查看答案</summary>

```bash
awk -F, '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log | sort -rn
```

- `-F,`：以逗号为分隔符
- `count[$1]++`：用关联数组统计每个 IP 出现的次数
- `END` 块：遍历数组输出结果
- `sort -rn`：按数字降序排列

</details>

### Process 管理（第 12 课）

**9. 选择题：以下哪种信号**无法**被进程捕获或忽略？**

A. `SIGTERM` (15)
B. `SIGINT` (2)
C. `SIGKILL` (9)
D. `SIGHUP` (1)

<details>
<summary>查看答案</summary>

**C**。`SIGKILL` (9) 和 `SIGSTOP` (19) 是两个无法被捕获、处理或忽略的信号，由内核直接处理。

</details>

**10. 场景题：你在 SSH 远程服务器上运行了一个数据导入脚本，突然网络不稳定需要断开 SSH。该脚本已经在前台运行了 2 小时，你不想中断它。写出操作步骤。**

<details>
<summary>查看答案</summary>

```bash
# Step 1: suspend the foreground process
Ctrl+Z

# Step 2: resume it in background
bg

# Step 3: detach it from the shell
disown

# Step 4: safely disconnect
exit
```

现在即使 SSH 断开，进程也会继续运行。下次登录后可以通过 `ps aux | grep <脚本名>` 查看其状态。

> 如果事先知道需要长时间运行，应该使用 `nohup cmd &` 或 `tmux` / `screen`。

</details>

**11. 填空题：写出命令，列出系统中占用内存最多的前 5 个进程（显示 PID、内存百分比和命令名）。**

<details>
<summary>查看答案</summary>

```bash
ps aux --sort=-%mem | head -6
```

或者更精确地指定输出列：

```bash
ps -eo pid,%mem,comm --sort=-%mem | head -6
```

`--sort=-%mem` 按内存使用率降序排列，`head -6` 包含标题行在内取前 6 行。

</details>

### 网络工具（第 13 课）

**12. 填空题：使用 `curl` 发送一个 POST 请求到 `https://api.example.com/data`，请求体为 JSON `{"status":"ok"}`，并带上 Header `Authorization: Bearer TOKEN`。**

<details>
<summary>查看答案</summary>

```bash
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"status":"ok"}'
```

</details>

**13. 选择题：以下哪个命令可以查看本机 443 端口是否处于监听状态？**

A. `ping localhost:443`
B. `ss -tlnp | grep :443`
C. `curl :443`
D. `dig localhost 443`

<details>
<summary>查看答案</summary>

**B**。`ss -tlnp` 显示所有 TCP 监听端口和对应的进程信息，通过 `grep :443` 过滤。

- A 错误：`ping` 使用 ICMP 协议，不支持端口
- C 错误：语法不正确
- D 错误：`dig` 用于 DNS 查询，不检查端口

</details>

**14. 场景题：你需要将本地 `~/backup/data` 目录同步到远程服务器 `192.168.1.50` 的 `/data/backup/` 目录，要求：排除 `.tmp` 文件、保持文件权限、压缩传输、先预览不实际执行。写出命令。**

<details>
<summary>查看答案</summary>

```bash
rsync -avzn --exclude='*.tmp' ~/backup/data/ user@192.168.1.50:/data/backup/
```

- `-a`：归档模式（保留权限、时间戳等）
- `-v`：详细输出
- `-z`：传输时压缩
- `-n`：dry run（模拟运行）
- `--exclude='*.tmp'`：排除 `.tmp` 文件

确认无误后去掉 `-n` 参数正式执行。

</details>

### 包管理器（第 14 课）

**15. 选择题：在 Ubuntu 上安装软件前，应该先执行哪个命令？为什么？**

A. `sudo apt upgrade` —— 升级所有软件
B. `sudo apt update` —— 刷新软件源索引
C. `sudo apt clean` —— 清理缓存
D. `sudo apt autoremove` —— 删除无用包

<details>
<summary>查看答案</summary>

**B**。`sudo apt update` 从配置的软件仓库下载最新的包索引信息。如果不先运行此命令，`apt install` 可能找不到最新的包版本，甚至可能因为索引过期而安装失败。

</details>

**16. 填空题：你通过 `dpkg -i package.deb` 安装了一个 `.deb` 包，提示缺少依赖。写出修复命令。**

<details>
<summary>查看答案</summary>

```bash
sudo apt install -f
```

`-f`（`--fix-broken`）让 `apt` 自动下载并安装缺失的依赖，修复 `dpkg` 留下的不完整安装状态。

</details>

**17. 场景题：在 CentOS 服务器上，你需要编译一个 C 程序但系统没有 `gcc`。写出安装编译工具链并编译的完整步骤。**

<details>
<summary>查看答案</summary>

```bash
# install development tools
sudo yum groupinstall -y "Development Tools"

# verify gcc is installed
gcc --version

# compile the program
gcc -o myprogram main.c

# or if the project uses autotools
./configure
make -j$(nproc)
sudo make install
```

"Development Tools" 包组包含 `gcc`、`g++`、`make`、`autoconf` 等编译所需的核心工具。

</details>

### Environment Variables 与 Shell 配置（第 15 课）

**18. 选择题：以下哪种方式设置的环境变量，子进程**无法**访问？**

A. `export MY_VAR="hello"`
B. `MY_VAR="hello"`
C. `MY_VAR="hello" ./script.sh`（在命令前设置）
D. 在 `~/.bashrc` 中写 `export MY_VAR="hello"`

<details>
<summary>查看答案</summary>

**B**。`MY_VAR="hello"` 仅设置了 Shell 变量，不会传递给子进程。

- A：`export` 将变量导出为环境变量，子进程可以继承
- C：命令前的 `VAR=value` 会临时将变量作为环境变量传递给该命令
- D：`~/.bashrc` 中的 `export` 在每次新 Shell 启动时执行，子进程可以继承

</details>

**19. 填空题：你安装了一个程序到 `/opt/mytool/bin`，需要永久添加到 PATH 中（使用 Bash）。写出需要在哪个文件中添加什么内容。**

<details>
<summary>查看答案</summary>

在 `~/.bashrc` 中添加：

```bash
export PATH="/opt/mytool/bin:$PATH"
```

然后执行 `source ~/.bashrc` 使其立即生效。

将新路径放在 `$PATH` 前面，这样 `/opt/mytool/bin` 中的命令会优先于系统中的同名命令。

</details>

**20. 场景题：你的同事反映在 SSH 登录服务器时 `.bashrc` 中的 alias 没有生效，但打开一个新的终端模拟器窗口时 alias 是正常的。分析原因并给出解决方案。**

<details>
<summary>查看答案</summary>

**原因**：SSH 登录创建的是 **login shell**，它读取 `~/.bash_profile`（或 `~/.profile`），而**不会**自动读取 `~/.bashrc`。终端模拟器通常创建 **non-login interactive shell**，会读取 `~/.bashrc`。

**解决方案**：在 `~/.bash_profile` 中添加以下内容：

```bash
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

这样 login shell 在加载 `~/.bash_profile` 时，会自动 source `~/.bashrc`，alias 就能在两种场景下都生效了。

</details>

---

## 成绩自评

统计你答对的题数，对照评分标准：

| 得分 | 等级 | 建议 |
|------|------|------|
| 18–20 | 🏆 精通 | 中级阶段全部掌握，准备进入高级阶段吧！ |
| 14–17 | ✅ 良好 | 掌握不错，复习一下错题对应的课程即可 |
| 10–13 | 📖 需要复习 | 回顾相关课程（特别是错题涉及的章节），再测一次 |
| 0–9 | 🔄 重修 | 建议从第 09 课开始重新学习中级阶段内容 |

**答错题目复习指引**：

| 题号 | 对应课程 |
|------|---------|
| 1–3 | [第 09 课：文本搜索 grep 与 Regex](./09-grep-and-regex.md) |
| 4–6 | [第 10 课：流编辑器 sed](./10-sed.md) |
| 7–8 | [第 11 课：数据处理工具 awk](./11-awk.md) |
| 9–11 | [第 12 课：Process 管理](./12-process-management.md) |
| 12–14 | [第 13 课：网络工具](./13-networking.md) |
| 15–17 | [第 14 课：包管理器](./14-package-management.md) |
| 18–20 | [第 15 课：Environment Variables 与 Shell 配置](./15-environment-variables.md) |
