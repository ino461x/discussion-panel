# Shared Context Pack — Creation Procedure

**Why share?**: The old skill had each panelist use Read/Grep to explore code independently.
This generated 5× duplicated exploration and caused token use to scale out. Worse, because
each panelist read different files, Collision Analysis became "information mismatches"
instead of "viewpoint clashes".

The orchestrator now explores ONCE and builds a Pack that is distributed to all panelists
(except the Outsider). This cuts tokens 60-80% AND improves collision quality.

## Procedure

### 1. Identify target area from the topic

From the brief's topic sentence, extract the keywords and file types to explore. Examples:
- "AI chat refactoring" → `services/ai_chat_service.py`, `routes_ai.py`, tests
- "API performance issue" → the relevant endpoint, repo, migrations
- "Auth flow redesign" → auth middleware, session, user routes

### 2. Glob for relevant files

```
Glob: services/*chat*.py
Glob: routes/*ai*.py
Glob: tests/*ai_chat*
```

Narrow to 3-5 primary files. Don't cast too wide.

### 3. Grep for the core sections

If specific symbols or function names are known, Grep for line numbers:
```
Grep: "def execute_chat" services/ai_chat_service.py
Grep: "_build_bookmark_context" --output content -n
```

### 4. Read and extract raw snippets

From each primary file, extract topic-relevant blocks with line numbers.
**Add no summaries or interpretation.** Collect raw material only.

### 5. Assemble the Pack

```markdown
# Shared Context Pack

## Related file structure
- services/ai_chat_service.py (2100 lines): execute_chat 3-pass unification, ExclusionCtx
- routes_ai.py (450 lines): LINE webhook, search API adapters
- gemini/chat.py (320 lines): stable_context / dynamic_hint split

## Snippet 1: services/ai_chat_service.py:450-490 (execute_chat entrypoint)
```python
def execute_chat(user_id, message, history, ...):
    # ... raw code ...
```

## Snippet 2: services/ai_chat_service.py:780-810 (ExclusionCtx.filter_history)
```python
...
```

## Related recent commits (if any)
- 70d6977 perf(stats): aggregate /api/stats into a single RPC
- c60b9c6 refactor(ai-chat): remove legacy context generation from the main path
```

## Size guidelines

- Target **3-5k tokens total**
- If it grows too large, cut file count (better than shortening every snippet)
- Don't explain "why this snippet is included". Let the panelists interpret it per role.

## When to skip

Do not build a Pack for:
- Non-technical topics (product strategy, prioritization, UX judgments)
- Decisions that don't need code access (naming, documentation structure)
- When the user explicitly requested `--independent` (old exploration mode)

## Distribution rule for Outsider

The Outsider is **deliberately kept on a blank slate**. Do NOT give them the Pack. Also
strip implementation-specific terms (function names, file paths, library names) from the
stakes. The Outsider receives only the topic and stakes, and thinks from
"where has a similar class of problem been solved in a non-software field?"

## `--independent` flag

Only if the user explicitly writes `/discussion [topic] --independent`: fall back to the
old mode (panelists each hold Read/Grep/Glob and explore independently). Do NOT build a Pack.

In that case only: pass Read/Grep/Glob to each panelist's Agent call, and append the
"independent mode addendum" from `references/panelist-prompt.md`.
