# Chapter 3 - Lesson 1: Process & Performance Management

**Chapter 3 | Lesson 1 of 10**


## Process কী? - সহজ ভাষায়

কল্পনা করুন আপনি একটা **রেস্টুরেন্ট** চালাচ্ছেন।

- **রেসিপি** = Program (disk-এ থাকা file)
- **রান্না করার কাজ** = Process (রেসিপি execute হচ্ছে)
- **রান্নাঘর** = RAM/CPU (যেখানে কাজ হচ্ছে)

অর্থাৎ **যখন কোনো program চালু হয়, সেটাই তখন Process হয়ে যায়।**

একই program থেকে **একাধিক process** তৈরি হতে পারে। যেমন আপনি একই সাথে দুটো terminal ওপেন করলেন, এক্ষেত্রে দুটো আলাদা process চলবে।


## PID - Process ID

Linux-এ প্রতিটা process-কে একটা **unique number** দেওয়া হয়, এটাই **PID (Process ID)**। 

মনে করুন প্রতিটা process-এর একটা জাতীয় পরিচয়পত্র আছে। সেই ID নম্বরই হলো PID।


| PID | Process |
|-----|---------|
| 1 | systemd (সবার বাবা - প্রথম process) |
| 2 | kthreadd (kernel thread) |
| 1234 | আপনার bash session |
| 5678 | আপনার চালু করা nginx |

> **PID 1 সবসময় `systemd`** এটাই Linux boot হওয়ার পর প্রথম চালু হয়, এবং বাকি সব process এর থেকেই জন্ম নেয়।


## PPID - Parent Process ID

Linux-এ প্রতিটা process কোনো না কোনো **parent process থেকে জন্ম নেয়**।

```
systemd (PID 1)
    └── bash (PID 500)
            └── ls (PID 501)
```

- **PPID** = Parent Process-এর PID
- `ls` command রান করলে, bash তার **child process** তৈরি করে
- child মরে গেলে parent-কে জানায় তারপর parent সেটা clean করে

> এটা একটা **family tree** এর মতো। সব process-এর একটা বাবা আছে।


## Foreground vs Background Process

### Foreground Process

Foreground Process terminal আটকে রাখে, যার কারনে আমরা অন্য কোন কাজ করতে পারি না।

**উদাহরণ:**
```bash
sleep 100
```
এটা রান করলে terminal আটকে যাবে 100 সেকেন্ড পর্যন্ত।


### Background Process

Background Process terminal আটকায় না যার কারনে আমরা অন্য কাজ করতে পারি।

**উদাহরণ:**
```bash
sleep 100 &
```

শেষে `&` দিলে process **background-এ** চলে যায়।

Output-এ এমন কিছু আসবে:
```
[1] 4523
```
- `[1]` = job number
- `4523` = PID


## Process-এর Lifecycle (জীবনচক্র)

```
Created (তৈরি)
    ↓
Running (চলছে) ←→ Waiting (কিছুর জন্য অপেক্ষা)
    ↓
Stopped (থামানো হয়েছে)
    ↓
Zombie (কাজ শেষ কিন্তু parent এখনো clean করেনি)
    ↓
Terminated (সম্পূর্ণ শেষ)
```

### Process States সহজ ভাষায়:

| State | Code | মানে |
|-------|------|------|
| Running | R | এখন CPU-তে কাজ করছে |
| Sleeping | S | কিছুর জন্য অপেক্ষা করছে (I/O, event) |
| Stopped | T | কেউ থামিয়ে দিয়েছে (Ctrl+Z) |
| Zombie | Z | কাজ শেষ কিন্তু parent এখনো তুলে নেয়নি |
| Idle | I | Kernel thread, কিছু করছে না |


## Commands


### `echo $$` - নিজের PID দেখা

**কী করে:** বর্তমান shell-এর PID দেখায়

```bash
echo $$
```

**Output:**
```
1842
```
> তোমার bash-এর PID হলো 1842


### `echo $PPID` - Parent PID দেখা

**কী করে:** বর্তমান shell-এর parent-এর PID দেখায়

```bash
echo $PPID
```

**Output:**
```
1801
```

### `ps` - Process দেখা

**কী করে:** বর্তমানে চলা process-গুলো দেখায়

```bash
ps
```

**Output:**
```
  PID TTY          TIME CMD
 1842 pts/0    00:00:00 bash
 1901 pts/0    00:00:00 ps
```


### `ps aux` - সব process বিস্তারিত দেখা

**Syntax breakdown:**
```
ps aux
│  ││└── x = terminal ছাড়াও চলা process দেখাও
│  │└─── u = user-friendly format (মালিক, CPU%, MEM% দেখাও)
│  └──── a = সব user-এর process দেখাও
└─────── ps = process status
```

```bash
ps aux
```

**Output (কিছু অংশ):**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169536 13100 ?        Ss   10:00   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    10:00   0:00 [kthreadd]
munir     1842  0.0  0.1  22756  5432 pts/0    Ss   10:05   0:00 bash
munir     1950  0.0  0.1  37348  3240 pts/0    R+   10:10   0:00 ps aux
```

### Column গুলোর মানে কি:

| Column | মানে |
|--------|------|
| USER | কোন user-এর process |
| PID | Process ID |
| %CPU | কতটুকু CPU ব্যবহার করছে |
| %MEM | কতটুকু Memory ব্যবহার করছে |
| STAT | Process-এর অবস্থা (R, S, Z...) |
| COMMAND | কোন command চলছে |


### `pstree` - Family Tree আকারে দেখা

এটা অনেক distro-তে বা linux-এর minimal version-এ থাকে না তখন ইন্সটল করে নিতে হয়। 

```bash 
sudo apt install psmisc
```

**কী করে:** কোন process কার থেকে জন্মেছে সেটা গাছের মতো (ট্রি আকারে) দেখায়

```bash
pstree
```

**Output (কিছু অংশ):**
```
systemd─┬─sshd───sshd───bash───pstree
        ├─cron
        ├─nginx───nginx
        └─2*[dhclient]
```

> **DevOps-এ কাজে লাগে:** কোন service কার child process সেটা দেখতে। সমস্যা diagnose করতে।


### একটা Background Process তৈরি করে দেখুন

```bash
# Background-এ একটা process রান করুন
sleep 200 &

# PID দেখাবে
# [1] 2045

# এখন confirm করুন যে process চলছে
ps aux | grep sleep
```

**Output:**

```
munir     2045  0.0  0.0  12060   576 pts/0    S    10:15   0:00 sleep 200
```


### `/proc` filesystem - Process-এর ভেতরে উঁকি মারা

Linux-এ প্রতিটা process-এর জন্য `/proc/<PID>/` নামে একটা folder তৈরি হয়।

```bash
# আপনার bash-এর PID দিয়ে দেখুন
cat /proc/$$/status
```

**Output (কিছু অংশ):**

```
Name:   bash
State:  S (sleeping)
Pid:    1842
PPid:   1801
Threads: 1
VmRSS:  5432 kB     ← RAM ব্যবহার
```

```bash
# একটা process কোন files খুলে রেখেছে
ls /proc/$$/fd
```

> **এটা অনেক powerful!** DevOps-এ কোনো process memory leak করলে বা hang করলে `/proc` দেখে diagnose করা হয়।


## Process Types - DevOps এর দৃষ্টিকোণ থেকে

| Type | উদাহরণ | বৈশিষ্ট্য |
|------|---------|-----------|
| **System Process** | systemd, kernel threads | root চালায়, সবসময় চলে |
| **Daemon Process** | nginx, sshd, cron | Background-এ চলে, terminal নেই |
| **User Process** | bash, vim, python script | User চালায় |
| **Zombie Process** | defunct process | কাজ শেষ কিন্তু এখনো আছে |
| **Orphan Process** | parent মারা গেছে | systemd adopt করে নেয় |


## Real DevOps Scenario

> **সমস্যা:** Server slow হয়ে গেছে। কে CPU খাচ্ছে?

```bash
# Step 1: সব process দেখুন CPU অনুযায়ী sort করে
ps aux --sort=-%cpu | head -10

# Step 2: সমস্যাজনক process-এর PID নিন
# Step 3: সেই PID দিয়ে বিস্তারিত দেখুন
cat /proc/<PID>/status

# Step 4: কে এই process চালু করেছে?
ps -p <PID> -o pid,ppid,user,cmd
```


## 📝 Quick Summary

- **Process** = চলমান program - RAM ও CPU ব্যবহার করে
- **PID** = প্রতিটা process-এর unique ID নম্বর
- **PPID** = Parent process-এর ID - সব process কোনো না কোনো parent থেকে জন্মায়
- **PID 1 = systemd** - সব process-এর মূল বাবা
- **Foreground** = terminal আটকায় | **Background** = terminal আটকায় না (`&` দিলে)
- **Process States:** R (running), S (sleeping), T (stopped), Z (zombie)
- `/proc/<PID>/` = প্রতিটা process-এর live তথ্য এখানে থাকে


## 🏋️ Practice Tasks

এগুলো নিজে করে দেখুন:

1. **`echo $$`** রান করুন - আপনার bash-এর PID দেখুন। তারপর **`echo $PPID`** রান করুন - parent PID দেখুন।

2. **`sleep 500 &`** দিয়ে একটা background process তৈরি করুন। তারপর **`ps aux | grep sleep`** দিয়ে confirm করুন সেটা চলছে।

3. **`cat /proc/$$/status`** রান করুন এবং `Name`, `State`, `Pid`, `PPid` এই চারটা line খুঁজে বের করুন।

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 2: Viewing Processes**
> `ps`, `top`, `htop`, `pgrep` - process দেখার সব tools বিস্তারিতভাবে শিখবো। `top` command-এর প্রতিটা column বুঝবো এবং real-time monitoring করতে শিখবো।

[Next Lesson | Viewing Processes](./02-Viewing-Processes)
