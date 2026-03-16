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

### Step 2: Add ML Python Code to the Branch

Create an ML script called `train_model.py` in your feature branch. This script trains a simple classifier and prints the accuracy:

```python
# train_model.py
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Load dataset
data = load_iris()
X, y = data.data, data.target

# Split into train and test sets
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train a Random Forest model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate
predictions = model.predict(X_test)
accuracy = accuracy_score(y_test, predictions)

print(f"✅ Model trained successfully!")
print(f"📊 Test Accuracy: {accuracy * 100:.2f}%")
```

Also create a `requirements.txt` file listing the needed package:

```
scikit-learn
```

Commit both files to the `pr-workflow` branch.

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

### Step 5: Add Checkout, Python Setup, and ML Steps

To run your ML script, you need setup steps first:

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

  # Step 3: Install ML dependencies
  - name: Install dependencies
    run: pip install -r requirements.txt

  # Step 4: Train the ML model and check accuracy
  - name: Train and evaluate model
    run: python train_model.py
```

**Why do you need `checkout` first?**
The runner is a brand-new, empty virtual machine. It has no idea about your code! The `checkout` Action downloads your repo's code onto that machine so the next steps can use it.

---

## 🔄 The Complete ML PR Workflow

Here's the full workflow file putting everything together:

```yaml
name: ML PR Workflow                 # Workflow name

on:
  pull_request:
    branches:
      - main                         # Triggers when PR targets main

jobs:
  train-and-evaluate:
    runs-on: ubuntu-latest           # Run on Linux machine

    steps:
      - name: Checkout repository    # Step 1: Get the code
        uses: actions/checkout@v3

      - name: Set up Python          # Step 2: Install Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Install dependencies   # Step 3: Install scikit-learn etc.
        run: pip install -r requirements.txt

      - name: Train and evaluate model  # Step 4: Run ML script
        run: python train_model.py
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
Pull Request: "Add ML training script"
pr-workflow → main

Checks:
  ⏳ ML PR Workflow / train-and-evaluate (running...)
  ✅ ML PR Workflow / train-and-evaluate (after completion)

  [Details] ← click this to see logs
```

### What the logs show:
```
Job: train-and-evaluate
│
├── ✅ Set up job
├── ✅ Checkout repository              ← downloads your ML code
├── ✅ Set up Python                    ← installs Python 3.10
├── ✅ Install dependencies             ← installs scikit-learn
├── ✅ Train and evaluate model
│       ✅ Model trained successfully!
│       📊 Test Accuracy: 100.00%
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

## 💡 Real-World ML Analogy

Think of it like a data science team reviewing a new model:

| ML Team Process | GitHub Actions Equivalent |
|---|---|
| Data scientist trains a new model | Work on a feature branch |
| Submits model for team review | Open a Pull Request |
| CI automatically trains + evaluates it | Workflow triggers on PR |
| Team sees accuracy score in the PR | Test results shown on PR page |
| Model gets approved and merged | PR merged into `main` |

---

## 🗺️ Summary

| Concept | What It Does | Example |
|---|---|---|
| **Feature branch** | Isolated workspace for new ML code | `pr-workflow` branch |
| **Pull Request (PR)** | Request to merge branch into main | `pr-workflow` → `main` |
| `on: pull_request` | Triggers workflow when PR is opened | Replaces `on: push` |
| `branches: [main]` | Target branch of the PR | Code merging INTO main |
| `uses:` | Calls a pre-built Action | `uses: actions/checkout@v3` |
| `with:` | Passes arguments to an Action | `python-version: "3.10"` |
| `actions/checkout@v3` | Downloads repo code onto runner | Must be first step! |
| `actions/setup-python@v4` | Installs Python on the runner | Use `with` to pick version |
| `pip install -r requirements.txt` | Installs ML libraries like scikit-learn | Run before your ML script |
| **Details link** | Opens the full job log on the PR page | See model accuracy in logs |

---

> ✅ **Key Takeaway:** Triggering CI on Pull Requests = automatic safety net for your ML codebase. Every PR trains and evaluates the model before merging, so broken or underperforming models never reach `main`. The pattern is always: **checkout → setup Python → install ML libraries → run your script.**