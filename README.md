# AI_HW3
# HW3: Drone Detection & Kalman Filter Tracking
 Dataset & Detector

Used the Drone-vs-Birds dataset (Roboflow, YOLOv8 format) with ~1,500 annotated images across drone and bird classes. 
Fine-tuned YOLOv8n for 12 epochs at 640×640 resolution, batch size 16, confidence threshold **0.25**. Drone detections use class index 1.

## Kalman Filter State Design

A position-only random-walk filter (filterpy.KalmanFilter):

| Symbol | Value | Meaning |
|--------|-------|---------|
| State x | [cx, cy] | Bounding-box centre in pixels |
| Measurement z | [cx, cy] | YOLO-predicted centre |
| F | I2 | No motion assumed between frames |
| H | I2 | Measurement equals state directly |
| R | 100 I2 | ~10 px measurement std dev |
| Q | 10 · I2 | Allows moderate position drift |
| P | 100  I2 | Initial position uncertainty |

Detections are matched to tracks each frame via Hungarian algorithm on a 1 − IoU cost matrix (threshold 0.30).

## Failure Cases & Missed Detection Handling

Coasting: when the detector misses the drone, the filter continues predicting for up to 5 consecutive frames using kf.predict() alone. The bounding box from the last known detection is shown dimmed with a `[coast N/5]` label. After 5 missed frames the track is killed.

Known failure cases:
- Fast motion across frames causes drift since the position-only model cannot extrapolate velocity.
- Low-contrast backgrounds or partial occlusion cause the detector to miss the drone, consuming the coast budget quickly.
- Bird false positives (class 0 filtered out) can briefly spawn spurious tracks before being killed.

Output videos contain only drone-present frames, with a green detector bounding box and yellow trajectory polyline overlaid.
