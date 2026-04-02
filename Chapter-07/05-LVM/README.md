# Chapter 7 - Lesson 5: LVM - Logical Volume Manager

**Chapter 7 | Lesson 5 of 10**

## 🎯 এই Lesson-এ আমরা কী শিখবো?

- LVM কী এবং কেন দরকার?
- LVM এর তিনটি Layer (PV → VG → LV)
- Real-world analogy দিয়ে সহজে বোঝা
- Commands: `pvcreate`, `vgcreate`, `lvcreate`, `pvdisplay`, `vgdisplay`, `lvdisplay`
- LVM দিয়ে একটি পূর্ণ disk setup করা step-by-step


## LVM কী? কেন দরকার?

এর আগের lessons-এ আমরা শিখেছিলাম traditional partitioning - মানে `fdisk` দিয়ে disk কে ভাগ করা।

Traditional partitioning-এর সমস্যা কী?

> আপনি যদি একটা partition বানান 50GB দিয়ে, আর পরে দেখলেন আরো 30GB দরকার, আপনি সহজে বাড়াতে পারবেন না। Resize করা অনেক ঝামেলার, অনেক সময় data হারানোর ভয় থাকে।

**LVM (Logical Volume Manager)** এই সমস্যাটা সমাধান করে।

LVM হলো একটা **abstraction layer** - এটা physical disk-এর উপর বসে এবং আপনাকে দেয়:
- যেকোনো সময় storage বাড়ানো/কমানো
- একাধিক disk কে একসাথে জোড়া লাগানো
- Snapshot নেওয়া (backup এর জন্য দারুণ!)
- Production system চলতে চলতেই resize করা


ধরুন আপনি একজন **বালি ব্যবসায়ী**:

| Real World | LVM World |
|---|---|
| আপনার কাছে আছে ৩টা বালির ট্রাক | ৩টা Physical Disk (PV) |
| সব ট্রাকের বালি ঢেলে দিলে একটা বড় গুদামে | একটা Volume Group (VG) |
| সেই গুদাম থেকে যতটুকু দরকার বালি নিয়ে বস্তায় ভরলেন | Logical Volume (LV) তৈরি করলেন |
| বস্তার সাইজ পরে বাড়ানো বা কমানো যায় | LV resize করা যায় |

এটাই LVM এর মূল ধারণা।

## LVM এর তিনটি Layer

```
┌─────────────────────────────────────┐
│         Logical Volume (LV)         │  ← আপনি এটাই use করুন (mount করুন)
│    /dev/myvg/data  /dev/myvg/logs   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Volume Group (VG)           │  ← সব PV একসাথে জোড়া লাগানো pool
│              myvg                   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│       Physical Volume (PV)          │  ← Actual disk/partition
│  /dev/sdb    /dev/sdc    /dev/sdd   │
└─────────────────────────────────────┘
```

### Layer 1: Physical Volume (PV)
- Actual physical disk বা partition যেটাকে LVM-এর জন্য তৈরি করা হয়
- Example: `/dev/sdb`, `/dev/sdc`, `/dev/sdb1`

### Layer 2: Volume Group (VG)
- একটা বা একাধিক PV কে একসাথে একটা বড় pool এ রাখা হয়
- এটাকে আপনি একটা "virtual big disk" মনে করতে পারেন
- Example: `myvg` (100GB + 200GB = 300GB pool)

### Layer 3: Logical Volume (LV)
- Volume Group থেকে কেটে নেওয়া একটা portion
- এটাকেই আপনি format করেন, mount করেন, use করেন
- Example: `data` (100GB), `logs` (50GB)

## Step-by-Step: LVM Setup করা

আমরা এখন একটা complete LVM setup করবো। ধরুন আপনার কাছে দুটো নতুন disk আছে: `/dev/sdb` এবং `/dev/sdc`।

### Step 0: LVM Tools Install করা

```bash
# Ubuntu/Debian
sudo apt install lvm2 -y

# RHEL/CentOS
sudo yum install lvm2 -y
```

### Step 1: Physical Volume তৈরি করা - `pvcreate`

**কী করে:** একটা disk বা partition-কে LVM Physical Volume হিসেবে চিহ্নিত করে।

**Syntax:**
```bash
pvcreate <device>
```

**Example:**
```bash
sudo pvcreate /dev/sdb /dev/sdc
```

**Expected Output:**
```
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
```

> ⚠️ **সতর্কতা:** এই command দেওয়ার আগে নিশ্চিত হন disk-এ কোনো important data নেই।


### PV দেখুন - `pvdisplay` ও `pvs`

```bash
sudo pvdisplay
```

**Expected Output:**
```
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               20.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               abc123-...
```

**Short summary দেখতে:**
```bash
sudo pvs
```

**Output:**
```
  PV         VG   Fmt  Attr PSize  PFree
  /dev/sdb        lvm2 ---  20.00g 20.00g
  /dev/sdc        lvm2 ---  20.00g 20.00g
```

### Step 2: Volume Group তৈরি করুন - `vgcreate`

**কী করে:** একটা বা একাধিক PV কে একটা Volume Group-এ যোগ করে।

**Syntax:**
```bash
vgcreate <vg_name> <pv1> <pv2> ...
```

**Example:**
```bash
sudo vgcreate myvg /dev/sdb /dev/sdc
```

**Expected Output:**
```
  Volume group "myvg" successfully created
```

এখন আপনার কাছে **40GB** (20+20) এর একটা virtual pool আছে নাম `myvg`।


### VG দেখুন - `vgdisplay` ও `vgs`

```bash
sudo vgdisplay myvg
```

**Expected Output:**
```
  --- Volume group ---
  VG Name               myvg
  System ID             
  Format                lvm2
  VG Size               39.99 GiB
  PE Size               4.00 MiB
  Total PE              10238
  Alloc PE / Size       0 / 0   
  Free  PE / Size       10238 / 39.99 GiB
  VG UUID               xyz789-...
```

**Important জিনিস:**
- **VG Size:** মোট available space
- **Free PE:** এখনো কতটুকু ফাঁকা আছে
- **PE Size:** Physical Extent size (LVM এর smallest unit - default 4MB)

**Short summary:**
```bash
sudo vgs
```

```
  VG   #PV #LV #SN Attr   VSize   VFree
  myvg   2   0   0 wz--n- 39.99g 39.99g
```

### Step 3: Logical Volume তৈরি করুন - `lvcreate`

**কী করে:** Volume Group থেকে একটা Logical Volume (LV) কেটে নেয়।

**Syntax:**
```bash
lvcreate -L <size> -n <lv_name> <vg_name>
```

**Options:**

| Option | মানে |
|---|---|
| `-L 10G` | Fixed size (10 Gigabytes) |
| `-l 100%FREE` | বাকি সব free space নেন |
| `-l 50%VG` | VG এর 50% নেন |
| `-n data` | LV এর নাম হবে "data" |

**Example - দুটো LV বানান:**
```bash
# 15GB এর "data" volume
sudo lvcreate -L 15G -n data myvg

# 10GB এর "logs" volume
sudo lvcreate -L 10G -n logs myvg
```

**Expected Output:**
```
  Logical volume "data" created.
  Logical volume "logs" created.
```

আপনার LV গুলো এখন এখানে পাবেন:
- `/dev/myvg/data` অথবা `/dev/mapper/myvg-data`
- `/dev/myvg/logs` অথবা `/dev/mapper/myvg-logs`

### LV দেখুন - `lvdisplay` ও `lvs`

```bash
sudo lvdisplay
```

**Expected Output:**
```
  --- Logical volume ---
  LV Path                /dev/myvg/data
  LV Name                data
  VG Name                myvg
  LV Size                15.00 GiB
  LV Status              available
  Block device           253:0
```

**Short summary:**
```bash
sudo lvs
```

```
  LV   VG   Attr       LSize  Pool Origin
  data myvg -wi-a----- 15.00g
  logs myvg -wi-a----- 10.00g
```

### Step 4: LV Format ও Mount করুন

LV বানানোর পর এটাকে format করতে হবে এবং mount করতে হবে - ঠিক regular partition এর মতোই।

```bash
# ext4 দিয়ে format করুন
sudo mkfs.ext4 /dev/myvg/data
sudo mkfs.ext4 /dev/myvg/logs
```

**Output:**
```
mke2fs 1.46.5
Creating filesystem with 3932160 4k blocks...
Writing superblocks and filesystem accounting information: done
```

```bash
# Mount point বানান
sudo mkdir -p /mnt/data
sudo mkdir -p /mnt/logs

# Mount করুন
sudo mount /dev/myvg/data /mnt/data
sudo mount /dev/myvg/logs /mnt/logs
```

**Verify করুন:**
```bash
df -h | grep myvg
```

**Output:**
```
/dev/mapper/myvg-data   15G   24K   14G   1% /mnt/data
/dev/mapper/myvg-logs   10G   24K    9G   1% /mnt/logs
```

### Step 5: Permanent Mount - `/etc/fstab`

Reboot এর পরেও mount থাকার জন্য `/etc/fstab` এ যোগ করুন:

```bash
sudo nano /etc/fstab
```

নিচের lines যোগ করুন:
```
/dev/myvg/data    /mnt/data    ext4    defaults    0 0
/dev/myvg/logs    /mnt/logs    ext4    defaults    0 0
```

Test করুন:
```bash
sudo mount -a   # কোনো error না আসলে সব ঠিক আছে
```

## পুরো LVM Architecture - এক নজরে

```
Physical Disks          Physical Volumes      Volume Group       Logical Volumes
──────────────          ────────────────      ────────────       ───────────────

/dev/sdb (20GB)  →→→   PV: /dev/sdb    ─┐
                                          ├──→  VG: myvg  ──→  LV: data (15GB) → /mnt/data
/dev/sdc (20GB)  →→→   PV: /dev/sdc    ─┘    (39.99GB)   ──→  LV: logs (10GB) → /mnt/logs
                                                            ──→  Free: ~15GB
```

## সব Important Commands এক জায়গায়

| Command | কাজ |
|---|---|
| `pvcreate /dev/sdb` | disk কে PV বানান |
| `pvdisplay` | PV এর বিস্তারিত দেখুন |
| `pvs` | PV এর short summary |
| `vgcreate myvg /dev/sdb` | VG বানান |
| `vgdisplay myvg` | VG এর বিস্তারিত দেখুন |
| `vgs` | VG এর short summary |
| `lvcreate -L 10G -n data myvg` | LV বানান |
| `lvdisplay` | LV এর বিস্তারিত দেখুন |
| `lvs` | LV এর short summary |
| `mkfs.ext4 /dev/myvg/data` | LV format করুন |
| `mount /dev/myvg/data /mnt/data` | LV mount করুন |


## DevOps-এ LVM কোথায় কাজে লাগে?

1. **Database Servers:** MySQL/PostgreSQL এর data directory `/var/lib/mysql` কে LVM-এ রাখলে যেকোনো সময় disk বাড়ানো যায়
2. **Log Management:** `/var/log` কে আলাদা LV-তে রাখলে log দিয়ে root filesystem full হয় না
3. **Snapshot Backup:** Production database-এর snapshot নিয়ে backup করা যায়, downtime ছাড়াই
4. **Cloud VMs:** AWS/GCP-তে নতুন disk attach করে LVM pool-এ যোগ করা খুব সহজ

## 📝 Quick Summary

- LVM হলো traditional partitioning এর চেয়ে flexible storage management system
- তিনটি layer: PV (Physical Volume) → VG (Volume Group) → LV (Logical Volume)
- `pvcreate` দিয়ে disk কে LVM-ready করে
- `vgcreate` দিয়ে PV গুলোকে একটা pool এ যোগ করে
- `lvcreate` দিয়ে সেই pool থেকে LV কেটে নাও
- LV-কে format করে → mount করে → `/etc/fstab`-এ যোগ করে
- `pvs`, `vgs`, `lvs` দিয়ে quick status দেখা যায়


## 🏋️ Practice Tasks

1. `lsblk` command রান করুন এবং দেখুন আপনার system-এ কোনো disk আছে কিনা। তারপর `pvs`, `vgs`, `lvs` চালান - দেখুন কোনো existing LVM আছে কিনা।

2. একটা test করুন (যদি VM থাকে): একটা নতুন virtual disk যোগ করুন, সেটা দিয়ে একটা PV বানান, তারপর VG, তারপর LV। Format করে mount করুন।

3. `vgdisplay` output থেকে বের করুন: আপনার VG-তে কতটুকু `Free PE` আছে? PE Size কত? এই দুটো গুণ করলে কত GB free আছে বের হবে?

---

## ⏭️ What's Next?

**Chapter 7 - Lesson 6: LVM - Extending & Reducing Volumes**
(`lvextend`, `resize2fs`, `lvreduce` - production system চালু থাকতেই storage বাড়ান!) *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../04-Formatting-And-Mounting">← Formatting &amp; Mounting</a>
    </td>
    <td align="right">
      <a href="../06-LVM-Extending-Reducing-Volumes">LVM Extending Reducing Volumes →</a>
    </td>
  </tr>
</table>