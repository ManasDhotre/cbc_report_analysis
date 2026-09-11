# CBC Dataset Analysis & Data Cleaning

## Overview

This project focuses on exploring, validating, cleaning, and preparing a
Complete Blood Count (CBC) dataset for downstream machine-learning model
training.

The notebook works with CBC laboratory features and a `Diagnosis`
target. The workflow emphasizes data quality before modeling:
identifying redundant features, reconstructing derived values, detecting
physiologically implausible values, checking internal consistency
between CBC measurements, removing duplicates and incomplete records,
exploring class distribution, visualizing the cleaned data, scaling
selected features, and exporting a final modeling-ready dataset.

> **Note:** This README describes what is implemented in the provided
> notebook. The notebook performs data preparation and exploratory
> analysis; it does not train or evaluate a diagnostic machine-learning
> model.

## Dataset

The original dataset contains **1,281 rows and 15 columns**:

-   14 numerical CBC-related features
-   1 categorical target: `Diagnosis`

The target contains the following diagnostic classes:

  Diagnosis                          Initial Count   Initial %
  -------------------------------- --------------- -----------
  Healthy                                      323      26.32%
  Normocytic hypochromic anemia                266      21.68%
  Normocytic normochromic anemia               255      20.78%
  Iron deficiency anemia                       184      15.00%
  Thrombocytopenia                              72       5.87%
  Other microcytic anemia                       56       4.56%
  Leukemia                                      44       3.59%
  Macrocytic anemia                             16       1.30%
  Leukemia with thrombocytopenia                11       0.90%

The notebook also identifies suspicious repeated values in several
percentage/count-derived fields, which motivated feature reconstruction
and cleaning.

## Features

The main CBC variables used in the analysis are:

-   `WBC`
-   `LYMn`
-   `LYMp`
-   `NEUTn`
-   `NEUTp`
-   `RBC`
-   `HGB`
-   `HCT`
-   `MCV`
-   `MCH`
-   `MCHC`
-   `PLT`
-   `PDW`

Target:

-   `Diagnosis`

## Workflow

The analysis follows this pipeline:

``` text
Raw CBC Data
     |
     v
Initial inspection
     |
     v
Identify redundant / suspicious fields
     |
     v
Recalculate LYMp and NEUTp
     |
     v
Physiological range validation
     |
     v
Convert implausible values to NaN
     |
     v
Remove duplicate records
     |
     v
Validate HCT consistency
     |
     v
Check NaN impact by diagnosis
     |
     v
Remove incomplete core CBC records
     |
     v
EDA and visualization
     |
     v
Feature scaling for visualization
     |
     v
Final validation
     |
     v
Export cleaned CSV
```

## Problems Faced and How They Were Addressed

### 1. Redundant and suspicious feature values

Some columns contained repeated values that did not provide reliable
independent information. For example, `LYMp`, `NEUTp`, and `PCT` showed
highly repeated values, with `LYMp` containing the same value 781 times
and `NEUTp` containing the same value 781 times.

**Approach:**

-   Removed `PCT`, `LYMp`, and `NEUTp` initially.
-   Kept the absolute-count variables `LYMn` and `NEUTn`.
-   Reconstructed the percentage variables from their underlying counts
    and `WBC`.

The notebook calculates:

``` python
LYMp = (LYMn / WBC) * 100
NEUTp = (NEUTn / WBC) * 100
```

This preserves the relationship between the absolute and percentage
measurements rather than trusting suspicious repeated values.

### 2. Physiologically implausible values

The initial descriptive statistics revealed extreme values, including
negative or unusually large measurements. Examples included:

-   `HGB` minimum of `-10`
-   `MCV` minimum of `-79.3`
-   `NEUTp` maximum of `5317`
-   `HCT` maximum of `3715`
-   `RBC` maximum of `90.8`

These values could distort exploratory analysis and any model trained on
the data.

**Approach:**

The notebook defined explicit validation ranges for key CBC variables:

``` python
HGB: 5–25
RBC: 2–8
MCV: 50–130
HCT: 15–60
PLT: 30–700
WBC: 2–30
```

Values outside these ranges were treated as invalid and converted to
`NaN` instead of being silently retained.

The initial invalid-value counts were:

  Feature     Invalid Values
  --------- ----------------
  HGB                     13
  RBC                     11
  MCV                      8
  HCT                      9
  PLT                      9
  WBC                      5

This affected 34 rows before duplicate removal.

### 3. Duplicate records

After invalid values were marked as missing, the dataset contained **54
duplicate rows**.

**Approach:**

The notebook used:

``` python
df_copy = df_copy.drop_duplicates().copy()
```

This reduced the dataset from 1,281 rows to **1,227 rows**, with zero
duplicates remaining.

### 4. Internal consistency of HCT

A particularly important data-quality problem was whether `HCT` was
consistent with `RBC` and `MCV`.

The notebook used the relationship:

``` text
HCT ≈ (RBC × MCV) / 10
```

An expected HCT was calculated and compared with the recorded HCT.
Relative error was then evaluated.

A 10% threshold initially flagged a substantial number of observations.
The notebook also tested several thresholds:

  Relative-error threshold     Rows flagged
  -------------------------- --------------
  5%                                    623
  10%                                   516
  15%                                   377
  20%                                   304

This investigation showed that the discrepancy was widespread rather
than limited to a handful of obvious errors.

**Decision:**

The notebook did not automatically replace all measured HCT values with
calculated values. Instead, it investigated the ratio between actual and
calculated HCT and retained the available HCT information rather than
aggressively overwriting it.

This is an important data-engineering choice: a consistency check can
identify questionable records without automatically assuming that the
derived value is always superior to the recorded measurement.

### 5. Missing values affecting minority classes

After invalid measurements were converted to `NaN`, the notebook checked
whether missingness disproportionately affected particular diagnostic
classes.

The affected records included:

-   Iron deficiency anemia: 9
-   Normocytic hypochromic anemia: 9
-   Other microcytic anemia: 6
-   Normocytic normochromic anemia: 3
-   Thrombocytopenia: 3
-   Macrocytic anemia: 2
-   Leukemia: 1

The percentage impact was also inspected. For example,
`Macrocytic anemia` had 2 affected records out of 16 (12.5%), while
`Other microcytic anemia` had 6 out of 56 (10.7%).

**Approach:**

Rather than deleting rows immediately, the notebook first reported the
impact by diagnosis. After this check, rows containing missing values in
the core CBC parameters were removed.

This resulted in a final cleaned dataset of **1,194 rows**.

### 6. Class imbalance

The cleaned dataset remained imbalanced. The largest class was `Healthy`
with 323 records, while `Leukemia with thrombocytopenia` had only 11
records.

**Approach:**

The notebook explicitly measured class counts and percentages and
visualized the distribution.

The final class distribution was:

  Diagnosis                          Final Count
  -------------------------------- -------------
  Healthy                                    323
  Normocytic hypochromic anemia              257
  Normocytic normochromic anemia             252
  Iron deficiency anemia                     175
  Thrombocytopenia                            69
  Other microcytic anemia                     50
  Leukemia                                    43
  Macrocytic anemia                           14
  Leukemia with thrombocytopenia              11

The imbalance is documented rather than artificially corrected in this
notebook.

### 7. Different feature scales

CBC variables have very different numerical scales. For example,
platelet count and hemoglobin values naturally operate on very different
ranges.

**Approach:**

`StandardScaler` was applied to:

-   `HGB`
-   `MCV`
-   `PLT`
-   `WBC`

The scaled values were then used for an additional pairplot to make
feature relationships easier to visually compare.

## Exploratory Data Analysis

The notebook performs several forms of EDA:

### Distribution analysis

Histograms are generated for the numerical CBC features to inspect their
distributions.

### Diagnosis-level comparison

Boxplots compare each numerical CBC feature across the diagnostic
classes.

### Correlation analysis

A correlation heatmap is generated for the numerical features to inspect
linear relationships and potential redundancy.

### Pairwise relationships

Pairplots are generated for selected features:

-   `HGB`
-   `MCV`
-   `PLT`
-   `WBC`

The diagnostic class is used as the hue to visually inspect class
separation.

## Final Dataset

After cleaning:

-   **Rows:** 1,194
-   **Columns:** 14
-   **Missing values:** None
-   **Duplicate rows:** None
-   **Target:** `Diagnosis`

The final columns are:

``` text
WBC
LYMn
NEUTn
RBC
HGB
HCT
MCV
MCH
MCHC
PLT
PDW
Diagnosis
LYMp
NEUTp
```

The notebook verifies that the exported CSV can be reloaded and that its
shape matches the cleaned dataframe:

``` text
Loaded: (1194, 14), matches original: True
```

## Technologies Used

-   Python
-   Pandas
-   NumPy
-   Matplotlib
-   Seaborn
-   Scikit-learn

Key techniques:

-   Data inspection
-   Descriptive statistics
-   Feature reconstruction
-   Rule-based data validation
-   Missing-value handling
-   Duplicate removal
-   Consistency checks
-   Exploratory data analysis
-   Correlation analysis
-   Feature scaling
-   CSV export

## How to Run

### 1. Install dependencies

``` bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

### 2. Prepare the dataset

Place the source CSV in an accessible location and update the path in
the notebook:

``` python
df = pd.read_csv("path/to/diagnosed_cbc_data_v4.csv")
```

The provided notebook currently contains a local Windows path, so this
should be changed when running on another machine.

### 3. Open the notebook

``` bash
jupyter notebook cbc_analysis.ipynb
```

or use JupyterLab / Google Colab.

### 4. Run the notebook

Execute the cells sequentially. The notebook performs cleaning,
validation, visualization, scaling, and final export.

### 5. Output

The cleaned dataset is exported as:

``` text
cbc_cleaned_dataset_final.csv
```

## Key Learning Outcomes

This project demonstrates that data preprocessing is more than simply
removing null values.

The major lessons from the workflow are:

1.  **Inspect the raw data before modeling.** Extreme values can reveal
    data corruption or extraction problems.
2.  **Use domain relationships as validation rules.** The HCT/RBC/MCV
    relationship provides a useful internal consistency check.
3.  **Do not blindly trust derived columns.** Reconstructing percentage
    features from their underlying counts can be more defensible when
    repeated or suspicious values are present.
4.  **Measure the impact of cleaning decisions.** Checking missingness
    by diagnosis helps reveal whether cleaning disproportionately
    affects minority classes.
5.  **Document class imbalance before model training.** A model can
    appear strong while performing poorly on rare classes.
6.  **Separate data cleaning from modeling.** This notebook prepares a
    clean dataset but does not claim that the resulting data is
    clinically validated or that it is ready for clinical deployment.

## Limitations and Next Steps

The notebook is a data-cleaning and EDA stage rather than a complete
diagnostic ML system.

Potential next steps include:

-   Train baseline classification models.
-   Use stratified train/validation/test splits.
-   Address class imbalance using appropriate training techniques.
-   Evaluate precision, recall, F1-score, confusion matrices, and
    per-class performance.
-   Investigate whether calculated and measured HCT values should be
    treated differently.
-   Validate the selected physiological ranges against authoritative
    clinical reference ranges and the population represented by the
    dataset.
-   Perform feature selection and compare model performance with and
    without derived percentage features.
-   Use cross-validation and hyperparameter tuning.
-   Assess model calibration and error patterns, especially for rare
    diagnoses.
-   Add an independent validation dataset before making claims about
    generalization.

## Project Structure

A recommended repository structure is:

``` text
cbc-analysis/
│
├── cbc_analysis.ipynb
├── data  
├── README.md
└── requirements.txt
```

## Disclaimer

This project is an educational/data-science workflow for CBC dataset
analysis and preparation. The diagnostic labels and preprocessing
decisions in the dataset should not be interpreted as medical advice or
as a clinically validated diagnostic system. Any clinical application
would require appropriate medical validation, governance, and
independent evaluation.
