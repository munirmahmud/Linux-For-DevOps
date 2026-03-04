# Chapter 1 - Lesson 1: What is Linux? History, Kernel, Distros & Why DevOps Uses It

✅ **Progress: Chapter 1 | Lesson 1 of 10**


## 🐧 Linux কী?

সহজ ভাষায় বলতে গেলে **Linux হলো একটি Operating System (OS)**।

আপনি যেমন Windows বা macOS ব্যবহার করেছেন, Linux-ও ঠিক তেমনই একটি OS। কিন্তু Linux-এর সাথে তাদের পার্থক্য হলো:

| বিষয় | Windows | Linux |
|---|---|---|
| Cost | Paid 💰 | Free ✅ |
| Source Code | Closed (দেখা যায় না) | Open Source (সবাই দেখতে পারে) |
| Customization | সীমিত | সম্পূর্ণ নিজের মতো বানানো যায় |
| Security | তুলনামূলক দুর্বল | অনেক বেশি Secure |
| Server Use | কম | ৯০%+ Server Linux চালায় |



## 📖 Linux-এর ইতিহাস - একটা ছোট গল্প

**১৯৬৯ সালে** Bell Labs-এ **Unix** নামে একটি OS তৈরি হয়। এটা ছিল অনেক powerful, কিন্তু এটা ছিল paid এবং closed।

**১৯৮৩ সালে** Richard Stallman **GNU Project** শুরু করেন, লক্ষ্য ছিল একটি free OS বানানো। কিন্তু তার একটা জিনিস missing ছিল আর তাহলো **Kernel**।

**১৯৯১ সালে** Finland-এর একজন ২১ বছর বয়সী university student **Linus Torvalds** একটি মেসেজ পোস্ট করলেন:

> *"I'm doing a (free) operating system (just a hobby, won't be big and professional like gnu)..."*

এটাই ছিল **Linux Kernel**-এর জন্ম! GNU + Linux Kernel মিলে তৈরি হলো আজকের Linux OS।



## 🧠 Kernel কী? সবচেয়ে গুরুত্বপূর্ণ concept

**Analogy দিয়ে বুঝি:**

> কল্পনা করুন আপনি একটি company-র **CEO**। আপনি directly office-এর প্রতিটা কাজ করেন না। আপনার একজন **Manager** আছে যে কর্মীদের কাজ দেয়, resource manage করে, আর সব কিছু coordinate করে।
>
> এখানে:
> - **আপনি (CEO)** = User / Application
> - **Manager** = **Kernel**
> - **কর্মীরা** = Hardware (CPU, RAM, Disk, Network)

**Kernel হলো Linux OS-এর core/heart।** এটা:
- আপনার application এবং hardware-এর মধ্যে **সেতু** হিসেবে কাজ করে
- CPU, RAM, Disk, Network সব কিছু **manage** করে
- কে কতটুকু resource পাবে সেটা **নিয়ন্ত্রণ** করে

```
┌─────────────────────────────────┐
│         User Applications       │  ← আমরা যা চালাই (nginx, docker, bash)
├─────────────────────────────────┤
│           Shell / CLI           │  ← আমরা যেখানে command লিখি
├─────────────────────────────────┤
│         Linux Kernel            │  ← সব কিছুর মস্তিষ্ক 🧠
├─────────────────────────────────┤
│    Hardware (CPU, RAM, Disk)    │  ← Physical machine
└─────────────────────────────────┘
```

## 🌍 Linux Distributions (Distros) কী?

**Kernel একা কোনো কাজের না।** Kernel-এর সাথে যখন tools, package manager, desktop environment ইত্যাদি যোগ করা হয় সেটাকে বলে **Distribution বা Distro**।

**Analogy:**
> Kernel হলো একটা **গাড়ির Engine**। শুধু engine দিয়ে গাড়ি চালানো যায় না, তার সাথে চাই body, seat, steering, AC। বিভিন্ন company এই engine-কে নিয়ে আলাদা আলাদা গাড়ি বানায়। কেউ বানায় sports car, কেউ truck, কেউ আবার প্রাইভেট car ইত্যাদি।
>
> এই আলাদা আলাদা গাড়িগুলোই হলো **Distros**।

### 🔥 Famous Distros এবং DevOps-এ তাদের ব্যবহার:

| Distro | Family | কোথায় ব্যবহার হয় |
|---|---|---|
| **Ubuntu** | Debian | সবচেয়ে popular, beginners + production servers |
| **CentOS / Rocky Linux** | Red Hat | Enterprise servers, কোম্পানির production |
| **RHEL** (Red Hat Enterprise) | Red Hat | বড় কোম্পানি, paid support |
| **Debian** | Debian | Stable servers, Docker base images |
| **Amazon Linux** | Red Hat | AWS-এ default |
| **Alpine Linux** | Independent | Docker containers (খুব ছোট, ~5MB) |
| **Kali Linux** | Debian | Security / Penetration Testing |
| **Arch Linux** | Independent | Advanced users, cutting-edge |

**Production-এ সবচেয়ে বেশি ব্যাবহার হয়:** Ubuntu, CentOS/Rocky, Amazon Linux, Alpine (Docker-এ)


## ⚙️ Production-এ কেন Linux বেশী ব্যবহার করে?

এটা জানা আমাদের জন্য অনেক জরুরি কারণ আমরা DevOps Engineer/System Admin হওয়ার পথে হাঁটছি।

### ১. 🖥️ Servers Run Linux
বিশ্বের **৯০%+ web server** Linux চালায়। Google, Amazon, Facebook, Netflix সবাই Linux ব্যবহার করে।

### ২. 🐳 Docker & Kubernetes = Linux
Docker containers মূলত **Linux Kernel features** (namespaces, cgroups) ব্যবহার করে। Kubernetes-ও Linux-এর উপর নির্মিত।

### ③. ⚡ Performance & Stability
Linux server মাসের পর মাস, বছরের পর বছর **restart ছাড়া** চলতে পারে।

### ৪. 🔒 Security
Linux-এ permission system অনেক strong। DevOps-এ security অনেক গুরুত্বপূর্ণ।

### ৫. 🛠️ Automation
Shell scripting, cron jobs, systemd ইত্যাদি Linux-এ automation করা অনেক সহজ এবং powerful।

### ৬. 💰 Free & Open Source
Cloud-এ হাজারটা server চালাতে হলে Windows licensing-এ লক্ষ লক্ষ টাকা লাগবে। Linux-এ **zero cost**।


## 🖥️ Linux Interface Terminal কেন?

আমরা Windows-এ mouse দিয়ে click করে কাজ করতে পারি। Linux-এও GUI আছে বা Windows-এর মতো ইন্টারফেইস আছে কিন্তু DevOps-এ আমরা **Terminal / Command Line Interface (CLI)** ব্যবহার করি।

**কেন?**
- Server-এ সাধারণত কোনো **graphical interface থাকে না** কারন GUI অনেক রিসোর্স consume করে আর প্রডাকশনে লিমিটেড রিসোর্স।
- CLI অনেক **দ্রুত এবং powerful**
- **Automation** করা যায় (script লিখে হাজারটা কাজ একসাথে)
- **Remote server**-এ SSH দিয়ে connect করলে শুধু terminal পাবে


## 📝 Quick Summary

- ✅ Linux হলো একটি **free, open-source Operating System**
- ✅ **Linus Torvalds** ১৯৯১ সালে Linux Kernel তৈরি করেন
- ✅ **Kernel** হলো OS-এর core, hardware ও software-এর মধ্যে সেতুর মতো কাজ করে।
- ✅ **Distro** = Kernel + tools + package manager (Ubuntu, CentOS, etc.)
- ✅ DevOps-এ Linux অপরিহার্য কারণ: servers, Docker, Kubernetes, automation, security, cost
- ✅ DevOps Engineer হিসেবে আমরা মূলত **Terminal/CLI** ব্যবহার করবো।


## 🏋️ Practice Tasks

এখন আপনার কাছে যদি Linux terminal থাকে (Ubuntu VM, WSL, বা যেকোনো Linux), এই কাজগুলো করে দেখেন:

**Task 1:** Kernel version দেখার জন্য
```bash
uname -r
```

**Task 2:** আপনি কোন distro ব্যবহার করছেন সেটা দেখা যায়।
```bash
cat /etc/os-release
```

**Task 3:** System কতক্ষণ ধরে চলছে তা দেখা যায়।
```bash
uptime
```

এই commands এখন বুঝতে না পারলেও সমস্যা নেই, আমরা সামনে সব শিখব। শুধু চালিয়ে দেখেন কী output আসে! 😊

---

## ⏭️ What's Next?

**Lesson 2: Linux File System Hierarchy**
> `/`, `/etc`, `/var`, `/proc`, `/sys`, `/tmp`, `/home` Linux-এ files কোথায় কোথায় থাকে এবং কেন সেই structure এতটা important।
