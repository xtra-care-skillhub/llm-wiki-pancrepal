# llm-wiki-pancrepal

> Inspired by [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), this project builds a **PDAC-first, long-term evolving medical wiki framework** that works with agents (OpenClaw / Hermes), and is designed for **knowledge reuse across model eras**.

## Why this project
Traditional RAG is query-time retrieval. We focus on a persistent, evolving wiki:
- LLM + human co-maintained knowledge
- traceable evidence and versioned updates
- reusable by different future agent/runtime stacks

## Current scope (MVP)
- Disease focus: **PDAC (pancreatic ductal adenocarcinoma)**
- Product focus:
  - Patient-facing: Web Chat + Feishu/WeChat bot
  - Builder-facing: knowledge production & review workflow
- Storage strategy:
  - Feishu as source-of-truth
  - GetNote as mirror
  - local standard core (Markdown + JSON cards + graph)

## Development cycle (v1.1)
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

## Recruiting developers / contributors
We are actively recruiting collaborators.

### Roles wanted
- **Backend / Platform**: sync pipeline, adapters, APIs, reliability
- **Agent / Skill Engineer**: OpenClaw/Hermes integration, skill contracts, routing
- **Data / Knowledge Engineer**: schema, card normalization, provenance, linting
- **Frontend Engineer**: patient chat UX, evidence display, timeline views
- **Clinical content reviewers** (doctor/senior patient volunteers)

### Preferred skills
- Python/TypeScript, Markdown automation, API integration
- Feishu API / bot integration experience is a plus
- medical evidence traceability mindset

### How to join
- Open an Issue with title prefix: `[JOIN] <role>`
- Introduce your background + available time + sample work
- Start from a good-first-task in roadmap/issues

## Docs
- Canonical design doc: `docs/architecture-v1.1.md`
- Docs index: `docs/README.md`
- Reference draft (kept for context): `reference_design.md`

## Project status
- v1.1 design consolidated
- repository initialized and synced

