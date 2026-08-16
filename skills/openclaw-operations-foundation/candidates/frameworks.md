# Frameworks — Batch A（Security、Upgrade、Gateway）

> 阶段 1 候选，仅来自冻结 P0 语料；版本锚点：commit `063ce57caf233dc3cac124349ccddf6cef9ca432`，release `v2026.7.1-2`。
> 本文件保留可迁移的决策/推理结构；原则、案例、反例与术语由其他 extractor 独立处理。

- id: f01
  title: 最小满足暴露面选择
  type: framework
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    "Prefer the narrowest pattern that satisfies the workflow."
  summary: |
    对网络暴露方案按最小攻击面排序选择：先保持 loopback，再考虑 SSH tunnel、Tailscale Serve、受限 tailnet/LAN、受信反向代理；公网暴露为罕见高风险选项。
    当业务需求要求扩大暴露时，不以“能连通”为完成条件，而是为新增表面配套认证、网络过滤、TLS、速率限制、allowlist 与隔离控制。
  tags: [security, exposure, network, decision]

- id: f02
  title: 单控制逐步放宽与逐步验证
  type: framework
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Widen one control at a time: add a specific channel allowlist before enabling
  summary: |
    将高风险暴露变更拆为单一控制变量的序列，而非一次同时修改 bind、代理、渠道策略和工具权限。
    每一步后验证授权成功、未授权拒绝、路由正确、敏感数据已脱敏及高影响工具被拒绝或要求批准；当前变更未理解前不继续下一步。
  tags: [security, change-management, validation, incremental]

- id: f03
  title: 身份优先、范围其次、模型最后
  type: framework
  source_chapter: docs/gateway/security/index.md
  source_quote: |
    1. **Identity first** - decide who can talk to the bot (DM pairing / allowlists / explicit "open").
  summary: |
    安全决策按三层顺序进行：先控制触发者身份，再限制可作用的资源与工具，最后把模型视作可能被操纵的组件。
    该顺序避免以提示词或模型行为替代身份认证和最小权限；模型防护只能补强，不能成为唯一控制面。
  tags: [security, identity, scope, model, defense-in-depth]

- id: f04
  title: 信任边界拆分判定
  type: framework
  source_chapter: docs/gateway/security/index.md
  source_quote: |
    OpenClaw is not a hostile multi-tenant security boundary for multiple
  summary: |
    先判断参与者是否处于同一受信操作员边界。若不是，不把 session 标签、共享 Gateway 认证或提示词隔离误当作租户隔离。
    对混合信任或对抗性用户，将 Gateway、凭证，并尽可能将 OS 用户或主机拆分；把共享 agent 的工具权限视为共享的委托权力。
  tags: [security, trust-boundary, isolation, multi-tenant]

- id: f05
  title: 工具执行三层控制模型
  type: framework
  source_chapter: docs/gateway/sandbox-vs-tool-policy-vs-elevated.md
  source_quote: |
    1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.entries.*.sandbox.*`)
  summary: |
    分析工具执行风险或“工具为何被阻止”时，分别检查运行位置、工具可用性及 exec 的沙箱逃逸，而不将三者混为一个权限开关。
    先以实际生效配置确认 sandbox mode/scope/workspace access、allow/deny 与 elevated gates，再决定最小范围的修复；允许某个工具不等于它在宿主执行，开启 elevated 也不增加其他工具权限。
  tags: [security, sandbox, tool-policy, elevated, execution]

- id: f06
  title: 两级审计证据模型
  type: framework
  source_chapter: docs/cli/security.md
  source_quote: |
    Plain `security audit` stays on the cold config/filesystem/read-only path:
  summary: |
    将安全结论分为静态冷路径证据和带运行时探测的深度证据：前者适合常规、低副作用配置/文件检查，后者用于暴露或重大变更后的 live 验证。
    不能将任一层单独误读为完整安全证明；深度探测是 best-effort，静态审计也不会加载每个插件运行时。
  tags: [security, audit, evidence, validation]

- id: f07
  title: 可恢复升级与双层回退
  type: framework
  source_chapter: docs/install/updating.md
  source_quote: |
    Rollback has two layers:
  summary: |
    将升级看作需要恢复路径的变更，而不是单纯的软件安装。先准备经验证的恢复点，发生问题时优先回退代码并保留当前状态。
    只有旧代码不能读取迁移后的配置或数据库时才恢复预更新状态；状态恢复是“时间旅行”，会丢弃备份后的变更，必须保存当前状态并显式验证恢复后的服务。
  tags: [upgrade, rollback, recovery, state-management]

- id: f08
  title: 先验证候选、后激活的恢复模型
  type: framework
  source_chapter: docs/install/backups.md
  source_quote: |
    "Restore is deliberately explicit; nothing overwrites live state in place."
  summary: |
    恢复流程将“验证/提取候选状态”和“激活为生产状态”分离：先将归档或数据库恢复到新的 staging 目标，验证结构、清单与 SQLite 完整性，再在 Gateway 停止时由操作员显式切换状态并执行 Doctor/健康检查。
    此模型避免未验证或不可信输入直接覆盖 live state，也保留了在激活前检查凭证、渠道状态与路径映射的窗口。
  tags: [backup, restore, staging, verification, recovery]

- id: f09
  title: 在线状态的备份路径选择
  type: framework
  source_chapter: docs/install/backups.md
  source_quote: |
    Never copy live `.sqlite`, `-wal`, `-shm`, or `-journal` files as a backup.
  summary: |
    先按状态是否在线和恢复目标选择备份机制，而非把文件复制等同于备份：便携完整恢复用 archive，单库紧凑可验证恢复用 SQLite snapshot，版本化增量用 Git backup，低 RPO 用连续复制。
    对在线数据库只采用捕获已提交状态的支持路径；同时把备份视为敏感状态副本，按 live state 的加密、权限与泄露轮换要求管理。
  tags: [backup, sqlite, data-integrity, recovery, security]

- id: f10
  title: 受监督更新的进程外交接
  type: framework
  source_chapter: docs/install/updating.md
  source_quote: |
    Package-manager updates requested through the live Gateway control-plane
  summary: |
    对正在提供服务的进程，更新程序不应在其自身运行中替换正在加载的包树。将替换交给进程外、可观察的交接流程，再进行服务元数据刷新、重启及版本/可达性核验。
    若无法安全交接，返回可审查的人工命令而非强行进程内更新；这把“包已安装”与“服务已在新版本健康运行”区分为不同完成条件。
  tags: [upgrade, gateway, handoff, supervision, verification]

## Extraction Receipt

- extractor: framework-extractor
- phase: 1
- source scope: Batch A — Security, Upgrade, Gateway operations
- items: 10
- quote rule: each quote is under 100 English words
- write scope: this candidate file only
