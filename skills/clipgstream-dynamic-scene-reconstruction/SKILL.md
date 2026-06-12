```yaml
---
name: clipgstream-dynamic-scene-reconstruction
description: Use ClipGStream for scalable multi-view dynamic scene reconstruction with Gaussian Splatting on long sequences
triggers:
  - reconstruct dynamic scenes with gaussian splatting
  - train clipgstream on multi-view video
  - process long dynamic sequences with clipgstream
  - render novel views from gaussian splatting
  - setup clipgstream for custom dataset
  - optimize multi-view video reconstruction
  - eliminate flickering in dynamic scene reconstruction
  - train reference and source clips separately
---
```

# ClipGStream Dynamic Scene Reconstruction

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

ClipGStream is a hybrid reconstruction framework for multi-view dynamic scene reconstruction using Gaussian Splatting. It performs stream-level optimization at clip granularity (rather than frame-level), enabling scalable and temporally coherent reconstruction of long dynamic sequences while eliminating flickering artifacts.

**Key Features:**
- Reconstructs any-length multi-view dynamic scenes
- Clip-based streaming approach prevents flickering between frames
- Supports parallel training of source clips
- Works with multi-view camera setups (e.g., 36 cameras for 360° capture)
- Includes custom dataset preprocessing pipeline

## Installation

### Prerequisites
- Ubuntu 20.04+ (tested)
- CUDA 11.8+ and matching GCC (e.g., gcc-11)
- Conda/Miniconda

### Setup Steps

```bash
# Clone repository with submodules
git clone https://github.com/liangjie1999/ClipGStream --recursive
cd ClipGStream

# Create conda environment
conda env create --file environment.yml
conda activate ClipGStream

# Install diff-gaussian-rasterization
cd submodules/diff-gaussian-rasterization
CC=gcc-11 CXX=g++-11 CUDAHOSTCXX=/usr/bin/g++-11 pip install . --no-build-isolation
cd ../..

# Install simple-knn
cd submodules/simple-knn
CC=gcc-11 CXX=g++-11 CUDAHOSTCXX=/usr/bin/g++-11 pip install . --no-build-isolation
cd ../..

# Install tiny-cuda-nn
cd tiny-cuda-nn-master/bindings/torch
python setup.py install
cd ../../..
```

**Note:** If you encounter compiler issues, remove the `CC=gcc-11 CXX=g++-11 CUDAHOSTCXX=/usr/bin/g++-11` prefix and use your default compiler.

## Dataset Structure

ClipGStream expects the following directory structure:

```
dataset_root/
├── frame000000/
│   ├── sparse/          # Camera parameters (COLMAP format)
│   └── images/          # Undistorted images
│       ├── cam_00.jpg
│       ├── cam_01.jpg
│       └── ...
├── frame000001/
│   └── ...
├── plys/
│   ├── 0.ply           # Reference clip point cloud
│   ├── 1.ply           # Source clip 1 residual point cloud
│   └── ...
└── sparse/             # Global camera info (optional)
```

## Training Workflow

ClipGStream divides long sequences into clips and trains in two stages:

### 1. Train Reference Clip (First Clip)

The reference clip establishes the foundational scene representation:

```python
# Train reference clip (frames 0-10 of 20 total frames)
python trainReferenceClip.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 0 10 \
    --iterations 5000 \
    -s "/path/to/dataset/" \
    -m ./output/my_scene \
    --configs arguments/tiny/basketball.py
```

**Key Parameters:**
- `--project_total_frames`: Total number of frames in entire sequence (N)
- `--clip_size`: Frames per clip (M)
- `--frames_start_end`: Start and end frame indices for this clip
- `-s`: Source dataset path
- `-m`: Model output directory
- `--configs`: Configuration file (defines network architecture, learning rates, etc.)

### 2. Train Source Clips (Subsequent Clips)

Source clips inherit static information from the reference clip:

```python
# Train source clip 1 (frames 10-20)
python trainSourceClip.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 10 20 \
    --iterations 5000 \
    -s "/path/to/dataset/" \
    -m ./output/my_scene \
    --configs arguments/tiny/basketball.py
```

**Parallel Training:**
Each source clip can be trained independently on different GPUs:

```bash
# GPU 0: Source clip 1
CUDA_VISIBLE_DEVICES=0 python trainSourceClip.py --frames_start_end 10 20 ...

# GPU 1: Source clip 2
CUDA_VISIBLE_DEVICES=1 python trainSourceClip.py --frames_start_end 20 30 ...

# GPU 2: Source clip 3
CUDA_VISIBLE_DEVICES=2 python trainSourceClip.py --frames_start_end 30 40 ...
```

## Rendering & Evaluation

### Render Novel Views

```python
# Render all test views for frames 0-20
python render.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 0 20 \
    --iteration 5000 \
    -s "/path/to/dataset/" \
    -m ./output/my_scene \
    --configs arguments/tiny/basketball.py \
    --skip_train \
    --skip_video
```

**Flags:**
- `--skip_train`: Don't render training views (only test views)
- `--skip_video`: Skip video generation (only render images)

**Output Structure:**
```
output/my_scene/
├── history/
│   ├── decoder/         # Trained MLP weights
│   ├── point/           # Point clouds per clip
│   └── FDHash_*.pth     # Feature hash grids
└── test/
    ├── renders/         # Rendered images
    ├── videos/          # Compiled videos
    └── *.csv            # Metrics per test view
```

### Compute Metrics

```bash
# Calculate PSNR, SSIM, LPIPS for all test views
python metrics.py -m ./output/my_scene --iteration 5000
```

Outputs CSV files with metrics per test view in `output/my_scene/test/`.

### Generate Videos from Rendered Images

```bash
# Compile rendered images into video files
python images2video.py -m ./output/my_scene/ --iteration 5000
```

## Custom Dataset Preparation

### Multi-View Video Processing

For custom multi-view video streams, use the preprocessing pipeline:

```bash
cd data_process/custom_dataset

# 1. Extract frames from videos
python step1_video2images.py --input_dir /path/to/videos --output_dir /path/to/frames

# 2. Run COLMAP for camera calibration (first frame only)
python step2_colmap.py --image_dir /path/to/frames/frame000000 --output_dir /path/to/sparse

# 3. Undistort all frames using camera parameters
python step3_undistort.py --sparse_dir /path/to/sparse --frames_dir /path/to/frames

# 4. Generate point clouds per clip
python step4_pointcloud.py --frames_dir /path/to/frames --clip_size 10
```

See [`data_process/custom_dataset/README.md`](data_process/custom_dataset/README.md) for detailed instructions.

## Configuration Files

Configuration files define network architecture and training hyperparameters:

**Example: `arguments/tiny/basketball.py`**

```python
# MLP decoder architecture
mlp_opacity_args = {
    'hidden_dim': 32,
    'n_layers': 2,
}

mlp_cov_args = {
    'hidden_dim': 64,
    'n_layers': 3,
}

# Feature hash grid settings
hash_grid_args = {
    'n_levels': 16,
    'n_features_per_level': 2,
    'log2_hashmap_size': 19,
}

# Learning rates
lr_opacity = 0.005
lr_scale = 0.005
lr_color = 0.005
lr_offset = 0.005

# Densification parameters
densify_grad_threshold = 0.0002
opacity_reset_interval = 3000
```

## Common Patterns

### Long Sequence Training (100+ frames)

```bash
# Define sequence parameters
TOTAL_FRAMES=300
CLIP_SIZE=30
SOURCE_PATH="/path/to/long_sequence"
OUTPUT_DIR="./output/long_scene"
CONFIG="arguments/long_360/basketball.py"
ITERATIONS=30000

# Train reference clip (frames 0-30)
python trainReferenceClip.py \
    --project_total_frames $TOTAL_FRAMES \
    --clip_size $CLIP_SIZE \
    --frames_start_end 0 30 \
    --iterations $ITERATIONS \
    -s $SOURCE_PATH \
    -m $OUTPUT_DIR \
    --configs $CONFIG

# Train source clips in parallel
for i in {1..9}; do
    START=$((i * CLIP_SIZE))
    END=$(((i + 1) * CLIP_SIZE))
    GPU=$((i % 4))  # Distribute across 4 GPUs
    
    CUDA_VISIBLE_DEVICES=$GPU python trainSourceClip.py \
        --project_total_frames $TOTAL_FRAMES \
        --clip_size $CLIP_SIZE \
        --frames_start_end $START $END \
        --iterations $ITERATIONS \
        -s $SOURCE_PATH \
        -m $OUTPUT_DIR \
        --configs $CONFIG &
done
wait  # Wait for all parallel jobs
```

### Rendering Specific Frame Range

```python
# Render only frames 50-80 for visualization
python render.py \
    --project_total_frames 300 \
    --clip_size 30 \
    --frames_start_end 50 80 \
    --iteration 30000 \
    -s $SOURCE_PATH \
    -m $OUTPUT_DIR \
    --configs $CONFIG \
    --skip_train
```

### Resume Training from Checkpoint

```python
# ClipGStream automatically saves checkpoints in history/
# To resume, simply run the same command with --start_checkpoint
python trainSourceClip.py \
    --project_total_frames 300 \
    --clip_size 30 \
    --frames_start_end 60 90 \
    --iterations 30000 \
    -s $SOURCE_PATH \
    -m $OUTPUT_DIR \
    --configs $CONFIG \
    --start_checkpoint 15000  # Resume from iteration 15000
```

## Troubleshooting

### CUDA Out of Memory

**Issue:** GPU memory overflow during training.

**Solutions:**
1. Reduce batch size in config file
2. Decrease `clip_size` (split into more, smaller clips)
3. Lower hash grid size: `log2_hashmap_size: 18` instead of `19`
4. Use gradient checkpointing (add `--checkpoint_gradients` flag if available)

### Poor Reconstruction Quality

**Issue:** Low PSNR/SSIM, blurry renders.

**Solutions:**
1. Increase training iterations: `--iterations 30000` (default 5000 is for tiny dataset)
2. Verify point cloud quality (`plys/*.ply` files should be dense)
3. Check camera calibration (visualize `sparse/` in COLMAP GUI)
4. Adjust densification threshold: lower `densify_grad_threshold` in config

### Flickering Between Clips

**Issue:** Temporal inconsistency at clip boundaries.

**Solutions:**
1. Ensure source clips inherit from reference clip (check `-m` path matches)
2. Increase clip overlap (modify `frames_start_end` to have 2-3 frame overlap)
3. Verify all clips use same config file
4. Check that `FDHash_*.pth` files exist for all clips in `output/history/`

### COLMAP Camera Calibration Fails

**Issue:** No sparse reconstruction generated.

**Solutions:**
1. Ensure sufficient image overlap between views
2. Use high-quality, sharp images (avoid motion blur)
3. Increase COLMAP feature extraction quality
4. Manually provide initial camera parameters if available

### Missing Dependencies

**Issue:** Import errors for `tinycudann` or rasterization modules.

**Solutions:**
```bash
# Verify CUDA version matches
echo $CUDA_HOME  # Should point to CUDA 11.8+

# Reinstall tiny-cuda-nn with correct CUDA
cd tiny-cuda-nn-master/bindings/torch
pip uninstall tinycudann
python setup.py install

# Rebuild diff-gaussian-rasterization
cd submodules/diff-gaussian-rasterization
pip uninstall diff-gaussian-rasterization
pip install . --no-build-isolation
```

## Quick Start Example

```bash
# Download tiny dataset (20 frames, preprocessed)
# From: https://github.com/liangjie1999/ClipGStream/releases/tag/v1.0.0

# Extract to /data/long_360_tiny_dataset

# Train reference clip (frames 0-10)
python trainReferenceClip.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 0 10 \
    --iterations 5000 \
    -s "/data/long_360_tiny_dataset/" \
    -m ./output/tiny_demo \
    --configs arguments/tiny/basketball.py

# Train source clip (frames 10-20)
python trainSourceClip.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 10 20 \
    --iterations 5000 \
    -s "/data/long_360_tiny_dataset/" \
    -m ./output/tiny_demo \
    --configs arguments/tiny/basketball.py

# Render and evaluate
python render.py \
    --project_total_frames 20 \
    --clip_size 10 \
    --frames_start_end 0 20 \
    --iteration 5000 \
    -s "/data/long_360_tiny_dataset/" \
    -m ./output/tiny_demo \
    --configs arguments/tiny/basketball.py \
    --skip_train

# Compute metrics
python metrics.py -m ./output/tiny_demo --iteration 5000

# Generate videos
python images2video.py -m ./output/tiny_demo --iteration 5000
```

**Expected Results (Tiny Dataset):**
- PSNR: ~23.47
- DSSIM1: ~0.105
- LPIPS: ~0.209
