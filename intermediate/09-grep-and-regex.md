# 第 09 课：文本搜索 grep 与 Regex

> 上一课：[第 08 课：初级综合测验](../beginner/08-beginner-quiz.md) | 下一课：[第 10 课：流编辑器 sed](./10-sed.md)

---

## 学习目标

1. 掌握 `grep` 的基本用法和常用选项
2. 理解 Basic Regular Expression（BRE）的核心语法
3. 掌握 Extended Regular Expression（ERE）的扩展语法
4. 能够在实际场景中使用 `grep` 进行高效文本搜索

---

## 知识讲解

### 1. grep 基础

`grep`（**G**lobal **R**egular **E**xpression **P**rint）用于在文件或输入流中搜索匹配特定模式的行。

基本语法：

```bash
grep [选项] pattern [文件...]
```

最简单的用法——在文件中搜索字符串：

```bash
grep "error" /var/log/syslog
```

从标准输入中搜索：

```bash
dmesg | grep "usb"
```

### 2. 常用选项

| 选项 | 作用 | 示例 |
|------|------|------|
| `-i` | 忽略大小写（case-insensitive） | `grep -i "error" log.txt` |
| `-n` | 显示匹配行的行号 | `grep -n "TODO" main.py` |
| `-r` | 递归搜索目录 | `grep -r "config" /etc/` |
| `-v` | 反向匹配（输出不匹配的行） | `grep -v "^#" config.conf` |
| `-c` | 只输出匹配的行数 | `grep -c "warning" log.txt` |
| `-l` | 只输出包含匹配的文件名 | `grep -rl "TODO" src/` |
| `-w` | 全词匹配（word match） | `grep -w "is" text.txt` |
| `-E` | 使用 Extended Regex | `grep -E "err|warn" log.txt` |
| `-A n` | 显示匹配行后 n 行（After） | `grep -A 3 "error" log.txt` |
| `-B n` | 显示匹配行前 n 行（Before） | `grep -B 2 "error" log.txt` |
| `-C n` | 显示匹配行前后各 n 行（Context） | `grep -C 2 "error" log.txt` |

### 3. Basic Regular Expression（BRE）

BRE 是 `grep` 默认使用的正则表达式语法。

| 符号 | 含义 | 示例 |
|------|------|------|
| `.` | 匹配任意单个字符 | `grep "h.t" file` → hat, hot, hit |
| `*` | 前一个字符出现 0 次或多次 | `grep "go*d" file` → gd, god, good |
| `^` | 匹配行首 | `grep "^#" config` |
| `$` | 匹配行尾 | `grep ";$" code.c` |
| `[]` | 字符集，匹配其中任意一个 | `grep "[aeiou]" file` |
| `[^]` | 反向字符集 | `grep "[^0-9]" file` |
| `\b` | 单词边界（word boundary） | `grep "\bthe\b" file` |
| `\` | 转义特殊字符 | `grep "192\.168" hosts` |

常用字符集简写：

```bash
[0-9]      # digits
[a-z]      # lowercase letters
[A-Z]      # uppercase letters
[a-zA-Z]   # all letters
[[:digit:]] # POSIX class for digits
[[:alpha:]] # POSIX class for letters
[[:space:]] # POSIX class for whitespace
```

### 4. Extended Regular Expression（ERE）

使用 `grep -E` 或 `egrep` 启用 ERE，提供更多元字符：

| 符号 | 含义 | 示例 |
|------|------|------|
| `+` | 前一个字符出现 1 次或多次 | `grep -E "go+d" file` → god, good |
| `?` | 前一个字符出现 0 次或 1 次 | `grep -E "colou?r" file` → color, colour |
| `{n}` | 前一个字符恰好出现 n 次 | `grep -E "[0-9]{3}" file` |
| `{n,m}` | 前一个字符出现 n 到 m 次 | `grep -E "[0-9]{1,3}" file` |
| `\|` | 或（alternation） | `grep -E "cat\|dog" file` |
| `()` | 分组（grouping） | `grep -E "(ab)+" file` |

> **BRE vs ERE 对比**：在 BRE 中使用 `+`、`?`、`{}`、`|`、`()` 时需要用 `\` 转义，而 ERE 中直接使用。

```bash
# BRE: need to escape
grep 'go\+d' file

# ERE: no escape needed
grep -E 'go+d' file
```

### 5. fgrep —— 固定字符串搜索

`fgrep`（等同于 `grep -F`）将 pattern 视为纯文本字符串，不解释正则表达式。搜索包含正则特殊字符的文本时更快更安全：

```bash
# search for literal string "192.168.1.*"
fgrep "192.168.1.*" config.txt

grep -F "$variable_name" script.sh
```

---

## 实战演练

### 练习环境准备

```bash
mkdir -p ~/grep-lab && cd ~/grep-lab

# create sample log file
cat > app.log << 'EOF'
2024-01-15 08:23:01 INFO  Server started on port 8080
2024-01-15 08:25:33 WARN  Connection pool reaching limit: 85%
2024-01-15 08:30:12 ERROR Failed to connect to database: timeout
2024-01-15 08:30:15 ERROR Retrying database connection (attempt 2)
2024-01-15 08:30:18 INFO  Database connection restored
2024-01-15 09:15:44 WARN  High memory usage: 92%
2024-01-15 09:20:01 ERROR OutOfMemoryError: heap space
2024-01-15 09:20:05 INFO  GC completed, memory freed
2024-01-15 10:00:00 INFO  Scheduled backup started
2024-01-15 10:05:30 INFO  Backup completed successfully
EOF

# create sample config file
cat > nginx.conf << 'EOF'
# Nginx configuration
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;

    location /api {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }

    location /static {
        expires 30d;
        access_log off;
    }
}
EOF
```

### 步骤 1：基本搜索

```bash
# search for ERROR lines
grep "ERROR" app.log

# case-insensitive search
grep -i "error" app.log

# show line numbers
grep -n "ERROR" app.log
```

### 步骤 2：反向匹配与计数

```bash
# show lines that are NOT INFO
grep -v "INFO" app.log

# count ERROR lines
grep -c "ERROR" app.log
```

### 步骤 3：上下文显示

```bash
# show 1 line after each ERROR
grep -A 1 "ERROR" app.log

# show 1 line before and after
grep -C 1 "ERROR" app.log
```

### 步骤 4：使用正则表达式

```bash
# match lines starting with date 2024-01-15 08
grep "^2024-01-15 08" app.log

# match lines containing percentage values
grep -E "[0-9]+%" app.log

# match time pattern HH:MM:SS
grep -E "[0-9]{2}:[0-9]{2}:[0-9]{2}" app.log
```

### 步骤 5：实际场景

```bash
# filter out comment lines and blank lines from config
grep -v -E "^#|^$" nginx.conf

# find lines with IP addresses
grep -E "\b[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\b" nginx.conf

# search for proxy-related settings
grep -n "proxy" nginx.conf
```

### 步骤 6：递归搜索

```bash
# search all .log files in current directory tree
grep -r "ERROR" --include="*.log" ~/grep-lab/

# list files containing a pattern
grep -rl "server" ~/grep-lab/
```

### 清理

```bash
rm -rf ~/grep-lab
```

---

## 小结

| 命令 / 选项 | 作用 |
|-------------|------|
| `grep pattern file` | 在文件中搜索匹配行 |
| `grep -i` | 忽略大小写 |
| `grep -n` | 显示行号 |
| `grep -r` | 递归搜索目录 |
| `grep -v` | 反向匹配 |
| `grep -c` | 输出匹配行数 |
| `grep -l` | 只输出匹配的文件名 |
| `grep -w` | 全词匹配 |
| `grep -E` / `egrep` | 使用 Extended Regex |
| `grep -F` / `fgrep` | 固定字符串搜索（不解释正则） |
| `grep -A/B/C n` | 显示上下文行 |
| `.` `*` `^` `$` `[]` | BRE 基本元字符 |
| `+` `?` `{}` `\|` `()` | ERE 扩展元字符 |

---

## 练习

**1. 如何在 `/var/log/syslog` 中搜索所有包含 "error" 或 "Error" 或 "ERROR" 的行？**

<details>
<summary>查看答案</summary>

```bash
grep -i "error" /var/log/syslog
```

使用 `-i` 选项忽略大小写即可匹配所有变体。

</details>

**2. 如何在当前目录下递归查找所有 `.py` 文件中包含 `import os` 的文件，只输出文件名？**

<details>
<summary>查看答案</summary>

```bash
grep -rl "import os" --include="*.py" .
```

`-r` 递归搜索，`-l` 只输出文件名，`--include` 限定文件类型。

</details>

**3. 写一个 grep 命令，从 `access.log` 中提取所有 HTTP 状态码为 4xx 或 5xx 的行。**

<details>
<summary>查看答案</summary>

```bash
grep -E "\b[45][0-9]{2}\b" access.log
```

使用 ERE，`[45]` 匹配 4 或 5 开头，`[0-9]{2}` 匹配后两位数字，`\b` 确保是独立的三位数。

</details>

**4. 如何用 grep 显示 `/etc/passwd` 中所有非注释行（不以 `#` 开头）且非空行？**

<details>
<summary>查看答案</summary>

```bash
grep -v -E "^#|^$" /etc/passwd
```

`-v` 反向匹配，`^#` 匹配注释行，`^$` 匹配空行，`|` 表示"或"。

</details>

**5. 如何在 `app.log` 中找到包含 "ERROR" 的行，同时显示其后 2 行内容？**

<details>
<summary>查看答案</summary>

```bash
grep -A 2 "ERROR" app.log
```

`-A 2` 表示显示匹配行之后的 2 行（After 2 lines）。

</details>
