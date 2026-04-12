# architecture v1.1 增量建议（基于 `reference_design.md`）

## 已吸收的高价值点
1. 存储后端抽象：Feishu Wiki / Bitable / Yunque / Get / Local Markdown
2. 医学分层知识结构：疾病、治疗、临床试验、药物、营养、心理、护理、社区经验
3. 进化日志（evolution-log）与时间线管理
4. 稳定性分层（Core / Guideline / Research / Experience）
5. 社区共建与 PR 审核流程

## 建议立刻补齐（关键）
1. **统一命名**：`vault/wiki/diseases` vs `wiki/disease/pdac` 二选一，避免路径分裂。
2. **契约先行**：所有 agent/skill 输出统一走 `skills-contract-v1.md` 的 request/response schema。
3. **双向同步降级策略**：
   - Feishu API 限流/失败时进入只读缓存模式
   - 延迟队列重放（避免丢更新）
4. **证据最小字段强制**：`source_name/source_date/url/evidence_level/confidence` 必填。
5. **变更审计**：所有 write-back 强制写入 `logs/` + `evolution-log/`。

## 结构建议（对齐现有文档）
- 保留 `docs/architecture-v1.md` 作为主架构文档。
- `reference_design.md` 作为扩展草案，拆分为：
  - `docs/storage-backends.md`
  - `docs/community-governance.md`
  - `docs/evolution-log-spec.md`

## 一周可执行清单
1. 锁定目录规范与命名（半天）
2. 定义 Feishu 文档 frontmatter 标准（半天）
3. 做 `feishu -> markdown/cards` 单向同步 POC（1天）
4. 接入 `patient_intake/plan_generate/evidence_trace/risk_check` 的空实现（1天）
5. 打通 Web + 飞书机器人只读问答（1天）

