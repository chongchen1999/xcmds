# xcmds - Linux 命令行从入门到精通

一个系统化的 Linux 命令行教程，帮助你从零基础成长为熟练的 Linux 用户。

每课包含：**知识讲解** → **实战演练** → **练习题**（答案可折叠查看）

## 课程目录

### 入门篇 Beginner

| # | 课程 | 核心命令 |
|---|------|----------|
| 01 | [什么是 Terminal 与 Shell](beginner/01-terminal-and-shell.md) | bash, zsh, terminal |
| 02 | [文件系统导航](beginner/02-navigating-filesystem.md) | pwd, cd, ls |
| 03 | [文件操作](beginner/03-file-operations.md) | cp, mv, rm, mkdir, touch |
| 04 | [查看文件内容](beginner/04-viewing-files.md) | cat, less, head, tail |
| 05 | [文件权限与所有权](beginner/05-permissions.md) | chmod, chown, ls -l |
| 06 | [I/O Redirection 与 Pipe](beginner/06-redirection-and-pipes.md) | >, >>, \|, < |
| 07 | [获取帮助](beginner/07-getting-help.md) | man, --help, tldr |
| 08 | [入门综合测验](beginner/08-beginner-quiz.md) | 综合复习 |

### 进阶篇 Intermediate

| # | 课程 | 核心命令 |
|---|------|----------|
| 09 | [文本搜索 grep 与 Regex](intermediate/09-grep-and-regex.md) | grep, egrep, regex |
| 10 | [流编辑器 sed](intermediate/10-sed.md) | sed |
| 11 | [数据处理工具 awk](intermediate/11-awk.md) | awk |
| 12 | [Process 管理](intermediate/12-process-management.md) | ps, top, kill, jobs, bg/fg |
| 13 | [网络工具](intermediate/13-networking.md) | curl, wget, ssh, scp, ping |
| 14 | [包管理器](intermediate/14-package-management.md) | apt, yum, pacman |
| 15 | [Environment Variables 与 Shell 配置](intermediate/15-environment-variables.md) | export, .bashrc, PATH |
| 16 | [进阶综合测验](intermediate/16-intermediate-quiz.md) | 综合复习 |

### 高级篇 Advanced

| # | 课程 | 核心命令 |
|---|------|----------|
| 17 | [Shell Script 基础](advanced/17-scripting-fundamentals.md) | variables, if, for, while |
| 18 | [函数与脚本组织](advanced/18-functions.md) | function, source, return |
| 19 | [高级脚本技巧](advanced/19-advanced-scripting.md) | arrays, trap, getopts |
| 20 | [Cron Job 与自动化](advanced/20-cron-and-automation.md) | crontab, at, systemd timer |
| 21 | [系统管理](advanced/21-system-admin.md) | useradd, systemctl, journalctl |
| 22 | [磁盘与存储管理](advanced/22-disk-and-storage.md) | df, du, mount, fdisk, lvm |
| 23 | [性能调优与监控](advanced/23-performance.md) | vmstat, iostat, sar, strace |
| 24 | [高级综合实战项目](advanced/24-capstone-project.md) | 综合实战 |

### 综合练习

| 难度 | 练习 |
|------|------|
| 入门 | [入门练习题集](exercises/beginner-exercises.md) |
| 进阶 | [进阶练习题集](exercises/intermediate-exercises.md) |
| 高级 | [高级练习题集](exercises/advanced-exercises.md) |

## 如何使用

1. **按顺序阅读**：从第 01 课开始，逐课学习
2. **动手练习**：每课的「实战演练」部分，请在自己的终端中跟着操作
3. **完成测验**：每课末尾有练习题，先自己思考，再点击查看答案
4. **综合训练**：完成一个阶段后，到 `exercises/` 目录做综合练习

## 环境准备

- **macOS**：自带 Terminal，打开即用
- **Windows**：推荐使用 [WSL](https://docs.microsoft.com/en-us/windows/wsl/install)（Windows Subsystem for Linux）
- **Linux**：任意发行版的 Terminal 即可

## License

MIT
