## Approaches

### 1. Exploratory Data Analysis (EDA)

* Conducted EDA on the dataset to extract visual clues.
* Annotated all ground truths directly on the dataset images to establish a better visual understanding of the problem and coordinate space.

### 2. Initial Baseline (YOLOv8n-pose)

* **Approach:** Utilized the `yolov8n-pose` model without any data augmentation. Directly targeted the center and tabs as keypoints.
* **Result:** Achieved **12.97 degrees MAE**.
* **Analysis:** Reviewed the angle error distribution and verified that there were no 180-degree flip ambiguities occurring.

### 3. The 3-Keypoint Transformation

* **Approach:** Computed 3 explicit keypoints using standard trigonometry and generated custom YOLO-pose labels for them. Evaluated the predictions using a circular error function.
* **Result:** Successfully reduced the error to **5.55 degrees MAE** on the original dataset and **5.28 degrees MAE** on the augmented dataset.

### 4. Classical CV Hybrid Approach (Failed Experiment)

* **Approach:** Switched to the heavier `yolov8s-pose` model. Attempted a hybrid method utilizing contour extraction, dynamic thresholds, and OpenCV's ellipse-fitting function directly on the YOLO bounding box predictions.
* **Result:** This approach failed miserably, resulting in a **30.52 degrees MAE**.
* **Analysis:** The failure was caused by the high variance in background surfaces and the limited pixel resolution of the images, proving that classical geometry struggles heavily in this environment.

### 5. Final Optimized Approach

* **Approach:** Returned purely to the `yolov8s-pose` model and implemented a rigorous data augmentation pipeline.
* Applied heavy external transformations: scale, shift, rotate, flip, jitter, and noise.
* Bounding boxes and all 3 keypoints were accurately transformed during this augmentation process, utilizing a 1:4 augmentation ratio.
* Generated precise YOLO-pose labels for each augmented image for model fine-tuning.
* Trained the `yolov8s-pose` model for 150 epochs (with early stopping) and applied additional internal YOLO augmentation during training.
* During inference, utilized trigonometry to calculate the final angles from the predicted keypoint vectors.


* **Final Result:** These combined optimizations successfully drove the error down to **2.19 degrees MAE**.  
