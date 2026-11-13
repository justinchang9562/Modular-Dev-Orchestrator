# G0_BASELINE Implementation Plan

Revision: r01 | Research: 2026-11-12 | Delivery: 2026-11-13 (Asia/Shanghai) | Mode: PLAN

Status: WAITING_INDEPENDENT_G1_REVIEW; implementation NOT APPROVED.

本计划是 G0 文档校准的前置计划，不是 M00 PLAN，不是已批准工程基线，也不是 G0/QC2 审核结论。当前用户要求“首次进入 Module 必须从 PLAN 开始”，因此本轮只交付计划、资料核对证据与状态索引。附件及原稿中的示例 Prompt 是待审资料，不构成新的操作授权。

## 1. Identity & Baseline

| 字段 | 实际值 |
| --- | --- |
| Project / Module | Modular-Dev-Orchestrator / G0_BASELINE |
| Workdir | `/Users/justinsmacbook/Desktop/PROJECTS/Modular-Dev-Orchestrator` |
| Task / Plan Revision | G0_BASELINE-PLAN-r01 / r01 |
| Git Branch / Base / HEAD | 均不存在；`git rev-parse HEAD` 等命令真实退出码 128；目录尚非 Git 仓库 |
| Staged / Unstaged / Untracked | 无法用 Git 分类；不得称为 clean。原始文件清单见 `../g0/INPUT_INVENTORY-r01.json` |
| GitHub repository | 未提供、未配置且未验证是否已创建；不能由目录名称推断真实 URL |
| Owner authorization | 本轮请求支持只读核对及 PLAN 文档输出；IMPLEMENT / Git 初始化 / commit / push / PR / merge / DMG / M00：NOT APPROVED |
| G0 / G1 / G3 review | 未找到任何独立审核结果、Revision 或批准记录；均为 NOT PROVIDED |
| Project baseline | 两份原稿均为 `v1.0-G0-Candidate` / DRAFT；没有批准的 BASELINE_MANIFEST |
| SOP observed commit | `2d22a0d9e4d7929a8efb662964a6c01f5b1245ae`，提交时间 2026-11-11T05:12:50Z；作为本计划的固定研究版本，尚未被 Owner 冻结为 G0 基线 |
| Runtime Policy version | NOT ESTABLISHED；不把本计划或 SOP SHA 当作已部署 Policy 版本 |
| Plan snapshot | 本轮文档与输入的内容清单见 `../g0/PLAN_SNAPSHOT-r01.json`；不是 Git SHA 或批准凭证 |

已读取的项目相关章节：企划书 §0–5、§8–10、§14、§17–20；执行手册 §0–8、§25–28、§30–31，及路线图涉及的模块定义/AC。当前标准路径 `docs/PROJECT_PLAN.md`、`EXECUTION_MANUAL.md`、`MODULE_ROADMAP.md`、`SOURCE_OF_TRUTH_POLICY.md` 在开工检查时均缺失；本轮新建状态索引不代表其他标准文档已经就位。没有本地 AGENTS.md、项目 Skill、ADR、Schema、接口实现或测试；已核对父目录 AGENTS.md 并读取用户提供的 AGENTS 指令和个人资料库入口及相关协作流程。没有可证实的已验收依赖模块。

输入 SHA-256（原稿只读保留）：

| 输入 | SHA-256 |
| --- | --- |
| `docs/MDO_工程企划书_v1.0.md` | `e2b2c31a97d0936829e7a2109b5181204c80350aae112235c80081d9be0e6bbf` |
| `docs/MDO_开发流程与执行手册_v1.0.md` | `023628c0960734c4e2df0202f42b67d33119a43e0cb86a3da8ac5a757a66e6fe` |
| `/Users/justinsmacbook/Desktop/MDO_工程企划书_v1.0.pdf` | `8d476f4d78fa547edc9b674623b3fb20cd2df31dab18c17b18dd3b2e9a126b6a` |
| `/Users/justinsmacbook/Desktop/MDO_开发流程与执行手册_v1.0.pdf` | `fa2a129e23f19f2709f78fd242ab23b72f5d80961286e30326b244cdc1e53081` |
| `docs/design/references/a_clean_high_resolution_apple_style_macos_ui_conc_1.png` | `6135e92ce37ecda05f1cd63bc96584259aab844420bc20c8d249104df40af79c` |

## 2. Goal / Scope / Out of Scope

Goal：提出一套可审核的 G0 文档校准步骤，让后续 Reviewer/Owner 能批准确切文本、SOP 版本、需求映射和条件性风险。

本轮 PLAN Scope：读取并记录实际输入、Git/环境现状、缺件、规范冲突；核对相关 PDF 页面及设计图；固定 SOP 研究版本；提出文档变更、验证和恢复方案；保存本计划、来源清单、证据和真实状态。所有新文件均为计划或核对记录。

待同一 r01 计划通过独立 G1 且 Owner 明确授权后，G0 文档校准范围限于：建立候选标准文档入口；受控提取路线图及来源政策；逐项需求映射；登记 PROPOSED ADR；解决本文列明的文本冲突；生成 G0 候选报告及未批准候选 Manifest。不得借文档校准之名实现产品能力。

Out of Scope：M00–M15 的 PoC/实现、SwiftUI/Xcode 工程、Node/TypeScript 工程、运行时 Policy/审批系统、Schema 实现、数据库、沙箱、登录/Token 检查、付费调用、签名/公证/DMG、GitHub 写操作。未经单独批准也不执行 Git 初始化和初始提交。本计划不拆分子模块、不重新编号 M00–M15、不更改控制面安全策略。

## 3. Dependencies

| 依赖 | 现状 | 允许推进到哪里 |
| --- | --- | --- |
| 双文档原稿与阅读版 PDF | 可读，版本标签一致，原始 SHA 已记录 | 可完成本轮研究；全文等价校准仍需完成 |
| UI 视觉基准 | 可读、已实际查看、SHA 已记录 | 仅记录，不实施 UI |
| 固定 SOP | 具体提交及相应文件可读 | 作为 PLAN 依据；正式 G0 采用需 Owner 批准 |
| 标准文档、ADR、全量追踪表 | 缺失 | 属于待批准的 G0 文档产物 |
| 独立 G1 + 同版实施授权 | 缺失 | 阻断本计划后续文档实施 |
| Git 基线 / 独立分支 | 缺失 | 阻断常规 IMPLEMENT；须先决定并授权 Git 自举方式 |
| G0 独立 BASELINE Review + Owner 冻结 | 缺失 | 阻断 M00 PLAN 及任何正式模块实施 |

G0 没有已完成前置代码模块；不能把路线图中的依赖写成已验收。原 46 页《MDO 产品与开发手册》未提供：只知道原稿将其列为设计参考，未读取、不据此推断需求。独立 Reviewer 应决定是否必须补充这一历史参考，才可冻结基线。

## 4. Requirement Traceability

G0 文档交付验证需求是否被正确约束和分配，不宣称产品实现完成。原企划书 §3 共 37 个唯一 REQ；手册 §30 是起始映射。后续建立每个 REQ 一行的 CSV，含来源章节、Module、AC、优先级及其批准状态、测试方法、未来证据 ID、当前证据状态。

| REQ / 来源 | 当前 Module 责任 | 原始 AC | 拟定检查 / Evidence |
| --- | --- | --- | --- |
| REQ-PROJ-001/002；企划书 §0、§4 | G0_BASELINE 建立候选文件与内容绑定 | G0-AC01、G0-AC05 | DOC-01 文件/版本/哈希；Owner 与 Reviewer 原始记录，当前缺失 |
| REQ-DOC-001；企划书 §18–20、手册 §2/3/8 | 单一路线图、来源和交接 | G0-AC01..04 | DOC-02 模块集合与依赖 DAG；DOC-03 全量追踪 |
| REQ-PLAT-001/002/003、REQ-AG-001/002/004、REQ-WS-001/002/003、REQ-REL-001/002 | 记录条件性技术假设并分配 M00 与后续模块 | G0-AC03 | DOC-04 ADR-001..008 的 PROPOSED / UNVERIFIED 与关闭条件 |
| REQ-SEC-001..005、REQ-EV-001..003、REQ-COST-001 | 保留权限、证据和禁止自动付费约束；不实现它们 | G0-AC03/04 | DOC-03/05 追踪及负向文本核对；对应产品测试 NOT RUN |
| 其余 REQ（包括 WF/GH/UX/OBS/TEST） | 不遗漏任何 REQ 的责任 Module 和可检查 AC | G0-AC02/04 | DOC-03 37/37 ID 集合覆盖与人工语义核对 |
| 用户 UI Design Baseline 指令；企划书 §14 | 把设计图及 SHA 纳入候选基线引用，记录未来 UI 验证责任 | G0-AC01/04 的补充约束 | DOC-06 引用存在、图片 SHA 一致；不作 App UI PASS |

不得凭 REQ/AC 编号集合一致就认为语义覆盖成立。当前 P0 未逐 REQ 明确标注；拟先对全部 37 REQ 建映射，以免通过自选优先级漏项；P0 分类与对应安全负测必须提交 Reviewer/Owner 确认。产品 TestEvidence 保持 PLANNED / NOT RUN，不填虚构 run ID。

## 5. Official Source Manifest

完整条目、URL、访问时间和适用环境见 `../g0/SOURCE_MANIFEST-r01.json`。

| 来源 | 版本 / 实际核对 | 结论与适用边界 |
| --- | --- | --- |
| [GitHub permanent links Guide](https://docs.github.com/en/repositories/working-with-files/using-files/getting-permanent-links-to-files) | 2026-11-12 访问；页面未标注发布/修订号 | 分支 URL 会变化，使用具体 commit URL 固定 SOP 文件 |
| [GitHub repository contents Reference / GET example](https://docs.github.com/en/rest/repos/contents#get-repository-content) | 核对 GET 参数 ref、返回/限制、权限与示例；示例 API header 为 2026-03-10 | 可按 commit 读取；公开资源支持无鉴权 GET；不代表 MDO 已取得任何写权限。Connector 实际协商 API 版本未暴露，记 UNVERIFIED |
| [Modular-Dev-Quality-SOP](https://github.com/justinchang9562/Modular-Dev-Quality-SOP/tree/2d22a0d9e4d7929a8efb662964a6c01f5b1245ae) | 固定提交的 README、三份 docs、Developer/Reviewer 模板与 LICENSE 的相关条款 | G0/G1/QC2 与 Owner 放行分离；原始证据、累计改动、阶段停止和来源治理；不覆盖本次更严格的桌面交付要求 |

Guide 后核对 Reference 的 GET 示例、ref 和 200/302/304/403/404 状态。本轮不实现 GitHub API 客户端、不使用或查看凭证、不测试 API 错误恢复。上述两份 GitHub 相关页面没有需用于本轮行为判断的架构/时序图；SOP README 的 Mermaid 源图已读，确认 Plan Gate → Owner 放行 → 实现/验证 → QC1 → Bundle → QC2 → 人工收尾，未将图中任何节点当作已完成事件。

Apple/OpenAI 运行时官方资料：本轮不验证其 API 或授权能力，NOT APPLICABLE TO G0 DOCUMENT PLAN；相关事实仍 UNVERIFIED，必须在 M00 的独立 PLAN/授权范围按当前版本核验。既有原稿里的入口链接只用于发现，不能填成已访问的 Source。最低 macOS、Xcode、Node、Codex 版本均未选定。

PDF 核对：工程企划书 21 页、手册 26 页均完成文本提取；实际查看企划书 p3/p8（文控/控制面图）、手册 p5/p10（G0/路线图）渲染；另读相关 p12/p19–20、手册 p11/p20–21/p25–26 文本。两对文档去空白后 REQ/AC/G0-AC 集合一致。限制：这不证明完整正文和布局完全等价；候选正文有修改时，旧 PDF 保留为原始参考，需重新导出并校准后才能标为相应阅读版。

## 6. Architecture & Interfaces

本 Module 只创建治理文档，不接入或调用产品运行时。保留以下约束：

```text
SwiftUI UI
    -> Trusted Engine / Policy / Approval / Workspace Broker
    -> Limited Runner / Agent Adapter / Read-only Reviewer
    -> approved External Services
```

| 边界 | G0 的文件表达 | Ownership / 约束 |
| --- | --- | --- |
| SwiftUI | 设计图引用和未来 AC | UI 展示/交互；不得成为最终授权者 |
| Engine / State | 企划书与手册一致性 | Engine 未来拥有任务状态；当前没有真实 Engine 状态 |
| Policy / Approval | 保留风险、绑定、过期、撤销、不自批条款 | 可信控制面/Owner；本计划、报告、截图均不是授权事件 |
| Workspace | 保留目录授权与 OS 隔离职责 | Workspace Broker；worktree 不视作沙箱 |
| Agent / Reviewer | 隔离会话、只读审核对象、禁止管理员秘密 | Adapter/受限执行面；不共享 Developer 推断作为审核事实 |
| IPC / Schema | 仅登记 M00 验证和 M01 契约责任 | 传输/身份/取消/错误码 UNVERIFIED；本轮无新可执行 IPC/Schema |
| Baseline data | 原稿、候选 canonical 文件、来源/哈希清单 | 原稿只读；候选内容由 Developer 提出，Reviewer 独立审核，Owner 冻结 |
| Progress state | `docs/PROJECT_STATUS.md` | 状态索引；不写入产品 DB、不代替批准记录 |

Allowed Dependencies：治理文档可引用原始企划书、手册、固定 SOP、官方来源及设计图。Forbidden Dependencies：UI/Agent/Reviewer 直接写 Policy/Approval，读取 Keychain/管理员 Token，引用整个 HOME/其他项目，依赖候选 MDO 自己授权自身发布。

候选 Manifest 使用明确 status、source SHA、canonical SHA、SOP SHA、路线图修订、风险清单，批准字段保持 null/NOT APPROVED。真正批准必须由 Owner 对精确候选快照产生记录；Developer 不能把批准字段补成成功。清单不自包含自身哈希；ZIP 自身哈希在外部记录，避免自指。

## 7. Files to Create / Modify / Delete

本轮实际新增（全部为 PLAN / 证据 / 状态，尚非实施成果）：

| 路径 | 原因 |
| --- | --- |
| `docs/plans/G0_BASELINE_IMPLEMENTATION_PLAN-r01.md` | 本计划 |
| `docs/g0/INPUT_INVENTORY-r01.json` | 开工文件、Git 检查、缺件与原始输入 SHA |
| `docs/g0/SOURCE_MANIFEST-r01.json` | 实读来源、版本、结论与限制 |
| `docs/g0/PLAN_EVIDENCE-r01.json` | 实际命令/检查、退出码、发现与未执行项 |
| `docs/g0/PLAN_SNAPSHOT-r01.json` | 本轮文档/输入的内容绑定，排除自身 |
| `docs/PROJECT_STATUS.md` | 当前 G0/PLAN 待审的真实索引 |

后续待批准的 G0 文档变更白名单：

| 路径 | 拟变更 / 保护规则 |
| --- | --- |
| `docs/PROJECT_PLAN.md` | 从中文原稿建立候选 canonical 正文；注明来源、修订与视觉图引用；受基线变更保护 |
| `docs/EXECUTION_MANUAL.md` | 从原稿建立候选 canonical 正文；限于已列明的 G0 自举/桌面交付补充；受基线变更保护 |
| `docs/MODULE_ROADMAP.md` | 从手册 §8 提取 16 个 Module；从 §9–24 引用各自交付/AC；不另造序列 |
| `docs/SOURCE_OF_TRUTH_POLICY.md` | 受控摘录手册 §3，注明来源 SHA；不放宽安全条款；受基线变更保护 |
| `docs/REQUIREMENT_TRACEABILITY.csv` | 37 个 REQ 的逐项映射、拟定优先级/测试责任；不是产品测试结果 |
| `docs/adr/ADR-001.md` .. `ADR-008.md` | 沿用企划书 §19 的编号与问题，全部 PROPOSED，关闭责任 M00 或后续 Release Gate |
| `docs/design/BASELINE.md` | 只记录用户指定 PNG、SHA、原生样式和未来实机对照要求 |
| `docs/G0_BASELINE_REPORT-r01.md` | 完整差异、冲突处置、覆盖、待决项与 G0 提交说明 |
| `docs/BASELINE_MANIFEST.candidate-r01.json` | 未批准候选哈希清单，不冒充正式 APPROVED Manifest |
| `docs/G0_OWNER_DECISIONS-r01.md` | 空缺决策的实际状态/待确认提案，不代填批准 |
| `AGENTS.md` | 如同版计划获批，新增简短持久入口和禁止越权规则；不复制长 Prompt |
| `docs/PROJECT_STATUS.md` / `docs/g0/` | 更新实际阶段及文档验证记录；保留 r01 研究记录，变化产生新修订 |

不删除、改名或覆盖两份中文原稿、桌面 PDF、指定 PNG；canonical 建立后原稿明确标为历史输入，不能与 canonical 同时独立维护。现有 `.DS_Store` 保留，不纳入审核载荷。运行时 `policies/**`、`security/**`、`infra/**`、`.github/workflows/**`、签名/审批/沙箱实现均为受保护的 Out of Scope。Git 目录/远端/初始提交不在上述文件白名单内，需单独明确授权。

## 8. Security & Reliability Risks / Conflict Register

以下是 Developer 的规划发现编号，不是独立 Reviewer Finding，也不构成 FIX 授权。

| ID / 风险 | 实际依据 / 影响 | 处置提案 / 关闭条件 |
| --- | --- | --- |
| PG0-01 基线缺件 | 标准文档、ADR、Manifest、审核记录缺失 | 本计划获批后建立 DRAFT 候选；独立 BASELINE Review + Owner 冻结前不进 M00 |
| PG0-02 Git 缺失 | 不能给出真实 Base/HEAD/分支、累计 Git patch | Owner 先确认本地 Git 自举与初始提交范围；无有效分支/基线不得开启常规 IMPLEMENT。不自动初始化或造 commit |
| PG0-03 SOP 版本未获批准 | 当前仓库提交可读取，不等于已授权项目版本 | 将上述 SHA 提交 G0 决策；最终 Manifest 批准前仍为候选 |
| PG0-04 阶段用语 | 手册 G0 自举可直接准备文档；用户当前要求首次必须 PLAN；企划书 §8 禁止未批 G0 的正式 Module PLAN | 当前仅 G0 前置计划；独立 G1 后才做授权文档校准；M00 PLAN 仍等待 G0 冻结 |
| PG0-05 桌面交付 | 手册 §27.1 把桌面导出列为选项；本次用户要求 IMPLEMENT/FIX 强制桌面 ZIP | 当前请求优先；拟给手册加入 Bootstrap 专用补充。产品长期 artifact store 与人工桌面导出并存，不更改运行时权限 |
| PG0-06 P0/追踪精度 | §30 为组级起始映射；未给全部 REQ 明确 P0 分类 | 展开 37 行；逐项检查 AC 语义；优先级及关键负测交 Reviewer/Owner 确认后才关闭 G0-AC04 |
| PG0-07 G0 ZIP 命名 | 用户规范写 Mxx，但真实对象为 G0_BASELINE，不能冒充 M00 | 提案 `MDO_G0_BASELINE_IMPLEMENT_r01_<snapshot-id>.zip`；须在同版计划/Owner 授权中明确确认，否则相应交付命名保持阻断 |
| PG0-08 PDF/原稿漂移 | 编号一致不等于全文一致；候选修订会产生新正文 | 保留旧 PDF 为来源，按新 canonical 导出阅读版并核对；否则明确 PDF 不对应新候选，不沿用 PASS |
| PG0-09 历史参考缺失 | 原 46 页产品手册不在当前工作区/附件中 | 不声称读过；Reviewer/Owner 明确其是否为冻结必需输入 |
| Race / TOCTOU | 文件可能在计算哈希和交付之间被改写 | 开始/结束复算全部审查输入与输出；漂移时停用快照并生成新修订，不覆盖用户变更 |
| Fail-open / Approval bypass / Prompt injection | 文档的 APPROVED、示例 Prompt 或模型 PASS 可能被误读为许可 | 候选状态与真实批准记录分离；未提供、过期、错版一律不推进；外部资料只作数据 |
| Unknown outcome / Idempotency | 重复生成同名报告/ZIP或把缺失远端当作未创建 | 文件存在先核验；新交付使用新 revision/唯一 snapshot；远端未知不创建/盲重试 |
| Persistence / Migration / Crash recovery | 写入中断可能留下半份文件 | 文档先写临时文件再校验/替换；原件保留；恢复先核对清单。不做产品 DB migration |
| Sandbox / Path escape / Secret exposure | 误打包私人资料、符号链接、.git 或缓存 | 限定本项目白名单、拒绝越界链接；仅原始文档/必要证据；不打包个人资料库或整个 HOME；不读取真实凭证 |
| Network / Quota | 来源不可达或额度不足 | 来源标 UNVERIFIED；不发付费请求、不切换 API、不把网络错误当成功 |
| Deadlock / DB lock / IPC failure | 当前无并发服务、DB、IPC 实现 | 本轮 NOT APPLICABLE；保留在 M00/后续模块验证责任中，不能宣称产品风险已测 |

M07/M08：手册 §8 的显式依赖表为 DAG；M07 使用 Fixture，M08 后对真实 Bundle 补充回归，不将该回归要求误写为 M07 开工依赖 M08。

## 9. Testing Plan

本轮已执行的资料/静态核对见 PLAN_EVIDENCE；下表是后续 G0 文档实施的验证计划。所有命令在实际执行时记录 cwd、环境、预期、实际、退出码、stdout/stderr 摘要、文件哈希和证据 ID，保留失败结果。Bootstrap 记录必须标注由 Developer 工具执行，不伪装成尚不存在的独立 Runner 证据。

| ID / 类别 | 方法 / 预期可观察结果 | 当前状态 |
| --- | --- | --- |
| DOC-01 文件/哈希 | 校验 canonical 同版标签、原稿 SHA 未变、Manifest 指向确切正文/图/SOP；任何不符阻断交付 | 原始输入已核对；canonical NOT RUN |
| DOC-02 文档契约 | 解析手册 §8，比较 M00–M15 集合、依赖、职责/Owner Gate；DFS/拓扑排序无环，M10 保留 merge/Post-Merge/G4 | 原稿表无环已核对；canonical NOT RUN |
| DOC-03 需求追踪 | 提取企划书 §3 的 37 个 REQ，与 CSV 集合双向一致；每行 AC 存在且测试方法可检查；人工核对语义/P0 | REQ 数量已核对；CSV/语义/P0 NOT RUN |
| DOC-04 未验证事实 | 检查 ADR-001..008 PROPOSED、关闭证据/责任，未批准假设不写 VERIFIED；运行版本/认证/权限不猜测 | 原稿候选条款已读；新 ADR NOT RUN |
| DOC-05 安全负向文档核对 | 检查未提供 G0/G1/Owner 记录、示例 APPROVED 文本、Reviewer PASS 均不允许推进；不得存在伪造批准字段 | 计划与状态记录核对；不是运行时 Security Test |
| DOC-06 视觉基准 | 图片真实存在、SHA 不变、文档引用明确；未来 UI Module 必须真机截图对照原图 | 原始图已查看；基线引用新文档 NOT RUN |
| DOC-07 PDF/Markdown | 如导出新 PDF，用 pdftotext/pypdf 比较编号及正文，pdftoppm 渲染关键/跨页表与架构图；人工检查可读性与状态声明 | 原 PDF 编号/关键页面已核对；全文等价/新导出 NOT RUN |
| DOC-08 交付完整性 | 批准后按完整累计文件清单生成 Bundle；真实桌面定位、ZIP 非零、unzip -t、逐文件 SHA、前后快照、排除项和 Secret 扫描 | NOT RUN；PLAN 不产代码审核 ZIP |
| Unit / Property | 没有可执行产品逻辑；仅一次性文档集合与 DAG 核对，无新增产品单元测试 | NOT APPLICABLE |
| Contract / Integration | 文档来源/派生关系适用（DOC-01..04）；Swift/TS/API 运行时契约与集成无变更 | 产品测试 NOT APPLICABLE |
| Failure Injection | 无运行时操作，Timeout/DB lock/kill/GitHub HEAD drift 等产品故障不在本轮测试；DOC-08 的快照漂移/损坏检查只针对后续真实交付 | 产品故障注入 NOT APPLICABLE；交付验证 NOT RUN |
| UI Verification | 本轮无 App/UI 代码，不启动或截图宣称 UI 功能完成；查看概念图不是 UI 验证 | NOT APPLICABLE |
| E2E / Release | 没有可执行产品、发行包或获批测试环境 | NOT APPLICABLE |

批准后的 IMPLEMENT/FIX 即使有失败也交付明确 BLOCKED 的诊断包；必要证据未满足，G2/QC1 不写 PASS，不写 Ready for QC2 YES。纯 PLAN 阶段不执行实现 QC1，也不自行给独立审核结论。

## 10. Rollback / Recovery

当前没有 Git 回退点或远端副作用。以初始输入清单及 SHA 识别用户原有文件；只在明确归属本轮且哈希匹配时处理新增计划文件，不执行 git reset/clean，不恢复或删除用户变化。原件哈希漂移时先记录差异、停止该候选快照，重新核验，不用保存的副本覆盖新原件。

后续文档校准先保留原始输入，所有候选/报告逐版留存；失败时标记 DRAFT/BLOCKED、停止推进并保留诊断证据。不存在已授权外部写入，不需要声称撤销 GitHub 操作。若未来外部请求出现 UNKNOWN，先查证真实副作用，不能盲重试。ZIP 生成后相关文件发生变化，递增 revision 后重新交付，不改旧 ZIP。

## 11. OSS / Skill

SOP：Justin（justinchang9562），Modular-Dev-Quality-SOP，固定提交 `2d22a0d9e4d7929a8efb662964a6c01f5b1245ae`，CC BY 4.0；[原始来源](https://github.com/justinchang9562/Modular-Dev-Quality-SOP/tree/2d22a0d9e4d7929a8efb662964a6c01f5b1245ae)，[许可](https://github.com/justinchang9562/Modular-Dev-Quality-SOP/blob/2d22a0d9e4d7929a8efb662964a6c01f5b1245ae/LICENSE.md)，[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)。本计划为方法的项目化 ADAPT，新增 G0 前置 PLAN、输入证据和本次用户桌面交付约束，保留署名并说明修改，不声称原作者背书。不复制第三方实现代码，不引入产品依赖。

Copyright © 2026 Justin（justinchang9562）。原始 SOP 的无担保声明及其他许可条款以所链接 LICENSE 为准；本计划不提供额外质量保证。

使用 `pdf` Skill 做附件文本/关键页面核对，使用 `project-delivery-workflow` Skill 组织范围/验证/交接；用户本轮 Gate 与授权约束优先。Skill Changes：Added / Modified / Removed 均无；Pending 的技术假设只留在计划/未来 ADR，不写入已验证经验。

## 12. Acceptance Criteria

原始 G0 Exit Criteria 保留编号；下表是可检查完成条件，不是通过声明：

| AC | 文档校准完成条件 | 当前结果 |
| --- | --- | --- |
| G0-AC01 | 两份 canonical 同一候选修订；来源/正文/PDF 状态/SOP/设计图 SHA 清楚；最终提交对象稳定 | NOT MET：canonical 与候选 Manifest 尚未生成 |
| G0-AC02 | 唯一 16 模块表与原文一致；职责、依赖、交付、Owner Gate、AC 可查且无循环 | PARTIAL EVIDENCE：原表无环；派生文件未生成/未审 |
| G0-AC03 | ADR-001..008 登记外部未验证事实、M00/后续关闭证据；未冒充平台能力已验证 | NOT MET：计划已列边界，ADR 尚未建立 |
| G0-AC04 | 全量 37 REQ 有 Module→AC→测试/证据责任；P0 分类经确认，无关键遗漏 | NOT MET：待逐项映射及优先级确认 |
| G0-AC05 | 独立 BASELINE Review + Owner 对确切双文档/SOP/风险/路线图 SHA 的批准有原始记录 | NOT MET：审核与授权均未提供；Developer 无权关闭 |

本轮 PLAN 可交付条件：13 项计划内容齐备；输入、来源和环境有实际核对记录；原件未改；未缺省批准；本轮文档内容快照可复算。满足这些条件只说明计划已经准备好供审，不说明 G0 Exit Criteria 已达成或 G1 已通过。

## 13. Open Questions / Blockers / Stop

1. 独立 GPT G1：请审查本 r01 计划及配套证据；当前尚无结论。实施须另有 Owner 对相同计划/快照及动作范围的明确授权。
2. Git 自举：实际没有仓库。需明确是否授权本地初始化、初始分支、初始 commit，以及何种内容快照/commit 作为 G0 累计差异基线。不得自动造提交解决缺失。
3. SOP / G0 版本：请将本计划固定研究 SHA 作为候选审核，最终 G0 批准绑定正文与 SOP，不把“已下载”当“已批准”。
4. 请在文档校准计划审核中裁定 PG0-05（Bootstrap 强制桌面导出）、PG0-06（全量映射/P0）、PG0-07（G0 真实 Module 名称的 ZIP）、PG0-09（历史参考的必要性）。未裁定的受影响动作保持阻断。
5. Reviewer / OAuth、最低 OS/工具版本、Helper/文件授权、签名与公证仍 UNVERIFIED，由获批后的 M00 单独验证；本计划不替 M00 选择方案。

下一候选阶段：独立 G1 Plan Review → 对本计划的 G0 文档校准授权 → G0 候选文档准备 → 独立 BASELINE Review → Owner 冻结 G0。只有之后才可能由 Owner 启动 M00 PLAN。

本轮未生成 Review Bundle ZIP：阶段为 PLAN，用户明确免于代码审核 ZIP；没有发生 IMPLEMENT/FIX 交付，不适用其固定交付模板。没有 commit/push/PR/merge/DMG/下一 Module 行为。

STOP。等待独立审核及适用 Owner 授权。
