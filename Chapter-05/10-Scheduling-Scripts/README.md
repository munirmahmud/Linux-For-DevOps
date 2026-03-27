# Chapter 5 - Lesson 10: Scheduling Scripts

**Chapter 5 | Lesson 10 of 11**


## 🎯 এই Lesson-এ আমরা শিখবো:

- Cron কী এবং কীভাবে কাজ করে
- Crontab syntax বিস্তারিত
- Anacron কী এবং কখন দরকার
- Systemd Timers modern alternative
- Real DevOps scheduling examples

## Part 1: Cron কী?

**Cron** হলো Linux-এর একটি **task scheduler** মানে আপনি বলে দিতে পারেন "এই কাজটা প্রতিদিন রাত ২টায় করো", Linux নিজেই করবে।

> ধরুন আপনার একটা alarm clock আছে যেটা শুধু বাজেই না নির্দিষ্ট সময়ে নির্দিষ্ট কাজও করে। প্রতি সোমবার সকালে database backup নেয়, প্রতি ঘণ্টায় log file check করো এটাই Cron।

## Part 2: Crontab Cron-এর Configuration File

প্রতিটি user-এর নিজস্ব **crontab** file থাকে।

### Crontab Commands:

```bash
crontab -e      # নিজের crontab edit করা
crontab -l      # নিজের crontab দেখা (list)
crontab -r      # নিজের crontab মুছে দেওয়া (সাবধান!)
crontab -u username -l   # অন্য user-এর crontab দেখা (root হিসেবে)
```

## Part 3: Crontab Syntax বিস্তারিত

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── Day of Week   (0-7, 0 এবং 7 = Sunday)
│ │ │ └──── Month         (1-12)
│ │ └────── Day of Month  (1-31)
│ └──────── Hour          (0-23)
└────────── Minute        (0-59)
```

### Special Characters:

| Character | মানে | Example |
|-----------|------|---------|
| `*` | সব value | `* * * * *` = প্রতি মিনিটে |
| `,` | একাধিক value | `1,15,30 * * * *` = মিনিট 1, 15, 30 তে |
| `-` | Range | `9-17 * * * *` = সকাল ৯টা থেকে বিকাল ৫টা পর্যন্ত প্রতি মিনিটে |
| `/` | Step/interval | `*/5 * * * *` = প্রতি ৫ মিনিটে |

## Part 4: Crontab Examples হাতে-কলমে

```bash
# প্রতি মিনিটে একটা script চালাও
* * * * * /home/user/scripts/check.sh

# প্রতিদিন রাত ২টায় backup নাও
0 2 * * * /home/user/scripts/backup.sh

# প্রতি সোমবার সকাল ৮টায় report পাঠাও
0 8 * * 1 /home/user/scripts/send_report.sh

# প্রতি ৫ মিনিটে server health check করো
*/5 * * * * /home/user/scripts/health_check.sh

# প্রতি মাসের ১ তারিখ রাত ১২টায় log rotate করো
0 0 1 * * /usr/sbin/logrotate /etc/logrotate.conf

# প্রতিদিন সকাল ৬টা থেকে রাত ১০টার মধ্যে প্রতি ঘণ্টায়
0 6-22 * * * /home/user/scripts/notify.sh

# শনি ও রবিবার বাদে প্রতিদিন দুপুর ১২টায়
0 12 * * 1-5 /home/user/scripts/weekday_task.sh
```

## Part 5: Crontab Shortcut Keywords

এই special keywords দিয়ে সহজে schedule করা যায়:

```bash
@reboot    # System boot হলেই একবার চালাও
@yearly    # বছরে একবার (0 0 1 1 *)
@monthly   # মাসে একবার (0 0 1 * *)
@weekly    # সপ্তাহে একবার (0 0 * * 0)
@daily     # প্রতিদিন একবার (0 0 * * *)
@hourly    # প্রতি ঘণ্টায় (0 * * * *)
```

**Example:**
```bash
@reboot /home/user/scripts/startup.sh
@daily /home/user/scripts/daily_backup.sh
```

## Part 6: Cron Job-এর Output handle করা

Default-এ cron job-এর output **email**-এ যায় (বেশিরভাগ server-এ configure থাকে না)। তাই output redirect করা উচিত:

```bash
# Output log file-এ লেখো
0 2 * * * /home/user/scripts/backup.sh >> /var/log/backup.log 2>&1

# Output সম্পূর্ণ বাতিল করো (কোনো log দরকার না হলে)
0 2 * * * /home/user/scripts/backup.sh > /dev/null 2>&1
```

> `2>&1` মানে stderr (error) কেও stdout-এর সাথে একই জায়গায় পাঠাও।

## Part 7: System-wide Cron Files

User crontab ছাড়াও system-level cron directories আছে:

```bash
/etc/crontab          # System crontab (user column আছে এখানে)
/etc/cron.d/          # Package বা admin-এর extra cron files
/etc/cron.hourly/     # প্রতি ঘণ্টায় চালানো scripts রাখো এখানে
/etc/cron.daily/      # প্রতিদিন চালানো scripts
/etc/cron.weekly/     # প্রতি সপ্তাহে
/etc/cron.monthly/    # প্রতি মাসে
```

**/etc/crontab এর format একটু আলাদা user column আছে:**

```bash
# /etc/crontab
# মিনিট ঘণ্টা দিন মাস সপ্তাহ  USER    command
  0      2    *   *    *      root    /usr/bin/backup.sh
  */5    *    *   *    *      www-data /usr/bin/check_web.sh
```

## Part 8: Anacron Cron-এর সমস্যার সমাধান

**Cron-এর সমস্যা কী?**

> মনে করুন আপনার server রাত ২টায় বন্ধ ছিল, কিন্তু cron schedule ছিল রাত ২টায় backup নিতে। **Cron সেই job মিস করবে** পরে আর run করবে না।

**Anacron** এই সমস্যা সমাধান করে। এটা track রাখে শেষ কখন job run হয়েছিল, এবং missed job পরে run করে।

```bash
# Anacron config file
cat /etc/anacrontab
```

**Output:**
```
# /etc/anacrontab
# period   delay   job-identifier   command
  1        5       cron.daily       run-parts /etc/cron.daily
  7        10      cron.weekly      run-parts /etc/cron.weekly
  @monthly 15      cron.monthly     run-parts /etc/cron.monthly
```

| Column | মানে |
|--------|------|
| `period` | কত দিন পরপর (1 = প্রতিদিন, 7 = সপ্তাহে) |
| `delay` | Boot-এর পর কত মিনিট অপেক্ষা করবে |
| `job-identifier` | Job-এর unique নাম |
| `command` | কী চালাবে |

> **Tip:** Laptop বা personal server-এ anacron দরকার। Production server-এ (24/7 চলে) cron-ই যথেষ্ট।

## Part 9: Systemd Timers Modern Alternative

**Systemd Timers** হলো cron-এর modern, powerful replacement।

**Cron vs Systemd Timer:**

| Feature | Cron | Systemd Timer |
|---------|------|---------------|
| Log দেখা | কঠিন | `journalctl` দিয়ে সহজ |
| Dependency | নেই | অন্য service-এর উপর depend করতে পারে |
| Missed job | চলে না | `Persistent=true` দিলে চলে |
| Accuracy | মিনিট পর্যন্ত | সেকেন্ড পর্যন্ত |
| Setup | সহজ | একটু বেশি কাজ |

### Systemd Timer তৈরি করার ধাপ:

**Systemd Timer-এ দুটো file লাগে:**
1. `.service` file কী কাজ করবে
2. `.timer` file কখন করবে

**Step 1: Service file তৈরি করুন**

```bash
sudo nano /etc/systemd/system/mybackup.service
```

```ini
[Unit]
Description=My Daily Backup Service

[Service]
Type=oneshot
ExecStart=/home/user/scripts/backup.sh
```

**Step 2: Timer file তৈরি করুন**

```bash
sudo nano /etc/systemd/system/mybackup.timer
```

```ini
[Unit]
Description=Run My Backup Daily at 2AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Step 3: Enable ও Start করুন**

```bash
sudo systemctl daemon-reload
sudo systemctl enable mybackup.timer
sudo systemctl start mybackup.timer
```

**Step 4: Timer দেখুন**

```bash
systemctl list-timers --all
```

**Output:**
```
NEXT                         LEFT     LAST                         PASSED  UNIT               ACTIVATES
Sun 2025-03-15 02:00:00 UTC  8h left  Sat 2025-03-14 02:00:00 UTC  16h ago mybackup.timer     mybackup.service
```

### OnCalendar Syntax Examples:

```bash
OnCalendar=daily                    # প্রতিদিন midnight
OnCalendar=weekly                   # প্রতি সোমবার midnight
OnCalendar=monthly                  # প্রতি মাসের ১ তারিখ
OnCalendar=*-*-* 02:00:00          # প্রতিদিন রাত ২টায়
OnCalendar=Mon *-*-* 08:00:00      # প্রতি সোমবার সকাল ৮টায়
OnCalendar=*:0/5                    # প্রতি ৫ মিনিটে
OnCalendar=Sat,Sun *-*-* 10:00:00  # শনি ও রবিবার সকাল ১০টায়
```

**Timer syntax validate করতে:**
```bash
systemd-analyze calendar "*-*-* 02:00:00"
```


## Part 10: Real DevOps Cron Examples

```bash
# ১. প্রতিদিন রাত ৩টায় database backup
0 3 * * * /opt/scripts/db_backup.sh >> /var/log/db_backup.log 2>&1

# ২. প্রতি ৫ মিনিটে server uptime check করে log-এ লেখো
*/5 * * * * echo "$(date): $(uptime)" >> /var/log/uptime.log

# ৩. প্রতি রবিবার ভোর ৪টায় system update করো
0 4 * * 0 apt update && apt upgrade -y >> /var/log/auto_update.log 2>&1

# ৪. প্রতি ঘণ্টায় disk usage check, ৮০% এর বেশি হলে alert
0 * * * * /opt/scripts/disk_alert.sh

# ৫. Server boot হলেই monitoring agent চালু করো
@reboot /opt/monitoring/start_agent.sh
```


## Quick Reference Cheat Sheet

```bash
# Crontab manage করা
crontab -e              # Edit
crontab -l              # List
crontab -r              # Remove (সাবধান!)

# Cron service
sudo systemctl status cron    # Ubuntu/Debian
sudo systemctl status crond   # CentOS/RHEL

# Systemd timer
systemctl list-timers         # Active timers দেখা যায়
systemctl list-timers --all   # সব timers দেখা যায়
journalctl -u mybackup.service  # Timer job-এর log দেখা যায়
```

## 📝 Lesson Summary

- **Cron** নির্দিষ্ট সময়ে automatically script/command চালানোর tool
- **Crontab syntax** `মিনিট ঘণ্টা দিন মাস সপ্তাহ command` format
- **Special chars** `*` (সব), `,` (multiple), `-` (range), `/` (step)
- **@shortcuts** `@daily`, `@reboot`, `@weekly` সহজ alternatives
- **Anacron** missed cron job পরে run করার solution
- **Systemd Timers** cron-এর modern replacement, journalctl-এ log দেখা যায়
- **Output redirect** করা উচিত `>> logfile 2>&1`

## 🏋️ Practice Tasks

**Task 1:**
নিজের crontab-এ একটা job add করুন যেটা প্রতি মিনিটে current date/time একটা file-এ append করবে। ৫ মিনিট পরে file-টা দেখুন।

```bash
# Hint:
* * * * * echo "$(date)" >> /tmp/time_log.txt
```

**Task 2:**
একটা simple script লিখুন `disk_check.sh` যেটা `df -h` output একটা log file-এ লেখে। তারপর cron দিয়ে প্রতি ঘণ্টায় একবার চালানোর schedule করুন।

**Task 3:**
`systemctl list-timers --all` রান করুন এবং দেখুন আপনার system-এ কোন কোন timer active আছে।

---

## ⏭️ What's Next?

**Chapter 5 Lesson 11: Real-World DevOps Scripts**

Log rotation script, server health check, auto-backup এই lesson-এ আপনি Chapter 5-এর সব জ্ঞান একসাথে use করে real production-ready scripts লিখবো! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../09-Error-Handling-N-Debugging">← Error Handling &amp; Debugging</a>
    </td>
    <td align="right">
      <a href="../11-Real-World-DevOps-Scripts">Real-World DevOps Scripts →</a>
    </td>
  </tr>
</table>