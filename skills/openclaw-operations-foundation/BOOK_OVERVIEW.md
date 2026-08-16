# OpenClaw 官方运维文档集：安全、升级与 Gateway — 整书理解（阶段 0 产出）

> 本文档是 cangjie-skill 流水线的阶段 0 产出，后续 extractor 和 skill 仅可将其作为全局理解上下文，不能把其中的归纳替代为逐项 P0 事实依据。
>
> **来源范围**：Batch A（安全、升级、Gateway 运维）；仅限 P0 官方语料。仓库快照：`063ce57caf233dc3cac124349ccddf6cef9ca432`（`2026-08-15T23:59:49-07:00`）；Release 锚点：`v2026.7.1-2`（`2026-08-04T00:41:26Z`）。候选条目跨专题重叠，去重后共 77 份，详见 `../../manifests/SOURCE_MANIFEST.md` 与 `../../manifests/source_manifest.tsv`。

## 基本信息

- **标题**：OpenClaw 官方运维文档集：安全、升级与 Gateway
- **作者**：OpenClaw maintainers / openclaw
- **出版/发布时间**：官方仓库快照 `063ce57caf233dc3cac124349ccddf6cef9ca432`；最新 Release `v2026.7.1-2`
- **内容类型**：官方技术文档集（Gateway 运维手册、安全治理与升级/恢复指南）
- **版本来源**：`../../sources/official/repository/docs/`；`../../sources/official/github-api/releases-latest.json`
- **处理时间**：2026-08-16
- **批次范围**：Security、Upgrade、Gateway operations；不覆盖 Agents/路由、Skills、Memory、渠道与通用排障专题的独立蒸馏。

---

## 1. 结构（Structural）

### 类型

实操运维手册 + 安全治理文档 + 平台生命周期运行规范。

### 一句话主旨

在单一受信操作员的部署假设下，指导操作者以访问控制、工具约束、可观测性、可验证升级和可恢复状态管理，安全运行具备消息、文件、网络与执行能力的 OpenClaw Gateway。

### 骨架（主要论点及其关系）

1. **信任模型先行**：一个 Gateway 以一个受信操作员边界为设计前提；互不信任的用户不应共享同一 Gateway、操作系统用户或主机。
2. **Gateway 是运维中枢**：Gateway 将路由、控制面、渠道连接、HTTP API、Control UI 和部分插件路由收敛于一个常驻服务与单一复用端口；启动、配置、监督、健康检查和诊断构成日常运行面。
3. **安全依靠分层收敛而非单点承诺**：渠道访问、会话/上下文、工具执行、外部内容和供应链构成不同信任面；认证、配对、allowlist、工具策略、exec 审批、沙箱和网络暴露控制共同限制爆炸半径。
4. **配置是受校验的运行时契约**：配置必须匹配 schema；无效配置会阻止启动或被拒绝热加载，已接受的运行时快照继续服务；`hybrid` reload 只在安全时热应用，否则要求重启。
5. **升级必须按可恢复变更处理**：更新通道、预检、服务交接、Doctor、插件收敛和健康核验构成更新流程；完整备份、代码回退与状态恢复是不同层次的恢复手段。
6. **运维以证据和症状为入口**：状态、Gateway 探测、日志、Doctor、渠道探测和针对性故障签名提供从症状到验证与处置的命令阶梯。
7. **威胁模型使风险显性化**：MITRE ATLAS 文档把提示注入、工具滥用、凭证窃取、恶意技能、网络暴露和数据外泄组织为攻击面、残余风险与优先级，而非把安全表述为“已解决”。

**论点之间的关系**：递进与并列。第 1 点限定全书适用前提；第 2 点定义被运维的对象；第 3、4 点描述安全与配置控制面；第 5 点覆盖变更生命周期；第 6 点给出运行中的证据路径；第 7 点把前述控制面放入对抗风险框架。第 3–6 点在第 1 点前提下并列，且在实际操作中通常遵循“先控制暴露与权限、再变更、再验证/排障”的顺序。

### 作者要解决的核心问题

如何让拥有 shell、文件、网络和消息能力的 AI agent 平台在个人/单一受信操作员环境中保持可用，同时使访问面、工具权限、外部内容、软件供应链、升级失败和状态丢失都能被明确约束、检查、恢复或追溯。

---

## 2. 解释（Interpretive）

### 关键术语（作者本人的用法）

| 术语 | 作者的定义 | 和常识用法的差异 |
|---|---|---|
| Gateway | 常驻的路由、控制面和渠道连接服务；同一端口复用 WebSocket/RPC、HTTP API、Control UI 等表面。 | 不只是传统 API gateway 或反向代理，而是 agent 运行时的服务中枢。 |
| Personal assistant trust model | 每个 Gateway 面向一个受信操作员边界；同一 Gateway 内的认证操作者不是彼此隔离的租户。 | 不等同于多租户 SaaS 的逐用户授权模型。 |
| `sessionKey` | 用于路由和选择上下文/session 的标识。 | 明确不是认证令牌或独立授权边界。 |
| `dmPolicy` | 对入站私信的前置门控：`pairing`、`allowlist`、`open`、`disabled`。 | 决定谁可触发 agent，不是加密、签名或隐私保护本身。 |
| `contextVisibility` | 限制哪些补充上下文进入模型提示词的控制项。 | 与触发授权分离；允许触发并不自动意味着可见所有历史或引用内容。 |
| exec approvals | 由 allowlist 与询问策略组成的操作员意图护栏，尽力绑定请求上下文与可识别文件操作数。 | 不被定义为对敌对多租户的强隔离。 |
| Sandbox | 可选的 Docker、Podman、SSH 或 OpenShell 工具执行隔离后端。 | Gateway 与原生插件仍在宿主侧；沙箱是降爆炸半径措施，不是完美边界。 |
| `tools.elevated` | 可使 exec 在沙箱外的 Gateway 或 node 路径运行的全局逃逸面。 | 不是自动安全提权流程，而是操作者选择扩大执行面。 |
| SecretRef | 通过 `env`、`file`、`exec` 或 `store` 间接解析密钥的引用契约。 | 不是单独的密钥管理产品或授权模型。 |
| `openclaw security audit` | 对访问、工具、文件权限、网络暴露、插件、策略漂移等进行结构化检查的命令；`--fix` 只执行窄范围修复。 | 不是渗透测试，也不证明系统不存在风险。 |
| Update channel | `stable`、`extended-stable`、`beta`、`dev` 等更新目标/节奏选择。 | 不只是功能标签；会影响安装方式、版本选择与插件收敛。 |
| Restart handoff | 受监督的更新中，Gateway 退出并由进程外 CLI helper 完成替换、重启和核验。 | 不是在运行进程内热替换自身代码。 |
| Backup restore | 对归档或数据库快照恢复到新目标的显式离线操作，随后由操作者激活。 | 不会原地覆盖 live state，也不保证消息渠道的 ratchet/审批状态无副作用。 |

### 核心命题（用自己的话）

1. 平台安全的首要边界是“谁能触发、谁能进入控制面”，而非把 `sessionKey` 或提示词隔离误当作身份认证。
2. 提示注入没有被系统提示词解决；实际防护依赖缩小可触发者范围、缩小工具权限、沙箱化执行和把外部内容当作不可信输入。
3. Gateway 默认的 loopback、认证和 DM pairing 倾向于私有运行；向 LAN、反向代理、公开群组或远程浏览器扩展时必须重新评估暴露面。
4. Schema 校验、原子配置替换、最后已知良好配置和拒绝无效 reload 共同使配置变更成为可验证操作，而非随意文本编辑。
5. `openclaw update` 是受控变更流程，不等于简单包升级：它涉及安装类型/通道、服务生命周期、Doctor、插件一致性和重启后的版本/健康核验。
6. 回滚应优先代码回退；只有旧代码无法读取已迁移状态时才考虑恢复预更新状态，因为状态恢复会丢弃其后的数据并可能影响渠道凭证状态。
7. 健康与故障结论应来自 `openclaw status`、`openclaw gateway status`、日志、Doctor 和 live probe 等证据，而不是 UI 可打开或请求被接受等中间信号。
8. 插件和技能应视作受信代码：安装时的来源审查、版本固定、allowlist、安装策略和审计是供应链治理；它们并非运行时隔离替代品。
9. 备份包含 auth profile、凭证、会话与工作区等敏感材料，必须按 live state 同等级别加密、限权和审慎传输。
10. ATLAS 威胁模型的价值在于列出残余风险与攻击链；它并不声称 prompt injection、技能投毒或宿主失陷已被消除。

### 论证链

文档先从个人助理信任模型和 Gateway 服务模型界定对象与边界，再以访问门控、会话/上下文、工具策略、沙箱、网络认证和供应链控制说明分层防御；随后用严格配置校验、健康/诊断命令与安全审计把控制变成可观测的运行程序。更新文档将安装切换、服务交接、Doctor、插件同步和验证串为变更闭环，备份/恢复文档补上可逆性。最后，MITRE ATLAS 威胁模型将这些机制映射到提示注入、远程执行、恶意技能、数据外泄和资源耗尽等风险链。其证据主要是官方命令语义、配置约束、默认行为、操作步骤和威胁建模表，而非独立实证研究。

---

## 3. 批判（Critical） ★

### 作者的时代局限

- 文档冻结于仓库快照 `063ce57caf233dc3cac124349ccddf6cef9ca432` 与 Release `v2026.7.1-2`；配置字段、默认值、渠道能力、插件行为和升级语义均可能随版本变化，不能把本概览作为未来版本的操作依据。
- 容器、Podman、AppArmor、systemd、反向代理和网络环境存在显著宿主差异；同一推荐配置在不同平台的实际隔离与可用性可能不同。
- 对提示注入、模型可靠性和 AI 供应链的防御技术仍快速演进，当前机制只能反映文档期的工程权衡，不能提供长期保证。

### 作者的立场盲点

- 文档明确偏向单一受信操作员，对“团队共享但成员并非完全互信”的中间场景主要给出拆分 Gateway/OS 用户/主机的建议，缺少细粒度多租户治理方案。
- 操作者被社会工程诱导去配对、放宽 allowlist、启用 elevated、安装插件或执行恢复操作时，现有控制更多是风险提示与人工流程，而非独立强制审批链。
- 文档以运维者与宿主可信为前提；一旦宿主、状态目录、日志或拥有 Gateway 配置写权限的账户被攻陷，日志/审计和本地控制的可信度都会降低。

### 未被证明的假设

- 假设个人操作者能持续理解并处理审计发现、网络模型、工具策略、版本迁移和备份恢复后果。
- 假设配对/allowlist、工具限制、审批和可选沙箱的组合可将提示注入的风险降至部署可接受的程度；文档本身承认提示注入未被彻底解决，且沙箱默认关闭。
- 假设插件/技能的扫描、审核与安装策略足以支持操作者的信任判断；官方威胁模型同时说明，已安装代码仍可能以 agent 权限运行。
- 假设升级交接与恢复命令在具体服务管理器、包管理器和宿主环境中都可顺利执行；文档保留人工恢复路径，表明自动路径并非无条件保证。

### 最强反对意见

> 该体系把关键安全决策集中在单一受信操作员与其宿主上：如果该操作员、认证凭证或主机被攻陷，agent 的消息、文件、网络和执行权限会同时失去可靠边界。对于需要服务互不信任终端用户或团队成员的场景，“拆分 Gateway/主机”虽然安全上直接，却会显著增加部署和运营复杂度；默认关闭的沙箱与尚未解决的提示注入也使默认部署不能被误读为强隔离服务。

> **以上批判会直接成为下游 skill 的 Boundary (B) 字段来源。**

---

## 4. 应用潜力（Applicability）

### 可 skill 化的内容

- [ ] Gateway 启动、服务监督与健康核验：启动 → `gateway status`/RPC 证明 → channels probe → 日志。
- [ ] Gateway 暴露前预检与安全审计：访问策略、工具爆炸半径、认证、网络绑定、权限和插件检查 → 修复 → 复核。
- [ ] DM/群组触发边界决策：`pairing`、`allowlist`、`open`、`disabled` 与 mention gating 的选择、验证和回滚。
- [ ] 工具策略、exec 审批与沙箱配置：按任务/信任级别选择 mode、scope、backend、workspace access，并明确 elevated 例外。
- [ ] 更新与回滚：更新前备份和 dry-run → update/doctor/restart → 版本、健康、插件与渠道核验 → 代码/状态回退。
- [ ] 备份与恢复策略：archive、SQLite snapshot、Git backup、Litestream 的选择，敏感数据保护与离线激活验证。
- [ ] Gateway 症状式诊断：`status` → `gateway status` → logs → doctor → channels probe，并按更新、认证、端口、协议和 UI 症状分流。
- [ ] 网络/远程访问加固：loopback、token/password、Tailscale、SSH tunnel、反向代理和 trusted proxy 的适用条件。
- [ ] 事件响应：Contain → Rotate → Audit → Collect，且将报告材料与敏感日志处理分开。
- [ ] 插件/技能安装治理：来源、版本、allowlist、安装策略、审计与重启验证。

### 不适合 skill 化的内容

- MITRE ATLAS 的完整威胁编号映射和残余风险表：适合做 reference，而不是独立可执行 skill。
- 全量配置 schema：应按具体路径查询官方 reference/schema，而不应压缩为静态技能说明。
- 各渠道专属接入步骤与完整渠道排障：属于本批次以外的渠道专题。
- 完整 Release 叙事或历史 changelog：只能作为目标版本核验依据，不能替代当前 release 的逐项检查。

### 预估 skill 数量

**约 8–10 个**（最终由阶段 1.5 三重验证决定；此处仅为阶段 0 粗估）。

### 优先级排序（按“最能赋能普通人”）

1. Gateway 启动与健康核验
2. Gateway 安全审计与暴露前预检
3. 升级、验证与回滚
4. 症状式排障命令阶梯
5. DM/群组触发访问控制
6. 工具策略、exec 审批与沙箱
7. 备份、恢复与恢复演练
8. 网络与远程访问加固
9. 事件响应
10. 插件/技能安装治理

---

## 来源与锚点

- P0 快照：`063ce57caf233dc3cac124349ccddf6cef9ca432` / `2026-08-15T23:59:49-07:00`
- 最新 Release 记录：`v2026.7.1-2` / `2026-08-04T00:41:26Z`
- 语料清单：`../../manifests/SOURCE_MANIFEST.md`、`../../manifests/source_manifest.tsv`
- 代表性核心依据：
  - `../../sources/official/repository/docs/gateway/index.md`
  - `../../sources/official/repository/docs/gateway/configuration.md`
  - `../../sources/official/repository/docs/gateway/security/index.md`
  - `../../sources/official/repository/docs/gateway/sandboxing.md`
  - `../../sources/official/repository/docs/gateway/troubleshooting.md`
  - `../../sources/official/repository/docs/install/updating.md`
  - `../../sources/official/repository/docs/install/backups.md`
  - `../../sources/official/repository/docs/security/THREAT-MODEL-ATLAS.md`

## ✅ 质量门检查

- [x] 主旨能用一句话说清
- [x] 骨架列出 3–7 个一级论点（7 个）
- [x] 关键术语词典 ≥5 条（14 条）
- [x] 批判阶段列出 ≥3 条作者局限
- [ ] 已向用户展示并得到确认

**用户确认时间**：（待确认）
