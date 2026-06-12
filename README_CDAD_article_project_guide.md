# CDAD Article Project Guide

This file is a compact project navigator for continuing the article work on:

```text
Truncated Anchor Diffusion for Single-Image Future-Path Prediction
in Autonomous Vehicles and Ground Robots
```

Purpose of this README:

```text
1. Keep only article-relevant files.
2. Record how the two trajectory datasets were generated.
3. Record how the final filtered cross-domain anchors were built.
4. Record which notebooks produced the numerical experiments.
5. Record where to look for server-side datasets, results, tables, and article materials.
6. Provide enough context for continuing the manuscript in another dialogue.
```

This guide intentionally removes old side paths such as segmentation-only baselines, early corridor-mask experiments, and obsolete notebook variants unless they are directly needed to explain the current article.

---

## 1. Current Article Framing

The article is not framed as corridor-mask prediction.

The article task is:

```text
single RGB image
    -> ordered image-space future trajectory
    -> ADE/FDE evaluation
```

Preferred article terminology:

```text
single-image future-path prediction
ordered image-space trajectory
compact path-level representation
cross-domain trajectory anchors
anchor-conditioned truncated diffusion
Truncated Anchor Diffusion
```

Avoid in the article except in historical/data-construction notes:

```text
corridor mask prediction
drivable-area segmentation
full planner
closed-loop control
safe path oracle
optimal trajectory
multi-trajectory diffusion planner
```

Correct interpretation:

```text
The model predicts one ordered future trajectory in normalized image coordinates.
The trajectory is an intermediate path-level representation, not an executable control command.
```

---

## 2. Server Root and Main Project Locations

Primary server root used in the project:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset
```

Main article-relevant dataset folders:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

Recommended article-side result organization, if cleaning the server:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/results/cdad_article/
    datasets/
    anchors/
    experiments/
    tables/
    figures/
    manuscript/
```

If result folders remain in notebook-generated locations, search for:

```text
runs/
outputs/
experiments/
checkpoints/
metrics.json
history.json
train.log
best_model.pt
last_model.pt
```

---

## 3. Vehicle Dataset: Waymo-Derived Front-Camera Future-Path Data

### 3.1 Final dataset-generation notebook

Use this notebook as the final vehicle-side dataset-generation reference:

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
```

Approximate local path in the working environment:

```text
/mnt/data/build_corridor_dataset_filtered_front_logged-3.ipynb
```

Expected server-side project location:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/
```

Dataset output folder:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1
```

### 3.2 What the notebook does

Article-relevant interpretation:

```text
Waymo front-camera RGB image
    -> logged future ego trajectory
    -> projection into image space
    -> ordered future trajectory target
```

Historical generated labels may include:

```text
corridor_mask/
centerline_mask/
gaussian_heatmap/
overlays/
```

For the current article, these are not the central model target. They are useful for inspection and historical dataset validation, but the article should emphasize the trajectory target.

### 3.3 Main vehicle dataset documentation

Use this Markdown document if available in the repository:

```text
wayomo_end_to_end_corridor_dataset_description.md
```

Approximate local path from previous work:

```text
/mnt/data/wayomo_end_to_end_corridor_dataset_description.md
```

What it should contain:

```text
- Waymo source data description;
- front-camera frame extraction;
- future ego-trajectory extraction;
- projection into image coordinates;
- output folder structure;
- manifests / splits;
- visual overlay checks;
- limitations of logged ego-trajectory supervision.
```

### 3.4 Paper-safe naming

Use in manuscript:

```text
Waymo-derived front-camera future-path dataset
vehicle front-camera future-path dataset
```

Avoid in main article framing:

```text
Waymo corridor dataset
corridor segmentation dataset
```

---

## 4. Robot Dataset: i2Nav-Robot Future-Path Data

### 4.1 Final dataset-generation notebook

Use this notebook as the final robot-side dataset-generation reference:

```text
build_i2nav_robot_corridor_dataset_final_N20_logged-3.ipynb
```

Also relevant earlier/final variants:

```text
build_i2nav_robot_corridor_dataset_final_N20_logged.ipynb
build_i2nav_robot_corridor_dataset_final_N20_logged-2.ipynb
build_i2nav_robot_corridor_dataset_logged_v2_fixed_segments.ipynb
```

Approximate local paths from previous work:

```text
/mnt/data/build_i2nav_robot_corridor_dataset_final_N20_logged-3.ipynb
/mnt/data/build_i2nav_robot_corridor_dataset_final_N20_logged-2.ipynb
/mnt/data/build_i2nav_robot_corridor_dataset_final_N20_logged.ipynb
```

Dataset output folder:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

### 4.2 What the notebook does

Article-relevant interpretation:

```text
i2Nav-Robot camera image
    -> logged future robot trajectory
    -> projection into image space using calibration
    -> ordered future trajectory target
```

Historical generated labels may include:

```text
corridor_mask/
centerline_mask/
gaussian_heatmap/
overlays/
meta/
splits/
manifest.jsonl
```

For the current article, the important label is the ordered image-space future trajectory. Masks and heatmaps are inspection / auxiliary historical artifacts.

### 4.3 Recommended final robot configuration

Check the actual notebook before reporting final values, but the intended dense configuration is:

```python
EVERY_N = 10
HORIZON_SEC = 2.0
STEP_SEC = 0.25
```

Article-level trajectory length:

```text
M = 8 future points
```

Expected sample scale:

```text
approximately 5,000-6,000 robot samples
```

### 4.4 Main robot dataset documentation

Use this Markdown document if available in the repository:

```text
i2nav_robot_corridor_dataset_technical_overview.md
```

Approximate local path from previous work:

```text
/mnt/data/i2nav_robot_corridor_dataset_technical_overview.md
```

What it should contain:

```text
- source i2Nav-Robot data;
- camera topics / image streams;
- trajectory files;
- calibration usage;
- projection pipeline;
- output structure;
- split and manifest logic;
- limitations of logged trajectory supervision.
```

### 4.5 Paper-safe naming

Use in manuscript:

```text
i2Nav-Robot ground-robot future-path dataset
ground-robot future-path dataset
```

Important methodological caveat:

```text
The robot labels are logged future trajectories.
They are weak imitation-style supervision.
They are not guaranteed optimal or safe navigation targets.
```

---

## 5. Final Cross-Domain Anchor Construction

### 5.1 Final anchor-construction notebook

Use this notebook as the final anchor-construction reference:

```text
build_cross_domain_anchor_trajectories_cdad_planner_cv2_kmeans.ipynb
```

Related notebook:

```text
build_cross_domain_anchor_trajectories_cdad_planner.ipynb
```

Approximate local paths from previous work:

```text
/mnt/data/build_cross_domain_anchor_trajectories_cdad_planner_cv2_kmeans.ipynb
/mnt/data/build_cross_domain_anchor_trajectories_cdad_planner.ipynb
```

Final anchor output folder:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

### 5.2 What the anchor notebook does

Pipeline:

```text
Waymo image-space future trajectories
    + i2Nav image-space future trajectories
    -> common normalized trajectory format
    -> trajectory filtering
    -> K-means / CV2-based anchor construction
    -> final cross-domain anchor vocabulary
```

Final anchor configuration:

```text
K = 20 anchors
M = 8 trajectory points
coordinate system = normalized image coordinates
filtering = extreme / invalid / unsuitable trajectory samples removed
normalization = no per-trajectory arc-length normalization
```

The exact filtering criteria should be checked inside the final notebook before writing the Method section. The article should explicitly state:

```text
Extreme or geometrically unsuitable trajectories were removed before anchor construction.
```

If the notebook uses length filtering, describe it as:

```text
trajectory-length filtering before anchor clustering
```

### 5.3 Important interpretation

Anchors are:

```text
coarse trajectory prototypes
```

Anchors are not:

```text
multiple final predictions
full trajectory distribution
final planner outputs
```

The model:

```text
selects one anchor
then refines it through image-conditioned truncated diffusion
```

### 5.4 Main anchor documentation

Use these Markdown documents if available in the repository:

```text
README_Cross_Domain_Anchors.md
README_Cross_Domain_Anchors_v2_filtered.md
```

Approximate local paths from previous work:

```text
/mnt/data/README_Cross_Domain_Anchors.md
/mnt/data/README_Cross_Domain_Anchors_v2_filtered.md
```

What they should contain:

```text
- source trajectory sources;
- normalization details;
- filtering logic;
- K value;
- trajectory length M;
- visualization of anchors;
- final anchor output folder;
- anchor artifact filenames.
```

---

## 6. Final Models and Numerical Experiments

The article uses two main numerical comparisons:

```text
Table 1 or 2: Cross-domain training / transfer comparison
Table 2 or 3: Parameter-matched ablation comparison
```

The exact table numbering depends on whether the article includes a dataset-statistics table first.

---

## 7. Main Model: Mixed-Domain Truncated Anchor Diffusion

### 7.1 Main mixed-domain training notebook

Use this notebook as the main model reference:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
```

Approximate local path:

```text
/mnt/data/train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
```

Expected server-side location:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/
```

### 7.2 Model logic

```text
RGB image
    -> visual encoder
    -> image embedding
    -> anchor prediction head
    -> selected cross-domain anchor
    -> image-conditioned truncated diffusion refinement
    -> predicted M x 2 trajectory
```

### 7.3 Main mixed-domain result

```text
Mixed ADE/FDE = 0.04719 / 0.05829
Waymo ADE/FDE = 0.05117 / 0.06559
i2Nav ADE/FDE = 0.03985 / 0.04481
```

---

## 8. Cross-Domain Training / Transfer Experiments

### 8.1 Mixed-domain training

Notebook:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
```

Result:

```text
Mixed ADE/FDE = 0.04719 / 0.05829
Waymo ADE/FDE = 0.05117 / 0.06559
i2Nav ADE/FDE = 0.03985 / 0.04481
```

### 8.2 i2Nav-only training on same mixed-test protocol

Notebook:

```text
train_cdad_planner_lite_v3_TRAIN_i2Nav_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Related notebook:

```text
train_cdad_planner_lite_v3_i2nav_only_same_split_filtered_anchors_v1_loss_logged.ipynb
```

Approximate local paths:

```text
/mnt/data/train_cdad_planner_lite_v3_TRAIN_i2Nav_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
/mnt/data/train_cdad_planner_lite_v3_i2nav_only_same_split_filtered_anchors_v1_loss_logged.ipynb
```

Result:

```text
Mixed ADE/FDE = 0.06901 / 0.09578
Waymo ADE/FDE = 0.08389 / 0.12159
i2Nav ADE/FDE = 0.04157 / 0.04818
```

### 8.3 Waymo-only training on same mixed-test protocol

Notebook:

```text
train_cdad_planner_lite_v3_TRAIN_Waymo_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Approximate local path:

```text
/mnt/data/train_cdad_planner_lite_v3_TRAIN_Waymo_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Result:

```text
Mixed ADE/FDE = 0.05238 / 0.06931
Waymo ADE/FDE = 0.05093 / 0.06607
i2Nav ADE/FDE = 0.05507 / 0.07528
```

### 8.4 Article table: cross-domain transfer

Use this table in the article:

```text
Train data | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE
Mixed      | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481
i2Nav-only | 0.06901 / 0.09578 | 0.08389 / 0.12159 | 0.04157 / 0.04818
Waymo-only | 0.05238 / 0.06931 | 0.05093 / 0.06607 | 0.05507 / 0.07528
```

Interpretation for the article:

```text
Mixed-domain training provides the best mixed-test result.
It improves robot-to-vehicle transfer relative to i2Nav-only training.
It improves vehicle-to-robot transfer relative to Waymo-only training.
```

---

## 9. Parameter-Matched Ablation Experiments

### 9.1 Direct coordinate regression baseline

Notebook:

```text
train_DIRECT_REGRESSION_CLASSICAL_PARAM_MATCHED_same_mixed_split_logged.ipynb
```

Approximate local path:

```text
/mnt/data/train_DIRECT_REGRESSION_CLASSICAL_PARAM_MATCHED_same_mixed_split_logged.ipynb
```

Result:

```text
Params = 833,072
Mixed ADE/FDE = 0.13716 / 0.15706
Waymo ADE/FDE = 0.13657 / 0.18012
i2Nav ADE/FDE = 0.13826 / 0.11453
```

Purpose:

```text
Tests whether direct image-to-coordinate regression is sufficient.
```

### 9.2 Anchor-only baseline

Evaluation notebook:

```text
evaluate_cdad_v3_anchor_only_vs_refined_mixed_same_test.ipynb
```

Approximate local path:

```text
/mnt/data/evaluate_cdad_v3_anchor_only_vs_refined_mixed_same_test.ipynb
```

Result:

```text
Mixed ADE/FDE = 0.14048 / 0.16984
Waymo ADE/FDE = 0.14914 / 0.18790
i2Nav ADE/FDE = 0.12451 / 0.13654
```

Purpose:

```text
Tests whether anchor selection alone is sufficient.
```

### 9.3 Anchor + deterministic residual baseline

Notebook:

```text
train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged_v2.ipynb
```

Related notebook:

```text
train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Approximate local paths:

```text
/mnt/data/train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged_v2.ipynb
/mnt/data/train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Result:

```text
Params = 835,397
Mixed ADE/FDE = 0.12302 / 0.15110
Waymo ADE/FDE = 0.14502 / 0.18252
i2Nav ADE/FDE = 0.08244 / 0.09315
```

Purpose:

```text
Tests whether a single deterministic correction of the selected anchor is sufficient.
```

### 9.4 Anchor + truncated diffusion

Main notebook:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
```

Result:

```text
Params = 835,397
Mixed ADE/FDE = 0.04719 / 0.05829
Waymo ADE/FDE = 0.05117 / 0.06559
i2Nav ADE/FDE = 0.03985 / 0.04481
```

Purpose:

```text
Tests whether image-conditioned truncated diffusion refinement improves over direct regression, anchor-only prediction, and deterministic residual refinement.
```

### 9.5 Article table: parameter-matched ablation

Use this table in the article:

```text
Method | Params | Anchors | Diffusion | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE
Direct regression classical | 833,072 | no  | no  | 0.13716 / 0.15706 | 0.13657 / 0.18012 | 0.13826 / 0.11453
Anchor-only                 | --      | yes | no  | 0.14048 / 0.16984 | 0.14914 / 0.18790 | 0.12451 / 0.13654
Anchor + deterministic res. | 835,397 | yes | no  | 0.12302 / 0.15110 | 0.14502 / 0.18252 | 0.08244 / 0.09315
Anchor + truncated diffusion| 835,397 | yes | yes | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481
```

Main improvement:

```text
Anchor + truncated diffusion vs direct regression:
ADE reduction = 65.6%
FDE reduction = 62.9%
```

---

## 10. Article Materials

### 10.1 Main conceptual documents

Use these files to continue writing the paper:

```text
CDAD_article_plan_and_motivation_corrected.md
AIS_reference_corpus_and_CAD_article_plan.md
CDAD_extended_reference_base_50_70_refs.md
```

Approximate local paths:

```text
/mnt/data/CDAD_article_plan_and_motivation_corrected.md
/mnt/data/AIS_reference_corpus_and_CAD_article_plan.md
/mnt/data/CDAD_extended_reference_base_50_70_refs.md
```

What they contain:

```text
- corrected article framing;
- target journal logic for Advanced Intelligent Systems;
- reference corpus;
- section plan;
- figure plan;
- table plan;
- limitation framing;
- contribution list.
```

### 10.2 Manuscript skeleton and references

Use:

```text
CDAD_WileyMSP_AIS_manuscript_skeleton_bibtex.tex
bible.bib
```

Approximate local paths:

```text
/mnt/data/CDAD_WileyMSP_AIS_manuscript_skeleton_bibtex.tex
/mnt/data/bible.bib
```

Current Introduction draft:

```text
CDAD_Introduction_draft.tex
```

Approximate local path:

```text
/mnt/data/CDAD_Introduction_draft.tex
```

Important BibTeX note:

```text
Remove \nocite{*} before final submission.
Verify all AUTHOR LIST TO VERIFY entries.
Compile with:
pdflatex -> bibtex -> pdflatex -> pdflatex
```

---

## 11. What to Provide to a New Consultant

When continuing the manuscript in another dialogue, provide this README plus the following files depending on the task.

### For Dataset section

Provide:

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
wayomo_end_to_end_corridor_dataset_description.md
build_i2nav_robot_corridor_dataset_final_N20_logged-3.ipynb
i2nav_robot_corridor_dataset_technical_overview.md
```

Also provide server paths:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

### For Anchor section

Provide:

```text
build_cross_domain_anchor_trajectories_cdad_planner_cv2_kmeans.ipynb
README_Cross_Domain_Anchors_v2_filtered.md
```

Also provide server path:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

### For Method section

Provide:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
build_cross_domain_anchor_trajectories_cdad_planner_cv2_kmeans.ipynb
```

Useful conceptual files:

```text
CDAD_article_plan_and_motivation_corrected.md
CDAD_detailed_AIS_article_architecture.md
```

### For Experiments / Results section

Provide:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
train_cdad_planner_lite_v3_TRAIN_i2Nav_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
train_cdad_planner_lite_v3_TRAIN_Waymo_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
train_DIRECT_REGRESSION_CLASSICAL_PARAM_MATCHED_same_mixed_split_logged.ipynb
train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged_v2.ipynb
evaluate_cdad_v3_anchor_only_vs_refined_mixed_same_test.ipynb
```

Provide these final tables:

```text
Cross-domain transfer table
Parameter-matched ablation table
```

### For Introduction / Related Work

Provide:

```text
CDAD_Introduction_draft.tex
AIS_reference_corpus_and_CAD_article_plan.md
CDAD_extended_reference_base_50_70_refs.md
bible.bib
```

---

## 12. Immediate Next Steps for the Article

```text
1. Freeze dataset statistics:
   - Waymo samples / train / validation / test
   - i2Nav samples / train / validation / test
   - horizon, step, number of trajectory points

2. Generate final Table 1:
   Dataset statistics.

3. Generate final Table 2:
   Cross-domain transfer comparison.

4. Generate final Table 3:
   Parameter-matched ablation.

5. Export qualitative overlays:
   - Waymo examples
   - i2Nav examples
   - selected anchor vs refined prediction
   - failure cases

6. Write Method section:
   - trajectory representation
   - anchor construction
   - anchor prediction
   - truncated diffusion formulation
   - training objective
   - ADE/FDE metrics

7. Write Experimental Setup:
   - datasets
   - splits
   - baselines
   - implementation details
   - evaluation protocol
```

---

## 13. Short Article Summary

This project studies single-image future-path prediction as a compact path-level representation for autonomous vehicles and ground robots. Both domains are expressed as normalized image-space trajectories. A shared trajectory-anchor vocabulary is built from filtered vehicle and robot trajectories. The proposed Truncated Anchor Diffusion model selects a coarse anchor from visual input and refines it through image-conditioned truncated diffusion. Experiments compare mixed-domain training, single-domain training, direct coordinate regression, anchor-only prediction, deterministic residual refinement, and truncated diffusion refinement. The final results show that truncated diffusion improves substantially over parameter-matched direct regression and deterministic residual baselines on the same mixed vehicle-robot test protocol.
