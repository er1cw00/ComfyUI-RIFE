# ComfyUI-RIFE

English | [中文](README_zhgit .md) 

RIFE video frame interpolation node for ComfyUI, based on Practical-RIFE implementation.

## Features

- **Real-time Video Frame Interpolation**: Uses RIFE (Real-Time Intermediate Flow Estimation) algorithm to generate smooth intermediate frames between video frames
- **Automatic Model Download**: Automatically downloads required models from GitHub Releases on first use
- **Model Caching**: Automatically caches loaded models to avoid reloading
- **Multiple Precision Support**: Supports float32, float16, and bfloat16 precision for speed/quality trade-off
- **Torch Compile**: Optional PyTorch 2.0+ `torch.compile()` acceleration (slower first run, faster subsequent inference)
- **Batch Inference**: Supports batch processing of interpolation tasks for higher throughput

## Supported Models

| Model Name | Version | Description |
|-----------|---------|-------------|
| `rife47.pth` | 4.7 | RIFE version 4.7 |
| `rife49.pth` | 4.7 | RIFE version 4.9 |
| `rife417.pth` | 4.17 | RIFE version 4.17 |
| `rife426.pth` | 4.26 | RIFE version 4.26 (ensemble not supported) |

Models are automatically downloaded to `ComfyUI/models/rife/` directory.

## Installation

1. Clone this repository into ComfyUI's `custom_nodes` directory:
   ```bash
   cd ComfyUI/custom_nodes
   git clone git@github.com:er1cw00/ComfyUI-RIFE.git ComfyUI-RIFE
   ```

2. Install dependencies:
   ```bash
   cd ComfyUI-RIFE
   pip install -r requirements.txt
   ```

## Node Parameters

### RIFE_VFI

**Required Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ckpt_name` | Dropdown | - | Select RIFE model (auto-downloads if not present) |
| `frames` | IMAGE | - | Input video frame sequence |
| `clear_cache_after_n_frames` | INT | 10 | Clear GPU cache every N frames (1-1000) |
| `multiplier` | INT | 2 | Interpolation multiplier, e.g., 2 generates 1 intermediate frame between each pair |
| `fast_mode` | BOOLEAN | True | Fast mode, trade some quality for speed |
| `ensemble` | BOOLEAN | True | Use ensemble mode for better quality (forced off for v4.26) |
| `scale_factor` | Dropdown | 1.0 | Optical flow scale factor [0.25, 0.5, 1.0, 2.0, 4.0] |
| `dtype` | Dropdown | float32 | Inference precision: float32 (safest), float16 (fast/less VRAM), bfloat16 (requires RTX 30xx+) |
| `torch_compile` | BOOLEAN | False | Use torch.compile() for acceleration (requires PyTorch 2.0+) |
| `batch_size` | INT | 1 | Number of interpolation tasks per batch (higher = more throughput but more VRAM, 1-64) |

**Optional Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `optional_interpolation_states` | INTERPOLATION_STATES | Interpolation state list for controlling frame skipping |

**Outputs:**

| Output | Type | Description |
|--------|------|-------------|
| `IMAGE` | IMAGE | Interpolated video frame sequence |

## Usage Example

### Basic Workflow

1. Load video frames using `Load Video` or `Load Image Batch` node
2. Connect to `RIFE_VFI` node
3. Select a model from the dropdown (e.g., `rife426.pth`)
4. Set `multiplier` (e.g., 2 means 30fps → 60fps)
5. Output using `Save Video` or `Preview Image` node

### Parameter Tuning Suggestions

- **Quality Priority**: `dtype=float32`, `ensemble=True`, `fast_mode=False`
- **Speed Priority**: `dtype=float16`, `fast_mode=True`, `torch_compile=True` (PyTorch 2.0+)
- **Low VRAM**: Reduce `batch_size` to 1, use `dtype=float16`
- **RTX 30xx/40xx**: Can use `dtype=bfloat16`, more stable than float16 with similar speed

## File Structure

```
ComfyUI-RIFE/
├── __init__.py          # Node registration entry
├── rife_node.py         # Main node implementation (model loading, inference)
├── rife_model.py        # RIFE IFNet model definition
├── requirements.txt     # Python dependencies
└── README.md           # This file (Chinese)
└── README_EN.md        # English version
```

## Model Download Notes

Model files are automatically downloaded to `ComfyUI/models/rife/`:

- On first use of a model, the node automatically downloads from `https://github.com/Fannovel16/ComfyUI-Frame-Interpolation/releases/download/models/`
- If download fails, check network connection or manually download models to the above directory

## Important Notes

1. **RIFE 4.26**: This version does not support `ensemble` mode, it will be automatically disabled
2. **torch.compile()**: First inference will be slow (compilation phase), subsequent runs are 10-30% faster
3. **bfloat16**: Only supported on Ampere architecture (RTX 30xx/40xx) or newer GPUs
4. **Model Caching**: Changing `ckpt_name`, `dtype`, or `torch_compile` triggers model reload

## Dependencies

- torch >= 2.0.0 (recommended)
- einops
- packaging
- numpy

## License

This project is based on Practical-RIFE implementation, following the corresponding open-source license.
