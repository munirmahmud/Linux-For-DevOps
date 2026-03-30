# Chapter 4 - Lesson 4: DNS Tools

এই লেসনে আমরা শিখবো `dig`, `nslookup`, `host`, `/etc/resolv.conf` এর বিস্তারিত

**Chapter 4 | Lesson 4 of 11**


আপনাকে স্বাগতম Lesson 4-এ! 🎉 আগের lesson-এ আমরা `ping`, `traceroute`, `mtr` দিয়ে connectivity test করতে শিখেছিলাম। আজ আমরা শিখবো **DNS** - মানে কীভাবে Linux domain name-কে IP address-এ convert করে, এবং সেটা নিয়ে কাজ করার tools।


## DNS কী?

**Real-world analogy:**
> আপনার phone-এ contact list আছে। আপনি "Rahim Bhai" লিখে তার নাম্বার সেইভ করলেন এবং তাকে call করেন কিন্তু আসলে phone dial করে একটা number। DNS ঠিক এরকম - আপনি `google.com` লিখেন, DNS সেটাকে `142.250.185.46` অর্থাৎ IP-তে convert করে।

```
আপনি লিখেন:  google.com
DNS বলে:    এর IP হলো 142.250.185.46
Browser যায়: 142.250.185.46
```

**DNS = Domain Name System** - এটা একটা globally distributed database যেটা domain → IP mapping রাখে।


## DNS Record Types - জানা দরকার

| Record Type | মানে কী | উদাহরণ |
|---|---|---|
| **A** | Domain → IPv4 address | `google.com → 142.250.x.x` |
| **AAAA** | Domain → IPv6 address | `google.com → 2607:f8b0:...` |
| **MX** | Mail server কোনটা | `gmail.com এর mail server` |
| **CNAME** | একটা domain → আরেকটা domain | `www → example.com` |
| **NS** | Nameserver কোনটা | `ns1.google.com` |
| **TXT** | Text info (SPF, verification) | `"v=spf1 include:..."` |
| **PTR** | IP → Domain (reverse lookup) | `142.x.x.x → google.com` |
| **SOA** | Zone এর authority info | Primary nameserver info |


## Tool 1: `dig` - সবচেয়ে powerful DNS tool

`dig` = **Domain Information Groper** এটা DevOps-এ সবচেয়ে বেশি ব্যবহৃত DNS tool। খুব detailed output দেয়।

### Basic Syntax:
```bash
dig [options] [domain] [record_type]
```


### Example 1: Basic A record lookup

```bash
dig google.com
```

**Expected Output:**
```
; <<>> DiG 9.18.1 <<>> google.com
;; global options: +cmd

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             112     IN      A       142.250.185.46

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun Mar 15 10:00:00 2026
;; MSG SIZE  rcvd: 55
```

**Output বুঝি:**
```
QUESTION SECTION  → তুমি কী জিজ্ঞেস করেছো
ANSWER SECTION    → উত্তর (google.com এর IP = 142.250.185.46)
112               → TTL (Time to Live) - ১১২ সেকেন্ড cache থাকবে
IN                → Internet class
A                 → A record (IPv4)
Query time        → DNS response কতটুকু সময় নিলো
SERVER            → কোন DNS server উত্তর দিলো
```


### Example 2: Short/Clean output - `+short`

```bash
dig google.com +short
```

**Output:**
```
142.250.185.46
```

> DevOps script-এ এটা সবচেয়ে বেশি ব্যবহার হয় - শুধু IP দরকার!


### Example 3: Specific record type query

```bash
# MX record - mail server কোনটা
dig google.com MX
```

**Output:**
```
;; ANSWER SECTION:
google.com.     600    IN    MX    10 smtp.google.com.
```

```bash
# NS record - nameserver কোনটা
dig google.com NS +short
```

**Output:**
```
ns1.google.com.
ns2.google.com.
ns3.google.com.
ns4.google.com.
```

```bash
# TXT record - SPF, verification ইত্যাদি
dig google.com TXT +short
```

**Output:**
```
"v=spf1 include:_spf.google.com ~all"
"google-site-verification=..."
```


### Example 4: Specific DNS server ব্যবহার করে query - `@`

```bash
# Google এর DNS (8.8.8.8) দিয়ে query করুন
dig @8.8.8.8 google.com

# Cloudflare এর DNS (1.1.1.1) দিয়ে query করুন
dig @1.1.1.1 google.com +short

# নিজের local DNS server দিয়ে
dig @192.168.1.1 google.com
```

> **DevOps use case:** দুটো different DNS server-এর উত্তর compare করতে বা কোনটা যদি কাজ না করে সেটা debug করতে লাগে!


### Example 5: Reverse DNS lookup - IP থেকে domain

```bash
dig -x 8.8.8.8 +short
```

**Output:**
```
dns.google.
```

> IP দিলে সেই IP কার domain সেটা বের করে - এটাকে **Reverse DNS** বলে।


### Example 6: Trace - DNS কোন path দিয়ে resolve হচ্ছে

```bash
dig google.com +trace
```

**Output (সংক্ষিপ্ত):**
```
.                       root nameservers থেকে শুরু
com.                    .com TLD nameserver
google.com.             google এর own nameserver
google.com.    A        142.250.185.46  ← final answer
```

> **DevOps use case:** DNS propagation সমস্যা debug করতে - কোন step-এ আটকে আছে!


### Example 7: ALL records একসাথে দেখা - `ANY`

```bash
dig google.com ANY +short
```

> সব ধরনের DNS record একবারে দেখাবে।


### dig Cheat Sheet:

```bash
dig domain.com              # A record (default)
dig domain.com MX           # Mail server
dig domain.com NS           # Nameserver
dig domain.com TXT          # TXT records
dig domain.com AAAA         # IPv6 address
dig domain.com CNAME        # CNAME record
dig domain.com SOA          # Start of Authority
dig domain.com +short       # Clean output শুধু
dig -x IP_ADDRESS           # Reverse lookup
dig @8.8.8.8 domain.com     # Specific DNS server use করো
dig domain.com +trace       # Full resolution path দেখাও
dig domain.com +noall +answer  # শুধু answer section দেখাও
```


## Tool 2: `nslookup` - Simple & Interactive DNS tool

`nslookup` পুরনো tool, কিন্তু Windows-এও কাজ করে তাই DevOps-এ জানা দরকার।

### Basic Syntax:
```bash
nslookup [domain] [dns_server]
```


### Example 1: Basic lookup

```bash
nslookup google.com
```

**Output:**
```
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.46
```

**বুঝি:**
```
Server          → কোন DNS server উত্তর দিলো
Non-authoritative → এটা cache করা উত্তর (google এর নিজের server থেকে সরাসরি না)
Address         → IP address
```


### Example 2: Specific DNS server দিয়ে

```bash
nslookup google.com 8.8.8.8
```


### Example 3: Specific record type

```bash
nslookup -type=MX gmail.com
nslookup -type=NS google.com
nslookup -type=TXT google.com
```


### Example 4: Interactive mode

```bash
nslookup        # শুধু enter চাপুন
```

**Output:**
```
>               ← prompt আসবে
```

তারপর type করুন:
```
> google.com           # A record
> set type=MX
> gmail.com            # MX record
> set type=NS
> google.com           # NS record
> exit                 # বের হও
```

> **Interactive mode** কখন দরকার: অনেকগুলো query একটার পর একটা করতে হলে।


## Tool 3: `host` - সবচেয়ে simple DNS tool

`host` command হলো সবচেয়ে সহজ এবং human-readable output দেয়।

### Basic Syntax:
```bash
host [domain]
```


### Example 1: Basic lookup

```bash
host google.com
```

**Output:**
```
google.com has address 142.250.185.46
google.com has IPv6 address 2607:f8b0:4004:c1b::71
google.com mail is handled by 10 smtp.google.com.
```

> এক command-এ A, AAAA, MX সব দেখাচ্ছে! Clean এবং simple।


### Example 2: Specific record type

```bash
host -t MX gmail.com
host -t NS google.com
host -t TXT google.com
```


### Example 3: Reverse lookup

```bash
host 8.8.8.8
```

**Output:**
```
8.8.8.8.in-addr.arpa domain name pointer dns.google.
```


### Example 4: Specific DNS server দিয়ে

```bash
host google.com 1.1.1.1
```


## `/etc/resolv.conf` - Linux এর DNS Configuration File

এটা সেই file যেটা Linux-কে বলে: **"DNS query করতে হলে কোন server-এ যাবি?"**

### দেখি file-টা:

```bash
cat /etc/resolv.conf
```

**Output:**
```
# This is /etc/resolv.conf managed by systemd-resolved
nameserver 127.0.0.53
nameserver 8.8.8.8
nameserver 1.1.1.1
search example.com office.local
options ndots:5
```

**প্রতিটা line বুঝি:**

```
nameserver 127.0.0.53    → প্রথমে এই DNS server-কে জিজ্ঞেস করো
                           (127.0.0.53 = systemd-resolved এর local address)

nameserver 8.8.8.8       → প্রথমটা fail করলে Google DNS ব্যবহার করো

nameserver 1.1.1.1       → তারপর Cloudflare DNS

search example.com       → শুধু "mail" type করলে আগে "mail.example.com"
                           try করবে। Short hostname resolve করার জন্য।

options ndots:5          → ৫টার কম dot থাকলে search domain যোগ করবে
```


### `/etc/resolv.conf` manually edit করা:

```bash
sudo nano /etc/resolv.conf
```

```
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

> ⚠️ **Warning:** অনেক modern system-এ এই file automatically overwrite হয়ে যায় (systemd-resolved বা NetworkManager দ্বারা)। Permanent change করতে হলে `/etc/systemd/resolved.conf` edit করতে হয়।

---

### Permanent DNS change - systemd-resolved:

```bash
sudo nano /etc/systemd/resolved.conf
```

```ini
[Resolve]
DNS=8.8.8.8 1.1.1.1
FallbackDNS=8.8.4.4
```

```bash
# Apply করুন
sudo systemctl restart systemd-resolved
```


## সব tool একসাথে - কখন কোনটা ব্যবহার করবো?

| Tool | কখন ব্যবহার করবো |
|---|---|
| `dig` | Detailed DNS debugging, DevOps script, trace করতে |
| `dig +short` | Script-এ শুধু IP দরকার হলে |
| `nslookup` | Quick check, Windows compatibility, interactive mode |
| `host` | Human-readable simple output |
| `/etc/resolv.conf` | System কোন DNS server ব্যবহার করছে দেখতে/বদলাতে |


## Real DevOps Scenarios

### Scenario 1: 

নতুন domain deploy করেছেন, DNS propagate হয়েছে কিনা যেভাবে check করবেন

```bash
# নিজের system এর DNS দিয়ে
dig myapp.example.com +short

# Google DNS দিয়ে - globally propagate হয়েছে?
dig @8.8.8.8 myapp.example.com +short

# Cloudflare দিয়ে
dig @1.1.1.1 myapp.example.com +short
```

> তিনটার উত্তর same হলে DNS propagation সফল।


### Scenario 2: 

Mail server সমস্যা - email deliver হচ্ছে না

```bash
# MX record সঠিক আছে কিনা দেখুন
dig gmail.com MX +short

# Mail server টা live আছে কিনা দেখুন
dig smtp.google.com +short
```


### Scenario 3: 

Internal Kubernetes cluster DNS debug

```bash
# কোন DNS server ব্যবহার হচ্ছে
cat /etc/resolv.conf

# CoreDNS সঠিকভাবে কাজ করছে কিনা
dig @10.96.0.10 kubernetes.default.svc.cluster.local

# Service resolve হচ্ছে কিনা
dig @10.96.0.10 myservice.mynamespace.svc.cluster.local
```


### Scenario 4: 

একটা IP দিয়ে hostname বের করুন

```bash
# এই IP কোন server-এ কানেক্ট করা আছে?
dig -x 93.184.216.34 +short
host 93.184.216.34
nslookup 93.184.216.34
```


## 📝 Lesson Summary

- **DNS** = Domain name কে IP address-এ convert করে
- **`dig`** = সবচেয়ে powerful tool, detailed output, DevOps-এ সবচেয়ে বেশি ব্যবহার
- **`dig +short`** = শুধু IP/result দেখায়, script-এ কাজে লাগে
- **`dig @server`** = specific DNS server দিয়ে query করুন
- **`dig -x IP`** = Reverse lookup - IP থেকে domain
- **`dig +trace`** = DNS resolution path দেখুন
- **`nslookup`** = Simple tool, interactive mode আছে, Windows-এও চলে
- **`host`** = সবচেয়ে simple, human-readable output
- **`/etc/resolv.conf`** = Linux কোন DNS server ব্যবহার করবে সেটা এখানে configured
- DNS record types: **A, AAAA, MX, NS, CNAME, TXT, PTR, SOA**


## 🏋️ Practice Tasks

**Task 1:**
`facebook.com` এর A record, MX record, এবং NS record বের করুন `dig` দিয়ে। তারপর `dig +short` দিয়ে শুধু IP দেখুন।

**Task 2:**
Google এর DNS (`8.8.8.8`) এবং Cloudflare এর DNS (`1.1.1.1`) দুটো দিয়ে আলাদাভাবে `amazon.com` query করুন এবং দেখুন উত্তর same কিনা।

**Task 3:**
`cat /etc/resolv.conf` রান করুন এবং আপনার system কোন DNS server ব্যবহার করছে সেটা জানুন। তারপর `8.8.8.8` এর reverse lookup করুন।

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 5: Downloading & Transferring Files**

> `curl`, `wget`, `scp`, `rsync` - file download করা, remote server-এ file পাঠানো, এবং efficiently sync করা শিখবো। এগুলো DevOps-এ প্রতিদিন কাজে লাগে! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../03-Testing-Connectivity">← Testing Connectivity</a>
    </td>
    <td align="right">
      <a href="../05-Downloading-And-Transferring-Files">Downloading And Transferring Files →</a>
    </td>
  </tr>
</table>