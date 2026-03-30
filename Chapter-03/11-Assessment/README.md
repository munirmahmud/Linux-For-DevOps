# Chapter 3 - Assessment 

**Total 13 Practice Tasks**


আপনি Chapter 3 শেষ করেছেন, এটা সত্যিই দারুণ! 🎉 আপনি অনেকদূর এগিয়ে এসেছেন এটা অনেক ভালো প্রগ্রেস। তবে এখানে থেমে থাকলেই হবে না আমাদের আরো সামনে এগোতে হবে। 


> আমি যেভাবে সাজিয়েছি তাহলো সব assignment আগে দেব, তারপর আপনি চেষ্টা করবেন। এতে আপনার **নিজে চিন্তা করার সুযোগ** থাকবে এবং **real interview/job feel** পাবেন।


## Rules of This Assessment

- প্রতিটি task **real production scenario** থেকে নেওয়া
- কোনো hint নেই - নিজে চেষ্টা করুন
- আটকে গেলে - সল্যুশন অংশ দেখতে পারেন


### 🔴 Assignment 1 - Zombie Process Investigation
আপনার production server-এ একজন developer জানাল যে application behave করছেনে না। আপনি suspect করছেন zombie process আছে।

**Tasks:**
1. সিস্টেমে কোনো zombie process আছে কিনা খুঁজে বের করুন
2. কতটি zombie process চলছে তা count করুন
3. সেই zombie process-এর parent process কে - সেটা বের করুন
4. ওই parent process-এর details দেখুন (কতক্ষণ ধরে চলছে, কে চালাচ্ছে)


### 🔴 Assignment 2 - Memory Leak Detection
Production-এ একটি Java application আছে (`java` process)। DevOps team lead বলল - "প্রতি ঘণ্টায় memory বাড়ছে, কিন্তু কমছে না। সন্দেহ হচ্ছে memory leak।"

**Tasks:**
1. সবচেয়ে বেশি memory ব্যবহার করা top 5 process বের করুন
2. শুধু `java` নামের process-গুলোর PID ও memory usage দেখুন
3. Total available memory, used memory এবং swap usage এক command-এ দেখুন (human-readable format-এ)
4. `/proc/meminfo` থেকে `MemAvailable` এবং `SwapFree` এর value বের করুন


### 🔴 Assignment 3 - Runaway CPU Process
Monitoring alert এসেছে - CPU usage 95%+ হয়ে গেছে। আপনাকে দ্রুত কারণ খুঁজে বের করতে হবে এবং সমাধান করতে হবে।

**Tasks:**
1. CPU usage অনুযায়ী process sort করে top 10 দেখুন (non-interactive, একবার output নিন)
2. সবচেয়ে বেশি CPU খাচ্ছে এমন process-এর PID বের করুন (single command-এ)
3. ঐ process-টিকে প্রথমে **SIGTERM** দিয়ে gracefully stop করার চেষ্টা করুন
4. যদি SIGTERM কাজ না করে - **SIGKILL** দিয়ে force kill করুন
5. Kill করার পরে verify করুন যে process আর নেই


### 🔴 Assignment 4 - Background Job Management
আপনাকে একটি long-running backup script চালাতে হবে। কিন্তু terminal close হলেও script রান থাকতে হবে এবং আপনি অন্য কাজও করতে পারবেন।

**Tasks:**
1. `sleep 500` কমান্ডটি background-এ রান করুন
2. এখন আরেকটি `sleep 600` রান করুন, এবার foreground-এ - তারপর সেটাকে suspend করুন, তারপর background-এ পাঠান
3. সব background jobs list করুন
4. প্রথম job-টাকে আবার foreground-এ নিয়ে আসো
5. এমনভাবে একটি `sleep 1000` রান করুন যেন terminal বন্ধ হলেও সেটা চলতে থাকে


### 🔴 Assignment 5 - Process Priority Tuning
আপনার server-এ একসাথে দুটো কাজ চলছে। একটি critical database backup এবং একটি non-critical log processing script। আপনাকে নিশ্চিত করতে হবে যে backup সবসময় বেশি CPU পাবে।

**Tasks:**
1. Log processing script (`sleep 300` দিয়ে simulate করুন) রান করুন এবং তার nice value দেখুন
2. ওই process-এর nice value **+15** করে দিন (low priority)
3. Database backup (`sleep 400` দিয়ে simulate করুন) রান করুন nice value **-5** দিয়ে (high priority - root লাগবে)
4. দুটো process-এর nice value পাশাপাশি দেখুন এবং verify করুন


### 🔴 Assignment 6 - Disk I/O Bottleneck
Application team জানাল যে তাদের app অনেক slow। আপনি suspect করছেন disk I/O problem আছে।

**Tasks:**
1. সব mounted filesystem-এর disk usage দেখুন (human-readable)
2. `/var/log` directory কত জায়গা নিচ্ছে বের করুন (summary format-এ)
3. Current disk I/O statistics দেখুন, কোন disk সবচেয়ে বেশি busy
4. সবচেয়ে বেশি জায়গা নেওয়া top 5 directory বের করুন `/var`-এর মধ্যে


### 🔴 Assignment 7 - Load Average Analysis
সকালে office-এ এসে দেখলে যে server-এর load average অনেক বেশি। Team lead জিজ্ঞেস করল, "এটা কি সমস্যা নাকি normal?"

**Tasks:**
1. Server-এর current load average দেখুন
2. Server-এ কতটি CPU core আছে বের করুন
3. Load average এবং CPU core সংখ্যা দেখে বলুন, এটা কি concerning?
4. কতক্ষণ ধরে server চলছে (uptime) এবং কতজন user logged in আছে দেখুন
5. Per-CPU statistics দেখুন (কোন CPU কতটুকু busy)


### 🔴 Assignment 8 - Process Monitoring One-Liner
আপনাকে একটি monitoring script লিখতে বলা হয়েছে যেটা specific process monitor করবে।

**Tasks:**
1. `nginx` নামের সব process-এর PID list করুন
2. `nginx` process চলছে কিনা - শুধু "Running" বা "Not Running" print করুন (single command)
3. যেকোনো একটি process-এর PID বের করুন এবং সেই PID দিয়ে process-এর সব details দেখুন `/proc/[PID]/` থেকে - status, cmdline, এবং fd (open file descriptors) দেখুন
4. প্রতি 2 সেকেন্ডে automatically refresh হয় এভাবে `ps` output দেখুন (top ছাড়া)


### 🔴 Assignment 9 - Signal Mastery
Production deployment-এ আপনাকে nginx কে gracefully reload করতে হবে (traffic drop ছাড়া), একটি hung process debug করতে হবে।

**Tasks:**
1. `nginx` process-এ **SIGHUP** signal পাঠান (graceful reload)
2. একটি process-এ **SIGSTOP** পাঠিয়ে pause করুন, তারপর **SIGCONT** দিয়ে resume করুন
3. সব available signals-এর list দেখুন
4. `kill` এবং `pkill` এর মধ্যে পার্থক্য demonstrate করুন। দুটো আলাদা `sleep` process চালিয়ে একটাকে PID দিয়ে, আরেকটাকে name দিয়ে kill করুন


### 🔴 Assignment 10 - System Performance Report
Weekly performance review meeting-এর জন্য আপনাকে একটি quick system health snapshot তৈরি করতে হবে।

**Tasks:**
নিচের সব information এক script-এ collect করুন এবং `/tmp/system_report.txt` ফাইলে save করুন:
1. Report generation time (date & time)
2. System uptime এবং load average
3. Total/Used/Free Memory (human-readable)
4. Top 3 CPU-consuming processes
5. Top 3 Memory-consuming processes
6. Disk usage summary (সব mounted filesystem)


### 🔴 Assignment 11 - Process Forensics
Security team জানাল, রাত 3টায় একটি suspicious process চলেছিল। আপনাকে investigate করতে হবে।

**Tasks:**
1. Currently running সব processes-এর মধ্যে `root` user ছাড়া অন্য কে কে process চালাচ্ছে দেখাও
2. কোন processes সবচেয়ে বেশিক্ষণ ধরে চলছে (start time সহ) দেখাও
3. একটি নির্দিষ্ট process (যেকোনো একটি বেছে নাও) - সেটি কোন port/socket use করছেনে কিনা দেখাও
4. PID 1 (init/systemd) process-এর সব children processes দেখাও (process tree format-এ)


### 🔴 Assignment 12 - Memory & Swap Emergency
Production server-এ OOM (Out of Memory) killer activate হয়েছে এবং একটি critical process kill হয়ে গেছে।

**Tasks:**
1. OOM killer কোন process kill করেছে, সেটা system log থেকে বের করুন
2. Current swap usage দেখুন, swap কি প্রায় full?
3. কোন process সবচেয়ে বেশি swap use করছেনে বের করুন
4. একটি specific process-এর OOM score দেখুন (`/proc/[PID]/oom_score`)
5. Emergency-তে swap clear করার command লিখুন (সাবধানে!)


### 🔴 Assignment 13 - Full Troubleshooting Scenario (Bonus 🌟)
রাত 2টায় PagerDuty alert - "Production API server responding slowly, CPU 87%, Memory 91%"

**আপনাকে step-by-step নিচের সব করতে হবে:**

1. প্রথমেই system-এর overall health দেখুন (load, memory, cpu - একসাথে)
2. Top CPU-consuming process identify করুন
3. Top Memory-consuming process identify করুন
4. সেই processes-এর details বের করুন (কতক্ষণ ধরে চলছে, কোন user, PPID)
5. যদি কোনো process zombie হয়, parent কে identify করুন
6. Disk I/O check করুন, disk bottleneck আছে কিনা
7. যদি একটি process clearly runaway হয়, gracefully terminate করুন
8. 2 মিনিট পরে আবার system health check করুন এবং improvement দেখান
9. পুরো investigation-এর একটি summary `/tmp/incident_report.txt`-এ লিখুন


# 🎯 Chapter 3 - Complete Assessment Answers


## 🔴 Assignment 1 - Zombie Process Investigation

```bash
# Task 1: Zombie process খুঁজে বের করুন
ps aux | grep 'Z'
# অথবা আরও clean way:
ps aux | awk '$8=="Z"'

# Task 2: Zombie process count করুন
ps aux | awk '$8=="Z"' | wc -l

# Task 3: Zombie process-এর Parent (PPID) বের করুন
ps aux | awk '$8=="Z" {print $2}'   # zombie এর PID পাবেন
ps -o ppid= -p <PID>                # সেই PID এর parent বের করুন

# অথবা একসাথে:
ps -eo pid,ppid,stat,comm | awk '$3~/Z/'

# Task 4: Parent process এর details দেখাও
ps -p <PPID> -o pid,ppid,user,etime,cmd
```

**Real Life Note:** Zombie process নিজে কোনো resource নেয় না, কিন্তু process table-এ জায়গা নেয়। Parent process-কে kill করলে zombie পরিষ্কার হয়। Production-এ এটা application bug-এর sign।


## 🔴 Assignment 2 - Memory Leak Detection

```bash
# Task 1: Top 5 memory-consuming processes
ps aux --sort=-%mem | head -6
# (প্রথম line header, তাই head -6)

# Task 2: শুধু java process এর PID ও memory
ps aux | grep '[j]ava' | awk '{print $2, $4, $11}'
# [j]ava লেখার কারণ: grep নিজেকে result-এ দেখাবে না

# Task 3: Total memory একসাথে human-readable
free -h

# Expected Output:
#               total        used        free      shared  buff/cache   available
# Mem:           15Gi        8Gi        2Gi        500Mi       4Gi        6Gi
# Swap:           2Gi        500Mi      1.5Gi

# Task 4: /proc/meminfo থেকে নির্দিষ্ট values
grep -E 'MemAvailable|SwapFree' /proc/meminfo
```

**Real Life Note:** `free -h` এর `available` column টাই সবচেয়ে important, এটা actually কতটুকু memory পাওয়া যাবে/খালি আছে তা বলে দেয়। `free` column misleading হতে পারে কারণ Linux cache হিসেবে memory ধরে রাখে।


## 🔴 Assignment 3 - Runaway CPU Process

```bash
# Task 1: CPU অনুযায়ী top 10 process (non-interactive)
ps aux --sort=-%cpu | head -11

# Task 2: সবচেয়ে বেশি CPU খাচ্ছে তার PID (single command)
ps aux --sort=-%cpu | awk 'NR==2{print $2}'
# NR==2 মানে header বাদ দিয়ে প্রথম result

# Task 3: SIGTERM দিয়ে gracefully stop
kill -15 <PID>
# অথবা
kill -SIGTERM <PID>

# Task 4: SIGKILL দিয়ে force kill
kill -9 <PID>
# অথবা
kill -SIGKILL <PID>

# Task 5: Verify করুন process আর নেই
ps aux | grep <PID>
# অথবা
kill -0 <PID> 2>/dev/null && echo "Still Running" || echo "Process Gone"
```

**Real Life Note:** সবসময় আগে `-15` (SIGTERM) দিন, process নিজে cleanly বন্ধ হওয়ার সুযোগ পায়। `-9` (SIGKILL) last resort, এটা দিলে process কোনো cleanup করতে পারে না, data corruption হতে পারে।


## 🔴 Assignment 4 - Background Job Management

```bash
# Task 1: Background-এ রান করুন
sleep 500 &
# Output: [1] 12345  ← [job number] PID

# Task 2: Foreground-এ রান করুন → suspend → background পাঠিয়ে দিন
sleep 600          # foreground-এ রান করুন

# এখন Ctrl+Z চাপো → suspend হবে
# Output: [2]+  Stopped   sleep 600
bg %2              # background-এ পাঠিয়ে দিন

# Task 3: সব background jobs list করুন
jobs -l
# Output:
# [1]  12345 Running    sleep 500 &
# [2]  12346 Running    sleep 600 &

# Task 4: প্রথম job foreground-এ আনুন
fg %1

# Task 5: Terminal বন্ধ হলেও চলবে
nohup sleep 1000 &
# অথবা আরও ভালো:
nohup sleep 1000 > /tmp/sleep.log 2>&1 &
```

**Real Life Note:** Production-এ long-running script রান করার জন্য `nohup` এর চেয়ে `tmux` বা `screen` বেশি ব্যবহার হয়। কিন্তু quick task-এর জন্য `nohup` perfect।


## 🔴 Assignment 5 - Process Priority Tuning

```bash
# Task 1: Log processing রান করুন এবং nice value দেখুন
sleep 300 &
ps -p $! -o pid,ni,cmd
# Default nice value: 0

# Task 2: Nice value +15 করুন (low priority)
renice +15 -p <PID>
# Output: <PID> (old priority: 0, new priority: 15)

# Task 3: Backup রান করুন nice -5 দিয়ে (root লাগবে)
sudo nice -n -5 sleep 400 &

# Task 4: দুটো process-এর nice value পাশাপাশি দেখুন
ps -p <PID1>,<PID2> -o pid,ni,cmd

# Expected Output:
#   PID  NI COMMAND
# 12345  15 sleep 300
# 12346  -5 sleep 400
```

**Real Life Note:** Nice value range: **-20 (highest priority)** থেকে **+19 (lowest priority)**। Negative value দিতে root/sudo লাগে। Production database-কে সবসময় low nice value দেওয়া হয়।


## 🔴 Assignment 6 - Disk I/O Bottleneck

```bash
# Task 1: সব filesystem-এর disk usage (human-readable)
df -h

# Task 2: /var/log কত জায়গা নিচ্ছে
du -sh /var/log

# Task 3: Disk I/O statistics
iostat -x 1 3
# -x = extended stats, 1 = প্রতি 1 সেকেন্ডে, 3 = 3 বার

# অথবা সহজে:
iostat -d -h

# Task 4: /var এর মধ্যে top 5 বড় directory
du -h /var/* 2>/dev/null | sort -rh | head -5
# অথবা আরও precise:
du -h --max-depth=1 /var | sort -rh | head -6
```

**Real Life Note:** Production-এ `/var/log` প্রায়ই disk ভরিয়ে দেয়। `df -h` রান করে দেখুন কোন partition full, তারপর `du` দিয়ে খুঁজে বের করুন কোন directory সবচেয়ে বেশি জায়গা নিচ্ছে।


## 🔴 Assignment 7 - Load Average Analysis

```bash
# Task 1: Current load average
uptime
# Output: 14:32:01 up 5 days, 3:21, 2 users, load average: 2.15, 1.87, 1.92
# তিনটি number = last 1 min, 5 min, 15 min এর average

# Task 2: CPU core count
nproc
# অথবা:
lscpu | grep '^CPU(s):'
# অথবা:
cat /proc/cpuinfo | grep processor | wc -l

# Task 3: Load average vs CPU core বিশ্লেষণ
# যদি load average > CPU core count → concerning ✅
# যদি load average < CPU core count → normal ✅
# Example: 4 core, load 2.15 → normal (50% utilized)
# Example: 4 core, load 6.50 → concerning! (queue তৈরি হচ্ছে)

# Task 4: Uptime এবং logged in users
uptime
who         # কে কে logged in
w           # আরও details সহ

# Task 5: Per-CPU statistics
mpstat -P ALL 1 3
# -P ALL = সব CPU দেখাও
# অথবা top চালিয়ে '1' চাপুন → per-CPU দেখাবে
```

**Real Life Note:** Load average-কে CPU core দিয়ে ভাগ করুন। Result যদি 1.0 এর বেশি হয় তাহলে system overloaded। 0.7 পর্যন্ত comfortable, 1.0+ মানে investigate করুন।


## 🔴 Assignment 8 - Process Monitoring One-Liner

```bash
# Task 1: nginx এর সব PID list করুন
pgrep nginx
# অথবা:
pidof nginx
# অথবা:
ps aux | grep '[n]ginx' | awk '{print $2}'

# Task 2: nginx চলছে কিনা "Running/Not Running" print করুন
pgrep nginx > /dev/null && echo "Running" || echo "Not Running"

# Task 3: নির্দিষ্ট PID এর /proc থেকে details
PID=$(pgrep nginx | head -1)

cat /proc/$PID/status      # process status details
cat /proc/$PID/cmdline | tr '\0' ' '  # command যেটা দিয়ে চালানো হয়েছে
ls /proc/$PID/fd | wc -l   # কতটি file descriptor open আছে
ls -la /proc/$PID/fd        # কোন কোন file open আছে

# Task 4: প্রতি 2 সেকেন্ডে refresh (top ছাড়া)
watch -n 2 'ps aux --sort=-%cpu | head -10'
```

**Real Life Note:** `watch` command টা monitoring-এ অনেক কাজে লাগে। `/proc/[PID]/fd` দেখে বোঝা যায় একটা process কতগুলো file/socket/connection open রেখেছে, memory leak বা connection leak ধরতে সাহায্য করে।


## 🔴 Assignment 9 - Signal Mastery

```bash
# Task 1: nginx কে SIGHUP পাঠান (graceful reload)
sudo kill -HUP $(pgrep nginx | head -1)
# অথবা:
sudo pkill -HUP nginx

# Task 2: SIGSTOP দিয়ে pause, SIGCONT দিয়ে resume
sleep 300 &
PID=$!
kill -STOP $PID    # pause করুন
ps -p $PID -o stat  # দেখবে: T (stopped)

kill -CONT $PID    # resume করুন
ps -p $PID -o stat  # দেখবে: S (sleeping/running)

# Task 3: সব signals-এর list
kill -l
# Output:
#  1) SIGHUP    2) SIGINT    3) SIGQUIT   4) SIGILL
#  9) SIGKILL  15) SIGTERM  18) SIGCONT  19) SIGSTOP ...

# Task 4: kill vs pkill demonstrate করুন
sleep 800 &   # PID দিয়ে kill করো
sleep 900 &   # name দিয়ে kill করো

# PID দিয়ে kill:
kill -15 <PID_of_sleep_800>

# Name দিয়ে pkill:
pkill -15 -f "sleep 900"
# -f মানে full command line match করবে

# Verify:
jobs -l
```

**Real Life Note:**
| Signal | Number | কখন ব্যবহার করবেন |
|--------|--------|-----------------|
| SIGHUP | 1 | Config reload (nginx, apache) |
| SIGTERM | 15 | Graceful shutdown |
| SIGKILL | 9 | Force kill (last resort) |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume paused process |


## 🔴 Assignment 10 - System Performance Report

```bash
#!/bin/bash
# system_report.sh

REPORT="/tmp/system_report.txt"

{
echo "========================================"
echo "   SYSTEM PERFORMANCE REPORT"
echo "========================================"

# Task 1: Report generation time
echo ""
echo "📅 Report Generated: $(date)"

# Task 2: Uptime এবং load average
echo ""
echo "⏱️  UPTIME & LOAD AVERAGE:"
uptime

# Task 3: Memory (human-readable)
echo ""
echo "🧠 MEMORY USAGE:"
free -h

# Task 4: Top 3 CPU-consuming processes
echo ""
echo "🔥 TOP 3 CPU-CONSUMING PROCESSES:"
ps aux --sort=-%cpu | awk 'NR==1 || NR<=4' 

# Task 5: Top 3 Memory-consuming processes
echo ""
echo "💾 TOP 3 MEMORY-CONSUMING PROCESSES:"
ps aux --sort=-%mem | awk 'NR==1 || NR<=4'

# Task 6: Disk usage summary
echo ""
echo "💿 DISK USAGE:"
df -h

echo ""
echo "========================================"
echo "   END OF REPORT"
echo "========================================"
} > "$REPORT"

echo "✅ Report saved to $REPORT"
cat "$REPORT"
```

```bash
# Script রান করার command:
chmod +x system_report.sh
./system_report.sh

# অথবা script ছাড়া একসাথে:
{ echo "=== $(date) ==="; uptime; free -h; \
  ps aux --sort=-%cpu | head -4; \
  ps aux --sort=-%mem | head -4; \
  df -h; } > /tmp/system_report.txt
```


## 🔴 Assignment 11 - Process Forensics

```bash
# Task 1: root ছাড়া অন্য কে কে process চালাচ্ছে
ps aux | awk '$1 != "root" && NR > 1 {print $1}' | sort -u
# অথবা details সহ:
ps aux | awk 'NR>1 && $1!="root"' | awk '{print $1, $2, $11}'

# Task 2: কোন processes সবচেয়ে বেশিক্ষণ ধরে চলছে
ps aux --sort=start_time | awk 'NR>1{print $1,$2,$9,$11}' | head -10
# অথবা:
ps -eo pid,user,etime,cmd --sort=etime | tail -10
# etime = elapsed time (কতক্ষণ ধরে চলছে)

# Task 3: Process কোন port/socket use করছে
PID=$(pgrep nginx | head -1)
ls -la /proc/$PID/fd 2>/dev/null
# অথবা:
ss -tlnp | grep $PID
# অথবা:
lsof -p $PID | grep -E 'IPv|sock'

# Task 4: PID 1 এর সব children (process tree)
pstree -p 1
# অথবা:
ps --ppid 1 -o pid,ppid,cmd
# অথবা পুরো tree:
pstree -p | head -30
```

**Real Life Note:** `ps -eo etime` দিয়ে দেখা যায় কোন process কতক্ষণ ধরে চলছে। Suspicious process যদি রাত 3টায় start হয়ে থাকে সেটা security incident এর sign হতে পারে।


## 🔴 Assignment 12 - Memory & Swap Emergency

```bash
# Task 1: OOM killer কোন process kill করেছে
sudo dmesg | grep -i 'oom\|killed process'
# অথবা:
sudo grep -i 'out of memory\|oom_kill' /var/log/syslog
# অথবা journald থেকে:
sudo journalctl -k | grep -i oom

# Task 2: Current swap usage
free -h
# Swap line দেখুন - used কলাম
swapon --show    # কোন swap device/file আছে details সহ

# Task 3: কোন process সবচেয়ে বেশি swap use করছে
# Method 1:
for pid in /proc/[0-9]*; do
    swap=$(grep VmSwap $pid/status 2>/dev/null | awk '{print $2}')
    if [ -n "$swap" ] && [ "$swap" -gt 0 ]; then
        name=$(cat $pid/comm 2>/dev/null)
        echo "$swap KB - $name (PID: ${pid##*/})"
    fi
done | sort -rn | head -10

# Method 2 (simpler):
ps aux --sort=-%mem | awk 'NR<=6{print $1,$2,$4,$6,$11}'
# $6 = VSZ (virtual memory including swap)

# Task 4: Specific process এর OOM score
PID=$(pgrep nginx | head -1)
cat /proc/$PID/oom_score
# Score যত বেশি → OOM killer ততো আগে এই process kill করবে (0-1000)

# Bonus: OOM score adjust করা
cat /proc/$PID/oom_score_adj  # current adjustment (-1000 থেকে +1000)
echo -500 | sudo tee /proc/$PID/oom_score_adj  # important process কে protect করুন

# Task 5: Emergency swap clear
# ⚠️ সাবধান! এটা করার আগে নিশ্চিত করুন RAM তে swap নেওয়ার মতো space আছে
sudo swapoff -a    # swap বন্ধ করুন (RAM এ নিয়ে আসবে)
sudo swapon -a     # আবার চালু করুন (এখন swap clear)
```

**Real Life Note:** OOM killer automatically সবচেয়ে বেশি memory নেওয়া process kill করে। Critical process (যেমন database) কে protect করতে `oom_score_adj` কে `-1000` করে দিন, OOM killer কখনো এটাকে kill করবে না।


## 🔴 Assignment 13 - Full Troubleshooting Scenario 🌟

```bash
#!/bin/bash
# incident_investigation.sh
# রাত 2টার Production Incident Response

REPORT="/tmp/incident_report.txt"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

{
echo "============================================"
echo "   🚨 INCIDENT REPORT"
echo "   Generated: $TIMESTAMP"
echo "============================================"

# Step 1: Overall system health
echo ""
echo "--- STEP 1: SYSTEM OVERVIEW ---"
echo "Uptime & Load:"
uptime

echo ""
echo "Memory Status:"
free -h

echo ""
echo "CPU Info:"
lscpu | grep -E 'CPU\(s\)|Model name'

# Step 2: Top CPU-consuming process
echo ""
echo "--- STEP 2: TOP CPU CONSUMERS ---"
ps aux --sort=-%cpu | head -6

# Step 3: Top Memory-consuming process
echo ""
echo "--- STEP 3: TOP MEMORY CONSUMERS ---"
ps aux --sort=-%mem | head -6

# Step 4: Details of top processes
echo ""
echo "--- STEP 4: TOP PROCESS DETAILS ---"
TOP_CPU_PID=$(ps aux --sort=-%cpu | awk 'NR==2{print $2}')
TOP_MEM_PID=$(ps aux --sort=-%mem | awk 'NR==2{print $2}')

echo "Top CPU Process (PID: $TOP_CPU_PID):"
ps -p $TOP_CPU_PID -o pid,ppid,user,etime,cmd 2>/dev/null

echo ""
echo "Top Memory Process (PID: $TOP_MEM_PID):"
ps -p $TOP_MEM_PID -o pid,ppid,user,etime,cmd 2>/dev/null

# Step 5: Zombie process check
echo ""
echo "--- STEP 5: ZOMBIE PROCESS CHECK ---"
ZOMBIES=$(ps aux | awk '$8=="Z"')
if [ -z "$ZOMBIES" ]; then
    echo "✅ No zombie processes found"
else
    echo "⚠️  Zombie processes detected:"
    echo "$ZOMBIES"
    echo "Parent PIDs:"
    ps aux | awk '$8=="Z"{print $2}' | while read zpid; do
        ps -o ppid= -p $zpid 2>/dev/null
    done
fi

# Step 6: Disk I/O check
echo ""
echo "--- STEP 6: DISK I/O STATUS ---"
df -h
echo ""
echo "Disk Usage Top Directories:"
du -h --max-depth=1 /var 2>/dev/null | sort -rh | head -5

# Step 7: Runaway process termination
echo ""
echo "--- STEP 7: PROCESS TERMINATION DECISION ---"
TOP_CPU=$(ps aux --sort=-%cpu | awk 'NR==2{print $3}')
echo "Top CPU Usage: $TOP_CPU%"

if (( $(echo "$TOP_CPU > 80" | bc -l) )); then
    echo "⚠️  High CPU detected! PID $TOP_CPU_PID using $TOP_CPU% CPU"
    echo "Action: Sending SIGTERM to PID $TOP_CPU_PID"
    # kill -15 $TOP_CPU_PID  # ← real scenario-তে uncomment করুন
    echo "(SIGTERM sent - monitoring for 30 seconds)"
else
    echo "✅ CPU usage acceptable: $TOP_CPU%"
fi

# Step 8: Post-action health check (2 minute wait simulate করছি)
echo ""
echo "--- STEP 8: POST-ACTION HEALTH CHECK ---"
echo "Waiting 10 seconds for system to stabilize..."
sleep 10

echo "Current system state:"
uptime
free -h | grep Mem

# Step 9: Summary
echo ""
echo "============================================"
echo "   📋 INCIDENT SUMMARY"
echo "============================================"
echo "Time of Investigation : $TIMESTAMP"
echo "Load Average          : $(uptime | awk -F'load average:' '{print $2}')"
echo "Memory Available      : $(free -h | awk '/Mem/{print $7}')"
echo "Top CPU Process       : $(ps aux --sort=-%cpu | awk 'NR==2{print $11, "("$3"%)"}')"
echo "Top MEM Process       : $(ps aux --sort=-%mem | awk 'NR==2{print $11, "("$4"%)"}')"
echo "Zombie Processes      : $(ps aux | awk '$8=="Z"' | wc -l)"
echo "Disk Usage (root)     : $(df -h / | awk 'NR==2{print $5}')"
echo "============================================"
echo "   Status: Investigation Complete ✅"
echo "============================================"

} | tee "$REPORT"

echo ""
echo "✅ Full incident report saved to: $REPORT"
```

```bash
# রান করার command:
chmod +x incident_investigation.sh
sudo ./incident_investigation.sh
```


## 📊 Assessment Score Card

এখন নিজেকে evaluate করুন:

| Assignment | Topic | আপনি কি পেরেছিলেন? |
|---|---|---|
| 1 | Zombie Process | ✅ / ❌ |
| 2 | Memory Leak | ✅ / ❌ |
| 3 | CPU Runaway | ✅ / ❌ |
| 4 | Background Jobs | ✅ / ❌ |
| 5 | Process Priority | ✅ / ❌ |
| 6 | Disk I/O | ✅ / ❌ |
| 7 | Load Average | ✅ / ❌ |
| 8 | Process Monitoring | ✅ / ❌ |
| 9 | Signal Mastery | ✅ / ❌ |
| 10 | Performance Report | ✅ / ❌ |
| 11 | Process Forensics | ✅ / ❌ |
| 12 | Memory Emergency | ✅ / ❌ |
| 13 | Full Incident 🌟 | ✅ / ❌ |


## 🎯 Key Takeaways - Chapter 3

> একজন DevOps Engineer হিসেবে এই commands গুলো তোমার **muscle memory** তে থাকতে হবে:

```
ps aux --sort=-%cpu     → CPU troubleshooting
ps aux --sort=-%mem     → Memory troubleshooting  
kill -15 / kill -9      → Graceful vs Force kill
renice / nice           → Priority management
nohup ... &             → Persistent background jobs
free -h / vmstat        → Memory monitoring
df -h / du -sh          → Disk monitoring
uptime / lscpu          → Load & CPU info
dmesg | grep oom        → OOM investigation
watch -n 2 '...'        → Continuous monitoring
```


Chapter 3 officially complete! 🎉 এখন Chapter 4: Networking in Linux শুরু করার জন্য ready! যদি কোন একটা চ্যাপ্টারে আপনার দুর্বলতা থাকে তাহলে আপনার প্রতি আমার পরামর্শ থাকবে সেটা আবার রিভিউ করে তারপরে Chapter 4 শুরু করা। ফাউন্ডেশন স্ট্রং না হলে মাঝ পথে হাল ছেড়ে দিতে হবে বা একটা পর্যায়ে প্রাকটিস করতে আর ভালো লাগবে না। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../10-Troubleshooting-High-CPU-Memory">← Troubleshooting High CPU/Memory</a>
    </td>
    <td align="right">
      <a href="../../Chapter-04/01-Networking-Basics-Recap/">Networking Basics →</a>
    </td>
  </tr>
</table>