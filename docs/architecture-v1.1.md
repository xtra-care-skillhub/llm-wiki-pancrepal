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
    disease/pdac/
    treatment/
    nutrition/
    psychology/
    daily-care/
    clinical-trials/
    drugs/
    community/
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

## 2.4 来自 `nvk/llm-wiki` 的可借鉴优势
## 2.4.1 其他参考项目的补充价值

除 `nvk/llm-wiki` 外，以下项目也对本项目有直接参考价值：

- **SamurAIGPT/llm-wiki-agent**：适合作为最小闭环参考，尤其是 `overview.md`、`graph/`、contradiction flag、query answer 写回 syntheses。
- **astro-han/karpathy-llm-wiki**：适合作为 Karpathy 原始模式的轻量实现参考，提醒我们不要过早工程过度。
- **Ar9av/obsidian-wiki**：适合作为大规模技能化、增量 ingest、provenance tracking、cross-linking、taxonomy 治理的工程参考。

统一原则：**不直接复制实现，按主设计文档吸收其稳定优势。**


参考仓库：<https://github.com/nvk/llm-wiki>

除前述基础点外，还应正式吸收以下设计：

### A. Hub-and-Spoke 研究架构
- 由一个中央协调器负责任务分解、Agent 调度、结果汇总、最终编译
- 多个子 Agent 分别负责一个研究视角或子命题
- 对本项目的映射：适用于复杂医疗议题，如“某方案对 KRAS G12D PDAC 的适应性、风险与证据分层”

### B. 三种研究模式
- **Standard**：默认 5 个并行 Agent，适合日常更新与专题补全
- **Deep**：8 个 Agent，适合争议问题、治疗比较、指南差异分析
- **Blitz / Retardmax**：10 个 Agent，适合时效性热点或快速情报汇总

在 PancrePal-Wiki 中，这三种模式应保留为编译/研究策略，而不是 UI 功能。不同模式影响的是：搜索广度、证据冗余、反证力度、时间预算。

### C. Thesis-Driven Investigation
- 不从泛话题出发，而从明确 claim / thesis 出发
- 同时组织支持证据、反对证据、机制证据、综述证据
- 最终要求给出结论：supported / mixed / contradicted / insufficient

这对医疗 Wiki 极其重要，因为很多真实问题本质上都是争议命题，而不是中性百科条目。

### D. 智能意图路由
- URL / 文件 -> ingest
- 自然语言问题 -> query
- 研究主题或命题 -> research / thesis
- “我做到哪了” -> resume

这类路由逻辑适合后续接入 OpenClaw / Hermes，但本质仍属于 access layer，而不是知识层。

### E. 项目化输出
- 报告、时间线、slides、比较表、playbook 等产物应进入独立 output/project 区域
- 不应把所有衍生产物都混入 wiki 主知识页

在不偏离 Karpathy 原始范式的前提下，这个项目有几项非常值得吸收的工程化优点：

1. **One topic, one wiki**  
   每个主题一个独立 wiki，降低跨主题噪音。对本项目的推论是：即使未来扩展到罕见病/慢病，也应保持 `pdac` 作为独立 topic wiki，而不是一开始就混成一个巨大总库。

2. **`_index.md` / `index.md` 优先导航**  
   查询时先读索引，再钻取页面，而不是全库盲扫。这与 Karpathy 原文完全一致，但 nvk 把它工程化得更彻底。

3. **Structural guardian（结构守护）**  
   每次操作后自动检查 wiki 完整性，并自动修复轻量问题。这对于医疗 wiki 很重要，因为结构错误会直接影响后续回答质量。

4. **Resume mode（上下文恢复）**  
   通过最近日志、最近更新页面、最近研究会话恢复“上次做到哪”。这非常适合长期疾病知识建设。

5. **Thesis-driven research（命题驱动研究）**  
   不是宽泛搜集，而是围绕一个明确 claim，同步搜支持证据、反证、机制证据、综述证据，并要求给出 verdict。这很适合医疗争议问题。

6. **Project-aware outputs（项目化输出）**  
   报告、图表、代码、总结等衍生产物进入项目输出区，而不是散落在 wiki 中。

7. **Append-only log**  
   统一时间日志、可 grep、可恢复，这是 Karpathy 提出的 log 思路的强化版。

结论：**Karpathy 提供范式，nvk 提供工程化细节。** 本项目应采用“Karpathy as doctrine, nvk as implementation reference”的路线。

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

