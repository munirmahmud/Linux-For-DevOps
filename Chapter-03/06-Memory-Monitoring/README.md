# Chapter 3 - Lesson 6: Memory Monitoring

**Chapter 3 | Lesson 6 of 10**


## Memory কী এবং কেন Monitor করতে হয়?

আপনার Linux server একটা অফিসের মতো। সেই অফিসে:

- **RAM** = আপনার **কাজের টেবিল** - যেখানে আপনি এখন কাজ করছেন
- **Swap** = আপনার **পাশের ড্রয়ার** - টেবিল ভরে গেলে কিছু জিনিস এখানে রাখেন
- **Disk (Storage)** = **গুদামঘর** - সব কিছু permanently রাখা হয়

যদি টেবিল (RAM) ভরে যায়, সব কিছু slow হয়ে যায়।  
DevOps Engineer হিসেবে আমাদের কাজ হলো টেবিল কে কতটুকু ব্যবহার করছে সেটা সবসময় নজরে রাখা।


## Tool 1: `free` - RAM ও Swap এর Overview

### এটা কী করে?
System এ মোট কত RAM আছে, কতটুকু ব্যবহার হচ্ছে, কতটুকু খালি আছে, এক নজরে দেখায়।


### Syntax:
```bash
free [options]
```


### Common Options:

| Option | মানে কী |
|--------|---------|
| `-h` | Human-readable (MB, GB তে দেখায়) |
| `-m` | Megabytes এ দেখায় |
| `-g` | Gigabytes এ দেখায় |
| `-s 2` | প্রতি 2 সেকেন্ডে auto-refresh |
| `-t` | Total row যোগ করে |


### Example:
```bash
free -h
```

### Expected Output:
```
              total        used        free      shared  buff/cache   available
Mem:           15Gi       4.2Gi       6.8Gi       312Mi       4.1Gi       10.5Gi
Swap:         2.0Gi          0B       2.0Gi
```


### Output বোঝা (Column by Column):

```
              total    used    free    shared   buff/cache   available
Mem:           15Gi    4.2Gi   6.8Gi   312Mi     4.1Gi        10.5Gi
```

| Column | মানে কী |
|--------|---------|
| `total` | মোট RAM (15 GB) |
| `used` | এখন actually ব্যবহার হচ্ছে (4.2 GB) |
| `free` | একদম খালি (কোনো কাজেই লাগছে না) |
| `shared` | Multiple process share করছে (tmpfs ইত্যাদি) |
| `buff/cache` | OS নিজে cache হিসেবে ব্যবহার করছে |
| `available` | নতুন process চালু করতে পারবে কতটুকু দিয়ে |

**Important Tip:** `free` column দেখে ভয় পাওয়া যাবে না!  

> Linux সবসময় RAM কে cache হিসেবে ব্যবহার করে performance বাড়াতে।  
> আসল সংখ্যা হলো **`available`**, এটা দিয়ে বোঝা যায় কতটুকু RAM বাকি আছে।


### Swap Row:
```
Swap:   2.0Gi    0B    2.0Gi
```
- Swap ব্যবহার **0** মানে ভালো, RAM এখনও যথেষ্ট আছে
- Swap ব্যবহার বেশি হলে = **RAM এর সমস্যা আছে**, performance কমে যাবে


### Live Monitoring (প্রতি 2 সেকেন্ডে):
```bash
free -h -s 2
```
> এটা বন্ধ করতে `Ctrl + C` চাপুন


### DevOps Use Case:
```bash
# Production server এ memory check করা
free -h

# Script এ available memory চেক করা
available=$(free -m | awk '/^Mem:/{print $7}')
echo "Available Memory: ${available}MB"

# যদি memory 500MB এর নিচে নামে alert দাও
if [ "$available" -lt 500 ]; then
    echo "WARNING: Low memory! Only ${available}MB available"
fi
```


## Tool 2: `vmstat` - Virtual Memory Statistics

### এটা কী করে?
`free` এর চেয়ে বেশি detail দেয়। শুধু memory না, **CPU, I/O, এবং system activity** একসাথে দেখায়।

> `free` যদি হয় আপনার অফিসের একটা snapshot ফটো। তাহলে `vmstat` হলো **CCTV ফুটেজ**, সব কিছু live দেখা যায়।


### Syntax:
```bash
vmstat [options] [delay] [count]
```

| Part | মানে কী |
|------|---------|
| `delay` | কত সেকেন্ড পর পর দেখাবে |
| `count` | কতবার দেখাবে |


### Example 1 - একবার দেখা:
```bash
vmstat
```

### Expected Output:
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 6900000 210000 4100000  0    0    10     5  200  400  5  2 92  1  0
```


### Output বোঝা:

#### `procs` Section:
| Column | মানে কী |
|--------|---------|
| `r` | Run queue - CPU এর জন্য অপেক্ষা করছে কতটি process |
| `b` | Blocked - I/O এর জন্য আটকে আছে কতটি process |

> `r` এর সংখ্যা CPU core এর চেয়ে বেশি হলে = **CPU bottleneck**


#### `memory` Section:
| Column | মানে কী |
|--------|---------|
| `swpd` | Swap এ কতটুকু ব্যবহার হচ্ছে (KB) |
| `free` | খালি RAM (KB) |
| `buff` | Buffer memory |
| `cache` | Cache memory |


#### `swap` Section:
| Column | মানে কী |
|--------|---------|
| `si` | Swap In - Swap থেকে RAM এ আনা হচ্ছে (KB/s) |
| `so` | Swap Out - RAM থেকে Swap এ পাঠানো হচ্ছে (KB/s) |

> **`si` বা `so` বেশি হলে, বড় সমস্যা!** RAM কম পড়ছে।


#### `io` Section:
| Column | মানে কী |
|--------|---------|
| `bi` | Blocks In - Disk থেকে পড়া হচ্ছে |
| `bo` | Blocks Out - Disk এ লেখা হচ্ছে |


#### `cpu` Section:
| Column | মানে কী |
|--------|---------|
| `us` | User processes CPU ব্যবহার % |
| `sy` | System (kernel) CPU ব্যবহার % |
| `id` | Idle - CPU খালি আছে % |
| `wa` | Wait - I/O এর জন্য CPU অপেক্ষা করছে % |

> `id` 90+ মানে সব ঠিকঠাক। `wa` বেশি হলে **Disk I/O সমস্যা।**


### Example 2 - প্রতি 2 সেকেন্ডে, 5 বার দেখা:
```bash
vmstat 2 5
```

### Expected Output:
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 6900000 210000 4100000   0    0    10     5  200  400  5  2 92  1  0
 0  0      0 6895000 210000 4100000   0    0     0     4  180  390  3  1 95  1  0
 1  0      0 6890000 210000 4100000   0    0     0     8  220  410  6  2 91  1  0
 0  0      0 6888000 210000 4100000   0    0     0     2  190  380  2  1 96  1  0
 0  0      0 6885000 210000 4100000   0    0     0     3  200  395  4  1 94  1  0
```

> **প্রথম লাইনটা** system boot থেকে এখন পর্যন্ত average।  
> **বাকি লাইনগুলো** real-time data।


### Example 3 - Memory Statistics বিস্তারিত:
```bash
vmstat -s
```

### Expected Output:
```
     16331264 K total memory
      4300000 K used memory
      4100000 K active memory
      2800000 K inactive memory
      6900000 K free memory
       210000 K buffer memory
      4100000 K swap cache
      2097148 K total swap
            0 K used swap
      2097148 K free swap
        ...
```


## Tool 3: `/proc/meminfo` - Kernel এর Memory রিপোর্ট

### এটা কী করে?
Linux Kernel নিজে memory সম্পর্কে যা জানে সব এখানে লেখা থাকে।  
`free` এবং `vmstat` এর data এখান থেকেই আসে।

> `/proc/meminfo` হলো মূল রেজিস্টার খাতা। `free` এবং `vmstat` সেই খাতা দেখে আপনাকে সুন্দরভাবে রিপোর্ট দেয়।


### Example - পুরো ফাইল দেখা:
```bash
cat /proc/meminfo
```

### Expected Output:
```
MemTotal:       16331264 kB
MemFree:         6900000 kB
MemAvailable:   10500000 kB
Buffers:          210000 kB
Cached:          4100000 kB
SwapCached:            0 kB
Active:          4100000 kB
Inactive:        2800000 kB
SwapTotal:       2097148 kB
SwapFree:        2097148 kB
Dirty:              1200 kB
Writeback:             0 kB
AnonPages:       2500000 kB
Mapped:           800000 kB
Shmem:            312000 kB
HugePages_Total:       0
HugePages_Free:        0
...
```


### Important Fields:

| Field | মানে কী |
|-------|---------|
| `MemTotal` | মোট RAM |
| `MemFree` | একদম খালি RAM |
| `MemAvailable` | নতুন process এর জন্য পাওয়া যাবে |
| `Buffers` | Disk metadata cache |
| `Cached` | File content cache |
| `SwapTotal` | মোট Swap space |
| `SwapFree` | খালি Swap space |
| `Dirty` | Disk এ লেখার জন্য অপেক্ষায় আছে |
| `HugePages` | Large memory page (database server এ দরকার হয়) |


### Specific Field দেখা:
শুধু Total আর Available Memory দেখা
```bash
grep -E "MemTotal|MemAvailable|SwapFree" /proc/meminfo
```

### Expected Output:
```
MemTotal:       16331264 kB
MemAvailable:   10500000 kB
SwapFree:        2097148 kB
```


## Tool 4: `smem` - Process অনুযায়ী Memory ব্যবহার

### এটা কী করে?
কোন Process কতটুকু memory নিচ্ছে সেটা detail এ দেখায়।

> অফিসে কোন employee টেবিলের কতটুকু জায়গা নিচ্ছে সেটার list।


### Installation (যদি না থাকে):
```bash
# Ubuntu/Debian
sudo apt install smem -y

# RHEL/CentOS
sudo yum install smem -y
```


### Key Memory Metrics যা `smem` দেখায়:

| Term | মানে কী |
|------|---------|
| **RSS** (Resident Set Size) | Process এখন RAM এ কতটুকু নিচ্ছে |
| **PSS** (Proportional Set Size) | Shared memory ভাগ করে হিসাব করলে কতটুকু |
| **USS** (Unique Set Size) | শুধু এই process এর নিজের memory |

> **USS সবচেয়ে accurate** - এটা দিয়ে বোঝা যায় কোন process আসলে কতটুকু নিচ্ছে।


### Example - সব process এর memory দেখা:
```bash
smem -r -s rss
```

### Expected Output:
```
  PID User     Command                         Swap      USS      PSS      RSS
 1234 root     /usr/bin/python3 app.py            0   150000   155000   200000
  567 www-data nginx: worker process             0    50000    55000    80000
  890 mysql    /usr/sbin/mysqld                  0   300000   310000   400000
  ...
```


### Example - Top Memory consumers দেখা:
```bash
smem -r -s uss | head -10
```

### Example - Percentage হিসেবে দেখা:
```bash
smem -p -r -s rss | head -10
```


## সব Tool এর তুলনা:

| Tool | কী দেখায় | কখন ব্যবহার করবে |
|------|----------|-----------------|
| `free -h` | Quick overall RAM/Swap overview | Daily check, scripting |
| `vmstat 2 5` | Memory + CPU + I/O একসাথে | Performance issue investigate |
| `/proc/meminfo` | Raw kernel memory data | Deep dive, scripting |
| `smem` | Process-wise memory breakdown | কোন process বেশি Memory খাচ্ছে খুঁজতে |


## Real DevOps Scenario - Memory Problem Diagnose করা

```bash
# Step 1: Overall memory দেখুন
free -h

# Step 2: Swap ব্যবহার হচ্ছে কিনা দেখুন
vmstat 1 5

# Step 3: কোন process বেশি memory নিচ্ছে দেখুন
ps aux --sort=-%mem | head -10

# Step 4: Detail এ দেখুন
smem -r -s uss | head -10

# Step 5: Kernel এর রিপোর্ট চেক করুন
grep -E "MemAvailable|SwapFree" /proc/meminfo
```


## 📝 Quick Summary

- **`free -h`** - RAM ও Swap এর সহজ overview দেখায়
- **`available` column** দেখুন, `free` column না - এটাই আসল available memory
- **`vmstat 2 5`** - Memory, CPU, I/O সব একসাথে monitor করা যায়
- **`si/so` বেশি হলে** = RAM কম পড়ছে, Swap বেশি ব্যবহার হচ্ছে, বিপদ!
- **`/proc/meminfo`** - Kernel এর raw memory data, scripting এ কাজে আসে
- **`smem`** - কোন process কতটুকু memory নিচ্ছে সেটা accurately দেখায়


## 🏋️ Practice Tasks

**Task 1:**
```bash
# এই command গুলো run করুন এবং output বোঝার চেষ্টা করুন
free -h
free -m -t
```

`available` column এর সংখ্যা নোট করুন। এটাই আপনার usable RAM।


**Task 2:**
```bash
# vmstat দিয়ে 3 সেকেন্ড পর পর 4 বার দেখুন
vmstat 3 4
```
`si` এবং `so` column দেখুন। দুটোই 0 হওয়া উচিত। `id` (idle CPU) কত %?


**Task 3:**
```bash
# একটা ছোট script লিখুন যেটা available memory চেক করবে
# এবং 1000MB এর নিচে নামলে warning দিবে
available=$(free -m | awk '/^Mem:/{print $7}')
echo "Available Memory: ${available}MB"

if [ "$available" -lt 1000 ]; then
    echo "WARNING: Low Memory!"
else
    echo "Memory OK"
fi
```


## ⏭️ What's Next?

**Chapter 3 - Lesson 7: CPU Monitoring**

`lscpu`, `mpstat`, `uptime`, `Load Average` কী এবং কীভাবে CPU performance measure করতে হয়, সব শিখবো পরের lesson এ! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../05-Process-Priority">← Process Priority</a>
    </td>
    <td align="right">
      <a href="../07-CPU-Monitoring">CPU Monitoring →</a>
    </td>
  </tr>
</table>
