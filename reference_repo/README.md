# Reference Repositories

本目录统一保存 `llm-wiki-pancrepal` 的工程参考仓库，用于吸收成熟实现思路，而不是直接耦合复制。

## 仓库列表

### 1. `nvk-llm-wiki`
- 来源：<https://github.com/nvk/llm-wiki>
- 重点借鉴：
  - Hub-and-Spoke 架构（中央协调器 + 并行研究 Agent）
  - 三种研究模式（standard / deep / blitz-or-retardmax）
  - thesis-driven investigation（论点驱动研究）
  - fuzzy intent router（自然语言意图路由）
  - `_index.md` / `log.md` / structural guardian
  - project-aware outputs / resume mode

### 2. `project1-llm-wiki-agent`
- 来源：<https://github.com/SamurAIGPT/llm-wiki-agent>
- 重点借鉴：
  - `raw/ -> wiki/ -> graph/` 最小闭环
  - `overview.md` / `index.md` / `log.md`
  - contradiction flags
  - graph export / graph.html

### 3. `project2-karpathy-llm-wiki`
- 来源：<https://github.com/astro-han/karpathy-llm-wiki>
- 重点借鉴：
  - 对 Karpathy 原始模式的轻量实现
  - 最简目录结构与三操作（Ingest / Query / Lint）
  - agentskills.io 兼容思路

### 4. `project3-obsidian-wiki`
- 来源：<https://github.com/Ar9av/obsidian-wiki>
- 重点借鉴：
  - 多 Agent / 多工具兼容
  - `.manifest.json` 增量跟踪
  - provenance tracking
  - cross-linking / taxonomy / lint / graph export
  - `_raw/` staging / optional semantic search

## 使用原则
- 以 Karpathy `LLM Wiki` 为上位方法论。
- 以 `docs/architecture-v1.1.md` 为唯一主设计文档。
- 参考仓库用于吸收优势实现，不直接决定主架构。
