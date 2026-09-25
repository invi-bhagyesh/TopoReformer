# TopoReformer

### Mitigating Adversarial Attacks Using Topological Purification in OCR Models

**AAAI 2026 · AI for Cyber Security (AICS) Workshop**

> Bhagyesh Kumar†, [A S Aravinthakashan](https://www.linkedin.com/in/aravinthakshan/)†, [Akshat Satyanarayan](https://www.linkedin.com/in/akshat-satyanarayan-111410378/?originalSubdomain=in)†, [Ishaan Gakhar](https://scholar.google.com/citations?user=z1DCdjAAAAAJ), [Ujjwal Verma](https://scholar.google.com/citations?user=XSzIFIgAAAAJ&hl=en)  

† Equal contribution.

[![arXiv](https://img.shields.io/badge/arXiv-2511.15807-b31b1b?style=flat-square)](https://arxiv.org/abs/2511.15807)
[![PDF](https://img.shields.io/badge/PDF-Read%20paper-555555?style=flat-square)](https://arxiv.org/pdf/2511.15807)
[![Paper page](https://huggingface.co/datasets/huggingface/badges/resolve/main/paper-page-sm.svg)](https://huggingface.co/papers/2511.15807)

TopoReformer studies a purification step before recognition: reconstruct an input while preserving topological relationships between samples and their latent representations. It is trained on unperturbed data and evaluated against classical attacks, adaptive attacks, and the OCR watermark attack FAWA.

## How it works

![TopoReformer pipeline from Figure 1 of the paper](assets/topo.png)

*Figure 1: an illustrative pipeline schematic, not exact layer specifications. Source: [TopoReformer paper](https://arxiv.org/pdf/2511.15807).*

1. **Topological Autoencoder:** learn a reconstruction while encouraging persistent-homology relationships to agree between input and latent spaces.
2. **Reformer and Auxiliary path:** feed the reconstruction to a variational Reformer; project the topological latent representation into its bottleneck through an Auxiliary module.
3. **Freeze–flow warm-up:** initially freeze the Reformer encoder to encourage use of the Auxiliary path, then refine jointly. Keep the downstream classifier frozen.

The topology loss compares selected pairwise distances between samples; it is not simply a count of holes in each letter.

**OCR uses a lighter configuration.** In the FAWA experiments, the Reformer module is omitted for efficiency. Word images are decomposed into character crops, purified, and recombined. Do not treat Table 3 as a test of the identical full pipeline used for the classification experiments.

## Paper results

Values below are percentages transcribed from [arXiv:2511.15807v1](https://arxiv.org/pdf/2511.15807), Tables 1–3. These are reported experiments, not a new evaluation of this checkout. ASR is attack success rate (lower is better); F1, precision, and character accuracy are higher-is-better metrics.

### Classical attacks · Table 1

Selected higher-strength settings, comparing no defense with the full **TopoAE + Reformer + Auxiliary + warm-up** configuration. Each cell is **F1 / precision**. The paper also includes lower-strength settings and intermediate ablations.

| Attack | Dataset | No defense | Full pipeline with warm-up |
| --- | --- | ---: | ---: |
| C&W, c = 10 | MNIST | 4.30 / 6.78 | 75.15 / 77.31 |
| C&W, c = 10 | EMNIST | 33.85 / 50.99 | 68.82 / 72.51 |
| PGD, ε = 0.01 | MNIST | 96.62 / 96.69 | 97.62 / 97.64 |
| PGD, ε = 0.01 | EMNIST | 72.66 / 74.54 | 83.79 / 84.43 |
| FGSM, ε = 0.01 | MNIST | 96.61 / 96.65 | 97.51 / 97.71 |
| FGSM, ε = 0.01 | EMNIST | 72.59 / 73.83 | 84.42 / 85.00 |

C&W uses a confidence parameter `c`; FGSM and PGD use perturbation budget `ε`. Improvements are not monotonic across all ablations: on EMNIST under PGD/FGSM, the Reformer stage can outperform the final warm-up configuration.

### Adaptive attacks · Table 2

EOT evaluates transformations/randomness; BPDA approximates the backward pass through the defense. Each cell is **ASR ↓ / F1 ↑ / precision ↑**.

| Attack | Dataset | No defense | TopoReformer |
| --- | --- | ---: | ---: |
| EOT | MNIST | 99.05 / 3.38 / 5.21 | 9.19 / 90.73 / 91.07 |
| EOT | EMNIST | 97.73 / 1.42 / 2.67 | 28.32 / 73.28 / 75.12 |
| EOT + BPDA | MNIST | 99.70 / 3.21 / 4.89 | 36.59 / 64.71 / 66.65 |
| EOT + BPDA | EMNIST | 98.69 / 1.36 / 2.54 | 44.26 / 58.92 / 61.31 |
| BPDA | MNIST | 99.60 / 4.44 / 7.28 | 81.14 / 15.65 / 40.98 |
| BPDA | EMNIST | 95.30 / 2.03 / 5.17 | 84.46 / 12.77 / 35.42 |

The EOT result is strong, but BPDA alone remains a substantial weakness: defended ASR is 81.14% on MNIST and 84.46% on EMNIST. Robustness depends on the evaluated attack and threat model.

### OCR under FAWA · Table 3

Each cell is **ASR ↓ / character accuracy ↑ / character precision ↑**. Character accuracy is not whole-word accuracy.

| Recognizer | Decoder | No defense | Topological purification |
| --- | --- | ---: | ---: |
| CRNN | CTC | 100.00 / 48.13 / 45.69 | 78.83 / 71.00 / 71.61 |
| Rosetta | CTC | 99.83 / 69.66 / 62.94 | 44.08 / 85.98 / 87.52 |
| STAR-Net | CTC | 98.92 / 74.52 / 69.77 | 65.17 / 79.81 / 81.44 |
| RARE | Attention | 99.92 / 51.25 / 49.77 | 87.67 / 65.18 / 64.74 |
| TRBA | Attention | 99.83 / 46.68 / 44.26 | 60.75 / 80.26 / 80.81 |

For example, Rosetta’s ASR decreases by **55.75 percentage points**, while TRBA’s character accuracy increases by **33.58 percentage points**. These are absolute differences calculated from Table 3, not relative percentage improvements.

## Getting started

The repository’s default implementation branch is **`aravinth`**.

```bash
git clone https://github.com/invi-bhagyesh/TopoReformer.git
cd TopoReformer
git switch aravinth
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Dependencies are not fully pinned. Use a PyTorch installation appropriate for your hardware. Training and attack generation require datasets and compatible model checkpoints; they have not been rerun for this documentation update.

### Train the Topological Autoencoder

Configurations are provided for `MNIST`, `EMNIST`, and `SYN`:

```bash
topo_dataset="MNIST"
python -m exp.train_model -F test_runs with \
  "experiments/train_model/best_runs/${topo_dataset}/TopoRegEdgeSymmetric.json" \
  device='cuda' evaluation.save_training_latents=True
```

Check the selected configuration’s dataset paths and device before running.

### OCR / FAWA evaluation

[Pretrained OCR recognizer weights](https://huggingface.co/datasets/invi-bhagyesh/ocr/tree/main/models) are available on Hugging Face. These are recognizer weights, not a claim that every defense checkpoint is included.

The example below uses the CRNN configuration. Replace the checkpoint and dataset paths with your local resources. For another recognizer, match its transformation, feature extractor, sequence model, and prediction head.

```bash
str_model_path="/path/to/CRNN_VGG_BiLSTM_CTC_model.pth"
output_path="./fawa-output"
python watermark/baselines/fawa.py \
  --root data/protego/test \
  --save_attacks "$output_path" \
  --iter_num 2000 \
  --eps 0.157 \
  --alpha 0.05 \
  --str_model "$str_model_path" \
  --Transformation None \
  --FeatureExtraction VGG \
  --SequenceModeling BiLSTM \
  --Prediction CTC
```

### Adaptive evaluation

The combined attack script supports `MNIST` and `EMNIST`. Supply a compatible full-pipeline checkpoint explicitly; the script’s default path refers to the original Kaggle environment. It also imports `torchattacks`, which is not listed in `requirements.txt`.

```bash
python -m pip install torchattacks
python -m scripts.combined.attack \
  --attack bpda_eot \
  --dataset MNIST \
  --full_pipeline_path /path/to/MNIST_full_pipeline.pth \
  --device cuda \
  --output ./mnist-bpda-eot.npz
```

Consult `scripts/combined/attack.py` for attack-specific budgets, iteration counts, and smoothing options. The example command uses script defaults; it is not a complete reproduction recipe for every table entry.

## Citation

```bibtex
@article{kumar2025toporeformer,
  title = {TopoReformer: Mitigating Adversarial Attacks Using Topological Purification in OCR Models},
  author = {Bhagyesh Kumar and A S Aravinthakashan and Akshat Satyanarayan and Ishaan Gakhar and Ujjwal Verma},
  journal = {arXiv preprint arXiv:2511.15807},
  year = {2025},
  url = {https://arxiv.org/abs/2511.15807}
}
```

## Acknowledgements

This repository builds on [ProTegO](https://github.com/Ruby-He/ProTegO) and [Topological Autoencoders](https://github.com/BorgwardtLab/topological-autoencoders). The pretrained OCR recognizers are provided by ProTegO. Please retain and follow the respective upstream licenses, model terms, and dataset requirements.
