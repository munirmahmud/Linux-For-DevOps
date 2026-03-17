# Chapter 4 - Lesson 1: Networking Basics Recap

**Chapter 4 | Lesson 1 of 11**

আপনাকে অভিনন্দন! 🎉 আপনি Chapter 4-এ পৌঁছে গেছেন। এটা DevOps Engineer-দের জন্য অত্যন্ত গুরুত্বপূর্ণ একটি Chapter। Server deploy করতে হোক, troubleshoot করতে হোক, বা container network বুঝতে হোক - networking ছাড়া কিছুই সম্ভব না।

এই Lesson-এ আমরা networking-এর foundational concepts রিফ্রেশ করব যেগুলো না বুঝলে পরের সব lesson কঠিন লাগবে।

এই লেসনে আমারা IP, MAC, DNS, Gateway, Ports ইত্যাদি বিষয় শিখবো।

## বাস্তব জীবনের Analogy দিয়ে শুরু করি

আমরা Internet কে কল্পনা করতে পারি একটি বিশাল শহরের মতো।

| Real World | Networking Concept |
|---|---|
| প্রতিটি বাড়ির ঠিকানা | **IP Address** |
| বাড়ির দরজার unique serial number | **MAC Address** |
| শহরের phone book / directory | **DNS** |
| শহরের মূল প্রবেশদ্বার | **Gateway** |
| বাড়ির আলাদা আলাদা ঘর | **Ports** |



## IP Address-এর বিস্তারিত

IP address-কে দুটি ভিন্ন দৃষ্টিকোণ থেকে ভাগ করা হয়:


### ১. Version অনুযায়ী → IP Version

| Version | নাম | উদাহরণ |
|---|---|---|
| IPv4 | Internet Protocol version 4 | `192.168.1.1` |
| IPv6 | Internet Protocol version 6 | `2001:0db8::1` |

এটাকে বলে **"IP Versioning"** বা **"IP Protocol Version Classification"**।


### ২. Scope/Use অনুযায়ী → **IP Address Types**
Public, Private, Loopback - এগুলো হলো IP-এর **"Types" বা "Classes by scope"**:

| Type | কী কাজ | উদাহরণ |
|---|---|---|
| **Public IP** | Internet-এ চেনা যায় | `8.8.8.8` |
| **Private IP** | শুধু Local network-এ | `192.168.x.x`, `10.x.x.x` |
| **Loopback IP** | নিজেকে নিজে ping করতে | `127.0.0.1` |
| **APIPA** | DHCP না পেলে auto-assign | `169.254.x.x` |

এটাকে বলে **"IP Address Scope Classification"** বা **"Types of IP Address"**।


## সহজ সারসংক্ষেপ

```
IP Address
├── Version অনুযায়ী → IPv4 / IPv6
└── Scope অনুযায়ী  → Public / Private / Loopback / APIPA
```

অর্থাৎ -
- **IPv4 vs IPv6** → এটা *কোন version* তা বলে
- **Public/Private/Loopback** → এটা *কোথায় ব্যবহার হয়* তা বলে

একটি IP যেমন `192.168.1.1` একই সাথে **IPv4** (version) এবং **Private** (scope) - দুটোই!


## Networking Address

নেটওয়ার্কিংয়ে আমরা মূলত **দুই ধরনের অ্যাড্রেস** ব্যবহার করি:

### 1️⃣ Logical Address

Logical Address-কে **L3 Address / IP Address / Network Address** ও বলে।

| ধরন | বিট সংখ্যা | উদাহরণ |
|------|-----------|--------|
| IPv4 | 32 Bits | 192.168.1.1 |
| IPv6 | 128 Bits | 2001:db8::1 |


### 2️⃣ Physical Address

Logical Address-কে **L2 Address / MAC Address / Ethernet Address** ও বলে।

- প্রতিটি **Network Interface Card (NIC)**-এর সাথে একটি **ইউনিক MAC Address** থাকে।
- এটি hardware-এ স্থায়ীভাবে বার্ন করা থাকে।

### 3️⃣ Loopback Address

- যেকোনো **OS**-এর সাথে একটি **Loopback Address** থাকে।
- এটি OS নিজে নিজেই পরীক্ষার জন্য ব্যবহার করে, কোনো physical network card লাগে না।

**মনে রাখুন:**
> - **OS** থাকলে → **Loopback Address** থাকবেই
> - **Network Card (NIC)** থাকলে → **MAC Address** থাকবেই
> - **Network Communication** করতে চাইলে → **IP Address** লাগবে


## IPv4 Address - বিস্তারিত

```
Format   : 32 Bits (4 Bytes)
Structure: 4 Octets  →  1st.2nd.3rd.4th
Separator: Dot (.)
Number   : Decimal (0–255 প্রতিটি Octet-এ)
Example  : 203.112.194.243
```

- **Binary আকারে:** `11001011.01110000.11000010.11110011`
- **Total Addresses:** 2³² = **4,294,967,296** (প্রায় ৪.৩ বিলিয়ন)
- প্রতিটি Octet সর্বোচ্চ মান: 2⁸ - 1 = **255** (range: 0–255)

**গুরুত্বপূর্ণ নিয়ম:**
> - IPv4-এর **1st Octet** কখনোই 0 হবে না
> - IPv4-এর যেকোনো Octet-এর সর্বোচ্চ মান **255**


## IPv4 Classification (ক্লাসিফিকেশন)

IP Address কোন class-এ পড়বে তা নির্ধারণ হয় **1st Octet** দেখে:

| Class | 1st Octet Range | ব্যবহার |
|-------|----------------|---------|
| **Class A** | 0 – 127 | LAN, MAN, WAN (Large Networks) |
| **Class B** | 128 – 191 | LAN, MAN, WAN (Medium Networks) |
| **Class C** | 192 – 223 | LAN, MAN, WAN (Small Networks) |
| **Class D** | 224 – 239 | Multicast, Routing Protocols |
| **Class E** | 240 – 255 | Reserved / Experimental |

> LAN, MAN, WAN নেটওয়ার্কে **Class A, B, C** ব্যবহার হয়।


## LAN - Local Area Network

LAN মানে হলো একটি ছোট এলাকার মধ্যে সীমাবদ্ধ নেটওয়ার্ক। একটা বাসার সবগুলো ডিভাইস যেমন laptop, phone, smart TV ইত্যাদি যখন একই WiFi router-এর সাথে connected থাকে, এটাই LAN।

**উদাহরণ:**
- আপনার বাসার WiFi network
- একটি অফিসের সব computer একসাথে connected
- একটি স্কুলের computer lab

**বৈশিষ্ট্য:**
- আকার ছোট যেমন একটি ঘর, একটি বিল্ডিং, একটি ক্যাম্পাস
- Speed অনেক বেশি (100 Mbps – 10 Gbps)
- খরচ কম
- Private IP ব্যবহার হয় (192.168.x.x)


## MAN - Metropolitan Area Network

MAN মানে হলো একটি শহর বা বড় এলাকা জুড়ে বিস্তৃত নেটওয়ার্ক। ঢাকা শহরের বিভিন্ন এলাকায় একটি ব্যাংকের অনেক শাখা আছে। গুলশান শাখা, মতিঝিল শাখা, ধানমন্ডি শাখা - সবগুলো একে অপরের সাথে connected। এটাই MAN।

**উদাহরণ:**
- একটি ব্যাংকের শহরের সব শাখার নেটওয়ার্ক
- একটি বিশ্ববিদ্যালয়ের বিভিন্ন ক্যাম্পাস (ঢাকা ও চট্টগ্রাম ক্যাম্পাস)
- সরকারি অফিসগুলোর city-wide নেটওয়ার্ক

**বৈশিষ্ট্য:**
- আকার মাঝারি - একটি শহর বা জেলা
- LAN-এর চেয়ে বড়, WAN-এর চেয়ে ছোট
- ISP বা সরকার manage করে


## WAN - Wide Area Network

WAN মানে হলো দেশ বা মহাদেশ পেরিয়ে বিশাল এলাকা জুড়ে বিস্তৃত নেটওয়ার্ক। আপনি ঢাকায় বসে আমেরিকার একটি website ব্রাউজ করছেন - এটা possible হচ্ছে WAN-এর কারণে। ইন্টারনেট নিজেই একটি WAN।

**উদাহরণ:**
- ইন্টারনেট (সবচেয়ে বড় WAN)
- একটি multinational company-র বাংলাদেশ অফিস ও আমেরিকা অফিস একসাথে connected

**বৈশিষ্ট্য:**
- আকার বিশাল - দেশ থেকে মহাদেশ পর্যন্ত
- Submarine cable, satellite দিয়ে চলে
- Speed LAN-এর তুলনায় কম
- খরচ অনেক বেশি


### তিনটির তুলনা এক নজরে

| বৈশিষ্ট্য | LAN | MAN | WAN |
|-----------|-----|-----|-----|
| আকার | ঘর/বিল্ডিং | শহর/জেলা | দেশ/মহাদেশ |
| উদাহরণ | বাসার WiFi | ব্যাংকের শাখা | ইন্টারনেট |
| Speed | সর্বোচ্চ | মাঝারি | তুলনামূলক কম |
| খরচ | কম | মাঝারি | অনেক বেশি |
| মালিক | তুমি/অফিস | ISP/সরকার | ISP/সরকার |


## DHCP - Dynamic Host Configuration Protocol

DHCP হলো একটি সার্ভিস যেটি নেটওয়ার্কে যুক্ত হওয়া ডিভাইসকে অটোমেটিক IP Address দেয়। আপনি এভাবে ভাবতে পারেন যে আপনি একটি হোটেলে গেছেন। রিসেপশনিস্ট আপনাকে অটোমেটিক একটি রুম নম্বর দিয়ে দেয়, আপনাকে নিজে রুম বেছে নিতে হয় না। DHCP সার্ভার হলো সেই রিসেপশনিস্ট, আর IP Address হলো রুম নম্বর।

**কীভাবে কাজ করে:**
```
আপনার PC চালু হলো
      ↓
Network-এ broadcast করলো → "আমাকে কেউ IP দাও!"
      ↓
DHCP Server উত্তর দিলো → "নাও, এটা তোমার IP: 192.168.1.105"
      ↓
আপনার PC সেই IP নিয়ে নিলো - internet চলছে!
```

**DHCP কী কী দেয়:**
- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

আপনি বাসার WiFi-তে connect করলে আপনার phone অটোমেটিক `192.168.0.X` টাইপের IP পায় - এটাই DHCP-এর কাজ।


## APIPA - Automatic Private IP Addressing

APIPA হলো Windows/Linux-এর একটি fallback mechanism। যখন কোনো DHCP Server পাওয়া যায় না, তখন ডিভাইস নিজেই একটি IP নিজেকে দিয়ে নেয়।

হোটেলের রিসেপশনিস্ট (DHCP) অফিসে নেই। আপনি নিজেই একটি রুম নম্বর বেছে নিলেন কিন্তু সেই রুমে বাইরে থেকে কেউ আসতে পারবে না, শুধু হোটেলের ভেতরেই চলাচল করতে পারবে।

**APIPA Range:**
```
169.254.0.1 - 169.254.255.254
```

**কখন হয়:**
- DHCP Server down থাকলে
- Network cable খোলা থাকলে
- Router বন্ধ থাকলে

**সমস্যা কী:**
- এই IP দিয়ে ইন্টারনেট চলে না
- শুধু একই subnet-এর ডিভাইসের সাথে কথা বলা যায়

> **DevOps টিপ:** কোনো server-এ `169.254.x.x` IP দেখলে বুঝবে DHCP পাচ্ছে না, নেটওয়ার্ক কানেকশন বা DHCP server চেক করতে হবে।


## CIDR - Classless Inter-Domain Routing

CIDR হলো IP Address-এর সাথে `/24` বা `/16` এই notation ব্যবহার করে network-এর আকার নির্ধারণের পদ্ধতি। একটি এলাকার ঠিকানা হলো "মিরপুর-১০, ঢাকা"। "মিরপুর-১০" হলো network address, আর নির্দিষ্ট বাড়ি নম্বর হলো host address। CIDR বলে দেয় কতটুকু জায়গা জুড়ে এই এলাকা।

**সহজ উদাহরণ:**

```
192.168.1.0/24

এখানে:
  192.168.1  → Network Part  (প্রথম 24 bit = 3 octet)
  .0         → Host Part     (শেষ 8 bit = 1 octet)

মানে এই network-এ থাকতে পারবে:
  2⁸ - 2 = 254 টি device (192.168.1.1 - 192.168.1.254)
```

**আরো উদাহরণ:**

| CIDR | Subnet Mask | মোট Host |
|------|-------------|---------|
| /8 | 255.0.0.0 | ~1.6 কোটি |
| /16 | 255.255.0.0 | ~65,000 |
| /24 | 255.255.255.0 | 254 |
| /30 | 255.255.255.252 | 2 (Point-to-Point link-এ ব্যবহার) |

**CIDR আসার আগে কী সমস্যা ছিল:**

আগে শুধু Class A, B, C ছিল। Class A মানেই ১.৬ কোটি IP। একটি ছোট কোম্পানির জন্য এত IP দরকার নেই, বিশাল অপচয়। CIDR এসে বলল তোমার ঠিক যতটুকু দরকার, ততটুকু নাও। এতে IP Address অপচয় কমলো।


## সারসংক্ষেপ

| Acronym | এক লাইনে মনে রাখার সূত্র |
|---------|------------------------|
| **LAN** | বাসার/অফিসের নেটওয়ার্ক |
| **MAN** | শহরের নেটওয়ার্ক |
| **WAN** | ইন্টারনেট = সবচেয়ে বড় WAN |
| **DHCP** | অটোমেটিক IP দেওয়ার সার্ভিস |
| **APIPA** | DHCP না পেলে নিজেই নেওয়া emergency IP |
| **CIDR** | `/24` notation দিয়ে network-এর সাইজ বলা |


### অনুশীলন - কোনটি কোন Class?

| IP Address | Class |
|-----------|-------|
| 203.112.194.243 | C |
| 224.0.0.10 | D (Multicast) |
| 191.224.242.20 | B |
| 135.253.104.25 | B |


## IPv4 Full Range (TCP/IP Unicast & Broadcast)

| Class | Range |
|-------|-------|
| Class A | 0.0.0.0 – 127.255.255.255 |
| Class B | 128.0.0.0 – 191.255.255.255 |
| Class C | 192.0.0.0 – 223.255.255.255 |


## IPv4 Address-এর ধরনসমূহ

| ধরন | বিবরণ |
|-----|-------|
| **Public IP** | Globally Routable, Paid/Leased, ইন্টারনেটে পরিচিত |
| **Private IP** | Non-Routable, ফ্রি, শুধু internal network-এ |
| **Loopback IP** | `127.0.0.0/8` - প্রতিটি সিস্টেমে ডিফল্ট থাকে, নিজেকে নিজে ping করতে ব্যবহার হয় |
| **APIPA** | `169.254.0.1 – 169.254.255.254` - DHCP না পেলে অটো-সেট হয় |
| **Documentation** | `198.51.100.0/24` - শুধু ডকুমেন্টেশন ও উদাহরণে ব্যবহার |

**সংক্ষিপ্ত পরিচয়:**

- **DHCP** = Dynamic Host Configuration Protocol
- **APIPA** = Automatic Private IP Addressing
- **CIDR** = Classless Inter-Domain Routing

### CIDR Notation বোঝা

IP Address-এর পাশে যে `/24` বা `/16` লেখা থাকে, তাকে **Subnet Mask** বা **CIDR notation** বলে। এটি দিয়ে network-এর আকার নির্ধারণ করা হয়।

```
127.0.0.0/8   →  Subnet Mask: 255.0.0.0
170.0.0.0/16  →  Subnet Mask: 255.255.0.0
192.0.0.0/24  →  Subnet Mask: 255.255.255.0
```

> **মনে রাখুন:** 8-bit-এর সর্বোচ্চ মান = 2⁸ - 1 = **255**


## Private IP Address Range

| Class | Private Range |
|-------|-------------|
| Class A | 10.0.0.0 – 10.255.255.255 |
| Class B | 172.16.0.0 – 172.31.255.255 *(RHCSA Exam-এ এই block থেকে প্রশ্ন আসে)* |
| Class C | 192.168.0.0 – 192.168.255.255 |


## Linux Network Management

### Windows vs Linux Interface Naming

**Windows-এ** দেখা যায়:
```
Ethernet1, Ethernet2
```

**Linux-এ** interface naming convention:

| Prefix | অর্থ |
|--------|------|
| `lo` | Loopback |
| `en` | Ethernet (wired) |
| `wl` | Wireless (WiFi) |
| `br` / `vbr` | Virtual Bridge |

### `ip link` - কী দেখায়?

| তথ্য | অর্থ |
|------|------|
| Interface Name | lo, ens33, eth0, etc. |
| State | UP / DOWN |
| Link Speed | 1000 Mbps (1 Gbps) |
| MAC Address | `ether` সেকশনে |
| MTU | সর্বোচ্চ frame size (default: **1500 bytes**) |

> **MTU (Maximum Transmission Unit):** নেটওয়ার্কে একবারে পাঠানো যায় এমন সর্বোচ্চ data packet-এর আকার।

### `ip addr show` - কী দেখায়?

| ফিল্ড | অর্থ |
|-------|------|
| `inet` | IPv4 Address |
| `inet6` | IPv6 Address (Link Local) |
| `brd` | Broadcast Address |
| `ether` | MAC Address |

> ⚠️ `ifconfig` ব্যবহার করতে হলে **`net-tools`** package ইনস্টল থাকতে হবে।

---

## RHEL/CentOS Interface Naming Convention

| ধরন | উদাহরণ |
|-----|--------|
| Ethernet (modern) | `enp2s0`, `eno1`, `ens33`, `ens33` |
| Legacy | `eth0`, `eth1` |
| Virtual/Sub Interface | `ens33:1`, `eth0:1` |
| Loopback | `lo` |
| Bridge | `br0`, `vbr0` |
| Wireless | `wlan0`, `wlan1` |


## Physical Connectivity Check

```bash
# Interface status দেখুন
ip link

# Detailed NIC info (speed, duplex, link detect)
ethtool ens33
```

`ens33` এটা নেটওয়ার্ক ইন্টারফেইস নাম। এটা সবার ক্ষেত্রে বা সব অপারেটিং সিস্টেমের জন্য একই হবে না। আপনার নেটওয়ার্ক ইন্টারফেইস দেখার জন্য `ip link` রান করুন।


## Hostname Configure করা

```bash
# বর্তমান hostname দেখুন
hostname

# বিস্তারিত hostname info
hostnamectl status

# নতুন hostname সেট করা
hostnamectl set-hostname node1.example.com

# Shell reload করুন (যাতে prompt-এ নতুন নাম দেখায়)
exec bash

# Confirm করুন
cat /etc/hostname
hostname
```


## Network Card Enable / Disable

```bash
# Step 1: Interface-এর নাম দেখুন
ip link

# Interface disable করুন
ip link set ens33 down

# Status verify করুন
ip link    # "DOWN" দেখাবে

# Interface enable করুন
ip link set ens33 up

# Status verify করুন
ip link    # "UP" দেখাবে
```


## Gateway ও DNS দেখা

```bash
# Gateway (Routing Table) দেখুন
route -n
# অথবা (route -n কাজ না করলে)
ip route

# DNS দেখুন
cat /etc/resolv.conf

# Connectivity test (external)
ping 8.8.8.8
ping www.google.com

# নির্দিষ্ট সংখ্যকবার ping
ping -c 4 8.8.8.8
ping -c4 8.8.8.8
```

> **Bridge Interface** = VMware-এ physical connection-এর মতো কাজ করে।


## Default Gateway Add / Remove (Optional / Temporary)

```bash
# নতুন Default Gateway add করুন
route add default gw 172.25.11.254
route -n

# Test করুন
ping 8.8.8.8

# Gateway remove করুন
route del default gw 172.25.11.254

# আবার verify করুন
route -n
ping 8.8.8.8
```

> এই পদ্ধতিতে করা পরিবর্তনগুলো **temporary** হয়, reboot দিলে চলে যাবে। Permanent করতে `nmcli` ব্যবহার করুন।


## IP Address Configure করার পদ্ধতি

আমরা দুইভাবে IP Address configure করতে পারি:

| পদ্ধতি | বিবরণ |
|--------|-------|
| **Static** | ম্যানুয়ালি IP বসানো হয় |
| **DHCP** | DHCP Server থেকে অটোমেটিক IP নেওয়া হয় |

বিভিন্ন tool দিয়ে Static IP configure করা যায়। Debian বেইজড অপারেটিং সিস্টেমে `yml` ফাইলে লিখতে হয়। কিন্ত fedora বেইজড অপারেটিং সিস্টেমে কমান্ড দিয়ে বা GUI ব্যাবহার করেও configure করতে পারি। 

### Debian based network configuration

`sudo vim /etc/netplan/50-cloud-init.yaml`

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.206.150/24
      routes:
        - to: default
          via: 192.168.206.2
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
```

Debian বেইজড অপারেটিং সিস্টেমে Network Configuration এর লোকেশন `/etc/netplan/` তবে ফাইলের নাম OS ভেদে ভিন্ন হতে পারে।

### Fedora based network configuration


| Tool | বিবরণ |
|------|-------|
| `nmtui` | NetworkManager Text UI (সহজ visual interface) |
| `nmcli` | NetworkManager Command Line Interface |
| GUI | Graphical Desktop থেকে |
| `vim` | সরাসরি config file edit করে |
| YAML | `/etc/NetworkManager/system-connections/` ফাইল edit করে |

> ⚠️ **IP বসানোর আগে** NetworkManager service চলছে কিনা নিশ্চিত করো:

```bash
systemctl status NetworkManager.service
systemctl restart NetworkManager.service
systemctl enable NetworkManager.service
```


## nmtui দিয়ে IP Configure

```bash
nmtui
```

ধাপগুলো:
1. **Edit a connection** → নিজের interface select করুন
2. **Edit** প্রেস করুন
3. **IPv4 Configuration** → `Automatic` থেকে `Manual`- সিলেক্ট করুন → **Show** প্রেস করুন
4. **Add** প্রেস করে নিচের তথ্য দিন:
   - Address: `172.25.11.X/24`
   - Gateway: `172.25.11.1`
   - DNS Server 1: `8.8.8.8`
   - DNS Server 2: `9.9.9.9`
5. **[x] Automatically Connect** চেক করুন
6. **OK** → **Back** → **Activate a connection** (দুইবার press করুন - deactivate তারপর activate)
7. `ifconfig` দিয়ে verify করুন


## IP Address দেখার Commands

```bash
# বিস্তারিত IP info
ip address show
# সংক্ষেপে
ip addr show / ip addr / ip a

# নির্দিষ্ট interface-এর info
ip address show ens33
# সংক্ষেপে
ip a s ens33

# পুরনো পদ্ধতি (net-tools লাগবে)
ifconfig
ifconfig ens33

# Gateway দেখো
ip route
# সংক্ষেপে
ip r
```

### `ifconfig` output বোঝা:

| ফিল্ড | অর্থ |
|-------|------|
| `inet` | IPv4 Address |
| `inet6` | IPv6 Address |
| `brd` | Broadcast Address |
| `mtu` | Maximum Transmission Unit (1500) |
| `netmask` | Subnet Mask |
| `ether` | MAC Address |

> Interface-এর নাম hardware ভেদে ভিন্ন হতে পারে: `ens33`, `ens33`, `enp2s0`, `eth0`, `wlan0` - সবই normal।


## nmcli - NetworkManager Command Line Interface

`nmcli` দিয়ে connection তৈরি, দেখা, সম্পাদনা, মুছে ফেলা, activate ও deactivate করা যায়।

### গুরুত্বপূর্ণ Commands:

```bash
# সব network device-এর বিস্তারিত দেখুন
nmcli device show

# সব connection-এর list দেখুন
nmcli connection show
```

### Static IPv4 Configure (Step by Step):

```bash
nmcli con modify ens33 ipv4.addresses 172.25.11.X/24
nmcli con modify ens33 ipv4.gateway 172.25.11.1
nmcli con modify ens33 ipv4.dns 8.8.8.8
nmcli con modify ens33 ipv4.method manual
nmcli con modify ens33 autoconnect yes

# Connection restart করুন
nmcli con down ens33
nmcli con up ens33

# Verify করুন
ifconfig ens33
```

### একটি Command-এ সব Configure:

```bash
nmcli connection modify ens33 \
  ipv4.addresses 192.168.0.110/24 \
  ipv4.gateway 192.168.0.2 \
  ipv4.dns 1.1.1.1,4.2.2.2,8.8.8.8 \
  ipv4.method manual \
  connection.autoconnect yes
```

### নতুন Connection Add করা:

```bash
# শুধু connection তৈরি
nmcli connection add con-name private-net type ethernet ifname ens192

# Connection তৈরি + IP সব একসাথে
nmcli connection add con-name private-net type ethernet ifname ens192 \
  ipv4.addresses 192.168.0.110/24 \
  ipv4.gateway 192.168.0.2 \
  ipv4.dns 1.1.1.1,4.2.2.2,8.8.8.8 \
  ipv4.method manual \
  connection.autoconnect yes
```

### Connection Delete করা:

```bash
nmcli connection delete "Wired Connection 1"
```

### Connection Up / Down:

```bash
nmcli connection down ens33
nmcli connection up ens33

# অথবা
nmcli networking off
nmcli networking on
```


## SNAT ও DNAT - সংক্ষিপ্ত পরিচয়

| সংক্ষিপ্ত | পূর্ণরূপ | কাজ |
|-----------|---------|-----|
| **SNAT** | Source Network Address Translation | প্যাকেটের source IP বদলানো হয় |
| **DNAT** | Destination Network Address Translation | প্যাকেটের destination IP বদলানো হয় |


## IPv6 - বিস্তারিত তুলনা

| বৈশিষ্ট্য | IPv4 | IPv6 |
|-----------|------|------|
| Bits | 32 | 128 |
| Parts | 4 | 8 |
| প্রতিটি অংশ | Octet | Hextet |
| Separator | Dot (`.`) | Colon (`:`) |
| Number Format | Decimal (0–9) | Hexadecimal (0–9, A–F) |
| Class | 5টি (A,B,C,D,E) | প্রযোজ্য নয় |
| উদাহরণ | `192.168.11.10` | `2001:db8::abcd:12c:0:1` |

### IPv4 ও IPv6-এর সমতুল্য Address:

| ধরন | IPv4 | IPv6 |
|-----|------|------|
| Loopback | `127.0.0.1` | `::1/128` |
| Private | `10.x.x.x`, `172.16–31.x.x`, `192.168.x.x` | `fd00::/8` |
| Public | Public IP | `2000::/3` (APNIC: `2400::`) |
| APIPA | `169.254.x.x` | `fe80::/10` (Link Local) |
| Multicast | `224.0.0.0–239.x.x.x` | `ff00::/8` |
| All IPs | `0.0.0.0` | `::` |

```bash
# IPv4 loopback test
ping 127.0.0.1

# IPv6 loopback test
ping ::1
```


## Static IPv6 Configure - nmcli দিয়ে

```bash
nmcli connection show

nmcli con modify ens33 ipv6.addresses fd00:db08:abcd::1234:X/64
nmcli con modify ens33 ipv6.gateway fd00:db08:abcd::1234:1
nmcli con modify ens33 ipv6.dns fd00:db08:abcd::1234:254
nmcli con modify ens33 ipv6.method manual
nmcli con modify ens33 autoconnect yes
nmcli con up ens33

# Verify করো
ifconfig

# Ping test
ping fd00:db08:abcd::1234:5
```


## RHEL 9 - Network Configuration

**Config file location:**
```
/etc/NetworkManager/system-connections/
```

> ⚠️ এই ফাইলে সরাসরি edit না করে **nmcli command** ব্যবহার করাই ভালো।

```bash
nmcli connection modify ens33 ipv4.addresses 192.168.0.110/24
nmcli connection modify ens33 ipv4.gateway 192.168.0.2
nmcli connection modify ens33 ipv4.dns 8.8.8.8
nmcli connection modify ens33 ipv4.method manual
nmcli connection modify ens33 autoconnect yes

# Reload করুন
nmcli connection down ens33
nmcli connection up ens33

# Verify করুন
ifconfig ens33
```


## VMware Network Modes

| Mode | অর্থ |
|------|------|
| **Host Only** | Same network block-এ শুধু host-এর সাথে communicate করতে পারে |
| **NAT** | Host-এর মাধ্যমে বাইরের network-এও access নেওয়া যায় |
| **Bridged** | Physical network-এর সরাসরি অংশ হয়ে যায় |


## /etc/resolv.conf - DNS Configuration

```bash
cat /etc/resolv.conf
```

> DNS হলো **ফোনবুকের মতো** - domain name দিলে সে IP address রিটার্ন করে।
> এই ফাইলে সর্বোচ্চ **৩টি** DNS nameserver কাজ করে।


## Quick Reference - সব গুরুত্বপূর্ণ Commands

```bash
# Interface দেখুন
ip link
ip addr show
ifconfig

# Gateway দেখুন
ip route
route -n

# DNS দেখুন
cat /etc/resolv.conf

# Hostname
hostname
hostnamectl status
hostnamectl set-hostname myserver.example.com

# Connectivity test
ping -c 4 8.8.8.8
ping www.google.com

# NIC enable/disable
ip link set ens33 up
ip link set ens33 down

# nmcli
nmcli device show
nmcli connection show
nmcli con up ens33
nmcli con down ens33
```