# Open Duck Playground — DuckMate 大鸭训练库（DuckMate fork）

> **DuckMate 大鸭项目 fork**：42cm 开源双足 AI 机器人鸭的 RL 训练仿真栈。
> 上游：https://github.com/apirrone/Open_Duck_Playground（ODM 作者官方库）
> 本 fork 由 DuckMate 团队维护，加入大鸭（Open Duck Mini v2 42cm）训练所需的环境修复、验证脚本与执行包。

## 与上游的差异（本 fork 增量）

| 文件 | 说明 |
|------|------|
| `SETUP.md` | 环境搭建完整手册（镜像加速 / 版本兼容 / 磁盘坑），9/3 实测踩坑记录 |
| `tools/apply_brax_fix.py` | brax 0.14.2 × jax 0.11.1 单卡兼容自动修复（device_put_replicated 移除） |
| `tools/m1_gate_eval.py` | M1 gate headless 验收：ONNX 策略在 mujoco 模拟中站 20s 检测（无头服务器可用） |
| `odm_m1_gate.sh` | M1 gate 一键执行包：部署 → standing 小验证 → 全量训练 → ONNX 导出 |

## 版本兼容（必读，2026-09-03 实测）

- **mujoco_playground 必须用 0.0.5**：0.2.0（PyPI 最新）移除了 `_src/collision.py`，ODM 代码 `from mujoco_playground._src.collision import geoms_colliding` 会 import 失败。
  0.1.0 起移除；0.0.5 是最后可用版。
  ```bash
  uv pip install "playground==0.0.5"
  ```
- **brax 0.14.2 × jax 0.11.1**：`jax.device_put_replicated` 被移除（pmap 新语义），brax 4 处调用需修复（单卡等价 = 加 leading device dim）：
  ```bash
  uv run python tools/apply_brax_fix.py
  ```
- 训练必须走 pmap 路径（`use_pmap_on_reset=True` 默认），jax 0.11 pmap 输入需带 device 维 `[1, N]`——apply_brax_fix.py 已处理。

## Installation

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 国内镜像加速（实测阿里云 3.7MB/s > 清华 2MB/s）：
export UV_DEFAULT_INDEX=https://mirrors.aliyun.com/pypi/simple/
uv sync
uv pip install "playground==0.0.5"
uv run python tools/apply_brax_fix.py   # 单卡 GPU 必跑
```

## Training（standing 最小验证）

```bash
# 小验证 10M（~6min on 4090，看收敛；reward 达标线 60-80）
WANDB_MODE=disabled uv run playground/open_duck_mini_v2/runner.py \
  --env standing --task flat_terrain --num_timesteps 10000000 --output_dir checkpoints_verify

# 全量 150M（~15min on 4090；2026-09-03 实测 reward 196.8）
WANDB_MODE=disabled uv run playground/open_duck_mini_v2/runner.py \
  --env standing --task flat_terrain --num_timesteps 150000000 --output_dir checkpoints_full
```

ONNX 自动导出：runner.py 每个 checkpoint 自动 `export_onnx` → `checkpoints_*/<date>_<step>.onnx`（obs 85 → 512-256-128 → act 14）。

## M1 gate 验收（模拟器实测，非 reward 推断）

```bash
uv run python tools/m1_gate_eval.py <policy.onnx> 20
# 通过标准：20s 全程站立零跌倒（body z 稳定 ~0.15m）
# 2026-09-03 实测：PASS（z mean 0.1515, min 0.1489）
```

M1 gate 四项标准：① 模拟实站 20s 零跌倒 ② reward 收敛（实测 196.8）③ GPU 成本 <$3（实测 ¥2.2）④ ONNX 落盘。

## Inference（mujoco viewer，需图形界面）

```bash
uv run playground/open_duck_mini_v2/mujoco_infer.py -o <path_to_.onnx> --standing
```

> ⚠️ mujoco_infer.py 的 get_obs 是 joystick 布局（101 维），standing 训练 obs 是 85 维——
> headless 验收请用 `tools/m1_gate_eval.py`（已精确对齐训练布局）。

## Tensorboard

```bash
uv run tensorboard --logdir=<yourlogdir>
```

## Project structure

```
.
├── pyproject.toml
├── README.md
├── SETUP.md                 # 本 fork：环境手册
├── odm_m1_gate.sh           # 本 fork：一键执行包
├── tools/
│   ├── apply_brax_fix.py    # 本 fork：brax 单卡兼容修复
│   └── m1_gate_eval.py      # 本 fork：headless 验收
└── playground
    ├── common/              # runner / export_onnx / onnx_infer / rewards
    └── open_duck_mini_v2/   # standing / joystick / mujoco_infer / xmls / data
```

## 许可与致谢

上游 MIT（见上游 LICENSE）。DuckMate 增量文件同 MIT。
感谢 apirrone 的 Open Duck Mini 开源生态（ODM 3923⭐ / MicroDuck 6922⭐）。
