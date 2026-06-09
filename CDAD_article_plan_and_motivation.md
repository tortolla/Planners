# Article Plan and Motivation
## Working title
**Cross-Domain Anchor Diffusion for Visual Future-Path Prediction in Autonomous Vehicles and Ground Robots**

Alternative title variants:

1. **Single-Image Visual Future-Path Prediction for Autonomous Vehicles and Ground Robots via Cross-Domain Anchor Diffusion**
2. **Learning a Cross-Domain Visual Path Prior for Autonomous Vehicles and Ground Robots**
3. **Anchor-Conditioned Truncated Diffusion for Image-Space Future-Path Prediction Across Ground Mobility Domains**

---

# 1. Core thesis of the paper

The paper should not be framed as a full autonomous-driving planner or as a complete robot-control system. The strongest framing is:

> We study whether a compact future-path prior can be inferred from a single RGB image and transferred across two ground mobility domains: vehicle scenes and ground-robot navigation scenes.

The key contribution is not “trajectory planning” in the classical sense. The key contribution is **visual future-path prior learning**:

- input: one RGB image;
- output: an ordered image-space future path;
- no HD map;
- no LiDAR;
- no temporal history;
- no object tracks;
- no explicit traffic-rule reasoning;
- no closed-loop control;
- two domains: autonomous vehicle scenes and ground-robot scenes;
- method: shared visual encoder, cross-domain anchor vocabulary, and truncated diffusion-based refinement.

The central claim:

> A single RGB image contains enough geometric and semantic structure to infer a useful centerline-like future-path hypothesis, and this representation can be shared across autonomous vehicles and ground robots.

---

# 2. Why this problem is practically important

Most autonomous-navigation pipelines rely on one of two dense representations:

1. **drivable-area / traversability masks** — where motion may be possible;
2. **map- or rule-based planners** — how the agent should move under constraints.

These are powerful, but they have limitations:

- dense masks are spatially informative but not directionally ordered;
- a drivable region does not directly encode the future path centerline, curvature, endpoint, or temporal progression;
- maps, LiDAR, object tracks, and historical trajectories are not always available;
- mobile robots, temporary platforms, low-cost systems, and field robots often operate in map-poor or sensor-limited settings;
- a full planner is expensive to deploy and difficult to transfer across domains.

This paper targets a deliberately minimal but practically relevant setting:

> Given only the current RGB image, infer the most likely ego-compatible direction of motion as a compact path prior.

The output is useful because it can serve as:

- a centerline-like prior for downstream navigation;
- an auxiliary cue for drivable-area segmentation;
- a low-dimensional representation of scene affordance;
- a visual initialization for a downstream planner;
- a compact representation for low-resource or map-free autonomous systems;
- a cross-domain bridge between vehicle navigation and ground-robot navigation.

The practical motivation is therefore not “replace the full planner,” but:

> provide a compact, transferable, vision-only path prior that can support downstream planning, segmentation, control, or decision modules.

---

# 3. The key conceptual risk and how to close it

## Reviewer concern

A reviewer may object:

> Why predict a single trajectory from one image? Navigation usually requires a corridor, a distribution of possible paths, or a full planner. A single trajectory may look overconfident.

This is the main conceptual risk.

## Correct answer

The predicted path should not be presented as a final executable plan. It should be presented as a **centerline-like visual future-path prior**.

The paper should consistently use language such as:

- visual future-path prior;
- image-space path prior;
- centerline-like trajectory hypothesis;
- ego-compatible path hypothesis;
- compact navigational representation;
- path-level affordance representation.

Avoid language such as:

- final plan;
- safe trajectory;
- full planner;
- autonomous-driving planner;
- control command;
- optimal trajectory.

Recommended statement:

> The predicted trajectory is not a control-level plan. It is a compact image-space path prior that summarizes the most likely ego-compatible direction of motion from visual scene geometry.

This turns the “single trajectory” issue from a weakness into a strength:

- one ordered path is compact;
- it encodes direction and progression;
- it can be converted into a corridor if needed;
- it is easier to evaluate with ADE/FDE;
- it can act as an interpretable intermediate representation.

---

# 4. Trajectory versus corridor: how to position the output

Dense corridor masks answer:

> Where can motion possibly occur?

The proposed trajectory prior answers:

> What is the most likely directionally ordered future path of the ego agent?

These are complementary, not competing, representations.

A drivable corridor may include many possible pixels: adjacent lanes, road shoulders, open floor regions, sidewalks, or side passages. It does not necessarily indicate the centerline, curvature, or endpoint of likely motion. A trajectory prior compresses this into an ordered path structure.

Recommended paragraph:

> Dense traversability or drivable-area masks describe where motion may be possible, but they do not directly provide a directionally ordered path hypothesis. In contrast, an image-space future path encodes a compact centerline-like representation with explicit progression, curvature, and endpoint information. The proposed representation is therefore complementary to corridor-style perception: when a dense cue is required, the predicted path can be rasterized and widened into a soft corridor prior.

This is important because the model uses truncated diffusion to refine one selected path hypothesis, not to generate many stochastic futures. Therefore, the article should not claim full multi-modal trajectory generation. It should claim **single-hypothesis path-prior refinement**.

---

# 5. Why single-image prediction is a strength, not a limitation

The paper should explicitly state that the input restriction is deliberate.

Wrong framing:

> We do not use HD maps, LiDAR, temporal history, or control validation.

Correct framing:

> We deliberately isolate the visual path-prior problem by using only a single RGB image.

This allows the paper to ask a clean scientific question:

> How much future-path structure can be inferred from static visual scene geometry alone?

This is relevant because one image contains:

- lane geometry;
- corridor geometry;
- free-space layout;
- perspective cues;
- vanishing direction;
- floor/road boundaries;
- obstacles;
- scene semantics;
- affordances for motion.

Recommended paragraph:

> Unlike map-based and history-based autonomous-navigation pipelines, this work studies a deliberately minimal visual setting. The model receives only a single RGB image and predicts an image-space future-path prior. This design isolates the contribution of visual scene understanding and enables cross-domain transfer between vehicle and ground-robot data.

---

# 6. Why truncated diffusion makes sense here

The model should not be sold as a full generative multi-trajectory diffusion planner. That would invite the wrong comparison.

The correct interpretation is:

> Truncated diffusion is used as a learned refinement mechanism for an anchor-conditioned path hypothesis.

The method has three stages:

1. a visual encoder extracts scene representation;
2. an anchor head selects a coarse cross-domain path prototype;
3. a truncated diffusion/refinement module improves the selected path toward the final future-path prior.

This has a clear motivation:

- anchor-only prediction is too coarse;
- deterministic residual refinement improves the anchor but remains limited;
- truncated diffusion gives a stronger learned refinement mechanism under the same parameter budget;
- the improvement is not due to parameter count because deterministic residual and diffusion have the same parameter count.

Recommended method framing:

> We use truncated diffusion not as an unrestricted sampler of multiple futures, but as a parameter-matched refinement process conditioned on a selected visual anchor. This design preserves interpretability through the anchor vocabulary while allowing the model to correct coarse path prototypes using image-conditioned residual structure.

---

# 7. Main experimental story

The experimental story should be simple and strong.

## Experiment 1: Main mixed-domain result

Train on mixed vehicle + ground-robot data and test on the same mixed test split.

Main result:

| Model | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---:|---:|
| Anchor + truncated diffusion | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |

Message:

> The proposed model predicts accurate image-space future paths across both vehicle and ground-robot domains.

## Experiment 2: Cross-domain training comparison

Compare mixed-domain training against domain-specific training.

| Train data | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---:|---:|
| Mixed | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |
| i2Nav-only | 0.06901 / 0.09578 | 0.08389 / 0.12159 | 0.04157 / 0.04818 |
| Waymo-only | 0.05238 / 0.06931 | 0.05093 / 0.06607 | 0.05507 / 0.07528 |

Message:

> Mixed-domain training improves cross-domain robustness, especially on the opposite domain.

## Experiment 3: Parameter-matched ablation

This is the strongest methodological table.

| Method | Params | Anchors | Diffusion | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE |
|---|---:|---|---|---:|---:|---:|
| Direct regression classical | 833,072 | no | no | 0.13716 / 0.15706 | 0.13657 / 0.18012 | 0.13826 / 0.11453 |
| Predicted anchor only | — | yes | no | 0.14048 / 0.16984 | 0.14914 / 0.18790 | 0.12451 / 0.13654 |
| Anchor + deterministic residual | 835,397 | yes | no | 0.12302 / 0.15110 | 0.14502 / 0.18252 | 0.08244 / 0.09315 |
| Anchor + truncated diffusion | 835,397 | yes | yes | 0.04719 / 0.05829 | 0.05117 / 0.06559 | 0.03985 / 0.04481 |

Message:

> The gain is not explained by parameter count or by the anchor vocabulary alone. The truncated diffusion refinement is the decisive component.

This is a strong argument for publication.

---

# 8. What exactly is novel

The novelty should not be exaggerated as “a new autonomous-driving planner.” The novelty is more precise:

1. **A cross-domain image-space future-path prediction problem** spanning autonomous vehicle and ground-robot domains.
2. **A shared anchor vocabulary** of future paths across both domains.
3. **Anchor-conditioned truncated diffusion** as a refinement mechanism for a single visual path hypothesis.
4. **Parameter-matched ablation** against direct regression and deterministic residual refinement.
5. **Evidence that single-image visual path priors can transfer across ground mobility domains.**

Recommended contribution list:

> The contributions of this work are:
>
> 1. We formulate single-image visual future-path prediction as a compact path-prior learning problem across autonomous vehicle and ground-robot domains.
> 2. We introduce a cross-domain anchor vocabulary that represents shared image-space path prototypes.
> 3. We propose an anchor-conditioned truncated diffusion refinement model that converts a coarse visual path prototype into an accurate future-path prior.
> 4. We provide parameter-matched comparisons against direct regression, anchor-only prediction, and deterministic residual refinement.
> 5. We demonstrate cross-domain transfer between vehicle and ground-robot datasets under matched test splits.

---

# 9. Suggested article structure

## Abstract

The abstract should contain:

1. problem: single-image future-path prediction;
2. motivation: compact visual prior for autonomous systems;
3. method: cross-domain anchors + truncated diffusion;
4. domains: vehicle and ground robot;
5. baselines: direct regression, anchor-only, deterministic residual;
6. key result: diffusion substantially improves ADE/FDE;
7. scope: path prior, not full closed-loop planner.

## Introduction

Recommended flow:

1. Autonomous systems need scene-level navigation cues.
2. Dense traversability masks and full planners are useful but have limitations.
3. A compact future-path prior is a useful intermediate representation.
4. Single-image prediction is minimal but practically relevant.
5. Cross-domain transfer between vehicles and robots is underexplored.
6. Proposed method: Cross-Domain Anchor Diffusion.
7. Contributions.

## Related Work

Sections:

1. Visual navigation and path prediction.
2. Drivable-area/traversability representations.
3. Learning-based planning for autonomous vehicles.
4. Mobile robot path planning and navigation.
5. Diffusion models for trajectory/path refinement.
6. Anchor-based trajectory representation.

## Method

Sections:

1. Problem formulation.
2. Cross-domain datasets and image-space trajectory representation.
3. Anchor vocabulary construction.
4. Visual encoder.
5. Anchor selection head.
6. Truncated diffusion refinement.
7. Losses and training objective.
8. Trajectory-to-corridor interpretation, optional conceptual subsection.

## Experiments

Sections:

1. Datasets and splits.
2. Metrics: ADE/FDE.
3. Main mixed-domain performance.
4. Cross-domain training study.
5. Parameter-matched ablations.
6. Qualitative visualization.
7. Failure cases.

## Discussion

Sections:

1. Why a trajectory prior is useful.
2. Trajectory versus corridor.
3. Why single-image input is meaningful.
4. Role of truncated diffusion.
5. Integration into downstream navigation stacks.

## Limitations

Should be honest but not self-destructive:

- The model predicts an image-space path prior, not a closed-loop control command.
- It does not model traffic rules, dynamic agents, map constraints, or collision checking.
- It currently produces a single refined path hypothesis, not a full multi-modal distribution.
- Evaluation is limited to two ground-mobility domains.
- Future work should integrate the prior with dense traversability, closed-loop robot control, and multi-hypothesis generation.

## Conclusion

Return to the central thesis:

> Single RGB images can support a transferable visual future-path prior across vehicle and ground-robot domains, and anchor-conditioned truncated diffusion provides an effective refinement mechanism under parameter-matched comparisons.

---

# 10. Suggested abstract draft

Autonomous vehicles and mobile robots require compact scene-level cues that connect visual perception to downstream navigation. While dense traversability maps indicate where motion may be possible, they do not directly provide an ordered future-path hypothesis. This paper studies a deliberately minimal setting: predicting an image-space future-path prior from a single RGB image, without HD maps, LiDAR, temporal history, or control-level supervision. We propose Cross-Domain Anchor Diffusion, an anchor-conditioned truncated diffusion framework for visual future-path prediction across vehicle and ground-robot domains. The method first maps an input image to a shared cross-domain vocabulary of path anchors and then refines the selected path hypothesis through a truncated diffusion process. Experiments on vehicle and ground-robot datasets show that mixed-domain training improves transfer robustness. Under a parameter-matched comparison, the proposed truncated diffusion model substantially outperforms direct coordinate regression, anchor-only prediction, and deterministic residual refinement. These results suggest that single-image visual scene structure contains transferable path-level information and that compact future-path priors can serve as interpretable intermediate representations for autonomous navigation systems.

---

# 11. Suggested introduction skeleton

Autonomous vehicles and mobile robots rely on perception modules that convert raw sensory input into navigation-relevant representations. Existing systems often use dense drivable-area maps, object-level scene understanding, HD maps, temporal histories, or rule-aware planning stacks. These components are essential in full autonomy pipelines, but they also introduce cost, complexity, and domain specificity. In many settings, especially for mobile robots, field platforms, low-cost autonomous systems, or map-poor environments, it is useful to infer a compact visual cue that summarizes the likely future direction of motion.

This work studies a minimal but practically relevant question: can a single RGB image support future-path prediction across different ground mobility domains? Instead of attempting to solve closed-loop planning, we focus on an intermediate representation: an image-space future-path prior. Such a prior is a centerline-like ordered path hypothesis that captures scene geometry, perspective, free-space layout, and likely ego-compatible motion. It is complementary to dense corridor or traversability representations. A corridor indicates where motion may be possible, whereas an ordered path prior encodes direction, curvature, endpoint, and progression.

We propose Cross-Domain Anchor Diffusion, a framework for single-image visual future-path prediction across autonomous vehicle and ground-robot scenes. The method uses a shared visual encoder, a cross-domain path-anchor vocabulary, and an anchor-conditioned truncated diffusion module. The anchor vocabulary provides interpretable coarse path prototypes, while the truncated diffusion module refines the selected hypothesis into an accurate future-path prior. Importantly, diffusion is used here as a refinement mechanism rather than as an unrestricted multi-trajectory sampler.

We evaluate the method on two domains: vehicle scenes and ground-robot navigation scenes. The experiments show that mixed-domain training improves cross-domain robustness and that the proposed model outperforms direct regression, anchor-only prediction, and deterministic residual refinement. Because the deterministic residual and diffusion variants have matched parameter counts, the improvement cannot be attributed simply to model capacity. The results support the view that single-image visual structure contains transferable path-level information that can bridge perception and navigation.

---

# 12. Cover-letter motivation for Advanced Intelligent Systems

Recommended positioning:

> The manuscript fits Advanced Intelligent Systems because it addresses AI-based perception-to-navigation representation learning for autonomous systems. Recent articles in the journal have covered autonomous mobile robot navigation, predictive path planning, autonomous vehicle trajectory planning, and learning-based navigation. Our manuscript contributes to this line by proposing a cross-domain visual future-path prior for autonomous vehicles and ground robots, with parameter-matched evidence that anchor-conditioned truncated diffusion improves over direct regression and deterministic refinement.

Use this only in the cover letter, not necessarily in the article body.

---

# 13. Final recommended framing

The safest and strongest framing is:

> This is not a full planner. It is a compact visual path-prior model.

But it should not sound apologetic. It should sound intentional:

> We deliberately remove maps, temporal history, LiDAR, and control-level supervision to isolate the path information contained in a single RGB image.

The final article should be built around this sentence:

> A single RGB image can provide a transferable centerline-like future-path prior for different ground autonomous systems.

This is the core message.
