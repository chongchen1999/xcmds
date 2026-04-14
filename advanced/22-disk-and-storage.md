# 第 22 课：磁盘与存储管理

> 上一课：[系统管理](21-system-admin.md) | 下一课：[性能调优与监控](23-performance.md)

## 学习目标

- 掌握 `df` 和 `du` 查看磁盘使用情况
- 理解 Linux 挂载机制和 `/etc/fstab` 配置
- 了解 `fdisk`/`parted` 分区和 `mkfs` 格式化
- 掌握 LVM 的核心概念与基本操作
- 学会管理 swap 空间

---

## 知识讲解

### 1. df — 查看磁盘空间

`df`（disk free）显示文件系统的整体空间使用情况。

```bash
# Human-readable output
df -h

# Example output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   32G   16G  67% /
# /dev/sda2       200G  180G   10G  95% /home
# tmpfs            16G  1.2M   16G   1% /tmp

# Show inode usage (files count, not space)
df -i

# Show only specific filesystem type
df -h -t ext4

# Show filesystem for a specific path
df -h /var/log
```

**inode 耗尽** 是一个容易忽略的问题——磁盘空间充足但无法创建新文件：

```bash
# Check if inode usage is high
df -i | awk '$5+0 > 80 {print}'
```

### 2. du — 查看目录大小

`du`（disk usage）统计文件和目录占用的空间。

```bash
# Summary of a directory
du -sh /var/log
# 2.3G    /var/log

# Top-level subdirectory sizes
du -h --max-depth=1 /var
# 2.3G    /var/log
# 560M    /var/cache
# 128M    /var/lib
# 3.0G    /var

# Sort by size (largest first)
du -h --max-depth=1 /var | sort -rh | head -10

# Exclude patterns
du -sh --exclude='*.log' /var

# Find largest directories system-wide
sudo du -h --max-depth=2 / 2>/dev/null | sort -rh | head -20
```

#### 快速定位大文件

```bash
# Find files larger than 100MB
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh

# Using ncdu (interactive, if available)
sudo ncdu /
```

### 3. mount / umount

Linux 通过挂载（mount）将文件系统附加到目录树上。

```bash
# View currently mounted filesystems
mount
mount | column -t

# Filter by type
mount -t ext4

# Mount a device
sudo mount /dev/sdb1 /mnt/data

# Mount with options
sudo mount -o ro,noexec /dev/sdb1 /mnt/data          # Read-only, no execution
sudo mount -o rw,nosuid,nodev /dev/sdb1 /mnt/data

# Mount ISO image
sudo mount -o loop disk.iso /mnt/iso

# Mount NFS share
sudo mount -t nfs 192.168.1.100:/shared /mnt/nfs

# Unmount
sudo umount /mnt/data

# Force unmount (use with caution)
sudo umount -f /mnt/data

# Lazy unmount: detach now, clean up when no longer busy
sudo umount -l /mnt/data
```

常见挂载选项：

| 选项 | 说明 |
|------|------|
| `ro` / `rw` | 只读 / 读写 |
| `noexec` | 禁止执行二进制文件 |
| `nosuid` | 忽略 SUID/SGID 位 |
| `nodev` | 禁止设备文件 |
| `noatime` | 不更新访问时间（提升性能） |
| `defaults` | `rw,suid,dev,exec,auto,nouser,async` |

### 4. /etc/fstab

`/etc/fstab` 定义系统启动时自动挂载的文件系统。

```
# <device>            <mount_point>  <type>  <options>        <dump> <pass>
UUID=abc123...         /              ext4    errors=remount-ro 0      1
UUID=def456...         /home          ext4    defaults          0      2
UUID=789xyz...         none           swap    sw                0      0
/dev/sdb1              /mnt/data      xfs     defaults,noatime  0      2
192.168.1.100:/shared  /mnt/nfs       nfs     defaults          0      0
tmpfs                  /tmp           tmpfs   defaults,size=2G  0      0
```

| 字段 | 说明 |
|------|------|
| device | 设备路径或 UUID（推荐用 UUID，因设备名可能变化） |
| mount_point | 挂载点目录 |
| type | 文件系统类型（ext4、xfs、nfs、swap） |
| options | 挂载选项 |
| dump | 备份标志（通常为 0） |
| pass | `fsck` 检查顺序（`/` 为 1，其他为 2，0=不检查） |

```bash
# Find UUID of a device
sudo blkid
sudo blkid /dev/sda1

# Test fstab without rebooting (mount all entries)
sudo mount -a

# Remount a filesystem with new options
sudo mount -o remount,rw /
```

> ⚠️ 编辑 `/etc/fstab` 要格外小心，错误的配置可能导致系统无法启动。修改后务必用 `mount -a` 测试。

### 5. fdisk / parted — 分区管理

#### fdisk（适用于 MBR，也支持 GPT）

```bash
# List all disks and partitions
sudo fdisk -l

# Partition a disk (interactive)
sudo fdisk /dev/sdb
# Common commands inside fdisk:
#   n - new partition
#   d - delete partition
#   p - print partition table
#   t - change partition type
#   w - write changes and exit
#   q - quit without saving
```

#### parted（推荐用于 GPT 和大磁盘）

```bash
# List partitions
sudo parted /dev/sdb print

# Create GPT partition table
sudo parted /dev/sdb mklabel gpt

# Create a partition using all space
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# Create specific size partition
sudo parted /dev/sdb mkpart primary ext4 1MiB 50GiB
```

### 6. mkfs — 格式化

```bash
# Format as ext4
sudo mkfs.ext4 /dev/sdb1

# Format as ext4 with label
sudo mkfs.ext4 -L mydata /dev/sdb1

# Format as XFS
sudo mkfs.xfs /dev/sdb1

# Force format (overwrite existing filesystem)
sudo mkfs.xfs -f /dev/sdb1
```

ext4 vs XFS：

| 特性 | ext4 | XFS |
|------|------|-----|
| 最大文件系统 | 1 EB | 8 EB |
| 最大文件 | 16 TB | 8 EB |
| 缩小分区 | 支持 | 不支持 |
| 适用场景 | 通用 | 大文件、高吞吐 |
| 默认发行版 | Debian/Ubuntu | RHEL/CentOS 7+ |

### 7. LVM — 逻辑卷管理

LVM 在物理磁盘和文件系统之间增加了一个抽象层，可以灵活地调整分区大小。

```
物理磁盘 → PV (Physical Volume) → VG (Volume Group) → LV (Logical Volume) → 文件系统
```

#### 创建 LVM

```bash
# 1. Create Physical Volumes
sudo pvcreate /dev/sdb /dev/sdc

# 2. Create Volume Group
sudo vgcreate datavg /dev/sdb /dev/sdc

# 3. Create Logical Volumes
sudo lvcreate -L 100G -n datalv datavg      # Fixed size
sudo lvcreate -l 100%FREE -n datalv datavg   # Use all remaining space

# 4. Format and mount
sudo mkfs.ext4 /dev/datavg/datalv
sudo mkdir -p /mnt/data
sudo mount /dev/datavg/datalv /mnt/data
```

#### 查看 LVM 状态

```bash
# Physical Volumes
sudo pvs
sudo pvdisplay

# Volume Groups
sudo vgs
sudo vgdisplay

# Logical Volumes
sudo lvs
sudo lvdisplay
```

#### 扩展 LV

LVM 最大的优势——在线扩容：

```bash
# Add a new disk to VG
sudo pvcreate /dev/sdd
sudo vgextend datavg /dev/sdd

# Extend LV by 50G
sudo lvextend -L +50G /dev/datavg/datalv

# Extend LV to use all free space
sudo lvextend -l +100%FREE /dev/datavg/datalv

# Resize filesystem to match (ext4)
sudo resize2fs /dev/datavg/datalv

# For XFS
sudo xfs_growfs /mnt/data

# Extend LV and resize filesystem in one step
sudo lvextend -L +50G --resizefs /dev/datavg/datalv
```

### 8. swap 管理

swap 是磁盘上的虚拟内存空间，当物理内存不足时使用。

#### 查看 swap

```bash
# View swap usage
free -h
swapon --show
cat /proc/swaps
```

#### 使用文件创建 swap

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Verify
swapon --show

# Make persistent (add to /etc/fstab)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### 使用分区创建 swap

```bash
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
```

#### 调整 swappiness

`swappiness`（0-100）控制内核使用 swap 的倾向，值越低越倾向于使用物理内存：

```bash
# View current value
cat /proc/sys/vm/swappiness

# Temporary change
sudo sysctl vm.swappiness=10

# Persistent change
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

| 值 | 策略 |
|------|------|
| 0 | 尽量不使用 swap（除非必要） |
| 10-30 | 服务器推荐值 |
| 60 | 默认值 |
| 100 | 积极使用 swap |

#### 关闭 swap

```bash
# Disable specific swap
sudo swapoff /swapfile

# Disable all swap
sudo swapoff -a
```

---

## 实战演练

### 练习 1：磁盘空间分析

```bash
# Check overall disk usage
df -h

# Find the largest directories under /var
sudo du -h --max-depth=1 /var | sort -rh | head -10

# Check inode usage
df -i

# Find files larger than 50MB in /var
sudo find /var -type f -size +50M -exec ls -lh {} \; 2>/dev/null
```

### 练习 2：创建并挂载文件系统

```bash
# Create a 100MB file to simulate a disk
dd if=/dev/zero of=/tmp/testdisk.img bs=1M count=100

# Create a loop device
sudo losetup /dev/loop0 /tmp/testdisk.img

# Format as ext4
sudo mkfs.ext4 /dev/loop0

# Mount
sudo mkdir -p /mnt/testdisk
sudo mount /dev/loop0 /mnt/testdisk

# Verify
df -h /mnt/testdisk
mount | grep testdisk

# Create some test files
sudo touch /mnt/testdisk/hello.txt
ls -la /mnt/testdisk/

# Clean up
sudo umount /mnt/testdisk
sudo losetup -d /dev/loop0
rm -f /tmp/testdisk.img
sudo rmdir /mnt/testdisk
```

### 练习 3：swap 管理

```bash
# Check current swap
free -h
swapon --show

# Create temporary swap file
sudo fallocate -l 256M /tmp/testswap
sudo chmod 600 /tmp/testswap
sudo mkswap /tmp/testswap
sudo swapon /tmp/testswap

# Verify
swapon --show

# Check swappiness
cat /proc/sys/vm/swappiness

# Clean up
sudo swapoff /tmp/testswap
sudo rm /tmp/testswap
```

### 练习 4：编写磁盘空间报告脚本

```bash
#!/bin/bash
# disk_report.sh - Generate a disk usage report

echo "=============================="
echo "  Disk Usage Report"
echo "  $(date '+%Y-%m-%d %H:%M:%S')"
echo "=============================="

echo ""
echo "--- Filesystem Usage ---"
df -h --output=source,size,used,avail,pcent,target -x tmpfs -x devtmpfs

echo ""
echo "--- Inode Usage ---"
df -i --output=source,itotal,iused,iavail,ipcent,target -x tmpfs -x devtmpfs

echo ""
echo "--- Top 10 Largest Directories (/) ---"
sudo du -h --max-depth=1 / 2>/dev/null | sort -rh | head -10

echo ""
echo "--- Swap ---"
free -h | grep -i swap

echo ""
echo "--- Mounted Filesystems ---"
mount | grep "^/dev" | column -t
```

---

## 小结

| 概念 | 命令 | 说明 |
|------|------|------|
| 磁盘空间 | `df -h` | 查看文件系统空间使用 |
| inode 使用 | `df -i` | 查看 inode 使用率 |
| 目录大小 | `du -sh dir` | 查看目录总大小 |
| 分级大小 | `du -h --max-depth=1` | 按子目录查看大小 |
| 挂载 | `mount dev dir` | 挂载设备到目录 |
| 卸载 | `umount dir` | 卸载文件系统 |
| 分区 | `fdisk` / `parted` | 管理磁盘分区 |
| 格式化 | `mkfs.ext4` / `mkfs.xfs` | 创建文件系统 |
| 查看 UUID | `blkid` | 查看设备 UUID |
| 创建 PV | `pvcreate` | 创建 LVM 物理卷 |
| 创建 VG | `vgcreate` | 创建卷组 |
| 创建 LV | `lvcreate` | 创建逻辑卷 |
| 扩展 LV | `lvextend --resizefs` | 在线扩展逻辑卷 |
| 创建 swap | `mkswap` + `swapon` | 创建并启用 swap |
| swappiness | `sysctl vm.swappiness=N` | 调整 swap 使用倾向 |

---

## 练习

**1. 你的服务器磁盘满了，写出排查和清理步骤。**

<details>
<summary>查看答案</summary>

```bash
# 1. Identify which filesystem is full
df -h

# 2. Check if it's an inode problem
df -i

# 3. Find the largest directories
sudo du -h --max-depth=1 / | sort -rh | head -10
# Then drill down into the largest directory
sudo du -h --max-depth=1 /var | sort -rh | head -10

# 4. Find large files
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh

# 5. Common cleanup targets:
# Old log files
sudo journalctl --vacuum-time=3d
sudo find /var/log -name "*.gz" -mtime +30 -delete

# Package cache
sudo apt clean           # Debian/Ubuntu
sudo yum clean all       # RHEL/CentOS

# Old kernels
sudo apt autoremove      # Debian/Ubuntu

# Temp files
sudo find /tmp -mtime +7 -delete
```

</details>

**2. 解释以下 `/etc/fstab` 行的含义：**

```
UUID=a1b2c3d4-e5f6 /data ext4 defaults,noatime,nosuid 0 2
```

<details>
<summary>查看答案</summary>

- `UUID=a1b2c3d4-e5f6`：通过 UUID 标识设备（比设备名更可靠）
- `/data`：挂载点
- `ext4`：文件系统类型
- `defaults,noatime,nosuid`：
  - `defaults`：默认选项（rw, suid, dev, exec, auto, nouser, async）
  - `noatime`：不更新文件的访问时间（减少磁盘写入，提升性能）
  - `nosuid`：忽略该文件系统上的 SUID/SGID 位（安全加固）
- `0`：dump 备份标志为 0（不备份）
- `2`：fsck 检查顺序为 2（在根分区之后检查）

</details>

**3. 解释 LVM 中 PV、VG、LV 的关系，并说明 LVM 相对于传统分区的优势。**

<details>
<summary>查看答案</summary>

**关系：**
- **PV（Physical Volume）**：物理层，可以是整块硬盘或分区
- **VG（Volume Group）**：由一个或多个 PV 组成的存储池
- **LV（Logical Volume）**：从 VG 中划分出来的逻辑分区，格式化后可挂载使用

层级关系：`物理磁盘 → PV → VG → LV → 文件系统`

**相对于传统分区的优势：**
1. **在线扩容**：可以在不卸载的情况下扩展 LV（`lvextend --resizefs`）
2. **跨磁盘**：一个 VG 可以包含多块物理磁盘，LV 可以使用来自不同磁盘的空间
3. **灵活分配**：可以动态调整各 LV 的大小，不受物理分区边界限制
4. **快照**：支持创建 LV 快照，用于备份或测试
5. **添加磁盘**：可以随时向 VG 添加新磁盘来扩展总容量

</details>

**4. 在什么场景下 `df` 显示空间充足但文件系统仍然报 "No space left on device"？如何排查？**

<details>
<summary>查看答案</summary>

**原因：inode 耗尽**

每个文件（不论大小）都占用一个 inode。当 inode 用完时，即使有剩余空间也无法创建新文件。

```bash
# Check inode usage
df -i
# Look for IUse% close to 100%

# Find directory with most files
sudo find / -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head -20
```

**常见原因：**
- 大量小文件（如邮件队列、session 文件、缓存文件）
- 应用产生了海量临时文件

**解决方法：**
- 删除不需要的文件
- 对于新文件系统，创建时指定更多 inode：`mkfs.ext4 -N 2000000 /dev/sdb1`

</details>

**5. 如何安全地将一台正在运行的服务器的 `/home` 分区从 50GB 扩展到 100GB？假设使用 LVM。**

<details>
<summary>查看答案</summary>

```bash
# 1. Check current status
sudo lvs
sudo vgs    # Verify free space in VG
df -h /home

# 2a. If VG has free space, extend LV directly
sudo lvextend -L 100G /dev/vg_name/home_lv

# 2b. If VG lacks space, add a new disk first
sudo pvcreate /dev/sdd
sudo vgextend vg_name /dev/sdd
sudo lvextend -L 100G /dev/vg_name/home_lv

# 3. Resize the filesystem (online, no downtime)
# For ext4:
sudo resize2fs /dev/vg_name/home_lv
# For XFS:
sudo xfs_growfs /home

# 4. Verify
df -h /home
sudo lvs
```

关键点：
- LVM 支持在线扩展，无需卸载 `/home`
- `lvextend` 和 `resize2fs` 都可以在文件系统挂载状态下执行
- 可以用 `lvextend -L 100G --resizefs` 一步完成扩展和调整
- 扩展前建议做备份以防万一

</details>
