---
name: minimal-exposure-selection
description: 当需让 Gateway 超出 loopback 可达时，按最小攻击面选择满足工作流的暴露模式；用于远程访问、Control UI 暴露、Tailscale/代理/公网方案比较与攻击面评估。
source: OpenClaw 官方运维文档集：安全、升级与 Gateway（Batch A）
source_commit: 063ce57caf233dc3cac124349ccddf6cef9ca432
release_anchor: v2026.7.1-2
framework_id: f01
---

# 最小满足暴露面选择

## R — Reading（原文）

> Prefer the narrowest pattern that satisfies the workflow.
>
> — `docs/gateway/security/exposure-runbook.md`

## I — Interpretation（方法论）

按暴露面由小到大评估：保持 loopback → SSH tunnel 或 Tailscale Serve → 受限 tailnet/LAN → 受信反向代理 → 罕见的公网暴露。每扩大一级，都须补齐认证、网络过滤、TLS、速率限制、allowlist 与隔离控制；“能连通”不是完成条件。

## A1 — Past Application（来源中的应用）

C-01：Gateway 远程暴露前，运行 Doctor、安全审计、深度审计和健康检查；选择最窄模式，并验证授权成功、未授权拒绝、正确路由、日志脱敏与高影响工具受控。

**证据边界**：案例是冻结 P0 语料中的运维证据，不是对当前环境的变更授权。

## A2 — Future Trigger（何时调用）

当需要为 Gateway 远程访问或 Control UI 比较 loopback、SSH tunnel、Tailscale Serve、受限 tailnet/LAN、受信反向代理或公网暴露模式，并选择最小满足暴露面时调用。

**相邻技能区分**：本技能只决定暴露模式及所需攻击面控制；模式选定后，若需把放宽动作拆成单变量序列，交给 `single-control-widen`；若需确定普通或深度审计的证据强度，交给 `two-tier-audit-evidence`。不用于跨不受信主体的隔离判定，后者交给 `trust-boundary-segmentation`。

## E — Execution（执行步骤）

1. 澄清访问者、来源网络与所需表面（Control UI、API 或渠道）。
2. 从 loopback 起，选择第一个满足需求的模式；不得因方便跳级到公网。
3. 为所选模式配套认证、网络过滤和最小工具权限，再验证授权/拒绝路径并记录回滚方案。

**输出要求**：给出选择理由、执行前提、验证证据、回滚条件和未决风险；信息不足时先提出最小澄清问题，不假定生产配置。

## B — Boundary（边界与限制）

不用于具体代理参数或单条配置键的操作指南。它也不提供敌对多租户隔离；跨不受信边界时改用“信任边界拆分判定”。公网暴露仍需独立威胁建模和 live 验证。

## 相关 skills

`single-control-widen`、`identity-first-scope-second-model-last`、`trust-boundary-segmentation`

## 证据索引

- 框架：`candidates/frameworks.md`（f01）
- 三重验证：`verified.md`（f01）
- 支撑资产：`candidates/cases.md`、`candidates/counter-examples.md`、`candidates/principles.md`、`candidates/glossary.md`
- 原始文档：`docs/gateway/security/exposure-runbook.md`
