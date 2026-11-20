# Evaluation

> **Note:** Evaluation code (`evaluate_depth.py`) **is** included in this release. The evaluation commands reference
> pretrained weights placed in `./models/` — see [Pretrained Models](../README.md#pretrained-models).
>
> To just run a model on your own images rather than a benchmark split, see [Inference](inference.md).

## KITTI

The default evaluation split is `eigen`, with 697 test images. Generate its
ground truth from the raw Velodyne point clouds and calibration files before
evaluating:

```bash
python export_gt_depth.py --data_path <kitti_data_path> --split eigen
```

This creates `splits/eigen/gt_depths.npz`. If using a custom `--split_path`, place
the generated file in its `eigen/` directory alongside the matching
`test_files.txt`. The `eigen_benchmark` split uses 652 images and separate depth
annotations; its GT file cannot be copied into `eigen/`, because both the sample
selection and order differ. To evaluate those annotations, pass
`--eval_split eigen_benchmark`. Evaluation checks GT counts before inference
and prediction counts before computing metrics.

Make3D evaluation runs after KITTI evaluation when `--make3d_path` contains
`make3d_test_files.txt`; otherwise it is skipped.

```bash
# Flex-Nano
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11n-seg --decoder_model_type flexn \
    --load_weights_folder ./models/kitti/flex_n \
    --data_path <kitti_data_path> --split_path <splits_path>

# Flex-Small
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11s-seg --decoder_model_type flexs \
    --load_weights_folder ./models/kitti/flex_s \
    --data_path <kitti_data_path> --split_path <splits_path>

# Flex-Medium
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11m-seg --decoder_model_type flexm \
    --load_weights_folder ./models/kitti/flex_m \
    --data_path <kitti_data_path> --split_path <splits_path>

# Flex-Large
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11l-seg --decoder_model_type flexl \
    --load_weights_folder ./models/kitti/flex_l \
    --data_path <kitti_data_path> --split_path <splits_path>

# Flex-X-Large
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11x-seg --decoder_model_type flexx \
    --load_weights_folder ./models/kitti/flex_x \
    --data_path <kitti_data_path> --split_path <splits_path>
```

## Comparison with Depth Anything 2 (Eigen benchmark, Least-Squares Alignment)

The improved Ground Truth  uses 5 consecutive frames with stereo completion to handle dynamic objects,covering 652 of 697 Eigen split test frames (93%) ,This split is usually called the KITTI Eigen benchmark split as monodepth2 introduce. benchmark labeled data from [official web](https://www.cvlibs.net/datasets/kitti/eval_depth.php?benchmark=depth_prediction) and follow [monodepth2](https://github.com/nianticlabs/monodepth2) sec. KITTI evaluation use prepare gt_depth.npz in ./splits/eigen_benchmark:
```bash 
python export_gt_depth.py --data_path kitti_data --split eigen_benchmark
```
To compare with Depth Anything 2 and other zero-shot depth models, we evaluate Flex-X-Large on the KITTI Eigen benchmark split using **dense ground truth** with **least-squares alignment** (instead of the median scaling alignment used in the standard evaluation above). This aligns with the evaluation protocol used by DA2.

First, generate the dense ground truth (following [Monodepth2](https://github.com/nianticlabs/monodepth2)):

```bash
python export_gt_depth.py --data_path <kitti_data_path> --split eigen_benchmark
```

RGB photographs may be JPG or PNG. Ground truth for `eigen_benchmark` comes from
the separate 16-bit PNG depth annotations in KITTI's `data_depth_annotated`
download; changing an RGB image's extension cannot provide depth ground truth.
The exporter supports either annotations merged into each raw sequence or the
official extracted layout under the same data root:

```text
<kitti_data_path>/
  2011_09_26/2011_09_26_drive_0002_sync/image_02/data/0000000069.jpg
  data_depth_annotated/
    train/<drive>/proj_depth/groundtruth/image_02/<frame>.png
    val/<drive>/proj_depth/groundtruth/image_02/<frame>.png
```

The result is saved to `splits/eigen_benchmark/gt_depths.npz`. For JPG RGB inputs,
omit `--png` from the evaluation command below.

Then evaluate:

```bash
python evaluate_depth.py --png --eval_mono --scale 4 \
    --encoder_model_type yolo11x-seg --decoder_model_type flexx \
    --load_weights_folder ./models/kitti/flex_x \
    --data_path <kitti_data_path> --split_path <splits_path> \
    --eval_split eigen_benchmark --use_lstsq_alignment
```

## Cityscapes

```bash
# Flex-Nano
python evaluate_depth.py --eval_mono --scale 4 \
    --dataset cityscapes_preprocessed --eval_split cityscapes \
    --encoder_model_type yolo11n-seg --decoder_model_type flexn \
    --load_weights_folder ./models/cs/flex_n \
    --data_path <cityscapes_path> --split_path <splits_path>

# Flex-Small
python evaluate_depth.py --eval_mono --scale 4 \
    --dataset cityscapes_preprocessed --eval_split cityscapes \
    --encoder_model_type yolo11s-seg --decoder_model_type flexs \
    --load_weights_folder ./models/cs/flex_s \
    --data_path <cityscapes_path> --split_path <splits_path>

# Flex-Medium
python evaluate_depth.py --eval_mono --scale 4 \
    --dataset cityscapes_preprocessed --eval_split cityscapes \
    --encoder_model_type yolo11m-seg --decoder_model_type flexm \
    --load_weights_folder ./models/cs/flex_m \
    --data_path <cityscapes_path> --split_path <splits_path>

# Flex-Large
python evaluate_depth.py --eval_mono --scale 4 \
    --dataset cityscapes_preprocessed --eval_split cityscapes \
    --encoder_model_type yolo11l-seg --decoder_model_type flexl \
    --load_weights_folder ./models/cs/flex_l \
    --data_path <cityscapes_path> --split_path <splits_path>

# Flex-X-Large
python evaluate_depth.py --eval_mono --scale 4 \
    --dataset cityscapes_preprocessed --eval_split cityscapes \
    --encoder_model_type yolo11x-seg --decoder_model_type flexx \
    --load_weights_folder ./models/cs/flex_x \
    --data_path <cityscapes_path> --split_path <splits_path>
```

## Results

**KITTI Eigen split (Cap 80m):**

| Model | Params | GFLOPs | Abs Rel ↓ | Sq Rel ↓ | RMSE ↓ | RMSE log ↓ | δ<1.25 ↑ | δ<1.25² ↑ | δ<1.25³ ↑ |
|-------|--------|--------|-----------|----------|--------|------------|----------|-----------|-----------|
| Flex-Nano | 1.5M | 0.7 | 0.110 | 0.794 | 4.678 | 0.184 | 0.878 | 0.961 | 0.983 |
| Flex-Small | 6.1M | 2.8 | 0.104 | 0.713 | 4.458 | 0.179 | 0.890 | 0.964 | 0.983 |
| Flex-Medium | 12.7M | 10.0 | 0.096 | 0.639 | 4.253 | 0.172 | 0.903 | 0.968 | 0.985 |
| Flex-Large | 15.2M | 11.5 | 0.095 | 0.642 | 4.199 | 0.171 | 0.906 | 0.968 | 0.984 |
| Flex-X-Large | 32.3M | 24.6 | **0.093** | **0.605** | **4.114** | **0.167** | **0.910** | **0.969** | **0.985** |

**KITTI Eigen benchmark (Dense GT, Least-Squares Alignment):**

| Method | Type | Params | GFLOPs | Resolution | Abs Rel ↓ | δ<1.25 ↑ |
|--------|------|--------|--------|------------|-----------|----------|
| DA2 (ViT-L) | Zero-Shot | 335M | 1947 | 1722×518 | 0.070 | **0.956** |
| DA2 (ViT-S) | Zero-Shot | 25M | 137 | 1722×518 | 0.077 | 0.944 |
| DA2 (ViT-L) | Zero-Shot | 335M | 276 | 644×196 | 0.092 | 0.915 |
| DA2 (ViT-S) | Zero-Shot | 25M | 19 | 644×196 | 0.110 | 0.881 |
| Flex-X-Large (Ours) | Self-Supervised | 32M | 25 | 640×192 | **0.063** | 0.952 |

**Cityscapes (During evaluation, crop follow manydepth,pro depth etc.):**

| Model | Params | GFLOPs | Abs Rel ↓ | Sq Rel ↓ | RMSE ↓ | RMSE log ↓ | δ<1.25 ↑ | δ<1.25² ↑ | δ<1.25³ ↑ |
|-------|--------|--------|-----------|----------|--------|------------|----------|-----------|-----------|
| Flex-Nano | 1.5M | 0.6 | 0.107 | 1.261 | 6.133 | 0.164 | 0.893 | 0.971 | 0.989 |
| Flex-Small | 6.1M | 2.2 | 0.100 | 1.078 | 5.813 | 0.153 | 0.904 | 0.975 | 0.991 |
| Flex-Medium | 12.7M | 8.0 | 0.089 | 0.885 | 5.358 | 0.143 | 0.917 | 0.979 | 0.993 |
| Flex-Large | 15.2M | 9.2 | 0.087 | 0.911 | 5.310 | 0.139 | 0.924 | 0.981 | 0.993 |
| Flex-X-Large | 32.3M | 19.7 | **0.086** | **0.877** | **5.268** | **0.137** | **0.926** | **0.982** | **0.993** |
