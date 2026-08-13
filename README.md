# A Comparative Study of Explainable Machine Learning Models for Ransomware Attack Detection

> **MSc Cyber Security Dissertation** — University of the West of Scotland (UWS)
> **Student:** Saneesh Ghimire (B01830427)
> **Year:** 2026

---

## Overview

This project implements and compares **five machine learning models** — Decision Tree, Explainable Boosting Machine (EBM), Random Forest, XGBoost, and Logistic Regression — for **ransomware detection** using memory-forensic features. All models are trained on the **CIC-MalMem-2022** dataset and explained using two model-agnostic Explainable AI (XAI) techniques, **SHAP** and **LIME**, alongside each model's intrinsic feature importance. The project measures not only detection accuracy but also **cross-method explanation consistency**, evaluating which models offer the best balance between performance and interpretability for Security Operations Centre (SOC) deployment.

> ⚠️ **Note:** The raw CIC-MalMem-2022 dataset (`MalMem2022.csv`) is **not included** in this repository due to file size. See the [Dataset](#dataset) section for how to download it.

---

## Table of Contents

- [Project Structure](#project-structure)
- [System Architecture](#system-architecture)
- [Dataset](#dataset)
- [Feature Selection](#feature-selection)
- [Model Training](#model-training)
- [Explainability (XAI)](#explainability-xai)
- [Results](#results)
- [Requirements](#requirements)
- [Setup & Installation](#setup--installation)
- [Usage](#usage)
- [Ethical & Legal Notice](#ethical--legal-notice)
- [Citation](#citation)

---

## Project Structure

```
b01830427-MSc-project-ransomware-detection/
│
├── 04_Features/               # Preprocessed and selected feature datasets
│   ├── X_train_selected.csv
│   ├── X_test_selected.csv
│   ├── y_train.csv
│   └── y_test.csv
│
├── 05_Models/                 # Saved trained models
│   ├── dt_model.joblib
│   ├── ebm_model.joblib
│   ├── rf_model.joblib
│   ├── xgb_model.joblib
│   └── lr_model.joblib
│
├── 06_Results/                # Evaluation outputs, tables, and figures
│   ├── confusion_matrices_all.png
│   ├── roc_curves_all.png
│   ├── precision_recall_curves.png
│   ├── performance_bars.png
│   ├── performance_heatmap.png
│   ├── shap_summary_*.png
│   ├── lime_explanation_RF.png
│   ├── model_comparison.csv
│   ├── cross_validation_results.csv
│   ├── shap_rankings_all_models.csv
│   ├── lime_rankings_all_models.csv
│   └── shap_lime_consistency.csv
│
├── 07_Code/                   # Jupyter notebooks (6 sequential phases)
│   ├── 01_Data_Loading_and_Preprocessing.ipynb
│   ├── 02_Feature_Selection.ipynb
│   ├── 03_Model_Training.ipynb
│   ├── 04_SHAP_Analysis.ipynb
│   ├── 05_LIME_Analysis.ipynb
│   └── 06_Cross_Method_Consistency.ipynb
│
└── README.md
```

---

## System Architecture

```
CIC-MalMem-2022 Dataset
(58,596 memory-dump samples, 55 features)
                   │
                   ▼
         Data Cleaning & Preprocessing
         (duplicate check, StandardScaler,
          stratified 80/20 split)
                   │
                   ▼
         Multi-Method Feature Selection
         (Correlation filter, Mutual Information,
          RFECV, SHAP ranking → Consensus scoring)
                   │
                   ▼
         5 ML Models Trained (GridSearchCV, 5-fold CV)
         Decision Tree | EBM | Random Forest |
         XGBoost | Logistic Regression
                   │
                   ▼
         XAI Integration
         SHAP (TreeExplainer / KernelSHAP)
         LIME (TabularExplainer)
         Intrinsic Feature Importance
                   │
                   ▼
         Cross-Method Consistency Analysis
         (Spearman correlation, Top-k overlap)
                   │
                   ▼
         Results & Explainability Reports (06_Results)
```

---

## Dataset

### CIC-MalMem-2022
- **Source:** [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/malmem-2022.html) — publicly available memory-forensic malware dataset
- **Volume:** 58,596 samples (29,298 benign, 29,298 malicious) — perfectly balanced
- **Malware categories:** Ransomware (Maze, Shade, Ako, Pysa, Conti), Spyware (Transponder, Gator, 180Solutions, CWS, TIBS), Trojan Horse (Refroso, Scar, Emotet, Zeus, Reconyc) — 15 families total
- **Features:** 55 numeric features extracted via **VolMemLyzer** from memory dumps (process lists, DLL lists, handles, LDR modules, malfind, psxview, svcscan, callbacks)

### Class Balance
The dataset is naturally balanced 50:50, so no synthetic oversampling (e.g. SMOTE) was required.

> ⚠️ **The raw CSV is not included in this repo.** To reproduce the dataset:
> 1. Download `MalMem2022.csv` from the [CIC-UNB website](https://www.unb.ca/cic/datasets/malmem-2022.html) or [Kaggle](https://www.kaggle.com/datasets/dhoogla/ciccicmalmem2022)
> 2. Place the file in `04_Features/MalMem2022.csv` (or `data/MalMem2022.csv` if running the original notebooks)
> 3. Run the notebooks in order: `01` → `02` → `03` → `04` → `05` → `06`

---

## Feature Selection

Features are reduced from 55 to 20 using a five-stage consensus pipeline:

| Stage | Method | Purpose |
|---|---|---|
| 1 | Correlation Filter (\|r\| > 0.95) | Remove redundant, highly correlated features |
| 2 | Mutual Information | Score each feature's relevance to the target |
| 3 | RFECV (Random Forest) | Recursive elimination with 5-fold CV |
| 4 | SHAP-based Ranking | Rank features by TreeExplainer importance |
| 5 | Consensus Scoring | Weighted combination (0.33 / 0.33 / 0.34) of the above |

All features are standardised (`StandardScaler`) before training.

---

## Model Training

- **Models:** Decision Tree, Explainable Boosting Machine (EBM), Random Forest, XGBoost, Logistic Regression
- **Train/Test Split:** 80% training (46,876 samples), 20% testing (11,720 samples), stratified
- **Cross-validation:** 5-fold stratified cross-validation via `GridSearchCV`
- **Hyperparameter Tuning:** Grid search on model-specific parameter grids (EBM validated via manual 5-fold CV)
- **Class Balancing:** Not required — dataset is naturally balanced

---

## Explainability (XAI)

| Method | Applied To | Purpose |
|---|---|---|
| **SHAP (TreeExplainer)** | Decision Tree, Random Forest, XGBoost | Exact global + local feature attribution |
| **SHAP (KernelSHAP)** | EBM, Logistic Regression | Model-agnostic Shapley approximation |
| **LIME (TabularExplainer)** | All 5 models | Local surrogate explanations per prediction |
| **Intrinsic Importance** | All 5 models | Gini importance, gain, additive terms, coefficients |

Cross-method agreement between SHAP and LIME is measured using **Spearman rank correlation** and **top-5 / top-10 feature overlap** for every model.

---

## Results

Results are saved in `06_Results/`. Key outputs include:

- Accuracy, Precision, Recall, F1-Score, ROC-AUC, PR-AUC per model
- Confusion matrices (all 5 models)
- ROC and Precision-Recall curves (overlaid)
- SHAP summary plots per model
- LIME local explanations
- SHAP vs LIME consistency table (Spearman ρ, top-k overlap)

---

## Requirements

### System Requirements
- Python 3.13
- Jupyter Notebook / JupyterLab
- Minimum 8GB RAM (16GB recommended for SHAP computation)

### Python Dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt`:**
```
numpy
pandas
scikit-learn
xgboost
interpret
shap
lime
matplotlib
seaborn
joblib
scipy
```

---

## Setup & Installation

### 1. Clone the Repository

```bash
git clone https://github.com/saneeshghimire/b01830427-MSc-project-ransomware-detection.git
cd b01830427-MSc-project-ransomware-detection
```

### 2. Create a Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/macOS
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Download the Dataset

Download `MalMem2022.csv` from the [CIC-UNB website](https://www.unb.ca/cic/datasets/malmem-2022.html) and place it in the `data/` folder as described in [Dataset](#dataset).

---

## Usage

Run the notebooks in `07_Code/` sequentially:

### Step 1 — Data Loading and Preprocessing

```bash
jupyter notebook 07_Code/01_Data_Loading_and_Preprocessing.ipynb
```

Loads the CSV, performs a data quality audit, encodes the target, and creates a stratified 80/20 train-test split with `StandardScaler`.

### Step 2 — Feature Selection

```bash
jupyter notebook 07_Code/02_Feature_Selection.ipynb
```

Reduces 55 features to 20 using correlation filtering, Mutual Information, RFECV, and SHAP-based consensus ranking.

### Step 3 — Model Training

```bash
jupyter notebook 07_Code/03_Model_Training.ipynb
```

Trains and tunes all 5 models with `GridSearchCV` and 5-fold cross-validation, then generates performance comparison tables and figures.

### Step 4 — SHAP Analysis

```bash
jupyter notebook 07_Code/04_SHAP_Analysis.ipynb
```

Generates global and local SHAP explanations for all 5 models.

### Step 5 — LIME Analysis

```bash
jupyter notebook 07_Code/05_LIME_Analysis.ipynb
```

Generates local LIME explanations for all 5 models and saves feature rankings.

### Step 6 — Cross-Method Consistency

```bash
jupyter notebook 07_Code/06_Cross_Method_Consistency.ipynb
```

Compares SHAP and LIME feature rankings using Spearman correlation and top-k overlap, producing the final dual-criterion comparison table.

---

## Ethical & Legal Notice

> **This project is strictly for academic and cybersecurity research purposes.**

- No live malware binaries are used or distributed — all data consists of pre-extracted, anonymised memory-forensic **numerical features** from the publicly available CIC-MalMem-2022 dataset.
- The dataset contains no personal data or personally identifiable information.
- No ethical approval was required, as confirmed with the project supervisor.
- The author and the University of the West of Scotland bear no responsibility for any misuse of the techniques described here.

If you are a researcher wishing to reproduce this work, please ensure compliance with your institution's ethics guidelines and applicable laws in your jurisdiction.

---

## Acknowledgements

- [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/) — CIC-MalMem-2022 dataset
- [scikit-learn](https://scikit-learn.org/) — Machine learning library
- [XGBoost](https://xgboost.readthedocs.io/) — Gradient boosting library
- [InterpretML](https://interpret.ml/) — Explainable Boosting Machine implementation
- [SHAP](https://shap.readthedocs.io/) — SHapley Additive exPlanations
- [LIME](https://github.com/marcotcr/lime) — Local Interpretable Model-agnostic Explanations
- University of the West of Scotland, School of Computing

---

## Citation

If you use this work, please cite the CIC-MalMem-2022 dataset:

```
Carrier, T., Victor, P., Tekeoglu, A. and Lashkari, A.H. (2022).
Detecting Obfuscated Malware using Memory Feature Engineering.
Proceedings of the 8th International Conference on Information
Systems Security and Privacy (ICISSP), pp.177-188.
```

---

*For questions or issues, please open a GitHub Issue.*
