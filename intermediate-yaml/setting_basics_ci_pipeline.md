# 🚀 Setting Up a Basic CI Pipeline with GitHub Actions

> **Simple Idea:** A CI (Continuous Integration) pipeline is like a automatic quality-check that runs every time you push code. GitHub Actions lets you set this up in minutes using just a YAML file.

---

## 🧱 Anatomy of a GitHub Actions Workflow

A workflow file has **4 main building blocks**, always written in this order:

```
1. name       →  What is this workflow called?
2. on         →  What event triggers it?
3. jobs       →  What groups of work should run?
4. steps      →  What individual tasks run inside each job?
```

### Full Example: A Basic CI Workflow

```yaml
name: CI                          # 1. Workflow name

on:                               # 2. Trigger event
  push:
    branches:
      - main                      # Only runs when pushing to 'main' branch

jobs:                             # 3. Jobs
  build:                          # Job name: "build"
    runs-on: ubuntu-latest        # Runner: use a Linux machine

    steps:                        # 4. Steps inside the job
      - name: Run a multi-line script
        run: |                    # | = literal style (runs line by line)
          echo "Hello, World!"
          echo "CI pipeline is running!"
          echo "All steps done!"
```

### What happens when you push to `main`:
```
Push code to main
       ↓
Workflow "CI" triggers
       ↓
Job "build" starts on Ubuntu machine
       ↓
Step 1: echo "Hello, World!"
Step 2: echo "CI pipeline is running!"
Step 3: echo "All steps done!"
       ↓
✅ Pipeline succeeds!
```

---

## 🔍 Breaking Down Each Part

### 1. `name` — Workflow Name
```yaml
name: CI
```
Just a label. Appears in the GitHub Actions tab so you can identify it easily.

---

### 2. `on` — Trigger Event
```yaml
on:
  push:
    branches:
      - main
```
This tells GitHub: *"Only run this workflow when someone pushes code to the `main` branch."*

Other common triggers:
```yaml
on: push                    # any push to any branch
on: pull_request            # when a PR is opened
on:
  push:
    branches:
      - main
      - dev                 # run on pushes to main OR dev
```

---

### 3. `jobs` — Groups of Work
```yaml
jobs:
  build:                    # you can name this anything
    runs-on: ubuntu-latest  # the machine type
```
- `build` is the **job name** (you choose it)
- `runs-on` tells GitHub which virtual machine to use
- All steps inside this job run on **the same machine**

---

### 4. `steps` — Individual Tasks
```yaml
    steps:
      - name: Run a multi-line script
        run: |
          echo "Hello, World!"
          echo "Step 2"
          echo "Step 3"
```
- Each step has a **name** (description) and a **run** command
- The `|` (pipe/literal style) means each `echo` runs **one after another**
- Steps always run **in order**, top to bottom

---

## 🛠️ Step-by-Step: Setting Up Your First GitHub Action

### Step 1: Create a GitHub Repository
Go to: `https://github.com/new`
- Add a Python `.gitignore`
- Choose a license
- Click **Create repository**

---

### Step 2: Enable GitHub Actions
Go to:
```
https://github.com/your-username/your-repo/settings/actions
```
Make sure Actions permissions are set to **"Allow all actions"**.

---

### Step 3: Create the Workflow File
1. On your repo page, click **Actions** in the top menu
2. Search for **"Simple Workflow"** and click **Configure**
3. You'll see a YAML editor — write your workflow here

> ⚠️ **Important:** All workflow files MUST be saved in this exact folder:
> ```
> .github/workflows/your-workflow-name.yml
> ```

Your repo structure should look like:
```
your-repo/
├── .github/
│   └── workflows/
│       └── ci.yml        ← your workflow file lives here
├── main.py
├── requirements.txt
└── README.md
```

---

### Step 4: Commit the Workflow
1. Write your YAML content in the editor
2. Type a commit message (e.g., `"Add CI workflow"`)
3. Click **Commit changes**
4. The workflow **kicks off immediately!** 🎉

---

## 🔎 Inspecting the Pipeline Run

### On the Repo Landing Page
After committing, look next to your latest commit — you'll see one of these icons:

| Icon | Color | Meaning |
|------|-------|---------|
| ⏳ | **Amber/Yellow** circle | Workflow is currently running |
| ✅ | **Green** tick | Workflow succeeded |
| ❌ | **Red** cross | Workflow failed |

---

### In the Actions Tab
Click **Actions** in the top menu → you'll see a list of all workflow runs.

```
Actions Tab
│
└── ✅ CI  (your workflow name)
       └── build  (your job name)
              ├── Set up job
              ├── Run a multi-line script   ← your custom step
              │     Hello, World!
              │     CI pipeline is running!
              │     All steps done!
              └── Complete job
```

Click on a run → click on a job → see the **full logs** of every step, including output!

---

## 💡 Real-World CI Pipeline Example

Here's a more practical CI pipeline that actually tests Python code:

```yaml
name: Python CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code              # Download your repo onto the runner
        uses: actions/checkout@v3

      - name: Set up Python              # Install Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Install dependencies       # Install packages
        run: pip install -r requirements.txt

      - name: Run tests                  # Run your tests
        run: python -m pytest tests/
```

Every time you push to `main`, this automatically:
1. Grabs your latest code
2. Sets up Python
3. Installs all packages
4. Runs your test suite
5. Reports ✅ pass or ❌ fail

---

## 🗺️ Summary

| Concept | What It Does | Example |
|---|---|---|
| `name` | Names the workflow | `name: CI` |
| `on: push: branches` | Triggers on push to specific branch | `branches: [main]` |
| `jobs` | Defines groups of work | `build:` |
| `runs-on` | Chooses the virtual machine | `ubuntu-latest` |
| `steps` | Lists tasks inside a job | `run: echo "hello"` |
| `|` after `run` | Runs multiple commands in order | literal block style |
| `.github/workflows/` | Where all workflow YAML files live | Required folder path |
| ✅ / ❌ / ⏳ | Pipeline status icons on GitHub | Shown next to each commit |

---

> ✅ **Key Takeaway:** Setting up CI with GitHub Actions = creating one YAML file in `.github/workflows/`. Define the trigger (`on`), the machine (`runs-on`), and the tasks (`steps`). Push your code, and GitHub automatically runs your pipeline and tells you if everything passed!