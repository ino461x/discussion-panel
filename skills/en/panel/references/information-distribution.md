# Information Distribution — Per-Panelist Views and Artifact Parameterization

Each panelist receives a different view of the brief. Only build views for panelists
active in the current mode (Standard = Critic + Realist only).

## Distribution rules

| Panelist | Information received |
|----------|---------------------|
| **Critic** | All categories + implicit assumptions placed prominently at the top. Additional: "List 3 questions this brief does NOT address — what is conspicuously missing?" + Shared Context Pack |
| **Realist** | All categories + technical and business constraints emphasized + Shared Context Pack |
| **Architect** | All categories + technical constraints + dependency/structural info emphasized + Shared Context Pack |
| **Outsider** | Topic and stakes ONLY — no constraints, no user's position, no past attempts, **no Pack** (deliberate blank slate) |
| **Contrarian** | All categories + user's position placed prominently at the top + Shared Context Pack |

## Outsider information hygiene

Strip implementation-specific terms (function names, file paths, library names) from the
Outsider's stakes description. Describe stakes only in business/user-impact language. For
non-technical topics, omit stakes entirely and give the Outsider only the topic sentence.

Also: no Shared Context Pack. That's the source of the Outsider's value — by not seeing the
code, they keep room to ask "why is this even this complicated?"

## Dynamic Artifact parameterization

Before launching each panelist, inject a topic-specific context snippet from step 1 into
each role's Starting Artifact instruction:

| Role | What to inject |
|------|----------------|
| Critic | The specific implicit assumption to attack (the most obvious one) |
| Realist | The specific approach to estimate (the most expensive phase) |
| Architect | The primary component to map from (the most dependency-dense one) |
| Contrarian | The user's core claim to invert |
| Outsider | **Do not inject** (keep the blank slate) |

Fallback: if multiple candidates exist, prefer the most concrete.

## Example: topic is "Chat feature refactoring"

**Critic Starting Artifact instruction:**
> Think through 3 failure scenarios. In particular, start from "what if the assumption that
> the history filter always works correctly is wrong?"

**Realist Starting Artifact instruction:**
> Think through per-phase person-day estimates. In particular, prioritize the implementation
> cost and regression verification cost of "stage-splitting the request handler".

**Architect Starting Artifact instruction:**
> Think through dependency chains. In particular, starting from the main service module,
> map where the multiple entry points (HTTP / webhook / external API) diverge and converge.

**Contrarian Starting Artifact instruction:**
> Imagine a world where the user's core claim "unification reduces complexity" was never
> proposed. Describe what would have been built instead and why that alternative would have
> been considered obviously correct.

**Outsider instruction (no injection):**
> (Starting Artifact only. Do not pass topic-specific parameters.)
