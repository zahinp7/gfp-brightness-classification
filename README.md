# GFP Brightness Classification from Protein Sequence

Binary classification of GFP mutant brightness (high vs low) from amino acid sequence, using hand-crafted physicochemical feature engineering and linear classifiers.

---

## Problem

Given a library of GFP mutant sequences (237 amino acids each), predict whether a mutant exhibits high brightness (class 1) or low brightness (class 0). The challenge is representing protein sequences as numeric features without using pre-trained protein language models.

- **31,029 training samples** with binary brightness labels
- **20,686 held-out test samples** (no labels)
- Class distribution: ~61% low brightness, ~39% high brightness
- Primary metric: **F1-score**

---

## Feature Engineering

Three complementary representations are concatenated into a single 2,086-dimensional sparse feature matrix:

| Feature set | Description | Dimensions |
|---|---|---|
| One-hot encoding | Position-specific residue identity (per-position OHE across 237 positions) | 1,930 |
| AA composition | Global frequency of each of 20 amino acids across the full sequence | 20 |
| Physicochemical descriptors | Mean, std, min, max of 6 established amino acid descriptor tables across sequence | 136 |

The descriptor tables encode known biochemical properties per amino acid:

- **DPPS** (Tian et al. 2008) -- electronic, steric, hydrophobic, and H-bond properties (10 PCs)
- **MS-WHIM** (Zaliani & Gancia 1999) -- 3D surface-derived steric and electrostatic indices (3 PCs)
- **Physical** (Barley et al. 2018) -- hydrophilicity and volume (2 orthogonal scales)
- **ST-scale** (Yang et al. 2010) -- structural topology from 827 structural variables (8 PCs)
- **VHSE-scale** (Mei et al. 2005) -- hydrophobic, steric, and electronic PCA scores (8 PCs)
- **Z-scale** (Hellberg et al. 1987) -- hydrophilicity, bulk, and electronic properties (3 PCs)

---

## Results

| Model | Validation F1 | Notes |
|---|---|---|
| LinearSVC (C=0.5) | 0.8660 | Best model |
| Logistic Regression (saga, L2) | 0.8625 | Close second |
| HistGradientBoosting | 0.8276 | Dense conversion required |
| Random Forest | 0.6909 | Underperforms on sparse data |

LinearSVC and Logistic Regression outperform tree-based models on this high-dimensional sparse feature space, consistent with what is generally observed for text-like or sparse biochemical representations.

---

## Repository Structure

- `notebooks/` - main analysis notebook
- `data/` - raw sequence data (not tracked, see note below) and descriptor tables
- `results/`
  - `final.csv` - final test set predictions

> **Note on data:** `train_X.csv`, `train_y.csv`, `test_X.csv`, and `y_sample_submission.csv` are not tracked due to size. The descriptor CSVs in `data/descriptors/` are tracked and required to reproduce results.

---

## Setup and Reproduction

```bash
git clone https://github.com/zahinp7/gfp-brightness-classification.git
cd gfp-brightness-classification
pip install -r requirements.txt
```

Place the four data CSVs in `data/`, then run:

```bash
jupyter notebook notebooks/gfp_brightness_classification.ipynb
```

---

## Dependencies

See `requirements.txt`. Core stack: `scikit-learn`, `pandas`, `numpy`, `scipy`.

---

## Skills

- Protein sequence feature engineering (OHE, composition, physicochemical descriptors)
- High-dimensional sparse classification with linear SVMs
- Amino acid descriptor literature integration (DPPS, MS-WHIM, VHSE, Z-scale)
- Stratified train/validation splitting and regularization hyperparameter search
