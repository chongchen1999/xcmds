# 第 08 课：入门综合测验

> 上一课：[第 07 课：获取帮助](./07-getting-help.md) | 下一课：[第 09 课：grep 与正则表达式](../intermediate/09-grep-and-regex.md)

## 学习目标

- 通过综合测验检验 01-07 课的学习成果
- 涵盖终端与 Shell、文件系统导航、文件操作、查看文件、权限、重定向与管道、获取帮助
- 巩固知识，查漏补缺

---

## 测验说明

- 共 **20 道题目**，涵盖入门课程的所有知识点
- 题型包括：填空题（写出命令）、选择题、情景题
- 每题 5 分，满分 100 分
- 所有答案在 `<details>` 标签中，先独立作答再对照
- 建议时间：30-45 分钟

---

## 题目

### 第 1 题（终端与 Shell）

Shell 和 Terminal 有什么区别？常见的 Shell 有哪些？

<details>
<summary>查看答案</summary>

- **Terminal（终端模拟器）** 是一个图形程序，提供输入和显示文本的窗口（如 GNOME Terminal、iTerm2、Windows Terminal）
- **Shell** 是运行在 Terminal 内的命令解释器，负责接收、解析和执行命令（如 Bash、Zsh、Fish）

常见的 Shell：
- **Bash**（Bourne Again Shell）— 大多数 Linux 发行版的默认 Shell
- **Zsh** — macOS 默认 Shell，功能丰富
- **Fish** — 用户友好的现代 Shell
- **sh** — 最基础的 POSIX Shell

查看当前使用的 Shell：`echo $SHELL`

</details>

---

### 第 2 题（文件系统导航）

写出以下操作对应的命令：

1. 打印当前工作目录的绝对路径
2. 回到主目录
3. 回到上一次所在的目录
4. 列出 `/var/log` 下所有文件（包括隐藏文件），并显示详细信息

<details>
<summary>查看答案</summary>

```bash
# 1
pwd

# 2
cd ~       # or just: cd

# 3
cd -

# 4
ls -la /var/log
```

</details>

---

### 第 3 题（文件系统导航）

绝对路径和相对路径有什么区别？`.` 和 `..` 分别代表什么？

<details>
<summary>查看答案</summary>

- **绝对路径**：从根目录 `/` 开始的完整路径，如 `/home/alice/documents/file.txt`
- **相对路径**：相对于当前工作目录的路径，如 `documents/file.txt`

特殊目录：
- `.` — 当前目录
- `..` — 上一级目录（父目录）

示例：当前在 `/home/alice` 时
- `./documents` 等同于 `/home/alice/documents`
- `../bob` 等同于 `/home/bob`

</details>

---

### 第 4 题（文件操作）

写出命令完成以下任务：

1. 创建目录结构 `project/src/utils/`（包括所有中间目录）
2. 将 `report.txt` 复制到 `backup/` 目录
3. 将 `old_name.txt` 重命名为 `new_name.txt`
4. 删除 `temp/` 目录及其所有内容

<details>
<summary>查看答案</summary>

```bash
# 1
mkdir -p project/src/utils/

# 2
cp report.txt backup/

# 3
mv old_name.txt new_name.txt

# 4
rm -r temp/
```

</details>

---

### 第 5 题（文件操作）

`cp -r` 和 `cp` 有什么区别？`rm` 和 `rm -r` 呢？

<details>
<summary>查看答案</summary>

- `cp` 只能复制文件，不能复制目录
- `cp -r` 递归复制，可以复制目录及其所有内容

- `rm` 只能删除文件，不能删除目录
- `rm -r` 递归删除，可以删除目录及其所有内容

> `-r` 代表 recursive（递归），对目录及其子目录中的所有内容生效。

</details>

---

### 第 6 题（文件操作 — 选择题）

以下哪个命令可以安全地创建一个 Hard Link？

A. `ln -s original.txt link.txt`
B. `ln original.txt link.txt`
C. `cp original.txt link.txt`
D. `mv original.txt link.txt`

<details>
<summary>查看答案</summary>

**B**

- A 创建的是 Symbolic Link（符号链接 / 软链接）
- B 创建的是 Hard Link（硬链接）
- C 是复制文件（独立的副本）
- D 是移动/重命名文件

Hard Link 与原文件共享同一个 inode，修改任一方内容另一方同步变化。删除原文件后 Hard Link 仍可访问数据。

</details>

---

### 第 7 题（查看文件）

有一个 10000 行的日志文件 `server.log`，写出命令完成以下任务：

1. 查看最后 20 行
2. 实时监控新增的日志
3. 查看第 100 到 110 行
4. 统计文件总行数

<details>
<summary>查看答案</summary>

```bash
# 1
tail -n 20 server.log

# 2
tail -f server.log

# 3
head -n 110 server.log | tail -n 11
# or: sed -n '100,110p' server.log

# 4
wc -l server.log
```

</details>

---

### 第 8 题（查看文件）

`cat`、`less`、`head`、`tail` 各适合什么场景？

<details>
<summary>查看答案</summary>

| 命令 | 适合场景 |
|------|----------|
| `cat` | 查看小文件、拼接多个文件 |
| `less` | 浏览大文件，支持搜索和上下翻页 |
| `head` | 快速查看文件开头部分（如前 N 行） |
| `tail` | 查看文件末尾、实时监控日志（`tail -f`） |

> 大文件避免使用 `cat`，因为它会一次性输出所有内容。

</details>

---

### 第 9 题（查看文件 — 选择题）

`file mystery_data` 输出 `mystery_data: gzip compressed data`。以下说法正确的是：

A. 该文件的扩展名一定是 `.gz`
B. `file` 命令是通过扩展名判断文件类型的
C. `file` 命令通过分析文件的 magic bytes 判断类型，与扩展名无关
D. 该文件无法被解压

<details>
<summary>查看答案</summary>

**C**

`file` 命令通过读取文件头部的特征字节（magic bytes）来判断文件类型，完全不依赖文件扩展名。即使文件没有扩展名或扩展名错误，`file` 也能正确识别。

- A 错误：扩展名可以是任意的，甚至可以没有
- B 错误：`file` 不看扩展名
- D 错误：gzip 压缩数据可以用 `gzip -d` 或 `gunzip` 解压

</details>

---

### 第 10 题（权限）

解读以下 `ls -l` 输出：

```
-rwxr-x--- 2 alice developers 8192 Apr 14 09:00 deploy.sh
```

1. 文件类型是什么？
2. 所有者、所属组分别是谁？
3. 所有者有什么权限？
4. developers 组的成员可以执行这个文件吗？
5. 不属于 developers 组的其他用户可以读取这个文件吗？

<details>
<summary>查看答案</summary>

1. **普通文件**（第一个字符 `-`）
2. 所有者：**alice**，所属组：**developers**
3. 所有者权限：**rwx**（读、写、执行）
4. **可以**。组权限为 `r-x`，有 execute 权限
5. **不可以**。其他用户权限为 `---`，没有任何权限

</details>

---

### 第 11 题（权限）

写出命令完成以下任务：

1. 将 `script.sh` 设置为所有者可读写执行、组可读执行、其他人无权限（用数字模式）
2. 给 `data.txt` 的所属组添加写权限（用符号模式，不影响其他权限）
3. 递归移除 `private/` 目录中所有文件对 others 的所有权限

<details>
<summary>查看答案</summary>

```bash
# 1
chmod 750 script.sh

# 2
chmod g+w data.txt

# 3
chmod -R o-rwx private/
```

</details>

---

### 第 12 题（权限 — 选择题）

`umask 027` 时，新创建的**目录**的默认权限是什么？

A. `777`
B. `750`
C. `640`
D. `027`

<details>
<summary>查看答案</summary>

**B** — `750`（`rwxr-x---`）

目录的默认基础权限为 `777`，减去 umask `027`：
- owner: 7 - 0 = 7 (rwx)
- group: 7 - 2 = 5 (r-x)
- others: 7 - 7 = 0 (---)

新文件的默认权限为 `666 - 027 = 640`（`rw-r-----`）。

</details>

---

### 第 13 题（权限）

为什么 `/tmp` 目录的权限是 `drwxrwxrwt`？最后的 `t` 代表什么？

<details>
<summary>查看答案</summary>

最后的 `t` 代表 **Sticky Bit**。

`/tmp` 是公共临时目录，所有用户都需要在其中创建和使用临时文件，因此权限为 `777`（所有人可读写执行）。

但如果没有 Sticky Bit，任何用户都可以删除其他用户的文件。Sticky Bit 确保**只有文件的所有者（和 root）才能删除该文件**，即使目录本身对所有人开放写权限。

设置 Sticky Bit：`chmod +t dir` 或 `chmod 1777 dir`

</details>

---

### 第 14 题（重定向）

写出以下操作对应的重定向语法：

1. 将 `ls` 的输出保存到 `files.txt`（覆盖）
2. 将 `echo "log entry"` 追加到 `app.log`
3. 将 `find /` 的错误信息丢弃，只显示正常结果
4. 将命令的 stdout 和 stderr 都保存到 `all.log`

<details>
<summary>查看答案</summary>

```bash
# 1
ls > files.txt

# 2
echo "log entry" >> app.log

# 3
find / 2>/dev/null

# 4
command > all.log 2>&1
# or: command &> all.log
```

</details>

---

### 第 15 题（重定向 — 选择题）

以下哪条命令会导致文件内容丢失？

A. `cat data.txt >> backup.txt`
B. `echo "new" > data.txt`
C. `wc -l < data.txt`
D. `grep "error" data.txt | tee result.txt`

<details>
<summary>查看答案</summary>

**B**

`>` 是覆盖模式，会先清空 `data.txt` 的全部内容，然后写入 "new"。

- A：`>>` 是追加模式，不影响已有内容
- C：`<` 是输入重定向，只读取文件，不修改
- D：`tee` 会覆盖 `result.txt`，但 `data.txt` 不受影响

</details>

---

### 第 16 题（管道）

写出一条管道命令：从 `access.log` 中提取所有包含 "404" 的行，统计数量，并将结果同时显示在屏幕上和保存到 `report.txt`。

<details>
<summary>查看答案</summary>

```bash
grep "404" access.log | wc -l | tee report.txt
```

管道链解析：
1. `grep "404" access.log` — 过滤包含 "404" 的行
2. `wc -l` — 统计行数
3. `tee report.txt` — 输出到屏幕并保存到文件

</details>

---

### 第 17 题（管道 — 情景题）

你需要找出系统中占用内存最多的前 5 个进程。写出完整的管道命令。

<details>
<summary>查看答案</summary>

```bash
ps aux --sort=-%mem | head -6
```

- `ps aux` — 列出所有进程
- `--sort=-%mem` — 按内存使用率降序排序（`-` 表示逆序）
- `head -6` — 取前 6 行（第 1 行是标题行，后面 5 行是进程）

替代方案（使用管道和 `sort`）：

```bash
ps aux | sort -k 4 -rn | head -5
```

- `sort -k 4 -rn` — 按第 4 列（%MEM）数字逆序排列

</details>

---

### 第 18 题（重定向 — 情景题）

你需要用 `sudo` 将一行文本追加到 `/etc/hosts` 文件中。为什么 `sudo echo "127.0.0.1 myapp" >> /etc/hosts` 会失败？正确的写法是什么？

<details>
<summary>查看答案</summary>

失败原因：重定向 `>>` 是由**当前 Shell**（普通用户权限）执行的，`sudo` 只提升了 `echo` 命令的权限。Shell 没有 root 权限，无法写入 `/etc/hosts`。

正确的写法：

```bash
echo "127.0.0.1 myapp" | sudo tee -a /etc/hosts
```

`tee -a` 在 `sudo` 提升的权限下运行，因此可以追加写入受保护的文件。`-a` 表示追加模式（不加 `-a` 是覆盖模式）。

</details>

---

### 第 19 题（获取帮助）

1. 如何查看 `passwd` 命令的用法？
2. 如何查看 `/etc/passwd` 文件格式的说明？
3. 你想压缩文件但不知道用什么命令，如何搜索？
4. `cd` 是 Shell Builtin，用什么命令查看它的帮助？

<details>
<summary>查看答案</summary>

```bash
# 1
man passwd        # or: man 1 passwd

# 2
man 5 passwd

# 3
apropos compress  # or: man -k compress

# 4
help cd
```

</details>

---

### 第 20 题（综合情景题）

你加入了一个新项目，需要完成以下操作。写出每一步的命令：

1. 在主目录下创建项目目录结构 `~/myproject/src/` 和 `~/myproject/logs/`
2. 进入 `src` 目录，创建 `app.sh` 文件并写入一行内容 `#!/bin/bash`
3. 给 `app.sh` 添加可执行权限（仅所有者）
4. 运行 `app.sh`，将正常输出保存到 `../logs/output.log`，错误输出保存到 `../logs/error.log`
5. 查看 `output.log` 的行数
6. 查找系统中与 "process" 相关的命令

<details>
<summary>查看答案</summary>

```bash
# 1
mkdir -p ~/myproject/src ~/myproject/logs

# 2
cd ~/myproject/src
echo '#!/bin/bash' > app.sh

# 3
chmod u+x app.sh

# 4
./app.sh > ../logs/output.log 2> ../logs/error.log

# 5
wc -l ../logs/output.log

# 6
apropos process
```

</details>

---

## 评分标准

| 得分 | 评价 |
|------|------|
| 90-100 | ⭐ 优秀！基础扎实，可以进入中级课程 |
| 75-89 | 👍 良好，大部分知识已掌握，建议回顾错题对应的课程 |
| 60-74 | 📖 及格，建议重新学习薄弱章节后再继续 |
| 60 以下 | 🔄 建议从头复习 01-07 课，重点练习实战演练部分 |

### 知识点分布

| 课程 | 对应题目 | 分值 |
|------|----------|------|
| 01 - 终端与 Shell | 第 1 题 | 5 分 |
| 02 - 文件系统导航 | 第 2、3 题 | 10 分 |
| 03 - 文件操作 | 第 4、5、6 题 | 15 分 |
| 04 - 查看文件内容 | 第 7、8、9 题 | 15 分 |
| 05 - 文件权限与所有权 | 第 10、11、12、13 题 | 20 分 |
| 06 - I/O Redirection 与 Pipe | 第 14、15、16、17、18 题 | 25 分 |
| 07 - 获取帮助 | 第 19 题 | 5 分 |
| 综合 | 第 20 题 | 5 分 |

> 完成测验后，无论得分如何，都建议回顾错题对应的课程内容。接下来可以进入中级课程，从 [第 09 课：grep 与正则表达式](../intermediate/09-grep-and-regex.md) 开始学习！
