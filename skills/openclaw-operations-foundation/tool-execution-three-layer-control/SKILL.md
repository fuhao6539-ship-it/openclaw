---
name: tool-execution-three-layer-control
description: 当工具被阻止、意外能访问宿主或需要最小权限修复时，分别诊断 sandbox 运行位置、tool policy 可用性与 elevated exec 逃逸门，而不把它们当作同一个开关。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f05
---

# 工具执行三层控制模型

## R — Reading（原文）

> 1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.entries.*.sandbox.*`)
>
> — `docs/gateway/sandbox-vs-tool-policy-vs-elevated.md`

## I — Interpretation（方法论）

三层分别回答：工具在哪里运行（sandbox）、能否被调用（allow/deny policy）、exec 是否可从沙箱外执行（elevated gate）。允许工具不代表其在宿主运行；开启 elevated 也不会增加其他工具权限。修复前先读取实际生效配置。

## A1 — Past Application（来源中的应用）

CE-01：沙箱仅移动启用后的工具执行；Gateway 进程保持在宿主，显式 elevated 的工具可在沙箱外运行。因此不能把“开启沙箱”当成完整隔离证明。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当工具不可调用、运行位置不符合预期、意外取得宿主访问，或需在 sandbox、tool policy 与 elevated exec gate 三层中定位并最小化修复一层时调用。

**相邻技能区分**：本技能诊断工具执行层，不决定谁可触发系统或不同主体是否能共享环境；控制优先级交给 `identity-first-scope-second-model-last`，信任边界拆分交给 `trust-boundary-segmentation`，多控制面变更排序交给 `single-control-widen`。

## E — Execution（执行步骤）

1. **分层分诊**：明确问题是“工具不可调用”“运行位置不对”还是“需要宿主 exec”，并收集生效配置；不要把 sandbox 位置、tool policy 可用性与 elevated exec 合并为同一个控制。
2. **人类确认检查点**：在跨越运行位置边界（例如 sandbox 到 host）或权限边界（例如开启 elevated exec）前，显式确认变更授权、最小必要范围、回滚条件与拒绝条件；信息不足时先提出最小澄清问题。
3. 按对应层采取最小变更：仅调整必要 allow/deny、sandbox scope 或单项 elevated gate。
4. 验证目标任务与拒绝路径，确认未意外扩大其他工具权限或宿主访问。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；每层都要说明哪些关键事实必须保持未知而不能假设。

## B — Boundary（边界与限制）

不用于对抗性多租户隔离证明；sandbox 并非完美边界。也不应用 elevated 作为依赖、网络或配置问题的默认绕过方案。

**每层最小证据与停止条件**：

- **Sandbox 层**：记录工具实际运行位置、生效配置路径与未验证假设。
- **Tool policy 层**：记录 allow/deny 规则、生效范围与未验证假设。
- **Elevated exec 层**：记录 gate 状态、授权条件与未验证假设。
- 任一层的关键证据无法验证时，不得声称修复完成；记录未决风险并提出最小澄清问题。

## 相关 skills

`identity-first-scope-second-model-last`、`trust-boundary-segmentation`、`single-control-widen`

## 证据索引

- 框架：`candidates/frameworks.md`（f05）
- 三重验证：`verified.md`（f05）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/gateway/sandbox-vs-tool-policy-vs-elevated.md`
