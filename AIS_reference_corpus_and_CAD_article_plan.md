# Advanced Intelligent Systems: reference corpus and article plan for CDAD

## Target logic

Target journal: **Advanced Intelligent Systems**.

Working article positioning:

> We propose a single-image visual future-path prediction method for autonomous vehicles and ground robots. The method predicts an ordered image-space future trajectory, not a corridor mask, not a drivable-area segmentation, and not a closed-loop control command.

Core message:

> A single RGB image contains enough scene geometry and semantic affordance information to infer a transferable future-path prior across ground autonomous systems.

Do not write that the method predicts or evaluates corridor masks. The method is trajectory-only:

```text
RGB image → ordered future trajectory points → ADE/FDE evaluation
```

---

# 1. Reference papers from Advanced Intelligent Systems

## 1.1 Must-read references for our paper

These are the most important papers to read first because they are closest to our scope: autonomous navigation, path planning, trajectory prediction, mobile robots, and autonomous vehicles.

### 1. Artificial Intelligence in Autonomous Mobile Robot Navigation: From Classical Approaches to Intelligent Adaptation

**Type:** Review  
**Journal:** Advanced Intelligent Systems  
**Year:** 2026  
**DOI:** `10.1002/aisy.202501376`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202501376

**Why it matters for CDAD:**  
This is the strongest scope reference. It frames autonomous mobile robot navigation as a field where AI augments or replaces classical pipelines for planning, localization, perception, and decision-making. It can justify our article as AI-based perception-to-navigation representation learning.

**How to use it:**  
Use in Introduction and Related Work to establish that AI-based navigation is a core topic of the journal and that learning-based navigation representations are relevant.

**Reference role in our article:**  
Background / scope anchor.

---

### 2. UP3: Unsupervised Predictive Path Planner for Mobile Robots in Complex Dynamic Environments

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2025  
**DOI:** `10.1002/aisy.202400916`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/abs/10.1002/aisy.202400916

**Why it matters for CDAD:**  
Very close thematically: predictive path planning for mobile robots. It uses learning to generate predictive paths in partially observable environments.

**How to use it:**  
Use in Related Work as an example of learning-based predictive path planning for mobile robots. Then distinguish our work: we do not train a full planner through environment interaction; we predict an image-space future trajectory from one RGB image and study cross-domain transfer.

**Reference role in our article:**  
Closest mobile-robot planning analogue.

---

### 3. Bidirectional Obstacle Avoidance Enhancement–DDPG: A Novel Algorithm for Mobile-Robot Path Planning in Unknown Dynamic Environments

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2024  
**DOI:** `10.1002/aisy.202300444`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/abs/10.1002/aisy.202300444

**Why it matters for CDAD:**  
Direct mobile-robot path-planning paper in unknown dynamic environments. Shows that the journal accepts algorithmic path-planning contributions for mobile robots.

**How to use it:**  
Use in Related Work to position CDAD relative to reinforcement-learning-based path planning. Our distinction: we are not optimizing a control policy; we predict a visual future-trajectory prior.

**Reference role in our article:**  
Learning-based mobile-robot path planning.

---

### 4. Low Computational Cost for Multiple Waypoints Trajectory Planning: A Time-Optimal-Based Approach

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2025  
**DOI:** `10.1002/aisy.202400363`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202400363

**Why it matters for CDAD:**  
Trajectory planning for mobile robots / autonomous systems. Useful because it demonstrates that explicit trajectory-level representations are acceptable in the journal.

**How to use it:**  
Use to justify trajectory-level planning/path representations as a natural target in intelligent systems.

**Reference role in our article:**  
Trajectory planning reference.

---

### 5. Navigating Intelligence: A Survey of Google OR-Tools and Machine Learning for Global Path Planning in Autonomous Ground Vehicles

**Type:** Survey  
**Journal:** Advanced Intelligent Systems  
**Year:** 2024  
**DOI:** `10.1002/aisy.202300840`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/abs/10.1002/aisy.202300840

**Why it matters for CDAD:**  
A survey on global path planning for autonomous ground vehicles. Very useful for connecting our work to autonomous ground vehicle navigation.

**How to use it:**  
Use as a survey reference in Related Work. Distinguish our work: global planning methods often assume map/graph/task structure; CDAD estimates a local visual future trajectory directly from an image.

**Reference role in our article:**  
Survey / autonomous ground vehicle planning context.

---

### 6. GACNet: Interactive Prediction of Surrounding Vehicles in Safety-Critical Scenarios

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2025  
**DOI:** `10.1002/aisy.202401040`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202401040

**Why it matters for CDAD:**  
Trajectory prediction for autonomous vehicles. Confirms that Advanced Intelligent Systems publishes trajectory-prediction work in the autonomous-driving context.

**How to use it:**  
Use in Related Work under autonomous-vehicle trajectory prediction. Distinguish our target: we predict ego-compatible future path from a single image, not interactive trajectories of surrounding vehicles.

**Reference role in our article:**  
Autonomous-vehicle trajectory prediction analogue.

---

### 7. TMPF: A Two-Stage Merging Planning Framework for Dense Traffic Scenarios

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2023  
**DOI:** `10.1002/aisy.202300081`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202300081

**Why it matters for CDAD:**  
Autonomous-vehicle planning in dense traffic. Shows that vehicle planning is a journal-relevant topic.

**How to use it:**  
Use as a reference for autonomous-vehicle planning. Distinguish our method as perception-to-path prior rather than high-level traffic merging planner.

**Reference role in our article:**  
Autonomous-vehicle planning reference.

---

### 8. Social Interaction-Aware Dynamical Models and Decision-Making for Autonomous Vehicles

**Type:** Research article / decision-making framework  
**Journal:** Advanced Intelligent Systems  
**Year:** 2023  
**DOI:** `10.1002/aisy.202300575`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202300575

**Why it matters for CDAD:**  
Useful for framing decision-making and path planning as key components of autonomous driving.

**How to use it:**  
Use in Related Work for autonomous-vehicle decision-making and planning. CDAD is lower-level and image-based: a visual future-path prior, not a complete decision-making framework.

**Reference role in our article:**  
Autonomous-vehicle decision/planning context.

---

### 9. VAE+DDPG: An Attention-Enhanced Variational Autoencoder for Deep Reinforcement Learning-Based Autonomous Navigation in Low-Light Environments

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2026  
**DOI:** `10.1002/aisy.202500636`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/abs/10.1002/aisy.202500636

**Why it matters for CDAD:**  
Learning-based autonomous navigation from visual/perceptual representations. Useful for connecting our work to image-based navigation.

**How to use it:**  
Use in Related Work as a learning-based navigation paper. Distinguish our focus: future-trajectory prediction from one image and cross-domain transfer.

**Reference role in our article:**  
Visual / neural autonomous navigation.

---

### 10. Autonomous Robotic Colonoscopy: A Supervised Learning Method for Autonomous Navigation

**Type:** Research article  
**Journal:** Advanced Intelligent Systems  
**Year:** 2026  
**DOI:** `10.1002/aisy.202500841`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202500841

**Why it matters for CDAD:**  
Supervised learning for autonomous navigation that predicts target steering points and collision probabilities. This is conceptually close to our target: supervised visual prediction of future navigation cues.

**How to use it:**  
Use as evidence that supervised learning for navigation cues is within AIS scope. Do not overuse it because the domain is medical robotics.

**Reference role in our article:**  
Supervised navigation cue prediction.

---

## 1.2 Secondary references

These are useful but less central.

### 11. Dynamic Motion Planning Model for Multirobot Using Graph Convolutional Networks and Topological Maps

**DOI:** `10.1002/aisy.202300036`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202300036

**Use:** Multi-robot motion planning context. Useful if we mention graph/topological methods, but not central.

---

### 12. Deep Q-Network-Based Hierarchical Path Planning of a Climbing Robot for Complete Coverage

**DOI:** `10.1002/aisy.202400745`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202400745

**Use:** Coverage path planning for robots. Useful as journal-scope evidence.

---

### 13. FMEA-Based Coverage-Path-Planning Strategy for Floor Cleaning Robots

**DOI:** `10.1002/aisy.202300260`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202300260

**Use:** Practical service-robot planning. Useful for proving the journal accepts applied robot-planning papers.

---

### 14. Trajectory Planning for Multiple Autonomous Vehicles at Short-Distance Tandem Signalized Intersections Based on Rule-Free Framework

**DOI:** `10.1002/aisy.202300692`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/abs/10.1002/aisy.202300692

**Use:** AV trajectory planning. Good journal-scope reference, less central than GACNet/TMPF.

---

### 15. Human-Like Interactive Behavior Generation for Autonomous Vehicles

**DOI:** `10.1002/aisy.202100211`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202100211

**Use:** Autonomous-vehicle behavior generation. Use only if discussing behavior prediction/planning broadly.

---

### 16. Human–Machine Mutual Trust Based Shared Control Strategy for Intelligent Vehicles

**DOI:** `10.1002/aisy.202501155`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202501155

**Use:** Intelligent vehicles and shared control. Peripheral, but confirms intelligent-vehicle scope.

---

### 17. Adaptive Autonomy in Microrobot Motion Control via Deep Reinforcement Learning and Classical Path Planning

**DOI:** `10.1002/aisy.202501053`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202501053

**Use:** Shows path planning as part of autonomy across robotic scales. Peripheral.

---

### 18. Independent Control and Path Planning of Microswimmers with a Uniform Magnetic Field

**DOI:** `10.1002/aisy.202100183`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202100183

**Use:** Path planning in microrobotics. Only for broad journal-scope evidence, not for our main Related Work.

---

### 19. Long-Distance Autonomous Navigation of Optical Microrobotic Swarms in Complex Environments

**DOI:** `10.1002/aisy.202470056`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.202470056

**Use:** Autonomous navigation in swarm/microrobotics. Scope evidence only.

---

### 20. Autonomous Navigation of Bio-Intelligent Cyborg Insect Based on Insect Visual Perception

**DOI:** `10.1002/aisy.70131`  
**URL:** https://advanced.onlinelibrary.wiley.com/doi/10.1002/aisy.70131

**Use:** Visual perception-based autonomous navigation. Interesting but peripheral.

---

# 2. Which references to actually read first

Do not read all 20 in full immediately. Read in this order.

## Tier A — mandatory

1. **Artificial Intelligence in Autonomous Mobile Robot Navigation**  
   Use for Introduction and navigation taxonomy.

2. **UP3: Unsupervised Predictive Path Planner for Mobile Robots**  
   Closest planning analogue.

3. **GACNet: Interactive Prediction of Surrounding Vehicles**  
   Closest autonomous-driving trajectory-prediction analogue.

4. **TMPF: A Two-Stage Merging Planning Framework**  
   Strong autonomous-vehicle planning analogue.

5. **Navigating Intelligence: Survey of OR-Tools and ML for Global Path Planning in Autonomous Ground Vehicles**  
   Good survey context for AGV/UGV planning.

## Tier B — read abstracts and figures

6. **Bidirectional Obstacle Avoidance Enhancement–DDPG**  
7. **Low Computational Cost for Multiple Waypoints Trajectory Planning**  
8. **VAE+DDPG for Autonomous Navigation in Low-Light Environments**  
9. **Autonomous Robotic Colonoscopy**  
10. **Trajectory Planning for Multiple Autonomous Vehicles**

## Tier C — only for scope evidence

11. Dynamic Motion Planning Model for Multirobot  
12. Deep Q-Network-Based Hierarchical Path Planning  
13. FMEA-Based Coverage-Path-Planning Strategy  
14. Human-Like Interactive Behavior Generation  
15. Human–Machine Mutual Trust Based Shared Control  
16. Microrobot / microswimmer / cyborg insect navigation papers

---

# 3. What to learn from these papers

## Expected structure in AIS-style articles

Advanced Intelligent Systems articles usually make the paper look like an intelligent-system contribution, not just a neural-network benchmark. Therefore, our paper should emphasize:

1. the autonomous-system problem;
2. the perception-to-navigation representation;
3. the method architecture;
4. experimentally validated behavior;
5. ablation and interpretation;
6. limitations and integration into downstream autonomy stacks.

The paper should not look like:

> “We trained a neural network on two datasets.”

It should look like:

> “We propose a transferable visual future-path prior for autonomous systems, implemented through cross-domain anchors and anchor-conditioned truncated diffusion.”

---

# 4. Recommended article plan

## Section 1. Introduction

Goal: define the problem and sell the motivation.

Order:

1. Autonomous vehicles and ground robots need navigation-relevant visual representations.
2. Full autonomy stacks often use maps, LiDAR, temporal histories, object tracks, dense traversability maps, and control-level planners.
3. There is a need for compact, low-cost, transferable visual priors.
4. We study a deliberately minimal problem: single RGB image → ordered future trajectory in image coordinates.
5. This is not a full planner; it is a visual future-path prior.
6. A trajectory is useful because it encodes direction, curvature, endpoint, and progression.
7. Cross-domain learning is important because vehicle and ground-robot navigation share visual path geometry but differ in camera viewpoint and scene statistics.
8. Introduce CDAD: cross-domain trajectory anchors + truncated diffusion refinement.
9. State contributions.

Recommended contribution list:

1. We formulate single-image future-path prediction as a compact visual trajectory-prior learning problem for ground autonomous systems.
2. We construct a cross-domain trajectory-anchor vocabulary shared by autonomous vehicle and ground-robot data.
3. We propose an anchor-conditioned truncated diffusion model that refines a coarse trajectory anchor into a scene-specific image-space future path.
4. We provide parameter-matched comparisons against direct regression, anchor-only prediction, and deterministic residual refinement.
5. We demonstrate cross-domain transfer across vehicle and ground-robot datasets using matched test splits.

---

## Section 2. Related Work

Use four subsections.

### 2.1 AI-based navigation and path planning for mobile robots

Use:
- Artificial Intelligence in Autonomous Mobile Robot Navigation;
- UP3;
- BOAE-DDPG;
- Low Computational Cost for Multiple Waypoints Trajectory Planning;
- Navigating Intelligence survey.

Message:

> Learning-based navigation and path planning are active topics in AIS. Most works address planner optimization, control policies, or map/global-planning settings. Our work focuses on a minimal visual future-trajectory prior from one image.

### 2.2 Autonomous-vehicle trajectory prediction and planning

Use:
- GACNet;
- TMPF;
- Social Interaction-Aware Dynamical Models;
- Trajectory Planning for Multiple Autonomous Vehicles.

Message:

> AV literature often studies interaction, traffic decision-making, and planning. Our work is different: image-only ego-compatible trajectory prior across vehicle and robot domains.

### 2.3 Visual path representations for autonomous systems

This is the section where we discuss trajectory vs dense visual representations.

Important: do not imply we predict corridor masks.

Message:

> Dense traversability and drivable-area representations are useful, but they are not the target of this paper. We instead learn a compact ordered trajectory that directly encodes path progression.

### 2.4 Anchor-based and diffusion-based trajectory refinement

Use general ML references here, not only AIS. This section must connect our method to:
- anchor-based trajectory representation;
- prototype-based path prediction;
- diffusion refinement;
- residual prediction;
- deterministic versus stochastic refinement.

Message:

> Our diffusion module is not used as a multi-sample trajectory generator; it is used as an anchor-conditioned refinement mechanism for one selected future-path hypothesis.

---

## Section 3. Problem Formulation

Keep this clean and mathematical.

Define:

- input image `I`;
- output trajectory `Y = {(x_1, y_1), ..., (x_M, y_M)}`;
- normalized image coordinates;
- number of future points `M = 8`;
- model `f_theta(I) -> Y_hat`;
- metrics ADE/FDE.

State explicitly:

> No dense mask target is used. The model is trained and evaluated on ordered future trajectories.

Suggested subsections:

### 3.1 Single-image future-path prediction

### 3.2 Cross-domain setting

Explain Waymo vehicle domain and i2Nav ground-robot domain at a high level.

### 3.3 Metrics

ADE and FDE.

---

## Section 4. Method: Cross-Domain Anchor Diffusion

This is the core technical section.

Suggested subsections:

### 4.1 Overview

Describe pipeline:

```text
RGB image → visual encoder → anchor selection → selected trajectory anchor → truncated diffusion refinement → final trajectory
```

Include Figure 1 here.

Figure 1 should show:

- vehicle image and robot image;
- shared encoder;
- cross-domain anchor vocabulary;
- selected anchor;
- truncated diffusion refinement;
- final predicted trajectory.

Do not show many anchors as the main selling point. Show the selected anchor and refinement.

### 4.2 Cross-domain trajectory-anchor vocabulary

Explain how anchors are built from trajectories.

Important phrasing:

> Anchors are coarse trajectory prototypes, not multiple final predictions.

### 4.3 Visual encoder and anchor prediction

Explain the image encoder and anchor head.

### 4.4 Anchor-conditioned truncated diffusion refinement

Explain:

- selected anchor;
- diffusion/refinement steps;
- image-conditioned residual structure;
- output final trajectory.

State explicitly:

> Truncated diffusion is used for refinement of one path hypothesis, not for unrestricted multi-modal sampling.

### 4.5 Training objective

Describe losses:

- trajectory loss;
- anchor loss;
- heat/auxiliary losses if actually used;
- total loss.

Only include losses that exist in the model. Do not invent corridor-mask loss.

---

## Section 5. Experimental Setup

Suggested subsections:

### 5.1 Datasets

- Waymo-derived vehicle trajectory dataset;
- i2Nav-Robot ground-robot trajectory dataset.

Include:
- number of samples;
- train/val/test split;
- horizon;
- image resolution;
- trajectory representation.

### 5.2 Baselines

Use exactly the baselines we have:

1. Direct regression classical;
2. predicted anchor only;
3. anchor + deterministic residual;
4. anchor + truncated diffusion;
5. domain-specific training variants.

### 5.3 Implementation details

Include:
- model size;
- optimizer;
- epochs;
- batch size;
- learning rate;
- parameter counts;
- same split for fair comparison.

### 5.4 Evaluation protocol

Explain:
- ADE/FDE;
- mixed test split;
- per-domain reporting;
- same split identity for transfer comparisons;
- parameter-matched comparison.

---

## Section 6. Results

This should be the strongest part.

### 6.1 Main mixed-domain performance

Show final result table.

### 6.2 Cross-domain transfer analysis

Show mixed vs i2Nav-only vs Waymo-only table.

Main conclusion:

> Mixed-domain training improves robustness across domains.

### 6.3 Parameter-matched ablation

Show direct regression, anchor-only, deterministic residual, diffusion.

Main conclusion:

> The improvement is not explained by model size or anchors alone.

### 6.4 Qualitative analysis

Show examples:

- correct Waymo predictions;
- correct i2Nav predictions;
- deterministic residual vs diffusion examples;
- failure cases.

Avoid showing a large cloud of anchors. Instead show:

```text
RGB image | selected anchor | final refined trajectory | ground truth trajectory
```

This figure directly supports our method.

### 6.5 Failure cases

Discuss:

- visually ambiguous scenes;
- intersections;
- open spaces;
- sharp turns;
- occlusion;
- domain-specific camera geometry.

---

## Section 7. Discussion

This section is important for AIS because it makes the paper look like an intelligent-system contribution rather than a pure benchmark.

Suggested subsections:

### 7.1 Why a single-image trajectory prior is useful

Key message:

> The trajectory is a compact, ordered visual prior between perception and downstream navigation.

### 7.2 Why direct trajectory prediction is different from dense traversability prediction

Careful wording:

> We do not predict dense traversability maps. Instead, we predict an ordered path. Dense maps and path priors are complementary representations.

### 7.3 Why truncated diffusion improves anchor refinement

Use ablation table.

### 7.4 Integration into downstream autonomy stacks

Possible downstream uses:

- planner initialization;
- waypoint prior;
- auxiliary cue for local navigation;
- low-resource robot navigation;
- map-free perception-to-navigation modules;
- safety-filtered planning input.

Do not claim the current method is already safe or control-valid.

---

## Section 8. Limitations and Future Work

Suggested content:

1. The method predicts image-space future trajectories, not control commands.
2. It produces one refined path hypothesis, not a full multi-modal future distribution.
3. It does not include traffic-rule reasoning, collision checking, object-agent interaction, or vehicle dynamics.
4. It does not use closed-loop robot evaluation.
5. The current validation covers two ground-mobility domains.
6. Future work:
   - multi-hypothesis version;
   - closed-loop robot integration;
   - integration with dense traversability;
   - safety filtering;
   - more datasets;
   - temporal input.

Phrase positively:

> These limitations define the intended role of the model as a compact visual path prior rather than a complete autonomy stack.

---

## Section 9. Conclusion

Short conclusion:

1. Single-image future-path prediction is feasible across vehicle and ground-robot domains.
2. Cross-domain anchors provide interpretable coarse prototypes.
3. Truncated diffusion provides strong refinement under matched parameter count.
4. The resulting trajectory prior is a compact intermediate representation for autonomous systems.

---

# 5. Recommended LaTeX structure

For Advanced Intelligent Systems, use a clean article structure. Do not over-fragment into too many subsections. The paper should look like a journal article, not a lab report.

Recommended LaTeX section layout:

```latex
\section{Introduction}

\section{Related Work}
\subsection{AI-Based Navigation and Path Planning for Mobile Robots}
\subsection{Autonomous-Vehicle Trajectory Prediction and Planning}
\subsection{Visual Path Representations for Autonomous Systems}
\subsection{Anchor-Based and Diffusion-Based Trajectory Refinement}

\section{Problem Formulation}
\subsection{Single-Image Future-Path Prediction}
\subsection{Cross-Domain Setting}
\subsection{Evaluation Metrics}

\section{Method}
\subsection{Overview}
\subsection{Cross-Domain Trajectory Anchors}
\subsection{Visual Encoder and Anchor Prediction}
\subsection{Anchor-Conditioned Truncated Diffusion}
\subsection{Training Objective}

\section{Experimental Setup}
\subsection{Datasets}
\subsection{Baselines}
\subsection{Implementation Details}
\subsection{Evaluation Protocol}

\section{Results}
\subsection{Main Mixed-Domain Performance}
\subsection{Cross-Domain Transfer}
\subsection{Parameter-Matched Ablation}
\subsection{Qualitative Analysis}
\subsection{Failure Cases}

\section{Discussion}
\subsection{Trajectory Priors as Intermediate Navigation Representations}
\subsection{Single-Hypothesis Refinement and Practical Use}
\subsection{Integration into Downstream Autonomy Stacks}

\section{Limitations and Future Work}

\section{Conclusion}
```

If the journal format prefers fewer subsections, compress to:

```latex
\section{Introduction}
\section{Related Work}
\section{Method}
\section{Experiments}
\section{Results and Discussion}
\section{Limitations}
\section{Conclusion}
```

But for the first draft, use the detailed structure. It is easier to write and later compress.

---

# 6. Figure plan

## Figure 1 — Method overview

Show:

```text
Vehicle RGB / Robot RGB
       ↓
Shared visual encoder
       ↓
Cross-domain trajectory anchor prediction
       ↓
Selected anchor
       ↓
Truncated diffusion refinement
       ↓
Final image-space future trajectory
```

Purpose:

> Explain method in one figure.

## Figure 2 — Dataset and trajectory representation

Show:

```text
Waymo example with GT trajectory
i2Nav example with GT trajectory
normalized image-space coordinates
```

Purpose:

> Make clear that the target is always trajectory, not mask.

## Figure 3 — Cross-domain anchor vocabulary

Show a compact visualization of representative anchors, but not too many and not as the main claim.

Purpose:

> Show interpretability of coarse trajectory prototypes.

## Figure 4 — Main qualitative results

Show:

```text
RGB | GT trajectory | predicted trajectory | overlay
```

Separate Waymo and i2Nav rows.

## Figure 5 — Ablation qualitative comparison

Show:

```text
direct regression | anchor only | deterministic residual | truncated diffusion | GT
```

Purpose:

> Visually prove diffusion refinement improves.

## Figure 6 — Failure cases

Show ambiguous scenes.

Purpose:

> Make paper honest and reviewer-resistant.

---

# 7. Table plan

## Table 1 — Dataset statistics

Columns:

```text
Dataset | Domain | Samples | Train | Val | Test | Horizon | Points | Coordinate system
```

## Table 2 — Main and transfer results

Use:

```text
Train data | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE
```

## Table 3 — Parameter-matched ablation

Use:

```text
Method | Params | Anchors | Diffusion | Mixed ADE/FDE | Waymo ADE/FDE | i2Nav ADE/FDE
```

## Table 4 — Model/component summary

Optional.

Columns:

```text
Model | Visual encoder | Anchor head | Refinement type | Trainable parameters
```

---

# 8. Writing order

Do not write from Introduction first. Recommended order:

1. Method.
2. Experimental Setup.
3. Results tables.
4. Figure captions.
5. Discussion.
6. Related Work.
7. Introduction.
8. Abstract.
9. Conclusion.
10. Cover letter.

This avoids empty motivation and keeps claims grounded in actual experiments.

---

# 9. Final article story in one paragraph

This paper studies single-image future-path prediction as a compact visual prior for autonomous vehicles and ground robots. Instead of predicting dense traversability maps or solving closed-loop planning, the method predicts an ordered trajectory in normalized image coordinates. A shared cross-domain trajectory-anchor vocabulary provides coarse path prototypes, and an anchor-conditioned truncated diffusion module refines the selected prototype into a scene-specific future path. Experiments on vehicle and ground-robot data show that mixed-domain training improves transfer robustness and that truncated diffusion strongly outperforms direct regression, anchor-only prediction, and deterministic residual refinement under matched parameter counts. The result is a transferable visual trajectory prior that can serve as an interpretable intermediate representation for downstream autonomous navigation systems.

