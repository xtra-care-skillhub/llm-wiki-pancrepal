# LLM-Wiki 医疗通用框架架构 v1（胰腺癌MVP）

## 1. 项目目标

- 3个月内：
  - A：上线患者可用入口（Web Chat + 飞书/微信机器人）
  - B：上线知识生产/审核工作台（社区协作）
- 6-36个月：沉淀可复用知识资产，跨模型/跨Agent代际复用。

## 2. 路线选择

采用“方案3增强版”：

- Feishu 作为主库（Source of Truth）
- Get 笔记作为镜像
- 后台持续导出到可迁移标准库（Markdown Wiki + JSON Cards + 引用链）

## 3. 总体架构（五层）

1) Source Layer：指南/论文/共识/笔记API（Feishu/Get）  
2) Knowledge Compiler：Ingest/Diff/Normalize/Conflict Detect/Citation Trace  
3) Dual Store：
   - Human Wiki（Markdown）
   - Machine Cards（JSON Schema）
4) Skills & Agent Runtime：Skills Registry + OpenClaw/Hermes Adapter + Policy  
5) Channels：Web、飞书机器人、微信机器人

## 4. 知识库结构（建议）

```text
llm-wiki-core/
  sources/
  wiki/
    guides/  //介绍NCCN，CSCO针对胰腺癌和并发症，营养，心理等主题的官方指南，共识文档
    disease/pdac/
    treatment/
     - immution免疫治疗
     - 靶向治疗
     - 指南化疗等
    combination/ //并发症管理非常重要，包括消化道出血，梗阻，感染等
    clinical_trials //非常重要，临床试验详情，效果评价，毒性风险和真实病友反馈（最后这个极其重要）
    nutrition/ 
    psycho/
    resource/  //非常重要，提供病友医疗资源介绍，比如靠谱的医生，专长经验和患者口碑积累， 比如临床中心清单，比如社区绿通资源，比如心理救助热线等实用的资源
  cards/
  graph/
  logs/
  schemas/
  adapters/
```

## 5. Skills 与 Wiki 的关系（强关联）

Skills 是能力层，Wiki 是知识层；两者通过统一 `entity_id`、Schema、标签路由绑定。

- 统一实体ID：`disease.pdac`、`regimen.folfirinox`
- 分类标签路由：`domain/topic/audience`
- Skills 只读写标准卡片与页面，不直接耦合渠道格式
- 输出必须带 `citations[]` 指向具体知识节点

## 6. Skills 双向闭环（已确认）

### 读路径（提取）

用户问题 → Skill 路由 → 检索 wiki/cards → 生成回答 + 引用

### 写路径（沉淀）

高价值对话 → 结构化提炼 → 写入候选卡片/页面 → 审核/自动发布 → 更新索引

### 写权限分层

1. `read`
2. `propose_write`（默认，写入 `draft/`）
3. `publish`（策略门控后入正式库）

## 7. MVP Skills（优先4个）

1. `patient_intake`  
2. `plan_generate`  
3. `evidence_trace`  
4. `risk_check`  
（后续：nutrition/psycho/followup/caregiver）

## 8. OpenClaw / Hermes 接入

- `adapters/openclaw/skill_manifest.json`
- `adapters/hermes/tool_router.yaml`

要求：

- 参数 schema 校验
- 版本协商（knowledge_version / skill_contract_version）
- 失败降级（只读问答）

## 9. 三个月里程碑

- M1（1-4周）：Feishu结构 + 同步器v0 + 30~50张PDAC核心cards
- M2（5-8周）：Web + 飞书/微信入口 + 4个MVP skills
- M3（9-12周）：lint体系 + 社区角色协作 + 月度知识发布

## 10. 核心原则

- 先深后广：先打穿胰腺癌
- 建议必须可追溯：每条结论绑定证据
- 知识与模型策略分离版本化
- 平台可替换：知识资产不依赖单一工具
