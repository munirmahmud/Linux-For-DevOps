# Chapter 3 - Lesson 9: System-wide Performance

**Chapter 3 | Lesson 9 of 10**


## 🎯 এই Lesson-এ কী শিখবে?

এই lesson-এ আমরা শিখবো তিনটি powerful tool যেগুলো দিয়ে আপনি **পুরো system-এর performance একসাথে** monitor করতে পারবেন:

| Tool | কাজ |
|------|-----|
| `dstat` | Real-time CPU, Memory, Disk, Network একসাথে দেখো |
| `sar` | Historical performance data collect ও analyze করো |
| `glances` | সব কিছু একটা beautiful dashboard-এ দেখো |

**Real-world Analogy:**
> মনে করেন আপনার একটা গাড়ি আছে।
> - `top`/`htop` = শুধু **এখন** speedometer দেখা
> - `dstat` = **সব gauge একসাথে** (speed + fuel + engine temp) real-time দেখা
> - `sar` = গাড়ির **পুরো trip log** - কোন সময় কত fuel খেয়েছে রেকর্ড দেখা
> - `glances` = **Full car dashboard** - সুন্দর করে সব কিছু সাজানো


## PART 1: `dstat` - Real-time All-in-One Monitor

### `dstat` কী?

`dstat` হলো একটা versatile tool যেটা **CPU, Disk, Network, Memory, Paging** সব কিছু **একই সাথে, একই line-এ** দেখায়।

পুরনো দিনে আলাদা আলাদা tool চালাতে হতো:
- CPU দেখতে → `vmstat`
- Disk দেখতে → `iostat`
- Network দেখতে → `ifstat`

`dstat` এই সব একসাথে করে!


### Installation

```bash
# Ubuntu/Debian
sudo apt install dstat -y

# RHEL/CentOS/Rocky
sudo yum install dstat -y
# অথবা
sudo dnf install dstat -y
```

> **Note:** কিছু নতুন distro-তে `dstat` এর বদলে `dool` use হয়। কিন্তু syntax একই।


### Basic Usage

```bash
dstat
```

**Expected Output:**
```
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai stl| read  writ| recv  send|  in   out | int   csw
  2   1  97   0   0|  45k  123k|   0     0 |   0     0 | 287   512
  1   0  99   0   0|   0    16k| 124B  892B|   0     0 | 243   445
  3   1  96   0   0|   0     0 | 240B  1.2k|   0     0 | 312   601
```


### Output বোঝা - Column by Column

#### CPU Section: `----total-cpu-usage----`
| Column | মানে |
|--------|------|
| `usr` | User processes CPU % (তোমার applications) |
| `sys` | Kernel/System CPU % |
| `idl` | Idle % - যত বেশি ততো ভালো |
| `wai` | I/O wait % - disk-এর জন্য CPU অপেক্ষা করছে |
| `stl` | Steal time % - VM-এ hypervisor CPU নিচ্ছে |

#### Disk Section: `-dsk/total-`
| Column | মানে |
|--------|------|
| `read` | প্রতি second-এ disk থেকে কত data read হচ্ছে |
| `writ` | প্রতি second-এ disk-এ কত data write হচ্ছে |

#### Network Section: `-net/total-`
| Column | মানে |
|--------|------|
| `recv` | Network-এ কত data receive হচ্ছে (incoming) |
| `send` | Network-এ কত data send হচ্ছে (outgoing) |

#### System Section: `---system--`
| Column | মানে |
|--------|------|
| `int` | Interrupts per second (hardware signals) |
| `csw` | Context switches per second (CPU কত বার task switch করছে) |


### dstat-এর Useful Options

#### Option 1: Memory সহ দেখো

```bash
dstat -m
```

```
------memory-usage-----
 used  buff  cach  free
1.2G  234M  1.8G  512M
1.2G  234M  1.8G  512M
```

#### Option 2: নির্দিষ্ট interval ও count

```bash
# প্রতি 2 second-এ update, মোট 5 বার
dstat 2 5
```

```
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai stl| read  writ| recv  send|  in   out | int   csw
  2   1  97   0   0|  45k  123k|   0     0 |   0     0 | 287   512
  1   0  99   0   0|   0    16k| 124B  892B|   0     0 | 243   445
  # ... মোট 5 line
```

> `dstat 5` মানে প্রতি 5 second-এ update হবে - indefinitely।

#### Option 3: সব কিছু একসাথে

```bash
dstat -cdngm
```

| Flag | মানে |
|------|------|
| `-c` | CPU |
| `-d` | Disk |
| `-n` | Network |
| `-g` | Paging (swap in/out) |
| `-m` | Memory |

#### Option 4: Top Process দেখা

```bash
dstat --top-cpu
```

```
----total-cpu-usage---- ---most-expensive---
usr sys idl wai stl|  cpu process
  5   2  93   0   0|nginx        2.1
  8   3  89   0   0|mysql        5.3
```

কোন process সবচেয়ে বেশি CPU খাচ্ছে তা real-time দেখতে পারবেন!

```bash
# সবচেয়ে বেশি memory খাচ্ছে কে?
dstat --top-mem

# সবচেয়ে বেশি disk I/O করছে কে?
dstat --top-io
```

#### Option 5: Specific Network Interface

```bash
# শুধু eth0 interface-এর network দেখুন
dstat -n -N eth0

# একাধিক interface
dstat -n -N eth0,lo
```

#### Option 6: Output File-এ Save করা

```bash
dstat --output /tmp/dstat_report.csv 2 10
```

এটা CSV file-এ save হবে, পরে Excel-এ open করে analyze করতে পারবেন! DevOps-এ খুবই কাজে লাগে।


### DevOps Real-World Use Cases

```bash
# Scenario 1: Production server slow কেন?
# CPU + Disk + Network + Top process একসাথে দেখুন
dstat -cdnm --top-cpu 2

# Scenario 2: Network bottleneck check
dstat -n --top-net 1

# Scenario 3: Database server I/O check
dstat -d --top-io 2

# Scenario 4: 1 ঘণ্টার data collect করুন (log analysis-এর জন্য)
dstat --output /var/log/perf_$(date +%Y%m%d_%H%M).csv 5 720
# প্রতি 5 second × 720 = 1 ঘণ্টা
```


## PART 2: `sar` - System Activity Reporter

### `sar` কী?

`sar` হলো **historical performance data** collect করার সেরা tool।

> **Analogy:** `dstat` হলো **live TV** এখন কী হচ্ছে তা দেখা যায়। `sar` হলো **DVR recording** আগে কী হয়েছিল সেটা দেখা।

DevOps-এ এটা অমূল্য কারণ:
- রাত 3টায় server slow হয়েছিল - কিন্তু আপনি তখন ঘুমাচ্ছিলেন!
- সকালে উঠে `sar` দিয়ে দেখবেন তখন ঠিক কী হয়েছিল


### Installation

`sar` আসে `sysstat` package-এর সাথে:

```bash
# Ubuntu/Debian
sudo apt install sysstat -y

# RHEL/CentOS
sudo yum install sysstat -y
```

#### sysstat Enable করা

```bash
# Data collection service চালু করুন
sudo systemctl enable sysstat
sudo systemctl start sysstat

# Ubuntu-তে এটাও করতে হতে পারে
sudo nano /etc/default/sysstat
# ENABLED="false" → ENABLED="true" করুন

sudo systemctl restart sysstat
```

> এরপর থেকে `sysstat` প্রতি **10 minutes** পর পর automatically data collect করবে এবং `/var/log/sysstat/` বা `/var/log/sa/` directory-তে save করবে।


### Basic Usage

#### আজকের CPU data দেখা

```bash
sar
# OR
sar -u
```

**Expected Output:**
```
Linux 5.15.0-91-generic (myserver)    03/11/2026    _x86_64_    (2 CPU)

12:00:01 AM     CPU     %user   %system   %iowait   %idle
12:10:01 AM     all      2.34      0.87      0.12    96.67
12:20:01 AM     all      1.98      0.76      0.08    97.18
12:30:01 AM     all     15.43      2.34      0.45    81.78  ← এখানে CPU বেড়েছিল!
12:40:01 AM     all      2.11      0.81      0.10    97.08
...
Average:        all      3.21      0.94      0.15    95.70
```

শেষে **Average** line দেয়, পুরো দিনের গড়!


### sar-এর সব Important Options

#### CPU Monitoring

```bash
# আজকের সব CPU data
sar -u

# প্রতি 2 second-এ 5 বার live CPU
sar -u 2 5

# সব individual CPU core দেখো
sar -P ALL
```

**Output (multi-core):**
```
12:00:01 AM     CPU     %user   %system   %iowait   %idle
12:10:01 AM       0      3.21      1.12      0.15    95.52
12:10:01 AM       1      1.45      0.67      0.09    97.79
```

#### Memory Monitoring

```bash
sar -r
```

```
12:00:01 AM  kbmemfree  kbmemused  %memused  kbbuffers  kbcached
12:10:01 AM     524288    1572864     75.00     234567    456789
12:20:01 AM     512000    1585152     75.59     235000    457000
```

| Column | মানে |
|--------|------|
| `kbmemfree` | Free memory (KB) |
| `kbmemused` | Used memory (KB) |
| `%memused` | Memory usage percentage |
| `kbbuffers` | Buffer cache |
| `kbcached` | Page cache |

#### Disk I/O Monitoring

```bash
sar -d
```

```
12:00:01 AM  DEV    tps   rkB/s   wkB/s   areq-sz
12:10:01 AM  sda  12.45  234.56   89.23     26.12
```

| Column | মানে |
|--------|------|
| `tps` | Transactions per second |
| `rkB/s` | KB read per second |
| `wkB/s` | KB written per second |

#### Network Monitoring

```bash
sar -n DEV
```

```
12:00:01 AM  IFACE   rxpck/s  txpck/s   rxkB/s   txkB/s
12:10:01 AM  eth0     123.45   89.23     456.78   234.56
12:10:01 AM  lo         2.34    2.34       1.23     1.23
```

#### Swap / Paging

```bash
sar -S    # Swap usage
sar -B    # Paging statistics
```


### আগের দিনের Data দেখা

`sar` data `/var/log/sysstat/` বা `/var/log/sa/` directory-তে `saXX` নামে save হয় (XX = দিনের তারিখ)।

```bash
# আজকের data file দেখা
ls /var/log/sysstat/
# অথবা
ls /var/log/sa/

# আগের দিনের (যেমন ১০ তারিখের) data
sar -f /var/log/sysstat/sa10

# নির্দিষ্ট দিনের CPU data
sar -u -f /var/log/sa/sa10

# নির্দিষ্ট সময়ের data দেখা
sar -u -s 02:00:00 -e 04:00:00
# রাত ২টা থেকে ৪টার মধ্যে কী হয়েছিল?
```


### DevOps Real-World Scenario

```bash
# Scenario: "রাত 3টায় server crash করেছিল কেন?"

# Step 1: সেই রাতের CPU দেখুন
sar -u -s 02:30:00 -e 03:30:00 -f /var/log/sa/sa10

# Step 2: Memory দেখুন সেই সময়ে
sar -r -s 02:30:00 -e 03:30:00 -f /var/log/sa/sa10

# Step 3: Disk I/O দেখুন
sar -d -s 02:30:00 -e 03:30:00 -f /var/log/sa/sa10

# Step 4: Network দেখুন (DDoS attack ছিল?)
sar -n DEV -s 02:30:00 -e 03:30:00 -f /var/log/sa/sa10
```

এভাবে আপনি **রাতের ঘটনা সকালে investigate** করতে পারো! এটাই `sar`-এর আসল শক্তি।


### sar Quick Reference Table

| Command | কী দেখায় |
|---------|----------|
| `sar -u` | CPU usage |
| `sar -r` | Memory usage |
| `sar -d` | Disk I/O |
| `sar -n DEV` | Network traffic |
| `sar -S` | Swap usage |
| `sar -B` | Paging |
| `sar -q` | Load average & run queue |
| `sar -W` | Swap in/out |
| `sar 2 5` | Live: প্রতি 2s, 5 বার |
| `sar -f /var/log/sa/sa10` | ১০ তারিখের data |
| `sar -s 02:00 -e 04:00` | নির্দিষ্ট সময়ের data |


## PART 3: `glances` - The Beautiful All-in-One Dashboard

### `glances` কী?

`glances` হলো Python দিয়ে তৈরি একটা **modern, beautiful** system monitor। এটা এক screen-এ সব কিছু দেখায়:

- CPU, Memory, Swap
- Disk I/O, Network
- Running Processes
- Alerts & Warnings (automatic!)
- Docker containers (যদি থাকে)

> **Analogy:** `htop` যদি হয় একটা সাধারণ cockpit, তাহলে `glances` হলো **modern fighter jet-এর full HUD display**।


### Installation

```bash
# Ubuntu/Debian
sudo apt install glances -y

# pip দিয়ে (সব distro-তে)
pip3 install glances --break-system-packages

# Docker দিয়ে
docker run -d --restart="always" \
  -p 61208-61209:61208-61209 \
  -e GLANCES_OPT="-w" \
  --pid host \
  --network host \
  --name glances \
  nicolargo/glances
```


### Basic Usage

```bash
glances
```

**এরকম একটা screen দেখবে:**
```
myserver (Ubuntu 22.04 64bit / Linux 5.15.0)                   Uptime: 5 days, 3:42:11

CPU        [|||       ]  28.5%    MEM        [|||||||   ]  72.3%    SWAP      [          ]   0.0%
user:       25.1%        total:    3.85G               total:    2.00G
system:      3.2%        used:     2.79G               used:     0.00G
idle:       71.5%        free:     1.06G               free:     2.00G

NETWORK              Rx/s          Tx/s         DISK I/O           R/s            W/s
eth0                1.2 Mb        456 Kb        sda              1.2 MB         456 KB

PROCESSES (sorted by CPU%)
CPU%  MEM%   PID  USER    NI  S   TIME+      IOR/s  IOW/s  Command
25.1   3.2   1234  www-d    0  R  1:23.45    1.2MB    0     /usr/sbin/apache2
 3.2   1.1   5678  mysql    0  S  0:45.12      0    256KB   /usr/sbin/mysqld
```


### glances Keyboard Shortcuts (Interactive)

glances চলার সময় এই keys press করে দেখুন:

| Key | কাজ |
|-----|-----|
| `q` বা `Ctrl+C` | বের হয়ে যাও |
| `h` | Help screen |
| `c` | CPU % অনুযায়ী sort |
| `m` | Memory % অনুযায়ী sort |
| `i` | I/O rate অনুযায়ী sort |
| `n` | Network অনুযায়ী sort |
| `d` | Disk I/O section দেখাও/লুকাও |
| `f` | File system section |
| `1` | CPU core গুলো আলাদা দেখাও |
| `t` | Network interface দেখাও |
| `e` | Sensors দেখাও (temperature) |
| `/` | Process filter করো (নাম দিয়ে) |
| `3` | Quick Look mode |


### glances Modes

#### Mode 1: Normal (Terminal)

```bash
glances
```

#### Mode 2: Web Server Mode

```bash
glances -w
# Browser-এ যাও: http://your-server-ip:61208
```

এটা দিয়ে **remote server** browser থেকে monitor করতে পারবে! DevOps-এ অনেক কাজে লাগে।

#### Mode 3: Client-Server Mode

```bash
# Server-এ (যে machine monitor করবে)
glances -s

# Client-এ (যেখান থেকে দেখবে)
glances -c 192.168.1.100
```

#### Mode 4: CSV/File Output

```bash
# CSV-তে export
glances --export csv --export-csv-file /tmp/glances_report.csv

# JSON output
glances --export json
```

#### Mode 5: Docker Monitoring

```bash
glances
# Docker section automatically দেখাবে যদি Docker installed থাকে
```

#### Mode 6: Specific Refresh Rate

```bash
# প্রতি 5 second-এ refresh
glances -t 5
```


### glances Alert System

`glances` automatically **color-coded alerts** দেয়:

| Color | মানে | Threshold |
|-------|------|-----------|
| 🟢 Green | Normal | OK |
| 🔵 Blue | Careful | >50% |
| 🟡 Yellow | Warning | >70% |
| 🔴 Red | Critical | >90% |

Custom threshold set করতে পারেন `/etc/glances/glances.conf` file-এ।


### তিনটি Tool-এর তুলনা

| Feature | dstat | sar | glances |
|---------|-------|-----|---------|
| Real-time | ✅ | ✅ (limited) | ✅ |
| Historical Data | ❌ | ✅✅ | ❌ |
| Beautiful UI | ❌ | ❌ | ✅✅ |
| Web Interface | ❌ | ❌ | ✅ |
| Script-friendly | ✅✅ | ✅✅ | ✅ |
| Per-process info | Limited | ❌ | ✅ |
| Docker support | ❌ | ❌ | ✅ |
| Remote monitoring | ❌ | ❌ | ✅ |
| Install size | Small | Small | Medium |
| **Best for** | Quick CLI check | Post-incident analysis | Full monitoring |


## Practical DevOps Scenarios

### Scenario 1: Production server হঠাৎ slow হয়েছে

```bash
# Step 1: Quick overview
glances
# OR
dstat -cdnm --top-cpu 2

# Step 2: কোন process অপরাধী?
dstat --top-cpu --top-mem 1

# Step 3: Network spike আছে কিনা দেখুন
dstat -n -N eth0 2
```

### Scenario 2: রাতের incident investigate করুন

```bash
# sar দিয়ে রাত ২-৪টার data দেখুন
sar -u -s 02:00:00 -e 04:00:00
sar -r -s 02:00:00 -e 04:00:00
sar -d -s 02:00:00 -e 04:00:00
```

### Scenario 3: 1 ঘণ্টার performance baseline তৈরি করা

```bash
# dstat দিয়ে log collect করা
dstat --output /var/log/perf_baseline.csv 10 360
# প্রতি 10 second × 360 = 1 hour

# sar automatically করে, শুধু দেখুন
sar -u >> /tmp/cpu_report.txt
```

### Scenario 4: Remote server monitor করা (glances)

```bash
# Remote server-এ
ssh user@remote-server
glances -w --port 61208 &

# Local machine-এ browser খোলো
# http://remote-server-ip:61208
```


## 📝 Quick Summary

- **`dstat`** → Real-time, all-in-one terminal monitor। Script ও log collection-এর জন্য perfect।
- **`sar`** → `sysstat` package-এর অংশ। Historical data save করে। Post-incident analysis-এর জন্য অপরিহার্য।
- **`glances`** → Beautiful dashboard। Web mode, remote monitoring, Docker support সহ।
- তিনটি একসাথে ব্যবহার করলে আপনি **complete observability** পাবেন।
- DevOps-এ **`sar`** সবচেয়ে গুরুত্বপূর্ণ কারণ এটা **আগের data** দেখাতে পারে।


## 🏋️ Practice Tasks

> এগুলো নিজে try করুন:

**Task 1:** `dstat` দিয়ে CPU, Memory, Network, Disk একসাথে দেখাও এবং কোন process সবচেয়ে বেশি CPU নিচ্ছে সেটাও দেখাও। প্রতি 2 second-এ 10 বার update নাও।

**Task 2:** `sysstat` install ও enable করো। তারপর `sar -u 2 5` দিয়ে live CPU দেখাও। এরপর `sar -r` দিয়ে আজকের memory history দেখো।

**Task 3:** `glances` install করো। প্রথমে normal mode-এ রান করেন এবং keyboard shortcuts (`1`, `c`, `m`) try করো। তারপর `-w` flag দিয়ে web mode-এ রান করেন এবং browser থেকে access করো।


## ⏭️ What's Next?

**Chapter 3 - Lesson 10: Troubleshooting High CPU/Memory (Real DevOps Scenarios)**

পরের lesson-এ আমরা শিখবো **real-world production problems** কীভাবে solve করতে হয়:
- CPU 100% হয়ে গেছে - কীভাবে identify করবেন ও fix করবেন?
- Memory leak কীভাবে detect করবেন?
- Load average অনেক বেশি, কী করবেন?
- OOM Killer কী এবং কখন activate হয়?

এটাই Chapter 3-এর সবচেয়ে exciting lesson! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../08-Disk-IO-Monitoring">← Disk IO Monitoring</a>
    </td>
    <td align="right">
      <a href="../10-Troubleshooting-High-CPU-Memory">Troubleshooting High CPU/Memory →</a>
    </td>
  </tr>
</table>
