# Modular-Dev-Orchestrator

# MDO 开发流程与执行手册

> **版本**：v1.0-G0-Candidate · **日期**：2026-11-12 · **状态**：DRAFT / 待 G0 独立审核与 Owner 批准  
> **唯一配套工程母版**：《MDO 工程企划书》v1.0-G0-Candidate（同版本）  
> **产品策略**：C — 第一版个人使用，技术与分发设计面向未来公开发布  
> **目标交付**：Apple Silicon arm64 原生 SwiftUI App + 独立签名、公证 DMG  
> **适用过程**：人工 SOP 自举开发 → 受控半自动辅助 → 在外部独立门禁下用稳定版 MDO 开发候选版  
> **提醒**：此文档规定操作方法而不代替 Owner 审批；未验证外部接口不得宣称已可用。

---

## 0. 使用须知、版本约束与开工边界

### 0.1 本手册回答什么

本手册回答 **HOW / WHEN / WITH WHAT EVIDENCE**：从 G0 开始，应当由谁执行、先读哪些资料、如何划分 Module、什么时候允许写代码、测试失败怎么办、审核包如何固定、何时允许 Push/PR/Merge、怎样在最终 Apple Silicon Mac 上交付 DMG。

与《MDO 工程企划书》的分工：企划书是 WHAT/WHY 和架构不变量的权威；本手册是实现顺序和每一步执行合同的权威；现有 46 页《MDO 产品与开发手册》为**设计参考**，不可与正式双文档并列形成冲突的唯一事实来源。

### 0.2 文档状态和唯一基线

**当前都是 G0 候选文档，不是已经正式批准的开发基线。** 第一轮 Codex 任务只能是 `G0 BASELINE PREPARATION`，检查文件、梳理冲突、产生待批文档，不开始 M01–M15 的实现。独立 Reviewer 完成 BASELINE Review 后，只有 Owner 能发出 `G0 APPROVED` 或 `CHANGES REQUIRED`；批准时固定双文档 SHA-256、SOP commit SHA、模块表修订和条件性风险。

存档建议：`docs/PROJECT_PLAN.md`（企划书正文）、`docs/EXECUTION_MANUAL.md`（本手册）、`docs/MODULE_ROADMAP.md`（从第 8 章受控提取，不能手改成第三版本）、`docs/SOURCE_OF_TRUTH_POLICY.md`（第 3 章受控摘录）、`docs/PROJECT_STATUS.md`（真实进度）。用户阅读版 PDF 是相应 Markdown 快照的排版，不应成为另一份被单独维护的事实文本。

### 0.3 不可谈判的 12 条规则

1. 一次只处理一个目标 Module；必要的 Mx.n 拆分需记录、审核，不得越级。
2. `PLAN` 只研究与制定方案；在 G1 和 Owner/Policy 授权前不能修改实现。
3. Developer 的 QC1 不能代替 Reviewer 的独立 QC2；Reviewer PASS 不能代替 Owner 批准。
4. 每次审查绑定当前真实 Base/Head/Snapshot、计划修订和测试证据，不相信会话记忆。
5. 缺证据、状态未知、授权失效、秘密泄露或路径越界时 Fail-Closed。
6. 高风险外部写操作先持久化意图，结果未知先查询/对账，禁止盲重试。
7. 不主动覆盖、不还原用户未归属改动；不删除其他项目或主控资产。
8. 官方资料确认外部事实；项目契约确认内部事实；Skill 仅存已验证的可复用经验。
9. Reviewer 只读，不运行具有写入副作用的工具，除非 Owner 单独批准隔离实验。
10. Developer 不接触生产凭证、GitHub 管理员令牌或 Developer ID 私钥。
11. v1.0 merge 默认人工批准，自动付费 API 默认关闭，实盘/资金/生产破坏性动作禁止。
12. MDO 自举开发时必须由**已有稳定受保护版本**和**外部独立 Reviewer/Owner**审核候选版本。

## 1. 开始开发前的工作站和仓库准备

### 1.1 macOS 工程环境

必须在 Apple Silicon 真机上验证 `uname -m` 是 `arm64`，检查 Xcode、Command Line Tools、Git、可用 Node.js LTS、Codex CLI、必要的官方 SDK 登录方式、网络/代理和磁盘。**不得在 G0 擅自写死最低 macOS、Node、Xcode、Codex 版本**；由 M00 形成有证据的兼容矩阵并在 ADR 冻结。

准备独立开发工作区：MDO 源码仓库为 `PROJECTS/Modular-Dev-Orchestrator`；SOP 文档仓库位于其同级目录；所有测试目标项目另设受控 `orchestrator-e2e-lab`，不能把测试数据直接写进 Orchestrator 的真实状态数据库。iCloud PROJECTS 可以存源码，但运行数据库、缓存、锁和临时 Runner 工作区在本机受控目录。

### 1.2 仓库建议结构

```text
Modular-Dev-Orchestrator/
  apps/macos/                        # SwiftUI / Xcode
  packages/engine/                   # 状态机、调度、恢复
  packages/policy/                   # 权限、审批验证
  packages/contracts/                # Schema、错误码、事件契约
  packages/storage/                  # SQLite、迁移、备份
  packages/workspace/                # 用户目录、权限、执行器隔离
  packages/adapters/{codex,reviewer,github}/
  packages/evidence/                 # Bundle、hash、证据核验
  packages/diagnostics/              # 日志脱敏、健康检查
  assets/sop/                        # 固定版本规范
  assets/templates/                  # 新项目基线模板
  tests/{unit,contract,integration,security,e2e}/
  docs/{PROJECT_PLAN.md,EXECUTION_MANUAL.md,MODULE_ROADMAP.md,
        SOURCE_OF_TRUTH_POLICY.md,PROJECT_STATUS.md}
  docs/adr/
  .github/workflows/
```

具体文件名允许在通过 G1 的 Implementation Plan 中细化，但跨层责任与安全边界不得以方便为由改变。`AGENTS.md` 可用于快速引导 Codex 读取两份正式基线及 SOP，而不是复制全部正文。

### 1.3 Git 操作纪律

首次在 Owner 明确允许后初始化仓库及远端。`main` 在每个合并后必须保持可回滚；每个模块在独立 Feature Branch 或 Git worktree 执行。建议分支名 `feat/m00-feasibility`、`feat/m05-workspace-security`，修复必须仍属于当前模块。记录 `base_commit`、`head_commit`、`branch`、`dirty_status`、`untracked_paths`。禁止为打包 Review Bundle 自动生成隐藏 commit 以改变审核基线；若计划使用固定提交供审核，要先获得这类操作的授权。

## 2. G0 — 两份正式基线的建立、审核和批准

### 2.1 G0 输入与动作

输入：本企划书、本执行手册、原产品手册作为参考，以及 `Modular-Dev-Quality-SOP` 的真实固定 SHA。Codex 在独立 `G0 BASELINE` 会话对照两份文本、核对路线图 M00–M15、工程边界、技术风险、Requirement Map，生成 `G0_BASELINE_REPORT.md` 和初始状态文件；若发现必须修改的冲突，输出 `CHANGES REQUIRED` 候选修订，不自己宣称已批准。

### 2.2 G0 必须输出

- `docs/PROJECT_PLAN.md`、`docs/EXECUTION_MANUAL.md` 与其真实文件 SHA。
- `docs/MODULE_ROADMAP.md`：单一 M00–M15 表，来自本手册第 8 章；有依赖、交付、Owner Gate 和 AC ID。
- `docs/SOURCE_OF_TRUTH_POLICY.md`：唯一外部资料优先级和访问方法。
- `docs/PROJECT_STATUS.md`：`phase=G0_REVIEW`、`module=NONE`、`baseline_status=PENDING`、`next_candidate=M00`。
- `docs/adr/`：至少登记尚待 M00 验证的 Reviewer、Helper 安全、文件授权及 DMG 分发 ADR；可将决定暂定为 `PROPOSED`。
- `G0_BASELINE_REPORT.md`：文档职责、冲突修订、需求追踪覆盖、未关闭 Open Questions 和 Owner 决策清单。

### 2.3 独立审核 G0

在独立 GPT Review 会话中上传双文档对应版本、候选目录清单、SOP 固定 SHA 和 G0 Report；Reviewer 检查两个文件是否互相矛盾，是否存在未分配的安全职责、未经验证的技术事实、不可验收的 AC、跨 Module 循环依赖。结论只能 `PASS` / `CHANGES REQUIRED` / `BLOCKED_UNVERIFIED`；**PASS 仅代表技术审核通过**，仍由 Owner 记录 `G0 APPROVED`。

### 2.4 G0 Exit Criteria

`G0-AC01` 双文件同一版号且哈希已记录；`G0-AC02` M00–M15 唯一模块表无重复职责与循环依赖；`G0-AC03` 最低 macOS、Reviewer OAuth、沙箱/Helper、Developer ID 等未经验证事实被标记为 M00 PoC；`G0-AC04` 需求 ID → Module → AC/test 的追踪至少覆盖所有 P0；`G0-AC05` Owner 明确批准并冻结基线。未满足不得进入正式模块实现。

## 3. Source of Truth、官方资料和 OSS/Skill 规则

### 3.1 不同事实分别由谁负责

| 内容 | 最高工程依据 | 验证方法 |
|---|---|---|
| macOS API / Entitlements / Notarization | Apple Developer 官方文档 + 真机行为 | 对应页面、系统版本、代码/日志截图 |
| Codex CLI/SDK、模型接口和认证 | OpenAI 开发者官方入口 + 已授权真机 PoC | 具体 SDK 版本、登录/返回/错误记录 |
| GitHub PR、Rulesets、CI | GitHub Docs + 测试仓库真实权限 | API 返回、Branch Protection 设置和 CI run |
| 项目目标与安全不变量 | 已批准《MDO 工程企划书》 | Requirement ID、G0 SHA、ADR |
| 阶段方法与交付纪律 | 已批准《MDO 开发流程与执行手册》+ SOP | Gate 证据和审查记录 |
| 代码实际行为 | 当前 commit / Snapshot / 测试 | 独立执行器、真实 diff、复现 |
| 已验证工程经验 | Skill、脚本及 refs | 来源、版本、适用条件、实测方法 |

### 3.2 Codex 每个模块的资料读取顺序

先读当前 Module 任务卡和批准基线的相关章节，随后定位当前模块需要的**最新官方 Guide / Tutorial / Example**，再核对精确 Reference、状态、错误、兼容、限制及安全文档。含架构图、流程图、时序图、状态图的官方页面要实际核对视觉行为，不能只读 `.txt` 缺图版本。记录 `Source Manifest`：URL、页面标题、厂商、版本/更新、访问日、适用环境、所确认的规则和冲突处理。网络不可达时如实标记阻塞，不伪造访问。

### 3.3 官方来源发生冲突

先判断版本、适用 OS、产品计划、预览/正式接口区别及发布时间；针对行为冲突设计最小 PoC。模型记忆、GitHub 示例和过时 Skill 不覆盖当前官方资料。无法确认而会影响权限/认证/恢复的行为必须 `UNVERIFIED` 并阻断相关实施或验收，不可为了推进而猜测。

### 3.4 OSS 复用和 Skill 更新

复用级别 `DIRECT / ADAPT / PATTERN_ONLY / DO_NOT_USE`；记录仓库、文件、License、版本、复用方式、验证结果和署名义务。未明确授予复制权限的代码不得直接复制。Skill 只写已经实现、测试、来源可追溯、未来仍适用的规则；冲突时修订或删除旧规则，保留 `Added / Modified / Removed / Pending` 清单。

## 4. 开发会话、Prompt 模式和人工交接

### 4.1 人工 Bootstrap 会话规划

建议 Codex 工程项目内：G0 单独会话；每个 Module 一个主开发会话（必要时 FIX 在同一模块记录中衔接，若重开应恢复真实状态）。ChatGPT 建立独立 `MDO Independent QC` 项目：`G0 BASELINE`、`M00 PLAN`、`M00 IMPLEMENTATION` 等分别审核；不共享 Developer 的会话内推断，始终以提交材料为准。

### 4.2 三种开发模式

**PLAN**：核对范围、官方来源、架构、风险、预期文件、测试、回滚、Open Questions；只允许调研与计划性文档输出；最后 `STOP`。**IMPLEMENT**：要求存在对应版本的 G1 PASS + 适用 Owner/Policy 授权；在沙箱/独立工作区按 Plan 实现、测试、QC1、证据与报告。**FIX**：携带 Finding ID 原文及本轮修复授权；只修本轮问题；若扩大范围/改变权限/合同则回 PLAN，不把 BUG FIX 当自动新功能。

### 4.3 新会话的最小任务卡

| 字段 | 约束 |
|---|---|
| Project / Repo / Workdir | 显示真实路径、分支与远端仓库 |
| Module / Mode / Phase | 唯一模块编号，PLAN/IMPLEMENT/FIX |
| Goal / Scope / Out of Scope / Dependencies | 不得出现“自由发挥下一个模块” |
| Acceptance Criteria | 有可执行验证的 AC ID |
| Baseline / SOP / ADR | 真实文件哈希 / commit SHA / 所需章节 |
| Approved Plan & Review | 修订号、结果、证据位置；未知写 UNKNOWN |
| Allowed Actions | 明确区分本地写入、测试、提交、Push、PR、Merge |
| Bundle Directory | 模块工作区受控 artifact 目录，不依赖桌面随意命名 |
| Budget / Stop Condition | 每阶段修复次数、模型额度和阻断条件 |

### 4.4 完整 Review 返回格式

Reviewer 必须明确审核阶段、被审 SHA/Snapshot、逐项 AC 状态、实际读取的 Bundle 文件、实际复跑与只读证据的区别、问题严重级别、Finding ID、原因和修复方向。通过则写 `PASS` + 适用下一候选阶段及所需授权；失败写 `CHANGES REQUIRED` + 一份可复制给 Codex 的 FIX 指令。不能声称 Owner 已批准，不能建议未经许可直接 merge 或越过模块。

## 5. 固定模块开发循环（21 步）

| 步骤 | 开发端/审核端动作 | 阶段与证据 |
|---|---|---|
| 01 | 锁定当前 Module、Owner 授权、任务卡与 WIP | `TASK_CARD.json` |
| 02 | 核对基线文档 SHA、SOP SHA 和 ADR | `BASELINE_MANIFEST.json` |
| 03 | 读取相关官方资料、版本、Guide/Visual、Reference | `SOURCE_MANIFEST.json` |
| 04 | 读取必要 Skill / OSS 矩阵，检查许可 | `REUSE_MATRIX.md` |
| 05 | 核对 Git HEAD、dirty/untracked、目标工作树权限 | `WORKTREE_STATUS.json` |
| 06 | Codex 只读调研并提出 Implementation Plan | `IMPLEMENTATION_PLAN-rNN.md` |
| 07 | GPT 独立 PLAN Review，返回 PASS/修订 | `PLAN_REVIEW-rNN.md` |
| 08 | Owner/Policy 对 G1 后动作进行授权 | `APPROVAL_RECORD.json` |
| 09 | 创建有界工作区/分支，确认沙箱和预算 | `RUN_CONTEXT.json` |
| 10 | Codex 只实现当前 Module 的批准 Scope | 代码与变更日志 |
| 11 | 运行 Unit/Property/Contract/Integration | 可信测试日志 |
| 12 | 依适用性注入超时、权限、断线、重复、损坏等故障 | `FAILURE_INJECTION.md` |
| 13 | 有 UI 影响就真机启动和截图/辅助功能检查 | `UI_VERIFICATION.md` |
| 14 | QC1 全量检查累计 diff、秘密、许可、过度设计 | `QC1_REPORT.md` |
| 15 | 更新已验证 Skill、ADR 和必要 Docs，未验证待确认 | `SKILL_CHANGES.md` |
| 16 | 更新实际项目状态，标记“实现待审” | `PROJECT_STATUS.md` |
| 17 | 固定 Snapshot，制作 Review Bundle 和校验清单 | `MANIFEST.json` / `HASHES.sha256` |
| 18 | GPT 独立 QC2 审核完整真实改动与证据 | `QC2_REVIEW-rNN.md` |
| 19 | 若失败则 FIX→重测→新 Bundle→重审；有界停止 | `FINDINGS_HISTORY.json` |
| 20 | G3 后按授权做 Commit/Push/PR、真实 CI 与 Merge Gate | PR、CI run、Approval |
| 21 | 通过受控合并与 Post-Merge 验证，G4 验收并 STOP | `ACCEPTANCE_RECORD.md` |

**中途任何权限或版本变更均可使步骤失效，必须回到适当 Gate。** 不能为了凑齐 21 步填写虚假的通过记录；NOT RUN 与 NOT APPLICABLE 必须分开。

## 6. 各 Gate 的机器判定与负责人放行

### 6.1 G0 — 项目基线

**输入**：两份同版工程文件、SOP SHA、风险与条件性技术假设。**技术 PASS**：独立 GPT 对文档做完整性、一致性和可实施性审查。**Owner Gate**：负责人明确批准 G0，签发对应双文档 SHA 的记录。**失败**：不准开始 M00 之外的任何实现；M00 本身也须先获 G0 授权作为 PoC。

### 6.2 G1 — Module Plan

PLAN Review 检查 `Goal/Scope/Out of Scope/Dependencies/AC/Files/Architecture/Source Manifest/Risks/Testing/Rollback/Open Questions`；涉及许可、身份、安全或官方行为不确定时阻断实施。`PASS` + 对应模块动作授权才可执行 IMPLEMENT；计划审查本身不能批准 Push/Merge。

### 6.3 G2 — Tests & QC1

累计全部变化、适用测试的实际记录、Failure Injection、UI Verification、Self Review、Skill 和 Docs、Bundle 完整性都必须满足。任一必要项 FAIL、NOT RUN 或哈希不一致不能将 G2 标记通过。纯文档任务可以对编译测试写 `NOT APPLICABLE`，但必须说明与 Scope 关联的理由。

### 6.4 G3 — Independent QC2

Reviewer 必须查真实差异/证据，列出具体已复核与仅查记录的部分；`CHANGES REQUIRED` 自动生成 FIX 候选，不直接进入 PR；`PASS` 仍需 Policy 授权。修复产生新 HEAD 或 Snapshot 时重新审核相关累计变更，不继承旧 PASS。

### 6.5 PR / CI / Merge Gate

只有受控 GitHub Adapter 可以执行授权 Push/PR/merge。PR HEAD、审核 Snapshot、CI run 必须对应同一版本，GitHub 必需检查必须实际完成且成功。首版 merge 必须得到 Owner 对特定 PR HEAD 的审批；GitHub 绿灯不能代替 Owner 许可，内部 PASS 也不能覆盖 GitHub 保护规则。

### 6.6 G4 — Module Acceptance

合并完成后核对 merge commit、必要回归测试、是否留有阻断 Issue、文档和项目状态是否更新、回滚点是否可用。任何未知或失败都不能标 `COMPLETED`。只有下一模块同时符合批准路线图、依赖条件与风险授权，才可能自动开始；Bootstrap 期一律等待 Owner 下一轮指令。

## 7. 跨模块通用安全、质量和交付契约

### 7.1 运行时安全模式

首版默认 `Supervised`：Agent 自动 Plan/Implement/Test/QC1/QC2/FIX 并在符合策略后创建 PR；merge 由 Owner 审批。将来 `Semi-Autonomous` 仅用于经过评估的 R0/R1；涉及 R3 政策、秘密、认证、生产发布、金融实盘依然强制人工。测试环境和生产环境必须明示，不得自动从 Mock/Shadow 升级真实写能力。

### 7.2 文件和进程隔离

UI 负责选目录和显示；可信 Workspace Manager 创建项目；Agent 只获当前任务需要的路径/命令/网络能力。Git Worktree 仅用于变更隔离，**不能代替 macOS 沙箱/受限进程策略**。绝不能将完整 HOME、Keychain 令牌、SSH Agent、Developer ID 签名证书默认注入不可信项目测试进程。

### 7.3 证据与报告标准

完成报告至少有 Executive Summary、计划版本与批准动作、真实来源清单、主要文件改动、架构说明、单测/契约/集成/失败注入/UI 的 PASS/FAIL/NOT RUN/NA、QC1 发现与处理、Skill/Docs 变化、Git 状态、开放问题、逐项 AC 结果、下一模块只做候选建议。Bundle 包含 MANIFEST、累计 patch、新文件副本、文件 SHA、Runner 证据、审查历史；包生成后检查当前源码与清单相符，不为打包自动提交。

### 7.4 恢复规则

进程异常：先核对真实子进程、工作区状态、快照和任务 Lease；网络 API 超时：先确认远端操作是否已发生；批准过期或 SHA 不符：丢弃旧授权重新询问；磁盘满/DB 锁：暂停而非跳过日志或修改历史；Prompt Injection：从未被 Owner 真实批准的内容一律当普通不可信数据。

### 7.5 非功能测试的统一要求

每个实现模块必须在适用性矩阵中明确 `Unit / Property / Contract / Integration / Failure Injection / UI Verification / Security / CI` 的执行计划、命令、环境、预期和实际。无法执行是 NOT RUN 而非 NA；文档/设计任务不得伪造已运行测试。UI 必须在实际 Mac 上验证导航、错误提示、禁用按钮、Accessibility / VoiceOver 标签与页面状态；UNKNOWN 不能显示为正常。

## 8. 唯一正式模块路线图 M00–M15

本路线与《MDO 工程企划书》第 18 章完全一致，**取代旧版 M00–M12 草案**。所有模块开发前分别执行完整 G1；每个模块结束后执行 G2、G3、PR/CI 和 G4。此章节可以机械提取为 `MODULE_ROADMAP.md`，但任何修改必须同步评审两份文档，不可让第三份文件自成一套流程。

| Module | 开发对象 | 显式依赖 | 对应工程需求 |
|---|---|---|---|
| M00 | 技术/授权/沙箱/签名可行性 PoC | G0 已批准 | PLAT、AG、WS、REL |
| M01 | 工程骨架与统一 Contracts/CI | M00 | DOC、PLAT |
| M02 | SQLite、证据/事件索引 | M01 | WF、EV、OBS |
| M03 | 状态机、队列、预算 | M02 | WF、COST |
| M04 | Policy Engine 与审批核心 | M03 | SEC |
| M05 | Workspace Manager 与 Sandbox | M04 | WS、SEC |
| M06 | Codex Developer Adapter | M05 | AG |
| M07 | Reviewer Adapter（先 Fixture） | M06 | AG |
| M08 | Evidence Builder / Verifier | M07 | EV |
| M09 | 自动返工与故障协调恢复 | M08 | WF、OBS |
| M10 | GitHub/CI/Merge/Post-Merge/G4 | M09 | GH、SEC |
| M11 | SwiftUI Onboarding / Projects | M05 + M10 | UX、PROJ、WS |
| M12 | Runs / Reviews / Approvals UI | M04 + M09 + M11 | UX、OBS |
| M13 | E2E Lab、安全负测、真实 Repo Pilot | M10 + M12 | TEST |
| M14 | arm64 App/DMG 签名、公证、迁移 | M11 + M12 + M13 | PLAT、REL |
| M15 | 发布冻结、文档、未来公开发布准备 | M14 | REL、DOC |

**版本责任重点**：M10 必须包含受控 merge、Post-Merge 核验以及 G4 状态落库；不能因“默认人工 merge”而省略相关实现。M07 先使用受控 Fixture 建立审查适配器，M08 完成后必须用真实 Bundle 回归复验 M07；所有模块不得让不安全的旧组件进入首版发行。

## 9. M00 — Technical Feasibility & Capability Gate

**Goal**：验证未来所有高风险技术集成的现实可行性，产生证据驱动的技术选型，而不是开始完整产品开发。**Dependencies**：已批准 G0。**Out of Scope**：多项目完整 UI、真实业务系统自动开发、长期任务引擎、自动合并。

**本轮 Scope 与任务**：

1. `M00-01` 核对 Apple Silicon、目标最低 macOS 候选、Xcode、Node、Codex、Git、开发者签名资源；输出版本和平台矩阵，不把当前开发机版本当全体最低要求。
2. `M00-02` 通过官方 Codex SDK 或 `codex exec` 完成一次**只读**的测试任务、一次结构化输出、一次受控超时/取消；核对实际登录、命令退出码、重连与额度限制错误。
3. `M00-03` 验证独立 Reviewer A 方案的授权资格、有效模型、结构化输出、撤销和账户额度。若不适用，保留失败证据并通过 ADR 批准 B 隔离 Reviewer/C 人工审核，不假装 OAuth 成功。
4. `M00-04` 验证 SwiftUI 与最小 Node Helper 的进程通信、身份验证、取消、异常关闭以及嵌入式执行组件签名可能性。
5. `M00-05` 使用 NSOpenPanel 选择用户目录，实验沙盒/非沙盒访问策略、Bookmark 重新启动、Helper 文件访问、授权撤销、iCloud 占位、Symlink。
6. `M00-06` 对隔离 GitHub 测试仓库查询 PR、CI、Rulesets/Branch Protection，可测试无副作用的只读接口和最小权限；不对主仓库擅自合并。
7. `M00-07` 生成一个最小 arm64 `.app` / DMG 签名与公证可行性小样，确认当前拥有有效 Developer ID 还是需要为发行前补齐；不能用本地开发签名代替正式公证通过。

**交付**：`M00_CAPABILITY_MATRIX.md`（每条 VERIFIED/FAILED/UNVERIFIED）、`M00_SOURCE_MANIFEST.json`、`M00_AUTH_POC.md`、`M00_SANDBOX_POC.md`、`M00_NOTARIZATION_POC.md`、ADR-001..008 决议、测试代码与日志。**风险**：账户资格变化、Apple Entitlement/Helper 不兼容、产品私有分发与未来公开发布条件不同。

**验收**：`AC-M00-01` Codex 真机调用/退出/结构化返回有可复现记录；`AC-M00-02` Reviewer 选定已验证主路径或明确批准降级且标注自动化能力受限；`AC-M00-03` 文件授权含重启、撤销和 symlink 负测；`AC-M00-04` SwiftUI ↔ Helper 和 arm64 小样签名测试有记录；`AC-M00-05` GitHub 权限/保护能力矩阵可检查；`AC-M00-06` 所有未解风险登记在 ADR，G1 不伪造已验证结论。若缺关键能力须重新规划，不允许无条件推进 M01。

## 10. M01 — Engineering Foundation & Contracts

**Goal**：建立可增量交付的 SwiftUI + Engine 单一源码仓库及机器可验证契约。**Dependencies**：M00 条件满足。**Out of Scope**：实现真实 Codex Agent、审批执行、用户完整界面。

**Scope**：建立 `apps/macos`、`packages/{engine,policy,contracts,storage,...}`、测试与文档结构；锁定 arm64 构建配置、Node/Xcode 版本、依赖锁文件；统一 lint、typecheck、unit、contract、Xcode build 命令；定义 `TaskCard`、`RunStatus`、`ReviewResult`、`ApprovalRequest`、`ArtifactManifest` Schema 的第一版；编写带清晰版本前缀的错误码和日志脱敏基础；配置真正执行检查的 GitHub Actions（不依赖 skipped 伪通过）。

**接口**：`schema_version` 必填，校验器对额外未知字段、路径长度、无效时间戳、非法枚举有确定行为；Swift/TS 类型变更必须通过契约测试。**失败注入**：非法 Schema、缺 Node/工具、平台不支持、构建脚本错误、CI job 被禁用或 skipped。**交付**：最小可构建 `.app` 开发版、CLI smoke、schema package、CI run ID、工程 Runbook。

**验收**：`AC-M01-01` 所有必需命令在干净 arm64 工作环境可复现；`AC-M01-02` 无效 TaskCard/ReviewResult 被拒绝；`AC-M01-03` SwiftUI/TS IPC 共享契约无兼容性漂移；`AC-M01-04` CI 针对 PR 的真实 HEAD 跑完必要检查；`AC-M01-05` 测试和构建过程不需要硬编码个人路径/密钥。

## 11. M02 — Persistent Storage, Artifacts Index & Migrations

**Goal**：实现可信本地持久化和证据索引，不依赖 iCloud 协调运行状态。**Dependencies**：M01。**Out of Scope**：自动执行 Agent 任务、UI 和实际批准决策。

**Scope**：SQLite WAL/事务、迁移版本表、Project/Module/TaskRun/AgentInvocation/Approval/ReviewDecision/ReviewFinding/Artifact/TestEvidence/ExternalOperation/AuditEvent/DeviceBinding 表；唯一索引约束 event_id、operation_key；存放本地 Application Support，缓存另设 Caches；证据文件采用 content-addressed 索引，保证 DB 和文件之间具备 reconciliation；支持备份、损坏检测和恢复/只读诊断。

**强制约束**：不可把审批从未发出改写为已批准；事件不可通过普通 UI 操作静默删除；保持迁移前版本副本。对敏感信息实行字段级脱敏，不存原始 OAuth Token。

**失败注入**：进程在事务提交前后被 kill、磁盘满、SQLite locked、迁移中断、证据文件丢失、重复事件。**交付**：迁移脚本、Schema 图、恢复测试与备份 Runbook。

**验收**：`AC-M02-01` 非法重复事件原子拒绝；`AC-M02-02` 崩溃重启数据库状态可解释且无半笔审批；`AC-M02-03` 坏证据进入 BLOCKED_EVIDENCE 而不伪造 PASS；`AC-M02-04` 升级迁移失败可以无数据损失退回已验证版本；`AC-M02-05` 权威 DB 不落 iCloud 默认同步目录。

## 12. M03 — Deterministic Workflow State Machine

**Goal**：把 G0–G4、重试、预算和 WIP 做成确定性状态机。**Dependencies**：M02。**Out of Scope**：真实调用 Codex、Reviewer、GitHub。

**Scope**：定义允许的状态转换、不可变事件、任务租约、版本乐观并发、队列与单项目单模块 WIP；`RUNNING / PAUSED / BLOCKED / RECONCILE_REQUIRED / COMPLETED` 权限区分；拆分 Plan/QC2/PR 重试计数器；提供取消、超时、恢复与 watchdog；使用 Fake Agent 返回结果全面测试。

**不变量**：任何 Reviewer PASS 只生成审核事实，不能直接写 `MERGED/COMPLETED`；状态更新必须含 old_state 与任务版本；外部操作不是数据库事务的假象；一项目不能同时让两条执行链写同一 Module。

**失败注入**：重复事件、乱序回调、两任务竞争 Lease、租约过期、超时恢复、状态版本冲突、Mac 杀进程。**交付**：状态转换表、Property-Based Tests、非法转换矩阵、恢复说明。

**验收**：`AC-M03-01` 全部非法转换被拒绝；`AC-M03-02` 同项目 WIP 约束由数据层保障；`AC-M03-03` 重复事件/重试不重复状态副作用；`AC-M03-04` 使用持久任务记录恢复并识别外部 UNKNOWN；`AC-M03-05` 预算和超轮数进入正确 BLOCKED 状态而非死循环。

## 13. M04 — Policy Engine & Human Approval Core

**Goal**：实现独立于模型的最小权限控制面。**Dependencies**：M03。**Out of Scope**：完整 SwiftUI 审批卡片（M12）、生产环境写权限。

**Scope**：R0–R3、受保护路径、操作/环境风险、默认拒绝规则、Approval Store、批次/一次性授权、版本/Scope Hash/有效期绑定、批准/拒绝/撤销、紧急 Deny Switch；安全地采集 Owner 身份与本机解锁状态；对跨项目、重放、过期、SHA 变化和权限升级做负向测试。Policy 配置位于 Agent 无写权限的 T0 控制面。

**必须避免**：AI 输出“APPROVED”触发真实授权；Reviewer PASS 覆盖 Owner 拒绝；管理员 Token 注入 Agent。批准 UI 与真正动作执行时都必须再次验权；Policy 自身更新属于 R3。

**失败注入**：审批重放、伪造 Owner、变更 HEAD、过期 Token、撤销后使用、政策文件篡改、同时申请两种不同操作。**交付**：Policy Schema、审批契约、访问控制测试、应急停止验证报告。

**验收**：`AC-M04-01` 所有未经授权 R2/R3 写请求被拒绝；`AC-M04-02` 旧 SHA/过期/撤销批准不可使用；`AC-M04-03` Agent 进程无法写 Policy/Approval Store；`AC-M04-04` 紧急停止阻断新外部写操作；`AC-M04-05` 只读查看与写入审批分离。

## 14. M05 — Workspace Manager, Native Picker & Execution Sandbox

**Goal**：安全地创建/导入多个项目，在 macOS 许可范围内隔离源码和不可信任务执行。**Dependencies**：M04。**Out of Scope**：完整项目列表 UI（M11）、任意用户目录全盘扫描。

**Scope**：NSOpenPanel 选根目录、Bookmark/实际文件授权保存、stale 检查、App/Helper 对应访问模型、`PROJECTS/<safe-slug>` 原子创建及可配置单项目目录、同名检测、真实路径与 Symlink 解析、防 TOCTOU、防 `../`、磁盘空间、iCloud 占位、只读目录、跨项目隔离；沙箱 Runner 限制 HOME、凭证、网络、进程/CPU/时长，使用 worktree 但不把 worktree 当安全隔离。

**操作流程**：用户选文件夹 → MDO 检查存在性/授权/可写/空间 → 显示最终真实路径 → Owner 确认 → 原子创建项目子文件夹 → 写最小基线与状态 → 出错补偿清理仅限本轮确实创建的内容。不同 Mac 重新选择根目录和重新授权，Bookmark 不直接移植。

**失败注入**：目录访问撤销、Bookmark stale、子进程拿不到文件授权、iCloud 离线未下载、符号链接绕到 `~/.ssh`、并发同名目录创建、空间耗尽、恶意 slug、创建中断。**交付**：Workspace Service/Swift 桥接、隔离 Runner、路径策略、权限诊断页面的基础组件。

**验收**：`AC-M05-01` 重启后仍能打开已授权根目录或可解释地要求重授权；`AC-M05-02` 越界路径/符号链接/同名覆盖全部拒绝；`AC-M05-03` 失败不得破坏已有项目；`AC-M05-04` iCloud 权限撤销时不改写其他路径；`AC-M05-05` Runner 不能读取控制面审批 DB、Token 或其他项目源码；`AC-M05-06` 第二台 Mac 的路径可不同且可独立配置。

## 15. M06 — Local Codex Developer Adapter

**Goal**：以官方程序化入口控制 Codex Plan、Implement、Fix 任务。**Dependencies**：M05。**Out of Scope**：Reviewer 审核与 GitHub merge。

**Scope**：能力探测、账户登录状态检查、CLI/SDK 兼容适配、任务启动/取消/超时、stdout/stderr 结构化解析、退出码分类、环境净化、可用工具策略、工作树访问范围；实现 `AgentAdapter.run(task, context, workdir, limits)`，返回 `invocation_id`、实际模型/模式、状态、结构化输出、证据路径、错误类型。保证不重复执行未知可能产生副作用的命令。

**安全**：不依赖模拟 Codex App 点击；不读取浏览器 Cookie；不自行导出账户凭证；将用户已有 ChatGPT 额度视为受限资源。阶段模式强制：PLAN 禁止实现写入；IMPLEMENT 要有 G1 与 Policy 授权；FIX 需指定 Finding ID 和修复范围。

**失败注入**：登录过期、网络代理切换、429/额度、进程被 kill、恶意工程脚本读取 Keychain、输出格式畸形、模型版本升级。**交付**：CodexAdapter、Adapter Contract Tests、兼容矩阵、Runbook。

**验收**：`AC-M06-01` Plan/Implement/Fix 三种模式返回可解析结果；`AC-M06-02` PLAN 越界实现被阻止；`AC-M06-03` 账户额度/网络异常安全暂停，不切换付费 API；`AC-M06-04` 子进程取消和重启能识别结果未知状态；`AC-M06-05` Agent 无法超出 M05 沙箱授权工作树。

## 16. M07 — Independent Reviewer Adapter

**Goal**：实现与 Developer 隔离的正式 Review 接口；先以不可变 Fixture 构建，再与 M08 真包集成。**Dependencies**：M06。**Out of Scope**：由 Reviewer 自行 merge、修改目标代码或签发人工审批。

**Scope**：接口 `review(reviewRequest): ReviewResult`，支持 `BASELINE/PLAN/IMPLEMENTATION/PR/POST_MERGE`；检测官方 OAuth 适用性，使用 M00 已批准的方案 A/B/C；建立独立会话和只读快照；返回结构化 `PASS / CHANGES_REQUIRED / BLOCKED_UNVERIFIED`、Finding ID、严重程度、证据覆盖、AC 状态、修复指令；验证异常、不完整响应、模型切换、错误分类及用量上限。

**隔离性测试**：向开发报告植入虚假的“全部 PASS”声明、让真实 TestEvidence 为 FAIL，Reviewer 必须检查版本与原始证据；模拟 Reviewer 尝试修改目标工作区，文件系统应拒绝。使用固定的已知缺陷 Fixture 做漏检/误报记录，不能用一次 PASS 表示高可靠性。

**失败注入**：OAuth 不可用、授权撤销、缺少结构化输出、Reviewer 产生无证据 PASS、超时、只读权限被绕过。**交付**：ReviewerAdapter、OAuth/隔离模式配置、Review Schema、独立性测试与错误指南。

**验收**：`AC-M07-01` 结构化结论和 Finding 有 Schema/版本检查；`AC-M07-02` Reviewer 无代码或审批写权限；`AC-M07-03` 无证据的 PASS 不能推进 G3；`AC-M07-04` OAuth 失败按批准方式转 B/C 或暂停，不偷偷收费；`AC-M07-05` 在 M08 完成后使用真实 Review Bundle 对此模块作补充回归。

## 17. M08 — Evidence Builder & Verifier

**Goal**：让每次 QC2 审核对象真实、完整、可复验。**Dependencies**：M07（使用 M07 契约的固定 Fixture；完成后回归真实审核）。**Out of Scope**：自动生成 GitHub PR、可用自然语言证明缺失的测试。

**Scope**：统一 Bundle Manifest、相对批准 Base 的累计 patch、已提交/暂存/未暂存/未跟踪文件覆盖、增加/删除/改名和二进制文件清单、源码快照固定、SHA-256、Runner TestEvidence、官方来源、Review History、外部包校验；核对 Bundle 和当前受审代码是否同版本；存 content-addressed store 并绑定任务/政策版本。

**证据真实度**：可信 Runner 记录命令、cwd、环境摘要、退出码、日志哈希；不得让 Developer 自己编辑的 JSON 直接代替执行器证据。Reviewer 必须显示“独立复跑”与“仅审阅开发记录”的区别。

**失败注入**：改包内文件、删除未跟踪文件、故意漏变更、替换 git HEAD、测试日志哈希不符、压缩包校验与记录不一致。**交付**：Bundle Builder/Validator、样本 ZIP、完整性测试、与 M07 联合测试。

**验收**：`AC-M08-01` 所有累计变更被覆盖且清单可追踪；`AC-M08-02` 任何关键哈希/快照不一致阻断 G2/G3；`AC-M08-03` NOT RUN 不会被伪装 NA；`AC-M08-04` M07 真实 Bundle 联合回归通过；`AC-M08-05` 报告和 ZIP 不含 Token/敏感密钥。

## 18. M09 — Bounded Fix Loop & Recovery Coordinator

**Goal**：形成端到端 PLAN/IMPLEMENT/QC2/FIX 有界循环与异常对账机制。**Dependencies**：M08。**Out of Scope**：未经审查的自动合并、无限修复尝试。

**Scope**：根据 Reviewer Finding ID 创建 FIX TaskCard，保留原文与受限动作；修复后重新执行受影响测试/QC1/Bundle/QC2；分阶段重试计数、预算控制、风险升级回 G1；Crash/Network/Quota 恢复协调器在重启时对照数据库 Lease、进程、工作树、Snapshot 和外部请求；未知副作用标记 `RECONCILE_REQUIRED`，不得当作失败直接重做。

**失败注入**：同一 Finding 连续三轮无法解决、Fix 修改 Scope、Plan 版本漂移、Mac 重启、CI 卡死、额度耗尽、任务被手动撤销。**交付**：Repair Coordinator、Recovery Runbook、故障模拟记录和恢复结果。

**验收**：`AC-M09-01` 审核失败自动生成可追踪 FIX、问题关闭历史完整；`AC-M09-02` 超限/范围改变自动停止；`AC-M09-03` 重启后不重复已知有副作用动作；`AC-M09-04` 额度/网络状态恢复需重新校验版本和批准；`AC-M09-05` 对数据损坏或未知外部状态执行保守阻断。

## 19. M10 — GitHub PR/CI, Controlled Merge, Post-Merge & G4

**Goal**：实现与真实 GitHub 的端到端可审计交付，明确承担旧计划遗漏的 merge / Post-Merge / G4。**Dependencies**：M09。**Out of Scope**：默认自动合并、绕过 Branch Protection 或真实生产部署。

**Scope**：最小权限 GitHub 授权与撤销、受控 commit/push/createOrFindPR、PR HEAD/BASE、Actions 状态和必需检查真实执行核验、审查问题解决、Policy 验证、`WAIT_APPROVAL_MERGE`、Owner 绑定指定 SHA 的授权、满足规则后受控 merge、真实 merge commit 查询、Post-Merge 回归、G4 Acceptance Record。保持 GitHub 外部请求的 Outbox/幂等键及未知结果对账。

**特殊规定**：G3 通过后如果生成新的提交，必须检查审核 Snapshot 是否覆盖实际 PR HEAD；必要时重审。不得以 `CI skipped`、`workflow completed` 而不是必需 job 真正 PASS 作为通过依据。合并失败、主分支变化或保护规则不满足，进入阻断/重新审核。

**失败注入**：PR 已创建但请求超时、Push 后 HEAD 改变、Check skipped、人工拒绝、GitHub 403、merge 已发生但请求返回超时、Post-Merge 失败。**交付**：GitHub Adapter、PR/CI 对账工具、merge 审批契约、G4 验收与恢复报告。

**验收**：`AC-M10-01` PR 创建超时不产生重复 PR；`AC-M10-02` CI 失败/skipped/不对应 HEAD 时阻止合并；`AC-M10-03` 未经 Owner merge 批准且没有自动策略许可时禁止 merge；`AC-M10-04` 合并未知时查询真实状态；`AC-M10-05` Post-Merge 不通过不得标 G4 COMPLETED；`AC-M10-06` GitHub 平台规则与内部 Gate 双重生效。

## 20. M11 — Native SwiftUI Onboarding, Projects & Workspace Settings

**Goal**：让 Owner 可以通过可用、原生的 UI 安装配置及启动 G0 新项目/导入项目。**Dependencies**：M05、M10。**Out of Scope**：完整运行/审核可视化（M12）、云端账号服务。

**Scope**：SwiftUI App 主窗口、侧边栏、首次引导（CPU/OS/工具链、Codex、Reviewer、GitHub、PROJECTS）、NSOpenPanel 目录选择及路径预览、原生权限状态、创建/导入项目、G0 基线摘要、项目列表与详情、运行前检查、基础诊断入口、错误文案及可访问性。支持“个人使用”默认情景，但所有路径和账户都从本设备设置读取，不能包含开发者硬编码信息。

**UI Verification**：在目标 Apple Silicon Mac 真机点击新建/导入/取消/改目录/重新授权；检查深色与浅色模式的文本可读性、键盘导航、VoiceOver、失败提示是否明确。**失败注入**：无 Git、Codex 未登录、PROJECTS 目录已删除、iCloud 脱机、同名目录、GitHub 认证失效。**交付**：SwiftUI Screen、状态绑定层、UI 自动测试与截图。

**验收**：`AC-M11-01` 新用户可不使用终端完成首次配置；`AC-M11-02` PROJECTS 默认目录保存并在重启后使用或明确提示失效；`AC-M11-03` 已存在目录不被覆盖；`AC-M11-04` G0 未批准时 UI 不开放实施；`AC-M11-05` 错误状态、按钮禁用和辅助功能可验证；`AC-M11-06` 不依赖固定用户名/路径。

## 21. M12 — Runs, Reviews, Approvals & Diagnostics UI

**Goal**：让 Owner 看到每一步的证据、失败和安全决策，并能准确批准/拒绝/暂停。**Dependencies**：M04、M09、M11。**Out of Scope**：由 UI 替代 Policy Server 的最终判断。

**Scope**：Run Timeline、阶段状态与进度、Reviewer Finding 列表、累计 diff/Bundle/测试证据索引、审批卡片（实际 action/repo/module/SHA/expiry/risk）、批准/拒绝/要求修改/撤销、Emergency Stop、诊断与安全日志导出、配额警告、运行恢复向导。所有 UI 操作发往可信服务端完成身份/版本二次校验，不接受“按钮点了所以已授权”的本地假状态。

**UI 测试**：未知 API/CI 状态不显示绿色；Stale Snapshot 不能显示新鲜；拒绝后按钮不可变成已批准；Approval 时间/风险/操作一屏可见；运行日志能筛选 Finding ID；能区分内部 PASS 与 Owner 批准。

**失败注入**：批准前 SHA 变化、并发两个审批窗口、撤销后重复点击、网络/IPC 失联、审查证据损坏、App 强制结束后打开同一任务。**交付**：Runs/Reviews/Approvals/Diagnostics 四组页面、Accessibility QA、状态与服务端契约测试。

**验收**：`AC-M12-01` 每个 Gate 状态、原因、证据可被直观定位；`AC-M12-02` 授权失效、重复、伪造点击被服务端拒绝；`AC-M12-03` 四种审批决策正确写入不可悄改审计事件；`AC-M12-04` 紧急停止真实阻断新操作而非仅 UI 变灰；`AC-M12-05` UI 崩溃不会丢失任务真实状态；`AC-M12-06` 诊断导出默认脱敏。

## 22. M13 — E2E Lab, Security Negative Tests & Real Repo Pilot

**Goal**：在完全隔离环境中证明安全闭环，而不仅是单次成功演示。**Dependencies**：M10、M12。**Out of Scope**：真实交易、生产发布、直接修改正式 SOP 主仓库。

**L1 Fake-Agent 测试**：T01–T16 全部可复现，记录失败预期和实际状态；对状态机/审批/路径使用 Property Tests。**L2 E2E Lab**：专用 TaskBoard Lite 五项任务：README（R0）、CRUD（R1）、同名删除 Bug（QC2）、字段迁移（R2）、修改策略/CI（R3）。验证真实 Codex、Reviewer、Runner、Review Bundle、PR 和 CI。**L3 Real Repo Pilot**：仅使用 SOP 的独立测试副本进行低风险文档修改，必要时再使用其他项目的 Mock/Shadow 分支，不连接真实交易权限。

**故障覆盖**：至少包括 T01–T16：超时、进程杀死、审批失效、篡改证据、目录权限/iCloud、PR 重复、Prompt Injection、CI skipped、GitHub Ruleset 拒绝。不能通过模型写“测试已通过”作为结果；必须检查 DB 状态、工作树、PR 数量、CI run 和实际权限事件。

**验收**：`AC-M13-01` 所有 P0 负向授权安全用例 100% 通过且 0 次越权；`AC-M13-02` 至少 10 个事先登记的低风险端到端样本达到正确结果；`AC-M13-03` 每个失败用例可通过证据复核；`AC-M13-04` Reviewer 的已知缺陷命中/漏检及假阳性有独立记录；`AC-M13-05` 不在实盘或生产环境运行测试；`AC-M13-06` 不把小样本成功率宣称为统计可靠性。

## 23. M14 — Apple Silicon App Packaging, Developer ID, DMG & Migration

**Goal**：构建可在其他 Apple Silicon Mac 安装的正式工件。**Dependencies**：M11、M12、M13。**Out of Scope**：Intel Binary、Mac App Store、公众计费与 SaaS。

**Scope**：arm64 Release 编译，校验所有 App/Helper/二进制架构与签名；Hardened Runtime、Developer ID Application、Notary 提交、ticket staple、Gatekeeper 检查、DMG 创建和校验；干净 macOS 环境首次启动与设备权限重建；从旧版升级备份/迁移/回滚；网络断线时公证失败可解释；签名私钥不进入 Agent 测试沙箱。

**跨设备验证**：在另一台 Apple Silicon Mac 解压/安装 DMG，选择不同 `PROJECTS` 路径，重新登录 Codex/Reviewer/GitHub，导入一份受控仓库并运行只读 smoke，查看签名/公证状态。**失败注入**：Helper 未签名、签名被破坏、安装到无权限目录、老 DB Schema、离线授权、旧 iCloud Bookmark。

**验收**：`AC-M14-01` `.app` 和嵌套可执行程序签名验证通过；`AC-M14-02` Apple Notary Accepted、staple/Gatekeeper 真机通过；`AC-M14-03` 第二台 Mac 无需重新编译即可安装；`AC-M14-04` 新设备不需要复制旧设备 Token/Bookmark；`AC-M14-05` 旧数据迁移/回滚有完整证据；`AC-M14-06` 最终 DMG SHA 和版本/发行源提交可核验。

## 24. M15 — Release Baseline, Documentation & Public-readiness Boundary

**Goal**：完成个人自用 v1.0 的发行交付、支持与已知限制声明，同时保留未来公开发行前置清单。**Dependencies**：M14。**Out of Scope**：把首版个人使用软件直接包装为面向公众的收费产品。

**Scope**：冻结 Release Candidate 源码、SOP/工程双文档 SHA、正式构建/签名/公证证明、Release Notes、已知问题、安装卸载、目录与 iCloud FAQ、账号/费用说明、恢复和诊断、备份与回滚 Runbook、必要开源许可证清单、敏感信息检查与事故处理。标明 v1.0 只供本人使用、未来公开发行尚需专门安全/法律/授权 Gate。

**未来公开发行准备**：独立检查 OAuth 公开分发资格、隐私披露、开源依赖/授权、签名证书保管、更新机制、许可证及用户支持机制；但不要求 v1.0 立即上线账号系统或公开下载站。**失败注入**：发行包中发现硬编码本机路径/账号、私钥、过期文档、无法验证的 DMG 校验值、卸载误删用户项目。

**验收**：`AC-M15-01` 使用者可独立安装、登录、选 PROJECTS、导入和开始受控任务；`AC-M15-02` Release Notes 与实际版本/限制一致；`AC-M15-03` 所有分发包和源码完成秘密/许可证扫描；`AC-M15-04` 备份/回滚/卸载不破坏已有用户项目；`AC-M15-05` Public Release Checklist 标记未来待办而不把它们伪装成已交付；`AC-M15-06` Owner 完成最终 Release Gate 批准。

## 25. Bootstrap：在 MDO 尚未运行前如何开发它自身

### 25.1 推荐会话结构

```text
Codex Project: Modular-Dev-Orchestrator
  G0 - Baseline Preparation
  M00 - PLAN / IMPLEMENT / FIX
  M01 - PLAN / IMPLEMENT / FIX
  ... M15

ChatGPT Project: MDO Independent Review
  G0 - BASELINE Review
  M00 - PLAN Review
  M00 - IMPLEMENTATION QC2
  M00 - PR / POST_MERGE（需要时）
  ... 后续模块独立会话或具备独立边界的审查轮次
```

首期人工完成每次 `Plan → GPT Review → 负责人批准 → Implement → Bundle → GPT QC2 → PR/CI → G4` 转交。这不是架构失败，而是安全的 Bootstrap 过程。MDO 组件完成后可以让**稳定版本**承担机械转交、证据校验和审计；不能由候选实现自评自己的政策修改。

### 25.2 最初目录的生成顺序

1. 在 Finder 中手动创建已确定的 `PROJECTS/Modular-Dev-Orchestrator`；如果已有空目录，保持，不另建重复目录。
2. 把两份正式 Markdown 和阅读版 PDF 复制到项目 `docs/`，记录原始 SHA（复制后需复核）。
3. 在 Codex 开一个 G0 BASELINE 会话，运行下方第 26.1 节的启动 Prompt；只可在 G0 范围内创建/整理规范和初始状态文件。
4. 对比 `Modular-Dev-Quality-SOP` 当前主分支，取得真实 commit SHA。不要仅记录“main 最新”作为长期基线。
5. 在独立 GPT 审核会话执行第 26.2 节；修订过程逐版留存，不覆盖原件不记录修订号。
6. 独立审核 PASS 后，Owner 才能批准 G0，并记录双文档哈希；然后在独立 M00 会话启动 PLAN。不要因为文档名称叫 v1.0 就当作已批准。

### 25.3 MDO 自举的不可突破边界

候选 MDO 可辅助运行低风险测试，但不能独自批准自己正在改动的 Policy Engine、Reviewer Adapter、升级/签名、MDO 自身 GitHub Rulesets 或 Owner 身份控制。稳定版控制面和候选版必须分离安装路径、数据库、Keychain 命名空间、凭证范围与政策版本。升级发布最后由外部独立审核 + Owner 合并/发行批准。

## 26. 可直接复制的标准开发与审核 Prompt

以下模板是与通用 SOP 兼容的**MDO 专属补充**。每次填写真实路径、版本、当前模块和批准动作；未获取的结果写 `UNVERIFIED` / `NOT RUN`，禁止编造。

### 26.1 Codex — G0 文档基线准备 Prompt

```text
你是 Modular-Dev-Orchestrator 的工程规划执行者。
本轮模式：G0 BASELINE PREPARATION。
目标项目：我指定的 PROJECTS/Modular-Dev-Orchestrator 工作区。
当前允许动作：读取和校准项目规范；仅在 Owner 已授权的目录中创建/修改 G0 文档。
严禁开始 M00–M15 的业务实现，也不得自动 push、PR、merge。

请实际读取：
1) 《MDO 工程企划书》v1.0-G0-Candidate。
2) 《MDO 开发流程与执行手册》v1.0-G0-Candidate。
3) Modular-Dev-Quality-SOP 的 README、G0–G4 执行规范和 Developer/Reviewer 模板；记录 SOP commit SHA。
4) 仓库当前真实 Git 状态、已有文件和任何未归属改动。

工作要求：
- 审查双文档是否在范围、Apple Silicon、SwiftUI/Engine、权限和 M00–M15 上一致。
- 把本轮两个文件整理为 docs/PROJECT_PLAN.md 和 docs/EXECUTION_MANUAL.md，保留来源与 SHA。
- 按执行手册第 8 章生成单一 docs/MODULE_ROADMAP.md，不另造模块序列。
- 生成 docs/SOURCE_OF_TRUTH_POLICY.md、docs/PROJECT_STATUS.md 和必要的 PROPOSED ADR。
- 标出所有未经 M00 实测的外部行为，不把它们写为已实现。
- 输出 G0_BASELINE_REPORT.md：文件清单、冲突、修订、Requirement-Module-AC 映射、阻断问题。
- 不要自行声称独立 G0 PASS，不要自行授权 M00 开工。
完成后 STOP，等待我转交独立 GPT 审核。
```

### 26.2 GPT — G0 独立 BASELINE Review Prompt

```text
你是 Modular-Dev-Orchestrator 的独立质量审核员。审核阶段：G0 / BASELINE。
目标：独立审核提交的当前版本《MDO 工程企划书》和《MDO 开发流程与执行手册》，
以及 G0_BASELINE_REPORT、MODULE_ROADMAP、SOURCE_OF_TRUTH_POLICY、ADR 和真实 Git 状态。

重点检查：
1) 两份文档是否具有唯一的事实来源和相同的 M00–M15 路线图；M10 是否拥有 merge/Post-Merge/G4。
2) 是否满足 Apple Silicon arm64、SwiftUI、独立 DMG、个人首版/未来公开发行准备的已确定决策。
3) G0–G4、QC1/QC2、政策授权、GitHub、沙箱、Reviewer 隔离与证据是否可强制执行。
4) 所有高风险的外部 API/权限是否作为 M00 待验证条件，而非未经证实的已知事实。
5) 每模块 Goal/Scope/Out of Scope/Dependencies/Deliverables/Test/AC 是否具体可验收；有无依赖循环。
6) Source Manifest、OSS 许可、Review Bundle、回滚/恢复、签名与公证是否充分。
7) PDF 与 Markdown 是否同版、是否出现后续修改却沿用原审核 PASS 的情况。

必须输出：PASS / CHANGES REQUIRED / BLOCKED_UNVERIFIED；
按严重程度列 Finding ID、具体位置、理由、修复要求、受影响的验收条件。
如不通过，给一份可直接用于 Codex G0 FIX 的完整 Prompt。
PASS 只代表技术审核，不能替代 Owner 的 G0 正式批准。
```

### 26.3 Codex — Mx PLAN Prompt

```text
你是 MDO 的开发端 Planner。模式：PLAN。当前模块：Mx [模块名称]。
输入：当前项目位置、真实 Git Base/HEAD、当前 G0 基线双文件 SHA、SOP SHA、
本模块 Goal/Scope/Out of Scope/Dependencies、AC-Mxx 清单、相关 ADR、真实 Owner 授权。
只读取与本模块相关的规范与官方资料；核对官方 Guide/Example/Reference/限制/错误/安全，
必要时审阅官方 HTML 中有意义的流程/状态图。记录 Source Manifest 与不确定事实。

在修改实现前输出 Implementation Plan：
Goal、Scope、Out of Scope、Dependencies、源资料与版本、视觉核对、OSS/License、
预计文件/接口/数据所有权、架构和风险、测试分层、Failure Injection、UI 验证、
回滚/恢复、逐项 Acceptance Criteria、Open Questions、当前工作树与版本。

不允许写实现代码、不允许越出当前 Module、不允许自动 commit/push/merge。
完成 Plan 后 STOP，等待独立 Plan Review 和 Owner/Policy 对实施的授权。
```

### 26.4 GPT — Mx PLAN Review Prompt

```text
你是 MDO 当前 Mx 的独立 PLAN Reviewer。
根据已批准双文件和 SOP 固定 SHA、当前模块 Scope/AC、真实 Implementation Plan rNN
及相关官方来源/ADR进行严格审核。
核对：范围、依赖、架构、权限、数据所有权、来源与 API 真伪、Apple/macOS 安全、
官方图示行为、Fail-Closed、状态/并发/持久化、测试与 Failure Injection、UI 和回退。
只就提供且实际读到的内容下判断，不以开发端自述当证据。
输出 PASS / CHANGES REQUIRED / BLOCKED_UNVERIFIED，逐项 Finding ID 与 AC 影响。
不通过提供完整 FIX PLAN Prompt；通过说明还需 Owner/Policy 授权的下一动作。
未经授权不得建议自动 Implement、Push、Merge 或下一 Module。
```

### 26.5 Codex — Mx IMPLEMENT Prompt

```text
你是 MDO 当前 Mx 的开发工程师。模式：IMPLEMENT。
请先校验：Owner/Policy 实施授权仍有效；G1 PLAN PASS 与当前计划版本匹配；
G0 双文件/SOP SHA 未漂移；任务 Scope、AC、工作树和分支真实有效。
任何缺失都停止并报告，不得用模型记忆补批准。

严格只实现已批准的当前模块，不开发下一模块，不进行无关重构。
执行 Unit/Property/Contract/Integration、适用 Failure Injection 与真实 UI Verification；
记录 PASS/FAIL/NOT RUN/NA、命令、环境、真实退出码、证据哈希。
执行 QC1 全量 diff，自查安全、权限、并发、状态、未知副作用、Secret、OSS License、漏测、死代码。
只将有来源、已经实现和验证、以后可复用的经验写入 Skill；更新必要 Docs/ADR/PROJECT_STATUS。
生成固定版本 Review Bundle、MANIFEST、完整累计 patch、新文件、测试证据、文件 SHA、Completion Report。
对 Bundle 与当前源码快照做一致性核对。

不因为自测成功宣布 QC2 PASS；除非本轮明确授权，不自动 commit/push/PR/merge/下一 Module。
输出准确文件位置后 STOP，等待独立 QC2。
```

### 26.6 GPT — Mx QC2 IMPLEMENTATION Review Prompt

```text
你是 MDO 当前 Mx 的独立 QC2 Reviewer。审核阶段：IMPLEMENTATION。
请读取已批准工程基线、SOP SHA、Implementation Plan 的批准版本、
完整 Review Bundle（MANIFEST、累积 PATCH、相关文件、EVIDENCE、HASHES）及当前 Git 快照。

独立检查：Scope/AC、真实全量 diff、架构/接口、Apple/macOS 权限、Token 和 GitHub 访问边界、
状态机/并发/幂等/恢复、失败注入、UI、秘密、许可证、测试真实性与版本一致性。
区分“自己复跑”与“只阅读测试记录”，NOT RUN 不得视为 NA。

输出 PASS / CHANGES REQUIRED / BLOCKED_UNVERIFIED、逐项 AC 矩阵、Finding ID、严重级别、
文件/行号/证据、修复方向及完整可复制 FIX Prompt。
如 PASS 只能表示技术审核合格，不能替代 Owner 的 Push、Merge、发布或 G4 决策。
```

### 26.7 Codex — Mx FIX Prompt

```text
你是 MDO 当前 Mx 的开发工程师。模式：FIX。
本轮仅针对附件中的 QC2/PLAN Finding ID 与 Owner 已批准的修复范围。
先核对实际 Base/HEAD、当前 Plan Revision、Review Snapshot、待修复问题原文。
逐项修复并记录 Finding → 修改 → 验证的映射；重新运行受影响测试、失败注入与 QC1。
若修复会改变 Scope、架构、外部契约、权限或审批策略，立即暂停并提交 Plan 修订回 G1。
生成新版 Completion Report、Review Bundle 和 SHA，旧 PASS 不覆盖新内容。
不得自动进入下一 Module 或执行未授权 GitHub 写操作。完成后 STOP。
```

## 27. 每轮审核与工程输出的固定目录和命名

### 27.1 目录和留痕

建议由受控 Evidence Manager 保存到本机 `Application Support/<app-id>/artifacts/<project-id>/<module-id>/rNN/`；人工 Bootstrap 可以临时在用户明确授权的 `PROJECTS/Modular-Dev-Orchestrator/.mdo-review/` 创建，只要该路径受到 `.gitignore` 和 Secret 扫描保护。**不以桌面固定 zip 路径作为长期实现约束**。导出桌面只是用户选项，真正的 Artifact ID/Hash 保留在可信索引。

### 27.2 命名规则

`G0_BASELINE_REPORT-rNN.md`、`Mxx_IMPLEMENTATION_PLAN-rNN.md`、`Mxx_COMPLETION_REPORT-rNN.md`、`Mxx_REVIEW_BUNDLE-rNN.zip`、`Mxx_QC2_REVIEW-rNN.md`、`Mxx_ACCEPTANCE_RECORD.md`。每一轮必须显式注明时区（内部时间 UTC，UI 本地化展示）、Base、Head、Snapshot、Policy 与 G0 基线哈希；压缩包及报告不得沿用旧修订号。

### 27.3 PR 检查列表

`PR_TITLE` 含 Module ID；描述列 Goal、Scope、Out of Scope、AC 状态、测试、Review Bundle 哈希及回滚；提交分支与审核快照相符；CI 真实执行；没有 Secret；审批与 GitHub Branch Protection 均允许；如 HEAD 在待合并期间变化，必须重验/重审。

## 28. 完整验收测试执行矩阵

| 层级 | 启动阶段 | 必测目标 | 判定证据 |
|---|---|---|---|
| Unit / Property | M01 起逐模块 | Schema、状态和权限不变量 | 测试命令、退出码、覆盖范围 |
| Contract | M01 起逐模块 | SwiftUI ↔ Engine、Adapter ↔ Provider | Schema 版本与 Mock/真实样本 |
| Integration | M02 起逐模块 | SQLite、IPC、工作区、Reviewer、GitHub | 可重复的环境/日志 |
| Failure Injection | M03 起逐模块 | kill、超时、429、重复、权限失效、磁盘满 | 注入条件、预期、最终状态 |
| UI Verification | M11/M12 & 有 UI 改动时 | Native UI、禁用状态、VoiceOver | 真机截图/记录、可访问性结果 |
| Security | M04/M05 与变更敏感模块 | R3 越权、路径逃逸、密钥与策略 | OS 沙箱负测和审计记录 |
| E2E Lab | M13 | 正常+恶意样例自动闭环 | 任务、PR、CI、Finding、批准 ID |
| Distribution | M14/M15 | DMG、Gatekeeper、换机与迁移 | 签名、公证、安装/回滚日志 |

### 28.1 T01–T16 最低验收套件

T01 PLAN PASS 授权后继续；T02 PLAN 失败自动修订；T03 QC2 拒绝自动 FIX；T04 超轮数停止；T05 CI FAIL 禁 merge；T06 Owner 拒绝无受保护动作；T07 SHA 变化使审批失效；T08 Codex 中断保存状态；T09 App kill 后无重复外部副作用；T10 Agent 修改 Policy 被阻止；T11 Bundle 缺失阻断；T12 额度耗尽安全暂停；T13 iCloud/目录授权失效停止写；T14 PR 超时对账；T15 Prompt Injection 伪批准无效；T16 GitHub Ruleset 未满足不得合并。所有用例需要**真实状态和动作验证**，不是自然语言答复。

### 28.2 退化能力如何验收

如果 M00 证明自动 GPT Reviewer 暂不可用，v1.0 可以经 Owner 同意改为隔离 Codex Reviewer；如果该模式也不能建立可信独立审查，必须明确将当前产品限制为“半自动人工审核”并修改发布范围，而不是在测试报告中称“全自动 QC2”。任何违反独立审查和审批强制性的降级都不能自动允许上线。

## 29. 发行、更新与维护清单

### 29.1 Apple Silicon DMG Release Gate

- arm64 App、Helper、嵌入二进制均有可复核架构与签名。
- Developer ID、Hardened Runtime、Notary Accepted、staple、Gatekeeper 在另一个干净 Mac 上验证。
- 发布工件 SHA-256、对应源 commit、SOP SHA、双文档修订和迁移版本一致。
- 新设备可独立选择 PROJECTS、授权 Codex/Reviewer/GitHub；不复制旧电脑 Keychain Token/Bookmark。
- 升级迁移先备份；失败能回滚，旧版状态不被悄悄删掉；卸载不主动删除用户项目目录。
- Release Notes 包含已知限制、额度策略、日志隐私、支持范围和未来公开发布待办。

### 29.2 未来公开发行（与首版严格分离）

个人使用版通过 M15 后，不自动获得公开发行资格。公开发布必须新增审查：OAuth / 模型计划授权能否面向其他用户分发、Apple Developer ID/签名密钥与更新通道、开源许可证、隐私政策和数据删除、故障/漏洞披露、用户支持、自动更新完整性和新用户安装演示。当前开发架构要为此保留边界，但不强行增加 v1.0 的 SaaS 或收费系统。

## 30. Requirement → Module → Acceptance 的最小追踪表

> 全量测试证据在实际开发中由 `REQUIREMENT_TRACEABILITY.csv` 自动生成；下表为 G0 起始映射，缺少关键 P0 要求时不得冻结 G0。

| 要求 / Requirement ID | Module | 对应验收依据 |
|---|---|---|
| REQ-PLAT-001 / 002 / 003 | M00, M14 | AC-M00-04、AC-M14-01..04 |
| REQ-PROJ-001 / 002 | G0, M11 | G0-AC01..05、AC-M11-01..04 |
| REQ-WS-001 / 002 / 003 / 004 | M05, M11 | AC-M05-01..06、AC-M11-02..03 |
| REQ-WF-001 / 002 | M02, M03 | AC-M02-01..04、AC-M03-01..04 |
| REQ-WF-003 / 004 | M09 | AC-M09-01..05 |
| REQ-AG-001 / 004 | M00, M06 | AC-M00-01、AC-M06-01..05 |
| REQ-AG-002 / 003 | M07, M08 | AC-M07-01..05、AC-M08-04 |
| REQ-EV-001 / 002 / 003 | M08 | AC-M08-01..05 |
| REQ-SEC-001 / 002 / 003 / 005 | M04, M10 | AC-M04-01..05、AC-M10-02..06 |
| REQ-SEC-004 | M05 | AC-M05-05 |
| REQ-GH-001 / 002 | M10 | AC-M10-01..06 |
| REQ-UX-001 | M11 | AC-M11-01..06 |
| REQ-UX-002 / 003 | M12 | AC-M12-01..06 |
| REQ-OBS-001 / 002 | M02, M12 | AC-M02-02..04、AC-M12-05..06 |
| REQ-COST-001 | M03, M06 | AC-M03-05、AC-M06-03 |
| REQ-REL-001 / 002 | M14, M15 | AC-M14-01..06、AC-M15-01..06 |
| REQ-DOC-001 | G0, M01, M15 | G0-AC01..05、AC-M01-03、AC-M15-02 |
| REQ-TEST-001 | M13 | AC-M13-01..06 |

## 31. 当前启动状态与下一步（非实时进度记录）

**G0 候选准备就绪**：两份文档已统一为 M00–M15，已确定 Apple Silicon arm64 / SwiftUI / 自用首版且未来公开发布预留 / 独立 DMG / 本地 Codex / 独立 Reviewer / 强制 Human Gate。**尚未完成**：独立 G0 审核、Owner 基线批准、M00 外部能力真机验证、任何正式 Module 实现。此段是发布时的初始说明，未来实时进度只能写入 `PROJECT_STATUS.md`，不回写本手册。

### 31.1 Owner 的最短启动动作

1. 将双文档的 Markdown 原稿放入 MDO 源码工作区的 `docs/`，不改变文件正文并记录哈希。
2. 在 Codex 项目执行第 26.1 节 G0 Prompt；输出基线清单与候选规范文件后 STOP。
3. 在独立 ChatGPT 项目执行第 26.2 节审核；根据 Finding 反复修订直到技术 PASS。
4. 人工确认 G0 的产品、范围与条件性技术风险后，冻结两份正式文件 SHA 和 SOP commit SHA。
5. 启动 M00 PLAN → 独立审核 → Owner 授权 → M00 PoC → QC1/Bundle → QC2 → PR/CI → G4。

### 31.2 最终原则

企划书定义目标和架构；执行手册定义顺序和操作；SOP 定义共同质量纪律；官方资料定义适用的外部契约；真实代码与可信测试定义事实；Reviewer 给出独立技术意见；Policy Engine 和 Owner 决定哪些行为真正允许发生。任何主体均不能凭一条自然语言“PASS”替代其他主体的职责。

---

**文件结束 · v1.0-G0-Candidate · 未经 G0 审核与 Owner 批准不得宣称 APPROVED。**
