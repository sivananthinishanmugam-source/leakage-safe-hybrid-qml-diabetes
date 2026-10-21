# Leakage-Safe Hybrid Quantum–Classical Framework for Calibrated Diabetes Risk Stratification

Code accompanying the manuscript:

> Shanmugam SN, Mozumder MAI, Joo M-I, Kim H-C. *A Leakage-Safe Hybrid Quantum-Classical Framework for Calibrated Diabetes Risk Stratification and Explainable Clinical Decision Support.* Submitted to PeerJ Computer Science.

The framework fuses a calibrated XGBoost classifier with a Variational Quantum Classifier using LDA encoding (VQC-LDA). All data-dependent preprocessing (KNN imputation, Min-Max scaling, PCA/LDA, SMOTE) is fitted on the training partition only.

## Repository structure

```
data/            Pima Indians Diabetes Database (diabetes.csv, n = 768)
notebooks/       Analysis notebooks, numbered in execution order
outputs/
  hybrid/        Branch probabilities, fitted XGBoost model, final hybrid predictions
  models/        Fitted preprocessors, quantum model weights, train/test arrays
  metrics/       Classical and quantum branch performance
  plots/         Created at run time
tables/          CSV tables produced by notebooks 05–09
figures/         Figures produced by notebooks 05–09
```

## Notebooks (run in this order)

| # | Notebook | Purpose | Main outputs |
|---|----------|---------|--------------|
| 01 | `01_classical_ml_pipeline.ipynb` | Leakage-safe preprocessing; classical ML branch with SMOTE inside cross-validation; XGBoost selection | `outputs/hybrid/classical_*`, `xgb_model.pkl`, `outputs/metrics/classical_ml_results.csv` |
| 02 | `02_quantum_ml_pipeline.ipynb` | Quantum branch: QSVC and VQC with PCA and LDA encodings | `outputs/hybrid/quantum_test_outputs.csv`, `outputs/models/qml_*` |
| 03 | `03_hybrid_validation_tuning.ipynb` | Training-set calibration, fusion-weight and threshold selection | `outputs/hybrid/hybrid_config_train_selected.csv` |
| 04 | `04_final_test_hybrid_evaluation.ipynb` | Final held-out test evaluation, risk stratification, SHAP (Figures 4–5) | `outputs/hybrid/final_hybrid_predictions_train_calibrated.csv`, `final_hybrid_metrics_train_calibrated.csv` |
| 05 | `05_calibration_analysis.ipynb` | Reliability diagrams and calibration metrics | `tables/calibration_metrics.csv` |
| 06 | `06_hybrid_ablation_study.ipynb` | Ablation and component contributions | `tables/ablation_study_results.csv` |
| 07 | `07_explainability_analysis.ipynb` | SHAP global, dependence and risk-group analysis | `tables/shap_*.csv` |
| 08 | `08_statistical_significance_analysis.ipynb` | Bootstrap CIs, DeLong and McNemar tests | `tables/delong_auc_comparison.csv`, `tables/mcnemar_test_results.csv` |
| 09 | `09_quantum_complexity_analysis.ipynb` | Quantum circuit complexity and computational cost | `tables/quantum_circuit_complexity.csv` |

Notebooks 01–02 are slow, because the quantum circuits are simulated. Their outputs are already included in `outputs/`, so you can run notebooks 04–09 directly to reproduce the reported results. Notebooks 07 and 09 read `outputs/hybrid/final_hybrid_predictions.csv`, which is an earlier-stage predictions file and is included as provided.

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows  (source .venv/bin/activate on Linux/macOS)
pip install -r requirements.txt
jupyter notebook notebooks/
```

All paths are relative to the `notebooks/` folder. Start Jupyter from the repository root, or from `notebooks/`.

## Environment used in the study

- Python 3.11; Qiskit 1.1.1, qiskit-machine-learning 0.8.2, qiskit-aer 0.14.2 (AerSimulator, statevector, noiseless)
- scikit-learn 1.5.1, XGBoost 2.1.1
- Windows 10 Pro (64-bit), Intel Core i7-10700, 16 GB RAM, NVIDIA GeForce RTX 3060

## Data

The Pima Indians Diabetes Database was originally collected by the National Institute of Diabetes and Digestive and Kidney Diseases (NIDDK). It is publicly available from Kaggle: https://www.kaggle.com/datasets/jamaltariqcheema/pima-indians-diabetes-dataset. The dataset is de-identified, and no new participant data were collected.

## Key results (held-out test set, n = 154)

| Model | Accuracy | Recall | Specificity | MCC | ROC-AUC | PR-AUC |
|-------|----------|--------|-------------|-----|---------|--------|
| Hybrid (XGBoost 0.5 + VQC-LDA 0.5, threshold 0.45) | 83.77% | 79.63% | 86.00% | 0.6486 | 88.13% | 83.04% |

## License

Code is released under the MIT License (see `LICENSE`).

## Contact

Corresponding author: Hee-Cheol Kim (heeki@inje.ac.kr), Institute of Digital Anti-Aging HealthCare, Inje University, Republic of Korea.
