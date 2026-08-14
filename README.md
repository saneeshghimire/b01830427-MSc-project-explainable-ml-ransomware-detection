# A Comparative Study of Explainable Machine Learning Models for Ransomware Attack Detection

> **MSc Cyber Security Dissertation** — University of the West of Scotland (UWS)
> **Student:** Saneesh Ghimire (B01830427)
> **Year:** 2026

---

## Overview

This project implements and compares **five machine learning models** — Decision Tree, Explainable Boosting Machine (EBM), Random Forest, XGBoost, and Logistic Regression — for **ransomware detection** using memory-forensic features. All models are trained on the **CIC-MalMem-2022** dataset and explained using two model-agnostic Explainable AI (XAI) techniques, **SHAP** and **LIME**, alongside each model's intrinsic feature importance. The project measures not only detection accuracy but also **cross-method explanation consistency**, evaluating which models offer the best balance between performance and interpretability for Security Operations Centre (SOC) deployment.

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
b01830427-MSc-project-explainable-ml-ransomware-detection/
│
├── Code/                        # Jupyter notebooks (6 sequential phases)
│   ├── 01_Data_Loading_and_Preprocessing.ipynb
│   ├── 02_Feature_Selection.ipynb
│   ├── 03_Model_Training.ipynb
│   ├── 04_SHAP_Analysis.ipynb
│   ├── 05_LIME_Analysis.ipynb
│   └── 06_Cross_Method_Consistency.ipynb
│
├── data/                        # Raw and processed datasets
│   ├── MalMem2022.csv           # Raw CIC-MalMem-2022 dataset
│   ├── X_train_unscaled.csv
│   ├── X_test_unscaled.csv
│   ├── X_train_selected.csv     # After feature selection (20 features)
│   ├── X_test_selected.csv
│   ├── y_train.csv
│   └── y_test.csv
│
├── models/                      # Saved trained models
│   ├── dt_model.joblib
│   ├── ebm_model.joblib
│   ├── rf_model.joblib
│   ├── xgb_model.joblib
│   └── lr_model.joblib
│
├── results/                     # Evaluation outputs, tables, and figures
│   ├── figures/
│   │   ├── category_distribution.png
│   │   ├── confusion_matrices_all.png
│   │   ├── dual_criterion_comparison.png
│   │   ├── feature_importance_RF.png
│   │   ├── lime_explanation_RF.png
│   │   ├── mutual_information_scores.png
│   │   ├── performance_bars.png
│   │   ├── performance_heatmap.png
│   │   ├── precision_recall_curves.png
│   │   └── roc_curves_all.png
│   │
│   ├── shap_plots/
│   │   ├── shap_summary_DecisionTree.png
│   │   ├── shap_summary_EBM.png
│   │   ├── shap_summary_LR.png
│   │   ├── shap_summary_RandomForest.png
│   │   ├── shap_summary_XGBoost.png
│   │   └── shap_waterfall_RF.png
│   │
│   └── tables/
│       ├── cross_validation_results.csv
│       ├── dual_criterion_comparison.csv
│       ├── feature_importance_RF.csv
│       ├── final_model_comparison.csv
│       ├── lime_rankings_all_models.csv
│       ├── model_comparison.csv
│       ├── shap_lime_consistency.csv
│       └── shap_rankings_all_models.csv
│
├── .gitignore
└── README.md
```

---

## System Architecture

```
CIC-MalMem-2022 Dataset
(58,596 memory-dump samples, 55 features)
                   │
                   ▼
         Data Loading & Preprocessing
         (data/MalMem2022.csv → data quality audit,
          StandardScaler, stratified 80/20 split)
                   │
                   ▼
         Multi-Method Feature Selection
         (Correlation filter, Mutual Information,
          RFECV, SHAP ranking → Consensus scoring)
         → data/X_train_selected.csv, X_test_selected.csv
                   │
                   ▼
         5 ML Models Trained (GridSearchCV, 5-fold CV)
         Decision Tree | EBM | Random Forest |
         XGBoost | Logistic Regression
         → models/*.joblib
                   │
                   ▼
         XAI Integration
         SHAP (TreeExplainer / KernelSHAP)
         LIME (TabularExplainer)
         Intrinsic Feature Importance
         → results/shap_plots/, results/figures/
                   │
                   ▼
         Cross-Method Consistency Analysis
         (Spearman correlation, Top-k overlap)
         → results/tables/shap_lime_consistency.csv
```

---

## Dataset

### CIC-MalMem-2022
- **Source:** [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/malmem-2022.html) — publicly available memory-forensic malware dataset
- **Volume:** 58,596 samples (29,298 benign, 29,298 malicious) — perfectly balanced
- **Malware categories:** Ransomware (Maze, Shade, Ako, Pysa, Conti), Spyware (Transponder, Gator, 180Solutions, CWS, TIBS), Trojan Horse (Refroso, Scar, Emotet, Zeus, Reconyc) — 15 families total
- **Features:** 55 numeric features extracted via **VolMemLyzer** from memory dumps (process lists, DLL lists, handles, LDR modules, malfind, psxview, svcscan, callbacks)
- **Location in repo:** `data/MalMem2022.csv`

### Class Balance
The dataset is naturally balanced 50:50, so no synthetic oversampling (e.g. SMOTE) was required.

> If cloning this repo fresh, ensure `data/MalMem2022.csv` is present before running `Code/01_Data_Loading_and_Preprocessing.ipynb`. It can also be downloaded from the [CIC-UNB website](https://www.unb.ca/cic/datasets/malmem-2022.html) or [Kaggle](https://www.kaggle.com/datasets/dhoogla/ciccicmalmem2022).

---

## Feature Selection

Features are reduced from 55 to 20 using a five-stage consensus pipeline in `Code/02_Feature_Selection.ipynb`:

| Stage | Method | Purpose |
|---|---|---|
| 1 | Correlation Filter (\|r\| > 0.95) | Remove redundant, highly correlated features |
| 2 | Mutual Information | Score each feature's relevance to the target |
| 3 | RFECV (Random Forest) | Recursive elimination with 5-fold CV |
| 4 | SHAP-based Ranking | Rank features by TreeExplainer importance |
| 5 | Consensus Scoring | Weighted combination (0.33 / 0.33 / 0.34) of the above |

All features are standardised (`StandardScaler`) before training. Selected features are saved to `data/X_train_selected.csv` and `data/X_test_selected.csv`.

---

## Model Training

- **Models:** Decision Tree, Explainable Boosting Machine (EBM), Random Forest, XGBoost, Logistic Regression
- **Train/Test Split:** 80% training (46,876 samples), 20% testing (11,720 samples), stratified
- **Cross-validation:** 5-fold stratified cross-validation via `GridSearchCV`
- **Hyperparameter Tuning:** Grid search on model-specific parameter grids (EBM validated via manual 5-fold CV)
- **Class Balancing:** Not required — dataset is naturally balanced
- **Output:** Trained models saved to `models/*.joblib`, performance tables saved to `results/tables/`

---

## Explainability (XAI)

| Method | Applied To | Notebook | Purpose |
|---|---|---|---|
| **SHAP (TreeExplainer)** | Decision Tree, Random Forest, XGBoost | `04_SHAP_Analysis.ipynb` | Exact global + local feature attribution |
| **SHAP (KernelSHAP)** | EBM, Logistic Regression | `04_SHAP_Analysis.ipynb` | Model-agnostic Shapley approximation |
| **LIME (TabularExplainer)** | All 5 models | `05_LIME_Analysis.ipynb` | Local surrogate explanations per prediction |
| **Intrinsic Importance** | All 5 models | `03_Model_Training.ipynb` | Gini importance, gain, additive terms, coefficients |

Cross-method agreement between SHAP and LIME is measured using **Spearman rank correlation** and **top-5 / top-10 feature overlap** for every model in `06_Cross_Method_Consistency.ipynb`.

---

## Results

Results are saved in `results/`, organised into `figures/`, `shap_plots/`, and `tables/`.

**`results/figures/`**

| File | Description |
|---|---|
| `category_distribution.png` | Class/category distribution in the raw dataset |
| `confusion_matrices_all.png` | Confusion matrices for all 5 models |
| `dual_criterion_comparison.png` | dual criteria comparison of SHAP & LIME |
| `feature_importance_RF.png` | Random Forest intrinsic feature importance |
| `lime_explanation_RF.png` | LIME local explanation for a single prediction (Random Forest) |
| `mutual_information_scores.png` | Mutual Information scores from feature selection |
| `performance_bars.png` | Bar chart comparing Accuracy, Precision, Recall, F1 across models |
| `performance_heatmap.png` | Heatmap of all performance metrics across models |
| `precision_recall_curves.png` | Overlaid Precision-Recall curves for all 5 models |
| `roc_curves_all.png` | Overlaid ROC curves for all 5 models |

**`results/shap_plots/`**

| File | Description |
|---|---|
| `shap_summary_DecisionTree.png` | SHAP global summary plot — Decision Tree |
| `shap_summary_EBM.png` | SHAP global summary plot — EBM (KernelSHAP) |
| `shap_summary_LR.png` | SHAP global summary plot — Logistic Regression (KernelSHAP) |
| `shap_summary_RandomForest.png` | SHAP global summary plot — Random Forest |
| `shap_summary_XGBoost.png` | SHAP global summary plot — XGBoost |
| `shap_waterfall_RF.png` | SHAP waterfall plot for a single prediction — Random Forest |

**`results/tables/`**

| File | Description |
|---|---|
| `model_comparison.csv` | Test-set Accuracy, Precision, Recall, F1, ROC-AUC, PR-AUC per model |
| `final_model_comparison.csv` | Consolidated final comparison table across all 5 models |
| `cross_validation_results.csv` | 5-fold CV mean ± std for all models |
| `feature_importance_RF.csv` | Random Forest intrinsic feature importance scores |
| `shap_rankings_all_models.csv` | SHAP feature rankings per model |
| `lime_rankings_all_models.csv` | LIME feature rankings per model |
| `shap_lime_consistency.csv` | Spearman ρ and top-k overlap between SHAP and LIME per model |
| `dual_criterion_comparison.csv` | Final dual-criterion table balancing detection accuracy against explanation stability |

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
git clone https://github.com/saneeshghimire/b01830427-MSc-project-explainable-ml-ransomware-detection.git
cd b01830427-MSc-project-explainable-ml-ransomware-detection
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

### 4. Verify the Dataset

Confirm `data/MalMem2022.csv` is present. If not, download it as described in [Dataset](#dataset).

---

## Usage

Run the notebooks in `Code/` sequentially — each notebook depends on files created by the previous one:

### Step 1 — Data Loading and Preprocessing

```bash
jupyter notebook Code/01_Data_Loading_and_Preprocessing.ipynb
```

Loads `data/MalMem2022.csv`, performs a data quality audit, encodes the target, and creates a stratified 80/20 train-test split with `StandardScaler`.

### Step 2 — Feature Selection

```bash
jupyter notebook Code/02_Feature_Selection.ipynb
```

Reduces 55 features to 20 using correlation filtering, Mutual Information, RFECV, and SHAP-based consensus ranking. Outputs `data/X_train_selected.csv` and `data/X_test_selected.csv`.

### Step 3 — Model Training

```bash
jupyter notebook Code/03_Model_Training.ipynb
```

Trains and tunes all 5 models with `GridSearchCV` and 5-fold cross-validation. Saves models to `models/` and comparison tables/figures to `results/`.

### Step 4 — SHAP Analysis

```bash
jupyter notebook Code/04_SHAP_Analysis.ipynb
```

Generates global and local SHAP explanations for all 5 models, saved to `results/shap_plots/`.

### Step 5 — LIME Analysis

```bash
jupyter notebook Code/05_LIME_Analysis.ipynb
```

Generates local LIME explanations for all 5 models and saves feature rankings to `results/tables/lime_rankings_all_models.csv`.

### Step 6 — Cross-Method Consistency

```bash
jupyter notebook Code/06_Cross_Method_Consistency.ipynb
```

Compares SHAP and LIME feature rankings using Spearman correlation and top-k overlap, producing the final dual-criterion comparison table in `results/tables/shap_lime_consistency.csv`.

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
