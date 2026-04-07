<p align="center">
  <img src="assets/banner.png" alt="Discussion Panel" width="600">
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
  <a href="#examples">Examples</a>
</p>

---

## The Problem

When you work 1-on-1 with an AI for a long session, dangerous patterns emerge:

- **Confirmation drift** - the AI adapts to your framing and stops pushing back
- **Tunnel vision** - you only see what's already been discussed
- **Band-aid fixes** - surface patches instead of root-cause solutions
- **Unchallenged assumptions** - wrong premises survive because nobody re-examines them

## The Solution

`/discussion` spawns **independent sub-agents with fresh context** who analyze your topic from different perspectives. They have no stake in your current conversation's conclusions.

Think of it as convening a review panel - not to override your judgment, but to make sure you've seen the full picture before committing.

## Requirements

- **Claude Code** (CLI, Desktop App, or IDE extension)
- **Claude Pro / Max / Team / Enterprise plan** (sub-agents require the Agent tool)
- Standard mode spawns 2 agents, Full spawns 4, Extrafull spawns 5

## Installation

```bash
git clone https://github.com/ino461x/discussion-panel.git
```

Copy the skill directories into your project's `.claude/skills/` folder:

```bash
# From your project root
mkdir -p .claude/skills
cp -r discussion-panel/skills/discussion .claude/skills/
cp -r discussion-panel/skills/panel .claude/skills/
```

Both `/discussion` and `/panel` invoke the same skill — `panel` is simply a shorter alias for convenience.

## Usage

```
/discussion Should we migrate from REST to GraphQL?
/panel Is this the right database schema?
```

That's it. The skill will assess the topic's weight and ask you to configure the panel:

**Q1: Scale** - How many panelists?

| Standard | Full | Extrafull |
|----------|------|-----------|
| 2 panelists | 4 panelists | 5 panelists |
| Critic, Realist | + Architect, Outsider | + Contrarian |
| Quick check | Design decisions | High-stakes calls |

**Q2: Model** - What quality level?

| All Sonnet | Balanced | All Opus |
|------------|----------|----------|
| Fast & cheap | Best cost/quality | Maximum depth |
| All panelists use Sonnet | Key roles use Opus | All panelists use Opus |

**Why 2 questions?** Scale controls *breadth* (how many perspectives), Model controls *depth* (how strong each perspective is). This lets you optimize cost vs. thoroughness for each decision.

For light topics, it just runs without asking.

## Modes

### Standard (2 panelists)

| Role | Focus |
|------|-------|
| **Critic** | Challenges assumptions, finds risks, suggests alternatives |
| **Realist** | Evaluates cost, maintenance burden, simpler alternatives |

### Full (4 panelists)

Adds:

| Role | Focus |
|------|-------|
| **Architect** | Root causes, systemic effects, long-term consequences |
| **Outsider** | Unnecessary complexity, "why not just...?" questions |

### Extrafull (5 panelists)

Adds:

| Role | Focus |
|------|-------|
| **Contrarian** | Builds the strongest case for the **opposite** approach |

The Contrarian doesn't just poke holes (that's Critic's job). They construct a coherent argument for a completely different direction: *"What if we did the exact opposite, and here's why it might actually be better..."*

### Balanced Model in Action

With `Balanced` model selection, the heavy-thinking roles (Critic, Architect, Contrarian) run on **Opus** while practical roles (Realist, Outsider) run on **Sonnet** - optimal cost/quality ratio.

## Output Format

Results are presented as a severity-rated findings table for quick scanning:

| Severity | Meaning |
|----------|---------|
| **CRITICAL** | Blocks progress or causes failure if ignored |
| **HIGH** | Significant risk or missed opportunity |
| **MEDIUM** | Worth considering but not urgent |
| **LOW** | Minor improvement or nitpick |

Example output:

```
## Discussion Panel: Should we migrate to microservices?
Mode: full | Panelists: 4

### Summary
- Consensus: Current monolith is the bottleneck for team scaling
- Tensions: Critic says "migrate selectively" vs Architect says "strangler fig pattern"
- Discoveries: The auth module alone causes 60% of deploy conflicts

### Findings

| # | Severity | Finding                                                          | Panelist  |
|---|----------|------------------------------------------------------------------|-----------|
| 1 | CRITICAL | No circuit breaker design — one service failure cascades         | Architect |
| 2 | HIGH     | Team has zero distributed systems experience                     | Realist   |
| 3 | HIGH     | Current test suite assumes single-process — breaks on migration  | Critic    |
| 4 | MEDIUM   | Start with auth module extraction only — lowest risk validation  | Outsider  |
| 5 | MEDIUM   | Define service boundaries before writing any code                | Architect |
```

## How It Works

```
1. Context Extraction
   Your topic is distilled into facts, constraints, and stakes.
   Opinions and justifications are stripped to prevent bias transfer.

2. Parallel Panel Spawn
   Each panelist is launched as an independent sub-agent with ONLY
   the structured brief - no conversation history, no prior conclusions.

3. Analysis
   Each panelist produces up to 5 severity-rated findings.

4. Synthesis
   Results are presented with a summary first:
   - Consensus: what all panelists agreed on
   - Tensions: where they disagreed
   - Discoveries: genuinely new angles
   Then a unified findings table sorted by severity.

5. Facilitation
   You decide which points to act on. The panel informs, it doesn't dictate.
```

## Examples

### Design Decision

```
/discussion Should we unify the marketing site and dashboard designs?
```

Output:
> **Consensus**: Full unification is unnecessary and risky. Different audiences need different UX.
>
> **Tensions**: None (unanimous)
>
> **Discoveries**: "Add the marketing site's brand color as one dashboard theme" achieves brand continuity at 1/10 the cost.

### Architecture Review

```
/discussion full Is our routes layer too fat? --ctx
```

Output:

> | # | Severity | Finding | Panelist |
> |---|----------|---------|----------|
> | 1 | CRITICAL | `delete_account` has no transaction boundary design | Architect |
> | 2 | HIGH | Routes import repo directly, bypassing service layer | Critic |
> | 3 | HIGH | Moving code to services just moves the fat — need to split by domain | Realist |
> | 4 | MEDIUM | Follow the bookmarks pattern — already proven in this codebase | Outsider |

## Flags

| Flag | Effect |
|------|--------|
| `--ctx` | Panelists can explore the codebase (Read, Grep, Glob) |

## Honest Framing

These panelists are role-played perspectives from the same underlying model, not truly independent thinkers. They're valuable because:

- Fresh context removes conversational inertia
- Structured roles force examination of specific angles
- Parallel execution surfaces things sequential conversation misses

They're **not** a substitute for actual peer review, domain expertise, or user testing.

## License

MIT
