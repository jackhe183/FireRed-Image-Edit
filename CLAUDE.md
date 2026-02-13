# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

FireRed-Image-Edit 是一个通用图像编辑模型，使用 Diffusers 框架中的 `QwenImageEditPlusPipeline` 进行推理。项目基于 Qwen-Image 架构构建。

## 常用命令

### 基础推理
```bash
python inference.py \
    --input_image ./examples/edit_example.png \
    --prompt "在书本封面Python的下方，添加一行英文文字2nd Edition" \
    --output_image output_edit.png \
    --seed 43
```

### 批量推理（RedBench评估）
```bash
python tools/redbench_infer.py \
    --model-path FireRedTeam/FireRed-Image-Edit-1.0 \
    --jsonl-path <数据集jsonl路径> \
    --save-path <输出目录> \
    --edit-task all \
    --seed 43 \
    --multi-folder
```

### RedBench评估
需要设置 `GEMINI_API_KEY` 环境变量：
```bash
python tools/redbench_eval.py \
    --result_img_folder <推理结果目录> \
    --edit_json <编辑信息jsonl路径> \
    --prompts_json <评估提示json路径> \
    --num_threads 50
```

## 核心架构

### Pipeline配置
- 使用 `QwenImageEditPlusPipeline`（来自 Diffusers）
- 默认使用 `torch.bfloat16` 数据类型
- 推理步骤默认 40 步
- True CFG scale 默认 4.0（关键参数，影响编辑指令跟随效果）
- Guidance scale 通常设为 1.0
- 需要 CUDA 设备

### RedBench数据格式
JSONL格式，每条记录包含：
- `id`: 唯一标识符
- `task`: 编辑任务类型（Add, Adjust, BG, Beauty, Color, Compose, Extract, Portrait, Low-level, Motion, Remove, Replace, Stylize, Text, Viewpoint）
- `source`: 源图片路径
- `a_to_b_instructions`: 中文编辑指令
- `a_to_b_instructions_eng`: 英文编辑指令

### 评估机制
- 使用 Gemini 3.0 Flash API 进行自动化评估
- 评估结果包含维度分数和按编辑类型分组的平均分
- 支持多线程并发评估（默认50线程）

## 重要参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `true_cfg_scale` | 4.0 | True CFG scale，控制指令跟随强度 |
| `guidance_scale` | 1.0 | 标准CFG scale，通常保持1.0 |
| `num_inference_steps` | 40 | 推理步数，影响质量和速度 |
| `negative_prompt` | " " | 空格作为默认负提示 |
