# Chapter 7 - Lesson 3: File System Types

**Chapter 7 | Lesson 3 of 10**


## 🎯 এই Lesson-এ আমরা কী শিখবো?

আজকে আমরা Linux-এর সবচেয়ে গুরুত্বপূর্ণ File System Types নিয়ে কথা বলবো। এর আগের লেসনে আমরা শিখেছি disk partition কিভাবে করতে হয়। এখন প্রশ্ন হলো, সেই partition-এ কোন ধরনের file system ব্যবহার করবো?

তিনটি major file system নিয়ে আজকে বিস্তারিত জানবো:
- **ext4** - সবচেয়ে পুরনো এবং সবচেয়ে popular
- **xfs** - high performance, enterprise-grade
- **btrfs** - modern, feature-rich, next-gen


## File System কী?

মনে করেন আপনার একটা বড় warehouse আছে। সেই warehouse-এ হাজারো জিনিস রাখতে হবে।

এখন আপনি কীভাবে জিনিস রাখবেন?
- প্রতিটা জিনিসের একটা নির্দিষ্ট জায়গা লাগবে
- কোথায় কী আছে সেটার একটা `catalog/index` লাগবে
- নতুন জিনিস আসলে কোথায় রাখবে সেটার নিয়ম লাগবে
- পুরনো জিনিস সরালে সেই জায়গা reuse করার নিয়ম লাগবে

এই পুরো `warehouse management system`-টাই হলো File System।

Disk = warehouse (খালি জায়গা)
File System = সেই warehouse কীভাবে manage হবে তার নিয়মকানুন

## File System-এর কাজ কী?

| কাজ | ব্যাখ্যা |
|-----|---------|
| File store করা | Data কোথায় physically থাকবে |
| Metadata রাখা | File-এর নাম, size, permission, timestamp |
| Directory structure | Folder hierarchy maintain করা |
| Free space track করা | কতটুকু জায়গা আছে/নেই |
| Journaling | System crash হলে data recover করা |

## ext4 - The Workhorse

### ext4 কী?

`ext4` মানে হলো `Fourth Extended File System`। এটা Linux-এর সবচেয়ে `traditional এবং widely used` file system।

Timeline:
```
ext → ext2 → ext3 → ext4 (current)
```

ext4 হলো Linux-এর default choice - Ubuntu, Debian, CentOS সবাই default-এ ext4 ব্যবহার করে।


### ext4-এর Key Features

#### 1️ Journaling
Journaling মানে হলো একটা diary রাখা।

আপনি যখন কোনো important কাজ করেন, আগে একটা notepad-এ লিখে রাখো "এই কাজটা করবো"। কাজ শেষে লিখেন "কাজ হয়েছে"। মাঝপথে light চলে গেলেও আপনি notepad দেখে বুঝতে পারবেন আপনি কোথায় ছিলেন।

ext4-এ system crash হলে এই journal দেখে automatically recover করে।

#### 2️ Extents
পুরনো ext2/ext3-তে file-এর প্রতিটা block আলাদা আলাদা track করতো। ext4-এ Extent concept আসলো - একটা continuous block range কে একটাই entry দিয়ে track করে।

```
Old way:  [block 1] [block 2] [block 3] [block 4] [block 5]  ← 5টা entry
ext4 way: [block 1 to 5]  ← মাত্র 1টা entry
```

এতে performance অনেক ভালো হয়।

#### 3️ Delayed Allocation
Data লেখার আগে ext4 একটু wait করে। অনেক ছোট ছোট write একসাথে জমিয়ে একবারে লেখে। এতে fragmentation কমে এবং performance বাড়ে।

#### 4️ Large File & Volume Support

| Feature | Limit |
|---------|-------|
| Max file size | **16 TB** |
| Max volume size | **1 EB** (Exabyte) |
| Max files | 4 billion |


### ext4-এর Pros & Cons

|  **Pros** | **Cons** |
|---------|---------|
| Very stable & mature | Older architecture |
| Great compatibility | No built-in snapshot |
| Fast fsck (repair) | No checksums for data |
| Default on most distros | Scaling limits at huge data |
| Well-documented | |


### কখন ext4 ব্যবহার করবো?

- General purpose Linux server
- Boot partition (`/boot`)
- Home server, VM disk
- যখন simplicity এবং stability দরকার
- বেশিরভাগ DevOps use case-এ


### ext4 দেখার Commands

```bash
# কোন partition-এ কোন file system আছে দেখুন
lsblk -f
```

**Output:**
```
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 ext4         a1b2c3d4-...                         /
└─sda2 swap         e5f6g7h8-...                         [SWAP]
```

```bash
# ext4 filesystem-এর info দেখুন
tune2fs -l /dev/sda1
```

**Output (partial):**
```
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr dir_index filetype
Filesystem state:         clean
Last mount time:          Sun Mar 15 10:00:00 2026
Last write time:          Sun Mar 15 10:05:00 2026
Block count:              10485760
Block size:               4096
Inode count:              2621440
```

```bash
# ext4 filesystem check করুন (unmounted state-এ)
fsck.ext4 /dev/sdb1
```

## xfs - The Performance Beast

### xfs কী?

XFS তৈরি করেছিল Silicon Graphics (SGI) 1993 সালে। পরে Linux-এ port করা হয়। এখন RHEL/CentOS/Rocky Linux-এর default file system।

xfs কে বলা হয় high-performance journaling file system।


### xfs-এর Key Features

#### 1️ Parallel I/O
xfs একসাথে multiple threads থেকে write/read handle করতে পারে অনেক efficiently।

ext4 হলো একটা single lane road। xfs হলো multi-lane highway - অনেক গাড়ি একসাথে দ্রুত যেতে পারে।

#### 2️ Dynamic Inode Allocation
ext4-এ format করার সময় inode count fix করতে হয়। পরে বাড়ানো যায় না।

xfs-এ inode dynamically allocate হয় - যতটা দরকার ততটা নেয়।

> Inode কী? প্রতিটা file-এর একটা unique ID এবং metadata (owner, permission, size, timestamp) থাকে। এটাই inode।

#### 3️ Online Defragmentation & Growth
```bash
# xfs চলতে চলতেই defrag করা যায়
xfs_fsr /dev/sdb1

# xfs চলতে চলতেই বড় করা যায়
xfs_growfs /mountpoint
```

#### 4️ xfs_repair - Powerful Recovery

```bash
xfs_repair /dev/sdb1
```

#### 5️ Large File & Volume Support

| Feature | Limit |
|---------|-------|
| Max file size | **8 EB** |
| Max volume size | **8 EB** |
| Max files | Practically unlimited |

### xfs-এর Pros & Cons

|  **Pros** | **Cons** |
|---------|---------|
| Excellent performance | Cannot shrink volume |
| Great for large files | Recovery কিছুটা complex |
| Parallel I/O | Newer users unfamiliar |
| RHEL/CentOS default | Slightly more overhead on small files |
| Online grow করা যায় | |
| Huge scalability | |


### কখন xfs ব্যবহার করবো?

- **Large file storage** - video, database, backup
- **High throughput servers** - media servers, NFS servers
- **RHEL/CentOS systems** - এগুলো already xfs use করে
- **Kubernetes Persistent Volumes**
- **Big Data workloads**


### xfs Commands

```bash
# xfs filesystem info দেখুন
xfs_info /dev/sdb1
```

**Output:**
```
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

```bash
# xfs filesystem check করুন
xfs_repair -n /dev/sdb1    # -n মানে dry-run, কিছু change করবে না

# xfs disk usage দেখুন
xfs_quota -c 'df -h' /mountpoint
```


## btrfs - The Modern File System

### btrfs কী?

**Btrfs** (B-tree File System, উচ্চারণ: "Butter FS" বা "Better FS") - এটা Oracle তৈরি করেছে 2007 সালে। এটা **next-generation** Linux file system।

btrfs-এর মূল লক্ষ্য ছিল ext4 এবং xfs-এর limitations দূর করে **ZFS-এর মতো** advanced features আনা Linux-এ।

> **ZFS** হলো Sun Microsystems-এর তৈরি enterprise file system - অনেক powerful কিন্তু licensing issue-এর কারণে Linux kernel-এ নেই।

### btrfs-এর Key Features

#### 1️ Copy-on-Write (CoW)

এটা btrfs-এর সবচেয়ে important feature।

আপনি একটা document edit করতে চান। CoW system বলে - original document touch না করতে। একটা copy বানান, সেটা edit করেন। শেষ হলে নতুনটা রাখেন, আর পুরনোটা delete করে ফেলেন।

এতে কী হয়?
- Data corruption এর chance অনেক কমে
- Snapshot নেওয়া হয় instantaneous

#### 2️ Snapshots (Built-in!)

```bash
# একটা subvolume-এর snapshot নিন মাত্র এক সেকেন্ডে!
btrfs subvolume snapshot /data /data_snapshot
```

এটা ext4 বা xfs-এ possible না। এজন্য LVM বা ZFS লাগে। btrfs-এ built-in।

**DevOps use case:** Deploy করার আগে snapshot নিন। কিছু ভুল হলে instantly rollback করা যাবে।

#### 3️ Checksums (Data Integrity)

btrfs প্রতিটা data block-এর checksum রাখে। কোনো bit corrupt হলে automatically detect করে।

```
Data written → Checksum stored
Data read → Checksum verify করুন
Mismatch → Error report / auto-repair
```

ext4 এবং xfs এটা করে না!

#### 4️ RAID Built-in

```bash
# btrfs দিয়েই RAID 1 (mirror) তৈরি করা যায়
mkfs.btrfs -d raid1 /dev/sdb /dev/sdc
```

আলাদা `mdadm` লাগে না।

#### 5️ Subvolumes

btrfs-এ subvolume concept আছে - এটা অনেকটা partition-এর মতো, কিন্তু একই disk-এ flexible boundary দিয়ে।

```bash
# subvolume তৈরি করুন
btrfs subvolume create /data/web
btrfs subvolume create /data/db
```

#### 6️ Online Scrub (Data Verification)

```bash
# চলতে চলতেই সব data verify করুন
btrfs scrub start /mountpoint
btrfs scrub status /mountpoint
```

#### 7️ Transparent Compression

```bash
# Mount করার সময় auto-compression চালু করুন
mount -o compress=zstd /dev/sdb1 /data
```

Data automatically compress হয়ে store হয় - আপনি টের পাবেন না, কিন্তু disk space বাঁচবে।


### btrfs-এর Pros & Cons

|  **Pros** | **Cons** |
|---------|---------|
| Built-in snapshots | Still maturing (less stable than ext4) |
| Data checksums | RAID 5/6 implementation buggy |
| CoW architecture | More complex to manage |
| Subvolumes | Slightly slower than xfs for raw I/O |
| Built-in compression | Less documentation |
| Online scrub | fsck (repair) কম reliable |
| Fedora Linux default | |


### কখন btrfs ব্যবহার করবো?

- Fedora/openSUSE systems - এগুলো btrfs default
- Snapshot দরকার - rollback capability
- Data integrity critical - financial, medical data
- Home/desktop systems
- Testing/lab environments

> ⚠️ **Warning:** Production database server-এ btrfs সাবধানে ব্যবহার করা উচিৎ কারন অনেক DBA এখনো ext4 বা xfs prefer করেন।


### btrfs Commands

```bash
# btrfs filesystem info দেখুন
btrfs filesystem show /dev/sdb1
```

**Output:**
```
Label: none  uuid: 1234abcd-...
        Total devices 1 FS bytes used 1.23GiB
        devid    1 size 20.00GiB used 3.02GiB path /dev/sdb1
```

```bash
# Subvolume list দেখুন
btrfs subvolume list /mountpoint

# Snapshot নিন
btrfs subvolume snapshot /data /data_snap_$(date +%Y%m%d)

# Scrub চালান
btrfs scrub start /mountpoint

# Usage দেখুন
btrfs filesystem usage /mountpoint
```


## তিনটা File System - পাশাপাশি তুলনা

| Feature | ext4 | xfs | btrfs |
|---------|------|-----|-------|
| **Stability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Max Volume** | 1 EB | 8 EB | 16 EB |
| **Max File** | 16 TB | 8 EB | 16 EB |
| **Journaling** | ✅ | ✅ | ❌ (CoW) |
| **Snapshots** | ❌ | ❌ | ✅ |
| **Checksums** | ❌ | ❌ | ✅ |
| **Online Grow** | ✅ | ✅ | ✅ |
| **Online Shrink** | ✅ | ❌ | ✅ |
| **Built-in RAID** | ❌ | ❌ | ✅ |
| **Compression** | ❌ | ❌ | ✅ |
| **Default on** | Ubuntu/Debian | RHEL/CentOS | Fedora/openSUSE |
| **Best for** | General use | High performance | Snapshots/integrity |


## কীভাবে দেখবো কোন File System আছে?

### Method 1: `lsblk -f`
```bash
lsblk -f
```

**Output:**
```
NAME   FSTYPE   LABEL    UUID                                 MOUNTPOINTS
sda
├─sda1 vfat     EFI      1234-ABCD                            /boot/efi
├─sda2 ext4              abcd1234-...                         /boot
└─sda3 ext4              efgh5678-...                         /
sdb
└─sdb1 xfs               ijkl9012-...                         /data
```

### Method 2: `df -T`
```bash
df -T
```

**Output:**
```
Filesystem     Type     1K-blocks    Used Available Use% Mounted on
/dev/sda3      ext4      20511312 4823104  14634160  25% /
/dev/sda2      ext4        999320  210836    719692  23% /boot
/dev/sdb1      xfs       20511312 1024000  19487312   5% /data
```

### Method 3: `blkid`
```bash
blkid /dev/sda1
```

**Output:**
```
/dev/sda1: UUID="abcd1234-5678-..." TYPE="ext4" PARTUUID="..."
```

### Method 4: `/proc/filesystems`
```bash
cat /proc/filesystems
```

**Output:**
```
nodev   sysfs
nodev   tmpfs
nodev   bdev
        ext3
        ext4
        xfs
        btrfs
```

এটা দেখায় currently ``loaded/supported`` file system types।


## Real-World DevOps Scenario

### নতুন Server Setup করছেন

```
/dev/sda  →  OS disk
/dev/sdb  →  Application data
/dev/sdc  →  Database
/dev/sdd  →  Backup storage
```

**কোথায় কোন file system দেবো?**

```
/dev/sda → ext4   (OS, boot - stability দরকার)
/dev/sdb → xfs    (Application - high throughput)
/dev/sdc → xfs    (Database - large files, performance)
/dev/sdd → btrfs  (Backup - snapshot + compression)
```

এই ধরনের decision নেওয়াটাই হলো real DevOps/SysAdmin কাজ।


## 📝 Quick Summary

- **File System** হলো disk-এ data কীভাবে store ও manage হবে তার নিয়ম
- **ext4:** সবচেয়ে stable, default, general purpose - বেশিরভাগ ক্ষেত্রে এটাই যথেষ্ট
- **xfs:** High performance, large files, parallel I/O - RHEL/CentOS এর default; database ও media server-এর জন্য
- **btrfs:** Modern, snapshot + checksums + compression built-in - Fedora default; এখনো maturing
- `lsblk -f`, `df -T`, `blkid` দিয়ে file system type দেখা যায়
- **CoW (Copy-on-Write)** = data modify হলে original রেখে copy তৈরি করে - btrfs এটা করে
- **Journaling** = system crash recovery-র জন্য diary রাখা - ext4 ও xfs করে


## 🏋️ Practice Tasks

**Task 1:**
আপনার system-এ কোন কোন partition-এ কোন file system আছে দেখুন:
```bash
lsblk -f
df -T
blkid
```
Output note করুন।

**Task 2:**
ext4 filesystem-এর details দেখুন:
```bash
sudo tune2fs -l /dev/sda1   # আপনার root partition অনুযায়ী
```
`Filesystem state`, `Block size`, `Inode count` - এই তিনটা value খুঁজে বের করুন।

**Task 3:**
`/proc/filesystems` দেখুন আপনার system-এ কতগুলো file system support করে এবং কোনগুলো?
```bash
cat /proc/filesystems
```

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 4: Formatting & Mounting**
`mkfs` দিয়ে partition-এ file system তৈরি করা, `mount` ও `umount` দিয়ে disk attach/detach করা, এবং `/etc/fstab` ফাইলে এন্ট্রি দেয়া যাতে system reboot-এ mount সেটআপ ঠিক থাকে।

আজকের lesson কেমন লাগলো? কোনো concept নিয়ে আরো জানতে চাইলে জিজ্ঞেস করুন! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../02-Disk-Partitioning">← Disk Partitioning</a>
    </td>
    <td align="right">
      <a href="../04-Formatting-And-Mounting">Formatting &amp; Mounting →</a>
    </td>
  </tr>
</table>