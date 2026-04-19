
<div align="center">

# Breast Cancer Prognosis — Explainable ML

**Clinical-Genomic Predictive Modeling for Breast Cancer Prognosis**  
*An Explainable Machine Learning Approach Using the METABRIC Cohort*

<br>

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-1.7+-189AB4?style=flat-square)
![SHAP](https://img.shields.io/badge/SHAP-0.41+-FF6B6B?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-00C853?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-00C853?style=flat-square)
![Dataset](https://img.shields.io/badge/Dataset-METABRIC-6C63FF?style=flat-square)

<br>

*Sashi Bharadwaj Pulikanti — [ORCID 0009-0002-8904-2209](https://orcid.org/0009-0002-8904-2209)*

</div>

---

## Overview

This study develops and evaluates an explainable machine learning pipeline
for breast cancer survival prediction using the METABRIC cohort. We
demonstrate that fourteen routine clinical variables achieve competitive
discriminative performance (AUC = 0.725, 95% CI: 0.707–0.757) without
requiring genomic profiling — and that SHAP-based explainability reveals
clinically meaningful patterns, including a treatment effect signal embedded
in HER2 status that the model learned without explicit treatment labels.

---

## Key Findings

| Finding | Value |
|---------|-------|
| XGBoost Clinical AUC | 0.725 (95% CI: 0.707–0.757) |
| Logistic Regression AUC | 0.712 |
| NPI Clinical Baseline AUC | ~0.630 |
| Cross-Validation AUC | 0.734 ± 0.016 (5-fold) |
| Bootstrap AUC | 0.733 (95% CI: 0.707–0.757) |
| Top Predictor | Age At Diagnosis (mean \|SHAP\| = 0.713) |
| Age Dominance | 2.9× over Nottingham Prognostic Index |
| HER2+ Mean SHAP | −0.281 (survival benefit) |
| HER2 KM Log-rank p | 0.0093 |
| Risk Group KM Log-rank p | < 0.0001 |
| Genomic Augmentation Gain | +0.000 AUC (clinical features sufficient) |

---

## Dataset

**METABRIC** — Molecular Taxonomy of Breast Cancer International Consortium

| Property | Value |
|----------|-------|
| Total patients | 1,904 |
| Training set | 1,523 |
| Test set | 381 |
| Clinical features | 14 |
| Gene features evaluated | 489 |
| Gene features selected | 20 |
| Class balance (test) | Alive = 160, Died = 221 |
| Class ratio | 1.38:1 (Died:Alive) |

> **Target coding:** `overall_survival = 1` means Alive, `0` means Died.
> Negative SHAP values indicate increased mortality risk.

**Sources**
- Curtis C, et al. *Nature.* 2012;486:346–352. [doi:10.1038/nature10983](https://doi.org/10.1038/nature10983)
- Pereira B, et al. *Nature Communications.* 2016;7:11479. [doi:10.1038/ncomms11479](https://doi.org/10.1038/ncomms11479)
- Cerami E, et al. *Cancer Discovery.* 2012;2(5):401–404. [doi:10.1158/2159-8290.CD-12-0095](https://doi.org/10.1158/2159-8290.CD-12-0095)
- Gao J, et al. *Science Signaling.* 2013;6(269):pl1. [doi:10.1126/scisignal.2004088](https://doi.org/10.1126/scisignal.2004088)
- cBioPortal: [cbioportal.org](https://www.cbioportal.org)
- Kaggle: [raghadalharbi/breast-cancer-gene-expression-profiles-metabric](https://www.kaggle.com/datasets/raghadalharbi/breast-cancer-gene-expression-profiles-metabric)

---

## Methods

```
Raw Data (1,904 patients, 693 columns)
        │
        ▼
Clinical Feature Selection (14 features)
        │
        ├──────────────────────────────────────┐
        ▼                                      ▼
Logistic Regression              XGBoost (scale_pos_weight=1.376)
AUC = 0.712                      AUC = 0.725
        │                                      │
        │                              SHAP TreeExplainer
        │                                      │
        │                         ┌────────────┼────────────┐
        │                         ▼            ▼            ▼
        │                    Beeswarm      Waterfall   Interaction
        │                    (global)    (Patient 234)  Values
        │                                              (14×14)
        │
        ▼
Gene Selection (mutual_info_classif → top 20 of 489)
        │
        ▼
XGBoost Combined (14 clinical + 20 gene = 34 features)
AUC = 0.725 — no improvement over clinical
        │
        ▼
Clinical Validation
Kaplan-Meier — HER2 (p = 0.0093) + Risk Groups (p < 0.0001)
```

---

## Pipeline

| Cell | Description | Output |
|------|-------------|--------|
| 1 | Imports and global constants | — |
| 2 | Data loading and shape assertions | — |
| 3 | Clinical feature selection (18→14) | — |
| 4 | Missing value analysis | fig_01 |
| 5 | Target variable distribution | fig_02 |
| 6 | HER2 EDA | fig_03 |
| 7 | Clinical feature analysis | fig_04, fig_05, fig_06 |
| 8 | Correlation heatmap | fig_07 |
| 9 | Save cleaned clinical data | metabric_clinical.csv |
| 10 | Drop leakage columns | — |
| 11 | Encode categorical features | — |
| 12 | Impute missing values | — |
| 13 | Stratified train/test split | — |
| 14 | Assertions and save splits | splits.pkl |
| 15 | Logistic Regression | fig_08, fig_12 |
| 16 | XGBoost clinical model | fig_09, fig_13 |
| 17 | Model comparison figures | fig_10, fig_12, fig_13 |
| 18 | Save best model | best_model.pkl |
| 19 | SHAP initialization | — |
| 20 | SHAP beeswarm | fig_14 |
| 21 | SHAP bar plot | fig_15 |
| 22 | SHAP waterfall — Patient 234 (false negative case study) | fig_16 |
| 23 | HER2 SHAP analysis — dependence + violin | fig_18, fig_19 |
| 24 | Save SHAP values | shap_values.csv |
| 24b | SHAP interaction values (14×14) | fig_20, fig_21 |
| 25 | 5-fold CV + bootstrap CIs | cv_results.csv |
| 26 | Gene selection (mutual information) | gene_mi_scores.csv |
| 27 | Combine clinical + gene features | — |
| 28 | XGBoost combined model | combined_model.pkl |
| 29 | Three-way ROC comparison | fig_11 |
| 30 | Kaplan-Meier survival curves | fig_22, fig_23 |
| 31 | Final results summary | final_results_summary.csv |

---

## Results

### Model Comparison

| Model | AUC | Sensitivity | Specificity | PPV | NPV | FN |
|-------|-----|-------------|-------------|-----|-----|----|
| Logistic Regression | 0.712 | 0.675 | 0.615 | 0.560 | 0.723 | 52 |
| XGBoost Clinical (14) | 0.725 | 0.688 | 0.652 | 0.588 | 0.742 | 50 |
| XGBoost Combined (34) | 0.725 | 0.619 | 0.719 | 0.615 | 0.723 | 61 |
| NPI Baseline | ~0.630 | — | — | — | — | — |

### Cross-Validation (XGBoost Clinical, 5-fold Stratified)

| Fold | AUC | Sensitivity | Specificity | FN |
|------|-----|-------------|-------------|----|
| 1 | 0.749 | 0.641 | 0.701 | 46 |
| 2 | 0.715 | 0.664 | 0.638 | 43 |
| 3 | 0.749 | 0.690 | 0.688 | 40 |
| 4 | 0.744 | 0.680 | 0.636 | 41 |
| 5 | 0.715 | 0.602 | 0.688 | 51 |
| **Mean** | **0.734** | **0.655** | **0.670** | **44.2** |
| **SD** | **0.016** | **0.032** | **0.027** | — |

**Bootstrap 95% CIs (n=1,000, OOF predictions)**

| Metric | Estimate | 95% CI |
|--------|----------|--------|
| AUC | 0.733 | 0.707 – 0.757 |
| Sensitivity | 0.656 | 0.620 – 0.690 |
| Specificity | 0.670 | 0.639 – 0.700 |

### SHAP Feature Importance

| Rank | Feature | Mean \|SHAP\| |
|------|---------|--------------|
| 1 | Age At Diagnosis | 0.713 |
| 2 | Nottingham Prognostic Index | 0.248 |
| 3 | Lymph Nodes Examined Positive | 0.206 |
| 4 | Type Of Breast Surgery | 0.186 |
| 5 | Tumor Size | 0.166 |
| 6 | Mutation Count | 0.154 |
| 7 | Hormone Therapy | 0.116 |
| 8 | HER2 Status | 0.079 |
| 9 | Cancer Type Detailed | 0.075 |
| 10 | Radio Therapy | 0.074 |

### SHAP Interaction Values (Top 5)

| Rank | Feature 1 | Feature 2 | Mean \|Interaction\| |
|------|-----------|-----------|---------------------|
| 1 | NPI | Tumor Size | 0.171 |
| 2 | Age | Tumor Size | 0.134 |
| 3 | Mutations | Tumor Size | 0.086 |
| 4 | Hormone | NPI | 0.068 |
| 5 | Age | Surgery | 0.065 |

### HER2 Paradox

Despite HER2-positive status being associated with aggressive tumour
biology, the model assigned HER2-positive patients a mean SHAP value of
−0.281, indicating survival benefit relative to HER2-negative patients
(+0.044). This reflects the confounding effect of Herceptin (trastuzumab)
treatment in the METABRIC cohort — the model learned the net treatment
outcome without explicit treatment labels.

| Group | n | Mean SHAP | Interpretation |
|-------|---|-----------|----------------|
| HER2-positive | 56 | −0.281 | Survival benefit |
| HER2-negative | 325 | +0.044 | Mortality risk |
| Difference | — | −0.326 | — |
| KM log-rank p | — | 0.0093 | Significant |

### Case Study — Patient 234 (False Negative)

| Property | Value |
|----------|-------|
| Predicted survival probability | 26.4% |
| Predicted mortality risk | 73.6% |
| Actual outcome | Died |
| Classification | False negative — model correctly flagged as high risk |

### Kaplan-Meier Clinical Validation

| Analysis | Log-rank p | Interpretation |
|----------|-----------|----------------|
| HER2+ vs HER2− | 0.0093 | Significant survival difference |
| Model risk groups (High vs Low) | < 0.0001 | Strong risk stratification |
| High mortality risk (n=147) @250m | ~5% survival | — |
| Low mortality risk (n=137) @250m | ~43% survival | — |
| Intermediate risk (n=97) | between groups | — |

---

## Reproducibility

```python
RANDOM_STATE = 42
SPLIT        = 0.80        # stratified
CV_FOLDS     = 5
BOOTSTRAP_N  = 1000
THRESHOLD    = 0.20        # missing value drop threshold (%)
TOP_GENES    = 20          # mutual information gene selection
```

| Component | Detail |
|-----------|--------|
| Imbalance handling | `scale_pos_weight=1.376` (XGBoost) |
| Imbalance handling | `class_weight=balanced` (LR) |
| Scaling | StandardScaler fit on train only |
| XGBoost input | Raw unscaled features (trees do not require scaling) |
| LR input | StandardScaler scaled features |
| SHAP explainer | TreeExplainer on XGBoost clinical model |
| SHAP patients | All 381 test patients |
| Interaction shape | (381, 14, 14) |
| Gene selection | `mutual_info_classif` on 489 candidate genes |
| Risk groups | pred_survival < 0.4 = High, > 0.6 = Low, else Intermediate |

---

## Installation

```bash
git clone https://github.com/heraclitus0/breast-cancer-prognosis-explainable-ml.git
cd breast-cancer-prognosis-explainable-ml
pip install -r requirements.txt
```

Open the notebook in Jupyter or Google Colab and run all cells
sequentially from Cell 1 to Cell 31. The dataset must be present
at `/content/METABRIC_RNA_Mutation.csv` before running.

---

## Data

The METABRIC dataset is not included in this repository due to size.
Download `METABRIC_RNA_Mutation.csv` from:

- **Kaggle:** [raghadalharbi/breast-cancer-gene-expression-profiles-metabric](https://www.kaggle.com/datasets/raghadalharbi/breast-cancer-gene-expression-profiles-metabric)
- **cBioPortal:** [cbioportal.org](https://www.cbioportal.org)

Place the file in the root directory before running the notebook.  
License: CC BY 4.0.

---

## Citation

If you use this code or findings in your research, please cite:

```bibtex
@software{sashi2025metabric,
  author  = {Pulikanti, Sashi Bharadwaj},
  title   = {{Clinical-Genomic Predictive Modeling for Breast Cancer
             Prognosis: An Explainable Machine Learning Approach
             Using the METABRIC Cohort}},
  year    = {2025},
  orcid   = {0009-0002-8904-2209},
  url     = {https://github.com/heraclitus0/breast-cancer-prognosis-explainable-ml},
  version = {1.0.0},
  license = {MIT}
}
```

**Dataset citations:**

```bibtex
@article{curtis2012genomic,
  title   = {The genomic and transcriptomic architecture of 2,000 breast
             tumours reveals novel subgroups},
  author  = {Curtis, Christina and others},
  journal = {Nature},
  volume  = {486},
  pages   = {346--352},
  year    = {2012},
  doi     = {10.1038/nature10983}
}

@article{pereira2016somatic,
  title   = {The somatic mutation profiles of 2,433 breast cancers refine
             their genomic and transcriptomic landscapes},
  author  = {Pereira, Bernard and others},
  journal = {Nature Communications},
  volume  = {7},
  pages   = {11479},
  year    = {2016},
  doi     = {10.1038/ncomms11479}
}
```

---

## License

This project is licensed under the MIT License.  
The METABRIC dataset is available under CC BY 4.0 via cBioPortal and Kaggle.

---

## Acknowledgements

The METABRIC study was funded by Cancer Research UK,
the British Columbia Cancer Agency, and the Wellcome Trust.

---

<p align="center">
  <sub>Built with Python · XGBoost · SHAP · lifelines · scikit-learn</sub>
</p>
```
