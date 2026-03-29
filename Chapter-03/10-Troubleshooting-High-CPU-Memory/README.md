# Chapter 3 - Lesson 10: Troubleshooting High CPU/Memory

**Chapter 3 | Lesson 10 of 10**


## 🎯 এই Lesson-এ আমরা যা কী শিখবো

এই lesson-টা হলো Chapter 3-এর **grand finale**। এখানে আমরা আগের সব lesson-এ শেখা tools গুলো ব্যবহার করে **real-world DevOps problems** solve করবো।

একজন DevOps Engineer হিসেবে আপনাকে প্রায়ই এই ধরনের situations face করতে হবে:

- "Server slow হয়ে গেছে, কী হচ্ছে?"
- "Application crash করছে, কেন?"
- "Disk full হয়ে গেছে, কোথায় জায়গা খাচ্ছে?"
- "Memory leak হচ্ছে কোথাও"

চলুন এগুলো **step-by-step systematically** debug করতে শিখি।


## The Golden Rule of Troubleshooting

> **"দেখুন → বুঝুন → Act করুন"**

কখনো blindly commands run করা যাবে না। প্রতিটা step-এ জানার চেস্টা করুন আপনি কী দেখছেন এবং কেন।


## Troubleshooting Workflow (Mental Model)

```
Problem আসলো
      ↓
System কি alive? (ping, ssh)
      ↓
Overall health check (uptime, top, free)
      ↓
CPU problem?      Memory problem?      Disk problem?
      ↓                  ↓                   ↓
   ps, top            free, vmstat       df, du, iostat
      ↓                  ↓                   ↓
Culprit process খুঁজো
      ↓
Kill করো / Fix করো / Escalate করো
```


## Scenario 1: High CPU Usage

### সমস্যা:
> "Server-এর response slow, সব কিছু hang করছে"


### Step 1 - প্রথমে Overall Picture দেখুন

```bash
uptime
```

**Output:**
```
10:45:23 up 5 days,  2:30,  3 users,  load average: 8.52, 7.30, 6.10
```

**বিশ্লেষণ:**
- আপনার server যদি **4-core** হয়, তাহলে load average 4.0 = 100% busy
- এখানে **8.52** মানে server-এর capacity-র **দ্বিগুণ** load আছে!

> **Analogy:** একটা 4-lane highway-তে 8টা গাড়ি এক সাথে চলতে চাইছে, jam inevitable!


### Step 2 - কোন Process CPU খাচ্ছে?

```bash
top
```

`top` open হলে **`P` key** চাপুন, CPU usage অনুযায়ী sort হবে।

**Output (example):**
```
PID    USER    PR   NI   VIRT    RES   SHR  S  %CPU  %MEM   TIME+    COMMAND
1842   www     20    0  512m    200m   10m  R  95.0   5.0  10:23.45  python3
2341   mysql   20    0  1.2g    800m   20m  S  45.0  20.0   5:10.22  mysqld
1001   root    20    0   25m     10m    5m  S   1.0   0.2   0:01.10  sshd
```

**দেখুন:** `python3` process (PID: 1842) CPU-র **95%** খাচ্ছে! এটাই villain! 🦹


### Step 3 - Process সম্পর্কে আরো জানুন

```bash
# Process-টা কোথা থেকে এসেছে?
ps -p 1842 -o pid,ppid,cmd,%cpu,%mem,etime
```

**Output:**
```
PID   PPID  CMD                        %CPU  %MEM  ELAPSED
1842  1500  python3 /opt/app/worker.py  95.0   5.0  00:10:23
```

**এখন জানলাম:** `/opt/app/worker.py` script-টা 10 মিনিট ধরে CPU খাচ্ছে।

```bash
# Parent process কে?
ps -p 1500 -o pid,cmd
```

```
PID   CMD
1500  /bin/bash /opt/app/start.sh
```


### Step 4 - Process-এর Details দেখা

```bash
# Process কোন files open করে রেখেছে?
lsof -p 1842 | head -20

# Process কোন network connection করছে?
lsof -p 1842 -i
```


### Step 5 - সিদ্ধান্ত নিন

```bash
# Option 1: Priority কমিয়ে দিন (kill না করে)
renice +15 -p 1842
# এখন এই process কম CPU পাবে, অন্যরা সুযোগ পাবে

# Option 2: Kill করো (যদি rogue process হয়)
kill -15 1842    # Graceful kill (SIGTERM)

# কাজ না হলে:
kill -9 1842     # Force kill (SIGKILL)

# Option 3: Developer-কে জানান যে bug আছে
```


### One-liner: সবচেয়ে বেশি CPU খাওয়া top 5 process

```bash
ps aux --sort=-%cpu | head -6
```

**Output:**
```
USER       PID  %CPU  %MEM    VSZ   RSS  STAT  COMMAND
www       1842  95.0   5.0  512000 200000  R   python3 /opt/app/worker.py
mysql     2341  45.0  20.0 1200000 800000  S   mysqld
nginx      891   2.5   1.0  50000  10000   S   nginx: worker
root      1001   1.0   0.2  25000  10000   S   sshd
root         1   0.1   0.1  20000   8000   S   systemd
```


## Scenario 2: High Memory Usage / Memory Leak

### সমস্যা:
> "Application crash করছে, OOM error আসছে"

**OOM = Out Of Memory** - Linux যখন memory শেষ হয়ে যায়, তখন OOM Killer automatically processes kill করে দেয়।


### Step 1 - Memory র Current State দেখা

```bash
free -h
```

**Output:**
```
              total    used    free   shared  buff/cache   available
Mem:            8.0G    7.8G   100M      50M        200M       150M
Swap:           2.0G    1.9G   100M
```

**বিশ্লেষণ:**
- RAM: 8GB এর মধ্যে **7.8GB used** - প্রায় শেষ!
- Swap: 2GB এর মধ্যে **1.9GB used** - Swap-ও শেষের পথে!
- Available: মাত্র **150MB** বাকি!

> **Analogy:** আপনার পকেটে মাত্র ১৫০ টাকা আছে, এবং ATM (swap)-ও প্রায় empty!


### Step 2 - কে Memory খাচ্ছে?

```bash
# Memory অনুযায়ী sort করে দেখুন
top
# তারপর 'M' key চাপুন
```

**অথবা:**

```bash
ps aux --sort=-%mem | head -10
```

**Output:**
```
USER      PID   %CPU  %MEM    VSZ       RSS   COMMAND
java     3421    5.0  45.0  4200000  3600000  java -jar app.jar
mysql    2341    2.0  20.0  1200000   800000  mysqld
node     4521   10.0  15.0   800000   600000  node server.js
```

**দেখুন:** `java` process মেমরির **45%** খাচ্ছে! (3.6GB RAM!)


### Step 3 - Memory Leak detect করো

```bash
# প্রতি 2 সেকেন্ডে process-এর memory দেখুন
watch -n 2 'ps -p 3421 -o pid,%mem,rss,vsz'
```

**Output (কয়েক মিনিট ধরে দেখুন):**
```
# 1st check:
PID   %MEM    RSS      VSZ
3421  45.0  3600000  4200000

# 2nd check (2 min later):
PID   %MEM    RSS      VSZ
3421  47.0  3760000  4300000

# 3rd check (4 min later):
PID   %MEM    RSS      VSZ
3421  50.0  4000000  4500000
```

**Memory ক্রমশ বাড়ছে কিন্তু কমছে না = Memory Leak!**


### Step 4 - OOM Killer Log চেক করুন

```bash
# OOM killer কাউকে kill করেছে কিনা দেখুন
dmesg | grep -i "oom\|killed process\|out of memory"
```

**Output:**
```
[142356.789] Out of memory: Kill process 3421 (java) score 900 or sacrifice child
[142356.790] Killed process 3421 (java) total-vm:4200000kB, anon-rss:3600000kB
```

**মানে:** Linux OOM Killer তোমার java application-কে automatically kill করে দিয়েছে! এজন্যই crash হচ্ছিলো।


### Step 5 - vmstat দিয়ে Memory প্রেশার দেখুন

```bash
vmstat 2 5
```

**Output:**
```
procs ----memory---- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs  us sy id wa
 3  1 1900000  100000 50000 200000  500  800  1000  1500  800 1200 80  5  5 10
 4  2 1950000   80000 50000 200000  600  900  1200  1800  900 1300 85  5  3 12
```

**Key columns:**
| Column | মানে | এখন কী হচ্ছে? |
|--------|------|----------------|
| `r` | Run queue (waiting processes) | 3-4 টা process অপেক্ষায় |
| `si` | Swap in (disk→RAM) | 500-600 KB/s - অনেক বেশি! |
| `so` | Swap out (RAM→disk) | 800-900 KB/s - অনেক বেশি! |
| `wa` | Wait for I/O | 10-12% - disk I/O slow |

**High si/so = RAM শেষ, disk-এ swap হচ্ছে = System slow হবেই!**


### Step 6 - সমাধান

```bash
# Option 1: Memory cache clear করুন (ফাঁকা memory free করে)
sync && echo 3 > /proc/sys/vm/drop_caches
# এটা file cache clear করে, running processes-কে affect করে না

# Option 2: Problematic process restart করুন
systemctl restart myapp

# Option 3: Process kill করুন
kill -9 3421

# Option 4: Swappiness কমিয়ে ফেলুন (কম swap use করবে)
sysctl vm.swappiness=10
# Default 60 থেকে 10 করলে RAM prefer করবে
```


## Scenario 3: Disk Full - "No space left on device"

### সমস্যা:
> "Application log write করতে পারছে না, disk full error!"


### Step 1 - কোন Partition Full?

```bash
df -h
```

**Output:**
```
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   49G   500M   99%  /
/dev/sda2       100G   20G    80G   20%  /data
tmpfs             4G   100M    4G    3%  /dev/shm
```

**দেখুন:** Root partition `/` মাত্র **500MB free**!

---

### Step 2 - কোন Directory বেশি জায়গা নিচ্ছে?

Top-level directories দেখুন

```bash
du -sh /*  2>/dev/null | sort -rh | head -10
```

**Output:**
```
15G    /var
12G    /home
8G     /usr
5G     /opt
2G     /root
```

**`/var` সবচেয়ে বেশি!** এখন `/var`-এর মধ্যে প্রবেশ করি:

```bash
du -sh /var/*  2>/dev/null | sort -rh | head -10
```

**Output:**
```
14G    /var/log
500M   /var/lib
200M   /var/cache
```

**`/var/log` = 14GB!** Log files জমে গেছে! এখন:

```bash
du -sh /var/log/* | sort -rh | head -10
```

**Output:**
```
8G    /var/log/app.log
3G    /var/log/nginx
2G    /var/log/mysql
500M  /var/log/syslog
```

**`app.log` একাই 8GB খেয়েছে!**


### Step 3 - Log File Investigate করুন

```bash
# File কবে থেকে আছে?
ls -lh /var/log/app.log

# শেষের কিছু লগ দেখা
tail -50 /var/log/app.log

# Error কতবার হচ্ছে?
grep -c "ERROR" /var/log/app.log
```


### Step 4 - জায়গা ফাঁকা করুন

```bash
# Option 1: Log file truncate করা (delete না করে খালি করো)
> /var/log/app.log
# OR
truncate -s 0 /var/log/app.log

# Option 2: পুরনো compressed logs delete করা
find /var/log -name "*.gz" -mtime +30 -delete
# 30 দিনের বেশি পুরনো compressed logs delete

# Option 3: নির্দিষ্ট size-এর বেশি files খুঁজে বের করা
find /var/log -size +1G -ls

# Option 4: Package cache clear করা
apt clean      # Ubuntu/Debian
yum clean all  # CentOS/RHEL
```


### Step 5 - Deleted Files এখনো Disk ধরে রেখেছে?

এটা একটা common gotcha! File delete করলেও যদি কোনো process সেই file open রাখে, disk space free হয় না।

```bash
# Deleted কিন্তু এখনো open files খুঁজে দেখা
lsof | grep deleted
```

**Output:**
```
nginx  1234  root  3r  REG  8,1  8589934592  12345  /var/log/app.log (deleted)
```

**Nginx এখনো সেই deleted file hold করে রেখেছে!** Solution:

```bash
# Nginx restart করলে file release হবে
systemctl restart nginx
# এখন df -h দেখুন - space free হয়ে যাবে
```


## Scenario 4: High Disk I/O

### সমস্যা:
> "Server respond করছে, কিন্তু অনেক slow। সব কিছু stuck মনে হচ্ছে।"


### Step 1 - I/O Wait দেখুন

```bash
top
```

CPU line দেখুন:
```
%Cpu(s): 5.0 us, 2.0 sy, 0.0 ni, 23.0 id, 70.0 wa, 0.0 hi, 0.0 si
```

**`wa = 70%`** মানে CPU-র 70% সময় disk I/O-র জন্য অপেক্ষা করছে। এটাই সমস্যা!


### Step 2 - কোন Process I/O করছে?

```bash
# iotop install না থাকলে:
apt install iotop

# চালাও:
iotop -o
# -o মানে শুধু active I/O করা processes দেখাও
```

**Output:**
```
TID    PRIO   USER     DISK READ    DISK WRITE   COMMAND
2341   be/4   mysql    0 B/s        150 M/s      mysqld
4521   be/4   www      50 M/s       0 B/s        php-fpm
```

**MySQL 150 MB/s লিখছে!** এটা কি normal? Investigate করুন।


### Step 3 - iostat দিয়ে Disk Statistics দেখা

```bash
iostat -x 2 3
```

**Output:**
```
Device   r/s    w/s   rMB/s  wMB/s  await  svctm  %util
sda      10.0  500.0    0.5  150.0  80.0    5.0   95.0
```

**Key values:**
| Column | মানে | এখন কী? |
|--------|------|---------|
| `await` | Average wait time (ms) | 80ms - অনেক বেশি! (normal < 5ms) |
| `%util` | Disk কতটা busy | 95% - প্রায় full! |


## Scenario 5: Server হঠাৎ Reboot হয়েছে - কেন?

### সমস্যা:
> "রাত 3টায় server reboot হয়েছে। কেউ করেনি। কী হয়েছিলো?"


### Step 1 - Reboot History দেখা

```bash
last reboot | head -10
```

**Output:**
```
reboot   system boot  5.15.0-78    Wed Mar 11 03:23   still running
reboot   system boot  5.15.0-78    Mon Mar  9 15:10 - 03:23 (2+12:13)
```

**রাত 3:23 AM-এ reboot হয়েছে।**


### Step 2 - Kernel Logs দেখা

```bash
# Reboot-এর আগে কী হয়েছিলো?
journalctl -b -1 | tail -50
# -b -1 মানে আগের boot-এর logs
```

```bash
# OOM kill হয়েছিলো কিনা?
journalctl -b -1 | grep -i "oom\|killed\|panic\|error" | tail -20
```

**Output:**
```
Mar 11 03:22:45 server kernel: Out of memory: Kill process 5432 (java) score 950
Mar 11 03:22:45 server kernel: Killed process 5432 (java) total-vm:8GB
Mar 11 03:22:50 server kernel: Kernel panic - not syncing: Fatal exception
Mar 11 03:22:50 server kernel: System is going down
```

**কারণ পাওয়া গেছে:** Java process সব memory শেষ করে দিয়েছিলো → Kernel panic → Automatic reboot!


### Step 3 - dmesg দিয়ে Hardware Error চেক করা

```bash
dmesg | grep -i "error\|fail\|warn" | tail -20
```

```bash
# Hardware issue আছে কিনা (disk, memory)?
dmesg | grep -iE "ata|sda|nvme|memory|mce" | tail -20
```


## The Complete DevOps Troubleshooting Cheat Sheet

```bash
# ============================================
# INITIAL ASSESSMENT (সবসময় এখান থেকে শুরু)
# ============================================

uptime                          # Load average দেখো
top                             # Overall system দেখো
free -h                         # Memory দেখো
df -h                           # Disk দেখো
iostat -x 1 3                   # Disk I/O দেখো

# ============================================
# CPU TROUBLESHOOTING
# ============================================

ps aux --sort=-%cpu | head -10  # Top CPU consumers
top (then press P)              # CPU sort in top
pidstat 1 5                     # Per-process CPU stats
mpstat -P ALL 1 3               # Per-core CPU stats

# ============================================
# MEMORY TROUBLESHOOTING
# ============================================

ps aux --sort=-%mem | head -10  # Top memory consumers
vmstat 2 5                      # Memory + swap stats
cat /proc/meminfo               # Detailed memory info
dmesg | grep -i oom             # OOM killer history

# ============================================
# DISK TROUBLESHOOTING
# ============================================

df -h                           # Partition usage
du -sh /* 2>/dev/null | sort -rh | head -10  # Largest dirs
find / -size +500M 2>/dev/null  # Large files খুঁজো
lsof | grep deleted             # Deleted but open files
iotop -o                        # Real-time I/O

# ============================================
# LOG INVESTIGATION
# ============================================

journalctl -b 0 | tail -100     # Current boot logs
journalctl -b -1 | tail -100    # Previous boot logs
journalctl -p err -b            # Only errors, current boot
dmesg | tail -50                # Kernel messages
tail -f /var/log/syslog         # Live syslog
```


## Real DevOps Investigation Template

যখনই কোনো production issue আসবে, এই sequence follow করুন:

```
1. ASSESS   → uptime, top, free -h, df -h
2. LOCATE   → ps aux, iotop, vmstat
3. IDENTIFY → lsof, dmesg, journalctl
4. ACT      → kill, renice, truncate, restart
5. PREVENT  → logrotate, monitoring, alerts
6. DOCUMENT → কী হয়েছিলো, কীভাবে fix হলো
```


## 📝 Quick Summary

- **High CPU** → `uptime` দিয়ে load দেখুন → `top` দিয়ে culprit process খুঁজুন → `renice` বা `kill` করুন
- **High Memory** → `free -h` দেখুন → `ps aux --sort=-%mem` → `vmstat` দিয়ে swap pressure দেখুন → `dmesg | grep oom` চেক করুন
- **Disk Full** → `df -h` দেখুন → `du -sh` দিয়ে drill down করুন → `lsof | grep deleted` চেক করুন
- **High I/O** → `top`-এ `wa%` দেখুন → `iotop` দিয়ে process খুঁজুন → `iostat` দিয়ে disk stats দেখুন
- **Unexpected Reboot** → `last reboot` → `journalctl -b -1` → `dmesg | grep -i error`


## 🏋️ Practice Tasks

**Task 1:** নিচের command run করুন এবং output বোঝার চেষ্টা করুন:
```bash
uptime && free -h && df -h
```
দেখে বলুন: Load average কত? Memory কতটুকু available? Disk কতটুকু free?

**Task 2:** CPU অনুযায়ী sort করা top 5 process দেখুন:
```bash
ps aux --sort=-%cpu | head -6
```

সবচেয়ে বেশি CPU কোন process খাচ্ছে?

**Task 3:** এই scenario simulate করুন:
```bash
# একটা CPU-intensive process তৈরি করুন
yes > /dev/null &
# এটার PID নোট করুন
# top দিয়ে দেখুন CPU কতটুকু নিচ্ছে
# তারপর kill করুন
kill %1
```


## Chapter 3 Complete!

**অভিনন্দন!** আপনি Chapter 3 - Process & Performance Management সম্পূর্ণ করে ফেলেছেন!

এই chapter-এ আমরা যা শিখেছি:
- Process কী এবং কীভাবে কাজ করে
- `ps`, `top`, `htop` দিয়ে process দেখা
- Signals দিয়ে process control করা
- Background/Foreground jobs manage করা
- `nice`/`renice` দিয়ে priority set করা
- Memory monitoring (`free`, `vmstat`)
- CPU monitoring (`mpstat`, `uptime`)
- Disk I/O monitoring (`iostat`, `iotop`)
- System-wide performance tools (`dstat`, `sar`)
- **Real-world troubleshooting scenarios**

---

## ⏭️ What's Next

Chapter 3 শেষ করার জন্য আপনাকে অনেক অভিনন্দন! 🎉

এই chapter-এ আমরা Linux-এর process ও performance management নিয়ে বিস্তারিত শিখেছি। একজন DevOps Engineer হিসেবে এগুলো প্রতিদিনের কাজে লাগবেই।

Chapter 3 শেষে একটি Assessment দেওয়া আছে যেখানে real-world scenario-based প্রশ্ন আছে। এই assessment-এ process troubleshooting, performance bottleneck identify করা এবং সঠিক tool ব্যবহারের দক্ষতা যাচাই করার জন্য বাস্তম সিনারিও তুলে ধরা হয়েছে। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../09-System-wide-Performance">← System-wide Performance</a>
    </td>
    <td align="right">
      <a href="../11-Assessment">Chapter 3 - Assessment →</a>
    </td>
  </tr>
</table>



