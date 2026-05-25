# i2Nav-Robot Corridor Dataset: technical overview and generation notes

File reviewed: `build_i2nav_robot_corridor_dataset_final_N20_logged-3.ipynb`  
Generated documentation date: 2026-05-25

This document describes the i2Nav-Robot dataset generation notebook, its current configuration, what it actually produces, how the projection pipeline works, where the generated files are stored, and how the result should and should not be interpreted for future experiments and article writing.

---

## 1. Verification of the uploaded notebook

I inspected the uploaded notebook `build_i2nav_robot_corridor_dataset_final_N20_logged-3.ipynb`. It is the same family of generator notebooks we were discussing: it builds an image-to-projected-trajectory/corridor dataset from i2Nav-Robot ROS bag files and trajectory files.

Important correction: despite the filename containing `N20` and a cell comment saying `Run full export: N=20`, the current code inside the notebook is configured as **N=10**, not N=20.

Current effective configuration in the uploaded notebook:

```python
PROJECT_ROOT = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset")
I2NAV_ROOT = PROJECT_ROOT / "external_datasets" / "i2Nav-Robot" / "sample_sequence" / "i2Nav-Robot"
OUT_ROOT = PROJECT_ROOT / "prepared_i2nav_robot_corridor_dataset_v1_2_sec"

DATASET_NAME = "i2nav_robot_corridor_leftcam_v1"
LEFT_TOPIC = "/avt_camera/left/image/compressed"

EVERY_N = 10
HORIZON_SEC = 2.0
STEP_SEC = 0.25

RUN_FULL_EXPORT = True
SAVE_OVERLAYS = True
CLEAN_OUTPUT = True
```

So, if this notebook is run as-is, it will generate a denser i2Nav dataset with one sample every 10 camera frames, using a 2-second future trajectory horizon. It will write into:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

The output folder name still says `v1_2_sec`, not `N10`. The dataset name also remains `i2nav_robot_corridor_leftcam_v1`; it does not encode `N10` or `2sec`. For future reproducibility I recommend renaming either the `OUT_ROOT` or `DATASET_NAME` to make the configuration explicit, for example:

```python
OUT_ROOT = PROJECT_ROOT / "prepared_i2nav_robot_corridor_dataset_v2_2_sec_N10"
DATASET_NAME = "i2nav_robot_corridor_leftcam_v2_2sec_N10"
```

The code is structurally correct for the intended generation pipeline. It includes:

- dependency setup for `rosbags` and `opencv-python-headless`;
- calibration loading from `calibration.yaml`;
- image extraction from the left compressed camera topic;
- trajectory loading from `*_trajectory.csv`;
- camera projection using intrinsic and extrinsic calibration;
- segmentation of valid projected line fragments to avoid false diagonal artifacts;
- generation of RGB images, corridor masks, centerline masks, Gaussian heatmaps, overlays, manifest, splits, summary, and live progress logs.

Known caveats in the uploaded notebook:

1. `CLEAN_OUTPUT = True`; rerunning the notebook deletes the current output directory before regeneration.
2. The filename/comment says `N20`, but the actual sampling interval is `EVERY_N = 10`.
3. `EXPECTED_CANDIDATES = 6906` and `EXPECTED_VALID = 6067` are stale values from the previous N20-style estimate and are used mainly for ETA/progress display. They do not control generation quality. For N10, expected candidates/valid examples should be approximately higher than N20, but the exact count should be checked from `sequences_summary.json` after export.
4. The label is a projected logged future trajectory, not a guaranteed optimal or safe navigation target.

---

## 2. What this dataset is

This generator builds a supervised image-to-mask dataset from i2Nav-Robot logs.

For each selected camera frame, it produces:

```text
input:  RGB image from the left camera
output: image-space projection of the robot's recorded future path
```

The output is saved in several raster forms:

```text
corridor_mask      thick binary future-path/corridor mask
centerline_mask    thin binary centerline mask
gaussian_heatmap   blurred centerline heatmap
overlay            visual debug image with projected trajectory on RGB
```

The dataset is **Wayomo-style compatible** in structure: it has image files, mask files, metadata, a JSONL manifest, and train/val/test split files. This makes it convenient to combine with the existing Waymo/Wayomo corridor dataset, provided that the training loader handles `source_dataset` and path roots correctly.

The key conceptual point:

> The generated label is the robot's **logged future trajectory projected into the current camera image**, not a hand-designed traversability label and not an oracle safe corridor.

This matters for interpretation. The dataset is suitable as a logged-path/imitation-style dataset or as a domain-adaptation auxiliary source. It is not, by itself, a clean free-space/traversability dataset.

---

## 3. Source dataset layout expected by the notebook

The notebook expects i2Nav-Robot data under:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/external_datasets/i2Nav-Robot/sample_sequence/i2Nav-Robot
```

Expected source structure:

```text
i2Nav-Robot/
  calibration.yaml
  building00/
    building00.bag
    building00_trajectory.csv
    building00_groundtruth.nav
    ...
  building01/
  building02/
  parking00/
  parking01/
  parking02/
  playground00/
  street00/
  street01/
  street02/
```

The generator uses the following camera topic from each ROS bag:

```text
/avt_camera/left/image/compressed
```

Important interpretation: `left` means the left camera of the stereo pair. It does not mean a side-looking camera. The calibration indicates that this camera is front-facing relative to the robot IMU/body convention.

The source trajectory file is:

```text
<sequence>/<sequence>_trajectory.csv
```

Despite the `.csv` suffix, the code supports whitespace-separated loading because these files may not be comma-delimited.

---

## 4. Sequences and splits

The uploaded notebook defines splits by sequence, not by random frame sampling. This is the correct approach because adjacent frames from the same sequence are highly correlated.

Training sequences:

```text
building00
building01
building02
parking00
playground00
street00
street01
```

Validation sequence:

```text
parking01
```

Test sequences:

```text
parking02
street02
```

This produces sequence-level separation:

```text
train.txt -> sample IDs from train sequences
val.txt   -> sample IDs from parking01
test.txt  -> sample IDs from parking02 and street02
```

The split assignment is stored both in the manifest rows and in the split text files.

---

## 5. Output dataset structure

The generated output root in the uploaded notebook is:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

Expected output structure:

```text
prepared_i2nav_robot_corridor_dataset_v1_2_sec/
  images/
    <sample_id>.png

  corridor_mask/
    <sample_id>_corridor_mask.png

  centerline_mask/
    <sample_id>_centerline_mask.png

  gaussian_heatmap/
    <sample_id>_gaussian_heatmap.png

  overlays/
    <sample_id>_overlay.png

  meta/
    manifest.jsonl
    dataset_config.json
    sequences_summary.json
    generation_log.txt
    progress_live.json

  splits/
    train.txt
    val.txt
    test.txt
```

The main machine-readable file is:

```text
meta/manifest.jsonl
```

Each line is one generated training example.

---

## 6. Per-sample ID convention

The sample ID is built as:

```python
sample_id = f"{seq}_{local_saved:06d}_msg_{msg_index:08d}"
```

Example:

```text
parking01_000123_msg_00004560
```

Meaning:

- `parking01`: source sequence;
- `000123`: local saved sample index within that sequence;
- `msg_00004560`: original left-camera message index inside the ROS bag topic stream.

This ID is used consistently for RGB, masks, heatmaps, overlays, manifest rows, and split files.

---

## 7. Manifest schema

Each row in `meta/manifest.jsonl` contains the fields below.

Core identifiers:

```json
{
  "id": "...",
  "dataset_name": "i2nav_robot_corridor_leftcam_v1",
  "source_dataset": "i2Nav-Robot",
  "sequence": "parking01",
  "split": "val",
  "camera": "left",
  "camera_topic": "/avt_camera/left/image/compressed"
}
```

Input/output type metadata:

```json
{
  "input_type": "rgb_image",
  "output_type": "image_space_future_trajectory_corridor"
}
```

Relative paths from `OUT_ROOT`:

```json
{
  "image_path": "images/<sample_id>.png",
  "corridor_mask_path": "corridor_mask/<sample_id>_corridor_mask.png",
  "centerline_mask_path": "centerline_mask/<sample_id>_centerline_mask.png",
  "gaussian_heatmap_path": "gaussian_heatmap/<sample_id>_gaussian_heatmap.png",
  "overlay_path": "overlays/<sample_id>_overlay.png"
}
```

Source file paths:

```json
{
  "source_bag_path": ".../<sequence>.bag",
  "source_traj_path": ".../<sequence>_trajectory.csv",
  "source_gt_nav_path": ".../<sequence>_groundtruth.nav"
}
```

Timing and trajectory metadata:

```json
{
  "bag_msg_index": 4560,
  "image_time": 1745898384.123,
  "image_rel_time": 12.34,
  "traj_idx": 12345,
  "future_used_indices": [12350, 12375, ...],
  "horizon_sec": 2.0,
  "step_sec": 0.25,
  "num_future_points": 8,
  "valid_points": 6,
  "valid_ratio": 0.75,
  "longest_valid_run": 6,
  "segments_count": 1
}
```

Image/mask metadata:

```json
{
  "image_width": 1600,
  "image_height": 1200,
  "mask_line_width": 28,
  "centerline_width": 8,
  "max_pixel_jump": 450,
  "calibration_path": ".../calibration.yaml"
}
```

This schema is intentionally compatible with a multi-source training loader. For combining with Waymo/Wayomo, the loader should resolve:

```python
absolute_image_path = dataset_root / row["image_path"]
absolute_mask_path = dataset_root / row["corridor_mask_path"]
```

where `dataset_root` is the root path for the particular source dataset.

---

## 8. Geometry and projection pipeline

The generator projects future recorded robot positions into the current camera frame.

The pipeline for each accepted camera frame is:

```text
1. Read current RGB image from /avt_camera/left/image/compressed.
2. Get image timestamp.
3. Align image time to trajectory time using relative time from sequence start.
4. Find nearest trajectory pose at current frame.
5. Sample future trajectory poses from t + STEP_SEC to t + HORIZON_SEC.
6. Convert each future robot-floor point from future IMU frame to world frame.
7. Convert world point into current IMU frame.
8. Convert current IMU point into current left-camera frame.
9. Project camera-frame 3D points into pixels using intrinsics and distortion.
10. Filter invalid projected points.
11. Split visible points into continuous image-space segments.
12. Draw masks and overlays.
13. Save files and append manifest row.
```

### 8.1 Calibration used

The notebook loads:

```text
I2NAV_ROOT/calibration.yaml
```

For the left camera it uses:

```python
left_calib = calib["camera"]["left"]
fx, fy, cx, cy = left_calib["intrinsic"]
dist = np.array(left_calib["distortions"])
T_imu_cam = np.array(left_calib["T_imu_cam"])
T_cam_imu = np.linalg.inv(T_imu_cam)
```

The calibration convention in the file is documented in the comments as:

```text
Pi = R_i_c * Pc + t_i_c
```

So `T_imu_cam` maps points from camera coordinates into IMU coordinates. For projection, the generator needs the inverse transform:

```text
T_cam_imu = inverse(T_imu_cam)
```

### 8.2 Camera model

The generator uses OpenCV's `cv2.projectPoints` with:

```python
K = [[fx, 0, cx],
     [0, fy, cy],
     [0, 0, 1]]

dist = [k1, k2, p1, p2, k3]
```

The model is pinhole with radial-tangential distortion.

### 8.3 Robot point projected

The projected path is not the IMU origin. It uses the robot floor-center point defined by the odometry lever arm:

```python
odo_lever_imu = calib["adis16465"]["odo_lever"]
```

This point is described in the calibration file as the Ranger MINI center on the floor, expressed in the ADIS16465 IMU frame. This is appropriate for drawing a ground-contact trajectory rather than a floating IMU trajectory.

### 8.4 Quaternion convention

The trajectory quaternion is read as:

```python
quat = arr[:, 4:8]
```

and interpreted as:

```text
[x, y, z, w]
```

The notebook converts this to a rotation matrix using a standard normalized quaternion formula.

### 8.5 Time alignment

The notebook assumes that absolute ROS image timestamps and trajectory timestamps are on different absolute scales. It therefore aligns them by relative time from the first camera frame:

```python
image_rel = image_time - first_image_time
traj_rel = traj_t - traj_t[0]
traj_idx = argmin(abs(traj_rel - image_rel))
```

This is a practical alignment strategy for this dataset. It is not a high-precision calibration of camera-to-trajectory timing. If future experiments require centimeter-level temporal accuracy, a separate time-offset estimation step should be added.

---

## 9. Horizon and sampling

Current uploaded notebook:

```python
EVERY_N = 10
HORIZON_SEC = 2.0
STEP_SEC = 0.25
```

Interpretation:

- use every 10th left-camera frame;
- project the next 2 seconds of recorded future trajectory;
- sample future path every 0.25 seconds;
- produce roughly 8 future path points before filtering.

This is intentionally shorter than the earlier 5-second version. The shorter horizon is preferable for robot images because it reduces labels that run into distant walls, cars, corners, and visually ambiguous future locations.

The N10 configuration gives a denser dataset than N20. It produces more correlated neighboring samples, but it is more appropriate if the goal is to train a smaller/faster diffusion-style or segmentation-style planner and combine it with a larger Waymo/Wayomo subset.

---

## 10. Filtering logic

The notebook does not require the first future point to be visible:

```python
REQUIRE_FIRST_POINT_VALID = False
```

This is intentional. The first point may be at the bottom edge of the image or outside the image due to camera geometry, but the visible future segment may still be valid.

Core filters:

```python
MIN_VALID_POINTS = 4
MIN_VALID_RATIO = 0.10
MIN_LONGEST_VALID_RUN = 4
MAX_PIXEL_JUMP = 450
```

A sample is accepted if:

1. it has a longest continuous valid run of at least 4 points;
2. it has at least 4 valid projected points in total;
3. at least 10% of sampled future points are valid;
4. if `REQUIRE_FIRST_POINT_VALID` were true, the first point would need to be visible, but it is false in the uploaded notebook.

The most important criterion is the longest visible run. This makes the label line coherent rather than a set of isolated points.

---

## 11. Segment drawing and diagonal-artifact prevention

A major bug in early projections was false diagonal lines. These appeared when valid points were separated by invalid points, and drawing code connected all valid points into one polyline.

The uploaded notebook fixes this with:

```python
def valid_segments_from_uv(uv, valid, max_pixel_jump=450):
    ...
```

The logic:

- iterate through projected points in time order;
- if `valid[i] == False`, close the current segment;
- if a pixel jump between adjacent valid points is larger than `MAX_PIXEL_JUMP`, close the current segment;
- only draw segments with at least two points;
- do not draw lines across invalid gaps.

This prevents artificial long diagonal labels across the image.

---

## 12. Generated target types

### 12.1 `corridor_mask`

A thick binary mask of the visible projected future path.

Configuration:

```python
MASK_LINE_WIDTH = 28
```

Saved as grayscale PNG under:

```text
corridor_mask/<sample_id>_corridor_mask.png
```

Typical use:

- main segmentation target;
- binary BCE/Dice loss;
- corridor/path mask prediction.

### 12.2 `centerline_mask`

A thinner binary mask of the projected path centerline.

Configuration:

```python
CENTERLINE_WIDTH = 8
```

Saved under:

```text
centerline_mask/<sample_id>_centerline_mask.png
```

Typical use:

- centerline auxiliary loss;
- skeleton-like target;
- diffusion/heatmap target generation.

### 12.3 `gaussian_heatmap`

A Gaussian-blurred version of the centerline mask.

Generated by:

```python
centerline_mask.filter(ImageFilter.GaussianBlur(radius=10))
```

Saved under:

```text
gaussian_heatmap/<sample_id>_gaussian_heatmap.png
```

Typical use:

- smoother regression target;
- heatmap target for diffusion or U-Net style training;
- easier optimization than hard binary masks.

### 12.4 `overlay`

A debug visualization: RGB image with projected trajectory drawn on top.

Saved under:

```text
overlays/<sample_id>_overlay.png
```

Typical use:

- manual inspection;
- presentation figures;
- sanity checks;
- failure-mode analysis.

---

## 13. Logging and progress tracking

The notebook logs aggressively.

Files:

```text
meta/generation_log.txt
meta/progress_live.json
meta/sequences_summary.json
meta/dataset_config.json
meta/manifest.jsonl
```

### 13.1 `generation_log.txt`

Plain text append-only log. It records:

- export start;
- output root;
- sequence list;
- per-sequence start;
- source bag and trajectory paths;
- trajectory shape and duration;
- periodic progress every `progress_update_every` saved samples;
- per-sequence completion stats;
- final summary paths.

Useful terminal command:

```bash
tail -f /home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/generation_log.txt
```

### 13.2 `progress_live.json`

Live JSON status updated during generation. It contains:

```json
{
  "dataset_name": "...",
  "out_root": "...",
  "current_seq": "building00",
  "started_at": "...",
  "updated_at": "...",
  "elapsed_sec": 123.45,
  "eta_sec": 6789.0,
  "expected_valid": 6067,
  "expected_candidates": 6906,
  "totals": {
    "seen_frames": 10000,
    "candidate_frames": 1000,
    "saved_samples": 850,
    "skipped_samples": 150,
    "errors": 0,
    "skip_reasons": {}
  }
}
```

Caveat: in the current uploaded notebook, `expected_valid` and `expected_candidates` are stale estimates from an older configuration. They are only used for approximate ETA. The actual final counts are in `sequences_summary.json` and `manifest.jsonl`.

Useful terminal command:

```bash
watch -n 5 'cat /home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/progress_live.json'
```

### 13.3 `sequences_summary.json`

Final export summary. It contains:

- total seen frames;
- total candidate frames;
- total saved samples;
- total skipped samples;
- total errors;
- skip reasons;
- split counts;
- per-sequence stats;
- output paths.

This is the final authoritative summary of the generated dataset.

---

## 14. How to verify the generated dataset

### 14.1 Count files and manifest rows

```python
from pathlib import Path
import json

OUT_ROOT = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec")
manifest_path = OUT_ROOT / "meta" / "manifest.jsonl"

print("manifest rows:", sum(1 for _ in open(manifest_path, "r", encoding="utf-8")))

for sub in ["images", "corridor_mask", "centerline_mask", "gaussian_heatmap", "overlays"]:
    d = OUT_ROOT / sub
    print(sub, len(list(d.glob("*.png"))))
```

Expected condition:

```text
manifest rows == images == corridor_mask == centerline_mask == gaussian_heatmap == overlays
```

if overlays were enabled.

### 14.2 Check that manifest paths exist

```python
import json
from pathlib import Path

OUT_ROOT = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec")
manifest_path = OUT_ROOT / "meta" / "manifest.jsonl"

missing = []
with open(manifest_path, "r", encoding="utf-8") as f:
    for line in f:
        r = json.loads(line)
        for key in ["image_path", "corridor_mask_path", "centerline_mask_path", "gaussian_heatmap_path", "overlay_path"]:
            if r.get(key) and not (OUT_ROOT / r[key]).exists():
                missing.append((r["id"], key, r[key]))

print("missing:", len(missing))
print(missing[:10])
```

Expected result:

```text
missing: 0
```

### 14.3 Visual sample grid

Use the presentation grid code from the notebook/conversation to inspect examples:

```text
RGB input | corridor mask | heatmap | overlay
```

Inspection should verify:

- overlay line is not connected by false diagonals;
- projected line is on image ground/visible path area in most cases;
- masks align with overlays;
- no empty or all-black masks among accepted samples;
- no corrupted images.

---

## 15. How this relates to Waymo/Wayomo dataset

This i2Nav dataset is intentionally organized similarly to the Waymo/Wayomo corridor dataset.

Common conceptual format:

```text
input: RGB image
output: image-space path/corridor target
metadata: manifest.jsonl
splits: train/val/test text files
```

This makes it possible to create a combined dataset:

```text
Waymo/Wayomo subset: 10k examples
i2Nav N10 subset: 5k-6k examples, or more depending final count
```

The combined manifest should include:

```json
{
  "source_dataset": "Waymo" or "i2Nav-Robot",
  "dataset_root": "...",
  "image_path": "...",
  "corridor_mask_path": "..."
}
```

Critical training recommendation: do not silently mix the two sources without preserving `source_dataset`. Their label semantics and visual domains differ.

Waymo/Wayomo labels are typically more road-like and ego-driving aligned. i2Nav labels are robot logged-path projections. The model can benefit from i2Nav as robot-domain visual data, but the label interpretation is weaker and more imitation-like.

---

## 16. Methodological interpretation for article writing

A technically precise description:

> We generated an image-space logged-path projection dataset from i2Nav-Robot. For each selected frame of the left stereo camera, the robot's future ground-truth trajectory over a short temporal horizon was transformed into the current camera frame using the provided camera-IMU calibration and then rasterized into corridor, centerline, and heatmap targets. The resulting dataset follows the same file organization as our Waymo-style corridor dataset and can be used for supervised training or domain adaptation experiments.

Important limitation paragraph:

> The generated target should be interpreted as a projection of the recorded future motion, not as an oracle traversability or optimal navigation label. Since the label is derived from the logged trajectory, it may encode the behavior of the data collection policy and may include maneuvers that are not uniquely implied by the current image. Furthermore, the projection is not visibility-aware: without depth- or LiDAR-based occlusion filtering, parts of the future path may project onto image regions corresponding to obstacles or occluders. To reduce such artifacts, we use a short 2-second prediction horizon and draw only continuous visible projected segments.

Why the 2-second horizon is used:

> We restrict the horizon to 2 seconds to focus the target on the local visible motion corridor and reduce cases where the future logged path leaves the current field of view, goes behind obstacles, or reaches visually ambiguous areas such as corners, parked vehicles, and walls.

Why segment drawing is used:

> Projected future points may temporarily leave the camera frame or become invalid. Drawing a single polyline through all valid points can create artificial long diagonal lines across the image. We therefore split the projected sequence into continuous valid segments and additionally break segments at large pixel jumps.

Potential future improvement:

> A stronger version of the dataset should incorporate LiDAR/depth-based visibility filtering or traversability estimation. Such filtering would remove projected future points that are geometrically behind visible obstacles and would move the target definition closer to free-space affordance prediction rather than pure logged-path imitation.

---

## 17. Known limitations

1. **Logged path is not optimal path.**  
   The robot trajectory is what happened in the recorded run, not necessarily what a planner should do.

2. **No occlusion reasoning.**  
   Future 3D trajectory points are projected into the image even if an obstacle lies between the camera and the future point.

3. **Relative timestamp alignment.**  
   The generator uses relative alignment between the first camera frame and the first trajectory timestamp. This is practical but not a full temporal calibration.

4. **Dataset name mismatch.**  
   The uploaded notebook's filename says `N20`, but the actual code has `EVERY_N = 10`.

5. **Stale expected counts.**  
   `EXPECTED_CANDIDATES` and `EXPECTED_VALID` are not updated for N10. Final counts must be read from `sequences_summary.json`.

6. **Domain difference from Waymo.**  
   i2Nav-Robot scenes, camera geometry, robot behavior, and label semantics differ from road-driving Waymo scenes.

---

## 18. Recommended next steps

### 18.1 Rename current N10 output for clarity

Current output root:

```text
prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

Recommended clearer output root:

```text
prepared_i2nav_robot_corridor_dataset_v2_2_sec_N10
```

Recommended dataset name:

```text
i2nav_robot_corridor_leftcam_v2_2sec_N10
```

### 18.2 Keep N20 as a clean baseline

N20 is less dense and less correlated. N10 gives more samples but more near-duplicate frames. Keep both if disk permits:

```text
v1_2sec_N20: cleaner, smaller baseline
v2_2sec_N10: denser training subset
```

### 18.3 Combine with Waymo carefully

For a first smaller/faster planner experiment:

```text
Waymo/Wayomo: 10k examples
i2Nav-Robot: 5k-6k+ examples from N10
```

Use `source_dataset` in the manifest and optionally balanced sampling in the dataloader.

### 18.4 Consider better robot labels later

If the goal becomes true robot navigability rather than logged-path imitation, switch to one of:

- manual/LLM/segmentation labels for visible traversable corridor;
- LiDAR-derived free-space masks;
- candidate trajectory feasibility labels using occupancy/depth;
- depth-aware occlusion filtering of projected logged trajectories.

---

## 19. Minimal loader concept

A training loader can use `manifest.jsonl` as follows:

```python
import json
from pathlib import Path
from PIL import Image

root = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec")
manifest = root / "meta" / "manifest.jsonl"

rows = [json.loads(line) for line in open(manifest, "r", encoding="utf-8") if line.strip()]

row = rows[0]
image = Image.open(root / row["image_path"]).convert("RGB")
mask = Image.open(root / row["corridor_mask_path"]).convert("L")
heatmap = Image.open(root / row["gaussian_heatmap_path"]).convert("L")
```

For a combined Waymo + i2Nav manifest, each row should include `dataset_root`, and the loader should resolve paths per-row:

```python
root = Path(row["dataset_root"])
image = Image.open(root / row["image_path"]).convert("RGB")
mask = Image.open(root / row["corridor_mask_path"]).convert("L")
```

---

## 20. Bottom line

The uploaded notebook is a correct i2Nav-Robot logged-future-path projection generator, with live logging and Wayomo-style output structure. The current effective configuration is **N10, 2-second horizon**, not N20. It produces RGB images and three label variants: thick corridor mask, thin centerline, and Gaussian heatmap. It is useful for logged-path/imitation-style experiments and for robot-domain adaptation, but it should not be described as a clean optimal traversability dataset without further label refinement.
