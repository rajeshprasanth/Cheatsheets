# Bash Beginners Cheat Sheet

## Basics

```bash
# Run script
bash script.sh
./script.sh

# Make executable
chmod +x script.sh
```

## Variables

```bash
# Basic assignment
name="Rajesh"

# Use variable
echo "Hello $name"

# Read user input
read user_input
echo "You typed: $user_input"

# Command substitution
today=$(date +"%Y-%m-%d")
```

## Conditionals

```bash
# If-else
if [ "$name" == "Rajesh" ]; then
  echo "Hello Rajesh"
else
  echo "Hello Guest"
fi

# Test operators
[ -f file.txt ]  # file exists
[ -d dir ]       # directory exists
[ -z "$var" ]    # empty string
[ -n "$var" ]    # non-empty string
```

## Loops

```bash
# For loop
for i in {1..5}; do
  echo "Number $i"
done

# While loop
count=1
while [ $count -le 5 ]; do
  echo $count
  ((count++))
done
```

## Functions

```bash
greet() {
  echo "Hello, $1"
}

greet Rajesh
```

## Arrays

```bash
arr=("apple" "banana" "cherry")
echo ${arr[0]}   # apple
echo ${arr[@]}   # all elements
```

---

# Bash Intermediate Cheat Sheet

## String Manipulation

```bash
str="Hello World"
echo ${#str}           # Length
echo ${str:6:5}        # Substring
echo ${str/World/Bash} # Replace first occurrence
echo ${str//o/X}       # Replace all occurrences
```

## Arithmetic & Math

```bash
a=5
b=3
echo $((a + b))
echo $((a * b))
((a++))
((b--))
```

## Arrays & Associative Arrays

```bash
# Indexed array
arr=(one two three)
for val in "${arr[@]}"; do
  echo $val
done

# Associative array
declare -A colors
colors[red]="#FF0000"
colors[green]="#00FF00"
echo ${colors[red]}
```

## I/O Redirection

```bash
# Redirect stdout
echo "Hello" > file.txt

# Append
echo "World" >> file.txt

# Redirect stderr
ls /nonexistent 2> error.log

# Redirect both stdout & stderr
command &> output.log
```

## Process Management

```bash
sleep 30 &
jobs
fg %1
kill 1234
kill -9 1234
```

## Debugging

```bash
bash -x script.sh
set -x
set +x
```

---

# Bash Advanced Cheat Sheet

## Signals & Traps

```bash
# Trap signals
trap 'echo "Exiting"; exit' SIGINT SIGTERM

# Ignore signal
trap '' SIGINT
```

## Here Documents & Strings

```bash
cat <<EOF > file.txt
Line 1
Line 2
EOF

multi_line=$(cat <<EOF
Hello
World
EOF
)
echo "$multi_line"
```

## Advanced Loops

```bash
# Read file line by line
while IFS= read -r line; do
  echo "$line"
done < file.txt

# Loop with step
for i in {0..10..2}; do
  echo $i
done
```

## Process Substitution

```bash
diff <(sort file1.txt) <(sort file2.txt)
while read line; do echo $line; done < <(ls -1)
```

## Shell Options & Strict Mode

```bash
set -e          # exit on error
set -u          # unset variable error
set -o pipefail # fail if any command in pipeline fails

shopt -s extglob # extended pattern matching
```

## Advanced Functions

```bash
# Function with local variables
sum() {
  local total=$(( $1 + $2 ))
  echo $total
}

result=$(sum 5 7)
echo $result
```

## Bash Tricks & Shortcuts

```bash
# Ternary-like operations
[[ -f file.txt ]] && echo "Exists" || echo "Missing"

# Short-circuit logic
true && echo "This runs"
false || echo "This runs"

# Test multiple conditions
if [[ -f file.txt && -r file.txt ]]; then echo "Readable file"; fi
```

## Networking & Homelab Use

```bash
# Ping multiple hosts
for ip in 192.168.1.{1..5}; do ping -c 1 $ip; done

# Check open ports
for port in 22 80 443; do nc -zv 192.168.1.10 $port; done

# Disk usage alert
[[ $(df / | tail -1 | awk '{print $5}' | sed 's/%//') -gt 80 ]] && echo "Disk >80%"
```

## Debugging & Logging

```bash
# Trace each command
set -x
# Turn off tracing
set +x

# Log output
exec > >(tee -i script.log)
exec 2>&1
```
