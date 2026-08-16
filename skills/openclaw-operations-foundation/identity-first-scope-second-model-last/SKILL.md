---
name: identity-first-scope-second-model-last
description: 当设计不受信输入、消息入口、webhook 或提示注入防护时，先确定谁可触发，再限制资源和工具范围，最后才以模型行为和提示词作为补强。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f03
---

# 身份优先、范围其次、模型最后

## R — Reading（原文）

> 1. **Identity first** - decide who can talk to the bot (DM pairing / allowlists / explicit "open").
>
> — `docs/gateway/security/index.md`

## I — Interpretation（方法论）

控制顺序是身份层 → 范围层 → 模型层。身份层决定谁能触发；范围层限制会话、资源、工具和执行位置；模型层面对可能被操纵的模型做补强。提示词或模型表现不能替代认证与最小权限。

## A1 — Past Application（来源中的应用）

C-01：暴露 Gateway 时以 DM pairing/allowlist 确认入口，以 per-channel-peer、exec deny、关闭 elevated 与沙箱收缩范围，最后才将模型指令作为补充。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当设计不受信输入、消息入口、webhook 或提示注入防护，且需按“身份认证 → 可作用范围 → 模型补强”的顺序安排控制时调用。

**相邻技能区分**：本技能安排控制优先级；若问题是多个不受信主体能否共用 Gateway、凭据或宿主，交给 `trust-boundary-segmentation`；若问题是 sandbox、tool policy 或 elevated gate 的具体执行层诊断，交给 `tool-execution-three-layer-control`。

## E — Execution（执行步骤）

1. 画出触发者清单与认证路径；没有可验证身份时先收缩或关闭入口。
2. 为已认证主体配置最小会话、工具、执行和网络范围。
3. 再配置提示词、输入处理或输出过滤，并测试“模型被诱导时范围层仍阻止越权”。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不用于判断不同团队是否应共享运行环境，后者应使用“信任边界拆分判定”。它不证明模型层防护有效，也不替代应用、操作系统或网络漏洞修复。

## 相关 skills

`trust-boundary-segmentation`、`tool-execution-three-layer-control`、`minimal-exposure-selection`

## 证据索引

- 框架：`candidates/frameworks.md`（f03）
- 三重验证：`verified.md`（f03）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/gateway/security/index.md`
