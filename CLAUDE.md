# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a study group repository for learning AI Engineering through Chip Huyen's "AI Engineering" book (칩 후옌의 『AI 엔지니어링』). The study runs for 8 weeks (2025.11.20 ~ 2026.01.15) with weekly sessions on Thursdays at 11:00-12:30.

## Repository Structure

```
ai-engineering-study/
├── weeks/                  # Weekly study materials
│   ├── week01-08/
│   │   ├── presentation.md # Presenter's slides
│   │   ├── summary.md      # Consolidated summary by assigned integrator
│   │   └── ...             # Individual notes (free format)
└── references/             # Reference materials (papers, articles, code examples)
```

## Content Structure

Each week follows this pattern:
- **1 presenter** delivers the weekly content presentation
- **1 integrator** consolidates everyone's notes into a unified summary
- All participants contribute individual learning notes in their preferred format

### Weekly Topics Schedule

| Week | Date | Chapters | Topics | Presenter | Integrator |
|:---:|:---:|:---:|:---|:---:|:---:|
| 1 | 11/20 | 1 | AI Engineering concepts, cases, stack | 김성언 | 이은정 |
| 2 | 11/27 | 2 | Data/Architecture/Sampling | 이은정 | 안태현 |
| 3 | 12/4 | 3-4 | Model & System Evaluation | 허채연 | 김성언 |
| 4 | 12/18 | 5 | Prompt Engineering/Defense | 안태현 | 이은정 |
| 5 | 12/25 | 6 | RAG/Agents | 김성언 | 허채연 |
| 6 | 1/1 | 7 | Fine-tuning (LoRA/PEFT/Quantization) | 이은정 | 안태현 |
| 7 | 1/8 | 8-9 | Dataset Engineering + Inference Optimization | 허채연 | 김성언 |
| 8 | 1/15 | 10 | Architecture & User Feedback | 안태현 | 이은정 |

## Working with Study Materials

### File Templates

**presentation.md template:**
```markdown
# 발표 자료

**발표자**: [Name]
**날짜**: [Date]

---

## 내용

[Presentation content]
```

**summary.md template:**
```markdown
# 통합 정리

**정리자**: [Name]
**날짜**: [Date]

---

## 주요 내용

[Consolidated summary]
```

### Language

- All documentation and study materials are in Korean (한국어)
- File names and structure use English
- Maintain Korean language when working with content in presentation.md and summary.md files

### Adding New Content

When creating or editing study materials:
1. Maintain the template structure for presentation.md and summary.md
2. Place additional materials (individual notes, code examples) in the respective week's folder
3. Reference materials (papers, articles, external resources) go in the `references/` directory
