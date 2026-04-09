# Output Format — Result Presentation Format

**Summary first** — most users only read this part.

## Template

```markdown
---

## Discussion Panel: [topic in ~10 words]
*Mode: [standard/full/max] | Panelists: [N] | Context: [shared/independent]*

### Summary
- **Agreement**: [what all panelists aligned on]
- **Tension**: [where panelists diverged — describe both sides. If unanimous: "None"]
- **Discovery**: [truly new angles not raised in the original conversation]

### Findings

| # | Severity | Finding | Rationale | Panelist |
|---|----------|---------|-----------|----------|
| 1 | CRITICAL | [most severe finding title] | [2-3 sentence rationale] | [role] |
| 2 | HIGH     | [next finding] | [rationale] | [role] |

### Collision Analysis
[Collision Analyst result — omit for Standard]

### Findings Validation
[Orchestrator's validation — omit for Standard]

| # | Finding | Rating | Reason |
|---|---------|--------|--------|
| 1 | [finding summary] | ◎/○/△/✕ | [1 sentence reason] |

---

*Panel complete. Which points resonated? Anything you want to dig deeper on?*
```

Severity label definitions (CRITICAL/HIGH/MEDIUM/LOW) live in `panelist-prompt.md`.

## Sort rules

Findings table rows:
- Sort by severity (CRITICAL > HIGH > MEDIUM > LOW)
- Within the same severity: Critic → Realist → Architect → Outsider → Contrarian (Standard mode order extended)

## Discovery honesty rule

Include only truly new insights under Discovery. If panelists merely reinforced
already-known concerns, write honestly:

> Discovery: None — the panel reinforced existing concerns rather than surfacing new angles.

**Prefer honest framing over padding.** It is not unusual for 50%+ of panel runs to come
back with "Discovery: None". That is not a failure — it is confirmation that the known
concerns were valid.

For unusually complex topics where you want to showcase especially deep reasoning, you MAY
add an optional "### Notable Logic" section after the Findings table. Usually the Rationale
column alone is sufficient.

## Facilitation message

Always end with "Panel complete. Which points resonated?" This is the explicit hand-off
to the user. Do not immediately adopt or reject any suggestion — show you are continuing
the dialogue.
