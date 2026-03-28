# Chapter 6 - Lesson 1: Service Management & Systemd

**Chapter 6 | Lesson 1 of 8**


## 🎯 এই Lesson-এ আমরা যা শিখবো
- Linux কীভাবে boot হওয়ার পর সব কিছু চালু করে
- Init system কী এবং কেন দরকার
- Systemd কী এবং কেন এটি সবচেয়ে popular
- Units এবং Targets কী

## Linux Boot হলে কী হয়?

আপনি যখন একটা Linux machine চালু করেন, তখন একটার পর একটা কাজ হয়:

```
BIOS/UEFI → Bootloader (GRUB) → Linux Kernel load → First Process চালু হয়
```

সেই "First Process" টাই হলো Init System।

**Real-world analogy:**

> মনে করেন আপনি একটা বড় অফিস খুলেছেন। অফিসে ঢোকার পরে প্রথমে একজন **Manager** আসে।
> সে বাকি সবাইকে ডাকে, তাদের কাজ বলে দেয়, কে কার পরে আসবে সেটা ঠিক করে।
> Linux-এ সেই **Manager** হলো **Init System।**


## Init System এর ইতিহাস - পুরনো vs নতুন

| যুগ | Init System | সমস্যা |
|-----|------------|--------|
| পুরনো (1980s) | **SysV init** | ধীর, serial (একটার পর একটা) |
| মাঝামাঝি | **Upstart** (Ubuntu) | ভালো, কিন্তু complex |
| আধুনিক (এখন) | **Systemd** | দ্রুত, parallel, সব-কিছু একসাথে |

> **SysV init** এর সমস্যা ছিল - সে সব service একটার পর একটা চালু করত।
> যেন ক্লাসে ৩০ জন ছাত্র আছে, কিন্তু রোল কল হয় একজন একজন করে - অনেক সময় লাগে!
>
> **Systemd** সব service **parallel** এ চালু করে - সবাইকে একসাথে ডাকে।


## Systemd কী?


একটা গল্প দিয়ে শুরু করি। মনে করুন আপনি একটা বড় হোটেল এর মালিক।

হোটেলে অনেক কিছু চলে যেমনঃ
- রেস্টুরেন্ট চলছে
- এসি চলছে
- WiFi চলছে
- Security guard কাজ করছে
- লিফট চলছে
- CCTV চলছে

এখন হোটেল সকালে open করার সময় কী হয়? 

কেউ একজন সব কিছু **চালু** করে দেয়। সে ঠিক করে -
> *আগে বিদ্যুৎ চালু হবে, তারপর এসি, তারপর WiFi, তারপর বাকি সব।*

আর কেউ কোনো সমস্যায় পড়লে সেই একই লোককে জানায়।

Linux-এ সেই লোকটার নাম হলো → Systemd


### Linux চালু হলে আসলে কী ঘটে?

আপনি PC-র power button চাপলেন এখন যা যা ঘটে তাহলো:

```
Power Button চাপলেন
        ↓
BIOS/UEFI চালু হয় (hardware check করে)
        ↓
GRUB চালু হয় (Linux kernel কে ডাকে)
        ↓
Linux Kernel চালু হয় (RAM, CPU, Disk চেনে)
        ↓
Kernel বলে - "আমার কাজ শেষ, এবার কেউ বাকি কাজ সামলাও"
        ↓
Systemd চালু হয় ← এখানেই Systemd এর কাজ শুরু
        ↓
Systemd সব service, program চালু করে
        ↓
আপনি login screen দেখতে পান
```

> Kernel নিজে জানে না WiFi কীভাবে চালাতে হয়, SSH কীভাবে চালাতে হয়।
> এই কাজগুলো **Systemd** করে।


### Systemd হলো PID 1

Linux-এ যে process সবার আগে চালু হয়, সে পায় **PID = 1** (Process ID নম্বর ১)।

Systemd-ই সেই process।

```bash
ps -p 1
```

```
PID TTY    TIME  CMD
  1  ?   00:00:02 systemd
```

**সহজ কথায়:**
> Linux-এ সব process এর একটা নম্বর আছে।
> নম্বর ১ মানে - সবার বাপ। সেটাই Systemd।
> বাকি সব process এর জন্ম Systemd এর হাত দিয়ে।


## Systemd শুধু Service চালায় না - আরো অনেক কিছু করে

এখানেই অনেকে confused হয়।

Systemd আসলে একটা **পরিবার** যা অনেকগুলো tool মিলে তৈরি:

| Tool | কাজ |
|------|-----|
| **systemd** (মূল) | সব service চালু/বন্ধ করে |
| **journald** | সব log জমা রাখে |
| **networkd** | Network manage করে |
| **timedatectl** | সময় ও timezone ঠিক রাখে |
| **logind** | User login manage করে |
| **systemd-analyze** | Boot speed বিশ্লেষণ করে |


> Systemd হলো একটা **পাড়া/মহল্লা** সেখানে আলাদা আলাদা লোক আলাদা কাজ করে,
> কিন্তু সবাই মিলে পাড়াটা চালায়।


> তাহলে আমরা বলতে পারি **Systemd** হলো Linux এর সেই প্রথম ও সবচেয়ে গুরুত্বপূর্ণ program,
> যে Linux চালু হওয়ার পর বাকি সব কিছু চালু করে, দেখাশোনা করে এবং বন্ধ করে।


## Systemd Units - সব কিছুই একটা "Unit"

Systemd-তে সব কিছুকে **Unit** বলা হয়। প্রতিটি Unit এক একটি configuration file।

### Unit এর প্রকারভেদ:

| Unit Type | Extension | কী করে |
|-----------|-----------|--------|
| **Service** | `.service` | একটা program/daemon চালায় |
| **Socket** | `.socket` | network socket manage করে |
| **Timer** | `.timer` | cron এর মতো schedule করে |
| **Mount** | `.mount` | filesystem mount করে |
| **Target** | `.target` | অনেক unit কে একসাথে group করে |
| **Device** | `.device` | hardware device represent করে |
| **Path** | `.path` | file/directory পরিবর্তন monitor করে |

> DevOps হিসেবে আপনি সবচেয়ে বেশি কাজ করবেন `.service` এবং `.timer` নিয়ে।

## Unit Files কোথায় থাকে?

```bash
# System-provided unit files (distro দেওয়া)
/usr/lib/systemd/system/

# Admin/custom unit files (আপনি নিজে বানাবেন এখানে)
/etc/systemd/system/

# Runtime unit files (temporary)
/run/systemd/system/
```

⚠️ **গুরুত্বপূর্ণ নিয়ম:**
> `/etc/systemd/system/` এর file সবসময় `/usr/lib/systemd/system/` এর file কে **override** করে। তাই custom কিছু বানালে সবসময় `/etc/systemd/system/`-এ রাখুন।

```bash
# সব available unit files দেখুন
systemctl list-unit-files

# শুধু service unit দেখুন
systemctl list-unit-files --type=service
```

**Expected Output (কিছু অংশ):**
```
UNIT FILE                              STATE           VENDOR PRESET
apparmor.service                       enabled         enabled
cron.service                           enabled         enabled
docker.service                         enabled         enabled
nginx.service                          enabled         enabled
ssh.service                            enabled         enabled
```


## একটা Unit File এর ভেতরে কী থাকে?

চলুন একটা real unit file দেখি:

```bash
# nginx এর unit file
cat /usr/lib/systemd/system/nginx.service
# অথবা
systemctl cat nginx.service
```

**Output দেখতে এরকম হবে:**
```ini
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target
ConditionFileIsExecutable=/usr/sbin/nginx

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

### Unit File এর ৩টি Section:

```
[Unit]    → এই service কী, কার পরে চালু হবে
[Service] → কীভাবে চালু/বন্ধ হবে, কোন command দিয়ে
[Install] → কোন target এ এটা চালু থাকবে
```

## Systemd Targets - Group of Units

**Target** মানে হলো একটা **goal state** অনেকগুলো unit কে একসাথে চালানোর একটা group।

পুরনো SysV init-এ **runlevel** ছিল (0, 1, 2, 3, 5, 6)।
Systemd-তে সেগুলোর পরিবর্তে **target** নিয়ে এসেছে:

| SysV Runlevel | Systemd Target | মানে |
|--------------|----------------|------|
| 0 | `poweroff.target` | System বন্ধ |
| 1 | `rescue.target` | Single user / rescue mode |
| 3 | `multi-user.target` | Full multi-user, **no GUI** |
| 5 | `graphical.target` | Full multi-user, **with GUI** |
| 6 | `reboot.target` | Restart |

```bash
# এখন কোন target এ আছেন তা দেখুন
systemctl get-default
```

**Expected Output (server হলে):**
```
multi-user.target
```

**Expected Output (desktop হলে):**
```
graphical.target
```

```bash
# Default target পরিবর্তন করুন (server to graphical)
sudo systemctl set-default graphical.target

# এখনই একটা target এ switch করুন (reboot ছাড়া)
sudo systemctl isolate multi-user.target
```


## Targets কীভাবে একে অপরের উপর নির্ভর করে?

```
poweroff.target
    ↑
rescue.target
    ↑
basic.target
    ↑
multi-user.target  ← Server এর জন্য এটাই সাধারণ
    ↑
graphical.target   ← Desktop এর জন্য এটা
```

প্রতিটি target তার আগের target এর সব কিছু **include** করে।

```bash
# একটা target কোন কোন unit চালায় দেখুন
systemctl list-dependencies multi-user.target
```


## Systemd এর গুরুত্বপূর্ণ Directories একনজরে

```bash
# সব active unit দেখুন
systemctl list-units

# শুধু failed unit দেখুন (DevOps এ অনেক কাজে লাগে!)
systemctl list-units --state=failed

# একটি নির্দিষ্ট unit এর সব তথ্য দেখুন
systemctl show nginx.service
```


## Systemd এর কিছু Useful কমান্ড (Preview)

এগুলো পরের lessons-এ বিস্তারিত শিখবো, কিন্তু এখন একটু দেখে রাখুন:

```bash
# Service চালু করো
sudo systemctl start nginx

# Service বন্ধ করো
sudo systemctl stop nginx

# Service restart করো
sudo systemctl restart nginx

# Service status দেখো
systemctl status nginx

# Boot এ auto-start চালু করো
sudo systemctl enable nginx

# Boot এ auto-start বন্ধ করো
sudo systemctl disable nginx
```


## Systemd Version ও Information দেখুন

Systemd version চেক করা

```bash
systemctl --version
```

**Expected Output:**
```
systemd 255 (255.4-1ubuntu8.14)
+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT...
```

System boot time দেখুন (systemd কতক্ষণে boot করেছে)
```bash
systemd-analyze
```

**Expected Output:**
```
Startup finished in 10.473s (kernel) + 9.892s (userspace) = 20.365s
graphical.target reached after 9.804s in userspace.
```

কোন service boot এ কতক্ষণ লেগেছে দেখুন
```bash
systemd-analyze blame
```


```bash
systemd-analyze blame | head -10
```

**Output:**
```
5.421s NetworkManager-wait-online.service
2.187s docker.service
1.203s dev-sda1.device
 891ms cloud-init.service
 654ms apt-daily-upgrade.service
```

> **DevOps Tip:** `systemd-analyze blame` দিয়ে আপনি দেখতে পারবেন কোন service boot slow করছে
> এটা দিয়ে server boot time optimize করা যায়!


## 📝 Quick Summary

- **Init System** হলো Linux boot হওয়ার পর চালু হওয়া **প্রথম process (PID 1)**
- **Systemd** হলো আধুনিক init system - fast, parallel, powerful
- Systemd-তে সব কিছু **Unit** - service, timer, mount, socket, target ইত্যাদি
- **Unit files** থাকে `/etc/systemd/system/` বা `/usr/lib/systemd/system/` এ
- **Targets** হলো unit-এর group - `multi-user.target` (server) ও `graphical.target` (desktop)
- `systemctl` হলো systemd কে control করার প্রধান কমান্ড
- `systemd-analyze blame` দিয়ে boot performance দেখা যায়


## 🏋️ Practice Tasks

এগুলো নিজে নিজে try করুন:

**Task 1:**
আপনার system এ PID 1-এ কে চলছে verify করুন

```bash
ps -p 1
```

এবং systemd version দেখুন
```bash
systemctl --version
```

**Task 2:**

সব running service দেখুন এবং count করুন কতগুলো এই মুহূর্তে রানিং আছে
```bash
systemctl list-units --type=service --state=running
```

**Task 3:**

Boot analysis করুন

```bash
systemd-analyze
systemd-analyze blame | head -5
```

দেখুন কোন service সবচেয়ে বেশি সময় নিয়েছে


**অসাধারণ!** Systemd এর foundation আপনি বুঝে ফেলেছেন! এটা Linux এর অনেক গুরুত্বপূর্ণ একটা topic। এখন practice tasks গুলো করুন, তারপর Lesson 2-তে জাম্প করুন।

## ⏭️ What's Next?

**Chapter 6 - Lesson 2: Managing Services**

`systemctl start, stop, restart, enable, disable` অর্থাৎ service কীভাবে control করতে হয় সেটা হাতে-কলমে শিখবো। Real nginx/apache সার্ভিস দিয়ে practice করবো। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../../12-Assessment">← Chapter 5 - Assessment</a>
    </td>
    <td align="right">
      <a href="../02-Managing-Services-with-systemctl">Managing Services with systemctl →</a>
    </td>
  </tr>
</table>