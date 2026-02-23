# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

LLaMA Factory 是一个统一的高效大语言模型微调框架，支持 100+ 种 LLM 模型。

**核心特性：**
- 支持多种模型：LLaMA、LLaVA、Mistral、Qwen、DeepSeek、Gemma、GLM、Phi 等
- 多种训练方法：预训练、监督微调、奖励建模、PPO、DPO、KTO、ORPO 等
- 灵活的资源需求：16-bit 全参数微调、冻结微调、LoRA 和 QLoRA 变体
- 高级算法：GaLore、BAdam、APOLLO、Adam-mini、Muon、OFT、DoRA 等
- Web UI（LLaMA Board）和 CLI 两种界面

## 常用开发命令

### 代码风格和质量检查
```bash
make style      # 自动修复代码风格问题（ruff）
make quality    # 检查代码质量
make commit     # 安装并运行 pre-commit hooks（包含 style + quality）
make license    # 检查 Apache 2.0 许可证头
```

### 测试
```bash
make test       # 运行测试套件（pytest）
# 单独运行特定测试：WANDB_DISABLED=true pytest -vv tests/test_xxx.py
```

### 构建和安装
```bash
pip install -e .                    # 从源码安装
pip install -e ".[dev]"             # 安装开发依赖
pip install -r requirements/metrics.txt      # 安装评估指标依赖
pip install -r requirements/deepspeed.txt    # 安装 DeepSpeed 依赖
```

### 训练和推理
```bash
# CLI 训练
llamafactory-cli train examples/train_lora/qwen3_lora_sft.yaml

# Web UI
llamafactory-cli webui
# 或
python src/webui.py

# API 服务器
llamafactory-cli api examples/inference/qwen3.yaml infer_backend=vllm

# 聊天界面
llamafactory-cli chat examples/inference/qwen3_lora_sft.yaml

# 导出/合并模型
llamafactory-cli export examples/merge_lora/qwen3_lora_sft.yaml
```

## 代码架构

### 双架构系统

LLaMA Factory 有两套并行的架构，可通过 `USE_V1` 环境变量切换：

**v0（默认架构）：**
```
src/llamafactory/
├── api/           # OpenAI 风格 API 实现
├── chat/          # 聊天界面实现
├── data/          # 数据处理和数据集处理
├── eval/          # 模型评估工具
├── extras/        # 额外工具和辅助函数
├── hparams/       # 超参数定义（dataclass）
├── model/         # 模型加载、补丁和工具
├── train/         # 训练流程实现
└── webui/         # Gradio Web 界面
```

**v1 架构（USE_V1=1）：**
```
src/llamafactory/v1/
├── trainers/      # 训练实现
├── core/          # 核心训练工具
├── accelerator/   # 加速和分布式训练
├── plugins/       # 可插拔组件（model、data、sampler、trainer）
├── config/        # 配置管理
└── utils/         # 工具函数
```

### 入口脚本
- `src/train.py` → `llamafactory.train.tuner`
- `src/webui.py` → `llamafactory.webui.interface`
- `src/api.py` → `llamafactory.api.app`

### 关键配置文件
- `data/dataset_info.json` - 所有可用数据集的定义
- `examples/` - 各种训练场景的示例配置（YAML/JSON）
- `src/llamafactory/hparams/` - 超参数 dataclass 定义

## 代码规范

### 风格要求
- 遵循 Google Python Style Guide
- 使用 **ruff** 进行 linting 和格式化（非 black）
- 行长度：119 字符
- 缩进：4 空格
- 引号风格：双引号
- 文档字符串：Google 风格

### 导入组织
- 已知第一方：`llamafactory`
- 已知第三方：`accelerate`, `datasets`, `gradio`, `numpy`, `peft`, `torch`, `transformers`, `trl`
- 导入后保留 2 个空行

### 许可证
- 所有源文件必须包含 Apache 2.0 许可证头
- 提交前运行 `make license` 检查

## 关键概念和模式

### 数据集格式
支持两种主要格式：
1. **Alpaca 格式** - 标准 instruction/input/output 结构
2. **ShareGPT 格式** - 多角色对话结构（human/gpt/observation/function）

数据集定义在 `data/dataset_info.json` 中。添加自定义数据集时，必须在此文件中添加描述。

### 模型支持
- 新模型支持通过 `src/llamafactory/model/` 中的模型补丁添加
- 视觉模型使用 `src/llamafactory/model/model_utils/visual.py`
- 量化支持在 `src/llamafactory/model/model_utils/quantization.py`
- 模型模板在 `src/llamafactory/data/template.py`

### 训练方法
支持以下训练方法（通过 `stage` 参数指定）：
- `pt` - 预训练（Pre-training）
- `sft` - 监督微调（Supervised Fine-tuning）
- `rm` - 奖励建模（Reward Modeling）
- `ppo` - PPO 训练
- `dpo` - DPO 训练
- `kto` - KTO 训练
- `orpo` - ORPO 训练
- `simpo` - SimPO 训练

### 推理后端
- `auto`（默认）- 原生 PyTorch
- `vllm` - vLLM 加速推理
- `sglang` - SGLang 加速推理

## 开发工作流

1. **环境设置**
   ```bash
   pip install -e ".[dev]"
   ```

2. **开发前检查**
   ```bash
   make style && make quality
   ```

3. **运行测试**
   ```bash
   make test
   ```

4. **提交代码**
   ```bash
   make commit  # 运行 pre-commit hooks
   git add ...
   git commit ...
   ```

## 重要注意事项

- 训练配置需要 GPU 机器，训练本身通常不在测试中进行端到端测试
- 使用 `WANDB_DISABLED=true` 禁用 wandb 以避免测试期间的外部依赖
- 项目支持多个模型和数据集 Hub：Hugging Face、ModelScope、Modelers
- Docker 支持可用的预构建镜像
- SwanLab 和 W&B 支持实验跟踪

## 扩展阅读

- 完整 README：`README.md` 和 `README_zh.md`
- 数据格式文档：`data/README.md`
- 示例配置：`examples/README.md`
- Copilot 指令：`.github/copilot-instructions.md`（英文）
