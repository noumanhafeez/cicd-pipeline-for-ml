# 📄 Intermediate YAML — Easy Guide with Examples

> **Simple Idea:** This builds on basic YAML by teaching you how to handle multi-line text neatly, inject dynamic values, and store multiple documents in one file.

---

## 1. 📝 Multiline Strings — Block Scalar Format

Sometimes you need to write text that spans **multiple lines** — like a shell script, a log message, or a long description. YAML handles this with **Block Scalar Strings**.

There are **2 styles** and **3 chomping indicators** to control how the text is handled.

---

## 2. 🎨 Style Indicators

### Style 1: Literal Style (`|`)
> **Preserves everything exactly** — line breaks, blank lines, and indentation stay as-is.

Use it for: **shell commands, code snippets, anything where exact formatting matters.**

```yaml
script: |
  echo "Step 1: Install packages"
  pip install -r requirements.txt

  echo "Step 2: Run tests"
  pytest tests/
```

**Output (what the program sees):**
```
echo "Step 1: Install packages"
pip install -r requirements.txt

echo "Step 2: Run tests"
pytest tests/
```
✅ Blank lines and indentation are preserved exactly.

---

### Style 2: Fold Style (`>`)
> **Collapses line breaks into spaces** — like wrapping a long sentence across lines for readability.

Use it for: **long messages or descriptions** where you want clean readable YAML but the output is one continuous line.

```yaml
message: >
  This is a very long message
  that is split across multiple lines
  just to make the YAML file easier to read.
```

**Output (what the program sees):**
```
This is a very long message that is split across multiple lines just to make the YAML file easier to read.
```
✅ All line breaks become spaces → one long sentence.

**Special rule in Fold Style:**
- A **blank line** → stays as a line break
- A line with **extra indentation** → stays as-is (not folded)

```yaml
message: >
  Line one
  Line two

  This line after blank line stays separate.
    This indented line is also preserved.
```

---

## 3. ✂️ Chomping Indicators

**Chomping** controls what happens to **blank lines at the very END** of a multiline string.

They are placed **right after** the style indicator: `|` or `>`

| Indicator | Symbol | What it does |
|-----------|--------|--------------|
| **Clip** (default) | `|` or `>` | Keeps ONE newline at the end |
| **Strip** | `|-` or `>-` | Removes ALL newlines at the end |
| **Keep** | `|+` or `>+` | Keeps ALL newlines at the end |

### Clip (Default) — `|`
```yaml
text: |
  Hello
  World
```
**Output:** `Hello\nWorld\n` ← one newline at end (default)

---

### Strip — `|-`
```yaml
text: |-
  Hello
  World
```
**Output:** `Hello\nWorld` ← NO newline at end (stripped clean)

Use when you don't want any trailing whitespace — great for commands or tokens.

---

### Keep — `|+`
```yaml
text: |+
  Hello
  World


```
**Output:** `Hello\nWorld\n\n\n` ← ALL trailing newlines preserved

Use when blank lines at the end are meaningful (e.g., log formatting).

---

### 🔑 Quick Memory Trick

```
|    →  clip   →  one newline  (default, safe choice)
|-   →  strip  →  zero newlines (clean cut)
|+   →  keep   →  all newlines  (keep everything)
```

---

## 4. 💉 Dynamic Value Injection

Sometimes you don't want to hardcode values in YAML — you want to **pull them from somewhere else dynamically** (like environment variables or another part of the config).

This is done using **expressions** inside `${{ }}`:

```yaml
# Example: GitHub Actions expression
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Print branch name
        run: echo "Branch is ${{ github.ref }}"

      - name: Use secret API key
        run: deploy --key ${{ secrets.API_KEY }}
```

### Another Example — referencing config values:
```yaml
database:
  host: localhost
  port: 5432
  name: mydb

connection_string: "postgresql://${{ database.host }}:${{ database.port }}/${{ database.name }}"
```

### ⚠️ Important Notes:
- `${{ }}` is **not standard YAML** — it's supported by specific tools (like GitHub Actions)
- Not every YAML parser understands it
- Always check if your tool supports expressions before using them

---

## 5. 📚 Multi-Document YAML

You can store **multiple independent YAML documents** in a single file, separated by `---` (three dashes).

### Why use it?
- Group related data together in one file
- Easier organization and management
- Common in Kubernetes config files

```yaml
# Document 1 — Person: Ali
---
name: Ali
age: 25
city: Lahore

# Document 2 — Person: Sara
---
name: Sara
age: 30
city: Karachi
occupation: Engineer

# Document 3 — Person: Bob
---
name: Bob
age: 22
city: Islamabad
occupation: Designer
```

Each `---` starts a completely new, independent YAML document — they don't share data with each other.

---

## 6. 🗺️ Full Symbols Reference

| Symbol | Name | Meaning |
|--------|------|---------|
| `\|` | Literal style | Preserve line breaks exactly |
| `>` | Fold style | Collapse line breaks into spaces |
| `\|` or `>` | Clip (default) | Keep one newline at end |
| `\|-` or `>-` | Strip | Remove all newlines at end |
| `\|+` or `>+` | Keep | Keep all newlines at end |
| `${{ }}` | Expression | Inject dynamic values |
| `---` | Document separator | Start a new YAML document |

> ⚠️ **Rule to remember:** Chomping indicators always come **after** style indicators.
> ✅ Correct: `|-`  ❌ Wrong: `-|`

---

## 7. 📋 Side-by-Side Comparison

```yaml
# Literal — preserves everything
script: |
  line 1
  line 2

  line 4 (blank line preserved)

# Fold — collapses into one line
description: >
  This long sentence
  is written on two lines
  but outputs as one.

# Strip — no trailing newline
token: |-
  abc123

# Keep — all trailing newlines kept
log: |+
  Error occurred


```

---

## ✅ Summary

| Concept | What It Does | Symbol |
|---|---|---|
| **Literal style** | Keeps text exactly as written | `\|` |
| **Fold style** | Wraps long text, joins lines with spaces | `>` |
| **Clip chomping** | One newline at end (default) | `\|` or `>` |
| **Strip chomping** | No newlines at end | `\|-` or `>-` |
| **Keep chomping** | All newlines at end | `\|+` or `>+` |
| **Dynamic injection** | Pull values from environment/config | `${{ value }}` |
| **Multi-document** | Multiple YAML docs in one file | `---` separator |

> ✅ **Key Takeaway:** Intermediate YAML gives you power over *how text is formatted* (`|` vs `>`), *how it ends* (clip/strip/keep), *how to stay flexible* (dynamic values), and *how to organize* (multi-document files). Master these and you can write any CI/CD config with confidence!