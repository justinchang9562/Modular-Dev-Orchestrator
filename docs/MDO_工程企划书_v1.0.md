# Modular-Dev-Orchestrator

# MDO 工程企划书

> **版本**：v1.0-G0-Candidate · **日期**：2026-11-12 · **状态**：DRAFT / 等待 G0 独立审查及 Owner 批准  
> **产品策略**：C — 首版个人使用，架构面向未来公开发布（Personal-first, Public-ready）  
> **目标环境**：Apple Silicon Mac（arm64） · SwiftUI 原生 App · 独立 DMG 分发  
> **配套文件**：《MDO 开发流程与执行手册》v1.0-G0-Candidate（同版本）  
> **标准规范**：[Modular-Dev-Quality-SOP](https://github.com/justinchang9562/Modular-Dev-Quality-SOP)，在 G0 批准时固定 commit SHA  
> **重要声明**：本文是拟开发系统的工程基线候选，而非已开发功能或已通过测试的事实；所有待验证接口须在 M00 取得真机证据。

---

## 0. 文档控制、阅读方法与决策权

### 0.1 本文件的职责

本文件回答 **WHAT / WHY / ARCHITECTURE / ACCEPTANCE BASIS**：MDO 是什么、服务谁、要实现哪些能力、组件之间有哪些不可突破的边界、什么证据才能证明目标实现。它不记录每日进度，也不作为每次运行时的临时任务清单。

与配套《MDO 开发流程与执行手册》的关系：企划书规定**工程事实与不变量**；执行手册规定**阶段顺序、模块责任、测试、文档交接和人工授权动作**。若实施细节与此文件的已批准架构冲突，必须提出 ADR / G0 修订，不允许开发会话自行选择方便的版本。

### 0.2 文件权威与版本冻结

1. **法律、平台和外部技术行为**：以执行当时适用的 Apple / OpenAI / GitHub 官方资料为依据，记录发布日期、访问日、适用条件及确认结论。官方资料无法确认时标记 `UNVERIFIED`，不可写成已知事实。
2. **产品与系统目标**：以 Owner 批准并锁定 SHA 的本工程企划书为准。
3. **开发执行规范**：以配套执行手册及 SOP 固定修订版为准；同等层级冲突应停机交 Owner 决策。
4. **具体模块实现**：以通过 G1 的 Implementation Plan、ADR、契约和该模块授权为准。
5. **当前真实状态**：以 Git HEAD、工作树、持久化任务状态、可信测试记录和批准事件为准；`PROJECT_STATUS.md` 只是索引，不代替证据。
6. **已验证经验**：Skill 仅是可复用工程经验，不覆盖外部官方规则和当前真实证据。

G0 批准时必须生成 `BASELINE_MANIFEST.json`，含 `project_plan_sha256`、`execution_manual_sha256`、`sop_commit_sha`、`module_roadmap_revision`、`approved_by`、`approved_at` 及未关闭的条件性风险；未经 Owner 批准，不得把 DRAFT 改写为 APPROVED。

### 0.3 术语和状态

- **G0–G4**：项目基线、计划、实现自检、独立审核及模块验收门槛；另外设 PR / CI / Merge Gate。
- **QC1 / QC2**：开发端完整自检 / 隔离 Reviewer 独立审核。QC2 PASS **不等于**获准 push、merge 或发布。
- **Snapshot**：固定源码、文档、测试证据和环境指纹的版本对象。
- **Owner / Policy Engine**：唯一具备人工授权资格的负责人 / 独立且确定性执行政策的控制组件。
- **Public-ready**：保留未来多用户公开分发的安全、许可、身份与路径扩展能力，**不代表 v1.0 要提供公众服务、收费、账号体系或云端 SaaS**。

## 1. 执行摘要与产品定位

### 1.1 一句话定义

MDO 是一款 **本地优先、Apple Silicon 原生的 AI 软件工程工作流编排器**：使用者用自然语言说明目标并批准项目基线，系统在边界内调度本地 Codex、独立 Reviewer、测试执行器及 GitHub，把现有手工 SOP 转化为可恢复、可追溯、可拒绝的模块开发闭环。

### 1.2 为什么需要 MDO

目前人工 SOP 中，开发计划、审核意见、修复 Prompt、Review Bundle、CI 结果与 Module Gate 依赖人工复制、核对和转交。这带来漏传、错版、越级、审查证据不全和断点难恢复等风险。MDO 不是要替代质量判断，而是把**重复转交变为自动状态转换**，将需要理解风险的决策交回人类。

### 1.3 产品愿景和成功标准

v1.0 以“**在授权范围内无人值守产生可审核 PR，默认由人类合并**”为主要成功定义。系统必须首先做到**正确地拒绝、暂停与恢复**，才谈得上缩短操作耗时。开发和审核端不得拥有能够覆盖政策与审批记录的权限。

### 1.4 使用范围（已确定）

| 维度 | v1.0 冻结目标 | 后续扩展边界 |
|---|---|---|
| 用户 | 首版仅 Owner 本人，支持其多台 Mac 分别安装 | 未来可公开分发，但不做 v1.0 的多租户产品 |
| 硬件 | Apple Silicon arm64 | 不提供 Intel Universal Binary |
| 应用 | SwiftUI 原生 macOS App，独立 DMG | 可为其他用户开放首次引导，不依赖开发者账号 |
| 执行 | 本机长期任务引擎、隔离 Runner、Codex CLI/SDK | 后续可增加远程 Runner；v1.0 不实现 |
| 审核 | 官方可验证的独立模型集成；失败则隔离 Codex 或人工审核 | 不抓取网页 ChatGPT 的私人会话 |
| 代码托管 | GitHub 指定仓库，最小权限，受控 PR/CI | 保留抽象适配器，不要求立即支持其他平台 |
| 合并 | 默认由人类明确批准；合并后独立复验 | 自动合并需要独立风险审查与功能开关 |
| 付费 | 不因额度耗尽自动购买 API 或 Credits | API 适配器可选且默认禁止 |

### 1.5 非目标（Out of Scope）

不开发自有大模型；不自动读取或操控普通 ChatGPT 网页会话；不实现云端多租户服务、Windows/Linux 桌面端、Intel Mac、App Store 上架、真实交易下单、任意生产部署、资金转移或无需批准的破坏性迁移；不承诺 Plus 额度无限；不把公开发行能力等同于 v1.0 的上线义务。

## 2. 用户角色、典型情景与用户旅程

### 2.1 角色权限

| 角色 | 责任 | 明确禁止 |
|---|---|---|
| Owner | 产品目标、G0 基线、权限政策、R2/R3、合并与发布决策 | 不能让普通聊天文本当作授权凭证 |
| Planner | 阅读规范与来源、输出有界计划 | PLAN 模式修改实现代码 |
| Developer（Codex） | 实现、测试、QC1、FIX、报告 | 自宣 QC2 PASS、自授 GitHub merge、修改政策主控 |
| Reviewer | 只读审核 Plan/Implementation/PR/Post-merge，给 Finding 与证据 | 修改目标工作树、提交代码、写审批数据库 |
| Orchestrator | 编排、存储、恢复、预算、版本绑定、执行安全动作 | 以模型自然语言绕过 Policy Engine |
| Policy / Approval Service | 身份验证、权限、签发和撤销操作授权 | 将审核通过自动升级为无限操作权限 |

### 2.2 用户旅程：从想法到首个 PR

1. 安装并启动 MDO.app，选择 PROJECTS 根目录，诊断本地 Git/Codex、Reviewer、GitHub、磁盘和执行环境。
2. 新建项目或导入已有项目；输入目标、非目标、系统约束、SOP 版本及风险授权偏好。
3. Planner 草拟长期计划、路线图和执行规范；独立 Reviewer 做 G0 候选审查；Owner 确认并冻结基线。
4. 按依赖与授权顺序选定 Mx；创建隔离工作区；Planner 生成 Implementation Plan；Reviewer 审核 G1。
5. G1 PASS 且 Policy 授权后，Codex 实现、执行 Unit/Contract/Integration/Failure Injection、UI 验证及 QC1。
6. 生成固定 Snapshot 与 Review Bundle；独立 Reviewer 审核 QC2，失败自动进入有界 FIX（每轮重新验证全部受影响内容）。
7. 通过 G3 后由受控 GitHub Adapter 创建或复用 PR；采集真实 CI 检查；版本、权限、审查均一致时请求合并授权。
8. 获得明确操作授权后进行受控合并；执行 Post-Merge 验证；完成 G4；只有已预授权且依赖满足的下一个模块才可开始。

### 2.3 导入已有仓库

读取实际 HEAD、暂存/未暂存/未跟踪内容、现有 ADR、测试、已合并模块和 GitHub 保护；默认使用只读扫描，并生成 `IMPORT_ASSESSMENT.md`。若已有工作与拟创建分支冲突则停止要求确认；不可自动重建旧项目或覆盖不属于本任务的代码。所有导入项目必须单独经 G0 基线批准。

### 2.4 错误与暂停的用户体验

任何暂停界面必须展示：当前真实状态、触发规则、被阻断的操作、修复建议、证据链接、最近一次安全检查点、是否可自动恢复。`UNKNOWN` 不得展示为成功；额度不足不得静默切换到付费渠道；审批过期不得被视为历史许可继续使用。

## 3. 需求分类与追踪规范

本企划书使用唯一 Requirement ID；实现必须从 `REQ` 映射到 Mx、Acceptance Criteria（`AC-Mxx-nn`）、测试证据 ID 和 Snapshot。没有来源的“已完成”状态无效。

| 类别 | 编号 | 必须达到的可验证能力 |
|---|---|---|
| Platform | REQ-PLAT-001 | 只支持 Apple Silicon arm64；非支持环境可解释地拒绝 |
| Platform | REQ-PLAT-002 | 构建原生 SwiftUI App；DMG 经开发者签名与公证验证 |
| Platform | REQ-PLAT-003 | 第二台 Mac 独立安装、重新授权并能导入项目 |
| Project | REQ-PROJ-001 | 创建/导入项目，有明确目录、范围、依赖和基线 |
| Project | REQ-PROJ-002 | G0 审核与 Owner 批准绑定内容哈希和 SOP SHA |
| Workspace | REQ-WS-001 | NSOpenPanel 原生目录选择与单项目覆盖默认根目录 |
| Workspace | REQ-WS-002 | 持久化用户授权并检测失效、iCloud 占位与权限改变 |
| Workspace | REQ-WS-003 | 路径正规化、同名保护、symlink / traversal / TOCTOU 防护 |
| Workspace | REQ-WS-004 | 每项目独立源码、工作树、状态与缓存；禁止跨项目越权 |
| Workflow | REQ-WF-001 | 具有确定性 G0–G4 和 PR Gate 状态机 |
| Workflow | REQ-WF-002 | 任务租约、数据库事务、WIP 限制、可恢复持久队列 |
| Workflow | REQ-WF-003 | PLAN/QC2/PR 修复循环有界、计数与升级机制 |
| Workflow | REQ-WF-004 | 崩溃/重启/未知副作用后对账而非盲重试 |
| Agent | REQ-AG-001 | 本地 Codex Plan / Implement / Fix 程序化调用 |
| Agent | REQ-AG-002 | 独立审核接口、隔离上下文、只读快照 |
| Agent | REQ-AG-003 | Reviewer 结构化审查结论并引用证据/问题编号 |
| Agent | REQ-AG-004 | 登录失效、额度上限、网络错误与安全降级可解释 |
| Evidence | REQ-EV-001 | 完整 Bundle 与逐文件哈希、累计 diff、新文件清单 |
| Evidence | REQ-EV-002 | 测试由独立执行器产生可核验命令、退出码与日志 |
| Evidence | REQ-EV-003 | PASS / CI / Approval 绑定同一版本与环境 |
| Security | REQ-SEC-001 | Fail-Closed；缺失授权、证据或状态禁止高风险动作 |
| Security | REQ-SEC-002 | 非模型可修改的 Policy Engine，R0–R3 风险分类 |
| Security | REQ-SEC-003 | 人工审批绑定动作、仓库、版本、环境、有效期和撤销 |
| Security | REQ-SEC-004 | 代码执行与主控配置/凭证有实际 OS 层隔离 |
| Security | REQ-SEC-005 | Agent 不得自我审批、修改策略或绕过 GitHub 门禁 |
| GitHub | REQ-GH-001 | 幂等提交/推送/建 PR、读取真实 CI 和保护规则 |
| GitHub | REQ-GH-002 | 受控 merge、Post-Merge verify、G4 并可追溯 |
| UX | REQ-UX-001 | SwiftUI 首次向导、PROJECTS 设置与项目管理 |
| UX | REQ-UX-002 | 实时运行状态、任务时间线、暂停恢复与错误说明 |
| UX | REQ-UX-003 | 审核 Finding、证据、批准/拒绝/修改/撤销操作 |
| Observability | REQ-OBS-001 | 审计日志、运行指标、脱敏、证据导出和保留策略 |
| Observability | REQ-OBS-002 | 环境诊断、错误分类、日志安全导出 |
| Cost | REQ-COST-001 | 模型预算、修复轮数上限；默认禁止付费 API 回退 |
| Release | REQ-REL-001 | App/Helper 签名、公证、DMG、升级迁移及回滚测试 |
| Release | REQ-REL-002 | Public-ready 安全准备清单与将来发布决策隔离 |
| Governance | REQ-DOC-001 | SOP 固定版本、Source Manifest、ADR 和模块交接一致 |
| Test | REQ-TEST-001 | Fake-Agent、E2E Lab、真实项目试点和 P0 负向测试 |

## 4. 功能边界与产品模块

### 4.1 项目管理

支持新建、导入、归档、只读查看；输入目标、In Scope、Out of Scope、AC、预算、首次执行模式、仓库位置。新建项目默认根目录 `PROJECTS/<safe-slug>`，提供真实路径预览、同名检查和取消操作，失败不留下半成品；由 Workspace Manager 创建项目，不向 Agent 暴露任意主目录写权限。为每个项目生成 `docs/PROJECT_PLAN.md`、`EXECUTION_MANUAL.md`、`MODULE_ROADMAP.md`、`SOURCE_OF_TRUTH_POLICY.md`、`PROJECT_STATUS.md` 与必要 ADR。

### 4.2 规划与模块治理

基线从用户需求生成，但不在获得 G0 Owner 许可前认为已获准。一个目标项目默认最多一个在制 Module；每个任务包含 `project_id`、`module_id`、`goal`、`scope`、`out_of_scope`、`dependencies`、`acceptance_criteria[]`、`budget`、`authorization_context`、`baseline_id`。缺任意必须字段先阻断，而非 Agent 补猜。

### 4.3 自动化质量闭环

遵循 G1 Plan → G2 Implement/Test/QC1 → G3 QC2/FIX → PR/CI → G4。Planner、Developer、Reviewer 的输出分别强制 Schema 校验；Reviewer 不能只根据 Completion Report 给 PASS；必需测试 `NOT RUN` 不等于 `NOT APPLICABLE`；人工拒绝不可通过自动重试被覆盖。

### 4.4 受控 GitHub 操作

普通 Agent 仅提出候选动作。只有可信 Adapter 持有受限授权，具备固定分支、PR、CI 检查与获准 merge 权限。首次 MVP 默认所有 merge 由 Owner 明确批准，不要求 GitHub 账号必须具有管理员权限；若受保护动作无法完成，保留 `WAIT_APPROVAL` 或 `BLOCKED_GITHUB` 状态。

### 4.5 原生界面范围

主要导航：Overview、Projects、Project Detail、Runs、Reviews、Approvals、Settings、Diagnostics。所有操作显示当前目标项目、模块、快照、审批影响和风险；按钮在权限未满足时禁用并说明理由。首版不以复杂动画、多人共享或外观主题为主目标。

## 5. 技术架构：本地优先的控制面与执行面

### 5.1 概念图

```text
┌──────────────────── SwiftUI macOS App ────────────────────┐
│ Onboarding / Projects / Runs / Reviews / Approvals / Settings │
└───────────────────────┬─────────────────────────────────┘
                        │ 受控 IPC / 本机身份验证
┌───────────────────────▼─────────────────────────────────┐
│ Trusted Control Plane: MDO Engine + Policy + Approval     │
│ Task/State/Lease · Event Log · SQLite · Audit · Keychain  │
│ Workspace Broker · Evidence Index · GitHub Action Broker  │
└───────────────────────┬─────────────────────────────────┘
                        │ 限制能力的任务合同
┌───────────────────────▼─────────────────────────────────┐
│ Untrusted Execution Plane: Agent Runner / Test Sandbox   │
│ Codex Planner / Developer · Read-only Reviewer · Test CI │
└───────────────────────┬─────────────────────────────────┘
                        │ 仅允许的外部调用
             Codex/OpenAI · GitHub · Apple Filesystem
```

**核心不变量**：UI、Planner、Developer、Reviewer 均不得直接更新审批数据库、绕过策略检查或使用 GitHub 管理员凭证；本机 loopback 接口即使只监听 `127.0.0.1` 也必须有可信客户端验证、防 CSRF / Origin 检查与能力边界，不能把 localhost 视为天然可信。

### 5.2 推荐技术栈（M00 后固定版本）

| 组件 | 候选技术 | 为什么 | M00 验证点 |
|---|---|---|---|
| 原生 UI | SwiftUI + Xcode | macOS 集成、文件选择、Keychain | Apple Silicon 构建与最小系统版本 |
| Engine | TypeScript + Node.js LTS | 可组合 Codex SDK / GitHub API | 签名 Helper、进程重启、IPC |
| IPC | XPC/受控本机协议 | 隔离控制与执行权限 | 权限、身份、消息大小与取消 |
| 状态存储 | SQLite（WAL、迁移） | 本地事务与恢复 | 锁、崩溃、损坏、备份 |
| 审核证据 | Content-addressed FS + SHA-256 | 固定 Bundle 与可复核性 | 未跟踪文件、修改后重审 |
| 文件授权 | NSOpenPanel、可持久化 Bookmark（按运行模型适用） | 原生用户授权 | 沙盒、Helper 访问、失效处理 |
| Agent | Codex SDK / `codex exec` | 复用官方本地代理 | 登录、输出、限额、取消 |
| Reviewer | 官方受支持的 OAuth 模型调用或隔离 Codex | 独立审查、机读输出 | 账户资格、输出能力、分发边界 |
| GitHub | 可撤销的最小权限 GitHub App/OAuth | PR/CI/merge 分离 | 计划、仓库授权、Ruleset |
| 包装 | arm64 .app + Developer ID + Notary + DMG | 跨用户设备安装 | Helper 签名、Gatekeeper |

### 5.3 信任域与权限归属

- **T0 Trusted Control**：政策引擎、审批令牌、状态机、密钥与受控 GitHub Adapter；Agent 无写入权限。
- **T1 Untrusted Run**：下载的目标代码、构建命令和 Agent 生成的脚本；不能继承 T0 的完整用户环境、HOME、浏览器凭证或 SSH Agent。
- **T2 External Services**：OpenAI、GitHub、Apple 服务，需检查返回来源、目标、版本及错误，不把页面文字当系统指令。
- **T3 Production / Money**：v1.0 不执行真实资金或未经授权的生产写入；交易相关项目只在 Mock/TEST/Shadow 无实盘权限运行。

### 5.4 SwiftUI / Node Helper 的责任和进程生命周期

SwiftUI 只负责展示、文件选择和用户身份交互；Engine 负责状态、事件、任务管理；Runner 执行受约束的 Codex/测试；GitHub Broker 负责外部写入。App 退出、系统睡眠、网络切换时，需要保存任务检查点并安全暂停。Node Helper 绝不能依赖未签名的外部资源才能在 DMG 安装后启动；内嵌或引用本地工具均需 M00 PoC 与 M14 完整签名验证。

## 6. 项目目录、iCloud 和沙箱

### 6.1 典型目录

```text
~/.../PROJECTS/                              # 用户通过原生选择器挑选的目录
├── Modular-Dev-Orchestrator/                # MDO 源码，和目标项目平级
├── Modular-Dev-Quality-SOP/
└── warehouse-management-system/             # MDO 创建的新项目
    ├── docs/
    ├── src/
    ├── tests/
    └── .github/

~/Library/Application Support/<app-id>/       # 每台 Mac 私有的可信状态
  state.db   policy/  audit/  artifacts/  backups/  bookmarks/
~/Library/Caches/<app-id>/                    # 可重建缓存及非权威文件
```

不能假设 Finder 中 `iCloud Drive / Desktop / PROJECTS` 的实际路径固定为 `~/Desktop/PROJECTS`；必须保存原生选择器返回的真实 URL 和授权能力。项目源码可以由用户决定存放在 iCloud，但临时工作树、任务锁、SQLite 和审计日志不得以 iCloud 同步充当并发协调机制。

### 6.2 授权持久化和目录验证

通过 `NSOpenPanel` 选择目录；沙盒模式按 Apple 规则创建 Security-Scoped Bookmark、解码、检查 stale、`startAccessingSecurityScopedResource()` 与平衡 `stopAccessing...()`；非沙盒发行时也必须验证相应 Files & Folders/TCC 权限，不能假设绝对路径可长期访问。SwiftUI 取得的访问权不自动等同 Node Helper 或测试子进程拥有权限：须明确跨进程授权机制并在 M00 验证。

每次执行前：规范化路径、真实路径、inode/文件 ID 与权限状态；阻止路径逃逸、绝对路径注入、重复项目名、symlink 重定向、TOCTOU 替换及覆盖已有目录；`mkdir` 应使用安全原子创建策略并校验结果。iCloud 占位、未下载、访问被撤销、磁盘空间不足时停在 `BLOCKED_WORKSPACE`，显示恢复入口，不偷偷改为本地其他路径。

### 6.3 跨 Mac 使用

在另一台 Apple Silicon Mac 安装 DMG 后，首次配置重新登录 Codex、Reviewer 与 GitHub，重新选择项目根目录并获取该设备自己的文件授权。项目源码通过 GitHub 导入；运行中的任务、未签名审批、Bookmark、凭证和本地锁不可简单复制成第二台设备的有效运行状态。迁移必须有版本化导出、重建与核对流程，避免双设备同时管理同一活动任务。

## 7. 模型调用、认证和费用边界

### 7.1 Codex Developer

官方允许的本地 SDK/CLI 入口可在用户本人明确登录后调用；不得通过 GUI 自动点击、Cookie 抓取或非公开协议模拟普通 ChatGPT 网页。M00 必须验证 ChatGPT 账户登录、实际任务能力、子进程退出状态、JSON/结构化输出、代理环境、使用限制及升级兼容性。Plus 订阅不是无限 API，Codex 与其他计划使用渠道可能共享额度。

### 7.2 独立 Reviewer 的三条路径

- **A：官方 Sign in with ChatGPT / 适用的受支持模型授权**。区分身份登录与计划使用权限，资格、模型发现、是否允许个人/未来公开分发、刷新 Token 与限额必须通过 M00 真机 PoC；不保证一定能选择网页版正在使用的同名模型，也不授予读取 ChatGPT 历史对话权。
- **B：隔离 Codex Reviewer**。独立进程、不同会话、只读固定快照、无 GitHub 写权限；可作为明确批准的首版兜底。应如实标注它与开发端可能使用同一模型，不等于跨模型交叉验证。
- **C：人工网页版 ChatGPT Review**。通过可导出的审核包人工转交；是明确的退化模式，不伪装成无人值守。

任何模式切换都记录 `reviewer_mode`、授权及实际使用模型；不可从订阅调用失败时静默切到付费 API。API Key 适配器默认禁用，未来必须经过 Owner 明确启用、限额、显示费用和风险批准。

### 7.3 令牌管理

敏感凭证只存 macOS Keychain 或等效 OS 保护存储，不写入目标仓库、Review Bundle、临时截图、日志、GitHub Actions Secrets（除非单独明确授权的部署场景）或发布 DMG。OAuth 的 `state`、PKCE、重定向处理、回调端口/深链合法性、Token 刷新和撤销为 M00/M07 必测契约。公开发行时需要独立评估客户端注册与授权品牌要求。

### 7.4 额度预算

`project_budget`、`module_budget`、`max_agent_turns`、`max_fix_rounds`、`max_runtime`、`max_parallel_runs` 均有默认非无限上限。额度耗尽转 `PAUSED_QUOTA`，网络异常进入带诊断的 `PAUSED_NETWORK`，不能把错误包装成“审核失败”无限重试。

## 8. 状态机、门槛和授权不变量

### 8.1 核心状态

```text
NEW → BASELINE_CHECK → PLAN_RUNNING → PLAN_REVIEW
    ↘ PLAN_FIX / WAIT_APPROVAL_PLAN
    → IMPLEMENT_RUNNING → VERIFY_RUNNING → QC1_RUNNING
    → BUNDLE_VALIDATE → QC2_RUNNING ↔ FIX_RUNNING
    → PR_PREP → PR_CI_WAIT ↔ PR_FIX
    → WAIT_APPROVAL_MERGE → MERGE_RUNNING → POST_MERGE_VERIFY
    → MODULE_ACCEPTANCE → COMPLETED

PAUSED_USER | PAUSED_QUOTA | PAUSED_NETWORK | BLOCKED_WORKSPACE
BLOCKED_EVIDENCE | BLOCKED_SECURITY | RECONCILE_REQUIRED
BLOCKED_HUMAN | CANCELLED | FAILED_TERMINAL
```

不能由 Agent 直接设置最终状态；所有状态变化必须记录 `expected_state`、`task_version`、`event_id`、`actor_id`、`policy_version`、`snapshot_id`、`reason`、UTC 时间戳与结果。非法转换或冲突版本一律拒绝。

### 8.2 G0–G4 与 PR Gate

| Gate | 检查重点 | 失败行为 |
|---|---|---|
| G0 Baseline | 双文档基线、SOP SHA、Owner 决定和风险记录 | 未授权不得启动 Module PLAN |
| G1 Plan | Plan、来源、风险、AC、Reviewer PASS 与动作授权 | 回 PLAN_FIX / WAIT_APPROVAL_PLAN |
| G2 Self/Verify | Unit/Contract/Integration/Failure/UI 适用性、QC1、Bundle 完整性 | BLOCKED_EVIDENCE / FIX |
| G3 Independent QC2 | 结构化 Finding、原始 diff、可信证据与同一 Snapshot | CHANGES REQUIRED → FIX；版本漂移则失效 |
| PR/CI/Merge | PR HEAD、必需 CI 真实执行、GitHub Ruleset、Owner/Policy 许可 | 未批准/检查不符绝不合并 |
| G4 Acceptance | merge commit、Post-Merge 验证、回滚点与状态记录 | 不标 COMPLETE，不开启无授权下一 Module |

### 8.3 原子性和幂等性

本地状态事务与 GitHub 外部调用不可能天然形成单个 ACID 事务。使用 `ExternalOperation` / Outbox 模式：先持久记录意图和幂等键，再执行外部请求，再核对真实结果。PR 创建超时后按分支/标签/幂等键查询远端，不能重新创建第二个 PR；merge 超时后查合并状态及 commit，不能直接再发一次。

### 8.4 有界返工

每阶段分开计数（Plan、QC2、PR）；默认每阶段最多三轮是**初始候选策略**，由 G0 确定配置；达到上限、问题未改善、风险升级、范围改变、主控被触及、证据无法读取时进入 `BLOCKED_HUMAN`，停止自动修复。修复跨越架构/契约/权限边界必须回 G1，必要时回 G0。

## 9. Policy Engine、Human Approval 和控制面安全

### 9.1 风险等级

| 风险 | 示例 | 默认规则（首版） |
|---|---|---|
| R0 极低 | 文档、注释、格式且不改变行为 | 可自动至 PR；merge 人工 |
| R1 低 | 已批准范围的小规模非敏感逻辑/测试 | 可自动至 PR；merge 人工 |
| R2 中 | 数据库迁移、接口契约变更、依赖升级、敏感邻接代码 | Plan/实施授权与合并均须明确人工确认 |
| R3 高 | 鉴权、密钥、策略主控、CI 安全、生产写入、审批逻辑 | 强制 Owner 审批，某些动作 v1.0 永久拒绝 |

等级由受保护路径、实际操作、目标环境、副作用、代码语义以及历史证据共同决定；模型自报 `low risk` 不具有最终效力；分类不确定则提升等级。至少将 `.github/workflows/**`、`policies/**`、`security/**`、`infra/**`、审核/批准及沙箱相关代码设为受保护路径。

### 9.2 审批令牌契约

批准对象至少包含：`approval_id`、`approver`、`project_id`、`repo_id`、`module_id`、`action`、`environment`、`plan_revision`、`baseline_sha`、`head_sha_or_snapshot`、`scope_hash`、`policy_version`、`risk`、`issued_at`、`expires_at`、`decision`、`reason`、`revoked_at`。敏感动作开始前重新验证身份、有效期、撤销、版本、操作和环境；一次性批准不复用于不同动作。拒绝必须记录，Agent 不得写审批表。

### 9.3 紧急停止与更新策略

Owner 具有紧急停止、暂停、撤销和只读恢复入口；Emergency Deny Switch 禁止新外部写入、阻止后续任务。对已经发出的 GitHub 请求不能声称撤回，应该进入 reconciliation。MDO 自举时，正在开发的候选版本不得运行其自身未通过审核的新政策引擎来批准发布。

## 10. Review Bundle、测试证据和独立审核

### 10.1 Bundle 标准

```text
Mxx-review-rNN/
  MANIFEST.json                    # 项目、模块、Base/Head/Snapshot、版本
  IMPLEMENTATION_PLAN.md
  COMPLETION_REPORT.md
  REVIEW.patch                     # 相对批准基线的累计全部变化
  CHANGED_FILES.json               # 新增、删除、改名、二进制及未跟踪文件
  FILES/                           # patch 不足以表达的受审真实内容
  EVIDENCE/                        # 可信测试、UI、失败注入、CI 凭据
  SOURCE_MANIFEST.json             # 真实读取资料与日期
  FINDINGS_HISTORY.json
  HASHES.sha256                    # 逐文件，不包含自身
```

压缩包哈希另记在外部的 `COMPLETION_REPORT` 与 `Artifact` 数据库记录，避免自指校验。所有证据必须关联 `baseline_sha`、`head_sha`/`snapshot_id`、`policy_version`、`test_run_id` 和审核阶段。

### 10.2 Reviewer 判定协议

Reviewer 输出 `PASS`、`CHANGES_REQUIRED` 或 `BLOCKED_UNVERIFIED`，附 `finding_id`、严重等级、位置、原因、修复方向、被审对象 ID、证据覆盖矩阵、AC PASS/FAIL/NOT RUN/NA。仅结构化字段合法不足以证明真实，独立 Evidence Validator 要核对相关文件存在性、哈希、真实测试退出码和版本匹配。

### 10.3 测试凭据与独立性

由受控 Runner/CI 生成 `command`、`cwd`、`env_fingerprint`、`start/end`、`exit_code`、`log_hash`、`snapshot_id`。Reviewer 应区分“实际复现”与“只阅读开发日志”；必要时在独立环境复跑关键场景。每个新的 FIX 版本使旧审核 PASS 在受影响范围内失效。

## 11. GitHub、PR、CI、受控合并及 G4

### 11.1 最小权限策略

Developer 不持有 GitHub 管理员密钥；允许的 PR/推送与合并权限由 GitHub Adapter 单独掌握。v1.0 创建 PR 可自动化但必须匹配已授权的项目和分支；merge 必须由 Owner 对特定 HEAD 给出明确批准。GitHub Branch Protection / Rulesets 和 Required Checks 必须在测试仓库验证，不能假设 API 查询不到保护就等于无保护。

### 11.2 PR 完整检查序列

`G3 PASS → Commit/Push（受控） → PR createOrFind → CI 真实结束 → PR HEAD/Snapshot 匹配 → 审核意见解决 → Policy 验证 → WAIT_APPROVAL_MERGE → merge → Post-Merge tests → G4`。PR 的 CI 不能简单依据 `skipped` 或仅有绿色徽章即判定执行完成；检查必需 job 的适用性、状态和 run ID。

### 11.3 失败与恢复

HEAD 变化、审批过期、主分支被他人更新、CI 未执行/失败、权限不足、保护规则不满足时一律阻断。merge 结果未知必须查询真实 PR 和 merge commit；合并后验证失败则标记 `POST_MERGE_FAILED` 并执行经批准的恢复计划，不自动在主分支做破坏性回滚。

## 12. 数据模型与持久化

### 12.1 核心实体

| 实体 | 核心字段 | 作用 |
|---|---|---|
| Project | id、name、repo、root、baseline、sop、policy | 顶层隔离边界 |
| Module | id、scope、depends_on、AC、status | 模块任务定义 |
| TaskRun | id、module、state、version、lease、budget、retries | 恢复和状态转换 |
| AgentInvocation | id、adapter、session、input_hash、status、error | 调用溯源 |
| Approval | id、actor、action、snapshot、expiry、revoked | 人类授权 |
| ReviewDecision | phase、reviewer、snapshot、result、coverage | 独立审查 |
| ReviewFinding | id、severity、location、resolution | 缺陷闭环 |
| Artifact | hash、path、size、type、retention | 证据保存 |
| TestEvidence | command、exit、logs、snapshot、environment | 测试真实性 |
| ExternalOperation | key、action、target、request/result、reconcile_state | 外部副作用去重 |
| AuditEvent | event_id、actor、policy、timestamp、before/after | 不可悄然改变的历史 |
| DeviceBinding | device_id、workspace_grants、app_version | 跨 Mac 的本机绑定 |

### 12.2 存储安全及迁移

SQLite 使用受控事务、迁移版本和校验。可信状态及审核数据库必须位于每台 Mac 的本地受保护目录而非目标仓库或 iCloud；日志和证据支持可配置保留期和可检查清理。任何 Schema 升级先备份、演练向前迁移，失败则恢复旧版本；不允许应用升级时静默丢弃仍有效的审批链。

### 12.3 IPC/Schema 的最低契约

每条消息含 `schema_version`、`request_id`、`origin`、`capability`、`project_id`、`task_id`、`issued_at` 和结构化 payload；参数有长度、路径、字段枚举限制；受控 UI 发审批动作时，服务端必须重新核验交互身份，不能仅信客户端传来的 `approved=true`。

## 13. 安全威胁模型和防线

### 13.1 核心攻击路径

恶意 README/Issue 中的 Prompt Injection、依赖 postinstall 脚本、Symlink 路径逃逸、任意子进程执行、工作树切换、日志中的 Secret、审核包注入、伪造人工批准、GitHub Token 过宽、MDO 自举时自修改政策、跨设备重复执行。

### 13.2 强制控制

- Agent 文件/网络/进程访问需经操作系统级隔离并做负向测试，**Git worktree 不是安全沙箱**。
- 执行目标代码前检查依赖脚本和网络需求；默认禁止传递用户完整 HOME、SSH_AUTH_SOCK、浏览器资料和 Code Signing 密钥。
- 审核与部署凭证分离；GitHub 仅授权选定仓库；远端凭证失效应进入等待重新登录而非隐藏重试。
- Source Manifest 记录官方资料；不可信网页或仓库文本不能成为 Owner 授权指令。
- 签名、公证、自动更新和发行文件需在干净 Runner / 受控本机发行环境中完成；目标仓库不能直接取到 Developer ID 私钥。

## 14. 原生 UX、页面信息结构与交互约束

### 14.1 全局导航

| 页面 | 核心内容 | 用户动作 | 必须暴露的异常 |
|---|---|---|---|
| Dashboard | 运行中的项目、待审批数量、最新问题 | 打开任务/暂停 | 额度、执行器离线 |
| Projects | 新建、导入、归档、目录和 G0 状态 | 新建/导入/确认基线 | 重名、目录不可写 |
| Project Detail | 目标、模块图、AC、当前 Git 和 PR | 开始模块、查看历史 | 依赖未满足、基线漂移 |
| Runs | 阶段时间线、测试、重试、日志 | 暂停、恢复、终止 | 外部操作结果未知 |
| Reviews | Plan/QC2 Findings、diff、测试证据 | 查看、筛选、导出 | 证据缺失或版本不符 |
| Approvals | 动作、风险、范围、版本、失效时间 | 批准、拒绝、要求修改、撤销 | 旧 SHA、策略变动 |
| Settings | Codex、Reviewer、GitHub、PROJECTS、配额 | 登录/重授权/目录选择 | 凭证失效、iCloud 不可用 |
| Diagnostics | 本机健康、状态一致性、签名与版本 | 运行自检、脱敏导出 | DB 损坏、磁盘满 |

### 14.2 视觉与可访问性

遵循 macOS 原生窗口、侧边栏、标准按钮语义和系统辅助功能；卡片用于风险/执行摘要，不滥用颜色代表唯一状态。必须支持键盘导航、VoiceOver 可识别标签、清晰警告文本、长日志可搜索/复制。审批界面必须展示将发生的**实际动作**及可能副作用，不允许隐藏重要条款在折叠文本里。

### 14.3 原生目录选择器体验

首次启动「选择根目录」→ 显示真实路径与可写检测 → 用户确认 → 创建空测试子目录并安全清理 → 保存授权；单项目覆盖时再次调用系统选择器。权限被撤销时引导用户重新选择同一目录，不主动把项目迁往未知位置。

## 15. 质量属性、SLO 和运维

### 15.1 工程目标（验收时用证据证明）

- 100% P0 安全负向用例通过；没有任何未经授权的 R3 写操作。
- 所有 G1/G3/PR/G4 结论均有版本、Actor、政策、证据引用和时间。
- 任何已接收的 Code Review PASS 都可以追溯到同一固定 Snapshot。
- 故障注入后的恢复不生成重复 PR/merge 及重复受控副作用。
- 首版至少 10 个预定义低风险 E2E 样本完成到人工待合并状态，同时注入失败样本得到正确阻断/修复。
- 支持在第二台干净 Apple Silicon Mac 上安装，并重新授权与导入一个项目。
- 提供合适的 Crash 报告、日志脱敏和问题导出；不承诺未经统计证据支持的成功率或平均节省时间。

### 15.2 日志、诊断和成本

可导出健康报告包含 App/OS/Codex/Node/Schema 版本、实际 Gate 状态、最近错误类型和已脱敏日志摘要；默认不导出源码、完整路径、Token、Cookie、账号凭证或密钥。持续显示运行时长、模型调用次数、修复轮数及预算剩余；订阅额度只显示可真实获取的信息，不虚构精确 Token 余额。

### 15.3 启动、睡眠与关闭

MDO 在用户开启时可执行本地任务，但不得保证 Mac 睡眠或关机时继续运行。支持安全暂停、资源清理和重启对账；不因 SwiftUI 窗口关闭就丢失已确认的外部操作状态。是否允许菜单栏后台执行需由系统设置明确启用，且安全边界一致。

## 16. 发布、安装、升级与未来公开分发准备

### 16.1 v1.0 交付包

目标：`MDO-<version>-arm64.dmg`，内含 `MDO.app` 及经验证的 Helper/嵌入式资源。直接发布前以 Apple Developer ID 签名、Hardened Runtime、公证、staple 验证和 Gatekeeper 实机安装为强制条件。无签名开发包可以仅用于内部调试，不等于可分发最终版。

### 16.2 Apple Silicon 交付与最低版本

只编译 arm64；最低 macOS 版本和 Xcode/Node/SDK 版本在 M00 以目标设备与依赖兼容性决定，G0 不凭猜测指定。每次发行验证干净安装、从旧版升级、数据迁移、重新登录、签名、依赖与受保护 Helper 状态。

### 16.3 未来公开发行的架构准备

v1.0 不需要注册公众账户系统、在线计费或多用户后台，但应避免把开发者用户名、绝对路径、Client Secret 和个人账号打包到 App。未来公开发布须经独立 Release Gate 检查：第三方许可证、官方 OAuth 使用与分发资格、隐私与使用条款、数据收集披露、漏洞报告渠道、签名密钥安全、更新源完整性及用户数据删除。**没有通过这些检查不得把个人试用 DMG 宣称为公开发行就绪。**

### 16.4 跨设备迁移边界

GitHub 用于代码历史；MDO 的本地 SQLite、Token、目录 Bookmark、签名设备绑定和任务 Lease 均每台设备独立。可导出可读的审计历史与项目配置模板，但恢复必须重新检查本机真实仓库、Git HEAD、审批有效性和外部操作状态。阻止两台设备在没有显式协调时同时接管同一活动模块。

## 17. 测试策略、用例与验收体系

### 17.1 三层验证

- **L1 Fake Agents**：模拟 PASS、CHANGES_REQUIRED、超时、429/额度、重复事件、DB 锁，验证状态机与权限，不消耗模型额度。
- **L2 E2E Lab**：独立 `orchestrator-e2e-lab` / TaskBoard Lite，使用真实 Codex 与 Reviewer，覆盖 CRUD、同名任务缺陷、迁移、Prompt Injection 和审批拒绝。
- **L3 Real Repo Pilot**：SOP 测试副本的文档维护，或既有项目低风险非敏感模块；不连接实盘交易权限。

### 17.2 16 个强制故障测试

| ID | 注入条件 | 期望不变量 |
|---|---|---|
| T01 | PLAN PASS | 仅获授权后推进 |
| T02 | PLAN CHANGES REQUIRED | 自动修订，不修改实现 |
| T03 | QC2 BLOCKER | FIX → 重验与新快照 |
| T04 | 修复超限 | BLOCKED_HUMAN |
| T05 | QC2 PASS，CI FAIL | merge 被拒绝 |
| T06 | Owner 拒绝 | 受保护动作未执行 |
| T07 | 审批后 HEAD 改变 | 旧审批失效 |
| T08 | Codex 进程中断 | 保存并恢复检查点 |
| T09 | Mac / App 强制退出 | 无重复外部副作用 |
| T10 | Agent 试图修改 Policy | 操作系统及策略双重阻断 |
| T11 | Bundle 缺材料或哈希错 | G2/G3 不通过 |
| T12 | 订阅额度耗尽 | 暂停，不自动付费 |
| T13 | PROJECTS 授权/iCloud 失效 | 禁止受影响写入 |
| T14 | PR 已创建但网络超时 | 远端对账，不重复 PR |
| T15 | README 中伪造批准 | 无效 Prompt Injection |
| T16 | GitHub 保护条件未满足 | 禁止合并 |

### 17.3 发布验收

所有 P0 安全用例 100% 通过；预先登记的正常/异常任务记录可复核；签名、公证、DMG 安装通过；一个新 Mac 完整走首次设置；所有阻断问题有责任人、风险、版本和处理结论。不能以“模型说 PASS”替代独立 CI 或 Owner 授权。

## 18. 工程路线与责任映射（唯一正式路线）

> **本路线取代早期 M00–M12 草案**。两份正式文档统一采用以下 M00–M15，不得在开发过程中擅自重编号。

| Module | 责任 | 主要产物 | 本文映射 |
|---|---|---|---|
| M00 | 关键技术/授权/沙箱/公证可行性 PoC | 能力矩阵、ADR、条件性决策 | PLAT/AG/WS/REL |
| M01 | 工程骨架、Contract、基础 CI | Xcode/TS 单体仓库、Lint/Test/CI | DOC/PLAT |
| M02 | SQLite 状态与证据索引 | Schema、迁移、审计数据 | WF/EV/OBS |
| M03 | 持久化状态机、队列与预算 | Transition Engine、Lease、Pause | WF/COST |
| M04 | Policy Engine + Approval Store | R0–R3、授权、紧急阻断 | SEC |
| M05 | Workspace Manager + Sandbox | NSOpenPanel、授权、执行隔离 | WS/SEC |
| M06 | Codex Agent Adapter | PLAN/IMPLEMENT/FIX、超时控制 | AG |
| M07 | Reviewer Adapter（先契约+Fixture） | 结构化审核、隔离模式、Fallback | AG |
| M08 | Evidence Builder + Verifier | Bundle、累计 diff、哈希与证据门禁 | EV |
| M09 | 自动返工和恢复协调器 | FIX、Retry、Reconciliation | WF/OBS |
| M10 | GitHub PR/CI/Merge/Post-Merge/G4 | PR/CI、审批绑定、受控合并与验收 | GH/SEC |
| M11 | SwiftUI Onboarding + Projects | 初始配置、项目创建/导入 | UX/PROJ/WS |
| M12 | Runs / Reviews / Approvals UI | 时间线、审核、人工 Gate、诊断 | UX/OBS |
| M13 | E2E Lab、真实低风险试点及安全负测 | AT/T01–T16 证据集 | TEST |
| M14 | DMG、签名、公证、升级与换机验证 | arm64 发行工件 | PLAT/REL |
| M15 | 发布冻结、用户手册与公开准备边界 | Release Notes、Runbook、许可证清单 | REL/DOC |

特别说明：**M10 明确拥有 `merge + Post-Merge + G4` 的工程责任**，但 v1.0 默认 merge 人工审批；M07 可用固定测试 Fixture 构建 Reviewer 契约，待 M08 完成后开展真实 Bundle 端到端验收；M11 可以较早做 UX 原型，但正式工作流不得越过 M04/M05 安全门禁。

## 19. 待确认事项、ADR 和 G0 裁决

### 19.1 已决定，不再重复询问

1. v1.0 自用，未来公开发布预留；不实现首版多用户后台。
2. Apple Silicon arm64、SwiftUI 原生 App、独立 DMG。
3. 本地 Codex 为主要开发端；独立 Reviewer；默认无自动 API 付费。
4. PROJECTS 根目录可选，支持 iCloud，但可信运行数据在本地。
5. G0 负责人确认、R2/R3 强制审批、默认人工 merge。

### 19.2 G0 批准的条件性项 / M00 必须验证

| ADR / 风险 | 尚待证实的事实 | 关闭条件 |
|---|---|---|
| ADR-001 | 最低 macOS、SwiftUI/Helper 打包方式 | 两台目标 arm64 环境验证 |
| ADR-002 | Reviewer OAuth 权限、可用模型、调用限制 | 实际授权、模型发现、结构化响应和失败场景 |
| ADR-003 | Codex SDK/CLI 登录、超时、恢复和版本 | 可重复任务 PoC 和变更测试 |
| ADR-004 | XPC / Helper 安全边界 | 未授权调用、注入与身份验证负测 |
| ADR-005 | iCloud Bookmark 和沙箱/子进程 | 授权持续性、stale、逃逸与访问撤销实验 |
| ADR-006 | GitHub PR、CI、Rulesets | 测试仓库真机权限与检查 |
| ADR-007 | Developer ID 与 notarization | 签名、公证最小 arm64 程序与 DMG 验证 |
| ADR-008 | 个人自用到公开分发的授权与许可证 | 独立 Public Release Readiness 审核 |

### 19.3 G0 审核与 Owner 的最终动作

独立 Reviewer 必须先检查本企划书与配套执行手册在版本、模块顺序、接口、安全、测试、外部事实方面的一致性，输出逐项 Finding 和对应修复建议。Owner 可批准当前基线进入 M00 **可行性验证**，但不能将未验证的外部能力当作已交付。批准记录应保存双文件 SHA、SOP SHA、Reviewer 结论和条件性事项。

## 20. 官方参考资料与版本核验入口

- Apple Developer — [Distributing outside Mac App Store](https://developer.apple.com/macos/distribution/)；[Notarizing macOS software](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)；[Accessing files from macOS App Sandbox](https://developer.apple.com/documentation/security/accessing-files-from-the-macos-app-sandbox)。
- OpenAI Developers — [Sign in with ChatGPT Quickstart](https://developers.openai.com/siwc/quickstart)；[Open-source Sign in with ChatGPT](https://developers.openai.com/cookbook/articles/sign-in-with-chatgpt)；[Codex SDK](https://developers.openai.com/codex/sdk)；[Codex non-interactive execution](https://developers.openai.com/codex/non-interactive)。
- GitHub Docs — [Repository rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)；[Required status checks](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)；[GitHub Apps](https://docs.github.com/en/apps/creating-github-apps)。
- 项目规范 — [Modular-Dev-Quality-SOP](https://github.com/justinchang9562/Modular-Dev-Quality-SOP)。

上述链接仅为发现入口。实施时必须记录实际官方页面、API/SDK 版本、访问日期及差异，不以静态入口清单替代具体契约验证。

---

**终版候选判断**：这份工程企划书已确定产品目标、唯一架构边界、统一 M00–M15、测试与发布基线。**只有通过独立 G0 审核并获 Owner 批准，才能将状态由 DRAFT 改为 APPROVED。**
