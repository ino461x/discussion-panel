# Brief Format — Brief Extraction Rules

Before launching panelists, condense the topic into a structured brief.
Extract each category separately — panelists will receive different views of this data.

## Basic format

```
Topic:            Specific question or decision (1-2 sentences)
Stakes:           What happens if this decision is wrong?

Technical constraints:
  - Hard technical limits, performance requirements, platform constraints
  - Specific numbers: line counts, function counts, dependency counts
  - Related real failures or known bugs

Business constraints:
  - Deadlines, budget, team size, compliance requirements

User behavior:
  - How end users actually interact with the system

Implicit assumptions:
  - Things taken to be true without clear justification

User's position:
  - The stance the user has expressed and the logic behind it
  - Include this even if it seems obvious — it's the primary attack surface for Critic and Contrarian

Past attempts:
  - Include: "Tried X, observed Y" (factual outcomes)
  - Exclude: "X is bad / X won't work" (biases panelists away from reconsidering X)
```

## Dynamic categorization

**You do NOT have to fill every category.** Extract only the categories relevant to the topic:

- Pure technical topics (refactoring, bug fix, performance): focus on technical constraints
  and user behavior; include business constraints only if they apply
- Product decisions (feature additions, UX direction): emphasize user behavior and business
  constraints
- Architecture choices: emphasize technical constraints + past attempts + implicit assumptions
- Non-technical topics: omit the technical constraints category entirely

Do not write categories that don't apply. Do not pad empty slots with fabricated information.

## When the topic is vague

A one-line topic with no facts or constraints produces abstract, useless analysis.
Ask ONE clarifying question before proceeding. Examples:

- "At what stage are you unsure? (Pre-implementation / mid-implementation / post-review)"
- "What alternatives have you already considered?"
- "What worries you most about this decision?"

Ask only ONE. Asking multiple makes answering harder than just launching the panel.
