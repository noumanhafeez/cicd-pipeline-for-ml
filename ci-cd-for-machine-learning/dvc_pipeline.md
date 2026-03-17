# 📦 DVC Pipelines — Simple Explanation

## 🚀 What is a DVC Pipeline?

A **DVC pipeline** is a way to organize your machine learning workflow into **steps (stages)** so you can:

- Reproduce results easily  
- Avoid unnecessary work  
- Keep everything structured  

---

## 🤔 Why Do We Need It?

In real ML projects, we don’t just train a model directly. We follow steps like:

1. Raw data  
2. Data cleaning & preprocessing  
3. Model training  
4. Evaluation  

These steps:

- Depend on each other  
- Should not always rerun if nothing changed  

👉 Example:  
If you only change model parameters, you **don’t need to clean data again**.

---

## 🔗 Pipeline = DAG (Dependency Graph)

DVC pipelines work like a **graph of steps**:
Raw Data → Preprocessing → Training → Evaluation


This is called a **DAG (Directed Acyclic Graph)**:

- Each step depends on the previous one  
- No circular loops  

---

## 🧱 How DVC Defines Pipelines

DVC uses a file called:
dvc.yaml

Each step (stage) contains:

- **deps** → dependencies (data, scripts)  
- **cmd** → command to run  
- **outs** → outputs (files created)  

---

## ⚙️ Creating Pipeline Stages

You can create a stage using:

```bash
dvc stage add -n preprocess \
              -d raw_data.csv \
              -d preprocess.py \
              -o processed_data.csv \
              python preprocess.py
```

This means:

Input: raw_data.csv

Script: preprocess.py

Output: processed_data.csv

# 🔁 Running the Pipeline

Run the full pipeline:
dvc repro

✔ This will:

Run steps in order

Only run steps that changed

# 🔒 dvc.lock File

When you run the pipeline, DVC creates:
dvc.lock

This file:

Stores exact pipeline state

Ensures reproducibility

👉 Always commit it to Git
Shows a graph of all steps and dependencies

# 🔄 DVC vs GitHub Actions

DVC Pipeline → ML workflow (data + model steps)

GitHub Actions → CI/CD automation

👉 You can even run DVC pipelines inside GitHub Actions