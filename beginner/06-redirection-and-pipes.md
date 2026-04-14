# 第 06 课：I/O Redirection 与 Pipe

> 上一课：[第 05 课：文件权限与所有权](./05-permissions.md) | 下一课：[第 07 课：获取帮助](./07-getting-help.md)

## 学习目标

- 理解 Linux 中 stdin、stdout、stderr 三种标准 I/O 流
- 掌握输出重定向 `>` 和 `>>`（覆盖与追加）
- 掌握输入重定向 `<`
- 掌握错误重定向 `2>` 和合并输出 `2>&1`
- 掌握管道 `|` 将多个命令串联
- 掌握 `tee` 命令同时输出到屏幕和文件
- 理解 `/dev/null` 的用途

---

## 知识讲解

### 1. 三种标准 I/O 流

每个 Linux 进程启动时，都会自动打开三个 File Descriptor（文件描述符）：

| 名称 | File Descriptor | 缩写 | 默认指向 |
|------|----------------|------|---------|
| Standard Input | `0` | stdin | 键盘 |
| Standard Output | `1` | stdout | 终端屏幕 |
| Standard Error | `2` | stderr | 终端屏幕 |

```
            ┌──────────┐
 stdin (0) ──▶│          │──▶ stdout (1) ──▶ 终端
  (键盘)     │  Process │
             │          │──▶ stderr (2) ──▶ 终端
             └──────────┘
```

I/O Redirection 的核心思想：将这些流从默认的键盘/终端**重新指向**文件或其他命令。

### 2. 输出重定向：`>` 和 `>>`

#### `>` — 覆盖写入

```bash
echo "Hello World" > output.txt    # write to file (overwrite)
ls -l /etc > filelist.txt          # redirect command output to file
```

> ⚠️ `>` 会先**清空**目标文件再写入。如果文件不存在则创建。

#### `>>` — 追加写入

```bash
echo "Line 1" > log.txt           # create and write
echo "Line 2" >> log.txt          # append to existing file
echo "Line 3" >> log.txt          # append another line
cat log.txt
# Line 1
# Line 2
# Line 3
```

一个实用技巧——用 `>` 清空文件内容：

```bash
> large_log.txt                   # truncate file to zero bytes
```

### 3. 输入重定向：`<`

将文件内容作为命令的 stdin 输入：

```bash
wc -l < data.txt                  # count lines (filename not shown in output)
sort < names.txt                  # sort file content
```

对比两种写法的区别：

```bash
wc -l data.txt      # output: 42 data.txt  (filename shown)
wc -l < data.txt    # output: 42           (no filename, data comes from stdin)
```

#### Here Document（`<<`）

```bash
cat << EOF
This is line 1
This is line 2
Variable: $HOME
EOF
```

Here Document 会将 `<<` 和终止符（如 `EOF`）之间的内容作为 stdin 输入，且支持变量展开。

#### Here String（`<<<`）

```bash
wc -w <<< "count these words"     # output: 3
grep "error" <<< "this is an error message"
```

将一个字符串直接作为 stdin 传入命令。

### 4. 错误重定向：`2>` 和 `2>&1`

#### `2>` — 重定向 stderr

```bash
ls /nonexistent 2> errors.txt     # redirect error messages to file
cat errors.txt
# ls: cannot access '/nonexistent': No such file or directory
```

#### 分别重定向 stdout 和 stderr

```bash
# stdout -> output.txt, stderr -> errors.txt
find / -name "*.conf" > output.txt 2> errors.txt
```

#### `2>&1` — 合并 stderr 到 stdout

```bash
# redirect both stdout and stderr to the same file
command > all_output.txt 2>&1

# Bash shorthand (equivalent)
command &> all_output.txt
```

> ⚠️ 顺序很重要：`> file 2>&1` 是正确的（先将 stdout 指向文件，再将 stderr 指向 stdout 的当前位置）。`2>&1 > file` 的效果不同——stderr 仍然输出到终端。

常见实用场景：

```bash
# run script, log everything to file
./deploy.sh > deploy.log 2>&1

# discard errors, only keep normal output
find / -name "*.log" 2>/dev/null

# keep errors, discard normal output
command > /dev/null 2> errors.txt
```

### 5. 管道 `|` — Pipe

管道将前一个命令的 stdout 作为后一个命令的 stdin：

```bash
command1 | command2 | command3
```

```
command1 stdout ──▶ stdin command2 stdout ──▶ stdin command3 ──▶ 终端
```

#### 常用管道组合

```bash
# find large files and sort by size
ls -lS /var/log | head -10

# count running processes
ps aux | wc -l

# search in command output
history | grep "git"

# sort and deduplicate
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# paginate long output
dpkg -l | less

# find specific process
ps aux | grep nginx | grep -v grep
```

#### 管道 vs 重定向的区别

```bash
# pipe: command-to-command (stdout → stdin)
cat file.txt | grep "error"

# redirect: command-to-file or file-to-command
grep "error" < file.txt          # file → command
grep "error" file.txt > result   # command → file
```

> 管道连接的是**命令**，重定向连接的是**命令与文件**。

### 6. `tee` — 分流输出

`tee` 从 stdin 读取数据，同时输出到 stdout **和**文件，像水管的 T 型接头：

```bash
# display on screen AND save to file
ls -l | tee filelist.txt

# append mode
echo "new log entry" | tee -a app.log

# write to multiple files simultaneously
echo "config" | tee file1.txt file2.txt file3.txt
```

```
              ┌──▶ stdout (终端)
command ──▶ tee
              └──▶ file
```

实用场景：

```bash
# monitor a long build while keeping a log
make 2>&1 | tee build.log

# need sudo to write to system file
echo "127.0.0.1 myapp.local" | sudo tee -a /etc/hosts
```

> `sudo echo "..." > /etc/hosts` 不起作用，因为重定向由当前 shell 执行（非 root）。`sudo tee` 可以解决这个问题。

### 7. `/dev/null` — 黑洞设备

`/dev/null` 是一个特殊文件，写入的任何数据都会被丢弃，读取时立即返回 EOF。

```bash
# discard all output
command > /dev/null 2>&1

# discard only errors
find / -name "*.conf" 2>/dev/null

# discard only stdout, keep stderr
command > /dev/null

# check if command succeeds (only care about exit code)
grep -q "pattern" file.txt > /dev/null 2>&1
echo $?    # 0 = found, 1 = not found
```

常见用途：
- 丢弃不需要的输出，保持终端整洁
- Cron job 中抑制输出
- 只关心命令的退出码，不关心输出内容

```bash
# crontab example: run backup silently
0 2 * * * /usr/local/bin/backup.sh > /dev/null 2>&1
```

---

## 实战演练

模拟日志处理与数据分析的场景。

**步骤 1：创建练习环境**

```bash
mkdir -p ~/redir-practice && cd ~/redir-practice
```

**步骤 2：用重定向创建测试数据**

```bash
echo "apple" > fruits.txt
echo "banana" >> fruits.txt
echo "cherry" >> fruits.txt
echo "apple" >> fruits.txt
echo "banana" >> fruits.txt
cat fruits.txt
```

**步骤 3：体验 `>` 覆盖 vs `>>` 追加**

```bash
echo "overwrite test" > fruits.txt
cat fruits.txt
# only "overwrite test" left — previous content is gone!
```

重新创建数据：

```bash
printf "apple\nbanana\ncherry\napple\nbanana\n" > fruits.txt
```

**步骤 4：使用管道排序和去重**

```bash
sort fruits.txt | uniq           # sorted unique items
sort fruits.txt | uniq -c        # with count
sort fruits.txt | uniq -c | sort -rn  # sorted by frequency
```

**步骤 5：生成模拟日志并分析**

```bash
for i in $(seq 1 50); do
  echo "2026-04-14 10:00:$((RANDOM % 60)) [INFO] request processed"
done > server.log

for i in $(seq 1 5); do
  echo "2026-04-14 10:01:$((RANDOM % 60)) [ERROR] connection failed"
done >> server.log

for i in $(seq 1 10); do
  echo "2026-04-14 10:02:$((RANDOM % 60)) [WARN] high latency"
done >> server.log
```

```bash
# count each log level
grep -c "INFO" server.log
grep -c "ERROR" server.log
grep -c "WARN" server.log

# extract only ERROR lines and save to file
grep "ERROR" server.log > errors_only.txt
cat errors_only.txt
```

**步骤 6：体验错误重定向**

```bash
ls /etc /nonexistent
# shows /etc contents on stdout, error on stderr

ls /etc /nonexistent > stdout.txt 2> stderr.txt
cat stdout.txt     # directory listing
cat stderr.txt     # error message

ls /etc /nonexistent &> both.txt
cat both.txt       # both output and error
```

**步骤 7：使用 `tee` 同时查看和保存**

```bash
grep "ERROR" server.log | tee error_report.txt
# displays on screen AND saves to error_report.txt
wc -l error_report.txt
```

**步骤 8：使用 `/dev/null` 丢弃输出**

```bash
find / -name "*.conf" 2>/dev/null | head -5
```

只显示找到的文件，权限错误全部被丢弃。

**步骤 9：综合管道练习**

```bash
# find the top 3 most frequent log levels
awk '{print $3}' server.log | sort | uniq -c | sort -rn | head -3
```

**步骤 10：清理**

```bash
cd ~
rm -r ~/redir-practice
```

---

## 小结

| 符号 | 作用 | 示例 |
|------|------|------|
| `>` | stdout 覆盖写入文件 | `echo "hi" > f.txt` |
| `>>` | stdout 追加写入文件 | `echo "hi" >> f.txt` |
| `<` | 从文件读取 stdin | `wc -l < f.txt` |
| `2>` | stderr 重定向到文件 | `cmd 2> err.txt` |
| `2>&1` | stderr 合并到 stdout | `cmd > all.txt 2>&1` |
| `&>` | stdout + stderr 一起重定向 | `cmd &> all.txt` |
| `|` | 管道：前一命令 stdout → 后一命令 stdin | `ls | grep ".txt"` |
| `tee` | 分流输出到 stdout 和文件 | `cmd | tee log.txt` |
| `/dev/null` | 丢弃输出的黑洞 | `cmd > /dev/null 2>&1` |

---

## 练习

**1. `>` 和 `>>` 有什么区别？如何用重定向清空一个文件？**

<details>
<summary>查看答案</summary>

- `>` 覆盖模式：先清空文件内容再写入，如果文件不存在则创建
- `>>` 追加模式：在文件末尾添加内容，不影响已有内容

清空文件的方法：

```bash
> filename.txt           # redirect nothing to file
cat /dev/null > filename.txt  # alternative
truncate -s 0 filename.txt   # another alternative
```

</details>

**2. 为什么 `> file 2>&1` 和 `2>&1 > file` 的效果不同？**

<details>
<summary>查看答案</summary>

Shell 从左到右处理重定向：

- `> file 2>&1`：先将 stdout 指向 file，然后将 stderr 指向 stdout 当前指向的位置（即 file）。结果：stdout 和 stderr **都**写入 file。

- `2>&1 > file`：先将 stderr 指向 stdout 当前指向的位置（即终端），然后将 stdout 指向 file。结果：stdout 写入 file，但 stderr 仍然输出到**终端**。

关键是 `2>&1` 复制的是执行这条语句**那一刻** stdout 的指向，而非之后的指向。

</details>

**3. 如何统计 `/var/log/syslog` 中包含 "error"（不区分大小写）的行数，并将结果保存到文件？**

<details>
<summary>查看答案</summary>

```bash
grep -ic "error" /var/log/syslog > error_count.txt
```

- `-i`：忽略大小写
- `-c`：只输出匹配行数

或者使用管道：

```bash
grep -i "error" /var/log/syslog | wc -l > error_count.txt
```

</details>

**4. `sudo echo "text" > /etc/somefile` 为什么会报权限错误？如何正确写入？**

<details>
<summary>查看答案</summary>

因为重定向 `>` 是由当前 shell（普通用户）执行的，`sudo` 只提升了 `echo` 的权限，而 shell 本身没有权限写入 `/etc/somefile`。

正确的写法：

```bash
echo "text" | sudo tee /etc/somefile        # overwrite
echo "text" | sudo tee -a /etc/somefile     # append
```

`tee` 在 `sudo` 权限下运行，因此可以写入受保护的文件。

</details>

**5. 写出一条命令：从 `access.log` 中提取所有 IP 地址（第一列），统计每个 IP 的访问次数，按次数降序排列，取前 5 名。**

<details>
<summary>查看答案</summary>

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5
```

管道链解析：
1. `awk '{print $1}'` — 提取每行的第一个字段（IP 地址）
2. `sort` — 排序（`uniq` 要求输入已排序）
3. `uniq -c` — 去重并统计每个 IP 出现的次数
4. `sort -rn` — 按数字降序排列（`-r` 逆序，`-n` 数字排序）
5. `head -5` — 取前 5 行

</details>
