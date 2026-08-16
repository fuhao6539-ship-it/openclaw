---
name: recoverable-upgrade-two-layer-rollback
description: 当升级 Gateway 或其状态迁移后出现故障时，先准备并验证恢复点，并按代码优先、状态恢复仅在兼容性必要时的双层回退策略处置。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f07
---

# 可恢复升级与双层回退

## R — Reading（原文）

> Rollback has two layers:
>
> — `docs/install/updating.md`

## I — Interpretation（方法论）

升级是带恢复路径的变更。第一层是回退代码并保留当前 state；第二层仅用于旧代码无法读取迁移后的配置或数据库。状态恢复是时间旅行，会丢弃备份后的更改，因此进入前须保存当前状态并明确确认。

## A1 — Past Application（来源中的应用）

C-04：重大更新前创建并验证完整备份。更新失败时优先用 known-good version 进行代码回退；仅在兼容性问题明确时才恢复 pre-update state。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当升级或状态迁移后发生故障，需在优先代码回退与仅在兼容性必要时恢复 state 的双层回退路径之间作选择时调用。

**相邻技能区分**：本技能处理故障后的回退决策；恢复输入的 staging 验证与显式激活交给 `candidate-validation-before-activation`，恢复点的事前选择交给 `online-state-backup-path-selection`，运行中服务的更新交接交给 `supervised-update-handoff`。

## E — Execution（执行步骤）

1. 更新前创建可验证恢复点，并记录已知良好版本与服务健康基线。
2. 故障时先确认健康失败原因并尝试仅代码回退，保留当前 state。
3. 仅在旧代码不能使用迁移后 config/database 时，保存当前状态后执行状态恢复，并重新验证版本、服务和数据影响。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不用于无备份的破坏性数据修复；需先声明恢复能力缺口。它也不允许在运行中的 Gateway 内直接替换包树，交接问题应使用“受监督更新的进程外交接”。

## 相关 skills

`candidate-validation-before-activation`、`online-state-backup-path-selection`、`supervised-update-handoff`

## 证据索引

- 框架：`candidates/frameworks.md`（f07）
- 三重验证：`verified.md`（f07）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/install/updating.md`
