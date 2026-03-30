# Chapter 3 - Lesson 8: Disk I/O Monitoring

**Chapter 3 | Lesson 8 of 10**


## 🎯 এই Lesson-এ আমরা যা শিখবো

আজকে আমরা শিখবো **Disk I/O Monitoring**। মানে আপনার Linux system-এর disk কতটা busy, কতটা data read/write হচ্ছে, কোন process disk ব্যবহার করছে এই সব কিছু track করা।

> **Real-world analogy:** কল্পনা করুন আপনার বাসায় একটা পানির পাইপ আছে। `df` দিয়ে বুঝবে ট্যাংকে কতটুকু পানি আছে, `du` দিয়ে বুঝবে কোন রুমে কতটা পানি খরচ হচ্ছে, `iostat` দিয়ে বুঝবে পাইপ দিয়ে কত দ্রুত পানি যাচ্ছে আসছে, আর `iotop` দিয়ে দেখবে কোন মানুষটা সবচেয়ে বেশি পানি ব্যবহার করছে।


## Tool গুলোর Overview

| Tool | কাজ কী |
|------|---------|
| `df` | Disk-এ কতটুকু space আছে/ব্যবহার হয়েছে দেখায় |
| `du` | কোন file/directory কতটা space নিচ্ছে দেখায় |
| `iostat` | Disk-এর read/write speed ও CPU usage দেখায় |
| `iotop` | কোন process সবচেয়ে বেশি disk I/O করছে দেখায় |


## 1. `df` - Disk Free (Disk Space দেখা)

### ধারণা:
`df` মানে **Disk Free**। এটা দিয়ে আপনি দেখতে পারবেন আপনার system-এর প্রতিটা **mounted filesystem**-এ কতটুকু space আছে, কতটুকু ব্যবহার হয়েছে।

> **Analogy:** এটা আপনার ফোনের Storage সেটিংস-এর মতো total, used, available দেখায়।


### Syntax:
```bash
df [options] [filesystem]
```


### সবচেয়ে Important Options:

| Option | মানে |
|--------|------|
| `-h` | Human-readable (KB, MB, GB তে দেখায়) |
| `-H` | Powers of 1000 ব্যবহার করে (1K=1000) |
| `-T` | Filesystem type দেখায় (ext4, xfs, tmpfs) |
| `-i` | Inode usage দেখায় |
| `--total` | সব মিলিয়ে total দেখায় |


### Example 1 - Basic disk space দেখা:
```bash
df -h
```

**Expected Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   18G   30G  38% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
/dev/sda2       100G   45G   50G  48% /home
/dev/sdb1       200G   80G  110G  43% /data
```

**চলুন Output বুঝি:**
| Column | মানে |
|--------|------|
| `Filesystem` | কোন disk বা partition |
| `Size` | মোট size |
| `Used` | কতটুকু ব্যবহার হয়েছে |
| `Avail` | কতটুকু বাকি আছে |
| `Use%` | কত % ব্যবহার হয়েছে |
| `Mounted on` | কোথায় mount করা আছে |


### Example 2 - Filesystem type সহ দেখা:
```bash
df -hT
```

**Expected Output:**
```
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sda1      ext4       50G   18G   30G  38% /
tmpfs          tmpfs     2.0G     0  2.0G   0% /dev/shm
/dev/sda2      xfs       100G   45G   50G  48% /home
```


### Example 3 - নির্দিষ্ট directory-র disk দেখা:
```bash
df -h /home
```

**Expected Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       100G   45G   50G  48% /home
```


### Example 4 - Inode usage দেখা:
```bash
df -i
```

**Expected Output:**
```
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/sda1      3276800 150000 3126800    5% /
```

> **DevOps Tip:** অনেক সময় disk-এ space থাকলেও inode শেষ হয়ে যায়, তখন নতুন file তৈরি করা যায় না! এটা একটা common production issue।


### DevOps Real-World Use Case:
```bash
# Disk 80% এর বেশি হলে alert দেয়া যা monitoring script এ ব্যবহার হয়
df -h | awk 'NR>1 {gsub(/%/,""); if ($5 > 80) print "WARNING: "$6" is "$5"% full"}'
```


## 2. `du` - Disk Usage (কোথায় কতটা space যাচ্ছে)

### ধারণা:
`du` মানে **Disk Usage**। এটা দিয়ে আপনি দেখতে পারবেন কোন **file বা directory** কতটুকু disk space ব্যবহার করছে।

> `df` আপনাকে বলে দেয় পুরো ঘরে কতটা জায়গা আছে। `du` আপনাকে বলে কোন আলমারিতে, কোন ড্রয়ারে কতটা জায়গা নিচ্ছে।


### Syntax:
```bash
du [options] [directory/file]
```


### Important Options:

| Option | মানে |
|--------|------|
| `-h` | Human-readable format |
| `-s` | Summary only (শুধু total দেখায়) |
| `-a` | সব file দেখায় (directory + file) |
| `--max-depth=N` | কত level গভীরে যাবে |
| `-c` | Grand total দেখায় |
| `--exclude` | নির্দিষ্ট pattern বাদ দেয় |


### Example 1 - Current directory-র size দেখা:
```bash
du -sh .
```

**Expected Output:**
```
4.5G    .
```


### Example 2 - প্রতিটা subdirectory-র size দেখা:
```bash
du -h --max-depth=1 /home
```

**Expected Output:**
```
1.2G    /home/alice
3.4G    /home/bob
512M    /home/charlie
5.1G    /home
```


### Example 3 - সবচেয়ে বড় directory খোঁজা (DevOps এ খুব কাজের!):
```bash
du -h /var | sort -rh | head -10
```

**Expected Output:**
```
8.5G    /var
4.2G    /var/log
2.1G    /var/log/nginx
1.8G    /var/lib
900M    /var/lib/docker
...
```

> এটা DevOps-এ সবচেয়ে বেশি ব্যবহার হয়। disk full হলে এই command দিয়ে কোথায় space গেছে বের করুন।


### Example 4 - নির্দিষ্ট file-এর size:
```bash
du -sh /var/log/syslog
```

**Expected Output:**
```
256M    /var/log/syslog
```


### Example 5 - Docker image/container space দেখা (DevOps):
```bash
du -sh /var/lib/docker/
```


### DevOps Combo - df + du একসাথে:
```bash
# প্রথমে df দিয়ে কোন partition full দেখুন
df -h
# তারপর সেই partition-এ du দিয়ে কোথায় space গেছে বের করুন
du -h /var --max-depth=2 | sort -rh | head -20
```


## 3. `iostat` - I/O Statistics (Disk Read/Write Speed)

### ধারণা:
`iostat` দিয়ে আপনি দেখতে পাবেন আপনার disk কত দ্রুত data **read** ও **write** করছে, এবং সেই সাথে CPU usage-ও দেখায়।

> এটা আপনার disk-এর speedometer বলে দেয় disk কত fast কাজ করছে।

### Installation (যদি না থাকে):
```bash
# Ubuntu/Debian
sudo apt install sysstat -y

# RHEL/CentOS
sudo yum install sysstat -y
```


### Syntax:
```bash
iostat [options] [device] [interval] [count]
```

| Part | মানে |
|------|------|
| `interval` | কত সেকেন্ড পর পর update দেখাবে |
| `count` | কতবার দেখাবে |


### Important Options:

| Option | মানে |
|--------|------|
| `-x` | Extended statistics (বিস্তারিত) |
| `-d` | শুধু disk statistics (CPU বাদ) |
| `-c` | শুধু CPU statistics |
| `-h` | Human-readable |
| `-m` | MB তে দেখায় |
| `-p` | Partition সহ দেখায় |


### Example 1 - Basic iostat:
```bash
iostat
```

**Expected Output:**
```
Linux 5.15.0-76-generic (myserver)   03/11/2026   _x86_64_   (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.25    0.00    1.45    0.82    0.00   94.48

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               8.20       145.30        89.45    1024000     630000
sdb               2.10        20.15        45.30     142000     319000
```

**Output বুঝি:**

**CPU Section:**
| Column | মানে |
|--------|------|
| `%user` | User program কতটা CPU ব্যবহার করছে |
| `%system` | Kernel কতটা CPU ব্যবহার করছে |
| `%iowait` | CPU disk-এর জন্য কতক্ষণ অপেক্ষা করছে |
| `%idle` | CPU কতটা সময় খালি থাকছে |

> ⚠️ **`%iowait` বেশি হলে সমস্যা!** এর মানে CPU কাজ করতে পারছে না কারণ disk slow, এটা **I/O bottleneck**।

**Disk Section:**
| Column | মানে |
|--------|------|
| `tps` | Transactions per second (কতটা request আসছে) |
| `kB_read/s` | প্রতি সেকেন্ডে কতটা KB read হচ্ছে |
| `kB_wrtn/s` | প্রতি সেকেন্ডে কতটা KB write হচ্ছে |


### Example 2 - Extended statistics (সবচেয়ে useful!):
```bash
iostat -x 2 3
```
> এই command ২ সেকেন্ড পর পর, মোট ৩ বার দেখাবে।

**Expected Output:**
```
Device  rrqm/s wrqm/s  r/s   w/s  rkB/s  wkB/s avgrq-sz avgqu-sz  await r_await w_await svctm  %util
sda       0.10   2.30 5.20  3.00 145.30  89.45    45.20     0.08   8.50    7.20    10.80  2.30  12.50
```

**Key Columns বুঝি:**
| Column | মানে | Threshold |
|--------|------|-----------|
| `r/s` | প্রতি সেকেন্ডে read operations | - |
| `w/s` | প্রতি সেকেন্ডে write operations | - |
| `rkB/s` | Read speed (KB/s) | - |
| `wkB/s` | Write speed (KB/s) | - |
| `await` | Average wait time (milliseconds) | >20ms = সমস্যা |
| `%util` | Disk কতটা busy (%) | >80% = সমস্যা |


### Example 3 - নির্দিষ্ট disk monitor করা:
```bash
iostat -x -d /dev/sda 1 5
```
> `/dev/sda` কে ১ সেকেন্ড পর পর ৫ বার দেখাবে।


### Example 4 - MB তে দেখা:
```bash
iostat -mx 2
```


### DevOps Real-World Use Case:
```bash
# Disk bottleneck আছে কিনা চেক করুন
iostat -x 1 | awk '/^sd/ {if ($NF > 80) print "ALERT: "$1" disk utilization is "$NF"%"}'
```


## 4. `iotop` - I/O Top (কোন Process সবচেয়ে বেশি Disk ব্যবহার করছে)

`iotop` হলো `top`-এর মতো - কিন্তু CPU-র বদলে **Disk I/O** দেখায়। কোন process সবচেয়ে বেশি disk read/write করছে সেটা real-time-এ দেখা যায়।

> `top` দেখায় কে বেশি CPU খাচ্ছে, `iotop` দেখায় কে বেশি disk খাচ্ছে।

### Installation:
```bash
# Ubuntu/Debian
sudo apt install iotop -y

# RHEL/CentOS
sudo yum install iotop -y
```


### Syntax:
```bash
sudo iotop [options]
```

> `iotop` চালাতে **sudo** লাগে।


### Important Options:

| Option | মানে |
|--------|------|
| `-o` | শুধু active (I/O করছে এমন) process দেখায় |
| `-b` | Batch mode (script-এ ব্যবহারের জন্য) |
| `-n N` | N বার দেখিয়ে বন্ধ হয়ে যাবে |
| `-d N` | N সেকেন্ড পর পর update |
| `-p PID` | নির্দিষ্ট PID monitor করা |
| `-u USER` | নির্দিষ্ট user-এর process দেখা |


### Example 1 - Basic iotop:
```bash
sudo iotop
```

**Expected Output:**
```
Total DISK READ:       45.23 M/s | Total DISK WRITE:      12.45 M/s
Current DISK READ:     45.23 M/s | Current DISK WRITE:    12.45 M/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 1234 be/4  mysql      40.12 M/s   8.23 M/s  0.00 %  92.45 % mysqld
 5678 be/4  root        4.50 M/s   4.10 M/s  0.00 %  45.20 % rsync
  890 be/4  www-data    0.61 M/s   0.12 M/s  0.00 %   5.30 % nginx
```

**Output বুঝি:**
| Column | মানে |
|--------|------|
| `TID` | Thread ID |
| `PRIO` | Priority |
| `USER` | কোন user-এর process |
| `DISK READ` | প্রতি সেকেন্ডে কতটা read করছে |
| `DISK WRITE` | প্রতি সেকেন্ডে কতটা write করছে |
| `IO>` | মোট I/O-র কত % এই process নিচ্ছে |
| `COMMAND` | Process-এর নাম |


### Example 2 - শুধু active process দেখা (সবচেয়ে useful):
```bash
sudo iotop -o
```

> এটাই সবচেয়ে বেশি ব্যবহার হয়, শুধু যারা এই মুহূর্তে disk ব্যবহার করছে তাদের দেখায়।


### Example 3 - Batch mode (log করার জন্য):

```bash
sudo iotop -b -n 5 -d 2
```

> ২ সেকেন্ড পর পর, ৫ বার output দেবে, script বা cron-এ ব্যবহার করা যায়।


### Example 4 - নির্দিষ্ট user monitor করা:
```bash
sudo iotop -u mysql
```


### Interactive Keyboard Shortcuts (iotop চলাকালীন):

| Key | কাজ |
|-----|-----|
| `o` | Only active processes toggle |
| `p` | Process/Thread toggle |
| `a` | Accumulated I/O দেখায় |
| `q` | Quit |
| `←` `→` | Sort column পরিবর্তন |


## সব Tool একসাথে - Real DevOps Scenario

### Scenario: Production server slow হয়ে গেছে, disk সমস্যা মনে হচ্ছে!

```bash
# Step 1: প্রথমে disk space চেক করুন
df -h
# Output: /dev/sda1 এ 95% full দেখাচ্ছে!

# Step 2: কোথায় space গেছে বের করুন
du -h / --max-depth=2 | sort -rh | head -20
# Output: /var/log তে 40GB দেখাচ্ছে!

# Step 3: Disk I/O কেমন চলছে দেখুন
iostat -x 2 5
# Output: %iowait 60%, %util 95% - disk অনেক busy!

# Step 4: কোন process disk খাচ্ছে দেখুন
sudo iotop -o
# Output: mysql process 80MB/s write করছে!

# Step 5: সমস্যা চিহ্নিত - পুরনো log delete করুন এবং mysql query optimize করুন
```


## Quick Reference Table

| পরিস্থিতি | Command |
|-----------|---------|
| Disk কতটুকু ভর্তি? | `df -h` |
| কোন folder সবচেয়ে বড়? | `du -h / --max-depth=2 \| sort -rh \| head -20` |
| Disk কত fast read/write করছে? | `iostat -x 2` |
| কোন process disk খাচ্ছে? | `sudo iotop -o` |
| Specific disk monitor | `iostat -x /dev/sda 1` |
| Log file কতটা বড়? | `du -sh /var/log/syslog` |


## Important Thresholds (DevOps এ মনে রাখুন)

| Metric | Normal | Warning | Critical |
|--------|--------|---------|----------|
| `df` Use% | < 70% | 70-85% | > 85% |
| `iostat %util` | < 60% | 60-80% | > 80% |
| `iostat await` | < 10ms | 10-20ms | > 20ms |
| `iostat %iowait` | < 10% | 10-25% | > 25% |


## 📝 Quick Summary

-  `df -h` → Disk-এ কতটুকু space আছে/নেই দেখায়
-  `du -sh` → কোন directory কতটা space নিচ্ছে দেখায়
-  `iostat -x` → Disk-এর read/write speed ও busy-ness দেখায়
-  `iotop -o` → কোন process সবচেয়ে বেশি disk ব্যবহার করছে দেখায়
-  `%iowait` বেশি = CPU disk-এর জন্য অপেক্ষা করছে মানে I/O bottleneck
-  `%util` > 80% = Disk অনেক busy = সমস্যা হতে পারে

> Production-এ কোন issue হলে এই sequence ফলো করা ভালো: `df` → `du` → `iostat` → `iotop`


## 🏋️ Practice Tasks

**Task 1:**
```bash
# আপনার system-এ এই command গুলো run করুন এবং output বোঝার চেষ্টা করুন:
df -hT
du -h /var --max-depth=2 | sort -rh | head -10
```

**Task 2:**
```bash
# iostat install করুন এবং run করুন:
sudo apt install sysstat -y
iostat -x 2 3
# %iowait এবং %util column খুঁজে বের করুন
```

**Task 3:**
```bash
# iotop দিয়ে দেখুন কোন process disk ব্যবহার করছে:
sudo apt install iotop -y
sudo iotop -o
# 'o' key press করে active process filter করুন
```

---

## ⏭️ What's Next?

**Lesson 9: System-wide Performance (dstat, sar, glances)**

পরের lesson-এ আমরা শিখবো কীভাবে CPU, Memory, Disk, Network সব কিছু **একসাথে** monitor করা যায় `dstat`, `sar`, এবং `glances` দিয়ে। এটা হবে আমাদের monitoring chapter-এর সবচেয়ে powerful lesson! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../07-CPU-Monitoring">← CPU Monitoring</a>
    </td>
    <td align="right">
      <a href="../09-System-wide-Performance">System-wide Performance →</a>
    </td>
  </tr>
</table>