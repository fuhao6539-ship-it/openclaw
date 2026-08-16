---
name: two-tier-audit-evidence
description: 当需要解释安全审计结论强度、暴露 Gateway 或重大变更后验证时，区分只读静态冷路径审计与包含运行时探测、插件扫描的深度证据。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f06
---

# 两级审计证据模型

## R — Reading（原文）

> Plain `security audit` stays on the cold config/filesystem/read-only path:
>
> — `docs/cli/security.md`

## I — Interpretation（方法论）

常规审计提供配置和文件系统的低副作用证据；深度审计增加 best-effort 的运行时探测与插件/技能扫描。两层覆盖面不同：静态通过不等于运行时安全，深度结果也不是完整证明。应根据变更风险选择并标注证据层级。

## A1 — Past Application（来源中的应用）

C-02：安全审计以 checkId 给出结构化发现；插件/技能扫描和 live Gateway probe 只在 --deep 运行。修复后需复跑相同审计范围以确认 finding 不再活跃。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当需要解释安全审计结论的覆盖边界，或为 Gateway 暴露与重大变更选择普通冷路径审计、深度运行时探测及其相应证据强度时调用。

**相邻技能区分**：本技能决定如何证明结论，不选择暴露模式或编排实际变更；暴露模式交给 `minimal-exposure-selection`，单变量变更顺序交给 `single-control-widen`，受管更新交接交给 `supervised-update-handoff`。

## E — Execution（执行步骤）

1. 明确待证明的结论与风险：日常配置检查先用冷路径，暴露或重大变更加入深度检查。
2. 保存命令范围、checkId、探测是否成功或受阻，不把缺失探测当作通过。
3. 结合授权/拒绝的实际测试解释结论边界，并在变化后复跑相同范围。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不用于自动修复 critical finding；需按 finding 的主修复路径处理。任何一层审计均不能替代威胁建模、人工代码审查或持续监控。

## 相关 skills

`single-control-widen`、`minimal-exposure-selection`、`supervised-update-handoff`

## 证据索引

- 框架：`candidates/frameworks.md`（f06）
- 三重验证：`verified.md`（f06）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/cli/security.md`
