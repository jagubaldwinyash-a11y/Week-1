Bash scripting basics

A bash script is a text file containing shell commands that are executed in sequence. Instead of typing commands one at a time, you write them into a script and run the script. This lets you automate repetitive tasks, process large datasets, and build CTF-solving tools.

Every security professional writes bash scripts — from automating reconnaissance to parsing log files to brute-forcing CTF challenges.

Why learn bash scripting?

Use case	Example

CTF automation	Try every word in a wordlist against a login form

Log analysis	Extract and count unique IPs from a 100,000-line access log

Recon automation	Scan a range of ports and save results

File processing	Rename, organize, or extract data from hundreds of files

System administration	Automated backups, health checks, user management


Step 1: Create the file

#!/bin/bash

echo "Hello!"

echo "User: $(whoami)"

echo "Date: $(date)"

echo "Directory: $(pwd)"

Step 2: Understand the shebang

The first line #!/bin/bash is called the shebang (or hashbang). It tells the operating system which interpreter to use for this file. Without it, the system might try to run your script with sh (which lacks some bash features) or fail entirely.

<img width="916" height="257" alt="image" src="https://github.com/user-attachments/assets/bd89a19f-a396-4bc1-935d-e477c42237c2" />

Step 3: Make it executable and run it

chmod +x hello.sh     # Add execute permission

./hello.sh            # Run the script

Why ./? The current directory isn't in your PATH by default (for security reasons). The ./ explicitly tells the shell to look in the current directory.

Alternative (without making executable):

bash hello.sh         # Explicitly use bash to interpret the file

Variables

Assignment and usage

name="student"    # No spaces around = (this is critical!)

count=3

directory="/var/log"

echo "$name has $count tasks"

echo "Checking $directory"

Common mistake: name = "student" (with spaces) doesn't work — bash interprets it as running a command called name with arguments = and "student".

Variable types

# Strings (default)

greeting="Hello, world"


# Integers (for arithmetic)

count=42

((count++))               # Increment

((result = count * 2))    # Arithmetic

# Arrays

fruits=("apple" "banana" "cherry")

echo "${fruits[0]}"       # First element: apple

echo "${fruits[@]}"       # All elements

echo "${#fruits[@]}"      # Length: 3

<img width="1191" height="345" alt="image" src="https://github.com/user-attachments/assets/dd0eef1c-1eeb-4cc5-89a2-9bce5ff16659" />

Rule of thumb: Always double-quote your variables ("$var") to prevent word splitting and globbing bugs. Use single quotes only when you explicitly don't want expansion.

<img width="952" height="387" alt="image" src="https://github.com/user-attachments/assets/e7fdaea7-d2ed-4626-bf6e-2c4a07b2dad8" />

Reading user input

read -p "Enter target IP: " target_ip

echo "Scanning $target_ip..."

# Read silently (for passwords)

read -sp "Enter password: " password

echo  # Newline after hidden input

Conditionals

if/elif/else

if [ "$count" -gt 10 ]; then
  echo "Many items"
elif [ "$count" -gt 0 ]; then
  echo "Some items"
else
  echo "No items"
fi


<img width="611" height="348" alt="image" src="https://github.com/user-attachments/assets/a525450d-64e8-491a-a6d5-f0b24962b9d8" />
<img width="696" height="240" alt="image" src="https://github.com/user-attachments/assets/74d715e2-a21d-45b9-842b-e83a12d322a6" />
<img width="672" height="416" alt="image" src="https://github.com/user-attachments/assets/443ae4fb-9a6f-4795-8c5f-215d05c2626a" />

for loops

# Iterate over a range

for i in {1..10}; do

  echo "Attempt $i"
  
done

# Iterate over files


for file in *.log;   do

  echo "Processing: $file ($(wc -l < "$file") lines)"
  
done

# C-style for loop

for ((i=0; i<5; i++));  do

  echo "Index: $i"
  
done

# Iterate over command output

for user in $(cut -d: -f1 /etc/passwd); do

  echo "User: $user"
  
done

WHILE loops

# Counter-based

counter=1
while [ $counter -le 5 ]; do

  echo "Count: $counter"
  
  ((counter++))
  
done

# Read file line by line (IMPORTANT for CTFs)

while IFS= read -r line; do

  echo "Line: $line"
  
done < input.txt

# Infinite loop with break condition
while true; do

  read -p "Enter 'quit' to exit: " input
  
  [ "$input" = "quit" ] && break
  
  echo "You said: $input"
  
done

until loops

# Run until condition becomes true

attempts=0

until [ $attempts -ge 5 ]; do

  echo "Attempt $((attempts + 1))"

  
  ((attempts++))
  
done

Loop control

# skip the current iteration


for i in {1..10}; do


  [ $i -eq 5 ] && continue    # Skip iteration 5
  
  echo $i
  
done

# exit the loop entirely

for i in {1..100}; do

  [ $i -eq 42 ] && break      # Stop at 42
  
  echo $i
  
done

Functions

# Define a function

greet() {

  local name="$1"    # local variables don't pollute global scope
  
  echo "Hello, $name!"
}

# Call it
greet "INFOSEC"         # Output: Hello, INFOSEC!
greet "$USER"        # Output: Hello, kali!

Functions with return values

# Return status (0 = success, 1-255 = failure)
is_root() {

  [ "$(id -u)" -eq 0 ]
  
}

if is_root; then

  echo "Running as root"
  
else

  echo "Not root"
  
fi

# Return data via stdout (capture with command substitution)
get_ip() {

  curl -s ifconfig.me
  
}

my_ip=$(get_ip)

echo "My IP: $my_ip"


Function with multiple parameters

scan_port() {

  local host="$1"
  
  local port="$2"
  
  if nc -z -w1 "$host" "$port" 2>/dev/null; then
  
    echo "[OPEN] $host:$port"
    return 0
  else
  
    echo "[CLOSED] $host:$port"
    return 1
  fi
}

scan_port "example.com" 80

scan_port "example.com" 443

Error handling

#!/bin/bash
set -e          # Exit immediately if any command fails

set -u          # Treat unset variables as errors

set -o pipefail # Pipe fails if any command in the pipe fails

# Check if required arguments are provided

if [ $# -lt 1 ]; then


  echo "Usage: $0 <target>" >&2  # Error messages go to stderr
  
  exit 1
  
fi

# Check if a required tool is installed

if ! command -v nmap &>/dev/null; then

  echo "Error: nmap is not installed" >&2
  
  exit 1
fi

# Check if a file exists before processing

if [ ! -f "$1" ]; then

  echo "Error: File '$1' not found" >&2
  
  exit 1
fi

<img width="597" height="393" alt="image" src="https://github.com/user-attachments/assets/fadcc2d7-b754-41a1-b154-56f0dfa51f9a" />

Practical CTF scripting patterns

Pattern 1: Brute-force with a wordlist

#!/bin/bash


# Try each password in a wordlist against a service

wordlist="$1"

target="$2"

if [ $# -ne 2 ]; then

  echo "Usage: $0 <wordlist> <target_url>"
  
  exit 1
  
fi

while IFS= read -r password; do

  response=$(curl -s -o /dev/null -w "%{http_code}" \
  
    -d "user=admin&pass=$password" "$target")
  
  if [ "$response" != "401" ]; then
  
    echo "[+] Found password: $password (HTTP $response)"
    exit 0
  fi
done < "$wordlist"


echo "[-] No valid password found"


Pattern 2: Timestamped output

#!/bin/bash
# Generate a report with timestamp

outfile="report_$(date +%Y%m%d_%H%M%S).txt"

{
  echo "=== System Report ==="
  echo "Generated: $(date)"
  echo ""
  echo "--- Uptime ---"
  uptime
  echo ""
  echo "--- Disk Usage ---"
  df -h
  echo ""
  echo "--- Listening Ports ---"
  ss -tulpn
} > "$outfile"

echo "Report saved to: $outfile"

#Pattern 3: Process multiple files#

#!/bin/bash
# Search for flags across all files in a directory

target_dir="${1:-.}"  # Default to current directory

echo "Searching for flags in: $target_dir"
echo "---"

find "$target_dir" -type f -print0 | while IFS= read -r -d '' file; do
  match=$(grep -l "csot26{" "$file" 2>/dev/null)
  if [ -n "$match" ]; then
    echo "[FLAG] Found in: $match"
    grep -o "csot26{[^}]*}" "$match"
  fi
done

Pattern 4: Port scanner (simple)

#!/bin/bash
# Scan common ports on a host

host="$1"
ports=(21 22 23 25 53 80 110 143 443 445 993 995 3306 3389 8080 8443)

if [ -z "$host" ]; then
  echo "Usage: $0 <host>"
  exit 1
fi

echo "Scanning $host..."
for port in "${ports[@]}"; do
  (echo >/dev/tcp/"$host"/"$port") 2>/dev/null && \
    echo "[OPEN] Port $port" &
done
wait
echo "Scan complete."

Pattern 5: Decode nested encodings

#!/bin/bash
# Iteratively decode base64 until no longer valid base64

input="$1"
iteration=0

while true; do
  decoded=$(echo "$input" | base64 -d 2>/dev/null)
  if [ $? -ne 0 ] || [ -z "$decoded" ]; then
    echo "Final result (after $iteration decodings):"
    echo "$input"
    break
  fi
  ((iteration++))
  echo "Decoded layer $iteration: $decoded"
  input="$decoded"
done

Style guide for this course

Always include a shebang (#!/bin/bash)

Quote all variables ("$var" not $var)

Use [[ ]] over [ ] for conditionals (bash-specific but safer)

Use local in functions to avoid polluting global scope

Check arguments at the start of the script

Use meaningful variable names (target_ip not t)

Add error handling (set -euo pipefail for strict mode)

Use $(command) not backticks for command substitution



