# Chapter 4 - Lesson 2: Viewing Network Configuration

**Chapter 4 | Lesson 2 of 11**


আপনাকে স্বাগতম Lesson 2-তে! 🎉 আগের lesson-এ আমরা networking-এর basics শিখেছিলাম - IP, MAC, DNS, Gateway, Ports। এখন আমরা শিখবো কিভাবে Linux-এ network configuration দেখতে হয়।

মনে করেন আপনি একজন doctor - রোগী দেখার আগে আপনি তার **vital signs** check করেন। একজন DevOps Engineer হিসেবে কোনো server-এ কাজ করার আগে আপনাকে সেই server-এর **network vital signs** check করতে হয়।


## আজকে আমরা যা শিখবো

| Tool | কী করে |
|------|---------|
| `ip` | Modern tool - IP address, routes, interfaces দেখায় |
| `ifconfig` | পুরনো classic tool - interface info দেখায় |
| `nmcli` | NetworkManager CLI - connections manage করে |
| `hostname` | System-এর নাম ও IP দেখায় |


## Tool 1: `ip` Command

`ip` হলো Linux networking-এর সবচেয়ে powerful ও modern tool। এটা দিয়ে আপনি প্রায় সব network-related কাজ করতে পারবেন।

### `ip addr` - IP Address দেখা

```bash
ip addr
```
অথবা short form:
```bash
ip a
```

**Expected Output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 08:00:27:ab:cd:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global ens33
    inet6 fe80::a00:27ff:feab:cdef/64 scope link
```

**এই output এর মানে কি?**

| অংশ | মানে |
|-----|------|
| `lo` | Loopback interface - এটা computer নিজেই নিজেকে। সবসময় `127.0.0.1` |
| `ens33` | আপনার actual network interface (Ethernet card) |
| `inet 192.168.1.100/24` | আপনার IPv4 address এবং subnet mask |
| `link/ether 08:00:27:...` | MAC address |
| `UP` | Interface চালু আছে |
| `mtu 1500` | Maximum Transmission Unit - একবারে কতটুকু data পাঠাতে পারবেন |

> `lo` হলো আপনার নিজের ঘরের ভেতরে কথা বলা। `ens33` হলো বাইরের দুনিয়ার সাথে connect হওয়ার দরজা।


### `ip addr show <interface>` - নির্দিষ্ট Interface দেখা

```bash
ip link # interface লিস্ট দেখা
ip addr show ens33 # নির্দিষ্ট interface দেখা
```

শুধু `ens33`-এর info দেখাবে। বড় server-এ অনেকগুলো interface থাকে, তখন এটা কাজে লাগে।


### `ip link` - Interface Status দেখা

```bash
ip link
```
অথবা:
```bash
ip l
```

**Expected Output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT
    link/ether 08:00:27:ab:cd:ef brd ff:ff:ff:ff:ff:ff
```

`ip link` মূলত interface-এর **physical layer status** দেখায় - UP নাকি DOWN।

**Interface UP/DOWN করা (DevOps কাজে লাগে):**

```bash
sudo ip link set ens33 down    # Interface বন্ধ করো
sudo ip link set ens33 up      # Interface চালু করো
```


### `ip route` - Routing Table দেখা

```bash
ip route
```
অথবা:
```bash
ip r
```

**Expected Output:**
```
default via 192.168.1.1 dev ens33 proto dhcp src 192.168.1.100 metric 100
192.168.1.0/24 dev ens33 proto kernel scope link src 192.168.1.100
```

**এই output এর মানে কি?**

| অংশ | মানে |
|-----|------|
| `default via 192.168.1.1` | Default gateway - অর্থাৎ অন্য সব traffic এই IP দিয়ে বের হবে |
| `dev ens33` | কোন interface দিয়ে বাইরে যাবে |
| `192.168.1.0/24` | এই network-এর জন্য directly ens33 দিয়ে যাবে |

> Routing table হলো আপনার **GPS map** - কোথাও যেতে হলে কোন রাস্তা নেবে তার নির্দেশনা।


### `ip -s link` - Network Statistics দেখা

```bash
ip -s link show ens33
```

**Expected Output:**
```
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    RX: bytes  packets  errors  dropped
    524288000  350000   0       0
    TX: bytes  packets  errors  dropped
    104857600  120000   0       0
```

| অংশ | মানে |
|-----|------|
| `RX` | Received - কতটুকু data আসছে |
| `TX` | Transmitted - কতটুকু data যাচ্ছে |
| `errors` | Error হয়েছে কিনা |
| `dropped` | Packet drop হয়েছে কিনা |

> ⚠️ **DevOps tip:** যদি `errors` বা `dropped` বাড়তে থাকে - network problem আছে!


## Tool 2: `ifconfig` Command

`ifconfig` হলো **পুরনো classic tool**। অনেক পুরনো system বা documentation-এ এটা দেখবেন। নতুন distro-তে default-এ নাও থাকতে পারে।

**Install করুন (যদি না থাকে):**
```bash
sudo apt install net-tools    # Ubuntu/Debian
sudo dnf install net-tools    # RHEL/Fedora
```

### সব Interface দেখা

```bash
ifconfig
```

**Expected Output:**
```
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::a00:27ff:feab:cdef  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:ab:cd:ef  txqueuelen 1000  (Ethernet)
        RX packets 350000  bytes 524288000 (524.2 MB)
        TX packets 120000  bytes 104857600 (104.8 MB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
```

### নির্দিষ্ট Interface দেখা

```bash
ifconfig ens33
```

### `ip` vs `ifconfig` - পার্থক্য

| বিষয় | `ip` | `ifconfig` |
|------|------|-----------|
| Status | Modern ✅ | পুরনো (deprecated) |
| Default available | হ্যাঁ | না (আলাদা install লাগে) |
| Features | বেশি | কম |
| DevOps use | বেশি ব্যবহৃত | পুরনো scripts-এ দেখবে |

> 💡 **পরামর্শ:** `ip` শিখুন, কিন্তু সেই সাথে `ifconfig`-এর output-ও চিনতে হবে কারণ পুরনো docs-এ এটাই দেখবেন।


## Tool 3: `nmcli` Command

`nmcli` মানে **NetworkManager Command Line Interface**। এটা শুধু দেখার জন্য না। এটা দিয়ে network **manage** করা যায়।

> যদি `ip` হয় আপনার **magnifying glass** (দেখার জন্য), তাহলে `nmcli` হলো আপনার **remote control** (control করার জন্য)।

### সব Connection দেখা

`nmcli` কাজ না করলে প্রথমে install করে নিতে হবে `sudo apt install network-manager`

```bash
nmcli connection show
```

**Expected Output:**
```
NAME       UUID                                  TYPE      DEVICE
Wired-1    a1b2c3d4-e5f6-7890-abcd-ef1234567890  ethernet  ens33
lo         b2c3d4e5-f6a7-8901-bcde-f01234567891  loopback  lo
```

### Device Status দেখা

```bash
nmcli device status
```

**Expected Output:**
```
DEVICE  TYPE      STATE      CONNECTION
ens33    ethernet  connected  Wired-1
lo      loopback  unmanaged  --
```

| Column | মানে |
|--------|------|
| `DEVICE` | Interface-এর নাম |
| `TYPE` | কী ধরনের (ethernet, wifi, etc.) |
| `STATE` | connected, disconnected, unavailable |
| `CONNECTION` | কোন connection profile ব্যবহার করছে |

### Detailed Device Info দেখা

```bash
nmcli device show ens33
```

**Expected Output (কিছু অংশ):**
```
GENERAL.DEVICE:                ens33
GENERAL.TYPE:                  ethernet
GENERAL.STATE:                 100 (connected)
IP4.ADDRESS[1]:                192.168.1.100/24
IP4.GATEWAY:                   192.168.1.1
IP4.DNS[1]:                    8.8.8.8
IP6.ADDRESS[1]:                fe80::a00:27ff:feab:cdef/64
```

এখানে একসাথে পাচ্ছো - IP address, Gateway, DNS সব কিছু! DevOps-এ এটা অনেক কাজে লাগে।

### WiFi Networks দেখা (যদি WiFi থাকে)

```bash
nmcli device wifi list
```

### Connection Restart করা

```bash
sudo nmcli connection down Wired-1
sudo nmcli connection up Wired-1
```


## Tool 4: `hostname` Command

`hostname` দিয়ে আপনার server-এর **নাম ও IP** জানতে পারেন।

### Hostname দেখা

```bash
hostname
```

**Output:**
```
myserver
```

### IP Address দেখা hostname দিয়ে

```bash
hostname -I
```

**Output:**
```
192.168.1.100 172.17.0.1
```

> এটা সব IP address দেখায় যদি আপনার server-এ Docker থাকে, তাহলে Docker-এর IP-ও দেখাবে।

### Full Hostname (FQDN) দেখা

```bash
hostname -f
```

**Output:**
```
myserver.example.com
```

### `hostnamectl` - Systemd দিয়ে Hostname দেখা ও পরিবর্তন করা

```bash
hostnamectl
```

**Expected Output:**
```
   Static hostname: myserver
         Icon name: computer-server
           Chassis: server
        Machine ID: a1b2c3d4e5f6789012345678
           Boot ID: b2c3d4e5f6a789012345679
  Operating System: Ubuntu 22.04.3 LTS
            Kernel: Linux 5.15.0-89-generic
      Architecture: x86-64
```

**Hostname পরিবর্তন করো:**
```bash
sudo hostnamectl set-hostname <New-Server-Name>
sudo hostnamectl set-hostname db-server
```


## সব Tools একসাথে - কখন কোনটা ব্যবহার করবো?

| পরিস্থিতি | ব্যবহার করো |
|-----------|------------|
| দ্রুত IP address দেখতে চাই | `ip a` বা `hostname -I` |
| Interface up/down আছে কিনা দেখতে চাই | `ip link` |
| Gateway কোথায় যাচ্ছে দেখতে চাই | `ip route` |
| Packet drop/error আছে কিনা দেখতে চাই | `ip -s link` |
| Network connection manage করতে চাই | `nmcli` |
| Server-এর নাম জানতে চাই | `hostname` |
| পুরনো script বুঝতে হচ্ছে | `ifconfig` চিনতে পারা |


## DevOps Real-World Scenario

একটা নতুন server পেলে আপনি প্রথমেই যা করবেন:

```bash
# Step 1: Server-এর নাম কী?
hostname

# Step 2: IP address কত?
ip a

# Step 3: Gateway কোথায়?
ip route

# Step 4: Network connected আছে কিনা?
nmcli device status

# Step 5: Packet error আছে কিনা?
ip -s link show ens33
```

এই 5টা command দিয়ে আপনি যেকোনো নতুন Linux server-এর network পুরোপুরি বুঝতে পারবেন।


## 📝 Quick Summary

- **`ip a`** → সব interface-এর IP address দেখায়
- **`ip link`** → Interface UP/DOWN status দেখায়
- **`ip route`** → Routing table ও gateway দেখায়
- **`ip -s link`** → Network statistics (RX/TX, errors) দেখায়
- **`ifconfig`** → পুরনো tool, কিন্তু চিনতে হবে
- **`nmcli`** → Network connections দেখা ও manage করা
- **`hostname`** → Server-এর নাম ও IP দেখা
- **`hostnamectl`** → Detailed system info ও hostname পরিবর্তন করা


## 🏋️ Practice Tasks

1. **`ip a`** রান করুন এবং আপনার system-এর সব interface-এর নাম ও IP address লিখে রাখুন।

2. **`ip route`** রান করুন এবং খুঁজে বের করুন আপনার **default gateway** কোনটা।

3. **`nmcli device show <আপনার interface>`** রান করুন এবং দেখুন কোন DNS server ব্যবহার হচ্ছে।

---

## ⏭️ What's Next

**Chapter 4 - Lesson 3: Testing Connectivity**
`ping`, `traceroute`, `mtr` - এই tools দিয়ে আপনি জানতে পারবেন আপনার server internet-এ পৌঁছাতে পারছে কিনা, এবং কোথায় network problem হচ্ছে। DevOps-এ network debugging-এর জন্য এগুলো অপরিহার্য! *Happy Learning* 🚀