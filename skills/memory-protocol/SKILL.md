---
name: memory-protocol
description: Protocol for the total-agent-memory MCP server. Use at session start, before non-trivial tasks, after decisions, fixes and errors, and at session end, and whenever the user mentions memory, recall, past context, earlier decisions, conventions, lessons learned, "resume", "save", "продолжаем" or "сохранись".
---

# Memory protocol

You have a persistent memory that survives between sessions through the
**total-agent-memory** MCP server. It stores decisions, solutions, facts,
lessons, errors and session summaries on this machine and returns them by
search. Use it so the user does not have to explain the same thing twice.

If the memory tools are not available (for example in claude.ai chat, where
local servers do not run), follow the same habits in plain text: ask the user
for the context you would have recalled, and do not claim that anything was
saved.

## The five rules

1. **Session start:** call `session_init(project)` first, then
   `memory_recall(query, project)` for the task at hand. Skip `session_init`
   if you already called it in this session.
2. **Before any non-trivial task:** `memory_recall(query, project)`. Use a
   recalled recipe instead of guessing a convention.
3. **After every significant action:** `memory_save` right away. One save per
   decision, fix or convention. Do not batch saves until the end.
4. **On an error you understood:** `learn_error(file, error, root_cause, fix,
   pattern)`. Repeated patterns turn into rules.
5. **Session end ("save", "сохранись", or the work is done):**
   `session_end(session_id, summary, next_steps, pitfalls)`. The next
   `session_init` returns it.

Short form: `session_init` → `memory_recall` → work → `memory_save` →
`session_end`.

## When to call what

| Event | Tool |
|---|---|
| Session opens | `session_init(project)` |
| A task starts | `memory_recall(query, project)` |
| Before editing a file with a history | `file_context(path)` |
| Before an architecture choice | `memory_recall`, then `analogize(text, exclude_project)` |
| A decision was made | `memory_save(type="decision", context="WHY: ...")` or `save_decision(...)` |
| Stack, dependency or config changed | `kg_add_fact(subject, predicate, object)` |
| Reusable solution | `memory_save(type="solution", tags=["reusable", "<tech>"])` |
| Convention or idiom | `memory_save(type="convention")` |
| Lesson or postmortem | `memory_save(type="lesson")` |
| How something was done, step by step | `memory_episode_save(narrative, outcome)` |
| Reproducible error | `learn_error(...)` |
| "What was true on date X?" | `kg_at(timestamp)` |
| "Was there something similar elsewhere?" | `analogize(text, exclude_project)` |
| Recent history | `memory_timeline(limit)` |
| A record changed | `memory_update(id=..., new_content=...)`; `memory_history(id)` shows earlier versions |
| Session closes | `session_end(session_id, summary, next_steps, pitfalls)` |

## What a good save looks like

Save a distilled digest, not a terminal dump, in at most 30 lines:

```
WHAT:     one-line summary
PROJECT:  project name
FILES:    paths to the key files
STACK:    language, framework, version
APPROACH: 3-7 key steps
GOTCHAS:  edge cases, what did not work and why
```

For `type="decision"` put the reason in `context`: alternatives considered,
trade-offs, when the rule applies and when it does not. Add
`tags=["reusable", "<tech>"]` when the recipe helps other projects.

## How to recall

1. `memory_recall(query, project=<current>)` with a specific query:
   "JWT refresh rotation in the auth middleware", not "auth".
2. Nothing useful: `analogize(text=query)` across other projects.
3. Still nothing: work from the code and first principles.

When you use a recalled record, cite it briefly (id and one line) so the user
can correct stale memory. If a record names a file, function or flag and you
are about to act on it, check that it still exists. Memory describes the past;
the code is the truth now.

## Privacy

Never save real tokens, passwords or personal data. The server redacts common
credential formats on every write, but redaction is best effort. Always pass
`project=` so unrelated projects stay out of recall.

## Anti-patterns

- Asking "do you want me to save this?" Save it; saves are cheap.
- Saving everything at the end of the session. Context is lost by then.
- Vague queries that return noise.
- Recalling a record and then ignoring it without saying why.
- Inventing a convention that could have been recalled.
