# InSpatio-World 1.5

[![HuggingFace](https://img.shields.io/badge/HuggingFace-Model-yellow?logo=huggingface)](https://huggingface.co/inspatio/world-1.5)
[![Project Page](https://img.shields.io/badge/Project-Page-green)](https://inspatio.github.io/inspatio-world-1.5/)
[![Repository](https://img.shields.io/badge/GitHub-HamidYaraliOfficial%2FInSpatio-black?logo=github)](https://github.com/HamidYaraliOfficial/InSpatio)
[![License](https://img.shields.io/badge/License-Apache--2.0-orange)](https://github.com/HamidYaraliOfficial/InSpatio/blob/main/LICENSE)
[![arXiv](https://img.shields.io/badge/arXiv-2604.07209-b31b1b)](https://arxiv.org/abs/2604.07209)
[![Live Demo](https://img.shields.io/badge/Live-Demo-blue?logo=googlechrome&logoColor=white)](https://world.inspatio.com/)
[![ModelScope](https://img.shields.io/badge/ModelScope-Model-purple)](https://modelscope.cn/)

> **Three-language README:** English · فارسی · 中文

---

# English

## Overview

**InSpatio-World 1.5** is a real-time 4D world simulation and novel-view generation system designed to move beyond the original camera frame and enable spatial exploration of visual scenes.

The project supports a broad range of source inputs, including:

- A single image
- A set of multiple images
- A panorama
- A video

InSpatio-World 1.5 is designed for large viewpoint changes while preserving scene structure, visual consistency, and temporal coherence. Its applications include cinematic camera control, immersive experiences, spatial intelligence research, embodied AI, dynamic environment modeling, and next-view prediction.

For the official project description, demos, and research material, see the [Project Page](https://inspatio.github.io/inspatio-world-1.5/).

## Key Capabilities

### Real-Time Scene Roaming

Starting from a single image, the system can explore the scene from different viewpoints and reveal areas that were not visible in the original camera view.

### Spatial and Visual Consistency

The model aims to maintain believable scene structure, subject identity, appearance, and visual style during substantial camera motion.

### Next-View Prediction

With video input, InSpatio-World 1.5 supports spatiotemporal exploration and prediction of novel viewpoints as the scene evolves over time.

### Multiple Input Types

The 1.5 release extends the interaction model to single-image, multi-image, panorama, and video inputs.

### Bullet-Time Creation

Multiple synchronized images can be used to construct a camera path around a subject and produce a coherent bullet-time style sequence.

## Examples

Official demonstrations include single-image scene roaming, next-view prediction from dynamic videos, multi-image reconstruction/exploration, and panorama-based scene exploration.

For interactive demos and example videos:

- [Official Project Page](https://inspatio.github.io/inspatio-world-1.5/)
- [Live Demo](https://world.inspatio.com/)

## Requirements

The released implementation documents the following environment:

- **Python 3.10**
- **CUDA 12.1**
- **FlashAttention-3** is optional and is intended for Hopper-class GPUs such as H100/H800 with `nvcc >= 12.3`.

> CUDA and PyTorch compatibility should be matched to the GPU and the environment configuration used for the implementation.

### 1. Create the Conda environment

```bash
conda env create -f environment.yml
conda activate inspatio_world
```

### 2. Install FlashAttention

```bash
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.5cxx11abiFALSE-cp310-cp310-linux_x86_64.whl
```

### 3. Optional: Install FlashAttention-3

For Hopper/H100-class NVIDIA GPUs:

```bash
git clone https://github.com/Dao-AILab/flash-attention.git
cd flash-attention/hopper
python setup.py install
```

The inference implementation can automatically use FlashAttention-3 when `flash_attn_interface` is available on a supported GPU; otherwise it falls back to FlashAttention-2.

## Model Weights

Download the required checkpoints into the `checkpoints/` directory.

| Model | Purpose | Source |
|---|---|---|
| **InSpatio-World-1.3B / 1.5** | v2v inference | [Hugging Face](https://huggingface.co/inspatio/world-1.5) |
| **Wan2.1-T2V-1.3B** | Text encoder, VAE, and base model components | [Hugging Face](https://huggingface.co/Wan-AI/Wan2.1-T2V-1.3B) |
| **DA3 (Depth-Anything-3)** | Depth estimation | [Hugging Face](https://huggingface.co/depth-anything/DA3NESTED-GIANT-LARGE) |
| **Florence-2-large** | Video caption generation | [Hugging Face](https://huggingface.co/microsoft/Florence-2-large) |
| **TAEHV** | Optional inference acceleration | [GitHub](https://github.com/madebyollin/taehv) |

Download/checkpoint preparation:

```bash
bash scripts/download.sh
```

Expected directory structure:

```text
checkpoints/
├── InSpatio-World-1.3B/
│   └── InSpatio-World-1.3B.safetensors
├── Wan2.1-T2V-1.3B/
├── DA3/
├── Florence-2-large/
└── taehv/
```

## Inference

The released video inference pipeline is organized into three stages:

1. **Step 1** — Generate video captions with Florence-2.
2. **Step 2** — Estimate depth with Depth-Anything-3, convert the results to the inference format, and render the scene representation.
3. **Step 3** — Run InSpatio-World v2v inference.

The complete pipeline can be executed with:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt
```

### Quick Start

```bash
# 1. Place your .mp4 video(s) in a folder
mkdir -p my_videos
cp your_video.mp4 my_videos/

# 2. Run the full pipeline
bash run_test_pipeline.sh \
  --input_dir ./my_videos \
  --traj_txt_path ./traj/x_y_circle_cycle.txt

# 3. Results are written to:
# ./output/my_videos/x_y_circle_cycle/
```

## Camera Trajectory Control

The `--traj_txt_path` argument controls the camera trajectory used for novel-view synthesis.

Predefined trajectories include:

| File | Motion |
|---|---|
| `x_y_circle_cycle.txt` | Cyclic combined pitch + yaw orbit |
| `zoom_out_in.txt` | Dolly zoom out + dolly zoom in |

### Trajectory File Format

A trajectory file contains three lines of space-separated keyframe values:

```text
<line 1>  pitch (degrees): positive = orbit up, negative = orbit down
<line 2>  yaw (degrees):   positive = orbit left, negative = orbit right
<line 3>  displacement:    relative camera displacement scale
```

The values are interpolated to match the output frame count.

The displacement value is a relative scale derived from the estimated foreground depth:

- When pitch/yaw are non-zero, displacement controls the orbit radius.
- When pitch and yaw are both zero, displacement acts as a dolly-zoom parameter:
  - Positive = move forward / zoom in
  - Negative = move backward / zoom out

## Command-Line Arguments

| Argument | Required | Default | Description |
|---|---|---|---|
| `--input_dir` | Yes | — | Folder containing `.mp4` input files |
| `--traj_txt_path` | Yes | — | Camera trajectory file |
| `--checkpoint_path` | No | `./checkpoints/InSpatio-World/InSpatio-World.safetensors` | InSpatio-World checkpoint |
| `--config_path` | No | `configs/inference.yaml` | Inference configuration |
| `--da3_model_path` | No | `./checkpoints/DA3` | DA3 model path |
| `--florence_model_path` | No | `./checkpoints/Florence-2-large` | Florence-2 model path |
| `--step1_gpus` | No | `0` | GPU IDs for caption generation |
| `--step2_gpus` | No | `0` | GPU IDs for depth estimation |
| `--step3_gpus` | No | `0` | GPU ID(s) for v2v inference |
| `--step3_nproc` | No | `1` | Number of GPUs for Step 3 |
| `--output_folder` | No | `./output/<name>/<traj>` | Custom output directory |
| `--master_port` | No | `29513` | Torch distributed master port |
| `--skip_step1` | No | `false` | Skip caption generation |
| `--skip_step2` | No | `false` | Skip depth estimation |
| `--skip_step3` | No | `false` | Skip v2v inference |
| `--relative_to_source` | No | `false` | Compose trajectory relative to the initial view |
| `--rotation_only` | No | `false` | Apply rotation only; ignore translation |
| `--render_backend` | No | `warper` | Rendering backend (`warper` or `ply`) |
| `--disable_adaptive_frame` | No | `false` | Disable adaptive frame expansion/subsampling |
| `--freeze_repeat` | No | `0` | Number of repeated frames for a temporal freeze |
| `--freeze_frame` | No | Middle frame | Frame index to freeze |
| `--use_tae` | No | `false` | Use TAE instead of WanVAE |
| `--tae_checkpoint_path` | No | `./checkpoints/taehv/taew2_1.pth` | TAE checkpoint path |
| `--compile_dit` | No | `false` | Apply `torch.compile` to the DiT model |

## Skip Completed Stages

If the outputs of Step 1 or Step 2 already exist, they can be skipped:

```bash
bash run_test_pipeline.sh \
  --input_dir ./my_videos \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --skip_step1 \
  --skip_step2
```

## Temporal Control

A temporal freeze can be created by repeating a selected frame:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --freeze_repeat 150 \
  --output_folder ./output/example_freeze_repeat_150 \
  --disable_adaptive_frame
```

Use:

- `--freeze_frame` to select the frame to freeze.
- `--freeze_repeat` to control the number of repeated frames.

## Autonomous-Driving-Oriented Usage

For camera-relative transformations:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example3 \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --relative_to_source \
  --rotation_only \
  --disable_adaptive_frame
```

## Speed-Up

TAE and `torch.compile` can be used to accelerate inference:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --use_tae \
  --disable_adaptive_frame
```

The released documentation also describes `--compile_dit` as an additional optimization for high-throughput deployment scenarios.

## Repository

This GitHub repository is maintained at:

**https://github.com/HamidYaraliOfficial/InSpatio**

The project documentation and model resources are linked above so that this repository can be used as a reference point for research, experimentation, reproducibility, and further development.

## License

This project is distributed under the [Apache-2.0 License](https://github.com/HamidYaraliOfficial/InSpatio/blob/main/LICENSE).

The Apache-2.0 notice applies to the project code under the applicable license terms. Dependencies and external components such as Depth-Anything-3, Florence-2, TAEHV, Wan2.1, and related projects are separately licensed by their respective authors and organizations.

## Citation

If you use InSpatio-World in academic or research work, cite the original project:

```bibtex
@misc{inspatio-world,
    title={INSPATIO-WORLD: A Real-Time 4D World Simulator via Spatiotemporal Autoregressive Modeling},
    author={InSpatio Team},
    journal={arXiv preprint arXiv:2604.07209},
    year={2026}
}
```

## Acknowledgements

InSpatio-World builds upon and acknowledges the contributions of:

- **Wan2.1**
- **Self-Forcing**
- **Depth-Anything-3**
- **Florence-2**
- **TAEHV / TAEV**
- **ReCamMaster**

Please refer to the respective upstream repositories and licenses when redistributing or extending the software.

---

# فارسی

## معرفی

**InSpatio-World 1.5** یک سامانهٔ شبیه‌سازی بلادرنگ جهان چهار‌بُعدی (4D) و تولید نمای جدید (Novel-View Generation) است که با عبور از قاب اولیهٔ دوربین، امکان کاوش فضایی در صحنه‌های تصویری را فراهم می‌کند.

این پروژه از انواع ورودی‌های زیر پشتیبانی می‌کند:

- یک تصویر
- مجموعه‌ای از چند تصویر
- تصویر پانوراما
- ویدئو

نسخهٔ 1.5 برای تغییرات قابل‌توجه در زاویهٔ دید طراحی شده است و هدف آن حفظ ساختار صحنه، سازگاری بصری و پیوستگی زمانی است. از کاربردهای آن می‌توان به کنترل دوربین سینمایی، تجربه‌های فراگیر، پژوهش در هوش فضایی، هوش مصنوعی تجسم‌یافته، مدل‌سازی محیط‌های پویا و پیش‌بینی نمای بعدی اشاره کرد.

برای توضیحات رسمی، ویدئوهای نمایشی و منابع پژوهشی به [صفحهٔ رسمی پروژه](https://inspatio.github.io/inspatio-world-1.5/) مراجعه کنید.

## قابلیت‌های اصلی

### کاوش بلادرنگ صحنه

با شروع از یک تصویر، می‌توان صحنه را از دیدگاه‌های مختلف بررسی کرد و بخش‌هایی را که در نمای اولیه دیده نمی‌شدند مشاهده کرد.

### سازگاری فضایی و بصری

مدل تلاش می‌کند ساختار صحنه، هویت سوژه، ظاهر و سبک بصری را در هنگام تغییرات قابل‌توجه دوربین حفظ کند.

### پیش‌بینی نمای بعدی

در حالت ورودی ویدئویی، InSpatio-World 1.5 امکان کاوش فضایی-زمانی و پیش‌بینی دیدگاه‌های جدید را هم‌زمان با تحول صحنه فراهم می‌کند.

### پشتیبانی از چند نوع ورودی

نسخهٔ 1.5 مدل تعامل را به تصویر تکی، چندتصویر، پانوراما و ویدئو گسترش می‌دهد.

### تولید Bullet-Time

با چند تصویر همگام می‌توان مسیر دوربین را پیرامون سوژه ایجاد کرد و یک توالی منسجم با سبک Bullet-Time تولید نمود.

## نمونه‌ها و دموی پروژه

دموهای رسمی شامل کاوش صحنه از تصویر تکی، پیش‌بینی نمای بعدی از ویدئوهای پویا، کاوش بر مبنای چند تصویر و کاوش صحنه‌های پانوراما هستند.

- [صفحهٔ رسمی پروژه](https://inspatio.github.io/inspatio-world-1.5/)
- [دموی آنلاین](https://world.inspatio.com/)

## نیازمندی‌ها

نسخهٔ منتشرشدهٔ پیاده‌سازی، محیط زیر را مستند کرده است:

- **Python 3.10**
- **CUDA 12.1**
- نصب **FlashAttention-3** اختیاری است و برای GPUهای مبتنی بر Hopper مانند H100/H800 با `nvcc >= 12.3` در نظر گرفته شده است.

> نسخهٔ CUDA و PyTorch باید با GPU و فایل پیکربندی محیط شما سازگار باشد.

### ۱. ایجاد محیط Conda

```bash
conda env create -f environment.yml
conda activate inspatio_world
```

### ۲. نصب FlashAttention

```bash
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.5cxx11abiFALSE-cp310-cp310-linux_x86_64.whl
```

### ۳. نصب اختیاری FlashAttention-3

برای GPUهای Hopper/H100:

```bash
git clone https://github.com/Dao-AILab/flash-attention.git
cd flash-attention/hopper
python setup.py install
```

در صورت فراهم بودن `flash_attn_interface` روی GPU پشتیبانی‌شده، پیاده‌سازی استنتاج می‌تواند به‌صورت خودکار از FlashAttention-3 استفاده کند و در غیر این صورت به FlashAttention-2 برگردد.

## وزن‌های مدل

Checkpointهای مورد نیاز را در پوشهٔ `checkpoints/` قرار دهید.

| مدل | کاربرد | منبع |
|---|---|---|
| **InSpatio-World-1.3B / 1.5** | استنتاج v2v | [Hugging Face](https://huggingface.co/inspatio/world-1.5) |
| **Wan2.1-T2V-1.3B** | Text Encoder، VAE و اجزای مدل پایه | [Hugging Face](https://huggingface.co/Wan-AI/Wan2.1-T2V-1.3B) |
| **DA3 (Depth-Anything-3)** | تخمین عمق | [Hugging Face](https://huggingface.co/depth-anything/DA3NESTED-GIANT-LARGE) |
| **Florence-2-large** | تولید کپشن ویدئو | [Hugging Face](https://huggingface.co/microsoft/Florence-2-large) |
| **TAEHV** | شتاب‌دهی اختیاری استنتاج | [GitHub](https://github.com/madebyollin/taehv) |

```bash
bash scripts/download.sh
```

ساختار مورد انتظار:

```text
checkpoints/
├── InSpatio-World-1.3B/
│   └── InSpatio-World-1.3B.safetensors
├── Wan2.1-T2V-1.3B/
├── DA3/
├── Florence-2-large/
└── taehv/
```

## استنتاج

pipeline ویدئویی منتشرشده در سه مرحله اجرا می‌شود:

1. **مرحلهٔ ۱** — تولید کپشن ویدئو با Florence-2
2. **مرحلهٔ ۲** — تخمین عمق با Depth-Anything-3، تبدیل خروجی و آماده‌سازی نمایش صحنه
3. **مرحلهٔ ۳** — اجرای استنتاج v2v با InSpatio-World

اجرای کل pipeline:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt
```

### شروع سریع

```bash
mkdir -p my_videos
cp your_video.mp4 my_videos/

bash run_test_pipeline.sh \
  --input_dir ./my_videos \
  --traj_txt_path ./traj/x_y_circle_cycle.txt
```

خروجی در مسیر زیر ذخیره می‌شود:

```text
./output/my_videos/x_y_circle_cycle/
```

## کنترل مسیر دوربین

پارامتر `--traj_txt_path` مسیر حرکت دوربین را برای Novel-View Synthesis کنترل می‌کند.

| فایل | نوع حرکت |
|---|---|
| `x_y_circle_cycle.txt` | حرکت مداری ترکیبی Pitch + Yaw |
| `zoom_out_in.txt` | زوم/حرکت دوربین به بیرون و سپس به داخل |

### قالب فایل Trajectory

فایل trajectory شامل سه خط با مقادیر keyframe جداشده با فاصله است:

```text
<خط ۱>  pitch بر حسب درجه
<خط ۲>  yaw بر حسب درجه
<خط ۳>  displacement و مقیاس نسبی جابه‌جایی دوربین
```

مقادیر متناسب با تعداد فریم خروجی درون‌یابی می‌شوند.

## پارامترهای مهم خط فرمان

پارامترهای کامل شامل مواردی مانند `--input_dir`، `--traj_txt_path`، مسیر checkpointها، تخصیص GPU برای مراحل مختلف، رد کردن مراحل تکمیل‌شده، کنترل چرخش، backend رندر، فریز زمانی، TAE و `torch.compile` هستند.

## کنترل زمانی

نمونهٔ ایجاد توقف زمانی با تکرار یک فریم:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --freeze_repeat 150 \
  --output_folder ./output/example_freeze_repeat_150 \
  --disable_adaptive_frame
```

- `--freeze_frame` فریم مورد نظر برای توقف را تعیین می‌کند.
- `--freeze_repeat` تعداد فریم‌های تکرارشده را تعیین می‌کند.

## استفاده در سناریوهای مرتبط با رانندگی خودکار

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example3 \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --relative_to_source \
  --rotation_only \
  --disable_adaptive_frame
```

## شتاب‌دهی

برای افزایش سرعت می‌توان از TAE و `torch.compile` استفاده کرد:

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --use_tae \
  --disable_adaptive_frame
```

## مخزن GitHub

مخزن این نسخه در:

**https://github.com/HamidYaraliOfficial/InSpatio**

قرار دارد و می‌تواند برای پژوهش، آزمایش، بازتولید نتایج و توسعه‌های بعدی مورد استفاده قرار گیرد.

## مجوز

این پروژه تحت [Apache-2.0 License](https://github.com/HamidYaraliOfficial/InSpatio/blob/main/LICENSE) منتشر شده است.

مجوز Apache-2.0 مربوط به کد پروژه است؛ وابستگی‌ها و اجزای خارجی مانند Depth-Anything-3، Florence-2، TAEHV، Wan2.1 و پروژه‌های مرتبط دارای مجوزهای جداگانه هستند.

## ارجاع علمی

برای استفاده در مقاله، پایان‌نامه یا پژوهش علمی، به پروژهٔ اصلی استناد کنید:

```bibtex
@misc{inspatio-world,
    title={INSPATIO-WORLD: A Real-Time 4D World Simulator via Spatiotemporal Autoregressive Modeling},
    author={InSpatio Team},
    journal={arXiv preprint arXiv:2604.07209},
    year={2026}
}
```

## سپاسگزاری

این پروژه بر پایه و با استفاده از دستاوردهای پروژه‌های زیر توسعه یافته است:

- **Wan2.1**
- **Self-Forcing**
- **Depth-Anything-3**
- **Florence-2**
- **TAEHV / TAEV**
- **ReCamMaster**

برای استفاده، بازتوزیع یا توسعهٔ هر یک از اجزای فوق، مجوز و شرایط مخزن اصلی آن پروژه را نیز بررسی کنید.

---

# 中文

## 项目简介

**InSpatio-World 1.5** 是一个实时 4D 世界模拟与新视角生成系统，旨在突破原始摄像机画面的限制，让用户能够从不同视角探索视觉场景。

该项目支持多种输入形式：

- 单张图像
- 多张图像
- 全景图像
- 视频

InSpatio-World 1.5 面向较大幅度的视角变化，同时尽可能保持场景结构、视觉一致性与时间连贯性。其应用方向包括电影级摄像机控制、沉浸式体验、空间智能研究、具身智能、动态环境建模以及下一视角预测。

官方项目介绍、演示视频和研究资料请访问[官方项目页面](https://inspatio.github.io/inspatio-world-1.5/)。

## 核心能力

### 实时场景漫游

从单张图像开始，从不同视角探索场景，并生成原始摄像机视野中不可见的区域。

### 空间与视觉一致性

在较大的摄像机运动过程中，模型旨在保持场景结构、主体身份、外观以及视觉风格的一致性。

### 下一视角预测

对于视频输入，InSpatio-World 1.5 支持时空探索，并能够随着场景变化预测新的观察视角。

### 多种输入类型

1.5 版本支持单图、多图、全景图以及视频输入。

### Bullet-Time 生成

利用多张同步图像，可以围绕目标设计摄像机运动轨迹，从而生成连贯的 Bullet-Time 风格视频序列。

## 官方演示

官方展示包括：

- 单图场景漫游
- 动态视频下一视角预测
- 多图输入场景探索
- 全景场景探索

相关资源：

- [官方项目页面](https://inspatio.github.io/inspatio-world-1.5/)
- [在线 Demo](https://world.inspatio.com/)

## 环境要求

当前公开实现所记录的环境要求为：

- **Python 3.10**
- **CUDA 12.1**
- **FlashAttention-3** 为可选组件，主要面向 H100/H800 等 Hopper GPU，并要求 `nvcc >= 12.3`。

> CUDA、PyTorch 与 GPU 驱动版本应根据实际硬件和环境进行匹配。

### 1. 创建 Conda 环境

```bash
conda env create -f environment.yml
conda activate inspatio_world
```

### 2. 安装 FlashAttention

```bash
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.5cxx11abiFALSE-cp310-cp310-linux_x86_64.whl
```

### 3. 可选：安装 FlashAttention-3

对于 Hopper/H100 GPU：

```bash
git clone https://github.com/Dao-AILab/flash-attention.git
cd flash-attention/hopper
python setup.py install
```

当 `flash_attn_interface` 可用并且当前 GPU 受支持时，推理代码可以自动使用 FlashAttention-3；否则回退到 FlashAttention-2。

## 模型权重

请将所需 checkpoint 放入 `checkpoints/` 目录。

| 模型 | 用途 | 来源 |
|---|---|---|
| **InSpatio-World-1.3B / 1.5** | v2v 推理 | [Hugging Face](https://huggingface.co/inspatio/world-1.5) |
| **Wan2.1-T2V-1.3B** | 文本编码器、VAE 与基础模型组件 | [Hugging Face](https://huggingface.co/Wan-AI/Wan2.1-T2V-1.3B) |
| **DA3 (Depth-Anything-3)** | 深度估计 | [Hugging Face](https://huggingface.co/depth-anything/DA3NESTED-GIANT-LARGE) |
| **Florence-2-large** | 视频字幕/描述生成 | [Hugging Face](https://huggingface.co/microsoft/Florence-2-large) |
| **TAEHV** | 可选推理加速 | [GitHub](https://github.com/madebyollin/taehv) |

执行：

```bash
bash scripts/download.sh
```

下载后的目录结构示例：

```text
checkpoints/
├── InSpatio-World-1.3B/
│   └── InSpatio-World-1.3B.safetensors
├── Wan2.1-T2V-1.3B/
├── DA3/
├── Florence-2-large/
└── taehv/
```

## 推理流程

公开的视频推理 pipeline 分为三个阶段：

1. **步骤 1** — 使用 Florence-2 生成视频描述。
2. **步骤 2** — 使用 Depth-Anything-3 进行深度估计，并转换推理格式与生成场景表示。
3. **步骤 3** — 执行 InSpatio-World v2v 推理。

完整执行命令：

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt
```

### 快速开始

```bash
mkdir -p my_videos
cp your_video.mp4 my_videos/

bash run_test_pipeline.sh \
  --input_dir ./my_videos \
  --traj_txt_path ./traj/x_y_circle_cycle.txt
```

输出默认保存于：

```text
./output/my_videos/x_y_circle_cycle/
```

## 摄像机轨迹控制

`--traj_txt_path` 参数用于控制新视角生成过程中的摄像机轨迹。

预定义轨迹：

| 文件 | 运动方式 |
|---|---|
| `x_y_circle_cycle.txt` | Pitch + Yaw 周期环绕 |
| `zoom_out_in.txt` | 先拉远再推进的 Dolly Zoom |

### Trajectory 文件格式

轨迹文件包含三行以空格分隔的关键帧数据：

```text
<第1行>  pitch（角度）
<第2行>  yaw（角度）
<第3行>  displacement（相对位移尺度）
```

系统会根据输出视频帧数自动进行插值。

## 主要命令行参数

支持的主要参数包括：

`--input_dir`、`--traj_txt_path`、checkpoint 与配置文件路径、不同阶段的 GPU 分配、跳过已完成步骤、仅旋转模式、渲染 backend、时间冻结、TAE 以及 `torch.compile` 等。

## 时间控制

通过重复指定帧可以生成时间冻结效果：

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --freeze_repeat 150 \
  --output_folder ./output/example_freeze_repeat_150 \
  --disable_adaptive_frame
```

- `--freeze_frame`：指定冻结的帧。
- `--freeze_repeat`：指定重复帧的数量。

## 自动驾驶相关场景

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example3 \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --relative_to_source \
  --rotation_only \
  --disable_adaptive_frame
```

## 推理加速

可以使用 TAE 和 `torch.compile` 进一步提升推理速度：

```bash
bash run_test_pipeline.sh \
  --input_dir ./test/example \
  --traj_txt_path ./traj/x_y_circle_cycle.txt \
  --use_tae \
  --disable_adaptive_frame
```

## GitHub 仓库

本项目的 GitHub 仓库：

**https://github.com/HamidYaraliOfficial/InSpatio**

该仓库可用于研究、实验、结果复现、工程测试以及后续扩展开发。

## 许可证

本项目使用 [Apache-2.0 License](https://github.com/HamidYaraliOfficial/InSpatio/blob/main/LICENSE)。

Apache-2.0 许可适用于项目代码；Depth-Anything-3、Florence-2、TAEHV、Wan2.1 以及其他外部组件分别遵循各自作者或组织发布的许可证。

## 学术引用

如果在论文、学位论文或科研项目中使用 InSpatio-World，请引用原始项目：

```bibtex
@misc{inspatio-world,
    title={INSPATIO-WORLD: A Real-Time 4D World Simulator via Spatiotemporal Autoregressive Modeling},
    author={InSpatio Team},
    journal={arXiv preprint arXiv:2604.07209},
    year={2026}
}
```

## 致谢

本项目使用并致谢以下开源工作：

- **Wan2.1**
- **Self-Forcing**
- **Depth-Anything-3**
- **Florence-2**
- **TAEHV / TAEV**
- **ReCamMaster**

如需重新分发、修改或扩展上述组件，请同时遵循各自官方仓库的许可证及使用条款。

---

## Project Links / 项目链接 / پروژه

| Resource | Link |
|---|---|
| GitHub | https://github.com/HamidYaraliOfficial/InSpatio |
| Official Project Page | https://inspatio.github.io/inspatio-world-1.5/ |
| Hugging Face | https://huggingface.co/inspatio/world-1.5 |
| arXiv | https://arxiv.org/abs/2604.07209 |
| Live Demo | https://world.inspatio.com/ |
