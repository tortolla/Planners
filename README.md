# Project Files Overview

This repository contains the current notebooks and documentation for the **cross-domain future-path / corridor prediction project**.

The project has two main dataset-generation pipelines:

1. **Vehicle domain** — Waymo front-camera images with projected future ego-vehicle trajectory / corridor labels.
2. **Robot domain** — i2Nav-Robot front-camera images with projected logged future robot trajectory / corridor labels.

The long-term goal is to train a lightweight **Diffusion-Lite / FlowPlanner-Lite** model that predicts an interpretable future-path prior from a single RGB image.

The predicted future-path prior is not intended to replace a full planner. Instead, it can be used as a first-stage visual planning layer for trajectory generation, candidate path scoring, local navigation, diffusion-based or flow-based planning, and downstream safety / traversability modules.

---

## File List

### `build_corridor_dataset_filtered_front_logged-3.ipynb`

Notebook for generating the **Waymo vehicle corridor dataset**.

This notebook creates the vehicle-side dataset where the input is a front camera RGB image and the output is a projected future ego-vehicle trajectory / corridor label in image space.

Main pipeline:

```text
Waymo front image
    → future ego trajectory
    → projection into image space
    → corridor mask / trajectory target
```

This is the primary dataset-construction notebook for the vehicle part of the project.

It extracts front-facing camera frames, filters usable samples, projects the future ego trajectory onto the camera image, and saves the generated image-label pairs in a structured dataset format.

---

### `model_training-3_stronger.ipynb`

Notebook for an early **end-to-end diffusion-style planner** from a single front image.

This experiment is conceptually similar to a simplified **DiffusionDrive / Diffusion Planner** setup, but using only one front-facing camera image as input.

Main task:

```text
single front RGB image → future trajectory prediction
```

This notebook is useful as an experimental prototype for direct image-to-trajectory learning.

For the current article direction, this notebook is not the central pipeline. It is kept as a reference for future **Diffusion-Lite / FlowPlanner-Lite** experiments.

---

### `train_corridor_resnet50_augmented_no_intent_v2_pbar-2.ipynb`

Notebook for training a **U-Net-style segmentation model** on the Waymo corridor dataset.

The dataset used here is produced by:

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
```

Main task:

```text
Waymo RGB image → corridor segmentation mask
```

The model uses a ResNet50-based encoder with augmentations and progress logging.

This notebook is important as a deterministic segmentation baseline before moving to diffusion-style or flow-based future-path generation.

---

### `wayomo_end_to_end_corridor_dataset_description.md`

Full technical description of the **Waymo / Wayomo corridor dataset**.

This document explains how the vehicle dataset was created, where it is stored, what the input/output format is, and how the generated files are organized.

It should be used as the main reference document for the vehicle-side dataset pipeline.

It contains:

```text
- dataset creation logic;
- input/output definition;
- folder structure;
- file naming conventions;
- generated masks / labels;
- intended use;
- technical notes.
```

---

### `build_i2nav_robot_corridor_dataset_final_N20_logged.ipynb`

Notebook for generating the **i2Nav-Robot corridor dataset**.

Despite the historical filename containing `N20`, the effective sampling parameter should always be checked inside the notebook. Recent versions may use either:

```python
EVERY_N = 20
```

or:

```python
EVERY_N = 10
```

depending on the intended dataset density.

Main task:

```text
i2Nav-Robot front/left stereo camera image
    → logged future robot trajectory
    → projected image-space corridor / centerline / heatmap labels
```

This notebook reads i2Nav-Robot ROS bag files, camera images, trajectory files, and calibration files.

It produces a robot dataset with a structure compatible with the Waymo corridor dataset.

Typical generated output:

```text
images/
corridor_mask/
centerline_mask/
gaussian_heatmap/
overlays/
meta/
splits/
```

The robot labels are based on **logged future trajectory projection**. They should be interpreted as weak imitation supervision, not as guaranteed optimal or safe navigation targets.

---

### `i2nav_robot_corridor_dataset_technical_overview.md`

Full technical description of the **i2Nav-Robot corridor dataset**.

This document describes how the robot dataset is generated, how calibration is used, how logged trajectories are projected into image space, and how masks / heatmaps / overlays are saved.

It should be used as the main reference document for the robot-side dataset pipeline.

It contains:

```text
- i2Nav-Robot source data;
- camera topic and trajectory files;
- calibration usage;
- projection pipeline;
- segmentation / heatmap target construction;
- logging and progress files;
- dataset structure;
- limitations of logged trajectory supervision.
```

---

## Current Conceptual Structure

The repository currently supports the following pipeline:

```text
Waymo logs
    ↓
front camera image + future ego path
    ↓
image-space corridor labels

i2Nav-Robot logs
    ↓
front/left camera image + logged robot future path
    ↓
image-space corridor labels

combined training set
    ↓
compact visual future-path predictor
    ↓
Diffusion-Lite / FlowPlanner-Lite trajectory prior
```

The key unifying idea is that both vehicles and ground robots can be represented through the same visual supervision format:

```text
RGB image → future-path mask / centerline / heatmap
```

This avoids directly merging incompatible physical action spaces.

Instead of forcing cars and robots into the same metric trajectory representation, the project maps both domains into **image-space future-path supervision**.

---

## Intended Research Direction

The target model is not a full navigation foundation model.

The intended model is a lightweight and interpretable first-stage planning module:

```text
single RGB image → future-path prior
```

Possible output forms:

```text
- binary corridor mask;
- thin centerline mask;
- Gaussian trajectory heatmap;
- sampled future trajectory.
```

This module can later be used by local trajectory planners, candidate path generators, diffusion-based planners, flow-matching planners, safety / traversability modules, and downstream robot or vehicle control systems.

The planned model family can be described as:

```text
Diffusion-Lite / FlowPlanner-Lite
```

The goal is to obtain something smaller, faster, and easier to interpret than full diffusion planners or large navigation foundation models, while still preserving the ability to predict plausible future-path structure from visual input.

---

## Important Methodological Note

The generated targets are based on **logged future trajectories**.

This means:

```text
The label is what the vehicle or robot actually did in the log.
It is not necessarily the globally optimal path.
It is not guaranteed to be collision-free.
It is not a complete traversability map.
```

For Waymo, logged future trajectory is usually a reasonable proxy because the ego vehicle mostly follows structured roads and lanes.

For i2Nav-Robot, logged future trajectory is weaker supervision because the robot may approach walls, parked cars, doors, narrow passages, or perform operator-specific maneuvers.

Therefore, the robot dataset should be interpreted as:

```text
weak imitation-style future-path supervision
```

not as:

```text
safe navigation oracle
```

This distinction is important for future experiments and for the article.

---

## Suggested Use of Files

### Vehicle dataset generation

Use:

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
```

Then inspect:

```text
wayomo_end_to_end_corridor_dataset_description.md
```

Then train a deterministic baseline with:

```text
train_corridor_resnet50_augmented_no_intent_v2_pbar-2.ipynb
```

Recommended flow:

```text
1. Generate Waymo corridor dataset.
2. Verify overlays and masks.
3. Train U-Net / ResNet50 segmentation baseline.
4. Use the trained baseline as a reference before Diffusion-Lite experiments.
```

---

### Robot dataset generation

Use:

```text
build_i2nav_robot_corridor_dataset_final_N20_logged.ipynb
```

Then inspect:

```text
i2nav_robot_corridor_dataset_technical_overview.md
```

Recommended flow:

```text
1. Generate i2Nav-Robot corridor dataset.
2. Check the actual values of EVERY_N and HORIZON_SEC inside the notebook.
3. Verify overlays, masks, centerlines, and heatmaps visually.
4. Use the resulting dataset for robot-domain or mixed-domain training.
```

Recommended dense robot configuration:

```python
EVERY_N = 10
HORIZON_SEC = 2.0
STEP_SEC = 0.25
```

Expected result:

```text
approximately 5,000–6,000 robot samples
```

---

### Early model experiments

Use:

```text
train_corridor_resnet50_augmented_no_intent_v2_pbar-2.ipynb
```

as the deterministic segmentation baseline.

Use:

```text
model_training-3_stronger.ipynb
```

as a reference for direct image-to-trajectory and diffusion-style experiments.

The planned next step is to extend these experiments toward:

```text
Diffusion-Lite / FlowPlanner-Lite
```

---

## Planned Combined Dataset

The planned mixed dataset should contain:

```text
Waymo subset:
- approximately 10,000 samples

i2Nav-Robot subset:
- approximately 5,000–6,000 samples with EVERY_N = 10
```

Each manifest row should preserve:

```text
source_dataset
dataset_root
image_path
corridor_mask_path
centerline_mask_path
gaussian_heatmap_path
split
sequence
horizon_sec
```

This is required because Waymo and i2Nav have different visual distributions, different embodiment dynamics, and different label reliability.

---

## Suggested Combined Training Setup

The first mixed-domain experiment should compare:

```text
1. Waymo-only training
2. i2Nav-only training
3. Waymo + i2Nav mixed training
```

The goal is not to claim a complete navigation planner.

The goal is to test whether a compact model can learn a shared visual future-path prior across vehicle and robot domains.

Possible evaluation targets:

```text
- corridor mask IoU;
- Dice score;
- centerline distance;
- endpoint error in image space;
- qualitative overlay inspection;
- cross-domain generalization.
```

---

## Research Positioning

Large navigation foundation models aim to solve general cross-task and cross-embodiment navigation using large VLM/VLA architectures, multimodal inputs, temporal history, language instructions, and massive datasets.

This project takes a lighter and more interpretable direction:

```text
single RGB image
    → future-path mask / centerline / heatmap
```

The central hypothesis is that a shared **image-space future-path representation** can serve as a compact bridge between autonomous driving data and ground robot navigation data.

This makes the project closer to a lightweight visual planning prior than to a full navigation foundation model.

---

## Short Summary

This repository contains the current experimental infrastructure for building a compact cross-domain future-path prediction system.

The central idea:

```text
Use a shared image-space future-path representation
to connect autonomous driving data and ground robot navigation data.
```

The current project does not aim to replace a full planner.

Instead, it aims to build a lightweight, interpretable visual prior that can be used as the first stage of future trajectory generation.

```text
Single RGB image
    ↓
future-path heatmap / corridor / centerline
    ↓
Diffusion-Lite or FlowPlanner-Lite downstream planner
```
