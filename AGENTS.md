# Repository Guidelines

## Project Structure & Module Organization

- `src/llamafactory/`: 核心 Python 包（`train/`, `model/`, `data/`, `api/`, `webui/`, `v1/` 等）。
- `src/train.py`, `src/webui.py`, `src/api.py`: 入口脚本（通常通过 `llamafactory-cli` 间接调用）。
- `examples/`: 训练/推理/导出/API 的示例 YAML 配置与用法（优先复用并在 PR 中引用具体路径）。
- `tests/`, `tests_v1/`: `pytest` 测试套件（含不同后端/设备相关用例）。
- `scripts/`: 维护脚本与开发工具；`docs/`: 文档；`docker/`: 容器环境；`assets/`, `data/`: 资源与示例数据。

## Build, Test, and Development Commands

- `pip install -e .`: 本地可编辑安装（开发推荐）。
- `pip install -r requirements/dev.txt`: 安装开发依赖（`ruff`/`pre-commit`/`pytest` 等）。
- `make style`: 使用 `ruff` 自动修复 lint/format 问题。
- `make quality`: 只检查（`ruff check` + `ruff format --check`），适合 CI/提交前自检。
- `make test`: 运行 `pytest`（默认 `WANDB_DISABLED=true`，覆盖 `tests/` 与 `tests_v1/`）。
- `make commit`: 安装并运行 `pre-commit` hooks（等价于一次完整提交前检查）。
- `make build`: 构建分发包（若安装 `uv` 会自动加速，否则使用 `python -m build`）。

## Coding Style & Naming Conventions

- Python 版本：`>=3.11`（见 `pyproject.toml`）。
- 格式化与静态检查：统一使用 `ruff`；缩进 4 空格，行宽 119，字符串双引号，import 自动排序。
- 约定：尽量沿用既有模块命名与目录分层；新增配置优先放到 `examples/` 并使用清晰的文件名（如 `qwen3_lora_sft.yaml`）。

## Testing Guidelines

- 测试框架：`pytest`；命名遵循 `test_*.py` / `Test*`。
- 常用标记：`@pytest.mark.slow`（默认跳过，设置 `RUN_SLOW=1` 才运行）、`runs_on([...])`（设备类型）、`require_distributed(n)`（多卡）。
- 运行示例：`pytest -vv tests/ tests_v1/`；单测优先在本地跑通再提交。

## Commit & Pull Request Guidelines

- 提交信息尽量与历史一致：常见格式为 `[scope] 简短动词开头的说明`（如 `[model] support ...`）；也接受 `feat:`/`fix:` 等 Conventional Commits 风格，但请保持同一 PR 内一致。
- PR 需包含：变更动机与影响面、复现/验证步骤、相关配置路径（优先 `examples/...`）、本地自检结果（至少 `make quality` 与 `make test`）。
- 涉及 WebUI/交互行为变更时，附上截图或简短录屏说明；不要提交密钥/Token（本地可用 `.env.local`，线上用环境变量如 `HF_TOKEN`）。

## Security & Configuration Tips

- 不要在 YAML/脚本中硬编码凭据；使用环境变量（如 `HF_TOKEN`）或本地 `.env.local`（切勿提交到仓库）。
- 可选依赖按需安装：如评测指标 `pip install -r requirements/metrics.txt`，分布式训练 `pip install -r requirements/deepspeed.txt`。

## Agent-Specific Notes

- 面向自动化/Agent 的仓库协作约定与快捷命令参考 `CLAUDE.md`。
