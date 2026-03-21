# Chapter 4 - Lesson 11: Network Bonding, Bridging & VLANs

**Chapter 4 | Lesson 11 of 11**

Chapter 4 এর শেষ লেসন এবং সবচেয়ে Advanced Lesson!


## 🎯 এই Lesson এ আমরা যা শিখবো

- **Network Bonding** - একাধিক NIC কে একসাথে জুড়ে High Availability বা Speed বাড়ানো
- **Network Bridging** - Virtual Switch তৈরি করা (VM/Container এর জন্য)
- **VLANs** - একটা Physical Network কে আলাদা Logical Network এ ভাগ করা

এগুলো সব DevOps/SysAdmin level এর real-world networking concept। চলুন শুরু করি!


## PART 1: Network Bonding

### Network Bonding কী?

ধরুন আপনার কাছে দুটো Network Card (NIC) আছে `eth0` আর `eth1`। আপনি চাইতেছেন:
- যদি একটা NIC নষ্ট হয়ে যায়, তাহলেও network চালু থাকুক **(Failover)**
- অথবা দুটো NIC মিলিয়ে Speed দ্বিগুণ করতে চান **(Load Balancing)**

**Bonding** মানে হলো এই দুটো NIC কে একটা **virtual interface** এ বেঁধে ফেলা যেটার নাম হয় `bond0`।

```
┌──────────────────────────────────┐
│          bond0 (virtual)         │
│    IP: 192.168.1.10              │
└────────────┬─────────────────────┘
             │
    ┌────────┴────────┐
    │                 │
  eth0              eth1
(Physical)        (Physical)
```


### Bonding Modes - কোনটা কীভাবে কাজ করে?

| Mode | নাম | কাজ | কখন ব্যবহার |
|------|-----|-----|-------------|
| 0 | balance-rr | Round-robin - পালাক্রমে packet পাঠায় | Speed বাড়াতে |
| 1 | active-backup | একটা active, একটা backup | High Availability |
| 2 | balance-xor | MAC address দিয়ে বেছে নেয় | Load Balancing |
| 4 | 802.3ad (LACP) | Switch এর সাথে মিলিয়ে কাজ করে | Data Center |
| 5 | balance-tlb | Outgoing load balance করে | সাধারণ ব্যবহার |
| 6 | balance-alb | In+Out দুটোই balance করে | Advanced |

> **DevOps এ সবচেয়ে বেশি ব্যবহার:** Mode 1 (active-backup) এবং Mode 4 (LACP)


### Bonding Setup করা (Ubuntu/Debian - netplan দিয়ে)

আধুনিক Ubuntu তে **netplan** ব্যবহার করা হয়। Config file থাকে `/etc/netplan/` এ।

**Step 1: Config file দেখুন**
```bash
ls /etc/netplan/
# Output: 00-installer-config.yaml বা 50-cloud-init.yaml
```

**Step 2: Bonding config লিখুন**
```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

Distro অনুযায়ী ফাইলের নাম ভিন্ন হতে পারে

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false    # eth0 কে slave হিসেবে দিবো
    eth1:
      dhcp4: false    # eth1 কেও slave হিসেবে দিবো
  bonds:
    bond0:
      interfaces: [eth0, eth1]   # এই দুটো NIC bond করবো
      addresses: [192.168.1.10/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
      parameters:
        mode: active-backup      # Mode 1 - failover
        primary: eth0            # eth0 প্রথমে active থাকবে
        mii-monitor-interval: 100  # প্রতি 100ms এ NIC check করবে
```

**Step 3: নতুন পরিবর্তনগুলো Apply করা**
```bash
sudo netplan apply
```

**Step 4: Bond interface দেখা**
```bash
ip addr show bond0
# Output:
# 3: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP>
#     link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
#     inet 192.168.1.10/24 brd 192.168.1.255 scope global bond0

cat /proc/net/bonding/bond0
# Output:
# Ethernet Channel Bonding Driver: v3.7.1
# Bonding Mode: fault-tolerance (active-backup)
# Primary Slave: eth0 (primary_reselect failure)
# Currently Active Slave: eth0
# MII Status: up
# MII Polling Interval (ms): 100
# Slave Interface: eth0
#   MII Status: up
# Slave Interface: eth1
#   MII Status: up
```

> `/proc/net/bonding/bond0` - এটা দিয়ে bond এর real-time status দেখা যায়। কোনটা active, কোনটা backup সব দেখাবে।


### Bonding - RHEL/CentOS এ (nmcli দিয়ে)

```bash
# Step 1: bond0 interface তৈরি করুন
sudo nmcli con add type bond \
    con-name bond0 \
    ifname bond0 \
    bond.options "mode=active-backup,miimon=100"

# Step 2: eth0 কে slave হিসেবে যোগ করুন
sudo nmcli con add type ethernet \
    con-name bond0-slave-eth0 \
    ifname eth0 \
    master bond0

# Step 3: eth1 কেও slave হিসেবে যোগ করুন
sudo nmcli con add type ethernet \
    con-name bond0-slave-eth1 \
    ifname eth1 \
    master bond0

# Step 4: IP assign করুন
sudo nmcli con mod bond0 \
    ipv4.addresses "192.168.1.10/24" \
    ipv4.gateway "192.168.1.1" \
    ipv4.method manual

# Step 5: Connection চালু করুন
sudo nmcli con up bond0
```


## PART 2: Network Bridging

### Bridge কী?

**Bridge** হলো একটা **virtual switch** - ঠিক যেন একটা physical network switch, কিন্তু software দিয়ে তৈরি।

**কোথায় ব্যবহার হয়?**
- **KVM/QEMU Virtual Machines** - VM গুলো যাতে physical network এ access পায়
- **Docker/LXC Containers** - container network তৈরিতে
- **Network Testing Labs** - virtual network তৈরি করতে

```
Physical Network (192.168.1.0/24)
          │
    ┌─────┴──────┐
    │   br0      │  ← Virtual Bridge (Software Switch)
    │ (bridge)   │
    └──┬───┬───┬─┘
       │   │   │
     eth0  VM1  VM2
  (Physical)
```

এখানে `eth0`, `VM1`, `VM2` সবাই একই network এ আছে। `br0` এদের সবাইকে connect করছে।


### Bridge তৈরি করা

**Ubuntu - netplan দিয়ে:**

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false          # eth0 কে bridge এর slave বানাবো
  bridges:
    br0:
      interfaces: [eth0]    # eth0 কে bridge এ যোগ করবো
      addresses: [192.168.1.20/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
      parameters:
        stp: true           # Spanning Tree Protocol (loop prevention)
        forward-delay: 4    # Bridge start হতে 4 second নেবে
```

```bash
sudo netplan apply

# Bridge status দেখা
ip addr show br0
bridge link show
```

**Manual ভাবে (ip command দিয়ে):**

```bash
# Bridge তৈরি করা
sudo ip link add name br0 type bridge

# eth0 কে bridge এ যোগ করা (slave করুন)
sudo ip link set eth0 master br0

# Bridge চালু করা
sudo ip link set br0 up

# IP দিন
sudo ip addr add 192.168.1.20/24 dev br0

# দেখুন
bridge link show
# Output:
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> master br0 state forwarding ...
```

### Bridge দেখার commands:

```bash
# সব bridge দেখা
brctl show
# Output:
# bridge name  bridge id          STP enabled  interfaces
# br0          8000.001122334455  yes          eth0

# Bridge এর MAC address table দেখুন (কোন MAC কোন port এ আছে)
brctl showmacs br0

# আধুনিক পদ্ধতি
bridge fdb show br0
bridge link show
```


## PART 3: VLANs (Virtual Local Area Networks)

### VLAN কী?

ধরুন একটা office-এ ৩টা department যেমন HR, Finance, Engineering। সবাই একই physical network switch এ connected। কিন্তু আপনি চান যে HR এর traffic Finance যাতে দেখতে না পারে।

এটা করার দুটো উপায়:
1. ৩টা আলাদা physical switch ক্রয় করা - **expensive!**
2. **VLAN** ব্যবহার করুন। একটাই switch, কিন্তু logically আলাদা নেটওয়ার্ক - **smart!**

```
Physical Switch (একটাই)
│
├── VLAN 10 → HR Department      (10.0.10.0/24)
├── VLAN 20 → Finance Department (10.0.20.0/24)
└── VLAN 30 → Engineering        (10.0.30.0/24)
```

প্রতিটা VLAN আলাদা network। এদের মধ্যে traffic যেতে পারে না (router ছাড়া)।


### VLAN Tagging কী?

Network packet এ একটা **tag (4 bytes)** লাগানো হয় যাতে বোঝা যায় এই packet কোন VLAN এর।

```
Normal Ethernet Frame:
[Destination MAC | Source MAC | EtherType | Payload]

VLAN Tagged Frame (802.1Q):
[Destination MAC | Source MAC | 0x8100 | VLAN ID | EtherType | Payload]
                                          ↑
                                    এটাই VLAN Tag
                                    (12 bits = VLAN 1-4094)
```

### VLAN Setup করা Linux এ

**প্রথমে `8021q` kernel module load করুন:**

```bash
sudo modprobe 8021q

# Check করুন module load হয়েছে কিনা
lsmod | grep 8021q
# Output:
# 8021q                  32768  0
# garp                   16384  1 8021q
```

**VLAN Interface তৈরি করা (ip command দিয়ে):**

```bash
# eth0 এর উপর VLAN 10 তৈরি করুন
sudo ip link add link eth0 name eth0.10 type vlan id 10

# eth0 এর উপর VLAN 20 তৈরি করুন
sudo ip link add link eth0 name eth0.20 type vlan id 20

# Interface গুলো চালু করুন
sudo ip link set eth0.10 up
sudo ip link set eth0.20 up

# IP assign করুন
sudo ip addr add 10.0.10.1/24 dev eth0.10
sudo ip addr add 10.0.20.1/24 dev eth0.20

# দেখুন
ip addr show eth0.10
# Output:
# 4: eth0.10@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
#     link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
#     inet 10.0.10.1/24 scope global eth0.10

# VLAN info দেখুন
cat /proc/net/vlan/eth0.10
# Output:
# eth0.10  VID: 10   REORDER_HDR: 1  dev->priv_flags: 1001
#          total frames received            0
#          total bytes received             0
```


### VLAN Setup - netplan দিয়ে (Permanent):

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
  vlans:
    eth0.10:
      id: 10
      link: eth0
      addresses: [10.0.10.1/24]
    eth0.20:
      id: 20
      link: eth0
      addresses: [10.0.20.1/24]
    eth0.30:
      id: 30
      link: eth0
      addresses: [10.0.30.0/24]
```

```bash
sudo netplan apply

# দেখুন
ip addr show eth0.10
ip addr show eth0.20
```


### VLAN - nmcli দিয়ে (RHEL/CentOS):

```bash
# VLAN 10 তৈরি করুন
sudo nmcli con add type vlan \
    con-name vlan10 \
    ifname eth0.10 \
    dev eth0 \
    id 10 \
    ipv4.addresses "10.0.10.1/24" \
    ipv4.method manual

# VLAN 20 তৈরি করুন
sudo nmcli con add type vlan \
    con-name vlan20 \
    ifname eth0.20 \
    dev eth0 \
    id 20 \
    ipv4.addresses "10.0.20.1/24" \
    ipv4.method manual

# Connection চালু করুন
sudo nmcli con up vlan10
sudo nmcli con up vlan20
```


## Bonding + VLAN একসাথে (Real-World DevOps Setup)

Data center এ এই combination অনেক বেশি দেখা যায়:

```
Physical Server
    ├── eth0 ──┐
    │           ├── bond0 (active-backup)
    └── eth1 ──┘
                    │
                    ├── bond0.10 (VLAN 10 - Management)
                    ├── bond0.20 (VLAN 20 - Production)
                    └── bond0.30 (VLAN 30 - Storage)
```

**netplan config:**
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
    eth1:
      dhcp4: false
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
  vlans:
    bond0.10:
      id: 10
      link: bond0
      addresses: [10.0.10.1/24]
    bond0.20:
      id: 20
      link: bond0
      addresses: [10.0.20.1/24]
```


## Useful Commands - Quick Reference

```bash
# === BONDING ===
cat /proc/net/bonding/bond0          # Bond status দেখুন
ip addr show bond0                   # Bond IP দেখুন
ip link show bond0                   # Bond link status

# === BRIDGING ===
brctl show                           # সব bridge দেখুন
bridge link show                     # Bridge ports দেখুন
bridge fdb show br0                  # MAC table দেখুন
ip addr show br0                     # Bridge IP দেখুন

# === VLAN ===
cat /proc/net/vlan/                  # সব VLAN দেখুন
ip -d link show eth0.10              # VLAN details দেখুন
vconfig add eth0 10                  # পুরনো পদ্ধতিতে VLAN (deprecated)
```


## 📝 Quick Summary

- **Bonding** - একাধিক NIC কে একটা virtual interface এ জুড়ে Failover বা Speed বাড়ানো যায়। Mode 1 (active-backup) সবচেয়ে সহজ ও নিরাপদ।
- **Bridging** - Virtual Switch তৈরি করে VM/Container কে physical network এ যুক্ত করা হয়। Docker ও KVM এটাই ব্যবহার করে।
- **VLAN** - একটা Physical network কে logically আলাদা করা হয় VLAN tag দিয়ে। প্রতিটা VLAN আলাদা subnet - traffic isolate থাকে।
- **`8021q` module** - VLAN support এর জন্য অবশ্যই load করতে হবে।
- **netplan/nmcli** - permanent config এর জন্য এগুলো ব্যবহার করুন, `ip` command শুধু temporary।


## 🏋️ Practice Tasks

1. `/proc/net/bonding/` folder টা দেখুন যদি কোনো bonding না থাকে তাহলে কী দেখায়?

   ```bash
   ls /proc/net/bonding/
   cat /proc/net/vlan/   # VLAN ও দেখুন
   ```

2. `8021q` module load এবং verify করুন:
   ```bash
   sudo modprobe 8021q
   lsmod | grep 8021q
   ```

3. আপনার system এর network interface দেখুন এবং কোনটা দিয়ে VLAN তৈরি করা যাবে সেটা চিহ্নিত করুন:
   ```bash
   ip link show
   ip addr show
   ```


Congratulations! 🎉 আপনি Chapter 4-এর Networking in Linux এর ১১টা Lesson সব শেষ করেছেন! 🏆

আপনি Chapter 4 সম্পন্ন করেছেন! Linux-এ এই লেসনগুলো অনেকগুরুত্বপূর্ণ এবং বলা যায় প্রায় প্রতিদিনই এগুলো আমাদের ইউজ করতে হয়। এই লেসনের পরে অবশ্যই Assesment করবেন যা এই লেসনের উপর ভিত্তি করেই তৈরি করেছি। এতোটুকু শেখার পরে আমার পরামর্শ থাকবে এই চারটি চ্যাপ্টার আবার নতুন করে রিভিশন দেয়ার জন্য। তাহলে সামনের লেসনগুলো আমাদের বুঝতে কোন সমস্যা হবে না। কারন এই লেসনগুলোই সবক্ষেত্রেই ইউজ হয়।


### Chapter 4 এ আমরা যা শিখেছি:
- Networking Basics (IP, MAC, DNS, Gateway)
- Network Config দেখা (ip, ifconfig, nmcli)
- Connectivity Test (ping, traceroute, mtr)
- DNS Tools (dig, nslookup)
- File Transfer (curl, wget, scp, rsync)
- Open Ports (ss, netstat, nmap)
- SSH Mastery
- Firewall (iptables, ufw, firewalld)
- tcpdump
- Network Config Files
- **Bonding, Bridging & VLANs** ← আজকের lesson


## ⏭️ What's Next

**Chapter 5 - Lesson 1: What is Shell Scripting?**
*(bash, sh, shebang, executing scripts)*

Shell Scripting শিখলে আপনি নিজেই automation tool বানাতে পারবেন। এটা DevOps এর সবচেয়ে exciting part! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../10-networking-configuration-file">← Networking Configuration File</a>
    </td>
    <td align="right">
      <a href="../12-Assesments">Chapter 4 - Assesments →</a>
    </td>
  </tr>
</table>