---
name: clip-finetune-recipes
description: Fine-tune CLIP models with DDP, LoRA, hard-negative mining, and leakage checks
triggers:
  - fine-tune CLIP model
  - train CLIP on custom data
  - CLIP distributed training
  - CLIP LoRA fine-tuning
  - contrastive learning image text
  - hard negative mining CLIP
  - evaluate CLIP zero-shot
  - CLIP dataset deduplication
---

# CLIP Fine-tuning Recipes Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

`clip-finetune-recipes` is a production-ready toolkit for fine-tuning CLIP-style dual encoders on custom image-text data. It emphasizes practical hygiene: data leakage prevention, proper contrastive batch construction for multi-GPU training, and stable learning rate schedules that preserve temperature parameters.

Key features:
- **Streaming data pipeline** with webdataset shards and automatic deduplication
- **DDP training** with proper local-loss aggregation across GPUs
- **LoRA, full fine-tuning, and linear probe** training modes
- **Hard-negative mining** for both text and image sides
- **Evaluation hooks** for zero-shot classification and retrieval tasks
- **Sanity checks** for temperature drift, gradient norms, and embedding collapse

## Installation

```bash
# Basic installation
pip install clip-recipes

# From source with training dependencies
pip install -e .[train]

# For Chinese/multilingual experiments
pip install clip-recipes[zh]
```

## Quick Start

### Single GPU Training

```bash
# Sanity run with bundled dataset
python -m clip_recipes.train --config configs/quickstart.yaml
```

### Multi-GPU Training (DDP)

```bash
# 4xA100 run with LoRA
torchrun --nproc_per_node=4 -m clip_recipes.train \
    --config configs/laion_400m_lora.yaml \
    --output_dir runs/laion_lora_v3

# 8-GPU run with custom config overrides
torchrun --nproc_per_node=8 -m clip_recipes.train \
    --config configs/laion_full_ft.yaml \
    train.lr=5e-5 \
    train.batch_size=1024 \
    model.lora_r=16 \
    --output_dir runs/full_ft_8gpu
```

## Configuration

All configurations are in YAML format. Override any parameter via CLI with `key=value` syntax:

```bash
python -m clip_recipes.train --config configs/quickstart.yaml \
    train.lr=3e-5 \
    train.batch_size=512 \
    train.warmup_steps=1000 \
    model.lora_r=8 \
    model.lora_alpha=16 \
    data.num_workers=8
```

### Example Configuration File

```yaml
# configs/custom_lora.yaml
model:
  name: "openai/clip-vit-base-patch32"
  lora_r: 8
  lora_alpha: 16
  lora_dropout: 0.1
  target_modules: ["q_proj", "v_proj"]

train:
  lr: 3e-5
  batch_size: 256
  epochs: 10
  warmup_steps: 500
  weight_decay: 0.01
  grad_clip: 1.0
  fp16: true

data:
  train_shards: "data/train/{00000..00999}.tar"
  eval_shards: "data/eval/{00000..00099}.tar"
  num_workers: 4
  shuffle_buffer: 10000

loss:
  type: "contrastive"  # or "sigclip"
  temperature_init: 0.07
  learnable_temperature: true

eval:
  every_n_steps: 1000
  datasets: ["imagenet1k", "flickr30k-cn"]
```

## Data Preparation

### Building Webdataset Shards

```python
# tools/build_shards.py
from clip_recipes.data.builder import ShardBuilder

builder = ShardBuilder(
    output_pattern="data/train/shard-%06d.tar",
    samples_per_shard=10000,
    dedup_against=["data/eval_hashes.txt"]  # Prevent leakage
)

# Add samples (image bytes + text)
for img_path, caption in dataset:
    with open(img_path, "rb") as f:
        img_bytes = f.read()
    builder.add_sample(img_bytes, caption)

builder.finalize()
```

### Deduplication Against Eval Sets

```python
from clip_recipes.data.dedup import compute_hash, build_hash_set

# Build hash set from eval data
eval_hashes = build_hash_set("data/eval/{00000..00099}.tar")

# Check for duplicates before training
from clip_recipes.data.dedup import deduplicate_shards

deduplicate_shards(
    input_pattern="data/train/{00000..00999}.tar",
    output_pattern="data/train_dedup/{00000..00999}.tar",
    exclude_hashes=eval_hashes
)
```

## Training Modes

### LoRA Fine-tuning

```python
# configs/lora_config.yaml
model:
  name: "openai/clip-vit-large-patch14"
  lora_r: 16
  lora_alpha: 32
  lora_dropout: 0.1
  target_modules: ["q_proj", "v_proj", "k_proj", "out_proj"]
  freeze_vision: false
  freeze_text: false
```

```bash
torchrun --nproc_per_node=4 -m clip_recipes.train \
    --config configs/lora_config.yaml \
    --output_dir runs/lora_large
```

### Full Fine-tuning

```yaml
# configs/full_ft.yaml
model:
  name: "openai/clip-vit-base-patch32"
  # No LoRA params = full fine-tuning

train:
  lr: 1e-5  # Lower LR for full fine-tuning
  batch_size: 512
  epochs: 5
```

### Linear Probe (Freeze Encoders)

```yaml
# configs/linear_probe.yaml
model:
  name: "openai/clip-vit-base-patch32"
  freeze_vision: true
  freeze_text: true
  # Only projection heads are trained

train:
  lr: 1e-3  # Higher LR for linear probe
  batch_size: 1024
  epochs: 20
```

## Contrastive Loss with Cross-GPU Negatives

The key to effective multi-GPU CLIP training is proper negative aggregation:

```python
from clip_recipes.losses.contrastive import ContrastiveLoss

# In your training loop
loss_fn = ContrastiveLoss(
    temperature_init=0.07,
    learnable=True,
    gather_with_grad=True  # Critical for DDP
)

# Forward pass
image_embeds = model.encode_image(images)  # (local_batch, dim)
text_embeds = model.encode_text(texts)      # (local_batch, dim)

# Loss automatically gathers across all GPUs
loss = loss_fn(image_embeds, text_embeds)
```

**Why this matters**: Without `gather_with_grad=True`, each GPU only sees its local batch as negatives (e.g., 256 samples instead of 1024 on 4 GPUs), severely degrading contrastive learning.

## Hard Negative Mining

```python
from clip_recipes.data.hard_negatives import HardNegativeMiner

miner = HardNegativeMiner(
    index_path="faiss_index.bin",
    k_negatives=5,
    sample_rate=0.3  # 30% of batches use hard negatives
)

# During training
for batch in dataloader:
    images, texts = batch
    
    # Optionally inject hard negatives
    if miner.should_mine():
        hard_texts = miner.mine_text_negatives(images)
        texts = torch.cat([texts, hard_texts])
    
    loss = train_step(images, texts)
```

### Building FAISS Index for Mining

```python
from clip_recipes.data.hard_negatives import build_faiss_index

# Extract embeddings from your dataset
embeddings = []
for batch in dataloader:
    with torch.no_grad():
        emb = model.encode_text(batch["text"])
        embeddings.append(emb.cpu())

embeddings = torch.cat(embeddings).numpy()

# Build and save index
build_faiss_index(
    embeddings,
    output_path="faiss_index.bin",
    index_type="IVF1024,Flat"  # or "Flat" for small datasets
)
```

## Evaluation

### Zero-shot Classification

```bash
# Evaluate on ImageNet
python -m clip_recipes.eval.zeroshot \
    --checkpoint runs/laion_lora_v3/final.pt \
    --dataset imagenet1k \
    --device cuda

# Evaluate on CIFAR-10
python -m clip_recipes.eval.zeroshot \
    --checkpoint runs/laion_lora_v3/final.pt \
    --dataset cifar10 \
    --device cuda
```

### Image-Text Retrieval

```bash
# Flickr30k Chinese
python -m clip_recipes.eval.retrieval \
    --checkpoint runs/zh_continue/final.pt \
    --dataset flickr30k-cn \
    --split test

# COCO Chinese
python -m clip_recipes.eval.retrieval \
    --checkpoint runs/zh_continue/final.pt \
    --dataset coco-cn \
    --metrics recall@1,recall@5,recall@10
```

### Programmatic Evaluation

```python
from clip_recipes.eval.zeroshot import ZeroShotEvaluator
from clip_recipes.models.builder import build_clip

model = build_clip(checkpoint_path="runs/final.pt")
evaluator = ZeroShotEvaluator(model, device="cuda")

results = evaluator.evaluate("imagenet1k")
print(f"Top-1 accuracy: {results['top1']:.2f}%")
print(f"Top-5 accuracy: {results['top5']:.2f}%")
```

## Sanity Checks and Debugging

### Temperature Drift Monitoring

```python
from clip_recipes.sanity import monitor_temperature

# Log temperature every N steps
if step % 100 == 0:
    temp = loss_fn.temperature.item()
    monitor_temperature(temp, step, threshold=0.15)
    # Warns if temperature drifts beyond [0.05, 0.15]
```

### Gradient Norm Tracking

```python
from clip_recipes.sanity import check_gradient_norms

# After loss.backward()
grad_norms = check_gradient_norms(model)
print(f"Vision encoder grad norm: {grad_norms['vision']:.4f}")
print(f"Text encoder grad norm: {grad_norms['text']:.4f}")

if grad_norms['vision'] > 10.0:
    print("WARNING: Vision gradients exploding!")
```

### Embedding Collapse Detection

```python
from clip_recipes.sanity import check_embedding_collapse

# Periodically check embedding diversity
if step % 1000 == 0:
    collapse_score = check_embedding_collapse(
        image_embeds,
        text_embeds,
        threshold=0.95
    )
    if collapse_score > 0.95:
        print("WARNING: Embeddings collapsing!")
```

## Common Patterns

### Custom Dataset Integration

```python
from clip_recipes.data.webdataset import create_webdataset_loader

dataloader = create_webdataset_loader(
    shard_pattern="s3://mybucket/train/{00000..01999}.tar",
    batch_size=256,
    num_workers=8,
    shuffle_buffer=10000,
    preprocessor=None  # Use default CLIP preprocessing
)

for batch in dataloader:
    images = batch["jpg"]  # PIL images or tensors
    texts = batch["txt"]   # strings
    # Train...
```

### Resuming from Checkpoint

```bash
# Automatically resumes if checkpoint exists
python -m clip_recipes.train \
    --config configs/laion_400m_lora.yaml \
    --output_dir runs/laion_lora_v3 \
    --resume_from_checkpoint runs/laion_lora_v3/checkpoint-5000.pt
```

### Learning Rate Scheduling

```yaml
train:
  lr: 5e-5
  warmup_steps: 2000
  lr_schedule: "cosine"  # or "linear", "constant"
  min_lr: 1e-6
  epochs: 10
```

```python
# In code
from clip_recipes.schedules import get_scheduler

scheduler = get_scheduler(
    optimizer,
    schedule_type="cosine",
    warmup_steps=2000,
    total_steps=100000,
    min_lr=1e-6
)
```

## Troubleshooting

### OOM with Large Models

```yaml
# Enable gradient checkpointing (TODO: currently limited)
model:
  gradient_checkpointing: true

# Or reduce batch size and accumulate gradients
train:
  batch_size: 128
  gradient_accumulation_steps: 4  # Effective batch = 512
```

### Loss Not Decreasing

1. **Check temperature**: Should be ~0.07 initially
   ```bash
   python -m clip_recipes.sanity.check_temperature --checkpoint runs/latest.pt
   ```

2. **Verify cross-GPU negatives** are working:
   ```python
   # In losses/contrastive.py, ensure gather_with_grad=True
   loss_fn = ContrastiveLoss(gather_with_grad=True)
   ```

3. **Check learning rate**:
   - LoRA: 3e-5 to 1e-4
   - Full fine-tuning: 1e-6 to 1e-5
   - Linear probe: 1e-3 to 1e-2

### Data Leakage

```bash
# Build hash set from eval data
python -m clip_recipes.data.dedup build_hashes \
    --input data/eval/*.tar \
    --output eval_hashes.txt

# Check training shards for leaks
python -m clip_recipes.data.dedup check_leakage \
    --train_shards data/train/*.tar \
    --eval_hashes eval_hashes.txt
```

### Shard Count Not Divisible by World Size

The loader drops trailing shards if count % world_size != 0. Ensure your shard count is divisible by GPU count:

```bash
# Good: 1000 shards, 4 GPUs (250 each)
# Bad: 1003 shards, 4 GPUs (last 3 dropped)

# Pad with empty shards if needed
python -m clip_recipes.data.pad_shards \
    --input data/train/*.tar \
    --target_count 1000
```

## Advanced: Custom Loss Functions

```python
from clip_recipes.losses.base import CLIPLoss

class CustomContrastiveLoss(CLIPLoss):
    def __init__(self, temperature=0.07, margin=0.2):
        super().__init__()
        self.temperature = nn.Parameter(torch.tensor(temperature))
        self.margin = margin
    
    def forward(self, image_embeds, text_embeds):
        # Normalize
        image_embeds = F.normalize(image_embeds, dim=-1)
        text_embeds = F.normalize(text_embeds, dim=-1)
        
        # Gather across GPUs
        if dist.is_initialized():
            image_embeds = self.all_gather(image_embeds)
            text_embeds = self.all_gather(text_embeds)
        
        # Your custom loss logic
        logits = image_embeds @ text_embeds.T / self.temperature
        # ...
        return loss

# Use in training
loss_fn = CustomContrastiveLoss()
```

## Environment Variables

```bash
# Distributed training
export MASTER_ADDR=localhost
export MASTER_PORT=29500
export WORLD_SIZE=4
export RANK=0

# Data loading
export WEBDATASET_CACHE_DIR=/tmp/wds_cache
export HF_DATASETS_CACHE=/path/to/cache

# Evaluation
export IMAGENET_PATH=/datasets/imagenet
export FLICKR30K_CN_PATH=/datasets/flickr30k-cn
```

## Project Structure

```
clip_recipes/
  train.py            # Main training entry point
  config.py           # Configuration schema
  data/
    webdataset.py     # Tar-shard dataloader
    dedup.py          # SHA-256 leakage prevention
    augment.py        # Image augmentations
    hard_negatives.py # FAISS-based mining
  losses/
    contrastive.py    # InfoNCE with cross-GPU gather
    sigclip.py        # SigLIP-style binary cross-entropy
  models/
    builder.py        # build_clip() factory
    lora.py           # LoRA implementation
  eval/
    zeroshot.py       # Zero-shot classification
    retrieval.py      # Image-text retrieval metrics
  sanity/             # Debugging utilities
  schedules.py        # LR schedulers
```
