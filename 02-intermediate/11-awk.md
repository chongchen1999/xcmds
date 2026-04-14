# 第 11 课：数据处理工具 awk

> 上一课：[第 10 课：流编辑器 sed](./10-sed.md) | 下一课：[第 12 课：Process 管理](./12-process-management.md)

---

## 学习目标

1. 掌握 `awk` 的基本语法和字段分割机制
2. 理解字段变量（`$0`、`$1`、`$NF`）和内置变量（`NR`、`NF`、`FS`）
3. 学会使用 `BEGIN` / `END` 块进行初始化和汇总
4. 掌握模式匹配与条件过滤
5. 能用 `awk` 处理 CSV、日志分析、生成报表等实际任务

---

## 知识讲解

### 1. awk 基础

`awk` 是一种面向行和列的文本处理语言。它将每一行（record）按分隔符拆分为多个字段（field），然后逐行执行指定动作。

基本语法：

```bash
awk 'pattern { action }' file
```

最简单的例子：

```bash
# print the first column
awk '{print $1}' file.txt
```

### 2. 字段与记录

| 变量 | 含义 |
|------|------|
| `$0` | 当前整行内容 |
| `$1` | 第 1 个字段 |
| `$2` | 第 2 个字段 |
| `$NF` | 最后一个字段 |
| `$(NF-1)` | 倒数第 2 个字段 |
| `NR` | 当前行号（Number of Record） |
| `NF` | 当前行的字段数（Number of Field） |

默认分隔符是空白字符（空格和 Tab）。

```bash
# print line number, first and last field
awk '{print NR, $1, $NF}' file.txt
```

### 3. 指定分隔符

使用 `-F` 选项或 `FS` 变量指定输入分隔符：

```bash
# use colon as delimiter
awk -F: '{print $1, $3}' /etc/passwd

# use comma for CSV
awk -F, '{print $2}' data.csv
```

### 4. 模式匹配

```bash
# only process lines matching regex
awk '/ERROR/ {print $0}' app.log

# conditional on field value
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# string comparison
awk -F, '$4 == "admin" {print $2}' users.csv

# combined conditions
awk -F, '$4 == "admin" && $1 > 2 {print $2}' users.csv
```

### 5. BEGIN 和 END 块

- `BEGIN` —— 在处理任何输入行**之前**执行，常用于初始化变量和打印表头
- `END` —— 在处理完所有行**之后**执行，常用于输出汇总

```bash
awk 'BEGIN {print "=== Report ==="} {print $0} END {print "Total:", NR, "lines"}' file.txt
```

统计示例：

```bash
# sum values in column 3
awk '{sum += $3} END {print "Total:", sum}' data.txt

# calculate average
awk '{sum += $3; count++} END {print "Average:", sum/count}' data.txt
```

### 6. 内置变量详解

| 变量 | 含义 | 默认值 |
|------|------|--------|
| `FS` | 输入字段分隔符（Field Separator） | 空白 |
| `OFS` | 输出字段分隔符（Output Field Separator） | 空格 |
| `RS` | 输入记录分隔符（Record Separator） | 换行 |
| `ORS` | 输出记录分隔符 | 换行 |
| `FILENAME` | 当前处理的文件名 | — |

```bash
# set input and output separator
awk 'BEGIN {FS=","; OFS="\t"} {print $1, $2, $3}' data.csv
```

### 7. printf 格式化输出

`awk` 的 `printf` 与 C 语言语法一致：

| 格式符 | 含义 |
|--------|------|
| `%s` | 字符串 |
| `%d` | 整数 |
| `%f` | 浮点数 |
| `%.2f` | 保留 2 位小数 |
| `%-10s` | 左对齐，宽度 10 |

```bash
awk -F, '{printf "%-10s %-20s %s\n", $1, $2, $3}' users.csv
```

### 8. awk 脚本中的流程控制

```bash
# if-else
awk -F, '{
    if ($4 == "admin")
        print $2, "-> Administrator"
    else
        print $2, "-> Regular User"
}' users.csv

# for loop
awk '{for (i=1; i<=NF; i++) print NR":"i, $i}' file.txt
```

---

## 实战演练

### 练习环境准备

```bash
mkdir -p ~/awk-lab && cd ~/awk-lab

cat > employees.csv << 'EOF'
id,name,department,salary
101,Alice,Engineering,95000
102,Bob,Marketing,72000
103,Charlie,Engineering,88000
104,Dave,Sales,68000
105,Eve,Engineering,102000
106,Frank,Marketing,75000
107,Grace,Sales,71000
108,Hank,Engineering,91000
EOF

cat > access.log << 'EOF'
192.168.1.10 - - [15/Jan/2024:08:23:01] "GET /index.html HTTP/1.1" 200 1234
192.168.1.20 - - [15/Jan/2024:08:25:33] "GET /api/users HTTP/1.1" 200 5678
192.168.1.10 - - [15/Jan/2024:08:30:12] "POST /api/login HTTP/1.1" 401 89
10.0.0.5 - - [15/Jan/2024:08:30:15] "GET /index.html HTTP/1.1" 200 1234
192.168.1.20 - - [15/Jan/2024:08:35:44] "GET /api/users HTTP/1.1" 200 5678
10.0.0.5 - - [15/Jan/2024:09:15:01] "GET /static/style.css HTTP/1.1" 304 0
192.168.1.10 - - [15/Jan/2024:09:20:22] "DELETE /api/users/3 HTTP/1.1" 403 120
192.168.1.30 - - [15/Jan/2024:09:25:00] "GET /index.html HTTP/1.1" 200 1234
EOF
```

### 步骤 1：基本字段提取

```bash
# print name and salary
awk -F, 'NR > 1 {print $2, $4}' employees.csv

# formatted output with header
awk -F, '
    NR == 1 {printf "%-10s %-15s %s\n", "Name", "Department", "Salary"; next}
    {printf "%-10s %-15s $%s\n", $2, $3, $4}
' employees.csv
```

### 步骤 2：条件过滤

```bash
# Engineering department only
awk -F, '$3 == "Engineering"' employees.csv

# salary > 80000
awk -F, 'NR > 1 && $4 > 80000 {print $2, "$"$4}' employees.csv
```

### 步骤 3：统计汇总

```bash
# total salary
awk -F, 'NR > 1 {sum += $4} END {printf "Total payroll: $%d\n", sum}' employees.csv

# average salary
awk -F, 'NR > 1 {sum += $4; n++} END {printf "Average salary: $%.0f\n", sum/n}' employees.csv

# department salary totals
awk -F, 'NR > 1 {dept[$3] += $4} END {for (d in dept) printf "%-15s $%d\n", d, dept[d]}' employees.csv
```

### 步骤 4：日志分析

```bash
# count requests per IP
awk '{ip[$1]++} END {for (i in ip) print i, ip[i]}' access.log

# extract unique URLs
awk '{print $7}' access.log | sort -u

# filter non-200 status codes
awk '$9 != 200 {print $1, $7, $9}' access.log

# total bytes transferred
awk '{bytes += $10} END {printf "Total: %d bytes (%.2f KB)\n", bytes, bytes/1024}' access.log
```

### 步骤 5：生成报表

```bash
awk -F, '
    BEGIN {
        printf "%-5s %-10s %-15s %10s\n", "ID", "Name", "Dept", "Salary"
        printf "%s\n", "-------------------------------------------"
    }
    NR > 1 {
        printf "%-5s %-10s %-15s %10s\n", $1, $2, $3, "$"$4
        sum += $4; count++
    }
    END {
        printf "%s\n", "-------------------------------------------"
        printf "%-30s %10s\n", "Total:", "$"sum
        printf "%-30s %10s\n", "Average:", "$"int(sum/count)
        printf "%-30s %10s\n", "Employees:", count
    }
' employees.csv
```

### 步骤 6：CSV 转换格式

```bash
# CSV to TSV
awk -F, 'BEGIN {OFS="\t"} {$1=$1; print}' employees.csv

# CSV to JSON-like
awk -F, 'NR > 1 {
    printf "{\"id\": %s, \"name\": \"%s\", \"dept\": \"%s\", \"salary\": %s}\n", $1, $2, $3, $4
}' employees.csv
```

### 清理

```bash
rm -rf ~/awk-lab
```

---

## 小结

| 命令 / 语法 | 作用 |
|-------------|------|
| `awk '{print $1}' file` | 打印第一个字段 |
| `awk -F, '{...}'` | 指定逗号为分隔符 |
| `$0` / `$1` / `$NF` | 整行 / 第一字段 / 最后字段 |
| `NR` / `NF` | 行号 / 当前行字段数 |
| `awk '/pattern/ {action}'` | 模式匹配后执行动作 |
| `BEGIN {...}` | 处理输入前执行 |
| `END {...}` | 处理完所有输入后执行 |
| `FS` / `OFS` | 输入 / 输出字段分隔符 |
| `printf "%s", var` | 格式化输出 |
| `array[key] += val` | 关联数组累加（统计利器） |

---

## 练习

**1. 如何用 awk 打印 `/etc/passwd` 中所有用户名（第 1 个字段，以 `:` 分隔）？**

<details>
<summary>查看答案</summary>

```bash
awk -F: '{print $1}' /etc/passwd
```

`-F:` 将分隔符设为冒号，`$1` 取第一个字段。

</details>

**2. 有一个空格分隔的文件 `scores.txt`，每行格式为 `姓名 分数`。如何计算所有人的平均分？**

<details>
<summary>查看答案</summary>

```bash
awk '{sum += $2; n++} END {print "Average:", sum/n}' scores.txt
```

逐行累加第二列的值，在 `END` 块中计算平均值。

</details>

**3. 如何用 awk 只打印 `access.log` 中 HTTP 状态码不是 200 的行？**

<details>
<summary>查看答案</summary>

```bash
awk '$9 != 200' access.log
```

假设状态码在第 9 个字段。`awk` 省略 action 时默认执行 `{print $0}`。

</details>

**4. 如何用 awk 统计一个文件中每个单词出现的次数？**

<details>
<summary>查看答案</summary>

```bash
awk '{for (i=1; i<=NF; i++) words[$i]++} END {for (w in words) print w, words[w]}' file.txt
```

遍历每行每个字段，用关联数组计数，最后在 `END` 块中输出。

</details>

**5. 如何将一个 CSV 文件（逗号分隔）转换为 Tab 分隔的 TSV 格式？**

<details>
<summary>查看答案</summary>

```bash
awk -F, 'BEGIN {OFS="\t"} {$1=$1; print}' input.csv
```

`OFS="\t"` 设置输出分隔符为 Tab。`$1=$1` 触发 awk 用新的 OFS 重新构建 `$0`。

</details>
