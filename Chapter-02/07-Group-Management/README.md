# Chapter 2 - Lesson 7: Group Management (groupadd, groupmod, groupdel, gpasswd)

**Chapter 2 | Lesson 7 of 10**

## 🎯 এই Lesson-এ কী শিখবো?

আগের Lesson-এ আমরা **User Management** শিখেছিলাম। এই Lesson-এ শিখবো **Group Management**। মানে Linux-এ group কীভাবে তৈরি করতে হয়, edit করতে হয়, delete করতে হয় এবং group-এ user যোগ/বাদ দিতে হয়।


## Group কী? - Real-World Analogy

কল্পনা করুন আপনি একটা company-তে কাজ করেন।

> **Company = Linux System**
> **Department = Group**
> **Employee = User**

- Developer department-এর সবাই একই code folder access করতে পারে
- HR department-এর সবাই একই payroll folder access করতে পারে
- Security team একই log folder access করতে পারে

Linux-এও ঠিক এভাবে কাজ করে। **Group** মানে হলো এক ধরনের "permission bundle", যে group-এ আছেন, সেই group-এর সব permission আপনার কাছে চলে আসে।


## Linux-এ দুই ধরনের Group হয়

| Type | বাংলায় মানে | Example |
|------|------------|---------|
| **Primary Group** | User তৈরির সময় automatically যে group হয় | `munir` user → primary group `munir` |
| **Secondary Group** | পরে manually যোগ করা group | `munir` কে `developers` group-এ add করা |

একটা user-এর **শুধু একটাই Primary Group** থাকতে পারে, কিন্তু **অনেকগুলো Secondary Group** থাকতে পারে।


## Commands Overview

| Command | কাজ কী |
|---------|--------|
| `groupadd` | নতুন group তৈরি করে |
| `groupmod` | existing group modify করে |
| `groupdel` | group delete করে |
| `gpasswd` | group-এ password দেয় এবং member manage করে |
| `groups` | কোন user কোন group-এ আছে দেখায় |
| `id` | user-এর UID, GID এবং সব group দেখায় |
| `getent group` | group database থেকে তথ্য দেখায় |


## 1️⃣ groupadd - নতুন Group তৈরি করা

### সহজ ভাষায়:
`groupadd` দিয়ে Linux-এ নতুন group তৈরি করা হয়।

### Syntax:
```bash
groupadd [options] group_name
```

### Basic Example:
```bash
sudo groupadd developers
```
**কী হবে:** `developers` নামে একটা নতুন group তৈরি হবে।

**Verify করো:**
```bash
getent group developers
```
**Expected Output:**
```
developers:x:1001:
```

এখানে:
- `developers` = group-এর নাম
- `x` = password (encrypted, `/etc/gshadow`-এ থাকে)
- `1001` = GID (Group ID)
- শেষের খালি জায়গা = এখনো কোনো member নেই


### Specific GID দিয়ে group তৈরি:
```bash
sudo groupadd -g 1500 devops
```
**কী হবে:** GID `1500` দিয়ে `devops` group তৈরি হবে।

**Verify:**
```bash
getent group devops
```
**Output:**
```
devops:x:1500:
```


### System Group তৈরি করা:
```bash
sudo groupadd -r appgroup
```

**`-r` মানে কী?**
> System service-এর জন্য group তৈরি হয়। GID number সাধারণত 1000-এর নিচে থাকে।
> যেমন: `nginx`, `docker`, `mysql` - এগুলো system group।


### groupadd-এর Important Options:

| Option | কাজ |
|--------|-----|
| `-g GID` | নির্দিষ্ট GID দিয়ে group তৈরি |
| `-r` | system group তৈরি |
| `-f` | group already থাকলে error না দিয়ে চুপ থাকে |


## 2️⃣ groupmod - Existing Group Modify করা

### সহজ ভাষায়:
তৈরি হয়ে যাওয়া group-এর নাম বা GID পরিবর্তন করতে `groupmod` ব্যবহার করুন।

### Syntax:
```bash
groupmod [options] group_name
```


### Group-এর নাম পরিবর্তন:
```bash
sudo groupmod -n backend developers
```
**কী হবে:** `developers` group-এর নাম পরিবর্তন হয়ে `backend` হয়ে যাবে।

**Verify:**
```bash
getent group backend
```
**Output:**
```
backend:x:1001:
```


### Group-এর GID পরিবর্তন:
```bash
sudo groupmod -g 2000 backend
```
**কী হবে:** `backend` group-এর GID `2000` হয়ে যাবে।


### groupmod-এর Important Options:

| Option | কাজ |
|--------|-----|
| `-n new_name` | group-এর নাম পরিবর্তন |
| `-g new_GID` | GID পরিবর্তন |


## 3️⃣ groupdel - Group Delete করা

### সহজ ভাষায়:
যে group আর দরকার নেই, সেটা `groupdel` দিয়ে মুছে ফেলো।

### Syntax:
```bash
groupdel group_name
```

### Example:
```bash
sudo groupdel backend
```
**কী হবে:** `backend` group মুছে যাবে।

**Verify:**
```bash
getent group backend
```
**Output:** কিছুই দেখাবে না, মানে group আর নেই।


### ⚠️ গুরুত্বপূর্ণ সতর্কতা:

- কোনো user-এর **Primary Group** delete করা যাবে না!
- আগে সেই user-কে delete করতে হবে অথবা তার primary group পরিবর্তন করতে হবে।

```bash
sudo groupdel munir   # munir যদি কারো primary group হয়
```
**Error Output:**
```
groupdel: cannot remove the primary group of user 'munir'
```


## 4️⃣ gpasswd - Group Password ও Member Management

### সহজ ভাষায়:
`gpasswd` হলো group management-এর সবচেয়ে powerful tool। এটা দিয়ে:
- Group-এ user **add** করা যায়
- Group থেকে user **remove** করা যায়
- Group **administrator** সেট করা যায়
- Group-এ **password** দেওয়া যায়


### Group-এ User Add করা:
```bash
sudo gpasswd -a munir developers
```
**কী হবে:** `munir` user-কে `developers` group-এ add করা হবে।

**Output:**
```
Adding user munir to group developers
```

**Verify:**
```bash
getent group developers
```
**Output:**
```
developers:x:1001:munir
```


### Group থেকে User Remove করা:
```bash
sudo gpasswd -d munir developers
```
**কী হবে:** `developers` group থেকে `munir` বাদ যাবে।

**Output:**
```
Removing user munir from group developers
```


### একসাথে Multiple User Set করা:
```bash
sudo gpasswd -M munir,alice,bob developers
```
**কী হবে:** `developers` group-এ `munir`, `alice`, `bob` - এই তিনজন থাকবে।

> ⚠️ সাবধান: `-M` option আগের সব member মুছে দিয়ে নতুন list সেট করে!


### Group Administrator সেট করা:
```bash
sudo gpasswd -A munir developers
```
**কী হবে:** `munir` এখন `developers` group-এর administrator হবে।
মানে `munir` নিজেই এই group-এ member add/remove করতে পারবে (sudo ছাড়াই)।


### Group Password সেট করা:
```bash
sudo gpasswd developers
```
**কী হবে:** `developers` group-এর জন্য password চাইবে।

**Group password কেন দরকার?**
> অন্য user যদি temporarily এই group-এ join করতে চায়, তখন `newgrp developers` command দিয়ে password দিলে join করতে পারবে।


### Group Password Remove করা:
```bash
sudo gpasswd -r developers
```


### gpasswd-এর Important Options:

| Option | কাজ |
|--------|-----|
| `-a user` | group-এ user add করে |
| `-d user` | group থেকে user remove করে |
| `-M user1,user2` | group-এর পুরো member list সেট করে |
| `-A user` | group administrator সেট করে |
| `-r` | group password remove করে |


## 5️⃣ groups ও id - Group দেখার Commands

### কোনো user কোন কোন group-এ আছে:
```bash
groups munir
```
**Output:**
```
munir : munir developers docker sudo
```
মানে `munir` চারটা group-এ আছে।

---

### নিজের group দেখা:
```bash
groups
```
**Output:**
```
youruser sudo docker developers
```


### id command দিয়ে বিস্তারিত দেখা:
```bash
id munir
```
**Output:**
```
uid=1001(munir) gid=1001(munir) groups=1001(munir),1002(developers),999(docker)
```

এখানে:
- `uid=1001(munir)` = User ID
- `gid=1001(munir)` = Primary Group ID
- `groups=...` = সব group (primary + secondary)


## 6️⃣ usermod দিয়ে Group Management

> আগের lesson-এ `usermod` শিখেছিলাম। এটাও group management-এ ব্যবহার হয়।

### Secondary Group-এ User Add করা (existing group ঠিক রেখে):
```bash
sudo usermod -aG docker munir
```
**কী হবে:** `munir`-কে `docker` group-এ add করা হবে, আগের অন্য group গুলো ঠিক থাকবে।

> ⚠️ `-aG`-এর `-a` (append) না দিলে আগের সব secondary group মুছে যাবে!


### Primary Group পরিবর্তন:
```bash
sudo usermod -g developers munir
```
**কী হবে:** `munir`-এর primary group `developers` হয়ে যাবে।


## 7️⃣ /etc/group ফাইল - গভীরে দেখি

সব group-এর তথ্য এই ফাইলে থাকে:

```bash
cat /etc/group
```

**Output (কিছু অংশ):**
```
root:x:0:
sudo:x:27:munir,alice
docker:x:999:munir
developers:x:1001:munir,alice,bob
devops:x:1500:
```

### প্রতিটা line-এর format:
```
group_name : password : GID : member_list
```

| Field | মানে |
|-------|------|
| `group_name` | group-এর নাম |
| `x` | password `/etc/gshadow`-এ আছে |
| `GID` | Group ID number |
| `member_list` | comma দিয়ে আলাদা করা সব member-এর নাম |


## 8️⃣ Real-World DevOps Scenario

### Scenario: নতুন DevOps team setup করা

```bash
# Step 1: Group তৈরি করুন
sudo groupadd devops-team
sudo groupadd docker-users
sudo groupadd deploy-access

# Step 2: User তৈরি করুন
sudo useradd -m alice
sudo useradd -m bob
sudo useradd -m charlie

# Step 3: Group-এ user add করুন
sudo gpasswd -a alice devops-team
sudo gpasswd -a bob devops-team
sudo gpasswd -a charlie devops-team

# Step 4: Docker access দাও শুধু alice ও bob-কে
sudo gpasswd -a alice docker-users
sudo gpasswd -a bob docker-users

# Step 5: Deploy access শুধু alice-কে
sudo gpasswd -a alice deploy-access

# Step 6: Verify করুন
getent group devops-team
getent group docker-users
id alice
```

**Final Output:**
```
devops-team:x:1502:alice,bob,charlie
docker-users:x:1503:alice,bob
uid=1003(alice) gid=1003(alice) groups=1003(alice),1502(devops-team),1503(docker-users),1504(deploy-access)
```

এভাবে real production-এ team permission structure তৈরি করা হয়!


## 📝 Quick Summary

- `groupadd` → নতুন group তৈরি করে (`-g` দিয়ে specific GID, `-r` দিয়ে system group)
- `groupmod` → group-এর নাম (`-n`) বা GID (`-g`) পরিবর্তন করে
- `groupdel` → group delete করে (primary group delete করা যায় না)
- `gpasswd -a` → group-এ user add করে
- `gpasswd -d` → group থেকে user remove করে
- `gpasswd -M` → পুরো member list একসাথে সেট করে (সাবধানে!)
- `gpasswd -A` → group administrator সেট করে
- `usermod -aG` → secondary group-এ user add করে (-a না দিলে বিপদ!)
- `groups` / `id` → user কোন group-এ আছে দেখায়
- `/etc/group` → সব group-এর তথ্যের ডেটাবেজ


## 🏋️ Practice Tasks

এগুলো নিজে নিজে try করুন:

**Task 1:**
`webteam` নামে একটা group তৈরি করুন GID `2100` দিয়ে। তারপর `testuser` নামে একজন user তৈরি করুন এবং তাকে `webteam` group-এ add করুন। `id testuser` দিয়ে verify করুন।

**Task 2:**
`webteam` group-এর নাম পরিবর্তন করে `frontend` রাখুন। তারপর `/etc/group` ফাইলে দেখুন নাম সত্যিই পরিবর্তন হয়েছে কিনা।

**Task 3:**
`gpasswd -M` ব্যবহার করে `frontend` group-এ একসাথে দুইজন user সেট করুন। তারপর একজনকে `gpasswd -d` দিয়ে remove করুন এবং `getent group frontend` দিয়ে confirm করুন।

---

## ⏭️ What's Next?

**Chapter 2 - Lesson 8: sudo & sudoers file**
`/etc/sudoers`, `visudo`, এবং sudo rules কীভাবে specific user-কে specific command-এর জন্য root permission দেওয়া যায়। DevOps-এর জন্য এটা অত্যন্ত গুরুত্বপূর্ণ একটা lesson! *Happy Learning* 🚀
