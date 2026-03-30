# Chapter 4 - Lesson 9: Network Packet Analysis

**Chapter 4 | Lesson 9 of 11**


## 🎯 এই Lesson-এ কী শিখবো?

আজকে আমরা শিখবো **tcpdump** - Linux-এর সবচেয়ে শক্তিশালী network packet analysis tool। এটা দিয়ে আপনি literally network-এর ভেতর দিয়ে যাওয়া সব data "দেখতে" পারবেন।


## tcpdump কী?

কল্পনা করুন আপনি একটা ব্যস্ত highway-এর পাশে দাঁড়িয়ে আছেন এবং প্রতিটা গাড়ির number plate, কোথা থেকে আসছে, কোথায় যাচ্ছে - সব নোট করছেন।

`tcpdump` ঠিক এই কাজটাই করে। শুধু highway-এর বদলে এটা আপনার `network interface` monitor করে এবং প্রতিটা `packet` (data packet) কোথা থেকে আসছে, কোথায় যাচ্ছে সব দেখায়।


**Real World Use Case in DevOps:**

- আমার app কি সত্যিই database-এ request পাঠাচ্ছে?
- কোন IP আমার server-এ বারবার hit করছে?
- DNS request কি সঠিক server-এ যাচ্ছে?
- Kubernetes pod-এর মধ্যে traffic কীভাবে flow হচ্ছে?


## tcpdump Install করা

```bash
# Ubuntu/Debian
sudo apt install tcpdump -y

# CentOS/RHEL
sudo yum install tcpdump -y

# Version check
tcpdump --version
```

> ⚠️ **Note:** tcpdump চালাতে `root/sudo` লাগে কারণ এটা network interface directly access করে।


## Basic Syntax

```bash
tcpdump [options] [filter expression]
```

| Part | মানে |
|------|------|
| `options` | কীভাবে দেখাবে - verbose, save to file, etc. |
| `filter expression` | কোন packets দেখাবো - port, host, protocol |


## Command 1: সবচেয়ে Simple - সব packets দেখা

```bash
sudo tcpdump
```

**কী করে:** আপনার default network interface-এ আসা-যাওয়া সব packet দেখায়।

**Output দেখতে এরকম:**
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes

14:32:01.123456 IP 192.168.1.5.52341 > 8.8.8.8.53: UDP, length 32
14:32:01.124789 IP 8.8.8.8.53 > 192.168.1.5.52341: UDP, length 48
14:32:01.200001 IP 192.168.1.5.45678 > 93.184.216.34.80: Flags [S], seq 0
```

**tcpdump-এর response বোঝার চেস্টা করি:**
```
14:32:01.123456   IP   192.168.1.5.52341   >   8.8.8.8.53:   UDP, length 32
     ↑             ↑         ↑             ↑       ↑              ↑
  Timestamp   Protocol  Source IP:Port  দিকনির্দেশ Dest IP:Port  Type & Size
```

> ⚠️ Ctrl+C দিয়ে বন্ধ করতে পারেন, নইলে সব packet দেখাতেই থাকবে!


## Command 2: নির্দিষ্ট Interface দেখা (`-i`)

```bash
# কোন কোন interface আছে দেখাও
sudo tcpdump -D

# নির্দিষ্ট interface monitor করো
sudo tcpdump -i eth0

# সব interface একসাথে
sudo tcpdump -i any
```

**Output of `-D`:**
```
1.eth0 [Up, Running]
2.lo [Up, Running, Loopback]
3.docker0 [Up, Running]
4.any (Pseudo-device that captures on all interfaces)
```

**DevOps Use Case:** আপনার server-এ multiple network card থাকলে (public + private network) নির্দিষ্ট একটা monitor করতে পারবে।


## Command 3: নির্দিষ্ট সংখ্যক Packet ধরো (`-c`)

```bash
# মাত্র 10টা packet দেখাও তারপর বন্ধ হয়ে যাও
sudo tcpdump -i eth0 -c 10
```

**Output:**
```
...10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

> Production server-এ `-c` ব্যবহার করা best practice নইলে terminal flood হয়ে যাবে!


## Command 4: File-এ Save করো (`-w`) ও পড়ো (`-r`)

### Save করা:
```bash
# packets একটা file-এ save করো (.pcap format)
sudo tcpdump -i eth0 -c 100 -w /tmp/capture.pcap

# Save করা file থেকে পড়া
sudo tcpdump -r /tmp/capture.pcap
```

**এটা কেন দরকার DevOps-এ?**

- Production issue investigate করতে capture করে রাখুন
- Wireshark (GUI tool)-এ open করে visually analyze করুন
- Team-এর সাথে share করুন
- Later analysis-এর জন্য archive করুন


## Command 5: Filter করা - এটাই tcpdump-এর আসল শক্তি!

### 5.1 - নির্দিষ্ট Host-এর traffic দেখা

```bash
# একটা নির্দিষ্ট IP-এর সব traffic
sudo tcpdump -i eth0 host 192.168.1.100

# শুধু ঐ IP থেকে আসা traffic (source)
sudo tcpdump -i eth0 src host 192.168.1.100

# শুধু ঐ IP-তে যাওয়া traffic (destination)
sudo tcpdump -i eth0 dst host 192.168.1.100
```

**Real Example:**
```bash
sudo tcpdump -i eth0 host google.com -c 5
```
```
14:35:10.001 IP myserver > 142.250.190.14.443: Flags [S]
14:35:10.052 IP 142.250.190.14.443 > myserver: Flags [S.]
```


### 5.2 - নির্দিষ্ট Port-এর traffic দেখা

```bash
# Port 80 (HTTP) traffic
sudo tcpdump -i eth0 port 80

# Port 443 (HTTPS) traffic
sudo tcpdump -i eth0 port 443

# Port 22 (SSH) traffic
sudo tcpdump -i eth0 port 22

# Port 3306 (MySQL) traffic
sudo tcpdump -i eth0 port 3306

# নির্দিষ্ট direction থেকে port filter
sudo tcpdump -i eth0 dst port 80
```

**DevOps Use Case:**
```bash
# App কি সত্যিই database connect করছে? দেখা:
sudo tcpdump -i eth0 port 5432 -c 20   # PostgreSQL
sudo tcpdump -i eth0 port 6379 -c 20   # Redis
```


### 5.3 - নির্দিষ্ট Protocol দেখা

```bash
# শুধু TCP traffic
sudo tcpdump -i eth0 tcp

# শুধু UDP traffic
sudo tcpdump -i eth0 udp

# শুধু ICMP (ping) traffic
sudo tcpdump -i eth0 icmp

# শুধু DNS traffic (UDP port 53)
sudo tcpdump -i eth0 udp port 53
```

**Example - ping করলে দেখুন কী হয়:**
```bash
# Terminal 1-এ রান করুন:
sudo tcpdump -i eth0 icmp

# Terminal 2-এ ping করুন:
ping -c 3 8.8.8.8
```

**Output:**
```
14:40:01.001 IP myserver > 8.8.8.8: ICMP echo request, id 1234, seq 1
14:40:01.015 IP 8.8.8.8 > myserver: ICMP echo reply, id 1234, seq 1
```


### 5.4 - AND / OR / NOT দিয়ে Complex Filter

```bash
# AND (&&) - দুটো condition একসাথে
sudo tcpdump -i eth0 host 192.168.1.100 and port 80

# OR (||) - যেকোনো একটা
sudo tcpdump -i eth0 port 80 or port 443

# NOT (!) - বাদ দেখাও
sudo tcpdump -i eth0 not port 22

# Complex: 192.168.1.100 এর HTTP বা HTTPS, SSH বাদে
sudo tcpdump -i eth0 host 192.168.1.100 and \(port 80 or port 443\) and not port 22
```


## Command 6: Verbose Output - বিস্তারিত দেখা

```bash
# একটু বেশি detail
sudo tcpdump -v -i eth0 -c 5

# আরো বেশি detail
sudo tcpdump -vv -i eth0 -c 5

# সর্বোচ্চ detail
sudo tcpdump -vvv -i eth0 -c 5
```

**`-v` output example:**
```
14:42:00.001 IP (tos 0x0, ttl 64, id 12345, offset 0, flags [DF], proto TCP (6), length 60)
    192.168.1.5.45678 > 93.184.216.34.80: Flags [S], cksum 0x1234, seq 0, win 65535
```

এখন TTL, flags, checksum সব দেখা যাচ্ছে!


## Command 7: Packet-এর Content দেখা (`-A` এবং `-X`)

```bash
# ASCII format-এ packet content দেখুন (readable text)
sudo tcpdump -i eth0 -A port 80 -c 5

# Hex + ASCII উভয়ই দেখুন
sudo tcpdump -i eth0 -X port 80 -c 5
```

**`-A` Output Example (HTTP request):**
```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: curl/7.68.0
Accept: */*
```

> ⚠️ **Security Warning:** `-A` দিয়ে unencrypted traffic (HTTP, Telnet) এর passwords পর্যন্ত দেখা যায়! এজন্যই HTTPS ব্যবহার করা জরুরি।


## Command 8: IP না দেখিয়ে Hostname রিজোলভ করা বন্ধ করো (`-n`)

```bash
# Default - hostname resolve করার চেষ্টা করে (ধীর হয়)
sudo tcpdump -i eth0

# -n দিলে IP হিসেবেই দেখায় (দ্রুত)
sudo tcpdump -n -i eth0

# -nn দিলে port নামও resolve করে না (80 হিসেবে দেখায়, http নয়)
sudo tcpdump -nn -i eth0
```

> **Best Practice:** Production-এ সবসময় `-nn` ব্যবহার করা কারন এটা faster এবং cleaner output দেয়।


## Real DevOps Scenarios

### Scenario 1: App কি Database-এ Connect হচ্ছে?
```bash
# App server-এ বসে দেখুন PostgreSQL traffic আছে কিনা
sudo tcpdump -nn -i any port 5432 -c 20
```

### Scenario 2: কোন IP বারবার আমার server hit করছে?
```bash
# Port 80-তে আসা সব unique source IP দেখুন
sudo tcpdump -nn -i eth0 dst port 80 -c 200 | awk '{print $3}' | sort | uniq -c | sort -rn
```

### Scenario 3: DNS কি সঠিকভাবে কাজ করছে?
```bash
# DNS query ও response দেখুন
sudo tcpdump -nn -i eth0 udp port 53
```
```
14:50:01 IP 192.168.1.5.12345 > 8.8.8.8.53: A? google.com
14:50:01 IP 8.8.8.8.53 > 192.168.1.5.12345: A 142.250.190.14
```

### Scenario 4: SSH Brute Force Attack detect করুন
```bash
# Port 22-তে অনেক বেশি connection attempt দেখুন
sudo tcpdump -nn -i eth0 tcp dst port 22 and 'tcp[tcpflags] == tcp-syn'
```

### Scenario 5: Kubernetes Pod-to-Pod Communication Debug
```bash
# Kubernetes node-এ রান করতে পারেন, pod network interface monitor করুন
sudo tcpdump -nn -i cni0 host 10.244.1.15
```


## Important Options - Quick Reference Table

| Option | কী করে | Example |
|--------|--------|---------|
| `-i eth0` | Interface বেছে নাও | `tcpdump -i eth0` |
| `-c 10` | মাত্র 10 packet | `tcpdump -c 10` |
| `-n` | IP as-is দেখাও | `tcpdump -n` |
| `-nn` | IP ও Port as-is | `tcpdump -nn` |
| `-v` | Verbose/বিস্তারিত | `tcpdump -v` |
| `-A` | ASCII content | `tcpdump -A` |
| `-X` | Hex + ASCII | `tcpdump -X` |
| `-w file` | File-এ save | `tcpdump -w cap.pcap` |
| `-r file` | File থেকে পড়ো | `tcpdump -r cap.pcap` |
| `host IP` | নির্দিষ্ট host | `tcpdump host 1.2.3.4` |
| `port 80` | নির্দিষ্ট port | `tcpdump port 80` |
| `tcp/udp/icmp` | Protocol filter | `tcpdump icmp` |
| `not port 22` | এটা বাদ দাও | `tcpdump not port 22` |


## Most Useful One-Liners (মুখস্থ রাখুন!)

```bash
# 1. সব HTTP traffic দেখুন
sudo tcpdump -nn -i any port 80 -A

# 2. সব DNS query দেখুন
sudo tcpdump -nn -i any udp port 53

# 3. নির্দিষ্ট host-এর সব traffic capture করুন file-এ
sudo tcpdump -nn -i eth0 host 10.0.0.5 -w /tmp/debug.pcap

# 4. SSH ছাড়া সব traffic (SSH flood এড়াতে)
sudo tcpdump -nn -i eth0 not port 22

# 5. ICMP (ping) monitor
sudo tcpdump -nn -i eth0 icmp
```


## 📝 Quick Summary

- **tcpdump** হলো Linux-এর command-line packet analyzer
- Network interface দিয়ে যাওয়া সব packet capture করতে পারে
- `Filter expressions` দিয়ে নির্দিষ্ট host, port, protocol দেখা যায়
- `-w` দিয়ে save করুন, `-r` দিয়ে পরে পড়ুন
- `-nn` সবসময় ব্যবহার করুন - faster ও cleaner
- `-A` দিয়ে unencrypted packet-এর content দেখা যায়
- DevOps-এ debugging, security analysis, network troubleshooting-এ অপরিহার্য


## 🏋️ Practice Tasks

**Task 1:**
```
নিজের machine-এ tcpdump রান করুন এবং শুধু ICMP traffic capture করুন।
তারপর আরেকটা terminal থেকে `ping 8.8.8.8` করুন এবং দেখুন tcpdump কী দেখায়।
```

**Task 2:**
```
DNS traffic capture করুন (udp port 53)।
তারপর `nslookup google.com` রান করুন এবং দেখুন DNS query ও response
tcpdump-এ কীভাবে দেখা যাচ্ছে।
```

**Task 3:**
```
100টা packet capture করে একটা file-এ save করুন।
তারপর সেই file থেকে পড়ুন এবং শুধু TCP packets filter করুন।
```


## ⏭️ What's Next

**Lesson 10: /etc/hosts, /etc/resolv.conf, /etc/network/interfaces - Deep Dive**

> এই lesson-এ আমরা শিখবো Linux-এর সবচেয়ে গুরুত্বপূর্ণ network configuration files - কোথায় hostname mapping হয়, DNS কীভাবে configure হয়, এবং network interface কীভাবে সেটআপ করতে হয়। DevOps-এ এই files না জানলে অনেক কিছুই অন্ধকারে থাকে! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../08-Firewall-Management">← Firewall Management</a>
    </td>
    <td align="right">
      <a href="../10-Network-File-Configuration">Networking File Configuration →</a>
    </td>
  </tr>
</table>