# Cangjie Phase 1 — 可核验案例候选

- 冻结输入：OpenClaw Operations Foundation Phase 0 corpus
- P0 commit：`063ce57caf233dc3cac124349ccddf6cef9ca432`
- 范围：Batch A — Security、Upgrade、Gateway operations
- 生成时间：`2026-08-16T16:25+08:00`
- 说明：本文件只提取冻结语料中可由命令、配置状态或 Gateway 行为验证的案例；它不是生产变更指令。

---

## C-01：Gateway 远程暴露的预检、最小基线与回滚

- **分类**：Security / Gateway
- **来源**：`docs/gateway/security/exposure-runbook.md`，`PHASE0_CORPUS.md:2924-3149`
- **情境**：计划通过 LAN、tailnet、Tailscale Serve/Funnel 或反向代理让 Gateway 超出 loopback 可达。
- **证据摘录**：
  - `Expose the Gateway only after you can explain who can reach it, how they are authenticated, which agents they can trigger, and which tools those agents can use.`（2937-2941）
  - 远程开放前运行 `openclaw doctor`、`openclaw security audit`、`openclaw security audit --deep`、`openclaw health`。（2978-2987）
  - 最小安全基线包括 `gateway.bind: "loopback"`、token 认证、`session.dmScope: "per-channel-peer"`、非主会话沙箱、`tools.exec.security: "deny"` 和关闭 elevated tools。（3001-3028）
- **操作要点**：采用满足需求的最窄暴露模式；每次只扩大一个控制面，并在开放非本地访问前完成预检。
- **验证条件**：
  1. `openclaw security audit --deep` 已运行，critical 发现先于暴露变更解决。
  2. 授权连接成功，未授权 sender 或浏览器会话被拒绝。
  3. DM/group 路由仅到预期 agent；高影响工具要求审批或被拒绝；日志不泄露 secrets。（3091-3104）
- **回滚**：恢复 `gateway.bind: "loopback"`；将列出的 channel `dmPolicy` 设为 `"disabled"`；设定 `tools.exec.security: "deny"` 和 `tools.elevated.enabled: false`；停止公开转发/Funnel/代理路由，轮换 Gateway 与受影响集成凭据，再重新运行 `openclaw security audit --deep`。（3106-3135）

---

## C-02：用 Security Audit 分诊远程绑定、开放消息面和高危工具组合

- **分类**：Security
- **来源**：`docs/gateway/security/audit-checks.md`，`PHASE0_CORPUS.md:2715-2870`
- **情境**：需要将安全审计发现映射到明确的配置修复点，并区分普通审计与 `--deep` 检查。
- **证据摘录**：
  - `openclaw security audit` 以 `checkId` 给出结构化 findings；plugin/skill 扫描和 live Gateway probe 仅在 `--deep` 运行。（2727-2734）
  - `gateway.bind_no_auth` 是 critical：远程 bind 没有 shared secret；修复键为 `gateway.bind`, `gateway.auth.*`。（2761）
  - `security.exposure.open_groups_with_elevated` 是 critical：开放 DM/group 与 elevated tools 结合形成高影响 prompt-injection 路径。（2844-2847）
  - `hooks.token_reuse_gateway_token` 是 critical：hook ingress token 同时解锁 Gateway auth。（2790）
- **操作要点**：运行常规审计获取静态配置/文件系统结论；当需要插件/技能代码扫描或 live Gateway 连通性检查时运行 `openclaw security audit --deep`。按 finding 的 `Primary fix key/path` 定位修复点。
- **验证条件**：复跑同一审计范围后，相关 `checkId` 不再处于 active findings；对于 `--deep` 专属检查，保留实际 probe/scan 成功或受阻的输出证据。
- **回滚/限制**：本案例不定义自动修复。目录明确将多数高影响配置标为无 auto-fix；撤销需恢复对应配置键并重跑审计验证。（2741-2860）

---

## C-03：受监管的 Gateway 更新与升级后验证

- **分类**：Upgrade / Gateway
- **来源**：`docs/install/updating.md`，`PHASE0_CORPUS.md:7905-8108`、`8256-8280`
- **情境**：对 npm、pnpm、Bun 或 git 安装执行标准更新，同时协调已安装的 Gateway 服务。
- **证据摘录**：
  - `openclaw update` 检测安装类型、获取最新版本、运行 `openclaw doctor` 并重启 Gateway。（7923-7929）
  - 对已安装 Gateway，更新会刷新服务元数据并重启，除非使用 `--no-restart`。（7998-8003）
  - 手动更新 supervised install 前应停止 managed Gateway，避免运行中 Gateway 在包替换过程中加载 core 或 plugin 文件。（8062-8073）
  - 更新后流程为运行 `openclaw doctor`、`openclaw gateway restart`、`openclaw health`。（8256-8280）
- **操作要点**：先用 `openclaw update --dry-run` 预览；执行更新后通过 doctor、restart 与 health 完成升级后检查。需手动包管理器替换时先停止 Gateway。
- **验证条件**：更新后的 Gateway 完成 restart，`openclaw doctor` 和 `openclaw health` 已实际执行并保留结果；对于 root-owned global install，文档给出的附加检查包含 `openclaw --version`、`/readyz`、`openclaw plugins list --json`、`openclaw gateway status --deep --json` 和 `openclaw doctor --lint --json`。（8075-8096）
- **回滚**：使用 C-04 的代码优先回滚路径，而不是在运行中的 Gateway 上直接替换包。

---

## C-04：更新失败后的双层回滚和已验证备份

- **分类**：Upgrade
- **来源**：`docs/install/updating.md`，`PHASE0_CORPUS.md:8282-8363`
- **情境**：更新后旧代码无法兼容迁移过的 config 或数据库，或新版本 Gateway 健康检查失败。
- **证据摘录**：
  - 回滚分为两层：先重装旧代码并保留 current state；只有旧代码不能使用迁移后 config/database 时才恢复 pre-update state。（8282-8292）
  - 重大更新前创建经验证的备份：`openclaw backup create --output ~/Backups/openclaw --verify`。（8293-8313）
  - 优先使用 `openclaw update --tag <known-good-version>`，它会识别降级、请求确认、执行 plugin convergence/compatibility checks、刷新服务元数据、重启并验证运行版本。（8314-8336）
- **操作要点**：先创建/确认可用的 verified backup；优先实行代码回滚，只有兼容性明确要求时再恢复状态。
- **验证条件**：
  1. 降级前 `openclaw update --tag <known-good-version> --dry-run` 已确认目标。
  2. 降级后 Gateway 已重启并验证运行版本。
  3. 若进入状态恢复，确认其会丢弃备份后产生的状态更改。（8288-8292）
- **回滚实现**：包安装的首选路径是 `openclaw update --tag <known-good-version>`；CLI update 路径不可用时，停止 Gateway，以当前 Gateway 所属包管理器和安装范围安装已知良好版本，`openclaw gateway install --force` 后重启。（8338-8350）源检出则 checkout 已知良好 tag/commit，构建后重启。（8352-8363）

---

## C-05：节点设备配对与能力审批分层

- **分类**：Gateway / Security
- **来源**：`docs/gateway/pairing.md`，`PHASE0_CORPUS.md:25018-25267`
- **情境**：新 node 连接 Gateway，需要防止仅凭设备配对即暴露 node command surface。
- **证据摘录**：
  - Device pairing gate `connect` handshake；node capability approval gate 已连接 node 可暴露的 capabilities/commands。（25030-25039）
  - 新建或扩大的 surface 会存为 pending request；审批前 node commands 保持过滤。（25047-25055）
  - 从 `2026.3.31` 起，node commands 在 node pairing 被批准前禁用；device pairing 本身不再足够。（25154-25169）
  - 审批 scope 随声明命令升级：commandless 为 `operator.pairing`；ordinary commands 还需 `operator.write`；列出的 admin-sensitive commands 还需 `operator.admin`。（25124-25141）
- **操作要点**：通过 `openclaw nodes pending` 查看请求；用 `openclaw nodes approve <requestId>` 或 `openclaw nodes reject <requestId>` 处理；以 `openclaw nodes status` 查看 paired/connected nodes 与 capabilities。（25083-25094）
- **验证条件**：首次连接产生 pending request；批准前声明命令不执行；批准后 capability 仍受 `gateway.nodes.commands.allow` / `gateway.nodes.commands.deny` 的全局策略约束。（25143-25163）
- **回滚**：拒绝待处理请求，或使用 `openclaw nodes remove --node <id|name|ip>` 移除 paired node。移除会撤销 node role、丢弃批准的 node surface 并使对应 node-role sessions 失效/断开。（25085-25091、25104-25116）

---

## C-06：出站网络代理与 Gateway loopback 边界

- **分类**：Security / Gateway
- **来源**：`docs/security/network-proxy.md`，`PHASE0_CORPUS.md:5722-5943`
- **情境**：为 runtime HTTP/WebSocket egress 设置 operator-managed forward proxy，以增强 SSRF、DNS rebinding 和目的地控制。
- **证据摘录**：
  - 代理是在网络边界提供 central egress control、stronger SSRF protection 和 destination auditability 的可选 defense-in-depth；它在 DNS resolution 后、实际连接前评估目的地。（5729-5733）
  - `proxy.loopbackMode` 可取 `gateway-only`、`proxy`、`block`；默认 `gateway-only` 对精确配置的 Gateway loopback authority 建立 direct-connect exception。（5807-5827）
  - 容器目标命令中，`127.0.0.1` 指向容器自身；除非显式设置 `OPENCLAW_CONTAINER_ALLOW_LOOPBACK_PROXY_URL=1`，OpenClaw 拒绝 container-targeted command 的 loopback proxy URL。（5831）
  - 代理目的地 policy 才是实际 security boundary；应限制其监听面，并拒绝 loopback、private、link-local、metadata、multicast、reserved 和 documentation ranges 的目标绕过。（5843-5850）
- **操作要点**：明确选择 loopback mode，实施并审查 proxy destination policy；针对容器网络语义使用容器可达的 proxy URL。
- **验证条件**：运行 `openclaw proxy validate` 的默认检查时，公开 `https://example.com/` 成功，临时 loopback canary 不可经 proxy 到达；命令在任一验证失败时以退出码 `1` 结束。（5877）
- **回滚/限制**：移除或替换受管 proxy URL 后重新验证 runtime egress；不得将“OpenClaw 未检查、测试或认证 proxy policy”的事实误作安全保证，proxy policy 变更应作为 security-sensitive operational change。（5940-5943）

---

## 来源映射

| 案例 | 主来源 | 语料行段 | 范围 |
| --- | --- | --- | --- |
| C-01 | `docs/gateway/security/exposure-runbook.md` | 2924-3149 | Security / Gateway |
| C-02 | `docs/gateway/security/audit-checks.md` | 2715-2870 | Security |
| C-03 | `docs/install/updating.md` | 7905-8108、8256-8280 | Upgrade / Gateway |
| C-04 | `docs/install/updating.md` | 8282-8363 | Upgrade |
| C-05 | `docs/gateway/pairing.md` | 25018-25267 | Gateway / Security |
| C-06 | `docs/security/network-proxy.md` | 5722-5943 | Security / Gateway |
