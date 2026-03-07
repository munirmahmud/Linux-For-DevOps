# Chapter 2 - Lesson 3: Special Permissions (SUID, SGID, Sticky Bit)

**Chapter 2 | Lesson 3 of 10**


## 🎯 এই Lesson-এ কী শিখবো?

আগের lesson-এ আমরা `chmod` এবং `chown` শিখেছিলাম। এখন আমরা শিখবো **Special Permissions** — এগুলো হলো Linux-এর কিছু "বিশেষ ক্ষমতা" যেগুলো normal `rwx` permission-এর বাইরে।

তিনটি Special Permission আছে:
| Permission | নাম | Symbol |
|---|---|---|
| **SUID** | Set User ID | `s` (owner execute জায়গায়) |
| **SGID** | Set Group ID | `s` (group execute জায়গায়) |
| **Sticky Bit** | Sticky Bit | `t` (others execute জায়গায়) |


## 1. SUID - Set User ID

**Concept টা কী?** একটা real-world analogy দিয়ে বুঝি:

> ধরুন একটা office-এ একটা **locked cabinet** আছে। Cabinet খুলতে পারে শুধু Manager। কিন্তু একটা নির্দিষ্ট **magic key** আছে যেটা যেকোনো employee ব্যবহার করলে, সে সাময়িকভাবে **Manager-এর ক্ষমতা পায়** শুধুমাত্র সেই cabinet খোলার জন্য।

Linux-এ **SUID** ঠিক এই কাজটাই করে।


### SUID কীভাবে কাজ করে?

যখন কোনো file-এ **SUID set** থাকে, তখন যেকোনো user সেই file **execute** করলে সে temporarily **file-এর owner-এর permission** পায়।

**Real Example:**
`/usr/bin/passwd` এই command দিয়ে আমরা password change করি।

```bash
ls -l /usr/bin/passwd
```

**Output:**
```
-rwsr-xr-x 1 root root 68208 Mar 14 2023 /usr/bin/passwd
```

লক্ষ্য করুন → owner-এর execute position-এ `x` এর জায়গায় **`s`** আছে।

এর মানে হলো:
- `/usr/bin/passwd` file-এর **owner হলো root**
- যখন একজন normal user (যেমন `john`) এই command run করে → সে temporarily **root-এর মতো** `/etc/shadow` file লিখতে পারে
- কিন্তু সে শুধু **নিজের password-ই** change করতে পারে কারণ program-এর ভেতরে logic আছে


### SUID Set ও Remove করা

**SUID Set করা (Symbolic):**
```bash
chmod u+s filename
```

**SUID Set করা (Numeric):**
```bash
chmod 4755 filename
```
> এখানে **`4`** হলো SUID-এর numeric value। `755` হলো normal permission।

**SUID Remove করা:**
```bash
chmod u-s filename
```


### হাতে-কলমে Example:

```bash
# একটা test file তৈরি করুন
touch /tmp/testfile
chmod 755 /tmp/testfile

# SUID set করুন
chmod u+s /tmp/testfile

# দেখুন কী হলো
ls -l /tmp/testfile
```

**Output:**
```
-rwsr-xr-x 1 user user 0 Jun 10 10:00 /tmp/testfile
```


### Capital `S` vs Lowercase `s`

```
-rwSr-xr-x   → Capital S মানে: SUID set আছে কিন্তু execute permission নেই (সমস্যা!)
-rwsr-xr-x   → Lowercase s মানে: SUID set আছে এবং execute permission-ও আছে (সঠিক)
```


### SUID files খোঁজা (DevOps Security কাজে লাগে):
```bash
find / -perm -4000 -type f 2>/dev/null
```

**এই command টা কী করে:**
- `find /` → পুরো system খোঁজে
- `-perm -4000` → SUID set আছে এমন files
- `-type f` → শুধু regular files
- `2>/dev/null` → error messages লুকায়


## 2. SGID - Set Group ID

### Concept টা কী?

> মনে করুন একটা **team project folder** আছে। সেই folder-এ যে-ই file তৈরি করুক না কেন সেই file automatically **team-এর group-এর ownership** পাবে। এতে সবাই সেই file access করতে পারবে।

SGID দুইভাবে কাজ করে:
1. **File-এ SGID** → Executable file run করলে file-এর group-এর permission পাওয়া যায়
2. **Directory-তে SGID** → সেই directory-তে যা-ই তৈরি হোক, parent directory-র group পাবে *(এটাই DevOps-এ বেশি ব্যবহার হয়)*


### Directory-তে SGID - Real DevOps Use Case

```bash
# একটা shared project folder তৈরি করুন
mkdir /project
chown :developers /project    # group হলো 'developers'
chmod 2775 /project           # 2 = SGID

# এখন developers group-এর যেকোনো member এই folder-এ
# file তৈরি করলে - সেই file-এর group হবে 'developers'
```

**দেখুন কীভাবে দেখা যায়:**
```bash
ls -ld /project
```

**Output:**
```
drwxrwsr-x 2 root developers 4096 Jun 10 10:00 /project
```

group-এর execute position-এ **`s`** দেখা যাচ্ছে → এটাই SGID।


### SGID Set ও Remove করা

**Symbolic:**
```bash
chmod g+s dirname
```

**Numeric:**
```bash
chmod 2755 dirname
```
> **`2`** হলো SGID-এর numeric value।

**Remove:**
```bash
chmod g-s dirname
```

### হাতে-কলমে Example:

```bash
# Group তৈরি করুন
groupadd developers

# Directory তৈরি করুন
mkdir /shared_project
chown :developers /shared_project
chmod 2770 /shared_project

# দেখুন
ls -ld /shared_project
```

**Output:**
```
drwxrws--- 2 root developers 4096 Jun 10 10:00 /shared_project
```

এখন এই directory-তে যে file-ই তৈরি হবে সেটার group হবে `developers`।

---

### SGID files/directories খোঁজা:
```bash
find / -perm -2000 -type f 2>/dev/null   # files
find / -perm -2000 -type d 2>/dev/null   # directories
```


## 3. Sticky Bit

**Concept টা কী?**

> ধরুন একটা **public library** আছে। সবাই বই রাখতে পারে, সবাই বই পড়তে পারে। কিন্তু **নিজের বই শুধু নিজেই নিতে পারবে** অন্য কেউ নিতে পারবে না।

Linux-এ **Sticky Bit** ঠিক এই কাজটাই করে।


### Sticky Bit কীভাবে কাজ করে?

কোনো **directory-তে Sticky Bit** set থাকলে:
- সবাই সেই directory-তে file তৈরি করতে পারবে
- সবাই সব file দেখতে পারবে
- কিন্তু **নিজের file শুধু নিজেই delete করতে পারবে** (অথবা root)


### সবচেয়ে বিখ্যাত Example `/tmp`

```bash
ls -ld /tmp
```

**Output:**
```
drwxrwxrwt 10 root root 4096 Jun 10 10:00 /tmp
```

others-এর execute position-এ **`t`** দেখা যাচ্ছে → এটাই Sticky Bit!

`/tmp` folder-এ সব user-ই file তৈরি করতে পারে। কিন্তু `user_A` কখনো `user_B`-এর file delete করতে পারবে না।


### Sticky Bit Set ও Remove করা

**Symbolic:**
```bash
chmod +t dirname
```

**Numeric:**
```bash
chmod 1777 dirname
```
> **`1`** হলো Sticky Bit-এর numeric value।

**Remove:**
```bash
chmod -t dirname
```


### হাতে-কলমে Example:

```bash
# একটা shared directory তৈরি করুন
mkdir /shared_temp
chmod 1777 /shared_temp

# দেখুন
ls -ld /shared_temp
```

**Output:**
```
drwxrwxrwt 2 root root 4096 Jun 10 10:00 /shared_temp
```


### Capital `T` vs Lowercase `t`

```
drwxrwxrwT   → Capital T: Sticky Bit আছে কিন্তু others-এর execute নেই
drwxrwxrwt   → Lowercase t: Sticky Bit আছে এবং execute-ও আছে (সঠিক)
```


## তিনটি Special Permission - একসাথে তুলনা

| Feature | SUID | SGID | Sticky Bit |
|---|---|---|---|
| **Numeric Value** | `4` | `2` | `1` |
| **Symbol** | `s` (owner) | `s` (group) | `t` (others) |
| **File-এ কাজ** | Owner-এর UID দিয়ে run হয় | Group-এর GID দিয়ে run হয় | কোনো effect নেই |
| **Directory-তে কাজ** | সাধারণত ব্যবহার হয় না | নতুন files parent-এর group পায় | নিজের file শুধু নিজে delete করতে পারে |
| **Real Example** | `/usr/bin/passwd` | `/shared_project` | `/tmp` |


## Numeric Special Permission - সব একসাথে

Special permissions numeric mode-এ **4-digit** হয়:

```
chmod XYZW filename
  │││└── Others permission
  ││└─── Group permission
  │└──── Owner permission
  └───── Special permission (4=SUID, 2=SGID, 1=Sticky)
```

**Examples:**
```bash
chmod 4755 file    # SUID + rwxr-xr-x
chmod 2755 dir     # SGID + rwxr-xr-x
chmod 1777 dir     # Sticky + rwxrwxrwx
chmod 6755 file    # SUID + SGID একসাথে (4+2=6)
chmod 7777 dir     # সব তিনটা একসাথে (4+2+1=7)
```


## সব Special Permission একসাথে দেখুন

```bash
# SUID files
find / -perm -4000 -type f 2>/dev/null

# SGID files/dirs
find / -perm -2000 2>/dev/null

# Sticky Bit directories
find / -perm -1000 -type d 2>/dev/null
```


## DevOps Security Warning

SUID/SGID files security risk হতে পারে কারণ:
- যদি কোনো **malicious program**-এ SUID set থাকে → attacker root access পেতে পারে
- তাই regularly `find` দিয়ে unexpected SUID files খোঁজা উচিত
- যেসব file-এ SUID/SGID দরকার নেই → সেখান থেকে remove করা উচিত


## 📝 Quick Summary

- ✅ **SUID** → File run করলে owner-এর permission পাওয়া যায় | `chmod u+s` বা `chmod 4xxx`
- ✅ **SGID** → Directory-তে নতুন file parent-এর group পায় | `chmod g+s` বা `chmod 2xxx`
- ✅ **Sticky Bit** → Shared directory-তে নিজের file শুধু নিজে delete করতে পারে | `chmod +t` বা `chmod 1xxx`
- ✅ Lowercase `s`/`t` = সঠিক | Capital `S`/`T` = execute permission missing
- ✅ Numeric: SUID=4, SGID=2, Sticky=1


## 🏋️ Practice Tasks

**Task 1:**
```bash
# /tmp directory-তে sticky bit আছে কিনা verify করুন
ls -ld /tmp
# এবং /usr/bin/passwd এ SUID আছে কিনা দেখুন
ls -l /usr/bin/passwd
```

**Task 2:**
```bash
# নিজে একটা directory তৈরি করুন এবং SGID set করুন
mkdir ~/test_sgid
chmod 2755 ~/test_sgid
ls -ld ~/test_sgid
# output-এ 's' দেখতে পাচ্ছেন?
```

**Task 3:**
```bash
# System-এ সব SUID files খুজুন এবং list করুন
find / -perm -4000 -type f 2>/dev/null
# কতটা file পেলেন? কোনগুলো চেনা মনে হচ্ছে?
```

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 4: `umask` - Default Permission Control**
> নতুন file বা directory তৈরি হলে automatically কোন permission থাকবে সেটা `umask` দিয়ে control করা যায়। DevOps-এ default security setting হিসেবে এটা খুবই গুরুত্বপূর্ণ।

Special permissions একটু tricky concept কিন্তু আপনি এখন জানেন যে `/tmp`-এ `t` কেন আছে এবং `passwd` command কীভাবে root ছাড়াও কাজ করে! Practice tasks গুলো করে দেখুন, কোনো সমস্যা হলে জানাতে ভুলবেন না। *Happy Learning* 😊
