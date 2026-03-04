# Chapter 1 - Lesson 3: Navigating the File System

✅ **Chapter 1 | Lesson 3 of 10**
📍 `pwd` | `ls` | `cd` | `tree`


## 🎯 এই Lesson-এ আমরা কী শিখবো?

Linux-এ কাজ করতে হলে File System-এর ভেতরে **navigate** করতে জানতে হবে। মানে কোন folder-এ আছি, কী আছে সেখানে, অন্য folder-এ কীভাবে যাবো, এই সব।

এটা অনেকটা **Windows Explorer** বা **File Manager** ব্যবহার করার মতো, শুধু এখানে mouse নেই, সব **command দিয়ে** করতে হবে।


> কল্পনা করুন আপনি একটা **বড় অফিস বিল্ডিং**-এ আছেন।
> - বিল্ডিং-এর প্রতিটা **ফ্লোর** = Linux-এর এক একটা **directory**
> - আপনি এখন কোন ফ্লোরে আছেন সেটা জানতে = `pwd`
> - ঐ ফ্লোরে কী কী room আছে তা দেখার জন্য = `ls`
> - অন্য ফ্লোরে যেতে = `cd`
> - পুরো বিল্ডিং-এর map দেখতে = `tree`

## 🔵 Command 1: `pwd` - Print Working Directory

### 📌 এটা কী করে?
আপনি এখন **কোন directory-তে আছেন** সেটা দেখায়। "আমি এখন কোথায় আছি?" এই প্রশ্নের উত্তর।

### 📌 Syntax:
```bash
pwd
```
কোনো option লাগে না। শুধু `pwd` লিখে Enter চাপলেই হবে।

### 📌 Example:
```bash
$ pwd
/home/munir
```

এর মানে হলো আমি এখন `/home/munir` directory-তে আছো। অর্থাৎ আমার ইউজার directory।

### 📌 DevOps-এ কাজে লাগে কখন?
- Script চালানোর আগে confirm করতে হয় যে সঠিক জায়গায় আছি কিনা।
- কোনো ভুল directory থেকে command রান করে ফেললে যাতে সহজে বুঝতে পারি।
- CI/CD pipeline-এ current path verify করতে।

## 🔵 Command 2: `ls` - List Directory Contents

### 📌 এটা কী করে?
Current directory-তে কী কী **file ও folder** আছে তা দেখায়।

### 📌 Basic Syntax:
```bash
ls [option] [path]
```

### 📌 সবচেয়ে গুরুত্বপূর্ণ Options:

| Option | মানে | কাজ |
|--------|------|-----|
| `ls` | কোনো option নেই | Basic list দেখায় |
| `ls -l` | long format | Permission, size, date সহ দেখায় |
| `ls -a` | all | Hidden file গুলোও দেখায় (`.` দিয়ে শুরু) |
| `ls -la` | long + all | সব কিছু detail সহ |
| `ls -lh` | human readable | Size MB/KB তে দেখায় |
| `ls -lt` | time sorted | সবচেয়ে নতুন file আগে |
| `ls -R` | recursive | সব sub-folder সহ দেখায় |

### 📌 Examples:

**Basic list:**
```bash
$ ls
Desktop  Documents  Downloads  Music  Pictures
```

**Long format (`-l`):**
```bash
$ ls -l
total 32
drwxr-xr-x 2 devops devops 4096 Jun  1 10:00 Desktop
drwxr-xr-x 3 devops devops 4096 Jun  1 09:30 Documents
-rw-r--r-- 1 devops devops 1024 Jun  1 08:00 notes.txt
```

> এখানে প্রতিটা column মানে:
> `drwxr-xr-x` → Permission | `2` → Links | `devops` → Owner | `4096` → Size | `Jun 1` → Date | `Desktop` → Name

**Hidden files দেখতে (`-a`):**
```bash
$ ls -a
.  ..  .bashrc  .profile  Desktop  Documents
```
> `.bashrc` এবং `.profile` — এগুলো hidden file, শুধু `-a` দিলেই দেখা যায়।

**Size সহ সুন্দরভাবে (`-lh`):**
```bash
$ ls -lh
total 32K
drwxr-xr-x 2 devops devops 4.0K Jun  1 Desktop
-rw-r--r-- 1 devops devops 1.0K Jun  1 notes.txt
```

**অন্য directory-র contents দেখা (না গিয়েই):**
```bash
$ ls /etc
apt  bash.bashrc  crontab  hostname  hosts  passwd  ssh
```

### 📌 DevOps-এ কাজে লাগে:
- Server-এ কোন config file আছে check করতে
- Log directory-তে কী files জমেছে দেখতে
- Script deploy করার আগে directory verify করতে

## 🔵 Command 3: `cd` - Change Directory

### 📌 এটা কী করে?
এক directory থেকে আরেক directory-তে **যাওয়ার** কাজ করে। এটাই তোমার সবচেয়ে বেশি ব্যবহার করা command হবে।

### 📌 Syntax:
```bash
cd [path]
```

### 📌 সব গুরুত্বপূর্ণ `cd` ব্যবহার:

**১. নির্দিষ্ট directory-তে যাওয়া:**
```bash
$ cd /etc
$ pwd
/etc
```

**২. Home directory-তে ফিরে যাওয়া (৩টা উপায়):**
```bash
$ cd ~
$ cd
$ cd $HOME
```
> তিনটাই আপনাকে `/home/your_username` এ নিয়ে যাবে। যেমন আমার ক্ষেত্রে `/home/munir`

**৩. এক ধাপ উপরে যাওয়া (Parent directory):**
```bash
$ pwd
/home/devops/Documents
$ cd ..
$ pwd
/home/devops
```
> `..` মানে "এক ধাপ উপরে"

**৪. দুই ধাপ উপরে যাওয়া:**
```bash
$ cd ../..
```

**৫. আগে যেখানে ছিলে সেখানে ফিরে যাওয়া:**
```bash
$ cd -
```
> এটা অনেক কাজের! দুটো directory-র মধ্যে দ্রুত switch করতে পারবেন।

**৬. Nested directory-তে সরাসরি যাওয়া:**
```bash
$ cd /var/log/nginx
$ pwd
/var/log/nginx
```

### 📌 Absolute Path vs Relative Path বোঝো:

| ধরন | মানে | Example |
|-----|------|---------|
| **Absolute Path** | Root `/` থেকে শুরু করে পুরো path | `cd /home/devops/Documents` |
| **Relative Path** | আপনি এখন যেখানে আছেন সেখান থেকে | `cd Documents` (যদি `/home/devops`-এ থাকেন) |

> Absolute path = বাড়ির পুরো ঠিকানা (জেলা, উপজেলা, গ্রাম সহ)। Relative path = "পাশের বাড়ি" (আপনি কোথায় আছেন সেটা জেনেই বলা)।

### 📌 DevOps-এ কাজে লাগে:
- `/etc/nginx/` তে গিয়ে config edit করতে
- `/var/log/` এ গিয়ে log দেখতে
- Script-এর মধ্যে directory change করতে

## 🔵 Command 4: `tree` - Visual Directory Tree

### 📌 এটা কী করে?
Directory structure টাকে **গাছের মতো করে** সুন্দরভাবে দেখায়। পুরো folder hierarchy একসাথে বোঝা যায়।

### 📌 Installation (অনেক system-এ default থাকে না):
```bash
# Ubuntu/Debian:
sudo apt install tree

# CentOS/RHEL:
sudo yum install tree
```

### 📌 Syntax:
```bash
tree [option] [path]
```

### 📌 Examples:

**Basic tree:**
```bash
$ tree
.
├── Desktop
├── Documents
│   ├── project
│   │   └── app.py
│   └── notes.txt
├── Downloads
└── Music

4 directories, 2 files
```

**শুধু Directory দেখাও (file বাদ):**
```bash
$ tree -d
.
├── Desktop
├── Documents
│   └── project
├── Downloads
└── Music
```

**নির্দিষ্ট level পর্যন্ত দেখাও:**
```bash
$ tree -L 2
```
> `-L 2` মানে ২ level গভীর পর্যন্ত দেখাও।

**File size সহ দেখাও:**
```bash
$ tree -h
```

**নির্দিষ্ট path-এর tree:**
```bash
$ tree /etc/ssh
/etc/ssh
├── moduli
├── ssh_config
├── sshd_config
└── ssh_host_rsa_key
```

### 📌 DevOps-এ কাজে লাগে:
- Project structure document করতে
- Server-এর config folder structure বুঝতে
- নতুন server setup-এ directory layout verify করতে

## 🧠 সব Command একসাথে Practice করুন

একটা realistic DevOps scenario দেখুন:

```bash
# ১. আমি এখন কোথায়?
$ pwd
/home/munir

# ২. এখানে কী আছে?
$ ls -la
total 48
drwxr-xr-x  6 munir munir 4096 Jun  1 .
drwxr-xr-x  3 root   root 4096 Jun  1 ..
-rw-r--r--  1 munir munir  220 Jun  1 .bash_logout
-rw-r--r--  1 munir munir 3526 Jun  1 .bashrc
drwxr-xr-x  2 munir munir 4096 Jun  1 projects

# ৩. projects folder-এ যাই আর না থাকলে `mkdir projects` রান করে তৈরি করুন।
$ cd projects

# ৪. এখানে কী আছে?
$ ls
webapp  database  scripts

# ৫. webapp-এ যাই
$ cd webapp

# ৬. আবার projects-এ ফিরি
$ cd ..

# ৭. আগের জায়গায় (webapp) ফিরে যাই instantly
$ cd -
/home/munir/projects/webapp

# ৮. Home-এ চলে যাই
$ cd ~

# ৯. পুরো structure দেখি
$ tree -L 2
.
└── projects
    ├── webapp
    ├── database
    └── scripts
```

## 📝 Quick Summary

- ✅ `pwd` → আপনি এখন **কোথায়** আছেন জানার জন্য।
- ✅ `ls` → directory-তে **কী আছে** দেখায়
- ✅ `ls -la` → **সব কিছু detail সহ** দেখায় (সবচেয়ে বেশি ব্যবহার হয়)
- ✅ `cd /path` → নির্দিষ্ট **directory-তে যাওয়া**
- ✅ `cd ..` → **এক ধাপ উপরে** যাওয়া
- ✅ `cd ~` → **Home directory**-তে যাওয়া
- ✅ `cd -` → **আগের জায়গায়** ফিরে আসা
- ✅ `tree` → **Visual tree** আকারে structure দেখা


## 🏋️ Practice Tasks

এগুলো নিজে নিজে করে দেখুন:

**Task 1:**

/etc directory-তে যান, সেখানে ls -lh রান করুন, তারপর home-এ ফিরে আসুন এবং pwd দিয়ে confirm করুন।


**Task 2:**

`cd` ব্যবহার করে /var/log এবং /etc/ssh এর মধ্যে বারবার switch করুন। কতটা fast কাজ করে দেখুন!

**Task 3:**

`tree -L 2 /` রান করুন এবং root filesystem-এর প্রথম দুটো level দেখুন। (Lesson 2-এ যা শিখেছেন মিলিয়ে দেখুন!)


## ⏭️ What's Next?

**Chapter 1 - Lesson 4: Working with Files & Directories**
> `touch`, `mkdir`, `cp`, `mv`, `rm`, `rmdir` file ও folder **তৈরি, কপি, move এবং delete** করা শিখবো। DevOps-এর সবচেয়ে daily-use commands! 🚀

আমরা খুব ভালোভাবেই এগিয়ে যাচ্ছি! Navigation হলো Linux-এর **backbone** এটা ভালো করে আয়ত্ত করতে পারলে বাকি সব অনেক সহজ হয়ে যাবে।
