# CDAD Article Plan and Motivation — Corrected Framing

## Working title

**Cross-Domain Anchor Diffusion for Single-Image Future-Path Prediction in Autonomous Vehicles and Ground Robots**

Alternative variants:

1. **Cross-Domain Anchor Diffusion for Visual Future-Path Prediction Across Vehicle and Ground-Robot Domains**
2. **Learning a Transferable Visual Future-Path Prior from Single RGB Images**
3. **Anchor-Conditioned Truncated Diffusion for Image-Space Trajectory Prediction in Ground Autonomous Systems**

---

# 1. Core correction

The paper must not imply that we train on, predict, or evaluate **corridor masks**.

The actual task is:

> Given a single RGB image, predict an ordered future trajectory in image coordinates.

The model is always trained on trajectories. The supervision is trajectory-based. The evaluation is trajectory-based. The main metrics are ADE/FDE. Corridor masks are not part of the method and should not be described as an experimental object.

The correct contrast is conceptual only:

- many navigation papers use drivable-area / traversability / corridor-like representations;
- our paper instead studies **direct image-space future-trajectory prediction**;
- the predicted trajectory is not a dense mask and not a full closed-loop plan;
- it is a compact visual future-path prior.

Recommended wording:

> Unlike dense traversability or drivable-area prediction, we directly learn an ordered image-space future trajectory. This trajectory is interpreted as a compact visual future-path prior rather than as a full control-level plan.

---

# 2. What exactly the paper studies

The paper studies a deliberately minimal perception-to-trajectory problem:

> Can a single RGB image provide enough visual structure to predict a plausible future path of motion across different ground mobility domains?

Input:

- one RGB image.

Output:

- an ordered sequence of future image-space points.

Not used:

- HD maps;
- LiDAR;
- object tracks;
- temporal image history;
- vehicle dynamics;
- rule-based planning;
- closed-loop control;
- dense corridor masks.

The scientific idea:

> A single image contains geometric and semantic cues — road/floor layout, perspective, boundaries, obstacles, visible free space, corridor direction, and scene affordances — that can support a compact future-trajectory hypothesis.

---

# 3. Why predicting a trajectory directly is useful

A reviewer may ask why the model predicts a single trajectory rather than a dense traversability region or multiple candidate paths.

The answer should be:

> The goal is not to enumerate all possible motions. The goal is to learn a compact, ordered, image-space path hypothesis that captures the most likely ego-compatible direction of motion from visual scene structure.

A trajectory has properties that a dense mask does not explicitly provide:

- ordering along the future path;
- directionality;
- curvature;
- endpoint;
- progression from near future to farther future;
- compactness;
- direct compatibility with ADE/FDE evaluation;
- direct use as an intermediate navigation cue.

This is the practical motivation:

> A predicted future trajectory can serve as a lightweight intermediate representation between perception and downstream navigation. It does not replace planning or control, but it gives a downstream system a compact guess of where the agent is visually likely to move.

Recommended paragraph:

> Dense scene representations are useful for identifying potentially traversable regions, but they do not directly encode an ordered future motion hypothesis. In this work, we study a complementary representation: a compact image-space trajectory predicted directly from a single RGB image. The trajectory encodes direction, curvature, endpoint, and path progression, making it suitable as an intermediate visual prior for downstream navigation modules.

Important: do not write that we compute or evaluate corridor masks.

---

# 4. How to handle the “single trajectory” concern

The model predicts one refined trajectory. It does not generate a distribution of many trajectories.

This is not a bug if stated correctly.

Wrong claim:

> The model generates multiple possible future trajectories.

Correct claim:

> The model refines a single anchor-conditioned future-path hypothesis.

The role of truncated diffusion is:

> not multi-modal generation, but learned refinement of one selected path hypothesis.

Recommended formulation:

> We use truncated diffusion as a refinement mechanism rather than as an unrestricted multi-sample generator. The model first selects a coarse cross-domain path anchor and then refines this anchor into a more accurate image-space future trajectory.

This is consistent with the experiments:

- anchor-only is weak;
- deterministic residual improves the anchor but remains limited;
- truncated diffusion gives a large improvement under matched parameter count;
- therefore the value of diffusion is in the refinement dynamics, not in producing many trajectories.

---

# 5. Main motivation for Advanced Intelligent Systems

Advanced Intelligent Systems is appropriate because the paper is about:

- AI for autonomous systems;
- perception-to-navigation representation learning;
- mobile robots and autonomous vehicles;
- visual trajectory prediction;
- path-level priors;
- cross-domain generalization.

The paper should be positioned as:

> a learning-based visual future-path prediction method for autonomous vehicles and ground robots.

Not as:

> an autonomous-driving planner;
> a robot controller;
> a segmentation method;
> a dense traversability estimator.

The strongest scope sentence:

> This work contributes to AI-based autonomous navigation by learning a transferable image-space future-trajectory prior from a single RGB image across vehicle and ground-robot domains.

---

# 6. Problem statement

Let an RGB image be denoted as `I`. The goal is to predict an ordered future trajectory:

`Y = {(x_1, y_1), ..., (x_M, y_M)}`

in normalized image coordinates.

The model learns:

`f_theta(I) -> Y_hat`

where `Y_hat` is the predicted future path.

Evaluation uses trajectory metrics:

- Average Displacement Error, ADE;
- Final Displacement Error, FDE.

No dense mask target is required.

Recommended text:

> Given a single RGB image, the task is to predict an ordered sequence of future image-space points. This formulation treats visual navigation as a compact trajectory-prediction problem and evaluates the model using ADE and FDE.

---

# 7. Method narrative

The method should be explained in four steps.

## Step 1. Visual encoding

A CNN encoder extracts a scene representation from the RGB image.

## Step 2. Cross-domain anchor vocabulary

A shared anchor vocabulary is built from trajectory shapes across both vehicle and ground-robot data.

The anchors represent coarse future-path prototypes.

Do not oversell anchors as multiple final predictions. They are prototypes.

## Step 3. Anchor selection

The model predicts which anchor best matches the input scene.

## Step 4. Truncated diffusion refinement

The selected anchor is refined by an image-conditioned truncated diffusion module.

The output is one final trajectory.

Recommended method sentence:

> The anchor vocabulary provides an interpretable coarse trajectory prior, while truncated diffusion refines this prior into a scene-specific future path.

---

# 8. Experimental story

## 8.1 Main result

The proposed mixed-domain model achieves:

| Model | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---:|---:|
| Anchor + truncated diffusion | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |

Interpretation:

> The model predicts accurate image-space future trajectories in both vehicle and ground-robot domains.

## 8.2 Cross-domain training study

| Train data | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---:|---:|
| Mixed | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |
| i2Nav-only | 0.06901 / 0.09578 | 0.08389 / 0.12159 | 0.04157 / 0.04818 |
| Waymo-only | 0.05238 / 0.06931 | 0.05093 / 0.06607 | 0.05507 / 0.07528 |

Interpretation:

> Mixed-domain training improves robustness across vehicle and robot domains.

## 8.3 Parameter-matched ablation

| Method | Params | Anchors | Diffusion | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---|---|---:|---:|---:|
| Direct regression classical | 833,072 | no | no | 0.13716 / 0.15706 | 0.13657 / 0.18012 | 0.13826 / 0.11453 |
| Predicted anchor only | — | yes | no | 0.14048 / 0.16984 | 0.14914 / 0.18790 | 0.12451 / 0.13654 |
| Anchor + deterministic residual | 835,397 | yes | no | 0.12302 / 0.15110 | 0.14502 / 0.18252 | 0.08244 / 0.09315 |
| Anchor + truncated diffusion | 835,397 | yes | yes | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |

Interpretation:

> The gain is not due to parameter count, direct coordinate regression, or the anchor vocabulary alone. The truncated diffusion refinement is the decisive component.

---

# 9. Contribution list

Recommended contribution list:

1. We formulate single-image future-path prediction as a compact visual trajectory-prior learning problem for ground autonomous systems.
2. We construct a cross-domain trajectory-anchor vocabulary shared by autonomous vehicle and ground-robot data.
3. We propose an anchor-conditioned truncated diffusion model that refines a coarse trajectory anchor into a scene-specific image-space future path.
4. We provide a parameter-matched comparison against direct regression and deterministic residual refinement.
5. We demonstrate cross-domain transfer across vehicle and ground-robot datasets using matched test splits.

---

# 10. Corrected abstract draft

Autonomous vehicles and mobile robots require compact visual representations that connect perception to downstream navigation. This paper studies a deliberately minimal setting: predicting an ordered future trajectory in image coordinates from a single RGB image, without HD maps, LiDAR, temporal history, object tracks, dense corridor masks, or control-level supervision. We propose Cross-Domain Anchor Diffusion, an anchor-conditioned truncated diffusion framework for visual future-path prediction across vehicle and ground-robot domains. The method maps an input image to a shared vocabulary of cross-domain trajectory anchors and refines the selected anchor into a scene-specific future path. Truncated diffusion is used as a refinement mechanism for a single path hypothesis, rather than as an unrestricted multi-trajectory generator. Experiments on vehicle and ground-robot datasets show that mixed-domain training improves cross-domain robustness. Under parameter-matched comparisons, the proposed model substantially outperforms direct coordinate regression, anchor-only prediction, and deterministic residual refinement. These results suggest that single-image visual structure contains transferable future-path information and that compact trajectory priors can serve as interpretable intermediate representations for autonomous navigation systems.

---

# 11. Corrected introduction skeleton

Autonomous vehicles and mobile robots rely on perception modules that transform raw sensory input into navigation-relevant representations. Many systems use dense traversability maps, HD maps, temporal histories, object tracks, or rule-aware planning modules. These components are important for full autonomy, but they also introduce additional sensing requirements, domain-specific assumptions, and system complexity.

This work studies a more minimal question: can a single RGB image support future-trajectory prediction across different ground mobility domains? Rather than predicting a dense traversability map or solving closed-loop planning, we focus on a compact intermediate representation: an ordered image-space future trajectory. Such a trajectory encodes direction, curvature, endpoint, and path progression, and can be interpreted as a visual future-path prior for downstream navigation.

We propose Cross-Domain Anchor Diffusion, a framework for single-image future-path prediction across autonomous vehicle and ground-robot scenes. The method combines a shared visual encoder, a cross-domain vocabulary of trajectory anchors, and an anchor-conditioned truncated diffusion module. The anchor vocabulary provides coarse trajectory prototypes, while the truncated diffusion module refines the selected anchor into a scene-specific future path.

The proposed method is evaluated on vehicle and ground-robot datasets. Mixed-domain training improves cross-domain robustness, and parameter-matched ablations show that truncated diffusion refinement substantially outperforms direct coordinate regression, anchor-only prediction, and deterministic residual refinement. These results indicate that single-image visual structure can provide transferable trajectory-level information across ground autonomous systems.

---

# 12. Limitations — corrected

The limitations should not mention corridor masks as missing experimental targets.

Correct limitations:

- The model predicts an image-space future trajectory, not a closed-loop control command.
- The current version predicts a single refined path hypothesis, not a full multi-modal distribution.
- It does not include traffic-rule reasoning, dynamic-agent forecasting, collision checking, vehicle dynamics, or map constraints.
- The evaluation covers two ground mobility domains, not all autonomous-navigation settings.
- Future work may integrate the visual path prior with downstream planners, robot controllers, dense traversability modules, or multi-hypothesis prediction.

Recommended wording:

> The proposed trajectory should be interpreted as a visual future-path prior rather than as an executable control command. It is intended to complement downstream planning and control modules, not to replace them.

---

# 13. One-sentence final positioning

> We learn a transferable, single-image, image-space future trajectory prior across autonomous vehicle and ground-robot domains, using cross-domain anchors and truncated diffusion-based refinement.

This is the cleanest framing.
