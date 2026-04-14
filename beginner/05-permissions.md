# 第 05 课：文件权限与所有权

> 上一课：[第 04 课：查看文件内容](./04-viewing-files.md) | 下一课：[第 06 课：I/O Redirection 与 Pipe](./06-redirection-and-pipes.md)

## 学习目标

- 理解 `ls -l` 输出格式中各字段的含义
- 掌握 read、write、execute 三种权限类型及其对文件和目录的不同含义
- 掌握 `chmod` 的数字模式和符号模式修改权限
- 掌握 `chown` 和 `chgrp` 修改文件所有者和所属组
- 了解 SUID、SGID、Sticky Bit 三种特殊权限
- 理解 `umask` 的作用及如何设置默认权限

---

## 知识讲解

### 1. `ls -l` 输出格式详解

```bash
ls -l
```

输出示例：

```
-rwxr-xr-- 2 alice developers 4096 Apr 14 10:00 script.sh
│└┬┘└┬┘└┬┘ │  │      │         │       │          │
│ │   │   │  │  │      │         │       │          └── filename
│ │   │   │  │  │      │         │       └── modification time
│ │   │   │  │  │      │         └── file size (bytes)
│ │   │   │  │  │      └── group owner
│ │   │   │  │  └── file owner
│ │   │   │  └── hard link count
│ │   │   └── others permissions (r--)
│ │   └── group permissions (r-x)
│ └── owner permissions (rwx)
└── file type (- = file, d = directory, l = symlink)
```

文件类型标识：

| 标识 | 类型 |
|------|------|
| `-` | 普通文件 |
| `d` | 目录 |
| `l` | 符号链接 |
| `c` | 字符设备 |
| `b` | 块设备 |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

### 2. 权限类型：r / w / x

三种基本权限对**文件**和**目录**有不同的含义：

| 权限 | 对文件的含义 | 对目录的含义 |
|------|-------------|-------------|
| `r` (read) | 读取文件内容 | 列出目录内容（`ls`） |
| `w` (write) | 修改文件内容 | 在目录中创建/删除文件 |
| `x` (execute) | 执行文件 | 进入目录（`cd`） |

权限分为三组，分别作用于不同的用户角色：

| 角色 | 缩写 | 说明 |
|------|------|------|
| Owner | `u` | 文件所有者 |
| Group | `g` | 所属组的成员 |
| Others | `o` | 其他所有用户 |

```bash
# -rwxr-xr--
#  |||  owner: rwx (read + write + execute)
#     ||| group: r-x (read + execute)
#        ||| others: r-- (read only)
```

> ⚠️ 目录的 `x` 权限非常重要。没有 `x` 权限就无法 `cd` 进入目录，即使有 `r` 权限也只能看到文件名而无法访问文件的元数据。

### 3. `chmod` — 修改权限

#### 数字模式（Octal Mode）

每种权限对应一个数值，三个权限位组成一个八进制数字：

| 权限 | 值 |
|------|----|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |
| `-` | 0 |

三组权限各自相加组成三位数字：

```bash
chmod 755 script.sh    # owner=rwx(7), group=r-x(5), others=r-x(5)
chmod 644 config.txt   # owner=rw-(6), group=r--(4), others=r--(4)
chmod 700 private/     # owner=rwx(7), group=---(0), others=---(0)
chmod 600 secret.key   # owner=rw-(6), group=---(0), others=---(0)
```

常用权限组合速查：

| 数字 | 权限 | 典型用途 |
|------|------|----------|
| `755` | `rwxr-xr-x` | 可执行文件、公共目录 |
| `644` | `rw-r--r--` | 普通文件（文档、配置） |
| `700` | `rwx------` | 私有目录 |
| `600` | `rw-------` | 私密文件（SSH key） |
| `777` | `rwxrwxrwx` | 所有人完全权限（⚠️ 避免使用） |

#### 符号模式（Symbolic Mode）

格式：`chmod [who][operator][permission]`

- **who**：`u`（owner）、`g`（group）、`o`（others）、`a`（all）
- **operator**：`+`（添加）、`-`（移除）、`=`（精确设置）
- **permission**：`r`、`w`、`x`

```bash
chmod u+x script.sh        # add execute for owner
chmod g-w file.txt          # remove write for group
chmod o-rwx private.txt     # remove all permissions for others
chmod a+r readme.txt        # add read for everyone
chmod u=rwx,g=rx,o=r file   # set exact permissions (same as 754)
chmod +x run.sh             # add execute for all (shorthand for a+x)
```

符号模式的优势是可以**增量修改**而不影响其他权限位：

```bash
# numeric mode overwrites all permissions
chmod 755 file.txt

# symbolic mode only changes what you specify
chmod g+w file.txt          # add group write without touching other bits
```

#### `-R` 递归修改

```bash
chmod -R 755 project/       # apply to directory and all contents recursively
chmod -R u+w project/       # add owner write recursively
```

### 4. `chown` / `chgrp` — 修改所有者和组

#### `chown` — Change Owner

```bash
chown alice file.txt              # change owner to alice
chown alice:developers file.txt   # change owner and group
chown :developers file.txt        # change group only (same as chgrp)
chown -R alice:staff project/     # recursive change
```

#### `chgrp` — Change Group

```bash
chgrp developers file.txt        # change group
chgrp -R staff project/          # recursive change
```

> ⚠️ `chown` 通常需要 `sudo` 权限（只有 root 可以将文件所有权交给其他用户）。普通用户只能使用 `chgrp` 将文件改到自己所属的组。

### 5. 特殊权限：SUID、SGID、Sticky Bit

除了基本的 rwx 权限，Linux 还有三种特殊权限位：

| 特殊权限 | 数字 | 对文件的作用 | 对目录的作用 |
|----------|------|-------------|-------------|
| SUID (Set User ID) | `4` | 执行时以文件所有者身份运行 | — |
| SGID (Set Group ID) | `2` | 执行时以文件所属组身份运行 | 新建文件继承目录的组 |
| Sticky Bit | `1` | — | 只有文件所有者可删除自己的文件 |

#### SUID

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
```

注意 owner 的 `x` 变成了 `s`。`passwd` 命令需要修改 `/etc/shadow`，但普通用户也能使用它，因为 SUID 使其以 root 身份执行。

```bash
chmod u+s program         # set SUID
chmod 4755 program        # set SUID with numeric mode (4 prefix)
```

#### SGID

```bash
chmod g+s shared_dir/     # set SGID on directory
chmod 2775 shared_dir/    # set SGID with numeric mode (2 prefix)
```

对目录设置 SGID 后，目录内新建的文件会自动继承该目录的组，非常适合团队共享目录。

#### Sticky Bit

```bash
ls -ld /tmp
# drwxrwxrwt 22 root root ... /tmp
```

注意 others 的 `x` 变成了 `t`。`/tmp` 目录所有用户都可以写入，但 Sticky Bit 确保用户只能删除自己创建的文件。

```bash
chmod +t shared_dir/      # set sticky bit
chmod 1777 shared_dir/    # set sticky bit with numeric mode (1 prefix)
```

### 6. `umask` — 默认权限掩码

`umask` 决定新建文件和目录的默认权限。系统默认文件权限为 `666`，目录为 `777`，实际权限 = 默认权限 - umask。

```bash
umask             # show current umask value
umask -S          # show in symbolic format
```

常见 umask 值：

| umask | 新文件权限 | 新目录权限 | 场景 |
|-------|-----------|-----------|------|
| `022` | `644` (rw-r--r--) | `755` (rwxr-xr-x) | 默认值 |
| `077` | `600` (rw-------) | `700` (rwx------) | 私密环境 |
| `002` | `664` (rw-rw-r--) | `775` (rwxrwxr-x) | 团队协作 |

```bash
umask 077             # set strict umask for current session
touch secret.txt      # created with 600 permissions
mkdir private_dir     # created with 700 permissions
```

> 永久修改 umask 需写入 `~/.bashrc` 或 `~/.profile`。

---

## 实战演练

模拟一个团队项目目录权限管理场景。

**步骤 1：创建练习环境**

```bash
mkdir -p ~/perm-practice && cd ~/perm-practice
```

**步骤 2：创建测试文件并观察默认权限**

```bash
touch myfile.txt
mkdir mydir
ls -l
ls -ld mydir
```

观察新文件和目录的默认权限（取决于你的 umask 设置）。

**步骤 3：使用数字模式修改权限**

```bash
chmod 755 myfile.txt
ls -l myfile.txt
# -rwxr-xr-x ... myfile.txt

chmod 600 myfile.txt
ls -l myfile.txt
# -rw------- ... myfile.txt
```

**步骤 4：使用符号模式修改权限**

```bash
chmod 644 myfile.txt       # reset to baseline
chmod u+x myfile.txt       # add execute for owner
ls -l myfile.txt
# -rwxr--r-- ... myfile.txt

chmod g+w,o-r myfile.txt   # add group write, remove others read
ls -l myfile.txt
# -rwxrw---- ... myfile.txt
```

**步骤 5：理解目录权限**

```bash
mkdir testdir
touch testdir/file1.txt testdir/file2.txt

chmod 700 testdir          # only owner can access
ls -l testdir/             # works (you are the owner)

chmod 000 testdir          # remove all permissions
ls testdir/                # Permission denied
cat testdir/file1.txt      # Permission denied

chmod 755 testdir          # restore permissions
```

**步骤 6：体验 Sticky Bit**

```bash
mkdir shared
chmod 1777 shared
ls -ld shared
# drwxrwxrwt ... shared
```

**步骤 7：查看 umask 并测试效果**

```bash
umask                      # show current value, e.g. 0022
touch default_file.txt
ls -l default_file.txt     # should be 644

umask 077                  # change to strict
touch private_file.txt
ls -l private_file.txt     # should be 600

umask 022                  # restore default
```

**步骤 8：查看系统中的 SUID 程序**

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x ... /usr/bin/passwd (note the 's')

ls -l /usr/bin/sudo
# -rwsr-xr-x ... /usr/bin/sudo
```

**步骤 9：清理**

```bash
cd ~
rm -r ~/perm-practice
```

---

## 小结

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `ls -l` | 查看权限详情 | `-d`（目录本身）、`-a`（含隐藏文件） |
| `chmod` | 修改权限 | 数字模式（`755`）、符号模式（`u+x`）、`-R`（递归） |
| `chown` | 修改所有者 | `user:group`（同时改组）、`-R`（递归） |
| `chgrp` | 修改所属组 | `-R`（递归） |
| `umask` | 查看/设置默认权限掩码 | `-S`（符号格式显示） |

特殊权限速查：

| 权限 | 符号 | 数字前缀 | 作用 |
|------|------|----------|------|
| SUID | `s`（owner x 位） | `4` | 以文件所有者身份执行 |
| SGID | `s`（group x 位） | `2` | 以文件所属组身份执行 / 新文件继承组 |
| Sticky Bit | `t`（others x 位） | `1` | 目录内只有文件主人可删除文件 |

---

## 练习

**1. 文件权限为 `-rw-r-----`，对应的数字模式是什么？哪些用户可以读取该文件？**

<details>
<summary>查看答案</summary>

数字模式为 `640`：
- owner: `rw-` = 4+2+0 = 6
- group: `r--` = 4+0+0 = 4
- others: `---` = 0+0+0 = 0

文件所有者可以读写，所属组的成员可以读取，其他用户没有任何权限。

</details>

**2. 如何让一个 Shell 脚本可执行？为什么 `chmod 777` 是不推荐的做法？**

<details>
<summary>查看答案</summary>

让脚本可执行：

```bash
chmod u+x script.sh    # recommended: only owner can execute
chmod 755 script.sh    # alternative: everyone can execute
```

`chmod 777` 不推荐的原因：
- 它赋予**所有用户**完全的读、写、执行权限
- 任何用户都可以修改甚至替换文件内容，带来严重的安全隐患
- 正确做法是按最小权限原则，只给真正需要的用户和操作赋予权限

</details>

**3. 一个目录权限为 `dr-xr-xr-x`，用户能进入该目录吗？能在该目录下创建文件吗？为什么？**

<details>
<summary>查看答案</summary>

- **能进入目录**：因为有 `x` 权限，可以 `cd` 进入
- **能列出内容**：因为有 `r` 权限，可以 `ls` 查看目录中的文件
- **不能创建文件**：因为没有 `w` 权限，无法在目录中创建、删除或重命名文件

目录的 `w` 权限控制的是对目录内容的修改（增删文件），不是修改文件本身的内容。

</details>

**4. 什么是 Sticky Bit？`/tmp` 目录为什么要设置 Sticky Bit？**

<details>
<summary>查看答案</summary>

Sticky Bit 是一种特殊权限，设置在目录上后，即使目录对所有人开放写权限，用户也只能删除自己创建的文件，不能删除其他用户的文件。

`/tmp` 是系统的公共临时目录，所有用户都需要在里面创建临时文件（权限 `777`）。如果没有 Sticky Bit，任何用户都可以删除其他用户的临时文件，导致数据丢失。设置 Sticky Bit（`1777`）后，每个用户只能管理自己的文件。

```bash
ls -ld /tmp
# drwxrwxrwt ... /tmp    (t = sticky bit)
```

</details>

**5. `umask 027` 时，新创建的文件和目录分别是什么权限？**

<details>
<summary>查看答案</summary>

- 新文件：`666 - 027 = 640`（`rw-r-----`）— 所有者可读写，同组可读，其他人无权限
- 新目录：`777 - 027 = 750`（`rwxr-x---`）— 所有者完全权限，同组可读和进入，其他人无权限

> 注意：umask 的计算不是简单的数学减法，而是位运算（按位取反再与）。但在大多数常见 umask 值下，减法可以得到正确结果。

</details>
