# Chapter 3 - Lesson 4: Background & Foreground Jobs

**Chapter 3 | Lesson 4 of 10**


## 🎯 এই Lesson-এ আমরা যা শিখবো

Linux-এ যখন আপনি কোনো command run করুন, সেটা default-এ **foreground**-এ চলে, মানে terminal টা block হয়ে যায়। কিন্তু DevOps কাজে আপনাকে অনেক সময় **একসাথে অনেক কাজ** চালাতে হয়। সেজন্যই দরকার **Job Control**।

এই lesson শেষে আমরা যা করতে পারবো:
- Terminal block না করে background-এ process চালাতে
- Background process-কে আবার সামনে আনতে
- Server বন্ধ হলেও process চালু রাখতে (`nohup`)
- সব running job দেখতে


## Foreground vs Background

### Real-World Analogy:

আগে Concept বোঝার চেস্টা করি। কল্পনা করুন আপনি একটা **রেস্তোরাঁর manager**।

| পরিস্থিতি | Linux-এ |
|---|---|
| আপনি নিজে কাউন্টারে দাঁড়িয়ে সরাসরি order নিচ্ছোন | **Foreground process** - terminal ব্লক |
| আপনি রান্নাঘরে কাউকে রান্না করতে পাঠিয়েছেন, নিজে অন্য কাজ করছেন | **Background process** - terminal ফ্রি |
| রেস্তোরাঁ বন্ধ হলেও রান্না চলতে থাকে | **nohup** - terminal বন্ধ হলেও চলে |


## Part 1: `&` - Background-এ Process চালানো

### এটা কী করে?
Command-এর শেষে `&` দিলে সেটা **background-এ চলে যায়** এবং terminal আপনার জন্য free থাকে।

### Syntax:
```bash
command &
```

### Example:
```bash
sleep 100 &
```

### Expected Output:
```
[1] 4321
```

এখানে:
- `[1]` = এটা job number **1**
- `4321` = এই process-এর **PID** (Process ID)

Terminal এখন free! আপনি অন্য command দিতে পারবেন।

### আরেকটি Example:
```bash
ping google.com > /tmp/ping_log.txt &
```
> ping চলবে background-এ, output যাবে একটা file-এ। Terminal ফ্রি থাকবে।


## Part 2: `jobs` - সব Background Job দেখা

### এটা কী করে?
বর্তমান terminal session-এ কতগুলো job background/stopped আছে সেটা দেখায়।

### Syntax:
```bash
jobs
jobs -l   # PID সহ দেখায়
```

### Example:
```bash
sleep 100 &
sleep 200 &
jobs
```

### Expected Output:
```
[1]-  Running                 sleep 100 &
[2]+  Running                 sleep 200 &
```

এখানে:
- `[1]`, `[2]` = Job number
- `+` = সবচেয়ে সাম্প্রতিক (current) job
- `-` = তার আগেরটা
- `Running` = এখন চলছে

### `jobs -l` output:
```
[1]-  4321 Running                 sleep 100 &
[2]+  4322 Running                 sleep 200 &
```
> `-l` দিলে PID-ও দেখায়।


## Part 3: `fg` - Background থেকে Foreground-এ আনা

### এটা কী করে?
Background-এ থাকা কোনো job-কে আবার **foreground-এ** নিয়ে আসে।

### Syntax:
```bash
fg          # সবচেয়ে recent job-কে foreground-এ আনে
fg %1       # Job number 1 কে foreground-এ আনে
fg %2       # Job number 2 কে foreground-এ আনে
```

### Example:
```bash
sleep 100 &    # background-এ দিলাম
jobs            # দেখলাম [1] আছে
fg %1           # আবার foreground-এ আনলাম
```

এখন terminal আবার block হয়ে যাবে কারণ `sleep 100` foreground-এ চলছে।

### বের হতে চাইলে?
`Ctrl + Z` চাপুন, এটা process-কে **pause (suspend)** করে দেবে।


## Part 4: `Ctrl+Z` - Process Pause করা

### এটা কী করে?
Foreground-এ চলা process-কে **সাময়িকভাবে থামিয়ে (suspend)** দেয়। Process মরে না, শুধু pause হয়।

### Example:
```bash
sleep 100        # foreground-এ চলছে
# এখন Ctrl+Z চাপুন
```

### Expected Output:
```
^Z
[1]+  Stopped                 sleep 100
```

> `Stopped` মানে process pause আছে, চলছে না।


## Part 5: `bg` - Paused Process-কে Background-এ চালানো

### এটা কী করে?
`Ctrl+Z` দিয়ে যে process pause করেছেন, সেটাকে **background-এ resume** করে।

### Syntax:
```bash
bg          # most recent stopped job resume করে
bg %1       # job number 1 resume করে background-এ
```

### Example:
```bash
sleep 100        # foreground-এ চলছে
# Ctrl+Z চাপলাম - process stopped হলো
bg %1            # background-এ resume করলাম
```

### Expected Output:
```
[1]+ sleep 100 &
```

> এখন `sleep 100` background-এ চলছে, terminal ফ্রি।


## Full Flow একসাথে:

```bash
# Step 1: foreground-এ চালু করুন
sleep 300

# Step 2: Ctrl+Z চাপুন
^Z
[1]+  Stopped    sleep 300

# Step 3: background-এ পাঠিয়ে দিন
bg %1
[1]+ sleep 300 &

# Step 4: সব job দেখুন
jobs
[1]+  Running    sleep 300 &

# Step 5: আবার foreground-এ আনুন
fg %1
sleep 300

# Step 6: Ctrl+C দিয়ে kill করুন (চাইলে)
^C
```


## Part 6: `nohup` - Terminal বন্ধ হলেও Process চালু রাখুন

### Analogy:
আপনি office থেকে বাসায় চলে গেলেও আপনার computer-এ download চলতে থাকে, এটাই `nohup`-এর কাজ।

### সমস্যাটা কী?
Normally, আপনি terminal বন্ধ করলে বা SSH session disconnect হলে, সেই terminal-এর সব process **মারা যায়** (`SIGHUP` signal পায়)।

DevOps-এ এটা বিপজ্জনক! আপনি server-এ `deploy.sh` রান করেছেন, হঠাৎ internet অফ হয়ে গেলে, সব শেষ!

### `nohup` এই সমস্যা সমাধান করে।

### Syntax:
```bash
nohup command &
nohup command > output.log 2>&1 &
```

### Example 1 - Basic:
```bash
nohup sleep 500 &
```

### Expected Output:
```
[1] 5678
nohup: ignoring input and appending output to 'nohup.out'
```

> Output automatically `nohup.out` file-এ যাবে।

### Example 2 - Custom output file:
```bash
nohup bash deploy.sh > /tmp/deploy.log 2>&1 &
```

এখানে:
- `nohup` = terminal বন্ধ হলেও চালু থাকবে
- `bash deploy.sh` = এই script চালাও
- `> /tmp/deploy.log` = stdout এই file-এ যাবে
- `2>&1` = stderr-ও একই file-এ যাবে
- `&` = background-এ চালাও

### Output দেখতে:
```bash
tail -f /tmp/deploy.log
```

### Process check করতে:
```bash
ps aux | grep deploy.sh
```


## Part 7: `disown` - Running Process-কে Terminal থেকে আলাদা করা

### এটা কী করে?
ইতোমধ্যে background-এ চলা process-কে terminal থেকে **detach** করে দেয়। মানে terminal বন্ধ হলেও process চলবে।

> `nohup` দিতে ভুলে গেছেন? `disown` দিয়ে সরিয়ে ফেলুন!

### Syntax:
```bash
disown %1        # job 1 কে disown করো
disown -a        # সব job disown করো
```

### Example:
```bash
sleep 1000 &          # background-এ দিলাম
jobs                   # [1] Running দেখাচ্ছে
disown %1              # terminal থেকে detach করলাম
jobs                   # এখন আর দেখাবে না, কিন্তু process চলছে
```


## `nohup` vs `disown` vs `&` - পার্থক্য

| Feature | `&` | `nohup` | `disown` |
|---|---|---|---|
| Background-এ চলে? | ✅ | ✅ | ✅ |
| Terminal বন্ধ হলে বাঁচে? | ❌ | ✅ | ✅ |
| Output file তৈরি করে? | ❌ | ✅ (`nohup.out`) | ❌ |
| Already running process-এ কাজ করে? | ❌ | ❌ | ✅ |


## DevOps Real-World Use Cases

### Use Case 1: Long Deployment Script
```bash
nohup bash /opt/scripts/deploy.sh > /var/log/deploy.log 2>&1 &
echo "Deploy started. PID: $!"
```
> `$!` = সবচেয়ে শেষে run হওয়া background process-এর PID

### Use Case 2: Log Monitoring
```bash
tail -f /var/log/nginx/access.log &
jobs   # দেখো চলছে কিনা
```

### Use Case 3: Server-এ SSH করে কাজ করা
```bash
# SSH করলে
nohup python3 /opt/app/server.py > /tmp/app.log 2>&1 &
# এখন SSH disconnect করুন, কিন্তু app চলতে থাকবে
```

### Use Case 4: Multiple Jobs চালানো
```bash
nohup bash backup.sh > backup.log 2>&1 &
nohup bash cleanup.sh > cleanup.log 2>&1 &
jobs   # দুটো job চলছে দেখাবে
```


## ⚠️ Common Mistakes

### ❌ Mistake 1: `nohup` ছাড়া SSH-এ script চালানো
```bash
# ভুল!
bash deploy.sh &
# internet গেলে = deploy বন্ধ
```

```bash
# সঠিক!
nohup bash deploy.sh > deploy.log 2>&1 &
```

### ❌ Mistake 2: `2>&1` না দেওয়া
```bash
# শুধু stdout capture হবে, error হারিয়ে যাবে
nohup bash script.sh > output.log &

# সঠিক - stdout + stderr দুটোই capture
nohup bash script.sh > output.log 2>&1 &
```

### ❌ Mistake 3: Job number ভুল করা
```bash
fg 1     # ❌ ভুল syntax
fg %1    # ✅ সঠিক - % চাই
```


## সব Commands এক নজরে

| Command | কী করে |
|---|---|
| `command &` | Background-এ চালায় |
| `jobs` | সব background/stopped job দেখায় |
| `jobs -l` | PID সহ job দেখায় |
| `fg %n` | Job n কে foreground-এ আনে |
| `bg %n` | Stopped job n কে background-এ resume করে |
| `Ctrl + Z` | Foreground process pause করে |
| `Ctrl + C` | Foreground process kill করে |
| `nohup cmd &` | Terminal বন্ধ হলেও চালায় |
| `disown %n` | Running job কে terminal থেকে detach করে |
| `$!` | Last background process-এর PID |


## 📝 Quick Summary

- **`&`** দিলে process background-এ যায়, terminal free থাকে
- **`jobs`** দিয়ে সব background/stopped job দেখা যায়
- **`Ctrl+Z`** দিয়ে process pause করা যায়
- **`bg`** দিয়ে paused process background-এ resume করা যায়
- **`fg`** দিয়ে background process foreground-এ আনা যায়
- **`nohup`** দিলে terminal/SSH বন্ধ হলেও process চলে, DevOps-এ অত্যন্ত গুরুত্বপূর্ণ
- **`disown`** দিয়ে already running process কে terminal থেকে detach করা যায়


## 🏋️ Practice Tasks

**Task 1:**
```bash
sleep 500 &
sleep 600 &
jobs -l
# দুটো job-এর PID নোট করুন

fg %1
# তারপর Ctrl+Z চাপুন

bg %1
jobs
```

**Task 2:**
```bash
# একটা script তৈরি যেটা 60 seconds loop করে log লেখে
while true; do echo "$(date): running"; sleep 5; done > /tmp/test.log &

# তারপর দেখুন:
tail -f /tmp/test.log
# Ctrl+C দিয়ে tail বন্ধ করুন, কিন্তু background job চলছে কিনা দেখুন
jobs
```

**Task 3:**
```bash
# nohup দিয়ে চালাও
nohup bash -c 'for i in {1..20}; do echo "Step $i"; sleep 2; done' > /tmp/nohup_test.log 2>&1 &
# PID নোট করুন
# তারপর log দেখুন
tail -f /tmp/nohup_test.log
```

---

## ⏭️ What's Next?

**Chapter 3 - Lesson 5: Process Priority**

`nice` এবং `renice` দিয়ে কোন process বেশি বা কম CPU পাবে সেটা control করতে শিখবো। DevOps-এ critical service-কে priority দেওয়া একটা গুরুত্বপূর্ণ skill! *Happy Learning* 🚀
