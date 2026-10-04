# Froth SAM Family

Fine-tuning, evaluation, prediction, and qualitative visualization scripts for SAM, HQ-SAM, and MedSAM on industrial froth imagery.

**Authors:** Sina Lotfi and Reza Dadbin.

## Research Status and Data Availability

This repository forms part of broader froth-image analysis research. Experimental work completed; a data-paper manuscript is currently in preparation.

The research dataset is private/proprietary and is not distributed. Pretrained and fine-tuned checkpoints are also not bundled. The code is provided for research and reproducibility with compatible, independently supplied data and weights; the public repository alone does not reproduce the private experiments.

## Overview

The scripts adapt published SAM-family models to binary froth segmentation. They fine-tune the mask decoder with full-image box prompts while freezing the other model parameters. The underlying SAM, HQ-SAM, and MedSAM architectures and pretrained weights are the work of their original authors.

The repository supplies froth-specific dataset loading, model integration, training/evaluation scripts, and mask visualization. Using MedSAM on this dataset is an industrial-imaging experiment, not evidence of completed medical-imaging research.

## Repository Structure

```text
config.py                       paths, hyperparameters, model selection
sam_froth/data/froth_dataset.py  TIFF/LabelMe loading and mask rasterization
sam_froth/models/                wrappers for published SAM-family models
sam_froth/utils/                 loss and metric helpers
scripts/train.py                decoder fine-tuning
scripts/eval.py                 segmentation metrics
scripts/predict.py              mask visualization export
scripts/amg_demo.py             interactive qualitative comparison
```

Local `data/`, `weights/`, and `outputs/` directories are ignored by Git.

## Environment Setup

Use Python 3.10 or newer and a virtual environment. From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install torch torchvision
python -m pip install numpy Pillow tifffile opencv-python matplotlib tqdm timm
python -m pip install git+https://github.com/facebookresearch/segment-anything.git
python -m pip install segment-anything-hq
```

On Windows, activate with `.venv\Scripts\activate`. Choose a compatible PyTorch/TorchVision build using the [official instructions](https://pytorch.org/get-started/locally/) if CUDA support is required.

The current `requirements.txt` is empty, so installing it alone does not install the dependencies. The commands above follow the code's imports and the upstream [SAM](https://github.com/facebookresearch/segment-anything) and [HQ-SAM](https://github.com/SysCV/sam-hq) installation paths. Both model packages are imported by the shared wrappers, including when selecting a single backend. Check upstream package compatibility and record the versions used for each experiment.

## Data Preparation

Supply paired TIFF images and LabelMe JSON annotations with matching filename stems:

```text
data/
├── train/
│   ├── image_001.tif
│   └── image_001.json
├── test/                       validation during training
│   └── ...
└── eval/                       optional separate evaluation split
    └── ...
```

Use polygons labeled `froth`, or change `Config.label_key`. The dataset loader rasterizes the polygons into binary masks. Training uses `train/` and `test/`; evaluation and prediction choose a nonempty `eval/`, then `test/`, then `train/`. Check the printed split before interpreting metrics. Evaluation on training data is not a held-out result.

## Configuration and Weights

Edit `config.py` for dataset paths, output paths, device, seed, batch size, learning rate, and epochs. The default batch size is one.

Download the appropriate ViT-B checkpoints from the original projects and place them at the configured paths:

```text
weights/sam_vit_b_01ec64.pth
weights/sam_hq_vit_b.pth
weights/medsam_vit_b.pth
```

These are pretrained inputs. Training writes the fine-tuned checkpoints under `outputs/<model>_finetune_out/`.

## Training

From the repository root:

```bash
python -m scripts.train --model sam --epochs 50
```

Replace `sam` with `hqsam` or `medsam` to run another backend. Training prints epoch loss and validation IoU. It writes decoder-only and full-model checkpoints using the tag `<model>_vit_b_decoder_only`:

```text
outputs/sam_finetune_out/
├── sam_vit_b_decoder_only_decoder_best.pth
├── sam_vit_b_decoder_only_decoder_last.pth
├── sam_vit_b_decoder_only_full_best.pth
└── sam_vit_b_decoder_only_full_last.pth
```

The best checkpoint is selected by validation IoU. The script starts a new training run when invoked again; it does not restore optimizer/epoch state automatically. Archive an output directory before repeating an experiment to avoid overwriting checkpoints.

## Evaluation

```bash
python -m scripts.eval --model sam --thr 0.5
```

Evaluation loads the corresponding `*_full_best.pth` checkpoint and prints mean IoU and Dice on the chosen dataset split. Capture terminal output if you need a metrics record; the script does not write `metrics.json`.

## Prediction

```bash
python -m scripts.predict --model sam
```

Prediction loads the best full-model checkpoint and writes sequentially named PNGs such as `mask_0000.png` under:

```text
outputs/pred_masks/sam_vit_b_decoder_only/soft/
```

Each predicted mask is independently min/max normalized to an 8-bit visualization. These PNGs are not calibrated probability maps, and raw float tensors are not exported by the script. The current prediction workflow uses the paired annotation dataset loader.

## Qualitative Visualization

```bash
python -m scripts.amg_demo --model sam --split test --idx 0
```

`--split` accepts `auto`, `train`, `test`, or `eval`; it does not accept an arbitrary folder path. The demo displays the original image, annotation mask, and generated overlay in Matplotlib. The HQ-SAM branch uses a custom point-grid procedure. The script has no automatic headless export path.

## Reproducibility and Limitations

- Record dataset provenance, train/validation/evaluation splits, seed, configuration, checkpoints, package versions, device, and repository commit.
- Seeds are set for Python, NumPy, and PyTorch; identical results across devices or versions are not guaranteed.
- The public workflow assumes binary segmentation and uses frozen encoders. It does not establish broad foundation-model or medical-domain research experience.
- Checkpoint compatibility, private data, and upstream dependencies must be supplied before running experiments. No quantitative performance claim is made in this README.

## Attribution

- [SAM — Meta AI](https://github.com/facebookresearch/segment-anything).
- [HQ-SAM — original project](https://github.com/SysCV/sam-hq).
- [MedSAM — original project](https://github.com/bowang-lab/MedSAM).

Froth-specific workflow and experimentation are credited to Sina Lotfi and Reza Dadbin. Please cite the original model papers when using their methods, and acknowledge this repository when using its froth-specific code. The manuscript in preparation is not a published reference.
