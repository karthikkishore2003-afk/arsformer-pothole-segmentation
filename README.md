# ARSFormer: Transformer-Based Pothole Segmentation with Proxy-Based Continuous Damage Indexing

ARSFormer is a transformer-based computer vision framework for **pixel-level pothole segmentation and road-damage assessment**.

The framework combines a **MiT-B2 transformer encoder**, a **Gated Recurrent Multi-Scale (GRMS) decoder**, boundary-aware and uncertainty-aware auxiliary prediction heads, and a continuous damage-index estimation module.

The system is designed to produce not only a pothole segmentation mask, but also additional information such as **boundary maps, per-pixel uncertainty, confidence information, and a continuous damage index**.

---

## Key Features

* Transformer-based semantic segmentation using a **MiT-B2 encoder**
* Custom **GRMS (Gated Recurrent Multi-Scale) decoder**
* Cross-Scale Affinity Attention (CSAA)
* Boundary-aware auxiliary supervision
* Per-pixel uncertainty estimation
* Continuous `[0, 1]` damage-index estimation
* Cluster-level dataset splitting to prevent cross-split near-duplicate leakage
* Test-time augmentation for the evaluated configurations
* Road-damage visualization and severity dashboard
* Qualitative failure-case analysis

---

## Architecture

The ARSFormer pipeline consists of four main stages:

1. **Input preprocessing**

   * Resize input images to `512 × 512`
   * ImageNet normalization
   * Training-time data augmentation

2. **MiT-B2 Transformer Encoder**

   * Hierarchical multi-scale feature extraction
   * Four feature maps at strides 4, 8, 16, and 32

3. **GRMS Decoder and Prediction Heads**

   * Gated top-down multi-scale fusion
   * ConvGRU-style feature gating
   * Cross-Scale Affinity Attention
   * Segmentation head
   * Boundary head
   * Uncertainty head

4. **Continuous Damage Assessment**

   * Damage area ratio
   * Instance count
   * Boundary irregularity
   * Mean segmentation confidence
   * Mean predictive uncertainty
   * Boundary response

These features are used by a compact regression module to estimate a continuous damage index in the range `[0, 1]`.

---

## Dataset

The experimental dataset contains **1,818 images** collected from multiple sources, including a custom-collected set, Pothole-600, and a Roboflow export.

A perceptual-hash duplicate audit grouped the images into **824 near-duplicate clusters**.

The final evaluation split was performed at the cluster level:

| Split      | Images |
| ---------- | -----: |
| Training   |  1,509 |
| Validation |    309 |
| Total      |  1,818 |

The cluster-level split was designed so that no near-duplicate cluster crossed the training/validation boundary. The split was subsequently re-verified using a duplicate check.

> The dataset itself is not included in this repository.

---

## Experimental Configuration

| Parameter             | Value                 |
| --------------------- | --------------------- |
| Encoder               | MiT-B2                |
| Input resolution      | 512 × 512             |
| Batch size            | 8                     |
| Optimizer             | AdamW                 |
| Decoder learning rate | 4 × 10⁻⁵              |
| Encoder learning rate | 4 × 10⁻⁶              |
| Scheduler             | OneCycleLR            |
| Training epochs       | 60                    |
| GPU                   | NVIDIA A100-SXM4-40GB |
| Framework             | PyTorch 2.7           |
| Transformers          | 4.57.6                |

---

## Results

ARSFormer achieved the strongest validation performance among the evaluated configurations.

| Configuration                     | Decoder  |        IoU |  Mean Dice |
| --------------------------------- | -------- | ---------: | ---------: |
| Standard SegFormer                | MLP      |     78.14% |     86.80% |
| + Boundary Loss + TTA             | MLP      |     74.90% |     84.28% |
| FPN Decoder + Boundary Loss + TTA | FPN      |     78.61% |     86.90% |
| **ARSFormer**                     | **GRMS** | **82.56%** | **89.26%** |

ARSFormer improves validation IoU by **4.42 percentage points** over the standard SegFormer reference configuration.

Pixel-level evaluation on the full validation set produced:

* **Precision:** 0.9389
* **Recall:** 0.7886
* **Micro-averaged F1:** 0.8572

---

## Inference Efficiency

The model was evaluated at `512 × 512` resolution with batch size 1 on an NVIDIA A100-SXM4-40GB.

* **Parameters:** 41.08M
* **Model size:** 156.7 MB
* **Latency:** 15.2 ms
* **Throughput:** 65.8 FPS

---

## Continuous Damage Index

Instead of assigning severity only through fixed geometric thresholds, ARSFormer estimates a continuous damage index in the range:

`0 = lower damage → 1 = higher damage`

The severity module uses segmentation-derived features together with confidence, uncertainty, and boundary information.

The regressor achieved:

* **Spearman correlation:** ρ = 0.757
* **p < 0.0001**

This correlation is measured against the **constructed proxy target** used for the severity module. It should therefore be interpreted as consistency with the proxy target rather than as validation against independently labelled real-world severity.

---

## Qualitative Results

The repository contains examples of:

* Input road images
* Ground-truth pothole masks
* ARSFormer predicted masks
* Failure cases
* Confidence maps
* Uncertainty maps
* Boundary maps
* Severity dashboard outputs
* Training and ablation curves
* Severity-regressor calibration

---

## Failure Cases

The lowest-IoU validation examples mainly involve:

* Low-contrast potholes
* Uneven illumination
* Visually cluttered pavement
* Difficult boundary separation between potholes and surrounding road texture

These examples are included to illustrate the limitations of the current model.

---

## Repository Contents

```text
arsformer-pothole-segmentation/
│
├── ARSFormer_DG_Notebook.ipynb
│
├── architecture.png
├── training_curves.png
├── segmentation_results.png
├── severity_dashboard.png
├── calibration.png
├── failure_cases.png
├── data_leakage_correction.png
│
└── README.md
```

*The exact filenames may be adjusted to match the files uploaded to this repository.*

---

## Limitations

The current study has several limitations:

* Evaluation uses a single fixed cluster-level train/validation split.
* Near-duplicate images remain within individual splits even though cross-split leakage was eliminated.
* The effective number of distinct underlying photographs is therefore smaller than the nominal image count.
* The severity index is based on segmentation-derived proxy features rather than independently labelled expert severity ground truth.
* The severity correlation should not be interpreted as agreement with human severity assessment.
* Evaluation on additional independently sourced pavement datasets is needed to further assess generalisation.

---

## Paper

This repository accompanies the research paper:

**"ARSFormer: Transformer-Based Pothole Segmentation with Proxy-Based Continuous Damage Indexing"**

**Authors:**
Karthik K. Kishore
Jyothisha J. Nair

School of Computer Sciences
Mahatma Gandhi University, Kottayam, Kerala, India

---

## Citation

If you use this work, please cite the associated paper.

```text
K. K. Kishore and J. J. Nair,
"ARSFormer: Transformer-Based Pothole Segmentation with Proxy-Based Continuous Damage Indexing."
```

---

## Disclaimer

The continuous damage index is a **proxy-based research output** and is not a clinically or physically validated measurement of real-world pavement severity. Further evaluation with independently labelled severity data is required before using the index for operational road-maintenance decisions.
