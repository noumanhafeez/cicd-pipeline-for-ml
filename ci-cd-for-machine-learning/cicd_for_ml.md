# 🤖 Model Training with GitHub Actions & CML

> **Simple Idea:** Every time a developer opens a Pull Request with new ML code, GitHub Actions automatically trains the model, evaluates it, and posts a visual report (metrics + confusion matrix) as a comment right on the PR — so the team can review model performance alongside the code.

---

## 🌦️ The Dataset: Weather Prediction in Australia

We use a real-world weather dataset to train a **binary classification model** that predicts: *"Will it rain tomorrow?"*

| Feature Type | Count | Examples |
|---|---|---|
| **Categorical** | 5 | Location, Wind Direction, RainToday |
| **Numerical** | 17 | Temperature, Wind Speed, Rainfall, Humidity |

**Target:** `RainTomorrow` → Yes or No (binary classification)

---

## 🔄 The Modeling Workflow

The pipeline follows these steps in order:

```
Raw Data
   ↓
1. Convert categorical → numerical  (Target Encoding)
   ↓
2. Fill missing values              (Imputation)
   ↓
3. Scale features                   (Standardization)
   ↓
4. Split into Train / Test sets
   ↓
5. Train RandomForestClassifier
   ↓
6. Report Metrics + Plot Confusion Matrix
```

---

## 🛠️ Step-by-Step: Data Preparation

### Step 1: Target Encoding (Categorical → Numerical)

**Problem:** ML models need numbers, but we have text columns like `Location = "Sydney"`.

**Target Encoding solution:** Replace each category with the **average target value** for that category.

```
Example: Does it rain more in Sydney vs Perth?

Location   RainTomorrow        After Target Encoding
---------  ------------   →    ---------------------
Sydney     Yes (1)              Sydney  → 0.45
Sydney     No  (0)              Perth   → 0.30
Sydney     Yes (1)
Perth      No  (0)
Perth      No  (0)
```

So "Sydney" becomes `0.45` (rains 45% of the time) and "Perth" becomes `0.30`.

```python
# Target encoding example
import category_encoders as ce

encoder = ce.TargetEncoder(cols=['Location', 'WindDir', 'RainToday'])
X_encoded = encoder.fit_transform(X_train, y_train)
```

---

### Step 2: Impute Missing Values + Scale Features

```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler

def impute_and_scale_data(X_train, X_test):
    pipeline = Pipeline([
        ('imputer', SimpleImputer(strategy='mean')),  # fill gaps with column mean
        ('scaler', StandardScaler())                  # scale to mean=0, std=1
    ])
    X_train_processed = pipeline.fit_transform(X_train)
    X_test_processed = pipeline.transform(X_test)
    return X_train_processed, X_test_processed
```

**Why scale?** Features like Temperature (range: 0–50) and Rainfall (range: 0–300) are on very different scales. Scaling puts them all on equal footing.

---

### Step 3: Train the Model

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train Random Forest
model = RandomForestClassifier(
    n_estimators=100,
    random_state=42
)
model.fit(X_train, y_train)
```

**Why Random Forest?**
- ✅ High predictive accuracy
- ✅ Resistant to overfitting
- ✅ Handles large number of features well

---

### Step 4: Evaluate — Metrics

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

y_pred = model.predict(X_test)

print(f"Accuracy:  {accuracy_score(y_test, y_pred):.4f}")
print(f"Precision: {precision_score(y_test, y_pred):.4f}")
print(f"Recall:    {recall_score(y_test, y_pred):.4f}")
print(f"F1 Score:  {f1_score(y_test, y_pred):.4f}")
```

| Metric | What It Measures |
|---|---|
| **Accuracy** | % of all predictions that are correct |
| **Precision** | Of predicted "Rain", how many were actually rain? |
| **Recall** | Of actual "Rain" days, how many did we catch? |
| **F1 Score** | Balance between Precision and Recall |

---

### Step 5: Plot Confusion Matrix

```python
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)

sns.heatmap(cm, annot=True, fmt='d', cmap='Blues',
            xticklabels=['No Rain', 'Rain'],
            yticklabels=['No Rain', 'Rain'])
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.title('Confusion Matrix')
plt.savefig('confusion_matrix.png')
```

```
              Predicted
              No Rain   Rain
Actual No Rain  [TN]    [FP]
       Rain     [FN]    [TP]

TN = correctly predicted No Rain
TP = correctly predicted Rain
FP = predicted Rain but was No Rain  (false alarm)
FN = predicted No Rain but was Rain  (missed!)
```

---

## ⚙️ What is CML?

**CML (Continuous Machine Learning)** is an open-source tool that connects ML experiments to GitHub Actions. Think of it as the bridge between your model results and your PR comments.

| CML Can Do | Example |
|---|---|
| Provision compute for training | Spin up a GPU runner |
| Train and evaluate models | Run your Python scripts |
| Compare ML experiments | Show metric changes across PRs |
| Generate visual reports | Post metrics + plots as PR comments |
| Monitor dataset changes | Alert when data drifts |

---

## 🔄 The Complete GitHub Actions + CML Workflow

### The Full YAML File

```yaml
name: ML Model Training

on:
  pull_request:
    branches:
      - main

permissions:
  pull-requests: write      # needed to post comments on PR

jobs:
  train-and-report:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Get the code
      - name: Checkout repository
        uses: actions/checkout@v3

      # Step 2: Set up CML (installs CML tools on runner)
      - name: Setup CML
        uses: iterative/setup-cml@v2

      # Step 3: Set up Python
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      # Step 4: Install ML dependencies
      - name: Install dependencies
        run: pip install -r requirements.txt

      # Step 5: Train model → outputs results.txt + confusion_matrix.png
      - name: Train and evaluate model
        run: python train_model.py

      # Step 6: Build the markdown report + post as PR comment
      - name: Create CML report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}   # needed to post comment
        run: |
          # Write metrics from results.txt into the report
          cat results.txt >> report.md

          # Embed confusion matrix image into the report
          cml asset publish confusion_matrix.png --md >> report.md

          # Post the report as a comment on the PR
          cml comment create report.md
```

---

## 🔁 How It All Flows Together

```
Developer opens PR: "new-feature" → main
           ↓
GitHub Actions triggers automatically
           ↓
Runner (Ubuntu) starts fresh
           ↓
✅ Checkout code
✅ Setup CML
✅ Install Python + packages
✅ Run train_model.py
      → saves results.txt
      → saves confusion_matrix.png
✅ CML builds report.md from both files
✅ CML posts report.md as PR comment
           ↓
PR now has an automatic bot comment:

┌───────────────────────────────────────┐
│ 📊 ML Model Training Report           │
│                                       │
│  Accuracy:   0.8563                   │
│  Precision:  0.8201                   │
│  Recall:     0.7945                   │
│  F1 Score:   0.8071                   │
│                                       │
│  [Confusion Matrix Heatmap Image]     │
└───────────────────────────────────────┘
```

The team can now review **code + model performance** in one place before approving the merge!

---

## 🔑 Key CML Commands Explained

```bash
# Publish an image/file so CML can embed it in markdown
cml asset publish confusion_matrix.png --md >> report.md

# Post a markdown file as a comment on the current PR
cml comment create report.md
```

The `GITHUB_TOKEN` environment variable is what authorizes CML to post the comment on your behalf.

---

## 🗺️ Summary

| Concept | What It Does |
|---|---|
| **Weather Dataset** | Binary classification: will it rain tomorrow? |
| **Target Encoding** | Converts categories to numbers using average target value |
| **Imputation** | Fills missing values with column mean |
| **StandardScaler** | Scales features to mean=0, std=1 |
| **RandomForestClassifier** | Robust ML model — handles many features well |
| **Metrics** | Accuracy, Precision, Recall, F1 Score |
| **Confusion Matrix** | Visual grid showing correct vs wrong predictions |
| **CML** | Tool that turns ML results into automated PR comments |
| `setup-cml` | GitHub Action that installs CML on the runner |
| `cml asset publish` | Embeds an image/file into a markdown report |
| `cml comment create` | Posts the markdown report as a PR comment |
| `GITHUB_TOKEN` | Built-in secret that authorizes CML to post comments |

---

> ✅ **Key Takeaway:** The full pipeline is: open PR → GitHub Actions trains your model → CML collects the metrics and plots → CML posts a visual report as a PR comment automatically. This means your team never has to run the model manually to review it — the results come to them!