# Chapter 5 - Lesson 4: Loops (for, while, until, break, continue)

**Chapter 5 | Lesson 4 of 11**


আগের lesson-এ আমরা শিখেছিলাম **Conditionals** if, elif, else দিয়ে decision নেওয়া। আজকে শিখবো **Loops** একটা কাজ বারবার automatically করার technique। DevOps-এ loops ছাড়া scripting প্রায় অসম্ভব! চলুন শুরু করি।


## Loop কী? কেন দরকার?

কল্পনা করুন আপনাকে বলা হলো "১০০টা server-এ একটা file create করো।"

আপনি কি manually ১০০ বার command লিখবেন? না! আপনি একটা loop লিখবেন যেটা automatically ১০০ বার কাজটা করবে।

**Loop = একটা কাজ নির্দিষ্ট বার বা একটা condition পূরণ না হওয়া পর্যন্ত বারবার করা।**

Bash-এ ৩ ধরনের loop আছে:
| Loop | কখন ব্যবহার করবে |
|------|-----------------|
| `for` | জানা আছে কতবার চলবে / list আছে |
| `while` | condition সত্য থাকলে চলতে থাকবে |
| `until` | condition মিথ্যা থাকলে চলতে থাকবে |


## for Loop

**For loop** একটা list-এর প্রতিটা item-এর জন্য একবার করে কাজ করে।

> আপনার সামনে ৫টা আম আছে। আপনি একে একে প্রতিটা আম তুলে দেখছেন, এটাই for loop।


### Syntax

```bash
for variable in list
do
    # commands
done
```

**প্রতিটা অংশ:**
- `variable` → প্রতিবার list-এর একটা item এই variable-এ stored হয়
- `in list` → যে list-এর উপর loop চলবে
- `do` → loop-এর body শুরু
- `done` → loop শেষ


### Example 1 - Simple List

```bash
#!/bin/bash

for fruit in apple banana mango orange
do
    echo "Fruit: $fruit"
done
```

**Output:**
```
Fruit: apple
Fruit: banana
Fruit: mango
Fruit: orange
```

**কী হলো?** `fruit` variable প্রথমে `apple`, তারপর `banana`, তারপর `mango`, তারপর `orange` হয়। প্রতিবার `echo` চলে।


### Example 2 - Number Range (C-style)

```bash
#!/bin/bash

for ((i=1; i<=5; i++))
do
    echo "Number: $i"
done
```

**Output:**
```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

**Syntax breakdown:**
- `i=1` → শুরু ১ থেকে
- `i<=5` → যতক্ষণ ৫-এর ছোট বা সমান
- `i++` → প্রতিবার ১ বাড়াও


### Example 3 - `seq` দিয়ে Range

```bash
#!/bin/bash

for i in $(seq 1 5)
do
    echo "Count: $i"
done
```

`seq 1 5` → 1 থেকে 5 পর্যন্ত সংখ্যা generate করে।


### Example 4 - Brace Expansion

```bash
#!/bin/bash

for i in {1..5}
do
    echo "Item: $i"
done
```

`{1..5}` → ১ থেকে ৫ পর্যন্ত automatically expand হয়। এটা সবচেয়ে clean way।


### Example 5 - Files-এর উপর Loop (DevOps Common!)

```bash
#!/bin/bash

for file in /var/log/*.log
do
    echo "Log file found: $file"
done
```

**Output (example):**
```
Log file found: /var/log/auth.log
Log file found: /var/log/syslog
Log file found: /var/log/kern.log
```

**DevOps use case:** সব log file process করা, backup করা, বা size check করা।


### Example 6 - Array-এর উপর Loop

```bash
#!/bin/bash

servers=("web01" "web02" "db01" "db02")

for server in "${servers[@]}"
do
    echo "Checking server: $server"
done
```

**Output:**
```
Checking server: web01
Checking server: web02
Checking server: db01
Checking server: db02
```

**DevOps use case:** একটা list of servers-এ একই command চালানো।


### Example 7 - Real DevOps Script: Multiple Files Create

```bash
#!/bin/bash

for i in {1..5}
do
    touch "/tmp/server_report_$i.txt"
    echo "Created: server_report_$i.txt"
done
```

**Output:**
```
Created: server_report_1.txt
Created: server_report_2.txt
Created: server_report_3.txt
Created: server_report_4.txt
Created: server_report_5.txt
```


## while Loop

**While loop** চলতে থাকে যতক্ষণ একটা **condition সত্য (true)** থাকে।

> আপনি পানি পান করছেন "যতক্ষণ পিপাসা আছে, পান করতে থাকুন।" পিপাসা না থাকলে থামুন। এটাই while loop।


### Syntax

```bash
while [ condition ]
do
    # commands
done
```


### Example 1 - Basic Counter

```bash
#!/bin/bash

count=1

while [ $count -le 5 ]
do
    echo "Count: $count"
    count=$((count + 1))
done
```

**Output:**
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
```

**কী হলো?**

- শুরুতে `count=1`
- Condition: `count` যতক্ষণ ৫-এর ছোট বা সমান
- প্রতিবার `count` এক বাড়ে
- `count=6` হলে condition false → loop শেষ


### Example 2 - User Input Read করা

```bash
#!/bin/bash

echo "Type 'quit' to exit"

while true
do
    read -p "Enter command: " cmd
    if [ "$cmd" == "quit" ]; then
        echo "Goodbye!"
        break
    fi
    echo "You typed: $cmd"
done
```

**`while true`** → এটা infinite loop, কখনো থামবে না যতক্ষণ `break` না দেওয়া হয়।


### Example 3 - File Line by Line Read করা (DevOps!)

```bash
#!/bin/bash

# servers.txt তে server names আছে
while IFS= read -r line
do
    echo "Server: $line"
done < servers.txt
```

**servers.txt:**
```
web01
web02
db01
```

**Output:**
```
Server: web01
Server: web02
Server: db01
```

> একটা file থেকে server list পড়ে প্রতিটায় action নেওয়া।

- `IFS=` → line-এর leading/trailing whitespace preserve করে
- `-r` → backslash escape prevent করে
- `< servers.txt` → file থেকে input নেওয়া


### Example 4 - Service Health Check Loop

```bash
#!/bin/bash

service="nginx"
max_attempts=5
attempt=1

while [ $attempt -le $max_attempts ]
do
    if systemctl is-active --quiet $service; then
        echo "$service is running!"
        break
    else
        echo "Attempt $attempt: $service not running. Retrying..."
        attempt=$((attempt + 1))
        sleep 2
    fi
done
```

> Service start হয়েছে কিনা বারবার check করা।


## until Loop


**Until loop** while-এর উল্টো। এটা চলতে থাকে যতক্ষণ condition **মিথ্যা (false)** থাকে। Condition সত্য হলে থামে।

> "যতক্ষণ রান্না শেষ না হয়, অপেক্ষা করো।" রান্না শেষ হলে (condition true) থামো।


### Syntax

```bash
until [ condition ]
do
    # commands
done
```


### Example 1 - Basic until

```bash
#!/bin/bash

count=1

until [ $count -gt 5 ]
do
    echo "Count: $count"
    count=$((count + 1))
done
```

**Output:**
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
```

**while vs until comparison:**
| | while | until |
|---|---|---|
| চলে যখন | condition **true** | condition **false** |
| থামে যখন | condition **false** | condition **true** |
| উপরেরটার equivalent | `while [ $count -le 5 ]` | `until [ $count -gt 5 ]` |


### Example 2 - Server Available না হওয়া পর্যন্ত Retry

```bash
#!/bin/bash

host="google.com"

until ping -c 1 $host &>/dev/null
do
    echo "⏳ Waiting for $host to be reachable..."
    sleep 3
done

echo "$host is now reachable!"
```

> Server deploy হওয়ার পর available হওয়া পর্যন্ত wait করা।


## break - Loop থেকে বের হয়ে যাওয়া

`break` একটা loop-কে মাঝপথে **সম্পূর্ণ থামিয়ে দেয়।**

> ধরুন আপনি বাজারে গেছেন, ১০টা জিনিস কিনবেন। কিন্তু ৫ নম্বর জিনিস পেয়েই বললেন "যথেষ্ট হয়েছে" বাকিগুলো আর না দেখে চলে আসলেন। এটাই `break`।


### Example

```bash
#!/bin/bash

for i in {1..10}
do
    if [ $i -eq 6 ]; then
        echo "Found 6! Stopping loop."
        break
    fi
    echo "Number: $i"
done

echo "Loop ended."
```

**Output:**
```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
Found 6! Stopping loop.
Loop ended.
```

**৬ পেয়ে সাথে সাথে loop থেমে গেছে।** ৭, ৮, ৯, ১০ আর run হয়নি।


## continue - Current Iteration Skip করা


`continue` current iteration-কে **skip** করে পরের iteration-এ চলে যায়। Loop সম্পূর্ণ বন্ধ হয় না।

> বাজারে ১০টা জিনিস দেখছেন। ৫ নম্বর জিনিসটা expired, সেটা skip করে সরাসরি ৬ নম্বরে চলে যান। বাকি কেনাকাটা চলতে থাকে।


### Example

```bash
#!/bin/bash

for i in {1..10}
do
    if [ $i -eq 5 ] || [ $i -eq 8 ]; then
        echo "Skipping $i"
        continue
    fi
    echo "Processing: $i"
done
```

**Output:**
```
Processing: 1
Processing: 2
Processing: 3
Processing: 4
Skipping 5
Processing: 6
Processing: 7
Skipping 8
Processing: 9
Processing: 10
```

**৫ আর ৮ skip হয়েছে, বাকি সব process হয়েছে।**


## break vs continue

```
Loop চলছে...
  ↓
  Item 5 আসলো
  ↓
break → সম্পূর্ণ loop বন্ধ, বাকি items আর দেখা হবে না
continue → শুধু item 5 skip, item 6 থেকে আবার শুরু
```


## Nested Loops (Loop-এর ভেতরে Loop)

### Example

```bash
#!/bin/bash

for i in 1 2 3
do
    for j in A B C
    do
        echo "$i-$j"
    done
done
```

**Output:**
```
1-A
1-B
1-C
2-A
2-B
2-C
3-A
3-B
3-C
```

**DevOps use case:** Multiple environments × multiple servers-এ কাজ করা।


## Real-World DevOps Script: Log Cleanup

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
MAX_SIZE=1048576  # 1MB in bytes

echo "=== Log Cleanup Script ==="

for logfile in $LOG_DIR/*.log
do
    # File exist করে কিনা check করো
    if [ ! -f "$logfile" ]; then
        continue
    fi

    filesize=$(stat -c%s "$logfile")

    if [ $filesize -gt $MAX_SIZE ]; then
        echo "🗑️  Large file found: $logfile ($(($filesize/1024)) KB) - Archiving..."
        gzip "$logfile"
    else
        echo "OK: $logfile ($(($filesize/1024)) KB)"
    fi
done

echo "=== Cleanup Done ==="
```

**কী হচ্ছে এখানে:**
- সব `.log` file-এর উপর loop চলছে
- File না থাকলে skip (`continue`)
- File size ১MB-এর বেশি হলে `gzip` দিয়ে compress করছে


## 📝 Quick Summary

- **`for` loop** → list বা range-এর প্রতিটা item-এর জন্য কাজ করে
- **`while` loop** → condition সত্য থাকলে চলে
- **`until` loop** → condition মিথ্যা থাকলে চলে (while-এর উল্টো)
- **`break`** → loop সম্পূর্ণ বন্ধ করে
- **`continue`** → current iteration skip করে পরেরটায় যায়
- **`{1..10}`** → 1 থেকে 10 পর্যন্ত brace expansion
- **`$(seq 1 10)`** → same কাজ, বেশি flexible
- **`while true`** → infinite loop, `break` দিয়ে বের হতে হয়
- **`while read line`** → file line by line পড়ার standard way


## 🏋️ Practice Tasks

**Task 1:**
একটা script লেখুন যেটা `{1..20}` range-এ loop চালাবে এবং শুধু **জোড় সংখ্যাগুলো** (2, 4, 6...) print করবে। (Hint: `continue` ব্যবহার করুন, odd হলে skip করুন। `$((i % 2))` দিয়ে remainder বের করুন।)

**Task 2:**
একটা script লেখুন যেটা `/tmp` directory-তে `test_1.txt` থেকে `test_5.txt` পর্যন্ত ৫টা file **for loop দিয়ে** create করবেন, এবং প্রতিটা file-এ লিখবে `"This is file number X"`।

**Task 3:**
একটা while loop লেখুন যেটা user-এর কাছে number input নেবে। যদি number `0` হয় তাহলে `"Goodbye!"` print করে বের হয়ে যাবে। অন্যথায় বলবে `"You entered: X"`।

---

## ⏭️ What's Next

**Chapter 5 - Lesson 5: Functions in Bash**
→ Reusable code block তৈরি করা, parameters pass করা, এবং return values - script-কে professional-level-এ নিয়ে যাওয়া! *Happy Learning* 🚀


<table width="100%">
  <tr>
    <td align="left">
      <a href="../03-Conditionals">← Conditionals</a>
    </td>
    <td align="right">
      <a href="../05-Functions-in-Bash">Functions in Bash →</a>
    </td>
  </tr>
</table>