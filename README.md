# COS802 Project

This project investigates **On-Policy Self-Distillation (OPSD)** for visual
question answering on a crop-suitable subset of the GQA dataset. The
experiment compares three model architectures and three conditions:

- pretrained **Base** model;
- supervised fine-tuning (**SFT**) with LoRA;
- on-policy self-distillation (**OPSD**) with LoRA.

The primary metric is BERTScore-F1. Normalised exact match is reported as a
secondary metric.

## Project workflow

Run the notebooks in this order:

1. **[01_dataset_setup.ipynb](src/notebooks/01_dataset_setup.ipynb)** —
   **Local** dataset preparation. It loads the GQA annotations, filters
   questions that can be answered from a local crop, obtains the required
   images, creates padded crops, and creates image-disjoint train,
   validation, and test splits.
2. **[02_model_baselines.ipynb](src/notebooks/02_model_baselines.ipynb)** —
   **Local** baseline evaluation. It evaluates the pretrained models on the
   validation split using the full image and the privileged crop.
3. **[03_sft_training.ipynb](src/notebooks/03_sft_training.ipynb)** —
   **Kaggle** SFT training on two NVIDIA T4 GPUs using PyTorch DDP. The
   model receives the full image and question and learns the ground-truth
   answer.
4. **[04_opsd_training.ipynb](src/notebooks/04_opsd_training.ipynb)** —
   **Kaggle** OPSD training on two NVIDIA T4 GPUs using PyTorch DDP. The
   full-image student and privileged-crop teacher score the same student generated
   answer trajectory, and training minimises their token-level Jensen-Shannon
   divergence.
5. **[05_final_test_evaluation.ipynb](src/notebooks/05_final_test_evaluation.ipynb)** —
   **Local** held-out test evaluation. It compares Base, SFT, and OPSD for
   SmolVLM, BLIP, and MiniCPM, and produces paired OPSD-versus-SFT analysis
   and bootstrap confidence intervals.

The test split is not used for checkpoint selection. The best SFT and OPSD
adapters are selected using validation BERTScore-F1 in Notebooks 3 and 4.

## Setup

The project requires Python 3.11 or newer. From the project root:

```bash
uv sync
```

The dependencies are listed in [pyproject.toml](pyproject.toml), with the
resolved environment recorded in [uv.lock](uv.lock).

Before running Notebook 1, follow the download and directory instructions in
the [data README](src/data/README.md). Run local notebooks from the project
root so their path discovery can find `src/`.

## Local notebooks

Notebooks 1, 2, and 5 require a local Python/Jupyter environment. Notebook 1
creates the processed data used by the rest of the pipeline. Notebook 2
requires the processed validation split and downloads the required pretrained
model weights through Hugging Face when they are not already cached. Notebook
5 additionally requires the best SFT and OPSD adapters copied from Kaggle
into the local model directories described below.

## Kaggle notebooks

Notebooks 3 and 4 are intended to run on Kaggle with **two NVIDIA T4 GPUs**.
Attach a Kaggle dataset containing the generated `processed/`, `images/`, and
`crops/` directories. Set `KAGGLE_DATASET_SLUG` to the mounted dataset name
when it differs from the value in the notebook.

Both training notebooks:

- use seed 42 by default and run three epochs;
- train LoRA adapters for `smolvlm`, `blip`, or `minicpm`;
- use BERTScore-F1 for best-checkpoint selection;
- save validation predictions, metrics, epoch adapters, and a `best/`
  adapter under `/kaggle/working/`.

Kaggle training uses read-only input data under `/kaggle/input/` and writes
models and results under `/kaggle/working/`. Preserve the best adapters and
training results after each Kaggle run before starting a new session.

## Moving Kaggle outputs back locally

For each model, copy the selected adapters into:

```text
src/models/
├── sft/
│   └── <model>/seed42/best/
└── opsd/
    └── <model>/seed42/best/
```

Each adapter directory should contain `adapter_config.json` and the adapter
weights (normally `adapter_model.safetensors`). Notebook 5 checks these paths
before evaluating the adapted models.

Training result files can be copied into:

```text
src/results/training/sft/<model>/seed42/
src/results/training/opsd/<model>/seed42/
```

Notebook 5 writes final evaluation outputs to
`src/results/final_test/`, including per-method predictions and scored
outputs, summary tables, paired comparisons, and plots.

## Repository layout

```text
project/
├── pyproject.toml
├── README.md
├── src/
│   ├── data/
│   │   ├── README.md
│   │   ├── raw/
│   │   ├── images/
│   │   ├── crops/
│   │   └── processed/
│   ├── models/
│   ├── notebooks/
│   └── results/
├── proposal/
└── report/
```

Large dataset files, model weights, and generated outputs are runtime
artifacts. They should be kept in the documented directories and should not be
downloaded again unnecessarily.