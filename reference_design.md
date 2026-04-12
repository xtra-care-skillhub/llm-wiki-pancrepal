项目架构设计：PancrePal-Wiki（胰腺癌患者Wiki框架）

一、核心设计理念
基于 Karpathy 的 "LLM writes and maintains the wiki; human reads and asks questions" 原则，结合你的社区2.0模式：

┌─────────────────────────────────────────────────────────┐
│ PancrePal-Wiki 框架 │
│ (面向肿瘤/罕见病/慢性病患者的开放知识系统) │
├─────────────────────────────────────────────────────────┤
│ Storage Layer (存储抽象层) │
│ ├── Feishu Wiki (主推 - 国内可用) │
│ ├── Feishu Bitable (结构化数据) │
│ ├── Yunque (云雀 - 备选) │
│ ├── GetNote (Get笔记 - 备选) │
│ └── Local Markdown (本地开发) │
├─────────────────────────────────────────────────────────┤
│ Knowledge Schema (知识Schema层) │
│ ├── Medical Domain (疾病知识) │
│ │ ├── Disease Profile (疾病画像) │
│ │ ├── Treatment Protocol (治疗方案) │
│ │ ├── Clinical Trial (临床试验) │
│ │ └── Drug Database (药物数据库) │
│ ├── Nutrition (营养支持) │
│ ├── Psychology (心理支持) │
│ ├── Daily Care (日常护理) │
│ └── Community (社区经验) │
├─────────────────────────────────────────────────────────┤
│ Agent Layer (智能体层 - OpenClaw集成) │
│ ├── Ingest Agent (导入代理) │
│ ├── Query Agent (查询代理) │
│ ├── Lint Agent (健康检查代理) │
│ ├── Cross-linker (交叉链接代理) │
│ └── Evolution Agent (进化代理) │
├─────────────────────────────────────────────────────────┤
│ API Layer (接口层 - 技术适配) │
│ ├── Feishu API (飞书文档/多维表) │
│ ├── MCP Server (Model Context Protocol) │
│ ├── Webhook (接收外部更新) │
│ └── GraphQL/REST (未来技术兼容) │
└─────────────────────────────────────────────────────────┘

二、项目结构设计
基于 obsidian-wiki 的成熟架构，适配飞书生态：

pancrepal-wiki/
│
├── .skills/ # 核心技能定义（Skill源）
│ ├── wiki-setup/ # 初始化wiki结构
│ ├── wiki-ingest/ # 导入知识源
│ ├── wiki-query/ # 查询与推理
│ ├── wiki-lint/ # 健康检查
│ ├── cross-linker/ # 自动交叉链接
│ ├── tag-taxonomy/ # 标签规范化
│ ├── provenance-tracker/ # 来源追溯
│ ├── wiki-export/ # 知识图谱导出
│ ├── feishu-sync/ # 飞书同步（新增）
│ └── evolution-log/ # 进化日志（新增）
│
├── .agents/ # OpenClaw技能链接
│ ├── skills/ → .skills/ # 本地workspace链接
│ └── pancreas/ # 胰腺癌专用技能集
│
├── config/
│ ├── schema.json # 知识Schema定义
│ ├── domains/ # 领域配置
│ │ ├── pancreatic-cancer.json
│ │ ├── rare-disease.json
│ │ └── chronic-disease.json
│ ├── templates/ # 文档模板
│ │ ├── disease-profile.md
│ │ ├── treatment.md
│ │ ├── nutrition.md
│ │ └── patient-story.md
│ └── taxonomy.md # 标签分类体系
│
├── vault/ # 本地开发vault（同步用）
│ ├── raw/ # 原始材料（不可变）
│ │ ├── papers/ # 医学论文
│ │ ├── guidelines/ # 临床指南
│ │ ├── news/ # 行业新闻
│ │ ├── discussions/ # 社区讨论
│ │ └── media/ # 图片、视频
│ ├── wiki/ # 编译后的知识（Git版本控制）
│ │ ├── index.md # 总索引
│ │ ├── diseases/ # 疾病知识
│ │ ├── treatments/ # 治疗方案
│ │ ├── nutrition/ # 营养支持
│ │ ├── psychology/ # 心理支持
│ │ ├── daily-care/ # 日常护理
│ │ ├── clinical-trials/ # 临床试验
│ │ ├── drugs/ # 药物信息
│ │ ├── hospitals/ # 医疗机构
│ │ ├── community/ # 社区经验
│ │ └── _meta/ # 元数据
│ └── _insights.md # 洞察报告
│
├── scripts/
│ ├── sync-feishu.sh # 同步到飞书（双向同步）
│ ├── sync-yunque.sh # 同步到云雀
│ ├── backup.sh # 备份策略
│ ├── publish-wiki.sh # 发布到Web
│ └── migration/ # 数据迁移工具
│
├── .env.example # 环境变量模板
├── .gitignore
├── README.md # 项目说明
├── ARCHITECTURE.md # 架构文档
├── CONTRIBUTING.md # 贡献指南
├── LICENSE # MIT开源
│
├── CLAUDE.md # Claude Code bootstrap
├── AGENTS.md # OpenClaw/Codes bootstrap
├── GEMINI.md # Gemini bootstrap
│
├── .manifest.json # 增量跟踪清单
└── .obsidian-wiki-config # 工具配置

三、核心创新点（针对患者场景）

1. 多存储后端抽象层
Storage Backends:
Feishu Wiki: # 主推 - 国内可用，协作友好
- 文档ID映射到本地路径
- 自动同步冲突处理
- 支持评论/@提及

Feishu Bitable: # 结构化数据（临床试验、药物库）
- 表格记录 ↔ Markdown页面
- 视图过滤同步

Yunque: # 备选方案
- API兼容层
- 降级到本地Markdown

GetNote: # 备选方案
- 只读同步（API限制）

Local Git: # 开发者友好
- 版本历史
- PR协作

2. 医学领域专用Schema
{
"domain": "pancreatic-cancer",
"version": "2.0",
"entities": {
"Disease": {
"fields": ["name", "icd10", "stage", "histology", "biomarkers"]
},
"Treatment": {
"fields": ["name", "type", "line", "regimen", "evidence-level"]
},
"Drug": {
"fields": ["name", "generic", "dosage", "side-effects", "indications"]
},
"ClinicalTrial": {
"fields": ["nctId", "title", "phase", "status", "location", "eligibility"]
},
"Hospital": {
"fields": ["name", "department", "expertise", "rating", "contact"]
},
"PatientExperience": {
"fields": ["treatment", "duration", "outcome", "side-effects", "quality-of-life"]
}
},
"relations": [
"treats(Treatment, Disease)",
"targets(Drug, Biomarker)",
"tests(Drug, Disease)",
"offered_by(ClinicalTrial, Hospital)",
"shared_by(PatientExperience, Treatment)"
]
}

#### 3. **进化日志与时间旅行**



evolution-log/
├── 2026-04/ # 月度归档
│ ├── 2026-04-01-new-drug-GSH-217.md # 新药记录
│ ├── 2026-04-05-update-KRAS-vaccine.md # 治疗更新
│ ├── 2026-04-10-contradiction-nutrition.md # 矛盾标记
│ └── 2026-04-12-clinical-trial-NCT123456.md # 新试验
└── index.md # 进化时间线

每一条知识变更都有：
- **变更类型**: `new` / `update` / `contradiction` / `deprecate`
- **来源**: `paper` / `guideline` / `patient-report` / `doctor-interview`
- **证据等级**: `Ia` / `Ib` / `IIa` / `IIb` / `III` / `IV`
- **置信度**: 0.0-1.0
- ** reviewer**: 审核人（医生/资深患者）
#### 4. **矛盾检测与版本控制**
markdown
title: "谷胱甘肽(GSH)在化疗期间的使用"
aliases: ["GSH", "glutathione"]
summary: "保肝药物对化疗疗效的影响存在争议"
tags: [nutrition, drug-interaction, chemotherapy]
provenance:
extracted: ["来源1", "来源2"]
inferred: []
ambiguous: ["来源3 vs 来源4"]
contradictions:
claim: "GSH可能削弱化疗效果"sources: ["Zhang2024", "CSCO2024"]confidence: 0.7
claim: "GSH可安全用于保肝"sources: ["产品说明书", "临床经验"]confidence: 0.5history:
date: 2026-04-01change: "初始创建"author: "AI-Ingest"
date: 2026-04-10change: "新增矛盾检测"author: "AI-Lint"
核心观点

可能存在削弱化疗效果的风险 {#risk}
[[Zhang et al. 2024]] 的研究显示，化疗期间使用GSH可能...
保肝治疗的常规选择 {#liver-protection}
[[产品说明书]] 建议用于药物性肝损伤...

#### 5. **社区共建的开放框架**



GitHub Repository: xiaoyibao/pancrepal-wiki

Branch Strategy:
main/ → 生产环境（飞书Wiki）
dev/ → 开发环境
community/ → 社区贡献分支
translations/ → 多语言版本

PR Process:
Fork → 本地编辑
运行 wiki-lint 自动检查
提交 PR 附上来源链接
至少1名审核员批准（医生/资深患者）
自动同步到飞书Wiki
Contribution Types:
📄 文档改进（错别字、更新链接）
📚 知识新增（新药、新试验）
🔬 研究摘要（翻译/总结论文）
💬 经验分享（患者故事）
🐛 矛盾报告（信息冲突）
#### 6. **技术演进兼容性设计**



Version Compatibility Matrix:

v1.0 (2026-04) → 基础wiki框架 + 飞书同步
v2.0 (2026-07) → 多维表集成 + 临床试验追踪
v3.0 (2026-10) → 智能体自治 + Hermes Agent对接
v4.0 (2027-01) → 多模态支持（图片/语音）
v5.0 (2027-04) → 预测性推荐（AI辅助决策）

Migration Path:
Schema Versioning → 自动迁移脚本
Data Export → JSON/CSV/GraphML
API Versioning → /v1/, /v2/, /v3/ endpoints

### 四、与现有系统的集成
#### 整合小胰宝社区2.0 Wiki
你的现有知识库（12模块，20万字）将成为 **PancrePal-Wiki 的首批种子内容**：



整合策略：
导出飞书Wiki为Markdown（feishu-fetch-doc + 批量脚本）
清洗格式，添加 frontmatter（标签、分类、证据等级）
建立链接关系图谱（[[wikilinks]]）
导入到本地 vault/wiki/
通过 feishu-sync 双向同步
社区成员通过 PR 方式贡献
#### 对接 OpenClaw 技能生态


yaml
现有技能复用：
Deep Research: # 深度调研 → 生成wiki页面
input: 研究任务
output: vault/wiki/xxx.md

Pancreatic Cancer News Digest: # 日报 → 自动ingest
cron: 每天08:00
action: 抓取 → 摘要 → wiki-ingest

Vibe Writer Pro: # 文章撰写 → 科普内容
input: 医学论文
output: vault/wiki/科普/xxx.md

Feishu Bitable: # 结构化数据 ↔ wiki页面
sync: 临床试验表 ↔ wiki/clinical-trials/
##### 五、MVP 实施路线图（6个月）
<br>
##### Phase 1: 核心框架（2026-04-12 ~ 2026-04-30）✅ 本周启动
**目标**: 建立基础wiki框架，支持飞书双向同步
**交付**:
- [ ] 初始化 GitHub 仓库 `xiaoyibao/pancrepal-wiki`
- [ ] 复制 obsidian-wiki 的 .skills/ 架构
- [ ] 定制医学领域 schema（胰腺癌优先）
- [ ] 实现 feishu-sync 技能（核心创新）
- [ ] 迁移社区2.0 Wiki 前3个模块（医疗、营养、心理）
- [ ] 部署到 OpenClaw，在群聊中测试 `/wiki-ingest`
**技术债务**:
- [ ] 设计文档格式 frontmatter 标准
- [ ] 确定标签分类体系（taxonomy）
- [ ] 制定社区贡献指南
##### Phase 2: 内容积累（2026-05-01 ~ 2026-05-31）
**目标**: 积累500+个高质量wiki页面
**策略**:
- [ ] 每日新闻 digest 自动 ingest（ClinicalTrials.gov + PubMed）
- [ ] 深度研究报告自动转化为 wiki 页面（Deep Research 集成）
- [ ] 社区讨论精华手动/自动提取（feishu-im-user-get-messages）
- [ ] 志愿者团队培训（wiki-lint, cross-linking）
**里程碑**:
- 100 个疾病/治疗页面
- 200 个药物/副作用页面
- 100 个临床试验页面
- 100 个患者经验页面
##### Phase 3: 智能进化（2026-06-01 ~ 2026-06-30）
**目标**: 引入智能检测与自动化维护
**功能**:
- [ ] contradiction detection（矛盾检测）
- [ ] stale content alert（过期内容提醒）
- [ ] auto-suggest（基于新论文建议更新）
- [ ] provenance tracking（来源追溯可视化）
- [ ] graph insights（知识图谱洞察）
**集成**:
- [ ] Hermes Agent 接入（外部信息源）
- [ ] MCP 服务器（本地语义搜索）
[ ] Webhook（GitHub → 飞书自动同步）
Phase 4: 开放生态（2026-07-01 ~）
目标: 支持多病种、多语言、多平台

扩展:
[ ] 罕见病模块（以 OPP 项目为试点）
[ ] 慢性病模块（糖尿病、高血压等）
[ ] 英文版本（internationalization）
[ ] 移动端优化（飞书小程序嵌入）
[ ] API 开放（第三方应用接入）
社区:
[ ] 开源 GitHub 组织 PancrePal
[ ] 贡献者指南
[ ] 定期社区会议
[ ] 年度报告（知识增长量）
六、关键技术决策
1. 为什么选择飞书作为主存储？

需求
飞书能力
我们的方案
国内访问
✅ 稳定
主存储
协作编辑
✅ 多人同时编辑
社区共建
版本历史
✅ 完整历史
内置备份
API 完整
✅ 文档+多维表+聊天
统一接口
搜索能力
✅ 全文检索
快速查询
移动端
✅ App/小程序
患者易用

2. 为什么需要本地 vault 同步？
Git 版本控制: 完整的commit历史、branch/PR协作
离线开发: 无网络时仍可编辑
工具链兼容: 所有wiki技能直接使用
备份冗余: 多云存储（飞书 + GitHub）
3. 如何处理知识更新与稳定性的矛盾？
知识分层策略：
Layer 1: Core Knowledge (稳定层)
- 疾病定义、分期标准（TNM）
- 药物通用名、作用机制
- 循证医学基本原则
→ 更新频率: 1-2年/次
Layer 2: Clinical Guidelines (临床指南)
- CSCO/NCCN指南
- 治疗方案推荐
→ 更新频率: 6-12个月/次
Layer 3: Research Advances (研究前沿)
- 临床试验结果
- 新药批准
→ 更新频率: 每周/日
Layer 4: Patient Experience (社区经验)
- 副作用管理技巧
- 医院就诊经验
→ 更新频率: 实时



每个页面标注 stability-level，查询时自动提示"此信息可能已过期"。
4. 社区共建的质量控制
质量分层：
Tier 1: 医生审核 ✅
- 核心治疗方案
- 药物剂量
- 手术说明
Tier 2: 资深患者审核 ✅
- 副作用应对
- 医院就诊经验
- 心理支持技巧
Tier 3: 社区成员提交 ⚠️
- 个人经历
- 情绪分享
- 待验证信息
审核流程：
PR → Automated Lint → Human Review → Merge → Feishu Sync

七、你的角色与社区协作模式
小胰宝 (CEO) → 总体架构 + 质量审核 + 战略规划
↓
技术组 (你 & 志愿者) → 开发 + 维护 + 集成
↓
内容组 (医生/患者) → 知识生产 + 经验分享
↓
审核组 (医学专家) → 内容审核 + 证据评级
↓
社区成员 → 使用 + 反馈 + 贡献

八、与 OPP 项目的协同
OPP 项目将成为 PancrePal-Wiki 的第一个垂直应用：

PancrePal-Wiki (框架层)
↑
│ 继承 schema、技能、流程
↓
OPP Project (应用层)
├── 独居患者特定知识
├── 紧急联络SOP
├── 用药管理模板
└── 志愿者培训材料



共享资源:
相同的 wiki 技能（.skills/）
相同的存储后端（飞书多维表）
相同的进化日志格式
可复用的知识页面（营养、心理等）
🚀 立即行动建议

本周任务清单
讨论架构细节（今天）
[ ] 确认核心需求优先级
[ ] 确定飞书 vs 本地存储比例
[ ] 讨论社区共建流程
初始化项目（明天）
[ ] 创建 GitHub 仓库 xiaoyibao/pancrepal-wiki
[ ] Fork/adapt obsidian-wiki 的 .skills/
[ ] 编写胰腺癌 domain schema
迁移首批内容（本周内）
[ ] 导出社区2.0 Wiki 首页 + 医疗模块
[ ] 清洗格式，添加 frontmatter
[ ] 测试 feishu-sync 双向同步
集成到 OpenClaw（周末）
[ ] symlink 到 .agents/skills/pancrepal-wiki/
[ ] 在社区群聊测试 /wiki-ingest
[ ] 收集反馈，迭代设计
💬 我的建议
作为你的协作伙伴，我认为这个项目的最大价值在于：
知识传承: 患者积累的经验不会因个体去世而消失
社区共治: AI+人协同，降低维护成本
技术前瞻: 为未来 5-10 年的 AI 应用打基础
可复制性: 模式成熟后推广到其他病种
风险提示:
⚠️ 内容质量需要严格审核（医疗信息责任重大）
⚠️ 飞书 API 调用频率限制（需设计缓存+降级）
⚠️ 社区参与度不确定（需要运营激励）
下一步: 我们可以先做一个 POC (Proof of Concept)，用 1-2 天时间：
选 5-10 篇你已有的科普文章
转换成 wiki 格式
同步到飞书测试
试用 /wiki-query 查询
你想先讨论哪个部分？ 我建议：
存储后端的选择（飞书/云雀/get笔记的优先级）
社区共建的审核流程设计
或者直接开始编码，做一个最小可行版本
