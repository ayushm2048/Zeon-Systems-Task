#  Microcentrifuge Tube Detection & Orientation Estimation

<p align="center">
  <img src="https://img.shields.io/badge/Recall-1.000-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Precision-0.863-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/F1_Score-0.926-blueviolet?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Angle_MAE-2.19°-orange?style=for-the-badge" />
</p>

---

##  Problem Statement

Given a dataset of **70 overhead RGB images (640×480 px)** of microcentrifuge tubes on varied surfaces, the task is to:

1. **Detect** tube positions and orientations in each image.
2. **Evaluate** system performance against ground truth annotations.
3. **Analyze** results and propose next steps.

Each image contains **3–6 tubes** placed on desks, white surfaces, black surfaces, or mixed-color backgrounds. The total dataset covers **371 tube instances**.

---

##  Final Performance

Evaluated on the full original 70-image dataset (371 tubes):

| Metric | Score | Description |
|---|---|---|
| **Recall** | **1.000** | Zero missed tube detections across all 371 instances |
| **Precision** | **0.863** | 59 False Positives (hallucinated tubes) |
| **F1 Score** | **0.926** | Harmonic mean of Precision and Recall |
| **Angle MAE** | **2.19°** | Mean Absolute angular Error — rivals human annotation accuracy |


---

##  Final Approach — 3-Keypoint Pose Estimation

Standard bounding box models struggled due to the **visual similarity between tube hinges and tabs**. The problem was reframed as a **3-Keypoint Estimation Task**.

### Key Design Decisions

| Decision | Rationale |
|---|---|
| **3 keypoints** (joint center + 2 tab points) instead of 2 | Removes 180° ambiguity; defines a direction vector |
| **Artificial 40px radius** for joint-tab labels | Improved localization accuracy during training |
| **YOLOv8s-pose** over YOLOv8n-pose | Larger backbone for better feature extraction |
| **5× dataset augmentation** (70 → 350 images) | Critical to prevent overfitting on a tiny dataset |
| **Trigonometric angle extraction** at inference | Deterministic and rotation-invariant |

### Pipeline Summary

```
Input Image (640×480 RGB)
        │
        ▼
  YOLOv8s-pose (fine-tuned)
        │
        ▼
  3 Keypoints per tube
  [joint_center, tab_pt_1, tab_pt_2]
        │
        ▼
  Angle vector = tab_center - joint_center
        │
        ▼
  θ = atan2(dy, dx)  →  Orientation in [0°, 360°)
```

### Training Configuration

- **Base Model:** `yolov8s-pose` (pre-trained)
- **Epochs:** 150 with early stopping
- **Augmentation (external):** flip, rotate, scale, shift, jitter, Gaussian noise → 1:4 ratio
- **Augmentation (internal YOLO):** enabled during training
- **Labels:** Custom YOLO-pose format generated with bounding box + keypoint transforms

---

## Evolution of Approaches

The final solution was reached iteratively through 5 documented approaches:

| # | Approach | Result | Key Insight |
|---|---|---|---|
| 1 | **EDA** — Annotated ground truths, analyzed distributions | — | Revealed OBB unreliability; confirmed 360° angular variance |
| 2 | **YOLOv8n-pose** (no augmentation, 2 keypoints) | 12.97° MAE | 180° flip ambiguity present; small model underfit |
| 3 | **3-Keypoint Transformation** with circular loss | 5.55° MAE | Unambiguous direction vector; augmentation helps modestly |
| 4 | **Classical CV Hybrid** (YOLO + OpenCV ellipse fit) | 30.52° MAE ❌ | Background variance destroys classical geometry |
| 5 | **Final** — YOLOv8s-pose + rigorous augmentation + 3KP | **2.19° MAE** ✅ | All gains compound; no classical postprocessing needed |

---

##  EDA Highlights

Four key findings from the Exploratory Data Analysis shaped the modeling strategy:

1. **Spatial Constraint (3×4 Grid):** Tubes are always placed inside a centralized rack. Background outside the grid is safe to ignore.
2. **Noisy OBB Annotations:** Ground truth `bbox_rotation` values show human inconsistency — confirms that Oriented Bounding Box regression will fail; keypoints are essential.
3. **Zero Occlusion:** Minimum tube-to-tube distance always exceeds average tube width — aggressive NMS is safe and won't suppress real detections.
4. **Full 360° Rotational Variance:** Uniform angle distribution demands robust rotation/flip augmentations and a circular loss function.

📁 See [`EDA/`](./EDA/) for visual outputs.

---

##  Repository Structure

```
Zeon-Systems-Task/
│
├── Code Files/
│   ├── approach1+2.ipynb       # EDA + Initial YOLOv8n-pose baseline (12.97° MAE)
│   ├── approach3.ipynb         # 3-Keypoint formulation with circular loss (5.55° MAE)
│   ├── approach4.ipynb         # Classical CV hybrid — YOLO + OpenCV ellipse fit (30.52° MAE)
│   ├── approach5(final).ipynb  # Final solution — YOLOv8s-pose + augmentation (2.19° MAE)
│   └── readme.md               # Per-approach technical notes
│
├── EDA/
│   ├── basic_EDA.png           # Tube counts, angle distribution, bbox area spread
│   ├── advanced_EDA.png        # Spatial heatmaps, aspect ratios, HSV distributions
│   └── readme.md               # EDA findings and modeling decisions
│
└── README.md                   
```

---

## Next Steps

| Priority | Proposal | Expected Impact |
|---|---|---|
| 🔴 High | **Confidence Threshold Tuning** — Sweep threshold to find the precision/recall sweet spot | Direct precision improvement |
| 🔴 High | **HRNet Backbone** — Maintain high-resolution feature maps throughout (no downsample-upsample) | Better keypoint localization at low resolutions |
| 🟡Medium | **Template Matching Post-processing** — Classical refinement within YOLO-predicted bounding boxes | Potential sub-degree angle refinement |

---

##  Original Dataset

The dataset (images + annotations) is available here:
🔗 [Google Drive — Dataset](https://drive.google.com/drive/folders/19XdosmtFivQ2mUODFSQgRPGRAqvVsDnI?usp=sharing
)

The `annotations.csv` contains: `image`, `center_x`, `center_y`, `bbox_x`, `bbox_y`, `bbox_w`, `bbox_h`, `bbox_rotation`, `angle_deg` for all 371 tube instances.

---
