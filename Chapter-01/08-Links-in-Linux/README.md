# Chapter 1 - Lesson 8: Links in Linux (Hard Links vs Soft/Symbolic Links)

**Chapter 1 | Lesson 8 of 10**

## 🎯 আজকে কী শিখবো?

আজকে আমরা Linux-এর একটা অসাধারণ concept শিখবো **Links!**

Links মানে হলো একটা file বা directory-কে **অন্য একটা নামে বা জায়গা থেকে access করার উপায়।**

Linux-এ দুই ধরনের link আছে:
1. **Hard Link**
2. **Soft Link (Symbolic Link / Symlink)**


মনে করেন আপনি একটা বই লিখেছেন। এখন:

- **Hard Link** → একই বইয়ের **দুটো আলাদা copy** যেগুলো **একই content share করে।** একটা নষ্ট হলেও অন্যটা ঠিকই থাকে।
- **Soft Link** → একটা বইয়ের **shortcut বা pointer।** মূল বই সরিয়ে ফেললে shortcut কাজ করবে না।


## আগে জানি inode কী?

Link বোঝার আগে **inode** সম্পর্কে একটু জানা দরকার।

Linux-এ প্রতিটা file তৈরি হলে সে একটা **inode number** পায়। inode হলো file-এর **identity card**। এখানে থাকে:

- File-এর size
- Permissions
- Owner
- কোথায় disk-এ data আছে তার address

**কিন্তু** filename inode-এ থাকে না! Filename শুধু directory-তে থাকে, যেটা ঐ inode-কে point করে।

```
Filename (directory তে) ──────→ inode ──────→ Actual Data (disk এ)
```


## Hard Link

### Hard Link কী?

Hard link মানে হলো **একই inode-কে দুটো আলাদা নাম দিয়ে point করা।**

```
file1.txt ──┐
            ├──→ inode 12345 ──→ Actual Data
file2.txt ──┘
```

`file1.txt` এবং `file2.txt` দুটোই **একই inode** point করছে, মানে **একই data।**

### Hard Link তৈরি করা যায় `ln` command দিয়ে

```bash
ln [original_file] [link_name]
```

### Example:

```bash
# প্রথমে একটা file তৈরি করুন
echo "I'll become a DevOps Engineer" > original.txt

# Hard link তৈরি করুন
ln original.txt hardlink.txt

# দুটো file দেখো
ls -li original.txt hardlink.txt
```

### Expected Output:

```
131072 -rw-r--r-- 2 user user 35 Mar 4 10:00 hardlink.txt
131072 -rw-r--r-- 2 user user 35 Mar 4 10:00 original.txt
```

এখানে লক্ষ্য করুন:
| Column | মানে কী |
|--------|---------|
| `131072` | **inode number** - দুটোর একই! |
| `2` | **Link count** - এই inode-কে ২টা নাম point করছে |
| `35` | File size |

### Hard Link-এর Magic পরীক্ষা:

```bash
# original file-এ কিছু লেখো
echo "This is a line" >> original.txt

# hardlink file কে cat করলে একই content দেখাবে!
cat hardlink.txt
```

```
I'll become a DevOps Engineer
This is a line
```

```bash
# এখন original file delete করুন
rm original.txt

# hardlink এখনো কাজ করবে!
cat hardlink.txt
```

```
I'll become a DevOps Engineer
This is a line
```

**Original ফাইল delete করলেও Hard Link কাজ করে** কারণ inode এখনো আছে, শুধু একটা নাম গেছে।

### Hard Link-এর Limitations:

| Limitation | কারণ |
|-----------|------|
| **Directory-তে hard link করা যায় না** | Filesystem loop তৈরি হওয়ার ভয় |
| **Different partition/filesystem-এ কাজ করে না** | inode শুধু একটা filesystem-এর মধ্যে unique |

## Soft Link (Symbolic Link / Symlink)

### Soft Link কী?

Soft link হলো একটা **shortcut file** যেটা original file-এর **path/নাম** store করে। Windows-এর `.lnk` shortcut-এর মতো।

```
symlink.txt ──→ (stores path: /home/user/original.txt) ──→ inode ──→ Data
```

Symlink-এর **নিজের আলাদা inode** আছে, সে শুধু original file-এর address মনে রাখে।

### Soft Link তৈরি করার command `ln -s`

```bash
ln -s [original_file_or_directory] [link_name]
```

এখানে `-s` মানে **symbolic।**

### Example:

```bash
# একটা file তৈরি করুন
echo "I am learning Linux" > original.txt

# Soft link তৈরি করুন
ln -s original.txt softlink.txt

# দেখা
ls -li original.txt softlink.txt
```

### Expected Output:

```
131073 -rw-r--r-- 1 user user 20 Mar 4 10:00 original.txt
131099 lrwxrwxrwx 1 user user 11 Mar 4 10:00 softlink.txt -> original.txt
```

লক্ষ্য করুন:
| বিষয় | মানে |
|------|------|
| **inode আলাদা** (131073 vs 131099) | Symlink-এর নিজস্ব inode আছে |
| `l` permission-এর শুরুতে | এটা **link** বোঝায় |
| `softlink.txt -> original.txt` | কোথায় point করছে দেখাচ্ছে |
| `lrwxrwxrwx` | Symlink সবসময় এই permission দেখায় (real permission original file-এর) |

### Soft Link-এর Behavior পরীক্ষা করা:

```bash
# softlink দিয়ে পড়া
cat softlink.txt
```
```
I am learning Linux
```

```bash
# এখন original delete করে ফেলেন
rm original.txt

# এখন softlink পড়ার চেষ্টা করুন
cat softlink.txt
```
```
cat: softlink.txt: No such file or directory
```

Original ফাইল ডিলিট হয়ে গেলে Soft Link broken হয়ে যায়, এটাকে বলে Dangling Symlink।

```bash
# Broken symlink দেখতে কেমন লাগে
ls -la softlink.txt
```
```
lrwxrwxrwx 1 user user 11 Mar 4 10:00 softlink.txt -> original.txt
```
Terminal-এ এটা **লাল রঙে** দেখাবে। broken symlink চেনার সহজ উপায়।

## Directory-তে Symlink (Hard Link পারে না, Symlink পারে!)

```bash
# একটা directory তৈরি করুন
mkdir /tmp/myproject

# Symlink তৈরি করুন
ln -s /tmp/myproject ~/project_shortcut

# এখন shortcut-এ ঢুকুন
cd ~/project_shortcut
pwd
```
```
/home/user/project_shortcut
```

## Hard Link vs Soft Link

| বিষয় | Hard Link | Soft Link |
|------|-----------|-----------|
| **inode** | একই inode share করে | আলাদা inode থাকে |
| **Original delete হলে** | কাজ করতে থাকে ✅ | Broken হয়ে যায় ❌ |
| **Directory-তে** | কাজ করে না ❌ | কাজ করে ✅ |
| **Different partition-এ** | কাজ করে না ❌ | কাজ করে ✅ |
| **Size দেখায়** | Original size | শুধু path-এর size |
| **Command** | `ln file link` | `ln -s file link` |
| **Permission** | Original-এর মতো | `lrwxrwxrwx` দেখায় |
| **DevOps use** | Log files, backups | Config files, version switching |

## DevOps-এ Real-World Use Cases

### Use Case 1: Software Version Switching (Symlink)

```bash
# দুটো Python version আছে
ls /usr/bin/python3.10
ls /usr/bin/python3.12

# Symlink দিয়ে default set করুন
ln -s /usr/bin/python3.12 /usr/local/bin/python

# এখন 'python' command মানেই python3.12
python --version
```
```
Python 3.12.0
```

নতুন version আসলে শুধু symlink update করলেই হবে, পুরো system ঠিক থাকবে।

### Use Case 2: Config File Management (Symlink)

```bash
# Nginx config আলাদা জায়গায় রাখুন
ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/myapp.conf

# Site disable করতে শুধু symlink delete করুন
rm /etc/nginx/sites-enabled/myapp.conf
# Original config নিরাপদ থাকলো!
```

### Use Case 3: Shared Log Files (Hard Link)

```bash
# একই log file দুই জায়গা থেকে access করুন
ln /var/log/app/error.log /home/devops/error.log
# Original delete হলেও hard link থেকে পড়া যাবে
```

## Important Commands Summary

```bash
# Hard link তৈরি
ln original.txt hardlink.txt

# Soft link তৈরি
ln -s original.txt softlink.txt

# Directory-তে soft link
ln -s /path/to/dir linkname

# Symlink কোথায় point করছে দেখুন
readlink softlink.txt

# Full path সহ দেখুন
readlink -f softlink.txt

# সব link সহ বিস্তারিত দেখুন
ls -la

# inode number দেখুন
ls -li

# Broken symlink খুঁজে বের করুন
find /path -xtype l

# Symlink delete করুন (/ ছাড়া)
rm softlink.txt
# অথবা
unlink softlink.txt
```

## 📝 Quick Summary

- **inode** হলো file-এর identity, data কোথায় আছে সে জানে
- **Hard Link** → একই inode share করে, original গেলেও link ফাইল ঠিক থাকে, directory/cross-partition এ কাজ করে না
- **Soft Link** → আলাদা inode, শুধু path মনে রাখে, original গেলে broken হয়, directory ও cross-partition এ কাজ করে
- **Dangling Symlink** → যে symlink-এর original নেই
- `ln` → hard link, `ln -s` → soft link
- `ls -li` → inode সহ details দেখায়
- `readlink` → symlink কোথায় point করে দেখায়

---

## 🏋️ Practice Tasks

**Task 1:**
```bash
# একটা file তৈরি করুন এবং তার hard link তৈরি করুন
# ls -li দিয়ে দুটোর inode একই কিনা verify করুন
# Original delete করে hard link কাজ করে কিনা দেখুন
```

**Task 2:**
```bash
# একটা directory তৈরি করুন /tmp/testdir
# সেই directory-তে একটা symlink তৈরি করুন home directory-তে
# symlink দিয়ে directory-তে ঢুকুন এবং pwd দিয়ে দেখুন
```

**Task 3:**
```bash
# একটা soft link তৈরি করুন
# readlink দিয়ে কোথায় point করছে দেখুন
# Original file delete করে "dangling symlink" তৈরি করুন
# ls -la দিয়ে কেমন দেখায় observe করুন
```

---

## ⏭️ What's Next?

**Chapter 1 - Lesson 9: Input/Output Redirection & Pipes**
`>`, `>>`, `<`, `|`, `tee` এই powerful tools দিয়ে commands-এর output কে control করতে শিখবো। DevOps-এ এটা প্রতিদিন ব্যবহার হয়! 🚀
