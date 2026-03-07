# Chapter 2 - Lesson 2: Changing Permissions (chmod, chown, chgrp)

**Chapter 2 | Lesson 2 of 10**

## 🎯 আজকের Lesson-এ আমরা কী শিখব?

আগের lesson-এ আমরা শিখেছিলাম যে Linux-এ permissions কীভাবে কাজ করে `rwx`, numeric mode, symbolic mode। আজকে আমরা শিখব সেই permissions কীভাবে পরিবর্তন করতে হয়।

তিনটা powerful command আজকে শিখব:
- **`chmod`** → file/directory-এর **permission** পরিবর্তন করে
- **`chown`** → file/directory-এর **owner** পরিবর্তন করে
- **`chgrp`** → file/directory-এর **group** পরিবর্তন করে

---

## 1. chmod - Change Mode

### chmod কী করে?

`chmod` মানে **Change Mode**। এটা দিয়ে আপনি যেকোনো file বা directory-এর read, write, execute permission যোগ করতে বা সরাতে পারবেন।

> **Analogy:** কল্পনা করুন আপনার বাড়ির দরজায় তালা আছে। `chmod` হলো সেই তালার চাবি বদলানোর system কাকে ঢুকতে দিবেন, কাকে দিবেন না, সেটা আপনি ঠিক করে দিতে পারেন।

---

**chmod-এর দুটো পদ্ধতি**

## পদ্ধতি ১: Symbolic Mode (অক্ষর দিয়ে)

```bash
chmod [who][operator][permission] filename
```

| Part | মানে | উদাহরণ |
|------|------|---------|
| `who` | কার জন্য? | `u` = user/owner, `g` = group, `o` = others, `a` = all |
| `operator` | কী করবে? | `+` = যোগ করুন, `-` = সরাও, `=` = সেট করুন |
| `permission` | কোন permission? | `r` = read, `w` = write, `x` = execute |

---

#### Symbolic Mode উদাহরণ:

**উদাহরণ ১: Owner-কে execute permission**
```bash
chmod u+x script.sh
```
```
Before: -rw-r--r--
After:  -rwxr--r--
```
> Owner (u) এর কাছে `+x` মানে execute permission যোগ হলো।

---

**উদাহরণ ২: Group থেকে write permission সরিয়ে দিন**
```bash
chmod g-w file.txt
```
```
Before: -rw-rw-r--
After:  -rw-r--r--
```

---

**উদাহরণ ৩: Others-কে কোনো permission দিবেন না**
```bash
chmod o= file.txt
```
```
Before: -rw-r--r--
After:  -rw-r-----
```
> `o=` মানে others-এর সব permission মুছে দিন।

---

**উদাহরণ ৪: সবাইকে (all) read permission দিন**
```bash
chmod a+r file.txt
```

---

## পদ্ধতি ২: Numeric Mode (সংখ্যা দিয়ে)

আগের lesson-এ শিখেছিলাম:
| সংখ্যা | মানে |
|--------|------|
| `4` | read (r) |
| `2` | write (w) |
| `1` | execute (x) |
| `0` | কোনো permission নেই |

তিনটা সংখ্যা = **[owner][group][others]**

```bash
chmod 755 script.sh
```

এখানে:
- `7` = 4+2+1 = rwx (owner সব পাবে)
- `5` = 4+0+1 = r-x (group শুধু read+execute পাবে)
- `5` = 4+0+1 = r-x (others শুধু read+execute পাবে)

---

#### Numeric Mode উদাহরণ:

**উদাহরণ ১: Script file-কে executable করুন**
```bash
chmod 755 deploy.sh
ls -l deploy.sh
```
```
-rwxr-xr-x 1 devops devops 0 Mar 4 10:00 deploy.sh
```

---

**উদাহরণ ২: Config file - শুধু owner পড়তে ও লিখতে পারবে**
```bash
chmod 600 config.env
ls -l config.env
```
```
-rw------- 1 devops devops 0 Mar 4 10:00 config.env
```
> **DevOps-এ এটা খুব গুরুত্বপূর্ণ!** SSH private key, `.env` file, secrets এইভাবে protect করা হয়।

---

**উদাহরণ ৩: সবার জন্য read-only**
```bash
chmod 444 readme.txt
ls -l readme.txt
```
```
-r--r--r-- 1 devops devops 0 Mar 4 10:00 readme.txt
```

---

### chmod -R (Recursive - সব ভেতরের file-এও apply করুন)

```bash
chmod -R 755 /var/www/html/
```
> এটা `/var/www/html/` folder এবং তার ভেতরের সব file ও folder-এ 755 permission দিবেন।

⚠️ **সতর্কতা:** `-R` ব্যবহারে সাবধান। ভুল হলে সব file-এর permission একসাথে নষ্ট হয়ে যেতে পারে।

---

### DevOps-এ chmod-এর Common Use Cases:

| Scenario | Command |
|----------|---------|
| Shell script চালানোর আগে | `chmod +x script.sh` |
| SSH private key protect করা | `chmod 600 ~/.ssh/id_rsa` |
| Web server files | `chmod 644 index.html` |
| Web server directories | `chmod 755 /var/www/` |
| Secret/env file | `chmod 600 .env` |

---

## 2. chown - Change Owner

**chown কী করে?**

`chown` মানে Change Owner। এটা দিয়ে আপনি file বা directory-এর owner এবং group পরিবর্তন করতে পারবেন।

> **Analogy:** একটা বাড়ির মালিকানা এক ব্যক্তি থেকে আরেক ব্যক্তির কাছে হস্তান্তর করার মতো।

---

### Syntax:

```bash
chown [new_owner] filename
chown [new_owner]:[new_group] filename
```

---

#### chown উদাহরণ:

**উদাহরণ ১: File-এর owner পরিবর্তন করুন**
```bash
chown alice file.txt
ls -l file.txt
```
```
-rw-r--r-- 1 alice devops 0 Mar 4 10:00 file.txt
```
> এখন file.txt-এর owner হলো `alice`।

---

**উদাহরণ ২: Owner এবং Group একসাথে পরিবর্তন করুন**
```bash
chown alice:developers file.txt
ls -l file.txt
```
```
-rw-r--r-- 1 alice developers 0 Mar 4 10:00 file.txt
```

---

**উদাহরণ ৩: Recursive - পুরো directory-এর owner পরিবর্তন করুন**
```bash
chown -R www-data:www-data /var/www/html/
```
> Web server (nginx/apache) চালানোর জন্য এই command DevOps-এ প্রতিদিন ব্যবহার হয়।

---

**উদাহরণ ৪: শুধু Group পরিবর্তন করুন (owner same রাখুন)**
```bash
chown :developers project/
ls -ld project/
```
```
drwxr-xr-x 2 alice developers 4096 Mar 4 10:00 project/
```
> `:developers` আগে colon দিলে শুধু group change হয়, owner same থাকে।

---

### DevOps-এ chown-এর Common Use Cases:

| Scenario | Command |
|----------|---------|
| Web files nginx/apache-এর জন্য | `chown -R www-data:www-data /var/www/` |
| App deploy করার পর | `chown -R appuser:appuser /opt/myapp/` |
| Log file ownership fix | `chown syslog:adm /var/log/app.log` |
| Docker volume ownership | `chown -R 1000:1000 /data/` |

---

## 3. chgrp - Change Group

**chgrp কী করে?**

`chgrp` মানে Change Group। এটা শুধু group পরিবর্তন করে, owner পরিবর্তন করে না।

> **মনে রাখুন:** `chown :groupname file` দিয়েও group change করা যায়। `chgrp` হলো সেটার একটা dedicated command।

---

### Syntax:

```bash
chgrp [new_group] filename
```

---

#### chgrp উদাহরণ:

**উদাহরণ ১: File-এর group পরিবর্তন করুন**
```bash
chgrp developers project.txt
ls -l project.txt
```
```
-rw-r--r-- 1 alice developers 0 Mar 4 10:00 project.txt
```

---

**উদাহরণ ২: Directory-এর group recursive পরিবর্তন করুন**
```bash
chgrp -R devteam /opt/project/
```

---

## chmod vs chown vs chgrp — পার্থক্য একনজরে

| Command | কী পরিবর্তন করে? | উদাহরণ |
|---------|----------------|---------|
| `chmod` | Permission (rwx) | `chmod 755 file.sh` |
| `chown` | Owner (এবং optionally Group) | `chown alice:dev file.txt` |
| `chgrp` | শুধু Group | `chgrp developers file.txt` |

---

## Real-World DevOps Scenario

**Scenario: নতুন Web App Deploy করলেন**

```bash
# ১. App files upload হলো /opt/myapp-এ
ls -l /opt/myapp/
# -rw-r--r-- 1 root root 1024 Mar 4 app.py

# ২. App-এর জন্য dedicated user আছে: appuser
# Owner পরিবর্তন করুন
chown -R appuser:appuser /opt/myapp/

# ৩. Script গুলো executable করুন
chmod +x /opt/myapp/start.sh

# ৪. Config file secure করুন (শুধু owner পড়তে পারবে)
chmod 600 /opt/myapp/config.env

# ৫. Static files web server পড়তে পারবে
chmod 644 /opt/myapp/static/*
```

---

## কিছু Shortcuts যা জানা দরকার

```bash
# Permission দেখার সহজ উপায়
stat filename

# Example output:
# File: filename
# Access: (0644/-rw-r--r--)  Uid: (1000/alice)   Gid: (1000/alice)
```

```bash
# Numeric permission দেখতে
stat -c "%a %n" filename
# Output: 644 filename
```

---

## 📝 Lesson Summary

- **`chmod`** → file-এর **permission** পরিবর্তন করে (symbolic বা numeric mode-এ)
- **`chown`** → file-এর **owner** (এবং group) পরিবর্তন করে
- **`chgrp`** → শুধু file-এর **group** পরিবর্তন করে
- **`-R` flag** → recursive - ভেতরের সব file-এও apply হয়
- **`chmod 600`** → secrets/keys এর জন্য সবচেয়ে common secure permission
- **`chmod 755`** → scripts ও directories-এর জন্য সবচেয়ে common permission
- **`chown user:group`** দিয়ে একসাথে owner ও group দুটোই change করা যায়

---

## 🏋️ Practice Tasks

এই তিনটা কাজ নিজে নিজে করে দেখুন:

**Task 1:**
```bash
# একটা file তৈরি করুন এবং permission পরিবর্তন করুন
touch myscript.sh
ls -l myscript.sh          # আগের permission দেখো
chmod 755 myscript.sh
ls -l myscript.sh          # পরিবর্তন দেখো
```

**Task 2:**
```bash
# একটা directory তৈরি করুন, owner ও group দেখো
mkdir myproject
ls -ld myproject
# এখন chown দিয়ে owner পরিবর্তন করার চেষ্টা করুন
# (আপনার system-এ যে user আছে সেটা দিয়ে)
sudo chown root:root myproject
ls -ld myproject
sudo chown $USER:$USER myproject   # আবার নিজের করে নাও
```

**Task 3:**
```bash
# একটা "secret" file তৈরি করুন এবং secure করুন
echo "DB_PASSWORD=secret123" > .env
chmod 600 .env
ls -l .env
# দেখো শুধু owner পড়তে পারছে
```

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 3: Special Permissions (SUID, SGID, Sticky Bit)**

পরের lesson-এ আমরা শিখবো Linux-এর কিছু **বিশেষ এবং শক্তিশালী permissions** যেগুলো না জানলে security বুঝতে পারবেন না। `passwd` command কেন root ছাড়াও কাজ করে? `/tmp` folder থেকে অন্যের file কেন delete করা যায় না? এই রহস্যগুলোর উত্তর পাবো পরের lesson-এ! 🚀
