# CBC Blood Report Data Preparation & Analysis

**Data Cleaning and Exploratory Data Analysis for Complete Blood Count (CBC) Diagnostic Dataset**

Part of the **AI-Based Multi-Agent Diagnostic Assistant** capstone project.

---

## Overview

This repository contains a comprehensive data preparation and exploratory data analysis pipeline for Complete Blood Count (CBC) laboratory reports. The goal is to transform raw, messy hematology data into a clean, validated dataset suitable for downstream disease diagnostic modeling.

**Key outputs:**
- Cleaned CBC dataset: 1,193 rows × 13 medical features + diagnosis labels
- 9 diagnosis classes: Healthy, Iron deficiency anemia, Normocytic anemia (3 subtypes), Macrocytic anemia, Leukemia, Thrombocytopenia, and Leukemia with thrombocytopenia
- Full data quality audit with documented cleaning decisions
- Exploratory visualizations: class distribution, feature histograms, per-class boxplots, correlation analysis, pairplots

---

## Dataset

### Source
- **Original dataset:** Complete Blood Count (CBC) results from laboratory tests
- **Size:** 1,281 rows (before cleaning) → **1,193 rows** (after quality assurance)
- **Features:** 15 columns (14 numeric CBC parameters + 1 diagnosis label)

### Features (13 diagnostic parameters retained)

| Parameter | Unit | Clinical Significance |
|---|---|---|
| **WBC** | thousands/μL | White blood cell count; elevated in infections, leukemia |
| **RBC** | millions/μL | Red blood cell count; low in anemia |
| **HGB** | g/dL | Hemoglobin; oxygen-carrying capacity; diagnostic for anemia |
| **HCT** | % | Hematocrit; volume fraction of RBCs |
| **MCV** | fL | Mean corpuscular volume; classifies anemia subtype (microcytic, normocytic, macrocytic) |
| **MCH** | pg | Mean corpuscular hemoglobin per cell |
| **MCHC** | g/dL | Mean corpuscular hemoglobin concentration |
| **PLT** | thousands/μL | Platelet count; low in thrombocytopenia, elevated in clotting disorders |
| **PDW** | % | Platelet distribution width; platelet morphology variability |
| **LYMn** | thousands/μL | Absolute lymphocyte count |
| **LYMp** | % | Lymphocyte percentage (derived) |
| **NEUTn** | thousands/μL | Absolute neutrophil count |
| **NEUTp** | % | Neutrophil percentage (derived) |

### Target Classes (9 diagnosis labels)
1. Healthy (n=336)
2. Normocytic hypochromic anemia (n=255)
3. Iron deficiency anemia (n=175)
4. Normocytic normochromic anemia (n=252)
5. Other microcytic anemia (n=50)
6. Macrocytic anemia (n=14)
7. Thrombocytopenia (n=69)
8. Leukemia (n=43)
9. Leukemia with thrombocytopenia (n=11) — **rare class**

---

## Data Quality Issues Found & Resolved

### Issue 1: Hidden Imputation in WBC Differential Columns
**Problem:** Columns `LYMp` (lymphocyte %), `NEUTp` (neutrophil %), `PDW` (platelet distribution width), and `PCT` (plateletcrit) contained 781 rows (61% of data) with identical repeated values → evidence of mean-value imputation without documentation.

**Solution:** 
- Derived new `LYMp` and `NEUTp` from clean absolute counts: `LYMp = (LYMn / WBC) × 100`
- Dropped original corrupted `LYMp`, `NEUTp`, `PCT` columns
- Retained `PDW` (independent measurement) after validation

### Issue 2: Physically Impossible Values
**Problem:** 
- Negative hemoglobin values (HGB < 0)
- RBC values > 90 million/μL (physiological max ~6.5)
- HGB > 25 g/dL (physiological max ~20)
- Negative MCV values

**Solution:** 
- Converted 34 rows with impossible values to NaN
- Removed those rows entirely (2.7% of data) rather than imputing, preserving data integrity for medical use

### Issue 3: HCT Consistency Anomaly
**Problem:** 476 rows (38% of dataset) showed HCT values consistent with calculated formula `HCT = (RBC × MCV) / 10`, indicating HCT was computed from other measurements rather than independently measured.

**Solution:** 
- Investigated distribution: spike at ratio=1.0 + tail of truly measured values
- Documented this split but retained all rows
- HCT is independently measured by modern analyzers, so mathematical relationship variability is expected and acceptable

### Issue 4: Exact Duplicate Rows
**Problem:** 49 rows were exact duplicates

**Solution:** Removed all 49 duplicates

### Issue 5: Class Imbalance
**Problem:** Extreme imbalance — Healthy (336 samples) vs. Leukemia with thrombocytopenia (11 samples)

**Solution:** Documented for downstream modeling; will require SMOTE oversampling during ML training, not during data cleaning

---

## Data Cleaning Pipeline

### Step 1: Load & Validate Structure
- Load raw CSV (1,281 rows × 15 columns)
- Check data types, null values, column structure

### Step 2: Feature Engineering (WBC Differential)
- Derive `LYMp = (LYMn / WBC) × 100`
- Derive `NEUTp = (NEUTn / WBC) × 100`
- Handle division by zero (WBC = 0) → set to NaN
- Drop original corrupted columns: `LYMp_old`, `NEUTp_old`, `PCT`

### Step 3: Remove Impossible Values
Applied physiological range checks:
- HGB: Keep 5 < HGB < 25 g/dL
- MCV: Keep 50 < MCV < 130 fL
- RBC: Keep 2 < RBC < 8 millions/μL
- HCT: Keep 15 < HCT < 60 %
- PLT: Keep 50 < PLT < 500 thousands/μL
- WBC: Keep 2 < WBC < 45 thousands/μL

Converted violations to NaN (34 rows affected)

### Step 4: Investigate HCT Consistency
- Calculate expected: `HCT_expected = (RBC × MCV) / 10`
- Calculate ratio: `HCT_actual / HCT_expected`
- Identified and documented calculated vs. independently-measured rows
- Decision: retain all rows; data quality acceptable

### Step 5: Remove Duplicates
- Identified and removed 49 exact duplicate rows

### Step 6: Drop Audit Columns
- Removed temporary calculation columns (`HCT_expected`, `HCT_ratio`, `HCT_source`, etc.)
- Kept only the 13 core CBC features + Diagnosis label

### Step 7: Remove NaN Rows
- Removed 34 rows with NaN in core CBC parameters
- **Final dataset: 1,193 rows × 14 columns**

---

## Exploratory Data Analysis (EDA)

### Visualizations Generated

#### 1. **Pairplot: Key Features by Diagnosis Class**
![CBC Pairplot](visualizations/pairplot_cbc.png)

*4-feature scatter matrix (HGB, MCV, PLT, WBC) showing clear visual separation of diagnosis classes. Each point represents one patient, colored by diagnosis.*

**Key observations:**
- Iron deficiency anemia clusters bottom-left (low HGB, low MCV)
- Healthy patients cluster top-middle (high HGB, normal MCV)
- Thrombocytopenia shows very low PLT (bottom-left of PLT plots)
- Leukemia shows high WBC (right side of WBC plots)

---

#### 2. **Feature Relationships: Four Critical Scatter Plots**
![Feature Relationships](visualizations/feature_relationships.png)

*Shows diagnostic power of key feature combinations:*
- **HGB vs MCV:** Anemia subtype classification (iron deficiency, macrocytic, normocytic)
- **HGB vs PLT:** Discriminates between anemia and thrombocytopenia
- **WBC vs MCV:** Leukemia detection by elevated WBC
- **WBC vs PLT:** Leukemia with thrombocytopenia (high WBC + low PLT)

---

#### 3. **Correlation Heatmap: Feature Dependencies**
![Correlation Heatmap](visualizations/correlation_heatmap.png)

*Shows relationships between all 13 CBC parameters:*

**Strongest correlations (expected):**
- RBC ↔ HGB: 0.72 (both measure oxygen-carrying capacity)
- HGB ↔ HCT: 0.60 (hematocrit derived from hemoglobin)
- LYMn ↔ LYMp: 0.58 (derived from same raw data)
- NEUTn ↔ NEUTp: 0.72 (derived from same raw data)

**Negative correlations (clinically meaningful):**
- WBC ↔ LYMp: -0.49 (leukemia patients show neutrophil shift, lower lymphocyte %)

**Conclusion:** No problematic multicollinearity; most features are independent and capture different diagnostic aspects.

---

#### 4. **Class Distribution Bar Chart**
- Shows count of each diagnosis class
- Highlights class imbalance (Leukemia with thrombocytopenia: only 11 samples)

#### 5. **Feature Histograms (per-feature distributions)**
- All 13 features visualized
- Confirms absence of negative/impossible values post-cleaning
- Shows expected multimodal distributions for anemia subtypes

#### 6. **Boxplots by Diagnosis Class (critical for validation)**
- Each of 13 features plotted across all 9 diagnosis classes
- Confirms features have real diagnostic signal across all classes

---

## Project Structure

```
cbc-data-analysis/
├── README.md                              (this file)
├── DATA_QUALITY_AUDIT.md                  (detailed cleaning decisions)
├── NOTEBOOKS_README.md                    (how to run each notebook)
├── requirements.txt                       (Python dependencies)
├── .gitignore                             (Git configuration)
├── data/
│   ├── raw/
│   │   └── diagnosed_cbc_data_v4.csv      (original, unmodified)
│   └── processed/
│       └── cbc_cleaned_final.csv          (cleaned, ready for ML)
├── notebooks/
│   ├── 1_data_preparation.ipynb           (cleaning pipeline)
│   └── 2_eda_visualization.ipynb          (all visualizations)
└── visualizations/
    ├── pairplot_cbc.png                   (4-feature pairplot by class)
    ├── feature_relationships.png          (scatter plots of key combinations)
    ├── correlation_heatmap.png            (feature interdependencies)
    ├── class_distribution.png             (diagnosis class counts)
    ├── histograms/                        (per-feature distributions)
    └── boxplots_by_diagnosis/             (per-feature by diagnosis class)
```

---

## Key Cleaning Decisions (Data Analyst Perspective)

### Decision 1: Derive vs. Impute WBC Differential
- **Chose:** Derive `LYMp`, `NEUTp` from absolute counts (`LYMn`, `NEUTn`, `WBC`)
- **Rationale:** Original corrupted columns (61% imputed means) had lost signal. Derivation recovers information from clean source columns without adding artificial data.
- **Impact:** Restored 1,270 rows with valid WBC differential information

### Decision 2: Drop vs. Impute Impossible Values
- **Chose:** Drop 34 rows with physically impossible values
- **Rationale:** 
  - Small sample loss (2.7% of dataset)
  - Rare classes preserved (only 1 Leukemia case lost, 0 Leukemia+thrombocytopenia lost)
  - Corruption suggests entire row unreliable, not just one column
  - More defensible than imputation for medical data
- **Impact:** Final dataset of 1,193 samples, all feasible

### Decision 3: Retain HCT Calculated Rows
- **Chose:** Keep all rows despite 38% being mathematically derived
- **Rationale:**
  - Modern hematology analyzers report both measured and calculated values
  - No evidence of corruption; just measurement method documentation
  - Removing would drop 476 samples (40%) with real underlying RBC/MCV data
  - Documented but did not penalize; downstream models handle naturally
- **Impact:** Preserved data volume and signal

### Decision 4: Handle Class Imbalance
- **Chose:** Document and flag, handle downstream during ML (SMOTE in cross-validation pipeline)
- **Rationale:**
  - Data cleaning phase shouldn't artificially rebalance; that's a modeling choice
  - Stratified splitting will ensure rare classes in both train and test
  - SMOTE inside CV folds prevents data leakage
- **Impact:** Clean separation of concerns: data quality (here) vs. learning strategy (ML phase)

---

## Data Quality Metrics (Post-Cleaning)

| Metric | Value |
|---|---|
| Total rows | 1,193 |
| Total features | 13 |
| Missing values | 0 |
| Duplicate rows | 0 |
| Impossible values | 0 |
| Rows with derived features | 1,193 (100%) |
| Rows with measured all others | ~717 (60%) |
| Diagnosis classes | 9 |
| Smallest class size | 11 (Leukemia with thrombocytopenia) |
| Largest class size | 336 (Healthy) |
| Class imbalance ratio | 30.5:1 |

---

## Usage

### Prerequisites
```bash
python 3.8+
pandas
numpy
matplotlib
seaborn
scikit-learn
```

### Install dependencies
```bash
pip install -r requirements.txt
```

### Run data cleaning notebook
```bash
jupyter notebook notebooks/1_data_preparation.ipynb
# Outputs: data/processed/cbc_cleaned_final.csv
```

### Run EDA notebook
```bash
jupyter notebook notebooks/2_eda_visualization.ipynb
# Outputs: visualizations in reports/visualizations/
```

### Load cleaned data for downstream use
```python
import pandas as pd

df_clean = pd.read_csv('data/processed/cbc_cleaned_final.csv')

# Features (13)
features = ['WBC', 'RBC', 'HGB', 'HCT', 'MCV', 'MCH', 'MCHC', 'PLT', 'PDW', 'LYMn', 'LYMp', 'NEUTn', 'NEUTp']
X = df_clean[features]

# Target (9 classes)
y = df_clean['Diagnosis']

print(f"Shape: {X.shape}")
print(f"Classes: {y.unique()}")
```

---

## Downstream Application

**This cleaned dataset feeds into:** Blood abnormality classification model (XGBoost + SMOTE pipeline)

**Expected usage:**
- Input: Raw CBC lab report (13 parameters)
- Processing: Apply same cleaning transformations (derive LYMp/NEUTp, validate ranges)
- Output: Diagnosis prediction + confidence + SHAP explainability

---

## Limitations & Caveats

1. **Single-source data:** All samples from one lab; lab-specific measurement protocols, equipment, and reference ranges may differ from other facilities
2. **Rule-derived labels:** Diagnosis classes likely derived from CBC cutoff thresholds; real-world noisier data expected to show overlap
3. **Rare class imbalance:** Leukemia (44 samples) and Leukemia with thrombocytopenia (11 samples) are statistically underpowered for robust model evaluation
4. **No longitudinal data:** Single snapshot per patient; no time-series or follow-up information
5. **No clinical outcomes:** Diagnosis labels not validated against actual patient outcomes; labels may represent lab-based classification, not confirmed clinical diagnosis

---

## Data Quality Report

For detailed explanations of every cleaning decision, see:
**[reports/data_quality_audit.md](reports/data_quality_audit.md)**

---

## Author

**Manas Dhotre**  
Data Analyst | CSE Department  
Brahmdevdada Mane Institute of Technology (BMIT), Solapur  
A.Y. 2026-27

**Supervised by:** Prof. S.A. Chabukswar

---

## License

This dataset and analysis are part of an academic capstone project. Use for educational and research purposes only.

---

## Acknowledgments

- Collaborators: Sangram Patil, Onkar Hiremath, Balu Nandiwale
- Guidance: Prof. S.A. Chabukswar
- Part of: AI-Based Multi-Agent Diagnostic Assistant capstone project

---

## Questions?

For questions about data preparation, cleaning decisions, or EDA:
- Review [notebooks/README_notebooks.md](notebooks/README_notebooks.md) for notebook-specific details
- Check [reports/data_quality_audit.md](reports/data_quality_audit.md) for decision justifications
- Contact: [your email or GitHub issues]
