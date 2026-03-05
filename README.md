# Early Detection of SARS-CoV-2 Variant Dominance and Hospitalization Impact

This repository contains the analysis pipeline used in the study:

**“From Sequences to Strategies: Early Detection of New SARS-CoV-2 Variants via Genetic Distance to Reduce Hospitalizations.”**

The project integrates genomic surveillance, machine learning, and epidemiological modeling to:

* reconstruct SARS-CoV-2 variant chains from spike protein sequences
* identify early signals of **variant dominance** using a **Deep Neural Network (DNN)**
* evaluate the relationship between **genetic distance and hospitalization trends** using **CatBoost regression**

The analysis spans genomic data from six European countries:

* Italy (IT)
* Germany (DE)
* Sweden (SE)
* Denmark (DK)
* France (FR)
* Spain (ES)

Sequence data originate from **GISAID** and are processed using the clustering framework introduced by **de Hoffer et al. (2022)**. 

# Pipeline

Spike sequences (GISAID)
        ↓
Clustering algorithm (de Hoffer et al.)
        ↓
Covid_cluster.csv
        ↓
Chain reconstruction
        ↓
Raw variant chains
        ↓
Double sigmoid fit
        ↓
Chain parameters
        ↓
Synthetic dataset generation
        ↓
DNN training
        ↓
Prediction of dominant variants
        ↓
Denmark analysis
        ↓
CatBoost model for hospitalization drivers

---

# Repository Structure

```
.
├── notebooks
│
│   ├── 1_Chains_reconstruction.ipynb
│   ├── 2_Fit_of_chains.ipynb
│   ├── 3_Calibration_curve.ipynb
│   ├── 4_Dataset_predominant_chains.ipynb
│   ├── 5_Dataset_transient_chains.ipynb
│   ├── 6_DNN_dataset.ipynb
│   ├── 7_DNN_training.ipynb
│   ├── 8_Denmark_data_preparation.ipynb
│   └── 9_CatBoost_Denmark.ipynb
│
├── data
│   ├── 2020_DE/
│   ├── 2020_IT/
│   ├── 2020_SE/
│   ├── 2020_DK/
│   ├── 2020_FR/
│   ├── 2020_SP/
│   │
│   └── Dataset_predominant_chains_realworld.csv
│
├── figures
│
├── environment.yml
│
└── README.md
```

---

# Input Data

The pipeline requires the following input data.

### 1. Clustering Step

The clustering of spike protein sequences is **not implemented in this repository**.

Instead, we rely on the unsupervised clustering algorithm described in:

de Hoffer et al., *Scientific Reports* (2022).

Repository: https://github.com/AdeledeHoffer/ML-Covid

The clustering algorithm from **de Hoffer et al.** produces, for each country, a folder containing:

```
Covid_cluster.csv
```

These folders must be organized as:

```
data/

2020_DE/Covid_cluster.csv
2020_IT/Covid_cluster.csv
2020_SE/Covid_cluster.csv
2020_DK/Covid_cluster.csv
2020_FR/Covid_cluster.csv
2020_ES/Covid_cluster.csv
```

Each `Covid_cluster.csv` contains the spike-sequence clustering results used to reconstruct variant chains.

---

### 2. Real-World Predominant Chains Dataset

The notebook

```
Dataset_predominant_chains.ipynb
```

requires a CSV file containing **real-world predominant chains**, derived from the fitted prevalence curves from real-world data for variants.

Example:

```
data/Dataset_predominant_chains_realworld.csv
```

This dataset contains the fitted parameters used to generate simulated predominant chains for training the deep learning model.

---

# Analysis Pipeline

The workflow is organized as a sequence of Jupyter notebooks.

---

## 1. Chains Reconstruction

```
1_Chains_reconstruction.ipynb
```

Input:

* `Covid_cluster.csv` for each country

Output:

* raw prevalence chains
* plots of **variant prevalence vs time**

These chains represent the temporal dynamics of spike sequence clusters.

---

## 2. Fit of Chains

```
2_Fit_of_chains.ipynb
```

Each chain is fitted using a **double sigmoid function**:

[
P_c(x)=L\frac{1}{1+e^{-(x-b)/a}} - L_2\frac{1}{1+e^{-(x-b_2)/a_2}}
]

This captures:

* the **growth phase** of a variant
* the **decline phase** after replacement by a new variant.

Key parameters extracted:

* `a` – growth rate
* `b` – inflection point
* `L` – maximum prevalence

Early-stage parameters (`a₃`, `a₄`, etc.) are later used for variant classification.

---

## 3. Calibration Curve

```
3_Calibration_curve.ipynb
```

This step evaluates the relationship between:

* sequencing volume
* time required to detect new variant chains.

The calibration follows an inverse power-law model:

[
|t_0| = \frac{A}{\sqrt{x}}
]

where:

* `x` = number of sequences per week
* `t₀` = time to isolate a chain.

---

## 4. Dataset Generation – Predominant Chains

```
4_Dataset_predominant_chains.ipynb
```

Using real-world chains, this notebook generates **synthetic predominant chains** by:

* sampling from distributions fitted on real parameters
* adding noise to simulate early epidemic dynamics.

---

## 5. Dataset Generation – Transient Chains

```
5_Dataset_transient_chains.ipynb
```

Transient chains are simulated using distributions of weekly prevalence differences.

Two classes are generated:

* **above-threshold transient chains**
* **below-threshold transient chains**

These mimic variants that fail to spread widely.

---

## 6. DNN Dataset Construction

```
6_DNN_dataset.ipynb
```

This step merges:

* simulated predominant chains
* simulated transient chains

to produce the dataset used to train the classifier.

Features include:

* early prevalence values
* sigmoid fit parameters
* derivatives of growth parameters.

---

## 7. Deep Neural Network Training

```
7_DNN_training.ipynb
```

A **Deep Neural Network classifier** is trained to identify variants likely to become dominant.

Architecture:

* Input layer: 12 features
* Hidden layers: 64 → 32 neurons
* Activation: ReLU
* Output: sigmoid

Training:

* optimizer: Adam
* loss: binary cross-entropy
* epochs: 50
* batch size: 128

Performance is evaluated using:

* ROC curves
* false positive rates
* confusion matrices.

---

# Hospitalization Modeling (Denmark)

After variant detection, hospitalization trends are modeled using additional notebooks.

---

## Denmark Data Preparation

```
Denmark_data_preparation.ipynb
```

This notebook prepares the dataset combining:

* variant chains
* vaccination coverage
* containment measures
* hospitalization data
* genetic distances between variants.

---

## CatBoost Regression Model

```
CatBoost_Denmark.ipynb
```

A **CatBoost regression model** is used to predict weekly hospitalizations per variant chain.

Key predictors include:

* detection week
* genetic distance between variants
* genetic distance from vaccine strains
* vaccination coverage
* containment measures.

Model outputs include:

* predicted hospitalizations
* feature importance analysis
* SHAP interpretation.

---

# Environment

The analysis can be reproduced using the provided environment file.

Create the environment with:

```
conda env create -f environment.yml
```

Activate it:

```
conda activate covML
```

---

# Data Availability

SARS-CoV-2 sequence data were obtained from:

GISAID
[https://www.gisaid.org](https://www.gisaid.org)

Additional epidemiological data sources include:

* Statens Serum Institut (Denmark)
* European Centre for Disease Prevention and Control
* Oxford COVID-19 Government Response Tracker.

Due to GISAID data sharing policies, raw sequence data are not redistributed in this repository.

---

# Citation

If you use this repository, please cite:

*From Sequences to Strategies: Early Detection of New SARS-CoV-2 Variants via Genetic Distance to Reduce Hospitalizations.*

Preprint:

[https://doi.org/10.1101/2025.09.03.25334908](https://doi.org/10.1101/2025.09.03.25334908)

---

# License

This repository is released for academic research purposes.


