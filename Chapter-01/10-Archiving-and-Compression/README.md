# Chapter 1 - Lesson 10: Archiving & Compression

### Chapter 1 | Lesson 10 of 10 🎉 (Chapter 1 শেষ!)


## 🎯 এই Lesson-এ কী শিখবো?

- Archiving কী এবং Compression কী, পার্থক্য বোঝা
- `tar` সবচেয়ে গুরুত্বপূর্ণ tool
- `gzip` / `gunzip` compress ও decompress
- `zip` / `unzip` Windows-friendly format
- Real DevOps use cases


## চলুন প্রথমে আমরা Concept বোঝার চেস্টা করি।

### Archiving vs Compression - দুটো আলাদা জিনিস!

একটু রিয়েল লাইফ analogy দিয়ে বুঝি:

> **Archiving** = অনেকগুলো জিনিস একটা বাক্সে ভরা।
> কিন্তু বাক্সের সাইজ একই থাকে।

> **Compression** = সেই বাক্সটাকে vacuum করে ছোট করে ফেলা।

Linux-এ:
- **`tar`** → Archive করে (অনেকগুলো ফাইলকে আর্কাইভ করে একটা ফাইল করে ফেলে)
- **`gzip`** → Compress করে (file ছোট করে)
- **`tar + gzip`** একসাথে → Archive করে **এবং** Compress করে।

## Tool 1: `tar`

### `tar` কী?

**tar = Tape Archive**
পুরনো দিনে data tape-এ backup রাখা হতো। সেখান থেকে নামটা এসেছে।
এখন এটা সবচেয়ে common archiving tool।

### Syntax:

```bash
tar [options] [archive_name.tar] [files/directories]
```

### গুরুত্বপূর্ণ Options (flags):

| Flag | মানে | সহজ কথায় |
|------|------|-----------|
| `-c` | create | নতুন archive তৈরি করো |
| `-x` | extract | archive খোলো |
| `-v` | verbose | কী হচ্ছে দেখাও |
| `-f` | file | archive file-এর নাম বলো |
| `-z` | gzip | gzip দিয়ে compress করো |
| `-j` | bzip2 | bzip2 দিয়ে compress করো |
| `-t` | list | archive-এর ভেতরে কী আছে দেখো |
| `-C` | directory | কোথায় extract করবে বলো |


### Example 1: শুধু Archive তৈরি করুন (compress ছাড়া)

```bash
tar -cvf mybackup.tar /home/user/projects
```

**ব্যাখ্যা:**
- `-c` → নতুন archive তৈরি করো
- `-v` → verbose (কোন file যাচ্ছে দেখাবে)
- `-f` → এরপরে archive-এর নাম দাও
- `mybackup.tar` → archive-এর নাম
- `/home/user/projects` → কোন folder archive করবে

**Expected Output:**
```
projects/
projects/app.py
projects/config.yml
projects/logs/
projects/logs/error.log
```


### Example 2: Archive + Compress একসাথে (`.tar.gz`)

```bash
tar -czvf mybackup.tar.gz /home/user/projects
```

**ব্যাখ্যা:**
- `-z` যোগ হয়েছে → gzip দিয়ে compress করবে
- Extension হবে `.tar.gz` (অথবা `.tgz`)

এটাই **সবচেয়ে বেশি ব্যবহৃত** format DevOps-এ!

### Example 3: Archive খোলো (Extract)

```bash
tar -xzvf mybackup.tar.gz
```

**ব্যাখ্যা:**
- `-x` → extract করো
- `-z` → gzip ছিল তাই decompress করো
- `-v` → কী extract হচ্ছে দেখাও
- `-f` → file-এর নাম


### Example 4: নির্দিষ্ট folder-এ Extract করুন

```bash
tar -xzvf mybackup.tar.gz -C /tmp/restore/
```

**ব্যাখ্যা:**
- `-C /tmp/restore/` → এই directory-তে extract করো
- আগে সেই directory থাকতে হবে: `mkdir -p /tmp/restore`

### Example 5: Archive-এর ভেতরে কী আছে দেখার জন্য (Extract না করে)

```bash
tar -tzvf mybackup.tar.gz
```

**ব্যাখ্যা:**
- `-t` → list করো (extract করবে না!)
- DevOps-এ কোনো backup দেখার আগে এটা করা ভালো practice

**Expected Output:**
```
-rw-r--r-- user/user   1024 2026-03-06 app.py
-rw-r--r-- user/user    512 2026-03-06 config.yml
```

### Example 6: একটা নির্দিষ্ট file extract করুন archive থেকে

```bash
tar -xzvf mybackup.tar.gz projects/config.yml
```

শুধু `config.yml` বের হবে, পুরো archive না।

## Tool 2: `gzip` এবং `gunzip`

### `gzip` কী?

`gzip` শুধু একটা file compress করে। Folder compress করতে পারে না। তাই সাধারণত `tar` এর সাথে use করা হয়।

### Syntax:

```bash
gzip [file]
gunzip [file.gz]
```

### Example 1: একটা file compress করো

```bash
gzip largefile.log
```

**Result:** `largefile.log` চলে যাবে, তার জায়গায় `largefile.log.gz` তৈরি হবে।

### Example 2: Compress করো কিন্তু original রাখো

```bash
gzip -k largefile.log
```

**`-k`** = keep (original file রেখে দাও)

### Example 3: Decompress করো

```bash
gunzip largefile.log.gz
```

অথবা:

```bash
gzip -d largefile.log.gz
```

### Example 4: Compression level দাও

Compression Level মানে হলো, কতটা চাপ দিয়ে file ছোট করবেন।

ধরুন একটা স্পঞ্জ চাপ দিচ্ছেন:

- **Level 1** = হালকা চাপ → file বেশি ছোট হবে না, কিন্তু কাজ হবে **দ্রুত**
- **Level 9** = সর্বোচ্চ চাপ → file অনেক ছোট হবে, কিন্তু সময় লাগবে **বেশি**


```bash
gzip -9 largefile.log   # সর্বোচ্চ compression (slow কিন্তু ছোট)
gzip -1 largefile.log   # সর্বনিম্ন compression (fast কিন্তু বড়)
```

Level 1-9 পর্যন্ত। Default হলো 6।

**DevOps-এ কখন কোনটা?**
- Real-time বা বড় log file → `-1` (speed দরকার)
- Long-term backup/archive → `-9` (size ছোট রাখতে চাই)

### Example 5: Compressed file দেখা (extract না করে)

```bash
zcat file.gz        # cat এর মতো
zless file.gz       # less এর মতো
zgrep "error" file.gz   # grep এর মতো
```

DevOps-এ log file compressed থাকলে এটা অনেক কাজে লাগে!

## Tool 3: `bzip2` এবং `bunzip2`

`gzip` এর মতোই কিন্তু আরো বেশি compress করে (একটু slow)।

```bash
bzip2 file.txt          # compress → file.txt.bz2
bunzip2 file.txt.bz2    # decompress
```

`tar` এর সাথে: `-j` flag ব্যবহার করুন

```bash
tar -cjvf backup.tar.bz2 /home/user/projects
tar -xjvf backup.tar.bz2
```

## Tool 4: `zip` এবং `unzip`

### `zip` কী?

Windows-এর সাথে compatible format। যখন কাউকে file পাঠাবো যে Windows ব্যবহার করে, তখন `zip` ব্যবহার করবো।

### Example 1: Zip তৈরি করা

```bash
zip myarchive.zip file1.txt file2.txt
```

### Example 2: পুরো Directory zip করা

```bash
zip -r myarchive.zip /home/user/projects
```

**`-r`** = recursive (folder-এর ভেতরের সব কিছু)

### Example 3: Unzip করো

```bash
unzip myarchive.zip
```

### Example 4: নির্দিষ্ট directory-তে unzip করো

```bash
unzip myarchive.zip -d /tmp/extracted/
```

### Example 5: Zip-এর ভেতরে কী আছে দেখো

```bash
unzip -l myarchive.zip
```

## সব Format-এর তুলনা

| Format | Command | Extension | Compression | Speed | Use Case |
|--------|---------|-----------|-------------|-------|----------|
| tar only | `tar -cvf` | `.tar` | নেই | Fast | শুধু একত্র করা |
| tar + gzip | `tar -czvf` | `.tar.gz` | ভালো | মাঝারি | সবচেয়ে common |
| tar + bzip2 | `tar -cjvf` | `.tar.bz2` | বেশি | Slow | ছোট size দরকার |
| gzip only | `gzip` | `.gz` | ভালো | মাঝারি | একক file |
| zip | `zip -r` | `.zip` | ভালো | মাঝারি | Windows-এর সাথে share |


## Real DevOps Use Cases

### 1. Log Backup করা (প্রতিদিনের কাজ)

```bash
tar -czvf /backup/logs_$(date +%F).tar.gz /var/log/
```

**`$(date +%F)`** → আজকের date automatically বসবে, যেমন `logs_2026-03-06.tar.gz`

### 2. Application Deploy করার সময়

```bash
# Application package করুন
tar -czvf app_v2.1.tar.gz /opt/myapp/

# Server-এ পাঠান
scp app_v2.1.tar.gz user@server:/tmp/

# Server-এ extract করুন
tar -xzvf /tmp/app_v2.1.tar.gz -C /opt/
```

### 3. Database Backup

```bash
# MySQL dump করে compress করুন
mysqldump mydb | gzip > mydb_backup_$(date +%F).sql.gz

# Restore করতে
gunzip < mydb_backup_2026-03-06.sql.gz | mysql mydb
```

### 4. Compressed Log থেকে Error খোঁজা

```bash
zgrep "ERROR" /var/log/app_2026-03-06.log.gz
```

## গুরুত্বপূর্ণ Tips ও Tricks

### Tip 1: Progress দেখুন বড় file-এ

```bash
tar -czvf backup.tar.gz /data/ | pv
```

`pv` install থাকলে progress bar দেখাবে।

### Tip 2: Archive করার সময় কিছু বাদ দিয়ে করতে পারেন

```bash
tar -czvf backup.tar.gz /home/user/ --exclude="*.log" --exclude=".cache"
```

### Tip 3: File-এর size compare করা

```bash
ls -lh original_folder/
ls -lh backup.tar.gz
```

কতটা compress হলো দেখতে পারবেন।

## 📝 Quick Summary

- ✅ **Archiving** = অনেক file একত্র করে একটা ফাইল বানানো (size কমে না)
- ✅ **Compression** = file-এর size কমানো
- ✅ **`tar -czvf`** = archive + compress (সবচেয়ে বেশি ব্যবহৃত)
- ✅ **`tar -xzvf`** = extract করো
- ✅ **`tar -tzvf`** = ভেতরে কী আছে দেখো
- ✅ **`gzip`** = single file compress
- ✅ **`gunzip`** = decompress
- ✅ **`zip -r`** = Windows-friendly archive
- ✅ **`zcat`, `zgrep`** = compressed file সরাসরি পড়া

## 🏋️ Practice Tasks

**Task 1:**
`/tmp/` folder-এর ভেতরে `practice` নামে একটা directory তৈরি করুন। তার ভেতরে ৩টা file তৈরি করুন (`file1.txt`, `file2.txt`, `file3.txt`)। এরপর পুরো `practice` folder টাকে `practice_backup.tar.gz` নামে archive করুন।

**Task 2:**
`practice_backup.tar.gz` extract না করে দেখুন এর ভেতরে কী কী file আছে।

**Task 3:**
`practice_backup.tar.gz` কে `/tmp/restored/` directory-তে extract করুন।


## 🎉 Chapter 1 সম্পূর্ণ!

আমরা Chapter 1 এর সব কটি Lesson (মোট 10টা) শেষ করেছি! এটা একটা বড় অর্জন।

### Chapter 1-এ আমরা যা শিখেছি:
- Linux কী, কেন DevOps-এ use হয়
- File System Hierarchy
- Navigation commands
- File ও Directory management
- File contents দেখা
- File খোঁজা
- Editors (nano, vim)
- Hard ও Soft links
- Input/Output Redirection ও Pipes
- Archiving ও Compression


## ⏭️ What's Next

Linux-এ Security-র ভিত্তি হলো Permissions।
কোন user কোন file পড়তে, লিখতে বা run করতে পারবে, সেটা কীভাবে কাজ করে সেটা শিখবো।
