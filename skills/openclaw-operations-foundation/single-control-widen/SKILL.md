---
name: single-control-widen
description: 当需要改变 Gateway 的 bind、代理、渠道入口或工具权限时，将复合高风险变更拆成单控制变量序列，并在每一步完成授权、拒绝与工具边界验证。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f02
---

# 单控制逐步放宽与逐步验证

## R — Reading（原文）

> Widen one control at a time: add a specific channel allowlist before enabling
>
> — `docs/gateway/security/exposure-runbook.md`

## I — Interpretation（方法论）

把 bind、代理、渠道策略与工具权限视为独立控制面。一次只改变一个，随后验证授权成功、未授权拒绝、路由正确、敏感数据不泄露，以及高影响工具被拒绝或进入审批。当前变化的影响未理解前，不进入下一步。

## A1 — Past Application（来源中的应用）

C-01：远程暴露采用预检、严格入口策略、网络可达性与工具放宽的分步路径；每项控制均有验证与回退动作。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当计划同时改变 bind、代理、渠道入口或工具权限，且必须将复合高风险变更拆成单控制变量、在每步验证授权、拒绝与路由结果时调用。

**相邻技能区分**：本技能管理变更顺序，不选择网络暴露模式；暴露模式选择交给 `minimal-exposure-selection`，审计结论所需的证据层级交给 `two-tier-audit-evidence`，受管服务的停止/替换/重启交接交给 `supervised-update-handoff`。

## E — Execution（执行步骤）

1. 列出计划中所有控制面，并按依赖关系拆成单变量序列。
2. 实施当前一个变量；保留变更前快照和可执行回滚动作。
3. 运行授权/未授权、路由、脱敏和高影响工具五类检查；未得到解释性证据即回滚或停止。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不适用于正在发生的安全事故：应先恢复已知安全状态。配置语法修复也不需要人为拆成多步。该模型不替代团队并发变更协调或完整变更管理。

## 相关 skills

`minimal-exposure-selection`、`two-tier-audit-evidence`、`supervised-update-handoff`

## 证据索引

- 框架：`candidates/frameworks.md`（f02）
- 三重验证：`verified.md`（f02）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/gateway/security/exposure-runbook.md`
