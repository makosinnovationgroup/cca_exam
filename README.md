# CCA-F Exam Prep

Study materials for the **Claude Certified Architect — Foundations** exam. Notes are written for Obsidian (YAML frontmatter + wiki-style links) but render fine as plain Markdown on GitHub.

## Exam at a glance

- **5 domains**, 6 scenarios, passing score 720
- **Target**: 85%+ on practice banks before sitting the real exam
- **Core technologies**: Claude Code, Claude Agent SDK, Claude API, MCP

## Top-level files

| File | Purpose |
| --- | --- |
| `cca_exam_guide.md` | Markdown rendering of the official exam guide — domains, task statements, scenarios, scoring. |
| `cca_docs_roadmap.md` | Original `docs.claude.com` reading roadmap organized by domain. |
| `cca_docs_roadmap_1.md` | Updated cross-domain roadmap revision. |
| `cca_progress.md` | Progress tracker — status dashboard, practice scores, weak spots, target date. |
| `cca_practice_bank_1.md` | 54-question mixed practice bank covering all 5 domains and 6 scenarios. |

## Domain folders

Each domain folder contains a **roadmap** (docs to read + key concepts), **exercises** (hands-on builds), and a **practice bank v2** (drill questions).

| Folder | Domain | Weight |
| --- | --- | --- |
| `domain_1/` | Agentic Architecture & Orchestration | 27% |
| `domain_2/` | Tool Design & MCP Integration | 18% |
| `domain_3/` | Claude Code Configuration & Workflows | 20% |
| `domain_4/` | Prompt Engineering & Structured Output | 20% |
| `domain_5/` | Context Management & Reliability | 15% |

Inside each folder:

- `cca_domainN_roadmap.md` — reading + concept map
- `cca_domainN_exercises.md` — hands-on exercises
- `cca_domainN_practice_v2.md` — drill questions
- (Domain 1 also keeps the original `cca_domain1_practice.md` for reference)

## Workflow

1. Read the relevant section of `cca_exam_guide.md`
2. Walk the domain roadmap in `domain_N/cca_domainN_roadmap.md`
3. Build the hands-on exercises end-to-end
4. Drill the domain's `_practice_v2` bank
5. Take the mixed `cca_practice_bank_1.md` as a cold check
6. Update `cca_progress.md` with scores and weak spots

## Notes

These files use Obsidian frontmatter and `[[wiki-links]]`. Links won't resolve as clickable on GitHub but the filenames map 1:1.
