# 第 14 课：包管理器

> 上一课：[第 13 课：网络工具](./13-networking.md) | 下一课：[第 15 课：Environment Variables 与 Shell 配置](./15-environment-variables.md)

---

## 学习目标

1. 理解 Linux 包管理的基本概念（仓库、依赖、包格式）
2. 掌握 Debian/Ubuntu 系统的 `apt` 包管理器
3. 了解 RHEL/CentOS 系统的 `yum` / `dnf` 包管理器
4. 了解 Arch Linux 的 `pacman` 包管理器
5. 认识通用包管理器 `snap` 和 `flatpak`
6. 了解从源码编译安装软件的基本流程

---

## 知识讲解

### 1. 包管理基本概念

**包（Package）** 是将软件的二进制文件、配置文件、文档等打包在一起的归档文件，同时包含元数据（版本号、依赖关系等）。

**包管理器** 的核心功能：

| 功能 | 说明 |
|------|------|
| 安装 | 从仓库下载并安装软件包及其依赖 |
| 卸载 | 移除软件包，可选择是否保留配置文件 |
| 更新 | 将软件包升级到最新版本 |
| 搜索 | 在仓库中查找软件包 |
| 依赖解析 | 自动处理软件包之间的依赖关系 |

**常见包格式**：

| 格式 | 发行版 | 底层工具 | 高层工具 |
|------|--------|---------|---------|
| `.deb` | Debian, Ubuntu, Mint | `dpkg` | `apt` |
| `.rpm` | RHEL, CentOS, Fedora | `rpm` | `yum` / `dnf` |
| `.pkg.tar.zst` | Arch, Manjaro | - | `pacman` |

**软件仓库（Repository）**：包管理器从配置好的远程仓库下载软件包。仓库信息通常存储在：

- Debian/Ubuntu：`/etc/apt/sources.list` 和 `/etc/apt/sources.list.d/`
- RHEL/CentOS：`/etc/yum.repos.d/`

### 2. Debian/Ubuntu：apt

`apt` 是 Debian 系最常用的包管理工具，整合了 `apt-get` 和 `apt-cache` 的常用功能。

#### 更新软件源

```bash
# refresh package index from repositories
sudo apt update
```

> **重要**：在安装任何包之前，都应该先运行 `apt update` 以获取最新的包信息。

#### 安装软件

```bash
# install a package
sudo apt install nginx

# install multiple packages
sudo apt install git curl vim

# install without confirmation prompt
sudo apt install -y nodejs

# install a specific version
sudo apt install nginx=1.18.0-0ubuntu1
```

#### 卸载软件

```bash
# remove package (keep config files)
sudo apt remove nginx

# remove package and its config files
sudo apt purge nginx

# remove unused dependencies
sudo apt autoremove
```

#### 更新软件

```bash
# upgrade all installed packages
sudo apt upgrade

# upgrade with dependency changes (may install/remove packages)
sudo apt full-upgrade

# upgrade a specific package
sudo apt install --only-upgrade nginx
```

#### 搜索与查询

```bash
# search for packages
apt search nginx

# show package details
apt show nginx

# list installed packages
apt list --installed

# list upgradable packages
apt list --upgradable

# check if a package is installed
dpkg -l | grep nginx

# show which package provides a file
dpkg -S /usr/bin/curl
```

#### dpkg —— 底层工具

```bash
# install a local .deb file
sudo dpkg -i package.deb

# fix broken dependencies after dpkg install
sudo apt install -f

# list files installed by a package
dpkg -L nginx

# show package status
dpkg -s nginx
```

### 3. RHEL/CentOS：yum / dnf

`dnf` 是 `yum` 的下一代替代品（Fedora 22+，RHEL 8+），语法几乎相同。

```bash
# refresh repository cache
sudo yum makecache
sudo dnf makecache

# install a package
sudo yum install httpd
sudo dnf install httpd

# remove a package
sudo yum remove httpd
sudo dnf remove httpd

# update all packages
sudo yum update
sudo dnf upgrade

# search for a package
yum search nginx
dnf search nginx

# show package info
yum info httpd
dnf info httpd

# list installed packages
yum list installed
dnf list installed

# install from local RPM
sudo yum localinstall package.rpm
sudo dnf install ./package.rpm

# show which package provides a file
yum provides /usr/bin/curl
dnf provides /usr/bin/curl

# list available package groups
yum grouplist
dnf grouplist

# install a package group
sudo yum groupinstall "Development Tools"
sudo dnf groupinstall "Development Tools"

# clean cache
sudo yum clean all
sudo dnf clean all
```

### 4. Arch Linux：pacman

```bash
# sync and update all packages
sudo pacman -Syu

# install a package
sudo pacman -S nginx

# remove a package
sudo pacman -R nginx

# remove package with unused dependencies
sudo pacman -Rns nginx

# search for a package in repositories
pacman -Ss nginx

# search installed packages
pacman -Qs nginx

# show package info
pacman -Si nginx

# list files installed by a package
pacman -Ql nginx

# clean package cache
sudo pacman -Sc
```

> **提示**：Arch 用户还常使用 AUR（Arch User Repository）Helper 如 `yay` 或 `paru` 安装社区维护的软件包。

### 5. 通用包管理器：snap / flatpak

这些工具提供跨发行版的打包方案，应用运行在沙盒中。

#### snap（Canonical 主导）

```bash
# install a snap
sudo snap install vlc

# list installed snaps
snap list

# update all snaps
sudo snap refresh

# remove a snap
sudo snap remove vlc

# find snaps
snap find "media player"
```

#### flatpak

```bash
# install from Flathub
flatpak install flathub org.videolan.VLC

# run a flatpak app
flatpak run org.videolan.VLC

# list installed apps
flatpak list

# update all
flatpak update

# remove an app
flatpak uninstall org.videolan.VLC
```

#### 传统包管理 vs snap/flatpak

| 特性 | apt/yum/pacman | snap/flatpak |
|------|---------------|-------------|
| 系统集成 | 深度集成 | 沙盒隔离 |
| 包大小 | 较小（共享库） | 较大（自带依赖） |
| 更新速度 | 依赖发行版发布周期 | 独立更新 |
| 安全隔离 | 无沙盒 | 有沙盒 |
| 跨发行版 | 不通用 | 通用 |

### 6. 从源码编译安装

当软件仓库中没有你需要的软件或版本时，可以从源码编译安装。

#### 典型流程

```bash
# Step 1: install build dependencies
sudo apt install build-essential    # Debian/Ubuntu
sudo yum groupinstall "Development Tools"  # RHEL/CentOS

# Step 2: download and extract source
wget https://example.com/software-1.0.tar.gz
tar xzf software-1.0.tar.gz
cd software-1.0

# Step 3: configure (check dependencies, set install path)
./configure --prefix=/usr/local

# Step 4: compile
make -j$(nproc)

# Step 5: install
sudo make install
```

#### 各步骤说明

| 步骤 | 命令 | 作用 |
|------|------|------|
| 配置 | `./configure` | 检查系统环境、依赖，生成 Makefile |
| 编译 | `make` | 根据 Makefile 编译源代码 |
| 安装 | `sudo make install` | 将编译好的文件复制到系统目录 |

`--prefix` 参数控制安装路径，默认为 `/usr/local`。使用自定义路径可以方便管理和卸载：

```bash
# install to custom directory
./configure --prefix=/opt/myapp
make -j$(nproc)
sudo make install

# uninstall (if supported by Makefile)
sudo make uninstall

# or simply remove the directory
sudo rm -rf /opt/myapp
```

> **建议**：优先使用包管理器安装软件。源码编译仅在必要时使用，因为手动编译的软件不受包管理器管理，更新和卸载不够方便。如果需要打包，考虑使用 `checkinstall` 将编译结果打成 `.deb` 或 `.rpm`。

### 7. 跨发行版命令对比

| 操作 | apt (Debian/Ubuntu) | yum/dnf (RHEL/CentOS) | pacman (Arch) |
|------|--------------------|-----------------------|---------------|
| 更新索引 | `apt update` | `yum makecache` | `pacman -Sy` |
| 升级所有包 | `apt upgrade` | `yum update` | `pacman -Syu` |
| 安装 | `apt install PKG` | `yum install PKG` | `pacman -S PKG` |
| 卸载 | `apt remove PKG` | `yum remove PKG` | `pacman -R PKG` |
| 搜索 | `apt search KEY` | `yum search KEY` | `pacman -Ss KEY` |
| 查看信息 | `apt show PKG` | `yum info PKG` | `pacman -Si PKG` |
| 列出已安装 | `apt list --installed` | `yum list installed` | `pacman -Q` |
| 清理缓存 | `apt clean` | `yum clean all` | `pacman -Sc` |
| 查文件属于哪个包 | `dpkg -S FILE` | `yum provides FILE` | `pacman -Qo FILE` |

---

## 实战演练

> 以下实验以 Debian/Ubuntu 系统为例。RHEL/CentOS 和 Arch 用户请替换为对应命令。

### 步骤 1：查看系统包信息

```bash
# check your package manager
which apt yum dnf pacman 2>/dev/null

# check system release info
cat /etc/os-release
```

### 步骤 2：更新软件源

```bash
sudo apt update
```

### 步骤 3：搜索和查询软件包

```bash
# search for a text editor
apt search "text editor" | head -20

# show details of a package
apt show vim

# check if tree is installed
dpkg -l | grep tree
```

### 步骤 4：安装和使用软件

```bash
# install tree (directory listing tool)
sudo apt install -y tree

# verify installation
which tree
tree --version

# use it
tree -L 2 /etc/apt/
```

### 步骤 5：管理软件包

```bash
# list upgradable packages
apt list --upgradable

# show installed package count
dpkg -l | grep "^ii" | wc -l

# find what package provides a command
dpkg -S $(which curl)
```

### 步骤 6：查看仓库配置

```bash
# view APT sources
cat /etc/apt/sources.list

# view additional repositories
ls /etc/apt/sources.list.d/
```

### 步骤 7：卸载软件

```bash
# remove tree
sudo apt remove -y tree

# verify removal
which tree

# remove leftover config and unused dependencies
sudo apt purge -y tree
sudo apt autoremove -y
```

---

## 小结

| 命令 | 作用 |
|------|------|
| `sudo apt update` | 刷新软件源索引 |
| `sudo apt install PKG` | 安装软件包 |
| `sudo apt remove PKG` | 卸载软件包（保留配置） |
| `sudo apt purge PKG` | 卸载软件包（删除配置） |
| `sudo apt upgrade` | 升级所有已安装包 |
| `apt search KEY` | 搜索软件包 |
| `apt show PKG` | 查看软件包详情 |
| `apt list --installed` | 列出已安装的包 |
| `sudo apt autoremove` | 清理无用依赖 |
| `dpkg -i file.deb` | 安装本地 .deb 文件 |
| `dpkg -S FILE` | 查找文件属于哪个包 |
| `./configure && make && sudo make install` | 源码编译三步曲 |

---

## 练习

**1. 你想安装 `htop`，但不确定包名是否正确，应该怎么做？**

<details>
<summary>查看答案</summary>

```bash
apt search htop
```

搜索结果会显示所有名称或描述中包含 `htop` 的软件包。确认包名后：

```bash
sudo apt install htop
```

</details>

**2. 你下载了一个 `.deb` 文件，安装时报依赖错误，应该如何解决？**

<details>
<summary>查看答案</summary>

```bash
# install the .deb file
sudo dpkg -i package.deb

# fix broken dependencies
sudo apt install -f
```

`dpkg` 不会自动处理依赖关系，`apt install -f` 会自动下载并安装缺失的依赖包。

</details>

**3. 如何查看系统上安装了哪些包，并找出哪个包提供了 `/usr/bin/git` 这个文件？**

<details>
<summary>查看答案</summary>

```bash
# list all installed packages
apt list --installed

# find which package provides the file
dpkg -S /usr/bin/git
```

输出类似：`git: /usr/bin/git`，表示该文件由 `git` 包提供。

</details>

**4. 在 CentOS 系统上，如何安装 "Development Tools" 包组？安装包组与单个包有什么区别？**

<details>
<summary>查看答案</summary>

```bash
sudo yum groupinstall "Development Tools"
```

包组（group）是一组相关软件包的集合。"Development Tools" 包含 `gcc`、`make`、`autoconf` 等编译所需工具。安装包组相当于一次性安装该组内所有的软件包，而安装单个包只安装指定的一个。

</details>

**5. 从源码编译软件时，`./configure --prefix=/opt/myapp` 中的 `--prefix` 有什么作用？这样做有什么好处？**

<details>
<summary>查看答案</summary>

`--prefix` 指定软件的安装目录。默认值是 `/usr/local`，使用自定义路径如 `/opt/myapp` 有以下好处：

1. **隔离管理**：所有文件集中在一个目录下，不会散落到系统各处
2. **方便卸载**：直接删除整个目录即可 `sudo rm -rf /opt/myapp`
3. **多版本共存**：可以同时安装不同版本到不同路径

需要注意的是，安装到自定义路径后需要手动将 `bin` 目录添加到 `PATH`：

```bash
export PATH="/opt/myapp/bin:$PATH"
```

</details>
