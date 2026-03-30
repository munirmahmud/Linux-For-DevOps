# Chapter 4 - Lesson 8: Firewall Management

**Chapter 4 | Lesson 8 of 11**


## Firewall কী এবং কেন দরকার?

কল্পনা করেন আপনার বাড়িতে একটা security guard আছে। সে সিদ্ধান্ত নেয় - কোন visitor ভেতরে যাবে, কাকে যেতে দেওয়া যাবে না। Linux-এ **Firewall** ঠিক এই কাজটাই করে।

> **Firewall** = একটা software-based security layer যে decide করে কোন network traffic আপনার system-এ **ঢুকতে পারবে**, **বের হতে পারবে**, বা **forward হবে**।

Linux-এ মূলত ৩টা firewall tool আছে:

| Tool | কোথায় ব্যবহার হয় | কেমন |
|---|---|---|
| `iptables` | সব distro-তে (low-level) | Powerful কিন্তু complex |
| `ufw` | Ubuntu/Debian | Simple & beginner-friendly |
| `firewalld` | CentOS/RHEL/Fedora | Zone-based, modern |


## Part 1: Linux Firewall এর মূল ধারণা - Netfilter & Chains

### Netfilter কী?

Linux kernel-এর ভেতরে **Netfilter** নামে একটা framework আছে। এটাই আসলে সব packet filtering করে। `iptables`, `ufw`, `firewalld` এরা সবাই এই Netfilter-কেই control করে, শুধু interface আলাদা।

```
Network Packet আসলে:
Internet → [Netfilter/iptables rules] → আপনার System
```

### Tables ও Chains

`iptables`-এ সব rules **table**-এর মধ্যে থাকে। সবচেয়ে গুরুত্বপূর্ণ table হলো **filter** table।

**filter table-এ ৩টা Chain আছে:**

```
┌─────────────────────────────────────────────┐
│              filter table                   │
│                                             │
│  INPUT   → আপনার system-এ আসা traffic     │
│  OUTPUT  → আপনার system থেকে যাওয়া traffic │
│  FORWARD → আপনার system দিয়ে pass করা      │
└─────────────────────────────────────────────┘
```

- **INPUT** = বাড়িতে কেউ ঢুকছে
- **OUTPUT** = বাড়ি থেকে কেউ বের হচ্ছে
- **FORWARD** = আপনার বাড়ির সামনে দিয়ে কেউ যাচ্ছে (আপনি router)


## Part 2: iptables - The Low-Level Powerhouse

### Basic Syntax

```bash
iptables -[Action] [Chain] [Conditions] -j [Target]
```

| Part | মানে |
|---|---|
| `-A` | Append (rule যোগ করো) |
| `-D` | Delete (rule মুছো) |
| `-I` | Insert (নির্দিষ্ট position-এ বসাও) |
| `-L` | List (সব rules দেখাও) |
| `-F` | Flush (সব rules মুছো) |
| `INPUT/OUTPUT` | কোন chain-এ |
| `-p tcp/udp` | Protocol |
| `--dport` | Destination port |
| `-s` | Source IP |
| `-j ACCEPT/DROP/REJECT` | কী করবে |


### সব rules দেখা

```bash
sudo iptables -L -v -n
```

```
# Expected Output:
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0    tcp dpt:22
DROP       tcp  --  192.168.1.100        0.0.0.0/0

Chain FORWARD (policy DROP)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

> **`-v`** = verbose (বিস্তারিত), **`-n`** = numeric (IP resolve না করে দেখাও)


### Port Allow করা (ACCEPT)

```bash
# Port 80 (HTTP) allow করা
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Port 443 (HTTPS) allow করা
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Port 22 (SSH) allow করা
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Breakdown:**
```
-A INPUT     → INPUT chain-এ rule যোগ করো
-p tcp       → TCP protocol
--dport 80   → destination port 80
-j ACCEPT    → Allow করো
```


### Port Block করা (DROP vs REJECT)

```bash
# Port 23 (Telnet) DROP করো - কোনো response দিবে না
sudo iptables -A INPUT -p tcp --dport 23 -j DROP

# Port 8080 REJECT করো - "connection refused" বলবে
sudo iptables -A INPUT -p tcp --dport 8080 -j REJECT
```

**DROP vs REJECT পার্থক্য:**
- `DROP` = চুপচাপ ফেলে দাও (attacker জানবে না কী হলো)
- `REJECT` = "না" বলে ফেরত পাঠাও (client জানবে connection refused)
- DevOps-এ বাইরের traffic-এর জন্য `DROP` বেশি secure


### নির্দিষ্ট IP Allow/Block করা

```bash
# নির্দিষ্ট IP থেকে সব traffic ALLOW করো
sudo iptables -A INPUT -s 192.168.1.50 -j ACCEPT

# নির্দিষ্ট IP থেকে সব traffic BLOCK করো
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# পুরো subnet block করো
sudo iptables -A INPUT -s 10.0.0.0/24 -j DROP
```


### Established Connection Allow করা

এটা অনেক গুরুত্বপূর্ণ! আপনি যদি সব INPUT block করেন, তাহলে আপনার নিজের করা request-এর response-ও আসবে না। তাই:

```bash
# Already established connection-এর traffic allow করুন
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

আপনি Google-এ request করলেন, Google-এর reply যেন আসতে পারে এই rule সেটা নিশ্চিত করে।


### Default Policy সেট করা

```bash
# সব incoming traffic block করা (default)
sudo iptables -P INPUT DROP

# সব outgoing traffic allow করা
sudo iptables -P OUTPUT ACCEPT

# Forward block করা
sudo iptables -P FORWARD DROP
```

> ⚠️ **সাবধান!** `INPUT DROP` set করার আগে SSH port allow করো, নইলে নিজেই lock out হয়ে যাবে!


### Rules Delete করা

```bash
# সব rules দেখাও (line number সহ)
sudo iptables -L --line-numbers

# নির্দিষ্ট rule delete করো (INPUT chain-এর rule 3)
sudo iptables -D INPUT 3

# সব rules flush (মুছে ফেলো)
sudo iptables -F
```


### iptables Rules Save করা

iptables rules reboot-এ হারিয়ে যায়! Save করতে:

```bash
# Ubuntu/Debian
sudo apt install iptables-persistent
sudo netfilter-persistent save

# RHEL/Fedora
sudo service iptables save
# অথবা
sudo iptables-save > /etc/sysconfig/iptables
```


## Part 3: UFW - Uncomplicated Firewall (Ubuntu-এর সহজ tool)

`ufw` আসলে `iptables` এরই একটা wrapper। ভেতরে iptables চলে, কিন্তু আপনি কিছু সহজ commands ব্যবহার করে ম্যানেজ করতে পারেন।

### UFW Install ও Enable

```bash
# Install (সাধারণত pre-installed)
sudo apt install ufw

# Status দেখা
sudo ufw status

# Enable করা
sudo ufw enable

# Disable করা
sudo ufw disable
```

```
# Output:
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
```


### UFW Default Policy

```bash
# Default: সব incoming block, সব outgoing allow
sudo ufw default deny incoming
sudo ufw default allow outgoing
```


### Port Allow করা

```bash
# Port 22 (SSH) allow
sudo ufw allow 22

# Port 80 (HTTP) allow
sudo ufw allow 80

# Port 443 (HTTPS) allow
sudo ufw allow 443

# Service name দিয়েও allow করা যায়
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Specific protocol দিয়ে
sudo ufw allow 22/tcp
sudo ufw allow 53/udp
```


### Port Block করা

```bash
# Port block করো
sudo ufw deny 8080

# Specific IP block করো
sudo ufw deny from 192.168.1.100

# Specific IP থেকে specific port block
sudo ufw deny from 192.168.1.100 to any port 22
```


### Specific IP Allow করা

```bash
# Specific IP-কে সব access দাও
sudo ufw allow from 192.168.1.50

# Specific IP-কে specific port-এ allow
sudo ufw allow from 192.168.1.50 to any port 22

# Subnet allow করো
sudo ufw allow from 10.0.0.0/24
```


### Rules Delete করা

```bash
# serial number অনুযায়ী Rules দেখা
sudo ufw status numbered

# Output:
# Status: active
#
#      To                         Action      From
#      --                         ------      ----
# [ 1] 22/tcp                     ALLOW IN    Anywhere
# [ 2] 80/tcp                     ALLOW IN    Anywhere
# [ 3] 443/tcp                    ALLOW IN    Anywhere

# Rule নম্বর দিয়ে delete করা
sudo ufw delete 2

# অথবা directly rule উল্লেখ করে delete করা
sudo ufw delete allow 80
```


### UFW Verbose Status

```bash
sudo ufw status verbose
```

```
# Output:
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
```


### UFW Application Profiles

```bash
# Available app profiles দেখা
sudo ufw app list

# Output:
# Available applications:
#   Nginx Full
#   Nginx HTTP
#   Nginx HTTPS
#   OpenSSH

# Application profile allow করা
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'
```


### UFW Logging

```bash
# Logging চালু করা
sudo ufw logging on

# Log level set করা
sudo ufw logging medium

# Logs দেখা
sudo tail -f /var/log/ufw.log
```


## Part 4: firewalld - Zone-Based Firewall (CentOS/RHEL)

`firewalld` আরও modern approach নেয় - **Zones** ব্যবহার করে।

### Zone কী?

Zone মানে হলো network-এর **trust level**।



**firewalld Zones**
                                               
- drop     → সব block, কোনো reply নেই        
- block    → সব block, ICMP error reply       
- public   → Public network (default)          
- external → External network (NAT চালু)        
- internal → Internal network (বেশি trust)     
- dmz      → DMZ zone                          
- work     → Office network                    
- home     → Home network                      
- trusted  → সব allow                          



### firewalld Basic Commands

```bash
# Status দেখা
sudo systemctl status firewalld

# Start/Stop
sudo systemctl start firewalld
sudo systemctl stop firewalld

# Enable on boot
sudo systemctl enable firewalld
```

---

### firewall-cmd - Main Tool

```bash
# Current rules দেখা
sudo firewall-cmd --list-all

# Output:
# public (active)
#   target: default
#   icmp-block-inversion: no
#   interfaces: eth0
#   sources:
#   services: dhcpv6-client ssh
#   ports:
#   protocols:
#   masquerade: no
#   forward-ports:
#   source-ports:
#   icmp-blocks:
#   rich rules:
```


### Service Allow/Remove করা

```bash
# SSH allow করা (permanent)
sudo firewall-cmd --permanent --add-service=ssh

# HTTP allow করা
sudo firewall-cmd --permanent --add-service=http

# HTTPS allow করা
sudo firewall-cmd --permanent --add-service=https

# Service remove করা
sudo firewall-cmd --permanent --remove-service=http

# Reload করা (changes apply হবে)
sudo firewall-cmd --reload
```

> **`--permanent`** ছাড়া দিলে reboot-এ হারিয়ে যাবে!

---

### Port Allow/Remove করা

```bash
# Port 8080 allow করা
sudo firewall-cmd --permanent --add-port=8080/tcp

# Port range allow করা
sudo firewall-cmd --permanent --add-port=8000-9000/tcp

# Port remove করা
sudo firewall-cmd --permanent --remove-port=8080/tcp

# Reload
sudo firewall-cmd --reload
```

---

### Specific IP Allow করা (Rich Rules)

```bash
# Specific IP থেকে SSH allow করা
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.50" service name="ssh" accept'

# Specific IP block করা
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" drop'

# Reload
sudo firewall-cmd --reload
```


### Zone Management

```bash
# Active zones দেখা
sudo firewall-cmd --get-active-zones

# Default zone দেখা
sudo firewall-cmd --get-default-zone

# Default zone change করা
sudo firewall-cmd --set-default-zone=public

# Interface কোন zone-এ আছে
sudo firewall-cmd --get-zone-of-interface=eth0

# Interface কে zone-এ assign করা
sudo firewall-cmd --permanent --zone=public --add-interface=eth0
```


## তিনটা Tool-এর তুলনা

| Feature | iptables | ufw | firewalld |
|---|---|---|---|
| সহজ কিনা | ❌ Complex | ✅ সহজ | ✅ মাঝামাঝি |
| Distro | সব | Ubuntu/Debian | CentOS/RHEL |
| Persistent rules | Manual | Auto | Auto |
| Zone support | ❌ | ❌ | ✅ |
| DevOps use | Low-level scripting | Quick setup | Enterprise |
| Reload without disconnect | ❌ | ✅ | ✅ |


## Real-World DevOps Scenario

### Web Server Firewall Setup (UFW দিয়ে)

```bash
# Default deny
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH allow (সবার আগে!)
sudo ufw allow 22/tcp

# Web traffic allow
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Monitoring port (শুধু internal network থেকে)
sudo ufw allow from 10.0.0.0/8 to any port 9100

# Enable
sudo ufw enable

# Verify
sudo ufw status verbose
```

### iptables দিয়ে একই কাজ

```bash
# Loopback allow (নিজের সাথে কথা বলার জন্য)
sudo iptables -A INPUT -i lo -j ACCEPT

# Established connections allow
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH allow
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# HTTP/HTTPS allow
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# বাকি সব DROP
sudo iptables -P INPUT DROP
```


## 📝 Quick Summary

- **Firewall** = network traffic-এর security guard
- **iptables** = low-level, powerful, সব distro-তে চলে - kernel-এর Netfilter control করে
- **Chains**: INPUT (ঢোকা), OUTPUT (বের হওয়া), FORWARD (pass করা)
- **Targets**: ACCEPT, DROP (চুপচাপ), REJECT (জানিয়ে দেয়)
- **ufw** = Ubuntu-তে iptables-এর সহজ wrapper
- **firewalld** = CentOS/RHEL-এ zone-based modern firewall
- সবসময় SSH allow করার **পরে** default DROP set করুন
- `--permanent` ছাড়া firewalld rules reboot-এ যাবে


## 🏋️ Practice Tasks

**Task 1 (ufw):**
```
Ubuntu system-এ ufw দিয়ে এই setup করুন:
- Default: সব incoming block
- SSH allow করুন
- HTTP ও HTTPS allow করুন
- আপনার নিজের IP থেকে port 3000 allow করুন
- Status verify করুন
```

**Task 2 (iptables):**
```
iptables দিয়ে:
- Port 22 allow করুন
- Port 80 allow করুন
- একটা নির্দিষ্ট IP (যেকোনো একটা নাও) block করুন
- সব rules list করুন line number সহ
- একটা rule delete করুন
```

**Task 3 (Troubleshooting):**
```
আপনি একটা web server setup করেছো port 8080-এ,
কিন্তু বাইরে থেকে access হচ্ছে না।
ufw অথবা iptables দিয়ে কীভাবে fix করবে?
```


## ⏭️ What's Next

**Chapter 4 - Lesson 9: Network Packet Analysis (tcpdump basics)**

পরের lesson-এ শিখবো কীভাবে network-এর **actual packets** দেখতে হয় - real-time-এ কী data যাচ্ছে-আসছে তা capture ও analyze করা। DevOps এবং security debugging-এর জন্য অনেক কাজের skill! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../07-SSH-Mastery">← SSH Mastery</a>
    </td>
    <td align="right">
      <a href="../09-Network-Packet-Analysis">Network Packet Analysis →</a>
    </td>
  </tr>
</table>