# Wayomo-end-to-end / Waymo image-space corridor dataset

Документ описывает датасет, который генерируется ноутбуком:

`build_corridor_dataset_filtered_front_logged.ipynb`

В приложенном файле ноутбук называется:

`build_corridor_dataset_filtered_front_logged-3.ipynb`

Назначение ноутбука: построить датасет для задачи **«одно фронтальное изображение с камеры автомобиля → будущая траектория/коридор движения на этом же изображении»**.

Важно: в этом датасете модель не получает LiDAR, карту, скорость, команду руля, навигационный маршрут или 3D-сцену. Целевой сигнал строится из GT future trajectory и калибровки камеры Waymo, а затем переводится в координаты пикселей текущего изображения.

---

## 1. Короткое определение задачи

### Вход модели

Один RGB-кадр с фронтальной камеры автомобиля Waymo:

```text
image.png
```

Камера используется только одна:

```text
FRONT
```

То есть это постановка:

```text
RGB image from FRONT camera -> image-space future corridor
```

### Выход модели

Основной целевой выход:

```text
corridor_mask.png
```

Это бинарная маска на изображении, где белые пиксели показывают область будущего движения автомобиля, спроецированную на плоскость изображения.

Дополнительно сохраняются два вспомогательных target-формата:

```text
centerline_mask.png
```

тонкая линия центра будущей траектории;

```text
gaussian_heatmap.npy
```

мягкая тепловая карта, полученная из corridor mask через Gaussian blur.

Также сохраняется:

```text
projected_uv_gt.npy
```

массив 2D-точек будущей траектории после проекции в изображение.

---

## 2. Где лежит исходный код генерации

Основной ноутбук:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/build_corridor_dataset_filtered_front_logged.ipynb
```

Приложенная версия:

```text
/mnt/data/build_corridor_dataset_filtered_front_logged-3.ipynb
```

Внутренний заголовок ноутбука:

```text
Waymo → corridor dataset builder
```

Цель ноутбука в исходном markdown-блоке:

```text
(camera image + high-level condition) → future corridor mask / heatmap
```

Фактически в текущей версии используется FRONT RGB image и GT future trajectory. High-level condition в текущем экспорте отдельно не записывается как входной канал модели.

---

## 3. Исходные данные

Ноутбук работает поверх уже подготовленной структуры `precomputed_waymo_e2e` и оригинальных TFRecord-файлов Waymo Open Dataset End-to-End.

### 3.1. Корневая папка precomputed samples

```python
PRECOMP_ROOT = Path("precomputed_waymo_e2e")
TRAIN_ROOT = PRECOMP_ROOT / "train"
VAL_ROOT = PRECOMP_ROOT / "val"
```

Абсолютно это соответствует рабочей директории проекта:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/precomputed_waymo_e2e
```

Структура:

```text
precomputed_waymo_e2e/
  train/
    samples/
      <sample_id>/
        meta.json
        gt_future_xy.npy или другой файл GT future trajectory
  val/
    val_samples.jsonl
    samples/...
```

### 3.2. Оригинальный Waymo E2E root

```python
RAW_WAYMO_ROOT = Path(
    "/home/Jupyter/datasets/tesla/Waymo_open_dataset/waymo_open_dataset_end_to_end_camera_v_1_0_0"
)
```

Эта папка нужна, потому что RGB-изображение и калибровка камеры берутся из оригинального Waymo E2E TFRecord, а не из `precomputed_waymo_e2e`.

### 3.3. Waymo source path

```python
WAYMO_SRC = Path("/home/Jupyter/datasets/tesla/Waymo_open_dataset/waymo-od/src")
```

Этот путь добавляется в `sys.path`, после чего импортируются:

```python
from waymo_open_dataset import dataset_pb2
from waymo_open_dataset.protos import end_to_end_driving_data_pb2
```

Также используется TensorFlow:

```python
import tensorflow as tf
```

TensorFlow нужен для чтения TFRecord:

```python
tf.data.TFRecordDataset(str(tfrecord_path), compression_type="")
```

---

## 4. Выходной датасет

Ноутбук пишет результат в новую папку:

```python
OUT_ROOT = Path("prepared_camera_corridor_dataset_filtered_front_v1")
```

Абсолютный путь:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1
```

Логи пишутся сюда:

```python
LOG_ROOT = OUT_ROOT / "_logs"
```

Абсолютный путь:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/_logs
```

---

## 5. Итоговая структура датасета

После экспорта ожидается такая структура:

```text
prepared_camera_corridor_dataset_filtered_front_v1/
  train/
    FRONT/
      manifest.jsonl
      <sample_id>/
        image.png
        corridor_mask.png
        centerline_mask.png
        gaussian_heatmap.npy
        projected_uv_gt.npy
        meta.json
  val/
    FRONT/
      manifest.jsonl
      <sample_id>/
        image.png
        corridor_mask.png
        centerline_mask.png
        gaussian_heatmap.npy
        projected_uv_gt.npy
        meta.json
  _logs/
    train_FRONT.log
    train_FRONT_progress.json
    train_FRONT_skipped.log
    train_FRONT_errors.jsonl
    val_FRONT.log
    val_FRONT_progress.json
    val_FRONT_skipped.log
    val_FRONT_errors.jsonl
```

Главные файлы для обучения:

```text
train/FRONT/manifest.jsonl
val/FRONT/manifest.jsonl
```

---

## 6. Что лежит в одном sample

Для каждого валидного sample создается директория:

```text
prepared_camera_corridor_dataset_filtered_front_v1/<split>/FRONT/<sample_id>/
```

Внутри:

### `image.png`

RGB-изображение с FRONT камеры Waymo.

Файл сохраняется как PNG:

```python
save_png(rgb.astype(np.uint8), image_path)
```

Это основной вход модели.

### `corridor_mask.png`

Бинарная маска будущего коридора движения.

Формат:

```text
uint8 PNG, значения 0 или 255
```

Белая область — будущий коридор движения автомобиля в координатах текущего изображения.

Это основной target для обучения.

### `centerline_mask.png`

Тонкая маска центральной линии будущей траектории.

Формат:

```text
uint8 PNG, значения 0 или 255
```

Она жестче, чем corridor mask, и хуже переносит ошибки проекции, но полезна для отладки и альтернативного обучения.

### `gaussian_heatmap.npy`

Мягкая тепловая карта, построенная из corridor mask через Gaussian blur.

Формат:

```text
float32 numpy array
значения нормируются в диапазон 0..1
```

Создается функцией:

```python
make_gaussian_heatmap_from_corridor(corridor_mask, sigma_px=gaussian_sigma_px)
```

### `projected_uv_gt.npy`

Массив спроецированных 2D-точек будущей траектории:

```text
shape: [N, 2]
dtype: float32
columns: u, v
```

Это не маска, а сама полилиния в координатах пикселей.

### `meta.json`

Метаданные одного примера. Содержит информацию о split, sample_id, camera_name, исходном sample_dir, TFRecord, параметрах генерации и диагностических числах.

---

## 7. Manifest-файлы

Для каждого split и камеры создается manifest:

```text
prepared_camera_corridor_dataset_filtered_front_v1/train/FRONT/manifest.jsonl
prepared_camera_corridor_dataset_filtered_front_v1/val/FRONT/manifest.jsonl
```

Каждая строка — JSON одного успешно экспортированного sample.

Типичная логика использования manifest:

```python
import json
from pathlib import Path

manifest_path = Path("prepared_camera_corridor_dataset_filtered_front_v1/train/FRONT/manifest.jsonl")
rows = [json.loads(line) for line in open(manifest_path, "r", encoding="utf-8")]
```

Далее из каждой строки берутся пути к:

```text
image.png
corridor_mask.png
centerline_mask.png
gaussian_heatmap.npy
projected_uv_gt.npy
meta.json
```

---

## 8. Как выбираются train и val samples

### Train

Train samples читаются из:

```text
precomputed_waymo_e2e/train/samples/*/meta.json
```

Функция:

```python
load_train_rows_from_samples(train_root: Path) -> List[dict]
```

Она проходит по всем директориям внутри:

```text
precomputed_waymo_e2e/train/samples/
```

и для каждой директории с `meta.json` собирает row:

```python
row = {
    "split": "train",
    "sample_dir": str(sample_dir.resolve())
}
row.update(meta)
```

### Val

Validation samples читаются из:

```text
precomputed_waymo_e2e/val/val_samples.jsonl
```

Функция:

```python
load_jsonl_rows(path: Path) -> List[dict]
```

читает файл построчно и парсит каждую строку как JSON.

---

## 9. Как находится sample directory

Для каждого row используется функция:

```python
resolve_sample_dir(row: dict, split_root: Optional[Path] = None) -> Path
```

Логика:

1. Берется `row["sample_dir"]`.
2. Если такой путь уже существует — он используется напрямую.
3. Если не существует, пробуется путь `split_root / row["sample_dir"]`.
4. Если и он не существует — выбрасывается `FileNotFoundError`.

После этого читается:

```text
<sample_dir>/meta.json
```

через функцию:

```python
load_meta_from_row(row, split_root)
```

---

## 10. Как находится оригинальный TFRecord

RGB-кадр и калибровка не лежат напрямую в `sample_dir`. Они извлекаются из оригинального Waymo E2E TFRecord.

Путь к shard берется из:

```python
meta["shard_path"]
```

Функция:

```python
resolve_tfrecord_path(meta: dict, raw_root: Path) -> Path
```

проверяет несколько вариантов:

```python
candidates = [
    shard_path,
    raw_root / shard_path,
    raw_root / shard_path.name,
]
```

Если ни один вариант не найден, делается рекурсивный поиск:

```python
matches = list(raw_root.rglob(shard_path.name))
```

Далее конкретный record внутри TFRecord выбирается по:

```python
record_idx = int(meta["record_idx"])
```

Функция чтения:

```python
load_e2ed_record(meta, RAW_WAYMO_ROOT)
```

Чтение устроено так:

```python
dataset = tf.data.TFRecordDataset(str(tfrecord_path), compression_type="")
for i, raw in enumerate(dataset):
    if i == record_idx:
        e2e = end_to_end_driving_data_pb2.E2EDFrame()
        e2e.ParseFromString(raw.numpy())
        return tfrecord_path, e2e
```

То есть для каждого sample ноутбук заново открывает соответствующий TFRecord и достает ровно один `E2EDFrame`.

---

## 11. Какая камера используется

Камера задается в ноутбуке так:

```python
CAMERA_NAME = dataset_pb2.CameraName.FRONT
```

То есть датасет фильтруется под одну камеру:

```text
FRONT
```

Имя камеры преобразуется в строку функцией:

```python
cam_name_to_str(camera_name)
```

Для текущей камеры строка:

```text
FRONT
```

Все результаты раскладываются в папки:

```text
train/FRONT/
val/FRONT/
```

---

## 12. Как извлекается изображение и калибровка камеры

Функция:

```python
get_camera_image_and_calib(frame, camera_name)
```

Ищет внутри `e2e.frame`:

```python
for img in frame.images:
    if img.name == camera_name:
        image_obj = img
```

и калибровку:

```python
for calib in frame.context.camera_calibrations:
    if calib.name == camera_name:
        calib_obj = calib
```

Изображение декодируется так:

```python
rgb = np.array(Image.open(io.BytesIO(image_obj.image)).convert("RGB"))
```

Калибровка `calib_obj` содержит:

1. Extrinsic transform камеры.
2. Intrinsic параметры камеры.
3. Distortion coefficients.

---

## 13. Как извлекается GT future trajectory

Целевая траектория извлекается функцией:

```python
extract_gt_future_xy(sample_dir: Path, row: dict, meta: dict) -> np.ndarray
```

Формат на выходе:

```text
shape: [T, 2]
dtype: float32
columns: x, y в vehicle frame
```

Функция сделана терпимой к разным вариантам хранения GT.

### 13.1. Сначала ищутся `.npy` файлы в sample_dir

Проверяются имена:

```python
GT_CANDIDATE_FILENAMES = [
    "gt_future_xy.npy",
    "gt.npy",
    "future_xy.npy",
    "traj_gt.npy",
]
```

Если файл найден, берутся первые две колонки:

```python
arr[:, :2].astype(np.float32)
```

### 13.2. Потом ищутся пути внутри `meta.json`

Проверяются ключи:

```python
meta_keys = [
    "gt_path",
    "gt_future_path",
    "future_xy_path",
    "traj_gt_path"
]
```

Если путь относительный, он интерпретируется относительно `sample_dir`.

### 13.3. Потом ищется массив прямо в `row` или `meta`

Проверяются ключи:

```python
direct_keys = [
    "gt",
    "future_xy",
    "traj_gt",
    "gt_future_xy"
]
```

Если ничего не найдено, выбрасывается:

```python
FileNotFoundError
```

---

## 14. Координатные системы

Это ключевая часть датасета.

### 14.1. GT trajectory

GT future trajectory хранится как двумерная траектория в vehicle frame:

```text
traj_xy = [x, y]
```

Здесь:

```text
x — вперед по ходу автомобиля
y — боковое смещение
```

В коде trajectory переводится в 3D-точки vehicle frame:

```python
traj_xy_to_vehicle_xyz(traj_xy, z_vehicle=0.0)
```

Функция добавляет координату `z`:

```python
z = np.full((len(traj_xy), 1), float(z_vehicle), dtype=np.float32)
pts_vehicle_xyz = np.concatenate([traj_xy, z], axis=1)
```

Параметр:

```python
Z_VEHICLE = 0.0
```

Это значит: будущая траектория считается лежащей на плоскости `z=0` в vehicle frame.

### 14.2. Camera extrinsic

Waymo calibration содержит transform:

```python
calib_obj.extrinsic.transform
```

В коде он читается так:

```python
def get_vehicle_from_camera(calib_obj) -> np.ndarray:
    return np.array(calib_obj.extrinsic.transform, dtype=np.float64).reshape(4, 4)
```

То есть матрица интерпретируется как:

```text
T_vehicle_camera
```

переводящая точки из camera frame в vehicle frame.

Для проекции нужна обратная матрица:

```python
def get_camera_from_vehicle(calib_obj) -> np.ndarray:
    return np.linalg.inv(get_vehicle_from_camera(calib_obj))
```

То есть:

```text
T_camera_vehicle = inverse(T_vehicle_camera)
```

### 14.3. Проекция vehicle points в camera frame

Сначала точки траектории переводятся из vehicle frame в camera frame:

```python
pts_cam = transform_points(get_camera_from_vehicle(calib_obj), pts_vehicle_xyz)
```

Далее используются координаты camera frame:

```python
x = pts_cam[:, 0]
y = pts_cam[:, 1]
z = pts_cam[:, 2]
```

Видимыми считаются точки, у которых:

```python
valid = x > 1e-6
```

То есть в этой реализации ось `x` camera frame считается направлением вперед из камеры.

---

## 15. Математика проекции в пиксели

Для валидных точек считаются нормированные координаты:

```python
xn = -y / x
yn = -z / x
```

Далее берутся intrinsic параметры камеры:

```python
intr = np.array(calib_obj.intrinsic, dtype=np.float64)
fu, fv, cu, cv = intr[:4]
k1, k2, p1, p2, k3 = intr[4:9]
```

Используется radial-tangential distortion:

```python
r2 = xn * xn + yn * yn
radial = 1.0 + k1 * r2 + k2 * r2 * r2 + k3 * r2 * r2 * r2
x_tan = 2 * p1 * xn * yn + p2 * (r2 + 2 * xn * xn)
y_tan = p1 * (r2 + 2 * yn * yn) + 2 * p2 * xn * yn

xd = xn * radial + x_tan
yd = yn * radial + y_tan

u = fu * xd + cu
v = fv * yd + cv
```

Итоговая точка изображения:

```text
(u, v)
```

где:

```text
u — горизонтальная координата пикселя
v — вертикальная координата пикселя
```

---

## 16. Фильтрация видимых точек

После проекции точки дополнительно фильтруются по границам изображения:

```python
inside = (
    np.isfinite(uv[:, 0]) & np.isfinite(uv[:, 1]) &
    (uv[:, 0] >= 0) & (uv[:, 0] < w) &
    (uv[:, 1] >= 0) & (uv[:, 1] < h)
)
```

Функция:

```python
clip_uv_to_image(uv, image_shape)
```

оставляет только точки, попавшие внутрь изображения.

Функция:

```python
project_traj_xy(traj_xy, calib_obj, image_shape, z_vehicle=0.0)
```

возвращает:

```text
uv — только видимые точки внутри изображения
pts_cam — соответствующие camera-frame точки, валидные по x > 1e-6
```

---

## 17. Генерация target-масок

### 17.1. Общая функция рисования полилинии

```python
_draw_polyline_mask(size_hw, uv, width)
```

Создает пустое grayscale-изображение:

```python
canvas = Image.new("L", (w, h), 0)
```

Рисует линию через PIL:

```python
draw.line(pts, fill=255, width=int(width), joint="curve")
```

Если точка одна, рисует круг:

```python
draw.ellipse((x-r, y-r, x+r, y+r), fill=255)
```

### 17.2. Centerline mask

```python
make_centerline_mask(uv, image_shape, line_width=3)
```

Параметр по умолчанию:

```text
line_width = 3 px
```

### 17.3. Corridor mask

```python
make_corridor_mask(uv, image_shape, corridor_width=17)
```

Параметр по умолчанию:

```text
corridor_width = 17 px
```

Это основной target, потому что он устойчивее тонкой линии и лучше переносит мелкие ошибки проекции.

### 17.4. Gaussian heatmap

```python
make_gaussian_heatmap_from_corridor(corridor_mask, sigma_px=7.0)
```

Алгоритм:

1. Берется бинарная corridor mask.
2. Применяется `ImageFilter.GaussianBlur(radius=sigma_px)`.
3. Массив переводится в `float32`.
4. Если максимум больше нуля, карта нормируется на максимум:

```python
arr = arr / arr.max()
```

Параметр по умолчанию:

```text
gaussian_sigma_px = 7.0
```

---

## 18. Условия отбраковки sample

При экспорте sample может быть пропущен через `ValueError`, если target получился невалидным.

Функция:

```python
build_sample(...)
```

использует следующие проверки.

### 18.1. Минимум видимых точек

```python
min_visible_points = 3
```

Если:

```python
visible_points < min_visible_points
```

sample пропускается.

### 18.2. Минимальная длина видимой траектории в пикселях

```python
min_visible_path_length_px = 25.0
```

Длина считается функцией:

```python
polyline_length_px(uv)
```

Она суммирует евклидовы расстояния между соседними projected UV точками.

Если:

```python
path_len_px < min_visible_path_length_px
```

sample пропускается.

### 18.3. Минимальное число пикселей corridor mask

```python
min_corridor_pixels = 40
```

Если:

```python
corridor_pixels < min_corridor_pixels
```

sample пропускается.

---

## 19. Экспорт одного sample

Главная функция одного примера:

```python
build_sample(
    row,
    split_root,
    split_name,
    out_root,
    camera_name,
    z_vehicle=0.0,
    corridor_width=17,
    centerline_width=3,
    gaussian_sigma_px=7.0,
    min_visible_points=3,
    min_visible_path_length_px=25.0,
    min_corridor_pixels=40,
)
```

Последовательность действий:

1. Найти `sample_dir` и `meta.json`.
2. Извлечь `gt_xy` future trajectory.
3. Найти и открыть исходный Waymo TFRecord.
4. Достать `E2EDFrame` по `record_idx`.
5. Извлечь FRONT RGB image и FRONT calibration.
6. Спроецировать `gt_xy` из vehicle frame в image plane.
7. Проверить число видимых точек.
8. Проверить длину projected path.
9. Сгенерировать `corridor_mask`.
10. Сгенерировать `centerline_mask`.
11. Сгенерировать `gaussian_heatmap`.
12. Проверить площадь corridor mask.
13. Сохранить файлы sample.
14. Вернуть metadata для manifest.

---

## 20. Экспорт split

Главная функция батчевого экспорта:

```python
export_split(
    rows,
    split_name,
    split_root,
    out_root,
    camera_name,
    z_vehicle=0.0,
    corridor_width=17,
    centerline_width=3,
    gaussian_sigma_px=7.0,
    min_visible_points=3,
    min_visible_path_length_px=25.0,
    min_corridor_pixels=40,
    limit=None,
)
```

Она создает:

```text
manifest.jsonl
train_FRONT.log / val_FRONT.log
train_FRONT_skipped.log / val_FRONT_skipped.log
train_FRONT_progress.json / val_FRONT_progress.json
train_FRONT_errors.jsonl / val_FRONT_errors.jsonl
```

### 20.1. Manifest path

```python
manifest_path = out_root / split_name / cam_str / "manifest.jsonl"
```

Для train:

```text
prepared_camera_corridor_dataset_filtered_front_v1/train/FRONT/manifest.jsonl
```

Для val:

```text
prepared_camera_corridor_dataset_filtered_front_v1/val/FRONT/manifest.jsonl
```

### 20.2. Logging

Для каждого split пишутся:

```text
_logs/<split>_FRONT.log
_logs/<split>_FRONT_skipped.log
_logs/<split>_FRONT_progress.json
_logs/<split>_FRONT_errors.jsonl
```

Логи пишутся синхронно с `flush()` и `os.fsync()`, чтобы прогресс не терялся при падении ядра.

### 20.3. Progress JSON

Progress JSON содержит:

```text
status
split
camera
started_at
updated_at
processed
total
ok
skipped
failed
keep_rate_percent
last_idx
last_sample_id
elapsed_sec
samples_per_sec
manifest_path
log_txt_path
skipped_log_path
errors_jsonl_path
error_type_counts
```

Это позволяет смотреть состояние долгого экспорта без чтения ноутбука.

---

## 21. Как запускался экспорт

В конце ноутбука вызываются два экспорта:

```python
manifest_train = export_split(
    rows=train_rows,
    split_name="train",
    split_root=TRAIN_ROOT,
    out_root=OUT_ROOT,
    camera_name=CAMERA_NAME,
    z_vehicle=Z_VEHICLE,
    corridor_width=CORRIDOR_WIDTH,
    centerline_width=CENTERLINE_WIDTH,
    gaussian_sigma_px=GAUSSIAN_SIGMA_PX,
    min_visible_points=MIN_VISIBLE_POINTS,
    min_visible_path_length_px=MIN_VISIBLE_PATH_LENGTH_PX,
    min_corridor_pixels=MIN_CORRIDOR_PIXELS,
)

manifest_val = export_split(
    rows=val_rows,
    split_name="val",
    split_root=VAL_ROOT,
    out_root=OUT_ROOT,
    camera_name=CAMERA_NAME,
    z_vehicle=Z_VEHICLE,
    corridor_width=CORRIDOR_WIDTH,
    centerline_width=CENTERLINE_WIDTH,
    gaussian_sigma_px=GAUSSIAN_SIGMA_PX,
    min_visible_points=MIN_VISIBLE_POINTS,
    min_visible_path_length_px=MIN_VISIBLE_PATH_LENGTH_PX,
    min_corridor_pixels=MIN_CORRIDOR_PIXELS,
)
```

В приложенной версии ноутбука сами переменные `CORRIDOR_WIDTH`, `CENTERLINE_WIDTH`, `GAUSSIAN_SIGMA_PX`, `MIN_VISIBLE_POINTS`, `MIN_VISIBLE_PATH_LENGTH_PX`, `MIN_CORRIDOR_PIXELS` используются в финальном вызове, но их явное присваивание в сохраненной версии ноутбука не видно. Однако значения по умолчанию в `build_sample` и `export_split` такие:

```text
corridor_width = 17
centerline_width = 3
gaussian_sigma_px = 7.0
min_visible_points = 3
min_visible_path_length_px = 25.0
min_corridor_pixels = 40
```

Если эти переменные были заданы в отдельной ячейке перед запуском, фактические параметры экспорта могли отличаться. Для строгой воспроизводимости нужно проверить `meta.json` конкретных sample или progress/log-файлы.

---

## 22. Как проверить один sample до массового экспорта

Для ручной проверки есть функция:

```python
debug_one_row(row, split_root=None, title="", corridor_width=17, centerline_width=3, gaussian_sigma_px=7.0)
```

Она делает:

1. Загружает sample.
2. Извлекает GT future trajectory.
3. Загружает Waymo frame.
4. Достает FRONT image и calibration.
5. Проецирует траекторию в изображение.
6. Строит centerline, corridor и heatmap.
7. Создает overlay.
8. Показывает grid из четырех изображений:

```text
RGB | centerline_mask | corridor_mask | overlay
```

Рекомендуемый запуск:

```python
debug_one_row(train_rows[0], split_root=TRAIN_ROOT, title="TRAIN")
debug_one_row(val_rows[0], split_root=VAL_ROOT, title="VAL")
```

---

## 23. Как использовать датасет для обучения

Основной режим обучения:

```text
input:  image.png
output: corridor_mask.png
```

Пример PyTorch Dataset:

```python
import json
from pathlib import Path
from PIL import Image
import numpy as np
import torch
from torch.utils.data import Dataset

class CorridorDataset(Dataset):
    def __init__(self, manifest_path, transform=None):
        self.manifest_path = Path(manifest_path)
        self.root = self.manifest_path.parent
        self.rows = [json.loads(line) for line in open(self.manifest_path, "r", encoding="utf-8")]
        self.transform = transform

    def __len__(self):
        return len(self.rows)

    def __getitem__(self, idx):
        row = self.rows[idx]

        # Если manifest/meta хранит абсолютные пути — использовать их напрямую.
        # Если относительные — резолвить относительно директории sample или OUT_ROOT.
        sample_dir = Path(row.get("output_dir", row.get("out_dir", "")))
        if not sample_dir.exists():
            sample_dir = Path(row["sample_dir"])

        image = Image.open(sample_dir / "image.png").convert("RGB")
        mask = Image.open(sample_dir / "corridor_mask.png").convert("L")

        image = np.asarray(image, dtype=np.float32) / 255.0
        mask = (np.asarray(mask, dtype=np.float32) > 0).astype(np.float32)

        image = torch.from_numpy(image).permute(2, 0, 1)
        mask = torch.from_numpy(mask)[None, ...]

        return image, mask
```

На практике лучше читать точные пути из `meta.json`/manifest, потому что конкретные ключи metadata надо сверить с фактическим экспортом.

---

## 24. Рекомендованный основной target

Основной target для первой модели:

```text
corridor_mask.png
```

Причина:

1. Centerline mask слишком тонкая и чувствительна к ошибкам проекции.
2. Corridor mask задает область допустимого движения, а не единственную пиксельную линию.
3. Corridor mask лучше подходит для сегментационной модели типа U-Net.
4. Corridor mask проще визуально проверять.

Альтернативные target-режимы:

```text
centerline_mask.png      # жесткая линия
 gaussian_heatmap.npy    # мягкое распределение около коридора
```

---

## 25. Что технически важно для статьи/описания

Главная идея датасета: **используется чужой датасет Waymo Open Dataset End-to-End для автоматического получения собственного датасета image-space trajectory/corridor prediction**.

Waymo содержит:

1. Сырые изображения с камер.
2. Калибровки камер.
3. GT future trajectory / precomputed trajectory samples.
4. TFRecord-структуру с E2EDFrame.

Собственный датасет строится не ручной разметкой, а программной проекцией GT trajectory на изображение через camera calibration.

То есть pipeline такой:

```text
Waymo E2E TFRecord
        ↓
E2EDFrame
        ↓
FRONT RGB image + FRONT camera calibration
        ↓
precomputed GT future trajectory in vehicle frame
        ↓
vehicle-frame trajectory -> camera frame -> pixel coordinates
        ↓
projected_uv_gt.npy
        ↓
centerline_mask / corridor_mask / gaussian_heatmap
        ↓
prepared_camera_corridor_dataset_filtered_front_v1
```

---

## 26. Ограничения датасета

### 26.1. Используется только FRONT camera

Модель обучается на одной камере, а не на multi-camera setup. Это делает постановку проще и ближе к задаче:

```text
single image -> future corridor on image
```

Но модель не использует боковые камеры и не видит полную сцену вокруг автомобиля.

### 26.2. Target строится из одной GT future trajectory

Если на дороге есть несколько допустимых направлений движения, датасет содержит только ту траекторию, по которой реально ехал автомобиль в записи.

Это значит, что target не описывает все возможные corridors, а только ego future corridor.

### 26.3. Нет явного intent/route input

В текущем сохраненном датасете основной вход — RGB. Если на перекрестке есть несколько вариантов, одной картинки может быть недостаточно, чтобы однозначно выбрать будущий маршрут.

Это важно для интерпретации ошибок модели.

### 26.4. Target зависит от качества калибровки и корректности GT

Если camera calibration, extrinsic или GT trajectory некорректны, target mask будет смещена.

### 26.5. Геометрия строится в image space

Модель предсказывает не 3D-траекторию, а маску/коридор в координатах изображения.

Это полезно для визуального планирования и сегментационной постановки, но не является полноценным планом движения в metric BEV/3D.

---

## 27. Минимальная команда проверки результата

После экспорта ноутбук печатает:

```python
print('OUT_ROOT =', OUT_ROOT.resolve())
print('train manifest =', (OUT_ROOT / 'train' / 'FRONT' / 'manifest.jsonl').resolve())
print('val manifest   =', (OUT_ROOT / 'val' / 'FRONT' / 'manifest.jsonl').resolve())
print('train log      =', (OUT_ROOT / '_logs' / 'train_FRONT.log').resolve())
print('val log        =', (OUT_ROOT / '_logs' / 'val_FRONT.log').resolve())
print('train progress =', (OUT_ROOT / '_logs' / 'train_FRONT_progress.json').resolve())
print('val progress   =', (OUT_ROOT / '_logs' / 'val_FRONT_progress.json').resolve())
```

Ожидаемые пути:

```text
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/train/FRONT/manifest.jsonl
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/val/FRONT/manifest.jsonl
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/_logs/train_FRONT.log
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/_logs/val_FRONT.log
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/_logs/train_FRONT_progress.json
/home/Jupyter/datasets/tesla/Waymo_open_dataset/prepared_camera_corridor_dataset_filtered_front_v1/_logs/val_FRONT_progress.json
```

---

## 28. Короткое техническое резюме

Датасет `prepared_camera_corridor_dataset_filtered_front_v1` — это производный датасет поверх Waymo Open Dataset End-to-End. Для каждого sample берется FRONT RGB image, GT future trajectory в vehicle frame и camera calibration из Waymo. Траектория переводится в camera frame, проецируется в пиксели с учетом intrinsics и distortion, после чего из projected polyline строятся три target-представления: `corridor_mask.png`, `centerline_mask.png`, `gaussian_heatmap.npy`. Основной target для обучения — `corridor_mask.png`. Основной вход — `image.png`. Все валидные samples записываются в split/camera директории и индексируются через `manifest.jsonl`; процесс экспорта логируется в `_logs`.
