# ⚙️ Introduction to GitHub Actions — Easy Guide with Examples

> **Simple Idea:** GitHub Actions is like a robot assistant living inside your GitHub repository. Whenever something happens (like you push code), it automatically runs tasks for you — testing, building, deploying — without you lifting a finger.

---

## 🤔 What is GitHub Actions?

**GitHub Actions (GHA)** is a built-in automation and CI/CD system provided by GitHub.

It lets you automatically:
- 🔨 **Build** your code
- 🧪 **Test** your code
- 🚀 **Deploy** your application

All of this happens directly inside your GitHub repository — no external tools needed.

### 🏭 Real-World Analogy: Car Assembly Line

| Car Factory | GitHub Actions |
|---|---|
| Workers perform tasks in order | Steps run in order automatically |
| Attach engine → Paint → Quality check | Build → Test → Deploy |
| Each worker knows their specific job | Each step has a specific task |
| Line stops if one task fails | Workflow stops if a step fails |

---

## 🧩 GitHub Actions Components

There are **5 key components** you need to understand:

```
Event → triggers → Workflow → contains → Jobs → contain → Steps/Actions
                                          (run on Runners)
```

---

### 1. 🎯 Event

An **Event** is what *triggers* (starts) the workflow automatically.

Common events:
- Someone **pushes** code to the repo
- Someone opens a **pull request**
- Someone creates an **issue**
- A **scheduled time** (like every night at midnight)

```yaml
on: push          # triggers when code is pushed
on: pull_request  # triggers when a PR is opened
on:
  schedule:
    - cron: "0 0 * * *"  # triggers every day at midnight
```

---

### 2. 📋 Workflow

A **Workflow** is the full automated process — it's a YAML file that defines everything.

- Stored in `.github/workflows/` folder in your repo
- One repo can have **multiple workflows**
- Each workflow handles a different job

```
your-repo/
├── .github/
│   └── workflows/
│       ├── test.yml        ← runs tests on every push
│       ├── deploy.yml      ← deploys app on every release
│       └── label-issues.yml ← labels new issues automatically
```

---

### 3. 💼 Job

A **Job** is a group of steps that run together.

Key points:
- One workflow can have **multiple jobs**
- Jobs run **in parallel** by default (at the same time)
- You can make one job wait for another if needed
- All steps in a job run on the **same machine**

```yaml
jobs:
  test-job:      # Job 1: runs tests
    runs-on: ubuntu-latest
    steps: ...

  build-job:     # Job 2: builds app (runs at same time as test-job)
    runs-on: ubuntu-latest
    steps: ...
```

---

### 4. 🪜 Steps & Actions

**Steps** are the individual tasks inside a job. They run **one after another** (in order).

Types of steps:
- **Shell scripts** — run terminal commands
- **Programs** — run compiled code
- **Actions** — reusable, pre-built tasks (GitHub's special feature)

```yaml
steps:
  - name: Print a message         # Step 1
    run: echo "Hello, World!"     # shell command

  - name: Run Python script       # Step 2
    run: python app.py            # runs after Step 1
```

**Actions** are like ready-made plugins you can use in steps:
```yaml
steps:
  - name: Checkout my code
    uses: actions/checkout@v3     # pre-built Action from GitHub

  - name: Set up Python
    uses: actions/setup-python@v4 # another pre-built Action
```

---

### 5. 🖥️ Runner

A **Runner** is the virtual machine (computer) that actually runs your job.

Common runners:
| Runner | Operating System |
|--------|-----------------|
| `ubuntu-latest` | Linux (most common) |
| `windows-latest` | Windows |
| `macos-latest` | macOS |

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest   # this job runs on a Linux machine
```

---

## 🔄 How Everything Connects

```
GitHub Repository
│
├── .github/workflows/main.yml
│
│   [EVENT] ──► Someone pushes code
│                      │
│                      ▼
│             [WORKFLOW] triggered
│                      │
│          ┌───────────┴───────────┐
│          ▼                       ▼
│       [JOB 1]                 [JOB 2]
│     Run Tests               Build App
│    (parallel)               (parallel)
│          │
│    [RUNNER: Ubuntu]
│          │
│    Step 1: Checkout code
│    Step 2: Install dependencies
│    Step 3: Run tests
│          │
│          ▼
│    ✅ Pass or ❌ Fail
```

---

## 💻 A Simple Complete Workflow Example

Here's a real GitHub Actions YAML file that runs Python tests every time you push code:

```yaml
# .github/workflows/test.yml

name: Run Python Tests       # name of the workflow

on: push                     # EVENT: triggers on every push

jobs:
  test:                      # JOB name
    runs-on: ubuntu-latest   # RUNNER: Linux machine

    steps:
      - name: Checkout repository         # STEP 1 (Action)
        uses: actions/checkout@v3

      - name: Set up Python               # STEP 2 (Action)
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Install dependencies        # STEP 3 (Shell script)
        run: pip install -r requirements.txt

      - name: Run tests                   # STEP 4 (Shell script)
        run: python -m pytest
```

### What happens step by step:
1. You **push code** to GitHub → Event triggers
2. GitHub spins up an **Ubuntu machine** (Runner)
3. **Step 1:** Downloads your code onto that machine
4. **Step 2:** Installs Python 3.10
5. **Step 3:** Installs your project's packages
6. **Step 4:** Runs all your tests
7. ✅ Green checkmark if all pass, ❌ Red X if something fails

---

## 🗺️ Summary Table

| Component | What It Is | Simple Analogy |
|---|---|---|
| **Event** | What starts the workflow | Alarm clock going off |
| **Workflow** | The full automation plan (YAML file) | Recipe book |
| **Job** | A group of related steps | One chef's job in the kitchen |
| **Step** | A single task inside a job | One cooking instruction |
| **Action** | A reusable pre-built step | A kitchen appliance (does one thing well) |
| **Runner** | The machine that runs the job | The kitchen itself |

---

> ✅ **Key Takeaway:** GitHub Actions = automated robot that watches your repo. When something happens (push, PR, schedule), it fires up a virtual machine, runs your defined steps in order, and tells you if everything passed or failed — all without you doing anything manually!