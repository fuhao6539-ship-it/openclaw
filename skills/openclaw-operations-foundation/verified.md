# Verified — Batch A（Security、Upgrade、Gateway）

> 仓颉阶段 1.5 三重验证结果。冻结 P0 锚点：commit `063ce57caf233dc3cac124349ccddf6cef9ca432`；release `v2026.7.1-2`。
> 案例、反例、术语只作为验证与后续 RIA-TV++ 的支撑证据，不单独进入技能生成池。

## 通过池（10 个框架）

```yaml
- id: f01
  title: 最小满足暴露面选择
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/gateway/security/exposure-runbook.md：loopback、tailnet/LAN、反向代理与公网暴露的排序
      - docs/gateway/security/index.md：本地优先、认证及网络暴露的分层安全模型
  V2_predictive_power:
    passed: true
    novel_question: "为仅供运维人员使用的 Control UI 提供远程访问，应直接开放公网端口吗？"
    derived_answer: "不应直接开放；先选 loopback+SSH tunnel 或 loopback+Tailscale Serve，仅在工作流要求时逐级扩大。"
  V3_exclusivity:
    passed: true
    why_not_common: "不是抽象的最小权限口号，而是对 Gateway 暴露模式、认证表面和必需控制的具体排序。"

- id: f02
  title: 单控制逐步放宽与逐步验证
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/gateway/security/exposure-runbook.md：渠道 allowlist、写工具与反向代理按单控制面放宽
      - docs/install/updating.md：受监管的包替换、服务重启与健康验证分离
  V2_predictive_power:
    passed: true
    novel_question: "将一个内部机器人接入新消息渠道时能否同时开放群组、exec 与公网代理？"
    derived_answer: "不能；先逐项开放并在每项后验证授权、拒绝、路由与工具审批，当前变化未理解前停止。"
  V3_exclusivity:
    passed: true
    why_not_common: "把渠道、工具、代理与验证点作为可审计的单变量序列，而非一般性的“谨慎变更”。"

- id: f03
  title: 身份优先、范围其次、模型最后
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/gateway/security/index.md：身份、作用域、模型的明确排序
      - docs/gateway/security/exposure-runbook.md：DM 配对、工具策略、沙箱与远程暴露检查
  V2_predictive_power:
    passed: true
    novel_question: "面对可能含提示注入的第三方 webhook，优先增加提示词还是先限制入口与工具？"
    derived_answer: "先认证/限制触发者，再收缩可作用范围与工具；将模型视为可能被操纵的组件。"
  V3_exclusivity:
    passed: true
    why_not_common: "明确否定以模型行为或提示词替代身份和权限控制，是该运维语料的核心安全排序。"

- id: f04
  title: 信任边界拆分判定
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/gateway/security/index.md：个人助理信任模型与 hostile multi-tenant 边界
      - docs/gateway/security/exposure-runbook.md：共享消息渠道属于共享委托工具权力，需独立部署隔离
  V2_predictive_power:
    passed: true
    novel_question: "两个互不信任团队能否只靠不同 sessionKey 共用一个 Gateway？"
    derived_answer: "不能；sessionKey 是路由而非授权，应拆分 Gateway、凭据，并尽可能拆分 OS 用户或主机。"
  V3_exclusivity:
    passed: true
    why_not_common: "把 Gateway 的个人操作员假设与租户隔离要求直接关联，而非泛化地要求“加强权限”。"

- id: f05
  title: 工具执行三层控制模型
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/gateway/sandbox-vs-tool-policy-vs-elevated.md：运行位置、工具可用性、exec 逃逸的三层分工
      - docs/gateway/sandboxing.md：沙箱覆盖范围与 elevated 例外
  V2_predictive_power:
    passed: true
    novel_question: "某工具被允许后仍无法访问宿主文件，应修改 allowlist 还是 sandbox 配置？"
    derived_answer: "先区分工具可用性与运行位置；allowlist 不能改变 sandbox/host 执行位置，禁止直接开启 elevated 作为默认修复。"
  V3_exclusivity:
    passed: true
    why_not_common: "对 sandbox、tool policy、elevated 的非等价关系给出明确诊断框架。"

- id: f06
  title: 两级审计证据模型
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/cli/security.md：常规冷路径审计与 --deep runtime probe/插件采集器的差异
      - docs/gateway/security/exposure-runbook.md：暴露前及代理修改后要求重跑 --deep
  V2_predictive_power:
    passed: true
    novel_question: "配置审计无 critical finding 后能否断言反向代理部署已安全？"
    derived_answer: "不能；静态审计不覆盖全部运行时/插件证据，暴露与代理变化需加入 --deep 并测试授权和拒绝路径。"
  V3_exclusivity:
    passed: true
    why_not_common: "按审计证据副作用和覆盖面划分结论强度，而非把单次审计当作安全证明。"

- id: f07
  title: 可恢复升级与双层回退
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/install/updating.md：代码回滚优先、状态恢复仅在兼容性必要时使用
      - docs/install/backups.md：归档、SQLite 与状态资产恢复边界
  V2_predictive_power:
    passed: true
    novel_question: "升级后健康检查失败，是否应立即覆盖回更新前的完整状态？"
    derived_answer: "先代码回滚并保留 current state；仅在旧代码无法读取迁移后的 config/database 时恢复状态，且先保存当前状态。"
  V3_exclusivity:
    passed: true
    why_not_common: "明确区分代码恢复和时间回退式状态恢复及其数据丢失代价。"

- id: f08
  title: 先验证候选、后激活的恢复模型
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/install/backups.md：归档验证、staging extraction 与显式离线激活
      - docs/install/migrating.md：恢复不原地激活，停止 Gateway 后再迁移或切换 state directory
  V2_predictive_power:
    passed: true
    novel_question: "收到一份备份归档后能否直接覆盖正在运行的实例以缩短恢复时间？"
    derived_answer: "不能；恢复到新目标，验证内容与数据库，停止 Gateway 后由操作员执行显式激活。"
  V3_exclusivity:
    passed: true
    why_not_common: "把恢复候选的验证与生产激活分开，直接对应 Gateway state、凭据与渠道状态的风险。"

- id: f09
  title: 在线状态的备份路径选择
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/install/backups.md：archive、SQLite snapshot、Git/连续复制的支持路径
      - docs/install/updating.md：重大升级前的 verified backup 与状态恢复边界
  V2_predictive_power:
    passed: true
    novel_question: "Gateway 正在写 SQLite 时，是否可仅复制 .sqlite 与 WAL 文件作为回滚点？"
    derived_answer: "不可；使用支持的、捕获 committed state 的备份路径，按恢复目标选择 archive 或 SQLite snapshot。"
  V3_exclusivity:
    passed: true
    why_not_common: "根据在线状态、恢复粒度和一致性要求选择机制，且把备份视为受保护状态副本。"

- id: f10
  title: 受监督更新的进程外交接
  type: framework
  V1_cross_domain:
    passed: true
    evidence:
      - docs/install/updating.md：受管服务由脱离式 CLI handoff 停止、替换、重启、验证
      - docs/gateway/runbook.md：Gateway 服务生命周期、状态和健康检查的运维完成条件
  V2_predictive_power:
    passed: true
    novel_question: "运行中的受管 Gateway 能否通过控制面在自身进程内替换包树？"
    derived_answer: "不能；交由进程外更新路径完成服务停止、包替换、元数据刷新、重启和可达性验证。"
  V3_exclusivity:
    passed: true
    why_not_common: "把“包安装成功”与“受监管服务已在新版本健康运行”区分为不同完成条件。"
```

## 合并与淘汰摘要

- 通过：10 个框架（`f01`–`f10`）。
- 不单独成 skill：20 个原则（`p01`–`p20`）；它们分别是通过框架的原子执行规则或检查项，缺少独立的 V1 跨域/ V3 独特性，详见 `rejected/principles.md`。
- 支撑资产：6 个案例、10 个反例、31 个术语；保留在候选目录，用于阶段 2 的 A1/B/术语约束。

## 验证回执

- YAML 框架/原则候选已解析校验：ID 唯一、必填字段完整、`docs/...` 来源有效。
- 引文均不超过 150 字符，且在各自声明的冻结 P0 来源段中精确存在。
- 未修改 Gateway、系统配置、权限或外部服务。