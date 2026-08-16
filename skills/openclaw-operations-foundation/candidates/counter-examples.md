# Cangjie Phase 1 — 反例候选（Counter-Examples）

- 冻结输入：OpenClaw Operations Foundation Phase 0 corpus
- P0 commit：`063ce57caf233dc3cac124349ccddf6cef9ca432`
- 范围：Batch A — Security、Upgrade、Gateway operations
- 生成时间：`2026-08-16T16:45+08:00`
- 说明：本文件只收录冻结语料中作者明确说明的误用、禁止、失败闭合或恢复陷阱。引文与行号均来自 `PHASE0_CORPUS.md` 已核验原文；不是生产变更指令。

---

## CE-01：把沙箱当作完整安全边界

- **分类**：Security / Gateway
- **来源**：`docs/gateway/sandboxing.md`，`PHASE0_CORPUS.md:1235-1249`
- **常见误用**：认为启用沙箱就隔离了 Gateway、所有插件与所有工具执行路径。
- **证据摘录**：
  - `Sandboxing is off by default and controlled by agents.defaults.sandbox ... The Gateway process always stays on the host; only tool execution moves into the sandbox when enabled.`（1235）
  - `This is not a perfect security boundary, but it materially limits filesystem and process access when the model does something dumb.`（1238）
  - `Not sandboxed: The Gateway process itself. Any tool explicitly allowed to run outside the sandbox via tools.elevated.`（1246-1249）
- **正确理解**：沙箱用于降低工具执行的爆炸半径，默认关闭；Gateway 进程不进入沙箱，`tools.elevated` 明确允许的工具也会绕过沙箱。
- **验证条件**：确认实际 sandbox mode、scope、backend 与工具策略；对高影响工具单独确认其是否被 `tools.elevated` 允许。
- **限制**：不可把沙箱作为敌对多租户或宿主进程隔离的充分证明。

---

## CE-02：把 SecretRef 当作进程隔离或磁盘明文清理

- **分类**：Security / Secrets
- **来源**：`docs/gateway/secrets.md`，`PHASE0_CORPUS.md:1825-1827`、`1857-1872`
- **常见误用**：迁移一个配置字段到 SecretRef 后，认为 agent 可读路径中的旧明文、备份与复制配置不再构成凭据风险。
- **证据摘录**：
  - `Plaintext credentials remain agent-readable when they sit in files the agent can inspect, including openclaw.json, .env, retired auth-profile JSON archives, or generated agents/*/agent/models.json files.`（1825-1827）
  - `SecretRefs stop credentials from being persisted in config and generated model files, but they are not a process-isolation boundary. A plaintext credential left on disk in a path the agent can read is still readable via file or shell tools, bypassing API-level redaction.`（1859）
  - `SecretRefs do not make arbitrary readable files safe. Backups, copied configs, old generated model catalogs, and unsupported credential classes stay production secrets until deleted, moved outside the agent trust boundary, or isolated separately.`（1871）
- **正确理解**：SecretRef 不会改变 agent 对可达明文文件的读取能力，也不会自动清理历史残留。
- **验证条件**：文档将迁移完成的条件包括 `openclaw secrets audit --check` 清洁，以及残留或不支持的凭据已受到 OS/container isolation 或外部凭据代理保护。（1861-1868）
- **限制**：Sentinel 与 SecretRef 降低配置/调用链暴露，不等同于进程隔离。

---

## CE-03：依靠 `gateway.remote.*` 保护本地 WebSocket 接入

- **分类**：Security / Gateway authentication
- **来源**：`docs/gateway/authentication.md`，`PHASE0_CORPUS.md:3762-3774`
- **常见误用**：把 `gateway.remote.token` 或 `gateway.remote.password` 当成对本地 WebSocket 接入面的保护配置，或期待未解析的 `gateway.auth.*` SecretRef 自动回退至 remote 凭据。
- **证据摘录**：
  - `Gateway auth is required by default - with no valid auth path configured, the Gateway refuses WebSocket connections (fail-closed).`（3764）
  - ``gateway.remote.token` and `gateway.remote.password` are client credential sources - they do not protect local WS access by themselves.`（3773）
  - `If gateway.auth.token or gateway.auth.password is explicitly configured via SecretRef and unresolved, resolution fails closed (no remote-fallback masking).`（3773）
- **正确理解**：Gateway 默认要求有效认证路径；`gateway.remote.*` 是客户端凭据来源而非本地 WebSocket 保护机制，显式配置但未解析的本地认证 SecretRef 不会被 remote fallback 掩盖。
- **验证条件**：无有效认证路径的 WebSocket 连接被拒绝；使用有效本地认证后连接才可建立。
- **限制**：不能以客户端配置存在替代 Gateway 的实际认证配置。

---

## CE-04：把反向代理的转发头处理成“追加”而非覆盖

- **分类**：Security / Gateway reverse proxy
- **来源**：`docs/gateway/authentication.md`，`PHASE0_CORPUS.md:3802-3829`
- **常见误用**：在 `trustedProxies` 配置下保留或追加客户端传入的 `X-Forwarded-For` / `X-Real-IP`，或误以为同机 loopback proxy 自动符合 `trusted-proxy` auth。
- **证据摘录**：
  - `trusted-proxy ... fails closed on loopback-source proxies by default.`（3806）
  - `Ensure your proxy overwrites X-Forwarded-For/X-Real-IP rather than appending to them:`（3818）
  - `# bad: preserves/appends untrusted client-supplied values`，其例子为 `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`（3825-3827）
- **正确理解**：反向代理应覆盖而不是追加转发身份头；同机 loopback proxy 只有在显式设置 `gateway.auth.trustedProxy.allowLoopback = true` 时才能满足 `trusted-proxy` auth。（3804-3806）
- **验证条件**：仅来自精确 `trustedProxies` 地址的代理头参与客户端 IP 判定；代理配置不保留客户端提交的转发头。
- **限制**：可信代理配置不自动授信 node device pairing。（3829）

---

## CE-05：在沙箱中期望默认网络仍能安装包，或以高风险网络规避限制

- **分类**：Security / Gateway sandboxing
- **来源**：`docs/gateway/sandboxing.md`，`PHASE0_CORPUS.md:1728-1731`、`1750-1759`
- **常见误用**：在默认 Docker sandbox 网络中执行包安装；或把 `network: "host"`、`network: "container:<id>"` 当作常规修复手段。
- **证据摘录**：
  - `network: "host" is blocked.`（1729）
  - `network: "container:<id>" is blocked by default (namespace join bypass risk).`（1730）
  - `Default docker.network is "none" (no egress), so package installs will fail.`（1751）
  - `docker.network: "container:<id>" requires dangerouslyAllowContainerNamespaceJoin: true and is break-glass only.`（1752）
- **正确理解**：默认无 egress，因此包安装失败是预期行为；host 网络被阻止，container namespace join 是显式 break-glass 例外而不是默认网络模式。
- **验证条件**：需要依赖时优先检查自定义镜像、`setupCommand` 所需权限与实际网络策略，而不是假设默认网络可用。
- **限制**：容器环境变量可被有容器引擎访问权的人用 `docker inspect` 或 `podman inspect` 查看。（1758-1759）

---

## CE-06：将 `openclaw.json` 作为 symlink 并期待原子写入穿透目标

- **分类**：Gateway / Configuration
- **来源**：`docs/gateway/configuration.md`，`PHASE0_CORPUS.md:19223-19225`
- **常见误用**：使用 symlink 作为 active `openclaw.json`，并认为 OpenClaw 自有写入会跟随该链接写入目标文件。
- **证据摘录**：
  - `The active config path must be a regular file.`（19225）
  - `OpenClaw-owned writes replace it atomically (rename onto the path), so a symlinked openclaw.json gets its target replaced rather than written through - avoid symlinked config layouts.`（19225）
- **正确理解**：active config 必须是常规文件；原子 rename 会在该路径上替换，而非穿透 symlink。非默认 state directory 的真实文件应由 `OPENCLAW_CONFIG_PATH` 直接指向。（19225）
- **验证条件**：在写配置或自动修复前，确认 active config 是 regular file 且所用路径为真实目标。
- **限制**：symlink 配置布局并非安全或可预测的 OpenClaw-owned write 布局。

---

## CE-07：把 schema 不匹配配置当作可在 Gateway 启动后修复的警告

- **分类**：Gateway / Configuration
- **来源**：`docs/gateway/configuration.md`，`PHASE0_CORPUS.md:19291-19302`
- **常见误用**：加入未知配置键、错误类型或无效值，并期待 Gateway 先启动再由运行时自动容错。
- **证据摘录**：
  - `OpenClaw only accepts configurations that fully match the schema.`（19294）
  - `Unknown keys, malformed types, or invalid values cause the Gateway to refuse to start.`（19294）
  - `The only root-level exception is $schema (string).`（19294）
- **正确理解**：配置是启动前契约；未知键、类型错误和无效值会阻止启动，根级 `$schema` 字符串是唯一所述例外。
- **验证条件**：编辑前使用 `config.schema.lookup` 查询字段级文档，或用 `openclaw config schema` 取得规范 schema。（19237-19240、19297-19302）
- **限制**：UI 和 runtime plugin/channel schema 可合并；不能以静态片段代替目标运行时的实际 schema。（19301-19302）

---

## CE-08：更新失败后立即恢复 pre-update state

- **分类**：Upgrade / Recovery
- **来源**：`docs/install/updating.md`，`PHASE0_CORPUS.md:8282-8336`
- **常见误用**：更新故障后直接恢复预更新备份，跳过保持当前 state 的代码回滚。
- **证据摘录**：
  - `Rollback has two layers: 1. Reinstall older OpenClaw code while keeping the current state. 2. Restore pre-update state only when the older code cannot use a migrated config or database.`（8284-8288）
  - `Start with a code-only rollback. Restoring state discards changes made after the backup.`（8290-8291）
  - `openclaw update preserves an automatic pre-update config copy, but it does not create a full state recovery point.`（8295-8297）
- **正确理解**：优先代码回滚；只有旧代码确实无法使用已迁移 config/database 时才恢复 pre-update state，且恢复会丢弃备份之后的 state 变更。
- **验证条件**：重大更新前创建经验证备份：`openclaw backup create --output ~/Backups/openclaw --verify`。（8293-8308）降级前先运行 `openclaw update --tag <known-good-version> --dry-run`。（8316-8322）
- **限制**：成功替换后若 Gateway health 失败，文档说明会报告旧版本与手动回滚说明，而不会自动再次替换 package。（8332-8336）

---

## CE-09：将 archive restore 视为原地、无副作用的激活

- **分类**：Upgrade / Backup recovery
- **来源**：`docs/install/backups.md`，`PHASE0_CORPUS.md:7800-7809`、`7817-7835`、`7859-7879`
- **常见误用**：对运行中的 Gateway 直接恢复 archive，或认为只复制 `openclaw.json`、恢复旧 channel state 后即可无条件继续运行。
- **证据摘录**：
  - `Stop the Gateway before taking a machine-move snapshot. A raw copy of a changing SQLite database can capture mismatched database and WAL files.`（7806-7808）
  - `Restore never activates in place.`（7826）
  - `Restoring older channel state can desynchronize ratcheting credentials such as WhatsApp. Approvals and delivery/dedupe state also roll back, and plugin node_modules trees must be reinstalled.`（7831-7834）
  - `The config file alone is not enough ... Always migrate the entire state directory.`（7866-7868）
- **正确理解**：恢复先落在新 staging target；停止 Gateway 后，依据 `manifest.json` 映射移动 state/workspace assets 或显式设置 `OPENCLAW_STATE_DIR`，并确认服务用户的 ownership。（7823-7829）
- **验证条件**：运行 `openclaw doctor`、`openclaw gateway restart`、`openclaw status`，并核验 Gateway、channels、dashboard/sessions 与 workspace files。（7839-7846、7883-7890）
- **限制**：备份含 auth profile、channel credentials 与 provider state；应加密存储、避免不安全传输，若怀疑泄露则轮换 keys。（7878-7879）

---

## CE-10：用 `OPENCLAW_ALLOW_MULTI_GATEWAY=1` 共享 mutable state 或忽略端口/锁冲突

- **分类**：Gateway / Operations
- **来源**：`docs/gateway/gateway-lock.md`，`PHASE0_CORPUS.md:21346-21403`
- **常见误用**：认为 `OPENCLAW_ALLOW_MULTI_GATEWAY=1` 允许多个 Gateway 使用同一 state directory，或把 `EADDRINUSE` 当作可立即并行重试的无害问题。
- **证据摘录**：
  - `Only one gateway process should own a state directory; run additional gateways with isolated profiles, state directories, configs, and ports.`（21348）
  - `Every Gateway participates, including Gateways started with OPENCLAW_ALLOW_MULTI_GATEWAY=1, so destructive SQLite maintenance cannot race a live owner.`（21356）
  - `On EADDRINUSE, startup retries the bind for up to 20 attempts at 500ms intervals (roughly 10 seconds total) to ride out a TIME_WAIT window after a recently exited process.`（21378-21379）
  - `OPENCLAW_ALLOW_MULTI_GATEWAY=1 permits multiple config/runtime instances, not shared mutable state. Each instance still needs a unique OPENCLAW_STATE_DIR.`（21401）
- **正确理解**：多实例模式不取消 state ownership lock；每个实例必须有独立 profile、state directory、config 与 port。端口冲突在内置限时重试后仍存在会失败。（21378-21389）
- **验证条件**：使用独立 `OPENCLAW_STATE_DIR` 和端口启动额外实例；确认无 `GatewayLockError`、`EADDRINUSE` 或 active lock。
- **限制**：若端口被非 Gateway 进程占用，错误相同；应释放端口或以 `openclaw gateway --port <port>` 选择其他端口。（21400）

---

## 来源映射

| 反例 | 主来源 | 语料行段 | 范围 |
| --- | --- | --- | --- |
| CE-01 | `docs/gateway/sandboxing.md` | 1235-1249 | Security / Gateway |
| CE-02 | `docs/gateway/secrets.md` | 1825-1827、1857-1872 | Security |
| CE-03 | `docs/gateway/authentication.md` | 3762-3774 | Security / Gateway |
| CE-04 | `docs/gateway/authentication.md` | 3802-3829 | Security / Gateway |
| CE-05 | `docs/gateway/sandboxing.md` | 1728-1731、1750-1759 | Security / Gateway |
| CE-06 | `docs/gateway/configuration.md` | 19223-19225 | Gateway |
| CE-07 | `docs/gateway/configuration.md` | 19291-19302 | Gateway |
| CE-08 | `docs/install/updating.md` | 8282-8336 | Upgrade |
| CE-09 | `docs/install/backups.md` | 7800-7809、7817-7835、7859-7879 | Upgrade |
| CE-10 | `docs/gateway/gateway-lock.md` | 21346-21403 | Gateway |
