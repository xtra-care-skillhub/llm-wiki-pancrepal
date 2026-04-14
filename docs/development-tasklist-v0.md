# Development Tasklist v0（从设计树导出）

> 目标：先实现可审计、可演化的 LLM-Wiki 内核，再扩展到多入口与智能体生态。

---

## A. Critical Path（必须先做）

1. Schema 基线
- [ ] `source_input`
- [ ] `CapturedSource`
- [ ] `NormalizedSource`
- [ ] `EnrichedSource`
- [ ] `GovernedSource`
- [ ] `source_authority / evidence_grade / policy` 枚举

2. 数据骨架与账本
- [ ] 目录骨架：`_raw/`, `raw/`, `raw/_restricted/`, `raw/_archive/`, `wiki/`, `cards/`
- [ ] `.manifest.json` 最小账本（`source_id + hash + status`）
- [ ] `index.md / log.md / overview.md` 自动更新

3. Compiler 主链路
- [ ] `capture_adapter`（URL / file / pasted text）
- [ ] `normalize_engine`（先文本）
- [ ] `enrich_engine`（entities/concepts/claims）
- [ ] `govern_engine`（authority/grade/risk/policy）
- [ ] `publish_engine`（draft/publish/restricted）

4. 最小可用查询
- [ ] `query`（必须带 citations）
- [ ] citation 可追溯到 `source_id`

### Critical Path 验收
- [ ] 一条 URL 能完整走通 capture → publish
- [ ] query 返回结果可追溯到 raw source
- [ ] 未分级内容不能进正式层

---

## B. Parallel Lanes（可并行）

### Lane B1：质量维护
- [ ] deterministic lint（索引/链接/frontmatter/manifest）
- [ ] semantic lint（冲突/过期/薄弱证据）
- [ ] cross-linker（独立维护模块）

### Lane B2：治理闭环
- [ ] review 周期（weekly/monthly）
- [ ] retract pipeline（blast radius + cleanup + optional recompile）
- [ ] `restrict/archive/delete` 联动

### Lane B3：追踪与审计
- [ ] traceability chain（source → wiki/card → answer）
- [ ] retraction audit 记录
- [ ] policy/version 变更记录

### Lane B5：医学 AST / 图谱派生层（Graphify 思路吸收）
- [ ] `graph/medical_ast` 目录与 schema（node/edge）
- [ ] 抽取状态字段：`extracted/inferred/ambiguous`
- [ ] node/edge 强制 `source_id + evidence_span`
- [ ] wiki ↔ graph 一致性检查（回归任务）

### Lane B4：入口接入（后置并行）
- [ ] OpenClaw/Hermes 最小 skill contract
- [ ] Web/Feishu/WeChat 入口接线（消费已治理知识）

### Lane B6：患者报告导出与动态更新
- [ ] `report.generate(case_id, format)`（md/html/pdf，默认 md）
- [ ] `report.update(case_id)` 增量重编译
- [ ] `report.notify(case_id, channel)` 提醒接线
- [ ] 变更摘要生成（新增/删除/证据等级变化）
- [ ] 高风险低证据段落折叠策略

---

## C. Backlog（可延后）

- [ ] query depth levels（quick/standard/deep）
- [ ] session resume（research/thesis）
- [ ] status/delta/insights 仪表盘
- [ ] thesis-driven 默认编排
- [ ] 全量多模态高级策略（video 分帧、多路 OCR 策略）
- [ ] 复杂输出形态（slides/playbook/report）

---

## M1 / M2 / M3 映射

### M1（基础内核）
- 覆盖 A 全部 + B1 的 deterministic lint

### M2（编译与治理）
- 覆盖 B1/B2 主体 + 最小接入能力

### M3（长期演化）
- 覆盖 B3 + B4 完整 + C 中高价值项

---

## 本周执行建议（按关键路径）

- Day 1: Schema + 目录 + Manifest
- Day 2: Capture Adapter
- Day 3: Normalize（文本）
- Day 4: Enrich + Govern（最小规则）
- Day 5: Publish + Query + 可追溯验收
