# Chapter 3 - Lesson 7: CPU Monitoring

**Chapter 3 | Lesson 7 of 10**


## 🎯 এই Lesson-এ আমরা যা শিখব

আজকে আমরা শিখব কীভাবে Linux-এ **CPU** monitor করতে হয়। DevOps Engineer হিসেবে আপনাকে জানতে হবে - system কতটা busy, কোন process কত CPU নিচ্ছে, এবং server কতক্ষণ ধরে চলছে।

আজকের tools:
- `lscpu` - CPU-র hardware info
- `mpstat` - CPU usage statistics
- `uptime` - system কতক্ষণ চলছে
- `Load Average` - এটা কী এবং কীভাবে পড়তে হয়


## CPU কী করে?

মনে করেন CPU হলো একটা রান্নাঘরের বাবুর্চির মতো।
- সে একসাথে অনেক কাজ করতে পারে (multi-core)
- যদি তার কাছে অনেক বেশি order আসে → সে overwhelmed হয়ে যায় → system slow হয়
- আমাদের কাজ হলো monitor করা, সে কতটা busy


## Command 1: `lscpu` - CPU Hardware Info দেখুন

### এটা কী করে?

আপনার server-এর CPU সম্পর্কে সব hardware information দেখায়। কতটা core আছে, কত speed, কোন architecture - সব।

### Syntax:
```bash
lscpu
```

### Example Output:
```
Architecture:            x86_64
CPU op-mode(s):          32-bit, 64-bit
Byte Order:              Little Endian
CPU(s):                  4
On-line CPU(s) list:     0-3
Thread(s) per core:      2
Core(s) per socket:      2
Socket(s):               1
NUMA node(s):            1
Vendor ID:               GenuineIntel
Model name:              Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
CPU MHz:                 1800.000
L1d cache:               32K
L2 cache:                256K
L3 cache:                6144K
```

### প্রতিটা line মানে কী?

| Field | মানে |
|---|---|
| `Architecture` | CPU কোন type - x86_64 মানে 64-bit |
| `CPU(s)` | মোট logical CPU সংখ্যা (cores × threads) |
| `Core(s) per socket` | Physical core কতটা |
| `Thread(s) per core` | প্রতি core-এ কটা thread (Hyper-Threading) |
| `Model name` | CPU-র exact model |
| `CPU MHz` | বর্তমান speed |
| `L1/L2/L3 cache` | CPU-র fast memory (cache) |

বাস্তবতার ভিত্তিতে একটা কম্প্যারিজন 

- **Socket** = একটা রান্নাঘর
- **Core** = একজন বাবুর্চি
- **Thread** = একজন বাবুর্চি দুই হাতে কাজ করতে পারে
- তাই 2 core × 2 thread = 4 logical CPU

### DevOps-এ কখন ব্যবহার করবেন?
```bash
# নতুন server পেলে প্রথমেই দেখুন
lscpu

# শুধু core সংখ্যা দেখতে চাইলে
lscpu | grep "CPU(s)"

# Architecture দেখতে চাইলে (Docker image বা binary নামানোর আগে জানা দরকার)
lscpu | grep Architecture
```


## Command 2: `uptime` - System কতক্ষণ চলছে?

### এটা কী করে?
Server কতক্ষণ ধরে চলছে, কতজন user logged in আছে, এবং `load average` কত, এই তিনটা তথ্য একলাইনে দেখায়।

### Syntax:
```bash
uptime
```

### Example Output:
```
14:32:10 up 5 days, 3:21,  2 users,  load average: 0.45, 0.60, 0.58
```

### এটা পড়ার নিয়ম:

```
14:32:10          → বর্তমান সময়
up 5 days, 3:21   → server 5 দিন 3 ঘণ্টা 21 মিনিট ধরে চলছে
2 users           → এখন 2 জন logged in অবস্থায় আছে
load average:     → শেষ 1, 5, 15 মিনিটের load
0.45, 0.60, 0.58  → 1 মিনিট, 5 মিনিট, 15 মিনিট
```


## Load Average কী? (খুব গুরুত্বপূর্ণ!)

এটা Linux-এর সবচেয়ে important metric-এর একটা।


ধরুন একটা toll booth আছে রাস্তায়।

- **1টা lane** = 1 CPU core
- **গাড়ি** = process যেগুলো CPU চাইছে
- **Load average** = গড়ে কতটা গাড়ি line-এ দাঁড়িয়ে ছিল

```
Load Average = 1.0  →  ঠিক 1টা core সম্পূর্ণ busy (100% utilized)
Load Average = 0.5  →  1টা core মাত্র 50% busy
Load Average = 2.0  →  2টা core-এর কাজ আছে (যদি core 1টা হয় → overloaded!)
```

### সবচেয়ে গুরুত্বপূর্ণ নিয়ম:

```
Load Average / CPU সংখ্যা = per-CPU load

যদি result:
  < 1.0  →  ভালো, system comfortable
  = 1.0  →  ⚠️  সীমায় আছে
  > 1.0  →  🔴 Overloaded! processes queue-তে অপেক্ষা করছে
```

### উদাহরণ:

```
Server-এ 4টা CPU আছে।
Load average: 3.5, 2.8, 2.1

Per-CPU load = 3.5 / 4 = 0.875  → এখনো okay
```

```
Server-এ 2টা CPU আছে।
Load average: 4.2, 3.9, 3.5

Per-CPU load = 4.2 / 2 = 2.1  → 🔴 Overloaded!
```

### তিনটা সংখ্যা কী বলে?

```
load average: 0.45,  0.60,  0.58
               ↑       ↑      ↑
           1 মিনিট  5 মিনিট  15 মিনিট
```

- **1 মিনিট** সংখ্যা বড় কিন্তু **15 মিনিট** ছোট → হঠাৎ spike এসেছে, দেখুন কী হলো
- **15 মিনিট** সংখ্যা বড় কিন্তু **1 মিনিট** ছোট → সমস্যা কমছে, আগে বেশি ছিল


## Command 3: `mpstat` - CPU-র Detailed Statistics

### এটা কী করে?
প্রতিটা CPU core কতটুকু busy সেটা percentage-এ দেখায়। এটা `sysstat` package-এর অংশ।

### Install করে নিন (যদি না থাকে):
```bash
# Ubuntu/Debian
sudo apt install sysstat -y

# RHEL/CentOS
sudo yum install sysstat -y
```

### Syntax:
```bash
mpstat [options] [interval] [count]
```

| Part | মানে |
|---|---|
| `interval` | কত সেকেন্ড পর পর update দিবে |
| `count` | কতবার দেখাবে |

### Example 1: একবার সব CPU-র stats দেখুন
```bash
mpstat -P ALL
```

**Output:**
```
Linux 5.15.0  (devserver)   03/11/2026   _x86_64_    (4 CPU)

14:35:22  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %idle
14:35:22  all    5.23    0.00    1.45    0.32    0.00    0.10    0.00   92.90
14:35:22    0    6.10    0.00    1.60    0.20    0.00    0.05    0.00   92.05
14:35:22    1    4.80    0.00    1.30    0.40    0.00    0.15    0.00   93.35
14:35:22    2    5.50    0.00    1.55    0.35    0.00    0.10    0.00   92.50
14:35:22    3    4.52    0.00    1.35    0.33    0.00    0.10    0.00   93.70
```

### প্রতিটা Column মানে কী?

| Column | পুরো নাম | মানে |
|---|---|---|
| `%usr` | User | Normal programs CPU কতটুকু নিচ্ছে |
| `%nice` | Nice | Low priority programs-এর CPU usage |
| `%sys` | System | Kernel (OS নিজে) কতটুকু CPU নিচ্ছে |
| `%iowait` | I/O Wait | Disk/Network-এর জন্য CPU অপেক্ষা করছে |
| `%irq` | Interrupt | Hardware interrupts handle করতে |
| `%soft` | Soft IRQ | Software interrupts |
| `%steal` | Steal | Virtual machine হলে host কতটা CPU চুরি করছে |
| `%idle` | Idle | CPU কতটা ফাঁকা আছে |

### Important Signals:

```
%idle কম (< 10%)     → 🔴 CPU heavily loaded
%iowait বেশি (> 30%) → 🔴 Disk/Network bottleneck আছে
%steal বেশি (> 10%)  → 🔴 Cloud VM-এ noisy neighbor সমস্যা
%sys বেশি (> 20%)    → 🔴 Kernel-level সমস্যা
```

### Example 2: প্রতি 2 সেকেন্ডে 5 বার দেখাও
```bash
mpstat 2 5
```

**Output:**
```
14:36:00  CPU   %usr  %sys  %iowait  %idle
14:36:02  all   5.23  1.45     0.32  92.90
14:36:04  all   8.10  2.00     0.10  89.80   ← হঠাৎ বাড়লো
14:36:06  all   6.50  1.80     0.20  91.50
14:36:08  all   5.80  1.50     0.30  92.40
14:36:10  all   5.20  1.40     0.35  93.05
```

### Example 3: শুধু একটা specific CPU core দেখুন
```bash
mpstat -P 0    # শুধু CPU 0 দেখুন
mpstat -P 1    # শুধু CPU 1 দেখুন
```


## Bonus: `watch` দিয়ে Real-time Monitor

```bash
# প্রতি 2 সেকেন্ডে mpstat refresh হবে
watch -n 2 mpstat
```


## Bonus: `/proc/cpuinfo` - Raw CPU Info

Linux-এ সব hardware info আসলে `/proc` filesystem-এ থাকে।

```bash
cat /proc/cpuinfo
```

**Output (একটা core-এর জন্য):**
```
processor   : 0
vendor_id   : GenuineIntel
cpu family  : 6
model name  : Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
cpu MHz     : 1800.000
cache size  : 6144 KB
physical id : 0
siblings    : 4
core id     : 0
cpu cores   : 2
```

```bash
# কতটা processor আছে তা দেখায়
grep -c "processor" /proc/cpuinfo

# শুধু model name দেখুন
grep "model name" /proc/cpuinfo | uniq
```


## Real-World DevOps Scenario

### Scenario: Production server হঠাৎ slow হয়ে গেছে!

```bash
# Step 1: Load average দেখুন
uptime
# Output: load average: 8.5, 7.2, 4.1
# ⚠️ 4-core server-এ load 8.5 → serious problem!

# Step 2: কোন CPU-তে কী হচ্ছে দেখুন
mpstat -P ALL 1 3

# Step 3: %iowait বেশি কিনা দেখুন
# %iowait > 30% → Disk সমস্যা (Lesson 8-এ iostat দিয়ে গভীরে যাবো)

# Step 4: %usr বেশি হলে → কোন process CPU নিচ্ছে?
top    # অথবা ps aux --sort=-%cpu | head -10
```


## 📝 Quick Summary

- `lscpu` → CPU hardware info দেখায় (`cores`, `threads`, `model`, `speed`)
- `uptime` → server কতক্ষণ চলছে + `load average` দেখায়
- `Load Average` → CPU-র queue কতটা ভারী - `load / cpu_count < 1` হলে ভালো
- `mpstat` → প্রতিটা CPU core-এর detailed % usage দেখায়
- `%idle` কম হলে → CPU busy; `%iowait` বেশি হলে → Disk bottleneck
- `/proc/cpuinfo` → raw CPU hardware data


## 🏋️ Practice Tasks

এই তিনটা কাজ নিজে করে দেখুন:

**Task 1:**
```bash
lscpu
```

আপনার system-এ কতটা logical CPU আছে? Model name কী? কমেন্ট করুন

**Task 2:**
```bash
uptime
```

আপনার load average কত? CPU সংখ্যা দিয়ে ভাগ করুন। System কি overloaded?

**Task 3:**
```bash
mpstat -P ALL 2 3
```
Run করুন এবং দেখুন, `%idle` কত? কোনো core কি অন্যটার চেয়ে বেশি busy?

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 8: Disk I/O Monitoring**

`iostat`, `iotop`, `df`, `du` দিয়ে শিখব disk কতটা busy, কোথায় কত space আছে, এবং কোন process সবচেয়ে বেশি disk read/write করছে। DevOps-এ disk bottleneck একটা common সমস্যা, সেটা ধরার পুরো technique শিখব! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../06-Memory-Monitoring">← Memory Monitoring</a>
    </td>
    <td align="right">
      <a href="../08-Disk-IO-Monitoring">Disk IO Monitoring →</a>
    </td>
  </tr>
</table>
