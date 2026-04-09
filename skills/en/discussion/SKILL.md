---
name: discussion
description: >
  Multi-perspective analysis panel that spawns independent sub-agents to challenge assumptions,
  find blind spots, and surface alternatives. Use this skill whenever the user says /discussion,
  /panel, or /challenge. Also trigger when the user asks for a "second opinion", wants to
  "stress-test" an idea, says "is this really the right approach?", debates trade-offs before
  a design decision, feels stuck in a direction, or when the conversation has been agreeing
  on everything for too long without questioning assumptions. Covers: architecture decisions,
  refactoring judgments, priority calls, tool/library selection, UX direction, and any
  "should we X or Y?" dilemma — both technical and non-technical.
---

<!-- English version. Canonical source: en/discussion/SKILL.md. Copy to en/panel/SKILL.md (change name: to panel). -->

# Discussion — Multi-Perspective Analysis Panel

## Why this skill exists

When you work 1-on-1 with an AI, a subtle but dangerous pattern emerges:

1. **Confirmation drift** — the AI adapts to your framing and stops pushing back
2. **Context tunnel vision** — both sides see only what's already been discussed
3. **Band-aid fixes** — surface-level patches accumulate instead of addressing root causes
4. **Unchallenged premises** — wrong assumptions survive because nobody re-examines them

This skill breaks the pattern by spawning fresh sub-agents with no stake in the current
conversation's conclusions. Be honest about what this is: **structured self-review from the
same underlying model, not truly independent experts.** It works because fresh context removes
conversational inertia, and structured roles force examination of angles the main conversation missed.

## Invocation

```
/discussion [topic]                 Standard (2 panelists)
/discussion full [topic]            Full panel (4 panelists)
/discussion max [topic]             MAX panel (5 panelists)
/discussion [topic] --independent   Give panelists Read/Grep tools (former --ctx; usually discouraged)
```

**Key design change**: This skill abolishes the old "each panelist explores with Read/Grep
independently" approach. By default, the orchestrator explores ONCE and builds a
**Shared Context Pack** that is distributed to all panelists. Benefits:
- 5-way duplicated exploration collapses to 1 pass (60-80% token reduction)
- All panelists see the same code, so Collision Analysis produces clean "viewpoint clashes"
- Fewer factual errors, lighter Findings Validation

`--independent` restores the old per-panelist exploration mode, but Shared Context is usually sufficient.

## Mode selection

When invoked without an explicit mode, evaluate topic weight:

- **Lightweight topic** (no architectural impact, easy rollback, single file): Run Standard + Sonnet silently
- **Medium / Heavy**: Use AskUserQuestion to confirm scale AND model in a single prompt

Recommend Full + Balanced for medium, Max + Balanced for heavy topics.

**Important**: Max mode defaults to **Balanced** (NOT All Opus). All Opus is an explicit opt-in.
Realist and Outsider have been empirically shown to produce sufficient quality on Sonnet —
Opus is overkill for them.

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

Skip the prompt and use defaults (Balanced) if the user explicitly specified mode/model,
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

Each panelist has a cognitive framework and a Starting Artifact. Starting Artifacts are
**executed internally as thinking scaffolds** (they are NOT emitted in output). This produces
diversity from a single base model.

| Role | Focus | Framework | Starting Artifact (internal) |
|------|-------|-----------|-----------------------------|
| **Critic** | Flawed assumptions, missed risks, what could break | Pre-mortem + 5 Whys | Imagine 3 failure scenarios |
| **Realist** | Implementation cost, maintenance burden, simpler alternatives | Concrete estimates | Estimate per-phase person-days |
| **Architect** | Root causes, systemic impact, long-term consequences | First-principles decomposition | Trace dependency chains |
| **Outsider** | Unnecessary complexity, unclear naming, beginner perspective | Beginner eyes + cross-domain analogy | Find a parallel in a non-software field |
| **Contrarian** | Strongest argument for the exact opposite approach | Steelman inversion | Imagine a world where the current approach was never proposed |

Standard = Critic + Realist / Full = + Architect, Outsider / Max = + Contrarian

## Execution flow

### Step 1: Brief extraction

Condense the topic into a structured brief. See `references/brief-format.md` for the full
category list and format. Key points:
- Topic (1-2 sentences) + Stakes
- Extract only the categories relevant to the topic (focus on technical constraints for
  technical topics; include business categories only when they matter)
- Always include user's position and past attempts (primary attack surface for Critic/Contrarian)

If the topic is vague, ask ONE clarifying question before proceeding.

### Step 2: Shared Context Pack (technical topics only)

The orchestrator uses Read/Grep/Glob ONCE to collect code material relevant to the topic.
See `references/context-pack.md` for details. Key points:
- Gather raw snippets with file paths and line numbers
- Add no interpretation or evaluation (raw material only)
- Target 3-5k tokens total
- Skip entirely for non-technical topics

The Pack is distributed to all panelists EXCEPT the Outsider. **The Outsider stays on a
deliberate blank slate — no Pack is given.**

### Step 3: Information distribution and Artifact parameterization

Each panelist receives a different view of the brief. See
`references/information-distribution.md` for the full distribution rules and dynamic
Artifact parameterization. Key points:
- Critic: full brief + implicit assumptions highlighted
- Realist: full brief + tech/business constraints emphasized
- Architect: full brief + dependency info
- Outsider: topic and stakes only (blank slate; no Pack)
- Contrarian: full brief + user's position highlighted

### Step 4: Parallel panelist launch

Launch all panelists in parallel via the Agent tool (multiple Agent calls in ONE message).
Set the `model` parameter on each Agent call.

**Panelists receive NO tools** — no Read/Grep/Glob. They analyze using only the Pack and
brief. Only under `--independent` are Read/Grep/Glob passed.

See `references/panelist-prompt.md` for the full prompt template. Key points:
- Starting Artifact is **executed internally** (scaffold only, not emitted)
- Output is 1-2 Findings, each with severity label and 2-3 sentence rationale
- Reasoning chain output is abolished (folded into the Findings rationale)
- CRITICAL / HIGH / MEDIUM / LOW severity labels are required

### Step 5: Collision Analysis (Full / Max only)

Skipped in Standard mode (just compare the two Findings inline).

For Full/Max: launch a dedicated sub-agent (no conversation history). Pass each panelist's
Findings along with a 2-3 line summary of each panelist's Starting Artifact extracted by the
orchestrator. See `references/collision-analyst.md` for the full prompt.

The Collision Analyst's job:
1. Identify contradictions between panelists → propose a third conclusion
2. Check for shared blind spots in consensus → build a dissenter's argument

### Step 6: Findings Validation (Full / Max only)

The orchestrator (not a sub-agent) validates each finding. With Shared Context Pack in place,
factual errors should be greatly reduced, but still confirm:
1. Does the finding correctly reference actual design/code?
2. Is it already handled (a design element the panelist wasn't aware of)?
3. Does it contradict something the panel itself said?

Ratings: ◎ (accurate) / ○ (valid but limited) / △ (partially correct) / ✕ (factual error)

### Step 7: Result presentation

**Summary first** — most users only read this part.
See `references/output-format.md` for the format. Key points:
- Summary: Agreement / Tension / Discovery
- Findings table (sorted by severity)
- Collision Analysis (Full/Max only)
- Findings Validation (Full/Max only)

On **Discovery**: include only truly new insights. If panelists merely reinforced known
concerns, write honestly: "None — the panel reinforced existing concerns rather than
surfacing new angles." Honest framing over padding.

### Step 8: Facilitation

- Do NOT immediately adopt or reject suggestions
- Ask what resonated
- Dig deeper on any point, or launch a focused follow-up panel

The panel informs. It does not instruct. The user decides.

## When NOT to use

- Simple factual questions with a clear answer
- User wants action, not debate
- Rapid iteration loops where speed matters more than rigor
- Trivial decisions where being wrong costs nothing

## Cost awareness

| Mode | Panelists | Default model | Approx. token multiplier |
|------|-----------|---------------|--------------------------|
| Standard | 2 | Sonnet | ~2-3× |
| Full | 4 | Balanced | ~3-4× |
| Max | 5 | Balanced | ~4-5× |

The old per-panelist exploration mode added ~1.5-2× on top. Shared Context Pack eliminates
that duplication. Only `--independent` restores the old cost profile.

All Opus is about 1.5× Balanced. Use only for critical decisions where the cost of being
wrong vastly exceeds the analysis cost.

**Model rationale**:
- **Sonnet**: Best cost-to-quality ratio. Default for Standard and for Realist/Outsider in Full
- **Opus**: Needed for Critic/Architect/Contrarian where deep reasoning and coherent counter-arguments matter
- **Haiku**: Not recommended. Exploration is inefficient and total tokens end up ~4× Sonnet for equivalent quality

## Reference files

Detailed guidance lives in `references/`. Read these on demand:
- `references/brief-format.md` — brief extraction categories and format
- `references/context-pack.md` — Shared Context Pack creation procedure
- `references/information-distribution.md` — per-panelist information distribution and Artifact parameterization
- `references/panelist-prompt.md` — full panelist prompt template
- `references/collision-analyst.md` — full Collision Analyst prompt
- `references/output-format.md` — result presentation format
