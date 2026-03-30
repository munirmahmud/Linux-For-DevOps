# Chapter 4 - Lesson 1: Networking Basics Recap

**Chapter 4 | Lesson 1 of 11**


আপনাকে অভিনন্দন 🎉! আপনি Chapter 4-এ পৌঁছে গেছেন। এটা DevOps Engineer-দের জন্য অত্যন্ত গুরুত্বপূর্ণ একটি Chapter। Server deploy করতে হোক, troubleshoot করতে হোক, বা container network বুঝতে হোক networking ছাড়া কিছুই সম্ভব না।

এই Lesson-এ আমরা networking-এর foundational concepts রিফ্রেশ করব যেগুলো না বুঝলে পরের সব lesson কঠিন লাগবে।

এই লেসনে আমারা IP, MAC, DNS, Gateway, Ports ইত্যাদি বিষয় শিখবো।



আমরা Internet কে কল্পনা করতে পারি একটি বিশাল শহরের মতো।

| Real World | Networking Concept |
|---|---|
| প্রতিটি বাড়ির ঠিকানা | **IP Address** |
| বাড়ির দরজার unique serial number | **MAC Address** |
| শহরের phone book / directory | **DNS** |
| শহরের মূল প্রবেশদ্বার | **Gateway** |
| বাড়ির আলাদা আলাদা ঘর | **Ports** |


## ১) IP Address কী?

**IP = Internet Protocol Address**

এটা হলো আপনার device-এর unique পরিচয়পত্র। network-এ প্রতিটি device-কে আলাদাভাবে চেনার জন্য।

### IP Address-এর Classification

#### IPv4 (সবচেয়ে বেশি ব্যবহৃত)
```
192.168.1.10
```
- ৪টি অংশ, প্রতিটি অংশ **0 থেকে 255** এর মধ্যে
- মোট **4.3 billion** unique address সম্ভব
- কিন্তু internet device এত বেশি হয়ে গেছে যে IPv4 শেষ হয়ে যাচ্ছে!

#### IPv6 (নতুন, বড়)
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```
- অনেক বড় practically অসীম address সম্ভব
- ধীরে ধীরে সবাই IPv6-এ যাচ্ছে


### IP Address-এর প্রকারভেদ:

| ধরন | মানে | উদাহরণ |
|---|---|---|
| **Private IP** | শুধু আপনার local network-এ ব্যবহার হয় | `192.168.x.x`, `10.x.x.x`, `172.16.x.x` |
| **Public IP** | Internet-এ আপনার device-কে চেনায় | `103.45.67.89` (ISP দেয়) |

> 💡 **DevOps Tip:** আপনার server-এর private IP হলো `10.0.0.5` হতে পারে - কিন্তু বাইরে থেকে আপনার কোম্পানির public IP দিয়ে access করতে হবে।

### Subnet Mask কী?

IP Address-এর সাথে সবসময় একটা **Subnet Mask** থাকে।

```
IP:      192.168.1.10
Subnet:  255.255.255.0   (বা /24 notation-এ)
```

এটা বলে দেয়: এই network-এ কোন কোন IP গুলো আমার পরিবার (same network)?

`/24` মানে হলো `192.168.1.0` থেকে `192.168.1.255` পর্যন্ত সবাই same network-এ আছে। মোট **256টি** address (254টি usable)।

## MAC Address কী?

**MAC = Media Access Control Address**

এটা হলো আপনার **Network Interface Card (NIC)**-এর hardware address যা factory থেকে বানানোর সময় দেওয়া হয়।

```
e4:5f:01:ab:cd:ef
```

- ৬টি অংশ, hexadecimal format
- প্রতিটি device-এর MAC address unique (theoretically)
- শুধু same local network-এর ভেতরে কাজ করে

> IP Address হলো আপনার **বর্তমান বাড়ির ঠিকানা** (বদলাতে পারে), কিন্তু MAC Address হলো আপনার **জাতীয় পরিচয়পত্র নম্বর** (hardware-এ fixed)।

### IP vs MAC - কখন কোনটা ব্যবহার হয়?

| পরিস্থিতি | কোনটা ব্যবহার হয় |
|---|---|
| Same network-এ packet পাঠানো | **MAC Address** |
| Different network-এ packet পাঠানো | **IP Address** |
| ARP - Address Resolution Protocol | IP → MAC খোঁজার কাজ করে |


## DNS কী?

**DNS = Domain Name System**

মানুষ নাম মনে রাখতে পারে, কিন্তু IP মনে রাখা কঠিন। DNS হলো *internet-এর phone book*।

```
google.com  →  DNS জিজ্ঞেস করো  →  142.250.190.46
```

### DNS কীভাবে কাজ করে? (Step by step)

```
আপনি browser-এ টাইপ করলেন: google.com

1. আপনার computer জিজ্ঞেস করে: "google.com এর IP জানো?"
         ↓
2. Local Cache চেক করে (আগে কখনো গিয়েছিলে?)
         ↓
3. /etc/resolv.conf - Configured DNS Server-কে জিজ্ঞেস করে
   (যেমন: Google DNS 8.8.8.8, বা Cloudflare 1.1.1.1)
         ↓
4. DNS Server জবাব দেয়: "142.250.190.46"
         ↓
5. আপনার browser সেই IP-তে connect করে
```

### গুরুত্বপূর্ণ DNS Records:

| Record Type | মানে | উদাহরণ |
|---|---|---|
| **A** | Domain → IPv4 | `google.com → 142.250.190.46` |
| **AAAA** | Domain → IPv6 | `google.com → 2607:f8b0:...` |
| **CNAME** | এক domain কে অন্যটায় point করা | `www.google.com → google.com` |
| **MX** | Mail server | `gmail.com → aspmx.l.google.com` |
| **NS** | Name Server | কোন server এই domain manage করে |
| **PTR** | IP → Domain (reverse) | `142.250.190.46 → google.com` |
| **TXT** | Text info | SPF, DKIM records |

> 💡 **DevOps Tip:** যখন নতুন server deploy করেন বা domain configure করেন তখন DNS records (A, CNAME, MX) সেট করতে হয়। এটা না জানলে production issue হবে!


## Default Gateway কী?

Gateway হলো আপনার network-এর দরজা। যে দরজা দিয়ে বাইরের দুনিয়ায় যাওয়া যায়।

```
আপনার PC IP:     192.168.1.10
Gateway IP:      192.168.1.1   ← এটা সাধারণত আপনার router IP
```

### কীভাবে কাজ করে?

```
আপনি google.com-এ যেতে চান (8.8.8.8)

1. আপনার computer বুঝলো: 8.8.8.8 আমার network-এ নেই
2. তাই packet পাঠালো Gateway-তে (192.168.1.1)
3. Gateway (Router) সেটা forward করলো internet-এ
4. জবাব আবার Gateway হয়ে আপনার কাছে ফিরে এলো
```

> Gateway হলো আপনার মহল্লার মূল গেট। ভেতরে ভেতরে যাতায়াত (same network) গেট ছাড়াই হয়। বাইরে যেতে হলে গেট দিয়েই যেতে হবে।

## Ports কী?

IP Address দিয়ে কোন machine তা বোঝা যায়। কিন্তু সেই machine-এর কোন service/application-এ যাবে সেটাকে বলে Port Number।

```
192.168.1.10:80   ← HTTP web server
192.168.1.10:443  ← HTTPS web server
192.168.1.10:22   ← SSH
192.168.1.10:3306 ← MySQL database
```

> IP Address হলো building-এর ঠিকানা, Port হলো সেই building-এর **ঘর নম্বর**।

### সবচেয়ে গুরুত্বপূর্ণ Well-Known Ports (DevOps-এ সবসময় দরকার):

| Port | Protocol/Service | কোথায় ব্যবহার হয় |
|---|---|---|
| **22** | SSH | Server-এ remote login |
| **80** | HTTP | Web traffic (plain) |
| **443** | HTTPS | Web traffic (encrypted) |
| **21** | FTP | File transfer |
| **25** | SMTP | Email sending |
| **53** | DNS | Domain name resolution |
| **3306** | MySQL | Database connection |
| **5432** | PostgreSQL | Database connection |
| **6379** | Redis | Cache/message broker |
| **8080** | HTTP alt | Development web server |
| **27017** | MongoDB | NoSQL database |
| **2181** | Zookeeper | Kafka coordination |
| **9092** | Kafka | Message streaming |

### Port Range:
```
0 - 1023      → Well-known ports (system use করে, root লাগে)
1024 - 49151  → Registered ports (applications ব্যবহার করে)
49152 - 65535 → Dynamic/Ephemeral ports (temporary connections)
```


## Protocol কী? TCP vs UDP

Data পাঠানোর দুটো প্রধান পদ্ধতি আছে:

### TCP (Transmission Control Protocol)
- **Reliable** - packet পৌঁছেছে কিনা নিশ্চিত করে
- **Ordered** - সঠিক ক্রমে পৌঁছায়
- **Slow** - কিন্তু নির্ভরযোগ্য
- ব্যবহার: HTTP, HTTPS, SSH, FTP, Email

> TCP হলো Registered Post - রশিদ নেওয়া হয়, নিশ্চিত হওয়া যায় চিঠি পৌঁছেছে।

### UDP (User Datagram Protocol)
- **Fast** - কিন্তু কোনো নিশ্চয়তা নেই
- **Unreliable** - packet হারিয়ে যেতে পারে
- ব্যবহার: DNS, Video streaming, Gaming, VoIP

> UDP হলো সাধারণ চিঠি - দ্রুত পাঠানো যায়, কিন্তু পৌঁছেছে কিনা জানা নেই।


## Complete Picture - একটি request কীভাবে কাজ করে?

```
আপনি browser-এ টাইপ করলেন: https://github.com

Step 1: DNS Resolution
        github.com → 140.82.121.3 (DNS server থেকে)

Step 2: Gateway Decision
        140.82.121.3 আমার network-এ নেই
        তাই packet → Gateway (192.168.1.1) → Internet

Step 3: TCP Connection (3-way handshake)
        আপনার PC → SYN → github.com:443
        github.com → SYN-ACK → আপনার PC
        আপনার PC → ACK → github.com
        (connection established!)

Step 4: Data Transfer
        HTTPS request পাঠানো → HTML response পাওয়া

Step 5: Browser render করলো GitHub homepage
```


## Linux Networking-এ গুরুত্বপূর্ণ Files

| File | কী কাজ করে |
|---|---|
| `/etc/hosts` | Local DNS - domain → IP manually map করা |
| `/etc/resolv.conf` | কোন DNS server ব্যবহার করবে |
| `/etc/network/interfaces` | Network interface configuration (Debian/Ubuntu) |
| `/etc/sysconfig/network-scripts/` | Network config (RHEL/CentOS) |
| `/proc/net/` | Kernel-এর network information |


## 📝 Quick Summary

- **IP Address** - network-এ device-এর unique পরিচয় (IPv4 বা IPv6)
- **Private IP** - local network-এ ব্যবহার হয়
- **Public IP** - internet-এ যেতে ব্যবহার হয়
- **Subnet Mask** - কোন IP গুলো same network-এ আছে তা বলে
- **MAC Address** - hardware-এর fixed unique identifier, local network-এ ব্যবহার
- **DNS** - domain name কে IP-তে convert করে (internet-এর phone book)
- **Gateway** - অন্য network-এ যাওয়ার দরজা (সাধারণত router)
- **Port** - একই machine-এর কোন service-এ যাবে তা বলে
- **TCP** - reliable কিন্তু slow | **UDP** - fast কিন্তু unreliable


## 🏋️ Practice Tasks

**Task 1:** আপনার নিজের machine-এ IP address, Gateway, এবং DNS দেখুন:
```bash
ip addr show          # IP address দেখাও
ip route show         # Gateway দেখাও
cat /etc/resolv.conf  # DNS server দেখাও
```

**Task 2:** নিচের commands রান করুন এবং output বোঝার চেষ্টা করুন:
```bash
ping -c 4 8.8.8.8        # Google DNS ping করা
ping -c 4 google.com     # Domain দিয়ে ping করা - দেখুন DNS কাজ করছে কিনা
```

**Task 3:** `/etc/hosts` file-টি দেখুন:
```bash
cat /etc/hosts
```
এই file-এ কী আছে? `127.0.0.1 localhost` মানে কী বলতে পারেন?

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 2: Viewing Network Configuration**

`ip`, `ifconfig`, `nmcli`, `hostname` Linux-এ network configuration কীভাবে দেখতে হয় এবং পরিবর্তন করতে হয়, সেটা হাতে-কলমে শিখবো! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../../Chapter-03/11-Assessment/">← Chapter 3 - Assessment</a>
    </td>
    <td align="right">
      <a href="../02-Viewing-Network-Configuration">Viewing Network Configuration →</a>
    </td>
  </tr>
</table>