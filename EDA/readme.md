# Exploratory Data Analysis (EDA)

This folder contains the scripts and visual outputs for analyzing the 70-image microcentrifuge tube dataset. The goal of this EDA is to validate annotation quality, understand spatial distributions, and define physical constraints before training a model.

##  Files Included
* `basic_EDA.png`: Statistical overview showing tube counts per image, the $0-360^\circ$ angle distribution, and bounding box area spread.
* `advanced_EDA.png`: Deep-dive analysis featuring spatial heatmaps, bounding box aspect ratios, inter-object distance (crowding), and HSV brightness distributions.

##  Key Findings & Modeling Decisions

1. **The 3x4 Grid Constraint (Spatial Heatmap):** Tubes are never randomly scattered; they are strictly placed within a centralized 3x4 rack. *Takeaway: Background noise outside the grid can be safely cropped or ignored.*
2. **Noisy Ground Truth Boxes (Aspect Ratio Plot):** Bounding box widths and heights scatter inconsistently, revealing human-annotation errors in the provided `bbox_rotation`. *Takeaway: Relying on Oriented Bounding Boxes (OBB) will fail. We must use Keypoint Detection instead.*
3. **Zero Occlusion (Inter-Object Distance):** The minimum distance between tubes is consistently larger than the average tube width. *Takeaway: Aggressive Non-Maximum Suppression (NMS) can be used without accidentally deleting adjacent tubes.*
4. **Full Rotational Variance:** Angles evenly cover the full $360^\circ$ spectrum. *Takeaway: The model requires robust geometric augmentations (rotations/flips) and a circular loss function to handle angular wrap-around.*
