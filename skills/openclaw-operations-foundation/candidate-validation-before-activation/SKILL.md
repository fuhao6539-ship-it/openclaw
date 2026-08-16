---
name: candidate-validation-before-activation
description: 当恢复 archive、数据库或 state directory 时，先将候选恢复到 staging 目标并验证结构、清单与完整性，再由操作员在 Gateway 停止后显式激活生产状态。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f08
---

# 先验证候选、后激活的恢复模型

## R — Reading（原文）

> Restore is deliberately explicit; nothing overwrites live state in place.
>
> — `docs/install/backups.md`

## I — Interpretation（方法论）

将恢复分成候选提取/验证与生产激活。候选在新的 staging 目标中检查 manifest、路径、SQLite 完整性、凭据和渠道状态；只有 Gateway 停止后才由操作员显式迁移或切换 state。这样避免不可信或未验证输入覆盖 live state。

## A1 — Past Application（来源中的应用）

CE-09：archive restore 不会原地激活；运行中的 Gateway 不应直接覆盖。恢复后需执行 Doctor、Gateway restart、status，并验证渠道、dashboard/session 与 workspace 文件。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当恢复 archive、数据库或 state directory，且需把候选提取与 staging 验证严格同生产状态的显式激活分离时调用。

**相邻技能区分**：本技能处理恢复候选到生产激活的闸门，不决定备份方法或升级回退层次；在线恢复点的选择交给 `online-state-backup-path-selection`，升级故障后的代码/state 回退选择交给 `recoverable-upgrade-two-layer-rollback`。

## E — Execution（执行步骤）

1. 将恢复输入解压或还原到新的 staging target，绝不直接覆盖 live state。
2. 核验 manifest、数据库完整性、路径映射、凭据和渠道状态，记录所有差异。
3. 停止 Gateway 后由操作员显式迁移或切换状态目录；运行 Doctor、重启和状态/功能验证，失败则不激活。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不替代备份真实性或供应链验证。旧渠道状态可能与当前外部凭据不同步；恢复不是无副作用操作，且需要处理凭据轮换与插件重新安装。

## 相关 skills

`recoverable-upgrade-two-layer-rollback`、`online-state-backup-path-selection`、`supervised-update-handoff`

## 证据索引

- 框架：`candidates/frameworks.md`（f08）
- 三重验证：`verified.md`（f08）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/install/backups.md`
