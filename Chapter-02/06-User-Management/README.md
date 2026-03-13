# Chapter 2 - Lesson 6: User Management

**Chapter 2 | Lesson 6 of 10**


## 🎯 এই Lesson-এ কী শিখবো?

আজকে আমরা শিখবো Linux-এ **User Management** মানে নতুন user তৈরি করা, তাদের modify করা, delete করা, এবং password সেট করা।


## Linux-এ User কী?

Linux একটি **multi-user operating system**। মানে একই সময়ে অনেক মানুষ একটি Linux system ব্যবহার করতে পারে।

**analogy:**
> একটি অফিস building-এর মতো চিন্তা করুন। Building-এ অনেক employee আছে। প্রত্যেকের নিজের **ID card** আছে, নিজের **desk** আছে, এবং কিছু জায়গায় access আছে, কিছু জায়গায় নেই। Linux-এ প্রতিটি user ঠিক এরকমই।

প্রতিটি user-এর থাকে:
- একটি **username** (নাম)
- একটি **UID** (User ID - একটি unique number)
- একটি **home directory** (নিজের folder)
- একটি **default shell** (bash, sh, etc.)

## User-সম্পর্কিত গুরুত্বপূর্ণ Files

| File | কী আছে এখানে? |
|------|--------------|
| `/etc/passwd` | সব user-এর basic info |
| `/etc/shadow` | সব user-এর encrypted password |
| `/etc/group` | সব group-এর info |

এই files-গুলো Lesson 10-এ বিস্তারিত দেখবো। আজকে শুধু জেনে রাখলাম এগুলো exist করে।


## Command 1: `useradd` - নতুন User তৈরি করা

### Plain English:
`useradd` দিয়ে Linux system-এ নতুন একটি user account তৈরি করা হয়।

### Basic Syntax:
```bash
useradd [options] username
```

### সবচেয়ে গুরুত্বপূর্ণ Options:

| Option | মানে কী? |
|--------|---------|
| `-m` | Home directory তৈরি করো (`/home/username`) |
| `-d /path` | Custom home directory দাও |
| `-s /bin/bash` | Default shell সেট করো |
| `-c "Full Name"` | Comment/Full name দাও |
| `-u 1050` | Custom UID দাও |
| `-g groupname` | Primary group দাও |
| `-G g1,g2` | Secondary groups দাও |
| `-e 2025-12-31` | Account expiry date দাও |


### Example 1: সবচেয়ে Simple User তৈরি

```bash
sudo useradd munir
```

এটি `munir` নামে একটি user তৈরি করবে। কিন্তু সমস্যা হলো - **home directory তৈরি হবে না** এভাবে।


### Example 2: সঠিকভাবে User তৈরি (DevOps style)

```bash
sudo useradd -m -s /bin/bash -c "Munir Mahmud" munir
```

**প্রতিটি অংশের মানে:**
- `sudo` → root permission নিয়ে রান করো
- `useradd` → নতুন user তৈরি করো
- `-m` → `/home/munir` folder তৈরি করো
- `-s /bin/bash` → default shell হবে bash
- `-c "Munir Mahmud"` → full name হবে "munir"
- `munir` → username হবে "munir"

**Verify করো user তৈরি হয়েছে কিনা:**
```bash
getent passwd munir
```

**Expected Output:**
```
munir:x:1000:1000:munir:/home/munir:/bin/bash
```

**এই output-এর মানে (`:` দিয়ে আলাদা করা):**
```
username : password : UID : GID : Comment : HomeDir : Shell
munir     :    x    : 1000: 1000: munir :/home/munir:/bin/bash
```

> `x` মানে password `/etc/shadow`-এ encrypted আকারে আছে।


### Example 3: Home Directory আছে কিনা Check করা

```bash
ls /home/munir
```

**Expected Output:**
```
(empty - নতুন user, কোনো file নেই এখনো)
```


## Command 2: `passwd` - Password সেট করা

### Plain English:
User তৈরির পরে তাকে password দিতে হবে। `passwd` command দিয়ে এটি করা হয়।

### Syntax:
```bash
sudo passwd username
```

### Example:

```bash
sudo passwd munir
```

**Output:**
```
New password:
Retype new password:
passwd: password updated successfully
```

> ⚠️ Password টাইপ করার সময় screen-এ কিছু দেখাবে না। এটি normal। Linux security-র জন্য এটি করে।


### নিজের password পরিবর্তন করা:

```bash
passwd
```

শুধু `passwd` লিখলে currently logged-in user-এর password পরিবর্তন হবে। কোনো username দিতে হবে না।

### Password-সম্পর্কিত আরো Options:

```bash
sudo passwd -l munir    #  munir-এর account Lock করো
sudo passwd -u munir    #  munir-এর account Unlock করো
sudo passwd -e munir    # Force করো next login-এ password পরিবর্তন করতে
sudo passwd -d munir    # Password delete করো (passwordless login)
```


## Command 3: `usermod` - User Modify করা

### Plain English:
`usermod` দিয়ে existing user-এর কোনো কিছু পরিবর্তন করা যায়। যেমন username, shell, home directory, group ইত্যাদি।

### Syntax:
```bash
sudo usermod [options] username
```

### গুরুত্বপূর্ণ Options:

| Option | কী করে? |
|--------|---------|
| `-l newname` | Username পরিবর্তন করো |
| `-d /new/path` | Home directory পরিবর্তন করো |
| `-s /bin/zsh` | Shell পরিবর্তন করো |
| `-c "New Name"` | Comment/Full name পরিবর্তন করো |
| `-aG groupname` | নতুন group-এ add করো (existing রেখে) |
| `-G g1,g2` | Groups replace করো |
| `-L` | Account lock করো |
| `-U` | Account unlock করো |
| `-e 2026-03-17` | Expiry date পরিবর্তন করো |


### Example 1: User-এর Shell পরিবর্তন করা

```bash
sudo usermod -s /bin/sh munir
```

Verify:
```bash
getent passwd munir
```

Output:
```
munir:x:1000:1000:munir:/home/munir:/bin/sh
```

Shell এখন `/bin/sh` হয়ে গেছে।


### Example 2: User-কে একটি নতুন Group-এ Add করা

```bash
sudo usermod -aG sudo munir
```

> ⚠️ অত্যন্ত গুরুত্বপূর্ণ: `-aG` এর মধ্যে `-a` মানে **append** (যোগ করো)। যদি শুধু `-G` দাও, তাহলে আগের সব groups **মুছে যাবে** এবং শুধু নতুন group থাকবে!

Verify:
```bash
groups munir
```

Output:
```
munir : munir sudo
```


### Example 3: Username পরিবর্তন করা

```bash
sudo usermod -l munirul munir
```

এখন `munir` username টি `munirul` হয়ে গেছে।

> ⚠️ Note: Username পরিবর্তন হবে, কিন্তু home directory `/home/munir` থেকে যাবে। Home directory-ও পরিবর্তন করতে চাইলে:

```bash
sudo usermod -l munirul -d /home/munirul -m munir
```


## Command 4: `userdel` - User Delete করা

### Plain English:
`userdel` দিয়ে system থেকে একটি user account মুছে ফেলা হয়।

### Syntax:
```bash
sudo userdel [options] username
```

### Options:

| Option | কী করে? |
|--------|---------|
| (কোনো option নেই) | শুধু user account মুছবে, home directory থাকবে |
| `-r` | User account + home directory + mail spool সব মুছবে |
| `-f` | Force করে মুছবে (user currently logged in থাকলেও) |


### Example 1: User মুছো, Home Directory রাখো

```bash
sudo userdel munir
```

এখন `munir` user নেই, কিন্তু `/home/munir` folder এখনো আছে।


### Example 2: User + সব কিছু মুছো (সবচেয়ে বেশি ব্যবহার হয়)

```bash
sudo userdel -r munir
```

**Output:**
```
userdel: munir mail spool (/var/mail/munir) not found
```

> এই message টি normal। এর মানে mail spool ছিল না, কিন্তু user এবং home directory মুছে গেছে।

Verify:
```bash
ls /home/
# munir folder আর নেই

getent passwd munir
# কোনো output আসবে না কারন এই user আর নেই।
```


## Bonus: User Information দেখার Commands

### `id` - User-এর UID, GID এবং Groups দেখা

```bash
id munir
```

**Output:**
```
uid=1000(munir) gid=1000(munir) groups=1000(munir),27(sudo)
```


### `whoami` - আমার user নাম দেখাতে চাইলে?

```bash
whoami
```

**Output:**
```
ubuntu   (অথবা আপনার current username)
```

### `who` - System-এ এখন কে কে logged in?

```bash
who
```

**Output:**
```
ubuntu   pts/0        2025-01-15 10:30 (192.168.1.5)
```

---

### `last` - কে কখন login করেছিল?

```bash
last
```

**Output:**
```
ubuntu   pts/0        Thu Jan 15 10:30   still logged in
reboot   system boot  Thu Jan 15 10:25
```


## Real-World DevOps Scenario

### Scenario: নতুন Developer "Shakil" join করেছে team-এ

```bash
# ধাপ 1: User তৈরি করুন
sudo useradd -m -s /bin/bash -c "Shakil Ahmed" shakil

# ধাপ 2: Password দিন
sudo passwd shakil

# ধাপ 3: তাকে sudo access দিন (সে senior developer)
sudo usermod -aG sudo shakil

# ধাপ 4: Verify করুন
id shakil
groups shakil

# ধাপ 5: Shakil login করতে পারছে কিনা test করুন
su - shakil
whoami    # shakil দেখাবে
exit      # নিজের user-এ ফিরে আসো
```


### Scenario: একজন Employee চলে গেছে - Account বন্ধ করুন

```bash
# Option 1: Account Lock করুন (data রাখতে চাইলে)
sudo passwd -l shakil
# অথবা
sudo usermod -L shakil

# Option 2: Account সম্পূর্ণ Delete করুন
sudo userdel -r shakil
```


## 📝 Quick Summary

- `useradd -m -s /bin/bash -c "Name" username` → নতুন user তৈরি করো (সঠিকভাবে)
- `passwd username` → password সেট বা পরিবর্তন করো
- `usermod -aG groupname username` → user-কে group-এ add করো (`-a` ভুলো না!)
- `usermod -l newname oldname` → username পরিবর্তন করো
- `userdel -r username` → user এবং home directory মুছো
- `id username` → user-এর UID, GID, groups দেখো
- `getent passwd username` → user-এর full info দেখো
- `passwd -l` / `passwd -u` → account lock/unlock করো


## 🏋️ Practice Tasks

এগুলো নিজে নিজে try করুন:

**Task 1:**
`devops` নামে একটি নতুন user তৈরি করুন যার:
- Home directory থাকবে
- Shell হবে `/bin/bash`
- Full name হবে "DevOps User"
- তারপর তাকে একটি password দিবেন

**Task 2:**
`devops` user-কে `sudo` group-এ add করুন। তারপর `groups devops` দিয়ে verify করুন।

**Task 3:**
`devops` user-এর account lock করুন `passwd -l` দিয়ে, তারপর আবার unlock করুন। `/etc/shadow` file-এ কী পরিবর্তন হয় দেখুন:

```bash
sudo grep devops /etc/shadow
```
Lock করলে password-এর আগে `!` চিহ্ন আসে, এটি লক্ষ্য করুন!

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 7: Group Management**
`groupadd`, `groupmod`, `groupdel`, `gpasswd` দিয়ে Linux groups কীভাবে manage করতে হয় - সেটাই পরের lesson-এ শিখবো! 🚀
