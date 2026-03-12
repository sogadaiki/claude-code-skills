# Error Handling & Stop Rules

## Stop Immediately If

- 2 validation failures on same error -> Pause, report, ask user
- <5 sources after exhaustive search -> Report limitation, request direction
- User interrupts/changes scope -> Confirm new direction

---

## Graceful Degradation

- 5-10 sources -> Note in limitations, proceed with extra verification
- Time constraint reached -> Package partial results, document gaps
- High-priority critique issue -> Address immediately

---

## Error Reporting Format

```
Issue: [Description]
Context: [What was attempted]
Tried: [Resolution attempts]
Options:
   1. [Option 1]
   2. [Option 2]
   3. [Option 3]
```

---

## Validation Failure Protocol

1. **First failure**: Auto-fix formatting/links, re-run validation
2. **Second failure**: Manual review + correction, re-run validation
3. **Third failure**: STOP execution, report all issues to user, ask for direction

---

## Common Error Patterns

### Insufficient Sources
- **Cause**: Niche topic, restricted domain, paywall-heavy area
- **Action**: Note in limitations section, increase search diversity, try alternative query formulations
- **Minimum**: If <5 sources after exhaustive search, STOP and report

### Citation Mismatches
- **Cause**: Bibliography entries don't match in-text citations
- **Action**: Run `verify_citations.py`, reconcile mismatches, remove fabricated entries

### Output Token Limit Reached
- **Cause**: Report exceeds 18,000 words in single generation
- **Action**: Trigger auto-continuation protocol (see output-contract.md)

### Search Tool Errors
- **Cause**: WebSearch/Exa API failures, rate limits
- **Action**: Retry with alternative query, switch search tool, proceed with available sources
