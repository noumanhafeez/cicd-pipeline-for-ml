# 📄 Introduction to YAML — Easy Guide with Examples

> **Simple Idea:** YAML is just a clean, human-friendly way to write and organize data — like a structured to-do list that both humans and computers can read easily.

---

## 🤔 What is YAML?

**YAML** stands for **"YAML Ain't Markup Language"** (yes, it's a fun, self-referencing name!).

It's a language used to:
- 📋 Write **configuration files** (settings for tools/apps)
- 🔄 **Exchange data** between different programs
- 🏗️ **Represent structured data** in a readable way

### How does it compare to others?

| Language | Looks Like | Human Readable? |
|----------|-----------|-----------------|
| **XML** | `<name>Ali</name>` | Hard to read |
| **JSON** | `{"name": "Ali"}` | Moderate |
| **YAML** | `name: Ali` | Very easy ✅ |

### Where is YAML used?
YAML is heavily used in **CI/CD tools** like:
- **GitHub Actions** — to define automated workflows
- **Data Version Control (DVC)** — to configure ML pipelines

---

## ✍️ YAML Syntax Rules

YAML is very strict about formatting. Here are the key rules:

### 1. Use Spaces, NOT Tabs
```yaml
# ✅ CORRECT — spaces for indentation
person:
  name: Ali
  age: 25

# ❌ WRONG — tabs will cause errors
person:
	name: Ali
```

### 2. Indentation Shows Structure
Just like Python, indentation tells YAML what belongs where.

```yaml
school:
  name: "City High School"
  location: "Lahore"
  grades:
    - 9
    - 10
    - 11
```

### 3. Comments use `#`
```yaml
# This is a comment — YAML ignores this line
name: Ali   # This is an inline comment
```

---

## 🔤 YAML Scalars (Basic Values)

Scalars are the **simple, single values** in YAML — like numbers, text, yes/no, and empty values.

### Strings
```yaml
city: Lahore               # unquoted string (works fine)
greeting: "Hello, World!"  # double-quoted string
message: 'It''s a test'    # single-quoted string
```

### Numbers
```yaml
age: 25          # integer
price: 9.99      # float (decimal)
```

### Booleans (True/False)
```yaml
is_active: true    # ✅ boolean — no quotes!
is_admin: false

# ❌ WRONG — this becomes a STRING, not a boolean
is_active: "true"
```

### Null (Empty/No Value)
```yaml
middle_name: null   # using the keyword null
nickname: ~         # using tilde (~) — same as null
```

---

## 📦 YAML Collections (Lists & Dictionaries)

Collections let you group multiple values together.

### 🔢 Sequences (Lists / Arrays)

**Block Style** (using hyphens `-`):
```yaml
fruits:
  - Apple
  - Banana
  - Mango
```

**Flow Style** (using square brackets `[]`):
```yaml
fruits: [Apple, Banana, Mango]
```

Both mean the exact same thing — just different ways to write it.

### 🗂️ Mappings (Dictionaries / Key-Value Pairs)

**Block Style:**
```yaml
student:
  name: Ali
  age: 20
  city: Lahore
```

**Nested (Dictionary inside Dictionary):**
```yaml
student:
  name: Ali
  grades:
    math: 90
    science: 85
```

**Mixed (Dictionary with a List inside):**
```yaml
person:
  name: Sara
  hobbies:
    - Reading
    - Coding
    - Painting
```

**Flow Style (compact, all on one line):**
```yaml
person: {name: Sara, age: 22, hobbies: [Reading, Coding]}
```

---

## 🌍 Real-World Example: GitHub Actions Config

Here's a simple YAML config used in GitHub Actions (a CI/CD tool):

```yaml
# This runs tests automatically when you push code
name: Run Tests

on: push   # trigger: when code is pushed

jobs:
  test:
    runs-on: ubuntu-latest   # use Ubuntu server
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run Python tests
        run: python -m pytest
```

This is just YAML — a clean, readable config file telling GitHub what to do automatically!

---

## 📝 Quick Reference Cheat Sheet

```yaml
# ── Scalars ──────────────────────────
string_value: Hello World
quoted_string: "Hello, YAML!"
integer: 42
float: 3.14
boolean_true: true
boolean_false: false
empty_value: null
also_empty: ~

# ── Sequence (List) ──────────────────
colors:
  - Red
  - Green
  - Blue

colors_inline: [Red, Green, Blue]

# ── Mapping (Dictionary) ─────────────
person:
  name: Ali
  age: 25
  city: Lahore

# ── Nested Structure ─────────────────
company:
  name: TechCorp
  employees:
    - name: Ali
      role: Developer
    - name: Sara
      role: Designer
```

---

## 🗺️ Summary

| Concept | What It Is | Example |
|---|---|---|
| **YAML** | Human-readable data format | Used in GitHub Actions, DVC |
| **Indentation** | Shows structure (spaces only, no tabs!) | 2 or 4 spaces per level |
| **Scalar** | A single value | `name: Ali`, `age: 25`, `active: true` |
| **Sequence** | An ordered list | `- Apple` or `[Apple, Banana]` |
| **Mapping** | Key-value pairs (dictionary) | `name: Ali` |
| **Comment** | Ignored by YAML | `# This is a comment` |

---

> ✅ **Key Takeaway:** YAML is like writing a well-organized, indented outline. Keep your spacing consistent, never use tabs, and you'll have clean, readable config files every time!