# 治理与撤回原则（Review / Retract）v1

> 本文档为治理规则的独立规范文件。  
> 主设计文档仅保留摘要，详细规则以本文档为准。

## 1) 稳定共识（来自原始思路与参考 repo）

1. raw 默认是 immutable  
2. raw 默认应保留，不应轻易删除  
3. archive 是比 delete 更优先的策略  
4. query / wiki 层的退役，不等于 raw 必须删除  
5. 真正删除 raw 是例外，不是常规治理动作

## 2) 治理总顺序

**Review > Retract > Restrict > Archive > Delete**

含义：
- **Review**：前置判断（复审、重判等级、确认是否需要治理）
- **Retract**：正式治理流程（影响分析、清理、标记、记录）
- **Restrict**：首选处理（降活跃度与传播能力）
- **Archive**：次选处理（退出活跃工作集，保留审计）
- **Delete**：例外处理（仅高门槛场景）

## 3) 紧急例外（Expedited Retract）

命中以下场景可先执行临时 retract/restrict，再补 review：
- privacy
- compliance
- ingestion_error
- immediate safety risk

## 4) Retract 流水线（最小要求）

1. 识别 source（`source_id`/path/title）  
2. 分类 reason（质量/安全/合规/被替代/误采集）  
3. 计算 blast radius（pages/cards/claims）  
4. 决定模式（restrict / archive / delete）  
5. 清理 wiki/card 引用并标记受影响 claims  
6. 可选 recompile  
7. 永久记录 retraction 日志

## 5) 默认策略

- 高风险或低质量来源：先 review，再 retract
- retract 中默认优先 restrict，其次 archive
- delete 仅用于隐私、合规、误采集或强理由场景

## 6) Raw 生命周期分区（配套）

- `_raw/`：暂存输入
- `raw/`：活跃证据
- `raw/_restricted/`：受限证据
- `raw/_archive/`：历史归档
- `deleted`：例外终态

