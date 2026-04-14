# 第 04 课：查看文件内容

> 上一课：[第 03 课：文件操作](./03-file-operations.md) | 下一课：[第 05 课：文件权限与所有权](./05-permissions.md)

## 学习目标

- 掌握 `cat` 查看和拼接文件内容
- 掌握 `less` / `more` 分页浏览大文件
- 掌握 `head` / `tail` 查看文件的开头和结尾
- 掌握 `wc` 统计文件的行数、单词数和字节数
- 掌握 `file` 判断文件类型
- 掌握 `diff` 比较两个文件的差异

---

## 知识讲解

### 1. `cat` — Concatenate and Display

`cat` 是最常用的文件查看命令，一次性输出整个文件内容：

```bash
cat file.txt                 # display file content
cat file1.txt file2.txt      # display multiple files sequentially
cat -n file.txt              # show with line numbers
cat -b file.txt              # show line numbers for non-empty lines only
```

`cat` 还可以用于创建文件和拼接文件：

```bash
# create file with inline content (Ctrl+D to finish)
cat > newfile.txt
This is line 1
This is line 2
^D

# append content to existing file
cat >> newfile.txt
This is line 3
^D

# concatenate multiple files into one
cat header.txt body.txt footer.txt > complete.txt
```

> ⚠️ `cat` 会一次性输出所有内容，不适合查看大文件。大文件请使用 `less`。

### 2. `less` / `more` — 分页查看

#### `less` — 功能更强的分页器

```bash
less largefile.log
```

`less` 打开文件后的操作快捷键：

| 按键 | 操作 |
|------|------|
| `Space` / `f` | 向下翻一页 |
| `b` | 向上翻一页 |
| `j` / `↓` | 向下一行 |
| `k` / `↑` | 向上一行 |
| `g` | 跳到文件开头 |
| `G` | 跳到文件末尾 |
| `/pattern` | 向下搜索 pattern |
| `?pattern` | 向上搜索 pattern |
| `n` | 跳到下一个搜索匹配 |
| `N` | 跳到上一个搜索匹配 |
| `q` | 退出 |

#### `more` — 基础分页器

```bash
more largefile.log
```

`more` 功能较少，只能向前翻页（`Space`），按 `q` 退出。在现代系统中推荐使用 `less`。

> 有句经典的 Unix 格言："less is more"（`less` 比 `more` 强大）。

### 3. `head` / `tail` — 查看文件开头和结尾

#### `head` — 查看文件开头

```bash
head file.txt                # show first 10 lines (default)
head -n 5 file.txt           # show first 5 lines
head -n 20 file.txt          # show first 20 lines
head -c 100 file.txt         # show first 100 bytes
```

#### `tail` — 查看文件结尾

```bash
tail file.txt                # show last 10 lines (default)
tail -n 5 file.txt           # show last 5 lines
tail -n +3 file.txt          # show from line 3 to end
tail -c 100 file.txt         # show last 100 bytes
```

#### `tail -f` — 实时监控文件变化

```bash
tail -f /var/log/syslog      # follow mode: monitor new lines in real-time
tail -f -n 50 app.log        # show last 50 lines then follow
```

`tail -f` 是运维和开发中监控日志的核心工具。它会持续输出文件新增的内容，按 `Ctrl + C` 退出。

组合使用 `head` 和 `tail` 查看文件的中间部分：

```bash
# show lines 11-20 of a file
head -n 20 file.txt | tail -n 10

# show lines 50-60
sed -n '50,60p' file.txt     # alternative: using sed
```

### 4. `wc` — Word Count

统计文件的行数、单词数和字节数：

```bash
wc file.txt                  # lines, words, bytes (all three)
wc -l file.txt               # line count only
wc -w file.txt               # word count only
wc -c file.txt               # byte count only
wc -m file.txt               # character count
```

`wc` 输出格式：

```bash
wc file.txt
#  42  318  2048  file.txt
#  行   词   字节   文件名
```

常见用法：

```bash
# count files in a directory
ls | wc -l

# count lines in multiple files
wc -l *.txt

# count lines of code in a project
find . -name "*.py" | xargs wc -l
```

### 5. `file` — 判断文件类型

Linux 不依赖文件扩展名判断类型，`file` 命令通过分析文件内容（magic bytes）来判断：

```bash
file photo.jpg               # JPEG image data
file script.sh               # Bourne-Again shell script, ASCII text
file document.pdf            # PDF document, version 1.4
file /bin/ls                 # ELF 64-bit LSB executable
file data.csv                # ASCII text
file mystery                 # works even without extension
```

这在文件扩展名缺失或不可信时非常有用：

```bash
# someone renamed an image to .txt
file fake.txt
# output: JPEG image data (reveals the true type)
```

### 6. `diff` — 比较文件差异

```bash
diff file1.txt file2.txt             # basic comparison
diff -u file1.txt file2.txt          # unified format (most readable)
diff -y file1.txt file2.txt          # side-by-side comparison
diff -r dir1/ dir2/                  # compare directories recursively
diff --color file1.txt file2.txt     # colorized output
```

#### Unified diff 格式说明

```bash
diff -u original.txt modified.txt
```

输出示例：

```diff
--- original.txt    2026-04-14 10:00:00
+++ modified.txt    2026-04-14 10:05:00
@@ -1,4 +1,4 @@
 line 1
-line 2 old version
+line 2 new version
 line 3
-line 4 removed
+line 4 changed
```

- `---` 标识原始文件
- `+++` 标识修改后的文件
- `-` 开头的行是被删除或修改前的内容
- `+` 开头的行是新增或修改后的内容
- 无前缀的行是未改变的上下文

> `diff -u` 的输出格式就是 Git 和 patch 工具使用的标准格式。

---

## 实战演练

模拟一个日志分析场景：

**步骤 1：创建练习环境**

```bash
mkdir -p ~/view-practice && cd ~/view-practice
```

**步骤 2：生成一个模拟日志文件**

```bash
for i in $(seq 1 100); do
  echo "$(date '+%Y-%m-%d %H:%M:%S') [INFO] Processing request #$i"
done > app.log

echo "$(date '+%Y-%m-%d %H:%M:%S') [ERROR] Connection timeout" >> app.log
echo "$(date '+%Y-%m-%d %H:%M:%S') [WARN] Retry attempt 1" >> app.log
echo "$(date '+%Y-%m-%d %H:%M:%S') [ERROR] Connection refused" >> app.log
```

**步骤 3：用 `cat` 查看（体验大文件的不便）**

```bash
cat app.log
```

内容太多，滚屏了。

**步骤 4：用 `less` 优雅地浏览**

```bash
less app.log
```

进入后试试：`Space` 翻页、`/ERROR` 搜索错误、`n` 跳到下一个匹配、`q` 退出。

**步骤 5：用 `head` 和 `tail` 查看关键部分**

```bash
head -n 5 app.log       # first 5 lines
tail -n 5 app.log       # last 5 lines (likely the errors)
```

**步骤 6：统计日志信息**

```bash
wc -l app.log            # total line count
grep -c "ERROR" app.log  # count ERROR lines
grep -c "INFO" app.log   # count INFO lines
```

**步骤 7：查看文件类型**

```bash
file app.log
```

输出类似：`app.log: ASCII text`

**步骤 8：创建对比文件并使用 `diff`**

```bash
head -n 5 app.log > version1.txt

# create a slightly modified version
cp version1.txt version2.txt
sed -i '3s/INFO/ERROR/' version2.txt    # change line 3

diff -u version1.txt version2.txt
```

你会看到第 3 行的 `INFO` 变成 `ERROR` 的差异。

**步骤 9：组合命令——查看日志第 50-55 行**

```bash
head -n 55 app.log | tail -n 6
```

先取前 55 行，再从结果中取最后 6 行，即第 50-55 行。

**步骤 10：清理**

```bash
cd ~
rm -r ~/view-practice
```

---

## 小结

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `cat` | 查看 / 拼接文件 | `-n`（行号） |
| `less` | 分页浏览（推荐） | `/`（搜索）、`q`（退出） |
| `more` | 基础分页浏览 | — |
| `head` | 查看文件开头 | `-n N`（前 N 行） |
| `tail` | 查看文件结尾 | `-n N`（后 N 行）、`-f`（实时跟踪） |
| `wc` | 统计行/词/字节数 | `-l`（行）、`-w`（词）、`-c`（字节） |
| `file` | 判断文件类型 | — |
| `diff` | 比较文件差异 | `-u`（unified）、`-y`（并排）、`-r`（递归目录） |

---

## 练习

**1. 如何查看一个文件的第 20 到 30 行？**

<details>
<summary>查看答案</summary>

```bash
head -n 30 file.txt | tail -n 11
```

或者使用 `sed`：

```bash
sed -n '20,30p' file.txt
```

`head -n 30` 取前 30 行，`tail -n 11` 从中取最后 11 行（即第 20-30 行）。

</details>

**2. `tail -f` 有什么用途？如何退出？**

<details>
<summary>查看答案</summary>

`tail -f` 以 follow 模式运行，实时监控文件末尾的新增内容。常用于监控日志文件：

```bash
tail -f /var/log/syslog
```

当有新日志写入时，会立即显示在终端。按 `Ctrl + C` 退出。

</details>

**3. `wc -l` 和 `cat -n` 最后一行的行号一定相同吗？为什么？**

<details>
<summary>查看答案</summary>

不一定相同。`wc -l` 统计的是换行符（`\n`）的数量。如果文件最后一行没有以换行符结尾，`wc -l` 的结果会比 `cat -n` 显示的最后行号少 1。

例如：一个文件有 3 行文本但最后一行没有换行符，`cat -n` 显示行号到 3，但 `wc -l` 只输出 2。

</details>

**4. 有一个文件没有扩展名，如何判断它是图片还是文本文件？**

<details>
<summary>查看答案</summary>

使用 `file` 命令：

```bash
file mystery_file
```

`file` 通过分析文件的 magic bytes（文件头部的特征字节）来判断文件类型，不依赖扩展名。输出示例：
- `mystery_file: JPEG image data` — 图片
- `mystery_file: ASCII text` — 文本文件
- `mystery_file: PNG image data, 1920 x 1080` — PNG 图片

</details>

**5. `diff -u` 输出中，`-` 和 `+` 开头的行分别代表什么？**

<details>
<summary>查看答案</summary>

- `-` 开头的行：存在于**第一个文件**中但在第二个文件中被修改或删除的内容
- `+` 开头的行：存在于**第二个文件**中的新增或修改后的内容
- 没有前缀的行：两个文件中相同的上下文内容

例如：

```diff
-old line content
+new line content
```

表示 `old line content` 被替换为 `new line content`。

</details>
