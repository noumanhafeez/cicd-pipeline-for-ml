# 📦 Versioning Datasets with Data Version Control (DVC)

> **Simple Idea:** DVC is like Git, but for data files. Git tracks your code changes — DVC tracks your data changes. Together they give you full control over every version of both your code AND your data.

---

## 🤔 Why Does Data Versioning Matter?

Imagine you trained a great model 3 months ago, but today it performs poorly. Without data versioning you can't answer:
- What exact dataset was used back then?
- Did the data change between then and now?
- Can someone else reproduce what you did?

### Key Reasons to Version Your Data

| Reason | Real-World Scenario |
|---|---|
| **Reproducibility** | A colleague wants to replicate your exact model results |
| **Iteration** | You try different dataset versions to improve model accuracy |
| **Collaboration** | Multiple team members use the exact same data |
| **Debugging** | A specific dataset version caused performance to drop — find out which one |
| **Monitoring** | Data changes over time and the model needs retraining |
| **Audit Trail** | Finance/healthcare industries require proof of what data was used |

---

## 🔧 What is DVC?

**DVC (Data Version Control)** is a free, open-source tool that manages data the same way Git manages code.

### Git vs DVC — Side by Side

| Feature | Git | DVC |
|---|---|---|
| **Tracks** | Code files | Data files, models, metrics |
| **Stores** | In `.git` folder | In remote storage (cloud, SSH, local) |
| **File types** | Text, scripts | Large CSVs, images, model binaries |
| **Commands** | `git add`, `git commit` | `dvc add`, `dvc push` |
| **Works alone?** | ✅ Yes | ❌ Needs Git alongside it |

### The Key Insight: Git + DVC Together
```
Your Repository
│
├── Code files (.py, .yaml)  ──► tracked by Git
├── data.csv.dvc             ──► small metadata file tracked by Git
│
└── Remote Storage (S3, GCS, SSH)
    └── data.csv             ──► actual large file tracked by DVC
```

Git stays lightweight (no huge files), DVC handles the heavy lifting.

---

## 🗄️ DVC Storage Options

The actual data lives in a **remote storage** — separate from your Git repo:

```
Local machine  ──► good for testing/development
Cloud Storage  ──► AWS S3, Google Cloud Storage, Azure Blob
SSH/Web APIs   ──► on-site servers
```

### Install DVC
```bash
pip install dvc

# Install with specific cloud support
pip install dvc[s3]    # for AWS S3
pip install dvc[gs]    # for Google Cloud Storage
```

---

## 🚀 Getting Started: Step-by-Step

### Step 1: Initialize Git (required first)
```bash
git init
```

### Step 2: Initialize DVC
```bash
dvc init
```

After running `dvc init`, a `.dvc/` folder is created with:

```
.dvc/
├── .gitignore   ← tells Git which DVC files NOT to track
├── config       ← DVC settings (remote storage location, compression, etc.)
└── tmp/         ← temporary caches and logs
```

### Step 3: Add a Data File to DVC
```bash
dvc add data/weather_australia.csv
```

This does two things:
1. Adds the actual CSV to the **DVC cache** (`.dvc/cache/`)
2. Creates a small **metadata file**: `data/weather_australia.csv.dvc`

### Step 4: Track the `.dvc` File with Git
```bash
git add data/weather_australia.csv.dvc .gitignore
git commit -m "Add weather dataset v1.0"
```

Now Git tracks the tiny `.dvc` file (not the huge CSV), and DVC tracks the actual data.

---

## 🔍 Inside a `.dvc` File

When you run `dvc add data/weather_australia.csv`, DVC creates this metadata file:

```yaml
# data/weather_australia.csv.dvc
outs:
  - md5: a3f8c2e1d4b9e7f0c5a1b2d3e4f5a6b7
    size: 29485312
    hash: md5
    path: weather_australia.csv
```

### What each field means:

| Field | What It Is | Example |
|---|---|---|
| `md5` | A unique fingerprint of the file's contents | Changes if even one byte changes |
| `size` | File size in bytes | `29485312` = ~28 MB |
| `hash` | The hashing algorithm used | Always `md5` |
| `path` | Location of the data file | `weather_australia.csv` |

### The MD5 Checksum — Simple Analogy
Think of `md5` like a **fingerprint** of your data file:
- Same data = same fingerprint ✅
- Even one row changed = completely different fingerprint ✅
- This is how DVC knows if your data has changed

```
weather_australia.csv (original)  →  md5: a3f8c2e1...
weather_australia.csv (1 row added)  →  md5: 9b2d4f7a...  ← different!
```

---

## 🔄 Full Workflow Example: Versioning ML Data

```bash
# 1. Set up Git + DVC
git init
dvc init

# 2. Configure remote storage (e.g., AWS S3)
dvc remote add -d myremote s3://my-ml-bucket/data

# 3. Add your dataset
dvc add data/weather_australia.csv

# 4. Commit the metadata to Git
git add data/weather_australia.csv.dvc .gitignore
git commit -m "Dataset v1.0 - raw weather data"

# 5. Push data to remote storage
dvc push

# --- Later: you update the dataset ---

# 6. Update the file, then re-add
dvc add data/weather_australia.csv

# 7. Commit the new version
git add data/weather_australia.csv.dvc
git commit -m "Dataset v2.0 - added 2024 weather records"
dvc push

# --- Need to go back to v1.0? ---

# 8. Checkout old Git commit, then pull old data
git checkout <commit-hash> data/weather_australia.csv.dvc
dvc pull
```

---

## 🏗️ How It All Fits Together

```
Your Project Folder
│
├── .git/                        ← Git manages code history
├── .dvc/
│   ├── config                   ← DVC settings
│   └── cache/                   ← local copy of tracked data
│
├── data/
│   ├── weather_australia.csv    ← actual data (NOT in Git)
│   └── weather_australia.csv.dvc  ← tiny metadata (IN Git ✅)
│
├── train_model.py               ← tracked by Git
└── requirements.txt             ← tracked by Git

             ↕ dvc push / dvc pull

Remote Storage (S3 / GCS / SSH)
└── weather_australia.csv        ← actual large file lives here
```

---

## 📊 DVC Versioning in Practice: ML Dataset Timeline

```
git commit: "Dataset v1.0"
└── weather_australia.csv.dvc
    └── md5: a3f8c2e1...  (original 3 years of data)

git commit: "Dataset v1.1 - added feature scaling"
└── weather_australia.csv.dvc
    └── md5: 9b2d4f7a...  (same rows, scaled values)

git commit: "Dataset v2.0 - new data source added"
└── weather_australia.csv.dvc
    └── md5: c7e3a1b5...  (50% more rows from new cities)
```

Any team member can `git checkout` any commit and `dvc pull` to get the exact dataset that was used at that point in time.

---

## 🗺️ Summary

| Concept | What It Does | Command |
|---|---|---|
| **DVC** | Tracks data versions alongside Git | — |
| `dvc init` | Sets up DVC in your repo | `dvc init` |
| `dvc add <file>` | Starts tracking a data file | `dvc add data.csv` |
| `.dvc file` | Tiny metadata file Git tracks instead of raw data | `data.csv.dvc` |
| `md5 checksum` | Unique fingerprint of file contents | Changes if data changes |
| `dvc push` | Uploads data to remote storage | `dvc push` |
| `dvc pull` | Downloads data from remote storage | `dvc pull` |
| `.dvc/cache` | Local hidden copy of tracked data | Auto-managed by DVC |
| **Remote storage** | Where actual data files live | S3, GCS, SSH, local |

---

> ✅ **Key Takeaway:** DVC = Git for data. Git stores tiny `.dvc` metadata files, while DVC stores the actual large data files in remote storage. Together they let you version, share, and roll back datasets just as easily as code — essential for reproducible ML.