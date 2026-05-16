# Microcentrifuge Tube Pose Estimator

## Model Summary

| Field | Value |
|---|---|
| **Model ID** | `tube-yolov8s-pose-v2` |
| **Base Model** | `yolov8s-pose` (Ultralytics 8.4.51) |
| **Task** | Object detection + 3-keypoint pose estimation |
| **Input** | RGB image, any resolution (resized to 640×640 internally) |
| **Output** | Per-tube: bounding box, 3 keypoints, confidence, derived angle |
| **Parameters** | 11.4M |
| **GFLOPs** | 29.4 (fused) |
| **Weights file** | `weights/best.pt` (~23 MB) |
| **Best epoch** | 114 / 150 (early stopping, patience=30) |
| **Training time** | ~32 min on Tesla T4 GPU |

---

##  Use

Detecting microcentrifuge tubes in **overhead RGB images** and estimating the orientation angle (joint-to-tab direction) of each tube lid.


---

## Keypoint Schema

The model predicts **3 keypoints per tube** (`kpt_shape=[3, 3]`):

| Index | Keypoint | Description |
|---|---|---|
| `KP0` | Joint center | Center of the hinge joint (angle origin) |
| `KP1` | Tab point 1 | First landmark on the tab |
| `KP2` | Tab point 2 | Second landmark on the tab |

**Angle derivation:**
```
tab_center = (KP1 + KP2) / 2
angle_deg  = atan2(-dy, dx) * 180/π  mod 360
```


---

## Training Details

| Parameter | Value |
|---|---|
| Optimizer | AdamW (lr=0.002) |
| Batch size | 8 |
| Image size | 640×640 |
| Epochs | 150 → stopped at 144 (best @ 114) |
| Augmentation ratio | 1:4 (70 → 350 images) |
| Aug transforms | flip, rotate±180°, scale±20%, shift±10%, HSV jitter, Gaussian noise |
| Box loss weight | 7.5 |
| Pose loss weight | 12.0 |

---

## Evaluation Results

### Final test (original 70 images, 371 tubes)

| Metric | Score |
|---|---|
| Precision | 0.863 |
| Recall | **1.000** |
| F1 Score | 0.926 |
| Angle MAE | **2.19°** |


## Quick Start

```bash
pip install -r requirements.txt
python inference.py --image path/to/image.png --weights weights/best.pt
```

```python
import math
from ultralytics import YOLO

model = YOLO("weights/best.pt")
results = model.predict("image.png", conf=0.45, iou=0.50)

for r in results:
    for kp, box in zip(r.keypoints.xy.cpu().numpy(), r.boxes.xyxy.cpu().numpy()):
        tab_center = (kp[1] + kp[2]) / 2
        dx, dy = tab_center - kp[0]
        angle = math.degrees(math.atan2(-dy, dx)) % 360
        cx, cy = (box[0]+box[2])/2, (box[1]+box[3])/2
        print(f"Tube at ({cx:.0f}, {cy:.0f}) — {angle:.1f}°")
```

---


*Base model: [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
