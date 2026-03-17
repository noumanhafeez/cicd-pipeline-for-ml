# 📊 DVC Metrics & Plots — Easy Explanation

## 🚀 What is This About?

DVC helps you:
- Track model performance (metrics)
- Visualize results (plots)
- Compare different experiments

👉 Goal: **Choose the best model easily**

---

## 📈 1. Tracking Metrics

### 🤔 What are Metrics?

Metrics are numbers that tell how good your model is, like:
- Accuracy  
- Precision  
- Recall  

Stored in a file like:
metrics.json


---

## ⚙️ 2. Configure Metrics in `dvc.yaml`

Instead of `outs`, use:

```yaml
metrics:
  - metrics.json:
      cache: false
```
Key Idea:

metrics → performance file

cache: false → store in Git (not DVC cache)

# Show metrics

dvc metrics show
Displays current model performance

Compare Metrics (Very Important)

Steps:

Change something (e.g., hyperparameter)

Run pipeline:
dvc repro

Compare:
dvc metrics diff

# 🤖 5. Use in GitHub Actions (CI/CD)

Instead of running Python scripts:

Run full DVC pipeline:
dvc repro
dvc metrics show --md
This creates a Markdown table for reports

# 🔁 6. Compare with Main Branch

To check if your model is better than existing one:

git fetch origin main
dvc metrics diff main