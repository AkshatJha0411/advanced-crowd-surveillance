# Crowd Surveillance System using YOLOv8 & BoT-SORT

**Authors:**  Akshat Jha (B23336)  & Harsh Vardhan Sharma (B23373)

**Institute:** Indian Institute of Technology (IIT) Mandi  

**Course:** AR–522 Robot Vision

**Professor:** Dr. Praful Hambarde

-----
Our model on a CCTV footage:
<img width="945" height="533" alt="Screenshot 2025-12-10 at 1 20 03 PM" src="https://github.com/user-attachments/assets/f4c6c831-6469-4612-9656-c23c93b7e0a5" />

## Overview

This project implements an automated **Crowd Surveillance System** designed for extremely dense pedestrian environments. By fine-tuning **YOLOv8s** on the **MOT20 dataset** and integrating it with **BoT-SORT** (Box Observation Tracking with Camera Motion Compensation), we achieve robust detection and tracking even under heavy occlusion.

Unlike standard detectors that fail in high-density scenarios, this system is hyper-specialized for crowd analytics, capable of generating heatmaps, flow trajectories, and dwell-time statistics from raw CCTV footage.

### Key Capabilities

  * **Dense Crowd Detection:** Optimized for scenes with 100+ pedestrians per frame.
  * **Robust Tracking:** Utilizes BoT-SORT to maintain IDs across camera movements and occlusions.
  * **Automated Analytics:** Generates occupancy counts, dwell times, and density heatmaps automatically.
  * **Privacy-Aware:** Focuses on full-body patterns rather than facial recognition.

-----

## Project Evolution

### Mid-Term Approach: Head Detection

Initially, we trained a **YOLOv8 Nano** model on the **JHU-Crowd++** dataset to detect heads.

  * **Limitation:** Head bounding boxes are too small and unstable for motion-based trackers (IoU often drops to zero with slight movement).
  * **Pivot:** We switched to **Body Detection** using **MOT20**. Bodies offer larger surface areas, ensuring stable Intersection-over-Union (IoU) for the Kalman filters used in tracking.

-----

## Dataset: MOT20

We utilized the **MOT20** benchmark, known for its extreme density (avg. 149 pedestrians/frame) and challenging indoor/outdoor scenarios.

  * **Training Sequences:** MOT20-01, MOT20-02, MOT20-03, MOT20-05
  * **Validation Sequence:** MOT20-04
  * **Testing Sequences:** MOT20-06, MOT20-07, MOT20-08

### Preprocessing

We developed a custom script to convert MOTChallenge format (`gt.txt`) into YOLO format:

1.  Filtered detections with confidence \< 0.5.
2.  Normalized coordinates (x\_center, y\_center, width, height) to the [0,1] range.

-----

## Methodology & Pipelines

### 1\. Training Pipeline

1.  **Ingestion:** Load MOT20 frames.
2.  **Fine-tuning:** Train **YOLOv8s** (Small) pretrained on COCO.
      * *Config:* Image Size 640, Batch 4, Epochs 50, Optimizer AdamW.
      * *Augmentations:* Blur, MedianBlur (for motion/noise), ToGray (for low light), CLAHE (for contrast).
3.  **Export:** Save weights as `.pt` (PyTorch) and `.onnx` (Deployment).

### 2\. Testing & Analysis Pipeline

1.  **Input:** Inject novel test video (CCTV/Drone/Phone).
2.  **Inference:** Run detection + BoT-SORT tracking.
3.  **Spatial Analytics:** Generate density heatmaps and velocity flow fields.
4.  **Temporal Analytics:** Calculate dwell time per ID and occupancy per frame.
5.  **Event Logic:** Monitor zone entries/exits (ROI).
6.  **Visualization:** Render annotated video with bounding boxes and trajectory trails.

-----

## Results & Performance

The model was validated on the MOT20-04 sequence after 50 epochs.

  * **mAP @ 50:** 0.982 (Extremely high accuracy for detecting pedestrians)
  * **mAP @ 50-95:** 0.837 (Strong bounding box precision even at strict thresholds)
  * **Precision:** 0.986 (Very few false positives/ghost detections)
  * **Recall:** 0.956 (Detects 95.6% of people even in dense crowds)

-----

## Installation & Usage

### One-Click Demo (Recommended)

We provide a Google Colab notebook that automates the entire testing pipeline. You can access the notebook and model weights via the link below:

**[Project Drive Folder](https://drive.google.com/drive/folders/1Yyp4zEeY9DNc1jLntIMFXt5cwKD3GfNS?usp=share_link)**

**Instructions:**

1.  Open the file `robot_vision_crowd_surveillance_demo_1.ipynb` in Google Colab.
2.  Execute all the cells. (Use run all button)
3.  Upload your test video when prompted in Cell 4.
4.  Download the processed output from the `output_videos` folder and view analytics in the cells below.


-----

## Future Scope

  * **Re-Identification (ReID):** Integrate DeepSORT to use visual appearance features, reducing ID switches after long occlusions.
  * **Crowd Counting via Regression:** For densities where boxes are impossible to separate, switch to density map regression (CSRNet).
  * **Anomaly Detection:** Implement logic to detect sudden running or dispersion events.
