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

## 2. 总体架构（五层）

1) **Source Layer**：指南、论文、飞书、Get、社区素材  
2) **Knowledge Compiler**：ingest / diff / normalize / conflict detect / citation trace  
3) **Dual Knowledge Store**：
   - Human Wiki（Markdown）
   - Machine Cards（JSON Schema）
4) **Skills & Agent Runtime**：Skills Registry + Policy + OpenClaw/Hermes adapters  
5) **Channels**：Web Chat、飞书机器人、微信机器人

---

## 3. 知识库结构（统一命名）

```text
pancrepal-wiki/
  sources/                     # 原始证据（不可变）
  wiki/                        # 人类可读知识层
    index.md
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
  graph/
    entities.jsonl
    relations.jsonl
  logs/
    ingest-log.md
    query-log.md
    lint-log.md
  evolution-log/
    index.md
    2026-04/
  schemas/
    card.schema.json
    citation.schema.json
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

