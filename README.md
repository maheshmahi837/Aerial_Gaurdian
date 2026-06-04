# Aerial Guardian: Drone-Based Person Detection and Tracking

## Overview

This project implements a complete aerial multi-object tracking (MOT) pipeline for drone surveillance using the VisDrone2019-MOT dataset.

The objective is to detect and track pedestrians from a moving drone while maintaining consistent identities despite camera motion, scale variation, partial occlusions, and crowded scenes.

The solution combines a fine-tuned YOLO11m detector with modern tracking algorithms (ByteTrack and BoT-SORT) and includes edge-deployment support through ONNX export.

---

## Key Features

* Fine-tuned YOLO11m detector for aerial pedestrian detection
* Single-class person detection
* ByteTrack tracking
* BoT-SORT tracking
* SAHI sliced inference for small object detection
* Trajectory visualization
* FPS benchmarking
* ONNX export for deployment
* Edge-device ready
* Lightweight model (< 300 MB requirement)

---

## Dataset

Dataset: VisDrone2019-MOT

Task: Multi-Object Tracking

Target Class:

* Pedestrian
* People

Both classes were merged into a single class:

```text
person
```

### Dataset Statistics

| Split      | Images | Bounding Boxes |
| ---------- | ------ | -------------- |
| Train      | 23,476 | 328,701        |
| Validation | 2,846  | 50,312         |

---

## Detection Architecture

### Base Detector

YOLO11m

### Training Configuration

| Parameter  | Value       |
| ---------- | ----------- |
| Input Size | 1152 × 1152 |
| Epochs     | 25          |
| Batch Size | 8           |
| Optimizer  | AdamW       |
| Pretrained | Yes         |
| Classes    | Person      |

### Why YOLO11m?

YOLO11m provides a strong balance between:

* Detection accuracy
* Inference speed
* Model size

This makes it suitable for real-time aerial surveillance and edge deployment.

---

## Small Object Detection Strategy

Detecting pedestrians from drone imagery is challenging because targets often occupy only a few pixels.

The following improvements were introduced:

### 1. High Resolution Training

Training was performed at:

```text
1152 × 1152
```

instead of conventional:

```text
640 × 640
```

This preserves small-object details and improves recall.

### 2. SAHI (Sliced Aided Hyper Inference)

SAHI divides large aerial images into overlapping patches before inference.

Benefits:

* Better detection of tiny pedestrians
* Higher recall
* Reduced missed detections

---

## Tracking Architecture

Two tracking approaches were evaluated.

### ByteTrack

Advantages:

* Lightweight
* High speed
* Real-time capable

### BoT-SORT

Advantages:

* Global Motion Compensation
* Improved robustness to drone motion
* Reduced ID switching

### Handling Drone Ego-Motion

Drone footage introduces significant camera motion.

To reduce identity switches:

* BoT-SORT Global Motion Compensation was used
* Temporal trajectory consistency was maintained
* Tracking thresholds were tuned specifically for aerial scenes

These modifications improve ID stability when the camera moves rapidly or pedestrians become partially occluded.

---

## Trajectory Visualization

Each tracked object contains:

* Bounding Box
* Unique ID
* Motion Tail

The trajectory tail provides a visual representation of movement history.

---

## Experimental Results

### Detection Performance

| Metric    | Score |
| --------- | ----- |
| Precision | 0.663 |
| Recall    | 0.609 |
| mAP50     | 0.608 |
| mAP50-95  | 0.250 |

Best validation performance occurred during early epochs before overfitting.

---

## Tracking Performance

### Standard YOLO11m + ByteTrack

Hardware:

```text
Tesla T4 GPU
```

| Metric     | Value |
| ---------- | ----- |
| Mean FPS   | 21.4  |
| Median FPS | 21.5  |
| Min FPS    | 1.3   |
| Max FPS    | 22.9  |

### SAHI + ByteTrack

Hardware:

```text
Tesla T4 GPU
```

| Metric     | Value |
| ---------- | ----- |
| Mean FPS   | 1.6   |
| Median FPS | 1.6   |
| Min FPS    | 0.6   |
| Max FPS    | 1.7   |

---

## Engineering Trade-Offs

### Faster Pipeline

YOLO11m + ByteTrack

Advantages:

* Real-time performance
* ~21 FPS
* Suitable for deployment

### Higher Recall Pipeline

YOLO11m + SAHI + ByteTrack

Advantages:

* Better small-object detection
* Improved recall

Disadvantages:

* Significant FPS reduction

For practical drone deployment, the standard YOLO11m + ByteTrack configuration is recommended.

---

## Model Size

### PyTorch

```text
best_visdrone_person_yolo11m.pt
```

Size:

```text
40.6 MB
```

### ONNX

```text
best_visdrone_person_yolo11m.onnx
```

Size:

```text
76.70 MB
```

Below 300 MB requirement.

---

## Edge Deployment

### ONNX Export

The trained model can be exported to ONNX for deployment on:

* NVIDIA Jetson Nano
* NVIDIA Jetson Xavier
* NVIDIA Jetson Orin
* Other edge devices

### Deployment Pipeline

```text
Drone Camera
      ↓
YOLO11m Detector
      ↓
ByteTrack / BoT-SORT
      ↓
Trajectory Visualization
      ↓
Output Video Stream
```

### Future Optimization

For Jetson deployment:

* TensorRT conversion
* FP16 inference
* Dynamic batching
* Quantization

These optimizations can significantly improve FPS while reducing power consumption.

---

## Repository Structure

```text
.
├── README.md
├── notebooks
│   ├── Aerial_Guardian_part1.ipynb
│   ├── Aerial_Guardian_part2.ipynb
│
├── configs
|   ├── args.yaml
│   ├── bytetrack_drone.yaml
│   ├── botsort_drone.yaml
│
├── outputs
│   ├── tracked_video.mp4
│
└── results
    
```

---

## Conclusion

This project demonstrates a lightweight aerial surveillance pipeline capable of detecting and tracking pedestrians from moving drone footage.

The final solution balances accuracy, speed, and deployability while satisfying the assignment requirements for:

* Detection
* Tracking
* FPS benchmarking
* Edge deployment readiness
* Model size constraints
