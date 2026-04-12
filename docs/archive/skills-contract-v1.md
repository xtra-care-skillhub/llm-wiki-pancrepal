# Skills Contract v1（OpenClaw / Hermes 对接）

## 1. 目标
定义 LLM-Wiki 医疗框架的统一 Skill 接口，确保：
- 可双向（读知识 + 沉淀知识）
- 可追溯（证据链）
- 可迁移（与渠道解耦）

## 2. 通用调用协议

### 2.1 Request
```json
{
  "request_id": "uuid",
  "skill_name": "plan_generate",
  "skill_version": "1.0.0",
  "knowledge_version": "2026-04",
  "patient_profile": {
    "patient_id": "anon-001",
    "disease": "pdac",
    "stage": "III",
    "biomarkers": [],
    "ecog": 1,
    "comorbidities": [],
    "current_regimen": "",
    "allergies": []
  },
  "question": "...",
  "context": {
    "domain": "oncology",
    "topic": "treatment",
    "audience": "patient",
    "locale": "zh-CN"
  },
  "policy_flags": {
    "allow_auto_personalized": true,
    "write_mode": "propose_write"
  }
}
```

### 2.2 Response
```json
{
  "request_id": "uuid",
  "skill_name": "plan_generate",
  "skill_version": "1.0.0",
  "answer": "...",
  "recommendations": [
    {
      "id": "rec-1",
      "text": "...",
      "risk_level": "medium",
      "entity_ids": ["disease.pdac", "regimen.folfirinox"]
    }
  ],
  "citations": [
    {
      "citation_id": "cit-1",
      "entity_id": "regimen.folfirinox",
      "source_type": "guideline",
      "source_name": "NCCN/CSCO",
      "source_date": "2025-01-01",
      "url": "https://..."
    }
  ],
  "next_actions": ["..."],
  "write_back": {
    "mode": "propose_write",
    "draft_targets": ["wiki/draft/...", "cards/draft/..."],
    "change_summary": "..."
  },
  "errors": []
}
```

## 3. Skills 清单

### 3.1 `patient_intake`（MVP）
- 输入：对话文本/表单
- 输出：结构化 `patient_profile`
- 写权限：`propose_write`

### 3.2 `plan_generate`（MVP）
- 输入：`patient_profile + question`
- 输出：个体化建议 `recommendations[]`
- 要求：必须附 `citations[]`

### 3.3 `evidence_trace`（MVP）
- 输入：建议项或问题
- 输出：中外指南/文献证据链 + 差异说明

### 3.4 `risk_check`（MVP）
- 输入：方案与患者画像
- 输出：风险分级、禁忌、预警

### 3.5 后续
`nutrition_support`, `psycho_support`, `followup_scheduler`, `caregiver_mode`

## 4. 与 Wiki 分类/结构的绑定规则

1. Skill 必须声明可见域：`domain/topic/audience`  
2. Skill 输出必须包含 `entity_ids[]` 指向知识节点  
3. 引用必须可回溯到 `wiki/*` 或 `cards/*`  
4. 仅允许写入标准层：`wiki/`、`cards/`（禁止直写渠道格式）

## 5. 双向闭环约束

### 读路径
问题 -> 路由 -> 检索 -> 回答 + 引用

### 写路径
高价值对话 -> 提炼 -> `draft/` -> 审核/自动发布 -> 索引更新

## 6. 写权限模型
- `read`：只读
- `propose_write`：写 `draft/`（默认）
- `publish`：入正式库（受策略门控）

## 7. 版本与兼容
- `skill_version`：能力接口版本
- `knowledge_version`：知识发布版本（如 `2026-04`）
- 不兼容变更：主版本递增（v1 -> v2）

## 8. 错误码（建议）
- `SCHEMA_VALIDATION_FAILED`
- `KNOWLEDGE_NOT_FOUND`
- `CITATION_REQUIRED`
- `POLICY_DENIED`
- `WRITEBACK_FAILED`
- `UPSTREAM_ADAPTER_ERROR`

## 9. OpenClaw / Hermes 适配文件
- `adapters/openclaw/skill_manifest.json`
- `adapters/hermes/tool_router.yaml`

两端适配器都必须实现：
- 参数校验
- 路由标签映射
- 错误码标准化
- 降级策略（失败时只读问答）

