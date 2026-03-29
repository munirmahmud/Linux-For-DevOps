# Chapter 6 - Lesson 3: Checking Service Status & Logs

**Chapter 6 | Lesson 3 of 8**


## 🎯 এই Lesson-এ আমার যা শিখবো

- `systemctl status` দিয়ে service-এর অবস্থা দেখা
- Service কি চলছে, কখন চালু হয়েছে, কতটুকু memory নিচ্ছে, সব জানা
- `journalctl` দিয়ে systemd-এর log দেখা
- Real DevOps-এ কীভাবে এই tools দিয়ে সমস্যা খুঁজে বের করা হয়

আগে একটু Background বোঝার চেস্টা করি। মনে করেন আপনার একটা দোকান আছে। আপনি এখন জানতে চান:
- দোকান কি খোলা আছে?
- কখন খুলেছিল?
- আজকে কী কী ঘটেছে? (log)
- কোনো সমস্যা হয়েছে কি?

Linux-এ service হলো সেই দোকানের মতো।
- `systemctl status` → দোকানের বর্তমান অবস্থা দেখা
- `journalctl` → দোকানের দৈনন্দিন diary (log) পড়া


## Part 1: `systemctl status`

### এটা কী করে?
কোনো একটা service এর সম্পূর্ণ তথ্য দেখায় - সে চলছে কিনা, কখন চালু হয়েছে, সর্বশেষ কী log এসেছে, তার PID কত ইত্যাদি।

**Syntax:**
```bash
systemctl status <service-name>
```

### উদাহরণ:
```bash
systemctl status ssh
```

### Expected Output:
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-01-06 10:30:45 UTC; 2h 15min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1234 (sshd)
      Tasks: 1 (limit: 4915)
     Memory: 2.1M
        CPU: 345ms
     CGroup: /system.slice/ssh.service
             └─1234 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Jan 06 10:30:45 myserver systemd[1]: Starting OpenBSD Secure Shell server...
Jan 06 10:30:45 myserver sshd[1234]: Server listening on 0.0.0.0 port 22.
Jan 06 10:30:45 myserver systemd[1]: Started OpenBSD Secure Shell server.
```

### Output-এর প্রতিটি লাইনের মানে কী?

| Field | মানে |
|---|---|
| `● ssh.service` | Service-এর নাম। **●** সবুজ হলে চলছে, লাল হলে বন্ধ, হলুদ হলে সমস্যা |
| `Loaded` | Service-এর unit file কোথায় আছে এবং boot-এ enable আছে কিনা |
| `Active: active (running)` | Service এখন চলছে |
| `since ... 2h 15min ago` | কখন থেকে চলছে |
| `Main PID: 1234` | Service-এর প্রধান Process ID |
| `Memory: 2.1M` | কতটুকু memory ব্যবহার করছে |
| `CPU: 345ms` | মোট কতটুকু CPU সময় নিয়েছে |
| `CGroup` | কোন control group-এ আছে |
| নিচের লাইনগুলো | সর্বশেষ কয়েকটি log entry |


### Active Status-এর বিভিন্ন রকম মানে:

```bash
Active: active (running)    # সার্ভিস চলছে
Active: inactive (dead)     # সার্ভিস বন্ধ
Active: failed              # সার্ভিস crash করেছে বা error-এ পড়েছে
Active: activating          # সার্ভিস এখন চালু হচ্ছে
Active: deactivating        # সার্ভিস এখন বন্ধ হচ্ছে
```

### আরো কিছু উদাহরণ:

```bash
# nginx-এর status দেখুন
systemctl status nginx

# একটি failed service-এর status
systemctl status myapp.service

# সংক্ষিপ্ত output চাইলে (--no-pager)
systemctl status nginx --no-pager

# শুধু active/inactive জানতে চাইলে
systemctl is-active nginx

# শুধু enabled/disabled জানতে চাইলে
systemctl is-enabled nginx
```

### `is-active` এবং `is-enabled` Output:
```bash
$ systemctl is-active nginx
active          # অথবা: inactive, failed

$ systemctl is-enabled nginx
enabled         # অথবা: disabled, masked
```

### DevOps-এ কখন ব্যবহার করবো?

- কোনো service deploy করার পরে confirm করতে যে সে চলছে
- Production-এ সমস্যা হলে প্রথমেই `systemctl status` রান করা হয়
- Monitoring script-এ `systemctl is-active` দিয়ে check করা

## Part 2: `journalctl` - Systemd-এর Log System

### এটা কী?

`journalctl` হলো systemd-এর **central log viewer**। Systemd সব service-এর log একটা জায়গায় রাখে যেটাকে বলে **journal**। `journalctl` দিয়ে সেই journal পড়া যায়।

> যেমন একটা building-এ সব CCTV footage একটাই control room-এ দেখা যায়, তেমনি সব service-এর log `journalctl` দিয়ে এক জায়গা থেকে দেখা যায়।

### Basic Usage:

```bash
journalctl
```
এটা সব log দেখাবে। system চালু হওয়া থেকে এখন পর্যন্ত সব log। অনেক বেশি হবে। তাই filter করে দেখা লাগবে।


### সবচেয়ে গুরুত্বপূর্ণ journalctl Options:

#### ১. নির্দিষ্ট Service-এর Log দেখা

```bash
journalctl -u nginx
```
```
-- Logs begin at Mon 2025-01-06 08:00:00 UTC --
Jan 06 10:30:00 myserver systemd[1]: Starting A high performance web server...
Jan 06 10:30:01 myserver nginx[1456]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 06 10:30:01 myserver nginx[1456]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 06 10:30:01 myserver systemd[1]: Started A high performance web server.
```

`-u` মানে **unit**। কোন service-এর log দেখতে চাই সেটার নাম।


#### ২. Real-time Log দেখা (Live Follow)

```bash
journalctl -u nginx -f
```

> `-f` মানে **follow** যেমন `tail -f` করে। নতুন log আসলেই সাথে সাথে দেখাবে। DevOps-এ এটা সবচেয়ে বেশি ব্যবহার হয়।

`Ctrl+C` চাপলে বন্ধ হবে।

#### ৩. শেষের কয়েকটি Line দেখা

```bash
# শেষের 50 লাইন log দেখুন
journalctl -u nginx -n 50

# শেষের 100 লাইন log দেখুন
journalctl -n 100
```


#### ৪. সময় অনুযায়ী Filter করা

```bash
# আজকের log
journalctl --since today

# নির্দিষ্ট সময় থেকে
journalctl --since "2025-01-06 10:00:00"

# সময়ের range
journalctl --since "2025-01-06 10:00:00" --until "2025-01-06 11:00:00"

# গত ১ ঘণ্টার log
journalctl --since "1 hour ago"

# গত ৩০ মিনিটের log
journalctl --since "30 min ago"
```

#### ৫. শুধু Error বা Warning দেখা (Priority Filter)

```bash
# শুধু error দেখুন
journalctl -p err

# error বা তার চেয়ে বড় সমস্যা
journalctl -p 3

# নির্দিষ্ট service-এর error
journalctl -u nginx -p err
```

### Priority Levels:
| Number | নাম | মানে |
|---|---|---|
| 0 | `emerg` | System crash হয়ে যাচ্ছে |
| 1 | `alert` | অবিলম্বে ব্যবস্থা নাও |
| 2 | `crit` | Critical error |
| 3 | `err` | সাধারণ error |
| 4 | `warning` | সতর্কবার্তা |
| 5 | `notice` | গুরুত্বপূর্ণ সাধারণ তথ্য |
| 6 | `info` | তথ্যমূলক message |
| 7 | `debug` | Debug তথ্য |


#### ৬. Boot-এর Log দেখা

```bash
# এই বারের boot-এর log
journalctl -b

# আগের boot-এর log
journalctl -b -1

# তার আগের boot-এর log
journalctl -b -2

# কতবার boot হয়েছে তার list
journalctl --list-boots
```

```
# --list-boots output:
-2 abc123... Mon 2025-01-04 09:00:00 UTC-Mon 2025-01-04 18:00:00 UTC
-1 def456... Tue 2025-01-05 09:00:00 UTC-Tue 2025-01-05 22:00:00 UTC
 0 ghi789... Wed 2025-01-06 08:00:00 UTC-Wed 2025-01-06 12:00:00 UTC
```


#### ৭. Kernel Log দেখুন

```bash
journalctl -k
```
> শুধু kernel-এর log দেখাবে। Hardware সমস্যা বা driver issue খুঁজতে কাজে লাগে।


#### ৮. নির্দিষ্ট Process-এর Log

```bash
# PID দিয়ে
journalctl _PID=1234

# নির্দিষ্ট executable-এর log
journalctl _EXE=/usr/sbin/sshd
```


#### ৯. Output Format পরিবর্তন করা

```bash
# JSON format-এ দেখুন (scripting-এ কাজে লাগে)
journalctl -u nginx -o json

# সুন্দর JSON format
journalctl -u nginx -o json-pretty

# শুধু message দেখুন
journalctl -u nginx -o cat

# Short format (default)
journalctl -u nginx -o short
```

#### ১০. Disk Space কতটুকু নিচ্ছে

```bash
journalctl --disk-usage
```
```
Archived and active journals take up 256.0M in the file system.
```

#### ১১. পুরনো Log মুছে ফেলো (Cleanup)

```bash
# ৩০ দিনের বেশি পুরনো log মুছে ফেলুন
journalctl --vacuum-time=30d

# ৫০০MB এর বেশি হলে মুছে ফেলুন
journalctl --vacuum-size=500M
```

## Real DevOps Scenario: Service Crash হলে কী করবো?

যদি কখনো আপনার `nginx` সার্ভার হঠাৎ বন্ধ হয়ে যায় তখন আপনি কীভাবে diagnose করবেন?

### Step 1: Status দেখুন
```bash
systemctl status nginx
```
```
● nginx.service - A high performance web server
   Active: failed (Result: exit-code) since Mon 2025-01-06 14:30:00 UTC
  Process: 5678 ExecStart=/usr/sbin/nginx (code=exited, status=1/FAILURE)
```
দেখলে `failed` আছে। এখন কারণ খুঁজতে হবে।

### Step 2: Log দেখুন
```bash
journalctl -u nginx -n 50 --no-pager
```
```
Jan 06 14:30:00 myserver nginx[5678]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Jan 06 14:30:00 myserver nginx[5678]: nginx: [emerg] still could not bind()
Jan 06 14:30:00 myserver systemd[1]: nginx.service: Control process exited with error code
Jan 06 14:30:00 myserver systemd[1]: Failed to start A high performance web server.
```

### Step 3: সমস্যা ধরা পড়লো!
Port 80 অন্য কেউ ব্যবহার করছে। এখন:
```bash
# কে port 80 ব্যবহার করছে?
ss -tlnp | grep :80

# অথবা
lsof -i :80
```

এইভাবে `systemctl status` + `journalctl` মিলিয়ে যেকোনো সমস্যা বের করা যায়।


## systemctl status vs journalctl - পার্থক্য

| | `systemctl status` | `journalctl` |
|---|---|---|
| কী দেখায় | Service-এর current অবস্থা + শেষ কয়েকটি log | পুরো log history |
| কতটুকু log | মাত্র শেষ কয়েকটি line | চাইলে সব দেখা যায় |
| Filter করা যায়? | না | হ্যাঁ - সময়, priority, service অনুযায়ী |
| প্রথমে কোনটা? | হ্যাঁ, প্রথমে এটা দেখুন | সমস্যা গভীরে খুঁজতে এরপর এটা |


## 📝 Quick Summary

- `systemctl status <service>` → Service-এর সম্পূর্ণ অবস্থা দেখায়
- `systemctl is-active` / `is-enabled` → Quick check-এর জন্য
- `journalctl -u <service>` → নির্দিষ্ট service-এর log
- `journalctl -u <service> -f` → Real-time live log
- `journalctl -u <service> -n 50` → শেষ ৫০ লাইন
- `journalctl --since "1 hour ago"` → সময় অনুযায়ী filter
- `journalctl -p err` → শুধু error দেখুন
- `journalctl -b` → এই boot-এর সব log
- `journalctl -k` → Kernel log


## 🏋️ Practice Tasks

এখন নিজে নিজে চেষ্টা করুন:

**Task 1:**
```bash
# SSH service-এর status দেখুন এবং নিচের প্রশ্নের উত্তর দিন:
# - সে কি চলছে?
# - কখন থেকে চলছে?
# - Main PID কত?
systemctl status ssh
```

**Task 2:**
```bash
# SSH service-এর শেষ ২০টি log দেখুন
journalctl -u ssh -n 20

# এরপর real-time log follow করুন এবং অন্য terminal থেকে SSH login করুন
journalctl -u ssh -f
```

**Task 3:**
```bash
# গত ১ ঘণ্টার সব error দেখুন
journalctl -p err --since "1 hour ago"

# Journal কতটুকু disk space নিচ্ছে জানুন
journalctl --disk-usage
```

---

## ⏭️ What's Next?

**Chapter 6 - Lesson 4: Writing a Custom Service Unit File (.service)**

পরের lesson-এ আমরা নিজেরাই একটা systemd service তৈরি করবো! নিজের application-কে systemd service হিসেবে register করতে শিখবো, যেন সে automatically boot-এ চালু হয় এবং crash করলে restart নেয়। এটা DevOps-এ অত্যন্ত গুরুত্বপূর্ণ একটি skill! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../02-Managing-Services-with-systemctl">← Managing Services with systemctl</a>
    </td>
    <td align="right">
      <a href="../04-Writing-a-Custom-Service-Unit-File">Writing a Custom Service Unit File →</a>
    </td>
  </tr>
</table>