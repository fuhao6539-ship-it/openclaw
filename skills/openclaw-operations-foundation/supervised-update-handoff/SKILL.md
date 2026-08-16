---
name: supervised-update-handoff
description: 当正在提供服务的受管 Gateway 需要更新时，将包替换交给进程外、可观察的交接流程，并以服务元数据刷新、重启及健康/版本验证而非“包已安装”作为完成条件。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f10
---

# 受监督更新的进程外交接

## R — Reading（原文）

> Package-manager updates requested through the live Gateway control-plane
>
> — `docs/install/updating.md`

## I — Interpretation（方法论）

运行中的服务不应在自身进程内替换正加载的包树。更新由外部交接路径协调停止、替换、插件/服务元数据收敛、重启及验证；若无法安全交接，应给出可审查的人工操作，而非强行进程内更新。

## A1 — Past Application（来源中的应用）

C-03：标准更新会检测安装类型、运行 Doctor 并重启 Gateway；手动包管理器替换受管安装前必须先停止 Gateway，更新后用 Doctor、restart 与 health 验证。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当受管 Gateway 需要更新且不能在运行进程内替换正加载的包树，需通过进程外交接完成停止、替换、服务元数据收敛、重启和健康验证时调用。

**相邻技能区分**：本技能处理受管更新的安全交接，不决定更新失败后的恢复层次；失败后的代码/state 回退交给 `recoverable-upgrade-two-layer-rollback`，验证结论的审计证据层级交给 `two-tier-audit-evidence`，多控制面放宽的变更排序交给 `single-control-widen`。

## E — Execution（执行步骤）

1. 确认安装类型、监督方式和当前服务健康；先用 dry-run 或等价预览确认计划。
2. **人类确认检查点**：在停止受管 Gateway、手动替换包或重启前，显式确认变更授权、最小必要范围、回滚条件与拒绝条件。若安装类型、监督方式或进程外交接路径未知，收集证据并提议人工交接，但不替换包。
3. 使用受管的进程外交接更新；手动替换时先停止 managed Gateway，完成后刷新服务元数据。
4. **验证清单**：重启后核验运行版本、Doctor、health 与必要的插件/可达性状态；任何失败都不以“安装成功”报告完成。
5. **降级输出路径**：无法安全交接时，给出可审查的人工操作方案，而非强行进程内更新。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；说明哪些关键事实必须保持未知而不能假设。

## B — Boundary（边界与限制）

不用于应用内热更新或不受监督的一次性开发试验。若需要回退，应使用“可恢复升级与双层回退”；该技能不保证第三方包管理器或插件兼容性。

**成功/失败验证与交接**：

- **成功条件**：运行版本符合预期、Doctor 通过、health 正常，且必要的插件/可达性状态符合预期。
- **失败条件**：任一验证失败时，不得声称完成；记录未决风险并交接给 `recoverable-upgrade-two-layer-rollback`。
- **最小证据包**：运行版本、Doctor 输出、health 状态、插件/可达性状态与回滚条件。
- **拒绝条件**：安装类型、监督方式或进程外交接路径无法明确时，不得强行进程内更新；应给出可审查的人工操作方案。

## 相关 skills

`recoverable-upgrade-two-layer-rollback`、`two-tier-audit-evidence`、`single-control-widen`

## 证据索引

- 框架：`candidates/frameworks.md`（f10）
- 三重验证：`verified.md`（f10）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/install/updating.md`
