# Chapter 7 - Lesson 10: Disk Encryption (LUKS, cryptsetup)

**Chapter 7 | Lesson 10 of 10**

আজকের lesson-এ আমরা শিখবো Linux-এ disk encryption কীভাবে কাজ করে। এটা একটি অত্যন্ত গুরুত্বপূর্ণ topic, বিশেষত যখন sensitive data protect করার প্রয়োজন হয়।


## Disk Encryption কী এবং কেন দরকার?

একটু চিন্তা করেন, আপনার কাছে একটা external hard drive আছে, যেখানে company-র confidential data আছে। কেউ যদি সেই drive চুরি করে নেয়, তাহলে সে কি সহজেই data পড়তে পারবে?

**Encryption ছাড়া:** হ্যাঁ, সে অন্য machine-এ drive mount করে সব data পড়তে পারবে।

**Encryption সহ:** না! সে শুধু দেখবে random garbage data - password ছাড়া কিছুই বুঝতে পারবে না।

```
Without Encryption:
Disk → Mount → Read Data (Anyone can read!)

With LUKS Encryption:
Disk → Unlock with Key/Password → Decrypt → Mount → Read Data
         ↑
    Without this step = Garbage data
```

## LUKS কী? (Linux Unified Key Setup)

**LUKS** হলো Linux-এর standard disk encryption format। এটা একটা specification যা define করে কীভাবে encrypted disk-এর header সংরক্ষিত থাকবে।


> LUKS হলো একটা bank vault-এর lock system এর মতো।
> - Vault-এর ভেতরে আছে আপনার data
> - Lock খুলতে হলে correct key দরকার
> - LUKS সেই lock-এর পুরো mechanism manage করে

### LUKS-এর বৈশিষ্ট্য:
| Feature | বিবরণ |
|---|---|
| Multiple Keys | একই disk-এ ৮টা পর্যন্ত আলাদা password/key রাখা যায় |
| Header | Disk-এর শুরুতে metadata সংরক্ষিত থাকে |
| Algorithm | AES-256 (industry standard) ব্যবহার করে |
| Key Slots | ৮টা key slot থাকে (LUKS1), ৩২টা (LUKS2) |


## cryptsetup কী?

`cryptsetup` হলো সেই tool যা দিয়ে আপনি LUKS encryption practically implement করতে পারেন।

```
LUKS = Standard/Specification (কীভাবে হবে তার নিয়ম)
cryptsetup = Tool (যেটা দিয়ে কাজ করবেন)
```


## Installation

```bash
# Ubuntu/Debian
sudo apt install cryptsetup

# RHEL/CentOS/Rocky Linux
sudo dnf install cryptsetup

# Version check
cryptsetup --version
```

**Expected Output:**
```
cryptsetup 2.4.3
```


## LUKS Encryption-এর পুরো Flow

```
Step 1: একটা disk/partition নাও (e.g., /dev/sdb)
    ↓
Step 2: LUKS দিয়ে encrypt করো (luksFormat)
    ↓
Step 3: Encrypted disk unlock/open করো (luksOpen)
    ↓
Step 4: Unlocked device-এ file system তৈরি করো (mkfs)
    ↓
Step 5: Mount করো এবং use করো
    ↓
Step 6: কাজ শেষে close করো (luksClose)
```


## Step-by-Step: একটা Disk Encrypt করা

### Step 1: Disk/Partition চিহ্নিত করুন

```bash
# Available disks দেখুন
lsblk

# অথবা
fdisk -l
```

**Expected Output:**
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   50G  0 disk
├─sda1   8:1    0   49G  0 part /
└─sda2   8:2    0    1G  0 part [SWAP]
sdb      8:16   0   10G  0 disk    ← এটা আমরা encrypt করবো
```

> ⚠️ **সতর্কতা:** `/dev/sdb` তে যদি কোনো data থাকে, encryption করলে সব মুছে যাবে। Lab-এ practice করার সময় একটা empty disk বা loop device ব্যবহার করুন।

### Lab Practice: Loop Device দিয়ে Safe Practice

Real disk না থাকলে loop device ব্যবহার করুন - এটা একটা virtual disk যা একটা file থেকে তৈরি হয়।

```bash
# একটা 100MB file তৈরি করুন (virtual disk হিসেবে)
dd if=/dev/zero of=/tmp/myencrypteddisk.img bs=1M count=100

# Loop device হিসেবে attach করুন
sudo losetup /dev/loop10 /tmp/myencrypteddisk.img

# Verify করুন
lsblk | grep loop10
```

**Output:**
```
loop10  7:10   0  100M  0 loop
```

এখন `/dev/loop10` কে আমরা একটা real disk-এর মতো treat করতে পারবো।


### Step 2: LUKS দিয়ে Encrypt করুন (luksFormat)

```bash
sudo cryptsetup luksFormat /dev/loop10
```

**এই command কী করে:**
- `/dev/loop10` কে LUKS encrypted container হিসেবে format করে
- একটা password চাইবে - এটাই হবে encryption key

**Interactive Output:**
```
WARNING!
========
This will overwrite data on /dev/loop10 irrevocably.

Are you sure? (Type uppercase yes): YES     ← YES টাইপ করতে হবে
Enter passphrase for /dev/loop10: ****      ← password দিতে হবে
Verify passphrase: ****                     ← এবং ঐ একই password আবার দিতে হবে
```

> YES uppercase-এ টাইপ করতে হবে কারন এটা intentional safety measure।


### Step 3: LUKS Header দেখুন

```bash
sudo cryptsetup luksDump /dev/loop10
```

**Output:**
```
LUKS header information
Version:        2
Epoch:          3
Metadata area:  16384 [bytes]
Keyslots area:  16744448 [bytes]
UUID:           a1b2c3d4-e5f6-7890-abcd-ef1234567890
Label:          (no label)
Subsystem:      (no subsystem)
Flags:          (no flags)

Data segments:
  0: crypt
        offset: 16777216 [bytes]
        length: (whole device)
        cipher: aes-xts-plain64
        sector: 512 [bytes]

Keyslots:
  0: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        ...
```

**গুরুত্বপূর্ণ তথ্য:**
| Field | মানে |
|---|---|
| Version | LUKS2 ব্যবহার হচ্ছে |
| UUID | Disk-এর unique identifier |
| Cipher | AES encryption algorithm |
| Keyslots | কতটা password slot আছে |


### Step 4: Encrypted Disk Open/Unlock করুন

```bash
sudo cryptsetup luksOpen /dev/loop10 myencrypteddisk
```

**Syntax breakdown:**
```
cryptsetup luksOpen  [encrypted_device]  [mapper_name]
                     /dev/loop10         myencrypteddisk
```

- `luksOpen` → encrypt করা device unlock করো
- `/dev/loop10` → যে device unlock করবে
- `myencrypteddisk` → unlock হওয়ার পর কোন নামে পাওয়া যাবে

**Password চাইবে:**
```
Enter passphrase for /dev/loop10: ****
```

**Verify করুন:**
```bash
ls /dev/mapper/
```

**Output:**
```
control  myencrypteddisk
```

এখন `/dev/mapper/myencrypteddisk` একটা decrypted virtual device - এতে file system তৈরি করা যাবে।


### Step 5: File System তৈরি করুন

```bash
# ext4 file system তৈরি করুন
sudo mkfs.ext4 /dev/mapper/myencrypteddisk
```

**Output:**
```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 71680 1k blocks and 17920 inodes
...
Writing superblocks and filesystem accounting information: done
```

---

### Step 6: Mount করুন

```bash
# Mount point তৈরি করুন
sudo mkdir /mnt/secure

# Mount করুন
sudo mount /dev/mapper/myencrypteddisk /mnt/secure

# Verify করুন
df -h | grep secure
```

**Output:**
```
/dev/mapper/myencrypteddisk   93M  1.6M   85M   2% /mnt/secure
```

এখন `/mnt/secure` এ যা রাখবে সব encrypted থাকবে!


### Step 7: Data রাখুন এবং দেখুন

```bash
# কিছু file তৈরি করুন
sudo bash -c "echo 'Top Secret Data' > /mnt/secure/secret.txt"
sudo bash -c "echo 'Server Password: abc123' > /mnt/secure/passwords.txt"

# দেখুন
ls -la /mnt/secure/
cat /mnt/secure/secret.txt
```

**Output:**
```
total 24
drwxr-xr-x 3 root root  1024 Mar 15 10:30 .
drwxr-xr-x 8 root root  4096 Mar 15 10:25 ..
drwx------ 2 root root 12288 Mar 15 10:28 lost+found
-rw-r--r-- 1 root root    16 Mar 15 10:30 secret.txt
-rw-r--r-- 1 root root    23 Mar 15 10:30 passwords.txt

Top Secret Data
```

### Step 8: Unmount এবং Lock করুন

```bash
# Unmount করুন
sudo umount /mnt/secure

# Encrypted device close/lock করুন
sudo cryptsetup luksClose myencrypteddisk

# Verify করুন তাহলে আর দেখা যাবে না
ls /dev/mapper/
```

**Output:**
```
control
```

`myencrypteddisk` আর নেই - এখন disk পুরোপুরি locked!


### Encryption প্রমাণ করুন

```bash
# সরাসরি raw device থেকে পড়ার চেষ্টা করুন
sudo strings /dev/loop10 | head -20
```

**Output:**
```
SKUL    ← LUKS header (উল্টো লেখা)
sha256
aes-xts-plain64
@}random garbage@#$%^&
```

সব garbage data! Password ছাড়া কিছুই বোঝা যাচ্ছে না।


## Multiple Keys/Passwords যোগ করুন

LUKS-এ ৮টা (LUKS1) বা ৩২টা (LUKS2) পর্যন্ত আলাদা password রাখা যায়।

```bash
# নতুন password যোগ করুন
sudo cryptsetup luksAddKey /dev/loop10
```

**Output:**
```
Enter any existing passphrase:    ← পুরনো password দিন
Enter new passphrase for key slot: ← নতুন password দিন
Verify passphrase:
```

**Use Case (DevOps):**
- Admin-1 এর আলাদা password
- Admin-2 এর আলাদা password
- Backup key (emergency জন্য)

### Key Slots দেখুন

```bash
sudo cryptsetup luksDump /dev/loop10 | grep -A5 "Keyslots"
```

## Password Remove করুন

```bash
# একটা নির্দিষ্ট password remove করুন
sudo cryptsetup luksRemoveKey /dev/loop10
```

```
Enter passphrase to be deleted: ****    ← যে password মুছবেন সেটা দিন
```

> ⚠️ **সতর্কতা:** শেষ password মুছে ফেললে disk চিরতরে inaccessible হয়ে যাবে!

## Key File ব্যবহার করা (Password-এর বদলে)

DevOps-এ অনেক সময় manual password দেওয়া সম্ভব হয় না (automated boot, scripts)। তখন key file ব্যবহার করা হয়।

```bash
# একটা random key file তৈরি করুন
sudo dd if=/dev/urandom of=/root/myluks.key bs=4096 count=1

# Permission tight করুন
sudo chmod 400 /root/myluks.key

# Key file কে LUKS-এ যোগ করুন
sudo cryptsetup luksAddKey /dev/loop10 /root/myluks.key
```

**এখন key file দিয়ে unlock করুন:**
```bash
sudo cryptsetup luksOpen /dev/loop10 myencrypteddisk --key-file /root/myluks.key
```

**Password চাইবে না!** Script বা automation-এ এটা অনেক কাজের।


## Boot-এ Auto Mount করা (/etc/crypttab)

সার্ভার reboot হলে encrypted disk আবার manually unlock করতে হয়। এটা automate করতে `/etc/crypttab` ব্যবহার হয়।

```bash
# UUID বের করুন
sudo cryptsetup luksDump /dev/sdb | grep UUID
# অথবা
sudo blkid /dev/sdb
```

**Output:**
```
/dev/sdb: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="crypto_LUKS"
```

```bash
# /etc/crypttab এ যোগ করুন
sudo nano /etc/crypttab
```

**Format:**
```
# <name>           <device>                                    <key_file>    <options>
myencrypteddisk    UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /root/myluks.key  luks
```

**তারপর /etc/fstab-এ mount যোগ করুন:**
```bash
sudo nano /etc/fstab
```

```
/dev/mapper/myencrypteddisk    /mnt/secure    ext4    defaults    0 2
```

এখন reboot-এ automatically unlock ও mount হবে!

## Encryption-এ Existing Partition (Full Disk)

Real production server-এ পুরো disk encrypt করতে:

```bash
# নতুন disk encrypt করুন (data destroy হবে!)
sudo cryptsetup luksFormat --type luks2 \
  --cipher aes-xts-plain64 \
  --hash sha256 \
  --iter-time 2000 \
  --key-size 256 \
  /dev/sdb
```

**Options breakdown:**
| Option | কাজ |
|---|---|
| `--type luks2` | নতুন LUKS2 format |
| `--cipher aes-xts-plain64` | Encryption algorithm |
| `--hash sha256` | Key derivation hash |
| `--iter-time 2000` | Key derivation time (ms) - brute force slow করে |
| `--key-size 256` | Key size in bits |


## LUKS1 vs LUKS2 তুলনা

| Feature | LUKS1 | LUKS2 |
|---|---|---|
| Key Slots | 8 | 32 |
| Header size | ছোট | বড় (better metadata) |
| Integrity | না | হ্যাঁ (dm-integrity) |
| Performance | ভালো | আরও ভালো |
| Argon2 support | না | হ্যাঁ (better key derivation) |
| Recommendation | Legacy | Use this |


## Encryption Methods তুলনা

| Method | কী Encrypt হয় | DevOps Use Case |
|---|---|---|
| **LUKS (Full disk)** | পুরো disk/partition | Database server, sensitive servers |
| **eCryptfs** | Individual files/directories | Home directory encryption |
| **VeraCrypt** | Cross-platform containers | Portable encrypted containers |
| **dm-crypt** | Block device level | LUKS এর underlying layer |
| **fscrypt** | File system level | Android, modern Linux |


## Common cryptsetup Commands চিটশিট

```bash
# Format (encrypt) a device
sudo cryptsetup luksFormat /dev/sdb

# Open/unlock an encrypted device
sudo cryptsetup luksOpen /dev/sdb myname

# Close/lock an encrypted device
sudo cryptsetup luksClose myname

# Show LUKS header info
sudo cryptsetup luksDump /dev/sdb

# Add a new key/password
sudo cryptsetup luksAddKey /dev/sdb

# Remove a key/password
sudo cryptsetup luksRemoveKey /dev/sdb

# Check if device is LUKS encrypted
sudo cryptsetup isLuks /dev/sdb && echo "YES - LUKS encrypted"

# Backup LUKS header (CRITICAL!)
sudo cryptsetup luksHeaderBackup /dev/sdb --header-backup-file /backup/luks-header.img

# Restore LUKS header
sudo cryptsetup luksHeaderRestore /dev/sdb --header-backup-file /backup/luks-header.img

# Change encryption password
sudo cryptsetup luksChangeKey /dev/sdb
```


## DevOps Real-World Scenarios

### Scenario 1: Database Server Disk Encryption

```bash
# Production DB server-এ data disk encrypt করুন
sudo cryptsetup luksFormat --type luks2 /dev/sdb
sudo cryptsetup luksOpen /dev/sdb db_data
sudo mkfs.xfs /dev/mapper/db_data
sudo mkdir /var/lib/mysql
sudo mount /dev/mapper/db_data /var/lib/mysql
```

### Scenario 2: Backup Encryption

```bash
#!/bin/bash
# encrypted_backup.sh
BACKUP_DISK="/dev/sdc"
KEY_FILE="/root/.backup.key"

# Unlock
cryptsetup luksOpen $BACKUP_DISK backup_vol --key-file $KEY_FILE

# Mount
mount /dev/mapper/backup_vol /mnt/backup

# Backup করুন
rsync -av /important/data/ /mnt/backup/

# Lock করুন
umount /mnt/backup
cryptsetup luksClose backup_vol

echo "Encrypted backup complete!"
```

### Scenario 3: LUKS Header Backup (অত্যন্ত গুরুত্বপূর্ণ!)

```bash
# Header backup করুন - এটা না করলে data চিরতরে হারাতে পারে!
sudo cryptsetup luksHeaderBackup /dev/sdb \
  --header-backup-file /secure_backup/luks-header-$(date +%Y%m%d).img

# Backup verify করুন
sudo cryptsetup luksDump /secure_backup/luks-header-20240315.img
```

> **Critical Warning:** LUKS header corrupt হলে সব data permanently হারাবে। Header backup সবসময় করতে হবে এবং secure জায়গায় রাখতে হবে!


## Important Security Considerations

```
১. Password ভুলে গেলে = Data চিরতরে গেছে (No recovery!)
২. Header corrupt = Same result
৩. Key file হারালে = Locked out
৪. সবসময় header backup রাখুন
৫. Key file কে /root/ তে রাখুন, permission 400
৬. Swap partition-ও encrypt করুন (sensitive data leak হতে পারে)
```

## 📝 Quick Summary

- **LUKS** হলো Linux-এর standard disk encryption format
- **cryptsetup** হলো LUKS manage করার tool
- **luksFormat** → disk encrypt করে
- **luksOpen** → unlock করে
- **luksClose** → lock করে
- Multiple passwords বা key files রাখা যায়
- **/etc/crypttab** দিয়ে auto-mount করা যায়
- **Header backup অত্যন্ত জরুরি** - না থাকলে data গেলে আর ফেরানো সম্ভব না


## 🏋️ Practice Tasks

**Task 1 - Virtual Encrypted Disk তৈরি করুন:**
```bash
# 1. 200MB loop device তৈরি করুন
dd if=/dev/zero of=/tmp/practice_disk.img bs=1M count=200
sudo losetup /dev/loop20 /tmp/practice_disk.img

# 2. LUKS encrypt করুন
sudo cryptsetup luksFormat /dev/loop20

# 3. Open করুন, ext4 format করুন, mount করুন
# 4. কিছু file রাখুন
# 5. Unmount ও close করুন
# 6. Raw device থেকে পড়ার চেষ্টা করুন (garbage দেখাবে)
```

**Task 2 - দ্বিতীয় Password যোগ করুন:**
```bash
# আপনার encrypted device-এ একটা second password যোগ করুন
# luksDump দিয়ে verify করুন যে দুটো key slot ব্যবহার হচ্ছে
```

**Task 3 - Key File দিয়ে Unlock করুন:**
```bash
# একটা key file তৈরি করুন
# সেই key file LUKS-এ যোগ করুন
# Key file দিয়ে unlock করুন (password ছাড়া)
```


## ⏭️ What's Next

আপনাকে অভিনন্দন 🎉! আপনি Chapter 7 সম্পন্ন করেছেন। এই পর্যন্ত অনেকেই আসতে পারেনি, আপনি পেরেছেন। আপনার এই প্রচেস্টা সফল হোক। আমরা সামনে আরো অনেক কিছু শিখবো।

আপনি এখন Linux Storage & File Systems-এর সব গুরুত্বপূর্ণ বিষয় শিখে ফেলেছেন:
- Disk basics থেকে শুরু করে partitioning
- LVM, RAID, NFS, Samba
- এবং এখন Disk Encryption পর্যন্ত!

পরবর্তী Chapter হলো Chapter 8 - Security Hardening, যেখানে আমরা শিখবো:

**Chapter 8, Lesson 1: Linux Security Model Overview** SELinux, AppArmor, capabilities, এবং Linux-কে production-ready secure করার পুরো roadmap! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../08-NFS-Network-File-System">← NFS Network File System</a>
    </td>
    <td align="right">
      <a href="../10-Disk-Encryption">Disk Encryption →</a>
    </td>
  </tr>
</table>