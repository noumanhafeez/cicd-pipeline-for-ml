# 🔐 Environment Variables & Secrets in GitHub Actions

> **Simple Idea:** Variables store non-sensitive settings (like model names or file paths), while Secrets store sensitive data (like API keys or passwords) — both let you avoid hardcoding values directly in your workflow files.

---

## 🌍 What are Contexts?

GitHub Actions provides **contexts** — predefined bundles of information automatically available in every workflow run. You access them using the expression syntax:

```yaml
${{ context.variable }}
```

### The 5 Most Useful Contexts

| Context | What It Contains | Example Use |
|---|---|---|
| `github` | Info about the repo & workflow run | `${{ github.ref }}` — current branch name |
| `env` | Variables you define in the workflow | `${{ env.MODEL_NAME }}` |
| `secrets` | Encrypted secret values | `${{ secrets.API_KEY }}` |
| `job` | Info about the currently running job | `${{ job.status }}` |
| `runner` | Info about the machine running the job | `${{ runner.os }}` |

---

## 📦 Variables vs Secrets — Key Difference

| Feature | Variables | Secrets |
|---|---|---|
| **Storage** | Plain text | Encrypted |
| **Used for** | Non-sensitive config | Sensitive data |
| **Examples** | Model name, file path, username | API keys, passwords, tokens |
| **Keyword** | `env` | `secrets` |
| **Visible in logs?** | ✅ Yes | ❌ Never (GitHub hides them) |

---

## 🔤 Variables — Non-Sensitive Configuration

Variables are declared using the `env` keyword and accessed via the `env` context.

### Declaring Variables at Different Levels

#### Workflow-Level (available to ALL jobs)
```yaml
name: ML Training Pipeline

env:
  MODEL_NAME: RandomForest        # available everywhere in this workflow
  PYTHON_VERSION: "3.10"
  DATA_PATH: data/iris.csv

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - name: Print model info
        run: echo "Training model: ${{ env.MODEL_NAME }}"
```

#### Job-Level (available only to that job)
```yaml
jobs:
  train:
    runs-on: ubuntu-latest
    env:
      N_ESTIMATORS: "100"         # only available inside 'train' job
    steps:
      - name: Train model
        run: python train.py --n-estimators ${{ env.N_ESTIMATORS }}
```

#### Step-Level (available only to that step)
```yaml
    steps:
      - name: Evaluate model
        env:
          THRESHOLD: "0.90"       # only available in this one step
        run: python evaluate.py --threshold ${{ env.THRESHOLD }}
```

### Scope Summary
```
Workflow-level env
│
├── Job 1  ← can access workflow-level env
│   ├── Job-level env
│   │   ├── Step 1  ← can access workflow + job level env
│   │   └── Step 2
│   │       └── Step-level env  ← only this step can access it
│   └── Step 3
└── Job 2  ← can access workflow-level env only
```

---

## 🔒 Secrets — Sensitive Data

Secrets are **encrypted** and never shown in logs. Perfect for API keys, passwords, and tokens.

### How to Set a Secret in GitHub

1. Go to your repository on GitHub
2. Click **Settings** (under repo name)
3. In the sidebar → **Security** → **Secrets and Variables** → **Actions**
4. Click the **Secrets** tab → **New repository secret**
5. Fill in:
   - **Name:** `MLFLOW_TRACKING_TOKEN`
   - **Secret:** `your-secret-value-here`
6. Click **Add secret**

```
Repository Settings
└── Security
    └── Secrets and Variables
        └── Actions
            └── Secrets tab
                └── [New repository secret]
                    ├── Name:   MLFLOW_TRACKING_TOKEN
                    └── Value:  ••••••••••••  (encrypted)
```

### Using Secrets in a Workflow

#### As an environment variable (`env` key):
```yaml
steps:
  - name: Log model to MLflow
    env:
      MLFLOW_TOKEN: ${{ secrets.MLFLOW_TRACKING_TOKEN }}
    run: python log_model.py
```

#### As an argument to an Action (`with` key):
```yaml
steps:
  - name: Push model to registry
    uses: some-action/model-push@v1
    with:
      api_key: ${{ secrets.MODEL_REGISTRY_API_KEY }}
```

### ⚠️ Secret Safety in Logs
```
# If you accidentally try to print a secret:
run: echo "${{ secrets.API_KEY }}"

# GitHub will show this in logs:
***
# ← The actual value is ALWAYS hidden automatically!
```

---

## 🤖 The `GITHUB_TOKEN` — Built-in Secret

GitHub automatically provides a special secret called `GITHUB_TOKEN` in **every** workflow run — no setup needed!

### What it can do:
- Clone the repository and fetch code
- Open and close issues and pull requests
- Comment on issues and pull requests
- Interact with the GitHub API

### Accessing it:
```yaml
${{ secrets.GITHUB_TOKEN }}    # access just like any other secret
```

### Adjusting Permissions
By default, `GITHUB_TOKEN` has limited permissions. You can elevate them using the `permissions` key:

```yaml
permissions:
  pull-requests: write    # allow writing comments on PRs
  contents: read          # allow reading repo contents
  issues: write           # allow writing to issues
```

---

## 💬 Full ML Example: Auto-Comment Model Accuracy on a PR

Here's a practical ML workflow that trains a model on a PR and **automatically posts the accuracy as a comment**:

```yaml
name: ML Model CI

on:
  pull_request:
    branches:
      - main

# Elevate permissions to allow commenting on the PR
permissions:
  pull-requests: write

env:
  MODEL_NAME: RandomForest        # workflow-level variable
  PYTHON_VERSION: "3.10"

jobs:
  train-and-report:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Train and evaluate model
        env:
          MLFLOW_TOKEN: ${{ secrets.MLFLOW_TRACKING_TOKEN }}   # secret injected here
        run: |
          python train_model.py > results.txt
          cat results.txt

      - name: Post results as PR comment
        uses: thollander/actions-comment-pull-request@v2
        with:
          message: |
            🤖 **ML Model CI Report**
            - Model: ${{ env.MODEL_NAME }}
            - Status: ✅ Training complete!
          github_token: ${{ secrets.GITHUB_TOKEN }}            # built-in token used here
```

### What happens step by step:
```
PR opened: "Add new RandomForest model"
           ↓
Workflow triggers automatically
           ↓
Runner (Ubuntu) starts
           ↓
Step 1: Downloads your code
Step 2: Installs Python 3.10
Step 3: Installs scikit-learn etc.
Step 4: Trains model, saves results to results.txt
        (MLFLOW_TOKEN injected secretly — never visible in logs)
Step 5: Posts a comment on the PR using GITHUB_TOKEN
           ↓
PR now shows a bot comment:
┌─────────────────────────────────────┐
│ 🤖 ML Model CI Report               │
│ - Model: RandomForest               │
│ - Status: ✅ Training complete!     │
└─────────────────────────────────────┘
```

---

## 🗺️ Summary

| Concept | What It Is | When to Use |
|---|---|---|
| **Context** | Pre-built data bundles from GitHub | Access repo/job/runner info |
| `${{ env.VAR }}` | Access an env variable | Use a variable in a step |
| `${{ secrets.KEY }}` | Access a secret | Use sensitive data safely |
| `${{ github.ref }}` | Current branch/tag name | Conditional logic by branch |
| `env:` at workflow level | Variable available to all jobs | Global config like model name |
| `env:` at step level | Variable only for that step | Step-specific config |
| `secrets.*` | Encrypted, hidden values | API keys, passwords, tokens |
| `GITHUB_TOKEN` | Built-in secret, auto-available | Comment on PRs, open issues |
| `permissions:` | Controls what GITHUB_TOKEN can do | Must set `write` to post comments |

---

> ✅ **Key Takeaway:** Use **variables** (`env`) for things you don't mind seeing in logs — model names, paths, versions. Use **secrets** for anything sensitive — API keys, passwords, tokens. GitHub never prints secrets in logs, and the built-in `GITHUB_TOKEN` lets your workflow interact with GitHub itself (like posting PR comments) without any extra setup.