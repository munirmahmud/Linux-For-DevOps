# Chapter 4 - Lesson 5: Downloading & Transferring Files

এই লেসনে আমরা শিখবো `curl`, `wget`, `scp`, `rsync` এর বিস্তারিত


**Chapter 4 | Lesson 5 of 11**


আপনাকে স্বাগতম এই lesson 5-এ! 🎉

একজন DevOps Engineer হিসেবে আপনাকে প্রতিদিন files download করতে হবে, servers-এর মধ্যে files transfer করতে হবে, APIs test করতে হবে। এই কাজগুলো করার জন্য Linux-এ ৪টি শক্তিশালী tool আছে:

- **curl** - HTTP requests করা, API test করা, files download করা
- **wget** - Internet থেকে files download করা
- **scp** - SSH দিয়ে securely files copy করা
- **rsync** - Smart file synchronization/transfer

চলুন একটা একটা করে শিখি!


## 1. curl - Client URL Tool

### curl কী?

**curl** হলো একটি tool যেটা দিয়ে আপনি internet-এ যেকোনো URL-এ request পাঠাতে এবং response দেখতে পারেন। শুধু file download না - API call করা, form submit করা, headers দেখা - সব কিছু curl দিয়ে হয়।

> curl হলো আপনার browser-এর মতো, মানে এটা terminal-এর browser। Browser যেভাবে একটা website-কে request করে response দেখায়, curl ঠিক সেটাই করে - শুধু graphical UI ছাড়া।


### Basic Syntax

```bash
curl [options] [URL]
```


### সবচেয়ে simple ব্যবহার - একটা URL-এর content দেখা

```bash
curl https://example.com
```

**Output:**
```html
<!doctype html>
<html>
<head>
    <title>Example Domain</title>
...
</html>
```

এটা সেই URL-এর HTML content terminal-এ দেখাবে।


### File Download করা - `-O` flag

```bash
curl -O https://wordpress.org/latest.zip
```

- `-O` মানে: URL-এর filename দিয়েই file save করো (এখানে `latest.zip` নামে save হবে)

```bash
curl -o wordpress.zip https://wordpress.org/latest.zip
```

- `-o` (lowercase) মানে: আপনি নিজে file-এর নাম বলে দিতে পারবেন (`wordpress.zip`)


### Download Progress দেখা

```bash
curl -O --progress-bar https://example.com/bigfile.zip
```

**Output:**
```
###########################                        55.2%
```


### Redirect Follow করা - `-L` flag

অনেক সময় একটা URL আরেকটা URL-এ redirect করে। `-L` দিলে curl সেই redirect follow করে।

```bash
curl -L https://bit.ly/some-short-url
```

DevOps-এ অনেক download link redirect করে, তাই `-L` প্রায়ই দরকার হয়।


### HTTP Headers দেখা - `-I` flag

```bash
curl -I https://example.com
```

**Output:**
```
HTTP/2 200
content-type: text/html; charset=UTF-8
server: ECS (nyb/1D2E)
content-length: 1256
```

শুধু headers দেখায়, body নয়। DevOps-এ server response check করতে এটা অনেক কাজে লাগে।


### API Call করা - DevOps-এর সবচেয়ে বড় ব্যবহার

**GET request:**
```bash
curl https://jsonplaceholder.typicode.com/posts/1
```

**Output:**
```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat...",
  "body": "quia et suscipit..."
}
```


**POST request - data পাঠানো:**
```bash
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "DevOps Test", "body": "Hello!", "userId": 1}'
```

- `-X POST` → HTTP method POST ব্যবহার করো
- `-H` → Header যোগ করো
- `-d` → Data/body পাঠাও


### Authentication সহ request

```bash
curl -u username:password https://api.example.com/data
```

অথবা Bearer token দিয়ে:
```bash
curl -H "Authorization: Bearer your_token_here" https://api.example.com/data
```

### Verbose mode - কী হচ্ছে সব দেখা

```bash
curl -v https://example.com
```

Connection থেকে শুরু করে headers, TLS handshake সব কিছু দেখাবে। Debugging-এ অনেক কাজে লাগে।


### curl-এর Common Options একসাথে

| Option | কাজ |
|--------|-----|
| `-O` | URL-এর নামে file save করো |
| `-o filename` | নিজের দেওয়া নামে save করো |
| `-L` | Redirect follow করো |
| `-I` | শুধু headers দেখাও |
| `-v` | Verbose - সব details দেখাও |
| `-s` | Silent mode - progress দেখাবে না |
| `-X` | HTTP method specify করো (GET/POST/PUT/DELETE) |
| `-H` | Custom header যোগ করো |
| `-d` | POST data পাঠাও |
| `-u` | Username:password দিয়ে authenticate করো |
| `--progress-bar` | সুন্দর progress bar দেখাও |


## 2. wget - Web Get

### wget কী?

**wget** হলো specifically file download করার জন্য তৈরি tool। curl-এর চেয়ে wget file downloading-এ বেশি সহজ ও powerful।

> curl হলো Swiss Army Knife (সব কাজ করে), আর wget হলো specifically download করার জন্য তৈরি একটি dedicated download manager।


### Basic Syntax

```bash
wget [options] [URL]
```


### Simple File Download

```bash
wget https://wordpress.org/latest.zip
```

curl-এর `-O` এর মতো - automatically URL-এর নামে save করে।

**Output:**
```
--2024-01-15 10:30:00--  https://wordpress.org/latest.zip
Resolving example.com... 93.184.216.34
Connecting to example.com|93.184.216.34|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5120 (5.0K) [application/x-tar]
Saving to: 'file.tar.gz'

file.tar.gz    100%[===================>]   5.00K  --.-KB/s    in 0.001s
```


### নিজের নামে Save করা

```bash
wget -O wordpress.zip https://wordpress.org/latest.zip
```


### Background-এ Download করা

```bash
wget -b https://example.com/bigfile.iso
```

- `-b` মানে background-এ চলবে
- Download progress `wget-log` file-এ লেখা হবে

```bash
tail -f wget-log    # progress দেখতে
```


### Incomplete Download Resume করা - DevOps-এ অনেক কাজে লাগে

```bash
wget -c https://example.com/bigfile.iso
```

- `-c` মানে continue - আগের বার যেখানে ছিল সেখান থেকে শুরু করো
- বড় file download করার সময় connection ছিঁড়ে গেলে এটা জীবন বাঁচায়! 😄


### Recursive Download - পুরো website বা directory নামানো

```bash
wget -r https://example.com/docs/
```

- `-r` মানে recursive - সব linked files সহ download করো

```bash
wget -r --no-parent https://example.com/docs/
```

- `--no-parent` মানে parent directory-তে যাবে না - শুধু `/docs/` এর ভেতরেরগুলো নামাবে


### Multiple Files Download করা

একটা text file বানান যেখানে প্রতি line-এ একটা URL থাকে।

```
# urls.txt
https://example.com/file1.tar.gz
https://example.com/file2.tar.gz
https://example.com/file3.tar.gz
```

তারপর:
```bash
wget -i urls.txt
```

সব গুলো একসাথে download হবে।


### wget Common Options

| Option | কাজ |
|--------|-----|
| `-O filename` | নিজের নামে save করো |
| `-b` | Background-এ download |
| `-c` | Resume incomplete download |
| `-r` | Recursive download |
| `-i file` | File থেকে URLs পড়ে download করো |
| `-q` | Quiet mode - কিছু print করবে না |
| `--limit-rate=100k` | Download speed limit করো |
| `--tries=3` | কতবার retry করবে |


### curl vs wget - কখন কোনটা ব্যবহার করবো?

| পরিস্থিতি | কোনটা ব্যবহার করবো |
|-----------|-------------------|
| API test/call করতে হবে | **curl** |
| File download করতে হবে | **wget** |
| Interrupted download resume করতে হবে | **wget** (`-c`) |
| POST/PUT/DELETE request করতে হবে | **curl** |
| Headers দেখতে হবে | **curl** (`-I`) |
| Background download | **wget** (`-b`) |
| Script-এ HTTP response process করতে হবে | **curl** |


## 3. scp - Secure Copy Protocol

### scp কী?

**scp** হলো SSH-এর উপর দিয়ে securely files copy করার tool। মানে একটা server থেকে আরেকটা server-এ, অথবা আপনার local machine থেকে remote server-এ files পাঠাতে পারবেন। সব কিছু encrypted।

> scp হলো একটা armored van-এর মতো। আপনি যখন দুই জায়গার মধ্যে valuable জিনিস (files) পাঠান, সেগুলো locked ও encrypted থাকে। কেউ মাঝপথে দেখতে পারবে না।


### Basic Syntax

```bash
scp [options] source destination
```

Source বা destination যেকোনোটা হতে পারে:

- **Local path:** `/home/user/file.txt`
- **Remote path:** `user@server_ip:/path/to/destination`


### Local থেকে Remote Server-এ File পাঠানো (Upload)

```bash
scp /home/devops/app.tar.gz ubuntu@192.168.1.100:/home/ubuntu/
```

**ব্যাখ্যা:**
- `/home/devops/app.tar.gz` → local file
- `ubuntu@192.168.1.100` → remote server-এর user এবং IP
- `:/home/ubuntu/` → remote server-এ কোথায় রাখবো

**Output:**
```
app.tar.gz                              100% 5120     5.0MB/s   00:01
```


### Remote Server থেকে Local-এ Download করা

```bash
scp ubuntu@192.168.1.100:/var/log/nginx/access.log /home/devops/logs/
```

Remote server-এর `access.log` file আপনার local machine-এ নামিয়ে আনবে।


### SSH Key দিয়ে scp (Password ছাড়া)

```bash
scp -i ~/.ssh/my_key.pem app.tar.gz ubuntu@192.168.1.100:/home/ubuntu/
```

- `-i` মানে identity file (SSH private key)
- AWS EC2 তে সব সময় এভাবে করতে হয়


### পুরো Directory Copy করা - `-r` flag

```bash
scp -r /home/devops/myproject ubuntu@192.168.1.100:/home/ubuntu/
```

- `-r` মানে recursive - folder সহ সব কিছু copy হবে


### Different Port ব্যবহার করা - `-P` flag

Default SSH port হলো 22। কিন্তু অনেক server different port ব্যবহার করে:

```bash
scp -P 2222 file.txt ubuntu@192.168.1.100:/home/ubuntu/
```

⚠️ **মনে রাখুন:** scp-তে port-এর জন্য capital `-P` ব্যবহার হয়!


### Server থেকে Server-এ Copy করা

```bash
scp ubuntu@server1:/home/ubuntu/file.txt ubuntu@server2:/home/ubuntu/
```

আপনার machine মাঝখান দিয়ে যাবে কিন্তু দুটো server-এর মধ্যে file transfer হবে।


### scp Common Options

| Option | কাজ |
|--------|-----|
| `-r` | Directory recursively copy করো |
| `-i keyfile` | SSH key দিয়ে authenticate করো |
| `-P port` | Different SSH port ব্যবহার করো |
| `-p` | File-এর permissions ও timestamps preserve করো |
| `-v` | Verbose - debugging তথ্য দেখাও |
| `-C` | Compression enable করো |


## 4. rsync - The Smart File Synchronizer

### rsync কী?

**rsync** হলো সবচেয়ে intelligent file transfer tool। এটা শুধু নতুন বা পরিবর্তিত files copy করে। সব কিছু আবার copy করে না। এই কারণে এটা অনেক দ্রুত এবং bandwidth সাশ্রয়ী।

> আপনি যদি একটা library-র ১০,০০০ বই অন্য library-তে নিয়ে যান, পরদিন আবার sync করতে হলে rsync শুধু নতুন বা পরিবর্তিত বইগুলো নেবে, ১০,০০০ বই আবার নেবে না। scp বা cp হলে প্রতিবার সব কিছু copy করতো।


### Basic Syntax

```bash
rsync [options] source destination
```


### Basic Local Copy

```bash
rsync -av /home/devops/project/ /backup/project/
```

- `-a` → archive mode (permissions, timestamps, symlinks সব preserve করে)
- `-v` → verbose (কী copy হচ্ছে দেখাও)

**Output:**
```
sending incremental file list
./
index.html
app.py
config/
config/settings.yml

sent 15,234 bytes  received 134 bytes  30,736.00 bytes/sec
total size is 15,000  speedup is 0.97
```


### Remote Server-এ Sync করা (rsync over SSH)

```bash
rsync -avz /home/devops/app/ ubuntu@192.168.1.100:/home/ubuntu/app/
```

`-z` → compression - transfer করার সময় data compress করো (slow network-এ helpful)


### Remote থেকে Local-এ Sync করা

```bash
rsync -avz ubuntu@192.168.1.100:/var/log/app/ /home/devops/logs/
```


### Dry Run - আসলে কিছু না করে দেখা কী হবে

এটা rsync-এর সবচেয়ে দরকারী feature! Production-এ run করার আগে দেখুন কী কী হবে:

```bash
rsync -avz --dry-run /home/devops/app/ ubuntu@192.168.1.100:/home/ubuntu/app/
```

- `--dry-run` বা `-n` → কিছু আসলে copy করবে না, শুধু দেখাবে কী কী করতো


### Delete করা - Source-এ নেই এমন files Destination থেকে মুছে ফেলা

```bash
rsync -avz --delete /home/devops/app/ ubuntu@192.168.1.100:/home/ubuntu/app/
```

- `--delete` → source-এ যে files নেই, destination থেকে সেগুলো delete করো
- এটা দিয়ে perfect mirror তৈরি করা যায়

⚠️ **সাবধান:** `--delete` ব্যবহারের আগে `--dry-run` করো!


### নির্দিষ্ট Files বাদ দিয়ে Sync করা

```bash
rsync -avz --exclude='*.log' --exclude='.git/' /home/devops/app/ ubuntu@192.168.1.100:/home/ubuntu/app/
```

- `--exclude` → এই pattern-এর files sync করবে না


### SSH Key দিয়ে rsync

```bash
rsync -avz -e "ssh -i ~/.ssh/my_key.pem" /home/devops/app/ ubuntu@192.168.1.100:/home/ubuntu/app/
```

- `-e "ssh -i keyfile"` → কোন SSH command ব্যবহার করবে সেটা specify করো


### Progress দেখা

```bash
rsync -avz --progress /home/devops/bigfile.tar.gz ubuntu@192.168.1.100:/home/ubuntu/
```

**Output:**
```
bigfile.tar.gz
    524,288,000 100%   45.23MB/s    0:00:11  (xfr#1, to-chk=0/1)
```


### rsync Common Options

| Option | কাজ |
|--------|-----|
| `-a` | Archive mode (সব attributes preserve করো) |
| `-v` | Verbose |
| `-z` | Compression |
| `-n` / `--dry-run` | Test run - আসলে কিছু করবে না |
| `--delete` | Source-এ নেই এমন files destination থেকে মুছে ফেলো |
| `--exclude='pattern'` | নির্দিষ্ট files বাদ দাও |
| `--progress` | প্রতিটা file-এর progress দেখাও |
| `-e "ssh -i key"` | Custom SSH command |
| `--bandwidth-limit=1000` | Bandwidth limit (KB/s) |
| `-P` | `--partial --progress` এর shorthand |


## ⚠️ Important: Trailing Slash-এর পার্থক্য rsync-এ

এটা অনেকেই confuse হয়! মনে দিয়ে দেখো:

```bash
# Source-এর শেষে / আছে → folder-এর CONTENTS copy হবে
rsync -av /home/devops/app/ /backup/

# Source-এর শেষে / নেই → folder নিজেই copy হবে
rsync -av /home/devops/app /backup/
```

**উদাহরণ:**
- `/app/` দিলে → `/backup/index.html`, `/backup/config/` এভাবে যাবে
- `/app` দিলে → `/backup/app/index.html`, `/backup/app/config/` এভাবে যাবে


## সব Tool-এর তুলনা

| Feature | curl | wget | scp | rsync |
|---------|------|------|-----|-------|
| API calls | ✅ সেরা | ❌ | ❌ | ❌ |
| File download | ✅ | ✅ সেরা | ❌ | ✅ |
| Resume download | ❌ | ✅ (`-c`) | ❌ | ✅ |
| Secure transfer | ❌ (HTTPS) | ❌ (HTTPS) | ✅ (SSH) | ✅ (SSH) |
| Smart sync | ❌ | ❌ | ❌ | ✅ সেরা |
| Directory copy | ❌ | ✅ (`-r`) | ✅ (`-r`) | ✅ |
| Bandwidth efficient | ❌ | ❌ | ❌ | ✅ সেরা |


## Real-World DevOps Use Cases

### Scenario 1: Deployment Script-এ Binary Download করা
```bash
# Terraform download করা
curl -LO https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

### Scenario 2: 

Server-এ Application Deploy করা

```bash
# Build artifacts production server-এ পাঠানো
scp -i ~/.ssh/prod_key.pem ./dist/app.tar.gz ubuntu@prod-server:/opt/app/
```

### Scenario 3: 

Daily Backup Script

```bash
# rsync দিয়ে নিয়মিত backup
rsync -avz --delete /var/www/html/ backup@192.168.1.200:/backups/website/
```

### Scenario 4: 

Health Check Script-এ curl

```bash
# Application up আছে কিনা check করা
STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://myapp.com/health)
if [ "$STATUS" -ne 200 ]; then
    echo "App is DOWN! Status: $STATUS"
fi
```


## 📝 Quick Summary

- **curl** → URLs-এ request পাঠানো, API test করা, files download করার জন্য ব্যাবহার হয়। DevOps scripting-এ সবচেয়ে বেশি ব্যবহৃত হয়।
- **wget** → Internet থেকে files download করা, resume করা, এবং recursive download করার জন্য।
- **scp** → SSH দিয়ে securely files copy করার জন্য। Simple ও reliable, কিন্তু smart না।
- **rsync** → Smart sync tool - শুধু changed files transfer করে। Backup ও deployment-এ সেরা।


## 🏋️ Practice Tasks

**Task 1:** curl দিয়ে `https://jsonplaceholder.typicode.com/users` এ GET request করুন এবং response দেখুন। তারপর `-I` flag দিয়ে শুধু headers দেখুন।

**Task 2:** wget দিয়ে `https://www.gnu.org/licenses/gpl-3.0.txt` file টি download করুন। তারপর `-c` flag দিয়ে আবার চেষ্টা করুন (resume হবে কারণ file already আছে - message দেখুন)।

**Task 3:** আপনার local machine-এ দুটো directory বানান `source_dir` এবং `backup_dir`। `source_dir`-এ কিছু files তৈরি করুন। তারপর `rsync -av source_dir/ backup_dir/` দিয়ে sync করুন। একটা file delete করে আবার `--delete` flag সহ rsync করুন এবং পার্থক্য দেখুন।

---

## ⏭️ What's Next?

**Chapter 4 - Lesson 6: Open Ports & Sockets (ss, netstat, lsof, nmap)**

পরের lesson-এ আমরা শিখবো আপনার system-এ কোন ports open আছে, কোন process কোন port ব্যবহার করছে, এবং network connections কীভাবে দেখতে হয় - DevOps troubleshooting-এর জন্য অপরিহার্য skill! *Happy Learning* 🚀

<table width="100%">
  <tr>
    <td align="left">
      <a href="../04-DNS-Tools">← DNS Tools</a>
    </td>
    <td align="right">
      <a href="../06-Open-Ports-Sockets">Open Ports Sockets →</a>
    </td>
  </tr>
</table>