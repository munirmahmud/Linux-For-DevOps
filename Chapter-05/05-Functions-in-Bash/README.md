# Chapter 5 - Lesson 5: Functions in Bash

**Chapter 5 | Lesson 5 of 11**


## 🎯 এই Lesson-এ আমরা যা শিখবো

- Function কী এবং কেন দরকার?
- Function declare করার syntax
- Function call করা
- Arguments পাস করা
- Return values
- Local vs Global variables
- Nested functions
- Real-world DevOps examples


## Function কী? - সহজ ভাষায়

কল্পনা করুন আপনি একটা বড় রান্নাঘরে কাজ করছেন। প্রতিদিন আপনাকে একই কাজ বারবার করতে হয় যেমন পেঁয়াজ কাটা। প্রতিবার নতুন করে শিখতে হয় না, একবার শিখলেই হয়।

**Function হলো ঠিক এটাই** - একটা code block যেটা একবার লিখলে বারবার use করা যায়।

```
Without Function:          With Function:
─────────────────         ──────────────────
echo "Checking disk..."   check_disk()  ←── একবার লিখুন
df -h                     {
echo "Done"                  echo "Checking disk..."
                             df -h
echo "Checking disk..."      echo "Done"
df -h                     }
echo "Done"
                          check_disk   ←── যতবার খুশি call করো
echo "Checking disk..."   check_disk
df -h                     check_disk
echo "Done"
```


## Function Declare করার Syntax

Bash-এ function লেখার **দুটো valid উপায়** আছে:

### উপায় ১ - `function` keyword দিয়ে:
```bash
function function_name {
    # commands here
}
```

### উপায় ২ - Parentheses দিয়ে (বেশি popular):
```bash
function_name() {
    # commands here
}
```

> দুটোই কাজ করে। DevOps scripts-এ সাধারণত **উপায় ২** বেশি দেখা যায়।


## সবচেয়ে Simple Function - Hello World

```bash
#!/bin/bash

# Function declare করা
say_hello() {
    echo "Hello! I am a function!"
    echo "You can call me as many times as you want."
}

# Function call করা (শুধু নাম লিখলেই হয়)
say_hello
say_hello
say_hello
```

**Output:**
```
Hello! I am a function!
You can call me as many times as you want.
Hello! I am a function!
You can call me as many times as you want.
Hello! I am a function!
You can call me as many times as you want.
```

> ⚠️ **গুরুত্বপূর্ণ নিয়ম:** Function **declare করতে হবে আগে**, তারপর call করতে হবে। উল্টোটা করলে error আসবে।


## Function-এ Arguments পাস করা

Function-এ data পাঠানোর জন্য arguments use করা হয়। Bash-এ function arguments গুলো `$1`, `$2`, `$3`... দিয়ে access করা হয় ঠিক script arguments-এর মতোই!

```bash
#!/bin/bash

# $1 = প্রথম argument, $2 = দ্বিতীয় argument
greet_user() {
    echo "Hello, $1!"
    echo "Your age is $2 years old।"
}

# Arguments সহ call করা
greet_user "Rahim" 25
greet_user "Karim" 30
```

**Output:**
```
Hello, Rahim!
Your age is 25 years old।
Hello, Karim!
Your age is 30 years old।
```

### Special Variables inside Function:

| Variable | মানে |
|----------|------|
| `$1`, `$2`... | Function-এর ১ম, ২য়... argument |
| `$@` | সব arguments একসাথে |
| `$#` | কতটা argument আছে |
| `$0` | Script-এর নাম (function-এর না) |

```bash
#!/bin/bash

show_all_args() {
    echo "Total arguments: $#"
    echo "All arguments: $@"
    echo "The first one: $1"
    echo "The second one: $2"
}

show_all_args "apple" "banana" "cherry"
```

**Output:**
```
Total arguments: 3
All arguments: apple banana cherry
The first one: apple
The second one: banana
```


## Return Values - Function থেকে Result ফেরত নেওয়া

Bash-এ `return` statement শুধু **0-255 এর মধ্যে integer** return করতে পারে।
- `return 0` মানে **success**
- `return 1` (বা অন্য non-zero) মানে **failure/error**

এটাকে বলে **exit status**।

```bash
#!/bin/bash

is_even() {
    if (( $1 % 2 == 0 )); then
        return 0   # success = সংখ্যাটি জোড়
    else
        return 1   # failure = সংখ্যাটি বিজোড়
    fi
}

is_even 4
if [ $? -eq 0 ]; then
    echo "4 is an even number"
fi

is_even 7
if [ $? -ne 0 ]; then
    echo "7 is an odd number"
fi
```

> `$?` দিয়ে আগের command বা function-এর exit status জানা যায়।

**Output:**
```
4 is an even number
7 is an odd number
```


## String/Data Return করতে চাইলে - `echo` ব্যবহার করুন

Real data (string, number ইত্যাদি) return করতে হলে `echo` use করে **command substitution** দিয়ে capture করতে হয়:

```bash
#!/bin/bash

get_current_date() {
    echo $(date +"%Y-%m-%d")
}

get_disk_usage() {
    echo $(df -h / | awk 'NR==2 {print $5}')
}

# $() দিয়ে function-এর output capture করা
today=$(get_current_date)
disk=$(get_disk_usage)

echo "Today's date: $today"
echo "Root disk usage: $disk"
```

**Output:**
```
Today's date: 2026-03-23
Root disk usage: 45%
```


## Local vs Global Variables

এটা খুবই গুরুত্বপূর্ণ concept! ভুল করলে script-এ unexpected behavior হয়।

```bash
#!/bin/bash

name="Global Rahim"    # Global variable

test_scope() {
    local name="Local Karim"   # local = শুধু এই function-এর ভেতরে
    age=25                      # local ছাড়া = Global হয়ে যায়!
    echo "Inside function name: $name"
}

echo "Before function call name: $name"
test_scope
echo "After function call name: $name"   # Global-টা অপরিবর্তিত!
echo "age is now globally accessible: $age"   # 25 দেখাবে!
```

**Output:**
```
Before function call name: Global Rahim
Inside function name: Local Karim
After function call name: Global Rahim
age is now globally accessible: 25
```

> **Best Practice:** Function-এর ভেতরের সব variable সবসময় `local` দিয়ে declare করুন। এতে bug কমবে।


## Nested Functions - Function-এর ভেতরে Function Call

```bash
#!/bin/bash

print_header() {
    echo "================================"
    echo "   $1"
    echo "================================"
}

print_footer() {
    echo "================================"
    echo "         Completed"
    echo "================================"
}

run_health_check() {
    print_header "System Health Check"   # nested call
    echo "CPU Load: $(uptime | awk -F'load average:' '{print $2}')"
    echo "Memory: $(free -h | awk '/^Mem:/{print $3"/"$2}')"
    echo "Disk: $(df -h / | awk 'NR==2{print $5}')"
    print_footer                          # nested call
}

run_health_check
```

**Output:**
```
================================
   System Health Check
================================
CPU Load:  0.15, 0.10, 0.08
Memory: 1.2G/3.8G
Disk: 45%
================================
         Completed
================================
```


## Real-World DevOps Example - Complete Script

এবার একটা পূর্ণাঙ্গ DevOps-style script দেখুন যেটায় functions-এর সব concept আছে:

```bash
#!/bin/bash

# ============================================
# DevOps Server Health Check Script
# ============================================

# --- Utility Functions ---

print_banner() {
    local title="$1"
    echo ""
    echo "======================================"
    echo "  🖥️  $title"
    echo "======================================"
}

print_ok() {
    echo "  ✅ $1"
}

print_warn() {
    echo "  ⚠️  $1"
}

print_fail() {
    echo "  ❌ $1"
}

# --- Check Functions ---

check_cpu_load() {
    print_banner "CPU Load Check"
    local load=$(uptime | awk -F'load average:' '{print $2}' | cut -d',' -f1 | tr -d ' ')
    local threshold=2.0

    echo "  Current Load: $load"

    # bc দিয়ে decimal comparison
    if (( $(echo "$load < $threshold" | bc -l) )); then
        print_ok "CPU load is normal"
        return 0
    else
        print_warn "CPU load is high! ($load)"
        return 1
    fi
}

check_disk_space() {
    print_banner "Disk Space Check"
    local usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')

    echo "  Root Disk Usage: ${usage}%"

    if [ "$usage" -lt 80 ]; then
        print_ok "Disk space is enough"
        return 0
    elif [ "$usage" -lt 90 ]; then
        print_warn "Disk space 80% exceeded!"
        return 1
    else
        print_fail "Disk space is critical! ${usage}% ব্যবহৃত"
        return 2
    fi
}

check_memory() {
    print_banner "Memory Check"
    local total=$(free -m | awk '/^Mem:/{print $2}')
    local used=$(free -m | awk '/^Mem:/{print $3}')
    local percent=$(( used * 100 / total ))

    echo "  Total: ${total}MB | Used: ${used}MB | Usage: ${percent}%"

    if [ "$percent" -lt 75 ]; then
        print_ok "Memory usage is normal"
        return 0
    else
        print_warn "Memory usage is high: ${percent}%"
        return 1
    fi
}

check_service() {
    local service_name="$1"
    print_banner "Service Check: $service_name"

    if systemctl is-active --quiet "$service_name" 2>/dev/null; then
        print_ok "$service_name is running (active)"
        return 0
    else
        print_fail "$service_name is closed!"
        return 1
    fi
}

# --- Main Function ---

main() {
    local errors=0

    echo ""
    echo "Server Health Checking is started..."
    echo "   Time: $(date)"
    echo "   Server: $(hostname)"

    # সব checks রান করুন, error count করুন
    check_cpu_load    || (( errors++ ))
    check_disk_space  || (( errors++ ))
    check_memory      || (( errors++ ))
    check_service "ssh" || (( errors++ ))

    # Summary
    print_banner "Summary"
    if [ "$errors" -eq 0 ]; then
        print_ok "Everything is fine! No issues detected"
    else
        print_warn "A total of $errors issues/problems were found।"
    fi
    echo ""
}

# Script শুরু
main
```

**Sample Output:**
```
   Server Health Check is started...
   Time: Mon Mar 23 10:30:00 UTC 2026
   Server: devops-server-01

======================================
   CPU Load Check
======================================
  Current Load: 0.25
  CPU load is normal

======================================
   Disk Space Check
======================================
  Root Disk Usage: 45%
  Disk space is enough

======================================
   Memory Check
======================================
  Total: 3800MB | Used: 1200MB | Usage: 31%
  Memory usage is normal

======================================
   Service Check: ssh
======================================
  SSH is active

======================================
   Summary
======================================
  Everything is fine. No problem at all.
```


## Function Design - Best Practices

| নিয়ম | কারণ |
|------|------|
| প্রতিটি function একটাই কাজ করবে | Code clean থাকে |
| সব variables `local` করুন | Unexpected bugs এড়ানো যায় |
| Function নাম descriptive রাখুন | `chk` না, `check_disk_space` লিখুন |
| `return 0/1` দিয়ে status দিন | Caller সিদ্ধান্ত নিতে পারে |
| Script-এ একটা `main()` function রাখুন | Professional structure |


## 📝 Quick Summary

- **Function** = reusable code block, একবার লিখুন, বারবার use করুন
- Declare করতে `function_name() { }` syntax use করুন
- Arguments: `$1`, `$2`... দিয়ে access করা হয়
- `return 0` = success, `return 1` = failure
- String return করতে `echo` + command substitution `$()` use করুন
- Function-এর variable সবসময় `local` করা উচিত
- `$?` দিয়ে function-এর exit status check করা যায়
- একটা `main()` function রাখা professional practice


## 🏋️ Practice Tasks

**Task 1**
একটা function লিখুন `is_root()` যেটা check করবে script টা root হিসেবে run হচ্ছে কিনা। হলে "Root হিসেবে চলছে" print করবে, না হলে "Root access নেই" print করবে।
> Hint: `$EUID` variable check করুন - root হলে এটা `0` হয়।

**Task 2**
একটা function `backup_file()` লিখুন যেটা একটা filename argument নিবে। File exist করলে সেটার backup তৈরি করবে (`filename.bak` নামে), না থাকলে error message দিবে।

**Task 3**
উপরের Health Check script-টা নিজে টাইপ করুন এবং run করুন। তারপর এতে একটা নতুন function যোগ করুন `check_uptime()` যেটা server-এর uptime দেখাবে এবং যদি uptime 30 দিনের বেশি হয়, তাহলে "Reboot recommended" print করবে।

---

## ⏭️ What's Next?

**Chapter 5 - Lesson 6: Arrays & String Manipulation**
পরের lesson-এ শিখবো Bash-এ arrays কীভাবে কাজ করে, string কীভাবে manipulate করতে হয়। substring, replace, length, split এগুলো DevOps scripts-এ প্রচুর কাজে লাগে! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../04-Loops">← Loops</a>
    </td>
    <td align="right">
      <a href="../06-Arrays-String-Manipulation">Arrays String Manipulation →</a>
    </td>
  </tr>
</table>