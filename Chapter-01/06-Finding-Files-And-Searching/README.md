# Chapter 1 - Lesson 6: Finding Files & Searching

**Chapter 1 | Lesson 6 of 10**


## এই Lesson-এ কী শিখবো?

Linux-এ হাজার হাজার file থাকে। সঠিক file খুঁজে বের করা একটা গুরুত্বপূর্ণ skill। আজকে আমরা শিখবো:

- `find` - file খোঁজার একটা শক্তিশালী tool
- `locate` - দ্রুত file খোঁজার tool
- `which` - কোন command কোথায় আছে তা বের করা
- `whereis` - command-এর সব related file খোঁজা


ধরুন আপনি একটা বড় library-তে আছো।

| Tool | Analogy |
|------|---------|
| `find` | প্রতিটা shelf নিজে হেঁটে হেঁটে বই খোঁজা, সময় লাগে কিন্তু সবচেয়ে accurate |
| `locate` | Library-র index card দেখা, অনেক দ্রুত কিন্তু পুরনো হতে পারে |
| `which` | জিজ্ঞেস করা "এই বইটা কোন section-এ রাখা আছে?" |
| `whereis` | জিজ্ঞেস করা "এই বইটা কোথায়, এর manual কোথায়, source কোথায়?" |


## 1️⃣ `find` Command

`find` হলো Linux-এর সবচেয়ে powerful file searching tool। এটা real-time এ disk scan করে।

### Basic Syntax:
```bash
find [কোথায় খুঁজবো] [কী দিয়ে খুঁজবো] [কী করবো]
```


### নাম দিয়ে খোঁজা (`-name`)

```bash
find /home -name "hello.txt"
```

**কী করে:** `/home` directory-র ভেতরে `hello.txt` নামের file খোঁজে।

**Expected Output:**
```
/home/user/hello.txt
```


### Case-Insensitive নামে খোঁজা (`-iname`)

```bash
find /home -iname "hello.txt"
```

**কী করে:** `hello.txt`, `Hello.txt`, `HELLO.TXT` - সব match করবে।


### Wildcard দিয়ে খোঁজা

```bash
find /var/log -name "*.log"
```

**কী করে:** `/var/log` ডিরেকটরির সব `.log` extension-এর file খোঁজে।

**Expected Output:**
```
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
```

> **Use Case:** Server-এ সব log file একসাথে খুঁজে বের করতে এটা daily use হয়।


### File Type দিয়ে খোঁজা (`-type`)

```bash
find /etc -type f -name "*.conf"
```

| Type Flag | মানে |
|-----------|------|
| `-type f` | শুধু regular file |
| `-type d` | শুধু directory |
| `-type l` | শুধু symbolic link |

**কী করে:** `/etc` ডিরেকটরির শুধু `.conf` extension-এর file (directory না) খোঁজে।

**Expected Output:**
```
/etc/resolv.conf
/etc/nsswitch.conf
/etc/host.conf
```

### Size দিয়ে খোঁজা (`-size`)

```bash
find /var -size +100M
```

**কী করে:** `/var` ডিরেকটরিতে ১০০MB-এর বড় সব file খোঁজে।

| Expression | মানে |
|------------|------|
| `+100M` | ১০০MB-এর **বেশি** |
| `-10k` | ১০KB-এর **কম** |
| `50c` | ঠিক ৫০ bytes |

> **Use Case:** Disk full হয়ে গেলে বড় file গুলো খুঁজে বের করার প্রয়োজন পড়ে।


### Modified Time দিয়ে খোঁজা (`-mtime`)

```bash
find /home -mtime -7
```

**কী করে:** গত ৭ দিনের মধ্যে modify হওয়া সব file খোঁজে।

| Flag | মানে |
|------|------|
| `-mtime -7` | গত ৭ দিনের মধ্যে modified |
| `-mtime +30` | ৩০ দিনের বেশি আগে modified |
| `-atime -1` | গত ২৪ ঘন্টায় accessed |

> **Use Case:** Recently changed config file কোনটা সেটা বের করতে প্রয়োজন পড়ে।


### Permission দিয়ে খোঁজা (`-perm`)

```bash
find / -perm 777
```

**কী করে:** সব জায়গায় `777` permission-এর file খোঁজে (সবাই read/write/execute করতে পারে, তথা security risk!)

> **Use Case:** Security audit করার সময় dangerous permission-এর file বের করতে ব্যাবহার করা হয়।


### Owner দিয়ে খোঁজা (`-user`, `-group`)

```bash
find /home -user munir
```

**কী করে:** `munir` user-এর সব file খোঁজে।

```bash
find /var -group www-data
```

**কী করে:** `www-data` group-এর সব file খোঁজে।


### Find + Execute (`-exec`)

এটা `find`-এর সবচেয়ে শক্তিশালী feature। খোঁজার পরেই কাজ করা যায়।

```bash
find /tmp -name "*.tmp" -exec rm {} \;
```

**কী করে:** `/tmp` ডিরেক্টরির সব `.tmp` file খুঁজে একে একে delete করে ফেলে।

**Syntax ব্যাখ্যা:**
- `{}` → খুঁজে পাওয়া প্রতিটা file-এর জায়গায় বসে
- `\;` → command শেষ হওয়ার চিহ্ন

আরেকটা উদাহরণ:
```bash
find /etc -name "*.conf" -exec ls -lh {} \;
```

**কী করে:** সব `.conf` file খুঁজে তাদের detail দেখায়।

> **Use Case:** পুরনো log file খুঁজে automatically delete করা, এটা automation-এ খুব কাজে লাগে।


### Multiple Conditions (`-and`, `-or`, `-not`)

```bash
find /var -name "*.log" -and -size +1M
```
**কী করে:** `.log` file যেগুলো ১MB-এর বেশি।

```bash
find / -name "*.jpg" -or -name "*.png"
```
**কী করে:** `.jpg` অথবা `.png` file।

```bash
find /tmp -not -name "*.log"
```
**কী করে:** `.log` ছাড়া বাকি সব file।


## 2️⃣ `locate` Command

`locate` অনেক দ্রুত কাজ করে কারণ এটা একটা pre-built **database** থেকে খোঁজে, disk scan করে না।

### Syntax:
```bash
locate filename
```

### উদাহরণ:
```bash
locate passwd
```

**Expected Output:**
```
/etc/passwd
/etc/passwd-
/usr/share/doc/passwd
/usr/bin/passwd
```


### Case-Insensitive খোঁজা:
```bash
locate -i readme
```


### কেন কাজ করছে না?

যদি `locate` এবং `updatedb` কাজ না করে তারমানে আলাদাভাবে install করতে হবে। এটা default-এ সব distro-তে থাকে না। এই দুটো tool একটা package-এর ভেতরে থাকে **`mlocate`** অথবা **`plocate`**।

## Debian / Ubuntu-তে Install করা

```bash
sudo apt update
sudo apt install plocate -y
```

Install হওয়ার পরে database update করুন:

```bash
sudo updatedb
```

তারপর test করুন:

```bash
locate passwd
```

**Expected Output:**
```
/etc/passwd
/etc/passwd-
/usr/bin/passwd
```

## Red Hat / CentOS / Fedora / Rocky Linux-এ Install করা

নতুন Red Hat / Rocky Linux / AlmaLinux / Fedora-তে mlocate এর জায়গায় এখন plocate ব্যবহার করা হয়।

```bash
sudo dnf install mlocate -y
# OR
sudo dnf install plocate -y
```

Install হওয়ার পরে database update করুন:

```bash
sudo updatedb
```

| Distro | Command |
|--------|---------|
| Debian / Ubuntu | `sudo apt install plocate -y` |
| Red Hat / CentOS 8+ / Rocky | `sudo dnf install mlocate -y` |
| সবখানে update | `sudo updatedb` |


### Database Update করা:

`locate`-এর database রাতে auto-update হয়। কিন্তু নতুন file তৈরি করলে সাথে সাথে দেখাবে না। তখন manually update করতে হয়:

```bash
sudo updatedb
```

**তারপর:**
```bash
locate newfile.txt
```

> **Important:** নতুন file তৈরি করার পর `updatedb` না চালালে `locate` সেই file খুঁজে পাবে না।


### `find` vs `locate` তুলনা:

| বিষয় | `find` | `locate` |
|-------|--------|---------|
| Speed | ধীর (disk scan) | দ্রুত (database) |
| Accuracy | সবসময় accurate | পুরনো হতে পারে |
| Real-time | ✅ হ্যাঁ | ❌ না |
| Filter options | অনেক বেশি | কম |
| Use case | Precise search | Quick search |


## 3️⃣ `which` Command

কোন command কোথায় installed আছে তা বের করতে `which` ব্যবহার করা হয়।

### Syntax:

```bash
which command_name
```

### উদাহরণ:

```bash
which python3
```

**Expected Output:**
```
/usr/bin/python3
```

```bash
which nginx
```

**Expected Output:**
```
/usr/sbin/nginx
```

```bash
which git
```

**Expected Output:**
```
/usr/bin/git
```

যদি command installed না থাকে:
```bash
which docker
```

**Expected Output:**
```
(কিছুই দেখাবে না অথবা "docker not found") Distro ভেদে রেসপন্স ভিন্ন হতে পারে।
```

> **Use Case:** Script লেখার সময় দেখতে হয় কোনো tool installed আছে কিনা এবং কোথায় আছে। যেমন: `which ansible`, `which terraform`।


## 4️⃣ `whereis` Command

`whereis` আরো বেশি তথ্য দেয়। শুধু binary না, manual page এবং source code-ও কোথায় আছে বলে দেয়।

### Syntax:
```bash
whereis command_name
```

### উদাহরণ:

```bash
whereis ls
```

**Expected Output:**
```
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

**ব্যাখ্যা:**
- `/usr/bin/ls` → actual binary (executable file)
- `/usr/share/man/man1/ls.1.gz` → manual page

```bash
whereis python3
```

**Expected Output:**
```
python3: /usr/bin/python3 /usr/lib/python3 /usr/share/man/man1/python3.1.gz
```

### `which` vs `whereis` তুলনা:

| বিষয় | `which` | `whereis` |
|-------|---------|-----------|
| Binary location | ✅ | ✅ |
| Manual page | ❌ | ✅ |
| Source code | ❌ | ✅ |
| Output | একটা path | একাধিক path |


## Real-World Scenarios

### Scenario 1: Disk Full - কারণ খোঁজা
```bash
find / -size +500M -type f 2>/dev/null
```
> ৫০০MB-এর বেশি সব file বের করো। `2>/dev/null` মানে error message গুলো ignore করো।

### Scenario 2: Config file কে recently change করেছে?
```bash
find /etc -mtime -1 -type f
```
> গত ২৪ ঘন্টায় `/etc`-এ কোন file পরিবর্তন হয়েছে।

### Scenario 3: Dangerous Permission আছে কিনা চেক করো
```bash
find / -perm -4000 -type f 2>/dev/null
```
> SUID permission আছে এমন file খোঁজো (security check)।

### Scenario 4: পুরনো log file delete করো
```bash
find /var/log -name "*.log" -mtime +30 -exec rm {} \;
```
> ৩০ দিনের পুরনো সব log file delete করো।


## 📝 Quick Summary

- ✅ `find` → Real-time, powerful, অনেক filter - disk scan করে
- ✅ `locate` → Database থেকে দ্রুত খোঁজে, কিন্তু `updatedb` দরকার হয়
- ✅ `which` → Command কোথায় installed সেটা বলে
- ✅ `whereis` → Binary + manual + source সব location বলে
- ✅ `find -exec` → খোঁজার পরে সরাসরি কাজ করার সুবিধা দেয়
- ✅ `find` DevOps/Sys Admin-এ সবচেয়ে বেশি কাজে লাগে


## 🏋️ Practice Tasks

এখন নিজে নিজে চেষ্টা করুন:

**Task 1:**
`/etc` directory-তে সব `.conf` file খুঁজুন এবং তাদের list করুন।

**Task 2:**
আপনার home directory-তে একটা file তৈরি করুন, তারপর `find` দিয়ে নাম দিয়ে খুঁজুন। তারপর `locate` দিয়ে খোঁজার চেষ্টা করুন, আর পার্থক্য দেখুন।

**Task 3:**
`which` দিয়ে `bash`, `python3`, এবং `curl` কোথায় আছে বের করুন। তারপর `whereis` দিয়ে একই কাজ করুন এবং output তুলনা করুন।

---

## ⏭️ What's Next?

**Chapter 1 - Lesson 7: File Editors**
পরের lesson-এ আমরা শিখবো Linux-এর দুটো সবচেয়ে popular text editor **nano** (সহজ) এবং **vim** (powerful)। কীভাবে file open করতে হয়, edit করতে হয়, save করতে হয় এবং quit করতে হয়, সব শিখবো। DevOps engineer হিসেবে vim জানাটা অনেক জরুরি! 🚀
