# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AGILE (AI-Guided Ionizable Lipid Engineering) is a deep learning platform for accelerating LNP (Lipid Nanoparticle) development for mRNA delivery. It uses Graph Neural Networks (GNN) to predict ionizable lipid properties and screen candidate libraries.

The codebase is built on top of [MolCLR](https://github.com/yuyangw/MolCLR) with MIT license.

## Commands

```bash
# Pre-training (self-supervised on molecular graphs)
python pretrain.py config_pretrain.yaml

# Fine-tuning (supervised regression on specific cell lines)
python finetune.py config_finetune.yaml

# Inference and visualization
python infer_vis.py <folder_name_of_finetuned_model>
```

**Installation** (from requirements.txt):
```bash
pip install -r requirements.txt
# Also requires PyTorch with CUDA, torch-geometric, torch-sparse, torch-scatter
# See README.md for exact versions (torch==1.12.1+cu113, etc.)
```

## Architecture

### Workflow
1. **Pre-training** (`pretrain.py`): Self-supervised learning using NT-Xent loss on molecular graph augmentations (node/subgraph/mix strategies)
2. **Fine-tuning** (`finetune.py`): Transfer learning with frozen base GNN + trainable prediction head for regression tasks
3. **Inference** (`infer_vis.py`): Model evaluation with UMAP visualization and molecular property analysis

### Key Components

- **Models** (`models/`):
  - `agile_pretrain.py` - Pre-training model with GINE conv layers, returns embeddings + projections
  - `agile_finetune.py` - Fine-tuning model with additional prediction head, supports extra molecular features

- **Datasets** (`dataset/`):
  - `dataset.py` - Pre-training dataset with graph augmentation (node masking)
  - `dataset_subgraph.py` - Subgraph augmentation strategy
  - `dataset_mix.py` - Mixed augmentation strategy
  - `dataset_test.py` - Fine-tuning/test dataset with labeled data

- **Utils** (`utils/`):
  - `nt_xent.py` - NT-Xent contrastive loss for pre-training
  - `constants.py` - LNP structural constants (R2/R3 types, chain lengths)
  - `plot.py` - Visualization utilities

### Data Format

- CSV files with `smiles` column (molecular SMILES strings)
- Fine-tuning sets include target columns (e.g., `expt_Hela`, `expt_Raw`)
- Descriptor columns prefixed with `desc_` are auto-detected as additional features

### Checkpoints

- Pre-trained models: `ckpt/<model_name>/checkpoints/model.pth`
- Fine-tuned models: `finetune/<timestamp>_<task_name>_<target>/checkpoints/model.pth`
