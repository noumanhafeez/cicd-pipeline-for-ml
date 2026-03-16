# 🔀 Running Repository Code with GitHub Actions (Pull Request Workflow)

> **Simple Idea:** Instead of only running your pipeline when you push to `main`, you can trigger it when someone opens a **Pull Request** — so the code gets tested *before* it's merged. This is the heart of real-world CI/CD.

---

## 👥 The Shared Repository Model

In a real team, multiple developers work on the **same repository** at the same time. Here's how the workflow looks:

```
Developer A        Developer B
     ↓                  ↓
feature-branch-A   feature-branch-B
     ↓                  ↓
  Open Pull Request (PR)
     ↓
CI/CD automatically runs tests ← GitHub Actions kicks in here!
     ↓
Code Review + Test Results
     ↓
✅ Merge into main  (only if tests pass)
```

### Why trigger on Pull Requests?
- Catches bugs **before** bad code enters `main`
- Gives reviewers **automated test results** alongside the code
- Checks for code quality, security issues, and compatibility
- No manual testing needed — it's all automatic!

---

## 🛠️ Step-by-Step: Setting Up a PR-Triggered Workflow

### Step 1: Create a Feature Branch

Instead of working directly on `main`, create a separate branch for your work:

1. Go to your repo → click **Branch** on the landing page
2. Click **New branch**
3. Name it something descriptive, e.g., `pr-workflow`

Your repo now has two branches:
```
main          ← stable, production-ready code
pr-workflow   ← your new feature branch (work in progress)
```

---

### Step 2: Add Python Code to the Branch

Create a simple Python script called `hello_world.py` in your feature branch:

```python
# hello_world.py
from datetime import datetime

print("Hello, World!")
print(f"Current time: {datetime.now()}")
```

Commit this file to the `pr-workflow` branch.

---

### Step 3: Update the Workflow Trigger

Change the workflow event from `push` to `pull_request`:

```yaml
# ❌ OLD — triggers on push
on:
  push:
    branches:
      - main

# ✅ NEW — triggers when a PR targets main
on:
  pull_request:
    branches:
      - main        # target branch (where code will be merged INTO)
```

> **Key point:** The `branches` key here means *"trigger when a PR is opened that wants to merge INTO main"* — not the branch the PR is coming from.

---

### Step 4: Understand Actions Syntax

Before adding steps, here's how **Actions** (pre-built reusable tasks) work:

```yaml
steps:
  - name: Some step name
    uses: organization/repo-name@version    # Action syntax
    with:
      argument-name: argument-value         # Like passing parameters to a function
```

Think of it like calling a function:
```
actions/checkout@v3
│        │        │
│        │        └── version (like v3, v4)
│        └────────── repository name
└─────────────────── GitHub organization/username
```

**Where to find Actions:**
- [GitHub Marketplace](https://github.com/marketplace?type=actions) — thousands of ready-to-use Actions
- Public GitHub repositories from contributors

---

### Step 5: Add Checkout and Python Setup Steps

To run your Python script, you need two setup steps first:

```yaml
steps:
  # Step 1: Download the repo code onto the runner machine
  - name: Checkout repository
    uses: actions/checkout@v3

  # Step 2: Install and configure Python
  - name: Set up Python
    uses: actions/setup-python@v4
    with:
      python-version: "3.10"       # Choose your Python version

  # Step 3: Run your Python script
  - name: Run hello world script
    run: python hello_world.py
```

**Why do you need `checkout` first?**
The runner is a brand-new, empty virtual machine. It has no idea about your code! The `checkout` Action downloads your repo's code onto that machine so the next steps can use it.

---

## 🔄 The Complete PR Workflow

Here's the full workflow file putting everything together:

```yaml
name: PR Workflow                    # Workflow name

on:
  pull_request:
    branches:
      - main                         # Triggers when PR targets main

jobs:
  build:
    runs-on: ubuntu-latest           # Run on Linux machine

    steps:
      - name: Checkout repository    # Step 1: Get the code
        uses: actions/checkout@v3

      - name: Set up Python          # Step 2: Install Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Run Python script      # Step 3: Run our code
        run: python hello_world.py
```

---

## 🔎 Triggering and Inspecting the Workflow

### How to trigger it:
1. Commit your changes to `pr-workflow` branch
2. Go to GitHub → click **Compare & pull request**
3. Set base branch as `main`, compare branch as `pr-workflow`
4. Click **Create pull request**
5. The workflow **fires immediately!** ⚡

### What you see on the PR page:
```
Pull Request: "Add hello world script"
pr-workflow → main

Checks:
  ⏳ PR Workflow / build (running...)
  ✅ PR Workflow / build (after completion)

  [Details] ← click this to see logs
```

### What the logs show:
```
Job: build
│
├── ✅ Set up job
├── ✅ Checkout repository          ← new step added
├── ✅ Set up Python                ← new step added
├── ✅ Run Python script
│       Hello, World!
│       Current time: 2024-03-16 10:30:45
└── ✅ Complete job
```

---

## 📊 Push vs Pull Request Trigger — Side by Side

| Feature | `on: push` | `on: pull_request` |
|---|---|---|
| **When it runs** | Every time you push code | When a PR is opened/updated |
| **Use case** | Quick checks on any commit | Code review + testing before merge |
| **Protects** | Current branch | The target branch (`main`) |
| **Team benefit** | Individual feedback | Collaborative quality gate |

---

## 💡 Real-World Analogy

Think of it like submitting work for review at a job:

| Work Process | GitHub Actions Equivalent |
|---|---|
| You work on a task separately | Work on a feature branch |
| You submit your work for review | Open a Pull Request |
| Manager auto-checks your work | CI pipeline triggers automatically |
| You get feedback before approval | See test results on the PR |
| Work gets approved and filed | PR is merged into `main` |

---

## 🗺️ Summary

| Concept | What It Does | Example |
|---|---|---|
| **Feature branch** | Isolated workspace for new code | `pr-workflow` branch |
| **Pull Request (PR)** | Request to merge branch into main | `pr-workflow` → `main` |
| `on: pull_request` | Triggers workflow when PR is opened | Replaces `on: push` |
| `branches: [main]` | Target branch of the PR | Code merging INTO main |
| `uses:` | Calls a pre-built Action | `uses: actions/checkout@v3` |
| `with:` | Passes arguments to an Action | `python-version: "3.10"` |
| `actions/checkout@v3` | Downloads repo code onto runner | Must be first step! |
| `actions/setup-python@v4` | Installs Python on the runner | Use `with` to pick version |
| **Details link** | Opens the full job log on the PR page | Click to inspect step output |

---

> ✅ **Key Takeaway:** Triggering CI on Pull Requests = automatic safety net for your `main` branch. Every PR gets tested before merging, so broken code never reaches production. The pattern is always: **checkout → setup environment → run your code.**