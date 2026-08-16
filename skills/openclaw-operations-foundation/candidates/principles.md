# Principles — Batch A（Security、Upgrade、Gateway）

> 阶段 1 候选，仅来自冻结 P0 语料；版本锚点：commit `063ce57caf233dc3cac124349ccddf6cef9ca432`，release `v2026.7.1-2`。
> 本文件保留可直接应用的原子规则与清单；框架、案例、反例与术语由其他候选文件独立处理。

- id: p01
  title: 暴露前可解释访问与权限边界
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Expose the Gateway only after you can explain who can reach it, how they are
  summary: |
    扩大 Gateway 暴露前，操作员必须能够说明谁可访问、如何认证、能触发哪些 agent，以及这些 agent 可用哪些工具。无法说明时，退回仅回环访问并重新审计。
  tags: [security, exposure, pre-flight, identity, tools]

- id: p02
  title: 选择满足工作流的最窄暴露模式
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    "Prefer the narrowest pattern that satisfies the workflow."
  summary: |
    从 loopback、SSH tunnel、受限 tailnet/LAN 到受信反向代理逐级选择；避免直接公网端口转发。必须公网访问时，以身份感知代理作为唯一网络路径。
  tags: [security, exposure, network, least-privilege]

- id: p03
  title: 单控制面逐步放宽
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Widen one control at a time: add a specific channel allowlist before enabling
  summary: |
    高风险暴露变更必须拆分为单一控制变量的序列，而不是同时改变 bind、代理、渠道与工具权限。每一项变更被理解并验证前，不进入下一项。
  tags: [security, change-management, incremental, validation]

- id: p04
  title: 暴露前先解决关键审计发现
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    "Resolve critical findings first."
  summary: |
    开放访问前运行 doctor、安全审计、深度审计与健康检查。critical 发现必须先解决；warning 仅可在部署意图明确且已记录时接受。
  tags: [security, audit, pre-flight, health-check]

- id: p05
  title: 消息入口优先配对或严格允许列表
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Prefer `dmPolicy: "pairing"` or a strict `allowFrom` list over `dmPolicy: "open"`.
  summary: |
    将 DM 和群组视为不受信任输入面。优先采用 pairing 或严格 allowFrom，而不是开放 DM；开放或半开放渠道仅可路由到最小工具、无个人凭据的 agent。
  tags: [security, messaging, allowlist, dm]

- id: p06
  title: 禁止通配允许列表叠加宽工具权限
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Do not combine `"*"` allowlists with broad tool access.
  summary: |
    任何入口允许任意发送者时，不能同时授予宽泛工具能力；否则不受信任输入可直接触及高影响能力。
  tags: [security, allowlist, tools, prompt-injection]

- id: p07
  title: 多人 DM 使用按渠道和对端隔离的会话范围
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    Set `session.dmScope: "per-channel-peer"`
  summary: |
    多人可向机器人发 DM 时，使用 per-channel-peer；多账户渠道使用 per-account-channel-peer，避免不同 DM 共享上下文。该措施是收件箱加固，不构成敌对用户隔离。
  tags: [security, session, dm, context-isolation]

- id: p08
  title: 远程 CLI 校验显式提供凭据
  type: principle
  source_chapter: docs/gateway/security/exposure-runbook.md
  source_quote: |
    "Do not assume local config credentials apply to an explicit remote URL."
  summary: |
    使用 gateway probe 验证显式远程 URL 时，显式传入目标所需凭据；不假定本地配置凭据自动适用于该 URL。
  tags: [security, cli, authentication, remote]

- id: p09
  title: Gateway 认证默认失败闭合
  type: principle
  source_chapter: docs/gateway/security/index.md
  source_quote: |
    Gateway auth is required by default - with no valid auth path configured, the Gateway refuses WebSocket connections (fail-closed).
  summary: |
    Gateway 默认要求有效认证路径；未配置有效路径时拒绝 WebSocket 连接。客户端 remote 凭据来源不能替代 Gateway 本地认证配置。
  tags: [security, authentication, fail-closed, gateway]

- id: p10
  title: 代理覆盖而非追加客户端转发头
  type: principle
  source_chapter: docs/gateway/security/index.md
  source_quote: |
    Ensure your proxy **overwrites** `X-Forwarded-For`/`X-Real-IP` rather than appending to them:
  summary: |
    配置 trustedProxies 时，反向代理必须覆盖而不是追加转发身份头，防止保留客户端伪造的来源信息。
  tags: [security, reverse-proxy, headers, trusted-proxy]

- id: p11
  title: 配置必须完整匹配 schema
  type: principle
  source_chapter: docs/gateway/configuration.md
  source_quote: |
    "OpenClaw only accepts configurations that fully match the schema."
  summary: |
    配置是启动前契约：未知键、错误类型或无效值会使 Gateway 拒绝启动。编辑前按当前运行时 schema 校验字段与值。
  tags: [gateway, configuration, schema, startup]

- id: p12
  title: 活跃配置路径必须为常规文件
  type: principle
  source_chapter: docs/gateway/configuration.md
  source_quote: |
    "The active config path must be a regular file."
  summary: |
    避免 symlink 化的 active openclaw.json；OpenClaw 自有原子写会 rename 到路径本身，而非穿透链接写入目标。
  tags: [gateway, configuration, filesystem, atomic-write]

- id: p13
  title: 手动更新受管安装前停止 Gateway
  type: principle
  source_chapter: docs/install/updating.md
  source_quote: |
    install, stop the managed Gateway first. Package managers replace files in
  summary: |
    手动使用包管理器替换受管安装前停止 Gateway，避免运行中进程在文件替换期间加载 core 或 plugin 文件；替换结束后再重启。
  tags: [upgrade, manual-update, service, package-manager]

- id: p14
  title: 重大更新前创建经验证备份
  type: principle
  source_chapter: docs/install/updating.md
  source_quote: |
    create a full state recovery point. Before a significant update, create one
  summary: |
    openclaw update 保留的自动 pre-update config 副本不是完整状态恢复点。重大更新前显式创建并验证备份，按 live state 同等级保护其中的凭据和渠道状态。
  tags: [upgrade, backup, pre-flight, verification]

- id: p15
  title: 先执行仅代码回滚
  type: principle
  source_chapter: docs/install/updating.md
  source_quote: |
    Start with a code-only rollback. Restoring state discards changes made after
  summary: |
    回滚先重装旧代码并保留当前 state；仅在旧代码无法使用已迁移 config 或 database 时恢复 pre-update state，因为状态恢复会丢弃备份后的变更。
  tags: [upgrade, rollback, recovery, state]

- id: p16
  title: 恢复不进行原地激活
  type: principle
  source_chapter: docs/install/migrating.md
  source_quote: |
    Restore never activates in place.
  summary: |
    将归档恢复到新的 staging target；停止 Gateway 后再显式迁移资产或切换 state directory，并核验所有权和服务行为。
  tags: [backup, restore, staging, recovery]

- id: p17
  title: 多实例不得共享可变状态
  type: principle
  source_chapter: docs/gateway/gateway-lock.md
  source_quote: |
    Each instance still needs a unique `OPENCLAW_STATE_DIR`.
  summary: |
    多个 Gateway 实例分别使用独立 profile、state directory、config 与端口。OPENCLAW_ALLOW_MULTI_GATEWAY=1 不授权共享 mutable state。
  tags: [gateway, multi-instance, state, isolation, lock]

- id: p18
  title: 沙箱不是完美安全边界
  type: principle
  source_chapter: docs/gateway/sandboxing.md
  source_quote: |
    "This is not a perfect security boundary, but it materially limits filesystem and process access"
  summary: |
    沙箱用于缩小工具执行的文件系统和进程爆炸半径；Gateway 进程与被 elevated 明确放行的工具不在此边界内，不能把沙箱当作敌对多租户隔离证明。
  tags: [security, sandbox, blast-radius, limitation]

- id: p19
  title: SecretRef 不等于进程隔离
  type: principle
  source_chapter: docs/gateway/secrets.md
  source_quote: |
    "SecretRefs stop credentials from being persisted in config and generated model files, but they are not a process-isolation boundary."
  summary: |
    SecretRef 减少配置与生成模型文件中的凭据持久化，但不改变 agent 对可读明文文件的访问能力，也不会自动清理备份或旧配置残留。
  tags: [security, secrets, process-isolation, credential-hygiene]

- id: p20
  title: 出站代理目的地策略才是安全边界
  type: principle
  source_chapter: docs/security/network-proxy.md
  source_quote: |
    "The proxy's destination policy is the actual security boundary; OpenClaw cannot verify that your proxy blocks the right targets."
  summary: |
    出站代理的安全依赖其目的地策略，而不只是代理存在。策略应拒绝可绕过的 loopback、private、link-local、metadata 等目的地范围。
  tags: [security, egress, proxy, ssrf, destination-policy]

## Source map

| IDs | Primary source |
| --- | --- |
| p01–p08 | `docs/gateway/security/exposure-runbook.md` |
| p09–p10 | `docs/gateway/authentication.md` |
| p11–p12 | `docs/gateway/configuration.md` |
| p13–p15 | `docs/install/updating.md` |
| p16 | `docs/install/backups.md` |
| p17 | `docs/gateway/gateway-lock.md` |
| p18 | `docs/gateway/sandboxing.md` |
| p19 | `docs/gateway/secrets.md` |
| p20 | `docs/security/network-proxy.md` |
