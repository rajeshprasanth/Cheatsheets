# **YAML Complete Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Basic Syntax](#basic-syntax)
3. [Data Structures](#data-structures)

   * [Lists / Arrays](#lists--arrays)
   * [Dictionaries / Maps](#dictionaries--maps)
   * [Nested Structures](#nested-structures)
4. [Anchors, Aliases & Merge Keys](#anchors-aliases--merge-keys)
5. [Special Data Types](#special-data-types)
6. [Comments](#comments)
7. [Advanced Features](#advanced-features)

   * [Tags & Types](#tags--types)
   * [Merge Multiple Maps](#merge-multiple-maps)
   * [Complex Nested Structures](#complex-nested-structures)
8. [Best Practices](#best-practices)
9. [Common Use Cases](#common-use-cases)
10. [Useful Commands / Validation](#useful-commands--validation)
11. [Tips](#tips)

---

## **Overview**

YAML (YAML Ain’t Markup Language) is a human-readable data serialization format widely used for configuration, automation, and data exchange.

Example placeholder:

```yaml
user:
  name: John Doe
  address:
    city: Houston
    state: Texas
    country: USA
```

---

## **Basic Syntax**

```yaml
# Key-value pairs
name: John Doe
age: 35
active: true

# Strings with quotes
single_quoted: 'Hello YAML'
double_quoted: "Hello YAML"

# Multiline strings
description: |
  This is a
  multiline string
description_folded: >
  This string
  folds newlines into spaces
```

---

## **Data Structures**

### **Lists / Arrays**

```yaml
fruits:
  - apple
  - banana
  - cherry

numbers: [1, 2, 3, 4]
nested_list:
  - [1, 2]
  - [3, 4]
```

### **Dictionaries / Maps**

```yaml
person:
  name: John Doe
  age: 35
  address:
    city: Houston
    state: Texas
    country: USA
```

### **Nested Structures**

```yaml
servers:
  - name: web1
    ip: 10.0.0.1
    roles:
      - web
      - api
  - name: db1
    ip: 10.0.0.2
    roles:
      - database
      - backup
```

---

## **Anchors, Aliases & Merge Keys**

```yaml
defaults: &default_settings
  retries: 3
  timeout: 30
  debug: true

server1:
  <<: *default_settings
  host: 10.0.0.1

server2:
  <<: *default_settings
  host: 10.0.0.2
  debug: false
```

---

## **Special Data Types**

```yaml
# Boolean
enabled: true
disabled: false

# Null
value: null
empty: ~

# Numbers
int_value: 42
float_value: 3.14
sci_value: 1e3

# Date / Time
date: 2025-08-20
datetime: 2025-08-20T13:30:00Z
```

---

## **Comments**

```yaml
# Full line comment
key: value  # Inline comment
```

---

## **Advanced Features**

### **Tags & Types**

```yaml
# Explicit type
int_explicit: !!int "42"
float_explicit: !!float "3.14"
bool_explicit: !!bool "true"

# Custom tags (for application-specific types)
!mytag "some value"
```

### **Merge Multiple Maps**

```yaml
defaults: &defaults
  retries: 3
  timeout: 30

overrides: &overrides
  timeout: 60

final_config:
  <<: [*defaults, *overrides]  # Last value wins
  debug: true
```

### **Complex Nested Structures**

```yaml
applications:
  - name: app1
    containers:
      - name: web
        image: nginx:latest
        ports: [80, 443]
      - name: api
        image: node:16
        ports: [3000]
  - name: app2
    containers:
      - name: db
        image: postgres:13
        volumes:
          - db_data:/var/lib/postgresql/data
```

---

## **Best Practices**

* Always use **spaces**, never tabs.
* Keep indentation consistent.
* Use **anchors and aliases** to avoid duplication.
* Use **folded (`>`) vs literal (`|`) strings** for multi-line text.
* Validate YAML files before use in production.

---

## **Common Use Cases**

```yaml
# Docker Compose
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"

# Kubernetes Config
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80

# Ansible Playbook
- hosts: all
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

---

## **Useful Commands / Validation**

```bash
# Check YAML syntax
yamllint file.yaml

# Convert YAML to JSON
python3 -c 'import yaml, json, sys; print(json.dumps(yaml.safe_load(sys.stdin)))' < file.yaml

# Python parse YAML
python3 -c 'import yaml; yaml.safe_load(open("file.yaml"))'
```
