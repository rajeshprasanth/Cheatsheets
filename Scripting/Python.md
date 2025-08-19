# Python Scripting Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Data Types & Variables](#data-types--variables)
3. [Operators](#operators)
4. [Control Flow](#control-flow)
5. [Functions](#functions)
6. [Modules & Packages](#modules--packages)
7. [File Handling](#file-handling)
8. [Exception Handling](#exception-handling)
9. [List Comprehensions & Generators](#list-comprehensions--generators)
10. [Classes & OOP](#classes--oop)
11. [Decorators & Context Managers](#decorators--context-managers)
12. [Python Standard Libraries](#python-standard-libraries)
13. [Useful Tips & Best Practices](#useful-tips--best-practices)
14. [Homelab Automation Examples](#homelab-automation-examples)

---

## Basics

```python
# Run Python script
python3 script.py

# Interactive shell
python3

# Print output
print("Hello, World!")

# Multi-line comment
"""
This is a multi-line comment
"""
```

---

## Data Types & Variables

```python
# Numbers
x = 10      # int
y = 3.14    # float
z = 1+2j    # complex

# Strings
s = "hello"
s_multi = """Multi-line
string"""

# Boolean
flag = True

# Lists
mylist = [1, 2, 3]

# Tuples (immutable)
mytuple = (1, 2, 3)

# Dictionaries
mydict = {"name": "Alice", "age": 25}

# Sets
myset = {1, 2, 3}

# Type check
type(x)
```

---

## Operators

```python
# Arithmetic
a + b, a - b, a * b, a / b, a // b, a % b, a ** b

# Comparison
a == b, a != b, a > b, a < b, a >= b, a <= b

# Logical
and, or, not

# Membership
in, not in

# Identity
is, is not
```

---

## Control Flow

```python
# If-Else
if x > 0:
    print("Positive")
elif x == 0:
    print("Zero")
else:
    print("Negative")

# For loop
for i in range(5):
    print(i)

# While loop
i = 0
while i < 5:
    print(i)
    i += 1

# Break & Continue
for i in range(10):
    if i == 5: break
    if i % 2 == 0: continue
```

---

## Functions

```python
# Function definition
def greet(name="World"):
    return f"Hello, {name}!"

# Call function
greet("Alice")

# Lambda function
square = lambda x: x**2
square(5)
```

---

## Modules & Packages

```python
# Import standard library
import os, sys, math
from datetime import datetime

# Third-party
import requests

# Check installed packages
pip list

# Create module: mymodule.py
def hello(): print("Hello from module")

# Use module
import mymodule
mymodule.hello()
```

---

## File Handling

```python
# Read file
with open("file.txt", "r") as f:
    data = f.read()

# Write file
with open("file.txt", "w") as f:
    f.write("Hello World")

# Append file
with open("file.txt", "a") as f:
    f.write("\nAppended text")
```

---

## Exception Handling

```python
try:
    x = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"Error: {e}")
finally:
    print("Cleanup actions")
```

---

## List Comprehensions & Generators

```python
# List comprehension
squares = [x**2 for x in range(10)]

# Filter even numbers
evens = [x for x in range(10) if x % 2 == 0]

# Generator expression
gen = (x**2 for x in range(10))
```

---

## Classes & OOP

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, {self.name}!"

# Create object
p = Person("Alice", 25)
p.greet()
```

---

## Decorators & Context Managers

```python
# Decorator
def my_decorator(func):
    def wrapper():
        print("Before function")
        func()
        print("After function")
    return wrapper

@my_decorator
def say_hello():
    print("Hello")

say_hello()

# Context manager
with open("file.txt", "r") as f:
    data = f.read()
```

---

## Python Standard Libraries

```python
# OS & Sys
import os
os.getcwd()
os.listdir(".")

# Date & Time
from datetime import datetime
now = datetime.now()
print(now.strftime("%Y-%m-%d %H:%M:%S"))

# Math & Random
import math, random
math.sqrt(16)
random.randint(1, 100)

# JSON
import json
data = {"name": "Alice"}
json_str = json.dumps(data)
json.loads(json_str)

# Subprocess
import subprocess
subprocess.run(["ls", "-l"])
```

---

## Useful Tips & Best Practices

```python
# Use __main__ to execute script
if __name__ == "__main__":
    print("Script executed directly")

# Type hints
def add(a: int, b: int) -> int:
    return a + b

# F-strings for formatting
name = "Alice"
f"Hello, {name}"

# Virtual environments
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## Homelab Automation Examples

```python
# Ping multiple hosts
import subprocess
hosts = ["10.0.0.1", "10.0.0.2"]
for host in hosts:
    subprocess.run(["ping", "-c", "2", host])

# Parse log file
with open("/var/log/syslog") as f:
    for line in f:
        if "error" in line.lower():
            print(line.strip())

# Monitor disk usage
import shutil
total, used, free = shutil.disk_usage("/")
print(f"Disk Usage: {used/total*100:.2f}%")
```

```python
# Simple API request (for IoT or homelab monitoring)
import requests
response = requests.get("http://10.0.0.91:8123/api/states")
if response.status_code == 200:
    data = response.json()
    print(data)
```
