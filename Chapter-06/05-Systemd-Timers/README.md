# Chapter 6 - Lesson 5: Systemd Timers

**Chapter 6 | Lesson 5 of 8**

## 🎯 এই Lesson-এ আমরা যা শিখবো

- Systemd Timer কী এবং কেন এটি Cron-এর চেয়ে ভালো
- `.timer` unit file কীভাবে কাজ করে
- Monotonic vs Realtime timer
- নিজে Timer তৈরি করে test করা
- DevOps-এ real-world ব্যবহার

## Systemd Timer কী?

একটু কল্পনা করুন আপনি প্রতিদিন রাত ১২টায় আপনার server-এর backup নিতে চান। এই কাজটি automatically করার দুটি উপায় আছে:

| পুরনো উপায় | নতুন উপায় |
|---|---|
| **cron job** | **systemd timer** |
| `/etc/crontab`-এ লেখা | `.timer` unit file |
| Log দেখা কঠিন | `journalctl` দিয়ে সহজে log দেখা যায় |
| Dependency নেই | Service dependency support আছে |
| Boot-এ missed job চলে না | Missed job পরে চালাতে পারে |

> Cron হলো পুরনো **alarm clock** শুধু বাজে। Systemd Timer হলো **স্মার্টফোনের alarm** মিস হলে মনে করিয়ে দেয়, log রাখে, snooze করা যায়!


## Systemd Timer-এর Structure

Systemd Timer কাজ করে দুটি file নিয়ে:

```
mybackup.timer   ←→   mybackup.service
(কখন চলবে?)          (কী চলবে?)
```

> **গুরুত্বপূর্ণ নিয়ম:** Timer এবং Service-এর নাম একই হতে হবে (extension বাদে)।

## Timer File কোথায় রাখবো?

```
/etc/systemd/system/       ← আপনার custom timer এখানে
/lib/systemd/system/       ← system-installed timer (touch করবেন না)
```

## Timer-এর দুই ধরন

### ১. Monotonic Timer (সময় গণনা করে)

Boot হওয়ার পর বা কোনো event-এর পর **কত সময় পরে** চলবে তা নির্ধারণ করে।

```ini
[Timer]
OnBootSec=10min        # boot-এর ১০ মিনিট পরে চলবে
OnUnitActiveSec=1h     # শেষবার চলার ১ ঘণ্টা পরে আবার চলবে
```

**সময়ের format:**

| লেখার নিয়ম | মানে |
|---|---|
| `30s` | ৩০ সেকেন্ড |
| `5min` | ৫ মিনিট |
| `2h` | ২ ঘণ্টা |
| `1d` | ১ দিন |
| `1w` | ১ সপ্তাহ |


### ২. Realtime Timer (ঘড়ির সময় ধরে)

নির্দিষ্ট তারিখ বা সময়ে চলবে, ঠিক cron-এর মতো।

```ini
[Timer]
OnCalendar=daily              # প্রতিদিন মধ্যরাতে
OnCalendar=weekly             # প্রতি সোমবার মধ্যরাতে
OnCalendar=Mon *-*-* 09:00:00 # প্রতি সোমবার সকাল ৯টায়
OnCalendar=*-*-* 02:30:00     # প্রতিদিন রাত ২:৩০-এ
```

**Calendar format বোঝার নিয়ম:**
```
DayOfWeek  Year-Month-Day  Hour:Minute:Second
```


## হাতে-কলমে: নিজে Timer তৈরি করা

### Step 1: Service file তৈরি করুন

এই service টি একটি simple কাজ করবে, একটি file-এ timestamp লিখবে।

```bash
sudo nano /etc/systemd/system/hellotimer.service
```

ফাইলে এই content লিখুন:

```ini
[Unit]
Description=Hello Timer Test Service
# এই service কী করবে তার বিবরণ

[Service]
Type=oneshot
# oneshot মানে একবার চলবে, তারপর বন্ধ হয়ে যাবে

ExecStart=/bin/bash -c 'echo "Timer is running: $(date)" >> /tmp/timer_test.log'
# প্রতিবার চলার সময় /tmp/timer_test.log ফাইলে সময় লিখবে
```

> **`Type=oneshot` কেন?** Timer দিয়ে যে service চালাই সেটি সাধারণত একটি কাজ করে শেষ হয় কিন্তু daemon হিসেবে চলে না। তাই `oneshot` use করি।


### Step 2: Timer file তৈরি করা

```bash
sudo nano /etc/systemd/system/hellotimer.timer
```

```ini
[Unit]
Description=Hello Timer - প্রতি ১ মিনিটে চলবে

[Timer]
OnBootSec=30s
# Boot-এর ৩০ সেকেন্ড পরে প্রথমবার চলবে

OnUnitActiveSec=1min
# তারপর প্রতি ১ মিনিটে চলবে

Unit=hellotimer.service
# কোন service চালাবে

[Install]
WantedBy=timers.target
# timers.target মানে - systemd যখন timer manage করে তখন এটি include করুন
```

### Step 3: Systemd-কে জানাতে হবে যে নতুন file এসেছে

```bash
sudo systemctl daemon-reload
```

**এটি কেন করি?** নতুন unit file তৈরির পরে systemd-কে বলতে হয় যে নতুন সার্ভিস যুক্ত হয়েছে অতএব সেই নতুন file পড়ো!


### Step 4: Timer চালু করা

```bash
sudo systemctl start hellotimer.timer
```

Boot-এ automatically চালু করতে চাইলে:

```bash
sudo systemctl enable hellotimer.timer
```

**Expected output:**
```
Created symlink /etc/systemd/system/timers.target.wants/hellotimer.timer
→ /etc/systemd/system/hellotimer.timer.
```

### Step 5: Timer-এর status দেখুন

```bash
systemctl status hellotimer.timer
```

**Expected output:**
```
● hellotimer.timer - Hello Timer - প্রতি ১ মিনিটে চলবে
     Loaded: loaded (/etc/systemd/system/hellotimer.timer; enabled)
     Active: active (waiting) since Sun 2026-03-15 10:00:00 +06
    Trigger: Sun 2026-03-15 10:01:00 +06
             (পরের বার কখন চলবে তা এখানে দেখাবে)
```

### Step 6: সব Timer একসাথে দেখুন

```bash
systemctl list-timers
```

**Expected output:**
```
NEXT                        LEFT     LAST                        PASSED  UNIT
Sun 2026-03-15 10:01:00     45s      Sun 2026-03-15 10:00:00     15s     hellotimer.timer
...
```

| Column | মানে |
|---|---|
| **NEXT** | পরের বার কখন চলবে |
| **LEFT** | কত সময় বাকি |
| **LAST** | শেষবার কখন চলেছিল |
| **PASSED** | শেষবার চলার কতক্ষণ হলো |
| **UNIT** | Timer-এর নাম |

### Step 7: Log দেখা

```bash
# Timer-এর log দেখুন
journalctl -u hellotimer.timer

# Service-এর log দেখুন
journalctl -u hellotimer.service

# দুটো একসাথে দেখুন
journalctl -u hellotimer.timer -u hellotimer.service

# Live দেখুন (follow mode)
journalctl -u hellotimer.service -f
```


### Step 8: File-এ output দেখা

```bash
cat /tmp/timer_test.log
```

**Expected output (২-৩ মিনিট পরে):**
```
Timer is running: Sun Mar 15 10:00:30 +06 2026
Timer is running: Sun Mar 15 10:01:30 +06 2026
Timer is running: Sun Mar 15 10:02:30 +06 2026
```

তারমানে Timer কাজ করছে!

## OnCalendar-এর কিছু গুরুত্বপূর্ণ উদাহরণ

```ini
# প্রতিদিন রাত ১২টায়
OnCalendar=daily

# প্রতিদিন সকাল ৬টায়
OnCalendar=*-*-* 06:00:00

# প্রতি সোমবার ও বুধবার রাত ১০টায়
OnCalendar=Mon,Wed *-*-* 22:00:00

# প্রতি মাসের ১ তারিখে
OnCalendar=*-*-01 00:00:00

# প্রতি ঘণ্টায়
OnCalendar=hourly

# প্রতি ৫ মিনিটে
OnCalendar=*:0/5
```

### Calendar expression verify করা:

```bash
systemd-analyze calendar "Mon,Wed *-*-* 22:00:00"
```

**Expected output:**
```
  Original form: Mon,Wed *-*-* 22:00:00
Normalized form: Mon,Wed *-*-* 22:00:00
    Next elapse: Mon 2026-03-16 22:00:00 +06
       (in UTC): Mon 2026-03-16 16:00:00 UTC
       From now: 11h left
```

> Timer লেখার পরে সবসময় `systemd-analyze calendar` দিয়ে check করুন। এটি বলে দিবে সঠিক কিনা এবং পরের বার কখন চলবে।


## AccuracySec - Timer-এর সময় accuracy

```ini
[Timer]
OnCalendar=daily
AccuracySec=1s
# ডিফল্ট হলো 1 মিনিট - এর মানে timer ঠিক সময়ে না চলে
# ১ মিনিটের মধ্যে যেকোনো সময়ে চলতে পারে
# Accurate timing চাইলে AccuracySec=1s লেখো
```


## Persistent Timer - Missed Job ধরে রাখা

```ini
[Timer]
OnCalendar=daily
Persistent=true
# যদি server বন্ধ থাকে এবং timer miss হয়
# server চালু হওয়ার পরেই missed job চালাবে
```

> **Real DevOps scenario:** রাত ২টায় backup নেওয়ার কথা ছিল। কিন্তু server বন্ধ ছিল। `Persistent=true` থাকলে server চালু হওয়ার পরেই backup নেবে।

## Cron vs Systemd Timer - পাশাপাশি তুলনা

| বিষয় | Cron | Systemd Timer |
|---|---|---|
| **Config file** | `/etc/crontab` | `.timer` + `.service` |
| **Log দেখা** | `/var/log/cron` বা `mail` | `journalctl` (সহজ!) |
| **Missed job** | চলে না | `Persistent=true` দিয়ে চলে |
| **Dependency** | নেই | After=, Requires= ব্যবহার করা যায় |
| **Boot delay** | নেই | `OnBootSec` দিয়ে delay দেওয়া যায় |
| **Random delay** | নেই | `RandomizedDelaySec` আছে |
| **Status check** | কঠিন | `systemctl status` সহজ |
| **Per-user timer** | `crontab -e` | User-level timer সম্ভব |


## DevOps-এ Real-World ব্যবহার

### ১. Daily Backup Timer

```ini
# /etc/systemd/system/db-backup.timer
[Unit]
Description=Database Daily Backup

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
AccuracySec=1s

[Install]
WantedBy=timers.target
```

### ২. Health Check Timer (প্রতি ৫ মিনিটে)

```ini
[Timer]
OnBootSec=2min
OnUnitActiveSec=5min
```

### ৩. Log Cleanup Timer (প্রতি সপ্তাহে)

```ini
[Timer]
OnCalendar=weekly
Persistent=true
```

## Timer বন্ধ করা

```bash
# Timer সাময়িক বন্ধ
sudo systemctl stop hellotimer.timer

# Boot-এ আর চালু না হোক
sudo systemctl disable hellotimer.timer

# পুরোপুরি মুছে ফেলো
sudo rm /etc/systemd/system/hellotimer.timer
sudo rm /etc/systemd/system/hellotimer.service
sudo systemctl daemon-reload
```

## 📝 Quick Summary

- Systemd Timer = `.timer` file + `.service` file - দুটো একসাথে কাজ করে
- **Monotonic Timer:** Boot বা event-এর পরে সময় গণনা করে (`OnBootSec`, `OnUnitActiveSec`)
- **Realtime Timer:** ঘড়ির নির্দিষ্ট সময়ে চলে (`OnCalendar`)
- `systemctl list-timers` দিয়ে সব timer দেখা যায়
- `journalctl -u <name>` দিয়ে log দেখা যায়
- `Persistent=true` দিলে missed job পরে চলে
- `systemd-analyze calendar` দিয়ে expression verify করা যায়
- Cron-এর চেয়ে বেশি powerful, flexible এবং debuggable


## 🏋️ Practice Tasks

**Task 1:**
`hellotimer` নামে একটি timer তৈরি করুন যেটি প্রতি ২ মিনিটে `/tmp/practice.log` ফাইলে current date ও time লিখবে। Timer চালু করুন এবং `journalctl` দিয়ে log দেখুন।

**Task 2:**
`systemd-analyze calendar` command দিয়ে এই expressions গুলো verify করুন:
- `daily`
- `Mon *-*-* 09:00:00`
- `*-*-01 00:00:00`

প্রতিটির জন্য দেখুন, পরের বার কখন চলবে।

**Task 3:**
`systemctl list-timers --all` রান করে দেখুন যে আপনার system-এ কতগুলো timer আছে। কোনটি সবচেয়ে বেশি কাজের মনে হচ্ছে?

---

## ⏭️ What's Next?

**Chapter 6 - Lesson 6: Systemd Targets**

পরের lesson-এ আমরা শিখবো Systemd Targets যা পুরনো Linux-এর runlevel-এর আধুনিক version। `multi-user.target`, `graphical.target`, `rescue.target` কী এবং কীভাবে system boot mode পরিবর্তন করা যায়। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../04-Writing-a-Custom-Service-Unit-File">← Writing a Custom Service Unit File</a>
    </td>
    <td align="right">
      <a href="../06-Systemd-Targets">Systemd Targets →</a>
    </td>
  </tr>
</table>