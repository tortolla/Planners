# Лабораторный журнал: Diffusion-Lite / FlowPlanner-Lite для image-space future-path prediction

Дата фиксации идеи: 2026-05-25  
Проект: unified visual future-path prediction для Waymo + i2Nav-Robot  
Рабочее направление: компактная модель, которая по одному RGB-кадру предсказывает будущую траекторию/коридор в координатах изображения.

---

## 1. Короткая формулировка идеи

Мы хотим обучить компактную модель, которая по одному RGB-изображению строит визуальный prior будущего движения агента:

```text
Input:  single/front RGB image
Output: image-space future-path representation
        - corridor mask
        - centerline mask
        - Gaussian heatmap
        - optional vectorized path / sampled waypoints
```

Это не full planner и не navigation foundation model. Это быстрый, понятный и интерпретируемый первый слой, который рисует вероятное направление будущего движения прямо на изображении.

Рабочие названия:

```text
Diffusion-Lite Future Path Predictor
FlowPlanner-Lite
Image-Space Future Path Prior
Cross-Domain Visual Path Prior
```

Главная идея: вместо тяжелой модели уровня VLM/VLA foundation model мы строим компактный модуль, который можно быстро обучить и использовать как промежуточный слой для последующего planning pipeline.

---

## 2. Почему это интересно

Современные trajectory planning и navigation модели обычно сильно завязаны на домен:

```text
Autonomous driving:
- road datasets
- car-centric geometry
- long-range trajectories
- structured lanes / roads

Robot navigation:
- indoor/outdoor robot logs
- short-horizon local motion
- narrow passages, walls, obstacles
- different embodiment and action scale
```

Большие работы уже показывают, что cross-embodiment navigation возможна. Например, NavFoM формулируется как cross-embodiment and cross-task navigation foundation model, обученный на данных от quadrupeds, drones, wheeled robots и vehicles, а также на задачах VLN, object search, target tracking и autonomous driving. Это подтверждает актуальность объединения разных embodied navigation domains.

Но NavFoM — это тяжелая foundation-model постановка:

```text
- VLM/VLA architecture
- multi-view / video input
- language instructions
- large LLM backbone
- millions of navigation samples
- cross-task training
- trajectory/action output
```

Наша постановка другая: мы хотим проверить, можно ли сделать маленький и быстрый слой, который учит общий image-space future-path prior по логам машин и роботов.

---

## 3. Отличие от NavFoM и других больших navigation foundation models

NavFoM решает широкую задачу:

```text
instruction + temporal multi-camera observations -> trajectory waypoints
```

Он работает как generalist navigation foundation model. В статье NavFoM вводятся temporal-viewpoint indicator tokens для кодирования camera viewpoint и времени, а также budget-aware temporal sampling для управления числом visual tokens.

Наша постановка намеренно проще:

```text
single RGB image -> future-path heatmap / mask
```

Ключевые отличия:

| Аспект | NavFoM-style foundation model | Наша постановка |
|---|---|---|
| Input | video, multi-camera, language | single RGB image |
| Output | trajectory waypoints / action | image-space mask / heatmap / centerline |
| Model scale | large VLM/VLA | compact Diffusion-Lite / FlowPlanner-Lite |
| Data scale | millions of samples | small-data regime: 10k Waymo + 5–6k i2Nav |
| Goal | generalist navigation | visual future-path prior |
| Deployment role | full navigation policy/planner | first-stage interpretable planning prior |
| Main contribution | foundation model across tasks and embodiments | unified image-space supervision across car + robot logs |

Это важная защита: мы не пытаемся конкурировать с navigation foundation models. Мы исследуем более дешевую и интерпретируемую форму cross-domain future-path learning.

---

## 4. Связь с Diffusion Planner, NoMaD и GNM

### 4.1 Diffusion Planner

Diffusion Planner показывает, что diffusion models могут быть эффективными для autonomous driving planning: они моделируют multimodal driving behavior и генерируют будущие trajectories для closed-loop planning.

Но это driving-specific постановка. Она ориентирована на autonomous driving, где структура дороги и поведение участников сильно отличаются от локальной роботной навигации.

Наша идея:

```text
не full driving planner,
а compact image-conditioned future-path generator,
который можно обучать на Waymo + i2Nav.
```

### 4.2 NoMaD

NoMaD показывает, что diffusion policy применима к роботной visual navigation и может объединять goal-directed navigation и exploration. Это важно для нас как подтверждение, что diffusion-like генерация полезна в robotics navigation.

Но NoMaD — это navigation policy. Мы же хотим сделать более легкий слой:

```text
image -> path heatmap
```

Этот слой можно потом использовать внутри policy/planner, но он сам по себе не обязан принимать goal image или выдавать executable control.

### 4.3 GNM / General Navigation Models

GNM формулирует general-purpose goal-conditioned visual navigation policies, обученные на cross-embodiment data. Это близко по идее переноса между роботами.

Наше отличие: мы не стандартизируем физический action space. Мы стандартизируем visual supervision space:

```text
машина: future path -> projection into image
робот: future path -> projection into image
оба: H x W heatmap/mask/centerline
```

То есть мы используем image-space target как общий слой между доменами.

---

## 5. Центральная гипотеза

Рабочая гипотеза:

```text
A shared image-space future-path representation can provide a simple and lightweight bridge between road-vehicle and ground-robot navigation logs.
```

По-русски:

```text
Единое представление будущей траектории в координатах изображения может служить простым общим форматом для обучения на автомобильных и роботных логах.
```

Мы не утверждаем, что logged future path всегда является оптимальной или безопасной траекторией. Это weak imitation supervision: модель учится visual prior движения из логов.

---

## 6. Что именно будет делать модель

Минимальный вариант:

```text
RGB image -> heatmap будущей траектории
```

Дополнительные выходы:

```text
RGB image -> corridor_mask
RGB image -> centerline_mask
RGB image -> Gaussian heatmap
RGB image -> optional waypoint samples
```

Практическая интерпретация:

```text
Raw RGB image
  -> visual future-path prior
  -> downstream module:
       - sample candidate trajectories
       - condition local planner
       - score planned paths
       - initialize diffusion trajectory generation
       - combine with depth/LiDAR/safety constraints
```

Модель отвечает не на вопрос:

```text
Какую управляющую команду выполнить прямо сейчас?
```

А на более мягкий вопрос:

```text
Где в изображении находится вероятное направление будущего движения?
```

Это делает постановку более устойчивой и менее претенциозной.

---

## 7. Почему image-space target удобен

Физические trajectories у машин и роботов разные:

```text
Cars:
- большие скорости
- длинный горизонт
- движение по дорожной сети
- метры/десятки метров вперед

Ground robots:
- короткий горизонт
- indoor/outdoor passages
- стены, машины, двери, люди
- локальная навигация на 1–3 метра
```

Если объединять физические waypoints напрямую, придется решать разные action scales и dynamics. Image-space representation позволяет свести оба домена к одному output format:

```text
H x W binary mask
H x W centerline
H x W heatmap
```

Это проще для первой модели и удобно для визуальной проверки.

---

## 8. Датасеты и текущий план

### 8.1 Waymo

Используется автомобильный датасет, где уже есть примерно 100k примеров corridor/path supervision. Для нового эксперимента план:

```text
sample 10k examples from Waymo corridor dataset
```

Waymo дает:

```text
- road domain
- стабильную дорожную структуру
- большой масштаб
- относительно чистые logged future paths
```

### 8.2 i2Nav-Robot

Используется i2Nav-Robot как ground robot visual domain. Текущий pipeline строит projected logged future trajectory:

```text
left RGB camera frame
+ trajectory.csv
+ calibration.yaml
-> projected future path labels
```

Текущая генерация:

```text
EVERY_N = 10
HORIZON_SEC = 2.0
STEP_SEC = 0.25
expected output: around 5k–6k robot examples
```

Почему horizon 2 sec:

```text
- меньше случайных уходов к стенам/машинам;
- меньше будущих точек за поворотом;
- лучше локальная image-space интерпретация;
- ближе к роботной local navigation.
```

### 8.3 Combined dataset

План:

```text
Waymo: 10k examples
+ i2Nav-Robot: 5k–6k examples
= around 15k–16k mixed-domain training set
```

В manifest обязательно хранить:

```json
{
  "source_dataset": "Waymo" или "i2Nav-Robot",
  "dataset_root": "...",
  "image_path": "...",
  "corridor_mask_path": "...",
  "centerline_mask_path": "...",
  "gaussian_heatmap_path": "...",
  "split": "train/val/test",
  "sequence": "..."
}
```

---

## 9. Важное ограничение: logged future trajectory is weak supervision

Критичный методологический пункт:

```text
Logged future path != optimal safe path.
```

Особенно для i2Nav-Robot:

```text
робот мог подъехать к стене;
робот мог подъехать к машине;
робот мог разворачиваться;
оператор мог выполнять случайный маневр;
будущая точка может быть видима плохо или быть за препятствием.
```

Поэтому правильная интерпретация label:

```text
logged future path projection
weak imitation signal
visual prior of observed behavior
```

Неправильная интерпретация:

```text
safe trajectory
optimal navigation command
true traversability
```

В статье это надо писать явно.

---

## 10. Возможная модель: Diffusion-Lite

### 10.1 Почему не full diffusion

Полная diffusion model может быть тяжелой:

```text
- много denoising steps;
- высокая стоимость обучения;
- медленный inference;
- большие данные нужны для стабильного обучения;
- тяжелая архитектура может быть избыточна для heatmap target.
```

Поэтому идея: сделать diffusion-lite.

### 10.2 Возможные варианты Diffusion-Lite

#### Вариант A: deterministic backbone + stochastic refinement

```text
RGB image
 -> encoder-decoder predicts coarse heatmap
 -> few-step denoising/refinement improves path distribution
```

Плюсы:

```text
- быстро;
- можно обучать на малом датасете;
- output визуально интерпретируем;
- diffusion используется только как refinement.
```

#### Вариант B: latent diffusion over low-resolution heatmap

```text
RGB image -> latent condition
noise heatmap 64x64 / 128x128 -> denoise -> upsample path heatmap
```

Плюсы:

```text
- меньше вычислений;
- можно генерировать multimodal candidate heatmaps;
- проще, чем full-resolution diffusion.
```

#### Вариант C: FlowPlanner-Lite

Идея FlowPlanner-Lite:

```text
predict a continuous flow field / vector field that points toward future path
```

Возможные outputs:

```text
- heatmap probability
- direction field dx, dy
- centerline confidence
```

Такой вариант может быть легче диффузии и ближе к dense prediction.

#### Вариант D: one-step consistency / rectified-flow style

```text
RGB + noise -> one/few-step path heatmap
```

Цель: получить генеративность diffusion-like модели, но без десятков sampling steps.

---

## 11. Почему Diffusion-Lite / FlowPlanner-Lite имеет смысл

Будущее движение часто неоднозначно:

```text
машина может продолжить прямо или повернуть;
робот может обойти препятствие слева или справа;
видимая сцена может иметь несколько проходимых коридоров;
logged path — только один из возможных вариантов.
```

Deterministic segmentation model даст средний/один путь. Diffusion-like модель может дать несколько plausible path hypotheses.

Но мы хотим сохранить модель маленькой, поэтому:

```text
не full diffusion planner,
а lightweight image-space generator.
```

---

## 12. Возможные формулировки вклада статьи

### Contribution 1: Unified image-space future-path representation

Мы приводим driving и robot logs к одному target format:

```text
RGB -> future-path mask/heatmap/centerline
```

Это позволяет обучать одну модель на Waymo и i2Nav-Robot без прямого согласования physical action spaces.

### Contribution 2: Compact Diffusion-Lite / FlowPlanner-Lite baseline

Мы предлагаем легкую модель для предсказания future-path prior по одному изображению. Она меньше и быстрее full diffusion/VLM planners.

### Contribution 3: Cross-domain small-data study

Экспериментальная схема:

```text
Waymo-only training
Robot-only training
Mixed Waymo+i2Nav training
```

Проверяем:

```text
- in-domain performance;
- cross-domain transfer;
- насколько robot domain помогает/мешает;
- насколько Waymo pretraining помогает роботам.
```

### Contribution 4: Failure analysis of logged-path supervision

Мы явно анализируем, где logged future trajectory как label плох:

```text
- trajectories toward obstacles;
- turns behind occluders;
- operator-specific behavior;
- parked/turnaround maneuvers;
- mismatch between future path and visual traversability.
```

Это важно для честной статьи.

---

## 13. Потенциальная структура статьи

### Title ideas

```text
From Roads to Robots: Lightweight Image-Space Future Path Prediction Across Driving and Ground Navigation Logs
```

```text
FlowPlanner-Lite: Compact Image-Space Future Path Prediction for Vehicles and Ground Robots
```

```text
Diffusion-Lite Future Path Priors for Cross-Domain Visual Navigation
```

### Abstract skeleton

```text
Large navigation foundation models have demonstrated the potential of cross-task and cross-embodiment navigation, but their scale and complexity make them expensive to train and deploy. We study a simpler question: can road-vehicle and ground-robot logs be unified through an image-space future-path representation, enabling compact models to learn a shared visual path prior from limited data? We convert Waymo and i2Nav-Robot trajectories into corridor masks, centerline masks, and Gaussian heatmaps projected into the camera view. We then train a lightweight Diffusion-Lite / FlowPlanner-Lite model from single RGB images to predict these representations. Experiments compare Waymo-only, robot-only, and mixed-domain training, analyzing transfer, data efficiency, and limitations of logged future trajectory supervision.
```

### Introduction argument

```text
1. Navigation models are usually domain-specific.
2. Foundation models show cross-embodiment navigation is possible but are heavy.
3. A simpler common supervision space may enable cheaper cross-domain learning.
4. Image-space future-path heatmaps are interpretable, compact, and easy to combine with downstream planners.
5. We study this in a small-data setting using Waymo + i2Nav-Robot.
```

---

## 14. Экспериментальный план

### Datasets

```text
D_waymo_10k:
  10k sampled Waymo examples

D_i2nav_N10_2sec:
  approximately 5k–6k i2Nav examples

D_mixed:
  D_waymo_10k + D_i2nav_N10_2sec
```

### Models

Baseline 1:

```text
U-Net / ResNet-U-Net
RGB -> heatmap/mask
```

Baseline 2:

```text
SegFormer-lite / MobileNet encoder-decoder
RGB -> heatmap/mask
```

Proposed:

```text
Diffusion-Lite / FlowPlanner-Lite
RGB -> stochastic or flow-based future-path heatmap
```

### Metrics

```text
Mask metrics:
- IoU
- Dice
- pixel precision/recall

Centerline/path metrics:
- Chamfer distance
- endpoint distance in image space
- average path distance
- heatmap MSE / BCE / focal loss

Cross-domain:
- train Waymo -> test i2Nav
- train i2Nav -> test Waymo
- train mixed -> test both
```

### Qualitative evaluation

```text
- RGB input
- ground-truth projected path
- predicted heatmap
- predicted centerline
- overlay visualization
- failure cases
```

---

## 15. Честная формулировка limitations

Обязательно указать:

```text
1. Labels are logged future trajectories, not optimal navigation targets.
2. The image-space projection is not visibility-aware unless occlusion filtering is added.
3. i2Nav logged behavior may include obstacle-approach or turnaround maneuvers.
4. Single-frame input cannot fully disambiguate intent or dynamic context.
5. This module is a visual prior, not a standalone safe planner.
```

Это не слабость, если правильно подать. Это делает работу честной и понятной.

---

## 16. Как это можно использовать дальше

Predicted future-path heatmap можно использовать как:

```text
- input channel для downstream planner;
- cost map prior;
- initializer для trajectory optimization;
- condition для diffusion trajectory sampler;
- auxiliary task при обучении visual navigation policy;
- interpretable diagnostic output;
- pseudo-label для более сложной модели.
```

Идея pipeline:

```text
RGB image
  -> Diffusion-Lite / FlowPlanner-Lite path prior
  -> geometry/safety layer using depth/LiDAR/occupancy
  -> local planner / trajectory sampler
  -> executable command
```

---

## 17. Текущий статус проекта

Сделано:

```text
1. Waymo corridor dataset pipeline существует.
2. i2Nav-Robot calibration/projection pipeline собран.
3. i2Nav N20 / horizon 2 sec версия получена как контрольная.
4. i2Nav N10 / horizon 2 sec версия считается для 5k–6k примеров.
5. Manifest format содержит source_dataset / sequence / split / paths.
6. Есть понимание, что i2Nav labels — weak logged-path supervision.
```

Следующее:

```text
1. Досчитать i2Nav N10 2sec.
2. Собрать combined dataset: Waymo 10k + i2Nav 5k–6k.
3. Проверить visual samples и статистику.
4. Обучить простой deterministic baseline.
5. Затем сделать Diffusion-Lite / FlowPlanner-Lite.
6. Сравнить Waymo-only, i2Nav-only, mixed.
```

---

## 18. Ссылки / ориентиры

- NavFoM / Embodied Navigation Foundation Model: cross-task and cross-embodiment navigation foundation model trained on quadrupeds, drones, wheeled robots and vehicles.
- Diffusion Planner: diffusion-based closed-loop autonomous driving planner for multimodal trajectory generation.
- NoMaD: goal-masked diffusion policies for robot navigation and exploration.
- GNM: general navigation model trained on cross-embodiment visual navigation data.

Эти работы подтверждают, что направление актуально. Наша ниша — не full foundation model, а компактный image-space future-path prior для смешанного car+robot visual domain.

---

## 19. Одна фраза для запоминания

```text
Мы не строим полноценный navigation foundation model. Мы строим маленький, быстрый и интерпретируемый visual future-path prior, который по одному RGB-кадру рисует вероятную будущую траекторию в координатах изображения и может служить первым слоем для более сложного планировщика.
```
