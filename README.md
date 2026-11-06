# Leakage-Safe Hybrid Quantum–Classical Framework for Calibrated Diabetes Risk Stratification

Code, data, and intermediate outputs for the manuscript:

> Shanmugam SN, Mozumder MAI, Joo M-I, Kim H-C. *A Leakage-Safe Hybrid Quantum-Classical Framework for Calibrated Diabetes Risk Stratification and Explainable Clinical Decision Support.* Submitted to PeerJ Computer Science.

Archived release (always resolves to the latest version): [https://doi.org/10.5281/zenodo.23527229](https://doi.org/10.5281/zenodo.23527229)

---

## 1. Description

This repository implements a hybrid quantum–classical framework for diabetes risk prediction on the Pima Indians Diabetes Database. It combines:

- a **classical branch**: XGBoost, selected from eight classical models;
- a **quantum branch**: a Variational Quantum Classifier with Linear Discriminant Analysis encoding (VQC-LDA), selected from QSVC and VQC models with PCA and LDA encodings;
- **probability calibration** of the XGBoost branch (quantile binning, fitted on training data);
- **weighted probabilistic fusion** of the two branches (XGBoost weight ω = 0.5) with a tuned classification threshold (τ = 0.45);
- **tertile-based risk stratification** (low / moderate / high), **SHAP explainability**, and **statistical validation** (DeLong, McNemar, bootstrap confidence intervals).

All data-dependent preprocessing (KNN imputation, Min-Max scaling, PCA/LDA, SMOTE) is fitted on the training partition only, so no information from the test set leaks into model development.

On the held-out test set (n = 154) the hybrid model reaches 83.77% accuracy, 79.63% recall, 86.00% specificity, MCC 0.6486, ROC-AUC 88.13%, and PR-AUC 83.04%.

## 2. Dataset Information

| Item | Details |
|------|---------|
| Name | Pima Indians Diabetes Database |
| Original source | National Institute of Diabetes and Digestive and Kidney Diseases (NIDDK) |
| Public copy | OpenML dataset ID 37: https://www.openml.org/d/37 |
| File in this repository | `data/diabetes.csv` (identical to OpenML dataset 37, row for row) |
| Size | 768 records × 9 columns (8 predictors + 1 outcome) |
| Population | Adult females of Pima Indian heritage, aged 21 years or older |
| Classes | `Outcome`: 0 = non-diabetic (500, 65.10%), 1 = diabetic (268, 34.90%) |
| Codebook | `data/CODEBOOK.txt`: variable descriptions, units, coding of the categorical outcome, and zero-coded missing values |
| Privacy | Public, de-identified secondary data; no new participant data were collected |

Zeros in `Glucose`, `BloodPressure`, `SkinThickness`, `Insulin`, and `BMI` are physiologically implausible and are treated as missing values (see the codebook).

## 3. Code Information

```
data/            diabetes.csv and CODEBOOK.txt
notebooks/       Jupyter notebooks, numbered in execution order
outputs/
  hybrid/        Branch probabilities, fitted XGBoost model, final hybrid predictions and metrics
  models/        Fitted preprocessors, quantum model weights, train/test arrays
  metrics/       Classical and quantum branch performance
  plots/         Created at run time
tables/          CSV tables produced by notebooks 05–09
figures/         Figures produced by notebooks 05–09
```

| # | Notebook | Purpose | Main outputs |
|---|----------|---------|--------------|
| 01 | `01_classical_ml_pipeline.ipynb` | Leakage-safe preprocessing; eight classical models with SMOTE inside cross-validation; XGBoost selection | `outputs/hybrid/classical_*`, `xgb_model.pkl`, `outputs/metrics/classical_ml_results.csv` |
| 02 | `02_quantum_ml_pipeline.ipynb` | Quantum branch: QSVC and VQC with PCA and LDA encodings | `outputs/hybrid/quantum_*`, `outputs/models/qml_*` |
| 03 | `03_hybrid_validation_tuning.ipynb` | Training-set calibration, fusion-weight and threshold selection | `outputs/hybrid/hybrid_config_train_selected.csv` |
| 04 | `04_final_test_hybrid_evaluation.ipynb` | Final held-out test evaluation, risk stratification, SHAP (manuscript Figures 4–5) | `outputs/hybrid/final_hybrid_predictions_train_calibrated.csv`, `final_hybrid_metrics_train_calibrated.csv` |
| 05 | `05_calibration_analysis.ipynb` | Reliability diagrams and calibration metrics (Supplementary Figure S1, Table 7) | `tables/calibration_metrics.csv` |
| 06 | `06_hybrid_ablation_study.ipynb` | Ablation study and component contributions (Table 9) | `tables/ablation_study_results.csv` |
| 07 | `07_explainability_analysis.ipynb` | SHAP global, dependence, and risk-group analysis (Table 11) | `tables/shap_*.csv` |
| 08 | `08_statistical_significance_analysis.ipynb` | Bootstrap CIs, DeLong and McNemar tests (Table 13, Supplementary Table S2) | `tables/delong_auc_comparison.csv`, `tables/mcnemar_test_results.csv`, `tables/bootstrap_confidence_intervals.csv` |
| 09 | `09_quantum_complexity_analysis.ipynb` | Quantum circuit complexity and computational cost (Table 5) | `tables/quantum_circuit_complexity.csv` |

Notebooks 07 and 09 read `outputs/hybrid/final_hybrid_predictions.csv`, an earlier-stage predictions file included as provided.

## 4. Requirements

- Python 3.11
- Packages (see `requirements.txt`; versions used in the study are pinned):
  - qiskit 1.1.1, qiskit-machine-learning 0.8.2, qiskit-aer 0.14.2, qiskit-algorithms 0.3.0
  - scikit-learn 1.5.1, xgboost 2.1.1
  - imbalanced-learn, shap, pandas, numpy, scipy, matplotlib, seaborn, joblib, jupyter
- Hardware used: Windows 10 Pro (64-bit), Intel Core i7-10700 (8 cores), 16 GB RAM, NVIDIA GeForce RTX 3060. No GPU or quantum hardware is required; quantum circuits run on the noiseless Qiskit AerSimulator (statevector).

## 5. Usage Instructions

1. **Get the code**
   ```bash
   git clone https://github.com/sivananthinishanmugam-source/leakage-safe-hybrid-qml-diabetes.git
   cd leakage-safe-hybrid-qml-diabetes
   ```
   Or download the archived release from https://doi.org/10.5281/zenodo.23527229.
2. **Create an environment and install dependencies**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate          # Windows   (Linux/macOS: source .venv/bin/activate)
   pip install -r requirements.txt
   ```
3. **Load the dataset.** It is already included at `data/diabetes.csv`; the notebooks read it with a relative path. To load it yourself:
   ```python
   import pandas as pd
   df = pd.read_csv("data/diabetes.csv")   # 768 rows, 9 columns; target column: Outcome
   ```
4. **Run the notebooks** in numerical order from the `notebooks/` folder:
   ```bash
   jupyter notebook notebooks/
   ```
   - **Full pipeline:** run 01 → 09. Notebooks 01–02 are slow because the quantum circuits are simulated.
   - **Reproduce the reported results only:** the outputs of 01–03 are already in `outputs/`, so you can run 04 → 09 directly. Notebook 04 reproduces the final hybrid metrics (accuracy 83.77%, ROC-AUC 88.13%), and 05–09 regenerate the tables in `tables/`.

All paths are relative to the `notebooks/` folder.

## 6. Methodology

1. **Partitioning:** stratified 80/20 train–test split (614 training, 154 test records), performed before any transformation.
2. **Leakage-safe preprocessing (fitted on training data only):** zero-coded values treated as missing → KNN imputation → Min-Max scaling → PCA (4 components) or LDA (1 component, polynomial-expanded to 4 features for quantum encoding) → SMOTE oversampling (training data only; inside each fold during cross-validation).
3. **Classical branch:** eight models (Logistic Regression, SVM, Random Forest, XGBoost, Extra Trees, Decision Tree, Gaussian Naive Bayes, KNN, plus a stacking ensemble) evaluated with 10-fold stratified cross-validation and grid search; XGBoost selected.
4. **Quantum branch:** QSVC and VQC with 4-qubit ZZFeatureMap encodings and a RealAmplitudes ansatz (COBYLA optimizer); VQC-LDA selected.
5. **Calibration and fusion:** XGBoost probabilities calibrated by quantile binning (10 bins, fitted on training data); raw VQC-LDA probabilities fused as `P_hybrid = ω·P_XGB + (1 − ω)·P_VQC`; ω and the threshold τ selected on training data only (ω = 0.5, τ = 0.45).
6. **Evaluation on the held-out test set:** accuracy, precision, recall, specificity, F1-score, MCC, ROC-AUC, PR-AUC; calibration by ECE and Brier score.
7. **Ablation study:** the full hybrid model compared with versions without calibration, without the quantum branch, without threshold optimization (τ = 0.50), and VQC-LDA alone.
8. **Risk stratification and explainability:** tertile-based risk groups from training-set cut-offs; SHAP TreeExplainer on the XGBoost branch.
9. **Statistical validation:** DeLong test (ROC-AUC), McNemar test (paired predictions), and bootstrap 95% confidence intervals (1,000 resamples).

## 7. Citations

If you use this code, please cite the article above and the archived software:

> Shanmugam SN, Mozumder MAI, Joo M-I, Kim H-C. Leakage-safe hybrid quantum–classical framework for diabetes risk stratification (code). Zenodo. https://doi.org/10.5281/zenodo.23527229

Dataset:

> Smith JW, Everhart JE, Dickson WC, Knowler WC, Johannes RS. 1988. Using the ADAP learning algorithm to forecast the onset of diabetes mellitus. *Proceedings of the Annual Symposium on Computer Application in Medical Care*: 261–265.
>
> Pima Indians Diabetes Database. OpenML dataset 37. https://www.openml.org/d/37

## 8. License & Contribution Guidelines

- **License:** the code is released under the MIT License (see `LICENSE`). The dataset remains subject to the terms of its original source.
- **Contributions:** bug reports and suggestions are welcome through GitHub Issues. To propose a change, fork the repository, create a branch, and open a pull request that describes the change and confirms that notebooks 04–09 still reproduce the reported results.

## Contact

Corresponding author: Hee-Cheol Kim (heeki@inje.ac.kr), Institute of Digital Anti-Aging HealthCare, Inje University, Gimhae, Republic of Korea.
