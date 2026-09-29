# Single-Cell RNA-seq Cell Type Classification

Machine learning and representation learning pipeline for predicting cell types from single-cell RNA sequencing (scRNA-seq) gene-expression profiles.

The project explores biological preprocessing, dimensionality reduction, nonlinear representation learning, and supervised classification, with particular attention to class imbalance and distribution shift between labeled and unlabeled cells.

---

## Overview

Single-cell RNA sequencing provides high-dimensional measurements of gene expression at the individual-cell level.

In this project, each cell is represented by the expression values of thousands of genes, while experimentally assigned FACS labels provide the target cell type for supervised learning.

The main objective is:

> **Can cell type be predicted from high-dimensional single-cell gene-expression profiles, and can nonlinear representations improve classification compared with standard PCA-based approaches?**

The analysis combines:

- exploratory analysis of scRNA-seq data;
- highly variable gene identification;
- gene filtering;
- analysis of housekeeping genes;
- PCA;
- t-SNE and UMAP visualization;
- autoencoder-based representation learning;
- Random Forest classification;
- Multilayer Perceptron experiments;
- stratified cross-validation;
- class-imbalance handling;
- labeled/unlabeled distribution-shift correction;
- pseudo-labeling and alternative preprocessing experiments.

---

## Dataset

The main single-cell dataset contains:

- **1,499 cells**
- **4,290 genes**
- gene-expression measurements stored in an `AnnData` object;
- cell identifiers in the observation metadata;
- gene identifiers and gene symbols in the variable metadata.

The expression matrix has shape:

```text
1499 cells × 4290 genes
```

A subset of the cells has experimentally assigned cell-type labels used for supervised learning.

For analyses requiring known labels, **1,074 labeled cells** are available.

The remaining cells form the unlabeled evaluation set.

The input expression values in the provided dataset are already normalized and log-transformed.

---

## Prediction Task

The project is formulated as a supervised multiclass classification problem.

### Input

For each cell:

```text
x ∈ R^4290
```

where each feature corresponds to the expression level of one gene.

### Target

The target is the FACS-assigned cell type.

The FACS labels are treated as the reference labels for supervised model development.

---

## Analysis Pipeline

The project follows the workflow:

```text
Single-cell expression matrix
        │
        ▼
Dataset inspection
        │
        ▼
Gene-level exploration
        │
        ▼
Highly Variable Gene analysis
        │
        ▼
Gene filtering
        │
        ├───────────────┐
        ▼               ▼
With housekeeping   Without housekeeping
genes               genes
        │               │
        └───────┬───────┘
                ▼
               PCA
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
       PCA    t-SNE     UMAP
                │
                ▼
          Autoencoder
                │
                ▼
     Latent representation
                │
                ▼
  PCA vs Autoencoder comparison
                │
                ▼
        Supervised learning
                │
        ┌───────┴────────┐
        ▼                ▼
 Random Forest          MLP
        │
        ▼
Distribution-shift correction
        │
        ▼
Autoencoder + Random Forest
        │
        ▼
    Final predictions
```

---

# 1. Exploratory Analysis

The first part of the project examines the structure of the single-cell dataset.

The analysis includes:

- number of cells and genes;
- expression matrix representation;
- sparsity and expression distribution;
- gene-level statistics;
- cell-level expression characteristics;
- label distribution;
- class imbalance.

Single-cell RNA-seq is particularly challenging because gene-expression measurements are high-dimensional and noisy, while biologically related cell populations may have overlapping transcriptional profiles.

---

# 2. Highly Variable Genes

Highly Variable Genes (HVGs) are identified using Scanpy.

The analysis selects approximately:

```text
2,000 HVGs out of 4,290 genes
```

These genes represent the subset showing the greatest variability across cells and are useful for identifying biologically informative transcriptional structure.

HVG analysis is used primarily for exploration and understanding of the dataset rather than as the only source of features for the final classifier.

---

# 3. Gene Filtering

Genes with insufficient expression information are removed before dimensionality reduction.

The project also explicitly investigates the effect of **housekeeping genes**.

Two alternative datasets are considered:

```text
Filtered expression matrix including housekeeping genes
```

and

```text
Filtered expression matrix excluding housekeeping genes
```

This comparison allows us to evaluate whether genes associated with basic cellular functions contribute useful information to cell-type discrimination or mainly introduce shared background expression.

The experiments do not show a sufficiently strong advantage from removing housekeeping genes to justify treating their exclusion as universally beneficial.

---

# 4. Principal Component Analysis

Principal Component Analysis (PCA) is used as the primary linear dimensionality-reduction baseline.

PCA provides:

- noise reduction;
- feature compression;
- a lower-dimensional representation of the expression matrix;
- an input representation for downstream visualization and classification.

The explained variance increases gradually with additional components, reflecting the highly distributed nature of transcriptional variation in the dataset.

PCA is also used as preprocessing for nonlinear visualization methods.

---

# 5. t-SNE and UMAP

Two nonlinear visualization methods are explored:

- **t-SNE**
- **UMAP**

These methods are used to visualize relationships among cells in low-dimensional space.

They help investigate whether FACS-defined cell populations form distinct transcriptional groups and whether biologically related populations occupy neighboring regions of the embedding.

These visualizations are exploratory and are not interpreted as direct measures of classification performance.

---

# 6. Autoencoder Representation Learning

To move beyond linear dimensionality reduction, the project implements a neural-network autoencoder using PyTorch.

The autoencoder consists of:

```text
High-dimensional gene expression
          │
          ▼
       Encoder
          │
          ▼
   Latent representation
          │
          ▼
       Decoder
          │
          ▼
Reconstructed gene expression
```

The model learns a nonlinear compressed representation by minimizing reconstruction error.

Multiple latent-space dimensionalities are explored to study the trade-off between compression and reconstruction quality.

---

# 7. PCA vs Autoencoder

PCA and the Autoencoder are compared using two complementary criteria.

## Reconstruction Error

The Autoencoder generally obtains lower reconstruction error than PCA at comparable latent dimensionalities.

Approximate reconstruction-error ranges observed in the notebook are:

| Method | Reconstruction MSE |
|---|---:|
| PCA | ~4.2–5.0 |
| Autoencoder | ~3.4–4.7 |

The Autoencoder therefore captures nonlinear information that improves reconstruction of the original expression profiles.

## Silhouette Score

Cluster separation is evaluated on the labeled cells using the Silhouette Score.

The computed values are approximately:

| Representation | Silhouette Score |
|---|---:|
| PCA | **0.0124** |
| Autoencoder | **0.0113** |

Both scores are close to zero.

This indicates that the known cell types are not cleanly separated into compact clusters in either latent representation.

Importantly, improved reconstruction does not automatically imply improved cluster separation.

---

# 8. Supervised Classification

After representation learning, the project evaluates supervised machine-learning models for cell-type prediction.

The main classifiers explored are:

- **Random Forest**
- **Multilayer Perceptron (MLP)**

Stratified validation procedures are used because the class distribution is highly imbalanced.

Random Forest models proved particularly useful in this relatively small labeled dataset because they provided stable performance without requiring a large neural-network training set.

---

# 9. Handling Class Imbalance

The cell-type distribution is strongly imbalanced.

Several strategies are considered to reduce the impact of rare classes.

A particularly difficult category is class `9`, corresponding to an `"other"` group.

This class is:

- very small;
- biologically heterogeneous;
- difficult to model reliably.

In some supervised experiments, class 9 is therefore excluded from classifier training while still being retained when learning the global dimensionality-reduction representation.

This is an experiment-specific modeling decision rather than a general recommendation for single-cell analysis.

---

# 10. Distribution Shift

One of the most important observations in the project is the presence of a distribution difference between:

- cells with known labels;
- cells in the unlabeled evaluation set.

A model performing well under validation on the labeled samples did not necessarily obtain equivalent performance on the evaluation data.

This motivated the use of the unlabeled expression distribution when constructing the feature representation.

For the PCA pipeline, adapting the representation to the unlabeled distribution improved leaderboard performance from approximately:

```text
~0.47 → ~0.49
```

The notebook reports a PCA + Random Forest leaderboard accuracy of approximately:

```text
0.499
```

These experiments suggest that, for this dataset, explicitly accounting for the labeled/unlabeled distribution difference improves generalization to the evaluation set.

---

# 11. Autoencoder + Random Forest

The strongest pipeline developed in the notebook combines nonlinear representation learning with a tree-based classifier.

The pipeline is:

```text
Gene-expression matrix
        │
        ▼
   Autoencoder
        │
        ▼
Latent representation
        │
        ▼
Latent-space shift correction
        │
        ▼
 Random Forest
        │
        ▼
Cell-type prediction
```

The motivation is to combine:

- nonlinear compression from the Autoencoder;
- information from the unlabeled distribution;
- robustness of Random Forest classification.

---

## Main Result

The strongest reported evaluation result is approximately:

| Pipeline | Leaderboard Accuracy |
|---|---:|
| PCA + Random Forest | **0.499** |
| Autoencoder + latent-shift correction + Random Forest | **0.528** |

The Autoencoder-based pipeline therefore improves the leaderboard score relative to the PCA + Random Forest baseline in the experiments reported in the notebook.

---

# 12. Additional Experiments

Several additional approaches are explored during model development.

These include:

- different PCA dimensionalities;
- different Autoencoder latent dimensions;
- alternative Random Forest hyperparameters;
- MLP classifiers;
- PCA/Autoencoder model combinations;
- pseudo-labeling;
- alternative input transformations;
- feature binning;
- different normalization strategies.

Some of these approaches achieve very high accuracy when evaluated on labeled training data, but those values should not be interpreted as equivalent to held-out or leaderboard performance.

For this reason, the leaderboard score is used as the main external performance reference in this repository.

---

# 13. Main Findings

The experiments lead to several observations.

### 1. Single-cell classification is difficult

Cell populations are not perfectly separable from the available gene-expression measurements.

Both PCA and Autoencoder embeddings have low Silhouette Scores.

### 2. Nonlinear representations improve reconstruction

The Autoencoder generally reconstructs expression profiles more accurately than PCA.

### 3. Reconstruction quality and clustering quality are different objectives

Although the Autoencoder achieves lower reconstruction error, its Silhouette Score is not higher than PCA's.

### 4. Random Forest is a strong classifier for this dataset

In the experiments performed here, Random Forest provides a stable supervised model despite the limited and imbalanced labeled dataset.

### 5. Distribution shift matters

Accounting for differences between labeled and unlabeled cells improves evaluation-set performance in this dataset.

### 6. The final Autoencoder-based pipeline outperforms the PCA baseline

The final Autoencoder + latent-shift correction + Random Forest pipeline reaches approximately **0.528 leaderboard accuracy**, compared with approximately **0.499** for the PCA + Random Forest baseline.

---

# Repository Structure

A suggested repository organization is:

```text
single-cell-cell-type-classification/
│
├── notebooks/
│   └── single_cell_analysis.ipynb
│
├── data/
│   ├── random_singlecell.csv
│   ├── singlecell_train.csv
│   └── single_cell.h5ad        # not tracked by Git
│
├── README.md
├── requirements.txt
└── .gitignore
```

If the original notebook filename is retained instead, the analysis notebook may appear as:

```text
LAB2_BEATRICE_MATILDE_JEAN_GUTHLAC.ipynb
```

---

# Data Availability

The main expression dataset is stored in:

```text
single_cell.h5ad
```

The file is intentionally excluded from Git version control because it is a relatively large binary dataset and may depend on the original course/data distribution conditions.

The smaller CSV files may be included in the repository if their redistribution is permitted.

To reproduce the analysis, place the required data files in the expected data directory before running the notebook.

Example:

```text
data/
├── single_cell.h5ad
├── singlecell_train.csv
└── random_singlecell.csv
```

If the notebook is kept unchanged, update the input paths accordingly or keep the files in the same directory as the notebook.

---

# Installation

Clone the repository:

```bash
git clone https://github.com/<YOUR-USERNAME>/single-cell-cell-type-classification.git
cd single-cell-cell-type-classification
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

or on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Launch Jupyter:

```bash
jupyter notebook
```

Then open the analysis notebook.

---

# Main Dependencies

The project uses:

- Python
- NumPy
- pandas
- SciPy
- AnnData
- Scanpy
- Matplotlib
- scikit-learn
- UMAP
- PyTorch
- Jupyter

---

# Reproducibility

The notebook contains the complete experimental workflow, including preprocessing, dimensionality reduction, representation learning, classification and evaluation.

However, exact numerical results may vary slightly because some algorithms involve stochastic initialization or randomized training.

For more reproducible experiments, random seeds should be fixed consistently for:

- NumPy;
- scikit-learn;
- PyTorch;
- UMAP;
- train/validation splitting.

---

# Limitations

Several limitations should be considered when interpreting the results.

### Small labeled dataset

Only a subset of the available cells has known FACS labels, limiting the amount of supervised training data.

### Class imbalance

Some cell populations are represented by substantially fewer examples than others.

### Rare heterogeneous class

Class 9 (`other`) is difficult to model because it contains very few heterogeneous examples.

### Distribution shift

The labeled and unlabeled populations are not identically distributed, making standard cross-validation an imperfect estimate of evaluation-set performance.

### Low latent-space cluster separation

Neither PCA nor the Autoencoder produces strongly separated clusters according to the Silhouette Score.

### Leaderboard-based model selection

Some modeling choices were informed by leaderboard performance. Therefore, the leaderboard score should not be interpreted as an entirely untouched independent test set.

### Autoencoder stochasticity

Autoencoder results depend on initialization, optimization parameters and latent-space dimensionality.

### Dataset-specific conclusions

The findings describe the behavior of the methods on this dataset and should not be interpreted as universal conclusions about single-cell RNA-seq classification.

---

# Future Work

Possible extensions include:

- nested cross-validation for model selection;
- repeated stratified evaluation across multiple random seeds;
- improved calibration for imbalanced classes;
- class-weighted neural-network training;
- alternative representation-learning methods;
- variational autoencoders;
- contrastive learning;
- graph-based single-cell representations;
- semi-supervised learning using both labeled and unlabeled cells;
- more systematic hyperparameter optimization;
- feature attribution and marker-gene analysis;
- comparison with specialized single-cell annotation methods.

---

# Conclusion

This project investigates cell-type prediction from high-dimensional single-cell RNA-seq data using both classical and nonlinear machine-learning approaches.

The experiments show that dimensionality reduction alone does not create clearly separated cell-type clusters, but learned representations can still provide useful predictive features.

The best reported pipeline combines:

```text
Autoencoder
    +
latent distribution-shift correction
    +
Random Forest
```

and achieves approximately:

```text
0.528 leaderboard accuracy
```

compared with approximately:

```text
0.499
```

for the PCA + Random Forest baseline.

The results highlight the importance of evaluating not only representation quality, but also class imbalance, limited labeled data and distribution differences between training and evaluation populations.