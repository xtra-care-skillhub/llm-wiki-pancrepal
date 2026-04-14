# PancrePal-Wiki 统一设计文档 v1.1（唯一主文档）

> 状态：Canonical / Single Source of Design Truth  
> 日期：2026-04-12  
> 范围：胰腺癌 MVP（3个月），可扩展到肿瘤/罕见病/慢病

---

## 1. 目标与边界

### 1.1 3个月目标（A+B）
- A：患者可用入口（Web Chat + 飞书/微信机器人）
- B：知识生产/审核工作台（社区协作）

### 1.2 长期目标（6-36个月）
- 知识资产可持续进化
- 与模型/Agent/工具解耦（技术更迭不重建）
- 支持 OpenClaw / Hermes / MCP 等未来接入

### 1.3 存储策略
- Feishu = 主库（Source of Truth）
- GetNote = 镜像
- 本地标准库（Markdown + JSON Cards + Graph）= 可迁移内核

---

## 1.5 与 Karpathy《LLM Wiki》原始模式的对齐

本项目明确遵循 Karpathy 提出的 `LLM Wiki` 核心模式，而不是把系统设计成一个“医疗版 RAG”。其关键约束如下：

1. **Raw sources 作为不可变真相层**  
   指南、论文、飞书原始讨论、社区素材、图片、表格等都进入 `sources/` 或等价原始层；LLM 只能读取，不能修改。

2. **Wiki 是持久化、累积式中间层**  
   用户日常查询默认优先面向 wiki/cards，而不是每次从原始资料重新拼接答案。新知识进入后，应先被“编译”进 wiki，再服务未来查询。

3. **Schema 决定行为纪律**  
   schema 不只是字段定义，而是 LLM 的操作手册：规定如何 ingest、如何 query、如何 lint、如何写回、如何记录 log。

4. **知识积累优先于即时回答**  
   高价值问答、对比分析、冲突说明，不应只停留在聊天记录里，而应回写为新页面、专题页或卡片。

5. **持续维护比一次性生成更重要**  
   系统必须把 cross-reference、冲突标记、页面更新、索引维护、日志维护当作一等能力，而不是附属功能。

因此，本项目的设计原则调整为：**先做 wiki 编译与维护闭环，再做 agent 接入与多渠道分发。**

## 2. 总体架构（按 Karpathy 模式重排）

1) **Raw Sources Layer**：指南、论文、飞书原始内容、Get 镜像、社区原始素材（只读、不可变）  
2) **Wiki Compiler Layer**：ingest / normalize / cross-link / contradiction detect / synthesis / log append  
3) **Persistent Wiki Layer**：
   - Human Wiki（Markdown 页面）
   - Machine Cards（JSON Schema 卡片）
   - `index.md`（内容索引）
   - `log.md`（时间日志）
4) **Schema / Policy Layer**：定义目录、frontmatter、ingest/query/lint/workflow 规范  
5) **Access Layer**：Skills、OpenClaw、Hermes、Web Chat、飞书/微信机器人

> 注意：这里将 **Schema/Policy** 提升为独立层，是因为按原文要求，它不是“配置细节”，而是 LLM 能否成为合格 wiki 维护者的核心。

---

## 2.1 三个一等操作：Ingest / Query / Lint

这是 Karpathy 原文中最关键的运行闭环，本项目必须完整保留：

### Ingest
- 新来源进入原始层
- LLM 读取并提炼要点
- 更新摘要页、实体页、专题页
- 更新 `index.md`
- 追加 `log.md` / `evolution-log/`
- 必要时更新多个关联页面，而不是只生成一篇摘要

### Query
- 查询默认先读 `index.md` 再定位相关页面
- 基于 wiki/cards 综合作答，并返回 citations
- 高价值问答结果允许回写成正式知识节点或草稿

### Lint
- 定期检查：
  - 页面冲突
  - 过期结论
  - 孤儿页
  - 缺失实体页
  - 缺失交叉引用
  - 数据空洞（可进一步搜索/补充）

## 2.2 `index.md` 与 `log.md` 是核心基础设施

按原文要求，这两者不是附属文件，而是 Wiki 可维护性的关键：

- `index.md`：面向内容导航，帮助 LLM 和人快速定位页面、分类、摘要、来源规模。
- `log.md`：面向时间演化，记录 ingest/query/lint 的时间线，便于回放与审计。

因此，MVP 即使暂时不接复杂向量检索，也必须先把 `index.md` + `log.md` 跑通。

## 2.3 Wiki 构建方法（目录即工作流）

为了符合 `LLM Wiki` 的持续编译模式，目录结构不能只是存储分类，还必须映射完整工作流。推荐采用以下四层：

### `/raw`：原始输入层
- 存放不可变原始资料：论文、指南、飞书原文、聊天摘录、网页快照、表格导出、图片等
- 只追加，不直接编辑
- 每条原始资料都应有唯一 ID、来源、时间、抓取方式

### `/compiler`：知识编译层
- 存放 ingest / normalize / summarize / cross-link / contradiction-detect / merge 的中间结果与规则
- 负责把 raw sources 编译成 wiki 页面与 cards
- 这里不是知识终态，而是“知识生成流水线”

### `/wiki`：持久化知识层
- 存放人可读页面、结构化卡片、索引与日志
- 是默认 query 的主目标
- 高价值问答写回时，也先落到这里的 `draft/` 或对应正式节点

### `/self-improvement`：自我优化层
- 存放 lint 结果、错误模式、失败案例、修复建议、prompt/策略迭代记录
- 目标不是存知识本身，而是存“系统如何变得更会维护知识”

这四层共同组成：
`raw -> compiler -> wiki -> self-improvement -> compiler` 的闭环。

## 3. 知识库结构（统一命名）

```text
pancrepal-wiki/
  .manifest.json              # 增量 ingest 账本
  _raw/                       # 草稿/暂存输入层
  raw/                         # 原始输入层（不可变）
    papers/
    guidelines/
    feishu-exports/
    community-notes/
    snapshots/
  compiler/                    # 知识编译层
    ingest/
    normalize/
    merge/
    crosslink/
    contradiction/
    publish/
  wiki/                        # 持久化知识层
    index.md
    log.md
    overview.md
    disease/pdac/
    treatment/
    nutrition/
    psychology/
    daily-care/
    clinical-trials/
    drugs/
    community/
    syntheses/
    draft/
    _meta/
  cards/                       # 机器可读卡片层
    draft/
    disease.pdac.json
    regimen.folfirinox.json
  self-improvement/            # 自我优化层
    structural-checks/
    failure-replays/
    evidence-gap-filling/
    rule-upgrades/
    regression-tests/
  graph/
    entities.jsonl
    relations.jsonl
  evolution-log/
    index.md
    2026-04/
  schemas/
    card.schema.json
    citation.schema.json
    workflow.schema.json
  adapters/
    feishu/
    getnote/
    openclaw/
    hermes/
  config/
    taxonomy.md
    domains/pancreatic-cancer.json
```

---

## 2.4 参考项目评估与合并结论

基于对 4 个 LLM-Wiki 参考项目（`nvk/llm-wiki`、`SamurAIGPT/llm-wiki-agent`、`astro-han/karpathy-llm-wiki`、`Ar9av/obsidian-wiki`）+ `graphify`（结构化抽取范式）的综合细读，结论不是“把所有优点都加进来”，而是要分为：**立即吸收、第二阶段吸收、暂不引入**。

### 2.4.1 立即吸收（纳入 v1.1 设计基线）

这些能力对 PancrePal-Wiki 的长期稳定性和医疗可信性影响最大，应直接并入主设计：

1. **Raw / Wiki / Schema 三层纪律**  
   来自 Karpathy 与 `astro-han/karpathy-llm-wiki`。继续保持：raw 只读、wiki 持久化、schema 作为行为协议。

2. **`index.md` + `log.md` + `overview.md` 三个核心文件**  
   来自 `SamurAIGPT/llm-wiki-agent`。其中：
   - `index.md` 负责导航
   - `log.md` 负责时间回放
   - `overview.md` 负责面向患者/家属的全局综述

3. **`.manifest.json` 增量账本**  
   来自 `Ar9av/obsidian-wiki`。必须记录 source path、modified_at、content_hash、pages_created、pages_updated，且以 `content_hash` 作为主判定，mtime 作为兼容辅助。

4. **页面级 `summary` + `provenance` frontmatter**  
   来自 `Ar9av/obsidian-wiki`。这是医疗场景的关键：
   - `summary` 支持廉价检索与快速概览
   - `provenance` 显式区分 extracted / inferred / ambiguous，避免把推断伪装成事实

5. **deterministic lint + semantic lint 双层检查**  
   来自 `SamurAIGPT/llm-wiki-agent` 与 `Ar9av/obsidian-wiki`。应明确拆开：
   - deterministic：索引、链接、frontmatter、manifest、canonical placement
   - semantic：冲突、过期、薄弱证据、缺失实体页、争议页缺口

6. **query 默认不写正式 wiki**  
   来自 `astro-han/karpathy-llm-wiki`。只有显式 archive/save 或命中 write-back policy 时，才写入 `wiki/draft/` 或 `wiki/syntheses/`。

7. **`syntheses/` 作为问答归档区**  
   来自 `SamurAIGPT/llm-wiki-agent`。高价值问答不直接污染概念页，而是先进入可回顾、可再编译的归档区。

8. **独立 `cross-linker` 模块**  
   来自 `Ar9av/obsidian-wiki`。补链接不应只靠 ingest 顺带完成，而要成为独立维护动作。

9. **五段式自我优化闭环**  
   结合 `nvk/llm-wiki` 与 `Ar9av/obsidian-wiki`，固定为：
   - 结构自检
   - 失败回放
   - 弱侧补证
   - 规则升级
   - 回归测试

### 2.4.2 第二阶段吸收（M2/M3 再引入）

这些能力价值很高，但不应压到第一个可用版本：

1. **query depth levels（quick / standard / deep）**  
   来自 `nvk/llm-wiki`。建议 M2 引入，先保证 quick=索引级、standard=文章级、deep=raw+跨专题级。

2. **session 持久化与 `resume`**  
   来自 `nvk/llm-wiki`。建议 M2/M3 引入，用于恢复 research / thesis 会话。

3. **status / delta / insights 仪表盘**  
   来自 `Ar9av/obsidian-wiki`。建议 M3 引入，作为内容运营与知识治理工具。

4. **thesis-driven investigation**  
   来自 `nvk/llm-wiki`。建议先在高争议医疗问题上使用，不作为所有 query 默认入口。

5. **lint 兼做迁移机制**  
   来自 `nvk/llm-wiki`。这是正确方向，但应该等目录与 frontmatter 稳定后再上，否则早期规则变化太频繁。

### 2.4.3 暂不引入或谨慎引入

1. **一开始就做通用 hub-and-spoke 大总控**  
   不建议。PDAC MVP 先做单 topic wiki，避免一开始就变成多病种总平台。

2. **一开始就上大量输出形态**  
   如 slides / report / playbook / project outputs。先把 wiki 主知识层跑稳，衍生输出后置。

3. **把患者入口直接等同于研究流水线**  
   不建议。患者对话入口应优先消费稳定 wiki，而不是直接暴露研究模式与多 agent 编排。

4. **过度依赖图谱可视化**  
   graph 很有用，但它是派生层，不应先于知识层治理。

### 2.4.4 合并后的设计建议（最终决策）

综合“4 个 wiki 参考项目 + graphify 抽取范式”后，PancrePal-Wiki 的主张是：

- **方法论**：以 Karpathy `LLM Wiki` 为上位约束
- **最小闭环**：吸收 `SamurAIGPT/llm-wiki-agent` 的 overview / syntheses / 双层 lint 思路
- **纪律边界**：吸收 `astro-han/karpathy-llm-wiki` 的极简边界与 query/lint 写入纪律
- **知识治理**：吸收 `Ar9av/obsidian-wiki` 的 manifest / provenance / summary / cross-linker / taxonomy
- **研究编排**：吸收 `nvk/llm-wiki` 的 thesis / resume / depth levels / multi-round research，但分阶段引入

一句话结论：**先把“可增量维护、可追溯、可体检”的医疗 wiki 做稳，再逐步加深 research orchestration。**

## 2.5 最小可用交互协议（来自 LLM Wiki 参考实现）

参考 `astro-han/karpathy-llm-wiki` 与其他实现，PancrePal-Wiki 必须支持一个极简但高频的三步交互协议：

1. **Ingest your first source**  
   用户可以直接提供：
   - 一个 URL
   - 一个本地文件
   - 一段粘贴文本

   系统必须把这些来源统一接入到 `raw/`，然后编译进入 `wiki/`。

2. **Ask your wiki a question**  
   用户直接提问，系统优先基于 wiki 回答，并带 citations。

3. **Keep it healthy**  
   用户可直接要求 lint，系统检查 broken links、missing index entries、stale cross-references、结构异常、证据缺口。

### 2.5.1 Source 输入需要分成“两层模型”

参考项目代码与文档表明，PancrePal-Wiki 不应只把 source 理解为 `url / file / pasted_text` 三种输入方式，而应拆成：

#### 第一层：输入通道（transport）
- **URL**：网页文章、指南页面、PubMed/期刊详情页、ClinicalTrials 页面、飞书分享链接等
- **File**：本地文件、导出文件、上传资料
- **Pasted text**：用户直接粘贴的一段资料、讨论记录、摘要、医生回复

#### 第二层：源内容类型（media/content type）
- **Text / Markdown / TXT**
- **PDF**（含可提取文本 PDF）
- **Image**：`.png/.jpg/.jpeg/.webp/.gif`
- **Screenshot / Whiteboard / Diagram / Slide Capture`**（属于 image 的高价值子类，应单独识别）
- **Image-heavy PDF / scanned PDF**
- **Chat / JSONL / transcript exports**
- **`_raw/` drafts / clipboard captures / quick notes**

也就是说：`ingest(source)` 的 source schema 至少应同时包含 `transport` 和 `content_type`。

### 2.5.1a 多模态 ingest 结论（基于参考代码/文档）

对照参考实现：
- `reference_repo/project3-obsidian-wiki/README.md` 明确支持：markdown、PDF、JSONL conversation exports、plain text logs、chat exports、meeting transcripts、images（screenshots / whiteboard photos / diagrams）
- `reference_repo/project3-obsidian-wiki/.skills/wiki-ingest/SKILL.md` 明确规定了 image 分支：转录可见文字、描述结构、提取概念、标记 ambiguity，并要求 vision-capable model
- `reference_repo/nvk-llm-wiki/claude-plugin/skills/wiki-manager/references/ingestion.md` 明确提到 `.pdf` URL 与 image metadata stub

因此，PancrePal-Wiki 的正式设计应承认：**图片、截图、白板、图表、扫描 PDF 不是边缘输入，而是正式的一等知识来源。**

### 2.5.1b 多模态输入的医疗场景要求

在医疗场景中，以下输入尤其常见，必须一等支持：
- 指南截图
- 医嘱单/化验单截图
- 幻灯片截屏
- 手写白板/讨论拍照
- 扫描版 PDF
- 聊天记录导出

这些内容进入系统后，应至少记录：
- `transport`
- `content_type`
- `vision_required`（是否需要视觉模型）
- `ocr_status`
- `provenance_risk`（图像解释导致的高推断风险）

### 2.5.1c 设计决策

因此主设计中的 source 模型应从：
- `source ∈ {url, file, pasted_text}`

扩展为：
- `source.transport ∈ {url, file, pasted_text}`
- `source.content_type ∈ {text, markdown, pdf, image, screenshot, diagram, scanned_pdf, chat_export, transcript, raw_draft}`

其中：
- `screenshot / diagram / scanned_pdf` 虽可归入 image/pdf，但为了后续策略分流，建议保留显式枚举值。
- 多模态来源写入 wiki 时，必须更严格地使用 `provenance` 标记，避免把图像解释结果误写为确定事实。

### 2.5.2 URL Ingest 作为重点能力

URL 提取在本项目中必须被设计为核心能力，而不是附属功能。原因：

- 医疗知识很多天然以链接形式传播
- 社区运营时最容易沉淀的是“发来一个链接，请收录”
- 飞书/微信/网页文章/指南更新都以 URL 为入口最自然

因此 URL ingest 至少要支持以下流程：

1. 抓取 URL 内容到 `raw/`
2. 保存来源元数据：
   - `source_url`
   - `collected_at`
   - `published_at`（若可识别）
   - `source_title`
   - `source_type`
3. 将抓取结果规范化为中间格式
4. 编译为 wiki 页面 / cards
5. 更新 `index.md`、`overview.md`、`log.md`、`.manifest.json`

### 2.5.3 URL ingest 的医疗场景额外要求

对于医疗 URL，还应额外记录：
- 是否为指南/论文/试验注册/新闻/社区帖子
- 机构或发布主体
- 是否疑似二手转述而非原始来源
- 是否需要进入待审核区而非直接进入正式知识层

### 2.5.4 设计决策

PancrePal-Wiki 的对外最小协议定义为：
- `ingest(source)`，其中 `source ∈ {url, file, pasted_text}`
- `query(question)`
- `lint(mode)`

这是用户侧与 agent 侧都应共享的最小交互面。其余复杂能力（research、thesis、status、insights）都属于增强层。

### 2.5.5 Graphify 启发：医疗 AST（Medical AST）作为派生层

参考 `safishamsi/graphify`（v4）后，新增一个明确结论：  
**AST 的核心不是“代码专用”，而是“复杂信息的层级结构化方法”。**

在本项目中，应把该思想迁移为“医疗 AST”抽取规范：

- 根节点：`patient_case`
- 一级节点：`diagnosis / treatment / adverse_event / follow_up / outcome`
- 二级节点：`treatment.chemotherapy.regimen`、`diagnosis.stage`、`diagnosis.biomarker` 等
- 关系边：`has_diagnosis`、`received_treatment`、`developed_adverse_event`、`supported_by_source`

#### 2.5.5a 设计边界（必须遵守）

1. **Graph 是派生层，不是事实主层**  
   仍以 `raw -> compiler -> wiki` 为真相主链；`graph/` 仅用于检索增强、可视化和结构分析。

2. **节点与边必须可追溯到证据**  
   每个 node/edge 至少关联：`source_id`、`evidence_span`、`provenance`。

3. **推断状态显式标记**  
   沿用 `extracted / inferred / ambiguous`，并叠加 `source_authority` 与 `evidence_grade`。

#### 2.5.5b 对应目录建议

在 `graph/` 下新增：
- `graph/medical_ast/nodes/`
- `graph/medical_ast/edges/`
- `graph/medical_ast/snapshots/`

并在自优化环节新增图谱回归项：
- orphan nodes
- dangling edges
- 高风险低证据节点告警
- wiki 与 graph 的双向一致性检查

## 2.6 入库前分级与标识（高优先级治理要求）

在医疗 wiki 中，**任何来源进入正式知识层之前，必须先完成“信源分级 + 质量分级 + 发布限制”**。这不是可选元数据，而是核心治理机制。

### 2.6.1 为什么这是高优先级要求

PancrePal-Wiki 将混合接收：
- 指南与专家共识
- 协会/学会/机构发布
- 同行评议论文
- 临床试验注册信息
- 医生公开表达
- 患者经验
- 社区帖子
- 公开搜索结果
- 图片/截图/扫描文档/视频转录

如果这些内容在进入 wiki 前不做分级与标识，就会被错误地“拉平”为同一种知识，最终导致：
- 指南与社区经验混权
- 二手转述冒充原始证据
- 图像/视频解释结果被误当作确定事实
- 患者问答时输出不区分证据强弱

因此，**分级与标识必须发生在“正式入库前”**，而不是等 query 时临时判断。

### 2.6.2 必须完成的三类标识

每个来源在进入正式 wiki/card 层前，至少必须拥有：

1. **信源类别（source authority）**  
   说明它来自什么类型的权威来源。

2. **信息质量等级（evidence grade）**  
   说明它的证据强度与可采信程度。

3. **发布限制（publication policy / display restriction）**  
   说明它能否直接面向患者展示，还是只能进入 draft / review / internal。

### 2.6.3 两阶段分级机制

分级必须分成两个阶段：

#### 阶段一：Normalize 初判
在 `NormalizedSource` 中记录：
- `source_authority_initial`
- `evidence_grade_initial`
- `grading_basis`

作用：让 compiler 在抽取与合成时就知道该来源的大致权重。

#### 阶段二：Govern 定级
在 `GovernedSource` 中记录：
- `source_authority_final`
- `evidence_grade_final`
- `grade_reason`
- `publication_policy`
- `display_restriction`

作用：决定该来源是否能进入正式知识层，以及以何种方式进入。

### 2.6.4 设计要求

- 低质量来源不能直接覆盖高质量来源结论。
- 患者经验、社区帖子、公开搜索结果，默认不能直接成为主结论。
- 图片、截图、视频、扫描 PDF 等高推断风险来源，必须显式提高 `provenance_risk`，必要时自动进入 review 流。
- Query 层后续必须消费这些分级结果：默认优先高等级来源，并对低等级来源加限制语。

### 2.6.5 写入 wiki/card 的要求

正式页面或卡片中，至少要有以下可追溯字段：
- `source_authority`
- `evidence_grade`
- `provenance`
- `review_status`

这四类字段共同决定：一个知识节点能否被系统安全地复用、展示、传播。

## 3.0 由参考项目补强的核心结构约束

### 3.0.1 必备特殊文件
- `wiki/index.md`：主导航
- `wiki/log.md`：append-only 日志
- `wiki/overview.md`：全局医疗综述页
- `.manifest.json`：增量 ingest 账本

### 3.0.2 必备特殊区域
- `wiki/draft/`：待审候选知识
- `wiki/syntheses/`：高价值问答/分析归档区
- `_raw/`：草稿暂存区（未正式编译）

### 3.0.3 页面 frontmatter 最低要求
- `title`
- `category` / `type`
- `tags`
- `sources`
- `created` / `updated`
- `summary`
- `provenance`（extracted / inferred / ambiguous）

### 3.0.4 Lint 分层要求
- **deterministic lint**：结构、索引、链接、frontmatter、canonical placement、manifest 一致性
- **semantic lint**：冲突、过期、薄弱证据、缺失实体页、争议点缺失、推断比例漂移

## 3.1 编译流水线（Compiler Pipeline）

每次新增资料时，系统不应直接“生成一段答案”，而应执行标准编译流程：

1. **capture**：接收来源并存入 `/raw`
2. **parse**：提取元数据、正文、表格、附件信息
3. **normalize**：转成统一中间格式
4. **classify**：判断 disease / treatment / nutrition / psycho / trial 等类别
5. **link**：寻找已有实体、建立 wikilink、补充 `entity_id`
6. **synthesize**：更新摘要页、专题页、实体页
7. **detect_conflict**：识别与现有知识冲突、重复、过期风险
8. **publish**：写入 `/wiki`、`/cards`、`index.md`、`log.md`
9. **reflect**：把失败案例、低质量输出、修复建议写入 `/self-improvement`

这个流程要尽量可重放、可审计、可局部重跑。

## 3.2 定期自我优化（Self-Improvement Loop）

对照 Karpathy 原文与 `nvk/llm-wiki`，自我优化不能只理解为“定期写总结”，而应是一套具体的运维闭环。

### 3.2.1 自我优化的本质
- **Karpathy 的定义**：周期性 lint，寻找矛盾、过期、孤儿页、缺失概念、缺失交叉引用、可补充的数据缺口，并由 LLM 提出新的问题和新来源。
- **nvk 的工程化实现**：把这些检查做成结构守护、恢复模式、项目感知 lint、自动修复与更深层 web-verify。

因此，本项目中的自我优化 = **Health Check + Gap Discovery + Repair Planning + Rule Upgrade + Regression Guard**。

### 3.2.2 五步自我优化法

#### A. Health Check（健康检查）
固定扫描：
- broken links
- orphan pages
- stale claims
- contradiction markers
- missing metadata/frontmatter
- index 失真（页面存在但索引没收录）
- log 断裂（关键操作没写日志）

#### B. Gap Discovery（缺口发现）
从查询失败和低质量回答中找缺口：
- 哪些问题总是答不全？
- 哪些领域只有单一来源？
- 哪些重要实体被频繁提及但没有独立页面？
- 哪些争议点缺少“正反证据并陈”的专题页？

#### C. Repair Planning（修复计划）
对问题分类：
- 结构问题：索引、链接、frontmatter、命名
- 知识问题：过期、冲突、证据薄弱
- 流程问题：ingest 漏字段、writeback 污染、路由错误
- 策略问题：prompt、taxonomy、schema 不足

#### D. Rule Upgrade（规则升级）
把单次修复上升为系统规则：
- 新 lint 规则
- 新 frontmatter 必填项
- 新 cross-link 约束
- 新 conflict 模板
- 新 thesis 模板
- 新 query routing 策略

#### E. Regression Guard（回归守护）
把失败案例沉淀为测试样本：
- query failure set
- ingest failure set
- contradiction regression set
- citation integrity set

今后每次 schema/prompt/pipeline 调整后，都要回跑这批样本，防止“修一个问题，坏一片页面”。

### 3.2.3 周期机制（具体）

#### Daily
- 跑轻量 lint
- 更新 `wiki/log.md`
- 抽取当天高价值问答，放入 `wiki/draft/`

#### Weekly
- 汇总 top query failures
- 汇总 top citation issues
- 生成 `self-improvement/weekly-review-YYYY-MM-DD.md`
- 选 1-3 个系统性问题做规则修复

#### Monthly
- 重建/刷新 `wiki/index.md`
- 做一次 `thesis-driven` 争议点复核
- 审查 taxonomy / schema / prompt / routing
- 发布新的 `knowledge_version`

### 3.2.4 医疗场景下必须新增的自我优化项

相比通用 wiki，医疗 wiki 还必须定期做：
- **guideline freshness review**：检查中外指南是否有更新
- **controversy resurfacing**：重新审视争议问题，不允许陈旧共识长期冻结
- **citation sufficiency check**：高风险建议必须有足够证据，不允许只有社区经验
- **safety drift check**：个体化建议是否逐渐偏离保守安全边界

### 3.2.5 自我优化的落盘位置
- `self-improvement/structural-checks/`
- `self-improvement/failure-replays/`
- `self-improvement/evidence-gap-filling/`
- `self-improvement/rule-upgrades/`
- `self-improvement/regression-tests/`


## 4. 知识分层与更新频率

- L1 Core Knowledge（稳定层）：1-2年更新
- L2 Clinical Guidelines（指南层）：6-12个月更新
- L3 Research Advances（前沿层）：周/日更新
- L4 Patient Experience（经验层）：实时更新

每个页面/卡片需标注：`stability_level`、`last_reviewed_at`、`knowledge_version`。

---

## 5. Skills 与 Wiki 强关联（关键原则）

Skills 是能力层，Wiki/Cards 是知识层。必须通过以下机制强绑定：

1. 统一 `entity_id`（如 `disease.pdac`）
2. 标签路由：`domain/topic/audience`
3. Skills 只读写标准库（`wiki/`、`cards/`），不直耦合渠道格式
4. 输出必须携带 `citations[]` 可追溯到具体知识节点

---

## 5.1 Skills 在本项目中的从属定位

根据 Karpathy 原始设计，**Skills 不是知识本体，而是操作 wiki 的工具化能力**。

因此本项目中：
- Wiki/Cards 才是长期资产
- Skills 是对 Wiki 执行 ingest/query/lint/writeback 的能力封装
- OpenClaw/Hermes 是访问层，不是知识主库

这意味着未来即使替换 Agent 框架，只要 wiki + schema 还在，知识能力仍然可迁移。

## 6. Skills 双向闭环（已确认）

### 6.1 读路径（提取知识）
问题 -> 路由 -> 检索 wiki/cards -> 回答 + 证据引用

### 6.2 写路径（沉淀知识）
默认 query **不直接写入正式 wiki**。只有显式 archive/save，或命中 write-back policy 时，才写入 `wiki/draft/` 或 `wiki/syntheses/`。

高价值对话 -> 结构化提炼 -> 写入 `draft/` -> 审核/自动发布 -> 更新索引与日志

### 6.3 写权限
- `read`
- `propose_write`（默认，仅写 draft）
- `publish`（策略门控）

---

## 7. Skills Contract v1.1（统一接口）

### 7.1 通用请求字段
- `request_id`
- `skill_name`, `skill_version`
- `knowledge_version`
- `patient_profile`
- `question`
- `context`（domain/topic/audience/locale）
- `policy_flags`

### 7.2 通用响应字段
- `answer`
- `recommendations[]`（包含 `entity_ids[]`）
- `citations[]`（来源、日期、URL、证据等级、置信度）
- `next_actions[]`
- `write_back`（mode/targets/change_summary）
- `errors[]`

### 7.3 MVP Skills
1. `patient_intake`
2. `plan_generate`
3. `evidence_trace`
4. `risk_check`

后续：`nutrition_support`、`psycho_support`、`followup_scheduler`、`caregiver_mode`

---

## 8. 证据与冲突处理

### 8.1 证据最小字段（必填）
- `source_name`
- `source_date`
- `url`
- `evidence_level`
- `confidence`

### 8.2 中外指南并行
冲突时不覆盖，采用并存字段：
- `consensus_cn`
- `consensus_global`
- `difference_note`
- `decision_policy`

### 8.3 进化日志
每次变更必须记录：
- `change_type`: new/update/contradiction/deprecate
- `source_type`: guideline/paper/patient-report/doctor-interview
- `reviewer`

---

## 8.1 研究编排模式（借鉴 nvk）

对于复杂主题，本项目建议引入中央协调器 + 并行研究 Agent 的编排方式：

- **coordinator**：拆解问题、分配 Agent、合并结果、生成发布候选
- **support agent**：寻找支持证据
- **opposition agent**：寻找反证与反例
- **mechanism agent**：分析机制链条
- **guideline agent**：比对中外指南
- **patient-experience agent**：提取社区经验与局限

在 PDAC 场景下，复杂研究任务默认应优先走 thesis-driven 模式，而不是宽泛搜集。

## 9. OpenClaw / Hermes 对接

- OpenClaw: `adapters/openclaw/skill_manifest.json`
- Hermes: `adapters/hermes/tool_router.yaml`

适配器必须实现：
1. 参数 schema 校验
2. 标签路由映射
3. 错误码标准化
4. 失败降级（只读问答）

建议错误码：
`SCHEMA_VALIDATION_FAILED` / `KNOWLEDGE_NOT_FOUND` / `CITATION_REQUIRED` / `POLICY_DENIED` / `WRITEBACK_FAILED` / `UPSTREAM_ADAPTER_ERROR`

---

## 10. 同步与可靠性

### 10.1 Feishu 同步策略
- 主流程：Feishu -> 本地标准库
- 回写流程：标准库 -> Feishu（受权限与审核控制）

### 10.2 降级机制
- Feishu API 限流/失败：切换只读缓存
- 写回失败：进入延迟队列重放
- 所有失败事件写入 `logs/` 与告警

---

## 10.0 治理总原则：Review > Retract > Restrict > Archive > Delete

详细规则请见：`docs/governance-retraction-principles.md`。

### 来自原始思路与参考 repo 的稳定共识

1. **raw 默认是 immutable**  
2. **raw 默认应保留，不应轻易删除**  
3. **archive 是比 delete 更优先的策略**  
4. **query / wiki 层的退役，不等于 raw 必须删除**  
5. **真正删除 raw 是例外，不是常规治理动作**  

这些共识来自 Karpathy `LLM Wiki` 原始模式，以及 `astro-han/karpathy-llm-wiki`、`SamurAIGPT/llm-wiki-agent`、`Ar9av/obsidian-wiki`、`nvk/llm-wiki` 的交叉对照。PancrePal-Wiki 在引入主动治理与 retract 机制时，仍应以这些原则作为默认基线。

PancrePal-Wiki 的治理顺序必须明确，不允许把删除作为默认动作。总体优先级顺序为：

1. **review**  
2. **retract**  
3. **restrict**  
4. **archive**  
5. **delete**（仅例外）

> 例外：若命中 **privacy / compliance / ingestion_error / immediate safety risk**，允许走 **expedited retract**，即先执行临时 retract / restrict，再补 review 记录。

### 含义说明

- **review**：前置判断动作。用于复审、重判等级、确认是否真的需要治理介入。
- **retract**：正式治理流程。用于启动 blast radius 分析、知识层清理、后续处理决策。
- **restrict**：retract 的首选处理结果。优先降低活跃度与传播能力，而不是直接删除。
- **archive**：次选处理结果。退出活跃工作集，但保留历史审计价值。
- **delete**：最终例外处理。仅用于隐私、合规、误采集或其他强理由场景。

### 设计要求

- 任何高风险或低质量来源，先进入 **review**。
- review 后若确认需要治理，再进入 **retract** 流程。
- retract 中默认优先选择 **restrict**，其次 **archive**。
- **delete 不是常规清理手段**，而是高门槛例外动作。

一句话原则：**先 review，再 retract；在 retract 中优先 restrict，其次 archive，delete 仅作例外。**

## 10.1 入库前治理红线

以下原则在正式开发前即视为治理红线：

1. 未完成 `source_authority` 与 `evidence_grade` 标识的内容，不进入正式知识层。
2. 未完成 `publication_policy` / `display_restriction` 决策的内容，不直接面向患者输出。
3. 高推断风险来源（截图、扫描件、视频解释、社区转述）默认不作为主结论来源。
4. 低等级来源可以保留，但必须以受限形式存在（draft / note / controversy / experience）。

## 11. 社区治理

- 分支：`main` / `dev` / `community`
- 流程：PR -> 自动 lint -> 人工审核 -> 合并 -> 同步
- 角色：技术组、内容组、审核组、社区成员
- 审核分层：医生审核（治疗核心）/资深患者审核（照护经验）

---

## 11.0 自我优化的五个固定模块

为避免“自我优化”过于抽象，统一落成以下五个模块：

1. **结构自检**（structural-checks）  
   检查 index、log、wikilinks、frontmatter、孤儿页、目录漂移。

2. **失败回放**（failure-replays）  
   回放 query / ingest / writeback / routing 失败案例，确认问题可复现。

3. **弱侧补证**（evidence-gap-filling）  
   识别单一来源区、争议区、薄弱证据区，并补抓来源或补建专题页。

4. **规则升级**（rule-upgrades）  
   把单次修复抽象为 lint、schema、taxonomy、prompt、routing 规则。

5. **回归测试**（regression-tests）  
   对失败样本集回跑，确认升级未引入新的系统性退化。

对应目录建议：

```text
self-improvement/
  structural-checks/
  failure-replays/
  evidence-gap-filling/
  rule-upgrades/
  regression-tests/
```

## 11.1 定期任务与自动化计划

除了功能开发，MVP 必须从一开始就安排定期维护任务：

- **每日**
  - ingest 新资料
  - 更新 `wiki/log.md`
  - 运行基础 lint
- **每周**
  - 汇总失败 query 与错误引用
  - 生成 `self-improvement/weekly-review-YYYY-MM-DD.md`
  - 回顾高价值对话，挑选写回 wiki
- **每月**
  - 更新 `index.md` 导航结构
  - 生成知识版本 `knowledge_version`
  - 审查 schema / taxonomy / prompt 变更

## 12. 里程碑（3个月）

### M1（1-4周）
- 目录与 schema 定版
- Feishu -> Markdown/Cards 同步 POC
- 首批 30-50 张 PDAC cards

### M2（5-8周）
- Web + 飞书/微信入口上线
- 4 个 MVP skills 接入
- 回答强制证据引用

### M3（9-12周）
- lint（冲突/过期/孤儿页）
- evolution-log 全量启用
- 社区协作流程上线

---

## 13. 文档唯一性约束

本文件 `docs/architecture-v1.1.md` 是**唯一主设计文档**。  
其他文档仅作历史归档或参考，不作为当前设计依据。


---

## 14. 开发周期（执行视图）

- **第1月（M1）**：规范与底座
  - 目录/Schema/Contract 锁定
  - Feishu 同步 POC 跑通
  - 首批 PDAC cards 沉淀
- **第2月（M2）**：入口与能力
  - Web + 飞书/微信入口
  - 4个MVP Skills 接入
  - 强制引用与可追溯输出
- **第3月（M3）**：质量与协作
  - lint + 冲突/过期检测
  - evolution-log 全流程启用
  - 社区 PR 审核闭环上线

## 15. 开发者与审核者招募策略

### 15.1 目标角色
- 平台后端（同步、适配、可靠性）
- Agent/Skill 工程师（OpenClaw/Hermes）
- 知识工程师（Schema/卡片/追溯）
- 前端工程师（患者体验与证据可视化）
- 医学审核者（医生/资深患者）

### 15.2 加入流程
1. Issue 报名（`[JOIN] role`）
2. 说明可投入时段与技能栈
3. 领取 first-task（1~3天可完成）
4. PR -> 自动 lint -> 人审 -> merge

### 15.3 贡献质量门槛
- 所有医学结论必须可追溯到证据
- 变更必须写入 evolution-log
- 新 skill 必须符合 contract schema

## 16. 患者病情报告导出与动态更新（新增）

新增一个患者侧高频能力：**病情整理报告导出 + 自动更新提醒**。

### 16.1 导出格式（3选）
- `markdown`（默认）
- `html`
- `pdf`

### 16.2 最小接口
- `report.generate(case_id, format=md)`
- `report.update(case_id)`（增量重编译）
- `report.notify(case_id, channel)`（更新提醒）

### 16.3 动态更新触发
当发生以下事件时触发对应病例报告更新：
- `ingest`
- `retract / restrict / archive`
- `recompile`

更新动作：
1. 定位受影响 `case_id`
2. 重建报告（默认 md，可选 html/pdf）
3. 生成变更摘要（新增/删除/证据等级变化）
4. 推送提醒（站内 / 飞书 / 微信）

### 16.4 报告结构（与患者目录模板对齐）
- 基础信息
- 治疗历史
- 确诊信息
- 基因与病理详情（含药物毒性基因如 UGT1A1、药敏性）
- 标志物与影像记录/曲线
- 并发症预防与风险
- 用药与副作用提醒
- 营养评估
- 心理评估量表

### 16.5 治理约束
- 报告必须包含：`updated_at`、`source_count`、`citations`、`evidence_grade_distribution`
- 高风险低证据内容默认折叠并标注“需复核”
- 报告是派生层，不得反写原始证据层
