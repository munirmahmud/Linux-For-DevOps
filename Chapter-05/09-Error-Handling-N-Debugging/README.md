# Chapter 5 - Lesson 9: Error Handling & Debugging

**Chapter 5 | Lesson 9 of 11**

আপনি এতদূর এসেছেন এটা সত্যিই দারুণ! আপনাকে অভিনন্দন 🎉। এখন আমরা Shell Scripting-এর সবচেয়ে professional এবং important অংশে এসেছি। আজকে আমরা শিখবো কিভাবে Error Handling & Debugging করতে হয়।

একজন junior আর senior DevOps Engineer-এর মধ্যে সবচেয়ে বড় পার্থক্য হলো senior জানে কীভাবে script fail হয়। আর fail হলে কী করতে হবে, কেন হবে, এবং কীভাবে fix করতে হবে তা সঠিকভাবে ম্যানেজ করতে পারে।

## প্রথমে বুঝি - Error Handling কেন দরকার?

কল্পনা করুন আপনি একটা script লিখেছেন যেটা:
1. একটা directory তৈরি করে
2. সেখানে কিছু files copy করে
3. তারপর সেই files compress করে
4. তারপর একটা server-এ upload করে

এখন যদি step 2 fail করে এবং files copy না হয় তাহলে কি script বন্ধ হবে? না! সে চুপচাপ বাকি steps চালাতে থাকবে। এটা খুবই বিপজ্জনক।

Error handling মানে হলো "কোনো সমস্যা হলে script বুদ্ধিমানের মতো আচরণ করবে।"

## 🎯 এই Lesson-এ আমরা যা শিখব:

| Topic | কী কাজ করে |
|---|---|
| `set -e` | যেকোনো error-এ script বন্ধ করে |
| `set -u` | undefined variable use করলে বন্ধ করে |
| `set -o pipefail` | pipe-এর মধ্যে error ধরে |
| `set -x` | প্রতিটা command debug করে দেখায় |
| `bash -x` | বাইরে থেকে script debug করে |
| `trap` | exit বা error-এ cleanup চালায় |

## PART 1: Default Bash Behavior - সমস্যাটা কী?

প্রথমে দেখি bash by default কতটা **"নীরবে"** error ignore করে:

```bash
#!/bin/bash

echo "Step 1: Starting..."
ls /fake/directory/that/does/not/exist    # এটা fail করবে
echo "Step 2: It's still running."        # তবুও এটা চলবে!
echo "Step 3: Everything is fine."        # এটাও চলবে!
```

**Output:**
```
Step 1: Starting...
ls: cannot access '/fake/directory/that/does/not/exist': No such file or directory
Step 2: It's still running.
Step 3: Everything is fine.
```

এখন দেখুন error হলো, কিন্তু script বন্ধ হলো না! এটাই সমস্যা।


## PART 2: `set -e` - Error হলেই বন্ধ করো

### `set -e` কী করে?

- Exit immediately if any command fails
- যেকোনো command যদি non-zero exit code return করে, script সাথে সাথে বন্ধ হয়ে যাবে।

### Exit Code কী?

Linux-এ প্রতিটা command শেষ হওয়ার পর একটা নম্বর return করে:
- `0` = সফল (success)
- non-zero (1, 2, 127...) = কোনো সমস্যা হয়েছে

```bash
ls /tmp          # সফল হলে → exit code 0
echo $?          # 0 দেখাবে

ls /fake/path    # fail হলে → exit code 2
echo $?          # 2 দেখাবে
```

> `$?` হলো last command-এর exit code।

### `set -e` দিয়ে উদাহরণ:

```bash
#!/bin/bash
set -e    # এই line-এর পরে যেকোনো error → script বন্ধ

echo "Step 1: Starting..."
ls /fake/directory/that/does/not/exist    # এটা fail করবে
echo "Step 2: This won't run anymore."    # এই line কখনো execute হবে না
echo "Step 3: Not this one either"
```

**Output:**
```
Step 1: Starting...
ls: cannot access '/fake/directory/that/does/not/exist': No such file or directory
```

এবার script সাথে সাথে বন্ধ হয়ে গেছে।

## PART 3: `set -u` - Undefined Variable ধরো

### `set -u` কী করে?

> Treat unset variables as an error।
> যদি কোনো variable আগে declare/define না করেন শুধু use করার চেষ্টা করেন, তাহলে set -u সেটাকে error হিসেবে ধরবে এবং script বন্ধ করে দেবে।

### সমস্যাটা কী `set -u` ছাড়া?

```bash
#!/bin/bash

username="devops"
echo "User: $usernaem"   # টাইপো! "username" লিখতে গিয়ে "usernaem" লিখেছি
```

**Output (set -u ছাড়া):**
```
User:        ← শুধু blank! Error নেই, warning নেই
```

এটা খুবই বিপজ্জনক। Script চলতে থাকবে কিন্তু ভুল data দিয়ে।

### `set -u` দিয়ে:

```bash
#!/bin/bash
set -u

username="devops"
echo "User: $usernaem"   # টাইপো আছে
```

**Output:**
```
bash: usernaem: unbound variable
```

এবার সাথে সাথে ধরা পড়ল!


## PART 4: `set -o pipefail` - Pipe-এর Error ধরো

### সমস্যাটা কী?

`set -e` একটা সমস্যায় পড়ে, সে pipe (`|`) এর মধ্যে error ধরতে পারে না।

```bash
#!/bin/bash
set -e

# এই command fail করবে কিন্তু grep সফল হবে
ls /fake/path | grep "something"
echo "Is this going to work?"
```

Bash শুধু সর্বশেষ command (grep)-এর exit code দেখে। `grep` হয়তো `0` return করেছে, তাই `set -e` বুঝতেই পারেনি error হয়েছে!

### `set -o pipefail` দিয়ে সমাধান:

```bash
#!/bin/bash
set -e
set -o pipefail    # pipe-এর যেকোনো অংশে error → পুরো pipe fail

ls /fake/path | grep "something"
echo "This won't run anymore."
```

**Output:**
```
ls: cannot access '/fake/path': No such file or directory
```

এখন pipe-এর মধ্যের error-ও ধরা পড়ছে।


## PART 5: তিনটা একসাথে - `set -euo pipefail`

Professional DevOps scripts-এ সবসময় এই তিনটা একসাথে লেখা হয়:

```bash
#!/bin/bash
set -euo pipefail
```

অথবা আলাদা করে:
```bash
#!/bin/bash
set -e          # error হলে বন্ধ
set -u          # undefined variable বন্ধ
set -o pipefail # pipe error ধরো
```

> **মনে রাখার উপায়:** **e**xit on error, **u**nset variable error, **o** pipefail

### একটা Complete উদাহরণ:

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/tmp/my_backup"
SOURCE_DIR="/etc/nginx"      # এই directory আছে ধরে নিচ্ছি

echo "Backup Starting..."
mkdir -p "$BACKUP_DIR"
cp -r "$SOURCE_DIR" "$BACKUP_DIR/"
echo "Backup successful: $BACKUP_DIR"
```

এই script-এ যদি কোনো step fail করে যেমন mkdir, cp যাই হোক, script বন্ধ হবে এবং পরের step চলবে না।

## PART 6: `bash -x` এবং `set -x` - Debug Mode

এখন আসি **debugging** - মানে বোঝা যে script কোথায় সমস্যা করছে।

### `set -x` কী করে?

> **Print each command before executing it**
> Script চলার সময় প্রতিটা command execute হওয়ার আগে সেটা screen-এ দেখায় variable-এর মান সহ।

```bash
#!/bin/bash
set -x    # Debug mode চালু

name="DevOps Engineer"
echo "I am a $name"
ls /tmp
```

**Output:**
```
+ name='DevOps Engineer'
+ echo 'I am a DevOps Engineer'
I am a DevOps Engineer
+ ls /tmp
(tmp এর contents...)
```

> `+` চিহ্ন মানে bash এই command execute করছে।

### বাইরে থেকে Debug - `bash -x`

Script file পরিবর্তন না করেও debug করতে পারা:

```bash
bash -x myscript.sh
```

এটা পুরো script-টা debug mode-এ চালাবে।

### শুধু নির্দিষ্ট অংশ debug করা:

```bash
#!/bin/bash

echo "It's running in normal mode"

set -x    # এখান থেকে debug শুরু
name="Munir "
echo "Name: $name"
set +x    # এখানে debug বন্ধ

echo "It's back in normal mode."
```

**Output:**
```
It's running in normal mode
+ name=Munir
+ echo 'Name: Munir'
Name: Munir
It's back in normal mode.
```

> `set +x` দিয়ে debug mode বন্ধ করা যায়।


## PART 7: `trap` - Cleanup & Exit Handling

### `trap` কী?

> ধরুন আপনি রান্না করছেন এবং বললেন "যদি কিছু পুড়ে যায়, তাহলে আমি রান্না বন্ধ করব এবং জানালা খুলব।" `trap` ঠিক এই কাজটাই করে মানে "কোনো event ঘটলে এই কাজটা করো।"

### Syntax:

```bash
trap 'commands_to_run' SIGNAL
```

### Common Signals:

| Signal | কখন fire হয় |
|---|---|
| `EXIT` | script যেকোনোভাবে শেষ হলে (success বা error) |
| `ERR` | যেকোনো command fail করলে |
| `INT` | Ctrl+C চাপলে |
| `TERM` | kill command দিলে |

### উদাহরণ 1: Cleanup on EXIT

```bash
#!/bin/bash
set -euo pipefail

TEMP_FILE="/tmp/my_temp_$$"   # $$ = current script-এর PID

# trap set করছি - script শেষ হলে temp file মুছে ফেলো
trap 'rm -f "$TEMP_FILE"; echo "Cleanup is done!"' EXIT

echo "Work started..."
echo "Some data" > "$TEMP_FILE"
echo "Temp file created: $TEMP_FILE"

# কোনো কাজ করো...
sleep 1

echo "Work done!"
# script শেষ হলে trap চলবে এবং TEMP_FILE মুছে যাবে
```

**Output:**
```
Work started...
Temp file created: /tmp/my_temp_12345
Work done!
Cleanup is done!
```

এমনকি যদি script মাঝপথে fail করে বা Ctrl+C চাপেন তাহলে temp file মুছে যাবে।

### উদাহরণ 2: Error-এ মেসেজ পাঠান

```bash
#!/bin/bash
set -euo pipefail

# Error হলে কোন line-এ হলো সেটা দেখাও
trap 'echo "❌ Error occurred! Line number: $LINENO"' ERR

echo "Step 1: Done"
ls /fake/nonexistent/path    # এটা fail করবে
echo "Step 2: This won't work"
```

**Output:**
```
Step 1: Done
ls: cannot access '/fake/nonexistent/path': No such file or directory
❌ Error occurred! Line number: 6
```

> `$LINENO` = script-এর কত নম্বর line-এ error হলো।

### উদাহরণ 3: Ctrl+C চাপুন

```bash
#!/bin/bash

trap 'echo ""; echo "You pressed Ctrl+C! The script is stopping..."; exit 1' INT

echo "Script is running... Press Ctrl+C to stop."
while true; do
    echo "Work in progress... $(date)"
    sleep 2
done
```

**Output (Ctrl+C চাপলে):**
```
Script is running... Press Ctrl+C to stop.
Work in progress... Sun Mar 15 10:00:00 2026
Work in progress... Sun Mar 15 10:00:02 2026
^C
You pressed Ctrl+C! The script is stopping...
```

## PART 8: সব একসাথে - Professional Error Handling Template

এটা হলো একটা **production-ready script template** যেটা আপনি সবসময় ব্যবহার করতে পারেন:

```bash
#!/bin/bash
# ============================================
# Script Name: deployment.sh
# Description: Professional deployment script
# Author: Munir Mahmud
# ============================================

set -euo pipefail

# ---- Variables ----
SCRIPT_NAME=$(basename "$0")
LOG_FILE="/tmp/${SCRIPT_NAME}.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# ---- Colors (optional, সুন্দর output-এর জন্য) ----
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'   # No Color

# ---- Functions ----
log() {
    echo -e "[$TIMESTAMP] $1" | tee -a "$LOG_FILE"
}

success() {
    echo -e "${GREEN}✅ $1${NC}" | tee -a "$LOG_FILE"
}

error() {
    echo -e "${RED}❌ $1${NC}" | tee -a "$LOG_FILE"
}

warn() {
    echo -e "${YELLOW}⚠️  $1${NC}" | tee -a "$LOG_FILE"
}

# ---- Cleanup Function ----
cleanup() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        error "Script failed! Exit code: $exit_code, Line: $LINENO"
        error "Log file: $LOG_FILE"
    else
        success "Script completed successfully!"
    fi
}

# ---- Trap Setup ----
trap cleanup EXIT
trap 'error "Interrupted by user!"; exit 130' INT TERM

# ---- Main Script ----
log "Deployment Starting..."

# Step 1
log "Step 1: Dependencies are checking..."
command -v git >/dev/null 2>&1 || { error "git is not found!"; exit 1; }
success "Git is found: $(git --version)"

# Step 2
log "Step 2: Backup is creating..."
BACKUP_DIR="/tmp/backup_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"
success "Backup directory is ready: $BACKUP_DIR"

# Step 3
log "Step 3: Work done!"
```

---

## 🔧 PART 9: কিছু Handy Error Handling Techniques

### Technique 1: Command fail হলে নিজেই error message দাও

```bash
#!/bin/bash

# || মানে "আগেরটা fail করলে এটা চালাও"
mkdir /fake/path || { echo "Directory could not be created!"; exit 1; }

# একটু ছোট করে:
cp important.txt /backup/ || exit 1
```

### Technique 2: Command exist করে কিনা check করা

```bash
#!/bin/bash

# কোনো tool আছে কিনা check
if ! command -v docker &>/dev/null; then
    echo "❌ Docker is not installed!"
    exit 1
fi

echo "✅ Docker is found"
```

### Technique 3: File exist করে কিনা check করুন

```bash
#!/bin/bash
set -euo pipefail

CONFIG_FILE="/etc/myapp/config.yaml"

if [ ! -f "$CONFIG_FILE" ]; then
    echo "❌ Config file is not found: $CONFIG_FILE"
    exit 1
fi

echo "✅ Config file is found"
```

### Technique 4: Script root হিসেবে চলছে কিনা check করুন

```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
    echo "❌ This script needs to be run as root!"
    echo "Try: sudo $0"
    exit 1
fi

echo "✅ Root permission is available, proceeding..."
```

## PART 10: `bash -x` দিয়ে Debugging Workflow

### DevOps-এ debugging করার পদ্ধতি:

```bash
# ধাপ 1: Script-এ syntax error আছে কিনা check করো
bash -n myscript.sh       # শুধু syntax check, execute করে না

# ধাপ 2: Debug mode-এ রান করো
bash -x myscript.sh

# ধাপ 3: Debug output log-এ save করো
bash -x myscript.sh 2> debug.log

# ধাপ 4: উভয় (stdout + stderr) log-এ দেখো
bash -x myscript.sh > output.log 2>&1
```

> `-n` flag = "dry run" - script execute না করে শুধু syntax চেক করে।

## 📝 Quick Summary

- ✅ **`set -e`** - যেকোনো command fail করলে script বন্ধ করে
- ✅ **`set -u`** - undefined variable use করলে error দেয়
- ✅ **`set -o pipefail`** - pipe-এর মধ্যে error ধরে
- ✅ **`set -euo pipefail`** - তিনটা একসাথে, সবসময় ব্যবহার করুন
- ✅ **`set -x` / `bash -x`** - debug mode, প্রতিটা command দেখায়
- ✅ **`set +x`** - debug mode বন্ধ করে
- ✅ **`bash -n`** - syntax check, execute করে না
- ✅ **`trap`** - নির্দিষ্ট event-এ (EXIT, ERR, INT) কাজ করে
- ✅ **`$LINENO`** - কত নম্বর line-এ error সেটা দেখায়
- ✅ **`$?`** - last command-এর exit code

## 🏋️ Practice Tasks

**Task 1:**
একটা script লেখুন `safe_copy.sh` যেটা:
- `set -euo pipefail` ব্যবহার করবে
- দুটো argument নেবে: source file এবং destination
- Check করবে source file exist করে কিনা
- না থাকলে error message দিয়ে exit করবে
- থাকলে copy করবে এবং success message দেখাবে
- `trap` দিয়ে EXIT-এ "Script শেষ হয়েছে" print করবে

**Task 2:**
একটা script লেখুন যেটা:
- ইচ্ছাকৃতভাবে `set -x` ব্যবহার করবে
- কিছু variable declare করবে
- একটা loop চালাবে
- Debug output দেখে বুঝবে কোন line-এ কী হচ্ছে

**Task 3:**
```bash
#!/bin/bash
username=$USER_NAME
echo "Hello $username"
```

এই script-এ কী সমস্যা? `set -u` যোগ করলে কী হবে? Test করুন।

---

## ⏭️ What's Next

**Chapter 5 - Lesson 10: Scheduling Scripts**
`cron`, `anacron`, এবং `systemd timers` কীভাবে script automatically নির্দিষ্ট সময়ে চালাবে সেটা শিখব। DevOps-এ automated backup, log cleanup, health check সবই এই `cron` দিয়ে হয়! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../08-Working-with-Files-in-Scripts">← Working with Files in Scripts</a>
    </td>
    <td align="right">
      <a href="../10-Scheduling-Scripts">Scheduling Scripts →</a>
    </td>
  </tr>
</table>