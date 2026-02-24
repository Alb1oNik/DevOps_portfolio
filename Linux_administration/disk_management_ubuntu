# Complete Guide to Disk Management in Ubuntu

**Version:** 1.0  
**Date:** October 2025  
**Author:** Practical Guide for System Administrators

---

## Table of Contents

1. [Fundamentals of Disk Architecture in Linux](#1-fundamentals-of-disk-architecture-in-linux)
2. [Basic Commands and Tools](#2-basic-commands-and-tools)
3. [Working with Disk Partitions](#3-working-with-disk-partitions)
4. [File Systems](#4-file-systems)
5. [Mounting and Auto-mounting](#5-mounting-and-auto-mounting)
6. [LVM - Logical Volume Manager](#6-lvm---logical-volume-manager)
7. [RAID Arrays](#7-raid-arrays)
8. [Performance and Monitoring](#8-performance-and-monitoring)
9. [Diagnostics and Troubleshooting](#9-diagnostics-and-troubleshooting)
10. [Additional Features](#10-additional-features)
11. [Cheat Sheet - Quick Reference](#11-cheat-sheet---quick-reference)

---

## 1. Fundamentals of Disk Architecture in Linux

### 1.1 "Everything is a File" Philosophy

In Linux, **everything** is represented as a file. Disks are no exception.

```bash
ls -la /dev/sd*
# brw-rw---- 1 root disk 8,  0 Jan  1 12:00 /dev/sda
# brw-rw---- 1 root disk 8,  1 Jan  1 12:00 /dev/sda1
# brw-rw---- 1 root disk 8,  2 Jan  1 12:00 /dev/sda2
```

**Explanation:**
- `b` at the beginning = **block device**
- `8,0` = **major,minor** device numbers
- `root disk` = owner and group

### 1.2 Device Types

#### Block Devices
- Support random data access
- Work with data in blocks (typically 512 bytes or 4KB)
- Have kernel-level buffering
- Examples: HDD, SSD, USB drives

#### Character Devices
- Sequential data access
- No buffering
- Examples: terminals, mouse, keyboard

### 1.3 Disk Naming

#### SATA/SCSI disks:
```
/dev/sda    - first SATA/SCSI disk (entire disk)
/dev/sdb    - second SATA/SCSI disk
/dev/sda1   - first partition of first disk
/dev/sda2   - second partition of first disk
```

#### NVMe disks:
```
/dev/nvme0n1     - first NVMe disk
/dev/nvme0n1p1   - first partition
/dev/nvme0n1p2   - second partition
/dev/nvme1n1     - second NVMe disk
```

**NVMe breakdown:**
- `nvme0` - controller 0
- `n1` - namespace 1
- `p1` - partition 1

#### Virtual disks:
```
/dev/vda1        - KVM/QEMU
/dev/xvda1       - Xen
/dev/mmcblk0p1   - SD cards
```

### 1.4 Major and Minor Numbers

```bash
ls -l /dev/sda1
# brw-rw---- 1 root disk 8, 1 Jan  1 12:00 /dev/sda1
#                        ^  ^
#                        |  └─ Minor number
#                        └─ Major number
```

**Major number** - defines **driver type**:
```bash
cat /proc/devices
# Block devices:
#   8 sd      (SCSI/SATA disks)
#   9 md      (Software RAID)
# 253 device-mapper (LVM, LUKS)
```

**Minor number** - defines **specific device**:
- `/dev/sda` = 8:0
- `/dev/sda1` = 8:1
- `/dev/sda2` = 8:2

---

## 2. Basic Commands and Tools

### 2.1 Viewing Disks and Partitions

#### lsblk - tree view of block devices
```bash
lsblk
lsblk -f          # with UUID and FS type
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,UUID
```

**Output:**
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
├─sda1   8:1    0     1G  0 part /boot
└─sda2   8:2    0    99G  0 part /
sdb      8:16   0   500G  0 disk 
```

#### fdisk - partition management (MBR)
```bash
fdisk -l              # list all disks
fdisk -l /dev/sda     # specific disk
fdisk /dev/sda        # interactive mode
```

#### gdisk - partition management (GPT)
```bash
gdisk -l /dev/sda     # list GPT partitions
gdisk /dev/sda        # interactive mode
```

#### parted - universal tool
```bash
parted -l             # list all disks
parted /dev/sda print # disk information
parted /dev/sda       # interactive mode
```

#### blkid - UUID and filesystem types
```bash
blkid                 # all devices
blkid /dev/sda1       # specific device
```

**Output:**
```
/dev/sda1: UUID="550e8400-e29b-41d4-a716-446655440000" 
           TYPE="ext4" 
           PARTUUID="12345678-01"
```

### 2.2 Filesystem Information

```bash
df -h                 # mounted filesystem usage
df -i                 # inode usage
du -sh /path          # directory size
du -h --max-depth=1 / | sort -hr | head -20  # top directories
```

### 2.3 Mount Information

```bash
mount                 # all mounts
mount | grep sda1     # specific device
cat /proc/mounts      # kernel view
cat /etc/mtab         # user view
findmnt               # tree view of mounts
findmnt /dev/sda1     # specific device
```

---

## 3. Working with Disk Partitions

### 3.1 Partition Tables: MBR vs GPT

#### MBR (Master Boot Record)
**Limitations:**
- Maximum 4 primary partitions
- Maximum 2TB per disk
- One Extended partition can contain multiple Logical partitions

**Usage:**
```bash
fdisk /dev/sdb

fdisk commands:
n - new partition
d - delete partition
t - change partition type
p - print partitions
w - write changes
q - quit without saving
```

#### GPT (GUID Partition Table)
**Advantages:**
- Up to 128 partitions by default
- Disks up to 9.4 ZB
- Backup partition table
- Integrity checks (CRC32)

**Usage:**
```bash
gdisk /dev/sdb

gdisk commands:
n - new partition
d - delete partition
t - change partition type
p - print partitions
w - write changes
q - quit without saving
```

### 3.2 Practice: Creating Partitions

#### Scenario 1: Single partition (GPT)
```bash
# 1. Launch gdisk
gdisk /dev/sdb

# 2. Create GPT table (if needed)
o  # create new empty GPT table

# 3. Create partition
n  # new partition
1  # partition number
Enter  # start (default)
Enter  # end (entire disk)
8300  # type: Linux filesystem

# 4. Write
w  # write changes
y  # confirm
```

#### Scenario 2: Multiple partitions
```bash
gdisk /dev/sdb

# Partition 1: 50GB
n → 1 → Enter → +50G → 8300

# Partition 2: remaining space
n → 2 → Enter → Enter → 8300

w → y
```

#### Scenario 3: Swap partition
```bash
gdisk /dev/sdb

# Swap partition
n → 1 → Enter → +8G → 8200  # 8200 = Linux swap

w → y
```

### 3.3 Partition Types

#### For MBR (fdisk):
```
82 - Linux swap
83 - Linux
8e - Linux LVM
fd - Linux raid autodetect
```

#### For GPT (gdisk):
```
8200 - Linux swap
8300 - Linux filesystem
8e00 - Linux LVM
fd00 - Linux RAID
ef00 - EFI System Partition
```

### 3.4 Deleting Partitions

```bash
# WARNING: All data will be lost!

# fdisk
fdisk /dev/sdb
d  # delete
1  # partition number
w  # write

# gdisk
gdisk /dev/sdb
d  # delete
1  # partition number
w  # write

# parted
parted /dev/sdb
rm 1  # remove partition 1
quit
```

---

## 4. File Systems

### 4.1 Filesystem Types

#### EXT4 (Fourth Extended Filesystem)
**Features:**
- Standard for Ubuntu
- Journaling for crash protection
- Extents for contiguous blocks
- Maximum file size: 16TB
- Maximum FS size: 1EB

**Commands:**
```bash
# Create
mkfs.ext4 /dev/sdb1
mkfs.ext4 -L MyDisk /dev/sdb1  # with label

# Information
tune2fs -l /dev/sdb1
dumpe2fs /dev/sdb1

# Resize
resize2fs /dev/sdb1
resize2fs /dev/sdb1 50G  # to specific size

# Check
e2fsck -f /dev/sdb1
fsck.ext4 /dev/sdb1
```

#### XFS
**Features:**
- High performance
- Files up to 8 exabytes
- Online defragmentation
- Cannot shrink (only grow)

**Commands:**
```bash
# Create
mkfs.xfs /dev/sdb1

# Information
xfs_info /dev/sdb1

# Grow size (only!)
xfs_growfs /mount/point

# Defragmentation
xfs_fsr /mount/point

# Check
xfs_repair /dev/sdb1
```

#### BTRFS
**Features:**
- Copy-on-Write (COW)
- Snapshots
- Subvolumes
- Built-in RAID
- Transparent compression

**Commands:**
```bash
# Create
mkfs.btrfs /dev/sdb1

# Subvolume
btrfs subvolume create /mnt/btrfs/subvol1
btrfs subvolume list /mnt/btrfs

# Snapshot
btrfs subvolume snapshot /mnt/btrfs/subvol1 /mnt/btrfs/snapshot1

# Information
btrfs filesystem show
btrfs filesystem usage /mnt/btrfs
```

#### FAT32 / exFAT
**Usage:**
- USB flash drives
- Windows compatibility

**Commands:**
```bash
# FAT32
mkfs.vfat /dev/sdb1
mkfs.vfat -F 32 /dev/sdb1

# exFAT
mkfs.exfat /dev/sdb1
```

#### NTFS
**Usage:**
- Windows disks

**Commands:**
```bash
# Create
mkfs.ntfs /dev/sdb1

# Quick format
mkfs.ntfs -f /dev/sdb1

# Change label
ntfslabel /dev/sdb1 NewLabel
```

### 4.2 ext4 Filesystem Structure

```
┌─────────────────────────────────────────────────────────┐
│                    PARTITION /dev/sda1                  │
├─────────────┬──────────────┬────────────┬──────────────┤
│ Superblock  │ Group Desc   │ Block Grp 0│ Block Grp 1  │
│ (metadata)  │ (descriptors)│            │              │
└─────────────┴──────────────┴────────────┴──────────────┘
```

**Superblock contains:**
- Magic number (0xEF53 for ext4)
- Block size (usually 4096 bytes)
- Number of inodes and blocks
- FS state (clean/dirty)

**Inode (index descriptor) contains:**
- Access permissions (rwx)
- Owner (UID/GID)
- File size
- Timestamps (atime, mtime, ctime)
- Number of hard links
- Pointers to data blocks

### 4.3 Working with Disk Labels

```bash
# View label
blkid /dev/sdb1

# Set label
# ext4
e2label /dev/sdb1 MyData

# xfs
xfs_admin -L MyData /dev/sdb1

# fat32
fatlabel /dev/sdb1 MyData

# ntfs
ntfslabel /dev/sdb1 MyData

# Mount by label
mount LABEL=MyData /mnt/data
```

### 4.4 UUID vs LABEL vs Device Path

```bash
blkid /dev/sda1
# UUID="550e8400-e29b-41d4-a716-446655440000"  <- FS UUID
# LABEL="MyDisk"                               <- label
# TYPE="ext4"                                  <- FS type
# PARTUUID="00000000-01"                       <- partition UUID (GPT)
```

**Differences:**
- **UUID** - unique ID of filesystem (created during formatting)
- **PARTUUID** - unique ID of partition (created during partitioning)
- **LABEL** - human-readable label (can be changed)
- **Device Path** - device path (can change!)

**Recommendation:** Always use UUID in `/etc/fstab`!

---

## 5. Mounting and Auto-mounting

### 5.1 What is Mounting?

**Mounting** - the process of connecting a filesystem to the Linux directory tree.

```
BEFORE mounting:
/mnt/data (empty directory) ← disk not visible

AFTER mounting:
/mnt/data → /dev/sdb1 (ext4) ← files accessible
```

### 5.2 mount Command

#### Basic mounting:
```bash
# Create mount point
mkdir -p /mnt/data

# Mount
mount /dev/sdb1 /mnt/data

# Verify
df -h /mnt/data
lsblk /dev/sdb1
```

#### With options:
```bash
mount -o rw,noatime /dev/sdb1 /mnt/data
mount -o ro /dev/sdb1 /mnt/data  # read-only
mount -t ext4 /dev/sdb1 /mnt/data  # specify FS type
```

#### Popular mount options:

| Option | Description |
|--------|-------------|
| `rw` | read-write (default) |
| `ro` | read-only |
| `noatime` | don't update access time → performance |
| `relatime` | update atime only when modified |
| `noexec` | prevent program execution |
| `nosuid` | ignore SUID bits |
| `user` | allow regular users to mount |
| `auto` | mount at boot |
| `noauto` | don't mount automatically |
| `defaults` | rw,suid,dev,exec,auto,nouser,async |

### 5.3 umount Command

```bash
# Unmount by mount point
umount /mnt/data

# Unmount by device
umount /dev/sdb1

# Force unmount
umount -f /mnt/data

# Lazy umount (deferred)
umount -l /mnt/data
```

#### "device is busy" error:
```bash
# Find processes using the partition
lsof +D /mnt/data
fuser -m /mnt/data

# Kill processes (careful!)
fuser -km /mnt/data

# Or exit from directory
cd /
umount /mnt/data
```

### 5.4 Auto-mounting via /etc/fstab

#### File format:
```
<device>  <mountpoint>  <fstype>  <options>  <dump>  <pass>
```

#### Example /etc/fstab:
```bash
# Root partition
UUID=550e8400-e29b-41d4-a716-446655440000  /      ext4  defaults            0  1

# Home partition
UUID=f47ac10b-58cc-4372-a567-0e02b2c3d479  /home  ext4  defaults            0  2

# Additional data partition
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /data  ext4  noatime,nodiratime  0  2

# Windows partition
/dev/sdb1  /mnt/windows  ntfs-3g  defaults,uid=1000,gid=1000  0  0

# Swap
UUID=12345678-90ab-cdef-1234-567890abcdef  none   swap  sw                  0  0

# USB with noauto (mount manually)
LABEL=MyUSB  /media/usb  vfat  noauto,user,rw  0  0
```

#### Field explanation:

**1. Device (what to mount):**
- `/dev/sda1` - device path
- `UUID=...` - by UUID (recommended!)
- `LABEL=MyDisk` - by label

**2. Mount point (where):**
- `/` - root partition
- `/home`, `/var`, `/boot` - system
- `/mnt/data` - user-defined

**3. Filesystem type:**
- `ext4`, `xfs`, `btrfs` - Linux
- `ntfs-3g` - Windows NTFS
- `vfat` - FAT32
- `swap` - swap partition

**4. Options:**
- `defaults` = rw,suid,dev,exec,auto,nouser,async
- See options table above

**5. Dump (0 or 1):**
- `0` - don't backup with dump
- `1` - create backup

**6. Pass (fsck order):**
- `0` - don't check
- `1` - check first (only `/`)
- `2` - check second (other partitions)

#### Apply fstab changes:
```bash
# Check syntax
mount -fav

# Mount everything from fstab
mount -a

# Mount specific point (using fstab)
mount /mnt/data
```

### 5.5 Special Mount Types

#### Bind mount (directory mirroring):
```bash
mount --bind /source /destination

# Example: make /home/user accessible in /mnt
mount --bind /home/user /mnt/user
```

#### Mount ISO image:
```bash
mount -o loop /path/to/image.iso /mnt/iso
```

#### Remount (change options on the fly):
```bash
# Remount as read-only
mount -o remount,ro /mnt/data

# Remount as read-write
mount -o remount,rw /mnt/data
```

---

## 6. LVM - Logical Volume Manager

### 6.1 LVM Architecture

```
PHYSICAL DISKS
    ↓
┌─────────────────────────────────────┐
│ Physical Volumes (PV)               │
│ /dev/sda1  /dev/sdb1  /dev/sdc1    │
└─────────────────────────────────────┘
    ↓ (combined into pool)
┌─────────────────────────────────────┐
│ Volume Group (VG)                   │
│ vg_main (300GB total space)         │
└─────────────────────────────────────┘
    ↓ (divided into logical volumes)
┌─────────────────────────────────────┐
│ Logical Volumes (LV)                │
│ lv_root  lv_home  lv_var  lv_data  │
└─────────────────────────────────────┘
    ↓
FILE SYSTEMS
/dev/vg_main/lv_root → ext4 → /
/dev/vg_main/lv_home → ext4 → /home
```

### 6.2 Creating LVM

#### Step 1: Physical Volumes
```bash
# Create PV on physical disks/partitions
pvcreate /dev/sdb1
pvcreate /dev/sdc1

# Or multiple at once
pvcreate /dev/sdb1 /dev/sdc1

# View PV
pvdisplay
pvs  # short version
pvscan
```

#### Step 2: Volume Group
```bash
# Create VG from multiple PVs
vgcreate vg_storage /dev/sdb1 /dev/sdc1

# View VG
vgdisplay
vgs  # short version
vgscan
```

#### Step 3: Logical Volumes
```bash
# Create LV with fixed size
lvcreate -L 50G -n lv_data vg_storage

# Create LV using all free space
lvcreate -l 100%FREE -n lv_data vg_storage

# Create LV with 50% of VG
lvcreate -l 50%VG -n lv_data vg_storage

# View LV
lvdisplay
lvs  # short version
lvscan
```

#### Step 4: Create FS and mount
```bash
# Create filesystem
mkfs.ext4 /dev/vg_storage/lv_data

# Mount
mkdir /storage/data
mount /dev/vg_storage/lv_data /storage/data

# In fstab:
# /dev/vg_storage/lv_data  /storage/data  ext4  defaults  0  2
```

### 6.3 Managing LVM

#### Extending Volume Group:
```bash
# Add new disk to VG
pvcreate /dev/sdd1
vgextend vg_storage /dev/sdd1

# Verify
vgs
```

#### Extending Logical Volume:
```bash
# Increase LV by 20GB
lvextend -L +20G /dev/vg_storage/lv_data

# Or use all free space in VG
lvextend -l +100%FREE /dev/vg_storage/lv_data

# Extend filesystem
# For ext4:
resize2fs /dev/vg_storage/lv_data

# For xfs (must be mounted):
xfs_growfs /storage/data

# All in one command (with automatic FS resize):
lvextend -L +20G -r /dev/vg_storage/lv_data
```

#### Reducing Logical Volume (DANGEROUS!):
```bash
# WARNING: only for ext2/ext3/ext4
# XFS cannot be shrunk!

# 1. Unmount
umount /storage/data

# 2. Check FS
e2fsck -f /dev/vg_storage/lv_data

# 3. Shrink FS
resize2fs /dev/vg_storage/lv_data 40G

# 4. Shrink LV
lvreduce -L 40G /dev/vg_storage/lv_data

# 5. Mount back
mount /dev/vg_storage/lv_data /storage/data
```

#### Removing LVM components:
```bash
# Remove LV
umount /storage/data
lvremove /dev/vg_storage/lv_data

# Remove VG
vgremove vg_storage

# Remove PV
pvremove /dev/sdb1
```

### 6.4 LVM Snapshots

```bash
# Create snapshot (5GB for storing changes)
lvcreate -L 5G -s -n lv_data_snap /dev/vg_storage/lv_data

# Mount snapshot (read-only or rw)
mkdir /mnt/snapshot
mount -o ro /dev/vg_storage/lv_data_snap /mnt/snapshot

# Revert to snapshot (merge)
umount /storage/data
umount /mnt/snapshot
lvconvert --merge /dev/vg_storage/lv_data_snap
# After reboot, snapshot will be merged with original

# Delete snapshot
umount /mnt/snapshot
lvremove /dev/vg_storage/lv_data_snap
```

### 6.5 LVM Advantages

✅ **Flexibility:** Resize on the fly  
✅ **Snapshots:** Instant backups for testing  
✅ **Migration:** Move data between disks without downtime  
✅ **Pooling:** Multiple disks in one pool  
✅ **Striping/Mirroring:** Performance and reliability  

---

## 7. RAID Arrays

### 7.1 RAID Types

#### RAID 0 (Striping)
```
┌────┬────┬────┬────┐
│ A1 │ A2 │ A3 │ A4 │  Data split across disks
└────┴────┴────┴────┘
Disk1  Disk2
```
✅ **Speed:** 2x (data written in parallel)  
❌ **Reliability:** 0 (one disk fails = all data lost)  
**Capacity:** N disks  

#### RAID 1 (Mirroring)
```
┌────┐  ┌────┐
│ A1 │  │ A1 │  Complete duplication
│ A2 │  │ A2 │
│ A3 │  │ A3 │
└────┘  └────┘
Disk1   Disk2
```
✅ **Reliability:** High (one disk fails = data intact)  
❌ **Capacity:** N/2 (from 2TB get 1TB)  
**Read speed:** 2x, write: 1x  

#### RAID 5 (Striping + Parity)
```
┌────┬────┬────┐
│ A1 │ A2 │ Ap │  Parity = checksum
│ B1 │ Bp │ B2 │
│ Cp │ C1 │ C2 │
└────┴────┴────┘
Disk1 Disk2 Disk3
```
✅ **Balance:** Speed + reliability  
✅ **Capacity:** (N-1) disks  
❌ **Write:** Slow (need to calculate parity)  
**Minimum:** 3 disks  

#### RAID 6 (Striping + Double Parity)
```
┌────┬────┬────┬────┐
│ A1 │ A2 │ Ap │ Aq │  Double protection
│ B1 │ Bp │ Bq │ B2 │
└────┴────┴────┴────┘
```
✅ **Reliability:** Can lose 2 disks  
✅ **Capacity:** (N-2) disks  
**Minimum:** 4 disks  

#### RAID 10 (1+0, Striped Mirrors)
```
       RAID 0 (Striping)
    ┌────────┴────────┐
RAID 1            RAID 1
┌─┴─┐            ┌─┴─┐
D1  D2           D3  D4
```
✅ **Speed:** High (2x read, 1x write)  
✅ **Reliability:** High  
❌ **Capacity:** N/2  
**Minimum:** 4 disks  

### 7.2 Creating RAID Arrays (mdadm)

#### Install mdadm:
```bash
apt-get install mdadm
```

#### RAID 1 (mirror):
```bash
# Create RAID 1 array
mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb1 /dev/sdc1

# Create filesystem
mkfs.ext4 /dev/md0

# Mount
mkdir /mnt/raid1
mount /dev/md0 /mnt/raid1

# Save configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# In fstab:
# /dev/md0  /mnt/raid1  ext4  defaults  0  2
```

#### RAID 5:
```bash
# Create RAID 5 from 3 disks
mdadm --create /dev/md0 \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1

# Remaining steps same as RAID 1
```

#### RAID 10:
```bash
# Create RAID 10 from 4 disks
mdadm --create /dev/md0 \
  --level=10 \
  --raid-devices=4 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
```

### 7.3 Monitoring RAID

```bash
# Status of all RAID arrays
cat /proc/mdstat
# md0 : active raid1 sdc1[1] sdb1[0]
#       104320 blocks [2/2] [UU]

# Detailed information
mdadm --detail /dev/md0

# Information about disk in RAID
mdadm --examine /dev/sdb1

# Real-time monitoring
watch -n 1 cat /proc/mdstat
```

### 7.4 Managing RAID

#### Adding disk:
```bash
# Add spare disk
mdadm --add /dev/md0 /dev/sde1

# Increase number of active disks in RAID 5
mdadm --grow /dev/md0 --raid-devices=4
```

#### Replacing failed disk:
```bash
# 1. Mark disk as failed
mdadm --fail /dev/md0 /dev/sdb1

# 2. Remove failed disk
mdadm --remove /dev/md0 /dev/sdb1

# 3. Physically replace disk

# 4. Add new disk
mdadm --add /dev/md0 /dev/sdb1

# 5. Watch synchronization
watch cat /proc/mdstat
```

#### Stopping and assembling RAID:
```bash
# Stop RAID
umount /mnt/raid1
mdadm --stop /dev/md0

# Assemble RAID (after reboot)
mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1

# Auto-assembly at boot
mdadm --assemble --scan
```

#### Removing RAID:
```bash
# 1. Unmount
umount /mnt/raid1

# 2. Stop array
mdadm --stop /dev/md0

# 3. Zero superblocks on disks
mdadm --zero-superblock /dev/sdb1
mdadm --zero-superblock /dev/sdc1

# 4. Remove from configuration
nano /etc/mdadm/mdadm.conf
```

---

## 8. Performance and Monitoring

### 8.1 Key Metrics

#### iostat - I/O statistics
```bash
# Installation
apt-get install sysstat

# Basic usage
iostat

# Extended statistics (every second)
iostat -x 1

# Output:
# Device  r/s   w/s  rkB/s  wkB/s  %util  await  svctm
# sda    10.0  20.0   400    800    45.0   15.2   2.1
```

**Key indicators:**
- **r/s, w/s** - read/write operations per second
- **rkB/s, wkB/s** - kilobytes read/written per second
- **%util** - disk utilization (>80% = bottleneck)
- **await** - average wait time in ms (>20ms = slow)
- **svctm** - service time (deprecated in newer versions)

#### iotop - top processes by I/O
```bash
# Installation
apt-get install iotop

# Interactive mode
iotop

# Accumulative statistics
iotop -ao

# Only processes (not threads)
iotop -P

# Batch mode for logging
iotop -b -n 5 > io_log.txt
```

#### dstat - comprehensive statistics
```bash
# Installation
apt-get install dstat

# Basic usage
dstat

# CPU, disk, network, memory
dstat -cdngy

# Top processes by I/O
dstat --top-bio --top-io
```

### 8.2 I/O Schedulers

```bash
# View current scheduler
cat /sys/block/sda/queue/scheduler
# [mq-deadline] kyber bfq none

# Change scheduler
echo noop > /sys/block/sda/queue/scheduler
echo deadline > /sys/block/sda/queue/scheduler
echo bfq > /sys/block/sda/queue/scheduler
```

**Scheduler types:**

| Scheduler | Description | Best for |
|-----------|-------------|----------|
| **noop/none** | Minimal processing (FIFO) | SSD, NVMe |
| **deadline/mq-deadline** | Guarantees max wait time | Servers, databases |
| **bfq** | Budget Fair Queuing | Desktops, low latency |
| **cfq** | Completely Fair Queuing (deprecated) | Old HDDs |

### 8.3 Performance Testing

#### hdparm - speed test
```bash
# Buffered read speed
hdparm -t /dev/sda
# Timing buffered disk reads: 500 MB in 3.01 seconds = 166.11 MB/sec

# Cached read speed
hdparm -T /dev/sda
# Timing cached reads: 12000 MB in 2.00 seconds = 6000.00 MB/sec

# Disk information
hdparm -I /dev/sda
```

#### dd - simple test
```bash
# Write test (1GB file)
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct
# 1073741824 bytes copied, 5.5 s, 195 MB/s

# Read test
dd if=/tmp/testfile of=/dev/null bs=1G count=1 iflag=direct

# Cleanup
rm /tmp/testfile
```

#### fio - professional benchmark
```bash
# Installation
apt-get install fio

# Random read 4K blocks
fio --name=random-read --ioengine=libaio --iodepth=32 \
    --rw=randread --bs=4k --direct=1 --size=1G --numjobs=4 \
    --runtime=60 --group_reporting

# Sequential write
fio --name=seq-write --ioengine=libaio --iodepth=32 \
    --rw=write --bs=1m --direct=1 --size=1G \
    --runtime=60 --group_reporting
```

### 8.4 SMART Monitoring

```bash
# Installation
apt-get install smartmontools

# Health status
smartctl -H /dev/sda
# SMART overall-health self-assessment test result: PASSED

# Full information
smartctl -a /dev/sda

# Error log
smartctl -l error /dev/sda

# Run short test
smartctl -t short /dev/sda

# Run long test
smartctl -t long /dev/sda

# View test results
smartctl -l selftest /dev/sda

# Enable automatic monitoring
systemctl enable smartd
systemctl start smartd
```

**Important SMART attributes:**
- **Reallocated_Sector_Ct** - reallocated sectors
- **Current_Pending_Sector** - sectors awaiting reallocation
- **Offline_Uncorrectable** - uncorrectable errors
- **Temperature_Celsius** - temperature
- **Power_On_Hours** - operating hours
- **Power_Cycle_Count** - power cycle count

### 8.5 Performance Optimization

#### Disable atime:
```bash
# In fstab change:
# UUID=xxx  /  ext4  defaults  0  1
# to:
# UUID=xxx  /  ext4  noatime,nodiratime  0  1

# Or remount:
mount -o remount,noatime,nodiratime /
```

#### Readahead (read-ahead):
```bash
# View current value
blockdev --getra /dev/sda
# 256 (in 512-byte sectors)

# Set (e.g., 8MB = 16384 sectors)
blockdev --setra 16384 /dev/sda
```

#### SSD TRIM:
```bash
# Manual TRIM
fstrim -v /

# Automatic TRIM in fstab:
# UUID=xxx  /  ext4  discard  0  1

# Or schedule via systemd timer
systemctl enable fstrim.timer
systemctl start fstrim.timer
```

#### Swappiness:
```bash
# View current value (0-100)
cat /proc/sys/vm/swappiness
# 60 (default)

# Change temporarily (10 = less swap)
sysctl vm.swappiness=10

# Change permanently
echo "vm.swappiness=10" >> /etc/sysctl.conf
```

---

## 9. Diagnostics and Troubleshooting

### 9.1 "No space left on device"

#### Problem 1: Out of inodes
```bash
# Check inode usage
df -i

# Filesystem     Inodes  IUsed  IFree IUse% Mounted on
# /dev/sda1     6553600 6553600    0  100% /

# Solution: delete unnecessary files or recreate FS
find / -xdev -type f | cut -d "/" -f 2 | sort | uniq -c | sort -n
```

#### Problem 2: Deleted but open files
```bash
# Find processes with open deleted files
lsof +L1

# COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NLINK NODE NAME
# java     1234 app    3w   REG  253,0  5000000     0  123  /var/log/app.log (deleted)

# Solution 1: restart process
systemctl restart application

# Solution 2: truncate file
> /proc/1234/fd/3

# Solution 3: kill process
kill -9 1234
```

#### Problem 3: Reserved for root (5%)
```bash
# Check reserved space
tune2fs -l /dev/sda1 | grep "Reserved block count"

# Reduce to 1% (for data partitions)
tune2fs -m 1 /dev/sda1

# Remove reserve completely (0%)
tune2fs -m 0 /dev/sda1
```

### 9.2 Read-only filesystem

```bash
# Cause: FS errors, kernel switched to RO for safety

# 1. Try remount as RW
mount -o remount,rw /

# 2. If doesn't help - unmount and check
umount /dev/sda1
fsck -y /dev/sda1

# 3. Mount back
mount /dev/sda1 /mnt

# For root partition:
# - Boot from Live USB
# - fsck -y /dev/sda1
# - Reboot
```

### 9.3 Slow System / High I/O wait

```bash
# 1. Check I/O wait
top
# %Cpu(s): 10.0 us, 5.0 sy, 0.0 ni, 20.0 id, 65.0 wa
#                                              ^^^^^ high wait

# 2. Find processes with high I/O
iotop -ao

# 3. Check disk utilization
iostat -x 1
# %util > 80% = disk overloaded

# 4. Check errors
dmesg | grep -i error
journalctl -xe | grep -i disk

# 5. SMART diagnostics
smartctl -H /dev/sda
smartctl -a /dev/sda

# 6. Check bad sectors
badblocks -sv /dev/sda > /tmp/badblocks.txt
```

### 9.4 Bad Sectors

```bash
# Check (read-only, safe)
badblocks -sv /dev/sda

# Check with write (WILL DELETE ALL DATA!)
badblocks -wsv /dev/sda

# Add bad blocks to ext4
badblocks -sv /dev/sda1 > /tmp/badblocks.txt
e2fsck -l /tmp/badblocks.txt /dev/sda1
```

### 9.5 Data Recovery

#### TestDisk - partition recovery
```bash
apt-get install testdisk

# Run
testdisk /dev/sda

# Follow on-screen instructions
```

#### PhotoRec - file recovery
```bash
# Included in testdisk package
photorec /dev/sda1

# Select FS type and save directory
```

#### ddrescue - cloning damaged disk
```bash
apt-get install gddrescue

# Clone disk with recovery attempts
ddrescue -f -n /dev/sda /dev/sdb rescue.log

# -f = force (overwrite)
# -n = no-scrape (fast pass)
```

### 9.6 Check and Repair Different FS

```bash
# ext4
e2fsck -f -y /dev/sda1

# xfs
xfs_repair /dev/sda1

# btrfs
btrfs check /dev/sda1
btrfs check --repair /dev/sda1

# ntfs
ntfsfix /dev/sda1

# fat32
fsck.vfat -a /dev/sda1
```

### 9.7 Log Analysis

```bash
# Errors in dmesg
dmesg | grep -i error
dmesg | grep -i "I/O error"

# systemd journal
journalctl -xe
journalctl -u systemd-fsck@dev-sda1.service

# SMART logs
smartctl -l error /dev/sda

# System logs
tail -f /var/log/syslog
grep -i disk /var/log/syslog
```

---

## 10. Additional Features

### 10.1 SWAP (swap space)

#### Creating swap partition:
```bash
# 1. Create swap partition (fdisk, type 82)
fdisk /dev/sdb
# n → p → 1 → Enter → +8G → t → 82 → w

# 2. Format as swap
mkswap /dev/sdb1

# 3. Enable swap
swapon /dev/sdb1

# 4. Verify
swapon --show
free -h

# 5. In fstab for auto-mounting:
# /dev/sdb1  none  swap  sw  0  0
```

#### Creating swap file:
```bash
# 1. Create 4GB file
fallocate -l 4G /swapfile
# or
dd if=/dev/zero of=/swapfile bs=1M count=4096

# 2. Set permissions
chmod 600 /swapfile

# 3. Format
mkswap /swapfile

# 4. Enable
swapon /swapfile

# 5. In fstab:
# /swapfile  none  swap  sw  0  0

# Remove swap file:
swapoff /swapfile
rm /swapfile
```

#### Managing swappiness:
```bash
# View (0-100, 60 default)
cat /proc/sys/vm/swappiness

# Change temporarily (10 = less swap usage)
sysctl vm.swappiness=10

# Permanently
echo "vm.swappiness=10" >> /etc/sysctl.conf
```

### 10.2 Disk Encryption (LUKS)

#### Creating encrypted partition:
```bash
# 1. Encrypt partition
cryptsetup luksFormat /dev/sdb1
# Confirm: YES
# Enter password

# 2. Open (unlock)
cryptsetup luksOpen /dev/sdb1 cryptdata
# Enter password

# 3. Create filesystem
mkfs.ext4 /dev/mapper/cryptdata

# 4. Mount
mount /dev/mapper/cryptdata /mnt/secure

# 5. Work with files
# ...

# 6. Unmount and close
umount /mnt/secure
cryptsetup luksClose cryptdata
```

#### Key management:
```bash
# Add new password
cryptsetup luksAddKey /dev/sdb1

# Remove password
cryptsetup luksRemoveKey /dev/sdb1

# Show key slots
cryptsetup luksDump /dev/sdb1

# Change password
cryptsetup luksChangeKey /dev/sdb1
```

#### Auto-mounting encrypted partition:
```bash
# /etc/crypttab
cryptdata  /dev/sdb1  none  luks

# /etc/fstab
/dev/mapper/cryptdata  /mnt/secure  ext4  defaults  0  2

# System will ask for password at boot
```

### 10.3 Loop Devices

```bash
# Create disk image
dd if=/dev/zero of=/tmp/disk.img bs=1M count=1024

# Create loop device
losetup -f                        # find free loop
losetup /dev/loop0 /tmp/disk.img  # attach

# Create partition and FS
fdisk /dev/loop0
mkfs.ext4 /dev/loop0p1

# Mount
mount /dev/loop0p1 /mnt

# Or mount ISO directly
mount -o loop /path/to/image.iso /mnt/iso

# Detach loop
umount /mnt
losetup -d /dev/loop0
```

### 10.4 Disk Quotas

```bash
# 1. Enable in fstab
# /dev/sda1  /home  ext4  defaults,usrquota,grpquota  0  2

# 2. Remount
mount -o remount /home

# 3. Create quota files
quotacheck -cugm /home
quotaon /home

# 4. Set quota for user
edquota -u username
# blocks: soft=5000000 hard=6000000 (KB)
# inodes: soft=50000 hard=60000

# 5. Check quota
quota -u username
repquota /home

# 6. Report all users
repquota -a

# Disable quotas
quotaoff /home
```

### 10.5 tmpfs (RAM disk)

```bash
# Create tmpfs
mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk

# In fstab:
# tmpfs  /mnt/ramdisk  tmpfs  size=1G,mode=1777  0  0

# Usage
df -h /mnt/ramdisk

# /tmp often uses tmpfs by default
mount | grep tmpfs
```

### 10.6 Backup

#### dd - byte-by-byte copying:
```bash
# Clone entire disk
dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# Create image
dd if=/dev/sda of=/backup/disk.img bs=4M status=progress

# Restore from image
dd if=/backup/disk.img of=/dev/sda bs=4M status=progress

# Compressed image
dd if=/dev/sda bs=4M | gzip > /backup/disk.img.gz

# Restore compressed
gunzip -c /backup/disk.img.gz | dd of=/dev/sda bs=4M
```

#### rsync - incremental backup:
```bash
# System backup
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /mnt/backup/

# Backup with deletion on destination
rsync -aAXv --delete /source/ /destination/

# Over network
rsync -aAXvz -e ssh /local/ user@remote:/backup/
```

#### Partition table backup:
```bash
# MBR
dd if=/dev/sda of=/backup/mbr.img bs=512 count=1

# GPT
sgdisk --backup=/backup/gpt-backup.img /dev/sda

# Restore GPT
sgdisk --load-backup=/backup/gpt-backup.img /dev/sda
```

---

## 11. Cheat Sheet - Quick Reference

### Disk Information
```bash
lsblk                      # all disks tree view
lsblk -f                   # + UUID, FS
fdisk -l                   # disk details (MBR)
gdisk -l /dev/sda          # disk details (GPT)
parted -l                  # universal
blkid                      # UUID of all devices
df -h                      # FS usage
du -sh /path               # directory size
mount                      # all mounts
findmnt                    # mount tree view
```

### Partitions
```bash
fdisk /dev/sdb             # MBR partitions
gdisk /dev/sdb             # GPT partitions
parted /dev/sdb            # universal
```

### File Systems
```bash
mkfs.ext4 /dev/sdb1        # create ext4
mkfs.xfs /dev/sdb1         # create xfs
mkfs.btrfs /dev/sdb1       # create btrfs
mkfs.vfat /dev/sdb1        # create FAT32
mkswap /dev/sdb1           # create swap

tune2fs -l /dev/sdb1       # ext4 info
xfs_info /dev/sdb1         # xfs info

resize2fs /dev/sdb1        # resize ext4
xfs_growfs /mount/point    # grow xfs

e2fsck -f /dev/sdb1        # check ext4
xfs_repair /dev/sdb1       # check xfs
fsck /dev/sdb1             # universal check
```

### Mounting
```bash
mount /dev/sdb1 /mnt       # mount
mount -o ro /dev/sdb1 /mnt # read-only
mount -a                   # mount all from fstab
umount /mnt                # unmount
umount -l /mnt             # lazy umount
```

### LVM
```bash
# Physical Volumes
pvcreate /dev/sdb1
pvdisplay
pvs

# Volume Groups
vgcreate vg0 /dev/sdb1
vgextend vg0 /dev/sdc1
vgdisplay
vgs

# Logical Volumes
lvcreate -L 10G -n lv0 vg0
lvextend -L +5G /dev/vg0/lv0
lvextend -r -L +5G /dev/vg0/lv0  # with FS resize
lvdisplay
lvs

# Snapshots
lvcreate -L 5G -s -n snap /dev/vg0/lv0
lvconvert --merge /dev/vg0/snap
```

### RAID
```bash
# Create
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1

# Monitor
cat /proc/mdstat
mdadm --detail /dev/md0

# Manage
mdadm --add /dev/md0 /dev/sdd1
mdadm --fail /dev/md0 /dev/sdb1
mdadm --remove /dev/md0 /dev/sdb1
mdadm --stop /dev/md0

# Configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
```

### Performance
```bash
iostat -x 1                # I/O stats
iotop -ao                  # top by I/O
dstat -cdngy               # comprehensive stats
hdparm -t /dev/sda         # read speed test
hdparm -T /dev/sda         # cache test
smartctl -H /dev/sda       # SMART health
smartctl -a /dev/sda       # SMART full info
badblocks -sv /dev/sda     # check bad sectors
```

### SWAP
```bash
mkswap /dev/sdb1           # create swap
swapon /dev/sdb1           # enable
swapoff /dev/sdb1          # disable
swapon --show              # show all swap
free -h                    # memory/swap usage
sysctl vm.swappiness=10    # change swappiness
```

### Encryption (LUKS)
```bash
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 cryptdata
mkfs.ext4 /dev/mapper/cryptdata
mount /dev/mapper/cryptdata /mnt
umount /mnt
cryptsetup luksClose cryptdata
```

### Diagnostics
```bash
dmesg | grep -i error      # kernel log errors
journalctl -xe             # system logs
lsof +L1                   # deleted but open files
lsof +D /mnt               # processes using directory
fuser -m /mnt              # processes using partition
df -i                      # inode usage
```

---

## 12. Practical Scenarios

### Scenario 1: Adding New Disk for Data

```bash
# 1. Verify disk is visible
lsblk

# 2. Create GPT table and partition
gdisk /dev/sdb
o → n → 1 → Enter → Enter → 8300 → w → y

# 3. Create filesystem
mkfs.ext4 -L DataDisk /dev/sdb1

# 4. Create mount point
mkdir -p /data

# 5. Get UUID
blkid /dev/sdb1

# 6. Add to fstab
echo "UUID=<your-uuid>  /data  ext4  noatime,nodiratime  0  2" >> /etc/fstab

# 7. Mount
mount -a

# 8. Verify
df -h /data
```

### Scenario 2: Extending Partition with LVM

```bash
# Current situation: /home is 95% full

# 1. Check free space in VG
vgs
#   VG      #PV #LV #SN Attr   VSize  VFree
#   vg_main   2   3   0 wz--n- 300.00g 50.00g

# 2. Extend LV by 20GB
lvextend -L +20G -r /dev/vg_main/lv_home
# -r automatically resizes FS

# 3. Verify
df -h /home

# If no free space in VG - add new disk:
pvcreate /dev/sdc1
vgextend vg_main /dev/sdc1
lvextend -L +20G -r /dev/vg_main/lv_home
```

### Scenario 3: Migrating to Larger Disk

```bash
# Old disk: /dev/sda (120GB)
# New disk: /dev/sdb (500GB)

# Method 1: Cloning with dd
# 1. Boot from Live USB
# 2. Clone
dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# 3. Expand partition table on new disk
gdisk /dev/sdb
x → e → w → y  # expand GPT to full disk
gdisk /dev/sdb
d → 2 → n → 2 → Enter → Enter → w → y  # recreate partition

# 4. Expand FS (if LVM - see scenario 2)
resize2fs /dev/sdb2

# 5. Install bootloader
mount /dev/sdb2 /mnt
mount /dev/sdb1 /mnt/boot
for i in /dev /dev/pts /proc /sys /run; do mount -B $i /mnt$i; done
chroot /mnt
grub-install /dev/sdb
update-grub
exit

# 6. Reboot, select boot from new disk
```

### Scenario 4: Recovery After RAID Failure

```bash
# RAID 1 array, disk /dev/sdb failed

# 1. Check status
cat /proc/mdstat
# md0 : active raid1 sdc1[1] sdb1[0](F)
#       [_U]  ← one disk failed

# 2. Remove failed disk
mdadm --fail /dev/md0 /dev/sdb1
mdadm --remove /dev/md0 /dev/sdb1

# 3. Physically replace disk

# 4. Copy partition table (if needed)
sfdisk -d /dev/sdc | sfdisk /dev/sdb

# 5. Add to array
mdadm --add /dev/md0 /dev/sdb1

# 6. Watch synchronization
watch cat /proc/mdstat
# recovery = 25.5% (...)
```

### Scenario 5: Cleaning Up Full Disk

```bash
# Disk is 95% full

# 1. Find large directories
du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20

# 2. Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr | head

# 3. Clean package cache
apt-get clean
apt-get autoclean
apt-get autoremove

# 4. Clean logs
journalctl --vacuum-size=100M
journalctl --vacuum-time=7d

# 5. Remove old kernels (Ubuntu)
apt-get autoremove --purge

# 6. Find and remove duplicates
apt-get install fdupes
fdupes -r /home/user/

# 7. Check deleted but open files
lsof +L1

# 8. Reduce reserved space for root (if data partition)
tune2fs -m 1 /dev/sda1  # reduce from 5% to 1%
```

### Scenario 6: Creating RAID 5 with LVM

```bash
# 3 disks 1TB each for RAID 5 + LVM

# 1. Create RAID 5
mdadm --create /dev/md0 --level=5 --raid-devices=3 \
  /dev/sdb /dev/sdc /dev/sdd

# 2. Wait for synchronization
cat /proc/mdstat

# 3. Create LVM on top of RAID
pvcreate /dev/md0
vgcreate vg_raid /dev/md0

# 4. Create logical volumes
lvcreate -L 500G -n lv_data vg_raid
lvcreate -L 300G -n lv_backup vg_raid
lvcreate -l 100%FREE -n lv_archive vg_raid

# 5. Create FS
mkfs.ext4 /dev/vg_raid/lv_data
mkfs.ext4 /dev/vg_raid/lv_backup
mkfs.ext4 /dev/vg_raid/lv_archive

# 6. Mount
mkdir -p /mnt/{data,backup,archive}
mount /dev/vg_raid/lv_data /mnt/data
mount /dev/vg_raid/lv_backup /mnt/backup
mount /dev/vg_raid/lv_archive /mnt/archive

# 7. Save configurations
mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# 8. In fstab (get UUID via blkid)
echo "/dev/vg_raid/lv_data    /mnt/data     ext4  defaults  0  2" >> /etc/fstab
echo "/dev/vg_raid/lv_backup  /mnt/backup   ext4  defaults  0  2" >> /etc/fstab
echo "/dev/vg_raid/lv_archive /mnt/archive  ext4  defaults  0  2" >> /etc/fstab
```

---

## 13. Important Files and Directories

### System Files
```bash
/etc/fstab                 # Filesystem auto-mounting
/etc/mtab                  # Current mounts
/etc/crypttab              # LUKS encrypted devices
/etc/mdadm/mdadm.conf      # RAID configuration
/etc/lvm/                  # LVM configuration
```

### Devices
```bash
/dev/                      # All devices
/dev/disk/by-uuid/         # Access by UUID
/dev/disk/by-label/        # Access by label
/dev/disk/by-id/           # Access by device ID
/dev/disk/by-path/         # Access by physical path
/dev/mapper/               # Device mapper (LVM, LUKS)
```

### Kernel Information
```bash
/proc/partitions           # Partitions known to kernel
/proc/mounts               # Mounted FS (kernel view)
/proc/mdstat               # RAID array status
/proc/sys/vm/swappiness    # Swap settings
```

### Sysfs
```bash
/sys/block/sda/            # Information about sda disk
/sys/block/sda/queue/      # I/O queue settings
/sys/block/sda/queue/scheduler  # I/O scheduler
```

---

## 14. Interview Questions

### Basic Level

**Q: What's the difference between /dev/sda and /dev/sda1?**  
A: `/dev/sda` is the entire physical disk, `/dev/sda1` is the first partition on that disk.

**Q: What are major and minor device numbers?**  
A: Major number identifies the driver type (e.g., 8 = SCSI/SATA), minor number identifies the specific device or partition.

**Q: Why use UUID instead of /dev/sdX?**  
A: Device paths can change on reboot (disk sda can become sdb), but UUID is unique and permanent for a specific partition.

**Q: What is mounting?**  
A: The process of connecting a filesystem to the Linux directory tree, making its contents accessible through a specific point (directory).

**Q: What's the difference between hard link and symbolic link?**  
A: Hard link is an additional name for the same inode (file), works only within one FS. Symbolic link is a separate file containing a path to another file, works across different FS and partitions.

### Intermediate Level

**Q: What happens when executing mkfs.ext4?**  
A: Creates filesystem structure: superblock with metadata, inode table for file indexing, group descriptors, journal for journaling, initializes data blocks.

**Q: Why use LVM?**  
A: For flexible disk space management: dynamic partition resizing, snapshots, pooling multiple disks, data migration without downtime.

**Q: What's the difference between RAID 1 and RAID 5?**  
A: RAID 1 (mirroring) - complete duplication on 2 disks, 50% usable capacity, high reliability. RAID 5 (striping + parity) - data and checksums distributed across 3+ disks, (N-1) usable capacity, survives 1 disk loss.

**Q: What to do if disk goes read-only?**  
A: It's a protective response to FS errors. Need to: 1) Unmount the partition, 2) Run fsck to check and fix, 3) Mount back. For root partition - boot from Live USB.

**Q: How to find processes using a partition?**  
A: `lsof +D /mount/point` or `fuser -m /mount/point`

### Advanced Level

**Q: How does journaling work in ext4?**  
A: Before writing data to disk, information about the transaction is first written to the journal. On crash, the system can restore FS integrity by replaying or rolling back incomplete operations from the journal.

**Q: Explain LVM architecture from bottom up.**  
A: 1) Physical Volumes (PV) - physical disks/partitions, 2) Volume Groups (VG) - pool of multiple PVs, 3) Logical Volumes (LV) - virtual partitions from VG, on which filesystems are created.

**Q: How do LVM snapshots work?**  
A: Copy-on-Write: when creating a snapshot, only metadata is saved. When a block changes in the original LV, the old content is copied to the snapshot. Snapshot stores only changes, saving space.

**Q: What is I/O scheduler and what types exist?**  
A: Kernel subsystem managing the order of I/O operations. Types: noop/none (for SSD), deadline/mq-deadline (for servers/databases), bfq (for desktops), cfq (deprecated).

**Q: Disk shows 100% usage in df, but there's plenty of space. What's the problem?**  
A: Possible causes: 1) Out of inodes (check `df -i`), 2) Deleted but open files (find via `lsof +L1`), 3) Reserved for root 5% (reduce via `tune2fs -m`).

**Q: How to migrate system to larger disk without reinstalling?**  
A: 1) Clone disk via `dd if=/dev/sda of=/dev/sdb`, 2) Expand partition table on new disk, 3) Expand partitions (LVM or resize2fs), 4) Reinstall GRUB bootloader, 5) Reboot from new disk.

---

## 15. Tips and Best Practices

### Data Safety

✅ **ALWAYS backup before:**
- Changing partition table
- Formatting
- Resizing partitions
- Migrating to another disk

✅ **Use UUID in fstab:**
```bash
# Bad:
/dev/sdb1  /data  ext4  defaults  0  2

# Good:
UUID=550e8400-...  /data  ext4  defaults  0  2
```

✅ **Unmount before checking:**
```bash
umount /dev/sda1
fsck /dev/sda1
```

✅ **Test fstab before rebooting:**
```bash
mount -fav  # check syntax
mount -a    # try mounting
```

### Performance

✅ **For SSD:**
- Use scheduler: noop/none
- Enable TRIM: `discard` in fstab or `fstrim.timer`
- Disable atime: `noatime,nodiratime`

✅ **For database servers:**
- Use XFS or ext4
- Scheduler: deadline/mq-deadline
- Disable atime
- Consider separate partition for database journal

✅ **ext4 optimization:**
```bash
tune2fs -o journal_data_writeback /dev/sda1  # faster, less safe
tune2fs -m 1 /dev/sda1                        # reduce reserve to 1%
```

### Monitoring

✅ **Regularly check:**
```bash
# SMART status
smartctl -H /dev/sda

# Disk usage
df -h

# RAID status (if applicable)
cat /proc/mdstat

# System logs
dmesg | grep -i error
journalctl -p err
```

✅ **Setup automatic monitoring:**
```bash
# SMART monitoring
systemctl enable smartd
systemctl start smartd

# Email notifications for RAID errors
echo "MAILADDR your@email.com" >> /etc/mdadm/mdadm.conf
```

### Data Organization

✅ **Recommended partition structure:**
```
/boot      - 1GB  (boot files)
/          - 20-50GB (system)
/home      - separate partition (user data)
/var       - separate partition for servers (logs, databases)
swap       - 2-8GB or by formula
/data      - remaining (user data)
```

✅ **For servers with LVM:**
```
PV: all available disks
VG: one large volume group
LV: separate volumes for /, /var, /home with room for expansion
```

---

## 16. Glossary of Terms

**Block Device** - device supporting random data access in blocks

**Character Device** - device with sequential access

**Major Number** - identifier of device driver type

**Minor Number** - identifier of specific device within driver

**MBR** - Master Boot Record, old disk partitioning scheme (max 2TB)

**GPT** - GUID Partition Table, modern partitioning scheme

**UUID** - Universally Unique Identifier, unique filesystem identifier

**PARTUUID** - unique partition identifier (GPT)

**Filesystem** - method of organizing and storing files on disk

**Inode** - index descriptor, structure with file metadata

**Superblock** - block with filesystem metadata

**Mount Point** - directory for connecting filesystem

**fstab** - file with automatic mounting configuration

**LVM** - Logical Volume Manager, logical volume management system

**PV** - Physical Volume, physical volume in LVM

**VG** - Volume Group, volume group in LVM

**LV** - Logical Volume, logical volume in LVM

**RAID** - Redundant Array of Independent Disks, disk array

**Striping** - distributing data across multiple disks (RAID 0)

**Mirroring** - data duplication (RAID 1)

**Parity** - checksum for data recovery (RAID 5/6)

**Journaling** - maintaining operation log for crash protection

**Swap** - swap file/partition for RAM expansion

**SMART** - Self-Monitoring, Analysis and Reporting Technology

**TRIM** - command for SSD optimization

**I/O Scheduler** - input/output operation scheduler

**Device Mapper** - kernel subsystem for block device virtualization

**LUKS** - Linux Unified Key Setup, disk encryption

---

## 17. Useful Links

### Documentation
- **Ubuntu Wiki:** https://wiki.ubuntu.com/
- **Arch Linux Wiki (excellent documentation):** https://wiki.archlinux.org/
- **Linux Man Pages:** https://linux.die.net/man/

### Tools
- **GParted (GUI for partitions):** https://gparted.org/
- **TestDisk (recovery):** https://www.cgsecurity.org/wiki/TestDisk
- **ddrescue:** https://www.gnu.org/software/ddrescue/

### Learning
- **Linux Journey:** https://linuxjourney.com/
- **The Linux Command Line (book):** http://linuxcommand.org/tlcl.php

---

## Conclusion

This guide covers the main aspects of disk management in Ubuntu/Linux - from basic concepts to advanced scenarios with LVM and RAID.

**Key Principles:**
1. Always backup before changes
2. Use UUID in fstab
3. Unmount before checking/modifying
4. Regularly monitor disk health
5. Test changes before rebooting

**For Further Study:**
- Practice in virtual machines
- Read man pages for commands (`man lvm`, `man mkfs.ext4`)
- Study advanced topics: ZFS, Btrfs, ceph

Good luck mastering Linux! 🐧

---

**Document Version:** 1.0  
**Last Updated:** October 2025  
**License:** Creative Commons BY-SA 4.0