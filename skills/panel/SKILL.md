---
name: panel
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

<!-- This is a copy of discussion/SKILL.md with name: panel. Always edit discussion/SKILL.md first, then copy here. -->

# Discussion — Multi-Perspective Analysis Panel

## Why this skill exists

When you work 1-on-1 with an AI, a subtle but dangerous pattern emerges:

1. **Confirmation drift** — the AI adapts to your framing and stops pushing back
2. **Context tunnel vision** — both sides see only what's already been discussed
3. **Band-aid fixes** — surface-level patches accumulate instead of addressing root causes
4. **Unchallenged premises** — wrong assumptions survive because nobody re-examines them

This skill breaks that pattern by spawning fresh sub-agents who have no stake in the
current conversation's conclusions. They analyze the topic independently, then you and
the main agent review their findings together.

Be honest about what this is: **structured self-review from the same underlying model,
not truly independent experts.** It works because fresh context removes conversational
inertia and structured roles force examination of angles the main conversation missed.

## Invocation

```
/discussion [topic]                Standard (2 panelists)
/discussion full [topic]           Full panel (4 panelists)
/discussion max [topic]      MAX panel (5 panelists)
/discussion [topic] --ctx          Force codebase exploration (default ON for technical topics)
/discussion [topic] --no-ctx       Explicitly disable codebase exploration
```

`--ctx` default: ON for technical topics (code, architecture, bugs), OFF for non-technical.
User can override with `--ctx` / `--no-ctx`.

If invoked **without a mode specifier**, assess the topic's weight:

#### Light topics (no architectural impact, reversible in minutes, single-file scope)
No questions needed. Just run Standard + Sonnet silently.

#### Medium / Heavy topics
Present both questions together using AskUserQuestion. Set the recommended option
based on topic weight (Medium → Full + Balanced, Heavy → Max + All Opus).

```json
{
  "questions": [
    {
      "question": "パネルの規模を選んでください。",
      "header": "Scale",
      "multiSelect": false,
      "options": [
        {
          "label": "Standard",
          "description": "2視点 (Critic, Realist)。軽い確認向き"
        },
        {
          "label": "Full (Recommended)",
          "description": "4視点 (+ Architect, Outsider)。設計判断に最適"
        },
        {
          "label": "Max",
          "description": "5視点 (+ Contrarian)。重大な意思決定向き"
        }
      ]
    },
    {
      "question": "パネリストのモデルを選んでください。",
      "header": "Model",
      "multiSelect": false,
      "options": [
        {
          "label": "All Sonnet",
          "description": "全員 Sonnet。高速・低コスト"
        },
        {
          "label": "Balanced (Recommended)",
          "description": "重要パネリストが Opus、他は Sonnet (コスパ最良)"
        },
        {
          "label": "All Opus",
          "description": "全員 Opus。最高精度だがコスト大"
        }
      ]
    }
  ]
}
```

Adjust the "(Recommended)" label based on topic weight:
- **Medium**: Full + Balanced recommended
- **Heavy**: Max + All Opus recommended

#### Balanced model assignment

Applies only to panelists in the current mode (Standard uses Critic + Realist only).

| Panelist | Balanced mode model | Why |
|----------|-------------------|-----|
| **Critic** | Opus | Deep assumption-challenging needs stronger reasoning |
| **Architect** | Opus | Structural/systemic analysis benefits from depth |
| **Contrarian** | Opus | Building a coherent counter-argument is the hardest task |
| **Realist** | Sonnet | Practical trade-off evaluation works well with Sonnet |
| **Outsider** | Sonnet | Fresh-eyes simplicity check doesn't need Opus |

If the user explicitly specifies mode and/or model, skip the questions.
If the user says "just do it" or seems impatient, use the recommended defaults silently.

## Panelist Roster

Each panelist has a cognitive framework and a Starting Artifact — a mandatory thinking
exercise they must complete before producing findings. This forces genuine divergence
from the same underlying model.

### Standard mode — 2 panelists:

| Role | Focus | Cognitive Framework | Starting Artifact |
|------|-------|---------------------|-------------------|
| **Critic** | Flawed assumptions, missing risks, alternative framings, what could go wrong | Pre-mortem + 5 Whys | Write 3 failure scenarios first. Use those as the entry point for analysis. |
| **Realist** | Implementation cost, maintenance burden, simpler alternatives, "is this worth it?" | Cost-benefit with concrete estimation | Estimate implementation cost in person-days first, broken down by phase. |

### Full mode — 4 panelists (adds):

| Role | Focus | Cognitive Framework | Starting Artifact |
|------|-------|---------------------|-------------------|
| **Architect** | Root causes, systemic effects, long-term consequences, design coherence, overlooked synergies and reuse opportunities | First Principles decomposition | Draw the dependency chain first. Map what depends on what before forming any opinion. |
| **Outsider** | Unnecessary complexity, confusing naming, "why not just...?" questions | Beginner's Mind + Cross-domain analogy | Name one example from outside software that solved this same class of problem. Start from that. |

### Max mode — 5 panelists (adds):

| Role | Focus | Cognitive Framework | Starting Artifact |
|------|-------|---------------------|-------------------|
| **Contrarian** | Constructs the strongest possible case for the OPPOSITE of the current approach — not just finding flaws (that's Critic's job), but building a coherent alternative direction | Steel-man inversion | Imagine a world where the current approach was never proposed. Describe what was built instead and why it was considered obviously correct. |

The Contrarian is the most expensive but often the most valuable panelist — it forces
genuine consideration of roads not taken, not just critique of the road chosen.

## Execution Flow

### Step 1: Context Extraction

Before spawning panelists, distill the topic into a structured brief.
The brief has four fact categories plus user argument and prior attempts.
Extract each category separately — panelists will receive different views of this data.

```
Topic:            The specific question or decision (1-2 sentences)
Stakes:           What happens if we get this wrong?

Technical constraints:
  - [Hard technical limits, performance requirements, platform constraints]
  - [Concrete numbers: line counts, function counts, dependency counts, etc.]
  - [Actual failure incidents or known bugs if relevant]

Business constraints:
  - [Deadlines, budget, team size, compliance requirements]

User behavior:
  - [How end users actually interact with the system; observed patterns]

Implicit assumptions:
  - [Things being treated as true without explicit justification]

User's argument:
  - [The user's stated position and the reasoning behind it]
  - Include this even if it seems obvious — it is the primary attack target for Critic and Contrarian.

Prior attempts:
  - Include: "We tried X and observed Y" (factual outcomes)
  - Exclude: "X is bad / X won't work" (conclusions that would bias panelists against revisiting X)
```

**If the topic is too short or vague**, ask one clarifying question before proceeding.
A one-line topic without facts/constraints produces abstract, unhelpful analysis.

### Step 1.5: Information Distribution

Each panelist sees a different view of the brief. Only construct views for
panelists in the current mode (Standard = Critic + Realist only).

| Panelist | Receives |
|----------|----------|
| **Critic** | All categories + implicit assumptions prominently at top. Also add: "List 3 questions this brief does NOT address — what's conspicuously absent?" |
| **Realist** | All categories + technical and business constraints highlighted |
| **Architect** | All categories + technical constraints + dependency/structural information highlighted |
| **Outsider** | Topic and stakes ONLY — no constraints, no user argument, no prior attempts (intentional blank slate) |
| **Contrarian** | All categories + user's argument placed prominently at top |

**Outsider information hygiene**: Strip implementation-specific terms (function names,
file paths, library names) from the Outsider's Stakes description. Write Stakes in
business/user-impact language only. For non-technical topics, omit Stakes entirely
and give Outsider the Topic sentence only.

#### Step 1.5b: Dynamic Artifact Parameterization

Before spawning panelists, inject topic-specific context from Step 1 into each
role's Starting Artifact instruction (from the Panelist Roster tables above). Append the relevant element: Critic gets a specific
implicit assumption to attack, Realist gets the specific approach to estimate,
Architect gets the key component to map from, Contrarian gets the user's stated approach
to invert. Outsider receives no injection (preserve blank-slate).

Fallback: if multiple candidates exist, prefer the most specific. Defaults:
Critic → first implicit assumption, Realist → the approach's costliest phase,
Architect → the component with most dependencies, Contrarian → user's core claim.

### Step 2: Spawn Panelists

Launch all panelists **in parallel** using the Agent tool in a single message.
Set the `model` parameter on each Agent call according to the confirmed configuration.

Use the panelist prompt template below for each, substituting `[ROLE]`, `[FRAMEWORK]`,
`[ARTIFACT_INSTRUCTION]`, and `[BRIEF_VIEW]` from the roster and Step 1.5.

#### Panelist Prompt Template

```
You are the [ROLE] on a review panel. You have NO prior context about this project
beyond what is provided below — this is intentional, to avoid inheriting biases.

## Your perspective: [ROLE]
[Focus sentence from the roster table]

## Cognitive framework: [FRAMEWORK]

## Starting Artifact (mandatory — do not skip; findings must emerge from this exercise)
[ARTIFACT_INSTRUCTION]
Then: identify one non-obvious implication from what you just wrote.

## Your information view
[BRIEF_VIEW from Step 1.5 — the panelist-specific subset]

## Output format
1. **Starting Artifact** — complete the exercise above, written out in full
2. **Reasoning chain** — your most important finding developed in 150-200 words.
   Must emerge from your Starting Artifact. Begin with "From [name a specific element —
   e.g., failure scenario 2, the hidden dependency you mapped, the Phase 3 cost estimate]..."
   to anchor reasoning in the exercise, not general pattern-matching.
3. **Findings** — 1 to 3 findings only, each with a severity label:
   CRITICAL = blocks progress or causes failure if ignored
   HIGH     = significant risk or missed opportunity
   MEDIUM   = worth considering but not urgent
   LOW      = minor improvement or nitpick
   Format: **[SEVERITY]** Finding text (2-4 sentences, specific and concrete)

Rules:
- "This might cause problems" is useless. "This breaks when X because Y" is useful.
- If you need to see code to give a real answer, say so rather than speculating.
- If the current approach is genuinely good, say so — then name what still warrants watching.
- At one point in your reasoning, name what would have to be true for your
  conclusion to be wrong — then decide if it changes anything.
- Write in the same language as the topic.
```

**When `--ctx` is active**, add Read/Grep/Glob tools to each panelist and append
the following block, with the role-specific exploration focus substituted in:

```
You have access to the codebase. Explore before you analyze.

Your exploration focus for [ROLE]:
  Critic:      Test files, error handling, input validation
               Ask: "Where are exceptions silenced? What inputs go unvalidated?"
  Realist:     Dependency files (package.json etc.), build config, CI config
               Ask: "What dependencies are unused? Where is build complexity hiding?"
  Architect:   Data models, schemas, inter-module dependency graph
               Ask: "Where do modules bypass the intended architecture? What's coupled that shouldn't be?"
  Outsider:    README, documentation, public API surface
               Ask: "Where does the docs promise something the code doesn't deliver?"
  Contrarian:  Oldest files, legacy code, TODO/FIXME comments
               Ask: "What old decisions still constrain us? Which TODOs reveal abandoned better paths?"
```

With `--ctx`, panelists may reference different files. This is a feature (broader
coverage), not a bug, but keep it in mind when synthesizing results.

### Step 3: Present Results

**Summary comes FIRST** — most users read only this.

```markdown
---

## Discussion Panel: [Topic in ~10 words]
*Mode: [standard/full/max] | Panelists: [N]*

### Summary
- **Consensus**: [Points all panelists agreed on]
- **Tensions**: [Where panelists disagreed — state both sides. "None" if unanimous]
- **Discoveries**: [New angles nobody in the original conversation raised]

### Reasoning Highlight
> [The single most developed reasoning chain from Step 2, quoted verbatim or lightly edited for clarity. Attribute to panelist.]

### Findings

| # | Severity | Finding | Panelist |
|---|----------|---------|----------|
| 1 | CRITICAL | [most severe finding] | [role] |
| 2 | HIGH     | [next finding] | [role] |
| 3 | MEDIUM   | [finding] | [role] |
| ... | ...   | ... | ... |

Sort rows by severity (CRITICAL > HIGH > MEDIUM > LOW).
Within the same severity: Critic → Realist → Architect → Outsider → Contrarian.

### Collision Analysis
[Results from Step 3.5 sub-agent — omit this section for Standard mode]

---

*Panel complete. What resonates? Want to dig deeper into any point?*
```

For **Discoveries**: only include genuinely new insights. If panelists simply
restated known concerns, write "None — panelists reinforced existing concerns
rather than surfacing new angles." Honest framing over padding.

### Step 3.5: Collision Analysis

**Full and Max modes only.** For Standard mode (2 panelists), skip the dedicated
sub-agent — briefly note any tension between the two reasoning chains inline.

For Full/Max: spawn a **dedicated sub-agent** (fresh context, no conversation history).
Pass it ONLY the Reasoning Chains from each panelist — not their Findings.
This prevents "solving the mystery after knowing the answer."

Agent prompt:
```
You are the Collision Analyst. You receive only the reasoning chains from a review panel.
You have NOT seen their conclusions.

## Topic under review
[Topic from Step 1]

## Reasoning chains
[Insert each panelist's Reasoning Chain, labeled by role — e.g., **Critic:** ...]

Note: Each panelist's reasoning may contain an internal self-challenge
(a "what if I'm wrong" moment). Do not treat these as contradictions
between panelists — they are within-panelist dialectic, not between-panelist conflict.

## Your task
1. Identify contradictions or tensions between the reasoning chains.
2. For each: "If both lines of reasoning are correct, the third conclusion is: [Z]"
3. If no genuine contradictions exist, say so honestly.

Format: **[Role A] vs [Role B]**: [contradiction]. Third conclusion: [Z].
Output 1-3 collision results. Do not manufacture collisions.
```

Include the Collision Analysis results in the Step 3 output after the Findings table.

### Step 4: Facilitate

After presenting results:

- Do NOT immediately adopt the panelists' suggestions
- Do NOT dismiss them either
- Ask the user which points resonate and which don't
- If the user wants to explore a point, discuss it or spawn a focused follow-up

The panel informs; it doesn't dictate. The USER decides.

### Step 5: Implementation Pipeline (optional)

If the user wants to act on findings (typically ≥2 accepted findings), offer a
two-agent workflow. Note: this adds significant token cost (Opus generation + review).

1. **Generator** (Opus — needs deepest reasoning for faithful implementation) —
   accepted findings as numbered checklist + file paths. Implement all, add nothing extra.
2. **Reviewer** — same checklist. Verify each item: PASS/FAIL with line references.
3. Present review results. Fix any FAILs before closing.

## When NOT to use

- Simple factual questions with clear answers
- The user explicitly wants execution, not deliberation
- Rapid iteration loops where speed matters more than reflection
- Trivial decisions where the cost of being wrong is negligible

## Cost awareness

| Mode | Panelists | Default model | Approx. token multiplier |
|------|-----------|---------------|-------------------------|
| Standard | 2 (Critic, Realist) | sonnet | ~3x (~4x with Balanced) |
| Full | 4 (+Architect, Outsider) | sonnet (opus recommended) | ~5-6x (~7x with Balanced) |
| Max | 5 (+Contrarian) | opus (Balanced = default) | ~7-8x |

With `--ctx` active (default for technical topics), add ~1.5-2x to the above multipliers
due to codebase exploration. Use `--no-ctx` to suppress this on technical topics where
exploration is not needed.

Max with Opus is the most expensive configuration but also the most thorough.
Reserve it for decisions where the cost of a wrong choice far exceeds the analysis cost.

**Model selection rationale (from empirical testing):**
- **Haiku**: Not recommended. Explores codebase inefficiently, consuming 4x more
  tokens than Sonnet for comparable or lower quality output.
- **Sonnet**: Best cost/quality ratio. Default for Standard and Full modes.
- **Opus**: Deepest analysis. Recommended for Full on important decisions, mandatory
  default for Max. The user can always override.
