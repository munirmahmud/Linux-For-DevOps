# Chapter 4 - Lesson 6: Open Ports & Sockets 

**Chapter 4 | Lesson 6 of 11**

এই লেসনে আমরা শিখবো `ss`, `netstat`, `lsof`, `nmap` এর বিস্তারিত



## 🎯 এই Lesson-এ কী শিখবো?

একটা সার্ভারে কোন কোন port খোলা আছে, কোন process কোন port ব্যবহার করছে, কোন connection active আছে - এই সব জানাটা একজন DevOps Engineer-এর জন্য অত্যন্ত জরুরি।

আজকে শিখবো:
- `ss` - modern socket statistics tool
- `netstat` - classic network statistics tool
- `lsof` - list open files (ports সহ!)
- `nmap` - network port scanner


## Socket কী?

> একটা বাড়ির অনেকগুলো দরজা আছে। প্রতিটা দরজার নম্বর হলো **port number**। যে দরজা দিয়ে কেউ ঢুকছে বা বেরোচ্ছে, সেটাই একটা **socket connection**।

একটা **Socket** হলো:

```
IP Address + Port Number + Protocol (TCP/UDP)
```

উদাহরণ:
```
192.168.1.10:80   → কেউ এই machine-এর port 80-তে connect করেছে (HTTP)
0.0.0.0:22        → SSH সব interface-এ listen করছে
127.0.0.1:3306    → MySQL শুধু localhost-এ listen করছে
```


## Connection State-এর কী মানে?

DevOps কাজে এই states গুলো প্রায়ই দেখবেন:

| State | মানে |
|---|---|
| `LISTEN` | Port খোলা আছে, connection-এর জন্য অপেক্ষা করছে |
| ` | Connection সক্রিয় আছে, data আদান-প্রদান চলছে |
| `TIME_WAIT` | Connection শেষ হয়েছে, cleanup চলছে |
| `CLOSE_WAIT` | Remote side connection বন্ধ করেছে, local side এখনো করেনি |
| `SYN_SENT` | Connection করার চেষ্টা চলছে |


## Tool 1: `ss` (Socket Statistics)

`ss` হলো সবচেয়ে modern ও fast tool Linux-এ socket/port দেখার জন্য। এটা `netstat`-এর replacement।

### Basic Syntax:
```bash
ss [options]
```


### সবচেয়ে দরকারি `ss` commands:

#### সব listening port দেখা
```bash
ss -tlpn
```

**প্রতিটা flag-এর মানে:**

| Flag | মানে |
|---|---|
| `-t` | TCP connections দেখাও |
| `-l` | শুধু LISTEN state দেখাও |
| `-n` | Numeric port (hostname resolve করবে না) |
| `-p` | কোন process ব্যবহার করছে দেখাও |

**Expected Output:**
```
State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22            0.0.0.0:*          users:(("sshd",pid=1234,fd=3))
LISTEN  0       80      0.0.0.0:80            0.0.0.0:*          users:(("nginx",pid=5678,fd=6))
LISTEN  0       128     127.0.0.1:3306        0.0.0.0:*          users:(("mysqld",pid=910,fd=19))
```

এখানে দেখুন:
- Port **22** → `sshd` ব্যবহার করছে (SSH)
- Port **80** → `nginx` ব্যবহার করছে (Web Server)
- Port **3306** → `mysqld` ব্যবহার করছে (MySQL), শুধু localhost-এ


#### সব active TCP connection দেখা
```bash
ss -tn
```
```
State        Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
       0       192.168.1.5:22      203.0.113.10:54321
TIME_WAIT    0       0       192.168.1.5:80      198.51.100.1:43210
```

এখানে দেখতে পাচ্ছেন কে SSH-এ connected আছে।


#### UDP ports দেখা
```bash
ss -ulnp
```
(`-u` মানে UDP)


#### TCP + UDP সব একসাথে দেখা
```bash
ss -tulpn
```


#### একটা specific port কে ব্যবহার করছে?
```bash
ss -tlpn | grep :80
```
```
LISTEN  0  128  0.0.0.0:80  0.0.0.0:*  users:(("nginx",pid=5678,fd=6))
```


#### একটা specific process-এর connections দেখা
```bash
ss -tp | grep nginx
```


#### Established connections count করা
```bash
ss -tn | grep ESTAB | wc -l
```
> এই command টা দিয়ে বুঝতে পারবেন এই মুহূর্তে কতজন user connected আছে।


#### Socket statistics summary দেখা
```bash
ss -s
```
```
Total: 156
TCP:   12 (estab 3, closed 4, orphaned 0, timewait 2)
UDP:   5
```


## Tool 2: `netstat` (Network Statistics - Classic Tool)

`netstat` পুরনো কিন্তু এখনো অনেক server-এ ব্যবহার হয়। এটা `net-tools` package-এর অংশ।

### Install করুন (যদি না থাকে)

```bash
# Ubuntu/Debian
sudo apt install net-tools

# RHEL/Fedora
sudo yum install net-tools
```


### দরকারি `netstat` commands:

#### সব listening ports দেখা

```bash
netstat -tlnp
```

(`ss -tlnp`-এর মতোই কাজ করে)

**Output:**
```
Proto  Recv-Q  Send-Q  Local Address    Foreign Address  State   PID/Program name
tcp    0       0       0.0.0.0:22       0.0.0.0:*        LISTEN  1234/sshd
tcp    0       0       0.0.0.0:80       0.0.0.0:*        LISTEN  5678/nginx
tcp    0       0       127.0.0.1:3306   0.0.0.0:*        LISTEN  910/mysqld
```


#### সব active connections দেখা

```bash
netstat -an
```


#### Specific port filter করা

```bash
netstat -tlnp | grep :443
```


#### Network statistics দেখা
```bash
netstat -s
```
> TCP, UDP, IP-এর detailed statistics দেখাবে।


#### Routing table দেখা

```bash
netstat -rn
```
```
Destination   Gateway       Genmask       Flags  Iface
0.0.0.0       192.168.1.1   0.0.0.0       UG     eth0
192.168.1.0   0.0.0.0       255.255.255.0 U      eth0
```


### `ss` vs `netstat` - কোনটা ব্যবহার করবো?

| বিষয় | `ss` | `netstat` |
|---|---|---|
| Speed | ⚡ অনেক দ্রুত | ধীর |
| Modern Linux | ✅ Built-in | ❌ আলাদা install লাগে |
| Features | বেশি | কম |
| DevOps use | Preferred | Legacy support-এর জন্য |

> সবসময় `ss` ব্যবহার করুন। `netstat` শুধু তখন যখন পুরনো system বা script-এ দরকার।


## Tool 3: `lsof` (List Open Files)

> Linux-এ "সবকিছুই একটা file"। Network socket-ও একটা file। `lsof` দিয়ে দেখা যায় কোন process কোন file/socket/port খুলে রেখেছে।

### Install:
```bash
sudo apt install lsof   # Ubuntu/Debian
sudo yum install lsof   # RHEL/Fedora
```


### দরকারি `lsof` commands:

#### সব network connections দেখা
```bash
sudo lsof -i
```
```
COMMAND   PID   USER   FD   TYPE  DEVICE  SIZE  NODE  NAME
sshd      1234  root   3u   IPv4  12345   0t0   TCP   *:ssh (LISTEN)
nginx     5678  www    6u   IPv4  23456   0t0   TCP   *:http (LISTEN)
mysqld    910   mysql  19u  IPv4  34567   0t0   TCP   localhost:mysql (LISTEN)
```


#### Specific port কে ব্যবহার করছে?
```bash
sudo lsof -i :80
```
```
COMMAND  PID    USER  FD   TYPE  DEVICE  SIZE  NODE  NAME
nginx    5678   www   6u   IPv4  23456   0t0   TCP   *:http (LISTEN)
```

> **DevOps tip:** Port conflict হলে এই command দিয়ে সাথে সাথে বের করুন কোন process দোষী!


#### TCP connections
```bash
sudo lsof -i TCP
```


#### UDP connections
```bash
sudo lsof -i UDP
```


#### Specific process-এর open files/ports
```bash
sudo lsof -p 1234
```
(1234 হলো PID)


#### Specific user-এর open connections
```bash
sudo lsof -u www-data -i
```


#### একটা deleted file কোন process ধরে রেখেছে?
```bash
sudo lsof | grep deleted
```
> এটা disk space debug করার সময় কাজে লাগে - log file delete হয়েছে কিন্তু process ধরে রেখেছে।


## Tool 4: `nmap` (Network Mapper - Port Scanner)

`nmap` হলো সবচেয়ে powerful **port scanner**। এটা দিয়ে:
- নিজের server-এর open ports check করুন
- Network-এ কোন devices আছে দেখুন
- Security audit করুন

> ⚠️ **সতর্কতা:** অন্যের server বা network scan করা **illegal** হতে পারে। শুধু নিজের system বা অনুমতি পাওয়া system-এ ব্যবহার করুন।

### Install:
```bash
sudo apt install nmap    # Ubuntu
sudo yum install nmap    # CentOS
```


### দরকারি `nmap` commands:

#### নিজের localhost scan করুন
```bash
nmap localhost
```
```
Starting Nmap 7.80
Nmap scan report for localhost (127.0.0.1)
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
3306/tcp open   mysql
```


#### Specific IP scan করুন
```bash
nmap 192.168.1.10
```


#### Service version দেখুন
```bash
nmap -sV localhost
```
```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 8.9p1
80/tcp  open   http     nginx 1.22.0
```


#### Specific port range scan করুন
```bash
nmap -p 1-1000 localhost
```


#### Specific ports check করুন
```bash
nmap -p 22,80,443,3306 192.168.1.10
```


#### সব ports scan করুন (সময় লাগবে)
```bash
nmap -p- localhost
```


#### OS detection করুন
```bash
sudo nmap -O 192.168.1.10
```


#### Network-এ সব active hosts দেখুন
```bash
nmap -sn 192.168.1.0/24
```
```
Nmap scan report for 192.168.1.1  → Router
Nmap scan report for 192.168.1.5  → আপনার server
Nmap scan report for 192.168.1.10 → অন্য device
```


#### UDP scan করুন
```bash
sudo nmap -sU -p 53,67,68,123 localhost
```


## Real-World DevOps Scenarios

### Scenario 1: 

Port 8080 already in use error

```bash
# কোন process 8080 ব্যবহার করছে?
ss -tlnp | grep :8080
# অথবা
sudo lsof -i :8080

# Process kill করুন
sudo kill -9 <PID>
```


### Scenario 2:

MySQL কি শুধু localhost-এ নাকি সব interface-এ?

```bash
ss -tlnp | grep :3306
```
- `127.0.0.1:3306` → শুধু localhost ✅ নিরাপদ
- `0.0.0.0:3306` → সবার কাছে exposed ⚠️ বিপজ্জনক!


### Scenario 3: 

Server-এ কতজন SSH connected?

```bash
ss -tn | grep :22 | grep 
```

### Scenario 4: 

Firewall rule দেওয়ার আগে open ports audit

```bash
sudo nmap -sV -p- localhost > open_ports_audit.txt
cat open_ports_audit.txt
```


### Scenario 5: 

একটা app deploy করার পরে port সঠিকভাবে খুলেছে কিনা?

```bash
ss -tlnp | grep :3000    # Node.js app
# অথবা
nmap -p 3000 localhost
```


## সব Tools-এর তুলনামূলক চিত্র

| কাজ | Best Tool |
|---|---|
| নিজের server-এর listening ports | `ss -tlnp` |
| Port কোন process ব্যবহার করছে | `ss -tlnp` বা `lsof -i :PORT` |
| Active connections দেখুন | `ss -tn` |
| Deleted file কোন process ধরে আছে | `lsof \| grep deleted` |
| Remote server port check | `nmap` |
| Network-এ কোন hosts আছে | `nmap -sn` |
| Legacy script compatibility | `netstat` |


## 📝 Quick Summary

- **`ss`** → Modern, fast tool। Linux-এ সব socket/port দেখার জন্য সেরা। `ss -tlnp` সবচেয়ে বেশি ব্যবহার হয়।
- **`netstat`** → পুরনো classic tool। এখনো অনেক জায়গায় দেখা যায়। `ss`-এর আগে এটাই ছিল।
- **`lsof`** → Open files ও ports দেখে। Port conflict debug করার জন্য অসাধারণ।
- **`nmap`** → Port scanner। নিজের server audit ও security check-এর জন্য।


## 🏋️ Practice Tasks

> এই tasks গুলো নিজে করে দেখুন:

**Task 1:**
```bash
ss -tlnp
```
আপনার system-এ কোন কোন port LISTEN করছে লিখে রাখুন। SSH কোন port-এ আছে?

**Task 2:**
```bash
sudo lsof -i :22
```
SSH port-এ কোন process আছে? PID কত? Output টা share করুন।

**Task 3:**
```bash
nmap -sV localhost
```

আপনার localhost-এ কোন কোন service চলছে এবং তাদের version কত?

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 7: SSH Mastery**

> SSH key generation, ssh-agent, SSH config file, port forwarding এবং tunneling - DevOps-এর সবচেয়ে গুরুত্বপূর্ণ skill গুলোর একটা! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../05-Downloading-And-Transferring-Files">← Downloading &amp;  Transferring Files</a>
    </td>
    <td align="right">
      <a href="../07-SSH-Mastery">SSH Mastery →</a>
    </td>
  </tr>
</table>