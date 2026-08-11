# Sensor Fusion for Vehicle Detection & Distance Estimation

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![YOLOv5](https://img.shields.io/badge/YOLOv5-Object%20Detection-brightgreen.svg)](https://github.com/ultralytics/yolov5)
[![Dataset](https://img.shields.io/badge/Dataset-Astyx%20HiRes2019-orange.svg)](https://www.astyx.com/)

A Sensor Fusion pipeline developed for autonomous vehicle perception. This project integrates 2D Object Detection (**YOLOv5**) with **LIDAR** and **RADAR** point clouds to detect vehicles in the **Astyx HiRes2019 dataset** and accurately estimate their physical distances using weighted sensor fusion and spatial transformations.

---

##  Features

- **Multi-Sensor Data Processing**: Handles JSON calibration matrices, 3D point cloud data from LIDAR, and high-resolution RADAR targets.
- **Coordinate Transformations**: Converts raw sensor data to a unified Reference Coordinate System and projects 3D spatial points onto 2D camera image planes.
- **YOLOv5 Vehicle Filtering**: Filters bounding boxes specifically for vehicle classes (`class 2: car`, `class 5: bus`, `class 7: truck`).
- **3D Spatial Bounding Box Masking**: Filters point clouds by isolating LIDAR and RADAR points falling within detected vehicle bounding boxes.
- **Weighted Distance Fusion**: Combines LIDAR depth measurements with RADAR range data using configurable confidence weights (default: `0.7` LIDAR / `0.3` RADAR).
- **Error Evaluation & Analytics**: Calculates Mean Absolute Error (MAE) and Root Mean Square Error (RMSE) against ground truth annotations across dataset frames.

---

##  Architecture & Pipeline

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│   Camera Image  │       │  LIDAR Points   │       │   RADAR Points  │
└────────┬────────┘       └────────┬────────┘       └────────┬────────┘
         │                         │                         │
         v                         v                         v
  [ YOLOv5 Detection ]     [ Sensor Calibration ]    [ Sensor Calibration ]
  (Vehicle Classes)        (Inverse Matrix xform)    (Inverse Matrix xform)
         │                         │                         │
         └─────────────┬───────────┴─────────────────────────┘
                       │
                       v
         [ 2D/3D Point Cloud Masking ]
         (Filter points in BBoxes)
                       │
                       v
         [ Weighted Sensor Fusion ]
         (0.7 LIDAR + 0.3 RADAR)
                       │
                       v
         [ Distance & Error Evaluation ]
         (MAE / RMSE vs Ground Truth)
```

---

##  Methodology

### 1. Calibration & Inverse Transformations (`calib_astyx`)
- Sensor calibration matrices (intrinsic and extrinsic parameters) are parsed from dataset JSON files.
- Computes inverse transformation matrices to project 3D points ($X, Y, Z$) between sensor-specific coordinate frames and the unified reference frame.

### 2. Camera Projection & Masking
- 3D spatial points are projected onto the camera matrix $K [R | T]$.
- An image mask removes points located outside the field of view (FOV) or behind the camera z-plane.

### 3. Point Filtering
- YOLOv5 detects vehicle bounding boxes $[x_{min}, y_{min}, x_{max}, y_{max}]$.
- Projected LIDAR and RADAR points falling inside these bounding boxes are isolated to derive object-specific depth profiles.

### 4. Distance Fusion
Fused distance ($D_{fused}$) is calculated as a weighted linear combination:

$$D_{fused} = w_{lidar} \cdot D_{lidar} + w_{radar} \cdot D_{radar}$$

*Default parameters*:
- **LIDAR Weight ($w_{lidar}$)**: `0.7` (Higher spatial accuracy and point density)
- **RADAR Weight ($w_{radar}$)**: `0.3` (Robust velocity tracking and material penetration)

---

## 📈 Evaluation & Results

The distance estimation logic was evaluated across **110 test frames** against ground-truth dataset annotations:

| Metric | Average Value | Notes |
| :--- | :--- | :--- |
| **MAE** | 10.0 – 20.0 m | Baseline range estimation accuracy |
| **RMSE** | 10.0 – 20.0 m | Penalizes large outlier deviations |
| **Peak Error** | ~50.0 m | Occurs primarily during object detection misses (Frame 33) |

> **Sample Frame Result (Frame #131)**:
> - **LIDAR Distance**: `28.65 m`
> - **RADAR Distance**: `31.88 m`
> - **Fused Distance**: `29.62 m`

---

##  Getting Started

### Prerequisites

- Python 3.8+
- PyTorch 1.8+
- OpenCV
- NumPy, Matplotlib, SciPy



## 🔮 Future Improvements

- **Dynamic Weighting**: Replace fixed weights ($0.7/0.3$) with dynamic uncertainty estimates based on environmental conditions (e.g., fog, rain, low light).
- **NaN Handling**: Implement Kalman Filtering or object tracking to bridge frames where YOLO fails to detect vehicles.
- **3D Bounding Box Estimation**: Transition from 2D image projections to full 3D bounding box estimation (e.g., PointRCNN / Frustum PointNets).

---

## 👤 Author

**Dhruv Sunilkumar Joshi**  
