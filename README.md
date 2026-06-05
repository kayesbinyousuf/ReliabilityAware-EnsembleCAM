[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kayesbinyousuf/ReliabilityAware-EnsembleCAM/blob/main/ReliabilityAware_EnsembleCAM_KayesBinYousuf.ipynb)

# Reliability-Aware Ensemble-CAM: A Proposed Extension

**Author:** [Kayes Bin Yousuf](https://linkedin.com/in/kayes-bin-yousuf) — ML Researcher | Renewable Energy & Industrial AI | Reviewer @Elsevier

> **Base paper:** Shadman et al. (2025) — *Ensemble-CAM: Explainable Face Presentation Attack Detection*  
> Repository: [`rashikshadman/Ensemble-CAM`](https://github.com/rashikshadman/Ensemble-CAM)

---

## Overview

This repository proposes a **calibration-aware and uncertainty-guided extension** to the Ensemble-CAM face Presentation Attack Detection (PAD) pipeline. The original paper achieves 93.33% accuracy on CelebA-Spoof using DenseNet-161 with Ensemble-CAM explainability, but does not address confidence calibration or uncertainty quantification — both critical for real-world biometric security deployment.

This extension adds post-hoc reliability without retraining the model, contributing to the emerging area of **trustworthy biometric AI**.

> **Note:** Uses simulated DenseNet-161 logits mirroring CelebA-Spoof (Table 1 of paper). Replace with actual model outputs from `rashikshadman/Ensemble-CAM` for real-data validation.

---

## Key Contributions

| # | Contribution | Extends Paper |
|---|---|---|
| 1 | **Temperature Scaling** — post-hoc calibration via NLL grid search, reduces ECE on overconfident softmax scores | Raw softmax overconfidence not addressed in paper |
| 2 | **Conformal Prediction** — uncertainty-aware prediction sets with formal 90% coverage guarantee | Flags ambiguous samples (e.g. subtle 3D mask attacks) for human review |
| 3 | **Uncertainty-Guided Ensemble-CAM** — heatmap intensity scaled by calibrated confidence | Bridges explainability with reliability; suppresses untrustworthy heatmaps |
| 4 | **Retention Comparison** — extends Table 3 of the paper; uncertainty-guided CAM reduces avg confidence drop | Direct comparison to paper's best result (15.43%) |
| 5 | **Distribution Shift Robustness** — ECE and prediction set size across unseen spoof types (OOD) | Critical gap identified in paper's Section 9 |
| 6 | **Computational Overhead Analysis** — temperature scaling + conformal + weighted CAM adds negligible cost | Quantified against Ensemble-CAM's three-CAM generation baseline |
| 7 | **Failure Case Analysis** — honest assessment of when calibrated confidence is high but heatmap is wrong | Proposed mitigation via IoU cross-check with CelebA-Spoof region masks |

---

## Methodology

### Temperature Scaling (Contribution 1)
A single scalar `T` is learned on a held-out validation set by minimizing NLL via grid search over T ∈ [1.0, 8.0]. No retraining required. `T > 1` softens overconfident predictions. ECE (Expected Calibration Error) is used as the calibration metric — lower is better.

**Why this matters for PAD:** A system that says "99% confident this is a spoof" but is wrong 20% of the time is a security liability. Calibrated confidence is trustworthy confidence.

### Conformal Prediction (Contribution 2)
Outputs a prediction *set* rather than a single class, with a distribution-free 90% coverage guarantee (alpha = 0.10). Set size > 1 signals high uncertainty and triggers a human-review flag — directly deployable in biometric pipelines.

### Uncertainty-Guided CAM (Contribution 3)
Ensemble-CAM heatmap intensity is scaled by calibrated confidence:
- **High confidence** → heatmap at full intensity → explanation is trustworthy
- **High uncertainty** (conformal set size > 1) → heatmap suppressed to 40% → visual warning to analyst

### Distribution Shift Robustness (Contribution 5)
ECE and average conformal set size measured across in-distribution domains (`print_attack`, `replay_attack`) and an OOD domain (`3d_mask_unseen`). OOD samples automatically receive larger prediction sets, flagging uncertainty rather than making confident wrong predictions.

---

## Results Summary

| Method | Avg Confidence Drop (Retention) |
|--------|--------------------------------|
| Ensemble-CAM — paper Table 3 | 15.43% |
| Uncertainty-Guided Ensemble-CAM (proposed) | lower (see notebook) |

*Run the notebook for full quantitative results across all 7 contributions.*

---

## How to Run

### Option 1 — Google Colab (no setup required)
Click the **Open in Colab** badge above. No dataset download needed.

### Option 2 — Local

```bash
git clone https://github.com/kayesbinyousuf/ReliabilityAware-EnsembleCAM.git
cd ReliabilityAware-EnsembleCAM
pip install -r requirements.txt
jupyter notebook ReliabilityAware_EnsembleCAM_KayesBinYousuf.ipynb
```

### Dependencies

```
numpy
torch
torchvision
matplotlib
scikit-learn
```

---

## Repository Structure

```
ReliabilityAware-EnsembleCAM/
├── ReliabilityAware_EnsembleCAM_KayesBinYousuf.ipynb   # Main notebook (7 cells)
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Target Venue

Q1 journal — Trustworthy Biometric AI. Next step: validate on real CelebA-Spoof data via `rashikshadman/Ensemble-CAM` pipeline.

---

## Citation

If you use this work, please also cite the base paper:

```
Shadman, R. et al. (2025). Ensemble-CAM: Explainable Face
Presentation Attack Detection. GitHub: rashikshadman/Ensemble-CAM
```

---

## License

[MIT License](LICENSE)
