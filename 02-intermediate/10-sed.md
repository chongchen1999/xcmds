# 第 10 课：流编辑器 sed

> 上一课：[第 09 课：文本搜索 grep 与 Regex](./09-grep-and-regex.md) | 下一课：[第 11 课：数据处理工具 awk](./11-awk.md)

---

## 学习目标

1. 掌握 `sed` 的基本替换语法
2. 理解地址范围（address range）的用法
3. 学会使用 `-i` 进行原地编辑
4. 掌握删除、插入、追加等操作
5. 能在实际场景中使用 `sed` 批量处理文本

---

## 知识讲解

### 1. sed 基础

`sed`（**S**tream **Ed**itor）逐行读取输入，对匹配的行执行编辑命令，然后输出结果。默认不修改原文件。

基本语法：

```bash
sed [选项] '命令' [文件...]
```

最常用的命令是替换（substitute）：

```bash
sed 's/old/new/' file
```

这会将每行中**第一个**匹配的 `old` 替换为 `new`。

### 2. 替换命令详解

```bash
sed 's/pattern/replacement/flags' file
```

常用 flags：

| Flag | 作用 |
|------|------|
| `g` | 替换行内所有匹配（global），不加只替换第一个 |
| `i` | 忽略大小写（GNU sed 扩展） |
| `p` | 打印被替换的行（常配合 `-n` 使用） |
| `数字` | 只替换第 n 个匹配 |

```bash
# replace all occurrences in each line
sed 's/foo/bar/g' file.txt

# case-insensitive replace
sed 's/error/WARNING/gi' log.txt

# replace only the 2nd occurrence per line
sed 's/the/THE/2' file.txt
```

### 3. 常用选项

| 选项 | 作用 |
|------|------|
| `-n` | 静默模式，不自动打印每行（配合 `p` flag 使用） |
| `-i` | 原地编辑文件（in-place）|
| `-i.bak` | 原地编辑并创建 `.bak` 备份 |
| `-e` | 指定多个编辑命令 |
| `-f script.sed` | 从文件读取 sed 命令 |
| `-E` / `-r` | 使用 Extended Regex |

```bash
# in-place edit with backup
sed -i.bak 's/debug/info/g' app.conf

# multiple commands
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt
```

### 4. 地址范围（Address Range）

sed 可以指定只对特定行执行操作：

```bash
# line 3 only
sed '3s/old/new/' file

# lines 2 to 5
sed '2,5s/old/new/' file

# from line 3 to end of file
sed '3,$s/old/new/' file

# lines matching pattern
sed '/^#/d' file

# range between two patterns
sed '/START/,/END/s/old/new/g' file
```

### 5. 删除行（d 命令）

```bash
# delete line 5
sed '5d' file

# delete lines 10-20
sed '10,20d' file

# delete blank lines
sed '/^$/d' file

# delete comment lines
sed '/^#/d' file

# delete from pattern to end of file
sed '/pattern/,$d' file
```

### 6. 插入与追加

```bash
# insert line BEFORE line 3 (i command)
sed '3i\This is inserted before line 3' file

# append line AFTER line 3 (a command)
sed '3a\This is appended after line 3' file

# append after matching line
sed '/\[section\]/a\new_key=value' config.ini
```

### 7. 其他实用命令

```bash
# print specific lines (with -n)
sed -n '5,10p' file

# replace delimiter: use any character
sed 's|/usr/local|/opt|g' file

# capture groups with & (entire match)
sed 's/[0-9]*/(&)/' file

# capture groups with \1, \2
sed -E 's/^([a-z]+):([0-9]+)/\2:\1/' file
```

---

## 实战演练

### 练习环境准备

```bash
mkdir -p ~/sed-lab && cd ~/sed-lab

cat > server.conf << 'EOF'
# Server Configuration
host=localhost
port=8080
debug=true
log_level=DEBUG
max_connections=100
timeout=30
# Database
db_host=localhost
db_port=3306
db_name=myapp
db_user=root
db_pass=secret123
EOF

cat > users.csv << 'EOF'
id,name,email,role
1,Alice,alice@example.com,admin
2,Bob,bob@example.com,user
3,Charlie,charlie@example.com,user
4,Dave,dave@example.com,admin
5,Eve,eve@test.org,user
EOF
```

### 步骤 1：基本替换

```bash
# replace localhost with 192.168.1.100
sed 's/localhost/192.168.1.100/' server.conf

# global replace
sed 's/localhost/192.168.1.100/g' server.conf
```

### 步骤 2：原地编辑

```bash
# create a working copy
cp server.conf server-test.conf

# in-place edit with backup
sed -i.bak 's/debug=true/debug=false/' server-test.conf

# verify
diff server-test.conf server-test.conf.bak
```

### 步骤 3：按地址范围操作

```bash
# show only lines 8-12 (Database section)
sed -n '8,12p' server.conf

# replace only within Database section
sed '/^# Database/,$ s/localhost/db-server/' server.conf
```

### 步骤 4：删除操作

```bash
# remove comment lines and blank lines
sed '/^#/d; /^$/d' server.conf

# remove the db_pass line
sed '/db_pass/d' server.conf
```

### 步骤 5：多重编辑

```bash
# change port and toggle debug in one pass
sed -e 's/port=8080/port=9090/' \
    -e 's/debug=true/debug=false/' \
    -e 's/log_level=DEBUG/log_level=INFO/' \
    server.conf
```

### 步骤 6：使用正则与分组

```bash
# wrap all values in quotes: key=value -> key="value"
sed -E 's/^([a-z_]+)=(.+)$/\1="\2"/' server.conf

# mask password
sed -E 's/(db_pass=).*/\1****/' server.conf

# swap CSV columns: id,name -> name,id (first two)
sed -E 's/^([^,]+),([^,]+)/\2,\1/' users.csv
```

### 步骤 7：实际场景 —— 批量修改配置

```bash
# simulate deploying to production
cp server.conf prod.conf
sed -i \
    -e 's/host=localhost/host=0.0.0.0/' \
    -e 's/port=8080/port=80/' \
    -e 's/debug=true/debug=false/' \
    -e 's/log_level=DEBUG/log_level=WARN/' \
    -e 's/db_host=localhost/db_host=db.prod.internal/' \
    -e '/db_pass/s/=.*/=VAULT_SECRET/' \
    prod.conf

cat prod.conf
```

### 清理

```bash
rm -rf ~/sed-lab
```

---

## 小结

| 命令 / 语法 | 作用 |
|-------------|------|
| `sed 's/old/new/' file` | 替换每行第一个匹配 |
| `sed 's/old/new/g' file` | 替换所有匹配 |
| `sed -i file` | 原地编辑 |
| `sed -i.bak` | 原地编辑并备份 |
| `sed -n 'Np'` | 打印第 N 行 |
| `sed 'Nd'` | 删除第 N 行 |
| `sed '/pattern/d'` | 删除匹配行 |
| `sed '/^$/d'` | 删除空行 |
| `sed 'Ni\text'` | 在第 N 行前插入 |
| `sed 'Na\text'` | 在第 N 行后追加 |
| `sed -e cmd1 -e cmd2` | 执行多条命令 |
| `sed -E 's/(group)/\1/'` | 使用 ERE 和捕获组 |

---

## 练习

**1. 如何用 sed 将文件 `data.txt` 中所有的 "apple" 替换为 "orange"（包括每行出现多次的情况）？**

<details>
<summary>查看答案</summary>

```bash
sed 's/apple/orange/g' data.txt
```

必须加 `g` flag 才能替换每行中的所有匹配，否则只替换每行第一个。

</details>

**2. 如何删除文件中所有空行和以 `#` 开头的注释行？**

<details>
<summary>查看答案</summary>

```bash
sed '/^$/d; /^#/d' file.txt
```

也可以写成：

```bash
sed -e '/^$/d' -e '/^#/d' file.txt
```

</details>

**3. 如何用 sed 只打印文件的第 5 到第 10 行？**

<details>
<summary>查看答案</summary>

```bash
sed -n '5,10p' file.txt
```

`-n` 关闭默认输出，`p` 只打印匹配范围内的行。

</details>

**4. 有一个配置文件，需要将 `timeout=30` 改为 `timeout=60`，并原地保存且保留备份，怎么做？**

<details>
<summary>查看答案</summary>

```bash
sed -i.bak 's/timeout=30/timeout=60/' config.conf
```

`-i.bak` 会在原地修改文件，并将原始文件保存为 `config.conf.bak`。

</details>

**5. 如何用 sed 将 `/etc/hosts` 格式的文件中，IP 地址 `192.168.1.10` 替换为 `10.0.0.10`？（注意点号需要转义）**

<details>
<summary>查看答案</summary>

```bash
sed 's/192\.168\.1\.10/10.0.0.10/g' hosts
```

在正则中 `.` 匹配任意字符，所以搜索模式中的点号需要用 `\.` 转义。替换字符串中的点号不需要转义。

</details>
