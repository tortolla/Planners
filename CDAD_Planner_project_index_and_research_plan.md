# CDAD-Planner-Lite: карта проекта, файлов, экспериментов и дальнейший план

Этот файл нужен как быстрый навигатор по проекту: где лежат датасеты, якоря, ноутбуки, результаты экспериментов, какие выводы уже получены и что делать дальше для статьи.

---

# 1. Главная идея работы

Мы строим компактную модель для предсказания будущей траектории по одному RGB-изображению.

Короткая формулировка:

```text
Single RGB image
→ shared visual encoder
→ cross-domain image-space anchor vocabulary
→ trajectory refinement
→ future path in normalized image coordinates
```

Главная научная идея:

```text
Можно ли построить общий image-space future-path representation,
который работает и для автомобилей, и для наземных роботов?
```

Мы не заявляем полноценный safety planner для автономного вождения. Правильная постановка:

```text
CDAD-Planner-Lite predicts a visual future-path prior from a single RGB image.
The output is a learned image-space trajectory prior, not a rule-validated control command.
```

То есть модель предсказывает не “куда обязана ехать машина по ПДД”, а визуальный prior будущего пути, который потом может быть передан в safety/rule-aware planner.

---

# 2. Основной посыл статьи

## 2.1. Проблема

Обычно модели для автомобилей и мобильных роботов обучаются отдельно:

```text
driving datasets → driving planners
robot navigation logs → robot navigation models
```

Из-за этого непонятно, можно ли построить общий визуальный future-path representation для разных наземных embodied domains.

## 2.2. Предлагаемое решение

Мы вводим:

```text
Cross-Domain Anchor Diffusion Planner / CDAD-Planner-Lite
```

Компоненты:

```text
1. Shared normalized image-space trajectory format.
2. Cross-domain K=20 anchor vocabulary from Waymo + i2Nav-Robot.
3. Lightweight visual encoder.
4. Anchor classifier.
5. Truncated diffusion trajectory refinement.
```

## 2.3. Главные доказательства, которые уже получены

### Аргумент 1. Cross-domain training нужен

Мы обучили одну и ту же модель на трех режимах:

```text
1. Mixed: Waymo + i2Nav-Robot
2. i2Nav-only
3. Waymo-only
```

Тестовый split везде один и тот же:

```text
Test = same mixed test split
Waymo = 970 samples
i2Nav-Robot = 526 samples
Total = 1496 samples
```

Главная таблица:

```text
Train data     Test Mixed ADE/FDE      Test Waymo ADE/FDE      Test i2Nav ADE/FDE
---------------------------------------------------------------------------------
Mixed          0.04719 / 0.05829       0.05117 / 0.06559       0.03985 / 0.04481
i2Nav-only     0.06901 / 0.09578       0.08389 / 0.12159       0.04157 / 0.04818
Waymo-only     0.05238 / 0.06931       0.05093 / 0.06607       0.05507 / 0.07528
```

Вывод:

```text
Mixed training дает лучший общий результат.
Robot-only плохо переносится на Waymo.
Waymo-only переносится на i2Nav умеренно, но хуже mixed.
Mixed почти не портит Waymo и улучшает i2Nav.
```

### Аргумент 2. Diffusion/refinement не выигрывает за счет числа параметров

Мы сравнили:

```text
Anchor + deterministic residual refinement
Anchor + truncated diffusion refinement
```

Число параметров одинаковое:

```text
Deterministic residual params = 835,397
Truncated diffusion params    = 835,397
Ratio                         = 1.0
```

Значит reviewer не может объяснить выигрыш diffusion тем, что модель просто больше.

### Аргумент 3. Truncated diffusion лучше deterministic residual baseline

Ablation table:

```text
Method                              Test Mixed ADE/FDE      Test Waymo ADE/FDE      Test i2Nav ADE/FDE
------------------------------------------------------------------------------------------------------
Predicted anchor only               0.14048 / 0.16984       0.14914 / 0.18790       0.12451 / 0.13654
Anchor + deterministic residual     0.12302 / 0.15110       0.14502 / 0.18252       0.08244 / 0.09315
Anchor + truncated diffusion        0.04719 / 0.05829       0.05117 / 0.06559       0.03985 / 0.04481
```

Вывод:

```text
Anchor-only слабый.
Deterministic residual лучше anchor-only, но намного хуже truncated diffusion.
Truncated diffusion дает сильнейшее улучшение при том же числе параметров.
```

---

# 3. Основная рабочая директория проекта

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset
```

Внутри нее лежат:

```text
external_datasets/
prepared_camera_corridor_dataset_filtered_front_v1/
prepared_i2nav_robot_corridor_dataset_v1_2_sec/
cross_domain_anchor_trajectories_cdad_planner_v1/
cross_domain_anchor_trajectories_cdad_planner_v2_filtered/
experiments/
```

---

# 4. Исходные и подготовленные датасеты

## 4.1. Waymo corridor dataset

Корень:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1
```

Связанный ноутбук создания:

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
```

Описание датасета:

```text
wayomo_end_to_end_corridor_dataset_description.md
```

Смысл:

```text
Из Waymo берутся фронтальные изображения.
Будущая trajectory проецируется в image-space.
Создаются:
- RGB image
- corridor mask
- centerline mask
- gaussian heatmap
- projected future trajectory
```

Этот датасет используется как vehicle domain.

## 4.2. i2Nav-Robot corridor dataset

Корень:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_i2nav_robot_corridor_dataset_v1_2_sec
```

Связанный ноутбук создания:

```text
build_i2nav_robot_corridor_dataset_final_N20_logged
```

Для N=10 была создана версия:

```text
dataset_name = i2nav_robot_corridor_leftcam_v2_2sec_N10
```

Итоговый размер:

```text
6511 samples
train = 5254
val   = 175
test  = 1082
```

Важные файлы:

```text
prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/manifest.jsonl
prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/progress_live.json
prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/generation_log.txt
prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/sequences_summary.json
prepared_i2nav_robot_corridor_dataset_v1_2_sec/meta/dataset_config.json
```

Техническое описание:

```text
i2nav_robot_corridor_dataset_technical_overview.md
```

---

# 5. Cross-domain anchor trajectories

## 5.1. v1 anchors

Корень:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v1
```

Главные файлы:

```text
trajectory_vectors_combined_train.jsonl
trajectory_vectors_with_anchor_K20.jsonl
anchors_K20_M8_norm.npy
anchors_K20_M8_norm.json
anchor_k_selection_metrics.json
anchor_generation_summary.json
anchor_K20_domain_distribution.json
anchors_K20_preview.jpg
trajectory_extraction_preview.jpg
```

Смысл:

```text
Взяли 10k Waymo + 5254 i2Nav-Robot train samples.
Извлекли normalized image-space trajectory vectors M=8.
Построили KMeans anchors для K = 8, 12, 16, 20, 24, 32.
Выбрали K=20 как баланс compactness и интерпретируемости.
```

Проблема v1:

```text
anchors 06, 10, 13 были слишком большими / extreme.
Они могли появиться из trajectory outliers.
```

## 5.2. v2 filtered anchors

Корень:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/cross_domain_anchor_trajectories_cdad_planner_v2_filtered
```

Главные файлы:

```text
trajectory_vectors_filtered_no_extreme.jsonl
trajectory_vectors_removed_extreme.jsonl
trajectory_vectors_with_anchor_K20_filtered.jsonl
anchors_K20_M8_norm.npy
anchors_K20_M8_norm.json
anchors_K20_filtered_preview.jpg
filtering_and_anchor_generation_stats.json
README_Cross_Domain_Anchors_v2_filtered.md
```

Смысл:

```text
Не удаляли anchors напрямую.
Сначала удалили extreme trajectory samples.
Потом заново построили K=20 anchors.
```

Фильтрационные признаки:

```text
total_len
max_step
u_span
v_span
min_v
max_v
bad_v_steps
roughness
```

Итог:

```text
v2 filtered anchors визуально заметно лучше.
Их используем во всех основных экспериментах.
```

---

# 6. Основные notebooks / код экспериментов

## 6.1. Waymo dataset creation

```text
build_corridor_dataset_filtered_front_logged-3.ipynb
```

Создает автомобильный corridor dataset: front camera image + future trajectory + corridor/centerline/heatmap.

## 6.2. Waymo end-to-end diffusion planner draft

```text
model_training-3_stronger.ipynb
```

Первый end-to-end diffusion planner по одному фото. Для текущей CDAD статьи не является главным.

## 6.3. Waymo U-Net segmentation

```text
train_corridor_resnet50_augmented_no_intent_v2_pbar-2.ipynb
```

U-Net segmentation baseline для Waymo corridor dataset.

## 6.4. i2Nav robot dataset creation

```text
build_i2nav_robot_corridor_dataset_final_N20_logged
```

Создает роботный image-space future trajectory dataset.

## 6.5. Mixed v1

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v1/
```

Run:

```text
2026-05-27_08-08-56_mixed_K20_M8_anchor_truncated_diffusion
```

Смысл:

```text
Первая mixed CDAD модель.
Использовала v1 anchors.
Показала, что модель в принципе учится, но были визуальные проблемы и extreme anchors.
```

Ключевые test metrics:

```text
test ADE = 0.05372
test FDE = 0.07059
Waymo ADE = 0.05957
i2Nav ADE = 0.04253
```

## 6.6. Mixed v2 filtered geometry

Ноутбук:

```text
train_cdad_planner_lite_v2_filtered_geometry_logged.ipynb
```

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v2_filtered_geometry/
```

Run:

```text
2026-05-30_17-29-18_mixed_K20_M8_filtered_geometry_truncated_diffusion
```

Изменения:

```text
filtered anchors
CoordConv
endpoint scale head
stronger trajectory/shape/end losses
top-k preview
```

Результат:

```text
test ADE = 0.13930
test FDE = 0.16062
```

Вывод:

```text
v2 визуально стабилизировал anchors, но top-1 regression стал хуже.
Слишком много изменений сразу.
От этой ветки отказались как от основной.
```

## 6.7. Mixed v3: главная рабочая модель

Ноутбук:

```text
train_cdad_planner_lite_v3_filtered_anchors_v1_loss_logged.ipynb
```

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_filtered_anchors_v1_loss
```

Run:

```text
2026-06-01_11-07-38_mixed_K20_M8_filtered_anchors_v1_loss_truncated_diffusion
```

Смысл:

```text
Откатились к v1-style stable model/loss.
Оставили только v2 filtered anchors.
```

Это текущая основная модель.

Ключевые результаты:

```text
Test Mixed ADE = 0.04719
Test Mixed FDE = 0.05829

Waymo ADE = 0.05117
Waymo FDE = 0.06559

i2Nav ADE = 0.03985
i2Nav FDE = 0.04481
```

Папка с полным экспортом результатов:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_filtered_anchors_v1_loss/2026-06-01_11-07-38_mixed_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 6.8. i2Nav-only experiment

Ноутбук:

```text
train_cdad_planner_lite_v3_TRAIN_i2Nav_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_i2nav_only_filtered_anchors_v1_loss
```

Run:

```text
2026-06-02_11-32-04_i2nav_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion
```

Split:

```text
TRAIN: i2Nav-Robot only = 4202
VAL:   i2Nav-Robot only = 526
TEST:  same mixed test split = 1496
       Waymo = 970
       i2Nav-Robot = 526
```

Results:

```text
Test Mixed ADE = 0.06901
Test Mixed FDE = 0.09578

Waymo ADE = 0.08389
Waymo FDE = 0.12159

i2Nav ADE = 0.04157
i2Nav FDE = 0.04818
```

Папка результатов:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_i2nav_only_filtered_anchors_v1_loss/2026-06-02_11-32-04_i2nav_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 6.9. Waymo-only experiment

Ноутбук:

```text
train_cdad_planner_lite_v3_TRAIN_Waymo_ONLY_TEST_same_mixed_split_filtered_anchors_v1_loss_logged.ipynb
```

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_waymo_only_filtered_anchors_v1_loss
```

Run:

```text
2026-06-02_22-38-44_waymo_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion
```

Split:

```text
TRAIN: Waymo only = 7760
VAL:   Waymo only = 968
TEST:  same mixed test split = 1496
       Waymo = 970
       i2Nav-Robot = 526
```

Results:

```text
Test Mixed ADE = 0.05238
Test Mixed FDE = 0.06931

Waymo ADE = 0.05093
Waymo FDE = 0.06607

i2Nav ADE = 0.05507
i2Nav FDE = 0.07528
```

Папка результатов:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_waymo_only_filtered_anchors_v1_loss/2026-06-02_22-38-44_waymo_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 6.10. Anchor-only vs refined evaluation

Ноутбук:

```text
evaluate_cdad_v3_anchor_only_vs_refined_mixed_same_test.ipynb
```

Артефакты:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_filtered_anchors_v1_loss/2026-06-01_11-07-38_mixed_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_ablation_anchor_only_vs_refined
```

Файлы:

```text
anchor_only_vs_refined_metrics.json
tables_csv/anchor_only_vs_refined_compact.csv
tables_csv/anchor_only_vs_refined_metrics_long.csv
previews/anchor_only_vs_refined_preview.jpg
```

Anchor-only result:

```text
Predicted anchor only:
Mixed ADE = 0.14048
Mixed FDE = 0.16984
Waymo ADE = 0.14914
Waymo FDE = 0.18790
i2Nav ADE = 0.12451
i2Nav FDE = 0.13654
```

Important note:

```text
Этот notebook также считал refined branch, но был замечен mismatch с official final_test_results.
Для статьи использовать anchor-only из этого notebook,
а diffusion результат брать из official Mixed v3 final_test_results.
```

## 6.11. Deterministic residual baseline

Ноутбук:

```text
train_cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss_logged_v2.ipynb
```

Эксперимент:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss
```

Run:

```text
2026-06-03_13-15-30_deterministic_residual_baseline_K20_M8_same_mixed_split_filtered_anchors_v1_loss
```

Смысл:

```text
Та же модельная база, те же anchors, те же splits,
но без diffusion noise и timestep denoising.
Обычный deterministic residual:
anchor + learned_delta = trajectory.
```

Results:

```text
Test Mixed ADE = 0.12302
Test Mixed FDE = 0.15110

Waymo ADE = 0.14502
Waymo FDE = 0.18252

i2Nav ADE = 0.08244
i2Nav FDE = 0.09315
```

Папка результатов:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss/2026-06-03_13-15-30_deterministic_residual_baseline_K20_M8_same_mixed_split_filtered_anchors_v1_loss/paper_artifacts_full
```

---

# 7. Где искать результаты

В каждом experiment package структура одинаковая:

```text
paper_artifacts_full/
  README_EXPERIMENT.md
  metadata/
  raw_json/
  tables_csv/
  plot_data/
  plots_png/
  previews/
  copied_core_files/
  reports/
```

Главные файлы:

```text
tables_csv/final_test_results_flat.csv
tables_csv/domain_wise_metrics.csv
tables_csv/epoch_metrics_flat.csv
plot_data/
plots_png/
previews/
raw_json/final_test_results.json
reports/main_result_summary.json
```

## 7.1. Mixed v3 result package

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_filtered_anchors_v1_loss/2026-06-01_11-07-38_mixed_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 7.2. i2Nav-only result package

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_i2nav_only_filtered_anchors_v1_loss/2026-06-02_11-32-04_i2nav_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 7.3. Waymo-only result package

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_waymo_only_filtered_anchors_v1_loss/2026-06-02_22-38-44_waymo_only_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_full
```

## 7.4. Deterministic residual baseline package

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_DETERMINISTIC_RESIDUAL_BASELINE_same_mixed_split_filtered_anchors_v1_loss/2026-06-03_13-15-30_deterministic_residual_baseline_K20_M8_same_mixed_split_filtered_anchors_v1_loss/paper_artifacts_full
```

## 7.5. Anchor-only ablation package

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/experiments/cdad_planner_lite_v3_filtered_anchors_v1_loss/2026-06-01_11-07-38_mixed_K20_M8_filtered_anchors_v1_loss_truncated_diffusion/paper_artifacts_ablation_anchor_only_vs_refined
```

---

# 8. Главные таблицы для статьи

## 8.1. Cross-domain transfer table

```text
Train data     Test Mixed ADE/FDE      Test Waymo ADE/FDE      Test i2Nav ADE/FDE
---------------------------------------------------------------------------------
Mixed          0.04719 / 0.05829       0.05117 / 0.06559       0.03985 / 0.04481
i2Nav-only     0.06901 / 0.09578       0.08389 / 0.12159       0.04157 / 0.04818
Waymo-only     0.05238 / 0.06931       0.05093 / 0.06607       0.05507 / 0.07528
```

## 8.2. Refinement ablation table

```text
Method                              Params     Test Mixed ADE/FDE      Test Waymo ADE/FDE      Test i2Nav ADE/FDE
------------------------------------------------------------------------------------------------------------------
Predicted anchor only               —          0.14048 / 0.16984       0.14914 / 0.18790       0.12451 / 0.13654
Anchor + deterministic residual     835397     0.12302 / 0.15110       0.14502 / 0.18252       0.08244 / 0.09315
Anchor + truncated diffusion        835397     0.04719 / 0.05829       0.05117 / 0.06559       0.03985 / 0.04481
```

## 8.3. Parameter count table

```text
Model                         Total params     Trainable params
---------------------------------------------------------------
Deterministic residual         835,397          835,397
Truncated diffusion            835,397          835,397
```

Important phrase:

```text
The deterministic residual and truncated diffusion variants use exactly the same number of trainable parameters.
Therefore, the performance gap cannot be attributed to a larger model.
```

---

# 9. Основные выводы для статьи

## 9.1. Cross-domain conclusion

```text
A shared image-space anchor vocabulary can support both driving and ground-robot navigation domains.
However, single-domain training is insufficient for robust transfer.
Mixed-domain training gives the most stable performance across both domains.
```

## 9.2. Anchor vocabulary conclusion

```text
Filtered cross-domain anchors provide a compact shared basis of vehicle-like and robot-like future-path primitives.
Filtering extreme trajectory samples before KMeans improves anchor interpretability and stabilizes downstream training.
```

## 9.3. Diffusion conclusion

```text
Truncated diffusion refinement substantially outperforms deterministic residual refinement under an identical parameter budget.
This supports the use of diffusion-style refinement around ranked trajectory anchors.
```

## 9.4. Limitations

```text
1. Logged trajectories are weak supervision, not optimal plans.
2. The method predicts image-space future-path priors, not rule-validated control commands.
3. No HD map, traffic rules, collision checking, or control-level validation.
4. The test currently covers Waymo and i2Nav-Robot only.
5. The model still depends on anchor-mode recognition.
```

---

# 10. Рекомендуемая структура статьи

## Title draft

```text
Cross-Domain Anchor Diffusion for Image-Space Future-Path Prediction in Vehicles and Ground Robots
```

## Paper sections

```text
1. Introduction
2. Related Work
   - visual trajectory prediction
   - anchor-based planning
   - diffusion planning
   - cross-domain embodied navigation
3. Dataset and Image-Space Future-Path Representation
   - Waymo corridor dataset
   - i2Nav-Robot corridor dataset
   - normalized trajectory representation
4. Cross-Domain Anchor Vocabulary
   - trajectory extraction
   - filtering extreme trajectories
   - K=20 anchor construction
5. CDAD-Planner-Lite
   - shared encoder
   - anchor classifier
   - path-prior head
   - truncated diffusion refinement
6. Experiments
   - mixed / i2Nav-only / Waymo-only
   - anchor-only / deterministic residual / diffusion
   - parameter count comparison
7. Results
8. Limitations
9. Conclusion
```

---

# 11. Что делать дальше

## Immediate next steps

```text
1. Собрать общий comparison package:
   - transfer table
   - refinement ablation table
   - parameter count table
   - all CSV/JSON source points

2. Сделать qualitative figures:
   - filtered anchors K=20
   - Waymo predictions
   - i2Nav predictions
   - transfer failure examples
   - anchor-only vs deterministic vs diffusion examples

3. Проверить clean anchor-only evaluation pipeline:
   - anchor-only should be computed as top-1 predicted anchor on same mixed test split.
   - avoid mismatch with official final_test_results.

4. Добавить direct regression baseline:
   image → trajectory directly, no anchors, no diffusion.
   Это закроет вопрос: зачем anchors вообще?

5. Подготовить GitHub:
   - configs
   - notebooks
   - scripts
   - anchors json
   - result CSVs
   - README
```

## Strong next experiment

```text
Direct regression baseline:
same encoder
same mixed train/val/test split
input: RGB
output: 8x2 trajectory directly
no anchors
no diffusion
```

Expected use:

```text
If direct regression is worse than CDAD, this proves that anchors are useful.
```

## GitHub package

Suggested structure:

```text
github/
  README.md
  configs/
  notebooks/
  scripts/
  anchors/
    anchors_K20_M8_norm.json
  results/
    transfer_table.csv
    refinement_ablation_table.csv
    parameter_counts.csv
  figures/
  docs/
```

---

# 12. Краткий status проекта

Сейчас проект имеет сильную основу для статьи:

```text
1. Есть рабочая модель Mixed v3.
2. Есть transfer matrix.
3. Есть ablation against deterministic residual.
4. Есть parameter-matched proof.
5. Есть сохраненные experiment artifacts.
```

Оценка публикационного потенциала:

```text
Хороший Q2: реалистично.
Слабый / borderline Q1: возможно при аккуратной упаковке, clean ablations и качественной визуализации.
Сильный Q1: пока нужен внешний baseline и более широкая экспериментальная проверка.
```

Главная формула статьи:

```text
Shared cross-domain image-space anchors + parameter-matched truncated diffusion refinement
provide a compact and transferable visual future-path prior for both vehicle and ground-robot domains.
```
