# Chapter 4 - Assessment


আমি প্রতিটা task-এর পরে একটা **"Hint"** দেবো (যাতে আপনি নিজে চেষ্টা করতে পারেন), এবং সব task শেষে **detailed answer + explanation** দিবো। এতে আপনি:

- নিজে চেষ্টা করার সুযোগ পাবেন
- Hint দেখে আটকে গেলে এগোতে পারবেন
- শেষে answer মিলিয়ে নিতে পারবেন

এটাই সবচেয়ে effective learning approach। চলুন শুরু করি!


- Chapter 4: Networking in Linux
- Production-Grade DevOps Assessment
- মোট Tasks: 15টি | Real-world Scenarios


> ⚠️ **নিয়ম:** প্রতিটা task নিজে করার চেষ্টা করুন। Hint দেখুন শুধু আটকে গেলে। সব শেষে answers দেওয়া আছে। কোনো issue থাকলে একটা pull request করার অনুরোধ করছি


## TASK 1 - Server-এর Network Identity বের করুন
**Scenario:** আপনি একটা নতুন production server-এ SSH করেছেন। আপনার team lead বললো "এই server-এর সব network information একটা file-এ save করুন।"

**আপনাকে যা করতে হবে:**
- Server-এর IP address (সব interface-এর)
- Hostname
- Default gateway
- DNS server
- সব এক সাথে একটা file `network_info.txt`-এ save করুন

**Hint:** `ip addr`, `ip route`, `cat /etc/resolv.conf`, এবং redirection operator `>` ও `>>` ব্যবহার করতে পারেন।


## TASK 2 - Connectivity Troubleshooting Chain
**Scenario:** একজন developer বললো "আমাদের app, database server `192.168.1.50`-এ connect হচ্ছে না।" আপনাকে systematically troubleshoot করতে হবে।

**আপনাকে যা করতে হবে:**
- প্রথমে basic ping test করুন
- তারপর route trace করুন, packet কোথায় আটকাচ্ছে দেখুন
- DNS resolution check করুন `google.com`-এর জন্য
- শেষে একটা report দিন, সমস্যা কোথায়

**Hint:** `ping -c 4`, `traceroute`, `mtr`, `dig` এই order-এ ব্যবহার করতে পারেন।


## TASK 3 - Port Scanning & Service Discovery
**Scenario:** Security audit-এর জন্য আপনাকে জানতে হবে - আপনার server-এ কোন কোন port open আছে এবং কোন service সেগুলো use করছে।

**আপনাকে যা করতে হবে:**
- সব listening port দেখুন
- প্রতিটা port-এর সাথে কোন process connected সেটা দেখুন
- Port `22`, `80`, `443` specifically check করুন
- Output একটা file-এ save করুন `open_ports_audit.txt`

**Hint:** `ss -tulnp`, `netstat -tulnp`, `lsof -i :22` ব্যবহার করুন


## TASK 4 - SSH Key-Based Authentication Setup
**Scenario:** আপনার company-র policy হলো কোনো server-এ password দিয়ে SSH করা যাবে না, শুধু SSH key দিয়ে করতে হবে। আপনাকে এটা setup করতে হবে।

**আপনাকে যা করতে হবে:**
- নতুন SSH key pair generate করুন (RSA 4096-bit)
- Public key remote server-এ copy করুন
- Test করুন key দিয়ে login হচ্ছে কিনা
- একটা SSH config file তৈরি করুন যাতে `ssh myserver` লিখলেই connect হয়

**Hint:** `ssh-keygen -t rsa -b 4096`, `ssh-copy-id`, `~/.ssh/config` file তৈরি করুন


## TASK 5 - File Transfer করুন Multiple Methods-এ
**Scenario:** আপনাকে একটা 500MB deployment package production server-এ পাঠাতে হবে এবং remote server থেকে logs নিয়ে আসতে হবে।

**আপনাকে যা করতে হবে:**
- Local থেকে remote-এ একটা file পাঠান `scp` দিয়ে
- Remote থেকে local-এ একটা directory নিয়ে আসুন `rsync` দিয়ে
- `rsync` দিয়ে এমনভাবে sync করুন যাতে শুধু changed files transfer হয়
- একটা remote URL থেকে file download করুন `wget` এবং `curl` দিয়ে

**Hint:** `scp -r`, `rsync -avz --progress`, `wget -O`, `curl -O` ব্যবহার করুন


## TASK 6 - DNS Deep Investigation
**Scenario:** আপনাদের application হঠাৎ `api.github.com`-এ connect করতে পারছে না। আপনাকে DNS layer থেকে debug করতে হবে।

**আপনাকে যা করতে হবে:**
- `api.github.com`-এর A record, MX record, এবং NS record বের করুন
- কোন DNS server ব্যবহার হচ্ছে সেটা দেখুন
- DNS query-র response time দেখুন
- `/etc/resolv.conf` এবং `/etc/hosts` check করুন

**Hint:** `dig api.github.com A`, `dig api.github.com MX`, `nslookup`, `host` command ব্যবহার করুন


## TASK 7 - Firewall Rules Management
**Scenario:** একটা নতুন web application deploy হয়েছে। Security team বললো শুধু port 80, 443, এবং 22 open রাখুন। বাকি সব block করুন।

**আপনাকে যা করতে হবে:**
- Current firewall rules দেখুন
- Port 80 এবং 443 allow করুন
- Port 22 শুধু specific IP থেকে allow করুন (যেমন: `203.0.113.10`)
- বাকি সব incoming traffic deny করুন
- Rules permanent করুন

**Hint:** `ufw` অথবা `firewalld` ব্যবহার করুন। `ufw allow`, `ufw deny`, `ufw allow from`


## TASK 8 - Network Traffic Analysis
**Scenario:** Server-এ suspicious activity দেখা যাচ্ছে, অনেক বেশি network traffic। আপনাকে দেখতে হবে কোথা থেকে কী traffic আসছে।

**আপনাকে যা করতে হবে:**
- `eth0` interface-এর সব traffic capture করুন ৩০ সেকেন্ডের জন্য
- শুধু port 80-এর HTTP traffic দেখুন
- একটা specific IP (`8.8.8.8`) থেকে আসা traffic filter করুন
- Captured data একটা file-এ save করুন analysis-এর জন্য

**Hint:** `tcpdump -i eth0`, `tcpdump -i eth0 port 80`, `tcpdump host 8.8.8.8`, `-w` flag দিয়ে save করুন


## TASK 9 - SSH Tunneling & Port Forwarding
**Scenario:** Production database (port 5432) publicly accessible না, শুধু internal network-এ আছে। আপনাকে local machine থেকে সেই database-এ connect করতে হবে SSH tunnel ব্যবহার করে।

**আপনাকে যা করতে হবে:**
- Local port 5433 থেকে remote server-এর port 5432-এ tunnel তৈরি করুন
- Tunnel টা background-এ run করুন
- Tunnel কাজ করছে কিনা verify করুন
- একটা reverse tunnel scenario explain করুন

**Hint:** `ssh -L 5433:localhost:5432 user@server`, `ssh -N -f` flags ব্যবহার করুন


## TASK 10 - Bulk File Download & Verification
**Scenario:** আপনাকে একটা software package download করতে হবে, তার integrity verify করতে হবে, এবং proper directory-তে রাখতে হবে।

**আপনাকে যা করতে হবে:**
- `wget` দিয়ে একটা file download করুন এবং progress দেখুন
- `curl` দিয়ে HTTP response headers দেখুন কোনো URL-এর
- Download করা file-এর checksum verify করুন
- Download resume করার ক্ষমতা test করুন

**Hint:** `wget --progress=bar`, `curl -I`, `md5sum` বা `sha256sum`, `wget -c` (continue) ব্যবহার করুন


## TASK 11 - `/etc/hosts` দিয়ে Local DNS Override
**Scenario:** Development environment-এ আপনাকে `prod.myapp.com` কে local server-এ point করতে হবে, actual DNS change না করেই।

**আপনাকে যা করতে হবে:**
- `/etc/hosts` file-এ entry যোগ করুন
- `prod.myapp.com` কে `127.0.0.1`-এ map করুন
- Verify করুন যে resolution কাজ করছে
- এরপর সেই entry remove করুন এবং আবার verify করুন

**Hint:** `sudo nano /etc/hosts`, `ping prod.myapp.com`, `dig prod.myapp.com` ব্যবহার করুন


## TASK 12 - Active Connections Monitoring
**Scenario:** Production server slow হয়ে গেছে। আপনাকে দেখতে হবে কতগুলো active connection আছে এবং কোন IP সবচেয়ে বেশি connection করছে।

**আপনাকে যা করতে হবে:**
- সব active TCP connection দেখুন
- `ESTABLISHED` state-এর connection count করুন
- কোন remote IP সবচেয়ে বেশি বার connect করছে সেটা বের করুন
- Port 80-তে কতগুলো connection আছে সেটা count করুন

**Hint:** `ss -tan`, `ss -tan | grep ESTABLISHED | wc -l`, `awk` এবং `sort | uniq -c | sort -rn` ব্যবহার করুন


## TASK 13 - Network Interface Configuration
**Scenario:** একটা নতুন server-এ static IP configure করতে হবে। Server-টা Ubuntu-তে আছে।

**আপনাকে যা করতে হবে:**
- Current network interface details দেখুন
- Interface-এর MAC address বের করুন
- Network statistics দেখুন (packets sent/received, errors)
- `nmcli` দিয়ে connection details দেখুন

**Hint:** `ip link show`, `ip -s link`, `nmcli device show`, `nmcli connection show` ব্যবহার করুন


## TASK 14 - Automated Network Health Check Script
**Scenario:** আপনার manager বললো "এমন একটা script লিখুন যেটা প্রতিদিন automatically run হবে এবং important servers-এর connectivity check করবে।"

**আপনাকে যা করতে হবে:**
- একটা bash script লিখুন যেটা নিচের কাজগুলো করবে:
  - ৩টা server (`8.8.8.8`, `1.1.1.1`, `google.com`) ping করবে
  - প্রতিটার জন্য "UP" বা "DOWN" print করবে
  - Result একটা log file-এ date/time সহ save করবে
  - কোনো server DOWN হলে সেটা highlight করবে

**Hint:** `ping -c 1 -W 2`, `if` statement, `date` command, `>> logfile` ব্যবহার করুন


## TASK 15 - SSH Hardening + Security Audit
**Scenario:** Security team একটা audit করবে। আপনাকে SSH configuration harden করতে হবে এবং একটা report তৈরি করতে হবে।

**আপনাকে যা করতে হবে:**
- `/etc/ssh/sshd_config` দেখুন এবং নিচের settings check করুন:
  - Root login disabled আছে কিনা
  - Password authentication disabled আছে কিনা
  - SSH port default (22) থেকে পরিবর্তন করা আছে কিনা
- Currently কতজন user SSH-এ logged in আছে দেখুন
- SSH login attempt logs দেখুন (failed attempts)
- সব findings একটা `ssh_audit_report.txt` file-এ লিখুন

**Hint:** `grep -E "PermitRootLogin|PasswordAuth|Port" /etc/ssh/sshd_config`, `who`, `w`, `grep "Failed" /var/log/auth.log` ব্যবহার করুন


## TASK 16 - Multi-Server SSH Automation (Jump Host / Bastion)

**Scenario:** আপনার production infrastructure-এ directly server-এ SSH করা যায় না। সব connection একটা **Bastion/Jump Host** এর মাধ্যমে যায়। আপনাকে এই setup configure করতে হবে।


আপনার Laptop → Bastion Server (203.0.113.5) → App Server (10.0.1.20)
                                              → DB Server (10.0.1.21)

**আপনাকে যা করতে হবে:**
- SSH config file-এ Jump Host configure করুন
- Single command-এ Bastion হয়ে App Server-এ connect করুন
- Bastion হয়ে DB Server-এ একটা file copy করুন `scp` দিয়ে
- Connection কাজ করছে কিনা verify করুন

**Hint:** `ProxyJump` অথবা `ProxyCommand` in `~/.ssh/config`, `scp -J` flag ব্যবহার করুন


## TASK 17 - SSL Certificate Expiry Check (Production Critical!)

**Scenario:** Production-এ SSL certificate expire হয়ে গেলে site down হয়ে যায়। আপনার manager বললো "একটা script লিখুন যেটা সব important domain-এর SSL certificate কতদিন বাকি আছে সেটা check করবে এবং ৩০ দিনের কম হলে warning দিবে।"

**আপনাকে যা করতে হবে:**
- `curl` বা `openssl` দিয়ে একটা domain-এর SSL certificate info দেখুন
- Certificate expiry date বের করুন
- আজকের date-এর সাথে compare করে কতদিন বাকি হিসাব করুন
- ৩০ দিনের কম হলে `⚠️ WARNING` print করুন
- Multiple domain check করার script লিখুন

**Hint:** `openssl s_client -connect domain:443`, `echo | openssl s_client`, `date` command দিয়ে date calculation করুন।


## TASK 18 - Network Bandwidth Monitoring & Throttling Detection

**Scenario:** আপনার team complaint করেছে যে রাত ১২টার পরে deployment slow হয়ে যায়। আপনাকে network bandwidth usage monitor করতে হবে এবং কোন process সবচেয়ে বেশি bandwidth use করছে সেটা খুঁজে বের করতে হবে।

**আপনাকে যা করতে হবে:**
- Real-time network traffic দেখুন interface-wise (bytes/second)
- কোন process কতটুকু bandwidth use করছে দেখুন
- একটা নির্দিষ্ট সময়ের network usage log করুন
- Upload এবং Download speed আলাদা আলাদা দেখুন

**Hint:** `iftop`, `nethogs`, `nload`, `/proc/net/dev` file read করুন, `ifstat` ব্যবহার করুন


## TASK 19 - Automated Log Collection from Multiple Servers

**Scenario:** Incident হয়েছে! আপনাকে ৩টা server থেকে logs collect করে একটা জায়গায় আনতে হবে analysis-এর জন্য। Manually করলে অনেক সময় লাগবে তাই script দিয়ে করতে হবে।

**আপনাকে যা করতে হবে:**
- একটা script লিখুন যেটা multiple server থেকে logs নিয়ে আসবে
- প্রতিটা server-এর জন্য আলাদা directory তৈরি করবে
- Log files compress করে আনবে
- Collection successful হয়েছে কিনা verify করবে
- একটা summary report তৈরি করবে

**Hint:** `rsync` অথবা `scp` loop-এ use করুন, `tar -czf` দিয়ে compress করুন, `ssh user@server "command"` দিয়ে remote command run করুন


## TASK 20 - DNS Failover Testing & Verification

**Scenario:** আপনাদের application-এর DNS failover setup আছে। primary server down হলে automatically secondary-তে যাবে। আপনাকে এই setup test করতে হবে এবং verify করতে হবে।

**আপনাকে যা করতে হবে:**
- একটা domain-এর সব DNS records দেখুন (A, CNAME, TTL)
- Different DNS servers থেকে same domain query করুন (8.8.8.8, 1.1.1.1, 9.9.9.9)
- TTL value কত এবং কেন এটা important সেটা document করুন
- DNS propagation check করুন, সব DNS server-এ same result আসছে কিনা
- একটা script লিখুন যেটা প্রতি ৫ সেকেন্ডে DNS response check করে

**Hint:** `dig @8.8.8.8 domain.com`, `dig @1.1.1.1 domain.com`, TTL field দেখুন, `watch` অথবা loop দিয়ে monitor করুন

## TASK 21 - Reverse Proxy & Port Redirection Testing

**Scenario:** আপনার server-এ NGINX Reverse Proxy already configure করা আছে। Port 80 থেকে আসা সব request port 8080-এ forward হবে। আপনাকে এই setup verify করতে হবে এবং response test করতে হবে।

**আপনাকে যা করতে হবে:**
- `curl` দিয়ে HTTP response code check করুন
- Redirect হচ্ছে কিনা trace করুন (`-L` flag)
- Response headers দেখুন - কোন server respond করছে
- Response time measure করুন
- একটা loop script লিখুন যেটা প্রতি সেকেন্ডে health check করে এবং response code log করে

**Hint:** `curl -I`, `curl -L`, `curl -w "%{http_code} %{time_total}"`, `curl -v` দিয়ে verbose output দেখুন

## TASK 22 - Complete Network Security Audit Script

**Scenario:** Monthly security audit-এর জন্য আপনাকে একটা **complete network security audit script** লিখতে হবে যেটা automatically সব important checks করবে এবং একটা professional report তৈরি করবে।

**Script-টা নিচের সব check করবে:**
1. Open ports এবং associated processes
2. Active network connections এবং suspicious IPs
3. Firewall rules status
4. SSH configuration security
5. Failed login attempts (last 24 ঘন্টা)
6. Listening services যেগুলো 0.0.0.0 (সব interface)-এ bind করা
7. Report একটা HTML file-এ export করবে

**Hint:** সব আগের commands একসাথে use করুন। HTML file তৈরিতে `echo` দিয়ে HTML tags লিখুন। `date`, `hostname`, `ss`, `ufw`, `grep` ব্যবহার করুন


# ✅ ANSWER KEY

## (সব task নিজে করার পর এখান থেকে মিলিয়ে নিন)


### TASK 1 - Answer

```bash
# সব network info collect করে file-এ save করুন
echo "=== Network Information Report ===" > network_info.txt
echo "Date: $(date)" >> network_info.txt

echo -e "\n--- IP Addresses ---" >> network_info.txt
ip addr show >> network_info.txt

echo -e "\n--- Hostname ---" >> network_info.txt
hostname -f >> network_info.txt

echo -e "\n--- Default Gateway ---" >> network_info.txt
ip route show default >> network_info.txt

echo -e "\n--- DNS Servers ---" >> network_info.txt
cat /etc/resolv.conf >> network_info.txt

# File দেখুন
cat network_info.txt
```

**কেন এভাবে করলাম:**
- `>` দিয়ে নতুন file তৈরি করলাম (প্রথমবার)
- `>>` দিয়ে existing file-এ append করলাম (প্রতিটা section যোগ করতে)
- `$(date)` দিয়ে current date/time নিলাম - এটা command substitution
- `echo -e` দিয়ে newline `\n` কাজ করে


### TASK 2 - Answer

```bash
# Step 1: Basic connectivity test
ping -c 4 192.168.1.50
# -c 4 মানে ৪টা packet পাঠাও তারপর থামো

# Step 2: Route trace - packet কোথায় আটকায় দেখুন
traceroute 192.168.1.50
# অথবা আরো detailed:
mtr --report 192.168.1.50
# mtr = ping + traceroute একসাথে, real-time দেখায়

# Step 3: DNS check
dig google.com
# দেখুন ANSWER SECTION-এ IP আসছে কিনা

# Step 4: Alternative DNS check
nslookup google.com

# Troubleshooting logic:
# - ping fail → network/firewall সমস্যা
# - traceroute timeout → কোন hop-এ block হচ্ছে বোঝা যাবে
# - dig fail → DNS সমস্যা
```

**DevOps Troubleshooting Order (মনে রাখো):**
```
ping → traceroute/mtr → dig/nslookup → ss/netstat (port check)
```


### TASK 3 - Answer

```bash
# সব listening port দেখুন (process name সহ)
sudo ss -tulnp

# বিকল্প (older systems-এ):
sudo netstat -tulnp

# Specific port check করুন
sudo lsof -i :22    # SSH
sudo lsof -i :80    # HTTP
sudo lsof -i :443   # HTTPS

# Output file-এ save করুন
echo "=== Open Ports Audit - $(date) ===" > open_ports_audit.txt
sudo ss -tulnp >> open_ports_audit.txt

echo -e "\n=== Port 22 (SSH) ===" >> open_ports_audit.txt
sudo lsof -i :22 >> open_ports_audit.txt

echo -e "\n=== Port 80 (HTTP) ===" >> open_ports_audit.txt
sudo lsof -i :80 >> open_ports_audit.txt

echo -e "\n=== Port 443 (HTTPS) ===" >> open_ports_audit.txt
sudo lsof -i :443 >> open_ports_audit.txt
```

**`ss -tulnp` Flag মানে:**
| Flag | মানে |
|------|------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening only |
| `-n` | Numeric (hostname resolve করবে না) |
| `-p` | Process name দেখুন |


### TASK 4 - Answer

```bash
# Step 1: SSH key pair generate করুন
ssh-keygen -t rsa -b 4096 -C "devops@mycompany.com"
# -t rsa = RSA type
# -b 4096 = 4096 bit (stronger)
# -C = comment (optional, label হিসেবে কাজ করে)
# Enter চাপলে default location (~/.ssh/id_rsa) use হবে

# Step 2: Public key remote server-এ copy করুন
ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote-server
# এটা automatically ~/.ssh/authorized_keys-এ add করে

# Step 3: Test করুন
ssh user@remote-server
# Password ছাড়াই login হওয়া উচিত

# Step 4: SSH config file তৈরি করুন
nano ~/.ssh/config
```

```
# ~/.ssh/config এর ভেতরে লিখুন:
Host myserver
    HostName 192.168.1.100
    User ubuntu
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

```bash
# এখন শুধু এটুকু লিখলেই connect হবে:
ssh myserver

# Config file-এর permission ঠিক রাখো
chmod 600 ~/.ssh/config
```


### TASK 5 - Answer

```bash
# SCP দিয়ে local → remote
scp /path/to/deployment.tar.gz user@192.168.1.100:/opt/deployments/
# Directory পাঠাতে:
scp -r /local/directory/ user@192.168.1.100:/remote/path/

# SCP দিয়ে remote → local
scp user@192.168.1.100:/var/log/app.log ./logs/

# Rsync দিয়ে directory sync (শুধু changed files)
rsync -avz --progress user@192.168.1.100:/var/log/ ./remote-logs/
# -a = archive (permissions, timestamps সব preserve করে)
# -v = verbose
# -z = compress করে transfer করে
# --progress = progress দেখুন

# Rsync দিয়ে local → remote (delete old files)
rsync -avz --delete /local/app/ user@server:/opt/app/
# --delete = source-এ নেই এমন files remote থেকে delete করবে

# Wget দিয়ে download
wget https://example.com/file.tar.gz -O myfile.tar.gz

# Curl দিয়ে download
curl -O https://example.com/file.tar.gz
# অথবা অন্য নামে save করতে:
curl -o myfile.tar.gz https://example.com/file.tar.gz
```


### TASK 6 - Answer

```bash
# A record (IP address)
dig api.github.com A

# MX record (mail server)
dig api.github.com MX

# NS record (name server)
dig api.github.com NS

# সব record একসাথে
dig api.github.com ANY

# Response time দেখুন (Query time: X msec)
dig api.github.com | grep "Query time"

# Specific DNS server use করে query
dig @8.8.8.8 api.github.com

# Shorter output (just the answer)
dig +short api.github.com

# nslookup দিয়ে
nslookup api.github.com

# host command দিয়ে (সবচেয়ে simple)
host api.github.com

# Current DNS server দেখুন
cat /etc/resolv.conf

# /etc/hosts check করুন
cat /etc/hosts
```

**Output-এ কী দেখবে:**
```
;; ANSWER SECTION:
api.github.com.    60  IN  A  140.82.121.5

;; Query time: 23 msec   ← এটা DNS response time
;; SERVER: 8.8.8.8       ← কোন DNS server use হলো
```


### TASK 7 - Answer

```bash
# === UFW দিয়ে (Ubuntu) ===

# Current status দেখুন
sudo ufw status verbose

# UFW enable করুন (যদি off থাকে)
sudo ufw enable

# Port 80 এবং 443 allow করুন
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Port 22 শুধু specific IP থেকে allow করুন
sudo ufw allow from 203.0.113.10 to any port 22

# Default policy - incoming সব deny করুন
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Rules দেখুন
sudo ufw status numbered

# === Firewalld দিয়ে (CentOS/RHEL) ===
# HTTP এবং HTTPS allow
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# Specific IP থেকে SSH allow
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="203.0.113.10" service name="ssh" accept'

# Apply changes
sudo firewall-cmd --reload

# Status দেখুন
sudo firewall-cmd --list-all
```


### TASK 8 - Answer

```bash
# সব traffic capture করুন eth0-তে (30 সেকেন্ড)
sudo tcpdump -i eth0 -c 100
# অথবা time limit দিয়ে:
sudo timeout 30 tcpdump -i eth0

# শুধু port 80 (HTTP) traffic
sudo tcpdump -i eth0 port 80

# Verbose output দিয়ে HTTP traffic
sudo tcpdump -i eth0 -v port 80

# Specific IP-র traffic দেখুন
sudo tcpdump -i eth0 host 8.8.8.8

# File-এ save করুন (binary format)
sudo tcpdump -i eth0 -w /tmp/capture.pcap

# Human readable format-এ পড়ুন
sudo tcpdump -r /tmp/capture.pcap

# HTTP GET request দেখুন (ascii output)
sudo tcpdump -i eth0 -A port 80 | grep "GET\|POST\|Host"

# Source এবং destination IP দেখুন
sudo tcpdump -i eth0 -n port 80
# -n = hostname resolve করবে না (faster)
```


### TASK 9 - Answer

```bash
# Local Port Forwarding (সবচেয়ে common use case)
# Local machine-এর 5433 port → remote server-এর 5432 (PostgreSQL)
ssh -L 5433:localhost:5432 user@jump-server.com

# Background-এ run করুন (no shell, just tunnel)
ssh -N -f -L 5433:localhost:5432 user@jump-server.com
# -N = command execute করবে না (শুধু tunnel)
# -f = background-এ পাঠাও

# Verify করুন tunnel কাজ করছে কিনা
ss -tlnp | grep 5433
# অথবা
netstat -tlnp | grep 5433

# এখন local machine থেকে database connect করুন:
psql -h localhost -p 5433 -U dbuser mydb

# Reverse Tunnel (remote → local):
# Remote server-এর 8080 port → local machine-এর 80 port
ssh -R 8080:localhost:80 user@remote-server.com
# Use case: local development server কে internet-এ expose করা

# Tunnel বন্ধ করতে:
# foreground tunnel: Ctrl+C
# background tunnel:
kill $(pgrep -f "ssh -N")
```

### TASK 10 - Answer

```bash
# wget দিয়ে progress দেখিয়ে download
wget --progress=bar https://releases.ubuntu.com/22.04/ubuntu-22.04-live-server-amd64.iso

# Download incomplete হলে resume করুন
wget -c https://example.com/largefile.tar.gz
# -c = continue (আগের progress থেকে শুরু করুন)

# curl দিয়ে HTTP headers দেখুন
curl -I https://google.com
# Output দেখাবে:
# HTTP/2 200
# content-type: text/html
# server: gws
# date: ...

# curl দিয়ে download (progress দেখুন)
curl -O --progress-bar https://example.com/file.tar.gz

# Checksum verify করুন (integrity check)
md5sum downloaded-file.tar.gz
sha256sum downloaded-file.tar.gz

# Compare করুন official checksum-এর সাথে:
echo "expectedhash  downloaded-file.tar.gz" | sha256sum -c
# Output: downloaded-file.tar.gz: OK ← মানে সব ঠিক আছে
```

### TASK 11 - Answer

```bash
# /etc/hosts file দেখুন
cat /etc/hosts

# Entry যোগ করুন
sudo nano /etc/hosts
# নিচের line add করুন:
# 127.0.0.1   prod.myapp.com

# অথবা single command দিয়ে:
echo "127.0.0.1 prod.myapp.com" | sudo tee -a /etc/hosts

# Verify করুন
ping -c 2 prod.myapp.com
# Output: PING prod.myapp.com (127.0.0.1)

# dig দিয়ে verify
dig prod.myapp.com
# দেখুন ANSWER SECTION-এ 127.0.0.1 দেখাচ্ছে কিনা

# Entry remove করুন
sudo nano /etc/hosts
# যে line add করেছিলে সেটা delete করুন

# আবার verify করুন
ping -c 2 prod.myapp.com
# এখন আর resolve হবে না (অথবা real IP দেখাবে)
```

**Real-world use case:**
- Development-এ production domain test করতে
- Staging environment-এ specific domain point করতে
- DNS propagation-এর আগে test করতে


### TASK 12 - Answer

```bash
# সব TCP connection দেখুন (state সহ)
ss -tan

# ESTABLISHED connections count করুন
ss -tan | grep ESTABLISHED | wc -l

# State অনুযায়ী connection count করুন
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
# Output:
# 45 ESTABLISHED
# 12 TIME-WAIT
#  3 LISTEN

# কোন remote IP সবচেয়ে বেশি connect করছে
ss -tan | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10

# Port 80-এ connection count
ss -tan | grep ':80' | wc -l

# Real-time connection monitor
watch -n 2 'ss -tan | grep ESTABLISHED | wc -l'
# প্রতি ২ সেকেন্ডে update হবে

# Top 5 connecting IPs দেখুন
netstat -tn | awk '/ESTABLISHED/{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -5
```


### TASK 13 - Answer

```bash
# Interface details দেখুন
ip addr show
ip link show

# Specific interface (যেমন eth0)
ip addr show eth0

# MAC address বের করুন
ip link show eth0 | grep "link/ether"
# Output: link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff

# Network statistics (packets, errors)
ip -s link show eth0
# Output দেখাবে:
# RX: bytes packets errors
# TX: bytes packets errors

# nmcli দিয়ে দেখুন
nmcli device show eth0

# সব connection দেখুন
nmcli connection show

# Active connections দেখুন
nmcli device status

# Interface statistics আরো detail-এ
cat /proc/net/dev
```


### TASK 14 - Answer

```bash
#!/bin/bash
# network_health_check.sh

# Configuration
SERVERS=("8.8.8.8" "1.1.1.1" "google.com")
LOG_FILE="/var/log/network_health.log"
DATE=$(date "+%Y-%m-%d %H:%M:%S")

echo "========================================" | tee -a $LOG_FILE
echo "Network Health Check - $DATE" | tee -a $LOG_FILE
echo "========================================" | tee -a $LOG_FILE

# Flag to track if any server is down
ALL_UP=true

for server in "${SERVERS[@]}"; do
    # Ping with 1 packet, 2 second timeout
    if ping -c 1 -W 2 "$server" &>/dev/null; then
        STATUS="✅ UP"
    else
        STATUS="❌ DOWN"
        ALL_UP=false
    fi

    echo "[$DATE] $server: $STATUS" | tee -a $LOG_FILE
done

# Summary
if [ "$ALL_UP" = true ]; then
    echo "[INFO] All servers are reachable." | tee -a $LOG_FILE
else
    echo "[WARNING] ⚠️  One or more servers are DOWN!" | tee -a $LOG_FILE
fi

echo "" >> $LOG_FILE
```

```bash
# Script-কে executable করুন
chmod +x network_health_check.sh

# Run করুন
sudo ./network_health_check.sh

# Log দেখুন
cat /var/log/network_health.log
```


### TASK 15 - Answer

```bash
# SSH config check করুন
echo "=== SSH Security Audit Report ===" > ssh_audit_report.txt
echo "Date: $(date)" >> ssh_audit_report.txt

# Key settings check করুন
echo -e "\n--- SSH Config Settings ---" >> ssh_audit_report.txt
grep -E "^PermitRootLogin|^PasswordAuthentication|^Port|^PubkeyAuthentication" \
    /etc/ssh/sshd_config >> ssh_audit_report.txt

# যদি comment করা থাকে তাহলে:
grep -E "PermitRootLogin|PasswordAuthentication|Port" \
    /etc/ssh/sshd_config >> ssh_audit_report.txt

# Currently logged in users দেখুন
echo -e "\n--- Currently Logged In Users ---" >> ssh_audit_report.txt
who >> ssh_audit_report.txt
w >> ssh_audit_report.txt

# Failed SSH attempts (last 20)
echo -e "\n--- Failed SSH Login Attempts (Last 20) ---" >> ssh_audit_report.txt
grep "Failed password" /var/log/auth.log | tail -20 >> ssh_audit_report.txt

# Successful SSH logins
echo -e "\n--- Successful SSH Logins (Last 10) ---" >> ssh_audit_report.txt
grep "Accepted" /var/log/auth.log | tail -10 >> ssh_audit_report.txt

# Report দেখুন
cat ssh_audit_report.txt

# Ideal SSH config কেমন হওয়া উচিত:
# PermitRootLogin no         ← root সরাসরি login করতে পারবে না
# PasswordAuthentication no  ← password দিয়ে login বন্ধ
# PubkeyAuthentication yes   ← শুধু key দিয়ে login
# Port 2222                  ← default port পরিবর্তন (optional)
```


### TASK 16 - Answer: Jump Host / Bastion Setup

```bash
# ~/.ssh/config file-এ এটা লিখুন:
nano ~/.ssh/config
```

```bash
# Bastion/Jump Host
Host bastion
    HostName 203.0.113.5
    User ec2-user
    IdentityFile ~/.ssh/devops-key.pem

# App Server - Bastion হয়ে connect
Host appserver
    HostName 10.0.1.20
    User ubuntu
    IdentityFile ~/.ssh/devops-key.pem
    ProxyJump bastion          # ← এটাই magic! Bastion হয়ে যাবে

# DB Server - Bastion হয়ে connect
Host dbserver
    HostName 10.0.1.21
    User ubuntu
    IdentityFile ~/.ssh/devops-key.pem
    ProxyJump bastion
```

```bash
# এখন সরাসরি connect করুন (Bastion automatically use হবে)
ssh appserver
ssh dbserver

# Bastion হয়ে SCP করুন
scp -J bastion /local/file.txt ubuntu@10.0.1.20:/home/ubuntu/

# অথবা config use করে:
scp /local/file.txt appserver:/home/ubuntu/

# Verify করুন - কোন path দিয়ে connect হচ্ছে
ssh -v appserver 2>&1 | grep "Connecting\|Connected\|proxy"

# Old style (ProxyCommand) - পুরনো SSH version-এ:
# ProxyCommand ssh -W %h:%p bastion
```

**কেন Bastion Host ব্যবহার করি:**

- Private servers publicly expose করতে হয় না
- Single point of entry → easy to audit
- Production security best practice
- AWS, GCP, Azure সবাই এই pattern use করে


### TASK 17 - Answer: SSL Certificate Expiry Check

```bash
# Single domain-এর certificate দেখুন
echo | openssl s_client -connect google.com:443 -servername google.com 2>/dev/null \
    | openssl x509 -noout -dates

# Output:
# notBefore=Jan  1 00:00:00 2024 GMT
# notAfter=Apr  1 00:00:00 2025 GMT   ← এই date-টা দরকার

# শুধু expiry date:
echo | openssl s_client -connect google.com:443 2>/dev/null \
    | openssl x509 -noout -enddate

# Certificate কতদিন বাকি (manually):
openssl s_client -connect google.com:443 2>/dev/null \
    | openssl x509 -noout -checkend 2592000
# 2592000 seconds = 30 days
# Exit code 0 = ৩০ দিনের বেশি বাকি
# Exit code 1 = ৩০ দিনের কম বাকি (expire হবে)
```

```bash
#!/bin/bash
# ssl_expiry_check.sh

DOMAINS=("google.com" "github.com" "amazon.com")
WARNING_DAYS=30
LOG_FILE="ssl_audit_$(date +%Y%m%d).txt"

echo "============================================" | tee $LOG_FILE
echo " SSL Certificate Expiry Report" | tee -a $LOG_FILE
echo " Date: $(date)" | tee -a $LOG_FILE
echo "============================================" | tee -a $LOG_FILE

for domain in "${DOMAINS[@]}"; do
    # Expiry date বের করুন
    EXPIRY=$(echo | openssl s_client -connect "$domain:443" \
        -servername "$domain" 2>/dev/null \
        | openssl x509 -noout -enddate 2>/dev/null \
        | cut -d= -f2)

    if [ -z "$EXPIRY" ]; then
        echo "❌ $domain : Could not retrieve certificate" | tee -a $LOG_FILE
        continue
    fi

    # Expiry date কে seconds-এ convert করুন
    EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
    NOW_EPOCH=$(date +%s)

    # কতদিন বাকি হিসাব করুন
    DAYS_LEFT=$(( (EXPIRY_EPOCH - NOW_EPOCH) / 86400 ))

    # Warning check
    if [ "$DAYS_LEFT" -le "$WARNING_DAYS" ]; then
        STATUS="⚠️  WARNING - EXPIRES SOON!"
    elif [ "$DAYS_LEFT" -le 0 ]; then
        STATUS= CRITICAL - ALREADY EXPIRED!"
    else
        STATUS="✅ OK"
    fi

    echo "$STATUS | $domain | Days Left: $DAYS_LEFT | Expires: $EXPIRY" \
        | tee -a $LOG_FILE
done

echo "============================================" | tee -a $LOG_FILE
echo "Report saved to: $LOG_FILE"
```

```bash
chmod +x ssl_expiry_check.sh
./ssl_expiry_check.sh

# Sample Output:
# ✅ OK        | google.com  | Days Left: 187 | Expires: Sep 15 ...
# ⚠️ WARNING  | myapp.com   | Days Left: 12  | Expires: Apr  2 ...
```


### TASK 18 - Answer: Network Bandwidth Monitoring

```bash
# Method 1: /proc/net/dev দিয়ে (সবসময় available)
cat /proc/net/dev
# দেখাবে প্রতিটা interface-এর bytes received/transmitted

# Real-time bandwidth দেখার simple script:
watch -n 1 'cat /proc/net/dev | grep -v lo'

# Method 2: iftop (install করতে হবে)
sudo apt install iftop -y
sudo iftop -i eth0
# Real-time connection-wise bandwidth দেখাবে

# Method 3: nethogs - process-wise bandwidth
sudo apt install nethogs -y
sudo nethogs eth0
# কোন process কতটুকু bandwidth use করছে দেখাবে

# Method 4: nload - simple interface bandwidth
sudo apt install nload -y
nload eth0

# Method 5: Script দিয়ে bandwidth log করুন
#!/bin/bash
# bandwidth_monitor.sh

INTERFACE="eth0"
LOG_FILE="bandwidth_log.txt"
INTERVAL=5  # seconds

echo "Time | RX (KB/s) | TX (KB/s)" | tee $LOG_FILE

while true; do
    # আগের reading
    RX1=$(cat /proc/net/dev | grep $INTERFACE | awk '{print $2}')
    TX1=$(cat /proc/net/dev | grep $INTERFACE | awk '{print $10}')

    sleep $INTERVAL

    # নতুন reading
    RX2=$(cat /proc/net/dev | grep $INTERFACE | awk '{print $2}')
    TX2=$(cat /proc/net/dev | grep $INTERFACE | awk '{print $10}')

    # Speed calculate করুন (KB/s)
    RX_SPEED=$(( (RX2 - RX1) / INTERVAL / 1024 ))
    TX_SPEED=$(( (TX2 - TX1) / INTERVAL / 1024 ))

    echo "$(date +%H:%M:%S) | RX: ${RX_SPEED} KB/s | TX: ${TX_SPEED} KB/s" \
        | tee -a $LOG_FILE
done
```

```bash
chmod +x bandwidth_monitor.sh
./bandwidth_monitor.sh

# Sample Output:
# 14:32:01 | RX: 245 KB/s | TX: 12 KB/s
# 14:32:06 | RX: 890 KB/s | TX: 45 KB/s
```


### TASK 19 - Answer: Multi-Server Log Collection

```bash
#!/bin/bash
# collect_logs.sh

# Configuration
SERVERS=("web01@192.168.1.10" "web02@192.168.1.11" "db01@192.168.1.12")
LOG_DIR="/opt/logs/$(date +%Y%m%d_%H%M%S)"
SSH_KEY="~/.ssh/devops-key.pem"
SUMMARY_FILE="$LOG_DIR/collection_summary.txt"

# Base directory তৈরি করুন
mkdir -p "$LOG_DIR"

echo "🚀 Log Collection Started: $(date)" | tee $SUMMARY_FILE
echo "==========================================" | tee -a $SUMMARY_FILE

for server in "${SERVERS[@]}"; do
    # Server name extract করুন (@ এর পরের অংশ)
    SERVER_NAME=$(echo $server | cut -d@ -f2 | tr '.' '_')
    SERVER_DIR="$LOG_DIR/$SERVER_NAME"

    echo -e "\n📦 Collecting from: $server" | tee -a $SUMMARY_FILE

    # Server-এর জন্য directory তৈরি করুন
    mkdir -p "$SERVER_DIR"

    # Remote server-এ logs compress করুন এবং নিয়ে আসো
    ssh -i $SSH_KEY $server \
        "tar -czf /tmp/logs_$(date +%Y%m%d).tar.gz \
        /var/log/syslog \
        /var/log/auth.log \
        /var/log/nginx/ \
        2>/dev/null; \
        echo 'Compression done'" 2>/dev/null

    # Compressed file local-এ নিয়ে আসো
    scp -i $SSH_KEY \
        "$server:/tmp/logs_$(date +%Y%m%d).tar.gz" \
        "$SERVER_DIR/" 2>/dev/null

    # Verify করুন
    if [ -f "$SERVER_DIR/logs_$(date +%Y%m%d).tar.gz" ]; then
        FILE_SIZE=$(du -sh "$SERVER_DIR/logs_$(date +%Y%m%d).tar.gz" | cut -f1)
        echo "✅ Success | Size: $FILE_SIZE" | tee -a $SUMMARY_FILE

        # Remote temp file clean করুন
        ssh -i $SSH_KEY $server "rm /tmp/logs_$(date +%Y%m%d).tar.gz" 2>/dev/null
    else
        echo "❌ FAILED - Could not collect logs" | tee -a $SUMMARY_FILE
    fi
done

echo -e "\n==========================================" | tee -a $SUMMARY_FILE
echo "✅ Collection Complete: $(date)" | tee -a $SUMMARY_FILE
echo "📁 Logs saved to: $LOG_DIR" | tee -a $SUMMARY_FILE

# Summary দেখুন
cat $SUMMARY_FILE
```

```bash
chmod +x collect_logs.sh
./collect_logs.sh

# Output Structure:
# /opt/logs/20250321_143000/
# ├── 192_168_1_10/
# │   └── logs_20250321.tar.gz
# ├── 192_168_1_11/
# │   └── logs_20250321.tar.gz
# ├── 192_168_1_12/
# │   └── logs_20250321.tar.gz
# └── collection_summary.txt
```

### TASK 20 - Answer: DNS Failover Testing

```bash
# একটা domain-এর full DNS record দেখুন
dig google.com ANY +noall +answer

# TTL value দেখুন (3rd column)
dig google.com A +noall +answer
# Output:
# google.com.  300  IN  A  142.250.184.14
#              ^^^
#              TTL = 300 seconds (৫ মিনিট)

# Different DNS server থেকে query করুন
echo "=== Google DNS (8.8.8.8) ===" && dig @8.8.8.8 yourapp.com +short
echo "=== Cloudflare (1.1.1.1) ===" && dig @1.1.1.1 yourapp.com +short
echo "=== Quad9 (9.9.9.9) ===" && dig @9.9.9.9 yourapp.com +short

# DNS Propagation Check Script:
#!/bin/bash
# dns_propagation_check.sh

DOMAIN="google.com"
DNS_SERVERS=("8.8.8.8" "1.1.1.1" "9.9.9.9" "208.67.222.222")
DNS_NAMES=("Google" "Cloudflare" "Quad9" "OpenDNS")

echo "DNS Propagation Check: $DOMAIN"
echo "================================"

for i in "${!DNS_SERVERS[@]}"; do
    RESULT=$(dig @"${DNS_SERVERS[$i]}" "$DOMAIN" +short 2>/dev/null | head -1)
    TTL=$(dig @"${DNS_SERVERS[$i]}" "$DOMAIN" +noall +answer 2>/dev/null \
        | awk '{print $2}' | head -1)

    if [ -n "$RESULT" ]; then
        echo "✅ ${DNS_NAMES[$i]} (${DNS_SERVERS[$i]}): $RESULT | TTL: ${TTL}s"
    else
        echo "❌ ${DNS_NAMES[$i]} (${DNS_SERVERS[$i]}): No response"
    fi
done

# Real-time DNS monitoring (প্রতি ৫ সেকেন্ডে)
echo ""
echo "🔄 Real-time monitoring (Ctrl+C to stop)..."
while true; do
    RESULT=$(dig @8.8.8.8 "$DOMAIN" +short 2>/dev/null | head -1)
    echo "$(date +%H:%M:%S) | $DOMAIN → $RESULT"
    sleep 5
done
```

**TTL কেন important:**
```
TTL = 300 → DNS change করলে ৫ মিনিটে propagate হবে
TTL = 3600 → DNS change করলে ১ ঘন্টা লাগবে

Failover-এর আগে TTL কমিয়ে দাও (যেমন 60)
তাহলে failover দ্রুত হবে!
```

### TASK 21 - Answer: Reverse Proxy & Port Testing

```bash
# HTTP response code দেখুন
curl -o /dev/null -s -w "%{http_code}" http://localhost

# Response headers দেখুন
curl -I http://localhost
# Output:
# HTTP/1.1 200 OK
# Server: nginx/1.18.0
# Content-Type: text/html

# Redirect trace করুন
curl -IL http://localhost
# -I = headers only
# -L = follow redirects

# Detailed timing দেখুন
curl -w "\nDNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTotal: %{time_total}s\n" \
    -o /dev/null -s http://google.com

# Verbose output (সব details)
curl -v http://localhost 2>&1 | head -30

# Health Check Loop Script:
#!/bin/bash
# health_check_loop.sh

URL="http://localhost"
LOG_FILE="health_check.log"
INTERVAL=5

echo "🔄 Health Check Started for: $URL"
echo "Timestamp | HTTP Code | Response Time" | tee $LOG_FILE

while true; do
    # Response code এবং time একসাথে পাও
    RESPONSE=$(curl -o /dev/null -s \
        -w "%{http_code} %{time_total}" \
        --max-time 5 \
        "$URL")

    HTTP_CODE=$(echo $RESPONSE | awk '{print $1}')
    RESPONSE_TIME=$(echo $RESPONSE | awk '{print $2}')
    TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

    # Status check
    if [ "$HTTP_CODE" -eq 200 ]; then
        STATUS="✅ UP"
    elif [ "$HTTP_CODE" -eq 000 ]; then
        STATUS="❌ DOWN (No Response)"
    else
        STATUS="⚠️  HTTP $HTTP_CODE"
    fi

    LOG_LINE="$TIMESTAMP | $STATUS | ${RESPONSE_TIME}s"
    echo "$LOG_LINE" | tee -a $LOG_FILE

    sleep $INTERVAL
done
```

```bash
chmod +x health_check_loop.sh
./health_check_loop.sh

# Sample Output:
# 2025-03-21 14:32:01 | ✅ UP       | 0.023s
# 2025-03-21 14:32:06 | ✅ UP       | 0.019s
# 2025-03-21 14:32:11 | ❌ DOWN     | 0.000s   ← server down!
# 2025-03-21 14:32:16 | ⚠️  HTTP 502 | 0.045s  ← proxy error
```

### TASK 22 - Answer: Complete Network Security Audit Script

```bash
#!/bin/bash
# network_security_audit.sh
# Complete Network Security Audit - Professional Report Generator

# Configuration
REPORT_DIR="/opt/security-reports"
DATE=$(date +%Y%m%d_%H%M%S)
HTML_REPORT="$REPORT_DIR/security_audit_$DATE.html"
TEXT_REPORT="$REPORT_DIR/security_audit_$DATE.txt"

mkdir -p "$REPORT_DIR"

# ─────────────────────────────────────────
# Helper Functions
# ─────────────────────────────────────────

log() {
    echo "$1" | tee -a "$TEXT_REPORT"
}

html_section() {
    echo "<h2>$1</h2>" >> "$HTML_REPORT"
}

html_code() {
    echo "<pre>$1</pre>" >> "$HTML_REPORT"
}

# ─────────────────────────────────────────
# HTML Report Header
# ─────────────────────────────────────────

cat > "$HTML_REPORT" << EOF
<!DOCTYPE html>
<html>
<head>
<title>Network Security Audit Report</title>
<style>
  body { font-family: monospace; background: #1a1a2e; color: #e0e0e0; padding: 20px; }
  h1 { color: #00d4ff; border-bottom: 2px solid #00d4ff; }
  h2 { color: #ff6b6b; margin-top: 30px; }
  pre { background: #16213e; padding: 15px; border-radius: 5px;
        border-left: 4px solid #00d4ff; overflow-x: auto; }
  .ok    { color: #00ff88; }
  .warn  { color: #ffaa00; }
  .crit  { color: #ff4444; }
  .meta  { color: #888; font-size: 0.9em; }
</style>
</head>
<body>
<h1>🔐 Network Security Audit Report</h1>
<p class="meta">Generated: $(date) | Host: $(hostname) | By: $(whoami)</p>
EOF

# ─────────────────────────────────────────
# Section 1: Open Ports
# ─────────────────────────────────────────

log "=== [1] OPEN PORTS & SERVICES ==="
PORTS=$(sudo ss -tulnp 2>/dev/null)
html_section "1. Open Ports & Services"
html_code "$PORTS"
log "$PORTS"

# ─────────────────────────────────────────
# Section 2: Services on 0.0.0.0 (Exposed to ALL interfaces)
# ─────────────────────────────────────────

log -e "\n=== [2] SERVICES EXPOSED ON ALL INTERFACES (0.0.0.0) ==="
EXPOSED=$(sudo ss -tulnp | grep "0.0.0.0")
html_section "2. ⚠️ Services Exposed on ALL Interfaces (0.0.0.0)"
html_code "$EXPOSED"
log "$EXPOSED"

# ─────────────────────────────────────────
# Section 3: Active Connections
# ─────────────────────────────────────────

log -e "\n=== [3] ACTIVE ESTABLISHED CONNECTIONS ==="
CONNECTIONS=$(ss -tan | grep ESTABLISHED)
CONN_COUNT=$(echo "$CONNECTIONS" | wc -l)

html_section "3. Active Connections (Total: $CONN_COUNT)"
html_code "$CONNECTIONS"
log "Total ESTABLISHED: $CONN_COUNT"
log "$CONNECTIONS"

# Top connecting IPs
log -e "\n--- Top 5 Connecting IPs ---"
TOP_IPS=$(ss -tan | grep ESTABLISHED \
    | awk '{print $5}' \
    | cut -d: -f1 \
    | sort | uniq -c | sort -rn | head -5)
html_section "3b. Top Connecting IPs"
html_code "$TOP_IPS"
log "$TOP_IPS"

# ─────────────────────────────────────────
# Section 4: Firewall Rules
# ─────────────────────────────────────────

log -e "\n=== [4] FIREWALL RULES STATUS ==="
html_section "4. Firewall Rules"

if command -v ufw &>/dev/null; then
    UFW_STATUS=$(sudo ufw status verbose 2>/dev/null)
    html_code "$UFW_STATUS"
    log "$UFW_STATUS"
elif command -v firewall-cmd &>/dev/null; then
    FW_STATUS=$(sudo firewall-cmd --list-all 2>/dev/null)
    html_code "$FW_STATUS"
    log "$FW_STATUS"
else
    log "⚠️ No firewall (ufw/firewalld) found!"
    html_code "WARNING: No firewall detected!"
fi

# ─────────────────────────────────────────
# Section 5: SSH Configuration
# ─────────────────────────────────────────

log -e "\n=== [5] SSH SECURITY CONFIGURATION ==="
SSH_CONFIG=$(grep -E \
    "^Port|^PermitRootLogin|^PasswordAuthentication|\
^PubkeyAuthentication|^MaxAuthTries|^AllowUsers" \
    /etc/ssh/sshd_config 2>/dev/null)

html_section "5. SSH Configuration"
html_code "$SSH_CONFIG"
log "$SSH_CONFIG"

# ─────────────────────────────────────────
# Section 6: Failed Login Attempts (Last 24 hours)
# ─────────────────────────────────────────

log -e "\n=== [6] FAILED SSH LOGIN ATTEMPTS (Last 24h) ==="
html_section "6. Failed Login Attempts"

FAILED=$(grep "Failed password" /var/log/auth.log 2>/dev/null \
    | grep "$(date +%b\ %d)" | tail -20)

FAILED_COUNT=$(echo "$FAILED" | grep -c "Failed" || echo "0")

log "Total failed attempts today: $FAILED_COUNT"
html_code "Total Today: $FAILED_COUNT&#10;&#10;$FAILED"
log "$FAILED"

# Top attacking IPs
log -e "\n--- Top Attacking IPs ---"
TOP_ATTACKERS=$(grep "Failed password" /var/log/auth.log 2>/dev/null \
    | awk '{print $(NF-3)}' \
    | sort | uniq -c | sort -rn | head -5)
html_code "Top Attackers:&#10;$TOP_ATTACKERS"
log "$TOP_ATTACKERS"

# ─────────────────────────────────────────
# HTML Report Footer
# ─────────────────────────────────────────

cat >> "$HTML_REPORT" << EOF
<hr>
<p class="meta">Report End | Total Size: $(du -sh $HTML_REPORT | cut -f1)</p>
</body>
</html>
EOF

# ─────────────────────────────────────────
# Final Summary
# ─────────────────────────────────────────

echo ""
echo "============================================"
echo "✅ Audit Complete!"
echo "📄 Text Report : $TEXT_REPORT"
echo "🌐 HTML Report : $HTML_REPORT"
echo "============================================"
```

```bash
# Script run করুন
chmod +x network_security_audit.sh
sudo ./network_security_audit.sh

# HTML report browser-এ দেখুন (যদি GUI থাকে):
xdg-open /opt/security-reports/security_audit_*.html

# অথবা text report দেখুন:
cat /opt/security-reports/security_audit_*.txt
```


*Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../11-Network-Bonding-Bridging-VLANs">← Network Bonding, Bridging & VLANs</a>
    </td>
    <td align="right">
      <a href="../../Chapter-05/01-Shell-Scripting-Fundamentals/">Shell Scripting Fundamentals →</a>
    </td>
  </tr>
</table>