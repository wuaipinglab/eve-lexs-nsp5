# eve-lexs-nsp5

**A Learn–Expand–Sample pipeline for designing high-activity SARS-CoV-2 NSP5 (Mpro) substrates.**

<p align="center">English | <a href="README.zh.md">中文</a></p>

---

## Overview

This repository contains the computational pipeline and associated data for the section **AI-driven accelerated evolution generates VIDAs with enhanced antiviral potency** in the paper [`Li, Lin, et al. "Viral Protease-Initiated Lytic Cell Death as a Universal Antiviral mRNA Therapy." Cell, 2026.`](https://www.cell.com/cell/abstract/S0092-8674(26)00751-8)

VIPA (Viral protease-Initiated Pyroptosis Activator) is a universal antiviral mRNA therapy platform. Its core mechanism is to replace the native cleavage site of gasdermin D with a substrate sequence of the target viral protease, so that VIPA is activated only in virus-infected cells and triggers pyroptosis, thereby selectively eliminating infected cells. In its application to SARS-CoV-2, to enhance the cleavage efficiency and mutation tolerance of the SARS-CoV-2 VIPA, we developed an evolution-constrained substrate design framework — **Learn–Expand–Sample (LEXS)** — for *de novo* design of the 8-amino-acid cleavage motif (P6–P2′) of the NSP5 (Mpro) protease.

### Biosafety Statement

To minimize potential biosafety risks, the scripts used for candidate sequence generation and sampling are not publicly released at this time. However, they may be provided upon reasonable request to qualified researchers for research purposes, subject to a biosafety review.

### Pipeline

|    Stage    | Description |
|:-----------:|-------------|
|  **Learn**  | Train an [EVE](https://github.com/OATML-Markslab/EVE) (Evolutionary model of Variant Effects) model to learn evolutionary constraints from 261 non-redundant natural SARS-CoV-2 NSP5 P6–P2′ cleavage motifs collected from the NCBI Virus database. |
| **Expand**  | Use the trained EVE model to generate 100,000 candidate sequences and quantify the fitness of each sequence via evolutionary scores; the generated set covers the natural sequence space and expands into unobserved regions. |
| **Sample**  | After de-redundancy and score-based filtering of the generated sequences, select 30 diverse representative candidate sequences via K-Means clustering and centroid-proximal sampling for subsequent experimental validation. |

<p align="center">
  <img src="assets/workflow.svg" alt="Pipeline schematic" width="600">
</p>

### Key Results

- On an independent test set (25 sequences with known cleavage outcomes), the classification performance of EVE evolutionary scores: **ROC-AUC = 0.80, F1 = 0.78**
- Of the 25 successfully synthesized candidate peptides, **7** showed cleavage rates more than **2×** that of the wild-type substrate
- Three of these VIPA variants exhibited **>90%** viral inhibition in SARS-CoV-2–infected cells

<table>
  <tr>
    <td align="center"><img src="assets/cleavage.svg" alt="Cleavage kinetics of AI-generated peptide candidates" width="600"></td>
    <td align="center"><img src="assets/antiviral.svg" alt="Antiviral activity in HeLa-ACE2 cells" width="200"></td>
  </tr>
  <tr>
    <td align="center">Cleavage kinetics of AI-generated candidate peptides</td>
    <td align="center">Antiviral activity measured in cellular assays</td>
  </tr>
</table>

---

## Repository Structure

```
├── src/
│   ├── README.md                      # EVE setup instructions
│   └── EVE/                           # [Not included] must be prepared manually
│
├── data/
│   ├── raw/
│   │   ├── SARS-CoV-2_nsp_sites.xlsx  # Raw NSP5 P6–P2′ sequences (from NCBI)
│   │   └── nsp5_testset.fasta         # Test set (labels embedded in IDs)
│   └── processed/
│       ├── nsp5_short.fasta
│       ├── nsp5_testset_8aa.fasta
│       ├── sars_nsp5_site_8aa_mapping.csv
│       └── nsp5_testset_8aa_mapping.csv
│
├── checkpoints/
│   ├── logistic_model.pkl             # Trained logistic regression model weights
│   └── sars_nsp5_8aa/
│       ├── nsp5_sars_nsp5_8aa_final   # VAE model weights
│       └── weights/
│           └── nsp5_theta_0.01.npy    # VAE theta weights
│
├── notebooks/
│   ├── 1_Data.ipynb                   # Data preparation
│   ├── 2_Model.ipynb                  # VAE training and testing
│   └── 3_Generate.ipynb               # Sequence generation (not publicly released)
│
├── logs/
│   └── sars_nsp5_8aa/
│       └── nsp5_sars_nsp5_8aa_losses.csv  # VAE training loss log
│
└── results/
    ├── EVE/
    │   └── sars_nsp_5_8aa_testset/
    │       ├── evol_indices/
    │       │   └── nsp5_20000_samples.csv  # Evolutionary indices of test-set point mutations computed by EVE
    │       └── mutations/
    │           └── nsp5_all_singles.csv     # List of all test-set point mutations
    └── generated_seq/
        └── sars_nsp5_8aa_final_selected.csv  # 30 final candidate sequences
```

---

## Environment Setup

### Dependencies

- Python 3.10+
- Biopython
- PyTorch (required by EVE)
- scikit-learn, NumPy, SciPy, pandas
- matplotlib, logomaker

### Clone EVE

The EVE source code (`commit 460d70efeeeded58bc69227a203540d68953ae88`) is not included in this repository and must be cloned into `src/`:

```bash
git clone https://github.com/OATML-Markslab/EVE.git src/EVE
```

---

## Notebooks

### 1 — Data Preparation (`notebooks/1_Data.ipynb`)

- Load NSP5 P6–P2′ sequences obtained from NCBI
- Visualize the amino acid distribution of the training set
- Generate processed FASTA files for training and testing
- Generate the mapping CSV files required by EVE
- Check for probable overlap between the training and test sets

### 2 — Model Training and Evaluation (`notebooks/2_Model.ipynb`)

- Train EVE on the training set (via `train_VAE.py`)
- Score all test-set sequences with the trained EVE model
- Evaluate classification performance on the test set (ROC curve)
- Train a logistic regression classifier to relate EVE scores to activity
- Save the model to `checkpoints/logistic_model.pkl`

---

## Citation

If this repository is helpful to your research, please cite:

> Li, Lin, et al. "Viral protease-Initiated Pyroptosis Activator mRNA therapy as a Universal Antiviral Strategy." Cell, 2026.

---

## License

This repository and its custom code (excluding the externally developed EVE framework) are distributed under the **Apache License 2.0**. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

- [EVE (OATML-Markslab)](https://github.com/OATML-Markslab/EVE) — variational autoencoder framework
- [NCBI Virus](https://www.ncbi.nlm.nih.gov/labs/virus/) — source of NSP5 cleavage motif data