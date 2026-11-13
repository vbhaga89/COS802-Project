# Project data

This directory contains the local GQA data used by the notebooks. The data
pipeline is implemented in
[Notebook 1](../notebooks/01_dataset_setup.ipynb), which should be run
locally from the project root.

## Required raw annotations

Download the GQA question and scene-graph annotation files from the official
GQA dataset release. 

Questions:

<https://downloads.cs.stanford.edu/nlp/data/gqa/questions1.2.zip>

Scene Graphs:

<https://downloads.cs.stanford.edu/nlp/data/gqa/sceneGraphs.zip>

Extract or copy the following files under
`src/data/raw/gqa/`:

```text
src/data/raw/gqa/
├── questions1.2/
│   ├── train_balanced_questions.json
│   └── val_balanced_questions.json
└── sceneGraphs/
    ├── train_sceneGraphs.json
    └── val_sceneGraphs.json
```

Notebook 1 searches below `src/data/raw/gqa/`, so the files may be nested
under these directory names as shown. Do not rename the files.

## Images

Notebook 1 expects full GQA images in:

```text
src/data/images/<image_id>.jpg
```

The official GQA image archive is:

<https://downloads.cs.stanford.edu/nlp/data/gqa/images.zip>

The notebook defaults to `REUSE_DOWNLOADED_IMAGES_ONLY = True`. This means it
uses images already present in `src/data/images/` and does not download the
entire archive. Set the flag to `False` only when the local image pool is too
small; Notebook 1 will then use the archive to download the missing required
images. This may require substantial disk space and network access.

## Generated directories

Notebook 1 creates these directories if they do not exist:

```text
src/data/
├── raw/gqa/                 # Manually downloaded annotations
├── images/                  # Full GQA images, <image_id>.jpg
├── crops/                   # Generated padded crops, <qa_id>.jpg
└── processed/
    ├── train.csv
    ├── val.csv
    ├── test.csv
    ├── all_gqa.csv
    ├── crop_filter_diagnostics.csv
    └── gqa_dataset_summary.json
```

The generated CSV files contain the question metadata, answer, image ID, and
paths needed by the training and evaluation notebooks. The split is
image-disjoint, so questions associated with the same source image remain in
the same split.

Notebook 3 uses `processed/` and `images/`. Notebook 4 uses those directories
plus `crops/`. The local baseline and final-evaluation notebooks read the
processed splits and full-image paths.

## Recommended setup

From the project root:

1. Install the project dependencies with `uv sync`.
2. Download and place the four annotation files as shown above.
3. Ensure any available full images are in `src/data/images/`.
4. Run Notebook 1 to filter the questions, create crops, and generate the
   train/validation/test files.
5. Upload `processed/`, `images/`, and `crops/` as one Kaggle dataset for
   Notebooks 3 and 4.
