# Chapter 1 - Lesson 5: Viewing File Contents

✅ **Chapter 1 | Lesson 5 of 10**
📂 **Topic: cat, less, more, head, tail, tac**


## 🎯 এই Lesson-এ কী শিখবো?

Linux-এ file-এর ভেতরের content দেখার অনেকগুলো উপায় আছে। প্রতিটা command আলাদা situation-এ কাজে লাগে। আজকে আমরা শিখবো:

| Command | কাজ |
|---------|-----|
| `cat` | পুরো file একসাথে দেখা |
| `less` | একটু একটু করে scroll করে দেখা |
| `more` | `less`-এর পুরনো version |
| `head` | file-এর শুরু থেকে কয়েকটা line দেখা |
| `tail` | file-এর শেষ থেকে কয়েকটা line দেখা |
| `tac` | file উল্টো করে দেখা (cat-এর উল্টো!) |


ধরুন আপনার কাছে একটা ৫০০ পৃষ্ঠার বই আছে।

- **`cat`** = বইটা এক নিঃশ্বাসে পুরোটা পড়ে ফেলে
- **`less`** = বই হাতে নিয়ে page-by-page উল্টানো
- **`head`** = বইয়ের শুধু প্রথম কয়েকটা page পড়ে
- **`tail`** = বইয়ের শুধু শেষের কয়েকটা page পড়ে
- **`tac`** = বইটা শেষ page থেকে পড়া শুরু করে

DevOps-এ আপনি সবচেয়ে বেশি এই commands গুলো use করবেন বিশেষ করে **log file** দেখার জন্য।


## 🔧 Practice-এর জন্য একটা File তৈরি করি

প্রথমে আমরা একটা sample file বানিয়ে নিই যেটা দিয়ে সব commands practice করবো।

এই command টি terminal-এ run করি

```bash
cat > myfile.txt << EOF
Line 1: This is the first line
Line 2: Hello from Linux
Line 3: DevOps is awesome
Line 4: Learning is fun
Line 5: cat command rocks
Line 6: tail and head are useful
Line 7: log files are important
Line 8: keep going!
Line 9: almost there
Line 10: this is the last line
EOF
```

এটা run করলে `myfile.txt` নামে একটা file তৈরি হবে যেখানে ১০টা line থাকবে।



## 1️⃣ `cat` - File-এর পুরো Content দেখা যায়

### 📌 এটা কী করে?
`cat` মানে **concatenate**। এটা file-এর পুরো content একসাথে terminal-এ দেখায়।

### 📌 Syntax:
```bash
cat [options] filename
```

### 📌 Basic Example:
```bash
cat myfile.txt
```

**Output:**
```
Line 1: This is the first line
Line 2: Hello from Linux
Line 3: DevOps is awesome
Line 4: Learning is fun
Line 5: cat command rocks
Line 6: tail and head are useful
Line 7: log files are important
Line 8: keep going!
Line 9: almost there
Line 10: this is the last line
```


### 📌 `cat`-এর কাজের Options:

#### `-n` → প্রতিটা line-এ number দেখাও
```bash
cat -n myfile.txt
```
**Output:**
```
     1  Line 1: This is the first line
     2  Line 2: Hello from Linux
     3  Line 3: DevOps is awesome
     ...
    10  Line 10: this is the last line
```

#### `-A` → Hidden characters দেখাও (space, tab, newline)
```bash
cat -A myfile.txt
```
**Output:**
```
Line 1: This is the first line$
Line 2: Hello from Linux$
```
> 💡 Line-এর শেষে `$` মানে সেখানে newline আছে। DevOps-এ config file debug করতে এটা কাজে লাগে।

#### Multiple files একসাথে দেখাও:
```bash
cat file1.txt file2.txt
```

#### File তৈরি করতে `cat` use করুন:
```bash
cat > newfile.txt
# এখন type করুন,
# type করা শেষ হলে Ctrl+D চেপে বের হয়ে আসুন।
```

#### দুটো file জোড়া লাগানো (merge করা):
```bash
cat file1.txt file2.txt > combined.txt
```

### 🏭 DevOps-এ `cat`-এর Use Case:
- Config file দেখা: `cat /etc/hostname`
- Log file দেখা: `cat /var/log/syslog`
- Script দেখা: `cat deploy.sh`
- Environment variables দেখা: `cat .env`


## 2️⃣ `less` - Scroll করে File দেখাও

### 📌 এটা কী করে?
`less` বড় file দেখার জন্য সবচেয়ে ভালো command। এটা একটা **interactive viewer** খোলে যেখানে আপনি উপর থেকে নিচে scroll করতে পারেন।

> 💡 **মনে রাখি:** `less` is **more** than `more`! (এটা একটা Linux joke 😄)

### 📌 Syntax:
```bash
less filename
```

### 📌 Example:
```bash
less myfile.txt
```

এটা run করলে একটা interactive screen খুলবে।

### 📌 `less`-এর ভেতরে কী করবো?

| Key | কাজ |
|-----|-----|
| `Space` বা `f` | পরের page-এ যাওয়া |
| `b` | আগের page-এ যাওয়া |
| `↑` `↓` arrow keys | এক line উপর/নিচ |
| `g` | File-এর একদম শুরুতে যাওয়া |
| `G` (capital) | File-এর একদম শেষে যাওয়া |
| `/word` | নির্দিষ্ট word search করো |
| `n` | পরের search result-এ যাওয়া |
| `N` (capital) | আগের search result-এ যাওয়া |
| `q` | বেরিয়ে যাওয়া |

### 📌 Useful Options:

#### `-N` → Line number দেখাও
```bash
less -N myfile.txt
```

#### `+F` → Real-time file follow করা (log দেখার সময় দারুণ!)
```bash
less +F /var/log/syslog
```
> 💡 এটা `tail -f`-এর মতো। নতুন log আসলে সাথে সাথে দেখাবে। বের হতে `Ctrl+C` তারপর `q`।

### 🏭 DevOps-এ `less`-এর Use Case:
- বড় log file দেখা
- Config file scroll করে পড়া
- Script review করা
- `man` page পড়া (actually `man` নিজেই `less` use করে!)


## 3️⃣ `more` - `less`-এর পুরনো ভাই

### 📌 এটা কী করে?
`more` হলো `less`-এর পুরনো version। এটাও file দেখায় কিন্তু **শুধু নিচের দিকে** যেতে পারে, উপরে ফিরে আসা যায় না।

### 📌 Syntax:
```bash
more filename
```

### 📌 Example:
```bash
more myfile.txt
```

| Key | কাজ |
|-----|-----|
| `Space` | পরের page |
| `Enter` | এক line নিচে |
| `q` | বেরিয়ে যাওয়া |

> ⚠️ **Practical Tip:** Modern Linux-এ সবসময় `less` use করবো। `more` এখন outdated। তবে কিছু minimal system বা script-এ `more` দেখতে পাবে, তাই জানা দরকার।


## 4️⃣ `head` - File-এর শুরু দেখাও

### 📌 এটা কী করে?
`head` file-এর **প্রথম দিক থেকে** কয়েকটা line দেখায়। Default হলো **প্রথম ১০টা line**।

### 📌 Syntax:
```bash
head [options] filename
```

### 📌 Basic Example:
```bash
head myfile.txt
```
**Output:**
```
Line 1: This is the first line
Line 2: Hello from Linux
Line 3: DevOps is awesome
Line 4: Learning is fun
Line 5: cat command rocks
Line 6: tail and head are useful
Line 7: log files are important
Line 8: keep going!
Line 9: almost there
Line 10: this is the last line
```
> (আমাদের file-এ মোট ১০ টা line তাই সব দেখাচ্ছে)

### 📌 Useful Options:

#### `-n` → কতটা line দেখাবে নির্দিষ্ট করো
```bash
head -n 3 myfile.txt
```
**Output:**
```
Line 1: This is the first line
Line 2: Hello from Linux
Line 3: DevOps is awesome
```

#### Short form (number সরাসরি দাও):
```bash
head -3 myfile.txt
```
> এটাও same কাজ করে।

#### Multiple files একসাথে:
```bash
head -n 2 file1.txt file2.txt
```
**Output:**
```
==> file1.txt <==
(first 2 lines)

==> file2.txt <==
(first 2 lines)
```

### 🏭 DevOps-এ `head`-এর Use Case:
- CSV file-এর header দেখা: `head -n 1 data.csv`
- Log file-এর শুরু দেখা
- Script-এর প্রথম কয়েক line দেখা (shebang আছে কিনা চেক)
- Config file-এর শুরু দেখা


## 5️⃣ `tail` - File-এর শেষ দেখাও ⭐

### 📌 এটা কী করে?
`tail` file-এর **শেষ দিক থেকে** কয়েকটা line দেখায়। Default হলো **শেষ ১০টা line**।

> 💡 **DevOps-এ `tail` সবচেয়ে গুরুত্বপূর্ণ command!** Log file monitor করতে এটা ছাড়া চলেই না।

### 📌 Syntax:
```bash
tail [options] filename
```

### 📌 Basic Example:
```bash
tail myfile.txt
```
**Output:**
```
Line 1: This is the first line
...
Line 10: this is the last line
```

### 📌 Useful Options:

#### `-n` → শেষের কতটা line দেখাবে
```bash
tail -n 3 myfile.txt
```
**Output:**
```
Line 8: keep going!
Line 9: almost there
Line 10: this is the last line
```

#### Short form:
```bash
tail -3 myfile.txt
```


### ⭐ `-f` → Real-time Log Follow করো! (সবচেয়ে Important!)

```bash
tail -f /var/log/syslog
```

এই command টা log file-কে **live monitor** করে। যখনই নতুন কিছু log-এ লেখা হয়, সাথে সাথে আপনার terminal-এ দেখাবে।

**বেরিয়ে আসতে:** `Ctrl + C`

#### `-f` with multiple files:
```bash
tail -f /var/log/syslog /var/log/auth.log
```

#### `-F` → File recreate হলেও follow করতে থাকো:
```bash
tail -F /var/log/app.log
```
> 💡 `-f` vs `-F` পার্থক্য: Log rotation-এ পুরনো file বন্ধ হয়ে নতুন file তৈরি হয়। `-F` সেই নতুন file-কেও automatically follow করে।

#### `+n` → নির্দিষ্ট line থেকে শেষ পর্যন্ত দেখাও:
```bash
tail -n +5 myfile.txt
```
**Output:** Line 5 থেকে শেষ পর্যন্ত সব line দেখাবে।

### 🏭 DevOps-এ `tail`-এর Use Case:
```bash
# Application log real-time দেখো
tail -f /var/log/nginx/access.log

# Error log দেখো
tail -f /var/log/nginx/error.log

# System log দেখো
tail -f /var/log/syslog

# শেষ ১০০ line দেখো
tail -n 100 /var/log/app.log

# Deployment-এর সময় log watch করো
tail -F /var/log/deploy.log
```


## 6️⃣ `tac` - File উল্টো করে দেখাও

### 📌 এটা কী করে?
`tac` হলো `cat`-এর উল্টো! (**cat** উল্টো করলে **tac** 😄)। এটা file-এর **শেষ line থেকে শুরু করে** সব দেখায়।

### 📌 Syntax:
```bash
tac filename
```

### 📌 Example:
```bash
tac myfile.txt
```
**Output:**
```
Line 10: this is the last line
Line 9: almost there
Line 8: keep going!
Line 7: log files are important
Line 6: tail and head are useful
Line 5: cat command rocks
Line 4: Learning is fun
Line 3: DevOps is awesome
Line 2: Hello from Linux
Line 1: This is the first line
```

### 🏭 DevOps-এ `tac`-এর Use Case:
```bash
# Log file উল্টো করে দেখো (নতুন entries আগে দেখাবে)
tac /var/log/syslog | less

# History উল্টো দেখো
history | tac | less
```


## 🔄 সব Commands-এর তুলনামূলক চার্ট

| Command | কতটুকু দেখায় | Scrollable? | Real-time? | Best For |
|---------|-------------|-------------|------------|----------|
| `cat` | পুরোটা | ❌ | ❌ | ছোট file |
| `less` | Page by page | ✅ (দুদিকে) | ✅ (`+F`) | বড় file read |
| `more` | Page by page | ✅ (শুধু নিচে) | ❌ | পুরনো systems |
| `head` | শুরুর অংশ | ❌ | ❌ | Header/শুরু দেখা |
| `tail` | শেষের অংশ | ❌ | ✅ (`-f`) | Log monitoring |
| `tac` | পুরোটা উল্টো | ❌ | ❌ | Latest entries আগে |


## 💡 DevOps Real-World Scenario

### Scenario: আপনার web server-এ কিছু একটা সমস্যা হচ্ছে। কী করবেন?

```bash
# Step 1: Error log-এর শেষ ৫০ line দেখো
tail -n 50 /var/log/nginx/error.log

# Step 2: Live log watch করো (নতুন request আসছে কিনা দেখো)
tail -f /var/log/nginx/access.log

# Step 3: Config file দেখো
cat /etc/nginx/nginx.conf

# Step 4: বড় log file search করে দেখো
less /var/log/nginx/error.log
# তারপর /error লিখে search করো
```

## 📝 Quick Summary

- ✅ **`cat`** → পুরো file একসাথে দেখাও, ছোট file বা quick check-এর জন্য
- ✅ **`less`** → বড় file scroll করে দেখার জন্য সেরা, search করা যায়
- ✅ **`more`** → `less`-এর পুরনো version, এখন কম use হয়
- ✅ **`head`** → File-এর শুরুর অংশ দেখো, default ১০ line
- ✅ **`tail`** → File-এর শেষের অংশ দেখো, `-f` দিয়ে real-time monitor করো
- ✅ **`tac`** → File উল্টো করে দেখাও, latest log entries দেখতে useful


## 🏋️ Practice Tasks

এই tasks গুলো নিজে নিজে করে দেখুন:

**Task 1:**
```bash
# /etc/passwd file দিয়ে practice করো
cat /etc/passwd          # পুরোটা দেখাও
head -n 5 /etc/passwd    # প্রথম ৫ line দেখাও
tail -n 5 /etc/passwd    # শেষ ৫ line দেখাও
```

**Task 2:**
```bash
# নিজে একটা ২০ line এর file বানান
# তারপর head ও tail দিয়ে বিভিন্ন সংখ্যক line দেখুন
# less দিয়ে open করে search করুন
```

**Task 3:**
```bash
# System log দেখার চেষ্টা করুন
sudo tail -f /var/log/syslog
# (Ctrl+C দিয়ে বেরিয়ে আসুন)
# অথবা
sudo less /var/log/syslog
```

## ⏭️ What's Next?

**Chapter 1 - Lesson 6: Finding Files & Searching**
> `find`, `locate`, `which`, `whereis` Linux-এ যেকোনো file খুঁজে বের করার সব উপায়। DevOps-এ এটা অত্যন্ত important!
