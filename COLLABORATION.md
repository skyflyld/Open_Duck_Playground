# DuckMate × Codex 协作契约（COLLABORATION.md）

> 本仓库：Open_Duck_Playground fork（上游 apirrone/Open_Duck_Playground）
> 项目：DuckMate 大鸭——低成本 RL 四足机器人（standing M1 gate 已过，行走/joystick 训练中）

## 三方分工（Sky 拍板 2026-09-05）

| 角色 | 职责 | 权限边界 |
|---|---|---|
| **Sky**（决策） | 方向 / 优先级 / 训不训 / 终验 | 一切最终决定权 |
| **Codex**（内容质量 + Spec） | 定义「怎么训」：训练 Spec / 代码审计 / 可行性碰撞 / 复核打勾 | 不碰 GPU / AutoDL / 训练执行 |
| **Ariste**（工程执行） | 按 Spec 机械执行：AutoDL 训练 / 验收 / 落盘 / 记忆 | 偏离 Spec 必须标红，不静默改 |

**关键差异（vs ad-astra）**：DuckMate 的 RL 训练全链（AutoDL SSH/GPU/venv/ONNX）锁死 Ariste 侧、Codex 无访问。
→ 切法 = **Codex 定怎么训（Spec）→ Ariste 负责训 → Sky 定训不训**。
Spec 质量直接决定 GPU 时费烧多少（4090 实测 ¥2.18/h）。

## Spec 循环（决策-执行-验收闭环）

```
Codex 出 Spec（含验收标准）→ Sky 审批 → Ariste 执行（偏离标红）
→ 结果回传 → Codex 打勾/打回 → 关批
```

- Spec 先行把复核从「重做」降为「检查」
- 验收数据（reward/时长/跌倒次数）必须原始记录，不加工

## Issue 约定

- `[spec]` = Codex 出规格请求（如 joystick 行走）
- `[audit]` = 代码审计请求（next_actor=codex）
- `[feasibility]` = 可行性碰撞（要天敌式独立挑战，不要验收确认）
- 每 Issue 标注 next_actor（codex / ariste / sky），轮到谁谁响应

## 项目状态（截至迁移 2026-09-05）

- standing M1 gate ✅：reward 196.79/20s 零跌倒（¥2.2 成本）
- kick 宽窗 ✅：95% 踢飞
- **standing config 速度命令被注释 = 只会站不会走**（实测给前进命令直接摔）→ joystick/velocity 任务重训才是真行走
- joystick.py 101 维 obs vs standing 85 维；训练 4090：10M≈6min / 150M≈90min

---

# ⚠️ 操作纪律（2026-09-05 jasoneip01-pixel 暂停事故后制度化）

**背景**：协作载体账号因 11 分钟内 11 次 API 写操作 burst 被 GitHub 反滥用系统暂停。
本纪律对所有通过 GitHub 操作本仓库的参与方（Ariste / Codex / 任何自动化）强制生效。

## 通用铁律（双侧强制）

1. **写操作限速**：单次会话写操作（建 issue / 评论 / push / PATCH / 删文件）≤5 次，间隔 ≥60 秒
2. **commit 聚合**：一个工作单元 = 1 个 commit + 1 次 push；禁止逐文件 commit、禁止空 commit/测试 commit
3. **commit message 写清楚**（说明目的，非 spam 特征）
4. **禁止 force-push 到共享分支**（main）；force-push 仅限私有临时分支
5. **不批量建 repo**：一次最多 1 个、间隔 ≥30 分钟、建完立即推真实内容
6. **PR 优先于直接 push main**：功能开发走 feature 分支 + 完整描述的 PR

## Codex 侧额外约束

- 只操作被指派的 repo，不碰无关仓库
- 一次会话最多 3 个 issue/评论，间隔自然化
- 评论/Issue 内容必须有信息量，不发空评论

## Ariste 侧额外约束

- 操作前检查账号状态（rate limit / 403 头）；会话结束复检一次
- 写操作日志记录（时间/端点/repo）→ 当天日记，出事可精确复盘
- 建 repo 需 Sky 单独审批

## 监控与熔断

```
🟢 正常：rate 满额、无 403 → 继续
🟡 预警：secondary rate limit / 偶发 403 → 全停写操作 30 分钟，只读检查
🔴 危险：账号 403 suspended → 全停所有自动化，人工介入（Sky）
```

## 账号事实（2026-09-05）

- jasoneip01-pixel：**被暂停**（申诉中）——本 fork 原载体，含 Issues #1-4
- skyflyld：健康——本仓库现载体（迁移 2026-09-05，97 文件全量）
- k286c7hg65-hub：Sky 新账号（养号中）= **jasoneip01-pixel 的替代主账号**（Sky 确认 2026-09-05）——产品线 + Codex 协作最终归它；skyflyld 退回备份/学术线

## 恢复路径

- jasoneip01-pixel 申诉成功 → 看板 Issues #1-4 无缝继续
- 或 Sky 决定 → 新账号重建看板（Ariste 执行）
