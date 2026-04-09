---
name: panel
description: >
  Multi-perspective analysis panel that spawns independent sub-agents to challenge assumptions,
  find blind spots, and surface alternatives. Triggers when the user says /discussion, /panel,
  or /challenge, or asks for a second opinion, wants to stress-test an idea, says "is this
  really the right approach?", debates trade-offs before a design decision, feels stuck, or
  has been agreeing on everything for too long without questioning assumptions.
---

# Discussion — Multi-Perspective Analysis Panel

Fresh sub-agents cut through conversational inertia and confirmation bias. This is NOT truly
independent expertise — it is **structured self-review from the same base model**, which works
because fresh context + fixed roles force examination of angles the main conversation missed.

## Invocation

```
/discussion [topic]                 Standard (2 panelists)
/discussion full [topic]            Full panel (4 panelists)
/discussion max [topic]             MAX panel (5 panelists)
/discussion [topic] --independent   Legacy per-panelist exploration (pass Read/Grep to each panelist)
```

Default: the orchestrator explores ONCE and builds a **Shared Context Pack** that all
panelists share. `--independent` restores the old per-panelist mode, but is rarely needed.
Details in `references/context-pack.md`.

## Mode selection

When invoked without an explicit mode, evaluate topic weight:

- **Lightweight** (no architectural impact, easy rollback, single file): Run Standard + Sonnet silently
- **Medium / Heavy**: Use AskUserQuestion to confirm scale AND model (recommend Full + Balanced / Max + Balanced)

**Important**: Max defaults to **Balanced** (NOT All Opus). All Opus is explicit opt-in.
Realist and Outsider produce sufficient quality on Sonnet.

```json
{
  "questions": [
    {
      "question": "Choose panel scale.",
      "header": "Scale",
      "multiSelect": false,
      "options": [
        {"label": "Standard", "description": "2 perspectives (Critic, Realist)"},
        {"label": "Full", "description": "4 perspectives (+ Architect, Outsider)"},
        {"label": "Max", "description": "5 perspectives (+ Contrarian)"}
      ]
    },
    {
      "question": "Choose panelist model.",
      "header": "Model",
      "multiSelect": false,
      "options": [
        {"label": "All Sonnet", "description": "All panelists on Sonnet. Fast and cheap"},
        {"label": "Balanced (Recommended)", "description": "Critic/Architect/Contrarian=Opus, others=Sonnet"},
        {"label": "All Opus", "description": "All panelists on Opus. Highest accuracy but costly"}
      ]
    }
  ]
}
```

Skip the prompt and use the Balanced default if the user explicitly specified mode/model,
said "just do it", or is in a hurry.

### Balanced model assignment

| Panelist   | Model  | Reason |
|------------|--------|--------|
| Critic     | Opus   | Deep assumption challenge benefits from stronger reasoning |
| Architect  | Opus   | Systematic analysis rewards deeper insight |
| Contrarian | Opus   | Constructing a coherent counter-argument is the hardest task |
| Realist    | Sonnet | Practical trade-off evaluation works well on Sonnet |
| Outsider   | Sonnet | Beginner-mind analysis does not need Opus |

## Panelist roster

**Starting Artifacts** run as internal thinking scaffolds — they are NOT emitted in output.
This is what produces diversity from a single base model.
Composition: Standard = Critic + Realist / Full adds Architect + Outsider / Max adds Contrarian.

| Role | Focus | Framework | Starting Artifact (internal) |
|------|-------|-----------|-----------------------------|
| **Critic** | Flawed assumptions, missed risks, what could break | Pre-mortem + 5 Whys | Think through 3 failure scenarios |
| **Realist** | Implementation cost, maintenance burden, simpler alternatives | Concrete estimates | Estimate per-phase person-days |
| **Architect** | Root causes, systemic impact, long-term consequences | First-principles decomposition | Trace dependency chains |
| **Outsider** | Unnecessary complexity, unclear naming, beginner perspective | Beginner eyes + cross-domain analogy | Find a parallel in a non-software field |
| **Contrarian** | Strongest argument for the exact opposite approach | Steelman inversion | Imagine a world where the current approach was never proposed |

## Execution flow

### Step 1: Brief extraction

Condense the topic into a structured brief. Categories and format: `references/brief-format.md`.
If the topic is vague, ask ONE clarifying question before proceeding.

### Step 2: Shared Context Pack (technical topics only)

The orchestrator uses Read/Grep/Glob ONCE to collect code material (target 3-5k tokens).
Procedure and skip conditions: `references/context-pack.md`. The Pack is distributed to all
panelists EXCEPT the Outsider (who stays on a deliberate blank slate).

### Step 3: Information distribution and Artifact parameterization

Each panelist receives a different view of the brief. Distribution rules and dynamic
Artifact injection: `references/information-distribution.md`.

### Step 4: Parallel panelist launch

Launch all panelists in parallel via the Agent tool (multiple Agent calls in ONE message).
Set the `model` parameter on each Agent call. **Panelists receive NO tools** — they analyze
using only the Pack and brief (`--independent` is the exception — pass Read/Grep/Glob then).

Full prompt template: `references/panelist-prompt.md`. Output is **1-2 Findings only**,
each with a severity label (CRITICAL/HIGH/MEDIUM/LOW) and 2-3 sentence rationale. Reasoning
chain output is abolished.

### Step 5: Collision Analysis (Full / Max only)

Skipped in Standard — just compare the two Findings inline. For Full/Max: launch a dedicated
sub-agent (no conversation history) with the Findings plus each panelist's Artifact summary
(2-3 lines). It detects contradictions and consensus risks. Full prompt:
`references/collision-analyst.md`.

### Step 6: Findings Validation (Full / Max only)

This is NOT a sub-agent call — the orchestrator itself validates each Finding. Check:
(1) does it correctly reference actual design/code, (2) is it already handled, (3) does it
contradict something the panel itself said. Ratings: ◎ (accurate) / ○ (valid but limited) /
△ (partially correct) / ✕ (factual error).

### Step 7: Result presentation

**Summary first** — most users only read this part. Format: `references/output-format.md`.
**Discovery**: include only truly new insights. If panelists reinforced known concerns,
write "None" honestly. Honest framing over padding.

### Step 8: Facilitation

Do NOT immediately adopt or reject suggestions. Ask what resonated. Dig deeper or launch a
focused follow-up. The panel informs. The user decides.

## When NOT to use

- Simple factual questions with a clear answer
- User wants action, not debate
- Rapid iteration loops where speed matters more than rigor
- Trivial decisions where being wrong costs nothing

## Cost awareness

Baseline: a single Sonnet call = 1.0. Multipliers are approximate.

| Mode | Panelists | Default model | Token multiplier (approx.) |
|------|-----------|---------------|----------------------------|
| Standard | 2 | Sonnet | ~2-3× |
| Full | 4 | Balanced | ~3-4× |
| Max | 5 | Balanced | ~4-5× |

All Opus is about 1.5× Balanced. Use only for critical decisions where the cost of being
wrong vastly exceeds the analysis cost. Haiku is NOT recommended (inefficient exploration
ends up costing ~4× Sonnet for equivalent quality).
