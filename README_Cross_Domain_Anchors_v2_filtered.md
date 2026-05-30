# Cross-Domain Anchors v2 Filtered

## 1. Purpose

This directory describes the second version of the cross-domain trajectory anchors for the **CDAD-Planner / Cross-Domain Anchor Diffusion Planner** project.

The goal of this step was to improve the anchor vocabulary used by the model. In the first version of the anchors, several clusters represented very long or extreme image-space trajectories. These anchors were not necessarily wrong, but they were risky for training because they mixed three different factors:

1. trajectory shape;
2. image-space scale;
3. possible trajectory-extraction outliers.

The v2 filtered version removes clear geometric outlier trajectories before rebuilding the anchor vocabulary.

The resulting anchor set is intended for the next model version:

```text
CDAD-Planner-Lite v2
RGB image → shared encoder → path prior head + anchor classifier + truncated anchor diffusion refiner
```

The main training files for v2 are:

```text
trajectory_vectors_with_anchor_K20_filtered.jsonl
anchors_K20_M8_norm.npy
```

---

## 2. Source Files

The v2 anchors are generated from the v1 cross-domain trajectory manifest:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v1/trajectory_vectors_with_anchor_K20.jsonl
```

This v1 manifest contains combined image-space trajectories from:

```text
Waymo corridor dataset
i2Nav-Robot corridor dataset
```

The trajectories are represented as:

```text
M = 8 future trajectory points
trajectory shape = [8, 2]
flattened trajectory dimension = 16
coordinate system = normalized image-space coordinates
```

Coordinate normalization:

```text
u_norm = 2 * u / image_width  - 1
v_norm = 2 * v / image_height - 1
```

Important: this is **coordinate normalization**, not trajectory-length normalization. Therefore, trajectories can still have different lengths and spatial extents.

This is intentional. We do not normalize every trajectory by arc length because that would remove useful scale information: a short local robot path and a longer road-vehicle path should not necessarily become identical.

---

## 3. Output Directory

The v2 filtered anchor artifacts are saved here:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

Expected files:

```text
anchors_K20_M8_norm.npy
anchors_K20_M8_norm.json
anchors_K20_filtered_preview.jpg
trajectory_vectors_filtered_no_extreme.jsonl
trajectory_vectors_removed_extreme.jsonl
trajectory_vectors_with_anchor_K20_filtered.jsonl
filtering_and_anchor_generation_stats.json
README_Cross_Domain_Anchors_v2_filtered.md
```

If these files already exist, the anchors have already been saved.

To check this on the server:

```python
from pathlib import Path

ROOT = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered")

for p in sorted(ROOT.iterdir()):
    if p.is_file():
        print(f"{p.name:70s} {p.stat().st_size / 1024**2:8.3f} MB")
```

---

## 4. What Was Changed Relative to v1

### v1

The original v1 pipeline was:

```text
Waymo + i2Nav trajectories
    ↓
normalize image coordinates to [-1, 1]
    ↓
KMeans with K=20
    ↓
anchors_K20_M8_norm.npy
```

The issue was that some anchors became very large or extreme. In particular, several low-count anchors represented long diagonal or strongly curved trajectories.

### v2

The v2 pipeline is:

```text
Waymo + i2Nav trajectories
    ↓
compute geometric trajectory features
    ↓
remove extreme trajectory samples
    ↓
run KMeans again with K=20
    ↓
save filtered anchors
```

The anchors are not edited manually. The filtering is applied to the trajectory samples before clustering.

---

## 5. Filtering Logic

For every trajectory, the following features are computed in normalized image-space coordinates:

```text
total_len   = sum of distances between consecutive trajectory points
max_step    = maximum distance between consecutive points
u_span      = max(u_norm) - min(u_norm)
v_span      = max(v_norm) - min(v_norm)
min_v       = minimum v_norm value, i.e. highest point reached in the image
max_v       = maximum v_norm value, i.e. lowest point reached in the image
bad_v_steps = number of strong downward steps in image space
roughness   = cumulative change in local direction vectors
```

For a front-facing camera, a future path usually moves from the lower part of the image toward the horizon. Therefore, extremely long paths, very large side-to-side spans, strong jumps between adjacent points, and strongly non-monotonic paths are treated as possible outliers.

The filtering is conservative. Its purpose is not to keep only visually perfect examples. Its purpose is to remove trajectory samples that can distort the anchor vocabulary.

---

## 6. Filtering Thresholds

The v2 filtering thresholds are:

```python
THRESHOLDS = {
    "max_total_len": 1.45,
    "max_step": 0.55,
    "max_u_span": 1.15,
    "max_v_span": 1.20,
    "min_v_allowed": -0.75,
    "max_bad_v_steps": 2,
    "max_roughness": 1.30,
}
```

Interpretation:

```text
max_total_len:
    removes trajectories with excessive total length in normalized image space.

max_step:
    removes trajectories with a large jump between neighboring points.

max_u_span:
    removes trajectories that cross too much of the image horizontally.

max_v_span:
    removes trajectories with excessive vertical extent.

min_v_allowed:
    removes trajectories that go too high toward the top of the image.

max_bad_v_steps:
    removes strongly non-monotonic trajectories that repeatedly move downward.

max_roughness:
    removes broken or unstable trajectory shapes.
```

These thresholds can be adjusted later, but this v2 version should be treated as a fixed experimental artifact.

---

## 7. Anchor Generation

After filtering, KMeans is run again on the remaining trajectories.

Configuration:

```text
K = 20
M = 8
trajectory dimension = 16
algorithm = OpenCV KMeans
initialization = KMeans++
max iterations = 300
attempts = 10
seed = 42
```

The result is saved as:

```text
anchors_K20_M8_norm.npy
anchors_K20_M8_norm.json
```

The `.npy` file is used for training.

The `.json` file is kept for inspection, version control, and easier reproducibility.

---

## 8. Why K=20 Is Kept

Earlier K selection was evaluated for:

```text
K = 8, 12, 16, 20, 24, 32
```

The v1 KMeans metrics showed:

```text
K= 8 | rmse=0.2763 | min_cluster=287 | max_cluster=6141
K=12 | rmse=0.2433 | min_cluster=38  | max_cluster=4955
K=16 | rmse=0.2185 | min_cluster=35  | max_cluster=4150
K=20 | rmse=0.2027 | min_cluster=34  | max_cluster=2753
K=24 | rmse=0.1908 | min_cluster=6   | max_cluster=2584
K=32 | rmse=0.1720 | min_cluster=5   | max_cluster=2157
```

K=20 was selected as the main trade-off:

```text
1. lower reconstruction error than K=16;
2. no near-empty clusters;
3. less over-fragmentation than K=24 or K=32;
4. compact enough for a lightweight anchor-conditioned diffusion model.
```

The v2 pipeline keeps K=20 for comparability and stability.

---

## 9. How to Use These Files for Training

For the next training notebook, set:

```python
ANCHOR_ROOT = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered")

ANCHORS_PATH = ANCHOR_ROOT / "anchors_K20_M8_norm.npy"
MANIFEST_PATH = ANCHOR_ROOT / "trajectory_vectors_with_anchor_K20_filtered.jsonl"
```

The model should use:

```text
input image:
    RGB image, later RGB + CoordConv channels

target:
    trajectory_norm with shape [8, 2]

anchor:
    anchor_id from the filtered K=20 vocabulary

main output:
    top-1 refined trajectory

diagnostic output:
    top-k refined trajectories for visualization and uncertainty analysis
```

---

## 10. Planned Model v2 Changes

The filtered anchors are intended to be used with the next version of the model:

```text
CDAD-Planner-Lite v2
```

Planned changes:

```text
1. use filtered anchors v2;
2. add CoordConv channels: RGB + x_map + y_map;
3. add endpoint / scale auxiliary head;
4. strengthen trajectory and endpoint losses;
5. keep path-prior auxiliary head;
6. keep anchor-conditioned truncated diffusion;
7. use top-k trajectories for visualization, not as final legal vehicle plans.
```

The final planning output for vehicles remains the top-1 trajectory. Top-k trajectories are used for diagnostics and paper visualizations.

---

## 11. Methodological Note

The filtering step does not make the labels optimal or safe.

The labels remain:

```text
logged future trajectories projected into image space
```

This means:

```text
The labels describe what the vehicle or robot did in the log.
They are not guaranteed to be globally optimal.
They are not guaranteed to be collision-free.
They are not a complete traversability map.
```

The filtered anchor vocabulary only improves the geometric stability of the trajectory prior.

---

## 12. Reproducibility Checklist

To reproduce this artifact, preserve:

```text
1. source manifest path;
2. filtering thresholds;
3. filtering script / notebook cell;
4. OpenCV KMeans settings;
5. output stats JSON;
6. anchor preview image;
7. final anchors .npy and .json;
8. filtered manifest with new anchor IDs.
```

Minimum files to keep in Git or lab journal:

```text
README_Cross_Domain_Anchors_v2_filtered.md
filtering_and_anchor_generation_stats.json
anchors_K20_M8_norm.json
anchors_K20_filtered_preview.jpg
```

Files needed locally for training:

```text
anchors_K20_M8_norm.npy
trajectory_vectors_with_anchor_K20_filtered.jsonl
```

The full `.jsonl` can be large and does not necessarily need to be committed to Git if the generation procedure is documented.

---

## 13. Download / Copy Notes

On the server, the complete v2 artifact folder is:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

To create a compact archive for download:

```bash
cd /home/Jupyter/datasets/tesla/Waymo_open_dataset

tar -czf cross_domain_anchor_trajectories_cdad_planner_v2_filtered.tar.gz \
  cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

To create a small Git-friendly archive without the large `.jsonl` files:

```bash
cd /home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered

tar -czf cdad_v2_filtered_anchors_git_artifacts.tar.gz \
  README_Cross_Domain_Anchors_v2_filtered.md \
  filtering_and_anchor_generation_stats.json \
  anchors_K20_M8_norm.json \
  anchors_K20_filtered_preview.jpg
```

To copy the important training files to another folder:

```bash
cp anchors_K20_M8_norm.npy /target/folder/
cp trajectory_vectors_with_anchor_K20_filtered.jsonl /target/folder/
```

---

## 14. Current Status

The v2 anchors looked significantly cleaner than the v1 anchors in visual inspection. The extreme long anchors observed in the v1 preview were reduced after filtering and reclustering.

This v2 filtered anchor vocabulary should be used for the next CDAD-Planner-Lite training run.
