<p align="right"><strong>English</strong> | <a href="README.ja.md">日本語</a></p>

<p align="center">
  <img src="assets/banner.png" alt="Discussion Panel">
</p>

<h1 align="center">Discussion Panel</h1>

<p align="center">
  <strong>Multi-perspective analysis skill for Claude Code</strong><br>
  Break free from confirmation bias. Get fresh eyes on every decision.
</p>

<p align="center">
  <a href="#installation">Install</a> &bull;
  <a href="#usage">Usage</a> &bull;
  <a href="#modes">Modes</a> &bull;
  <a href="#output-format">Output</a> &bull;
  <a href="#how-it-works">How it works</a> &bull;
  <a href="#deep-dive">Deep dive</a> &bull;
  <a href="#examples">Examples</a>
</p>

---

## What's new in v4.0.0

- **Shared Context Pack** — The orchestrator explores the codebase ONCE and distributes raw
  snippets to all panelists. The old "5 panelists each run Read/Grep independently" pattern
  is gone. Token usage drops ~50-60%, and Collision Analysis sees real viewpoint clashes
  instead of information mismatches.
- **Findings-only output** — Panelists no longer emit 150-200 word Reasoning chains.
  Reasoning happens internally; output is 1-2 Findings with 2-3 sentence rationale. Output
  tokens per panelist drop ~60-70%.
- **Balanced is the default** — Max mode now defaults to Balanced (not All Opus). All Opus
  is an explicit opt-in. Realist and Outsider have been empirically shown to work well on
  Sonnet.
- **Skeleton split** — SKILL.md is now a short skeleton; detailed procedures live under
  `references/` and are loaded on demand (Anthropic's Progressive Disclosure pattern).
- **`--independent` flag** — Reverts to the old per-panelist exploration mode. Rarely
  needed; Shared Context Pack is sufficient for most topics.

## The Problem

When you work 1-on-1 with an AI for a long session, dangerous patterns emerge:

- **Confirmation drift** - the AI adapts to your framing and stops pushing back
- **Tunnel vision** - you only see what's already been discussed
- **Band-aid fixes** - surface patches instead of root-cause solutions
- **Unchallenged assumptions** - wrong premises survive because nobody re-examines them

## The Solution

`/discussion` spawns **independent sub-agents with fresh context** who analyze your topic from different perspectives. They have no stake in your current conversation's conclusions.

Each panelist uses a different **cognitive framework** (Pre-mortem, First Principles, Steel-man inversion, etc.) and receives a **different view of the facts** — so even though they're the same model, their thinking genuinely diverges.

## Requirements

- **Claude Code** (CLI, Desktop App, or IDE extension)
- **Claude Pro / Max / Team / Enterprise plan** (sub-agents require the Agent tool)
- Standard mode spawns 2 agents, Full spawns 4, Max spawns 5

## Installation

```bash
git clone https://github.com/ino461x/discussion-panel.git
```

Copy the skill directories into your project's `.claude/skills/` folder. **Include the `references/` directory** — v4.0.0 depends on it:

```bash
# From your project root
mkdir -p .claude/skills
cp -r discussion-panel/skills/en/discussion .claude/skills/
cp -r discussion-panel/skills/en/panel .claude/skills/
```

> **Japanese version:** `cp -r discussion-panel/skills/ja/discussion .claude/skills/` (and `/panel`)

Both `/discussion` and `/panel` invoke the same skill — `panel` is simply a shorter alias for convenience.

### Updating

```bash
cd discussion-panel && git pull
cp -r skills/en/discussion /path/to/your/project/.claude/skills/
cp -r skills/en/panel /path/to/your/project/.claude/skills/
```

## Usage

```
/discussion Should we migrate from REST to GraphQL?
/panel Is this the right database schema?
```

That's it. The skill will assess the topic's weight and ask you to configure the panel:

**Q1: Scale** - How many panelists?

| Standard | Full | Max |
|----------|------|-----------|
| 2 panelists | 4 panelists | 5 panelists |
| Critic, Realist | + Architect, Outsider | + Contrarian |
| Quick check | Design decisions | High-stakes calls |

**When to use which mode:**

| Your situation | Recommended |
|----------------|-------------|
| Quick sanity check, single-file change, easily reversible | Standard |
| Design decision, multi-file refactor, choosing between approaches | Full |
| High-stakes architecture call, hard-to-reverse migration, team-wide impact | Max |

**Q2: Model** - What quality level?

| All Sonnet | Balanced (Recommended) | All Opus |
|------------|-------------------------|----------|
| Fast & cheap | Best cost/quality — **default for Full and Max** | Maximum depth, explicit opt-in |
| All panelists use Sonnet | Critic/Architect/Contrarian use Opus, others use Sonnet | All panelists use Opus |

For light topics, it just runs without asking.

## Modes

Each panelist has a **cognitive framework** and a **Starting Artifact** — a thinking
exercise executed internally before producing findings. Starting Artifacts are NOT emitted
in output (v4.0.0 change). This forces genuine divergence from the same underlying model.

### Standard (2 panelists)

| Role | Focus | Framework | Starting Artifact |
|------|-------|-----------|-------------------|
| **Critic** | Assumptions, risks, alternatives | Pre-mortem + 5 Whys | Think through 3 failure scenarios |
| **Realist** | Cost, maintenance, simpler paths | Cost-benefit estimation | Think through per-phase person-days |

### Full (4 panelists)

Adds:

| Role | Focus | Framework | Starting Artifact |
|------|-------|-----------|-------------------|
| **Architect** | Root causes, systemic effects, missed synergies | First Principles | Map the dependency chain |
| **Outsider** | Unnecessary complexity, "why not just...?" | Beginner's Mind | Name a non-software domain that solved this |

### Max (5 panelists)

Adds:

| Role | Focus | Framework | Starting Artifact |
|------|-------|-----------|-------------------|
| **Contrarian** | Strongest case for the **opposite** approach | Steel-man inversion | Describe what was built in a world where this approach was never proposed |

## Output Format

Each panelist produces:
1. **1-2 Findings** with severity ratings and 2-3 sentence rationale
2. The rationale explicitly reflects "what the Starting Artifact produced"

Results are synthesized into:

| Section | Content |
|---------|---------|
| **Summary** | Agreement, tension, and discovery |
| **Findings Table** | All findings sorted by severity (CRITICAL > HIGH > MEDIUM > LOW) |
| **Collision Analysis** | (Full/Max) where panelists contradicted each other — and what third conclusion follows |
| **Findings Validation** | (Full/Max) orchestrator's fact-check against actual design/code |

Severity is assigned by each panelist based on their analysis, then sorted in the final synthesis.

Severity definitions:

| Severity | Meaning |
|----------|---------|
| **CRITICAL** | Blocks progress or causes failure if ignored |
| **HIGH** | Significant risk or missed opportunity |
| **MEDIUM** | Worth considering but not urgent |
| **LOW** | Minor improvement or nitpick |

## How It Works

1. **Brief extraction** — Facts categorized (technical, business, user behavior, implicit assumptions). User's position preserved as an attackable target.
2. **Shared Context Pack** — Orchestrator explores the codebase ONCE and assembles raw snippets (v4.0.0 change). Pack is distributed to all panelists except the Outsider.
3. **Differentiated input** — Each panelist gets a different view of the brief. Outsider sees topic + stakes only, on a deliberate blank slate.
4. **Starting Artifact (internal)** — Mandatory thinking exercise (failure scenarios, cost estimates, dependency maps) before Findings. Executed internally — not emitted.
5. **Parallel panelist launch** — All panelists spawned via Agent tool in parallel. No tools passed unless `--independent`.
6. **Collision Analysis** (Full/Max) — A fresh sub-agent examines Findings + Artifact summaries for contradictions: "If both are right, what third conclusion follows?"
7. **Findings Validation** (Full/Max) — Orchestrator fact-checks each Finding against actual design/code.
8. **Facilitation** — You decide what to act on.

## Deep Dive

### Differentiated Input

Each panelist receives a **different view** of the same facts:

| Panelist | Information view |
|----------|-----------------|
| Critic | All facts + implicit assumptions highlighted; also asked to list 3 questions the brief does NOT address; receives Shared Context Pack |
| Realist | All facts + technical & business constraints highlighted; receives Shared Context Pack |
| Architect | All facts + dependency/structural info highlighted; receives Shared Context Pack |
| Outsider | **Topic and stakes ONLY** (intentional blank slate; topic only for non-technical); **no Pack** |
| Contrarian | All facts + user's position placed prominently; receives Shared Context Pack |

### File layout (v4.0.0)

```
skills/en/discussion/
├── SKILL.md                    # Skeleton — invocation, modes, flow
└── references/
    ├── brief-format.md         # Brief extraction rules
    ├── context-pack.md         # Shared Context Pack procedure
    ├── information-distribution.md
    ├── panelist-prompt.md      # Full prompt template
    ├── collision-analyst.md    # Full Collision Analyst prompt
    └── output-format.md        # Result presentation format
```

The orchestrator loads `references/*.md` on demand (Progressive Disclosure). Only SKILL.md
is always resident.

## Examples

### Architecture Review

```
/discussion full Is our routes layer too fat?
```

> **Findings**
>
> | # | Severity | Finding | Panelist |
> |---|----------|---------|----------|
> | 1 | CRITICAL | `handle_payment` has no transaction boundary — partial writes on failure | Architect |
> | 2 | HIGH | Routes import DB directly, bypassing service layer | Critic |
> | 3 | HIGH | Moving code to services just moves the fat — need to split by domain | Realist |
> | 4 | MEDIUM | Split routes by resource (users, orders, payments) like the existing auth module | Outsider |

> **Collision Analysis**:
> Critic said "routes bypass service layer" while Realist said "moving to services just moves the fat."
> If both are correct: the fix isn't extracting to services — it's splitting by domain first, THEN extracting.

## Flags

| Flag | Effect |
|------|--------|
| `--independent` | Revert to legacy per-panelist exploration mode (panelists get Read/Grep/Glob). Rarely needed — Shared Context Pack is default and usually sufficient. |

**v4.0.0 change**: The old `--ctx` / `--no-ctx` flags are removed. Technical topics now
always build a Shared Context Pack (orchestrator explores once); non-technical topics skip
it entirely. Use `--independent` only if you specifically want every panelist to explore
separately.

## Honest Framing

These panelists are role-played perspectives from the same underlying model, not truly independent thinkers. They're valuable because:

- **Cognitive frameworks** force different thinking processes, not just different labels
- **Differentiated input** means each panelist literally sees different information
- **Starting Artifacts** make them think before they opine
- **Collision Analysis** extracts emergent insights from contradictions

They're **not** a substitute for actual peer review, domain expertise, or user testing.

## License

MIT
