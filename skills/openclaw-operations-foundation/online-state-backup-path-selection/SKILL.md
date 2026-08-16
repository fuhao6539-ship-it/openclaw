---
name: online-state-backup-path-selection
description: 当需要为正在运行的 Gateway、SQLite 或完整 state 建立恢复点时，按在线状态、恢复目标与一致性要求选择 archive、SQLite snapshot、Git 或连续复制，而非直接复制 live 数据库文件。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f09
---

# 在线状态的备份路径选择

## R — Reading（原文）

> Never copy live `.sqlite`, `-wal`, `-shm`, or `-journal` files as a backup.
>
> — `docs/install/backups.md`

## I — Interpretation（方法论）

备份方法由“状态是否在线”“需要完整便携恢复还是单库恢复”“RPO 与版本化需求”决定。在线数据库只能走捕获已提交状态的受支持路径；备份本身含敏感 state，必须按 live state 的加密、权限和泄露轮换要求保护。

## A1 — Past Application（来源中的应用）

CE-09：机器迁移快照前停止 Gateway，避免变化中的 SQLite 与 WAL 不一致；配置文件本身不足以恢复，应迁移完整 state directory。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当需要为仍在线的 Gateway、SQLite 或完整 state 建立一致恢复点，并在 archive、SQLite snapshot、Git 与连续复制之间按恢复目标和一致性取舍时调用。

**相邻技能区分**：本技能选择备份路径，不激活恢复候选或决定升级失败后的回退；候选恢复验证交给 `candidate-validation-before-activation`，代码与 state 回退选择交给 `recoverable-upgrade-two-layer-rollback`，审计结论强度交给 `two-tier-audit-evidence`。

## E — Execution（执行步骤）

1. 明确恢复目标、可接受的数据损失窗口和状态是否在线。
2. 选择受支持路径：完整便携恢复用 archive，单数据库用 SQLite snapshot，版本化增量用 Git，低 RPO 用连续复制。
3. **用户确认门**：在执行任何备份路径前，显式确认恢复目标、RPO、状态范围、在线状态、回滚条件与拒绝条件；信息不足时先提出最小澄清问题。
4. **决策记录**：路径选择后记录恢复目标、在线/离线状态、RPO、选择路径及理由、验证证据、回滚条件与未决风险。
5. 验证备份可读取/恢复，并按 live state 管理加密、访问权限与泄露后的凭据轮换。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；说明哪些关键事实必须保持未知而不能假设。

## B — Boundary（边界与限制）

不把普通文件复制、只复制 SQLite/WAL 或仅保存配置当作一致性备份。该框架不规定组织的保留期限、异地策略或合规要求。

**停止条件与安全回退**：

- 备份可读性或可恢复性无法证明时，不得声称完成；记录未决风险并提出最小澄清问题。
- 连续复制的已提交状态一致性无法验证时，不得声称完成；记录未决风险并提出最小澄清问题。
- 恢复目标、RPO 或状态范围无法明确时，先澄清而不假定生产配置。

## 相关 skills

`candidate-validation-before-activation`、`recoverable-upgrade-two-layer-rollback`、`two-tier-audit-evidence`

## 证据索引

- 框架：`candidates/frameworks.md`（f09）
- 三重验证：`verified.md`（f09）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/install/backups.md`
