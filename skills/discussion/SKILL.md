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
/discussion extrafull [topic]      MAX panel (5 panelists)
/discussion [topic] --ctx          Allow codebase exploration
```

**Mode can also be combined:** `/discussion full [topic] --ctx`

If invoked **without a mode specifier**, do NOT default to standard blindly.
Assess the topic's weight, then use **AskUserQuestion with 2 questions** to let
the user configure the panel in one interaction.

#### Light topics (naming, minor UX tweak, simple yes/no)
No questions needed. Just run Standard + Sonnet silently.

#### Medium / Heavy topics
Present both questions together using AskUserQuestion. Set the recommended option
based on topic weight (Medium → Full + Balanced, Heavy → Extrafull + All Opus).

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
          "label": "Extrafull",
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
- **Heavy**: Extrafull + All Opus recommended

#### Balanced model assignment

In Balanced mode, panelists with deeper analytical roles get Opus,
practical/fresh-perspective roles get Sonnet:

| Panelist | Balanced mode model | Why |
|----------|-------------------|-----|
| **Critic** | Opus | Deep assumption-challenging needs stronger reasoning |
| **Architect** | Opus | Structural/systemic analysis benefits from depth |
| **Contrarian** | Opus | Building a coherent counter-argument is the hardest task |
| **Realist** | Sonnet | Practical trade-off evaluation works well with Sonnet |
| **Outsider** | Sonnet | Fresh-eyes simplicity check doesn't need Opus |

**Why not Haiku?** Testing showed Haiku explores codebase inefficiently,
consuming 4x more tokens than Sonnet for comparable quality — a false economy.

If the user explicitly specifies mode and/or model, skip the questions.
If the user says "just do it" or seems impatient, use the recommended defaults silently.

## Panelist Roster

### Standard mode — 2 panelists:

| Role | What they focus on |
|------|-------------------|
| **Critic** | Flawed assumptions, missing risks, alternative framings, what could go wrong |
| **Realist** | Implementation cost, maintenance burden, simpler alternatives, "is this worth it?" |

### Full mode — 4 panelists (adds):

| Role | What they focus on |
|------|-------------------|
| **Architect** | Root causes, systemic effects, long-term consequences, design coherence |
| **Outsider** | Unnecessary complexity, confusing naming, "why not just...?" questions |

### Extrafull mode — 5 panelists (adds):

| Role | What they focus on |
|------|-------------------|
| **Contrarian** | Constructs the strongest possible case for the OPPOSITE of the current approach. Not just pointing out flaws (that's Critic's job), but building a coherent argument for a completely different direction. "What if we did the exact opposite, and here's why it might actually be better..." |

The Contrarian is the most expensive but often the most valuable panelist.
While Critic finds holes in the plan, Contrarian presents an alternative universe
where the plan was never proposed and a different path was taken. This is the
"steelman the opposition" role — it forces genuine consideration of roads not taken.

## Execution Flow

### Step 1: Context Extraction

Before spawning panelists, distill the topic into a structured brief.
Raw conversation carries bias; zero context produces irrelevant analysis.
The middle ground: **facts only, no opinions.**

Extract:
- **Topic**: The specific question or decision (1-2 sentences)
- **Facts**: Objective constraints, requirements, technical realities (bullet list)
- **Current approach**: What has been decided or proposed (neutral description)
- **Stakes**: What happens if we get this wrong?

Do NOT include: opinions, justifications for the current approach, emotional framing,
or "we already tried X" (this biases against revisiting X).

**If the topic is too short or vague** (e.g., just a few words with no context),
ask the user one clarifying question before proceeding. A one-line topic without
facts/constraints will produce abstract, unhelpful analysis. Better to ask than
to waste tokens on empty output.

### Step 2: Spawn Panelists

Launch all panelists **in parallel** using the Agent tool in a single message.
Set the `model` parameter on each Agent call according to the confirmed configuration.

Use the panelist prompt template below for each, substituting the role and brief.

### Panelist Prompt Template

```
You are the [ROLE] on a review panel. Your job is to analyze a topic from your
specific perspective. You have NO prior context about this project beyond what
is provided below — this is intentional, to avoid inheriting biases from an
ongoing conversation.

## Your perspective: [ROLE]
[One sentence from the roster table above]

## Topic
[From Step 1]

## Known facts and constraints
[From Step 1]

## Current approach being considered
[From Step 1]

## Stakes
[From Step 1]

## Rules
- Be specific and concrete. "This might cause problems" is useless.
  "This breaks when X because Y" is useful.
- If you'd need to see code to give a real answer, say so explicitly
  rather than speculating.
- Limit your response to 5 bullet points maximum. 1-2 sentences each.
- You're not here to be contrarian for sport. If the current approach is
  genuinely good, say so — then point out what's still worth watching.
- Write in the same language as the topic.
```

**When `--ctx` is used**, add Read/Grep/Glob tools to each panelist and append:

```
You have access to the codebase. Before giving your analysis, explore the
relevant files to ground your opinions in actual code, not assumptions.
```

Note: with `--ctx`, panelists may reference different files. This is a feature
(broader coverage), not a bug, but keep it in mind when synthesizing.

### Step 3: Present Results

**Summary comes FIRST** — most users read only this.

```markdown
---

## Discussion Panel: [Topic in ~10 words]
*Mode: [standard/full/extrafull] | Panelists: [N]*

### Summary
- **Consensus**: [Points all panelists agreed on]
- **Tensions**: [Where panelists disagreed — state both sides. "None" if unanimous]
- **Discoveries**: [New angles nobody in the original conversation raised]

---

### Critic
- [bullet 1]
- ...

### Realist
- [bullet 1]
- ...

[### Architect — full/extrafull only]
[### Outsider — full/extrafull only]
[### Contrarian — extrafull only]

---

*Panel complete. What resonates? Want to dig deeper into any point?*
```

For **Discoveries**: only include genuinely new insights. If panelists simply
restated known concerns, write "None — panelists reinforced existing concerns
rather than surfacing new angles." Honest framing over padding.

### Step 4: Facilitate

After presenting results:

- Do NOT immediately adopt the panelists' suggestions
- Do NOT dismiss them either
- Ask the user which points resonate and which don't
- If the user wants to explore a point, discuss it or spawn a focused follow-up

The panel informs; it doesn't dictate. The USER decides.

## When NOT to use

- Simple factual questions with clear answers
- The user explicitly wants execution, not deliberation
- Rapid iteration loops where speed matters more than reflection
- Trivial decisions where the cost of being wrong is negligible

## Cost awareness

| Mode | Panelists | Default model | Approx. token multiplier |
|------|-----------|---------------|-------------------------|
| Standard | 2 (Critic, Realist) | sonnet | ~3x |
| Full | 4 (+Architect, Outsider) | sonnet (opus recommended) | ~5-6x |
| Extrafull | 5 (+Contrarian) | opus | ~7-8x |

Extrafull with Opus is the most expensive configuration but also the most thorough.
Reserve it for decisions where the cost of a wrong choice far exceeds the analysis cost.

**Model selection rationale (from empirical testing):**
- **Haiku**: Not recommended. Explores codebase inefficiently, consuming 4x more
  tokens than Sonnet for comparable or lower quality output.
- **Sonnet**: Best cost/quality ratio. Default for Standard and Full modes.
- **Opus**: Deepest analysis. Recommended for Full on important decisions, mandatory
  default for Extrafull. The user can always override.
