# Smart Crowd Panic Detection System

A computer vision prototype that watches a video or camera feed, tracks people and estimates a **panic score** from how the crowd is moving.

> **Prototype only.** The score uses hand-set weights and thresholds and has not been evaluated on labelled data. Do not rely on it for real safety decisions.

## Features

- Person detection with YOLOv8n
- Lightweight IoU-based tracking with motion trails
- Dense optical flow (Farneback) for scene-wide motion
- Panic score with four levels
- Heatmap, crowd map and on-screen metrics
- Annotated video output and screenshots
- Uses a CUDA GPU if available, otherwise the CPU

## How It Works

1. YOLOv8n detects people on every second frame.
2. A tracker matches boxes between frames and records movement.
3. Optical flow measures motion across the whole image.
4. Five signals are combined into a score from 0 to 1:

| Signal | Meaning | Weight |
|---|---|---|
| Speed spike ratio | Share of tracks averaging more than 18 px/frame | 0.25 |
| Direction chaos | How sharply tracked people turn | 0.25 |
| Motion entropy | Spread of optical flow directions | 0.20 |
| Convergence | Motion flowing inwards | 0.15 |
| Density pressure | Tracked people divided by 35, capped at 1 | 0.15 |

5. The score is averaged over the last 20 analyses:

| Score | State |
|---|---|
| Below 0.25 | NORMAL |
| 0.25 to below 0.50 | ALERT |
| 0.50 to below 0.72 | PANIC RISK |
| 0.72 and above | MASS PANIC |

## Installation

You need Python 3 and a desktop environment, because the program opens a video window.

```bash
pip install ultralytics opencv-python numpy torch scipy
```

The script also checks for `torchvision`, `matplotlib`, `Pillow` and `supervision`, and installs any that are missing when it starts.

**Model weights:** place `yolov8n.pt` in the folder you run the script from. If it is missing, Ultralytics normally downloads it on first run.

## Usage

Run these from the project folder.

```bash
# Default webcam
python app.py

# Video file
python app.py path/to/video.mp4

# Network camera
python app.py rtsp://<camera-address>
```

| Key | Action |
|---|---|
| `Q` | Quit |
| `P` | Pause or resume |
| `S` | Save a screenshot |

The annotated video is saved as `output_crowd_panic_ai.mp4` and is overwritten on each run.

## Configuration

These values are set in `app.py`:

| Setting | Default | Purpose |
|---|---|---|
| `SPEED_SPIKE_THRESH` | 18.0 | Speed counted as a spike (px/frame) |
| `DENSITY_MODERATE` / `HIGH` / `CRITICAL` | 8 / 18 / 35 | People counts for density levels |
| `CONVERGENCE_THRESH` | 0.60 | Triggers the stampede flow warning |
| `INFER_EVERY` | 2 | Run detection every N frames |
| `INFER_SIZE` | 416 | Size frames are resized to before detection |

## Known Limitations

- **Not validated:** no accuracy, precision or recall figures are available.
- **Speed depends on the video:** pixels per frame changes with resolution, frame rate and camera distance.
- **Skipped frames lower speed:** reused boxes are recorded as zero movement.
- **Reversals are missed:** direction chaos uses `abs(sin(angle change))`, so a 180° turn scores about zero.
- **Colours are wrong:** values are written as RGB but OpenCV reads BGR, so red appears blue.
- **Counts can be inflated:** recently lost tracks are still counted for a few updates.
- **Crowded scenes are hard:** overlapping people can be missed or change ID.
- **Crowd map is approximate:** it scales image positions and is not a true bird's-eye view.
- **Small videos:** the crowd map is hidden if the frame is too small to fit it.

## Future Work

- Fix the speed, direction and colour issues above
- Normalise speed by person height and frame rate
- Evaluate on labelled normal and panic footage
- Tune weights and thresholds on data kept separate from testing
- Add a headless mode and alert notifications

