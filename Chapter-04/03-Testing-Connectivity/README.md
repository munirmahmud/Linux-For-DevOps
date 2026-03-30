# Chapter 4 - Lesson 3: Testing Connectivity

**Chapter 4 | Lesson 3 of 11**


আপনাকে স্বাগতম Lesson 3-তে! 🎉 আগের lesson-এ আমরা শিখেছিলাম কিভাবে network configuration দেখতে হয়। এখন আমরা শিখবো কোনো server বা machine আসলে reachable কিনা, সেটা কিভাবে test করতে হয়।

DevOps Engineer হিসেবে এটা আপনার daily কাজের অংশ হবে - কেন এই server-এ connect হচ্ছে না? এই প্রশ্নের উত্তর খুঁজতে হবে প্রায়ই।


মনে করেন আপনি কাউকে phone করলেন:

| Tool | Analogy |
|------|---------|
| `ping` | Phone করা, ring গেলো কিনা দেখা |
| `traceroute` | Phone-এর signal কোন কোন tower হয়ে গেলো সেটা দেখা |
| `mtr` | Live-এ দেখা প্রতিটা tower-এ কতটুকু delay হচ্ছে |


## Command 1: `ping`

### এটা কী করে?
`ping` একটা ছোট্ট packet পাঠায় target machine-এ, এবং সেটা ফিরে আসে কিনা দেখে। এটা দিয়ে বোঝা যায়:
- Machine টা **alive (reachable)** কিনা
- Network-এ কতটুকু **delay (latency)** আছে
- কোনো **packet loss** হচ্ছে কিনা

> এটা ICMP (Internet Control Message Protocol) ব্যবহার করে।


### Syntax:
```bash
ping [options] <destination>
```


### Basic Example:
```bash
ping google.com
```

**Expected Output:**
```
PING google.com (142.250.195.78) 56(84) bytes of data.
64 bytes from bom12s01-in-f14.1e100.net (142.250.195.78): icmp_seq=1 ttl=117 time=2.45 ms
64 bytes from bom12s01-in-f14.1e100.net (142.250.195.78): icmp_seq=2 ttl=117 time=2.51 ms
64 bytes from bom12s01-in-f14.1e100.net (142.250.195.78): icmp_seq=3 ttl=117 time=2.48 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 2.45/2.48/2.51/0.025 ms
```

### Output-এর প্রতিটা অংশের কী মানে?

| অংশ | মানে |
|-----|------|
| `64 bytes` | পাঠানো packet-এর size |
| `icmp_seq=1` | এটা কততম packet |
| `ttl=117` | Time To Live - packet কতটা "হপ" করতে পারবে |
| `time=2.45 ms` | কতটুকু সময় লাগলো যেতে-আসতে (latency) |
| `0% packet loss` | কোনো packet হারায়নি মানে connection ভালো |


### Important Options:

```bash
# শুধু 4টা packet পাঠাও (Linux-এ এটা দিলে automatically বন্ধ হয়)
ping -c 4 google.com
# অথবা
ping -c4 google.com

# প্রতি 0.5 সেকেন্ডে packet পাঠাও (default 1 সেকেন্ড)
ping -i 0.5 google.com

# Packet size বড় করে পাঠাও (network stress test)
ping -s 1000 google.com

# Timeout দাও - 5 সেকেন্ডের মধ্যে জবাব না পেলে বন্ধ
ping -W 5 google.com

# IP address দিয়েও করা যায়
ping 8.8.8.8
```


### যদি ping fail করে:
```
ping: connect: Network is unreachable
```
এর মানে - আপনার machine-এর কোনো network route নেই।

```
Request timeout for icmp_seq 1
```
এর মানে - destination machine reachable না, বা ICMP block করা আছে (firewall)।


### DevOps Use Cases:
- Deployment-এর পরে server alive কিনা check করা
- Database server reachable কিনা দেখা
- CI/CD pipeline-এ health check script-এ ব্যবহার


## Command 2: `traceroute`

### এটা কী করে?
`ping` শুধু বলে "পৌঁছালো কি না।" কিন্তু `traceroute` বলে "কোন কোন রাস্তা দিয়ে গেলো।"

Internet-এ একটা packet সরাসরি destination-এ যায় না, সে অনেকগুলো **router (hop)** পার হয়ে যায়। `traceroute` প্রতিটা hop-এর IP এবং latency দেখায়।

> ঢাকা থেকে সিলেট যেতে কোন কোন শহর পার হলো - সেটা দেখানো।


### Installation (যদি না থাকে):
```bash
# Ubuntu/Debian
sudo apt install traceroute

# RHEL/Fedora
sudo yum install traceroute
```


### Syntax:
```bash
traceroute [options] <destination>
```


### Example:
```bash
traceroute google.com
```

**Expected Output:**
```
traceroute to google.com (142.250.195.78), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)          0.532 ms  0.498 ms  0.473 ms
 2  10.10.10.1 (10.10.10.1)         1.234 ms  1.201 ms  1.178 ms
 3  103.87.124.1 (103.87.124.1)     3.456 ms  3.412 ms  3.398 ms
 4  * * *
 5  209.85.243.46 (209.85.243.46)  10.234 ms  10.198 ms 10.187 ms
 6  142.250.195.78 (142.250.195.78) 12.345 ms 12.312 ms 12.298 ms
```

### Output বোঝা:

| অংশ | মানে |
|-----|------|
| `1`, `2`, `3`... | Hop নম্বর - কততম router |
| `_gateway (192.168.1.1)` | Router-এর hostname ও IP |
| `0.532 ms` | ৩টা probe-এর latency (৩ বার পাঠায়) |
| `* * *` | এই hop-টা ICMP response দেয়নি (firewall blocked) |


### Important Options:

```bash
# DNS lookup না করে শুধু IP দেখাও (দ্রুত হয়)
traceroute -n google.com

# Maximum hop limit বাড়াও (default 30)
traceroute -m 50 google.com

# UDP এর বদলে ICMP ব্যবহার করো
traceroute -I google.com

# TCP ব্যবহার করো (firewall bypass করতে)
traceroute -T -p 80 google.com
```


### DevOps Use Cases:
- "কোন router-এ গিয়ে packet আটকে যাচ্ছে?" - এটা বের করা
- Cloud server-এর network path analyze করা
- ISP-এর সমস্যা vs. আপনার server-এর সমস্যা আলাদা করা


## Command 3: `mtr` (My TraceRoute)

### এটা কী করে?
`mtr` হলো `ping` + `traceroute` এর combination এবং এটা **real-time live update** দেয়। এটা continuously packet পাঠাতে থাকে এবং প্রতিটা hop-এর:

- Packet loss %
- Average latency
- Best/Worst latency

সব live দেখায়। এটা **সবচেয়ে powerful connectivity testing tool।**


### Installation:
```bash
# Ubuntu/Debian
sudo apt install mtr

# RHEL/CentOS
sudo yum install mtr
```


### Syntax:
```bash
mtr [options] <destination>
```


### Example:
```bash
mtr google.com
```

**Expected Output (live interactive screen):**
```
                             My traceroute  [v0.95]
myserver (0.0.0.0) -> google.com                        2024-01-15

Keys: Help   Display mode   Restart statistics   Order of fields   quit

                                   Packets               Pings
 Host                            Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. _gateway                      0.0%    10   0.5    0.5   0.4   0.7   0.1
 2. 10.10.10.1                    0.0%    10   1.2    1.2   1.1   1.4   0.1
 3. 103.87.124.1                  0.0%    10   3.4    3.5   3.3   3.8   0.2
 4. ???                          100.0%   10   0.0    0.0   0.0   0.0   0.0
 5. 209.85.243.46                 0.0%    10  10.2   10.3  10.1  10.6   0.2
 6. google.com                    0.0%    10  12.3   12.4  12.2  12.7   0.2
```

### প্রতিটা Column কী মানে?

| Column | মানে |
|--------|------|
| `Host` | Router বা hop-এর address |
| `Loss%` | কত % packet হারিয়ে গেছে |
| `Snt` | মোট কতটা packet পাঠানো হয়েছে |
| `Last` | সর্বশেষ packet-এর latency (ms) |
| `Avg` | গড় latency |
| `Best` | সবচেয়ে কম latency |
| `Wrst` | সবচেয়ে বেশি latency |
| `StDev` | Latency কতটা unstable (বেশি হলে jitter আছে) |


### Important Options:

```bash
# Report mode - interactive না হয়ে একবার দেখিয়ে বন্ধ হবে
mtr --report google.com

# 10টা packet পাঠিয়ে report দাও
mtr --report --report-cycles 10 google.com

# DNS lookup ছাড়া শুধু IP দেখাও
mtr -n google.com

# TCP mode (port 80) - firewall bypass
mtr --tcp --port 80 google.com

# Output সংরক্ষণ করো
mtr --report google.com > mtr_report.txt
```


### DevOps Use Cases:
- Production incident-এ "network কোথায় slow?" বের করা
- Cloud provider-এর network quality measure করা
- Continuous monitoring script-এ ব্যবহার


## তিনটা Tool কখন কোনটা ব্যবহার করবো?

```
সমস্যা হচ্ছে?
│
├── Server alive কিনা জানতে চাই?
│   └── ping google.com
│
├── কোন hop-এ সমস্যা হচ্ছে জানতে চাই?
│   └── traceroute google.com
│
└── Live-তে দেখতে চাই + packet loss দেখতে চাই?
    └── mtr google.com
```


## Practical Comparison:

```bash
# Scenario: আপনার app server production-এ slow respond করছে

# Step 1: আগে দেখুন রিচাবল কিনা
ping -c 4 your-server.com

# Step 2: যদি রিচাবল হয়, কিন্তু slow - route দেখুন
traceroute your-server.com

# Step 3: কোন hop-এ packet loss আছে কিনা live দেখুন
mtr --report your-server.com
```


## 📝 Quick Summary

- **`ping`** - Machine reachable কিনা এবং basic latency check করে। Simple, fast, সবচেয়ে বেশি ব্যবহৃত।
- **`traceroute`** - Packet কোন কোন router হয়ে যাচ্ছে সেটা দেখায়। Network path debug করতে কাজে লাগে।
- **`mtr`** - `ping` + `traceroute` একসাথে, real-time। Production debugging-এর সেরা tool।
- `* * *` দেখলে মানে সেই hop ICMP block করেছে - সমস্যা নাও হতে পারে।
- Packet loss 0% এবং latency কম থাকলে healthy connection।


## 🏋️ Practice Tasks

**Task 1:**
```bash
# Google-এ ঠিক 5টা packet পাঠান এবং output দেখুন
ping -c 5 google.com
# Packet loss কত % সেটা note করুন
```

**Task 2:**
```bash
# 8.8.8.8 (Google DNS) পর্যন্ত route trace করুন
traceroute -n 8.8.8.8
# কতটা hop লাগলো সেটা গণনা করুন
```

**Task 3:**
```bash
# mtr দিয়ে report বানান এবং file-এ save করুন
mtr --report --report-cycles 5 google.com > /tmp/network_report.txt
cat /tmp/network_report.txt
# কোন hop-এ সবচেয়ে বেশি latency সেটা খুঁজে বের করুন
```

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 4: DNS Tools**
`dig`, `nslookup`, `host` এবং `/etc/resolv.conf` - DNS কিভাবে কাজ করে এবং DNS সমস্যা কিভাবে debug করতে হয়, সেটা শিখবো। DevOps-এ DNS issue খুবই common - এটাতে master হতে পারলে অনেক সমস্যা সহজে সমাধান করতে পারবেন! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../02-Viewing-Network-Configuration">← Viewing Network Configuration</a>
    </td>
    <td align="right">
      <a href="../04-DNS-Tools">DNS Tools →</a>
    </td>
  </tr>
</table>