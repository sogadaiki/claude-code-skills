# Quality Gates & Standards

## Mandatory Quality Standards (Always Enforce)

Every report must meet these criteria:

- 10+ sources (document in limitations if fewer)
- 3+ sources per major claim
- Executive summary <250 words
- Full citations with URLs
- Credibility assessment included
- Limitations section present
- Methodology documented
- No placeholders (TBD, TODO, [citation needed])

**Priority:** Thoroughness over speed. Quality > speed.

---

## Anti-Hallucination Protocol (CRITICAL)

- **Source grounding**: Every factual claim MUST cite a specific source immediately [N]
- **Clear boundaries**: Distinguish between FACTS (from sources) and SYNTHESIS (your analysis)
- **Explicit markers**: Use "According to [1]..." or "[1] reports..." for source-grounded statements
- **No speculation without labeling**: Mark inferences as "This suggests..." not "Research shows..."
- **Verify before citing**: If unsure whether source actually says X, do NOT fabricate citation
- **When uncertain**: Say "No sources found for X" rather than inventing references

---

## Validation Scripts

### Step 1: Citation Verification (Catches Fabricated Sources)

```bash
python scripts/verify_citations.py --report [path]
```

**Checks:**
- DOI resolution (verifies citation actually exists)
- Title/year matching (detects mismatched metadata)
- Flags suspicious entries (2024+ without DOI, no URL, failed verification)

**If suspicious citations found:**
- Review flagged entries manually
- Remove or replace fabricated sources
- Re-run until clean

### Step 2: Structure & Quality Validation

```bash
python scripts/validate_report.py --report [path]
```

**8 automated checks:**
1. Executive summary length (50-250 words)
2. Required sections present (+ recommended: Claims table, Counterevidence)
3. Citations formatted [1], [2], [3]
4. Bibliography matches citations
5. No placeholder text (TBD, TODO)
6. Word count reasonable (500-10000)
7. Minimum 10 sources
8. No broken internal links

**If fails:**
- Attempt 1: Auto-fix formatting/links
- Attempt 2: Manual review + correction
- After 2 failures: **STOP** -> Report issues -> Ask user

---

## Anti-Fatigue Quality Check (Apply to EVERY Section)

Before considering a section complete, verify:
- [ ] **Paragraph count**: >=3 paragraphs for major sections (## headings)
- [ ] **Prose-first**: <20% of content is bullet points (>=80% must be flowing prose)
- [ ] **No placeholders**: Zero instances of "Content continues", "Due to length", "[Sections X-Y]"
- [ ] **Evidence-rich**: Specific data points, statistics, quotes (not vague statements)
- [ ] **Citation density**: Major claims cited within same sentence

**If ANY check fails:** Regenerate the section before moving to next.

---

## "Loss in the Middle" Prevention

- Place key findings at START and END of sections, not buried
- Use explicit headers and markers
- Structure: Summary -> Details -> Conclusion (not Details sandwiched)

---

## Mode-Specific Quality Thresholds

| Mode | Min Sources | Avg Credibility | Min Words | Time Budget |
|------|------------|----------------|-----------|-------------|
| Quick | 10 | >60/100 | 2,000 | 2-5 min |
| Standard | 15 | >60/100 | 4,000 | 5-10 min |
| Deep | 25 | >70/100 | 6,000 | 10-20 min |
| UltraDeep | 30 | >75/100 | 10,000+ | 20-45 min |
