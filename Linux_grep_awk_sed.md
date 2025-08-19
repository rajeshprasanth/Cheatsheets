# grep, awk, and sed Cheat Sheet

## grep - Global Regular Expression Print

### Basic Usage
```bash
grep "pattern" file.txt              # Search for pattern in file
grep "pattern" file1 file2           # Search in multiple files
grep -r "pattern" /path/             # Recursive search in directory
cat file.txt | grep "pattern"       # Search in piped input
```

### Common Options
```bash
grep -i "pattern" file.txt           # Case insensitive search
grep -v "pattern" file.txt           # Invert match (exclude lines)
grep -n "pattern" file.txt           # Show line numbers
grep -c "pattern" file.txt           # Count matching lines
grep -l "pattern" *.txt              # Show only filenames with matches
grep -L "pattern" *.txt              # Show filenames without matches
grep -w "word" file.txt              # Match whole words only
grep -x "exact line" file.txt        # Match exact lines only
grep -A 3 "pattern" file.txt         # Show 3 lines after match
grep -B 2 "pattern" file.txt         # Show 2 lines before match
grep -C 2 "pattern" file.txt         # Show 2 lines before and after
```

### Regular Expressions
```bash
grep "^start" file.txt               # Lines starting with "start"
grep "end$" file.txt                 # Lines ending with "end"
grep "^$" file.txt                   # Empty lines
grep "[0-9]" file.txt                # Lines containing digits
grep "[a-zA-Z]" file.txt             # Lines containing letters
grep "colou?r" file.txt              # Match "color" or "colour"
grep "files\?" file.txt              # Match "file" or "files"
grep "ab*c" file.txt                 # Match "ac", "abc", "abbc", etc.
grep "ab+c" file.txt                 # Match "abc", "abbc", etc. (not "ac")
grep "a\{3\}" file.txt               # Match exactly 3 'a's
grep "a\{2,4\}" file.txt             # Match 2 to 4 'a's
grep "\<word\>" file.txt             # Match whole word "word"
```

### Extended Regular Expressions (-E or egrep)
```bash
grep -E "pattern1|pattern2" file.txt # OR operation
grep -E "(ab)+c" file.txt            # One or more "ab" followed by "c"
grep -E "^(start|begin)" file.txt    # Lines starting with "start" or "begin"
grep -E "\d+" file.txt               # One or more digits
```

### Practical Examples
```bash
grep -r "TODO" /path/to/code/        # Find TODO comments in code
grep -v "^#" config.conf             # Show non-comment lines
grep -n "error" /var/log/syslog      # Find errors with line numbers
ps aux | grep -v grep | grep python  # Find Python processes
grep -c "^>" sequences.fasta         # Count FASTA sequences
```

---

## awk - Pattern Scanning and Processing

### Basic Syntax
```bash
awk 'pattern { action }' file.txt    # Basic structure
awk '{ print }' file.txt             # Print all lines (like cat)
awk '{ print $1 }' file.txt          # Print first field
awk '{ print $NF }' file.txt         # Print last field
awk '{ print NF }' file.txt          # Print number of fields per line
```

### Built-in Variables
```bash
awk '{ print NR, $0 }' file.txt      # NR = line number
awk '{ print NF, $0 }' file.txt      # NF = number of fields
awk '{ print length($0) }' file.txt  # Length of line
awk '{ print FILENAME }' file.txt    # Current filename
```

### Field Separators
```bash
awk -F: '{ print $1 }' /etc/passwd   # Use colon as field separator
awk -F',' '{ print $2 }' data.csv    # Use comma as separator
awk 'BEGIN{FS=","} { print $1 }' file.csv  # Set separator in BEGIN block
```

### Pattern Matching
```bash
awk '/pattern/ { print }' file.txt   # Print lines matching pattern
awk '!/pattern/ { print }' file.txt  # Print lines NOT matching pattern
awk '/start/,/end/ { print }' file.txt # Print lines between patterns
awk '$1 == "value" { print }' file.txt # Match specific field value
awk '$3 > 100 { print }' file.txt    # Numeric comparison
awk 'NR > 1 { print }' file.txt      # Skip first line
```

### Mathematical Operations
```bash
awk '{ sum += $1 } END { print sum }' numbers.txt        # Sum first column
awk '{ print $1 * $2 }' file.txt                        # Multiply fields
awk '{ if($1 > max) max=$1 } END { print max }' file.txt # Find maximum
awk '{ count++; sum+=$1 } END { print sum/count }' file.txt # Average
```

### String Operations
```bash
awk '{ print toupper($1) }' file.txt          # Convert to uppercase
awk '{ print tolower($1) }' file.txt          # Convert to lowercase
awk '{ print length($1) }' file.txt           # Length of first field
awk '{ print substr($1, 2, 3) }' file.txt     # Substring (start pos 2, length 3)
awk '{ gsub(/old/, "new"); print }' file.txt   # Global substitution
awk '{ print index($1, "substring") }' file.txt # Find substring position
```

### Control Structures
```bash
# If-else
awk '{ if($1 > 100) print "big"; else print "small" }' file.txt

# For loop
awk '{ for(i=1; i<=NF; i++) print $i }' file.txt

# While loop
awk '{ i=1; while(i<=NF) { print $i; i++ } }' file.txt
```

### Practical Examples
```bash
# Print specific columns from CSV
awk -F',' '{ print $1, $3 }' data.csv

# Calculate disk usage summary
df | awk '{ sum += $3 } END { print sum/1024/1024 " GB used" }'

# Process log files
awk '$4 ~ /ERROR/ { print $1, $2, $NF }' access.log

# Pretty print with formatting
awk '{ printf "%-20s %10s\n", $1, $2 }' file.txt
```

---

## sed - Stream Editor

### Basic Usage
```bash
sed 's/old/new/' file.txt            # Replace first occurrence per line
sed 's/old/new/g' file.txt           # Replace all occurrences (global)
sed 's/old/new/2' file.txt           # Replace second occurrence per line
sed -i 's/old/new/g' file.txt        # Edit file in-place
```

### Address Ranges
```bash
sed '2s/old/new/' file.txt           # Replace only on line 2
sed '2,5s/old/new/g' file.txt        # Replace on lines 2-5
sed '/pattern/s/old/new/g' file.txt  # Replace on lines matching pattern
sed '2,/pattern/s/old/new/g' file.txt # Replace from line 2 to pattern match
```

### Delete Operations
```bash
sed '2d' file.txt                    # Delete line 2
sed '2,5d' file.txt                  # Delete lines 2-5
sed '/pattern/d' file.txt            # Delete lines matching pattern
sed '/^$/d' file.txt                 # Delete empty lines
sed '/^#/d' file.txt                 # Delete comment lines
```

### Insert and Append
```bash
sed '2i\New line' file.txt           # Insert before line 2
sed '2a\New line' file.txt           # Append after line 2
sed '/pattern/i\New line' file.txt   # Insert before pattern match
sed '/pattern/a\New line' file.txt   # Append after pattern match
```

### Multiple Commands
```bash
sed 's/old/new/g; s/foo/bar/g' file.txt      # Multiple substitutions
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt # Multiple -e options
sed 's/old/new/g
s/foo/bar/g' file.txt                         # Multi-line script
```

### Advanced Substitutions
```bash
sed 's/[0-9]/X/g' file.txt           # Replace all digits with X
sed 's/^[ \t]*//' file.txt           # Remove leading whitespace
sed 's/[ \t]*$//' file.txt           # Remove trailing whitespace
sed 's/.*/\U&/' file.txt             # Convert to uppercase (\U)
sed 's/.*/\L&/' file.txt             # Convert to lowercase (\L)
```

### Using Different Delimiters
```bash
sed 's|/old/path|/new/path|g' file.txt   # Use | as delimiter
sed 's#old#new#g' file.txt               # Use # as delimiter
```

### Hold Space and Pattern Space
```bash
sed 'N;s/\n/ /' file.txt             # Join lines (N = next line)
sed '1!G;h;$!d' file.txt             # Reverse file (tac equivalent)
sed '$!d' file.txt                   # Print last line only
sed -n '$=' file.txt                 # Count lines (wc -l equivalent)
```

### Practical Examples
```bash
# Remove comments and empty lines
sed '/^#/d; /^$/d' config.file

# Extract email addresses
sed -n 's/.*\([a-zA-Z0-9._%+-]*@[a-zA-Z0-9.-]*\.[a-zA-Z]\{2,\}\).*/\1/p' file.txt

# Convert DOS line endings to Unix
sed 's/\r$//' dos_file.txt

# Add line numbers
sed = file.txt | sed 'N;s/\n/: /'

# Replace tabs with spaces
sed 's/\t/    /g' file.txt

# Remove HTML tags
sed 's/<[^>]*>//g' file.html

# Extract text between patterns
sed -n '/start_pattern/,/end_pattern/p' file.txt
```

---

## Combining grep, awk, and sed

### Powerful Pipelines
```bash
# Extract, process, and format data
grep "ERROR" /var/log/syslog | awk '{ print $1, $2, $5 }' | sed 's/://'

# Find and replace in specific lines
grep -n "TODO" *.txt | sed 's/TODO/DONE/'

# Process CSV data
cat data.csv | grep -v "^#" | awk -F',' '{ print $1, $3 }' | sed 's/,/ -> /g'

# Complex log analysis
cat access.log | awk '$9 >= 400 { print $7 }' | sort | uniq -c | sort -nr

# Clean and format configuration files
grep -v "^#" config.conf | sed '/^$/d' | awk '{ print toupper($1), "=", $3 }'
```

### Performance Tips
- Use `grep` first to filter data before processing with `awk` or `sed`
- `awk` is generally faster than `sed` for complex field processing
- Use `sed` for simple text substitutions
- Consider `cut` for simple field extraction instead of `awk`

### Common Gotchas
- Remember to escape special characters in regular expressions
- Use single quotes to prevent shell interpretation
- Be careful with in-place editing (`-i` option)
- Test commands on sample data first
- Remember that `sed` addresses start from 1, not 0
