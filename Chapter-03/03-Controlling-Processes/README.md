# Chapter 3 - Lesson 3: Controlling Processes

**Chapter 3 | Lesson 3 of 10**


## 🎯 আজকের Lesson এ কী শিখবো?

আজকে আমরা শিখবো কীভাবে Linux এ process কে **control** করা যায় - মানে কোনো process কে **থামানো, বন্ধ করা, বা পুনরায় চালু করা**।

এই lesson এ cover হবে:
- **Signals** কী এবং কেন দরকার
- `kill` - PID দিয়ে signal পাঠানো
- `killall` - নাম দিয়ে সব process বন্ধ করা
- `pkill` - pattern দিয়ে process বন্ধ করা
- Real DevOps scenarios


## Signal কী? - সহজ ভাষায়

ধরুন আপনি একটা দোকানে কাজ করছেন। আপনার manager আপনাকে বিভিন্ন **ইশারা (signal)** দিতে পারে:

| Manager এর ইশারা | মানে |
|---|---|
| "একটু বিরতি নাও" | Pause করো |
| "কাজ শেষ করো" | Gracefully বন্ধ হও |
| "এখনই বের হও!" | Force বন্ধ হও |

Linux এও ঠিক একইভাবে **Kernel, process কে signal পাঠায়** এবং process সেই signal অনুযায়ী কাজ করে।


## সবচেয়ে গুরুত্বপূর্ণ Signals

| Signal Number | Signal Name | কী করে |
|---|---|---|
| 1 | `SIGHUP` | Process কে reload করে (config পুনরায় পড়ে) |
| 2 | `SIGINT` | Interrupt - `Ctrl+C` চাপলে এটাই যায় |
| 9 | `SIGKILL` | **Force kill** - কোনো উপায় নেই বাঁচার |
| 15 | `SIGTERM` | **Graceful kill** - process নিজে পরিষ্কার করে বন্ধ হয় |
| 18 | `SIGCONT` | Paused process কে আবার চালু করে |
| 19 | `SIGSTOP` | Process কে Pause করে |

**সহজ কথায়:**
> - `SIGTERM (15)` = ভদ্রভাবে বলা "ভাই, এখন বন্ধ হও"
> - `SIGKILL (9)` = জোর করে বন্ধ করা, process এর কোনো কথা শোনা হয় না

সব signal দেখতে চাইলে:
```bash
kill -l
```

**Expected Output:**
```
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
 5) SIGTRAP      6) SIGABRT      7) SIGBUS        8) SIGFPE
 9) SIGKILL     10) SIGUSR1     11) SIGSEGV      12) SIGUSR2
13) SIGPIPE     14) SIGALRM     15) SIGTERM      ...
```


## Command 1: `kill`

### এটা কী করে?
**নির্দিষ্ট PID** এর process কে signal পাঠায়।

> নাম দেখে চিন্তার কারন নাই যে এটা শুধু "মারে", এটা যেকোনো signal পাঠাতে পারে!

### Syntax:
```bash
kill [signal] <PID>
```


### উদাহরণ ১: Graceful kill (default - SIGTERM)

প্রথমে একটা process চালাই যেটা আমরা kill করবো:
```bash
# একটা process চালাও background এ
sleep 1000 &
```

**Output:**
```
[1] 4321        ← এটা PID
```

এখন সেটাকে gracefully বন্ধ করো:
```bash
kill 4321
```

Verify করো:
```bash
ps aux | grep sleep
```
**Output:**
```
[1]+  Terminated    sleep 1000
```
Process চলে গেছে!


### উদাহরণ ২: Force kill (SIGKILL)

কখনো কখনো process SIGTERM ignore করে, তখন SIGKILL ব্যবহার করুন:

```bash
sleep 1000 &
# ধরুন PID হলো 4500

kill -9 4500
# অথবা
kill -SIGKILL 4500
```

**Output:**
```
[1]+  Killed        sleep 1000
```


### উদাহরণ ৩: Config Reload (SIGHUP)

DevOps এ অনেক সময় Nginx বা Apache এর config reload করতে হয়:

```bash
# Nginx এর PID বের করুন
cat /var/run/nginx.pid

# Config reload করুন, process বন্ধ না করে!
kill -1 <nginx_pid>
# অথবা
kill -SIGHUP <nginx_pid>
```

> **DevOps Use Case:** Production এ service restart না করে শুধু config reload করতে SIGHUP ব্যবহার হয়!


### উদাহরণ ৪: Process Pause ও Resume

```bash
sleep 1000 &
# PID ধরুন 4600

# Process pause করো
kill -19 4600     # SIGSTOP

# Process resume করো
kill -18 4600     # SIGCONT
```


## Command 2: `killall`

### এটা কী করে?
**নাম দিয়ে** সব matching process কে signal পাঠায়।

> PID মনে রাখতে হবে না, শুধু process এর নাম জানলেই হবে!

### Syntax:
```bash
killall [options] <process_name>
```


### উদাহরণ ১: নাম দিয়ে kill

কয়েকটা sleep রান করুন

```bash
sleep 500 &
sleep 600 &
sleep 700 &
```

সবগুলো process একসাথে বন্ধ/kill করুন

```bash
killall sleep
```

**Output:**
```
[1]   Terminated    sleep 500
[2]   Terminated    sleep 600
[3]+  Terminated    sleep 700
```

একটা command এ সব শেষ!


### উদাহরণ ২: Force kill

```bash
killall -9 firefox
# অথবা
killall -SIGKILL firefox
```


### উদাহরণ ৩: Verbose mode - কী হচ্ছে দেখা

```bash
killall -v sleep
```

**Output:**
```
Killed sleep(4321) with signal 15
Killed sleep(4322) with signal 15
```


### উদাহরণ ৪: নির্দিষ্ট user এর process kill

```bash
killall -u munir firefox
```
শুধু `munir` user এর firefox বন্ধ হবে, অন্যদেরটা না।


### গুরুত্বপূর্ণ Options:

| Option | কী করে |
|---|---|
| `-9` | Force kill |
| `-v` | Verbose - কী হলো দেখায় |
| `-u <user>` | নির্দিষ্ট user এর process |
| `-i` | Kill করার আগে confirm চায় |
| `-r` | Regex pattern ব্যবহার |


## Command 3: `pkill`

### এটা কী করে?
`killall` এর মতোই, কিন্তু আরও **powerful**, **partial name বা pattern** দিয়ে process খুঁজে signal পাঠায়।

### Syntax:
```bash
pkill [options] <pattern>
```


### উদাহরণ ১: Partial name দিয়ে kill

```bash
# "fire" দিয়ে শুরু হওয়া সব process বন্ধ করো
pkill fire
# এটা firefox, firebird সব বন্ধ করবে
```


### উদাহরণ ২: নির্দিষ্ট user এর process

```bash
pkill -u munir
# munir এর সব process বন্ধ হবে
```


### উদাহরণ ৩: Terminal (TTY) অনুযায়ী

```bash
pkill -t pts/1
# pts/1 terminal এ চলা সব process বন্ধ
```


### উদাহরণ ৪: Signal সহ

```bash
pkill -SIGHUP nginx
# nginx এর config reload
```


### উদাহরণ ৫: Process টা আছে কিনা check করা (pgrep)

`pkill` এর "ভাই" হলো `pgrep`, এটা kill করে না, শুধু matching PID দেখায়:

```bash
pgrep sleep
```
**Output:**
```
4321
4322
4323
```

Kill করার আগে `pgrep` দিয়ে confirm করা ভালো অভ্যাস:
```bash
pgrep nginx       # আগে দেখো
pkill nginx       # তারপর kill করো
```


## kill vs killall vs pkill - তুলনা

| বিষয় | `kill` | `killall` | `pkill` |
|---|---|---|---|
| কীভাবে target করে | PID দিয়ে | exact নাম দিয়ে | pattern/partial নাম দিয়ে |
| একাধিক process? | না (একটা PID) | হ্যাঁ | হ্যাঁ |
| User filter? | না | হ্যাঁ (-u) | হ্যাঁ (-u) |
| Regex support? | না | হ্যাঁ (-r) | হ্যাঁ (default) |
| কখন ব্যবহার | নির্দিষ্ট process | exact নাম জানলে | flexible search |


## SIGTERM vs SIGKILL - কোনটা কখন?

SIGTERM (15) ব্যবহার করবেন যখন:
- Process কে gracefully বন্ধ করতে চান
- Process যেন open file, connection ঠিকমতো বন্ধ করতে পারে
- Data loss এড়াতে চান
- Production environment এ

SIGKILL (9) ব্যবহার করবেন যখন:
- Process SIGTERM ignore করছে
- Process hang/freeze হয়ে গেছে
- Zombie process (এখানে কাজ নাও করতে পারে!)
- Emergency তে


> ⚠️ **সতর্কতা:** `kill -9` সবসময় ব্যবহার করা যাবে না! এতে database corruption বা data loss হতে পারে। সবসময় আগে `kill -15` try করুন।


## Zombie Process কী?

একটু বোনাস জ্ঞান! Zombie process হলো এমন process যেটা:
- কাজ শেষ করেছে কিন্তু **process table থেকে সরেনি**
- Parent process তাকে "collect" করেনি
- `ps` এ `Z` status দেখাবে

```bash
ps aux | grep Z
```

Zombie কে `kill -9` ও মারতে পারবে না! Parent process কে kill করতে হয়।


## Real DevOps Scenarios

### Scenario 1: Nginx reload (zero downtime)
```bash
# PID বের করুন
cat /var/run/nginx.pid    # অথবা
pgrep nginx

# Reload - traffic বন্ধ না করে!
kill -HUP $(cat /var/run/nginx.pid)
```

### Scenario 2: Hung process বন্ধ করা
```bash
# আগে gracefully try করুন
kill -15 <PID>

# ৫ সেকেন্ড অপেক্ষা করুন, তারপর check করুন
sleep 5
ps -p <PID>

# এখনো আছে? Force kill:
kill -9 <PID>
```

### Scenario 3: একটা user এর সব process বন্ধ
```bash
# একজন employee এর access revoke করতে হবে
pkill -u baduser
# বা
killall -u baduser
```

### Scenario 4: Script দিয়ে graceful shutdown
```bash
PID=$(pgrep myapp)
kill -15 $PID
sleep 10
if kill -0 $PID 2>/dev/null; then
    echo "Still running, force killing..."
    kill -9 $PID
fi
```
> `kill -0` কোনো signal পাঠায় না, শুধু process আছে কিনা check করে!


## 📝 Quick Summary

- **Signal** হলো Linux এ process কে message পাঠানোর উপায়
- **SIGTERM (15)** = ভদ্রভাবে বন্ধ করো - সবসময় এটা আগে try করো
- **SIGKILL (9)** = জোর করে বন্ধ - শেষ অস্ত্র
- **SIGHUP (1)** = Reload করো - config change এ কাজে লাগে
- **`kill`** = PID দিয়ে signal পাঠায়
- **`killall`** = exact নাম দিয়ে সব matching process কে signal পাঠায়
- **`pkill`** = pattern/partial নাম দিয়ে signal পাঠায়
- **`pgrep`** = kill না করে শুধু matching PID দেখায়
- Production এ সবসময় SIGTERM আগে, তারপর SIGKILL


## 🏋️ Practice Tasks

**Task 1:**
```
- Terminal এ `sleep 9999 &` কমান্ড দিয়ে ৩টা process রান করুন
- `ps aux | grep sleep` দিয়ে PID গুলো দেখুন
- একটাকে `kill` দিয়ে, বাকিগুলো `killall` দিয়ে বন্ধ করুন
- Verify করুন যে সব বন্ধ হয়েছে
```

**Task 2:**
```
- `sleep 5000 &` রান করুন
- `kill -19 <PID>` দিয়ে pause করুন
- `ps aux | grep sleep` দিয়ে status দেখুন (T দেখাবে)
- `kill -18 <PID>` দিয়ে resume করুন
- আবার status check করুন
```

**Task 3:**
```
- `pgrep sleep` দিয়ে running sleep process এর PID দেখুন
- `pkill -v sleep` দিয়ে verbose mode এ সব বন্ধ করুন
- `kill -l` দিয়ে সব signal এর list দেখুন
```

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 4: Background & Foreground Jobs**
`bg`, `fg`, `jobs`, `nohup`, `&` কীভাবে process কে background এ পাঠানো যায়, terminal বন্ধ করলেও process চালু রাখা যায়, এবং jobs manage করা যায়!


আপনি এখন process control এর master হয়ে যাচ্ছেন! Practice tasks গুলো করুন এবং কোনো সমস্যা হলে error কমেন্টে দিন, আমি ইনশাআল্লাহ solve করে দিবো। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../02-Viewing-Processes">← Viewing Processes</a>
    </td>
    <td align="right">
      <a href="../04-Background-Foreground-Jobs">Background &amp; Foreground Jobs →</a>
    </td>
  </tr>
</table>
