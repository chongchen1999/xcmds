# 第 03 课：文件操作

> 上一课：[第 02 课：文件系统导航](./02-navigating-filesystem.md) | 下一课：[第 04 课：查看文件内容](./04-viewing-files.md)

## 学习目标

- 掌握文件和目录的创建：`touch`、`mkdir`
- 掌握文件和目录的复制与移动：`cp`、`mv`
- 掌握文件和目录的删除：`rm`，理解其危险性
- 理解符号链接（symbolic link）和硬链接（hard link）的概念与使用

---

## 知识讲解

### 1. `touch` — 创建文件 / 更新时间戳

`touch` 的主要作用是创建空文件。如果文件已存在，则更新其修改时间戳。

```bash
touch newfile.txt            # create empty file
touch file1.txt file2.txt    # create multiple files
touch existing.txt           # update timestamp only
```

### 2. `mkdir` — 创建目录

```bash
mkdir mydir                  # create single directory
mkdir dir1 dir2 dir3         # create multiple directories
mkdir -p a/b/c/d             # create nested directories recursively
```

`-p` 选项非常重要：它会自动创建路径中所有不存在的父目录，且目标目录已存在时不会报错。

```bash
# without -p: fails if 'a' or 'a/b' doesn't exist
mkdir a/b/c

# with -p: creates all missing parent directories
mkdir -p a/b/c
```

### 3. `cp` — 复制文件和目录

```bash
cp source.txt dest.txt              # copy file
cp source.txt /tmp/                 # copy to directory
cp file1.txt file2.txt destdir/     # copy multiple files to directory
cp -r sourcedir/ destdir/           # copy directory recursively
cp -i source.txt dest.txt          # prompt before overwriting
cp -v source.txt dest.txt          # verbose output
```

关键选项：

| 选项 | 说明 |
|------|------|
| `-r` | 递归复制，用于目录（必须） |
| `-i` | 覆盖前提示确认 |
| `-v` | 显示复制过程 |
| `-p` | 保留文件属性（权限、时间戳等） |
| `-a` | 归档模式，等同于 `-rpL`，完整保留所有属性 |

### 4. `mv` — 移动和重命名

`mv` 同时承担"移动"和"重命名"两个功能：

```bash
# rename
mv oldname.txt newname.txt

# move to another directory
mv file.txt /tmp/

# move and rename simultaneously
mv file.txt /tmp/newname.txt

# move multiple files to a directory
mv file1.txt file2.txt destdir/

# move a directory
mv srcdir/ /tmp/destdir/
```

> `mv` 不需要 `-r` 选项，它天然支持移动目录。

```bash
mv -i file.txt /tmp/        # prompt before overwriting
mv -v file.txt /tmp/        # verbose output
```

### 5. `rm` — 删除文件和目录

```bash
rm file.txt                  # delete a file
rm file1.txt file2.txt       # delete multiple files
rm -i file.txt               # prompt before deleting
rm -r mydir/                 # delete directory and its contents recursively
rm -f file.txt               # force delete, no prompt, no error if missing
```

常用选项组合：

| 命令 | 说明 |
|------|------|
| `rm file` | 删除单个文件 |
| `rm -r dir/` | 递归删除目录及其内容 |
| `rm -i file` | 删除前确认 |
| `rm -f file` | 强制删除，忽略不存在的文件 |
| `rm -ri dir/` | 递归删除但逐个确认 |

> ⚠️ **严重警告：`rm -rf` 的危险性**
>
> ```bash
> # NEVER run these commands:
> rm -rf /           # destroys entire filesystem
> rm -rf /*          # same effect
> rm -rf ~           # destroys your entire home directory
> ```
>
> `rm` 删除的文件**不会进入回收站**，几乎无法恢复。使用 `rm -rf` 前请务必：
> 1. 用 `ls` 先确认目标路径
> 2. 用 `pwd` 确认当前所在位置
> 3. 避免在变量未定义时使用 `rm -rf $VAR/`（若 `$VAR` 为空，等同于 `rm -rf /`）

如果只想删除空目录，可以使用更安全的 `rmdir`：

```bash
rmdir emptydir/              # only removes empty directories
```

### 6. `ln` — 创建链接

Linux 支持两种链接：

**硬链接（Hard Link）** — 指向同一个 inode（文件的底层数据），删除原文件后硬链接仍然有效：

```bash
ln original.txt hardlink.txt
```

**符号链接（Symbolic Link / Soft Link）** — 类似快捷方式，指向文件路径。原文件删除后符号链接失效：

```bash
ln -s original.txt symlink.txt
ln -s /usr/local/bin/python3 ~/bin/python    # common use case
```

两者对比：

| 特性 | 硬链接 | 符号链接 |
|------|--------|----------|
| 创建方式 | `ln file link` | `ln -s file link` |
| 跨文件系统 | ❌ 不支持 | ✅ 支持 |
| 链接目录 | ❌ 不支持 | ✅ 支持 |
| 原文件删除后 | 仍然可用 | 失效（dangling link） |
| 本质 | 指向相同 inode | 指向文件路径 |

查看链接信息：

```bash
ls -l symlink.txt
# lrwxrwxrwx 1 tourist staff 12 Apr 14 10:00 symlink.txt -> original.txt
```

符号链接的权限显示为 `l` 开头，并用 `->` 标明指向。

---

## 实战演练

模拟一个项目目录的创建和管理过程：

**步骤 1：创建项目目录结构**

```bash
mkdir -p ~/practice-project/{src,docs,tests,config}
cd ~/practice-project
tree
```

输出：

```
.
├── config
├── docs
├── src
└── tests
```

**步骤 2：创建文件**

```bash
touch src/main.py src/utils.py
touch docs/README.md
touch tests/test_main.py
touch config/settings.json
tree
```

**步骤 3：复制文件作为备份**

```bash
cp config/settings.json config/settings.json.bak
ls config/
```

**步骤 4：复制整个目录**

```bash
cp -r src/ src_backup/
tree
```

**步骤 5：重命名文件**

```bash
mv docs/README.md docs/guide.md
ls docs/
```

**步骤 6：移动文件到另一个目录**

```bash
touch TODO.txt
mv TODO.txt docs/
ls docs/
```

**步骤 7：创建符号链接**

```bash
ln -s src/main.py main_link.py
ls -l main_link.py
```

你会看到 `main_link.py -> src/main.py`。

**步骤 8：验证符号链接**

```bash
echo "print('hello')" > src/main.py
cat main_link.py
```

通过符号链接可以直接读取原文件内容。

**步骤 9：安全删除文件**

```bash
rm -i config/settings.json.bak
```

系统会提示确认，输入 `y` 确认删除。

**步骤 10：清理整个练习项目**

```bash
cd ~
rm -r ~/practice-project
```

---

## 小结

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `touch` | 创建空文件 / 更新时间戳 | — |
| `mkdir` | 创建目录 | `-p`（递归创建） |
| `cp` | 复制文件或目录 | `-r`（递归）、`-i`（确认）、`-v`（详细） |
| `mv` | 移动或重命名 | `-i`（确认）、`-v`（详细） |
| `rm` | 删除文件或目录 | `-r`（递归）、`-f`（强制）、`-i`（确认） |
| `rmdir` | 删除空目录 | — |
| `ln` | 创建硬链接 | — |
| `ln -s` | 创建符号链接 | `-s`（symbolic） |

---

## 练习

**1. 如何一条命令创建 `a/b/c/d` 这样的多层嵌套目录？**

<details>
<summary>查看答案</summary>

```bash
mkdir -p a/b/c/d
```

`-p` 选项会自动创建路径中所有不存在的父目录。

</details>

**2. `cp file.txt dir/` 和 `mv file.txt dir/` 有什么区别？**

<details>
<summary>查看答案</summary>

- `cp file.txt dir/` — **复制**文件到 `dir/`，原文件仍然保留
- `mv file.txt dir/` — **移动**文件到 `dir/`，原位置的文件不再存在

`cp` 是创建副本，`mv` 是移动位置。

</details>

**3. 硬链接和符号链接有什么区别？分别在什么场景下使用？**

<details>
<summary>查看答案</summary>

**硬链接**：与原文件共享同一个 inode，删除原文件后硬链接仍有效。不能跨文件系统，不能链接目录。适用于需要确保数据安全的场景（如备份）。

**符号链接**：类似快捷方式，记录的是目标路径。删除原文件后符号链接会失效。可以跨文件系统，可以链接目录。适用于创建便捷访问路径（如将常用命令链接到 `~/bin/`）。

</details>

**4. 为什么 `rm -rf $DIR/` 在 `$DIR` 变量为空时非常危险？如何避免？**

<details>
<summary>查看答案</summary>

当 `$DIR` 为空时，`rm -rf $DIR/` 会展开为 `rm -rf /`，这将**删除整个文件系统**。

避免方法：
1. 使用前先检查变量：`[ -z "$DIR" ] && echo "DIR is empty" && exit 1`
2. 使用带引号的变量：`rm -rf "${DIR:?Variable not set}/"` — 变量未设置时会报错而不是继续执行
3. 先用 `echo` 确认命令：`echo rm -rf "$DIR/"` 确认无误后再执行

</details>

**5. 执行以下命令后，`tree` 的输出是什么？**

```bash
mkdir -p project/{frontend,backend}/{src,test}
touch project/frontend/src/app.js project/backend/src/server.py
```

<details>
<summary>查看答案</summary>

```
project
├── backend
│   ├── src
│   │   └── server.py
│   └── test
└── frontend
    ├── src
    │   └── app.js
    └── test
```

`{}` 是 brace expansion，`{frontend,backend}` 会展开为两个目录，`{src,test}` 同理，所以一条 `mkdir -p` 命令就创建了完整的目录结构。

</details>
