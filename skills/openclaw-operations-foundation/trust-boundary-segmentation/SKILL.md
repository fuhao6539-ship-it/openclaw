---
name: trust-boundary-segmentation
description: 当多个团队、客户或对抗性用户考虑共享 Gateway 时，判断是否处于同一受信操作员边界；不把 session 标签、共享认证或提示词隔离误作多租户隔离。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f04
---

# 信任边界拆分判定

## R — Reading（原文）

> OpenClaw is not a hostile multi-tenant security boundary for multiple
>
> — `docs/gateway/security/index.md`

## I — Interpretation（方法论）

先判定参与者是否共享同一受信操作员边界。若不共享，sessionKey 只是路由，不能充当授权或租户隔离；应拆分 Gateway 与凭据，并尽可能拆分 OS 用户或主机。共享 agent 的工具权力等于共享委托能力。

## A1 — Past Application（来源中的应用）

C-05：节点身份配对与能力审批被拆分；配对不自动开放命令面，待批准能力仍受全局 allow/deny 约束，体现身份与可执行范围不能混同。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当多个团队、客户或相互不信任、可能对抗的参与者考虑共享 Gateway、凭据、state 或宿主时调用，以判定是否处于同一受信操作员边界及应拆分到何种层级。

**相邻技能区分**：本技能决定部署与委托权力的边界，不配置单一入口的认证顺序；入口控制排序交给 `identity-first-scope-second-model-last`，单个 agent 的工具执行层诊断交给 `tool-execution-three-layer-control`。

## E — Execution（执行步骤）

1. 识别参与者关系、数据敏感度及是否存在相互不信任或对抗。
2. 检查当前隔离是否仅依赖 sessionKey、共享认证或提示词；若是，判定为不足。
3. 按风险拆分 Gateway、凭据、state/config/port；对高风险关系进一步拆分 OS 用户或主机，并验证交叉访问失败。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

纯单操作员、同一受信边界内的路由问题通常无需此技能。此框架不提供企业级细粒度 RBAC 设计，也不能消除共享宿主的侧信道风险。

## 相关 skills

`identity-first-scope-second-model-last`、`minimal-exposure-selection`、`tool-execution-three-layer-control`

## 证据索引

- 框架：`candidates/frameworks.md`（f04）
- 三重验证：`verified.md`（f04）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/gateway/security/index.md`
