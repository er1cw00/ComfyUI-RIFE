# ComfyUI-RIFE

[English](README.md) | 中文

用于 ComfyUI 的 RIFE 视频插帧节点，基于 Practical-RIFE 实现。

## 功能特性

- **实时视频插帧**: 使用 RIFE（Real-Time Intermediate Flow Estimation）算法在视频帧之间生成平滑的中间帧
- **自动模型下载**: 首次使用时会自动从 GitHub Releases 下载所需模型
- **模型缓存**: 自动缓存已加载的模型，避免重复加载
- **多种精度支持**: 支持 float32、float16、bfloat16 精度，可在速度和精度间权衡
- **Torch Compile**: 可选 PyTorch 2.0+ 的 `torch.compile()` 加速（首次编译较慢，后续推理更快）
- **批量推理**: 支持批量处理插帧任务，提高吞吐量

## 支持的模型

| 模型名称 | 版本 | 说明 |
|---------|------|------|
| `rife47.pth` | 4.7 | RIFE 4.7 版本 |
| `rife49.pth` | 4.7 | RIFE 4.9 版本 |
| `rife417.pth` | 4.17 | RIFE 4.17 版本 |
| `rife426.pth` | 4.26 | RIFE 4.26 版本（不支持 ensemble）|

模型会自动下载到 `ComfyUI/models/rife/` 目录。

## 安装

1. 将本节点文件夹放入 ComfyUI 的 `custom_nodes` 目录:
   ```bash
   cd ComfyUI/custom_nodes
   git clone git@github.com:er1cw00/ComfyUI-RIFE.git ComfyUI-RIFE
   ```

2. 安装依赖:
   ```bash
   cd ComfyUI-RIFE
   pip install -r requirements.txt
   ```

## 节点参数

### RIFE_VFI

**必需参数:**

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `ckpt_name` | 下拉选择 | - | 选择 RIFE 模型（自动下载） |
| `frames` | IMAGE | - | 输入视频帧序列 |
| `clear_cache_after_n_frames` | INT | 10 | 每处理 N 帧后清理显存缓存（1-1000） |
| `multiplier` | INT | 2 | 插帧倍数，如 2 表示在每帧之间生成 1 个中间帧 |
| `fast_mode` | BOOLEAN | True | 快速模式，牺牲少量质量换取速度 |
| `ensemble` | BOOLEAN | True | 使用 ensemble 模式提高质量（4.26 版本强制关闭） |
| `scale_factor` | 下拉选择 | 1.0 | 光流计算缩放比例 [0.25, 0.5, 1.0, 2.0, 4.0] |
| `dtype` | 下拉选择 | float32 | 推理精度：float32（最安全）、float16（快/省显存）、bfloat16（需 RTX 30xx+） |
| `torch_compile` | BOOLEAN | False | 使用 torch.compile() 加速（需 PyTorch 2.0+） |
| `batch_size` | INT | 1 | 每批次处理的插帧任务数（提高吞吐量但增加显存占用，1-64） |

**可选参数:**

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `optional_interpolation_states` | INTERPOLATION_STATES | 插帧状态列表，用于控制跳过特定帧 |

**输出:**

| 输出名 | 类型 | 说明 |
|--------|------|------|
| `IMAGE` | IMAGE | 插帧后的视频帧序列 |

## 使用示例

### 基本流程

1. 使用 `Load Video` 或 `Load Image Batch` 节点加载视频帧
2. 连接到 `RIFE_VFI` 节点
3. 从下拉菜单选择模型（如 `rife426.pth`）
4. 设置 `multiplier`（如 2 表示 30fps → 60fps）
5. 使用 `Save Video` 或 `Preview Image` 节点输出结果

### 参数调优建议

- **追求质量**: `dtype=float32`, `ensemble=True`, `fast_mode=False`
- **追求速度**: `dtype=float16`, `fast_mode=True`, `torch_compile=True`（PyTorch 2.0+）
- **显存不足**: 降低 `batch_size` 至 1，使用 `dtype=float16`
- **RTX 30xx/40xx**: 可尝试 `dtype=bfloat16`，比 float16 更稳定且速度相近

## 文件结构

```
ComfyUI-RIFE/
├── __init__.py          # 节点注册入口
├── rife_node.py         # 节点主要实现（模型加载、推理逻辑）
├── rife_model.py        # RIFE IFNet 模型定义
├── requirements.txt     # Python 依赖
└── README.md           # 本文件
```

## 模型下载说明

模型文件会自动下载到 `ComfyUI/models/rife/` 目录：

- 首次使用某个模型时，节点会自动从 `https://github.com/Fannovel16/ComfyUI-Frame-Interpolation/releases/download/models/` 下载
- 如果下载失败，请检查网络连接或手动下载模型放到上述目录

## 注意事项

1. **RIFE 4.26 版本**: 该版本不支持 `ensemble` 模式，会自动强制关闭
2. **torch.compile()**: 首次推理会很慢（编译阶段），后续推理会快 10-30%
3. **bfloat16**: 仅在 Ampere 架构（RTX 30xx/40xx）或更新的 GPU 上支持
4. **模型缓存**: 修改 `ckpt_name`、`dtype` 或 `torch_compile` 会触发重新加载模型

## 依赖

- torch >= 2.0.0（推荐）
- einops
- packaging
- numpy

## 许可证

本项目基于 Practical-RIFE 实现，遵循相应开源协议。
