# Thesis Supporting Code — TIR Analysis

## Project overview

This repository contains the Jupyter notebook used for the thesis analysis of **Time in Range (TIR)** across three insulin-delivery treatment groups:

- **MDI** — Multiple Daily Injections
- **Pump** — Insulin pump therapy
- **HCL** — Hybrid Closed Loop therapy

The notebook implements an end-to-end workflow covering data import, cleaning and reshaping, descriptive analysis, causal modelling with **DoWhy**, treatment-effect estimation with linear regression, **inverse probability weighting (IPW)** using a multinomial propensity-score model, and a **Random Forest propensity model** as an alternative sensitivity analysis.

The analysis is intended as a methodological demonstration using observational clinical data. The causal estimates depend on the stated causal assumptions and the measured variables available in the dataset; they should not be interpreted as definitive clinical evidence.

---

## Files required

For the notebook to run, place the following files in the **same working directory**:

```text
thesis_supporting_material/
├── eesha_khawaja_thesis_code.ipynb
├── filled_TIR_dataset.xlsx
├── README.md
└── requirements.txt        
```

The uploaded notebook is currently named:

```text
eesha_khawaja_thesis_code.ipynb
```


The input workbook filename is hard-coded in the notebook and therefore **must be**:

```text
filled_TIR_dataset.xlsx
```

The notebook reads:

```python
pd.read_excel(
    "filled_TIR_dataset.xlsx",
    sheet_name="Sheet4",
    skiprows=1
)
```

If the workbook filename, worksheet name, row positions, or column layout are changed, the import and cleaning code may also need to be updated.

---

## Input dataset structure

The workbook is expected to contain patient characteristics and repeated TIR measurements in the same worksheet.

The patient-characteristic section contains the following variables:

```text
Patient
MDI/Pump/HCL
High risk/ low risk
Social deprivation
age
ethnicity
```

The notebook treats the first five characteristic rows as patient metadata and the subsequent rows as date-based TIR measurements.

The current notebook expects:

- five patient-characteristic rows;
- approximately 33 date-based TIR rows;
- treatment labels corresponding to MDI, Pump and HCL;
- TIR values that can be converted to numeric values;
- age and social deprivation values that can be converted to numeric values.

The initial worksheet also contains four unused columns named:

```text
Unnamed: 0
Unnamed: 1
Unnamed: 2
Unnamed: 3
```

These are dropped during import.

### Treatment changes and patient identifiers

The notebook does **not** create treatment-change identifiers itself. If the source workbook already contains identifiers such as `11_A` and `11_B`, these are read as separate patient-treatment records and are analysed separately.

### Synthetic or pre-filled covariate values

The notebook does **not** generate synthetic age, ethnicity, deprivation, risk, or treatment values. If missing covariates were filled before the notebook was run, those values are already part of the input workbook and are treated as ordinary input values by the code.

---

## Software environment

The notebook metadata records:

```text
Python 3.13.5
Kernel: Python 3
```

The notebook directly imports or relies on:

```text
pandas
numpy
matplotlib
seaborn
scikit-learn
ipywidgets
dowhy
openpyxl
```

Graph rendering through DoWhy may also require Graphviz support in the local environment.

A typical installation command is:

```bash
python -m pip install pandas numpy matplotlib seaborn scikit-learn ipywidgets dowhy openpyxl graphviz jupyter
```

Package versions are **not embedded in the notebook metadata**. For exact reproducibility, the final environment used to generate the submitted outputs should be frozen after a successful clean run:

```bash
python -m pip freeze > requirements.txt
```

The resulting `requirements.txt` should be included with the supporting material if possible.

---

## Recommended setup

Create and activate a dedicated virtual environment before installing the dependencies.

### macOS / Linux

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install pandas numpy matplotlib seaborn scikit-learn ipywidgets dowhy openpyxl graphviz jupyter
```

### Windows

```powershell
python -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
python -m pip install pandas numpy matplotlib seaborn scikit-learn ipywidgets dowhy openpyxl graphviz jupyter
```

Then start Jupyter:

```bash
jupyter notebook
```

or:

```bash
jupyter lab
```

---

## How to run the analysis

For the final reproducible run:

1. Put `eesha_khawaja_thesis_code.ipynb` and `filled_TIR_dataset.xlsx` in the same folder.
2. Open the notebook in Jupyter Notebook or JupyterLab.
3. Confirm that the active kernel is the intended Python environment.
4. Restart the kernel.
5. Run **all cells from top to bottom** without skipping cells.
6. Confirm that no cell ends with an error.
7. Confirm that the main numerical checkpoints described below are reproduced.
8. Save the executed notebook.
9. Export a PDF or HTML copy only after the clean run if a human-readable code export is required.
10. Freeze the successful environment to `requirements.txt`.

Running from a fresh kernel is important because later cells depend on objects created earlier in the notebook.

---

# Analysis workflow

## 1. Setup

The notebook imports the Python libraries required for:

- data manipulation;
- numerical calculations;
- plotting;
- causal inference;
- propensity-score estimation;
- weighted regression;
- Random Forest modelling;
- interactive propensity-score inspection.

The main imported classes include:

```python
LogisticRegression
LinearRegression
OneHotEncoder
RandomForestClassifier
CausalModel
LinearRegressionEstimator
```

---

## 2. Data import

The workbook is loaded from `Sheet4`.

The notebook removes the four unused `Unnamed` columns and divides the worksheet into:

- `df_patient_char` — patient characteristics;
- `df_patient_vals` — repeated date-based TIR values.

The current notebook identifies 33 TIR/date rows before reshaping.

---

## 3. Data cleaning and reshaping

### Patient characteristics

Completely empty patient-characteristic columns are removed.

The retained characteristics are:

```text
treatment
risk
social deprivation
age
ethnicity
```

### TIR values

The TIR section is inspected for text values.

Text entries in the TIR measurement section are treated as non-numeric/missing information and are replaced with `NaN`.

Patient columns containing no usable TIR information are removed.

The code may display pandas `FutureWarning` messages during this cleaning step. These are warnings rather than stored execution errors in the current notebook. If package versions are changed, the cleaning code should be rechecked before submission.

### Reshaping

The cleaned patient-characteristic and TIR tables are recombined.

The repeated TIR table is converted from wide to long format with `pandas.melt`.

The resulting `final_df` contains one patient-date TIR record per row together with the corresponding patient characteristics.

### Entry dates

For each patient-treatment record, the notebook derives:

```text
first_entry_date
last_entry_date
```

using the first and last non-missing TIR dates.

---

## 4. Descriptive analysis

The descriptive section is intentionally separate from the causal analysis.

### 4.1 Distribution of TIR readings

Overlapping histograms compare observed TIR readings for:

```text
MDI
Pump
HCL
```

The plotted means are descriptive summaries of observed readings and are not interpreted as causal treatment effects.

### 4.2 Achievement of mean TIR ≥ 70%

The notebook first calculates mean TIR for each patient-treatment record.

It then calculates the percentage of records in each treatment group with:

```text
mean_TIR >= 70
```

The result is visualised using a 100-square graphic for each treatment group.

### 4.3 TIR over time

The notebook converts observation dates to monthly periods and defines a treatment-specific month index.

`month_number = 1` represents the first observed calendar month for the corresponding treatment group.

The notebook calculates, by treatment and month:

```text
average_TIR
total_readings
readings_70_or_above
pct_70_or_above
```

The current saved notebook contains **2,776 usable TIR observations** in the time-trend dataset.

A line plot displays average TIR by month, with a horizontal reference line at 70%.

---

# 5. Causal analysis with DoWhy

## Patient-level causal outcome

For the causal analysis, repeated longitudinal TIR observations are reduced to a patient-treatment-record level outcome:

```text
mean_TIR
```

The notebook also retains:

```text
number_of_TIR_measurements
```

This count is descriptive; it is not used as a precision weight in the causal models.

After the patient-level dataset is constructed, the current analysis contains **144 patient-treatment records**.

Current treatment counts are:

```text
MDI     76
HCL     37
Pump    31
```

---

## Causal graph

The Directed Acyclic Graph (DAG) specifies four measured variables as common causes of treatment and mean TIR:

```text
age
ethnicity
deprivation
risk
```

The graph encoded in the notebook is:

```text
age -> treatment
age -> mean_TIR

ethnicity -> treatment
ethnicity -> mean_TIR

deprivation -> treatment
deprivation -> mean_TIR

risk -> treatment
risk -> mean_TIR

treatment -> mean_TIR
```

DoWhy identifies a backdoor/general-adjustment estimand based on these assumptions.

No instrumental-variable or front-door estimand is identified by the supplied graph.

### Important causal interpretation

The DAG is an **assumed causal structure**.

DoWhy does not prove that age, ethnicity, deprivation and risk are the only confounders. Causal interpretation therefore requires the assumption that the measured adjustment set adequately controls the relevant confounding pathways.

Unmeasured confounding may remain.

---

# 5.4 DoWhy linear-regression treatment effects

The DoWhy regression estimator uses the model:

```text
mean_TIR ~ treatment + deprivation + age + ethnicity + risk
```

Three pairwise treatment contrasts are estimated:

```text
Pump vs MDI
HCL vs MDI
HCL vs Pump
```

The current saved notebook gives the following treatment-effect point estimates:

```text
Pump vs MDI   ≈ 3.970
HCL vs MDI    ≈ 5.391
HCL vs Pump   ≈ 1.421
```

These values are differences in **mean TIR percentage points**, not relative percentage increases.

## Regression bootstrap confidence intervals

The current code requests:

```text
95% confidence level
399 bootstrap simulations
```

through DoWhy's bootstrap confidence-interval functionality.

### Reproducibility note for the regression confidence intervals

The current DoWhy regression-bootstrap section does **not** set an explicit random seed before the 399 bootstrap simulations.

Therefore:

- the regression point estimates should remain stable when the same data and model are used;
- the exact bootstrap confidence-interval endpoints can vary between runs;
- small differences in the regression CI values do not necessarily imply that the fitted regression effect itself changed.

For exact byte-for-byte reproducibility of the regression confidence intervals, the regression bootstrap would need to be run under a deliberately fixed random state and the final outputs regenerated consistently.

The dissertation and submitted code should always report results from the **same final run**.

---

# 6. Inverse Probability Weighting (IPW)

## 6.1 Adjustment set

The IPW analysis obtains the adjustment set directly from the DoWhy identified estimand:

```text
deprivation
age
ethnicity
risk
```

The current analysis confirms zero missing values in these four variables for the 144 records entering the propensity model.

---

## 6.2 Multinomial propensity-score model

Ethnicity and risk are one-hot encoded.

Age and deprivation remain numeric.

A multinomial logistic regression estimates the probability of each patient-treatment record receiving:

```text
HCL
MDI
Pump
```

conditional on the four measured confounders.

For each record, the probability corresponding to the treatment actually observed is selected.

---

## 6.3 Inverse probability weights

The weight is calculated as:

```text
IPW = 1 / P(actual treatment | measured confounders)
```

Current notebook checkpoints:

```text
Lowest observed-treatment propensity ≈ 0.142375
Highest IPW                         ≈ 7.023718
```

The notebook also plots the IPW distribution.

---

## 6.4 Covariate-balance diagnostics

Balance is evaluated using absolute standardised mean differences (SMDs).

For each confounder, the notebook calculates all pairwise treatment-group SMDs and retains the maximum absolute value.

Current results are:

| Confounder | Before IPW | After IPW |
|---|---:|---:|
| Age | 0.328 | 0.077 |
| Deprivation | 0.244 | 0.015 |
| Risk | 0.220 | 0.053 |
| Ethnicity | 0.378 | 0.137 |

The notebook uses **0.10** as a visual reference threshold in the love plot.

Age, deprivation and risk fall below 0.10 after weighting. Ethnicity improves substantially but remains above 0.10, indicating residual measured imbalance.

---

## 6.5 Weighted treatment-effect estimates

Treatment is one-hot encoded with MDI as the reference category.

A weighted linear regression is fitted using the IPW values as sample weights.

Current point estimates are:

```text
Pump vs MDI   = 4.816 percentage points
HCL vs MDI    = 4.864 percentage points
HCL vs Pump   = 0.048 percentage points
```

---

## 6.6 IPW bootstrap confidence intervals

The IPW bootstrap:

- uses **1,000 bootstrap resamples**;
- resamples patient-treatment records with replacement;
- refits the multinomial propensity model inside each bootstrap sample;
- reconstructs the actual-treatment propensity scores;
- recalculates the inverse probability weights;
- refits the weighted outcome regression;
- calculates percentile-based 95% confidence intervals.

The notebook explicitly sets:

```python
np.random.seed(42)
```

before this bootstrap, which makes this section reproducible under the same data and software environment.

Current output:

```text
Pump vs MDI   4.816   95% CI [-3.451, 13.396]
HCL vs MDI    4.864   95% CI [-2.610, 12.056]
HCL vs Pump   0.048   95% CI [-9.406,  8.900]
```

All three intervals include zero.

The results should therefore be interpreted as uncertain rather than as definitive evidence of a treatment difference.

The near-zero HCL-vs-Pump point estimate should not be interpreted as proof of equivalence because its confidence interval is wide.

---

# 7. Random Forest propensity model

The Random Forest analysis is an alternative propensity-score specification using the same measured confounders.

It is used as a sensitivity analysis rather than as a replacement for the main logistic-IPW analysis.

## 7.1 Model

The classifier is fitted with:

```python
RandomForestClassifier(
    n_estimators=100,
    random_state=42
)
```

The Random Forest is therefore explicitly seeded.

Predicted probabilities are generated for all three treatment groups.

The propensity for the treatment actually received is selected for each record.

---

## 7.2 Random Forest inverse weights

The Random Forest weight is calculated as:

```text
RF_IPW = 1 / RF_propensity
```

Current diagnostics are:

```text
Records                 144
Mean RF_IPW             1.668835
Median RF_IPW           1.402712
Minimum RF_IPW          1.010101
Maximum RF_IPW          4.028970
Lowest RF propensity    0.248202
```

---

## 7.3 Random Forest-weighted treatment effects

A weighted linear regression using the Random Forest inverse weights gives the current point estimates:

```text
Pump vs MDI   =  6.456 percentage points
HCL vs MDI    =  6.260 percentage points
HCL vs Pump   = -0.196 percentage points
```

The notebook does not calculate confidence intervals for these Random Forest-weighted estimates.

They should therefore be treated as sensitivity-analysis point estimates only.

A final figure compares the logistic-IPW and Random-Forest-IPW point estimates.

---

# Expected verification checkpoints

After a successful clean run with the supplied dataset, the following are useful checks:

| Check | Current notebook value |
|---|---:|
| Date/TIR rows before reshaping | 33 |
| Usable TIR observations in time-trend analysis | 2,776 |
| Patient-treatment records in causal analysis | 144 |
| MDI records | 76 |
| HCL records | 37 |
| Pump records | 31 |
| Lowest logistic actual-treatment propensity | 0.142375 |
| Highest logistic IPW | 7.023718 |
| IPW Pump vs MDI | 4.816 |
| IPW HCL vs MDI | 4.864 |
| IPW HCL vs Pump | 0.048 |
| Highest Random Forest IPW | 4.028970 |
| RF Pump vs MDI | 6.456 |
| RF HCL vs MDI | 6.260 |
| RF HCL vs Pump | -0.196 |

The DoWhy **regression confidence-interval endpoints are not an exact reproducibility checkpoint under the current code**, because that bootstrap is not explicitly seeded.

---

# Figures produced by the notebook

The notebook displays the following analysis figures inline:

```text
Distribution of TIR Readings by Treatment Type
Achievement of Mean TIR ≥70% by Treatment
Average TIR by Month Since First Observation
DoWhy causal DAG
Estimated Treatment Effects with 95% Confidence Intervals
Individual Treatment Propensity Profile (interactive)
Confounder Balance Before and After IPW
Distribution of Inverse Probability Weights
Sensitivity of IPW Estimates to Propensity-Score Model
```

The interactive propensity-profile plot requires `ipywidgets`.

The DAG call:

```python
causal_model.view_model(file_name="diabetes_causal_dag")
```

may create a graph file in the working directory depending on the DoWhy/Graphviz configuration.

Most other plots are displayed inline and are not explicitly saved to disk by the notebook.

---

# Interpretation of the estimates

All treatment effects in this notebook are expressed as **percentage-point differences in mean TIR**.

For example:

```text
ATE = +4.816
```

means an estimated difference of approximately **4.816 TIR percentage points**, not a 4.816% relative increase.

Positive pairwise estimates favour the first treatment named in the comparison.

An interval containing zero means that the data and model are compatible with both positive and negative treatment differences at the stated confidence level.

---

# Methodological limitations

This analysis has several important limitations that must accompany interpretation of the numerical results.

## Observational treatment allocation

Treatment was not randomised. The causal methods attempt to adjust for measured differences, but causal validity depends on the assumptions represented by the DAG.

## Measured-confounder set

The causal graph adjusts for:

```text
age
ethnicity
social deprivation
risk
```

The methods cannot guarantee that all clinically important treatment-selection variables were observed.

## Residual ethnicity imbalance after IPW

The maximum ethnicity SMD decreases from approximately 0.378 to 0.137, but remains above the 0.10 reference line.

## Patient-level mean TIR

Repeated TIR observations are collapsed into a single mean TIR for each patient-treatment record.

Records with one or a few TIR observations therefore contribute to the causal model in the same basic way as records with many observations. The count of TIR measurements is retained but is not used to weight the regression by measurement precision.

## Treatment-change records

Where one biological patient is represented by separate treatment-period identifiers in the source workbook, the notebook treats those identifiers as separate analytical records.

## Missing TIR information

Text/non-numeric TIR entries are converted to missing values and do not contribute to the TIR calculations.

Records with no usable TIR are removed from the patient-level causal analysis.

## Pre-filled covariates

Any synthetic or manually filled covariate values are external to the notebook. The notebook does not distinguish them from genuinely observed values once they are present in the workbook.

## Random Forest sensitivity analysis

The Random Forest is fitted and used to predict treatment probabilities on the same 144 records.

The Random Forest section does not report bootstrap confidence intervals or its own post-weighting SMD table.

It should therefore be interpreted as an alternative propensity-model sensitivity analysis rather than as independent confirmation of a clinical treatment effect.

---

# Reproducibility summary

The notebook contains three different sources of randomness:

| Analysis | Randomness | Seed in current code? |
|---|---|---|
| DoWhy regression bootstrap | 399 bootstrap simulations | **No explicit seed** |
| Logistic-IPW bootstrap | 1,000 bootstrap resamples | **Yes — `np.random.seed(42)`** |
| Random Forest propensity model | tree construction | **Yes — `random_state=42`** |

As a result, exact IPW and Random Forest outputs should be reproducible under the same data and package versions, while DoWhy regression confidence-interval endpoints can vary between runs.

---

# Troubleshooting

## `FileNotFoundError`

If the notebook cannot find the Excel file, verify that the file is named exactly:

```text
filled_TIR_dataset.xlsx
```

and is in the same working directory as the notebook.

You can check the current working directory with:

```python
import os
print(os.getcwd())
```

## Worksheet not found

The notebook expects:

```text
Sheet4
```

If the workbook uses a different sheet name, either rename the worksheet or update the import cell and rerun the complete notebook.

## Missing Excel engine

If pandas cannot open the `.xlsx` file:

```bash
python -m pip install openpyxl
```

## DAG/Graphviz error

If `causal_model.view_model(...)` fails while the rest of the model works, install Graphviz support.

Python package:

```bash
python -m pip install graphviz
```

A system-level Graphviz installation may also be necessary depending on the operating system.

## `ipywidgets` display issue

If the patient propensity dropdown does not display:

```bash
python -m pip install ipywidgets
```

Restart Jupyter after installation.

## pandas `FutureWarning`

The current notebook can display `FutureWarning` messages during replacement/type handling in the TIR-cleaning section.

Warnings are not the same as execution errors, but the final submission should ideally use the same tested package environment captured in `requirements.txt`.

## scikit-learn version differences

The notebook was executed in a particular scikit-learn environment and some APIs can change between releases.

If a newer scikit-learn version rejects an argument used by the saved notebook, reproduce the original working package environment where possible rather than changing modelling parameters silently.

If code must be changed because of a library update, rerun the complete notebook and update every dependent result in the dissertation/supporting material.

## Different DoWhy regression confidence intervals

If the regression ATE remains approximately:

```text
3.970
5.391
1.421
```

but the bootstrap CI endpoints change, this is expected under the current code because the DoWhy regression bootstrap does not use an explicit seed.

The IPW bootstrap is separately seeded and should not be confused with the DoWhy regression bootstrap.

---

# Final submission checks

Before packaging the supporting material:

- rename the notebook to `eesha_khawaja_thesis_code.ipynb`;
- ensure the dataset is named `filled_TIR_dataset.xlsx`;
- restart the Jupyter kernel and run all cells from top to bottom;
- make sure no cell ends with an execution error;
- check that the dissertation and notebook report results from the same final run;
- ensure treatment labels and treatment comparisons are named correctly;
- keep regression results separate from IPW results;
- verify that IPW estimates and confidence intervals match the final notebook output;
- verify that Random Forest labels are `Pump vs MDI`, `HCL vs MDI`, and `HCL vs Pump`;
- save the executed `.ipynb`;
- include `README.md`;
- generate and include `requirements.txt` from the successful environment where possible;
- optionally include a PDF/HTML notebook export for convenient examiner viewing;
- ensure the supporting dataset contains no personally identifying information and is permitted for submission under the project’s data/ethics arrangements.

A suggested final package is:

```text
Eesha_Khawaja_supporting_material/
├── eesha_khawaja_thesis_code.ipynb
├── eesha_khawaja_thesis_code.pdf      # optional human-readable export
├── filled_TIR_dataset.xlsx
├── README.md
└── requirements.txt
```

---

# Notes for examiners / users of the supporting material

The notebook is designed to be read and executed sequentially.

The main analysis follows this order:

```text
Excel input
    ↓
clean patient characteristics
    ↓
clean longitudinal TIR values
    ↓
reshape to final_df
    ↓
descriptive TIR analysis
    ↓
patient-level mean_TIR
    ↓
DAG + DoWhy identification
    ↓
DoWhy regression
    ↓
multinomial propensity model
    ↓
IPW + balance diagnostics
    ↓
weighted treatment effects + bootstrap CIs
    ↓
Random Forest propensity sensitivity analysis
```

The main inferential conclusion should be based on both the point estimates and their uncertainty, together with the limitations of the observational data and the causal assumptions.

