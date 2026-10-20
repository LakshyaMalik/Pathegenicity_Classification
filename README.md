# ClinVar Genetic Variant Pathogenicity Classification

An end-to-end machine learning pipeline for classifying the pathogenicity of human genetic variants using ClinVar dataset annotations. This project explores binary and multi-class target mappings across Random Forest, XGBoost, and PyTorch Feedforward Neural Network (FFN) architectures, with a focus on handling severe class imbalances and noisy text features.

---

## 📌 Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Dataset & Features](#dataset--features)
- [Model Architectures](#model-architectures)
- [Class Imbalance & Optimization](#class-imbalance--optimization)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [Results & Performance](#results--performance)

---

## 🔬 Overview
Classifying genetic variants according to ACMG/AMP standards (e.g., *Pathogenic*, *Likely Pathogenic*, *Likely Benign*, *Benign*) often suffers from fuzzy decision boundaries and heavy class imbalance. This repository provides a workflow to:
1. Clean and preprocess complex genomic metadata (such as Molecular Consequence (`MC`) strings).
2. Experiment with **4-way multi-class classification** (`CLNSIG`) vs. clinically collapsed **binary classification** (*Pathogenic/Likely Pathogenic* vs. *Benign/Likely Benign*).
3. Compare ensemble models (Random Forest, XGBoost) against Deep Neural Networks (PyTorch FFN) using cost-sensitive learning and balanced sample weighting.

---

## ✨ Key Features
- **Robust Text Cleaning**: Automated handling of missing/malformed text strings in genomic annotation columns (e.g., parsing raw `MC` values into a clean `MC_Clean` feature space).
- **Cost-Sensitive Learning**: Implementation of custom class weight matrices for Random Forest and dynamic `sample_weight` arrays for XGBoost to heavily penalize false-negative predictions on disease-causing variants.
- **Binary Mapping**: Optional collapse of compound/ambiguous labels into binary targets to align with clinical decision-making.
- **Deep Learning Pipeline**: Modular PyTorch Feedforward Neural Network with Dropout regularization for tabular variant data.

---

## 📊 Dataset & Features
The pipeline processes tabular genomic data containing the following primary features:

| Feature | Description |
| :--- | :--- |
| `CHROM` | Chromosome location |
| `POS` | Genomic position |
| `REF` / `ALT` | Reference and alternate alleles |
| `CLNSIG` | Target clinical significance variable |
| `MC` / `MC_Clean` | Raw and cleaned Molecular Consequence terms |
| `AF_EXAC` / `AF_TGP` / `AF_ESP` | Allele frequencies across ExAC, 1000 Genomes, and ESP |
| `CLNVC` | Variant type (e.g., single nucleotide variant, deletion) |

---

## 🛠️ Model Architectures

### 1. Random Forest (`scikit-learn`)
* Evaluated with `'balanced'` weights

### 2. XGBoost (`xgboost`)
* Gradient-boosted decision trees trained with scikit-learn's `compute_sample_weight` array to handle multi-class frequency disparities efficiently.

### 3. Deep Learning (`PyTorch FFN`)
* Multi-layer Feedforward Neural Network architecture with `ReLU` activations and regularized `Dropout(0.3)` layers.

---

## ⚖️ Class Imbalance & Optimization
Due to the vast over-representation of benign variants over rare pathogenic variants (e.g., >1.14M benign vs. minority pathogenic classes), the project incorporates multiple mitigation strategies:
- **Sample Weighting**: Adjusting sample loss gradients in XGBoost.
- **Custom Penalty Matrices**: Assigning higher error weights to misclassifications of *Pathogenic* (e.g., weight = 6) vs. *Benign* (weight = 1).
- **Binary Decision Collapse**: Mapping targets to remove ambiguity between *Likely Pathogenic* and *Pathogenic*.

---

## ⚡ Installation & Setup

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
   cd your-repo-name
