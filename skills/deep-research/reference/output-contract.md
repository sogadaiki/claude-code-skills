# Output Contract & Writing Standards

## Report Format

**Format:** Comprehensive markdown report following [template](../templates/report_template.md) EXACTLY

### Required Sections (all must be detailed)

- Executive Summary (2-3 concise paragraphs, 50-250 words)
- Introduction (2-3 paragraphs: question, scope, methodology, assumptions)
- Main Analysis (4-8 findings, each 300-500 words with citations [1], [2], [3])
- Synthesis & Insights (500-1000 words: patterns, novel insights, implications)
- Limitations & Caveats (2-3 paragraphs: gaps, assumptions, uncertainties)
- Recommendations (3-5 immediate actions, 3-5 next steps, 3-5 further research)
- **Bibliography (CRITICAL - see rules below)**
- Methodology Appendix (2-3 paragraphs: process, sources, verification)

### Bibliography Requirements (ZERO TOLERANCE)

Report is UNUSABLE without complete bibliography:

- MUST include EVERY citation [N] used in report body (if report has [1]-[50], write all 50 entries)
- Format: [N] Author/Org (Year). "Title". Publication. URL (Retrieved: Date)
- Each entry on its own line, complete with all metadata
- NO placeholders: NEVER use "[8-75] Additional citations", "...continue...", "etc.", "[Continue with sources...]"
- NO ranges: Write [3], [4], [5]... individually, NOT "[3-50]"
- NO truncation: If 30 sources cited, write all 30 entries in full
- Validation WILL FAIL if bibliography contains placeholders or missing citations

### Strictly Prohibited

- Placeholder text (TBD, TODO, [citation needed])
- Uncited major claims
- Broken links
- Missing required sections
- **Short summaries instead of detailed analysis**
- **Vague statements without specific evidence**

---

## Writing Standards

- **Narrative-driven**: Write in flowing prose with complete sentences that build understanding progressively
- **Precision**: Choose each word deliberately - every word must carry intention
- **Economy**: Eliminate fluff, unnecessary adjectives, fancy grammar
- **Clarity**: Use precise technical terms, avoid ambiguity. Embed exact numbers in sentences, not bullets
- **Directness**: State findings clearly without embellishment
- **Signal-to-noise**: High information density, respect reader's time
- **Bullet discipline**: Use bullets only for distinct lists (products, companies, steps). Default to prose paragraphs
- **Examples of precision**:
  - Bad: "significantly improved outcomes" -> Good: "reduced mortality 23% (p<0.01)"
  - Bad: "several studies suggest" -> Good: "5 RCTs (n=1,847) show"
  - Bad: "potentially beneficial" -> Good: "increased biomarker X by 15%"
  - Bad: "* Market: $2.4B" -> Good: "The market reached $2.4 billion in 2023, driven by consumer demand [1]."

---

## Bullet Point Policy (Anti-Fatigue Enforcement)

- Use bullets SPARINGLY: Only for distinct lists (product names, company roster, enumerated steps)
- NEVER use bullets as primary content delivery - they fragment thinking
- Each findings section requires substantive prose paragraphs (3-5+ paragraphs minimum)
- Example: Instead of "* Market size: $2.4B" write "The global market reached $2.4 billion in 2023, driven by increasing consumer demand and regulatory tailwinds [1]."

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

## Source Attribution Standards (Critical for Preventing Fabrication)

- **Immediate citation**: Every factual claim followed by [N] citation in same sentence
- **Quote sources directly**: Use "According to [1]..." or "[1] reports..." for factual statements
- **Distinguish fact from synthesis**:
  - GOOD: "Mortality decreased 23% (p<0.01) in the treatment group [1]."
  - BAD: "Studies show mortality improved significantly."
- **No vague attributions**:
  - NEVER: "Research suggests...", "Studies show...", "Experts believe..."
  - ALWAYS: "Smith et al. (2024) found..." [1], "According to FDA data..." [2]
- **Label speculation explicitly**:
  - GOOD: "This suggests a potential mechanism..." (analysis, not fact)
  - BAD: "The mechanism is..." (presented as fact without citation)
- **Admit uncertainty**:
  - GOOD: "No sources found addressing X directly."
  - BAD: Fabricating a citation to fill the gap
- **Template pattern**: "[Specific claim with numbers/data] [Citation]. [Analysis/implication]."

---

## Length Requirements

- Quick mode: 2,000+ words (baseline quality threshold)
- Standard mode: 4,000+ words (comprehensive analysis)
- Deep mode: 6,000+ words (thorough investigation)
- UltraDeep mode: 10,000-50,000+ words (NO UPPER LIMIT)

### Output Token Limit Safeguard (Claude Code Default: 32K)

Claude Code default limit: 32,000 output tokens (approximately 24,000 words total per skill execution). This is a HARD LIMIT.

**Realistic report sizes per mode:**
- Quick mode: 2,000-4,000 words (well under limit)
- Standard mode: 4,000-8,000 words (comfortably under limit)
- Deep mode: 8,000-15,000 words (achievable with care)
- UltraDeep mode: 15,000-20,000 words (at limit, monitor closely)

**For reports >20,000 words:** User must run skill multiple times or use auto-continuation.

---

## File Organization

### 1. Create Organized Folder in Documents

- ALWAYS create dedicated folder: `~/Documents/[TopicName]_Research_[YYYYMMDD]/`
- Extract clean topic name from research question (remove special chars, use underscores/CamelCase)
- Examples:
  - "psilocybin research 2025" -> `~/Documents/Psilocybin_Research_20251104/`
  - "compare React vs Vue" -> `~/Documents/React_vs_Vue_Research_20251104/`

### 2. Save All Formats to Same Folder

**Markdown (Primary Source):**
- Save to: `[Documents folder]/research_report_[YYYYMMDD]_[topic_slug].md`
- Also save copy to: `~/.claude/research_output/` (internal tracking)

**HTML (McKinsey Style - ALWAYS GENERATE):**
- Save to: `[Documents folder]/research_report_[YYYYMMDD]_[topic_slug].html`
- Use McKinsey template: [mckinsey_template](../templates/mckinsey_report_template.html)
- Design principles: Sharp corners (NO border-radius), muted corporate colors (navy #003d5c, gray #f8f9fa), ultra-compact layout, info-first structure
- Place critical metrics dashboard at top (extract 3-4 key quantitative findings)
- Use data tables for dense information presentation
- 14px base font, compact spacing, no decorative gradients or colors
- **Attribution Gradients (2025):** Wrap each citation [N] in `<span class="citation">` with nested tooltip div showing source details
- OPEN in browser automatically after generation

**PDF (Professional Print - ALWAYS GENERATE):**
- Save to: `[Documents folder]/research_report_[YYYYMMDD]_[topic_slug].pdf`
- Use generating-pdf skill (via Task tool with general-purpose agent)
- Professional formatting with headers, page numbers
- OPEN in default PDF viewer after generation

### 3. File Naming Convention

All files use same base name:
- `research_report_20251104_psilocybin_2025.md`
- `research_report_20251104_psilocybin_2025.html`
- `research_report_20251104_psilocybin_2025.pdf`

---

## Delivery Checklist

1. Executive summary (inline in chat)
2. Organized folder path (e.g., "All files saved to: ~/Documents/Psilocybin_Research_20251104/")
3. Confirmation of all three formats generated:
   - Markdown (source)
   - HTML (McKinsey-style, opened in browser)
   - PDF (professional print, opened in viewer)
4. Source quality assessment summary (source count)
5. Next steps (if relevant)

---

## HTML Generation Workflow

1. Read McKinsey template from `../templates/mckinsey_report_template.html`
2. Extract 3-4 key quantitative metrics from findings for dashboard
3. **Use Python script for MD to HTML conversion:**

   ```bash
   cd ~/.claude/skills/deep-research
   python scripts/md_to_html.py [markdown_report_path]
   ```

   The script returns two parts:
   - **Part A ({{CONTENT}}):** All sections except Bibliography, properly converted to HTML
   - **Part B ({{BIBLIOGRAPHY}}):** Bibliography section only, formatted as HTML

   **CRITICAL:** The script handles ALL conversion automatically:
   - Headers: ## -> `<div class="section"><h2 class="section-title">`, ### -> `<h3 class="subsection-title">`
   - Lists: Markdown bullets -> `<ul><li>` with proper nesting
   - Tables: Markdown tables -> `<table>` with thead/tbody
   - Paragraphs: Text wrapped in `<p>` tags
   - Bold/italic: **text** -> `<strong>`, *text* -> `<em>`
   - Citations: [N] preserved for tooltip conversion in step 4

4. **Add Citation Tooltips (Attribution Gradients) - optional:**
   ```html
   <span class="citation">[N]
     <span class="citation-tooltip">
       <div class="tooltip-title">[Source Title]</div>
       <div class="tooltip-source">[Author/Publisher]</div>
       <div class="tooltip-claim">
         <div class="tooltip-claim-label">Supports Claim:</div>
         [Extract sentence with this citation]
       </div>
     </span>
   </span>
   ```

5. Replace placeholders in template:
   - {{TITLE}} - Report title
   - {{DATE}} - Generation date (YYYY-MM-DD)
   - {{SOURCE_COUNT}} - Number of unique sources
   - {{METRICS_DASHBOARD}} - Metrics HTML from step 2
   - {{CONTENT}} - HTML from Part A
   - {{BIBLIOGRAPHY}} - HTML from Part B

6. **CRITICAL: NO EMOJIS** - Remove any emoji characters from final HTML
7. Save to: `[folder]/research_report_[YYYYMMDD]_[slug].html`
8. **Verify HTML (MANDATORY):**
   ```bash
   python scripts/verify_html.py --html [html_path] --md [md_path]
   ```
9. Open in browser: `open [html_path]`

---

## PDF Generation

1. Use Task tool with general-purpose agent
2. Invoke generating-pdf skill with markdown as input
3. Save to: `[folder]/research_report_[YYYYMMDD]_[slug].pdf`
4. PDF will auto-open when complete

---

## Progressive File Assembly (Unlimited Length)

### Phase 8.1: Setup
```bash
# Create folder: ~/Documents/[TopicName]_Research_[YYYYMMDD]/
mkdir -p ~/Documents/[folder_name]
# Create initial markdown file with frontmatter
```

### Phase 8.2: Progressive Section Generation

**CRITICAL STRATEGY:** Generate and write each section individually to file using Write/Edit tools. This allows unlimited report length while keeping each generation manageable.

**Initialize Citation Tracking:**
```
citations_used = []  # Maintain this list in working memory throughout
```

**Section Generation Loop:**

**Pattern:** Generate section content -> Use Write/Edit tool with that content -> Move to next section. Each Write/Edit call contains ONE section (<=2,000 words per call).

1. **Executive Summary** (200-400 words) -> Write(file, content=frontmatter + Executive Summary)
2. **Introduction** (400-800 words) -> Edit(file, append Introduction)
3. **Finding N** (600-2,000 words each) -> Edit(file, append Finding N) -- repeat for ALL findings
4. **Synthesis & Insights** -> Edit(file, append)
5. **Limitations & Caveats** -> Edit(file, append)
6. **Recommendations** -> Edit(file, append)
7. **Bibliography (CRITICAL - ALL Citations)** -> Edit(file, append) -- NO ranges, NO placeholders, NO truncation
8. **Methodology Appendix** -> Edit(file, append)

### Phase 8.3: Auto-Continuation Decision Point

**If total output <=18,000 words:** Complete normally.

**If total output will exceed 18,000 words:** Auto-Continuation Protocol:

**Step 1: Save Continuation State**

Create file: `~/.claude/research_output/continuation_state_[report_id].json`

```json
{
  "version": "2.1.1",
  "report_id": "[unique_id]",
  "file_path": "[absolute_path_to_report.md]",
  "mode": "[quick|standard|deep|ultradeep]",
  "progress": {
    "sections_completed": [],
    "total_planned_sections": 0,
    "word_count_so_far": 0,
    "continuation_count": 1
  },
  "citations": {
    "used": [],
    "next_number": 1,
    "bibliography_entries": []
  },
  "research_context": {
    "research_question": "",
    "key_themes": [],
    "main_findings_summary": [],
    "narrative_arc": ""
  },
  "quality_metrics": {
    "avg_words_per_finding": 0,
    "citation_density": 0,
    "prose_vs_bullets_ratio": "",
    "writing_style": "technical-precise-data-driven"
  },
  "next_sections": []
}
```

**Step 2: Spawn Continuation Agent** via Task tool with general-purpose agent. The continuation agent reads state, generates next batch, and spawns the next agent if needed. Chain continues recursively until complete.

**Step 3: Report Continuation Status** to user with progress percentage.

### Phase 8.4: Continuation Agent Quality Protocol

**Context Loading (CRITICAL):**
1. Read continuation_state.json -> Load ALL context
2. Read existing report file -> Review last 3 sections
3. Extract patterns: sentence structure, terminology, citation placement, transition style

**Pre-Generation Checklist:**
- [ ] Loaded research context (themes, question, narrative arc)
- [ ] Reviewed previous sections for flow
- [ ] Loaded citation numbering (start from N+1)
- [ ] Loaded quality targets (words, density, style)
- [ ] Understand where in narrative arc (beginning/middle/end)

**Per-Section Generation:**
1. Generate section content
2. Quality checks: word count, citation density, prose ratio, theme connection, style match
3. If ANY check fails: Regenerate section
4. If passes: Write to file, update state

**Final Agent Responsibilities:**
- Generate final content sections
- Generate COMPLETE bibliography using ALL citations from state.citations.bibliography_entries
- Read entire assembled report
- Run validation: `python scripts/validate_report.py --report [path]`
- Delete continuation_state.json (cleanup)
- Report complete to user with metrics
