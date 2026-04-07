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

## Installation

```bash
# Clone and install
git clone https://github.com/YOUR_ACCOUNT/discussion-panel.git
```

Copy the `skills/discussion/` and `skills/panel/` directories into your Claude Code skills directory.

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

## How It Works

```
1. Context Extraction
   Your topic is distilled into facts, constraints, and stakes.
   Opinions and justifications are stripped to prevent bias transfer.

2. Parallel Panel Spawn
   Each panelist is launched as an independent sub-agent with ONLY
   the structured brief - no conversation history, no prior conclusions.

3. Analysis
   Each panelist produces up to 5 concrete, actionable bullet points.

4. Synthesis
   Results are presented with a summary first:
   - Consensus: what all panelists agreed on
   - Tensions: where they disagreed
   - Discoveries: genuinely new angles

5. Facilitation
   You decide which points to act on. The panel informs, it doesn't dictate.
```

## Examples

### Design Decision

```
/discussion Should we unify the LP and dashboard designs?
```

Output:
> **Consensus**: Full unification is unnecessary and risky. Multi-theme system conflicts with Glass Garden's fixed design.
>
> **Tensions**: None (unanimous)
>
> **Discoveries**: "Add LP's brand color as one dashboard theme" achieves brand continuity at 1/10 the cost.

### Architecture Review

```
/discussion full Is our routes layer too fat? --ctx
```

Output:
> **Consensus**: Problem isn't line count but responsibility mixing. Direct repo imports from routes violate the 3-layer architecture.
>
> **Tensions**: Critic says "move to services is just moving the fat" vs Realist says "split routes files like bookmarks pattern, add a no-DB-in-routes rule"
>
> **Discoveries**: `api_delete_account` has no transaction boundary design - fixing this matters more than moving code around.

### Balanced Model in Action

With `Balanced` model selection, the heavy-thinking roles (Critic, Architect, Contrarian) run on **Opus** while practical roles (Realist, Outsider) run on **Sonnet** - optimal cost/quality ratio.

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
