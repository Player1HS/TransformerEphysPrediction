# The Unreasonable Effectiveness of Cell Types in Describing Neuronal Physiological Features

This repository contains the code accompanying the paper:

> **The Unreasonable Effectiveness of Cell Types in Describing Neuronal Physiological Features**
> Harshil Sharma, Xiao-Ping Liu, Thomas Chartrand, Brian Kalmbach, Ed Lein, Stefan Mihalas, Zhixin Lu.

The manuscript itself is included as [main.pdf](main.pdf).

## Overview

We benchmark four transcriptomic representations of human cortical neurons for their ability to predict intrinsic electrophysiological features from paired Patch-seq data (495 neurons; 229 L1 GABAergic + 266 L2/3 glutamatergic):

1. **scGPT cell embeddings** (512-dim) from the scGPT model pretrained on 13.2M human brain cells.
2. **Top 512 highly variable genes** (HVG) selected via Scanpy.
3. **328 ion channel-coding genes** from the HGNC database.
4. **15 transcriptomic cell types** encoded as one-hot vectors.

For each representation we train a multilayer perceptron (MLP) to predict eight percentile-normalized electrophysiological features (sag ratio, resting membrane potential, rheobase, f-I slope, input resistance, time constant τ, mean firing rate, adaptation). Performance is evaluated against a cell-type-mean baseline using stratified 4-fold cross-validation with nested 3-fold hyperparameter tuning (100 trials) via Optuna. All MLPs share the same architecture: two tanh-activated hidden layers of 512 units, trained with SGD (batch size 25) for 1500 epochs under an MSE loss.

Four experiments are reported in the paper, each implemented as its own notebook pair (`*_comparison.ipynb` for training and `*_plotting.ipynb` for figures):

| Experiment | Notebook folder | Description |
|---|---|---|
| Baseline (no cell type info) | [Human/wo_types_comparison/](Human/wo_types_comparison) | Three MLPs taking only scGPT / HVG / ion channel inputs. |
| With one-hot cell type | [Human/w_types_comparison/](Human/w_types_comparison) | Same three inputs concatenated with a 15-dim one-hot cell type vector. |
| Cell-type-initialized | [Human/celltypeinitialized_comparison/](Human/celltypeinitialized_comparison) | Same as above, but with weights initialized from a network pretrained on cell type alone. |
| Dual MLP fusion | [Human/DualMLP/](Human/DualMLP) | Two separately pretrained sub-MLPs (transcriptomic + one-hot cell type) combined via a fusion layer initialized to compute their average. |

## Repository structure

```
.
├── main.tex / main.pdf                 # Manuscript
├── environment.yml                     # Conda environment specification
└── Human/
    ├── Data Preprocessing/
    │   └── data_preprocessing.ipynb    # Builds the .h5ad + pickled ephys dataframe used by all downstream notebooks
    ├── wo_types_comparison/            # Baseline comparison
    ├── w_types_comparison/             # With one-hot cell type concatenation
    ├── celltypeinitialized_comparison/ # With cell-type pretraining initialization
    ├── DualMLP/                        # Dual MLP fusion model
    ├── onefeaturepermlp/               # Supplementary analysis: one MLP per ephys feature
    └── Supplement/                     # Further supplementary plotting
```

## Data

The study uses Patch-seq recordings of human neocortical neurons collected by the Allen Institute for Brain Science, combining the L1 GABAergic interneuron dataset of Chartrand et al. (2023) with the L2/3 glutamatergic neuron dataset of Berg et al. (2021). Raw data are available from the corresponding publications and the Allen Brain Map. The preprocessing notebook expects:

- Per-cell transcriptomic CSVs (intron + exon counts, 50,281 genes) for L1 and L2/3.
- Per-cell electrophysiological feature tables for L1 and L2/3.
- The scGPT brain pretrained model checkpoint and gene vocabulary (`scGPT_brain/`).
- The HGNC ion-channel gene list (`ion_channel_genes.csv`).

After filtering to genes overlapping the scGPT vocabulary (30,682 genes), dropping the latency feature, and removing cell types with fewer than four cells, the preprocessing notebook outputs the cleaned `.h5ad` and pickled ephys dataframe used by every downstream notebook.

## Reproducing the results

1. Create the environment:
   ```bash
   conda env create -f environment.yml
   conda activate gene2ephys
   ```
2. Obtain the raw Patch-seq data, scGPT brain checkpoint, and ion channel gene list (see *Data* above) and place them at the paths expected in [Human/Data Preprocessing/data_preprocessing.ipynb](Human/Data%20Preprocessing/data_preprocessing.ipynb).
3. Run the preprocessing notebook to generate the processed dataset.
4. Run each `*_comparison.ipynb` notebook to reproduce the corresponding experiment.
5. Run the matching `*_plotting.ipynb` notebook to regenerate the paper figures.

## Citation

If you use this code, please cite the paper (citation to be updated upon publication).
