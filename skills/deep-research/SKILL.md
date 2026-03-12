---
name: deep-research
description: Conduct enterprise-grade research with multi-source synthesis, citation tracking, and verification. Use when user needs comprehensive analysis requiring 10+ sources, verified claims, or comparison of approaches. Triggers include "deep research", "comprehensive analysis", "research report", "compare X vs Y", or "analyze trends". Do NOT use for simple lookups, debugging, or questions answerable with 1-2 searches.
---

# Deep Research

## Decision Tree (Execute First)

```
Request Analysis
|- Simple lookup? -> STOP: Use WebSearch, not this skill
|- Debugging? -> STOP: Use standard tools, not this skill
+- Complex analysis needed? -> CONTINUE

Mode Selection
|- Initial exploration? -> quick (3 phases, 2-5 min)
|- Standard research? -> standard (6 phases, 5-10 min) [DEFAULT]
|- Critical decision? -> deep (8 phases, 10-20 min)
+- Comprehensive review? -> ultradeep (8+ phases, 20-45 min)

Execution Loop (per phase)
|- Load phase instructions from reference/methodology.md
|- Execute phase tasks
|- Spawn parallel agents if applicable
+- Update progress

Validation Gate
|- Run `python scripts/validate_report.py --report [path]`
|- Pass? -> Deliver
+- Fail? -> Fix (max 2 attempts) -> Still fails? -> Escalate
```

---

## Workflow: Clarify -> Plan -> Act -> Verify -> Report

**AUTONOMY PRINCIPLE:** Operate independently. Infer assumptions from context. Only stop for critical errors or incomprehensible queries.

### 1. Clarify (Rarely Needed)

**DEFAULT: Proceed autonomously.** Only ask if query is incomprehensible or has contradictory requirements.

Default assumptions: Technical query -> technical audience. Comparison -> balanced perspective. Trend -> recent 1-2 years. Standard mode is default.

### 2. Plan

| Mode | Time | Use Case |
|------|------|----------|
| Quick | 2-5 min | Exploration, broad overview |
| Standard | 5-10 min | Most use cases [DEFAULT] |
| Deep | 10-20 min | Important decisions |
| UltraDeep | 20-45 min | Critical analysis, max rigor |

Announce plan and execute without waiting for approval.

### 3. Act (Phase Execution)

**All modes:** Phase 1 (SCOPE) + Phase 3 (RETRIEVE) + Phase 8 (PACKAGE)
**Standard/Deep/UltraDeep add:** Phase 2 (PLAN) + Phase 4 (TRIANGULATE) + Phase 4.5 (OUTLINE REFINEMENT) + Phase 5 (SYNTHESIZE)
**Deep/UltraDeep add:** Phase 6 (CRITIQUE) + Phase 7 (REFINE)

> Consult `reference/methodology.md` for phase-specific execution details, including parallel search protocol, FFS pattern, and agent deployment.

**Parallel Execution (CRITICAL for Speed):**
- Decompose query into 5-10 independent search angles BEFORE any searches
- Launch ALL searches in single message with multiple tool calls (NOT sequential)
- Spawn 3-5 parallel agents using Task tool for deep-dive investigations

> Consult `reference/quality-gates.md` for anti-hallucination protocol and mode-specific quality thresholds.

### 4. Verify (Always Execute)

**Step 1:** Run `python scripts/verify_citations.py --report [path]` -- catches fabricated sources
**Step 2:** Run `python scripts/validate_report.py --report [path]` -- 8 automated structure/quality checks

> Consult `reference/quality-gates.md` for validation details and failure protocol.
> Consult `reference/error-handling.md` for stop rules and graceful degradation.

### 5. Report (Generate & Deliver)

Generate comprehensive report in 3 formats (Markdown + HTML + PDF) saved to `~/Documents/[TopicName]_Research_[YYYYMMDD]/`.

**Key rules:**
- Use progressive file assembly (Write/Edit per section, <=2,000 words per call)
- Bibliography: EVERY citation in full, NO placeholders, NO ranges, NO truncation
- HTML: McKinsey template via `python scripts/md_to_html.py`, verify with `python scripts/verify_html.py`
- PDF: Via generating-pdf skill (Task tool)
- Writing: Narrative-driven prose, no bullet-heavy sections, precision over vagueness

> Before writing, consult `reference/output-contract.md` for:
> - Required sections and length requirements
> - Writing standards and anti-fatigue checks
> - Source attribution standards
> - File organization and naming conventions
> - Progressive file assembly workflow
> - Auto-continuation protocol (reports >18K words)
> - HTML generation and PDF generation details

**Deliver to user:**
1. Executive summary (inline in chat)
2. Organized folder path
3. Confirmation of all three formats (MD, HTML, PDF)
4. Source quality assessment
5. Next steps (if relevant)

---

## Inputs & Assumptions

**Required:** Research question (string)
**Optional:** Mode, time constraints, required perspectives, output format

**Assumptions:** User requires verified, citation-backed information. 10-50 sources available. Time: 5-45 minutes.

---

## When to Use / NOT Use

**Use:** Comprehensive analysis (10+ sources), comparing approaches, state-of-art reviews, multi-perspective investigations, technical decisions, market/trend analysis.

**Do NOT use:** Simple lookups (WebSearch), debugging (standard tools), 1-2 search answers, time-sensitive quick answers.

---

## Scripts (Offline, Python stdlib only)

**Location:** `./scripts/`

- **research_engine.py** - Orchestration engine
- **validate_report.py** - Quality validation (8 checks)
- **verify_citations.py** - Citation verification (DOI, metadata)
- **verify_html.py** - HTML output verification
- **md_to_html.py** - Markdown to McKinsey HTML conversion
- **citation_manager.py** - Citation tracking
- **source_evaluator.py** - Credibility scoring (0-100)

---

## Progressive References (Load On-Demand)

- [Methodology](./reference/methodology.md) - 8-phase detailed execution steps
- [Output Contract](./reference/output-contract.md) - Report format, writing standards, file assembly
- [Quality Gates](./reference/quality-gates.md) - Validation, anti-hallucination, thresholds
- [Error Handling](./reference/error-handling.md) - Stop rules, degradation, error patterns
- [Report Template](./templates/report_template.md) - Output structure
- [McKinsey Template](./templates/mckinsey_report_template.html) - HTML report template

**Context Management:** Load files on-demand for current phase only. Do not preload all content.

---

## Dynamic Execution Zone

**User Query Processing:**
[User research question will be inserted here during execution]

**Retrieved Information:**
[Search results and sources will be accumulated here]

**Generated Analysis:**
[Findings, synthesis, and report content generated here]

**Note:** This section remains empty in the skill definition. Content populated during runtime only.
