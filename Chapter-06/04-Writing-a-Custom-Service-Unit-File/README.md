# Chapter 6 - Lesson 4: Writing a Custom Service Unit File

**Chapter 6 | Lesson 4 of 8**

আগের lesson-এ আমরা শিখেছিলাম কীভাবে service-এর status দেখতে হয় এবং logs check করতে হয়। আজকের lesson-টা অনেক বেশি exciting। কারণ আজ আমরা নিজের হাতে একটা **custom systemd service** তৈরি করবো! এটা একটা super important DevOps skill।


## Service Unit File কী?

মনে করেন আপনি একটা রেস্টুরেন্ট চালান। রেস্টুরেন্টের প্রতিটা কাজের জন্য একটা নিয়মের (rules and regulations book) বই আছে:

- রান্নাঘর কখন খুলবে?
- কোন কাজের পরে কোন কাজ হবে?
- কেউ অসুস্থ হলে কী করবে?
- রেস্টুরেন্ট বন্ধ হলে কী steps নেবে?

**Systemd service unit file** হলো ঠিক এই নিয়মের বই কিন্তু এটা Linux-এর জন্য। এই file-এ আপনি systemd-কে বলে দিতে পারবেন:
- এই service কীভাবে **start** করবে
- কোন service-এর **পরে** start করবে
- Crash হলে **automatically restart** করবে কিনা
- কোন **user** দিয়ে run করবে


## Service Unit File কোথায় থাকে?

```
/etc/systemd/system/          ← আপনার custom services এখানে রাখুন (এটাই সবচেয়ে important)
/lib/systemd/system/          ← Package manager (apt/yum) এর installed services
/run/systemd/system/          ← Runtime-generated services (temporary)
```

**Rule of thumb:** আপনি নিজে যা বানাবেন, সব `/etc/systemd/system/` এ রাখবেন।


## Service Unit File এর Structure

একটা `.service` file তিনটা section-এ ভাগ করা:

```
[Unit]          ← Service সম্পর্কে তথ্য ও dependencies
[Service]       ← কীভাবে service চলবে তা এখানে বলা থাকে। এটাই মূল অংশ
[Install]       ← কোন target-এ enable হবে
```

চলুন প্রতিটা section বিস্তারিত দেখি

## [Unit] Section - পরিচয় ও Dependencies

```ini
[Unit]
Description=My Awesome Web App
Documentation=https://myapp.com/docs
After=network.target postgresql.service
Requires=postgresql.service
Wants=redis.service
```

| Directive | কী করে | উদাহরণ |
|---|---|---|
| `Description=` | Service-এর সংক্ষিপ্ত বর্ণনা | "Nginx Web Server" |
| `Documentation=` | Docs-এর link | man page বা URL |
| `After=` | এই service-গুলোর পরে start হবে | `network.target` |
| `Before=` | এই service-গুলোর আগে start হবে | `shutdown.target` |
| `Requires=` | এই service must চালু থাকতে হবে, না হলে এটাও বন্ধ | `postgresql.service` |
| `Wants=` | এই service চালু থাকলে ভালো, কিন্তু না থাকলেও চলবে | `redis.service` |

### `After=` vs `Requires=` - গুরুত্বপূর্ণ পার্থক্য

```
After=postgresql.service    → PostgreSQL এর পরে start হবে, কিন্তু
                              PostgreSQL বন্ধ থাকলেও এই service চলবে

Requires=postgresql.service → PostgreSQL MUST চালু থাকতে হবে।
                              PostgreSQL বন্ধ হলে এই service-ও বন্ধ হয়ে যাবে
```

## [Service] Section - আসল কাজের জায়গা

এটাই সবচেয়ে important section। এখানে বলা থাকে কীভাবে service চলবে।

```ini
[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
ExecStop=/bin/kill -SIGTERM $MAINPID
ExecReload=/bin/kill -SIGHUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
Environment=APP_ENV=production
EnvironmentFile=/etc/myapp/myapp.conf
```

### Type= এর ধরন

এটা অনেকের কাছে confusing হয়, তাই ভালো করে বুঝতে:

| Type | মানে কী | কখন ব্যবহার |
|---|---|---|
| `simple` | ExecStart-এর process-ই main process (default) | বেশিরভাগ ক্ষেত্রে |
| `forking` | Parent process fork করে child তৈরি করে, parent exit করে | Traditional daemons (Apache) |
| `oneshot` | একবার চলে শেষ হয়ে যায় | Scripts, backup jobs |
| `notify` | Service নিজে systemd-কে জানায় যে ready | nginx, modern apps |
| `idle` | সব অন্য jobs শেষ হওয়ার পরে start হয় | বিরল |

### Exec Directives

```ini
ExecStartPre=  → Main command এর আগে রান করো (যেমন: config check)
ExecStart=     → এটাই main command - service এটা দিয়ে start হয়
ExecStartPost= → Start হওয়ার পরে রান করো
ExecStop=      → Service stop করার command
ExecReload=    → Config reload করার command (systemctl reload)
```

### Restart Policy - DevOps-এ অনেক গুরুত্বপূর্ণ

```ini
Restart=no           → কখনো restart করবে না (default)
Restart=always       → যেকোনো কারণে বন্ধ হলে restart করবে
Restart=on-failure   → শুধু failure/crash হলে restart করবে (সবচেয়ে common)
Restart=on-abnormal  → Signal বা timeout এ বন্ধ হলে restart করবে
RestartSec=5s        → Restart এর আগে 5 সেকেন্ড অপেক্ষা করবে
```

### Security Directives

```ini
User=appuser              → এই user দিয়ে service রান করবে (root নয়!)
Group=appgroup            → এই group দিয়ে রান করবে
WorkingDirectory=/opt/app → এই directory থেকে commands রান করবে
```

### Logging

```ini
StandardOutput=journal   → Normal output systemd journal-এ যাবে
StandardError=journal    → Error output journal-এ যাবে
```

### Environment Variables

```ini
Environment=KEY=value               → সরাসরি variable দাও
Environment="KEY1=val1" "KEY2=val2" → একাধিক variable
EnvironmentFile=/etc/myapp/.env     → File থেকে variables load করো
```


## [Install] Section - Enabling এর নিয়ম

```ini
[Install]
WantedBy=multi-user.target
```

| Directive | মানে |
|---|---|
| `WantedBy=multi-user.target` | Normal boot-এ (non-graphical) enable হবে - সবচেয়ে common |
| `WantedBy=graphical.target` | GUI সহ boot-এ enable হবে |
| `RequiredBy=` | কোন target-এর জন্য required |
| `Alias=` | Service-এর আরেকটা নাম |


## হাতে-কলমে - প্রথম Custom Service বানাই

চলুন একটা real example বানাই। আমরা একটা simple Python web app-এর জন্য service তৈরি করবো।

### Step 1: App Script তৈরি করা

```bash
sudo mkdir -p /opt/mywebapp
sudo nano /opt/mywebapp/app.py
```

```python
#!/usr/bin/env python3
# /opt/mywebapp/app.py

from http.server import HTTPServer, BaseHTTPRequestHandler
from datetime import datetime

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        msg = f"Hello from MyWebApp! Time: {datetime.now()}\n"
        self.wfile.write(msg.encode())
    
    def log_message(self, format, *args):
        print(f"[{datetime.now()}] {format % args}")

if __name__ == '__main__':
    print("MyWebApp starting on port 8080...")
    server = HTTPServer(('0.0.0.0', 8080), Handler)
    server.serve_forever()
```

### Step 2: Dedicated User তৈরি করা (Security best practice)

System user তৈরি করুন (no home directory, no login shell)

```bash
sudo useradd --system --no-create-home --shell /bin/false webappuser
```

App directory-র ownership দিন
```bash
sudo chown -R webappuser:webappuser /opt/mywebapp
```

### Step 3: Service Unit File লিখা

```bash
sudo nano /etc/systemd/system/mywebapp.service
```

```ini
[Unit]
Description=My Python Web Application
Documentation=https://myapp.example.com/docs
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=webappuser
Group=webappuser
WorkingDirectory=/opt/mywebapp

# Main command
ExecStart=/usr/bin/python3 /opt/mywebapp/app.py

# Config check before start (optional)
# ExecStartPre=/usr/bin/python3 -m py_compile /opt/mywebapp/app.py

# Restart policy
Restart=on-failure
RestartSec=5s

# Environment
Environment=APP_ENV=production
Environment=APP_PORT=8080

# Logging
StandardOutput=journal
StandardError=journal

# Resource limits (optional but good practice)
# LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

### Step 4: Systemd-কে নতুন file সম্পর্কে জানানো

```bash
sudo systemctl daemon-reload
```

**এটা কেন দরকার?** যতবার আপনি `.service` file তৈরি বা edit করবেন, systemd-কে জানাতে হবে। না জানালে পুরনো config দিয়েই কাজ করবে।

### Step 5: Service চালু করা ও test করা

Service start করুন
```bash
sudo systemctl start mywebapp
```

Service-এর Status দেখুন
```bash
sudo systemctl status mywebapp
```

**Expected Output:**
```
● mywebapp.service - My Python Web Application
     Loaded: loaded (/etc/systemd/system/mywebapp.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-03-15 10:30:00 UTC; 3s ago
   Main PID: 12345 (python3)
      Tasks: 1 (limit: 4915)
     Memory: 12.3M
        CPU: 0.052s
     CGroup: /system.slice/mywebapp.service
             └─12345 /usr/bin/python3 /opt/mywebapp/app.py
```

App-কে HTTP request পাঠান
```bash
curl http://localhost:8080
```

**Expected Output:**
```
Hello from MyWebApp! Time: 2026-03-15 10:30:05.123456
```

Boot-এ auto-start enable করুন
```bash
sudo systemctl enable mywebapp
```

**Expected Output:**
```
Created symlink /etc/systemd/system/multi-user.target.wants/mywebapp.service →
/etc/systemd/system/mywebapp.service
```

Logs দেখুন
```bash
sudo journalctl -u mywebapp -f
```

## আরেকটা Example - Environment File সহ

Real DevOps-এ credentials বা config সরাসরি service file-এ রাখা হয় না। আলাদা file ব্যবহার করা হয়।

### Environment File তৈরি করা

```bash
sudo mkdir -p /etc/mywebapp
sudo nano /etc/mywebapp/mywebapp.env
```

```bash
# /etc/mywebapp/mywebapp.env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydb
DB_USER=appuser
DB_PASS=supersecret
APP_ENV=production
LOG_LEVEL=info
```

```bash
# এই file যেন শুধু root পড়তে পারে
sudo chmod 600 /etc/mywebapp/mywebapp.env
sudo chown root:root /etc/mywebapp/mywebapp.env
```

### Service File Update করা

```bash
sudo nano /etc/systemd/system/mywebapp.service
```

```ini
[Unit]
Description=My Python Web Application
After=network.target

[Service]
Type=simple
User=webappuser
Group=webappuser
WorkingDirectory=/opt/mywebapp
ExecStart=/usr/bin/python3 /opt/mywebapp/app.py
Restart=on-failure
RestartSec=5s

# Environment file থেকে variables load করো
EnvironmentFile=/etc/mywebapp/mywebapp.env

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
# Reload & restart
sudo systemctl daemon-reload
sudo systemctl restart mywebapp
```

## Oneshot Type - Script হিসেবে Service

কখনো কখনো আপনি চাইতে পারেন যে একটা script একবার চলুক এবং শেষ হোক। যেমন: backup script, cleanup script।

```bash
sudo nano /etc/systemd/system/daily-backup.service
```

```ini
[Unit]
Description=Daily Database Backup
After=postgresql.service

[Service]
Type=oneshot
User=root
ExecStart=/opt/scripts/backup.sh
StandardOutput=journal
StandardError=journal

# Oneshot এর জন্য service completion পরেও "active" দেখাবে
RemainAfterExit=no

[Install]
WantedBy=multi-user.target
```

## Useful Commands - Custom Service এর সাথে কাজ করা

```bash
# Service file-এর syntax check করুন (save করার আগে)
sudo systemd-analyze verify /etc/systemd/system/mywebapp.service

# Service file কোথায় আছে দেখুন
sudo systemctl cat mywebapp

# Service-এর সব properties দেখুন
sudo systemctl show mywebapp

# Service-এর dependency tree দেখুন
sudo systemctl list-dependencies mywebapp

# সব custom service দেখুন
sudo systemctl list-units --type=service --state=running
```

## Real DevOps Scenario - Node.js App Service

DevOps-এ Node.js app খুবই common। চলুন একটা দেখি:

```ini
[Unit]
Description=Node.js Production App
After=network.target

[Service]
Type=simple
User=nodejs
Group=nodejs
WorkingDirectory=/var/www/nodeapp

# Environment
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/etc/nodeapp/.env

# Start with npm
ExecStart=/usr/bin/node /var/www/nodeapp/server.js

# Graceful stop
ExecStop=/bin/kill -SIGTERM $MAINPID

# Aggressive restart policy for production
Restart=always
RestartSec=3s

# Crash limit - 5 restarts in 60 seconds হলে give up
StartLimitBurst=5
StartLimitIntervalSec=60s

# Resource limits
LimitNOFILE=65536

StandardOutput=journal
StandardError=journal
SyslogIdentifier=nodeapp

[Install]
WantedBy=multi-user.target
```

## 📝 Quick Summary

- Service Unit File - systemd-কে বলে কীভাবে service রান করবে
- Custom services সবসময় `/etc/systemd/system/` এ রাখুন
- তিনটা section: `[Unit]` (পরিচয়), `[Service]` (কাজ), `[Install]` (enable নিয়ম)
- File edit করলে সবসময় `systemctl daemon-reload` কমান্ড রান করুন
- Security: কখনো root user দিয়ে app রান করা যাবে না, dedicated user ব্যবহার করা বেস্ট প্রাকটিস
- Restart=on-failure - production-এ সবচেয়ে সাধারণ choice
- EnvironmentFile - secrets ও config আলাদা file-এ রাখুন

## 🏋️ Practice Tasks

**Task 1 - Basic Service:**
একটা simple bash script বানান যেটা `/tmp/myservice.log` file-এ প্রতি 10 সেকেন্ডে timestamp লেখে। তারপর সেটার জন্য একটা systemd service তৈরি করুন, start করুন, enable করুন এবং journalctl দিয়ে logs দেখুন।

```bash
# Script এর hint:
#!/bin/bash
while true; do
    echo "$(date): Service is running" >> /tmp/myservice.log
    sleep 10
done
```

**Task 2 - Edit ও Reload:**
আপনার তৈরি service-এ একটা `Environment=MY_NAME=YourName` line যোগ করুন। তারপর daemon-reload ও restart করুন। journalctl দিয়ে verify করুন যে service ঠিকমতো চলছে।

**Task 3 - Dependency Test:**
আপনার service file-এ `Requires=nonexistent.service` যোগ করুন এবং start করার চেষ্টা করুন। কী error আসে দেখুন। তারপর সেটা সরিয়ে দিয়ে আবার ঠিক করুন।

---

## ⏭️ What's Next

**Chapter 6 - Lesson 5: Systemd Timers**

Cron-এর modern replacement! Cron ছাড়াই কীভাবে systemd দিয়ে scheduled tasks চালাবেন। যেমন daily backup, hourly cleanup, weekly reports সব কিছু।


আপনি এখন নিজে service বানাতে পারেন। এটা একটা real DevOps superpower! যেকোনো app-কে এখন আপনি proper Linux service হিসেবে deploy করতে পারবেন। *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../03-Checking-Service-Status-And-Logs">← Checking Service Status & Logs</a>
    </td>
    <td align="right">
      <a href="../05-Systemd-Timers">Systemd Timers →</a>
    </td>
  </tr>
</table>

