# Chapter 3 - Lesson 2: Viewing Processes (ps, top, htop, pgrep)

**Chapter 3 | Lesson 2 of 10**


## প্রথমে একটু মনে করার চেস্টা করি...

আগের lesson-এ আমরা শিখেছিলাম যে **process** হলো একটি running program, যার নিজের PID আছে, PPID আছে, এবং সে foreground বা background-এ চলতে পারে।

কিন্তু এখন প্রশ্ন হলো আমি কীভাবে দেখবো কোন কোন process এখন চলছে?

এটাই আজকের lesson। Linux-এ process দেখার জন্য কয়েকটি powerful tool আছে:

| Tool | কাজ |
|------|-----|
| `ps` | একটি snapshot - এই মুহূর্তে কী চলছে |
| `top` | Live/real-time process monitor |
| `htop` | top-এর উন্নত version (colorful & interactive) |
| `pgrep` | নাম দিয়ে process খোঁজা |


## Tool 1: `ps` - Process Snapshot

### এটা কী?

`ps` মানে **Process Status**। এটা তোমাকে একটা still photo-র মতো দেখায়, ঠিক এই মুহূর্তে কোন processes চলছে।

> **Analogy:** কল্পনা করুন আপনি একটা ব্যস্ত রাস্তার ছবি তুললেন। সেই ছবিতে যে গাড়িগুলো আছে, সেটাই `ps`। Live video না, একটা snapshot।


### Basic Syntax:

```bash
ps [options]
```


### সবচেয়ে Common `ps` Commands:

#### 1️⃣ শুধু নিজের (current user-এর) processes দেখুন:

```bash
ps
```

**Expected Output:**
```
  PID TTY          TIME CMD
 1234 pts/0    00:00:00 bash
 5678 pts/0    00:00:00 ps
```

এখানে:
- **PID** → Process ID
- **TTY** → কোন terminal থেকে চলছে
- **TIME** → কতক্ষণ CPU ব্যবহার করেছে
- **CMD** → command-এর নাম


#### 2️⃣ সিস্টেমের সব process দেখুন (সবচেয়ে বেশি ব্যবহৃত):

```bash
ps aux
```

**Expected Output (কিছু অংশ):**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169936 13200 ?        Ss   10:00   0:01 /sbin/init
root       512  0.0  0.0  15432  1024 ?        Ss   10:00   0:00 /usr/sbin/sshd
www-data  1023  0.1  0.5 512340 45000 ?        S    10:01   0:05 nginx: worker
rahman    1500  0.0  0.1  21232  8000 pts/0    Ss   10:05   0:00 bash
rahman    1580  0.0  0.0   8936   812 pts/0    R+   10:10   0:00 ps aux
```


### `ps aux` - প্রতিটি column মানে কী?

| Column | মানে |
|--------|------|
| `USER` | কোন user এই process চালাচ্ছে |
| `PID` | Process ID |
| `%CPU` | কতটুকু CPU ব্যবহার করছে |
| `%MEM` | কতটুকু RAM ব্যবহার করছে |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | Actual RAM usage (KB) |
| `TTY` | Terminal (? মানে কোনো terminal নেই - background service) |
| `STAT` | Process-এর অবস্থা (নিচে বিস্তারিত) |
| `START` | কখন শুরু হয়েছে |
| `TIME` | Total CPU time |
| `COMMAND` | কোন command/program |


### STAT কলামের মানে:

| Code | মানে |
|------|------|
| `R` | Running (এখন CPU-তে চলছে) |
| `S` | Sleeping (কিছুর জন্য অপেক্ষা করছে) |
| `D` | Uninterruptible sleep (disk I/O-তে আটকে) |
| `Z` | Zombie (মরা কিন্তু parent এখনো নেয়নি) |
| `T` | Stopped (থামানো হয়েছে) |
| `s` | Session leader |
| `+` | Foreground process |


#### 3️⃣ নির্দিষ্ট একটি process খুঁজুন:

```bash
ps aux | grep nginx
```

**Output:**
```
www-data  1023  0.1  0.5 512340 45000 ?   S  10:01  0:05 nginx: worker process
rahman    1610  0.0  0.0   6432   720 pts/0  S+  10:12  0:00 grep --color=auto nginx
```

**Tip:** grep নিজেই output-এ দেখায়। এটা ignore করতে চাইলে:

```bash
ps aux | grep nginx | grep -v grep
```


#### 4️⃣ Process tree হিসেবে দেখুন (parent-child সম্পর্ক):

```bash
ps auxf
```

**Output (কিছু অংশ):**
```
USER       PID %CPU %MEM COMMAND
root         1  0.0  0.1 /sbin/init
root       512  0.0  0.0  \_ /usr/sbin/sshd
root      1200  0.0  0.0      \_ sshd: rahman
rahman    1201  0.0  0.1          \_ -bash
rahman    1580  0.0  0.0              \_ ps auxf
```

এখানে `\_ ` দিয়ে বোঝাচ্ছে কে কার child process।


#### 5️⃣ নির্দিষ্ট columns দেখুন:

```bash
ps -eo pid,user,pcpu,pmem,comm
```

**Output:**
```
  PID USER     %CPU %MEM COMMAND
    1 root      0.0  0.1 systemd
  512 root      0.0  0.0 sshd
 1023 www-data  0.1  0.5 nginx
 1500 rahman    0.0  0.1 bash
```

> এখানে `-e` মানে সব process, `-o` মানে output format নিজে বেছে নাও।


## Tool 2: `top` - Live Process Monitor

### এটা কী?

`top` হলো real-time process viewer। এটা প্রতি কয়েক সেকেন্ডে automatically update হয়।

> **Analogy:** `ps` যদি হয় রাস্তার still photo, তাহলে `top` হলো **live CCTV footage** - সবকিছু real-time-এ দেখাচ্ছে।


### চালু করুন:

```bash
top
```

**Expected Output:**
```
top - 10:30:01 up 2:15,  2 users,  load average: 0.15, 0.10, 0.08
Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 96.8 id,  0.3 wa,  0.0 hi,  0.1 si
MiB Mem :   3900.0 total,   1200.0 free,   1500.0 used,   1200.0 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   2100.0 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1023 www-data  20   0  512340  45000  12000 S   0.7   1.1   0:05.23 nginx
  512 root      20   0   15432   1024    800 S   0.3   0.0   0:00.45 sshd
 1500 rahman    20   0   21232   8000   6000 S   0.0   0.2   0:00.12 bash
```


### `top` এর Header অংশ বোঝার চেস্টা করি:

**Line 1: System summary**
```
top - 10:30:01 up 2:15,  2 users,  load average: 0.15, 0.10, 0.08
```
- `10:30:01` → বর্তমান সময়
- `up 2:15` → সিস্টেম ২ ঘণ্টা ১৫ মিনিট ধরে চলছে (uptime)
- `2 users` → ২ জন logged in
- `load average: 0.15, 0.10, 0.08` → গত ১, ৫, ১৫ মিনিটের average load

**Line 2: Task summary**
```
Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
```
- মোট ১২০টি process, তার মধ্যে ১টি এখন চলছে, বাকিরা ঘুমাচ্ছে

**Line 3: CPU usage**
```
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 96.8 id,  0.3 wa
```
- `us` → user processes কতটুকু CPU নিচ্ছে
- `sy` → system/kernel কতটুকু নিচ্ছে
- `id` → CPU কতটুকু **idle** (খালি) - এটা বেশি হলে ভালো
- `wa` → disk I/O-র জন্য কতটুকু CPU অপেক্ষা করছে

**Line 4 & 5: Memory**
- RAM এবং Swap কতটুকু used/free


### `top` চলাকালীন Keyboard Shortcuts:

| Key | কাজ |
|-----|-----|
| `q` | top বন্ধ করো |
| `k` | একটি process kill করো (PID চাইবে) |
| `r` | process-এর priority (nice value) পরিবর্তন করো |
| `M` | Memory usage অনুযায়ী sort করো |
| `P` | CPU usage অনুযায়ী sort করো (default) |
| `N` | PID অনুযায়ী sort করো |
| `T` | Running time অনুযায়ী sort করো |
| `1` | প্রতিটি CPU core আলাদা দেখাও |
| `u` | নির্দিষ্ট user-এর processes ফিল্টার করো |
| `h` | Help দেখুন |
| Space | Manual refresh করো |


### `top` এ নির্দিষ্ট user-এর process দেখুন:

```bash
top -u rahman
```


### `top` কে non-interactive mode-এ রান করুন (scripting-এ কাজে লাগে):

```bash
top -b -n 1
```
- `-b` → batch mode (interactive না)
- `-n 1` → মাত্র ১ বার output দিয়ে বন্ধ হয়ে যাবে


## Tool 3: `htop` - top-এর উন্নত Version

### এটা কী?

`htop` হলো `top`-এর একটি **user-friendly, colorful, interactive** version। এটা দেখতে অনেক সুন্দর এবং ব্যবহার করা সহজ।

> **Analogy:** `top` যদি হয় পুরনো black-and-white TV, তাহলে `htop` হলো **modern smart TV** সব কিছু রঙিন, সুন্দর করে সাজানো।


### Install করুন (যদি না থাকে):

```bash
# Ubuntu/Debian
sudo apt install htop -y

# CentOS/RHEL
sudo yum install htop -y
```

### চালু করুন:

```bash
htop
```

**Output দেখতে এরকম:**
```
  CPU[|||||||           25.0%]   Tasks: 45, 120 thr; 1 running
  Mem[|||||||||||||     1.5G/3.9G]   Load average: 0.15 0.10 0.08
  Swp[                  0K/2.0G]   Uptime: 02:15:30

  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
 1023 www-data   20   0  500M  43M   11M S  0.7  1.1  0:05.23 nginx
  512 root       20   0   15M 1024K  800K S  0.3  0.0  0:00.45 sshd
 1500 rahman     20   0   20M  7.8M  5.8M S  0.0  0.2  0:00.12 bash
```


### `htop` এর Keyboard Shortcuts:

| Key | কাজ |
|-----|-----|
| `F1` | Help |
| `F2` | Setup (customize করো) |
| `F3` | Search (process নাম দিয়ে খোঁজো) |
| `F4` | Filter (ফিল্টার করো) |
| `F5` | Tree view (parent-child দেখাও) |
| `F6` | Sort by column |
| `F9` | Kill a process |
| `F10` | Quit |
| `u` | নির্দিষ্ট user-এর process দেখুন |
| `Space` | একটি process select করো |
| `t` | Tree view toggle |


### `htop` vs `top` - পার্থক্য:

| Feature | `top` | `htop` |
|---------|-------|--------|
| Color | নেই | আছে |
| Mouse support | নেই | আছে |
| Scroll | সীমিত | সহজে scroll করা যায় |
| Kill process | কঠিন | সহজ (F9) |
| Tree view | `auxf` দিয়ে আলাদা | built-in (F5) |
| Install | default-এ থাকে | আলাদা install করতে হয় |
| DevOps use | Server-এ বেশি | Local debugging-এ বেশি |


## Tool 4: `pgrep` - নাম দিয়ে Process খুঁজুন

### এটা কী?

`pgrep` মানে **Process GREP**। এটা process-এর নাম দিয়ে তার **PID** বের করে দেয়।

> **Analogy:** কল্পনা করুন তোমার কাছে একটা লম্বা attendance sheet আছে। `pgrep` হলো সেই tool যেটা শুধু "Rahman" লিখলেই তার roll number বলে দেয়।


### Basic Syntax:

```bash
pgrep [options] process_name
```


### Examples:

#### 1️⃣ nginx-এর PID বের করুন:

```bash
pgrep nginx
```

**Output:**
```
1023
1024
1025
```
(তিনটি nginx worker process চলছে)


#### 2️⃣ Process নাম সহ দেখুন:

```bash
pgrep -l nginx
```

**Output:**
```
1023 nginx
1024 nginx
1025 nginx
```


#### 3️⃣ নির্দিষ্ট user-এর process খুঁজুন:

```bash
pgrep -u rahman bash
```

**Output:**
```
1500
```

---

#### 4️⃣ Full command দিয়ে খুঁজুন:

```bash
pgrep -f "python manage.py"
```

> `-f` flag দিলে শুধু process name না, পুরো command line-এ search করে।


#### 5️⃣ Process আছে কিনা check করুন (scripting-এ কাজে লাগে):

```bash
if pgrep nginx > /dev/null; then
    echo "Nginx is running"
else
    echo "Nginx is NOT running"
fi
```


## DevOps Real-World Use Cases

### Scenario 1: Server slow হয়ে গেছে - কোন process সমস্যা করছে?

```bash
# CPU অনুযায়ী top processes দেখুন
ps aux --sort=-%cpu | head -10

# অথবা top চালু করে M চাপুন (memory sort)
top
```


### Scenario 2: একটি service চলছে কিনা জানুন:

```bash
pgrep -l nginx
# বা
ps aux | grep nginx | grep -v grep
```


### Scenario 3: কোন user সবচেয়ে বেশি resource নিচ্ছে:

```bash
ps aux --sort=-%cpu | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
```


### Scenario 4: Zombie process আছে কিনা দেখুন:

```bash
ps aux | grep -w Z
```


### Scenario 5: একটি process-এর সব details দেখুন:

```bash
ps -p 1023 -o pid,ppid,user,pcpu,pmem,stat,comm
```

**Output:**
```
  PID  PPID USER     %CPU %MEM STAT COMMAND
 1023     1 www-data  0.1  1.1 S    nginx
```


## 📝 Quick Summary

- **`ps`** → একটি snapshot - এই মুহূর্তে কী process চলছে দেখায়
- **`ps aux`** → সব user-এর সব process বিস্তারিত দেখায়
- **`ps aux | grep <name>`** → নির্দিষ্ট process খোঁজো
- **`ps auxf`** → parent-child tree হিসেবে দেখাও
- **`top`** → Real-time live process monitor
- **`top -u user`** → নির্দিষ্ট user-এর processes দেখাও
- **`htop`** → top-এর colorful, interactive version
- **`pgrep nginx`** → নাম দিয়ে PID বের করো
- **`pgrep -l nginx`** → নাম সহ PID দেখাও
- **`pgrep -u user`** → নির্দিষ্ট user-এর process-এর PID বের করো


## 🏋️ Practice Tasks

এখন তুমি নিজে চেষ্টা করো:

**Task 1:**
```bash
# আপনার system-এ সব process দেখুন এবং
# সবচেয়ে বেশি CPU ব্যবহারকারী top 5 process বের করুন
ps aux --sort=-%cpu | head -6
```

**Task 2:**
```bash
# top চালু করুন, তারপর:
# - M চাপুন (memory অনুযায়ী sort)
# - 1 চাপুন (প্রতিটি CPU আলাদা দেখুন)
# - q চাপুন (বের হয়ে আসুন)
top
```

**Task 3:**
```bash
# bash process-এর PID বের করুন দুটো উপায়ে:
pgrep -l bash
ps aux | grep bash | grep -v grep
# দুটোর output কি একই আসছে?
```

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 3: Controlling Processes**

পরের lesson-এ আমরা শিখবো কীভাবে process বন্ধ করতে হয়। `kill`, `killall`, `pkill` এবং বিভিন্ন **signals** (SIGTERM, SIGKILL, SIGHUP) কী এবং কখন কোনটা ব্যবহার করতে হয়। DevOps-এ stuck process handle করার সবচেয়ে গুরুত্বপূর্ণ skill! *Happy Learning* 🚀
