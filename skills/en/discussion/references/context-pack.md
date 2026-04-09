# Shared Context Pack — Creation Procedure

The orchestrator explores ONCE and builds a Pack distributed to all panelists (except the
Outsider). This eliminates 5× duplicated exploration and makes Collision Analysis work as
"viewpoint clash" rather than "information mismatch".

## Procedure

### 1. Identify target area from the topic

From the brief's topic sentence, extract the keywords and file types to explore. Examples:
- "Chat feature refactoring" → `services/chat_*.py`, `routes/chat.py`, tests
- "API performance issue" → the relevant endpoint, repo, migrations
- "Auth flow redesign" → auth middleware, session, user routes

### 2. Glob for relevant files

```
Glob: services/*chat*.py
Glob: routes/*chat*.py
Glob: tests/*chat*
```

Narrow to 3-5 primary files. Don't cast too wide.

### 3. Grep for the core sections

If specific symbols or function names are known, Grep for line numbers:
```
Grep: "def handle_request" services/chat_service.py
```

### 4. Read and extract raw snippets

From each primary file, extract topic-relevant blocks with line numbers.
**Add no summaries or interpretation.** Collect raw material only.

### 5. Assemble the Pack

````markdown
# Shared Context Pack

## Related file structure
- services/chat_service.py (~1200 lines): main request flow, preprocessing, history
- routes/chat.py (~300 lines): HTTP entry point, validation
- tests/test_chat.py: existing coverage

## Snippet 1: services/chat_service.py:120-160 (handle_request)
```python
def handle_request(user_id, message, history, ...):
    # ... raw code ...
```

## Snippet 2: services/chat_service.py:340-370 (filter_history)
```python
...
```

## Related recent commits (if any)
- abc1234 perf: reduce duplicate reads in hot path
- def5678 refactor: split request handler into stages
````

## Size guidelines

- Target **3-5k tokens total**
- If it grows too large, cut file count (better than shortening every snippet)
- Don't explain "why this snippet is included". Let the panelists interpret it per role.

## When to skip

Do not build a Pack for:
- Non-technical topics (product strategy, prioritization, UX judgments)
- Decisions that don't need code access (naming, documentation structure)
- When the user explicitly requested `--independent`

## Distribution rule for Outsider

The Outsider is deliberately kept on a blank slate and does NOT receive the Pack.
Detailed information-hygiene rules are in `information-distribution.md`.

## `--independent` flag

Only if the user explicitly writes `/discussion [topic] --independent`: fall back to the
old mode (panelists each hold Read/Grep/Glob and explore independently). Do NOT build a Pack.
Instead, pass Read/Grep/Glob to each panelist's Agent call and append the
"independent mode addendum" from `panelist-prompt.md`.
