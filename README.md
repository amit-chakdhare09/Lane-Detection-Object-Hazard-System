# 🚗 Lane Detection & Object Hazard System

> Real-time lane detection with YOLOv8 object recognition for autonomous driving assistance — built and tested on Mumbai Coastal Road footage.

## 📌 Overview

This project combines **classical computer vision** (Canny edge detection + Hough Transform) with **deep learning** (YOLOv8) to build a lane-aware driving assistant system. It processes dashcam video and outputs:

- ✅ Detected and filled lane overlay (green polygon)
- ✅ Real-time object detection (cars, trucks, buses, persons)
- ✅ Steering command: `STRAIGHT`, `LEFT`, `RIGHT`, or `STOP`
- ✅ Steering angle estimate

---

## 🧠 How It Works

```
Input Video
    │
    ▼
┌─────────────────────┐
│  Canny Edge + ROI   │  ← Grayscale → Gaussian Blur → Canny → Masked Region
└────────┬────────────┘
         │
    ▼
┌─────────────────────┐
│   Hough Transform   │  ← Detects left/right lane lines by slope
└────────┬────────────┘
         │
    ▼
┌─────────────────────┐
│  Lane Fill Polygon  │  ← Fills detected lane with green overlay (0.3 alpha)
└────────┬────────────┘
         │
    ▼
┌─────────────────────┐
│  YOLOv8 Detection   │  ← Detects nearby hazards in lower 40% of frame
└────────┬────────────┘
         │
    ▼
┌─────────────────────┐
│  Command Decision   │  ← Deviation from center → LEFT/RIGHT/STRAIGHT/STOP
└─────────────────────┘
         │
    ▼
Output Annotated Video
```

---

## 📁 Repository Structure

```
lane-detection/
│
├── README.md                    # You are here
├── LICENSE                      # MIT License
├── requirements.txt             # Python dependencies
├── .gitignore                   # Git ignore rules
│
├── lane-detection.ipynb         # 📓 Main Kaggle notebook (all-in-one)
│
└── docs/
    └── pipeline.md              # Detailed pipeline explanation
```

---

## 🛠️ Installation

### Prerequisites
- Python 3.10+
- GPU recommended (NVIDIA, CUDA 11.8+)
- [Kaggle account](https://kaggle.com) (to run the notebook as-is)

### Local Setup

```bash
# Clone the repo
git clone https://github.com/yourusername/lane-detection.git
cd lane-detection

# Install dependencies
pip install -r requirements.txt
```

---

## 🚀 Usage

### Option 1: Run on Kaggle (Recommended)

1. Open [Kaggle](https://www.kaggle.com) and create a new notebook
2. Upload `lane-detection.ipynb`
3. Add the dataset: **Mumbai Coastal Road Video Sample** (`amitchakdhare/mumbai-coastal-road-video-sample`)
4. Enable **GPU (T4)** accelerator
5. Run all cells → output saved at `/kaggle/working/output.mp4`

### Option 2: Run Locally

```bash
# Update DATASET_PATH in the notebook, or run the modular script:
python src/video_processor.py --input /path/to/your/video.mp4 --output output.mp4
```

---

## ⚙️ Configuration

| Parameter | Default | Description |
|---|---|---|
| `video_speed_scale` | `2.0` | Frame skip multiplier (higher = faster processing) |
| `conf` (YOLO) | `0.5` | Minimum detection confidence |
| Hazard zone | `y > h*0.6` | Lower 40% of frame triggers hazard check |
| Hazard size | `box_height > h*0.25` | Box must be >25% frame height to trigger STOP |
| Deviation threshold | `30 px` | Lane center offset before issuing LEFT/RIGHT |
| Smoothing factor | `0.7 / 0.3` | EMA weights for lane center smoothing |

---

## 🧩 Core Modules

### `detect_lanes_and_fill(frame)`
```
1. Convert to grayscale → Gaussian Blur (5×5) → Canny (50, 150)
2. Apply rectangular ROI mask (bottom 40% of frame)
3. Hough Line Transform (threshold=50, minLen=40, maxGap=150)
4. Classify lines: slope < -0.5 → left lane, slope > 0.5 → right lane
5. Fit convex polygon from extreme points → blend overlay at 30% opacity
6. Return annotated frame + lane center x-coordinate
```

### `YOLOv8 Hazard Detection`
```
Model  : yolov8n.pt (nano — fast inference)
Classes: car, person, truck, bus
Trigger: Object in lower 40% of frame AND height > 25% of frame
Result : Sets hazard=True → overrides command to "STOP"
```

### `Decision Engine`
```python
if hazard:          → "STOP"
elif |deviation| < 30:  → "STRAIGHT"
elif deviation > 0:     → "RIGHT"
else:               → "LEFT"

steering_angle = deviation / frame_width * 45  # degrees
```

---

## 📊 Output Annotations

| Annotation | Color | Description |
|---|---|---|
| Lane polygon | 🟢 Green (30% alpha) | Detected drivable lane |
| Object boxes | 🔵 Blue | YOLOv8 bounding boxes |
| Center line | 🔴 Red | Frame center reference |
| Lane dot | 🟡 Yellow | Estimated lane center |
| `CMD:` text | ⚪ White | Current driving command |
| `ANGLE:` text | 🩵 Cyan | Estimated steering angle |

---

## 🔬 Model Details

| Property | Value |
|---|---|
| Detection model | `yolov8n.pt` (YOLOv8 Nano) |
| Framework | Ultralytics |
| Lane method | Hough Transform (classical CV) |
| Input resolution | Native video resolution |
| GPU used | NVIDIA Tesla T4 (Kaggle) |
| Dataset | Mumbai Coastal Road Video |

---

## 📈 Performance Notes

- **Frame skipping** (`frame_skip = int(video_speed_scale) = 2`) processes every 3rd frame to balance speed vs. coverage
- **EMA smoothing** (`0.7 * prev + 0.3 * curr`) prevents jittery lane center estimates
- **YOLOv8n** is chosen for inference speed; swap to `yolov8s.pt` or `yolov8m.pt` for higher accuracy

---

## 🔮 Future Improvements

- [ ] Curved lane fitting (polynomial regression instead of straight Hough lines)
- [ ] Temporal lane tracking (Kalman filter)
- [ ] Night / rain mode (adaptive edge thresholds)
- [ ] Depth-based distance estimation for hazard proximity
- [ ] Real-time dashboard overlay with speed, GPS, and alerts
- [ ] Export to ONNX for edge deployment (Raspberry Pi / Jetson Nano)

---


## 🙌 Acknowledgements

- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
- [OpenCV](https://opencv.org/)
- Dataset: [Mumbai Coastal Road Video Sample](https://www.kaggle.com/datasets/amitchakdhare/mumbai-coastal-road-video-sample) by amitchakdhare on Kaggle

---

<p align="center">Made with ❤️ for autonomous driving research</p>
