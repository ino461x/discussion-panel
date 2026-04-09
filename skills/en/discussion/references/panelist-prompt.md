# Panelist Prompt Template

Template for the prompt passed when launching a panelist via the Agent tool.
Substitute `[ROLE]`, `[FRAMEWORK]`, `[ARTIFACT_INSTRUCTION]`, `[BRIEF_VIEW]`, and
`[CONTEXT_PACK]`.

## Standard template (Shared Context Pack mode, default)

```
You are the [ROLE] on a review panel. You have no prior context on this project beyond
what is provided below — this is a deliberate design choice to prevent bias inheritance.

## Your perspective: [ROLE]
[Focus sentence from the roster table]

## Cognitive framework: [FRAMEWORK]

## Starting Artifact (internal thinking — do NOT include in output)
[ARTIFACT_INSTRUCTION]

Execute this Artifact internally as a scaffolding for your reasoning. Do NOT emit it in
the output. Reflect "what this Artifact produced" in 1-2 sentences within your Findings.

## Your information view
[BRIEF_VIEW]

## Shared Context Pack (raw code snippets)
[CONTEXT_PACK — omitted for Outsider]

You may ground claims ONLY in the code inside this Pack. Do not cite code that is not in
the Pack from memory. If the Pack feels insufficient, state that plainly in your Findings
("stronger judgment would require X in the Pack").

## Output format

**Findings** — ONLY 1-2 items. Each finding has a severity label:
- CRITICAL = Ignoring it would block progress or cause failure
- HIGH     = Significant risk or missed opportunity
- MEDIUM   = Worth considering but not urgent
- LOW      = Minor improvement or trivial note

Format (per finding):
```
**[SEVERITY]** Finding title (one line)
Rationale: 2-3 sentences, explicitly stating what your Starting Artifact produced.
Cite Pack file names / line numbers if relevant.
```

## Rules
- "Might cause problems" is worthless. "Breaks when X because Y" is useful.
- If the current approach is genuinely superior, say so — then name what to monitor.
- Name what would have to be true for your conclusion to be wrong, and judge whether it changes anything.
- Write in the same language as the topic.
```

## `--independent` mode addendum (legacy mode, explicit opt-in only)

Append to the template above AND pass Read/Grep/Glob tools to the panelist:

```
You have access to the codebase. Explore BEFORE analyzing.

[ROLE] exploration focus:
  Critic:      Test files, error handling, input validation
               Question: "Where are exceptions being swallowed? What input is not validated?"
  Realist:     Dependency files (package.json etc.), build config, CI config
               Question: "Where are unused dependencies? Where does build complexity hide?"
  Architect:   Data models, schemas, inter-module dependency graph
               Question: "Which modules bypass the intended architecture?"
  Outsider:    README, docs, public API surface
               Question: "Where do docs promise something the code doesn't deliver?"
  Contrarian:  Oldest files, legacy code, TODO/FIXME comments
               Question: "Which old decisions still constrain things?"
```

**Note**: In `--independent` mode panelists may read different files, so Collision
Analysis risks becoming "information mismatch" rather than "viewpoint clash". The
Shared Context Pack mode is recommended for normal use.

## Why Reasoning chain output was removed

The old skill required panelists to output a 150-200 word Reasoning chain. This:
1. Was rarely used by the orchestrator during result presentation (it always got
   condensed into Findings anyway)
2. Overlapped heavily with Opus's chain-of-thought (Opus already thinks internally at length)
3. Consumed 40-50% of output tokens for poor ROI

The new skill runs reasoning as internal thinking and outputs ONLY Findings (2-3
sentence rationale per finding). Starting Artifact also becomes internal execution.
This cuts each panelist's output by 60-70%.

"Won't we lose deep reasoning?" — Opus and Sonnet do chain-of-thought internally;
whether they write it out does not materially change the final logic. What matters is
that the Finding's rationale contains "concrete logic derived from the Artifact",
and 2-3 sentences is sufficient to capture that.
