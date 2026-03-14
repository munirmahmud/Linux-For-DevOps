# Chapter 3 - Lesson 5: Process Priority (nice, renice)

**Chapter 3 | Lesson 5 of 10**


## 🎯 এই Lesson-এ আমরা যা শিখবো

- Process Priority কী এবং কেন দরকার?
- Linux কীভাবে CPU time ভাগ করে?
- `nice` দিয়ে নতুন process শুরু করা
- `renice` দিয়ে চলমান process-এর priority পরিবর্তন করা
- DevOps-এ এটা কোথায় কাজে লাগে?


## Process Priority কী? - সহজ ভাষায়

মনে করেন আপনি একটি restaurant-এর manager। রান্নাঘরে একজনই chef আছে। একসাথে অনেক order এসেছে:

- Table 1: VIP guest-এর order
- Table 2: Normal guest-এর order
- Table 3: Staff-এর খাবার

আপনি কি সবাইকে **সমান priority** দিবেন? না! VIP-কে আগে serve করবেন, এরপর Normal guest কে serve করবেন।

Linux-এও ঠিক এটাই হয়। CPU হলো সেই **একজন chef**, আর processes হলো **orders**। Linux kernel decide করে কোন process আগে CPU পাবে, এটাই হলো **Process Priority**।


## Nice Value কী?

Linux-এ priority নির্ধারণ হয় **"Nice Value"** দিয়ে।

```
Nice Value Range: -20 থেকে +19
```

| Nice Value | অর্থ | কে ব্যবহার করে |
|------------|------|----------------|
| **-20** | সর্বোচ্চ priority (সবচেয়ে আগে CPU পাবে) | শুধু root user |
| **0** | Default priority (সবার জন্য) | সব user |
| **+19** | সর্বনিম্ন priority (সবার পরে CPU পাবে) | সব user |


**মনে রাখার trick:** Nice value যত **কম (negative)**, priority তত **বেশি**।

> যত **বেশি (positive)**, priority তত **কম**।
>
> মানে, "nice" মানে হলো অন্যদের প্রতি ভদ্র - নিজে কম নেওয়া! 😄


## Priority কীভাবে দেখবো?

### `ps` দিয়ে দেখা:

```bash
ps -eo pid,ni,pri,comm
```

**Output:**
```
  PID  NI PRI COMMAND
    1   0  19 systemd
  874   0  19 sshd
 1200  10   9 backup.sh
 1350  -5  24 nginx
```

**প্রতিটি column মানে:**
- `PID` → Process ID
- `NI` → Nice value (এটাই আমরা control করি)
- `PRI` → Actual kernel priority (Linux নিজে calculate করে)
- `COMMAND` → Process-এর নাম

### `top` দিয়ে দেখা:

```bash
top
```

`top` রান করলে **NI** column-এ nice value দেখতে পাবে।

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1200 deploy    20   0  123456  12345   1234 R  45.0   2.1   0:30.00 backup.sh
  874 root      20   0   89012   8901    890 S   5.0   0.5   1:20.00 sshd
```


## Command 1: `nice` - নতুন Process শুরু করা নির্দিষ্ট Priority দিয়ে

### সহজ ভাষায়:
আপনি যখন কোনো program রান করেন, তখন থেকেই তার priority set করতে চাইলে `nice` ব্যবহার করুন।

### Syntax:
```bash
nice -n <nice_value> <command>
```

| Part | মানে |
|------|------|
| `nice` | priority set করার command |
| `-n` | "number" - কত nice value দিতে চাই |
| `<nice_value>` | -20 থেকে +19 এর মধ্যে যেকোনো সংখ্যা |
| `<command>` | কোন program চালাতে চাই |


### Example 1: কম priority দিয়ে একটি script চালানো

```bash
nice -n 10 bash backup.sh
```

**মানে:** `backup.sh` script টি রান করো, কিন্তু তার priority কম রাখো (nice value = 10)। অন্য important process-গুলো আগে CPU পাবে।


### Example 2: Default nice value দেখা

```bash
nice
```

**Output:**
```
0
```

এর মানে, default nice value হলো `0`।


### Example 3: একটি process চালিয়ে তার priority দেখা

```bash
nice -n 15 sleep 100 &
ps -eo pid,ni,comm | grep sleep
```

**Output:**
```
[1] 2345
 2345  15 sleep
```

এখানে `sleep` process-টির nice value `15`, মানে এটা সবার শেষে CPU পাবে।


### Example 4: High priority দেওয়া (শুধু root পারবে)

```bash
sudo nice -n -10 bash critical_task.sh
```

**মানে:** `critical_task.sh` কে high priority দাও, সে অন্যদের আগে CPU পাবে।

> ⚠️ **Important:** Negative nice value শুধুমাত্র `root` বা `sudo` দিয়ে দেওয়া যায়।
> Normal user শুধু `0` থেকে `+19` পর্যন্ত দিতে পারবে।


## Command 2: `renice` - চলমান Process-এর Priority পরিবর্তন

### সহজ ভাষায়:

মনে করেন একটি process আগেই চলছে, এখন আপনি তার priority পরিবর্তন করতে চান। `nice` দিয়ে আর হবে না - এখন দরকার `renice`।

### Syntax:
```bash
renice -n <nice_value> -p <PID>
```

| Part | মানে |
|------|------|
| `renice` | চলমান process-এর priority পরিবর্তন করো |
| `-n` | নতুন nice value |
| `<nice_value>` | যে value দিতে চাই |
| `-p` | "PID" দিয়ে process identify করো |
| `<PID>` | Process ID নম্বর |


### Example 1: একটি process-এর priority কমানো

প্রথমে একটি process চালাই:
```bash
sleep 200 &
```

**Output:**
```
[1] 3456
```

এখন PID `3456` এর priority পরিবর্তন করি:
```bash
renice -n 15 -p 3456
```

**Output:**
```
3456 (process ID) old priority 0, new priority 15
```

এখন verify করি:
```bash
ps -eo pid,ni,comm | grep sleep
```

**Output:**
```
 3456  15 sleep
```

Priority সফলভাবে পরিবর্তন হয়েছে!


### Example 2: Process-এর priority বাড়ানো (root দিয়ে)

```bash
sudo renice -n -5 -p 3456
```

**Output:**
```
3456 (process ID) old priority 15, new priority -5
```

---

### Example 3: একটি পুরো User-এর সব process-এর priority পরিবর্তন

```bash
sudo renice -n 10 -u deploy
```

**মানে:** `deploy` user-এর সমস্ত process-এর nice value `10` করে দাও।

**Output:**
```
1200 (user ID) old priority 0, new priority 10
1201 (user ID) old priority 0, new priority 10
```


### Example 4: একটি Group-এর priority পরিবর্তন

```bash
sudo renice -n 5 -g developers
```

**মানে:** `developers` group-এর সব process-এর priority পরিবর্তন করো।


## `nice` VS `renice` - পার্থক্য

| বিষয় | `nice` | `renice` |
|-------|--------|---------|
| **কখন ব্যবহার** | Process শুরু করার সময় | Process চলার সময় |
| **Target** | নতুন command/process | চলমান PID |
| **User filter** | নেই | `-u` দিয়ে user অনুযায়ী |
| **Group filter** | নেই | `-g` দিয়ে group অনুযায়ী |
| **Syntax** | `nice -n VALUE CMD` | `renice -n VALUE -p PID` |


## Real-World DevOps Scenario

### Scenario 1: Backup script যেন Production কে affect না করে

```bash
# রাত ২টায় backup চলে, কিন্তু সে CPU নিয়ে নিলে web server slow হবে
# তাই backup-কে কম priority দাও
nice -n 19 bash /opt/scripts/nightly_backup.sh
```

### Scenario 2: হঠাৎ দেখলে একটি log-processing script সব CPU নিয়ে নিয়েছে

```bash
# আগে PID খুঁজুন
pgrep -l log_processor
# Output: 4567 log_processor

# এখন priority কমিয়ে দিন
renice -n 15 -p 4567
```

### Scenario 3: Crontab-এ কম priority দিয়ে task চালানো

```bash
# /etc/crontab বা crontab -e তে এভাবে লিখবেন:
0 2 * * * root nice -n 19 /opt/scripts/heavy_task.sh
```

### Scenario 4: Database backup চলাকালীন API server-কে high priority দেওয়া

```bash
# API server-এর PID খুঁজুন
pgrep -l api_server
# Output: 890 api_server

# High priority দিন
sudo renice -n -10 -p 890
```


## Bonus: `top` দিয়ে Interactively Priority পরিবর্তন

`top` command চালু থাকা অবস্থায়:

1. **`r`** চাপুন → "Renice a process" mode চালু হবে
2. PID enter করুন
3. নতুন nice value enter করুন

এটা directly terminal থেকে graphically priority পরিবর্তনের সহজ উপায়!


## Priority-র পুরো চিত্র এক নজরে

```
Nice Value:  -20    -10     0     +10    +19
              |      |      |      |      |
Priority:   HIGH   HIGH  NORMAL  LOW   LOWEST
              |      |      |      |      |
কে দিতে পারে: root  root  সবাই  সবাই  সবাই
```


## 📝 Quick Summary

- **Nice Value** হলো process-এর priority নির্ধারক, range `-20` (highest) থেকে `+19` (lowest)
- **Default nice value** হলো `0`
- **`nice`** ব্যবহার করুন নতুন process শুরু করার সময় priority set করতে
- **`renice`** ব্যবহার করুন চলমান process-এর priority পরিবর্তন করতে
- **Negative value** শুধু `root` দিতে পারে
- `renice`-এ `-u` দিয়ে পুরো user-এর এবং `-g` দিয়ে পুরো group-এর priority পরিবর্তন করা যায়
- DevOps-এ backup, log processing, heavy tasks-এ এটা ব্যাপকভাবে ব্যবহৃত হয়


## 🏋️ Practice Tasks

**Task 1:**
```bash
# একটি sleep process রান করুন background-এ nice value 10 দিয়ে
# তারপর ps দিয়ে verify করুন যে nice value সঠিক আছে কিনা
```

**Task 2:**
```bash
# একটি sleep process রান করুন (background-এ)
# pgrep দিয়ে তার PID খুঁজুন
# renice দিয়ে তার priority পরিবর্তন করুন nice value 18-তে
# ps দিয়ে confirm করুন
```

**Task 3:**
```bash
# top রান করুন
# 'r' চাপো
# যেকোনো একটি process-এর nice value পরিবর্তন করুন
# পরিবর্তন হয়েছে কিনা NI column-এ দেখো
```

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 6: Memory Monitoring**

> `free`, `vmstat`, `/proc/meminfo`, `smem` দিয়ে Linux-এর memory কীভাবে monitor করতে হয় - DevOps-এ memory leak ধরা এবং performance optimize করার real skills শিখবো! *Happy Learning* 🚀
