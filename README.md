# llm-wiki-pancrepal

[English](#english) | [中文](#中文)

---

## English

> Inspired by [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), this project builds a **PDAC-first, long-term evolving medical wiki framework** that works with agents (OpenClaw / Hermes), and is designed for **knowledge reuse across model eras**.

### Why this project
Traditional RAG is query-time retrieval. We focus on a persistent, evolving wiki:
- LLM + human co-maintained knowledge
- traceable evidence and versioned updates
- reusable by different future agent/runtime stacks

### Current scope (MVP)
- Disease focus: **PDAC (pancreatic ductal adenocarcinoma)**
- Product focus:
  - Patient-facing: Web Chat + Feishu/WeChat bot
  - Builder-facing: knowledge production & review workflow
- Storage strategy:
  - Feishu as source-of-truth
  - GetNote as mirror
  - local standard core (Markdown + JSON cards + graph)

### Development cycle (v1.1)
- **M1 (Week 1-4)**
  - finalize schema + folder conventions
  - Feishu -> Markdown/Cards sync POC
  - first 30-50 PDAC knowledge cards
- **M2 (Week 5-8)**
  - launch Web + Feishu/WeChat entry
  - integrate 4 MVP skills: `patient_intake`, `plan_generate`, `evidence_trace`, `risk_check`
  - force citation-backed answers
- **M3 (Week 9-12)**
  - lint for contradiction/staleness/orphans
  - enable evolution-log and review loop
  - community workflow online (PR -> lint -> review -> merge)

### Recruiting developers / contributors
We are actively recruiting collaborators.

#### Roles wanted
- **Backend / Platform**: sync pipeline, adapters, APIs, reliability
- **Agent / Skill Engineer**: OpenClaw/Hermes integration, skill contracts, routing
- **Data / Knowledge Engineer**: schema, card normalization, provenance, linting
- **Frontend Engineer**: patient chat UX, evidence display, timeline views
- **Clinical content reviewers** (doctor/senior patient volunteers)

#### Preferred skills
- Python/TypeScript, Markdown automation, API integration
- Feishu API / bot integration experience is a plus
- medical evidence traceability mindset

#### How to join
- Open an Issue with title prefix: `[JOIN] <role>`
- Introduce your background + available time + sample work
- Start from a good-first-task in roadmap/issues

### Docs
- Canonical design doc: `docs/architecture-v1.1.md`
- Docs index: `docs/README.md`
- Reference draft (kept for context): `reference_design.md`

### Project status
- v1.1 design consolidated
- repository initialized and synced

[Back to top](#llm-wiki-pancrepal)

---

## 中文

> 本项目受 [Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 启发，目标是构建一个**以胰腺癌（PDAC）为起点、可长期演化的医疗 Wiki 框架**，可与 OpenClaw / Hermes 等智能体协同工作，并在不同模型时代持续复用知识资产。

### 为什么做这个项目
传统 RAG 主要在“提问时检索”。本项目更关注一个持续演化的 Wiki：
- 由 LLM 与人类协同维护知识
- 每条结论尽量具备证据追溯与版本历史
- 可以被未来不同 Agent / Runtime / 应用层复用

### 当前范围（MVP）
- 病种聚焦：**PDAC（胰腺导管腺癌）**
- 产品聚焦：
  - 面向患者：Web Chat + 飞书/微信机器人
  - 面向建设者：知识生产与审核工作流
- 存储策略：
  - Feishu 作为主库（source of truth）
  - GetNote 作为镜像
  - 本地标准核心库（Markdown + JSON cards + graph）

### 开发周期（v1.1）
- **M1（第 1-4 周）**
  - 定稿 schema 与目录规范
  - 打通 Feishu -> Markdown/Cards 同步 POC
  - 形成首批 30-50 张 PDAC 知识卡片
- **M2（第 5-8 周）**
  - 上线 Web 与飞书/微信入口
  - 接入 4 个 MVP skills：`patient_intake`、`plan_generate`、`evidence_trace`、`risk_check`
  - 强制答案附带证据引用
- **M3（第 9-12 周）**
  - 建立矛盾/过期/孤儿页 lint 机制
  - 启用 evolution-log 与审核回路
  - 上线社区协作流程（PR -> lint -> review -> merge）

### 招募开发者 / 贡献者
我们正在持续招募协作者。

#### 需要的角色
- **后端 / 平台工程师**：同步链路、适配器、API、可靠性
- **Agent / Skill 工程师**：OpenClaw/Hermes 集成、skill 契约、路由
- **数据 / 知识工程师**：schema、卡片规范化、来源追溯、lint
- **前端工程师**：患者聊天体验、证据展示、时间线视图
- **医学内容审核者**（医生 / 资深患者志愿者）

#### 优先技能
- Python / TypeScript、Markdown 自动化、API 集成
- 有 Feishu API / Bot 集成经验更好
- 对医疗证据追溯与知识治理有意识

#### 如何加入
- 提交 Issue，标题前缀：`[JOIN] <role>`
- 简述你的背景、可投入时间、相关作品
- 从 roadmap / issues 中认领一个 good-first-task 开始

### 文档
- 主设计文档：`docs/architecture-v1.1.md`
- 文档索引：`docs/README.md`
- 参考草案：`reference_design.md`

### 项目状态
- 已完成 v1.1 设计整合
- 仓库已初始化并同步

[返回顶部](#llm-wiki-pancrepal)
