# INDEX — RIA-TV++ 技能索引

> 阶段 2 生成产物；阶段 3 链接关系为初稿，待独立审阅与压力测试后定稿。

| ID | Skill | 框架 |
| --- | --- | --- |
| f01 | `minimal-exposure-selection` | 最小满足暴露面选择 |
| f02 | `single-control-widen` | 单控制逐步放宽与逐步验证 |
| f03 | `identity-first-scope-second-model-last` | 身份优先、范围其次、模型最后 |
| f04 | `trust-boundary-segmentation` | 信任边界拆分判定 |
| f05 | `tool-execution-three-layer-control` | 工具执行三层控制模型 |
| f06 | `two-tier-audit-evidence` | 两级审计证据模型 |
| f07 | `recoverable-upgrade-two-layer-rollback` | 可恢复升级与双层回退 |
| f08 | `candidate-validation-before-activation` | 先验证候选、后激活的恢复模型 |
| f09 | `online-state-backup-path-selection` | 在线状态的备份路径选择 |
| f10 | `supervised-update-handoff` | 受监督更新的进程外交接 |

## 路由与交接关系（阶段 3 审阅后）

- 暴露决策：先用 `minimal-exposure-selection` 选择模式；需要按控制面分步实施时转 `single-control-widen`；需要界定验证证据强度时转 `two-tier-audit-evidence`。
- 安全控制：入口控制的优先级由 `identity-first-scope-second-model-last` 主导；工具执行层问题转 `tool-execution-three-layer-control`；多个不受信主体的共用环境问题转 `trust-boundary-segmentation`。
- 恢复更新：在线恢复点选择用 `online-state-backup-path-selection`；恢复候选的验证与生产激活用 `candidate-validation-before-activation`；升级故障的代码/state 回退用 `recoverable-upgrade-two-layer-rollback`；运行服务的替换交接用 `supervised-update-handoff`。
