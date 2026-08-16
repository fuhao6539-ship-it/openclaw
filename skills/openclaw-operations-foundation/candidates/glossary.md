# OpenClaw 运维术语表（候选）

> **范围**：Batch A（Security、Upgrade、Gateway operations）。
>
> **依据**：仅使用冻结来源 `PHASE0_CORPUS.md` 和 `BOOK_OVERVIEW.md`。`PHASE0_CORPUS.md` 行号为定位锚点；定义保持对原文的忠实概括，不替代原始文档。
>
> **语料版本**：仓库快照 `063ce57caf233dc3cac124349ccddf6cef9ca432`；Release 锚点 `v2026.7.1-2`。

---

## Security

| ID | 术语 | 定义 | Corpus 证据锚点 |
|---|---|---|---|
| S1 | 信任模型（trust model） | 默认是个人助理模型：一个 Gateway 对应一个受信操作员边界，而不是供相互对抗的多用户共享时使用的多租户隔离边界。多名不受信用户应拆分到独立 Gateway，理想情况下再拆分 OS 用户或主机。 | `PHASE0_CORPUS.md` L101, L3162–L3180 |
| S2 | 信任边界（trust boundary） | 在一个 Gateway 内，经认证的操作员是受信控制面角色，不是按用户隔离的租户角色；会话标签或 `sessionKey` 只负责路由，不承担授权功能。 | `PHASE0_CORPUS.md` L3175–L3178 |
| S3 | DM 配对（DM pairing） | 默认 `dmPolicy: "pairing"` 下，未知发送方取得配对码，获批前不能触发机器人；配对代码一小时后过期，每个渠道的待处理请求数受限。配对只批准发送方触发机器人，不形成独立主机安全边界。 | `PHASE0_CORPUS.md` L3039–L3054, L3311–L3329 |
| S4 | 设备配对（device pairing） | 设备配对记录是获批角色和作用域的持久来源。已配对设备若请求更大角色或作用域，会创建待处理升级请求，不会被静默扩权。 | `PHASE0_CORPUS.md` L977–L992 |
| S5 | 节点配对（node pairing） | 建立节点身份和信任，但不替代节点自身的 `system.run` 执行审批策略；危险或隐私敏感命令仍需持久的 `gateway.nodes.commands.allow` 条目。 | `PHASE0_CORPUS.md` L1045–L1053 |
| S6 | 共享密钥认证（shared-secret auth） | Gateway token 或 password 的共享密钥认证被当作该 Gateway 的受信操作员访问；特定 HTTP 面对 bearer 认证恢复完整默认操作员作用域。 | `PHASE0_CORPUS.md` L1055–L1060 |
| S7 | 受信反向代理认证（trusted-proxy auth） | Gateway 将认证委托给反向代理。代理须先认证用户，Gateway 只接受列于 `gateway.trustedProxies` 的代理源 IP；代理还必须剥离或覆写客户端提交的身份与转发头。 | `PHASE0_CORPUS.md` L3056–L3066, L4301–L4328 |
| S8 | 回环绑定（loopback bind） | `gateway.bind: "loopback"` 是个人使用、管理访问和调试的推荐基线。超出回环的网络暴露须配置认证、防火墙和相应访问控制。 | `PHASE0_CORPUS.md` L2952–L2956, L3001–L3028 |
| S9 | Tailscale Serve / Funnel | Serve 用于 tailnet 内访问；Funnel 代表公网暴露。通过 Serve 的 Tailscale 身份头只认证 Control UI WebSocket 面，不适用于其他认证路径。 | `PHASE0_CORPUS.md` L2952–L2956, L3203 |
| S10 | 安全审计（`openclaw security audit`） | 应在配置变更后或网络面暴露前执行；覆盖入站访问、工具爆炸半径、exec 策略漂移、网络和浏览器控制暴露、磁盘卫生及插件加载等检查。 | `PHASE0_CORPUS.md` L3184–L3206 |
| S11 | 沙箱（sandbox） | 面向暴露部署的最小安全基线将非主会话置于 `sandbox.mode: "non-main"`，以配合受限工具策略收缩执行爆炸半径。 | `PHASE0_CORPUS.md` L3001–L3028 |
| S12 | Exec 策略与审批 | `tools.exec.security: "deny"` 阻止所有 exec 调用；若要放宽，文档要求先明确发送方、代理、命令与审批模式，使其匹配威胁模型。 | `PHASE0_CORPUS.md` L3022–L3037 |
| S13 | Allowlist（允许列表） | 对 DM 和群组入口，优先选用 `dmPolicy: "pairing"` 或严格的 `allowFrom`，且不应把 `"*"` 通配允许列表与宽泛工具权限组合。 | `PHASE0_CORPUS.md` L3039–L3051 |
| S14 | DM 会话作用域（`session.dmScope`） | 多人可向机器人发送 DM 时，使用 `per-channel-peer`（多账户渠道可用 `per-account-channel-peer`）避免 DM 共用上下文；这属于共享收件箱加固，而不是对对抗用户的隔离。 | `PHASE0_CORPUS.md` L98–L102, L3047–L3049 |

## Upgrade

| ID | 术语 | 定义 | Corpus 证据锚点 |
|---|---|---|---|
| U1 | 更新通道（update channel） | `stable`、`extended-stable`、`beta`、`dev` 决定更新目标和安装流程；`extended-stable` 仅支持包安装，不支持 Git checkout。 | `PHASE0_CORPUS.md` L6313–L6321, L6501–L6508 |
| U2 | 更新预演（`--dry-run`） | 预览通道、标签、目标与重启流程，但不写配置、不安装、不同步插件，也不重启。 | `PHASE0_CORPUS.md` L6315–L6318 |
| U3 | 更新修复（`openclaw update repair`） | 用于核心包已变更、但后续插件同步、元数据、注册表或 Doctor 修复未收敛的情形。它不安装新核心包，也不重启 Gateway。 | `PHASE0_CORPUS.md` L6362–L6390 |
| U4 | Doctor（`openclaw doctor`） | 用于迁移、配置和更新后的诊断及安全修复；Git checkout 更新流程将其作为最终安全检查。 | `PHASE0_CORPUS.md` L115, L6501–L6540 |
| U5 | 降级与状态恢复 | 降级需要确认，因为旧版本可能无法兼容现有配置。已将会话迁移至 SQLite 时，启动旧的文件型版本前须恢复归档的旧版转录产物。 | `PHASE0_CORPUS.md` L6333–L6338 |
| U6 | 托管服务交接（managed-service handoff） | 由 Gateway 控制面触发的包管理器更新会交给 Gateway 进程外的脱离式 CLI 助手执行；只有服务重启后的健康检查完成，更新才可报告完成。 | `PHASE0_CORPUS.md` L6469–L6497, L6560 |
| U7 | 重启哨兵（restart sentinel） | Gateway 退出前写入 sentinel；CLI 在托管服务重启健康检查结束后更新它。哨兵待处理或失败时，`openclaw status` 显示 `Update restart` 行。 | `PHASE0_CORPUS.md` L6489–L6497 |
| U8 | 核心后收敛（post-core convergence） | Gateway 重启前必须修复缺失插件载荷并验证活跃插件安装记录及可加载性；失败会使更新返回错误，并阻止以未经验证的插件集重启。 | `PHASE0_CORPUS.md` L6555–L6560 |
| U9 | 插件同步（plugin sync） | 更新后按活跃通道同步受跟踪插件。beta 优先尝试 `@beta`；无 beta 或安装验证失败时回退到记录的默认/`latest` 规格并报告警告。 | `PHASE0_CORPUS.md` L6537–L6549 |

## Gateway operations

| ID | 术语 | 定义 | Corpus 证据锚点 |
|---|---|---|---|
| G1 | Gateway | 常驻的运维中枢：收敛路由、控制面、渠道连接、HTTP API、Control UI 和部分插件路由；日常运行包括启动、配置、监督、健康检查与诊断。 | `BOOK_OVERVIEW.md` L31–L39 |
| G2 | Gateway probe（`openclaw gateway probe`） | 用于远程 CLI 验证 Gateway 的探测命令。对显式远程 URL，必须显式提供凭据，不能假定本地配置凭据会自动适用。 | `PHASE0_CORPUS.md` L2993–L2999 |
| G3 | 健康检查（`openclaw health`） | Gateway 暴露前的基线检查命令之一；应先处理 critical 发现，警告仅在部署意图明确且有文档记录时接受。 | `PHASE0_CORPUS.md` L2978–L2991 |
| G4 | Control UI | Gateway 的控制界面表面。若部署在非回环地址，必须显式设置 `gateway.controlUi.allowedOrigins`；过时的设备认证绕过配置不应加入当前配置。 | `PHASE0_CORPUS.md` L2771–L2774, L4831–L4833 |
| G5 | 操作员作用域（operator scopes） | 用于控制面操作的权限集合。设备升级时，请求的 `operator.read`、`operator.write`、`operator.approvals`、`operator.pairing` 等作用域须由审批方已持有，或由 `operator.admin` 覆盖。 | `PHASE0_CORPUS.md` L1002–L1024 |
| G6 | 会话键（`sessionKey`） | 会话 ID 或标签，是路由选择器而非授权令牌。 | `PHASE0_CORPUS.md` L3175–L3179 |
| G7 | Gateway 暴露运行手册（exposure runbook） | 对 bind、反向代理、Tailscale 或渠道策略的变更使用的预检与回滚清单；其基线包含 `doctor`、安全审计、深度审计和健康检查。 | `PHASE0_CORPUS.md` L2928–L2991, L3106–L3149 |
| G8 | 混合重载（hybrid reload） | 只在安全时热应用配置；若不安全则要求重启。无效配置会阻止启动或遭到热加载拒绝，但已接受的运行时快照继续服务。 | `BOOK_OVERVIEW.md` L33–L35 |

## 使用与校验说明

- 本表将 Security、Upgrade 与 Gateway operations 的高频运维概念分组；未将语料之外的实现细节或建议性推断作为定义。
- 所有 `PHASE0_CORPUS.md` 锚点均可直接用于后续 reviewer 的逐项复核；`BOOK_OVERVIEW.md` 只用于整体运行模型及跨文档概括。
- 实际变更前仍应以对应路径标记下的原始官方文档为准。
